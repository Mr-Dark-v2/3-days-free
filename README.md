--// Android "Show Taps" - Perfect Replica (Outline + Transparent Center)
--// Place this in StarterPlayerScripts (LocalScript)

local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local PlayerGui = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

--═══════════════════════════════════════════════
-- CONFIGURATION TABLE
--═══════════════════════════════════════════════
local Config = {
DOT_SIZE = 20,                             -- Android default circle size
DOT_COLOR = Color3.fromRGB(255, 255, 255),
DOT_TRANSPARENCY = 0.45,                   -- inner brightness
FADE_TIME = 0.18,                          -- linear quick fade
FADE_EASING = Enum.EasingStyle.Linear,

OUTLINE_ENABLED = true,  
OUTLINE_THICKNESS = 2,                     -- thickness of surrounding ring  
OUTLINE_TRANSPARENCY = 0.75,               -- softer than dot  

POSITION_OFFSET = Vector2.new(0, 0),  
Z_INDEX = 100,  
IGNORE_PROCESSED = true,

}
--═══════════════════════════════════════════════

-- Screen container
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TapVisualizer"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local activeDots = {}

-- Helper: create perfect circular frame
local function makeCircle(size, color, transparency, zIndex)
local frame = Instance.new("Frame")
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.Size = UDim2.fromOffset(size, size)
frame.BackgroundColor3 = color
frame.BackgroundTransparency = transparency
frame.BorderSizePixel = 0
frame.ZIndex = zIndex

local corner = Instance.new("UICorner")  
corner.CornerRadius = UDim.new(1, 0)  
corner.Parent = frame  

return frame

end

-- Create tap visuals
local function createTapVisual(position)
-- Main dot (with transparent center illusion)
local dot = makeCircle(Config.DOT_SIZE, Config.DOT_COLOR, Config.DOT_TRANSPARENCY, Config.Z_INDEX)
dot.Position = UDim2.fromOffset(position.X + Config.POSITION_OFFSET.X, position.Y + Config.POSITION_OFFSET.Y)
dot.Parent = screenGui

-- Create inner transparency (center is clearer)  
local inner = makeCircle(Config.DOT_SIZE * 0.65, Color3.fromRGB(0, 0, 0), 1, Config.Z_INDEX + 1)  
inner.Position = UDim2.fromScale(0.5, 0.5)  
inner.Parent = dot  

-- Outline  
local outline  
if Config.OUTLINE_ENABLED then  
	outline = makeCircle(  
		Config.DOT_SIZE + (Config.OUTLINE_THICKNESS * 2),  
		Config.DOT_COLOR,  
		Config.OUTLINE_TRANSPARENCY,  
		Config.Z_INDEX - 1  
	)  
	outline.Position = dot.Position  
	outline.Parent = screenGui  
end  

return { dot = dot, outline = outline }

end

-- Input start
UserInputService.InputBegan:Connect(function(input, gameProcessed)
if Config.IGNORE_PROCESSED and gameProcessed then return end
if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
local visuals = createTapVisual(input.Position)
activeDots[input] = visuals
end
end)

-- Input move
UserInputService.InputChanged:Connect(function(input)
local visuals = activeDots[input]
if visuals and visuals.dot then
local pos = UDim2.fromOffset(input.Position.X + Config.POSITION_OFFSET.X, input.Position.Y + Config.POSITION_OFFSET.Y)
visuals.dot.Position = pos
if visuals.outline then visuals.outline.Position = pos end
end
end)

-- Input end (fade out both smoothly)
UserInputService.InputEnded:Connect(function(input)
local visuals = activeDots[input]
if visuals then
activeDots[input] = nil

local tweenInfo = TweenInfo.new(Config.FADE_TIME, Config.FADE_EASING, Enum.EasingDirection.Out)  

	local dotTween = TweenService:Create(visuals.dot, tweenInfo, { BackgroundTransparency = 1 })  
	dotTween:Play()  

	if visuals.outline then  
		local outlineTween = TweenService:Create(visuals.outline, tweenInfo, { BackgroundTransparency = 1 })  
		outlineTween:Play()  
		outlineTween.Completed:Connect(function()  
			if visuals.outline then visuals.outline:Destroy() end  
		end)  
	end  

	dotTween.Completed:Connect(function()  
		if visuals.dot then visuals.dot:Destroy() end  
	end)  
end

end)

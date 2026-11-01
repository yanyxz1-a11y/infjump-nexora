--!strict
-- Services
local Players: Players = cloneref and cloneref(game:GetService("Players")) or game:GetService("Players")
local UserInputService: UserInputService = cloneref and cloneref(game:GetService("UserInputService")) or game:GetService("UserInputService")
local CoreGui: CoreGui = cloneref and cloneref(game:GetService("CoreGui")) or game:GetService("CoreGui")

local localPlayer: Player = Players.LocalPlayer

-- Determine safe GUI container (UNC compatibility)
local function getGuiContainer(): Instance
    if gethui then
        local success, hui = pcall(gethui)
        if success and hui then
            return hui
        end
    end
    
    local success, _ = pcall(function()
        return CoreGui.Name
    end)
    if success then
        return CoreGui
    end

    return localPlayer:WaitForChild("PlayerGui")
end

-- State
local isEnabled: boolean = false

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InfiniteJump_UI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = getGuiContainer()

-- Main Container Window
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 220, 0, 130)
mainFrame.Position = UDim2.new(0.5, -110, 0.4, -65)
mainFrame.BackgroundColor3 = Color3.fromRGB(24, 26, 32)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 8)
frameCorner.Parent = mainFrame

local frameStroke = Instance.new("UIStroke")
frameStroke.Color = Color3.fromRGB(48, 52, 64)
frameStroke.Thickness = 1.5
frameStroke.Parent = mainFrame

-- Title Bar
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, 0, 0, 36)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(18, 20, 24)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "Infinite Jump"
titleLabel.TextColor3 = Color3.fromRGB(240, 240, 240)
titleLabel.TextSize = 14
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleLabel

-- Toggle Button
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0.85, 0, 0, 36)
toggleButton.Position = UDim2.new(0.075, 0, 0, 48)
toggleButton.BackgroundColor3 = Color3.fromRGB(205, 50, 50) -- Red initially (Disabled)
toggleButton.BorderSizePixel = 0
toggleButton.Font = Enum.Font.GothamSemibold
toggleButton.Text = "Status: Disabled"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextSize = 13
toggleButton.AutoButtonColor = false
toggleButton.Parent = mainFrame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 6)
buttonCorner.Parent = toggleButton

-- Hint / Footer Label
local hintLabel = Instance.new("TextLabel")
hintLabel.Name = "HintLabel"
hintLabel.Size = UDim2.new(1, 0, 0, 22)
hintLabel.Position = UDim2.new(0, 0, 0, 94)
hintLabel.BackgroundTransparency = 1
hintLabel.Text = "Click button to toggle"
hintLabel.TextColor3 = Color3.fromRGB(140, 145, 155)
hintLabel.TextSize = 11
hintLabel.Font = Enum.Font.Gotham
hintLabel.Parent = mainFrame

-- Update Visuals
local function updateButtonVisuals()
    if isEnabled then
        toggleButton.Text = "Status: Enabled"
        toggleButton.BackgroundColor3 = Color3.fromRGB(45, 175, 80) -- Green
    else
        toggleButton.Text = "Status: Disabled"
        toggleButton.BackgroundColor3 = Color3.fromRGB(205, 50, 50) -- Red
    end
end

-- Button Click Listener
toggleButton.MouseButton1Click:Connect(function()
    isEnabled = not isEnabled
    updateButtonVisuals()
end)

-- Draggable Window Logic
local isDragging = false
local dragInput: InputObject? = nil
local dragStart: Vector3 = Vector3.zero
local startPos: UDim2 = UDim2.new()

mainFrame.InputBegan:Connect(function(input: InputObject)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                isDragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input: InputObject)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input: InputObject)
    if input == dragInput and isDragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- Infinite Jump Core Mechanics
UserInputService.JumpRequest:Connect(function()
    if not isEnabled then
        return
    end

    local character = localPlayer.Character
    if not character then
        return
    end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.Health > 0 then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

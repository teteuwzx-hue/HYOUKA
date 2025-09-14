--[[
    HYOUKA Universal Script for Roblox
    Features:
        - Floating, draggable, minimizable, and closable panel
        - Stylish large gray UI
        - Aim FOV circle (toggleable)
        - Aimbot (basic, customizable)
        - "Spe Line": Draw lines to friends (green) and enemies (red)
    Note: This script is universal, but features like friends/enemies may require adaptation for specific games.
]]

-- Universal Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

-- UI Library (uses Instance)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HYOUKA"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- Main Panel
local Panel = Instance.new("Frame")
Panel.Size = UDim2.new(0, 480, 0, 320)
Panel.Position = UDim2.new(0.5, -240, 0.5, -160)
Panel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
Panel.BorderSizePixel = 0
Panel.AnchorPoint = Vector2.new(0.5, 0.5)
Panel.Active = true
Panel.Draggable = true
Panel.Parent = ScreenGui

-- Panel Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 38)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Text = "HYOUKA"
Title.TextSize = 32
Title.TextColor3 = Color3.fromRGB(200, 200, 200)
Title.Parent = Panel

-- Close Button
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 38, 0, 38)
CloseBtn.Position = UDim2.new(1, -38, 0, 0)
CloseBtn.BackgroundColor3 = Color3.fromRGB(90,90,90)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 24
CloseBtn.TextColor3 = Color3.fromRGB(220,60,60)
CloseBtn.Parent = Panel

-- Minimize Button
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size = UDim2.new(0, 38, 0, 38)
MiniBtn.Position = UDim2.new(1, -76, 0, 0)
MiniBtn.BackgroundColor3 = Color3.fromRGB(90,90,90)
MiniBtn.Text = "—"
MiniBtn.Font = Enum.Font.GothamBold
MiniBtn.TextSize = 24
MiniBtn.TextColor3 = Color3.fromRGB(200,200,200)
MiniBtn.Parent = Panel

-- Main Content Holder
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -24, 1, -48)
Content.Position = UDim2.new(0, 12, 0, 44)
Content.BackgroundTransparency = 1
Content.Parent = Panel

-- Options
local opts = {
    AimFov = true,
    Aimbot = false,
    FovRadius = 100,
    SpeLine = true
}

-- Option toggles
local function makeToggle(name, pos, default, callback)
    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(0, 220, 0, 38)
    Toggle.Position = UDim2.new(0, 0, 0, pos)
    Toggle.BackgroundColor3 = Color3.fromRGB(80,80,80)
    Toggle.Font = Enum.Font.GothamSemibold
    Toggle.TextSize = 22
    Toggle.TextColor3 = Color3.fromRGB(220,220,220)
    Toggle.Text = name .. ": " .. (default and "ON" or "OFF")
    Toggle.Parent = Content
    Toggle.MouseButton1Click:Connect(function()
        default = not default
        Toggle.Text = name .. ": " .. (default and "ON" or "OFF")
        callback(default)
    end)
    return Toggle
end

makeToggle("Aim FOV", 0, opts.AimFov, function(v) opts.AimFov = v end)
makeToggle("Aimbot", 48, opts.Aimbot, function(v) opts.Aimbot = v end)
makeToggle("Spe Line", 96, opts.SpeLine, function(v) opts.SpeLine = v end)

-- FOV Slider
local FovLabel = Instance.new("TextLabel")
FovLabel.Size = UDim2.new(0, 220, 0, 32)
FovLabel.Position = UDim2.new(0, 0, 0, 144)
FovLabel.BackgroundTransparency = 1
FovLabel.Font = Enum.Font.GothamSemibold
FovLabel.TextSize = 18
FovLabel.TextColor3 = Color3.fromRGB(200,200,200)
FovLabel.Text = "FOV Radius: " .. opts.FovRadius
FovLabel.Parent = Content

local FovSlider = Instance.new("TextButton")
FovSlider.Size = UDim2.new(0, 220, 0, 24)
FovSlider.Position = UDim2.new(0, 0, 0, 180)
FovSlider.BackgroundColor3 = Color3.fromRGB(90,90,90)
FovSlider.Text = "Set FOV Radius (Click to +10)"
FovSlider.Font = Enum.Font.Gotham
FovSlider.TextSize = 16
FovSlider.TextColor3 = Color3.fromRGB(180,180,180)
FovSlider.Parent = Content
FovSlider.MouseButton1Click:Connect(function()
    opts.FovRadius = (opts.FovRadius + 10) % 300
    if opts.FovRadius < 50 then opts.FovRadius = 50 end
    FovLabel.Text = "FOV Radius: " .. opts.FovRadius
end)

-- Minimize / Close logic
local minimized = false
MiniBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    Content.Visible = not minimized
    Panel.Size = minimized and UDim2.new(0, 480, 0, 38) or UDim2.new(0, 480, 0, 320)
end)
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Floating panel drag fix for new Roblox
local dragging, dragInput, dragStart, startPos
Panel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Panel.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
Panel.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        Panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- FOV Circle Drawing
local FovCircle = Drawing and Drawing.new("Circle") or nil
if FovCircle then
    FovCircle.Transparency = 1
    FovCircle.Color = Color3.fromRGB(120,120,120)
    FovCircle.Thickness = 2
    FovCircle.NumSides = 100
    FovCircle.Filled = false
end

RunService.RenderStepped:Connect(function()
    if FovCircle then
        FovCircle.Visible = opts.AimFov and not minimized
        FovCircle.Position = Camera.ViewportSize/2
        FovCircle.Radius = opts.FovRadius
    end
end)

-- Basic Aimbot (Head aim, toggleable)
local function getClosestTarget()
    local maxDist = opts.FovRadius
    local closest, closestPos
    for i,player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
            local dist = (Vector2.new(pos.X, pos.Y) - Camera.ViewportSize/2).Magnitude
            if onScreen and dist < maxDist then
                maxDist = dist
                closest = player
                closestPos = pos
            end
        end
    end
    return closest, closestPos
end

UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and opts.Aimbot and input.UserInputType == Enum.UserInputType.MouseButton2 then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

-- Spe Line Drawing (uses Drawing API, may need adaptation for some exploits)
local speLines = {}

RunService.RenderStepped:Connect(function()
    -- Remove old lines
    for _,line in pairs(speLines) do
        if line then line:Remove() end
    end
    speLines = {}

    if opts.SpeLine and not minimized then
        for _,player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
                if onScreen then
                    local line = Drawing and Drawing.new("Line") or nil
                    if line then
                        line.From = Camera.ViewportSize/2
                        line.To = Vector2.new(pos.X, pos.Y)
                        line.Thickness = 2
                        line.Transparency = 1
                        if table.find(LocalPlayer:GetFriendsOnline(), player.UserId) then
                            line.Color = Color3.fromRGB(60,220,80)
                        else
                            line.Color = Color3.fromRGB(220,60,60)
                        end
                        line.Visible = true
                        table.insert(speLines, line)
                    end
                end
            end
        end
    end
end)

-- Credits
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(0, 220, 0, 24)
Credits.Position = UDim2.new(0, 0, 1, -24)
Credits.BackgroundTransparency = 1
Credits.Font = Enum.Font.Gotham
Credits.TextSize = 16
Credits.TextColor3 = Color3.fromRGB(120,120,120)
Credits.Text = "Script by teteuwzx-hue | Universal | HYOUKA"
Credits.Parent = Content

-- End of Script

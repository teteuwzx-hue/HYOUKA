--[[
  ROBLOX LUA SCRIPT - CLIENT SIDE
  - Menu cinza móvel, com minimizar/fechar
  - Freeze Player: selecionar player e congelar
  - Spectate Player: acompanhar visão do player escolhido
  - Não é visual avançado, é Ui padrão
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

--== Menu GUI Setup ==--
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GrayMenu"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 220)
MainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60) -- cinza
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(80, 80, 80) -- cinza escuro
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Text = "Gray Menu"
TitleLabel.Size = UDim2.new(0.8, 0, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.TextColor3 = Color3.new(1,1,1)
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 22
TitleLabel.Parent = TitleBar

-- Minimize & Close Buttons
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Text = "_"
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -70, 0, 5)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(120,120,120)
MinimizeBtn.TextColor3 = Color3.new(1,1,1)
MinimizeBtn.Parent = TitleBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Text = "X"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(180,80,80)
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.Parent = TitleBar

--== Menu Options ==--
local PlayerDropdown = Instance.new("TextButton")
PlayerDropdown.Text = "Selecionar Player"
PlayerDropdown.Size = UDim2.new(1, -40, 0, 30)
PlayerDropdown.Position = UDim2.new(0, 20, 0, 60)
PlayerDropdown.BackgroundColor3 = Color3.fromRGB(100,100,100)
PlayerDropdown.TextColor3 = Color3.new(1,1,1)
PlayerDropdown.Parent = MainFrame

local SelectedPlayer = nil

local FreezeBtn = Instance.new("TextButton")
FreezeBtn.Text = "ACTIVATE FREEZE PLAYER"
FreezeBtn.Size = UDim2.new(1, -40, 0, 30)
FreezeBtn.Position = UDim2.new(0, 20, 0, 100)
FreezeBtn.BackgroundColor3 = Color3.fromRGB(120,120,120)
FreezeBtn.TextColor3 = Color3.new(1,1,1)
FreezeBtn.Parent = MainFrame

local SpectateBtn = Instance.new("TextButton")
SpectateBtn.Text = "SPECTATE PLAYER"
SpectateBtn.Size = UDim2.new(1, -40, 0, 30)
SpectateBtn.Position = UDim2.new(0, 20, 0, 140)
SpectateBtn.BackgroundColor3 = Color3.fromRGB(80,120,180)
SpectateBtn.TextColor3 = Color3.new(1,1,1)
SpectateBtn.Parent = MainFrame

--== Minimize & Close ==--
local minimized = false
MinimizeBtn.MouseButton1Click:Connect(function()
    if minimized then
        MainFrame.Size = UDim2.new(0, 300, 0, 220)
        for _, obj in pairs(MainFrame:GetChildren()) do
            if obj ~= TitleBar then obj.Visible = true end
        end
        minimized = false
    else
        MainFrame.Size = UDim2.new(0, 300, 0, 40)
        for _, obj in pairs(MainFrame:GetChildren()) do
            if obj ~= TitleBar then obj.Visible = false end
        end
        minimized = true
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

--== Player Dropdown Logic ==--
local function updateDropdown()
    local menu = Instance.new("Frame")
    menu.Size = UDim2.new(0, 220, 0, #Players:GetPlayers()*30)
    menu.Position = PlayerDropdown.Position + UDim2.new(0,0,0,30)
    menu.BackgroundColor3 = Color3.fromRGB(70,70,70)
    menu.Parent = MainFrame
    menu.ZIndex = 10

    for i, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, 0, 0, 30)
            btn.Position = UDim2.new(0, 0, 0, (i-1)*30)
            btn.Text = p.Name
            btn.BackgroundColor3 = Color3.fromRGB(100,100,100)
            btn.TextColor3 = Color3.new(1,1,1)
            btn.Parent = menu

            btn.MouseButton1Click:Connect(function()
                SelectedPlayer = p
                PlayerDropdown.Text = "Selecionado: " .. p.Name
                menu:Destroy()
            end)
        end
    end

    menu.MouseLeave:Connect(function()
        menu:Destroy()
    end)
end

PlayerDropdown.MouseButton1Click:Connect(function()
    updateDropdown()
end)

--== Freeze Player Logic ==--
FreezeBtn.MouseButton1Click:Connect(function()
    if not SelectedPlayer then
        FreezeBtn.Text = "Selecione um player!"
        wait(1)
        FreezeBtn.Text = "ACTIVATE FREEZE PLAYER"
        return
    end
    -- Freeze: Anchora o HumanoidRootPart do player remoto (apenas local)
    local char = SelectedPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.Anchored = true
        FreezeBtn.Text = "Player Freezado!"
        wait(1)
        FreezeBtn.Text = "ACTIVATE FREEZE PLAYER"
    end
end)

--== Spectate Player Logic ==--
local spectating = false
SpectateBtn.MouseButton1Click:Connect(function()
    if not SelectedPlayer then
        SpectateBtn.Text = "Selecione um player!"
        wait(1)
        SpectateBtn.Text = "SPECTATE PLAYER"
        return
    end
    local cam = workspace.CurrentCamera
    if not spectating then
        spectating = true
        SpectateBtn.Text = "PARAR SPECTATE"
        cam.CameraSubject = SelectedPlayer.Character and SelectedPlayer.Character:FindFirstChild("Humanoid")
    else
        spectating = false
        SpectateBtn.Text = "SPECTATE PLAYER"
        cam.CameraSubject = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
    end
end)

--== Atualizar lista de players quando alguém entra/sai ==--
Players.PlayerAdded:Connect(function()
    PlayerDropdown.Text = "Selecionar Player"
    SelectedPlayer = nil
end)
Players.PlayerRemoving:Connect(function(p)
    if SelectedPlayer == p then
        PlayerDropdown.Text = "Selecionar Player"
        SelectedPlayer = nil
    end
end)


"Corrige o código do script Lua, arrumando o erro em que, ao selecionar o jogador na lista para spectar, os nomes ficam invisíveis, e a gente precisa clicar na lista invisível para escolher o jogador. Também corrige o erro do Freeze Player, que atualmente congela o jogador apenas para mim e não para todos. Quero que o Freeze Player funcione para que todos possam ver. Além disso, aumenta o tamanho do menu e deixa ele em um cinza transparente. Por fim, adiciona uma função para selecionar o jogador e, em seguida, usar o Kill Player para que o jogador seja morto."

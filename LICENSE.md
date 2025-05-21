--[[
    Script para Roblox - Garena Free Fire MAX
    Inclui: Aimbot, ESP (linha, caixa e nome), GUI funcional
    AVISO: Este script é apenas para fins educacionais em servidores privados com permissão.
    O uso em servidores públicos é contra os Termos de Serviço do Roblox.
]]

-- Configurações iniciais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Configurações do ESP
local ESPEnabled = true
local BoxESP = true
local LineESP = true
local NameESP = true
local TeamCheck = true

-- Configurações do Aimbot
local AimbotEnabled = true
local AimbotKey = Enum.UserInputType.MouseButton2
local Smoothness = 0.5
local FOV = 100
local TargetPart = "Head"

-- Cores
local EnemyColor = Color3.fromRGB(255, 0, 0)
local TeamColor = Color3.fromRGB(0, 255, 0)

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FreeFireHax"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 250)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = false  -- Inicia oculta
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "Free Fire MAX Hacks v1.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

-- Função para mostrar/ocultar GUI com a tecla Q
local function ToggleGUI()
    MainFrame.Visible = not MainFrame.Visible
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Q and not gameProcessed then
        ToggleGUI()
    end
end)

-- Funções de ESP
local function CreateESP(player)
    local character = player.Character
    if not character then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = player.Name .. "_ESP"
    highlight.Adornee = character
    highlight.OutlineColor = EnemyColor
    highlight.OutlineTransparency = 0
    highlight.FillTransparency = 1
    highlight.Parent = character
    
    local box = Instance.new("BoxHandleAdornment")
    box.Name = player.Name .. "_Box"
    box.Adornee = character:FindFirstChild("HumanoidRootPart") or character:WaitForChild("HumanoidRootPart")
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(2, 3.5, 1)
    box.Color3 = EnemyColor
    box.Transparency = 0.5
    box.Parent = character
    
    local line = Instance.new("LineHandleAdornment")
    line.Name = player.Name .. "_Line"
    line.Adornee = character:FindFirstChild("HumanoidRootPart") or character:WaitForChild("HumanoidRootPart")
    line.AlwaysOnTop = true
    line.ZIndex = 10
    line.Length = 100
    line.Thickness = 1
    line.Color3 = EnemyColor
    line.Parent = character
    
    local nameTag = Instance.new("BillboardGui")
    nameTag.Name = player.Name .. "_Name"
    nameTag.Adornee = character:FindFirstChild("Head") or character:WaitForChild("Head")
    nameTag.Size = UDim2.new(0, 100, 0, 40)
    nameTag.StudsOffset = Vector3.new(0, 3, 0)
    nameTag.AlwaysOnTop = true
    nameTag.Parent = character
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "NameLabel"
    nameLabel.Size = UDim2.new(1, 0, 0, 20)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = EnemyColor
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.Parent = nameTag
    
    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Name = "DistanceLabel"
    distanceLabel.Size = UDim2.new(1, 0, 0, 20)
    distanceLabel.Position = UDim2.new(0, 0, 0, 20)
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.TextColor3 = EnemyColor
    distanceLabel.Font = Enum.Font.SourceSans
    distanceLabel.Parent = nameTag
    
    -- Atualizar distância
    RunService.Heartbeat:Connect(function()
        if character and character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local distance = (character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            distanceLabel.Text = tostring(math.floor(distance)) .. " studs"
        end
    end)
end

local function RemoveESP(player)
    local character = player.Character
    if character then
        for _, v in pairs(character:GetChildren()) do
            if v.Name:find("_ESP") or v.Name:find("_Box") or v.Name:find("_Line") or v.Name:find("_Name") then
                v:Destroy()
            end
        end
    end
end

-- Função Aimbot
local function FindClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            if TeamCheck and player.Team == LocalPlayer.Team then
                continue
            end
            
            local character = player.Character
            local targetPart = character:FindFirstChild(TargetPart)
            if targetPart then
                local screenPoint, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

-- Função para suavizar o movimento do mouse
local function SmoothMove(targetPosition)
    local currentPosition = Camera.CFrame
    local delta = targetPosition - currentPosition.Position
    local smoothDelta = delta * Smoothness
    Camera.CFrame = CFrame.new(currentPosition.Position + smoothDelta, targetPosition)
end

-- Ativar/Desativar ESP
local function ToggleESP()
    ESPEnabled = not ESPEnabled
    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                CreateESP(player)
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            RemoveESP(player)
        end
    end
end

-- Ativar/Desativar Aimbot
local function ToggleAimbot()
    AimbotEnabled = not AimbotEnabled
end

-- Configurar GUI
local function CreateToggle(text, position, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0.9, 0, 0, 30)
    toggleFrame.Position = position
    toggleFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    toggleFrame.Parent = MainFrame
    
    local toggleText = Instance.new("TextLabel")
    toggleText.Size = UDim2.new(0.7, 0, 1, 0)
    toggleText.Position = UDim2.new(0, 5, 0, 0)
    toggleText.BackgroundTransparency = 1
    toggleText.Text = text
    toggleText.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleText.TextXAlignment = Enum.TextXAlignment.Left
    toggleText.Parent = toggleFrame
    
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.25, 0, 0.8, 0)
    toggleButton.Position = UDim2.new(0.75, 0, 0.1, 0)
    toggleButton.Text = "OFF"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    toggleButton.Parent = toggleFrame
    
    toggleButton.MouseButton1Click:Connect(function()
        callback()
        if toggleButton.Text == "OFF" then
            toggleButton.Text = "ON"
            toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        else
            toggleButton.Text = "OFF"
            toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        end
    end)
    
    return toggleButton
end

-- Criar toggles na GUI
CreateToggle("ESP", UDim2.new(0.05, 0, 0.15, 0), ToggleESP)
CreateToggle("Aimbot", UDim2.new(0.05, 0, 0.25, 0), ToggleAimbot)

-- Conexões de eventos
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if ESPEnabled then
            CreateESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
    -- Aimbot
    if AimbotEnabled and UserInputService:IsMouseButtonPressed(AimbotKey) then
        local closestPlayer = FindClosestPlayer()
        if closestPlayer and closestPlayer.Character then
            local targetPart = closestPlayer.Character:FindFirstChild(TargetPart)
            if targetPart then
                SmoothMove(targetPart.Position)
            end
        end
    end
    
    -- Atualizar ESP
    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local esp = player.Character:FindFirstChild(player.Name .. "_ESP")
                if not esp then
                    CreateESP(player)
                end
            end
        end
    end
end)

-- Inicializar ESP para jogadores existentes
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        CreateESP(player)
    end
end

print("Script carregado com sucesso! Pressione Q para mostrar/ocultar a GUI.")

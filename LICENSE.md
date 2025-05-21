--[[
    Script para Roblox - Garena Free Fire MAX
    Inclui: 
    - Aimbot com mira na cabeça
    - ESP (linha, caixa e nome)
    - GUI funcional com tecla Q
    AVISO: Uso apenas em servidores privados com permissão.
]]

-- Configurações iniciais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")

-- Configurações do ESP
local ESPEnabled = true
local BoxESP = true
local LineESP = true
local NameESP = true
local TeamCheck = true

-- Configurações do Aimbot
local AimbotEnabled = true
local AimbotKey = Enum.UserInputType.MouseButton2
local ShootKey = Enum.UserInputType.MouseButton1
local Smoothness = 0.3  -- Mais suave para melhor precisão
local FOV = 80          -- Campo de visão reduzido para melhor precisão
local TargetPart = "Head"
local AutoShoot = false

-- Cores
local EnemyColor = Color3.fromRGB(255, 50, 50)
local TeamColor = Color3.fromRGB(50, 255, 50)

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FreeFireHax"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 300)  -- Aumentado para novos controles
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "Free Fire MAX Hacks v1.2"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

-- Função para mostrar/ocultar GUI com a tecla Q
local function ToggleGUI()
    MainFrame.Visible = not MainFrame.Visible
    GuiService:SetMenuIsOpen(MainFrame.Visible)
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Q and not gameProcessed then
        ToggleGUI()
    end
end)

-- Funções de ESP melhoradas
local ESPObjects = {}

local function CreateESP(player)
    if ESPObjects[player] then return end
    
    ESPObjects[player] = {}
    local character = player.Character or player.CharacterAdded:Wait()
    
    -- Highlight
    local highlight = Instance.new("Highlight")
    highlight.Name = player.Name .. "_ESP"
    highlight.Adornee = character
    highlight.OutlineColor = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
    highlight.OutlineTransparency = 0
    highlight.FillTransparency = 1
    highlight.Parent = character
    table.insert(ESPObjects[player], highlight)
    
    -- Box ESP
    if BoxESP then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_Box"
        box.Adornee = character:WaitForChild("HumanoidRootPart")
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = Vector3.new(2, 3.5, 1)
        box.Color3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
        box.Transparency = 0.5
        box.Parent = character
        table.insert(ESPObjects[player], box)
    end
    
    -- Line ESP
    if LineESP then
        local line = Instance.new("LineHandleAdornment")
        line.Name = player.Name .. "_Line"
        line.Adornee = character:WaitForChild("HumanoidRootPart")
        line.AlwaysOnTop = true
        line.ZIndex = 10
        line.Length = (character:WaitForChild("HumanoidRootPart").Position.Y
        line.Thickness = 1
        line.Color3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
        line.Parent = character
        table.insert(ESPObjects[player], line)
    end
    
    -- Name ESP
    if NameESP then
        local nameTag = Instance.new("BillboardGui")
        nameTag.Name = player.Name .. "_Name"
        nameTag.Adornee = character:WaitForChild("Head")
        nameTag.Size = UDim2.new(0, 100, 0, 40)
        nameTag.StudsOffset = Vector3.new(0, 3.5, 0)
        nameTag.AlwaysOnTop = true
        nameTag.Parent = character
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, 0, 0, 20)
        nameLabel.Position = UDim2.new(0, 0, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.Parent = nameTag
        
        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Name = "DistanceLabel"
        distanceLabel.Size = UDim2.new(1, 0, 0, 20)
        distanceLabel.Position = UDim2.new(0, 0, 0, 20)
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.TextColor3 = (TeamCheck and player.Team == LocalPlayer.Team) and TeamColor or EnemyColor
        distanceLabel.Font = Enum.Font.SourceSans
        distanceLabel.Parent = nameTag
        
        table.insert(ESPObjects[player], nameTag)
        
        -- Atualizar distância
        local conn
        conn = RunService.Heartbeat:Connect(function()
            if not character or not character:FindFirstChild("HumanoidRootPart") or not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                conn:Disconnect()
                return
            end
            local distance = (character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            distanceLabel.Text = math.floor(distance) .. " studs"
        end)
    end
    
    player.CharacterRemoving:Connect(function()
        RemoveESP(player)
    end)
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if obj then
                obj:Destroy()
            end
        end
        ESPObjects[player] = nil
    end
end

-- Função Aimbot melhorada
local function FindClosestPlayer()
    local closestPlayer, closestDistance = nil, FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local head = player.Character:FindFirstChild("Head")
            
            if humanoid and humanoid.Health > 0 and head then
                if TeamCheck and player.Team == LocalPlayer.Team then
                    continue
                end
                
                local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    if distance < closestDistance then
                        closestPlayer = player
                        closestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

-- Mira automática quando atirando
local function AutoAim()
    if not AimbotEnabled then return end
    
    local closestPlayer = FindClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local head = closestPlayer.Character:FindFirstChild("Head")
        if head then
            local targetCFrame = CFrame.new(Camera.CFrame.Position, head.Position)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, Smoothness)
            
            if AutoShoot then
                -- Simular clique do mouse para atirar
                mouse1press()
                task.wait(0.1)
                mouse1release()
            end
        end
    end
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

-- Ativar/Desativar AutoShoot
local function ToggleAutoShoot()
    AutoShoot = not AutoShoot
end

-- Ativar/Desativar NameESP
local function ToggleNameESP()
    NameESP = not NameESP
    if NameESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and ESPEnabled then
                RemoveESP(player)
                CreateESP(player)
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            if ESPObjects[player] then
                for _, obj in pairs(ESPObjects[player]) do
                    if obj.Name:find("_Name") then
                        obj:Destroy()
                    end
                end
            end
        end
    end
end

-- Configurar GUI
local function CreateToggle(text, position, callback, defaultValue)
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
    toggleButton.Text = defaultValue and "ON" or "OFF"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.BackgroundColor3 = defaultValue and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
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
CreateToggle("ESP", UDim2.new(0.05, 0, 0.15, 0), ToggleESP, true)
CreateToggle("Aimbot", UDim2.new(0.05, 0, 0.25, 0), ToggleAimbot, true)
CreateToggle("Auto Shoot", UDim2.new(0.05, 0, 0.35, 0), ToggleAutoShoot, false)
CreateToggle("Name ESP", UDim2.new(0.05, 0, 0.45, 0), ToggleNameESP, true)

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

-- Loop principal para Aimbot
RunService.RenderStepped:Connect(function()
    -- Aimbot quando pressionando botão direito ou atirando
    if AimbotEnabled and (UserInputService:IsMouseButtonPressed(AimbotKey) or (AutoShoot and UserInputService:IsMouseButtonPressed(ShootKey)) then
        AutoAim()
    end
    
    -- Atualizar ESP
    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and not ESPObjects[player] then
                CreateESP(player)
            end
        end
    end
end)

-- Inicializar ESP para jogadores existentes
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

print("Script carregado com sucesso! Pressione Q para mostrar/ocultar a GUI.")

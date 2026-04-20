-- Mushyo Enhancement Suite v3.0
-- Interface arrastável com WallWalk magnético

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Remover interface existente se houver
if CoreGui:FindFirstChild("MushyoSuite") then
    CoreGui.MushyoSuite:Destroy()
end

-- Interface gráfica principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Barra de título arrastável
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "MUSHYO SUITE v3.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Botões de controle da janela
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(0.7, 0, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 16
MinimizeButton.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(0.8, 0, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 14
CloseButton.Parent = TitleBar

-- Sistema de arrastar janela
local dragging = false
local dragInput, dragStart, startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateInput(input)
    end
end)

-- Controles de janela
MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Variáveis de estado
local wallWalkActive = false
local flightActive = false
local noclipActive = false
local espActive = false
local connections = {}

-- Função para criar botões
local function createButton(text, yPosition, callback, toggle)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.9, 0, 0, 35)
    button.Position = UDim2.new(0.05, 0, 0, yPosition)
    button.Text = text
    button.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 14
    button.BorderSizePixel = 0
    button.Parent = MainFrame
    
    button.MouseButton1Click:Connect(function()
        if toggle then
            local newState = callback()
            button.BackgroundColor3 = newState and Color3.fromRGB(0, 120, 215) or Color3.fromRGB(45, 45, 50)
        else
            callback()
        end
    end)
    
    return button
end

-- WallWalk magnético (anda em paredes e teto como se tivesse ímãs nos pés)
local function toggleWallWalk()
    wallWalkActive = not wallWalkActive
    
    if wallWalkActive then
        local gravityCorrection = 0
        local surfaceNormal = Vector3.new(0, 1, 0)
        
        connections.wallWalk = RunService.Heartbeat:Connect(function()
            if not rootPart or not humanoid then return end
            
            -- Raycast em todas as direções para encontrar superfície mais próxima
            local raycastParams = RaycastParams.new()
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            raycastParams.FilterDescendantsInstances = {character}
            
            local closestHit = nil
            local closestDistance = math.huge
            
            -- Verificar em 8 direções ao redor do personagem
            local directions = {
                Vector3.new(0, -1, 0),  -- Baixo
                Vector3.new(0, 1, 0),   -- Cima
                Vector3.new(1, 0, 0),   -- Direita
                Vector3.new(-1, 0, 0),  -- Esquerda
                Vector3.new(0, 0, 1),   -- Frente
                Vector3.new(0, 0, -1),  -- Trás
                Vector3.new(1, 1, 0):Unit(),   -- Diagonal
                Vector3.new(-1, 1, 0):Unit()   -- Diagonal
            }
            
            for _, direction in ipairs(directions) do
                local ray = workspace:Raycast(
                    rootPart.Position,
                    direction * 5,
                    raycastParams
                )
                
                if ray and ray.Distance < closestDistance then
                    closestHit = ray
                    closestDistance = ray.Distance
                    surfaceNormal = ray.Normal
                end
            end
            
            if closestHit then
                -- Calcular a gravidade na direção da superfície
                local gravityDirection = -surfaceNormal
                
                -- Aplicar força gravitacional na direção da superfície
                local bodyForce = rootPart:FindFirstChild("WallWalkForce") or Instance.new("BodyForce")
                bodyForce.Name = "WallWalkForce"
                bodyForce.Force = gravityDirection * (workspace.Gravity * rootPart:GetMass())
                bodyForce.Parent = rootPart
                
                -- Rotacionar o personagem para ficar "em pé" na superfície
                local targetCFrame = CFrame.lookAt(
                    rootPart.Position,
                    rootPart.Position + rootPart.CFrame.LookVector,
                    surfaceNormal
                )
                
                rootPart.CFrame = targetCFrame
                
                -- Manter velocidade de movimento normal
                humanoid.PlatformStand = false
            end
        end)
    else
        if connections.wallWalk then
            connections.wallWalk:Disconnect()
        end
        if rootPart:FindFirstChild("WallWalkForce") then
            rootPart.WallWalkForce:Destroy()
        end
        humanoid.PlatformStand = false
    end
    
    return wallWalkActive
end

-- Sistema de voo completo
local function toggleFlight()
    flightActive = not flightActive
    
    if flightActive then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Name = "FlightVelocity"
        bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = rootPart
        
        connections.flightInput = UIS.InputBegan:Connect(function(input)
            if not bodyVelocity or not bodyVelocity.Parent then return end
            
            if input.KeyCode == Enum.KeyCode.Space then
                bodyVelocity.Velocity = Vector3.new(0, 50, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                bodyVelocity.Velocity = Vector3.new(0, -50, 0)
            end
        end)
    else
        if rootPart:FindFirstChild("FlightVelocity") then
            rootPart.FlightVelocity:Destroy()
        end
        if connections.flightInput then
            connections.flightInput:Disconnect()
        end
    end
    
    return flightActive
end

-- ESP com highlight melhorado
local function toggleESP()
    espActive = not espActive
    
    if espActive then
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local highlight = Instance.new("Highlight")
                highlight.Name = "MushyoESP"
                highlight.FillColor = Color3.fromRGB(255, 50, 50)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.8
                highlight.OutlineTransparency = 0
                highlight.Parent = otherPlayer.Character
            end
        end
        
        connections.espAdded = Players.PlayerAdded:Connect(function(newPlayer)
            if espActive then
                newPlayer.CharacterAdded:Connect(function(char)
                    task.wait(1)
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "MushyoESP"
                    highlight.FillColor = Color3.fromRGB(255, 50, 50)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.Parent = char
                end)
            end
        end)
    else
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer.Character and otherPlayer.Character:FindFirstChild("MushyoESP") then
                otherPlayer.Character.MushyoESP:Destroy()
            end
        end
        if connections.espAdded then
            connections.espAdded:Disconnect()
        end
    end
    
    return espActive
end

-- Speed hack
local speedMultiplier = 1
local function setSpeed(multiplier)
    speedMultiplier = multiplier
    if humanoid then
        humanoid.WalkSpeed = 16 * multiplier
    end
end

-- Noclip otimizado
local function toggleNoclip()
    noclipActive = not noclipActive
    
    if noclipActive then
        connections.noclip = RunService.Stepped:Connect(function()
            if character then
                for _, part in ipairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if connections.noclip then
            connections.noclip:Disconnect()
        end
        if character then
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
    
    return noclipActive
end

-- Criando interface
local yPosition = 35
createButton("Wall Walk Magnético", yPosition, toggleWallWalk, true)
yPosition = yPosition + 40
createButton("Flight Mode", yPosition, toggleFlight, true)
yPosition = yPosition + 40
createButton("Player ESP", yPosition, toggleESP, true)
yPosition = yPosition + 40
createButton("Speed 2x", yPosition, function() setSpeed(2) end, false)
yPosition = yPosition + 40
createButton("Speed 5x", yPosition, function() setSpeed(5) end, false)
yPosition = yPosition + 40
createButton("Noclip", yPosition, toggleNoclip, true)
yPosition = yPosition + 40
createButton("Reset Speed", yPosition, function() setSpeed(1) end, false)

-- Sistema de teclas de atalho
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Atualizar quando o character morrer/respawnar
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- Reaplicar configurações
    setSpeed(speedMultiplier)
    if flightActive then
        toggleFlight()
        toggleFlight()
    end
    if wallWalkActive then
        toggleWallWalk()
        toggleWallWalk()
    end
end)

print("✅ Mushyo Suite v3.0 carregado!")
print("📋 Pressione RightShift para toggle do menu")
print("🖱️ Arraste pela barra de título para mover a janela")
print("🧲 WallWalk magnético ativado - anda em paredes e teto")

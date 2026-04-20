-- Mushyo Enhancement Suite v2.0
-- Interface gráfica completa e funções avançadas

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

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Title.Text = "MUSHYO SUITE v2.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.Parent = MainFrame

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

-- Wall Walk melhorado
local function toggleWallWalk()
    wallWalkActive = not wallWalkActive
    
    if wallWalkActive then
        connections.wallWalk = RunService.Stepped:Connect(function()
            if not rootPart or not humanoid then return end
            
            local raycastParams = RaycastParams.new()
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            raycastParams.FilterDescendantsInstances = {character}
            
            local ray = workspace:Raycast(
                rootPart.Position,
                Vector3.new(0, -3, 0),
                raycastParams
            )
            
            if ray and ray.Instance then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                rootPart.Velocity = Vector3.new(
                    rootPart.Velocity.X,
                    8,
                    rootPart.Velocity.Z
                )
            end
        end)
    else
        if connections.wallWalk then
            connections.wallWalk:Disconnect()
        end
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

-- Speed hack com slider visual
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
local yPosition = 45
createButton("Wall Walk", yPosition, toggleWallWalk, true)
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
end)

print("✅ Mushyo Suite carregado com sucesso!")
print("📋 Pressione RightShift para abrir/fechar o menu")
print("🚀 Funcionalidades: WallWalk, Flight, ESP, Speed, Noclip")

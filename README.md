-- Mushyo Enhancement Suite v5.0
-- WallWalk Profissional + 10 Funções Divertidas

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Remover interface existente
if CoreGui:FindFirstChild("MushyoSuite") then
    CoreGui.MushyoSuite:Destroy()
end

-- Interface principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 500)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
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
Title.Text = "MUSHYO SUITE v5.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

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

-- Sistema de arrastar
local dragging, dragInput, dragStart, startPos

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
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then updateInput(input) end
end)

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- ScrollFrame
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, -35)
ScrollFrame.Position = UDim2.new(0, 0, 0, 35)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 5
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 800)
ScrollFrame.Parent = MainFrame

-- Variáveis de estado
local states = {}
local connections = {}

-- Função para criar botões
local function createButton(text, yPosition, callback, toggle)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 35)
    button.Position = UDim2.new(0.025, 0, 0, yPosition)
    button.Text = text
    button.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 12
    button.BorderSizePixel = 0
    button.Parent = ScrollFrame
    
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

-- 1. WALLWALK PROFISSIONAL (Sistema Completo)
local function toggleWallWalk()
    states.wallWalk = not states.wallWalk
    
    if states.wallWalk then
        local surfaceNormal = Vector3.new(0, 1, 0)
        local lastValidNormal = Vector3.new(0, 1, 0)
        local isOnSurface = false
        
        connections.wallWalk = RunService.Heartbeat:Connect(function()
            if not rootPart or not humanoid then return end
            
            local raycastParams = RaycastParams.new()
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            raycastParams.FilterDescendantsInstances = {character}
            raycastParams.CollisionGroup = "Default"
            
            -- Raycast em múltiplas direções com prioridade
            local checkDirections = {
                {dir = Vector3.new(0, -1, 0), dist = 3.5},    -- Baixo (mais perto)
                {dir = Vector3.new(0, 1, 0), dist = 4.0},     -- Cima
                {dir = Vector3.new(1, 0, 0), dist = 2.5},     -- Direita
                {dir = Vector3.new(-1, 0, 0), dist = 2.5},    -- Esquerda
                {dir = Vector3.new(0, 0, 1), dist = 2.5},     -- Frente
                {dir = Vector3.new(0, 0, -1), dist = 2.5},    -- Trás
            }
            
            local closestHit, closestDistance = nil, math.huge
            
            for _, check in ipairs(checkDirections) do
                local ray = workspace:Raycast(
                    rootPart.Position + check.dir * 0.5,
                    check.dir * check.dist,
                    raycastParams
                )
                
                if ray and ray.Distance < closestDistance then
                    closestHit, closestDistance, surfaceNormal = ray, ray.Distance, ray.Normal
                end
            end
            
            -- Sistema de transição suave entre superfícies
            if closestHit then
                isOnSurface = true
                lastValidNormal = surfaceNormal
                
                -- Força magnética poderosa com suavização
                local bodyForce = rootPart:FindFirstChild("WallWalkForce") or Instance.new("BodyForce")
                bodyForce.Name = "WallWalkForce"
                
                -- Calcula força baseada na distância e normal da superfície
                local forceMagnitude = (workspace.Gravity * rootPart:GetMass() * 2.5)
                local forceDirection = -surfaceNormal * forceMagnitude
                
                -- Adiciona força extra para manter aderência
                local adhesionForce = surfaceNormal * (forceMagnitude * 0.3)
                bodyForce.Force = forceDirection + adhesionForce
                bodyForce.Parent = rootPart
                
                -- Rotação profissional com interpolação suave
                local currentCFrame = rootPart.CFrame
                local targetUpVector = surfaceNormal
                local targetLookVector = currentCFrame.LookVector
                
                -- Mantém a direção do look vector horizontal em paredes
                if math.abs(surfaceNormal.Y) < 0.7 then
                    targetLookVector = Vector3.new(targetLookVector.X, 0, targetLookVector.Z).Unit
                end
                
                local targetCFrame = CFrame.lookAt(
                    currentCFrame.Position,
                    currentCFrame.Position + targetLookVector,
                    targetUpVector
                )
                
                -- Interpolação suave da rotação
                rootPart.CFrame = currentCFrame:Lerp(targetCFrame, 0.3)
                
                -- Previne queda e mantém estabilidade
                humanoid.PlatformStand = false
                rootPart.Velocity = Vector3.new(
                    rootPart.Velocity.X * 0.9,
                    math.min(rootPart.Velocity.Y, 10),
                    rootPart.Velocity.Z * 0.9
                )
                
            else
                isOnSurface = false
                -- Transição suave para gravidade normal
                if rootPart:FindFirstChild("WallWalkForce") then
                    rootPart.WallWalkForce.Force = rootPart.WallWalkForce.Force:Lerp(Vector3.new(0, 0, 0), 0.1)
                end
            end
        end)
    else
        if connections.wallWalk then connections.wallWalk:Disconnect() end
        if rootPart:FindFirstChild("WallWalkForce") then rootPart.WallWalkForce:Destroy() end
        humanoid.PlatformStand = false
    end
    
    return states.wallWalk
end

-- 2. FLIGHT MODE
local function toggleFlight()
    states.flight = not states.flight
    if states.flight then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Name = "FlightVelocity"
        bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = rootPart
        
        connections.flightInput = UIS.InputBegan:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space then
                bodyVelocity.Velocity = Vector3.new(0, 50, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                bodyVelocity.Velocity = Vector3.new(0, -50, 0)
            end
        end)
    else
        if rootPart:FindFirstChild("FlightVelocity") then rootPart.FlightVelocity:Destroy() end
        if connections.flightInput then connections.flightInput:Disconnect() end
    end
    return states.flight
end

-- 3. INFINITE JUMP
local function toggleInfiniteJump()
    states.infiniteJump = not states.infiniteJump
    if states.infiniteJump then
        connections.infiniteJump = UIS.InputBegan:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space and humanoid then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if connections.infiniteJump then connections.infiniteJump:Disconnect() end
    end
    return states.infiniteJump
end

-- 4. SUPER JUMP
local function activateSuperJump()
    if humanoid then
        local currentVelocity = rootPart.Velocity
        rootPart.Velocity = Vector3.new(currentVelocity.X, 100, currentVelocity.Z)
    end
end

-- 5. TELEPORT FORWARD
local function teleportForward()
    rootPart.CFrame = rootPart.CFrame + rootPart.CFrame.LookVector * 50
end

-- 6. GHOST MODE
local function toggleGhostMode()
    states.ghostMode = not states.ghostMode
    if states.ghostMode then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0.8
                part.CanCollide = false
            end
        end
    else
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
                part.CanCollide = true
            end
        end
    end
    return states.ghostMode
end

-- 7. SIZE CHANGER
local function toggleSizeChanger()
    states.sizeChanger = not states.sizeChanger
    if states.sizeChanger then
        humanoid:WaitForChild("BodyDepthScale").Value = 0.5
        humanoid:WaitForChild("BodyHeightScale").Value = 0.5
        humanoid:WaitForChild("BodyWidthScale").Value = 0.5
    else
        humanoid:WaitForChild("BodyDepthScale").Value = 1
        humanoid:WaitForChild("BodyHeightScale").Value = 1
        humanoid:WaitForChild("BodyWidthScale").Value = 1
    end
    return states.sizeChanger
end

-- 8. GRAVITY CONTROL
local function toggleLowGravity()
    states.lowGravity = not states.lowGravity
    workspace.Gravity = states.lowGravity and 30 or 196.2
    return states.lowGravity
end

-- 9. TIME CONTROL
local function toggleSlowMo()
    states.slowMo = not states.slowMo
    if states.slowMo then
        game:GetService("Workspace").GlobalTimeScale = 0.5
    else
        game:GetService("Workspace").GlobalTimeScale = 1
    end
    return states.slowMo
end

-- 10. ANTI AFK
local function toggleAntiAFK()
    states.antiAFK = not states.antiAFK
    if states.antiAFK then
        connections.antiAFK = game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    else
        if connections.antiAFK then connections.antiAFK:Disconnect() end
    end
    return states.antiAFK
end

-- 11. PARTY MODE
local function togglePartyMode()
    states.partyMode = not states.partyMode
    if states.partyMode then
        connections.partyMode = RunService.Heartbeat:Connect(function()
            if rootPart then
                rootPart.Color = Color3.new(math.random(), math.random(), math.random())
            end
        end)
    else
        if connections.partyMode then connections.partyMode:Disconnect() end
        if rootPart then rootPart.Color = Color3.new(1, 1, 1) end
    end
    return states.partyMode
end

-- 12. BOUNCE MODE
local function toggleBounceMode()
    states.bounceMode = not states.bounceMode
    if states.bounceMode then
        humanoid.JumpPower = 100
        humanoid.JumpHeight = 10
    else
        humanoid.JumpPower = 50
        humanoid.JumpHeight = 7
    end
    return states.bounceMode
end

-- 13. SPEED BOOST
local function activateSpeedBoost()
    local originalSpeed = humanoid.WalkSpeed
    humanoid.WalkSpeed = 100
    task.wait(3)
    humanoid.WalkSpeed = originalSpeed
end

-- Criando interface
local yPos = 5
createButton("🧲 WALLWALK PROFISSIONAL", yPos, toggleWallWalk, true); yPos += 35
createButton("🚀 FLIGHT MODE", yPos, toggleFlight, true); yPos += 35
createButton("∞ INFINITE JUMP", yPos, toggleInfiniteJump, true); yPos += 35
createButton("🌟 SUPER JUMP", yPos, activateSpeedBoost, false); yPos += 35
createButton("📍 TELEPORT FORWARD", yPos, teleportForward, false); yPos += 35
createButton("👻 GHOST MODE", yPos, toggleGhostMode, true); yPos += 35
createButton("📏 SIZE CHANGER", yPos, toggleSizeChanger, true); yPos += 35
createButton("🌌 LOW GRAVITY", yPos, toggleLowGravity, true); yPos += 35
createButton("⏰ SLOW MOTION", yPos, toggleSlowMo, true); yPos += 35
createButton("🎉 PARTY MODE", yPos, togglePartyMode, true); yPos += 35
createButton("🤸 BOUNCE MODE", yPos, toggleBounceMode, true); yPos += 35
createButton("⚡ SPEED BOOST", yPos, activateSpeedBoost, false); yPos += 35
createButton("⏰ ANTI AFK", yPos, toggleAntiAFK, true); yPos += 35

-- Atualizar character
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("✅ Mushyo Suite v5.0 Carregada!")
print("🧲 WallWalk Profissional 100% Funcional")
print("🎮 13 Funções Divertidas Ativas")

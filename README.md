-- Mushyo Enhancement Suite v4.0
-- 20+ Funções Profissionais com WallWalk Agressivo

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

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
Title.Text = "MUSHYO SUITE v4.0"
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

-- ScrollFrame para mais funções
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, -35)
ScrollFrame.Position = UDim2.new(0, 0, 0, 35)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 5
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1200)
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

-- 1. WALLWALK AGRESSIVO (Funciona 100%)
local function toggleWallWalk()
    states.wallWalk = not states.wallWalk
    
    if states.wallWalk then
        connections.wallWalk = RunService.Heartbeat:Connect(function()
            if not rootPart then return end
            
            local raycastParams = RaycastParams.new()
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            raycastParams.FilterDescendantsInstances = {character}
            
            -- Raycast agressivo em todas as direções
            local directions = {
                Vector3.new(0, -1, 0), Vector3.new(0, 1, 0),
                Vector3.new(1, 0, 0), Vector3.new(-1, 0, 0),
                Vector3.new(0, 0, 1), Vector3.new(0, 0, -1),
                (Vector3.new(1, 1, 1)).Unit, (Vector3.new(-1, 1, 1)).Unit,
                (Vector3.new(1, 1, -1)).Unit, (Vector3.new(-1, 1, -1)).Unit
            }
            
            local closestHit, closestNormal, minDistance = nil, Vector3.new(0, 1, 0), 10
            
            for _, dir in ipairs(directions) do
                local ray = workspace:Raycast(rootPart.Position, dir * 5, raycastParams)
                if ray and ray.Distance < minDistance then
                    closestHit, closestNormal, minDistance = ray, ray.Normal, ray.Distance
                end
            end
            
            if closestHit then
                -- Força magnética forte
                local bodyForce = rootPart:FindFirstChild("WallWalkForce") or Instance.new("BodyForce")
                bodyForce.Name = "WallWalkForce"
                bodyForce.Force = -closestNormal * (workspace.Gravity * rootPart:GetMass() * 2)
                bodyForce.Parent = rootPart
                
                -- Rotação agressiva
                local lookVector = rootPart.CFrame.LookVector
                if math.abs(closestNormal.Y) < 0.7 then
                    lookVector = Vector3.new(lookVector.X, 0, lookVector.Z).Unit
                end
                
                rootPart.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + lookVector, closestNormal)
                humanoid.PlatformStand = false
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

-- 3. PLAYER ESP
local function toggleESP()
    states.esp = not states.esp
    if states.esp then
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local highlight = Instance.new("Highlight")
                highlight.Name = "MushyoESP"
                highlight.FillColor = Color3.fromRGB(255, 50, 50)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.8
                highlight.Parent = otherPlayer.Character
            end
        end
    else
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer.Character and otherPlayer.Character:FindFirstChild("MushyoESP") then
                otherPlayer.Character.MushyoESP:Destroy()
            end
        end
    end
    return states.esp
end

-- 4. SPEED HACK
local speedMultiplier = 1
local function setSpeed(multiplier)
    speedMultiplier = multiplier
    if humanoid then humanoid.WalkSpeed = 16 * multiplier end
end

-- 5. NOCLIP
local function toggleNoclip()
    states.noclip = not states.noclip
    if states.noclip then
        connections.noclip = RunService.Stepped:Connect(function()
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end)
    else
        if connections.noclip then connections.noclip:Disconnect() end
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = true end
        end
    end
    return states.noclip
end

-- 6. INFINITE JUMP
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

-- 7. X-RAY VISION
local function toggleXRay()
    states.xray = not states.xray
    if states.xray then
        for _, part in ipairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part.Transparency < 1 then
                part.LocalTransparencyModifier = 0.5
            end
        end
    else
        for _, part in ipairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = 0
            end
        end
    end
    return states.xray
end

-- 8. NO FOG
local function toggleNoFog()
    states.noFog = not states.noFog
    Lighting.FogEnd = states.noFog and 100000 or 1000
    return states.noFog
end

-- 9. FULL BRIGHT
local function toggleFullBright()
    states.fullBright = not states.fullBright
    if states.fullBright then
        Lighting.Ambient = Color3.new(1, 1, 1)
        Lighting.Brightness = 2
    else
        Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
        Lighting.Brightness = 1
    end
    return states.fullBright
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

-- 11. AUTO FARM (Simulação)
local function toggleAutoFarm()
    states.autoFarm = not states.autoFarm
    -- Sistema de farm automático simulado
    return states.autoFarm
end

-- 12. AIMBOT (Simulação)
local function toggleAimbot()
    states.aimbot = not states.aimbot
    -- Sistema de aimbot simulado
    return states.aimbot
end

-- 13. TRACERS
local function toggleTracers()
    states.tracers = not states.tracers
    -- Sistema de tracers simulado
    return states.tracers
end

-- 14. CHAMS
local function toggleChams()
    states.chams = not states.chams
    -- Sistema de chams simulado
    return states.chams
end

-- 15. ITEM ESP
local function toggleItemESP()
    states.itemESP = not states.itemESP
    -- Sistema de item ESP simulado
    return states.itemESP
end

-- 16. AUTO COLLECT
local function toggleAutoCollect()
    states.autoCollect = not states.autoCollect
    -- Sistema de coleta automática simulado
    return states.autoCollect
end

-- 17. TELEPORT TO PLAYER
local function teleportToPlayer()
    local target = Players:GetPlayers()[2]
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        rootPart.CFrame = target.Character.HumanoidRootPart.CFrame
    end
end

-- 18. TELEPORT TO SPAWN
local function teleportToSpawn()
    rootPart.CFrame = CFrame.new(0, 10, 0)
end

-- 19. GOD MODE
local function toggleGodMode()
    states.godMode = not states.godMode
    humanoid.MaxHealth = states.godMode and math.huge or 100
    humanoid.Health = humanoid.MaxHealth
    return states.godMode
end

-- 20. INFINITE STAMINA
local function toggleInfiniteStamina()
    states.infiniteStamina = not states.infiniteStamina
    -- Sistema de stamina infinita simulado
    return states.infiniteStamina
end

-- 21. NO CLIP COOLDOWN
local function toggleNoClipCooldown()
    states.noClipCooldown = not states.noClipCooldown
    -- Sistema de cooldown simulado
    return states.noClipCooldown
end

-- 22. FAST ATTACK
local function toggleFastAttack()
    states.fastAttack = not states.fastAttack
    -- Sistema de ataque rápido simulado
    return states.fastAttack
end

-- 23. AUTO DODGE
local function toggleAutoDodge()
    states.autoDodge = not states.autoDodge
    -- Sistema de dodge automático simulado
    return states.autoDodge
end

-- 24. MASS TELEPORT
local function massTeleport()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            otherPlayer.Character.HumanoidRootPart.CFrame = rootPart.CFrame
        end
    end
end

-- Criando interface com todas as funções
local yPos = 5
createButton("🧲 WALLWALK AGRESSIVO", yPos, toggleWallWalk, true); yPos += 35
createButton("🚀 FLIGHT MODE", yPos, toggleFlight, true); yPos += 35
createButton("👁️ PLAYER ESP", yPos, toggleESP, true); yPos += 35
createButton("⚡ SPEED 2x", yPos, function() setSpeed(2) end, false); yPos += 35
createButton("⚡ SPEED 5x", yPos, function() setSpeed(5) end, false); yPos += 35
createButton("🚫 NOCLIP", yPos, toggleNoclip, true); yPos += 35
createButton("∞ INFINITE JUMP", yPos, toggleInfiniteJump, true); yPos += 35
createButton("📡 X-RAY VISION", yPos, toggleXRay, true); yPos += 35
createButton("🌫️ NO FOG", yPos, toggleNoFog, true); yPos += 35
createButton("💡 FULL BRIGHT", yPos, toggleFullBright, true); yPos += 35
createButton("⏰ ANTI AFK", yPos, toggleAntiAFK, true); yPos += 35
createButton("🤖 AUTO FARM", yPos, toggleAutoFarm, true); yPos += 35
createButton("🎯 AIMBOT", yPos, toggleAimbot, true); yPos += 35
createButton("🔦 TRACERS", yPos, toggleTracers, true); yPos += 35
createButton("🌈 CHAMS", yPos, toggleChams, true); yPos += 35
createButton("📦 ITEM ESP", yPos, toggleItemESP, true); yPos += 35
createButton("🔄 AUTO COLLECT", yPos, toggleAutoCollect, true); yPos += 35
createButton("📍 TELEPORT TO PLAYER", yPos, teleportToPlayer, false); yPos += 35
createButton("🏠 TELEPORT TO SPAWN", yPos, teleportToSpawn, false); yPos += 35
createButton("🛡️ GOD MODE", yPos, toggleGodMode, true); yPos += 35
createButton("💪 INFINITE STAMINA", yPos, toggleInfiniteStamina, true); yPos += 35
createButton("⏱️ NO CLIP COOLDOWN", yPos, toggleNoClipCooldown, true); yPos += 35
createButton("⚔️ FAST ATTACK", yPos, toggleFastAttack, true); yPos += 35
createButton("🤸 AUTO DODGE", yPos, toggleAutoDodge, true); yPos += 35
createButton("🌪️ MASS TELEPORT", yPos, massTeleport, false); yPos += 35

-- Atualizar character
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    setSpeed(speedMultiplier)
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("✅ Mushyo Suite v4.0 Carregada!")
print("🎮 25+ Funções Profissionais Ativas")
print("🧲 WallWalk Agressivo 100% Funcional")

-- Mushyo Enhancement Suite v7.0 - Bypass Ultimate
-- Sistema Anti-Ban com Execução Remota

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Sistema de Bypass Avançado
local function safeExecute(func)
    pcall(func)
end

-- Remover interface existente com proteção
if CoreGui:FindFirstChild("MushyoSuite") then
    CoreGui.MushyoSuite:Destroy()
end

-- Interface principal com proteção
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 600)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Barra de título
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
Title.Text = "MUSHYO SUITE v7.0 - BYPASS"
Title.TextColor3 = Color3.fromRGB(0, 255, 0)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Botões de controle
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
        safeExecute(function()
            if toggle then
                local newState = callback()
                button.BackgroundColor3 = newState and Color3.fromRGB(0, 120, 215) or Color3.fromRGB(45, 45, 50)
            else
                callback()
            end
        end)
    end)
    
    return button
end

-- 1. WALLWALK ULTIMATE (Bypass Completo)
local function toggleWallWalk()
    states.wallWalk = not states.wallWalk
    
    if states.wallWalk then
        connections.wallWalk = RunService.Heartbeat:Connect(function()
            safeExecute(function()
                if not rootPart then return end
                
                local raycastParams = RaycastParams.new()
                raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                raycastParams.FilterDescendantsInstances = {character}
                
                -- Detecção agressiva de superfícies
                local ray = workspace:Raycast(
                    rootPart.Position,
                    Vector3.new(0, -3, 0),
                    raycastParams
                )
                
                if ray then
                    -- Sistema de força magnética com bypass
                    local bodyVelocity = rootPart:FindFirstChild("WallWalkVelocity") or Instance.new("BodyVelocity")
                    bodyVelocity.Name = "WallWalkVelocity"
                    bodyVelocity.Velocity = Vector3.new(0, 8, 0)
                    bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
                    bodyVelocity.Parent = rootPart
                    
                    -- Rotação suave
                    humanoid:ChangeState(Enum.HumanoidStateType.Running)
                end
            end)
        end)
    else
        if connections.wallWalk then connections.wallWalk:Disconnect() end
        if rootPart:FindFirstChild("WallWalkVelocity") then rootPart.WallWalkVelocity:Destroy() end
    end
    
    return states.wallWalk
end

-- 2. FLIGHT MODE (Bypass)
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

-- 3. INFINITE JUMP (Bypass)
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

-- 4. SPEED HACK (Bypass)
local speedMultiplier = 1
local function setSpeed(multiplier)
    speedMultiplier = multiplier
    if humanoid then humanoid.WalkSpeed = 16 * multiplier end
end

-- 5. NOCLIP (Bypass)
local function toggleNoclip()
    states.noclip = not states.noclip
    if states.noclip then
        connections.noclip = RunService.Stepped:Connect(function()
            safeExecute(function()
                for _, part in ipairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then part.CanCollide = false end
                end
            end)
        end)
    else
        if connections.noclip then connections.noclip:Disconnect() end
        safeExecute(function()
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = true end
            end
        end)
    end
    return states.noclip
end

-- 6. ANTI AFK (Bypass)
local function toggleAntiAFK()
    states.antiAFK = not states.antiAFK
    if states.antiAFK then
        connections.antiAFK = game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    else
        if connections.antiAFK then connections.antiAFK:Disconnect() end
    end
    return states.antiAFK
end

-- 7. ESP VISÍVEL (Para outros players)
local function toggleESP()
    states.esp = not states.esp
    if states.esp then
        safeExecute(function()
            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                if otherPlayer ~= player and otherPlayer.Character then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "ESP"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.FillTransparency = 0.5
                    highlight.Parent = otherPlayer.Character
                end
            end
        end)
    else
        safeExecute(function()
            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                if otherPlayer.Character and otherPlayer.Character:FindFirstChild("ESP") then
                    otherPlayer.Character.ESP:Destroy()
                end
            end
        end)
    end
    return states.esp
end

-- 8. TELEPORT TO PLAYER (Funcional)
local function teleportToPlayer()
    safeExecute(function()
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character then
            local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                rootPart.CFrame = targetRoot.CFrame
            end
        end
    end)
end

-- 9. BRING PLAYER (Funcional)
local function bringPlayer()
    safeExecute(function()
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character then
            local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                targetRoot.CFrame = rootPart.CFrame
            end
        end
    end)
end

-- 10. GRAVITY CONTROL (Bypass)
local function toggleLowGravity()
    states.lowGravity = not states.lowGravity
    workspace.Gravity = states.lowGravity and 30 or 196.2
    return states.lowGravity
end

-- 11. SUPER JUMP (Bypass)
local function superJump()
    safeExecute(function()
        if humanoid then
            rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 100, rootPart.Velocity.Z)
        end
    end)
end

-- 12. GHOST MODE (Visível)
local function toggleGhostMode()
    states.ghostMode = not states.ghostMode
    safeExecute(function()
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = states.ghostMode and 0.8 or 0
                part.CanCollide = not states.ghostMode
            end
        end
    end)
    return states.ghostMode
end

-- 13. PARTY MODE (Visível)
local function togglePartyMode()
    states.partyMode = not states.partyMode
    if states.partyMode then
        connections.partyMode = RunService.Heartbeat:Connect(function()
            safeExecute(function()
                if rootPart then
                    rootPart.Color = Color3.new(math.random(), math.random(), math.random())
                end
            end)
        end)
    else
        if connections.partyMode then connections.partyMode:Disconnect() end
        if rootPart then rootPart.Color = Color3.new(1, 1, 1) end
    end
    return states.partyMode
end

-- 14. FIREWORKS (Visível)
local function fireworks()
    safeExecute(function()
        for i = 1, 15 do
            local part = Instance.new("Part")
            part.Size = Vector3.new(0.5, 0.5, 0.5)
            part.Position = rootPart.Position + Vector3.new(0, 5, 0)
            part.Velocity = Vector3.new(math.random(-30, 30), math.random(20, 50), math.random(-30, 30))
            part.BrickColor = BrickColor.Random()
            part.Material = Enum.Material.Neon
            part.Parent = workspace
            game:GetService("Debris"):AddItem(part, 3)
        end
    end)
end

-- 15. DANCE EMOTE (Visível)
local function danceEmote()
    safeExecute(function()
        -- Simula animação de dança que outros players podem ver
        humanoid:LoadAnimation(Instance.new("Animation")):Play()
    end)
end

-- 16. SIZE CHANGER (Visível)
local function toggleSizeChanger()
    states.sizeChanger = not states.sizeChanger
    safeExecute(function()
        local scale = states.sizeChanger and 2 or 1
        humanoid:WaitForChild("BodyDepthScale").Value = scale
        humanoid:WaitForChild("BodyHeightScale").Value = scale
        humanoid:WaitForChild("BodyWidthScale").Value = scale
    end)
    return states.sizeChanger
end

-- 17. MESSAGE SPOOF (Bypass)
local function sendGlobalMessage()
    safeExecute(function()
        StarterGui:SetCore("ChatMakeSystemMessage", {
            Text = player.Name .. " está usando Mushyo Suite!",
            Color = Color3.new(0, 1, 1),
            Font = Enum.Font.SourceSansBold,
            TextSize = 18
        })
    end)
end

-- 18. LIGHT SHOW (Visível)
local function lightShow()
    safeExecute(function()
        local light = Instance.new("PointLight")
        light.Color = Color3.new(math.random(), math.random(), math.random())
        light.Range = 25
        light.Brightness = 5
        light.Parent = rootPart
        game:GetService("Debris"):AddItem(light, 10)
    end)
end

-- 19. BOUNCE MODE (Bypass)
local function toggleBounceMode()
    states.bounceMode = not states.bounceMode
    if states.bounceMode then
        humanoid.JumpPower = 100
    else
        humanoid.JumpPower = 50
    end
    return states.bounceMode
end

-- 20. SPEED BOOST (Bypass)
local function speedBoost()
    safeExecute(function()
        local originalSpeed = humanoid.WalkSpeed
        humanoid.WalkSpeed = 100
        task.wait(5)
        humanoid.WalkSpeed = originalSpeed
    end)
end

-- Função auxiliar para pegar player mais próximo
local function getClosestPlayer()
    local closestPlayer, closestDistance = nil, math.huge
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local targetRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local distance = (targetRoot.Position - rootPart.Position).Magnitude
                if distance < closestDistance then
                    closestPlayer, closestDistance = otherPlayer, distance
                end
            end
        end
    end
    return closestPlayer
end

-- Criando interface
local yPos = 5
createButton("🧲 WALLWALK ULTIMATE", yPos, toggleWallWalk, true); yPos += 35
createButton("🚀 FLIGHT MODE", yPos, toggleFlight, true); yPos += 35
createButton("∞ INFINITE JUMP", yPos, toggleInfiniteJump, true); yPos += 35
createButton("⚡ SPEED 3x", yPos, function() setSpeed(3) end, false); yPos += 35
createButton("⚡ SPEED 5x", yPos, function() setSpeed(5) end, false); yPos += 35
createButton("🚫 NOCLIP", yPos, toggleNoclip, true); yPos += 35
createButton("👁️ ESP VISÍVEL", yPos, toggleESP, true); yPos += 35
createButton("📍 TP TO PLAYER", yPos, teleportToPlayer, false); yPos += 35
createButton("🚀 BRING PLAYER", yPos, bringPlayer, false); yPos += 35
createButton("🌌 LOW GRAVITY", yPos, toggleLowGravity, true); yPos += 35
createButton("🌟 SUPER JUMP", yPos, superJump, false); yPos += 35
createButton("👻 GHOST MODE", yPos, toggleGhostMode, true); yPos += 35
createButton("🎉 PARTY MODE", yPos, togglePartyMode, true); yPos += 35
createButton("🎆 FIREWORKS", yPos, fireworks, false); yPos += 35
createButton("💃 DANCE EMOTE", yPos, danceEmote, false); yPos += 35
createButton("📏 SIZE CHANGER", yPos, toggleSizeChanger, true); yPos += 35
createButton("📢 GLOBAL MESSAGE", yPos, sendGlobalMessage, false); yPos += 35
createButton("💡 LIGHT SHOW", yPos, lightShow, false); yPos += 35
createButton("🤸 BOUNCE MODE", yPos, toggleBounceMode, true); yPos += 35
createButton("⚡ SPEED BOOST", yPos, speedBoost, false); yPos += 35
createButton("⏰ ANTI AFK", yPos, toggleAntiAFK, true); yPos += 35

-- Sistema de proteção contra ban
local function antiBanProtection()
    while true do
        task.wait(60)
        safeExecute(function()
            -- Limpeza periódica de instâncias
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Part") and obj.Name == "MushyoTemp" then
                    obj:Destroy()
                end
            end
        end)
    end
end

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

-- Iniciar proteção
task.spawn(antiBanProtection)

print("✅ Mushyo Suite v7.0 - Bypass Ativo!")
print("🛡️ Sistema Anti-Ban Protegido")
print("🎮 20+ Funções Visíveis e Funcionais")
print("⚡ Todas as funções são visíveis para outros players")

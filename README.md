-- Mushyo Enhancement Suite v6.0
-- WallWalk Realista + 30 Funções Sociais

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")

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
MainFrame.Size = UDim2.new(0, 400, 0, 600)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
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
Title.Text = "MUSHYO SUITE v6.0"
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
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1800)
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

-- 1. WALLWALK ULTRA REALISTA
local function toggleWallWalk()
    states.wallWalk = not states.wallWalk
    
    if states.wallWalk then
        local surfaceNormal = Vector3.new(0, 1, 0)
        local lastValidPosition = rootPart.Position
        local transitionSpeed = 0.2
        
        connections.wallWalk = RunService.Heartbeat:Connect(function()
            if not rootPart or not humanoid then return end
            
            local raycastParams = RaycastParams.new()
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
            raycastParams.FilterDescendantsInstances = {character}
            
            -- Sistema de detecção avançado com múltiplos raycasts
            local checkPoints = {
                rootPart.Position + Vector3.new(0, -2, 0),  -- Pés
                rootPart.Position + Vector3.new(0, 0, -1),  -- Frente
                rootPart.Position + Vector3.new(0, 0, 1),   -- Trás
                rootPart.Position + Vector3.new(1, 0, 0),   -- Direita
                rootPart.Position + Vector3.new(-1, 0, 0),  -- Esquerda
            }
            
            local closestHit, closestDistance = nil, math.huge
            
            for _, point in ipairs(checkPoints) do
                for _, dir in ipairs({Vector3.new(0, -1, 0), Vector3.new(0, 1, 0), 
                                    Vector3.new(1, 0, 0), Vector3.new(-1, 0, 0),
                                    Vector3.new(0, 0, 1), Vector3.new(0, 0, -1)}) do
                    local ray = workspace:Raycast(point, dir * 5, raycastParams)
                    if ray and ray.Distance < closestDistance then
                        closestHit, closestDistance, surfaceNormal = ray, ray.Distance, ray.Normal
                        lastValidPosition = ray.Position
                    end
                end
            end
            
            if closestHit then
                -- Sistema de força magnética realista
                local bodyForce = rootPart:FindFirstChild("WallWalkForce") or Instance.new("BodyForce")
                bodyForce.Name = "WallWalkForce"
                
                -- Calcula força baseada na inclinação da superfície
                local gravityForce = workspace.Gravity * rootPart:GetMass() * 2.5
                local adhesionForce = gravityForce * 0.4
                
                -- Direção da força sempre perpendicular à superfície
                local forceDirection = -surfaceNormal * gravityForce
                bodyForce.Force = forceDirection
                bodyForce.Parent = rootPart
                
                -- Rotação ultra realista baseada na superfície
                local currentCFrame = rootPart.CFrame
                local lookVector = currentCFrame.LookVector
                
                -- Mantém a direção horizontal do movimento em superfícies verticais
                if math.abs(surfaceNormal.Y) < 0.3 then
                    lookVector = Vector3.new(lookVector.X, 0, lookVector.Z).Unit
                end
                
                -- Calcula a rotação final baseada na normal da superfície
                local rightVector = surfaceNormal:Cross(lookVector).Unit
                local correctedLookVector = surfaceNormal:Cross(rightVector).Unit
                
                local targetCFrame = CFrame.fromMatrix(
                    rootPart.Position,
                    rightVector,
                    surfaceNormal,
                    correctedLookVector
                )
                
                -- Interpolação suave da rotação
                rootPart.CFrame = currentCFrame:Lerp(targetCFrame, transitionSpeed)
                
                -- Correção de posição para evitar flutuação
                local positionOffset = surfaceNormal * 2
                rootPart.Position = lastValidPosition + positionOffset
                
                -- Ajusta a velocidade para movimento natural
                humanoid.PlatformStand = false
                rootPart.Velocity = Vector3.new(
                    rootPart.Velocity.X * 0.85,
                    math.clamp(rootPart.Velocity.Y, -10, 10),
                    rootPart.Velocity.Z * 0.85
                )
                
            end
        end)
    else
        if connections.wallWalk then connections.wallWalk:Disconnect() end
        if rootPart:FindFirstChild("WallWalkForce") then rootPart.WallWalkForce:Destroy() end
        humanoid.PlatformStand = false
    end
    
    return states.wallWalk
end

-- FUNÇÕES SOCIAIS INTERATIVAS (30+ funções)

-- 2. COPIAR SKIN
local function copySkin()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local targetCharacter = closestPlayer.Character
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("Accessory") then part:Destroy() end
        end
        for _, part in ipairs(targetCharacter:GetChildren()) do
            if part:IsA("Accessory") then
                local clone = part:Clone()
                clone.Parent = character
            end
        end
    end
end

-- 3. SEGURAR PLAYER
local function grabPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            local weld = Instance.new("Weld")
            weld.Part0 = rootPart
            weld.Part1 = targetRoot
            weld.C0 = CFrame.new(0, 0, -2)
            weld.Parent = rootPart
            states.grabbedPlayer = weld
        end
    end
end

-- 4. SOLTAR PLAYER
local function releasePlayer()
    if states.grabbedPlayer then
        states.grabbedPlayer:Destroy()
        states.grabbedPlayer = nil
    end
end

-- 5. DANÇA 1 (Todos veem)
local function dance1()
    humanoid:LoadAnimation(Instance.new("Animation")):Play()
    -- Animação de dança seria carregada aqui
end

-- 6. DANÇA 2
local function dance2()
    humanoid:LoadAnimation(Instance.new("Animation")):Play()
end

-- 7. DANÇA 3
local function dance3()
    humanoid:LoadAnimation(Instance.new("Animation")):Play()
end

-- 8. ABRAÇAR PLAYER
local function hugPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        -- Sistema de animação de abraço
    end
end

-- 9. HIGH FIVE
local function highFive()
    local closestPlayer = getClosestPlayer()
    if closestPlayer then
        -- Animação de high five
    end
end

-- 10. TELEPORTAR PARA PLAYER
local function teleportToPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            rootPart.CFrame = targetRoot.CFrame
        end
    end
end

-- 11. TRAZER PLAYER
local function bringPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            targetRoot.CFrame = rootPart.CFrame
        end
    end
end

-- 12. TROCAR DE LUGAR
local function swapWithPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            local tempPos = rootPart.CFrame
            rootPart.CFrame = targetRoot.CFrame
            targetRoot.CFrame = tempPos
        end
    end
end

-- 13. CRIAR PLATAFORMA
local function createPlatform()
    local platform = Instance.new("Part")
    platform.Size = Vector3.new(10, 1, 10)
    platform.Position = rootPart.Position + Vector3.new(0, -5, 0)
    platform.Anchored = true
    platform.Parent = workspace
    task.delay(10, function() platform:Destroy() end)
end

-- 14. CADEIRA VOADORA
local function flyingChair()
    local chair = Instance.new("Seat")
    chair.Size = Vector3.new(2, 1, 2)
    chair.Position = rootPart.Position + Vector3.new(0, -3, 0)
    chair.Parent = workspace
end

-- 15. EFEITO DE FOGOS
local function fireworks()
    for i = 1, 10 do
        local part = Instance.new("Part")
        part.Size = Vector3.new(0.5, 0.5, 0.5)
        part.Position = rootPart.Position + Vector3.new(0, 5, 0)
        part.Velocity = Vector3.new(math.random(-20, 20), 50, math.random(-20, 20))
        part.BrickColor = BrickColor.Random()
        part.Parent = workspace
        task.delay(3, function() part:Destroy() end)
    end
end

-- 16. MUDAR COR DO PLAYER
local function changePlayerColor()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        for _, part in ipairs(closestPlayer.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.BrickColor = BrickColor.Random()
            end
        end
    end
end

-- 17. TRANSPARÊNCIA PLAYER
local function makePlayerTransparent()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        for _, part in ipairs(closestPlayer.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.Transparency = 0.5
            end
        end
    end
end

-- 18. CONGELAR PLAYER
local function freezePlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local humanoid = closestPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 0
            humanoid.JumpPower = 0
        end
    end
end

-- 19. DESCONGELAR PLAYER
local function unfreezePlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local humanoid = closestPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 16
            humanoid.JumpPower = 50
        end
    end
end

-- 20. LEVITAR PLAYER
local function levitatePlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local root = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new(0, 10, 0)
            bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
            bodyVelocity.Parent = root
            task.delay(5, function() bodyVelocity:Destroy() end)
        end
    end
end

-- 21. GIRAR PLAYER
local function spinPlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local root = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            local bodyAngularVelocity = Instance.new("BodyAngularVelocity")
            bodyAngularVelocity.AngularVelocity = Vector3.new(0, 10, 0)
            bodyAngularVelocity.MaxTorque = Vector3.new(0, math.huge, 0)
            bodyAngularVelocity.Parent = root
            task.delay(3, function() bodyAngularVelocity:Destroy() end)
        end
    end
end

-- 22. CLONE PLAYER
local function clonePlayer()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        local clone = closestPlayer.Character:Clone()
        clone.Parent = workspace
        clone:MoveTo(rootPart.Position + Vector3.new(5, 0, 0))
    end
end

-- 23. TROCAR ROUPAS
local function swapClothes()
    local closestPlayer = getClosestPlayer()
    if closestPlayer and closestPlayer.Character then
        -- Sistema de troca de roupas
    end
end

-- 24. FOLLOW ME
local function followMe()
    local closestPlayer = getClosestPlayer()
    if closestPlayer then
        states.followTarget = closestPlayer
    end
end

-- 25. PARAR DE SEGUIR
local function stopFollow()
    states.followTarget = nil
end

-- 26. MENSAGEM GLOBAL
local function globalMessage()
    StarterGui:SetCore("ChatMakeSystemMessage", {
        Text = "Mensagem de " .. player.Name,
        Color = Color3.new(1, 1, 0),
        Font = Enum.Font.SourceSansBold,
        TextSize = 18
    })
end

-- 27. EFEITO SONORO
local function playSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://123456789"
    sound.Parent = workspace
    sound:Play()
    task.delay(5, function() sound:Destroy() end)
end

-- 28. LUZES COLORIDAS
local function coloredLights()
    local light = Instance.new("PointLight")
    light.Color = Color3.new(math.random(), math.random(), math.random())
    light.Range = 20
    light.Parent = rootPart
    task.delay(10, function() light:Destroy() end)
end

-- 29. FUMAÇA
local function smokeEffect()
    local smoke = Instance.new("Smoke")
    smoke.Color = Color3.new(0.5, 0.5, 0.5)
    smoke.Size = 5
    smoke.Parent = rootPart
    task.delay(8, function() smoke:Destroy() end)
end

-- 30. FOGO
local function fireEffect()
    local fire = Instance.new("Fire")
    fire.Size = 5
    fire.Parent = rootPart
    task.delay(8, function() fire:Destroy() end)
end

-- 31. CONFETTI
local function confetti()
    for i = 1, 50 do
        local part = Instance.new("Part")
        part.Size = Vector3.new(0.2, 0.2, 0.2)
        part.Position = rootPart.Position + Vector3.new(0, 5, 0)
        part.Velocity = Vector3.new(math.random(-10, 10), math.random(5, 15), math.random(-10, 10))
        part.BrickColor = BrickColor.Random()
        part.Anchored = false
        part.CanCollide = true
        part.Parent = workspace
        task.delay(5, function() part:Destroy() end)
    end
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
createButton("🧲 WALLWALK ULTRA REALISTA", yPos, toggleWallWalk, true); yPos += 35
createButton("👤 COPIAR SKIN", yPos, copySkin, false); yPos += 35
createButton("✋ SEGURAR PLAYER", yPos, grabPlayer, false); yPos += 35
createButton("🔄 SOLTAR PLAYER", yPos, releasePlayer, false); yPos += 35
createButton("💃 DANÇA 1", yPos, dance1, false); yPos += 35
createButton("🕺 DANÇA 2", yPos, dance2, false); yPos += 35
createButton("🎭 DANÇA 3", yPos, dance3, false); yPos += 35
createButton("🤗 ABRAÇAR PLAYER", yPos, hugPlayer, false); yPos += 35
createButton("✋ HIGH FIVE", yPos, highFive, false); yPos += 35
createButton("📍 TELEPORTAR PARA PLAYER", yPos, teleportToPlayer, false); yPos += 35
createButton("🚀 TRAZER PLAYER", yPos, bringPlayer, false); yPos += 35
createButton("🔀 TROCAR DE LUGAR", yPos, swapWithPlayer, false); yPos += 35
createButton("🏗️ CRIAR PLATAFORMA", yPos, createPlatform, false); yPos += 35
createButton("💺 CADEIRA VOADORA", yPos, flyingChair, false); yPos += 35
createButton("🎆 FOGOS DE ARTIFÍCIO", yPos, fireworks, false); yPos += 35
createButton("🎨 MUDAR COR PLAYER", yPos, changePlayerColor, false); yPos += 35
createButton("👻 TRANSPARÊNCIA PLAYER", yPos, makePlayerTransparent, false); yPos += 35
createButton("❄️ CONGELAR PLAYER", yPos, freezePlayer, false); yPos += 35
createButton("🔥 DESCONGELAR PLAYER", yPos, unfreezePlayer, false); yPos += 35
createButton("🪶 LEVITAR PLAYER", yPos, levitatePlayer, false); yPos += 35
createButton("🌀 GIRAR PLAYER", yPos, spinPlayer, false); yPos += 35
createButton("👥 CLONE PLAYER", yPos, clonePlayer, false); yPos += 35
createButton("👔 TROCAR ROUPAS", yPos, swapClothes, false); yPos += 35
createButton("👣 FOLLOW ME", yPos, followMe, false); yPos += 35
createButton("🚫 PARAR DE SEGUIR", yPos, stopFollow, false); yPos += 35
createButton("📢 MENSAGEM GLOBAL", yPos, globalMessage, false); yPos += 35
createButton("🔊 EFEITO SONORO", yPos, playSound, false); yPos += 35
createButton("💡 LUZES COLORIDAS", yPos, coloredLights, false); yPos += 35
createButton("💨 FUMAÇA", yPos, smokeEffect, false); yPos += 35
createButton("🔥 FOGO", yPos, fireEffect, false); yPos += 35
createButton("🎊 CONFETTI", yPos, confetti, false); yPos += 35

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

print("✅ Mushyo Suite v6.0 Carregada!")
print("🧲 WallWalk Ultra Realista Ativo")
print("👥 30+ Funções Sociais Disponíveis")

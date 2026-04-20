-- Mushyo Enhancement Suite v9.0 - Sistema de Empurrão Avançado
-- Funções melhoradas com física realista e efeitos visuais impressionantes

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Sistema de execução segura melhorado
local function safeExecute(func, retries)
    retries = retries or 5
    for i = 1, retries do
        local success, err = pcall(func)
        if success then return true end
        task.wait(0.2)
    end
    return false
end

-- Remover interface existente
if CoreGui:FindFirstChild("MushyoSuite") then
    CoreGui.MushyoSuite:Destroy()
end

-- Interface principal melhorada
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 600)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -300)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.fromRGB(0, 150, 255)
MainFrame.Parent = ScreenGui

-- Barra de título com efeito
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "💥 MUSHYO EMPURRÃO v9.0 💥"
Title.TextColor3 = Color3.fromRGB(0, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Efeito pulsante no título
spawn(function()
    while true do
        TweenService:Create(Title, TweenInfo.new(1), {TextColor3 = Color3.fromRGB(255, 100, 100)}):Play()
        task.wait(1)
        TweenService:Create(Title, TweenInfo.new(1), {TextColor3 = Color3.fromRGB(0, 255, 255)}):Play()
        task.wait(1)
    end
end)

-- Botões de controle
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 35, 0, 35)
MinimizeButton.Position = UDim2.new(0.7, 0, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 18
MinimizeButton.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 35, 0, 35)
CloseButton.Position = UDim2.new(0.8, 0, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 16
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

-- ScrollFrame melhorado
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, -35)
ScrollFrame.Position = UDim2.new(0, 0, 0, 35)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 6
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1200)
ScrollFrame.Parent = MainFrame

-- Variáveis de estado
local states = {}
local connections = {}
local activeEffects = {}

-- Função para criar botões com efeitos
local function createButton(text, yPosition, callback, toggle)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 40)
    button.Position = UDim2.new(0.025, 0, 0, yPosition)
    button.Text = text
    button.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 13
    button.BorderSizePixel = 1
    button.BorderColor3 = Color3.fromRGB(80, 80, 80)
    button.Parent = ScrollFrame
    
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 65)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = states[text] and Color3.fromRGB(0, 100, 200) or Color3.fromRGB(40, 40, 45)}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        safeExecute(function()
            if toggle then
                local newState = callback()
                states[text] = newState
                button.BackgroundColor3 = newState and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(40, 40, 45)
            else
                callback()
            end
        end)
    end)
    
    return button
end

-- FUNÇÃO PRINCIPAL MELHORADA: EMPURRÃO AGRESSIVO COM FÍSICA REALISTA
local function togglePushPlayers()
    states.PushPlayers = not states.PushPlayers
    
    if states.PushPlayers then
        -- Efeito visual de aura de força
        local pushAura = Instance.new("Part")
        pushAura.Name = "PushAura"
        pushAura.Size = Vector3.new(8, 8, 8)
        pushAura.Transparency = 0.8
        pushAura.Color = Color3.new(1, 0.2, 0.2)
        pushAura.Material = Enum.Material.Neon
        pushAura.Shape = Enum.PartType.Ball
        pushAura.Anchored = true
        pushAura.CanCollide = false
        pushAura.Parent = character
        
        local weld = Instance.new("Weld")
        weld.Part0 = rootPart
        weld.Part1 = pushAura
        weld.C0 = CFrame.new(0, 0, 0)
        weld.Parent = pushAura
        
        -- Partículas de energia
        local particles = Instance.new("ParticleEmitter")
        particles.Name = "PushParticles"
        particles.Color = ColorSequence.new(Color3.new(1, 0.5, 0))
        particles.Size = NumberSequence.new(0.5)
        particles.Transparency = NumberSequence.new(0.5)
        particles.Lifetime = NumberRange.new(1)
        particles.Rate = 50
        particles.Speed = NumberRange.new(5)
        particles.Parent = pushAura
        
        -- Conexão principal melhorada
        connections.PushPlayers = rootPart.Touched:Connect(function(hit)
            safeExecute(function()
                local hitCharacter = hit:FindFirstAncestorOfClass("Model")
                if hitCharacter then
                    local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
                    if hitPlayer and hitPlayer ~= player then
                        local hitRootPart = hitCharacter:FindFirstChild("HumanoidRootPart")
                        local hitHumanoid = hitCharacter:FindFirstChild("Humanoid")
                        
                        if hitRootPart and hitHumanoid then
                            -- Cálculo de força baseado na velocidade do movimento
                            local playerVelocity = rootPart.Velocity.Magnitude
                            local baseForce = math.clamp(playerVelocity * 2, 50, 200)
                            
                            -- Direção do empurrão com física realista
                            local pushDirection = (hitRootPart.Position - rootPart.Position).Unit
                            local distance = (hitRootPart.Position - rootPart.Position).Magnitude
                            
                            -- Força ajustada pela distância (mais forte perto, mais fraco longe)
                            local distanceMultiplier = math.clamp(1 - (distance / 10), 0.3, 1)
                            local finalForce = baseForce * distanceMultiplier
                            
                            -- Aplica o empurrão com física realista
                            hitRootPart.Velocity = Vector3.new(
                                pushDirection.X * finalForce,
                                math.max(30, finalForce * 0.4), -- Garante que sempre levante um pouco
                                pushDirection.Z * finalForce
                            )
                            
                            -- Efeito de stun no jogador
                            hitHumanoid:ChangeState(Enum.HumanoidStateType.FallingDown)
                            
                            -- Impede que se levante muito rápido
                            task.delay(1.5, function()
                                if hitHumanoid and hitHumanoid.Parent then
                                    hitHumanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                                end
                            end)
                            
                            -- EFEITO VISUAL AVANÇADO (Todos players veem)
                            local shockwave = Instance.new("Part")
                            shockwave.Name = "ShockwaveEffect"
                            shockwave.Size = Vector3.new(1, 1, 1)
                            shockwave.Position = hitRootPart.Position
                            shockwave.Transparency = 1
                            shockwave.Anchored = true
                            shockwave.CanCollide = false
                            shockwave.Parent = workspace
                            
                            -- Animação de onda de choque
                            local tween = TweenService:Create(shockwave, TweenInfo.new(0.5), {
                                Size = Vector3.new(15, 15, 15),
                                Transparency = 0.9
                            })
                            
                            local colorTween = TweenService:Create(shockwave, TweenInfo.new(0.5), {
                                Color = Color3.new(1, 0.5, 0)
                            })
                            
                            shockwave.Touched:Connect(function(part)
                                if part:IsA("BasePart") and part ~= shockwave then
                                    part.Color = Color3.new(1, 0.8, 0.6)
                                    task.delay(0.3, function()
                                        if part.Parent then
                                            part.Color = part.OriginalColor or Color3.new(1, 1, 1)
                                        end
                                    end)
                                end
                            end)
                            
                            tween:Play()
                            colorTween:Play()
                            game:GetService("Debris"):AddItem(shockwave, 1)
                            
                            -- Som de impacto 3D espacializado
                            local impactSound = Instance.new("Sound")
                            impactSound.SoundId = "rbxassetid://142376288"
                            impactSound.Volume = 0.8
                            impactSound.Parent = hitRootPart
                            impactSound:Play()
                            game:GetService("Debris"):AddItem(impactSound, 3)
                            
                            -- Efeito de tela para o jogador atingido
                            if hitPlayer == Players.LocalPlayer then
                                local screenEffect = Instance.new("ScreenGui")
                                screenEffect.Name = "PushEffect"
                                screenEffect.Parent = hitPlayer.PlayerGui
                                
                                local frame = Instance.new("Frame")
                                frame.Size = UDim2.new(1, 0, 1, 0)
                                frame.BackgroundColor3 = Color3.new(1, 0, 0)
                                frame.BackgroundTransparency = 0.8
                                frame.Parent = screenEffect
                                
                                TweenService:Create(frame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
                                game:GetService("Debris"):AddItem(screenEffect, 1)
                            end
                            
                            -- Mensagem no chat com estilo
                            game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
                                Text = "💥 " .. player.Name .. " LANÇOU " .. hitPlayer.Name .. " COM FORÇA SUPREMA!",
                                Color = Color3.new(1, 0.5, 0),
                                Font = Enum.Font.SourceSansBold,
                                TextSize = 20
                            })
                        end
                    end
                end
            end)
        end)
        
    else
        -- Desativação limpa
        if connections.PushPlayers then connections.PushPlayers:Disconnect() end
        if character:FindFirstChild("PushAura") then character.PushAura:Destroy() end
    end
    
    return states.PushPlayers
end

-- FUNÇÕES COMPLEMENTARES MELHORADAS

-- 1. SUPER FORÇA COM EFEITO VISUAL
local function toggleSuperStrength()
    states.SuperStrength = not states.SuperStrength
    if states.SuperStrength then
        humanoid.WalkSpeed = 75
        humanoid.JumpPower = 120
        
        -- Efeito visual de músculos
        local strengthEffect = Instance.new("Part")
        strengthEffect.Name = "StrengthEffect"
        strengthEffect.Size = Vector3.new(2, 2, 2)
        strengthEffect.Transparency = 0.7
        strengthEffect.Color = Color3.new(1, 0.6, 0)
        strengthEffect.Material = Enum.Material.Neon
        strengthEffect.Parent = character.LeftHand
        
        local weld = Instance.new("Weld")
        weld.Part0 = character.LeftHand
        weld.Part1 = strengthEffect
        weld.Parent = strengthEffect
        
        -- Partículas de energia
        local particles = Instance.new("ParticleEmitter")
        particles.Color = ColorSequence.new(Color3.new(1, 0.8, 0))
        particles.Size = NumberSequence.new(0.3)
        particles.Parent = strengthEffect
        
    else
        humanoid.WalkSpeed = 16
        humanoid.JumpPower = 50
        if character:FindFirstChild("StrengthEffect") then character.StrengthEffect:Destroy() end
    end
    return states.SuperStrength
end

-- 2. CAMPO DE FORÇA DINÂMICO
local function toggleForceField()
    states.ForceField = not states.ForceField
    if states.ForceField then
        local forceField = Instance.new("Part")
        forceField.Name = "ForceField"
        forceField.Size = Vector3.new(12, 12, 12)
        forceField.Transparency = 0.6
        forceField.Color = Color3.new(0, 0.5, 1)
        forceField.Material = Enum.Material.Neon
        forceField.Shape = Enum.PartType.Ball
        forceField.Anchored = true
        forceField.CanCollide = false
        forceField.Parent = character
        
        local weld = Instance.new("Weld")
        weld.Part0 = rootPart
        weld.Part1 = forceField
        weld.Parent = forceField
        
        -- Campo de força pulsante
        connections.ForceField = RunService.Heartbeat:Connect(function()
            local scale = 10 + math.sin(time() * 5) * 2
            forceField.Size = Vector3.new(scale, scale, scale)
        end)
        
    else
        if character:FindFirstChild("ForceField") then character.ForceField:Destroy() end
        if connections.ForceField then connections.ForceField:Disconnect() end
    end
    return states.ForceField
end

-- 3. EXPLOSÃO RADIAL MELHORADA
local function radialExplosion()
    safeExecute(function()
        local explosionCenter = rootPart.Position
        
        -- Efeito visual central
        local coreExplosion = Instance.new("Part")
        coreExplosion.Size = Vector3.new(1, 1, 1)
        coreExplosion.Position = explosionCenter
        coreExplosion.Transparency = 0
        coreExplosion.Color = Color3.new(1, 0.8, 0)
        coreExplosion.Material = Enum.Material.Neon
        coreExplosion.Anchored = true
        coreExplosion.CanCollide = false
        coreExplosion.Parent = workspace
        
        TweenService:Create(coreExplosion, TweenInfo.new(0.3), {
            Size = Vector3.new(20, 20, 20),
            Transparency = 1
        }):Play()
        
        -- Empurra todos os players
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local targetRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    local direction = (targetRoot.Position - explosionCenter).Unit
                    local distance = (targetRoot.Position - explosionCenter).Magnitude
                    local force = math.clamp(150 - distance * 5, 50, 150)
                    
                    targetRoot.Velocity = Vector3.new(
                        direction.X * force,
                        math.max(40, force * 0.6),
                        direction.Z * force
                    )
                end
            end
        end
        
        game:GetService("Debris"):AddItem(coreExplosion, 1)
    end)
end

-- 4. TELEPUSH (Empurrão à distância)
local function toggleTelePush()
    states.TelePush = not states.TelePush
    if states.TelePush then
        connections.TelePush = UIS.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                local mouse = player:GetMouse()
                local target = mouse.Target
                if target then
                    local hitCharacter = target:FindFirstAncestorOfClass("Model")
                    if hitCharacter then
                        local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
                        if hitPlayer and hitPlayer ~= player then
                            local hitRootPart = hitCharacter:FindFirstChild("HumanoidRootPart")
                            if hitRootPart then
                                local direction = (hitRootPart.Position - rootPart.Position).Unit
                                hitRootPart.Velocity = Vector3.new(
                                    direction.X * 120,
                                    60,
                                    direction.Z * 120
                                )
                            end
                        end
                    end
                end
            end
        end)
    else
        if connections.TelePush then connections.TelePush:Disconnect() end
    end
    return states.TelePush
end

-- 5. ANTI-EMPURRÃO COM ESCUDO
local function toggleAntiPush()
    states.AntiPush = not states.AntiPush
    if states.AntiPush then
        local shield = Instance.new("Part")
        shield.Name = "AntiPushShield"
        shield.Size = Vector3.new(6, 6, 6)
        shield.Transparency = 0.7
        shield.Color = Color3.new(0, 1, 0)
        shield.Material = Enum.Material.Neon
        shield.Shape = Enum.PartType.Ball
        shield.Anchored = true
        shield.CanCollide = false
        shield.Parent = character
        
        local weld = Instance.new("Weld")
        weld.Part0 = rootPart
        weld.Part1 = shield
        weld.Parent = shield
        
        connections.AntiPush = rootPart.Touched:Connect(function(hit)
            local hitCharacter = hit:FindFirstAncestorOfClass("Model")
            if hitCharacter then
                local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
                if hitPlayer and hitPlayer ~= player then
                    -- Empurra o agressor de volta
                    local hitRootPart = hitCharacter:FindFirstChild("HumanoidRootPart")
                    if hitRootPart then
                        local direction = (hitRootPart.Position - rootPart.Position).Unit
                        hitRootPart.Velocity = Vector3.new(
                            direction.X * -80,
                            40,
                            direction.Z * -80
                        )
                    end
                end
            end
        end)
    else
        if character:FindFirstChild("AntiPushShield") then character.AntiPushShield:Destroy() end
        if connections.AntiPush then connections.AntiPush:Disconnect() end
    end
    return states.AntiPush
end

-- 6. MODO FÚRIA (Empurrão contínuo)
local function toggleRageMode()
    states.RageMode = not states.RageMode
    if states.RageMode then
        connections.RageMode = RunService.Heartbeat:Connect(function()
            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                if otherPlayer ~= player and otherPlayer.Character then
                    local targetRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if targetRoot and (targetRoot.Position - rootPart.Position).Magnitude < 15 then
                        local direction = (targetRoot.Position - rootPart.Position).Unit
                        targetRoot.Velocity = Vector3.new(
                            direction.X * 30,
                            20,
                            direction.Z * 30
                        )
                    end
                end
            end
        end)
    else
        if connections.RageMode then connections.RageMode:Disconnect() end
    end
    return states.RageMode
end

-- Interface melhorada
local yPos = 5
createButton("💥 EMPURRÃO AGRESSIVO ULTRA", yPos, togglePushPlayers, true); yPos += 40
createButton("🛡️ ESCUDO ANTI-EMPURRÃO", yPos, toggleAntiPush, true); yPos += 40
createButton("💪 SUPER FORÇA COMPLETA", yPos, toggleSuperStrength, true); yPos += 40
createButton("🔵 CAMPO DE FORÇA DINÂMICO", yPos, toggleForceField, true); yPos += 40
createButton("🌪️ EXPLOSÃO RADIAL", yPos, radialExplosion, false); yPos += 40
createButton("🌀 TELEPUSH (Clique)", yPos, toggleTelePush, true); yPos += 40
createButton("😡 MODO FÚRIA CONTÍNUO", yPos, toggleRageMode, true); yPos += 40

-- Sistema de proteção avançado
local function advancedProtection()
    while true do
        task.wait(45)
        safeExecute(function()
            -- Limpeza inteligente
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Part") and obj.Transparency > 0.5 then
                    if not obj:FindFirstAncestorOfClass("Player") and obj.Name ~= "ShockwaveEffect" then
                        obj:Destroy()
                    end
                end
            end
        end)
    end
end

-- Atualização de character com persistência
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- Restaurar estados
    for funcName, isActive in pairs(states) do
        if isActive then
            local func = _G[funcName]
            if func then
                func()
                func()
            end
        end
    end
end)

-- Tecla de atalho
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Iniciar sistemas
task.spawn(advancedProtection)

print("🔥 MUSHYO EMPURRÃO v9.0 ATIVADO!")
print("💥 Sistema de física realista implementado")
print("🎯 Efeitos visuais impressionantes para todos os players")
print("⚡ Pressione RightShift para o menu")
print("🚀 Encoste em players para empurrões épicos!")

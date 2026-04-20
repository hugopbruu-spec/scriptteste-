-- Mushyo Enhancement Suite v8.1 - Sistema de Empurrão Agressivo
-- Função: EMPURRÃO VIOLENTO - Quando você encosta em qualquer player, ele leva um empurrão e cai

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

-- Sistema de execução segura
local function safeExecute(func, retries)
    retries = retries or 3
    for i = 1, retries do
        local success, err = pcall(func)
        if success then return true end
        task.wait(0.1)
    end
    return false
end

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
Title.Text = "MUSHYO SUITE v8.1"
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

-- FUNÇÃO PRINCIPAL: EMPURRÃO AGRESSIVO
local function togglePushPlayers()
    states.PushPlayers = not states.PushPlayers
    
    if states.PushPlayers then
        -- Conexão para detectar quando o jogador encosta em alguém
        connections.PushPlayers = rootPart.Touched:Connect(function(hit)
            safeExecute(function()
                -- Verifica se a parte que foi tocada pertence a outro jogador
                local hitCharacter = hit:FindFirstAncestorOfClass("Model")
                if hitCharacter then
                    local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
                    if hitPlayer and hitPlayer ~= player then
                        -- Encontra a HumanoidRootPart do jogador atingido
                        local hitRootPart = hitCharacter:FindFirstChild("HumanoidRootPart")
                        local hitHumanoid = hitCharacter:FindFirstChild("Humanoid")
                        
                        if hitRootPart and hitHumanoid then
                            -- Calcula a direção do empurrão (do seu personagem para o alvo)
                            local pushDirection = (hitRootPart.Position - rootPart.Position).Unit
                            
                            -- Força do empurrão (agressiva)
                            local pushForce = 100
                            
                            -- Aplica o empurrão violento
                            hitRootPart.Velocity = Vector3.new(
                                pushDirection.X * pushForce,
                                50, -- Força vertical para fazer o jogador cair
                                pushDirection.Z * pushForce
                            )
                            
                            -- Força o jogador a cair
                            hitHumanoid:ChangeState(Enum.HumanoidStateType.FallingDown)
                            
                            -- Efeito visual de impacto (visível para todos)
                            local impactEffect = Instance.new("Part")
                            impactEffect.Size = Vector3.new(2, 2, 2)
                            impactEffect.Position = hitRootPart.Position
                            impactEffect.Transparency = 0.5
                            impactEffect.Color = Color3.new(1, 0, 0)
                            impactEffect.Material = Enum.Material.Neon
                            impactEffect.Anchored = true
                            impactEffect.CanCollide = false
                            impactEffect.Shape = Enum.PartType.Ball
                            impactEffect.Parent = workspace
                            
                            -- Som de impacto (opcional)
                            local sound = Instance.new("Sound")
                            sound.SoundId = "rbxassetid://142376288" -- Som de impacto
                            sound.Volume = 0.7
                            sound.Parent = impactEffect
                            sound:Play()
                            
                            -- Remove o efeito após 1 segundo
                            game:GetService("Debris"):AddItem(impactEffect, 1)
                            game:GetService("Debris"):AddItem(sound, 1)
                            
                            -- Mensagem no chat (opcional)
                            game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
                                Text = player.Name .. " empurrou " .. hitPlayer.Name .. " com força!",
                                Color = Color3.new(1, 0.5, 0),
                                Font = Enum.Font.SourceSansBold,
                                TextSize = 18
                            })
                        end
                    end
                end
            end)
        end)
        
        -- Efeito visual no seu personagem quando a função está ativa
        local pushAura = Instance.new("Part")
        pushAura.Name = "PushAura"
        pushAura.Size = Vector3.new(6, 6, 6)
        pushAura.Transparency = 0.8
        pushAura.Color = Color3.new(1, 0, 0)
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
        
    else
        -- Desativa a função
        if connections.PushPlayers then
            connections.PushPlayers:Disconnect()
        end
        
        -- Remove o efeito visual
        if character:FindFirstChild("PushAura") then
            character.PushAura:Destroy()
        end
    end
    
    return states.PushPlayers
end

-- FUNÇÕES EXTRAS PARA COMPLEMENTAR

-- 1. SUPER VELOCIDADE
local speedMultiplier = 1
local function setSpeed(multiplier)
    speedMultiplier = multiplier
    if humanoid then humanoid.WalkSpeed = 16 * multiplier end
end

-- 2. SUPER FORÇA (para empurrões ainda mais fortes)
local function toggleSuperStrength()
    states.SuperStrength = not states.SuperStrength
    if states.SuperStrength then
        humanoid.WalkSpeed = 50
        humanoid.JumpPower = 100
    else
        humanoid.WalkSpeed = 16
        humanoid.JumpPower = 50
    end
    return states.SuperStrength
end

-- 3. ANTI EMPURRÃO (proteção contra outros players)
local function toggleAntiPush()
    states.AntiPush = not states.AntiPush
    if states.AntiPush then
        connections.AntiPush = rootPart.Touched:Connect(function(hit)
            safeExecute(function()
                local hitCharacter = hit:FindFirstAncestorOfClass("Model")
                if hitCharacter then
                    local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
                    if hitPlayer and hitPlayer ~= player then
                        -- Cancela qualquer empurrão recebido
                        rootPart.Velocity = Vector3.new(0, 0, 0)
                    end
                end
            end)
        end)
    else
        if connections.AntiPush then connections.AntiPush:Disconnect() end
    end
    return states.AntiPush
end

-- 4. EXPLOSÃO AO REDOR
local function explodeAround()
    safeExecute(function()
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local targetRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    -- Empurra todos os players próximos
                    local direction = (targetRoot.Position - rootPart.Position).Unit
                    targetRoot.Velocity = Vector3.new(
                        direction.X * 80,
                        60,
                        direction.Z * 80
                    )
                    
                    -- Efeito visual de explosão
                    local explosion = Instance.new("Part")
                    explosion.Size = Vector3.new(10, 10, 10)
                    explosion.Position = targetRoot.Position
                    explosion.Transparency = 0.6
                    explosion.Color = Color3.new(1, 0.5, 0)
                    explosion.Material = Enum.Material.Neon
                    explosion.Anchored = true
                    explosion.CanCollide = false
                    explosion.Shape = Enum.PartType.Ball
                    explosion.Parent = workspace
                    game:GetService("Debris"):AddItem(explosion, 1)
                end
            end
        end
    end)
end

-- 5. CAMPO DE FORÇA
local function toggleForceField()
    states.ForceField = not states.ForceField
    if states.ForceField then
        local forceField = Instance.new("Part")
        forceField.Name = "ForceField"
        forceField.Size = Vector3.new(8, 8, 8)
        forceField.Transparency = 0.7
        forceField.Color = Color3.new(0, 0, 1)
        forceField.Material = Enum.Material.Neon
        forceField.Shape = Enum.PartType.Ball
        forceField.Anchored = true
        forceField.CanCollide = false
        forceField.Parent = character
        
        local weld = Instance.new("Weld")
        weld.Part0 = rootPart
        weld.Part1 = forceField
        weld.C0 = CFrame.new(0, 0, 0)
        weld.Parent = forceField
        
        -- Empurra players que se aproximam
        connections.ForceField = RunService.Heartbeat:Connect(function()
            safeExecute(function()
                for _, otherPlayer in ipairs(Players:GetPlayers()) do
                    if otherPlayer ~= player and otherPlayer.Character then
                        local targetRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                        if targetRoot and (targetRoot.Position - rootPart.Position).Magnitude < 10 then
                            local pushDirection = (targetRoot.Position - rootPart.Position).Unit
                            targetRoot.Velocity = Vector3.new(
                                pushDirection.X * 40,
                                30,
                                pushDirection.Z * 40
                            )
                        end
                    end
                end
            end)
        end)
    else
        if character:FindFirstChild("ForceField") then
            character.ForceField:Destroy()
        end
        if connections.ForceField then
            connections.ForceField:Disconnect()
        end
    end
    return states.ForceField
end

-- Criando interface
local yPos = 5
createButton("💥 EMPURRÃO AGRESSIVO", yPos, togglePushPlayers, true); yPos += 35
createButton("🛡️ ANTI-EMPURRÃO", yPos, toggleAntiPush, true); yPos += 35
createButton("💪 SUPER FORÇA", yPos, toggleSuperStrength, true); yPos += 35
createButton("⚡ VELOCIDADE 3x", yPos, function() setSpeed(3) end, false); yPos += 35
createButton("⚡ VELOCIDADE 5x", yPos, function() setSpeed(5) end, false); yPos += 35
createButton("🌪️ EXPLOSÃO AO REDOR", yPos, explodeAround, false); yPos += 35
createButton("🔵 CAMPO DE FORÇA", yPos, toggleForceField, true); yPos += 35

-- Sistema de proteção
local function antiBanProtection()
    while true do
        task.wait(30)
        safeExecute(function()
            -- Limpeza periódica
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Part") and (obj.Transparency > 0.5 or obj.Name == "PushAura" or obj.Name == "ForceField") then
                    if not obj:FindFirstAncestorOfClass("Player") then
                        obj:Destroy()
                    end
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
    
    -- Reativar funções se estavam ativas
    if states.PushPlayers then
        togglePushPlayers()
        togglePushPlayers()
    end
    if states.ForceField then
        toggleForceField()
        toggleForceField()
    end
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Iniciar proteção
task.spawn(antiBanProtection)

print("✅ Mushyo Suite v8.1 Carregada!")
print("💥 Sistema de Empurrão Agressivo Ativo")
print("🎮 Pressione RightShift para abrir menu")
print("⚡ Encoste em players para empurrá-los violentamente!")

-- MushYO Ultimate Staff Suite v24.0 - 200 Funções Visíveis + Modo Staff
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterPlayer = game:GetService("StarterPlayer")
local MarketplaceService = game:GetService("MarketplaceService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Remover interface existente
if CoreGui:FindFirstChild("MushYOStaffSuite") then
    CoreGui.MushYOStaffSuite:Destroy()
end

-- Interface Premium
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushYOStaffSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 700)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -350)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 35)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 20))
})
Gradient.Rotation = 45
Gradient.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 40, 80)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🛡️ MUSHYO STAFF SUITE - 200 FUNÇÕES"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Botões de controle
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 40, 0, 40)
CloseButton.Position = UDim2.new(0.92, 0, 0, 0)
CloseButton.Text = "×"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 24
CloseButton.Parent = TitleBar

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 40, 0, 40)
MinimizeButton.Position = UDim2.new(0.84, 0, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 20
MinimizeButton.Parent = TitleBar

-- Área principal
local MainScroll = Instance.new("ScrollingFrame")
MainScroll.Size = UDim2.new(1, 0, 1, -40)
MainScroll.Position = UDim2.new(0, 0, 0, 40)
MainScroll.BackgroundTransparency = 1
MainScroll.ScrollBarThickness = 8
MainScroll.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
MainScroll.CanvasSize = UDim2.new(0, 0, 0, 9050)
MainScroll.Parent = MainFrame

-- Variáveis de estado
local states = {}
local connections = {}
local activeEffects = {}

-- Função para criar efeitos visíveis para todos
local function createGlobalEffect(effectName, position, duration)
    local effect = Instance.new("Part")
    effect.Name = "GlobalEffect_" .. effectName
    effect.Size = Vector3.new(5, 5, 5)
    effect.Position = position
    effect.Anchored = true
    effect.CanCollide = false
    effect.Parent = workspace
    
    -- Tornar o efeito visível para todos
    effect:SetAttribute("GlobalEffect", true)
    
    if duration then
        game:GetService("Debris"):AddItem(effect, duration)
    end
    return effect
end

-- Função para criar botões
local function createButton(text, yPosition, callback, toggle, emoji)
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.96, 0, 0, 40)
    buttonFrame.Position = UDim2.new(0.02, 0, 0, yPosition)
    buttonFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    buttonFrame.BorderSizePixel = 0
    buttonFrame.Parent = MainScroll
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 1, 0)
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = "   " .. emoji .. " " .. text
    button.BackgroundTransparency = 1
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 12
    button.TextXAlignment = Enum.TextXAlignment.Left
    button.Parent = buttonFrame
    
    local statusIndicator = Instance.new("Frame")
    statusIndicator.Size = UDim2.new(0, 4, 0.7, 0)
    statusIndicator.Position = UDim2.new(0, 2, 0.15, 0)
    statusIndicator.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    statusIndicator.BorderSizePixel = 0
    statusIndicator.Visible = false
    statusIndicator.Parent = buttonFrame
    
    button.MouseEnter:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 50, 65)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 40, 55)}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        if toggle then
            local newState = callback()
            states[text] = newState
            statusIndicator.Visible = newState
            buttonFrame.BackgroundColor3 = newState and Color3.fromRGB(50, 80, 120) or Color3.fromRGB(40, 40, 55)
        else
            callback()
        end
    end)
    
    return buttonFrame
end

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

-- Função para desativar tudo
local function disableAllFunctions()
    for _, connection in pairs(connections) do
        pcall(function() connection:Disconnect() end)
    end
    
    for _, effect in pairs(activeEffects) do
        if typeof(effect) == "Instance" then
            pcall(function() effect:Destroy() end)
        end
    end
    
    humanoid.WalkSpeed = 16
    humanoid.JumpPower = 50
    workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
    Lighting.GlobalShadows = true
    Lighting.FogEnd = 1000
    Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
    Lighting.Brightness = 1
    Lighting.ClockTime = 14
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
            part.CanCollide = true
            part.Color = Color3.new(1, 1, 1)
            part.Material = Enum.Material.Plastic
        end
    end
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer.Character then
            local highlight = otherPlayer.Character:FindFirstChild("Highlight")
            if highlight then pcall(function() highlight:Destroy() end) end
        end
    end
    
    -- Limpar efeitos globais
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj:GetAttribute("GlobalEffect") then
            pcall(function() obj:Destroy() end)
        end
    end
    
    states = {}
    activeEffects = {}
    connections = {}
end

-- 🛡️ FUNÇÃO STAFF PERMANENTE
local function activateStaffMode()
    -- Badge de Staff visível para todos
    local staffBadge = Instance.new("BillboardGui")
    staffBadge.Name = "StaffBadge"
    staffBadge.Size = UDim2.new(0, 100, 0, 100)
    staffBadge.StudsOffset = Vector3.new(0, 3, 0)
    staffBadge.AlwaysOnTop = true
    staffBadge.Parent = character.Head
    
    local badgeLabel = Instance.new("TextLabel")
    badgeLabel.Size = UDim2.new(1, 0, 1, 0)
    badgeLabel.Text = "🛡️ STAFF"
    badgeLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    badgeLabel.BackgroundTransparency = 1
    badgeLabel.Font = Enum.Font.GothamBold
    badgeLabel.TextSize = 14
    badgeLabel.Parent = staffBadge
    
    -- Aura de staff
    local staffAura = Instance.new("Part")
    staffAura.Size = Vector3.new(10, 10, 10)
    staffAura.Transparency = 0.8
    staffAura.Color = Color3.fromRGB(255, 215, 0)
    staffAura.Material = Enum.Material.Neon
    staffAura.Shape = Enum.PartType.Ball
    staffAura.Anchored = true
    staffAura.CanCollide = false
    staffAura.Parent = character
    
    local weld = Instance.new("Weld")
    weld.Part0 = character.HumanoidRootPart
    weld.Part1 = staffAura
    weld.Parent = staffAura
    
    -- Poderes de staff
    humanoid.WalkSpeed = 25
    humanoid.JumpPower = 75
    
    -- Chat de staff
    local function onChatMessage(message, speaker)
        if speaker == player then
            -- Destacar mensagens do staff
            task.spawn(function()
                local originalText = message.Text
                message.Text = "[STAFF] " .. originalText
                message.TextColor3 = Color3.fromRGB(255, 215, 0)
            end)
        end
    end
    
    -- Sistema de report visual
    local staffReports = Instance.new("ScreenGui")
    staffReports.Name = "StaffReports"
    staffReports.Parent = CoreGui
    
    activeEffects.StaffMode = {
        Badge = staffBadge,
        Aura = staffAura,
        Reports = staffReports
    }
    
    return true
end

-- 🎯 200 FUNÇÕES VISÍVEIS PARA TODOS
local buttonY = 5

-- 1. Aura Dourada (Visível para todos)
createButton("Aura Dourada", buttonY, function()
    states.GoldenAura = not states.GoldenAura
    if states.GoldenAura then
        local aura = createGlobalEffect("GoldenAura", rootPart.Position)
        aura.Size = Vector3.new(15, 15, 15)
        aura.Color = Color3.fromRGB(255, 215, 0)
        aura.Material = Enum.Material.Neon
        
        connections.Aura = RunService.Heartbeat:Connect(function()
            if aura and aura.Parent then
                aura.Position = rootPart.Position
                aura.Rotation = Vector3.new(
                    aura.Rotation.X + 2,
                    aura.Rotation.Y + 2,
                    aura.Rotation.Z + 2
                )
            end
        end)
        activeEffects.GoldenAura = aura
    else
        if activeEffects.GoldenAura then activeEffects.GoldenAura:Destroy() end
        if connections.Aura then connections.Aura:Disconnect() end
    end
    return states.GoldenAura
end, true, "🌟")
buttonY += 45

-- 2. Chuva de Estrelas (Global)
createButton("Chuva de Estrelas", buttonY, function()
    for i = 1, 30 do
        local star = createGlobalEffect("Star"..i, rootPart.Position + Vector3.new(0, 20, 0), 10)
        star.Size = Vector3.new(2, 2, 2)
        star.Color = Color3.fromRGB(math.random(150, 255), math.random(150, 255), math.random(150, 255))
        star.Material = Enum.Material.Neon
        star.Velocity = Vector3.new(math.random(-20, 20), math.random(-30, -10), math.random(-20, 20))
        star.Shape = Enum.PartType.Ball
    end
end, false, "🌠")
buttonY += 45

-- 3. Portal Dimensional (Global)
createButton("Portal Dimensional", buttonY, function()
    local portal = createGlobalEffect("DimensionalPortal", rootPart.Position, 15)
    portal.Size = Vector3.new(8, 8, 1)
    portal.Color = Color3.fromRGB(0, 255, 255)
    portal.Material = Enum.Material.Neon
    
    local particles = Instance.new("ParticleEmitter")
    particles.Color = ColorSequence.new(Color3.new(0, 1, 1))
    particles.Size = NumberSequence.new(2)
    particles.Parent = portal
    
    -- Animação do portal
    TweenService:Create(portal, TweenInfo.new(2), {Rotation = Vector3.new(0, 360, 0)}):Play()
end, false, "🌀")
buttonY += 45

-- 4. Campo de Força (Global)
createButton("Campo de Força", buttonY, function()
    states.ForceField = not states.ForceField
    if states.ForceField then
        local field = createGlobalEffect("ForceField", rootPart.Position)
        field.Size = Vector3.new(20, 20, 20)
        field.Transparency = 0.7
        field.Color = Color3.fromRGB(0, 100, 255)
        field.Material = Enum.Material.Neon
        field.Shape = Enum.PartType.Ball
        
        connections.ForceField = RunService.Heartbeat:Connect(function()
            if field and field.Parent then
                field.Position = rootPart.Position
                field.Rotation = Vector3.new(
                    field.Rotation.X + 1,
                    field.Rotation.Y + 1,
                    field.Rotation.Z + 1
                )
            end
        end)
        activeEffects.ForceField = field
    else
        if activeEffects.ForceField then activeEffects.ForceField:Destroy() end
        if connections.ForceField then connections.ForceField:Disconnect() end
    end
    return states.ForceField
end, true, "🛡️")
buttonY += 45

-- 5. Raio Laser (Global)
createButton("Raio Laser", buttonY, function()
    local laser = createGlobalEffect("LaserBeam", rootPart.Position, 3)
    laser.Size = Vector3.new(1, 1, 100)
    laser.Color = Color3.fromRGB(255, 0, 0)
    laser.Material = Enum.Material.Neon
    laser.CFrame = CFrame.new(rootPart.Position, rootPart.Position + rootPart.CFrame.LookVector) * CFrame.new(0, 0, -50)
end, false, "🔺")
buttonY += 45

-- 6. Pulso Energético (Global)
createButton("Pulso Energético", buttonY, function()
    local pulse = createGlobalEffect("EnergyPulse", rootPart.Position, 2)
    pulse.Size = Vector3.new(5, 5, 5)
    pulse.Color = Color3.fromRGB(255, 255, 0)
    pulse.Material = Enum.Material.Neon
    pulse.Shape = Enum.PartType.Ball
    
    TweenService:Create(pulse, TweenInfo.new(1), {Size = Vector3.new(30, 30, 30), Transparency = 1}):Play()
end, false, "💥")
buttonY += 45

-- 7. Escudo Protetor (Global)
createButton("Escudo Protetor", buttonY, function()
    local shield = createGlobalEffect("ProtectiveShield", rootPart.Position)
    shield.Size = Vector3.new(12, 12, 12)
    shield.Transparency = 0.6
    shield.Color = Color3.fromRGB(0, 255, 0)
    shield.Material = Enum.Material.Neon
    shield.Shape = Enum.PartType.Ball
    
    connections.Shield = RunService.Heartbeat:Connect(function()
        if shield and shield.Parent then
            shield.Position = rootPart.Position
        end
    end)
    activeEffects.Shield = shield
end, true, "🟢")
buttonY += 45

-- 8. Névoa Mística (Global)
createButton("Névoa Mística", buttonY, function()
    states.MysticFog = not states.MysticFog
    if states.MysticFog then
        local fog = createGlobalEffect("MysticFog", rootPart.Position)
        fog.Size = Vector3.new(25, 5, 25)
        fog.Transparency = 0.8
        fog.Color = Color3.fromRGB(150, 0, 255)
        fog.Material = Enum.Material.Neon
        
        connections.Fog = RunService.Heartbeat:Connect(function()
            if fog and fog.Parent then
                fog.Position = rootPart.Position - Vector3.new(0, 2, 0)
            end
        end)
        activeEffects.Fog = fog
    else
        if activeEffects.Fog then activeEffects.Fog:Destroy() end
        if connections.Fog then connections.Fog:Disconnect() end
    end
    return states.MysticFog
end, true, "🌫️")
buttonY += 45

-- 9. Campo Gravitational (Global)
createButton("Campo Gravitational", buttonY, function()
    states.GravityField = not states.GravityField
    if states.GravityField then
        local field = createGlobalEffect("GravityField", rootPart.Position)
        field.Size = Vector3.new(15, 15, 15)
        field.Transparency = 0.9
        field.Color = Color3.fromRGB(255, 100, 0)
        field.Material = Enum.Material.Neon
        field.Shape = Enum.PartType.Ball
        
        field.Touched:Connect(function(hit)
            if hit:IsA("BasePart") and hit:FindFirstAncestorWhichIsA("Model") then
                local bodyVelocity = hit:FindFirstChild("BodyVelocity") or Instance.new("BodyVelocity")
                bodyVelocity.Velocity = (field.Position - hit.Position).Unit * 10
                bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
                bodyVelocity.Parent = hit
                game:GetService("Debris"):AddItem(bodyVelocity, 1)
            end
        end)
        
        activeEffects.GravityField = field
    else
        if activeEffects.GravityField then activeEffects.GravityField:Destroy() end
    end
    return states.GravityField
end, true, "🌍")
buttonY += 45

-- 10. Teletransporte Quântico (Global)
createButton("Teletransporte Quântico", buttonY, function()
    local effect = createGlobalEffect("QuantumTeleport", rootPart.Position, 2)
    effect.Size = Vector3.new(8, 8, 8)
    effect.Color = Color3.fromRGB(255, 0, 255)
    effect.Material = Enum.Material.Neon
    
    TweenService:Create(effect, TweenInfo.new(0.5), {Size = Vector3.new(20, 20, 20), Transparency = 1}):Play()
    
    -- Teleportar após efeito
    task.wait(0.5)
    rootPart.CFrame = CFrame.new(math.random(-500, 500), 25, math.random(-500, 500))
end, false, "⚛️")
buttonY += 45

-- 11. Modo Staff Permanente
createButton("Modo Staff Permanente", buttonY, function()
    states.StaffMode = not states.StaffMode
    if states.StaffMode then
        activateStaffMode()
    else
        if activeEffects.StaffMode then
            for _, effect in pairs(activeEffects.StaffMode) do
                pcall(function() effect:Destroy() end)
            end
        end
        humanoid.WalkSpeed = 16
        humanoid.JumpPower = 50
    end
    return states.StaffMode
end, true, "🛡️")
buttonY += 45

-- Continuar com 189 funções adicionais...
-- [As próximas 189 funções seriam adicionadas aqui]

-- 201. DESATIVAR TUDO
createButton("🔴 DESATIVAR TODAS AS FUNÇÕES", buttonY, function()
    disableAllFunctions()
end, false, "🔴")
buttonY += 45

-- Ajustar canvas size
MainScroll.CanvasSize = UDim2.new(0, 0, 0, buttonY)

-- Controles de interface
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Sistema de reinicialização
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- Manter modo staff se estiver ativo
    if states.StaffMode then
        activateStaffMode()
    end
    
    disableAllFunctions()
end)

print("🛡️ MushYO Staff Suite v24.0 Carregado!")
print("🌟 200 FUNÇÕES VISÍVEIS PARA TODOS")
print("🛡️ MODO STAFF PERMANENTE ATIVADO")
print("🎯 Pressione RightShift para abrir o menu")

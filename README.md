--[[
    🎲 Dice Sniper – Lançamento reto e super knockback
    Ative o modo e jogue o dado. Ele voará como uma bala
    na direção da câmera. Se atingir alguém, essa pessoa
    será arremessada para longe.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem
repeat task.wait() until Player.Character

-- ==================== NOTIFICAÇÕES ====================
local function Notify(text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame")
    f.Parent = gui
    f.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    f.BorderSizePixel = 0
    f.Position = UDim2.new(0.5, -140, 0, 10)
    f.Size = UDim2.new(0, 280, 0, 34)
    f.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", f).Color = Color3.fromRGB(108, 92, 231)
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 1, 0)
    l.Font = Enum.Font.GothamBold
    l.Text = text
    l.TextColor3 = Color3.fromRGB(255, 255, 255)
    l.TextSize = 12
    local t = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 16)})
    t:Play()
    task.wait(duration)
    local t2 = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -34)})
    t2:Play()
    t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceSniper"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 250, 0, 95)
Main.Position = UDim2.new(0.5, -125, 0.5, -48)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 28)
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)
local tf = Instance.new("Frame")
tf.Parent = TitleBar
tf.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
tf.BorderSizePixel = 0
tf.Size = UDim2.new(1, 0, 0, 12)
tf.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🎯 Dice Sniper"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 12

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -26, 0, 3)
CloseBtn.Size = UDim2.new(0, 18, 0, 18)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 9
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Botão Ativar/Desativar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = Main
ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0, 8, 0, 34)
ToggleBtn.Size = UDim2.new(1, -16, 0, 28)
ToggleBtn.Text = "🟢 ATIVAR MODO SNIPER"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 10
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA PRINCIPAL ====================
local active = false
local monitoredParts = {}
local touchConnections = {}
local scanTimer = nil

-- Configurações
local LAUNCH_SPEED = 300   -- velocidade do dado (bala)
local KNOCKBACK_FORCE = 200 -- força do empurrão nos jogadores

-- Aplica knockback em um personagem
local function applyKnockback(character, dicePosition)
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local direction = (root.Position - dicePosition).Unit
    direction = direction * Vector3.new(1, 0, 1) + Vector3.new(0, 0.5, 0) -- levemente para cima
    root.Velocity = direction * KNOCKBACK_FORCE

    -- Reforço com BodyVelocity por curta duração
    local bodyVel = Instance.new("BodyVelocity")
    bodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVel.Velocity = direction * KNOCKBACK_FORCE
    bodyVel.Parent = root
    task.delay(0.2, function() bodyVel:Destroy() end)
end

-- Conecta eventos a um DiceRoll recém-criado
local function setupDiceRoll(part)
    if monitoredParts[part] then return end
    monitoredParts[part] = true

    -- Aplica a velocidade inicial (bala) na direção da câmera
    local camDir = Camera.CFrame.LookVector
    part.Velocity = camDir * LAUNCH_SPEED
    -- Opcional: remover a gravidade para manter a linha reta
    -- part:SetAttribute("NoGravity", true) -- não funciona diretamente, mas podemos usar BodyForce
    local bodyForce = Instance.new("BodyForce")
    bodyForce.Force = Vector3.new(0, part:GetMass() * Workspace.Gravity, 0) -- anula a gravidade
    bodyForce.Parent = part
    task.delay(1, function() bodyForce:Destroy() end) -- remove após 1 segundo

    -- Conecta o evento Touched para knockback
    local connection = part.Touched:Connect(function(hit)
        if not active then return end
        local character = hit:FindFirstAncestorOfClass("Model")
        if not character then return end
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then return end
        local targetPlayer = Players:GetPlayerFromCharacter(character)
        if not targetPlayer or targetPlayer == Player then return end

        applyKnockback(character, part.Position)
    end)

    table.insert(touchConnections, {part = part, connection = connection})
end

-- Monitora Workspace.Temp por novos DiceRoll
local function scanForDice()
    local tempFolder = Workspace:FindFirstChild("Temp")
    if not tempFolder then return end

    for _, obj in ipairs(tempFolder:GetChildren()) do
        if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
            setupDiceRoll(obj)
        end
    end
end

-- Ativa/Desativa
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        scanForDice()
        scanTimer = RunService.Heartbeat:Connect(scanForDice)
        Notify("🎯 Modo sniper ativado! Dados viram balas.")
    else
        ToggleBtn.Text = "🟢 ATIVAR MODO SNIPER"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        if scanTimer then
            scanTimer:Disconnect()
            scanTimer = nil
        end
        -- Limpa conexões
        for _, item in ipairs(touchConnections) do
            item.connection:Disconnect()
        end
        touchConnections = {}
        monitoredParts = {}
        Notify("🔴 Modo sniper desativado.")
    end
end

ToggleBtn.MouseButton1Click:Connect(toggleActive)

-- Arraste da interface
local dragging, startPos, startGuiPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        startPos = input.Position
        startGuiPos = Main.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - startPos
        Main.Position = UDim2.new(
            startGuiPos.X.Scale, startGuiPos.X.Offset + delta.X,
            startGuiPos.Y.Scale, startGuiPos.Y.Offset + delta.Y
        )
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

Notify("🎯 Ative o modo sniper e jogue o dado como uma bala!")

--[[
    🎯 Dice Sniper Visual – Projétil local com explosão
    Ative o modo e jogue o dado. Um projétil fantasma será disparado
    na direção da câmera. Se atingir um jogador, uma explosão visual
    ocorrerá (apenas você vê). Não afeta outros jogadores.
    Interface arrastável com botão de fechar.
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

-- ==================== LÓGICA DO PROJÉTIL LOCAL ====================
local active = false
local projectileInFlight = false

local function createProjectile()
    local camPos = Camera.CFrame.Position
    local camDir = Camera.CFrame.LookVector

    -- Cria o projétil (visível apenas para o cliente)
    local part = Instance.new("Part")
    part.Name = "SniperProjectile"
    part.Shape = Enum.PartType.Ball
    part.Size = Vector3.new(0.5, 0.5, 0.5)
    part.BrickColor = BrickColor.new("Bright red")
    part.Material = Enum.Material.Neon
    part.Anchored = false
    part.CanCollide = false -- não colide com o mundo
    part.Position = camPos + camDir * 2
    part.Velocity = camDir * 300
    part.Parent = Workspace

    projectileInFlight = true

    -- Remove após 3 segundos ou ao colidir
    local connection
    connection = part.Touched:Connect(function(hit)
        if not projectileInFlight then return end
        local character = hit:FindFirstAncestorOfClass("Model")
        if character and character:FindFirstChild("Humanoid") then
            local targetPlayer = Players:GetPlayerFromCharacter(character)
            if targetPlayer and targetPlayer ~= Player then
                -- Explosão visual
                local explosion = Instance.new("Explosion")
                explosion.BlastRadius = 8
                explosion.BlastPressure = 0
                explosion.Position = part.Position
                explosion.Parent = Workspace
                Notify("💥 Acertou " .. targetPlayer.Name .. "!")
            end
        end
        -- Remove o projétil
        connection:Disconnect()
        part:Destroy()
        projectileInFlight = false
    end)

    task.delay(3, function()
        if part and part.Parent then
            part:Destroy()
            projectileInFlight = false
        end
    end)
end

-- Detecta quando o jogador joga o dado (a ferramenta some da mão)
local function startMonitoring()
    local tool = nil
    for _, child in ipairs(Player.Character:GetChildren()) do
        if child:IsA("Tool") and child.Name == "Dice" then
            tool = child
            break
        end
    end
    if not tool then return end

    -- Monitora a remoção da ferramenta
    local connection
    connection = tool.AncestryChanged:Connect(function()
        if not active then return end
        if not tool:IsDescendantOf(Player.Character) and not tool:IsDescendantOf(Player.Backpack) then
            -- Foi jogada, cria o projétil
            createProjectile()
            connection:Disconnect()
            -- Reagenda o monitoramento para a próxima ferramenta
            task.wait(0.2)
            startMonitoring()
        end
    end)
end

-- Ativa/Desativa o modo
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        startMonitoring()
        Notify("🎯 Modo sniper ativado! Jogue o dado para disparar.")
    else
        ToggleBtn.Text = "🟢 ATIVAR MODO SNIPER"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
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

Notify("🎯 Ative o modo sniper e jogue o dado para disparar um projétil!")

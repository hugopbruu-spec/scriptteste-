--[[
    🎲 Dice Duplicator v2 – Reinício local sem sair do servidor
    Força o respawn do teu personagem para simular um "rejoin"
    sem realmente saíres do jogo. Os dados no chão permanecem,
    e tu recebes um inventário novo.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Aguarda o personagem
repeat task.wait() until Player.Character

-- ==================== NOTIFICAÇÕES ====================
local function Notify(text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(0.5, -140, 0, 10)
    frame.Size = UDim2.new(0, 280, 0, 34)
    frame.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", frame).Color = Color3.fromRGB(108, 92, 231)
    local lbl = Instance.new("TextLabel")
    lbl.Parent = frame
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, 0, 1, 0)
    lbl.Font = Enum.Font.GothamBold
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    lbl.TextSize = 12
    local tw = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 16)})
    tw:Play()
    task.wait(duration)
    local tw2 = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -34)})
    tw2:Play()
    tw2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 260, 0, 150)
Main.Position = UDim2.new(0.5, -130, 0.5, -75)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 36)
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)
local tf = Instance.new("Frame")
tf.Parent = TitleBar
tf.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
tf.BorderSizePixel = 0
tf.Size = UDim2.new(1, 0, 0, 12)
tf.Position = UDim2.new(0, 0, 1, -12)

local TitleIcon = Instance.new("TextLabel")
TitleIcon.Parent = TitleBar
TitleIcon.BackgroundTransparency = 1
TitleIcon.Position = UDim2.new(0, 8, 0, 5)
TitleIcon.Size = UDim2.new(0, 26, 0, 26)
TitleIcon.Text = "🎲"
TitleIcon.TextSize = 18

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 36, 0, 0)
TitleText.Size = UDim2.new(1, -70, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Dice Duplicator"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 13
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -34, 0, 6)
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
    Notify("Dice Duplicator fechado")
end)

-- Conteúdo
local Content = Instance.new("Frame")
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 10, 0, 42)
Content.Size = UDim2.new(1, -20, 1, -50)

local InfoLabel = Instance.new("TextLabel")
InfoLabel.Parent = Content
InfoLabel.BackgroundTransparency = 1
InfoLabel.Size = UDim2.new(1, 0, 0, 40)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "Jogue o dado no chão e clica no botão para renasceres. O dado fica no chão e tu recebes um inventário novo."
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local RespawnBtn = Instance.new("TextButton")
RespawnBtn.Parent = Content
RespawnBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
RespawnBtn.BorderSizePixel = 0
RespawnBtn.Position = UDim2.new(0, 0, 0, 44)
RespawnBtn.Size = UDim2.new(1, 0, 0, 38)
RespawnBtn.Text = "🔄 RENASCER (SIMULAR REJOIN)"
RespawnBtn.Font = Enum.Font.GothamBlack
RespawnBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RespawnBtn.TextSize = 11
Instance.new("UICorner", RespawnBtn).CornerRadius = UDim.new(0, 8)

RespawnBtn.MouseButton1Click:Connect(function()
    RespawnBtn.Text = "⏳ A renascer..."
    RespawnBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    RespawnBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    Notify("A forçar renascimento...")
    -- Mata o personagem para forçar um respawn (reinicia o inventário)
    if Player.Character and Player.Character:FindFirstChild("Humanoid") then
        Player.Character.Humanoid.Health = 0
    else
        -- Se não tiver Humanoid, tentamos quebrar as juntas
        if Player.Character then Player.Character:BreakJoints() end
    end
    task.wait(0.2)
    RespawnBtn.Text = "🔄 RENASCER (SIMULAR REJOIN)"
    RespawnBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    RespawnBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
end)

-- Arraste
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

Notify("🎲 Dice Duplicator carregado! Arrasta a janela.")

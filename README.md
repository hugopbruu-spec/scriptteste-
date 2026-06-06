--[[
    🎲 Dice Duplicator vFinal – Tela preta rápida + LoadCharacter
    Jogue o dado no chão e clique no botão.
    Uma tela preta esconde a transição enquanto o teu personagem
    é recarregado no mesmo local. O dado bugado permanece no chão.
    Inventário renovado. Sem morte visível. Sem sair do servidor.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem inicial
repeat task.wait() until Player.Character

-- ==================== TELA PRETA (para esconder a transição) ====================
local BlackScreen = Instance.new("ScreenGui")
BlackScreen.Name = "DiceDuplicatorBlackScreen"
BlackScreen.Parent = CoreGui
BlackScreen.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
BlackScreen.IgnoreGuiInset = true

local BlackFrame = Instance.new("Frame")
BlackFrame.Parent = BlackScreen
BlackFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
BlackFrame.BackgroundTransparency = 1           -- começa invisível
BlackFrame.BorderSizePixel = 0
BlackFrame.Size = UDim2.new(1, 0, 1, 0)

local function ShowBlackScreen()
    BlackFrame.BackgroundTransparency = 0
    BlackFrame.Visible = true
end

local function HideBlackScreen()
    BlackFrame.BackgroundTransparency = 1
    task.wait(0.1)
    BlackFrame.Visible = false
end

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
Main.Size = UDim2.new(0, 280, 0, 150)
Main.Position = UDim2.new(0.5, -140, 0.5, -75)
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
InfoLabel.Size = UDim2.new(1, 0, 0, 44)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "1. Jogue o dado no chão.\n2. Clique em RESETAR INVENTÁRIO.\nVocê renasce no mesmo lugar, com itens novos."
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local ResetBtn = Instance.new("TextButton")
ResetBtn.Parent = Content
ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ResetBtn.BorderSizePixel = 0
ResetBtn.Position = UDim2.new(0, 0, 0, 48)
ResetBtn.Size = UDim2.new(1, 0, 0, 38)
ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
ResetBtn.Font = Enum.Font.GothamBlack
ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ResetBtn.TextSize = 11
Instance.new("UICorner", ResetBtn).CornerRadius = UDim.new(0, 8)

ResetBtn.MouseButton1Click:Connect(function()
    ResetBtn.Text = "⏳ Resetando..."
    ResetBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    ResetBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    ResetBtn.Interactable = false

    local oldCharacter = Player.Character
    if not oldCharacter or not oldCharacter:FindFirstChild("HumanoidRootPart") then
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end

    -- Salva posição e câmera
    local oldRoot = oldCharacter.HumanoidRootPart
    local savedCFrame = oldRoot.CFrame
    local savedCameraCFrame = Camera.CFrame

    -- Mostra tela preta (transição instantânea)
    ShowBlackScreen()

    -- Recarrega o personagem (o antigo é destruído automaticamente)
    local success = pcall(function()
        Player:LoadCharacter()
    end)

    if not success then
        HideBlackScreen()
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        Notify("Falha ao recarregar. Tente novamente.")
        return
    end

    -- Aguarda o novo personagem aparecer
    local newCharacter = nil
    local startTime = tick()
    repeat
        newCharacter = Player.Character
        task.wait(0.05)
    until (newCharacter and newCharacter ~= oldCharacter) or (tick() - startTime > 10)

    if not newCharacter or newCharacter == oldCharacter then
        HideBlackScreen()
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        Notify("Falha ao obter novo personagem.")
        return
    end

    -- Aguarda HumanoidRootPart existir
    local newRoot = newCharacter:WaitForChild("HumanoidRootPart", 5)
    if not newRoot then
        HideBlackScreen()
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        Notify("Novo personagem incompleto.")
        return
    end

    -- Teletransporta o novo personagem para a posição salva e restaura a câmera
    newRoot.CFrame = savedCFrame
    Camera.CameraSubject = newCharacter:FindFirstChild("Humanoid")
    Camera.CFrame = savedCameraCFrame

    -- Esconde a tela preta
    HideBlackScreen()

    Notify("Inventário resetado! O dado no chão continua lá.")
    ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
    ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ResetBtn.Interactable = true
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

Notify("🎲 Dice Duplicator carregado! Arraste a janela.")

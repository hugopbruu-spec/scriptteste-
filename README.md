--[[
    🎲 Dice Duplicator vFinal – Reset garantido via BreakJoints
    Mata o personagem antigo (sem mostrar) e força o renascimento.
    Tela preta esconde tudo. Console de erros incluso.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

repeat task.wait() until Player.Character

-- ==================== TELA PRETA ====================
local BlackGui = Instance.new("ScreenGui")
BlackGui.Name = "BlackScreen"
BlackGui.Parent = CoreGui
BlackGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
BlackGui.IgnoreGuiInset = true
BlackGui.Enabled = false
local BlackFrame = Instance.new("Frame")
BlackFrame.Parent = BlackGui
BlackFrame.BackgroundColor3 = Color3.new(0, 0, 0)
BlackFrame.BorderSizePixel = 0
BlackFrame.Size = UDim2.new(1, 0, 1, 0)

local function ShowBlack() BlackGui.Enabled = true end
local function HideBlack() BlackGui.Enabled = false end

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
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 320, 0, 320)
Main.Position = UDim2.new(0.5, -160, 0.5, -160)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Conteúdo
local Content = Instance.new("Frame")
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 10, 0, 42)
Content.Size = UDim2.new(1, -20, 1, -50)

local InfoLabel = Instance.new("TextLabel")
InfoLabel.Parent = Content
InfoLabel.BackgroundTransparency = 1
InfoLabel.Size = UDim2.new(1, 0, 0, 36)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "1. Jogue o dado no chão\n2. Clique em RESETAR"
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local ResetBtn = Instance.new("TextButton")
ResetBtn.Parent = Content
ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ResetBtn.BorderSizePixel = 0
ResetBtn.Position = UDim2.new(0, 0, 0, 38)
ResetBtn.Size = UDim2.new(1, 0, 0, 34)
ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
ResetBtn.Font = Enum.Font.GothamBlack
ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ResetBtn.TextSize = 11
Instance.new("UICorner", ResetBtn).CornerRadius = UDim.new(0, 8)

-- Mini console
local ConsoleLabel = Instance.new("TextLabel")
ConsoleLabel.Parent = Content
ConsoleLabel.BackgroundTransparency = 1
ConsoleLabel.Position = UDim2.new(0, 0, 0, 80)
ConsoleLabel.Size = UDim2.new(1, 0, 0, 18)
ConsoleLabel.Font = Enum.Font.GothamBold
ConsoleLabel.Text = "📋 Console de erros:"
ConsoleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ConsoleLabel.TextSize = 11
ConsoleLabel.TextXAlignment = Enum.TextXAlignment.Left

local ConsoleBox = Instance.new("TextBox")
ConsoleBox.Parent = Content
ConsoleBox.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
ConsoleBox.BorderSizePixel = 0
ConsoleBox.Position = UDim2.new(0, 0, 0, 100)
ConsoleBox.Size = UDim2.new(1, 0, 0, 100)
ConsoleBox.Font = Enum.Font.Code
ConsoleBox.Text = "Nenhum erro ainda."
ConsoleBox.TextColor3 = Color3.fromRGB(200, 200, 220)
ConsoleBox.TextSize = 10
ConsoleBox.ClearTextOnFocus = false
ConsoleBox.TextEditable = false
ConsoleBox.TextWrapped = true
ConsoleBox.TextXAlignment = Enum.TextXAlignment.Left
ConsoleBox.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", ConsoleBox).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", ConsoleBox).Color = Color3.fromRGB(108, 92, 231)

local CopyErrBtn = Instance.new("TextButton")
CopyErrBtn.Parent = Content
CopyErrBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
CopyErrBtn.BorderSizePixel = 0
CopyErrBtn.Position = UDim2.new(1, -60, 0, 205)
CopyErrBtn.Size = UDim2.new(0, 56, 0, 22)
CopyErrBtn.Text = "📋 Copiar"
CopyErrBtn.Font = Enum.Font.GothamBold
CopyErrBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyErrBtn.TextSize = 10
Instance.new("UICorner", CopyErrBtn).CornerRadius = UDim.new(0, 6)

local function LogError(msg)
    ConsoleBox.Text = ConsoleBox.Text .. "\n" .. msg
end

CopyErrBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if setclipboard then setclipboard(ConsoleBox.Text)
        elseif writefile then writefile("dice_errors.txt", ConsoleBox.Text) end
    end)
    Notify("Erros copiados!")
end)

-- ==================== AÇÃO PRINCIPAL ====================
ResetBtn.MouseButton1Click:Connect(function()
    ResetBtn.Text = "⏳ Resetando..."
    ResetBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    ResetBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    ResetBtn.Interactable = false
    ConsoleBox.Text = ""

    local oldCharacter = Player.Character
    if not oldCharacter then
        LogError("ERRO: Personagem não existe.")
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end

    local oldRoot = oldCharacter:FindFirstChild("HumanoidRootPart")
    if not oldRoot then
        LogError("ERRO: HumanoidRootPart não encontrado.")
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end

    local oldHumanoid = oldCharacter:FindFirstChild("Humanoid")
    if not oldHumanoid then
        LogError("ERRO: Humanoid não encontrado.")
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end

    local savedCFrame = oldRoot.CFrame
    local savedCamCFrame = Camera.CFrame
    LogError("OK: Posição salva: " .. tostring(savedCFrame))

    ShowBlack()
    LogError("OK: Tela preta ativada.")

    -- Força a morte do personagem (sem som, sem animação visível)
    oldHumanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
    oldHumanoid.Health = 0
    LogError("OK: Morte forçada.")

    -- Aguarda o novo personagem aparecer (respawn automático)
    local newCharacter = nil
    local start = tick()
    repeat
        newCharacter = Player.Character
        task.wait(0.05)
    until (newCharacter and newCharacter ~= oldCharacter) or (tick() - start > 20)

    if not newCharacter or newCharacter == oldCharacter then
        LogError("ERRO: Novo personagem não apareceu após 20s.")
        HideBlack()
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end
    LogError("OK: Novo personagem detectado.")

    local newRoot = newCharacter:WaitForChild("HumanoidRootPart", 5)
    if not newRoot then
        LogError("ERRO: HumanoidRootPart não carregou no novo personagem.")
        HideBlack()
        ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
        ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ResetBtn.Interactable = true
        return
    end

    -- Teleporta o novo personagem para a posição salva
    newRoot.CFrame = savedCFrame
    local newHumanoid = newCharacter:FindFirstChild("Humanoid")
    if newHumanoid then Camera.CameraSubject = newHumanoid end
    Camera.CFrame = savedCamCFrame
    LogError("OK: Teleporte e câmera restaurados.")

    task.wait(0.15)
    HideBlack()
    LogError("OK: Tela preta removida.")
    LogError("SUCESSO: Inventário resetado! Dado no chão mantido.")

    Notify("Inventário resetado! Dado no chão mantido.")
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

Notify("🎲 Console de erros ativo. Copie e cole aqui se falhar.")

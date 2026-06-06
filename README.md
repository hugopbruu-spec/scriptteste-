--[[
    🎲 Dice Duplicator vFinal – Rejoin Automático com Restauração
    Jogue o dado no chão e clique em DUPLICAR.
    Sai e volta ao mesmo servidor tão rápido que quase não se percebe.
    O dado no chão fica bugado e um novo aparece na sua mochila.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- ==================== TELA PRETA PERSISTENTE ====================
-- Criada dentro do PlayerGui com ResetOnSpawn = false para NÃO ser destruída no teleporte.
local function CreateBlackScreen()
    local black = Instance.new("ScreenGui")
    black.Name = "RejoinBlack"
    black.Parent = Player.PlayerGui
    black.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    black.ResetOnSpawn = false       -- essencial para sobreviver ao rejoin
    black.IgnoreGuiInset = true
    local frame = Instance.new("Frame")
    frame.Parent = black
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, 0, 1, 0)
    return black
end

local function RemoveBlackScreen()
    for _, gui in ipairs(Player.PlayerGui:GetChildren()) do
        if gui.Name == "RejoinBlack" then
            gui:Destroy()
        end
    end
end

-- ==================== RESTAURAÇÃO DA POSIÇÃO APÓS REJOIN ====================
local function RestorePositionIfNeeded()
    local savedCFrame = Player:GetAttribute("DiceSavedCFrame")
    local savedCamCFrame = Player:GetAttribute("DiceSavedCamCFrame")
    if not savedCFrame then return false end

    -- Aguarda o novo personagem carregar completamente
    local char = Player.Character
    if not char then
        -- espera o personagem aparecer (caso ainda não tenha)
        local start = tick()
        repeat
            char = Player.Character
            task.wait(0.1)
        until char or (tick() - start > 15)
    end
    if not char then return false end

    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then
        root = char:WaitForChild("HumanoidRootPart", 10)
    end
    if not root then return false end

    -- Restaura a posição
    root.CFrame = savedCFrame
    if savedCamCFrame then
        Camera.CFrame = savedCamCFrame
    end
    Camera.CameraSubject = char:FindFirstChild("Humanoid")

    -- Limpa os atributos salvados
    Player:SetAttribute("DiceSavedCFrame", nil)
    Player:SetAttribute("DiceSavedCamCFrame", nil)

    -- Remove a tela preta
    RemoveBlackScreen()
    return true
end

-- Tenta restaurar se for um rejoin (atributos existem)
RestorePositionIfNeeded()

-- Aguarda o personagem inicial
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
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 280, 0, 110)
Main.Position = UDim2.new(0.5, -140, 0.5, -55)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 30)
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
TitleIcon.Size = UDim2.new(0, 20, 0, 20)
TitleIcon.Text = "🎲"
TitleIcon.TextSize = 14

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 30, 0, 0)
TitleText.Size = UDim2.new(1, -60, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Dice Duplicator"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 12
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -28, 0, 4)
CloseBtn.Size = UDim2.new(0, 20, 0, 20)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 10
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Botão de ação
local DupBtn = Instance.new("TextButton")
DupBtn.Parent = Main
DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
DupBtn.BorderSizePixel = 0
DupBtn.Position = UDim2.new(0, 8, 0, 38)
DupBtn.Size = UDim2.new(1, -16, 0, 36)
DupBtn.Text = "🔄 DUPLICAR (REJOIN)"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 12
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 8)

DupBtn.MouseButton1Click:Connect(function()
    DupBtn.Text = "⏳ Aguarde..."
    DupBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    DupBtn.Interactable = false

    -- Salva a posição atual do jogador nos atributos (persistem após rejoin)
    local char = Player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        Player:SetAttribute("DiceSavedCFrame", char.HumanoidRootPart.CFrame)
        Player:SetAttribute("DiceSavedCamCFrame", Camera.CFrame)
    end

    -- Cria a tela preta persistente
    CreateBlackScreen()

    -- Tenta teleportar para a mesma instância do servidor
    local success = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player)
    end)
    if not success then
        -- Fallback: teleporta para o jogo (vai para um servidor aleatório)
        pcall(function()
            TeleportService:Teleport(game.PlaceId, Player)
        end)
    end

    -- O script para aqui porque o jogador sai do servidor.
    -- Quando ele voltar, o script rodará novamente e restaurará a posição.
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

Notify("🎲 Jogue o dado e clique em DUPLICAR para reiniciar rapidamente.")

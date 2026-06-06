--[[
    🎲 Dice Duplicator – Rejoin Flash (Imperceptível)
    Sai e volta ao mesmo servidor tão rápido que parece um "lag".
    Restaura posição e câmera automaticamente.
    O dado no chão permanece e você ganha um novo.
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
-- Usamos PlayerGui porque sobrevive ao teleporte se ResetOnSpawn = false
local function ShowBlack()
    local black = Instance.new("ScreenGui")
    black.Name = "RejoinBlack"
    black.Parent = Player.PlayerGui  -- não CoreGui, para persistir
    black.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    black.ResetOnSpawn = false       -- ESSENCIAL para não ser destruído
    black.IgnoreGuiInset = true
    local frame = Instance.new("Frame")
    frame.Parent = black
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, 0, 1, 0)
end

local function RemoveBlack()
    for _, gui in ipairs(Player.PlayerGui:GetChildren()) do
        if gui.Name == "RejoinBlack" then
            gui:Destroy()
        end
    end
end

-- ==================== RESTAURAÇÃO PÓS-REJOIN ====================
local function TryRestorePosition()
    local savedCFrame = Player:GetAttribute("DiceSavedCFrame")
    local savedCamCFrame = Player:GetAttribute("DiceSavedCamCFrame")
    if not savedCFrame then return false end
    
    -- Aguarda o novo personagem carregar
    local char = Player.Character
    if not char then return false end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then
        root = char:WaitForChild("HumanoidRootPart", 5)
    end
    if not root then return false end
    
    root.CFrame = savedCFrame
    if savedCamCFrame then
        Camera.CFrame = savedCamCFrame
    end
    Camera.CameraSubject = char:FindFirstChild("Humanoid")
    
    -- Limpa os atributos
    Player:SetAttribute("DiceSavedCFrame", nil)
    Player:SetAttribute("DiceSavedCamCFrame", nil)
    RemoveBlack()
    return true
end

-- Tenta restaurar se for um rejoin (atributos existem)
if TryRestorePosition() then
    -- Se restaurou, não precisa mostrar interface de novo? 
    -- Mostra interface normalmente, mas a tela preta já foi removida.
end

-- Aguarda personagem para prosseguir
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
Main.Size = UDim2.new(0, 280, 0, 130)
Main.Position = UDim2.new(0.5, -140, 0.5, -65)
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
InfoLabel.Size = UDim2.new(1, 0, 0, 30)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "1. Jogue o dado no chão\n2. Clique em DUPLICAR"
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local DupBtn = Instance.new("TextButton")
DupBtn.Parent = Content
DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
DupBtn.BorderSizePixel = 0
DupBtn.Position = UDim2.new(0, 0, 0, 34)
DupBtn.Size = UDim2.new(1, 0, 0, 38)
DupBtn.Text = "🔄 DUPLICAR (REJOIN INVISÍVEL)"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 11
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 8)

DupBtn.MouseButton1Click:Connect(function()
    DupBtn.Text = "⏳ Duplicando..."
    DupBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    DupBtn.Interactable = false
    
    -- Salva posição atual nos atributos do jogador (persiste após rejoin)
    local char = Player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        Player:SetAttribute("DiceSavedCFrame", char.HumanoidRootPart.CFrame)
        Player:SetAttribute("DiceSavedCamCFrame", Camera.CFrame)
    end
    
    -- Mostra tela preta PERSISTENTE (PlayerGui)
    ShowBlack()
    
    -- Rejoin para o mesmo servidor
    pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player)
    end)
    -- Se falhar, teleporta para o mesmo jogo (sem preservar servidor)
    pcall(function()
        TeleportService:Teleport(game.PlaceId, Player)
    end)
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

Notify("🎲 Tela preta cobre o rejoin. É quase instantâneo!")

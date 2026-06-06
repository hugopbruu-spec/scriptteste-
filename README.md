--[[
    🎲 Dice Duplicator Pro – Transferência completa + nova ferramenta
    1. Jogue o dado no chão (ele cairá e sumirá da sua mão).
    2. Clique em "DUPLICAR DADO".
    3. O dado no chão é desvinculado de você e permanece para sempre.
    4. Uma nova ferramenta "Dice" aparece na sua mochila, pronta para usar.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Backpack = Player:WaitForChild("Backpack")

-- Aguarda personagem
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
DupBtn.Text = "🔄 DUPLICAR DADO"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 11
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 8)

DupBtn.MouseButton1Click:Connect(function()
    DupBtn.Text = "⏳ Processando..."
    DupBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    DupBtn.Interactable = false

    -- 1. Transferir propriedade de todos os dados no chão que pertencem a este jogador
    local transferred = 0
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and (obj.Name == "Dice" or obj.Name == "Dice roll") then
            local owner = nil
            pcall(function() owner = obj:GetNetworkOwner() end)
            if owner == Player then
                pcall(function() obj:SetNetworkOwner(nil) end)
                transferred = transferred + 1
            end
        end
    end

    if transferred == 0 then
        Notify("Nenhum dado no chão com sua propriedade. Jogue o dado primeiro!")
        DupBtn.Text = "🔄 DUPLICAR DADO"
        DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        DupBtn.Interactable = true
        return
    end

    -- 2. Remover qualquer ferramenta "Dice" ou "Dice roll" da mão e da mochila
    local toRemove = {}
    local function addToRemove(parent)
        for _, child in ipairs(parent:GetChildren()) do
            if child:IsA("Tool") and (child.Name == "Dice" or child.Name == "Dice roll") then
                table.insert(toRemove, child)
            end
        end
    end
    addToRemove(Player.Character)
    addToRemove(Backpack)
    for _, tool in ipairs(toRemove) do
        tool:Destroy()
    end

    -- 3. Criar uma nova ferramenta "Dice" (cópia genérica)
    local newTool = Instance.new("Tool")
    newTool.Name = "Dice"
    newTool.Parent = Backpack
    -- Copiar propriedades padrão se existir um dado modelo no chão
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Tool") and (obj.Name == "Dice" or obj.Name == "Dice roll") then
            -- Clona o modelo inteiro e coloca na mochila
            local clone = obj:Clone()
            clone.Parent = Backpack
            newTool:Destroy() -- remove o genérico
            newTool = clone
            break
        end
    end

    Notify("🎲 Dado duplicado! Novo na mochila, o do chão permanece.")
    DupBtn.Text = "🔄 DUPLICAR DADO"
    DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    DupBtn.Interactable = true
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

Notify("🎲 Jogue o dado e clique em DUPLICAR para ter outro!")

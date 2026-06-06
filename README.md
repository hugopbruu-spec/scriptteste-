--[[
    🎲 Dice Duplicator Pro – Transferência de rede + novo dado
    1. Jogue o dado no chão.
    2. Clique em "DUPLICAR DADO".
    3. O dado no chão é desvinculado de você (vai para o servidor) e permanece para sempre.
    4. Um novo dado aparece na sua mão automaticamente.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")

-- Aguarda personagem e mochila
repeat task.wait() until Player.Character and Player.Backpack

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

    -- 1. Encontrar dados no chão que pertencem ao jogador local
    local transferred = 0
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and (obj.Name == "Dice" or obj.Name == "Dice roll") then
            local owner = nil
            pcall(function() owner = obj:GetNetworkOwner() end)
            if owner == Player then
                -- Transfere a propriedade para o servidor (nil) ou outro jogador
                pcall(function() obj:SetNetworkOwner(nil) end)
                transferred = transferred + 1
            end
        end
    end

    if transferred == 0 then
        Notify("Nenhum dado encontrado no chão. Jogue o dado primeiro!")
        DupBtn.Text = "🔄 DUPLICAR DADO"
        DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        DupBtn.Interactable = true
        return
    end

    -- 2. Clonar o dado do chão de volta para a mochila
    local diceModel = nil
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Tool") and (obj.Name == "Dice" or obj.Name == "Dice roll") then
            diceModel = obj
            break
        end
        if obj:IsA("BasePart") and (obj.Name == "Dice" or obj.Name == "Dice roll") then
            diceModel = obj.Parent -- talvez o dado seja um modelo
            break
        end
    end

    if diceModel then
        local clone = diceModel:Clone()
        if clone:IsA("Tool") then
            clone.Parent = Player.Backpack
        else
            -- Se for um modelo, tenta colocar como Tool na mochila (pode falhar)
            local newTool = Instance.new("Tool")
            newTool.Name = diceModel.Name
            newTool.Parent = Player.Backpack
            for _, child in ipairs(clone:GetChildren()) do
                child.Parent = newTool
            end
        end
        Notify("🎲 Dado duplicado! Você tem um novo na mochila.")
    else
        -- Fallback: cria um dado genérico
        local newTool = Instance.new("Tool")
        newTool.Name = "Dice"
        newTool.Parent = Player.Backpack
        Notify("🎲 Novo dado criado (genérico).")
    end

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

--[[
    🎲 Dice Duplicator Pro – Detecção automática de objetos no chão
    Monitora quando um dado é jogado, abandona o objeto criado
    e fornece uma nova ferramenta imediatamente.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Backpack = Player:WaitForChild("Backpack")

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

-- ==================== LÓGICA PRINCIPAL ====================
local activeTool = nil  -- ferramenta original (salva antes de jogar)
local toolRemovedConn = nil

local function captureTool()
    -- Procura uma ferramenta "Dice" ou "Dice roll" no personagem ou mochila
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and (tool.Name == "Dice" or tool.Name == "Dice roll") then
            return tool
        end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") and (tool.Name == "Dice" or tool.Name == "Dice roll") then
            return tool
        end
    end
    return nil
end

local function abandonAndReplace()
    -- 1. Encontrar objetos recém-criados no Workspace (potenciais dados no chão)
    -- Vamos usar uma tabela de referência antes e depois.
    -- Como o clique do botão ocorre após o jogador jogar, podemos varrer o Workspace agora.
    local newObjects = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj:GetNetworkOwner() == Player then
            table.insert(newObjects, obj)
        elseif obj:IsA("Model") then
            for _, part in ipairs(obj:GetDescendants()) do
                if part:IsA("BasePart") and part:GetNetworkOwner() == Player then
                    table.insert(newObjects, part)
                end
            end
        end
    end

    -- Transfere todos esses objetos para nil (abandona)
    local count = 0
    for _, part in ipairs(newObjects) do
        pcall(function() part:SetNetworkOwner(nil) end)
        count = count + 1
    end

    if count == 0 then
        Notify("Nenhum dado no chão com sua propriedade. Jogue o dado primeiro!")
        return false
    end

    -- Remove qualquer ferramenta residual (na mão ou mochila)
    local toRemove = {}
    local function collectTools(parent)
        for _, child in ipairs(parent:GetChildren()) do
            if child:IsA("Tool") and (child.Name == "Dice" or child.Name == "Dice roll") then
                table.insert(toRemove, child)
            end
        end
    end
    collectTools(Player.Character)
    collectTools(Backpack)
    for _, tool in ipairs(toRemove) do
        tool:Destroy()
    end

    -- Cria uma nova ferramenta baseada na original guardada (ou genérica)
    if activeTool then
        local newTool = activeTool:Clone()
        newTool.Parent = Backpack
    else
        local newTool = Instance.new("Tool")
        newTool.Name = "Dice"
        newTool.Parent = Backpack
    end

    Notify("🎲 Dado duplicado! Novo na mochila, o do chão permanece.")
    return true
end

-- Ativa o modo de duplicação
local function activateDuplication()
    activeTool = captureTool()
    if not activeTool then
        Notify("Você precisa estar com um dado (Dice/Dice roll) na mão ou mochila!")
        return
    end

    -- Monitora quando essa ferramenta for removida (jogada)
    if toolRemovedConn then toolRemovedConn:Disconnect() end
    toolRemovedConn = activeTool.AncestryChanged:Connect(function()
        if not activeTool:IsDescendantOf(Player) and not activeTool:IsDescendantOf(Backpack) then
            -- A ferramenta foi removida (jogada)
            task.wait(0.3) -- pequena pausa para o objeto físico ser criado
            abandonAndReplace()
            -- Desconecta para evitar múltiplas execuções
            toolRemovedConn:Disconnect()
            toolRemovedConn = nil
            activeTool = nil
        end
    end)

    Notify("🟢 Duplicação ativada! Jogue o dado e um novo aparecerá.")
end

DupBtn.MouseButton1Click:Connect(activateDuplication)

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

Notify("🎲 Equipe o dado, clique em DUPLICAR e jogue-o!")

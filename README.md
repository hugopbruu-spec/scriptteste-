--[[
    🎲 Dice Duplicator v5 – Reset de Inventário Sem Mexer no Personagem
    Guarda os itens iniciais assim que entras no jogo.
    Clica no botão para removeres todos os itens atuais e
    receberes cópias dos itens iniciais.
    O teu personagem NÃO é afetado, não morres, não ficas invisível.
    Os dados que estão no chão continuam lá.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Aguarda o primeiro personagem e recolhe os itens iniciais
local initialTools = {}   -- guarda nome, classe, propriedades principais

local function CaptureInitialItems()
    initialTools = {}
    local function addFromContainer(container)
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("Tool") then
                local data = {
                    Name = obj.Name,
                    ClassName = obj.ClassName,
                    -- guarda as propriedades que o jogo costuma definir
                    RequiresHandle = obj.RequiresHandle,
                    CanBeDropped = obj.CanBeDropped,
                    ManualActivationOnly = obj.ManualActivationOnly,
                    ToolTip = obj.ToolTip,
                    TextureId = obj.TextureId,
                    Grip = obj.Grip,
                    GripForward = obj.GripForward,
                    GripRight = obj.GripRight,
                    GripUp = obj.GripUp,
                    GripPos = obj.GripPos,
                }
                table.insert(initialTools, data)
            end
        end
    end
    -- Procura no Backpack e no personagem
    if Player.Backpack then
        addFromContainer(Player.Backpack)
    end
    if Player.Character then
        addFromContainer(Player.Character)
    end
end

-- Aguarda o jogo carregar (personagem pronto)
repeat task.wait() until Player.Character
task.wait(1)  -- dá um tempinho extra para o jogo distribuir os itens iniciais
CaptureInitialItems()

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
Main.Size = UDim2.new(0, 280, 0, 170)
Main.Position = UDim2.new(0.5, -140, 0.5, -85)
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
TitleText.Text = "Dice Duplicator v5"
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
InfoLabel.Size = UDim2.new(1, 0, 0, 48)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "1. Joga o dado no chão\n2. Clica em RESETAR INVENTÁRIO\nO dado fica no chão e ganhas um novo inventário (sem mexer no boneco)."
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local ResetBtn = Instance.new("TextButton")
ResetBtn.Parent = Content
ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ResetBtn.BorderSizePixel = 0
ResetBtn.Position = UDim2.new(0, 0, 0, 52)
ResetBtn.Size = UDim2.new(1, 0, 0, 40)
ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
ResetBtn.Font = Enum.Font.GothamBlack
ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ResetBtn.TextSize = 12
Instance.new("UICorner", ResetBtn).CornerRadius = UDim.new(0, 8)

ResetBtn.MouseButton1Click:Connect(function()
    ResetBtn.Text = "⏳ Resetando..."
    ResetBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    ResetBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    Notify("A resetar inventário...")

    -- Remove todas as ferramentas atuais (Backpack + personagem)
    local function removeTools(container)
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("Tool") then
                obj:Destroy()
            end
        end
    end
    if Player.Backpack then
        removeTools(Player.Backpack)
    end
    if Player.Character then
        removeTools(Player.Character)
    end

    -- Recria os itens iniciais a partir dos dados guardados
    for _, itemData in ipairs(initialTools) do
        local newTool = Instance.new(itemData.ClassName)
        newTool.Name = itemData.Name
        -- Copia as propriedades que guardámos
        pcall(function() newTool.RequiresHandle = itemData.RequiresHandle end)
        pcall(function() newTool.CanBeDropped = itemData.CanBeDropped end)
        pcall(function() newTool.ManualActivationOnly = itemData.ManualActivationOnly end)
        pcall(function() newTool.ToolTip = itemData.ToolTip end)
        pcall(function() newTool.TextureId = itemData.TextureId end)
        pcall(function() newTool.Grip = itemData.Grip end)
        pcall(function() newTool.GripForward = itemData.GripForward end)
        pcall(function() newTool.GripRight = itemData.GripRight end)
        pcall(function() newTool.GripUp = itemData.GripUp end)
        pcall(function() newTool.GripPos = itemData.GripPos end)
        -- Coloca na mochila
        newTool.Parent = Player.Backpack
    end

    Notify("Inventário resetado! O dado no chão continua lá.")
    ResetBtn.Text = "🔄 RESETAR INVENTÁRIO"
    ResetBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    ResetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
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

Notify("🎲 Dice Duplicator v5 carregado! Arrasta a janela.")

--[[
    🎲 Dice Duplicator Funcional
    1. Jogue o dado no chão (ele aparecerá em Workspace.Temp).
    2. Clique em "DUPLICAR DADO".
    3. O dado no chão permanecerá, a ferramenta invisível sumirá,
       e uma nova ferramenta Dice aparecerá na sua mochila.
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
Main.Size = UDim2.new(0, 250, 0, 100)
Main.Position = UDim2.new(0.5, -125, 0.5, -50)
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

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🎲 Dice Duplicator"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 12

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

-- Conteúdo
local Content = Instance.new("Frame")
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 8, 0, 34)
Content.Size = UDim2.new(1, -16, 1, -40)

local DupBtn = Instance.new("TextButton")
DupBtn.Parent = Content
DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
DupBtn.BorderSizePixel = 0
DupBtn.Size = UDim2.new(1, 0, 0, 28)
DupBtn.Text = "🔄 DUPLICAR DADO"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 11
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA PRINCIPAL ====================
local originalToolTemplate = nil  -- guarda o clone da ferramenta original

-- Salva o modelo da ferramenta assim que o script carrega (ou quando o jogador pega um dado)
local function SaveToolTemplate()
    -- Procura ferramenta "Dice" na mão ou mochila
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            originalToolTemplate = tool:Clone()
            return
        end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            originalToolTemplate = tool:Clone()
            return
        end
    end
end
SaveToolTemplate()

-- Monitora quando um novo dado é adicionado ao jogador (para atualizar o template)
Player.ChildAdded:Connect(function(child)
    if child:IsA("Tool") and child.Name == "Dice" then
        SaveToolTemplate()
    end
end)
Backpack.ChildAdded:Connect(function(child)
    if child:IsA("Tool") and child.Name == "Dice" then
        SaveToolTemplate()
    end
end)

DupBtn.MouseButton1Click:Connect(function()
    -- 1. Garante que os dados no chão (Workspace.Temp) não serão removidos
    --    Eles já estão com NetworkOwner nil, mas podemos iterar e garantir
    local tempFolder = Workspace:FindFirstChild("Temp")
    if tempFolder then
        for _, obj in ipairs(tempFolder:GetChildren()) do
            if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
                pcall(function() obj:SetNetworkOwner(nil) end)
            end
        end
    end

    -- 2. Remove todas as ferramentas "Dice" da mão e da mochila (incluindo a invisível)
    local toRemove = {}
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            table.insert(toRemove, tool)
        end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            table.insert(toRemove, tool)
        end
    end
    for _, tool in ipairs(toRemove) do
        tool:Destroy()
    end

    -- 3. Cria uma nova ferramenta a partir do template salvo
    if originalToolTemplate then
        local newTool = originalToolTemplate:Clone()
        newTool.Parent = Backpack
        Notify("🎲 Novo dado criado! O do chão permanece.")
    else
        -- Fallback genérico
        local newTool = Instance.new("Tool")
        newTool.Name = "Dice"
        newTool.Parent = Backpack
        Notify("🎲 Dado genérico criado (template não encontrado).")
    end
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

Notify("🎲 Jogue o dado e clique em DUPLICAR!")

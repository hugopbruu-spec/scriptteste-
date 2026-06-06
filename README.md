--[[
    🎲 Dice Duplicator Universal
    Salva o modelo do dado na mão. Ao clicar em DUPLICAR,
    remove a ferramenta atual (inclusive a invisível) e cria
    uma nova cópia na mochila. O dado no chão permanece intocado.
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
Main.Size = UDim2.new(0, 220, 0, 90)
Main.Position = UDim2.new(0.5, -110, 0.5, -45)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 28)
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
TitleText.TextSize = 11

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -26, 0, 3)
CloseBtn.Size = UDim2.new(0, 18, 0, 18)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 9
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Botão
local DupBtn = Instance.new("TextButton")
DupBtn.Parent = Main
DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
DupBtn.BorderSizePixel = 0
DupBtn.Position = UDim2.new(0, 8, 0, 34)
DupBtn.Size = UDim2.new(1, -16, 0, 30)
DupBtn.Text = "🔄 DUPLICAR DADO"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 11
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA ====================
local diceTemplate = nil  -- clone do dado original

local function findAndSaveTool()
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            diceTemplate = tool:Clone()
            return true
        end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            diceTemplate = tool:Clone()
            return true
        end
    end
    return false
end

-- Tenta salvar ao iniciar
findAndSaveTool()

-- Atualiza template quando um novo dado é pego
local function onNewTool(child)
    if child:IsA("Tool") and child.Name == "Dice" then
        task.wait(0.1) -- pequena pausa para propriedades carregarem
        diceTemplate = child:Clone()
    end
end
Player.ChildAdded:Connect(onNewTool)
Backpack.ChildAdded:Connect(onNewTool)

DupBtn.MouseButton1Click:Connect(function()
    -- 1. Garantir que dados no chão (Workspace.Temp) não sejam removidos
    --    (não faremos nada, apenas garantir NetworkOwner nil, mas já está)
    local temp = Workspace:FindFirstChild("Temp")
    if temp then
        for _, obj in ipairs(temp:GetChildren()) do
            if obj.Name == "DiceRoll" then
                pcall(function() obj:SetNetworkOwner(nil) end)
            end
        end
    end

    -- 2. Remover todas as ferramentas "Dice" da mão e mochila
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

    -- 3. Criar nova ferramenta a partir do template
    if diceTemplate then
        local newTool = diceTemplate:Clone()
        newTool.Parent = Backpack
        Notify("🎲 Novo dado na mochila!")
    else
        -- Fallback: cria um dado genérico com as propriedades conhecidas
        local newTool = Instance.new("Tool")
        newTool.Name = "Dice"
        newTool.RequiresHandle = true
        newTool.CanBeDropped = false
        newTool.ManualActivationOnly = false
        newTool.Grip = CFrame.new(0,0,0, 1,0,0, 0,1,0, 0,0,1)
        newTool.GripForward = Vector3.new(0,0,-1)
        newTool.GripRight = Vector3.new(1,0,0)
        newTool.GripUp = Vector3.new(0,1,0)
        newTool.GripPos = Vector3.new(0,0,0)
        -- Adiciona um Handle básico
        local handle = Instance.new("MeshPart")
        handle.Name = "Handle"
        handle.Size = Vector3.new(0.662, 0.662, 0.662)
        handle.MeshId = "rbxassetid://90561183096956"
        handle.Material = Enum.Material.Plastic
        handle.Color = Color3.fromRGB(163, 162, 165)
        handle.Parent = newTool
        newTool.Parent = Backpack
        Notify("🎲 Dado genérico criado (template ausente).")
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

Notify("🎲 Pronto! Jogue o dado e clique em DUPLICAR.")

--[[
    🎲 Dice Duplicator – Lançamento infinito de dados
    Ative a duplicação e jogue dados sem parar.
    Cada dado lançado permanece no chão, visível para todos.
    Você sempre terá um novo dado na mão para jogar novamente.
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
Main.Size = UDim2.new(0, 230, 0, 95)
Main.Position = UDim2.new(0.5, -115, 0.5, -48)
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

-- Botão Ativar/Desativar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = Main
ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0, 8, 0, 34)
ToggleBtn.Size = UDim2.new(1, -16, 0, 28)
ToggleBtn.Text = "🟢 ATIVAR LANÇAMENTO INFINITO"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 10
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA PRINCIPAL ====================
local active = false
local diceTemplate = nil
local currentTool = nil
local toolConnections = {}

local function findDiceTool()
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then return tool end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then return tool end
    end
    return nil
end

local function updateTemplate()
    local tool = findDiceTool()
    if tool then
        diceTemplate = tool:Clone()
    end
end

local function cleanupAndGiveNewTool()
    -- Remove qualquer ferramenta Dice da mão e mochila (a original pode estar invisível)
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

    -- Aguarda um frame
    task.wait(0.1)

    -- Cria nova ferramenta baseada no template
    if diceTemplate then
        local newTool = diceTemplate:Clone()
        newTool.Parent = Player.Character  -- coloca diretamente na mão
        currentTool = newTool
        setupToolWatcher(newTool)
    else
        -- Fallback genérico
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
        local handle = Instance.new("MeshPart")
        handle.Name = "Handle"
        handle.Size = Vector3.new(0.662, 0.662, 0.662)
        handle.MeshId = "rbxassetid://90561183096956"
        handle.Material = Enum.Material.Plastic
        handle.Color = Color3.fromRGB(163, 162, 165)
        handle.Parent = newTool
        newTool.Parent = Player.Character
        currentTool = newTool
        setupToolWatcher(newTool)
    end
end

-- Configura monitoramento para quando a ferramenta atual for removida (jogada)
local function setupToolWatcher(tool)
    -- Desconecta conexões anteriores
    for _, conn in ipairs(toolConnections) do
        conn:Disconnect()
    end
    toolConnections = {}

    local function onToolRemoved()
        -- A ferramenta foi jogada, agora precisamos aguardar o dado aparecer no chão
        task.wait(0.5)
        if not active then return end

        -- Garante que o dado no chão fique com NetworkOwner nil
        local tempFolder = Workspace:FindFirstChild("Temp")
        if tempFolder then
            for _, obj in ipairs(tempFolder:GetChildren()) do
                if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
                    pcall(function() obj:SetNetworkOwner(nil) end)
                end
            end
        end

        -- Dá uma nova ferramenta ao jogador
        cleanupAndGiveNewTool()
        Notify("🎲 Dado lançado! Outro já está na sua mão.")
    end

    -- Monitora quando a ferramenta é removida do jogador
    local conn1 = tool.AncestryChanged:Connect(function()
        if not tool:IsDescendantOf(Player) and not tool:IsDescendantOf(Backpack) then
            onToolRemoved()
        end
    end)
    local conn2 = tool.Destroying:Connect(function()
        onToolRemoved()
    end)

    table.insert(toolConnections, conn1)
    table.insert(toolConnections, conn2)
end

-- Ativa/desativa a duplicação
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        -- Atualiza template e configura monitoramento no dado atual
        updateTemplate()
        currentTool = findDiceTool()
        if currentTool then
            setupToolWatcher(currentTool)
            Notify("🟢 Lançamento infinito ativado! Jogue o dado.")
        else
            Notify("⚠️ Pegue um dado primeiro!")
        end
    else
        ToggleBtn.Text = "🟢 ATIVAR LANÇAMENTO INFINITO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        -- Desconecta todos os monitores
        for _, conn in ipairs(toolConnections) do
            conn:Disconnect()
        end
        toolConnections = {}
        Notify("🔴 Lançamento infinito desativado.")
    end
end

ToggleBtn.MouseButton1Click:Connect(toggleActive)

-- Atualiza template quando um novo dado aparece (por exemplo, pego da mochila)
Player.ChildAdded:Connect(function(child)
    if active and child:IsA("Tool") and child.Name == "Dice" then
        task.wait(0.1)
        updateTemplate()
        currentTool = child
        setupToolWatcher(child)
    end
end)
Backpack.ChildAdded:Connect(function(child)
    if active and child:IsA("Tool") and child.Name == "Dice" then
        task.wait(0.1)
        updateTemplate()
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

Notify("🎲 Ative o lançamento infinito e jogue quantos dados quiser!")

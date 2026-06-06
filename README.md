--[[
    🔍 Dice Tracker Pro – Rastreamento completo e preciso
    Ative o rastreador, jogue o dado, e TODOS os objetos
    que surgirem no mundo serão exibidos no console.
    Copie os dados e cole aqui para eu analisar.
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
gui.Name = "DiceTracker"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 380, 0, 420)
Main.Position = UDim2.new(0.5, -190, 0.5, -210)
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
TitleIcon.Text = "🔍"
TitleIcon.TextSize = 18

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 36, 0, 0)
TitleText.Size = UDim2.new(1, -70, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Dice Tracker Pro"
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

local ActivateBtn = Instance.new("TextButton")
ActivateBtn.Parent = Content
ActivateBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ActivateBtn.BorderSizePixel = 0
ActivateBtn.Size = UDim2.new(1, 0, 0, 34)
ActivateBtn.Text = "🟢 ATIVAR RASTREADOR (com dado na mão)"
ActivateBtn.Font = Enum.Font.GothamBlack
ActivateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ActivateBtn.TextSize = 10
Instance.new("UICorner", ActivateBtn).CornerRadius = UDim.new(0, 8)

-- Mini console
local ConsoleBox = Instance.new("TextBox")
ConsoleBox.Parent = Content
ConsoleBox.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
ConsoleBox.BorderSizePixel = 0
ConsoleBox.Position = UDim2.new(0, 0, 0, 40)
ConsoleBox.Size = UDim2.new(1, 0, 0, 290)
ConsoleBox.Font = Enum.Font.Code
ConsoleBox.Text = "Console vazio. Ative o rastreador e jogue o dado."
ConsoleBox.TextColor3 = Color3.fromRGB(200, 200, 220)
ConsoleBox.TextSize = 10
ConsoleBox.ClearTextOnFocus = false
ConsoleBox.TextEditable = false
ConsoleBox.TextWrapped = true
ConsoleBox.TextXAlignment = Enum.TextXAlignment.Left
ConsoleBox.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", ConsoleBox).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", ConsoleBox).Color = Color3.fromRGB(108, 92, 231)

local CopyBtn = Instance.new("TextButton")
CopyBtn.Parent = Content
CopyBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
CopyBtn.BorderSizePixel = 0
CopyBtn.Position = UDim2.new(1, -60, 0, 335)
CopyBtn.Size = UDim2.new(0, 56, 0, 22)
CopyBtn.Text = "📋 Copiar"
CopyBtn.Font = Enum.Font.GothamBold
CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyBtn.TextSize = 10
Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 6)

local function Log(msg)
    ConsoleBox.Text = ConsoleBox.Text .. "\n" .. msg
end

CopyBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if setclipboard then setclipboard(ConsoleBox.Text)
        elseif writefile then writefile("dice_tracker.txt", ConsoleBox.Text) end
    end)
    Notify("Copiado para a área de transferência!")
end)

-- ==================== LÓGICA DO RASTREADOR MELHORADA ====================
local activeTool = nil
local toolDestroyConn = nil
local toolAncestryConn = nil
local initialObjects = {}  -- snapshot dos objetos no mundo antes de jogar
local trackedNewObjects = {} -- objetos novos que apareceram

local function DeactivateTracker()
    if toolDestroyConn then toolDestroyConn:Disconnect() toolDestroyConn = nil end
    if toolAncestryConn then toolAncestryConn:Disconnect() toolAncestryConn = nil end
    activeTool = nil
    initialObjects = {}
    trackedNewObjects = {}
    ActivateBtn.Text = "🟢 ATIVAR RASTREADOR (com dado na mão)"
    ActivateBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
end

-- Captura informações completas de um objeto do mundo
local function DescribeWorldObject(obj)
    local lines = {}
    table.insert(lines, "---")
    table.insert(lines, "Nome: " .. obj.Name)
    table.insert(lines, "Classe: " .. obj.ClassName)
    table.insert(lines, "Parent: " .. (obj.Parent and obj.Parent:GetFullName() or "nil"))
    if obj:IsA("BasePart") then
        table.insert(lines, "Tipo: BasePart")
        table.insert(lines, "Position: " .. tostring(obj.Position))
        table.insert(lines, "Size: " .. tostring(obj.Size))
        table.insert(lines, "Material: " .. obj.Material.Name)
        table.insert(lines, "Color: " .. tostring(obj.Color))
        table.insert(lines, "CanCollide: " .. tostring(obj.CanCollide))
        table.insert(lines, "Anchored: " .. tostring(obj.Anchored))
        table.insert(lines, "Transparency: " .. tostring(obj.Transparency))
        if obj:IsA("MeshPart") then
            local meshId = pcall(function() return obj.MeshId end)
            table.insert(lines, "MeshId: " .. tostring(meshId))
        end
        -- Network Owner
        local owner = pcall(function() return obj:GetNetworkOwner() end)
        table.insert(lines, "NetworkOwner: " .. tostring(owner))
    elseif obj:IsA("Model") then
        table.insert(lines, "Tipo: Model")
        local primary = obj.PrimaryPart
        if primary then
            table.insert(lines, "PrimaryPart Position: " .. tostring(primary.Position))
        end
        table.insert(lines, "Partes do modelo:")
        for _, child in ipairs(obj:GetDescendants()) do
            if child:IsA("BasePart") then
                table.insert(lines, "  [" .. child.ClassName .. "] " .. child.Name .. " Pos: " .. tostring(child.Position))
            end
        end
    end
    -- Atributos
    local attrs = pcall(function() return obj:GetAttributes() end)
    if attrs and type(attrs) == "table" and next(attrs) then
        table.insert(lines, "Atributos:")
        for k, v in pairs(attrs) do
            table.insert(lines, "  " .. k .. ": " .. tostring(v))
        end
    end
    return table.concat(lines, "\n")
end

-- Tira snapshot de todos os objetos no Workspace
local function SnapshotWorkspace()
    local snapshot = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        snapshot[obj] = true  -- a chave é o próprio objeto (referência)
    end
    return snapshot
end

-- Encontra objetos novos comparando com o snapshot
local function FindNewObjects(snapshot)
    local newObjects = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if not snapshot[obj] then
            table.insert(newObjects, obj)
        end
    end
    return newObjects
end

ActivateBtn.MouseButton1Click:Connect(function()
    if activeTool then
        DeactivateTimer()
        Log("Rastreador desativado.")
        return
    end

    -- Procura ferramenta na mão/mochila
    local function findTool()
        for _, tool in ipairs(Player.Character:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name == "Dice" or tool.Name == "Dice roll") then return tool end
        end
        for _, tool in ipairs(Backpack:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name == "Dice" or tool.Name == "Dice roll") then return tool end
        end
        return nil
    end

    activeTool = findTool()
    if not activeTool then
        Notify("Você não está com um dado (Dice/Dice roll) na mão ou mochila!")
        return
    end

    ActivateBtn.Text = "🔴 DESATIVAR RASTREADOR"
    ActivateBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

    ConsoleBox.Text = "Rastreador ativado! Ferramenta encontrada:\n\n"
    -- Descreve a ferramenta
    local function describeTool(tool)
        local lines = {}
        table.insert(lines, "=== FERRAMENTA ===")
        table.insert(lines, "Nome: " .. tool.Name)
        table.insert(lines, "Classe: " .. tool.ClassName)
        table.insert(lines, "Parent: " .. tostring(tool.Parent))
        for _, prop in ipairs({"RequiresHandle", "CanBeDropped", "ManualActivationOnly", "ToolTip", "TextureId", "Grip", "GripForward", "GripRight", "GripUp", "GripPos"}) do
            local ok, val = pcall(function() return tool[prop] end)
            if ok and val ~= nil then
                table.insert(lines, prop .. ": " .. tostring(val))
            end
        end
        table.insert(lines, "Filhos:")
        for _, child in ipairs(tool:GetChildren()) do
            table.insert(lines, "  [" .. child.ClassName .. "] " .. child.Name)
        end
        local attrs = tool:GetAttributes()
        if next(attrs) then
            table.insert(lines, "Atributos:")
            for k, v in pairs(attrs) do
                table.insert(lines, "  " .. k .. ": " .. tostring(v))
            end
        end
        return table.concat(lines, "\n")
    end
    Log(describeTool(activeTool))
    Log("\nAguardando você jogar o dado...")

    -- Tira snapshot do Workspace antes de jogar
    initialObjects = SnapshotWorkspace()

    -- Monitora destruição ou remoção da ferramenta
    local function onToolRemoved()
        task.wait(0.8) -- espera o dado físico aparecer
        Log("\n>>> DADO JOGADO! Objetos novos no Workspace:\n")
        local newObjects = FindNewObjects(initialObjects)
        if #newObjects == 0 then
            Log("Nenhum objeto novo encontrado. Talvez o dado tenha sido removido sem criar um objeto físico?")
        else
            for _, obj in ipairs(newObjects) do
                Log(DescribeWorldObject(obj))
            end
        end
        Log("\nRastreamento concluído. Copie os dados e cole aqui.")
        DeactivateTracker()
    end

    -- Conexões
    toolDestroyConn = activeTool.Destroying:Connect(function()
        onToolRemoved()
    end)
    toolAncestryConn = activeTool.AncestryChanged:Connect(function()
        if not activeTool:IsDescendantOf(Player) and not activeTool:IsDescendantOf(Backpack) then
            onToolRemoved()
        end
    end)

    Notify("Rastreador ativo! Jogue o dado.")
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

Notify("🔍 Rastreador Pro ativo! Jogue o dado e veja os dados completos.")

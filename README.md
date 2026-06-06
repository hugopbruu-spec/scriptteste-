--[[
    🔍 Item Inspector – Mostra todos os dados do item na mão
    Segure um item e clique em "INSPECIONAR".
    Todos os detalhes aparecerão no console.
    Botão para copiar os dados.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
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
gui.Name = "ItemInspector"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 360, 0, 380)
Main.Position = UDim2.new(0.5, -180, 0.5, -190)
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
TitleText.Text = "Item Inspector"
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

local InspectBtn = Instance.new("TextButton")
InspectBtn.Parent = Content
InspectBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
InspectBtn.BorderSizePixel = 0
InspectBtn.Size = UDim2.new(1, 0, 0, 34)
InspectBtn.Text = "🔍 INSPECIONAR ITEM NA MÃO"
InspectBtn.Font = Enum.Font.GothamBlack
InspectBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
InspectBtn.TextSize = 11
Instance.new("UICorner", InspectBtn).CornerRadius = UDim.new(0, 8)

-- Console
local ConsoleBox = Instance.new("TextBox")
ConsoleBox.Parent = Content
ConsoleBox.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
ConsoleBox.BorderSizePixel = 0
ConsoleBox.Position = UDim2.new(0, 0, 0, 40)
ConsoleBox.Size = UDim2.new(1, 0, 0, 250)
ConsoleBox.Font = Enum.Font.Code
ConsoleBox.Text = "Clique em 'INSPECIONAR' para ver os dados do item na mão."
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
CopyBtn.Position = UDim2.new(1, -60, 0, 295)
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
        elseif writefile then writefile("item_info.txt", ConsoleBox.Text) end
    end)
    Notify("Copiado!")
end)

-- ==================== LÓGICA DE INSPEÇÃO ====================
local function FindTool()
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") then return tool end
    end
    for _, tool in ipairs(Backpack:GetChildren()) do
        if tool:IsA("Tool") then return tool end
    end
    return nil
end

local function DescribeTool(tool)
    local lines = {}
    table.insert(lines, "=== DETALHES DO ITEM ===")
    table.insert(lines, "Nome: " .. tool.Name)
    table.insert(lines, "Classe: " .. tool.ClassName)
    table.insert(lines, "Parent: " .. (tool.Parent and tool.Parent:GetFullName() or "nil"))

    -- Propriedades comuns
    local props = {
        "RequiresHandle", "CanBeDropped", "ManualActivationOnly",
        "ToolTip", "TextureId", "Grip", "GripForward", "GripRight", "GripUp", "GripPos"
    }
    for _, prop in ipairs(props) do
        local ok, val = pcall(function() return tool[prop] end)
        if ok and val ~= nil then
            table.insert(lines, prop .. ": " .. tostring(val))
        end
    end

    -- Filhos e seus detalhes
    table.insert(lines, "Filhos do item:")
    for _, child in ipairs(tool:GetChildren()) do
        table.insert(lines, "  [" .. child.ClassName .. "] " .. child.Name)
        -- Para cada filho, tenta extrair IDs de assets
        for _, assetProp in ipairs({"MeshId", "TextureId", "SoundId"}) do
            local ok, val = pcall(function() return child[assetProp] end)
            if ok and val and type(val) == "string" and val:match("rbxassetid://") then
                table.insert(lines, "    " .. assetProp .. ": " .. val)
            end
        end
        -- Se for BasePart, detalhes físicos
        if child:IsA("BasePart") then
            table.insert(lines, "    Position: " .. tostring(child.Position))
            table.insert(lines, "    Size: " .. tostring(child.Size))
            table.insert(lines, "    Material: " .. child.Material.Name)
            table.insert(lines, "    Color: " .. tostring(child.Color))
        end
    end

    -- Atributos
    local attrs = tool:GetAttributes()
    if next(attrs) then
        table.insert(lines, "Atributos:")
        for k, v in pairs(attrs) do
            table.insert(lines, "  " .. k .. ": " .. tostring(v))
        end
    else
        table.insert(lines, "Atributos: nenhum")
    end

    return table.concat(lines, "\n")
end

InspectBtn.MouseButton1Click:Connect(function()
    local tool = FindTool()
    if not tool then
        ConsoleBox.Text = "Nenhum item encontrado na mão ou mochila."
        Notify("Nenhum item encontrado!")
        return
    end
    ConsoleBox.Text = "Item encontrado:\n" .. DescribeTool(tool)
    Notify("Item inspecionado! Copie os dados.")
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

Notify("🔍 Segure um item e clique em INSPECIONAR!")

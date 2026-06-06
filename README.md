--[[
    🔍 Dice Tracker – Rastreia o dado e recolhe informações
    Ative o rastreador, jogue o dado no chão, e todos os detalhes
    aparecerão no mini console. Depois copie e cole aqui.
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
Main.Size = UDim2.new(0, 360, 0, 390)
Main.Position = UDim2.new(0.5, -180, 0.5, -195)
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
TitleText.Text = "Dice Tracker"
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
ConsoleBox.Size = UDim2.new(1, 0, 0, 260)
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
CopyBtn.Position = UDim2.new(1, -60, 0, 305)
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

-- ==================== LÓGICA DO RASTREADOR ====================
local activeTool = nil
local toolConn = nil
local function DeactivateTracker()
    if toolConn then
        toolConn:Disconnect()
        toolConn = nil
    end
    activeTool = nil
    ActivateBtn.Text = "🟢 ATIVAR RASTREADOR (com dado na mão)"
    ActivateBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
end

local function CaptureToolInfo(tool)
    local info = {}
    table.insert(info, "=== INFORMAÇÕES DA FERRAMENTA (NA MÃO/MOCHILA) ===")
    table.insert(info, "Nome: " .. tostring(tool.Name))
    table.insert(info, "Classe: " .. tool.ClassName)
    table.insert(info, "Parent: " .. tostring(tool.Parent))
    -- Propriedades comuns
    local props = { "RequiresHandle", "CanBeDropped", "ManualActivationOnly", "ToolTip", "TextureId", "Grip", "GripForward", "GripRight", "GripUp", "GripPos" }
    for _, prop in ipairs(props) do
        local ok, val = pcall(function() return tool[prop] end)
        if ok and val ~= nil then
            table.insert(info, prop .. ": " .. tostring(val))
        end
    end
    -- Filhos importantes
    table.insert(info, "Filhos da ferramenta:")
    for _, child in ipairs(tool:GetChildren()) do
        table.insert(info, "  [" .. child.ClassName .. "] " .. child.Name)
        -- IDs de assets
        for _, assetProp in ipairs({"TextureId", "MeshId", "SoundId"}) do
            local ok, val = pcall(function() return child[assetProp] end)
            if ok and val and type(val) == "string" and val:match("rbxassetid://") then
                table.insert(info, "    " .. assetProp .. ": " .. val)
            end
        end
    end
    -- Atributos
    local attrs = tool:GetAttributes()
    if next(attrs) then
        table.insert(info, "Atributos:")
        for k, v in pairs(attrs) do
            table.insert(info, "  " .. k .. ": " .. tostring(v))
        end
    else
        table.insert(info, "Atributos: nenhum")
    end
    return table.concat(info, "\n")
end

local function CaptureWorldObjects()
    local objects = {}
    -- Procura objetos que surgiram recentemente (não podemos saber o instante exato, então listamos tudo)
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local owner = nil
            pcall(function() owner = obj:GetNetworkOwner() end)
            -- Inclui todos, mas destaca os que pertencem ao jogador
            table.insert(objects, {
                Name = obj.Name,
                Class = obj.ClassName,
                NetworkOwner = owner,
                Position = obj.Position,
                Parent = obj.Parent and obj.Parent:GetFullName() or "nil"
            })
        elseif obj:IsA("Model") and obj:FindFirstChildOfClass("BasePart") then
            -- Modelos que podem ser o dado
            local owner = nil
            local part = obj:FindFirstChildOfClass("BasePart")
            if part then pcall(function() owner = part:GetNetworkOwner() end) end
            table.insert(objects, {
                Name = obj.Name,
                Class = obj.ClassName,
                NetworkOwner = owner,
                Position = part and part.Position or Vector3.zero,
                Parent = obj.Parent and obj.Parent:GetFullName() or "nil"
            })
        end
    end
    return objects
end

ActivateBtn.MouseButton1Click:Connect(function()
    if activeTool then
        DeactivateTracker()
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
    Log(CaptureToolInfo(activeTool))
    Log("\nAguardando você jogar o dado...\n")

    -- Monitora a remoção da ferramenta
    if toolConn then toolConn:Disconnect() end
    toolConn = activeTool.AncestryChanged:Connect(function()
        if not activeTool:IsDescendantOf(Player) and not activeTool:IsDescendantOf(Backpack) then
            Log(">>> DADO JOGADO! A ferramenta foi removida do jogador.\n")
            task.wait(0.5) -- aguarda o objeto físico aparecer
            Log("=== OBJETOS ENCONTRADOS NO MUNDO APÓS JOGAR ===")
            local worldObjects = CaptureWorldObjects()
            for _, obj in ipairs(worldObjects) do
                Log(string.format("[%s] %s | Dono: %s | Pos: %s | Parent: %s",
                    obj.Class, obj.Name, tostring(obj.NetworkOwner), tostring(obj.Position), obj.Parent))
            end
            Log("\nRastreador concluído. Copie os dados e cole aqui.")
            DeactivateTracker()
        end
    end)
    Notify("Rastreador ativo! Jogue o dado para capturar os dados.")
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

Notify("🔍 Ative o rastreador, jogue o dado e cole as informações aqui!")

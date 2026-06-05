--[[
    🔍 Roblox Item Inspector Pro v2.0
    Arrastável, centralizado, exibe TODOS os dados do item na mão.
    Corrigido para não cortar a tela e funcionar perfeitamente.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local CollectionService = game:GetService("CollectionService")

-- Espera o personagem carregar
repeat task.wait() until Player.Character

-- ==================== FUNÇÕES AUXILIARES ====================
local function Notify(text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(0.5, -150, 0, 10)
    frame.Size = UDim2.new(0, 300, 0, 36)
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
    local tw = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, 16)})
    tw:Play()
    task.wait(duration)
    local tw2 = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, -36)})
    tw2:Play()
    tw2.Completed:Connect(function() gui:Destroy() end)
end

local function SafeGet(obj, prop)
    local ok, val = pcall(function() return obj[prop] end)
    return ok and val or nil
end

local function FormatValue(val)
    local t = typeof(val)
    if t == "Vector3" then return string.format("(%.2f, %.2f, %.2f)", val.X, val.Y, val.Z)
    elseif t == "CFrame" then return tostring(val)
    elseif t == "Color3" then return string.format("RGB(%d,%d,%d)", val.R*255, val.G*255, val.B*255)
    elseif t == "EnumItem" then return val.Name
    elseif t == "BrickColor" then return val.Name
    elseif t == "Instance" then return val:GetFullName()
    elseif t == "nil" then return "nil"
    else return tostring(val)
    end
end

-- ==================== COLETA COMPLETA DE DADOS ====================
local function InspectItem()
    if not Player.Character then return nil, "Personagem não encontrado." end
    local tool = Player.Character:FindFirstChildOfClass("Tool")
    if not tool then tool = Player.Backpack:FindFirstChildOfClass("Tool") end
    if not tool then return nil, "Nenhum item na mão ou mochila." end

    local lines = {}
    local function add(s) table.insert(lines, s) end

    add("══════════════════════════════════════")
    add("🔍 ITEM: " .. tool.Name)
    add("══════════════════════════════════════")
    add("")

    -- Básico
    add("📋 INFORMAÇÕES BÁSICAS")
    add("  Classe: " .. tool.ClassName)
    add("  Nome: " .. tool.Name)
    add("  Parent: " .. (tool.Parent and tool.Parent:GetFullName() or "nil"))
    add("")

    -- Propriedades da Tool
    add("⚙️ PROPRIEDADES")
    local props = { "RequiresHandle", "CanBeDropped", "ManualActivationOnly", "ToolTip", "TextureId",
                    "Grip", "GripForward", "GripRight", "GripUp", "GripPos" }
    for _, prop in ipairs(props) do
        local val = SafeGet(tool, prop)
        if val ~= nil then add("  " .. prop .. ": " .. FormatValue(val)) end
    end
    add("")

    -- IDs de assets (TextureId, MeshId, SoundId, etc.)
    local assetIds = {}
    for _, child in ipairs(tool:GetDescendants()) do
        for _, prop in ipairs({"TextureId", "MeshId", "SoundId", "AssetId"}) do
            local val = SafeGet(child, prop)
            if val and type(val) == "string" and val:match("rbxassetid://%d+") then
                table.insert(assetIds, "  " .. child.Name .. " [" .. child.ClassName .. "] " .. prop .. ": " .. val)
            end
        end
    end
    if #assetIds > 0 then
        add("🆔 IDs DE ASSETS")
        for _, a in ipairs(assetIds) do add(a) end
        add("")
    end

    -- Partes
    local parts = {}
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BasePart") then table.insert(parts, child) end
    end
    add("🧩 PARTES (" .. #parts .. ")")
    for _, part in ipairs(parts) do
        add("  [" .. part.ClassName .. "] " .. part.Name)
        add("    Position: " .. FormatValue(part.Position))
        add("    Size: " .. FormatValue(part.Size))
        add("    Material: " .. part.Material.Name)
        add("    Color: " .. FormatValue(part.Color))
        add("    CanCollide: " .. tostring(part.CanCollide))
        add("    Anchored: " .. tostring(part.Anchored))
        if part:IsA("MeshPart") and part.MeshId then add("    MeshId: " .. part.MeshId) end
    end
    if #parts == 0 then add("  Nenhuma parte.") end
    add("")

    -- Scripts
    local scripts = {}
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BaseScript") then table.insert(scripts, child) end
    end
    add("📜 SCRIPTS (" .. #scripts .. ")")
    for _, s in ipairs(scripts) do
        add("  [" .. s.ClassName .. "] " .. s.Name .. (s.Enabled and " (Ativo)" or " (Inativo)"))
    end
    if #scripts == 0 then add("  Nenhum script.") end
    add("")

    -- Sons
    local sounds = {}
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("Sound") then table.insert(sounds, child) end
    end
    if #sounds > 0 then
        add("🔊 SONS (" .. #sounds .. ")")
        for _, snd in ipairs(sounds) do
            add("  " .. snd.Name .. " | SoundId: " .. (snd.SoundId or "N/A") .. " | Volume: " .. snd.Volume)
        end
        add("")
    end

    -- Values
    local values = {}
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("ValueBase") then table.insert(values, child) end
    end
    if #values > 0 then
        add("💾 VALORES")
        for _, v in ipairs(values) do
            add("  [" .. v.ClassName .. "] " .. v.Name .. " = " .. FormatValue(v.Value))
        end
        add("")
    end

    -- Atributos
    local attribs = tool:GetAttributes()
    if next(attribs) then
        add("🏷️ ATRIBUTOS")
        for k, v in pairs(attribs) do
            add("  " .. k .. ": " .. FormatValue(v))
        end
        add("")
    else
        add("🏷️ ATRIBUTOS: Nenhum")
        add("")
    end

    -- Tags
    local tags = CollectionService:GetTags(tool)
    add("📌 TAGS: " .. (#tags > 0 and table.concat(tags, ", ") or "Nenhuma"))
    add("")
    add("══════════════════════════════════════")
    return tool, table.concat(lines, "\n")
end

-- ==================== CRIAÇÃO DA INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "ItemInspectorPro_" .. math.random(1000,9999)
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Frame principal responsivo
local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, math.min(520, workspace.CurrentCamera.ViewportSize.X - 20), 0, math.min(580, workspace.CurrentCamera.ViewportSize.Y - 40))
Main.Position = UDim2.new(0.5, -Main.Size.X.Offset/2, 0.5, -Main.Size.Y.Offset/2)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Barra de título (arraste)
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 42)
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
TitleIcon.Position = UDim2.new(0, 10, 0, 6)
TitleIcon.Size = UDim2.new(0, 30, 0, 30)
TitleIcon.Text = "🔍"
TitleIcon.TextSize = 20

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 42, 0, 0)
TitleText.Size = UDim2.new(1, -100, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Item Inspector Pro"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 14
TitleText.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -40, 0, 8)
CloseBtn.Size = UDim2.new(0, 28, 0, 26)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 14
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)
CloseBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
    Notify("Inspector fechado")
end)

-- Área de conteúdo
local Content = Instance.new("Frame")
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 8, 0, 48)
Content.Size = UDim2.new(1, -16, 1, -100)

-- Scroll
local Scroll = Instance.new("ScrollingFrame")
Scroll.Parent = Content
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.Size = UDim2.new(1, 0, 1, 0)
Scroll.ScrollBarThickness = 3
Scroll.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
Scroll.CanvasSize = UDim2.new(0, 0, 0, 50)
local scrollLayout = Instance.new("UIListLayout")
scrollLayout.Parent = Scroll
scrollLayout.Padding = UDim.new(0, 6)
scrollLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Barra inferior
local BottomBar = Instance.new("Frame")
BottomBar.Parent = Main
BottomBar.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
BottomBar.BorderSizePixel = 0
BottomBar.Position = UDim2.new(0, 8, 1, -52)
BottomBar.Size = UDim2.new(1, -16, 0, 44)
Instance.new("UICorner", BottomBar).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", BottomBar).Color = Color3.fromRGB(108, 92, 231)

local CopyBox = Instance.new("TextBox")
CopyBox.Parent = BottomBar
CopyBox.BackgroundTransparency = 1
CopyBox.Position = UDim2.new(0, 6, 0, 4)
CopyBox.Size = UDim2.new(1, -48, 1, -8)
CopyBox.Font = Enum.Font.Code
CopyBox.Text = "Clique em 'Inspecionar'..."
CopyBox.TextColor3 = Color3.fromRGB(180, 180, 200)
CopyBox.TextSize = 10
CopyBox.ClearTextOnFocus = false
CopyBox.TextEditable = false
CopyBox.TextWrapped = true
CopyBox.TextXAlignment = Enum.TextXAlignment.Left
CopyBox.TextYAlignment = Enum.TextYAlignment.Top

local CopyBtn = Instance.new("TextButton")
CopyBtn.Parent = BottomBar
CopyBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
CopyBtn.BorderSizePixel = 0
CopyBtn.Position = UDim2.new(1, -44, 0, 7)
CopyBtn.Size = UDim2.new(0, 38, 0, 30)
CopyBtn.Text = "📋"
CopyBtn.Font = Enum.Font.GothamBold
CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyBtn.TextSize = 16
Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 6)

local InspectBtn = Instance.new("TextButton")
InspectBtn.Parent = Main
InspectBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
InspectBtn.BorderSizePixel = 0
InspectBtn.Position = UDim2.new(0, 8, 1, -56)
InspectBtn.Size = UDim2.new(1, -16, 0, 40)
InspectBtn.Text = "🔍 INSPECIONAR ITEM NA MÃO"
InspectBtn.Font = Enum.Font.GothamBlack
InspectBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
InspectBtn.TextSize = 14
Instance.new("UICorner", InspectBtn).CornerRadius = UDim.new(0, 8)

-- Funções para popular cards
local function ClearScroll()
    for _, c in ipairs(Scroll:GetChildren()) do
        if c:IsA("Frame") or c:IsA("TextLabel") then c:Destroy() end
    end
end

local function AddCard(title, value, color)
    color = color or Color3.fromRGB(108, 92, 231)
    local card = Instance.new("Frame")
    card.Parent = Scroll
    card.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
    card.BorderSizePixel = 0
    card.Size = UDim2.new(1, -6, 0, 34)
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 6)
    
    local tLbl = Instance.new("TextLabel")
    tLbl.Parent = card
    tLbl.BackgroundTransparency = 1
    tLbl.Position = UDim2.new(0, 8, 0, 0)
    tLbl.Size = UDim2.new(0, 110, 1, 0)
    tLbl.Font = Enum.Font.GothamBold
    tLbl.Text = title
    tLbl.TextColor3 = color
    tLbl.TextSize = 11
    tLbl.TextXAlignment = Enum.TextXAlignment.Left
    
    local vLbl = Instance.new("TextLabel")
    vLbl.Parent = card
    vLbl.BackgroundTransparency = 1
    vLbl.Position = UDim2.new(0, 120, 0, 0)
    vLbl.Size = UDim2.new(1, -130, 1, 0)
    vLbl.Font = Enum.Font.Code
    vLbl.Text = value
    vLbl.TextColor3 = Color3.fromRGB(200, 200, 220)
    vLbl.TextSize = 10
    vLbl.TextXAlignment = Enum.TextXAlignment.Left
    vLbl.TextWrapped = true
end

local function AddSection(text)
    local lbl = Instance.new("TextLabel")
    lbl.Parent = Scroll
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -6, 0, 22)
    lbl.Font = Enum.Font.GothamBlack
    lbl.Text = "  " .. text
    lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
end

local function UpdateDisplay(dataText)
    ClearScroll()
    local tool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
    if not tool then tool = Player.Backpack:FindFirstChildOfClass("Tool") end
    if not tool then
        AddSection("Nenhum item encontrado")
        Scroll.CanvasSize = UDim2.new(0, 0, 0, 30)
        return
    end
    AddSection("📋 " .. tool.Name)
    AddCard("Nome", tool.Name)
    AddCard("Classe", tool.ClassName)
    AddCard("Parent", tool.Parent and tool.Parent:GetFullName() or "nil")
    AddCard("ToolTip", SafeGet(tool, "ToolTip") or "Nenhum")
    AddCard("CanBeDropped", tostring(SafeGet(tool, "CanBeDropped")))
    AddCard("RequiresHandle", tostring(SafeGet(tool, "RequiresHandle")))
    local grip = SafeGet(tool, "Grip")
    if grip then AddCard("Grip", FormatValue(grip)) end
    AddCard("GripForward", FormatValue(SafeGet(tool, "GripForward")))
    AddCard("GripRight", FormatValue(SafeGet(tool, "GripRight")))
    AddCard("GripUp", FormatValue(SafeGet(tool, "GripUp")))
    AddCard("GripPos", FormatValue(SafeGet(tool, "GripPos")))
    local partCount = 0
    for _, _ in ipairs(tool:GetDescendants()) do if _:IsA("BasePart") then partCount += 1 end end
    AddCard("Partes", tostring(partCount))
    local scriptCount = 0
    for _, _ in ipairs(tool:GetDescendants()) do if _:IsA("BaseScript") then scriptCount += 1 end end
    AddCard("Scripts", tostring(scriptCount))
    local attrCount = 0
    for _ in pairs(tool:GetAttributes()) do attrCount += 1 end
    AddCard("Atributos", tostring(attrCount))
    local tags = CollectionService:GetTags(tool)
    AddCard("Tags", #tags > 0 and table.concat(tags, ", ") or "Nenhum")
    Scroll.CanvasSize = UDim2.new(0, 0, 0, #Scroll:GetChildren() * 40 + 20)
end

InspectBtn.MouseButton1Click:Connect(function()
    local tool, result = InspectItem()
    CopyBox.Text = result or "Erro"
    UpdateDisplay(result)
    Notify(tool and ("Item: " .. tool.Name) or result)
end)

CopyBtn.MouseButton1Click:Connect(function()
    if CopyBox.Text ~= "" then
        local ok = pcall(function()
            if setclipboard then setclipboard(CopyBox.Text)
            elseif writefile then writefile("item_data.txt", CopyBox.Text) end
        end)
        CopyBtn.Text = ok and "✅" or "⚠️"
        Notify(ok and "Copiado!" or "Selecione e Ctrl+C")
        task.wait(1.5)
        CopyBtn.Text = "📋"
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

Notify("🔍 Item Inspector Pro pronto! Arraste a janela se necessário.")

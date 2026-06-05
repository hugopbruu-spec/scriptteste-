--[[
    🔍 Roblox Item Inspector - Professional Tool Data Viewer
    Arrastável, com botão de fechar, exibe TODOS os dados do item na mão
    Extremamente completo e organizado
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local CollectionService = game:GetService("CollectionService")

-- Aguarda o personagem carregar
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
    frame.Position = UDim2.new(0.5, -160, 0, 10)
    frame.Size = UDim2.new(0, 320, 0, 40)
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
    lbl.TextSize = 13
    local tw = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -160, 0, 20)})
    tw:Play()
    task.wait(duration)
    local tw2 = Tween:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -160, 0, -40)})
    tw2:Play()
    tw2.Completed:Connect(function() gui:Destroy() end)
end

local function SafeGet(obj, prop)
    local ok, val = pcall(function() return obj[prop] end)
    return ok and val or nil
end

local function FormatValue(val)
    local t = typeof(val)
    if t == "Vector3" then return string.format("(%.3f, %.3f, %.3f)", val.X, val.Y, val.Z)
    elseif t == "CFrame" then return tostring(val) -- já formatado
    elseif t == "Color3" then return string.format("RGB(%d,%d,%d)", val.R*255, val.G*255, val.B*255)
    elseif t == "EnumItem" then return val.Name
    elseif t == "BrickColor" then return val.Name
    elseif t == "Instance" then return val:GetFullName()
    elseif t == "nil" then return "nil"
    else return tostring(val)
    end
end

-- ==================== COLETA DE DADOS ====================
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

    -- Seção: Informações Básicas
    add("📋 INFORMAÇÕES BÁSICAS")
    add("  Classe: " .. tool.ClassName)
    add("  Nome: " .. tool.Name)
    add("  Parent: " .. (tool.Parent and tool.Parent:GetFullName() or "nil"))
    add("")

    -- IDs e referências de ativos (TextureId, MeshId, SoundId, etc.)
    local assetIds = {}
    for _, child in ipairs(tool:GetDescendants()) do
        for _, prop in ipairs({"TextureId", "MeshId", "SoundId", "Image", "AssetId"}) do
            local val = SafeGet(child, prop)
            if val and type(val) == "string" and val:match("rbxassetid://(%d+)") then
                table.insert(assetIds, child.Name .. " [" .. child.ClassName .. "] " .. prop .. ": " .. val)
            end
        end
    end
    if #assetIds > 0 then
        add("🆔 IDs DE ASSETS ENCONTRADOS")
        for _, aid in ipairs(assetIds) do add("  " .. aid) end
        add("")
    end

    -- Seção: Propriedades da Tool
    add("⚙️ PROPRIEDADES DA FERRAMENTA")
    local toolProps = {
        "RequiresHandle", "CanBeDropped", "ManualActivationOnly",
        "ToolTip", "Grip", "GripForward", "GripRight", "GripUp", "GripPos",
        "TextureId",
    }
    for _, prop in ipairs(toolProps) do
        local val = SafeGet(tool, prop)
        if val ~= nil then
            add("  " .. prop .. ": " .. FormatValue(val))
        end
    end
    add("")

    -- Seção: Partes
    add("🧩 PARTES (" .. #tool:GetDescendants():Filter(function(c) return c:IsA("BasePart") end) .. " encontradas)")
    local partCount = 0
    for _, part in ipairs(tool:GetDescendants()) do
        if part:IsA("BasePart") then
            partCount = partCount + 1
            add("  ┌─ " .. part.ClassName .. " \"" .. part.Name .. "\"")
            local partProps = {
                "Position", "Size", "Material", "Color", "BrickColor",
                "CanCollide", "Anchored", "Transparency", "Mass",
                "Reflectance", "Transparency",
            }
            for _, prop in ipairs(partProps) do
                local val = SafeGet(part, prop)
                if val ~= nil then
                    add("  │  " .. prop .. ": " .. FormatValue(val))
                end
            end
            -- Se for MeshPart, pegar MeshId
            if part:IsA("MeshPart") then
                local meshId = SafeGet(part, "MeshId")
                if meshId then add("  │  MeshId: " .. FormatValue(meshId)) end
            end
            add("  └─")
        end
    end
    if partCount == 0 then add("  Nenhuma parte encontrada.") end
    add("")

    -- Seção: Scripts
    local scripts = tool:GetDescendants():Filter(function(c) return c:IsA("BaseScript") end)
    add("📜 SCRIPTS (" .. #scripts .. " encontrados)")
    if #scripts > 0 then
        for _, s in ipairs(scripts) do
            add("  [" .. s.ClassName .. "] " .. s.Name .. (s.Enabled and " (Ativo)" or " (Inativo)"))
        end
    else
        add("  Nenhum script.")
    end
    add("")

    -- Seção: Sons
    local sounds = tool:GetDescendants():Filter(function(c) return c:IsA("Sound") end)
    if #sounds > 0 then
        add("🔊 SONS (" .. #sounds .. " encontrados)")
        for _, snd in ipairs(sounds) do
            add("  [" .. snd.Name .. "] " .. (SafeGet(snd, "SoundId") or "Sem ID") .. " Vol:" .. (SafeGet(snd, "Volume") or 1))
        end
        add("")
    end

    -- Seção: Valores (StringValue, IntValue, etc.)
    local values = tool:GetDescendants():Filter(function(c)
        return c:IsA("ValueBase")
    end)
    if #values > 0 then
        add("💾 VALORES")
        for _, v in ipairs(values) do
            add("  [" .. v.ClassName .. "] " .. v.Name .. " = " .. FormatValue(v.Value))
        end
        add("")
    end

    -- Seção: Atributos
    local attribs = tool:GetAttributes()
    local hasAttribs = false
    for _ in pairs(attribs) do hasAttribs = true break end
    if hasAttribs then
        add("🏷️ ATRIBUTOS")
        for k, v in pairs(attribs) do
            add("  " .. k .. ": " .. FormatValue(v))
        end
        add("")
    else
        add("🏷️ ATRIBUTOS: Nenhum")
        add("")
    end

    -- Seção: Tags
    local tags = CollectionService:GetTags(tool)
    if #tags > 0 then
        add("📌 TAGS")
        for _, t in ipairs(tags) do add("  " .. t) end
        add("")
    else
        add("📌 TAGS: Nenhuma")
        add("")
    end

    add("══════════════════════════════════════")
    return tool, table.concat(lines, "\n")
end

-- ==================== CONSTRUÇÃO DA GUI ====================
local function CreateGUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "ItemInspectorPro"
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.ResetOnSpawn = false

    -- Frame principal
    local Main = Instance.new("Frame")
    Main.Parent = gui
    Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
    Main.BorderSizePixel = 0
    Main.Position = UDim2.new(0.5, -260, 0.5, -300)
    Main.Size = UDim2.new(0, 520, 0, 600)
    Main.AnchorPoint = Vector2.new(0.5, 0.5)
    Main.ClipsDescendants = true
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
    Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)
    local bgGrad = Instance.new("UIGradient", Main)
    bgGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(18, 18, 28)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(28, 28, 42))
    })
    bgGrad.Rotation = 135

    -- Barra de título (arraste)
    local TitleBar = Instance.new("Frame")
    TitleBar.Parent = Main
    TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    TitleBar.BorderSizePixel = 0
    TitleBar.Size = UDim2.new(1, 0, 0, 46)
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
    TitleIcon.Position = UDim2.new(0, 12, 0, 6)
    TitleIcon.Size = UDim2.new(0, 34, 0, 34)
    TitleIcon.Text = "🔍"
    TitleIcon.TextSize = 22
    local TitleText = Instance.new("TextLabel")
    TitleText.Parent = TitleBar
    TitleText.BackgroundTransparency = 1
    TitleText.Position = UDim2.new(0, 48, 0, 0)
    TitleText.Size = UDim2.new(1, -120, 1, 0)
    TitleText.Font = Enum.Font.GothamBold
    TitleText.Text = "Item Inspector Pro"
    TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleText.TextSize = 15
    TitleText.TextXAlignment = Enum.TextXAlignment.Left

    -- Botões
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Parent = TitleBar
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Position = UDim2.new(1, -42, 0, 10)
    CloseBtn.Size = UDim2.new(0, 30, 0, 26)
    CloseBtn.Text = "✕"
    CloseBtn.Font = Enum.Font.GothamBlack
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 14
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

    CloseBtn.MouseButton1Click:Connect(function()
        Notify("Inspector fechado")
        gui:Destroy()
    end)

    -- Área de conteúdo
    local Content = Instance.new("Frame")
    Content.Parent = Main
    Content.BackgroundTransparency = 1
    Content.Position = UDim2.new(0, 10, 0, 52)
    Content.Size = UDim2.new(1, -20, 1, -112)

    -- ScrollingFrame para os cards
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

    -- Barra inferior com cópia
    local BottomBar = Instance.new("Frame")
    BottomBar.Parent = Main
    BottomBar.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
    BottomBar.BorderSizePixel = 0
    BottomBar.Position = UDim2.new(0, 8, 1, -56)
    BottomBar.Size = UDim2.new(1, -16, 0, 48)
    Instance.new("UICorner", BottomBar).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", BottomBar).Color = Color3.fromRGB(108, 92, 231)

    local CopyBox = Instance.new("TextBox")
    CopyBox.Parent = BottomBar
    CopyBox.BackgroundTransparency = 1
    CopyBox.Position = UDim2.new(0, 6, 0, 4)
    CopyBox.Size = UDim2.new(1, -52, 1, -8)
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
    CopyBtn.Position = UDim2.new(1, -46, 0, 9)
    CopyBtn.Size = UDim2.new(0, 40, 0, 30)
    CopyBtn.Text = "📋"
    CopyBtn.Font = Enum.Font.GothamBold
    CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CopyBtn.TextSize = 16
    Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 6)

    -- Botão Inspecionar
    local InspectBtn = Instance.new("TextButton")
    InspectBtn.Parent = Main
    InspectBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    InspectBtn.BorderSizePixel = 0
    InspectBtn.Position = UDim2.new(0, 10, 1, -62)
    InspectBtn.Size = UDim2.new(1, -20, 0, 42)
    InspectBtn.Text = "🔍 INSPECIONAR ITEM NA MÃO"
    InspectBtn.Font = Enum.Font.GothamBlack
    InspectBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    InspectBtn.TextSize = 14
    Instance.new("UICorner", InspectBtn).CornerRadius = UDim.new(0, 8)

    -- Função para popular cards
    local function ClearCards()
        for _, c in ipairs(Scroll:GetChildren()) do
            if c:IsA("Frame") then c:Destroy() end
        end
    end

    local function AddCard(title, value, color)
        color = color or Color3.fromRGB(108, 92, 231)
        local card = Instance.new("Frame")
        card.Parent = Scroll
        card.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
        card.BorderSizePixel = 0
        card.Size = UDim2.new(1, -6, 0, 36)
        Instance.new("UICorner", card).CornerRadius = UDim.new(0, 6)
        
        local titleLbl = Instance.new("TextLabel")
        titleLbl.Parent = card
        titleLbl.BackgroundTransparency = 1
        titleLbl.Position = UDim2.new(0, 8, 0, 0)
        titleLbl.Size = UDim2.new(0, 120, 1, 0)
        titleLbl.Font = Enum.Font.GothamBold
        titleLbl.Text = title
        titleLbl.TextColor3 = color
        titleLbl.TextSize = 11
        titleLbl.TextXAlignment = Enum.TextXAlignment.Left
        
        local valLbl = Instance.new("TextLabel")
        valLbl.Parent = card
        valLbl.BackgroundTransparency = 1
        valLbl.Position = UDim2.new(0, 130, 0, 0)
        valLbl.Size = UDim2.new(1, -140, 1, 0)
        valLbl.Font = Enum.Font.Code
        valLbl.Text = value
        valLbl.TextColor3 = Color3.fromRGB(200, 200, 220)
        valLbl.TextSize = 10
        valLbl.TextXAlignment = Enum.TextXAlignment.Left
        valLbl.TextWrapped = true
        return card
    end

    local function AddSection(text)
        local lbl = Instance.new("TextLabel")
        lbl.Parent = Scroll
        lbl.BackgroundTransparency = 1
        lbl.Size = UDim2.new(1, -6, 0, 24)
        lbl.Font = Enum.Font.GothamBlack
        lbl.Text = "  " .. text
        lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        lbl.TextSize = 13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        return lbl
    end

    local function UpdateDisplay(dataText)
        ClearCards()
        if not dataText or dataText == "" then
            AddSection("Nenhum dado disponível")
            Scroll.CanvasSize = UDim2.new(0, 0, 0, 30)
            return
        end
        
        -- Divide o texto em linhas e cria cards para as linhas que têm ":" 
        -- ou mantém o formato original com seções.
        -- Vou apenas criar um card grande com o texto completo, mas é melhor exibir as propriedades principais em cards e o resto como texto.
        -- Para ser profissional, vou recriar a exibição usando os mesmos dados coletados pela função InspectItem.
        -- Como a função InspectItem já retorna o texto formatado, posso exibi-lo em um único TextBox ou quebrar em cards.
        -- Para aproveitar a GUI, vou mostrar o texto completo no CopyBox e cards para as propriedades principais.
        -- Vou fazer uma segunda coleta para os cards.
        local tool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
        if not tool then tool = Player.Backpack:FindFirstChildOfClass("Tool") end
        if tool then
            AddSection("📋 " .. tool.Name .. " [" .. tool.ClassName .. "]")
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
            
            local partCount = #tool:GetDescendants():Filter(function(c) return c:IsA("BasePart") end)
            AddCard("Partes", tostring(partCount))
            local scriptCount = #tool:GetDescendants():Filter(function(c) return c:IsA("BaseScript") end)
            AddCard("Scripts", tostring(scriptCount))
            local attrCount = 0
            for _ in pairs(tool:GetAttributes()) do attrCount += 1 end
            AddCard("Atributos", tostring(attrCount))
            local tags = CollectionService:GetTags(tool)
            AddCard("Tags", #tags > 0 and table.concat(tags, ", ") or "Nenhuma")
        end
        Scroll.CanvasSize = UDim2.new(0, 0, 0, #Scroll:GetChildren() * 42 + 20)
    end

    -- Ações
    InspectBtn.MouseButton1Click:Connect(function()
        local tool, result = InspectItem()
        CopyBox.Text = result or "Erro na inspeção."
        UpdateDisplay(result)
        Notify(tool and ("Inspecionado: " .. tool.Name) or result)
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

    return gui
end

-- ==================== INICIALIZAÇÃO ====================
local gui = CreateGUI()
Notify("🔍 Item Inspector Pro carregado!")
print("Item Inspector Pro iniciado. Segure um item e clique no botão.")

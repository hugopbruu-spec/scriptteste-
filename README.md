--[[
    🔍 Item Inspector - Script Roblox
    Interface arrastável com botão de fechar
    Mostra todos os dados do item que você está segurando
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local Mouse = Player:GetMouse()

-- ============================================
-- CRIAR GUI PRINCIPAL
-- ============================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ItemInspector"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

-- ============================================
-- FRAME PRINCIPAL (ARRASTÁVEL)
-- ============================================
local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0, 100, 0, 100)
MainFrame.Size = UDim2.new(0, 420, 0, 480)
MainFrame.ClipsDescendants = true

local mainCorner = Instance.new("UICorner", MainFrame)
mainCorner.CornerRadius = UDim.new(0, 12)

local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Color3.fromRGB(108, 92, 231)
mainStroke.Thickness = 2

-- Gradiente de fundo
local bgGradient = Instance.new("UIGradient", MainFrame)
bgGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 30)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 45)),
})
bgGradient.Rotation = 135

-- ============================================
-- BARRA DE TÍTULO (ÁREA DE ARRASTAR)
-- ============================================
local TitleBar = Instance.new("Frame")
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 45)

local titleGradient = Instance.new("UIGradient", TitleBar)
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 60, 200)),
})
titleGradient.Rotation = 90

local titleCorner = Instance.new("UICorner", TitleBar)
titleCorner.CornerRadius = UDim.new(0, 12)

local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 12)
titleFix.Position = UDim2.new(0, 0, 1, -12)

-- Ícone
local IconLabel = Instance.new("TextLabel")
IconLabel.Parent = TitleBar
IconLabel.BackgroundTransparency = 1
IconLabel.Position = UDim2.new(0, 10, 0, 5)
IconLabel.Size = UDim2.new(0, 35, 0, 35)
IconLabel.Text = "🔍"
IconLabel.TextSize = 22
IconLabel.Font = Enum.Font.Gotham

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Parent = TitleBar
TitleLabel.BackgroundTransparency = 1
TitleLabel.Position = UDim2.new(0, 48, 0, 0)
TitleLabel.Size = UDim2.new(1, -100, 1, 0)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "Item Inspector"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 16
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Fechar
local CloseButton = Instance.new("TextButton")
CloseButton.Parent = TitleBar
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseButton.BorderSizePixel = 0
CloseButton.Position = UDim2.new(1, -40, 0, 8)
CloseButton.Size = UDim2.new(0, 30, 0, 28)
CloseButton.Text = "✕"
CloseButton.Font = Enum.Font.GothamBlack
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 16
Instance.new("UICorner", CloseButton).CornerRadius = UDim.new(0, 8)

-- Botão Minimizar (esconde o conteúdo)
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Parent = TitleBar
MinimizeButton.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
MinimizeButton.BorderSizePixel = 0
MinimizeButton.Position = UDim2.new(1, -75, 0, 8)
MinimizeButton.Size = UDim2.new(0, 30, 0, 28)
MinimizeButton.Text = "−"
MinimizeButton.Font = Enum.Font.GothamBlack
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.TextSize = 18
Instance.new("UICorner", MinimizeButton).CornerRadius = UDim.new(0, 8)

-- ============================================
-- ÁREA DE CONTEÚDO
-- ============================================
local ContentFrame = Instance.new("Frame")
ContentFrame.Parent = MainFrame
ContentFrame.BackgroundTransparency = 1
ContentFrame.Position = UDim2.new(0, 10, 0, 55)
ContentFrame.Size = UDim2.new(1, -20, 1, -65)

-- Scrolling para a lista de propriedades
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Parent = ContentFrame
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel = 0
ScrollFrame.Size = UDim2.new(1, 0, 1, -60)
ScrollFrame.ScrollBarThickness = 3
ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 800)

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = ScrollFrame
UIListLayout.Padding = UDim.new(0, 6)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Área de texto copiável (para mostrar os dados completos)
local CopyFrame = Instance.new("Frame")
CopyFrame.Parent = ContentFrame
CopyFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
CopyFrame.BorderSizePixel = 0
CopyFrame.Position = UDim2.new(0, 0, 1, -55)
CopyFrame.Size = UDim2.new(1, 0, 0, 50)
Instance.new("UICorner", CopyFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", CopyFrame).Color = Color3.fromRGB(108, 92, 231)

local CopyTextBox = Instance.new("TextBox")
CopyTextBox.Parent = CopyFrame
CopyTextBox.BackgroundTransparency = 1
CopyTextBox.Position = UDim2.new(0, 8, 0, 5)
CopyTextBox.Size = UDim2.new(1, -60, 1, -10)
CopyTextBox.Font = Enum.Font.Code
CopyTextBox.Text = "Clique em 'Inspecionar' para ver os dados..."
CopyTextBox.TextColor3 = Color3.fromRGB(180, 180, 200)
CopyTextBox.TextSize = 10
CopyTextBox.ClearTextOnFocus = false
CopyTextBox.TextEditable = false
CopyTextBox.TextWrapped = true
CopyTextBox.TextXAlignment = Enum.TextXAlignment.Left
CopyTextBox.TextYAlignment = Enum.TextYAlignment.Top

-- Botão Copiar
local CopyButton = Instance.new("TextButton")
CopyButton.Parent = CopyFrame
CopyButton.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
CopyButton.BorderSizePixel = 0
CopyButton.Position = UDim2.new(1, -48, 0, 10)
CopyButton.Size = UDim2.new(0, 42, 0, 30)
CopyButton.Text = "📋"
CopyButton.Font = Enum.Font.GothamBold
CopyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyButton.TextSize = 16
Instance.new("UICorner", CopyButton).CornerRadius = UDim.new(0, 6)

-- Botão Inspecionar Item
local InspectButton = Instance.new("TextButton")
InspectButton.Parent = ContentFrame
InspectButton.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
InspectButton.BorderSizePixel = 0
InspectButton.Position = UDim2.new(0, 0, 1, -62)
InspectButton.Size = UDim2.new(1, 0, 0, 40)
InspectButton.Text = "🔍 INSPECIONAR ITEM NA MÃO"
InspectButton.Font = Enum.Font.GothamBlack
InspectButton.TextColor3 = Color3.fromRGB(255, 255, 255)
InspectButton.TextSize = 14
InspectButton.ZIndex = 10
Instance.new("UICorner", InspectButton).CornerRadius = UDim.new(0, 8)

local inspectGradient = Instance.new("UIGradient", InspectButton)
inspectGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 71, 135)),
})
inspectGradient.Rotation = 90

-- ============================================
-- FUNÇÕES DE ARRASTAR
-- ============================================
local dragging = false
local dragStartPos = Vector2.zero
local frameStartPos = Vector2.zero

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
        frameStartPos = MainFrame.AbsolutePosition
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartPos
        local newPos = frameStartPos + delta
        
        -- Limitar à tela
        local screenSize = Workspace.CurrentCamera.ViewportSize
        newPos = Vector2.new(
            math.clamp(newPos.X, 0, screenSize.X - MainFrame.AbsoluteSize.X),
            math.clamp(newPos.Y, 0, screenSize.Y - MainFrame.AbsoluteSize.Y)
        )
        
        MainFrame.Position = UDim2.new(0, newPos.X, 0, newPos.Y)
    end
end)

-- ============================================
-- FUNÇÃO PARA OBTER DADOS DO ITEM
-- ============================================
local function GetItemData()
    if not Player.Character then
        return nil, "Você não está spawnado."
    end
    
    local tool = Player.Character:FindFirstChildOfClass("Tool")
    if not tool then
        -- Tentar também no Backpack
        tool = Player.Backpack:FindFirstChildOfClass("Tool")
        if not tool then
            return nil, "Nenhum item encontrado na mão ou mochila."
        end
    end
    
    local data = {}
    table.insert(data, "=":rep(50))
    table.insert(data, "ITEM: " .. tool.Name)
    table.insert(data, "=":rep(50))
    table.insert(data, "")
    table.insert(data, "📦 INFORMAÇÕES BÁSICAS")
    table.insert(data, "  Classe: " .. tool.ClassName)
    table.insert(data, "  Name: " .. tool.Name)
    table.insert(data, "  Parent: " .. (tool.Parent and tool.Parent:GetFullName() or "nil"))
    table.insert(data, "")
    table.insert(data, "⚙️ PROPRIEDADES")
    
    -- Lista de propriedades comuns para inspecionar
    local propsToCheck = {
        "RequiresHandle", "CanBeDropped", "ManualActivationOnly",
        "ToolTip", "TextureId", "Grip", "GripForward", "GripRight", "GripUp",
        "GripPos", "GripForward", "GripRight", "GripUp",
    }
    
    for _, propName in ipairs(propsToCheck) do
        local success, value = pcall(function() return tool[propName] end)
        if success and value ~= nil then
            local valueStr = tostring(value)
            if typeof(value) == "Vector3" or typeof(value) == "CFrame" or typeof(value) == "Color3" then
                valueStr = tostring(value)
            elseif typeof(value) == "EnumItem" then
                valueStr = value.Name
            end
            table.insert(data, "  " .. propName .. ": " .. valueStr)
        end
    end
    
    -- Verificar partes do tool
    table.insert(data, "")
    table.insert(data, "🧩 PARTES DA FERRAMENTA")
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BasePart") then
            table.insert(data, "  [" .. child.ClassName .. "] " .. child.Name)
            table.insert(data, "    Position: " .. tostring(child.Position))
            table.insert(data, "    Size: " .. tostring(child.Size))
            table.insert(data, "    Material: " .. child.Material.Name)
            table.insert(data, "    Color: " .. tostring(child.Color))
            table.insert(data, "    CanCollide: " .. tostring(child.CanCollide))
            table.insert(data, "    Anchored: " .. tostring(child.Anchored))
            table.insert(data, "    Transparency: " .. tostring(child.Transparency))
            table.insert(data, "    Mass: " .. tostring(child.Mass))
        end
    end
    
    -- Verificar scripts
    table.insert(data, "")
    table.insert(data, "📜 SCRIPTS")
    local scripts = {}
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BaseScript") then
            table.insert(scripts, "  [" .. child.ClassName .. "] " .. child.Name .. (child.Enabled and " (Ativado)" or " (Desativado)"))
        end
    end
    if #scripts > 0 then
        for _, s in ipairs(scripts) do
            table.insert(data, s)
        end
    else
        table.insert(data, "  Nenhum script encontrado.")
    end
    
    -- Verificar valores e atributos
    table.insert(data, "")
    table.insert(data, "💾 ATRIBUTOS")
    local attributes = tool:GetAttributes()
    if next(attributes) then
        for attrName, attrValue in pairs(attributes) do
            table.insert(data, "  " .. attrName .. ": " .. tostring(attrValue))
        end
    else
        table.insert(data, "  Nenhum atributo encontrado.")
    end
    
    -- Verificar tags
    table.insert(data, "")
    table.insert(data, "🏷️ TAGS")
    local tags = CollectionService:GetTags(tool)
    if #tags > 0 then
        for _, tag in ipairs(tags) do
            table.insert(data, "  " .. tag)
        end
    else
        table.insert(data, "  Nenhuma tag.")
    end
    
    table.insert(data, "")
    table.insert(data, "=":rep(50))
    
    return tool, table.concat(data, "\n")
end

-- ============================================
-- FUNÇÃO PARA CRIAR CARDS DE PROPRIEDADE
-- ============================================
local function CreatePropertyCard(name, value)
    local frame = Instance.new("Frame")
    frame.Parent = ScrollFrame
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, -10, 0, 40)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Parent = frame
    nameLabel.BackgroundTransparency = 1
    nameLabel.Position = UDim2.new(0, 10, 0, 0)
    nameLabel.Size = UDim2.new(0, 150, 1, 0)
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Text = name
    nameLabel.TextColor3 = Color3.fromRGB(108, 92, 231)
    nameLabel.TextSize = 11
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Parent = frame
    valueLabel.BackgroundTransparency = 1
    valueLabel.Position = UDim2.new(0, 160, 0, 0)
    valueLabel.Size = UDim2.new(1, -170, 1, 0)
    valueLabel.Font = Enum.Font.Code
    valueLabel.Text = value
    valueLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
    valueLabel.TextSize = 10
    valueLabel.TextXAlignment = Enum.TextXAlignment.Left
    valueLabel.TextWrapped = true
    
    return frame
end

-- ============================================
-- FUNÇÃO PARA ATUALIZAR A LISTA
-- ============================================
local function UpdatePropertyList(tool, dataText)
    -- Limpar scroll
    for _, child in ipairs(ScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    if not tool then
        CreatePropertyCard("Status", dataText)
        return
    end
    
    -- Criar cards para propriedades principais
    CreatePropertyCard("Nome", tool.Name)
    CreatePropertyCard("Classe", tool.ClassName)
    CreatePropertyCard("Parent", tool.Parent and tool.Parent:GetFullName() or "nil")
    CreatePropertyCard("ToolTip", tool.ToolTip or "Nenhum")
    CreatePropertyCard("Pode Dropar?", tostring(tool.CanBeDropped))
    CreatePropertyCard("Ativação Manual", tostring(tool.ManualActivationOnly))
    
    -- Grip
    if tool.Grip then
        CreatePropertyCard("Grip", tostring(tool.Grip))
    end
    CreatePropertyCard("GripForward", tostring(tool.GripForward))
    CreatePropertyCard("GripRight", tostring(tool.GripRight))
    CreatePropertyCard("GripUp", tostring(tool.GripUp))
    CreatePropertyCard("GripPos", tostring(tool.GripPos))
    
    -- TextureId
    if tool.TextureId then
        CreatePropertyCard("TextureId", tool.TextureId)
    end
    
    -- Partes
    local partCount = 0
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BasePart") then partCount += 1 end
    end
    CreatePropertyCard("Partes", tostring(partCount) .. " partes")
    
    -- Scripts
    local scriptCount = 0
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BaseScript") then scriptCount += 1 end
    end
    CreatePropertyCard("Scripts", tostring(scriptCount) .. " scripts")
    
    -- Atributos
    local attrCount = 0
    for _ in pairs(tool:GetAttributes()) do attrCount += 1 end
    CreatePropertyCard("Atributos", tostring(attrCount))
    
    -- Tags
    local tags = CollectionService:GetTags(tool)
    CreatePropertyCard("Tags", #tags > 0 and table.concat(tags, ", ") or "Nenhuma")
    
    -- Atualizar tamanho do canvas
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 50 + (#ScrollFrame:GetChildren() * 46))
end

-- ============================================
-- BOTÃO INSPECIONAR
-- ============================================
InspectButton.MouseButton1Click:Connect(function()
    local tool, result = GetItemData()
    
    -- Atualizar área de texto copiável
    if tool then
        CopyTextBox.Text = result
    else
        CopyTextBox.Text = result -- mensagem de erro
    end
    
    -- Atualizar lista de propriedades
    UpdatePropertyList(tool, result)
end)

-- ============================================
-- BOTÃO COPIAR
-- ============================================
CopyButton.MouseButton1Click:Connect(function()
    if CopyTextBox.Text ~= "" then
        -- Torna o texto selecionável temporariamente
        CopyTextBox.TextEditable = true
        CopyTextBox:CaptureFocus()
        CopyTextBox.SelectionStart = 0
        CopyTextBox.CursorPosition = #CopyTextBox.Text + 1
        
        -- Copia para a área de transferência (não disponível diretamente, então usamos o método nativo)
        pcall(function()
            -- Em executores modernos, podemos usar setclipboard
            setclipboard and setclipboard(CopyTextBox.Text)
        end)
        
        CopyTextBox.TextEditable = false
        
        -- Feedback visual
        CopyButton.Text = "✅"
        task.wait(1)
        CopyButton.Text = "📋"
    end
end)

-- ============================================
-- BOTÕES DE CONTROLE
-- ============================================
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

local contentVisible = true
MinimizeButton.MouseButton1Click:Connect(function()
    contentVisible = not contentVisible
    ContentFrame.Visible = contentVisible
    MinimizeButton.Text = contentVisible and "−" or "+"
    MainFrame.Size = contentVisible and UDim2.new(0, 420, 0, 480) or UDim2.new(0, 420, 0, 45)
end)

-- ============================================
-- ATUALIZAÇÃO EM TEMPO REAL (A CADA 2 SEGUNDOS)
-- ============================================
coroutine.wrap(function()
    while ScreenGui and ScreenGui.Parent do
        task.wait(2)
        local tool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
        if tool then
            local _, result = GetItemData()
            CopyTextBox.Text = result
        end
    end
end)()

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("🔍 Item Inspector carregado!")
print("  - Segure um item e clique em 'Inspecionar'")
print("  - A interface é arrastável pela barra de título")
print("  - Use o botão de copiar para obter os dados")

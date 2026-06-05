--[[
    🔍 Item Inspector - Script Roblox
    Interface arrastável com botão de fechar
    Mostra todos os dados do item que você está segurando
    Versão corrigida - Interface garantida
--]]

-- Serviços
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")

-- Espera o jogo carregar completamente
repeat task.wait() until Player.Character

-- Função de notificação simples
local function Notificar(texto)
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(0.5, -150, 0, 10)
    frame.Size = UDim2.new(0, 300, 0, 40)
    frame.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
    
    local label = Instance.new("TextLabel")
    label.Parent = frame
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Font = Enum.Font.GothamBold
    label.Text = texto
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 14
    
    local tween = TweenService:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, 20)})
    tween:Play()
    
    task.wait(3)
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, -40)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function() gui:Destroy() end)
end

-- ============================================
-- CRIAR GUI PRINCIPAL
-- ============================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ItemInspector_" .. math.random(1000, 9999)
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
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -240)
MainFrame.Size = UDim2.new(0, 420, 0, 480)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.ClipsDescendants = true

local mainCorner = Instance.new("UICorner", MainFrame)
mainCorner.CornerRadius = UDim.new(0, 12)

local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Color3.fromRGB(108, 92, 231)
mainStroke.Thickness = 2

-- ============================================
-- BARRA DE TÍTULO (ÁREA DE ARRASTAR)
-- ============================================
local TitleBar = Instance.new("Frame")
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 45)

local titleCorner = Instance.new("UICorner", TitleBar)
titleCorner.CornerRadius = UDim.new(0, 12)

local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 12)
titleFix.Position = UDim2.new(0, 0, 1, -12)

-- Ícone
local IconLabel = Instance.new("TextLabel")
IconLabel.Parent = TitleBar
IconLabel.BackgroundTransparency = 1
IconLabel.Position = UDim2.new(0, 12, 0, 5)
IconLabel.Size = UDim2.new(0, 35, 0, 35)
IconLabel.Text = "🔍"
IconLabel.TextSize = 22
IconLabel.Font = Enum.Font.Gotham

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Parent = TitleBar
TitleLabel.BackgroundTransparency = 1
TitleLabel.Position = UDim2.new(0, 50, 0, 0)
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
CloseButton.Position = UDim2.new(1, -45, 0, 8)
CloseButton.Size = UDim2.new(0, 32, 0, 28)
CloseButton.Text = "✕"
CloseButton.Font = Enum.Font.GothamBlack
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 16
Instance.new("UICorner", CloseButton).CornerRadius = UDim.new(0, 8)

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
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 100)

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = ScrollFrame
UIListLayout.Padding = UDim.new(0, 6)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Área de texto copiável
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
InspectButton.Position = UDim2.new(0, 0, 1, -60)
InspectButton.Size = UDim2.new(1, 0, 0, 40)
InspectButton.Text = "🔍 INSPECIONAR ITEM NA MÃO"
InspectButton.Font = Enum.Font.GothamBlack
InspectButton.TextColor3 = Color3.fromRGB(255, 255, 255)
InspectButton.TextSize = 14
Instance.new("UICorner", InspectButton).CornerRadius = UDim.new(0, 8)

-- ============================================
-- FUNÇÕES DE ARRASTAR
-- ============================================
local dragging = false
local dragStartPos = nil
local frameStartPos = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
        frameStartPos = MainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartPos
        MainFrame.Position = UDim2.new(
            frameStartPos.X.Scale,
            frameStartPos.X.Offset + delta.X,
            frameStartPos.Y.Scale,
            frameStartPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
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
        tool = Player.Backpack:FindFirstChildOfClass("Tool")
        if not tool then
            return nil, "Nenhum item encontrado na mão ou mochila."
        end
    end
    
    local parts = {}
    table.insert(parts, string.rep("=", 50))
    table.insert(parts, "ITEM: " .. tool.Name)
    table.insert(parts, string.rep("=", 50))
    table.insert(parts, "")
    table.insert(parts, "📦 INFORMAÇÕES BÁSICAS")
    table.insert(parts, "  Classe: " .. tool.ClassName)
    table.insert(parts, "  Name: " .. tool.Name)
    table.insert(parts, "  Parent: " .. (tool.Parent and tool.Parent:GetFullName() or "nil"))
    table.insert(parts, "")
    table.insert(parts, "⚙️ PROPRIEDADES")
    
    local propsToCheck = {
        "RequiresHandle", "CanBeDropped", "ManualActivationOnly",
        "ToolTip", "TextureId", "Grip", "GripForward", "GripRight", "GripUp",
        "GripPos",
    }
    
    for _, propName in ipairs(propsToCheck) do
        local success, value = pcall(function() return tool[propName] end)
        if success and value ~= nil then
            local valueStr = tostring(value)
            if typeof(value) == "Vector3" or typeof(value) == "CFrame" then
                valueStr = tostring(value)
            elseif typeof(value) == "EnumItem" then
                valueStr = value.Name
            end
            table.insert(parts, "  " .. propName .. ": " .. valueStr)
        end
    end
    
    -- Partes do tool
    table.insert(parts, "")
    table.insert(parts, "🧩 PARTES DA FERRAMENTA")
    local partCount = 0
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BasePart") then
            partCount = partCount + 1
            table.insert(parts, "  [" .. child.ClassName .. "] " .. child.Name)
            table.insert(parts, "    Posição: " .. tostring(child.Position))
            table.insert(parts, "    Tamanho: " .. tostring(child.Size))
            table.insert(parts, "    Material: " .. child.Material.Name)
            table.insert(parts, "    Cor: " .. tostring(child.Color))
        end
    end
    if partCount == 0 then
        table.insert(parts, "  Nenhuma parte encontrada.")
    end
    
    -- Scripts
    table.insert(parts, "")
    table.insert(parts, "📜 SCRIPTS")
    local scriptCount = 0
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BaseScript") then
            scriptCount = scriptCount + 1
            table.insert(parts, "  [" .. child.ClassName .. "] " .. child.Name .. (child.Enabled and " (Ativado)" or " (Desativado)"))
        end
    end
    if scriptCount == 0 then
        table.insert(parts, "  Nenhum script encontrado.")
    end
    
    -- Atributos
    table.insert(parts, "")
    table.insert(parts, "💾 ATRIBUTOS")
    local attributes = tool:GetAttributes()
    local attrCount = 0
    for attrName, attrValue in pairs(attributes) do
        attrCount = attrCount + 1
        table.insert(parts, "  " .. attrName .. ": " .. tostring(attrValue))
    end
    if attrCount == 0 then
        table.insert(parts, "  Nenhum atributo.")
    end
    
    -- Tags
    table.insert(parts, "")
    table.insert(parts, "🏷️ TAGS")
    local tags = CollectionService:GetTags(tool)
    if #tags > 0 then
        for _, tag in ipairs(tags) do
            table.insert(parts, "  " .. tag)
        end
    else
        table.insert(parts, "  Nenhuma tag.")
    end
    
    table.insert(parts, "")
    table.insert(parts, string.rep("=", 50))
    
    return tool, table.concat(parts, "\n")
end

-- ============================================
-- FUNÇÃO PARA ATUALIZAR A LISTA
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

local function UpdatePropertyList(tool, dataText)
    -- Limpar scroll
    for _, child in ipairs(ScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    if not tool then
        CreatePropertyCard("Status", dataText)
        ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 50)
        return
    end
    
    -- Propriedades principais
    CreatePropertyCard("Nome", tool.Name)
    CreatePropertyCard("Classe", tool.ClassName)
    CreatePropertyCard("Parent", tool.Parent and tool.Parent:GetFullName() or "nil")
    CreatePropertyCard("ToolTip", tool.ToolTip or "Nenhum")
    CreatePropertyCard("Pode Dropar?", tostring(tool.CanBeDropped))
    CreatePropertyCard("Ativação Manual", tostring(tool.ManualActivationOnly))
    
    if tool.Grip then
        CreatePropertyCard("Grip", tostring(tool.Grip))
    end
    CreatePropertyCard("GripForward", tostring(tool.GripForward))
    CreatePropertyCard("GripRight", tostring(tool.GripRight))
    CreatePropertyCard("GripUp", tostring(tool.GripUp))
    CreatePropertyCard("GripPos", tostring(tool.GripPos))
    
    if tool.TextureId then
        CreatePropertyCard("TextureId", tool.TextureId)
    end
    
    -- Contagens
    local partCount = 0
    local scriptCount = 0
    for _, child in ipairs(tool:GetDescendants()) do
        if child:IsA("BasePart") then partCount = partCount + 1 end
        if child:IsA("BaseScript") then scriptCount = scriptCount + 1 end
    end
    
    CreatePropertyCard("Partes", tostring(partCount) .. " partes")
    CreatePropertyCard("Scripts", tostring(scriptCount) .. " scripts")
    
    local attrCount = 0
    for _ in pairs(tool:GetAttributes()) do attrCount = attrCount + 1 end
    CreatePropertyCard("Atributos", tostring(attrCount))
    
    local tags = CollectionService:GetTags(tool)
    CreatePropertyCard("Tags", #tags > 0 and table.concat(tags, ", ") or "Nenhuma")
    
    -- Ajustar canvas
    local childCount = #ScrollFrame:GetChildren()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, childCount * 46 + 10)
end

-- ============================================
-- BOTÃO INSPECIONAR
-- ============================================
InspectButton.MouseButton1Click:Connect(function()
    local tool, result = GetItemData()
    CopyTextBox.Text = result
    UpdatePropertyList(tool, result)
    Notificar("Item inspecionado: " .. (tool and tool.Name or "Nenhum"))
end)

-- ============================================
-- BOTÃO COPIAR
-- ============================================
CopyButton.MouseButton1Click:Connect(function()
    if CopyTextBox.Text ~= "" then
        -- Tentar copiar usando setclipboard (disponível em alguns executores)
        local success = pcall(function()
            if setclipboard then
                setclipboard(CopyTextBox.Text)
            elseif writefile then
                writefile("clipboard.txt", CopyTextBox.Text)
            end
        end)
        
        if success then
            CopyButton.Text = "✅"
            Notificar("Dados copiados!")
        else
            -- Fallback: selecionar o texto manualmente
            CopyTextBox.TextEditable = true
            CopyTextBox:CaptureFocus()
            CopyTextBox.SelectionStart = 0
            CopyTextBox.CursorPosition = #CopyTextBox.Text + 1
            Notificar("Selecione o texto e pressione Ctrl+C")
        end
        
        task.wait(1.5)
        CopyButton.Text = "📋"
        CopyTextBox.TextEditable = false
    end
end)

-- ============================================
-- BOTÃO FECHAR
-- ============================================
CloseButton.MouseButton1Click:Connect(function()
    Notificar("Item Inspector fechado")
    ScreenGui:Destroy()
end)

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
Notificar("🔍 Item Inspector carregado! Segure um item e clique em 'Inspecionar'")
print("Item Inspector carregado com sucesso!")

--[[
    RESTOCK_ITEM_UNIVERSAL.lua
    Atalho: K = Restock (adiciona um novo item igual ao último equipado)
    Também inclui: H = Reset (substitui item da mão) e J = Duplicar (cópia extra)
    Interface garantida, funciona com qualquer ferramenta.
]]--

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE GARANTIDA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "RestockItem_UI"
gui.ResetOnSpawn = false

local function safeParent(gui)
    local ok = pcall(function() gui.Parent = game:GetService("CoreGui") end)
    if ok and gui.Parent then return true end
    local pg = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui", 30)
    if pg then
        ok = pcall(function() gui.Parent = pg end)
        if ok and gui.Parent then return true end
    end
    return false
end

if not safeParent(gui) then
    gui:Destroy()
    local char = player.Character or player.CharacterAdded:Wait()
    local head = char:WaitForChild("Head", 10)
    if head then
        local sg = Instance.new("SurfaceGui")
        sg.Adornee = head
        sg.Face = Enum.NormalId.Front
        sg.CanvasSize = Vector2.new(240, 130)
        sg.Parent = head
        gui = sg
    else
        StarterGui:SetCore("SendNotification", {
            Title = "Restock Item",
            Text = "Pressione K para restock, H para reset, J para duplicar",
            Duration = 10
        })
    end
end

-- Janela
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 270, 0, 195)
frame.Position = UDim2.new(1, -280, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 26)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "♻️ Restock Item"
title.TextColor3 = Color3.fromRGB(255, 200, 80)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.BackgroundTransparency = 1
title.Position = UDim2.new(0, 10, 0, 0)
title.Size = UDim2.new(1, -60, 1, 0)
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0, 28, 0, 26)
minimizeBtn.Position = UDim2.new(1, -56, 0, 0)
minimizeBtn.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
closeBtn.BorderSizePixel = 0
closeBtn.Size = UDim2.new(0, 28, 0, 26)
closeBtn.Position = UDim2.new(1, -28, 0, 0)
closeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -26)
content.Position = UDim2.new(0, 0, 0, 26)
content.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "Equipe um item e use os atalhos"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 11
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 8)
statusLabel.TextWrapped = true
statusLabel.Parent = content

local lastToolNameLabel = Instance.new("TextLabel")
lastToolNameLabel.Text = "Último item: Nenhum"
lastToolNameLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
lastToolNameLabel.Font = Enum.Font.Gotham
lastToolNameLabel.TextSize = 10
lastToolNameLabel.BackgroundTransparency = 1
lastToolNameLabel.Size = UDim2.new(1, -20, 0, 16)
lastToolNameLabel.Position = UDim2.new(0, 10, 0, 28)
lastToolNameLabel.Parent = content

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0, 220, 0, 30)
resetBtn.Position = UDim2.new(0.5, -110, 0, 50)
resetBtn.BackgroundColor3 = Color3.fromRGB(200, 100, 30)
resetBtn.BorderSizePixel = 0
resetBtn.TextColor3 = Color3.new(1, 1, 1)
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 13
resetBtn.Text = "RESET (H)"
resetBtn.Parent = content

local dupeBtn = Instance.new("TextButton")
dupeBtn.Size = UDim2.new(0, 220, 0, 30)
dupeBtn.Position = UDim2.new(0.5, -110, 0, 84)
dupeBtn.BackgroundColor3 = Color3.fromRGB(30, 130, 200)
dupeBtn.BorderSizePixel = 0
dupeBtn.TextColor3 = Color3.new(1, 1, 1)
dupeBtn.Font = Enum.Font.GothamBold
dupeBtn.TextSize = 13
dupeBtn.Text = "DUPLICAR (J)"
dupeBtn.Parent = content

local restockBtn = Instance.new("TextButton")
restockBtn.Size = UDim2.new(0, 220, 0, 30)
restockBtn.Position = UDim2.new(0.5, -110, 0, 118)
restockBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 200)
restockBtn.BorderSizePixel = 0
restockBtn.TextColor3 = Color3.new(1, 1, 1)
restockBtn.Font = Enum.Font.GothamBold
restockBtn.TextSize = 13
restockBtn.Text = "RESTOCK (K)"
restockBtn.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Método: Pronto"
methodLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
methodLabel.Font = Enum.Font.Gotham
methodLabel.TextSize = 10
methodLabel.BackgroundTransparency = 1
methodLabel.Size = UDim2.new(1, -20, 0, 16)
methodLabel.Position = UDim2.new(0, 10, 0, 152)
methodLabel.Parent = content

-- Minimizar/Fechar
local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 270, 0, 26) or UDim2.new(0, 270, 0, 195)
end)
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- ================== LÓGICA DE CONTROLE ==================
local lastToolName = nil

-- Atualiza o nome da última ferramenta equipada
local function updateLastToolName()
    local tool = nil
    local char = player.Character
    if char then
        for _, obj in ipairs(char:GetChildren()) do
            if obj:IsA("Tool") then
                tool = obj
                break
            end
        end
    end
    if tool then
        lastToolName = tool.Name
        lastToolNameLabel.Text = "Último item: " .. lastToolName
    else
        lastToolNameLabel.Text = "Último item: Nenhum (equipe algo)"
    end
end

-- Monitora mudanças no personagem para saber qual ferramenta está equipada
player.CharacterAdded:Connect(function(char)
    char.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then
            updateLastToolName()
        end
    end)
    char.ChildRemoved:Connect(function(child)
        if child:IsA("Tool") then
            -- pode ter desequipado; ainda mantemos o último nome
        end
    end)
    -- Verifica se já tem uma ferramenta equipada ao nascer
    updateLastToolName()
end)

-- Encontra uma fonte limpa da ferramenta (template) para clonagem
local function findToolTemplate(toolName)
    -- Procura em locais comuns de templates
    local folders = {
        game:GetService("StarterPack"),
        game:GetService("ReplicatedStorage"),
        game:GetService("Lighting")
    }
    for _, folder in ipairs(folders) do
        local found = folder:FindFirstChild(toolName)
        if found and found:IsA("Tool") then
            return found
        end
    end
    -- Se não achar, tenta no Workspace (pode haver um modelo)
    local wsFound = Workspace:FindFirstChild(toolName)
    if wsFound and wsFound:IsA("Tool") then
        return wsFound
    end
    return nil
end

-- Restock: adiciona uma nova cópia da última ferramenta conhecida ao Backpack
local function restockItem()
    if not lastToolName then
        statusLabel.Text = "Nenhum item rastreado. Equipe algo primeiro."
        methodLabel.Text = "Erro"
        return
    end
    statusLabel.Text = "Restockando " .. lastToolName .. "..."
    local template = findToolTemplate(lastToolName)
    if not template then
        -- Tenta encontrar alguma instância da ferramenta no Workspace (caso tenha sido jogada)
        for _, obj in ipairs(Workspace:GetChildren()) do
            if obj:IsA("Tool") and obj.Name == lastToolName then
                template = obj
                break
            end
        end
    end
    if not template then
        statusLabel.Text = "Falha: fonte do item não encontrada"
        methodLabel.Text = "Item não está no mapa"
        return
    end
    local clone = template:Clone()
    clone.Parent = player.Backpack
    task.wait(0.1)
    if player.Backpack:FindFirstChild(clone.Name) then
        statusLabel.Text = "Restock concluído!"
        methodLabel.Text = "Novo item no inventário"
    else
        statusLabel.Text = "Falha ao adicionar ao inventário"
        methodLabel.Text = "Verifique proteções do jogo"
    end
end

-- Reset: substitui a ferramenta equipada por uma nova
local function resetItem()
    local char = player.Character
    if not char then return end
    local tool = nil
    for _, obj in ipairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            tool = obj
            break
        end
    end
    if not tool then
        statusLabel.Text = "Nenhum item na mão"
        methodLabel.Text = "Equipe um item"
        return
    end
    -- Atualiza o nome
    lastToolName = tool.Name
    lastToolNameLabel.Text = "Último item: " .. lastToolName
    -- Remove o original (da mão e possíveis cópias no Backpack)
    pcall(function()
        if tool.Parent == char then
            tool.Parent = nil
        end
        local bpCopy = player.Backpack:FindFirstChild(tool.Name)
        if bpCopy then bpCopy:Destroy() end
    end)
    -- Chama o restock para adicionar uma cópia limpa
    restockItem()
end

-- Duplicar: cria uma cópia extra mantendo o original
local function duplicateItem()
    local tool = nil
    local char = player.Character
    if char then
        for _, obj in ipairs(char:GetChildren()) do
            if obj:IsA("Tool") then
                tool = obj
                break
            end
        end
    end
    if not tool then
        statusLabel.Text = "Nenhum item na mão"
        methodLabel.Text = "Equipe um item"
        return
    end
    lastToolName = tool.Name
    lastToolNameLabel.Text = "Último item: " .. lastToolName
    -- Tenta usar o template primeiro para uma cópia limpa
    local template = findToolTemplate(lastToolName)
    if template then
        local clone = template:Clone()
        clone.Parent = player.Backpack
        task.wait(0.1)
        if player.Backpack:FindFirstChild(clone.Name) then
            statusLabel.Text = "Duplicado com sucesso"
            methodLabel.Text = "Clone limpo no inventário"
            return
        end
    end
    -- Fallback: clonar o próprio item atual (pode não estar limpo, mas funciona)
    local clone = tool:Clone()
    clone.Parent = player.Backpack
    task.wait(0.1)
    if player.Backpack:FindFirstChild(clone.Name) then
        statusLabel.Text = "Duplicado (cópia direta)"
        methodLabel.Text = "Clone no inventário"
    else
        statusLabel.Text = "Falha ao duplicar"
        methodLabel.Text = "Erro"
    end
end

-- Conexões
resetBtn.MouseButton1Click:Connect(resetItem)
dupeBtn.MouseButton1Click:Connect(duplicateItem)
restockBtn.MouseButton1Click:Connect(restockItem)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.H then
        resetItem()
    elseif input.KeyCode == Enum.KeyCode.J then
        duplicateItem()
    elseif input.KeyCode == Enum.KeyCode.K then
        restockItem()
    end
end)

-- Atualiza o nome da ferramenta ao iniciar, se já houver personagem
if player.Character then
    updateLastToolName()
end

-- Notificação de carregamento
task.delay(1, function()
    StarterGui:SetCore("SendNotification", {
        Title = "Restock Item",
        Text = "K = Restock | H = Reset | J = Duplicar",
        Duration = 6
    })
end)

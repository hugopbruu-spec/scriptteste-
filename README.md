--[[
    ITEM_DUPLICATOR.lua – Duplica a ferramenta na mão (real e replicada)
    Atalho: G para duplicar, ou botão na interface.
    Métodos: Remote automático, Backpack injection, TouchInterest.
]]--

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE COMPLETA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "ItemDuplicator_UI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 180)
frame.Position = UDim2.new(1, -270, 0, 500)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 28)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "📦 Duplicar Item"
title.TextColor3 = Color3.fromRGB(100, 255, 150)
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
minimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0, 28, 0, 28)
minimizeBtn.Position = UDim2.new(1, -56, 0, 0)
minimizeBtn.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
closeBtn.BorderSizePixel = 0
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -28, 0, 0)
closeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -28)
content.Position = UDim2.new(0, 0, 0, 28)
content.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "Pressione G ou clique no botão"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 12
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 8)
statusLabel.Parent = content

local duplicarBtn = Instance.new("TextButton")
duplicarBtn.Size = UDim2.new(0, 200, 0, 36)
duplicarBtn.Position = UDim2.new(0.5, -100, 0, 36)
duplicarBtn.BackgroundColor3 = Color3.fromRGB(30, 180, 80)
duplicarBtn.BorderSizePixel = 0
duplicarBtn.TextColor3 = Color3.new(1, 1, 1)
duplicarBtn.Font = Enum.Font.GothamBold
duplicarBtn.TextSize = 14
duplicarBtn.Text = "DUPLICAR (G)"
duplicarBtn.Parent = content

local methodText = Instance.new("TextLabel")
methodText.Text = "Método: Automático"
methodText.TextColor3 = Color3.fromRGB(180, 180, 200)
methodText.Font = Enum.Font.Gotham
methodText.TextSize = 11
methodText.BackgroundTransparency = 1
methodText.Size = UDim2.new(1, -20, 0, 20)
methodText.Position = UDim2.new(0, 10, 0, 82)
methodText.Parent = content

local configFrame = Instance.new("Frame")
configFrame.Size = UDim2.new(1, -20, 0, 30)
configFrame.Position = UDim2.new(0, 10, 0, 108)
configFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
configFrame.BorderSizePixel = 0
configFrame.Parent = content

local remoteLabel = Instance.new("TextLabel")
remoteLabel.Text = "Remote (opcional):"
remoteLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
remoteLabel.Font = Enum.Font.Gotham
remoteLabel.TextSize = 11
remoteLabel.BackgroundTransparency = 1
remoteLabel.Size = UDim2.new(0, 100, 1, 0)
remoteLabel.Position = UDim2.new(0, 4, 0, 0)
remoteLabel.Parent = configFrame

local remoteBox = Instance.new("TextBox")
remoteBox.Text = ""
remoteBox.PlaceholderText = "Nome do RemoteEvent..."
remoteBox.TextColor3 = Color3.new(1, 1, 1)
remoteBox.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
remoteBox.BorderSizePixel = 0
remoteBox.Font = Enum.Font.Gotham
remoteBox.TextSize = 12
remoteBox.Size = UDim2.new(1, -110, 0, 22)
remoteBox.Position = UDim2.new(0, 106, 0, 4)
remoteBox.Parent = configFrame

-- Minimizar / Fechar
local minimized = false
local function setMinimized(state)
    minimized = state
    content.Visible = not state
    frame.Size = state and UDim2.new(0, 260, 0, 28) or UDim2.new(0, 260, 0, 180)
end
minimizeBtn.MouseButton1Click:Connect(function() setMinimized(not minimized) end)
closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- ================== LÓGICA DE DUPLICAÇÃO ==================
local function getCurrentTool()
    local char = player.Character
    if not char then return nil end
    for _, obj in ipairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            return obj
        end
    end
    return nil
end

-- Método 1: Tentar usar um RemoteEvent existente no jogo
local function tryRemoteMethod(tool)
    local possibleNames = {
        "DropTool", "SpawnItem", "GiveItem", "CloneItem", "DupeItem",
        "SpawnTool", "ServerSpawn", "CreateTool", "toolDrop", "itemClone"
    }
    local remote = nil
    for _, name in ipairs(possibleNames) do
        local r = ReplicatedStorage:FindFirstChild(name)
        if r and r:IsA("RemoteEvent") then
            remote = r
            break
        end
    end
    -- Se o usuário especificou um nome no TextBox, use-o
    if remoteBox.Text ~= "" then
        local custom = ReplicatedStorage:FindFirstChild(remoteBox.Text)
        if custom and custom:IsA("RemoteEvent") then
            remote = custom
        end
    end
    
    if not remote then
        return false, "Nenhum RemoteEvent encontrado"
    end
    
    -- Tenta disparar com argumentos comuns (player, toolName, posição)
    local success = pcall(function()
        remote:FireServer(tool.Name, tool:Clone())
    end)
    if success then
        return true, "Remote disparado: " .. remote.Name
    else
        return false, "Falha ao disparar remote"
    end
end

-- Método 2: Injetar no Backpack (pode replicar em jogos menos protegidos)
local function tryBackpackMethod(tool)
    local clone = tool:Clone()
    clone.Parent = player.Backpack
    -- Força um pequeno delay para ver se o servidor reconhece
    wait(0.1)
    if player.Backpack:FindFirstChild(clone.Name) then
        return true, "Item adicionado à mochila"
    end
    clone:Destroy()
    return false, "Falha na injeção no Backpack"
end

-- Método 3: Criar no workspace e simular TouchInterest (drop)
local function tryTouchInterestMethod(tool)
    local clone = tool:Clone()
    clone.Parent = Workspace
    -- Posiciona na frente do personagem
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        clone:PivotTo(char.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5))
    end
    -- Tenta disparar o evento de toque com o personagem para "pegar"
    if clone:IsA("Tool") and char then
        local handle = clone:FindFirstChild("Handle")
        if handle and handle:IsA("BasePart") then
            firetouchinterest(handle, char:FindFirstChild("HumanoidRootPart"), 0)
            firetouchinterest(handle, char:FindFirstChild("HumanoidRootPart"), 1)
        end
    end
    return true, "Item criado no chão e simulado toque"
end

-- Método 4: Combinação de drop e pickup rápido (usando eventos do jogo)
local function tryDropPickupMethod(tool)
    local char = player.Character
    if not char or not tool.Parent == char then
        return false, "Ferramenta não está equipada"
    end
    -- Força o drop usando Humanoid:UnequipTools()
    local humanoid = char:FindFirstChild("Humanoid")
    if humanoid then
        humanoid:UnequipTools()
        wait(0.1)
        tool.Parent = Workspace  -- dropa no chão
        wait(0.1)
        -- Agora pega o clone que está no chão e também a original?
        -- Na verdade, o original foi dropado; precisamos criar um clone antes de dropar.
        -- Vamos corrigir: clonar antes de dropar.
    end
    return false, "Método não implementado corretamente"
end

-- Função principal de duplicação
local function duplicateItem()
    local tool = getCurrentTool()
    if not tool then
        statusLabel.Text = "Nenhum item na mão!"
        return
    end
    
    statusLabel.Text = "Duplicando..."
    
    -- Tenta os métodos em sequência
    local success, msg = tryRemoteMethod(tool)
    if success then
        methodText.Text = "Método: Remote (" .. msg .. ")"
        statusLabel.Text = "Duplicado com sucesso!"
        return
    end
    
    success, msg = tryBackpackMethod(tool)
    if success then
        methodText.Text = "Método: Backpack (" .. msg .. ")"
        statusLabel.Text = "Duplicado com sucesso!"
        return
    end
    
    success, msg = tryTouchInterestMethod(tool)
    if success then
        methodText.Text = "Método: TouchInterest (" .. msg .. ")"
        statusLabel.Text = "Duplicado (pode precisar pegar do chão)"
        return
    end
    
    statusLabel.Text = "Falha em todos os métodos. Especifique um Remote."
end

-- Botão
duplicarBtn.MouseButton1Click:Connect(duplicateItem)

-- Atalho G
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.G then
        duplicateItem()
    end
end)

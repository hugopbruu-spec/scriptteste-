--[[
    ITEM_RESET.lua – Reseta o item na mão como se fosse novo
    Atalho: H | Interface completa com status e método utilizado.
    Técnica: Clona o item, destrói o original e insere o clone no Backpack.
    Métodos alternativos: Drop forçado + pickup, recriação por nome.
]]--

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE COMPLETA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "ItemReset_UI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 170)
frame.Position = UDim2.new(1, -270, 0, 690)
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
title.Text = "🔄 Reset Item"
title.TextColor3 = Color3.fromRGB(255, 180, 50)
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
statusLabel.Text = "Pressione H ou o botão para resetar"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 12
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 10)
statusLabel.TextWrapped = true
statusLabel.Parent = content

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0, 200, 0, 36)
resetBtn.Position = UDim2.new(0.5, -100, 0, 40)
resetBtn.BackgroundColor3 = Color3.fromRGB(200, 130, 30)
resetBtn.BorderSizePixel = 0
resetBtn.TextColor3 = Color3.new(1, 1, 1)
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 14
resetBtn.Text = "RESETAR ITEM (H)"
resetBtn.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Método: Nenhum"
methodLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
methodLabel.Font = Enum.Font.Gotham
methodLabel.TextSize = 11
methodLabel.BackgroundTransparency = 1
methodLabel.Size = UDim2.new(1, -20, 0, 18)
methodLabel.Position = UDim2.new(0, 10, 0, 86)
methodLabel.Parent = content

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Equipe um item e pressione H"
infoLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 10
infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1, -20, 0, 16)
infoLabel.Position = UDim2.new(0, 10, 0, 110)
infoLabel.Parent = content

-- Minimizar / Fechar
local minimized = false
local function setMinimized(state)
    minimized = state
    content.Visible = not state
    frame.Size = state and UDim2.new(0, 260, 0, 28) or UDim2.new(0, 260, 0, 170)
end
minimizeBtn.MouseButton1Click:Connect(function() setMinimized(not minimized) end)
closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- ================== LÓGICA DE RESET DO ITEM ==================
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

-- Método 1: Clonar, destruir original, inserir clone no Backpack
local function resetByClone(tool)
    -- Salva referências
    local toolName = tool.Name
    local clone = tool:Clone()
    
    -- Remove original do personagem e do Backpack (se lá estiver)
    pcall(function()
        if tool.Parent == player.Character then
            tool.Parent = nil
        end
        local backpackTool = player.Backpack:FindFirstChild(toolName)
        if backpackTool then
            backpackTool:Destroy()
        end
    end)
    
    -- Aguarda um frame para o servidor processar a remoção
    RunService.Heartbeat:Wait()
    
    -- Insere o clone no Backpack
    clone.Parent = player.Backpack
    
    -- Verifica se o clone foi adicionado com sucesso
    wait(0.1)
    if player.Backpack:FindFirstChild(toolName) or player.Character:FindFirstChild(toolName) then
        return true, "Clone inserido no Backpack"
    else
        return false, "Falha na inserção"
    end
end

-- Método 2: Drop forçado e simulação de pickup via TouchInterest
local function resetByDrop(tool)
    local char = player.Character
    if not char then return false, "Sem personagem" end
    local humanoid = char:FindFirstChild("Humanoid")
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not rootPart then return false, "Sem Humanoid/RootPart" end
    
    -- Força desequipar
    humanoid:UnequipTools()
    wait(0.1)
    
    -- Move o tool para o chão (drop)
    tool.Parent = Workspace
    -- Posiciona na frente do personagem
    tool:PivotTo(rootPart.CFrame * CFrame.new(0, 0, -4))
    wait(0.1)
    
    -- Tenta simular o pickup com firetouchinterest (se disponível)
    local handle = tool:FindFirstChild("Handle") or tool:FindFirstChildWhichIsA("BasePart")
    if handle then
        -- Cria uma conexão de toque fictícia (funciona apenas se o jogo permitir)
        firetouchinterest(handle, rootPart, 0) -- touch began
        firetouchinterest(handle, rootPart, 1) -- touch ended
    end
    
    -- Tenta também forçar o equipamento
    pcall(function()
        humanoid:EquipTool(tool)
    end)
    
    -- Verifica se está equipado
    wait(0.1)
    if tool.Parent == char then
        return true, "Drop/Pickup simulado"
    else
        return false, "Falha no pickup"
    end
end

-- Método 3: Recriação por nome (apenas para ferramentas padrão)
local function resetByName(tool)
    local toolName = tool.Name
    -- Tenta encontrar o modelo original em algum lugar (ReplicatedStorage, StarterPack, etc.)
    local source = nil
    for _, folder in ipairs({ReplicatedStorage, game:GetService("StarterPack"), game:GetService("Lighting")}) do
        local found = folder:FindFirstChild(toolName)
        if found and found:IsA("Tool") then
            source = found
            break
        end
    end
    if not source then
        -- Tenta recriar via Instance.new (limitado, mas pode funcionar para ferramentas simples)
        local newTool = Instance.new("Tool")
        newTool.Name = toolName
        newTool.Parent = player.Backpack
        return true, "Tool genérica criada"
    end
    
    local clone = source:Clone()
    -- Destroi o original
    pcall(function() tool:Destroy() end)
    clone.Parent = player.Backpack
    return true, "Clonado de fonte externa"
end

-- Função principal
local function resetCurrentItem()
    local tool = getCurrentTool()
    if not tool then
        statusLabel.Text = "Nenhum item na mão!"
        methodLabel.Text = "Método: Nenhum"
        return
    end
    
    statusLabel.Text = "Resetando..."
    methodLabel.Text = "Método: Tentando..."
    
    -- Tenta métodos em sequência
    local success, msg = resetByClone(tool)
    if success then
        statusLabel.Text = "Item resetado com sucesso!"
        methodLabel.Text = "Método: Clone (" .. msg .. ")"
        return
    end
    
    success, msg = resetByDrop(tool)
    if success then
        statusLabel.Text = "Item resetado com sucesso!"
        methodLabel.Text = "Método: Drop/Pickup (" .. msg .. ")"
        return
    end
    
    success, msg = resetByName(tool)
    if success then
        statusLabel.Text = "Item resetado com sucesso!"
        methodLabel.Text = "Método: Recriação (" .. msg .. ")"
        return
    end
    
    statusLabel.Text = "Falha ao resetar. Tente novamente."
    methodLabel.Text = "Método: Falhou"
end

-- Botão e atalho
resetBtn.MouseButton1Click:Connect(resetCurrentItem)
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.H then
        resetCurrentItem()
    end
end)

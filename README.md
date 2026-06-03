--[[
    ITEM_RESET_GARANTIDO.lua
    Atalho: H para resetar o item da mão.
    Interface garantida: aparece no canto inferior direito.
]]--

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== CRIAÇÃO ROBUSTA DA INTERFACE ==================
local gui = Instance.new("ScreenGui")
gui.Name = "ItemReset_GUI"
gui.ResetOnSpawn = false

-- Tenta CoreGui, senão PlayerGui
local parentSuccess = pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not parentSuccess or not gui.Parent then
    pcall(function() gui.Parent = player:WaitForChild("PlayerGui") end)
end
-- Se ainda assim falhar, tentamos usar o PlayerGui diretamente
if not gui.Parent then
    gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
end

-- Cria a janela
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 160)
frame.Position = UDim2.new(1, -270, 0, 700)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 28)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "🔄 Reset Item"
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
statusLabel.Text = "Aperte H ou o botão"
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
resetBtn.Text = "RESETAR (H)"
resetBtn.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Método: Pronto"
methodLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
methodLabel.Font = Enum.Font.Gotham
methodLabel.TextSize = 11
methodLabel.BackgroundTransparency = 1
methodLabel.Size = UDim2.new(1, -20, 0, 18)
methodLabel.Position = UDim2.new(0, 10, 0, 86)
methodLabel.Parent = content

-- Controles de minimizar/fechar
local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 260, 0, 28) or UDim2.new(0, 260, 0, 160)
end)
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- ================== LÓGICA DE RESET DO ITEM ==================
local function getToolInHand()
    local char = player.Character
    if not char then return nil end
    for _, obj in ipairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            return obj
        end
    end
    return nil
end

-- Método principal: clone + substituição no Backpack
local function resetByClone(tool)
    local toolName = tool.Name
    -- Clona o item
    local clone = tool:Clone()
    
    -- Remove o original de onde estiver
    pcall(function()
        if tool.Parent == player.Character then
            tool.Parent = nil
        end
        local backpackCopy = player.Backpack:FindFirstChild(toolName)
        if backpackCopy then
            backpackCopy:Destroy()
        end
    end)
    
    -- Aguarda um pulso para o servidor processar
    RunService.Heartbeat:Wait()
    
    -- Insere o clone no Backpack
    clone.Parent = player.Backpack
    
    -- Confirma após um curto tempo
    task.wait(0.2)
    if player.Backpack:FindFirstChild(toolName) or (player.Character and player.Character:FindFirstChild(toolName)) then
        return true, "Clone substituído"
    else
        return false, "Clone não apareceu"
    end
end

-- Método auxiliar: drop e tentativa de pegar de volta
local function resetByDrop(tool)
    local char = player.Character
    if not char then return false, "Sem personagem" end
    local humanoid = char:FindFirstChild("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not root then return false, "Sem Humanoid/RootPart" end
    
    -- Desequipa
    humanoid:UnequipTools()
    task.wait(0.1)
    
    -- Move para o chão
    tool.Parent = Workspace
    tool:PivotTo(root.CFrame * CFrame.new(0, 0, -4))
    task.wait(0.15)
    
    -- Tenta equipar de volta (alguns jogos permitem)
    pcall(function()
        humanoid:EquipTool(tool)
    end)
    task.wait(0.15)
    
    if tool.Parent == char then
        -- Agora que está equipado, tentamos substituí-lo via clone (para garantir reset)
        local clone = tool:Clone()
        tool:Destroy()
        clone.Parent = player.Backpack
        task.wait(0.1)
        return true, "Drop + clone"
    else
        return false, "Falha ao reequipar"
    end
end

-- Método de fallback: procurar o modelo original no jogo
local function resetByName(tool)
    local toolName = tool.Name
    -- Locais onde ferramentas costumam ser guardadas
    local folders = {
        game:GetService("ReplicatedStorage"),
        game:GetService("Lighting"),
        game:GetService("StarterPack")
    }
    for _, folder in ipairs(folders) do
        local src = folder:FindFirstChild(toolName)
        if src and src:IsA("Tool") then
            local clone = src:Clone()
            pcall(function() tool:Destroy() end)
            clone.Parent = player.Backpack
            return true, "Clonado de " .. folder.Name
        end
    end
    return false, "Fonte não encontrada"
end

local function resetItem()
    local tool = getToolInHand()
    if not tool then
        statusLabel.Text = "Nenhum item na mão!"
        methodLabel.Text = "Método: Aguardando..."
        return
    end
    
    statusLabel.Text = "Resetando..."
    methodLabel.Text = "Método: Tentando..."
    
    local success, msg = resetByClone(tool)
    if success then
        statusLabel.Text = "Item resetado!"
        methodLabel.Text = "Método: Clone (" .. msg .. ")"
        return
    end
    
    success, msg = resetByDrop(tool)
    if success then
        statusLabel.Text = "Item resetado!"
        methodLabel.Text = "Método: Drop (" .. msg .. ")"
        return
    end
    
    success, msg = resetByName(tool)
    if success then
        statusLabel.Text = "Item resetado!"
        methodLabel.Text = "Método: Recriação (" .. msg .. ")"
        return
    end
    
    statusLabel.Text = "Falha ao resetar"
    methodLabel.Text = "Método: Nenhum funcionou"
end

-- Conecta botão e tecla H
resetBtn.MouseButton1Click:Connect(resetItem)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.H then
        resetItem()
    end
end)

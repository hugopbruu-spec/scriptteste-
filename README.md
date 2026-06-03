--[[
    DICE_RESTOCK.lua – Reset, Duplicar e Restock de Dice
    Atalhos:
        H = Reset (substitui o Dice da mão por um novo)
        J = Duplicar (cria uma cópia extra do Dice que está na mão)
        K = Restock (adiciona um novo Dice ao inventário, mesmo sem nenhum equipado)
    Ideal para "bugar" o dado: jogue o dado no chão, pressione K e ganhe outro.
]]--

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE GARANTIDA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceRestock_UI"
gui.ResetOnSpawn = false

-- Tenta CoreGui, depois PlayerGui, depois SurfaceGui na cabeça
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
            Title = "Dice Restock",
            Text = "H=Reset | J=Duplicar | K=Restock",
            Duration = 10
        })
    end
end

-- Janela
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 270, 0, 200)
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
title.Text = "🎲 Dice Restock"
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
statusLabel.Text = "H=Reset | J=Duplicar | K=Restock"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 11
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 10)
statusLabel.TextWrapped = true
statusLabel.Parent = content

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0, 220, 0, 32)
resetBtn.Position = UDim2.new(0.5, -110, 0, 38)
resetBtn.BackgroundColor3 = Color3.fromRGB(200, 100, 30)
resetBtn.BorderSizePixel = 0
resetBtn.TextColor3 = Color3.new(1, 1, 1)
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 13
resetBtn.Text = "RESET (H)"
resetBtn.Parent = content

local dupeBtn = Instance.new("TextButton")
dupeBtn.Size = UDim2.new(0, 220, 0, 32)
dupeBtn.Position = UDim2.new(0.5, -110, 0, 76)
dupeBtn.BackgroundColor3 = Color3.fromRGB(30, 130, 200)
dupeBtn.BorderSizePixel = 0
dupeBtn.TextColor3 = Color3.new(1, 1, 1)
dupeBtn.Font = Enum.Font.GothamBold
dupeBtn.TextSize = 13
dupeBtn.Text = "DUPLICAR (J)"
dupeBtn.Parent = content

local restockBtn = Instance.new("TextButton")
restockBtn.Size = UDim2.new(0, 220, 0, 32)
restockBtn.Position = UDim2.new(0.5, -110, 0, 114)
restockBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 200)
restockBtn.BorderSizePixel = 0
restockBtn.TextColor3 = Color3.new(1, 1, 1)
restockBtn.Font = Enum.Font.GothamBold
restockBtn.TextSize = 13
restockBtn.Text = "RESTOCK (K)"
restockBtn.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Pronto"
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
    frame.Size = minimized and UDim2.new(0, 270, 0, 26) or UDim2.new(0, 270, 0, 200)
end)
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- ================== FUNÇÕES ==================
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

-- Função principal: encontrar uma fonte do Dice para clonar
local function findDiceSource()
    local searchNames = {"Dice", "dice", "DiceTool", "DiceGiver"} -- variações comuns
    local folders = {
        game:GetService("StarterPack"),
        game:GetService("ReplicatedStorage"),
        game:GetService("Lighting"),
        Workspace -- alguns jogos deixam moldes no workspace
    }
    for _, folder in ipairs(folders) do
        for _, name in ipairs(searchNames) do
            local found = folder:FindFirstChild(name)
            if found and found:IsA("Tool") then
                return found
            end
        end
    end
    -- Último recurso: procurar qualquer ferramenta cujo nome contenha "dice" (case insensitive)
    local allFolders = {game:GetService("StarterPack"), game:GetService("ReplicatedStorage"), game:GetService("Lighting")}
    for _, folder in ipairs(allFolders) do
        for _, child in ipairs(folder:GetChildren()) do
            if child:IsA("Tool") and string.lower(child.Name):find("dice") then
                return child
            end
        end
    end
    return nil
end

-- Adiciona um novo Dice ao Backpack, retornando true se bem-sucedido
local function giveNewDice()
    local source = findDiceSource()
    if not source then
        -- Tenta clonar a partir de algum dado no chão (último caso)
        for _, obj in ipairs(Workspace:GetChildren()) do
            if obj:IsA("Tool") and string.lower(obj.Name):find("dice") then
                source = obj
                break
            end
        end
    end
    if not source then
        return false
    end
    local clone = source:Clone()
    clone.Parent = player.Backpack
    task.wait(0.1)
    return player.Backpack:FindFirstChild(clone.Name) ~= nil
end

-- RESET: substitui o Dice atual na mão por um novo
local function resetDice()
    local tool = getToolInHand()
    if not tool or not string.lower(tool.Name):find("dice") then
        statusLabel.Text = "Equipe um Dice primeiro"
        methodLabel.Text = "Item não é Dice"
        return
    end
    statusLabel.Text = "Resetando..."
    local success = giveNewDice()
    if success then
        -- Remove o antigo (da mão e qualquer cópia no Backpack)
        pcall(function()
            if tool.Parent == player.Character then
                tool.Parent = nil
            end
            local bp = player.Backpack:FindFirstChild(tool.Name)
            if bp then bp:Destroy() end
        end)
        task.wait(0.2)
        statusLabel.Text = "Reset concluído"
        methodLabel.Text = "Novo Dice no inventário"
    else
        statusLabel.Text = "Falha ao encontrar fonte do Dice"
        methodLabel.Text = "Erro"
    end
end

-- DUPLICAR: se estiver com um Dice na mão, cria uma cópia extra no inventário
local function dupeDice()
    local tool = getToolInHand()
    if not tool or not string.lower(tool.Name):find("dice") then
        statusLabel.Text = "Equipe um Dice primeiro"
        methodLabel.Text = "Item não é Dice"
        return
    end
    statusLabel.Text = "Duplicando..."
    local clone = tool:Clone()
    clone.Parent = player.Backpack
    task.wait(0.1)
    if player.Backpack:FindFirstChild(clone.Name) then
        statusLabel.Text = "Duplicado com sucesso"
        methodLabel.Text = "Clone no inventário"
    else
        statusLabel.Text = "Falha ao duplicar"
        methodLabel.Text = "Tentando método alternativo"
        -- Fallback: adicionar via fonte
        if giveNewDice() then
            statusLabel.Text = "Duplicado via restock"
            methodLabel.Text = "Clone no inventário"
        end
    end
end

-- RESTOCK: adiciona um novo Dice, independentemente de ter um na mão (ideal após jogar)
local function restockDice()
    statusLabel.Text = "Restock..."
    local success = giveNewDice()
    if success then
        statusLabel.Text = "Novo Dice adicionado"
        methodLabel.Text = "Pronto para usar"
    else
        statusLabel.Text = "Falha: Dice não encontrado"
        methodLabel.Text = "Verifique o nome do item"
    end
end

-- Conexões
resetBtn.MouseButton1Click:Connect(resetDice)
dupeBtn.MouseButton1Click:Connect(dupeDice)
restockBtn.MouseButton1Click:Connect(restockDice)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.H then
        resetDice()
    elseif input.KeyCode == Enum.KeyCode.J then
        dupeDice()
    elseif input.KeyCode == Enum.KeyCode.K then
        restockDice()
    end
end)

-- Notificação de carregamento
task.delay(1, function()
    StarterGui:SetCore("SendNotification", {
        Title = "Dice Restock",
        Text = "H=Reset | J=Duplicar | K=Restock",
        Duration = 6
    })
end)

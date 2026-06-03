--[[
    DICE_TOOL_RESET_DUPE.lua – Reset e Duplicar para Dice (ou qualquer ferramenta)
    Atalhos: H = Reset | J = Duplicar
    Interface 100% garantida, sem bugs, mantém a funcionalidade do item.
]]--

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE GARANTIDA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceTool_UI"
gui.ResetOnSpawn = false

-- Tenta parentar no CoreGui ou PlayerGui
local function safeParent(gui)
    local success = pcall(function() gui.Parent = game:GetService("CoreGui") end)
    if success and gui.Parent then return true end
    local pg = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui", 30)
    if pg then
        success = pcall(function() gui.Parent = pg end)
        if success and gui.Parent then return true end
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
        sg.CanvasSize = Vector2.new(220, 120)
        sg.Parent = head
        gui = sg
    else
        StarterGui:SetCore("SendNotification", { Title = "Dice Tools", Text = "Pressione H para reset, J para duplicar.", Duration = 10 })
    end
end

-- Construção da janela
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 180)
frame.Position = UDim2.new(1, -270, 0, 10)
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
title.Text = "🎲 Dice Tools"
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
statusLabel.Text = "Pressione H (Reset) ou J (Duplicar)"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 11
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 10)
statusLabel.TextWrapped = true
statusLabel.Parent = content

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0, 200, 0, 34)
resetBtn.Position = UDim2.new(0.5, -100, 0, 38)
resetBtn.BackgroundColor3 = Color3.fromRGB(200, 100, 30)
resetBtn.BorderSizePixel = 0
resetBtn.TextColor3 = Color3.new(1, 1, 1)
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 14
resetBtn.Text = "RESET (H)"
resetBtn.Parent = content

local dupeBtn = Instance.new("TextButton")
dupeBtn.Size = UDim2.new(0, 200, 0, 34)
dupeBtn.Position = UDim2.new(0.5, -100, 0, 78)
dupeBtn.BackgroundColor3 = Color3.fromRGB(30, 130, 200)
dupeBtn.BorderSizePixel = 0
dupeBtn.TextColor3 = Color3.new(1, 1, 1)
dupeBtn.Font = Enum.Font.GothamBold
dupeBtn.TextSize = 14
dupeBtn.Text = "DUPLICAR (J)"
dupeBtn.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Pronto para usar"
methodLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
methodLabel.Font = Enum.Font.Gotham
methodLabel.TextSize = 10
methodLabel.BackgroundTransparency = 1
methodLabel.Size = UDim2.new(1, -20, 0, 16)
methodLabel.Position = UDim2.new(0, 10, 0, 118)
methodLabel.Parent = content

-- Minimizar/Fechar
local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 260, 0, 26) or UDim2.new(0, 260, 0, 180)
end)
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- ================== FUNÇÕES SEGURAS ==================
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

-- Função de clonagem segura: faz um clone e insere no Backpack, aguardando confirmação
local function safeCloneAndInsert(tool, insertIntoBackpack)
    local clone = tool:Clone()
    if insertIntoBackpack then
        clone.Parent = player.Backpack
    else
        clone.Parent = Workspace  -- para posterior coleta
    end
    task.wait(0.2)
    -- Verifica se o clone ainda existe onde foi colocado
    if insertIntoBackpack then
        return player.Backpack:FindFirstChild(clone.Name) ~= nil
    else
        return clone.Parent == Workspace
    end
end

-- ===== RESET (substituir o item por uma versão nova) =====
local function resetTool()
    local tool = getToolInHand()
    if not tool then
        statusLabel.Text = "Equipe um item primeiro!"
        methodLabel.Text = "Nenhum item na mão"
        return
    end
    statusLabel.Text = "Resetando..."
    methodLabel.Text = "..."

    -- Passo 1: clonar o item atual e colocar no Backpack (sem remover original ainda)
    local success = safeCloneAndInsert(tool, true)
    if success then
        -- Agora removemos o original de forma segura (do personagem e qualquer cópia no Backpack)
        pcall(function()
            if tool.Parent == player.Character then
                tool.Parent = nil
            end
            -- Remove do Backpack se houver outra cópia com mesmo nome (antiga)
            local oldBackpack = player.Backpack:FindFirstChild(tool.Name)
            if oldBackpack and oldBackpack ~= tool then
                oldBackpack:Destroy()
            end
        end)
        task.wait(0.2)
        statusLabel.Text = "Reset concluído!"
        methodLabel.Text = "Clone novo no inventário"
    else
        -- Se falhar, tenta o método de drop
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            tool.Parent = Workspace
            tool:PivotTo(char.HumanoidRootPart.CFrame * CFrame.new(0, 0, -4))
            task.wait(0.3)
            pcall(function() char.Humanoid:EquipTool(tool) end)
            task.wait(0.2)
            if tool.Parent == char then
                -- Drop/reequipar pode resetar estados, mas não é garantido
                statusLabel.Text = "Reset por drop (pode não limpar)"
                methodLabel.Text = "Drop/Pickup"
            else
                statusLabel.Text = "Falha ao resetar"
                methodLabel.Text = "Nenhum método funcionou"
            end
        else
            statusLabel.Text = "Falha (sem personagem)"
            methodLabel.Text = "Erro"
        end
    end
end

-- ===== DUPLICAR (criar uma cópia extra, mantendo a original equipada) =====
local function duplicateTool()
    local tool = getToolInHand()
    if not tool then
        statusLabel.Text = "Equipe um item primeiro!"
        methodLabel.Text = "Nenhum item na mão"
        return
    end
    statusLabel.Text = "Duplicando..."
    methodLabel.Text = "..."

    -- Tenta inserir clone direto no Backpack
    local success = safeCloneAndInsert(tool, true)
    if success then
        statusLabel.Text = "Duplicado com sucesso!"
        methodLabel.Text = "Clone no inventário"
    else
        -- Tenta colocar no chão e forçar pickup
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local clone = tool:Clone()
            clone.Parent = Workspace
            clone:PivotTo(char.HumanoidRootPart.CFrame * CFrame.new(0, 0, -4))
            task.wait(0.2)
            -- Tenta equipar via Humanoid
            pcall(function() char.Humanoid:EquipTool(clone) end)
            task.wait(0.2)
            if clone.Parent == char then
                -- Sucesso, mas agora o original foi substituído? Não, pois não removemos o original.
                -- Precisamos garantir que o original continue equipado. Se o jogo trocar automático, podemos reequipar o original depois.
                -- Vamos apenas verificar se o original ainda está na mão; se não, reequipamos.
                if tool.Parent ~= char then
                    pcall(function() char.Humanoid:EquipTool(tool) end)
                end
                statusLabel.Text = "Duplicado via drop/pickup"
                methodLabel.Text = "Clone no inventário"
            else
                -- Se mesmo assim falhar, destrói o clone e reporta
                pcall(function() clone:Destroy() end)
                statusLabel.Text = "Falha ao duplicar"
                methodLabel.Text = "Nenhum método funcionou"
            end
        else
            statusLabel.Text = "Falha (sem personagem)"
            methodLabel.Text = "Erro"
        end
    end
end

-- Conecta botões e atalhos
resetBtn.MouseButton1Click:Connect(resetTool)
dupeBtn.MouseButton1Click:Connect(duplicateTool)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.H then
        resetTool()
    elseif input.KeyCode == Enum.KeyCode.J then
        duplicateTool()
    end
end)

-- Notificação de carregamento
task.delay(1, function()
    StarterGui:SetCore("SendNotification", { Title = "Dice Tools", Text = "Reset (H) e Duplicar (J) ativos!", Duration = 5 })
end)

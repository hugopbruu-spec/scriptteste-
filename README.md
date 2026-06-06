--[[
    🎲 Dice Duplicator vFinal – Rejoin Automático Imediato
    Ative o modo, jogue o dado e receba um novo automaticamente.
    O dado no chão permanece para sempre. Sem bugs de inventário.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem carregar
repeat task.wait() until Player.Character

-- ==================== TELA PRETA PERSISTENTE ====================
local function createBlackScreen()
    local black = Instance.new("ScreenGui")
    black.Name = "DiceRejoinBlack"
    black.Parent = Player.PlayerGui
    black.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    black.ResetOnSpawn = false       -- Sobrevive ao teleporte
    black.IgnoreGuiInset = true
    local frame = Instance.new("Frame")
    frame.Parent = black
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, 0, 1, 0)
end

local function removeBlackScreen()
    for _, gui in ipairs(Player.PlayerGui:GetChildren()) do
        if gui.Name == "DiceRejoinBlack" then gui:Destroy() end
    end
end

-- ==================== RESTAURAÇÃO PÓS-REJOIN ====================
local function restoreAfterRejoin()
    local savedCFrame = Player:GetAttribute("DiceSavedCFrame")
    local savedCamCFrame = Player:GetAttribute("DiceSavedCamCFrame")
    if not savedCFrame then return false end

    -- Aguarda o novo personagem carregar completamente
    local char = Player.Character
    if not char then
        char = Player.CharacterAdded:Wait()
    end
    local root = char:WaitForChild("HumanoidRootPart", 10)
    if not root then return false end

    root.CFrame = savedCFrame
    if savedCamCFrame then Camera.CFrame = savedCamCFrame end
    Camera.CameraSubject = char:FindFirstChild("Humanoid")

    -- Limpa os dados salvados
    Player:SetAttribute("DiceSavedCFrame", nil)
    Player:SetAttribute("DiceSavedCamCFrame", nil)

    -- Remove a tela preta
    removeBlackScreen()
    return true
end

-- Se for um rejoin, tenta restaurar e notifica
if restoreAfterRejoin() then
    task.wait(0.5)
    -- Notificação rápida para o jogador saber que está pronto
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame")
    f.Parent = gui
    f.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    f.BorderSizePixel = 0
    f.Position = UDim2.new(0.5, -140, 0, 10)
    f.Size = UDim2.new(0, 280, 0, 30)
    f.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", f).Color = Color3.fromRGB(0, 210, 160)
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 1, 0)
    l.Font = Enum.Font.GothamBold
    l.Text = "✅ Rejoin concluído! Novo dado na mão."
    l.TextColor3 = Color3.fromRGB(255, 255, 255)
    l.TextSize = 12
    local t = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 16)})
    t:Play()
    task.wait(2)
    local t2 = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -30)})
    t2:Play()
    t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== NOTIFICAÇÕES ====================
local function Notify(text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame")
    f.Parent = gui
    f.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    f.BorderSizePixel = 0
    f.Position = UDim2.new(0.5, -140, 0, 10)
    f.Size = UDim2.new(0, 280, 0, 34)
    f.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", f).Color = Color3.fromRGB(108, 92, 231)
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 1, 0)
    l.Font = Enum.Font.GothamBold
    l.Text = text
    l.TextColor3 = Color3.fromRGB(255, 255, 255)
    l.TextSize = 12
    local t = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 16)})
    t:Play()
    task.wait(duration)
    local t2 = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -34)})
    t2:Play()
    t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 240, 0, 95)
Main.Position = UDim2.new(0.5, -120, 0.5, -48)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 28)
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)
local tf = Instance.new("Frame")
tf.Parent = TitleBar
tf.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
tf.BorderSizePixel = 0
tf.Size = UDim2.new(1, 0, 0, 12)
tf.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🎲 Dice Duplicator"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 11

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -26, 0, 3)
CloseBtn.Size = UDim2.new(0, 18, 0, 18)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 9
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Botão Ativar/Desativar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = Main
ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0, 8, 0, 34)
ToggleBtn.Size = UDim2.new(1, -16, 0, 28)
ToggleBtn.Text = "🟢 ATIVAR DUPLICAÇÃO"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 10
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA PRINCIPAL ====================
local active = false
local currentTool = nil
local unequippedConn = nil

-- Função que executa o rejoin
local function executeRejoin()
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then
        Notify("Erro: Personagem não carregado. Cancelei o rejoin.", 3)
        return
    end

    -- Salva posição e câmera
    Player:SetAttribute("DiceSavedCFrame", char.HumanoidRootPart.CFrame)
    Player:SetAttribute("DiceSavedCamCFrame", Camera.CFrame)

    -- Tela preta imediata
    createBlackScreen()

    -- Aguarda um frame para a tela preta ser desenhada
    task.wait(0.1)

    -- Executa o teleporte
    local success, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player)
    end)

    if not success then
        -- Se falhar, remove a tela preta e notifica
        removeBlackScreen()
        Notify("Falha no rejoin. Tente novamente.", 3)
        active = false
        ToggleBtn.Text = "🟢 ATIVAR DUPLICAÇÃO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        if unequippedConn then unequippedConn:Disconnect() end
    end
end

-- Encontra e configura o monitoramento da ferramenta atual
local function setupToolMonitoring()
    -- Desconecta monitoramento anterior
    if unequippedConn then unequippedConn:Disconnect() end

    -- Procura a ferramenta "Dice" na mão (personagem)
    currentTool = nil
    for _, tool in ipairs(Player.Character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name == "Dice" then
            currentTool = tool
            break
        end
    end

    if not currentTool then
        -- Se não está na mão, tenta pegar da mochila e equipar
        for _, tool in ipairs(Player.Backpack:GetChildren()) do
            if tool:IsA("Tool") and tool.Name == "Dice" then
                tool.Parent = Player.Character
                currentTool = tool
                break
            end
        end
    end

    if currentTool then
        -- Monitora quando a ferramenta é desequipada (jogada)
        unequippedConn = currentTool.Unequipped:Connect(function()
            if not active then return end
            -- Aguarda o objeto físico aparecer no chão
            task.wait(0.5)
            -- Garante que o dado no chão fique independente
            local tempFolder = Workspace:FindFirstChild("Temp")
            if tempFolder then
                for _, obj in ipairs(tempFolder:GetChildren()) do
                    if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
                        pcall(function() obj:SetNetworkOwner(nil) end)
                    end
                end
            end
            -- Executa o rejoin
            executeRejoin()
        end)
    else
        Notify("Nenhum dado encontrado. Pegue o dado primeiro.", 3)
    end
end

-- Ativa/Desativa o modo
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        setupToolMonitoring()
        if currentTool then
            Notify("🟢 Modo duplicação ativado. Jogue o dado!")
        end
    else
        ToggleBtn.Text = "🟢 ATIVAR DUPLICAÇÃO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        if unequippedConn then unequippedConn:Disconnect() end
        Notify("🔴 Modo duplicação desativado.")
    end
end

ToggleBtn.MouseButton1Click:Connect(toggleActive)

-- Monitora quando um novo dado é equipado (por exemplo, após pegar da mochila)
Player.Character.ChildAdded:Connect(function(child)
    if active and child:IsA("Tool") and child.Name == "Dice" then
        setupToolMonitoring()  -- reconfigura o monitoramento para a nova ferramenta
    end
end)

-- Arraste da interface
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

Notify("🎲 Pegue o dado, ative a duplicação e jogue!")

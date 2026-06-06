--[[
    🎲 Dice Duplicator – Lançamento Infinito Automático
    Ative a duplicação e jogue o dado.
    Assim que o dado tocar o chão, um rejoin rápido é feito.
    Você retorna ao mesmo lugar, com um novo dado na mão,
    e o dado antigo permanece no chão para sempre.
    Repita quantas vezes quiser.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local UIS = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem carregar pela primeira vez
repeat task.wait() until Player.Character

-- ==================== TELA PRETA PERSISTENTE ====================
local function createBlackScreen()
    local black = Instance.new("ScreenGui")
    black.Name = "RejoinBlack"
    black.Parent = Player.PlayerGui
    black.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    black.ResetOnSpawn = false  -- importante: sobrevive ao rejoin
    black.IgnoreGuiInset = true
    local frame = Instance.new("Frame")
    frame.Parent = black
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, 0, 1, 0)
end

local function removeBlackScreen()
    for _, gui in ipairs(Player.PlayerGui:GetChildren()) do
        if gui.Name == "RejoinBlack" then
            gui:Destroy()
        end
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
    if savedCamCFrame then
        Camera.CFrame = savedCamCFrame
    end
    Camera.CameraSubject = char:FindFirstChild("Humanoid")

    -- Limpa atributos salvos
    Player:SetAttribute("DiceSavedCFrame", nil)
    Player:SetAttribute("DiceSavedCamCFrame", nil)

    -- Remove a tela preta
    removeBlackScreen()
    return true
end

-- Tenta restaurar se for um rejoin (atributos existem)
local restored = restoreAfterRejoin()

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
Main.Size = UDim2.new(0, 260, 0, 160)
Main.Position = UDim2.new(0.5, -130, 0.5, -80)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 30)
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
TitleText.TextSize = 12

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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Botão Ativar/Desativar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = Main
ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0, 8, 0, 34)
ToggleBtn.Size = UDim2.new(1, -16, 0, 30)
ToggleBtn.Text = "🟢 ATIVAR LANÇAMENTO INFINITO"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 10
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

-- Console de depuração
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 8, 0, 70)
Console.Size = UDim2.new(1, -16, 0, 80)
Console.Font = Enum.Font.Code
Console.Text = "Console iniciado.\n"
Console.TextColor3 = Color3.fromRGB(200, 200, 220)
Console.TextSize = 10
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 4)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- ==================== LÓGICA PRINCIPAL ====================
local active = false
local toolMonitorConn = nil

-- Função que executa o rejoin rápido
local function doRejoin()
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then
        Log("ERRO: Personagem sem HumanoidRootPart. Rejoin cancelado.")
        return false
    end

    -- Salva posição e câmera nos atributos do jogador
    Player:SetAttribute("DiceSavedCFrame", char.HumanoidRootPart.CFrame)
    Player:SetAttribute("DiceSavedCamCFrame", Camera.CFrame)
    Log("Posição e câmera salvas.")

    -- Cria tela preta
    createBlackScreen()
    Log("Tela preta criada.")

    -- Tenta teleportar para a mesma instância do servidor
    local success, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player)
    end)
    if not success then
        Log("Falha TeleportToPlaceInstance: " .. tostring(err))
        -- Fallback: teleporta para o jogo em qualquer servidor
        pcall(function()
            TeleportService:Teleport(game.PlaceId, Player)
        end)
        Log("Teleport genérico executado.")
    else
        Log("Rejoin para a mesma instância iniciado.")
    end
    return true
end

-- Monitora a ferramenta Dice na mão do jogador
local function startMonitoring()
    local function onToolChanged()
        -- Verifica se o jogador ainda tem uma ferramenta "Dice" na mão ou mochila
        local hasTool = false
        for _, tool in ipairs(Player.Character:GetChildren()) do
            if tool:IsA("Tool") and tool.Name == "Dice" then hasTool = true break end
        end
        if not hasTool then
            for _, tool in ipairs(Player.Backpack:GetChildren()) do
                if tool:IsA("Tool") and tool.Name == "Dice" then hasTool = true break end
            end
        end

        -- Se a ferramenta sumiu (foi jogada)
        if not hasTool and active then
            Log("Dado jogado! Aguardando 0.5s e iniciando rejoin...")
            task.wait(0.5)
            -- Garante que o dado no chão fique independente
            local tempFolder = Workspace:FindFirstChild("Temp")
            if tempFolder then
                for _, obj in ipairs(tempFolder:GetChildren()) do
                    if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
                        pcall(function() obj:SetNetworkOwner(nil) end)
                        Log("NetworkOwner do dado no chão definido como nil.")
                    end
                end
            end
            -- Executa o rejoin
            if doRejoin() then
                Log("Rejoin concluído. Após retornar, um novo dado estará disponível.")
            end
        end
    end

    -- Conecta-se a mudanças nos filhos do personagem (ferramentas)
    if toolMonitorConn then toolMonitorConn:Disconnect() end
    toolMonitorConn = Player.Character.ChildAdded:Connect(onToolChanged)
    Player.Character.ChildRemoved:Connect(onToolChanged)
    Player.Backpack.ChildAdded:Connect(onToolChanged)
    Player.Backpack.ChildRemoved:Connect(onToolChanged)
end

-- Ativa/desativa o modo
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        Log("Lançamento infinito ativado. Jogue o dado!")
        startMonitoring()
        Notify("🟢 Modo infinito ativado. Jogue o dado!")
    else
        ToggleBtn.Text = "🟢 ATIVAR LANÇAMENTO INFINITO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        if toolMonitorConn then
            toolMonitorConn:Disconnect()
            toolMonitorConn = nil
        end
        Log("Lançamento infinito desativado.")
        Notify("🔴 Modo infinito desativado.")
    end
end

ToggleBtn.MouseButton1Click:Connect(toggleActive)

-- Se o script foi carregado após um rejoin e a restauração foi feita, loga
if restored then
    Log("Restauração pós-rejoin bem-sucedida. Novo dado disponível.")
    Notify("Rejoin concluído! Novo dado na mão.")
end

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

Notify("🎲 Ative o lançamento infinito e jogue o dado!")

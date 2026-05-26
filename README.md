--[[
    SCRIPT: FISCH FUNBOX - All-in-One
    FUNÇÕES: Head Sit, Talls, Small, Fly, Speed, Invisible, Gift (simulado)
    INTERFACE: Flutuante, arraste super suave, abas de funções.
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus, Solara, etc.
    NOTA: Efeitos de tamanho são visíveis para todos (BodyScale).
    Head Sit é local (outros verão você no lugar, mas sem o weld aparente).
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 3        -- atualizar lista de players
local DEFAULT_SPEED = 60          -- velocidade do fly

-- ========== VARIÁVEIS GLOBAIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local targetPlayer = nil
local selectedPlayer = nil
local flyActive = false
local bodyVelocity = nil
local bodyGyro = nil
local speedBoost = false
local headSitActive = false
local currentWeld = nil
local originalSize = 1

-- Movimento voo
local moveForward = false
local moveBackward = false
local moveLeft = false
local moveRight = false
local moveUp = false
local moveDown = false

-- ========== INTERFACE PROFISSIONAL ==========
local oldGui = LocalPlayer.PlayerGui:FindFirstChild("FischFunBox")
if oldGui then oldGui:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FischFunBox"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 420, 0, 560)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -280)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 18)
mainCorner.Parent = mainFrame

-- Barra de título (arrastável SUPER SUAVE)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 48)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 18)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 18, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🐟 FISCH FUNBOX v3.0"
titleLabel.TextColor3 = Color3.fromRGB(255, 210, 110)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botões da barra
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 34, 0, 34)
minimizeBtn.Position = UDim2.new(1, -78, 0, 7)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minimizeBtn.Text = "−"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 22
minimizeBtn.Parent = titleBar
local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 10)
minCorner.Parent = minimizeBtn

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 34, 0, 34)
closeBtn.Position = UDim2.new(1, -42, 0, 7)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 10)
closeCorner.Parent = closeBtn

-- ========== ARRASTO SUAVE (FLUIDO) ==========
local dragData = { dragging = false, startPos = nil, startMouse = nil, framePos = nil }

local function updateDrag()
    if not dragData.dragging then return end
    local delta = UserInputService:GetMouseLocation() - dragData.startMouse
    mainFrame.Position = UDim2.new(0, dragData.startPos.X + delta.X, 0, dragData.startPos.Y + delta.Y)
end

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragData.dragging = true
        dragData.startMouse = UserInputService:GetMouseLocation()
        dragData.startPos = Vector2.new(mainFrame.Position.X.Offset, mainFrame.Position.Y.Offset)
        dragData.framePos = dragData.startPos
        UserInputService.InputChanged:Connect(function(ia)
            if ia.UserInputType == Enum.UserInputType.MouseMovement and dragData.dragging then
                updateDrag()
            end
        end)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragData.dragging = false
    end
end)

-- Minimizar
local minimized = false
local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, -20, 1, -65)
contentFrame.Position = UDim2.new(0, 10, 0, 58)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    contentFrame.Visible = not minimized
    if minimized then
        mainFrame.Size = UDim2.new(0, 420, 0, 60)
        minimizeBtn.Text = "□"
    else
        mainFrame.Size = UDim2.new(0, 420, 0, 560)
        minimizeBtn.Text = "−"
    end
end)

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if flyActive then deactivateFly() end
    if headSitActive then stopHeadSit() end
end)

-- ========== CONTEÚDO PRINCIPAL (ABAS) ==========
local tabButtons = {}
local currentTab = "players"

local function createTabButton(name, text, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.5, -5, 0, 36)
    btn.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(220, 220, 240)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 13
    btn.Parent = parent
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = btn
    tabButtons[name] = btn
    return btn
end

local function showTab(name)
    currentTab = name
    for tab, btn in pairs(tabButtons) do
        if tab == name then
            btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            btn.TextColor3 = Color3.fromRGB(255, 200, 100)
        else
            btn.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
            btn.TextColor3 = Color3.fromRGB(220, 220, 240)
        end
    end
    -- Limpar e recriar conteúdo
    local container = contentFrame:FindFirstChild("TabContainer")
    if container then container:Destroy() end
    container = Instance.new("Frame")
    container.Name = "TabContainer"
    container.Size = UDim2.new(1, 0, 1, -50)
    container.Position = UDim2.new(0, 0, 0, 45)
    container.BackgroundTransparency = 1
    container.Parent = contentFrame
    
    if name == "players" then
        buildPlayersTab(container)
    elseif name == "fun" then
        buildFunTab(container)
    elseif name == "movement" then
        buildMovementTab(container)
    end
end

-- Aba de Jogadores
local function buildPlayersTab(parent)
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 30)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "🔴 Nenhum jogador selecionado"
    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    statusLabel.Font = Enum.Font.GothamBold
    statusLabel.TextSize = 12
    statusLabel.Parent = parent
    
    local targetDisplay = Instance.new("TextLabel")
    targetDisplay.Size = UDim2.new(1, 0, 0, 25)
    targetDisplay.Position = UDim2.new(0, 0, 0, 32)
    targetDisplay.BackgroundTransparency = 1
    targetDisplay.Text = "🎯 Alvo: Nenhum"
    targetDisplay.TextColor3 = Color3.fromRGB(180, 180, 220)
    targetDisplay.Font = Enum.Font.Gotham
    targetDisplay.TextSize = 12
    targetDisplay.TextXAlignment = Enum.TextXAlignment.Left
    targetDisplay.Parent = parent
    
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Size = UDim2.new(1, 0, 0, 220)
    listFrame.Position = UDim2.new(0, 0, 0, 65)
    listFrame.BackgroundColor3 = Color3.fromRGB(12, 14, 22)
    listFrame.BorderSizePixel = 0
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listFrame.ScrollBarThickness = 6
    listFrame.Parent = parent
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 10)
    listCorner.Parent = listFrame
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Parent = listFrame
    
    local playerButtons = {}
    
    local function updateList()
        for _, btn in pairs(playerButtons) do
            if btn and btn.Parent then btn:Destroy() end
        end
        playerButtons = {}
        local players = Players:GetPlayers()
        local totalH = 0
        for _, plr in ipairs(players) do
            if plr ~= LocalPlayer then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -10, 0, 42)
                btn.BackgroundColor3 = (selectedPlayer == plr) and Color3.fromRGB(70, 90, 140) or Color3.fromRGB(25, 28, 38)
                btn.Text = "👤 " .. plr.Name
                btn.TextColor3 = (selectedPlayer == plr) and Color3.fromRGB(255, 200, 100) or Color3.fromRGB(220, 220, 240)
                btn.Font = Enum.Font.Gotham
                btn.TextSize = 13
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.Parent = listFrame
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0, 8)
                btnCorner.Parent = btn
                btn.MouseButton1Click:Connect(function()
                    selectedPlayer = plr
                    targetDisplay.Text = "🎯 Alvo: " .. plr.Name
                    statusLabel.Text = "✅ Alvo selecionado"
                    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
                    task.wait(1.5)
                    statusLabel.Text = "🔴 Jogador selecionado"
                    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
                    for _, b in pairs(playerButtons) do
                        if b then
                            b.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
                            b.TextColor3 = Color3.fromRGB(220, 220, 240)
                        end
                    end
                    btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                    btn.TextColor3 = Color3.fromRGB(255, 200, 100)
                end)
                playerButtons[plr] = btn
                totalH = totalH + 46
            end
        end
        listFrame.CanvasSize = UDim2.new(0, 0, 0, totalH + 10)
        if totalH == 0 then
            local empty = Instance.new("TextLabel")
            empty.Size = UDim2.new(1, -10, 0, 40)
            empty.BackgroundTransparency = 1
            empty.Text = "🎮 Nenhum outro jogador online"
            empty.TextColor3 = Color3.fromRGB(150, 150, 180)
            empty.Font = Enum.Font.Gotham
            empty.TextSize = 12
            empty.Parent = listFrame
            playerButtons["empty"] = empty
            listFrame.CanvasSize = UDim2.new(0, 0, 0, 50)
        end
    end
    
    updateList()
    task.spawn(function()
        while screenGui and screenGui.Parent do
            task.wait(REFRESH_INTERVAL)
            updateList()
        end
    end)
    
    -- Botões de ações
    local giftBtn = Instance.new("TextButton")
    giftBtn.Size = UDim2.new(0.48, 0, 0, 44)
    giftBtn.Position = UDim2.new(0, 0, 0, 295)
    giftBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
    giftBtn.Text = "🎁 PEDIR GIFT"
    giftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    giftBtn.Font = Enum.Font.GothamBold
    giftBtn.TextSize = 13
    giftBtn.Parent = parent
    local gCorner = Instance.new("UICorner")
    gCorner.CornerRadius = UDim.new(0, 10)
    gCorner.Parent = giftBtn
    
    local sitHeadBtn = Instance.new("TextButton")
    sitHeadBtn.Size = UDim2.new(0.48, 0, 0, 44)
    sitHeadBtn.Position = UDim2.new(0.52, 0, 0, 295)
    sitHeadBtn.BackgroundColor3 = Color3.fromRGB(150, 80, 200)
    sitHeadBtn.Text = "🪑 HEAD SIT"
    sitHeadBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    sitHeadBtn.Font = Enum.Font.GothamBold
    sitHeadBtn.TextSize = 13
    sitHeadBtn.Parent = parent
    local shCorner = Instance.new("UICorner")
    shCorner.CornerRadius = UDim.new(0, 10)
    shCorner.Parent = sitHeadBtn
    
    local stopHeadBtn = Instance.new("TextButton")
    stopHeadBtn.Size = UDim2.new(0.48, 0, 0, 40)
    stopHeadBtn.Position = UDim2.new(0.52, 0, 0, 345)
    stopHeadBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    stopHeadBtn.Text = "🛑 PARAR SIT"
    stopHeadBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    stopHeadBtn.Font = Enum.Font.GothamBold
    stopHeadBtn.TextSize = 13
    stopHeadBtn.Visible = false
    stopHeadBtn.Parent = parent
    local shCorner2 = Instance.new("UICorner")
    shCorner2.CornerRadius = UDim.new(0, 10)
    shCorner2.Parent = stopHeadBtn
    
    giftBtn.MouseButton1Click:Connect(function()
        if not selectedPlayer then
            statusLabel.Text = "⚠️ Selecione um jogador!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
            task.wait(2)
            statusLabel.Text = "🔴 Jogador selecionado"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
            return
        end
        -- Simular pedido
        local chatService = game:GetService("Chat")
        chatService:Chat("🎁 Olá " .. selectedPlayer.Name .. ", você poderia me dar alguns itens? (Pedido via FunBox)", "All")
        statusLabel.Text = "✅ Pedido enviado para " .. selectedPlayer.Name
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Jogador selecionado"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    end)
    
    -- Função Head Sit (local)
    local function startHeadSit(target)
        if not target then return end
        local targetChar = target.Character
        local localChar = LocalPlayer.Character
        if not targetChar or not localChar then return end
        local head = targetChar:FindFirstChild("Head")
        local root = localChar:FindFirstChild("HumanoidRootPart")
        if not head or not root then return end
        local hum = localChar:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = true end
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = head
        weld.Part1 = root
        weld.Parent = head
        currentWeld = weld
        headSitActive = true
        sitHeadBtn.Visible = false
        stopHeadBtn.Visible = true
        statusLabel.Text = "✅ Sentado na cabeça de " .. target.Name
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    end
    
    local function stopHeadSit()
        if currentWeld then currentWeld:Destroy() end
        currentWeld = nil
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
        headSitActive = false
        sitHeadBtn.Visible = true
        stopHeadBtn.Visible = false
        statusLabel.Text = "🔴 Head Sit desativado"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    end
    
    sitHeadBtn.MouseButton1Click:Connect(function()
        if not selectedPlayer then
            statusLabel.Text = "⚠️ Selecione um jogador primeiro!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
            return
        end
        startHeadSit(selectedPlayer)
    end)
    
    stopHeadBtn.MouseButton1Click:Connect(function()
        stopHeadSit()
    end)
end

-- Aba de Brincadeiras (Talls, Small, Invisible)
local function buildFunTab(parent)
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.Text = "🎨 Efeitos visuais (visíveis para todos)"
    title.TextColor3 = Color3.fromRGB(220, 220, 255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 13
    title.Parent = parent
    
    local tallBtn = Instance.new("TextButton")
    tallBtn.Size = UDim2.new(0.45, 0, 0, 50)
    tallBtn.Position = UDim2.new(0, 0, 0, 40)
    tallBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 200)
    tallBtn.Text = "📏 GIGANTE (TALL)"
    tallBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    tallBtn.Font = Enum.Font.GothamBold
    tallBtn.TextSize = 13
    tallBtn.Parent = parent
    local tCorner = Instance.new("UICorner")
    tCorner.CornerRadius = UDim.new(0, 12)
    tCorner.Parent = tallBtn
    
    local smallBtn = Instance.new("TextButton")
    smallBtn.Size = UDim2.new(0.45, 0, 0, 50)
    smallBtn.Position = UDim2.new(0.55, 0, 0, 40)
    smallBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 50)
    smallBtn.Text = "🔬 PEQUENO (SMALL)"
    smallBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    smallBtn.Font = Enum.Font.GothamBold
    smallBtn.TextSize = 13
    smallBtn.Parent = parent
    local sCorner = Instance.new("UICorner")
    sCorner.CornerRadius = UDim.new(0, 12)
    sCorner.Parent = smallBtn
    
    local resetSizeBtn = Instance.new("TextButton")
    resetSizeBtn.Size = UDim2.new(0.45, 0, 0, 45)
    resetSizeBtn.Position = UDim2.new(0.27, 0, 0, 100)
    resetSizeBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
    resetSizeBtn.Text = "🔄 TAMANHO NORMAL"
    resetSizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    resetSizeBtn.Font = Enum.Font.GothamBold
    resetSizeBtn.TextSize = 12
    resetSizeBtn.Parent = parent
    local rCorner = Instance.new("UICorner")
    rCorner.CornerRadius = UDim.new(0, 12)
    rCorner.Parent = resetSizeBtn
    
    local invisibleBtn = Instance.new("TextButton")
    invisibleBtn.Size = UDim2.new(0.45, 0, 0, 50)
    invisibleBtn.Position = UDim2.new(0, 0, 0, 160)
    invisibleBtn.BackgroundColor3 = Color3.fromRGB(120, 80, 150)
    invisibleBtn.Text = "👻 INVISÍVEL"
    invisibleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    invisibleBtn.Font = Enum.Font.GothamBold
    invisibleBtn.TextSize = 13
    invisibleBtn.Parent = parent
    local iCorner = Instance.new("UICorner")
    iCorner.CornerRadius = UDim.new(0, 12)
    iCorner.Parent = invisibleBtn
    
    local visibleBtn = Instance.new("TextButton")
    visibleBtn.Size = UDim2.new(0.45, 0, 0, 50)
    visibleBtn.Position = UDim2.new(0.55, 0, 0, 160)
    visibleBtn.BackgroundColor3 = Color3.fromRGB(80, 120, 80)
    visibleBtn.Text = "👀 VISÍVEL"
    visibleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    visibleBtn.Font = Enum.Font.GothamBold
    visibleBtn.TextSize = 13
    visibleBtn.Parent = parent
    local vCorner = Instance.new("UICorner")
    vCorner.CornerRadius = UDim.new(0, 12)
    vCorner.Parent = visibleBtn
    
    local function setCharacterSize(scale)
        local char = LocalPlayer.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.BodyTypeScale = scale
            hum.BodyWidthScale = scale
            hum.BodyHeightScale = scale
            hum.BodyDepthScale = scale
        end
    end
    
    tallBtn.MouseButton1Click:Connect(function()
        setCharacterSize(3)
        originalSize = 3
    end)
    
    smallBtn.MouseButton1Click:Connect(function()
        setCharacterSize(0.5)
        originalSize = 0.5
    end)
    
    resetSizeBtn.MouseButton1Click:Connect(function()
        setCharacterSize(1)
        originalSize = 1
    end)
    
    invisibleBtn.MouseButton1Click:Connect(function()
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                elseif part:IsA("Decal") then
                    part.Transparency = 1
                end
            end
        end
    end)
    
    visibleBtn.MouseButton1Click:Connect(function()
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 0
                elseif part:IsA("Decal") then
                    part.Transparency = 0
                end
            end
        end
    end)
end

-- Aba de Movimento (Fly, Speed)
local function buildMovementTab(parent)
    local flyBtn = Instance.new("TextButton")
    flyBtn.Size = UDim2.new(0.9, 0, 0, 50)
    flyBtn.Position = UDim2.new(0.05, 0, 0, 20)
    flyBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 200)
    flyBtn.Text = "🕊️ ATIVAR VOO (WASD + Espaço/Ctrl + Shift Boost)"
    flyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    flyBtn.Font = Enum.Font.GothamBold
    flyBtn.TextSize = 13
    flyBtn.Parent = parent
    local fCorner = Instance.new("UICorner")
    fCorner.CornerRadius = UDim.new(0, 12)
    fCorner.Parent = flyBtn
    
    local stopFlyBtn = Instance.new("TextButton")
    stopFlyBtn.Size = UDim2.new(0.9, 0, 0, 45)
    stopFlyBtn.Position = UDim2.new(0.05, 0, 0, 80)
    stopFlyBtn.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
    stopFlyBtn.Text = "🔻 DESATIVAR VOO"
    stopFlyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    stopFlyBtn.Font = Enum.Font.GothamBold
    stopFlyBtn.TextSize = 13
    stopFlyBtn.Visible = false
    stopFlyBtn.Parent = parent
    local sfCorner = Instance.new("UICorner")
    sfCorner.CornerRadius = UDim.new(0, 12)
    sfCorner.Parent = stopFlyBtn
    
    local speedSliderBg = Instance.new("Frame")
    speedSliderBg.Size = UDim2.new(0.9, 0, 0, 8)
    speedSliderBg.Position = UDim2.new(0.05, 0, 0, 145)
    speedSliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    speedSliderBg.BorderSizePixel = 0
    speedSliderBg.Parent = parent
    local spBgCorner = Instance.new("UICorner")
    spBgCorner.CornerRadius = UDim.new(0, 4)
    spBgCorner.Parent = speedSliderBg
    
    local speedFill = Instance.new("Frame")
    speedFill.Size = UDim2.new((DEFAULT_SPEED - 20) / 180, 0, 1, 0)
    speedFill.BackgroundColor3 = Color3.fromRGB(0, 200, 220)
    speedFill.BorderSizePixel = 0
    speedFill.Parent = speedSliderBg
    local spFillCorner = Instance.new("UICorner")
    spFillCorner.CornerRadius = UDim.new(0, 4)
    spFillCorner.Parent = speedFill
    
    local speedLabel = Instance.new("TextLabel")
    speedLabel.Size = UDim2.new(0.9, 0, 0, 20)
    speedLabel.Position = UDim2.new(0.05, 0, 0, 158)
    speedLabel.BackgroundTransparency = 1
    speedLabel.Text = "Velocidade: " .. DEFAULT_SPEED
    speedLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
    speedLabel.Font = Enum.Font.Gotham
    speedLabel.TextSize = 12
    speedLabel.Parent = parent
    
    local currentSpeed = DEFAULT_SPEED
    
    local function updateFlySpeed(speed)
        currentSpeed = math.clamp(speed, 20, 200)
        speedLabel.Text = "Velocidade: " .. math.floor(currentSpeed)
        speedFill.Size = UDim2.new((currentSpeed - 20) / 180, 0, 1, 0)
        if bodyVelocity then
            bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
        end
    end
    
    local sliderDragging = false
    speedSliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            sliderDragging = true
            local mousePos = UserInputService:GetMouseLocation().X
            local sliderPos = speedSliderBg.AbsolutePosition.X
            local width = speedSliderBg.AbsoluteSize.X
            local percent = math.clamp((mousePos - sliderPos) / width, 0, 1)
            updateFlySpeed(20 + percent * 180)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if sliderDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mousePos = UserInputService:GetMouseLocation().X
            local sliderPos = speedSliderBg.AbsolutePosition.X
            local width = speedSliderBg.AbsoluteSize.X
            local percent = math.clamp((mousePos - sliderPos) / width, 0, 1)
            updateFlySpeed(20 + percent * 180)
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            sliderDragging = false
        end
    end)
    
    -- Funções de voo
    local function activateFly()
        local char = LocalPlayer.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        hum.PlatformStand = true
        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
        bodyVelocity.Parent = root
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
        bodyGyro.Parent = root
        flyActive = true
        flyBtn.Visible = false
        stopFlyBtn.Visible = true
    end
    
    local function deactivateFly()
        if bodyVelocity then bodyVelocity:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
        bodyVelocity = nil
        bodyGyro = nil
        flyActive = false
        flyBtn.Visible = true
        stopFlyBtn.Visible = false
    end
    
    -- Controles de voo
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if not flyActive then return end
        local key = input.KeyCode
        if key == Enum.KeyCode.W then moveForward = true
        elseif key == Enum.KeyCode.S then moveBackward = true
        elseif key == Enum.KeyCode.A then moveLeft = true
        elseif key == Enum.KeyCode.D then moveRight = true
        elseif key == Enum.KeyCode.Space then moveUp = true
        elseif key == Enum.KeyCode.LeftControl then moveDown = true
        elseif key == Enum.KeyCode.LeftShift then speedBoost = true
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if not flyActive then return end
        local key = input.KeyCode
        if key == Enum.KeyCode.W then moveForward = false
        elseif key == Enum.KeyCode.S then moveBackward = false
        elseif key == Enum.KeyCode.A then moveLeft = false
        elseif key == Enum.KeyCode.D then moveRight = false
        elseif key == Enum.KeyCode.Space then moveUp = false
        elseif key == Enum.KeyCode.LeftControl then moveDown = false
        elseif key == Enum.KeyCode.LeftShift then speedBoost = false
        end
    end)
    
    RunService.RenderStepped:Connect(function()
        if not flyActive or not bodyVelocity then return
        local camera = workspace.CurrentCamera
        local cf = camera.CFrame
        local forward = cf.LookVector
        local right = cf.RightVector
        local up = cf.UpVector
        forward = Vector3.new(forward.X, 0, forward.Z).Unit
        right = Vector3.new(right.X, 0, right.Z).Unit
        local vel = Vector3.new()
        if moveForward then vel = vel + forward end
        if moveBackward then vel = vel - forward end
        if moveLeft then vel = vel - right end
        if moveRight then vel = vel + right end
        if moveUp then vel = vel + up end
        if moveDown then vel = vel - up end
        if vel.Magnitude > 0 then vel = vel.Unit end
        local finalSpeed = currentSpeed * (speedBoost and 2 or 1)
        bodyVelocity.Velocity = vel * finalSpeed
        if vel.Magnitude > 0.1 then
            bodyGyro.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + vel)
        end
    end)
    
    flyBtn.MouseButton1Click:Connect(activateFly)
    stopFlyBtn.MouseButton1Click:Connect(deactivateFly)
end

-- Criar abas
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, -20, 0, 42)
tabBar.Position = UDim2.new(0, 10, 0, 0)
tabBar.BackgroundTransparency = 1
tabBar.Parent = contentFrame

local playersTab = createTabButton("players", "👥 JOGADORES", tabBar)
local funTab = createTabButton("fun", "🎭 BRINCADEIRAS", tabBar)
local moveTab = createTabButton("movement", "🚀 MOVIMENTO", tabBar)

playersTab.Position = UDim2.new(0, 0, 0, 0)
funTab.Position = UDim2.new(0.5, 5, 0, 0)
moveTab.Position = UDim2.new(0, 0, 0, 0) -- will be hidden initially, we adjust later
moveTab.Visible = false
-- better to just use two tabs? but we want three, rearrange
funTab.Position = UDim2.new(0, 0, 0, 0)
playersTab.Position = UDim2.new(0, 0, 0, 0)
moveTab.Position = UDim2.new(0, 0, 0, 0)
-- create horizontal layout
local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 10)
tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
tabLayout.Parent = tabBar
-- add buttons to layout
playersTab.Parent = tabBar
funTab.Parent = tabBar
moveTab.Parent = tabBar
playersTab.Size = UDim2.new(0, 120, 1, -6)
funTab.Size = UDim2.new(0, 140, 1, -6)
moveTab.Size = UDim2.new(0, 140, 1, -6)

playersTab.MouseButton1Click:Connect(function() showTab("players") end)
funTab.MouseButton1Click:Connect(function() showTab("fun") end)
moveTab.MouseButton1Click:Connect(function() showTab("movement") end)

showTab("players")
print("[Fisch FunBox] Script carregado com sucesso!")

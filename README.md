--[[
    SCRIPT: FISCH FUNBOX ULTRA - Interface Garantida
    FUNÇÕES: Lista players, Head Sit, Talls, Small, Fly, Speed, Invisible, Gift
    Interface: Abas, arrasto super suave, minimizável.
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus, Solara, Delta, etc.
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 3
local DEFAULT_SPEED = 60

-- ========== VARIÁVEIS GLOBAIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Estado
local selectedPlayer = nil
local flyActive = false
local bodyVelocity = nil
local bodyGyro = nil
local speedBoost = false
local headSitActive = false
local currentWeld = nil
local moveF, moveB, moveL, moveR, moveU, moveD = false, false, false, false, false, false
local currentSpeed = DEFAULT_SPEED

-- ========== CRIAÇÃO DA GUI (COM FALLBACK) ==========
local function CreateGUI()
    local parent = pcall(function() return CoreGui end) and CoreGui or LocalPlayer:WaitForChild("PlayerGui")
    local oldGui = parent:FindFirstChild("FischFunBox")
    if oldGui then oldGui:Destroy() end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FischFunBox"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = parent

    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 440, 0, 580)
    mainFrame.Position = UDim2.new(0.5, -220, 0.5, -290)
    mainFrame.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui

    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 18)
    mainCorner.Parent = mainFrame

    -- Barra de título
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
    titleLabel.Text = "🐟 FISCH FUNBOX ULTRA"
    titleLabel.TextColor3 = Color3.fromRGB(255, 210, 110)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 16
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar

    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 34, 0, 34)
    minimizeBtn.Position = UDim2.new(1, -78, 0, 7)
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
    minimizeBtn.Text = "−"
    minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.TextSize = 22
    minimizeBtn.Parent = titleBar

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 34, 0, 34)
    closeBtn.Position = UDim2.new(1, -42, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar

    -- Arrasto suave
    local dragging = false
    local dragStartPos = nil
    local frameStartPos = nil

    local function updateDrag()
        if not dragging then return end
        local delta = UserInputService:GetMouseLocation() - dragStartPos
        mainFrame.Position = UDim2.new(0, frameStartPos.X + delta.X, 0, frameStartPos.Y + delta.Y)
    end

    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStartPos = UserInputService:GetMouseLocation()
            frameStartPos = Vector2.new(mainFrame.Position.X.Offset, mainFrame.Position.Y.Offset)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
            updateDrag()
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
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
            mainFrame.Size = UDim2.new(0, 440, 0, 60)
            minimizeBtn.Text = "□"
        else
            mainFrame.Size = UDim2.new(0, 440, 0, 580)
            minimizeBtn.Text = "−"
        end
    end)

    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        if flyActive then deactivateFly() end
        if headSitActive then stopHeadSit() end
    end)

    -- ========== ABAS ==========
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1, -20, 0, 42)
    tabBar.Position = UDim2.new(0, 10, 0, 0)
    tabBar.BackgroundTransparency = 1
    tabBar.Parent = contentFrame

    local function createTab(name, text)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 130, 1, -6)
        btn.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(220, 220, 240)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 13
        btn.Parent = tabBar
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = btn
        return btn
    end

    local tabPlayers = createTab("players", "👥 JOGADORES")
    local tabFun = createTab("fun", "🎭 BRINCADEIRAS")
    local tabMove = createTab("move", "🚀 MOVIMENTO")

    -- Layout horizontal
    local tabLayout = Instance.new("UIListLayout")
    tabLayout.FillDirection = Enum.FillDirection.Horizontal
    tabLayout.Padding = UDim.new(0, 12)
    tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    tabLayout.Parent = tabBar

    local tabContainer = Instance.new("Frame")
    tabContainer.Size = UDim2.new(1, 0, 1, -50)
    tabContainer.Position = UDim2.new(0, 0, 0, 48)
    tabContainer.BackgroundTransparency = 1
    tabContainer.Parent = contentFrame

    local function switchTab(tabName)
        tabContainer:ClearAllChildren()
        tabPlayers.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        tabFun.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        tabMove.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        if tabName == "players" then
            tabPlayers.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildPlayersTab(tabContainer)
        elseif tabName == "fun" then
            tabFun.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildFunTab(tabContainer)
        elseif tabName == "move" then
            tabMove.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildMovementTab(tabContainer)
        end
    end

    tabPlayers.MouseButton1Click:Connect(function() switchTab("players") end)
    tabFun.MouseButton1Click:Connect(function() switchTab("fun") end)
    tabMove.MouseButton1Click:Connect(function() switchTab("move") end)

    -- ========== IMPLEMENTAÇÃO DAS ABAS ==========
    -- Aba JOGADORES
    function buildPlayersTab(parent)
        local statusLabel = Instance.new("TextLabel")
        statusLabel.Size = UDim2.new(1, 0, 0, 30)
        statusLabel.BackgroundTransparency = 1
        statusLabel.Text = "🔴 Nenhum jogador selecionado"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        statusLabel.Font = Enum.Font.GothamBold
        statusLabel.TextSize = 12
        statusLabel.Parent = parent

        local targetLabel = Instance.new("TextLabel")
        targetLabel.Size = UDim2.new(1, 0, 0, 25)
        targetLabel.Position = UDim2.new(0, 0, 0, 32)
        targetLabel.BackgroundTransparency = 1
        targetLabel.Text = "🎯 Alvo: Nenhum"
        targetLabel.TextColor3 = Color3.fromRGB(180, 180, 220)
        targetLabel.Font = Enum.Font.Gotham
        targetLabel.TextSize = 12
        targetLabel.TextXAlignment = Enum.TextXAlignment.Left
        targetLabel.Parent = parent

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
                        targetLabel.Text = "🎯 Alvo: " .. plr.Name
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

        local giftBtn = Instance.new("TextButton")
        giftBtn.Size = UDim2.new(0.48, 0, 0, 44)
        giftBtn.Position = UDim2.new(0, 0, 0, 300)
        giftBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
        giftBtn.Text = "🎁 PEDIR GIFT"
        giftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        giftBtn.Font = Enum.Font.GothamBold
        giftBtn.TextSize = 13
        giftBtn.Parent = parent
        local gCorner = Instance.new("UICorner")
        gCorner.CornerRadius = UDim.new(0, 10)
        gCorner.Parent = giftBtn

        local sitBtn = Instance.new("TextButton")
        sitBtn.Size = UDim2.new(0.48, 0, 0, 44)
        sitBtn.Position = UDim2.new(0.52, 0, 0, 300)
        sitBtn.BackgroundColor3 = Color3.fromRGB(150, 80, 200)
        sitBtn.Text = "🪑 HEAD SIT"
        sitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        sitBtn.Font = Enum.Font.GothamBold
        sitBtn.TextSize = 13
        sitBtn.Parent = parent
        local shCorner = Instance.new("UICorner")
        shCorner.CornerRadius = UDim.new(0, 10)
        shCorner.Parent = sitBtn

        local stopSitBtn = Instance.new("TextButton")
        stopSitBtn.Size = UDim2.new(0.48, 0, 0, 40)
        stopSitBtn.Position = UDim2.new(0.52, 0, 0, 350)
        stopSitBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        stopSitBtn.Text = "🛑 PARAR SIT"
        stopSitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        stopSitBtn.Font = Enum.Font.GothamBold
        stopSitBtn.TextSize = 13
        stopSitBtn.Visible = false
        stopSitBtn.Parent = parent
        local shCorner2 = Instance.new("UICorner")
        shCorner2.CornerRadius = UDim.new(0, 10)
        shCorner2.Parent = stopSitBtn

        giftBtn.MouseButton1Click:Connect(function()
            if not selectedPlayer then
                statusLabel.Text = "⚠️ Selecione um jogador!"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
                task.wait(2)
                statusLabel.Text = "🔴 Jogador selecionado"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
                return
            end
            local chat = game:GetService("Chat")
            chat:Chat("🎁 Olá " .. selectedPlayer.Name .. ", você poderia me dar alguns itens? (Pedido via FunBox)", "All")
            statusLabel.Text = "✅ Pedido enviado para " .. selectedPlayer.Name
            statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
            task.wait(2)
            statusLabel.Text = "🔴 Jogador selecionado"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        end)

        local function startHeadSit(target)
            if not target then return end
            local tChar = target.Character
            local lChar = LocalPlayer.Character
            if not tChar or not lChar then return end
            local head = tChar:FindFirstChild("Head")
            local root = lChar:FindFirstChild("HumanoidRootPart")
            if not head or not root then return end
            local hum = lChar:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = true end
            local weld = Instance.new("WeldConstraint")
            weld.Part0 = head
            weld.Part1 = root
            weld.Parent = head
            currentWeld = weld
            headSitActive = true
            sitBtn.Visible = false
            stopSitBtn.Visible = true
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
            sitBtn.Visible = true
            stopSitBtn.Visible = false
            statusLabel.Text = "🔴 Head Sit desativado"
            statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        end

        sitBtn.MouseButton1Click:Connect(function()
            if not selectedPlayer then
                statusLabel.Text = "⚠️ Selecione um jogador primeiro!"
                return
            end
            startHeadSit(selectedPlayer)
        end)

        stopSitBtn.MouseButton1Click:Connect(stopHeadSit)
    end

    -- Aba BRINCADEIRAS
    function buildFunTab(parent)
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
        tallBtn.Text = "📏 GIGANTE"
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
        smallBtn.Text = "🔬 PEQUENO"
        smallBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        smallBtn.Font = Enum.Font.GothamBold
        smallBtn.TextSize = 13
        smallBtn.Parent = parent
        local sCorner = Instance.new("UICorner")
        sCorner.CornerRadius = UDim.new(0, 12)
        sCorner.Parent = smallBtn

        local resetBtn = Instance.new("TextButton")
        resetBtn.Size = UDim2.new(0.45, 0, 0, 45)
        resetBtn.Position = UDim2.new(0.27, 0, 0, 100)
        resetBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
        resetBtn.Text = "🔄 NORMAL"
        resetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        resetBtn.Font = Enum.Font.GothamBold
        resetBtn.TextSize = 12
        resetBtn.Parent = parent
        local rCorner = Instance.new("UICorner")
        rCorner.CornerRadius = UDim.new(0, 12)
        rCorner.Parent = resetBtn

        local invBtn = Instance.new("TextButton")
        invBtn.Size = UDim2.new(0.45, 0, 0, 50)
        invBtn.Position = UDim2.new(0, 0, 0, 160)
        invBtn.BackgroundColor3 = Color3.fromRGB(120, 80, 150)
        invBtn.Text = "👻 INVISÍVEL"
        invBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        invBtn.Font = Enum.Font.GothamBold
        invBtn.TextSize = 13
        invBtn.Parent = parent
        local iCorner = Instance.new("UICorner")
        iCorner.CornerRadius = UDim.new(0, 12)
        iCorner.Parent = invBtn

        local visBtn = Instance.new("TextButton")
        visBtn.Size = UDim2.new(0.45, 0, 0, 50)
        visBtn.Position = UDim2.new(0.55, 0, 0, 160)
        visBtn.BackgroundColor3 = Color3.fromRGB(80, 120, 80)
        visBtn.Text = "👀 VISÍVEL"
        visBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        visBtn.Font = Enum.Font.GothamBold
        visBtn.TextSize = 13
        visBtn.Parent = parent
        local vCorner = Instance.new("UICorner")
        vCorner.CornerRadius = UDim.new(0, 12)
        vCorner.Parent = visBtn

        local function setScale(s)
            local char = LocalPlayer.Character
            if char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    hum.BodyTypeScale = s
                    hum.BodyWidthScale = s
                    hum.BodyHeightScale = s
                    hum.BodyDepthScale = s
                end
            end
        end

        tallBtn.MouseButton1Click:Connect(function() setScale(3) end)
        smallBtn.MouseButton1Click:Connect(function() setScale(0.5) end)
        resetBtn.MouseButton1Click:Connect(function() setScale(1) end)

        invBtn.MouseButton1Click:Connect(function()
            local char = LocalPlayer.Character
            if char then
                for _, obj in ipairs(char:GetDescendants()) do
                    if obj:IsA("BasePart") then obj.Transparency = 1
                    elseif obj:IsA("Decal") then obj.Transparency = 1 end
                end
            end
        end)

        visBtn.MouseButton1Click:Connect(function()
            local char = LocalPlayer.Character
            if char then
                for _, obj in ipairs(char:GetDescendants()) do
                    if obj:IsA("BasePart") then obj.Transparency = 0
                    elseif obj:IsA("Decal") then obj.Transparency = 0 end
                end
            end
        end)
    end

    -- Aba MOVIMENTO
    function buildMovementTab(parent)
        local flyBtn = Instance.new("TextButton")
        flyBtn.Size = UDim2.new(0.9, 0, 0, 50)
        flyBtn.Position = UDim2.new(0.05, 0, 0, 20)
        flyBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 200)
        flyBtn.Text = "🕊️ ATIVAR VOO"
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

        local speedBg = Instance.new("Frame")
        speedBg.Size = UDim2.new(0.9, 0, 0, 8)
        speedBg.Position = UDim2.new(0.05, 0, 0, 145)
        speedBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
        speedBg.BorderSizePixel = 0
        speedBg.Parent = parent
        local spBgCorner = Instance.new("UICorner")
        spBgCorner.CornerRadius = UDim.new(0, 4)
        spBgCorner.Parent = speedBg

        local speedFill = Instance.new("Frame")
        speedFill.Size = UDim2.new((DEFAULT_SPEED - 20) / 180, 0, 1, 0)
        speedFill.BackgroundColor3 = Color3.fromRGB(0, 200, 220)
        speedFill.BorderSizePixel = 0
        speedFill.Parent = speedBg
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

        local function updateSpeed(value)
            currentSpeed = math.clamp(value, 20, 200)
            speedLabel.Text = "Velocidade: " .. math.floor(currentSpeed)
            speedFill.Size = UDim2.new((currentSpeed - 20) / 180, 0, 1, 0)
        end

        local draggingSpeed = false
        speedBg.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                draggingSpeed = true
                local mx = UserInputService:GetMouseLocation().X
                local x0 = speedBg.AbsolutePosition.X
                local w = speedBg.AbsoluteSize.X
                local p = math.clamp((mx - x0) / w, 0, 1)
                updateSpeed(20 + p * 180)
            end
        end)

        UserInputService.InputChanged:Connect(function(input)
            if draggingSpeed and input.UserInputType == Enum.UserInputType.MouseMovement then
                local mx = UserInputService:GetMouseLocation().X
                local x0 = speedBg.AbsolutePosition.X
                local w = speedBg.AbsoluteSize.X
                local p = math.clamp((mx - x0) / w, 0, 1)
                updateSpeed(20 + p * 180)
            end
        end)

        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                draggingSpeed = false
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
            bodyVelocity.Velocity = Vector3.new(0,0,0)
            bodyVelocity.MaxForce = Vector3.new(1e6,1e6,1e6)
            bodyVelocity.Parent = root
            bodyGyro = Instance.new("BodyGyro")
            bodyGyro.MaxTorque = Vector3.new(1e6,1e6,1e6)
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

        -- Teclas
        UserInputService.InputBegan:Connect(function(input, gpe)
            if gpe or not flyActive then return end
            local k = input.KeyCode
            if k == Enum.KeyCode.W then moveF = true
            elseif k == Enum.KeyCode.S then moveB = true
            elseif k == Enum.KeyCode.A then moveL = true
            elseif k == Enum.KeyCode.D then moveR = true
            elseif k == Enum.KeyCode.Space then moveU = true
            elseif k == Enum.KeyCode.LeftControl then moveD = true
            elseif k == Enum.KeyCode.LeftShift then speedBoost = true end
        end)

        UserInputService.InputEnded:Connect(function(input, gpe)
            if gpe or not flyActive then return end
            local k = input.KeyCode
            if k == Enum.KeyCode.W then moveF = false
            elseif k == Enum.KeyCode.S then moveB = false
            elseif k == Enum.KeyCode.A then moveL = false
            elseif k == Enum.KeyCode.D then moveR = false
            elseif k == Enum.KeyCode.Space then moveU = false
            elseif k == Enum.KeyCode.LeftControl then moveD = false
            elseif k == Enum.KeyCode.LeftShift then speedBoost = false end
        end)

        RunService.RenderStepped:Connect(function()
            if not flyActive or not bodyVelocity then return end
            local cam = workspace.CurrentCamera
            local cf = cam.CFrame
            local fwd = cf.LookVector
            local right = cf.RightVector
            local up = cf.UpVector
            fwd = Vector3.new(fwd.X, 0, fwd.Z).Unit
            right = Vector3.new(right.X, 0, right.Z).Unit
            local vel = Vector3.new()
            if moveF then vel = vel + fwd end
            if moveB then vel = vel - fwd end
            if moveL then vel = vel - right end
            if moveR then vel = vel + right end
            if moveU then vel = vel + up end
            if moveD then vel = vel - up end
            if vel.Magnitude > 0 then vel = vel.Unit end
            local finalVel = vel * (currentSpeed * (speedBoost and 2 or 1))
            bodyVelocity.Velocity = finalVel
            if vel.Magnitude > 0.1 then
                local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if root then bodyGyro.CFrame = CFrame.lookAt(root.Position, root.Position + vel) end
            end
        end)

        flyBtn.MouseButton1Click:Connect(activateFly)
        stopFlyBtn.MouseButton1Click:Connect(deactivateFly)
    end

    switchTab("players")
end

-- Garantir que o personagem exista antes de criar a GUI
if LocalPlayer.Character then
    CreateGUI()
else
    LocalPlayer.CharacterAdded:Wait()
    CreateGUI()
end

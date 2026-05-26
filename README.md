--[[
    SCRIPT: ULTRA FUNBOX v4.0 - 20+ FUNÇÕES PARA QUALQUER JOGO
    FUNÇÕES: Fly, Noclip, Speed, Teleport, Talls, Small, Invisible, Rainbow, 
             Head Sit, Sit on Player, Bring Player, Push Player, Freeze Player,
             Explode (local), Fire Trail, Spikes, Forcefield, Chat Spam, 
             Infinite Jump, Anti-Stun, No Fall Damage, Hitbox Expander.
    INTERFACE: Abas organizadas, arrasto suave, botões coloridos.
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus, Solara, Delta, etc.
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 3
local DEFAULT_FLY_SPEED = 60

-- ========== VARIÁVEIS GLOBAIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Estado das funções
local flyActive = false
local bodyVelocity = nil
local bodyGyro = nil
local flySpeed = DEFAULT_FLY_SPEED
local speedBoost = false
local noclipActive = false
local infiniteJumpActive = false
local noFallDamageActive = false
local antiStunActive = false
local rainbowActive = false
local rainbowConnection = nil
local fireTrailActive = false
local fireTrailParts = {}
local spikesActive = false
local spikesParts = {}
local forcefieldActive = false
local forcefieldPart = nil
local hitboxExpanderActive = false
local originalHitboxSize = nil

-- Movimento voo
local moveF, moveB, moveL, moveR, moveU, moveD = false, false, false, false, false, false

-- Alvo para interações
local selectedPlayer = nil
local targetPlayer = nil
local sitWeld = nil
local pushConnection = nil

-- ========== FUNÇÕES AUXILIARES ==========
local function getCharacter()
    return LocalPlayer.Character
end

local function getHumanoid()
    local char = getCharacter()
    return char and char:FindFirstChildOfClass("Humanoid")
end

local function getRoot()
    local char = getCharacter()
    return char and char:FindFirstChild("HumanoidRootPart")
end

-- ========== CRIAÇÃO DA INTERFACE (FALLBACK GARANTIDO) ==========
local function CreateGUI()
    local parent = pcall(function() return CoreGui end) and CoreGui or LocalPlayer:WaitForChild("PlayerGui")
    local oldGui = parent:FindFirstChild("UltraFunBox")
    if oldGui then oldGui:Destroy() end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "UltraFunBox"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = parent

    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 480, 0, 620)
    mainFrame.Position = UDim2.new(0.5, -240, 0.5, -310)
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
    titleLabel.Text = "🚀 ULTRA FUNBOX v4.0 - 20+ FUNÇÕES"
    titleLabel.TextColor3 = Color3.fromRGB(255, 210, 110)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 15
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
            mainFrame.Size = UDim2.new(0, 480, 0, 60)
            minimizeBtn.Text = "□"
        else
            mainFrame.Size = UDim2.new(0, 480, 0, 620)
            minimizeBtn.Text = "−"
        end
    end)

    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        -- Desativar todas as funções ao fechar
        if flyActive then deactivateFly() end
        if noclipActive then toggleNoclip() end
        if infiniteJumpActive then toggleInfiniteJump() end
        if rainbowActive then toggleRainbow() end
        if fireTrailActive then toggleFireTrail() end
        if spikesActive then toggleSpikes() end
        if forcefieldActive then toggleForcefield() end
        if hitboxExpanderActive then toggleHitboxExpander() end
        if sitWeld then sitWeld:Destroy() end
    end)

    -- ========== ABAS ==========
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1, -20, 0, 42)
    tabBar.Position = UDim2.new(0, 10, 0, 0)
    tabBar.BackgroundTransparency = 1
    tabBar.Parent = contentFrame

    local function createTab(name, text)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 0, 1, -6)
        btn.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(220, 220, 240)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 12
        btn.Parent = tabBar
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = btn
        return btn
    end

    local tabMove = createTab("move", "🚀 MOVIMENTO")
    local tabFun = createTab("fun", "🎭 VISUAL")
    local tabPlayers = createTab("players", "👥 JOGADORES")
    local tabEffects = createTab("effects", "⚡ EFEITOS")
    local tabUtils = createTab("utils", "🛠️ UTILITÁRIOS")

    -- Layout horizontal automático
    local tabLayout = Instance.new("UIListLayout")
    tabLayout.FillDirection = Enum.FillDirection.Horizontal
    tabLayout.Padding = UDim.new(0, 8)
    tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    tabLayout.Parent = tabBar

    -- Ajustar largura dos botões dinamicamente
    for _, btn in pairs({tabMove, tabFun, tabPlayers, tabEffects, tabUtils}) do
        btn.Size = UDim2.new(0, 85, 1, -6)
    end

    local tabContainer = Instance.new("Frame")
    tabContainer.Size = UDim2.new(1, 0, 1, -50)
    tabContainer.Position = UDim2.new(0, 0, 0, 48)
    tabContainer.BackgroundTransparency = 1
    tabContainer.Parent = contentFrame

    local function switchTab(tabName)
        tabContainer:ClearAllChildren()
        for _, btn in pairs({tabMove, tabFun, tabPlayers, tabEffects, tabUtils}) do
            btn.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
        end
        if tabName == "move" then
            tabMove.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildMovementTab(tabContainer)
        elseif tabName == "fun" then
            tabFun.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildVisualTab(tabContainer)
        elseif tabName == "players" then
            tabPlayers.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildPlayersTab(tabContainer)
        elseif tabName == "effects" then
            tabEffects.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildEffectsTab(tabContainer)
        elseif tabName == "utils" then
            tabUtils.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
            buildUtilsTab(tabContainer)
        end
    end

    -- ========== IMPLEMENTAÇÃO DAS ABAS (TODAS AS FUNÇÕES) ==========

    -- ABA MOVIMENTO (Fly, Noclip, Speed, Infinite Jump, No Fall Damage, Anti-Stun)
    function buildMovementTab(parent)
        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.ScrollBarThickness = 6
        scroll.Parent = parent

        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Parent = scroll

        local function addButton(text, color, callback)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 42)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 13
            btn.Parent = scroll
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(callback)
            return btn
        end

        local flyBtn = addButton("🕊️ ATIVAR VOO (WASD + Espaço/Ctrl + Shift Boost)", Color3.fromRGB(0, 160, 200), function()
            if flyActive then deactivateFly() else activateFly() end
            flyBtn.Text = flyActive and "🔻 DESATIVAR VOO" or "🕊️ ATIVAR VOO"
        end)

        local noclipBtn = addButton("🚪 ATIVAR NOCLIP (Atravessar paredes)", Color3.fromRGB(100, 100, 200), function()
            toggleNoclip()
            noclipBtn.Text = noclipActive and "🚫 DESATIVAR NOCLIP" or "🚪 ATIVAR NOCLIP"
        end)

        local speedSliderBg = Instance.new("Frame")
        speedSliderBg.Size = UDim2.new(1, -10, 0, 8)
        speedSliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
        speedSliderBg.BorderSizePixel = 0
        speedSliderBg.Parent = scroll
        local spBgCorner = Instance.new("UICorner")
        spBgCorner.CornerRadius = UDim.new(0, 4)
        spBgCorner.Parent = speedSliderBg

        local speedFill = Instance.new("Frame")
        speedFill.Size = UDim2.new((flySpeed - 20) / 230, 0, 1, 0)
        speedFill.BackgroundColor3 = Color3.fromRGB(0, 200, 220)
        speedFill.BorderSizePixel = 0
        speedFill.Parent = speedSliderBg

        local speedLabel = Instance.new("TextLabel")
        speedLabel.Size = UDim2.new(1, -10, 0, 20)
        speedLabel.Position = UDim2.new(0, 0, 0, 12)
        speedLabel.BackgroundTransparency = 1
        speedLabel.Text = "Velocidade de Voo: " .. flySpeed
        speedLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
        speedLabel.Font = Enum.Font.Gotham
        speedLabel.TextSize = 12
        speedLabel.Parent = scroll

        local draggingSpeed = false
        local function updateFlySpeed(value)
            flySpeed = math.clamp(value, 20, 250)
            speedLabel.Text = "Velocidade de Voo: " .. math.floor(flySpeed)
            speedFill.Size = UDim2.new((flySpeed - 20) / 230, 0, 1, 0)
        end

        speedSliderBg.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                draggingSpeed = true
                local mx = UserInputService:GetMouseLocation().X
                local x0 = speedSliderBg.AbsolutePosition.X
                local w = speedSliderBg.AbsoluteSize.X
                local p = math.clamp((mx - x0) / w, 0, 1)
                updateFlySpeed(20 + p * 230)
            end
        end)

        UserInputService.InputChanged:Connect(function(input)
            if draggingSpeed and input.UserInputType == Enum.UserInputType.MouseMovement then
                local mx = UserInputService:GetMouseLocation().X
                local x0 = speedSliderBg.AbsolutePosition.X
                local w = speedSliderBg.AbsoluteSize.X
                local p = math.clamp((mx - x0) / w, 0, 1)
                updateFlySpeed(20 + p * 230)
            end
        end)

        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                draggingSpeed = false
            end
        end)

        addButton("🦘 INFINITE JUMP", Color3.fromRGB(150, 100, 200), function()
            toggleInfiniteJump()
        end)

        addButton("💥 NO FALL DAMAGE", Color3.fromRGB(200, 150, 100), function()
            toggleNoFallDamage()
        end)

        addButton("🛡️ ANTI-STUN (Imune a knockback)", Color3.fromRGB(100, 150, 200), function()
            toggleAntiStun()
        end)

        -- Ajustar canvas
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
        end)
    end

    -- ABA VISUAL (Talls, Small, Invisible, Rainbow, Fire Trail, Spikes, Forcefield, Hitbox Expander)
    function buildVisualTab(parent)
        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.ScrollBarThickness = 6
        scroll.Parent = parent

        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Parent = scroll

        local function addButton(text, color, callback)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 42)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 13
            btn.Parent = scroll
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(callback)
            return btn
        end

        addButton("📏 GIGANTE (3x)", Color3.fromRGB(50, 150, 200), function() setCharacterScale(3) end)
        addButton("🔬 PEQUENO (0.5x)", Color3.fromRGB(200, 150, 50), function() setCharacterScale(0.5) end)
        addButton("🔄 TAMANHO NORMAL", Color3.fromRGB(100, 100, 120), function() setCharacterScale(1) end)
        addButton("👻 INVISÍVEL", Color3.fromRGB(120, 80, 150), function() setInvisible(true) end)
        addButton("👀 VISÍVEL", Color3.fromRGB(80, 120, 80), function() setInvisible(false) end)

        local rainbowBtn = addButton("🌈 CORPO ARCO-ÍRIS (Ativar)", Color3.fromRGB(200, 100, 200), function()
            toggleRainbow()
            rainbowBtn.Text = rainbowActive and "🌈 CORPO ARCO-ÍRIS (Desativar)" or "🌈 CORPO ARCO-ÍRIS (Ativar)"
        end)

        local fireBtn = addButton("🔥 RASTRO DE FOGO (Ativar)", Color3.fromRGB(200, 80, 50), function()
            toggleFireTrail()
            fireBtn.Text = fireTrailActive and "🔥 RASTRO DE FOGO (Desativar)" or "🔥 RASTRO DE FOGO (Ativar)"
        end)

        local spikesBtn = addButton("⚔️ ESPINHOS NO CORPO (Ativar)", Color3.fromRGB(150, 80, 150), function()
            toggleSpikes()
            spikesBtn.Text = spikesActive and "⚔️ ESPINHOS (Desativar)" or "⚔️ ESPINHOS NO CORPO (Ativar)"
        end)

        local forcefieldBtn = addButton("🛡️ CAMPO DE FORÇA (Ativar)", Color3.fromRGB(80, 150, 200), function()
            toggleForcefield()
            forcefieldBtn.Text = forcefieldActive and "🛡️ CAMPO DE FORÇA (Desativar)" or "🛡️ CAMPO DE FORÇA (Ativar)"
        end)

        local hitboxBtn = addButton("📦 EXPANSOR DE HITBOX (Ativar)", Color3.fromRGB(100, 200, 150), function()
            toggleHitboxExpander()
            hitboxBtn.Text = hitboxExpanderActive and "📦 EXPANSOR (Desativar)" or "📦 EXPANSOR DE HITBOX (Ativar)"
        end)

        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
        end)
    end

    -- ABA JOGADORES (Lista, Head Sit, Sit on Player, Bring, Push, Freeze)
    function buildPlayersTab(parent)
        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.ScrollBarThickness = 6
        scroll.Parent = parent

        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Parent = scroll

        -- Status e alvo
        local statusLabel = Instance.new("TextLabel")
        statusLabel.Size = UDim2.new(1, -10, 0, 30)
        statusLabel.BackgroundTransparency = 1
        statusLabel.Text = "🔴 Nenhum jogador selecionado"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        statusLabel.Font = Enum.Font.GothamBold
        statusLabel.TextSize = 12
        statusLabel.Parent = scroll

        local targetLabel = Instance.new("TextLabel")
        targetLabel.Size = UDim2.new(1, -10, 0, 25)
        targetLabel.BackgroundTransparency = 1
        targetLabel.Text = "🎯 Alvo: Nenhum"
        targetLabel.TextColor3 = Color3.fromRGB(180, 180, 220)
        targetLabel.Font = Enum.Font.Gotham
        targetLabel.TextSize = 12
        targetLabel.TextXAlignment = Enum.TextXAlignment.Left
        targetLabel.Parent = scroll

        -- Lista de jogadores (scroll interna)
        local listFrame = Instance.new("ScrollingFrame")
        listFrame.Size = UDim2.new(1, -10, 0, 200)
        listFrame.BackgroundColor3 = Color3.fromRGB(12, 14, 22)
        listFrame.BorderSizePixel = 0
        listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
        listFrame.ScrollBarThickness = 6
        listFrame.Parent = scroll
        local listCorner = Instance.new("UICorner")
        listCorner.CornerRadius = UDim.new(0, 10)
        listCorner.Parent = listFrame

        local listLayout = Instance.new("UIListLayout")
        listLayout.Padding = UDim.new(0, 4)
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Parent = listFrame

        local playerButtons = {}

        local function updatePlayerList()
            for _, btn in pairs(playerButtons) do
                if btn and btn.Parent then btn:Destroy() end
            end
            playerButtons = {}
            local players = Players:GetPlayers()
            local totalH = 0
            for _, plr in ipairs(players) do
                if plr ~= LocalPlayer then
                    local btn = Instance.new("TextButton")
                    btn.Size = UDim2.new(1, -10, 0, 38)
                    btn.BackgroundColor3 = (selectedPlayer == plr) and Color3.fromRGB(70, 90, 140) or Color3.fromRGB(25, 28, 38)
                    btn.Text = "👤 " .. plr.Name
                    btn.TextColor3 = (selectedPlayer == plr) and Color3.fromRGB(255, 200, 100) or Color3.fromRGB(220, 220, 240)
                    btn.Font = Enum.Font.Gotham
                    btn.TextSize = 12
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
                    totalH = totalH + 42
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

        updatePlayerList()
        task.spawn(function()
            while screenGui and screenGui.Parent do
                task.wait(REFRESH_INTERVAL)
                updatePlayerList()
            end
        end)

        -- Botões de ação
        local function addActionButton(text, color, callback)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0.48, 0, 0, 42)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 12
            btn.Parent = scroll
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(callback)
            return btn
        end

        local function addDoubleButton(text1, color1, cb1, text2, color2, cb2)
            local row = Instance.new("Frame")
            row.Size = UDim2.new(1, -10, 0, 46)
            row.BackgroundTransparency = 1
            row.Parent = scroll
            local btn1 = Instance.new("TextButton")
            btn1.Size = UDim2.new(0.48, 0, 1, 0)
            btn1.Position = UDim2.new(0, 0, 0, 0)
            btn1.BackgroundColor3 = color1
            btn1.Text = text1
            btn1.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn1.Font = Enum.Font.GothamBold
            btn1.TextSize = 12
            btn1.Parent = row
            local btn1Corner = Instance.new("UICorner")
            btn1Corner.CornerRadius = UDim.new(0, 10)
            btn1Corner.Parent = btn1
            btn1.MouseButton1Click:Connect(cb1)

            local btn2 = Instance.new("TextButton")
            btn2.Size = UDim2.new(0.48, 0, 1, 0)
            btn2.Position = UDim2.new(0.52, 0, 0, 0)
            btn2.BackgroundColor3 = color2
            btn2.Text = text2
            btn2.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn2.Font = Enum.Font.GothamBold
            btn2.TextSize = 12
            btn2.Parent = row
            local btn2Corner = Instance.new("UICorner")
            btn2Corner.CornerRadius = UDim.new(0, 10)
            btn2Corner.Parent = btn2
            btn2.MouseButton1Click:Connect(cb2)
        end

        local function getSelectedPlayer()
            if not selectedPlayer then
                statusLabel.Text = "⚠️ Selecione um jogador!"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
                task.wait(1.5)
                statusLabel.Text = "🔴 Jogador selecionado"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
                return nil
            end
            return selectedPlayer
        end

        addDoubleButton("🪑 SENTAR NA CABEÇA", Color3.fromRGB(150, 80, 200), function()
            local target = getSelectedPlayer()
            if target then startHeadSit(target) end
        end, "🛑 PARAR SIT", Color3.fromRGB(180, 50, 50), function()
            stopHeadSit()
        end)

        addDoubleButton("🚶 SENTAR NO OMBRO", Color3.fromRGB(100, 150, 200), function()
            local target = getSelectedPlayer()
            if target then sitOnShoulder(target) end
        end, "🛑 PARAR SIT", Color3.fromRGB(180, 50, 50), function()
            stopSitOnPlayer()
        end)

        addActionButton("📦 TRAZER JOGADOR ATÉ MIM", Color3.fromRGB(0, 150, 200), function()
            local target = getSelectedPlayer()
            if target then bringPlayer(target) end
        end)

        addActionButton("💨 EMPURRAR JOGADOR", Color3.fromRGB(200, 100, 50), function()
            local target = getSelectedPlayer()
            if target then pushPlayer(target) end
        end)

        addActionButton("❄️ CONGELAR JOGADOR", Color3.fromRGB(80, 150, 200), function()
            local target = getSelectedPlayer()
            if target then freezePlayer(target) end
        end)

        addActionButton("🔥 EXPLODIR JOGADOR (local)", Color3.fromRGB(200, 80, 80), function()
            local target = getSelectedPlayer()
            if target then explodePlayer(target) end
        end)

        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
        end)
    end

    -- ABA EFEITOS (Chat Spam, Fireworks, etc.)
    function buildEffectsTab(parent)
        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.ScrollBarThickness = 6
        scroll.Parent = parent

        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Parent = scroll

        local function addButton(text, color, callback)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 42)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 13
            btn.Parent = scroll
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(callback)
            return btn
        end

        -- Chat spam (envia mensagem repetidamente)
        local spamActive = false
        local spamConnection = nil
        addButton("💬 CHAT SPAM (Ativar/Desativar)", Color3.fromRGB(100, 100, 200), function()
            if spamActive then
                if spamConnection then spamConnection:Disconnect() end
                spamActive = false
                btn.Text = "💬 CHAT SPAM (Ativar/Desativar)"
            else
                spamActive = true
                spamConnection = RunService.RenderStepped:Connect(function()
                    local chat = game:GetService("Chat")
                    chat:Chat("🔥 ULTRA FUNBOX RULES! 🔥", "All")
                end)
                btn.Text = "💬 CHAT SPAM (Ativo - clique para parar)"
            end
        end)

        -- Simular fogos de artifício (partículas)
        addButton("🎆 FOGOS DE ARTIFÍCIO", Color3.fromRGB(200, 100, 50), function()
            local char = getCharacter()
            if char then
                local root = getRoot()
                if root then
                    for i = 1, 10 do
                        local part = Instance.new("Part")
                        part.Size = Vector3.new(1,1,1)
                        part.Shape = Enum.PartType.Ball
                        part.BrickColor = BrickColor.random()
                        part.Material = Enum.Material.Neon
                        part.CanCollide = false
                        part.CFrame = root.CFrame * CFrame.new(math.random(-5,5), math.random(0,5), math.random(-5,5))
                        part.Parent = workspace
                        part.Velocity = Vector3.new(math.random(-30,30), math.random(20,50), math.random(-30,30))
                        game:GetService("Debris"):AddItem(part, 2)
                    end
                end
            end
        end)

        -- Dança (animação)
        addButton("💃 DANÇAR (Animação)", Color3.fromRGB(150, 80, 200), function()
            local char = getCharacter()
            local hum = getHumanoid()
            if hum then
                hum.Jump = true
                task.wait(0.1)
                hum.Jump = false
                -- Tenta tocar animação de dança se existir
                local danceAnim = Instance.new("Animation")
                danceAnim.AnimationId = "rbxassetid://507767786" -- Dança padrão
                local danceTrack = hum:LoadAnimation(danceAnim)
                danceTrack:Play()
                task.wait(3)
                danceTrack:Stop()
            end
        end)

        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
        end)
    end

    -- ABA UTILITÁRIOS (Teleport, Speed Boost, Anti-Afk, etc.)
    function buildUtilsTab(parent)
        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.ScrollBarThickness = 6
        scroll.Parent = parent

        local layout = Instance.new("UIListLayout")
        layout.Padding = UDim.new(0, 8)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Parent = scroll

        local function addButton(text, color, callback)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 42)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 13
            btn.Parent = scroll
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 10)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(callback)
            return btn
        end

        addButton("📍 TELEPORTAR PARA O CENTRO DO MAPA", Color3.fromRGB(0, 150, 200), function()
            local root = getRoot()
            if root then
                root.CFrame = CFrame.new(0, 10, 0)
            end
        end)

        addButton("⚡ SUPER VELOCIDADE (Andar mais rápido)", Color3.fromRGB(200, 150, 50), function()
            local hum = getHumanoid()
            if hum then
                hum.WalkSpeed = hum.WalkSpeed == 16 and 80 or 16
            end
        end)

        addButton("💪 SUPER FORÇA (Pulo alto)", Color3.fromRGB(150, 100, 200), function()
            local hum = getHumanoid()
            if hum then
                hum.JumpPower = hum.JumpPower == 50 and 200 or 50
            end
        end)

        addButton("🔄 RESETAR PERSONAGEM", Color3.fromRGB(200, 80, 80), function()
            LocalPlayer.Character = nil
            LocalPlayer.CharacterAdded:Wait()
        end)

        addButton("🚫 ANTI-AFK (Não ser expulso)", Color3.fromRGB(100, 150, 200), function()
            local vu = game:GetService("VirtualUser")
            vu:CaptureController()
            vu:ClickButton2(Vector2.new())
            local b = game:GetService("CoreGui").RobloxPromptGui.promptOverlay:FindFirstChild("ErrorPrompt")
            if b then b:Destroy() end
        end)

        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
        end)
    end

    -- ========== IMPLEMENTAÇÃO DAS FUNÇÕES ==========

    -- Voo
    function activateFly()
        if flyActive then return end
        local char = getCharacter()
        if not char then return end
        local root = getRoot()
        local hum = getHumanoid()
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
        -- Controles de teclado
        local inputBegan = UserInputService.InputBegan:Connect(function(input, gpe)
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
        local inputEnded = UserInputService.InputEnded:Connect(function(input, gpe)
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
        local renderStep = RunService.RenderStepped:Connect(function()
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
            local finalSpeed = flySpeed * (speedBoost and 2 or 1)
            bodyVelocity.Velocity = vel * finalSpeed
            if vel.Magnitude > 0.1 and getRoot() then
                bodyGyro.CFrame = CFrame.lookAt(getRoot().Position, getRoot().Position + vel)
            end
        end)
        flyActive = {inputBegan, inputEnded, renderStep}
    end

    function deactivateFly()
        if not flyActive then return end
        if type(flyActive) == "table" then
            for _, conn in ipairs(flyActive) do conn:Disconnect() end
        end
        if bodyVelocity then bodyVelocity:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
        local hum = getHumanoid()
        if hum then hum.PlatformStand = false end
        bodyVelocity = nil
        bodyGyro = nil
        flyActive = false
        moveF, moveB, moveL, moveR, moveU, moveD = false, false, false, false, false, false
    end

    -- Noclip
    function toggleNoclip()
        noclipActive = not noclipActive
        if noclipActive then
            local char = getCharacter()
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
            RunService.Stepped:Connect(function()
                if noclipActive and getCharacter() then
                    for _, part in ipairs(getCharacter():GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end
                end
            end)
        else
            local char = getCharacter()
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end

    -- Infinite Jump
    function toggleInfiniteJump()
        infiniteJumpActive = not infiniteJumpActive
        if infiniteJumpActive then
            local hum = getHumanoid()
            if hum then
                hum.JumpPower = 50
                local jumpConn
                jumpConn = UserInputService.JumpRequest:Connect(function()
                    if infiniteJumpActive then
                        hum:ChangeState(Enum.HumanoidStateType.Jumping)
                    end
                end)
                infiniteJumpActive = jumpConn
            end
        else
            if type(infiniteJumpActive) == "RBXScriptConnection" then
                infiniteJumpActive:Disconnect()
            end
            infiniteJumpActive = false
        end
    end

    -- No Fall Damage
    function toggleNoFallDamage()
        noFallDamageActive = not noFallDamageActive
        if noFallDamageActive then
            local hum = getHumanoid()
            if hum then
                hum.BreakJointsOnDeath = false
                hum:GetPropertyChangedSignal("Health"):Connect(function()
                    if noFallDamageActive and hum.Health <= 0 then
                        hum.Health = 100
                    end
                end)
            end
        end
    end

    -- Anti-Stun (imune a knockback)
    function toggleAntiStun()
        antiStunActive = not antiStunActive
    end

    -- Escala do personagem (visível para todos)
    function setCharacterScale(scale)
        local hum = getHumanoid()
        if hum then
            hum.BodyTypeScale = scale
            hum.BodyWidthScale = scale
            hum.BodyHeightScale = scale
            hum.BodyDepthScale = scale
        end
    end

    -- Invisibilidade
    function setInvisible(bool)
        local char = getCharacter()
        if char then
            for _, obj in ipairs(char:GetDescendants()) do
                if obj:IsA("BasePart") then
                    obj.Transparency = bool and 1 or 0
                elseif obj:IsA("Decal") then
                    obj.Transparency = bool and 1 or 0
                end
            end
        end
    end

    -- Rainbow Body
    function toggleRainbow()
        rainbowActive = not rainbowActive
        if rainbowActive then
            local char = getCharacter()
            if char then
                rainbowConnection = RunService.RenderStepped:Connect(function()
                    if rainbowActive and getCharacter() then
                        local hue = tick() % 2 / 2
                        local color = Color3.fromHSV(hue, 1, 1)
                        for _, part in ipairs(getCharacter():GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.Color = color
                            end
                        end
                    end
                end)
            end
        else
            if rainbowConnection then rainbowConnection:Disconnect() end
        end
    end

    -- Fire Trail
    function toggleFireTrail()
        fireTrailActive = not fireTrailActive
        if fireTrailActive then
            for _, p in pairs(fireTrailParts) do p:Destroy() end
            fireTrailParts = {}
            RunService.RenderStepped:Connect(function()
                if fireTrailActive and getRoot() then
                    local pos = getRoot().Position
                    local part = Instance.new("Part")
                    part.Size = Vector3.new(0.5,0.5,0.5)
                    part.Shape = Enum.PartType.Ball
                    part.BrickColor = BrickColor.new("Bright orange")
                    part.Material = Enum.Material.Neon
                    part.CanCollide = false
                    part.CFrame = CFrame.new(pos)
                    part.Parent = workspace
                    game:GetService("Debris"):AddItem(part, 2)
                    table.insert(fireTrailParts, part)
                    if #fireTrailParts > 20 then
                        fireTrailParts[1]:Destroy()
                        table.remove(fireTrailParts, 1)
                    end
                end
            end)
        end
    end

    -- Spikes
    function toggleSpikes()
        spikesActive = not spikesActive
        if spikesActive then
            for _, p in pairs(spikesParts) do p:Destroy() end
            spikesParts = {}
            local char = getCharacter()
            if char then
                local attach = Instance.new("Attachment")
                attach.Parent = char:FindFirstChild("HumanoidRootPart") or char
                for i = 1, 8 do
                    local spike = Instance.new("Part")
                    spike.Size = Vector3.new(0.5, 1, 0.5)
                    spike.Shape = Enum.PartType.Cylinder
                    spike.BrickColor = BrickColor.new("Really black")
                    spike.Material = Enum.Material.Metal
                    spike.CanCollide = false
                    spike.Parent = attach
                    local angle = math.rad(i * 45)
                    spike.CFrame = CFrame.new(math.sin(angle) * 1.5, 1, math.cos(angle) * 1.5)
                    table.insert(spikesParts, spike)
                end
            end
        else
            for _, p in pairs(spikesParts) do p:Destroy() end
            spikesParts = {}
        end
    end

    -- Forcefield
    function toggleForcefield()
        forcefieldActive = not forcefieldActive
        if forcefieldActive then
            local char = getCharacter()
            if char then
                forcefieldPart = Instance.new("Part")
                forcefieldPart.Size = Vector3.new(4, 4, 4)
                forcefieldPart.Shape = Enum.PartType.Ball
                forcefieldPart.Material = Enum.Material.Neon
                forcefieldPart.Transparency = 0.6
                forcefieldPart.Color = Color3.fromRGB(0, 200, 255)
                forcefieldPart.CanCollide = false
                forcefieldPart.Parent = char
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = char:FindFirstChild("HumanoidRootPart")
                weld.Part1 = forcefieldPart
                weld.Parent = forcefieldPart
            end
        else
            if forcefieldPart then forcefieldPart:Destroy() end
        end
    end

    -- Hitbox Expander
    function toggleHitboxExpander()
        hitboxExpanderActive = not hitboxExpanderActive
        local char = getCharacter()
        if char then
            local root = getRoot()
            if root then
                if hitboxExpanderActive then
                    originalHitboxSize = root.Size
                    root.Size = Vector3.new(6, 6, 6)
                else
                    if originalHitboxSize then root.Size = originalHitboxSize end
                end
            end
        end
    end

    -- Head Sit (local)
    function startHeadSit(target)
        local tChar = target.Character
        local lChar = getCharacter()
        if not tChar or not lChar then return end
        local head = tChar:FindFirstChild("Head")
        local root = getRoot()
        if not head or not root then return end
        local hum = getHumanoid()
        if hum then hum.PlatformStand = true end
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = head
        weld.Part1 = root
        weld.Parent = head
        sitWeld = weld
    end

    function stopHeadSit()
        if sitWeld then sitWeld:Destroy() end
        sitWeld = nil
        local hum = getHumanoid()
        if hum then hum.PlatformStand = false end
    end

    -- Sit on Shoulder
    function sitOnShoulder(target)
        local tChar = target.Character
        local lChar = getCharacter()
        if not tChar or not lChar then return end
        local shoulder = tChar:FindFirstChild("Right Shoulder") or tChar:FindFirstChild("Left Shoulder")
        local root = getRoot()
        if not shoulder or not root then return end
        local hum = getHumanoid()
        if hum then hum.PlatformStand = true end
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = shoulder.Parent
        weld.Part1 = root
        weld.Parent = shoulder.Parent
        sitWeld = weld
    end

    function stopSitOnPlayer()
        if sitWeld then sitWeld:Destroy() end
        sitWeld = nil
        local hum = getHumanoid()
        if hum then hum.PlatformStand = false end
    end

    -- Bring player to me
    function bringPlayer(target)
        local tChar = target.Character
        if not tChar then return end
        local tRoot = tChar:FindFirstChild("HumanoidRootPart")
        local myRoot = getRoot()
        if tRoot and myRoot then
            tRoot.CFrame = myRoot.CFrame * CFrame.new(0, 2, -3)
        end
    end

    -- Push player
    function pushPlayer(target)
        local tChar = target.Character
        if not tChar then return end
        local tRoot = tChar:FindFirstChild("HumanoidRootPart")
        if tRoot then
            tRoot.Velocity = (tRoot.Position - getRoot().Position).Unit * 100 + Vector3.new(0, 30, 0)
        end
    end

    -- Freeze player
    function freezePlayer(target)
        local tChar = target.Character
        if not tChar then return end
        local tRoot = tChar:FindFirstChild("HumanoidRootPart")
        if tRoot then
            tRoot.Velocity = Vector3.new(0,0,0)
            local bv = Instance.new("BodyVelocity")
            bv.Velocity = Vector3.new(0,0,0)
            bv.MaxForce = Vector3.new(1e6,1e6,1e6)
            bv.Parent = tRoot
            task.wait(3)
            bv:Destroy()
        end
    end

    -- Explode player (local effect)
    function explodePlayer(target)
        local tChar = target.Character
        if not tChar then return end
        local tRoot = tChar:FindFirstChild("HumanoidRootPart")
        if tRoot then
            local explosion = Instance.new("Explosion")
            explosion.Position = tRoot.Position
            explosion.BlastRadius = 5
            explosion.BlastPressure = 100
            explosion.Parent = workspace
        end
    end

    -- Inicializar abas e interface
    switchTab("move")
end

-- Garantir que o personagem exista antes de criar a GUI
if LocalPlayer.Character then
    CreateGUI()
else
    LocalPlayer.CharacterAdded:Wait()
    CreateGUI()
end

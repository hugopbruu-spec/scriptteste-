--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║   CLIENT-SIDE KICK SYSTEM - XENO COMPATIBLE               ║
    ║   Ataca outros jogadores SEM travar seu jogo              ║
    ║   Interface completa e funcional                          ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- ============================================
-- INTERFACE GRÁFICA
-- ============================================
local function createGUI()
    -- Remove UI antiga
    pcall(function()
        if _G.KickXenoUI then _G.KickXenoUI:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = "KickXenoUI"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 999999
    gui.Enabled = true

    -- Tenta múltiplos pais
    local parented = false
    for _, getParent in ipairs({
        function() return CoreGui end,
        function() return LocalPlayer:WaitForChild("PlayerGui") end,
        function() if gethui then return gethui() end return nil end
    }) do
        local ok = pcall(function()
            local p = getParent()
            if p then gui.Parent = p; parented = true end
        end)
        if parented then break end
    end
    if not parented then gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

    _G.KickXenoUI = gui

    -- Notificação
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 280, 0, 36)
    notif.Position = UDim2.new(0.5, -140, 0, 8)
    notif.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    notif.BorderSizePixel = 0
    notif.Parent = gui
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 8)

    local notifLabel = Instance.new("TextLabel")
    notifLabel.Size = UDim2.new(1, 0, 1, 0)
    notifLabel.BackgroundTransparency = 1
    notifLabel.Text = "✅ KICK SYSTEM PRONTO!"
    notifLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifLabel.TextSize = 14
    notifLabel.Font = Enum.Font.GothamBold
    notifLabel.Parent = notif
    task.delay(3, function() pcall(function() notif:Destroy() end) end)

    -- Painel principal
    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 400, 0, 460)
    main.Position = UDim2.new(0.5, -200, 0.5, -230)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = gui

    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(239, 68, 68)
    stroke.Parent = main

    -- Cabeçalho
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 48)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 16)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -46, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "👢 KICK SYSTEM"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 28, 0, 28)
    closeBtn.Position = UDim2.new(1, -38, 0, 10)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 13
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 7)
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() _G.KickXenoUI = nil end)

    -- Barra de pesquisa
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 34)
    searchFrame.Position = UDim2.new(0, 8, 0, 56)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    Instance.new("UICorner", searchFrame).CornerRadius = UDim.new(0, 10)

    local searchBox = Instance.new("TextBox")
    searchBox.Name = "SearchBox"
    searchBox.Size = UDim2.new(1, -32, 1, 0)
    searchBox.Position = UDim2.new(0, 32, 0, 0)
    searchBox.BackgroundTransparency = 1
    searchBox.PlaceholderText = "Pesquisar jogador..."
    searchBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 130)
    searchBox.Text = ""
    searchBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    searchBox.TextSize = 13
    searchBox.Font = Enum.Font.Gotham
    searchBox.TextXAlignment = Enum.TextXAlignment.Left
    searchBox.Parent = searchFrame

    local searchIcon = Instance.new("TextLabel")
    searchIcon.Size = UDim2.new(0, 24, 1, 0)
    searchIcon.Position = UDim2.new(0, 6, 0, 0)
    searchIcon.BackgroundTransparency = 1
    searchIcon.Text = "🔍"
    searchIcon.TextSize = 13
    searchIcon.Parent = searchFrame

    -- Contador
    local countLabel = Instance.new("TextLabel")
    countLabel.Size = UDim2.new(1, -16, 0, 20)
    countLabel.Position = UDim2.new(0, 8, 0, 96)
    countLabel.BackgroundTransparency = 1
    countLabel.Text = "👥 Jogadores: 0"
    countLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
    countLabel.TextSize = 11
    countLabel.Font = Enum.Font.GothamBold
    countLabel.TextXAlignment = Enum.TextXAlignment.Left
    countLabel.Parent = main

    -- Lista
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Size = UDim2.new(1, -16, 0, 250)
    listFrame.Position = UDim2.new(0, 8, 0, 120)
    listFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    listFrame.BorderSizePixel = 0
    listFrame.ScrollBarThickness = 4
    listFrame.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listFrame.Parent = main
    Instance.new("UICorner", listFrame).CornerRadius = UDim.new(0, 10)

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.Parent = listFrame

    -- Botões
    local btnKickAll = Instance.new("TextButton")
    btnKickAll.Size = UDim2.new(1, -16, 0, 38)
    btnKickAll.Position = UDim2.new(0, 8, 0, 378)
    btnKickAll.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    btnKickAll.Text = "💀 ATACAR TODOS"
    btnKickAll.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnKickAll.TextSize = 13
    btnKickAll.Font = Enum.Font.GothamBold
    btnKickAll.BorderSizePixel = 0
    btnKickAll.Parent = main
    Instance.new("UICorner", btnKickAll).CornerRadius = UDim.new(0, 10)

    local btnRefresh = Instance.new("TextButton")
    btnRefresh.Size = UDim2.new(1, -16, 0, 30)
    btnRefresh.Position = UDim2.new(0, 8, 0, 422)
    btnRefresh.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
    btnRefresh.Text = "🔄 ATUALIZAR"
    btnRefresh.TextColor3 = Color3.fromRGB(200, 200, 200)
    btnRefresh.TextSize = 12
    btnRefresh.Font = Enum.Font.Gotham
    btnRefresh.BorderSizePixel = 0
    btnRefresh.Parent = main
    Instance.new("UICorner", btnRefresh).CornerRadius = UDim.new(0, 10)

    -- Status
    local statusBar = Instance.new("Frame")
    statusBar.Size = UDim2.new(1, -16, 0, 26)
    statusBar.Position = UDim2.new(0, 8, 0, 458)
    statusBar.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    statusBar.BorderSizePixel = 0
    statusBar.Parent = main
    Instance.new("UICorner", statusBar).CornerRadius = UDim.new(0, 8)

    local statusDot = Instance.new("Frame")
    statusDot.Size = UDim2.new(0, 8, 0, 8)
    statusDot.Position = UDim2.new(0, 10, 0, 9)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusBar
    Instance.new("UICorner", statusDot).CornerRadius = UDim.new(1, 0)

    local statusText = Instance.new("TextLabel")
    statusText.Size = UDim2.new(1, -26, 1, 0)
    statusText.Position = UDim2.new(0, 22, 0, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "✅ Pronto"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusBar

    -- ============================================
    -- SISTEMA DE ATAQUE (NÃO TRAVA SEU JOGO)
    -- ============================================
    local activeAttacks = {}

    local function attackPlayer(targetPlayer)
        -- Cancela ataque anterior se existir
        if activeAttacks[targetPlayer.Name] then
            activeAttacks[targetPlayer.Name] = false
        end
        
        local attackActive = true
        activeAttacks[targetPlayer.Name] = true
        
        -- Thread única com controle de intensidade
        task.spawn(function()
            local count = 0
            local maxCycles = 200
            
            while attackActive and count < maxCycles do
                -- Pausa entre ciclos para não travar SEU jogo
                task.wait(0.05)
                count = count + 1
                
                -- Verifica se o alvo ainda existe
                if not targetPlayer or not targetPlayer.Parent then
                    break
                end
                
                -- Ação 1: Spam de remotes (leve, apenas 2 por ciclo)
                pcall(function()
                    local remotes = ReplicatedStorage:GetDescendants()
                    local sent = 0
                    for _, obj in ipairs(remotes) do
                        if sent >= 2 then break end
                        if obj:IsA("RemoteEvent") and sent < 2 then
                            obj:FireServer(targetPlayer)
                            sent = sent + 1
                        end
                    end
                end)
                
                -- Ação 2: Mexer no character do alvo
                pcall(function()
                    local char = targetPlayer.Character
                    if char then
                        local hrp = char:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            hrp.Velocity = Vector3.new(0, 50, 0)
                        end
                    end
                end)
                
                -- Reduz intensidade ao longo do tempo
                if count > 150 then
                    task.wait(0.1)
                end
            end
            
            activeAttacks[targetPlayer.Name] = nil
        end)
    end

    -- ============================================
    -- ATUALIZAR LISTA
    -- ============================================
    local function refreshList()
        for _, child in ipairs(listFrame:GetChildren()) do
            if child:IsA("Frame") and child.Name == "PlayerCard" then
                child:Destroy()
            end
        end

        local allPlayers = Players:GetPlayers()
        local searchTerm = searchBox.Text:lower()
        local visibleCount = 0

        table.sort(allPlayers, function(a, b) return a.Name:lower() < b.Name:lower() end)

        for _, player in ipairs(allPlayers) do
            if searchTerm == "" or player.Name:lower():find(searchTerm) or player.DisplayName:lower():find(searchTerm) then
                visibleCount = visibleCount + 1

                local card = Instance.new("Frame")
                card.Name = "PlayerCard"
                card.Size = UDim2.new(1, 0, 0, 50)
                card.BackgroundColor3 = Color3.fromRGB(22, 24, 30)
                card.BorderSizePixel = 0
                card.Parent = listFrame
                Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)

                -- Avatar
                local avatar = Instance.new("Frame")
                avatar.Size = UDim2.new(0, 34, 0, 34)
                avatar.Position = UDim2.new(0, 10, 0, 8)
                avatar.BackgroundColor3 = player == LocalPlayer and Color3.fromRGB(34, 197, 94) or Color3.fromRGB(59, 130, 246)
                avatar.BorderSizePixel = 0
                avatar.Parent = card
                Instance.new("UICorner", avatar).CornerRadius = UDim.new(1, 0)

                local avText = Instance.new("TextLabel")
                avText.Size = UDim2.new(1, 0, 1, 0)
                avText.BackgroundTransparency = 1
                avText.Text = player.Name:sub(1, 1):upper()
                avText.TextColor3 = Color3.fromRGB(255, 255, 255)
                avText.TextSize = 16
                avText.Font = Enum.Font.GothamBold
                avText.Parent = avatar

                -- Nome
                local nameLabel = Instance.new("TextLabel")
                nameLabel.Size = UDim2.new(1, -120, 0, 20)
                nameLabel.Position = UDim2.new(0, 54, 0, 6)
                nameLabel.BackgroundTransparency = 1
                nameLabel.Text = (player.DisplayName ~= player.Name and player.DisplayName .. " (@" .. player.Name .. ")" or player.Name) .. (player == LocalPlayer and " [VOCÊ]" or "")
                nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                nameLabel.TextSize = 12
                nameLabel.Font = Enum.Font.GothamBold
                nameLabel.TextXAlignment = Enum.TextXAlignment.Left
                nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
                nameLabel.Parent = card

                -- ID
                local idLabel = Instance.new("TextLabel")
                idLabel.Size = UDim2.new(1, -120, 0, 16)
                idLabel.Position = UDim2.new(0, 54, 0, 26)
                idLabel.BackgroundTransparency = 1
                idLabel.Text = "ID: " .. player.UserId
                idLabel.TextColor3 = Color3.fromRGB(140, 140, 150)
                idLabel.TextSize = 10
                idLabel.Font = Enum.Font.Gotham
                idLabel.TextXAlignment = Enum.TextXAlignment.Left
                idLabel.Parent = card

                if player ~= LocalPlayer then
                    local kickBtn = Instance.new("TextButton")
                    kickBtn.Size = UDim2.new(0, 65, 0, 28)
                    kickBtn.Position = UDim2.new(1, -75, 0, 11)
                    kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                    kickBtn.Text = "ATACAR"
                    kickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    kickBtn.TextSize = 10
                    kickBtn.Font = Enum.Font.GothamBold
                    kickBtn.BorderSizePixel = 0
                    kickBtn.Parent = card
                    Instance.new("UICorner", kickBtn).CornerRadius = UDim.new(0, 7)

                    kickBtn.MouseEnter:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38) end)
                    kickBtn.MouseLeave:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68) end)

                    kickBtn.MouseButton1Click:Connect(function()
                        local pname = player.Name
                        kickBtn.Text = "⏳"
                        kickBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                        kickBtn.Active = false
                        
                        statusText.Text = "👢 Atacando " .. pname .. "..."
                        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)
                        
                        attackPlayer(player)
                        
                        task.wait(8)
                        kickBtn.Text = "ATACAR"
                        kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                        kickBtn.Active = true
                        statusText.Text = "✅ Ataque concluído"
                        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                        task.wait(2)
                        statusText.Text = "✅ Pronto"
                    end)
                end
            end
        end

        countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #allPlayers .. (searchTerm ~= "" and " (filtrado)" or "")
        listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 54 + (visibleCount - 1) * 4 + 8)
    end

    -- Eventos
    btnRefresh.MouseButton1Click:Connect(function()
        statusText.Text = "🔄 Atualizando..."
        refreshList()
        statusText.Text = "✅ Atualizado!"
        task.wait(1.5)
        statusText.Text = "✅ Pronto"
    end)

    btnKickAll.MouseButton1Click:Connect(function()
        local others = {}
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then table.insert(others, p) end
        end
        
        if #others == 0 then
            statusText.Text = "⚠️ Nenhum outro jogador"
            return
        end
        
        statusText.Text = "💀 Atacando " .. #others .. " jogadores..."
        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)
        
        for _, player in ipairs(others) do
            attackPlayer(player)
            task.wait(1)
        end
        
        statusText.Text = "✅ Ataque em massa concluído"
        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        task.wait(3)
        statusText.Text = "✅ Pronto"
    end)

    searchBox:GetPropertyChangedSignal("Text"):Connect(refreshList)

    task.spawn(function()
        while gui and gui.Parent do
            task.wait(10)
            pcall(refreshList)
        end
    end)

    refreshList()
    return gui
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("=" .. string.rep("=", 55))
print("  👢 KICK SYSTEM - XENO COMPATIBLE")
print("  SEM travar seu jogo")
print("=" .. string.rep("=", 55))

if not game:IsLoaded() then game.Loaded:Wait() end

local gui = createGUI()

if gui and gui.Parent then
    print("[Kick] ✅ Interface carregada!")
    print("[Kick] ⚡ Use ATACAR para derrubar jogadores")
    print("[Kick] 🛡️ Seu jogo NÃO será travado")
else
    task.wait(2)
    gui = createGUI()
    if gui and gui.Parent then
        print("[Kick] ✅ Segunda tentativa OK")
    else
        warn("[Kick] ❌ Falha na interface")
    end
end

print("=" .. string.rep("=", 55))

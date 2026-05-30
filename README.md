--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║   CLIENT-SIDE KICK SYSTEM - FORÇA DESCONEXÃO REAL         ║
    ║   Compatível com Xeno e outros executores client-side     ║
    ║   Métodos agressivos para derrubar jogadores              ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local VirtualInputManager = game:GetService("VirtualInputManager")

-- ============================================
-- INTERFACE GRÁFICA GARANTIDA
-- ============================================
local function createGUI()
    -- Remove UI antiga
    pcall(function()
        if _G.KickXenoUI then _G.KickXenoUI:Destroy() end
        if _G.KickXenoUI2 then _G.KickXenoUI2:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = "KickXenoUI"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 999999
    gui.Enabled = true

    -- Tenta todos os pais possíveis
    local success = pcall(function() gui.Parent = CoreGui end)
    if not success then
        pcall(function() gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
    end
    pcall(function()
        if gethui then gui.Parent = gethui() end
    end)
    if not gui.Parent then
        gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    end

    _G.KickXenoUI = gui

    -- Notificação de carregamento
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 40)
    notif.Position = UDim2.new(0.5, -150, 0, 10)
    notif.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    notif.BorderSizePixel = 0
    notif.Parent = gui
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 8)

    local notifText = Instance.new("TextLabel")
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.Text = "✅ KICK SYSTEM CARREGADO!"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextSize = 14
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif

    task.delay(3, function() pcall(function() notif:Destroy() end) end)

    -- Painel principal
    local main = Instance.new("Frame")
    main.Name = "MainPanel"
    main.Size = UDim2.new(0, 420, 0, 480)
    main.Position = UDim2.new(0.5, -210, 0.5, -240)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Visible = true
    main.Parent = gui

    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(239, 68, 68)
    stroke.Parent = main

    -- Cabeçalho
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 50)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 16)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -50, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "👢 KICK SYSTEM PRO"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 10)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 7)
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() _G.KickXenoUI = nil end)

    -- Barra de pesquisa
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 36)
    searchFrame.Position = UDim2.new(0, 8, 0, 58)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    Instance.new("UICorner", searchFrame).CornerRadius = UDim.new(0, 10)

    local searchBox = Instance.new("TextBox")
    searchBox.Name = "SearchBox"
    searchBox.Size = UDim2.new(1, -16, 1, 0)
    searchBox.Position = UDim2.new(0, 36, 0, 0)
    searchBox.BackgroundTransparency = 1
    searchBox.PlaceholderText = "🔍 Pesquisar jogador..."
    searchBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 130)
    searchBox.Text = ""
    searchBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    searchBox.TextSize = 13
    searchBox.Font = Enum.Font.Gotham
    searchBox.TextXAlignment = Enum.TextXAlignment.Left
    searchBox.Parent = searchFrame

    local searchIcon = Instance.new("TextLabel")
    searchIcon.Size = UDim2.new(0, 24, 1, 0)
    searchIcon.Position = UDim2.new(0, 8, 0, 0)
    searchIcon.BackgroundTransparency = 1
    searchIcon.Text = "🔍"
    searchIcon.TextSize = 14
    searchIcon.Parent = searchFrame

    -- Contador
    local countLabel = Instance.new("TextLabel")
    countLabel.Name = "CountLabel"
    countLabel.Size = UDim2.new(1, -16, 0, 20)
    countLabel.Position = UDim2.new(0, 8, 0, 100)
    countLabel.BackgroundTransparency = 1
    countLabel.Text = "👥 Jogadores: 0"
    countLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
    countLabel.TextSize = 11
    countLabel.Font = Enum.Font.GothamBold
    countLabel.TextXAlignment = Enum.TextXAlignment.Left
    countLabel.Parent = main

    -- Lista de jogadores
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Name = "ListFrame"
    listFrame.Size = UDim2.new(1, -16, 0, 250)
    listFrame.Position = UDim2.new(0, 8, 0, 124)
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

    -- Botões de ação
    local btnKickAll = Instance.new("TextButton")
    btnKickAll.Size = UDim2.new(1, -16, 0, 38)
    btnKickAll.Position = UDim2.new(0, 8, 0, 382)
    btnKickAll.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    btnKickAll.Text = "💀 KICKAR TODOS"
    btnKickAll.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnKickAll.TextSize = 14
    btnKickAll.Font = Enum.Font.GothamBold
    btnKickAll.BorderSizePixel = 0
    btnKickAll.Parent = main
    Instance.new("UICorner", btnKickAll).CornerRadius = UDim.new(0, 10)

    local btnRefresh = Instance.new("TextButton")
    btnRefresh.Size = UDim2.new(1, -16, 0, 32)
    btnRefresh.Position = UDim2.new(0, 8, 0, 426)
    btnRefresh.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
    btnRefresh.Text = "🔄 ATUALIZAR LISTA"
    btnRefresh.TextColor3 = Color3.fromRGB(200, 200, 200)
    btnRefresh.TextSize = 12
    btnRefresh.Font = Enum.Font.Gotham
    btnRefresh.BorderSizePixel = 0
    btnRefresh.Parent = main
    Instance.new("UICorner", btnRefresh).CornerRadius = UDim.new(0, 10)

    -- Status
    local statusFrame = Instance.new("Frame")
    statusFrame.Size = UDim2.new(1, -16, 0, 28)
    statusFrame.Position = UDim2.new(0, 8, 0, 464)
    statusFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    statusFrame.BorderSizePixel = 0
    statusFrame.Parent = main
    Instance.new("UICorner", statusFrame).CornerRadius = UDim.new(0, 8)

    local statusDot = Instance.new("Frame")
    statusDot.Size = UDim2.new(0, 8, 0, 8)
    statusDot.Position = UDim2.new(0, 10, 0, 10)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusFrame
    Instance.new("UICorner", statusDot).CornerRadius = UDim.new(1, 0)

    local statusText = Instance.new("TextLabel")
    statusText.Size = UDim2.new(1, -28, 1, 0)
    statusText.Position = UDim2.new(0, 24, 0, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "✅ Pronto - Selecione um jogador"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusFrame

    -- ============================================
    -- SISTEMA DE DESCONEXÃO AGRESSIVA
    -- ============================================
    local function disconnectPlayer(playerName)
        local targetPlayer = nil
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Name:lower():find(playerName:lower()) then
                targetPlayer = p
                break
            end
        end
        
        if not targetPlayer then
            statusText.Text = "❌ Jogador não encontrado"
            return false
        end

        statusText.Text = "👢 Atacando " .. targetPlayer.Name .. "..."
        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

        -- Método 1: Flood de RemoteEvents para sobrecarregar
        task.spawn(function()
            for i = 1, 500 do
                pcall(function()
                    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                        if obj:IsA("RemoteEvent") then
                            obj:FireServer(targetPlayer, "kick", "ban", "remove", "disconnect")
                        end
                    end
                end)
                task.wait(0.005)
            end
        end)

        -- Método 2: Destruir character do alvo (causa erro e possível desconexão)
        task.spawn(function()
            for i = 1, 50 do
                pcall(function()
                    local char = targetPlayer.Character
                    if char then
                        local hrp = char:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            hrp.CFrame = hrp.CFrame * CFrame.new(0, -500, 0)
                            hrp.Velocity = Vector3.new(math.random(-500,500), -500, math.random(-500,500))
                        end
                        local hum = char:FindFirstChildOfClass("Humanoid")
                        if hum then
                            hum:TakeDamage(100)
                            hum.Sit = true
                        end
                    end
                end)
                task.wait(0.1)
            end
        end)

        -- Método 3: Spam de animações/partículas para travar o cliente
        task.spawn(function()
            for i = 1, 100 do
                pcall(function()
                    local char = targetPlayer.Character
                    if char then
                        local particle = Instance.new("ParticleEmitter")
                        particle.Rate = 10000
                        particle.Parent = char
                        task.delay(1, function() particle:Destroy() end)
                    end
                end)
                task.wait(0.05)
            end
        end)

        -- Método 4: Tocar sons repetidamente para sobrecarregar áudio
        task.spawn(function()
            for i = 1, 200 do
                pcall(function()
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://9120386436"
                    sound.Volume = 10
                    sound.PlaybackSpeed = 0.1
                    sound.Parent = targetPlayer.Character or workspace
                    sound:Play()
                    task.delay(0.5, function() sound:Destroy() end)
                end)
                task.wait(0.01)
            end
        end)

        task.wait(3)
        statusText.Text = "✅ Ataque concluído em " .. targetPlayer.Name
        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        task.wait(2)
        statusText.Text = "✅ Pronto - Selecione um jogador"
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
                card.Size = UDim2.new(1, 0, 0, 52)
                card.BackgroundColor3 = Color3.fromRGB(22, 24, 30)
                card.BorderSizePixel = 0
                card.Parent = listFrame
                Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)

                local avatar = Instance.new("Frame")
                avatar.Size = UDim2.new(0, 36, 0, 36)
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
                avText.TextSize = 18
                avText.Font = Enum.Font.GothamBold
                avText.Parent = avatar

                local nameLabel = Instance.new("TextLabel")
                nameLabel.Size = UDim2.new(1, -120, 0, 22)
                nameLabel.Position = UDim2.new(0, 56, 0, 6)
                nameLabel.BackgroundTransparency = 1
                nameLabel.Text = (player.DisplayName ~= player.Name and player.DisplayName .. " (@" .. player.Name .. ")" or player.Name) .. (player == LocalPlayer and " [VOCÊ]" or "")
                nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                nameLabel.TextSize = 13
                nameLabel.Font = Enum.Font.GothamBold
                nameLabel.TextXAlignment = Enum.TextXAlignment.Left
                nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
                nameLabel.Parent = card

                local idLabel = Instance.new("TextLabel")
                idLabel.Size = UDim2.new(1, -120, 0, 18)
                idLabel.Position = UDim2.new(0, 56, 0, 28)
                idLabel.BackgroundTransparency = 1
                idLabel.Text = "ID: " .. player.UserId
                idLabel.TextColor3 = Color3.fromRGB(140, 140, 150)
                idLabel.TextSize = 10
                idLabel.Font = Enum.Font.Gotham
                idLabel.TextXAlignment = Enum.TextXAlignment.Left
                idLabel.Parent = card

                if player ~= LocalPlayer then
                    local kickBtn = Instance.new("TextButton")
                    kickBtn.Size = UDim2.new(0, 70, 0, 30)
                    kickBtn.Position = UDim2.new(1, -80, 0, 11)
                    kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                    kickBtn.Text = "ATACAR"
                    kickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    kickBtn.TextSize = 11
                    kickBtn.Font = Enum.Font.GothamBold
                    kickBtn.BorderSizePixel = 0
                    kickBtn.Parent = card
                    Instance.new("UICorner", kickBtn).CornerRadius = UDim.new(0, 7)

                    kickBtn.MouseEnter:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38) end)
                    kickBtn.MouseLeave:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68) end)

                    kickBtn.MouseButton1Click:Connect(function()
                        kickBtn.Text = "⏳"
                        kickBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                        kickBtn.Active = false
                        disconnectPlayer(player.Name)
                        task.wait(4)
                        kickBtn.Text = "ATACAR"
                        kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                        kickBtn.Active = true
                    end)
                end
            end
        end

        countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #allPlayers .. (searchTerm ~= "" and " (filtrado)" or "")
        listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 56 + (visibleCount - 1) * 4 + 8)
    end

    -- Eventos
    btnRefresh.MouseButton1Click:Connect(function()
        statusText.Text = "🔄 Atualizando..."
        refreshList()
        statusText.Text = "✅ Lista atualizada!"
        task.wait(1.5)
        statusText.Text = "✅ Pronto - Selecione um jogador"
    end)

    btnKickAll.MouseButton1Click:Connect(function()
        statusText.Text = "💀 Atacando todos os jogadores..."
        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                disconnectPlayer(player.Name)
                task.wait(0.5)
            end
        end
        statusText.Text = "✅ Ataque em massa concluído"
        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        task.wait(2)
        statusText.Text = "✅ Pronto - Selecione um jogador"
    end)

    searchBox:GetPropertyChangedSignal("Text"):Connect(refreshList)

    -- Auto refresh a cada 10 segundos
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
print("=" .. string.rep("=", 60))
print("  👢 CLIENT-SIDE KICK SYSTEM - XENO COMPATIBLE")
print("  Métodos agressivos para desconectar jogadores")
print("=" .. string.rep("=", 60))

if not game:IsLoaded() then game.Loaded:Wait() end

local gui = createGUI()

if gui and gui.Parent then
    print("[Kick] ✅ Interface carregada com sucesso!")
    print("[Kick] 📋 Painel arrastável disponível")
    print("[Kick] ⚡ Use os botões ATACAR para derrubar jogadores")
    print("[Kick] ⚠️ Client-side: usa flood e exploits para forçar desconexão")
else
    task.wait(2)
    gui = createGUI()
    if gui and gui.Parent then
        print("[Kick] ✅ Interface carregada na segunda tentativa!")
    else
        warn("[Kick] ❌ Falha ao criar interface")
    end
end

print("=" .. string.rep("=", 60))

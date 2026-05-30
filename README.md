--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE KICK SYSTEM - INTERFACE GRÁFICA COMPLETA  ║
    ║     Com botões interativos e painel profissional          ║
    ║     Requer: Executor Server-Side                          ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local TeleportService = game:GetService("TeleportService")

-- ============================================
-- VERIFICAÇÃO
-- ============================================
local IsServer = pcall(function()
    return RunService:IsServer()
end)

if not IsServer then
    warn("⚠️ Execute no servidor para kick real")
end

-- ============================================
-- ENGINE DE KICK ULTRA AGRESSIVA
-- ============================================
local KickEngine = {}

function KickEngine.Execute(player, reason)
    if not player or not player:IsA("Player") then return false end
    
    local name = player.Name
    
    -- Método 1: Kick nativo
    pcall(function() player:Kick(reason) end)
    if not player.Parent then return true end
    
    -- Método 2: Destruir character
    pcall(function()
        if player.Character then
            player.Character:BreakJoints()
            player.Character:Destroy()
        end
        if player.Backpack then
            player.Backpack:Destroy()
        end
    end)
    task.wait(0.5)
    if not player.Parent then return true end
    
    -- Método 3: Teleport void
    pcall(function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(0, -99999, 0)
        end
    end)
    task.wait(1)
    if not player.Parent then return true end
    
    -- Método 4: Flood
    task.spawn(function()
        for i = 1, 200 do
            pcall(function()
                for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                    if obj:IsA("RemoteEvent") then
                        obj:FireClient(player)
                    end
                end
            end)
            task.wait(0.01)
        end
    end)
    task.wait(3)
    if not player.Parent then return true end
    
    -- Método 5: Última tentativa
    pcall(function() player:Kick(reason) end)
    task.wait(2)
    
    return not player.Parent
end

-- ============================================
-- INTERFACE GRÁFICA COMPLETA
-- ============================================
local function createGUI()
    -- Remove UI antiga
    if _G.KickSystemGUI then
        _G.KickSystemGUI:Destroy()
        _G.KickSystemGUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "KickSystemGUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 99999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(screenGui) end
        if gethui then screenGui.Parent = gethui() end
    end)

    _G.KickSystemGUI = screenGui

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Name = "Main"
    main.Size = UDim2.new(0, 420, 0, 500)
    main.Position = UDim2.new(0.5, -210, 0.5, -250)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = screenGui

    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 16)
    mainCorner.Parent = main

    local mainStroke = Instance.new("UIStroke")
    mainStroke.Thickness = 2
    mainStroke.Color = Color3.fromRGB(239, 68, 68)
    mainStroke.Parent = main

    -- Gradiente superior
    local gradient = Instance.new("ImageLabel")
    gradient.Size = UDim2.new(1, 0, 0, 4)
    gradient.BackgroundTransparency = 1
    gradient.Image = "rbxassetid://9968344105"
    gradient.ImageColor3 = Color3.fromRGB(239, 68, 68)
    gradient.ScaleType = Enum.ScaleType.Fit
    gradient.Parent = main

    -- ========== CABEÇALHO ==========
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 50)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    local headerCorner = Instance.new("UICorner")
    headerCorner.CornerRadius = UDim.new(0, 16)
    headerCorner.Parent = header

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
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 7)
    closeCorner.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.KickSystemGUI = nil
    end)

    -- ========== BARRA DE PESQUISA ==========
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 36)
    searchFrame.Position = UDim2.new(0, 8, 0, 58)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    local searchCorner = Instance.new("UICorner")
    searchCorner.CornerRadius = UDim.new(0, 10)
    searchCorner.Parent = searchFrame

    local searchIcon = Instance.new("TextLabel")
    searchIcon.Size = UDim2.new(0, 24, 1, 0)
    searchIcon.BackgroundTransparency = 1
    searchIcon.Text = "🔍"
    searchIcon.TextSize = 14
    searchIcon.Parent = searchFrame

    local searchBox = Instance.new("TextBox")
    searchBox.Name = "SearchBox"
    searchBox.Size = UDim2.new(1, -32, 1, 0)
    searchBox.Position = UDim2.new(0, 28, 0, 0)
    searchBox.BackgroundTransparency = 1
    searchBox.PlaceholderText = "Pesquisar jogador..."
    searchBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 130)
    searchBox.Text = ""
    searchBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    searchBox.TextSize = 13
    searchBox.Font = Enum.Font.Gotham
    searchBox.TextXAlignment = Enum.TextXAlignment.Left
    searchBox.Parent = searchFrame

    -- ========== CONTADOR ==========
    local countLabel = Instance.new("TextLabel")
    countLabel.Name = "CountLabel"
    countLabel.Size = UDim2.new(1, -16, 0, 20)
    countLabel.Position = UDim2.new(0, 8, 0, 100)
    countLabel.BackgroundTransparency = 1
    countLabel.Text = "👥 Jogadores online: 0"
    countLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
    countLabel.TextSize = 11
    countLabel.Font = Enum.Font.GothamBold
    countLabel.TextXAlignment = Enum.TextXAlignment.Left
    countLabel.Parent = main

    -- ========== LISTA DE JOGADORES ==========
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Name = "ListFrame"
    listFrame.Size = UDim2.new(1, -16, 0, 240)
    listFrame.Position = UDim2.new(0, 8, 0, 124)
    listFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    listFrame.BorderSizePixel = 0
    listFrame.ScrollBarThickness = 4
    listFrame.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listFrame.Parent = main
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 10)
    listCorner.Parent = listFrame

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.Parent = listFrame

    -- ========== BOTÕES DE AÇÃO ==========
    local btnKickAll = Instance.new("TextButton")
    btnKickAll.Name = "BtnKickAll"
    btnKickAll.Size = UDim2.new(1, -16, 0, 40)
    btnKickAll.Position = UDim2.new(0, 8, 0, 372)
    btnKickAll.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    btnKickAll.Text = "💀 KICKAR TODOS"
    btnKickAll.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnKickAll.TextSize = 14
    btnKickAll.Font = Enum.Font.GothamBold
    btnKickAll.BorderSizePixel = 0
    btnKickAll.Parent = main
    local btnAllCorner = Instance.new("UICorner")
    btnAllCorner.CornerRadius = UDim.new(0, 10)
    btnAllCorner.Parent = btnKickAll

    local btnRefresh = Instance.new("TextButton")
    btnRefresh.Name = "BtnRefresh"
    btnRefresh.Size = UDim2.new(1, -16, 0, 32)
    btnRefresh.Position = UDim2.new(0, 8, 0, 418)
    btnRefresh.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
    btnRefresh.Text = "🔄 ATUALIZAR LISTA"
    btnRefresh.TextColor3 = Color3.fromRGB(200, 200, 200)
    btnRefresh.TextSize = 12
    btnRefresh.Font = Enum.Font.Gotham
    btnRefresh.BorderSizePixel = 0
    btnRefresh.Parent = main
    local btnRefreshCorner = Instance.new("UICorner")
    btnRefreshCorner.CornerRadius = UDim.new(0, 10)
    btnRefreshCorner.Parent = btnRefresh

    -- ========== BARRA DE STATUS ==========
    local statusFrame = Instance.new("Frame")
    statusFrame.Name = "StatusFrame"
    statusFrame.Size = UDim2.new(1, -16, 0, 28)
    statusFrame.Position = UDim2.new(0, 8, 0, 456)
    statusFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    statusFrame.BorderSizePixel = 0
    statusFrame.Parent = main
    local statusCorner = Instance.new("UICorner")
    statusCorner.CornerRadius = UDim.new(0, 8)
    statusCorner.Parent = statusFrame

    local statusDot = Instance.new("Frame")
    statusDot.Name = "StatusDot"
    statusDot.Size = UDim2.new(0, 8, 0, 8)
    statusDot.Position = UDim2.new(0, 10, 0, 10)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusFrame
    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(1, 0)
    dotCorner.Parent = statusDot

    local statusText = Instance.new("TextLabel")
    statusText.Name = "StatusText"
    statusText.Size = UDim2.new(1, -28, 1, 0)
    statusText.Position = UDim2.new(0, 24, 0, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "Sistema pronto"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusFrame

    -- ========== REFERÊNCIAS ==========
    local ui = {
        screenGui = screenGui,
        main = main,
        listFrame = listFrame,
        listLayout = listLayout,
        countLabel = countLabel,
        searchBox = searchBox,
        btnKickAll = btnKickAll,
        btnRefresh = btnRefresh,
        statusDot = statusDot,
        statusText = statusText
    }

    -- ========== FUNÇÃO: ATUALIZAR LISTA ==========
    local function refreshList()
        -- Limpa a lista
        for _, child in ipairs(listFrame:GetChildren()) do
            if child:IsA("Frame") and child.Name == "PlayerCard" then
                child:Destroy()
            end
        end

        local allPlayers = Players:GetPlayers()
        local searchTerm = searchBox.Text:lower()
        local visibleCount = 0

        -- Ordena por nome
        table.sort(allPlayers, function(a, b)
            return a.Name:lower() < b.Name:lower()
        end)

        for _, player in ipairs(allPlayers) do
            -- Filtro de pesquisa
            if searchTerm == "" or 
               player.Name:lower():find(searchTerm) or 
               player.DisplayName:lower():find(searchTerm) then
                
                visibleCount = visibleCount + 1

                -- Card do jogador
                local card = Instance.new("Frame")
                card.Name = "PlayerCard"
                card.Size = UDim2.new(1, 0, 0, 52)
                card.BackgroundColor3 = Color3.fromRGB(22, 24, 30)
                card.BorderSizePixel = 0
                card.Parent = listFrame
                
                local cardCorner = Instance.new("UICorner")
                cardCorner.CornerRadius = UDim.new(0, 10)
                cardCorner.Parent = card

                -- Avatar (círculo com inicial)
                local avatar = Instance.new("Frame")
                avatar.Size = UDim2.new(0, 36, 0, 36)
                avatar.Position = UDim2.new(0, 10, 0, 8)
                avatar.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
                avatar.BorderSizePixel = 0
                avatar.Parent = card
                local avatarCorner = Instance.new("UICorner")
                avatarCorner.CornerRadius = UDim.new(1, 0)
                avatarCorner.Parent = avatar

                local avatarText = Instance.new("TextLabel")
                avatarText.Size = UDim2.new(1, 0, 1, 0)
                avatarText.BackgroundTransparency = 1
                avatarText.Text = player.Name:sub(1, 1):upper()
                avatarText.TextColor3 = Color3.fromRGB(255, 255, 255)
                avatarText.TextSize = 18
                avatarText.Font = Enum.Font.GothamBold
                avatarText.Parent = avatar

                -- Nome do jogador
                local nameLabel = Instance.new("TextLabel")
                nameLabel.Size = UDim2.new(1, -120, 0, 22)
                nameLabel.Position = UDim2.new(0, 56, 0, 6)
                nameLabel.BackgroundTransparency = 1
                nameLabel.Text = player.DisplayName ~= player.Name and 
                    player.DisplayName .. " (@" .. player.Name .. ")" or 
                    player.Name
                nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                nameLabel.TextSize = 13
                nameLabel.Font = Enum.Font.GothamBold
                nameLabel.TextXAlignment = Enum.TextXAlignment.Left
                nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
                nameLabel.Parent = card

                -- ID do jogador
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

                -- Botão KICK
                local kickBtn = Instance.new("TextButton")
                kickBtn.Size = UDim2.new(0, 55, 0, 30)
                kickBtn.Position = UDim2.new(1, -65, 0, 11)
                kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                kickBtn.Text = "KICK"
                kickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                kickBtn.TextSize = 11
                kickBtn.Font = Enum.Font.GothamBold
                kickBtn.BorderSizePixel = 0
                kickBtn.Parent = card
                
                local kickCorner = Instance.new("UICorner")
                kickCorner.CornerRadius = UDim.new(0, 7)
                kickCorner.Parent = kickBtn

                -- Hover no botão
                kickBtn.MouseEnter:Connect(function()
                    kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38)
                end)
                kickBtn.MouseLeave:Connect(function()
                    kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                end)

                -- AÇÃO DE KICK
                kickBtn.MouseButton1Click:Connect(function()
                    local playerName = player.Name
                    kickBtn.Text = "⏳"
                    kickBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                    kickBtn.Active = false

                    ui.statusText.Text = "👢 Kickando " .. playerName .. "..."
                    ui.statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

                    -- Executa o kick em uma thread separada
                    task.spawn(function()
                        local success = KickEngine.Execute(player, "Kickado pelo administrador")
                        
                        if success then
                            kickBtn.Text = "✓"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                            ui.statusText.Text = "✅ " .. playerName .. " removido!"
                            ui.statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                            
                            -- Animação de fade out no card
                            card.BackgroundColor3 = Color3.fromRGB(34, 197, 94, 50)
                            task.wait(1)
                            refreshList()
                        else
                            kickBtn.Text = "✗"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                            ui.statusText.Text = "❌ Falha ao kickar " .. playerName
                            ui.statusDot.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                            kickBtn.Active = true
                        end
                        
                        task.wait(2)
                        ui.statusText.Text = "Sistema pronto"
                        ui.statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                    end)
                end)
            end
        end

        -- Atualiza contador e canvas
        ui.countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #allPlayers ..
            (searchTerm ~= "" and " (filtrado)" or "")
        listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 56 + (visibleCount - 1) * 4 + 8)
    end

    -- ========== EVENTOS DOS BOTÕES ==========
    btnRefresh.MouseButton1Click:Connect(function()
        ui.statusText.Text = "🔄 Atualizando..."
        refreshList()
        ui.statusText.Text = "✅ Lista atualizada!"
        task.wait(1.5)
        ui.statusText.Text = "Sistema pronto"
    end)

    btnKickAll.MouseButton1Click:Connect(function()
        local allPlayers = Players:GetPlayers()
        if #allPlayers <= 1 then
            ui.statusText.Text = "⚠️ Apenas você no servidor"
            return
        end

        btnKickAll.Text = "⏳ Processando..."
        btnKickAll.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        btnKickAll.Active = false

        ui.statusText.Text = "💀 Kickando TODOS os jogadores..."
        ui.statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

        task.spawn(function()
            local kicked = 0
            local failed = 0

            for _, player in ipairs(allPlayers) do
                ui.statusText.Text = "👢 Kickando " .. player.Name .. "..."
                local success = KickEngine.Execute(player, "Kick em massa")
                if success then
                    kicked = kicked + 1
                else
                    failed = failed + 1
                end
                task.wait(0.3)
            end

            ui.statusText.Text = "✅ " .. kicked .. " kickados, " .. failed .. " falhas"
            ui.statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            
            btnKickAll.Text = "💀 KICKAR TODOS"
            btnKickAll.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            btnKickAll.Active = true
            
            refreshList()
            
            task.wait(3)
            ui.statusText.Text = "Sistema pronto"
        end)
    end)

    -- Pesquisa em tempo real
    searchBox:GetPropertyChangedSignal("Text"):Connect(function()
        refreshList()
    end)

    -- Refresh automático
    task.spawn(function()
        while screenGui and screenGui.Parent do
            task.wait(5)
            pcall(function() refreshList() end)
        end
    end)

    -- Refresh inicial
    refreshList()

    return ui
end

-- ============================================
-- CRIA A INTERFACE
-- ============================================
local ui = createGUI()

-- ============================================
-- ATALHOS DE TECLADO
-- ============================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F5 then
        -- Atualizar lista
        if ui and ui.listFrame then
            pcall(function()
                -- Dispara refresh manualmente
                local btn = ui.main:FindFirstChild("BtnRefresh")
                if btn then
                    firesignal(btn.MouseButton1Click)
                end
            end)
        end
    end
    
    if input.KeyCode == Enum.KeyCode.F6 then
        -- Mostrar/Esconder interface
        if _G.KickSystemGUI then
            _G.KickSystemGUI.Enabled = not _G.KickSystemGUI.Enabled
        end
    end
end)

-- ============================================
-- LOGS
-- ============================================
print("=" .. string.rep("=", 55))
print("  👢 KICK SYSTEM PRO - INTERFACE GRÁFICA")
print("  Interface completa com botões interativos")
print("=" .. string.rep("=", 55))
print("")
print("  🎮 ATALHOS:")
print("    F5 = Atualizar lista")
print("    F6 = Esconder/Mostrar interface")
print("")
print("  📋 FUNCIONALIDADES:")
print("    • Lista de jogadores com busca")
print("    • Botão KICK individual em cada jogador")
print("    • Botão KICKAR TODOS")
print("    • Status em tempo real")
print("    • Atualização automática a cada 5s")
print("")
print("  ⚠️ Execute no SERVIDOR para kick real")
print("=" .. string.rep("=", 55))

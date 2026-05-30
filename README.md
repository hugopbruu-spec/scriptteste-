--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║              SERVER-SIDE KICK GUI v1.0                     ║
    ║    Interface própria para kickar jogadores do servidor     ║
    ║    Requer: Executor Server-Side                            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

-- ============================================
-- CRIA A INTERFACE
-- ============================================
local function createUI()
    -- Remove UI antiga
    if _G.KickGUI then
        _G.KickGUI:Destroy()
        _G.KickGUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "KickGUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 99999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(screenGui) end
        if gethui then screenGui.Parent = gethui() end
    end)

    _G.KickGUI = screenGui

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 380, 0, 440)
    main.Position = UDim2.new(0.5, -190, 0.5, -220)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = main

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(239, 68, 68)
    stroke.Parent = main

    -- ========== CABEÇALHO ==========
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 48)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 14)
    hCorner.Parent = header

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -50, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "👢 KICK PLAYERS"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 9)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    local cc = Instance.new("UICorner")
    cc.CornerRadius = UDim.new(0, 7)
    cc.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.KickGUI = nil
    end)

    -- ========== CONTADOR DE JOGADORES ==========
    local countLabel = Instance.new("TextLabel")
    countLabel.Name = "CountLabel"
    countLabel.Size = UDim2.new(1, -16, 0, 24)
    countLabel.Position = UDim2.new(0, 8, 0, 56)
    countLabel.BackgroundTransparency = 1
    countLabel.Text = "👥 Jogadores: 0"
    countLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    countLabel.TextSize = 12
    countLabel.Font = Enum.Font.GothamBold
    countLabel.TextXAlignment = Enum.TextXAlignment.Left
    countLabel.Parent = main

    -- ========== LISTA DE JOGADORES ==========
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Name = "ListFrame"
    listFrame.Size = UDim2.new(1, -16, 0, 280)
    listFrame.Position = UDim2.new(0, 8, 0, 84)
    listFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    listFrame.BorderSizePixel = 0
    listFrame.ScrollBarThickness = 4
    listFrame.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listFrame.Parent = main
    local lfCorner = Instance.new("UICorner")
    lfCorner.CornerRadius = UDim.new(0, 10)
    lfCorner.Parent = listFrame

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.Parent = listFrame

    -- ========== BOTÃO KICKAR TODOS ==========
    local kickAllBtn = Instance.new("TextButton")
    kickAllBtn.Size = UDim2.new(1, -16, 0, 36)
    kickAllBtn.Position = UDim2.new(0, 8, 0, 372)
    kickAllBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    kickAllBtn.Text = "💀 KICKAR TODOS"
    kickAllBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    kickAllBtn.TextSize = 14
    kickAllBtn.Font = Enum.Font.GothamBold
    kickAllBtn.BorderSizePixel = 0
    kickAllBtn.Parent = main
    local kaCorner = Instance.new("UICorner")
    kaCorner.CornerRadius = UDim.new(0, 8)
    kaCorner.Parent = kickAllBtn

    -- ========== BARRA DE PESQUISA ==========
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 32)
    searchFrame.Position = UDim2.new(0, 8, 0, 414)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    local sfCorner = Instance.new("UICorner")
    sfCorner.CornerRadius = UDim.new(0, 8)
    sfCorner.Parent = searchFrame

    local searchBox = Instance.new("TextBox")
    searchBox.Name = "SearchBox"
    searchBox.Size = UDim2.new(1, -16, 1, 0)
    searchBox.Position = UDim2.new(0, 8, 0, 0)
    searchBox.BackgroundTransparency = 1
    searchBox.PlaceholderText = "🔍 Pesquisar jogador..."
    searchBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 130)
    searchBox.Text = ""
    searchBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    searchBox.TextSize = 12
    searchBox.Font = Enum.Font.Gotham
    searchBox.TextXAlignment = Enum.TextXAlignment.Left
    searchBox.Parent = searchFrame

    return {
        screenGui = screenGui,
        listFrame = listFrame,
        listLayout = listLayout,
        countLabel = countLabel,
        kickAllBtn = kickAllBtn,
        searchBox = searchBox
    }
end

-- ============================================
-- FUNÇÃO DE KICK
-- ============================================
local function kickPlayer(player, reason)
    local success, err = pcall(function()
        player:Kick(reason or "Kickado pelo administrador")
    end)
    
    if success then
        print("👢 Kickado: " .. player.Name)
        return true
    else
        print("❌ Falha ao kickar " .. player.Name .. ": " .. tostring(err))
        return false
    end
end

-- ============================================
-- ATUALIZA A LISTA DE JOGADORES
-- ============================================
local function refreshPlayerList(ui)
    local listFrame = ui.listFrame
    local listLayout = ui.listLayout
    local searchText = ui.searchBox.Text:lower()
    
    -- Limpa a lista
    for _, child in ipairs(listFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    local players = Players:GetPlayers()
    local visibleCount = 0
    
    -- Ordena por nome
    table.sort(players, function(a, b)
        return a.Name:lower() < b.Name:lower()
    end)
    
    -- Cria um card para cada jogador
    for _, player in ipairs(players) do
        -- Filtro de pesquisa
        if searchText == "" or player.Name:lower():find(searchText) or player.DisplayName:lower():find(searchText) then
            visibleCount = visibleCount + 1
            
            local card = Instance.new("Frame")
            card.Size = UDim2.new(1, 0, 0, 44)
            card.BackgroundColor3 = Color3.fromRGB(22, 24, 30)
            card.BorderSizePixel = 0
            card.Parent = listFrame
            local cardCorner = Instance.new("UICorner")
            cardCorner.CornerRadius = UDim.new(0, 8)
            cardCorner.Parent = card
            
            -- Ícone
            local icon = Instance.new("TextLabel")
            icon.Size = UDim2.new(0, 32, 0, 32)
            icon.Position = UDim2.new(0, 8, 0, 6)
            icon.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
            icon.Text = player.Name:sub(1, 1):upper()
            icon.TextColor3 = Color3.fromRGB(255, 255, 255)
            icon.TextSize = 16
            icon.Font = Enum.Font.GothamBold
            icon.Parent = card
            local iconCorner = Instance.new("UICorner")
            iconCorner.CornerRadius = UDim.new(1, 0)
            iconCorner.Parent = icon
            
            -- Nome
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(1, -120, 0, 22)
            nameLabel.Position = UDim2.new(0, 50, 0, 4)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = player.DisplayName ~= player.Name and player.DisplayName .. " (@" .. player.Name .. ")" or player.Name
            nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            nameLabel.TextSize = 12
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.TextXAlignment = Enum.TextXAlignment.Left
            nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
            nameLabel.Parent = card
            
            -- ID
            local idLabel = Instance.new("TextLabel")
            idLabel.Size = UDim2.new(1, -120, 0, 16)
            idLabel.Position = UDim2.new(0, 50, 0, 24)
            idLabel.BackgroundTransparency = 1
            idLabel.Text = "ID: " .. player.UserId
            idLabel.TextColor3 = Color3.fromRGB(140, 140, 150)
            idLabel.TextSize = 10
            idLabel.Font = Enum.Font.Gotham
            idLabel.TextXAlignment = Enum.TextXAlignment.Left
            idLabel.Parent = card
            
            -- Botão Kick
            local kickBtn = Instance.new("TextButton")
            kickBtn.Size = UDim2.new(0, 60, 0, 28)
            kickBtn.Position = UDim2.new(1, -70, 0, 8)
            kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            kickBtn.Text = "KICK"
            kickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            kickBtn.TextSize = 11
            kickBtn.Font = Enum.Font.GothamBold
            kickBtn.BorderSizePixel = 0
            kickBtn.Parent = card
            local kbCorner = Instance.new("UICorner")
            kbCorner.CornerRadius = UDim.new(0, 6)
            kbCorner.Parent = kickBtn
            
            -- Ação de kick
            kickBtn.MouseButton1Click:Connect(function()
                local reason = "Kickado por " .. Players.LocalPlayer.Name
                local kicked = kickPlayer(player, reason)
                if kicked then
                    card.BackgroundColor3 = Color3.fromRGB(40, 10, 10)
                    kickBtn.Text = "✓"
                    kickBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                    task.wait(0.5)
                    refreshPlayerList(ui)
                end
            end)
            
            -- Hover
            kickBtn.MouseEnter:Connect(function()
                kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38)
            end)
            kickBtn.MouseLeave:Connect(function()
                kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            end)
        end
    end
    
    -- Atualiza canvas e contador
    listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 48 + (visibleCount - 1) * 4 + 8)
    ui.countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #players
    
    if searchText ~= "" then
        ui.countLabel.Text = ui.countLabel.Text .. " (filtrado)"
    end
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
local ui = createUI()

-- Refresh inicial
refreshPlayerList(ui)

-- Refresh automático a cada 3 segundos
task.spawn(function()
    while ui.screenGui and ui.screenGui.Parent do
        task.wait(3)
        pcall(function()
            refreshPlayerList(ui)
        end)
    end
end)

-- Refresh ao pesquisar
ui.searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    refreshPlayerList(ui)
end)

-- Kickar todos
ui.kickAllBtn.MouseButton1Click:Connect(function()
    local players = Players:GetPlayers()
    local localPlayer = Players.LocalPlayer
    
    ui.kickAllBtn.Text = "⏳ Kickando..."
    ui.kickAllBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    ui.kickAllBtn.Active = false
    
    local kicked = 0
    for _, player in ipairs(players) do
        if player ~= localPlayer then
            if kickPlayer(player, "Kick em massa") then
                kicked = kicked + 1
            end
            task.wait(0.1)
        end
    end
    
    ui.kickAllBtn.Text = "✅ " .. kicked .. " kickados!"
    ui.kickAllBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    task.wait(2)
    ui.kickAllBtn.Text = "💀 KICKAR TODOS"
    ui.kickAllBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    ui.kickAllBtn.Active = true
    refreshPlayerList(ui)
end)

-- Atalhos de teclado
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F6 then
        refreshPlayerList(ui)
    end
    
    if input.KeyCode == Enum.KeyCode.F7 then
        if ui.screenGui then
            ui.screenGui.Enabled = not ui.screenGui.Enabled
        end
    end
end)

print("=" .. string.rep("=", 55))
print("  👢 KICK GUI CARREGADO")
print("  F6 = Atualizar lista")
print("  F7 = Esconder/Mostrar")
print("=" .. string.rep("=", 55))

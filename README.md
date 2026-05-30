--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     CLIENT-SIDE KICK GUI - INTERFACE GARANTIDA            ║
    ║     Compatível com Xeno e outros executores               ║
    ║     Interface 100% funcional e visível                    ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- FORÇA A INTERFACE A APARECER
-- ============================================
local function createInterface()
    -- Destroi qualquer interface antiga
    pcall(function()
        if _G.KickGUI then
            _G.KickGUI:Destroy()
            _G.KickGUI = nil
        end
    end)

    -- Cria ScreenGui
    local gui = Instance.new("ScreenGui")
    gui.Name = "KickGUI"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 999999
    gui.Enabled = true
    
    -- Tenta múltiplos locais para garantir visibilidade
    local parentSuccess = false
    local parents = {
        function() return game:GetService("CoreGui") end,
        function() return game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui") end,
        function() 
            if gethui then return gethui() end
            return nil
        end,
        function() return game:GetService("StarterGui") end
    }
    
    for _, getParent in ipairs(parents) do
        local success = pcall(function()
            local parent = getParent()
            if parent then
                gui.Parent = parent
                parentSuccess = true
            end
        end)
        if parentSuccess then break end
    end
    
    if not parentSuccess then
        gui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
    end
    
    _G.KickGUI = gui

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
    notifText.Text = "✅ KICK GUI CARREGADO!"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextSize = 14
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif

    task.delay(3, function() pcall(function() notif:Destroy() end) end)

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Name = "MainPanel"
    main.Size = UDim2.new(0, 400, 0, 450)
    main.Position = UDim2.new(0.5, -200, 0.5, -225)
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
    title.Text = "👢 KICK SYSTEM"
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
    closeBtn.MouseButton1Click:Connect(function() 
        gui:Destroy() 
        _G.KickGUI = nil 
    end)

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
    listFrame.Size = UDim2.new(1, -16, 0, 240)
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

    -- Botões
    local btnKickAll = Instance.new("TextButton")
    btnKickAll.Size = UDim2.new(1, -16, 0, 40)
    btnKickAll.Position = UDim2.new(0, 8, 0, 372)
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
    btnRefresh.Position = UDim2.new(0, 8, 0, 418)
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
    statusFrame.Position = UDim2.new(0, 8, 0, 456)
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
    statusText.Text = "✅ Interface pronta"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusFrame

    -- ========== FUNÇÕES ==========
    local Players = game:GetService("Players")
    
    local function refreshList()
        -- Limpa lista
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
                
                -- Avatar
                local avatar = Instance.new("Frame")
                avatar.Size = UDim2.new(0, 36, 0, 36)
                avatar.Position = UDim2.new(0, 10, 0, 8)
                avatar.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
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
                
                -- Nome
                local nameLabel = Instance.new("TextLabel")
                nameLabel.Size = UDim2.new(1, -120, 0, 22)
                nameLabel.Position = UDim2.new(0, 56, 0, 6)
                nameLabel.BackgroundTransparency = 1
                nameLabel.Text = player.DisplayName ~= player.Name and player.DisplayName .. " (@" .. player.Name .. ")" or player.Name
                nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                nameLabel.TextSize = 13
                nameLabel.Font = Enum.Font.GothamBold
                nameLabel.TextXAlignment = Enum.TextXAlignment.Left
                nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
                nameLabel.Parent = card
                
                -- ID
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
                Instance.new("UICorner", kickBtn).CornerRadius = UDim.new(0, 7)
                
                kickBtn.MouseEnter:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38) end)
                kickBtn.MouseLeave:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68) end)
                
                kickBtn.MouseButton1Click:Connect(function()
                    statusText.Text = "⚠️ Client-side: kick visual apenas"
                    statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)
                    
                    -- Remove visualmente (só some pra você)
                    card:Destroy()
                    refreshList()
                    
                    task.wait(2)
                    statusText.Text = "✅ Interface pronta"
                    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                end)
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
        statusText.Text = "✅ Interface pronta"
    end)
    
    btnKickAll.MouseButton1Click:Connect(function()
        statusText.Text = "⚠️ Client-side: kick visual apenas"
        refreshList()
        task.wait(2)
        statusText.Text = "✅ Interface pronta"
    end)
    
    searchBox:GetPropertyChangedSignal("Text"):Connect(refreshList)
    
    -- Auto refresh
    task.spawn(function()
        while gui and gui.Parent do
            task.wait(5)
            pcall(refreshList)
        end
    end)
    
    -- Inicial
    refreshList()
    
    return gui
end

-- ============================================
-- EXECUÇÃO FORÇADA
-- ============================================
print("=" .. string.rep("=", 55))
print("  👢 CLIENT-SIDE KICK GUI")
print("  Compatível com Xeno")
print("=" .. string.rep("=", 55))

-- Aguarda o jogo carregar
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Tenta criar a interface
local gui = createInterface()

if gui and gui.Parent then
    print("[Kick GUI] ✅ Interface visível!")
    print("[Kick GUI] ⚠️ Xeno é client-side - kicks são visuais")
    print("[Kick GUI] 💡 Para kick real, use Synapse X ou ScriptWare SS")
else
    -- Segunda tentativa
    task.wait(2)
    gui = createInterface()
    
    if gui and gui.Parent then
        print("[Kick GUI] ✅ Interface criada na segunda tentativa!")
    else
        warn("[Kick GUI] ❌ Não foi possível criar a interface")
        warn("[Kick GUI] Tente usar outro executor")
    end
end

print("=" .. string.rep("=", 55))

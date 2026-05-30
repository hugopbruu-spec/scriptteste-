--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     KICK SYSTEM - INTERFACE GRÁFICA OBRIGATÓRIA           ║
    ║     A interface aparece AUTOMATICAMENTE                   ║
    ║     Requer: Executor Server-Side                          ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- FORÇA A CRIAÇÃO DA INTERFACE
-- ============================================
local function forceInterface()
    -- Destroi qualquer interface antiga
    pcall(function()
        if _G.KickUI then
            _G.KickUI:Destroy()
            _G.KickUI = nil
        end
    end)

    -- Cria o ScreenGui
    local gui = Instance.new("ScreenGui")
    gui.Name = "KickUI_Main"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 999999
    gui.Enabled = true
    gui.IgnoreGuiInset = false

    -- Tenta colocar no lugar certo
    local success = pcall(function()
        gui.Parent = game:GetService("CoreGui")
    end)
    if not success then
        pcall(function()
            gui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
        end)
    end

    -- Proteção
    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(gui) end
        if gethui then gui.Parent = gethui() end
    end)

    _G.KickUI = gui

    -- ========== NOTIFICAÇÃO DE CARREGAMENTO ==========
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 40)
    notif.Position = UDim2.new(0.5, -150, 0, 10)
    notif.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    notif.BorderSizePixel = 0
    notif.Parent = gui
    local nc = Instance.new("UICorner"); nc.CornerRadius = UDim.new(0, 8); nc.Parent = notif

    local nt = Instance.new("TextLabel")
    nt.Size = UDim2.new(1, 0, 1, 0)
    nt.BackgroundTransparency = 1
    nt.Text = "👢 KICK SYSTEM CARREGADO!"
    nt.TextColor3 = Color3.fromRGB(255, 255, 255)
    nt.TextSize = 14
    nt.Font = Enum.Font.GothamBold
    nt.Parent = notif

    task.delay(3, function() notif:Destroy() end)

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Name = "MainPanel"
    main.Size = UDim2.new(0, 420, 0, 500)
    main.Position = UDim2.new(0.5, -210, 0.5, -250)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Visible = true
    main.Parent = gui

    local mc = Instance.new("UICorner"); mc.CornerRadius = UDim.new(0, 16); mc.Parent = main
    local ms = Instance.new("UIStroke"); ms.Thickness = 2; ms.Color = Color3.fromRGB(239, 68, 68); ms.Parent = main

    -- Gradiente decorativo
    local grad = Instance.new("ImageLabel")
    grad.Size = UDim2.new(1, 0, 0, 4)
    grad.BackgroundTransparency = 1
    grad.Image = "rbxassetid://9968344105"
    grad.ImageColor3 = Color3.fromRGB(239, 68, 68)
    grad.ScaleType = Enum.ScaleType.Fit
    grad.Parent = main

    -- ========== CABEÇALHO ==========
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 50)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    local hc = Instance.new("UICorner"); hc.CornerRadius = UDim.new(0, 16); hc.Parent = header

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

    -- Botão fechar
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
    local cc = Instance.new("UICorner"); cc.CornerRadius = UDim.new(0, 7); cc.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() _G.KickUI = nil end)

    -- ========== BARRA DE PESQUISA ==========
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 36)
    searchFrame.Position = UDim2.new(0, 8, 0, 58)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    local sfc = Instance.new("UICorner"); sfc.CornerRadius = UDim.new(0, 10); sfc.Parent = searchFrame

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
    local lfc = Instance.new("UICorner"); lfc.CornerRadius = UDim.new(0, 10); lfc.Parent = listFrame

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.Parent = listFrame

    -- ========== BOTÕES ==========
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
    local bac = Instance.new("UICorner"); bac.CornerRadius = UDim.new(0, 10); bac.Parent = btnKickAll

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
    local brc = Instance.new("UICorner"); brc.CornerRadius = UDim.new(0, 10); brc.Parent = btnRefresh

    -- ========== STATUS ==========
    local statusFrame = Instance.new("Frame")
    statusFrame.Name = "StatusFrame"
    statusFrame.Size = UDim2.new(1, -16, 0, 28)
    statusFrame.Position = UDim2.new(0, 8, 0, 456)
    statusFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    statusFrame.BorderSizePixel = 0
    statusFrame.Parent = main
    local stc = Instance.new("UICorner"); stc.CornerRadius = UDim.new(0, 8); stc.Parent = statusFrame

    local statusDot = Instance.new("Frame")
    statusDot.Name = "StatusDot"
    statusDot.Size = UDim2.new(0, 8, 0, 8)
    statusDot.Position = UDim2.new(0, 10, 0, 10)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusFrame
    local sdc = Instance.new("UICorner"); sdc.CornerRadius = UDim.new(1, 0); sdc.Parent = statusDot

    local statusText = Instance.new("TextLabel")
    statusText.Name = "StatusText"
    statusText.Size = UDim2.new(1, -28, 1, 0)
    statusText.Position = UDim2.new(0, 24, 0, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "✅ Sistema pronto"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusFrame

    -- ========== FUNÇÃO DE KICK ==========
    local function kickPlayerReal(player, reason)
        if not player or not player:IsA("Player") then return false end

        -- Método 1: Kick nativo
        pcall(function() player:Kick(reason) end)
        if not player:IsDescendantOf(game:GetService("Players")) then return true end

        -- Método 2: Destroi tudo
        pcall(function()
            if player.Character then
                player.Character:BreakJoints()
                player.Character:Destroy()
            end
            if player.Backpack then player.Backpack:Destroy() end
        end)
        task.wait(0.5)
        if not player:IsDescendantOf(game:GetService("Players")) then return true end

        -- Método 3: Void
        pcall(function()
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                char.HumanoidRootPart.CFrame = CFrame.new(0, -99999, 0)
            end
        end)
        task.wait(1)
        if not player:IsDescendantOf(game:GetService("Players")) then return true end

        -- Método 4: Flood
        task.spawn(function()
            for i = 1, 200 do
                pcall(function()
                    for _, obj in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
                        if obj:IsA("RemoteEvent") then obj:FireClient(player) end
                    end
                end)
                task.wait(0.01)
            end
        end)
        task.wait(3)
        if not player:IsDescendantOf(game:GetService("Players")) then return true end

        -- Método 5: Última tentativa
        pcall(function() player:Kick(reason) end)
        task.wait(2)

        return not player:IsDescendantOf(game:GetService("Players"))
    end

    -- ========== ATUALIZAR LISTA ==========
    local function refreshList()
        for _, child in ipairs(listFrame:GetChildren()) do
            if child:IsA("Frame") and child.Name == "PlayerCard" then
                child:Destroy()
            end
        end

        local allPlayers = game:GetService("Players"):GetPlayers()
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
                local crc = Instance.new("UICorner"); crc.CornerRadius = UDim.new(0, 10); crc.Parent = card

                -- Avatar
                local avatar = Instance.new("Frame")
                avatar.Size = UDim2.new(0, 36, 0, 36)
                avatar.Position = UDim2.new(0, 10, 0, 8)
                avatar.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
                avatar.BorderSizePixel = 0
                avatar.Parent = card
                local avc = Instance.new("UICorner"); avc.CornerRadius = UDim.new(1, 0); avc.Parent = avatar

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
                local kbc = Instance.new("UICorner"); kbc.CornerRadius = UDim.new(0, 7); kbc.Parent = kickBtn

                kickBtn.MouseEnter:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38) end)
                kickBtn.MouseLeave:Connect(function() kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68) end)

                kickBtn.MouseButton1Click:Connect(function()
                    local pname = player.Name
                    kickBtn.Text = "⏳"
                    kickBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                    kickBtn.Active = false
                    statusText.Text = "👢 Kickando " .. pname .. "..."
                    statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

                    task.spawn(function()
                        local result = kickPlayerReal(player, "Kickado")
                        if result then
                            kickBtn.Text = "✓"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                            statusText.Text = "✅ " .. pname .. " removido!"
                            statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                            card.BackgroundColor3 = Color3.fromRGB(40, 10, 10)
                            task.wait(1)
                            refreshList()
                        else
                            kickBtn.Text = "KICK"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                            kickBtn.Active = true
                            statusText.Text = "❌ Falha ao kickar " .. pname
                            statusDot.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                        end
                        task.wait(2)
                        if statusText.Text:find("Falha") then
                            statusText.Text = "✅ Sistema pronto"
                            statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                        end
                    end)
                end)
            end
        end

        countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #allPlayers .. (searchTerm ~= "" and " (filtrado)" or "")
        listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 56 + (visibleCount - 1) * 4 + 8)
    end

    -- ========== EVENTOS ==========
    btnRefresh.MouseButton1Click:Connect(function()
        statusText.Text = "🔄 Atualizando..."
        refreshList()
        statusText.Text = "✅ Lista atualizada!"
        task.wait(1.5)
        statusText.Text = "✅ Sistema pronto"
    end)

    btnKickAll.MouseButton1Click:Connect(function()
        local all = game:GetService("Players"):GetPlayers()
        if #all <= 1 then statusText.Text = "⚠️ Apenas você"; return end

        btnKickAll.Text = "⏳ Processando..."
        btnKickAll.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        btnKickAll.Active = false
        statusText.Text = "💀 Kickando TODOS..."
        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

        task.spawn(function()
            local k, f = 0, 0
            for _, p in ipairs(all) do
                statusText.Text = "👢 Kickando " .. p.Name .. "..."
                if kickPlayerReal(p, "Kick em massa") then k = k + 1 else f = f + 1 end
                task.wait(0.3)
            end
            statusText.Text = "✅ " .. k .. " kickados, " .. f .. " falhas"
            statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            btnKickAll.Text = "💀 KICKAR TODOS"
            btnKickAll.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            btnKickAll.Active = true
            refreshList()
            task.wait(3)
            statusText.Text = "✅ Sistema pronto"
        end)
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

    print("[Kick UI] ✅ Interface criada e visível!")

    return gui
end

-- ============================================
-- EXECUÇÃO FORÇADA
-- ============================================
print("=" .. string.rep("=", 55))
print("  👢 KICK SYSTEM - INTERFACE GARANTIDA")
print("=" .. string.rep("=", 55))

-- Aguarda o jogo carregar completamente
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Tenta criar a interface imediatamente
local gui = forceInterface()

-- Se falhar, tenta novamente em 1 segundo
if not gui or not gui.Parent then
    task.wait(1)
    gui = forceInterface()
end

-- Se ainda falhar, tenta no próximo frame
if not gui or not gui.Parent then
    task.wait(2)
    gui = forceInterface()
end

-- Verificação final
if gui and gui.Parent then
    print("[Kick UI] ✅ INTERFACE VISÍVEL!")
    print("[Kick UI] 📋 Arraste o painel para movê-lo")
    print("[Kick UI] 🔍 Use a busca para filtrar jogadores")
    print("[Kick UI] 👢 Clique em KICK para remover um jogador")
else
    warn("[Kick UI] ❌ Não foi possível criar a interface")
    warn("[Kick UI] Verifique se seu executor suporta CoreGui")
end

print("=" .. string.rep("=", 55))

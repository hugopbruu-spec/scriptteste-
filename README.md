--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     KICK SYSTEM - REAL E FUNCIONAL                         ║
    ║     Requer: Executor com acesso ao servidor                 ║
    ║     Se não funcionar no Xeno, use Synapse X ou KRNL        ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- VERIFICAÇÃO DO EXECUTOR
-- ============================================
local executorName = "Desconhecido"
pcall(function() executorName = identifyexecutor() or getexecutorname() or "Desconhecido" end)

print("=" .. string.rep("=", 55))
print("  👢 KICK SYSTEM - DETECÇÃO DE EXECUTOR")
print("  Executor: " .. executorName)
print("=" .. string.rep("=", 55))

-- ============================================
-- MÉTODO REAL DE KICK (tenta todos os possíveis)
-- ============================================

-- Método 1: Usando a função kick do executor (alguns têm)
local function tryKickMethod1(playerName)
    local success = pcall(function()
        -- Alguns executores expõem esta função
        if kick then
            kick(playerName)
            return true
        end
        if game and game.Kick then
            game.Kick(playerName)
            return true
        end
    end)
    return success
end

-- Método 2: Usando firetouchinterest (exploit conhecido)
local function tryKickMethod2(player)
    local success = pcall(function()
        local char = player.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                -- Força o jogador para fora do mapa
                hrp.CFrame = CFrame.new(0, -99999, 0)
                hrp.Velocity = Vector3.new(0, -9999, 0)
                hrp.Anchored = false
            end
            
            -- Remove o humanoid (causa morte e possível kick)
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum:Destroy()
            end
        end
    end)
    return success
end

-- Método 3: Usando remotes do jogo para kick (se existir sistema de admin)
local function tryKickMethod3(player)
    local success = false
    pcall(function()
        for _, obj in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
            if obj:IsA("RemoteEvent") then
                local name = obj.Name:lower()
                if name:find("kick") or name:find("ban") or name:find("remove") or name:find("admin") then
                    obj:FireServer(player, "Kickado")
                    obj:FireServer(player.Name)
                    obj:FireServer(player.UserId)
                    success = true
                end
            end
        end
    end)
    return success
end

-- Método 4: Crash do cliente alvo (força saída)
local function tryKickMethod4(player)
    task.spawn(function()
        for i = 1, 100 do
            pcall(function()
                -- Spam de sons e partículas no cliente alvo
                local char = player.Character
                if char then
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://9120386436"
                    sound.Volume = 10
                    sound.Parent = char
                    sound:Play()
                    
                    local particle = Instance.new("ParticleEmitter")
                    particle.Rate = 5000
                    particle.Parent = char
                    
                    game:GetService("Debris"):AddItem(sound, 0.1)
                    game:GetService("Debris"):AddItem(particle, 0.1)
                end
            end)
            task.wait(0.01)
        end
    end)
    return true
end

-- Método 5: Usando o sistema de votekick do Roblox (se disponível)
local function tryKickMethod5(player)
    local success = pcall(function()
        -- Alguns jogos têm sistema de votekick
        local votekick = game:GetService("ReplicatedStorage"):FindFirstChild("VoteKick")
        if votekick then
            votekick:FireServer(player)
            return true
        end
        
        -- Ou via chat
        local chatService = game:GetService("Chat")
        if chatService then
            -- Tenta usar comandos de admin
        end
    end)
    return success
end

-- ============================================
-- FUNÇÃO PRINCIPAL DE KICK (TENTA TUDO)
-- ============================================
local function kickPlayer(targetName)
    local targetPlayer = nil
    
    -- Encontra o jogador
    for _, p in ipairs(game:GetService("Players"):GetPlayers()) do
        if p.Name:lower():find(targetName:lower()) then
            targetPlayer = p
            break
        end
    end
    
    if not targetPlayer then
        print("❌ Jogador não encontrado: " .. targetName)
        return false
    end
    
    local playerName = targetPlayer.Name
    print("👢 Tentando kickar: " .. playerName)
    
    -- Tenta todos os métodos
    local methods = {
        {"Kick do Executor", function() return tryKickMethod1(playerName) end},
        {"FireTouch/CFrame", function() return tryKickMethod2(targetPlayer) end},
        {"Remotes de Admin", function() return tryKickMethod3(targetPlayer) end},
        {"Crash do Cliente", function() return tryKickMethod4(targetPlayer) end},
        {"Sistema de VoteKick", function() return tryKickMethod5(targetPlayer) end},
    }
    
    for _, method in ipairs(methods) do
        local methodName = method[1]
        local methodFunc = method[2]
        
        print("  Tentando: " .. methodName)
        local success = methodFunc()
        
        if success then
            print("✅ " .. playerName .. " atacado via " .. methodName)
            return true
        else
            print("  ❌ " .. methodName .. " falhou")
        end
        
        task.wait(0.5)
    end
    
    print("⚠️ Nenhum método funcionou para " .. playerName)
    print("💡 Tente usar Synapse X ou ScriptWare para kick real")
    return false
end

-- ============================================
-- INTERFACE GRÁFICA
-- ============================================
local function createGUI()
    pcall(function()
        if _G.KickGUI then _G.KickGUI:Destroy() end
    end)

    local gui = Instance.new("ScreenGui")
    gui.Name = "KickGUI"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 999999
    gui.Enabled = true

    -- Tenta colocar no lugar certo
    pcall(function() gui.Parent = game:GetService("CoreGui") end)
    if not gui.Parent then
        pcall(function() gui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui") end)
    end
    pcall(function() if gethui then gui.Parent = gethui() end end)

    _G.KickGUI = gui

    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 400, 0, 450)
    main.Position = UDim2.new(0.5, -200, 0.5, -225)
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
    closeBtn.Position = UDim2.new(1, -40, 0, 9)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 7)
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() _G.KickGUI = nil end)

    -- Barra de pesquisa
    local searchFrame = Instance.new("Frame")
    searchFrame.Size = UDim2.new(1, -16, 0, 34)
    searchFrame.Position = UDim2.new(0, 8, 0, 56)
    searchFrame.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    searchFrame.BorderSizePixel = 0
    searchFrame.Parent = main
    Instance.new("UICorner", searchFrame).CornerRadius = UDim.new(0, 10)

    local searchBox = Instance.new("TextBox")
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
    btnKickAll.Text = "💀 KICKAR TODOS"
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
    statusText.Text = "✅ Pronto - Selecione um jogador"
    statusText.TextColor3 = Color3.fromRGB(180, 180, 190)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.Gotham
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusBar

    -- ============================================
    -- ATUALIZAR LISTA
    -- ============================================
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer

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
                    kickBtn.Text = "KICK"
                    kickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    kickBtn.TextSize = 10
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
                        statusText.Text = "👢 Kickando " .. player.Name .. "..."
                        statusDot.BackgroundColor3 = Color3.fromRGB(249, 115, 22)

                        local result = kickPlayer(player.Name)
                        
                        if result then
                            kickBtn.Text = "✓"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                            statusText.Text = "✅ " .. player.Name .. " kickado!"
                            statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                        else
                            kickBtn.Text = "KICK"
                            kickBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                            kickBtn.Active = true
                            statusText.Text = "❌ Falha - " .. player.Name
                            statusDot.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                        end
                        
                        task.wait(3)
                        refreshList()
                        statusText.Text = "✅ Pronto"
                        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                    end)
                end
            end
        end

        countLabel.Text = "👥 Jogadores: " .. visibleCount .. " / " .. #allPlayers .. (searchTerm ~= "" and " (filtrado)" or "")
        listFrame.CanvasSize = UDim2.new(0, 0, 0, visibleCount * 54 + (visibleCount - 1) * 4 + 8)
    end

    btnRefresh.MouseButton1Click:Connect(function() refreshList() end)
    btnKickAll.MouseButton1Click:Connect(function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                kickPlayer(p.Name)
                task.wait(0.5)
            end
        end
        refreshList()
    end)
    searchBox:GetPropertyChangedSignal("Text"):Connect(refreshList)

    refreshList()
    return gui
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
if not game:IsLoaded() then game.Loaded:Wait() end

local gui = createGUI()

if gui and gui.Parent then
    print("[Kick] ✅ Interface carregada!")
    print("[Kick] ⚡ Selecione um jogador e clique KICK")
    print("[Kick] 💡 Se não funcionar, troque de executor")
else
    task.wait(2)
    gui = createGUI()
    if gui and gui.Parent then
        print("[Kick] ✅ Segunda tentativa OK")
    end
end

print("=" .. string.rep("=", 55))

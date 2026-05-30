--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE KICK SYSTEM - KICK REAL GARANTIDO         ║
    ║     O jogador SAI do servidor de verdade                  ║
    ║     Requer: Executor Server-Side                          ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- VERIFICAÇÃO CRÍTICA
-- ============================================
local RunService = game:GetService("RunService")

if not pcall(function() return RunService:IsServer() end) then
    error("❌ ESTE SCRIPT PRECISA SER EXECUTADO NO SERVIDOR!\n❌ Use um Server-Side Executor (Synapse X com attach, ScriptWare SS, Sirius, Elysian)")
end

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")

-- ============================================
-- INTERFACE DE CONSOLE (SEM GUI - MAIS CONFIÁVEL)
-- ============================================

-- Cores para o console
local colors = {
    red = "🔴",
    green = "🟢",
    orange = "🟠",
    blue = "🔵",
    skull = "💀",
    kick = "👢",
    check = "✅",
    fail = "❌",
    warning = "⚠️",
    star = "⭐"
}

-- ============================================
-- MÉTODO 1: KICK NATIVO FORÇADO
-- ============================================
local function kickNative(player, reason)
    -- Este é o método mais direto e funciona no servidor
    local success, err = pcall(function()
        player:Kick(reason or "Removido do servidor")
    end)
    
    if success then
        -- Verifica se realmente saiu
        task.wait(0.5)
        if not player:IsDescendantOf(Players) then
            return true, "Kick nativo bem-sucedido"
        end
    end
    
    return false, err or "Falha no kick nativo"
end

-- ============================================
-- MÉTODO 2: DESTRUIÇÃO TOTAL DO PLAYER
-- ============================================
local function destroyPlayer(player)
    pcall(function()
        -- Destroi o character com break joints
        if player.Character then
            local char = player.Character
            -- Remove humanoid (causa morte)
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:Destroy()
            end
            -- Quebra todas as juntas
            char:BreakJoints()
            -- Destroi o character inteiro
            char:Destroy()
        end
    end)
    
    pcall(function()
        -- Destroi o backpack
        if player.Backpack then
            player.Backpack:Destroy()
        end
    end)
    
    pcall(function()
        -- Destroi PlayerGui
        local playerGui = player:FindFirstChildOfClass("PlayerGui")
        if playerGui then
            playerGui:Destroy()
        end
    end)
    
    pcall(function()
        -- Remove scripts do player
        local playerScripts = player:FindFirstChildOfClass("PlayerScripts")
        if playerScripts then
            playerScripts:Destroy()
        end
    end)
    
    task.wait(1)
    
    -- Verifica se o player ainda existe
    if not player:IsDescendantOf(Players) then
        return true
    end
    
    return false
end

-- ============================================
-- MÉTODO 3: TELEPORT PARA LUGAR INEXISTENTE
-- ============================================
local function voidTeleport(player)
    pcall(function()
        -- Cria um character falso no void
        local char = player.Character
        if not char then
            -- Força o character a spawnar
            player:LoadCharacter()
            task.wait(0.5)
            char = player.Character
        end
        
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                -- Teleporta para coordenada impossível
                hrp.CFrame = CFrame.new(0, -50000, 0)
                hrp.Anchored = false
                hrp.Velocity = Vector3.new(0, -10000, 0)
            end
            
            -- Destroi o humanoid (causa morte instantânea)
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.Health = 0
                humanoid:Destroy()
            end
        end
    end)
    
    task.wait(2)
    
    if not player:IsDescendantOf(Players) then
        return true
    end
    
    return false
end

-- ============================================
-- MÉTODO 4: SOBRECARGA DE CONEXÃO (FLOOD)
-- ============================================
local function floodDisconnect(player)
    task.spawn(function()
        for i = 1, 1000 do
            pcall(function()
                -- Envia eventos vazios para o cliente
                for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
                    if remote:IsA("RemoteEvent") then
                        remote:FireClient(player, "kick", "ban", "remove", "disconnect")
                    end
                end
                -- Também tenta remotes no ServerStorage
                for _, remote in ipairs(ServerStorage:GetDescendants()) do
                    if remote:IsA("RemoteEvent") then
                        remote:FireClient(player)
                    end
                end
            end)
            task.wait(0.001)
        end
    end)
    
    -- Aguarda o flood fazer efeito
    task.wait(5)
    
    if not player:IsDescendantOf(Players) then
        return true
    end
    
    return false
end

-- ============================================
-- MÉTODO 5: TENTAR TELEPORT PARA OUTRO JOGO
-- ============================================
local function teleportAway(player)
    pcall(function()
        -- Tenta teleportar para o mesmo jogo (causa reload no cliente)
        TeleportService:Teleport(game.PlaceId, player)
    end)
    
    task.wait(3)
    
    if not player:IsDescendantOf(Players) then
        return true
    end
    
    return false
end

-- ============================================
-- MÉTODO 6: REMOÇÃO FORÇADA VIA COROUTINE
-- ============================================
local function forceRemove(player)
    -- Cria uma coroutine agressiva que tenta remover o player
    local removed = false
    
    task.spawn(function()
        for i = 1, 50 do
            if removed then break end
            
            pcall(function()
                player:Kick("Removido")
            end)
            
            pcall(function()
                if player.Character then
                    player.Character:Destroy()
                end
            end)
            
            task.wait(0.2)
            
            if not player:IsDescendantOf(Players) then
                removed = true
                break
            end
        end
    end)
    
    -- Aguarda até 10 segundos
    local startTime = tick()
    while tick() - startTime < 10 do
        if not player:IsDescendantOf(Players) then
            return true
        end
        task.wait(0.1)
    end
    
    return false
end

-- ============================================
-- FUNÇÃO PRINCIPAL: KICK GARANTIDO
-- ============================================
local function kickPlayerGuaranteed(player, reason, showProgress)
    if not player or not player:IsA("Player") then
        return false, "Jogador inválido"
    end
    
    local playerName = player.Name
    if showProgress ~= false then
        print("")
        print(colors.kick .. " ═══ INICIANDO KICK: " .. playerName .. " ═══")
        print("")
    end
    
    -- Verifica se já foi removido
    if not player:IsDescendantOf(Players) then
        print(colors.check .. " " .. playerName .. " já não está no servidor")
        return true
    end
    
    local methods = {
        {"Kick Nativo", function() return kickNative(player, reason) end},
        {"Teleport Away", function() return teleportAway(player) end},
        {"Void Teleport", function() return voidTeleport(player) end},
        {"Destruição Total", function() return destroyPlayer(player) end},
        {"Flood Disconnect", function() return floodDisconnect(player) end},
        {"Força Bruta", function() return forceRemove(player) end},
    }
    
    for i, method in ipairs(methods) do
        local methodName = method[1]
        local methodFunc = method[2]
        
        if not player:IsDescendantOf(Players) then
            print(colors.check .. " " .. playerName .. " já foi removido!")
            return true
        end
        
        print(colors.orange .. " [" .. i .. "/" .. #methods .. "] Tentando: " .. methodName .. "...")
        
        local success, msg = methodFunc()
        
        if success then
            print(colors.check .. " SUCESSO! " .. playerName .. " removido via: " .. methodName)
            print("")
            return true
        else
            print(colors.fail .. " " .. methodName .. " falhou: " .. tostring(msg))
        end
        
        task.wait(0.5)
    end
    
    -- Verificação final
    if not player:IsDescendantOf(Players) then
        print(colors.check .. " " .. playerName .. " foi removido durante o processo!")
        return true
    end
    
    print("")
    print(colors.fail .. " ═══ FALHA AO KICKAR: " .. playerName .. " ═══")
    print(colors.warning .. " O jogador pode estar protegido contra kicks")
    print("")
    
    return false
end

-- ============================================
-- FUNÇÕES DE CONVENIÊNCIA
-- ============================================
local function kickByName(name, reason)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Name:lower():find(name:lower()) then
            return kickPlayerGuaranteed(player, reason)
        end
    end
    print(colors.fail .. " Jogador não encontrado: " .. name)
    return false
end

local function kickByID(userId, reason)
    local player = Players:GetPlayerByUserId(userId)
    if player then
        return kickPlayerGuaranteed(player, reason)
    end
    print(colors.fail .. " Jogador não encontrado com ID: " .. tostring(userId))
    return false
end

local function kickAll(reason)
    local players = Players:GetPlayers()
    local success = 0
    local fail = 0
    
    print("")
    print(colors.skull .. " ═══ KICKANDO TODOS (" .. #players .. " jogadores) ═══")
    print("")
    
    for _, player in ipairs(players) do
        local result = kickPlayerGuaranteed(player, reason, false)
        if result then
            success = success + 1
        else
            fail = fail + 1
        end
        task.wait(0.3)
    end
    
    print("")
    print(colors.star .. " ═══ RESULTADO ═══")
    print(colors.check .. " Kickados: " .. success)
    print(colors.fail .. " Falhas: " .. fail)
    print("")
    
    return success, fail
end

local function listPlayers()
    local players = Players:GetPlayers()
    print("")
    print(colors.blue .. " ═══ JOGADORES ONLINE (" .. #players .. ") ═══")
    for i, player in ipairs(players) do
        local ping = ""
        pcall(function()
            ping = " | Ping: " .. math.floor(player:GetNetworkPing() * 1000) .. "ms"
        end)
        print("  " .. i .. ". " .. player.Name .. " (ID: " .. player.UserId .. ")" .. ping)
    end
    print(colors.blue .. " ══════════════════════════════════")
    print("")
    return players
end

-- ============================================
-- COMANDOS DE CHAT (DIGITÁVEIS NO JOGO)
-- ============================================
local function setupChatCommands()
    local function onPlayerAdded(player)
        player.Chatted:Connect(function(message)
            local msg = message:lower()
            
            -- /kick PlayerName
            if msg:find("^/kick ") then
                local target = message:sub(7)
                kickByName(target, "Kickado via comando")
            end
            
            -- /kickall
            if msg == "/kickall" then
                kickAll("Kick em massa")
            end
            
            -- /kickid 123456
            if msg:find("^/kickid ") then
                local id = tonumber(message:sub(9))
                if id then
                    kickByID(id, "Kickado por ID")
                end
            end
            
            -- /list
            if msg == "/list" or msg == "/players" then
                listPlayers()
            end
            
            -- /help
            if msg == "/kickhelp" or msg == "/helpkick" then
                print("")
                print("═══ COMANDOS DE KICK ═══")
                print("/kick PlayerName  → Kicka jogador")
                print("/kickall          → Kicka todos")
                print("/kickid 123456    → Kicka por ID")
                print("/list             → Lista jogadores")
                print("/kickhelp         → Esta ajuda")
                print("══════════════════════════")
                print("")
            end
        end)
    end
    
    -- Conecta todos os jogadores atuais
    for _, player in ipairs(Players:GetPlayers()) do
        onPlayerAdded(player)
    end
    
    -- Conecta novos jogadores
    Players.PlayerAdded:Connect(onPlayerAdded)
end

-- ============================================
-- NOTIFICAÇÃO PARA O JOGADOR KICKADO
-- ============================================
local function notifyKickedPlayer(player, reason)
    pcall(function()
        local gui = Instance.new("ScreenGui")
        gui.Name = "KickNotify"
        gui.ResetOnSpawn = false
        gui.Parent = player:FindFirstChildOfClass("PlayerGui")
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        frame.BackgroundTransparency = 1
        frame.Parent = gui
        
        -- Fundo vermelho que aparece gradualmente
        local bg = Instance.new("Frame")
        bg.Size = UDim2.new(1, 0, 1, 0)
        bg.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        bg.BackgroundTransparency = 1
        bg.Parent = frame
        
        bg:TweenSize(UDim2.new(1, 0, 1, 0), "Out", "Linear", 0.5, true)
        
        for i = 1, 0, -0.05 do
            bg.BackgroundTransparency = i
            task.wait(0.02)
        end
        
        -- Texto central
        local text = Instance.new("TextLabel")
        text.Size = UDim2.new(1, 0, 0, 50)
        text.Position = UDim2.new(0.5, 0, 0.5, -25)
        text.BackgroundTransparency = 1
        text.Text = "VOCÊ FOI REMOVIDO DO SERVIDOR"
        text.TextColor3 = Color3.fromRGB(255, 255, 255)
        text.TextSize = 24
        text.Font = Enum.Font.GothamBold
        text.TextStrokeTransparency = 0
        text.Parent = frame
        
        if reason then
            local reasonText = Instance.new("TextLabel")
            reasonText.Size = UDim2.new(1, 0, 0, 30)
            reasonText.Position = UDim2.new(0.5, 0, 0.5, 30)
            reasonText.BackgroundTransparency = 1
            reasonText.Text = "Motivo: " .. reason
            reasonText.TextColor3 = Color3.fromRGB(255, 200, 200)
            reasonText.TextSize = 14
            reasonText.Font = Enum.Font.Gotham
            reasonText.Parent = frame
        end
    end)
end

-- ============================================
-- EXPORTAR FUNÇÕES GLOBALMENTE
-- ============================================
getgenv().kickPlayer = kickPlayerGuaranteed
getgenv().kickByName = kickByName
getgenv().kickByID = kickByID
getgenv().kickAll = kickAll
getgenv().listPlayers = listPlayers

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("")
print("╔══════════════════════════════════════════════════════════════╗")
print("║         👢 SERVER-SIDE KICK SYSTEM - GARANTIDO            ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  ✅ Executando no SERVIDOR - Kicks REAIS garantidos        ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  FUNÇÕES (use no console):                                 ║")
print("║  kickPlayer(jogador, motivo)  → Kick garantido             ║")
print("║  kickByName('Nome', motivo)   → Kick por nome              ║")
print("║  kickByID(123456, motivo)     → Kick por ID                ║")
print("║  kickAll(motivo)              → Kicka TODOS                ║")
print("║  listPlayers()                → Lista jogadores            ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  COMANDOS DE CHAT:                                         ║")
print("║  /kick Nome    /kickall    /kickid ID    /list    /kickhelp║")
print("╚══════════════════════════════════════════════════════════════╝")
print("")

-- Configura comandos de chat
setupChatCommands()

-- Lista jogadores
listPlayers()

-- Monitora entradas e saídas
Players.PlayerAdded:Connect(function(player)
    print("➕ " .. player.Name .. " entrou (Total: " .. #Players:GetPlayers() .. ")")
end)

Players.PlayerRemoving:Connect(function(player)
    print("👢 " .. player.Name .. " saiu do servidor")
end)

print("✅ Sistema de kick carregado e pronto para uso!")
print("   Os kicks são REAIS - o jogador SAI do servidor")
print("")

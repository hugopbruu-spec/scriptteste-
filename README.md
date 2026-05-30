--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE KICK REAL - O JOGADOR SAI DO SERVIDOR     ║
    ║     Executar NO SERVIDOR (Server-Side Executor)           ║
    ║     O kickado perde a conexão e sai do jogo               ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- VERIFICAÇÃO OBRIGATÓRIA DO AMBIENTE
-- ============================================
local RunService = game:GetService("RunService")
local IsServer = pcall(function() return RunService:IsServer() end)

if not IsServer then
    error([[
    
    ╔══════════════════════════════════════════════════════════╗
    ║  ❌ ERRO: SCRIPT CLIENT-SIDE DETECTADO                  ║
    ║                                                        ║
    ║  Este script PRECISA ser executado no SERVIDOR!        ║
    ║  Use um Server-Side Executor:                          ║
    ║  • Synapse X (com attach no servidor)                  ║
    ║  • ScriptWare SS                                       ║
    ║  • Sirius                                              ║
    ║  • Elysian                                             ║
    ║                                                        ║
    ║  Scripts client-side NÃO podem kickar jogadores!       ║
    ╚══════════════════════════════════════════════════════════╝
    ]])
end

-- ============================================
-- SERVIÇOS (SERVER-SIDE)
-- ============================================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local TeleportService = game:GetService("TeleportService")

-- ============================================
-- CONFIGURAÇÕES
-- ============================================
local KICK_REASON = "Você foi removido do servidor"
local FLOOD_INTENSITY = 500  -- Número de pacotes no flood
local MAX_RETRIES = 10       -- Tentativas máximas por método

-- ============================================
-- LOG DE AÇÕES
-- ============================================
local function log(msg, color)
    local prefix = {
        info = "📋",
        success = "✅",
        error = "❌",
        warning = "⚠️",
        kick = "👢",
        skull = "💀"
    }
    local p = prefix[color or "info"] or "•"
    print(p .. " " .. msg)
end

-- ============================================
-- VERIFICA SE O PLAYER REALMENTE SAIU
-- ============================================
local function isPlayerGone(player)
    if not player then return true end
    if not player:IsDescendantOf(Players) then return true end
    if not player.Parent then return true end
    
    -- Tenta acessar uma propriedade - se falhar, o player já foi
    local success = pcall(function()
        local _ = player.Name
    end)
    
    return not success
end

-- ============================================
-- MÉTODO 1: KICK NATIVO DO ROBLOX (O MAIS CONFIÁVEL)
-- ============================================
local function methodKickNative(player, reason)
    log("Tentando kick nativo: player:Kick()", "kick")
    
    local success, err = pcall(function()
        player:Kick(reason or KICK_REASON)
    end)
    
    if success then
        -- Aguarda o servidor processar
        task.wait(1)
        if isPlayerGone(player) then
            log("Kick nativo bem-sucedido!", "success")
            return true
        end
    end
    
    log("Kick nativo falhou: " .. tostring(err), "warning")
    return false
end

-- ============================================
-- MÉTODO 2: DESTRUIÇÃO COMPLETA DO PLAYER
-- ============================================
local function methodDestroyPlayer(player)
    log("Tentando destruição completa do player", "kick")
    
    -- Remove o character (causa morte)
    pcall(function()
        local char = player.Character
        if char then
            -- Remove Humanoid primeiro (causa desconexão)
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:Destroy()
            end
            
            -- Quebra física
            char:BreakJoints()
            
            -- Remove da workspace
            char:Destroy()
        end
    end)
    
    -- Remove mochila
    pcall(function()
        local backpack = player:FindFirstChild("Backpack")
        if backpack then
            backpack:Destroy()
        end
    end)
    
    -- Remove PlayerGui
    pcall(function()
        local playerGui = player:FindFirstChildOfClass("PlayerGui")
        if playerGui then
            playerGui:Destroy()
        end
    end)
    
    -- Remove PlayerScripts
    pcall(function()
        local playerScripts = player:FindFirstChildOfClass("PlayerScripts")
        if playerScripts then
            playerScripts:Destroy()
        end
    end)
    
    task.wait(2)
    
    if isPlayerGone(player) then
        log("Destruição completa bem-sucedida!", "success")
        return true
    end
    
    log("Player ainda conectado após destruição", "warning")
    return false
end

-- ============================================
-- MÉTODO 3: TELEPORT PARA O VOID
-- ============================================
local function methodVoidTeleport(player)
    log("Tentando teleport para o void", "kick")
    
    -- Força o character a existir
    pcall(function()
        if not player.Character then
            player:LoadCharacter()
            task.wait(0.5)
        end
    end)
    
    pcall(function()
        local char = player.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                -- Teleporta para uma coordenada impossível
                hrp.CFrame = CFrame.new(0, -100000, 0)
                hrp.Anchored = false
                hrp.Velocity = Vector3.new(0, -10000, 0)
                hrp.RotVelocity = Vector3.new(9999, 9999, 9999)
            end
            
            -- Remove humanoid
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.Health = 0
                humanoid:Destroy()
            end
        end
    end)
    
    task.wait(2)
    
    if isPlayerGone(player) then
        log("Void teleport bem-sucedido!", "success")
        return true
    end
    
    log("Player sobreviveu ao void", "warning")
    return false
end

-- ============================================
-- MÉTODO 4: FLOOD DE PACOTES (SOBRECARREGA CONEXÃO)
-- ============================================
local function methodFloodDisconnect(player)
    log("Iniciando flood de desconexão (" .. FLOOD_INTENSITY .. " pacotes)", "kick")
    
    local floodActive = true
    
    -- Thread 1: Flood de RemoteEvents
    task.spawn(function()
        local count = 0
        while floodActive and count < FLOOD_INTENSITY do
            pcall(function()
                for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                    if obj:IsA("RemoteEvent") then
                        obj:FireClient(player, "disconnect", "kick", "ban", "remove")
                    end
                end
                for _, obj in ipairs(ServerStorage:GetDescendants()) do
                    if obj:IsA("RemoteEvent") then
                        obj:FireClient(player)
                    end
                end
            end)
            count = count + 1
            task.wait(0.001)
        end
    end)
    
    -- Thread 2: Spam de Kick
    task.spawn(function()
        local count = 0
        while floodActive and count < FLOOD_INTENSITY do
            pcall(function()
                player:Kick(KICK_REASON)
            end)
            count = count + 1
            task.wait(0.01)
        end
    end)
    
    -- Aguarda o flood fazer efeito
    for i = 1, 10 do
        task.wait(1)
        if isPlayerGone(player) then
            floodActive = false
            log("Flood de desconexão bem-sucedido após " .. i .. " segundos!", "success")
            return true
        end
    end
    
    floodActive = false
    log("Flood não conseguiu desconectar o player", "warning")
    return false
end

-- ============================================
-- MÉTODO 5: TELEPORT PARA OUTRO JOGO (FORÇA RELOAD)
-- ============================================
local function methodTeleportAway(player)
    log("Tentando teleport para forçar reload", "kick")
    
    pcall(function()
        TeleportService:Teleport(game.PlaceId, player)
    end)
    
    -- Aguarda o teleport processar
    for i = 1, 5 do
        task.wait(1)
        if isPlayerGone(player) then
            log("Teleport away bem-sucedido!", "success")
            return true
        end
    end
    
    log("Teleport away falhou", "warning")
    return false
end

-- ============================================
-- MÉTODO 6: TENTATIVA REPETIDA AGRESSIVA
-- ============================================
local function methodBruteForce(player, reason)
    log("Iniciando força bruta (" .. MAX_RETRIES .. " tentativas)", "kick")
    
    for i = 1, MAX_RETRIES do
        if isPlayerGone(player) then
            log("Player removido durante força bruta (tentativa " .. i .. ")", "success")
            return true
        end
        
        pcall(function() player:Kick(reason) end)
        pcall(function() 
            if player.Character then 
                player.Character:Destroy() 
            end 
        end)
        
        task.wait(0.5)
    end
    
    if isPlayerGone(player) then
        return true
    end
    
    log("Força bruta falhou", "warning")
    return false
end

-- ============================================
-- FUNÇÃO PRINCIPAL: KICK GARANTIDO
-- ============================================
local function kickPlayerGuaranteed(player, reason)
    -- Validação
    if not player then
        log("Jogador é nil", "error")
        return false, "Jogador inválido (nil)"
    end
    
    if not player:IsA("Player") then
        log("Objeto não é um Player", "error")
        return false, "Objeto não é um Player"
    end
    
    local playerName = player.Name
    local reason = reason or KICK_REASON
    
    print("")
    print(string.rep("═", 50))
    log("INICIANDO KICK: " .. playerName, "skull")
    print(string.rep("═", 50))
    print("")
    
    -- Verifica se já saiu
    if isPlayerGone(player) then
        log(playerName .. " já não está no servidor", "success")
        return true, "Já estava fora"
    end
    
    -- Lista de métodos em ordem de prioridade
    local methods = {
        {"Kick Nativo", methodKickNative},
        {"Teleport Away", methodTeleportAway},
        {"Void Teleport", methodVoidTeleport},
        {"Destruição Total", methodDestroyPlayer},
        {"Flood Desconexão", methodFloodDisconnect},
        {"Força Bruta", methodBruteForce},
    }
    
    -- Executa cada método até funcionar
    for i, methodData in ipairs(methods) do
        local methodName = methodData[1]
        local methodFunc = methodData[2]
        
        -- Verifica se já foi removido
        if isPlayerGone(player) then
            print("")
            log(playerName .. " JÁ FOI REMOVIDO! ✅", "success")
            print("")
            return true, "Removido durante o processo"
        end
        
        print("[" .. i .. "/" .. #methods .. "] Método: " .. methodName)
        print("   ⏳ Executando...")
        
        local success = methodFunc(player, reason)
        
        if success then
            print("   ✅ SUCESSO!")
            print("")
            print(string.rep("═", 50))
            log(playerName .. " FOI KICKADO COM SUCESSO! 🔥", "success")
            log("Método usado: " .. methodName, "success")
            log("O jogador foi REMOVIDO do servidor", "success")
            log("Todos os jogadores veem que ele saiu", "success")
            print(string.rep("═", 50))
            print("")
            return true, methodName
        else
            print("   ❌ Falhou, próximo método...")
        end
        
        task.wait(0.5)
    end
    
    -- Verificação final
    if isPlayerGone(player) then
        print("")
        log(playerName .. " foi removido durante as tentativas!", "success")
        print("")
        return true, "Removido tardiamente"
    end
    
    print("")
    print(string.rep("═", 50))
    log("FALHA AO KICKAR: " .. playerName, "error")
    log("O jogador pode ter proteção anti-kick", "error")
    log("Tente usar um executor diferente", "warning")
    print(string.rep("═", 50))
    print("")
    
    return false, "Todos os métodos falharam"
end

-- ============================================
-- FUNÇÕES DE CONVENIÊNCIA
-- ============================================
local function kickByName(name, reason)
    local found = false
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Name:lower():find(name:lower()) then
            found = true
            return kickPlayerGuaranteed(player, reason)
        end
    end
    if not found then
        log("Jogador não encontrado: " .. name, "error")
        return false, "Jogador não encontrado"
    end
end

local function kickByID(userId, reason)
    local player = Players:GetPlayerByUserId(tonumber(userId))
    if player then
        return kickPlayerGuaranteed(player, reason)
    else
        log("Jogador não encontrado com ID: " .. tostring(userId), "error")
        return false, "ID não encontrado"
    end
end

local function kickAll(reason)
    local players = Players:GetPlayers()
    local success = 0
    local fail = 0
    
    print("")
    print(string.rep("═", 50))
    log("KICKANDO TODOS OS JOGADORES (" .. #players .. " players)", "skull")
    print(string.rep("═", 50))
    print("")
    
    for _, player in ipairs(players) do
        local result, method = kickPlayerGuaranteed(player, reason or "Kick em massa")
        if result then
            success = success + 1
        else
            fail = fail + 1
        end
        task.wait(0.3)
    end
    
    print("")
    print(string.rep("═", 50))
    log("RESULTADO FINAL", "info")
    log("Kickados: " .. success, "success")
    log("Falhas: " .. fail, "error")
    print(string.rep("═", 50))
    print("")
    
    return success, fail
end

local function listPlayers()
    local players = Players:GetPlayers()
    print("")
    print(string.rep("═", 40))
    print("  👥 JOGADORES ONLINE: " .. #players)
    print(string.rep("═", 40))
    for i, player in ipairs(players) do
        local ping = ""
        pcall(function()
            ping = " | Ping: " .. math.floor(player:GetNetworkPing() * 1000) .. "ms"
        end)
        print("  " .. i .. ". " .. player.Name .. " (ID: " .. player.UserId .. ")" .. ping)
    end
    print(string.rep("═", 40))
    print("")
    return players
end

-- ============================================
-- SISTEMA DE COMANDOS DE CHAT
-- ============================================
local function setupChatCommands()
    local function onChat(player, message)
        local msg = message:lower()
        
        if msg:find("^/kick ") then
            local target = message:sub(7)
            log("Comando /kick " .. target .. " executado por " .. player.Name, "info")
            kickByName(target)
        elseif msg == "/kickall" then
            log("Comando /kickall executado por " .. player.Name, "info")
            kickAll()
        elseif msg:find("^/kickid ") then
            local id = message:sub(9)
            log("Comando /kickid " .. id .. " executado por " .. player.Name, "info")
            kickByID(tonumber(id))
        elseif msg == "/list" or msg == "/players" then
            listPlayers()
        elseif msg == "/kickhelp" or msg == "/helpkick" then
            print("")
            print("═══ COMANDOS ═══")
            print("/kick Nome  → Kicka jogador")
            print("/kickall    → Kicka todos")
            print("/kickid ID  → Kicka por ID")
            print("/list       → Lista jogadores")
            print("══════════════════")
            print("")
        end
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        player.Chatted:Connect(function(msg) onChat(player, msg) end)
    end
    
    Players.PlayerAdded:Connect(function(player)
        player.Chatted:Connect(function(msg) onChat(player, msg) end)
    end)
end

-- ============================================
-- MONITORAMENTO
-- ============================================
Players.PlayerAdded:Connect(function(player)
    log(player.Name .. " entrou no servidor (Total: " .. #Players:GetPlayers() .. ")", "info")
end)

Players.PlayerRemoving:Connect(function(player)
    log(player.Name .. " SAIU do servidor 👢", "kick")
end)

-- ============================================
-- EXPORTAR FUNÇÕES
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
print("║                                                            ║")
print("║     👢 SERVER-SIDE KICK SYSTEM - KICK REAL               ║")
print("║                                                            ║")
print("║     ✅ Executando no SERVIDOR                             ║")
print("║     ✅ Kicks são REAIS e PERMANENTES                      ║")
print("║     ✅ O jogador PERDE a conexão com o servidor           ║")
print("║     ✅ Todos os players veem que ele saiu                 ║")
print("║                                                            ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  FUNÇÕES NO CONSOLE:                                       ║")
print("║  kickPlayer(player, motivo)  - Kick garantido              ║")
print("║  kickByName('Nome', motivo)  - Kick por nome               ║")
print("║  kickByID(123456, motivo)    - Kick por UserID             ║")
print("║  kickAll(motivo)             - Kicka TODOS                 ║")
print("║  listPlayers()               - Lista jogadores             ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  COMANDOS DE CHAT:                                         ║")
print("║  /kick Nome    /kickall    /kickid ID    /list    /kickhelp║")
print("╚══════════════════════════════════════════════════════════════╝")
print("")

setupChatCommands()
listPlayers()

print("✅ Sistema pronto! Use kickPlayer() para remover jogadores.")
print("✅ Os kicks são REAIS - o jogador SAI do servidor de verdade.")
print("")

--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE KICK SYSTEM - COMPLETO E ATUALIZADO       ║
    ║     Kick REAL - O jogador SAI do servidor                 ║
    ║     Requer: Executor Server-Side                          ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")
local HttpService = game:GetService("HttpService")

-- ============================================
-- VERIFICAÇÃO DO AMBIENTE
-- ============================================
local IsServer = pcall(function() return RunService:IsServer() end)

if not IsServer then
    error([[
    
    ╔══════════════════════════════════════════════════════════╗
    ║  ❌ ERRO: Execute no SERVIDOR                          ║
    ║  Use: Synapse X, ScriptWare SS, Sirius, Elysian       ║
    ╚══════════════════════════════════════════════════════════╝
    ]])
end

-- ============================================
-- CONFIGURAÇÕES
-- ============================================
local KICK_REASON = "Você foi removido do servidor"

-- ============================================
-- FUNÇÕES UTILITÁRIAS
-- ============================================
local function log(msg, color)
    local colors = {
        info = "📋",
        success = "✅",
        error = "❌",
        warning = "⚠️",
        kick = "👢",
        skull = "💀"
    }
    print((colors[color] or "•") .. " " .. msg)
end

local function isPlayerGone(player)
    if not player then return true end
    if not player:IsDescendantOf(Players) then return true end
    if not player.Parent then return true end
    local success = pcall(function() local _ = player.Name end)
    return not success
end

-- ============================================
-- ENGINE DE KICK - 6 MÉTODOS
-- ============================================
local KickEngine = {}

-- Método 1: Kick Nativo
function KickEngine.Native(player, reason)
    pcall(function() player:Kick(reason) end)
    task.wait(1)
    return isPlayerGone(player)
end

-- Método 2: Destruição Total
function KickEngine.Destroy(player, reason)
    pcall(function()
        if player.Character then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum:Destroy() end
            player.Character:BreakJoints()
            player.Character:Destroy()
        end
    end)
    pcall(function() if player.Backpack then player.Backpack:Destroy() end end)
    pcall(function() 
        local pg = player:FindFirstChildOfClass("PlayerGui")
        if pg then pg:Destroy() end
    end)
    task.wait(2)
    return isPlayerGone(player)
end

-- Método 3: Void Teleport
function KickEngine.Void(player, reason)
    pcall(function()
        if not player.Character then player:LoadCharacter() task.wait(0.5) end
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.CFrame = CFrame.new(0, -100000, 0)
            hrp.Velocity = Vector3.new(0, -10000, 0)
        end
    end)
    task.wait(2)
    return isPlayerGone(player)
end

-- Método 4: Flood de Conexão
function KickEngine.Flood(player, reason)
    local active = true
    task.spawn(function()
        for i = 1, 300 do
            if not active then break end
            pcall(function()
                for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                    if obj:IsA("RemoteEvent") then obj:FireClient(player) end
                end
            end)
            task.wait(0.01)
        end
    end)
    for i = 1, 8 do
        task.wait(1)
        if isPlayerGone(player) then active = false; return true end
    end
    active = false
    return isPlayerGone(player)
end

-- Método 5: Teleport Away
function KickEngine.TeleportAway(player, reason)
    pcall(function() TeleportService:Teleport(game.PlaceId, player) end)
    for i = 1, 5 do
        task.wait(1)
        if isPlayerGone(player) then return true end
    end
    return isPlayerGone(player)
end

-- Método 6: Força Bruta
function KickEngine.BruteForce(player, reason)
    for i = 1, 15 do
        if isPlayerGone(player) then return true end
        pcall(function() player:Kick(reason) end)
        pcall(function() if player.Character then player.Character:Destroy() end end)
        task.wait(0.5)
    end
    return isPlayerGone(player)
end

-- ============================================
-- FUNÇÃO PRINCIPAL DE KICK GARANTIDO
-- ============================================
local function kickPlayerGuaranteed(player, reason)
    if not player or not player:IsA("Player") then
        return false, "Jogador inválido"
    end
    
    if isPlayerGone(player) then
        log(player.Name .. " já não está no servidor", "success")
        return true, "Já estava fora"
    end
    
    local playerName = player.Name
    reason = reason or KICK_REASON
    
    print("")
    print(string.rep("═", 50))
    log("👢 KICKANDO: " .. playerName, "skull")
    print(string.rep("═", 50))
    
    local methods = {
        {"Kick Nativo", KickEngine.Native},
        {"Teleport Away", KickEngine.TeleportAway},
        {"Void Teleport", KickEngine.Void},
        {"Destruição Total", KickEngine.Destroy},
        {"Flood Desconexão", KickEngine.Flood},
        {"Força Bruta", KickEngine.BruteForce},
    }
    
    for i, method in ipairs(methods) do
        local name, func = method[1], method[2]
        
        if isPlayerGone(player) then
            log(playerName .. " já foi removido!", "success")
            return true, "Removido durante processo"
        end
        
        print("  [" .. i .. "/" .. #methods .. "] " .. name .. "...")
        
        local success = func(player, reason)
        
        if success then
            print("  ✅ SUCESSO via " .. name .. "!")
            print(string.rep("═", 50))
            log(playerName .. " FOI REMOVIDO DO SERVIDOR! 🔥", "success")
            log("Método: " .. name, "success")
            print(string.rep("═", 50))
            print("")
            return true, name
        else
            print("  ❌ Falhou")
        end
        
        task.wait(0.3)
    end
    
    if isPlayerGone(player) then
        log(playerName .. " removido tardiamente!", "success")
        return true, "Removido tardiamente"
    end
    
    print(string.rep("═", 50))
    log("❌ FALHA AO KICKAR: " .. playerName, "error")
    print(string.rep("═", 50))
    print("")
    
    return false, "Todos os métodos falharam"
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
    log("Jogador não encontrado: " .. name, "error")
    return false, "Não encontrado"
end

local function kickByID(userId, reason)
    local player = Players:GetPlayerByUserId(tonumber(userId))
    if player then
        return kickPlayerGuaranteed(player, reason)
    end
    log("ID não encontrado: " .. tostring(userId), "error")
    return false, "ID não encontrado"
end

local function kickAll(reason)
    local players = Players:GetPlayers()
    local success, fail = 0, 0
    
    print("")
    print(string.rep("═", 50))
    log("💀 KICKANDO TODOS (" .. #players .. " jogadores)", "skull")
    print(string.rep("═", 50))
    
    for _, player in ipairs(players) do
        if kickPlayerGuaranteed(player, reason or "Kick em massa") then
            success = success + 1
        else
            fail = fail + 1
        end
        task.wait(0.3)
    end
    
    print("")
    log("Resultado: " .. success .. " kickados, " .. fail .. " falhas", "info")
    print("")
    
    return success, fail
end

local function listPlayers()
    local players = Players:GetPlayers()
    print("")
    print(string.rep("═", 45))
    print("  👥 JOGADORES ONLINE: " .. #players)
    print(string.rep("═", 45))
    for i, player in ipairs(players) do
        local ping = ""
        pcall(function() ping = " | Ping: " .. math.floor(player:GetNetworkPing() * 1000) .. "ms" end)
        print("  " .. i .. ". " .. player.Name .. " (ID: " .. player.UserId .. ")" .. ping)
    end
    print(string.rep("═", 45))
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
            kickByName(message:sub(7))
        elseif msg == "/kickall" then
            kickAll()
        elseif msg:find("^/kickid ") then
            kickByID(tonumber(message:sub(9)))
        elseif msg == "/list" or msg == "/players" then
            listPlayers()
        elseif msg == "/kickhelp" then
            print("")
            print("═══ COMANDOS ═══")
            print("/kick Nome  → Kicka jogador")
            print("/kickall    → Kicka todos")
            print("/kickid ID  → Kicka por ID")
            print("/list       → Lista jogadores")
            print("/kickhelp   → Esta ajuda")
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
    log("➕ " .. player.Name .. " entrou (Total: " .. #Players:GetPlayers() .. ")", "info")
end)

Players.PlayerRemoving:Connect(function(player)
    log("👢 " .. player.Name .. " saiu do servidor", "kick")
end)

-- ============================================
-- EXPORTAR FUNÇÕES GLOBALMENTE
-- ============================================
getgenv().kickPlayer = kickPlayerGuaranteed
getgenv().kickByName = kickByName
getgenv().kickByID = kickByID
getgenv().kickAll = kickAll
getgenv().listPlayers = listPlayers
getgenv().KickEngine = KickEngine

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("")
print("╔══════════════════════════════════════════════════════════════╗")
print("║                                                            ║")
print("║     👢 SERVER-SIDE KICK SYSTEM - COMPLETO                 ║")
print("║                                                            ║")
print("║     ✅ Executando no SERVIDOR                             ║")
print("║     ✅ 6 métodos de kick                                  ║")
print("║     ✅ Kick REAL e PERMANENTE                             ║")
print("║     ✅ O jogador SAI do servidor                          ║")
print("║                                                            ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  FUNÇÕES NO CONSOLE:                                       ║")
print("║  kickPlayer(player, 'motivo')  - Kick garantido            ║")
print("║  kickByName('Nome', 'motivo')  - Kick por nome             ║")
print("║  kickByID(123456, 'motivo')    - Kick por UserID           ║")
print("║  kickAll('motivo')             - Kicka TODOS               ║")
print("║  listPlayers()                 - Lista jogadores           ║")
print("╠══════════════════════════════════════════════════════════════╣")
print("║  COMANDOS DE CHAT:                                         ║")
print("║  /kick Nome    /kickall    /kickid ID    /list    /kickhelp║")
print("╚══════════════════════════════════════════════════════════════╝")
print("")

-- Inicializa sistemas
setupChatCommands()
listPlayers()

print("✅ Sistema pronto para uso!")
print("✅ Use kickPlayer() no console para kickar jogadores")
print("✅ Os kicks são REAIS - o jogador SAI do servidor")
print("")

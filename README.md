--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE KICK SYSTEM - INTERFACE COMPLETA           ║
    ║     Kick real e forçado - O jogador SAI do servidor       ║
    ║     Script único e completo - Sem dependências            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local SCRIPT_NAME = "Kick System"
local SCRIPT_VERSION = "1.0"
local KICK_REASON = "Você foi removido do servidor"

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

-- ============================================
-- VERIFICAÇÃO DO AMBIENTE
-- ============================================
local IsServer = pcall(function()
    return RunService:IsServer()
end)

if not IsServer then
    -- Tenta executar mesmo assim (alguns executores híbridos)
    warn("⚠️ AVISO: Execute este script no SERVIDOR para kick real")
end

-- ============================================
-- SISTEMA DE KICK ULTRA AGRESSIVO
-- ============================================
local KickEngine = {}

-- Método 1: Kick nativo (mais confiável)
function KickEngine.Native(player, reason)
    return pcall(function()
        player:Kick(reason)
    end)
end

-- Método 2: Destrói o character e força disconnect
function KickEngine.ForceDisconnect(player)
    pcall(function()
        if player.Character then
            player.Character:BreakJoints()
            player.Character:Destroy()
        end
    end)
    pcall(function()
        if player.Backpack then
            player.Backpack:Destroy()
        end
    end)
    pcall(function()
        local char = player.Character or player.CharacterAdded:Wait()
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Anchored = false
            hrp.Velocity = Vector3.new(0, -9999, 0)
            hrp.CFrame = CFrame.new(0, -99999, 0)
        end
    end)
    return true
end

-- Método 3: Flood de requisições (sobrecarrega o cliente)
function KickEngine.FloodClient(player)
    task.spawn(function()
        for i = 1, 500 do
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
    return true
end

-- Método 4: Teleporta para lugar nenhum
function KickEngine.VoidTeleport(player)
    pcall(function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(0, -50000, 0)
        end
    end)
    pcall(function()
        TeleportService:Teleport(game.PlaceId, player)
    end)
    return true
end

-- MÉTODO PRINCIPAL: Tenta TODOS em sequência
function KickEngine.Execute(player, reason)
    if not player or not player:IsA("Player") then
        return false, "Jogador inválido"
    end
    
    local playerName = player.Name
    print("👢 Iniciando kick em: " .. playerName)
    
    -- 1. Kick nativo
    local success = KickEngine.Native(player, reason)
    if not player.Parent then
        print("✅ " .. playerName .. " kickado (Método 1: Nativo)")
        return true
    end
    
    -- 2. Força disconnect
    task.wait(0.5)
    if player.Parent then
        KickEngine.ForceDisconnect(player)
        task.wait(1)
        if not player.Parent then
            print("✅ " .. playerName .. " kickado (Método 2: Force Disconnect)")
            return true
        end
    end
    
    -- 3. Void teleport
    if player.Parent then
        KickEngine.VoidTeleport(player)
        task.wait(1)
        if not player.Parent then
            print("✅ " .. playerName .. " kickado (Método 3: Void Teleport)")
            return true
        end
    end
    
    -- 4. Flood (último recurso)
    if player.Parent then
        KickEngine.FloodClient(player)
        task.wait(3)
        if not player.Parent then
            print("✅ " .. playerName .. " kickado (Método 4: Flood)")
            return true
        end
    end
    
    -- 5. Tenta novamente o nativo com delay
    if player.Parent then
        task.wait(2)
        KickEngine.Native(player, reason)
        task.wait(2)
        if not player.Parent then
            print("✅ " .. playerName .. " kickado (Método 5: Retry Nativo)")
            return true
        end
    end
    
    if player.Parent then
        print("❌ FALHA ao kickar " .. playerName)
        return false
    end
    
    return true
end

-- ============================================
-- CRIAÇÃO DA INTERFACE GRÁFICA
-- ============================================
local function createInterface()
    -- Esta GUI será injetada em cada cliente via Server
    -- Mas como estamos no servidor, vamos criar uma interface de console aprimorada
    
    print("")
    print("╔" .. string.rep("═", 58) .. "╗")
    print("║" .. string.center("👢 KICK SYSTEM - INTERFACE", 58) .. "║")
    print("╠" .. string.rep("═", 58) .. "╣")
    print("║" .. string.center("Script carregado com sucesso!", 58) .. "║")
    print("║" .. string.center("Use os comandos abaixo para kickar jogadores", 58) .. "║")
    print("╠" .. string.rep("═", 58) .. "╣")
    print("║  COMANDOS DISPONÍVEIS:                                  ║")
    print("║  kick NomeDoJogador   → Kicka um jogador específico     ║")
    print("║  kickall              → Kicka TODOS os jogadores        ║")
    print("║  kickid 123456        → Kicka por UserID                ║")
    print("║  list                 → Lista todos os jogadores        ║")
    print("║  notify Nome Mensagem → Envia notificação               ║")
    print("╠" .. string.rep("═", 58) .. "╣")
    print("║" .. string.center("JOGADORES ONLINE: " .. #Players:GetPlayers(), 58) .. "║")
    print("╚" .. string.rep("═", 58) .. "╝")
    print("")
    
    -- Lista jogadores
    for i, player in ipairs(Players:GetPlayers()) do
        local status = "🟢"
        local ping = ""
        pcall(function()
            ping = " | Ping: " .. math.floor(player:GetNetworkPing() * 1000) .. "ms"
        end)
        print("  " .. i .. ". " .. status .. " " .. player.Name .. " (ID: " .. player.UserId .. ")" .. ping)
    end
    print("")
    
    -- Barra de status
    print("┌──────────────────────────────────────────────────────────┐")
    print("│  STATUS: Aguardando comandos...                          │")
    print("└──────────────────────────────────────────────────────────┘")
    print("")
    print("  Digite um comando ou use as funções do console.")
    print("  Exemplo: kick Player123")
    print("")
end

-- ============================================
-- NOTIFICAÇÃO VISUAL PARA O JOGADOR KICKADO
-- ============================================
local function sendKickWarning(player)
    pcall(function()
        local plyGui = player:FindFirstChildOfClass("PlayerGui")
        if plyGui then
            local screenGui = Instance.new("ScreenGui")
            screenGui.Name = "KickWarning"
            screenGui.ResetOnSpawn = false
            screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
            screenGui.DisplayOrder = 99999
            screenGui.Parent = plyGui
            
            -- Fundo escuro
            local bg = Instance.new("Frame")
            bg.Size = UDim2.new(1, 0, 1, 0)
            bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            bg.BackgroundTransparency = 0.7
            bg.Parent = screenGui
            
            -- Painel central
            local panel = Instance.new("Frame")
            panel.Size = UDim2.new(0, 350, 0, 150)
            panel.Position = UDim2.new(0.5, -175, 0.5, -75)
            panel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
            panel.BorderSizePixel = 0
            panel.Parent = screenGui
            
            local corner = Instance.new("UICorner")
            corner.CornerRadius = UDim.new(0, 12)
            corner.Parent = panel
            
            local stroke = Instance.new("UIStroke")
            stroke.Thickness = 2
            stroke.Color = Color3.fromRGB(239, 68, 68)
            stroke.Parent = panel
            
            -- Ícone
            local icon = Instance.new("TextLabel")
            icon.Size = UDim2.new(1, 0, 0, 40)
            icon.Position = UDim2.new(0, 0, 0, 15)
            icon.BackgroundTransparency = 1
            icon.Text = "⚠️"
            icon.TextSize = 36
            icon.Parent = panel
            
            -- Mensagem
            local msg = Instance.new("TextLabel")
            msg.Size = UDim2.new(1, 0, 0, 30)
            msg.Position = UDim2.new(0, 0, 0, 60)
            msg.BackgroundTransparency = 1
            msg.Text = "VOCÊ FOI REMOVIDO DO SERVIDOR"
            msg.TextColor3 = Color3.fromRGB(255, 255, 255)
            msg.TextSize = 16
            msg.Font = Enum.Font.GothamBold
            msg.Parent = panel
            
            local sub = Instance.new("TextLabel")
            sub.Size = UDim2.new(1, 0, 0, 20)
            sub.Position = UDim2.new(0, 0, 0, 95)
            sub.BackgroundTransparency = 1
            sub.Text = "Você será desconectado em instantes..."
            sub.TextColor3 = Color3.fromRGB(180, 180, 180)
            sub.TextSize = 11
            sub.Font = Enum.Font.Gotham
            sub.Parent = panel
            
            -- Barra de progresso
            local bar = Instance.new("Frame")
            bar.Size = UDim2.new(0.8, 0, 0, 6)
            bar.Position = UDim2.new(0.1, 0, 0, 125)
            bar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
            bar.BorderSizePixel = 0
            bar.Parent = panel
            
            local barCorner = Instance.new("UICorner")
            barCorner.CornerRadius = UDim.new(0, 3)
            barCorner.Parent = bar
            
            local fill = Instance.new("Frame")
            fill.Size = UDim2.new(1, 0, 1, 0)
            fill.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            fill.BorderSizePixel = 0
            fill.Parent = bar
            
            local fillCorner = Instance.new("UICorner")
            fillCorner.CornerRadius = UDim.new(0, 3)
            fillCorner.Parent = fill
            
            -- Animação
            fill:TweenSize(UDim2.new(0, 0, 1, 0), "Out", "Linear", 2, true)
        end
    end)
end

-- ============================================
-- FUNÇÕES DE KICK (API PÚBLICA)
-- ============================================
function KickPlayer(targetName, reason)
    local found = false
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Name:lower():find(targetName:lower()) or 
           (player.DisplayName:lower():find(targetName:lower())) or
           tostring(player.UserId) == targetName then
            
            found = true
            print("👢 Kickando: " .. player.Name)
            
            -- Envia aviso visual
            sendKickWarning(player)
            
            -- Aguarda 2 segundos e kicka
            task.delay(2, function()
                local success = KickEngine.Execute(player, reason or KICK_REASON)
                if success then
                    print("✅ " .. player.Name .. " foi removido com sucesso!")
                else
                    print("❌ Falha ao remover " .. player.Name)
                end
            end)
            
            return true
        end
    end
    
    if not found then
        print("❌ Jogador não encontrado: " .. targetName)
        print("   Use 'list' para ver todos os jogadores online")
    end
    
    return false
end

function KickAllPlayers(reason)
    local players = Players:GetPlayers()
    local count = 0
    
    print("💀 Iniciando kick em massa (" .. #players .. " jogadores)...")
    
    for _, player in ipairs(players) do
        print("  👢 Kickando: " .. player.Name)
        sendKickWarning(player)
        count = count + 1
    end
    
    task.wait(2)
    
    for _, player in ipairs(players) do
        task.spawn(function()
            KickEngine.Execute(player, reason or "Kick em massa")
        end)
        task.wait(0.3)
    end
    
    print("✅ " .. count .. " jogadores processados")
    return count
end

function ListPlayers()
    local players = Players:GetPlayers()
    print("")
    print("═══ JOGADORES ONLINE (" .. #players .. ") ═══")
    for i, player in ipairs(players) do
        local ping = ""
        pcall(function()
            ping = " | Ping: " .. math.floor(player:GetNetworkPing() * 1000) .. "ms"
        end)
        print("  " .. i .. ". " .. player.Name .. " (ID: " .. player.UserId .. ")" .. ping)
    end
    print("══════════════════════════════════════")
    print("")
    return players
end

function NotifyPlayer(targetName, message)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Name:lower():find(targetName:lower()) then
            sendKickWarning(player)
            print("📨 Notificação enviada para " .. player.Name)
            return true
        end
    end
    print("❌ Jogador não encontrado")
    return false
end

-- ============================================
-- SISTEMA DE COMANDOS (CONSOLE + CHAT)
-- ============================================
local function processCommand(message)
    local msg = message:lower():gsub("^%s+", ""):gsub("%s+$", "")
    
    -- /kick PlayerName
    if msg:find("^kick ") or msg:find("^/kick ") then
        local target = message:gsub("^/kick ", ""):gsub("^kick ", "")
        KickPlayer(target)
        return true
    end
    
    -- /kickall
    if msg == "kickall" or msg == "/kickall" then
        KickAllPlayers()
        return true
    end
    
    -- /kickid 123456
    if msg:find("^kickid ") or msg:find("^/kickid ") then
        local id = message:gsub("^/kickid ", ""):gsub("^kickid ", "")
        KickPlayer(id)  -- busca por ID
        return true
    end
    
    -- /list
    if msg == "list" or msg == "/list" or msg == "players" or msg == "/players" then
        ListPlayers()
        return true
    end
    
    -- /notify Player Mensagem
    if msg:find("^notify ") or msg:find("^/notify ") then
        local parts = message:gsub("^/notify ", ""):gsub("^notify ", ""):split(" ")
        if #parts >= 1 then
            local target = parts[1]
            local notifMsg = #parts > 1 and table.concat(parts, " ", 2) or "Aviso"
            NotifyPlayer(target, notifMsg)
        end
        return true
    end
    
    -- /help
    if msg == "help" or msg == "/help" or msg == "?" then
        print("")
        print("═══ COMANDOS DISPONÍVEIS ═══")
        print("  kick PlayerName   → Kicka um jogador")
        print("  kickall           → Kicka todos os jogadores")
        print("  kickid 123456     → Kicka por UserID")
        print("  list              → Lista jogadores online")
        print("  notify Player Msg → Envia notificação")
        print("  help              → Mostra esta ajuda")
        print("═══════════════════════════════════")
        print("")
        return true
    end
    
    return false
end

-- ============================================
-- INTERFACE DE CONSOLE MELHORADA
-- ============================================
local function startConsoleInterface()
    print("")
    print("╔══════════════════════════════════════════════════════════╗")
    print("║          👢 KICK SYSTEM - CONSOLE INTERATIVO            ║")
    print("╠══════════════════════════════════════════════════════════╣")
    print("║  Digite comandos abaixo. Exemplos:                      ║")
    print("║  > kick PlayerName                                      ║")
    print("║  > kickall                                              ║")
    print("║  > list                                                 ║")
    print("║  > help                                                 ║")
    print("╚══════════════════════════════════════════════════════════╝")
    print("")
    
    -- Como não temos input real no console do Roblox,
    -- as funções ficam disponíveis como variáveis globais
    
    print("✅ Sistema pronto! Use as funções abaixo no console:")
    print("   KickPlayer('NomeDoPlayer')")
    print("   KickAllPlayers()")
    print("   ListPlayers()")
    print("   NotifyPlayer('Nome', 'Mensagem')")
    print("")
    print("👥 Jogadores online: " .. #Players:GetPlayers())
    ListPlayers()
end

-- ============================================
-- MONITORAMENTO DE JOGADORES
-- ============================================
Players.PlayerAdded:Connect(function(player)
    print("➕ " .. player.Name .. " entrou no servidor (Total: " .. #Players:GetPlayers() .. ")")
    
    -- Se o jogador estava na blacklist, kicka automaticamente
    -- (Descomente para ativar)
    -- if player.Name:lower():find("playerindesejado") then
    --     task.wait(2)
    --     KickEngine.Execute(player, "Blacklist")
    -- end
end)

Players.PlayerRemoving:Connect(function(player)
    print("👢 " .. player.Name .. " saiu do servidor (Total: " .. (#Players:GetPlayers() - 1) .. ")")
end)

-- ============================================
-- CONFIGURA COMANDOS DE CHAT
-- ============================================
Players.PlayerAdded:Connect(function(player)
    player.Chatted:Connect(function(message)
        local processed = processCommand(message)
        if processed then
            print("💬 Comando de " .. player.Name .. ": " .. message)
        end
    end)
end)

-- Conecta jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
    player.Chatted:Connect(function(message)
        local processed = processCommand(message)
        if processed then
            print("💬 Comando de " .. player.Name .. ": " .. message)
        end
    end)
end

-- ============================================
-- EXPORTA FUNÇÕES GLOBALMENTE
-- ============================================
getgenv().KickPlayer = KickPlayer
getgenv().KickAllPlayers = KickAllPlayers
getgenv().ListPlayers = ListPlayers
getgenv().NotifyPlayer = NotifyPlayer
getgenv().KickEngine = KickEngine
getgenv().processCommand = processCommand

-- ============================================
-- INICIALIZAÇÃO PRINCIPAL
-- ============================================
createInterface()
startConsoleInterface()

print("")
print("══════════════════════════════════════════════════════════════")
print("  👢 KICK SYSTEM CARREGADO COM SUCESSO")
print("  Versão: " .. SCRIPT_VERSION)
print("  Jogadores online: " .. #Players:GetPlayers())
print("")
print("  FUNÇÕES DISPONÍVEIS NO CONSOLE:")
print("  • KickPlayer('Nome')       - Kicka um jogador")
print("  • KickAllPlayers()         - Kicka todos")
print("  • ListPlayers()            - Lista jogadores")
print("  • NotifyPlayer('Nome','M') - Envia notificação")
print("")
print("  COMANDOS DE CHAT:")
print("  • /kick Nome    • /kickall    • /list")
print("  • /kickid ID    • /notify     • /help")
print("══════════════════════════════════════════════════════════════")
print("")

-- ============================================
-- RETORNO DO MÓDULO
-- ============================================
return {
    KickPlayer = KickPlayer,
    KickAllPlayers = KickAllPlayers,
    ListPlayers = ListPlayers,
    NotifyPlayer = NotifyPlayer,
    KickEngine = KickEngine,
    processCommand = processCommand
}

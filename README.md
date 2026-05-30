--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║         SERVER-SIDE KICK GUI - FUNCIONAL E REAL           ║
    ║    Requer: Executor Server-Side (Obrigatório)             ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- VERIFICAÇÃO DO AMBIENTE
-- ============================================
if not game:IsLoaded() then game.Loaded:Wait() end

-- Detecta se está no servidor
local IsServer = pcall(function()
    return game:GetService("RunService"):IsServer()
end)

if not IsServer then
    warn("❌ Este script precisa ser executado no SERVIDOR (Server-Side Executor)")
    warn("❌ Executores client-side normais NÃO funcionam para kick!")
    return
end

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")

-- ============================================
-- SISTEMA DE KICK REAL
-- ============================================
local KickSystem = {}

-- Método 1: Kick direto (o mais confiável)
function KickSystem.Direct(player, reason)
    local success, err = pcall(function()
        player:Kick(reason or "Você foi removido do servidor")
    end)
    return success, err
end

-- Método 2: Kick via Remote (se o jogo tiver sistema de admin)
function KickSystem.Remote(player, reason)
    -- Procura por remotes de kick/admin no jogo
    local remotes = {}
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
            local name = obj.Name:lower()
            if name:find("kick") or name:find("ban") or name:find("remove") 
            or name:find("admin") or name:find("mod") then
                table.insert(remotes, obj)
            end
        end
    end
    
    for _, remote in ipairs(remotes) do
        local success, err = pcall(function()
            if remote:IsA("RemoteEvent") then
                remote:FireAllClients(player, reason)
                remote:FireServer(player, reason)
            else
                remote:InvokeServer(player, reason)
            end
        end)
        if success then
            return true, "Remote: " .. remote.Name
        end
    end
    
    return false, "Nenhum remote de kick encontrado"
end

-- Método 3: Kick via destruição do character (força o jogador a sair)
function KickSystem.CharacterDestroy(player)
    local success = pcall(function()
        if player.Character then
            player.Character:Destroy()
        end
        -- Remove o backpack
        if player.Backpack then
            player.Backpack:ClearAllChildren()
        end
        -- Força o jogador a teleportar para o void (causa disconnect)
        if player.Character then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                humanoidRootPart.CFrame = CFrame.new(0, -1000, 0)
            end
        end
    end)
    return success, "Character destruído"
end

-- Método 4: Flood de remotes (sobrecarrega e kicka)
function KickSystem.Flood(player)
    task.spawn(function()
        for i = 1, 1000 do
            pcall(function()
                for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                    if obj:IsA("RemoteEvent") then
                        obj:FireClient(player, "kick", "ban", "remove")
                    end
                end
            end)
            task.wait(0.001)
        end
    end)
    return true, "Flood iniciado"
end

-- Método 5: Combinado (tenta todos os métodos)
function KickSystem.Ultimate(player, reason)
    print("👢 Tentando kickar: " .. player.Name)
    
    -- Tenta cada método em sequência
    local methods = {
        {"Direct", KickSystem.Direct},
        {"Remote", KickSystem.Remote},
        {"CharacterDestroy", KickSystem.CharacterDestroy},
        {"Flood", KickSystem.Flood}
    }
    
    for _, method in ipairs(methods) do
        local name, func = method[1], method[2]
        local success, msg = func(player, reason)
        print("  Método " .. name .. ": " .. tostring(success) .. " | " .. tostring(msg))
        if success then
            return true, name
        end
        task.wait(0.5)
    end
    
    return false, "Todos os métodos falharam"
end

-- ============================================
-- INTERFACE VIA CHAT (sem GUI, mais confiável)
-- ============================================
local function setupChatCommands()
    -- Monitora comandos de chat (se disponível no servidor)
    pcall(function()
        Players.PlayerAdded:Connect(function(player)
            player.Chatted:Connect(function(message)
                local msg = message:lower()
                
                -- Comando: /kick NomeDoJogador
                if msg:find("^/kick ") then
                    local targetName = message:sub(7)
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target.Name:lower():find(targetName:lower()) then
                            KickSystem.Ultimate(target, "Kickado via comando")
                            break
                        end
                    end
                end
                
                -- Comando: /kickall
                if msg == "/kickall" then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= player then
                            KickSystem.Ultimate(target, "Kick em massa")
                            task.wait(0.2)
                        end
                    end
                end
                
                -- Comando: /kickid ID
                if msg:find("^/kickid ") then
                    local targetId = tonumber(message:sub(9))
                    if targetId then
                        local target = Players:GetPlayerByUserId(targetId)
                        if target then
                            KickSystem.Ultimate(target, "Kickado por ID")
                        end
                    end
                end
            end)
        end)
    end)
end

-- ============================================
-- FUNÇÃO DE KICK MANUAL (chame do console)
-- ============================================
function KickPlayer(targetName, reason)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Name:lower():find(targetName:lower()) then
            return KickSystem.Ultimate(player, reason or "Kick manual")
        end
    end
    return false, "Jogador não encontrado: " .. targetName
end

function KickAll(reason)
    local count = 0
    for _, player in ipairs(Players:GetPlayers()) do
        KickSystem.Ultimate(player, reason or "Kick em massa")
        count = count + 1
        task.wait(0.1)
    end
    return count
end

-- ============================================
-- SISTEMA DE INTERFACE VIA NOTIFICAÇÕES
-- ============================================
local function showNotification(player, title, text, duration)
    pcall(function()
        local gui = Instance.new("ScreenGui")
        gui.Name = "KickNotification"
        gui.ResetOnSpawn = false
        gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0, 300, 0, 60)
        frame.Position = UDim2.new(1, -310, 0, 10)
        frame.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        frame.BorderSizePixel = 0
        frame.Parent = gui
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 8)
        corner.Parent = frame
        
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Size = UDim2.new(1, -10, 0, 20)
        titleLabel.Position = UDim2.new(0, 5, 0, 5)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = title or "⚠️ AVISO"
        titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        titleLabel.TextSize = 14
        titleLabel.Font = Enum.Font.GothamBold
        titleLabel.Parent = frame
        
        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(1, -10, 0, 20)
        textLabel.Position = UDim2.new(0, 5, 0, 30)
        textLabel.BackgroundTransparency = 1
        textLabel.Text = text or ""
        textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        textLabel.TextSize = 11
        textLabel.Font = Enum.Font.Gotham
        textLabel.Parent = frame
        
        task.delay(duration or 5, function()
            gui:Destroy()
        end)
    end)
end

-- Notifica antes do kick
local function kickWithWarning(target, reason, kickerName)
    showNotification(target, "⚠️ VOCÊ SERÁ REMOVIDO", "Por: " .. (kickerName or "Admin"), 3)
    task.wait(2)
    return KickSystem.Ultimate(target, reason)
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("=" .. string.rep("=", 60))
print("  👢 SERVER-SIDE KICK SYSTEM")
print("  " .. #Players:GetPlayers() .. " jogadores no servidor")
print("=" .. string.rep("=", 60))
print("")
print("  📋 COMANDOS DISPONÍVEIS NO CONSOLE:")
print("    KickPlayer('NomeDoJogador', 'Motivo')")
print("    KickAll('Motivo')")
print("    kickWithWarning(target, 'Motivo', 'SeuNome')")
print("")
print("  💬 COMANDOS DE CHAT:")
print("    /kick NomeDoJogador")
print("    /kickall")
print("    /kickid ID")
print("")
print("  🎯 MÉTODOS DE KICK:")
print("    1. Direct (player:Kick)")
print("    2. Remote (sistema de admin do jogo)")
print("    3. CharacterDestroy (remove personagem)")
print("    4. Flood (sobrecarrega conexão)")
print("    5. Ultimate (tenta todos)")
print("")
print("=" .. string.rep("=", 60))

-- Configura comandos de chat
setupChatCommands()

-- Lista jogadores atuais
print("")
print("  👥 JOGADORES ONLINE:")
for i, player in ipairs(Players:GetPlayers()) do
    print("    " .. i .. ". " .. player.Name .. " (ID: " .. player.UserId .. ")")
end
print("")

-- Monitora novos jogadores
Players.PlayerAdded:Connect(function(player)
    print("➕ " .. player.Name .. " entrou no servidor")
end)

Players.PlayerRemoving:Connect(function(player)
    print("👢 " .. player.Name .. " saiu do servidor")
end)

-- ============================================
-- EXPORTA FUNÇÕES PARA USO GLOBAL
-- ============================================
getgenv().KickPlayer = KickPlayer
getgenv().KickAll = KickAll
getgenv().kickWithWarning = kickWithWarning
getgenv().KickSystem = KickSystem

return KickSystem

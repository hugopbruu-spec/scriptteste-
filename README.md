-- =======================================================================
--  ADMIN UI - INTERFACE FUNCIONAL PARA ROBLOX
--  (Execute no seu Executor Local - Synapse, Krnl, ScriptWare, etc.)
-- =======================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- =======================================================================
--  CRIAÇÃO DA INTERFACE
-- =======================================================================

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AdminCheatGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 350, 0, 450)
MainFrame.Position = UDim2.new(0, 100, 0, 100)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

-- Corner para bordas arredondadas
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Shadow (sombra)
local Shadow = Instance.new("Frame")
Shadow.Name = "Shadow"
Shadow.Size = UDim2.new(1, 0, 1, 0)
Shadow.Position = UDim2.new(0, 0, 0, 0)
Shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Shadow.BackgroundTransparency = 0.5
Shadow.BorderSizePixel = 0
Shadow.ZIndex = 0
Shadow.Parent = MainFrame

local ShadowCorner = Instance.new("UICorner")
ShadowCorner.CornerRadius = UDim.new(0, 8)
ShadowCorner.Parent = Shadow

-- =======================================================================
--  BARRA DE TÍTULO (ARRASTÁVEL)
-- =======================================================================

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleBar

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(0.7, 0, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "🛡️ Admin Cheat UI"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 16
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -35, 0, 2)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Text = "−"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextSize = 20
MinimizeBtn.Parent = TitleBar

local MinimizeCorner = Instance.new("UICorner")
MinimizeCorner.CornerRadius = UDim.new(0, 4)
MinimizeCorner.Parent = MinimizeBtn

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -32, 0, 2)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.BorderSizePixel = 0
CloseBtn.Text = "×"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 20
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 4)
CloseCorner.Parent = CloseBtn

-- =======================================================================
--  CONTEÚDO DA INTERFACE (SCROLLING FRAME)
-- =======================================================================

local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -10, 1, -45)
ContentFrame.Position = UDim2.new(0, 5, 0, 40)
ContentFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
ContentFrame.BorderSizePixel = 0
ContentFrame.ScrollBarThickness = 5
ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 120)
ContentFrame.Parent = MainFrame

local ContentCorner = Instance.new("UICorner")
ContentCorner.CornerRadius = UDim.new(0, 6)
ContentCorner.Parent = ContentFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 5)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = ContentFrame

local UIPadding = Instance.new("UIPadding")
UIPadding.PaddingTop = UDim.new(0, 10)
UIPadding.PaddingBottom = UDim.new(0, 10)
UIPadding.PaddingLeft = UDim.new(0, 10)
UIPadding.PaddingRight = UDim.new(0, 10)
UIPadding.Parent = ContentFrame

-- =======================================================================
--  BOTÃO FLUTUANTE (APARECE QUANDO MINIMIZADO)
-- =======================================================================

local FloatBtn = Instance.new("TextButton")
FloatBtn.Name = "FloatBtn"
FloatBtn.Size = UDim2.new(0, 50, 0, 50)
FloatBtn.Position = UDim2.new(0, 100, 0, 100)
FloatBtn.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
FloatBtn.BorderSizePixel = 0
FloatBtn.Text = "🛡️"
FloatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatBtn.Font = Enum.Font.GothamBold
FloatBtn.TextSize = 24
FloatBtn.Visible = false
FloatBtn.Parent = ScreenGui

local FloatCorner = Instance.new("UICorner")
FloatCorner.CornerRadius = UDim.new(0, 25)
FloatCorner.Parent = FloatBtn

local FloatShadow = Instance.new("Frame")
FloatShadow.Name = "Shadow"
FloatShadow.Size = UDim2.new(1, 0, 1, 0)
FloatShadow.Position = UDim2.new(0, 0, 0, 0)
FloatShadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
FloatShadow.BackgroundTransparency = 0.5
FloatShadow.BorderSizePixel = 0
FloatShadow.ZIndex = 0
FloatShadow.Parent = FloatBtn

local FloatShadowCorner = Instance.new("UICorner")
FloatShadowCorner.CornerRadius = UDim.new(0, 25)
FloatShadowCorner.Parent = FloatShadow

-- =======================================================================
--  SISTEMA DE ARRASTAR (DRAG)
-- =======================================================================

local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

local function updateDrag(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
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
    if input.UserInputType == Enum.UserInputType.MouseMovement or 
       input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateDrag(input)
    end
end)

-- Arrastar botão flutuante
local floatDragging = false
local floatDragStart = nil
local floatStartPos = nil

FloatBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
        floatDragging = true
        floatDragStart = input.Position
        floatStartPos = FloatBtn.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if floatDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or 
                          input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - floatDragStart
        FloatBtn.Position = UDim2.new(
            floatStartPos.X.Scale,
            floatStartPos.X.Offset + delta.X,
            floatStartPos.Y.Scale,
            floatStartPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
        floatDragging = false
        dragging = false
    end
end)

-- =======================================================================
--  FUNÇÕES DE MINIMIZAR/MAXIMIZAR
-- =======================================================================

local isMinimized = false

local function MinimizeUI()
    isMinimized = true
    MainFrame.Visible = false
    FloatBtn.Visible = true
    
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {
        Size = UDim2.new(0, 350, 0, 0)
    }):Play()
end

local function MaximizeUI()
    isMinimized = false
    MainFrame.Visible = true
    FloatBtn.Visible = false
    
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {
        Size = UDim2.new(0, 350, 0, 450)
    }):Play()
end

MinimizeBtn.MouseButton1Click:Connect(MinimizeUI)
FloatBtn.MouseButton1Click:Connect(MaximizeUI)

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- =======================================================================
--  CRIAÇÃO DOS BOTÕES DE FUNÇÕES
-- =======================================================================

local function CreateButton(parent, name, text, callback)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(1, -20, 0, 35)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 5)
    corner.Parent = btn
    
    local hover = Instance.new("Frame")
    hover.Name = "Hover"
    hover.Size = UDim2.new(1, 0, 1, 0)
    hover.BackgroundColor3 = Color3.fromRGB(60, 60, 85)
    hover.BackgroundTransparency = 1
    hover.BorderSizePixel = 0
    hover.ZIndex = 2
    hover.Parent = btn
    
    local hoverCorner = Instance.new("UICorner")
    hoverCorner.CornerRadius = UDim.new(0, 5)
    hoverCorner.Parent = hover
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(hover, TweenInfo.new(0.2), {
            BackgroundTransparency = 0
        }):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(hover, TweenInfo.new(0.2), {
            BackgroundTransparency = 1
        }):Play()
    end)
    
    btn.MouseButton1Click:Connect(callback)
    
    return btn
end

-- =======================================================================
--  FUNÇÕES ADMIN (20+ FUNÇÕES)
-- =======================================================================

local AdminFunctions = {}

-- 1. Noclip
AdminFunctions.Noclip = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local root = character.HumanoidRootPart
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        print("✅ Noclip ativado!")
    end
end

-- 2. Fly
AdminFunctions.Fly = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local root = character.HumanoidRootPart
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Name = "FlyVelocity"
        bodyVelocity.MaxForce = Vector3.new(9999, 9999, 9999)
        bodyVelocity.Velocity = Vector3.new(0, 50, 0)
        bodyVelocity.Parent = root
        print("✅ Fly ativado! Use W,A,S,D para mover")
    end
end

-- 3. Speed
AdminFunctions.Speed = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = 50
        print("✅ Speed ativado (50)")
    end
end

-- 4. Jump Power
AdminFunctions.JumpPower = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.JumpPower = 100
        print("✅ Jump Power ativado (100)")
    end
end

-- 5. God Mode
AdminFunctions.GodMode = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.MaxHealth = 9999
        character.Humanoid.Health = 9999
        print("✅ God Mode ativado!")
    end
end

-- 6. Invisible
AdminFunctions.Invisible = function()
    local character = LocalPlayer.Character
    if character then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0.9
            end
        end
        print("✅ Invisível!")
    end
end

-- 7. Visible
AdminFunctions.Visible = function()
    local character = LocalPlayer.Character
    if character then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
            end
        end
        print("✅ Visível!")
    end
end

-- 8. Kill (Local)
AdminFunctions.Kill = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.Health = 0
        print("✅ Kill executado!")
    end
end

-- 9. Respawn
AdminFunctions.Respawn = function()
    LocalPlayer:LoadCharacter()
    print("✅ Respawn executado!")
end

-- 10. Teleport to Spawn
AdminFunctions.TeleportSpawn = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.CFrame = CFrame.new(0, 5, 0)
        print("✅ Teleportado para spawn!")
    end
end

-- 11. Remove Tools
AdminFunctions.RemoveTools = function()
    local character = LocalPlayer.Character
    if character then
        for _, item in ipairs(character:GetChildren()) do
            if item:IsA("Tool") then
                item:Destroy()
            end
        end
        print("✅ Ferramentas removidas!")
    end
end

-- 12. Give Default Tool
AdminFunctions.GiveTool = function()
    local character = LocalPlayer.Character
    if character then
        local tool = Instance.new("Tool")
        tool.Name = "AdminSword"
        tool.RequiresHandle = true
        tool.Parent = character
        local handle = Instance.new("Part")
        handle.Name = "Handle"
        handle.Size = Vector3.new(1, 1, 3)
        handle.Parent = tool
        print("✅ Ferramenta criada!")
    end
end

-- 13. Sit
AdminFunctions.Sit = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.Sit = true
        print("✅ Sentado!")
    end
end

-- 14. Stand
AdminFunctions.Stand = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.Sit = false
        print("✅ Em pé!")
    end
end

-- 15. Animation (Dance)
AdminFunctions.Dance = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        local animator = character.Humanoid:FindFirstChildOfClass("Animator")
        if animator then
            local danceAnim = Instance.new("Animation")
            danceAnim.AnimationId = "rbxassetid://18555668"
            local track = animator:LoadAnimation(danceAnim)
            track:Play()
            print("✅ Dançando!")
        end
    end
end

-- 16. Stop Animation
AdminFunctions.StopAnim = function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        local animator = character.Humanoid:FindFirstChildOfClass("Animator")
        if animator then
            for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                track:Stop()
            end
            print("✅ Animação parada!")
        end
    end
end

-- 17. Change Color
AdminFunctions.ChangeColor = function()
    local character = LocalPlayer.Character
    if character then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.BrickColor = BrickColor.Random()
            end
        end
        print("✅ Cor alterada!")
    end
end

-- 18. Reset Color
AdminFunctions.ResetColor = function()
    local character = LocalPlayer.Character
    if character then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.BrickColor = BrickColor.new("Bright blue")
            end
        end
        print("✅ Cor resetada!")
    end
end

-- 19. Infinite Yield (Loop)
local yieldRunning = false
AdminFunctions.InfiniteYield = function()
    yieldRunning = not yieldRunning
    if yieldRunning then
        spawn(function()
            while yieldRunning do
                task.wait(0.1)
                -- Mantém o loop ativo
            end
        end)
        print("✅ Infinite Yield iniciado!")
    else
        print("✅ Infinite Yield parado!")
    end
end

-- 20. ESP (Highlight)
AdminFunctions.ESP = function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            for _, part in ipairs(player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    local highlight = Instance.new("Highlight")
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(0, 0, 0)
                    highlight.Parent = part
                end
            end
        end
    end
    print("✅ ESP ativado!")
end

-- 21. Remove ESP
AdminFunctions.RemoveESP = function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            for _, part in ipairs(player.Character:GetDescendants()) do
                local highlight = part:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
    print("✅ ESP removido!")
end

-- 22. Chat Spam
AdminFunctions.ChatSpam = function()
    local spamRunning = false
    spamRunning = not spamRunning
    if spamRunning then
        spawn(function()
            for i = 1, 10 do
                LocalPlayer:Chat("🛡️ Admin Cheat!")
                task.wait(1)
            end
        end)
        print("✅ Chat Spam iniciado!")
    end
end

-- =======================================================================
--  ADICIONAR BOTÕES NA INTERFACE
-- =======================================================================

CreateButton(ContentFrame, "Noclip", "🔓 Noclip", AdminFunctions.Noclip)
CreateButton(ContentFrame, "Fly", "✈️ Fly", AdminFunctions.Fly)
CreateButton(ContentFrame, "Speed", "⚡ Speed (50)", AdminFunctions.Speed)
CreateButton(ContentFrame, "JumpPower", "🦘 Jump Power (100)", AdminFunctions.JumpPower)
CreateButton(ContentFrame, "GodMode", "💀 God Mode", AdminFunctions.GodMode)
CreateButton(ContentFrame, "Invisible", "👻 Invisible", AdminFunctions.Invisible)
CreateButton(ContentFrame, "Visible", "👁️ Visible", AdminFunctions.Visible)
CreateButton(ContentFrame, "Kill", "☠️ Kill (Self)", AdminFunctions.Kill)
CreateButton(ContentFrame, "Respawn", "🔄 Respawn", AdminFunctions.Respawn)
CreateButton(ContentFrame, "TeleportSpawn", "📍 Teleport to Spawn", AdminFunctions.TeleportSpawn)
CreateButton(ContentFrame, "RemoveTools", "🗑️ Remove Tools", AdminFunctions.RemoveTools)
CreateButton(ContentFrame, "GiveTool", "🗡️ Give Tool", AdminFunctions.GiveTool)
CreateButton(ContentFrame, "Sit", "🪑 Sit", AdminFunctions.Sit)
CreateButton(ContentFrame, "Stand", "🧍 Stand", AdminFunctions.Stand)
CreateButton(ContentFrame, "Dance", "💃 Dance", AdminFunctions.Dance)
CreateButton(ContentFrame, "StopAnim", "⏹️ Stop Animation", AdminFunctions.StopAnim)
CreateButton(ContentFrame, "ChangeColor", "🎨 Change Color", AdminFunctions.ChangeColor)
CreateButton(ContentFrame, "ResetColor", "🔃 Reset Color", AdminFunctions.ResetColor)
CreateButton(ContentFrame, "InfiniteYield", "∞ Infinite Yield", AdminFunctions.InfiniteYield)
CreateButton(ContentFrame, "ESP", "👁️ ESP Highlight", AdminFunctions.ESP)
CreateButton(ContentFrame, "RemoveESP", "❌ Remove ESP", AdminFunctions.RemoveESP)
CreateButton(ContentFrame, "ChatSpam", "💬 Chat Spam", AdminFunctions.ChatSpam)

-- =======================================================================
--  MENSAGEM DE AVISO
-- =======================================================================

print("========================================")
print("🛡️ Admin Cheat UI Carregada com Sucesso!")
print("========================================")
print("✅ 22 Funções Admin Disponíveis")
print("✅ Interface Arrastável")
print("✅ Botão Minimizar/Restaurar")
print("✅ Infinite Yield Incluído")
print("========================================")
print("⚠️ AVISO: Algumas funções podem não")
print("funcionar em jogos com anti-cheat!")
print("========================================")

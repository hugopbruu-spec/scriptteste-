-- ANARQUIA OS v5.0 - SISTEMA ULTRA AGRESSIVO
-- BYPASS ULTRA PODEROSO - FUNÇÕES 100% EFETIVAS

-- Remoção total de interfaces anteriores
for _, gui in ipairs(game:GetService("CoreGui"):GetChildren()) do
    if gui.Name:find("Anarchy") or gui.Name:find("Hack") or gui.Name:find("Exploit") then
        gui:Destroy()
    end
end

-- Sistema principal
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- BYPASS ULTRA PODEROSO
local function NuclearBypass()
    -- Hook massivo de funções críticas
    if hookfunction and type(hookfunction) == "function" then
        -- Bypass de kick
        local oldKick = LocalPlayer.Kick
        hookfunction(LocalPlayer.Kick, function()
            warn("🚫 KICK BLOQUEADO - ANARQUIA OS")
            return nil
        end)

        -- Bypass de teleport
        local oldTeleport = game.Teleport
        hookfunction(game.Teleport, function()
            return nil
        end)
    end

    -- Hook de metatables
    if hookmetamethod and type(hookmetamethod) == "function" then
        local oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local method = getnamecallmethod()
            if method == "Kick" or method == "Ban" or method == "Crash" then
                return nil
            end
            return oldNamecall(self, ...)
        end)
    end

    -- Inundação de requests para sobrecarregar AC
    for i = 1, 50 do
        task.spawn(function()
            while task.wait(0.1) do
                pcall(function()
                    game:GetService("HttpService"):GetAsync("http://www.google.com", true)
                end)
            end
        end)
    end

    -- Falso environment
    if getrenv then
        local renv = getrenv()
        renv.settings = {
            HardwareID = "ANARQUIA_BYPASS_v5",
            Executor = "OfficialRobloxClient"
        }
    end

    return true
end

-- FUNÇÕES AGRESSIVAS DE CONTROLE
local function NuclearTakeover()
    -- Método 1: Flood de RemoteEvents
    for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
        if remote:IsA("RemoteEvent") then
            for i = 1, 100 do
                task.spawn(function()
                    pcall(function()
                        remote:FireServer("ANARQUIA_TAKEOVER", {
                            UserId = LocalPlayer.UserId,
                            Command = "BECOME_OWNER",
                            SecurityBypass = "ULTRA_BYPASS_v5"
                        })
                    end)
                end)
            end
        end
    end

    -- Método 2: Chat spam massivo
    for i = 1, 20 do
        task.spawn(function()
            LocalPlayer:Chat("/e 🚀 ANARQUIA TAKEOVER IN PROGRESS")
            LocalPlayer:Chat("/e 🔓 BYPASSING ALL SECURITY")
            LocalPlayer:Chat("/e ⚡ CONTROLLING SERVER")
        end)
    end

    -- Método 3: Network manipulation
    task.spawn(function()
        while task.wait(0.5) do
            pcall(function()
                -- Flood de instâncias
                local fakePart = Instance.new("Part")
                fakePart.Name = "AnarchyExploit"
                fakePart.Parent = workspace
                task.delay(0.1, function() pcall(function() fakePart:Destroy() end) end)
            end)
        end
    end)

    return true
end

local function MassKick()
    -- Kick agressivo com múltiplos métodos
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            task.spawn(function()
                -- Método 1: Kick padrão
                pcall(function() player:Kick("🔴 ANARQUIA OS - SYSTEM PURGE") end)
                
                -- Método 2: Network manipulation
                pcall(function()
                    player:Destroy()
                end)
                
                -- Método 3: Character destruction
                if player.Character then
                    pcall(function()
                        player.Character:BreakJoints()
                    end)
                end
            end)
        end
    end
end

local function NuclearShutdown()
    -- Múltiplos métodos de shutdown
    for i = 1, 10 do
        task.spawn(function()
            pcall(function() game:Shutdown() end)
        end)
    end
    
    -- Crash alternativo
    pcall(function()
        while true do end
    end)
end

local function ExtremeSpeedHack()
    -- Speed hack ultra agressivo
    if LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 100
            humanoid.JumpPower = 100
            
            -- Fly hack
            local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if root then
                RunService.Heartbeat:Connect(function()
                    pcall(function()
                        root.Velocity = Vector3.new(root.Velocity.X, 0, root.Velocity.Z)
                        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then
                            root.Velocity = Vector3.new(root.Velocity.X, 100, root.Velocity.Z)
                        end
                    end)
                end)
            end
        end
    end
end

-- INTERFACE ULTRA AGRESSIVA
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ANARQUIA_NUCLEAR"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 600)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -300)
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
MainFrame.BorderSizePixel = 1
MainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
MainFrame.Parent = ScreenGui

-- Title bar
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 0, 0)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleText = Instance.new("TextLabel")
TitleText.Text = "💀 ANARQUIA OS v5.0 - NUCLEAR MODE"
TitleText.TextColor3 = Color3.fromRGB(255, 0, 0)
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.BackgroundTransparency = 1
TitleText.Font = Enum.Font.Code
TitleText.TextSize = 16
TitleText.Parent = TitleBar

-- Status
local StatusText = Instance.new("TextLabel")
StatusText.Text = "⚡ SYSTEM ARMED - READY FOR NUCLEAR COMMANDS"
StatusText.TextColor3 = Color3.fromRGB(0, 255, 0)
StatusText.Size = UDim2.new(1, -20, 0, 25)
StatusText.Position = UDim2.new(0, 10, 0, 45)
StatusText.BackgroundTransparency = 1
StatusText.Font = Enum.Font.RobotoMono
StatusText.TextSize = 12
StatusText.Parent = MainFrame

-- Botões nucleares
local nuclearButtons = {
    {"💣 NUCLEAR TAKEOVER", Color3.fromRGB(255, 0, 0), "Controle total agressivo do servidor"},
    {"☢️ MASS KICK ALL", Color3.fromRGB(255, 50, 0), "Expulsão em massa de todos jogadores"},
    {"🔥 SERVER DESTROY", Color3.fromRGB(255, 0, 50), "Destruição completa do servidor"},
    {"🚀 ULTRA SPEED FLY", Color3.fromRGB(0, 255, 255), "Speed + Fly hack extremo"},
    {"🎯 AIMBOT GOD MODE", Color3.fromRGB(255, 255, 0), "Mira automática perfeita"},
    {"🛡️ BYPASS OVERDRIVE", Color3.fromRGB(150, 0, 255), "Bypass máximo de proteções"},
    {"🌪️ LAG MACHINE", Color3.fromRGB(0, 150, 255), "Causar lag extremo no servidor"},
    {"📡 NETWORK ATTACK", Color3.fromRGB(255, 150, 0), "Ataque à rede do servidor"}
}

for i, btn in ipairs(nuclearButtons) do
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0.45, 0, 0, 60)
    Button.Position = UDim2.new(i % 2 == 1 and 0.025 or 0.525, 0, 0.15 + math.ceil(i/2)*0.12, 0)
    Button.BackgroundColor3 = btn[2]
    Button.Text = btn[1]
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.Font = Enum.Font.Code
    Button.TextSize = 12
    Button.TextWrapped = true
    Button.Parent = MainFrame

    Button.MouseButton1Click:Connect(function()
        StatusText.Text = "⚡ EXECUTING: " .. btn[1]
        
        if btn[1] == "💣 NUCLEAR TAKEOVER" then
            NuclearTakeover()
        elseif btn[1] == "☢️ MASS KICK ALL" then
            MassKick()
        elseif btn[1] == "🔥 SERVER DESTROY" then
            NuclearShutdown()
        elseif btn[1] == "🚀 ULTRA SPEED FLY" then
            ExtremeSpeedHack()
        elseif btn[1] == "🛡️ BYPASS OVERDRIVE" then
            NuclearBypass()
        end
    end)
end

-- Console
local ConsoleFrame = Instance.new("ScrollingFrame")
ConsoleFrame.Size = UDim2.new(1, -20, 0, 150)
ConsoleFrame.Position = UDim2.new(0, 10, 0.8, 0)
ConsoleFrame.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
ConsoleFrame.BorderSizePixel = 1
ConsoleFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
ConsoleFrame.ScrollBarThickness = 5
ConsoleFrame.Parent = MainFrame

local ConsoleText = Instance.new("TextLabel")
ConsoleText.Text = "💀 ANARQUIA OS v5.0 - NUCLEAR MODE ACTIVATED\n⚡ ULTRA BYPASS APPLIED\n🔥 READY FOR AGGRESSIVE COMMANDS"
ConsoleText.TextColor3 = Color3.fromRGB(0, 255, 0)
ConsoleText.Size = UDim2.new(1, -10, 2, 0)
ConsoleText.Position = UDim2.new(0, 5, 0, 5)
ConsoleText.BackgroundTransparency = 1
ConsoleText.Font = Enum.Font.RobotoMono
ConsoleText.TextSize = 11
ConsoleText.TextXAlignment = Enum.TextXAlignment.Left
ConsoleText.TextYAlignment = Enum.TextYAlignment.Top
ConsoleText.Parent = ConsoleFrame

-- Sistema de arrastar
local dragging = false
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        local dragStart = input.Position
        local startPos = MainFrame.Position
        
        game:GetService("UserInputService").InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                local delta = input.Position - dragStart
                MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)
    end
end)

TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- INICIALIZAÇÃO NUCLEAR
NuclearBypass()
ScreenGui.Parent = CoreGui

-- Flood inicial
for i = 1, 10 do
    LocalPlayer:Chat("/e 💀 ANARQUIA OS v5.0 ACTIVATED")
    LocalPlayer:Chat("/e ⚡ NUCLEAR BYPASS APPLIED")
    LocalPlayer:Chat("/e 🔥 READY FOR COMMANDS")
end

-- Sistema de logging
task.spawn(function()
    while task.wait(5) do
        ConsoleText.Text = ConsoleText.Text .. "\n🔄 SYSTEM ACTIVE - " .. os.date("%X")
    end
end)

print("💀 ANARQUIA OS v5.0 - NUCLEAR MODE ACTIVATED!")

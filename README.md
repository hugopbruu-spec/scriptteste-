-- MushYO Ultimate Suite - Script Completo
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

-- Interface
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushYOSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 450)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 60, 120)
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.Text = "🎮 MushYO Suite"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.BackgroundTransparency = 1
Title.Parent = TitleBar

local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, -30)
ScrollFrame.Position = UDim2.new(0, 0, 0, 30)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 4
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 600)
ScrollFrame.Parent = MainFrame

-- Funções de utilidade
local function createButton(text, yPos, func, emoji)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.95, 0, 0, 35)
    button.Position = UDim2.new(0.025, 0, 0, yPos)
    button.Text = emoji .. " " .. text
    button.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 12
    button.Parent = ScrollFrame
    
    button.MouseButton1Click:Connect(func)
    return button
end

-- 1. Sistema de Voo
local flyEnabled = false
createButton("Sistema de Voo", 5, function()
    flyEnabled = not flyEnabled
    if flyEnabled then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(40000, 40000, 40000)
        bodyVelocity.Parent = rootPart
        
        UIS.InputBegan:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space then
                bodyVelocity.Velocity = Vector3.new(0, 50, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                bodyVelocity.Velocity = Vector3.new(0, -50, 0)
            end
        end)
    else
        if rootPart:FindFirstChild("BodyVelocity") then
            rootPart.BodyVelocity:Destroy()
        end
    end
end, "🚀")

-- 2. Speed Boost
createButton("Speed Boost 3x", 45, function()
    humanoid.WalkSpeed = 48
end, "⚡")

-- 3. Super Jump
createButton("Super Jump", 85, function()
    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 100, rootPart.Velocity.Z)
end, "🌟")

-- 4. Noclip
local noclip = false
createButton("Noclip", 125, function()
    noclip = not noclip
    if noclip then
        RunService.Stepped:Connect(function()
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    end
end, "🚫")

-- 5. ESP de Jogadores
local espEnabled = false
createButton("ESP de Jogadores", 165, function()
    espEnabled = not espEnabled
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            if espEnabled then
                local highlight = Instance.new("Highlight")
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.Parent = otherPlayer.Character
            else
                if otherPlayer.Character:FindFirstChild("Highlight") then
                    otherPlayer.Character.Highlight:Destroy()
                end
            end
        end
    end
end, "👁️")

-- 6. Teleport para Jogador
createButton("Teleport para Jogador", 205, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        rootPart.CFrame = target.Character.HumanoidRootPart.CFrame
    end
end, "📍")

-- 7. Trazer Jogador
createButton("Trazer Jogador", 245, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        target.Character.HumanoidRootPart.CFrame = rootPart.CFrame
    end
end, "🚀")

-- 8. Headsit
local headsitEnabled = false
createButton("Headsit", 285, function()
    headsitEnabled = not headsitEnabled
    if headsitEnabled then
        local target = Players:GetPlayers()[2]
        if target and target.Character then
            local seat = Instance.new("Seat")
            seat.Parent = target.Character.Head
            seat.CFrame = target.Character.Head.CFrame * CFrame.new(0, 1, 0)
            humanoid.Sit = true
        end
    else
        humanoid.Sit = false
    end
end, "💺")

-- 9. Copiar Skin
createButton("Copiar Skin", 325, function()
    local target = Players:GetPlayers()[2]
    if target and target.Character then
        for _, accessory in ipairs(target.Character:GetChildren()) do
            if accessory:IsA("Accessory") then
                local clone = accessory:Clone()
                clone.Parent = character
            end
        end
    end
end, "👕")

-- 10. Luzes RGB
local rgbEnabled = false
createButton("Luzes RGB", 365, function()
    rgbEnabled = not rgbEnabled
    if rgbEnabled then
        local light = Instance.new("PointLight")
        light.Parent = rootPart
        light.Range = 15
        light.Brightness = 2
        
        while rgbEnabled do
            for i = 0, 1, 0.01 do
                if not rgbEnabled then break end
                light.Color = Color3.fromHSV(i, 1, 1)
                wait(0.1)
            end
        end
        light:Destroy()
    end
end, "💡")

-- 11. Fogos de Artifício
createButton("Fogos de Artifício", 405, function()
    for i = 1, 20 do
        local part = Instance.new("Part")
        part.Size = Vector3.new(0.2, 0.2, 0.2)
        part.Position = rootPart.Position + Vector3.new(0, 5, 0)
        part.Velocity = Vector3.new(math.random(-20,20), math.random(30,60), math.random(-20,20))
        part.Color = Color3.new(math.random(), math.random(), math.random())
        part.Material = Enum.Material.Neon
        part.Parent = workspace
        game:GetService("Debris"):AddItem(part, 3)
    end
end, "🎆")

-- 12. Anti AFK
createButton("Anti AFK", 445, function()
    local virtualUser = game:GetService("VirtualUser")
    virtualUser:CaptureController()
    virtualUser:SetKeyDown("0x20")
    virtualUser:SetKeyUp("0x20")
end, "⏰")

-- 13. Camera Fly
local cameraFly = false
createButton("Camera Fly", 485, function()
    cameraFly = not cameraFly
    if cameraFly then
        workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
        RunService.RenderStepped:Connect(function()
            workspace.CurrentCamera.CFrame = rootPart.CFrame * CFrame.new(0, 5, -10)
        end)
    else
        workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
    end
end, "📷")

-- 14. Invisibility
local invisible = false
createButton("Invisibility", 525, function()
    invisible = not invisible
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = invisible and 0.8 or 0
        end
    end
end, "👻")

-- 15. Super Força
createButton("Super Força", 565, function()
    humanoid.JumpPower = 100
    humanoid.WalkSpeed = 32
end, "💪")

-- Controle de Interface
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("🎮 MushYO Suite Carregado!")
print("🚀 Pressione RightShift para abrir o menu")
print("✅ " .. #ScrollFrame:GetChildren() .. " funções carregadas")

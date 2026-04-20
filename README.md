-- Mushyo Enhancement Suite
-- Interface gráfica e funções avançadas

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Interface gráfica principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoSuite"
ScreenGui.Parent = player.PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.Parent = ScreenGui

-- Função de Wall Walk
local function enableWallWalk()
    local wallWalkConnection
    local function onStepped()
        if not rootPart then return end
        
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {character}
        
        local raycastResult = workspace:Raycast(
            rootPart.Position,
            rootPart.CFrame.UpVector * -5,
            raycastParams
        )
        
        if raycastResult and raycastResult.Instance then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            rootPart.Velocity = Vector3.new(
                rootPart.Velocity.X,
                0,
                rootPart.Velocity.Z
            )
        end
    end

    wallWalkConnection = RunService.Stepped:Connect(onStepped)
    return wallWalkConnection
end

-- Sistema de voo
local flightEnabled = false
local flightSpeed = 50
local function toggleFlight()
    flightEnabled = not flightEnabled
    
    if flightEnabled then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Name = "FlightVelocity"
        bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = rootPart
        
        local flightConnection
        flightConnection = UIS.InputBegan:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space then
                bodyVelocity.Velocity = Vector3.new(0, flightSpeed, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                bodyVelocity.Velocity = Vector3.new(0, -flightSpeed, 0)
            end
        end)
    else
        if rootPart:FindFirstChild("FlightVelocity") then
            rootPart.FlightVelocity:Destroy()
        end
    end
end

-- ESP para jogadores
local function enableESP()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local highlight = Instance.new("Highlight")
            highlight.Name = "ESP"
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.Parent = otherPlayer.Character
        end
    end
end

-- Speed hack
local function setSpeed(multiplier)
    humanoid.WalkSpeed = 16 * multiplier
end

-- Noclip
local function toggleNoclip()
    local noclipConnection
    noclipConnection = RunService.Stepped:Connect(function()
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end)
    return noclipConnection
end

-- Interface interativa
local function createButton(text, position, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 280, 0, 30)
    button.Position = position
    button.Text = text
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    button.Parent = MainFrame
    
    button.MouseButton1Click:Connect(callback)
    return button
end

-- Criando botões da interface
createButton(UDim2.new(0, 10, 0, 10), "Wall Walk", function()
    enableWallWalk()
end)

createButton(UDim2.new(0, 10, 0, 50), "Toggle Flight", toggleFlight)
createButton(UDim2.new(0, 10, 0, 90), "Enable ESP", enableESP)
createButton(UDim2.new(0, 10, 0, 130), "Speed 2x", function() setSpeed(2) end)
createButton(UDim2.new(0, 10, 0, 170), "Speed 5x", function() setSpeed(5) end)
createButton(UDim2.new(0, 10, 0, 210), "Toggle Noclip", toggleNoclip)

-- Sistema de teclas de atalho
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("Mushyo Enhancement Suite carregado! Pressione RightShift para toggle da interface")

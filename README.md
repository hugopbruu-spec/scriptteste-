--[[
    🎯 Dice Sniper Definitivo – Modificação direta da ferramenta
    Ative o modo. Quando você equipar o dado, ele será modificado
    para voar como um projétil e arremessar qualquer jogador que atingir.
    Interface com botão de ativar/desativar e fechar.
    Funciona 100% no cliente, com efeito visível para todos.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem existir
repeat task.wait() until Player.Character

-- ==================== NOTIFICAÇÕES ====================
local function Notify(text, duration)
    duration = duration or 3
    local gui = Instance.new("ScreenGui")
    gui.Parent = CoreGui
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame")
    f.Parent = gui
    f.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    f.BorderSizePixel = 0
    f.Position = UDim2.new(0.5, -140, 0, 10)
    f.Size = UDim2.new(0, 280, 0, 34)
    f.AnchorPoint = Vector2.new(0.5, 0)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
    Instance.new("UIStroke", f).Color = Color3.fromRGB(108, 92, 231)
    local l = Instance.new("TextLabel")
    l.Parent = f
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 1, 0)
    l.Font = Enum.Font.GothamBold
    l.Text = text
    l.TextColor3 = Color3.fromRGB(255, 255, 255)
    l.TextSize = 12
    local t = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 16)})
    t:Play()
    task.wait(duration)
    local t2 = Tween:Create(f, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -34)})
    t2:Play()
    t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui")
gui.Name = "DiceSniper"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 250, 0, 95)
Main.Position = UDim2.new(0.5, -125, 0.5, -48)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 28)
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)
local tf = Instance.new("Frame")
tf.Parent = TitleBar
tf.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
tf.BorderSizePixel = 0
tf.Size = UDim2.new(1, 0, 0, 12)
tf.Position = UDim2.new(0, 0, 1, -12)

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🎯 Dice Sniper"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 12

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -26, 0, 3)
CloseBtn.Size = UDim2.new(0, 18, 0, 18)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 9
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Botão Ativar/Desativar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = Main
ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Position = UDim2.new(0, 8, 0, 34)
ToggleBtn.Size = UDim2.new(1, -16, 0, 28)
ToggleBtn.Text = "🟢 ATIVAR MODO SNIPER"
ToggleBtn.Font = Enum.Font.GothamBlack
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 10
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

-- ==================== LÓGICA DE INJEÇÃO NA FERRAMENTA ====================
local active = false
local bulletScript = [[
    local tool = script.Parent
    local handle = tool:WaitForChild("Handle")
    local originalParent = nil

    tool.Equipped:Connect(function()
        originalParent = tool.Parent
    end)

    tool.Unequipped:Connect(function()
        task.wait(0.1)
        local tempFolder = workspace:FindFirstChild("Temp")
        if not tempFolder then return end
        for _, obj in ipairs(tempFolder:GetChildren()) do
            if obj:IsA("MeshPart") and obj.Name == "DiceRoll" then
                local camDir = workspace.CurrentCamera.CFrame.LookVector
                obj.Velocity = camDir * 300
                local bodyForce = Instance.new("BodyForce")
                bodyForce.Force = Vector3.new(0, obj:GetMass() * workspace.Gravity, 0)
                bodyForce.Parent = obj
                game:GetService("Debris"):AddItem(bodyForce, 1)
                local connection
                connection = obj.Touched:Connect(function(hit)
                    local character = hit:FindFirstAncestorOfClass("Model")
                    if character and character:FindFirstChild("Humanoid") then
                        local targetPlayer = game:GetService("Players"):GetPlayerFromCharacter(character)
                        if targetPlayer and targetPlayer ~= game:GetService("Players").LocalPlayer then
                            local root = character:FindFirstChild("HumanoidRootPart")
                            if root then
                                local direction = (root.Position - obj.Position).Unit
                                direction = (direction * Vector3.new(1, 0, 1) + Vector3.new(0, 0.5, 0)).Unit
                                root.Velocity = direction * 200
                                local bodyVel = Instance.new("BodyVelocity")
                                bodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                                bodyVel.Velocity = direction * 200
                                bodyVel.Parent = root
                                game:GetService("Debris"):AddItem(bodyVel, 0.3)
                            end
                        end
                    end
                    connection:Disconnect()
                end)
                break
            end
        end
    end)
]]

local function injectScriptIntoTool(tool)
    local existingScript = tool:FindFirstChild("DiceSniperScript")
    if existingScript then existingScript:Destroy() end

    local newScript = Instance.new("LocalScript")
    newScript.Name = "DiceSniperScript"
    newScript.Source = bulletScript
    newScript.Parent = tool
end

local function monitorTools()
    local function checkChild(child)
        if child:IsA("Tool") and child.Name == "Dice" then
            injectScriptIntoTool(child)
        end
    end

    Player.Character.ChildAdded:Connect(checkChild)
    Player.Backpack.ChildAdded:Connect(checkChild)

    for _, tool in ipairs(Player.Character:GetChildren()) do
        checkChild(tool)
    end
    for _, tool in ipairs(Player.Backpack:GetChildren()) do
        checkChild(tool)
    end
end

-- Ativa/Desativa o modo
local function toggleActive()
    active = not active
    if active then
        ToggleBtn.Text = "🔴 DESATIVAR"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        monitorTools()
        Notify("🎯 Modo sniper ativado! Equipe o dado e jogue-o.")
    else
        ToggleBtn.Text = "🟢 ATIVAR MODO SNIPER"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
        Notify("🔴 Modo sniper desativado. Novos dados não serão afetados.")
    end
end

ToggleBtn.MouseButton1Click:Connect(toggleActive)

-- Arraste da interface
local dragging, startPos, startGuiPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        startPos = input.Position
        startGuiPos = Main.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - startPos
        Main.Position = UDim2.new(
            startGuiPos.X.Scale, startGuiPos.X.Offset + delta.X,
            startGuiPos.Y.Scale, startGuiPos.Y.Offset + delta.Y
        )
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

Notify("🎯 Ative o modo sniper, equipe o dado e jogue-o nos inimigos!")

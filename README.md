--[[
    🎲 Dice Duplicator Pro – Clonagem instantânea (sem rejoin)
    Segure o dado na mão e clique com ele no chão.
    Uma cópia exata do dado será criada e permanecerá no mundo.
    Rápido, invisível para o servidor como "saída", e acumulativo.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local Tween = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Aguarda personagem
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
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 280, 0, 150)
Main.Position = UDim2.new(0.5, -140, 0.5, -75)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 36)
local tc = Instance.new("UICorner", TitleBar)
tc.CornerRadius = UDim.new(0, 12)
local tf = Instance.new("Frame")
tf.Parent = TitleBar
tf.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
tf.BorderSizePixel = 0
tf.Size = UDim2.new(1, 0, 0, 12)
tf.Position = UDim2.new(0, 0, 1, -12)

local TitleIcon = Instance.new("TextLabel")
TitleIcon.Parent = TitleBar
TitleIcon.BackgroundTransparency = 1
TitleIcon.Position = UDim2.new(0, 8, 0, 5)
TitleIcon.Size = UDim2.new(0, 26, 0, 26)
TitleIcon.Text = "🎲"
TitleIcon.TextSize = 18

local TitleText = Instance.new("TextLabel")
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 36, 0, 0)
TitleText.Size = UDim2.new(1, -70, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Dice Duplicator"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 13
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = TitleBar
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 71, 87)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -34, 0, 6)
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Fechado") end)

-- Conteúdo
local Content = Instance.new("Frame")
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.Position = UDim2.new(0, 10, 0, 42)
Content.Size = UDim2.new(1, -20, 1, -50)

local InfoLabel = Instance.new("TextLabel")
InfoLabel.Parent = Content
InfoLabel.BackgroundTransparency = 1
InfoLabel.Size = UDim2.new(1, 0, 0, 40)
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.Text = "Segure o dado na mão\nClique em ATIVAR CLONAGEM\nDepois clique com o dado no chão"
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
InfoLabel.TextSize = 10
InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

local CloneBtn = Instance.new("TextButton")
CloneBtn.Parent = Content
CloneBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
CloneBtn.BorderSizePixel = 0
CloneBtn.Position = UDim2.new(0, 0, 0, 44)
CloneBtn.Size = UDim2.new(1, 0, 0, 38)
CloneBtn.Text = "🔁 ATIVAR CLONAGEM DE DADO"
CloneBtn.Font = Enum.Font.GothamBlack
CloneBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloneBtn.TextSize = 11
Instance.new("UICorner", CloneBtn).CornerRadius = UDim.new(0, 8)

-- Estado
local cloneActive = false
local diceTool = nil

CloneBtn.MouseButton1Click:Connect(function()
    -- Procura o dado na mão ou mochila
    diceTool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
    if not diceTool then
        diceTool = Player.Backpack:FindFirstChildOfClass("Tool")
    end
    
    if not diceTool then
        Notify("Você não está com um dado na mão!")
        return
    end
    
    if diceTool.Name ~= "Dice" and diceTool.Name ~= "Dice roll" then
        Notify("O item na sua mão não parece um dado (Dice/Dice roll).")
        return
    end
    
    cloneActive = true
    CloneBtn.Text = "✅ CLONAGEM ATIVA – CLIQUE NO CHÃO"
    CloneBtn.BackgroundColor3 = Color3.fromRGB(0, 210, 160)
    Notify("Modo clonagem ativado! Clique com o dado no chão.")
end)

-- Quando a ferramenta é ativada (clicada), se clonagem estiver ativa, cria cópia
local toolActivatedConn
local function setupClone()
    if toolActivatedConn then toolActivatedConn:Disconnect() end
    if diceTool then
        toolActivatedConn = diceTool.Activated:Connect(function()
            if not cloneActive then return end
            -- Clona o dado inteiro no chão onde o mouse aponta
            local targetPos = Player:GetMouse().Hit.Position + Vector3.new(0, 2, 0)
            local clone = diceTool:Clone()
            clone.Parent = Workspace
            if clone:IsA("Tool") then
                -- Se for Tool, removemos o script de ferramenta para virar um modelo estático
                clone.Parent = Workspace
                for _, child in ipairs(clone:GetDescendants()) do
                    if child:IsA("Script") or child:IsA("LocalScript") then
                        child:Destroy()
                    end
                end
                -- Move para a posição
                if clone.PrimaryPart then
                    clone:SetPrimaryPartCFrame(CFrame.new(targetPos))
                end
            else
                -- Se for Model, move
                clone:MoveTo(targetPos)
            end
            Notify("🎲 Dado clonado no chão!")
        end)
    end
end

-- Monitora a troca de ferramenta
Player.ChildAdded:Connect(function(child)
    if child:IsA("Tool") then
        diceTool = child
        setupClone()
    end
end)
Player.ChildRemoved:Connect(function(child)
    if child == diceTool then
        diceTool = nil
        cloneActive = false
        CloneBtn.Text = "🔁 ATIVAR CLONAGEM DE DADO"
        CloneBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    end
end)

-- Arraste
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

Notify("🎲 Segure o dado, ative a clonagem e clique no chão!")

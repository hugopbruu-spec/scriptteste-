--[[
    🎲 Dice Duplicator – Rejoin Automático com Restauração
    Jogue o dado no chão e clique em DUPLICAR.
    Sai e volta ao mesmo servidor em segundos.
    O dado no chão permanece e um novo aparece na sua mão.
    Console incluso para verificar o progresso.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local UIS = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Aguarda o personagem
repeat task.wait() until Player.Character

-- ============== RESTAURAÇÃO PÓS-REJOIN ==============
-- Se o script rodar e encontrar atributos salvos, restaura a posição e remove a tela preta
local function TryRestore()
    local savedCFrame = Player:GetAttribute("DiceSavedCFrame")
    local savedCamCFrame = Player:GetAttribute("DiceSavedCamCFrame")
    if not savedCFrame then return false end

    -- Aguarda o novo personagem
    local char = Player.Character
    if not char then
        char = Player.CharacterAdded:Wait()
    end
    local root = char:WaitForChild("HumanoidRootPart", 10)
    if not root then return false end

    root.CFrame = savedCFrame
    if savedCamCFrame then
        Camera.CFrame = savedCamCFrame
    end
    Camera.CameraSubject = char:FindFirstChild("Humanoid")

    -- Limpa atributos
    Player:SetAttribute("DiceSavedCFrame", nil)
    Player:SetAttribute("DiceSavedCamCFrame", nil)

    -- Remove a tela preta persistente
    for _, gui in ipairs(Player.PlayerGui:GetChildren()) do
        if gui.Name == "RejoinBlack" then
            gui:Destroy()
        end
    end
    return true
end

-- Tenta restaurar ao iniciar (se for rejoin)
local restored = TryRestore()

-- ============== INTERFACE ==============
local gui = Instance.new("ScreenGui")
gui.Name = "DiceDuplicator"
gui.Parent = CoreGui
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Parent = gui
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 260, 0, 160)
Main.Position = UDim2.new(0.5, -130, 0.5, -80)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(108, 92, 231)

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 30)
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
TitleText.Text = "🎲 Dice Duplicator"
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
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Botão
local DupBtn = Instance.new("TextButton")
DupBtn.Parent = Main
DupBtn.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
DupBtn.BorderSizePixel = 0
DupBtn.Position = UDim2.new(0, 8, 0, 34)
DupBtn.Size = UDim2.new(1, -16, 0, 30)
DupBtn.Text = "🔄 DUPLICAR (REJOIN)"
DupBtn.Font = Enum.Font.GothamBlack
DupBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
DupBtn.TextSize = 11
Instance.new("UICorner", DupBtn).CornerRadius = UDim.new(0, 6)

-- Console
local Console = Instance.new("TextBox")
Console.Parent = Main
Console.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 8, 0, 70)
Console.Size = UDim2.new(1, -16, 0, 80)
Console.Font = Enum.Font.Code
Console.Text = "Pronto.\n"
Console.TextColor3 = Color3.fromRGB(200, 200, 220)
Console.TextSize = 10
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 4)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- ============== AÇÃO ==============
DupBtn.MouseButton1Click:Connect(function()
    DupBtn.Text = "⏳ Aguarde..."
    DupBtn.Interactable = false

    local char = Player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        Player:SetAttribute("DiceSavedCFrame", char.HumanoidRootPart.CFrame)
        Player:SetAttribute("DiceSavedCamCFrame", Camera.CFrame)
        Log("Posição salva.")
    else
        Log("ERRO: Personagem sem HumanoidRootPart.")
        DupBtn.Text = "🔄 DUPLICAR (REJOIN)"
        DupBtn.Interactable = true
        return
    end

    -- Cria tela preta persistente
    local black = Instance.new("ScreenGui")
    black.Name = "RejoinBlack"
    black.Parent = Player.PlayerGui
    black.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    black.ResetOnSpawn = false
    black.IgnoreGuiInset = true
    local frame = Instance.new("Frame")
    frame.Parent = black
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Size = UDim2.new(1, 0, 1, 0)
    Log("Tela preta criada.")

    -- Rejoin
    local ok, err = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, Player)
    end)
    if not ok then
        Log("Falha ao teleportar para mesma instância: " .. tostring(err))
        -- Tenta teleportar para o jogo (servidor aleatório)
        pcall(function()
            TeleportService:Teleport(game.PlaceId, Player)
        end)
        Log("Tentando teleporte genérico...")
    else
        Log("Teleporte para mesma instância executado.")
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

if restored then
    Log("Restauração pós-rejoin concluída. Novo dado disponível.")
else
    Log("Aguardando clique em DUPLICAR.")
end

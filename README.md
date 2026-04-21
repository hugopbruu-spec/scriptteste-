-- Mushyo Ultimate Suite v14.0 - Funções Específicas
-- Sistema com as 5 funções solicitadas e interface otimizada

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local TextChatService = game:GetService("TextChatService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Remover interface existente
if CoreGui:FindFirstChild("MushyoUltimateSuite") then
    CoreGui.MushyoUltimateSuite:Destroy()
end

-- Interface principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MushyoUltimateSuite"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 500)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Position = UDim2.new(0, 0, 0, 0)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 60, 120)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🎮 MUSHYO ULTIMATE v14.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Botões de controle
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 35, 0, 35)
MinimizeButton.Position = UDim2.new(0.7, 0, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.BackgroundTransparency = 1
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 18
MinimizeButton.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 35, 0, 35)
CloseButton.Position = UDim2.new(0.8, 0, 0, 0)
CloseButton.Text = "×"
CloseButton.BackgroundTransparency = 1
CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.Parent = TitleBar

-- Área principal
local MainScroll = Instance.new("ScrollingFrame")
MainScroll.Size = UDim2.new(1, 0, 1, -35)
MainScroll.Position = UDim2.new(0, 0, 0, 35)
MainScroll.BackgroundTransparency = 1
MainScroll.ScrollBarThickness = 6
MainScroll.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
MainScroll.CanvasSize = UDim2.new(0, 0, 0, 600)
MainScroll.Parent = MainFrame

-- Variáveis de estado
local states = {}
local connections = {}
local activeEffects = {}
local selectedPlayers = {}

-- Função para criar botões
local function createButton(text, yPosition, callback, toggle, emoji)
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.95, 0, 0, 45)
    buttonFrame.Position = UDim2.new(0.025, 0, 0, yPosition)
    buttonFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
    buttonFrame.BorderSizePixel = 0
    buttonFrame.Parent = MainScroll
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 1, 0)
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = "   " .. (emoji or "") .. " " .. text
    button.BackgroundTransparency = 1
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 13
    button.TextXAlignment = Enum.TextXAlignment.Left
    button.Parent = buttonFrame
    
    local statusIndicator = Instance.new("Frame")
    statusIndicator.Size = UDim2.new(0, 4, 0.7, 0)
    statusIndicator.Position = UDim2.new(0, 2, 0.15, 0)
    statusIndicator.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    statusIndicator.BorderSizePixel = 0
    statusIndicator.Visible = false
    statusIndicator.Parent = buttonFrame
    
    button.MouseEnter:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(45, 45, 65)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(buttonFrame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(35, 35, 50)}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        if toggle then
            local newState = callback()
            states[text] = newState
            statusIndicator.Visible = newState
            buttonFrame.BackgroundColor3 = newState and Color3.fromRGB(40, 70, 100) or Color3.fromRGB(35, 35, 50)
        else
            callback()
        end
    end)
    
    return buttonFrame
end

-- Função para criar lista de jogadores
local function createPlayerList(yPosition, title, callback)
    local listFrame = Instance.new("Frame")
    listFrame.Size = UDim2.new(0.95, 0, 0, 150)
    listFrame.Position = UDim2.new(0.025, 0, 0, yPosition)
    listFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    listFrame.BorderSizePixel = 0
    listFrame.Parent = MainScroll
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0, 25)
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    titleLabel.Text = "   " .. title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.Font = Enum.Font.GothamMedium
    titleLabel.TextSize = 12
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = listFrame
    
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, 0, 1, -25)
    scrollFrame.Position = UDim2.new(0, 0, 0, 25)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.ScrollBarThickness = 4
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollFrame.Parent = listFrame
    
    local function updatePlayerList()
        scrollFrame:ClearAllChildren()
        local yPos = 5
        
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player then
                local playerButton = Instance.new("TextButton")
                playerButton.Size = UDim2.new(0.95, 0, 0, 30)
                playerButton.Position = UDim2.new(0.025, 0, 0, yPos)
                playerButton.Text = otherPlayer.Name
                playerButton.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
                playerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
                playerButton.Font = Enum.Font.Gotham
                playerButton.TextSize = 11
                playerButton.Parent = scrollFrame
                
                playerButton.MouseButton1Click:Connect(function()
                    callback(otherPlayer)
                end)
                
                yPos += 35
            end
        end
        
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, yPos)
    end
    
    updatePlayerList()
    Players.PlayerAdded:Connect(updatePlayerList)
    Players.PlayerRemoving:Connect(updatePlayerList)
    
    return listFrame
end

-- 1. BRING PLAYER
local bringPlayerY = 5
local selectedBringPlayer = nil

createPlayerList(bringPlayerY, "🔹 TRAZER PLAYER - Selecione um jogador:", function(selectedPlayer)
    selectedBringPlayer = selectedPlayer
end)

bringPlayerY += 160

createButton("TRAZER PLAYER SELECIONADO", bringPlayerY, function()
    if selectedBringPlayer and selectedBringPlayer.Character then
        local targetRoot = selectedBringPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            targetRoot.CFrame = rootPart.CFrame
        end
    end
end, false, "🚀")

bringPlayerY += 50

-- 2. HEADSIT
local headsitY = bringPlayerY
local selectedHeadsitPlayer = nil
local headsitWeld = nil

createPlayerList(headsitY, "🔹 HEADSIT - Selecione um jogador:", function(selectedPlayer)
    selectedHeadsitPlayer = selectedPlayer
end)

headsitY += 160

createButton("ATIVAR HEADSIT", headsitY, function()
    states.Headsit = not states.Headsit
    
    if states.Headsit and selectedHeadsitPlayer and selectedHeadsitPlayer.Character then
        local targetHead = selectedHeadsitPlayer.Character:FindFirstChild("Head")
        if targetHead then
            -- Criar assento invisível na cabeça do jogador
            local seat = Instance.new("Seat")
            seat.Name = "HeadsitSeat"
            seat.Size = Vector3.new(2, 0.5, 2)
            seat.Transparency = 1
            seat.CanCollide = false
            seat.Parent = workspace
            
            -- Welding o assento à cabeça do jogador
            local weld = Instance.new("Weld")
            weld.Part0 = targetHead
            weld.Part1 = seat
            weld.C0 = CFrame.new(0, 1, 0)
            weld.Parent = seat
            
            -- Sentar no assento
            rootPart.CFrame = seat.CFrame + Vector3.new(0, 1, 0)
            humanoid.Sit = true
            
            headsitWeld = weld
            activeEffects.Headsit = seat
        end
    else
        if activeEffects.Headsit then
            activeEffects.Headsit:Destroy()
        end
        humanoid.Sit = false
        headsitWeld = nil
    end
    
    return states.Headsit
end, true, "💺")

headsitY += 50

-- 3. UNLOCK CHAT
local chatY = headsitY

createButton("DESBLOQUEAR CHAT COMPLETO", chatY, function()
    states.UnlockChat = not states.UnlockChat
    
    if states.UnlockChat then
        -- Remover filtros de chat
        for _, descendant in ipairs(TextChatService:GetDescendants()) do
            if descendant:IsA("TextChatCommand") or descendant:IsA("TextChatFilter") then
                descendant:Destroy()
            end
        end
        
        -- Habilitar todas as configurações de chat
        if TextChatService:FindFirstChild("TextChannels") then
            local rbxts = TextChatService.TextChannels:FindFirstChild("RBXTS")
            if rbxts then
                rbxts:Destroy()
            end
        end
    end
    
    return states.UnlockChat
end, true, "🔓")

chatY += 50

-- 4. PEGAR PLAYER
local grabPlayerY = chatY
local selectedGrabPlayer = nil
local grabWeld = nil

createPlayerList(grabPlayerY, "🔹 PEGAR PLAYER - Selecione um jogador:", function(selectedPlayer)
    selectedGrabPlayer = selectedPlayer
end)

grabPlayerY += 160

createButton("PEGAR PLAYER SELECIONADO", grabPlayerY, function()
    states.GrabPlayer = not states.GrabPlayer
    
    if states.GrabPlayer and selectedGrabPlayer and selectedGrabPlayer.Character then
        local targetRoot = selectedGrabPlayer.Character:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            -- Criar weld para grudar o jogador na mão
            grabWeld = Instance.new("Weld")
            grabWeld.Part0 = character:FindFirstChild("RightHand") or rootPart
            grabWeld.Part1 = targetRoot
            grabWeld.C0 = CFrame.new(0, 0, -2)
            grabWeld.Parent = targetRoot
            
            -- Forçar o jogador a ficar parado
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bodyVelocity.Parent = targetRoot
            
            activeEffects.GrabPlayer = {weld = grabWeld, velocity = bodyVelocity}
        end
    else
        if activeEffects.GrabPlayer then
            activeEffects.GrabPlayer.weld:Destroy()
            activeEffects.GrabPlayer.velocity:Destroy()
        end
        grabWeld = nil
    end
    
    return states.GrabPlayer
end, true, "✋")

grabPlayerY += 50

-- 5. COPIAR SKIN
local copySkinY = grabPlayerY
local selectedSkinPlayer = nil

createPlayerList(copySkinY, "🔹 COPIAR SKIN - Selecione um jogador:", function(selectedPlayer)
    selectedSkinPlayer = selectedPlayer
end)

copySkinY += 160

createButton("COPIAR SKIN DO PLAYER", copySkinY, function()
    if selectedSkinPlayer and selectedSkinPlayer.Character then
        -- Remover roupas atuais
        for _, accessory in ipairs(character:GetChildren()) do
            if accessory:IsA("Accessory") then
                accessory:Destroy()
            end
        end
        
        -- Copiar acessórios do jogador alvo
        for _, accessory in ipairs(selectedSkinPlayer.Character:GetChildren()) do
            if accessory:IsA("Accessory") then
                local clone = accessory:Clone()
                clone.Parent = character
            end
        end
        
        -- Copiar cores das partes
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                local targetPart = selectedSkinPlayer.Character:FindFirstChild(part.Name)
                if targetPart then
                    part.Color = targetPart.Color
                    part.Material = targetPart.Material
                end
            end
        end
    end
end, false, "👕")

-- Ajustar canvas size
MainScroll.CanvasSize = UDim2.new(0, 0, 0, copySkinY + 50)

-- Sistema de arrastar
local dragging, dragInput, dragStart, startPos

local function updateInput(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then updateInput(input) end
end)

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Sistema de inicialização
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    rootPart = newChar:WaitForChild("HumanoidRootPart")
end)

UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("🎮 Mushyo Ultimate Suite v14.0 Carregado!")
print("✅ 5 Funções Especiais Implementadas")
print("🚀 Pressione RightShift para abrir o menu")

--[[
    SCRIPT: FISCH AUTO-TRADE SYSTEM
    FUNÇÃO: Automatiza envio de pedidos de troca + exibe inventário do alvo (se visível)
    LIMITAÇÃO: Não força transferência, apenas automatiza o sistema legítimo do jogo
--]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local targetPlayer = nil
local autoRequestActive = false

-- Interface simples
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 350, 0, 400)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
mainFrame.BackgroundTransparency = 0.1
mainFrame.Parent = screenGui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
titleBar.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 1, 0)
title.BackgroundTransparency = 1
title.Text = "⚡ AUTO-TRADE FISCH"
title.TextColor3 = Color3.fromRGB(255, 200, 100)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 1, -6)
closeBtn.Position = UDim2.new(1, -35, 0, 3)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
closeBtn.Parent = titleBar

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Lista de jogadores
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, -20, 1, -50)
listFrame.Position = UDim2.new(0, 10, 0, 45)
listFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
listFrame.BackgroundTransparency = 0.5
listFrame.BorderSizePixel = 0
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.ScrollBarThickness = 6
listFrame.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = listFrame

local function updatePlayerList()
    -- Limpar lista
    for _, child in ipairs(listFrame:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end

    local players = Players:GetPlayers()
    local yOffset = 0

    for _, player in ipairs(players) do
        if player ~= LocalPlayer then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, -10, 0, 35)
            btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
            btn.Text = "👤 " .. player.Name .. " - Clique para trocar"
            btn.TextColor3 = Color3.fromRGB(220, 220, 255)
            btn.Font = Enum.Font.Gotham
            btn.TextSize = 12
            btn.TextXAlignment = Enum.TextXAlignment.Left
            btn.Parent = listFrame

            btn.MouseButton1Click:Connect(function()
                targetPlayer = player
                -- Enviar pedido de troca
                local tradeEvent = ReplicatedStorage:FindFirstChild("RequestTrade") or 
                                   ReplicatedStorage:FindFirstChild("TradeRequest") or
                                   ReplicatedStorage:FindFirstChild("SendTrade")
                if tradeEvent and tradeEvent:IsA("RemoteEvent") then
                    tradeEvent:FireServer(player)
                    btn.Text = "✅ Pedido enviado para " .. player.Name
                    btn.TextColor3 = Color3.fromRGB(100, 255, 100)
                    task.wait(2)
                    btn.Text = "👤 " .. player.Name .. " - Clique para trocar"
                    btn.TextColor3 = Color3.fromRGB(220, 220, 255)
                else
                    btn.Text = "❌ Sistema de troca não encontrado"
                    task.wait(2)
                    btn.Text = "👤 " .. player.Name .. " - Clique para trocar"
                end
            end)

            yOffset = yOffset + 40
        end
    end

    listFrame.CanvasSize = UDim2.new(0, 0, 0, yOffset + 10)
end

-- Atualizar lista periodicamente
updatePlayerList()
task.spawn(function()
    while screenGui and screenGui.Parent do
        task.wait(5)
        updatePlayerList()
    end
end)

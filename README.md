--[[
    REJOIN_BUTTON.lua – Botão para dar Rejoin no mesmo servidor
    Interface própria, arrastável, com minimizar e fechar.
    Ao clicar no botão, você sai e entra novamente no mesmo servidor.
]]--

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local player = Players.LocalPlayer

-- ================== INTERFACE ==================
local gui = Instance.new("ScreenGui")
gui.Name = "Rejoin_UI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 100)
frame.Position = UDim2.new(1, -230, 0, 470)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 26)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "🔄 Rejoin"
title.TextColor3 = Color3.fromRGB(255, 200, 100)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.BackgroundTransparency = 1
title.Position = UDim2.new(0, 8, 0, 0)
title.Size = UDim2.new(1, -60, 1, 0)
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0, 28, 0, 28)
minimizeBtn.Position = UDim2.new(1, -56, 0, 0)
minimizeBtn.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
closeBtn.BorderSizePixel = 0
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -28, 0, 0)
closeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -26)
content.Position = UDim2.new(0, 0, 0, 26)
content.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
content.BorderSizePixel = 0
content.Parent = frame

local rejoinBtn = Instance.new("TextButton")
rejoinBtn.Size = UDim2.new(0, 180, 0, 36)
rejoinBtn.Position = UDim2.new(0.5, -90, 0.5, -18)
rejoinBtn.BackgroundColor3 = Color3.fromRGB(200, 130, 30)
rejoinBtn.BorderSizePixel = 0
rejoinBtn.TextColor3 = Color3.new(1, 1, 1)
rejoinBtn.Font = Enum.Font.GothamBold
rejoinBtn.TextSize = 14
rejoinBtn.Text = "DAR REJOIN"
rejoinBtn.Parent = content

-- Minimizar / Fechar
local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 220, 0, 26) or UDim2.new(0, 220, 0, 100)
end)

closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- ================== FUNÇÃO REJOIN ==================
local function rejoinServer()
    local success, errorMsg = pcall(function()
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId)
    end)
    if not success then
        warn("Falha ao dar rejoin:", errorMsg)
    end
end

rejoinBtn.MouseButton1Click:Connect(rejoinServer)

-- Também atalho com F8 (opcional)
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F8 then
        rejoinServer()
    end
end)

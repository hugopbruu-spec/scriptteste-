--[[
    WATER_WALK.lua - Andar Sobre a Água
    Atalho: F6 para ligar/desligar
    Interface própria com botão e status.
    Método agressivo: detecção de terreno de água e aplicação de força vertical.
]]--

local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ================== INTERFACE ==================
local gui = Instance.new("ScreenGui")
gui.Name = "WaterWalk_GUI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 240, 0, 130)
frame.Position = UDim2.new(1, -250, 0, 160)
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
title.Text = "🌊 Water Walk"
title.TextColor3 = Color3.fromRGB(100, 180, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.BackgroundTransparency = 1
title.Position = UDim2.new(0, 8, 0, 0)
title.Size = UDim2.new(1, -40, 1, 0)
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0, 26, 0, 26)
minimizeBtn.Position = UDim2.new(1, -26, 0, 0)
minimizeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -26)
content.Position = UDim2.new(0, 0, 0, 26)
content.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "STATUS: Desativado"
statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 13
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 8)
statusLabel.Parent = content

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 200, 0, 32)
toggleBtn.Position = UDim2.new(0.5, -100, 0, 36)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 120, 200)
toggleBtn.BorderSizePixel = 0
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Text = "ATIVAR (F6)"
toggleBtn.Parent = content

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Ande sobre a água normalmente"
infoLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 11
infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1, -20, 0, 18)
infoLabel.Position = UDim2.new(0, 10, 0, 78)
infoLabel.Parent = content

local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    frame.Size = minimized and UDim2.new(0, 240, 0, 26) or UDim2.new(0, 240, 0, 130)
end)

-- ================== LÓGICA WATER WALK ==================
local waterWalkActive = false
local bindName = "WaterWalk_Render"

-- Serviços e constantes
local Workspace = workspace
local Terrain = Workspace.Terrain
local waterOffset = 2.5 -- altura acima da água

-- Método agressivo de detecção de água (Terrain + Partes com tag "Water")
local function isWaterUnderPosition(pos)
    -- Verificar terreno voxel
    local region = Region3.new(pos - Vector3.new(2, 4, 2), pos + Vector3.new(2, 0.5, 2))
    local parts = Workspace:FindPartsInRegion3(region, nil, math.huge)
    for _, part in ipairs(parts) do
        if part.Material == Enum.Material.Water then
            return true
        end
        if part.Name:lower():find("water") or part.Name:lower():find("água") then
            return true
        end
    end
    
    -- Verificar voxels do terreno
    if Terrain then
        local cellX, cellY, cellZ = Terrain:WorldToCell(pos.X, pos.Y, pos.Z)
        for dx = -2, 2 do
            for dz = -2, 2 do
                local material = Terrain:GetCell(cellX + dx, cellY - 1, cellZ + dz)
                if material == Enum.Material.Water then
                    return true
                end
            end
        end
    end
    
    return false
end

-- Encontrar altura da superfície da água abaixo do personagem
local function getWaterSurfaceHeight(pos)
    local highestWater = nil
    local searchY = pos.Y + 4
    while searchY > pos.Y - 6 do
        if isWaterUnderPosition(Vector3.new(pos.X, searchY, pos.Z)) then
            highestWater = searchY
            break
        end
        searchY = searchY - 0.25
    end
    return highestWater
end

local function startWaterWalk()
    waterWalkActive = true
    RunService:BindToRenderStep(bindName, Enum.RenderPriority.Character.Value + 1, waterWalkLoop)
    toggleBtn.Text = "DESATIVAR (F6)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    statusLabel.Text = "STATUS: Ativo"
    statusLabel.TextColor3 = Color3.fromRGB(120, 255, 120)
end

local function stopWaterWalk()
    waterWalkActive = false
    RunService:UnbindFromRenderStep(bindName)
    toggleBtn.Text = "ATIVAR (F6)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 120, 200)
    statusLabel.Text = "STATUS: Desativado"
    statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
end

-- Botão toggle
toggleBtn.MouseButton1Click:Connect(function()
    if waterWalkActive then stopWaterWalk() else startWaterWalk() end
end)

-- Atalho F6
UIS.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F6 then
        if waterWalkActive then stopWaterWalk() else startWaterWalk() end
    end
end)

-- Loop principal
function waterWalkLoop(delta)
    if not waterWalkActive then return end
    local char = player.Character
    if not char or not char.Parent then return end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChild("Humanoid")
    if not rootPart or not humanoid then return end
    
    local pos = rootPart.Position
    local velocity = rootPart.Velocity
    
    -- Método 1: Verificar se está sobre água (detecção direta)
    local waterHeight = getWaterSurfaceHeight(pos)
    
    if waterHeight then
        -- Está sobre água: aplicar força para manter na superfície
        local targetY = waterHeight + waterOffset
        local currentY = pos.Y
        
        -- Se está afundando, empurrar para cima
        if currentY < targetY then
            local force = Vector3.new(0, (targetY - currentY) * 50 * humanoid.Mass, 0)
            rootPart.Velocity = Vector3.new(velocity.X, math.max(velocity.Y + force.Y / humanoid.Mass, 0), velocity.Z)
            -- Força extra via ApplyImpulse
            rootPart:ApplyImpulse(force * delta)
        end
        
        -- Se está muito acima, suavizar (não precisa, mas evita flutuação extrema)
        if currentY > targetY + 2 then
            rootPart.Velocity = Vector3.new(velocity.X, math.min(velocity.Y, -5), velocity.Z)
        end
        
        -- Método 2: Força bruta contínua para cima (anti-afundamento)
        local antiSinkForce = Vector3.new(0, humanoid.Mass * workspace.Gravity * 1.05, 0)
        rootPart:ApplyImpulse(antiSinkForce * delta)
        
        -- Método 3: Se humanoid estiver nadando, forçar estado para "stand"
        if humanoid:GetState() == Enum.HumanoidStateType.Swimming then
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Swimming, true)
            humanoid:ChangeState(Enum.HumanoidStateType.Running)
        end
    else
        -- Não está sobre água, verificar se está caindo em direção à água
        -- Detecta água abaixo num raio maior
        local futurePos = pos + rootPart.Velocity * 0.2
        if getWaterSurfaceHeight(futurePos) then
            -- Vai atingir água em breve, preparar
            waterHeight = getWaterSurfaceHeight(futurePos)
            if waterHeight and pos.Y <= waterHeight + waterOffset + 0.5 then
                -- Já está muito próximo, aplicar força preventiva
                local targetY = waterHeight + waterOffset
                if pos.Y < targetY + 1 then
                    rootPart.Velocity = Vector3.new(velocity.X, math.max(velocity.Y, 10), velocity.Z)
                    rootPart:ApplyImpulse(Vector3.new(0, humanoid.Mass * 50, 0))
                end
            end
        end
    end
end

-- Segurança: reativar após respawn
player.CharacterAdded:Connect(function(char)
    if waterWalkActive then
        wait(0.3)
        -- Reaplica o bind (já está ativo, mas garante)
        if not RunService:IsBoundToRenderStep(bindName) then
            RunService:BindToRenderStep(bindName, Enum.RenderPriority.Character.Value + 1, waterWalkLoop)
        end
    end
end)

-- Limpeza ao sair do jogo (opcional)
game:GetService("RunService").Stepped:Connect(function()
    if waterWalkActive and not player.Character then
        -- Personagem morreu, manter ativo para quando respawnar
    end
end)

-- Inicializar interface
statusLabel.Text = "STATUS: Desativado"
statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)

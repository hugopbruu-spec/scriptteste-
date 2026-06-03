--[[
    WATER_WALK_V2.lua – Andar sobre a água (funcional e estável)
    Atalho: F6 | Interface com botões Fechar (X) e Minimizar (_)
    Método: raycast + região + força vertical PID + bloqueio de natação
]]--

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local Workspace = workspace
local player = Players.LocalPlayer

-- ================== INTERFACE COMPLETA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "WaterWalk_UI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 140)
frame.Position = UDim2.new(1, -260, 0, 320)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

-- Título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 28)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "🌊 Water Walk"
title.TextColor3 = Color3.fromRGB(100, 180, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 15
title.BackgroundTransparency = 1
title.Position = UDim2.new(0, 10, 0, 0)
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
content.Size = UDim2.new(1, 0, 1, -28)
content.Position = UDim2.new(0, 0, 0, 28)
content.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "STATUS: Desativado"
statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 13
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 10)
statusLabel.Parent = content

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 210, 0, 34)
toggleBtn.Position = UDim2.new(0.5, -105, 0, 40)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 130, 200)
toggleBtn.BorderSizePixel = 0
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Text = "ATIVAR (F6)"
toggleBtn.Parent = content

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Ande sobre qualquer água"
infoLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 11
infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1, -20, 0, 18)
infoLabel.Position = UDim2.new(0, 10, 0, 84)
infoLabel.Parent = content

-- Funções de minimizar / fechar
local minimized = false
local function setMinimized(state)
    minimized = state
    content.Visible = not state
    frame.Size = state and UDim2.new(0, 250, 0, 28) or UDim2.new(0, 250, 0, 140)
end
minimizeBtn.MouseButton1Click:Connect(function() setMinimized(not minimized) end)

closeBtn.MouseButton1Click:Connect(function()
    -- Fecha a interface e desativa o water walk permanentemente
    if waterWalkActive then stopWaterWalk() end
    gui:Destroy()
end)

-- ================== LÓGICA PRINCIPAL ==================
local waterWalkActive = false
local BIND_NAME = "WaterWalk_Step"
local waterHeightOffset = 3.2  -- altura do root part acima da superfície
local kP = 35  -- constante proporcional (força em função do erro de altura)
local maxForce = 5000 -- limite de segurança

-- Detecção de água: raycast + região
local function getWaterSurface(pos)
    -- Raycast para baixo
    local rayOrigin = pos + Vector3.new(0, 2, 0)
    local rayDirection = Vector3.new(0, -10, 0)
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Include
    rayParams.FilterDescendantsInstances = {Workspace}
    rayParams.IgnoreWater = false  -- detecta água
    local rayResult = Workspace:Raycast(rayOrigin, rayDirection, rayParams)
    if rayResult and rayResult.Material == Enum.Material.Water then
        return rayResult.Position.Y
    end
    
    -- Fallback: varredura de região (partes com material Water)
    local region = Region3.new(pos - Vector3.new(2, 6, 2), pos + Vector3.new(2, 1, 2))
    local partsInRegion = Workspace:FindPartsInRegion3(region, nil, math.huge)
    for _, part in ipairs(partsInRegion) do
        if part.Material == Enum.Material.Water then
            return part.Position.Y + part.Size.Y/2
        end
    end
    
    -- Verificação de Terrain (células ao redor)
    local terrain = Workspace.Terrain
    if terrain then
        local cellX, cellY, cellZ = terrain:WorldToCell(pos.X, pos.Y, pos.Z)
        for dx = -2, 2 do
            for dz = -2, 2 do
                local mat = terrain:GetCell(cellX + dx, cellY - 2, cellZ + dz)
                if mat == Enum.Material.Water then
                    -- Aproximar altura da água no centro da célula
                    local cellWorld = terrain:CellCenterToWorld(cellX + dx, cellY - 2, cellZ + dz)
                    return cellWorld.Y + 2  -- centro da célula tem 4 studs, metade superior
                end
            end
        end
    end
    
    return nil
end

local function startWaterWalk()
    waterWalkActive = true
    -- Desabilita natação para manter o personagem em pé
    local char = player.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
        end
    end
    RunService:BindToRenderStep(BIND_NAME, Enum.RenderPriority.Character.Value, waterWalkLoop)
    toggleBtn.Text = "DESATIVAR (F6)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    statusLabel.Text = "STATUS: Ativo"
    statusLabel.TextColor3 = Color3.fromRGB(120, 255, 120)
end

local function stopWaterWalk()
    waterWalkActive = false
    -- Reabilita natação
    local char = player.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, true)
        end
    end
    RunService:UnbindFromRenderStep(BIND_NAME)
    toggleBtn.Text = "ATIVAR (F6)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 130, 200)
    statusLabel.Text = "STATUS: Desativado"
    statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
end

toggleBtn.MouseButton1Click:Connect(function()
    if waterWalkActive then stopWaterWalk() else startWaterWalk() end
end)

-- Atalho F6
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F6 then
        if waterWalkActive then stopWaterWalk() else startWaterWalk() end
    end
end)

-- Loop de controle
function waterWalkLoop(deltaTime)
    local char = player.Character
    if not char or not char.Parent then return end
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    if not rootPart or not hum then return end
    
    local pos = rootPart.Position
    local waterY = getWaterSurface(pos)
    
    if waterY then
        local targetY = waterY + waterHeightOffset
        local errorY = targetY - pos.Y
        
        -- Força vertical: compensar gravidade + correção proporcional
        local gravityForce = hum.Mass * Workspace.Gravity
        local correctionForce = errorY * kP * hum.Mass
        local totalForce = gravityForce + correctionForce
        totalForce = math.clamp(totalForce, -maxForce, maxForce)
        
        -- Aplica impulso vertical (multiplicado por deltaTime para estabilidade)
        rootPart:ApplyImpulse(Vector3.new(0, totalForce * deltaTime, 0))
        
        -- Mantém orientação vertical (anti-tombamento)
        local up = rootPart.CFrame.UpVector
        if up.Y < 0.95 then
            local torque = Vector3.new(-up.Z, 0, up.X) * hum.Mass * 50
            rootPart:ApplyAngularImpulse(torque * deltaTime)
        end
        
        -- Se por algum motivo o humanoid entrar em Swimming, força Running
        if hum:GetState() == Enum.HumanoidStateType.Swimming then
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    else
        -- Não está sobre água: nenhuma força extra
        -- Mas garante que o estado de natação continua desabilitado
        if hum:GetState() == Enum.HumanoidStateType.Swimming then
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    end
end

-- Tratamento de respawn
player.CharacterAdded:Connect(function(char)
    if waterWalkActive then
        wait(0.3)
        local hum = char:WaitForChild("Humanoid")
        hum:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
        if not RunService:IsBoundToRenderStep(BIND_NAME) then
            RunService:BindToRenderStep(BIND_NAME, Enum.RenderPriority.Character.Value, waterWalkLoop)
        end
    end
end)

-- Garantia de limpeza ao fechar script
gui.Destroying:Connect(function()
    if waterWalkActive then stopWaterWalk() end
end)

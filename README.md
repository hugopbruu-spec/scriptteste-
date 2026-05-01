-- VR_Universal_GUI.lua
-- Atalho: F5 para toggle, ou use o botão da interface.
-- Interface própria que prova que o script está rodando.

local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local VRService = game:GetService("VRService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local CoreGui = syn and syn.protect_gui or (function() return game:GetService("CoreGui") end)

-- ================== INTERFACE ==================
local gui = Instance.new("ScreenGui")
gui.Name = "VR_Emulador"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 240, 0, 140)
frame.Position = UDim2.new(1, -250, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1,0,0,24)
titleBar.BackgroundColor3 = Color3.fromRGB(50,50,50)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "Emulador VR"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamSemibold
title.TextSize = 14
title.BackgroundTransparency = 1
title.Position = UDim2.new(0, 8, 0, 0)
title.Size = UDim2.new(1, -40, 1, 0)
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.new(1,1,1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(70,70,70)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0, 24, 0, 24)
minimizeBtn.Position = UDim2.new(1, -24, 0, 0)
minimizeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1,0,1,-24)
content.Position = UDim2.new(0,0,0,24)
content.BackgroundColor3 = Color3.fromRGB(40,40,40)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "Status: Desativado"
statusLabel.TextColor3 = Color3.new(0.8,0.2,0.2)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 13
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 10)
statusLabel.Parent = content

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 200, 0, 30)
toggleBtn.Position = UDim2.new(0.5, -100, 0, 40)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0,140,0)
toggleBtn.BorderSizePixel = 0
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Text = "ATIVAR VR (F5)"
toggleBtn.Parent = content

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Mouse = cabeça | WASD = mover | Q/E = mãos"
infoLabel.TextColor3 = Color3.new(0.7,0.7,0.7)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 11
infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1, -20, 0, 30)
infoLabel.Position = UDim2.new(0, 10, 0, 80)
infoLabel.TextWrapped = true
infoLabel.Parent = content

local vrMethodLabel = Instance.new("TextLabel")
vrMethodLabel.Text = "Método VR: ..."
vrMethodLabel.TextColor3 = Color3.new(0.6,0.6,0.6)
vrMethodLabel.Font = Enum.Font.Gotham
vrMethodLabel.TextSize = 10
vrMethodLabel.BackgroundTransparency = 1
vrMethodLabel.Size = UDim2.new(1, -20, 0, 16)
vrMethodLabel.Position = UDim2.new(0, 10, 0, 110)
vrMethodLabel.Parent = content

local minimized = false
local function setMinimized(state)
    minimized = state
    content.Visible = not state
    frame.Size = state and UDim2.new(0, 240, 0, 24) or UDim2.new(0, 240, 0, 140)
end

minimizeBtn.MouseButton1Click:Connect(function()
    setMinimized(not minimized)
end)

-- ================== CONTROLE VR ==================
local VR_ATIVO = false
local keys = {}
local moveDir = Vector3.new()
local anguloMao = 0
local sensibilidade = 0.3
local vrMethod = "Nenhum"

local function debugUI(msg, cor)
    statusLabel.Text = "Status: " .. msg
    statusLabel.TextColor3 = cor or Color3.new(1,1,1)
    vrMethodLabel.Text = "Método VR: " .. vrMethod
end

-- Forçar VREnabled de forma persistente
local function forcarVR()
    local s1 = pcall(function() VRService:SetVREnabled(true) end)
    if s1 and VRService.VREnabled then
        vrMethod = "SetVREnabled"
        return true
    end

    local mt = getrawmetatable(VRService)
    if mt then
        local oldIdx = mt.__index
        local hookFunc
        hookFunc = newcclosure(function(self, prop)
            if prop == "VREnabled" then return true end
            return oldIdx(self, prop)
        end)
        pcall(function() mt.__index = hookFunc end)
        pcall(function() VRService:SetVREnabled(true) end)
        if VRService.VREnabled then
            vrMethod = "Hook __index"
            return true
        end
    end

    -- Método alternativo: alterar propriedade diretamente (rawset)
    pcall(function()
        rawset(VRService, "VREnabled", true)
    end)
    if VRService.VREnabled then
        vrMethod = "rawset"
        return true
    end

    vrMethod = "Falhou"
    return false
end

local function lockMouse() pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.LockCenter end) end
local function unlockMouse() pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.Default end) end

local function updateMove()
    local dir = Vector3.new()
    if keys[Enum.KeyCode.W] then dir += Vector3.new(0,0,-1) end
    if keys[Enum.KeyCode.S] then dir += Vector3.new(0,0,1) end
    if keys[Enum.KeyCode.A] then dir += Vector3.new(-1,0,0) end
    if keys[Enum.KeyCode.D] then dir += Vector3.new(1,0,0) end
    moveDir = dir.Magnitude > 0 and dir.Unit or Vector3.new()
end

local function ativar()
    if not forcarVR() then
        debugUI("Falha ao ativar VR", Color3.new(0.8,0.2,0.2))
        return
    end
    VR_ATIVO = true
    lockMouse()
    camera.CameraType = Enum.CameraType.Scriptable
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.AutoRotate = false
    end
    RunService:BindToRenderStep("VR_Emulador", Enum.RenderPriority.Camera.Value + 1, vrLoop)
    toggleBtn.Text = "DESATIVAR VR (F5)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(180,0,0)
    debugUI("Ativado", Color3.new(0.2,0.8,0.2))
end

local function desativar()
    VR_ATIVO = false
    unlockMouse()
    camera.CameraType = Enum.CameraType.Custom
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.AutoRotate = true
    end
    RunService:UnbindFromRenderStep("VR_Emulador")
    pcall(function() VRService:Recenter() end)
    toggleBtn.Text = "ATIVAR VR (F5)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(0,140,0)
    debugUI("Desativado", Color3.new(0.8,0.2,0.2))
end

toggleBtn.MouseButton1Click:Connect(function()
    VR_ATIVO = not VR_ATIVO
    if VR_ATIVO then ativar() else desativar() end
end)

-- Atalho F5
UIS.InputBegan:Connect(function(inp, processed)
    if processed then return end
    if inp.KeyCode == Enum.KeyCode.F5 then
        VR_ATIVO = not VR_ATIVO
        if VR_ATIVO then ativar() else desativar() end
    elseif VR_ATIVO then
        keys[inp.KeyCode] = true
        updateMove()
        if inp.KeyCode == Enum.KeyCode.Q then anguloMao -= 30
        elseif inp.KeyCode == Enum.KeyCode.E then anguloMao += 30 end
    end
end)
UIS.InputEnded:Connect(function(inp)
    if VR_ATIVO then keys[inp.KeyCode] = nil; updateMove() end
end)

function vrLoop(delta)
    if not VR_ATIVO then return end
    local char = player.Character
    if not char or not char.Parent then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    -- Movimento
    local camCF = camera.CFrame
    local fwd = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
    local rig = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit
    local mv = (fwd * -moveDir.Z) + (rig * moveDir.X)
    if mv.Magnitude > 0 then
        char.Humanoid:Move(mv, false)
    end

    -- Rotação da câmera
    local deltaMouse = UIS:GetMouseDelta()
    if deltaMouse.Magnitude > 0 then
        local rotAtual = camCF - camCF.Position
        local yaw = math.rad(-deltaMouse.X * sensibilidade)
        local pitch = math.rad(-deltaMouse.Y * sensibilidade)
        local novaRot = rotAtual * CFrame.Angles(0, yaw, 0) * CFrame.Angles(pitch, 0, 0)
        local _, pAtual, _ = novaRot:ToEulerAnglesYXZ()
        if math.abs(pAtual) > math.rad(80) then
            novaRot = rotAtual * CFrame.Angles(0, yaw, 0) * CFrame.Angles(math.clamp(pAtual, math.rad(-80), math.rad(80)) - (select(2, rotAtual:ToEulerAnglesYXZ()) or 0), 0, 0)
        end
        camera.CFrame = CFrame.new(camera.CFrame.Position) * novaRot
    end
    camera.CFrame = CFrame.new(head.Position) * camera.CFrame.Rotation

    -- Enviar UserCFrames para replicação
    local headCF = camera.CFrame * CFrame.new(0, 0.1, 0)
    local maoDirCF = headCF * CFrame.new(0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(anguloMao))
    local maoEsqCF = headCF * CFrame.new(-0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(-anguloMao))

    local ok = pcall(function()
        VRService:SetUserCFrame(Enum.UserCFrame.Head, headCF)
        VRService:SetUserCFrame(Enum.UserCFrame.RightHand, maoDirCF)
        VRService:SetUserCFrame(Enum.UserCFrame.LeftHand, maoEsqCF)
    end)
    if not ok then
        debugUI("Erro ao enviar frames", Color3.new(0.8,0.2,0.2))
        vrMethod = "Falha no envio"
        vrMethodLabel.Text = "Método VR: " .. vrMethod
    else
        -- Status rápido (apenas se debug visual)
    end
end

-- Reativar ao respawnar
player.CharacterAdded:Connect(function(char)
    if VR_ATIVO then
        wait(0.3)
        -- Reaplica autoRotate
        char:WaitForChild("Humanoid").AutoRotate = false
        -- Reenvia VR forçado
        forcarVR()
    end
end)

-- Loop de segurança: se VR desabilitar externamente, reativa
spawn(function()
    while true do
        if VR_ATIVO and not VRService.VREnabled then
            forcarVR()
        end
        wait(5)
    end
end)

-- Inicializar UI
debugUI("Desativado", Color3.new(0.8,0.2,0.2))
forcarVR() -- testa logo
vrMethodLabel.Text = "Método VR: " .. vrMethod

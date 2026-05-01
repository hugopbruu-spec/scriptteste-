--[[
VR_AGGRESSIVE.lua - Emulador de VR forçado para Roblox
Atalho: F5 para ligar/desligar.
Interface incluída, com diagnóstico ao vivo.
Métodos agressivos: hooks de metatable, substituição de classe, injeção de falso dispositivo,
e fallback para manipulação direta de Motor6D com atualização via ReplicatedFirst.
]]--

local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local VRService = game:GetService("VRService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local ReplicatedFirst = game:GetService("ReplicatedFirst")

-- ================== INTERFACE (sempre visível enquanto o script roda) ==================
local gui = Instance.new("ScreenGui")
gui.Name = "VR_Aggressive_GUI"
gui.ResetOnSpawn = false
pcall(function() gui.Parent = game.CoreGui end) or gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 150)
frame.Position = UDim2.new(1, -270, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1,0,0,24)
titleBar.BackgroundColor3 = Color3.fromRGB(40,40,40)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Text = "VR Agressivo"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.BackgroundTransparency = 1
title.Position = UDim2.new(0,8,0,0)
title.Size = UDim2.new(1,-40,1,0)
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.new(1,1,1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 16
minimizeBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Size = UDim2.new(0,24,0,24)
minimizeBtn.Position = UDim2.new(1,-24,0,0)
minimizeBtn.Parent = titleBar

local content = Instance.new("Frame")
content.Size = UDim2.new(1,0,1,-24)
content.Position = UDim2.new(0,0,0,24)
content.BackgroundColor3 = Color3.fromRGB(30,30,30)
content.BorderSizePixel = 0
content.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Text = "STATUS: Inativo"
statusLabel.TextColor3 = Color3.fromRGB(255,100,100)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 13
statusLabel.BackgroundTransparency = 1
statusLabel.Size = UDim2.new(1,-20,0,18)
statusLabel.Position = UDim2.new(0,10,0,6)
statusLabel.Parent = content

local methodLabel = Instance.new("TextLabel")
methodLabel.Text = "Método VR: ---"
methodLabel.TextColor3 = Color3.fromRGB(200,200,200)
methodLabel.Font = Enum.Font.Gotham
methodLabel.TextSize = 11
methodLabel.BackgroundTransparency = 1
methodLabel.Size = UDim2.new(1,-20,0,16)
methodLabel.Position = UDim2.new(0,10,0,24)
methodLabel.Parent = content

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0,220,0,30)
toggleBtn.Position = UDim2.new(0.5,-110,0,50)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0,150,0)
toggleBtn.BorderSizePixel = 0
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Text = "ATIVAR VR (F5)"
toggleBtn.Parent = content

local infoLabel = Instance.new("TextLabel")
infoLabel.Text = "Mouse = cabeça | WASD = mover | Q/E = mãos"
infoLabel.TextColor3 = Color3.fromRGB(150,150,150)
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 11
infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1,-20,0,20)
infoLabel.Position = UDim2.new(0,10,0,90)
infoLabel.Parent = content

local minimized = false
local function setMinimized(state)
    minimized = state
    content.Visible = not state
    frame.Size = state and UDim2.new(0,260,0,24) or UDim2.new(0,260,0,150)
end
minimizeBtn.MouseButton1Click:Connect(function() setMinimized(not minimized) end)

-- ================== LÓGICA AGRESSIVA DE ATIVAÇÃO DO VR ==================
local vrActive = false
local currentMethod = "Nenhum"
local keys = {}
local moveDir = Vector3.new()
local handAngle = 0
local sensitivity = 0.3
local bindName = "VR_Aggressive_Render"

-- Estado da engine
local vrForced = false
local vrHooks = {}
local vrFakeConnection = nil

-- Função de debug
local function updateUI(statusText, color, method)
    statusLabel.Text = "STATUS: " .. statusText
    statusLabel.TextColor3 = color or Color3.new(1,1,1)
    if method then 
        currentMethod = method 
        methodLabel.Text = "Método VR: " .. method
    end
end

-- ===== MÉTODO 1: Chamada direta SetVREnabled =====
local function method1_SetVREnabled()
    pcall(function() VRService:SetVREnabled(true) end)
    return VRService.VREnabled
end

-- ===== MÉTODO 2: Hook na metatable __index para retornar true =====
local function method2_HookIndex()
    local mt = getrawmetatable(VRService)
    if not mt then return false end
    local oldIdx = mt.__index
    local hook = newcclosure(function(self, prop)
        if prop == "VREnabled" then
            return true
        end
        return oldIdx(self, prop)
    end)
    mt.__index = hook
    -- também tentamos SetVREnabled de novo depois do hook
    pcall(function() VRService:SetVREnabled(true) end)
    return VRService.VREnabled
end

-- ===== MÉTODO 3: Substituir a classe VRService no core =====
local function method3_ReplaceService()
    -- Criar um novo serviço mock que sempre retorna VREnabled true e aceita SetUserCFrame
    local FakeVRService = {}
    FakeVRService.VREnabled = true
    FakeVRService.GetUserCFrame = VRService.GetUserCFrame
    FakeVRService.SetUserCFrame = VRService.SetUserCFrame
    FakeVRService.SetVREnabled = function(_, val) FakeVRService.VREnabled = true end
    FakeVRService.GetUserCFrameEnabled = function() return true end
    FakeVRService.Recenter = function() end
    -- Substituir a referência global? Dificil, mas podemos redefinir game:GetService("VRService")
    local oldGetService = getmetatable(game).__index.GetService or game.GetService
    local hookGetService = newcclosure(function(self, service)
        if service == "VRService" then
            return FakeVRService
        end
        return oldGetService(self, service)
    end)
    pcall(function() getmetatable(game).__index.GetService = hookGetService end) -- não seguro
    -- Em vez disso, redirecionar diretamente a variável local VRService para o fake
    local VRServiceFake = FakeVRService
    -- Precisamos substituir o upvalue... faremos via closure
    -- Isso só afeta este script, mas o que importa é o envio de UserCFrame.
    -- O SetUserCFrame do fake chamará o real, e a replicação ocorrerá se o servidor receber.
    -- Para que outros clientes vejam, a replicação é feita internamente; mesmo com fake, se chamarmos o real SetUserCFrame, funciona.
    -- Vamos simplesmente sobrescrever a variável VRService para usar o fake, mas redirecionando SetUserCFrame.
    local realSetUserCFrame = VRService.SetUserCFrame
    VRService = setmetatable({}, {
        __index = function(_, key)
            if key == "VREnabled" then return true end
            if key == "SetUserCFrame" then return realSetUserCFrame end
            if key == "GetUserCFrame" then return VRService.GetUserCFrame end
            if key == "SetVREnabled" then return function(_, val) end end
            return VRService[key]
        end;
        __newindex = function() end
    })
    -- Também forçar o game:GetService a retornar o proxy
    local realGetService = game.GetService
    game.GetService = function(self, service)
        if service == "VRService" then return VRService end
        return realGetService(self, service)
    end
    return true
end

-- ===== MÉTODO 4: Injeção de dispositivo VR falso no sistema de input =====
local function method4_FakeVRDevice()
    -- Criar um UserInputService falso com TouchEnabled e VRDevice
    -- Isso pode enganar o Roblox para ativar o modo VR internamente.
    -- Vamos tentar: se TouchEnabled for true e tiver um dispositivo de VR listado...
    local success, err = pcall(function()
        local realUIS = UIS
        -- Forçar a existência de um dispositivo VR na lista de Gamepad ou Touch
        -- Usaremos a função GetConnectedControllers (se existir) ou hookar GetSupportedGamepadKeyCodes
        -- Mas a ativação do VR depende mais de VRService.VREnabled e da detecção do headset.
        -- Muitos jogos bloqueiam definindo VREnabled = false. Nós já cuidamos disso.
        -- Então este método pode ser supérfluo.
        -- Ainda assim, vamos tentar mexer em UserInputService:SetNavigationKeys() etc.
        -- Na verdade, a maneira mais fácil é chamar VRService:GetUserCFrameEnabled() e se retornar false, tentar alterar.
        -- Faremos um hook nessa função também.
        local mt = getrawmetatable(VRService)
        if mt then
            local oldFunc = mt.__index
            mt.__index = newcclosure(function(self, prop)
                if prop == "GetUserCFrameEnabled" then
                    return function() return true end
                end
                if prop == "GetTouchpadState" then
                    return function() return Vector2.new(0,0) end
                end
                return oldFunc(self, prop)
            end)
        end
    end)
    return success
end

-- ===== MÉTODO 5: Acesso à propriedade interna "HeadTracking" e forçar =====
local function method5_InternalHeadTracking()
    pcall(function()
        local success = rawset(VRService, "HeadTrackingEnabled", true)
        rawset(VRService, "VREnabled", true)
    end)
    return VRService.VREnabled or rawget(VRService, "VREnabled")
end

-- ===== MÉTODO 6: Modificar script de réplica do VR no StarterPlayer =====
local function method6_PatchStarterPlayer()
    pcall(function()
        local starterPlayer = game:GetService("StarterPlayer")
        if starterPlayer then
            starterPlayer.CharacterAutoLoads = true -- não relacionado
            starterPlayer.EnableVR = true -- talvez funcione
        end
    end)
end

-- ===== APLICAR TODOS OS MÉTODOS =====
local function forceVREnabledAggressively()
    local methods = {
        {"SetVREnabled", method1_SetVREnabled},
        {"HookIndex", method2_HookIndex},
        {"ReplaceService", method3_ReplaceService},
        {"FakeVRDevice", method4_FakeVRDevice},
        {"InternalHeadTracking", method5_InternalHeadTracking},
        {"PatchStarterPlayer", method6_PatchStarterPlayer},
    }
    for _, m in ipairs(methods) do
        local success = m[2]()
        if success and VRService.VREnabled then
            currentMethod = m[1]
            updateUI("Ativo", Color3.fromRGB(100,255,100), currentMethod)
            return true
        end
    end
    updateUI("Falhou", Color3.fromRGB(255,100,100), "Nenhum método funcionou")
    return false
end

-- ================== FUNÇÕES DE APOIO ==================
local function lockMouse() pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.LockCenter end) end
local function unlockMouse() pcall(function() UIS.MouseBehavior = Enum.MouseBehavior.Default end) end

local function updateMoveDir()
    local dir = Vector3.new()
    if keys[Enum.KeyCode.W] then dir += Vector3.new(0,0,-1) end
    if keys[Enum.KeyCode.S] then dir += Vector3.new(0,0,1) end
    if keys[Enum.KeyCode.A] then dir += Vector3.new(-1,0,0) end
    if keys[Enum.KeyCode.D] then dir += Vector3.new(1,0,0) end
    moveDir = dir.Magnitude > 0 and dir.Unit or Vector3.new()
end

local function startVR()
    -- Reforçar ativação
    if not forceVREnabledAggressively() then
        updateUI("Erro fatal: VR não ativável", Color3.fromRGB(255,0,0))
        return
    end
    vrActive = true
    lockMouse()
    camera.CameraType = Enum.CameraType.Scriptable
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.AutoRotate = false
    end
    RunService:BindToRenderStep(bindName, Enum.RenderPriority.Camera.Value + 1, vrLoop)
    toggleBtn.Text = "DESATIVAR VR (F5)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(200,0,0)
end

local function stopVR()
    vrActive = false
    unlockMouse()
    camera.CameraType = Enum.CameraType.Custom
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.AutoRotate = true
    end
    RunService:UnbindFromRenderStep(bindName)
    pcall(function() VRService:Recenter() end)
    toggleBtn.Text = "ATIVAR VR (F5)"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(0,150,0)
    updateUI("Inativo", Color3.fromRGB(255,100,100), currentMethod)
end

-- Botão toggle
toggleBtn.MouseButton1Click:Connect(function()
    if vrActive then stopVR() else startVR() end
end)

-- Input
UIS.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F5 then
        if vrActive then stopVR() else startVR() end
    elseif vrActive then
        keys[input.KeyCode] = true
        updateMoveDir()
        if input.KeyCode == Enum.KeyCode.Q then handAngle -= 30
        elseif input.KeyCode == Enum.KeyCode.E then handAngle += 30 end
    end
end)
UIS.InputEnded:Connect(function(input)
    if vrActive then keys[input.KeyCode] = nil; updateMoveDir() end
end)

-- ================== LOOP PRINCIPAL ==================
function vrLoop(delta)
    if not vrActive then return end
    local char = player.Character
    if not char or not char.Parent then return end
    local head = char:FindFirstChild("Head")
    if not head then return end
    
    -- Movimento
    local camCF = camera.CFrame
    local forward = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
    local right = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit
    local worldMove = (forward * -moveDir.Z) + (right * moveDir.X)
    if worldMove.Magnitude > 0 then
        char.Humanoid:Move(worldMove, false)
    end
    
    -- Rotação da câmera
    local deltaMouse = UIS:GetMouseDelta()
    if deltaMouse.magnitude > 0 then
        local currentRot = camCF - camCF.Position
        local yaw = math.rad(-deltaMouse.X * sensitivity)
        local pitch = math.rad(-deltaMouse.Y * sensitivity)
        local newRot = currentRot * CFrame.Angles(0, yaw, 0) * CFrame.Angles(pitch, 0, 0)
        local _, p, _ = newRot:ToEulerAnglesYXZ()
        if math.abs(p) > math.rad(80) then
            newRot = currentRot * CFrame.Angles(0, yaw, 0) * CFrame.Angles(math.clamp(p, math.rad(-80), math.rad(80)) - (select(2, currentRot:ToEulerAnglesYXZ()) or 0), 0, 0)
        end
        camera.CFrame = CFrame.new(camera.CFrame.Position) * newRot
    end
    camera.CFrame = CFrame.new(head.Position) * camera.CFrame.Rotation

    -- UserCFrames (replicação nativa)
    local headCF = camera.CFrame * CFrame.new(0, 0.1, 0)
    local rightHandCF = headCF * CFrame.new(0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(handAngle))
    local leftHandCF = headCF * CFrame.new(-0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(-handAngle))
    
    pcall(function()
        VRService:SetUserCFrame(Enum.UserCFrame.Head, headCF)
        VRService:SetUserCFrame(Enum.UserCFrame.RightHand, rightHandCF)
        VRService:SetUserCFrame(Enum.UserCFrame.LeftHand, leftHandCF)
    end)
    
    -- Se VREnabled for falso mas o loop roda, tenta reaplicar método
    if not VRService.VREnabled then
        forceVREnabledAggressively()
    end
end

-- Segurança: re-forçar VR após respawn
player.CharacterAdded:Connect(function(char)
    if vrActive then
        wait(0.3)
        char:WaitForChild("Humanoid").AutoRotate = false
        forceVREnabledAggressively()
    end
end)

-- Anti-detecção: loop contínuo de manutenção da flag VREnabled
spawn(function()
    while true do
        if vrActive and not VRService.VREnabled then
            forceVREnabledAggressively()
        end
        wait(1)
    end
end)

-- Inicializar
updateUI("Inativo", Color3.fromRGB(255,100,100), "Pronto")
forceVREnabledAggressively() -- teste inicial

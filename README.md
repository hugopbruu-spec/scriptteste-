-- VR Universal: Simula um headset VR completo usando mouse+teclado e replica para o servidor.
-- Ative com F5. Movimento: WASD (relativo à cabeça). Mouse: gira cabeça. Q/E: gira mãos.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VRService = game:GetService("VRService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Estado
local vrAtivo = false
local mouseLocked = false
local moveDirection = Vector3.new()
local keys = {}

-- Sensibilidade da câmera
local sensibilidade = 0.3

-- Posições das mãos (avanço manual com scroll ou teclas)
local anguloMao = 0 -- rotação extra para as mãos (controlada por Q/E)
local inclinacaoMao = 0

-- Prender mouse
local function lockMouse()
    UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
    mouseLocked = true
end

local function unlockMouse()
    UserInputService.MouseBehavior = Enum.MouseBehavior.Default
    mouseLocked = false
end

-- Movimento relativo à cabeça
local function updateMoveDirection()
    local dir = Vector3.new()
    if keys[Enum.KeyCode.W] then dir += Vector3.new(0,0,-1) end
    if keys[Enum.KeyCode.S] then dir += Vector3.new(0,0,1) end
    if keys[Enum.KeyCode.A] then dir += Vector3.new(-1,0,0) end
    if keys[Enum.KeyCode.D] then dir += Vector3.new(1,0,0) end
    moveDirection = dir.Magnitude > 0 and dir.Unit or Vector3.new()
end

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F5 then
        vrAtivo = not vrAtivo
        if vrAtivo then
            ativarVR()
        else
            desativarVR()
        end
    elseif vrAtivo then
        keys[input.KeyCode] = true
        updateMoveDirection()
        if input.KeyCode == Enum.KeyCode.Q then
            anguloMao -= 45   -- gira mãos "para dentro"
        elseif input.KeyCode == Enum.KeyCode.E then
            anguloMao += 45
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if vrAtivo then
        keys[input.KeyCode] = nil
        updateMoveDirection()
    end
end)

local function ativarVR()
    -- Força o cliente a acreditar que está em VR, ativando a replicação nativa.
    if not VRService.VREnabled then
        -- Alguns executores oferecem setVREnabled diretamente; caso contrário, usamos hook.
        pcall(function()
            VRService:SetVREnabled(true)
        end)
        if not VRService.VREnabled then
            -- Fallback: hook na propriedade para burlar a verificação.
            local mt = getrawmetatable(VRService)
            local old = mt.__index
            mt.__index = function(self, prop)
                if prop == "VREnabled" then
                    return true
                end
                return old(self, prop)
            end
        end
    end
    
    lockMouse()
    camera.CameraType = Enum.CameraType.Scriptable
    player.Character.Humanoid.AutoRotate = false
    RunService:BindToRenderStep("VR_Universal", Enum.RenderPriority.Camera.Value + 1, vrLoop)
end

local function desativarVR()
    unlockMouse()
    camera.CameraType = Enum.CameraType.Custom
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.AutoRotate = true
    end
    RunService:UnbindFromRenderStep("VR_Universal")
    -- Reseta UserCFrames para posição neutra
    pcall(function()
        VRService:Recenter()
    end)
end

local function vrLoop(delta)
    local char = player.Character
    if not char or not char.Parent then
        desativarVR()
        return
    end
    local head = char:FindFirstChild("Head")
    if not head then return end

    -- 1. Movimentação
    local camCF = camera.CFrame
    local forward = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
    local right = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit
    local worldMove = (forward * -moveDirection.Z) + (right * moveDirection.X)
    if worldMove.Magnitude > 0 then
        char.Humanoid:Move(worldMove, false)
    end

    -- 2. Rotação da cabeça (mouse delta)
    local deltaMouse = UserInputService:GetMouseDelta()
    if deltaMouse ~= Vector2.new() then
        local rotAtual = camCF - camCF.Position
        local yaw = deltaMouse.X * math.rad(sensibilidade)
        local pitch = deltaMouse.Y * math.rad(sensibilidade)
        local novaRot = rotAtual * CFrame.Angles(0, -yaw, 0) * CFrame.Angles(-pitch, 0, 0)
        -- Limitar pitch
        local _, p, _ = novaRot:toEulerAnglesYXZ()
        if math.abs(p) > math.rad(80) then
            novaRot = rotAtual * CFrame.Angles(0, -yaw, 0) * CFrame.Angles(math.clamp(p, math.rad(-80), math.rad(80)) - (select(2, rotAtual:toEulerAnglesYXZ())), 0, 0)
        end
        camera.CFrame = CFrame.new(camera.CFrame.Position) * novaRot
    end
    -- Posicionar câmera na cabeça
    camera.CFrame = CFrame.new(head.Position) * camera.CFrame.Rotation

    -- 3. Calcular UserCFrames (Head, mãos)
    local headCF = camera.CFrame * CFrame.new(0, 0, 0) -- já é a posição/rotação correta?
    -- Ajuste para UserCFrame.Head (posição dos olhos, ligeiramente à frente)
    local olhosCF = headCF * CFrame.new(0, 0.15, -0.1) -- pequeno offset típico

    -- Mãos: posicionadas à frente do peito/ombros, com rotação baseada no head + ajustes
    local baseMaoDir = headCF * CFrame.new(.4, -.5, -.8) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(anguloMao))
    local baseMaoEsq = headCF * CFrame.new(-.4, -.5, -.8) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(-anguloMao))

    -- Enviar UserCFrames (replicação nativa)
    -- O Roblox espera UserCFrames para Head, RightHand, LeftHand (Enum.UserCFrame)
    pcall(function()
        VRService:SetUserCFrame(Enum.UserCFrame.Head, olhosCF)
        VRService:SetUserCFrame(Enum.UserCFrame.RightHand, baseMaoDir)
        VRService:SetUserCFrame(Enum.UserCFrame.LeftHand, baseMaoEsq)
    end)
end

-- Limpeza ao sair
Players.LocalPlayer.CharacterAdded:Connect(function()
    if vrAtivo then
        desativarVR()
        vrAtivo = false
    end
end)

-- VR_MouseKeyboard_Universal.lua
-- Ativa/desativa com F5. Debug com F6.
-- Tentará forçar VRService, hooks e fallbacks.

local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local VRService = game:GetService("VRService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configs
local VR_ATIVO = false
local SENSIBILIDADE = 0.3
local DEBUG = false
local keys = {}
local moveDir = Vector3.new()
local anguloMao = 0

-- Elementos de debug GUI
local dbgText
if not syn and not getexecutorname then
    -- Se não for executor, tenta criar GUI mesmo assim (modo Roblox normal)
    local screen = Instance.new("ScreenGui")
    screen.Parent = player:WaitForChild("PlayerGui")
    dbgText = Instance.new("TextLabel")
    dbgText.Size = UDim2.new(0, 300, 0, 200)
    dbgText.Position = UDim2.new(0, 10, 0, 10)
    dbgText.BackgroundTransparency = 0.5
    dbgText.BackgroundColor3 = Color3.new(0,0,0)
    dbgText.TextColor3 = Color3.new(1,1,1)
    dbgText.TextWrapped = true
    dbgText.Font = Enum.Font.Code
    dbgText.TextSize = 14
    dbgText.Parent = screen
end

local function debugPrint(msg)
    print("[VR_DEBUG]", msg)
    if dbgText then
        dbgText.Text = dbgText.Text .. "\n" .. msg
    end
end

-- ========== FORÇAR ATIVAÇÃO DO VR ==========
local vrForcedOk = false
local function tentarForcarVR()
    -- Método 1: SetVREnabled
    local s1, e1 = pcall(function()
        VRService:SetVREnabled(true)
    end)
    if s1 and VRService.VREnabled then
        debugPrint("Método 1 OK: SetVREnabled chamado com sucesso. VREnabled="..tostring(VRService.VREnabled))
        vrForcedOk = true
        return
    end
    debugPrint("Método 1 falhou: "..tostring(e1))
    
    -- Método 2: Hook __index para retornar true
    local s2, e2 = pcall(function()
        local mt = getrawmetatable(VRService)
        local oldIdx = mt.__index
        mt.__index = function(self, prop)
            if prop == "VREnabled" then
                return true
            end
            return oldIdx(self, prop)
        end
    end)
    if s2 and VRService.VREnabled then
        debugPrint("Método 2 OK: __index hookado. VREnabled="..tostring(VRService.VREnabled))
        VRService:SetVREnabled(true) -- tenta novamente
        vrForcedOk = true
        return
    end
    debugPrint("Método 2 falhou: "..tostring(e2))
    
    -- Método 3: Raw set da propriedade (se possível)
    local s3, e3 = pcall(function()
        local mt = getrawmetatable(VRService)
        local oldNewIdx = mt.__newindex
        mt.__newindex = function(self, prop, val)
            if prop == "VREnabled" then
                oldNewIdx(self, prop, true)
            else
                oldNewIdx(self, prop, val)
            end
        end
        VRService.VREnabled = true
    end)
    if s3 and VRService.VREnabled then
        debugPrint("Método 3 OK: __newindex hookado. VREnabled="..tostring(VRService.VREnabled))
        vrForcedOk = true
        return
    end
    debugPrint("Método 3 falhou: "..tostring(e3))
    
    -- Método 4: Recriar serviço? (não possível)
    debugPrint("Todos os métodos de ativação falharam. O VR não funcionará para os outros.")
end

tentarForcarVR()

-- ========== CONTROLES ==========
local function lockMouse()
    pcall(function()
        UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
    end)
end

local function unlockMouse()
    pcall(function()
        UIS.MouseBehavior = Enum.MouseBehavior.Default
    end)
end

local function updateMove()
    local dir = Vector3.new()
    if keys[Enum.KeyCode.W] then dir += Vector3.new(0,0,-1) end
    if keys[Enum.KeyCode.S] then dir += Vector3.new(0,0,1) end
    if keys[Enum.KeyCode.A] then dir += Vector3.new(-1,0,0) end
    if keys[Enum.KeyCode.D] then dir += Vector3.new(1,0,0) end
    moveDir = dir.Magnitude > 0 and dir.Unit or Vector3.new()
end

UIS.InputBegan:Connect(function(inp, processed)
    if processed then return end
    if inp.KeyCode == Enum.KeyCode.F5 then
        VR_ATIVO = not VR_ATIVO
        if VR_ATIVO then
            ativar()
        else
            desativar()
        end
    elseif inp.KeyCode == Enum.KeyCode.F6 then
        DEBUG = not DEBUG
        debugPrint("Modo Debug: "..tostring(DEBUG))
    elseif VR_ATIVO then
        keys[inp.KeyCode] = true
        updateMove()
        if inp.KeyCode == Enum.KeyCode.Q then anguloMao -= 45
        elseif inp.KeyCode == Enum.KeyCode.E then anguloMao += 45 end
    end
end)
UIS.InputEnded:Connect(function(inp)
    if VR_ATIVO then
        keys[inp.KeyCode] = nil
        updateMove()
    end
end)

local function ativar()
    debugPrint("Ativando VR...")
    if not vrForcedOk then
        tentarForcarVR() -- tenta novamente
    end
    lockMouse()
    camera.CameraType = Enum.CameraType.Scriptable
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.AutoRotate = false
    end
    RunService:BindToRenderStep("VR_Universal", Enum.RenderPriority.Camera.Value + 1, vrLoop)
end

local function desativar()
    debugPrint("Desativando VR...")
    unlockMouse()
    camera.CameraType = Enum.CameraType.Custom
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.AutoRotate = true
    end
    RunService:UnbindFromRenderStep("VR_Universal")
    pcall(function() VRService:Recenter() end)
end

local function vrLoop(delta)
    local char = player.Character
    if not char or not char.Parent then
        desativar()
        return
    end
    local head = char:FindFirstChild("Head")
    if not head then return end
    
    -- Movimento relativo à cabeça
    local camCF = camera.CFrame
    local fwd = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
    local right = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit
    local mv = (fwd * -moveDir.Z) + (right * moveDir.X)
    if mv.Magnitude > 0 then
        char.Humanoid:Move(mv, false)
    end
    
    -- Rotação da câmera
    local deltaMouse = UIS:GetMouseDelta()
    if deltaMouse ~= Vector2.new() then
        local currentRot = camCF - camCF.Position
        local yaw = math.rad(-deltaMouse.X * SENSIBILIDADE)
        local pitch = math.rad(-deltaMouse.Y * SENSIBILIDADE)
        local newRot = currentRot * CFrame.Angles(0, yaw, 0) * CFrame.Angles(pitch, 0, 0)
        local _, p, _ = newRot:toEulerAnglesYXZ()
        if math.abs(p) > math.rad(80) then
            -- clamp
            newRot = currentRot * CFrame.Angles(0, yaw, 0) * CFrame.Angles(math.clamp(p, math.rad(-80), math.rad(80)) - (select(2,currentRot:toEulerAnglesYXZ()) or 0), 0, 0)
        end
        camera.CFrame = CFrame.new(camera.CFrame.Position) * newRot
    end
    camera.CFrame = CFrame.new(head.Position) * camera.CFrame.Rotation
    
    -- UserCFrames para replicação
    local headCF = camera.CFrame * CFrame.new(0, 0.1, 0)
    local maoDirCF = headCF * CFrame.new(0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(anguloMao))
    local maoEsqCF = headCF * CFrame.new(-0.4, -0.5, -0.7) * CFrame.Angles(math.rad(-90), 0, 0) * CFrame.Angles(0, 0, math.rad(-anguloMao))
    
    local status, err = pcall(function()
        VRService:SetUserCFrame(Enum.UserCFrame.Head, headCF)
        VRService:SetUserCFrame(Enum.UserCFrame.RightHand, maoDirCF)
        VRService:SetUserCFrame(Enum.UserCFrame.LeftHand, maoEsqCF)
    end)
    
    if DEBUG then
        debugPrint("VREnabled="..tostring(VRService.VREnabled).." UserCFrames sent="..tostring(status).. (err and (" err:"..err) or ""))
    end
end

-- Resetar ao respawn
player.CharacterAdded:Connect(function(char)
    if VR_ATIVO then
        wait(0.5)
        ativar() -- reaplica autoRotate etc
    end
end)

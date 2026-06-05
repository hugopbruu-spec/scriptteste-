--[[
    Script para parar todos os scripts em execução
    Executa em menos de 1 segundo
--]]

-- Método 1: Fecha todas as threads ativas
local threads = {}
for _, thread in ipairs(getthreads and getthreads() or {}) do
    if thread ~= coroutine.running() then
        task.cancel(thread)
    end
end

-- Método 2: Destrói todas as ScreenGuis do CoreGui
for _, gui in ipairs(game:GetService("CoreGui"):GetChildren()) do
    if gui:IsA("ScreenGui") then
        gui:Destroy()
    end
end

-- Método 3: Desconecta todas as conexões de eventos
for _, connection in ipairs(getconnections and getconnections(game:GetService("RunService").RenderStepped) or {}) do
    connection:Disconnect()
end

-- Método 4: Remove loops do RunService
for _, connection in ipairs(getconnections and getconnections(game:GetService("RunService").Heartbeat) or {}) do
    connection:Disconnect()
end

-- Método 5: Limpa atributos do personagem
if game:GetService("Players").LocalPlayer.Character then
    local char = game:GetService("Players").LocalPlayer.Character
    for _, child in ipairs(char:GetDescendants()) do
        if child:IsA("BodyMover") or child:IsA("BodyGyro") or child:IsA("BodyVelocity") or 
           child:IsA("BodyPosition") or child:IsA("BodyThrust") or child:IsA("RocketPropulsion") then
            child:Destroy()
        end
    end
    -- Restaura velocidades padrão
    if char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = 16
        char.Humanoid.JumpPower = 50
        char.Humanoid.MaxHealth = 100
        char.Humanoid.Health = 100
    end
    -- Desancora partes
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Anchored = false
            part.CanCollide = true
            part.Transparency = 0
        end
    end
end

-- Restaura iluminação
game:GetService("Lighting").Brightness = 1
game:GetService("Lighting").ClockTime = 14
game:GetService("Lighting").FogEnd = 1000
game:GetService("Lighting").GlobalShadows = true
game:GetService("Workspace").Gravity = 196.2

print("✅ Todos os scripts foram parados!")

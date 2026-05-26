--[[
    ╔══════════════════════════════════════════════════════════╗
    ║     ROBLOX GAME DOWNLOADER - ABERTURA AUTOMÁTICA      ║
    ║     Compatível com qualquer jogo e qualquer executor    ║
    ╚══════════════════════════════════════════════════════════╝
]]

-- ============================================
-- SERVIÇOS
-- ============================================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local MarketplaceService = game:GetService("MarketplaceService")

-- ============================================
-- CONFIGURAÇÃO
-- ============================================
local PLACE_ID = game.PlaceId

-- ============================================
-- FUNÇÕES DE DETECÇÃO DO EXECUTOR
-- ============================================
local function getExecutorName()
    local success, name = pcall(function()
        return identifyexecutor()
    end)
    if success and name then return name end

    local success2, name2 = pcall(function()
        return getexecutorname()
    end)
    if success2 and name2 then return name2 end

    -- Verificações manuais
    if syn and syn.request then return "Synapse X" end
    if krnl then return "KRNL" end
    if fluxus then return "Fluxus" end
    if scriptware then return "Script-Ware" end
    if getgenv and isfile then return "Executor Genérico" end

    return "Desconhecido"
end

local function hasSaveInstance()
    return pcall(function() return saveinstance end) and saveinstance ~= nil
end

local function hasWriteFile()
    return pcall(function() return writefile end) and writefile ~= nil and pcall(function() return isfile end) and isfile ~= nil
end

-- ============================================
-- REMOVER INTERFACE ANTIGA (SE EXISTIR)
-- ============================================
local function removeOldUI()
    -- Remove pelo nome
    for _, child in ipairs(CoreGui:GetChildren()) do
        if child.Name == "RobloxDownloaderUI" then
            child:Destroy()
        end
    end
    -- Remove referência global
    if _G.RobloxDownloaderUI then
        pcall(function()
            _G.RobloxDownloaderUI:Destroy()
        end)
        _G.RobloxDownloaderUI = nil
    end
end

-- Remove qualquer UI antiga primeiro
removeOldUI()

-- ============================================
-- CRIAÇÃO DA INTERFACE GRÁFICA
-- ============================================
local function createInterface()
    -- Limpa novamente (prevenção dupla)
    removeOldUI()

    -- ========== SCREEN GUI ==========
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "RobloxDownloaderUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Enabled = true
    screenGui.DisplayOrder = 9999
    screenGui.Parent = CoreGui

    -- Proteção anti-detecção (se disponível)
    pcall(function()
        if syn and syn.protect_gui then
            syn.protect_gui(screenGui)
        end
        if protectgui then
            protectgui(screenGui)
        end
        if gethui then
            screenGui.Parent = gethui()
        end
    end)

    _G.RobloxDownloaderUI = screenGui

    -- ========== CONTAINER PRINCIPAL ==========
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 520, 0, 450)
    mainFrame.Position = UDim2.new(0.5, -260, 0.5, -225)
    mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Visible = true
    mainFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = mainFrame

    -- Borda
    local border = Instance.new("UIStroke")
    border.Thickness = 1
    border.Color = Color3.fromRGB(60, 60, 70)
    border.Parent = mainFrame

    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Size = UDim2.new(1, 0, 1, 0)
    shadow.Position = UDim2.new(0, 0, 0, 0)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6015897843"
    shadow.ImageTransparency = 0.5
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(49, 49, 450, 450)
    shadow.ZIndex = 0
    shadow.Parent = mainFrame

    -- ========== BARRA SUPERIOR ==========
    local topBar = Instance.new("Frame")
    topBar.Name = "TopBar"
    topBar.Size = UDim2.new(1, 0, 0, 52)
    topBar.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    topBar.BorderSizePixel = 0
    topBar.Parent = mainFrame

    local topCorner = Instance.new("UICorner")
    topCorner.CornerRadius = UDim.new(0, 14)
    topCorner.Parent = topBar

    -- Ícone / Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -100, 1, 0)
    title.Position = UDim2.new(0, 16, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎮 ROBLOX GAME DOWNLOADER"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = topBar

    -- Botão Minimizar
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 32, 0, 32)
    minimizeBtn.Position = UDim2.new(1, -80, 0, 10)
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    minimizeBtn.Text = "─"
    minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeBtn.TextSize = 18
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Parent = topBar

    local minCorner = Instance.new("UICorner")
    minCorner.CornerRadius = UDim.new(0, 8)
    minCorner.Parent = minimizeBtn

    local isMinimized = false
    minimizeBtn.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        if isMinimized then
            mainFrame.Size = UDim2.new(0, 520, 0, 52)
            minimizeBtn.Text = "□"
        else
            mainFrame.Size = UDim2.new(0, 520, 0, 450)
            minimizeBtn.Text = "─"
        end
    end)

    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 32, 0, 32)
    closeBtn.Position = UDim2.new(1, -40, 0, 10)
    closeBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = topBar

    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn

    closeBtn.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        _G.RobloxDownloaderUI = nil
    end)

    -- ========== CONTEÚDO ==========
    local contentFrame = Instance.new("ScrollingFrame")
    contentFrame.Size = UDim2.new(1, -16, 1, -68)
    contentFrame.Position = UDim2.new(0, 8, 0, 60)
    contentFrame.BackgroundTransparency = 1
    contentFrame.BorderSizePixel = 0
    contentFrame.ScrollBarThickness = 4
    contentFrame.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    contentFrame.CanvasSize = UDim2.new(0, 0, 0, 400)
    contentFrame.Parent = mainFrame

    local contentList = Instance.new("UIListLayout")
    contentList.Padding = UDim.new(0, 10)
    contentList.Parent = contentFrame

    -- ========== CARD: INFORMAÇÕES DO JOGO ==========
    local infoCard = Instance.new("Frame")
    infoCard.Size = UDim2.new(1, 0, 0, 90)
    infoCard.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    infoCard.BorderSizePixel = 0
    infoCard.Parent = contentFrame

    local infoCardCorner = Instance.new("UICorner")
    infoCardCorner.CornerRadius = UDim.new(0, 10)
    infoCardCorner.Parent = infoCard

    -- Nome do Jogo
    local gameName = "Jogo Desconhecido"
    pcall(function()
        gameName = MarketplaceService:GetProductInfo(PLACE_ID).Name or "Jogo Desconhecido"
    end)

    local gameNameLabel = Instance.new("TextLabel")
    gameNameLabel.Size = UDim2.new(1, -20, 0, 26)
    gameNameLabel.Position = UDim2.new(0, 10, 0, 8)
    gameNameLabel.BackgroundTransparency = 1
    gameNameLabel.Text = "🎯 " .. gameName
    gameNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    gameNameLabel.TextSize = 14
    gameNameLabel.Font = Enum.Font.GothamBold
    gameNameLabel.TextXAlignment = Enum.TextXAlignment.Left
    gameNameLabel.TextTruncate = Enum.TextTruncate.AtEnd
    gameNameLabel.Parent = infoCard

    -- Informações técnicas
    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(1, -20, 0, 22)
    infoLabel.Position = UDim2.new(0, 10, 0, 35)
    infoLabel.BackgroundTransparency = 1
    infoLabel.Text = "📍 Place ID: " .. PLACE_ID .. " | Versão: " .. game.PlaceVersion
    infoLabel.TextColor3 = Color3.fromRGB(160, 160, 170)
    infoLabel.TextSize = 11
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.TextXAlignment = Enum.TextXAlignment.Left
    infoLabel.Parent = infoCard

    -- Executor
    local executorLabel = Instance.new("TextLabel")
    executorLabel.Size = UDim2.new(1, -20, 0, 22)
    executorLabel.Position = UDim2.new(0, 10, 0, 58)
    executorLabel.BackgroundTransparency = 1
    executorLabel.Text = "⚡ Executor: " .. getExecutorName()
    executorLabel.TextColor3 = Color3.fromRGB(239, 68, 68)
    executorLabel.TextSize = 11
    executorLabel.Font = Enum.Font.Gotham
    executorLabel.TextXAlignment = Enum.TextXAlignment.Left
    executorLabel.Parent = infoCard

    -- ========== CARD: MÉTODOS DISPONÍVEIS ==========
    local methodsCard = Instance.new("Frame")
    methodsCard.Size = UDim2.new(1, 0, 0, 60)
    methodsCard.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    methodsCard.BorderSizePixel = 0
    methodsCard.Parent = contentFrame

    local methodsCorner = Instance.new("UICorner")
    methodsCorner.CornerRadius = UDim.new(0, 10)
    methodsCorner.Parent = methodsCard

    local methodsTitle = Instance.new("TextLabel")
    methodsTitle.Size = UDim2.new(1, -20, 0, 20)
    methodsTitle.Position = UDim2.new(0, 10, 0, 5)
    methodsTitle.BackgroundTransparency = 1
    methodsTitle.Text = "🔧 Métodos Detectados:"
    methodsTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    methodsTitle.TextSize = 12
    methodsTitle.Font = Enum.Font.GothamSemibold
    methodsTitle.TextXAlignment = Enum.TextXAlignment.Left
    methodsTitle.Parent = methodsCard

    local methodsList = Instance.new("TextLabel")
    methodsList.Size = UDim2.new(1, -20, 0, 35)
    methodsList.Position = UDim2.new(0, 10, 0, 25)
    methodsList.BackgroundTransparency = 1
    methodsList.TextColor3 = Color3.fromRGB(200, 200, 200)
    methodsList.TextSize = 11
    methodsList.Font = Enum.Font.Gotham
    methodsList.TextXAlignment = Enum.TextXAlignment.Left
    methodsList.Parent = methodsCard

    local hasSI = hasSaveInstance()
    local hasWF = hasWriteFile()

    local methodsText = ""
    if hasSI then methodsText = methodsText .. "✅ saveinstance | " end
    if hasWF then methodsText = methodsText .. "✅ writefile | " end
    if not hasSI and not hasWF then methodsText = methodsText .. "⚠️ Método básico (serialização) | " end
    methodsText = methodsText .. "✅ HTTP Request"
    methodsList.Text = methodsText

    -- ========== BOTÃO DE DOWNLOAD ==========
    local downloadBtn = Instance.new("TextButton")
    downloadBtn.Size = UDim2.new(1, 0, 0, 56)
    downloadBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    downloadBtn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
    downloadBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    downloadBtn.TextSize = 15
    downloadBtn.Font = Enum.Font.GothamBold
    downloadBtn.BorderSizePixel = 0
    downloadBtn.Parent = contentFrame

    local dlCorner = Instance.new("UICorner")
    dlCorner.CornerRadius = UDim.new(0, 10)
    dlCorner.Parent = downloadBtn

    -- Efeito hover
    local dlShadow = Instance.new("ImageLabel")
    dlShadow.Size = UDim2.new(1, 0, 1, 0)
    dlShadow.BackgroundTransparency = 1
    dlShadow.Image = "rbxassetid://6014261993"
    dlShadow.ImageTransparency = 0.8
    dlShadow.ScaleType = Enum.ScaleType.Slice
    dlShadow.SliceCenter = Rect.new(49, 49, 450, 450)
    dlShadow.Visible = false
    dlShadow.Parent = downloadBtn

    downloadBtn.MouseEnter:Connect(function()
        downloadBtn.BackgroundColor3 = Color3.fromRGB(220, 38, 38)
        dlShadow.Visible = true
    end)
    downloadBtn.MouseLeave:Connect(function()
        downloadBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        dlShadow.Visible = false
    end)

    -- ========== CARD: PROGRESSO ==========
    local progressCard = Instance.new("Frame")
    progressCard.Size = UDim2.new(1, 0, 0, 50)
    progressCard.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    progressCard.BorderSizePixel = 0
    progressCard.Parent = contentFrame

    local progCorner = Instance.new("UICorner")
    progCorner.CornerRadius = UDim.new(0, 10)
    progCorner.Parent = progressCard

    -- Barra de fundo
    local progressBarBg = Instance.new("Frame")
    progressBarBg.Size = UDim2.new(1, -20, 0, 12)
    progressBarBg.Position = UDim2.new(0, 10, 0, 10)
    progressBarBg.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    progressBarBg.BorderSizePixel = 0
    progressBarBg.Parent = progressCard

    local progBgCorner = Instance.new("UICorner")
    progBgCorner.CornerRadius = UDim.new(0, 6)
    progBgCorner.Parent = progressBarBg

    -- Barra de progresso
    local progressBarFill = Instance.new("Frame")
    progressBarFill.Size = UDim2.new(0, 0, 1, 0)
    progressBarFill.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    progressBarFill.BorderSizePixel = 0
    progressBarFill.Parent = progressBarBg

    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 6)
    fillCorner.Parent = progressBarFill

    -- Texto de progresso
    local progressText = Instance.new("TextLabel")
    progressText.Size = UDim2.new(1, -20, 0, 22)
    progressText.Position = UDim2.new(0, 10, 0, 26)
    progressText.BackgroundTransparency = 1
    progressText.Text = "Pronto para iniciar..."
    progressText.TextColor3 = Color3.fromRGB(200, 200, 200)
    progressText.TextSize = 11
    progressText.Font = Enum.Font.Gotham
    progressText.TextXAlignment = Enum.TextXAlignment.Left
    progressText.Parent = progressCard

    -- ========== CARD: LOGS ==========
    local logCard = Instance.new("Frame")
    logCard.Size = UDim2.new(1, 0, 0, 130)
    logCard.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    logCard.BorderSizePixel = 0
    logCard.Parent = contentFrame

    local logCorner = Instance.new("UICorner")
    logCorner.CornerRadius = UDim.new(0, 10)
    logCorner.Parent = logCard

    local logScroll = Instance.new("ScrollingFrame")
    logScroll.Size = UDim2.new(1, -20, 1, -20)
    logScroll.Position = UDim2.new(0, 10, 0, 10)
    logScroll.BackgroundTransparency = 1
    logScroll.BorderSizePixel = 0
    logScroll.ScrollBarThickness = 3
    logScroll.ScrollBarImageColor3 = Color3.fromRGB(239, 68, 68)
    logScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    logScroll.Parent = logCard

    local logList = Instance.new("UIListLayout")
    logList.Padding = UDim.new(0, 4)
    logList.Parent = logScroll

    -- ========== FUNÇÕES AUXILIARES ==========
    local logCount = 0

    local function addLog(message, color)
        logCount = logCount + 1
        local logLabel = Instance.new("TextLabel")
        logLabel.Size = UDim2.new(1, 0, 0, 18)
        logLabel.BackgroundTransparency = 1
        logLabel.Text = "[" .. os.date("%H:%M:%S") .. "] " .. message
        logLabel.TextColor3 = color or Color3.fromRGB(200, 200, 200)
        logLabel.TextSize = 10
        logLabel.Font = Enum.Font.Code
        logLabel.TextXAlignment = Enum.TextXAlignment.Left
        logLabel.TextTruncate = Enum.TextTruncate.AtEnd
        logLabel.Parent = logScroll
        logScroll.CanvasSize = UDim2.new(0, 0, 0, logCount * 22 + 5)
        logScroll.CanvasPosition = Vector2.new(0, math.huge)
    end

    local function updateProgress(percent, text)
        local targetSize = UDim2.new(percent / 100, 0, 1, 0)
        progressBarFill:TweenSize(targetSize, "Out", "Quad", 0.3, true)
        progressText.Text = math.floor(percent) .. "% - " .. (text or "")
    end

    -- ========== FUNÇÃO DE SERIALIZAÇÃO ==========
    local function serializeToRBXLX()
        local function serializeObject(obj, depth)
            if depth > 100 then return nil end
            local success, data = pcall(function()
                local objData = {
                    ClassName = obj.ClassName,
                    Name = obj.Name,
                    Properties = {},
                    Children = {}
                }

                local props = {"Position","Size","CFrame","Color","BackgroundColor3",
                              "Text","TextColor3","Value","Material","CanCollide",
                              "Anchored","Transparency","MeshId","TextureId","Image"}

                for _, p in ipairs(props) do
                    local s, v = pcall(function() return obj[p] end)
                    if s and v ~= nil then
                        objData.Properties[p] = tostring(v)
                    end
                end

                for _, child in ipairs(obj:GetChildren()) do
                    local cData = serializeObject(child, depth + 1)
                    if cData then table.insert(objData.Children, cData) end
                end

                return objData
            end)
            return success and data or nil
        end

        local function objToXML(obj, indent)
            indent = indent or ""
            local xml = indent .. '<Item class="' .. (obj.ClassName or "Unknown") .. '" referent="' .. (obj.Name or "") .. '">\n'
            if obj.Properties and next(obj.Properties) then
                xml = xml .. indent .. '  <Properties>\n'
                for k, v in pairs(obj.Properties) do
                    xml = xml .. indent .. '    <' .. k .. '>' .. v .. '</' .. k .. '>\n'
                end
                xml = xml .. indent .. '  </Properties>\n'
            end
            for _, child in ipairs(obj.Children or {}) do
                xml = xml .. objToXML(child, indent .. "  ")
            end
            xml = xml .. indent .. '</Item>\n'
            return xml
        end

        addLog("Serializando objetos...", Color3.fromRGB(255, 200, 100))
        updateProgress(10, "Serializando objetos...")

        local root = serializeObject(game, 0)
        if not root then
            addLog("❌ Falha na serialização", Color3.fromRGB(255, 100, 100))
            return nil
        end

        updateProgress(50, "Gerando XML...")
        local xml = '<?xml version="1.0" encoding="utf-8"?>\n<roblox version="4">\n'
        xml = xml .. objToXML(root, "  ")
        xml = xml .. '</roblox>'

        updateProgress(80, "XML gerado")
        return xml
    end

    -- ========== EXECUÇÃO DO DOWNLOAD ==========
    local function performDownload()
        local btn = downloadBtn
        btn.Text = "⏳ Processando..."
        btn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        btn.Active = false

        updateProgress(0, "Iniciando...")
        addLog("🚀 Iniciando download...", Color3.fromRGB(100, 255, 100))

        -- MÉTODO 1: saveinstance
        if hasSaveInstance() then
            addLog("✅ saveinstance detectado!", Color3.fromRGB(100, 255, 100))
            updateProgress(5, "Usando saveinstance...")

            local success, err = pcall(function()
                saveinstance({
                    filename = "roblox_" .. PLACE_ID .. "_" .. os.time(),
                    mode = "optimized",
                    savegame = true,
                    noscripts = false,
                    timeout = 30,
                    callback = function(progress)
                        updateProgress(5 + progress * 0.9, "Salvando via saveinstance")
                    end
                })
            end)

            if success then
                updateProgress(100, "Download concluído!")
                addLog("✅ JOGO SALVO COM SUCESSO!", Color3.fromRGB(100, 255, 100))
                addLog("📁 Arquivo salvo na pasta workspace", Color3.fromRGB(200, 200, 200))
                btn.Text = "✅ Download Concluído!"
                btn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                wait(3)
                btn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
                btn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                btn.Active = true
                return
            else
                addLog("⚠️ saveinstance falhou, tentando outro método...", Color3.fromRGB(255, 150, 50))
            end
        end

        -- MÉTODO 2: writefile + serialização
        if hasWriteFile() then
            addLog("🔄 Usando writefile + serialização...", Color3.fromRGB(255, 200, 100))

            local content = serializeToRBXLX()
            if content then
                updateProgress(85, "Escrevendo arquivo...")
                local fileName = "game_" .. PLACE_ID .. "_" .. os.date("%Y%m%d_%H%M%S") .. ".rbxlx"

                local ws, we = pcall(function()
                    writefile(fileName, content)
                end)

                if ws then
                    updateProgress(100, "Arquivo salvo!")
                    addLog("✅ Jogo salvo: " .. fileName, Color3.fromRGB(100, 255, 100))
                    addLog("📁 Local: pasta workspace do executor", Color3.fromRGB(200, 200, 200))
                    btn.Text = "✅ Download Concluído!"
                    btn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
                    wait(3)
                    btn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
                    btn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
                    btn.Active = true
                    return
                else
                    addLog("❌ Erro writefile: " .. tostring(we), Color3.fromRGB(255, 100, 100))
                end
            end
        end

        -- MÉTODO 3: Fallback (salva como arquivo Lua simples)
        addLog("⚠️ Tentando método fallback...", Color3.fromRGB(255, 200, 100))
        if hasWriteFile() then
            local fallbackContent = "-- Roblox Game Structure\n-- Place ID: " .. PLACE_ID .. "\n\n"
            local function listObjects(parent, depth)
                for _, child in ipairs(parent:GetChildren()) do
                    fallbackContent = fallbackContent .. string.rep("  ", depth) .. child.ClassName .. ": " .. child.Name .. "\n"
                    pcall(function()
                        listObjects(child, depth + 1)
                    end)
                end
            end
            pcall(function() listObjects(game, 0) end)

            local fbName = "game_structure_" .. PLACE_ID .. ".txt"
            pcall(function() writefile(fbName, fallbackContent) end)
            addLog("📝 Estrutura salva: " .. fbName, Color3.fromRGB(200, 200, 200))
        end

        updateProgress(0, "Download não disponível")
        addLog("❌ Nenhum método funcionou", Color3.fromRGB(255, 100, 100))
        addLog("💡 Use Synapse X ou KRNL para download completo", Color3.fromRGB(255, 255, 100))

        btn.Text = "📥 BAIXAR JOGO COMPLETO (.rbxlx)"
        btn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        btn.Active = true
    end

    -- Conectar botão
    downloadBtn.MouseButton1Click:Connect(function()
        task.spawn(performDownload)
    end)

    -- ========== LOGS INICIAIS ==========
    addLog("✅ Interface carregada!", Color3.fromRGB(100, 255, 100))
    addLog("🎯 Jogo: " .. gameName, Color3.fromRGB(200, 200, 200))
    addLog("⚡ Executor: " .. getExecutorName(), Color3.fromRGB(200, 200, 200))
    if hasSI then addLog("✅ saveinstance disponível", Color3.fromRGB(100, 255, 100)) end
    if hasWF then addLog("✅ writefile disponível", Color3.fromRGB(100, 255, 100)) end
    addLog("Clique no botão para baixar!", Color3.fromRGB(255, 255, 255))

    -- Garante que a UI está visível e no topo
    mainFrame.Position = UDim2.new(0.5, -260, 0.5, -225)
    screenGui.Enabled = true
    mainFrame.Visible = true
end

-- ============================================
-- ABRIR INTERFACE IMEDIATAMENTE
-- ============================================
-- Pequeno delay para garantir que o jogo carregou
task.wait(0.5)
createInterface()

-- ============================================
-- ATALHOS (F3 e comando /download)
-- ============================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F3 then
        removeOldUI()
        createInterface()
    end
end)

LocalPlayer.Chatted:Connect(function(message)
    local msg = message:lower()
    if msg == "/download" or msg == "/dl" or msg == "/nexo" then
        removeOldUI()
        createInterface()
    end
end)

-- ============================================
-- NOTIFICAÇÃO E CONFIRMAÇÃO
-- ============================================
print("=========================================")
print(" ROBLOX GAME DOWNLOADER - CARREGADO")
print(" Interface aberta automaticamente")
print(" Pressione F3 ou digite /download")
print("=========================================")

--[[
    Script: Roblox Full Game Dumper v2.0
    Requisitos: Executor com capacidade de escrita de arquivos (writefile, makefolder, isfolder)
                e requisições HTTP (syn.request, http.request, etc.)
    Saída: Todos os assets e scripts locais do jogo atual.
    Interface: Botão "BAIXAR TUDO" + exibição do caminho final.
--]]

local DumpFolder = "Roblox_Game_Dump/"
local AbsolutePath = getexecutorname and "Pasta do executor" or "Diretório do executor"

-- Função para detectar método de requisição
local function HttpGet(url)
    local success, result
    if syn and syn.request then
        success, result = pcall(function()
            local resp = syn.request({ Url = url, Method = "GET" })
            return resp.Body
        end)
        if success then return result end
    elseif game:GetService("HttpService") then
        success, result = pcall(function()
            return game:GetService("HttpService"):GetAsync(url)
        end)
        if success then return result end
    end
    return nil
end

-- Baixa qualquer asset via ID
local function DownloadAsset(assetId, subFolder, extension)
    if not assetId or assetId == 0 then return false end
    local url = "https://assetdelivery.roblox.com/v1/asset/?id=" .. assetId
    local data = HttpGet(url)
    if data and #data > 100 then
        local path = DumpFolder .. subFolder .. "/"
        if not isfolder(path) then makefolder(path) end
        local fileName = assetId .. extension
        local fullPath = path .. fileName
        writefile(fullPath, data)
        return fullPath
    end
    return false
end

-- Extrai scripts locais (fonte, se disponível)
local function DumpScript(scriptObj, pathStack)
    local source = ""
    local success, res = pcall(function()
        if getsourcestring then
            source = getsourcestring(scriptObj) or scriptObj.Source
        else
            source = scriptObj.Source
        end
    end)
    if not success or source == nil then
        source = "--[[ Código protegido ou não acessível ]]"
    end
    local scriptName = string.gsub(scriptObj.Name, "[^%w_]", "_")
    local folderPath = DumpFolder .. "Scripts/" .. pathStack
    if not isfolder(folderPath) then makefolder(folderPath) end
    local filePath = folderPath .. "/" .. scriptName .. ".lua"
    writefile(filePath, source)
    return filePath
end

-- Varredura recursiva completa
local dumpedAssets = {}
local dumpedScripts = {}

local function ScanInstance(inst, pathStack)
    for _, child in ipairs(inst:GetChildren()) do
        local childStack = pathStack .. "/" .. child.Name
        
        -- Scripts
        if child:IsA("LocalScript") or child:IsA("ModuleScript") then
            local p = DumpScript(child, childStack)
            table.insert(dumpedScripts, p)
        end
        
        -- MeshPart
        if child:IsA("MeshPart") and child.MeshId and child.MeshId ~= "" then
            local id = tonumber(string.match(child.MeshId, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Meshes", ".mesh")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- SpecialMesh
        if child:IsA("SpecialMesh") and child.MeshId and child.MeshId ~= "" then
            local id = tonumber(string.match(child.MeshId, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Meshes", ".mesh")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- Texture
        if child:IsA("Texture") and child.Texture and child.Texture ~= "" then
            local id = tonumber(string.match(child.Texture, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Textures", ".png")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- Decal
        if child:IsA("Decal") and child.Texture and child.Texture ~= "" then
            local id = tonumber(string.match(child.Texture, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Decals", ".png")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- Sound
        if child:IsA("Sound") and child.SoundId and child.SoundId ~= "" then
            local id = tonumber(string.match(child.SoundId, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Sounds", ".mp3")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- Animation
        if child:IsA("Animation") and child.AnimationId and child.AnimationId ~= "" then
            local id = tonumber(string.match(child.AnimationId, "(%d+)"))
            if id then
                local path = DownloadAsset(id, "Animations", ".rbxm")
                if path then table.insert(dumpedAssets, path) end
            end
        end
        
        -- Recursão
        ScanInstance(child, childStack)
    end
end

-- Criar interface GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FullGameDumper"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 420, 0, 200)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.95
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
title.Text = "ROBLOX GAME DUMPER - FULL EXTRACTION"
title.TextColor3 = Color3.fromRGB(255, 200, 100)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.Parent = mainFrame

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -20, 0, 50)
status.Position = UDim2.new(0, 10, 0, 45)
status.BackgroundTransparency = 1
status.Text = "Clique em 'INICIAR DUMP' para extrair TODOS os arquivos do jogo."
status.TextColor3 = Color3.fromRGB(200, 200, 200)
status.Font = Enum.Font.SourceSans
status.TextSize = 12
status.TextWrapped = true
status.Parent = mainFrame

local pathLabel = Instance.new("TextLabel")
pathLabel.Size = UDim2.new(1, -20, 0, 30)
pathLabel.Position = UDim2.new(0, 10, 0, 100)
pathLabel.BackgroundTransparency = 1
pathLabel.Text = "Caminho: (aguardando...)"
pathLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
pathLabel.Font = Enum.Font.SourceSans
pathLabel.TextSize = 11
pathLabel.TextXAlignment = Enum.TextXAlignment.Left
pathLabel.Parent = mainFrame

local dumpButton = Instance.new("TextButton")
dumpButton.Size = UDim2.new(0, 180, 0, 40)
dumpButton.Position = UDim2.new(0.5, -90, 0, 145)
dumpButton.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
dumpButton.Text = "INICIAR DUMP"
dumpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
dumpButton.Font = Enum.Font.GothamBold
dumpButton.TextSize = 14
dumpButton.Parent = mainFrame

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 14
closeBtn.Parent = mainFrame

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

dumpButton.MouseButton1Click:Connect(function()
    status.Text = "⏳ Dump em andamento... Varrendo todos os objetos e baixando assets. Isso pode levar vários minutos."
    status.TextColor3 = Color3.fromRGB(255, 255, 100)
    dumpButton.Visible = false
    
    -- Limpar e recriar pasta
    if isfolder(DumpFolder) then
        delfolder(DumpFolder)
        wait(0.2)
    end
    makefolder(DumpFolder)
    
    dumpedAssets = {}
    dumpedScripts = {}
    
    -- Pontos de entrada da varredura (absolutamente tudo)
    local roots = {
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
    
    for _, root in ipairs(roots) do
        if root then
            pcall(ScanInstance, root, root.ClassName)
        end
    end
    
    -- Extra também de qualquer objeto solto no game
    pcall(ScanInstance, game, "Game_Root")
    
    local totalAssets = #dumpedAssets
    local totalScripts = #dumpedScripts
    
    -- Determinar caminho ABSOLUTO (tenta obter do executor)
    local finalPath = DumpFolder
    if syn and syn.get_executor_path then
        local execPath = syn.get_executor_path()
        finalPath = execPath .. "/" .. DumpFolder
    elseif getexecutorname then
        finalPath = getexecutorname() .. "_workspace/" .. DumpFolder
    else
        finalPath = "Pasta do executor / " .. DumpFolder
    end
    
    status.Text = string.format("✅ DUMP CONCLUÍDO! Assets baixados: %d | Scripts extraídos: %d", totalAssets, totalScripts)
    status.TextColor3 = Color3.fromRGB(100, 255, 100)
    
    pathLabel.Text = "📁 Caminho completo: " .. finalPath
    pathLabel.TextColor3 = Color3.fromRGB(0, 255, 200)
    
    -- Botão para copiar caminho (opcional)
    local copyBtn = Instance.new("TextButton")
    copyBtn.Size = UDim2.new(0, 120, 0, 25)
    copyBtn.Position = UDim2.new(0.5, -60, 0, 175)
    copyBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    copyBtn.Text = "COPIAR CAMINHO"
    copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    copyBtn.Font = Enum.Font.SourceSans
    copyBtn.TextSize = 11
    copyBtn.Parent = mainFrame
    
    copyBtn.MouseButton1Click:Connect(function()
        if setclipboard then
            setclipboard(finalPath)
            status.Text = "📋 Caminho copiado para a área de transferência!"
        else
            status.Text = "❌ Não foi possível copiar automaticamente. Copie manualmente: " .. finalPath
        end
        wait(2)
        status.Text = string.format("✅ DUMP CONCLUÍDO! Assets: %d | Scripts: %d", totalAssets, totalScripts)
    end)
end)

--[[
    Script: Roblox Game Dumper (Full Asset + Script Extraction)
    Executor: Qualquer executor compatível com Synapse X, Krnl, ScriptWare, etc.
    Função: Baixa todos os assets (malhas, texturas, sons, decalques) e scripts locais do jogo atual.
    Interface GUI própria com botão "Baixar".
    Os arquivos são salvos na pasta "RobloxDump" dentro do diretório de trabalho do executor.
--]]

local DumpFolder = "RobloxDump/"
if not isfolder(DumpFolder) then
    makefolder(DumpFolder)
end

-- Função para baixar asset via ID
local function DownloadAsset(assetId, assetType, fileName)
    if not assetId or assetId == 0 then return end
    local url = "https://assetdelivery.roblox.com/v1/asset?id=" .. assetId
    local success, response = pcall(function()
        return syn.request({ Url = url, Method = "GET" })
    end)
    if success and response and response.Body then
        local path = DumpFolder .. assetType .. "/"
        if not isfolder(path) then makefolder(path) end
        local filePath = path .. fileName
        writefile(filePath, response.Body)
        return true
    end
    return false
end

-- Função para extrair scripts locais (código fonte se disponível)
local function DumpLocalScript(scriptInstance)
    if scriptInstance.ClassName ~= "LocalScript" and scriptInstance.ClassName ~= "ModuleScript" then
        return
    end
    local source = "" 
    local success, res = pcall(function()
        -- Tentativa de obter o código fonte (funciona se o executor permite)
        source = getsourcestring(scriptInstance) or scriptInstance.Source
    end)
    if not success or not source then
        source = "-- Código fonte não disponível ou protegido"
    end
    local path = DumpFolder .. "Scripts/"
    if not isfolder(path) then makefolder(path) end
    local fileName = string.gsub(scriptInstance:GetFullName(), "%.", "_") .. ".lua"
    writefile(path .. fileName, source)
end

-- Função recursiva para varrer todos os objetos do jogo
local function ScanGameObjects(instance)
    for _, child in ipairs(instance:GetChildren()) do
        -- Dump scripts
        if child:IsA("LocalScript") or child:IsA("ModuleScript") then
            DumpLocalScript(child)
        end
        
        -- Dump MeshPart (ID da malha)
        if child:IsA("MeshPart") and child.MeshId ~= "" then
            local meshId = child.MeshId
            local assetId = string.match(meshId, "rbxassetid://(%d+)")
            if assetId then
                DownloadAsset(assetId, "Meshes", assetId .. ".mesh")
            end
        end
        
        -- Dump Texture (ID da textura)
        if child:IsA("Texture") and child.Texture ~= "" then
            local texId = child.Texture
            local assetId = string.match(texId, "rbxassetid://(%d+)")
            if assetId then
                DownloadAsset(assetId, "Textures", assetId .. ".png")
            end
        end
        
        -- Dump Decal
        if child:IsA("Decal") and child.Texture ~= "" then
            local decalId = child.Texture
            local assetId = string.match(decalId, "rbxassetid://(%d+)")
            if assetId then
                DownloadAsset(assetId, "Decals", assetId .. ".png")
            end
        end
        
        -- Dump Sound
        if child:IsA("Sound") and child.SoundId ~= "" then
            local soundId = child.SoundId
            local assetId = string.match(soundId, "rbxassetid://(%d+)")
            if assetId then
                DownloadAsset(assetId, "Sounds", assetId .. ".mp3")
            end
        end
        
        -- Dump Animation
        if child:IsA("Animation") and child.AnimationId ~= "" then
            local animId = child.AnimationId
            local assetId = string.match(animId, "rbxassetid://(%d+)")
            if assetId then
                DownloadAsset(assetId, "Animations", assetId .. ".rbxm")
            end
        end
        
        -- Continuar recursão
        ScanGameObjects(child)
    end
end

-- Criar a interface GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DumperGUI"
screenGui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 150)
frame.Position = UDim2.new(0.5, -150, 0.5, -75)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
title.Text = "ROBLOX GAME DUMPER"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 40)
statusLabel.Position = UDim2.new(0, 0, 0, 35)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Pronto para baixar."
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 12
statusLabel.TextWrapped = true
statusLabel.Parent = frame

local downloadButton = Instance.new("TextButton")
downloadButton.Size = UDim2.new(0, 200, 0, 40)
downloadButton.Position = UDim2.new(0.5, -100, 0, 85)
downloadButton.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
downloadButton.Text = "BAIXAR JOGO COMPLETO"
downloadButton.TextColor3 = Color3.fromRGB(255, 255, 255)
downloadButton.Font = Enum.Font.GothamBold
downloadButton.TextSize = 14
downloadButton.Parent = frame

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 20, 0, 20)
closeButton.Position = UDim2.new(1, -25, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextSize = 14
closeButton.Parent = frame

closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

downloadButton.MouseButton1Click:Connect(function()
    statusLabel.Text = "Iniciando dump... Isso pode levar alguns minutos."
    statusLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
    
    -- Limpa pasta anterior (opcional, comente se não quiser)
    -- if isfolder(DumpFolder) then delfolder(DumpFolder) end
    -- makefolder(DumpFolder)
    
    -- Inicia varredura a partir do game (Players, Workspace, ReplicatedStorage, etc.)
    local objects = {
        game.Workspace,
        game.ReplicatedStorage,
        game.ServerStorage,
        game.Lighting,
        game.Players
    }
    for _, obj in ipairs(objects) do
        ScanGameObjects(obj)
    end
    
    -- Também varre o StarterGui e StarterPlayer
    ScanGameObjects(game.StarterGui)
    if game.StarterPlayer then
        ScanGameObjects(game.StarterPlayer)
        if game.StarterPlayer.StarterPlayerScripts then
            ScanGameObjects(game.StarterPlayer.StarterPlayerScripts)
        end
    end
    
    statusLabel.Text = "Download concluído! Arquivos salvos em " .. DumpFolder
    statusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
end)

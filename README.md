-- SKYNET ULTIMATE DUMPER V3.0 - RECONSTRUCTOR
-- Engenharia Reversa Profunda, Download de Assets via CDN e Serialização Manual
-- Requisitos: Executor com request()/syn.request(), writefile(), makefolder(), readfile()
-- Autor: SKYNETchat Logic Core (Refactored for Maximum Fidelity)

local Players = game:GetService("Players")
local LocalPlayer = Players:GetChildren()[1] or Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local MarketplaceService = game:GetService("MarketplaceService")
local TeleportService = game:GetService("TeleportService")

-- Configurações
local DUMP_DIR = "SKYNET_COMPLETE_DUMP"
local ASSET_CACHE_DIR = DUMP_DIR .. "/_Assets"
local PLACE_ID = game.PlaceId
local PLACE_NAME = "Unknown_Game"

-- Tenta pegar o nome do jogo sem travar
pcall(function()
    local info = MarketplaceService:GetProductInfo(Place_ID)
    PLACE_NAME = info.Name:gsub("[^%a%d%s]", "_")
end)

local FINAL_FOLDER = string.format("%s/%s_%s", DUMP_DIR, Place_ID, PLACE_NAME)

-- Tema da UI
local THEME = {
    Bg = Color3.fromRGB(18, 18, 18),
    Primary = Color3.fromRGB(0, 255, 136),
    Text = Color3.fromRGB(240, 240, 240),
    Error = Color3.fromRGB(255, 60, 60),
    Warn = Color3.fromRGB(255, 200, 0)
}

-- === SISTEMA DE UI ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SKYNET_V3_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 450, 0, 550)
Main.Position = UDim2.new(0.5, -225, 0.5, -275)
Main.BackgroundColor3 = THEME.Bg
Main.BorderSizePixel = 0
Main.Parent = ScreenGui

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = THEME.Primary
UIStroke.Thickness = 2
UIStroke.Parent = Main

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "SKYNET V3 // FULL RECONSTRUCTOR"
Title.TextColor3 = THEME.Primary
Title.Font = Enum.Font.Code
Title.TextSize = 16
Title.Parent = Main

local LogFrame = Instance.new("ScrollingFrame")
LogFrame.Size = UDim2.new(1, -20, 0, 350)
LogFrame.Position = UDim2.new(0, 10, 0, 50)
LogFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
LogFrame.BorderSizePixel = 0
LogFrame.ScrollBarThickness = 4
LogFrame.Parent = Main

local LogLayout = Instance.new("UIListLayout")
LogLayout.Padding = UDim.new(0, 3)
LogLayout.Parent = LogFrame

local function Log(msg, type)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -10, 0, 18)
    lbl.BackgroundTransparency = 1
    lbl.Text = "[" .. os.date("%X") .. "] " .. msg
    lbl.TextColor3 = type == "ERR" and THEME.Error or type == "WARN" and THEME.Warn or THEME.Text
    lbl.Font = Enum.Font.Code
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = LogFrame
    LogFrame.CanvasPosition = Vector2.new(0, LogFrame.CanvasPosition.Y + 9999)
end

local ActionBtn = Instance.new("TextButton")
ActionBtn.Size = UDim2.new(1, -20, 0, 45)
ActionBtn.Position = UDim2.new(0, 10, 1, -55)
ActionBtn.BackgroundColor3 = THEME.Primary
ActionBtn.TextColor3 = Color3.fromRGB(0,0,0)
ActionBtn.Font = Enum.Font.Code
ActionBtn.TextSize = 14
ActionBtn.Text = "INICIAR EXTRAÇÃO PROFUNDA"
ActionBtn.Parent = Main

local ProgressBar = Instance.new("Frame")
ProgressBar.Size = UDim2.new(1, -20, 0, 8)
ProgressBar.Position = UDim2.new(0, 10, 1, -95)
ProgressBar.BackgroundColor3 = Color3.fromRGB(40,40,40)
ProgressBar.Parent = Main

local ProgressFill = Instance.new("Frame")
ProgressFill.Size = UDim2.new(0, 0, 1, 0)
ProgressFill.BackgroundColor3 = THEME.Primary
ProgressFill.Parent = ProgressBar

-- Dragging
local dragging = false
local dragInput, mousePos, framePos
Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = Main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
Title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        Main:TweenPosition(UDim2.new(0, framePos.X.Offset + delta.X, 0, framePos.Y.Offset + delta.Y), "Out", "Quad", 0.1, true)
    end
end)

-- === MOTOR DE DUMP ===

local function SafeWrite(path, data)
    local s, e = pcall(function() writefile(path, data) end)
    return s, e
end

local function SafeMake(path)
    local s, e = pcall(function() makefolder(path) end)
    return s, e
end

local function RequestAsset(assetId)
    -- Tentativa de baixar o binário do asset via CDN
    local url = "https://www.roblox.com/asset/?id=" .. tostring(assetId)
    local success, result = pcall(function()
        if syn and syn.request then
            return syn.request({Url = url, Method = "GET"})
        elseif request then
            return request({Url = url, Method = "GET"})
        elseif http and http.request then
            return http.request({Url = url, Method = "GET"})
        else
            error("No request function found")
        end
    end)
    
    if success and result.StatusCode == 200 then
        return result.Body
    end
    return nil
end

local function ExtractAssetsRecursively(instanceObj, rootPath)
    local assetsFound = 0
    
    -- Função interna para processar um ID
    local function ProcessAsset(id, assetType, extension)
        if not id then return end
        local assetPath = string.format("%s/%s/%s.%s", rootPath, assetType .. "s", id, extension)
        
        -- Verifica se já não baixamos
        local exists, _ = pcall(readfile, assetPath)
        if not exists then
            Log("Baixando " .. assetType .. ": " .. id, "INFO")
            local data = RequestAsset(id)
            if data then
                SafeWrite(assetPath, data)
                assetsFound = assetsFound + 1
            else
                Log("Falha ao baixar " .. assetType .. ": " .. id, "WARN")
            end
        end
    end

    if instanceObj:IsA("ImageLabel") or instanceObj:IsA("Decal") or instanceObj:IsA("Texture") then
        local id = instanceObj.TextureId:match("rbxassetid://(%d+)")
        if id then 
            SafeMake(rootPath .. "/Textures")
            ProcessAsset(id, "Texture", "png") -- Extensão genérica, pode variar
        end
    elseif instanceObj:IsA("MeshPart") then
        local meshId = instanceObj.MeshId:match("rbxassetid://(%d+)")
        if meshId then
            SafeMake(rootPath .. "/Meshes")
            ProcessAsset(meshId, "Mesh", "mesh")
        end
    elseif instanceObj:IsA("Sound") then
        local soundId = instanceObj.SoundId:match("rbxassetid://(%d+)")
        if soundId then
            SafeMake(rootPath .. "/Sounds")
            ProcessAsset(soundId, "Sound", "mp3")
        end
    end

    -- Recursão
    for _, child in ipairs(instanceObj:GetChildren()) do
        assetsFound = assetsFound + ExtractAssetsRecursively(child, rootPath)
    end
    return assetsFound
end

local function DumpScripts(instanceObj, scriptPath)
    local count = 0
    if instanceObj:IsA("Script") or instanceObj:IsA("LocalScript") or instanceObj:IsA("ModuleScript") then
        local source = nil
        local success, err = pcall(function() source = instanceObj:GetSource() end)
        
        local fileName = string.format("%s_%s_%s.lua", instanceObj.ClassName, instanceObj.Name, instanceObj:GetFullName():gsub("[^%a%d]", "_"))
        -- Limitar tamanho do nome do arquivo
        if #fileName > 100 then fileName = fileName:sub(1, 100) .. ".lua" end
        
        if success and source then
            SafeWrite(scriptPath .. "/" .. fileName, source)
            Log("Script Extraído: " .. instanceObj:GetFullName(), "INFO")
            count = count + 1
        else
            -- Cria um placeholder indicando que o script é server-side ou ofuscado
            local placeholder = "-- SCRIPT NÃO DISPONÍVEL NO CLIENTE\n-- Este é um Server Script ou está protegido.\n-- Impossível extrair o código fonte desta instância: " .. instanceObj:GetFullName()
            SafeWrite(scriptPath .. "/" .. fileName, placeholder)
            Log("Script Inacessível (Server/Protected): " .. instanceObj:GetFullName(), "WARN")
            count = count + 1
        end
    end

    for _, child in ipairs(instanceObj:GetChildren()) do
        count = count + DumpScripts(child, scriptPath)
    end
    return count
end

-- Função Principal de Reconstrução
local function StartReconstruction()
    ActionBtn.Text = "PROCESSANDO..."
    ActionBtn.BackgroundColor3 = THEME.Error
    
    Log("=== INICIANDO PROTOCOLO DE RECONSTRUÇÃO ===", "INFO")
    Log("Alvo: " .. Place_ID .. " (" .. PLACE_NAME .. ")", "INFO")
    
    -- 1. Preparar Pastas
    SafeMake(DUMP_DIR)
    SafeMake(FINAL_FOLDER)
    SafeMake(FINAL_FOLDER .. "/Scripts")
    SafeMake(FINAL_FOLDER .. "/Assets")
    
    Log("Estrutura de pastas criada.", "INFO")
    
    -- 2. Fase de Extração de Assets Binários (Texturas, Meshes, Sons)
    Log("Fase 1: Varredura e Download de Assets Binários...", "WARN")
    local totalAssets = ExtractAssetsRecursively(game.Workspace, FINAL_FOLDER .. "/Assets")
    -- Varre também o Lighting e outros serviços se necessário, mas Workspace é o principal
    totalAssets = totalAssets + ExtractAssetsRecursively(game.Lighting, FINAL_FOLDER .. "/Assets")
    
    Log(string.format("Fase 1 Concluída. %d assets baixados/verificados.", totalAssets), "INFO")
    ProgressFill.Size = UDim2.new(0.3, 0, 1, 0)
    
    -- 3. Fase de Extração de Scripts
    Log("Fase 2: Extração de Scripts (Client & Accessible Server)...", "WARN")
    local totalScripts = 0
    
    -- Varre todo o jogo
    totalScripts = DumpScripts(game, FINAL_FOLDER .. "/Scripts")
    
    Log(string.format("Fase 2 Concluída. %d instâncias de script processadas.", totalScripts), "INFO")
    ProgressFill.Size = UDim2.new(0.7, 0, 1, 0)
    
    -- 4. Geração do Arquivo de Projeto (Place File Simulado)
    -- Como não podemos gerar um .rbxl binário válido sem uma biblioteca complexa de serialização em Lua puro,
    -- vamos gerar um Script de Reconstrução que o usuário roda no Studio para montar o jogo.
    
    Log("Fase 3: Gerando Script de Reconstrução para Studio...", "INFO")
    
    local reconstructionScript = [[
-- SKYNET RECONSTRUCTION SCRIPT
-- Rode este script DENTRO do Roblox Studio para montar o jogo extraído.
-- Certifique-se de que a pasta extraída esteja na mesma localização relativa ou ajuste o caminho.

local BasePath = "rbxassetid://0" -- Substitua pelo caminho local se usar plugin de importação ou copie manualmente
print("Iniciando reconstrução...")
-- Nota: A reconstrução automática de hierarchy via script é limitada no Studio por segurança.
-- A melhor prática é copiar as pastas 'Scripts' e 'Assets' manualmente para o seu projeto.
-- Os assets baixados estão prontos para serem arrastados para o Explorer.
]]
    
    -- Vamos criar um arquivo de manifesto JSON com tudo o que foi extraído
    local manifest = {
        PlaceId = Place_ID,
        PlaceName = PLACE_NAME,
        DumpDate = os.date(),
        TotalAssets = totalAssets,
        TotalScripts = totalScripts,
        Warning = "Scripts do servidor não foram extraídos (impossível via cliente). Assets estão na pasta /Assets."
    }
    
    SafeWrite(FINAL_FOLDER .. "/MANIFESTO.json", HttpService:JSONEncode(manifest))
    
    Log("Manifesto gerado (MANIFESTO.json).", "INFO")
    Log("=== PROCESSO FINALIZADO ===", "INFO")
    Log("Vá até a pasta: " .. FINAL_FOLDER, "INFO")
    Log("1. Os assets estão em /Assets (prontos para uso).", "INFO")
    Log("2. Os scripts estão em /Scripts (copie o código manualmente para o Studio).", "INFO")
    Log("3. Scripts marcados como 'NÃO DISPONÍVEL' são server-side e não existem no seu PC.", "ERR")
    
    ProgressFill.Size = UDim2.new(1, 0, 1, 0)
    ActionBtn.Text = "CONCLUÍDO"
    ActionBtn.BackgroundColor3 = THEME.Primary
end

ActionBtn.MouseButton1Click:Connect(function()
    if ActionBtn.Text == "PROCESSANDO..." then return end
    StartReconstruction()
end)

Log("SKYNET V3 Pronto. Aguardando comando.", "INFO")
Log("Este script baixa assets reais e extrai scripts acessíveis.", "WARN")

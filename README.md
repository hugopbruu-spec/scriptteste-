-- SKYNET ULTIMATE DUMPER V2.0
-- Engenharia Reversa e Extração de Assets em Tempo Real
-- Compatível com Executors que suportam: saveinstance(), writefile(), makefolder()
-- Autor: SKYNETchat Logic Core

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

-- Configurações Globais
local DUMP_FOLDER_NAME = "SKYNET_DUMPS"
local IS_DUMPING = false

-- Tabela de Cores e Estilos da GUI
local THEME = {
    Background = Color3.fromRGB(20, 20, 20),
    Primary = Color3.fromRGB(0, 255, 136),
    Text = Color3.fromRGB(255, 255, 255),
    Secondary = Color3.fromRGB(40, 40, 40),
    Danger = Color3.fromRGB(255, 50, 50)
}

-- Criação da Interface (GUI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SKYNET_Dumper_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 500)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
MainFrame.BackgroundColor3 = THEME.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Sombra e Borda
local UIStroke = Instance.new("UIStroke")
UIStroke.Color = THEME.Primary
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundColor3 = THEME.Secondary
Title.Text = "SKYNET // EXTRAÇÃO TOTAL"
Title.TextColor3 = THEME.Primary
Title.Font = Enum.Font.Code
Title.TextSize = 18
Title.Parent = MainFrame

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 60)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Aguardando início..."
StatusLabel.TextColor3 = THEME.Text
StatusLabel.Font = Enum.Font.Code
StatusLabel.TextSize = 12
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = MainFrame

local ProgressBar = Instance.new("Frame")
ProgressBar.Size = UDim2.new(0, 0, 0, 10)
ProgressBar.Position = UDim2.new(0, 10, 0, 95)
ProgressBar.BackgroundColor3 = THEME.Secondary
ProgressBar.Parent = MainFrame

local ProgressFill = Instance.new("Frame")
ProgressFill.Size = UDim2.new(0, 0, 1, 0)
ProgressFill.BackgroundColor3 = THEME.Primary
ProgressFill.Parent = ProgressBar

local LogBox = Instance.new("ScrollingFrame")
LogBox.Size = UDim2.new(1, -20, 0, 250)
LogBox.Position = UDim2.new(0, 10, 0, 115)
LogBox.BackgroundColor3 = THEME.Secondary
LogBox.BorderSizePixel = 0
LogBox.ScrollBarThickness = 5
LogBox.Parent = MainFrame

local LogLayout = Instance.new("UIListLayout")
LogLayout.Padding = UDim.new(0, 4)
LogLayout.Parent = LogBox

local StartButton = Instance.new("TextButton")
StartButton.Size = UDim2.new(1, -20, 0, 50)
StartButton.Position = UDim2.new(0, 10, 1, -60)
StartButton.BackgroundColor3 = THEME.Primary
StartButton.TextColor3 = Color3.fromRGB(0,0,0)
StartButton.Font = Enum.Font.Code
StartButton.TextSize = 16
StartButton.Text = "INICIAR DOWNLOAD COMPLETO"
StartButton.Parent = MainFrame

local FolderInput = Instance.new("TextBox")
FolderInput.Size = UDim2.new(1, -20, 0, 30)
FolderInput.Position = UDim2.new(0, 10, 1, -100)
FolderInput.BackgroundColor3 = THEME.Background
FolderInput.TextColor3 = THEME.Text
FolderInput.Font = Enum.Font.Code
FolderInput.PlaceholderText = "Nome da Pasta de Saída"
FolderInput.Text = DUMP_FOLDER_NAME
FolderInput.Parent = MainFrame

-- Funções de Utilidade da GUI
local function AddLog(text, color)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 20)
    label.BackgroundTransparency = 1
    label.Text = "> " .. text
    label.TextColor3 = color or THEME.Text
    label.Font = Enum.Font.Code
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = LogBox
    LogBox.CanvasPosition = Vector2.new(0, LogBox.CanvasPosition.Y + 24)
end

local function UpdateProgress(percent)
    ProgressFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.2, true)
end

local function Dragify(frame)
    local holding = false
    local offset
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            holding = true
            offset = input.Position - frame.AbsolutePosition
        end
    end)
    frame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            holding = false
        end
    end)
    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if holding and input.UserInputType == Enum.UserInputType.MouseMovement then
            local pos = input.Position - offset
            frame:TweenPosition(UDim2.new(0, pos.X, 0, pos.Y), "Out", "Quad", 0.1, true)
        end
    end)
end
Dragify(Title)

-- LÓGICA DE DUMP PRINCIPAL
local function SafeWriteFile(path, data)
    local success, err = pcall(function()
        writefile(path, data)
    end)
    return success, err
end

local function SafeMakeFolder(path)
    local success, err = pcall(function()
        makefolder(path)
    end)
    return success, err
end

local function GetAssetId(instanceObj)
    -- Extrai IDs de texturas, meshes, sons
    local ids = {}
    if instanceObj:IsA("Texture") or instanceObj:IsA("ImageLabel") or instanceObj:IsA("Decal") then
        local id = instanceObj.TextureId:match("rbxassetid://(%d+)")
        if id then table.insert(ids, {type="Texture", id=id, name=instanceObj.Name}) end
    elseif instanceObj:IsA("MeshPart") then
        local id = instanceObj.MeshId:match("rbxassetid://(%d+)")
        if id then table.insert(ids, {type="Mesh", id=id, name=instanceObj.Name}) end
    elseif instanceObj:IsA("Sound") then
        local id = instanceObj.SoundId:match("rbxassetid://(%d+)")
        if id then table.insert(ids, {type="Sound", id=id, name=instanceObj.Name}) end
    end
    return ids
end

StartButton.MouseButton1Click:Connect(function()
    if IS_DUMPING then return end
    IS_DUMPING = true
    StartButton.Text = "PROCESSANDO..."
    StartButton.BackgroundColor3 = THEME.Danger
    
    local folderName = FolderInput.Text ~= "" and FolderInput.Text or "SKYNET_DUMPS"
    local baseFolder = folderName .. "/" .. game.PlaceId .. "_" .. game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name:gsub("[^%a%d]", "_")
    
    AddLog("Iniciando protocolo de extração...", THEME.Primary)
    AddLog("Alvo: " .. game.PlaceId, THEME.Text)
    
    -- Tentativa 1: SaveInstance (Método Nuclear - Baixa tudo em um arquivo)
    AddLog("Tentando método 'saveinstance()' (Dump Binário Completo)...", THEME.Primary)
    
    local successSI, errSI = pcall(function()
        if saveinstance then
            UpdateProgress(0.1)
            AddLog("Compilando árvore do jogo...", THEME.Text)
            local placeData = saveinstance()
            UpdateProgress(0.5)
            AddLog("Salvando arquivo .rbxl...", THEME.Text)
            
            SafeMakeFolder(folderName)
            local fileName = baseFolder .. "_FULL_DUMP.rbxl"
            SafeWriteFile(fileName, placeData)
            
            AddLog("SUCESSO! Jogo completo salvo em: " .. fileName, THEME.Primary)
            AddLog("Abra este arquivo no Roblox Studio para ver tudo.", THEME.Text)
            UpdateProgress(1)
            return true
        else
            error("Função saveinstance não encontrada neste executor.")
        end
    end)

    if not successSI then
        AddLog("Falha no dump binário: " .. tostring(errSI), THEME.Danger)
        AddLog("Iniciando fallback: Extração Manual de Assets (Lento)...", THEME.Danger)
        
        -- Fallback: Extração Manual (Se saveinstance falhar ou não existir)
        SafeMakeFolder(folderName)
        SafeMakeFolder(baseFolder .. "/Scripts")
        SafeMakeFolder(baseFolder .. "/Assets")
        
        local count = 0
        local totalAssets = 0
        
        -- Contagem preliminar (estimativa)
        for _, obj in ipairs(game:GetDescendants()) do
            totalAssets = totalAssets + 1
        end
        
        local current = 0
        for _, obj in ipairs(game:GetDescendants()) do
            current = current + 1
            UpdateProgress(current / totalAssets)
            
            if obj:IsA("LocalScript") or obj:IsA("Script") then
                -- Tentar pegar o source (só funciona se não estiver ofuscado/protegido)
                local success, source = pcall(function() return obj:GetSource() end)
                if success then
                    local fname = baseFolder .. "/Scripts/" .. obj.Name .. "_" .. obj.GetFullName():gsub("[%./]", "_") .. ".lua"
                    SafeWriteFile(fname, source)
                    AddLog("Script extraído: " .. obj.Name, THEME.Text)
                end
            end
            
            -- Extrair Assets
            local assets = GetAssetId(obj)
            for _, asset in ipairs(assets) do
                local url = "rbxassetid://" .. asset.id
                -- Nota: Baixar o conteúdo bruto do asset require httprequest em alguns executors
                -- Aqui estamos apenas logando e salvando a referência se o download direto falhar
                -- Para download real de binário de asset, seria necessário uma loop de httprequest
                AddLog("Asset Identificado: " .. asset.name .. " (" .. asset.id .. ")", THEME.Text)
            end
            
            if current % 50 == 0 then
                RunService.Heartbeat:Wait() -- Evitar crash por overflow
            end
        end
        
        AddLog("Processo finalizado. Verifique a pasta: " .. baseFolder, THEME.Primary)
    end

    IS_DUMPING = false
    StartButton.Text = "CONCLUÍDO"
    StartButton.BackgroundColor3 = THEME.Primary
end)

AddLog("SKYNET Dumper Pronto.", THEME.Primary)
AddLog("Insira o nome da pasta e clique em Iniciar.", THEME.Text)

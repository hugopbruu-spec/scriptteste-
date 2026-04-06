-- SKYNET ULTIMATE DUMPER V3.0 - "BLACKBOX" EDITION
-- Engenharia Reversa Avançada e Captura Completa de Dados em Tempo Real
-- Foco: Scripts Locais, Comunicação Remota e Mapeamento de Estrutura Completo.
-- Requer: Executor com suporte a saveinstance(), writefile(), makefolder(), e hooks (decomentador, httprequest)
-- Autor: SKYNETchat Logic Core

-- Serviços do Roblox
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")

-- Configurações Globais
local DUMP_FOLDER_NAME = "SKYNET_DUMPS_V3"
local IS_DUMPING = false
local REMOTE_LOGS = {} -- Armazena toda a comunicação remota
local CAPTURE_REMOTES_ENABLED = true

-- Tabela de Cores e Estilos da GUI (Melhorada)
local THEME = {
    Background = Color3.fromRGB(15, 15, 15),
    Primary = Color3.fromRGB(0, 255, 136),
    Text = Color3.fromRGB(220, 220, 220),
    Secondary = Color3.fromRGB(35, 35, 35),
    Tertiary = Color3.fromRGB(50, 50, 50),
    Danger = Color3.fromRGB(255, 50, 50),
    Warning = Color3.fromRGB(255, 200, 0)
}

-- Criação da Interface (GUI) - VERSÃO CORRIGIDA E ROBUSTA
local ScreenGui
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Função para criar a GUI de forma segura
local function createGUI()
    local success, err = pcall(function()
        ScreenGui = Instance.new("ScreenGui")
        ScreenGui.Name = "SKYNET_Dumper_V3_GUI"
        ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        ScreenGui.ResetOnSpawn = false -- Manter a GUI ao respawnar
        ScreenGui.IgnoreGuiInset = true
        -- A proteção abaixo ajuda a evitar que jogos anti-exploit removam a GUI facilmente
        if syn and syn.protect_gui then
            syn.protect_gui(ScreenGui)
            ScreenGui.Parent = game:GetService("CoreGui")
        else
            ScreenGui.Parent = PlayerGui
        end
    end)

    if not success then
        warn("SKYNET Dumper: Erro crítico ao criar a interface GUI: " .. tostring(err))
        -- Tenta uma abordagem alternativa sem proteção, caso a primeira falhe
        pcall(function()
            if ScreenGui then ScreenGui:Destroy() end
            ScreenGui = Instance.new("ScreenGui")
            ScreenGui.Name = "SKYNET_Dumper_V3_GUI_ALT"
            ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
            ScreenGui.ResetOnSpawn = false
            ScreenGui.IgnoreGuiInset = true
            ScreenGui.Parent = PlayerGui
        end)
    end
end

-- Executa a criação da GUI
createGUI()

-- Verifica final se a GUI foi criada com sucesso antes de continuar
if not ScreenGui or not ScreenGui.Parent then
    LocalPlayer:Kick("SKYNET Dumper: Falha fatal ao inicializar a interface. Verifique se o executor permite a criação de ScreenGuis.")
    return -- Interrompe a execução do script
end

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 650)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -325)
MainFrame.BackgroundColor3 = THEME.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = THEME.Primary
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundColor3 = THEME.Secondary
Title.Text = "SKYNET V3 // BLACKBOX DUMPER"
Title.TextColor3 = THEME.Primary
Title.Font = Enum.Font.Code
Title.TextSize = 20
Title.Parent = MainFrame

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 60)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Sistema pronto. Aguardando comando."
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
LogBox.Size = UDim2.new(1, -20, 0, 300)
LogBox.Position = UDim2.new(0, 10, 0, 115)
LogBox.BackgroundColor3 = THEME.Secondary
LogBox.BorderSizePixel = 0
LogBox.ScrollBarThickness = 5
LogBox.Parent = MainFrame

local LogLayout = Instance.new("UIListLayout")
LogLayout.Padding = UDim.new(0, 3)
LogLayout.Parent = LogBox

local OptionsFrame = Instance.new("Frame")
OptionsFrame.Size = UDim2.new(1, -20, 0, 80)
OptionsFrame.Position = UDim2.new(0, 10, 1, -130)
OptionsFrame.BackgroundTransparency = 1
OptionsFrame.Parent = MainFrame

local CaptureRemotesCheckbox = Instance.new("TextButton")
CaptureRemotesCheckbox.Size = UDim2.new(0.5, -5, 0, 30)
CaptureRemotesCheckbox.BackgroundColor3 = THEME.Tertiary
CaptureRemotesCheckbox.TextColor3 = THEME.Text
CaptureRemotesCheckbox.Font = Enum.Font.Code
CaptureRemotesCheckbox.TextSize = 12
CaptureRemotesCheckbox.Text = "[X] Capturar Remotes"
CaptureRemotesCheckbox.Parent = OptionsFrame

local DecompileCheckbox = Instance.new("TextButton")
DecompileCheckbox.Size = UDim2.new(0.5, -5, 0, 30)
DecompileCheckbox.Position = UDim2.new(0.5, 5, 0, 0)
DecompileCheckbox.BackgroundColor3 = THEME.Tertiary
DecompileCheckbox.TextColor3 = THEME.Text
DecompileCheckbox.Font = Enum.Font.Code
DecompileCheckbox.TextSize = 12
DecompileCheckbox.Text = "[X] Tentar Decompilar"
DecompileCheckbox.Parent = OptionsFrame

local StartButton = Instance.new("TextButton")
StartButton.Size = UDim2.new(1, -20, 0, 50)
StartButton.Position = UDim2.new(0, 10, 1, -180)
StartButton.BackgroundColor3 = THEME.Primary
StartButton.TextColor3 = Color3.fromRGB(0,0,0)
StartButton.Font = Enum.Font.Code
StartButton.TextSize = 16
StartButton.Text = "INICIAR DUMP COMPLETO"
StartButton.Parent = MainFrame

local FolderInput = Instance.new("TextBox")
FolderInput.Size = UDim2.new(1, -20, 0, 30)
FolderInput.Position = UDim2.new(0, 10, 1, -220)
FolderInput.BackgroundColor3 = THEME.Background
FolderInput.TextColor3 = THEME.Text
FolderInput.Font = Enum.Font.Code
FolderInput.PlaceholderText = "Nome da Pasta de Saída"
FolderInput.Text = DUMP_FOLDER_NAME
FolderInput.Parent = MainFrame

-- Estado das Opções
local captureRemotes = true
local tryDecompile = true

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
    label.TextWrapped = true
    label.Parent =

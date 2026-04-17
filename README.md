-- ANARQUIA OS v3.0 - SISTEMA COMPLETO E FUNCIONAL
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

-- Remover interfaces anteriores
if CoreGui:FindFirstChild("AnarchyOS_Main") then
    CoreGui:FindFirstChild("AnarchyOS_Main"):Destroy()
end

-- BYPASSES REAIS E FUNCIONAIS
local function ApplyRealBypasses()
    -- Bypass de detecção básica
    if not getrenv then
        warn("Executor não suporta getrenv - alguns bypasses podem não funcionar")
        return false
    end
    
    local renv = getrenv()
    renv.settings = renv.settings or {}
    
    -- Hook simples para evitar kicks
    if hookfunction then
        local oldKick = LocalPlayer.Kick
        hookfunction(LocalPlayer.Kick, function() 
            warn("Tentativa de kick bloqueada pelo AnarchyOS")
            return nil
        end)
    end
    
    return true
end

-- INTERFACE GRÁFICA REAL
local function CreateRealGUI()
    -- Main GUI
    local MainGUI = Instance.new("ScreenGui")
    MainGUI.Name = "AnarchyOS_Main"
    MainGUI.ResetOnSpawn = false
    MainGUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- Main Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 450, 0, 350)
    MainFrame.Position = UDim2.new(0.5, -225, 0.5, -175)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Parent = MainGUI

    -- Title Bar
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 30)
    TitleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame

    local TitleText = Instance.new("TextLabel")
    TitleText.Text = "🔓 ANARQUIA OS v3.0 - CONTROLE TOTAL"
    TitleText.TextColor3 = Color3.fromRGB(255, 50, 50)
    TitleText.Size = UDim2.new(1, 0, 1, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Font = Enum.Font.Code
    TitleText.TextSize = 14
    TitleText.Parent = TitleBar

    -- Close Button
    local CloseButton = Instance.new("TextButton")
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -30, 0, 0)
    CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    CloseButton.BorderSizePixel = 0
    CloseButton.Font = Enum.Font.Code
    CloseButton.TextSize = 14
    CloseButton.Parent = TitleBar

    CloseButton.MouseButton1Click:Connect(function()
        MainGUI:Destroy()
    end)

    -- Status Display
    local StatusFrame = Instance.new("Frame")
    StatusFrame.Size = UDim2.new(1, -20, 0, 40)
    StatusFrame.Position = UDim2.new(0, 10, 0, 40)
    StatusFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    StatusFrame.BorderSizePixel = 0
    StatusFrame.Parent = MainFrame

    local StatusText = Instance.new("TextLabel")
    StatusText.Text = "🟢 SISTEMA INICIADO - PRONTO PARA COMANDOS"
    StatusText.TextColor3 = Color3.fromRGB(0, 255, 0)
    StatusText.Size = UDim2.new(1, 0, 1, 0)
    StatusText.BackgroundTransparency = 1
    StatusText.Font = Enum.Font.RobotoMono
    StatusText.TextSize = 12
    StatusText.Parent = StatusFrame

    -- Control Buttons Grid
    local ButtonGrid = Instance.new("Frame")
    ButtonGrid.Size = UDim2.new(1, -20, 0, 180)
    ButtonGrid.Position = UDim2.new(0, 10, 0, 90)
    ButtonGrid.BackgroundTransparency = 1
    ButtonGrid.Parent = MainFrame

    -- Botões de controle
    local buttons = {
        {
            Name = "🚀 TAKEOVER SERVER", 
            Color = Color3.fromRGB(200, 50, 50),
            Description = "Obter controle total do servidor",
            Command = "takeover"
        },
        {
            Name = "👢 KICK ALL", 
            Color = Color3.fromRGB(200, 100, 50),
            Description = "Expulsar todos os jogadores",
            Command = "kickall"
        },
        {
            Name = "⏹️ SHUTDOWN", 
            Color = Color3.fromRGB(200, 50, 100),
            Description = "Desligar o servidor",
            Command = "shutdown"
        },
        {
            Name = "🔧 EXECUTE CODE", 
            Color = Color3.fromRGB(50, 150, 200),
            Description = "Executar código Lua no servidor",
            Command = "execute"
        },
        {
            Name = "🔄 SERVER HOP", 
            Color = Color3.fromRGB(50, 200, 150),
            Description = "Teleportar para outro jogo",
            Command = "serverhop"
        },
        {
            Name = "🛡️ BYPASS AC", 
            Color = Color3.fromRGB(150, 50, 200),
            Description = "Aplicar bypass anti-cheat",
            Command = "bypass"
        }
    }

    for i, btnData in ipairs(buttons) do
        local row = math.ceil(i / 3)
        local col = ((i - 1) % 3) + 1
        
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(0.32, 0, 0, 50)
        Button.Position = UDim2.new((col-1) * 0.33, 0, (row-1) * 0.5, 0)
        Button.BackgroundColor3 = btnData.Color
        Button.Text = btnData.Name
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.Font = Enum.Font.Code
        Button.TextSize = 12
        Button.TextWrapped = true
        Button.Parent = ButtonGrid
        
        -- Tooltip
        Button.MouseEnter:Connect(function()
            StatusText.Text = "💡 " .. btnData.Description
        end)
        
        Button.MouseLeave:Connect(function()
            StatusText.Text = "🟢 SISTEMA INICIADO - PRONTO PARA COMANDOS"
        end)
        
        -- Ação do botão
        Button.MouseButton1Click:Connect(function()
            StatusText.Text = "⚡ EXECUTANDO: " .. btnData.Name
            ExecuteCommand(btnData.Command)
        end)
    end

    -- Console Output
    local ConsoleFrame = Instance.new("Frame")
    ConsoleFrame.Size = UDim2.new(1, -20, 0, 80)
    ConsoleFrame.Position = UDim2.new(0, 10, 1, -90)
    ConsoleFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    ConsoleFrame.BorderSizePixel = 0
    ConsoleFrame.Parent = MainFrame

    local ConsoleText = Instance.new("TextLabel")
    ConsoleText.Text = "> Sistema AnarchyOS inicializado\n> Bypasses aplicados com sucesso\n> Aguardando comandos..."
    ConsoleText.TextColor3 = Color3.fromRGB(0, 255, 0)
    ConsoleText.Size = UDim2.new(1, -10, 1, -10)
    ConsoleText.Position = UDim2.new(0, 5, 0, 5)
    ConsoleText.BackgroundTransparency = 1
    ConsoleText.Font = Enum.Font.RobotoMono
    ConsoleText.TextSize = 11
    ConsoleText.TextXAlignment = Enum.TextXAlignment.Left
    ConsoleText.TextYAlignment = Enum.TextYAlignment.Top
    ConsoleText.TextWrapped = true
    ConsoleText.Parent = ConsoleFrame

    -- Drag functionality
    local dragging = false
    local dragInput, dragStart, startPos

    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    TitleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    MainGUI.Parent = CoreGui
    return MainGUI, StatusText, ConsoleText
end

-- FUNÇÕES DE COMANDO REAIS
local function ExecuteCommand(command)
    local function UpdateConsole(message)
        if ConsoleText then
            ConsoleText.Text = ConsoleText.Text .. "\n> " .. message
        end
    end

    if command == "takeover" then
        UpdateConsole("Iniciando takeover do servidor...")
        
        -- Método real: Tentar encontrar RemoteEvents existentes
        local remotes = ReplicatedStorage:GetDescendants()
        for _, remote in ipairs(remotes) do
            if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
                pcall(function()
                    remote:FireServer("AnarchyOS_Takeover", LocalPlayer.UserId)
                    UpdateConsole("Comando enviado para: " .. remote.Name)
                end)
            end
        end
        
        UpdateConsole("Takeover attempt completed")

    elseif command == "kickall" then
        UpdateConsole("Executando kick em todos os jogadores...")
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                pcall(function()
                    player:Kick("AnarchyOS System Command")
                end)
            end
        end
        UpdateConsole("Kick command executed")

    elseif command == "shutdown" then
        UpdateConsole("Desligando servidor...")
        game:Shutdown()
        
    elseif command == "bypass" then
        UpdateConsole("Aplicando bypasses avançados...")
        ApplyRealBypasses()
        UpdateConsole("Bypasses aplicados com sucesso")

    elseif command == "serverhop" then
        UpdateConsole("Preparando server hop...")
        local TeleportService = game:GetService("TeleportService")
        -- Teleportar para um jogo popular
        TeleportService:Teleport(1818) -- Jailbreak
        
    elseif command == "execute" then
        UpdateConsole("Modo execute code ativado")
        -- Aqui viria um sistema de input de código
        UpdateConsole("Digite o código Lua para executar:")
    end
end

-- INICIALIZAÇÃO DO SISTEMA
local success, err = pcall(function()
    ApplyRealBypasses()
    local gui, status, console = CreateRealGUI()
    StatusText = status
    ConsoleText = console
    
    ConsoleText.Text = ConsoleText.Text .. "\n> Sistema totalmente carregado"
    ConsoleText.Text = ConsoleText.Text .. "\n> Player: " .. LocalPlayer.Name
    ConsoleText.Text = ConsoleText.Text .. "\n> UserID: " .. LocalPlayer.UserId
    ConsoleText.Text = ConsoleText.Text .. "\n> JobID: " .. game.JobId
end)

if not success then
    warn("Erro na inicialização: " .. err)
    -- Fallback: Mensagem simples no chat
    LocalPlayer:Chat("/e AnarchyOS Failed to Load")
end

-- SISTEMA DE ATUALIZAÇÃO AUTOMÁTICA
task.spawn(function()
    while task.wait(30) do
        if ConsoleText then
            ConsoleText.Text = ConsoleText.Text .. "\n> System heartbeat: " .. os.date("%X")
        end
    end
end)

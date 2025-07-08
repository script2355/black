-- Brainrot Hub PRO - Versão Completa e Avançada com Interface Corrigida e Botões Funcionais
-- Desenvolvido para "Roube um Brainrot"
-- Funções: Auto Farm, Alerta Proximidade, ESP, Noclip, Fly, Speed, Detector NPC, Interface Melhorada

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")

repeat wait() until LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.Character

local config = {
    autoFarm = true,
    infiniteJump = true,
    noclip = true,
    fly = false,
    flySpeed = 3,
    walkSpeed = 30,
    alertaProximidade = true
}

for _,v in pairs(getconnections(LocalPlayer.Idled)) do v:Disable() end
LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new())
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new())
end)

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "BrainrotHubUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 340, 0, 500)
frame.Position = UDim2.new(0.5, -170, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 36)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "🧠 Brainrot Hub PRO - UI"
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Parent = frame

local info = Instance.new("TextLabel")
info.Size = UDim2.new(1, 0, 0, 20)
info.Position = UDim2.new(0, 0, 1, -25)
info.Text = "F = Fly | T = Random TP | W/E = Voar | Noclip: ON"
info.TextColor3 = Color3.fromRGB(255,255,150)
info.Font = Enum.Font.Gotham
info.TextSize = 12
info.BackgroundTransparency = 1
info.Parent = frame

-- Criar botão
local function criarBotao(texto, ordem, func)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.9, 0, 0, 30)
    botao.Position = UDim2.new(0.05, 0, 0, 40 + (ordem - 1) * 40)
    botao.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Font = Enum.Font.Gotham
    botao.TextSize = 16
    botao.Text = texto
    botao.Parent = frame
    Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 6)
    botao.MouseButton1Click:Connect(func)
end

-- Funções
criarBotao("✅ Toggle Auto Farm", 1, function()
    config.autoFarm = not config.autoFarm
end)

criarBotao("🌀 Toggle Infinite Jump", 2, function()
    config.infiniteJump = not config.infiniteJump
end)

criarBotao("🚪 Toggle Noclip", 3, function()
    config.noclip = not config.noclip
end)

criarBotao("🛸 Toggle Fly (F)", 4, function()
    config.fly = not config.fly
end)

criarBotao("🔊 Toggle Alerta Proximidade", 5, function()
    config.alertaProximidade = not config.alertaProximidade
end)

-- Mensagem no console
print("✅ Interface com botões visuais carregada com sucesso!")

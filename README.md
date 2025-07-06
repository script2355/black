local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Criar GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "BrainrotInterface"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false
gui.Enabled = true

-- Criar janela principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 180)
frame.Position = UDim2.new(0.5, -140, 0.5, -90)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.15
frame.Active = true
frame.Draggable = true
frame.Parent = gui
frame.ClipsDescendants = true

-- Cantos arredondados
local uicorner = Instance.new("UICorner", frame)
uicorner.CornerRadius = UDim.new(0, 12)

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "Brainrot Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = frame

-- Criar botão
local function criarBotao(texto, ordem, callback)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.9, 0, 0, 32)
    botao.Position = UDim2.new(0.05, 0, 0, 35 + (ordem - 1) * 38)
    botao.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Font = Enum.Font.Gotham
    botao.TextSize = 16
    botao.Text = texto
    botao.AutoButtonColor = true
    botao.Parent = frame
    Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 6)
    botao.MouseButton1Click:Connect(callback)
end

-- Função: Roubar Brainrot (simula G)
local function roubarBrainrot()
    local prompt
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("Model") then
            prompt = obj
            break
        end
    end
    if prompt then
        fireproximityprompt(prompt, 0)
    end
end

-- Função: Teleporte para o céu
local function teleportarCeu()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(0, 1000, 0)
    end
end

-- Função: Descer
local function descer()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 5, 0)
    end
end

-- Noclip toggle
local noclip = false
local RunService = game:GetService("RunService")
RunService.Stepped:Connect(function()
    if noclip then
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid"):ChangeState(11)
        end
    end
end)

-- Botões na interface
criarBotao("🔓 Roubar Brainrot (G)", 1, roubarBrainrot)
criarBotao("☁️ Teleporte para o Céu (C)", 2, teleportarCeu)
criarBotao("⬇️ Descer (Q)", 3, descer)
criarBotao("🚪 Toggle Noclip (N)", 4, function() noclip = not noclip end)

-- Atalhos de teclado
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    elseif input.KeyCode == Enum.KeyCode.G then
        roubarBrainrot()
    elseif input.KeyCode == Enum.KeyCode.C then
        teleportarCeu()
    elseif input.KeyCode == Enum.KeyCode.Q then
        descer()
    elseif input.KeyCode == Enum.KeyCode.N then
        noclip = not noclip
    end
end)

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "BrainrotInterface"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false
gui.Enabled = true

-- Janela principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 260)
frame.Position = UDim2.new(0.5, -140, 0.5, -130)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.15
frame.Active = true
frame.Draggable = true
frame.Parent = gui
frame.ClipsDescendants = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

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
local ordem = 0
local function criarBotao(texto, callback)
	ordem += 1
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
	return botao
end

-- Funções de movimentação
local function teleportarCeu()
	local char = LocalPlayer.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(0, 1000, 0)
	end
end

local function descer()
	local char = LocalPlayer.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 5, 0)
	end
end

local function descer50()
	local char = LocalPlayer.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 50, 0)
	end
end

-- Pulo infinito
local infiniteJumpEnabled = false

UserInputService.InputBegan:Connect(function(input, isTyping)
	if isTyping then return end
	if input.KeyCode == Enum.KeyCode.Space and infiniteJumpEnabled then
		local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- Criar botões
criarBotao("☁️ Teleporte para o Céu (C)", teleportarCeu)
criarBotao("⬇️ Descer 5 (Q)", descer)
criarBotao("⬇️ Descer 50 (V)", descer50)

-- Botão pulo infinito
local puloBotao = criarBotao("🦘 Pulo Infinito: OFF", function()
	infiniteJumpEnabled = not infiniteJumpEnabled
	puloBotao.Text = infiniteJumpEnabled and "🦘 Pulo Infinito: ON" or "🦘 Pulo Infinito: OFF"
end)

-- Temporizador base
local timerLabel = Instance.new("TextLabel")
timerLabel.Size = UDim2.new(0.9, 0, 0, 30)
timerLabel.Position = UDim2.new(0.05, 0, 1, -35)
timerLabel.BackgroundTransparency = 1
timerLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
timerLabel.Font = Enum.Font.GothamBold
timerLabel.TextSize = 18
timerLabel.Text = "⌛ Tempo restante: carregando..."
timerLabel.Parent = frame

local tempoTotal = 300
local tempoRestante = tempoTotal

task.spawn(function()
	while true do
		if tempoRestante <= 0 then
			timerLabel.Text = "✅ Bases abertas!"
			wait(5)
			tempoRestante = tempoTotal
		else
			timerLabel.Text = "⌛ Tempo restante: " .. tostring(tempoRestante) .. "s"
			tempoRestante -= 1
		end
		wait(1)
	end
end)

-- ESP jogadores
local function criarESP(jogador)
	if jogador == LocalPlayer then return end
	local function adicionar()
		local character = jogador.Character or jogador.CharacterAdded:Wait()
		local head = character:WaitForChild("Head", 5)
		if not head then return end

		local billboard = Instance.new("BillboardGui")
		billboard.Name = "ESP_" .. jogador.Name
		billboard.Adornee = head
		billboard.Size = UDim2.new(0, 100, 0, 30)
		billboard.StudsOffset = Vector3.new(0, 2, 0)
		billboard.AlwaysOnTop = true
		billboard.Parent = head

		local label = Instance.new("TextLabel")
		label.Size = UDim2.new(1, 0, 1, 0)
		label.BackgroundTransparency = 1
		label.Text = jogador.Name
		label.TextColor3 = Color3.fromRGB(255, 255, 255)
		label.TextStrokeTransparency = 0.5
		label.Font = Enum.Font.GothamBold
		label.TextSize = 14
		label.Parent = billboard
	end

	adicionar()
	jogador.CharacterAdded:Connect(function()
		wait(1)
		adicionar()
	end)
end

-- ESP inicial + novos jogadores
for _, jogador in pairs(Players:GetPlayers()) do
	criarESP(jogador)
end
Players.PlayerAdded:Connect(criarESP)

-- Teclas atalho
UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.F then
		gui.Enabled = not gui.Enabled
	elseif input.KeyCode == Enum.KeyCode.C then
		teleportarCeu()
	elseif input.KeyCode == Enum.KeyCode.Q then
		descer()
	elseif input.KeyCode == Enum.KeyCode.V then
		descer50()
	end
end)

-- Brainrot Hub PRO - Edição Turbo com Funções Completas
-- Desenvolvido para "Roube um Brainrot" com funções extras e automações avançadas

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

repeat wait() until LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.Character

local config = {
    autoFarm = false,
    infiniteJump = false,
    noclip = false,
    fly = false,
    flySpeed = 3,
    walkSpeed = 30,
    alertaProximidade = false,
    randomTP = false,
    espPlayers = false,
    speedBoost = false,
    floatToBase = false
}

for _,v in pairs(getconnections(LocalPlayer.Idled)) do v:Disable() end
LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new())
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new())
end)

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "BrainrotHubUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 360, 0, 600)
frame.Position = UDim2.new(0.5, -180, 0.5, -300)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 36)
title.Text = "🧠 Brainrot Hub PRO Turbo"
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1

local info = Instance.new("TextLabel", frame)
info.Size = UDim2.new(1, 0, 0, 20)
info.Position = UDim2.new(0, 0, 1, -25)
info.Text = "F = Fly | T = Random TP | Q = Boost | V = Float To Base"
info.TextColor3 = Color3.fromRGB(255,255,150)
info.Font = Enum.Font.Gotham
info.TextSize = 12
info.BackgroundTransparency = 1
info.Parent = frame

-- Funções contínuas
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if config.autoFarm then
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                pcall(function() fireproximityprompt(obj) end)
            end
        end
    end
    if config.noclip and char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid"):ChangeState(11)
    end
    if config.fly and hrp then
        hrp.Velocity = Vector3.new(0, config.flySpeed * 50, 0)
    end
    if config.speedBoost and hrp then
        hrp.CFrame = hrp.CFrame + hrp.CFrame.LookVector * 2
    end
    if config.floatToBase and hrp then
        local goal = Vector3.new(0, 100, 0)
        hrp.CFrame = hrp.CFrame:Lerp(CFrame.new(goal), 0.01)
    end
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space and config.infiniteJump then
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
    if input.KeyCode == Enum.KeyCode.T then
        local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            root.CFrame = CFrame.new(math.random(-500,500), 50, math.random(-500,500))
        end
    end
    if input.KeyCode == Enum.KeyCode.Q then
        config.speedBoost = not config.speedBoost
    end
    if input.KeyCode == Enum.KeyCode.V then
        config.floatToBase = not config.floatToBase
    end
end)

-- ESP jogadores
local function criarESP()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") and not p.Character.Head:FindFirstChild("ESP") then
            local esp = Instance.new("BillboardGui", p.Character.Head)
            esp.Name = "ESP"
            esp.Size = UDim2.new(0,100,0,40)
            esp.AlwaysOnTop = true
            local texto = Instance.new("TextLabel", esp)
            texto.Size = UDim2.new(1,0,1,0)
            texto.BackgroundTransparency = 1
            texto.Text = p.DisplayName
            texto.TextColor3 = Color3.new(1,0,0)
            texto.TextScaled = true
        end
    end
end
RunService.RenderStepped:Connect(function()
    if config.espPlayers then criarESP() end
end)

-- Botões
local function criarBotao(texto, ordem, chave)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.9, 0, 0, 30)
    botao.Position = UDim2.new(0.05, 0, 0, 40 + (ordem - 1) * 40)
    botao.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Font = Enum.Font.Gotham
    botao.TextSize = 16
    botao.Text = texto .. ": OFF"
    botao.Parent = frame
    Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 6)

    botao.MouseButton1Click:Connect(function()
        config[chave] = not config[chave]
        botao.Text = texto .. ": " .. (config[chave] and "ON" or "OFF")
    end)
end

criarBotao("✅ Auto Farm", 1, "autoFarm")
criarBotao("🌀 Infinite Jump", 2, "infiniteJump")
criarBotao("🚪 Noclip", 3, "noclip")
criarBotao("🛸 Fly", 4, "fly")
criarBotao("🔊 Alerta Proximidade", 5, "alertaProximidade")
criarBotao("👁️ ESP Jogadores", 6, "espPlayers")
criarBotao("💨 Boost Velocidade (Q)", 7, "speedBoost")
criarBotao("🚀 Flutuar até Base (V)", 8, "floatToBase")

print("✅ Brainrot Hub Turbo Carregado com Sucesso!")

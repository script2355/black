-- Brainrot Hub PRO - Todas as Funções Ativadas
-- Desenvolvido para jogos como "Roube um Brainrot"
-- Script modular com ESP, Auto Farm, Anti AFK, Fly, Teleportes, Proteção e mais

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")

repeat wait() until LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.Character

-- Proteção Anti-Detecção
pcall(function()
    hookfunction(getfenv().print, function(...) end)
end)

-- Configuração
local config = {
    autoFarm = false,
    infiniteJump = false,
    noclip = false,
    fly = false,
    flySpeed = 2,
    walkSpeed = 24,
    chams = true,
    antiAFK = true
}

-- Anti-AFK
if config.antiAFK then
    for _,v in pairs(getconnections(LocalPlayer.Idled)) do v:Disable() end
    LocalPlayer.Idled:Connect(function()
        VirtualUser:Button2Down(Vector2.new())
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new())
    end)
end

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "BRHub" .. math.random(1000, 9999)
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Janela
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 500)
frame.Position = UDim2.new(0.5, -160, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "🧠 Brainrot Hub PRO"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.BackgroundTransparency = 1
title.Parent = frame

-- Fly system
local flyVelocity = Instance.new("BodyVelocity")
local flying = false
local function toggleFly()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    flying = not flying
    if flying then
        flyVelocity.Velocity = Vector3.zero
        flyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
        flyVelocity.Parent = char.HumanoidRootPart
        RunService:BindToRenderStep("FlyControl", Enum.RenderPriority.Input.Value, function()
            local vel = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then vel += Workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then vel -= Workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.E) then vel += Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.Q) then vel -= Vector3.new(0,1,0) end
            flyVelocity.Velocity = vel.Unit * config.flySpeed * 50
        end)
    else
        RunService:UnbindFromRenderStep("FlyControl")
        flyVelocity.Parent = nil
    end
end

-- Noclip
RunService.Stepped:Connect(function()
    if config.noclip then
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid"):ChangeState(11)
        end
    end
end)

-- WalkSpeed
LocalPlayer.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").WalkSpeed = config.walkSpeed
end)

-- Infinite Jump
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Space and config.infiniteJump then
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- Auto Farm
task.spawn(function()
    while task.wait(1.5) do
        if config.autoFarm then
            for _, obj in pairs(Workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("Model") then
                    pcall(function()
                        fireproximityprompt(obj)
                    end)
                end
            end
        end
    end
end)

-- ESP jogadores
local function criarESP(player)
    if player == LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        local head = char:WaitForChild("Head")
        local esp = Instance.new("BillboardGui")
        esp.Size = UDim2.new(0, 100, 0, 20)
        esp.AlwaysOnTop = true
        esp.Adornee = head
        esp.Name = "PlayerESP"
        esp.Parent = head

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.TextStrokeTransparency = 0.5
        label.Font = Enum.Font.GothamBold
        label.TextSize = 14
        label.Text = player.Name
        label.Parent = esp
    end)
end
for _,p in pairs(Players:GetPlayers()) do criarESP(p) end
Players.PlayerAdded:Connect(criarESP)

-- Interface de botões
local ordem = 0
local function criarBotao(texto, func)
    ordem += 1
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.9, 0, 0, 30)
    botao.Position = UDim2.new(0.05, 0, 0, 35 + (ordem - 1) * 35)
    botao.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    botao.TextColor3 = Color3.new(1, 1, 1)
    botao.Text = texto
    botao.Font = Enum.Font.Gotham
    botao.TextSize = 14
    botao.AutoButtonColor = false
    botao.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = botao

    local function animateHover(enter)
        local goalColor = enter and Color3.fromRGB(70, 70, 70) or Color3.fromRGB(40, 40, 40)
        TweenService:Create(botao, TweenInfo.new(0.2), {BackgroundColor3 = goalColor}):Play()
    end

    botao.MouseEnter:Connect(function() animateHover(true) end)
    botao.MouseLeave:Connect(function() animateHover(false) end)

    local sound = Instance.new("Sound", botao)
    sound.SoundId = "rbxassetid://12221967"
    sound.Volume = 0.5

    botao.MouseButton1Click:Connect(function()
        sound:Play()
        func()
    end)
end

criarBotao("🧠 Auto Farm", function()
    config.autoFarm = not config.autoFarm
end)
criarBotao("🦘 Pulo Infinito", function()
    config.infiniteJump = not config.infiniteJump
end)
criarBotao("🚷 Noclip", function()
    config.noclip = not config.noclip
end)
criarBotao("🕊️ Fly Mode (X)", function()
    toggleFly()
end)
criarBotao("🏃 WalkSpeed +10", function()
    config.walkSpeed += 10
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
    if hum then hum.WalkSpeed = config.walkSpeed end
end)
criarBotao("🔄 Resetar Personagem", function()
    LocalPlayer:LoadCharacter()
end)

-- Teclas rápidas
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    elseif input.KeyCode == Enum.KeyCode.X then
        toggleFly()
    end
end)

-- Brainrot Hub PRO Avançado - Roube um Brainrot

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local ChatService = game:GetService("StarterGui")

-- Espera personagem carregar
repeat task.wait() until LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
local character = LocalPlayer.Character

-- Configuração Inicial
local Config = {
    AutoFarm = false,
    InfiniteJump = false,
    NoClip = false,
    Fly = false,
    FlySpeed = 3,
    WalkSpeed = 20,
    WalkSpeedBoost = 60,
    IsBoosting = false,
    TeleportCooldown = false,
}

-- Função para log no chat
local function Log(message, color)
    color = color or Color3.new(1,1,1)
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "[Brainrot Hub] "..message;
        Color = color;
        Font = Enum.Font.GothamBold;
        FontSize = Enum.FontSize.Size24;
    })
end

-- Anti AFK
for _,conn in pairs(getconnections(LocalPlayer.Idled)) do
    conn:Disable()
end
LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new())
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new())
end)

-- GUI Setup
local gui = Instance.new("ScreenGui")
gui.Name = "BrainrotHubProAdvanced"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.IgnoreGuiInset = true

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 350, 0, 520)
frame.Position = UDim2.new(0.5, -175, 0.5, -260)
frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 26
title.Text = "🧠 Brainrot Hub PRO Avançado"
title.Parent = frame

local ordem = 0
local function criarBotao(texto, ativo, callback)
    ordem += 1
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.9, 0, 0, 38)
    botao.Position = UDim2.new(0.05, 0, 0, 45 + (ordem-1)*44)
    botao.BackgroundColor3 = ativo and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
    botao.TextColor3 = Color3.new(1,1,1)
    botao.Font = Enum.Font.GothamBold
    botao.TextSize = 18
    botao.Text = texto
    botao.AutoButtonColor = false
    botao.Parent = frame
    Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 8)

    local function atualizarCor(ativo)
        botao.BackgroundColor3 = ativo and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
    end

    botao.MouseButton1Click:Connect(function()
        ativo = not ativo
        atualizarCor(ativo)
        callback(ativo)
    end)

    return botao
end

-- Variáveis e funções para WalkSpeed + Boost
local walkSpeedNormal = Config.WalkSpeed
local walkSpeedBoost = Config.WalkSpeedBoost
local boosting = false

local function atualizarWalkSpeed()
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.WalkSpeed = boosting and walkSpeedBoost or walkSpeedNormal
    end
end

-- Fly
local flyVelocity = Instance.new("BodyVelocity")
local flyEnabled = false
local function toggleFly(on)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    flyEnabled = on
    if flyEnabled then
        flyVelocity.Velocity = Vector3.new(0,0,0)
        flyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
        flyVelocity.Parent = char.HumanoidRootPart
        RunService:BindToRenderStep("FlyControl", Enum.RenderPriority.Input.Value, function()
            local vel = Vector3.new(0,0,0)
            local cam = Workspace.CurrentCamera.CFrame
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then vel += cam.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then vel -= cam.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then vel += cam.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then vel -= cam.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.E) then vel += Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.Q) then vel -= Vector3.new(0,1,0) end

            if vel.Magnitude > 0 then
                flyVelocity.Velocity = vel.Unit * Config.FlySpeed * 75
            else
                flyVelocity.Velocity = Vector3.new(0,0,0)
            end
        end)
        Log("Fly ativado", Color3.fromRGB(50,200,50))
    else
        RunService:UnbindFromRenderStep("FlyControl")
        flyVelocity.Parent = nil
        Log("Fly desativado", Color3.fromRGB(200,50,50))
    end
end

-- Noclip
RunService.Stepped:Connect(function()
    if Config.NoClip then
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Physics)
        end
    end
end)

-- Infinite Jump
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space and Config.InfiniteJump then
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- Auto Farm
local function roubarBrainrot()
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("Model") then
            pcall(function()
                fireproximityprompt(obj)
            end)
        end
    end
end

task.spawn(function()
    while true do
        task.wait(1.5)
        if Config.AutoFarm then
            roubarBrainrot()
        end
    end
end)

-- ESP jogadores com distância
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESPFolder"
ESPFolder.Parent = gui

local function criarESP(player)
    if player == LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        local head = char:WaitForChild("Head")
        local esp = Instance.new("BillboardGui")
        esp.Size = UDim2.new(0,150,0,40)
        esp.Adornee = head
        esp.AlwaysOnTop = true
        esp.Name = "ESP"
        esp.Parent = ESPFolder

        local label = Instance.new("TextLabel")
        label.BackgroundTransparency = 1
        label.Size = UDim2.new(1,0,1,0)
        label.Font = Enum.Font.GothamBold
        label.TextSize = 18
        label.TextColor3 = Color3.fromRGB(255,255,255)
        label.TextStrokeColor3 = Color3.new(0,0,0)
        label.TextStrokeTransparency = 0
        label.Parent = esp

        RunService.RenderStepped:Connect(function()
            if head.Parent then
                local distance = (head.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                label.Text = string.format("%s\n%.1f m", player.Name, distance)
            else
                esp:Destroy()
            end
        end)
    end)
end

for _,p in pairs(Players:GetPlayers()) do
    criarESP(p)
end
Players.PlayerAdded:Connect(criarESP)

-- Velocidade & Boost
local function setWalkSpeed(speed)
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
    if hum then
        hum.WalkSpeed = speed
    end
end

local function toggleBoost(on)
    Config.IsBoosting = on
    setWalkSpeed(on and Config.WalkSpeedBoost or Config.WalkSpeed)
    Log(on and "Boost de velocidade ativado" or "Boost de velocidade desativado", on and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
end

-- Interface botões
local btnAutoFarm = criarBotao("🧠 Auto Farm", Config.AutoFarm, function(state)
    Config.AutoFarm = state
    Log("Auto Farm " .. (state and "ativado" or "desativado"), state and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
end)

local btnPuloInfinito = criarBotao("🦘 Pulo Infinito", Config.InfiniteJump, function(state)
    Config.InfiniteJump = state
    Log("Pulo infinito " .. (state and "ativado" or "desativado"), state and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
end)

local btnNoclip = criarBotao("🚷 Noclip", Config.NoClip, function(state)
    Config.NoClip = state
    Log("Noclip " .. (state and "ativado" or "desativado"), state and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
end)

local btnFly = criarBotao("🕊️ Fly Mode", Config.Fly, function(state)
    Config.Fly = state
    toggleFly(state)
end)

local btnBoostSpeed = criarBotao("⚡ Boost de Velocidade (Q)", false, function(state)
    -- Apenas para mostrar toggle (boost é ativado pela tecla Q)
    Log("Boost de velocidade habilitado (use Q para ativar/desativar)", Color3.fromRGB(40,200,40))
end)

local btnWalkSpeedUp = criarBotao("🏃 WalkSpeed +10", false, function()
    Config.WalkSpeed += 10
    if not Config.IsBoosting then
        setWalkSpeed(Config.WalkSpeed)
    end
    Log("WalkSpeed aumentado para " .. Config.WalkSpeed, Color3.fromRGB(40,200,40))
end)

local btnResetSpeed = criarBotao("🔄 Resetar Velocidade", false, function()
    Config.WalkSpeed = 20
    if not Config.IsBoosting then
        setWalkSpeed(Config.WalkSpeed)
    end
    Log("WalkSpeed resetada para 20", Color3.fromRGB(255,255,0))
end)

local btnResetChar = criarBotao("🔄 Resetar Personagem", false, function()
    LocalPlayer:LoadCharacter()
    Log("Personagem resetado", Color3.fromRGB(255,255,0))
end)

local btnTeleportUp = criarBotao("📦 Teleporte para o Céu (C)", false, function()
    if Config.TeleportCooldown then
        Log("Teleport em cooldown, aguarde...", Color3.fromRGB(255,100,100))
        return
    end
    Config.TeleportCooldown = true
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame + Vector3.new(0, 1000, 0)
        Log("Teleporte para o céu realizado", Color3.fromRGB(100,255,100))
    end
    task.delay(3, function()
        Config.TeleportCooldown = false
    end)
end)

local btnTeleportDown = criarBotao("⬇️ Teleporte para Baixo (V)", false, function()
    if Config.TeleportCooldown then
        Log("Teleport em cooldown, aguarde...", Color3.fromRGB(255,100,100))
        return
    end
    Config.TeleportCooldown = true
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame - Vector3.new(0, 100, 0)
        Log("Teleporte para baixo realizado", Color3.fromRGB(100,255,100))
    end
    task.delay(3, function()
        Config.TeleportCooldown = false
    end)
end)

-- Toggle GUI com F
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
        Log("Interface "..(gui.Enabled and "aberta" or "fechada"), Color3.fromRGB(150,150,150))
    elseif input.KeyCode == Enum.KeyCode.Q then
        boosting = not boosting
        toggleBoost(boosting)
    elseif input.KeyCode == Enum.KeyCode.C then
        btnTeleportUp.MouseButton1Click:Fire()
    elseif input.KeyCode == Enum.KeyCode.V then
        btnTeleportDown.MouseButton1Click:Fire()
    elseif input.KeyCode == Enum.KeyCode.X then
        Config.Fly = not Config.Fly
        toggleFly(Config.Fly)
        btnFly.BackgroundColor3 = Config.Fly and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
    elseif input.KeyCode == Enum.KeyCode.Z then
        Config.AutoFarm = not Config.AutoFarm
        btnAutoFarm.BackgroundColor3 = Config.AutoFarm and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
        Log("Auto Farm "..(Config.AutoFarm and "ativado" or "desativado"), Config.AutoFarm and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
    elseif input.KeyCode == Enum.KeyCode.N then
        Config.NoClip = not Config.NoClip
        btnNoclip.BackgroundColor3 = Config.NoClip and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
        Log("Noclip "..(Config.NoClip and "ativado" or "desativado"), Config.NoClip and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
    elseif input.KeyCode == Enum.KeyCode.CapsLock then
        Config.InfiniteJump = not Config.InfiniteJump
        btnPuloInfinito.BackgroundColor3 = Config.InfiniteJump and Color3.fromRGB(40,100,40) or Color3.fromRGB(80,80,80)
        Log("Pulo infinito "..(Config.InfiniteJump and "ativado" or "desativado"), Config.InfiniteJump and Color3.fromRGB(40,200,40) or Color3.fromRGB(200,50,50))
    elseif input.KeyCode == Enum.KeyCode.L then
        Config.WalkSpeed += 10
        if not Config.IsBoosting then
            setWalkSpeed(Config.WalkSpeed)
        end
        Log("WalkSpeed aumentado para "..Config.WalkSpeed, Color3.fromRGB(40,200,40))
    elseif input.KeyCode == Enum.KeyCode.R then
        LocalPlayer:LoadCharacter()
        Log("Personagem resetado", Color3.fromRGB(255,255,0))
    end
end)

-- Resetar walk speed no spawn
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(2)
    setWalkSpeed(Config.WalkSpeed)
    boosting = false
end)

Log("Brainrot Hub PRO Avançado carregado! Aperte F para abrir o menu", Color3.fromRGB(100,255,100))

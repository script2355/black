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

-- Janela principal
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

-- Função Log para mensagens no chat (útil para feedback)
local function Log(msg, cor)
    cor = cor or Color3.new(1,1,1)
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "[Brainrot Hub PRO] "..msg;
        Color = cor;
        Font = Enum.Font.SourceSansBold;
        FontSize = Enum.FontSize.Size24;
    })
end

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
        Log("Fly ativado", Color3.fromRGB(50,255,50))
    else
        RunService:UnbindFromRenderStep("FlyControl")
        flyVelocity.Parent = nil
        Log("Fly desativado", Color3.fromRGB(255,50,50))
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
local function setWalkSpeed(speed)
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = speed end
    end
end

setWalkSpeed(config.walkSpeed)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(2)
    setWalkSpeed(config.walkSpeed)
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

-- ESP avançado com barra de vida e distância
local ESPFolder = Instance.new("Folder", gui)
ESPFolder.Name = "ESPFolder"

local function criarESP(player)
    if player == LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        local head = char:WaitForChild("Head")
        local humanoid = char:WaitForChild("Humanoid")

        local esp = Instance.new("BillboardGui")
        esp.Name = "ESP"
        esp.Adornee = head
        esp.AlwaysOnTop = true
        esp.Size = UDim2.new(0, 140, 0, 50)
        esp.Parent = ESPFolder

        local nameLabel = Instance.new("TextLabel", esp)
        nameLabel.Size = UDim2.new(1, 0, 0, 18)
        nameLabel.Position = UDim2.new(0, 0, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Color3.new(1,1,1)
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.TextSize = 16
        nameLabel.TextStrokeTransparency = 0.5

        local distanceLabel = Instance.new("TextLabel", esp)
        distanceLabel.Size = UDim2.new(1, 0, 0, 14)
        distanceLabel.Position = UDim2.new(0, 0, 0, 20)
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.TextColor3 = Color3.new(0.8,0.8,0.8)
        distanceLabel.Font = Enum.Font.Gotham
        distanceLabel.TextSize = 12

        local healthBarBack = Instance.new("Frame", esp)
        healthBarBack.Size = UDim2.new(0.8, 0, 0, 6)
        healthBarBack.Position = UDim2.new(0.1, 0, 0, 38)
        healthBarBack.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
        healthBarBack.BorderSizePixel = 0
        healthBarBack.ClipsDescendants = true
        healthBarBack.AnchorPoint = Vector2.new(0,0)

        local healthBar = Instance.new("Frame", healthBarBack)
        healthBar.Size = UDim2.new(1, 0, 1, 0)
        healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        healthBar.BorderSizePixel = 0

        -- Atualizar dados na renderstep
        local updateConnection
        updateConnection = RunService.RenderStepped:Connect(function()
            if not char or not char.Parent or humanoid.Health <= 0 then
                esp:Destroy()
                updateConnection:Disconnect()
                return
            end
            local distance = (head.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            nameLabel.Text = player.Name
            distanceLabel.Text = string.format("Distância: %.1f m", distance)
            healthBar.Size = UDim2.new(math.clamp(humanoid.Health/humanoid.MaxHealth,0,1), 0, 1, 0)

            -- Cor da barra de vida - verde para alto, vermelho para baixo
            local healthPercent = humanoid.Health/humanoid.MaxHealth
            healthBar.BackgroundColor3 = Color3.fromHSV(healthPercent * 0.33,1,1)
        end)
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    criarESP(player)
end
Players.PlayerAdded:Connect(criarESP)

-- Funções de Teleporte e boost para base voando
local boostingToBase = false
local tween -- Para guardar tween e poder cancelar

-- Posicione aqui a posição da sua base (substitua pelos valores corretos)
local basePosition = Vector3.new(0, 10, 0)

local function startBoostToBase()
    if boostingToBase then return end
    boostingToBase = true
    Log("Boost ativado: Voando para a base...", Color3.fromRGB(50, 255, 50))

    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        boostingToBase = false
        return
    end

    tween = TweenService:Create(hrp, TweenInfo.new(5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = CFrame.new(basePosition + Vector3.new(0,5,0))})
    tween:Play()
    tween.Completed:Connect(function()
        Log("Chegou na base!", Color3.fromRGB(100, 255, 100))
        boostingToBase = false
    end)
end

local function stopBoostToBase()
    if tween then tween:Cancel() end
    boostingToBase = false
    Log("Boost desativado", Color3.fromRGB(255, 50, 50))
end

-- Teleporte para o céu
local function teleportarParaCeu()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(0, 1000, 0)
        Log("Teleporte para o céu executado", Color3.fromRGB(100, 255, 100))
    end
end

-- Teleporte para baixo (100 unidades)
local function descer100()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 100, 0)
        Log("Descida de 100 unidades executada", Color3.fromRGB(100, 255, 100))
    end
end

-- Noclip toggle
local noclip = false
RunService.Stepped:Connect(function()
    if noclip then
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid"):ChangeState(11)
        end
    end
end)

-- Botões na interface
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

-- Criar botões na interface
criarBotao("🧠 Auto Farm", function()
    config.autoFarm = not config.autoFarm
    Log("Auto Farm: "..tostring(config.autoFarm), Color3.fromRGB(255,255,0))
end)
criarBotao("🦘 Pulo Infinito", function()
    config.infiniteJump = not config.infiniteJump
    Log("Pulo Infinito: "..tostring(config.infiniteJump), Color3.fromRGB(255,255,0))
end)
criarBotao("🚷 Noclip", function()
    noclip = not noclip
    Log("Noclip: "..tostring(noclip), Color3.fromRGB(255,255,0))
end)
criarBotao("🕊️ Fly Mode (X)", function()
    toggleFly()
end)
criarBotao("🏃 WalkSpeed +10", function()
    config.walkSpeed += 20
    setWalkSpeed(config.walkSpeed)
    Log("WalkSpeed aumentado para "..config.walkSpeed, Color3.fromRGB(255,255,0))
end)
criarBotao("🔄 Resetar Personagem", function()
    LocalPlayer:LoadCharacter()
    Log("Personagem resetado", Color3.fromRGB(255,255,0))
end)
criarBotao("🚀 Boost Voar para Base (Q)", function()
    if boostingToBase then
        stopBoostToBase()
    else
        startBoostToBase()
    end
end)
criarBotao("📦 Teleporte Base 1", function()
    teleportarPara(basePosition)
    Log("Teleportado para base", Color3.fromRGB(255,255,0))
end)
criarBotao("☁️ Teleporte para o Céu (C)", teleportarParaCeu)
criarBotao("⬇️ Descer 100 (V)", descer100)

-- Função para teleporte genérico (usar para botões se quiser)
local function teleportarPara(pos)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char:MoveTo(pos)
    end
end

-- Teclas rápidas
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    elseif input.KeyCode == Enum.KeyCode.X then
        toggleFly()
    elseif input.KeyCode == Enum.KeyCode.Z then
        config.autoFarm = not config.autoFarm
        Log("Auto Farm: "..tostring(config.autoFarm), Color3.fromRGB(255,255,0))
    elseif input.KeyCode == Enum.KeyCode.C then
        teleportarParaCeu()
    elseif input.KeyCode == Enum.KeyCode.V then
        descer100()
    elseif input.KeyCode == Enum.KeyCode.Q then
        if boostingToBase then
            stopBoostToBase()
        else
            startBoostToBase()
        end
    elseif input.KeyCode == Enum.KeyCode.N then
        noclip = not noclip
        Log("Noclip: "..tostring(noclip), Color3.fromRGB(255,255,0))
    elseif input.KeyCode == Enum.KeyCode.L then
        config.walkSpeed += 10
        setWalkSpeed(config.walkSpeed)
        Log("WalkSpeed aumentado para "..config.walkSpeed, Color3.fromRGB(255,255,0))
    elseif input.KeyCode == Enum.KeyCode.R then
        LocalPlayer:LoadCharacter()
        Log("Personagem resetado", Color3.fromRGB(255,255,0))
    end
end)

Log("Brainrot Hub PRO Avançado carregado! Aperte F para abrir o menu", Color3.fromRGB(100,255,100))

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local StarterGui = game:GetService("StarterGui")

local ALTURA_CEU = 1000
local ALTURA_DESCIDA = 5
local CHECAR_CHAO = true
local noclipOn = false

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "ProtegidoGUI_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.Enabled = false

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 270, 0, 140)
frame.Position = UDim2.new(0.5, -135, 0.5, -70)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.ZIndex = 9999
frame.Parent = gui

local text = Instance.new("TextLabel")
text.Size = UDim2.new(1, 0, 1, 0)
text.BackgroundTransparency = 1
text.TextColor3 = Color3.fromRGB(200,200,200)
text.Text = "🛡️ Segurança Ativa\n[Espaço] Pulo Infinito\n[C] Teleporte para o Céu\n[Q] Descer com chão\n[V] Teleporte -50\n[N] Noclip\n[G] Roubar Brainrot\n[F] Mostrar/Esconder painel"
text.Font = Enum.Font.SourceSansBold
text.TextSize = 16
text.TextYAlignment = Enum.TextYAlignment.Top
text.Parent = frame

-- ANTI AFK
task.spawn(function()
    while true do
        wait(20 + math.random(5))
        pcall(function()
            local vu = game:GetService("VirtualUser")
            vu:Button2Down(Vector2.new(math.random(0,10), math.random(0,10)), Workspace.CurrentCamera.CFrame)
            wait(0.5 + math.random())
            vu:Button2Up(Vector2.new(math.random(0,10), math.random(0,10)), Workspace.CurrentCamera.CFrame)
        end)
    end
end)

-- Noclip
local function toggleNoclip()
    noclipOn = not noclipOn
    local char = LocalPlayer.Character
    if char then
        for _, part in pairs(char:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = not noclipOn
            end
        end
    end
    StarterGui:SetCore("SendNotification", {
        Title = "Noclip",
        Text = noclipOn and "Ativado" or "Desativado",
        Duration = 2
    })
end

-- Teleporte Céu
local function teleportarParaOCeu()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(
            hrp.Position.X, ALTURA_CEU, hrp.Position.Z,
            hrp.CFrame.LookVector.X, 0, hrp.CFrame.LookVector.Z,
            0, 1, 0,
            -hrp.CFrame.LookVector.Z, 0, hrp.CFrame.LookVector.X
        )
        StarterGui:SetCore("SendNotification", {
            Title = "Teleporte",
            Text = "Teleportado para o céu!",
            Duration = 2
        })
    end
end

-- Descer com Raycast
local function descer()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end

    if hum then hum.PlatformStand = true end

    if CHECAR_CHAO then
        local params = RaycastParams.new()
        params.FilterDescendantsInstances = {char}
        params.FilterType = Enum.RaycastFilterType.Blacklist
        local result = Workspace:Raycast(hrp.Position, Vector3.new(0, -100, 0), params)

        if result and result.Instance and result.Instance.CanCollide then
            hrp.CFrame = CFrame.new(result.Position + Vector3.new(0, 3, 0))
        else
            hrp.CFrame = hrp.CFrame + Vector3.new(0, -ALTURA_DESCIDA, 0)
        end
    else
        hrp.CFrame = hrp.CFrame + Vector3.new(0, -ALTURA_DESCIDA, 0)
    end

    if hum then
        task.delay(0.2, function()
            hum.PlatformStand = false
        end)
    end
end

-- Teleportar -50 pra baixo
local function teleportar50ParaBaixo()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame + Vector3.new(0, -50, 0)
        StarterGui:SetCore("SendNotification", {
            Title = "Teleporte",
            Text = "Desceu 50 unidades.",
            Duration = 2
        })
    end
end

-- Pulo Infinito
UserInputService.JumpRequest:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- Roubar Brainrot com fireproximityprompt
local function segurarPrompt()
    local prompt = nil
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled then
            local distance = (v.Parent.Position - char.HumanoidRootPart.Position).Magnitude
            if distance < 12 then
                prompt = v
                break
            end
        end
    end

    if prompt then
        fireproximityprompt(prompt, 1) -- segura por 1 segundo
        StarterGui:SetCore("SendNotification", {
            Title = "Brainrot",
            Text = "Segurando para roubar...",
            Duration = 2
        })
    else
        StarterGui:SetCore("SendNotification", {
            Title = "Erro",
            Text = "Nenhum Brainrot próximo!",
            Duration = 2
        })
    end
end

-- Teclas
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    elseif input.KeyCode == Enum.KeyCode.C then
        teleportarParaOCeu()
    elseif input.KeyCode == Enum.KeyCode.Q then
        descer()
    elseif input.KeyCode == Enum.KeyCode.V then
        teleportar50ParaBaixo()
    elseif input.KeyCode == Enum.KeyCode.N then
        toggleNoclip()
    elseif input.KeyCode == Enum.KeyCode.G then
        segurarPrompt()
    end
end)

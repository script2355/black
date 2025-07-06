local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local StarterGui = game:GetService("StarterGui")

local ALTURA_CEU = 105 -- Altura para onde será teleportado ao apertar C
local ALTURA_DESCIDA = 100 -- Quanto desce ao apertar Q
local CHECAR_CHAO = true -- Se true, usa Raycast para encontrar chão

--[[ 
    Base com segurança + pulo infinito + teleporte para o céu:
    - Anti-AFK
    - Delay aleatório
    - Checagem segura
    - Simulação de input real
    - Interface protegida por F
    - Pulo infinito (barra de espaço)
    - Teleporte para o céu (C)
    - Descer (Q)
]]

-- ANTI-AFK
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

-- Delay aleatório
local function randomDelay(min, max)
    wait(min + math.random() * (max - min))
end

-- Checagem segura de existência
local function safeCheck(obj)
    return obj ~= nil and obj.Parent ~= nil
end

-- Simulação de pulo
local function simulateJump()
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
    wait(0.13 + math.random() * 0.09)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
end

-- Interface protegida por F
local gui = Instance.new("ScreenGui")
gui.Name = "ProtegidoGUI_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.Enabled = false

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 270, 0, 120)
frame.Position = UDim2.new(0.5, -135, 0.5, -60)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.ZIndex = 9999
frame.Parent = gui

local text = Instance.new("TextLabel")
text.Size = UDim2.new(1, 0, 1, 0)
text.BackgroundTransparency = 1
text.TextColor3 = Color3.fromRGB(200,200,200)
text.Text = "Segurança Ativa\n[Espaço] Pulo Infinito\n[C] Teleporte para o Céu\n[Q] Descer\n[F] Mostrar/Esconder este painel"
text.Font = Enum.Font.SourceSansBold
text.TextSize = 16
text.TextYAlignment = Enum.TextYAlignment.Top
text.Parent = frame

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    end
end)

-- Pulo infinito
UserInputService.JumpRequest:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- Teleporte para o céu (C)
local function teleportarParaOCeu()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(
            hrp.Position.X, 
            ALTURA_CEU, 
            hrp.Position.Z, 
            hrp.CFrame.LookVector.X, 0, hrp.CFrame.LookVector.Z, 
            0, 1, 0, 
            -hrp.CFrame.LookVector.Z, 0, hrp.CFrame.LookVector.X
        )
        StarterGui:SetCore("SendNotification", {
            Title = "Teleporte";
            Text = "Teleportado para o céu!";
            Duration = 3;
        })
    end
end

-- ✅ Descer com Raycast e segurança (Q)
local function descer()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end

    -- Ativar PlatformStand
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

    -- Desativar PlatformStand depois de 0.2s
    if hum then
        task.delay(0.2, function()
            if hum then
                hum.PlatformStand = false
            end
        end)
    end
end

-- Atalhos de teclado
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.C then
        teleportarParaOCeu()
    elseif input.KeyCode == Enum.KeyCode.Q then
        descer()
    end
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

--[[ 
    Base com segurança + pulo infinito + teleporte para o céu aprimorado:
    - Anti-AFK
    - Delay aleatório
    - Checagem segura
    - Simulação de input real
    - Interface protegida por F
    - Pulo infinito (barra de espaço)
    - Teleporte para o céu (tecla C, aprimorado)
]]

-- Anti-AFK usando VirtualUser
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

-- Checagem segura de existência de objetos
local function safeCheck(obj)
    return obj ~= nil and obj.Parent ~= nil
end

-- Simulação de input real (exemplo: pulo)
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
frame.Size = UDim2.new(0, 200, 0, 80)
frame.Position = UDim2.new(0.5, -100, 0.5, -40)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.ZIndex = 9999
frame.Parent = gui

local text = Instance.new("TextLabel")
text.Size = UDim2.new(1, 0, 1, 0)
text.BackgroundTransparency = 1
text.TextColor3 = Color3.fromRGB(200,200,200)
text.Text = "Segurança Ativa\n[Espaço] Pulo Infinito\n[C] Teleporte Céu"
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

-- Teleporte para o céu ao apertar C (aprimorado)
local function teleportarParaOCeu()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(
            hrp.Position.X, 
            1000, 
            hrp.Position.Z, 
            hrp.CFrame.LookVector.X, 0, hrp.CFrame.LookVector.Z, 
            0, 1, 0, 
            -hrp.CFrame.LookVector.Z, 0, hrp.CFrame.LookVector.X
        )
    end
end

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.C then
        teleportarParaOCeu()
    end
end)

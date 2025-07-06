local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

--[[ 
    Segurança + pulo infinito + teleporte para o céu + teleporte para zona de coleta da base (com bypass de barreira):
    - [Espaço] Pulo Infinito
    - [C] Teleporte Céu
    - [V] Teleporte para zona de coleta (chega por cima para evitar barreira)
    - [F] Interface proteção
]]

local ZONA_NOMES = {
    "ZonaDeColeta", "Zona de Coleta", "ZONA DE COLETA", "Coleta", "ColetaArea", "DropZone"
}

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

local function randomDelay(min, max)
    wait(min + math.random() * (max - min))
end

local function safeCheck(obj)
    return obj ~= nil and obj.Parent ~= nil
end

local gui = Instance.new("ScreenGui")
gui.Name = "ProtegidoGUI_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.Enabled = false

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 110)
frame.Position = UDim2.new(0.5, -110, 0.5, -55)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.ZIndex = 9999
frame.Parent = gui

local text = Instance.new("TextLabel")
text.Size = UDim2.new(1, 0, 1, 0)
text.BackgroundTransparency = 1
text.TextColor3 = Color3.fromRGB(200,200,200)
text.Text = "Segurança Ativa\n[Espaço] Pulo Infinito\n[C] Teleporte Céu\n[V] Zona de Coleta (bypass barreira)"
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

-- Teleporte para o céu ao apertar C
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

-- Procurar zona de coleta
local function procurarZonaDeColeta()
    for _, nome in ipairs(ZONA_NOMES) do
        local zona = Workspace:FindFirstChild(nome)
        if zona and zona:IsA("BasePart") then
            return zona
        elseif zona and zona:IsA("Model") and zona.PrimaryPart then
            return zona.PrimaryPart
        end
    end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find("coleta") then
            return obj
        end
    end
    return nil
end

-- Teleporte para zona de coleta por cima (bypass barreira)
local function teleportarParaZonaDeColeta()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local zona = procurarZonaDeColeta()
    if zona then
        -- Primeiro teleporta bem acima (Y+20)
        hrp.CFrame = CFrame.new(zona.Position + Vector3.new(0, 20, 0))
        wait(0.5)
        -- Depois desce para a posição certa (Y+4)
        hrp.CFrame = CFrame.new(zona.Position + Vector3.new(0, 4, 0))
    end
end

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.C then
        teleportarParaOCeu()
    elseif input.KeyCode == Enum.KeyCode.V then
        teleportarParaZonaDeColeta()
    end
end)

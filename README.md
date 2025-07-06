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
local

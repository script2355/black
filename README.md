local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local StarterGui = game:GetService("StarterGui")

local ALTURA_EXTRA = 4 -- Altura acima do centro da parte/modelo
local ALTURA_DESCIDA = 5 -- Quanto desce ao apertar Q

--[[ 
    Base com segurança + pulo infinito + teleporte universal:
    - Anti-AFK
    - Delay aleatório
    - Checagem segura
    - Simulação de input real
    - Interface protegida por F
    - Pulo infinito (barra de espaço)
    - Teleporte universal (T)
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
text.Text = "Segurança Ativa\n[Espaço] Pulo Infinito\n[T] Teleporte Universal\n[Q] Descer"
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

-- Caixa de input (pelo chat) para teleporte universal
local function promptNomeParteUniversal(callback)
    StarterGui:SetCore("SendNotification", {
        Title = "Teleporte Universal";
        Text = "Digite o nome da parte/modelo no chat!";
        Duration = 5;
    })
    local connection
    connection = LocalPlayer.Chatted:Connect(function(msg)
        if callback then callback(msg) end
        if connection then connection:Disconnect() end
    end)
end

-- Teleporte universal para o centro de qualquer parte/modelo
local function teleportarParaNomeUniversal(nomeParte)
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find(nomeParte:lower()) then
            local destino = obj.Position + Vector3.new(0, ALTURA_EXTRA, 0)
            -- Proteção para chão sólido
            local params = RaycastParams.new()
            params.FilterDescendantsInstances = {LocalPlayer.Character}
            params.FilterType = Enum.RaycastFilterType.Blacklist
            local result = Workspace:Raycast(destino, Vector3.new(0, -100, 0), params)
            if result and result.Instance and result.Instance.CanCollide then
                destino = result.Position + Vector3.new(0, ALTURA_EXTRA, 0)
            end
            hrp.CFrame = CFrame.new(destino)
            StarterGui:SetCore("SendNotification", {
                Title = "Teleporte Universal";
                Text = "Teleportado para: "..obj.Name;
                Duration = 3;
            })
            return
        end
    end
    StarterGui:SetCore("SendNotification", {
        Title = "Teleporte Universal";
        Text = "Parte/Modelo não encontrado!";
        Duration = 3;
    })
end

-- Função de descer (Q)
local function descer()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame + Vector3.new(0, -ALTURA_DESCIDA, 0)
    end
end

-- Controles: T para teleporte universal, Q para descer
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.T then
        promptNomeParteUniversal(function(nome)
            teleportarParaNomeUniversal(nome)
        end)
    elseif input.KeyCode == Enum.KeyCode.Q then
        descer()
    end
end)

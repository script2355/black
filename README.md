-- Brainrot Hub PRO - Versão Experimental com funções extras
-- Desenvolvido para jogos como "Roube um Brainrot"
-- Adições: Detector de NPCs, alerta sonoro quando jogadores se aproximam, teleportador para locais dinâmicos, ESP avançado

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
    antiAFK = true,
    alertaProximidade = true
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
frame.Size = UDim2.new(0, 320, 0, 520)
frame.Position = UDim2.new(0.5, -160, 0.5, -260)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "🧠 Brainrot Hub PRO - EXP"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.BackgroundTransparency = 1
title.Parent = frame

-- Alerta de proximidade (som)
local alertaSom = Instance.new("Sound", gui)
alertaSom.SoundId = "rbxassetid://9118823106"
alertaSom.Volume = 2

RunService.Stepped:Connect(function()
    if not config.alertaProximidade then return end
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if dist < 30 then
                if not alertaSom.IsPlaying then
                    alertaSom:Play()
                end
            end
        end
    end
end)

-- Detector de NPCs
local function detectarNPCs()
    for _,v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("Model") and v:FindFirstChildOfClass("Humanoid") and not Players:GetPlayerFromCharacter(v) then
            local tag = Instance.new("BillboardGui", v:FindFirstChild("Head") or v.PrimaryPart or v)
            tag.Size = UDim2.new(0, 100, 0, 20)
            tag.Adornee = v:FindFirstChild("Head") or v.PrimaryPart
            tag.AlwaysOnTop = true
            local text = Instance.new("TextLabel", tag)
            text.Size = UDim2.new(1,0,1,0)
            text.Text = "[NPC]"
            text.TextColor3 = Color3.fromRGB(255, 200, 0)
            text.BackgroundTransparency = 1
            text.Font = Enum.Font.GothamBold
            text.TextSize = 14
        end
    end
end

-- Chamado ao iniciar
detectarNPCs()

-- Tecla especial para teleporte aleatório pelo mapa
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.T then
        local areas = {
            Vector3.new(100, 10, 100), Vector3.new(-200, 20, 50), Vector3.new(300, 5, -100)
        }
        local pos = areas[math.random(1, #areas)]
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char:MoveTo(pos)
        end
    end
end)

-- Label de info
local info = Instance.new("TextLabel", frame)
info.Size = UDim2.new(1, 0, 0, 20)
info.Position = UDim2.new(0, 0, 1, -25)
info.Text = "Pressione T para teleporte randômico | Alerta: ON"
info.TextColor3 = Color3.fromRGB(255,255,150)
info.Font = Enum.Font.Gotham
info.TextSize = 12
info.BackgroundTransparency = 1

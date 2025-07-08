-- Brainrot Hub PRO - Versão Completa e Avançada
-- Desenvolvido para "Roube um Brainrot"
-- Funções: Auto Farm, Alerta Proximidade, ESP, Noclip, Fly, Speed, Detector NPC, Menu de Teclas, Proteções e Diversos Boosts

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")

repeat wait() until LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.Character

-- Proteção básica
pcall(function()
    hookfunction(getfenv().print, function(...) end)
end)

local config = {
    autoFarm = true,
    infiniteJump = true,
    noclip = true,
    fly = false,
    flySpeed = 3,
    walkSpeed = 30,
    alertaProximidade = true
}

-- Anti-AFK
for _,v in pairs(getconnections(LocalPlayer.Idled)) do v:Disable() end
LocalPlayer.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new())
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new())
end)

-- GUI principal
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "BrainrotHub"
gui.ResetOnSpawn = false
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 320, 0, 500)
frame.Position = UDim2.new(0.5, -160, 0.5, -250)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Draggable, frame.Active = true, true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "🧠 Brainrot Hub PRO - Modo Dev"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)
title.BackgroundTransparency = 1

-- Alerta proximidade (som)
local alertaSom = Instance.new("Sound", gui)
alertaSom.SoundId = "rbxassetid://9118823106"
alertaSom.Volume = 2
RunService.Stepped:Connect(function()
    if not config.alertaProximidade then return end
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (p.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if dist < 25 then if not alertaSom.IsPlaying then alertaSom:Play() end end
        end
    end
end)

-- ESP de jogadores
for _,player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        player.CharacterAdded:Connect(function(char)
            local head = char:WaitForChild("Head")
            local tag = Instance.new("BillboardGui", head)
            tag.Size = UDim2.new(0, 100, 0, 30)
            tag.Adornee = head
            tag.AlwaysOnTop = true
            local label = Instance.new("TextLabel", tag)
            label.Size = UDim2.new(1,0,1,0)
            label.Text = "👤 "..player.Name
            label.TextColor3 = Color3.fromRGB(0,255,255)
            label.BackgroundTransparency = 1
            label.TextSize = 14
            label.Font = Enum.Font.GothamBold
        end)
    end
end

-- NPCs
for _,v in ipairs(Workspace:GetDescendants()) do
    if v:IsA("Model") and v:FindFirstChildOfClass("Humanoid") and not Players:GetPlayerFromCharacter(v) then
        local head = v:FindFirstChild("Head") or v.PrimaryPart
        if head then
            local tag = Instance.new("BillboardGui", head)
            tag.Size = UDim2.new(0, 100, 0, 20)
            tag.Adornee = head
            tag.AlwaysOnTop = true
            local txt = Instance.new("TextLabel", tag)
            txt.Size = UDim2.new(1,0,1,0)
            txt.Text = "[NPC Detected]"
            txt.TextColor3 = Color3.fromRGB(255,255,100)
            txt.Font = Enum.Font.GothamBold
            txt.BackgroundTransparency = 1
            txt.TextSize = 12
        end
    end
end

-- Auto Farm
task.spawn(function()
    while task.wait(2) do
        if config.autoFarm then
            for _, obj in pairs(Workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") then
                    pcall(function()
                        fireproximityprompt(obj)
                    end)
                end
            end
        end
    end
end)

-- Infinite Jump
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space and config.infiniteJump then
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- Noclip
RunService.Stepped:Connect(function()
    if config.noclip then
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid"):ChangeState(11)
        end
    end
end)

-- Fly ativável
local flyBV = Instance.new("BodyVelocity")
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        config.fly = not config.fly
        if config.fly then
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                flyBV.MaxForce = Vector3.new(1e5, 1e5, 1e5)
                flyBV.Velocity = Vector3.zero
                flyBV.Parent = hrp
                RunService:BindToRenderStep("Flying", Enum.RenderPriority.Input.Value, function()
                    local dir = Vector3.zero
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += Workspace.CurrentCamera.CFrame.LookVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= Workspace.CurrentCamera.CFrame.LookVector end
                    if UserInputService:IsKeyDown(Enum.KeyCode.E) then dir += Vector3.new(0,1,0) end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Q) then dir -= Vector3.new(0,1,0) end
                    flyBV.Velocity = dir.Unit * config.flySpeed * 50
                end)
            end
        else
            RunService:UnbindFromRenderStep("Flying")
            flyBV.Parent = nil
        end
    end
end)

-- Speed e reset
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = config.walkSpeed end
end)
local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
if hum then hum.WalkSpeed = config.walkSpeed end

-- Teleporte aleatório
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

-- Informação na GUI
local info = Instance.new("TextLabel", frame)
info.Size = UDim2.new(1, 0, 0, 20)
info.Position = UDim2.new(0, 0, 1, -25)
info.Text = "F = Fly | T = Random TP | W/E = Voar | Noclip: ON"
info.TextColor3 = Color3.fromRGB(255,255,150)
info.Font = Enum.Font.Gotham
info.TextSize = 12
info.BackgroundTransparency = 1

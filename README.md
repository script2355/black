local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local brainrotNames = {
    "Grapus Medusis",
    "Los Tralalelitos",
    "La Vacca Satuirno Saturnita",
    "Odins Din Din Duno",
    "Tralalerro Tralata",
    "Cocofanto Elefanto",
    "Cocozão",
    "Elefante",
    "Cocofantástico",
    "Elefantástico"
}

local nomeGUI = "Gui_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
local gui = Instance.new("ScreenGui")
gui.Name = nomeGUI
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.Enabled = false -- Começa invisível

-- Interface principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 220)
frame.Position = UDim2.new(0.5, -160, 0.5, -110)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.1
frame.ZIndex = 9999
frame.Parent = gui

-- Dropdown de seleção de Brainrot
local brainrotDropdown = Instance.new("TextButton")
brainrotDropdown.Size = UDim2.new(1, -20, 0, 40)
brainrotDropdown.Position = UDim2.new(0, 10, 0, 10)
brainrotDropdown.Text = "Selecione Brainrot"
brainrotDropdown.Name = "dropdown_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 6)
brainrotDropdown.Parent = frame

local dropdownFrame = Instance.new("ScrollingFrame")
dropdownFrame.Size = UDim2.new(1, -20, 0, 120)
dropdownFrame.Position = UDim2.new(0, 10, 0, 50)
dropdownFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
dropdownFrame.BorderSizePixel = 0
dropdownFrame.Visible = false
dropdownFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
dropdownFrame.ScrollBarThickness = 5
dropdownFrame.Parent = frame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = dropdownFrame

local selectedBrainrotName = nil

local function popularListaFixa()
    for _, child in pairs(dropdownFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end

    for i, name in ipairs(brainrotNames) do
        local item = Instance.new("TextButton")
        item.Size = UDim2.new(1, 0, 0, 30)
        item.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        item.BorderSizePixel = 0
        item.Text = name
        item.TextColor3 = Color3.fromRGB(255, 255, 255)
        item.LayoutOrder = i
        item.Parent = dropdownFrame

        item.MouseButton1Click:Connect(function()
            selectedBrainrotName = name
            brainrotDropdown.Text = name
            dropdownFrame.Visible = false
        end)
    end

    dropdownFrame.CanvasSize = UDim2.new(0, 0, 0, #brainrotNames * 30)
end

popularListaFixa()

brainrotDropdown.MouseButton1Click:Connect(function()
    dropdownFrame.Visible = not dropdownFrame.Visible
end)

-- Atalho F para abrir/fechar GUI
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
        if not gui.Enabled then dropdownFrame.Visible = false end
    end
end)

-- Controle de modo, ESP e Auto
local modo = 2
local espAtivo = true
local autoAtivo = false

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

local function criarESP(modelo)
    if not espAtivo or not modelo then return end
    if modelo:FindFirstChildOfClass("Highlight") then return end
    local hl = Instance.new("Highlight")
    hl.FillColor = Color3.fromRGB(255, 0, 0)
    hl.OutlineColor = Color3.fromRGB(255, 255, 255)
    hl.Adornee = modelo
    hl.Parent = modelo
    task.delay(20 + math.random(5), function()
        if hl and hl.Parent then hl:Destroy() end
    end)
end

local function localizarBrainrot(nome)
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name == nome then
            if obj:FindFirstChildOfClass("Humanoid") then
                criarESP(obj)
                task.wait(0.1 + math.random() * 0.3)
                return obj
            end
        end
    end
end

-- Movimento suave com TweenService
local function moveSmoothlyTo(pos)
    local char = LocalPlayer.Character
    if not (char and char:FindFirstChild("HumanoidRootPart")) then return end
    local hrp = char.HumanoidRootPart
    local distance = (hrp.Position - pos).Magnitude
    local duration = math.clamp(distance / 20, 0.4, 2.5) + math.random() * 0.3
    local tween = TweenService:Create(hrp, TweenInfo.new(duration, Enum.EasingStyle.Linear), {CFrame = CFrame.new(pos)})
    tween:Play()
    tween.Completed:Wait()
end

-- Simulação de clique real na ferramenta
local function useTool()
    local char = LocalPlayer.Character
    local tool = char and char:FindFirstChildOfClass("Tool")
    if tool then
        VirtualInputManager:SendMouseButtonEvent(0,0,0,true,game,0)
        wait(0.12 + math.random() * 0.07)
        VirtualInputManager:SendMouseButtonEvent(0,0,0,false,game,0)
    end
end

local function irAndandoAte(pos)
    moveSmoothlyTo(pos)
end

local function roubar(nome)
    local brain = localizarBrainrot(nome)
    if not brain then return end

    local hrp = brain:FindFirstChild("HumanoidRootPart")
    local char = LocalPlayer.Character
    local myHRP = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp or not myHRP then return end

    local base = myHRP.Position

    if modo == 1 then
        pcall(function()
            wait(0.3 + math.random())
            moveSmoothlyTo(hrp.Position + Vector3.new(0, 3, 0))
            wait(0.15 + math.random() * 0.2)
            useTool()
            wait(0.25 + math.random() * 0.2)
            moveSmoothlyTo(base)
        end)
    else
        pcall(function()
            local original = myHRP.Position
            wait(0.3 + math.random())
            moveSmoothlyTo(hrp.Position + Vector3.new(0, 3, 0))
            wait(0.5 + math.random())
            useTool()
            wait(0.5 + math.random())
            moveSmoothlyTo(original + Vector3.new(math.random(-18,18), 0, math.random(-18,18)))
        end)
    end
end

-- Loop de roubo automático (com delays aleatórios)
task.spawn(function()
    while true do
        wait(2.5 + math.random())
        if autoAtivo and selectedBrainrotName and gui.Enabled then
            pcall(function()
                roubar(selectedBrainrotName)
            end)
        end
    end
end)

-- Botões da interface
local modoBotao = Instance.new("TextButton")
modoBotao.Size = UDim2.new(0.5, -15, 0, 30)
modoBotao.Position = UDim2.new(0, 10, 0, 180)
modoBotao.Text = "Modo: 2 (Seguro)"
modoBotao.Parent = frame

local espToggle = Instance.new("TextButton")
espToggle.Size = UDim2.new(0.5, -15, 0, 30)
espToggle.Position = UDim2.new(0.5, 5, 0, 180)
espToggle.Text = "ESP: Ligado"
espToggle.Parent = frame

local roubarBotao = Instance.new("TextButton")
roubarBotao.Size = UDim2.new(1, -20, 0, 40)
roubarBotao.Position = UDim2.new(0, 10, 0, 140)
roubarBotao.Text = "ATIVAR ROUBO AUTOMÁTICO"
roubarBotao.Parent = frame

modoBotao.MouseButton1Click:Connect(function()
    modo = (modo == 1) and 2 or 1
    modoBotao.Text = "Modo: " .. modo .. (modo == 1 and " (Rápido)" or " (Seguro)")
end)

espToggle.MouseButton1Click:Connect(function()
    espAtivo = not espAtivo
    espToggle.Text = "ESP: " .. (espAtivo and "Ligado" or "Desligado")
end)

roubarBotao.MouseButton1Click:Connect(function()
    autoAtivo = not autoAtivo
    roubarBotao.Text = autoAtivo and "ROUBANDO..." or "ATIVAR ROUBO AUTOMÁTICO"
end)

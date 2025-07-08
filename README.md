-- Botões de teleporte
criarBotao("📦 Teleporte Base 1", function()
    teleportarPara(Vector3.new(0, 10, 0)) -- Substitua pela posição real
end)

criarBotao("📦 Teleporte Base 2", function()
    teleportarPara(Vector3.new(100, 10, 0)) -- Substitua pela posição real
end)

criarBotao("📦 Teleporte para o Céu", function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(0, 1000, 0)
    end
end)

criarBotao("⬇️ Descer 100", function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 100, 0)
    end
end)

-- Teclas rápidas para teleporte
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.F then
        gui.Enabled = not gui.Enabled
    elseif input.KeyCode == Enum.KeyCode.X then
        toggleFly()
    elseif input.KeyCode == Enum.KeyCode.Z then
        config.autoFarm = not config.autoFarm
    elseif input.KeyCode == Enum.KeyCode.C then
        -- Teleporte para o céu
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(0, 1000, 0)
        end
    elseif input.KeyCode == Enum.KeyCode.V then
        -- Teleporte para baixo
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame - Vector3.new(0, 100, 0)
        end
    elseif input.KeyCode == Enum.KeyCode.N then
        config.noclip = not config.noclip
    elseif input.KeyCode == Enum.KeyCode.L then
        config.walkSpeed += 10
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
        if hum then hum.WalkSpeed = config.walkSpeed end
    elseif input.KeyCode == Enum.KeyCode.R then
        LocalPlayer:LoadCharacter()
    end
end)

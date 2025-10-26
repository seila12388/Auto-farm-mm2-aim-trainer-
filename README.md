# Auto-farm-mm2-aim-trainer-
It's in beta auto farm 
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

local config = {
    ESPEnabled = true,
    TPToggle = false,
    EnemyColor = Color3.fromRGB(255, 0, 0),
    SpeedMultiplier = 2,
    HitboxSize = Vector3.new(4, 6, 2),
    AttackCooldown = 0,
}

local function isEnemy(player)
    if player.Team and LocalPlayer.Team then
        return player.Team ~= LocalPlayer.Team
    end
    return true
end

local function removeHitbox(character)
    local hitbox = character and character:FindFirstChild("MM2Hitbox")
    if hitbox then
        hitbox:Destroy()
    end
end

local function createHitbox(character)
    if character:FindFirstChild("MM2Hitbox") then return end
    local torso = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso")
    if not torso then return end

    local hitbox = Instance.new("Part")
    hitbox.Name = "MM2Hitbox"
    hitbox.Size = config.HitboxSize
    hitbox.Transparency = 0.7
    hitbox.CanCollide = false
    hitbox.Anchored = false
    hitbox.Massless = true
    hitbox.Color = config.EnemyColor
    hitbox.Material = Enum.Material.Neon
    hitbox.Parent = character

    local weld = Instance.new("WeldConstraint")
    weld.Part0 = hitbox
    weld.Part1 = torso
    weld.Parent = hitbox
end

local function applyESP(player)
    if not player.Character then return end
    if player.Character:FindFirstChild("MM2ESP") then return end
    if not isEnemy(player) then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "MM2ESP"
    highlight.FillColor = config.EnemyColor
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = player.Character
    highlight.Parent = player.Character

    createHitbox(player.Character)
end

local function removeESP(player)
    if player.Character then
        local esp = player.Character:FindFirstChild("MM2ESP")
        if esp then esp:Destroy() end
        removeHitbox(player.Character)
    end
end

local function updateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            removeESP(player)
            if config.ESPEnabled then
                applyESP(player)
            end
        end
    end
end

local function getClosestEnemy()
    local closestPlayer = nil
    local shortestDist = math.huge
    local char = LocalPlayer.Character
    if not char then return nil end
    local localHRP = char:FindFirstChild("HumanoidRootPart")
    if not localHRP then return nil end
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isEnemy(player) then
            local dist = (player.Character.HumanoidRootPart.Position - localHRP.Position).Magnitude
            if dist < shortestDist then
                shortestDist = dist
                closestPlayer = player
            end
        end
    end
    return closestPlayer
end

RunService.Heartbeat:Connect(function()
    if config.TPToggle then
        local target = getClosestEnemy()
        local char = LocalPlayer.Character
        if target and char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
        end
    end

    if config.ESPEnabled then
        updateESP()
    end
end)

local function applySpeed()
    local char = LocalPlayer.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 16 * config.SpeedMultiplier
            humanoid.AutoRotate = true
        end
    end
end

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    updateESP()
    applySpeed()
end)

LocalPlayer.CharacterRemoving:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        removeESP(player)
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(1)
        if config.ESPEnabled then
            applyESP(player)
        end
    end)
end)

-- GUI

local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local ScreenGui = PlayerGui:FindFirstChild("MM2AimTrainerGUI")
if ScreenGui then ScreenGui:Destroy() end

ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MM2AimTrainerGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local Frame = Instance.new("Frame", ScreenGui)
Frame.Name = "MainFrame"
Frame.Size = UDim2.new(0, 220, 0, 180)
Frame.Position = UDim2.new(0.05, 0, 0.05, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel", Frame)
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 28)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Text = "MM2 ESP + TP + Speed + Hitbox"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18

local function createToggle(text, pos, default, callback)
    local button = Instance.new("TextButton", Frame)
    button.Size = UDim2.new(0, 200, 0, 36)
    button.Position = UDim2.new(0, 10, 0, pos)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.Gotham
    button.TextScaled = true
    button.Text = text .. ": " .. (default and "ON" or "OFF")

    button.MouseButton1Click:Connect(function()
        default = not default
        button.Text = text .. ": " .. (default and "ON" or "OFF")
        callback(default)
    end)

    return button
end

local function createSlider(labelText, pos, min, max, default, step, callback)
    local lbl = Instance.new("TextLabel", Frame)
    lbl.Size = UDim2.new(0, 90, 0, 20)
    lbl.Position = UDim2.new(0, 10, 0, pos)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 14
    lbl.Text = labelText .. ": " .. tostring(default)

    local sliderBg = Instance.new("Frame", Frame)
    sliderBg.Size = UDim2.new(0, 100, 0, 16)
    sliderBg.Position = UDim2.new(0, 100, 0, pos + 2)
    sliderBg.BackgroundColor3 = Color3.fromRGB(50,50,50)
    sliderBg.BorderSizePixel = 0

    local sliderFill = Instance.new("Frame", sliderBg)
    sliderFill.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    sliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)

    local sliderBtn = Instance.new("TextButton", sliderBg)
    sliderBtn.Size = UDim2.new(0, 14, 0, 16)
    sliderBtn.Position = UDim2.new((default - min) / (max - min), 0, 0, 0)
    sliderBtn.BackgroundColor3 = Color3.fromRGB(200,200,200)
    sliderBtn.Text = ""
    sliderBtn.AutoButtonColor = false

    local dragging = false

    sliderBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)

    sliderBtn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    RunService.RenderStepped:Connect(function()
        if dragging then
            local mousePos = game:GetService("UserInputService"):GetMouseLocation()
            local sliderPos = sliderBg.AbsolutePosition.X
            local sliderSize = sliderBg.AbsoluteSize.X

            local relativePos = math.clamp(mousePos.X - sliderPos, 0, sliderSize)
            local value = min + (relativePos / sliderSize) * (max - min)
            value = math.floor(value / step + 0.5) * step

            sliderFill.Size = UDim2.new((value - min) / (max - min), 0, 1, 0)
            sliderBtn.Position = UDim2.new((value - min) / (max - min), 0, 0, 0)
            lbl.Text = labelText .. ": " .. tostring(value)
            callback(value)
        end
    end)

    return lbl, sliderBg, sliderFill, sliderBtn
end

createToggle("ESP Inimigos", 38, config.ESPEnabled, function(val)
    config.ESPEnabled = val
    updateESP()
end)

createToggle("Teleportar Inimigo", 78, config.TPToggle, function(val)
    config.TPToggle = val
end)

createToggle("Aumentar Velocidade", 118, config.SpeedMultiplier > 1, function(val)
    if val then
        config.SpeedMultiplier = 2
    else
        config.SpeedMultiplier = 1
    end
    applySpeed()
end)

createSlider("Velocidade", 158, 1, 5, config.SpeedMultiplier, 0.1, function(val)
    config.SpeedMultiplier = val
    applySpeed()
end)

createSlider("Hitbox Height", 198, 2, 10, config.HitboxSize.Y, 0.1, function(val)
    config.HitboxSize = Vector3.new(config.HitboxSize.X, val, config.HitboxSize.Z)
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("MM2Hitbox") then
            player.Character.MM2Hitbox.Size = config.HitboxSize
        end
    end
end)

createSlider("Hitbox Width", 238, 1, 6, config.HitboxSize.X, 0.1, function(val)
    config.HitboxSize = Vector3.new(val, config.HitboxSize.Y, config.HitboxSize.Z)
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("MM2Hitbox") then
            player.Character.MM2Hitbox.Size = config.HitboxSize
        end
    end
end)

-- Botão "M" para abrir/fechar GUI
local buttonM = Instance.new("TextButton", PlayerGui)
buttonM.Size = UDim2.new(0, 30, 0, 30)
buttonM.Position = UDim2.new(0, 10, 0, 10)
buttonM.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
buttonM.TextColor3 = Color3.new(1,1,1)
buttonM.Font = Enum.Font.GothamBold
buttonM.TextSize = 20
buttonM.Text = "M"
buttonM.Modal = false
buttonM.ZIndex = 10
buttonM.AutoButtonColor = true
buttonM.AnchorPoint = Vector2.new(0,0)
buttonM.BackgroundTransparency = 0.2
buttonM.TextStrokeTransparency = 0.5
buttonM.BorderSizePixel = 0

local guiVisible = true
buttonM.MouseButton1Click:Connect(function()
    guiVisible = not guiVisible
    Frame.Visible = guiVisible
end)

updateESP()
applySpeed()

print("[✔] Script MM2 Aim Trainer atualizado com sliders e botão M para GUI!")
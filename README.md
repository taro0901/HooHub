local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "Rayfield Example Window",
    Icon = 0,
    LoadingTitle = "Rayfield Interface Suite",
    LoadingSubtitle = "by Sirius",
    Theme = "Default",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "roKu Hub"
    },
    KeySystem = false,
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    }
})

local Tab = Window:CreateTab("Main", 4483362458)
local Section = Tab:CreateSection("Aimbot")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

local aimEnabled = false
local MAX_DISTANCE = 65
local AIM_SMOOTHNESS = 0.3
local espEnabled = false
local staminaEnabled = false

local BLOCKED_NAMES = {
    ["1tygho_134"] = true
}

local function getClosestTarget()
    local closestTarget = nil
    local shortestDistance = math.huge

    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and not BLOCKED_NAMES[target.Name] and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head
            if head and head:IsA("BasePart") and target.Character:FindFirstChild("Humanoid") and target.Character.Humanoid.Health > 0 then
                local headPos, onScreen = camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local mousePos = Vector2.new(mouse.X, mouse.Y)
                    local distance = (mousePos - Vector2.new(headPos.X, headPos.Y)).Magnitude
                    local distance3D = (camera.CFrame.Position - head.Position).Magnitude
                    if distance < shortestDistance and distance3D <= MAX_DISTANCE then
                        shortestDistance = distance
                        closestTarget = target
                    end
                end
            end
        end
    end
    return closestTarget
end

RunService.RenderStepped:Connect(function()
    if aimEnabled then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local newCFrame = CFrame.new(camera.CFrame.Position, headPos)
            camera.CFrame = camera.CFrame:Lerp(newCFrame, AIM_SMOOTHNESS)
        end
    end
end)

-- Rayfield Toggle for Aimbot
local ToggleAimbot = Tab:CreateToggle({
    Name = "Aimbot (Head Aiming with Hotkey)",
    CurrentValue = false,
    Flag = "Aimbot_Toggle",
    Callback = function(Value)
        aimEnabled = Value
    end
})

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.LeftControl then
        aimEnabled = not aimEnabled
        ToggleAimbot:Set(aimEnabled)
    end
end)

-- ESP (name and health display) setup
local function createESP(character, player)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    if not character:FindFirstChild("ESP_Highlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.FillColor = Color3.fromRGB(255, 0, 0)
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Adornee = character
        highlight.Parent = character
    end

    if not character:FindFirstChild("ESP_Billboard") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP_Billboard"
        billboard.Adornee = character:FindFirstChild("HumanoidRootPart")
        billboard.Size = UDim2.new(0, 120, 0, 30)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = character

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, 0, 0.6, 0)
        nameLabel.Position = UDim2.new(0, 0, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextStrokeTransparency = 0
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.TextSize = 11
        nameLabel.Text = ""
        nameLabel.Parent = billboard

        local healthBarBackground = Instance.new("Frame")
        healthBarBackground.Name = "HealthBarBackground"
        healthBarBackground.Size = UDim2.new(1, -8, 0, 5)
        healthBarBackground.Position = UDim2.new(0, 4, 1, -7)
        healthBarBackground.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        healthBarBackground.BorderSizePixel = 0
        healthBarBackground.Parent = billboard

        local healthBar = Instance.new("Frame")
        healthBar.Name = "HealthBar"
        healthBar.Size = UDim2.new(1, 0, 1, 0)
        healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        healthBar.BorderSizePixel = 0
        healthBar.Parent = healthBarBackground
    end
end

local function removeESP(character)
    if not character then return end
    local highlight = character:FindFirstChild("ESP_Highlight")
    if highlight then
        highlight:Destroy()
    end
    local billboard = character:FindFirstChild("ESP_Billboard")
    if billboard then
        billboard:Destroy()
    end
end

local function updateESP(player)
    if player ~= player and player.Character then
        local character = player.Character
        if espEnabled then
            createESP(character, player)

            local billboard = character:FindFirstChild("ESP_Billboard")
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if billboard and humanoid and billboard:FindFirstChild("NameLabel") then
                billboard.NameLabel.Text = player.Name .. " (" .. math.floor(humanoid.Health) .. " HP)"

                if humanoid.Health <= 25 then
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                elseif humanoid.Health <= 50 then
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
                else
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                end

                local healthBarBackground = billboard:FindFirstChild("HealthBarBackground")
                if healthBarBackground and healthBarBackground:FindFirstChild("HealthBar") then
                    local healthBar = healthBarBackground.HealthBar
                    healthBar.Size = UDim2.new(math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1), 0, 1, 0)
                    if humanoid.Health <= 25 then
                        healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                    elseif humanoid.Health <= 50 then
                        healthBar.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
                    else
                        healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                    end
                end
            end
        else
            removeESP(character)
        end
    end
end

local function setupPlayer(player)
    player.CharacterAdded:Connect(function(character)
        character:WaitForChild("HumanoidRootPart", 5)
        if espEnabled then
            createESP(character, player)
        end
    end)
    if player.Character then
        createESP(player.Character, player)
    end
end

RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            updateESP(player)
        end
    end
end)

for _, player in ipairs(Players:GetPlayers()) do
    setupPlayer(player)
end

Players.PlayerAdded:Connect(setupPlayer)

local ToggleESP = Tab:CreateToggle({
    Name = "ESP (Small Set)",
    CurrentValue = false,
    Flag = "ESP_Toggle",
    Callback = function(Value)
        espEnabled = Value
        for _, player in ipairs(Players:GetPlayers()) do
            updateESP(player)
        end
    end
})

local Tab = Window:CreateTab("Stamina", 4483362458)
local Section = Tab:CreateSection("Stamin")

-- Infinite Stamina (Prevents Stamina from Depleting)
local function lockStamina(character)
    while not character:GetAttribute("Stamina") do
        task.wait()
    end

    RunService.RenderStepped:Connect(function()
        if staminaEnabled and character and character:GetAttribute("Stamina") then
            character:SetAttribute("Stamina", 100)
        end
    end)
end

local function setupCharacter(char)
    Character = char
    lockStamina(Character)
end

LocalPlayer.CharacterAdded:Connect(function(char)
    setupCharacter(char)
end)

setupCharacter(Character)

local ToggleStamina = Tab:CreateToggle({
    Name = "Infinite Stamina",
    CurrentValue = false,
    Flag = "Stamina_Toggle",
    Callback = function(Value)
        staminaEnabled = Value
    end,
})

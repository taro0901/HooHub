local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "Rayfield Example Window",
    Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
    LoadingTitle = "Rayfield Interface Suite",
    LoadingSubtitle = "by Sirius",
    Theme = "Default", -- Check https://docs.sirius.menu/rayfield/configuration/themes
 
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface
 
    ConfigurationSaving = {
       Enabled = true,
       FolderName = nil, -- Create a custom folder for your hub/game
       FileName = "roKu Hub"
    },
 
    Discord = {
       Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
       Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ ABCD would be ABCD
       RememberJoins = true -- Set this to false to make them join the discord every time they load it up
    },
 
    KeySystem = false, -- Set this to true to use our key system
    KeySettings = {
       Title = "Untitled",
       Subtitle = "Key System",
       Note = "No method of obtaining the key is provided", -- Use this to tell the user how to get a key
       FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
       SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
       GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
       Key = {"123"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
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

local BLOCKED_NAMES = { -- ❌ ชื่อที่ไม่ล็อก
    ["1tygho_134"] = true
}

-- หาเป้าหมายใกล้สุด
local function getClosestTarget()
    local closestTarget = nil
    local shortestDistance = math.huge

    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player
        and not BLOCKED_NAMES[target.Name]
        and target.Character
        and target.Character:FindFirstChild("Head") then

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

-- อัพเดทกล้องตามเป้า
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

-- 🌟 Rayfield UI Toggle
local Toggle = Tab:CreateToggle({
   Name = "Aimbot เล็งหัว (มีคีย์ลัด)",
   CurrentValue = false,
   Flag = "StableAimbotWithKeybind",
   Callback = function(Value)
      aimEnabled = Value
   end,
})

-- 🎯 ระบบคีย์ลัด (Left Control เปิด/ปิด)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.LeftControl then
        aimEnabled = not aimEnabled -- เปิด/ปิด
        Toggle:Set(aimEnabled) -- อัพเดท UI Toggle ด้วย
    end
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local espEnabled = false

-- 📦 สร้าง ESP (Highlight + BillboardGui)
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
        billboard.Size = UDim2.new(0, 120, 0, 30) -- 📏 Billboard เล็กลง
        billboard.StudsOffset = Vector3.new(0, 2, 0) -- สูงขึ้นเล็กน้อย
        billboard.AlwaysOnTop = true
        billboard.Parent = character

        -- 🔤 ชื่อผู้เล่น
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, 0, 0.6, 0) -- เอาสูงนิดเดียวพอ
        nameLabel.Position = UDim2.new(0, 0, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextStrokeTransparency = 0
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.TextSize = 11 -- ✏️ ตัวอักษรเล็กลง
        nameLabel.Text = ""
        nameLabel.Parent = billboard

        -- 🩸 แถบเลือดพื้นหลัง
        local healthBarBackground = Instance.new("Frame")
        healthBarBackground.Name = "HealthBarBackground"
        healthBarBackground.Size = UDim2.new(1, -8, 0, 5) -- กว้างเล็กลง สูงแค่ 5px
        healthBarBackground.Position = UDim2.new(0, 4, 1, -7)
        healthBarBackground.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        healthBarBackground.BorderSizePixel = 0
        healthBarBackground.Parent = billboard

        -- ❤️ ตัวแถบเลือด
        local healthBar = Instance.new("Frame")
        healthBar.Name = "HealthBar"
        healthBar.Size = UDim2.new(1, 0, 1, 0)
        healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        healthBar.BorderSizePixel = 0
        healthBar.Parent = healthBarBackground
    end
end

-- 🗑️ ลบ ESP
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

-- 🎯 อัปเดต ESP
local function updateESP(player)
    if player ~= localPlayer and player.Character then
        local character = player.Character
        if espEnabled then
            createESP(character, player)

            local billboard = character:FindFirstChild("ESP_Billboard")
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if billboard and humanoid and billboard:FindFirstChild("NameLabel") then
                billboard.NameLabel.Text = player.Name .. " (" .. math.floor(humanoid.Health) .. " HP)"

                -- สีตัวอักษร
                if humanoid.Health <= 25 then
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                elseif humanoid.Health <= 50 then
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
                else
                    billboard.NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                end

                -- อัปเดต HealthBar
                local healthBarBackground = billboard:FindFirstChild("HealthBarBackground")
                if healthBarBackground and healthBarBackground:FindFirstChild("HealthBar") then
                    local healthBar = healthBarBackground.HealthBar
                    healthBar.Size = UDim2.new(math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1), 0, 1, 0)

                    -- เปลี่ยนสีหลอดเลือด
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

-- ⚙️ ติดตั้ง ESP
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

-- ติดตั้งกับทุกคนที่อยู่
for _, player in ipairs(Players:GetPlayers()) do
    setupPlayer(player)
end

-- เมื่่อมีคนใหม่เข้า
Players.PlayerAdded:Connect(setupPlayer)

-- 🟢 Toggle ESP
local Toggle = Tab:CreateToggle({
    Name = "ESP (เล็กลงทั้งชุด)",
    CurrentValue = false,
    Flag = "ESP_Toggle",
    Callback = function(Value)
        espEnabled = Value
        for _, player in ipairs(Players:GetPlayers()) do
            updateESP(player)
        end
    end,
})

-- 🎥 RenderStepped เช็กตลอด
RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            updateESP(player)
        end
    end
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local staminaEnabled = false

-- 📦 ฟังก์ชันล็อก Stamina (ไม่ให้ลด)
local function lockStamina(character)
    -- รอ Attribute 'Stamina'
    while not character:GetAttribute("Stamina") do
        task.wait()
    end

    -- อัปเดตค่า Stamina ให้เต็มตลอด
    RunService.RenderStepped:Connect(function()
        if staminaEnabled and character and character:GetAttribute("Stamina") then
            character:SetAttribute("Stamina", 100) -- เซ็ตเต็ม 100
        end
    end)
end

-- 🗑️ ฟังก์ชันรีเซ็ต Character ใหม่
local function setupCharacter(char)
    Character = char
    lockStamina(Character)
end

-- 🎯 ฟังก์ชันเมื่อเกิดตัวใหม่
LocalPlayer.CharacterAdded:Connect(function(char)
    setupCharacter(char)
end)

-- 🛠️ เริ่มต้นทำงานกับตัวที่มีอยู่แล้ว
setupCharacter(Character)

-- 🟢 ใส่ Toggle UI เปิด/ปิด ระบบ Stamina Infinite
local Toggle = Tab:CreateToggle({
    Name = "Infinite Stamina (สแตมิน่าไม่ลด)",
    CurrentValue = false,
    Flag = "Stamina_Toggle",
    Callback = function(Value)
        staminaEnabled = Value
    end,
})





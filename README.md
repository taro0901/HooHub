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

-- 📦 ฟังก์ชันสร้าง ESP (Highlight)
local function createESP(character)
    if not character:FindFirstChild("ESP_Highlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.FillColor = Color3.fromRGB(255, 0, 0) -- สีตัว ESP
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- สีขอบ
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Adornee = character
        highlight.Parent = character
    end
end

-- 🗑️ ฟังก์ชันลบ ESP
local function removeESP(character)
    local highlight = character:FindFirstChild("ESP_Highlight")
    if highlight then
        highlight:Destroy()
    end
end

-- 🎯 ฟังก์ชันอัปเดต ESP ให้กับผู้เล่นคนเดียว
local function updateESP(player)
    if player ~= localPlayer then
        if player.Character then
            if espEnabled then
                createESP(player.Character)
            else
                removeESP(player.Character)
            end
        end
    end
end

-- 🛠️ ฟังก์ชันติดตั้ง ESP ให้ผู้เล่นที่มีอยู่แล้ว
for _, player in ipairs(Players:GetPlayers()) do
    if player.Character then
        updateESP(player)
    end
end

-- 🛠️ เมื่่อมีผู้เล่นใหม่เข้ามา
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        character:WaitForChild("HumanoidRootPart") -- รอโหลดตัวละคร
        updateESP(player)
    end)
end)

-- 🟢 ใส่ Toggle UI (เหมือน Aimbot Toggle)
local Toggle = Tab:CreateToggle({
    Name = "ESP (มองเห็นศัตรู)",
    CurrentValue = false,
    Flag = "ESP_Toggle",
    Callback = function(Value)
        espEnabled = Value
    end,
})

-- ⚡ อัปเดต ESP ทุกเฟรม
RunService.RenderStepped:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            updateESP(player)
        end
    end
end)


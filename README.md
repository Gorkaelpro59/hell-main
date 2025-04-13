-- Library loading and whitelist check
local success, library = pcall(function()
    return loadstring(game:HttpGet("https://pastebin.com/raw/Abg3RkND", true))()
end)

if not success then
    error("Error al cargar la biblioteca: " .. tostring(library))
elseif not library then
    error("La biblioteca cargada es nil.")
end

local window = library:AddWindow("HELL clan Script by darkiller", { main_color = Color3.fromRGB(41, 74, 122), min_size = Vector2.new(600, 550), can_resize = false })

-- Whitelist system using usernames
local whitelist = {
    "Quiraz29", 
    "Shadow_RipperZ0",
    "HoloGrindz",
    "Xxdarkiller_10ZKT2",
    "TARZANfeio",
    "Pathaan122378",    
    "Gorkaelpro59",
}

local player = game.Players.LocalPlayer
if not player then error("LocalPlayer not found!") end

local function isWhitelisted(player)
    for _, username in ipairs(whitelist) do
        if player.Name == username then
            return true
        end
    end
    return false
end

if not isWhitelisted(player) then
    error("You are not whitelisted to use this script.")
end

----------------------------------------------------------------
-- Paid Tab and its functionalities
local Paid = window:AddTab("Paid")

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth then
        spawn(function()
            local ReplicatedStorage = game:GetService("ReplicatedStorage")
            local Players = game:GetService("Players")
            local c = Players.LocalPlayer
            
            if not c:FindFirstChild("petsFolder") then
                error("petsFolder not found.")
                return
            end
            
            local function d()
                local f = c.petsFolder
                for _, folder in pairs(f:GetChildren()) do
                    if folder:IsA("Folder") then
                        for _, pet in pairs(folder:GetChildren()) do
                            ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                        end
                    end
                end
                task.wait(.1)
            end
            
            local function k(l)
                d()
                task.wait(.01)
                for _, pet in pairs(c.petsFolder.Unique:GetChildren()) do
                    if pet.Name == l then
                        ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                    end
                end
            end
            
            local function o(p)
                local q = workspace.machinesFolder:FindFirstChild(p)
                if not q then
                    for _, s in pairs(workspace:GetChildren()) do
                        if s:IsA("Folder") and s.Name:find("machines") then
                            q = s:FindFirstChild(p)
                            if q then break end
                        end
                    end
                end
                return q
            end
            
            local function t()
                local u = game:GetService("VirtualInputManager")
                u:SendKeyEvent(true, "E", false, game)
                task.wait(.1)
                u:SendKeyEvent(false, "E", false, game)
            end
            
            while fastRebirth do
                local v = (c.leaderstats and c.leaderstats.Rebirths and c.leaderstats.Rebirths.Value) or 0
                local w = 10000 + (5000 * v)
                if c.ultimatesFolder and c.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                    local x = c.ultimatesFolder["Golden Rebirth"].Value
                    w = math.floor(w * (1 - (x * 0.1)))
                end
                d()
                task.wait(.1)
                k("Swift Samurai")
                while c.leaderstats and c.leaderstats.Strength and c.leaderstats.Strength.Value < w do
                    for _ = 1, 10 do
                        c.muscleEvent:FireServer("rep")
                    end
                    task.wait()
                end
                d()
                task.wait(.1)
                k("Tribal Overlord")
                local z = o("Jungle Bar Lift")
                if z and z:FindFirstChild("interactSeat") then
                    c.Character.HumanoidRootPart.CFrame = z.interactSeat.CFrame * CFrame.new(0, 3, 0)
                    repeat
                        task.wait(.1)
                        t()
                    until c.Character.Humanoid.Sit
                end
                local A = (c.leaderstats and c.leaderstats.Rebirths and c.leaderstats.Rebirths.Value) or 0
                repeat
                    ReplicatedStorage.rEvents.rebirthRemote:InvokeServer("rebirthRequest")
                    task.wait(.1)
                until c.leaderstats and c.leaderstats.Rebirths and c.leaderstats.Rebirths.Value > A
                task.wait()
            end
        end)
    end
end)

local switch = Paid:AddSwitch("Fast Strength", function(Value)
    _G.RepOP = Value
    if not Value then return end
    local function FastRep()
        while _G.RepOP do
            for _ = 1, 50 do
                game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
            end
            task.wait()
        end
    end
    task.spawn(FastRep)
end)

local switchHideFrame = Paid:AddSwitch("Hide Frame", function(bool)
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        local frame = game:GetService("ReplicatedStorage"):FindFirstChild(frameName)
        if frame and frame:IsA("GuiObject") then
            frame.Visible = not bool
        end
    end
end)

----------------------------------------------------------------
-- Position and Teleport Tab
local PositionAndTeleport = window:AddTab("Position and Teleport")
local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
local lockpos = false
local cp

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool  -- Toggle lockpos state
    if lockpos and hrp then
        cp = hrp.Position  -- Save current position on enabling lock
    end
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if lockpos and hrp then
        hrp.CFrame = CFrame.new(cp)
        hrp.Velocity = Vector3.new(0, 0, 0)
        hrp.RotVelocity = Vector3.new(0, 0, 0)
    end
end)

----------------------------------------------------------------
-- Proteins Tab
local Proteins = window:AddTab("Proteins")
Proteins:AddSwitch("Autoeat Proteins", function(bool)
    local Players = game:GetService("Players")
    local vim = game:GetService("VirtualInputManager")
    local player = Players.LocalPlayer

    local snacks = {
        "TOUGH Bar",
        "Protein Bar",
        "Tropical Shake",
        "Energy Shake",
        "Energy Bar"
    }

    task.spawn(function()
        while true do
            for _, snackName in ipairs(snacks) do
                local tool = player.Backpack:FindFirstChild(snackName)
                if tool then
                    tool.Parent = player.Character
                    task.wait(0.1)
                end
            end
            task.wait(1)
        end
    end)

    task.spawn(function()
        while true do
            vim:SendMouseButtonEvent(500, 500, 0, true, game, 1)
            task.wait()
            vim:SendMouseButtonEvent(500, 500, 0, false, game, 1)
        end
    end)
end)

----------------------------------------------------------------
-- Fast Glitch Tab (first instance)
local fastglitch = window:AddTab("Fast Glitch")

fastglitch:AddSwitch("Fast glitch", function(bool)
    local RockSection = fastglitch:CreateSection("Fast Glitch Rocks")
    local selectrock = ""
    getgenv().autoFarm = bool

    local function farmRock(neededDurability, rockName)
        while getgenv().autoFarm do
            task.wait()
            if game:GetService("Players").LocalPlayer.Durability.Value >= neededDurability then
                for _, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v.Value == neededDurability and 
                       game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and 
                       game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                        firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                        -- Assuming gettool() is defined elsewhere in the script.
                        gettool()
                    end
                end
            end
        end
    end

    RockSection:AddToggle("TinyIslandRock", {
        Title = "Fast Glitch Tiny Rock",
        Description = "Farm rocks at Tiny Island",
        Default = false,
        Callback = function(Value)
            selectrock = "Tiny Island Rock"
            getgenv().autoFarm = Value
            if Value then
                farmRock(0, "Tiny Island Rock")
            end
        end
    })

    RockSection:AddToggle("StarterIslandRock", {
        Title = "Fast Glitch Starter Island Rock",
        Description = "Farm rocks at Starter Island",
        Default = false,
        Callback = function(Value)
            selectrock = "Starter Island Rock"
            getgenv().autoFarm = Value
            if Value then
                farmRock(100, "Starter Island Rock")
            end
        end
    })

    RockSection:AddToggle("LegendBeachRock", {
        Title = "Fast Glitch Legend Beach Rock",
        Description = "Farm rocks at Legend Beach",
        Default = false,
        Callback = function(Value)
            selectrock = "Legend Beach Rock"
            getgenv().autoFarm = Value
            if Value then
                farmRock(5000, "Legend Beach Rock")
            end
        end
    })

    RockSection:AddToggle("FrostGymRock", {
        Title = "Fast Glitch Frost Rock",
        Description = "Farm rocks at Frost Gym",
        Default = false,
        Callback = function(Value)
            selectrock = "Frost Gym Rock"
            getgenv().autoFarm = Value
            if Value then
                farmRock(150000, "Frost Gym Rock")
            end
        end
    })

    RockSection:AddToggle("MythicalGymRock", {
        Title = "Fast Glitch Mythical Rock",
        Description = "Farm rocks at Mythical Gym",
        Default = false,
        Callback = function(Value)
            selectrock = "Mythical Gym Rock"
            getgenv().autoFarm = Value
            if Value then
                farmRock(400000, "Mythical Gym Rock")
            end
        end
    })

    RockSection:AddToggle("EternalGymRock", {
        Title = "Fast Glitch Eternal Rock",
        Description = "Farm rocks at Eternal Gym",
        Default = false,
        Callback = function(Value)
            selectrock = "Eternal Gym Rock"
            getgenv().autoFarm = Value
            while getgenv().autoFarm do
                task.wait()
                if game:GetService("Players").LocalPlayer.Durability.Value >= 750000 then
                    for _, v in pairs(game:GetService("Workspace").machinesFolder:GetDescendants()) do
                        if v.Name == "neededDurability" and v.Value == 750000 and 
                           game.Players.LocalPlayer.Character:FindFirstChild("LeftHand") and 
                           game.Players.LocalPlayer.Character:FindFirstChild("RightHand") then
                            firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 0)
                            firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.RightHand, 1)
                            firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 0)
                            firetouchinterest(v.Parent.Rock, game:GetService("Players").LocalPlayer.Character.LeftHand, 1)
                            gettool()
                        end
                    end
                end
            end
        end
    })
end)

----------------------------------------------------------------
-- Fast Glitch Tab (second instance for autopunch)
local fastglitch_autopunch = window:AddTab("Fast Glitch")

fastglitch_autopunch:AddSwitch("autopunch", function(bool)
    local function performPunchActions()
        local Players = game:GetService("Players")
        local player = Players.LocalPlayer
        if not player then
            warn("LocalPlayer not found")
            return
        end

        local muscleEvent = player:FindFirstChild("muscleEvent")
        if not muscleEvent then
            warn("muscleEvent not found on LocalPlayer")
            return
        end

        muscleEvent:FireServer("punch", "leftHand")
        muscleEvent:FireServer("punch", "rightHand")
    end

    performPunchActions()
end)

----------------------------------------------------------------
-- Anti-AFK Tab and its functionality

-- Variables and connection holders for Anti-AFK
local antiAfkEnabled = false
local antiAfkConnections = {}

-- Function to start Anti-AFK
local function startAntiAfk()
    antiAfkEnabled = true

    local Players = game:GetService("Players")
    local VirtualUser = game:GetService("VirtualUser")
    local UIS = game:GetService("UserInputService")
    local localPlayer = Players.LocalPlayer
    if not localPlayer then
        warn("LocalPlayer not found for Anti-AFK")
        return
    end

    -- Disconnect idle by simulating activity
    local idledConnection = localPlayer.Idled:Connect(function()
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end)
    table.insert(antiAfkConnections, {conn = idledConnection})

    -- Create GUI to display Anti-AFK status and timer
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "AntiAfkGUI"
    ScreenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    local Frame = Instance.new("Frame")
    Frame.Parent = ScreenGui
    Frame.Size = UDim2.new(0, 250, 0, 90)
    Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Frame.BackgroundTransparency = 0

    local UICorner = Instance.new("UICorner")
    UICorner.Parent = Frame
    UICorner.CornerRadius = UDim.new(0, 20)

    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Parent = Frame
    TitleLabel.Size = UDim2.new(1, 0, 0.5, 0)
    TitleLabel.Position = UDim2.new(0, 0, 0, 0)
    TitleLabel.Text = "ANTI-AFK BY Darkiller"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextScaled = true
    TitleLabel.Font = Enum.Font.SourceSansBold

    local TimerLabel = Instance.new("TextLabel")
    TimerLabel.Parent = Frame
    TimerLabel.Size = UDim2.new(1, 0, 0.5, 0)
    TimerLabel.Position = UDim2.new(0, 0, 0.5, 0)
    TimerLabel.Text = "Time: 00:00:00"
    TimerLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TimerLabel.BackgroundTransparency = 1
    TimerLabel.TextScaled = true
    TimerLabel.Font = Enum.Font.SourceSansBold

    -- Enable dragging functionality for the frame
    local dragging, dragInput, dragStart, startPos
    local function update(input)
        local delta = input.Position - dragStart
        Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    Frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = Frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    Frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)

    -- Create a BillboardGui to display text above the player's head
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Parent = workspace
    billboardGui.Adornee = localPlayer.Character and localPlayer.Character:WaitForChild("Head")
    billboardGui.Size = UDim2.new(0, 200, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, 3, 0)

    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = billboardGui
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.Text = "ANTI-AFK BY Darkiller"
    textLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    textLabel.BackgroundTransparency = 1
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.SourceSansBold

    -- Run timer to update the timer label
    spawn(function()
        local seconds = 0
        while antiAfkEnabled do
            wait(1)
            seconds = seconds + 1
            local hours = math.floor(seconds / 3600)
            local minutes = math.floor((seconds % 3600) / 60)
            local secs = seconds % 60
            TimerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
        end
    end)
end

-- Function to stop Anti-AFK and clean up
local function stopAntiAfk()
    antiAfkEnabled = false
    for _, info in ipairs(antiAfkConnections) do
        if info.conn and info.conn.Disconnect then
            info.conn:Disconnect()
        end
    end
    antiAfkConnections = {}
    local playerGui = game.Players.LocalPlayer and game.Players.LocalPlayer:FindFirstChild("PlayerGui")
    if playerGui then
        local gui = playerGui:FindFirstChild("AntiAfkGUI")
        if gui then
            gui:Destroy()
        end
    end
end

-- Create Anti-AFK Tab in window
local antiafk = window:AddTab("antiafk")
antiafk:AddSwitch("antiafk", function(bool)
    if bool then
        startAntiAfk()
    else
        stopAntiAfk()
    end
end)

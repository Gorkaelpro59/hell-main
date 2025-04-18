-- Ensure LocalPlayer and Character exist
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser") -- Added for Anti-AFK

local player = Players.LocalPlayer
if not player then
    error("LocalPlayer not found!")
end
if not player.Character then
    player.CharacterAdded:Wait()
end

-- Wait for HumanoidRootPart to be available
local function getHumanoidRootPart(character)
    if not character then return nil end -- Added check
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        -- Use WaitForChild with a timeout to prevent infinite yields
        hrp = character:WaitForChild("HumanoidRootPart", 10)
    end
    return hrp
end

-- Dummy definition for gettool if it doesn't exist to prevent nil errors.
-- Assuming gettool might be provided by the execution environment or library
if not gettool then
    gettool = function()
        -- This is a placeholder. The actual implementation should be provided elsewhere.
        -- warn("gettool function is not defined, using placeholder.")
    end
end

-- Global flag for fast rebirth
local fastRebirth = false
getgenv().autoFarm = false -- Initialize autoFarm in getgenv
_G.RepOP = false -- Initialize RepOP global

-- Library loading and whitelist check
local success, library = pcall(function()
    local rawLib = game:HttpGet("https://pastebin.com/raw/Abg3RkND", true)
    if not rawLib or rawLib == "" then
        return nil, "Failed to fetch library code or it was empty."
    end
    local loadedFunc, err = loadstring(rawLib)
    if not loadedFunc then
        return nil, "Failed to load library string: " .. tostring(err)
    end
    return loadedFunc()
end)

if not success then
    error("Error during library loading: " .. tostring(library)) -- library contains the error message here
elseif not library then
    error("The loaded library is nil.")
end

-- Check if essential library functions exist
if typeof(library) ~= "table" or not library.AddWindow then
     error("Loaded library is not valid or missing AddWindow method.")
end

local window = library:AddWindow("HELL clan Script by darkiller", { main_color = Color3.fromRGB(41, 74, 122), min_size = Vector2.new(600, 550), can_resize = false })
if not window or not window.AddTab then
    error("Failed to create window or window is missing AddTab method.")
end

-- Whitelist system using usernames
local whitelist = {
    "Quiraz29",
    "Shadow_RipperZ0",
    "HoloGrindz",
    "Xxdarkiller_10ZKT2",
    "TARZANfeio",
    "Pathaan122378",
    "Gorkaelpro59",
    "VYXONON_TOP",
}

local function isWhitelisted(p)
    if not p then return false end
    for _, username in ipairs(whitelist) do
        if p.Name == username then
            return true
        end
    end
    return false
end

if not isWhitelisted(player) then
    -- Use library notification if available, otherwise error
    if library and library.Notify then
        library:Notify("Not Whitelisted", "You are not whitelisted to use this script.", 5)
    else
        warn("You are not whitelisted to use this script.")
    end
    return -- Stop script execution if not whitelisted
end

----------------------------------------------------------------
-- Paid Tab and its functionalities
local Paid = window:AddTab("Paid")
if not Paid or not Paid.AddSwitch then
    error("Failed to create Paid tab or tab is missing AddSwitch method.")
end

local fastRebirthThread -- To manage the rebirth thread

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth then
        -- Terminate existing thread if running
        if fastRebirthThread and coroutine.status(fastRebirthThread) ~= "dead" then
            task.cancel(fastRebirthThread)
        end
        -- Start new thread
        fastRebirthThread = task.spawn(function()
            local rEvents = ReplicatedStorage:FindFirstChild("rEvents")
            if not rEvents then
                warn("rEvents not found in ReplicatedStorage.")
                fastRebirth = false -- Stop the process
                return
            end
            local equipPetEvent = rEvents:FindFirstChild("equipPetEvent")
            local rebirthRemote = rEvents:FindFirstChild("rebirthRemote")
            if not equipPetEvent or not rebirthRemote then
                 warn("Required remote events/functions (equipPetEvent, rebirthRemote) not found under rEvents.")
                 fastRebirth = false -- Stop the process
                 return
            end

            local c = Players.LocalPlayer -- Use 'c' as in original code

            local function d() -- unequip all pets
                if not c or not c:FindFirstChild("petsFolder") then return end
                local f = c.petsFolder
                for _, folder in pairs(f:GetChildren()) do
                    if folder:IsA("Folder") then
                        for _, pet in pairs(folder:GetChildren()) do
                            if pet and equipPetEvent then
                                equipPetEvent:FireServer("unequipPet", pet)
                            end
                        end
                    end
                end
                task.wait(.1)
            end

            local function k(l) -- equip specific pet
                if not c or not c:FindFirstChild("petsFolder") then return end
                d() -- Ensure others are unequipped first
                task.wait(.01)
                local uniqueFolder = c.petsFolder:FindFirstChild("Unique")
                if not uniqueFolder then return end
                for _, pet in pairs(uniqueFolder:GetChildren()) do
                    if pet and pet.Name == l and equipPetEvent then
                        equipPetEvent:FireServer("equipPet", pet)
                        break -- Assume only one pet with this name needs equipping
                    end
                end
            end

            local function o(p) -- find machine
                local machinesFolder = workspace:FindFirstChild("machinesFolder")
                if not machinesFolder then
                    -- Fallback search (as in original code)
                    for _, s in pairs(workspace:GetChildren()) do
                        if s:IsA("Folder") and s.Name:find("machines") then
                            machinesFolder = s
                            break
                        end
                    end
                end
                if not machinesFolder then return nil end
                return machinesFolder:FindFirstChild(p)
            end

            local function t() -- simulate 'E' key press
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                task.wait(.1)
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
            end

            while fastRebirth and task.wait() do -- Check flag at start of loop and wait
                if not c or not c.Character or not c.Character:FindFirstChild("Humanoid") then
                    warn("Player, Character or Humanoid not found during fast rebirth loop.")
                    task.wait(1) -- Wait before retrying
                    c = Players.LocalPlayer -- Re-get player just in case
                    if not c then fastRebirth = false; break end -- Stop if player is gone
                    if not c.Character then player.CharacterAdded:Wait() end -- Wait for character respawn
                    if not c.Character then fastRebirth = false; break end -- Stop if character still not available
                    continue
                end

                local leaderstats = c:FindFirstChild("leaderstats")
                local rebirthsStat = leaderstats and leaderstats:FindFirstChild("Rebirths")
                local strengthStat = leaderstats and leaderstats:FindFirstChild("Strength")
                local muscleEvent = c:FindFirstChild("muscleEvent")

                if not leaderstats or not rebirthsStat or not strengthStat or not muscleEvent then
                    warn("Missing leaderstats, Rebirths, Strength, or muscleEvent.")
                    task.wait(1)
                    continue -- Skip this iteration if stats/event missing
                end

                local v = rebirthsStat.Value -- Current rebirths
                local w = 10000 + (5000 * v) -- Target strength

                local ultimatesFolder = c:FindFirstChild("ultimatesFolder")
                local goldenRebirth = ultimatesFolder and ultimatesFolder:FindFirstChild("Golden Rebirth")
                if goldenRebirth and goldenRebirth:IsA("IntValue") then -- Check type
                    local x = goldenRebirth.Value
                    w = math.floor(w * (1 - (x * 0.1)))
                end

                d() -- Unequip pets
                task.wait(.1)
                k("Swift Samurai") -- Equip strength pet

                while fastRebirth and strengthStat.Value < w do
                    for _ = 1, 10 do
                        muscleEvent:FireServer("rep")
                    end
                    task.wait() -- Yield inside inner loop
                end
                if not fastRebirth then break end -- Exit if toggled off during strength gain

                d() -- Unequip pets
                task.wait(.1)
                k("Tribal Overlord") -- Equip durability pet (assuming)

                local z = o("Jungle Bar Lift") -- Find machine
                local hrp = getHumanoidRootPart(c.Character)
                if z and hrp then
                    local interactSeat = z:FindFirstChild("interactSeat")
                    if interactSeat and interactSeat:IsA("Seat") then -- Check type
                        hrp.CFrame = interactSeat.CFrame * CFrame.new(0, 3, 0)
                        local sitStartTime = tick()
                        repeat
                            task.wait(.1)
                            t() -- Press E
                            -- Add timeout to prevent infinite loop if sitting fails
                        until not fastRebirth or (c.Character and c.Character:FindFirstChildOfClass("Humanoid") and c.Character.Humanoid.Sit) or (tick() - sitStartTime > 5)
                        if not fastRebirth then break end -- Exit if toggled off
                        if not (c.Character and c.Character:FindFirstChildOfClass("Humanoid") and c.Character.Humanoid.Sit) then
                            warn("Failed to sit on Jungle Bar Lift.")
                        end
                    else
                        warn("Jungle Bar Lift interactSeat not found or not a Seat.")
                    end
                else
                    warn("Jungle Bar Lift machine or HumanoidRootPart not found.")
                end

                local A = rebirthsStat.Value -- Rebirths before attempting
                local rebirthStartTime = tick()
                repeat
                    if rebirthRemote then
                        rebirthRemote:InvokeServer("rebirthRequest")
                    end
                    task.wait(.1)
                    -- Re-fetch rebirthsStat in case leaderstats was reset
                    leaderstats = c:FindFirstChild("leaderstats")
                    rebirthsStat = leaderstats and leaderstats:FindFirstChild("Rebirths")
                    -- Add timeout to prevent infinite loop if rebirth fails
                until not fastRebirth or not rebirthsStat or rebirthsStat.Value > A or (tick() - rebirthStartTime > 10)
                if not fastRebirth then break end -- Exit if toggled off
                if not rebirthsStat or rebirthsStat.Value <= A then
                    warn("Rebirth failed or timed out.")
                end
                task.wait() -- Wait before next cycle
            end
        end)
    elseif fastRebirthThread and coroutine.status(fastRebirthThread) ~= "dead" then
        task.cancel(fastRebirthThread)
        fastRebirthThread = nil
    end
end)

local fastStrengthThread -- To manage the strength thread

Paid:AddSwitch("Fast Strength", function(Value)
    _G.RepOP = Value
    if Value then
        -- Terminate existing thread if running
        if fastStrengthThread and coroutine.status(fastStrengthThread) ~= "dead" then
            task.cancel(fastStrengthThread)
        end
        -- Start new thread
        fastStrengthThread = task.spawn(function()
            local c = Players.LocalPlayer
            while _G.RepOP do -- Use _G and check flag
                local muscleEvent = c and c:FindFirstChild("muscleEvent")
                if muscleEvent then
                    for _ = 1, 30 do
                        muscleEvent:FireServer("rep")
                    end
                else
                    warn("muscleEvent not found for Fast Strength.")
                    _G.RepOP = false -- Stop if event is missing
                    break
                end
                task.wait(0.0001) -- Use task.wait for better performance/yielding
            end
        end)
    elseif fastStrengthThread and coroutine.status(fastStrengthThread) ~= "dead" then
        task.cancel(fastStrengthThread)
        fastStrengthThread = nil
    end
end)

Paid:AddSwitch("Hide Frame", function(bool)
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        -- Assuming these frames are in PlayerGui, not ReplicatedStorage, as they are UI elements
        local playerGui = player:FindFirstChild("PlayerGui")
        if playerGui then
             local frame = playerGui:FindFirstChild(frameName, true) -- Search recursively
             if frame and frame:IsA("GuiObject") then
                 frame.Visible = not bool
             else
                 -- Fallback check in ReplicatedStorage as per original code
                 local rsFrame = ReplicatedStorage:FindFirstChild(frameName)
                 if rsFrame and rsFrame:IsA("GuiObject") then
                     -- This is unusual, UI templates shouldn't be made visible in ReplicatedStorage
                     -- warn("Found UI frame '" .. frameName .. "' in ReplicatedStorage. Hiding/showing it here might not be intended.")
                     -- rsFrame.Visible = not bool -- Avoid modifying templates directly unless intended
                 end
             end
        end
    end
end)

----------------------------------------------------------------
-- Position and Teleport Tab
local PositionAndTeleport = window:AddTab("Position and Teleport")
if not PositionAndTeleport or not PositionAndTeleport.AddSwitch then
    error("Failed to create PositionAndTeleport tab or tab is missing AddSwitch method.")
end

local lockpos = false
local cp
local lockPosConnection = nil

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool
    local currentCharacter = player.Character
    local hrp = getHumanoidRootPart(currentCharacter)
    if lockpos and hrp then
        cp = hrp.Position  -- Save current position on enabling lock
        if not lockPosConnection then
            lockPosConnection = RunService.Heartbeat:Connect(function()
                -- Check lockpos flag inside the connection
                if not lockpos then
                    if lockPosConnection then
                        lockPosConnection:Disconnect()
                        lockPosConnection = nil
                    end
                    return
                end
                -- Re-validate character and HRP as they might change (respawn)
                local char = player.Character
                local currentHrp = getHumanoidRootPart(char)
                if currentHrp and cp then -- Ensure cp was set
                    currentHrp.CFrame = CFrame.new(cp)
                    currentHrp.Velocity = Vector3.new(0, 0, 0)
                    currentHrp.RotVelocity = Vector3.new(0, 0, 0)
                elseif lockpos then
                    -- If lock is still desired but HRP is gone, try to re-acquire cp when HRP reappears
                    -- Or simply stop locking if HRP is lost. Let's stop for simplicity.
                    -- warn("HRP not found while lockposition active. Disabling.")
                    -- lockpos = false
                    -- if lockPosConnection then lockPosConnection:Disconnect(); lockPosConnection = nil end
                end
            end)
        end
    elseif lockPosConnection then
        lockPosConnection:Disconnect()
        lockPosConnection = nil
    end
end)

----------------------------------------------------------------
-- Proteins Tab
local Proteins = window:AddTab("Proteins")
if not Proteins or not Proteins.AddSwitch then
    error("Failed to create Proteins tab or tab is missing AddSwitch method.")
end

local autoEatEnabled = false
local autoEatEquipThread = nil
local autoEatClickThread = nil

Proteins:AddSwitch("Autoeat Proteins", function(bool)
    autoEatEnabled = bool

    if autoEatEnabled then
        -- Terminate existing threads if running
        if autoEatEquipThread and coroutine.status(autoEatEquipThread) ~= "dead" then task.cancel(autoEatEquipThread) end
        if autoEatClickThread and coroutine.status(autoEatClickThread) ~= "dead" then task.cancel(autoEatClickThread) end

        local snacks = {
            "TOUGH Bar", "Protein Bar", "Tropical Shake", "Energy Shake", "Energy Bar"
        }

        -- Thread for equipping tools
        autoEatEquipThread = task.spawn(function()
            while autoEatEnabled and task.wait(1) do -- Check flag and wait
                local currentCharacter = player.Character
                local backpack = player:FindFirstChild("Backpack")
                if not currentCharacter or not backpack then
                    warn("Character or Backpack not found for Autoeat.")
                    task.wait(2) -- Wait longer if character/backpack missing
                    continue
                end
                for _, snackName in ipairs(snacks) do
                    if not autoEatEnabled then break end -- Check flag again inside loop
                    local tool = backpack:FindFirstChild(snackName)
                    if tool and tool:IsA("Tool") then -- Check if it's actually a tool
                        -- Check if tool is already equipped to avoid unnecessary parenting
                        if tool.Parent ~= currentCharacter then
                             tool.Parent = currentCharacter
                             task.wait(0.2) -- Give time for equip animation/effect
                        end
                    end
                end
            end
        end)

        -- Thread for clicking
        autoEatClickThread = task.spawn(function()
            while autoEatEnabled and task.wait() do -- Check flag and wait minimally
                -- Ensure coordinates are within screen bounds, though 500,500 is likely safe
                VirtualInputManager:SendMouseButtonEvent(500, 500, 0, true, game, 1)
                task.wait() -- Minimal delay between down and up
                VirtualInputManager:SendMouseButtonEvent(500, 500, 0, false, game, 1)
                -- Add a small delay to prevent excessive clicking if no tool is equipped
                task.wait(0.1)
            end
        end)

    else
        -- Stop threads when switch is turned off
        if autoEatEquipThread and coroutine.status(autoEatEquipThread) ~= "dead" then task.cancel(autoEatEquipThread); autoEatEquipThread = nil end
        if autoEatClickThread and coroutine.status(autoEatClickThread) ~= "dead" then task.cancel(autoEatClickThread); autoEatClickThread = nil end
    end
end)

----------------------------------------------------------------
-- Fast Glitch Tab (combined logic)
local fastglitch = window:AddTab("Fast Glitch")
if not fastglitch or not fastglitch.AddSwitch or not fastglitch.CreateSection then
    error("Failed to create Fast Glitch tab or tab is missing methods.")
end

local fastGlitchEnabled = false
local autoPunchEnabled = false
local currentFarmRock = nil -- Name of the rock being farmed
local farmRockThread = nil
local autoPunchThread = nil

-- Main switch for the entire feature (optional, could be removed if only toggles are desired)
fastglitch:AddSwitch("Enable Fast Glitch Features", function(bool)
    fastGlitchEnabled = bool
    getgenv().autoFarm = bool -- Link to the global flag used in original rock farm logic
    if not bool then
        -- Stop farming if the main switch is turned off
        if farmRockThread and coroutine.status(farmRockThread) ~= "dead" then
            task.cancel(farmRockThread)
            farmRockThread = nil
        end
        currentFarmRock = nil
        -- Also update the global flag if controlled externally
        getgenv().autoFarm = false
    end
    -- Note: This main switch doesn't directly control autopunch in this revised logic.
    -- Autopunch has its own switch.
end)

-- Section for Rock Farming
local RockSection = fastglitch:CreateSection("Fast Glitch Rocks")
if not RockSection or not RockSection.AddToggle then
     error("Failed to create RockSection or section is missing AddToggle method.")
end

local function stopFarmRockThread()
    if farmRockThread and coroutine.status(farmRockThread) ~= "dead" then
        task.cancel(farmRockThread)
        farmRockThread = nil
    end
    currentFarmRock = nil
    getgenv().autoFarm = false -- Ensure global flag is off
end

local function startFarmRock(neededDurability, rockName)
    stopFarmRockThread() -- Stop any previous farming first
    currentFarmRock = rockName
    getgenv().autoFarm = true -- Set global flag

    farmRockThread = task.spawn(function()
        local machinesFolder = workspace:FindFirstChild("machinesFolder")
        if not machinesFolder then
            warn("machinesFolder not found for rock farming.")
            getgenv().autoFarm = false
            return
        end

        -- Check if firetouchinterest exists globally or in the environment
        local touchFunc = firetouchinterest
        if not touchFunc then
            warn("firetouchinterest function not found. Rock farming will not work.")
            getgenv().autoFarm = false
            return
        end

        while getgenv().autoFarm and currentFarmRock == rockName and task.wait() do
            local c = Players.LocalPlayer
            if not c or not c.Character then
                warn("Player or Character not found during rock farm.")
                task.wait(1)
                continue
            end

            local durabilityStat = c:FindFirstChild("Durability") -- Assuming Durability is direct child of player
            if not durabilityStat or not durabilityStat:IsA("IntValue") then -- Check type
                warn("Durability stat not found or not an IntValue.")
                task.wait(1)
                continue
            end

            if durabilityStat.Value >= neededDurability then
                local leftHand = c.Character:FindFirstChild("LeftHand")
                local rightHand = c.Character:FindFirstChild("RightHand")

                if not leftHand or not rightHand then
                    warn("Player hands not found.")
                    task.wait(0.5)
                    continue
                end

                local foundRock = false
                for _, v in pairs(machinesFolder:GetDescendants()) do
                    if v.Name == "neededDurability" and v:IsA("IntValue") and v.Value == neededDurability then
                        local parentMachine = v.Parent
                        local rockPart = parentMachine and parentMachine:FindFirstChild("Rock")
                        if rockPart then
                            foundRock = true
                            -- Fire touch events
                            touchFunc(rockPart, rightHand, 0)
                            touchFunc(rockPart, rightHand, 1)
                            touchFunc(rockPart, leftHand, 0)
                            touchFunc(rockPart, leftHand, 1)
                            -- Call gettool (placeholder or actual)
                            if gettool then gettool() end
                            break -- Assume we only need to interact with one rock of this type per cycle
                        end
                    end
                end
                -- if not foundRock then warn("No rock found for durability:", neededDurability) end
            else
                 task.wait(0.5) -- Wait if durability is too low
            end
        end
        -- Clean up if loop exits naturally (e.g., rock changed)
        if currentFarmRock == rockName then
             currentFarmRock = nil
             getgenv().autoFarm = false
        end
    end)
end

-- Rock Toggles using the centralized function
local rockConfigs = {
    { Name = "TinyIslandRock", Title = "Fast Glitch Tiny Rock", Desc = "Farm rocks at Tiny Island", Durability = 0 },
    { Name = "StarterIslandRock", Title = "Fast Glitch Starter Island Rock", Desc = "Farm rocks at Starter Island", Durability = 100 },
    { Name = "LegendBeachRock", Title = "Fast Glitch Legend Beach Rock", Desc = "Farm rocks at Legend Beach", Durability = 5000 },
    { Name = "FrostGymRock", Title = "Fast Glitch Frost Rock", Desc = "Farm rocks at Frost Gym", Durability = 150000 },
    { Name = "MythicalGymRock", Title = "Fast Glitch Mythical Rock", Desc = "Farm rocks at Mythical Gym", Durability = 400000 },
    { Name = "EternalGymRock", Title = "Fast Glitch Eternal Rock", Desc = "Farm rocks at Eternal Gym", Durability = 750000 },
}

for _, config in ipairs(rockConfigs) do
    RockSection:AddToggle(config.Name, {
        Title = config.Title,
        Description = config.Desc,
        Default = false,
        Callback = function(Value)
            if Value then
                -- Start farming this rock (will stop others)
                startFarmRock(config.Durability, config.Name)
            else
                -- Stop farming only if this specific rock was being farmed
                if currentFarmRock == config.Name then
                    stopFarmRockThread()
                end
            end
        end
    })
end

-- Section for Autopunch
local AutopunchSection = fastglitch:CreateSection("Fast Glitch Autopunch")
if not AutopunchSection or not AutopunchSection.AddSwitch then
     error("Failed to create AutopunchSection or section is missing AddSwitch method.")
end

AutopunchSection:AddSwitch("autopunch", function(bool)
    autoPunchEnabled = bool
    if autoPunchEnabled then
        -- Terminate existing thread if running
        if autoPunchThread and coroutine.status(autoPunchThread) ~= "dead" then
            task.cancel(autoPunchThread)
        end
        -- Start new thread
        autoPunchThread = task.spawn(function()
            while autoPunchEnabled and task.wait() do -- Check flag and wait minimally
                local c = Players.LocalPlayer
                local muscleEvent = c and c:FindFirstChild("muscleEvent")
                if muscleEvent then
                    -- Perform punch actions
                    muscleEvent:FireServer("punch", "leftHand")
                    muscleEvent:FireServer("punch", "rightHand")
                else
                    warn("muscleEvent not found for autopunch.")
                    autoPunchEnabled = false -- Stop if event is missing
                    break
                end
                -- Add a small delay between punch pairs if needed
                -- task.wait(0.1)
            end
        end)
    elseif autoPunchThread and coroutine.status(autoPunchThread) ~= "dead" then
        task.cancel(autoPunchThread)
        autoPunchThread = nil
    end
end)

----------------------------------------------------------------
-- Anti-AFK Tab and its functionality

local antiafk = window:AddTab("antiafk")
if not antiafk or not antiafk.AddSwitch then
    error("Failed to create antiafk tab or tab is missing AddSwitch method.")
end

-- Variables and connection holders for Anti-AFK
local antiAfkEnabled = false
local antiAfkConnections = {}
local antiAfkGui = nil
local antiAfkBillboard = nil
local antiAfkTimerThread = nil

-- Function to clean up Anti-AFK resources
local function cleanupAntiAfk()
    antiAfkEnabled = false -- Ensure flag is set to false

    -- Disconnect all connections
    for _, info in ipairs(antiAfkConnections) do
        if info.conn and info.conn.Connected then -- Check if connected before disconnecting
            info.conn:Disconnect()
        end
    end
    antiAfkConnections = {} -- Clear the table

    -- Stop timer thread
    if antiAfkTimerThread and coroutine.status(antiAfkTimerThread) ~= "dead" then
        task.cancel(antiAfkTimerThread)
        antiAfkTimerThread = nil
    end

    -- Destroy GUI elements
    if antiAfkGui and antiAfkGui.Parent then
        antiAfkGui:Destroy()
    end
    antiAfkGui = nil

    if antiAfkBillboard and antiAfkBillboard.Parent then
        antiAfkBillboard:Destroy()
    end
    antiAfkBillboard = nil
end

-- Function to start Anti-AFK
local function startAntiAfk()
    cleanupAntiAfk() -- Clean up any previous instances first
    antiAfkEnabled = true

    local localPlayer = Players.LocalPlayer
    if not localPlayer then
        warn("LocalPlayer not found for Anti-AFK")
        antiAfkEnabled = false
        return
    end

    local playerGui = localPlayer:WaitForChild("PlayerGui") -- Ensure PlayerGui exists

    -- Disconnect idle by simulating activity
    local idledConnection = localPlayer.Idled:Connect(function()
        -- Check if still enabled before acting
        if antiAfkEnabled then
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end
    end)
    table.insert(antiAfkConnections, {conn = idledConnection})

    -- Create GUI to display Anti-AFK status and timer
    antiAfkGui = Instance.new("ScreenGui")
    antiAfkGui.Name = "AntiAfkGUI"
    antiAfkGui.ResetOnSpawn = false -- Keep GUI across respawns
    antiAfkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling -- Or Global if needed on top
    antiAfkGui.Parent = playerGui

    local Frame = Instance.new("Frame")
    Frame.Parent = antiAfkGui
    Frame.Size = UDim2.new(0, 250, 0, 90)
    Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- Darker background
    Frame.BackgroundTransparency = 0.2
    Frame.BorderSizePixel = 0

    local UICorner = Instance.new("UICorner")
    UICorner.Parent = Frame
    UICorner.CornerRadius = UDim.new(0, 12)

    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Parent = Frame
    TitleLabel.Size = UDim2.new(1, -10, 0.5, 0) -- Padding
    TitleLabel.Position = UDim2.new(0, 5, 0, 0)
    TitleLabel.Text = "ANTI-AFK BY Darkiller"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextScaled = true
    TitleLabel.Font = Enum.Font.SourceSansBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Center
    TitleLabel.TextYAlignment = Enum.TextYAlignment.Center

    local TimerLabel = Instance.new("TextLabel")
    TimerLabel.Parent = Frame
    TimerLabel.Size = UDim2.new(1, -10, 0.5, 0) -- Padding
    TimerLabel.Position = UDim2.new(0, 5, 0.5, 0)
    TimerLabel.Text = "Time: 00:00:00"
    TimerLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TimerLabel.BackgroundTransparency = 1
    TimerLabel.TextScaled = true
    TimerLabel.Font = Enum.Font.SourceSansBold
    TimerLabel.TextXAlignment = Enum.TextXAlignment.Center
    TimerLabel.TextYAlignment = Enum.TextYAlignment.Center

    -- Enable dragging functionality for the frame
    Frame.Active = true -- Make frame interactive
    Frame.Draggable = true -- Use built-in draggable property

    -- Create a BillboardGui to display text above the player's head
    local function setupBillboard()
        if antiAfkBillboard and antiAfkBillboard.Parent then antiAfkBillboard:Destroy() end -- Remove old one if exists

        local char = localPlayer.Character
        local head = char and char:FindFirstChild("Head")
        if head then
            antiAfkBillboard = Instance.new("BillboardGui")
            antiAfkBillboard.Name = "AntiAfkBillboard"
            antiAfkBillboard.Adornee = head
            antiAfkBillboard.Size = UDim2.new(0, 200, 0, 50)
            antiAfkBillboard.StudsOffset = Vector3.new(0, 3, 0)
            antiAfkBillboard.AlwaysOnTop = true
            antiAfkBillboard.ResetOnSpawn = false -- Keep across respawns (though Adornee needs reset)
            antiAfkBillboard.Parent = head -- Parent to head for auto-cleanup on death

            local textLabel = Instance.new("TextLabel")
            textLabel.Parent = antiAfkBillboard
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.Text = "ANTI-AFK BY Darkiller"
            textLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
            textLabel.BackgroundTransparency = 1
            textLabel.TextScaled = true
            textLabel.Font = Enum.Font.SourceSansBold
        else
            warn("Could not find player Head for Anti-AFK Billboard.")
        end
    end

    setupBillboard() -- Initial setup
    -- Re-setup billboard on character added
    local charAddedConn = localPlayer.CharacterAdded:Connect(function(character)
        if antiAfkEnabled then
             task.wait(0.5) -- Wait briefly for parts to load
             local head = character:FindFirstChild("Head")
             if head and antiAfkBillboard then
                 antiAfkBillboard.Adornee = head
                 antiAfkBillboard.Enabled = true -- Ensure it's enabled
                 if antiAfkBillboard.Parent ~= head then antiAfkBillboard.Parent = head end -- Re-parent if needed
             elseif head and not antiAfkBillboard then
                 setupBillboard() -- Create if missing
             elseif antiAfkBillboard then
                 antiAfkBillboard.Enabled = false -- Disable if no head found
             end
        end
    end)
    table.insert(antiAfkConnections, {conn = charAddedConn})

    -- Run timer to update the timer label
    antiAfkTimerThread = task.spawn(function()
        local seconds = 0
        while antiAfkEnabled do
            task.wait(1)
            seconds = seconds + 1
            local hours = math.floor(seconds / 3600)
            local minutes = math.floor((seconds % 3600) / 60)
            local secs = seconds % 60
            -- Check if TimerLabel still exists before updating
            if TimerLabel and TimerLabel.Parent then
                TimerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
            else
                -- If label is gone, stop the timer (GUI was likely destroyed)
                warn("Anti-AFK TimerLabel lost.")
                antiAfkEnabled = false -- Stop the timer loop
                break
            end
        end
    end)
end

-- Create Anti-AFK Tab switch
antiafk:AddSwitch("antiafk", function(bool)
    if bool then
        startAntiAfk()
    else
        cleanupAntiAfk()
    end
end)

-- Add cleanup hook for when script stops or player leaves
game:GetService("Players").LocalPlayerRemoving:Connect(cleanupAntiAfk)


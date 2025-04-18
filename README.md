-- Ensure LocalPlayer and Character exist
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
if not player then
    error("LocalPlayer not found!")
end
if not player.Character then
    player.CharacterAdded:Wait()
end

-- Wait for HumanoidRootPart to be available
local function getHumanoidRootPart(character)
    if not character then return nil end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        hrp = character:WaitForChild("HumanoidRootPart")
    end
    return hrp
end

-- Dummy definition for gettool if it doesn't exist to prevent nil errors.
if not gettool then
    function gettool()
        -- This is a placeholder. The actual implementation should be provided elsewhere.
        -- warn("gettool() is not defined, using placeholder.")
    end
end

-- Dummy definition for firetouchinterest if it doesn't exist
if not firetouchinterest then
    function firetouchinterest(part1, part2, touchType)
        -- This is a placeholder. The actual implementation should be provided elsewhere.
        -- warn("firetouchinterest() is not defined, using placeholder.")
    end
end


-- Global flag for fast rebirth
local fastRebirth = false
local fastRebirthThread = nil

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
    -- Consider using library:Notify or a similar non-blocking notification
    warn("You are not whitelisted to use this script.")
    -- error("You are not whitelisted to use this script.") -- error might be too disruptive
    return -- Stop script execution if not whitelisted
end

----------------------------------------------------------------
-- Paid Tab and its functionalities
local Paid = window:AddTab("Paid")

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth and not fastRebirthThread then
        fastRebirthThread = task.spawn(function()
            local c = Players.LocalPlayer
            local rsEvents = ReplicatedStorage:FindFirstChild("rEvents")
            local muscleEvent = c:FindFirstChild("muscleEvent")
            local petsFolder = c:FindFirstChild("petsFolder")
            local ultimatesFolder = c:FindFirstChild("ultimatesFolder")
            local leaderstats = c:FindFirstChild("leaderstats")

            if not rsEvents then warn("rEvents not found in ReplicatedStorage"); return end
            local equipPetEvent = rsEvents:FindFirstChild("equipPetEvent")
            local rebirthRemote = rsEvents:FindFirstChild("rebirthRemote")
            if not equipPetEvent then warn("equipPetEvent not found in rEvents"); return end
            if not rebirthRemote then warn("rebirthRemote not found in rEvents"); return end
            if not muscleEvent then warn("muscleEvent not found on player"); return end
            if not petsFolder then warn("petsFolder not found on player"); return end
            if not leaderstats then warn("leaderstats not found on player"); return end

            local strengthStat = leaderstats:FindFirstChild("Strength")
            local rebirthsStat = leaderstats:FindFirstChild("Rebirths")
            if not strengthStat then warn("Strength stat not found in leaderstats"); return end
            if not rebirthsStat then warn("Rebirths stat not found in leaderstats"); return end

            local function d() -- unequip all pets
                local currentPetsFolder = c:FindFirstChild("petsFolder")
                if not currentPetsFolder then return end
                for _, folder in ipairs(currentPetsFolder:GetChildren()) do
                    if folder:IsA("Folder") then
                        for _, pet in ipairs(folder:GetChildren()) do
                            if pet:IsA("StringValue") then -- Assuming pets are represented by StringValues or similar
                                equipPetEvent:FireServer("unequipPet", pet)
                            end
                        end
                    end
                end
                task.wait(.1)
            end

            local function k(petName) -- equip specific pet
                d() -- Ensure others are unequipped first
                task.wait(.01)
                local currentPetsFolder = c:FindFirstChild("petsFolder")
                local uniqueFolder = currentPetsFolder and currentPetsFolder:FindFirstChild("Unique")
                if not uniqueFolder then return end

                for _, pet in ipairs(uniqueFolder:GetChildren()) do
                    if pet.Name == petName and pet:IsA("StringValue") then
                        equipPetEvent:FireServer("equipPet", pet)
                        break -- Found and equipped, exit loop
                    end
                end
            end

            local function o(machineName) -- find machine
                local machinesFolder = Workspace:FindFirstChild("machinesFolder")
                local foundMachine = machinesFolder and machinesFolder:FindFirstChild(machineName)
                if not foundMachine then
                    for _, child in ipairs(Workspace:GetChildren()) do
                        if child:IsA("Folder") and child.Name:find("machines") then
                            foundMachine = child:FindFirstChild(machineName)
                            if foundMachine then break end
                        end
                    end
                end
                return foundMachine
            end

            local function t() -- simulate 'E' key press
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                task.wait(.1)
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
            end

            while fastRebirth and task.wait() do
                local char = c.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                local currentHrp = getHumanoidRootPart(char)
                local currentRebirths = rebirthsStat.Value or 0
                local neededStrength = 10000 + (5000 * currentRebirths)

                local goldenRebirth = ultimatesFolder and ultimatesFolder:FindFirstChild("Golden Rebirth")
                if goldenRebirth and goldenRebirth:IsA("IntValue") then -- Check type
                    neededStrength = math.floor(neededStrength * (1 - (goldenRebirth.Value * 0.1)))
                end

                d() -- Unequip pets
                task.wait(.1)
                k("Swift Samurai") -- Equip strength pet

                while fastRebirth and (strengthStat.Value or 0) < neededStrength do
                    for _ = 1, 10 do
                        muscleEvent:FireServer("rep")
                    end
                    task.wait() -- Yield for other processes and checks
                end
                if not fastRebirth then break end -- Exit if toggled off during strength gain

                d() -- Unequip pets
                task.wait(.1)
                k("Tribal Overlord") -- Equip rebirth pet

                local jungleBarLift = o("Jungle Bar Lift")
                local interactSeat = jungleBarLift and jungleBarLift:FindFirstChild("interactSeat")

                if interactSeat and interactSeat:IsA("Seat") and currentHrp and hum then
                    currentHrp.CFrame = interactSeat.CFrame * CFrame.new(0, 3, 0)
                    repeat
                        task.wait(.1)
                        t() -- Press E
                    until not fastRebirth or hum.Sit -- Stop if toggled off or sitting
                end
                if not fastRebirth then break end -- Exit if toggled off

                local initialRebirths = rebirthsStat.Value or 0
                repeat
                    local success, result = pcall(function()
                        return rebirthRemote:InvokeServer("rebirthRequest")
                    end)
                    if not success then
                        warn("Rebirth request failed:", result)
                        break -- Stop trying if invoke fails
                    end
                    task.wait(.1)
                until not fastRebirth or (rebirthsStat.Value or 0) > initialRebirths

                if hum and hum.Sit then -- Stand up after rebirth
                    hum.Sit = false
                end
            end
            fastRebirthThread = nil -- Clear the thread variable when done or stopped
        end)
    elseif not fastRebirth and fastRebirthThread then
        -- If toggled off, the loop inside the thread will stop itself
        -- No need to explicitly kill the thread, let it finish its current iteration
        -- fastRebirthThread = nil -- Resetting here might cause issues if the loop hasn't exited yet
    end
end)

local fastStrengthThread = nil
Paid:AddSwitch("Fast Strength", function(Value)
    _G.RepOP = Value -- Using _G is generally discouraged, consider local flags
    if Value and not fastStrengthThread then
        fastStrengthThread = task.spawn(function()
            local c = Players.LocalPlayer
            local muscleEvent = c and c:FindFirstChild("muscleEvent")
            if not muscleEvent then
                warn("Fast Strength: muscleEvent not found.")
                _G.RepOP = false -- Stop if event is missing
                fastStrengthThread = nil
                return
            end

            while _G.RepOP do
                for _ = 1, 30 do
                    muscleEvent:FireServer("rep")
                end
                task.wait(0.0001) -- Use task.wait() for adaptive yielding
            end
            fastStrengthThread = nil -- Clear thread variable when done
        end)
    elseif not Value and fastStrengthThread then
        -- The loop checks _G.RepOP, so setting it to false will stop the thread.
        -- fastStrengthThread = nil -- Resetting here might be premature
    end
end)

Paid:AddSwitch("Hide Frame", function(bool)
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        -- Assuming these frames are in PlayerGui, not ReplicatedStorage
        local playerGui = player:FindFirstChild("PlayerGui")
        local frame = playerGui and playerGui:FindFirstChild(frameName, true) -- Search recursively
        if frame and frame:IsA("GuiObject") then
            frame.Visible = not bool
        else
            -- If not in PlayerGui, check ReplicatedStorage as fallback
            local rsFrame = ReplicatedStorage:FindFirstChild(frameName)
            if rsFrame and rsFrame:IsA("GuiObject") then
                 -- Note: Modifying things in ReplicatedStorage might not affect the local UI
                 -- This part might need adjustment based on how these frames are used/cloned.
                 -- If they are cloned to PlayerGui, hiding the original might not work.
                 -- warn("Found frame in ReplicatedStorage, visibility change might not affect UI.")
                 -- rsFrame.Visible = not bool -- This line is likely ineffective for UI
            end
        end
    end
end)

----------------------------------------------------------------
-- Position and Teleport Tab
local PositionAndTeleport = window:AddTab("Position and Teleport")
local lockpos = false
local cp = nil -- Initialize cp to nil
local lockPosConnection = nil

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool
    local currentHrp = getHumanoidRootPart(player.Character)
    if lockpos and currentHrp then
        cp = currentHrp.Position -- Save current position on enabling lock
        if not lockPosConnection then
            lockPosConnection = RunService.Heartbeat:Connect(function()
                -- Re-check lockpos and hrp inside the connection
                local hrpCheck = getHumanoidRootPart(player.Character)
                if lockpos and hrpCheck and cp then
                    hrpCheck.CFrame = CFrame.new(cp)
                    -- Setting velocity might interfere with game physics, use cautiously
                    -- hrpCheck.Velocity = Vector3.new(0, 0, 0)
                    -- hrpCheck.RotVelocity = Vector3.new(0, 0, 0)
                elseif not lockpos and lockPosConnection then
                    -- Disconnect if lockpos becomes false
                    lockPosConnection:Disconnect()
                    lockPosConnection = nil
                    cp = nil -- Clear saved position
                end
            end)
        end
    elseif not lockpos then
        if lockPosConnection then
            lockPosConnection:Disconnect()
            lockPosConnection = nil
        end
        cp = nil -- Clear saved position
    end
end)

----------------------------------------------------------------
-- Proteins Tab
local Proteins = window:AddTab("Proteins")
local autoEatEnabled = false
local autoEatEquipThread = nil
local autoEatClickThread = nil

Proteins:AddSwitch("Autoeat Proteins", function(bool)
    autoEatEnabled = bool

    if autoEatEnabled then
        -- Start Equip Thread if not running
        if not autoEatEquipThread then
            autoEatEquipThread = task.spawn(function()
                local snacks = {
                    "TOUGH Bar", "Protein Bar", "Tropical Shake", "Energy Shake", "Energy Bar"
                }
                while autoEatEnabled do
                    local char = player.Character
                    local backpack = player:FindFirstChild("Backpack")
                    if not char or not backpack then
                        warn("AutoEat: Character or Backpack not found.")
                        task.wait(1) -- Wait before retrying
                        goto continueEquipLoop -- Skip to next iteration
                    end

                    local equippedTool = char:FindFirstChildOfClass("Tool")
                    if not equippedTool or not table.find(snacks, equippedTool.Name) then
                        -- Equip next available snack if nothing valid is equipped
                        for _, snackName in ipairs(snacks) do
                            local tool = backpack:FindFirstChild(snackName)
                            if tool then
                                tool.Parent = char -- Equip the tool
                                task.wait(0.2) -- Wait a bit after equipping
                                break -- Stop looking once one is equipped
                            end
                        end
                    end
                    task.wait(0.5) -- Check periodically
                    ::continueEquipLoop::
                end
                autoEatEquipThread = nil -- Clear thread variable when done
            end)
        end

        -- Start Click Thread if not running
        if not autoEatClickThread then
            autoEatClickThread = task.spawn(function()
                while autoEatEnabled do
                    -- Simulate mouse click (coordinates don't matter much for tool activation)
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
                    task.wait() -- Minimum yield
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
                    task.wait(0.1) -- Short delay between clicks
                end
                autoEatClickThread = nil -- Clear thread variable when done
            end)
        end
    else
        -- Setting autoEatEnabled to false will stop the loops in the threads.
        -- Threads will clear themselves when they exit.
    end
end)


----------------------------------------------------------------
-- Fast Glitch Tab (first instance - Rocks)
local fastglitch = window:AddTab("Fast Glitch Rocks") -- Renamed for clarity
local RockSection = fastglitch:CreateSection("Fast Glitch Rocks")
local selectrock = ""
getgenv().autoFarmRock = false -- Use a specific variable
local autoFarmRockThread = nil

local function farmRock(neededDurability, rockName)
    if autoFarmRockThread then return end -- Prevent multiple threads

    autoFarmRockThread = task.spawn(function()
        local machinesFolder = Workspace:FindFirstChild("machinesFolder")
        if not machinesFolder then
            warn("Fast Glitch Rocks: machinesFolder not found.")
            getgenv().autoFarmRock = false -- Stop if folder missing
            autoFarmRockThread = nil
            return
        end

        while getgenv().autoFarmRock and task.wait() do
            local char = player.Character
            local durabilityStat = player:FindFirstChild("Durability") -- Assuming it's directly under player
            local leftHand = char and char:FindFirstChild("LeftHand")
            local rightHand = char and char:FindFirstChild("RightHand")

            if not durabilityStat or not durabilityStat:IsA("IntValue") then -- Check type
                warn("Fast Glitch Rocks: Durability stat not found or not IntValue.")
                getgenv().autoFarmRock = false; break
            end
            if not leftHand or not rightHand then
                -- warn("Fast Glitch Rocks: Hands not found.") -- Can be spammy
                task.wait(0.5) -- Wait if hands are missing (respawn?)
                goto continueFarmLoop
            end

            if durabilityStat.Value >= neededDurability then
                local foundTargetRock = false
                for _, v in ipairs(machinesFolder:GetDescendants()) do
                    -- Check if v and its parent/rock exist before accessing properties
                    if v.Name == "neededDurability" and v:IsA("IntValue") and v.Value == neededDurability then
                        local parentMachine = v.Parent
                        local rockPart = parentMachine and parentMachine:FindFirstChild("Rock")
                        if rockPart and rockPart:IsA("BasePart") then
                            -- Fire touch events
                            firetouchinterest(rockPart, rightHand, 0)
                            firetouchinterest(rockPart, rightHand, 1)
                            firetouchinterest(rockPart, leftHand, 0)
                            firetouchinterest(rockPart, leftHand, 1)
                            gettool() -- Call the (potentially dummy) gettool function
                            foundTargetRock = true
                            break -- Assume only one rock needs farming per iteration
                        end
                    end
                end
                -- if not foundTargetRock then
                --     warn("Fast Glitch Rocks: Could not find rock part for durability:", neededDurability)
                -- end
            else
                -- Optional: Add a small delay if durability is too low
                 task.wait(0.2)
            end
            ::continueFarmLoop::
        end
        autoFarmRockThread = nil -- Clear thread variable when done
    end)
end

local function createRockToggle(title, description, durability, rockName)
    RockSection:AddToggle(title, {
        Title = title,
        Description = description,
        Default = false,
        Callback = function(Value)
            if Value then
                -- Stop any existing farm before starting a new one
                if getgenv().autoFarmRock and autoFarmRockThread then
                    getgenv().autoFarmRock = false
                    task.wait(0.1) -- Give time for the old thread to potentially exit
                end
                selectrock = rockName
                getgenv().autoFarmRock = true
                farmRock(durability, rockName)
            else
                -- Only stop if this specific rock was selected
                if selectrock == rockName then
                    getgenv().autoFarmRock = false
                    selectrock = ""
                end
            end
        end
    })
end

createRockToggle("TinyIslandRock", "Farm rocks at Tiny Island", 0, "Tiny Island Rock")
createRockToggle("StarterIslandRock", "Farm rocks at Starter Island", 100, "Starter Island Rock")
createRockToggle("LegendBeachRock", "Farm rocks at Legend Beach", 5000, "Legend Beach Rock")
createRockToggle("FrostGymRock", "Farm rocks at Frost Gym", 150000, "Frost Gym Rock")
createRockToggle("MythicalGymRock", "Farm rocks at Mythical Gym", 400000, "Mythical Gym Rock")
createRockToggle("EternalGymRock", "Farm rocks at Eternal Gym", 750000, "Eternal Gym Rock")


----------------------------------------------------------------
-- Fast Glitch Tab (second instance for autopunch) - Merged/Clarified
-- NOTE: Having two tabs with the exact same name "Fast Glitch" is confusing.
-- Renaming the second one or merging functionalities is recommended.
-- Assuming this is for a different kind of "fast glitch" related to punching.
local fastglitch_punch = window:AddTab("Fast Glitch Punch") -- Renamed for clarity
local autoPunchEnabled = false
local autoPunchThread = nil

fastglitch_punch:AddSwitch("autopunch", function(bool)
    autoPunchEnabled = bool
    if autoPunchEnabled and not autoPunchThread then
        autoPunchThread = task.spawn(function()
            local muscleEvent = player:FindFirstChild("muscleEvent")
            if not muscleEvent then
                warn("Autopunch: muscleEvent not found.")
                autoPunchEnabled = false -- Stop if event missing
                autoPunchThread = nil
                return
            end

            while autoPunchEnabled do
                muscleEvent:FireServer("punch", "leftHand")
                task.wait(0.05) -- Small delay between punches
                if not autoPunchEnabled then break end -- Check again before right punch
                muscleEvent:FireServer("punch", "rightHand")
                task.wait(0.05) -- Small delay before looping
            end
            autoPunchThread = nil -- Clear thread variable when done
        end)
    elseif not autoPunchEnabled and autoPunchThread then
        -- The loop checks autoPunchEnabled, so setting it to false will stop the thread.
    end
end)

----------------------------------------------------------------
-- Anti-AFK Tab and its functionality

local antiAfkEnabled = false
local antiAfkConnections = {}
local antiAfkGui = nil
local antiAfkBillboard = nil
local antiAfkTimerThread = nil

-- Function to start Anti-AFK
local function startAntiAfk()
    if antiAfkEnabled then return end -- Already running
    antiAfkEnabled = true

    local localPlayer = Players.LocalPlayer -- Already defined globally as 'player'
    if not player then
        warn("Anti-AFK: LocalPlayer not found.")
        antiAfkEnabled = false
        return
    end

    -- Disconnect idle by simulating activity
    local idledConnection = player.Idled:Connect(function()
        -- Check if still enabled before simulating input
        if antiAfkEnabled then
            VirtualUser:CaptureController() -- May not be necessary, ClickButton2 might suffice
            VirtualUser:ClickButton2(Vector2.new())
        end
    end)
    table.insert(antiAfkConnections, {conn = idledConnection, type = "Idled"})

    -- Create GUI to display Anti-AFK status and timer
    local playerGui = player:WaitForChild("PlayerGui")
    if not playerGui then
        warn("Anti-AFK: PlayerGui not found.")
        antiAfkEnabled = false
        return -- Cannot create GUI
    end

    -- Clean up previous GUI if it exists
    local existingGui = playerGui:FindFirstChild("AntiAfkGUI")
    if existingGui then existingGui:Destroy() end

    antiAfkGui = Instance.new("ScreenGui")
    antiAfkGui.Name = "AntiAfkGUI"
    antiAfkGui.ResetOnSpawn = false -- Keep GUI across respawns
    antiAfkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling -- Or Global if needed on top

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 250, 0, 90)
    Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- Darker background
    Frame.BackgroundTransparency = 0.2
    Frame.BorderSizePixel = 0
    Frame.Parent = antiAfkGui

    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 10)
    UICorner.Parent = Frame

    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Size = UDim2.new(1, -10, 0.5, -5) -- Padding
    TitleLabel.Position = UDim2.new(0, 5, 0, 0)
    TitleLabel.Text = "ANTI-AFK BY Darkiller"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextScaled = true
    TitleLabel.Font = Enum.Font.SourceSansBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    TitleLabel.Parent = Frame

    local TimerLabel = Instance.new("TextLabel")
    TimerLabel.Name = "TimerLabel" -- Give it a name for easy access
    TimerLabel.Size = UDim2.new(1, -10, 0.5, -5) -- Padding
    TimerLabel.Position = UDim2.new(0, 5, 0.5, 0)
    TimerLabel.Text = "Time: 00:00:00"
    TimerLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
    TimerLabel.BackgroundTransparency = 1
    TimerLabel.TextScaled = true
    TimerLabel.Font = Enum.Font.SourceSans
    TimerLabel.TextXAlignment = Enum.TextXAlignment.Left
    TimerLabel.Parent = Frame

    -- Enable dragging functionality for the frame
    Frame.Active = true -- Make frame interactive
    Frame.Draggable = true

    -- Create a BillboardGui to display text above the player's head
    local head = player.Character and player.Character:FindFirstChild("Head")
    if head then
        -- Clean up previous billboard if it exists
        local existingBillboard = head:FindFirstChild("AntiAfkBillboard")
        if existingBillboard then existingBillboard:Destroy() end

        antiAfkBillboard = Instance.new("BillboardGui")
        antiAfkBillboard.Name = "AntiAfkBillboard"
        antiAfkBillboard.Adornee = head
        antiAfkBillboard.Size = UDim2.new(0, 200, 0, 50)
        antiAfkBillboard.StudsOffset = Vector3.new(0, 3, 0)
        antiAfkBillboard.AlwaysOnTop = true
        antiAfkBillboard.ResetOnSpawn = false

        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.Text = "ANTI-AFK BY Darkiller"
        textLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
        textLabel.BackgroundTransparency = 1
        textLabel.TextScaled = true
        textLabel.Font = Enum.Font.SourceSansBold
        textLabel.Parent = antiAfkBillboard
        antiAfkBillboard.Parent = head -- Parent to head
    else
        warn("Anti-AFK: Head not found for BillboardGui.")
    end

    -- Parent GUI last
    antiAfkGui.Parent = playerGui

    -- Run timer to update the timer label
    if not antiAfkTimerThread then
        antiAfkTimerThread = task.spawn(function()
            local seconds = 0
            while antiAfkEnabled do
                task.wait(1)
                seconds = seconds + 1
                local hours = math.floor(seconds / 3600)
                local minutes = math.floor((seconds % 3600) / 60)
                local secs = seconds % 60
                -- Update label only if GUI still exists
                if antiAfkGui and antiAfkGui.Parent then
                    local timerLabelInstance = antiAfkGui:FindFirstChild("Frame", true):FindFirstChild("TimerLabel")
                    if timerLabelInstance then
                        timerLabelInstance.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
                    else
                        -- Stop timer if label is gone
                        warn("Anti-AFK Timer: Label not found.")
                        break
                    end
                else
                    -- Stop timer if GUI is gone
                    warn("Anti-AFK Timer: GUI not found.")
                    break
                end
            end
            antiAfkTimerThread = nil -- Clear thread variable when done
        end)
    end
end

-- Function to stop Anti-AFK and clean up
local function stopAntiAfk()
    if not antiAfkEnabled then return end -- Already stopped
    antiAfkEnabled = false

    -- Disconnect all connections
    for i = #antiAfkConnections, 1, -1 do
        local info = antiAfkConnections[i]
        if info.conn and info.conn.Connected then -- Check if connected before disconnecting
            info.conn:Disconnect()
        end
        table.remove(antiAfkConnections, i)
    end

    -- Destroy GUI
    if antiAfkGui and antiAfkGui.Parent then
        antiAfkGui:Destroy()
    end
    antiAfkGui = nil

    -- Destroy BillboardGui
    if antiAfkBillboard and antiAfkBillboard.Parent then
        antiAfkBillboard:Destroy()
    end
    antiAfkBillboard = nil

    -- The timer thread will stop itself because antiAfkEnabled is false
end

-- Create Anti-AFK Tab in window
local antiafkTab = window:AddTab("Anti AFK") -- Use space for better readability
antiafkTab:AddSwitch("Enable Anti AFK", function(bool) -- More descriptive label
    if bool then
        startAntiAfk()
    else
        stopAntiAfk()
    end
end)

-- Ensure cleanup if the script is destroyed or player leaves
game:GetService("Players").LocalPlayerRemoving:Connect(stopAntiAfk)
script.Destroying:Connect(stopAntiAfk)

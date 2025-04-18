-- Ensure LocalPlayer and Character exist
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser") -- Added for Anti-AFK consistency
local Workspace = game:GetService("Workspace") -- Added for consistency

local player = Players.LocalPlayer
if not player then
    error("LocalPlayer not found!")
end
if not player.Character then
    player.CharacterAdded:Wait()
end

-- Wait for HumanoidRootPart to be available
local function getHumanoidRootPart(character)
    if not character then return nil end -- Added check for character validity
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        -- Use WaitForChild with a timeout to prevent infinite yields
        hrp = character:WaitForChild("HumanoidRootPart", 5)
    end
    return hrp
end

-- Dummy definition for gettool if it doesn't exist to prevent nil errors.
-- NOTE: This is a placeholder. The actual implementation should be provided elsewhere
-- or the calls to gettool() will do nothing.
if not gettool then
    function gettool()
        -- Placeholder implementation
    end
end

-- Dummy definition for firetouchinterest if it doesn't exist
-- NOTE: This function is often environment-specific (exploit injectors).
-- If it's not available, the Fast Glitch feature relying on it will not work.
if not firetouchinterest then
    function firetouchinterest(...)
        warn("firetouchinterest function is not available in this environment.")
    end
end


-- Global flag for fast rebirth
local fastRebirth = false
local fastRebirthThread = nil -- To manage the rebirth thread

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

local function isWhitelisted(p) -- Renamed parameter for clarity
    if not p then return false end
    for _, username in ipairs(whitelist) do
        if p.Name == username then
            return true
        end
    end
    return false
end

if not isWhitelisted(player) then
    -- Consider using library notification instead of error if available
    library:Notify("Access Denied", "You are not whitelisted to use this script.", 5)
    -- Optionally, destroy the window or script components
    -- window:Destroy() -- Example if library supports it
    return -- Stop script execution gracefully
end

----------------------------------------------------------------
-- Paid Tab and its functionalities
local Paid = window:AddTab("Paid")

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth then
        -- Start a new thread only if one isn't already running
        if fastRebirthThread == nil then
            fastRebirthThread = task.spawn(function()
                local c = Players.LocalPlayer -- Use local reference inside the thread

                -- Ensure necessary components exist before starting the loop
                local petsFolder = c:FindFirstChild("petsFolder")
                local ultimatesFolder = c:FindFirstChild("ultimatesFolder")
                local leaderstats = c:FindFirstChild("leaderstats")
                local muscleEvent = c:FindFirstChild("muscleEvent")
                local equipPetEvent = ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("equipPetEvent")
                local rebirthRemote = ReplicatedStorage:FindFirstChild("rEvents") and ReplicatedStorage.rEvents:FindFirstChild("rebirthRemote")

                if not petsFolder then warn("petsFolder not found.") return end
                if not leaderstats then warn("leaderstats not found.") return end
                if not muscleEvent then warn("muscleEvent not found.") return end
                if not equipPetEvent then warn("equipPetEvent remote not found.") return end
                if not rebirthRemote then warn("rebirthRemote remote not found.") return end

                local function d() -- unequip all pets
                    local currentPetsFolder = c:FindFirstChild("petsFolder") -- Re-check in case it gets removed
                    if not currentPetsFolder then return end
                    for _, folder in ipairs(currentPetsFolder:GetChildren()) do
                        if folder:IsA("Folder") then
                            for _, pet in ipairs(folder:GetChildren()) do
                                if equipPetEvent then
                                    equipPetEvent:FireServer("unequipPet", pet)
                                end
                            end
                        end
                    end
                    task.wait(.1)
                end

                local function k(l) -- equip specific pet by name
                    d() -- Unequip first
                    task.wait(.01)
                    local currentPetsFolder = c:FindFirstChild("petsFolder")
                    local uniqueFolder = currentPetsFolder and currentPetsFolder:FindFirstChild("Unique")
                    if not uniqueFolder then return end

                    for _, pet in ipairs(uniqueFolder:GetChildren()) do
                        if pet.Name == l then
                            if equipPetEvent then
                                equipPetEvent:FireServer("equipPet", pet)
                                break -- Assume only one pet with this name needs equipping
                            end
                        end
                    end
                end

                local function o(p) -- find machine
                    local machinesRoot = Workspace:FindFirstChild("machinesFolder")
                    if not machinesRoot then
                        -- Fallback search if primary folder name changed or doesn't exist
                        for _, item in ipairs(Workspace:GetChildren()) do
                            if item:IsA("Folder") and item.Name:find("machines") then
                                machinesRoot = item
                                break
                            end
                        end
                    end
                    return machinesRoot and machinesRoot:FindFirstChild(p)
                end

                local function t() -- simulate 'E' key press
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                    task.wait(.1)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
                end

                -- Main rebirth loop
                while fastRebirth and task.wait() do -- Check flag at start of each iteration
                    local currentCharacter = c.Character -- Get current character each iteration
                    local currentHrp = getHumanoidRootPart(currentCharacter)
                    local currentLeaderstats = c:FindFirstChild("leaderstats") -- Re-check leaderstats
                    local currentMuscleEvent = c:FindFirstChild("muscleEvent") -- Re-check muscleEvent

                    if not currentCharacter or not currentHrp or not currentLeaderstats or not currentMuscleEvent then
                        warn("Player character, HRP, leaderstats, or muscleEvent missing, skipping rebirth cycle.")
                        task.wait(1) -- Wait before retrying
                        continue
                    end

                    local rebirthsStat = currentLeaderstats:FindFirstChild("Rebirths")
                    local strengthStat = currentLeaderstats:FindFirstChild("Strength")

                    if not rebirthsStat or not strengthStat then
                        warn("Rebirths or Strength stat missing, skipping rebirth cycle.")
                        task.wait(1)
                        continue
                    end

                    local v = rebirthsStat.Value -- Current rebirths
                    local w = 10000 + (5000 * v) -- Target strength

                    local currentUltimatesFolder = c:FindFirstChild("ultimatesFolder")
                    local goldenRebirth = currentUltimatesFolder and currentUltimatesFolder:FindFirstChild("Golden Rebirth")
                    if goldenRebirth and goldenRebirth:IsA("IntValue") then -- Check type
                        local x = goldenRebirth.Value
                        w = math.floor(w * (1 - (x * 0.1)))
                    end

                    d() -- Unequip pets
                    task.wait(.1)
                    k("Swift Samurai") -- Equip strength pet

                    -- Gain strength
                    while fastRebirth and strengthStat.Value < w do
                        for _ = 1, 10 do
                            currentMuscleEvent:FireServer("rep")
                        end
                        task.wait() -- Yield for other processes
                        -- Re-check strengthStat in case leaderstats was reset
                        strengthStat = currentLeaderstats:FindFirstChild("Strength")
                        if not strengthStat then break end -- Exit strength loop if stat disappears
                    end
                    if not fastRebirth then break end -- Exit if disabled during strength gain

                    d() -- Unequip pets
                    task.wait(.1)
                    k("Tribal Overlord") -- Equip rebirth pet

                    local z = o("Jungle Bar Lift") -- Find rebirth machine
                    if z then
                        local interactSeat = z:FindFirstChild("interactSeat")
                        if interactSeat and interactSeat:IsA("Seat") then -- Check type
                            currentHrp.CFrame = interactSeat.CFrame * CFrame.new(0, 3, 0)
                            local humanoid = currentCharacter:FindFirstChildOfClass("Humanoid")
                            if humanoid then
                                local sitStartTime = tick()
                                repeat
                                    task.wait(.1)
                                    t() -- Press E
                                    -- Timeout check to prevent infinite loop if sitting fails
                                    if tick() - sitStartTime > 5 then
                                        warn("Failed to sit on Jungle Bar Lift seat.")
                                        break
                                    end
                                until humanoid.Sit or not fastRebirth -- Check flag here too
                            end
                        else
                            warn("Jungle Bar Lift interactSeat not found or not a Seat.")
                        end
                    else
                        warn("Jungle Bar Lift machine not found.")
                    end
                    if not fastRebirth then break end -- Exit if disabled before rebirth attempt

                    local A = rebirthsStat.Value -- Rebirths before attempting
                    local rebirthAttemptStart = tick()
                    repeat
                        if rebirthRemote then
                            rebirthRemote:InvokeServer("rebirthRequest")
                        end
                        task.wait(.1)
                        -- Re-check rebirthsStat
                        rebirthsStat = currentLeaderstats:FindFirstChild("Rebirths")
                        if not rebirthsStat then
                            warn("Rebirths stat missing during rebirth attempt.")
                            break -- Exit rebirth loop
                        end
                        -- Timeout check
                        if tick() - rebirthAttemptStart > 10 then
                            warn("Rebirth attempt timed out.")
                            break
                        end
                    until not fastRebirth or rebirthsStat.Value > A -- Check flag and success

                    task.wait() -- Short pause before next cycle
                end
                fastRebirthThread = nil -- Clear the thread variable when done or stopped
            end)
        end
    else
        -- If the switch is turned off, the loop condition `while fastRebirth do` will handle stopping.
        -- No need to explicitly kill the thread here unless immediate stop is required,
        -- but task.cancel is unreliable. Letting the loop finish its current cycle is safer.
        -- If immediate stop was critical, you'd need more complex thread management.
        if fastRebirthThread then
             -- Optional: Signal the thread to stop if it checks another flag, but relying on `fastRebirth` is simpler.
             -- task.cancel(fastRebirthThread) -- Avoid task.cancel if possible
        end
    end
end)

-- Fast Strength Section
local fastStrengthEnabled = false
local fastStrengthThread = nil
_G.RepOP = false -- Initialize global flag state (though using local is preferred)

Paid:AddSwitch("Fast Strength", function(Value)
    _G.RepOP = Value -- Still using global as per original code, but track locally too
    fastStrengthEnabled = Value

    if fastStrengthEnabled then
        if fastStrengthThread == nil then
            fastStrengthThread = task.spawn(function()
                local c = Players.LocalPlayer
                local muscleEvent = c:FindFirstChild("muscleEvent")

                if not muscleEvent then
                    warn("Fast Strength: muscleEvent not found.")
                    fastStrengthThread = nil
                    return
                end

                while _G.RepOP do -- Use the global flag as per original logic
                    for _ = 1, 30 do -- Corrected loop syntax
                        muscleEvent:FireServer("rep")
                    end
                    task.wait(0.0001) -- Small delay
                end
                fastStrengthThread = nil -- Clear thread variable when done
            end)
        end
    else
        -- Loop will stop based on _G.RepOP flag
        if fastStrengthThread then
            -- Optional: Signal stop if needed, but loop condition handles it.
        end
    end
end)

-- Hide Frame Section
local hideFramesEnabled = false
Paid:AddSwitch("Hide Frame", function(bool)
    hideFramesEnabled = bool
    -- Ensure ReplicatedStorage exists
    if not ReplicatedStorage then
        warn("Hide Frame: ReplicatedStorage not found.")
        return
    end
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        -- Use FindFirstChild to avoid errors if frames don't exist
        local frame = ReplicatedStorage:FindFirstChild(frameName)
        if frame and frame:IsA("GuiObject") then -- Check type
            frame.Visible = not hideFramesEnabled
        end
    end
end)

----------------------------------------------------------------
-- Position and Teleport Tab
local PositionAndTeleport = window:AddTab("Position and Teleport")
local hrp = getHumanoidRootPart(player.Character) -- Get initial HRP
local lockpos = false
local cp = nil -- Current position to lock to
local lockPosConnection = nil

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool
    hrp = getHumanoidRootPart(player.Character) -- Update HRP reference when toggling

    if lockpos then
        if hrp then
            cp = hrp.Position -- Save current position only if HRP exists
            if not lockPosConnection then -- Connect only once
                 lockPosConnection = RunService.Heartbeat:Connect(function()
                    -- Check lockpos and hrp validity inside the loop
                    if lockpos then
                        local currentCharacter = player.Character
                        local currentHrp = getHumanoidRootPart(currentCharacter)
                        if currentHrp and cp then
                            -- Use CFrame for position and orientation lock (more stable)
                            currentHrp.CFrame = CFrame.new(cp)
                            -- Zero out velocity to prevent physics drift
                            currentHrp.Velocity = Vector3.new(0, 0, 0)
                            currentHrp.RotVelocity = Vector3.new(0, 0, 0)
                        else
                            -- If HRP is lost, disable lock temporarily or stop connection
                            -- lockpos = false -- Option 1: Disable toggle
                            -- cp = nil
                            -- Or just wait for HRP to return
                        end
                    end
                end)
            end
        else
            warn("Lock Position: HumanoidRootPart not found. Cannot enable lock.")
            lockpos = false -- Ensure state reflects reality
            -- Optionally, update the switch state in the UI if the library supports it
        end
    else
        -- Disconnect when turned off
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
        -- Start Equip Thread
        if autoEatEquipThread == nil then
            autoEatEquipThread = task.spawn(function()
                local snacks = {
                    "TOUGH Bar", "Protein Bar", "Tropical Shake", "Energy Shake", "Energy Bar"
                }
                while autoEatEnabled do
                    local currentCharacter = player.Character
                    local backpack = player:FindFirstChild("Backpack")

                    if not currentCharacter or not backpack then
                        warn("AutoEat: Character or Backpack not found, pausing equip.")
                        task.wait(1)
                        goto continueEquipLoop -- Skip to end of loop iteration
                    end

                    local equippedSomething = false
                    for _, snackName in ipairs(snacks) do
                        local tool = backpack:FindFirstChild(snackName)
                        if tool and tool:IsA("Tool") then -- Check if it's a tool
                            -- Check if already equipped (or a tool with same name is)
                            if not currentCharacter:FindFirstChild(snackName) then
                                tool.Parent = currentCharacter
                                equippedSomething = true
                                task.wait(0.2) -- Wait a bit after equipping
                                break -- Equip one snack at a time per cycle
                            end
                        end
                    end

                    -- If nothing was equipped this cycle, wait longer
                    if not equippedSomething then
                        task.wait(1)
                    end

                    ::continueEquipLoop::
                    -- Small delay even if something was equipped
                    task.wait(0.1)
                end
                autoEatEquipThread = nil
            end)
        end

        -- Start Click Thread
        if autoEatClickThread == nil then
            autoEatClickThread = task.spawn(function()
                while autoEatEnabled do
                    -- Check if a tool is currently equipped
                    local currentCharacter = player.Character
                    local currentTool = currentCharacter and currentCharacter:FindFirstChildOfClass("Tool")

                    if currentTool then
                        -- Simulate mouse click only if a tool is equipped
                        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1) -- Button1Down at (0,0)
                        task.wait(0.05) -- Short delay between down and up
                        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1) -- Button1Up
                        task.wait(0.1) -- Delay after clicking
                    else
                        -- Wait longer if no tool is equipped
                        task.wait(0.5)
                    end
                end
                autoEatClickThread = nil
            end)
        end
    else
        -- Threads will stop automatically because autoEatEnabled is false
        if autoEatEquipThread then
            -- task.cancel(autoEatEquipThread) -- Avoid if possible
        end
        if autoEatClickThread then
            -- task.cancel(autoEatClickThread) -- Avoid if possible
        end
    end
end)


----------------------------------------------------------------
-- Fast Glitch Tab (Combined Rocks and Autopunch)
local fastglitch = window:AddTab("Fast Glitch") -- Only one tab named this

-- Variables for Fast Glitch Rocks
local autoFarmRocksEnabled = false
local selectedRock = { name = "", durability = 0 }
local farmRockThread = nil
getgenv().autoFarm = false -- Keep global for compatibility if needed, but manage locally

-- Variables for Autopunch
local autoPunchEnabled = false
local autoPunchThread = nil

-- Combined Switch (Example - could be separate switches)
-- For simplicity, let's keep them separate as in the original, but on the same tab.

-- Autopunch Section within Fast Glitch Tab
fastglitch:AddSwitch("Autopunch", function(bool)
    autoPunchEnabled = bool
    if autoPunchEnabled then
        if autoPunchThread == nil then
            autoPunchThread = task.spawn(function()
                local c = Players.LocalPlayer
                local muscleEvent = c:FindFirstChild("muscleEvent")

                if not muscleEvent then
                    warn("Autopunch: muscleEvent not found.")
                    autoPunchThread = nil
                    return
                end

                while autoPunchEnabled do
                    muscleEvent:FireServer("punch", "leftHand")
                    task.wait(0.05) -- Small delay between punches
                    if not autoPunchEnabled then break end -- Check again before second punch
                    muscleEvent:FireServer("punch", "rightHand")
                    task.wait(0.1) -- Delay after both punches
                end
                autoPunchThread = nil
            end)
        end
    else
        -- Loop will stop based on autoPunchEnabled flag
        if autoPunchThread then
            -- Optional: Signal stop if needed
        end
    end
end)


-- Fast Glitch Rocks Section
local RockSection = fastglitch:CreateSection("Fast Glitch Rocks")

-- Master switch for Rock Farming
RockSection:AddToggle("Enable Rock Farming", { -- Changed from AddSwitch to AddToggle if library supports it, or use AddSwitch
    Title = "Enable Rock Farming",
    Description = "Master switch for rock farming",
    Default = false,
    Callback = function(Value)
        autoFarmRocksEnabled = Value
        getgenv().autoFarm = Value -- Update global flag too

        if autoFarmRocksEnabled then
            -- Start the farming thread if not already running and a rock is selected
            if farmRockThread == nil and selectedRock.name ~= "" then
                farmRockThread = task.spawn(function()
                    local c = Players.LocalPlayer

                    while getgenv().autoFarm and selectedRock.name ~= "" do -- Use global flag and check selection
                        local currentCharacter = c.Character
                        local durabilityStat = c:FindFirstChild("Durability") -- Check if Durability is directly under Player
                        local machinesFolder = Workspace:FindFirstChild("machinesFolder")

                        -- Validate prerequisites each loop iteration
                        if not currentCharacter or not currentCharacter:FindFirstChild("LeftHand") or not currentCharacter:FindFirstChild("RightHand") then
                            warn("Rock Farm: Character or hands not found. Pausing.")
                            task.wait(1)
                            goto continueFarmLoop
                        end
                        if not durabilityStat or not durabilityStat:IsA("IntValue") then -- Check type
                            warn("Rock Farm: Durability stat not found under Player or is not an IntValue. Pausing.")
                            task.wait(1)
                            goto continueFarmLoop
                        end
                         if not machinesFolder then
                            warn("Rock Farm: machinesFolder not found in Workspace. Pausing.")
                            task.wait(1)
                            goto continueFarmLoop
                        end

                        -- Check durability requirement
                        if durabilityStat.Value >= selectedRock.durability then
                            local foundTargetRock = false
                            for _, v in ipairs(machinesFolder:GetDescendants()) do
                                -- Assuming the structure is MachineModel -> RockPart and MachineModel -> neededDurability ValueInstance
                                if v.Name == "neededDurability" and v:IsA("IntValue") and v.Value == selectedRock.durability then
                                    local rockPart = v.Parent:FindFirstChild("Rock") -- Assuming the rock part is named "Rock"
                                    local rightHand = currentCharacter:FindFirstChild("RightHand")
                                    local leftHand = currentCharacter:FindFirstChild("LeftHand")

                                    if rockPart and rightHand and leftHand then
                                        -- Check if firetouchinterest exists before calling
                                        if firetouchinterest then
                                            firetouchinterest(rockPart, rightHand, 0)
                                            firetouchinterest(rockPart, rightHand, 1)
                                            firetouchinterest(rockPart, leftHand, 0)
                                            firetouchinterest(rockPart, leftHand, 1)
                                        else
                                            warn("Rock Farm: firetouchinterest not available.")
                                            -- Stop farming if the required function is missing
                                            getgenv().autoFarm = false
                                            autoFarmRocksEnabled = false
                                            -- Optionally update UI toggle state
                                            break
                                        end

                                        -- Call gettool if needed (ensure it's defined and functional)
                                        if gettool then gettool() end

                                        foundTargetRock = true
                                        task.wait(0.1) -- Add a small delay after interacting
                                        break -- Stop searching once the correct rock is found and interacted with
                                    end
                                end
                            end
                            -- If the specific rock wasn't found after checking all descendants
                            -- if not foundTargetRock then
                            --     warn("Rock Farm: Could not find rock matching durability:", selectedRock.durability)
                            --     task.wait(1) -- Wait before retrying search
                            -- end
                        else
                            -- Wait if durability is not met
                            task.wait(0.5)
                        end

                        ::continueFarmLoop::
                        task.wait() -- Yield at the end of each loop iteration
                    end
                    farmRockThread = nil -- Clear thread variable when loop finishes
                end)
            end
        else
            -- Stop the farming thread (loop condition getgenv().autoFarm will become false)
            if farmRockThread then
                -- Optional: Signal stop if needed
            end
        end
    end
})

-- Function to handle rock selection toggles
local function createRockToggle(config)
    RockSection:AddToggle(config.Name, {
        Title = config.Title,
        Description = config.Description,
        Default = false,
        Callback = function(Value)
            if Value then
                -- Deselect other rocks if library doesn't handle radio buttons
                -- (Requires library support or manual management of toggle states)
                selectedRock = { name = config.RockName, durability = config.Durability }

                -- If master switch is already on, potentially restart the farm thread
                if autoFarmRocksEnabled then
                    getgenv().autoFarm = true -- Ensure global flag is set
                    -- If thread exists, it might pick up the new selectedRock, or restart it:
                    if farmRockThread then
                       -- Let the existing thread finish its loop and pick up the new rock,
                       -- or manage thread cancellation/restart if immediate change is needed.
                       -- For simplicity, we assume the loop checks selectedRock.
                    else
                       -- If thread wasn't running (e.g., master switch was on but no rock selected before), start it.
                       -- Trigger the master switch callback again to potentially start the thread
                       -- This depends heavily on how the UI library handles callbacks.
                       -- A safer approach might be to directly call the thread starting logic here
                       -- if autoFarmRocksEnabled is true.
                       if autoFarmRocksEnabled then
                           -- Manually trigger the start logic from the master switch callback
                           local masterSwitchCallback = RockSection.Toggles["Enable Rock Farming"].Callback -- Pseudo-code access
                           if masterSwitchCallback then masterSwitchCallback(true) end
                       end
                    end
                end
            else
                -- If this rock is deselected, check if it was the active one
                if selectedRock.name == config.RockName then
                    selectedRock = { name = "", durability = 0 }
                    -- Stop the farm thread by setting the flag (it will exit on next check)
                    getgenv().autoFarm = false
                    -- Note: This doesn't automatically turn off the master switch.
                end
            end
        end
    })
end

-- Define rock configurations
local rocks = {
    { Name = "TinyIslandRockToggle", Title = "Fast Glitch Tiny Rock", Description = "Farm rocks at Tiny Island", RockName = "Tiny Island Rock", Durability = 0 },
    { Name = "StarterIslandRockToggle", Title = "Fast Glitch Starter Island Rock", Description = "Farm rocks at Starter Island", RockName = "Starter Island Rock", Durability = 100 },
    { Name = "LegendBeachRockToggle", Title = "Fast Glitch Legend Beach Rock", Description = "Farm rocks at Legend Beach", RockName = "Legend Beach Rock", Durability = 5000 },
    { Name = "FrostGymRockToggle", Title = "Fast Glitch Frost Rock", Description = "Farm rocks at Frost Gym", RockName = "Frost Gym Rock", Durability = 150000 },
    { Name = "MythicalGymRockToggle", Title = "Fast Glitch Mythical Rock", Description = "Farm rocks at Mythical Gym", RockName = "Mythical Gym Rock", Durability = 400000 },
    { Name = "EternalGymRockToggle", Title = "Fast Glitch Eternal Rock", Description = "Farm rocks at Eternal Gym", RockName = "Eternal Gym Rock", Durability = 750000 },
}

-- Create toggles for each rock
for _, rockConfig in ipairs(rocks) do
    createRockToggle(rockConfig)
end


----------------------------------------------------------------
-- Anti-AFK Tab and its functionality

-- Variables and connection holders for Anti-AFK
local antiAfkEnabled = false
local antiAfkConnections = {}
local antiAfkGui = nil -- Reference to the GUI
local antiAfkBillboardGui = nil -- Reference to BillboardGui
local antiAfkTimerThread = nil

-- Function to start Anti-AFK
local function startAntiAfk()
    if antiAfkEnabled then return end -- Already running
    antiAfkEnabled = true

    local localPlayer = Players.LocalPlayer -- Re-get just in case
    if not localPlayer then
        warn("Anti-AFK: LocalPlayer not found.")
        antiAfkEnabled = false
        return
    end

    local playerGui = localPlayer:WaitForChild("PlayerGui") -- Ensure PlayerGui exists

    -- Disconnect idle by simulating activity
    local idledConnection = localPlayer.Idled:Connect(function()
        if antiAfkEnabled then -- Check if still enabled
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new()) -- Simulate right-click
        end
    end)
    table.insert(antiAfkConnections, {conn = idledConnection, type = "Idled"})

    -- Create GUI to display Anti-AFK status and timer (only if it doesn't exist)
    if not antiAfkGui or not antiAfkGui.Parent then
        antiAfkGui = Instance.new("ScreenGui")
        antiAfkGui.Name = "AntiAfkGUI"
        antiAfkGui.ResetOnSpawn = false -- Keep GUI on respawn
        antiAfkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling -- Or Global if needed

        local Frame = Instance.new("Frame")
        Frame.Size = UDim2.new(0, 250, 0, 90)
        Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
        Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- Darker background
        Frame.BorderColor3 = Color3.fromRGB(255, 182, 193)
        Frame.BorderSizePixel = 2
        Frame.Active = true -- Enable input processing
        Frame.Draggable = true -- Make frame draggable
        Frame.Parent = antiAfkGui

        local UICorner = Instance.new("UICorner")
        UICorner.CornerRadius = UDim.new(0, 10) -- Slightly smaller radius
        UICorner.Parent = Frame

        local TitleLabel = Instance.new("TextLabel")
        TitleLabel.Size = UDim2.new(1, 0, 0.5, 0)
        TitleLabel.Position = UDim2.new(0, 0, 0, 0)
        TitleLabel.Text = "ANTI-AFK BY Darkiller"
        TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
        TitleLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60) -- Slightly different bg for title
        TitleLabel.BackgroundTransparency = 0
        TitleLabel.TextScaled = true
        TitleLabel.Font = Enum.Font.SourceSansSemibold
        TitleLabel.Parent = Frame
        local TitleCorner = Instance.new("UICorner")
        TitleCorner.CornerRadius = UDim.new(0, 10)
        TitleCorner.Parent = TitleLabel

        local TimerLabel = Instance.new("TextLabel")
        TimerLabel.Name = "TimerLabel" -- Give it a name for easy access
        TimerLabel.Size = UDim2.new(1, 0, 0.5, 0)
        TimerLabel.Position = UDim2.new(0, 0, 0.5, 0)
        TimerLabel.Text = "Time: 00:00:00"
        TimerLabel.TextColor3 = Color3.fromRGB(220, 220, 220) -- Lighter text color
        TimerLabel.BackgroundTransparency = 1
        TimerLabel.TextScaled = true
        TimerLabel.Font = Enum.Font.SourceSans
        TimerLabel.Parent = Frame

        antiAfkGui.Parent = playerGui -- Parent the GUI at the end
    end
    antiAfkGui.Enabled = true -- Ensure GUI is visible

    -- Create BillboardGui (only if it doesn't exist)
    local head = localPlayer.Character and localPlayer.Character:FindFirstChild("Head")
    if head and (not antiAfkBillboardGui or not antiAfkBillboardGui.Parent) then
        antiAfkBillboardGui = Instance.new("BillboardGui")
        antiAfkBillboardGui.Name = "AntiAfkBillboard"
        antiAfkBillboardGui.Adornee = head
        antiAfkBillboardGui.Size = UDim2.new(0, 200, 0, 50)
        antiAfkBillboardGui.StudsOffset = Vector3.new(0, 3, 0)
        antiAfkBillboardGui.ResetOnSpawn = false -- Keep on respawn
        antiAfkBillboardGui.AlwaysOnTop = true

        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.Text = "ANTI-AFK BY Darkiller"
        textLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
        textLabel.BackgroundTransparency = 1
        textLabel.TextScaled = true
        textLabel.Font = Enum.Font.SourceSansBold
        textLabel.Parent = antiAfkBillboardGui

        antiAfkBillboardGui.Parent = head -- Parent to head
    end
    if antiAfkBillboardGui then antiAfkBillboardGui.Enabled = true end

    -- Update Billboard Adornee if character respawns
    local charAddedConn = localPlayer.CharacterAdded:Connect(function(character)
        if antiAfkEnabled and antiAfkBillboardGui then
            local newHead = character:WaitForChild("Head", 5)
            if newHead then
                antiAfkBillboardGui.Adornee = newHead
                if antiAfkBillboardGui.Parent ~= newHead then -- Ensure parenting if lost
                    antiAfkBillboardGui.Parent = newHead
                end
                antiAfkBillboardGui.Enabled = true
            end
        end
    end)
    table.insert(antiAfkConnections, {conn = charAddedConn, type = "CharacterAdded"})


    -- Run timer to update the timer label (only if not already running)
    if antiAfkTimerThread == nil then
        antiAfkTimerThread = task.spawn(function()
            local seconds = 0
            while antiAfkEnabled do
                task.wait(1)
                seconds = seconds + 1
                local hours = math.floor(seconds / 3600)
                local minutes = math.floor((seconds % 3600) / 60)
                local secs = seconds % 60
                -- Update GUI timer label safely
                if antiAfkGui and antiAfkGui.Parent then
                    local timerLabel = antiAfkGui:FindFirstChild("Frame") and antiAfkGui.Frame:FindFirstChild("TimerLabel")
                    if timerLabel then
                        timerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
                    end
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

    -- Hide or destroy GUI elements
    if antiAfkGui then
        antiAfkGui.Enabled = false
        -- Optionally destroy: antiAfkGui:Destroy()
        -- antiAfkGui = nil
    end
    if antiAfkBillboardGui then
        antiAfkBillboardGui.Enabled = false
        -- Optionally destroy: antiAfkBillboardGui:Destroy()
        -- antiAfkBillboardGui = nil
    end

    -- The timer thread will stop naturally on its next check of antiAfkEnabled
    if antiAfkTimerThread then
        -- task.cancel(antiAfkTimerThread) -- Avoid if possible
    end
end

-- Create Anti-AFK Tab in window
local antiafkTab = window:AddTab("Anti-AFK") -- Changed name slightly for clarity
antiafkTab:AddSwitch("Enable Anti-AFK", function(bool) -- More descriptive title
    if bool then
        startAntiAfk()
    else
        stopAntiAfk()
    end
end)

-- Cleanup function if the script is destroyed or player leaves
game:GetService("Players").LocalPlayer.Destroying:Connect(stopAntiAfk)

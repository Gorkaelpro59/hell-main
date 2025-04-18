-- Ensure LocalPlayer and Character exist
local Players = game:GetService("Players")
local player = Players.LocalPlayer
if not player then
    -- Wait for the player object if it's not immediately available
    Players.PlayerAdded:Wait()
    player = Players.LocalPlayer
    if not player then
        error("LocalPlayer could not be found!")
    end
end

if not player.Character then
    player.CharacterAdded:Wait()
end
local character = player.Character -- Store character reference

-- Wait for HumanoidRootPart to be available
local function getHumanoidRootPart(char)
    if not char then return nil end -- Add check if character is nil
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then
        -- Use a timeout for WaitForChild to prevent infinite yields
        hrp = char:WaitForChild("HumanoidRootPart", 15)
    end
    return hrp
end

-- Dummy definition for gettool if it doesn't exist to prevent nil errors.
-- Ensure gettool is defined in the global environment or replace this dummy
if typeof(gettool) ~= "function" then
    gettool = function()
        -- This is a placeholder. The actual implementation should be provided elsewhere.
        -- warn("gettool() is not defined, using placeholder.")
    end
end

-- Global flag for fast rebirth
local fastRebirth = false

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
    -- Consider using library notification instead of error if available
    library:Notify("Access Denied", "You are not whitelisted to use this script.", 5)
    error("You are not whitelisted to use this script.")
end

----------------------------------------------------------------
-- Paid Tab and its functionalities
local Paid = window:AddTab("Paid")

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth then
        task.spawn(function() -- Use task.spawn for better performance/scheduling
            local ReplicatedStorage = game:GetService("ReplicatedStorage")
            local PlayersService = game:GetService("Players") -- Use different name to avoid conflict
            local c = PlayersService.LocalPlayer -- Use local player reference

            -- Ensure character exists within the loop's context
            if not c or not c.Character then
                warn("Player or Character not found for Fast Rebirth.")
                fastRebirth = false -- Stop the loop if player/char is lost
                return
            end

            local petsFolder = c:FindFirstChild("petsFolder")
            if not petsFolder then
                warn("petsFolder not found.")
                fastRebirth = false -- Stop if essential folder is missing
                return
            end

            local function d() -- unequip all pets
                local currentPetsFolder = c:FindFirstChild("petsFolder") -- Re-check in case it changes
                if not currentPetsFolder then return end
                for _, folder in pairs(currentPetsFolder:GetChildren()) do
                    if folder:IsA("Folder") then
                        for _, pet in pairs(folder:GetChildren()) do
                            if pet and ReplicatedStorage and ReplicatedStorage:FindFirstChild("rEvents") then
                                ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                            end
                        end
                    end
                end
                task.wait(.1)
            end

            local function k(l) -- equip specific pet by name
                d() -- Ensure others are unequipped first
                task.wait(.01)
                local currentPetsFolder = c:FindFirstChild("petsFolder")
                local uniqueFolder = currentPetsFolder and currentPetsFolder:FindFirstChild("Unique")
                if not uniqueFolder then return end

                for _, pet in pairs(uniqueFolder:GetChildren()) do
                    if pet and pet.Name == l then
                        if ReplicatedStorage and ReplicatedStorage:FindFirstChild("rEvents") then
                            ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
                        end
                        break -- Assume only one pet with this name needs equipping
                    end
                end
            end

            local function o(p) -- find machine by name
                local machinesFolder = workspace:FindFirstChild("machinesFolder")
                local q = machinesFolder and machinesFolder:FindFirstChild(p)
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

            local function t() -- simulate 'E' key press
                local VirtualInputManager = game:GetService("VirtualInputManager")
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                task.wait(.1)
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
            end

            while fastRebirth and c and c.Parent do -- Loop condition includes player existence check
                -- Check character validity each iteration
                if not c.Character or not c.Character:FindFirstChild("HumanoidRootPart") then
                    warn("Character or HRP lost during Fast Rebirth, waiting...")
                    c.CharacterAdded:Wait() -- Wait for respawn
                    task.wait(1) -- Allow time for character setup
                    if not fastRebirth then break end -- Check flag again after waiting
                    character = c.Character -- Update local character reference if needed
                end

                local leaderstats = c:FindFirstChild("leaderstats")
                local rebirthsStat = leaderstats and leaderstats:FindFirstChild("Rebirths")
                local strengthStat = leaderstats and leaderstats:FindFirstChild("Strength")
                local ultimatesFolder = c:FindFirstChild("ultimatesFolder")
                local muscleEvent = c:FindFirstChild("muscleEvent")
                local rEvents = ReplicatedStorage:FindFirstChild("rEvents")

                -- Check essential components exist before proceeding
                if not leaderstats or not rebirthsStat or not strengthStat or not muscleEvent or not rEvents then
                    warn("Missing essential components (leaderstats, stats, muscleEvent, rEvents) for Fast Rebirth.")
                    task.wait(1)
                    if not fastRebirth then break end
                    next -- Skip this iteration if components are missing
                end

                local v = rebirthsStat.Value or 0
                local w = 10000 + (5000 * v) -- Calculate required strength

                local goldenRebirth = ultimatesFolder and ultimatesFolder:FindFirstChild("Golden Rebirth")
                if goldenRebirth and goldenRebirth:IsA("IntValue") then -- Check type
                    local x = goldenRebirth.Value
                    w = math.floor(w * (1 - (x * 0.1)))
                end

                d() -- Unequip pets
                task.wait(.1)
                k("Swift Samurai") -- Equip strength pet

                -- Strength grinding loop
                while fastRebirth and strengthStat and strengthStat.Value < w do
                    for _ = 1, 10 do
                        if muscleEvent then muscleEvent:FireServer("rep") end
                    end
                    task.wait() -- Yield
                end
                if not fastRebirth then break end -- Check flag after inner loop

                d() -- Unequip pets
                task.wait(.1)
                k("Tribal Overlord") -- Equip durability pet (assuming)

                local z = o("Jungle Bar Lift") -- Find the machine
                local charHRP = c.Character and c.Character:FindFirstChild("HumanoidRootPart")
                local charHumanoid = c.Character and c.Character:FindFirstChildOfClass("Humanoid")

                if z and charHRP and charHumanoid then
                    local interactSeat = z:FindFirstChild("interactSeat")
                    if interactSeat and interactSeat:IsA("Seat") then -- Check type
                        charHRP.CFrame = interactSeat.CFrame * CFrame.new(0, 3, 0)
                        local sitStartTime = tick()
                        repeat
                            task.wait(.1)
                            t() -- Interact
                            -- Timeout check to prevent getting stuck
                            if tick() - sitStartTime > 5 then
                                warn("Failed to sit on Jungle Bar Lift within 5 seconds.")
                                break
                            end
                        until not fastRebirth or charHumanoid.Sit
                    end
                end
                if not fastRebirth then break end -- Check flag

                local A = rebirthsStat.Value -- Store rebirths before attempting
                local rebirthRemote = rEvents and rEvents:FindFirstChild("rebirthRemote")
                if rebirthRemote and rebirthRemote:IsA("RemoteEvent") then -- Check type
                    local rebirthStartTime = tick()
                    repeat
                        rebirthRemote:InvokeServer("rebirthRequest")
                        task.wait(.1)
                        -- Re-fetch rebirthsStat value
                        local currentRebirths = (c:FindFirstChild("leaderstats") and c.leaderstats:FindFirstChild("Rebirths"))
                        if not currentRebirths then break end -- Exit if stat disappears
                        -- Timeout check
                        if tick() - rebirthStartTime > 10 then
                            warn("Rebirth request timed out after 10 seconds.")
                            break
                        end
                    until not fastRebirth or (currentRebirths and currentRebirths.Value > A)
                else
                    warn("rebirthRemote not found or not a RemoteEvent.")
                end

                task.wait() -- Yield before next cycle
            end
            -- Cleanup if loop exited
            d() -- Unequip pets when stopping
            fastRebirth = false -- Ensure flag is false on exit
            print("Fast Rebirth stopped.")
        end)
    else
        -- If the switch is turned off, fastRebirth flag is set to false,
        -- the running task.spawn loop will detect it and exit.
        print("Stopping Fast Rebirth...")
    end
end)

local switch = Paid:AddSwitch("Fast Strength", function(Value)
    _G.RepOP = Value -- Using _G is generally discouraged, consider alternatives if possible
    if not Value then return end

    local function FastRep()
        local localPlayer = game:GetService("Players").LocalPlayer -- Get player reference inside task
        while _G.RepOP do
            if not localPlayer or not localPlayer.Parent then break end -- Stop if player leaves
            local muscleEvent = localPlayer:FindFirstChild("muscleEvent")
            if muscleEvent then
                for _ = 1, 30 do
                    muscleEvent:FireServer("rep")
                end
            else
                warn("muscleEvent not found for Fast Strength.")
                _G.RepOP = false -- Stop if event is missing
                break
            end
            task.wait(0.0001) -- Very short delay, potentially high CPU usage
        end
        print("Fast Strength stopped.")
    end
    task.spawn(FastRep)
end)


local switchHideFrame = Paid:AddSwitch("Hide Frame", function(bool)
    local replicatedStorage = game:GetService("ReplicatedStorage")
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        -- Use WaitForChild with timeout for potentially non-existent frames
        local frame = replicatedStorage:WaitForChild(frameName, 1)
        if frame and frame:IsA("GuiObject") then
            frame.Visible = not bool
        end
    end
end)

----------------------------------------------------------------
-- Position and Teleport Tab
local PositionAndTeleport = window:AddTab("Position and Teleport")
local hrp = getHumanoidRootPart(character) -- Get initial HRP
local lockpos = false
local cp
local heartbeatConnection = nil -- Store connection to disconnect later

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool  -- Toggle lockpos state
    if lockpos then
        hrp = getHumanoidRootPart(player.Character) -- Re-get HRP in case character respawned
        if hrp then
            cp = hrp.Position  -- Save current position on enabling lock
            if not heartbeatConnection or not heartbeatConnection.Connected then -- Connect only if not already connected
                heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function()
                    if lockpos then -- Check flag inside connection
                        local currentHrp = getHumanoidRootPart(player.Character) -- Check HRP validity each frame
                        if currentHrp and cp then
                            currentHrp.CFrame = CFrame.new(cp)
                            -- Setting velocity might interfere with physics, use CFrame primarily
                            currentHrp.Velocity = Vector3.new(0, 0, 0)
                            currentHrp.RotVelocity = Vector3.new(0, 0, 0)
                        else
                            -- If HRP is lost (e.g., death), stop locking temporarily
                            -- lockpos = false -- Optionally disable the lock automatically
                            -- print("HRP lost, pausing position lock.")
                        end
                    else
                        -- Disconnect if lockpos becomes false
                        if heartbeatConnection and heartbeatConnection.Connected then
                            heartbeatConnection:Disconnect()
                            heartbeatConnection = nil
                        end
                    end
                end)
            end
        else
            warn("Cannot lock position: HumanoidRootPart not found.")
            lockpos = false -- Ensure lockpos is false if HRP isn't available
        end
    else
        -- Disconnect when toggled off
        if heartbeatConnection and heartbeatConnection.Connected then
            heartbeatConnection:Disconnect()
            heartbeatConnection = nil
        end
    end
end)


----------------------------------------------------------------
-- Proteins Tab
local Proteins = window:AddTab("Proteins")
local autoEatEnabled = false
local eatLoopThread = nil
local clickLoopThread = nil

Proteins:AddSwitch("Autoeat Proteins", function(bool)
    autoEatEnabled = bool

    if autoEatEnabled then
        -- Start Equip Loop
        if not eatLoopThread or coroutine.status(eatLoopThread) == "dead" then
            eatLoopThread = task.spawn(function()
                local snacks = {
                    "TOUGH Bar", "Protein Bar", "Tropical Shake", "Energy Shake", "Energy Bar"
                }
                while autoEatEnabled do
                    local currentCharacter = player.Character -- Get current character each iteration
                    local backpack = player:FindFirstChild("Backpack")
                    if not currentCharacter or not backpack then
                        warn("Character or Backpack not found for AutoEat.")
                        task.wait(1)
                        if not autoEatEnabled then break end
                        next
                    end

                    local toolEquipped = false
                    for _, toolInstance in pairs(currentCharacter:GetChildren()) do
                        if toolInstance:IsA("Tool") and table.find(snacks, toolInstance.Name) then
                            toolEquipped = true
                            break
                        end
                    end

                    if not toolEquipped then -- Only equip if no snack is currently held
                        for _, snackName in ipairs(snacks) do
                            local tool = backpack:FindFirstChild(snackName)
                            if tool then
                                tool.Parent = currentCharacter -- Equip the tool
                                task.wait(0.1) -- Short delay after equipping
                                break -- Equip only one tool per cycle
                            end
                        end
                    end
                    task.wait(0.5) -- Check/equip interval
                end
                print("AutoEat Equip Loop stopped.")
            end)
        end

        -- Start Click Loop
        if not clickLoopThread or coroutine.status(clickLoopThread) == "dead" then
            clickLoopThread = task.spawn(function()
                local VirtualInputManager = game:GetService("VirtualInputManager")
                while autoEatEnabled do
                    -- Check if a snack tool is currently equipped
                    local currentCharacter = player.Character
                    local toolEquipped = false
                    if currentCharacter then
                         local snacks = {
                            "TOUGH Bar", "Protein Bar", "Tropical Shake", "Energy Shake", "Energy Bar"
                         }
                         for _, toolInstance in pairs(currentCharacter:GetChildren()) do
                            if toolInstance:IsA("Tool") and table.find(snacks, toolInstance.Name) then
                                toolEquipped = true
                                break
                            end
                        end
                    end

                    if toolEquipped then
                       VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1) -- Send mouse down at (0,0) - position often doesn't matter
                       task.wait(0.05) -- Short hold duration
                       VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1) -- Send mouse up
                       task.wait(0.1) -- Delay between clicks
                    else
                       task.wait(0.2) -- Wait longer if no tool is equipped
                    end
                end
                print("AutoEat Click Loop stopped.")
            end)
        end
    else
        -- Stop loops by setting the flag (they will exit on their next check)
        print("Stopping AutoEat...")
    end
end)


----------------------------------------------------------------
-- Fast Glitch Tab (first instance - Rocks)
local fastglitch = window:AddTab("Fast Glitch")
local RockSection = fastglitch:CreateSection("Fast Glitch Rocks")
local selectrock = ""
getgenv().autoFarm = false -- Initialize to false
local farmThread = nil

-- Define firetouchinterest if it's not globally available (common in exploit environments)
if typeof(firetouchinterest) ~= "function" then
    firetouchinterest = function(part1, part2, touchType)
        -- Placeholder: Actual implementation depends on the exploit executor
        -- warn("firetouchinterest is not defined.")
        game:GetService("RunService").Stepped:Wait() -- Simulate some delay
    end
end


local function farmRock(neededDurability, rockName)
    if farmThread and coroutine.status(farmThread) ~= "dead" then
        -- Optionally warn or just ignore if already farming
        -- print("Already farming, request ignored.")
        return
    end

    farmThread = task.spawn(function()
        print("Starting farm for:", rockName, "Requires:", neededDurability)
        while getgenv().autoFarm and selectrock == rockName do
            local localPlayer = game:GetService("Players").LocalPlayer
            if not localPlayer or not localPlayer.Parent then
                warn("Player left, stopping farm.")
                getgenv().autoFarm = false
                break
            end

            local durabilityStat = localPlayer:FindFirstChild("Durability") -- Find stat safely
            local currentCharacter = localPlayer.Character
            local leftHand = currentCharacter and currentCharacter:FindFirstChild("LeftHand")
            local rightHand = currentCharacter and currentCharacter:FindFirstChild("RightHand")

            if not durabilityStat or not currentCharacter or not leftHand or not rightHand then
                warn("Missing components (Durability, Character, Hands) for farming.")
                task.wait(1)
                if not getgenv().autoFarm then break end
                next -- Skip iteration if components missing
            end

            if durabilityStat.Value >= neededDurability then
                local targetRockPart = nil
                local machinesFolder = workspace:FindFirstChild("machinesFolder")

                -- Search for the specific rock interaction part
                if machinesFolder then
                    for _, v in pairs(machinesFolder:GetDescendants()) do
                        -- Assuming the structure is MachineModel -> RockPart and MachineModel -> neededDurability ValueInstance
                        if v.Name == "neededDurability" and v:IsA("IntValue") and v.Value == neededDurability then
                            local rockPart = v.Parent:FindFirstChild("Rock") -- Adjust "Rock" if the part name is different
                            if rockPart and rockPart:IsA("BasePart") then
                                targetRockPart = rockPart
                                break -- Found the correct rock
                            end
                        end
                    end
                end

                if targetRockPart then
                    -- Fire touch events
                    firetouchinterest(targetRockPart, rightHand, 0)
                    firetouchinterest(targetRockPart, rightHand, 1)
                    firetouchinterest(targetRockPart, leftHand, 0)
                    firetouchinterest(targetRockPart, leftHand, 1)
                    -- Call gettool if necessary (ensure it's defined and does what's needed)
                    gettool()
                else
                     -- warn("Could not find rock part for durability:", neededDurability)
                     -- Optionally stop farming if the rock can't be found after a while
                end
            else
                 -- Wait longer if durability is not met
                 task.wait(0.5)
            end
            task.wait() -- Yield even if farming
        end
        print("Stopped farming:", rockName)
        -- Ensure flag is false if loop exited naturally or due to error
        if selectrock == rockName then
             getgenv().autoFarm = false
             selectrock = ""
        end
        farmThread = nil -- Clear the thread variable
    end)
end

-- Helper function to manage toggles
local function createRockToggle(config)
    RockSection:AddToggle(config.Name, {
        Title = config.Title,
        Description = config.Description,
        Default = false,
        Callback = function(Value)
            if Value then
                -- If another rock is selected, disable its farming first
                if getgenv().autoFarm and selectrock ~= "" and selectrock ~= config.RockName then
                     print("Switching farm target, stopping previous farm.")
                     getgenv().autoFarm = false -- Stop previous farm loop
                     task.wait(0.1) -- Give time for the old loop to potentially exit
                end
                selectrock = config.RockName
                getgenv().autoFarm = true
                farmRock(config.Durability, config.RockName)
            else
                -- Only stop if this specific rock was being farmed
                if selectrock == config.RockName then
                    getgenv().autoFarm = false
                    selectrock = ""
                end
            end
        end
    })
end

-- Create toggles using the helper
createRockToggle({ Name = "TinyIslandRock", Title = "Fast Glitch Tiny Rock", Description = "Farm rocks at Tiny Island", Durability = 0, RockName = "Tiny Island Rock" })
createRockToggle({ Name = "StarterIslandRock", Title = "Fast Glitch Starter Island Rock", Description = "Farm rocks at Starter Island", Durability = 100, RockName = "Starter Island Rock" })
createRockToggle({ Name = "LegendBeachRock", Title = "Fast Glitch Legend Beach Rock", Description = "Farm rocks at Legend Beach", Durability = 5000, RockName = "Legend Beach Rock" })
createRockToggle({ Name = "FrostGymRock", Title = "Fast Glitch Frost Rock", Description = "Farm rocks at Frost Gym", Durability = 150000, RockName = "Frost Gym Rock" })
createRockToggle({ Name = "MythicalGymRock", Title = "Fast Glitch Mythical Rock", Description = "Farm rocks at Mythical Gym", Durability = 400000, RockName = "Mythical Gym Rock" })
createRockToggle({ Name = "EternalGymRock", Title = "Fast Glitch Eternal Rock", Description = "Farm rocks at Eternal Gym", Durability = 750000, RockName = "Eternal Gym Rock" })


----------------------------------------------------------------
-- Fast Glitch Tab (second instance for autopunch) - Naming conflict, rename tab or merge
-- Renaming the second tab instance to avoid conflicts
local AutoPunchTab = window:AddTab("Auto Punch") -- Changed tab name
local autoPunchEnabled = false
local punchThread = nil

AutoPunchTab:AddSwitch("autopunch", function(bool) -- Changed tab reference
    autoPunchEnabled = bool
    if autoPunchEnabled then
        if not punchThread or coroutine.status(punchThread) == "dead" then
            punchThread = task.spawn(function()
                print("Auto Punch started.")
                while autoPunchEnabled do
                    local localPlayer = game:GetService("Players").LocalPlayer
                    if not localPlayer or not localPlayer.Parent then
                        warn("Player left, stopping Auto Punch.")
                        autoPunchEnabled = false
                        break
                    end

                    local muscleEvent = localPlayer:FindFirstChild("muscleEvent")
                    if muscleEvent then
                        -- Perform punch actions
                        muscleEvent:FireServer("punch", "leftHand")
                        task.wait(0.05) -- Small delay between punches
                        if not autoPunchEnabled then break end -- Check flag again
                        muscleEvent:FireServer("punch", "rightHand")
                    else
                        warn("muscleEvent not found for Auto Punch.")
                        autoPunchEnabled = false -- Stop if event is missing
                        break
                    end
                    task.wait(0.1) -- Delay between punch pairs
                end
                print("Auto Punch stopped.")
                punchThread = nil -- Clear thread variable
            end)
        end
    else
        -- Flag is set to false, the loop will terminate on its next check.
        print("Stopping Auto Punch...")
    end
end)


----------------------------------------------------------------
-- Anti-AFK Tab and its functionality

-- Variables and connection holders for Anti-AFK
local antiAfkEnabled = false
local antiAfkConnections = {}
local antiAfkGui = nil -- Reference to the GUI
local antiAfkTimerThread = nil

-- Function to start Anti-AFK
local function startAntiAfk()
    if antiAfkEnabled then return end -- Prevent starting multiple times
    antiAfkEnabled = true
    print("Starting Anti-AFK...")

    local VirtualUser = game:GetService("VirtualUser")
    local UIS = game:GetService("UserInputService")
    local localPlayer = Players.LocalPlayer -- Use the global player variable

    if not localPlayer then
        warn("LocalPlayer not found for Anti-AFK")
        antiAfkEnabled = false
        return
    end

    -- Ensure PlayerGui exists
    local playerGui = localPlayer:WaitForChild("PlayerGui", 10)
    if not playerGui then
        warn("PlayerGui not found for Anti-AFK GUI.")
        -- Continue without GUI if needed, but log warning
    end

    -- Disconnect idle by simulating activity
    local idledConnection = localPlayer.Idled:Connect(function()
        if antiAfkEnabled then -- Check flag before acting
            print("Anti-AFK: Player Idled, simulating input.")
            VirtualUser:CaptureController() -- May not be necessary/available in all environments
            VirtualUser:ClickButton2(Vector2.new()) -- Simulate right-click
        end
    end)
    table.insert(antiAfkConnections, {conn = idledConnection, type = "Idled"})

    -- Create GUI only if PlayerGui is available
    if playerGui and not antiAfkGui then
        antiAfkGui = Instance.new("ScreenGui")
        antiAfkGui.Name = "AntiAfkGUI"
        antiAfkGui.ResetOnSpawn = false -- Keep GUI across respawns
        antiAfkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling -- Or Global if needed

        local Frame = Instance.new("Frame")
        Frame.Name = "AntiAfkFrame"
        Frame.Size = UDim2.new(0, 250, 0, 90)
        Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
        Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- Darker background
        Frame.BorderColor3 = Color3.fromRGB(255, 182, 193)
        Frame.BorderSizePixel = 2
        Frame.Active = true -- Enable input processing
        Frame.Draggable = true -- Make frame draggable

        local UICorner = Instance.new("UICorner")
        UICorner.CornerRadius = UDim.new(0, 10)
        UICorner.Parent = Frame

        local TitleLabel = Instance.new("TextLabel")
        TitleLabel.Name = "Title"
        TitleLabel.Size = UDim2.new(1, 0, 0.5, 0)
        TitleLabel.Position = UDim2.new(0, 0, 0, 0)
        TitleLabel.Text = "ANTI-AFK BY Darkiller"
        TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
        TitleLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        TitleLabel.BackgroundTransparency = 0
        TitleLabel.TextScaled = true
        TitleLabel.Font = Enum.Font.SourceSansBold
        TitleLabel.Parent = Frame
        local TitleCorner = Instance.new("UICorner")
        TitleCorner.CornerRadius = UDim.new(0, 10)
        TitleCorner.Parent = TitleLabel


        local TimerLabel = Instance.new("TextLabel")
        TimerLabel.Name = "TimerLabel"
        TimerLabel.Size = UDim2.new(1, 0, 0.5, 0)
        TimerLabel.Position = UDim2.new(0, 0, 0.5, 0)
        TimerLabel.Text = "Time: 00:00:00"
        TimerLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
        TimerLabel.BackgroundTransparency = 1
        TimerLabel.TextScaled = true
        TimerLabel.Font = Enum.Font.SourceSansBold
        TimerLabel.Parent = Frame

        Frame.Parent = antiAfkGui
        antiAfkGui.Parent = playerGui -- Parent the GUI at the end
    end

    -- Create/Update BillboardGui
    local function setupBillboard()
        local currentCharacter = localPlayer.Character
        local head = currentCharacter and currentCharacter:FindFirstChild("Head")
        if not head then return end -- Exit if no head

        local billboardGui = head:FindFirstChild("AntiAfkBillboard")
        if not billboardGui then
            billboardGui = Instance.new("BillboardGui")
            billboardGui.Name = "AntiAfkBillboard"
            billboardGui.Adornee = head
            billboardGui.Size = UDim2.new(0, 200, 0, 50)
            billboardGui.StudsOffset = Vector3.new(0, 3, 0)
            billboardGui.AlwaysOnTop = true
            billboardGui.Enabled = antiAfkEnabled -- Initial state

            local textLabel = Instance.new("TextLabel")
            textLabel.Name = "StatusLabel"
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.Text = "ANTI-AFK BY Darkiller"
            textLabel.TextColor3 = Color3.fromRGB(255, 182, 193)
            textLabel.BackgroundTransparency = 1
            textLabel.TextScaled = true
            textLabel.Font = Enum.Font.SourceSansBold
            textLabel.Parent = billboardGui
            billboardGui.Parent = head -- Parent to head
        end
        billboardGui.Enabled = antiAfkEnabled -- Ensure it's enabled if anti-afk is on
    end

    setupBillboard() -- Initial setup
    -- Re-setup billboard on character added
    local charAddedConn = localPlayer.CharacterAdded:Connect(function(newChar)
        task.wait(0.5) -- Wait for head to potentially exist
        if antiAfkEnabled then setupBillboard() end
    end)
    table.insert(antiAfkConnections, {conn = charAddedConn, type = "CharacterAdded"})


    -- Run timer to update the timer label
    if not antiAfkTimerThread or coroutine.status(antiAfkTimerThread) == "dead" then
        antiAfkTimerThread = task.spawn(function()
            local seconds = 0
            while antiAfkEnabled do
                task.wait(1)
                seconds = seconds + 1
                if antiAfkGui then -- Check if GUI exists before updating
                    local timerLabel = antiAfkGui:FindFirstChildDeep("TimerLabel")
                    if timerLabel and timerLabel:IsA("TextLabel") then
                        local hours = math.floor(seconds / 3600)
                        local minutes = math.floor((seconds % 3600) / 60)
                        local secs = seconds % 60
                        timerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
                    end
                end
            end
            print("Anti-AFK Timer thread stopped.")
            antiAfkTimerThread = nil
        end)
    end
end

-- Function to stop Anti-AFK and clean up
local function stopAntiAfk()
    if not antiAfkEnabled then return end
    antiAfkEnabled = false -- Set flag first to stop loops/events
    print("Stopping Anti-AFK...")

    -- Disconnect all stored connections
    for i = #antiAfkConnections, 1, -1 do
        local info = antiAfkConnections[i]
        if info.conn and info.conn.Connected then
            info.conn:Disconnect()
        end
        table.remove(antiAfkConnections, i)
    end

    -- Destroy the GUI
    if antiAfkGui then
        antiAfkGui:Destroy()
        antiAfkGui = nil
    end

    -- Disable/Destroy BillboardGui
    local currentCharacter = player and player.Character
    local head = currentCharacter and currentCharacter:FindFirstChild("Head")
    if head then
        local billboardGui = head:FindFirstChild("AntiAfkBillboard")
        if billboardGui then
            billboardGui.Enabled = false -- Disable it
            -- Optionally destroy: billboardGui:Destroy()
        end
    end
    -- The timer thread will stop automatically because antiAfkEnabled is false
end

-- Create Anti-AFK Tab in window
local antiafkTab = window:AddTab("AntiAFK") -- Changed name slightly for clarity
antiafkTab:AddSwitch("Enable AntiAFK", function(bool) -- Changed label for clarity
    if bool then
        startAntiAfk()
    else
        stopAntiAfk()
    end
end)

-- Ensure cleanup on script removal/player leaving
game:GetService("Players").LocalPlayer.CharacterRemoving:Connect(function()
    -- Optional: Stop certain features on death if needed, e.g., position lock
    if lockpos then
        if heartbeatConnection and heartbeatConnection.Connected then
            heartbeatConnection:Disconnect()
            heartbeatConnection = nil
        end
        -- Don't necessarily set lockpos = false, let the switch control it
    end
end)

-- Add a final cleanup mechanism if the script environment supports it
-- or connect to player removing if necessary.
local playerRemovingConn = player.AncestryChanged:Connect(function(_, parent)
    if not parent then -- Player is removed from Players service
        print("Player removing, cleaning up Anti-AFK and other loops...")
        stopAntiAfk()
        fastRebirth = false
        _G.RepOP = false
        getgenv().autoFarm = false
        autoEatEnabled = false
        autoPunchEnabled = false
        if playerRemovingConn then playerRemovingConn:Disconnect() end -- Disconnect self
    end
end)

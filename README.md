--[[ 
         H3ll paid Script

        Updates:
        - Improved error handling, automatic pet equipping, strength farming.
        - Fixed pet equip issue.
        ]]


        local Players                = game:GetService("Players")
        local RunService             = game:GetService("RunService")
        local ReplicatedStorage      = game:GetService("ReplicatedStorage")
        local VirtualInputManager    = game:GetService("VirtualInputManager")
        local UserInputService       = game:GetService("UserInputService")
        local VirtualUser            = game:GetService("VirtualUser") 

        local player = Players.LocalPlayer
        if not player then
                error("LocalPlayer not found!")
        end
        if not player.Character then
                player.CharacterAdded:Wait()
        end

        local function getHumanoidRootPart(character)
                if not character then return nil end 
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if not hrp then
                        hrp = character:WaitForChild("HumanoidRootPart", 10)
                end
                return hrp
        end

        if not gettool then
                gettool = function()

                end
        end

        local fastRebirth            = false -- // set to true it wont rebirth you trust me
        getgenv().autoFarm           = false 
        _G.RepOP                     = false 

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
                error("Error during library loading: " .. tostring(library)) 
        elseif not library then
                error("The loaded library is nil.")
        end
        
        if typeof(library) ~= "table" or not library.AddWindow then
                error("Loaded library is not valid or missing AddWindow method.")
        end
        local window = library:AddWindow("HELL clan Script by darkiller | Fixed by Vyxon", {
                main_color = Color3.fromRGB(0, 0, 0),
                min_size = Vector2.new(400, 300),
                can_resize = false
        })
        if not window or not window.AddTab then
                error("Failed to create window or window is missing AddTab method.")
        end
        -- // fixed by Vyxon
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
                if library and library.Notify then
                        library:Notify("Not Whitelisted", "You are not whitelisted to use this script.", 5)
                else
                        warn("You are not whitelisted to use this script.")
                end
                return 
        else
                local PaidTab = window:AddTab("Paid")
                local fastRebirth = false
                local fastRebirthThread = nil
                local autoEatEnabled = false
                local autoEatEquipThread = nil
                local autoEatClickThread = nil
                local lockpos = false
                local lockPosConnection = nil
                local ReplicatedStorage = game:GetService("ReplicatedStorage")
                local Players = game:GetService("Players")
                local VirtualInputManager = game:GetService("VirtualInputManager")
                local RunService = game:GetService("RunService")
                local player = Players.LocalPlayer

                PaidTab:AddSwitch("Fast Rebirth", function(Value)
                        fastRebirth = Value
                        if fastRebirth then
                        if fastRebirthThread and coroutine.status(fastRebirthThread) ~= "dead" then
                                task.cancel(fastRebirthThread)
                        end

                        fastRebirthThread = task.spawn(function()
                                local rEvents = ReplicatedStorage:FindFirstChild("rEvents")
                                if not rEvents then
                                warn("rEvents not found in ReplicatedStorage.")
                                fastRebirth = false 
                                return
                                end
                                local equipPetEvent = rEvents:FindFirstChild("equipPetEvent")
                                local rebirthRemote = rEvents:FindFirstChild("rebirthRemote")
                                if not equipPetEvent or not rebirthRemote then
                                warn("Required remote events/functions (equipPetEvent, rebirthRemote) not found under rEvents.")
                                fastRebirth = false 
                                return
                                end

                                local c = Players.LocalPlayer

                                local function unequipPets() 
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
                                task.wait(0.1)
                                end

                                local function equipSpecificPet(petName) 
                                if not c or not c:FindFirstChild("petsFolder") then return end
                                unequipPets() 
                                task.wait(0.01)
                                local uniqueFolder = c.petsFolder:FindFirstChild("Unique")
                                if not uniqueFolder then return end
                                for _, pet in pairs(uniqueFolder:GetChildren()) do
                                        if pet and pet.Name == petName and equipPetEvent then
                                        equipPetEvent:FireServer("equipPet", pet)
                                        break
                                        end
                                end
                                end

                                local function findMachine(petName) 
                                local machinesFolder = workspace:FindFirstChild("machinesFolder")
                                if not machinesFolder then
                                        for _, s in pairs(workspace:GetChildren()) do
                                        if s:IsA("Folder") and s.Name:find("machines") then
                                                machinesFolder = s
                                                break
                                        end
                                        end
                                end
                                if not machinesFolder then return nil end
                                return machinesFolder:FindFirstChild(petName)
                                end

                                local function pressInteractKey() 
                                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                                task.wait(0.1)
                                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
                                end

                                while fastRebirth and task.wait() do
                                if not c or not c.Character or not c.Character:FindFirstChild("Humanoid") then
                                        warn("Player, Character or Humanoid not found during fast rebirth loop.")
                                        task.wait(1) 
                                        c = Players.LocalPlayer
                                        if not c then fastRebirth = false; break end 
                                        if not c.Character then player.CharacterAdded:Wait() end 
                                        if not c.Character then fastRebirth = false; break end 
                                        return
                                end

                                local leaderstats = c:FindFirstChild("leaderstats")
                                local rebirthsStat = leaderstats and leaderstats:FindFirstChild("Rebirths")
                                local strengthStat = leaderstats and leaderstats:FindFirstChild("Strength")
                                local muscleEvent = c:FindFirstChild("muscleEvent")

                                if not leaderstats or not rebirthsStat or not strengthStat or not muscleEvent then
                                        warn("Missing leaderstats, Rebirths, Strength, or muscleEvent.")
                                        task.wait(1)
                                        return 
                                end

                                local v = rebirthsStat.Value 
                                local w = 10000 + (5000 * v)

                                local ultimatesFolder = c:FindFirstChild("ultimatesFolder")
                                local goldenRebirth = ultimatesFolder and ultimatesFolder:FindFirstChild("Golden Rebirth")
                                if goldenRebirth and goldenRebirth:IsA("IntValue") then 
                                        local x = goldenRebirth.Value
                                        w = math.floor(w * (1 - (x * 0.1)))
                                end

                                unequipPets() 
                                task.wait(0.1)
                                equipSpecificPet("Swift Samurai") 

                                while fastRebirth and strengthStat.Value < w do
                                        for _ = 1, 10 do
                                        muscleEvent:FireServer("rep")
                                        end
                                        task.wait() 
                                end
                                if not fastRebirth then break end 

                                unequipPets() 
                                task.wait(0.1)
                                equipSpecificPet("Tribal Overlord") 

                                local z = findMachine("Jungle Bar Lift") 
                                local hrp = getHumanoidRootPart(c.Character)
                                if z and hrp then
                                        local interactSeat = z:FindFirstChild("interactSeat")
                                        if interactSeat and interactSeat:IsA("Seat") then 
                                        hrp.CFrame = interactSeat.CFrame * CFrame.new(0, 3, 0)
                                        local sitStartTime = tick()
                                        repeat
                                                task.wait(0.1)
                                                pressInteractKey() 
                                        until not fastRebirth or (c.Character and c.Character:FindFirstChildOfClass("Humanoid") and c.Character.Humanoid.Sit) or (tick() - sitStartTime > 5)
                                        if not fastRebirth then break end 
                                        if not (c.Character and c.Character:FindFirstChildOfClass("Humanoid") and c.Character.Humanoid.Sit) then
                                                warn("Failed to sit on Jungle Bar Lift.")
                                        end
                                        else
                                        warn("Jungle Bar Lift interactSeat not found or not a Seat.")
                                        end
                                else
                                        warn("Jungle Bar Lift machine or HumanoidRootPart not found.")
                                end

                                local A = rebirthsStat.Value 
                                local rebirthStartTime = tick()
                                repeat
                                        if rebirthRemote then
                                        rebirthRemote:InvokeServer("rebirthRequest")
                                        end
                                        task.wait(0.1)

                                        leaderstats = c:FindFirstChild("leaderstats")
                                        rebirthsStat = leaderstats and leaderstats:FindFirstChild("Rebirths")

                                until not fastRebirth or not rebirthsStat or rebirthsStat.Value > A or (tick() - rebirthStartTime > 10)
                                if not fastRebirth then break end 
                                if not rebirthsStat or rebirthsStat.Value <= A then
                                        warn("Rebirth failed or timed out.")
                                end
                                task.wait() 
                                end
                        end)
                        elseif fastRebirthThread and coroutine.status(fastRebirthThread) ~= "dead" then
                        task.cancel(fastRebirthThread)
                        fastRebirthThread = nil
                        end
                end)

                PaidTab:AddSwitch("Fast Strength", function(Value)
                        _G.RepOP = Value
                        if Value then
                        if fastStrengthThread and coroutine.status(fastStrengthThread) ~= "dead" then
                                task.cancel(fastStrengthThread)
                        end

                        fastStrengthThread = task.spawn(function()
                                local c = Players.LocalPlayer
                                while _G.RepOP do 
                                local muscleEvent = c and c:FindFirstChild("muscleEvent")
                                if muscleEvent then
                                        for _ = 1, 15.00 do
                                        muscleEvent:FireServer("rep")
                                        end
                                else
                                        warn("muscleEvent not found for Fast Strength.")
                                        _G.RepOP = false 
                                        break
                                end
                                task.wait(0.00999) 
                                end
                        end)
                        elseif fastStrengthThread and coroutine.status(fastStrengthThread) ~= "dead" then
                        task.cancel(fastStrengthThread)
                        fastStrengthThread = nil
                        end
                end)

                PaidTab:AddSwitch("Hide Frame", function(Value)
                        for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
                        local playerGui = Players.LocalPlayer:FindFirstChild("PlayerGui")
                        if playerGui then
                                local frame = playerGui:FindFirstChild(frameName, true)
                                if frame and frame:IsA("GuiObject") then
                                frame.Visible = not Value
                                end
                        end
                        end
                end)

                PaidTab:AddSwitch("Lock Position", function(Value)
                        lockpos = Value
                        local currentCharacter = Players.LocalPlayer.Character
                        local hrp = getHumanoidRootPart(currentCharacter)

                        if lockpos and hrp then
                        cp = hrp.Position  
                        if not lockPosConnection then
                                lockPosConnection = RunService.Heartbeat:Connect(function()
                                if not lockpos then
                                        if lockPosConnection then
                                        lockPosConnection:Disconnect()
                                        lockPosConnection = nil
                                        end
                                        return
                                end

                                local char = Players.LocalPlayer.Character
                                local currentHrp = getHumanoidRootPart(char)

                                if currentHrp and cp then 
                                        currentHrp.CFrame = CFrame.new(cp)
                                        currentHrp.Velocity = Vector3.new(0, 0, 0)
                                        currentHrp.RotVelocity = Vector3.new(0, 0, 0)
                                end
                                end)
                        end
                        elseif lockPosConnection then
                        lockPosConnection:Disconnect()
                        lockPosConnection = nil
                        end
                end)

                PaidTab:AddSwitch("Autoeat Proteins", function(Value)
                        autoEatEnabled = Value

                        if autoEatEnabled then
                        if autoEatEquipThread and coroutine.status(autoEatEquipThread) ~= "dead" then
                                task.cancel(autoEatEquipThread)
                        end

                        if autoEatClickThread and coroutine.status(autoEatClickThread) ~= "dead" then
                                task.cancel(autoEatClickThread)
                        end

                        autoEatEquipThread = task.spawn(function()
                                local proteinEgg = ReplicatedStorage:FindFirstChild("proteinEgg")
                                if not proteinEgg then
                                warn("proteinEgg not found in ReplicatedStorage.")
                                return
                                end

                        end)
                        elseif autoEatThread and coroutine.status(autoEatThread) ~= "dead" then
                        task.cancel(autoEatThread)
                        autoEatThread = nil
                        end
                end)

                local paidTab = window:AddTab("Auto Rocks")

                local fgEnabled = false
                paidTab:AddSwitch("Enable Fast Glitch Features", function(bool)
                        fgEnabled = bool
                        getgenv().autoFarm = bool
                        if not bool then
                                stopFarmRockThread()
                                getgenv().autoFarm = false
                        end
                end)

                local currFarmRock = nil
                local farmThread = nil

                local function stopFarmRockThread()
                        if farmThread and coroutine.status(farmThread) ~= "dead" then
                                task.cancel(farmThread)
                                farmThread = nil
                        end
                        currFarmRock = nil
                        getgenv().autoFarm = false
                end

                local function startFarmRock(durability, rockName)
                        stopFarmRockThread()
                        currFarmRock = rockName
                        getgenv().autoFarm = true

                        farmThread = task.spawn(function()
                                local mFolder = workspace:FindFirstChild("machinesFolder")
                                if not mFolder then
                                        warn("machinesFolder not found for rock farming.")
                                        getgenv().autoFarm = false
                                        return
                                end

                                local tFunc = firetouchinterest
                                if not tFunc then
                                        warn("firetouchinterest function not found. Rock farming will not work.")
                                        getgenv().autoFarm = false
                                        return
                                end

                                while getgenv().autoFarm and currFarmRock == rockName and task.wait() do
                                        local p = Players.LocalPlayer
                                        if not p or not p.Character then
                                                warn("Player or Character not found during rock farm.")
                                                task.wait(1)
                                                return
                                        end

                                        local durStat = p:FindFirstChild("Durability")
                                        if not durStat or not durStat:IsA("IntValue") then
                                                warn("Durability stat not found or not an IntValue.")
                                                task.wait(1)
                                                return
                                        end

                                        if durStat.Value >= durability then
                                                local lHand = p.Character:FindFirstChild("LeftHand")
                                                local rHand = p.Character:FindFirstChild("RightHand")

                                                if not lHand or not rHand then
                                                        warn("Player hands not found.")
                                                        task.wait(0.5)
                                                        return
                                                end

                                                local foundRock = false
                                                for _, v in pairs(mFolder:GetDescendants()) do
                                                        if v.Name == "neededDurability" and v:IsA("IntValue") and v.Value == durability then
                                                                local parentMachine = v.Parent
                                                                local rockPart = parentMachine and parentMachine:FindFirstChild("Rock")
                                                                if rockPart then
                                                                        foundRock = true

                                                                        tFunc(rockPart, rHand, 0)
                                                                        tFunc(rockPart, rHand, 1)
                                                                        tFunc(rockPart, lHand, 0)
                                                                        tFunc(rockPart, lHand, 1)

                                                                        if gettool then gettool() end
                                                                        break
                                                                end
                                                        end
                                                end
                                        else
                                                task.wait(0.5)
                                        end
                                end

                                if currFarmRock == rockName then
                                        currFarmRock = nil
                                        getgenv().autoFarm = false
                                end
                        end)
                end

                local rockConfigs = {
                        { Name = "TinyIslandRock", Title = "Fast Glitch Tiny Rock", Durability = 0 },
                        { Name = "StarterIslandRock", Title = "Fast Glitch Starter Island Rock", Durability = 100 },
                        { Name = "LegendBeachRock", Title = "Fast Glitch Legend Beach Rock", Durability = 5000 },
                        { Name = "FrostGymRock", Title = "Fast Glitch Frost Rock", Durability = 150000 },
                        { Name = "MythicalGymRock", Title = "Fast Glitch Mythical Rock", Durability = 400000 },
                        { Name = "EternalGymRock", Title = "Fast Glitch Eternal Rock", Durability = 750000 },
                }

                for _, cfg in ipairs(rockConfigs) do
                        paidTab:AddSwitch(cfg.Title, function(Value)
                                if Value then
                                        startFarmRock(cfg.Durability, cfg.Name)
                                else
                                        if currFarmRock == cfg.Name then
                                                stopFarmRockThread()
                                        end
                                end
                        end)
                end

                local apEnabled = false
                local apThread = nil

                paidTab:AddSwitch("Enable Autopunch", function(bool)
                        apEnabled = bool
                        if apEnabled then
                                if apThread and coroutine.status(apThread) ~= "dead" then
                                        task.cancel(apThread)
                                end

                                apThread = task.spawn(function()
                                        while apEnabled and task.wait() do
                                                local p = Players.LocalPlayer
                                                local muscleEvent = p and p:FindFirstChild("muscleEvent")
                                                if muscleEvent then
                                                        muscleEvent:FireServer("punch", "leftHand")
                                                        muscleEvent:FireServer("punch", "rightHand")
                                                else
                                                        warn("muscleEvent not found for autopunch.")
                                                        apEnabled = false
                                                        break
                                                end
                                        end
                                end)
                        elseif apThread and coroutine.status(apThread) ~= "dead" then
                                task.cancel(apThread)
                                apThread = nil
                        end
                end)

        end

local library = loadstring(game:HttpGet("https://pastebin.com/raw/Abg3RkND", true))()
local window = library:AddWindow("HELL clan Script by darkiller", { main_color = Color3.fromRGB(41, 74, 122), min_size = Vector2.new(600, 550), can_resize = false })

-- Whitelist system using usernames
local whitelist = {
    "Quiraz29", 
    "Shadow_RipperZ0",
    "HoloGrindz",
    "Xxdarkiller_10ZKT2",
    "TARZANfeio",
}

local player = game.Players.LocalPlayer

-- Check if the player is whitelisted
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

local Paid = window:AddTab("Paid")

Paid:AddSwitch("Fast Rebirth", function(bool)
    fastRebirth = bool
    if fastRebirth then
        spawn(function()
            local a = game:GetService("ReplicatedStorage")
            local b = game:GetService("Players")
            local c = b.LocalPlayer
            
            -- Check if petsFolder exists
            if not c:FindFirstChild("petsFolder") then
                error("petsFolder not found.")
                return
            end
            
            local d = function(e)
                local f = c.petsFolder
                for g, h in pairs(f:GetChildren()) do
                    if h:IsA("Folder") then
                        for i, j in pairs(h:GetChildren()) do
                            a.rEvents.equipPetEvent:FireServer("unequipPet", j)
                        end
                    end
                end
                task.wait(.1)
            end
            
            local k = function(l)
                d()
                task.wait(.01)
                for m, n in pairs(c.petsFolder.Unique:GetChildren()) do
                    if n.Name == l then
                        a.rEvents.equipPetEvent:FireServer("equipPet", n)
                    end
                end
            end
            
            local o = function(p)
                local q = workspace.machinesFolder:FindFirstChild(p)
                if not q then
                    for r, s in pairs(workspace:GetChildren()) do
                        if s:IsA("Folder") and s.Name:find("machines") then
                            q = s:FindFirstChild(p)
                            if q then break end
                        end
                    end
                end
                return q
            end
            
            local t = function()
                local u = game:GetService("VirtualInputManager")
                u:SendKeyEvent(true, "E", false, game)
                task.wait(.1)
                u:SendKeyEvent(false, "E", false, game)
            end
            
            while fastRebirth do
                local v = c.leaderstats and c.leaderstats.Rebirths and c.leaderstats.Rebirths.Value or 0
                local w = 10000 + (5000 * v)
                if c.ultimatesFolder and c.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                    local x = c.ultimatesFolder["Golden Rebirth"].Value
                    w = math.floor(w * (1 - (x * 0.1)))
                end
                d()
                task.wait(.1)
                k("Swift Samurai")
                while c.leaderstats and c.leaderstats.Strength and c.leaderstats.Strength.Value < w do
                    for y = 1, 10 do
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
                local A = c.leaderstats and c.leaderstats.Rebirths and c.leaderstats.Rebirths.Value or 0
                repeat
                    a.rEvents.rebirthRemote:InvokeServer("rebirthRequest")
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
            for _ = 1, 400 do
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

local Paid = window:AddTab("Position and Teleport")

Position and Teleport:AddSwitch("lockposition", function(bool)
    fastRebirth = bool
    if lockposition then
        spawn(function()
           local Players = game:GetService("Players")
           local plr = Players.LocalPlayer
           local character = plr.Character or plr.CharacterAdded:Wait()
           local hrp = character:WaitForChild("HumanoidRootPart")
           local rs = game:GetService("RunService")
           local lockpos = false
           local cp = nil

rs.Heartbeat:Connect(function()
    if lockpos then
        if not cp then
            cp = hrp.Position
        end
        hrp.CFrame = CFrame.new(cp)
    else
        cp = nil
    end
end)

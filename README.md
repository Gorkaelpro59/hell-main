local library = loadstring(game:HttpGet("https://pastebin.com/raw/Abg3RkND", true))()
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

local PositionAndTeleport = window:AddTab("Position and Teleport")

PositionAndTeleport:AddSwitch("lockposition", function(bool)
    lockpos = bool  -- Cambia el estado de lockpos según el interruptor
    if lockpos then
        cp = hrp.Position  -- Guarda la posición actual al activar el bloqueo
    end
end)

-- Conexión al evento Heartbeat para bloquear la posición
rs.Heartbeat:Connect(function()
    if lockpos then
        hrp.CFrame = CFrame.new(cp)  -- Mantiene al jugador en la posición guardada
        hrp.Velocity = Vector3.new(0, 0, 0)  -- Detiene cualquier movimiento
        hrp.RotVelocity = Vector3.new(0, 0, 0)  -- Detiene la rotación
    end
end)

local AutoPunch = window:AddTab("Autopunch")

Autopunch:AddSwitch("autopunch", function(bool)
local function performPunch()
    local player = game.Players.LocalPlayer
    player.muscleEvent:FireServer("punch", "leftHand")
    player.muscleEvent:FireServer("punch", "rightHand")
end

local PositionAndTeleport = window:AddTab("Autoeat Proteins")

Autoeat Proteins:AddSwitch("Autoeat Proteins", function(bool)
-- Create the Frame
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 300, 0, 250)
Frame.Position = UDim2.new(0.5, -150, 0.5, -125)
Frame.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

-- Create the TextLabel
local TextLabel = Instance.new("TextLabel")
TextLabel.Size = UDim2.new(1, 0, 0.2, 0)
TextLabel.Position = UDim2.new(0, 0, 0, 0)
TextLabel.BackgroundTransparency = 1
TextLabel.Text = "Auto EAT BOOSTS BY MOHA"
TextLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TextLabel.Font = Enum.Font.SourceSans
TextLabel.TextSize = 24
TextLabel.Parent = Frame

-- Create the Boost selection buttons
local Boosts = {"Protein Bar", "Ultra Shake", "Energy Bar", "Tough Bar", "Protein Shake"}
local BoostSelections = {}
for i, boost in ipairs(Boosts) do
    local BoostButton = Instance.new("TextButton")
    BoostButton.Size = UDim2.new(0.8, 0, 0.15, 0)
    BoostButton.Position = UDim2.new(0.1, 0, 0.2 + (i - 1) * 0.18, 0)
    BoostButton.BackgroundColor3 = Color3.fromRGB(255, 0, 255)
    BoostButton.Text = boost
    BoostButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    BoostButton.Font = Enum.Font.SourceSans
    BoostButton.TextSize = 18
    BoostButton.Parent = Frame
    BoostButton.MouseButton1Click:Connect(function()
        if BoostSelections[boost] then
            BoostSelections[boost] = nil
            BoostButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        else
            BoostSelections[boost] = true
            BoostButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        end
    end)
end

-- Function to auto consume selected boosts
local function autoConsumeBoosts()
    while true do
        wait(1) -- Adjust the wait time as needed
        local player = game.Players.LocalPlayer
        local backpack = player.Backpack
        for _, item in pairs(backpack:GetChildren()) do
            if BoostSelections[item.Name] then
                player.Character.Humanoid:EquipTool(item)
                item:Activate()
            end
        end
    end
end

-- Create the Start Button
local StartButton = Instance.new("TextButton")
StartButton.Size = UDim2.new(0, 100, 0, 50)
StartButton.Position = UDim2.new(1, 10, 0, 0) -- Position it to the right side of ScreenGui
StartButton.BackgroundColor3 = Color3.fromRGB(0, 128, 0)
StartButton.Text = "Start"
StartButton.TextColor3 = Color3.fromRGB(255, 255, 255)
StartButton.Font = Enum.Font.SourceSans
StartButton.TextSize = 24
StartButton.Parent = Frame
StartButton.Active = true
StartButton.Draggable = true

-- Start the auto consume when the button is clicked
StartButton.MouseButton1Click:Connect(function()
    if StartButton.Text == "Start" then
        StartButton.Text = "Running..."
        autoConsumeBoosts()
    else
        StartButton.Text = "Start"
        -- Add logic to stop the autoConsumeBoosts function if needed
    end
end)

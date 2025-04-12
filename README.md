local success, library = pcall(function()
    return loadstring(game:HttpGet("https://pastebin.com/raw/Abg3RkND", true))()
end)

if not success or not library then
    error("Failed to load library.")
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
game:GetService("RunService").Heartbeat:Connect(function()
    if lockpos then
        hrp.CFrame = CFrame.new(cp)  -- Mantiene al jugador en la posición guardada
        hrp.Velocity = Vector3.new(0, 0, 0)  -- Detiene cualquier movimiento
        hrp.RotVelocity = Vector3.new(0, 0, 0)  -- Detiene la rotación
    end
end)

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

local Afk = window:AddTab("Afk")

Afk:AddSwitch("Anti Afk", function(bool)
    local Players = game:GetService("Players")
    local VirtualUser  = game:GetService("VirtualUser ")
    local UIS = game:GetService("User InputService")

    Players.LocalPlayer.Idled:Connect(function()
        Virtual:User CaptureController()
        Virtual:User ClickButton2(Vector2.new())
    end)

    -- Create GUI
    local ScreenGui = Instance.new("ScreenGui")
    local Frame = Instance.new("Frame")
    local TitleLabel = Instance.new("TextLabel")
    local TimerLabel = Instance.new("TextLabel")
    local UICorner = Instance.new("UICorner")  -- Add UICorner to adjust corners

    ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    -- Background (White)
    Frame.Parent = ScreenGui
    Frame.Size = UDim2.new(0, 250, 0, 90)
    Frame.Position = UDim2.new(0.5, -125, 0.1, 0)
    Frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- White
    Frame.BackgroundTransparency = 0 -- Not Transparent

    -- Add UICorner to adjust rounded corners
    UICorner.Parent = Frame
    UICorner.CornerRadius = UDim.new(0, 20) -- Set corner radius

    -- Main text (Light Pink)
    TitleLabel.Parent = Frame
    TitleLabel.Size = UDim2.new(1, 0, 0.5, 0)
    TitleLabel.Position = UDim2.new(0, 0, 0, 0)
    TitleLabel.Text = "ANTI-AFK BY Darkiller"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 182, 193) -- Light Pink
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextScaled = true
    TitleLabel.Font = Enum.Font.SourceSansBold

    -- Timer text (Light Pink)
    TimerLabel.Parent = Frame
    TimerLabel.Size = UDim2.new(1, 0, 0.5, 0)
    TimerLabel.Position = UDim2.new(0, 0, 0.5, 0)
    TimerLabel.Text = "Time: 00:00:00"
    TimerLabel.TextColor3 = Color3.fromRGB(255, 182, 193) -- Light Pink
    TimerLabel.BackgroundTransparency = 1
    TimerLabel.TextScaled = true
    TimerLabel.Font = Enum.Font.SourceSansBold

    -- Dragging functionality
    local dragging
    local dragInput
    local dragStart
    local startPos

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
                if input.UserInputState == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
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

-- Create a BillboardGui to display the text in the world
local billboardGui = Instance.new("BillboardGui")
billboardGui.Parent = game.Workspace
billboardGui.Adornee = game.Players.LocalPlayer.Character:WaitForChild("Head") -- Positioning the text above the player's head
billboardGui.Size = UDim2.new(0, 200, 0, 50)
billboardGui.StudsOffset = Vector3.new(0, 3, 0) -- Adjust the height above the player

local textLabel = Instance.new("TextLabel")
textLabel.Parent = billboardGui
textLabel.Size = UDim2.new(1, 0, 1, 0)
textLabel.Text = "ANTI-AFK BY Darkiller"
textLabel.TextColor3 = Color3.fromRGB(255, 182, 193) -- Light Pink
textLabel.BackgroundTransparency = 1
textLabel.TextScaled = true
textLabel.Font = Enum.Font.SourceSansBold

-- Run timer
local seconds = 0
while true do
    wait(1)
    seconds = seconds + 1
    local hours = math.floor(seconds / 3600)
    local minutes = math.floor((seconds % 3600) / 60)
    local secs = seconds % 60
    TimerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, secs)
end

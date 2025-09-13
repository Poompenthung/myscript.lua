local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- ===============================
-- Rainbow color
-- ===============================
local hue = 0
local textHue = 0
local function rainbowColor()
    hue = (hue + 0.01) % 1
    return Color3.fromHSV(hue,1,1)
end
local function rainbowTextColor()
    textHue = (textHue + 0.01) % 1
    return Color3.fromHSV(textHue,1,1)
end

-- ===============================
-- GUI
-- ===============================
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ElevatorGUI"
ScreenGui.Parent = PlayerGui

local ButtonF = Instance.new("TextButton")
ButtonF.Size = UDim2.new(0,180,0,50)
ButtonF.Position = UDim2.new(0,20,0,100)
ButtonF.BackgroundColor3 = Color3.fromRGB(100,100,100)
ButtonF.TextColor3 = Color3.fromRGB(255,255,255)
ButtonF.Font = Enum.Font.GothamBold
ButtonF.TextScaled = true
ButtonF.Text = "Elevator: OFF ❌"
ButtonF.Parent = ScreenGui

local ButtonG = Instance.new("TextButton")
ButtonG.Size = UDim2.new(0,180,0,50)
ButtonG.Position = UDim2.new(0,20,0,160)
ButtonG.BackgroundColor3 = Color3.fromRGB(100,100,100)
ButtonG.TextColor3 = Color3.fromRGB(255,255,255)
ButtonG.Font = Enum.Font.GothamBold
ButtonG.TextScaled = true
ButtonG.Text = "Rainbow: OFF ❌"
ButtonG.Parent = ScreenGui

-- ===============================
-- Elevator (F)
-- ===============================
local elevatorEnabled = false
local elevatorPlatform = nil
local elevatorStartY = 0
local elevatorCurrentY = 0
local elevatorRiseSpeed = 20
local elevatorMaxRise = 150

local function createElevatorPlatform(hrp)
    elevatorStartY = hrp.Position.Y - 3
    elevatorCurrentY = elevatorStartY
    if elevatorPlatform then elevatorPlatform:Destroy() end
    elevatorPlatform = Instance.new("Part")
    elevatorPlatform.Size = Vector3.new(5,0.6,5)
    elevatorPlatform.Anchored = true
    elevatorPlatform.CanCollide = true
    elevatorPlatform.Material = Enum.Material.Neon
    elevatorPlatform.Color = rainbowColor()
    elevatorPlatform.Parent = workspace
    elevatorPlatform.CFrame = CFrame.new(hrp.Position.X, elevatorStartY, hrp.Position.Z)
end

local function removeElevatorPlatform()
    if elevatorPlatform then elevatorPlatform:Destroy() elevatorPlatform = nil end
end

local function toggleElevator(hrp)
    elevatorEnabled = not elevatorEnabled
    if elevatorEnabled then createElevatorPlatform(hrp) else removeElevatorPlatform() end
    if elevatorEnabled then
        ButtonF.Text = "Elevator: ON ✅"
        ButtonF.BackgroundColor3 = Color3.fromRGB(0,170,255)
    else
        ButtonF.Text = "Elevator: OFF ❌"
        ButtonF.BackgroundColor3 = Color3.fromRGB(100,100,100)
    end
end

-- ===============================
-- Rainbow Platform (G)
-- ===============================
local rainbowEnabled = false
local rainbowPlatform = nil
local rainbowStartY = 0
local rainbowCurrentY = 0
local rainbowRiseSpeed = 20
local rainbowMaxRise = 10

local function createRainbowPlatform(hrp)
    rainbowStartY = hrp.Position.Y - 3
    rainbowCurrentY = rainbowStartY
    if rainbowPlatform then rainbowPlatform:Destroy() end
    rainbowPlatform = Instance.new("Part")
    rainbowPlatform.Size = Vector3.new(5,0.6,5)
    rainbowPlatform.Anchored = true
    rainbowPlatform.CanCollide = true
    rainbowPlatform.Material = Enum.Material.Neon
    rainbowPlatform.Color = rainbowColor()
    rainbowPlatform.Parent = workspace
    rainbowPlatform.CFrame = CFrame.new(hrp.Position.X, rainbowCurrentY, hrp.Position.Z)
end

local function removeRainbowPlatform()
    if rainbowPlatform then rainbowPlatform:Destroy() rainbowPlatform = nil end
end

local function toggleRainbow(hrp)
    rainbowEnabled = not rainbowEnabled
    if rainbowEnabled then createRainbowPlatform(hrp) else removeRainbowPlatform() end
    if rainbowEnabled then
        ButtonG.Text = "Rainbow: ON ✅"
        ButtonG.BackgroundColor3 = Color3.fromRGB(0,170,255)
    else
        ButtonG.Text = "Rainbow: OFF ❌"
        ButtonG.BackgroundColor3 = Color3.fromRGB(100,100,100)
    end
end

-- ===============================
-- GUI Click
-- ===============================
ButtonF.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then toggleElevator(char.HumanoidRootPart) end
end)
ButtonG.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then toggleRainbow(char.HumanoidRootPart) end
end)

-- ===============================
-- Key F/G toggle
-- ===============================
UserInputService.InputBegan:Connect(function(input,gpe)
    if gpe then return end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    if input.KeyCode == Enum.KeyCode.F then toggleElevator(char.HumanoidRootPart)
    elseif input.KeyCode == Enum.KeyCode.G then toggleRainbow(char.HumanoidRootPart) end
end)

-- ===============================
-- Character Respawn
-- ===============================
LocalPlayer.CharacterAdded:Connect(function(char)
    local hrp = char:WaitForChild("HumanoidRootPart")
    if elevatorEnabled then createElevatorPlatform(hrp) end
    if rainbowEnabled then createRainbowPlatform(hrp) end
end)

-- ===============================
-- Update ทุกเฟรม
-- ===============================
RunService.RenderStepped:Connect(function(dt)
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Elevator
    if elevatorEnabled and elevatorPlatform then
        elevatorCurrentY = math.min(elevatorCurrentY + elevatorRiseSpeed*dt, elevatorStartY + elevatorMaxRise)
        elevatorPlatform.CFrame = CFrame.new(hrp.Position.X, elevatorCurrentY, hrp.Position.Z)
        elevatorPlatform.Color = rainbowColor()
        hrp.CFrame = CFrame.new(hrp.Position.X, elevatorCurrentY + 3, hrp.Position.Z, hrp.CFrame.LookVector.X,0,hrp.CFrame.LookVector.Z)
    end

    -- Rainbow Platform
    if rainbowEnabled and rainbowPlatform then
        -- ถ้าเราออกจาก platform > 5 studs ให้หาย
        if math.abs((hrp.Position.Y - 3) - rainbowCurrentY) > 5 then
            removeRainbowPlatform()
            rainbowEnabled = false
            ButtonG.Text = "Rainbow: OFF ❌"
            ButtonG.BackgroundColor3 = Color3.fromRGB(100,100,100)
        else
            rainbowCurrentY = math.min(rainbowCurrentY + rainbowRiseSpeed*dt, rainbowStartY + rainbowMaxRise)
            rainbowPlatform.CFrame = CFrame.new(hrp.Position.X, rainbowCurrentY, hrp.Position.Z)
            rainbowPlatform.Color = rainbowColor()
            hrp.CFrame = CFrame.new(hrp.Position.X, rainbowCurrentY + 3, hrp.Position.Z, hrp.CFrame.LookVector.X,0,hrp.CFrame.LookVector.Z)
        end
    end

    -- ไล่สีปุ่ม
    ButtonF.TextColor3 = rainbowTextColor()
    ButtonG.TextColor3 = rainbowTextColor()
end)

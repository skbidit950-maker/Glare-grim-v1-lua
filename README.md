--[[
  Glare Grim Lua v2
  Features: Orbit, Void, Void Desync
]]

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Configuration
local Config = {
    OrbitEnabled = false,
    VoidEnabled = false,
    VoidDesyncEnabled = false,
    OrbitSpeed = 8,
    OrbitRadius = 12,
    OrbitHeight = 3,
    VoidHeight = 500,
}

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GlareGrimV2"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 220)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -110)
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 8, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local FrameCorner = Instance.new("UICorner")
FrameCorner.CornerRadius = UDim.new(0, 10)
FrameCorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(20, 15, 35)
Title.BorderSizePixel = 0
Title.Text = "Glare Grim Lua v2"
Title.TextColor3 = Color3.fromRGB(180, 160, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 15
Title.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = Title

-- Close button
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Position = UDim2.new(1, -28, 0, 6)
CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
CloseBtn.BorderSizePixel = 0
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
CloseBtn.Parent = MainFrame

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Toggle factory (3 buttons, no descriptions — clean)
local function CreateToggle(name, yOffset, configKey)
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, -20, 0, 42)
    bg.Position = UDim2.new(0, 10, 0, yOffset)
    bg.BackgroundColor3 = Color3.fromRGB(18, 16, 30)
    bg.BorderSizePixel = 0
    bg.Parent = MainFrame
    
    local bgCorner = Instance.new("UICorner")
    bgCorner.CornerRadius = UDim.new(0, 8)
    bgCorner.Parent = bg
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.6, -10, 1, 0)
    label.Position = UDim2.new(0, 12, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(210, 210, 240)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = bg
    
    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 60, 0, 30)
    toggleBtn.Position = UDim2.new(1, -72, 0, 6)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 48, 65)
    toggleBtn.BorderSizePixel = 0
    toggleBtn.Text = "OFF"
    toggleBtn.TextColor3 = Color3.fromRGB(255, 110, 110)
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.TextSize = 13
    toggleBtn.Parent = bg
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = toggleBtn
    
    local enabled = false
    toggleBtn.MouseButton1Click:Connect(function()
        enabled = not enabled
        Config[configKey] = enabled
        if enabled then
            toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 175, 80)
            toggleBtn.TextColor3 = Color3.fromRGB(200, 255, 200)
            toggleBtn.Text = "ON"
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 48, 65)
            toggleBtn.TextColor3 = Color3.fromRGB(255, 110, 110)
            toggleBtn.Text = "OFF"
        end
    end)
end

-- Create exactly 3 buttons
CreateToggle("Orbit", 45, "OrbitEnabled")
CreateToggle("Void", 97, "VoidEnabled")
CreateToggle("Void Desync", 149, "VoidDesyncEnabled")

-- Status line
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 18)
StatusLabel.Position = UDim2.new(0, 10, 0, 198)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Ready"
StatusLabel.TextColor3 = Color3.fromRGB(140, 140, 180)
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 10
StatusLabel.Parent = MainFrame

-- ==================== CORE LOGIC ====================

local function getNearestEnemy()
    local nearest, nearestDist = nil, math.huge
    local myPos = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myPos then return nil end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if hrp and humanoid and humanoid.Health > 0 then
                local dist = (myPos.Position - hrp.Position).Magnitude
                if dist < nearestDist then
                    nearest = player
                    nearestDist = dist
                end
            end
        end
    end
    return nearest
end

local function getHRP()
    local char = LocalPlayer.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

-- Orbit
local orbitAngle = 0
local function doOrbit()
    if not Config.OrbitEnabled then return end
    local hrp = getHRP()
    if not hrp then return end
    local target = getNearestEnemy()
    if not target then return end
    local targetHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if not targetHRP then return end
    
    orbitAngle = orbitAngle + math.rad(Config.OrbitSpeed)
    local x = targetHRP.Position.X + math.cos(orbitAngle) * Config.OrbitRadius
    local z = targetHRP.Position.Z + math.sin(orbitAngle) * Config.OrbitRadius
    local y = targetHRP.Position.Y + Config.OrbitHeight + math.sin(orbitAngle * 2) * 1.5
    
    hrp.CFrame = CFrame.new(x, y, z) * CFrame.Angles(0, -orbitAngle, 0)
end

-- Void
local function doVoid()
    if not Config.VoidEnabled then return end
    local hrp = getHRP()
    if not hrp then return end
    
    local tween = TweenService:Create(hrp, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        CFrame = CFrame.new(hrp.Position.X, Config.VoidHeight, hrp.Position.Z)
    })
    tween:Play()
    
    task.spawn(function()
        while Config.VoidEnabled and getHRP() do
            task.wait(0.1)
            local h = getHRP()
            if h then
                h.Velocity = Vector3.new(0, 0, 0)
                h.CFrame = CFrame.new(h.Position.X, Config.VoidHeight, h.Position.Z)
            end
        end
    end)
end

-- Void Desync
local desyncConnection = nil
local desyncActive = false

local function startDesync()
    if desyncActive then return end
    desyncActive = true
    local hrp = getHRP()
    if not hrp then desyncActive = false return end
    
    hrp.CFrame = CFrame.new(hrp.Position.X, Config.VoidHeight, hrp.Position.Z)
    StatusLabel.Text = "Desync Active"
    
    desyncConnection = RunService.Stepped:Connect(function()
        if not Config.VoidDesyncEnabled or not getHRP() then
            stopDesync()
            return
        end
        local h = getHRP()
        if h then
            h.Velocity = Vector3.new(0, 5, 0)
            h.CFrame = CFrame.new(h.Position.X, Config.VoidHeight, h.Position.Z)
        end
    end)
end

local function stopDesync()
    if desyncConnection then
        desyncConnection:Disconnect()
        desyncConnection = nil
    end
    desyncActive = false
    local hrp = getHRP()
    if hrp then
        hrp.CFrame = CFrame.new(hrp.Position.X, 10, hrp.Position.Z)
    end
    StatusLabel.Text = "Ready"
end

-- FFlag desync
local fflagActive = false
local function applyFFlag()
    pcall(function() setfflag("WorldStepMax", "-9999999999999") end)
    fflagActive = true
end
local function resetFFlag()
    pcall(function() setfflag("WorldStepMax", "-1") end)
    fflagActive = false
end

-- Main loop
RunService.Stepped:Connect(function()
    pcall(function()
        doOrbit()
        
        if Config.VoidDesyncEnabled and not desyncActive then
            startDesync()
        elseif not Config.VoidDesyncEnabled and desyncActive then
            stopDesync()
        end
        
        if Config.VoidEnabled then
            doVoid()
        end
        
        if Config.VoidEnabled or Config.VoidDesyncEnabled then
            if not fflagActive then applyFFlag() end
        else
            if fflagActive then resetFFlag() end
        end
    end)
end)

-- Respawn handling
LocalPlayer.CharacterAdded:Connect(function()
    if desyncConnection then
        desyncConnection:Disconnect()
        desyncConnection = nil
    end
    desyncActive = false
    resetFFlag()
    StatusLabel.Text = "Ready"
end)

print("Glare Grim Lua v2 loaded")

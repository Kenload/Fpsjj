-- FPS BOOSTER + LOCAL T-POSE
-- Modified from RIP#6666's script
-- Enhanced by piore

-- ====== CONFIGURATION ======
local Config = {
    Notifications = true,      -- Enable/disable notifications
    ConsoleLogs = false,       -- Enable debug logs
    InitialCheckDelay = 1,     -- Delay before initial check (seconds)
    MaintenanceInterval = 5,   -- T-Pose maintenance interval (seconds)
    
    Performance = {
        FPS_Cap = 300,        -- Set to false to disable
        WaitPerAmount = 500,   -- Adjust based on performance
        InstanceCheckDelay = 0.1 -- Delay between instance checks
    },
    
    IgnoreList = {}            -- Add instances here to ignore them
}

-- ====== SERVICES ======
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local MaterialService = game:GetService("MaterialService")
local RunService = game:GetService("RunService")

-- ====== CORE FUNCTIONS ======
local function SendNotification(title, text, duration)
    if Config.Notifications then
        StarterGui:SetCore("SendNotification", {
            Title = title,
            Text = text,
            Duration = duration or 5,
            Button1 = "Okay"
        })
    end
end

local function Log(message)
    if Config.ConsoleLogs then
        print("[FPS Booster] " .. message)
    end
end

-- ====== T-POSE SYSTEM ======
local TPose = {
    Active = true,
    JointSettings = {
        RightShoulder = CFrame.new(1, 0.5, 0, 0, 0, 1, 0, 1, 0, -1, 0, 0),
        LeftShoulder = CFrame.new(-1, 0.5, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0),
        RightHip = CFrame.new(0.5, -1, 0, 0, 0, 1, 0, 1, 0, -1, 0, 0),
        LeftHip = CFrame.new(-0.5, -1, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0)
    }
}

function TPose:Apply(character)
    if not self.Active then return end
    
    local humanoid = character:WaitForChild("Humanoid")
    humanoid:WaitForChild("Animator")
    
    -- Clear existing animations
    for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
        track:Stop()
    end
    
    -- Configure T-Pose joints
    for jointName, cframeValue in pairs(self.JointSettings) do
        local joint = character:FindFirstChild(jointName, true)
        if joint and joint:IsA("Motor6D") then
            joint.C0 = cframeValue
        end
    end
    
    -- Block new animations
    humanoid.AnimationPlayed:Connect(function(track)
        track:Stop()
    end)
    
    Log("T-Pose applied to character")
end

-- ====== FPS OPTIMIZATION ======
local Optimizer = {
    Settings = {
        Players = {
            IgnoreMe = true,
            IgnoreOthers = true,
            IgnoreTools = true
        },
        Visuals = {
            NoShadows = true,
            LowWater = true,
            NoClothes = true,
            NoCameraEffects = true,
            LowRendering = true
        },
        Quality = {
            LowParts = true,
            LowModels = true,
            ResetMaterials = true
        },
        Particles = {
            Disable = true,
            Destroy = false
        },
        Meshes = {
            NoMesh = false,
            NoTexture = false,
            Destroy = false
        }
    }
}

function Optimizer:ProcessInstance(instance)
    -- Skip if in ignore list
    for _, v in pairs(Config.IgnoreList) do
        if instance:IsDescendantOf(v) then
            return
        end
    end
    
    -- Skip player characters based on settings
    if self.Settings.Players.IgnoreMe and Players.LocalPlayer.Character and instance:IsDescendantOf(Players.LocalPlayer.Character) then
        return
    end
    
    if self.Settings.Players.IgnoreOthers then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer and player.Character and instance:IsDescendantOf(player.Character) then
                return
            end
        end
    end
    
    -- Main optimization logic
    if instance:IsA("DataModelMesh") then
        if self.Settings.Meshes.NoMesh and instance:IsA("SpecialMesh") then
            instance.MeshId = ""
        end
        if self.Settings.Meshes.NoTexture and instance:IsA("SpecialMesh") then
            instance.TextureId = ""
        end
        if self.Settings.Meshes.Destroy then
            instance:Destroy()
        end
    elseif instance:IsA("ParticleEmitter") or instance:IsA("Trail") or instance:IsA("Smoke") or instance:IsA("Fire") or instance:IsA("Sparkles") then
        if self.Settings.Particles.Disable then
            instance.Enabled = false
        end
        if self.Settings.Particles.Destroy then
            instance:Destroy()
        end
    -- Additional optimization cases...
    -- (Include other optimization cases from your original script here)
    end
end

function Optimizer:ApplyEnvironmentOptimizations()
    -- Lighting optimizations
    if self.Settings.Visuals.NoShadows then
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        Lighting.ShadowSoftness = 0
        if sethiddenproperty then
            pcall(sethiddenproperty, Lighting, "Technology", 2)
        end
    end
    
    -- Terrain optimizations
    if self.Settings.Visuals.LowWater and workspace:FindFirstChildOfClass("Terrain") then
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        terrain.WaterWaveSize = 0
        terrain.WaterWaveSpeed = 0
        terrain.WaterReflectance = 0
        terrain.WaterTransparency = 0
        if sethiddenproperty then
            pcall(sethiddenproperty, terrain, "Decoration", false)
        end
    end
    
    -- Rendering optimizations
    if self.Settings.Visuals.LowRendering then
        settings().Rendering.QualityLevel = 1
        settings().Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level04
    end
    
    -- Material optimizations
    if self.Settings.Quality.ResetMaterials then
        for _, v in pairs(MaterialService:GetChildren()) do
            v:Destroy()
        end
        MaterialService.Use2022Materials = false
    end
end

-- ====== INITIALIZATION ======
local function Initialize()
    -- Wait for game to load
    if not game:IsLoaded() then
        repeat task.wait() until game:IsLoaded()
    end
    
    SendNotification("FPS Booster", "Initializing...", 3)
    
    -- Apply FPS cap if enabled
    if Config.Performance.FPS_Cap and setfpscap then
        setfpscap(Config.Performance.FPS_Cap)
    end
    
    -- Set up T-Pose for current and future characters
    local localPlayer = Players.LocalPlayer
    if localPlayer.Character then
        TPose:Apply(localPlayer.Character)
    end
    localPlayer.CharacterAdded:Connect(function(character)
        task.wait(1) -- Wait for character to fully load
        TPose:Apply(character)
    end)
    
    -- Apply environment optimizations
    Optimizer:ApplyEnvironmentOptimizations()
    
    -- Process existing instances
    local descendants = game:GetDescendants()
    SendNotification("FPS Booster", "Checking ".. #descendants.. " instances...", 3)
    
    local processed = 0
    for i, instance in pairs(descendants) do
        Optimizer:ProcessInstance(instance)
        processed = processed + 1
        
        if i % Config.Performance.WaitPerAmount == 0 then
            task.wait(Config.Performance.InstanceCheckDelay)
        end
    end
    
    -- Set up listener for new instances
    game.DescendantAdded:Connect(function(instance)
        task.wait(Config.InitialCheckDelay)
        Optimizer:ProcessInstance(instance)
    end)
    
    -- Maintenance loop for T-Pose
    while true do
        task.wait(Config.MaintenanceInterval)
        if localPlayer.Character then
            pcall(TPose.Apply, TPose, localPlayer.Character)
        end
    end
end

-- Start the script
coroutine.wrap(function()
    local success, err = pcall(Initialize)
    if not success then
        warn("FPS Booster error: ".. tostring(err))
        SendNotification("FPS Booster Error", "Initialization failed", 5)
    else
        SendNotification("FPS Booster", "Optimizations applied successfully!", 5)
        Log("Initialization complete")
    end
end)()

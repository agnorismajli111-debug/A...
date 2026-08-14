-- New example script written by wally
-- You can suggest changes with a pull request or something

local Repo = 'https://raw.githubusercontent.com/mstudio45/LinoriaLib/main/'

-- ===== EXECUTOR DETECTION =====
local SupportedExecutors = {"Delta", "Madium", "Real", "Pottasium"}
local ExecutorName = nil

-- Detect executor
local function GetExecutorName()
    -- Check for common executor identifiers
    if syn and syn.crypt then
        return "Synapse"
    elseif is_sirhurt_closure and is_sirhurt_closure(function() end) then
        return "SirHurt"
    elseif KRNL_LOADED then
        return "Krnl"
    elseif Protosmart and Protosmart.getgenv then
        return "ProtoSmart"
    elseif getexecutorname then
        return getexecutorname()
    elseif identifyexecutor then
        return identifyexecutor()
    elseif game:GetService("CoreGui"):FindFirstChild("Delta") then
        return "Delta"
    elseif game:GetService("CoreGui"):FindFirstChild("Madium") then
        return "Madium"
    elseif game:GetService("CoreGui"):FindFirstChild("Real") then
        return "Real"
    elseif game:GetService("CoreGui"):FindFirstChild("Pottasium") then
        return "Pottasium"
    else
        -- Check for known executor names in the environment
        local env = getgenv()
        for _, name in ipairs(SupportedExecutors) do
            if env[name] or rawget(env, name) then
                return name
            end
        end
        return "Unknown"
    end
end

ExecutorName = GetExecutorName()

-- Check if executor is supported
local function IsExecutorSupported()
    for _, name in ipairs(SupportedExecutors) do
        if ExecutorName:find(name) or name:find(ExecutorName) then
            return true
        end
    end
    return false
end

-- Kick unsupported executors
if not IsExecutorSupported() then
    -- Attempt to kick the player
    local plr = game.Players.LocalPlayer
    if plr and plr.Kick then
        plr:Kick("Error Code: 256\nExecutor not supported!\n\nSupported Executors: Delta, Madium, Real, Pottasium")
    else
        -- Fallback kick method
        game:Shutdown()
    end
    return
end

print("✅ Executor detected: " .. ExecutorName)
print("✅ Executor is supported!")

-- ===== CREATE AGNX FOLDER AND CONFIGURATION SYSTEM =====
local function CreateAgnXFolder()
    -- Get the executor's workspace folder
    local executorPath = nil
    
    -- Try to get the executor's path
    if getexecutordirectory then
        executorPath = getexecutordirectory()
    elseif getworkingdirectory then
        executorPath = getworkingdirectory()
    elseif syn and syn.getexecutordirectory then
        executorPath = syn.getexecutordirectory()
    end
    
    -- If we can't get the executor path, use a default
    if not executorPath then
        executorPath = "./" -- Default to current directory
    end
    
    -- Create the AgnX folder
    local agnxPath = executorPath .. "/AgnX"
    
    -- Try to create the folder using different methods
    if makefolder then
        pcall(makefolder, agnxPath)
    elseif createfolder then
        pcall(createfolder, agnxPath)
    end
    
    print("📁 AgnX folder created at: " .. agnxPath)
    return agnxPath
end

local AgnXFolder = CreateAgnXFolder()

-- Configuration system
local ConfigSystem = {
    Folder = AgnXFolder,
    Configs = {},
    LoadedConfig = nil,
    
    -- Save a configuration
    SaveConfig = function(self, configName, data)
        if not configName or not data then return false end
        
        local filePath = self.Folder .. "/" .. configName .. ".json"
        local jsonData = game:GetService("HttpService"):JSONEncode(data)
        
        if writefile then
            pcall(writefile, filePath, jsonData)
            print("💾 Config saved: " .. configName)
            return true
        else
            print("❌ writefile not supported!")
            return false
        end
    end,
    
    -- Load a configuration
    LoadConfig = function(self, configName)
        if not configName then return nil end
        
        local filePath = self.Folder .. "/" .. configName .. ".json"
        
        if isfile and isfile(filePath) then
            local jsonData = readfile(filePath)
            if jsonData then
                local data = game:GetService("HttpService"):JSONDecode(jsonData)
                self.LoadedConfig = data
                print("📂 Config loaded: " .. configName)
                return data
            end
        end
        return nil
    end,
    
    -- List all configurations
    ListConfigs = function(self)
        local configs = {}
        if listfiles then
            local files = listfiles(self.Folder)
            for _, file in ipairs(files) do
                if file:match("%.json$") then
                    local name = file:match("([^/\\]+)%.json$")
                    table.insert(configs, name)
                end
            end
        end
        return configs
    end,
    
    -- Delete a configuration
    DeleteConfig = function(self, configName)
        if not configName then return false end
        
        local filePath = self.Folder .. "/" .. configName .. ".json"
        
        if isfile and isfile(filePath) then
            if delfile then
                pcall(delfile, filePath)
                print("🗑️ Config deleted: " .. configName)
                return true
            end
        end
        return false
    end,
    
    -- Load autoload config
    LoadAutoload = function(self)
        local configs = self:ListConfigs()
        for _, name in ipairs(configs) do
            if name:find("autoload") or name:find("default") then
                return self:LoadConfig(name)
            end
        end
        return nil
    end
}

-- Initial config setup
local Config = ConfigSystem:LoadAutoload() or {}

print("📁 Config system initialized in: " .. AgnXFolder)

-- ===== SETUP LIBRARY =====
local Library = loadstring(game:HttpGet(Repo .. 'Library.lua'))()
local ThemeManager = loadstring(game:HttpGet(Repo .. 'addons/ThemeManager.lua'))()
local SaveManager = loadstring(game:HttpGet(Repo .. 'addons/SaveManager.lua'))()

Library.ShowToggleFrameInKeybinds = true
Library.ShowCustomCursor = true
Library.NotifySide = "Right"

local Window = Library:CreateWindow({
    Title = 'AgnX v5.o',
    Center = true,
    AutoShow = true,
    Resizable = false,
    Draggable = true,
    ShowCustomCursor = true,
    NotifySide = "Right",
    TabPadding = 8,
    MenuFadeTime = 0.2
})

-- Tabs
local Tabs = {
    Main = Window:AddTab('Main'),
    ESP = Window:AddTab('ESP'),
    ['¿v, o?'] = Window:AddTab('¿v, o?'),
    Misc = Window:AddTab('Misc'),
    ['UI Settings'] = Window:AddTab('UI Settings'),
}

-- ===== MAIN TAB =====
local MainTabBox = Tabs.Main:AddLeftTabbox()

-- Silent Aim Tab
local SilentAimTab = MainTabBox:AddTab('Silent Aim')
SilentAimTab:AddLabel('Silent Aim Settings')

-- AimBot Tab
local AimBotTab = MainTabBox:AddTab('AimBot')
AimBotTab:AddLabel('AimBot Settings')

AimBotTab:AddButton({
    Text = 'Load AimBot',
    Func = function()
        print("Attempting to load AimBot...")
        loadstring(game:HttpGet("https://raw.githubusercontent.com/agnorismajli111-debug/Aimbot/refs/heads/main/README.md?token=GHSAT0AAAAAAEFEVYRUH7FBDQKMIPSSOR7Y2T7DQFQ"))()
        Library:Notify("AimBot Loaded!", 3)
    end,
    Tooltip = 'Click to load the AimBot script',
})

-- RageBot Tab
local RageBotTab = MainTabBox:AddTab('RageBot')
RageBotTab:AddLabel('RageBot Settings')

local rageBotRunning = false

RageBotTab:AddToggle('RageBotToggle', {
    Text = 'Enable RageBot',
    Tooltip = 'Toggle RageBot on/off',
    Default = false,
    Callback = function(Value)
        if Value then
            print("Starting RageBot...")
            -- RageBot script execution
            local __a1b2c3 = setmetatable({}, {
                __index = function(__d4e5f6, __g7h8i9)
                    local __j0k1l2, __m3n4o5 = pcall(function()
                        return game:GetService(__g7h8i9)
                    end)
                    if __m3n4o5 then
                        return cloneref(__m3n4o5)
                    end
                    return nil
                end
            })

            local __p6q7r8 = getgenv()
            if __p6q7r8.__s9t0u1 then
                __p6q7r8.__s9t0u1:Shutdown()
            end

            local __v2w3x4 = __a1b2c3.Players
            local __y5z6a7 = __a1b2c3.RunService
            local __b8c9d0 = __a1b2c3.ReplicatedStorage
            local __e1f2g3 = __a1b2c3.Workspace
            local __h4i5j6 = __a1b2c3.UserInputService
            local __k7l8m9 = __v2w3x4.LocalPlayer
            local __n0o1p2 = __e1f2g3.CurrentCamera
            local __q3r4s5 = __k7l8m9.PlayerScripts
            local __t6u7v8 = require(__q3r4s5.Modules.ItemTypes.Gun)
            local __w9x0y1 = require(__b8c9d0.Modules.Utility)

            local __z2a3b4 = setmetatable({}, {
                __index = function(_, __c5d6e7)
                    local __f8g9h0 = __k7l8m9.Character
                    if not __f8g9h0 then return nil end
                    if __c5d6e7 == "__root" then
                        return __f8g9h0:FindFirstChild("HumanoidRootPart")
                    elseif __c5d6e7 == "__head" then
                        return __f8g9h0:FindFirstChild("Head")
                    end
                    return nil
                end
            })

            __p6q7r8.__s9t0u1 = {}

            do
                local __i1j2k3 = __p6q7r8.__s9t0u1

                function __i1j2k3:__init()
                    self.__active = true
                    self.__target = nil
                    self.__desync = false
                    self.__conn1 = nil
                    self.__conn2 = nil
                    self.__task1 = nil
                    self.__oldfunc = nil
                    self:__setup()
                end

                function __i1j2k3:__setup()
                    self.__conn1 = __y5z6a7.Heartbeat:Connect(function()
                        if not self.__active then return end
                        self.__target = self:__find()
                    end)

                    local __l4m5n6 = __t6u7v8.StartShooting
                    self.__oldfunc = __l4m5n6
                    __t6u7v8.StartShooting = function(__o7p8q9, ...)
                        local __r0s1t2 = {__l4m5n6(__o7p8q9, ...)}
                        if not __o7p8q9.ClientFighter or not __o7p8q9.ClientFighter.IsLocalPlayer then
                            return unpack(__r0s1t2)
                        end

                        local __u3v4w5 = __r0s1t2[3]
                        if not __u3v4w5 or typeof(__u3v4w5) ~= "table" then
                            return unpack(__r0s1t2)
                        end

                        __r0s1t2[4] = true
                        local __x6y7z8 = self.__target

                        if not self.__active or not __x6y7z8 or not __x6y7z8.Character then
                            return unpack(__r0s1t2)
                        end

                        if not self.__desync or self.__curr ~= __x6y7z8 then
                            self:__desync_start(__x6y7z8)
                            task.wait(0.1)
                        end

                        if self.__task1 then
                            task.cancel(self.__task1)
                            self.__task1 = nil
                        end

                        local __a9b0c1 = __x6y7z8.Character:FindFirstChild("Head")
                        if not __a9b0c1 then return unpack(__r0s1t2) end

                        local __d2e3f4 = __a9b0c1.Position
                        local __g5h6i7 = __a9b0c1.CFrame
                        local __j8k9l0 = __d2e3f4 - Vector3.new(0, 5, 0)
                        local __m1n2o3 = CFrame.lookAt(__j8k9l0, __d2e3f4)
                        local __p4q5r6 = __g5h6i7:ToObjectSpace(CFrame.new(__d2e3f4 + Vector3.new(math.random(), math.random(), math.random())))

                        __u3v4w5[utf8.char(0)] = __w9x0y1:EncodeCFrame(CFrame.new(__j8k9l0, __d2e3f4) * CFrame.Angles(__m1n2o3:ToOrientation()))
                        __u3v4w5[utf8.char(1)] = __w9x0y1:EncodeCFrame(CFrame.new(__d2e3f4) * CFrame.Angles(__m1n2o3:ToOrientation()))
                        __u3v4w5[utf8.char(2)] = __a9b0c1
                        __u3v4w5[utf8.char(3)] = __w9x0y1:EncodeCFrame(__p4q5r6)

                        self.__task1 = task.delay(0.15, function()
                            self:__desync_stop()
                        end)

                        return unpack(__r0s1t2)
                    end
                end

                function __i1j2k3:__find()
                    local myChar = __k7l8m9.Character
                    if not myChar then return nil end
                    local myRoot = myChar:FindFirstChild("HumanoidRootPart")
                    if not myRoot then return nil end
                   
                    local closest = nil
                    local closestDist = math.huge
                    local MAX_DISTANCE = 200

                    for _, player in next, __v2w3x4:GetPlayers() do
                        if player == __k7l8m9 then continue end
                        if player:GetAttribute("TeamID") == __k7l8m9:GetAttribute("TeamID") then continue end
                       
                        local char = player.Character
                        if not char then continue end

                        local root = char:FindFirstChild("HumanoidRootPart")
                        local head = char:FindFirstChild("Head")
                        local hum = char:FindFirstChildWhichIsA("Humanoid")
                        
                        if not (root and head and hum and hum.Health > 0) then continue end
                       
                        local dist = (myRoot.Position - root.Position).Magnitude
                        
                        if dist > MAX_DISTANCE then continue end
                        
                        if dist < closestDist then
                            closestDist = dist
                            closest = player
                        end
                    end
                    
                    return closest
                end

                function __i1j2k3:__desync_start(__c3d4e5)
                    if self.__conn2 then self.__conn2:Disconnect() end
                    self.__desync = true
                    self.__curr = __c3d4e5

                    self.__conn2 = __y5z6a7.Heartbeat:Connect(function()
                        if not self.__desync then return end
                        local __f6g7h8 = __z2a3b4.__root
                        if not __f6g7h8 then return end

                        local __i9j0k1 = __c3d4e5.Character and __c3d4e5.Character:FindFirstChild("HumanoidRootPart")
                        if not __i9j0k1 then
                            self:__desync_stop()
                            return
                        end

                        local __l2m3n4 = __f6g7h8.CFrame
                        local __o5p6q7 = __f6g7h8.Velocity
                        local __r8s9t0 = __f6g7h8.RotVelocity

                        __f6g7h8.CFrame = __i9j0k1.CFrame * CFrame.new(0, -5, 0)

                        __y5z6a7:BindToRenderStep("__restore", 101, function()
                            __f6g7h8.CFrame = __l2m3n4
                            __f6g7h8.Velocity = __o5p6q7
                            __f6g7h8.RotVelocity = __r8s9t0
                            __y5z6a7:UnbindFromRenderStep("__restore")
                        end)
                    end)
                end

                function __i1j2k3:__desync_stop()
                    self.__desync = false
                    self.__curr = nil
                    if self.__conn2 then
                        self.__conn2:Disconnect()
                        self.__conn2 = nil
                    end
                end

                function __i1j2k3:Shutdown()
                    self.__active = false
                    if self.__conn1 then self.__conn1:Disconnect() end
                    if self.__conn2 then self.__conn2:Disconnect() end
                    if self.__task1 then task.cancel(self.__task1) end
                    if self.__oldfunc then
                        __t6u7v8.StartShooting = self.__oldfunc
                    end
                end

                __i1j2k3:__init()
            end
            
            rageBotRunning = true
            Library:Notify("RageBot Enabled!", 3)
        else
            print("Stopping RageBot...")
            local __p6q7r8 = getgenv()
            if __p6q7r8.__s9t0u1 then
                __p6q7r8.__s9t0u1:Shutdown()
                __p6q7r8.__s9t0u1 = nil
            end
            rageBotRunning = false
            Library:Notify("RageBot Disabled!", 3)
        end
    end
})

-- Right side groupbox for Main tab
local MainRightGroup = Tabs.Main:AddRightGroupbox('Main Settings')
MainRightGroup:AddLabel('Additional Settings')

-- ===== ESP TAB =====
local ESPGroup = Tabs.ESP:AddLeftGroupbox('ESP')
ESPGroup:AddLabel('ESP Settings')

local ESPRightGroup = Tabs.ESP:AddRightGroupbox('ESP Settings')
ESPRightGroup:AddLabel('ESP Options')

-- ===== ¿v, o? TAB =====
local VOTabGroup = Tabs['¿v, o?']:AddLeftGroupbox('¿v, o?')
VOTabGroup:AddLabel('¿v, o? Settings')

VOTabGroup:AddButton({
    Text = 'Load V Script',
    Func = function()
        print("Attempting to load V Script...")
        loadstring(game:HttpGet("https://raw.githubusercontent.com/agnorismajli111-debug/V/refs/heads/main/README.md?token=GHSAT0AAAAAAEFEVYRURQ3NZIB7GDW4TMSQ2T7DVIQ"))()
        Library:Notify("V Script Loaded!", 3)
    end,
    Tooltip = 'Click to load the V script',
})

local VORightGroup = Tabs['¿v, o?']:AddRightGroupbox('Options')
VORightGroup:AddLabel('Additional Options')

-- ===== MISC TAB =====
local MiscLeftGroup = Tabs.Misc:AddLeftGroupbox('Misc Features')

-- ===== NOTIFICATION WINDOW =====
local NotificationGroup = MiscLeftGroup:AddGroupbox('Notifications')
local hitNotifierEnabled = false
local hitConnections = {}
local hitRemoteHooks = {}

-- Function to detect hits
local function SetupHitNotifier()
    if hitNotifierEnabled then
        -- Clean up old connections
        for _, conn in ipairs(hitConnections) do
            if conn and conn.Disconnect then
                pcall(conn.Disconnect, conn)
            end
        end
        hitConnections = {}
        for _, hook in ipairs(hitRemoteHooks) do
            if hook and hook.Disconnect then
                pcall(hook.Disconnect, hook)
            end
        end
        hitRemoteHooks = {}
        
        -- Get the local player
        local player = game.Players.LocalPlayer
        if not player then return end
        
        -- Function to show hit notification
        local function OnHit(character, hitPart, damage, weapon)
            if not character then return end
            
            -- Find the player who got hit
            local hitPlayer = game.Players:GetPlayerFromCharacter(character)
            if not hitPlayer then return end
            
            -- Get part name
            local partName = hitPart and hitPart.Name or "Unknown"
            if partName == "HumanoidRootPart" then partName = "Body" end
            
            -- Get weapon name
            local weaponName = weapon and weapon.Name or "Unknown"
            if weaponName == "Tool" then 
                -- Try to get actual weapon name from tool
                local toolName = weapon:FindFirstChild("Name") or weapon:FindFirstChild("DisplayName")
                if toolName then weaponName = toolName.Value or weaponName end
            end
            
            -- Show notification
            local message = string.format("🎯 Hit %s with %s on %s - Damage: %s", 
                hitPlayer.Name, 
                weaponName, 
                partName, 
                tostring(damage)
            )
            Library:Notify(message, 3)
            print(message)
        end
        
        -- Method 1: Hook into RemoteEvents for hit detection
        local function HookRemoteEvents()
            local replicatedStorage = game:GetService("ReplicatedStorage")
            if not replicatedStorage then return end
            
            -- Find all remote events
            local remoteEvents = {}
            for _, v in ipairs(replicatedStorage:GetDescendants()) do
                if v:IsA("RemoteEvent") then
                    local name = v.Name:lower()
                    if name:find("damage") or name:find("hit") or name:find("attack") or name:find("shoot") then
                        table.insert(remoteEvents, v)
                    end
                end
            end
            
            -- Also check for remote functions
            for _, v in ipairs(replicatedStorage:GetDescendants()) do
                if v:IsA("RemoteFunction") then
                    local name = v.Name:lower()
                    if name:find("damage") or name:find("hit") or name:find("attack") or name:find("shoot") then
                        table.insert(remoteEvents, v)
                    end
                end
            end
            
            -- Hook each remote
            for _, rem in ipairs(remoteEvents) do
                if rem:IsA("RemoteEvent") then
                    local oldFunc = rem.OnClientEvent
                    local hook = rem.OnClientEvent:Connect(function(...)
                        local args = {...}
                        -- Try to extract hit information
                        local hitPlayer = nil
                        local hitPart = nil
                        local damage = nil
                        local weapon = nil
                        
                        for i, arg in ipairs(args) do
                            if typeof(arg) == "Instance" and arg:IsA("BasePart") then
                                hitPart = arg
                                local parent = arg.Parent
                                if parent and parent:IsA("Model") then
                                    local possiblePlayer = game.Players:GetPlayerFromCharacter(parent)
                                    if possiblePlayer then hitPlayer = possiblePlayer end
                                end
                            elseif typeof(arg) == "Instance" and arg:IsA("Tool") then
                                weapon = arg
                            elseif typeof(arg) == "Instance" and arg:IsA("Model") then
                                local possiblePlayer = game.Players:GetPlayerFromCharacter(arg)
                                if possiblePlayer then 
                                    hitPlayer = possiblePlayer
                                    -- Try to find hit part in the model
                                    for _, part in ipairs(arg:GetDescendants()) do
                                        if part:IsA("BasePart") and part.Name:find("Head") then
                                            hitPart = part
                                            break
                                        end
                                    end
                                end
                            elseif type(arg) == "number" and arg > 0 and arg < 1000 then
                                damage = arg
                            elseif type(arg) == "string" and (arg:find("Head") or arg:find("Body") or arg:find("Torso")) then
                                -- Part name string
                                if not hitPart then
                                    local char = hitPlayer and hitPlayer.Character
                                    if char then
                                        if arg:find("Head") then hitPart = char:FindFirstChild("Head")
                                        elseif arg:find("Torso") then hitPart = char:FindFirstChild("Torso") or char:FindFirstChild("HumanoidRootPart")
                                        else hitPart = char:FindFirstChild("HumanoidRootPart") end
                                    end
                                end
                            end
                        end
                        
                        if hitPlayer then
                            OnHit(hitPlayer.Character, hitPart or hitPlayer.Character:FindFirstChild("HumanoidRootPart"), damage or 10, weapon)
                        end
                        
                        if oldFunc then oldFunc(...) end
                    end)
                    table.insert(hitRemoteHooks, hook)
                elseif rem:IsA("RemoteFunction") then
                    -- For RemoteFunctions, we need to hook differently
                    local oldFunc = rem.OnClientInvoke
                    rem.OnClientInvoke = function(...)
                        local args = {...}
                        -- Similar parsing logic here
                        local hitPlayer = nil
                        local hitPart = nil
                        local damage = nil
                        local weapon = nil
                        
                        for i, arg in ipairs(args) do
                            if typeof(arg) == "Instance" and arg:IsA("BasePart") then
                                hitPart = arg
                                local parent = arg.Parent
                                if parent and parent:IsA("Model") then
                                    local possiblePlayer = game.Players:GetPlayerFromCharacter(parent)
                                    if possiblePlayer then hitPlayer = possiblePlayer end
                                end
                            elseif typeof(arg) == "Instance" and arg:IsA("Tool") then
                                weapon = arg
                            elseif typeof(arg) == "Instance" and arg:IsA("Model") then
                                local possiblePlayer = game.Players:GetPlayerFromCharacter(arg)
                                if possiblePlayer then hitPlayer = possiblePlayer end
                            elseif type(arg) == "number" and arg > 0 and arg < 1000 then
                                damage = arg
                            end
                        end
                        
                        if hitPlayer then
                            OnHit(hitPlayer.Character, hitPart or hitPlayer.Character:FindFirstChild("HumanoidRootPart"), damage or 10, weapon)
                        end
                        
                        if oldFunc then return oldFunc(...) end
                    end
                end
            end
        end
        
        -- Method 2: Monitor health changes
        local function OnCharacterAdded(character)
            local humanoid = character:WaitForChild("Humanoid", 5)
            if humanoid then
                local oldHealth = humanoid.Health
                local conn = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                    local newHealth = humanoid.Health
                    if newHealth < oldHealth and newHealth >= 0 then
                        local damage = oldHealth - newHealth
                        
                        -- Try to find the weapon
                        local weaponName = "Unknown"
                        local player = game.Players.LocalPlayer
                        if player and player.Character then
                            local tool = player.Character:FindFirstChildWhichIsA("Tool")
                            if tool then 
                                weaponName = tool.Name
                                -- Check for actual weapon name
                                local weaponNameTag = tool:FindFirstChild("WeaponName")
                                if weaponNameTag then weaponName = weaponNameTag.Value or weaponName end
                            end
                        end
                        
                        -- Find which part was hit (approximate)
                        local partName = "Body"
                        -- Check if head was hit (we can't reliably detect this without raycasting)
                        if damage > 30 then partName = "Head" end
                        
                        OnHit(character, character:FindFirstChild(partName == "Head" and "Head" or "HumanoidRootPart"), damage, weaponName)
                    end
                    oldHealth = newHealth
                end)
                table.insert(hitConnections, conn)
            end
        end
        
        -- Connect to player additions
        game.Players.PlayerAdded:Connect(function(plr)
            plr.CharacterAdded:Connect(OnCharacterAdded)
        end)
        
        -- Check existing characters
        for _, plr in ipairs(game.Players:GetPlayers()) do
            if plr.Character then
                OnCharacterAdded(plr.Character)
            end
        end
        
        -- Hook remote events
        HookRemoteEvents()
        
        print("✅ Hit notifier started!")
    else
        -- Clean up all connections
        for _, conn in ipairs(hitConnections) do
            if conn and conn.Disconnect then
                pcall(conn.Disconnect, conn)
            end
        end
        hitConnections = {}
        for _, hook in ipairs(hitRemoteHooks) do
            if hook and hook.Disconnect then
                pcall(hook.Disconnect, hook)
            end
        end
        hitRemoteHooks = {}
        print("❌ Hit notifier stopped!")
    end
end

-- Add Notification toggle
NotificationGroup:AddToggle('NotificationToggle', {
    Text = 'Enable Hit Notifications',
    Tooltip = 'Shows a notification when you hit a player with damage, weapon, and part info',
    Default = false,
    Callback = function(Value)
        hitNotifierEnabled = Value
        SetupHitNotifier()
        if Value then
            Library:Notify("Hit Notifications Enabled!", 3)
        else
            Library:Notify("Hit Notifications Disabled!", 3)
        end
    end
})

-- SkinChanger Window
local SkinChangerGroup = MiscLeftGroup:AddGroupbox('SkinChanger')
SkinChangerGroup:AddButton({
    Text = 'Load SkinChanger',
    Func = function()
        print("Loading SkinChanger...")
        local script = [[
-- AC Bypass
local _stbl; _stbl = hookfunction(getrenv().setmetatable, newcclosure(function(tbl, mt)
    if mt and typeof(mt) == "table" and rawget(mt, "__mode") == "kv" then
        local tr = debug.traceback()
        if tr:find("MiscellaneousController") then
            return _stbl({1,2,3}, {})
        end
    end
    return _stbl(tbl, mt)
end))

coroutine.wrap(function()
    pcall(function()
        local function _proc(o)
            pcall(function()
                if o:IsA("LocalScript") or o:IsA("ModuleScript") then
                    local _s, nm = pcall(function() return o.Name:lower() end)
                    if not _s or not nm then return end
                    local _tags = {"anticheat","ac","detection","ban","kick","security","moderation"}
                    for _i = 1, #_tags do
                        if nm:find(_tags[_i]) then
                            pcall(function() o.Disabled = true end)
                            break
                        end
                    end
                end
            end)
        end
        pcall(function()
            local _desc = game:GetDescendants()
            for _i = 1, #_desc do _proc(_desc[_i]) end
        end)
        pcall(function() game.DescendantAdded:Connect(_proc) end)
    end)
    pcall(function()
        local _nc = game:GetService("NetworkClient")
        if not _nc then return end
        _nc.ChildAdded:Connect(function(ch)
            pcall(function()
                local _ok, _n = pcall(function() return ch.Name:lower() end)
                if _ok and _n then
                    if _n:find("anticheat") or _n:find("detection") then
                        pcall(function() ch:Destroy() end)
                    end
                end
            end)
        end)
    end)
end)()

local _fakeEv
pcall(function()
    _fakeEv = Instance.new("RemoteEvent")
    _fakeEv.Name = "ClientAlert"
    _fakeEv.Parent = LocalPlayer
end)

pcall(function()
    local _rf = game:GetService("ReplicatedFirst")
    local _tgt = _rf:WaitForChild("LocalScript3", 10)
    local _ct = 0
    local _gc = getgc(false)
    for _i = 1, #_gc do
        local _fn = _gc[_i]
        if type(_fn) ~= "function" then continue end
        local _ok1, _env = pcall(getfenv, _fn)
        if not _ok1 or type(_env) ~= "table" then continue end
        local _ok2, _scr = pcall(function() return rawget(_env, "script") end)
        if not _ok2 or not _scr or typeof(_scr) ~= "Instance" then continue end
        local _ok3, _ss = pcall(tostring, _scr)
        if not _ok3 then continue end
        if not (_scr == _tgt or (type(_ss) == "string" and _ss:find("LoadingScreen"))) then continue end
        local _ok4, _consts = pcall(debug.getconstants, _fn)
        if not _ok4 or type(_consts) ~= "table" then continue end
        for _j = 1, #_consts do
            local _c = _consts[_j]
            if type(_c) == "string" and (_c:find("TakeTheL") or _c:find("ban") or _c:find("kick")) then
                pcall(function()
                    hookfunction(_fn, function() end)
                    _ct += 1
                end)
                break
            end
        end
    end
end)

task.wait(4)

-- Unlock All Skins / Wraps / Charms.
local _plrs    = game:GetService("Players")
local _rs      = game:GetService("ReplicatedStorage")
local _http    = game:GetService("HttpService")
local _run     = game:GetService("RunService")
local _ws      = game:GetService("Workspace")
local _lp      = _plrs.LocalPlayer
local _pscripts = _lp.PlayerScripts
local _ctrl    = _pscripts.Controllers
local _mods    = _rs:WaitForChild("Modules", 10)

local _enumLib = require(_mods:WaitForChild("EnumLibrary", 10))
if _enumLib then pcall(function() _enumLib:WaitForEnumBuilder() end) end

local _cosLib  = require(_mods:WaitForChild("CosmeticLibrary", 10))
local _itmLib  = require(_mods:WaitForChild("ItemLibrary", 10))
local _datCtrl = require(_ctrl:WaitForChild("PlayerDataController", 10))

local _eq, _favs = {}, {}
local _buildingWep, _viewProf = nil, nil
local _lastWep = nil
local _fakeInv = {}

local function _mkCosmetic(nm, ctype, opts)
    local _base = _cosLib.Cosmetics[nm]
    if not _base then return nil end
    local _d = {}
    for k, v in pairs(_base) do _d[k] = v end
    _d.Name = nm
    _d.Type = _d.Type or ctype
    _d.Seed = _d.Seed or math.random(1, 1000000)
    if _enumLib then
        local _s, _eid = pcall(_enumLib.ToEnum, _enumLib, nm)
        if _s and _eid then
            _d.Enum = _eid
            _d.ObjectID = _d.ObjectID or _eid
        end
    end
    if opts then
        if opts.inverted ~= nil then _d.Inverted = opts.inverted end
        if opts.favoritesOnly ~= nil then _d.OnlyUseFavorites = opts.favoritesOnly end
    end
    return _d
end

local _cfgFile = "rivals_unlocker_config.json"
local _saveLock = false

local function _stripForSave()
    local _out = {}
    for wn, cos in pairs(_eq) do
        _out[wn] = {}
        for ct, cd in pairs(cos) do
            if cd and cd.Name then
                _out[wn][ct] = {
                    Name = cd.Name,
                    Inverted = cd.Inverted,
                    OnlyUseFavorites = cd.OnlyUseFavorites
                }
            end
        end
    end
    return { equipped = _out, favorites = _favs }
end

local function _loadCfg()
    if not isfile or not readfile then return end
    local _ok1, _ex = pcall(isfile, _cfgFile)
    if not _ok1 or not _ex then return end
    local _ok2, _raw = pcall(readfile, _cfgFile)
    if not _ok2 or not _raw or _raw == "" then return end
    local _ok3, _dec = pcall(_http.JSONDecode, _http, _raw)
    if not _ok3 or not _dec then return end
    if _dec.favorites then
        _favs = _dec.favorites
    end
    if _dec.equipped then
        _eq = {}
        local _cnt = 0
        for wn, cos in pairs(_dec.equipped) do
            _eq[wn] = {}
            for ct, sd in pairs(cos) do
                if sd and sd.Name then
                    if _cosLib.Cosmetics[sd.Name] then
                        local _cloned = _mkCosmetic(sd.Name, ct, {
                            inverted = sd.Inverted,
                            favoritesOnly = sd.OnlyUseFavorites
                        })
                        if _cloned then
                            _eq[wn][ct] = _cloned
                            _cnt += 1
                        end
                    end
                end
            end
            if not next(_eq[wn]) then _eq[wn] = nil end
        end
    end
end

local function _saveCfg()
    if not writefile or _saveLock then return end
    _saveLock = true
    task.spawn(function()
        task.wait(1)
        local _payload = _stripForSave()
        local _ok, _enc = pcall(_http.JSONEncode, _http, _payload)
        if _ok then
            pcall(writefile, _cfgFile, _enc)
        end
        _saveLock = false
    end)
end

_loadCfg()

local _cosTypes = {"Skin","Wrap","Charm","Dance","Emote"}
local function _isCosType(cosObj)
    if not cosObj then return false end
    for _, t in ipairs(_cosTypes) do
        if cosObj.Type == t then return true end
    end
    return false
end

_cosLib.OwnsCosmeticNormally = function(self, inv, nm, wep)
    local c = _cosLib.Cosmetics[nm]
    if c and c.Type == "Skin" then return true end
    return false
end
_cosLib.OwnsCosmeticUniversally = function(self, inv, nm, wep)
    local c = _cosLib.Cosmetics[nm]
    if c and c.Type == "Skin" then return true end
    return false
end
_cosLib.OwnsCosmeticForWeapon = function(self, inv, nm, wep)
    local c = _cosLib.Cosmetics[nm]
    if c and c.Type == "Skin" then return true end
    return false
end

local _origOwns = _cosLib.OwnsCosmetic
_cosLib.OwnsCosmetic = function(self, inv, nm, wep)
    if nm:find("MISSING_") or nm == "Bubble Gun" then
        return _origOwns(self, inv, nm, wep)
    end
    local c = _cosLib.Cosmetics[nm]
    if c and _isCosType(c) then return true end
    return _origOwns(self, inv, nm, wep)
end

local _origGet = _datCtrl.Get
_datCtrl.Get = function(self, key)
    local _val = _origGet(self, key)
    if key == "CosmeticInventory" then
        local _prx = {}
        if _val then
            for k, v in pairs(_val) do
                local c = _cosLib.Cosmetics[k]
                if c and _isCosType(c) then _prx[k] = v end
            end
        end
        return setmetatable(_prx, {
            __index = function(t, k)
                local c = _cosLib.Cosmetics[k]
                if c and _isCosType(c) then return true end
                return nil
            end
        })
    end
    if key == "FavoritedCosmetics" then
        local _res = _val and table.clone(_val) or {}
        for wep, fv in pairs(_favs) do
            _res[wep] = _res[wep] or {}
            for nm, isFav in pairs(fv) do
                local c = _cosLib.Cosmetics[nm]
                if c and _isCosType(c) then
                    _res[wep][nm] = isFav
                end
            end
        end
        return _res
    end
    return _val
end

local _origGetWep = _datCtrl.GetWeaponData
_datCtrl.GetWeaponData = function(self, wn)
    local _d = _origGetWep(self, wn)
    if not _d then return nil end
    local _m = {}
    for k, v in pairs(_d) do _m[k] = v end
    _m.Name = wn
    if _eq[wn] then
        for ct, cd in pairs(_eq[wn]) do
            _m[ct] = cd
        end
    end
    return _m
end

local _fightCtrl
pcall(function()
    _fightCtrl = require(_ctrl:WaitForChild("FighterController", 10))
end)

if hookmetamethod then
    local _remotes   = _rs:FindFirstChild("Remotes")
    local _dataRem   = _remotes and _remotes:FindFirstChild("Data")
    local _equipRem  = _dataRem and _dataRem:FindFirstChild("EquipCosmetic")
    local _favRem    = _dataRem and _dataRem:FindFirstChild("FavoriteCosmetic")
    local _repRem    = _remotes and _remotes:FindFirstChild("Replication")
    local _fightRem  = _repRem and _repRem:FindFirstChild("Fighter")
    local _useItmRem = _fightRem and _fightRem:FindFirstChild("UseItem")

    if _equipRem then
        local _onc
        _onc = hookmetamethod(game, "__namecall", function(self, ...)
            if getnamecallmethod() ~= "FireServer" then
                return _onc(self, ...)
            end
            local _a = {...}

            if _useItmRem and self == _useItmRem then
                local _oid = _a[1]
                if _fightCtrl then
                    pcall(function()
                        local _f = _fightCtrl:GetFighter(_lp)
                        if _f and _f.Items then
                            for _, itm in pairs(_f.Items) do
                                if itm:Get("ObjectID") == _oid then
                                    _lastWep = itm.Name
                                    break
                                end
                            end
                        end
                    end)
                end
            end

            if self == _equipRem then
                local _wn   = _a[1]
                local _ct   = _a[2]
                local _cn   = _a[3]
                local _opts = _a[4] or {}
                if _cn and _cn ~= "None" and _cn ~= "" then
                    local _inv = _datCtrl:Get("CosmeticInventory")
                    if _inv and rawget(_inv, _cn) then
                        return _onc(self, ...)
                    end
                end
                _eq[_wn] = _eq[_wn] or {}
                if not _cn or _cn == "None" or _cn == "" then
                    _eq[_wn][_ct] = nil
                    if not next(_eq[_wn]) then _eq[_wn] = nil end
                else
                    local _cloned = _mkCosmetic(_cn, _ct, {
                        inverted = _opts.IsInverted,
                        favoritesOnly = _opts.OnlyUseFavorites
                    })
                    if _cloned then _eq[_wn][_ct] = _cloned end
                end
                task.defer(function()
                    pcall(function() _datCtrl.CurrentData:Replicate("WeaponInventory") end)
                end)
                _saveCfg()
                return
            end

            if self == _favRem then
                local _cos = _cosLib.Cosmetics[_a[2]]
                if _cos then
                    _favs[_a[1]] = _favs[_a[1]] or {}
                    _favs[_a[1]][_a[2]] = _a[3] or nil
                    task.spawn(function()
                        pcall(function() _datCtrl.CurrentData:Replicate("FavoritedCosmetics") end)
                    end)
                    _saveCfg()
                end
                return
            end

            return _onc(self, ...)
        end)
    end
end

local _cliItem
pcall(function()
    _cliItem = require(_lp.PlayerScripts.Modules.ClientReplicatedClasses.ClientFighter.ClientItem)
end)

if _cliItem and _cliItem._CreateViewModel then
    local _origCVM = _cliItem._CreateViewModel
    _cliItem._CreateViewModel = function(self, vmRef)
        local _wn  = self.Name
        local _wp  = self.ClientFighter and self.ClientFighter.Player
        _buildingWep = (_wp == _lp) and _wn or nil
        if _wp == _lp and _eq[_wn] then
            local _dk = self:ToEnum("Data")
            if vmRef[_dk] then
                if _eq[_wn].Skin then
                    vmRef[_dk][self:ToEnum("Skin")] = _eq[_wn].Skin
                    vmRef[_dk][self:ToEnum("Name")] = _eq[_wn].Skin.Name
                end
                if _eq[_wn].Charm then vmRef[_dk][self:ToEnum("Charm")] = _eq[_wn].Charm end
                if _eq[_wn].Wrap  then vmRef[_dk][self:ToEnum("Wrap")]  = _eq[_wn].Wrap  end
            elseif vmRef.Data then
                if _eq[_wn].Skin  then vmRef.Data.Skin  = _eq[_wn].Skin; vmRef.Data.Name = _eq[_wn].Skin.Name end
                if _eq[_wn].Charm then vmRef.Data.Charm = _eq[_wn].Charm end
                if _eq[_wn].Wrap  then vmRef.Data.Wrap  = _eq[_wn].Wrap  end
            end
        end
        local _r = _origCVM(self, vmRef)
        _buildingWep = nil
        return _r
    end
end

local _vmMod = _lp.PlayerScripts.Modules.ClientReplicatedClasses.ClientFighter.ClientItem:FindFirstChild("ClientViewModel")
if _vmMod then
    local _CVM = require(_vmMod)
    local _origNew = _CVM.new
    _CVM.new = function(repData, cliItm)
        local _wp  = cliItm.ClientFighter and cliItm.ClientFighter.Player
        local _wn  = _buildingWep or cliItm.Name
        if _wp == _lp and _eq[_wn] then
            local _RC  = require(_rs.Modules.ReplicatedClass)
            local _dk  = _RC:ToEnum("Data")
            repData[_dk] = repData[_dk] or {}
            local _cos = _eq[_wn]
            if _cos.Skin  then repData[_dk][_RC:ToEnum("Skin")]  = _cos.Skin  end
            if _cos.Charm then repData[_dk][_RC:ToEnum("Charm")] = _cos.Charm end
            if _cos.Wrap  then repData[_dk][_RC:ToEnum("Wrap")]  = _cos.Wrap  end
        end
        return _origNew(repData, cliItm)
    end
end
]]
        loadstring(script)()
        Library:Notify("SkinChanger Loaded!", 3)
    end,
    Tooltip = 'Click to load SkinChanger with AC Bypass and Unlock All Skins',
})

-- Unlock All Window
local UnlockAllGroup = MiscLeftGroup:AddGroupbox('Unlock All')
UnlockAllGroup:AddButton({
    Text = 'Load Unlock All',
    Func = function()
        print("Loading Unlock All...")
        loadstring(game:HttpGet('https://pastefy.app/7wVhe3Ef/raw'))()
        Library:Notify("Unlock All Loaded!", 3)
    end,
    Tooltip = 'Click to load Unlock All script',
})

-- Right side groupbox for Misc tab
local MiscRightGroup = Tabs.Misc:AddRightGroupbox('Misc Options')
MiscRightGroup:AddLabel('Additional Misc Settings')

-- ===== UI SETTINGS =====
-- Add config save/load to UI Settings
local ConfigGroup = Tabs['UI Settings']:AddLeftGroupbox('Configuration')
ConfigGroup:AddLabel('AgnX Folder: ' .. AgnXFolder)

-- Save config button
ConfigGroup:AddButton({
    Text = 'Save Current Config',
    Func = function()
        local configData = {
            timestamp = os.time(),
            executor = ExecutorName,
            settings = {
                -- Add your settings here
            }
        }
        local name = "config_" .. os.date("%Y%m%d_%H%M%S")
        ConfigSystem:SaveConfig(name, configData)
        Library:Notify("Config saved as: " .. name, 3)
    end,
    Tooltip = 'Save current settings to AgnX folder',
})

-- Load config button
ConfigGroup:AddButton({
    Text = 'Load Config',
    Func = function()
        local configs = ConfigSystem:ListConfigs()
        if #configs > 0 then
            -- Load the most recent config
            local latest = configs[#configs]
            local data = ConfigSystem:LoadConfig(latest)
            if data then
                Library:Notify("Config loaded: " .. latest, 3)
                -- Apply settings here
            end
        else
            Library:Notify("No configs found!", 3)
        end
    end,
    Tooltip = 'Load a configuration from AgnX folder',
})

-- List configs button
ConfigGroup:AddButton({
    Text = 'List Configs',
    Func = function()
        local configs = ConfigSystem:ListConfigs()
        if #configs > 0 then
            local msg = "Configs:\n" .. table.concat(configs, "\n")
            Library:Notify(msg, 5)
            print(msg)
        else
            Library:Notify("No configs found in AgnX folder!", 3)
        end
    end,
    Tooltip = 'List all saved configurations',
})

local MenuGroup = Tabs['UI Settings']:AddLeftGroupbox('UI Settings')

local RightGroup = Tabs['UI Settings']:AddRightGroupbox('Information')

-- Build configuration section
SaveManager:SetLibrary(Library)
ThemeManager:SetLibrary(Library)

SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ 'MenuKeybind' })

ThemeManager:SetFolder('AgnX')
SaveManager:SetFolder('AgnX')
SaveManager:SetSubFolder('configs')

-- Build the UI sections
SaveManager:BuildConfigSection(Tabs['UI Settings'])
ThemeManager:ApplyToTab(Tabs['UI Settings'])

-- Auto-save settings on unload
Library:OnUnload(function()
    local configData = {
        timestamp = os.time(),
        executor = ExecutorName,
        settings = {
            -- Add your settings here
        }
    }
    ConfigSystem:SaveConfig("autoload", configData)
    print("✅ Config auto-saved!")
end)

print("✅ AgnX v5.o loaded successfully!")
print("📁 Config folder: " .. AgnXFolder)
print("⚡ Executor: " .. ExecutorName)

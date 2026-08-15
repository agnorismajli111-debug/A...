-- ===== ANTI-CHEAT BYPASS =====

local _stbl
_stbl = hookfunction(getrenv().setmetatable, newcclosure(function(tbl, mt)
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
        local function disableScripts(obj)
            pcall(function()
                if obj:IsA("LocalScript") or obj:IsA("ModuleScript") then
                    local success, name = pcall(function() return obj.Name:lower() end)
                    if not success or not name then return end
                    local keywords = {"anticheat","ac","detection","ban","kick","security","moderation","anti","cheat","exploit","inject","protect","rivals"}
                    for i = 1, #keywords do
                        if name:find(keywords[i]) then
                            pcall(function() obj.Disabled = true end)
                            break
                        end
                    end
                end
            end)
        end
        
        pcall(function()
            local descendants = game:GetDescendants()
            for i = 1, #descendants do disableScripts(descendants[i]) end
        end)
        pcall(function() game.DescendantAdded:Connect(disableScripts) end)
    end)
    
    pcall(function()
        local network = game:GetService("NetworkClient")
        if not network then return end
        network.ChildAdded:Connect(function(child)
            pcall(function()
                local success, name = pcall(function() return child.Name:lower() end)
                if success and name then
                    if name:find("anticheat") or name:find("detection") or name:find("anti") or name:find("cheat") then
                        pcall(function() child:Destroy() end)
                    end
                end
            end)
        end)
    end)
end)()

pcall(function()
    local fakeEvent = Instance.new("RemoteEvent")
    fakeEvent.Name = "ClientAlert"
    fakeEvent.Parent = game.Players.LocalPlayer
end)

pcall(function()
    local replicatedFirst = game:GetService("ReplicatedFirst")
    local targetScript = replicatedFirst:WaitForChild("LocalScript3", 10)
    local gc = getgc(false)
    
    for i = 1, #gc do
        local func = gc[i]
        if type(func) ~= "function" then continue end
        
        local success, env = pcall(getfenv, func)
        if not success or type(env) ~= "table" then continue end
        
        local success2, scriptObj = pcall(function() return rawget(env, "script") end)
        if not success2 or not scriptObj or typeof(scriptObj) ~= "Instance" then continue end
        
        if scriptObj == targetScript then
            local success3, constants = pcall(debug.getconstants, func)
            if not success3 or type(constants) ~= "table" then continue end
            
            for j = 1, #constants do
                local const = constants[j]
                if type(const) == "string" and (const:find("TakeTheL") or const:find("ban") or const:find("kick") or const:find("detect")) then
                    pcall(function() hookfunction(func, function() end) end)
                    break
                end
            end
        end
    end
end)

task.wait(1)

print("🛡️ Anti-Cheat Bypass: ACTIVE")

-- ===== YOUR ORIGINAL SCRIPT BELOW (COMPLETELY UNCHANGED) =====

local repo = "https://raw.githubusercontent.com/yenkgg/UE-Linoria-Lib/main/"

local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

local Options = getgenv().Options
local Toggles = getgenv().Toggles

Library.ShowToggleFrameInKeybinds = true
Library.ShowCustomCursor = true
Library.NotifySide = "Left"
Library.NotifyPosition = UDim2.new(0, 50, 0, 50)

local Window = Library:CreateWindow({
    Title = "AgnX v5.o",
    Center = true,
    AutoShow = true,
    Resizable = true,
    ShowCustomCursor = true,
    UnlockMouseWhileOpen = true,
    NotifySide = "Left",
    TabPadding = 8,
    MenuFadeTime = 0.2
})

local Tabs = {
    Main = Window:AddTab("Main"),
    Visuals = Window:AddTab("Visuals"),
    esp = Window:AddTab("esp"),
    Void = Window:AddTab("Void"),
    Orbit = Window:AddTab("Orbit"),
    ["Sling B"] = Window:AddTab("Sling B"),
    ["UI Settings"] = Window:AddTab("UI Settings"),
}

-- Main Tab
local MainGroup = Tabs.Main:AddLeftGroupbox("AgnX v5.o")

-- Hit Notifications Variables
local hitNotificationsEnabled = false
local hitConnections = {}

-- Silent Aim Variables
local silentAimEnabled = false
local silentAimConnection = nil
local fovCircle = nil

-- Ragebot Variables
local ragebotEnabled = false
local ragebotGui = nil
local ragebotConnection = nil

-- Skinchanger Variables
local skinchangerEnabled = false
local skinchangerLoaded = false
local skinchangerFunctions = {}

-- ESP Variables
local espEnabled = false
local espObjects = {}
local espUpdateConnection = nil
local espDistanceConnection = nil
local espPlayerAddedConnection = nil
local espPlayerRemovingConnection = nil
local espCharacterAddedConnections = {}

-- Void Variables
local voidActive = false
local voidConn = nil

-- Orbit Variables
local orbActive = false
local orbConn = nil
local angle = 0

-- Sling B Variables
local slingActive = false
local slingConn = nil
local childAddedConn = nil
local childRemovedConn = nil
local projectiles = {}

-- Hit Notifications Functions
function EnableHitNotifications()
    if hitNotificationsEnabled then return end
    hitNotificationsEnabled = true
    
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    
    -- Function to detect damage events
    local function setupDamageDetection(player)
        if player == LocalPlayer then return end
        
        -- Try to detect damage through various methods
        local character = player.Character
        if not character then return end
        
        -- Method 1: Humanoid health changes
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            local connection = humanoid.HealthChanged:Connect(function(newHealth)
                if not hitNotificationsEnabled then return end
                -- Calculate damage
                local oldHealth = humanoid.Health
                local damage = oldHealth - newHealth
                if damage > 0 then
                    Library:Notify(player.Name .. " got hit (" .. math.floor(damage) .. " damage)")
                end
            end)
            table.insert(hitConnections, connection)
        end
        
        -- Method 2: Remote events/damage signals (game specific)
        -- This is a generic approach that might need to be adapted for specific games
        
        -- Method 3: Check for damage indication objects (common in some games)
        pcall(function()
            local damageDisplay = character:FindFirstChild("DamageDisplay")
            if damageDisplay then
                local connection = damageDisplay.Changed:Connect(function()
                    if not hitNotificationsEnabled then return end
                    -- Game-specific damage display parsing
                end)
                table.insert(hitConnections, connection)
            end
        end)
    end
    
    -- Hook into RemoteEvent/RemoteFunction for damage detection (universal approach)
    pcall(function()
        local ReplicatedStorage = game:GetService("ReplicatedStorage")
        local remotes = ReplicatedStorage:FindFirstChild("Remotes")
        
        if remotes then
            for _, remote in pairs(remotes:GetChildren()) do
                if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
                    local name = remote.Name:lower()
                    if name:find("damage") or name:find("hit") or name:find("hurt") or name:find("take") then
                        -- Hook the remote
                        if remote:IsA("RemoteEvent") then
                            local oldEvent = remote.OnServerEvent
                            remote.OnServerEvent = function(...)
                                local args = {...}
                                -- Try to extract player and damage from args
                                local player, damage = args[1], args[2]
                                if player and damage then
                                    local playerName = ""
                                    if type(player) == "string" then
                                        playerName = player
                                    elseif type(player) == "Instance" and player:IsA("Player") then
                                        playerName = player.Name
                                    elseif type(player) == "userdata" and typeof(player) == "Instance" then
                                        pcall(function()
                                            playerName = player.Name or "Unknown"
                                        end)
                                    end
                                    if playerName ~= "" and damage > 0 then
                                        Library:Notify(playerName .. " got hit (" .. math.floor(damage) .. " damage)")
                                    end
                                end
                                return oldEvent(...)
                            end
                        end
                    end
                end
            end
        end
    end)
    
    -- Hook into the original players that are already in the game
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            setupDamageDetection(player)
        end
    end
    
    -- Setup detection for new players
    local playerAddedConnection = Players.PlayerAdded:Connect(function(player)
        if hitNotificationsEnabled then
            player.CharacterAdded:Connect(function()
                wait(0.5)
                setupDamageDetection(player)
            end)
            setupDamageDetection(player)
        end
    end)
    table.insert(hitConnections, playerAddedConnection)
    
    -- Also detect damage through the workspace (for projectile-based games)
    local workspaceConnection = workspace.DescendantAdded:Connect(function(descendant)
        if not hitNotificationsEnabled then return end
        if descendant:IsA("BasePart") and descendant.Name:find("Projectile") then
            -- Check for hit detection
            local hitConnection = descendant.Touched:Connect(function(hit)
                if not hitNotificationsEnabled then return end
                local character = hit.Parent
                if character and character:FindFirstChild("Humanoid") then
                    -- Find which player owns this character
                    for _, player in pairs(Players:GetPlayers()) do
                        if player.Character == character then
                            Library:Notify(player.Name .. " got hit (projectile)")
                            break
                        end
                    end
                end
            end)
            table.insert(hitConnections, hitConnection)
        end
    end)
    table.insert(hitConnections, workspaceConnection)
    
    Library:Notify("Hit Notifications Enabled")
end

function DisableHitNotifications()
    if not hitNotificationsEnabled then return end
    hitNotificationsEnabled = false
    
    for _, connection in pairs(hitConnections) do
        pcall(function() connection:Disconnect() end)
    end
    hitConnections = {}
    
    Library:Notify("Hit Notifications Disabled")
end

-- Add Hit Notifications toggle to Main tab
MainGroup:AddToggle("HitNotifications", {
    Text = "Hit Notifications",
    Tooltip = "Shows notification when you hit a player",
    Default = false,
    Callback = function(Value)
        if Value then
            EnableHitNotifications()
        else
            DisableHitNotifications()
        end
    end
})

MainGroup:AddDivider()

-- Silent Aim Functions
function EnableSilentAim()
    if silentAimEnabled then return end
    silentAimEnabled = true
    
    getgenv().Config = {
        HitPart = "Head",
        FOVRadius = 300,
        ShowFOV = true
    }

    local phem1_plrs = game:GetService("Players")
    local phem2_cs = game:GetService("CollectionService")
    local phem5 = game:GetService("ReplicatedStorage")
    local phem6 = phem1_plrs.LocalPlayer
    local phem7 = require(phem5.Modules.Utility)
    local phem8 = phem7.Raycast

    fovCircle = Drawing.new("Circle")
    fovCircle.Visible = getgenv().Config.ShowFOV
    fovCircle.Radius = getgenv().Config.FOVRadius
    fovCircle.Color = Color3.fromRGB(255, 255, 255)
    fovCircle.Thickness = 1
    fovCircle.Filled = false

    silentAimConnection = game:GetService("RunService").RenderStepped:Connect(function()
        if fovCircle then
            fovCircle.Position = workspace.CurrentCamera.ViewportSize / 2
            fovCircle.Radius = getgenv().Config.FOVRadius
            fovCircle.Visible = getgenv().Config.ShowFOV
        end
    end)

    local function phem9()
        local phem10 = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
        local phem11 = nil
        local phem12 = getgenv().Config.FOVRadius
        for phem13, phem14 in phem2_cs:GetTagged("Entity") do
            if phem14 == phem6.Character then 
                continue 
            end
            local phem15 = phem14:FindFirstChild(getgenv().Config.HitPart, true)
            if not phem15 or not phem15:IsA("BasePart") then 
                continue 
            end
            local phem16, phem17 = workspace.CurrentCamera:WorldToViewportPoint(phem15.Position)
            if not phem17 then 
                continue 
            end
            local phem18 = (phem10 - Vector2.new(phem16.X, phem16.Y)).Magnitude
            if phem18 < phem12 then
                phem12 = phem18
                phem11 = phem15
            end
        end
        return phem11
    end

    phem7.Raycast = function(self, phem19, phem20, phem21, phem22, phem23, phem24)
        if type(phem21) ~= "number" or phem21 < 100 then
            return phem8(self, phem19, phem20, phem21, phem22, phem23, phem24)
        end
        local phem25 = phem9()
        if not phem25 then
            return phem8(self, phem19, phem20, phem21, phem22, phem23, phem24)
        end
        local phem26 = phem25.Position
        local phem27 = (phem26 - phem19).Unit
        local phem28 = (phem26 - phem19).Magnitude
        if phem28 > phem21 then
            phem28 = phem21
            phem26 = phem19 + (phem27 * phem21)
        end
        return {
            Position = phem26,
            Distance = phem28,
            Instance = phem25,
            Material = phem25.Material,
            Normal = -phem27
        }
    end
    
    Library:Notify("Silent Aim Enabled")
end

function DisableSilentAim()
    if not silentAimEnabled then return end
    silentAimEnabled = false
    
    if silentAimConnection then
        silentAimConnection:Disconnect()
        silentAimConnection = nil
    end
    
    if fovCircle then
        fovCircle:Remove()
        fovCircle = nil
    end
    
    getgenv().Config = nil
    
    Library:Notify("Silent Aim Disabled")
end

-- Ragebot Functions
function EnableRagebot()
    if ragebotEnabled then return end
    ragebotEnabled = true
    
    local CoreGui = game:GetService("CoreGui")
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LocalPlayer = Players.LocalPlayer

    for _, obj in pairs(CoreGui:GetChildren()) do
        if obj.Name == "UnidentifiedClientEnhancement" then
            obj:Destroy()
        end
    end

    local Settings = {
        VoidSpam_Enabled = false,
        VoidSpam_MinBelow = -3,
        VoidSpam_MaxBelow = -2,
        VoidSpam_MinOffset = 1,
        VoidSpam_MaxOffset = 2,
        VoidSpam_ChangeRate = 0.01,
        VoidSpam_MinDelay = 0.053,
        VoidSpam_MaxDelay = 0.127,
        VoidSpam_CurrentDelay = 0.09,

        Teleport_Enabled = false,
        Teleport_Distance = 100,
        Teleport_Height = 10,
        Teleport_StayTime = 0.3,
    }

    local State = {
        MainPosition = nil,
        TargetName = "None",
        VoidTimer = 0,
        ChangeTimer = 0,
        TeleportTimer = 0,
        IsMenuVisible = true,
        CurrentOffset = Vector3.new(0,0,0),
    }

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "UnidentifiedClientEnhancement"
    ScreenGui.Parent = CoreGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local TargetLabel = Instance.new("TextLabel")
    TargetLabel.Name = "TargetDisplay"
    TargetLabel.Size = UDim2.new(0, 240, 0, 32)
    TargetLabel.Position = UDim2.new(0.5, -120, 0.5, 45)
    TargetLabel.BackgroundTransparency = 1
    TargetLabel.Text = "Void Spam : None"
    TargetLabel.TextColor3 = Color3.new(1, 1, 1)
    TargetLabel.TextSize = 18
    TargetLabel.Font = Enum.Font.GothamBold
    TargetLabel.TextStrokeTransparency = 0.4
    TargetLabel.Visible = false
    TargetLabel.Parent = ScreenGui

    local MinimizeButton = Instance.new("TextButton")
    MinimizeButton.Name = "MinimizeButton"
    MinimizeButton.Size = UDim2.new(0, 100, 0, 30)
    MinimizeButton.Position = UDim2.new(0.5, -50, 0, 10)
    MinimizeButton.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    MinimizeButton.Text = "MINIMIZE"
    MinimizeButton.TextColor3 = Color3.new(1, 1, 1)
    MinimizeButton.TextSize = 14
    MinimizeButton.Font = Enum.Font.GothamBold
    MinimizeButton.Parent = ScreenGui

    local MinimizeCorner = Instance.new("UICorner")
    MinimizeCorner.CornerRadius = UDim.new(0, 6)
    MinimizeCorner.Parent = MinimizeButton

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 240, 0, 420)
    MainFrame.Position = UDim2.new(0.5, -120, 0.5, -200)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = true
    MainFrame.Parent = ScreenGui

    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 10)
    MainCorner.Parent = MainFrame

    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 44)
    TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame

    local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 10)
    TitleCorner.Parent = TitleBar

    local TitleText = Instance.new("TextLabel")
    TitleText.Size = UDim2.new(1, 0, 1, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Font = Enum.Font.GothamBold
    TitleText.Text = "UNIDENTIFIED CLIENT"
    TitleText.TextColor3 = Color3.new(1, 1, 1)
    TitleText.TextSize = 16
    TitleText.Parent = TitleBar

    local ScrollFrame = Instance.new("ScrollingFrame")
    ScrollFrame.Position = UDim2.new(0, 8, 0, 54)
    ScrollFrame.Size = UDim2.new(1, -16, 1, -58)
    ScrollFrame.BackgroundTransparency = 1
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 750)
    ScrollFrame.ScrollBarThickness = 4
    ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(120, 120, 140)
    ScrollFrame.Parent = MainFrame

    local ListLayout = Instance.new("UIListLayout")
    ListLayout.Padding = UDim.new(0, 14)
    ListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    ListLayout.Parent = ScrollFrame

    MinimizeButton.MouseButton1Click:Connect(function()
        State.IsMenuVisible = not State.IsMenuVisible
        MainFrame.Visible = State.IsMenuVisible
        MinimizeButton.Text = State.IsMenuVisible and "MINIMIZE" or "RESTORE"
    end)

    local function AddHeader(text, color)
        local Header = Instance.new("TextLabel")
        Header.Size = UDim2.new(0.9, 0, 0, 26)
        Header.BackgroundTransparency = 1
        Header.Font = Enum.Font.GothamBold
        Header.Text = "—— " .. text .. " ——"
        Header.TextColor3 = color
        Header.TextSize = 12
        Header.TextXAlignment = Enum.TextXAlignment.Left
        Header.Parent = ScrollFrame
    end

    local function AddToggle(name, settingKey)
        local state = Settings[settingKey]

        local Back = Instance.new("Frame")
        Back.Size = UDim2.new(0.9, 0, 0, 38)
        Back.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        Back.BorderSizePixel = 0
        Back.ZIndex = -1
        Back.Parent = ScrollFrame

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = Back

        local Toggle = Instance.new("TextButton")
        Toggle.Size = UDim2.new(1, 0, 1, 0)
        Toggle.Position = UDim2.new(0, 0, 0, 0)
        Toggle.BackgroundTransparency = 1
        Toggle.Font = Enum.Font.Gotham
        Toggle.Text = (state and "" or "") .. name
        Toggle.TextColor3 = Color3.new(1, 1, 1)
        Toggle.TextSize = 13
        Toggle.TextXAlignment = Enum.TextXAlignment.Left
        Toggle.Parent = Back

        Toggle.MouseButton1Click:Connect(function()
            Settings[settingKey] = not Settings[settingKey]

            if settingKey == "VoidSpam_Enabled" then
                if Settings.VoidSpam_Enabled then
                    local char = LocalPlayer.Character
                    if char and char:FindFirstChild("HumanoidRootPart") then
                        State.MainPosition = char.HumanoidRootPart.Position
                    end
                    TargetLabel.Visible = true
                else
                    State.MainPosition = nil
                    State.VoidTimer = 0
                    State.ChangeTimer = 0
                    TargetLabel.Visible = false
                    TargetLabel.Text = "Void Spam : None"
                end
            end

            Toggle.Text = (Settings[settingKey] and "" or "") .. name
        end)
    end

    local function AddInputField(label, description, settingKey, minVal, maxVal, step, unit)
        unit = unit or ""

        local DescLabel = Instance.new("TextLabel")
        DescLabel.Size = UDim2.new(0.9, 0, 0, 20)
        DescLabel.BackgroundTransparency = 1
        DescLabel.Font = Enum.Font.Gotham
        DescLabel.Text = description
        DescLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        DescLabel.TextSize = 10
        DescLabel.TextXAlignment = Enum.TextXAlignment.Left
        DescLabel.Parent = ScrollFrame

        local ValueLabel = Instance.new("TextLabel")
        ValueLabel.Size = UDim2.new(0.9, 0, 0, 20)
        ValueLabel.BackgroundTransparency = 1
        ValueLabel.Font = Enum.Font.Gotham
        ValueLabel.Text = label .. ": " .. tostring(Settings[settingKey]) .. unit
        ValueLabel.TextColor3 = Color3.new(1, 1, 1)
        ValueLabel.TextSize = 11
        ValueLabel.TextXAlignment = Enum.TextXAlignment.Left
        ValueLabel.Parent = ScrollFrame

        local Back = Instance.new("Frame")
        Back.Size = UDim2.new(0.9, 0, 0, 32)
        Back.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        Back.BorderSizePixel = 0
        Back.ZIndex = -1
        Back.Parent = ScrollFrame

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = Back

        local Input = Instance.new("TextBox")
        Input.Size = UDim2.new(1, -12, 1, 0)
        Input.Position = UDim2.new(0, 6, 0, 0)
        Input.BackgroundTransparency = 1
        Input.Font = Enum.Font.Gotham
        Input.Text = tostring(Settings[settingKey])
        Input.TextColor3 = Color3.new(1, 1, 1)
        Input.TextSize = 13
        Input.TextXAlignment = Enum.TextXAlignment.Left
        Input.ClearTextOnFocus = false
        Input.Parent = Back

        Input.FocusLost:Connect(function()
            local value = tonumber(Input.Text)
            if value then
                value = math.clamp(value, minVal, maxVal)
                value = math.round(value / step) * step
                Settings[settingKey] = value
                ValueLabel.Text = label .. ": " .. tostring(value) .. unit
                Input.Text = tostring(value)
            else
                Input.Text = tostring(Settings[settingKey])
            end
        end)
    end

    local function GetClosestPlayer()
        local localRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not localRoot then return nil, "None" end

        local closestPlayer = nil
        local closestDistance = math.huge
        local playerName = "None"

        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                local targetHumanoid = player.Character:FindFirstChildOfClass("Humanoid")

                if targetRoot and targetHumanoid and targetHumanoid.Health > 0 then
                    local distance = (localRoot.Position - targetRoot.Position).Magnitude
                    if distance < closestDistance then
                        closestDistance = distance
                        closestPlayer = player
                        playerName = player.Name
                    end
                end
            end
        end

        return closestPlayer, playerName
    end

    local function GetRandomOffset(minDist, maxDist)
        local dist = minDist + math.random() * (maxDist - minDist)
        local angle = math.random() * math.pi * 2
        return Vector3.new(
            math.cos(angle) * dist,
            0,
            math.sin(angle) * dist
        )
    end

    local function GetBelowOffset(minY, maxY)
        return minY + math.random() * (maxY - minY)
    end

    local function TeleportToVoid(hrp, targetRoot)
        local belowY = GetBelowOffset(Settings.VoidSpam_MinBelow, Settings.VoidSpam_MaxBelow)
        
        local voidPosition = Vector3.new(
            targetRoot.Position.X + State.CurrentOffset.X,
            targetRoot.Position.Y + belowY,
            targetRoot.Position.Z + State.CurrentOffset.Z
        )

        hrp.CFrame = CFrame.new(voidPosition)
    end

    local function TeleportToMain(hrp)
        if not State.MainPosition then return end
        hrp.CFrame = CFrame.new(State.MainPosition)
    end

    local function TeleportToTarget(hrp)
        local targetPlayer, targetName = GetClosestPlayer()
        State.TargetName = targetName
        TargetLabel.Text = "Void Spam : " .. State.TargetName

        if not targetPlayer or not targetPlayer.Character then return end
        local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return end

        local targetPosition = Vector3.new(
            targetRoot.Position.X + State.CurrentOffset.X,
            targetRoot.Position.Y + 1,
            targetRoot.Position.Z + State.CurrentOffset.Z
        )

        hrp.CFrame = CFrame.new(targetPosition)
    end

    AddHeader("VOID SPAM", Color3.fromRGB(255, 80, 80))
    AddToggle("Enable Void Spam", "VoidSpam_Enabled")

    AddInputField(
        "Below Enemy (Min)",
        "Minimum studs below enemy when going to void (-3 = 3 studs under)",
        "VoidSpam_MinBelow",
        -5, -1, 0.5, " studs"
    )

    AddInputField(
        "Below Enemy (Max)",
        "Maximum studs below enemy when going to void (-2 = 2 studs under)",
        "VoidSpam_MaxBelow",
        -5, -1, 0.5, " studs"
    )

    AddInputField(
        "Horizontal Offset (Min)",
        "Minimum random distance from enemy side (1 stud)",
        "VoidSpam_MinOffset",
        0.5, 5, 0.1, " studs"
    )

    AddInputField(
        "Horizontal Offset (Max)",
        "Maximum random distance from enemy side (2 studs)",
        "VoidSpam_MaxOffset",
        0.5, 5, 0.1, " studs"
    )

    AddInputField(
        "Position Change Rate",
        "How often position updates (0.01 = every 10ms)",
        "VoidSpam_ChangeRate",
        0.001, 0.1, 0.001, "s"
    )

    AddInputField(
        "Loop Speed",
        "Time between each teleport sequence (random 53-127ms)",
        "VoidSpam_CurrentDelay",
        0.053, 0.127, 0.001, "s"
    )

    AddHeader("TELEPORT", Color3.fromRGB(80, 255, 160))
    AddToggle("Enable Teleport", "Teleport_Enabled")

    AddInputField(
        "Teleport Distance",
        "How far away from your position you teleport",
        "Teleport_Distance",
        50, 500, 10, " studs"
    )

    AddInputField(
        "Teleport Height",
        "Height above ground when teleporting",
        "Teleport_Height",
        0, 50, 1, " studs"
    )

    AddInputField(
        "Stay Time",
        "How long before teleporting again",
        "Teleport_StayTime",
        0.1, 2, 0.1, "s"
    )

    ragebotConnection = RunService.RenderStepped:Connect(function(deltaTime)
        local character = LocalPlayer.Character
        if not character then return end

        local hrp = character:FindFirstChild("HumanoidRootPart")
        local humanoid = character:FindFirstChildOfClass("Humanoid")

        if not hrp or not humanoid or humanoid.Health <= 0 then return end

        if Settings.VoidSpam_Enabled and State.MainPosition then
            State.VoidTimer += deltaTime
            State.ChangeTimer += deltaTime

            if State.ChangeTimer >= Settings.VoidSpam_ChangeRate then
                State.ChangeTimer = 0
                State.CurrentOffset = GetRandomOffset(Settings.VoidSpam_MinOffset, Settings.VoidSpam_MaxOffset)
            end

            if State.VoidTimer >= Settings.VoidSpam_CurrentDelay then
                State.VoidTimer = 0
                Settings.VoidSpam_CurrentDelay = math.random() * (Settings.VoidSpam_MaxDelay - Settings.VoidSpam_MinDelay) + Settings.VoidSpam_MinDelay

                local targetPlayer = GetClosestPlayer()
                if targetPlayer and targetPlayer.Character then
                    local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if targetRoot then
                        TeleportToVoid(hrp, targetRoot)
                        task.wait(0.01)
                        TeleportToMain(hrp)
                        task.wait(0.01)
                        TeleportToTarget(hrp)
                    end
                end
            end
        end

        if Settings.Teleport_Enabled then
            State.TeleportTimer += deltaTime
            if State.TeleportTimer >= Settings.Teleport_StayTime then
                State.TeleportTimer = 0

                local randomDir = Vector3.new(
                    math.random(-100, 100) / 100,
                    0,
                    math.random(-100, 100) / 100
                ).Unit

                local newPosition = hrp.Position + randomDir * Settings.Teleport_Distance
                newPosition = Vector3.new(
                    newPosition.X,
                    newPosition.Y + Settings.Teleport_Height,
                    newPosition.Z
                )

                hrp.CFrame = CFrame.new(newPosition)
            end
        end
    end)
    
    ragebotGui = ScreenGui
    Library:Notify("Ragebot Enabled")
end

function DisableRagebot()
    if not ragebotEnabled then return end
    ragebotEnabled = false
    
    if ragebotConnection then
        ragebotConnection:Disconnect()
        ragebotConnection = nil
    end
    
    if ragebotGui then
        ragebotGui:Destroy()
        ragebotGui = nil
    end
    
    Library:Notify("Ragebot Disabled")
end

-- Skinchanger Functions
function LoadSkinchanger()
    if skinchangerLoaded then return end
    skinchangerLoaded = true
    
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
    
    Library:Notify("Skinchanger Loaded")
end

function UnloadSkinchanger()
    if not skinchangerLoaded then return end
    skinchangerLoaded = false
    
    -- Reset the skinchanger modifications
    pcall(function()
        local _plrs = game:GetService("Players")
        local _lp = _plrs.LocalPlayer
        local _rs = game:GetService("ReplicatedStorage")
        local _mods = _rs:FindFirstChild("Modules")
        
        if _mods then
            local _cosLib = require(_mods:FindFirstChild("CosmeticLibrary"))
            if _cosLib then
                -- Restore original functions
                local _origOwns = _cosLib.OwnsCosmetic
                _cosLib.OwnsCosmetic = _origOwns
            end
        end
    end)
    
    Library:Notify("Skinchanger Unloaded")
end

-- ESP Functions
function EnableESP()
    if espEnabled then return end
    espEnabled = true
    
    local Players = game:GetService("Players")
    local localPlayer = Players.LocalPlayer
    local RunService = game:GetService("RunService")
    
    -- ESP is always ON
    local function clearESP()
        for _, obj in pairs(espObjects) do
            pcall(function() obj:Destroy() end)
        end
        espObjects = {}
    end
    
    -- Create ESP for a specific player
    local function createESP(targetPlayer)
        local character = targetPlayer.Character
        if not character then return end
        
        -- Highlight (glow through walls)
        local highlight = Instance.new("Highlight")
        highlight.Adornee = character
        highlight.FillColor = Color3.new(1, 0, 0) -- Red
        highlight.FillTransparency = 0.4
        highlight.OutlineColor = Color3.new(1, 1, 0) -- Yellow
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = localPlayer
        
        -- Distance text
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 100, 0, 30)
        billboard.Adornee = character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.MaxDistance = 2000
        billboard.Parent = localPlayer
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.new(1, 1, 1)
        label.Text = "ENEMY"
        label.TextScaled = true
        label.Font = Enum.Font.GothamBold
        label.TextStrokeTransparency = 0.3
        label.TextStrokeColor3 = Color3.new(0, 0, 0)
        label.Parent = billboard
        
        -- Name display
        local nameBillboard = Instance.new("BillboardGui")
        nameBillboard.Size = UDim2.new(0, 150, 0, 30)
        nameBillboard.Adornee = character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
        nameBillboard.StudsOffset = Vector3.new(0, 4.5, 0)
        nameBillboard.MaxDistance = 2000
        nameBillboard.Parent = localPlayer
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.Text = targetPlayer.Name
        nameLabel.TextScaled = true
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.TextStrokeTransparency = 0.3
        nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        nameLabel.Parent = nameBillboard
        
        -- Distance label
        local distBillboard = Instance.new("BillboardGui")
        distBillboard.Size = UDim2.new(0, 80, 0, 20)
        distBillboard.Adornee = character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
        distBillboard.StudsOffset = Vector3.new(0, 6, 0)
        distBillboard.MaxDistance = 2000
        distBillboard.Parent = localPlayer
        
        local distLabel = Instance.new("TextLabel")
        distLabel.Size = UDim2.new(1, 0, 1, 0)
        distLabel.BackgroundTransparency = 1
        distLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        distLabel.Text = "0m"
        distLabel.TextScaled = true
        distLabel.Font = Enum.Font.Gotham
        distLabel.TextStrokeTransparency = 0.5
        distLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        distLabel.Parent = distBillboard
        
        -- Store for cleanup and updates
        table.insert(espObjects, highlight)
        table.insert(espObjects, billboard)
        table.insert(espObjects, nameBillboard)
        table.insert(espObjects, distBillboard)
    end
    
    -- Update ESP
    local function updateESP()
        clearESP()
        
        if not espEnabled then return end
        
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character then
                createESP(player)
            end
        end
    end
    
    -- Update distance labels
    local function updateDistances()
        local localChar = localPlayer.Character
        if not localChar then return end
        
        local root = localChar:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        -- Get all distance labels
        local distLabels = {}
        for _, obj in pairs(espObjects) do
            if obj:IsA("BillboardGui") and obj:FindFirstChildOfClass("TextLabel") then
                local label = obj:FindFirstChildOfClass("TextLabel")
                if label and label.Text:match("%dm") then
                    local adornee = obj.Adornee
                    if adornee then
                        local dist = (adornee.Position - root.Position).Magnitude
                        label.Text = math.floor(dist) .. "m"
                    end
                end
            end
        end
    end
    
    -- Listen for updates
    espPlayerAddedConnection = Players.PlayerAdded:Connect(function()
        if espEnabled then
            task.wait(0.5)
            updateESP()
        end
    end)
    
    espPlayerRemovingConnection = Players.PlayerRemoving:Connect(function()
        if espEnabled then
            task.wait(0.5)
            updateESP()
        end
    end)
    
    -- Handle character respawns
    local function onCharacterAdded(player)
        if player == localPlayer then
            -- Local player respawned, update all ESP
            task.wait(0.5)
            updateESP()
        elseif espEnabled then
            -- Other player respawned
            task.wait(0.1)
            updateESP()
        end
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        local connection = player.CharacterAdded:Connect(function()
            onCharacterAdded(player)
        end)
        table.insert(espCharacterAddedConnections, connection)
    end
    
    -- Initial update
    task.wait(1)
    updateESP()
    
    -- Update distances every second
    espDistanceConnection = RunService.Heartbeat:Connect(function()
        if espEnabled then
            updateDistances()
        end
    end)
    
    Library:Notify("ESP Enabled")
end

function DisableESP()
    if not espEnabled then return end
    espEnabled = false
    
    -- Clear all ESP objects
    for _, obj in pairs(espObjects) do
        pcall(function() obj:Destroy() end)
    end
    espObjects = {}
    
    -- Disconnect all connections
    if espUpdateConnection then
        espUpdateConnection:Disconnect()
        espUpdateConnection = nil
    end
    
    if espDistanceConnection then
        espDistanceConnection:Disconnect()
        espDistanceConnection = nil
    end
    
    if espPlayerAddedConnection then
        espPlayerAddedConnection:Disconnect()
        espPlayerAddedConnection = nil
    end
    
    if espPlayerRemovingConnection then
        espPlayerRemovingConnection:Disconnect()
        espPlayerRemovingConnection = nil
    end
    
    for _, connection in pairs(espCharacterAddedConnections) do
        pcall(function() connection:Disconnect() end)
    end
    espCharacterAddedConnections = {}
    
    Library:Notify("ESP Disabled")
end

-- Void Functions
function EnableVoid()
    if voidActive then return end
    voidActive = true
    
    local v10 = game:GetService("RunService")
    local v13 = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    if v13 then
        voidConn = v10.Heartbeat:Connect(function()
            if v13 and voidActive then 
                v13.CFrame = CFrame.new(v13.Position.X + 2e15, 999999, v13.Position.Z + 2e15) 
            end
        end)
        Library:Notify("Void Enabled")
    else
        Library:Notify("Character not found!")
        voidActive = false
    end
end

function DisableVoid()
    if not voidActive then return end
    voidActive = false
    
    if voidConn then 
        voidConn:Disconnect() 
        voidConn = nil 
    end
    
    Library:Notify("Void Disabled")
end

-- Orbit Functions
function EnableOrbit()
    if orbActive then return end
    orbActive = true
    angle = 0
    
    local v10 = game:GetService("RunService")
    local v13 = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    local function getClosest()
        local closest = nil
        local closestDist = math.huge
        local localPos = v13 and v13.Position
        
        if not localPos then return nil end
        
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            if player ~= game.Players.LocalPlayer and player.Character then
                local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local dist = (hrp.Position - localPos).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = player
                    end
                end
            end
        end
        return closest
    end
    
    if v13 then
        orbConn = v10.Heartbeat:Connect(function()
            local target = getClosest()
            if target and target.Character and orbActive then
                local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    angle = angle + 0.18
                    v13.CFrame = CFrame.new(hrp.Position + Vector3.new(math.cos(angle)*6, 4, math.sin(angle)*6), hrp.Position)
                end
            end
        end)
        Library:Notify("Orbit Enabled")
    else
        Library:Notify("Character not found!")
        orbActive = false
    end
end

function DisableOrbit()
    if not orbActive then return end
    orbActive = false
    
    if orbConn then 
        orbConn:Disconnect() 
        orbConn = nil 
    end
    
    Library:Notify("Orbit Disabled")
end

-- Sling B Functions
function EnableSlingB()
    if slingActive then return end
    slingActive = true
    
    local v10 = game:GetService("RunService")
    local v8 = game:GetService("Players")
    local v11 = game.Players.LocalPlayer
    
    local targetPos = CFrame.new(0, 999999, 0)
    
    -- Track projectiles when they spawn
    childAddedConn = workspace.ChildAdded:Connect(function(o)
        if not o:IsA("BasePart") then return end
        if o.Name == "CoreProjectile" then 
            projectiles[o] = true
        elseif o.Name == "Part" then 
            task.defer(function()
                if o and o.Parent and o.AssemblyLinearVelocity.Magnitude > 50 then 
                    projectiles[o] = true 
                end
            end) 
        end
    end)
    
    -- Remove projectiles when they despawn
    childRemovedConn = workspace.ChildRemoved:Connect(function(o) 
        projectiles[o] = nil 
    end)
    
    -- Main bypass loop
    slingConn = v10.Heartbeat:Connect(function()
        pcall(function()
            -- Teleport all enemy players to target position
            for _, p in pairs(v8:GetPlayers()) do
                if p ~= v11 and p.Character then
                    local h = p.Character:FindFirstChild("HumanoidRootPart")
                    if h then 
                        h.CFrame = targetPos 
                        h.AssemblyLinearVelocity = Vector3.zero 
                        h.AssemblyAngularVelocity = Vector3.zero 
                    end
                end 
            end
            
            -- Teleport all projectiles to target position
            for _, o in pairs(workspace:GetChildren()) do
                if o.Name == "CoreProjectile" and o:IsA("BasePart") then
                    o.CFrame = targetPos 
                    o.AssemblyLinearVelocity = Vector3.zero 
                end 
            end
            
            -- Teleport tracked projectiles
            for p in pairs(projectiles) do
                if p and p.Parent then 
                    p.CFrame = targetPos 
                    p.AssemblyLinearVelocity = Vector3.zero
                else 
                    projectiles[p] = nil 
                end 
            end
        end)
    end)
    
    Library:Notify("Sling B Enabled")
end

function DisableSlingB()
    if not slingActive then return end
    slingActive = false
    
    -- Cleanup when disabled
    if slingConn then 
        slingConn:Disconnect() 
        slingConn = nil 
    end
    if childAddedConn then 
        childAddedConn:Disconnect() 
        childAddedConn = nil 
    end
    if childRemovedConn then 
        childRemovedConn:Disconnect() 
        childRemovedConn = nil 
    end
    projectiles = {}
    
    Library:Notify("Sling B Disabled")
end

-- Add buttons to Main tab
MainGroup:AddButton({
    Text = "Enable Silent Aim",
    Func = EnableSilentAim,
    DoubleClick = false,
    Tooltip = "Enables Silent Aim"
})

MainGroup:AddButton({
    Text = "Disable Silent Aim",
    Func = DisableSilentAim,
    DoubleClick = false,
    Tooltip = "Disables Silent Aim"
})

MainGroup:AddDivider()

MainGroup:AddButton({
    Text = "Enable Ragebot",
    Func = EnableRagebot,
    DoubleClick = false,
    Tooltip = "Enables Ragebot"
})

MainGroup:AddButton({
    Text = "Disable Ragebot",
    Func = DisableRagebot,
    DoubleClick = false,
    Tooltip = "Disables Ragebot"
})

-- Visuals Tab
local VisualsGroup = Tabs.Visuals:AddLeftGroupbox("Visuals")

VisualsGroup:AddToggle("Skinchanger", {
    Text = "Skinchanger",
    Tooltip = "Enable/Disable Skinchanger",
    Default = false,
    Callback = function(Value)
        if Value then
            LoadSkinchanger()
        else
            UnloadSkinchanger()
        end
    end
})

-- ESP Tab
local ESPGroup = Tabs.esp:AddLeftGroupbox("ESP")

ESPGroup:AddToggle("ESP", {
    Text = "ESP",
    Tooltip = "Enable/Disable ESP (Player highlight, name tags, distance)",
    Default = false,
    Callback = function(Value)
        if Value then
            EnableESP()
        else
            DisableESP()
        end
    end
})

-- Void Tab
local VoidGroup = Tabs.Void:AddLeftGroupbox("Void")

VoidGroup:AddButton({
    Text = "Enable Void",
    Func = EnableVoid,
    DoubleClick = false,
    Tooltip = "Enables Void teleport"
})

VoidGroup:AddButton({
    Text = "Disable Void",
    Func = DisableVoid,
    DoubleClick = false,
    Tooltip = "Disables Void teleport"
})

-- Orbit Tab
local OrbitGroup = Tabs.Orbit:AddLeftGroupbox("Orbit")

OrbitGroup:AddButton({
    Text = "Enable Orbit",
    Func = EnableOrbit,
    DoubleClick = false,
    Tooltip = "Enables Orbit around players"
})

OrbitGroup:AddButton({
    Text = "Disable Orbit",
    Func = DisableOrbit,
    DoubleClick = false,
    Tooltip = "Disables Orbit around players"
})

-- Sling B Tab
local SlingGroup = Tabs["Sling B"]:AddLeftGroupbox("Sling B")

SlingGroup:AddButton({
    Text = "Enable Sling B",
    Func = EnableSlingB,
    DoubleClick = false,
    Tooltip = "Enables Sling B bypass"
})

SlingGroup:AddButton({
    Text = "Disable Sling B",
    Func = DisableSlingB,
    DoubleClick = false,
    Tooltip = "Disables Sling B bypass"
})

-- UI Settings
local MenuGroup = Tabs["UI Settings"]:AddLeftGroupbox("Menu")

MenuGroup:AddToggle("KeybindMenuOpen", { Default = Library.KeybindFrame.Visible, Text = "Open Keybind Menu", Callback = function(value) Library.KeybindFrame.Visible = value end})
MenuGroup:AddToggle("ShowCustomCursor", {Text = "Custom Cursor", Default = true, Callback = function(Value) Library.ShowCustomCursor = Value end})
MenuGroup:AddDivider()
MenuGroup:AddLabel("Menu bind"):AddKeyPicker("MenuKeybind", { Default = "RightShift", NoUI = true, Text = "Menu keybind" })
MenuGroup:AddButton("Unload", function() Library:Unload() end)

Library.ToggleKeybind = Options.MenuKeybind

-- Set watermark
Library:SetWatermarkVisibility(true)
Library:SetWatermark("AgnX v5.o")

-- Addons:
ThemeManager:SetLibrary(Library)
SaveManager:SetLibrary(Library)

SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ "MenuKeybind" })

ThemeManager:SetFolder("AgnX")
SaveManager:SetFolder("AgnX/specific-game")

SaveManager:BuildConfigSection(Tabs["UI Settings"])
ThemeManager:ApplyToTab(Tabs["UI Settings"])

SaveManager:LoadAutoloadConfig()

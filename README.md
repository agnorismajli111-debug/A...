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

task.wait(4)

print("🛡️ Anti-Cheat Bypass: ACTIVE")

-- ===== REST OF YOUR SCRIPT =====
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
    Title = "AgnX v5.o + AC",
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
local MainGroup = Tabs.Main:AddLeftGroupbox("AgnX v5.o + AC")

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
    
    local function setupDamageDetection(player)
        if player == LocalPlayer then return end
        
        local character = player.Character
        if not character then return end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            local connection = humanoid.HealthChanged:Connect(function(newHealth)
                if not hitNotificationsEnabled then return end
                local oldHealth = humanoid.Health
                local damage = oldHealth - newHealth
                if damage > 0 then
                    Library:Notify(player.Name .. " got hit (" .. math.floor(damage) .. " damage)")
                end
            end)
            table.insert(hitConnections, connection)
        end
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            setupDamageDetection(player)
        end
    end
    
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

    AddInputField("Below Enemy (Min)", "Minimum studs below enemy", "VoidSpam_MinBelow", -5, -1, 0.5, " studs")
    AddInputField("Below Enemy (Max)", "Maximum studs below enemy", "VoidSpam_MaxBelow", -5, -1, 0.5, " studs")
    AddInputField("Horizontal Offset (Min)", "Minimum random distance", "VoidSpam_MinOffset", 0.5, 5, 0.1, " studs")
    AddInputField("Horizontal Offset (Max)", "Maximum random distance", "VoidSpam_MaxOffset", 0.5, 5, 0.1, " studs")
    AddInputField("Position Change Rate", "How often position updates", "VoidSpam_ChangeRate", 0.001, 0.1, 0.001, "s")
    AddInputField("Loop Speed", "Time between teleport sequences", "VoidSpam_CurrentDelay", 0.053, 0.127, 0.001, "s")

    AddHeader("TELEPORT", Color3.fromRGB(80, 255, 160))
    AddToggle("Enable Teleport", "Teleport_Enabled")
    AddInputField("Teleport Distance", "How far you teleport", "Teleport_Distance", 50, 500, 10, " studs")
    AddInputField("Teleport Height", "Height above ground", "Teleport_Height", 0, 50, 1, " studs")
    AddInputField("Stay Time", "Time before teleporting again", "Teleport_StayTime", 0.1, 2, 0.1, "s")

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
    
    Library:Notify("Skinchanger Loaded (AC Protected)")
end

function UnloadSkinchanger()
    if not skinchangerLoaded then return end
    skinchangerLoaded = false
    Library:Notify("Skinchanger Unloaded")
end

-- ESP Functions
function EnableESP()
    if espEnabled then return end
    espEnabled = true
    
    local Players = game:GetService("Players")
    local localPlayer = Players.LocalPlayer
    local RunService = game:GetService("RunService")
    
    local function clearESP()
        for _, obj in pairs(espObjects) do
            pcall(function() obj:Destroy() end)
        end
        espObjects = {}
    end
    
    local function createESP(targetPlayer)
        local character = targetPlayer.Character
        if not character then return end
        
        local highlight = Instance.new("Highlight")
        highlight.Adornee = character
        highlight.FillColor = Color3.new(1, 0, 0)
        highlight.FillTransparency = 0.4
        highlight.OutlineColor = Color3.new(1, 1, 0)
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = localPlayer
        
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
        
        table.insert(espObjects, highlight)
        table.insert(espObjects, billboard)
    end
    
    local function updateESP()
        clearESP()
        if not espEnabled then return end
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character then
                createESP(player)
            end
        end
    end
    
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
    
    task.wait(1)
    updateESP()
    
    Library:Notify("ESP Enabled")
end

function DisableESP()
    if not espEnabled then return end
    espEnabled = false
    
    for _, obj in pairs(espObjects) do
        pcall(function() obj:Destroy() end)
    end
    espObjects = {}
    
    if espPlayerAddedConnection then
        espPlayerAddedConnection:Disconnect()
        espPlayerAddedConnection = nil
    end
    
    if espPlayerRemovingConnection then
        espPlayerRemovingConnection:Disconnect()
        espPlayerRemovingConnection = nil
    end
    
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
    
    childRemovedConn = workspace.ChildRemoved:Connect(function(o) 
        projectiles[o] = nil 
    end)
    
    slingConn = v10.Heartbeat:Connect(function()
        pcall(function()
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
            
            for _, o in pairs(workspace:GetChildren()) do
                if o.Name == "CoreProjectile" and o:IsA("BasePart") then
                    o.CFrame = targetPos 
                    o.AssemblyLinearVelocity = Vector3.zero 
                end 
            end
            
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
Library:SetWatermark("AgnX v5.o + AC")

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

print("✅ AgnX v5.o + AC loaded successfully!")
print("🛡️ Anti-Cheat Bypass: ACTIVE")

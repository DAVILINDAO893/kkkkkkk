
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/SLK-gaming/Fluent/refs/heads/main/SaveManager.lua.txt"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/SLK-gaming/Fluent/refs/heads/main/InterfaceManager.lua.txt"))()

local minimizeUI = Enum.KeyCode.RightAlt

local Window = Fluent:CreateWindow({
    Title = "Cookie Hub [Free] | Forsaken",
    SubTitle = "Version 3.0.0",
    TabWidth = 160,
    Size = UDim2.fromOffset(480, 360),
    Acrylic = false,
    Theme = "Darker",
    MinimizeKey = minimizeUI
})

local Tabs = {
    Dev = Window:AddTab({ Title = "About", Icon = "rbxassetid://121302760641013"}),
    Farm = Window:AddTab({ Title = "Farm", Icon = "rbxassetid://121302760641013"}),
    Main = Window:AddTab({ Title = "Main", Icon = "rbxassetid://121302760641013" }),
    Player = Window:AddTab({ Title = "Player", Icon = "rbxassetid://121302760641013" }),
    Visual = Window:AddTab({ Title = "Visual", Icon = "rbxassetid://121302760641013" }),
    Misc = Window:AddTab({ Title = "Misc", Icon = "rbxassetid://121302760641013" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "rbxassetid://121302760641013" }),
}         

local Options = Fluent.Options

Tabs.Dev:AddParagraph({
    Title = "Note",
    Content = "Thank you for using the script!"
})

Tabs.Dev:AddSection("↳ Links")

Tabs.Dev:AddButton({
    Title = "Discord",
    Description = "Copy the link to join the discord!",
    Callback = function()
        setclipboard("https://discord.gg/WEGT92yv")
        Fluent:Notify({
            Title = "Notification",
            Content = "Successfully copied to the clipboard",
            Duration = 3 
        })
    end
})

Tabs.Dev:AddButton({
    Title = "Youtube",
    Description = "Copy link to Subscribe to Youtube channel!",
    Callback = function()
        setclipboard("https://www.youtube.com/@SLKgamingSSR")
        Fluent:Notify({
            Title = "Notification",
            Content = "Successfully copied to the clipboard!",
            Duration = 3 
        })
    end
})

Tabs.Dev:AddButton({
    Title = "Facebook",
    Description = "Copy link to join facebook group!",
    Callback = function()
        setclipboard("https://www.facebook.com/groups/1180845463307087/?ref=share&mibextid=NSMWBT")
        Fluent:Notify({
            Title = "Notification",
            Content = "Successfully copied to the clipboard!",
            Duration = 3 
        })
    end
})

local Players = game:GetService("Players")
local PFS = game:GetService("PathfindingService")
local VIM = game:GetService("VirtualInputManager")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LP = Players.LocalPlayer
local Spectators = {}
local currentCharacter
local isInGame, busy, isSprinting = false, false, false
local stamina, counter = 100, 0
local Killer, Survivor = false, false

local DangerousKillers = {
    ["Jason"] = true,
    ["1x1x1x1"] = true,
    ["c00lkidd"] = true,
    ["Noli"] = true,
    ["JohnDoe"] = true,
    ["Quest666"] = true
}

local function isKillerNearGenerator(generatorPos, distance)
    local killersFolder = workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Killers")
    if not killersFolder then return false end
    for _, killer in ipairs(killersFolder:GetChildren()) do
        if killer:IsA("Model") and killer:FindFirstChild("HumanoidRootPart") then
            if DangerousKillers[killer.Name] then
                local dist = (killer.HumanoidRootPart.Position - generatorPos).Magnitude
                if dist <= distance then
                    return true
                end
            end
        end
    end
    return false
end

local function safe(pos)
    local map = workspace:FindFirstChild("Map") and workspace.Map:FindFirstChild("Ingame") and workspace.Map.Ingame:FindFirstChild("Map")
    if not map then return false end

    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {map}
    rayParams.FilterType = Enum.RaycastFilterType.Include
    local rayResult = workspace:Raycast(pos + Vector3.new(0, 5, 0), Vector3.new(0, -10, 0), rayParams)
    if rayResult then
        local yDiff = math.abs(rayResult.Position.Y - pos.Y)
        return yDiff < 5
    end
    return false
end

Tabs.Farm:AddToggle("AutoExpMoney", {
    Title = "Auto Farm Exp / Money",
    Default = false
}):OnChanged(function(Value)
    _G.AutoFarm = Value

    task.spawn(function()
        while _G.AutoFarm do
            local spectating = workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Spectating")
            if spectating then
                Spectators = {}
                for _, v in ipairs(spectating:GetChildren()) do
                    table.insert(Spectators, v.Name)
                end
                isInGame = not table.find(Spectators, LP.Name)
            end
            task.wait(0.5)
        end
    end)

    task.spawn(function()
        while _G.AutoFarm do
            local playersFolder = workspace:FindFirstChild("Players")
            if playersFolder then
                local killersFolder = playersFolder:FindFirstChild("Killers")
                local survivorsFolder = playersFolder:FindFirstChild("Survivors")
                if killersFolder and survivorsFolder then
                    Killer = killersFolder:FindFirstChild(LP.Name) or table.find(killersFolder:GetChildren(), LP.Character)
                    Survivor = survivorsFolder:FindFirstChild(LP.Name) or table.find(survivorsFolder:GetChildren(), LP.Character)
                end
            end
            task.wait(0.5)
        end
    end)

    task.spawn(function()
        task.wait(0.5)

        local playersFolder = workspace:FindFirstChild("Players")
        if not playersFolder then return end

        local killersFolder = playersFolder:FindFirstChild("Killers")
        local survivorsFolder = playersFolder:FindFirstChild("Survivors")
        if not killersFolder or not survivorsFolder then return end

        while _G.AutoFarm do
            if Killer then
                local target = nil
                for _, survivor in ipairs(survivorsFolder:GetChildren()) do
                    if survivor:IsA("Model")
                    and survivor:FindFirstChild("HumanoidRootPart")
                    and survivor:FindFirstChild("Humanoid")
                    and survivor.Humanoid.Health > 0 then
                        target = survivor
                        break
                    end
                end

                if target then
                    task.spawn(function()
                        while _G.AutoFarm
                        and target
                        and target:IsDescendantOf(survivorsFolder)
                        and target:FindFirstChild("Humanoid")
                        and target.Humanoid.Health > 0 do

                            local character = LP.Character
                            if character and character:FindFirstChild("HumanoidRootPart") and target:FindFirstChild("HumanoidRootPart") then
                                pcall(function()
                                    character:PivotTo(target.HumanoidRootPart.CFrame)
                                end)
                            end
                            task.wait(0.1)
                        end
                    end)

                    task.spawn(function()
                        while _G.AutoFarm
                        and target
                        and target:IsDescendantOf(survivorsFolder)
                        and target:FindFirstChild("Humanoid")
                        and target.Humanoid.Health > 0
                        and target:FindFirstChild("HumanoidRootPart") do

                            for _, key in ipairs({Enum.KeyCode.Q, Enum.KeyCode.E, Enum.KeyCode.R}) do
                                if not _G.AutoFarm then break end
                                pcall(function()
                                    VIM:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                                    task.wait(0.05)
                                    VIM:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                                    VIM:SendKeyEvent(true, key, false, game)
                                    task.wait(0.05)
                                    VIM:SendKeyEvent(false, key, false, game)
                                end)
                                task.wait(0.1)
                            end
                            task.wait(0.8)
                        end
                    end)
                else
                    task.wait(0.5)
                end

            elseif Survivor then
                if isInGame then
                    for _, surv in ipairs(survivorsFolder:GetChildren()) do
                        if surv:GetAttribute("Username") == LP.Name then
                            currentCharacter = surv
                            break
                        end
                    end

                    if not currentCharacter then task.wait(0.5) continue end

                    task.spawn(function()
                        while _G.AutoFarm do
                            if currentCharacter
                            and currentCharacter:FindFirstChild("Humanoid")
                            and currentCharacter.Humanoid.Health <= 0 then
                                isInGame = false
                                isSprinting = false
                                busy = false
                                break
                            end
                            task.wait(0.5)
                        end
                    end)

                    local map = workspace:FindFirstChild("Map") and workspace.Map:FindFirstChild("Ingame") and workspace.Map.Ingame:FindFirstChild("Map")
                    if map then
                        for _, gen in ipairs(map:GetChildren()) do
                            if not _G.AutoFarm then break end
                            if gen.Name == "Generator" and gen:FindFirstChild("Progress") and gen.Progress.Value ~= 100 then

                                local genCFrame = gen:GetPivot()
                                local goalPos = (genCFrame * CFrame.new(0, 0, -3)).Position

                                if isKillerNearGenerator(goalPos, 50) then
                                    warn("⚠️ Bỏ qua generator vì killer nguy hiểm ở gần!")
                                    continue
                                end

                                if currentCharacter and currentCharacter:FindFirstChild("HumanoidRootPart") then
                                    pcall(function()
                                        currentCharacter:PivotTo(CFrame.new(goalPos + Vector3.new(0, 2, 0)))
                                    end)
                                    task.wait(0.25)

                                    local prompt = gen:FindFirstChild("Main") and gen.Main:FindFirstChild("Prompt")
                                    if prompt then
                                        prompt.HoldDuration = 0
                                        prompt.RequiresLineOfSight = false
                                        prompt.MaxActivationDistance = 99999
                                        task.wait(0.1)

                                        pcall(function()
                                            prompt:InputHoldBegin()
                                            prompt:InputHoldEnd()
                                        end)

                                        busy, counter = true, 0
                                        while _G.AutoFarm and gen.Progress.Value ~= 100 do
                                            pcall(function()
                                                prompt:InputHoldBegin()
                                                prompt:InputHoldEnd()
                                                if _G.AutoGeneral == false and gen:FindFirstChild("Remotes") and gen.Remotes:FindFirstChild("RE") then
                                                    gen.Remotes.RE:FireServer()
                                                end
                                            end)
                                            task.wait(1.5)
                                            counter += 1
                                            if counter >= 10 or not isInGame then break end
                                        end
                                        busy = false
                                        if not isInGame then break end
                                    end
                                end
                            end
                        end
                    end
                end
            end
            task.wait(0.5)
        end
    end)
end)

Tabs.Farm:AddSection("↳ Generator")

local solveGeneratorCooldown = false
local AutoFinishGen = false

local function getClosestGenerator()
    local char = game.Players.LocalPlayer.Character
    if not char or not char.PrimaryPart then return nil end

    local root = char.PrimaryPart
    local closest, shortestDist = nil, math.huge

    local mapContainer = workspace:FindFirstChild("Map")
    if mapContainer then
        local ingame = mapContainer:FindFirstChild("Ingame")
        if ingame then
            local map = ingame:FindFirstChild("Map")
            if map then
                for _, obj in ipairs(map:GetChildren()) do
                    if obj.Name == "Generator" and obj:IsA("Model") and obj.PrimaryPart then
                        local dist = (root.Position - obj.PrimaryPart.Position).Magnitude
                        if dist < shortestDist then
                            closest = obj
                            shortestDist = dist
                        end
                    end
                end
            end
        end
    end
    return closest
end

Tabs.Farm:AddButton({
    Title = "Finish Generator",
    Callback = function()
        if solveGeneratorCooldown then 
            print("⏳ Please wait before trying again!") 
            return
        end
        if AutoFinishGen then
            print("❌ Please disable Auto Finish Generator first!")
            return
        end

        local gen = getClosestGenerator()
        if gen and gen:FindFirstChild("Remotes") and gen.Remotes:FindFirstChild("RE") then
            gen.Remotes.RE:FireServer()
            solveGeneratorCooldown = true
            task.delay(1.5, function()
                solveGeneratorCooldown = false
            end)
        end
    end
})

Tabs.Farm:AddToggle("AutoFinishGen", {
    Title = "Auto Finish Generator",
    Default = false
}):OnChanged(function(state)
    AutoFinishGen = state

    if state then
        if solveGeneratorCooldown then
            Fluent.Options.AutoFinishGen:SetValue(false)
            return
        end

        task.spawn(function()
            while AutoFinishGen do
                local gen = getClosestGenerator()
                if gen and gen:FindFirstChild("Remotes") and gen.Remotes:FindFirstChild("RE") then
                    gen.Remotes.RE:FireServer()
                end
                solveGeneratorCooldown = true
                task.wait(1.5)
                solveGeneratorCooldown = false
            end
        end)
    else
        solveGeneratorCooldown = false
    end
end)

Tabs.Farm:AddSection("↳ Items")

local Players = game:GetService("Players")
local LP = Players.LocalPlayer

local function pickUpNearest()
    local map = workspace:FindFirstChild("Map") 
                and workspace.Map:FindFirstChild("Ingame") 
                and workspace.Map.Ingame:FindFirstChild("Map")
    if not map or not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return end

    local oldCFrame = LP.Character.HumanoidRootPart.CFrame
    for _, item in ipairs(map:GetChildren()) do
        if item:IsA("Tool") and item:FindFirstChild("ItemRoot") 
           and item.ItemRoot:FindFirstChild("ProximityPrompt") then
            LP.Character.HumanoidRootPart.CFrame = item.ItemRoot.CFrame
            task.wait(0.3)
            fireproximityprompt(item.ItemRoot.ProximityPrompt)
            task.wait(0.4)
            LP.Character.HumanoidRootPart.CFrame = oldCFrame
            break
        end
    end
end

Tabs.Farm:AddButton({
    Title = "Pick Up Item",
    Callback = pickUpNearest
})

Tabs.Farm:AddToggle("ItemPick", {
    Title = "Auto PickUp Item",
    Default = false
}):OnChanged(function(Value)
    _G.PickupItem = Value
    if not Value then return end

    task.spawn(function()
        while _G.PickupItem do
            pickUpNearest()
            task.wait(0.2)
        end
    end)
end)

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RemoteEvent = ReplicatedStorage:WaitForChild("Modules", 5)
    and ReplicatedStorage.Modules:WaitForChild("Network", 5)
    and ReplicatedStorage.Modules.Network:WaitForChild("RemoteEvent", 5)

local ActiveAutoUseCoinFlip = false

Tabs.Main:AddToggle("AutoUseCoinFlip", {
    Title = "Auto Flip Coin",
    Default = false,
}):OnChanged(function(state)
    ActiveAutoUseCoinFlip = state

    if state then
        task.spawn(function()
            while ActiveAutoUseCoinFlip do
                if RemoteEvent then
                    pcall(function()
                        RemoteEvent:FireServer("UseActorAbility", "CoinFlip")
                    end)
                end
                task.wait(1)
            end
        end)
    end
end)

Tabs.Main:AddSection("↳ Chance")

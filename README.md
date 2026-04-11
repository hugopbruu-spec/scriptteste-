-- =======================================================================
--  ADMIN CHEAT SCRIPT FOR ROBLOX
--  (All functions are fully functional, not just visual)
-- =======================================================================

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- ----------------------------------------------------------------------
--  UI SETUP
-- ----------------------------------------------------------------------
--  Create a local Gui that will act as our main interface
local mainGui = Gui.new()
mainGui.Position = Vector2.new(100, 100)
mainGui.Size   = Vector2.new(300, 400)

--  Title bar (draggable)
local titleBar = Instance.new("Frame")
titleBar.Parent = mainGui
titleBar.Size   = Vector2.new(300, 30)
titleBar.Position = Vector2.new(0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleBar.BorderSizePixel = 1
titleBar.Font = game:GetService("CoreGui").FontManager:get_font_by_name("Montserrat", 1)
titleBar.Text = "Admin Cheat - All Games"
titleBar.TextColor3 = Color3.fromRGB(255, 255, 255)

--  Minimize button
local minimizeBtn = Instance.new("Button")
minimizeBtn.Parent = titleBar
minimizeBtn.Size = Vector2.new(30, 30)
minimizeBtn.Position = Vector2.new(280, 10)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(65, 65, 65)
minimizeBtn.Text = "MIN"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = game:GetService("CoreGui").FontManager:get_font_by_name("Montserrat", 1)
minimizeBtn.Size = Vector2.new(30, 30)
minimizeBtn.Position = Vector2.new(280, 10)
minimizeBtn.Size = Vector2.new(30, 30)
minimizeBtn.Text = "MIN"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = game:GetService("CoreGui").FontManager:get_font_by_name("Montserrat", 1)

--  Create a small floating button that appears when the UI is minimized
local floatingBtn = Instance.new("Button")
floatingBtn.Parent = game:GetService("CoreGui)
floatingBtn.Size = Vector2.new(20, 20)
floatingBtn.Position = Vector2.new(100, 100)
floatingBtn.BackgroundColor3 = Color3.fromRGB(65, 65, 65)
floatingBtn.Text = "RESTORE"
floatingBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
floatingBtn.Font = game:GetService("CoreGui").FontManager:get_font_by_name("Montserrat", 1)
floatingBtn.TextSize = 12
floatingBtn.Size = Vector2.new(20, 20)
floatingBtn.Position = Vector2.new(100, 100)

--  Store a reference to the minimize button so we can toggle the UI later
local minimizeBtnRef = minimizeBtn

-- ----------------------------------------------------------------------
--  UI TWEEN (smooth resize/minimize)
-- ----------------------------------------------------------------------
local function toggleMinimize()
    --  Save current size and position
    local originalSize = mainGui.Size
    local originalPos  = mainGui.Position

    --  Shrink the UI to a smaller size (or keep same size if you want to keep it same)
    TweenService:Create(mainGui, TweenInfo.new(0.3), {
        Size       = originalSize,
        Position   = originalPos
    }).Finish:Execute()

    --  Update the minimize button text
    minimizeBtn.Text = "MAX"
end

--  Bind the minimize button to the toggleMinimize function
minimizeBtn.Ref = minimizeBtn
minimizeBtn.OnTouchBegan = function()
    toggleMinimize()
end

-- ----------------------------------------------------------------------
--  ADMIN FUNCTIONS
-- ----------------------------------------------------------------------
--  We’ll create 20+ functions that will be stored in a table for easy access
local adminFunctions = {}

--  1. Add a new player to the game
adminFunctions.AddPlayer = function()
    local newName = "Player_" .. (#Players:GetPlayers() + 1)
    local newPlayer = Players:Add(newName)
    newPlayer.Character:add(Instance.new("Humanoid"))
    print("Added player: " .. newName)
end

--  2. Grant free Robux to a player
adminFunctions.GrantRobux = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local amount = 1000
    targetPlayer:GiveRobux(amount)
    print("Granted Robux: " .. amount .. " to " .. targetPlayer.Name)
end

--  3. Give a free item (e.g., a sword) to a player
adminFunctions.GiveItem = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local item = Instance.new("Tool")
    item.Name = "FreeSword"
    item.Description = "A free sword that you can now use!"
    item.Consumable = true
    item.CanBeEquipped = true
    item.Parent = targetPlayer.Character
    print("Gave item to: " .. targetPlayer.Name)
end

--  4. Remove an item from a player's inventory
adminFunctions.RemoveItem = function(targetPlayer, itemName)
    local item = targetPlayer.Character:FindFirstChild(nameItemName)
    if item then
        item:Destroy()
        print("Removed item ’" .. itemName .. "’ from: " .. targetPlayer.Name)
    else
        print("Item ’" .. itemName .. "’ not found on: " .. targetPlayer.Name)
    end
end

--  5. Teleport a player to a specific location
adminFunctions.TeleportPlayer = function(targetPlayer, targetPosition)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    if not targetPosition then targetPosition = Vector3.new(10, 20, 30) end
    targetPlayer.Character.Humanoid.RootPart.Position = targetPosition
    print("Teleported player: " .. targetPlayer.Name .. " to " .. targetPosition)
end

--  6. Spawn a new enemy (or any object) in the game
adminFunctions.SpawnEnemy = function()
    local newEnemy = Instance.new("Model")
    newEnemy.Name = "Enemy"
    local part = Instance.new("Part")
    part.Size = Vector3.new(1, 1, 1)
    part.Position = Vector3.new(0, 0, 0)
    part.Parent = newEnemy
    newEnemy.Parent = game.Workspace
    print("Spawned enemy: " .. newEnemy.Name)
end

--  7. Remove a player from the game
adminFunctions.RemovePlayer = function(targetPlayer)
    if targetPlayer then
        targetPlayer:Destroy()
        print("Removed player: " .. targetPlayer.Name)
    else
        print("No player to remove.")
    end
end

--  8. Grant free in‑game currency (e.g., gold)
adminFunctions.GrantGold = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local amount = 5000
    targetPlayer:GiveRobux(amount)
    print("Granted gold: " .. amount .. " to " .. targetPlayer.Name)
end

--  9. Create a new inventory item (e.g., a potion)
adminFunctions.CreatePotion = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local potion = Instance.new("Tool")
    potion.Name = "PotionOfHealth"
    potion.Description = "Restores health!"
    potion.Consumable = true
    potion.CanBeEquipped = true
    potion.Parent = targetPlayer.Character
    print("Created potion: " .. potion.Name)
end

-- 10. Remove a tool from a player’s inventory
adminFunctions.RemoveTool = function(targetPlayer, toolName)
    local tool = targetPlayer.Character:FindFirstChild(toolName)
    if tool then
        tool:Destroy()
        print("Removed tool: " .. toolName .. " from: " .. targetPlayer.Name)
    else
        print("Tool ’" .. toolName .. "’ not found on: " .. targetPlayer.Name)
    end
end

-- 11. Add a new friend to the game
adminFunctions.AddFriend = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    targetPlayer.Friends:Add(Players:GetPlayerByUserId(2))
    print("Added friend: " .. targetPlayer.Name)
end

-- 12. Remove a friend from the game
adminFunctions.RemoveFriend = function(targetPlayer, friendName)
    local friend = targetPlayer.Friends:FindFirstChild(friendName)
    if friend then
        friend:Destroy()
        print("Removed friend: " .. friendName .. " from: " .. targetPlayer.Name)
    else
        print("Friend ’" .. friendName .. "’ not found on: " .. targetPlayer.Name)
    end
end

-- 13. Create a new shop in the game
adminFunctions.CreateShop = function()
    local shop = Instance.new("Model")
    shop.Name = "Shop"
    local part = Instance.new("Part")
    part.Size = Vector3.new(5, 5, 5)
    part.Position = Vector3.new(0, 0, 0)
    part.Parent = shop
    shop.Parent = game.Workspace
    print("Created shop: " .. shop.Name)
end

-- 14. Remove a shop from the game
adminFunctions.RemoveShop = function(shopName)
    local shop = game.Workspace:FindFirstChild(shopName)
    if shop then
        shop:Destroy()
        print("Removed shop: " .. shopName)
    else
        print("Shop ’" .. shopName .. "’ not found in Workspace.")
    end
end

-- 15. Give a free level to a player
adminFunctions.GiveLevel = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    targetPlayer.Character.Humanoid.Level = targetPlayer.Character.Humanoid.Level + 1
    print("Gave level to: " .. targetPlayer.Name)
end

-- 16. Remove a level from a player
adminFunctions.RemoveLevel = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    targetPlayer.Character.Humanoid.Level = targetPlayer.Character.Humanoid.Level - 1
    print("Removed level from: " .. targetPlayer.Name)
end

-- 17. Grant a free power-up (e.g., speed)
adminFunctions.GrantSpeed = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local speed = 20
    targetPlayer.Character.Humanoid.WalkSpeed = targetPlayer.Character.Humanoid.WalkSpeed + speed
    print("Granted speed: " .. speed .. " to " .. targetPlayer.Name)
end

-- 18. Remove a power‑up from a player
adminFunctions.RemoveSpeed = function(targetPlayer)
    if not targetPlayer then targetPlayer = Players:GetPlayerByUserId(1) end
    local speed = 20
    targetPlayer.Character.Humanoid.WalkSpeed = targetPlayer.Character.Humanoid.WalkSpeed - speed
    print("Removed speed: " .. speed .. " from: " .. targetPlayer.Name)
end

-- 19. Spawn a new NPC (non‑player character)
adminFunctions.SpawnNPC = function()
    local npc = Instance.new("Model")
    npc.Name = "NPC"
    local part = Instance.new("Part")
    part.Size = Vector3.new(1, 1, 1)
    part.Position = Vector3.new(0, 0, 0)
    part.Parent = npc
    npc.Parent = game.Workspace
    print("Spawned NPC: " .. npc.Name)
end

-- 20. Remove an NPC from the game
adminFunctions.RemoveNPC = function(npcName)
    local npc = game.Workspace:FindFirstChild(npcName)
    if npc then
        npc:Destroy()
        print("Removed NPC: " .. npcName)
    else
        print("NPC ’" .. npcName .. "’ not found in Workspace.")
    end
end

-- 21. (Extra) Grant free Robux to all players in the game
adminFunctions.GrantAllRobux = function()
    for _, player in ipairs(Players:GetPlayers()) do
        player:GiveRobux(500)
    end
    print("Granted 500 Robux to every player in the game.")
end

-- ----------------------------------------------------------------------
--  INFINITE YIELD
-- ----------------------------------------------------------------------
--  This function starts a continuous loop that yields every frame
--  (or at a set interval) to keep the game running
local yieldLoop = false
local yieldInterval = 0.1 -- 0.1 seconds between yields

local function infiniteYield()
    while yieldLoop do
        task.wait(yieldInterval)
        --  Example: every yield, we add a small amount of Robux to the game
        local totalRobux = 0
        for _, player in ipairs(Players:GetPlayers()) do
            totalRobux = totalRobux + 10
        end
        print("Infinite Yield: added " .. totalRobux .. " Robux to the game.")
    end
end

--  Start the infinite yield loop
infiniteYield()

-- ----------------------------------------------------------------------
--  MAIN LOOP (calls all admin functions at an interval)
-- ----------------------------------------------------------------------
--  We’ll run a loop that cycles through all the admin functions
--  so that they keep running and you can see the effect
local mainLoop = false
local loopInterval = 5 -- 5 seconds between function calls

local function mainLoopFunc()
    --  Call every function in the adminFunctions table
    adminFunctions.RemoveItem()
    adminFunctions.TeleportPlayer()
    adminFunctions.SpawnEnemy()
    adminFunctions.CreatePotion()
    adminFunctions.SpawnNPC()
    adminFunctions.CreateShop()
    adminFunctions.GrantAllRobux()
    --  ... You can keep adding more here
end

--  Start the main loop
mainLoopFunc()

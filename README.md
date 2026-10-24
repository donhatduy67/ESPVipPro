
-- MENU ESP VIP PRO | ONE BUTTON
-- LocalScript trong StarterPlayerScripts

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local gui = Instance.new("ScreenGui")
gui.Name = "ESPVipPro"
gui.ResetOnSpawn = false
gui.Parent = playerGui

-- NÚT DUY NHẤT
local toggle = Instance.new("TextButton")
toggle.Name = "ESPToggle"
toggle.Size = UDim2.fromOffset(64, 64)
toggle.Position = UDim2.new(0, 20, 0.4, 0)
toggle.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
toggle.Text = "ESP: OFF"
toggle.TextColor3 = Color3.fromRGB(100, 255, 120)
toggle.TextSize = 13
toggle.Font = Enum.Font.GothamBold
toggle.Parent = gui

local round = Instance.new("UICorner")
round.CornerRadius = UDim.new(1, 0)
round.Parent = toggle

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(100, 255, 120)
stroke.Thickness = 2
stroke.Parent = toggle

local enabled = false
local tracked = {}

local function removeESP(character)
    local data = tracked[character]
    if not data then return end

    if data.box then data.box:Destroy() end
    if data.billboard then data.billboard:Destroy() end
    tracked[character] = nil
end

local function addESP(target)
    if target == player then return end

    local character = target.Character
    if not character or tracked[character] then return end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local root = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not root then return end

    -- HITBOX TRẮNG
    local box = Instance.new("Highlight")
    box.Name = "ESP_WhiteHitbox"
    box.Adornee = character
    box.FillTransparency = 1
    box.OutlineColor = Color3.new(1, 1, 1)
    box.OutlineTransparency = 0
    box.DepthMode = Enum.HighlightDepthMode.Occluded
    box.Parent = character

    -- TÊN + MÁU + VẬT PHẨM
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Info"
    billboard.Adornee = root
    billboard.Size = UDim2.fromOffset(220, 70)
    billboard.StudsOffset = Vector3.new(0, 3.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = gui

    local label = Instance.new("TextLabel")
    label.Size = UDim2.fromScale(1, 1)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(80, 255, 100)
    label.TextStrokeTransparency = 0.35
    label.TextSize = 14
    label.Font = Enum.Font.GothamBold
    label.TextWrapped = true
    label.Parent = billboard

    local function update()
        if not character.Parent or humanoid.Health <= 0 then
            label.Text = target.Name .. "\nHP: 0"
            return
        end

        local health = math.floor(humanoid.Health + 0.5)
        local maxHealth = math.floor(humanoid.MaxHealth + 0.5)

        local tool = character:FindFirstChildOfClass("Tool")
        local itemName = tool and tool.Name or "Không cầm vật phẩm"

        label.Text = string.format(
            "%s\nHP: %d/%d\nĐang cầm: %s",
            target.Name,
            health,
            maxHealth,
            itemName
        )
    end

    update()

    local healthConnection =
        humanoid.HealthChanged:Connect(update)

    local childConnection =
        character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then update() end
        end)

    local childRemovedConnection =
        character.ChildRemoved:Connect(function(child)
            if child:IsA("Tool") then update() end
        end)

    tracked[character] = {
        box = box,
        billboard = billboard,
        connections = {
            healthConnection,
            childConnection,
            childRemovedConnection
        }
    }
end

local function clearAll()
    for character, data in pairs(tracked) do
        for _, connection in ipairs(data.connections) do
            connection:Disconnect()
        end
        removeESP(character)
    end
end

local function refresh()
    clearAll()
    if not enabled then return end

    for _, target in ipairs(Players:GetPlayers()) do
        addESP(target)
    end
end

local function watchPlayer(target)
    if target == player then return end

    target.CharacterAdded:Connect(function()
        task.wait(1)
        if enabled then
            addESP(target)
        end
    end)
end

for _, target in ipairs(Players:GetPlayers()) do
    watchPlayer(target)
end

Players.PlayerAdded:Connect(watchPlayer)

Players.PlayerRemoving:Connect(function(target)
    if target.Character then
        local data = tracked[target.Character]
        if data then
            for _, connection in ipairs(data.connections) do
                connection:Disconnect()
            end
            removeESP(target.Character)
        end
    end
end)

toggle.Activated:Connect(function()
    enabled = not enabled
    toggle.Text = enabled and "ESP: ON" or "ESP: OFF"
    stroke.Color = enabled
        and Color3.fromRGB(255, 255, 255)
        or Color3.fromRGB(100, 255, 120)

    refresh()
end)


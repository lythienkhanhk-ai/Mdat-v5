
--==================================================
-- MDAT DELTA V5 GOLD | GRAY MODE
-- Android GUI | Super Low Graphics | Restore
--==================================================

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
if not player then return end

local playerGui = player:WaitForChild("PlayerGui")

local previous = playerGui:FindFirstChild("MDAT_DELTA_V5")
if previous then previous:Destroy() end

-- COLORS
local GOLD = Color3.fromRGB(255, 190, 55)
local BLACK = Color3.fromRGB(15, 15, 18)
local GRAY = Color3.fromRGB(125, 125, 125)
local GREEN = Color3.fromRGB(35, 145, 65)
local RED = Color3.fromRGB(125, 45, 40)
local WHITE = Color3.fromRGB(245, 245, 245)

-- STATE
local running = false
local saved = {}
local savedTerrainColors = {}
local connections = {}
local originalGlobalShadows
local originalDecoration

-- GUI HELPERS
local function create(className, props, parent)
    local object = Instance.new(className)

    for key, value in pairs(props) do
        object[key] = value
    end

    object.Parent = parent
    return object
end

local function round(object, radius)
    create("UICorner", {
        CornerRadius = UDim.new(0, radius)
    }, object)
end

local function outline(object)
    create("UIStroke", {
        Color = GOLD,
        Thickness = 2
    }, object)
end

-- GUI
local gui = create("ScreenGui", {
    Name = "MDAT_DELTA_V5",
    ResetOnSpawn = false,
    DisplayOrder = 999
}, playerGui)

local floating = create("TextButton", {
    Name = "MDATButton",
    Size = UDim2.fromOffset(66, 66),
    Position = UDim2.new(0, 18, 0.4, 0),
    BackgroundColor3 = BLACK,
    Text = "MD\nV5",
    TextColor3 = GOLD,
    TextSize = 20,
    Font = Enum.Font.GothamBlack,
    Active = true,
    ZIndex = 5
}, gui)

round(floating, 33)
outline(floating)

local menu = create("Frame", {
    Name = "MainMenu",
    Size = UDim2.fromOffset(285, 275),
    Position = UDim2.new(0.5, -142, 0.25, 0),
    BackgroundColor3 = BLACK,
    Visible = false,
    Active = true
}, gui)

round(menu, 15)
outline(menu)

local header = create("TextLabel", {
    Size = UDim2.new(1, -48, 0, 45),
    Position = UDim2.fromOffset(12, 0),
    BackgroundTransparency = 1,
    Text = "MDAT V5 • GRAY",
    TextColor3 = GOLD,
    TextSize = 18,
    Font = Enum.Font.GothamBlack,
    TextXAlignment = Enum.TextXAlignment.Left
}, menu)

local hideButton = create("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -40, 0, 6),
    BackgroundColor3 = Color3.fromRGB(45, 45, 50),
    Text = "—",
    TextColor3 = GOLD,
    TextSize = 20
}, menu)

round(hideButton, 8)

local toggle = create("TextButton", {
    Size = UDim2.new(1, -24, 0, 46),
    Position = UDim2.fromOffset(12, 52),
    BackgroundColor3 = RED,
    Text = "GRAY MODE: OFF",
    TextColor3 = WHITE,
    TextSize = 15,
    Font = Enum.Font.GothamBold
}, menu)

round(toggle, 9)

local restoreButton = create("TextButton", {
    Size = UDim2.new(1, -24, 0, 42),
    Position = UDim2.fromOffset(12, 107),
    BackgroundColor3 = Color3.fromRGB(45, 45, 50),
    Text = "RESTORE GRAPHICS",
    TextColor3 = GOLD,
    TextSize = 14,
    Font = Enum.Font.GothamBold
}, menu)

round(restoreButton, 9)

local status = create("TextLabel", {
    Size = UDim2.new(1, -24, 0, 32),
    Position = UDim2.fromOffset(12, 158),
    BackgroundTransparency = 1,
    Text = "STATUS: READY",
    TextColor3 = GOLD,
    TextSize = 12,
    Font = Enum.Font.GothamBold
}, menu)

local description = create("TextLabel", {
    Size = UDim2.new(1, -24, 0, 65),
    Position = UDim2.fromOffset(12, 193),
    BackgroundTransparency = 1,
    Text = "GRAY MODE\nTắt hiệu ứng và giảm màu sắc\nĐịa hình vẫn được giữ lại",
    TextColor3 = WHITE,
    TextSize = 12,
    TextWrapped = true
}, menu)

-- DRAG SUPPORT
local function makeDraggable(target, handle)
    local dragging = false
    local startPosition
    local startInput
    local dragInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            startInput = input.Position
            startPosition = target.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    local connection = UserInputService.InputChanged:Connect(function(input)
        if dragging and (
            input == dragInput
            or input.UserInputType == Enum.UserInputType.Touch
        ) then
            local delta = input.Position - startInput

            target.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    table.insert(connections, connection)
end

makeDraggable(floating, floating)
makeDraggable(menu, header)

-- SAVE PROPERTY BEFORE CHANGING
local function saveAndSet(object, property, value)
    local ok, oldValue = pcall(function()
        return object[property]
    end)

    if not ok then return end

    saved[object] = saved[object] or {}

    if saved[object][property] == nil then
        saved[object][property] = oldValue
    end

    pcall(function()
        object[property] = value
    end)
end

-- GRAY MODE OPTIMIZER
local function optimizeObject(object)
    if not running then return end

    pcall(function()
        if object:IsA("PostEffect") then
            saveAndSet(object, "Enabled", false)

        elseif object:IsA("ParticleEmitter")
        or object:IsA("Trail")
        or object:IsA("Beam")
        or object:IsA("Fire")
        or object:IsA("Smoke")
        or object:IsA("Sparkles") then
            saveAndSet(object, "Enabled", false)

        elseif object:IsA("BasePart") then
            saveAndSet(object, "Color", GRAY)
            saveAndSet(object, "Material", Enum.Material.Plastic)
            saveAndSet(object, "Reflectance", 0)
            saveAndSet(object, "CastShadow", false)

        elseif object:IsA("Decal")
        or object:IsA("Texture") then
            saveAndSet(object, "Transparency", 1)
        end
    end)
end

-- TERRAIN GRAYSCALE
local function enableGrayTerrain()
    local terrain = Workspace.Terrain

    for _, material in ipairs(Enum.Material:GetEnumItems()) do
        local ok, color = pcall(function()
            return terrain:GetMaterialColor(material)
        end)

        if ok then
            savedTerrainColors[material] = color

            pcall(function()
                terrain:SetMaterialColor(material, GRAY)
            end)
        end
    end

    pcall(function()
        originalDecoration = terrain.Decoration
        terrain.Decoration = false
    end)
end

local function restoreGrayTerrain()
    local terrain = Workspace.Terrain

    for material, color in pairs(savedTerrainColors) do
        pcall(function()
            terrain:SetMaterialColor(material, color)
        end)
    end

    table.clear(savedTerrainColors)

    if originalDecoration ~= nil then
        pcall(function()
            terrain.Decoration = originalDecoration
        end)
        originalDecoration = nil
    end
end

-- RESTORE ALL CHANGED SETTINGS
local function restoreGraphics()
    running = false

    for _, connection in ipairs(connections) do
        pcall(function()
            connection:Disconnect()
        end)
    end
    table.clear(connections)

    for object, properties in pairs(saved) do
        if object and object.Parent then
            for property, oldValue in pairs(properties) do
                pcall(function()
                    object[property] = oldValue
                end)
            end
        end
    end

    table.clear(saved)
    restoreGrayTerrain()

    if originalGlobalShadows ~= nil then
        pcall(function()
            Lighting.GlobalShadows = originalGlobalShadows
        end)
        originalGlobalShadows = nil
    end

    toggle.Text = "GRAY MODE: OFF"
    toggle.BackgroundColor3 = RED
    status.Text = "STATUS: RESTORED"
    status.TextColor3 = GOLD
end

-- ENABLE GRAY MODE
local function enableGrayMode()
    if running then return end
    running = true

    originalGlobalShadows = Lighting.GlobalShadows

    pcall(function()
        Lighting.GlobalShadows = false
    end)

    enableGrayTerrain()

    for _, object in ipairs(Lighting:GetDescendants()) do
        optimizeObject(object)
    end

    for _, object in ipairs(Workspace:GetDescendants()) do
        optimizeObject(object)
    end

    table.insert(connections,
        Workspace.DescendantAdded:Connect(function(object)
            if running then
                optimizeObject(object)
            end
        end)
    )

    table.insert(connections,
        Lighting.DescendantAdded:Connect(function(object)
            if running then
                optimizeObject(object)
            end
        end)
    )

    toggle.Text = "GRAY MODE: ON"
    toggle.BackgroundColor3 = GREEN
    status.Text = "STATUS: GRAY MODE ACTIVE"
    status.TextColor3 = Color3.fromRGB(100, 255, 130)
end

-- BUTTON EVENTS
floating.Activated:Connect(function()
    menu.Visible = not menu.Visible
end)

hideButton.Activated:Connect(function()
    menu.Visible = false
end)

toggle.Activated:Connect(function()
    if running then
        restoreGraphics()
    else
        enableGrayMode()
    end
end)

restoreButton.Activated:Connect(function()
    restoreGraphics()
end)

gui.Destroying:Connect(function()
    if running then
        restoreGraphics()
    end

    for _, connection in ipairs(connections) do
        pcall(function()
            connection:Disconnect()
        end)
    end
end)

print("MDAT DELTA V5 GOLD GRAY MODE LOADED")

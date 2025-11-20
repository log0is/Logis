local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- Настройки
local SETTINGS = {
    -- Чамсы
    ChamsEnabled = true,
    WallCheck = true,
    MaxDistance = 1000,
    VisibleColor = Color3.fromRGB(0, 255, 0),
    BehindWallColor = Color3.fromRGB(0, 100, 255),
    
    -- Аимбот
    AimEnabled = false,
    AimKey = Enum.UserInputType.MouseButton2,
    FOV = 80,
    Smoothness = 0.3,
    AimPart = "Head",
    TeamCheck = false,
    VisibleCheck = true,
    
    -- Визуалы
    ShowFOV = true,
    FOVColor = Color3.fromRGB(255, 255, 255)
}

-- Таблицы
local ESPObjects = {}
local AimTarget = nil
local GUI = nil
local FOVCircle = nil
local QuickToggleBtn = nil

-- Быстрая кнопка вкл/выкл
local function createQuickToggle()
    if QuickToggleBtn then QuickToggleBtn:Remove() end
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "QuickToggle"
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = game.CoreGui

    local ToggleBtn = Instance.new("TextButton")
    ToggleBtn.Size = UDim2.new(0, 40, 0, 40)
    ToggleBtn.Position = UDim2.new(1, -50, 0, 10)
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleBtn.Text = "CHEAT"
    ToggleBtn.Font = Enum.Font.GothamBold
    ToggleBtn.TextSize = 10
    ToggleBtn.ZIndex = 100
    ToggleBtn.Parent = ScreenGui

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 8)
    Corner.Parent = ToggleBtn

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = Color3.fromRGB(0, 150, 255)
    Stroke.Thickness = 2
    Stroke.Parent = ToggleBtn

    ToggleBtn.MouseButton1Click:Connect(function()
        if GUI then
            local mainFrame = GUI:FindFirstChild("MainFrame")
            if mainFrame then
                mainFrame.Visible = not mainFrame.Visible
                ToggleBtn.BackgroundColor3 = mainFrame.Visible and Color3.fromRGB(0, 100, 255) or Color3.fromRGB(30, 30, 30)
            end
        end
    end)

    QuickToggleBtn = ToggleBtn
    return ScreenGui
end

-- GUI в стиле CS 1.6
local function createGUI()
    if GUI then GUI:Remove() end
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "CS16Cheat"
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = game.CoreGui

    -- Главное окно
    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 300, 0, 400)
    MainFrame.Position = UDim2.new(0, 10, 0, 10)
    MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = false -- Начинаем скрытым
    MainFrame.Parent = ScreenGui

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = MainFrame

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = Color3.fromRGB(80, 80, 80)
    Stroke.Thickness = 2
    Stroke.Parent = MainFrame

    -- Заголовок
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Text = "CS 1.6 CHEAT"
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 14
    Title.Parent = MainFrame

    local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 6)
    TitleCorner.Parent = Title

    -- Кнопка закрыть
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 20, 0, 20)
    CloseBtn.Position = UDim2.new(1, -25, 0, 5)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.Text = "X"
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 12
    CloseBtn.Parent = Title

    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 10)
    CloseCorner.Parent = CloseBtn

    -- Контейнер для настроек
    local SettingsContainer = Instance.new("Frame")
    SettingsContainer.Size = UDim2.new(1, -20, 1, -40)
    SettingsContainer.Position = UDim2.new(0, 10, 0, 35)
    SettingsContainer.BackgroundTransparency = 1
    SettingsContainer.Parent = MainFrame

    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Padding = UDim.new(0, 5)
    UIListLayout.Parent = SettingsContainer

    -- Функция создания переключателя
    local function createToggle(name, setting, callback)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(1, 0, 0, 25)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = SettingsContainer

        local ToggleLabel = Instance.new("TextLabel")
        ToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
        ToggleLabel.BackgroundTransparency = 1
        ToggleLabel.Text = name
        ToggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleLabel.Font = Enum.Font.Gotham
        ToggleLabel.TextSize = 12
        ToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
        ToggleLabel.Parent = ToggleFrame

        local ToggleBtn = Instance.new("TextButton")
        ToggleBtn.Size = UDim2.new(0, 40, 0, 20)
        ToggleBtn.Position = UDim2.new(1, -40, 0, 2)
        ToggleBtn.BackgroundColor3 = SETTINGS[setting] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        ToggleBtn.Text = SETTINGS[setting] and "ON" or "OFF"
        ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleBtn.Font = Enum.Font.GothamBold
        ToggleBtn.TextSize = 10
        ToggleBtn.Parent = ToggleFrame

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 10)
        Corner.Parent = ToggleBtn

        ToggleBtn.MouseButton1Click:Connect(function()
            SETTINGS[setting] = not SETTINGS[setting]
            ToggleBtn.BackgroundColor3 = SETTINGS[setting] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
            ToggleBtn.Text = SETTINGS[setting] and "ON" or "OFF"
            if callback then callback(SETTINGS[setting]) end
        end)
    end

    -- Функция создания слайдера
    local function createSlider(name, setting, min, max, callback)
        local SliderFrame = Instance.new("Frame")
        SliderFrame.Size = UDim2.new(1, 0, 0, 40)
        SliderFrame.BackgroundTransparency = 1
        SliderFrame.Parent = SettingsContainer

        local SliderLabel = Instance.new("TextLabel")
        SliderLabel.Size = UDim2.new(1, 0, 0, 20)
        SliderLabel.BackgroundTransparency = 1
        SliderLabel.Text = name .. ": " .. SETTINGS[setting]
        SliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        SliderLabel.Font = Enum.Font.Gotham
        SliderLabel.TextSize = 12
        SliderLabel.TextXAlignment = Enum.TextXAlignment.Left
        SliderLabel.Parent = SliderFrame

        local Slider = Instance.new("Frame")
        Slider.Size = UDim2.new(1, 0, 0, 6)
        Slider.Position = UDim2.new(0, 0, 0, 25)
        Slider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        Slider.Parent = SliderFrame

        local SliderCorner = Instance.new("UICorner")
        SliderCorner.CornerRadius = UDim.new(0, 3)
        SliderCorner.Parent = Slider

        local Fill = Instance.new("Frame")
        Fill.Size = UDim2.new((SETTINGS[setting] - min) / (max - min), 0, 1, 0)
        Fill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        Fill.Parent = Slider

        local FillCorner = Instance.new("UICorner")
        FillCorner.CornerRadius = UDim.new(0, 3)
        FillCorner.Parent = Fill

        local Dragging = false
        
        local function updateSlider(input)
            local pos = UDim2.new(math.clamp((input.Position.X - Slider.AbsolutePosition.X) / Slider.AbsoluteSize.X, 0, 1), 0, 1, 0)
            Fill.Size = pos
            local value = math.floor(min + (pos.X.Scale * (max - min)))
            SETTINGS[setting] = value
            SliderLabel.Text = name .. ": " .. value
            if callback then callback(value) end
        end

        Slider.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                Dragging = true
                updateSlider(input)
            end
        end)

        Slider.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                Dragging = false
            end
        end)

        UserInputService.InputChanged:Connect(function(input)
            if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                updateSlider(input)
            end
        end)
    end

    -- Создаем настройки
    createToggle("Чамсы", "ChamsEnabled", function(value)
        if not value then
            for player in pairs(ESPObjects) do
                removeESP(player)
            end
        end
    end)

    createToggle("Аимбот", "AimEnabled")
    createToggle("Проверка стен", "WallCheck")
    createToggle("Проверка видимости", "VisibleCheck")
    createToggle("Показывать FOV", "ShowFOV", function(value)
        if FOVCircle then
            FOVCircle.Enabled = value
        end
    end)

    createSlider("Дистанция", "MaxDistance", 50, 2000, function(value)
        SETTINGS.MaxDistance = value
    end)

    createSlider("FOV", "FOV", 10, 200, function(value)
        SETTINGS.FOV = value
        updateFOV()
    end)

    createSlider("Плавность", "Smoothness", 1, 50, function(value)
        SETTINGS.Smoothness = value / 100
    end)

    -- Выбор части тела
    local PartFrame = Instance.new("Frame")
    PartFrame.Size = UDim2.new(1, 0, 0, 30)
    PartFrame.BackgroundTransparency = 1
    PartFrame.Parent = SettingsContainer

    local PartLabel = Instance.new("TextLabel")
    PartLabel.Size = UDim2.new(0.5, 0, 1, 0)
    PartLabel.BackgroundTransparency = 1
    PartLabel.Text = "Цель:"
    PartLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    PartLabel.Font = Enum.Font.Gotham
    PartLabel.TextSize = 12
    PartLabel.TextXAlignment = Enum.TextXAlignment.Left
    PartLabel.Parent = PartFrame

    local HeadBtn = Instance.new("TextButton")
    HeadBtn.Size = UDim2.new(0, 60, 0, 20)
    HeadBtn.Position = UDim2.new(0.5, 0, 0, 5)
    HeadBtn.BackgroundColor3 = SETTINGS.AimPart == "Head" and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(60, 60, 60)
    HeadBtn.Text = "Голова"
    HeadBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    HeadBtn.Font = Enum.Font.Gotham
    HeadBtn.TextSize = 10
    HeadBtn.Parent = PartFrame

    local TorsoBtn = Instance.new("TextButton")
    TorsoBtn.Size = UDim2.new(0, 60, 0, 20)
    TorsoBtn.Position = UDim2.new(0.5, 65, 0, 5)
    TorsoBtn.BackgroundColor3 = SETTINGS.AimPart == "Torso" and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(60, 60, 60)
    TorsoBtn.Text = "Тело"
    TorsoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    TorsoBtn.Font = Enum.Font.Gotham
    TorsoBtn.TextSize = 10
    TorsoBtn.Parent = PartFrame

    local function updatePartButtons()
        HeadBtn.BackgroundColor3 = SETTINGS.AimPart == "Head" and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(60, 60, 60)
        TorsoBtn.BackgroundColor3 = SETTINGS.AimPart == "Torso" and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(60, 60, 60)
    end

    HeadBtn.MouseButton1Click:Connect(function()
        SETTINGS.AimPart = "Head"
        updatePartButtons()
    end)

    TorsoBtn.MouseButton1Click:Connect(function()
        SETTINGS.AimPart = "Torso"
        updatePartButtons()
    end)

    -- Кнопка закрыть
    CloseBtn.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
        if QuickToggleBtn then
            QuickToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        end
    end)

    GUI = ScreenGui
    return ScreenGui
end

-- Функции ESP
local function isVisible(targetPart)
    if not SETTINGS.WallCheck or not SETTINGS.VisibleCheck then return true end
    if not LocalPlayer.Character then return false end
    
    local camera = Workspace.CurrentCamera
    local origin = camera.CFrame.Position
    local target = targetPart.Position
    
    local direction = (target - origin)
    local ray = Ray.new(origin, direction.Unit * math.min(direction.Magnitude, SETTINGS.MaxDistance))
    
    local ignoreList = {LocalPlayer.Character, targetPart.Parent}
    local hit = Workspace:FindPartOnRayWithIgnoreList(ray, ignoreList)
    
    return hit == nil or hit:IsDescendantOf(targetPart.Parent)
end

local function createHighlight(part)
    local highlight = Instance.new("Highlight")
    highlight.FillTransparency = 0.3
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = part
    return highlight
end

local function createESP(player)
    local character = player.Character
    if not character then return end
    
    local highlights = {}
    local bodyParts = {"Head", "UpperTorso", "LowerTorso", "Torso", "LeftUpperArm", "RightUpperArm", "LeftArm", "RightArm", "LeftUpperLeg", "RightUpperLeg", "LeftLeg", "RightLeg"}
    
    for _, partName in pairs(bodyParts) do
        local part = character:FindFirstChild(partName)
        if part then
            local highlight = createHighlight(part)
            highlights[partName] = highlight
        end
    end
    
    ESPObjects[player] = {character = character, highlights = highlights}
end

local function removeESP(player)
    if ESPObjects[player] then
        for _, highlight in pairs(ESPObjects[player].highlights) do
            if highlight then highlight:Destroy() end
        end
        ESPObjects[player] = nil
    end
end

local function updateESPColors(player, data)
    if not SETTINGS.ChamsEnabled then
        for _, highlight in pairs(data.highlights) do
            if highlight then highlight.Enabled = false end
        end
        return
    end
    
    local character = player.Character
    if not character or not character.Parent then
        removeESP(player)
        return
    end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso")
    if not rootPart then 
        removeESP(player)
        return
    end
    
    local localRoot = LocalPlayer.Character and (LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or LocalPlayer.Character:FindFirstChild("Torso"))
    local distance = localRoot and (localRoot.Position - rootPart.Position).Magnitude or math.huge
    
    local isOutOfRange = distance > SETTINGS.MaxDistance
    
    if isOutOfRange then
        for _, highlight in pairs(data.highlights) do
            if highlight then highlight.Enabled = false end
        end
        return
    end
    
    local visible = isVisible(rootPart)
    
    for _, highlight in pairs(data.highlights) do
        if highlight and highlight.Parent then
            highlight.Enabled = true
            if visible then
                highlight.FillColor = SETTINGS.VisibleColor
                highlight.OutlineColor = SETTINGS.VisibleColor
                highlight.FillTransparency = 0.2
            else
                highlight.FillColor = SETTINGS.BehindWallColor
                highlight.OutlineColor = SETTINGS.BehindWallColor
                highlight.FillTransparency = 0.5
            end
        end
    end
end

-- Исправленный аимбот
local function getClosestPlayer()
    if not LocalPlayer.Character then return nil end
    
    local camera = Workspace.CurrentCamera
    local mousePos = UserInputService:GetMouseLocation()
    local closestPlayer = nil
    local closestDistance = SETTINGS.FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local character = player.Character
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            
            if humanoid and humanoid.Health > 0 and rootPart then
                -- Проверка команды
                if SETTINGS.TeamCheck and LocalPlayer.Team and player.Team and player.Team == LocalPlayer.Team then
                    continue
                end
                
                -- Проверка дистанции
                local distance = (rootPart.Position - camera.CFrame.Position).Magnitude
                if distance > SETTINGS.MaxDistance then
                    continue
                end
                
                -- Получаем позицию цели на экране
                local targetPart = character:FindFirstChild(SETTINGS.AimPart) or rootPart
                local screenPoint, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                
                if onScreen then
                    local mouseDistance = (Vector2.new(mousePos.X, mousePos.Y) - Vector2.new(screenPoint.X, screenPoint.Y)).Magnitude
                    
                    if mouseDistance < closestDistance then
                        -- Проверка видимости
                        if not SETTINGS.VisibleCheck or isVisible(targetPart) then
                            closestDistance = mouseDistance
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

local function aimAt(target)
    if not target or not target.Character then return end
    
    local camera = Workspace.CurrentCamera
    local targetPart = target.Character:FindFirstChild(SETTINGS.AimPart) or target.Character:FindFirstChild("HumanoidRootPart")
    if not targetPart then return end
    
    local currentCFrame = camera.CFrame
    local targetPosition = targetPart.Position + Vector3.new(0, 1.5, 0) -- Немного выше для головы
    
    -- Плавное прицеливание
    local direction = (targetPosition - currentCFrame.Position).Unit
    local newCFrame = CFrame.lookAt(currentCFrame.Position, currentCFrame.Position + direction:Lerp((targetPosition - currentCFrame.Position).Unit, SETTINGS.Smoothness))
    camera.CFrame = newCFrame
end

-- FOV Circle
local function createFOV()
    if FOVCircle then FOVCircle:Remove() end
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "FOVCircle"
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = game.CoreGui

    local Circle = Instance.new("Frame")
    Circle.Size = UDim2.new(0, SETTINGS.FOV * 2, 0, SETTINGS.FOV * 2)
    Circle.Position = UDim2.new(0.5, -SETTINGS.FOV, 0.5, -SETTINGS.FOV)
    Circle.BackgroundColor3 = SETTINGS.FOVColor
    Circle.BackgroundTransparency = 0.8
    Circle.BorderSizePixel = 0
    Circle.Visible = SETTINGS.ShowFOV
    Circle.Parent = ScreenGui

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(1, 0)
    Corner.Parent = Circle

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = SETTINGS.FOVColor
    Stroke.Thickness = 2
    Stroke.Parent = Circle

    FOVCircle = ScreenGui
end

local function updateFOV()
    if FOVCircle then
        local Circle = FOVCircle:FindFirstChildOfClass("Frame")
        if Circle then
            Circle.Size = UDim2.new(0, SETTINGS.FOV * 2, 0, SETTINGS.FOV * 2)
            Circle.Position = UDim2.new(0.5, -SETTINGS.FOV, 0.5, -SETTINGS.FOV)
            Circle.BackgroundColor3 = SETTINGS.FOVColor
            Circle.Visible = SETTINGS.ShowFOV
        end
    end
end

-- Основные циклы
local function initialize()
    -- Создаем GUI и кнопки
    createGUI()
    createQuickToggle()
    createFOV()
    
    -- ESP
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            coroutine.wrap(function()
                wait(2)
                if player.Character then createESP(player) end
                player.CharacterAdded:Connect(function() wait(2) createESP(player) end)
            end)()
        end
    end
    
    Players.PlayerAdded:Connect(function(player)
        if player == LocalPlayer then return end
        player.CharacterAdded:Connect(function() wait(2) createESP(player) end)
    end)
    
    Players.PlayerRemoving:Connect(removeESP)
    
    -- Основной цикл ESP
    RunService.Heartbeat:Connect(function()
        for player, data in pairs(ESPObjects) do
            if player and data.character and data.character.Parent then
                updateESPColors(player, data)
            else
                removeESP(player)
            end
        end
    end)
    
    -- Цикл аимбота
    RunService.RenderStepped:Connect(function()
        updateFOV()
        
        if SETTINGS.AimEnabled and UserInputService:IsMouseButtonPressed(SETTINGS.AimKey) then
            local target = getClosestPlayer()
            if target then
                aimAt(target)
            end
        end
    end)
end

-- Запуск
initialize()

print("CS 1.6 CHEAT loaded!")
print("Controls:")
print("- Click CHEAT button to open/close menu")
print("- RMB to aim")
print("- GUI to configure settings")

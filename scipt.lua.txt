-- [[ УЛУЧШЕННЫЙ HUB ДЛЯ BUILD A BOAT ]]
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer

-- Создание графического интерфейса (GUI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MyCustomHub"
ScreenGui.Parent = game:GetService("CoreGui")

-- Главная панель
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 200)
MainFrame.Position = UDim2.new(0.5, -125, 0.4, -100)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 8)
Corner.Parent = MainFrame

-- Заголовок меню
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "⚡ MY PRIVATE HUB v1.1 ⚡"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 14
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1
Title.Parent = MainFrame

-- КНОПКА 1: Полет (Fly)
local FlyButton = Instance.new("TextButton")
FlyButton.Size = UDim2.new(0, 210, 0, 40)
FlyButton.Position = UDim2.new(0, 20, 0, 55)
FlyButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
FlyButton.Text = "Fly: OFF"
FlyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FlyButton.Font = Enum.Font.Gotham
FlyButton.TextSize = 14
FlyButton.Parent = MainFrame

local FlyCorner = Instance.new("UICorner")
FlyCorner.CornerRadius = UDim.new(0, 6)
FlyCorner.Parent = FlyButton

-- КНОПКА 2: Авто-Ферма Золота (Auto-Farm)
local FarmButton = Instance.new("TextButton")
FarmButton.Size = UDim2.new(0, 210, 0, 40)
FarmButton.Position = UDim2.new(0, 20, 0, 110)
FarmButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
FarmButton.Text = "Auto-Farm: OFF"
FarmButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FarmButton.Font = Enum.Font.Gotham
FarmButton.TextSize = 14
FarmButton.Parent = MainFrame

local FarmCorner = Instance.new("UICorner")
FarmCorner.CornerRadius = UDim.new(0, 6)
FarmCorner.Parent = FarmButton

-- Переменные логики
local isFlying = false
local isFarming = false
local flySpeed = 70 -- Скорость полета

-- Физические объекты для плавного полета
local bVelocity, bGyro
local flyConnection

-- Функция включения/выключения полета
local function toggleFly(state)
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart
    local hum = char.Humanoid

    if state then
        -- Создаем физический контроль скорости
        bVelocity = Instance.new("BodyVelocity")
        bVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bVelocity.Velocity = Vector3.new(0, 0, 0)
        bVelocity.Parent = root

        -- Создаем стабилизатор направления (чтобы не крутило)
        bGyro = Instance.new("BodyGyro")
        bGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bGyro.CFrame = root.CFrame
        bGyro.Parent = root

        -- Поток обработки движений
        flyConnection = RunService.RenderStepped:Connect(function()
            if not isFlying or not char or not root:IsDescendantOf(workspace) then 
                toggleFly(false)
                return 
            end
            
            local camera = workspace.CurrentCamera
            local moveDir = hum.MoveDirection
            local newVelocity = Vector3.new(0, 0, 0)

            -- Если игрок идет, направляем силу в сторону камеры
            if moveDir.Magnitude > 0 then
                local camCFrame = camera.CFrame
                local lookVectors = camCFrame.LookVector * moveDir.Z
                local rightVectors = camCFrame.RightVector * moveDir.X
                newVelocity = (lookVectors + rightVectors).Unit * flySpeed
            end

            bVelocity.Velocity = newVelocity
            bGyro.CFrame = camera.CFrame -- Персонаж плавно поворачивается за камерой
        end)
    else
        -- Чистим за собой при выключении
        if bVelocity then bVelocity:Destroy() bVelocity = nil end
        if bGyro then bGyro:Destroy() bGyro = nil end
        if flyConnection then flyConnection:Disconnect() flyConnection = nil end
    end
end

-- ЛОГИКА КНОПКИ ПОЛЕТА
FlyButton.MouseButton1Click:Connect(function()
    isFlying = not isFlying
    if isFlying then
        FlyButton.Text = "Fly: ON"
        FlyButton.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
        toggleFly(true)
    else
        FlyButton.Text = "Fly: OFF"
        FlyButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
        toggleFly(false)
    end
end)

-- Обновление полета при респавне персонажа
player.CharacterAdded:Connect(function()
    task.wait(0.5)
    if isFlying then
        toggleFly(false)
        toggleFly(true)
    end
end)

-- ЛОГИКА АВТО-ФЕРМЫ
FarmButton.MouseButton1Click:Connect(function()
    isFarming = not isFarming
    if isFarming then
        FarmButton.Text = "Auto-Farm: ON"
        FarmButton.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
        
        task.spawn(function()
            while isFarming do
                local char = player.Character
                if char and char:FindFirstChild("HumanoidRootPart") then
                    -- Отключаем коллизию блоков
                    for _, part in pairs(char:GetChildren()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                    
                    -- Телепорт по стадиям
                    for i = 1, 10 do
                        if not isFarming then break end
                        local stage = workspace:FindFirstChild("TheStages") and workspace.TheStages:FindFirstChild("Stage" .. i)
                        if stage and stage:FindFirstChild("CaveStart") then
                            local tween = TweenService:Create(char.HumanoidRootPart, TweenInfo.new(1.2, Enum.EasingStyle.Linear), {CFrame = stage.CaveStart.CFrame})
                            tween:Play()
                            tween.Completed:Wait()
                        end
                    end
                    
                    -- Телепорт к сундуку
                    if isFarming and workspace:FindFirstChild("BoatStages") and workspace.BoatStages:FindFirstChild("NormalStages") then
                        local endStage = workspace.BoatStages.NormalStages:FindFirstChild("TheEnd")
                        if endStage and endStage:FindFirstChild("GoldenChest") then
                            char.HumanoidRootPart.CFrame = endStage.GoldenChest.Trigger.CFrame
                            task.wait(5)
                        end
                    end
                end
                task.wait(1)
            end
        end)
    else
        FarmButton.Text = "Auto-Farm: OFF"
        FarmButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    end
end)

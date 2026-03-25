-- ==================================================
-- Delta Security Test - نسخة متوافقة مع Delta متصفح
-- للاستخدام في بيئة الاختبار والتطوير فقط
-- ==================================================

-- لا توجد حماية لمنع التشغيل على Delta
-- هذا السكربت مصمم للاختبار الأمني

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- ==================================================
-- تكوين السكربت (يمكن تعديله)
-- ==================================================
local Config = {
    -- تفعيل الميزات
    Features = {
        Aimbot = true,
        SpeedHack = true,
        FlyHack = false,
        ESP = true,
        AutoFarm = false
    },
    
    -- إعدادات Aimbot
    Aimbot = {
        Enabled = true,
        TargetPart = "Head",
        FOV = 200,
        Smoothness = 5,
        ShowFOVCircle = true
    },
    
    -- إعدادات Speed Hack
    SpeedHack = {
        SpeedValue = 80,
        OriginalSpeed = 16
    },
    
    -- إعدادات Fly Hack
    FlyHack = {
        FlySpeed = 60
    },
    
    -- إعدادات Auto Farm
    AutoFarm = {
        Range = 50,
        AttackDelay = 0.5
    },
    
    -- المفاتيح السريعة
    Keybinds = {
        ToggleAimbot = Enum.KeyCode.Q,
        ToggleSpeed = Enum.KeyCode.E,
        ToggleFly = Enum.KeyCode.F,
        ToggleESP = Enum.KeyCode.V
    }
}

-- ==================================================
-- المتغيرات العامة
-- ==================================================
local State = {
    Flying = false,
    FlyBodyVelocity = nil,
    OriginalGravity = workspace.Gravity,
    ESPObjects = {},
    LastAttack = 0,
    AimbotActive = true,
    SpeedActive = true,
    FlyActive = false,
    ESPActive = true,
    LogEvents = {}  -- لتسجيل الأحداث للتحليل
}

-- ==================================================
-- وظيفة تسجيل الأحداث (لتحليل الأمان)
-- ==================================================
local function LogEvent(eventType, details)
    local eventData = {
        type = eventType,
        time = tick(),
        details = details or {}
    }
    table.insert(State.LogEvents, eventData)
    
    -- الاحتفاظ بآخر 100 حدث فقط
    if #State.LogEvents > 100 then
        table.remove(State.LogEvents, 1)
    end
    
    -- طباعة في Console للاختبار
    print(string.format("[Delta Test] %s: %s", eventType, tostring(details)))
end

-- ==================================================
-- واجهة المستخدم (GUI) - تعمل على Delta
-- ==================================================
local function CreateDeltaMenu()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DeltaSecurityTest"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    
    -- الإطار الرئيسي (بألوان Delta المميزة)
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 280, 0, 400)
    mainFrame.Position = UDim2.new(0, 10, 0, 10)
    mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    -- شريط العنوان (قابل للسحب)
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.BackgroundColor3 = Color3.fromRGB(255, 70, 70)
    titleBar.BackgroundTransparency = 0.3
    titleBar.Parent = mainFrame
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 1, 0)
    title.Text = "🔴 Delta Security Test | Blox Fruits"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 12
    title.Parent = titleBar
    
    -- جعل الإطار قابل للسحب
    local function makeDraggable(frame)
        local dragging, dragInput, dragStart, startPos
        
        frame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                dragStart = input.Position
                startPos = mainFrame.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                    end
                end)
            end
        end)
        
        frame.InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                local delta = input.Position - dragStart
                mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                                                startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)
    end
    
    makeDraggable(titleBar)
    
    -- محتوى القائمة
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, 0, 1, -35)
    content.Position = UDim2.new(0, 0, 0, 35)
    content.BackgroundTransparency = 1
    content.Parent = mainFrame
    
    local yOffset = 5
    local function addButton(text, yPos, callback, color)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.9, 0, 0, 32)
        btn.Position = UDim2.new(0.05, 0, 0, yPos)
        btn.Text = text
        btn.BackgroundColor3 = color or Color3.fromRGB(35, 35, 50)
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 12
        btn.Parent = content
        
        btn.MouseButton1Click:Connect(callback)
        return btn
    end
    
    local function addToggle(text, yPos, getState, setState)
        local btn = addButton(text, yPos, nil, Color3.fromRGB(35, 35, 50))
        
        local function update()
            if getState() then
                btn.Text = "✅ " .. text
                btn.BackgroundColor3 = Color3.fromRGB(70, 50, 50)
            else
                btn.Text = "⬜ " .. text
                btn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
            end
        end
        
        btn.MouseButton1Click:Connect(function()
            setState(not getState())
            update()
            LogEvent("toggle", {feature = text, state = getState()})
        end)
        
        update()
        return btn
    end
    
    -- أزرار التبديل
    addToggle("Aimbot [Q]", yOffset, function() return State.AimbotActive end, function(v) 
        State.AimbotActive = v 
        Config.Features.Aimbot = v
    end)
    yOffset = yOffset + 38
    
    addToggle("Speed Hack [E]", yOffset, function() return State.SpeedActive end, function(v) 
        State.SpeedActive = v 
        Config.Features.SpeedHack = v
    end)
    yOffset = yOffset + 38
    
    addToggle("Fly Hack [F]", yOffset, function() return State.FlyActive end, function(v) 
        State.FlyActive = v 
        Config.Features.FlyHack = v
    end)
    yOffset = yOffset + 38
    
    addToggle("ESP [V]", yOffset, function() return State.ESPActive end, function(v) 
        State.ESPActive = v 
        Config.Features.ESP = v
        if not v then
            for _, obj in ipairs(State.ESPObjects) do
                pcall(function() obj:Destroy() end)
            end
            State.ESPObjects = {}
        end
    end)
    yOffset = yOffset + 45
    
    -- خط فاصل
    local line = Instance.new("Frame")
    line.Size = UDim2.new(0.9, 0, 0, 1)
    line.Position = UDim2.new(0.05, 0, 0, yOffset - 5)
    line.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    line.Parent = content
    yOffset = yOffset + 10
    
    -- زر عرض التقرير
    addButton("📊 عرض تقرير الأمان", yOffset, function()
        print("\n========== تقرير اختبار Delta ==========")
        print("عدد الأحداث المسجلة:", #State.LogEvents)
        print("آخر 10 أحداث:")
        for i = math.max(1, #State.LogEvents - 9), #State.LogEvents do
            local e = State.LogEvents[i]
            print(string.format("  %d. %s - %s", i, e.type, tostring(e.details):sub(1, 50)))
        end
        print("=========================================\n")
    end, Color3.fromRGB(50, 70, 50))
    yOffset = yOffset + 38
    
    -- زر إعادة الضبط
    addButton("🔄 إعادة ضبط", yOffset, function()
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChild("Humanoid")
            if hum then
                hum.WalkSpeed = Config.SpeedHack.OriginalSpeed
                hum.JumpPower = 50
            end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.Velocity = Vector3.new(0, 0, 0)
            end
        end
        if State.FlyBodyVelocity then
            State.FlyBodyVelocity:Destroy()
            State.FlyBodyVelocity = nil
        end
        workspace.Gravity = State.OriginalGravity
        State.Flying = false
        LogEvent("reset", "تم إعادة ضبط اللاعب")
        print("[Delta Test] تم إعادة الضبط")
    end, Color3.fromRGB(70, 50, 50))
    yOffset = yOffset + 38
    
    -- زر الإغلاق
    addButton("✖ إغلاق", yOffset, function()
        screenGui:Destroy()
        LogEvent("menu_closed", "تم إغلاق واجهة الاختبار")
    end, Color3.fromRGB(80, 40, 40))
    
    return screenGui
end

-- ==================================================
-- Aimbot System (نسخة متوافقة مع Delta)
-- ==================================================
local function GetClosestEnemy()
    if not Config.Features.Aimbot then return nil end
    
    local closestDist = Config.Aimbot.FOV
    local closest = nil
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hum = player.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then
                local targetPart = player.Character:FindFirstChild(Config.Aimbot.TargetPart) or
                                   player.Character:FindFirstChild("HumanoidRootPart")
                if targetPart then
                    local screenPos, onScreen = Camera:WorldToScreenPoint(targetPart.Position)
                    if onScreen then
                        local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                        local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                        if dist < closestDist then
                            closestDist = dist
                            closest = targetPart
                        end
                    end
                end
            end
        end
    end
    
    return closest
end

local function AimAt(target)
    if not target then return end
    
    local screenPos = Camera:WorldToScreenPoint(target.Position)
    if screenPos then
        -- Delta متصفح: استخدام mousemoverel للتحكم بالماوس
        local mouse = LocalPlayer:GetMouse()
        if mouse and mouse.mousemoverel then
            local centerX = Camera.ViewportSize.X / 2
            local centerY = Camera.ViewportSize.Y / 2
            local deltaX = screenPos.X - centerX
            local deltaY = screenPos.Y - centerY
            
            -- تطبيق Smoothness لتجنب الحركة المفاجئة
            local smooth = Config.Aimbot.Smoothness
            mouse.mousemoverel(deltaX / smooth, deltaY / smooth)
            
            LogEvent("aimbot", {target = target.Parent and target.Parent.Name, part = target.Name})
        end
    end
end

-- ==================================================
-- Speed Hack System
-- ==================================================
local function ApplySpeedHack()
    if not Config.Features.SpeedHack then return end
    
    local char = LocalPlayer.Character
    if not char then return end
    
    local hum = char:FindFirstChild("Humanoid")
    if hum and hum.WalkSpeed ~= Config.SpeedHack.SpeedValue then
        hum.WalkSpeed = Config.SpeedHack.SpeedValue
        LogEvent("speed_hack", {speed = Config.SpeedHack.SpeedValue})
    end
end

-- ==================================================
-- Fly Hack System (متوافق مع Delta)
-- ==================================================
local function ApplyFlyHack()
    if not Config.Features.FlyHack then
        if State.Flying then
            -- إيقاف الطيران
            if State.FlyBodyVelocity then
                State.FlyBodyVelocity:Destroy()
                State.FlyBodyVelocity = nil
            end
            local char = LocalPlayer.Character
            if char then
                local hum = char:FindFirstChild("Humanoid")
                if hum then hum.PlatformStand = false end
            end
            workspace.Gravity = State.OriginalGravity
            State.Flying = false
            LogEvent("fly_hack", {active = false})
        end
        return
    end
    
    local char = LocalPlayer.Character
    if not char then return end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    
    if not State.Flying then
        State.Flying = true
        
        if hrp then
            State.FlyBodyVelocity = Instance.new("BodyVelocity")
            State.FlyBodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
            State.FlyBodyVelocity.Velocity = Vector3.new(0, Config.FlyHack.FlySpeed, 0)
            State.FlyBodyVelocity.Parent = hrp
        end
        if hum then hum.PlatformStand = true end
        workspace.Gravity = 0
        
        LogEvent("fly_hack", {active = true, speed = Config.FlyHack.FlySpeed})
    end
end

-- ==================================================
-- Auto Farm System (محاكاة)
-- ==================================================
local function ApplyAutoFarm()
    if not Config.Features.AutoFarm then return end
    
    local char = LocalPlayer.Character
    if not char then return end
    
    local now = tick()
    if now - State.LastAttack >= Config.AutoFarm.AttackDelay then
        State.LastAttack = now
        LogEvent("auto_farm", {position = char.HumanoidRootPart and char.HumanoidRootPart.Position})
    end
end

-- ==================================================
-- ESP System (متوافق مع Delta)
-- ==================================================
local function UpdateESP()
    if not Config.Features.ESP then return end
    
    -- تنظيف ESP القديم
    for _, obj in ipairs(State.ESPObjects) do
        pcall(function() obj:Destroy() end)
    end
    State.ESPObjects = {}
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                -- إنشاء دائرة حول اللاعب
                local circle = Instance.new("BillboardGui")
                circle.Size = UDim2.new(0, 40, 0, 40)
                circle.AlwaysOnTop = true
                circle.StudsOffset = Vector3.new(0, 2, 0)
                
                local frame = Instance.new("Frame")
                frame.Size = UDim2.new(1, 0, 1, 0)
                frame.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                frame.BackgroundTransparency = 0.5
                frame.BorderSizePixel = 0
                frame.Parent = circle
                
                -- إنشاء نص باسم اللاعب
                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(0, 100, 0, 20)
                label.Position = UDim2.new(0.5, -50, 1, 5)
                label.Text = player.Name
                label.TextColor3 = Color3.fromRGB(255, 255, 255)
                label.BackgroundTransparency = 1
                label.TextScaled = true
                label.Font = Enum.Font.GothamBold
                label.Parent = circle
                
                circle.Parent = hrp
                table.insert(State.ESPObjects, circle)
            end
        end
    end
end

-- ==================================================
-- رسم دائرة FOV (اختياري)
-- ==================================================
local function DrawFOVCircle()
    if not Config.Aimbot.ShowFOVCircle then return end
    
    -- بحث عن دائرة موجودة
    local existing = LocalPlayer.PlayerGui:FindFirstChild("FOVCircle")
    if existing then existing:Destroy() end
    
    local circleGui = Instance.new("ScreenGui")
    circleGui.Name = "FOVCircle"
    circleGui.Parent = LocalPlayer.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, Config.Aimbot.FOV * 2, 0, Config.Aimbot.FOV * 2)
    frame.Position = UDim2.new(0.5, -Config.Aimbot.FOV, 0.5, -Config.Aimbot.FOV)
    frame.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    frame.BackgroundTransparency = 0.8
    frame.BorderSizePixel = 0
    frame.Parent = circleGui
    
    -- جعل الدائرة دائرية
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = frame
end

-- ==================================================
-- المفاتيح السريعة
-- ==================================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local key = input.KeyCode
    
    if key == Config.Keybinds.ToggleAimbot then
        State.AimbotActive = not State.AimbotActive
        Config.Features.Aimbot = State.AimbotActive
        LogEvent("keybind", {key = "Aimbot", state = State.AimbotActive})
        
    elseif key == Config.Keybinds.ToggleSpeed then
        State.SpeedActive = not State.SpeedActive
        Config.Features.SpeedHack = State.SpeedActive
        LogEvent("keybind", {key = "Speed", state = State.SpeedActive})
        
    elseif key == Config.Keybinds.ToggleFly then
        State.FlyActive = not State.FlyActive
        Config.Features.FlyHack = State.FlyActive
        LogEvent("keybind", {key = "Fly", state = State.FlyActive})
        
    elseif key == Config.Keybinds.ToggleESP then
        State.ESPActive = not State.ESPActive
        Config.Features.ESP = State.ESPActive
        LogEvent("keybind", {key = "ESP", state = State.ESPActive})
        
        if not State.ESPActive then
            for _, obj in ipairs(State.ESPObjects) do
                pcall(function() obj:Destroy() end)
            end
            State.ESPObjects = {}
        end
    end
end)

-- ==================================================
-- الحلقة الرئيسية
-- ==================================================
-- تحديث ESP كل ثانية
task.spawn(function()
    while true do
        task.wait(1)
        if Config.Features.ESP then
            UpdateESP()
        end
        if Config.Aimbot.ShowFOVCircle then
            DrawFOVCircle()
        end
    end
end)

-- حلقة RenderStepped للحركات السريعة
RunService.RenderStepped:Connect(function()
    -- Aimbot
    if Config.Features.Aimbot then
        local target = GetClosestEnemy()
        if target then
            AimAt(target)
        end
    end
    
    -- Speed Hack
    if Config.Features.SpeedHack then
        ApplySpeedHack()
    end
    
    -- Fly Hack
    if Config.Features.FlyHack then
        ApplyFlyHack()
    end
    
    -- Auto Farm
    if Config.Features.AutoFarm then
        ApplyAutoFarm()
    end
end)

-- إعادة تطبيق الإعدادات عند إعادة الظهور
LocalPlayer.CharacterAdded:Connect(function(character)
    task.wait(2)
    if Config.Features.SpeedHack then
        local hum = character:FindFirstChild("Humanoid")
        if hum then
            hum.WalkSpeed = Config.SpeedHack.SpeedValue
        end
    end
    LogEvent("respawn", "تم إعادة ظهور اللاعب")
end)

-- ==================================================
-- بدء التشغيل
-- ==================================================
print("\n========== Delta Security Test ==========")
print("تم تحميل سكربت الاختبار الأمني")
print("متوافق مع Delta متصفح")
print("\nالمفاتيح السريعة:")
print("  [Q] - Aimbot")
print("  [E] - Speed Hack")
print("  [F] - Fly Hack")
print("  [V] - ESP")
print("\nلإنشاء تق

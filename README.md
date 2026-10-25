local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- حذف الواجهة القديمة إن وجدت
if CoreGui:FindFirstChild("YassinSavannahGUI") then
    CoreGui.YassinSavannahGUI:Destroy()
end

-- إنشاء الواجهة
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "YassinSavannahGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 240, 0, 280)
MainFrame.Position = UDim2.new(0.5, -120, 0.5, -140)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Text = "YASSIN - Savannah Life"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 15
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

local function createButton(name, posY, text)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(0.85, 0, 0, 35)
    btn.Position = UDim2.new(0.075, 0, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamSemibold
    btn.Parent = MainFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    return btn
end

local AutoWaterBtn = createButton("AutoWaterBtn", 45, "شرب تلقائي (بدون رفع رأس): معطل")
local AutoFoodBtn = createButton("AutoFoodBtn", 90, "أكل تلقائي (بدون رفع رأس): معطل")
local GazelleBtn = createButton("GazelleBtn", 135, "وضع الغزال السريع: معطل")

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0.85, 0, 0, 40)
ToggleBtn.Position = UDim2.new(0.075, 0, 0, 180)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
ToggleBtn.Text = "إغلاق القائمة"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 14
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.Parent = MainFrame

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 6)
BtnCorner.Parent = ToggleBtn

local function getCharacterParts()
    local char = LocalPlayer.Character
    if not char then
        return nil, nil, nil
    end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local head = char:FindFirstChild("Head")
    local neck = head and head:FindFirstChild("Neck")

    return hum, head, neck
end

-- إيقاف الأنيميشن والرقبة
local function lockHeadPosition()
    local hum, head, neck = getCharacterParts()
    if not hum or not head then
        return
    end

    local animator = hum:FindFirstChildOfClass("Animator")
    if animator then
        for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
            local animId = track.Animation and track.Animation.AnimationId:lower() or ""
            if animId:find("eat") or animId:find("drink") or animId:find("head") or animId:find("neck") then
                track:Stop(0)
            end
        end
    end

    if neck and neck:IsA("Motor6D") then
        -- تعيين رقبة مستقيمة
        neck.C0 = CFrame.new(0, 0.5, 0) * CFrame.Angles(0, 0, 0)
    end
end

-- تشغيل قفل الرأس بشكل مستمر إذا كان التفعيل فعالاً
local headLockLoop = nil

local function startHeadLockLoop()
    if headLockLoop then
        return
    end

    headLockLoop = RunService.RenderStepped:Connect(function()
        lockHeadPosition()
    end)
end

local function stopHeadLockLoop()
    if headLockLoop then
        headLockLoop:Disconnect()
        headLockLoop = nil
    end
end

-- 1. شرب تلقائي
local autoWaterActive = false
AutoWaterBtn.MouseButton1Click:Connect(function()
    autoWaterActive = not autoWaterActive

    if autoWaterActive then
        AutoWaterBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        AutoWaterBtn.Text = "شرب تلقائي (بدون رفع رأس): مفعل"
        startHeadLockLoop()
        task.spawn(function()
            while autoWaterActive do
                lockHeadPosition()
                task.wait(0.15)
            end
            stopHeadLockLoop()
        end)
    else
        AutoWaterBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        AutoWaterBtn.Text = "شرب تلقائي (بدون رفع رأس): معطل"
    end
end)

-- 2. أكل تلقائي
local autoFoodActive = false
AutoFoodBtn.MouseButton1Click:Connect(function()
    autoFoodActive = not autoFoodActive

    if autoFoodActive then
        AutoFoodBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        AutoFoodBtn.Text = "أكل تلقائي (بدون رفع رأس): مفعل"
        startHeadLockLoop()
        task.spawn(function()
            while autoFoodActive do
                lockHeadPosition()
                task.wait(0.15)
            end
            stopHeadLockLoop()
        end)
    else
        AutoFoodBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        AutoFoodBtn.Text = "أكل تلقائي (بدون رفع رأس): معطل"
    end
end)

-- 3. وضع الغزال
local gazelleActive = false
GazelleBtn.MouseButton1Click:Connect(function()
    gazelleActive = not gazelleActive

    if gazelleActive then
        GazelleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        GazelleBtn.Text = "وضع الغزال السريع: مفعل"

        task.spawn(function()
            while gazelleActive do
                pcall(function()
                    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                        LocalPlayer.Character.Humanoid.WalkSpeed = 35
                    end
                end)
                task.wait(0.5)
            end
        end)
    else
        GazelleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        GazelleBtn.Text = "وضع الغزال السريع: معطل"
        pcall(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid.WalkSpeed = 16
            end
        end)
    end
end)

-- 4. إظهار/إخفاء القائمة
local isOpen = true
ToggleBtn.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    Title.Visible = isOpen
    AutoWaterBtn.Visible = isOpen
    AutoFoodBtn.Visible = isOpen
    GazelleBtn.Visible = isOpen

    if isOpen then
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        ToggleBtn.Text = "إغلاق القائمة"
        MainFrame.Size = UDim2.new(0, 240, 0, 280)
        ToggleBtn.Position = UDim2.new(0.075, 0, 0, 180)
    else
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
        ToggleBtn.Text = "فتح القائمة"
        MainFrame.Size = UDim2.new(0, 240, 0, 55)
        ToggleBtn.Position = UDim2.new(0.075, 0, 0, 8)
    end
end)

-- ========================================
-- SYLAX KEY SYSTEM
-- ========================================

local Services = {
    Players = game:GetService("Players"),
    TweenService = game:GetService("TweenService"),
    UserInputService = game:GetService("UserInputService"),
    RunService = game:GetService("RunService")
}

local Player = Services.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local DISCORD_LINK = "https://discord.gg/nnxRBba7d"
local DISCORD_IMAGE = "rbxassetid://95649005720075"
local ICON_IMAGE = "rbxassetid://86801451021941"
local API_URL = "https://sylax-key.onrender.com/check?key="

local Config = {
    MaxKeyLength = 50,
    ParticleCount = 40,
    ParticleSpeed = 60,
    ValidateTime = 5
}

local Colors = {
    Background    = Color3.fromRGB(18, 10, 28),
    Surface       = Color3.fromRGB(28, 14, 42),
    Primary       = Color3.fromRGB(80, 30, 100),
    Border        = Color3.fromRGB(255, 255, 255),
    TextPrimary   = Color3.fromRGB(240, 220, 255),
    TextSecondary = Color3.fromRGB(180, 140, 200),
    Success       = Color3.fromRGB(80, 200, 120),
    Error         = Color3.fromRGB(220, 60, 80),
    Warning       = Color3.fromRGB(200, 120, 30),
    Discord       = Color3.fromRGB(88, 101, 242),
    GetKey        = Color3.fromRGB(120, 30, 100),
    HoverPrimary  = Color3.fromRGB(110, 50, 140),
    HoverDiscord  = Color3.fromRGB(70, 82, 210),
    HoverGetKey   = Color3.fromRGB(150, 50, 130),
    Glow1         = Color3.fromRGB(180, 60, 220),
    Glow2         = Color3.fromRGB(220, 80, 140),
}

local State = {
    IsLoading = false,
    Particles = {},
    IsDestroyed = false,
    MousePosition = {X = 0, Y = 0},
}

local UI = {}

local old = PlayerGui:FindFirstChild("SylaxKeyGUI")
if old then old:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SylaxKeyGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.DisplayOrder = 100
screenGui.Parent = PlayerGui
UI.ScreenGui = screenGui

local backdrop = Instance.new("Frame")
backdrop.Size = UDim2.new(1, 0, 1, 0)
backdrop.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
backdrop.BackgroundTransparency = 0.15
backdrop.BorderSizePixel = 0
backdrop.ZIndex = 100
backdrop.Parent = screenGui
UI.Backdrop = backdrop

local particleContainer = Instance.new("Frame")
particleContainer.Size = UDim2.new(1, 0, 1, 0)
particleContainer.BackgroundTransparency = 1
particleContainer.ZIndex = 101
particleContainer.Parent = backdrop
UI.ParticleContainer = particleContainer

local container = Instance.new("Frame")
container.Name = "MainContainer"
container.Size = UDim2.new(0, 340, 0, 460)
container.Position = UDim2.new(0.5, -170, 0.5, -230)
container.BackgroundColor3 = Colors.Background
container.BorderSizePixel = 0
container.ZIndex = 110
container.Parent = screenGui
UI.Container = container
Instance.new("UICorner", container).CornerRadius = UDim.new(0, 18)

local borderFrame = Instance.new("Frame")
borderFrame.Size = UDim2.new(1, 6, 1, 6)
borderFrame.Position = UDim2.new(0, -3, 0, -3)
borderFrame.BackgroundTransparency = 1
borderFrame.ZIndex = 109
borderFrame.Parent = container
Instance.new("UICorner", borderFrame).CornerRadius = UDim.new(0, 21)

local borderStroke = Instance.new("UIStroke")
borderStroke.Color = Colors.Border
borderStroke.Thickness = 2
borderStroke.Transparency = 0.1
borderStroke.Parent = borderFrame

local borderGradient = Instance.new("UIGradient")
borderGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0,   Colors.Glow1),
    ColorSequenceKeypoint.new(0.5, Colors.Border),
    ColorSequenceKeypoint.new(1,   Colors.Glow2),
}
borderGradient.Transparency = NumberSequence.new{
    NumberSequenceKeypoint.new(0,    0.7),
    NumberSequenceKeypoint.new(0.25, 0),
    NumberSequenceKeypoint.new(0.75, 0),
    NumberSequenceKeypoint.new(1,    0.7),
}
borderGradient.Parent = borderStroke
UI.BorderGradient = borderGradient

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -38, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(60, 20, 70)
closeBtn.BorderSizePixel = 0
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.AutoButtonColor = false
closeBtn.ZIndex = 120
closeBtn.Parent = container
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

closeBtn.MouseEnter:Connect(function()
    Services.TweenService:Create(closeBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(180, 40, 40)}):Play()
end)
closeBtn.MouseLeave:Connect(function()
    Services.TweenService:Create(closeBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(60, 20, 70)}):Play()
end)
closeBtn.MouseButton1Click:Connect(function()
    State.IsDestroyed = true
    Services.TweenService:Create(container, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)}):Play()
    Services.TweenService:Create(backdrop, TweenInfo.new(0.35), {BackgroundTransparency = 1}):Play()
    task.wait(0.4)
    screenGui:Destroy()
end)

local iconContainer = Instance.new("Frame")
iconContainer.Size = UDim2.new(0, 52, 0, 52)
iconContainer.Position = UDim2.new(0.5, -26, 0, 22)
iconContainer.BackgroundColor3 = Colors.Primary
iconContainer.BorderSizePixel = 0
iconContainer.ZIndex = 112
iconContainer.Parent = container
Instance.new("UICorner", iconContainer).CornerRadius = UDim.new(0, 13)

local iconGlowFrame = Instance.new("Frame")
iconGlowFrame.Size = UDim2.new(1, 14, 1, 14)
iconGlowFrame.Position = UDim2.new(0, -7, 0, -7)
iconGlowFrame.BackgroundTransparency = 1
iconGlowFrame.ZIndex = 111
iconGlowFrame.Parent = iconContainer
Instance.new("UICorner", iconGlowFrame).CornerRadius = UDim.new(0, 19)

local iconGlowStroke = Instance.new("UIStroke")
iconGlowStroke.Color = Colors.Glow1
iconGlowStroke.Thickness = 2.5
iconGlowStroke.Transparency = 0.1
iconGlowStroke.Parent = iconGlowFrame

local iconGlowGrad = Instance.new("UIGradient")
iconGlowGrad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0,   Colors.Glow1),
    ColorSequenceKeypoint.new(0.5, Colors.Glow2),
    ColorSequenceKeypoint.new(1,   Colors.Glow1),
}
iconGlowGrad.Transparency = NumberSequence.new{
    NumberSequenceKeypoint.new(0,    0.8),
    NumberSequenceKeypoint.new(0.25, 0),
    NumberSequenceKeypoint.new(0.75, 0),
    NumberSequenceKeypoint.new(1,    0.8),
}
iconGlowGrad.Parent = iconGlowStroke
UI.IconGlowGrad = iconGlowGrad

local iconImage = Instance.new("ImageLabel")
iconImage.Size = UDim2.new(0.82, 0, 0.82, 0)
iconImage.Position = UDim2.new(0.09, 0, 0.09, 0)
iconImage.BackgroundTransparency = 1
iconImage.Image = ICON_IMAGE
iconImage.ScaleType = Enum.ScaleType.Fit
iconImage.ZIndex = 113
iconImage.Parent = iconContainer

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -20, 0, 30)
titleLabel.Position = UDim2.new(0, 10, 0, 82)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "S Y L A X"
titleLabel.TextColor3 = Colors.TextPrimary
titleLabel.TextSize = 22
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Center
titleLabel.ZIndex = 112
titleLabel.Parent = container

local titleGrad = Instance.new("UIGradient")
titleGrad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0,   Colors.Glow2),
    ColorSequenceKeypoint.new(0.5, Colors.TextPrimary),
    ColorSequenceKeypoint.new(1,   Colors.Glow1),
}
titleGrad.Parent = titleLabel

local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Size = UDim2.new(1, -20, 0, 22)
subtitleLabel.Position = UDim2.new(0, 10, 0, 114)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Text = "Enter your access key to continue"
subtitleLabel.TextColor3 = Colors.TextSecondary
subtitleLabel.TextSize = 13
subtitleLabel.Font = Enum.Font.Gotham
subtitleLabel.TextXAlignment = Enum.TextXAlignment.Center
subtitleLabel.ZIndex = 112
subtitleLabel.Parent = container

local inputContainer = Instance.new("Frame")
inputContainer.Size = UDim2.new(1, -36, 0, 46)
inputContainer.Position = UDim2.new(0, 18, 0, 150)
inputContainer.BackgroundColor3 = Colors.Surface
inputContainer.BorderSizePixel = 0
inputContainer.ZIndex = 112
inputContainer.Parent = container
Instance.new("UICorner", inputContainer).CornerRadius = UDim.new(0, 10)

local inputStroke = Instance.new("UIStroke")
inputStroke.Color = Colors.Primary
inputStroke.Thickness = 1.5
inputStroke.Transparency = 0.3
inputStroke.Parent = inputContainer
UI.InputStroke = inputStroke

local textInput = Instance.new("TextBox")
textInput.Size = UDim2.new(1, -20, 1, 0)
textInput.Position = UDim2.new(0, 10, 0, 0)
textInput.BackgroundTransparency = 1
textInput.Text = ""
textInput.PlaceholderText = "Enter key here..."
textInput.TextColor3 = Colors.TextPrimary
textInput.PlaceholderColor3 = Colors.TextSecondary
textInput.TextSize = 15
textInput.Font = Enum.Font.Gotham
textInput.TextXAlignment = Enum.TextXAlignment.Left
textInput.ClearTextOnFocus = false
textInput.ZIndex = 113
textInput.Parent = inputContainer
UI.TextInput = textInput

local charCounter = Instance.new("TextLabel")
charCounter.Size = UDim2.new(0, 70, 0, 16)
charCounter.Position = UDim2.new(1, -88, 0, 200)
charCounter.BackgroundTransparency = 1
charCounter.Text = "0/50"
charCounter.TextColor3 = Colors.TextSecondary
charCounter.TextSize = 11
charCounter.Font = Enum.Font.Gotham
charCounter.TextXAlignment = Enum.TextXAlignment.Right
charCounter.ZIndex = 112
charCounter.Parent = container
UI.CharCounter = charCounter

local submitBtn = Instance.new("TextButton")
submitBtn.Size = UDim2.new(1, -36, 0, 44)
submitBtn.Position = UDim2.new(0, 18, 0, 216)
submitBtn.BackgroundColor3 = Colors.Primary
submitBtn.BorderSizePixel = 0
submitBtn.Text = "Verify Access Key"
submitBtn.TextColor3 = Colors.TextPrimary
submitBtn.TextSize = 15
submitBtn.Font = Enum.Font.GothamMedium
submitBtn.AutoButtonColor = false
submitBtn.ZIndex = 112
submitBtn.Parent = container
Instance.new("UICorner", submitBtn).CornerRadius = UDim.new(0, 10)
UI.SubmitBtn = submitBtn

local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(1, 0, 1, 0)
progressBg.BackgroundColor3 = Color3.fromRGB(50, 15, 65)
progressBg.BackgroundTransparency = 1
progressBg.BorderSizePixel = 0
progressBg.ZIndex = 112
progressBg.ClipsDescendants = true
progressBg.Parent = submitBtn
Instance.new("UICorner", progressBg).CornerRadius = UDim.new(0, 10)

local progressBar = Instance.new("Frame")
progressBar.Size = UDim2.new(0, 0, 1, 0)
progressBar.BackgroundColor3 = Colors.Glow1
progressBar.BackgroundTransparency = 0.5
progressBar.BorderSizePixel = 0
progressBar.ZIndex = 113
progressBar.Parent = progressBg
Instance.new("UICorner", progressBar).CornerRadius = UDim.new(0, 10)

local spinnerFrame = Instance.new("Frame")
spinnerFrame.Size = UDim2.new(0, 60, 0, 16)
spinnerFrame.Position = UDim2.new(0.5, -30, 0.5, -8)
spinnerFrame.BackgroundTransparency = 1
spinnerFrame.ZIndex = 115
spinnerFrame.Visible = false
spinnerFrame.Parent = submitBtn
UI.SpinnerFrame = spinnerFrame

for i = 1, 3 do
    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 10, 0, 10)
    dot.Position = UDim2.new(0, (i-1) * 22, 0.5, -5)
    dot.BackgroundColor3 = Colors.TextPrimary
    dot.BorderSizePixel = 0
    dot.ZIndex = 116
    dot.Name = "Dot" .. i
    dot.Parent = spinnerFrame
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
end

local getKeyBtn = Instance.new("TextButton")
getKeyBtn.Size = UDim2.new(0.5, -24, 0, 46)
getKeyBtn.Position = UDim2.new(0, 18, 0, 276)
getKeyBtn.BackgroundColor3 = Colors.GetKey
getKeyBtn.BorderSizePixel = 0
getKeyBtn.Text = "Get Key"
getKeyBtn.TextColor3 = Colors.TextPrimary
getKeyBtn.TextSize = 15
getKeyBtn.Font = Enum.Font.GothamBold
getKeyBtn.AutoButtonColor = false
getKeyBtn.ZIndex = 112
getKeyBtn.Parent = container
Instance.new("UICorner", getKeyBtn).CornerRadius = UDim.new(0, 10)

local discordBtn = Instance.new("TextButton")
discordBtn.Size = UDim2.new(0.5, -24, 0, 46)
discordBtn.Position = UDim2.new(0.5, 6, 0, 276)
discordBtn.BackgroundColor3 = Colors.Discord
discordBtn.BorderSizePixel = 0
discordBtn.Text = ""
discordBtn.AutoButtonColor = false
discordBtn.ZIndex = 112
discordBtn.Parent = container
Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 10)

local discordInner = Instance.new("Frame")
discordInner.Size = UDim2.new(0, 110, 0, 28)
discordInner.Position = UDim2.new(0.5, -55, 0.5, -14)
discordInner.BackgroundTransparency = 1
discordInner.ZIndex = 113
discordInner.Parent = discordBtn

local discordIcon = Instance.new("ImageLabel")
discordIcon.Size = UDim2.new(0, 24, 0, 24)
discordIcon.Position = UDim2.new(0, 0, 0.5, -12)
discordIcon.BackgroundTransparency = 1
discordIcon.Image = DISCORD_IMAGE
discordIcon.ScaleType = Enum.ScaleType.Fit
discordIcon.ZIndex = 114
discordIcon.Parent = discordInner

local discordText = Instance.new("TextLabel")
discordText.Size = UDim2.new(0, 80, 1, 0)
discordText.Position = UDim2.new(0, 30, 0, 0)
discordText.BackgroundTransparency = 1
discordText.Text = "Discord"
discordText.TextColor3 = Colors.TextPrimary
discordText.TextSize = 15
discordText.Font = Enum.Font.GothamBold
discordText.TextXAlignment = Enum.TextXAlignment.Left
discordText.ZIndex = 114
discordText.Parent = discordInner

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -36, 0, 50)
statusLabel.Position = UDim2.new(0, 18, 0, 336)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = ""
statusLabel.TextColor3 = Colors.Error
statusLabel.TextSize = 13
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextXAlignment = Enum.TextXAlignment.Center
statusLabel.TextWrapped = true
statusLabel.ZIndex = 112
statusLabel.Parent = container
UI.StatusLabel = statusLabel

-- ========================================
-- PARTICULAS
-- ========================================

local function CreateParticle()
    if State.IsDestroyed then return end
    local size = math.random(6, 18)
    local p = Instance.new("Frame")
    p.Size = UDim2.new(0, size, 0, size)
    p.Position = UDim2.new(math.random() * 1.4 - 0.2, 0, 1.1, 0)
    p.BackgroundColor3 = (math.random(2) == 1) and Colors.Glow1 or Colors.Glow2
    p.BackgroundTransparency = math.random(65, 88) / 100
    p.BorderSizePixel = 0
    p.ZIndex = 102
    p.Parent = particleContainer
    Instance.new("UICorner", p).CornerRadius = UDim.new(1, 0)
    local pd = {
        frame = p,
        vx = (math.random() - 0.5) * 0.003,
        vy = -math.random(15, 40) / 10000,
        created = tick(),
        pulsePhase = math.random() * math.pi * 2,
        wobblePhase = math.random() * math.pi * 2,
        originalTransparency = p.BackgroundTransparency,
        originalSize = size,
        lifetime = math.random(25, 55),
    }
    table.insert(State.Particles, pd)
end

local function UpdateParticles()
    for i = #State.Particles, 1, -1 do
        local pd = State.Particles[i]
        if not pd or not pd.frame or not pd.frame.Parent then
            table.remove(State.Particles, i)
        else
            local pos = pd.frame.Position
            if pos.Y.Scale < -0.2 or (tick() - pd.created) > pd.lifetime then
                pd.frame:Destroy()
                table.remove(State.Particles, i)
            else
                local nx = pos.X.Scale + pd.vx + math.sin(tick() * 1.5 + pd.wobblePhase) * 0.002
                local ny = pos.Y.Scale + pd.vy
                if nx < -0.2 then nx = 1.2 elseif nx > 1.2 then nx = -0.2 end
                pd.frame.Position = UDim2.new(nx, 0, ny, 0)
                local breathe = math.sin(tick() * 2.5 + pd.pulsePhase) * 0.08 + 1
                pd.frame.Size = UDim2.new(0, pd.originalSize * breathe, 0, pd.originalSize * breathe)
                pd.frame.BackgroundTransparency = math.max(0.5, math.min(0.92, pd.originalTransparency + math.sin(tick() * 3 + pd.pulsePhase) * 0.08))
            end
        end
    end
end

-- ========================================
-- HELPERS
-- ========================================

local function ShowStatus(msg, isError, isSuccess)
    UI.StatusLabel.Text = msg
    if isSuccess then
        UI.StatusLabel.TextColor3 = Colors.Success
    elseif isError then
        UI.StatusLabel.TextColor3 = Colors.Error
    else
        UI.StatusLabel.TextColor3 = Colors.Warning
    end
    UI.StatusLabel.TextTransparency = 1
    Services.TweenService:Create(UI.StatusLabel, TweenInfo.new(0.3), {TextTransparency = 0}):Play()
end

local function ClearStatus()
    Services.TweenService:Create(UI.StatusLabel, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
end

local function HoverEffect(btn, hoverColor, normalColor)
    btn.MouseEnter:Connect(function()
        if not State.IsLoading then
            Services.TweenService:Create(btn, TweenInfo.new(0.18), {BackgroundColor3 = hoverColor}):Play()
        end
    end)
    btn.MouseLeave:Connect(function()
        if not State.IsLoading then
            Services.TweenService:Create(btn, TweenInfo.new(0.18), {BackgroundColor3 = normalColor}):Play()
        end
    end)
end

HoverEffect(submitBtn, Colors.HoverPrimary, Colors.Primary)
HoverEffect(getKeyBtn, Colors.HoverGetKey, Colors.GetKey)
HoverEffect(discordBtn, Colors.HoverDiscord, Colors.Discord)

local function UpdateCharCounter()
    local len = #textInput.Text
    UI.CharCounter.Text = len .. "/" .. Config.MaxKeyLength
    if len >= Config.MaxKeyLength then
        UI.CharCounter.TextColor3 = Colors.Error
    elseif len >= Config.MaxKeyLength * 0.8 then
        UI.CharCounter.TextColor3 = Colors.Warning
    else
        UI.CharCounter.TextColor3 = Colors.TextSecondary
    end
end

-- ========================================
-- SPINNER
-- ========================================

local spinnerConn = nil

local function StartSpinner()
    UI.SpinnerFrame.Visible = true
    local dots = {
        UI.SpinnerFrame:FindFirstChild("Dot1"),
        UI.SpinnerFrame:FindFirstChild("Dot2"),
        UI.SpinnerFrame:FindFirstChild("Dot3"),
    }
    local phase = 0
    spinnerConn = Services.RunService.Heartbeat:Connect(function()
        phase = phase + 0.08
        for i, dot in ipairs(dots) do
            if dot then
                local scale = 0.7 + math.sin(phase + (i - 1) * 2) * 0.3
                local size = math.max(6, 10 * scale)
                dot.Size = UDim2.new(0, size, 0, size)
                dot.Position = UDim2.new(0, (i-1) * 22 + (10 - size) / 2, 0.5, -size / 2)
                dot.BackgroundTransparency = 0.3 - math.sin(phase + (i - 1) * 2) * 0.3
            end
        end
    end)
end

local function StopSpinner()
    if spinnerConn then
        spinnerConn:Disconnect()
        spinnerConn = nil
    end
    UI.SpinnerFrame.Visible = false
end

-- ========================================
-- VALIDATE KEY - API
-- ========================================

local function ValidateKey()
    if State.IsLoading then return end
    local key = textInput.Text
    if key == "" then
        ShowStatus("Please enter an access key.", true)
        textInput:CaptureFocus()
        return
    end

    State.IsLoading = true
    submitBtn.Text = ""
    progressBg.BackgroundTransparency = 0
    progressBar.Size = UDim2.new(0, 0, 1, 0)
    StartSpinner()
    ShowStatus("Validating key...", false, false)

    Services.TweenService:Create(progressBar, TweenInfo.new(Config.ValidateTime, Enum.EasingStyle.Linear), {
        Size = UDim2.new(1, 0, 1, 0)
    }):Play()

    task.spawn(function()
        task.wait(Config.ValidateTime)
        StopSpinner()
        progressBg.BackgroundTransparency = 1
        progressBar.Size = UDim2.new(0, 0, 1, 0)
        State.IsLoading = false

        local success, response = pcall(function()
            return game:HttpGet(API_URL .. key)
        end)

        if success and response == "valid" then
            ShowStatus("Access granted! Loading...", false, true)
            submitBtn.Text = "Access Granted!"
            submitBtn.BackgroundColor3 = Colors.Success
            task.wait(1.5)
            State.IsDestroyed = true
            Services.TweenService:Create(container, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)}):Play()
            Services.TweenService:Create(backdrop, TweenInfo.new(0.4), {BackgroundTransparency = 1}):Play()
            task.wait(0.45)
            screenGui:Destroy()
        elseif not success then
            ShowStatus("Server error. Try again later.", true)
            submitBtn.Text = "Verify Access Key"
            submitBtn.BackgroundColor3 = Colors.Primary
        else
            ShowStatus("Invalid key. Join Discord to get yours.", true)
            submitBtn.Text = "Verify Access Key"
            submitBtn.BackgroundColor3 = Colors.Primary
        end
    end)
end

-- ========================================
-- EVENTOS
-- ========================================

Services.UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        State.MousePosition.X = input.Position.X
        State.MousePosition.Y = input.Position.Y
    end
end)

textInput:GetPropertyChangedSignal("Text"):Connect(function()
    if #textInput.Text > Config.MaxKeyLength then
        textInput.Text = textInput.Text:sub(1, Config.MaxKeyLength)
    end
    UpdateCharCounter()
    ClearStatus()
end)

textInput.Focused:Connect(function()
    Services.TweenService:Create(inputStroke, TweenInfo.new(0.2), {Color = Colors.Glow1, Transparency = 0}):Play()
end)

textInput.FocusLost:Connect(function()
    Services.TweenService:Create(inputStroke, TweenInfo.new(0.2), {Color = Colors.Primary, Transparency = 0.3}):Play()
end)

Services.UserInputService.InputBegan:Connect(function(input, gp)
    if gp or State.IsDestroyed then return end
    if input.KeyCode == Enum.KeyCode.Return and textInput:IsFocused() then
        ValidateKey()
    end
end)

submitBtn.MouseButton1Click:Connect(ValidateKey)

getKeyBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if setclipboard then
            setclipboard(DISCORD_LINK)
            ShowStatus("Discord link copied! discord.gg/nnxRBba7d", false, true)
        else
            ShowStatus("discord.gg/nnxRBba7d", false, true)
        end
    end)
end)

discordBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if setclipboard then
            setclipboard(DISCORD_LINK)
            ShowStatus("Discord link copied! discord.gg/nnxRBba7d", false, true)
        else
            ShowStatus("discord.gg/nnxRBba7d", false, true)
        end
    end)
end)

-- ========================================
-- LOOPS
-- ========================================

task.spawn(function()
    for i = 1, 20 do
        if State.IsDestroyed then break end
        CreateParticle()
        task.wait(math.random(30, 120) / 1000)
    end
    while not State.IsDestroyed do
        if #State.Particles < Config.ParticleCount then CreateParticle() end
        task.wait(math.random(500, 1500) / 1000)
    end
end)

task.spawn(function()
    while not State.IsDestroyed do
        pcall(UpdateParticles)
        task.wait(1 / Config.ParticleSpeed)
    end
end)

task.spawn(function()
    while not State.IsDestroyed and borderFrame.Parent do
        Services.TweenService:Create(borderGradient, TweenInfo.new(4, Enum.EasingStyle.Linear), {Rotation = borderGradient.Rotation + 360}):Play()
        task.wait(4)
    end
end)

task.spawn(function()
    while not State.IsDestroyed and iconGlowFrame.Parent do
        Services.TweenService:Create(iconGlowGrad, TweenInfo.new(3, Enum.EasingStyle.Linear), {Rotation = iconGlowGrad.Rotation + 360}):Play()
        task.wait(3)
    end
end)

-- ========================================
-- ENTRADA
-- ========================================

container.Size = UDim2.new(0, 0, 0, 0)
container.BackgroundTransparency = 1
backdrop.BackgroundTransparency = 1

Services.TweenService:Create(backdrop, TweenInfo.new(0.3), {BackgroundTransparency = 0.15}):Play()
task.wait(0.1)
Services.TweenService:Create(container, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 340, 0, 460),
    BackgroundTransparency = 0
}):Play()
task.wait(0.55)
textInput:CaptureFocus()
UpdateCharCounter()

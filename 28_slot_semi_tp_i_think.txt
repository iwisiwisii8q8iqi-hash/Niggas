_G.iCollectPro_SemiTP_Executed = true

local iCollectPro_ENV = (type(getgenv) == "function" and getgenv()) or _G
iCollectPro_ENV.__semiTpGen = (tonumber(iCollectPro_ENV.__semiTpGen) or 0) + 1
iCollectPro_ENV.iCollectPro_SemiTP_Gen = (tonumber(iCollectPro_ENV.iCollectPro_SemiTP_Gen) or 0) + 1
local iCollectPro_GEN = iCollectPro_ENV.iCollectPro_SemiTP_Gen
local function iCollectPro_ALIVE()
    return iCollectPro_ENV.iCollectPro_SemiTP_Gen == iCollectPro_GEN
end

do
    local function neuter(t)
        if type(t) ~= "table" then return end
        t.debounce = true
        t.execute = function() end
        t.SSDoTeleport = function() end
    end
    pcall(neuter, rawget(_G, "iCollectPro"))
    pcall(neuter, rawget(_G, "iCollectPro_SemiTP"))
    pcall(neuter, iCollectPro_ENV.iCollectPro)
    pcall(neuter, iCollectPro_ENV.iCollectPro_SemiTP)
    for _, key in ipairs({ "__semiTpTables", "iCollectPro_SemiTP_Tables" }) do
        if type(iCollectPro_ENV[key]) == "table" then
            for _, t in ipairs(iCollectPro_ENV[key]) do pcall(neuter, t) end
        end
    end
    iCollectPro_ENV.__semiTpTables = nil
    iCollectPro_ENV.iCollectPro = nil
    iCollectPro_ENV.iCollectPro_SemiTP_Tables = {}
end

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
    repeat task.wait() until Players.LocalPlayer
    LocalPlayer = Players.LocalPlayer
end
local player = LocalPlayer

local IsMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

local safeGuiTarget = nil
local successCore, _ = pcall(function()
    local test = Instance.new("Folder")
    test.Parent = CoreGui
    test:Destroy()
end)
safeGuiTarget = successCore and CoreGui or (player:WaitForChild("PlayerGui", 5) or player.PlayerGui)

local function getGuiParent()
    if gethui then return gethui() end
    return safeGuiTarget
end

pcall(function()
    local roots = { getGuiParent(), safeGuiTarget }
    for _, name in ipairs({"iCollectPro_SemiTP", "iCollectPro_SemiTP_Progress", "iCollectPro_SemiTP_Speed", "iCollectPro_SemiTP_AP", "iCollectPro_SemiTP_Allow", "iCollectPro_SemiTP_ESP"}) do
        for _, root in ipairs(roots) do
            while root and root:FindFirstChild(name) do
                root[name]:Destroy()
            end
        end
    end
end)

local T = {
    BG = Color3.fromRGB(20, 12, 34),
    SURF = Color3.fromRGB(28, 16, 46),
    SURF2 = Color3.fromRGB(48, 26, 76),
    HOVER = Color3.fromRGB(62, 34, 96),
    TEXT = Color3.fromRGB(240, 232, 255),
    DIM = Color3.fromRGB(155, 120, 200),
    ACCENT = Color3.fromRGB(168, 85, 247),
    ACCENT2 = Color3.fromRGB(124, 45, 190),
    STROKE = Color3.fromRGB(150, 70, 230),
    GREEN1 = Color3.fromRGB(18, 88, 58),
    GREEN2 = Color3.fromRGB(21, 120, 76),
    GREEN_STROKE = Color3.fromRGB(60, 185, 120),
    ON_TEXT = Color3.fromRGB(232, 255, 240),
    OFF_BG = Color3.fromRGB(48, 26, 74),
    OFF_TEXT = Color3.fromRGB(140, 110, 180),
    TRACK = Color3.fromRGB(34, 20, 54),
    TRACK2 = Color3.fromRGB(46, 26, 72),
    FILL1 = Color3.fromRGB(150, 70, 235),
    FILL2 = Color3.fromRGB(200, 150, 255),
}

local function corner(obj, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r)
    c.Parent = obj
    return c
end

local function stroke(obj, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = obj
    return s
end

local function gradient(obj, c1, c2, rot)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, c1), ColorSequenceKeypoint.new(1, c2) })
    g.Rotation = rot or 0
    g.Parent = obj
    return g
end

local function addShadow(obj, pad, transparency)
    local s = Instance.new("ImageLabel")
    s.Name = "Shadow"
    s.AnchorPoint = Vector2.new(0.5, 0.5)
    s.Position = UDim2.new(0.5, 0, 0.5, 2)
    s.Size = UDim2.new(1, pad or 24, 1, pad or 24)
    s.BackgroundTransparency = 1
    s.Image = "rbxassetid://6014261993"
    s.ImageColor3 = Color3.new(0, 0, 0)
    s.ImageTransparency = transparency or 0.72
    s.ScaleType = Enum.ScaleType.Slice
    s.SliceCenter = Rect.new(49, 49, 450, 450)
    s.ZIndex = math.max(obj.ZIndex - 1, 0)
    s.Parent = obj
    return s
end

local function tween(obj, t, props)
    TweenService:Create(obj, TweenInfo.new(t or 0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), props):Play()
end

local function addHover(obj, normalColor, hoverColor, rowStroke)
    obj.MouseEnter:Connect(function()
        tween(obj, 0.14, { BackgroundColor3 = hoverColor })
        if rowStroke then tween(rowStroke, 0.14, { Transparency = 0.38 }) end
    end)
    obj.MouseLeave:Connect(function()
        tween(obj, 0.14, { BackgroundColor3 = normalColor })
        if rowStroke then tween(rowStroke, 0.14, { Transparency = 0.52 }) end
    end)
end

local function makeDraggable(handle, target, onEnd)
    local dragging, dragInput, startInputPos, startPos = false, nil, nil, nil
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragInput = input
            startInputPos = input.Position
            startPos = target.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End and dragging then
                    dragging = false
                    if onEnd then onEnd() end
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input == dragInput) then
            local d = input.Position - startInputPos
            target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

local configFile = "iCollectPro_SemiTP.json"
local HubConfig = {
    stealBoostAutoOnSteal = true,
    speedValue = 22,
    hexSpeedVisible = false,
    speedPos = {ScaleX = 0.98, OffsetX = 0, ScaleY = 0.02, OffsetY = 0},
    stealKeybind = "E",
    potionEnabled = true,
    autoTPOnAllowEnabled = false,
    kickAfterStealEnabled = false,
    antiRagdollEnabled = true,
    antiRocketEnabled = true,
    antiBeeEnabled = true,
    antiGummyEnabled = true,
    antiDiscoEnabled = true,
    selectedSlot = 1
}

if isfile and isfile(configFile) then
    local success, decoded = pcall(function() return HttpService:JSONDecode(readfile(configFile)) end)
    if success and decoded then
        for k, v in pairs(decoded) do
            if k ~= "selectedSlot" then HubConfig[k] = v end
        end
    end
end
HubConfig.selectedSlot = 1
HubConfig.apOnStealEnabled = nil
HubConfig.autoActivateEnabled = nil

local function saveHubConfig()
    if writefile then pcall(function() writefile(configFile, HttpService:JSONEncode(HubConfig)) end) end
end

if type(HubConfig.positions) ~= "table" then HubConfig.positions = {} end

local function restorePos(frame, key)
    local p = HubConfig.positions[key]
    if type(p) == "table" and #p == 4 then
        frame.Position = UDim2.new(tonumber(p[1]) or 0, tonumber(p[2]) or 0, tonumber(p[3]) or 0, tonumber(p[4]) or 0)
    end
end

local function rememberPos(frame, key)
    return function()
        local q = frame.Position
        HubConfig.positions[key] = { q.X.Scale, q.X.Offset, q.Y.Scale, q.Y.Offset }
        saveHubConfig()
    end
end

_G.iCollectPro_SemiTP_SpeedBoost = HubConfig.stealBoostAutoOnSteal ~= false
local currentSpeed = HubConfig.speedValue or 22
local currentStealKey = Enum.KeyCode[HubConfig.stealKeybind or "E"]
local PotionEnabled = HubConfig.potionEnabled
local AutoTPOnAllowEnabled = HubConfig.autoTPOnAllowEnabled or false
local KickAfterStealEnabled = HubConfig.kickAfterStealEnabled or false
-- Anti tab toggle (on by default); STEAL NOW / the steal key switch it back on
local AntiRagdollEnabled = HubConfig.antiRagdollEnabled ~= false
local AntiRocketEnabled = HubConfig.antiRocketEnabled ~= false
-- grouped in one table: the file sits near Luau's 200 top-level locals limit
local iCollectProFx = {
    bee = HubConfig.antiBeeEnabled ~= false,
    gummy = HubConfig.antiGummyEnabled ~= false,
    disco = HubConfig.antiDiscoEnabled ~= false,
    paint = HubConfig.antiPaintEnabled ~= false,
    web = HubConfig.antiWebEnabled ~= false,
    swap = HubConfig.antiSwapEnabled ~= false,
    infJump = HubConfig.infJumpEnabled == true,
    unlockTP = HubConfig.autoTPOnUnlockEnabled == true,
    sentry = HubConfig.destroySentryEnabled ~= false,
    doge = HubConfig.destroyDogeEnabled ~= false,
    ghost = HubConfig.serverGhostEnabled == true,
    esp = HubConfig.playerEspEnabled == true,
    tracers = HubConfig.tracersEnabled == true,
    aim = HubConfig.aimbotEnabled == true,
    spam = HubConfig.autoSpamEnabled == true,
    xray = HubConfig.baseXrayEnabled == true,
    -- from Aurora, batch 3 (all default off)
    timerEsp = HubConfig.baseTimerEspEnabled == true,
    timerFloors = HubConfig.timerEveryFloorEnabled == true,
    tags = HubConfig.stealTagsEnabled == true,
    baseEsp = HubConfig.baseEspEnabled == true,
    thiefBar = HubConfig.thiefBarEnabled == true,
    rejoin2 = HubConfig.autoRejoinEnabled == true,
    speedOn = HubConfig.moveSpeedEnabled == true,
    carpet = HubConfig.carpetSpeedEnabled == true,
    grav = HubConfig.gravityEnabled == true,
    fps = HubConfig.fpsOptimizerEnabled == true,
    pillBar = HubConfig.pillBarEnabled ~= false,
    -- from Aurora, batch 4 (all default off)
    defBypass = HubConfig.defenderBypassEnabled == true,
    pubGrab = HubConfig.publicGrabEnabled == true,
    quickGrab = HubConfig.quickGrabEnabled == true,
    fov = HubConfig.customFovEnabled == true,
    xfps = HubConfig.extremeFpsEnabled == true,
    hideAP = HubConfig.hideApIconEnabled == true,
}
local selectedSlot = 1
local targetPlot = nil

-- the server voids a carried brainrot ("Delivery Failed") when you move faster than ~23 while stealing
local STEAL_SPEED_MIN, STEAL_SPEED_MAX = 15, 23
local STEAL_CYCLE_EVERY, STEAL_CYCLE_GAP = 2, 0.15

local function stealSpeed()
    return math.clamp(tonumber(currentSpeed) or 22, STEAL_SPEED_MIN, STEAL_SPEED_MAX)
end

local stealBoostPaused = false
local stealCycleToken = 0

local function isStealBoostLive()
    return player:GetAttribute("Stealing") == true and not stealBoostPaused
end

local function onStealingChanged()
    stealCycleToken = stealCycleToken + 1
    stealBoostPaused = false
    if not player:GetAttribute("Stealing") then return end
    local mine = stealCycleToken
    task.spawn(function()
        while true do
            task.wait(STEAL_CYCLE_EVERY)
            if not iCollectPro_ALIVE() or mine ~= stealCycleToken or not player:GetAttribute("Stealing") then return end
            stealBoostPaused = true
            task.wait(STEAL_CYCLE_GAP)
            stealBoostPaused = false
        end
    end)
end
local stealAttrConn
stealAttrConn = player:GetAttributeChangedSignal("Stealing"):Connect(function()
    if not iCollectPro_ALIVE() then stealAttrConn:Disconnect() return end
    onStealingChanged()
end)
task.defer(onStealingChanged)

local activeSpeedConnections = {}
local function clearSpeedConnections()
    for _, conn in ipairs(activeSpeedConnections) do if conn then conn:Disconnect() end end
    activeSpeedConnections = {}
end

local function initSpeedFeatures(char)
    clearSpeedConnections()
    local hum = char:WaitForChild("Humanoid", 5)
    local hrp = char:WaitForChild("HumanoidRootPart", 5)
    if not (hum and hrp) then return end

    local speedConn
    speedConn = RunService.Heartbeat:Connect(function(dt)
        if not iCollectPro_ALIVE() then speedConn:Disconnect() return end
        if not char or not char.Parent or not hum or not hrp or hum.Health <= 0 then return end
        if not _G.iCollectPro_SemiTP_SpeedBoost or not isStealBoostLive() then return end
        if hum.MoveDirection.Magnitude <= 0 then return end
        if hum.FloorMaterial == Enum.Material.Air then return end
        local extra = math.max(stealSpeed() - hum.WalkSpeed, 0)
        if extra > 0 then
            hrp.CFrame = hrp.CFrame + (hum.MoveDirection * extra * dt)
        end
    end)
    table.insert(activeSpeedConnections, speedConn)
end

if player.Character then task.spawn(initSpeedFeatures, player.Character) end
local speedCharConn
speedCharConn = player.CharacterAdded:Connect(function(char)
    if not iCollectPro_ALIVE() then speedCharConn:Disconnect() return end
    task.wait(0.2)
    initSpeedFeatures(char)
end)

local STEAL_DURATION = 1.3
local progressFill, percentLabel = nil, nil

local function updateProgressBar(p)
    if progressFill then
        TweenService:Create(progressFill, TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Size = UDim2.new(math.clamp(p, 0, 1), 0, 1, 0)
        }):Play()
    end
    if percentLabel then percentLabel.Text = math.floor(math.clamp(p, 0, 1) * 100) .. "%" end
end

local findMyBase
do
    -- called from several loops a second; the base never moves, so re-check the cached one cheaply
    -- and only rescan the plots when it stops being ours (or every 2s as a safety net)
    local cached, checkedAt = nil, 0
    local function isMine(base)
        local sign = base and base.Parent and base:FindFirstChild("PlotSign")
        local yourBase = sign and sign:FindFirstChild("YourBase")
        return yourBase ~= nil and yourBase.Enabled
    end
    findMyBase = function()
        local now = os.clock()
        if cached and now - checkedAt < 2 and cached.Parent then return cached end
        if cached and isMine(cached) then checkedAt = now return cached end
        cached = nil
        local plots = Workspace:FindFirstChild("Plots")
        if not plots then return nil end
        for _, base in ipairs(plots:GetChildren()) do
            if isMine(base) then cached, checkedAt = base, now return base end
        end
        for _, base in ipairs(plots:GetChildren()) do
            if base:IsA("Model") then
                for _, d in ipairs(base:GetDescendants()) do
                    if d:IsA("TextLabel") and (string.find(d.Text, player.Name, 1, true) or string.find(d.Text, player.DisplayName, 1, true)) then
                        cached, checkedAt = base, now
                        return base
                    end
                end
            end
        end
        return nil
    end
end

local function isEnemyPlot(plot)
    if not plot or not plot:IsA("Model") then return false end
    local sign = plot:FindFirstChild("PlotSign")
    local yourBase = sign and sign:FindFirstChild("YourBase")
    if yourBase and yourBase.Enabled then return false end
    local sg = sign and sign:FindFirstChild("SurfaceGui")
    local frame = sg and sg:FindFirstChild("Frame")
    local label = frame and frame:FindFirstChild("TextLabel")
    if not label or label.Text == "Empty Base" then return false end
    local owner = label.Text:gsub("'s [Bb]ase$", ""):gsub("%s+$", "")
    return owner ~= player.Name and owner ~= player.DisplayName
end

pcall(function()
    for _, d in ipairs(Workspace:GetDescendants()) do
        if d:IsA("Highlight") and d.Name == "iCollectPro_SemiTP_Podium_Highlight" then
            d:Destroy()
        end
    end
end)

local slotHasTarget = true
local podiumInfoText = ""

local function readPodiumInfo(podium)
    local sp = podium and podium:FindFirstChild("Base") and podium.Base:FindFirstChild("Spawn")
    local debris = Workspace:FindFirstChild("Debris")
    if not sp or not debris then return "" end
    local base = sp.Position
    local best, bestD = nil, 7
    for _, o in ipairs(debris:GetChildren()) do
        if o.Name == "FastOverheadTemplate" and o:IsA("BasePart") then
            local p = o.Position
            local d = Vector3.new(p.X - base.X, 0, p.Z - base.Z).Magnitude
            if d < bestD and p.Y > base.Y - 2 and p.Y < base.Y + 25 then best, bestD = o, d end
        end
    end
    local bb = best and best:FindFirstChild("AnimalOverhead")
    if not bb then return "empty" end
    local function txt(name)
        local l = bb:FindFirstChild(name, true)
        return (l and l:IsA("TextLabel") and l.Visible and l.Text ~= "") and l.Text or nil
    end
    local name, gen, mut = txt("DisplayName"), txt("Generation"), txt("Mutation")
    if not name then return "empty" end
    return (mut and (mut .. " ") or "") .. name .. (gen and ("  " .. gen) or "")
end

local darkBlueHighlight = Instance.new("Highlight")
darkBlueHighlight.Name = "iCollectPro_SemiTP_Podium_Highlight"
darkBlueHighlight.FillColor = T.ACCENT2
darkBlueHighlight.OutlineColor = T.FILL2
darkBlueHighlight.FillTransparency = 0.35
darkBlueHighlight.OutlineTransparency = 0

local function getTargetPodiumForSlot(slot)
    local plots = Workspace:FindFirstChild("Plots")
    if not plots then return nil end
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end

    local bestPodium, bestScore = nil, math.huge
    for _, plot in ipairs(plots:GetChildren()) do
        local podiums = (not targetPlot or plot == targetPlot) and isEnemyPlot(plot) and plot:FindFirstChild("AnimalPodiums")
        local podium = podiums and podiums:FindFirstChild(tostring(slot))
        local spawn = podium and podium:FindFirstChild("Base") and podium.Base:FindFirstChild("Spawn")
        local pa = spawn and spawn:FindFirstChild("PromptAttachment")
        local prompt = pa and pa:FindFirstChildWhichIsA("ProximityPrompt")
        if spawn and prompt then
            local score = (hrp.Position - spawn.Position).Magnitude
            if not prompt.Enabled then score = score + 100000 end
            if score < bestScore then
                bestScore = score
                bestPodium = podium
            end
        end
    end
    return bestPodium
end

task.spawn(function()
    while task.wait(0.25) do
        if not iCollectPro_ALIVE() then darkBlueHighlight:Destroy() return end
        local targetPodium = getTargetPodiumForSlot(selectedSlot)
        slotHasTarget = targetPodium ~= nil
        podiumInfoText = targetPodium and readPodiumInfo(targetPodium) or "no enemy base has this podium"
        if targetPodium then
            if darkBlueHighlight.Adornee ~= targetPodium or darkBlueHighlight.Parent ~= targetPodium then
                darkBlueHighlight.Adornee = targetPodium
                darkBlueHighlight.Parent = targetPodium
            end
        else
            darkBlueHighlight.Adornee = nil
            darkBlueHighlight.Parent = nil
        end
    end
end)

local function getNearestDeliveryHitbox()
    local myBase = findMyBase()
    if not myBase then return nil end
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    local refPos = hrp and hrp.Position or myBase:GetPivot().Position

    local nearestPos = nil
    local minDist = math.huge
    for _, d in pairs(myBase:GetDescendants()) do
        if d.Name == "DeliveryHitbox" then
            local pos = d:IsA("BasePart") and d.Position or (d:IsA("Model") and d:GetPivot().Position)
            if pos then
                local dist = (pos - refPos).Magnitude
                if dist < minDist then
                    minDist = dist
                    nearestPos = pos
                end
            end
        end
    end

    if not nearestPos then
        for _, d in pairs(Workspace:GetDescendants()) do
            if d.Name == "DeliveryHitbox" then
                local pos = d:IsA("BasePart") and d.Position or (d:IsA("Model") and d:GetPivot().Position)
                if pos then
                    local dist = (pos - refPos).Magnitude
                    if dist < minDist then
                        minDist = dist
                        nearestPos = pos
                    end
                end
            end
        end
    end

    return nearestPos
end

local setStealStatus = function() end

local CLAIM_WATCH = 10
local KICK_INVITE = "\n\ndiscord.gg/Z7vPrxecnJ"

local function myFilledPodiums()
    local plot = findMyBase()
    local pods = plot and plot:FindFirstChild("AnimalPodiums")
    if not pods then return nil end
    local n = 0
    for _, p in ipairs(pods:GetChildren()) do
        local info = readPodiumInfo(p)
        if info ~= "" and info ~= "empty" then n = n + 1 end
    end
    return n
end

do
    local carrying, before, stolenName, stealChar = false, nil, nil, nil
    local claimToken, lastEnd, claimed = 0, 0, false

    local function onClaim(token)
        if token ~= claimToken or claimed then return end
        claimed = true
        claimToken = claimToken + 1
        setStealStatus("DELIVERED" .. (stolenName and (": " .. stolenName) or ""), true)
        if KickAfterStealEnabled then
            task.defer(function()
                pcall(function()
                    player:Kick("You stole " .. (stolenName or "a brainrot") .. KICK_INVITE)
                end)
            end)
        end
    end

    local net = ReplicatedStorage:FindFirstChild("Packages")
    net = net and net:FindFirstChild("Net")
    local success = net and net:FindFirstChild("RE/StealService/StealingSuccess")
    if success then
        local conn
        conn = success.OnClientEvent:Connect(function()
            if not iCollectPro_ALIVE() then conn:Disconnect() return end
            if carrying or os.clock() - lastEnd < CLAIM_WATCH then onClaim(claimToken) end
        end)
    end

    local attrConn
    attrConn = player:GetAttributeChangedSignal("Stealing"):Connect(function()
        if not iCollectPro_ALIVE() then attrConn:Disconnect() return end
        if player:GetAttribute("Stealing") then
            carrying = true
            claimed = false
            claimToken = claimToken + 1
            stealChar = player.Character
            before = myFilledPodiums()
            local idx = player:GetAttribute("StealingIndex")
            stolenName = (idx ~= nil and idx ~= "") and tostring(idx) or nil
            local hs = _G.iCollectPro_SemiTP
            local took = hs and hs.pressedAt and (tick() - hs.pressedAt)
            local tookText = (took and took < 15) and string.format("  %.1fs", took) or ""
            setStealStatus("CARRYING" .. (stolenName and (": " .. stolenName) or "") .. tookText, nil)
            return
        end
        if not carrying then return end
        carrying = false
        if claimed then return end
        -- a delivery only happens alive, on the same character, at your own delivery hitbox.
        -- Resetting while carrying used to count as a claim: your base's overheads stream in on
        -- respawn, the podium count "rises", and Kick After Delivery fired on a brainrot you lost.
        local ch = player.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        local hit = hrp and getNearestDeliveryHitbox()
        local atBase = hit and Vector3.new(hrp.Position.X - hit.X, 0, hrp.Position.Z - hit.Z).Magnitude <= 40
        if ch ~= stealChar or not hum or hum.Health <= 0 or _G.iCollectPro_SemiTP_ResetBusy or not atBase then
            claimToken = claimToken + 1
            setStealStatus("DROPPED" .. (stolenName and (": " .. stolenName) or ""), false)
            return
        end
        lastEnd = os.clock()
        local mine = claimToken
        task.spawn(function()
            local t0 = os.clock()
            while os.clock() - t0 < CLAIM_WATCH do
                if mine ~= claimToken or not iCollectPro_ALIVE() then return end
                local now = myFilledPodiums()
                if now and before and now > before then onClaim(mine) return end
                task.wait(0.1)
            end
            if mine == claimToken then
                claimToken = claimToken + 1
                setStealStatus("DROPPED" .. (stolenName and (": " .. stolenName) or ""), false)
            end
        end)
    end)

    local idxConn
    idxConn = player:GetAttributeChangedSignal("StealingIndex"):Connect(function()
        if not iCollectPro_ALIVE() then idxConn:Disconnect() return end
        local idx = player:GetAttribute("StealingIndex")
        if idx ~= nil and idx ~= "" then stolenName = tostring(idx) end
    end)
end

local FFlags = {
    GameNetPVHeaderRotationalVelocityZeroCutoffExponent = -5000, LargeReplicatorWrite5 = true,
    LargeReplicatorEnabled9 = true, AngularVelociryLimit = 360,
    TimestepArbiterVelocityCriteriaThresholdTwoDt = 2147483646, S2PhysicsSenderRate = 15000,
    DisableDPIScale = true, MaxDataPacketPerSend = 2147483647, PhysicsSenderMaxBandwidthBps = 20000,
    TimestepArbiterHumanoidLinearVelThreshold = 21, MaxMissedWorldStepsRemembered = -2147483648,
    PlayerHumanoidPropertyUpdateRestrict = true, SimDefaultHumanoidTimestepMultiplier = 0,
    StreamJobNOUVolumeLengthCap = 2147483647, DebugSendDistInSteps = -2147483648,
    GameNetDontSendRedundantNumTimes = 1, CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent = 1,
    CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth = 1,
    LargeReplicatorSerializeRead3 = true, ReplicationFocusNouExtentsSizeCutoffForPauseStuds = 2147483647,
    CheckPVCachedVelThresholdPercent = 10, CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth = 1,
    GameNetDontSendRedundantDeltaPositionMillionth = 1, InterpolationFrameVelocityThresholdMillionth = 5,
    StreamJobNOUVolumeCap = 2147483647, InterpolationFrameRotVelocityThresholdMillionth = 5,
    CheckPVCachedRotVelThresholdPercent = 10, WorldStepMax = 30,
    InterpolationFramePositionThresholdMillionth = 5, TimestepArbiterHumanoidTurningVelThreshold = 1,
    SimOwnedNOUCountThresholdMillionth = 2147483647, GameNetPVHeaderLinearVelocityZeroCutoffExponent = -5000,
    NextGenReplicatorEnabledWrite4 = true, TimestepArbiterOmegaThou = 1073741823, MaxAcceptableUpdateDelay = 1,
    LargeReplicatorSerializeWrite4 = true
}

local setFFlags = function()
    if type(setfflag) ~= "function" then return end
    for name, value in pairs(FFlags) do pcall(function() setfflag(tostring(name), tostring(value)) end) end
end

local FLY_GEAR_KEYS ={ "carpet", "broom", "wings", "sleigh", "waverider", "jetpack", "hoverboard", "glider" }

-- Fly Tool picker (Misc): a named tool wins when you own it, otherwise the first flying item found
local function findFlyGear()
    local want, fallback = HubConfig.flyTool, nil
    for _, cont in ipairs({ player.Character, player:FindFirstChild("Backpack") }) do
        if cont then
            for _, t in ipairs(cont:GetChildren()) do
                if t:IsA("Tool") then
                    if t.Name == want then return t end
                    local nm = t.Name:lower()
                    for _, k in ipairs(FLY_GEAR_KEYS) do
                        if not fallback and nm:find(k, 1, true) then fallback = t end
                    end
                end
            end
        end
    end
    return fallback
end

local flyGearToken = 0

local function holdFlyGearUntilCarrying()
    flyGearToken = flyGearToken + 1
    local mine = flyGearToken
    task.spawn(function()
        local tool = findFlyGear()
        local t0 = os.clock()
        while mine == flyGearToken and iCollectPro_ALIVE() and os.clock() - t0 < 12 do
            if player:GetAttribute("Stealing") then break end
            local char = player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0 then break end
            if not tool or not tool.Parent then tool = findFlyGear() end
            if tool and tool.Parent ~= char then
                pcall(function() hum:EquipTool(tool) end)
            end
            task.wait(0.3)
        end
    end)
end

local function stopFlyGear()
    flyGearToken = flyGearToken + 1
end

local walkTo = function(hrp, targetCords, desiredSpeed, precisionThreshold, stopWhen)
    if not hrp or not hrp.Parent or not targetCords then return end
    desiredSpeed = desiredSpeed or 180
    local running = true
    local connection
    local threshold = precisionThreshold or 3

    local _ctrls
    pcall(function() _ctrls = require(player.PlayerScripts:WaitForChild("PlayerModule", 2)):GetControls() end)
    if _ctrls then pcall(function() _ctrls:Disable() end) end

    connection = RunService.Heartbeat:Connect(function()
        if not hrp or not hrp.Parent or not running then
            if connection then connection:Disconnect() end
            return
        end
        local currentPos  = hrp.Position
        local flatCurrent = Vector3.new(currentPos.X, targetCords.Y, currentPos.Z)
        local direction   = targetCords - flatCurrent
        local distance    = direction.Magnitude
        if distance <= threshold or (stopWhen and stopWhen()) then
            running = false
            connection:Disconnect()
            hrp.Velocity = Vector3.zero
            return
        end
        local vel = direction.Unit * desiredSpeed
        hrp.Velocity = Vector3.new(vel.X, hrp.Velocity.Y, vel.Z)
    end)

    local startT = tick()
    while running do
        if tick() - startT > 6 then break end
        task.wait()
    end

    if _ctrls and not _G.iCollectPro_SemiTP_InputLock then pcall(function() _ctrls:Enable() end) end
end


local canDirectTp = function(HRP, targetPos)
    if not HRP or not targetPos then return false end
    local origin = HRP.Position
    local ignored = { player.Character }
    for _ = 1, 12 do
        local direction = targetPos - origin
        if direction.Magnitude <= 0.05 then return true end
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Blacklist
        params.FilterDescendantsInstances = ignored
        params.IgnoreWater = true
        local result = Workspace:Raycast(origin, direction, params)
        if not result then return true end
        local hit = result.Instance
        if not hit then return true end
        if hit:IsA("BasePart") and not hit.CanCollide then
            table.insert(ignored, hit)
            origin = result.Position + direction.Unit * 0.1
        else
            return (result.Position - targetPos).Magnitude <= 3
        end
    end
    return false
end

-- Auto Potion + Shield (Aurora's "Auto Giant Potion + Grief Shield"): pop the Grief Shield first so
-- nobody can knock the brainrot off you, then the Giant Potion
local function iCollectProAutoPotion()
    if not PotionEnabled then return end
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local bp = player:FindFirstChild("Backpack")
    if not hum then return end
    local shield = char:FindFirstChild("Grief Shield") or (bp and bp:FindFirstChild("Grief Shield"))
    if shield then
        pcall(function() hum:EquipTool(shield) end)
        local t0 = os.clock()
        while shield.Parent ~= char and os.clock() - t0 < 0.3 do task.wait() end
        if shield.Parent == char then pcall(function() shield:Activate() end) end
        task.wait(0.03)
    end
    local potion = char:FindFirstChild("Giant Potion") or (bp and bp:FindFirstChild("Giant Potion"))
    if potion then
        pcall(function() hum:EquipTool(potion) end)
        pcall(function() potion:Activate() end)
    end
end

local iCollectProStealCache = setmetatable({}, { __mode = "k" })
-- iCollectProHold.lastHold = when WE last began a hold. The game's hold-began handler
-- (PlotClient.AnimalPrompt) fires one StealService remote with a timestamp and no podium in it,
-- so the server keeps one hold per player - any enemy podium's hold counts for the next grab.
local iCollectProHold = { lastHold = 0 }
local iCollectProStealCallbacks = function(prompt)
    if type(getconnections) ~= "function" then return nil end
    if iCollectProStealCache[prompt] then return iCollectProStealCache[prompt] end
    local data = { hold = {}, trigger = {} }
    local ok1, c1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
    if ok1 then
        for _, c in ipairs(c1) do
            local f = c.Function
            local src = type(f) == "function" and (debug.info(f, "s") or "") or ""
            -- PlotClient's own hold-began handler only disables every OTHER podium's prompt on the plot
            -- (hold-ended turns them back on, and we never send that) - skip it, it greys out the
            -- podiums we grab next
            if type(f) == "function" and not (src:find("PlotClient", 1, true) and not src:find("AnimalPrompt", 1, true)) then
                table.insert(data.hold, function(...)
                    iCollectProHold.lastHold = tick()
                    return f(...)
                end)
            end
        end
    end
    local ok2, c2 = pcall(getconnections, prompt.Triggered)
    if ok2 then for _, c in ipairs(c2) do if type(c.Function) == "function" then table.insert(data.trigger, c.Function) end end end
    if #data.hold == 0 and #data.trigger == 0 then return nil end
    iCollectProStealCache[prompt] = data
    return data
end

function iCollectProHold.startStealHold(prompt)
    if not prompt or not prompt.Parent then return nil end
    local cb = iCollectProStealCallbacks(prompt)
    if not cb then return nil end
    for _, fn in ipairs(cb.hold) do task.spawn(fn) end
    local now = tick()
    return { prompt = prompt, cb = cb, ragdollFireTime = now, startedAt = now, holdBeganAt = now, holdDone = true }
end

function iCollectProHold.doHoldAndWait(ctx)
    if ctx.holdDone then return end
    for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
    ctx.holdBeganAt = tick()
    task.wait(1.3)
    ctx.holdDone = true
end

function iCollectProHold.waitForStealTime(ctx, sec)
    if not ctx or sec >= 1.0 then return end
    local elapsed = tick() - ctx.ragdollFireTime
    if elapsed < sec then task.wait(sec - elapsed) end
end

function iCollectProHold.finishStealHold(ctx)
    if not ctx then return false end
    if not ctx.holdBeganAt then iCollectProHold.doHoldAndWait(ctx) end
    local heldFor = tick() - (ctx.holdBeganAt or tick())
    if heldFor < 1.3 then task.wait(1.3 - heldFor) end
    task.wait(0.02)
    for _, fn in ipairs(ctx.cb.trigger) do task.spawn(fn) end
    return true
end

local ROW_ROUTES = {
    { z = -14.5, inside = Vector3.new(12.89, 4.88, -17.06), green = Vector3.new(30.63, 2.98, -32.97) },
    { z = -7.0, inside = Vector3.new(11.95, 5.13, -9.17), green = Vector3.new(22.58, 2.98, -35.87) },
    { z = 0.5, inside = Vector3.new(14.39, 4.88, -2.44), green = Vector3.new(21.60, 2.98, -28.10) },
    { z = 8.0, inside = Vector3.new(14.62, 4.93, 5.65), green = Vector3.new(21.71, 5.08, -20.68), mid = Vector3.new(19.54, 2.98, -29.61) },
    { z = 15.5, inside = Vector3.new(11.60, 4.72, 13.31), green = Vector3.new(20.40, 4.92, -11.40), mid = Vector3.new(19.54, 2.98, -29.61) },
}
local CORRIDOR_Y, CORRIDOR_Z, CORRIDOR_REACH = 3.1, -35.9, 107
local MAX_TRIGGER_DIST = 30

local function clampToPodium(from, to, podiumPos)
    local function flatDist(p)
        return Vector3.new(p.X - podiumPos.X, 0, p.Z - podiumPos.Z).Magnitude
    end
    if flatDist(to) <= MAX_TRIGGER_DIST then return to end
    local best = from
    for i = 1, 40 do
        local p = from:Lerp(to, i / 40)
        if flatDist(p) > MAX_TRIGGER_DIST then break end
        best = p
    end
    return best
end

local GROUND_MAX_Y = 10
-- every floor: 1-10 ground (walk-in steal), 11-28 2nd / 3rd floor (grapple + Quantum Cloner route)
local MAX_PODIUM = 28
local function rowFor(localZ)
    local best, bestD = ROW_ROUTES[1], math.huge
    for _, r in ipairs(ROW_ROUTES) do
        local d = math.abs(r.z - localZ)
        if d < bestD then best, bestD = r, d end
    end
    return best
end

local function podiumParts(podium)
    local spawn = podium and podium:FindFirstChild("Base") and podium.Base:FindFirstChild("Spawn")
    local pa = spawn and spawn:FindFirstChild("PromptAttachment")
    local prompt = pa and pa:FindFirstChildWhichIsA("ProximityPrompt")
    return spawn, prompt
end

local function findPodiumTarget(podiumNo, hrp)
    local plots = Workspace:FindFirstChild("Plots")
    if not plots then return nil end
    local best, bestScore = nil, math.huge
    for _, plot in ipairs(plots:GetChildren()) do
        local root = plot:FindFirstChild("MainRoot")
        local podiums = plot:FindFirstChild("AnimalPodiums")
        local podium = podiums and podiums:FindFirstChild(tostring(podiumNo))
        if root and podium and (not targetPlot or plot == targetPlot) and isEnemyPlot(plot) then
            local spawn, prompt = podiumParts(podium)
            if spawn and prompt then
                local score = (hrp.Position - spawn.Position).Magnitude
                if not prompt.Enabled then score = score + 100000 end
                if score < bestScore then
                    bestScore = score
                    best = { plot = plot, podium = podium, rootCF = root.CFrame, spawn = spawn, prompt = prompt }
                end
            end
        end
    end
    return best
end

local iCollectProUI = { maxPath = 90, state = { locked = false, lockText = "", inRange = true } }

-- one-line text that always fits its box: start at max size and shrink until the whole text shows
-- (TextScaled can't do this: in Roblox it forces wrapping onto two lines)
function iCollectProUI.fit(l, max, pad)
    l.TextScaled = false
    l.TextWrapped = false
    l.TextTruncate = Enum.TextTruncate.None
    local busy = false
    local function refit()
        if busy then return end
        busy = true
        l.TextSize = max
        local w, b = l.AbsoluteSize.X - (pad or 0), l.TextBounds.X
        if w > 0 and b > w then l.TextSize = math.max(6, math.floor(max * w / b)) end
        busy = false
    end
    refit()
    l:GetPropertyChangedSignal("Text"):Connect(refit)
    l:GetPropertyChangedSignal("AbsoluteSize"):Connect(refit)
end

function iCollectProUI.lockInfo(plot)
    local pur = plot and plot:FindFirstChild("Purchases")
    local pb = pur and pur:FindFirstChild("PlotBlock")
    local main = pb and pb:FindFirstChild("Main")
    local bb = main and main:FindFirstChild("BillboardGui")
    local lockedLbl = bb and bb:FindFirstChild("Locked")
    if lockedLbl and lockedLbl.Visible then
        local rt = bb:FindFirstChild("RemainingTime")
        return true, rt and rt.Text or ""
    end
    return false, ""
end

function iCollectProUI.flightPath(pod, hrp)
    local cf = pod.rootCF
    local podLocal = cf:PointToObjectSpace(pod.spawn.Position)
    local s = podLocal.X >= 0 and 1 or -1
    local row = rowFor(podLocal.Z)
    local here = cf:PointToObjectSpace(hrp.Position)
    local pts = {
        cf * Vector3.new(math.clamp(here.X, -CORRIDOR_REACH, CORRIDOR_REACH), CORRIDOR_Y, CORRIDOR_Z),
        cf * Vector3.new(0, CORRIDOR_Y, CORRIDOR_Z),
    }
    if podLocal.Y <= GROUND_MAX_Y then
        pts[#pts + 1] = cf * Vector3.new(row.inside.X * s, row.inside.Y, row.inside.Z)
    end
    local startIndex = 1
    for i = #pts, 1, -1 do
        if canDirectTp(hrp, pts[i]) then startIndex = i; break end
    end
    local len, from = 0, hrp.Position
    for i = startIndex, #pts do
        len = len + Vector3.new(pts[i].X - from.X, 0, pts[i].Z - from.Z).Magnitude
        from = pts[i]
    end
    return len
end

function iCollectProUI.check(ignoreLock)
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false, "NO CHARACTER" end
    local pod = findPodiumTarget(selectedSlot, hrp)
    if not pod then return false, "NO TARGET PODIUM" end
    local locked, t = iCollectProUI.lockInfo(pod.plot)
    -- the 2nd / 3rd floor route clones in through the wall, so the ground-floor lock doesn't stop it
    local upper = pod.rootCF:PointToObjectSpace(pod.spawn.Position).Y > GROUND_MAX_Y
    -- a locked base whose owner allows friends can still be walked into
    if locked and not upper and not ignoreLock and not (iCollectProUI.friendsAllowed and iCollectProUI.friendsAllowed(pod.plot)) then
        return false, "BASE LOCKED " .. t
    end
    -- 2nd / 3rd floor needs the Quantum Cloner ready, or the run ends outside the wall
    if upper and iCollectProUI.clonerState then
        local ready, why = iCollectProUI.clonerState()
        if not ready then return false, why end
    end
    return true
end

iCollectProUI.flash = function(msg) setStealStatus(msg, false) end

-- E / STEAL NOW: ignore every player input until the steal sequence is over, so a key press can't
-- walk, jump, swap tools or click in the middle of the grab. Movement goes through the game's
-- controls (off), the rest is swallowed by a top-priority action. GUI buttons still work.
function iCollectProUI.inputLock(on)
    local CAS = game:GetService("ContextActionService")
    local ACTION = "iCollectPro_SemiTP_InputLock"
    _G.iCollectPro_SemiTP_InputLock = on and true or nil
    local ctrls
    pcall(function() ctrls = require(player.PlayerScripts:WaitForChild("PlayerModule", 2)):GetControls() end)
    if on then
        if ctrls then pcall(function() ctrls:Disable() end) end
        local K = Enum.KeyCode
        pcall(function()
            CAS:BindActionAtPriority(ACTION, function(_, _, input)
                -- the steal key itself still goes through (pressing it again cancels Defender Bypass)
                if input.KeyCode == currentStealKey then return Enum.ContextActionResult.Pass end
                return Enum.ContextActionResult.Sink
            end, false, Enum.ContextActionPriority.High.Value + 2000,
                K.W, K.A, K.S, K.D, K.Up, K.Down, K.Left, K.Right, K.Space, K.LeftShift, K.Backspace, K.Q,
                K.One, K.Two, K.Three, K.Four, K.Five, K.Six, K.Seven, K.Eight, K.Nine, K.Zero,
                K.ButtonA, K.ButtonB, K.ButtonX, K.ButtonY, K.ButtonL1, K.ButtonR1, K.ButtonR2, K.Thumbstick1,
                Enum.UserInputType.MouseButton1)
        end)
    else
        pcall(function() CAS:UnbindAction(ACTION) end)
        if ctrls then pcall(function() ctrls:Enable() end) end
    end
end

-- keeps iCollectProUI.state fresh for the steal button ("LOCKED" text); the old "GO HERE" marker is gone
do
    pcall(function()
        for _, d in ipairs(Workspace:GetChildren()) do
            if d.Name == "iCollectPro_SemiTP_StandSpot" then d:Destroy() end
        end
    end)

    task.spawn(function()
        while task.wait(0.2) do
            if not iCollectPro_ALIVE() then return end
            local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            local pod = hrp and not _G.iCollectPro_SemiTP_Busy and not player:GetAttribute("Stealing") and findPodiumTarget(selectedSlot, hrp)
            if pod then
                local ok, len = pcall(iCollectProUI.flightPath, pod, hrp)
                local locked, lockText = iCollectProUI.lockInfo(pod.plot)
                -- upstairs targets clone in, the lock doesn't matter
                iCollectProUI.state.locked = locked and pod.rootCF:PointToObjectSpace(pod.spawn.Position).Y <= GROUND_MAX_Y
                iCollectProUI.state.lockText = lockText
                iCollectProUI.state.friends = iCollectProUI.friendsAllowed ~= nil and iCollectProUI.friendsAllowed(pod.plot)
                iCollectProUI.state.inRange = ok and len <= iCollectProUI.maxPath
            else
                iCollectProUI.state.friends = false
                iCollectProUI.state.locked = false
                iCollectProUI.state.inRange = true
            end
        end
    end)
end

local iCollectPro = { debounce = false }

-- how many podiums the target base has (it depends on the player's base upgrades); with Target on
-- Nearest, the biggest enemy base counts. MAX_PODIUM is only the fallback when nothing is loaded.
function iCollectPro.maxPodium()
    local plots = Workspace:FindFirstChild("Plots")
    local best = 0
    for _, plot in ipairs(plots and plots:GetChildren() or {}) do
        if (not targetPlot or plot == targetPlot) and isEnemyPlot(plot) then
            local pods = plot:FindFirstChild("AnimalPodiums")
            for _, p in ipairs(pods and pods:GetChildren() or {}) do
                local k = tonumber(p.Name)
                if k and k > best then best = k end
            end
        end
    end
    return best > 0 and best or MAX_PODIUM
end

function iCollectPro.setSlot(slot)
    slot = math.floor(tonumber(slot) or 1)
    if slot >= 1 and slot <= iCollectPro.maxPodium() then
        selectedSlot = slot
        HubConfig.selectedSlot = slot
        saveHubConfig()
    end
end

-- 2nd / 3rd floor steal (Meerko's route): grapple boost -> carpet -> fly to the clone spot just outside
-- the base's side wall -> Quantum Cloner swap lands us inside -> glide under the podium -> grab there.
-- Clone spots measured from Meerko's table, local to MainRoot: (+-24.6, 26.4, -14.5 / 0.7), facing in.
iCollectPro.CARPETS = { "Flying Carpet", "Cupid's Wings", "Santa's Sleigh", "Waverider", "Witch's Broom" }

function iCollectPro.findTool(name)
    local char, bp = player.Character, player:FindFirstChild("Backpack")
    return (char and char:FindFirstChild(name)) or (bp and bp:FindFirstChild(name))
end

-- steal log: every step of a STEAL NOW, timed from the key press, printed to the console as
-- "[SEMI TP] +1.23s ..." and kept in _G.iCollectPro_SemiTP_Log (last 80 lines)
iCollectPro.logT0 = os.clock()
function iCollectPro.log(msg)
    local line = string.format("+%.2fs %s", os.clock() - iCollectPro.logT0, msg)
    local buf = _G.iCollectPro_SemiTP_Log or {}
    _G.iCollectPro_SemiTP_Log = buf
    buf[#buf + 1] = line
    if #buf > 80 then table.remove(buf, 1) end
    print("[SEMI TP] " .. line)
end

-- the Quantum Cloner carries a CooldownTime attribute only while it's cooling down (measured live)
function iCollectProUI.clonerState()
    local cl = iCollectPro.findTool("Quantum Cloner")
    if not cl then return false, "NO QUANTUM CLONER" end
    local cd = cl:GetAttribute("CooldownTime")
    if cd ~= nil then
        local n = tonumber(cd)
        return false, "CLONER COOLDOWN" .. (n and string.format(" %.1fs", n) or "")
    end
    return true
end

function iCollectPro.carpetOn()
    local char = player.Character
    if not char then return false end
    for _, n in ipairs(iCollectPro.CARPETS) do
        if char:FindFirstChild(n) then return true end
    end
    return false
end

function iCollectPro.equipCarpet()
    if iCollectPro.carpetOn() then return true end
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    -- the Fly Tool picked in Misc first, then any carpet
    local pick = HubConfig.flyTool and iCollectPro.findTool(HubConfig.flyTool)
    if not (pick and table.find(iCollectPro.CARPETS, pick.Name)) then pick = nil end
    for _, n in ipairs(iCollectPro.CARPETS) do
        if pick then break end
        pick = iCollectPro.findTool(n)
    end
    if pick then pcall(function() hum:EquipTool(pick) end) end
    return false
end

-- Grapple FX off (user's call, "so it looks cool"): the tool's Fire / Hit sounds muted, its hook,
-- beam and rope hidden - and for a moment after firing, anything hook / rope shaped the grapple
-- spawns too. Render / volume only, on our screen; nothing is sent to the server.
function iCollectPro.hushGrapple(g, secs)
    local char = player.Character
    local function ours(d)
        return (char and d:IsDescendantOf(char)) or d:IsDescendantOf(g)
    end
    local function hush(d)
        if d:IsA("Sound") then
            d.Volume = 0
            pcall(d.Stop, d)
        elseif d:IsA("Beam") or d:IsA("Trail") or d:IsA("ParticleEmitter") then
            d.Enabled = false
        elseif d:IsA("RopeConstraint") or d:IsA("RodConstraint") then
            d.Visible = false
        elseif d:IsA("BasePart") then
            d.LocalTransparencyModifier = 1
        end
    end
    for _, d in ipairs(g:GetDescendants()) do pcall(hush, d) end
    if not secs then return end
    local function added(d)
        local n = d.Name:lower()
        local named = n:find("hook", 1, true) or n:find("rope", 1, true) or n:find("grapple", 1, true)
        local linked = false
        if d:IsA("Beam") or d:IsA("RopeConstraint") or d:IsA("RodConstraint") then
            local a0, a1 = d.Attachment0, d.Attachment1
            linked = (a0 and ours(a0)) or (a1 and ours(a1))
        end
        -- the grapple's two clips are 16211041 (fire) / 16211030 (hit); the carpet's own effects stay
        local sound = d:IsA("Sound") and (d:IsDescendantOf(g) or tostring(d.SoundId):find("1621103", 1, true)
            or tostring(d.SoundId):find("16211041", 1, true))
        if named or linked or sound or d:IsDescendantOf(g) then pcall(hush, d) end
        -- the grapple's pull: logged 2026-09-29, the first ~0.3s of every flight went BACKWARDS at 100
        -- (to end 151 -> 158 -> 167) until our 400 took over. A constraint tying us to something
        -- outside our body (the hook) is that pull - switch it off; SpeedAllowance was already handed
        -- out when the grapple fired. Carpet forces sit inside our own body and are left alone.
        if d:IsA("Constraint") then
            local a0, a1 = d.Attachment0, d.Attachment1
            local in0, in1 = a0 and char and a0:IsDescendantOf(char), a1 and char and a1:IsDescendantOf(char)
            if a0 and a1 and in0 ~= in1 then
                pcall(function() d.Enabled = false end)
                if iCollectPro.log then iCollectPro.log("  grapple pull cut: " .. d.ClassName .. " " .. d.Name) end
            elseif (in0 or in1) and iCollectPro.log then
                iCollectPro.log("  grapple window: " .. d.ClassName .. " " .. d.Name .. " (left on)")
            end
        elseif char and d:IsDescendantOf(char) and d:IsA("BodyMover") and d.Name == "FlightPower" then
            -- the grapple's own pull (measured 2026-09-30: BodyVelocity ~100 toward the hook, infinite
            -- force, gone after 0.58s) - it held the first 0.6s of every flight in place. Meerko
            -- destroys it right after firing; so do we
            task.defer(function() pcall(function() d:Destroy() end) end)
            if iCollectPro.log then iCollectPro.log("  grapple pull removed: FlightPower") end
        elseif char and d:IsDescendantOf(char) and d:IsA("BodyMover") and iCollectPro.log then
            iCollectPro.log("  grapple window: " .. d.ClassName .. " " .. d.Name .. " in " .. d.Parent.Name .. " (left on)")
        end
    end
    local conn = Workspace.DescendantAdded:Connect(function(d) task.defer(added, d) end)
    task.delay(secs, function() conn:Disconnect() end)
end

-- firing the grapple hands out the SpeedAllowance boost; unequip right away and hop on the carpet
function iCollectPro.grappleBoost()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local g = iCollectPro.findTool("Grapple Hook")
    if hum and g then
        pcall(iCollectPro.hushGrapple, g, 1.5)
        pcall(function() hum:EquipTool(g) end)
        local t0 = os.clock()
        while g.Parent ~= char and os.clock() - t0 < 0.35 do RunService.Heartbeat:Wait() end
        if g.Parent == char then task.spawn(pcall, g.Activate, g) end
        task.wait(0.06)
        pcall(function() hum:UnequipTools() end)
        RunService.Heartbeat:Wait()
    end
    local t0 = os.clock()
    while not iCollectPro.equipCarpet() and os.clock() - t0 < 2 do RunService.Heartbeat:Wait() end
end

-- carpet flight straight at a point (carpets carry IgnoreAntiCheat)
function iCollectPro.glide(hrp, dest, speed, cap)
    local t0 = os.clock()
    while hrp.Parent and os.clock() - t0 < (cap or 3) do
        local d = dest - hrp.Position
        if d.Magnitude <= 2 then break end
        iCollectPro.equipCarpet()
        hrp.AssemblyAngularVelocity = Vector3.zero
        hrp.AssemblyLinearVelocity = d.Unit * math.min(speed, math.max(d.Magnitude * 8, 40))
        RunService.Heartbeat:Wait()
    end
    if hrp.Parent then hrp.AssemblyLinearVelocity = Vector3.zero end
end

-- Meerko's flight (velMoveThrough): ONE heartbeat loop at a constant speed that flows through the
-- waypoints (switches to the next one ~8 studs early instead of stopping), slows to 240 only on a
-- sharp corner, caps the climb rate, and snaps onto the last point at the end or if it stalls.
-- Velocity goes on the torso (same assembly as the root). Meerko's forced jumps are left out:
-- a Jumping state in the air kills you in this game.
function iCollectPro.flyRoute(hrp, waypoints, speed)
    if not hrp or not hrp.Parent or #waypoints == 0 then return end
    speed = speed or 400
    local CLIMB = 120
    local idx, done, lastDist, stallSince = 1, false, math.huge, nil
    local total, prev = 0, hrp.Position
    for _, wp in ipairs(waypoints) do total = total + (wp - prev).Magnitude prev = wp end
    local function body()
        local ch = hrp.Parent
        return ch and (ch:FindFirstChild("UpperTorso") or ch:FindFirstChild("Torso")) or hrp
    end
    local conn
    local function finish()
        if done then return end
        done = true
        if conn then conn:Disconnect() end
        if hrp.Parent then
            hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
            -- only a small last-bit snap: logged 2026-09-29, a 138-stud snap here got reverted by the
            -- server (pulled back 155 studs) and the clone then spawned where the server had us
            local last = waypoints[#waypoints]
            if (hrp.Position - last).Magnitude <= 6 then
                local _, yaw = hrp.CFrame:ToEulerAnglesYXZ()
                hrp.CFrame = CFrame.new(last) * CFrame.Angles(0, yaw, 0)
            end
        end
    end
    conn = RunService.Heartbeat:Connect(function()
        if not hrp.Parent or not iCollectPro_ALIVE() then finish() return end
        iCollectPro.equipCarpet()
        local target = waypoints[idx]
        local diff = target - hrp.Position
        local mag = diff.Magnitude
        local spd = speed
        local lastWp = idx == #waypoints
        -- last waypoint: ease in and only finish within 1.5 studs. Logged 2026-09-29: the 8.3-stud
        -- switch radius (400/60*1.25) ended the flight 8.0 short of the clone spot -> FLIGHT FAILED
        if lastWp then spd = math.min(spd, math.max(40, mag * 12)) end
        if idx < #waypoints and mag < 26 then
            local turn = waypoints[idx + 1] - target
            if mag > 0.1 and turn.Magnitude > 0.1 and diff.Unit:Dot(turn.Unit) < 0.9 then spd = math.min(spd, 240) end
        end
        if mag < (lastWp and 1.5 or math.max(3, spd / 60 * 1.25)) then
            idx = idx + 1
            if idx > #waypoints then finish() return end
            lastDist, stallSince = math.huge, nil
            target = waypoints[idx]
            diff = target - hrp.Position
            mag = diff.Magnitude
        end
        -- stalled = no progress for 0.4s of real time (Meerko counts 18 frames, which is only 0.12s
        -- at a high frame rate and ended our flights right at takeoff)
        if mag > lastDist - 0.05 then
            stallSince = stallSince or os.clock()
            if os.clock() - stallSince >= 0.4 then finish() return end
        else
            stallSince = nil
        end
        lastDist = mag
        if mag >= 0.1 then
            local dir = diff.Unit
            local sp = spd
            if dir.Y > 0 and dir.Y * sp > CLIMB then sp = CLIMB / dir.Y end
            hrp.AssemblyAngularVelocity = Vector3.zero
            body().AssemblyLinearVelocity = dir * sp
        end
    end)
    local timeout = os.clock() + total / math.min(125, speed) + 2
    local nextTrace, t0 = 0, os.clock()
    while not done and os.clock() < timeout and hrp.Parent do
        if os.clock() >= nextTrace and iCollectPro.log then
            nextTrace = os.clock() + 0.1
            local ch = hrp.Parent
            local tool = ch and ch:FindFirstChildOfClass("Tool")
            local hum = ch and ch:FindFirstChildOfClass("Humanoid")
            iCollectPro.log(string.format("  fly %.2fs  wp %d/%d  to end %.0f  vel %.0f  tool=%s  state=%s  anchored=%s",
                os.clock() - t0, idx, #waypoints, (waypoints[#waypoints] - hrp.Position).Magnitude,
                hrp.AssemblyLinearVelocity.Magnitude, tool and tool.Name or "-",
                hum and hum:GetState().Name or "?", tostring(hrp.Anchored)))
        end
        task.wait(0.03)
    end
    finish()
end

-- Route planning, ported from Meerko's computeRoute: a segment only counts as clear if a 3-stud
-- sphere (our body) sweeps through it without touching anything solid - a thin ray slips past corners
-- the character can't. The last few studs at each end are skipped, so a spot hugging a wall still
-- counts. Players and our clone never block.
function iCollectPro.blocked(a, b, slackA, slackB)
    local d = b - a
    local len = d.Magnitude
    if len < 0.5 then return false end
    local u = d / len
    local a2 = a + u * math.min(slackA or 0, len * 0.4)
    local b2 = b - u * math.min(slackB or 0, len * 0.4)
    local rp = RaycastParams.new()
    rp.FilterType = Enum.RaycastFilterType.Exclude
    rp.RespectCanCollide = true
    local skip = {}
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Character then skip[#skip + 1] = pl.Character end
    end
    local clone = Workspace:FindFirstChild(tostring(player.UserId) .. "_Clone")
    if clone then skip[#skip + 1] = clone end
    rp.FilterDescendantsInstances = skip
    local ok, hit = pcall(Workspace.Spherecast, Workspace, a2, 3, b2 - a2, rp)
    if not ok then hit = Workspace:Raycast(a2, b2 - a2, rp) end
    return hit ~= nil
end

-- Pathing (user's call 2026-09-30: go AROUND things at the spot's height instead of climbing over
-- them): A* over a flat 6-stud grid at height y. A cell is open when nothing solid sits within 3
-- studs of it; the cells found are string-pulled with the body-sized sweep above, so the flight
-- only turns where it has to. Returns the points after `from` (last one = goal), or nil.
function iCollectPro.gridPath(from, goal, y)
    local CELL, R, MARGIN, MAXN, W = 6, 3, 90, 6000, 100000
    local op = OverlapParams.new()
    op.FilterType = Enum.RaycastFilterType.Exclude
    op.RespectCanCollide = true
    local skip = {}
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Character then skip[#skip + 1] = pl.Character end
    end
    local clone = Workspace:FindFirstChild(tostring(player.UserId) .. "_Clone")
    if clone then skip[#skip + 1] = clone end
    op.FilterDescendantsInstances = skip
    local x0 = math.min(from.X, goal.X) - MARGIN
    local z0 = math.min(from.Z, goal.Z) - MARGIN
    local nx = math.ceil((math.max(from.X, goal.X) + MARGIN - x0) / CELL)
    local nz = math.ceil((math.max(from.Z, goal.Z) + MARGIN - z0) / CELL)
    local function pos(k) return Vector3.new(x0 + (k // W) * CELL, y, z0 + (k % W) * CELL) end
    local function cellOf(p) return math.floor((p.X - x0) / CELL + 0.5), math.floor((p.Z - z0) / CELL + 0.5) end
    local open = {}
    local function isOpen(i, j)
        if i < 0 or j < 0 or i > nx or j > nz then return false end
        local k = i * W + j
        local v = open[k]
        if v == nil then
            v = #Workspace:GetPartBoundsInRadius(pos(k), R, op) == 0
            open[k] = v
        end
        return v
    end
    local si, sj = cellOf(from)
    local gi, gj = cellOf(goal)
    local sk, gk = si * W + sj, gi * W + gj
    open[sk], open[gk] = true, true
    local function h(i, j)
        local dx, dz = math.abs(i - gi), math.abs(j - gj)
        return math.max(dx, dz) + 0.414 * math.min(dx, dz)
    end
    -- binary heap of { f, key }
    local heap = {}
    local function push(f, k)
        heap[#heap + 1] = { f, k }
        local c = #heap
        while c > 1 do
            local p = c // 2
            if heap[p][1] <= heap[c][1] then break end
            heap[p], heap[c] = heap[c], heap[p]
            c = p
        end
    end
    local function pop()
        local top = heap[1]
        local last = table.remove(heap)
        if #heap > 0 then
            heap[1] = last
            local c = 1
            while true do
                local l, r, m = c * 2, c * 2 + 1, c
                if heap[l] and heap[l][1] < heap[m][1] then m = l end
                if heap[r] and heap[r][1] < heap[m][1] then m = r end
                if m == c then break end
                heap[m], heap[c] = heap[c], heap[m]
                c = m
            end
        end
        return top
    end
    local DIRS = { { 1, 0, 1 }, { -1, 0, 1 }, { 0, 1, 1 }, { 0, -1, 1 },
        { 1, 1, 1.414 }, { 1, -1, 1.414 }, { -1, 1, 1.414 }, { -1, -1, 1.414 } }
    -- Meerko's turn penalty (3 studs per change of direction on a 6-stud grid): fewer zig-zags,
    -- so the pulled path keeps long straight legs
    local TURN = 0.5
    local g, came, closed, pdir = { [sk] = 0 }, {}, {}, {}
    push(h(si, sj), sk)
    local n, found = 0, false
    while #heap > 0 and n < MAXN do
        local k = pop()[2]
        if not closed[k] then
            closed[k] = true
            n = n + 1
            if k == gk then found = true break end
            local i, j = k // W, k % W
            local pd = pdir[k]
            for _, d in ipairs(DIRS) do
                local a, b = i + d[1], j + d[2]
                -- diagonals only when both sides are open (no cutting a corner)
                if isOpen(a, b) and (d[3] == 1 or (isOpen(a, j) and isOpen(i, b))) then
                    local nk, ng = a * W + b, g[k] + d[3] + ((pd and pd ~= d) and TURN or 0)
                    if not closed[nk] and (g[nk] == nil or ng < g[nk]) then
                        g[nk], came[nk], pdir[nk] = ng, k, d
                        push(ng + h(a, b), nk)
                    end
                end
            end
        end
    end
    if not found then return nil, n end
    local cells, k = {}, gk
    while k do cells[#cells + 1] = k k = came[k] end
    local pts = {}
    for idx = #cells, 1, -1 do pts[#pts + 1] = pos(cells[idx]) end
    pts[1], pts[#pts] = from, goal
    -- string-pull: from each kept point, jump to the furthest point the body can sweep to
    local out, cur = {}, 1
    while cur < #pts do
        local j = #pts
        while j > cur + 1 and iCollectPro.blocked(pts[cur], pts[j]) do j = j - 1 end
        out[#out + 1] = pts[j]
        cur = j
    end
    return out, n
end

-- how long flyRoute takes on a route: each leg at `speed`, but a climb is capped at 120 up
function iCollectPro.routeTime(from, pts, speed)
    local t, prev = 0, from
    for _, p in ipairs(pts) do
        local d = p - prev
        t = t + math.max(d.Magnitude / speed, math.max(d.Y, 0) / 120)
        prev = p
    end
    return t
end

-- from -> approach (10 studs off the wall) -> spot. Meerko's way: a straight line if it's clear;
-- otherwise every candidate is built and the FASTEST one flies (Meerko also swaps a route that's
-- over 1.3x the geometric one). Candidates: the pathing above at the spot's height and at +12 / +25
-- / +45 (high enough to go over the low stuff instead of around it, like Meerko's lift-over), the
-- crest over the top, a sideways bend, and the old high cruise as the last resort.
function iCollectPro.planRoute(from, approach, spot, cruiseY, speed)
    local B = iCollectPro.blocked
    local SLACK = 6
    speed = speed or 360
    if not B(from, approach, SLACK, 2) then return { approach, spot }, "straight" end
    local best, bestT, bestKind = nil, math.huge, nil
    local function consider(route, kind)
        local t = iCollectPro.routeTime(from, route, speed)
        if t < bestT then best, bestT, bestKind = route, t, kind end
    end
    local t0 = os.clock()
    for _, lift in ipairs({ 0, 12, 25, 45 }) do
        local y = approach.Y + lift
        local rise = Vector3.new(from.X, y, from.Z)
        local goal = Vector3.new(approach.X, y, approach.Z)
        local up = math.abs(from.Y - y) > 2
        -- the drop from the high goal back down onto the approach point must be clear too
        if (not up or not B(from, rise, SLACK, 0)) and (lift == 0 or not B(goal, approach, 0, 2)) then
            local pts = iCollectPro.gridPath(rise, goal, y)
            if pts then
                local route = {}
                -- skip the straight rise when the first leg can climb on the diagonal
                if up and B(from, pts[1], SLACK, 0) then route[1] = rise end
                for _, p in ipairs(pts) do route[#route + 1] = p end
                if lift > 0 then route[#route + 1] = approach end
                route[#route + 1] = spot
                consider(route, "path" .. (lift > 0 and (" +" .. lift) or ""))
            end
        end
    end
    local function pull(pts)
        local out, i = {}, 0
        local cur = from
        while i < #pts do
            local j = #pts
            while j > i + 1 and B(cur, pts[j], i == 0 and SLACK or 0, 0) do j = j - 1 end
            out[#out + 1] = pts[j]
            cur, i = pts[j], j
        end
        return out
    end
    for _, lift in ipairs({ 12, 25, 45 }) do
        local cy = math.max(from.Y, approach.Y) + lift
        local up, over = Vector3.new(from.X, cy, from.Z), Vector3.new(approach.X, cy, approach.Z)
        if not B(from, up, SLACK, 0) and not B(up, over) and not B(over, approach, 0, 2) then
            local pts = pull({ up, over, approach })
            pts[#pts + 1] = spot
            consider(pts, "crest +" .. lift)
            break
        end
    end
    local flat = Vector3.new(approach.X - from.X, 0, approach.Z - from.Z)
    if flat.Magnitude > 1 then
        local perp = Vector3.new(-flat.Unit.Z, 0, flat.Unit.X)
        local mid0 = (from + approach) * 0.5
        for _, off in ipairs({ 14, -14, 24, -24, 38, -38, 56, -56, 76, -76 }) do
            local mid = mid0 + perp * off
            if not B(from, mid, SLACK, 0) and not B(mid, approach, 0, 2) then
                consider({ mid, approach, spot }, "bend " .. off)
                break
            end
        end
    end
    if best then
        return best, string.format("%s (%.2fs, planned in %.0fms)", bestKind, bestT, (os.clock() - t0) * 1000)
    end
    local high = Vector3.new(approach.X, cruiseY, approach.Z)
    local lift = from:Lerp(high, 0.25)
    return { Vector3.new(lift.X, cruiseY, lift.Z), high, approach, spot }, "high cruise"
end

-- Show Path (Steal tab): the planned route drawn as a glowing purple line with a dot on the
-- target spot, like Meerko's path view. Only you see it; it clears itself after the run.
function iCollectPro.drawPath(from, pts)
    local old = Workspace:FindFirstChild("iCollectPro_SemiTP_Path")
    if old then old:Destroy() end
    if HubConfig.showPathEnabled == false then return end
    local folder = Instance.new("Folder")
    folder.Name = "iCollectPro_SemiTP_Path"
    local function part(cf, size, ball)
        local p = Instance.new("Part")
        p.Anchored, p.CanCollide, p.CanQuery, p.CanTouch, p.CastShadow = true, false, false, false, false
        p.Material = Enum.Material.Neon
        p.Color = T.ACCENT
        if ball then p.Shape = Enum.PartType.Ball end
        p.Size, p.CFrame = size, cf
        p.Parent = folder
    end
    local prev = from
    for _, wp in ipairs(pts) do
        local d = wp - prev
        if d.Magnitude > 0.05 then
            part(CFrame.lookAt((prev + wp) * 0.5, wp), Vector3.new(0.3, 0.3, d.Magnitude), false)
        end
        prev = wp
    end
    part(CFrame.new(pts[#pts]), Vector3.new(1.6, 1.6, 1.6), true)
    folder.Parent = Workspace
    task.delay(6, function() if folder.Parent then folder:Destroy() end end)
end

-- Quantum Cloner: drop the clone 3 studs ahead (through the wall), then swap onto it
function iCollectPro.quantumClone()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local cloner = iCollectPro.findTool("Quantum Cloner")
    if not hum or not hrp or not cloner then return false end
    local function stealing()
        local st = player:GetAttribute("Stealing")
        return st ~= nil and st ~= false
    end
    -- Meerko: never fire the swap twice within 5s (a second press pulls us back to the old clone)
    if iCollectPro.lastSwapAt and os.clock() - iCollectPro.lastSwapAt < 5 then
        iCollectPro.log("clone skipped - swapped under 5s ago")
        return false
    end
    if cloner.Parent ~= char then pcall(function() hum:EquipTool(cloner) end) end
    local t0 = os.clock()
    while cloner.Parent ~= char and os.clock() - t0 < 0.5 do RunService.Heartbeat:Wait() end
    -- Meerko's clone hold: stop dead and anchor on the wall spot for 0.15s (was a plain 0.15s wait,
    -- user's call) so the server has us parked there when the clone spawns; released once it's out
    pcall(function() hum:Move(Vector3.zero, false) end)
    hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
    local lockCF = hrp.CFrame
    hrp.Anchored = true
    task.delay(2.5, function() if hrp.Parent then hrp.Anchored = false end end)
    t0 = os.clock()
    while os.clock() - t0 < 0.15 do
        RunService.Heartbeat:Wait()
        if hrp.Parent then hrp.CFrame = lockCF end
    end
    local cloneName = tostring(player.UserId) .. "_Clone"
    pcall(function() cloner:Activate() end)
    t0 = os.clock()
    local clone
    repeat
        RunService.Heartbeat:Wait()
        clone = Workspace:FindFirstChild(cloneName)
        -- only a clone right next to us counts: logged 2026-09-29, a leftover clone 166 studs away was
        -- taken and the swap pulled us across the map
        if clone and (clone:GetPivot().Position - hrp.Position).Magnitude > 12 then clone = nil end
    until clone or os.clock() - t0 > 1
    hrp.Anchored = false
    iCollectPro.log(string.format("clone %s after %.2fs  (%.1f studs away)  equipped=%s",
        clone and "spawned" or "NOT seen", os.clock() - t0,
        clone and (clone:GetPivot().Position - hrp.Position).Magnitude or -1, tostring(cloner.Parent == char)))
    local frames = player.PlayerGui:FindFirstChild("ToolsFrames")
    local qc = frames and frames:FindFirstChild("QuantumCloner")
    local btn = qc and qc:FindFirstChild("TeleportToClone")
    -- no clone next to us = no swap (a press would pull us to wherever an old clone stands)
    if not btn or not firesignal or not clone then pcall(function() hum:UnequipTools() end) return false end
    -- Meerko: a clone that landed further from the brainrot than we are (behind us, the wall bounced
    -- it back) would swap us the wrong way - don't swap
    local cloneAt = clone:GetPivot().Position
    local target = iCollectPro.cloneTarget
    if target then
        local function flat(v) return Vector2.new(v.X - target.X, v.Z - target.Z).Magnitude end
        if flat(cloneAt) > flat(hrp.Position) + 10 then
            iCollectPro.log(string.format("clone landed BEHIND us (%.1f vs %.1f from the brainrot) - no swap",
                flat(cloneAt), flat(hrp.Position)))
            pcall(function() hum:UnequipTools() end)
            return false
        end
    end
    -- the grab can land from the wall already; teleporting while carrying kills, so never swap then
    if stealing() then pcall(function() hum:UnequipTools() end) return "stole" end
    -- measured live: one press right as the clone spawns does nothing (the server isn't ready yet),
    -- so keep pressing every 0.1s until we actually land on the clone
    local from = hrp.Position
    local gapV = Vector3.new(cloneAt.X - from.X, 0, cloneAt.Z - from.Z)
    local gap = gapV.Magnitude
    local dir = gap > 0.01 and gapV / gap or Vector3.zero
    -- Meerko's swap check: we moved at least half the gap toward where the clone was, AND the clone
    -- moved at least half the gap away from its spot (it took our old place)
    local function swapDone()
        local me = (hrp.Position - from):Dot(dir)
        local cl = clone.Parent and Vector2.new(clone:GetPivot().Position.X - cloneAt.X,
            clone:GetPivot().Position.Z - cloneAt.Z).Magnitude or gap
        return me >= gap * 0.5 and cl >= gap * 0.5
    end
    local lastPress, presses = 0, 0
    t0 = os.clock()
    repeat
        if os.clock() - lastPress >= 0.1 then
            lastPress = os.clock()
            presses = presses + 1
            btn.Visible = true
            pcall(firesignal, btn.MouseButton1Up)
            iCollectPro.lastSwapAt = os.clock()
        end
        RunService.Heartbeat:Wait()
        if stealing() then break end
    until player.Character ~= char or not hrp.Parent or (gap >= 1.25 and swapDone()) or os.clock() - t0 > 1.5
    local ok = gap >= 1.25 and hrp.Parent and swapDone()
    iCollectPro.log(string.format("swap: %d presses in %.2fs  moved %.1f studs (gap %.1f)  confirmed=%s  newChar=%s",
        presses, os.clock() - t0, (hrp.Position - from).Magnitude, gap, tostring(ok), tostring(player.Character ~= char)))
    pcall(function() hum:UnequipTools() end)
    if stealing() and not ok then return "stole" end
    return ok and true or false
end

-- where the brainrot itself sits (its model's centre), like Meerko's getPetPosition
function iCollectPro.petPos(pod)
    for _, d in ipairs(pod.podium:GetDescendants()) do
        if d:IsA("Model") and d.Name ~= "Claim" and d.Name ~= "Base" and d.Name ~= "Decorations"
            and d:FindFirstChildWhichIsA("MeshPart", true) then
            local ok, cf = pcall(d.GetBoundingBox, d)
            if ok then return cf.Position end
        end
    end
    return pod.spawn.Position
end

-- invisible platform that lets us through going up and holds us going down (Meerko's one-way platform)
function iCollectPro.oneWayPlatform(pos)
    local old = Workspace:FindFirstChild("iCollectPro_SemiTP_Platform")
    if old then old:Destroy() end
    local plat = Instance.new("Part")
    plat.Name = "iCollectPro_SemiTP_Platform"
    plat.Size = Vector3.new(10, 1, 10)
    plat.Position = pos
    plat.Anchored, plat.CanCollide, plat.Transparency = true, false, 1
    plat.Parent = Workspace
    local conn, lastY
    conn = RunService.Stepped:Connect(function()
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if not plat.Parent or not hrp then conn:Disconnect() return end
        local y = hrp.Position.Y
        local up = hrp.AssemblyLinearVelocity.Y > 1 or (lastY and y - lastY > 0.01 and y - lastY < 5)
        plat.CanCollide = not up and y > plat.Position.Y + 0.1
        lastY = y
    end)
    task.delay(20, function() if plat.Parent then plat:Destroy() end end)
    return plat
end

function iCollectPro.upperSteal(pod, ctx)
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local cf = pod.rootCF
    local podLocal = cf:PointToObjectSpace(pod.spawn.Position)
    local s = podLocal.X >= 0 and 1 or -1
    local L = iCollectPro.log
    local function podDist()
        local h = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        return h and (h.Position - pod.spawn.Position).Magnitude or -1
    end
    L(string.format("UPPER podium %s  local(%.1f, %.1f, %.1f)  dist %.0f  hold=%s  prompt=%s",
        pod.podium.Name, podLocal.X, podLocal.Y, podLocal.Z, podDist(), tostring(ctx ~= nil), tostring(pod.prompt.Enabled)))
    -- a brainrot that just dropped back onto its podium can't be grabbed for a few seconds (prompt
    -- disabled): don't burn the Quantum Cloner on it
    if not pod.prompt.Enabled then
        L("podium prompt disabled - brainrot not grabbable yet, stopping")
        setStealStatus("BRAINROT NOT GRABBABLE YET - WAIT", false)
        return
    end
    stopFlyGear()

    -- 1) grapple boost, carpet on
    iCollectPro.grappleBoost()
    L(string.format("grapple done  SpeedAllowance=%s  carpet=%s",
        tostring(char:GetAttribute("SpeedAllowance")), tostring(iCollectPro.carpetOn())))

    -- the game drops a prompt hold after ~2.6s; keep the banked hold alive while we fly
    local flying = true
    if ctx then
        task.spawn(function()
            while flying do
                if tick() - (ctx.holdBeganAt or 0) >= 2.4 then
                    for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
                    ctx.holdBeganAt = tick()
                end
                task.wait(0.05)
            end
        end)
    end

    -- 2) fly (Meerko's way, one smooth run at 360): straight at the wall if nothing is in the way,
    -- otherwise up over the bases and down onto it. The clone spot is lined up with the target podium
    -- (same row, standing height of its floor) so the clone lands right beside it
    -- (Meerko's wall spots: local x 24.6, 5.4 above the 2nd-floor podiums)
    local spotY = podLocal.Y + 5.4
    local spotZ = math.clamp(podLocal.Z, -18, 18)
    local spot = cf * Vector3.new(24.6 * s, spotY, spotZ)
    -- 10 studs out from the wall, so the last leg comes in straight and facing the base
    local approach = cf * Vector3.new(34 * s, spotY, spotZ)
    -- last resort: cruise above every roof (measured: bases top out at local 45, decorated ones 79)
    local route, kind = iCollectPro.planRoute(hrp.Position, approach, spot, (cf * Vector3.new(0, 85, 0)).Y, 360)
    iCollectPro.drawPath(hrp.Position, route)
    L(string.format("route: %s, %d points", kind, #route))
    local flyT = os.clock()
    -- 360 with the grapple boost (user's call: 200 was too slow). Measured live 2026-09-30: 400 got
    -- pulled back by the server 4 times (up to 103 studs, the steal flight never arrived); 360 with the
    -- grapple and 320 without flew clean
    iCollectPro.flyRoute(hrp, route, 360)
    flying = false
    local offSpot = hrp.Parent and (hrp.Position - spot).Magnitude or math.huge
    L(string.format("flight done in %.2fs  off spot %.1f  pod dist %.1f", os.clock() - flyT, offSpot, podDist()))
    if offSpot > 6 then
        flying = false
        L("FLIGHT FAILED - never reached the clone spot, stopping")
        setStealStatus("FLIGHT BLOCKED - TRY AGAIN", false)
        return
    end
    -- restart the hold HERE, at the wall (~8.4 studs from the podium = inside its 10-stud range). The
    -- server only counts a hold begun in range: logged 2026-09-30 (podium 23), the hold begun at STEAL
    -- NOW 302 studs away was 2.29s old at the grab and still didn't count - Stealing came 0.84s later,
    -- ~1.5s after we got close. Begun here, it runs out while we clone, swap, snap and settle (~1.0s),
    -- so the grab lands ~1.55s after the wall
    if ctx then
        for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
        ctx.holdBeganAt = tick()
        L(string.format("hold restarted at the wall (pod dist %.1f)", podDist()))
    end

    local function stealing()
        local st = player:GetAttribute("Stealing")
        return st ~= nil and st ~= false
    end
    -- fire the banked hold's grab once it has been held the prompt's full HoldDuration (1.5s on
    -- podiums, measured 2026-09-29) - finishStealHold only waits 1.3s. 2nd / 3rd floor only.
    -- Meerko also never fires on a hold older than 2.5s (the server has dropped it): restart it then.
    local function upperTrigger()
        local need = (tonumber(pod.prompt.HoldDuration) or 1.3) + 0.05
        while not stealing() do
            local age = tick() - (ctx.holdBeganAt or 0)
            if age > 2.5 then
                for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
                ctx.holdBeganAt = tick()
                L("hold older than 2.5s - restarted")
            elseif age >= need then
                break
            end
            RunService.Heartbeat:Wait()
        end
        if stealing() then return end
        -- Meerko only fires with the brainrot within 10 studs
        if podDist() > 10 then L(string.format("grab held back - pod dist %.1f > 10", podDist())) return end
        L(string.format("grab fired  (hold age %.2fs, pod dist %.1f)", tick() - (ctx.holdBeganAt or 0), podDist()))
        for _, fn in ipairs(ctx.cb.trigger) do task.spawn(fn) end
    end
    -- the grab (INSTA STEAL / Quick Grab style): press the target's prompt every frame once the gate
    -- opens. Meerko's order: NO grab until the move is fully done (swapped in, snapped under the
    -- podium, settled), so the gate starts shut
    iCollectPro.cloneTarget = pod.spawn.Position
    local grabbing = true
    -- no presses from the swap until 0.5s after it: logged 2026-09-29 (3rd floor, podium 20) - grabs
    -- 0.12s / 0.21s after the swap were taken back 0.2-0.3s later standing still (Stealing -> nil, no
    -- damage); grabs 0.76s / 0.85s after it stuck
    local grabGate = math.huge
    task.spawn(function()
        while grabbing and not stealing() and iCollectPro_ALIVE() do
            if os.clock() >= grabGate and fireproximityprompt and pod.prompt.Parent then pcall(fireproximityprompt, pod.prompt, 0) end
            RunService.Heartbeat:Wait()
        end
    end)

    -- 3) settle on the spot facing into the base, a hidden floor under us, then clone in
    local face = cf:VectorToWorldSpace(Vector3.new(-s, 0, 0))
    local hold = Instance.new("Part")
    hold.Name = "iCollectPro_SemiTP_ClonePad"
    hold.Size = Vector3.new(12, 1, 12)
    hold.Position = spot - Vector3.new(0, 3.5, 0)
    hold.Anchored, hold.CanCollide, hold.Transparency = true, true, 1
    hold.Parent = Workspace
    for _ = 1, 2 do
        hrp.CFrame = CFrame.lookAt(spot, spot + face)
        hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
        RunService.Heartbeat:Wait()
    end
    L(string.format("at the wall  pod dist %.1f  stealing=%s  cloner=%s", podDist(), tostring(stealing()),
        select(2, iCollectProUI.clonerState()) or "ready"))
    local swapped = not stealing() and iCollectPro.quantumClone()
    local swapT = os.clock()
    hold:Destroy()
    L(string.format("clone result=%s  pod dist %.1f  stealing=%s", tostring(swapped), podDist(), tostring(stealing())))
    if stealing() or swapped == "stole" then
        grabbing = false
        L("GRABBED from the wall (no swap)")
        task.spawn(iCollectProAutoPotion)
        return
    end
    if not swapped then
        grabbing = false
        L("CLONE FAILED - swap never moved us")
        setStealStatus("CLONE FAILED (Quantum Cloner on cooldown?)", false)
        return
    end

    -- 4) cloned in: the prompt spam is waiting on grabGate (shut until we're under the podium);
    -- stand still (moving while carrying = death / Delivery Failed)
    char = player.Character
    hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then grabbing = false return end
    hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
    -- the swap leaves our Quantum Clone standing 3-4 studs away. Logged 2026-09-29 (3rd floor): walked
    -- into it after the grab (touched _Clone.__HITBOX, Head, UpperTorso) and HP 100 -> 0 0.14s later -
    -- the same "first step after the grab" death as every upper-floor run. Our client reports our
    -- body's touches, so switch the clone's touch + collision off on our side for the rest of the run.
    task.spawn(function()
        local cl = Workspace:FindFirstChild(tostring(player.UserId) .. "_Clone")
        if not cl then return end
        local function off(p)
            if p:IsA("BasePart") then p.CanTouch, p.CanCollide, p.CanQuery = false, false, false end
        end
        for _, p in ipairs(cl:GetDescendants()) do pcall(off, p) end
        local c = cl.DescendantAdded:Connect(function(p) pcall(off, p) end)
        L("clone neutralized (no touch / collide on our side)")
        task.delay(30, function() c:Disconnect() end)
    end)
    -- 2nd / 3rd floor only: switch Quick Grab on the moment we're in (the Steal tab chip turns green),
    -- and let it run inside this sequence (it normally sits out while STEAL NOW is busy)
    local qgChip = iCollectProUI.cfgChips and iCollectProUI.cfgChips.quickGrabEnabled
    local function qg(on)
        if qgChip then pcall(qgChip, on) else iCollectProFx.quickGrab = on end
    end
    -- 5) Meerko's way in: carpet back on, then snap straight to 8 studs under the podium's spawn
    -- (2nd AND 3rd floor, his F2StealBelow default) with a small solid platform 5 under that.
    -- Nothing is grabbed yet - the snap comes BEFORE the grab, never after it.
    local t0 = os.clock()
    while not iCollectPro.carpetOn() and os.clock() - t0 < 0.5 do iCollectPro.equipCarpet(); RunService.Heartbeat:Wait() end
    t0 = os.clock()
    while os.clock() - t0 < 0.08 do RunService.Heartbeat:Wait() end
    local to = pod.spawn.Position - Vector3.new(0, 8, 0)
    local old = Workspace:FindFirstChild("iCollectPro_SemiTP_Platform")
    if old then old:Destroy() end
    local plat = Instance.new("Part")
    plat.Name = "iCollectPro_SemiTP_Platform"
    plat.Size = Vector3.new(3, 1, 3)
    plat.Position = to - Vector3.new(0, 5, 0)
    plat.Anchored, plat.CanCollide, plat.Transparency = true, true, 1
    plat.Parent = Workspace
    -- kept while we carry (Meerko: pulling it out from under a carrier drops him), gone after 30s
    task.spawn(function()
        task.wait(30)
        while plat.Parent and stealing() do task.wait(0.5) end
        if plat.Parent then plat:Destroy() end
    end)
    RunService.Heartbeat:Wait()
    -- Meerko's snap: write the spot, check 0.05s later, once more if we're over 3 studs off
    local snapped = false
    for _ = 1, 2 do
        if not hrp.Parent then grabbing = false return end
        hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
        hrp.CFrame = CFrame.new(to) * hrp.CFrame.Rotation
        hrp.AssemblyLinearVelocity, hrp.AssemblyAngularVelocity = Vector3.zero, Vector3.zero
        t0 = os.clock()
        while os.clock() - t0 < 0.05 do RunService.Heartbeat:Wait() end
        if hrp.Parent and (hrp.Position - to).Magnitude <= 3 then snapped = true break end
    end
    local snapT = os.clock()
    L(string.format("snapped under the podium %.2fs after the swap  ok=%s  pod dist %.1f",
        snapT - swapT, tostring(snapped), podDist()))

    -- 6) settle, then grab: 0.35s after the snap (Meerko's settle) and never sooner than 0.5s after
    -- the swap (see grabGate). Stand still from here - no more moving until the steal is ours.
    local settleUntil = math.max(snapT + 0.35, swapT + 0.5)
    while os.clock() < settleUntil and not stealing() do RunService.Heartbeat:Wait() end
    grabGate = os.clock()
    L(string.format("settled %.2fs after the swap - grabbing  pod dist %.1f  hold age %.2fs",
        os.clock() - swapT, podDist(), ctx and tick() - (ctx.holdBeganAt or 0) or -1))
    qg(true)
    iCollectPro.upperGrab = true
    if ctx then task.spawn(upperTrigger) end
    -- Quick Grab pulse (user's call): flick the chip on / off until the grab lands. Every flick back on
    -- makes Quick Grab pick the target fresh and press it that same frame, then every frame while on.
    -- 0.1s on, 0.03s off = a fresh press ~7-8 times a second; it's left ON once we're carrying.
    local pulses = 0
    task.spawn(function()
        local t = os.clock()
        while not stealing() and iCollectPro.upperGrab and iCollectPro_ALIVE() and os.clock() - t < 1.5 do
            task.wait(0.1)
            if stealing() or not iCollectPro.upperGrab then break end
            qg(false)
            task.wait(0.03)
            qg(true)
            pulses = pulses + 1
        end
        -- grab landed = Quick Grab goes OFF (user's call); no grab yet = leave it on
        qg(not stealing())
    end)
    local function grabFor(sec)
        local t = os.clock()
        while not stealing() and os.clock() - t < sec do RunService.Heartbeat:Wait() end
        return stealing()
    end
    -- 2.5s: covers a hold restart (upperTrigger restarts a >2.5s hold, grabbable 1.55s later)
    local got = grabFor(2.5)
    grabbing = false
    iCollectPro.upperGrab = false
    if got then
        qg(false)
        L(string.format("GRABBED under the podium %.2fs after the swap  (quick grab pulses %d, now OFF)",
            os.clock() - swapT, pulses))
        task.spawn(iCollectProAutoPotion)
        return
    end
    L(string.format("NO GRAB under the podium (pod dist %.1f, prompt=%s, hold age %.2fs)",
        podDist(), tostring(pod.prompt.Enabled), ctx and tick() - (ctx.holdBeganAt or 0) or -1))
    setStealStatus("NO GRAB - TRY AGAIN", false)
end

function iCollectPro.SSDoTeleport()
    local char = player.Character
    local hum = char and char:FindFirstChild("Humanoid")
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hum or not hrp then return end

    setFFlags()

    local pod = findPodiumTarget(selectedSlot, hrp)
    if not pod then return end

    local function doTpSequence(HRP)
        local cf = pod.rootCF
        local podLocal = cf:PointToObjectSpace(pod.spawn.Position)
        local s = podLocal.X >= 0 and 1 or -1
        local row = rowFor(podLocal.Z)
        local upper = podLocal.Y > GROUND_MAX_Y
        local function at(v) return cf * Vector3.new(v.X * s, v.Y, v.Z) end

        local inside = at(row.inside)

        local here = cf:PointToObjectSpace(HRP.Position)
        local waypoints = {
            cf * Vector3.new(math.clamp(here.X, -CORRIDOR_REACH, CORRIDOR_REACH), CORRIDOR_Y, CORRIDOR_Z),
            cf * Vector3.new(0, CORRIDOR_Y, CORRIDOR_Z),
            inside,
        }

        local function doApproachPath(HRP_, speed)
            local startIndex = 1
            for i = #waypoints, 1, -1 do
                if canDirectTp(HRP_, waypoints[i]) then startIndex = i; break end
            end
            for i = startIndex, #waypoints do
                walkTo(HRP_, waypoints[i], speed or 180)
            end
        end

        local ctx = nil
        if pod.prompt and pod.prompt.Parent then
            pod.prompt.RequiresLineOfSight = false
            pod.prompt.MaxActivationDistance = math.huge
            if type(getconnections) == "function" then
                ctx = iCollectProHold.startStealHold(pod.prompt)
            else
                task.spawn(function()
                    if fireproximityprompt then fireproximityprompt(pod.prompt) end
                end)
            end
        end

        -- the 0.8s settle is the 1st floor's; upstairs the grapple goes off at once (the banked hold
        -- is kept alive in flight and restarted at the wall anyway)
        if ctx and not upper then iCollectProHold.waitForStealTime(ctx, 0.8) end
        if upper then
            -- 2nd / 3rd floor: grapple -> carpet -> clone in -> under the podium -> grab
            iCollectPro.upperSteal(pod, ctx)
        else
            doApproachPath(HRP, 180)
            task.wait(0.25)

            iCollectProAutoPotion()

            if pod.prompt and pod.prompt.Parent then
                if ctx then iCollectProHold.waitForStealTime(ctx, 1.3) end
                -- Defender Bypass: keep holding the prompt and grab the moment the base owner leaves
                if ctx and iCollectProFx.defBypass and iCollectProUI.waitOwnerLeave and not iCollectProUI.waitOwnerLeave(pod, ctx) then return end
                -- 1st floor: original stealing, user's call - keep as is
                local green = clampToPodium(inside, at(row.green), pod.spawn.Position)
                HRP.CFrame = CFrame.new(green)
                HRP.AssemblyLinearVelocity = Vector3.zero
                iCollectPro.log("ground grab (original green snap)")
                if ctx then iCollectProHold.finishStealHold(ctx) end
            end
        end
    end

    task.spawn(function()
        _G.iCollectPro_SemiTP_Busy = true
        pcall(doTpSequence, hrp)
        iCollectPro.upperGrab = false
        _G.iCollectPro_SemiTP_Busy = false
        stopFlyGear()
    end)
end

local TeleportBtn = nil

-- ignoreLock: TP on Allow steals from a locked base whose owner let friends in
function iCollectPro.execute(ignoreLock)
    if not iCollectPro_ALIVE() then return end
    -- pressing again while Defender Bypass waits cancels the wait
    if iCollectProUI.bypassWaiting then iCollectProUI.bypassWaiting = false return end
    if player:GetAttribute("Stealing") or iCollectPro.debounce then return end
    iCollectPro.logT0 = os.clock()
    iCollectPro.log("---- STEAL NOW  podium " .. tostring(selectedSlot))
    local ok, why = iCollectProUI.check(ignoreLock == true)
    if not ok then
        iCollectPro.log("refused: " .. tostring(why))
        iCollectProUI.flash(why)
        return
    end
    -- for 10s after the press, log what the game does to us: steal start/end, damage, death, respawn
    task.spawn(function()
        local ch = player.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local conns = {}
        -- after the grab: where we are, how fast we move and what we hold, every 0.1s for 1.5s,
        -- so a death right after a grab shows what moved us
        local grabPos
        local function state()
            local h = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
            if not h then return "no root" end
            return string.format("moved %.1f  vel %.0f  tool=%s  busy=%s", grabPos and (h.Position - grabPos).Magnitude or 0,
                h.AssemblyLinearVelocity.Magnitude, tool and tool.Name or "-", tostring(_G.iCollectPro_SemiTP_Busy))
        end
        conns[1] = player:GetAttributeChangedSignal("Stealing"):Connect(function()
            iCollectPro.log("Stealing -> " .. tostring(player:GetAttribute("Stealing")) .. "  " .. tostring(player:GetAttribute("StealingIndex") or ""))
            local h = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if player:GetAttribute("Stealing") and h and not grabPos then
                grabPos = h.Position
                -- what the body touches for 2s after the grab: the death on the first step may be a
                -- kill part inside the base (each part name logged once)
                local seen, tconns = {}, {}
                for _, p in ipairs(player.Character:GetChildren()) do
                    if p:IsA("BasePart") then
                        tconns[#tconns + 1] = p.Touched:Connect(function(o)
                            if o:IsDescendantOf(player.Character) or seen[o] then return end
                            seen[o] = true
                            iCollectPro.log(string.format("  touched %s  (%s, collide=%s)", o:GetFullName():sub(-70),
                                o.ClassName, tostring(o.CanCollide)))
                        end)
                    end
                end
                task.delay(2, function() for _, c in ipairs(tconns) do c:Disconnect() end end)
                task.spawn(function()
                    for _ = 1, 15 do
                        task.wait(0.1)
                        iCollectPro.log("  after grab: " .. state())
                    end
                end)
            end
        end)
        if hum then
            local lastHp = hum.Health
            conns[2] = hum:GetPropertyChangedSignal("Health"):Connect(function()
                if hum.Health < lastHp - 1 then iCollectPro.log(string.format("HP %.0f -> %.0f  %s", lastHp, hum.Health, state())) end
                lastHp = hum.Health
            end)
        end
        conns[3] = player.CharacterAdded:Connect(function() iCollectPro.log("RESPAWNED (died / reset)") end)
        -- what the server tells us: a brainrot dropped with no damage (Stealing -> nil standing still,
        -- 0.2s after a clone-in grab, 2026-09-29) should show up here with the game's reason
        local net = game:GetService("ReplicatedStorage"):FindFirstChild("Packages")
        net = net and net:FindFirstChild("Net")
        for _, n in ipairs({ "RE/NotificationService/Notify", "RE/StealService/StealingFailure",
            "RE/StealService/StealingSuccess", "RE/StealService/Grab", "RE/QuantumCloner/OnTeleport" }) do
            local re = net and net:FindFirstChild(n)
            if re then
                conns[#conns + 1] = re.OnClientEvent:Connect(function(...)
                    local a = {}
                    for i, v in ipairs({ ... }) do
                        a[i] = typeof(v) == "Instance" and v:GetFullName():sub(-40) or tostring(v):sub(1, 60)
                    end
                    iCollectPro.log("  server " .. n:match("[^/]+/[^/]+$") .. ": " .. table.concat(a, " | "))
                end)
            end
        end
        task.wait(10)
        for _, c in ipairs(conns) do c:Disconnect() end
        iCollectPro.log("---- log window closed  busy=" .. tostring(_G.iCollectPro_SemiTP_Busy))
    end)
    -- STEAL NOW / the steal key switches Quick Grab + Public Grab off (toggles too) so they can't
    -- fire their own grab in the middle of this one
    for _, key in ipairs({ "quickGrabEnabled", "publicGrabEnabled" }) do
        local set = iCollectProUI.cfgChips and iCollectProUI.cfgChips[key]
        if set then pcall(set, false) end
    end
    iCollectProFx.quickGrab, iCollectProFx.pubGrab = false, false
    -- ...and switches Anti Ragdoll on, so a hit on the way in can't knock the steal over
    local ragdollChip = iCollectProUI.cfgChips and iCollectProUI.cfgChips.antiRagdollEnabled
    if ragdollChip then pcall(ragdollChip, true) end
    iCollectPro.debounce = true
    iCollectPro.pressedAt = tick()
    STEAL_DURATION = 1.3

    task.spawn(function()
        local startT = tick()
        while tick() - startT < STEAL_DURATION do
            local p = math.clamp((tick() - startT) / STEAL_DURATION, 0, 1)
            updateProgressBar(p); task.wait()
        end
        updateProgressBar(1); task.wait(0.3); updateProgressBar(0)
    end)

    task.spawn(function()
        setFFlags()
        iCollectProUI.inputLock(true)
        holdFlyGearUntilCarrying()
        pcall(iCollectPro.SSDoTeleport)
        if not _G.iCollectPro_SemiTP_Busy then stopFlyGear() end
        -- inputs come back once the whole sequence (fly-in, hold, grab) has finished
        local t0 = os.clock()
        while _G.iCollectPro_SemiTP_Busy and os.clock() - t0 < 20 do task.wait() end
        iCollectProUI.inputLock(false)
        task.wait(0.1)
        iCollectPro.debounce = false
    end)
end

-- Allow/Disallow Friends prompts, collected once and kept up to date. The loops below used to walk
-- every object in the whole map (Workspace:GetDescendants) twice a second each; that was the
-- script's biggest cost. A prompt's text flips between Allow/Disallow, so keep every prompt that
-- ever says either and filter by the current text.
iCollectProUI.friendPrompts = {}
do
    local set = iCollectProUI.friendPrompts
    local function consider(d)
        if d:IsA("ProximityPrompt") then
            local function check()
                if string.find(d.ObjectText, "llow Friends", 1, true) then set[d] = true end
            end
            check()
            d:GetPropertyChangedSignal("ObjectText"):Connect(check)
            d.AncestryChanged:Connect(function(_, parent) if not parent then set[d] = nil end end)
        end
    end
    for _, d in ipairs(Workspace:GetDescendants()) do consider(d) end
    local addConn
    addConn = Workspace.DescendantAdded:Connect(function(d)
        if not iCollectPro_ALIVE() then addConn:Disconnect() return end
        consider(d)
    end)
end

function iCollectProUI.promptPart(desc)
    local p = desc.Parent
    if not p then return nil end
    return p:IsA("BasePart") and p or p:FindFirstChildWhichIsA("BasePart", true)
end

-- friends allowed on this plot? (its friend prompt then reads "Disallow Friends")
local function friendsAllowed(plot)
    for desc in pairs(iCollectProUI.friendPrompts) do
        if desc.Parent and string.find(desc.ObjectText, "Disallow", 1, true) then
            local part = iCollectProUI.promptPart(desc)
            if part and part:IsDescendantOf(plot) then return true end
        end
    end
    return false
end
iCollectProUI.friendsAllowed = friendsAllowed

-- TP on Allow: watch the base you're targeting (Target + Podium) and fire STEAL NOW the moment its
-- owner allows friends - even while it's locked, since allowed friends walk through the lock
local hasAutoTPTriggered = false
task.spawn(function()
    while task.wait(0.1) do
        if not iCollectPro_ALIVE() then return end
        local hrp = AutoTPOnAllowEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        local pod = hrp and findPodiumTarget(selectedSlot, hrp)
        if pod and friendsAllowed(pod.plot) then
            if not hasAutoTPTriggered and not iCollectPro.debounce and not player:GetAttribute("Stealing") then
                hasAutoTPTriggered = true
                iCollectPro.execute(true)
            end
        else
            hasAutoTPTriggered = false
        end
    end
end)

-- Auto TP On Unlock: fire STEAL NOW the moment the target base's lock timer runs out
task.spawn(function()
    local wasLocked = setmetatable({}, { __mode = "k" })
    while iCollectPro_ALIVE() do
        task.wait(0.05)
        local hrp = iCollectProFx.unlockTP and player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        local pod = hrp and findPodiumTarget(selectedSlot, hrp)
        if pod then
            local locked = iCollectProUI.lockInfo(pod.plot)
            if wasLocked[pod.plot] and not locked and iCollectProFx.unlockTP
                and not iCollectPro.debounce and not player:GetAttribute("Stealing") then
                iCollectPro.execute()
            end
            wasLocked[pod.plot] = locked
        end
    end
end)

do
    local _RunService = RunService
    local LP          = player

    _G.iCollectPro_SemiTP_ResetBusy = false
    -- logged 2026-09-29: this flag was stuck on true (a reset that overlapped another restored the
    -- other one's "true"), so Anti Ragdoll's revive was silently off. Start clean on every load.
    _G.iCollectPro_SemiTP_AntiDieDisabled = nil

    _G.iCollectPro_SemiTP_InstaReset = function()
        local _now = os.clock()
        if _G.iCollectPro_SemiTP_ResetBusy
            and (_now - (tonumber(_G.iCollectPro_SemiTP_ResetAt) or 0))
                < (tonumber(_G.iCollectPro_SemiTP_ResetCooldown) or 2.5) then
            return
        end
        _G.iCollectPro_SemiTP_ResetBusy = true
        _G.iCollectPro_SemiTP_ResetAt   = _now

        task.spawn(function()
            local _prevAntiDie = _G.iCollectPro_SemiTP_AntiDieDisabled
            _G.iCollectPro_SemiTP_AntiDieDisabled = true
            _G.iCollectPro_SemiTP_StealHold  = false
            if _G.iCollectPro_SemiTP_SoftenAntiDie then pcall(_G.iCollectPro_SemiTP_SoftenAntiDie) end

            local _restored, _holding = false, true
            local function _restore()
                if _restored then return end
                _restored = true
                _holding  = false
                -- always back to on: restoring the previous value could carry an overlapping reset's "true"
                _G.iCollectPro_SemiTP_AntiDieDisabled  = nil
                _G.iCollectPro_SemiTP_ResetBusy = false
            end

            local _conn
            _conn = LP.CharacterAdded:Connect(function(newChar)
                if _conn then _conn:Disconnect(); _conn = nil end
                task.defer(function()
                    pcall(function() newChar:WaitForChild("Humanoid", 12) end)
                    _RunService.Heartbeat:Wait()
                    _restore()
                end)
            end)
            task.delay(8, function()
                if _conn then _conn:Disconnect(); _conn = nil end
                _restore()
            end)

            pcall(function()
                local char = LP.Character
                if not char then return end

                local _origChar = char

                local bp = LP:FindFirstChild("Backpack")
                if bp then
                    local hum = char:FindFirstChildOfClass("Humanoid")
                    if hum then pcall(function() hum:UnequipTools() end) end
                    for _, ch in ipairs(char:GetChildren()) do
                        if ch:IsA("Tool") then
                            pcall(function() ch.Parent = bp end)
                        end
                    end
                end

                local function _flingPart()
                    local c = LP.Character
                    if not c then return nil end
                    return c:FindFirstChild("UpperTorso")
                        or c:FindFirstChild("Torso")
                        or c:FindFirstChild("HumanoidRootPart")
                end

                local _t0 = os.clock()
                while os.clock() - _t0
                    < (tonumber(_G.iCollectPro_SemiTP_ResetFlingTime) or 5) do
                    if LP.Character ~= _origChar then break end
                    local _h = _origChar:FindFirstChildOfClass("Humanoid")
                    if not _h or _h.Health <= 0
                        or _h:GetState() == Enum.HumanoidStateType.Dead then
                        break
                    end
                    local part = _flingPart()
                    if not part then break end

                    pcall(function()
                        for _, o in ipairs(part:GetChildren()) do
                            if o:IsA("BodyPosition") or o:IsA("BodyVelocity")
                                or o:IsA("BodyGyro") or o:IsA("AlignPosition")
                                or o:IsA("LinearVelocity") then
                                o:Destroy()
                            end
                        end
                    end)

                    pcall(function()
                        part.Velocity = Vector3.new(0, 9999999, 0)
                    end)
                    _RunService.Heartbeat:Wait()
                end
            end)
        end)
    end
end

local iCollectProAntiRagdoll = {}
do
    local RS = ReplicatedStorage
    local LP = player

    local connections = {}
    local character, humanoid, rootPart, animator

    local function antiDieOff()
        return not AntiRagdollEnabled or _G.iCollectPro_SemiTP_AntiDieDisabled == true or _G.iCollectPro_SemiTP_ResetBusy == true
    end

    local function isFlyingCarpetActive()
        if not character then return false end
        if not character:FindFirstChildWhichIsA("Tool") then return false end
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if hrp then
            for _, obj in ipairs(hrp:GetChildren()) do
                if obj:IsA("BodyVelocity") or obj:IsA("BodyPosition") or obj:IsA("BodyGyro") then
                    return true
                end
            end
        end
        return false
    end

    local function isRagdolled()
        if not humanoid then return false end
        local state = humanoid:GetState()
        return state == Enum.HumanoidStateType.Physics
            or state == Enum.HumanoidStateType.Ragdoll
            or state == Enum.HumanoidStateType.FallingDown
            or state == Enum.HumanoidStateType.GettingUp
    end

    local controls
    local function enableControls()
        if not controls then
            local ps = LP:FindFirstChild("PlayerScripts")
            local PlayerModule = ps and ps:FindFirstChild("PlayerModule")
            if not PlayerModule then return end
            pcall(function() controls = require(PlayerModule):GetControls() end)
        end
        if controls then pcall(function() controls:Enable() end) end
    end

    local function cleanupRagdoll()
        if not character then return end
        local carpetEquipped = isFlyingCarpetActive()
        pcall(function()
            for _, obj in ipairs(character:GetChildren()) do
                if obj:IsA("BallSocketConstraint") or obj:IsA("NoCollisionConstraint") or obj:IsA("HingeConstraint")
                    or (obj:IsA("Attachment") and (obj.Name == "A" or obj.Name == "B")) then
                    obj:Destroy()
                elseif obj:IsA("BodyVelocity") or obj:IsA("BodyPosition") or obj:IsA("BodyGyro") then
                    if not carpetEquipped then obj:Destroy() end
                elseif obj:IsA("Motor6D") then
                    obj.Enabled = true
                elseif obj:IsA("BasePart") then
                    for _, child in ipairs(obj:GetChildren()) do
                        if child:IsA("Motor6D") then
                            child.Enabled = true
                        elseif child:IsA("BallSocketConstraint") or child:IsA("NoCollisionConstraint") or child:IsA("HingeConstraint") then
                            child:Destroy()
                        elseif child:IsA("Attachment") and (child.Name == "A" or child.Name == "B") then
                            child:Destroy()
                        end
                    end
                end
            end
        end)
        if animator then
            for _, track in pairs(animator:GetPlayingAnimationTracks()) do
                local animName = track.Animation and track.Animation.Name:lower() or ""
                if animName:find("rag") or animName:find("fall") or animName:find("hurt") or animName:find("down") then
                    track:Stop(0)
                end
            end
        end
    end

    local function harden(hum)
        pcall(function() hum.BreakJointsOnDeath = false end)
        pcall(function() hum.RequiresNeck = false end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false) end)
    end

    local function soften(hum)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true) end)
        pcall(function() hum.BreakJointsOnDeath = true end)
        pcall(function() hum.RequiresNeck = true end)
    end

    local function revive(hum)
        pcall(function() hum.Health = hum.MaxHealth end)
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    end

    local function clear()
        for _, c in pairs(connections) do pcall(function() c:Disconnect() end) end
        connections = {}
    end

    local function bind(signal, fn)
        local c
        c = signal:Connect(function(...)
            if not iCollectPro_ALIVE() then c:Disconnect() return end
            return fn(...)
        end)
        table.insert(connections, c)
    end

    local function setup()
        clear()
        if not AntiRagdollEnabled or not humanoid or not rootPart then return end
        local hum = humanoid

        if not antiDieOff() then harden(hum) end

        bind(hum:GetPropertyChangedSignal("Health"), function()
            if antiDieOff() then return end
            if hum.Parent and hum.Health <= 0 then revive(hum) end
        end)

        bind(hum.Died, function()
            if antiDieOff() then return end
            if hum.Parent then revive(hum) end
        end)

        local lastHarden = 0
        bind(RunService.Heartbeat, function()
            if not hum.Parent or antiDieOff() then return end
            local now = os.clock()
            if now - lastHarden >= 1.0 then lastHarden = now; harden(hum) end
            if hum.Health <= 0 then revive(hum) end
        end)

        local HOLD_GRACE = 0.35
        local holdPos, holdUntil, lastSafe = nil, 0, nil

        local function myRoot()
            local ch = LP.Character
            return ch and ch:FindFirstChild("HumanoidRootPart")
        end

        local function ragLeft()
            local t = LP:GetAttribute("RagdollEndTime")
            if type(t) ~= "number" then return 0 end
            return t - workspace:GetServerTimeNow()
        end

        local function lockOff()
            return _G.iCollectPro_SemiTP_ResetBusy == true
                or _G.iCollectPro_SemiTP_Busy == true
                or isFlyingCarpetActive()
        end

        local function hitNow()
            if lockOff() then return end
            local rp = myRoot()
            if not rp then return end
            holdUntil = math.max(holdUntil, os.clock() + HOLD_GRACE)
            if not holdPos then holdPos = lastSafe or rp.Position end
            pcall(function()
                rp.AssemblyLinearVelocity = Vector3.zero
                rp.AssemblyAngularVelocity = Vector3.zero
            end)
        end

        local function holding()
            return holdPos ~= nil and (os.clock() < holdUntil or ragLeft() > 0)
        end

        local function pin(rp, dt)
            if dt then
                local md = hum.MoveDirection
                if md.Magnitude > 0 then
                    holdPos = holdPos + Vector3.new(md.X, 0, md.Z) * hum.WalkSpeed * dt
                end
            end
            local _, yaw = rp.CFrame:ToEulerAnglesYXZ()
            rp.CFrame = CFrame.new(holdPos) * CFrame.Angles(0, yaw, 0)
            rp.AssemblyLinearVelocity = Vector3.zero
            rp.AssemblyAngularVelocity = Vector3.zero
        end

        bind(hum.StateChanged, function()
            if isRagdolled() then
                hitNow()
                if not isFlyingCarpetActive() then hum:ChangeState(Enum.HumanoidStateType.Running) end
                cleanupRagdoll()
                workspace.CurrentCamera.CameraSubject = hum
                enableControls()
            end
        end)

        bind(LP:GetAttributeChangedSignal("RagdollEndTime"), function()
            if ragLeft() > 0 then
                hitNow()
                enableControls()
            end
        end)

        local net = RS:FindFirstChild("Packages")
        net = net and net:FindFirstChild("Net")
        for _, nm in ipairs({ "RE/CombatService/ApplyImpulse", "RE/Ragdoll" }) do
            local re = net and net:FindFirstChild(nm)
            if re and re:IsA("RemoteEvent") then
                bind(re.OnClientEvent, function() hitNow() end)
            end
        end

        bind(character.DescendantAdded, function()
            if isRagdolled() then cleanupRagdoll() end
        end)

        bind(RunService.PreSimulation, function()
            if not holdPos or lockOff() or not holding() then return end
            local rp = myRoot()
            if rp then pcall(pin, rp) end
        end)

        bind(RunService.Heartbeat, function(dt)
            local rp = myRoot()
            if not rp then return end
            if ragLeft() > 0 then
                enableControls()
                if not holdPos then hitNow() end
            end
            if isRagdolled() then cleanupRagdoll() end
            if holdPos then
                if lockOff() then
                    holdPos = nil
                elseif holding() then
                    pcall(pin, rp, dt)
                else
                    holdPos = nil
                    pcall(function()
                        rp.AssemblyLinearVelocity = Vector3.zero
                        rp.AssemblyAngularVelocity = Vector3.zero
                    end)
                end
            else
                lastSafe = rp.Position
            end
        end)

        local CAM_BIND = "iCollectProSemiTPCamLock"
        pcall(RunService.UnbindFromRenderStep, RunService, CAM_BIND)
        pcall(function()
            RunService:BindToRenderStep(CAM_BIND, Enum.RenderPriority.Camera.Value - 1, function()
                if _G.iCollectPro_SemiTP_ResetBusy == true then return end
                local cam = workspace.CurrentCamera
                local ch = LP.Character
                if not cam or not ch or not hum.Parent then return end
                local sub = cam.CameraSubject
                if sub ~= hum and typeof(sub) == "Instance" and sub:IsDescendantOf(ch) then
                    cam.CameraSubject = hum
                end
            end)
        end)
        table.insert(connections, {
            Disconnect = function() pcall(RunService.UnbindFromRenderStep, RunService, CAM_BIND) end,
        })

        enableControls()
        cleanupRagdoll()
    end

    local function attach(char)
        character = char
        humanoid = char:WaitForChild("Humanoid", 10)
        rootPart = char:WaitForChild("HumanoidRootPart", 10)
        animator = humanoid and humanoid:WaitForChild("Animator", 10)
    end

    _G.iCollectPro_SemiTP_SoftenAntiDie = function()
        if humanoid then soften(humanoid) end
    end

    function iCollectProAntiRagdoll.set(on)
        AntiRagdollEnabled = on and true or false
        if AntiRagdollEnabled then
            setup()
        else
            clear()
            if humanoid then soften(humanoid) end
        end
    end

    local arCharConn
    arCharConn = LP.CharacterAdded:Connect(function(char)
        if not iCollectPro_ALIVE() then arCharConn:Disconnect() return end
        clear()
        character, humanoid, rootPart, animator = nil, nil, nil, nil
        local h = char:WaitForChild("Humanoid", 10)
        local r = char:WaitForChild("HumanoidRootPart", 10)
        if not h or not r then return end
        task.wait(0.2)
        attach(char)
        setup()
    end)

    if LP.Character then
        task.spawn(function()
            attach(LP.Character)
            setup()
        end)
    end
end

-- Anti Rocket (admin panel "rocket"): the server pushes the character up with a VectorForce
-- and PlatformStands the humanoid. Physics is client-owned, so strip the force and stand back up.
local iCollectProAntiRocket = {}
do
    local LP = player
    local charConn
    local nextTick = 0

    local function off()
        return not AntiRocketEnabled or _G.iCollectPro_SemiTP_ResetBusy == true
    end

    local function strip(char)
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("VectorForce") then pcall(function() d:Destroy() end) end
        end
    end

    local function hookChar(char)
        if charConn then charConn:Disconnect() charConn = nil end
        if not char then return end
        strip(char)
        charConn = char.DescendantAdded:Connect(function(d)
            if off() or not d:IsA("VectorForce") then return end
            task.defer(function() pcall(function() d:Destroy() end) end)
        end)
    end

    local hbConn
    hbConn = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then
            hbConn:Disconnect()
            if charConn then charConn:Disconnect() end
            return
        end
        if off() then return end
        local now = os.clock()
        if now < nextTick then return end
        nextTick = now + 0.1

        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not (hum and hrp) then return end

        strip(char)
        if hum.PlatformStand or hum:GetState() == Enum.HumanoidStateType.PlatformStanding then
            hum.PlatformStand = false
            local v = hrp.AssemblyLinearVelocity
            if v.Y > 5 then
                hrp.AssemblyLinearVelocity = Vector3.new(v.X, 0, v.Z)
            end
            pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
        end
    end)

    -- Rocket fling guard (own function: register budget). The rocket = a VectorForce on the root, then an
    -- explosion that ragdolls + launches you. From the moment the force shows up (or an Explosion lands
    -- near you) for 8s: remember where you stood, cancel any upward/launch velocity every physics step,
    -- and put you back on that spot if you still got lifted. Walking sideways keeps working.
    ;(function()
        -- runs with Anti Admin OR Anti Ragdoll on
        local function off()
            return not (AntiRocketEnabled or AntiRagdollEnabled) or _G.iCollectPro_SemiTP_ResetBusy == true
        end
        local guardUntil, anchor = 0, nil
        local function arm()
            if off() then return end
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            if os.clock() > guardUntil then anchor = hrp.Position end
            guardUntil = os.clock() + 8
        end
        iCollectPro.rocketArm = arm
        local lastFree = nil
        local function step()
            local ch = LP.Character
            local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
            local hum = ch and ch:FindFirstChildOfClass("Humanoid")
            if not (hrp and hum) then return end
            local busy = _G.iCollectPro_SemiTP_Busy == true or _G.iCollectPro_SemiTP_ResetBusy == true
            local v = hrp.AssemblyLinearVelocity
            if os.clock() < guardUntil and anchor and not busy and not off() then
                hum.PlatformStand = false
                local flat = Vector3.new(v.X, 0, v.Z)
                if flat.Magnitude > hum.WalkSpeed + 4 then flat = flat.Unit * hum.WalkSpeed end
                hrp.AssemblyLinearVelocity = Vector3.new(flat.X, math.min(v.Y, 0), flat.Z)
                hrp.AssemblyAngularVelocity = Vector3.zero
                local p = hrp.Position
                anchor = Vector3.new(p.X, anchor.Y, p.Z)
                if p.Y - anchor.Y > 3 then
                    local _, yaw = hrp.CFrame:ToEulerAnglesYXZ()
                    hrp.CFrame = CFrame.new(anchor) * CFrame.Angles(0, yaw, 0)
                end
                return
            end
            -- outside a rocket: any sudden fling (>150 studs/s) we didn't cause gets cancelled
            -- (falling is not a fling: only a sideways or upward launch counts)
            if not busy and not off() and (Vector3.new(v.X, 0, v.Z).Magnitude > 150 or v.Y > 120)
                and not ch:FindFirstChildOfClass("Tool") then
                hrp.AssemblyLinearVelocity = Vector3.zero
                hrp.AssemblyAngularVelocity = Vector3.zero
                if lastFree then hrp.CFrame = lastFree end
                return
            end
            if v.Magnitude < 60 and hum.FloorMaterial ~= Enum.Material.Air then lastFree = hrp.CFrame end
        end
        local c1, c2, c3
        c1 = RunService.PreSimulation:Connect(function()
            if not iCollectPro_ALIVE() then c1:Disconnect() c2:Disconnect() c3:Disconnect() return end
            step()
        end)
        c2 = RunService.Heartbeat:Connect(step)
        -- "not even bothered": while the guard is up, anything the rocket bolts onto us is removed or
        -- hidden on our screen - the push force itself, fire/smoke/particles/trails, its sounds - and an
        -- explosion next to us is made invisible and harmless. Physics of our character is ours, so
        -- other players see us stay put too.
        local FX = { ParticleEmitter = true, Fire = true, Smoke = true, Sparkles = true, Trail = true, Beam = true, Sound = true }
        local function hideFx(d)
            pcall(function()
                if d:IsA("Sound") then d.Volume = 0; d:Stop() else d.Enabled = false end
            end)
        end
        c3 = Workspace.DescendantAdded:Connect(function(d)
            local ch = LP.Character
            if d:IsA("Explosion") then
                local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
                if hrp and not off() and (d.Position - hrp.Position).Magnitude <= math.max(d.BlastRadius, 12) + 6 then
                    arm()
                    pcall(function() d.BlastPressure = 0; d.DestroyJointRadiusPercent = 0; d.Visible = false end)
                end
            elseif ch and d:IsDescendantOf(ch) then
                if d:IsA("VectorForce") then
                    arm()
                    if not off() and not _G.iCollectPro_SemiTP_Busy then task.defer(function() pcall(function() d:Destroy() end) end) end
                elseif FX[d.ClassName] and os.clock() < guardUntil and not off() then
                    hideFx(d)
                end
            end
        end)
    end)()

    local rcCharConn
    rcCharConn = LP.CharacterAdded:Connect(function(char)
        if not iCollectPro_ALIVE() then rcCharConn:Disconnect() return end
        char:WaitForChild("HumanoidRootPart", 10)
        hookChar(char)
    end)
    if LP.Character then task.spawn(hookChar, LP.Character) end

    -- the rest of the admin panel: the game runs each command's effects.Victim on your client
    -- (jumpscare, balloon low gravity, inverse controls, night vision). Those modules are shared
    -- tables looked up at call time, so swap Victim for a no-op while the toggle is on.
    -- jail / tiny / morph happen on the server; only the morph model can be hidden locally.
    local VICTIMS = { "jumpscare", "balloon", "inverse", "nightvision" }
    local cmds = ReplicatedStorage:FindFirstChild("Datas")
    cmds = cmds and cmds:FindFirstChild("AdminCommands")
    for _, name in ipairs(VICTIMS) do
        pcall(function()
            local m = require(cmds[name])
            if type(m.effects) == "table" and type(m.effects.Victim) == "function" then
                local orig = rawget(m.effects, "__semiOrigVictim") or m.effects.Victim
                m.effects.__semiOrigVictim = orig
                m.effects.Victim = function(...)
                    if AntiRocketEnabled then return end
                    return orig(...)
                end
            end
        end)
    end

    local charCtrl
    pcall(function() charCtrl = require(ReplicatedStorage.Controllers.CharacterController) end)
    local adminConn, adminNext = nil, 0
    adminConn = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then adminConn:Disconnect() return end
        if off() or os.clock() < adminNext then return end
        adminNext = os.clock() + 0.1
        -- backstops in case a command landed before the wrap (inverse controls, balloon gravity)
        if charCtrl and charCtrl.Controls and charCtrl.originalMoveFunction
            and charCtrl.Controls.moveFunction ~= charCtrl.originalMoveFunction then
            charCtrl.Controls.moveFunction = charCtrl.originalMoveFunction
        end
        if Workspace.Gravity < 150 then Workspace.Gravity = 196.2 end
        local char = LP.Character
        if char then
            for _, m in ipairs(char:GetChildren()) do
                if m:IsA("Model") then
                    for _, p in ipairs(m:GetDescendants()) do
                        if (p:IsA("BasePart") or p:IsA("Decal")) and p.LocalTransparencyModifier ~= 1 then
                            p.LocalTransparencyModifier = 1
                        end
                    end
                end
            end
        end
    end)

    function iCollectProAntiRocket.set(on)
        AntiRocketEnabled = on and true or false
        if AntiRocketEnabled and LP.Character then strip(LP.Character) end
    end
end

-- Anti Bee / Anti Disco: the server fires UseItem("Bee Attack") / UseItem("Boogie") and the game's
-- BeeLauncherController / BoogieBombController run the whole effect locally (inverted controls, blur,
-- FOV, shake). Undo each piece as it lands and block the disco shake before it starts.
-- Anti Gummy Bear: a hit sets BlockTools (tools off ~3s) and drops workspace.GummyBear.
local iCollectProAntiFx = {}
do
    local LP = player
    local Lighting = game:GetService("Lighting")
    local net = ReplicatedStorage:FindFirstChild("Packages")
    net = net and net:FindFirstChild("Net")

    local charCtrl, camCtrl
    pcall(function() charCtrl = require(ReplicatedStorage.Controllers.CharacterController) end)
    pcall(function() camCtrl = require(ReplicatedStorage.Controllers.CameraController) end)
    local boomSound = ReplicatedStorage.Controllers:FindFirstChild("ItemController")
    boomSound = boomSound and boomSound:FindFirstChild("BoogieBombController")
    boomSound = boomSound and boomSound:FindFirstChild("BOOM")

    local function defaultFov()
        local ok, f = pcall(function() return camCtrl:GetDefaultFov() end)
        return ok and tonumber(f) or 70
    end

    -- The game's UseItem remote is hashed now (RE/<hash>); "RE/UseItem" still exists but never fires.
    -- Find the real one by scanning every remote for listeners that come from the two controllers,
    -- then switch those listeners off. The undo code further down stays as a backstop.
    local FX_SRC = { BeeLauncherController = "bee", BoogieBombController = "disco", PaintballGunController = "paint" }
    local fxConns, fxRemote = {}, nil

    local function scanRemote(r, into)
        local ok, cons = pcall(getconnections, r.OnClientEvent)
        if not ok or type(cons) ~= "table" then return end
        for _, c in ipairs(cons) do
            local fn = c.Function
            if type(fn) == "function" then
                local okS, src = pcall(debug.info, fn, "s")
                local kind = okS and FX_SRC[tostring(src):match("([^%.]+)$") or ""]
                if kind then
                    fxRemote = r
                    into[#into + 1] = { c = c, kind = kind }
                end
            end
        end
    end

    local function scanFx()
        if not net or type(getconnections) ~= "function" then return end
        local found = {}
        if fxRemote and fxRemote.Parent then
            scanRemote(fxRemote, found)
        else
            for _, r in ipairs(net:GetChildren()) do
                if r:IsA("RemoteEvent") then scanRemote(r, found) end
            end
        end
        fxConns = found
    end

    local function applyMute()
        for _, e in ipairs(fxConns) do
            local mute = iCollectProFx[e.kind]
            pcall(function() if mute then e.c:Disable() else e.c:Enable() end end)
        end
    end

    pcall(scanFx)
    applyMute()

    -- The disco shake is the one piece with no handle to undo: it is started through
    -- ShakePresets.BindShakeToCamera, which the game looks up on the shared table at call time,
    -- so wrap it and refuse calls coming from BoogieBombController.
    pcall(function()
        local presets = require(ReplicatedStorage.Shared.ShakePresets)
        local orig = rawget(presets, "__semiOrigBind") or presets.BindShakeToCamera
        presets.__semiOrigBind = orig
        presets.BindShakeToCamera = function(...)
            if iCollectProFx.disco then
                local ok, src = pcall(debug.info, 2, "s")
                if ok and tostring(src):find("BoogieBombController", 1, true) then
                    return function() end
                end
            end
            return orig(...)
        end
    end)

    local function undoBee()
        if charCtrl and charCtrl.Controls and charCtrl.originalMoveFunction then
            charCtrl.Controls.moveFunction = charCtrl.originalMoveFunction
        end
        for _, e in ipairs(Lighting:GetChildren()) do
            if e.Name == "BeeBlur" or (e:IsA("ColorCorrectionEffect") and e.Name == "ColorCorrection") then e:Destroy() end
        end
        local cam = Workspace.CurrentCamera
        if cam then cam.FieldOfView = 70 end
    end

    local discoUntil = 0
    local function discoOn() return iCollectProFx.disco and os.clock() < discoUntil end

    local function undoDisco()
        for _, e in ipairs(Lighting:GetChildren()) do
            if e.Name == "DiscoEffect" or (e:IsA("BlurEffect") and e.Name == "Blur") then e:Destroy() end
        end
        local cc = Lighting:FindFirstChild("ColorCCorrection")
        if cc then cc.Enabled = true end
        if boomSound and boomSound.IsPlaying then boomSound:Stop() end
    end

    local function onUseItem(kind)
        if kind == "Bee Attack" and iCollectProFx.bee then
            task.defer(undoBee)
            task.delay(0.1, undoBee)
        elseif kind == "Boogie" and iCollectProFx.disco then
            discoUntil = os.clock() + 10.5
            undoDisco()
            task.defer(undoDisco)
        end
    end

    local hooked = {}
    local function hookRemote(r)
        if not r or hooked[r] then return end
        hooked[r] = true
        local conn
        conn = r.OnClientEvent:Connect(function(kind)
            if not iCollectPro_ALIVE() then conn:Disconnect() return end
            onUseItem(kind)
        end)
    end
    hookRemote(fxRemote)
    hookRemote(net and net:FindFirstChild("RE/UseItem"))

    -- the controllers connect when their module is required; rescan so a late connection can't slip past
    task.spawn(function()
        while iCollectPro_ALIVE() do
            task.wait(10)
            pcall(scanFx)
            applyMute()
            hookRemote(fxRemote)
        end
    end)

    do
        local lConn
        lConn = Lighting.ChildAdded:Connect(function(e)
            if not iCollectPro_ALIVE() then lConn:Disconnect() return end
            if discoOn() and (e.Name == "DiscoEffect" or (e:IsA("BlurEffect") and e.Name == "Blur")) then
                task.defer(undoDisco)
            end
        end)

        -- the disco FOV tween repeats for 10s with no handle to cancel; win every frame by writing
        -- the FOV last in the render step
        local BIND = "iCollectProSemiTPAntiDisco"
        pcall(RunService.UnbindFromRenderStep, RunService, BIND)
        RunService:BindToRenderStep(BIND, Enum.RenderPriority.Last.Value, function()
            if not iCollectPro_ALIVE() then pcall(RunService.UnbindFromRenderStep, RunService, BIND) return end
            if not discoOn() then return end
            local cam = Workspace.CurrentCamera
            if cam then cam.FieldOfView = defaultFov() end
            if boomSound and boomSound.IsPlaying then boomSound:Stop() end
        end)
    end

    local function unGummy()
        if not iCollectProFx.gummy then return end
        if LP:GetAttribute("BlockTools") then LP:SetAttribute("BlockTools", false) end
        local ch = LP.Character
        if ch and ch:GetAttribute("BackpackReady") == false then ch:SetAttribute("BackpackReady", true) end
        local g = Workspace:FindFirstChild("GummyBear")
        if g then pcall(function() g:Destroy() end) end
    end

    local gConn1, gConn2
    gConn1 = LP:GetAttributeChangedSignal("BlockTools"):Connect(function()
        if not iCollectPro_ALIVE() then gConn1:Disconnect() return end
        unGummy()
    end)
    gConn2 = Workspace.ChildAdded:Connect(function(c)
        if not iCollectPro_ALIVE() then gConn2:Disconnect() return end
        if c.Name == "GummyBear" and iCollectProFx.gummy then task.defer(unGummy) end
    end)

    function iCollectProAntiFx.setBee(on) iCollectProFx.bee = on and true or false; applyMute() end
    function iCollectProAntiFx.setDisco(on) iCollectProFx.disco = on and true or false; applyMute() end
    function iCollectProAntiFx.setGummy(on) iCollectProFx.gummy = on and true or false; unGummy() end
    function iCollectProAntiFx.setPaint(on) iCollectProFx.paint = on and true or false; applyMute() end
    -- the same hashed remote carries item use both ways; Anti Body Swap fires it to swap back
    function iCollectProAntiFx.useRemote() return fxRemote end
    -- lets the MCP probe check the state without a real hit
    iCollectPro.fx = iCollectProFx
    iCollectPro.cfg = HubConfig
    iCollectPro.antiDiscoTest = function() discoUntil = os.clock() + 10.5; undoDisco() end
    iCollectPro.antiFxState = function()
        local s = {}
        for _, e in ipairs(fxConns) do s[#s + 1] = e.kind .. "=" .. tostring(e.c.Enabled) end
        return (fxRemote and fxRemote.Name or "none"), s
    end
end

-- Anti WebSling: a Web Slinger hit sets the "Web" attribute and ties you up with a rope/align
-- constraints; clear it and stand back up. Anti Body Swap: when someone holding a Body Swap Potion
-- trades places with you, swap straight back with your own potion (needs one in your backpack).
-- Infinite Jump: jump again from the air.
do
    local LP = player

    -- The web is all server-side (the tool only sends the mouse hit): the server sets "Web" +
    -- "BlockTools", PlatformStands you and ties you to something OUTSIDE your character (the Gummy
    -- Bear version welds your HRP from a model in workspace - a scan of the character alone never
    -- finds it). So every joint/constraint on our parts that leads out of the character is cut
    -- (BasePart:GetJoints), stray attachments like WebTargetAttch go, and we're stood back up -
    -- repeated every frame while "Web" stays set, since the server keeps re-applying it.
    local webUsedAt = 0
    local function unWebOnce(char)
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not (hum and hrp) then return end
        if LP:GetAttribute("Web") then LP:SetAttribute("Web", false) end
        if char:GetAttribute("Web") then char:SetAttribute("Web", false) end
        if LP:GetAttribute("BlockTools") then LP:SetAttribute("BlockTools", false) end
        local attach = hrp:FindFirstChild("WebTargetAttch")
        if attach then attach:Destroy() end
        local cut = 0
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") then
                local ok, joints = pcall(part.GetJoints, part)
                for _, j in ipairs(ok and joints or {}) do
                    if not j:IsA("Motor6D") and not j:IsA("AnimationConstraint") then
                        local a, b
                        if j:IsA("Constraint") then
                            a = j.Attachment0 and j.Attachment0.Parent
                            b = j.Attachment1 and j.Attachment1.Parent
                        else
                            a, b = j.Part0, j.Part1
                        end
                        local out = (a and not a:IsDescendantOf(char)) or (b and not b:IsDescendantOf(char))
                            or not j:IsDescendantOf(char)
                        if out then cut = cut + 1 pcall(j.Destroy, j) end
                    end
                end
            elseif (part:IsA("RopeConstraint") or part:IsA("SpringConstraint")) then
                cut = cut + 1
                pcall(part.Destroy, part)
            end
        end
        local gb = Workspace:FindFirstChild("GummyBear")
        if gb and (gb.Position - hrp.Position).Magnitude < 12 then pcall(gb.Destroy, gb) end
        if hum.PlatformStand then hum.PlatformStand = false end
        local s = hum:GetState()
        if s == Enum.HumanoidStateType.Physics or s == Enum.HumanoidStateType.PlatformStanding
            or s == Enum.HumanoidStateType.FallingDown then
            pcall(hum.ChangeState, hum, Enum.HumanoidStateType.GettingUp)
        end
        return cut
    end

    local webbing = false
    local function unWeb(char)
        if not iCollectProFx.web or not char or webbing then return end
        -- we just fired our OWN Web Slinger: leave it (only for 1.5s - holding one no longer
        -- switches Anti WebSling off)
        if os.clock() - webUsedAt < 1.5 then return end
        webbing = true
        local t0, cuts = os.clock(), 0
        while iCollectPro_ALIVE() and os.clock() - t0 < 6 do
            local c = unWebOnce(char) or 0
            cuts = cuts + c
            local w = LP:GetAttribute("Web") or char:GetAttribute("Web")
            -- keep at it 0.5s past the last sign of the web
            if not (w or c > 0) and os.clock() - t0 > 0.5 then break end
            RunService.Heartbeat:Wait()
        end
        webbing = false
        pcall(function() iCollectPro.log(string.format("ANTI WEBSLING: freed  (%d links cut, %.2fs)", cuts, os.clock() - t0)) end)
    end

    local function webHit(src)
        local v = src:GetAttribute("Web")
        if v ~= nil and v ~= false then task.spawn(unWeb, LP.Character) end
    end

    local webConns = {}
    local function hookWeb(char)
        for _, c in ipairs(webConns) do c:Disconnect() end
        webConns = { LP:GetAttributeChangedSignal("Web"):Connect(function() webHit(LP) end) }
        if char then
            table.insert(webConns, char:GetAttributeChangedSignal("Web"):Connect(function() webHit(char) end))
            -- our own Web Slinger's shot (so we don't undo it)
            local function ownTool(c)
                if c:IsA("Tool") and c.Name == "Web Slinger" then
                    table.insert(webConns, c.Activated:Connect(function() webUsedAt = os.clock() end))
                end
            end
            table.insert(webConns, char.ChildAdded:Connect(ownTool))
            local own = char:FindFirstChild("Web Slinger")
            if own then ownTool(own) end
            -- a web can land without the attribute arriving first: PlatformStand forced on us
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                table.insert(webConns, hum:GetPropertyChangedSignal("PlatformStand"):Connect(function()
                    if hum.PlatformStand and (LP:GetAttribute("Web") or LP:GetAttribute("BlockTools")) then
                        task.spawn(unWeb, char)
                    end
                end))
            end
        end
    end
    hookWeb(LP.Character)
    local webCharConn
    webCharConn = LP.CharacterAdded:Connect(function(char)
        if not iCollectPro_ALIVE() then
            webCharConn:Disconnect()
            for _, c in ipairs(webConns) do c:Disconnect() end
            return
        end
        hookWeb(char)
    end)

    local SWAP_POTION = "Body Swap Potion"
    local threat, prevOthers, prevSelf, lastCounter = {}, {}, nil, 0

    local function myPotion()
        local ch, bp = LP.Character, LP:FindFirstChild("Backpack")
        return (ch and ch:FindFirstChild(SWAP_POTION)) or (bp and bp:FindFirstChild(SWAP_POTION))
    end

    local function swapBack(target, home)
        local ch = LP.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local potion = myPotion()
        local remote = iCollectProAntiFx.useRemote()
        if not (hum and potion and remote) or potion:GetAttribute("CooldownTime") then
            iCollectPro.log(string.format("SWAPPED by %s but can't swap back (potion=%s remote=%s cooldown=%s)",
                target.Name, tostring(potion ~= nil), tostring(remote ~= nil),
                tostring(potion and potion:GetAttribute("CooldownTime"))))
            return
        end
        -- the spot we get swapped back to: if it's an enemy podium, begin holding its prompt NOW
        -- (same trick as the 2nd / 3rd floor clone-in) so the hold is already done when we land
        local homePrompt = home and iCollectPro.enemyPromptAt and iCollectPro.enemyPromptAt(home)
        -- banked even while the prompt shows disabled (a just-dropped brainrot): the hold runs on the
        -- game's own handlers, and it's already old enough once the prompt comes back
        local homeCtx = homePrompt and iCollectProHold.startStealHold(homePrompt)
        iCollectPro.logT0 = os.clock()
        iCollectPro.log(string.format("---- SWAPPED by %s  home podium=%s  enabled=%s  hold banked=%s",
            target.Name, homePrompt and homePrompt:GetFullName():match("AnimalPodiums%.(%d+)") or "none",
            tostring(homePrompt and homePrompt.Enabled), tostring(homeCtx ~= nil and homeCtx ~= false)))
        local held = ch:FindFirstChildOfClass("Tool")
        pcall(function() hum:EquipTool(potion) end)
        local t0 = os.clock()
        while potion.Parent ~= ch and os.clock() - t0 < 0.5 do task.wait() end
        if potion.Parent == ch then
            local hrp = ch:FindFirstChild("HumanoidRootPart")
            local from = hrp and hrp.Position
            pcall(function() remote:FireServer(target) end)
            -- back on our own spot: if that's an enemy podium, grab its brainrot the moment we land
            task.spawn(function()
                local t0 = os.clock()
                while hrp and hrp.Parent and from and (hrp.Position - from).Magnitude < 3 and os.clock() - t0 < 1.5 do
                    RunService.Heartbeat:Wait()
                end
                local moved = hrp and from and (hrp.Position - from).Magnitude or -1
                iCollectPro.log(string.format("swap-back landed=%s (moved %.1f)  off home %.1f  hold age %.2fs  prompt enabled=%s",
                    tostring(moved >= 3), moved, (hrp and home) and (hrp.Position - home).Magnitude or -1,
                    homeCtx and tick() - (homeCtx.holdBeganAt or 0) or -1, tostring(homePrompt and homePrompt.Enabled)))
                local lt = os.clock()
                -- up to 8s: covers a just-dropped brainrot whose prompt is still switched off
                local got = iCollectPro.grabUnderFeet and iCollectPro.grabUnderFeet(8, homePrompt, homeCtx or nil)
                iCollectPro.log(string.format("%s %.2fs after landing  (prompt enabled now=%s)", got and "GRABBED" or "NO GRAB",
                    os.clock() - lt, tostring(homePrompt and homePrompt.Enabled)))
            end)
        end
        task.wait(0.3)
        if held and held ~= potion and held.Parent then
            pcall(function() hum:EquipTool(held) end)
        else
            pcall(function() hum:UnequipTools() end)
        end
    end

    local swapConn, swapNext = nil, 0
    swapConn = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then swapConn:Disconnect() return end
        local hrp = iCollectProFx.swap and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if not hrp then
            if prevSelf then prevSelf = nil; table.clear(prevOthers) end
            return
        end
        local now = tick()
        if now < swapNext then return end
        swapNext = now + 0.05
        local me = hrp.Position
        local others = {}
        for _, p in ipairs(Players:GetPlayers()) do
            local r = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
            if r then
                others[p] = r.Position
                local tool = p.Character:FindFirstChildOfClass("Tool")
                if tool and tool.Name == SWAP_POTION then threat[p] = now end
            end
        end
        -- a swap: I jumped onto where they stood while they landed where I stood
        if prevSelf and (me - prevSelf).Magnitude >= 3 and now - lastCounter >= 2 and not _G.iCollectPro_SemiTP_Busy then
            for p, was in pairs(prevOthers) do
                local nowPos = others[p]
                if nowPos and (me - was).Magnitude <= 6 and (nowPos - prevSelf).Magnitude <= 6
                    and threat[p] and now - threat[p] <= 6 then
                    lastCounter = now
                    threat[p] = nil
                    task.spawn(swapBack, p, prevSelf)
                    break
                end
            end
        end
        prevSelf, prevOthers = me, others
    end)

    -- Logged 2026-09-28: forcing the Jumping state while already in the air got the character killed
    -- by the server on every mid-air jump (health 100 -> 0 the same frame, then a respawn). So only
    -- push upward in the air and never change the humanoid state; a normal ground jump is left alone.
    -- Infinite Jump, same as MEERKO EXTRAS: every jump press sets upward velocity to JumpPower,
    -- and holding Space keeps doing it every frame so you can climb
    local held = false
    local function hop()
        local ch = LP.Character
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum or hum.Health <= 0 then return end
        hrp.Velocity = Vector3.new(hrp.Velocity.X, hum.JumpPower or 50, hrp.Velocity.Z)
    end
    local jumpConns = {}
    jumpConns[1] = UserInputService.JumpRequest:Connect(function()
        if iCollectProFx.infJump then hop() end
    end)
    jumpConns[2] = UserInputService.InputBegan:Connect(function(i, g)
        if not g and i.KeyCode == Enum.KeyCode.Space then held = true end
    end)
    jumpConns[3] = UserInputService.InputEnded:Connect(function(i)
        if i.KeyCode == Enum.KeyCode.Space then held = false end
    end)
    jumpConns[4] = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then
            for _, c in ipairs(jumpConns) do c:Disconnect() end
            return
        end
        if held and iCollectProFx.infJump then hop() end
    end)

    -- The "mid-air jump kill" is the game's OWN client: CharacterController counts Freefall->Jumping /
    -- Jumping->Jumping and sets Health = 0 on the 5th in 60s (+ExtraJumps) - only when ServerAuthority
    -- is off, as on public servers now (read from the game 2026-09-30). The local death then goes to
    -- the server = the "HP 0 right after the grab" deaths with Infinite Jump on. That one StateChanged
    -- listener (the only one CharacterController has) is switched off on every character.
    local function unJumpKill(ch)
        if type(getconnections) ~= "function" then return end
        local hum = ch and ch:WaitForChild("Humanoid", 10)
        if not hum then return end
        for _ = 1, 20 do
            local found = false
            for _, c in ipairs(getconnections(hum.StateChanged)) do
                local f = c.Function
                local src = type(f) == "function" and (debug.info(f, "s") or "") or ""
                if src:find("CharacterController", 1, true) then
                    found = true
                    pcall(function() c:Disable() end)
                end
            end
            if found then
                pcall(function() iCollectPro.log("jump-count kill switched off") end)
                return
            end
            task.wait(0.25)
        end
    end
    task.spawn(unJumpKill, LP.Character)
    table.insert(jumpConns, LP.CharacterAdded:Connect(function(ch) task.spawn(unJumpKill, ch) end))

    function iCollectProAntiFx.setWeb(on) iCollectProFx.web = on and true or false; if iCollectProFx.web then unWeb(LP.Character) end end
    function iCollectProAntiFx.setSwap(on) iCollectProFx.swap = on and true or false end
    function iCollectProAntiFx.setInfJump(on) iCollectProFx.infJump = on and true or false end
end

-- Auto Destroy Sentry / Doge (from Aurora): an enemy sentry is a workspace part "Sentry_<userId>"
-- (or SentryCandy_), an attack doge a model "PlayerName_<name>_Doge". Pull it in front of you and
-- hit it with your Bat until it breaks. Skipped while you carry or fly so it never swaps your tool.
-- Aimbot / Auto Spam: aim items read Packages.PlayerMouse.Hit/Target, so point those at the nearest
-- player while you hold one; Auto Spam also fires it every 0.1s.
-- Server Ghost: a see-through copy of you drawn where the server last saw you (your position one
-- network ping ago) - if the ghost lags behind at the moment of a grab, the server saw you there.
-- Player ESP: a highlight + name/distance tag on every player.
do
    local LP = player

    local function canFight()
        local ch = LP.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if not (hum and hrp) or hum.Health <= 0 then return nil end
        if LP:GetAttribute("Stealing") or _G.iCollectPro_SemiTP_Busy then return nil end
        return ch, hum, hrp
    end

    local function swing(ch, hum)
        local bat = ch:FindFirstChild("Bat") or (LP:FindFirstChild("Backpack") and LP.Backpack:FindFirstChild("Bat"))
        if not bat then return false end
        if bat.Parent ~= ch then pcall(function() hum:EquipTool(bat) end) end
        if bat.Parent == ch and bat.Enabled ~= false then pcall(function() bat:Activate() end) end
        return true
    end

    local function enemySentry(p)
        if not p:IsA("BasePart") then return false end
        local id = p.Name:match("^Sentry_(%d+)$") or p.Name:match("^SentryCandy_(%d+)$")
        return id ~= nil and id ~= tostring(LP.UserId)
    end

    local function enemyDoge(m)
        if not m:IsA("Model") then return false end
        local who = m.Name:match("^PlayerName_(.+)_Doge$")
        if not who or who == LP.Name or who == LP.DisplayName then return false end
        local h = m:FindFirstChildOfClass("Humanoid")
        return h ~= nil and h.Health > 0
    end

    local busyDefence = false
    local function breakIt(obj, isDoge)
        if busyDefence then return end
        busyDefence = true
        task.spawn(function()
            local limit = os.clock() + (isDoge and 10 or 3)
            while iCollectPro_ALIVE() and obj.Parent and os.clock() < limit do
                if isDoge and not (iCollectProFx.doge and enemyDoge(obj)) then break end
                if not isDoge and not iCollectProFx.sentry then break end
                local ch, hum, hrp = canFight()
                if not ch then break end
                local part = isDoge and (obj.PrimaryPart or obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChildWhichIsA("BasePart")) or obj
                if not part or (part.Position - hrp.Position).Magnitude > 220 then break end
                local look = hrp.CFrame.LookVector
                local spot = hrp.Position + look * 4 + Vector3.new(0, isDoge and 0 or 1.2, 0)
                pcall(function()
                    for _, d in ipairs((isDoge and obj or part):GetDescendants()) do
                        if d:IsA("BasePart") then d.CanCollide = false end
                    end
                    part.CanCollide = false
                    part.AssemblyLinearVelocity = Vector3.zero
                    if isDoge then obj:PivotTo(CFrame.lookAt(spot, spot + look)) else part.CFrame = CFrame.lookAt(spot, spot + look) end
                end)
                if not swing(ch, hum) then break end
                task.wait(isDoge and 0.1 or 0.12)
            end
            busyDefence = false
        end)
    end

    local function scanDefences()
        for _, c in ipairs(Workspace:GetChildren()) do
            if iCollectProFx.sentry and enemySentry(c) then breakIt(c, false)
            elseif iCollectProFx.doge and enemyDoge(c) then breakIt(c, true) end
        end
    end
    local defConn
    defConn = Workspace.ChildAdded:Connect(function(c)
        if not iCollectPro_ALIVE() then defConn:Disconnect() return end
        task.delay(0.3, function()
            if iCollectProFx.sentry and enemySentry(c) then breakIt(c, false)
            elseif iCollectProFx.doge and enemyDoge(c) then breakIt(c, true) end
        end)
    end)
    task.spawn(function()
        while iCollectPro_ALIVE() do
            if iCollectProFx.sentry or iCollectProFx.doge then pcall(scanDefences) end
            task.wait(1)
        end
    end)

    -- aim items (from Aurora's list); anything else you hold is left alone
    local AIM_ITEMS = {}
    for _, n in ipairs({ "Blackhole Bomb", "BlowDryer", "Candy Launcher", "Candycane Bow", "Christmas Launcher",
        "Freeze Ray", "Gravity Gun", "Hunter Crossbow", "Jelly Gun", "Laser Cape", "Lava Blaster", "Paintball Gun",
        "Pumpkin Launcher", "Radioactive Airstrike", "Sabuk Bepak", "Sandal Jepit", "Slingshot", "Snowball",
        "Snowball Cannon", "Summer Soaker", "Super-GLS33", "Taser Gun", "Tripple Plungers", "Web Slinger",
        "Wormhole Tunneler", "Zombie Blaster" }) do AIM_ITEMS[n] = true end
    local mouse
    pcall(function() mouse = require(ReplicatedStorage.Packages.PlayerMouse) end)
    local aimAt, lastScan, lastShot = nil, 0, 0
    local function aimStep()
        if not (iCollectProFx.aim and mouse) then aimAt = nil return end
        local ch = LP.Character
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        local tool = ch and ch:FindFirstChildOfClass("Tool")
        if not (hrp and tool and AIM_ITEMS[tool.Name]) then aimAt = nil return end
        if not aimAt or not aimAt.Parent or os.clock() - lastScan >= 0.05 then
            lastScan = os.clock()
            local best, bestD = nil, math.huge
            for _, p in ipairs(Players:GetPlayers()) do
                local r = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                local h = r and p.Character:FindFirstChildOfClass("Humanoid")
                if r and h and h.Health > 0 then
                    local d = (r.Position - hrp.Position).Magnitude
                    if d < bestD then best, bestD = r, d end
                end
            end
            aimAt = best
        end
        if aimAt then
            pcall(function() mouse.Hit = CFrame.new(aimAt.Position); mouse.Target = aimAt end)
            if iCollectProFx.spam and os.clock() - lastShot >= 0.1 then
                lastShot = os.clock()
                pcall(function() tool:Activate() end)
            end
        end
    end
    local aimConn1, aimConn2
    aimConn1 = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then aimConn1:Disconnect() return end
        aimStep()
    end)
    -- the item reads the mouse on its own frame; re-point it right before render too
    aimConn2 = (RunService.PreRender or RunService.RenderStepped):Connect(function()
        if not iCollectPro_ALIVE() then aimConn2:Disconnect() return end
        if aimAt and aimAt.Parent and mouse then
            pcall(function() mouse.Hit = CFrame.new(aimAt.Position); mouse.Target = aimAt end)
        end
    end)

    -- Server Ghost: a see-through copy of your real body (every body part and accessory, meshes and
    -- all) with a glowing outline that shows through walls, animating with you: each frame the pose of
    -- every part is recorded and played back one ping late, so the ghost walks, jumps and holds poses
    -- exactly like you, just where the server has you. Pill tag: ping + how far behind the server is.
    -- Green = in sync, yellow = a few studs off, red = the server sees you elsewhere.
    pcall(function()
        for _, d in ipairs(Workspace:GetChildren()) do
            if d.Name == "iCollectPro_SemiTP_ServerGhost" then d:Destroy() end
        end
    end)
    local ghost = Instance.new("Model")
    ghost.Name = "iCollectPro_SemiTP_ServerGhost"
    local ghostHl = Instance.new("Highlight")
    ghostHl.FillColor = T.FILL2
    ghostHl.FillTransparency = 0.82
    ghostHl.OutlineColor = T.FILL2
    ghostHl.OutlineTransparency = 0
    ghostHl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    ghostHl.Adornee = ghost
    ghostHl.Parent = ghost
    -- ghostSrc[i] = your real part, ghostParts[i] = its see-through copy (same mesh, same size)
    local ghostSrc, ghostParts, ghostCFs, ghostChar, ghostDirty = {}, {}, {}, nil, true
    local anchorPart, charConns = nil, {}
    local KEEP = { SpecialMesh = true, Decal = true, SurfaceAppearance = true }
    local function buildGhost(char)
        for _, p in ipairs(ghostParts) do p:Destroy() end
        for _, c in ipairs(charConns) do c:Disconnect() end
        table.clear(ghostSrc); table.clear(ghostParts); table.clear(ghostCFs); table.clear(charConns)
        ghostChar, ghostDirty, anchorPart = char, false, nil
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        for _, src in ipairs(char:GetDescendants()) do
            -- body parts + accessories; held tools and the invisible root are left out
            if src:IsA("BasePart") and src ~= hrp and src.Transparency < 1 and not src:FindFirstAncestorOfClass("Tool") then
                local arch = src.Archivable
                src.Archivable = true
                local ok, p = pcall(function() return src:Clone() end)
                src.Archivable = arch
                if ok and p then
                    -- only the look stays: joints/welds would tie the copy to your real body
                    for _, d in ipairs(p:GetChildren()) do
                        if not KEEP[d.ClassName] then d:Destroy() end
                    end
                    p.Anchored, p.CanCollide, p.CanQuery, p.CanTouch, p.CastShadow = true, false, false, false, false
                    p.Massless = true
                    p.Transparency = math.max(src.Transparency, 0.72)
                    p.Parent = ghost
                    ghostSrc[#ghostSrc + 1] = src
                    ghostParts[#ghostParts + 1] = p
                    ghostCFs[#ghostCFs + 1] = src.CFrame
                    if src.Name == "Head" then anchorPart = p end
                end
            end
        end
        anchorPart = anchorPart or ghostParts[1]
        -- an accessory put on / taken off (or a respawned limb) rebuilds the copy
        local function mark(d)
            if d:IsA("BasePart") and not d:FindFirstAncestorOfClass("Tool") then ghostDirty = true end
        end
        charConns[1] = char.DescendantAdded:Connect(mark)
        charConns[2] = char.DescendantRemoving:Connect(mark)
    end

    local ghostTag = Instance.new("BillboardGui")
    ghostTag.Size = UDim2.fromOffset(150, 30)
    ghostTag.StudsOffset = Vector3.new(0, 2.6, 0)
    ghostTag.AlwaysOnTop = true
    ghostTag.LightInfluence = 0
    ghostTag.MaxDistance = 100000
    ghostTag.Parent = ghost
    local tagPill = Instance.new("Frame")
    tagPill.Size = UDim2.fromScale(1, 1)
    tagPill.BackgroundColor3 = T.BG
    tagPill.BorderSizePixel = 0
    tagPill.Parent = ghostTag
    corner(tagPill, 15)
    local tagStroke = stroke(tagPill, T.ACCENT, 2, 0.1)
    gradient(tagPill, T.SURF2, T.BG, 90)
    local ghostLbl = Instance.new("TextLabel")
    ghostLbl.Size = UDim2.new(1, -12, 1, 0)
    ghostLbl.Position = UDim2.fromOffset(6, 0)
    ghostLbl.BackgroundTransparency = 1
    ghostLbl.Font = Enum.Font.GothamBlack
    ghostLbl.TextSize = 13
    ghostLbl.TextColor3 = T.TEXT
    ghostLbl.TextStrokeTransparency = 0.6
    ghostLbl.Parent = tagPill
    local COL_OK, COL_WARN, COL_BAD = Color3.fromRGB(110, 235, 160), Color3.fromRGB(255, 205, 90), Color3.fromRGB(255, 110, 130)
    -- record on Heartbeat (after physics), draw on render and blend between the two samples around
    -- "now - lag" so the ghost glides instead of stepping from frame to frame
    local trail, ping, lastPingRead, lastTagMs = {}, 0.05, 0, -1
    local recConn, drawConn
    recConn = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then recConn:Disconnect() return end
        local hrp = iCollectProFx.ghost and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if not hrp then if #trail > 0 then table.clear(trail) end return end
        local now = os.clock()
        -- the pose: every copied part's place relative to your root this frame
        local pose = {}
        if LP.Character == ghostChar then
            local rootCF = hrp.CFrame
            for i, src in ipairs(ghostSrc) do pose[i] = rootCF:ToObjectSpace(src.CFrame) end
        end
        trail[#trail + 1] = { t = now, cf = hrp.CFrame, pose = pose }
        while #trail > 2 and now - trail[1].t > 1.5 do table.remove(trail, 1) end
        if now - lastPingRead > 0.5 then
            lastPingRead = now
            pcall(function() ping = LP:GetNetworkPing() end)
        end
    end)
    drawConn = (RunService.PreRender or RunService.RenderStepped):Connect(function()
        if not iCollectPro_ALIVE() then
            drawConn:Disconnect()
            for _, c in ipairs(charConns) do c:Disconnect() end
            ghost:Destroy()
            return
        end
        if not iCollectProFx.ghost or #trail == 0 then
            if ghost.Parent then ghost.Parent = nil end
            return
        end
        -- one-way trip plus one physics send (~1/60s)
        local at = os.clock() - (ping + 1 / 60)
        local cf = trail[1].cf
        local sa, sb, alpha = trail[1], trail[1], 0
        for i = #trail, 2, -1 do
            local a, b = trail[i - 1], trail[i]
            if a.t <= at then
                local span = b.t - a.t
                alpha = span > 0 and math.clamp((at - a.t) / span, 0, 1) or 1
                cf = a.cf:Lerp(b.cf, alpha)
                sa, sb = a, b
                break
            end
        end
        local char = LP.Character
        if char ~= ghostChar or ghostDirty or #ghostParts == 0 then
            buildGhost(char)
            table.clear(trail)
            return
        end
        if #ghostParts == 0 then return end
        -- place every copied part in its recorded pose, blended between the two samples
        local pa, pb = sa.pose, sb.pose
        for i = 1, #ghostParts do
            local oa, ob = pa[i], pb[i]
            if oa and ob then
                ghostCFs[i] = cf * oa:Lerp(ob, alpha)
            elseif oa or ob then
                ghostCFs[i] = cf * (oa or ob)
            end
        end
        Workspace:BulkMoveTo(ghostParts, ghostCFs, Enum.BulkMoveMode.FireCFrameChanged)
        if anchorPart and ghostTag.Adornee ~= anchorPart then ghostTag.Adornee = anchorPart end

        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local off = hrp and (hrp.Position - cf.Position).Magnitude or 0
        local col = off < 2 and COL_OK or (off < 6 and COL_WARN or COL_BAD)
        local ms = math.floor(ping * 1000 + 0.5)
        local key = ms * 1000 + math.floor(off * 10)
        if key ~= lastTagMs then
            lastTagMs = key
            ghostLbl.Text = string.format("SERVER  %dms  ·  %.1f", ms, off)
        end
        tagStroke.Color = col
        ghostLbl.TextColor3 = col
        ghostHl.OutlineColor = col
        if ghost.Parent ~= Workspace then ghost.Parent = Workspace end
    end)

    -- Player ESP
    local espFolder = Instance.new("ScreenGui")
    espFolder.Name = "iCollectPro_SemiTP_ESP"
    espFolder.ResetOnSpawn = false
    espFolder.Parent = safeGuiTarget
    local esp = {}
    local function clearEsp(p)
        local e = esp[p]
        if e then pcall(function() e.hl:Destroy(); e.tag:Destroy() end) esp[p] = nil end
    end
    task.spawn(function()
        while iCollectPro_ALIVE() do
            if not iCollectProFx.esp and next(esp) == nil then task.wait(0.5) continue end
            local me = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            for _, p in ipairs(Players:GetPlayers()) do
                local ch = p.Character
                local head = ch and ch:FindFirstChild("Head")
                local r = ch and ch:FindFirstChild("HumanoidRootPart")
                if iCollectProFx.esp and p ~= LP and head and r then
                    local e = esp[p]
                    if not e or e.char ~= ch then
                        clearEsp(p)
                        local hl = Instance.new("Highlight")
                        hl.FillColor = T.ACCENT2
                        hl.OutlineColor = T.FILL2
                        hl.FillTransparency = 0.75
                        hl.OutlineTransparency = 0
                        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        hl.Adornee = ch
                        hl.Parent = espFolder
                        local tag = Instance.new("BillboardGui")
                        tag.Size = UDim2.fromOffset(160, 16)
                        tag.StudsOffset = Vector3.new(0, -3.6, 0)
                        tag.AlwaysOnTop = true
                        tag.MaxDistance = 100000
                        tag.Adornee = r
                        tag.Parent = espFolder
                        local l = Instance.new("TextLabel")
                        l.Size = UDim2.fromScale(1, 1)
                        l.BackgroundTransparency = 1
                        l.Font = Enum.Font.GothamBold
                        l.TextSize = 11
                        l.TextColor3 = T.TEXT
                        l.TextStrokeTransparency = 0.4
                        l.Parent = tag
                        e = { hl = hl, tag = tag, lbl = l, char = ch }
                        esp[p] = e
                    end
                    local dist = me and math.floor((r.Position - me.Position).Magnitude) or 0
                    if e.dist ~= dist then
                        e.dist = dist
                        e.lbl.Text = p.Name .. "  " .. dist .. "m"
                    end
                else
                    clearEsp(p)
                end
            end
            for p in pairs(esp) do
                if not p.Parent then clearEsp(p) end
            end
            task.wait(0.2)
        end
        espFolder:Destroy()
    end)

    function iCollectProAntiFx.setSentry(on) iCollectProFx.sentry = on and true or false end
    function iCollectProAntiFx.setDoge(on) iCollectProFx.doge = on and true or false end
    function iCollectProAntiFx.setAim(on) iCollectProFx.aim = on and true or false end
    function iCollectProAntiFx.setSpam(on) iCollectProFx.spam = on and true or false end
    function iCollectProAntiFx.setGhost(on) iCollectProFx.ghost = on and true or false end
    function iCollectProAntiFx.setEsp(on) iCollectProFx.esp = on and true or false end
end

function iCollectPro.activate()
    task.spawn(function()
        setFFlags()
        _G.iCollectPro_SemiTP_InstaReset()
    end)
end

local AllowDisallowGui = Instance.new("ScreenGui")
AllowDisallowGui.Name = "iCollectPro_SemiTP_Allow"
AllowDisallowGui.ResetOnSpawn = false
AllowDisallowGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
AllowDisallowGui.Parent = safeGuiTarget

local MainAllowBtn = Instance.new("TextButton")
MainAllowBtn.Size = UDim2.new(0, 100, 0, 34)
MainAllowBtn.Position = UDim2.new(0.65, 36, 0, 15)
MainAllowBtn.BackgroundColor3 = T.SURF
MainAllowBtn.AutoButtonColor = false
MainAllowBtn.BorderSizePixel = 0
MainAllowBtn.Text = "WAITING"
MainAllowBtn.TextColor3 = T.TEXT
MainAllowBtn.Font = Enum.Font.GothamBlack
MainAllowBtn.TextSize = 12
MainAllowBtn.ZIndex = 20
MainAllowBtn.Parent = AllowDisallowGui
corner(MainAllowBtn, 11)
stroke(MainAllowBtn, T.STROKE, 1.2, 0.35)
addShadow(MainAllowBtn, 20)
addHover(MainAllowBtn, T.SURF, T.HOVER)
restorePos(MainAllowBtn, "allow")
makeDraggable(MainAllowBtn, MainAllowBtn, rememberPos(MainAllowBtn, "allow"))

local activeHubs = {}
local function createFloatingHub(parent, statusText)
    if activeHubs[parent] then return end
    -- friends pill, same look as the head pills: FRIENDS ON (green) / FRIENDS OFF (red)
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.fromOffset(96, 20)
    billboard.Adornee = parent
    billboard.AlwaysOnTop = true
    billboard.LightInfluence = 0
    billboard.ExtentsOffset = Vector3.new(0, 2.5, 0)
    billboard.Parent = AllowDisallowGui

    local HubFrame = Instance.new("Frame")
    HubFrame.Name = "Frame"
    HubFrame.Size = UDim2.new(1, 0, 1, 0)
    HubFrame.BackgroundColor3 = T.BG
    HubFrame.BackgroundTransparency = 0.12
    HubFrame.BorderSizePixel = 0
    HubFrame.Parent = billboard
    corner(HubFrame, 10)
    gradient(HubFrame, T.SURF2, T.BG, 90)
    stroke(HubFrame, T.STROKE, 1.5, 0.1)

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "TextLabel"
    statusLabel.Size = UDim2.new(1, -8, 1, 0)
    statusLabel.Position = UDim2.fromOffset(4, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = statusText
    statusLabel.TextColor3 = T.TEXT
    statusLabel.TextSize = 10
    statusLabel.Font = Enum.Font.GothamBlack
    statusLabel.Parent = HubFrame

    activeHubs[parent] = billboard
end

task.spawn(function()
    while task.wait(0.5) do
        if not iCollectPro_ALIVE() then return end
        local currentObjects = {}
        local myBase = findMyBase()
        local nearestPrompt = nil
        local minDist = math.huge

        for desc in pairs(iCollectProUI.friendPrompts) do
            if desc.Parent then
                local isDisallow = string.find(desc.ObjectText, "Disallow Friends", 1, true)
                local isAllow = not isDisallow and string.find(desc.ObjectText, "Allow Friends", 1, true)
                if isAllow or isDisallow then
                    local part = iCollectProUI.promptPart(desc)
                    if part then
                        currentObjects[part] = true
                        -- the prompt says "Allow" while friends are currently NOT allowed
                        local status = isAllow and "FRIENDS OFF" or "FRIENDS ON"
                        if not activeHubs[part] then createFloatingHub(part, status) end
                        local frame = activeHubs[part].Frame
                        local label = frame.TextLabel
                        if label.Text ~= status then label.Text = status end
                        local col = isAllow and Color3.fromRGB(255, 110, 130) or Color3.fromRGB(110, 235, 160)
                        label.TextColor3 = col
                        local st = frame:FindFirstChildOfClass("UIStroke")
                        if st then st.Color = col end

                        if myBase and part:IsDescendantOf(myBase) then
                            local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                            if hrp then
                                local dist = (hrp.Position - part.Position).Magnitude
                                if dist < minDist then
                                    minDist = dist
                                    nearestPrompt = desc
                                end
                            end
                        end
                    end
                end
            end
        end

        MainAllowBtn.Text = nearestPrompt and (string.find(nearestPrompt.ObjectText, "Disallow") and "DISALLOW" or "ALLOW") or "NO BASE"

        for part, bbg in pairs(activeHubs) do
            if not currentObjects[part] then
                bbg:Destroy()
                activeHubs[part] = nil
            end
        end
    end
end)

MainAllowBtn.MouseButton1Click:Connect(function()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    local myBase = findMyBase()
    local target = nil
    local minDist = math.huge

    for desc in pairs(iCollectProUI.friendPrompts) do
        if desc.Parent then
            local part = iCollectProUI.promptPart(desc)
            if part and hrp and myBase and part:IsDescendantOf(myBase) then
                local dist = (hrp.Position - part.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    target = desc
                end
            end
        end
    end
    if target then fireproximityprompt(target) end
end)

local ProgressGui = Instance.new("ScreenGui")
ProgressGui.Name = "iCollectPro_SemiTP_Progress"
ProgressGui.ResetOnSpawn = false
ProgressGui.IgnoreGuiInset = true
ProgressGui.DisplayOrder = 998
ProgressGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ProgressGui.Parent = safeGuiTarget

local PBarMain = Instance.new("Frame")
PBarMain.AnchorPoint = Vector2.new(0.5, 1)
PBarMain.Size = UDim2.new(0, 230, 0, 46)
PBarMain.Position = UDim2.new(0.5, 0, 1, -160)
PBarMain.BackgroundColor3 = T.SURF
PBarMain.BackgroundTransparency = 0.02
PBarMain.BorderSizePixel = 0
PBarMain.ZIndex = 70
PBarMain.Parent = ProgressGui
corner(PBarMain, 12)
stroke(PBarMain, T.STROKE, 1, 0.35)
stroke(PBarMain, T.ACCENT, 3, 0.84)
addShadow(PBarMain, 20)
restorePos(PBarMain, "progress")
makeDraggable(PBarMain, PBarMain, rememberPos(PBarMain, "progress"))

local PBarTitle = Instance.new("TextLabel")
PBarTitle.Size = UDim2.new(1, -12, 0, 13)
PBarTitle.Position = UDim2.fromOffset(6, 3)
PBarTitle.BackgroundTransparency = 1
PBarTitle.Font = Enum.Font.GothamBold
PBarTitle.TextSize = 11
PBarTitle.TextColor3 = T.TEXT
PBarTitle.Text = "SEMI TP"
PBarTitle.ZIndex = 72
iCollectProUI.fit(PBarTitle, 11)
PBarTitle.Parent = PBarMain

local statusStamp = 0
setStealStatus = function(text, ok)
    local stamp = os.clock()
    statusStamp = stamp
    PBarTitle.Text = text
    PBarTitle.TextColor3 = (ok == true and Color3.fromRGB(110, 235, 160))
        or (ok == false and Color3.fromRGB(255, 110, 130))
        or T.TEXT
    if ok ~= nil then
        task.delay(4, function()
            if statusStamp == stamp and PBarTitle.Parent then
                PBarTitle.Text = "SEMI TP"
                PBarTitle.TextColor3 = T.TEXT
            end
        end)
    end
end

local PBarTrack = Instance.new("Frame")
PBarTrack.Size = UDim2.new(1, -10, 0, 18)
PBarTrack.Position = UDim2.fromOffset(5, 18)
PBarTrack.BackgroundColor3 = T.TRACK
PBarTrack.BorderSizePixel = 0
PBarTrack.ZIndex = 72
PBarTrack.Parent = PBarMain
corner(PBarTrack, 8)
stroke(PBarTrack, T.STROKE, 1, 0.55)

local PBarInner = Instance.new("Frame")
PBarInner.Size = UDim2.new(1, -2, 1, -2)
PBarInner.Position = UDim2.fromOffset(1, 1)
PBarInner.BackgroundColor3 = T.TRACK2
PBarInner.BackgroundTransparency = 0.15
PBarInner.BorderSizePixel = 0
PBarInner.ZIndex = 72
PBarInner.Parent = PBarTrack
corner(PBarInner, 7)

progressFill = Instance.new("Frame")
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = T.FILL1
progressFill.BorderSizePixel = 0
progressFill.ZIndex = 73
progressFill.Parent = PBarTrack
corner(progressFill, 8)
gradient(progressFill, T.FILL1, T.FILL2)
stroke(progressFill, Color3.fromRGB(205, 160, 255), 1, 0.45)

percentLabel = Instance.new("TextLabel")
percentLabel.Size = UDim2.new(1, 0, 1, 0)
percentLabel.BackgroundTransparency = 1
percentLabel.Font = Enum.Font.GothamBold
percentLabel.TextSize = 12
percentLabel.TextColor3 = T.TEXT
percentLabel.TextStrokeTransparency = 0.7
percentLabel.Text = "0%"
percentLabel.ZIndex = 74
percentLabel.Parent = PBarTrack

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "iCollectPro_SemiTP"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = safeGuiTarget

local panelWidth = IsMobile and 276 or 262

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, panelWidth, 0, 0)
MainFrame.AutomaticSize = Enum.AutomaticSize.Y
MainFrame.AnchorPoint = Vector2.new(1, 0)
MainFrame.Position = IsMobile and UDim2.new(0.88, 0, 0.05, 0) or UDim2.new(0.84, 0, 0.04, 0)
MainFrame.BackgroundColor3 = T.BG
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ZIndex = 100
MainFrame.Parent = ScreenGui
corner(MainFrame, 18)
stroke(MainFrame, T.STROKE, 1.2, 0.4)
addShadow(MainFrame)

local MainPad = Instance.new("UIPadding")
MainPad.PaddingBottom = UDim.new(0, 10)
MainPad.Parent = MainFrame

local HeaderFrame = Instance.new("Frame")
HeaderFrame.Size = UDim2.new(1, 0, 0, 36)
HeaderFrame.BackgroundTransparency = 1
HeaderFrame.ZIndex = 101
HeaderFrame.Parent = MainFrame
restorePos(MainFrame, "main")
makeDraggable(HeaderFrame, MainFrame, rememberPos(MainFrame, "main"))

local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(1, -28, 0, 22)
TitleText.Position = UDim2.new(0, 14, 0, 7)
TitleText.BackgroundTransparency = 1
TitleText.Text = "SEMI TP"
TitleText.Font = Enum.Font.GothamBlack
TitleText.TextSize = 18
TitleText.TextColor3 = T.TEXT
TitleText.ZIndex = 102
TitleText.Parent = HeaderFrame

local TitleRule = Instance.new("Frame")
TitleRule.AnchorPoint = Vector2.new(0.5, 0)
TitleRule.Position = UDim2.new(0.5, 0, 0, 32)
TitleRule.Size = UDim2.new(0, 124, 0, 1)
TitleRule.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
TitleRule.BackgroundTransparency = 0.15
TitleRule.BorderSizePixel = 0
TitleRule.ZIndex = 101
TitleRule.Parent = MainFrame

local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(1, -20, 0, 28)
TabContainer.Position = UDim2.fromOffset(10, 42)
TabContainer.BackgroundColor3 = T.SURF
TabContainer.BorderSizePixel = 0
TabContainer.ZIndex = 101
TabContainer.Parent = MainFrame
corner(TabContainer, 10)
stroke(TabContainer, T.STROKE, 1, 0.48)

local TabPad = Instance.new("UIPadding")
TabPad.PaddingLeft, TabPad.PaddingRight = UDim.new(0, 3), UDim.new(0, 3)
TabPad.PaddingTop, TabPad.PaddingBottom = UDim.new(0, 3), UDim.new(0, 3)
TabPad.Parent = TabContainer

local TabList = Instance.new("UIListLayout", TabContainer)
TabList.FillDirection = Enum.FillDirection.Horizontal
TabList.SortOrder = Enum.SortOrder.LayoutOrder
TabList.Padding = UDim.new(0, 4)

local function createTabButton(text, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1 / 6, -4, 1, 0)
    btn.BackgroundTransparency = 1
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Text = ""
    btn.LayoutOrder = order
    btn.ZIndex = 103
    btn.Parent = TabContainer

    local pill = Instance.new("Frame")
    pill.Name = "Pill"
    pill.Size = UDim2.new(1, 0, 1, 0)
    pill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    pill.BackgroundTransparency = 1
    pill.BorderSizePixel = 0
    pill.ZIndex = 103
    pill.Parent = btn
    corner(pill, 8)
    gradient(pill, T.FILL1, T.ACCENT2, 90)

    local label = Instance.new("TextLabel")
    label.Name = "Label"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = T.DIM
    label.Font = Enum.Font.GothamBold
    label.TextSize = 10
    iCollectProUI.fit(label, 10)
    label.ZIndex = 104
    label.Parent = btn
    return btn
end

-- six tabs by category so no page scrolls: Main / Steal / Anti / Visual / Move / Misc
local MainTabBtn = createTabButton("Main", 1)
local UtilsTabBtn = createTabButton("Steal", 2)
-- kept on an existing table: the file sits at Luau's 200 top-level locals limit
iCollectProUI.tabBtns = { Main = MainTabBtn, Steal = UtilsTabBtn, Protect = createTabButton("Anti", 3),
    Extras = createTabButton("Visual", 4), Move = createTabButton("Move", 5), Misc = createTabButton("Misc", 6) }

local ContentCard = Instance.new("Frame")
ContentCard.AutomaticSize = Enum.AutomaticSize.Y
ContentCard.Size = UDim2.new(1, -20, 0, 0)
ContentCard.Position = UDim2.fromOffset(10, 78)
ContentCard.BackgroundColor3 = T.SURF
ContentCard.BorderSizePixel = 0
ContentCard.ZIndex = 101
ContentCard.Parent = MainFrame
corner(ContentCard, 16)
stroke(ContentCard, T.STROKE, 1, 0.48)

local ContentLayout = Instance.new("Frame")
ContentLayout.AutomaticSize = Enum.AutomaticSize.Y
ContentLayout.Size = UDim2.new(1, -8, 0, 0)
ContentLayout.Position = UDim2.fromOffset(4, 4)
ContentLayout.BackgroundTransparency = 1
ContentLayout.ZIndex = 102
ContentLayout.Parent = ContentCard

local ContentPad = Instance.new("UIPadding")
ContentPad.PaddingBottom = UDim.new(0, 8)
ContentPad.Parent = ContentCard

local List = Instance.new("UIListLayout", ContentLayout)
List.SortOrder = Enum.SortOrder.LayoutOrder
List.Padding = UDim.new(0, 6)

local function createRowFrame(height, order)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, 0, 0, math.max(height, 30))
    card.BackgroundColor3 = T.SURF2
    card.BackgroundTransparency = 0.02
    card.BorderSizePixel = 0
    card.LayoutOrder = order or 1
    card.ZIndex = 103
    card.Parent = ContentLayout
    corner(card, 11)
    local rowStroke = stroke(card, T.STROKE, 1, 0.52)
    addHover(card, T.SURF2, T.HOVER, rowStroke)
    return card
end

local function rowLabel(parent, text, widthScale)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(widthScale or 0.55, 0, 1, 0)
    lbl.Position = UDim2.fromOffset(12, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = T.TEXT
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    iCollectProUI.fit(lbl, 12)
    lbl.ZIndex = 104
    lbl.Parent = parent
    return lbl
end

local function pillButton(parent, width, text)
    local btn = Instance.new("TextButton")
    btn.AutoButtonColor = false
    btn.Size = UDim2.fromOffset(width, 22)
    btn.Position = UDim2.new(1, -(width + 10), 0.5, -11)
    btn.BackgroundColor3 = T.OFF_BG
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = T.TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    iCollectProUI.fit(btn, 11)
    btn.ZIndex = 104
    btn.Parent = parent
    corner(btn, 7)
    local s = stroke(btn, T.STROKE, 1, 0.55)
    btn.MouseEnter:Connect(function() tween(s, 0.14, { Transparency = 0.12 }) end)
    btn.MouseLeave:Connect(function() tween(s, 0.14, { Transparency = 0.55 }) end)
    return btn, s
end

local function createModernToggle(parentCard, textLabel, initialState, onClick)
    rowLabel(parentCard, textLabel, 0.6)

    local btn = Instance.new("TextButton")
    btn.AutoButtonColor = false
    btn.Size = UDim2.fromOffset(62, 22)
    btn.Position = UDim2.new(1, -70, 0.5, -11)
    btn.BackgroundColor3 = T.OFF_BG
    btn.BorderSizePixel = 0
    btn.Text = ""
    btn.ZIndex = 104
    btn.Parent = parentCard
    corner(btn, 7)
    local btnStroke = stroke(btn, T.STROKE, 1, 0.55)

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(1, 0, 1, 0)
    knob.BackgroundTransparency = 1
    knob.BorderSizePixel = 0
    knob.ZIndex = 104
    knob.Parent = btn
    corner(knob, 7)
    gradient(knob, T.GREEN1, T.GREEN2)

    local stateLabel = Instance.new("TextLabel")
    stateLabel.BackgroundTransparency = 1
    stateLabel.Size = UDim2.fromScale(1, 1)
    stateLabel.Font = Enum.Font.GothamBold
    stateLabel.TextSize = 11
    stateLabel.ZIndex = 105
    stateLabel.Parent = btn

    local state = initialState and true or false
    local function paint()
        if state then
            btn.BackgroundColor3 = T.GREEN1
            knob.BackgroundTransparency = 0
            stateLabel.Text = "ON"
            stateLabel.TextColor3 = T.ON_TEXT
            btnStroke.Color = T.GREEN_STROKE
            btnStroke.Transparency = 0.22
        else
            btn.BackgroundColor3 = T.OFF_BG
            knob.BackgroundTransparency = 1
            stateLabel.Text = "OFF"
            stateLabel.TextColor3 = T.OFF_TEXT
            btnStroke.Color = T.STROKE
            btnStroke.Transparency = 0.55
        end
    end
    paint()

    btn.MouseButton1Click:Connect(function()
        state = not state
        paint()
        onClick(state)
    end)
    return btn, paint
end

local function actionButton(parentCard, text, primary)
    local bg
    if primary then
        bg = Instance.new("Frame")
        bg.Size = UDim2.new(1, 0, 1, 0)
        bg.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        bg.BorderSizePixel = 0
        bg.ZIndex = 104
        bg.Parent = parentCard
        corner(bg, 11)
        gradient(bg, T.FILL1, T.ACCENT2, 90)
        stroke(bg, Color3.fromRGB(205, 160, 255), 1, 0.45)
    end
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.AutoButtonColor = false
    btn.BackgroundTransparency = 1
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextStrokeTransparency = primary and 0.6 or 1
    btn.Font = primary and Enum.Font.GothamBlack or Enum.Font.GothamBold
    btn.TextSize = primary and 14 or 12
    btn.ZIndex = 105
    btn.Parent = parentCard
    return btn, bg
end

-- Row builders for the settings tabs, kept on iCollectProUI (the main chunk is out of local registers).
-- numPair: one row card with two "LABEL [value]" boxes. cycleRow: "Label  < option >". textRow: a text box.
;(function()
    -- Configs: every chip registers a setter by its config key, every box/cycle/text row a refresher,
    -- so loading a profile can repaint the whole menu without re-running the script
    iCollectProUI.cfgChips, iCollectProUI.cfgRefresh = {}, {}
    local function box(row, right, spec)
        local holder = Instance.new("Frame")
        -- right == nil: the box has the row to itself and spans it
        holder.Size = right == nil and UDim2.new(1, -16, 0, 22) or UDim2.new(0.5, -12, 0, 22)
        holder.Position = right and UDim2.new(0.5, 4, 0.5, -11) or UDim2.new(0, 8, 0.5, -11)
        holder.BackgroundColor3 = T.OFF_BG
        holder.BorderSizePixel = 0
        holder.ZIndex = 104
        holder.Parent = row
        corner(holder, 7)
        stroke(holder, T.STROKE, 1, 0.55)
        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Size = UDim2.new(1, -44, 1, 0)
        l.Position = UDim2.fromOffset(6, 0)
        l.Font = Enum.Font.GothamBold
        l.TextSize = 10
        l.TextXAlignment = Enum.TextXAlignment.Left
        iCollectProUI.fit(l, 10)
        l.TextColor3 = T.OFF_TEXT
        l.Text = string.upper(spec[1])
        l.ZIndex = 105
        l.Parent = holder
        local tb = Instance.new("TextBox")
        tb.Size = UDim2.new(0, 36, 1, -6)
        tb.Position = UDim2.new(1, -39, 0, 3)
        tb.BackgroundColor3 = T.TRACK
        tb.BorderSizePixel = 0
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 10
        tb.TextColor3 = T.TEXT
        tb.ClearTextOnFocus = false
        tb.ZIndex = 106
        tb.Parent = holder
        corner(tb, 5)
        local key, def, lo, hi = spec[2], spec[3], spec[4], spec[5]
        local function get() return math.clamp(tonumber(HubConfig[key]) or def, lo, hi) end
        tb.Text = tostring(get())
        iCollectProUI.cfgRefresh[#iCollectProUI.cfgRefresh + 1] = function() tb.Text = tostring(get()) end
        tb.FocusLost:Connect(function()
            local n = tonumber(tb.Text)
            if n then
                HubConfig[key] = math.clamp(math.floor(n * 10 + 0.5) / 10, lo, hi)
                saveHubConfig()
            end
            tb.Text = tostring(get())
        end)
    end

    function iCollectProUI.num(key, def, lo, hi)
        return math.clamp(tonumber(HubConfig[key]) or def, lo, hi)
    end

    function iCollectProUI.numPair(order, a, b)
        local r = createRowFrame(30, order)
        -- (not "b and false or nil": false-or-nil is nil, which stretched the left box under the right one)
        if b then
            box(r, false, a)
            box(r, true, b)
        else
            box(r, nil, a)
        end
        return r
    end

    function iCollectProUI.cycleRow(order, text, options, key, onSet)
        local r = createRowFrame(30, order)
        rowLabel(r, text, 0.4)
        local holder = Instance.new("Frame")
        holder.Size = UDim2.fromOffset(120, 22)
        holder.Position = UDim2.new(1, -128, 0.5, -11)
        holder.BackgroundTransparency = 1
        holder.ZIndex = 104
        holder.Parent = r
        local val = Instance.new("TextLabel")
        val.Size = UDim2.new(1, -52, 1, 0)
        val.Position = UDim2.fromOffset(26, 0)
        val.BackgroundColor3 = T.GREEN1
        val.BorderSizePixel = 0
        val.Font = Enum.Font.GothamBold
        val.TextSize = 11
        val.TextColor3 = T.ON_TEXT
        val.ZIndex = 105
        val.Parent = holder
        iCollectProUI.fit(val, 11, 6)
        corner(val, 7)
        stroke(val, T.GREEN_STROKE, 1, 0.22)
        local idx = table.find(options, HubConfig[key]) or 1
        local function show()
            val.Text = string.upper(options[idx])
            HubConfig[key] = options[idx]
            if onSet then onSet(options[idx]) end
        end
        show()
        iCollectProUI.cfgRefresh[#iCollectProUI.cfgRefresh + 1] = function()
            idx = table.find(options, HubConfig[key]) or idx
            show()
        end
        local function step(d)
            idx = (idx - 1 + d) % #options + 1
            show()
            saveHubConfig()
        end
        for i, spec in ipairs({ { "<", 0, 0, -1 }, { ">", 1, -22, 1 } }) do
            local b = Instance.new("TextButton")
            b.Size = UDim2.fromOffset(22, 22)
            b.Position = UDim2.new(spec[2], spec[3], 0, 0)
            b.AutoButtonColor = false
            b.BackgroundColor3 = T.OFF_BG
            b.BorderSizePixel = 0
            b.Text = spec[1]
            b.TextColor3 = T.TEXT
            b.Font = Enum.Font.GothamBlack
            b.TextSize = 12
            b.ZIndex = 105
            b.Parent = holder
            corner(b, 7)
            stroke(b, T.STROKE, 1, 0.55)
            addHover(b, T.OFF_BG, T.HOVER)
            b.MouseButton1Click:Connect(function() step(spec[4]) end)
        end
        return r
    end

    function iCollectProUI.textRow(order, text, key, hint)
        local r = createRowFrame(30, order)
        rowLabel(r, text, 0.3)
        local tb = Instance.new("TextBox")
        tb.Size = UDim2.new(0.62, -8, 0, 22)
        tb.Position = UDim2.new(0.38, 0, 0.5, -11)
        tb.BackgroundColor3 = T.OFF_BG
        tb.BorderSizePixel = 0
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 10
        tb.TextColor3 = T.TEXT
        tb.PlaceholderText = hint or ""
        tb.PlaceholderColor3 = T.OFF_TEXT
        tb.TextTruncate = Enum.TextTruncate.AtEnd
        tb.ClearTextOnFocus = false
        tb.Text = tostring(HubConfig[key] or "")
        tb.ZIndex = 105
        tb.Parent = r
        corner(tb, 7)
        stroke(tb, T.STROKE, 1, 0.55)
        tb.FocusLost:Connect(function()
            HubConfig[key] = tb.Text
            saveHubConfig()
        end)
        iCollectProUI.cfgRefresh[#iCollectProUI.cfgRefresh + 1] = function() tb.Text = tostring(HubConfig[key] or "") end
        return r
    end

    -- slider: "LABEL  [=====o-----]  value", drag or click anywhere on the track
    function iCollectProUI.slider(order, text, key, def, lo, hi)
        local r = createRowFrame(30, order)
        local lbl = rowLabel(r, string.upper(text), 0.2)
        -- same small dim caps as the number boxes
        iCollectProUI.fit(lbl, 10)
        lbl.TextColor3 = T.OFF_TEXT
        local val = Instance.new("TextLabel")
        val.Size = UDim2.fromOffset(34, 22)
        val.Position = UDim2.new(1, -42, 0.5, -11)
        val.BackgroundColor3 = T.TRACK
        val.BorderSizePixel = 0
        val.Font = Enum.Font.GothamBold
        val.TextSize = 11
        val.TextColor3 = T.TEXT
        val.ZIndex = 105
        val.Parent = r
        corner(val, 6)
        local track = Instance.new("TextButton")
        track.AutoButtonColor = false
        track.Text = ""
        track:SetAttribute("__semiNoPop", true)
        track.Size = UDim2.new(0.8, -62, 0, 8)
        track.Position = UDim2.new(0.2, 8, 0.5, -4)
        track.BackgroundColor3 = T.TRACK
        track.BorderSizePixel = 0
        track.ZIndex = 105
        track.Parent = r
        corner(track, 4)
        stroke(track, T.STROKE, 1, 0.55)
        local fill = Instance.new("Frame")
        fill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        fill.BorderSizePixel = 0
        fill.ZIndex = 106
        fill.Parent = track
        corner(fill, 4)
        gradient(fill, T.GREEN1, T.GREEN2)
        local knob = Instance.new("Frame")
        knob.AnchorPoint = Vector2.new(0.5, 0.5)
        knob.Size = UDim2.fromOffset(14, 14)
        knob.BackgroundColor3 = T.ON_TEXT
        knob.BorderSizePixel = 0
        knob.ZIndex = 107
        knob.Parent = track
        corner(knob, 7)
        stroke(knob, T.GREEN_STROKE, 1.5, 0)
        local function get() return math.clamp(tonumber(HubConfig[key]) or def, lo, hi) end
        local function show()
            local a = (get() - lo) / (hi - lo)
            fill.Size = UDim2.new(a, 0, 1, 0)
            knob.Position = UDim2.new(a, 0, 0.5, 0)
            val.Text = tostring(math.floor(get() + 0.5))
        end
        local function setFromX(x)
            local a = math.clamp((x - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
            HubConfig[key] = math.floor(lo + a * (hi - lo) + 0.5)
            show()
        end
        local dragging = false
        track.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                setFromX(i.Position.X)
            end
        end)
        UserInputService.InputChanged:Connect(function(i)
            if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                setFromX(i.Position.X)
            end
        end)
        UserInputService.InputEnded:Connect(function(i)
            if dragging and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then
                dragging = false
                saveHubConfig()
            end
        end)
        show()
        iCollectProUI.cfgRefresh[#iCollectProUI.cfgRefresh + 1] = show
        return r
    end
end)()

-- Configs (Misc tab): named profiles saved as files, plus share codes you can copy and paste to a
-- friend (the code is the profile itself, base64 - no website involved). Window positions and the
-- podium stay yours when a profile loads.
;(function()
    local DIR = "iCollectPro_SemiTP_Configs"
    -- share codes read iCollectPro-...; codes made before still import (SEMITP-...)
    local PREFIX = "iCollectPro-"
    local SKIP = { positions = true, speedPos = true, selectedSlot = true, cfgActive = true }
    local B64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

    local function b64enc(data)
        local out = {}
        for i = 1, #data, 3 do
            local a, b, c = data:byte(i, i + 2)
            local n = a * 65536 + (b or 0) * 256 + (c or 0)
            for j = 1, 4 do
                if j == 3 and not b or j == 4 and not c then
                    out[#out + 1] = "="
                else
                    local k = math.floor(n / 64 ^ (4 - j)) % 64
                    out[#out + 1] = B64:sub(k + 1, k + 1)
                end
            end
        end
        return table.concat(out)
    end

    local function b64dec(data)
        data = data:gsub("[^%w%+/]", "")
        local out, n, bits = {}, 0, 0
        for i = 1, #data do
            n = n * 64 + (B64:find(data:sub(i, i), 1, true) - 1)
            bits = bits + 6
            if bits >= 8 then
                bits = bits - 8
                out[#out + 1] = string.char(math.floor(n / 2 ^ bits) % 256)
                n = n % 2 ^ bits
            end
        end
        return table.concat(out)
    end

    local function snapshot()
        local t = {}
        for k, v in pairs(HubConfig) do
            if not SKIP[k] then t[k] = v end
        end
        return t
    end

    local function clean(name)
        name = tostring(name or ""):gsub("[^%w%s_%-]", ""):match("^%s*(.-)%s*$")
        return name:sub(1, 24)
    end

    local function path(name) return DIR .. "/" .. name .. ".json" end

    local function list()
        local names = {}
        pcall(function()
            if not isfolder(DIR) then makefolder(DIR) end
            for _, f in ipairs(listfiles(DIR)) do
                local n = tostring(f):match("([^/\\]+)%.json$")
                if n then names[#names + 1] = n end
            end
        end)
        table.sort(names, function(a, b) return a:lower() < b:lower() end)
        return names
    end

    -- put a profile's values into the live menu: chips flip (and run their effect), boxes refresh
    local function apply(cfg)
        if type(cfg) ~= "table" then return false end
        for k, v in pairs(cfg) do
            if not SKIP[k] then HubConfig[k] = v end
        end
        for key, set in pairs(iCollectProUI.cfgChips) do
            if cfg[key] ~= nil then pcall(set, cfg[key]) end
        end
        for _, f in ipairs(iCollectProUI.cfgRefresh) do pcall(f) end
        saveHubConfig()
        return true
    end

    local function actionBtn(row, right, text, fn)
        local b = Instance.new("TextButton")
        b.AutoButtonColor = false
        b.Size = UDim2.new(0.5, -12, 0, 22)
        b.Position = right and UDim2.new(0.5, 4, 0.5, -11) or UDim2.new(0, 8, 0.5, -11)
        b.BackgroundColor3 = T.OFF_BG
        b.BorderSizePixel = 0
        b.Font = Enum.Font.GothamBold
        b.TextSize = 10
        b.TextColor3 = T.TEXT
        b.Text = text
        b.ZIndex = 104
        b.Parent = row
        corner(b, 7)
        local s = stroke(b, T.STROKE, 1, 0.55)
        addHover(b, T.OFF_BG, T.HOVER)
        b.MouseButton1Click:Connect(function()
            b.BackgroundColor3 = T.GREEN1
            s.Color = T.GREEN_STROKE
            task.delay(0.25, function()
                b.BackgroundColor3 = T.OFF_BG
                s.Color = T.STROKE
            end)
            local ok, err = pcall(fn)
            if not ok then setStealStatus("CONFIG ERROR", false) warn("[SEMI TP] config:", err) end
        end)
        return b
    end

    local function textBox(parent, hint)
        local tb = Instance.new("TextBox")
        tb.BackgroundColor3 = T.OFF_BG
        tb.BorderSizePixel = 0
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 10
        tb.TextColor3 = T.TEXT
        tb.PlaceholderText = hint
        tb.PlaceholderColor3 = T.OFF_TEXT
        tb.TextTruncate = Enum.TextTruncate.AtEnd
        tb.ClearTextOnFocus = false
        tb.Text = ""
        tb.ZIndex = 105
        tb.Parent = parent
        corner(tb, 7)
        stroke(tb, T.STROKE, 1, 0.55)
        return tb
    end

    function iCollectProUI.configRows(order)
        local rows = {}
        -- Profile  < name >   (arrows step through saved profiles; type a new name to save a new one)
        local r1 = createRowFrame(30, order)
        rowLabel(r1, "Profile", 0.3)
        local holder = Instance.new("Frame")
        holder.Size = UDim2.new(0.66, -8, 0, 22)
        holder.Position = UDim2.new(0.34, 0, 0.5, -11)
        holder.BackgroundTransparency = 1
        holder.ZIndex = 104
        holder.Parent = r1
        local nameBox = textBox(holder, "profile name")
        nameBox.Size = UDim2.new(1, -52, 1, 0)
        nameBox.Position = UDim2.fromOffset(26, 0)
        nameBox.TextXAlignment = Enum.TextXAlignment.Center
        nameBox.Text = tostring(HubConfig.cfgActive or "")
        local selected = clean(nameBox.Text)
        local function step(d)
            local names = list()
            if #names == 0 then setStealStatus("NO SAVED PROFILES", false) return end
            local i = table.find(names, selected) or (d > 0 and 0 or 1)
            i = (i - 1 + d) % #names + 1
            selected = names[i]
            nameBox.Text = selected
        end
        for _, spec in ipairs({ { "<", 0, 0, -1 }, { ">", 1, -22, 1 } }) do
            local b = Instance.new("TextButton")
            b.Size = UDim2.fromOffset(22, 22)
            b.Position = UDim2.new(spec[2], spec[3], 0, 0)
            b.AutoButtonColor = false
            b.BackgroundColor3 = T.OFF_BG
            b.BorderSizePixel = 0
            b.Text = spec[1]
            b.TextColor3 = T.TEXT
            b.Font = Enum.Font.GothamBlack
            b.TextSize = 12
            b.ZIndex = 105
            b.Parent = holder
            corner(b, 7)
            stroke(b, T.STROKE, 1, 0.55)
            addHover(b, T.OFF_BG, T.HOVER)
            b.MouseButton1Click:Connect(function() step(spec[4]) end)
        end
        rows[1] = r1

        local function typed()
            local n = clean(nameBox.Text)
            if n == "" then setStealStatus("TYPE A PROFILE NAME", false) return nil end
            return n
        end
        local function exists(n) return table.find(list(), n) ~= nil end

        local r2 = createRowFrame(30, order + 1)
        actionBtn(r2, false, "SAVE", function()
            local n = typed()
            if not n then return end
            if not isfolder(DIR) then makefolder(DIR) end
            HubConfig.cfgActive = n
            writefile(path(n), HttpService:JSONEncode(snapshot()))
            saveHubConfig()
            selected = n
            setStealStatus("SAVED PROFILE: " .. n, true)
        end)
        actionBtn(r2, true, "LOAD", function()
            local n = typed()
            if not n then return end
            if not exists(n) then setStealStatus("NO PROFILE: " .. n, false) return end
            local cfg = HttpService:JSONDecode(readfile(path(n)))
            HubConfig.cfgActive = n
            selected = n
            apply(cfg)
            setStealStatus("LOADED PROFILE: " .. n, true)
        end)
        rows[2] = r2

        local r3 = createRowFrame(30, order + 2)
        actionBtn(r3, false, "RENAME", function()
            local n = typed()
            if not n then return end
            if not selected or selected == "" or not exists(selected) then setStealStatus("PICK A PROFILE WITH < >", false) return end
            if n == selected then return end
            writefile(path(n), readfile(path(selected)))
            delfile(path(selected))
            if HubConfig.cfgActive == selected then HubConfig.cfgActive = n saveHubConfig() end
            setStealStatus("RENAMED " .. selected .. " TO " .. n, true)
            selected = n
        end)
        actionBtn(r3, true, "DELETE", function()
            local n = typed()
            if not n then return end
            if not exists(n) then setStealStatus("NO PROFILE: " .. n, false) return end
            delfile(path(n))
            if HubConfig.cfgActive == n then HubConfig.cfgActive = nil saveHubConfig() end
            selected = nil
            nameBox.Text = ""
            setStealStatus("DELETED PROFILE: " .. n, true)
        end)
        rows[3] = r3

        -- share codes: COPY puts the current settings on your clipboard; paste one below and IMPORT
        local r5 = createRowFrame(30, order + 4)
        local codeBox = textBox(r5, "paste a share code here")
        codeBox.Size = UDim2.new(1, -16, 0, 22)
        codeBox.Position = UDim2.new(0, 8, 0.5, -11)
        -- the real code is long; the box keeps it aside and just shows iCollectPro-********
        local realCode = ""
        local MASK = PREFIX .. "********"
        local function showCode(code)
            realCode = code
            codeBox.Text = code ~= "" and MASK or ""
        end
        codeBox.Focused:Connect(function()
            if codeBox.Text == MASK then codeBox.Text = "" end
        end)
        codeBox.FocusLost:Connect(function()
            local raw = codeBox.Text:gsub("%s", "")
            if raw == "" then
                showCode(realCode)
            else
                showCode(raw)
            end
        end)

        local r4 = createRowFrame(30, order + 3)
        actionBtn(r4, false, "COPY CODE", function()
            local code = PREFIX .. b64enc(HttpService:JSONEncode({ name = clean(nameBox.Text), data = snapshot() }))
            if setclipboard then
                setclipboard(code)
                setStealStatus("SHARE CODE COPIED", true)
            else
                setStealStatus("CODE IN THE BOX (NO CLIPBOARD)", true)
            end
            showCode(code)
        end)
        actionBtn(r4, true, "IMPORT CODE", function()
            local raw = realCode
            local body
            for _, pre in ipairs({ PREFIX, "SEMITP-" }) do
                if raw:sub(1, #pre) == pre then body = raw:sub(#pre + 1) break end
            end
            if not body then setStealStatus("NOT AN iCollectPro CODE", false) return end
            local ok, pack = pcall(function() return HttpService:JSONDecode(b64dec(body)) end)
            if not ok or type(pack) ~= "table" or type(pack.data) ~= "table" then setStealStatus("BROKEN CODE", false) return end
            apply(pack.data)
            showCode("")
            local n = clean(pack.name)
            if n ~= "" then nameBox.Text = n end
            setStealStatus("IMPORTED" .. (n ~= "" and (": " .. n) or "") .. " (SAVE TO KEEP)", true)
        end)
        rows[4] = r4
        rows[5] = r5
        return rows
    end
end)()

local Row1 = createRowFrame(30, 1)
iCollectProUI.podiumLbl = rowLabel(Row1, "Podium (1-" .. MAX_PODIUM .. ")", 0.45)

local SelectorFrame = Instance.new("Frame")
SelectorFrame.Size = UDim2.fromOffset(100, 22)
SelectorFrame.Position = UDim2.new(1, -110, 0.5, -11)
SelectorFrame.BackgroundTransparency = 1
SelectorFrame.ZIndex = 104
SelectorFrame.Parent = Row1

local function arrowButton(text, xScale, xOffset, parent)
    local b = Instance.new("TextButton")
    b.Size = UDim2.fromOffset(22, 22)
    b.Position = UDim2.new(xScale, xOffset, 0, 0)
    b.AutoButtonColor = false
    b.BackgroundColor3 = T.OFF_BG
    b.BorderSizePixel = 0
    b.Text = text
    b.TextColor3 = T.TEXT
    b.Font = Enum.Font.GothamBlack
    b.TextSize = 12
    b.ZIndex = 105
    b.Parent = parent or SelectorFrame
    corner(b, 7)
    stroke(b, T.STROKE, 1, 0.55)
    addHover(b, T.OFF_BG, T.HOVER)
    return b
end

local LeftBtn = arrowButton("<", 0, 0)
local RightBtn = arrowButton(">", 1, -22)

local SlotDisplay = Instance.new("TextBox")
SlotDisplay.Size = UDim2.new(1, -52, 1, 0)
SlotDisplay.Position = UDim2.fromOffset(26, 0)
SlotDisplay.BackgroundColor3 = T.GREEN1
SlotDisplay.BorderSizePixel = 0
SlotDisplay.Text = tostring(selectedSlot)
SlotDisplay.TextColor3 = T.ON_TEXT
SlotDisplay.Font = Enum.Font.GothamBold
SlotDisplay.TextSize = 12
SlotDisplay.ClearTextOnFocus = true
SlotDisplay.ZIndex = 105
SlotDisplay.Parent = SelectorFrame
corner(SlotDisplay, 7)
stroke(SlotDisplay, T.GREEN_STROKE, 1, 0.22)

local function updateSlot(newSlot)
    selectedSlot = newSlot
    SlotDisplay.Text = tostring(selectedSlot)
    iCollectPro.setSlot(selectedSlot)
end

local slotStroke = SlotDisplay:FindFirstChildOfClass("UIStroke")
task.spawn(function()
    local was = nil
    while iCollectPro_ALIVE() and SlotDisplay.Parent do
        -- the label follows the target base's podium count; a slot past it drops back to the last one
        local max = iCollectPro.maxPodium()
        local text = "Podium (1-" .. max .. ")"
        if iCollectProUI.podiumLbl.Text ~= text then iCollectProUI.podiumLbl.Text = text end
        if selectedSlot > max then updateSlot(max) end
        if slotHasTarget ~= was then
            was = slotHasTarget
            SlotDisplay.BackgroundColor3 = was and T.GREEN1 or Color3.fromRGB(120, 30, 50)
            SlotDisplay.TextColor3 = was and T.ON_TEXT or Color3.fromRGB(255, 200, 210)
            if slotStroke then slotStroke.Color = was and T.GREEN_STROKE or Color3.fromRGB(220, 80, 110) end
        end
        task.wait(0.25)
    end
end)

SlotDisplay.FocusLost:Connect(function()
    local n = math.floor(tonumber(SlotDisplay.Text) or selectedSlot)
    updateSlot(math.clamp(n, 1, iCollectPro.maxPodium()))
end)

LeftBtn.MouseButton1Click:Connect(function()
    local nextSlot = selectedSlot - 1
    if nextSlot < 1 then nextSlot = iCollectPro.maxPodium() end
    updateSlot(nextSlot)
end)

RightBtn.MouseButton1Click:Connect(function()
    local nextSlot = selectedSlot + 1
    if nextSlot > iCollectPro.maxPodium() then nextSlot = 1 end
    updateSlot(nextSlot)
end)

Row1.LayoutOrder = 0

local RowBase = createRowFrame(30, -1)
rowLabel(RowBase, "Target", 0.3)

local BaseSelector = Instance.new("Frame")
BaseSelector.Size = UDim2.fromOffset(136, 22)
BaseSelector.Position = UDim2.new(1, -144, 0.5, -11)
BaseSelector.BackgroundTransparency = 1
BaseSelector.ZIndex = 104
BaseSelector.Parent = RowBase

local BaseLeft = arrowButton("<", 0, 0, BaseSelector)
local BaseRight = arrowButton(">", 1, -22, BaseSelector)

local BaseName = Instance.new("TextLabel")
BaseName.Size = UDim2.new(1, -52, 1, 0)
BaseName.Position = UDim2.fromOffset(26, 0)
BaseName.BackgroundColor3 = T.OFF_BG
BaseName.BorderSizePixel = 0
BaseName.Text = "Nearest"
BaseName.TextColor3 = T.TEXT
BaseName.Font = Enum.Font.GothamBold
BaseName.TextSize = 11
iCollectProUI.fit(BaseName, 11)
BaseName.ZIndex = 105
BaseName.Parent = BaseSelector
corner(BaseName, 7)
stroke(BaseName, T.STROKE, 1, 0.55)

local function plotOwnerName(plot)
    local sign = plot and plot:FindFirstChild("PlotSign")
    local sg = sign and sign:FindFirstChild("SurfaceGui")
    local lbl = sg and sg:FindFirstChild("Frame") and sg.Frame:FindFirstChild("TextLabel")
    return lbl and (lbl.Text:gsub("'s [Bb]ase$", "")) or "?"
end

local function enemyPlotList()
    local list = {}
    local plots = Workspace:FindFirstChild("Plots")
    for _, plot in ipairs(plots and plots:GetChildren() or {}) do
        if isEnemyPlot(plot) and plot:FindFirstChild("AnimalPodiums") then
            list[#list + 1] = plot
        end
    end
    table.sort(list, function(a, b)
        return (tonumber(a:GetAttribute("Order")) or 0) < (tonumber(b:GetAttribute("Order")) or 0)
    end)
    return list
end

local function refreshBaseName()
    BaseName.Text = targetPlot and plotOwnerName(targetPlot) or "Nearest"
end

local function cycleBase(dir)
    local list = enemyPlotList()
    local idx = 0
    for i, p in ipairs(list) do
        if p == targetPlot then idx = i break end
    end
    idx = (idx + dir) % (#list + 1)
    targetPlot = list[idx]
    refreshBaseName()
end

BaseLeft.MouseButton1Click:Connect(function() cycleBase(-1) end)
BaseRight.MouseButton1Click:Connect(function() cycleBase(1) end)

task.spawn(function()
    while iCollectPro_ALIVE() and BaseName.Parent do
        if targetPlot and (not targetPlot.Parent or not isEnemyPlot(targetPlot)) then
            targetPlot = nil
        end
        refreshBaseName()
        task.wait(0.5)
    end
end)

local InfoRow = createRowFrame(26, 1)
InfoRow.BackgroundTransparency = 0.35
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -16, 1, 0)
InfoLabel.Position = UDim2.fromOffset(8, 0)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Font = Enum.Font.GothamBold
InfoLabel.TextSize = 11
InfoLabel.TextColor3 = T.DIM
iCollectProUI.fit(InfoLabel, 11)
InfoLabel.Text = ""
InfoLabel.ZIndex = 104
InfoLabel.Parent = InfoRow

task.spawn(function()
    local last
    while iCollectPro_ALIVE() and InfoLabel.Parent do
        if podiumInfoText ~= last then
            last = podiumInfoText
            InfoLabel.Text = last
            InfoLabel.TextColor3 = (last == "empty" or last:find("no enemy")) and T.OFF_TEXT or T.TEXT
        end
        task.wait(0.25)
    end
end)

local Row6 = createRowFrame(36, 2)
local StealBg
TeleportBtn, StealBg = actionButton(Row6, "STEAL NOW", true)
-- long states like STEAL NOW (FRIENDS ALLOWED) shrink to fit
iCollectProUI.fit(TeleportBtn, 14, 16)
TeleportBtn.MouseButton1Click:Connect(function() iCollectPro.execute() end)

do
    local flashUntil, flashText = 0, ""
    iCollectProUI.flash = function(msg)
        setStealStatus(msg, false)
        flashText = msg
        flashUntil = os.clock() + 1.2
    end
    task.spawn(function()
        while iCollectPro_ALIVE() and TeleportBtn.Parent do
            local st = iCollectProUI.state
            local flashing = os.clock() < flashUntil
            if flashing then
                TeleportBtn.Text = flashText
                StealBg.BackgroundColor3 = Color3.fromRGB(255, 120, 140)
            elseif st.locked and st.friends then
                TeleportBtn.Text = "STEAL NOW (FRIENDS ALLOWED)"
                StealBg.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            elseif st.locked then
                TeleportBtn.Text = "LOCKED  " .. st.lockText
                StealBg.BackgroundColor3 = Color3.fromRGB(110, 110, 120)
            else
                TeleportBtn.Text = "STEAL NOW"
                StealBg.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            end
            TeleportBtn.TextColor3 = (st.locked and not st.friends and not flashing) and Color3.fromRGB(200, 200, 210) or Color3.fromRGB(255, 255, 255)
            task.wait(0.1)
        end
    end)
end

local Row5 = createRowFrame(30, 3)
local ActivateBtn = actionButton(Row5, "Instant Reset", false)
ActivateBtn.MouseButton1Click:Connect(function() iCollectPro.activate() end)

-- Settings tab, grouped and compact: small section headers + two switch chips per row
local isSpeedVisible = HubConfig.hexSpeedVisible or false
local HexSpeedFrameRef = nil
local SettingsRows = { Steal = {}, Protect = {}, Extras = {}, Move = {}, Misc = {} }
do
    local bucket = SettingsRows.Steal
    local function tab(name) bucket = SettingsRows[name] end
    local function header(text, order)
        local h = Instance.new("TextLabel")
        h.Size = UDim2.new(1, 0, 0, 14)
        h.BackgroundTransparency = 1
        h.Font = Enum.Font.GothamBlack
        h.TextSize = 10
        h.TextColor3 = T.DIM
        h.TextXAlignment = Enum.TextXAlignment.Left
        h.Text = "   " .. text
        h.LayoutOrder = order
        h.ZIndex = 104
        h.Parent = ContentLayout
        bucket[#bucket + 1] = h
    end

    -- same look as the original rows and iCollectPro INSTA STEAL: a rounded row card holding
    -- two pills with the name inside each; the pill turns green when it's on
    local function pairRow(order)
        local r = createRowFrame(30, order)
        bucket[#bucket + 1] = r
        return r
    end

    local function chip(row, right, text, initial, onToggle)
        local btn = Instance.new("TextButton")
        btn.AutoButtonColor = false
        -- right == nil: a chip on its own row spans the full width
        btn.Size = right == nil and UDim2.new(1, -16, 0, 22) or UDim2.new(0.5, -12, 0, 22)
        btn.Position = right and UDim2.new(0.5, 4, 0.5, -11) or UDim2.new(0, 8, 0.5, -11)
        btn.BackgroundColor3 = T.OFF_BG
        btn.BorderSizePixel = 0
        btn.Text = ""
        btn.ZIndex = 104
        btn.Parent = row
        corner(btn, 7)
        local s = stroke(btn, T.STROKE, 1, 0.55)
        local knob = Instance.new("Frame")
        knob.Size = UDim2.fromScale(1, 1)
        knob.BackgroundTransparency = 1
        knob.BorderSizePixel = 0
        knob.ZIndex = 104
        knob.Parent = btn
        corner(knob, 7)
        gradient(knob, T.GREEN1, T.GREEN2)
        local l = Instance.new("TextLabel")
        l.BackgroundTransparency = 1
        l.Size = UDim2.new(1, -10, 1, -8)
        l.Position = UDim2.fromOffset(5, 4)
        l.Font = Enum.Font.GothamBold
        l.TextSize = 10
        l.Text = string.upper(text)
        l.ZIndex = 105
        l.Parent = btn
        -- shrink long names to fit the pill on one line instead of cutting them off with "..."
        iCollectProUI.fit(l, 10)
        local state = initial and true or false
        local cfgKey = iCollectProUI.lastKey
        iCollectProUI.lastKey = nil
        local function paint()
            knob.BackgroundTransparency = state and 0 or 1
            btn.BackgroundColor3 = state and T.GREEN1 or T.OFF_BG
            s.Color = state and T.GREEN_STROKE or T.STROKE
            s.Transparency = state and 0.22 or 0.55
            l.TextColor3 = state and T.ON_TEXT or T.OFF_TEXT
        end
        paint()
        btn.MouseButton1Click:Connect(function()
            state = not state
            paint()
            onToggle(state)
        end)
        if cfgKey then
            iCollectProUI.cfgChips[cfgKey] = function(v)
                v = v and true or false
                if v == state then return end
                state = v
                paint()
                onToggle(state)
            end
        end
    end

    -- a lone setting gets the classic row: name on the left, ON/OFF pill on the right
    local function single(order, text, initial, onToggle)
        local r = createRowFrame(30, order)
        bucket[#bucket + 1] = r
        createModernToggle(r, text, initial, onToggle)
        return r
    end

    local function saved(key, apply)
        -- the chip built right after this call picks the key up for the Configs registry
        iCollectProUI.lastKey = key
        return function(on)
            HubConfig[key] = on
            saveHubConfig()
            if apply then apply(on) end
        end
    end

    -- six tabs, each split into short sections: Steal / Anti / Visual / Move / Misc (+ Main)
    tab("Steal")
    header("STEAL", 10)
    local r = pairRow(11)
    chip(r, false, "Potion + Shield", PotionEnabled, saved("potionEnabled", function(on) PotionEnabled = on end))
    chip(r, true, "Show Path", HubConfig.showPathEnabled ~= false, saved("showPathEnabled"))
    bucket[#bucket + 1] = iCollectProUI.cycleRow(13, "Fly Tool",
        { "Auto", "Flying Carpet", "Cupid's Wings", "Witch's Broom", "Waverider", "Santa's Sleigh" }, "flyTool")
    header("AUTO", 14)
    r = pairRow(15)
    chip(r, false, "TP on Allow", AutoTPOnAllowEnabled, saved("autoTPOnAllowEnabled", function(on) AutoTPOnAllowEnabled = on end))
    chip(r, true, "TP on Unlock", iCollectProFx.unlockTP, saved("autoTPOnUnlockEnabled", function(on) iCollectProFx.unlockTP = on end))
    r = pairRow(16)
    chip(r, false, "Auto Kick at 2s", iCollectProFx.rejoin2, saved("autoRejoinEnabled", function(on) iCollectProFx.rejoin2 = on end))
    chip(r, true, "Kick on Deliver", KickAfterStealEnabled, saved("kickAfterStealEnabled", function(on) KickAfterStealEnabled = on end))
    header("GRAB MODES", 17)
    r = pairRow(18)
    chip(r, false, "Quick Grab", iCollectProFx.quickGrab, saved("quickGrabEnabled", function(on) iCollectProFx.quickGrab = on end))
    chip(r, true, "Public Grab", iCollectProFx.pubGrab, saved("publicGrabEnabled", function(on) iCollectProFx.pubGrab = on end))
    r = pairRow(19)
    chip(r, nil, "Defender Bypass", iCollectProFx.defBypass, saved("defenderBypassEnabled", function(on) iCollectProFx.defBypass = on end))

    tab("Protect")
    header("PROTECTION", 10)
    r = pairRow(11)
    chip(r, false, "Anti Admin", AntiRocketEnabled, saved("antiRocketEnabled", iCollectProAntiRocket.set))
    chip(r, true, "Anti Bee", iCollectProFx.bee, saved("antiBeeEnabled", iCollectProAntiFx.setBee))
    r = pairRow(12)
    chip(r, false, "Anti Gummy", iCollectProFx.gummy, saved("antiGummyEnabled", iCollectProAntiFx.setGummy))
    chip(r, true, "Anti Disco", iCollectProFx.disco, saved("antiDiscoEnabled", iCollectProAntiFx.setDisco))
    r = pairRow(13)
    chip(r, false, "Anti Paintball", iCollectProFx.paint, saved("antiPaintEnabled", iCollectProAntiFx.setPaint))
    chip(r, true, "Anti Web", iCollectProFx.web, saved("antiWebEnabled", iCollectProAntiFx.setWeb))
    r = pairRow(14)
    chip(r, false, "Anti Body Swap", iCollectProFx.swap, saved("antiSwapEnabled", iCollectProAntiFx.setSwap))
    chip(r, true, "Anti Ragdoll", AntiRagdollEnabled, saved("antiRagdollEnabled", iCollectProAntiRagdoll.set))
    header("COMBAT", 15)
    r = pairRow(16)
    chip(r, false, "Destroy Sentry", iCollectProFx.sentry, saved("destroySentryEnabled", iCollectProAntiFx.setSentry))
    chip(r, true, "Destroy Doge", iCollectProFx.doge, saved("destroyDogeEnabled", iCollectProAntiFx.setDoge))
    r = pairRow(17)
    chip(r, false, "Aimbot", iCollectProFx.aim, saved("aimbotEnabled", iCollectProAntiFx.setAim))
    chip(r, true, "Auto Spam", iCollectProFx.spam, saved("autoSpamEnabled", iCollectProAntiFx.setSpam))

    tab("Extras")
    header("PLAYERS", 20)
    r = pairRow(21)
    chip(r, false, "Player ESP", iCollectProFx.esp, saved("playerEspEnabled", iCollectProAntiFx.setEsp))
    chip(r, true, "Tracers", iCollectProFx.tracers, saved("tracersEnabled", function(on) iCollectProFx.tracers = on end))
    r = pairRow(22)
    chip(r, false, "Server Ghost", iCollectProFx.ghost, saved("serverGhostEnabled", iCollectProAntiFx.setGhost))
    chip(r, true, "Steal Tags", iCollectProFx.tags, saved("stealTagsEnabled", function(on) iCollectProFx.tags = on end))
    header("BASES", 23)
    r = pairRow(24)
    chip(r, false, "Base Timers", iCollectProFx.timerEsp, saved("baseTimerEspEnabled", function(on) iCollectProFx.timerEsp = on end))
    chip(r, true, "Every Floor", iCollectProFx.timerFloors, saved("timerEveryFloorEnabled", function(on) iCollectProFx.timerFloors = on end))
    r = pairRow(25)
    chip(r, false, "Base ESP", iCollectProFx.baseEsp, saved("baseEspEnabled", function(on) iCollectProFx.baseEsp = on end))
    chip(r, true, "Base X-Ray", iCollectProFx.xray, saved("baseXrayEnabled", function(on) iCollectProFx.xray = on end))

    tab("Move")
    -- each toggle sits right above its own number: speeds, then jump/gravity, then camera
    header("SPEED", 60)
    r = pairRow(61)
    chip(r, false, "Walk Speed", iCollectProFx.speedOn, saved("moveSpeedEnabled", function(on) iCollectProFx.speedOn = on end))
    chip(r, true, "Carpet Speed", iCollectProFx.carpet, saved("carpetSpeedEnabled", function(on) iCollectProFx.carpet = on end))
    bucket[#bucket + 1] = iCollectProUI.numPair(62, { "Walk", "walkSpeedValue", 28, 16, 80 }, { "Carpet", "carpetSpeedValue", 130, 16, 500 })
    bucket[#bucket + 1] = iCollectProUI.numPair(63, { "Giant Walk", "giantSpeedValue", 30, 16, 60 })
    header("JUMP & GRAVITY", 64)
    r = pairRow(65)
    chip(r, false, "Infinite Jump", iCollectProFx.infJump, saved("infJumpEnabled", iCollectProAntiFx.setInfJump))
    chip(r, true, "Gravity", iCollectProFx.grav, saved("gravityEnabled", function(on) iCollectProFx.grav = on end))
    bucket[#bucket + 1] = iCollectProUI.numPair(66, { "Gravity", "gravityValue", 196.2, 0, 300 })
    header("CAMERA", 67)
    r = pairRow(68)
    chip(r, nil, "Custom FOV", iCollectProFx.fov, saved("customFovEnabled", function(on) iCollectProFx.fov = on end))
    bucket[#bucket + 1] = iCollectProUI.slider(69, "FOV", "fovValue", 80, 30, 120)

    tab("Misc")
    header("HUD", 30)
    r = pairRow(31)
    chip(r, false, "Thief Bar", iCollectProFx.thiefBar, saved("thiefBarEnabled", function(on) iCollectProFx.thiefBar = on end))
    chip(r, true, "Speed Panel", isSpeedVisible, function(on)
        if HexSpeedFrameRef then
            HexSpeedFrameRef.Visible = on
            HubConfig.hexSpeedVisible = on
            saveHubConfig()
        end
    end)
    r = pairRow(32)
    chip(r, false, "Pill Bar", iCollectProFx.pillBar, saved("pillBarEnabled", function(on) iCollectProFx.pillBar = on end))
    chip(r, true, "Hide AP Icon", iCollectProFx.hideAP, saved("hideApIconEnabled", function(on) iCollectProFx.hideAP = on end))
    header("PERFORMANCE", 33)
    r = pairRow(34)
    chip(r, false, "FPS Boost", iCollectProFx.fps, saved("fpsOptimizerEnabled", function(on) iCollectProFx.fps = on end))
    chip(r, true, "Extreme FPS", iCollectProFx.xfps, saved("extremeFpsEnabled", function(on) iCollectProFx.xfps = on end))
    -- the Steal Keybind row (order 40) joins this tab below; phones have no keyboard
    if not IsMobile then header("CONTROLS", 38) end

    header("CONFIGS", 42)
    if iCollectProUI.configRows then
        for _, row in ipairs(iCollectProUI.configRows(43)) do bucket[#bucket + 1] = row end
    end
end

local Row4 = createRowFrame(30, 40)
if IsMobile then Row4.Visible = false end
rowLabel(Row4, "Steal Keybind", 0.5)

local KeybindBtn = pillButton(Row4, 56, currentStealKey.Name)

local isBinding = false
KeybindBtn.MouseButton1Click:Connect(function() isBinding = true; KeybindBtn.Text = "..." end)

local ContextActionService = game:GetService("ContextActionService")
local STEAL_ACTION = "iCollectPro_SemiTP_StealKey"

local function bindStealKey()
    pcall(function() ContextActionService:UnbindAction(STEAL_ACTION) end)
    if not currentStealKey then return end
    ContextActionService:BindActionAtPriority(STEAL_ACTION, function(_, state)
        if not iCollectPro_ALIVE() then
            pcall(function() ContextActionService:UnbindAction(STEAL_ACTION) end)
            return Enum.ContextActionResult.Pass
        end
        if isBinding or UserInputService:GetFocusedTextBox() then
            return Enum.ContextActionResult.Pass
        end
        if state == Enum.UserInputState.Begin and not iCollectPro.debounce and not player:GetAttribute("Stealing") then
            task.spawn(iCollectPro.execute)
        end
        return Enum.ContextActionResult.Sink
    end, false, Enum.ContextActionPriority.High.Value + 1000, currentStealKey)
end
bindStealKey()

UserInputService.InputBegan:Connect(function(input)
    if isBinding and input.UserInputType == Enum.UserInputType.Keyboard then
        isBinding = false
        currentStealKey = input.KeyCode
        KeybindBtn.Text = input.KeyCode.Name
        HubConfig.stealKeybind = input.KeyCode.Name
        saveHubConfig()
        bindStealKey()
    end
end)

local mainRows = { RowBase, Row1, InfoRow, Row6, Row5 }
local settingsRows = SettingsRows
settingsRows.Main = mainRows
settingsRows.Misc[#settingsRows.Misc + 1] = Row4

local function paintTab(btn, active)
    tween(btn.Pill, 0.2, { BackgroundTransparency = active and 0 or 1 })
    btn.Label.TextColor3 = active and Color3.fromRGB(255, 255, 255) or T.DIM
    btn.Label.Font = active and Enum.Font.GothamBlack or Enum.Font.GothamBold
end

local function switchTab(tabName)
    for name, btn in pairs(iCollectProUI.tabBtns) do
        paintTab(btn, name == tabName)
        for _, r in ipairs(settingsRows[name]) do r.Visible = name == tabName end
    end
    if IsMobile then Row4.Visible = false end
end

for name, btn in pairs(iCollectProUI.tabBtns) do
    btn.MouseButton1Click:Connect(function() switchTab(name) end)
end
switchTab("Main")

local SpeedScreenGui = Instance.new("ScreenGui")
SpeedScreenGui.Name = "iCollectPro_SemiTP_Speed"
SpeedScreenGui.ResetOnSpawn = false
SpeedScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
SpeedScreenGui.IgnoreGuiInset = true
SpeedScreenGui.DisplayOrder = 999
SpeedScreenGui.Parent = safeGuiTarget

local HexSpeed = Instance.new("Frame")
HexSpeed.Name = "SpeedPanel"
HexSpeed.Size = UDim2.new(0, 210, 0, 0)
HexSpeed.AutomaticSize = Enum.AutomaticSize.Y
HexSpeed.BackgroundColor3 = T.BG
HexSpeed.BorderSizePixel = 0
HexSpeed.Visible = isSpeedVisible
HexSpeed.ZIndex = 100
HexSpeed.Parent = SpeedScreenGui
HexSpeedFrameRef = HexSpeed
corner(HexSpeed, 18)
stroke(HexSpeed, T.STROKE, 1.2, 0.4)
addShadow(HexSpeed)

local SpeedPad = Instance.new("UIPadding")
SpeedPad.PaddingBottom = UDim.new(0, 10)
SpeedPad.Parent = HexSpeed

HexSpeed.AnchorPoint = Vector2.new(1, 0)
if HubConfig.speedPos and HubConfig.speedPos.ScaleX ~= 0.5 then
    HexSpeed.Position = UDim2.new(HubConfig.speedPos.ScaleX, HubConfig.speedPos.OffsetX, HubConfig.speedPos.ScaleY, HubConfig.speedPos.OffsetY)
else
    HexSpeed.Position = UDim2.new(0.98, 0, 0.04, 0)
end

local SpeedHeader = Instance.new("Frame")
SpeedHeader.Size = UDim2.new(1, 0, 0, 36)
SpeedHeader.BackgroundTransparency = 1
SpeedHeader.ZIndex = 101
SpeedHeader.Parent = HexSpeed

local SpeedTitle = Instance.new("TextLabel")
SpeedTitle.Size = UDim2.new(1, -28, 0, 22)
SpeedTitle.Position = UDim2.new(0, 14, 0, 7)
SpeedTitle.BackgroundTransparency = 1
SpeedTitle.Text = "STEAL SPEED"
SpeedTitle.Font = Enum.Font.GothamBlack
SpeedTitle.TextSize = 16
SpeedTitle.TextColor3 = T.TEXT
SpeedTitle.ZIndex = 102
SpeedTitle.Parent = SpeedHeader

local SpeedRule = Instance.new("Frame")
SpeedRule.AnchorPoint = Vector2.new(0.5, 0)
SpeedRule.Position = UDim2.new(0.5, 0, 0, 32)
SpeedRule.Size = UDim2.new(0, 110, 0, 1)
SpeedRule.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
SpeedRule.BackgroundTransparency = 0.15
SpeedRule.BorderSizePixel = 0
SpeedRule.ZIndex = 101
SpeedRule.Parent = HexSpeed

local SpeedCard = Instance.new("Frame")
SpeedCard.AutomaticSize = Enum.AutomaticSize.Y
SpeedCard.Size = UDim2.new(1, -20, 0, 0)
SpeedCard.Position = UDim2.fromOffset(10, 40)
SpeedCard.BackgroundColor3 = T.SURF
SpeedCard.BorderSizePixel = 0
SpeedCard.ZIndex = 101
SpeedCard.Parent = HexSpeed
corner(SpeedCard, 16)
stroke(SpeedCard, T.STROKE, 1, 0.48)

local SpeedCardPad = Instance.new("UIPadding")
SpeedCardPad.PaddingTop, SpeedCardPad.PaddingBottom = UDim.new(0, 4), UDim.new(0, 4)
SpeedCardPad.PaddingLeft, SpeedCardPad.PaddingRight = UDim.new(0, 4), UDim.new(0, 4)
SpeedCardPad.Parent = SpeedCard

local SpeedList = Instance.new("UIListLayout", SpeedCard)
SpeedList.SortOrder = Enum.SortOrder.LayoutOrder
SpeedList.Padding = UDim.new(0, 6)

local function speedRow(order)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 30)
    row.BackgroundColor3 = T.SURF2
    row.BackgroundTransparency = 0.02
    row.BorderSizePixel = 0
    row.LayoutOrder = order
    row.ZIndex = 103
    row.Parent = SpeedCard
    corner(row, 11)
    local rs = stroke(row, T.STROKE, 1, 0.52)
    addHover(row, T.SURF2, T.HOVER, rs)
    return row
end

local BoostRow = speedRow(1)
createModernToggle(BoostRow, "Auto on Steal", _G.iCollectPro_SemiTP_SpeedBoost, function(newState)
    _G.iCollectPro_SemiTP_SpeedBoost = newState
    HubConfig.stealBoostAutoOnSteal = newState
    saveHubConfig()
end)

local SpeedValueRow = speedRow(2)
rowLabel(SpeedValueRow, "Speed (max 23)", 0.6)

local InputBox = Instance.new("TextBox")
InputBox.Size = UDim2.fromOffset(56, 22)
InputBox.Position = UDim2.new(1, -66, 0.5, -11)
InputBox.BackgroundColor3 = T.OFF_BG
InputBox.BorderSizePixel = 0
InputBox.Text = tostring(stealSpeed())
InputBox.TextColor3 = T.TEXT
InputBox.Font = Enum.Font.GothamBold
InputBox.TextSize = 11
InputBox.ClearTextOnFocus = false
InputBox.ZIndex = 104
InputBox.Parent = SpeedValueRow
corner(InputBox, 7)
stroke(InputBox, T.STROKE, 1, 0.55)

InputBox.FocusLost:Connect(function()
    local num = tonumber(InputBox.Text)
    if num then
        num = math.clamp(math.floor(num + 0.5), STEAL_SPEED_MIN, STEAL_SPEED_MAX)
        currentSpeed = num
        HubConfig.speedValue = num
        saveHubConfig()
    end
    InputBox.Text = tostring(stealSpeed())
end)

makeDraggable(SpeedHeader, HexSpeed, function()
    HubConfig.speedPos = { ScaleX = HexSpeed.Position.X.Scale, OffsetX = HexSpeed.Position.X.Offset, ScaleY = HexSpeed.Position.Y.Scale, OffsetY = HexSpeed.Position.Y.Offset }
    saveHubConfig()
end)

-- Head pills, fixed screen size (offset-sized, so zoom doesn't change them):
--   TARGET over whoever owns the base you're aiming at: STEALING "<brainrot>" / INSIDE YOUR BASE !! /
--   INSIDE HIS BASE / OUTSIDE BASE
--   <username> over anyone else standing inside YOUR base: INSIDE YOUR BASE !! / STEALING "<brainrot>"
do
    local function makePill(name, titleText)
        local bb = Instance.new("BillboardGui")
        bb.Name = name
        -- pixel-sized (no scale part), so it is this exact size on screen at any distance or zoom
        bb.Size = UDim2.fromOffset(120, 32)
        bb.StudsOffset = Vector3.new(0, 2.6, 0)
        bb.AlwaysOnTop = true
        bb.LightInfluence = 0
        bb.MaxDistance = 100000
        bb.Enabled = false
        bb.Parent = AllowDisallowGui

        local pill = Instance.new("Frame")
        pill.Size = UDim2.fromScale(1, 1)
        pill.BackgroundColor3 = T.BG
        pill.BorderSizePixel = 0
        pill.Parent = bb
        pill.BackgroundTransparency = 0.12
        corner(pill, 16)
        local pillStroke = stroke(pill, T.ACCENT, 1.5, 0.1)
        gradient(pill, T.SURF2, T.BG, 90)

        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -14, 0, 15)
        title.Position = UDim2.fromOffset(7, 3)
        title.BackgroundTransparency = 1
        title.Font = Enum.Font.GothamBlack
        title.TextSize = 13
        title.TextScaled = false
        title.TextTruncate = Enum.TextTruncate.AtEnd
        title.TextColor3 = T.TEXT
        title.TextStrokeTransparency = 0.6
        title.Text = titleText
        title.Parent = pill

        local sub = Instance.new("TextLabel")
        sub.Size = UDim2.new(1, -14, 0, 11)
        sub.Position = UDim2.fromOffset(7, 18)
        sub.BackgroundTransparency = 1
        sub.Font = Enum.Font.GothamBold
        sub.TextSize = 9
        sub.TextColor3 = T.DIM
        sub.TextTruncate = Enum.TextTruncate.AtEnd
        sub.Text = ""
        sub.Parent = pill
        return { bb = bb, title = title, sub = sub, stroke = pillStroke }
    end

    local target = makePill("iCollectPro_SemiTP_TargetPill", "TARGET")
    local bb, sub, pillStroke = target.bb, target.sub, target.stroke
    local intruders = {}
    local tracers, myAtt = {}, nil

    local COL_STEAL = Color3.fromRGB(255, 110, 130)
    local COL_IN = Color3.fromRGB(110, 235, 160)

    local function ownerOf(plot)
        local name = plotOwnerName(plot)
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Name == name or p.DisplayName == name then return p end
        end
        return nil
    end

    local boxes = setmetatable({}, { __mode = "k" })
    local function insidePlot(plot, pos)
        if not plot then return false end
        local box = boxes[plot]
        if not box then
            local ok, cf, size = pcall(plot.GetBoundingBox, plot)
            box = ok and { cf, size } or false
            boxes[plot] = box
        end
        if not box then return false end
        local l = box[1]:PointToObjectSpace(pos)
        return math.abs(l.X) <= box[2].X / 2 and math.abs(l.Z) <= box[2].Z / 2
    end

    task.spawn(function()
        while iCollectPro_ALIVE() and bb.Parent do
            local podium = getTargetPodiumForSlot(selectedSlot)
            -- podium lives at Plots.<plot>.AnimalPodiums.<n>
            local plot = targetPlot or (podium and podium.Parent and podium.Parent.Parent)
            local owner = plot and ownerOf(plot)
            local head = owner and owner.Character and owner.Character:FindFirstChild("Head")
            local root = owner and owner.Character and owner.Character:FindFirstChild("HumanoidRootPart")
            if head and root then
                bb.Adornee = head
                bb.Enabled = true
                local idx = owner:GetAttribute("StealingIndex")
                local col
                if owner:GetAttribute("Stealing") then
                    sub.Text = (idx ~= nil and idx ~= "") and ('STEALING "' .. string.upper(tostring(idx)) .. '"') or "STEALING"
                    col = COL_STEAL
                elseif insidePlot(findMyBase(), root.Position) then
                    sub.Text = "INSIDE YOUR BASE !!"
                    col = COL_STEAL
                elseif insidePlot(plot, root.Position) then
                    sub.Text = "INSIDE HIS BASE"
                    col = COL_IN
                else
                    sub.Text = "OUTSIDE BASE"
                    col = T.ACCENT
                end
                sub.TextColor3 = col
                pillStroke.Color = col
                pillStroke.Transparency = (col == COL_STEAL and (os.clock() % 0.6 < 0.3)) and 0.6 or 0.1
            else
                bb.Enabled = false
                bb.Adornee = nil
            end

            -- everyone else inside your base gets their own pill with their username
            local myPlot = findMyBase()
            local seen = {}
            for _, p in ipairs(Players:GetPlayers()) do
                local ch = p.Character
                local h = ch and ch:FindFirstChild("Head")
                local r = ch and ch:FindFirstChild("HumanoidRootPart")
                if p ~= player and p ~= owner and h and r and insidePlot(myPlot, r.Position) then
                    seen[p] = true
                    local pl = intruders[p]
                    if not pl then
                        pl = makePill("iCollectPro_SemiTP_IntruderPill", p.Name)
                        intruders[p] = pl
                    end
                    pl.title.Text = p.Name
                    pl.bb.Adornee = h
                    pl.bb.Enabled = true
                    local pidx = p:GetAttribute("StealingIndex")
                    if p:GetAttribute("Stealing") then
                        pl.sub.Text = (pidx ~= nil and pidx ~= "") and ('STEALING "' .. string.upper(tostring(pidx)) .. '"') or "STEALING"
                    else
                        pl.sub.Text = "INSIDE YOUR BASE !!"
                    end
                    pl.sub.TextColor3 = COL_STEAL
                    pl.stroke.Color = COL_STEAL
                    pl.stroke.Transparency = (os.clock() % 0.6 < 0.3) and 0.6 or 0.1
                end
            end
            for p, pl in pairs(intruders) do
                if not seen[p] then
                    pl.bb:Destroy()
                    intruders[p] = nil
                end
            end

            -- Tracers: a line from you to the target owner (purple) and to every intruder (red)
            local want = {}
            local myHrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if iCollectProFx.tracers and myHrp then
                if owner and root then want[owner] = { root, T.ACCENT } end
                for p in pairs(seen) do
                    local r = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                    if r then want[p] = { r, COL_STEAL } end
                end
                if not myAtt or myAtt.Parent ~= myHrp then
                    if myAtt then myAtt:Destroy() end
                    myAtt = Instance.new("Attachment")
                    myAtt.Name = "iCollectPro_SemiTP_Tracer"
                    myAtt.Parent = myHrp
                end
            end
            for p, tr in pairs(tracers) do
                local w = want[p]
                if not w or tr.att.Parent ~= w[1] or tr.beam.Attachment0 ~= myAtt then
                    tr.att:Destroy()
                    tracers[p] = nil
                end
            end
            for p, w in pairs(want) do
                local tr = tracers[p]
                if not tr then
                    local att = Instance.new("Attachment")
                    att.Name = "iCollectPro_SemiTP_Tracer"
                    att.Parent = w[1]
                    local beam = Instance.new("Beam")
                    beam.Attachment0, beam.Attachment1 = myAtt, att
                    beam.Width0, beam.Width1 = 0.12, 0.12
                    beam.FaceCamera = true
                    beam.LightEmission = 1
                    beam.Segments = 1
                    beam.Parent = att
                    tr = { att = att, beam = beam }
                    tracers[p] = tr
                end
                tr.beam.Color = ColorSequence.new(w[2])
            end
            if not iCollectProFx.tracers and myAtt then myAtt:Destroy(); myAtt = nil end
            task.wait(0.15)
        end
        bb:Destroy()
        for _, pl in pairs(intruders) do pl.bb:Destroy() end
        for _, tr in pairs(tracers) do tr.att:Destroy() end
        if myAtt then myAtt:Destroy() end
    end)
end

-- Base X-Ray (ported from iCollectPro INSTA STEAL's See Thru Base): the target base's walls and
-- floors fade to see-through so you can watch the brainrots and the owner inside. Uses
-- LocalTransparencyModifier - a client-only render override that never reaches the server and
-- never touches the parts' real Transparency - and leaves AnimalPodiums (podiums + brainrots)
-- solid. Collision is untouched. Parts streaming in are queued and drained 5x a second.
-- Same coverage as MEERKO EXTRAS' "XRAY BASES": EVERY base, only its building folders plus each
-- podium's Claim pad and decorations, faded to 0.9 - brainrots and podium stands stay solid.
do
    local FADE = 0.9
    local FOLDERS = { "Base", "PlotSign", "FriendPanel", "Cash", "Laser", "Decorations", "Skin", "Unlock", "Purchases" }
    local faded, queue, conns = {}, {}, {}
    local on = false

    local function one(d)
        if d:IsA("BasePart") and not faded[d] then
            faded[d] = true
            d.LocalTransparencyModifier = FADE
        end
    end

    local function track(root)
        if not root then return end
        one(root)
        for _, d in ipairs(root:GetDescendants()) do one(d) end
        conns[#conns + 1] = root.DescendantAdded:Connect(function(d) queue[#queue + 1] = d end)
    end

    local function doPlot(plot)
        for _, name in ipairs(FOLDERS) do track(plot:FindFirstChild(name)) end
        conns[#conns + 1] = plot.ChildAdded:Connect(function(c)
            if table.find(FOLDERS, c.Name) then task.defer(track, c) end
        end)
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then return end
        local function pod(pd)
            track(pd:FindFirstChild("Claim"))
            local b = pd:FindFirstChild("Base")
            track(b and b:FindFirstChild("Decorations"))
        end
        for _, pd in ipairs(pods:GetChildren()) do pod(pd) end
        conns[#conns + 1] = pods.ChildAdded:Connect(function(pd) task.delay(0.1, pod, pd) end)
    end

    local function enable()
        local plots = Workspace:FindFirstChild("Plots")
        if not plots then return false end
        for _, p in ipairs(plots:GetChildren()) do pcall(doPlot, p) end
        conns[#conns + 1] = plots.ChildAdded:Connect(function(p) task.delay(0.2, function() if on then pcall(doPlot, p) end end) end)
        return true
    end

    local function disable()
        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        table.clear(conns); table.clear(queue)
        for p in pairs(faded) do pcall(function() if p.Parent then p.LocalTransparencyModifier = 0 end end) end
        table.clear(faded)
    end

    task.spawn(function()
        while iCollectPro_ALIVE() do
            if iCollectProFx.xray and not on then
                on = enable()
            elseif not iCollectProFx.xray and on then
                on = false
                disable()
            end
            if on and #queue > 0 then
                local batch = queue
                queue = {}
                for _, d in ipairs(batch) do if d.Parent then pcall(one, d) end end
            end
            task.wait(0.2)
        end
        disable()
    end)
end

_G.iCollectPro_SemiTP = iCollectPro
iCollectPro_ENV.iCollectPro_SemiTP = iCollectPro
table.insert(iCollectPro_ENV.iCollectPro_SemiTP_Tables, iCollectPro)
_G.iCollectPro_SemiTP_Steal = function() pcall(iCollectPro.execute) end
_G.iCollectPro_SemiTP_SetPodium = function(slot) iCollectPro.setSlot(slot) end

iCollectPro.debounce = false

-- Auto-fit to the user's screen (same idea as iCollectPro INSTA STEAL's bar): ONE factor for the
-- whole UI. Desktop follows the resolution against 1080p; phones/tablets size off their short
-- side. Panels also keep inside the screen, and the menu never grows taller than 80% of it.
;(function()
    local function viewport()
        local cam = Workspace.CurrentCamera
        local vp = cam and cam.ViewportSize
        if not vp or vp.X < 2 or vp.Y < 2 then vp = Vector2.new(1920, 1080) end
        return vp
    end
    function iCollectProUI.uiFactor()
        local vp = viewport()
        if IsMobile then
            local short = math.min(vp.X, vp.Y)
            return math.clamp(short / (short < 500 and 430 or 650), 0.6, 1.15)
        end
        return math.clamp(math.min(vp.X / 1920, vp.Y / 1080), 0.7, 1.35)
    end

    -- { frame, base multiplier }: the menu and speed panel were drawn small, so they sit a bit larger
    local fitted = { { MainFrame, 1.2 }, { HexSpeed, 1.2 }, { PBarMain, 1 }, { MainAllowBtn, 1 } }
    local scalers = {}
    for i, e in ipairs(fitted) do
        local s = e[1]:FindFirstChildOfClass("UIScale") or Instance.new("UIScale")
        s.Parent = e[1]
        scalers[i] = s
    end

    local function keepOnScreen(frame)
        local sg = frame:FindFirstAncestorOfClass("ScreenGui")
        if not sg or not frame.Visible then return end
        local bounds, pos, size = sg.AbsoluteSize, frame.AbsolutePosition, frame.AbsoluteSize
        local dx, dy = 0, 0
        if pos.X < 0 then dx = -pos.X elseif pos.X + size.X > bounds.X then dx = bounds.X - (pos.X + size.X) end
        if pos.Y < 0 then dy = -pos.Y elseif pos.Y + size.Y > bounds.Y then dy = bounds.Y - (pos.Y + size.Y) end
        if math.abs(dx) > 0.5 or math.abs(dy) > 0.5 then
            frame.Position = frame.Position + UDim2.fromOffset(math.floor(dx), math.floor(dy))
        end
    end

    local busy = false
    local function update()
        if busy or not iCollectPro_ALIVE() then return end
        busy = true
        local f, vp = iCollectProUI.uiFactor(), viewport()
        for i, e in ipairs(fitted) do
            local s = f * e[2]
            if e[1] == MainFrame then
                -- never taller than 80% of the screen, whatever tab is open
                local cur = scalers[i].Scale
                local baseH = cur > 0 and MainFrame.AbsoluteSize.Y / cur or 0
                if baseH > 0 then s = math.min(s, vp.Y * 0.8 / baseH) end
            end
            s = math.clamp(s, 0.5, 1.6)
            if math.abs(scalers[i].Scale - s) > 0.01 then scalers[i].Scale = s end
        end
        task.defer(function()
            for _, e in ipairs(fitted) do pcall(keepOnScreen, e[1]) end
            busy = false
        end)
    end
    iCollectProUI.fitUI = update

    update()
    local camConn
    local function watchCam()
        if camConn then camConn:Disconnect() end
        local cam = Workspace.CurrentCamera
        if cam then camConn = cam:GetPropertyChangedSignal("ViewportSize"):Connect(update) end
        update()
    end
    watchCam()
    Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(watchCam)
    -- switching tabs changes the menu's height
    MainFrame:GetPropertyChangedSignal("AbsoluteSize"):Connect(function() task.defer(update) end)

    -- pills / tags / base boards over the world: grow or shrink with the same factor, so they stay
    -- one size on screen but match the resolution (a 4K screen gets bigger pills, a phone smaller)
    local function fitBillboard(bb)
        if not bb:IsA("BillboardGui") then return end
        task.defer(function()
            if not bb.Parent then return end
            local w, h = bb:GetAttribute("__semiW"), bb:GetAttribute("__semiH")
            if not w then
                if bb.Size.X.Scale ~= 0 or bb.Size.Y.Scale ~= 0 then return end
                w, h = bb.Size.X.Offset, bb.Size.Y.Offset
                bb:SetAttribute("__semiW", w)
                bb:SetAttribute("__semiH", h)
            end
            local f = iCollectProUI.uiFactor()
            bb.Size = UDim2.fromOffset(w * f, h * f)
            local sc = bb:FindFirstChild("__semiScale") or Instance.new("UIScale")
            sc.Name = "__semiScale"
            sc.Scale = f
            sc.Parent = bb
        end)
    end
    for _, d in ipairs(AllowDisallowGui:GetDescendants()) do fitBillboard(d) end
    AllowDisallowGui.DescendantAdded:Connect(fitBillboard)
    local lastF = iCollectProUI.uiFactor()
    task.spawn(function()
        while iCollectPro_ALIVE() do
            task.wait(1)
            local f = iCollectProUI.uiFactor()
            if math.abs(f - lastF) > 0.01 then
                lastF = f
                for _, d in ipairs(AllowDisallowGui:GetDescendants()) do fitBillboard(d) end
            end
        end
    end)
end)()

-- UI sounds, same as iCollectPro INSTA STEAL: a pop on every click, and a quiet tick (one of three
-- clips at random) on hover. Hooks every button in our guis, including ones built later.
-- Its own function, not a do-block: the main chunk is at Luau's 200-register limit.
;(function()
    local POP_ID = "rbxassetid://102289499477049"
    local HOVER_IDS = { "rbxassetid://95003901725897", "rbxassetid://72335876826381", "rbxassetid://82122230376488" }
    local sounds = {}
    local function play(id, vol)
        local s = sounds[id]
        if not s or not s.Parent then
            local ok, made = pcall(function()
                local n = Instance.new("Sound")
                n.Name = "__SemiTPUISound"
                n.SoundId = id
                n.Volume = vol
                n.Parent = ScreenGui
                return n
            end)
            s = ok and made or nil
            sounds[id] = s
        end
        if s then pcall(function() s.TimePosition = 0; s:Play() end) end
    end
    -- load them up front so the very first click/hover isn't silent
    task.defer(function()
        for _, id in ipairs({ POP_ID, table.unpack(HOVER_IDS) }) do
            play(id, 0)
            if sounds[id] then pcall(function() sounds[id]:Stop() end) end
        end
    end)
    local hooked = setmetatable({}, { __mode = "k" })
    local function hook(b)
        if hooked[b] or not b:IsA("GuiButton") then return end
        hooked[b] = true
        -- press / hover animation (same feel as INSTA STEAL): grow a little on hover, squash to 92%
        -- on the press and spring back. The slider track is skipped so dragging stays steady.
        local sc
        if not b:GetAttribute("__semiNoPop") and not (b.Text == "" and b:FindFirstChildWhichIsA("GuiObject") == nil) then
            sc = b:FindFirstChildOfClass("UIScale") or Instance.new("UIScale")
            sc.Parent = b
        end
        local over = false
        local function to(s, t, style)
            if sc then
                TweenService:Create(sc, TweenInfo.new(t, style or Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Scale = s }):Play()
            end
        end
        b.Activated:Connect(function()
            play(POP_ID, 0.6)
            if sc then
                sc.Scale = 0.92
                to(over and 1.04 or 1, 0.22, Enum.EasingStyle.Back)
            end
        end)
        b.MouseEnter:Connect(function()
            over = true
            play(HOVER_IDS[math.random(1, #HOVER_IDS)], 0.12)
            to(1.04, 0.12)
        end)
        b.MouseLeave:Connect(function()
            over = false
            to(1, 0.12)
        end)
    end
    for _, gui in ipairs({ ScreenGui, SpeedScreenGui, AllowDisallowGui, ProgressGui }) do
        for _, d in ipairs(gui:GetDescendants()) do hook(d) end
        gui.DescendantAdded:Connect(hook)
    end
end)()

-- From Aurora, batch 3 (all off by default, each its own function for the register budget):
-- Base Timers, Base ESP, Steal Tags, Thief Bar, Kick at 2s,
-- Walk/Giant/Carpet speed + Gravity, FPS Boost.
;(function()
local iCollectProAur = {
    RED = Color3.fromRGB(255, 110, 130),
    GREEN = Color3.fromRGB(110, 235, 160),
    YELLOW = Color3.fromRGB(255, 205, 90),
    INVIS = Color3.fromRGB(195, 135, 255),
}

-- two-line pill, identical to the TARGET pill: bold name on top, coloured status underneath
function iCollectProAur.pill()
    local t = iCollectProAur.tag(120, 32)
    t.lbl.Size = UDim2.new(1, -14, 0, 15)
    t.lbl.Position = UDim2.fromOffset(7, 3)
    t.lbl.TextSize = 13
    t.lbl.TextStrokeTransparency = 0.6
    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(1, -14, 0, 11)
    sub.Position = UDim2.fromOffset(7, 18)
    sub.BackgroundTransparency = 1
    sub.Font = Enum.Font.GothamBold
    sub.TextSize = 9
    sub.TextTruncate = Enum.TextTruncate.AtEnd
    sub.Parent = t.lbl.Parent
    t.sub = sub
    return t
end

function iCollectProAur.paint2(t, title, status, col)
    if t.lbl.Text ~= title then t.lbl.Text = title end
    if t.sub.Text ~= status then t.sub.Text = status end
    t.lbl.TextColor3 = T.TEXT
    t.sub.TextColor3 = col
    t.stroke.Color = col
    t.bb.Enabled = true
end

-- small pixel-sized pill (same look as the head pills; one size at any zoom/distance)
function iCollectProAur.tag(w, h)
    local bb = Instance.new("BillboardGui")
    bb.Size = UDim2.fromOffset(w, h)
    bb.AlwaysOnTop = true
    bb.LightInfluence = 0
    bb.MaxDistance = 100000
    bb.Enabled = false
    bb.Parent = AllowDisallowGui
    local f = Instance.new("Frame")
    f.Size = UDim2.fromScale(1, 1)
    f.BackgroundColor3 = T.BG
    f.BackgroundTransparency = 0.12
    f.BorderSizePixel = 0
    f.Parent = bb
    corner(f, h // 2)
    gradient(f, T.SURF2, T.BG, 90)
    local s = stroke(f, T.ACCENT, 1.5, 0.1)
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -10, 1, 0)
    l.Position = UDim2.fromOffset(5, 0)
    l.BackgroundTransparency = 1
    l.Font = Enum.Font.GothamBlack
    l.TextSize = 11
    l.TextColor3 = T.TEXT
    l.TextTruncate = Enum.TextTruncate.AtEnd
    l.Parent = f
    return { bb = bb, lbl = l, stroke = s }
end

function iCollectProAur.paint(t, text, col)
    if t.lbl.Text ~= text then t.lbl.Text = text end
    t.lbl.TextColor3 = col
    t.stroke.Color = col
    t.bb.Enabled = true
end

-- the base lock board: UNLOCKED / RELOCK IN n / LOCKED n (same reading as Aurora)
function iCollectProAur.timerState(plot)
    local pur = plot:FindFirstChild("Purchases")
    local pb = pur and pur:FindFirstChild("PlotBlock")
    local main = pb and pb:FindFirstChild("Main")
    local gui = main and main:FindFirstChild("BillboardGui")
    if not gui then return nil end
    local rt = gui:FindFirstChild("RemainingTime")
    if not rt or not rt.Visible then return "UNLOCKED", iCollectProAur.GREEN, main, nil end
    local n = tonumber(rt.Text:match("(%d+)"))
    local ts = n and (n >= 60 and string.format("%d:%02d", n // 60, n % 60) or (n .. "s")) or ""
    local delay = gui:FindFirstChild("Delay")
    if delay and delay.Visible then return "RELOCK IN " .. ts, iCollectProAur.YELLOW, main, n end
    return "LOCKED " .. ts, iCollectProAur.RED, main, n
end

function iCollectProAur.hitbox(plot)
    local h = plot and plot:FindFirstChild("StealHitbox", true)
    return h and h:IsA("BasePart") and h or nil
end

function iCollectProAur.inBox(part, pos)
    if not part then return false end
    local l = part.CFrame:PointToObjectSpace(pos)
    return math.abs(l.X) <= part.Size.X / 2 and math.abs(l.Z) <= part.Size.Z / 2
end

-- Base Timers (+ Every Floor) and Base ESP
;(function()
    local boards, boxes = {}, {}
    local function clearBoards(plot)
        for _, t in ipairs(boards[plot] or {}) do t.bb:Destroy() end
        boards[plot] = nil
    end
    local function floorsOf(plot)
        local list = {}
        local un = plot:FindFirstChild("Unlock")
        for _, c in ipairs(un and un:GetChildren() or {}) do
            local ub = c:FindFirstChild("UnlockBase")
            if c:IsA("BasePart") and ub and ub:GetAttribute("Floor") then list[#list + 1] = c end
        end
        table.sort(list, function(a, b) return a.Position.Y < b.Position.Y end)
        return list
    end
    -- double-click a board = the game's own "Unlock Base" prompt for that floor (Unlock.<part>.UnlockBase):
    -- its Triggered handler asks the server, which opens the Robux unlock window. Only locked floors
    -- have it switched on. A single board (Every Floor off) uses the lowest locked floor.
    local function unlockPrompt(plot, anchor)
        local un = plot and plot:FindFirstChild("Unlock")
        if not un then return nil end
        if anchor and anchor.Parent == un then
            local pp = anchor:FindFirstChild("UnlockBase")
            return pp and pp:IsA("ProximityPrompt") and pp or nil
        end
        local best
        for _, c in ipairs(un:GetChildren()) do
            local pp = c:FindFirstChild("UnlockBase")
            if pp and pp:IsA("ProximityPrompt") and pp.Enabled and c:IsA("BasePart")
                and (not best or c.Position.Y < best.Parent.Position.Y) then
                best = pp
            end
        end
        return best
    end
    local function openUnlock(t)
        if not t.plot or t.plot == findMyBase() then return end
        local pp = unlockPrompt(t.plot, t.anchor)
        if not pp or not pp.Enabled then
            setStealStatus("THAT FLOOR ISN'T LOCKED", false)
            return
        end
        setStealStatus("OPENING UNLOCK...", nil)
        if firesignal then
            pcall(firesignal, pp.Triggered, player)
        elseif fireproximityprompt then
            pcall(fireproximityprompt, pp)
        end
    end
    local function clickable(t)
        t.bb.Active = true
        local b = Instance.new("TextButton")
        b.Size = UDim2.fromScale(1, 1)
        b.BackgroundTransparency = 1
        b.Text = ""
        b.ZIndex = 5
        b.Parent = t.lbl.Parent
        local last = 0
        b.MouseButton1Click:Connect(function()
            local now = os.clock()
            if now - last <= 0.4 then
                last = 0
                openUnlock(t)
            else
                last = now
            end
        end)
    end
    task.spawn(function()
        while iCollectPro_ALIVE() do
            local plots = Workspace:FindFirstChild("Plots")
            local mine = findMyBase()
            local seen = {}
            for _, plot in ipairs(plots and plots:GetChildren() or {}) do
                local owner = plotOwnerName(plot)
                local empty = owner == "?" or owner == "Empty Base"
                local text, col, main = iCollectProAur.timerState(plot)
                -- timer boards
                if iCollectProFx.timerEsp and text and not empty then
                    seen[plot] = true
                    local anchors = iCollectProFx.timerFloors and floorsOf(plot) or {}
                    if #anchors == 0 then anchors = { main } end
                    local list = boards[plot]
                    if not list or #list ~= #anchors then
                        clearBoards(plot)
                        list = {}
                        for _ = 1, #anchors do
                            local t = iCollectProAur.pill()
                            clickable(t)
                            list[#list + 1] = t
                        end
                        boards[plot] = list
                    end
                    local who = plot == mine and "YOUR BASE" or owner
                    for i, t in ipairs(list) do
                        t.plot, t.anchor = plot, anchors[i]
                        t.bb.Adornee = anchors[i]
                        t.bb.StudsOffsetWorldSpace = Vector3.new(0, 4, 0)
                        iCollectProAur.paint2(t, who, (#list > 1 and ("FLOOR " .. i .. "  ·  ") or "") .. text, col)
                    end
                end
                -- base outline around the steal zone: yours green, locked red, open purple
                local hb = iCollectProFx.baseEsp and not empty and iCollectProAur.hitbox(plot)
                if hb then
                    local sb = boxes[plot]
                    if not sb or not sb.Parent then
                        sb = Instance.new("SelectionBox")
                        sb.LineThickness = 0.12
                        sb.SurfaceTransparency = 1
                        sb.Parent = AllowDisallowGui
                        boxes[plot] = sb
                    end
                    sb.Adornee = hb
                    sb.Color3 = plot == mine and iCollectProAur.GREEN or (col == iCollectProAur.RED and iCollectProAur.RED or T.ACCENT)
                elseif boxes[plot] then
                    boxes[plot]:Destroy()
                    boxes[plot] = nil
                end
            end
            for plot in pairs(boards) do
                if not seen[plot] then clearBoards(plot) end
            end
            for plot, sb in pairs(boxes) do
                if not plot.Parent then sb:Destroy() boxes[plot] = nil end
            end
            task.wait(0.25)
        end
        for plot in pairs(boards) do clearBoards(plot) end
        for _, sb in pairs(boxes) do sb:Destroy() end
    end)
end)()

-- Steal Tags: STEALING "<brainrot>" / INVISIBLE over other players (sits just above the head pills),
-- plus a see-through highlight on anyone wearing the Invisibility Cloak
;(function()
    local tags, lights = {}, {}
    local function invisible(ch)
        local lvl = ch:GetAttribute("InvisibilityLevel")
        if type(lvl) ~= "number" or lvl <= 0 then return false end
        for _, m in ipairs(ch:GetChildren()) do
            if m:IsA("Model") then
                local n = 0
                for _, d in ipairs(m:GetDescendants()) do
                    if d:IsA("BasePart") then n = n + 1 if n >= 2 then return false end end
                end
            end
        end
        return true
    end
    task.spawn(function()
        while iCollectPro_ALIVE() do
            local seen = {}
            if iCollectProFx.tags then
                for _, p in ipairs(Players:GetPlayers()) do
                    local ch = p ~= player and p.Character
                    local head = ch and ch:FindFirstChild("Head")
                    if head then
                        local st = p:GetAttribute("Stealing")
                        local stealing = st ~= nil and st ~= false
                        local inv = invisible(ch)
                        if stealing or inv then
                            seen[p] = true
                            local t = tags[p]
                            if not t then
                                t = iCollectProAur.tag(130, 18)
                                t.bb.StudsOffset = Vector3.new(0, 2.6, 0)
                                t.bb.SizeOffset = Vector2.new(0, 1.7)
                                tags[p] = t
                            end
                            t.bb.Adornee = head
                            local idx = p:GetAttribute("StealingIndex")
                            local s = stealing and ((idx ~= nil and idx ~= "") and ('STEALING "' .. string.upper(tostring(idx)) .. '"') or "STEALING") or nil
                            iCollectProAur.paint(t, inv and (s and ("INVISIBLE · " .. s) or "INVISIBLE") or s, inv and iCollectProAur.INVIS or iCollectProAur.RED)
                            if inv then
                                local hl = lights[p]
                                if not hl or not hl.Parent then
                                    hl = Instance.new("Highlight")
                                    hl.FillColor = iCollectProAur.INVIS
                                    hl.OutlineColor = iCollectProAur.INVIS
                                    hl.FillTransparency = 0.6
                                    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                                    hl.Parent = AllowDisallowGui
                                    lights[p] = hl
                                end
                                hl.Adornee = ch
                            elseif lights[p] then
                                lights[p]:Destroy()
                                lights[p] = nil
                            end
                        end
                    end
                end
            end
            for p, t in pairs(tags) do
                if not seen[p] then t.bb:Destroy() tags[p] = nil end
            end
            for p, hl in pairs(lights) do
                if not seen[p] then hl:Destroy() lights[p] = nil end
            end
            task.wait(0.15)
        end
        for _, t in pairs(tags) do t.bb:Destroy() end
        for _, hl in pairs(lights) do hl:Destroy() end
    end)
end)()

-- Thief Bar: when someone grabs a brainrot inside YOUR base, a red bar at the top shows who, what,
-- and how far they've carried it toward their own base (full = about to deliver)
;(function()
    local bar = Instance.new("Frame")
    bar.AnchorPoint = Vector2.new(0.5, 0)
    bar.Position = UDim2.new(0.5, 0, 0, 70)
    -- same build (and purple theme) as the SEMI TP progress bar, with a HIT button on the right
    bar.Size = UDim2.fromOffset(320, 50)
    bar.BackgroundColor3 = T.SURF
    bar.BackgroundTransparency = 0.02
    bar.BorderSizePixel = 0
    bar.Visible = false
    bar.ZIndex = 80
    bar.Parent = ProgressGui
    corner(bar, 12)
    gradient(bar, T.SURF2, T.BG, 90)
    stroke(bar, T.STROKE, 1, 0.35)
    local barStroke = stroke(bar, T.ACCENT, 3, 0.6)
    addShadow(bar, 20)
    local barScale = Instance.new("UIScale")
    barScale.Parent = bar
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -90, 0, 14)
    title.Position = UDim2.fromOffset(7, 4)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBlack
    title.TextSize = 11
    title.TextColor3 = T.TEXT
    title.TextXAlignment = Enum.TextXAlignment.Left
    iCollectProUI.fit(title, 11)
    title.ZIndex = 82
    title.Parent = bar
    local hitBtn = Instance.new("TextButton")
    hitBtn.AnchorPoint = Vector2.new(1, 0.5)
    -- HIT: big, bright and white-rimmed so it's the first thing you see (it only moves when tapped)
    hitBtn.Position = UDim2.new(1, -6, 0.5, 0)
    hitBtn.Size = UDim2.fromOffset(76, 40)
    hitBtn.AutoButtonColor = false
    hitBtn.BackgroundTransparency = 1
    hitBtn.BorderSizePixel = 0
    hitBtn.Text = ""
    hitBtn.ZIndex = 85
    hitBtn.Parent = bar
    -- the red face is its own frame: a UIGradient on the button itself tinted the letters red too,
    -- which is why HIT used to vanish into the button
    local hitFace = Instance.new("Frame")
    hitFace.Size = UDim2.fromScale(1, 1)
    hitFace.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    hitFace.BorderSizePixel = 0
    hitFace.ZIndex = 85
    hitFace.Parent = hitBtn
    corner(hitFace, 10)
    gradient(hitFace, Color3.fromRGB(255, 70, 90), Color3.fromRGB(185, 15, 45), 90)
    local hitStroke = stroke(hitFace, Color3.fromRGB(255, 255, 255), 2, 0)
    local hitLbl = Instance.new("TextLabel")
    hitLbl.Size = UDim2.new(1, -8, 1, 0)
    hitLbl.Position = UDim2.fromOffset(4, 0)
    hitLbl.BackgroundTransparency = 1
    hitLbl.Font = Enum.Font.GothamBlack
    hitLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    hitLbl.TextStrokeColor3 = Color3.fromRGB(60, 0, 15)
    hitLbl.TextStrokeTransparency = 0
    hitLbl.Text = "HIT"
    hitLbl.ZIndex = 86
    hitLbl.Parent = hitBtn
    -- RETURN / RETURNING / RESETTING shrink to fit the button
    iCollectProUI.fit(hitLbl, 22, 4)
    -- the hunt code sets hitBtn.Text; show it on the label instead
    hitBtn:GetPropertyChangedSignal("Text"):Connect(function()
        if hitBtn.Text ~= "" then hitLbl.Text = hitBtn.Text hitBtn.Text = "" end
    end)
    hitBtn.MouseEnter:Connect(function() tween(hitStroke, 0.14, { Color = Color3.fromRGB(255, 230, 120) }) end)
    hitBtn.MouseLeave:Connect(function() tween(hitStroke, 0.14, { Color = Color3.fromRGB(255, 255, 255) }) end)
    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, -94, 0, 20)
    track.Position = UDim2.fromOffset(6, 22)
    track.BackgroundColor3 = T.TRACK
    track.BorderSizePixel = 0
    track.ZIndex = 82
    track.Parent = bar
    corner(track, 8)
    stroke(track, T.STROKE, 1, 0.55)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.fromScale(0, 1)
    fill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    fill.BorderSizePixel = 0
    fill.ZIndex = 83
    fill.Parent = track
    corner(fill, 8)
    gradient(fill, T.FILL1, T.FILL2)
    local pct = Instance.new("TextLabel")
    pct.Size = UDim2.fromScale(1, 1)
    pct.BackgroundTransparency = 1
    pct.Font = Enum.Font.GothamBold
    pct.TextSize = 11
    pct.TextColor3 = T.TEXT
    pct.TextStrokeTransparency = 0.7
    pct.Text = ""
    pct.ZIndex = 84
    pct.Parent = track

    local lastIn, thieves = {}, {}
    local shown, hunting = nil, false
    local pendingHome, pendingUntil = nil, 0

    -- HIT: fly to the thief, hit them with the best tool you own (Bat / any Slap up close,
    -- otherwise a ranged item aimed at them) until their brainrot drops, then fly back to where you were
    local RANGED = {}
    for _, n in ipairs({ "Laser Cape", "Taser Gun", "Freeze Ray", "Web Slinger", "Paintball Gun", "Slingshot",
        "Snowball", "Jelly Gun", "Hunter Crossbow", "Candycane Bow", "Gravity Gun", "Tripple Plungers",
        "Zombie Blaster", "Lava Blaster", "Summer Soaker", "Candy Launcher", "Pumpkin Launcher" }) do RANGED[n] = true end
    local function pickWeapon()
        local ch, bp = player.Character, player:FindFirstChild("Backpack")
        local melee, ranged
        for _, cont in ipairs({ ch, bp }) do
            for _, t in ipairs(cont and cont:GetChildren() or {}) do
                if t:IsA("Tool") then
                    if t.Name == "Bat" or t.Name:find("Slap", 1, true) then melee = melee or t
                    elseif RANGED[t.Name] then ranged = ranged or t end
                end
            end
        end
        return melee or ranged, melee ~= nil
    end
    local mouse
    pcall(function() mouse = require(ReplicatedStorage.Packages.PlayerMouse) end)
    -- flight velocity toward dest that goes around whatever is in the way: a wall/podium/tree ahead
    -- -> climb over it; a roof above too -> slide to whichever side is open. Only solid parts count.
    local steerRp = RaycastParams.new()
    steerRp.FilterType = Enum.RaycastFilterType.Exclude
    steerRp.RespectCanCollide = true
    local function steer(hrp, dest, speed, ignore)
        local d = dest - hrp.Position
        local dist = d.Magnitude
        if dist < 0.05 then return Vector3.zero end
        local dir = d.Unit
        steerRp.FilterDescendantsInstances = { player.Character, ignore }
        local look = math.min(dist, 14)
        local o = hrp.Position
        local ahead = Workspace:Raycast(o, dir * look, steerRp)
            or Workspace:Raycast(o + Vector3.new(0, -2.2, 0), dir * look, steerRp)
        if not ahead then return dir * speed end
        local flat = Vector3.new(dir.X, 0, dir.Z)
        flat = flat.Magnitude > 0.01 and flat.Unit or Vector3.zero
        if not Workspace:Raycast(o, Vector3.new(0, 7, 0), steerRp) then
            return (Vector3.new(0, 1, 0) * 0.85 + flat * 0.3).Unit * speed
        end
        local side = flat:Cross(Vector3.new(0, 1, 0))
        if side.Magnitude < 0.01 then side = Vector3.new(1, 0, 0) end
        for _, s in ipairs({ side, -side }) do
            if not Workspace:Raycast(o, s * 8, steerRp) then return (s * 0.8 + flat * 0.2).Unit * speed end
        end
        return -flat * speed * 0.5
    end
    -- standing inside someone else's base while it's locked = boxed in by its lasers
    local function inLockedBase(hrp)
        local plots = Workspace:FindFirstChild("Plots")
        local mine = findMyBase()
        for _, plot in ipairs(plots and plots:GetChildren() or {}) do
            if plot ~= mine then
                local hb = iCollectProAur.hitbox(plot)
                if hb and iCollectProAur.inBox(hb, hrp.Position) and iCollectProUI.lockInfo(plot) then return true end
            end
        end
        return false
    end
    local function flyTo(hrp, getPos, stopDist, maxTime, onStep)
        local t0 = os.clock()
        while iCollectPro_ALIVE() and os.clock() - t0 < maxTime do
            local pos = getPos()
            if not pos or not hrp.Parent then return false end
            local d = pos - hrp.Position
            if onStep and onStep(d.Magnitude) then return true end
            if d.Magnitude <= stopDist then
                hrp.AssemblyLinearVelocity = Vector3.zero
                if not onStep then return true end
            else
                hrp.AssemblyLinearVelocity = steer(hrp, pos, math.clamp(d.Magnitude * 8, 30, 160))
            end
            RunService.Heartbeat:Wait()
        end
        return false
    end
    local function theirRoot(target)
        local st = target:GetAttribute("Stealing")
        if st == nil or st == false then return nil end
        return target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    end
    local function wear(hum, t)
        if t and t.Parent and t.Parent ~= player.Character then pcall(function() hum:EquipTool(t) end) end
    end
    -- admin panel / knock-back on the way: ragdolled, rocket force, platform-stood, anchored (jail),
    -- dead - or stuck (not getting any closer for 2.5s, e.g. caged)
    local function adminHit(ch, hum, hrp)
        if not hum.Parent or hum.Health <= 0 or not hrp.Parent then return true end
        local t = player:GetAttribute("RagdollEndTime")
        if type(t) == "number" and t > Workspace:GetServerTimeNow() then return true end
        if player:GetAttribute("Ragdoll") == true or hrp.Anchored then return true end
        -- a flying item can use the same physics as a rocket; with one held, rely on the stuck check
        local held = ch:FindFirstChildOfClass("Tool")
        if held and held == findFlyGear() then return false end
        return hum.PlatformStand or ch:FindFirstChildWhichIsA("VectorForce", true) ~= nil
    end
    -- one run at the thief: "done" (brainrot dropped), "hit" (admin-panelled / stuck), "timeout"
    local function chase(target, ch, hum, hrp, tool, isMelee, gear)
        local range = isMelee and 4 or 30
        local t0, lastSwing = os.clock(), 0
        local bestD, bestAt = math.huge, os.clock()
        -- the carpet goes on before we move at all, so the flight never lags back
        wear(hum, gear or tool)
        while iCollectPro_ALIVE() and os.clock() - t0 < 15 do
            local r = theirRoot(target)
            if not r then return "done" end
            if adminHit(ch, hum, hrp) then return "hit" end
            if inLockedBase(hrp) then return "locked" end
            local pos = r.Position + r.AssemblyLinearVelocity * 0.1
            local d = pos - hrp.Position
            local dist = d.Magnitude
            if dist < bestD - 3 or dist <= range + 4 then bestD, bestAt = dist, os.clock() end
            if os.clock() - bestAt > 2.5 then return "hit" end
            if dist > range + 8 then wear(hum, gear or tool) else wear(hum, tool) end
            if dist <= range then
                hrp.AssemblyLinearVelocity = Vector3.zero
            else
                hrp.AssemblyLinearVelocity = steer(hrp, pos, math.clamp(dist * 8, 30, 160), target.Character)
            end
            local flat = Vector3.new(r.Position.X, hrp.Position.Y, r.Position.Z)
            if (flat - hrp.Position).Magnitude > 0.1 then hrp.CFrame = CFrame.lookAt(hrp.Position, flat) end
            if mouse then pcall(function() mouse.Hit = CFrame.new(r.Position); mouse.Target = r end) end
            if dist <= range + 2 and os.clock() - lastSwing >= 0.12 then
                lastSwing = os.clock()
                pcall(function() tool:Activate() end)
            end
            RunService.Heartbeat:Wait()
        end
        return "timeout"
    end
    -- HIT: chase the thief with the carpet on, hit them until the brainrot drops, fly back home.
    -- Admin-panelled on the way -> Instant Reset, and go straight back at them (up to 3 times).
    local function hunt(target)
        local ch = player.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if not (hum and hrp) or player:GetAttribute("Stealing") then return end
        if not pickWeapon() then setStealStatus("NO BAT / SLAP / WEAPON", false) return end
        hunting = true
        hitBtn.Text = "HUNTING"
        local home = hrp.CFrame
        _G.iCollectPro_SemiTP_Busy = true
        local resets = 0
        while iCollectPro_ALIVE() do
            ch = player.Character
            hum = ch and ch:FindFirstChildOfClass("Humanoid")
            hrp = ch and ch:FindFirstChild("HumanoidRootPart")
            local tool, isMelee = pickWeapon()
            if not (hum and hrp and tool) then break end
            local result = chase(target, ch, hum, hrp, tool, isMelee, findFlyGear())
            if (result ~= "hit" and result ~= "locked") or resets >= 3 or not theirRoot(target) then break end
            resets = resets + 1
            hitBtn.Text = "RESETTING"
            setStealStatus((result == "locked" and "STUCK IN A LOCKED BASE" or "ADMIN PANELLED")
                .. " - RESETTING (" .. resets .. "/3)", false)
            local old = player.Character
            pcall(_G.iCollectPro_SemiTP_InstaReset)
            local t0 = os.clock()
            while player.Character == old and os.clock() - t0 < 8 do task.wait(0.1) end
            if player.Character == old then break end
            local nc = player.Character
            if not (nc and nc:WaitForChild("HumanoidRootPart", 6) and nc:WaitForChild("Humanoid", 6)) then break end
            task.wait(0.35)
            if not theirRoot(target) then break end
            setStealStatus("BACK ON " .. string.upper(target.Name), nil)
            hitBtn.Text = "HUNTING"
        end
        -- going back is the user's call: the button turns into RETURN for 3s, then the bar goes away
        pcall(function() player.Character:FindFirstChildOfClass("Humanoid"):UnequipTools() end)
        _G.iCollectPro_SemiTP_Busy = false
        hunting = false
        pendingHome, pendingUntil = home, os.clock() + 3
        hitBtn.Text = "RETURN"
    end
    local function goHome()
        local home = pendingHome
        pendingHome = nil
        local ch = player.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if not (home and hum and hrp) or player:GetAttribute("Stealing") then hitBtn.Text = "HIT" return end
        hunting = true
        hitBtn.Text = "RETURNING"
        _G.iCollectPro_SemiTP_Busy = true
        wear(hum, findFlyGear())
        pcall(flyTo, hrp, function() return home.Position end, 2.5, 8)
        pcall(function()
            hrp.CFrame = home
            hrp.AssemblyLinearVelocity = Vector3.zero
            hum:UnequipTools()
        end)
        _G.iCollectPro_SemiTP_Busy = false
        hunting = false
        hitBtn.Text = "HIT"
    end
    hitBtn.MouseButton1Click:Connect(function()
        -- the press/hover animation + sounds come from the shared UI hook (all our buttons)
        if hunting then return end
        if pendingHome then task.spawn(goHome) return end
        if shown then task.spawn(hunt, shown) end
    end)

    task.spawn(function()
        while iCollectPro_ALIVE() do
            local show = false
            local mine = iCollectProFx.thiefBar and findMyBase()
            local myHb = mine and iCollectProAur.hitbox(mine)
            if myHb then
                local now = os.clock()
                local plots = Workspace:FindFirstChild("Plots")
                for _, p in ipairs(Players:GetPlayers()) do
                    local r = p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                    if r then
                        if iCollectProAur.inBox(myHb, r.Position) then lastIn[p] = now end
                        local st = p:GetAttribute("Stealing")
                        local stealing = st ~= nil and st ~= false
                        if stealing and (thieves[p] or now - (lastIn[p] or -99) < 5) then
                            thieves[p] = true
                        elseif not stealing then
                            thieves[p] = nil
                        end
                        if thieves[p] and not show then
                            show = true
                            shown = p
                            local theirHb
                            for _, pl in ipairs(plots and plots:GetChildren() or {}) do
                                local o = plotOwnerName(pl)
                                if o == p.Name or o == p.DisplayName then theirHb = iCollectProAur.hitbox(pl) break end
                            end
                            local total = theirHb and (theirHb.Position - myHb.Position).Magnitude or 150
                            local done = (r.Position - myHb.Position).Magnitude
                            local frac = math.clamp(done / math.max(total, 1), 0, 1)
                            fill.Size = UDim2.fromScale(frac, 1)
                            pct.Text = math.floor(frac * 100) .. "% TO THEIR BASE"
                            local idx = p:GetAttribute("StealingIndex")
                            title.Text = p.Name .. " STOLE " .. ((idx ~= nil and idx ~= "") and string.upper(tostring(idx)) or "YOUR BRAINROT")
                            barStroke.Transparency = (now % 0.6 < 0.3) and 0.85 or 0.3
                        end
                    end
                end
            end
            if not show and not hunting then shown = nil end
            if pendingHome and not hunting and os.clock() > pendingUntil then
                pendingHome = nil
                hitBtn.Text = "HIT"
            end
            if pendingHome and not show and not hunting then
                title.Text = "TAP RETURN TO FLY BACK"
                pct.Text = ""
            end
            bar.Visible = show or hunting or pendingHome ~= nil
            barScale.Scale = iCollectProUI.uiFactor()
            task.wait(0.1)
        end
        bar:Destroy()
    end)
end)()

-- Kick at 2s: your base's lock timer at 2s or less kicks you from the server
;(function()
    local rejoined = false
    task.spawn(function()
        while iCollectPro_ALIVE() do
            if iCollectProFx.rejoin2 and not rejoined then
                local mine = findMyBase()
                -- not "mine and timerState(mine)": an `and` keeps only the first return, so n was always nil
                local text, n
                if mine then
                    local t, _, _, secs = iCollectProAur.timerState(mine)
                    text, n = t, secs
                end
                if text and n and n > 0 and n <= 2 and not text:find("RELOCK") then
                    rejoined = true
                    pcall(function()
                        player:Kick("⏱  AUTO KICK AT 2s  ⏱\n\n"
                            .. "Your base unlocks in " .. n .. "s\n"
                            .. "You left before anyone could get in.\n"
                            .. "discord.gg/Z7vPrxecnJ")
                    end)
                end
            elseif not iCollectProFx.rejoin2 then
                rejoined = false
            end
            task.wait(0.1)
        end
    end)
end)()

-- Walk / Giant / Carpet speed + Gravity (never while carrying: the server voids a brainrot above ~23)
;(function()
    local conn
    conn = RunService.Heartbeat:Connect(function(dt)
        if not iCollectPro_ALIVE() then conn:Disconnect() return end
        if not (iCollectProFx.speedOn or iCollectProFx.carpet or iCollectProFx.grav) then return end
        if _G.iCollectPro_SemiTP_Busy or _G.iCollectPro_SemiTP_ResetBusy then return end
        local ch = player.Character
        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if not (hum and hrp) or hum.Health <= 0 then return end
        local st = player:GetAttribute("Stealing")
        local stealing = st ~= nil and st ~= false
        local tool = ch:FindFirstChildOfClass("Tool")
        local fly = false
        if tool then
            local nm = tool.Name:lower()
            for _, k in ipairs(FLY_GEAR_KEYS) do if nm:find(k, 1, true) then fly = true break end end
        end
        local speed
        if not stealing then
            if iCollectProFx.carpet and fly then
                speed = iCollectProUI.num("carpetSpeedValue", 130, 16, 500)
            elseif iCollectProFx.speedOn and not fly then
                speed = player:GetAttribute("GiantPotion") ~= nil and iCollectProUI.num("giantSpeedValue", 30, 16, 60)
                    or iCollectProUI.num("walkSpeedValue", 28, 16, 80)
            end
        end
        local v = hrp.AssemblyLinearVelocity
        local vy = v.Y
        if iCollectProFx.grav and not fly and hum.FloorMaterial == Enum.Material.Air then
            vy = math.clamp(vy, -500, 500) - (iCollectProUI.num("gravityValue", 196.2, 0, 300) - 196.2) * dt
        end
        local md = hum.MoveDirection
        if speed and md.Magnitude > 0 then
            md = md.Unit
            hrp.AssemblyLinearVelocity = Vector3.new(md.X * speed, vy, md.Z * speed)
        elseif speed then
            -- keys let go: stop dead instead of gliding on (no slide on the carpet or on foot)
            hrp.AssemblyLinearVelocity = Vector3.new(0, vy, 0)
        elseif vy ~= v.Y then
            hrp.AssemblyLinearVelocity = Vector3.new(v.X, vy, v.Z)
        end
    end)
end)()

-- FPS Boost: shadows / reflections off, lowest quality, particles hidden - all restored when off
;(function()
    local Lighting = game:GetService("Lighting")
    local on, saved, parts, addConn = false, nil, {}, nil
    local FX = { ParticleEmitter = true, Trail = true, Smoke = true, Fire = true, Sparkles = true }
    local function hide(d)
        if FX[d.ClassName] and d.Enabled then
            parts[d] = true
            d.Enabled = false
        end
    end
    local function enable()
        saved = {
            shadows = Lighting.GlobalShadows,
            spec = Lighting.EnvironmentSpecularScale,
            diff = Lighting.EnvironmentDiffuseScale,
        }
        pcall(function() saved.quality = UserSettings():GetService("UserGameSettings").SavedQualityLevel end)
        Lighting.GlobalShadows = false
        Lighting.EnvironmentSpecularScale = 0
        Lighting.EnvironmentDiffuseScale = 0
        pcall(function() UserSettings():GetService("UserGameSettings").SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel1 end)
        for _, d in ipairs(Workspace:GetDescendants()) do pcall(hide, d) end
        addConn = Workspace.DescendantAdded:Connect(function(d) pcall(hide, d) end)
    end
    local function disable()
        if addConn then addConn:Disconnect() addConn = nil end
        if saved then
            Lighting.GlobalShadows = saved.shadows
            Lighting.EnvironmentSpecularScale = saved.spec
            Lighting.EnvironmentDiffuseScale = saved.diff
            if saved.quality then pcall(function() UserSettings():GetService("UserGameSettings").SavedQualityLevel = saved.quality end) end
        end
        for d in pairs(parts) do pcall(function() if d.Parent then d.Enabled = true end end) end
        table.clear(parts)
    end
    task.spawn(function()
        while iCollectPro_ALIVE() do
            if iCollectProFx.fps and not on then on = true pcall(enable)
            elseif not iCollectProFx.fps and on then on = false pcall(disable) end
            if on then Lighting.GlobalShadows = false end
            task.wait(1)
        end
        if on then pcall(disable) end
    end)
end)()

-- Pill Bar, same build as iCollectPro INSTA STEAL's bottom bar: dot, title, shimmering invite,
-- FPS and PING. Drawn at 520x46, scaled to the screen, fixed at the bottom centre (not draggable).
;(function()
    local W, H = 520, 46
    local bar = Instance.new("Frame")
    bar.Name = "PillBar"
    bar.AnchorPoint = Vector2.new(0.5, 1)
    bar.Size = UDim2.fromOffset(W, H)
    -- sits clear above Roblox's tool hotbar
    bar.Position = UDim2.new(0.5, 0, 1, -100)
    bar.BackgroundColor3 = Color3.fromRGB(18, 14, 28)
    bar.BorderSizePixel = 0
    bar.ZIndex = 60
    bar.Parent = ProgressGui
    local scale = Instance.new("UIScale")
    scale.Parent = bar
    local function rescale()
        local cam = Workspace.CurrentCamera
        local vp = cam and cam.ViewportSize or Vector2.new(1920, 1080)
        -- same factor as the rest of the UI, but never wider than the screen
        local s = math.min(iCollectProUI.uiFactor(), vp.X * 0.94 / W)
        scale.Scale = math.clamp(s, 0.4, 1.25)
    end
    rescale()
    if Workspace.CurrentCamera then Workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(rescale) end

    corner(bar, 13)
    stroke(bar, T.STROKE, 1, 0.18)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(28, 22, 44)),
        ColorSequenceKeypoint.new(0.35, Color3.fromRGB(22, 17, 34)),
        ColorSequenceKeypoint.new(0.72, Color3.fromRGB(16, 12, 26)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(23, 18, 36)),
    })
    g.Parent = bar

    local function make(class, props)
        local o = Instance.new(class)
        for k, v in pairs(props) do o[k] = v end
        o.Parent = bar
        return o
    end
    make("Frame", { Name = "Sheen", Position = UDim2.new(0, 14, 0, 3), Size = UDim2.new(1, -28, 0, 1),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255), BackgroundTransparency = 0.95, BorderSizePixel = 0, ZIndex = 63 })
    local dot = make("Frame", { AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.new(0, 22, 0.5, 0), Size = UDim2.fromOffset(9, 9),
        BackgroundColor3 = T.STROKE, BorderSizePixel = 0, ZIndex = 62 })
    corner(dot, 5)
    stroke(dot, Color3.fromRGB(190, 140, 255), 1, 0.35)
    local function label(x, y, w, h, text, size, font, col, align)
        local l = make("TextLabel", { BackgroundTransparency = 1, Position = UDim2.new(0, x, 0, y), Size = UDim2.new(0, w, h == 0 and 1 or 0, h),
            Font = font, Text = text, TextSize = size, TextColor3 = col, TextXAlignment = align or Enum.TextXAlignment.Left,
            TextYAlignment = Enum.TextYAlignment.Center, ZIndex = 62 })
        l.AutoLocalize = false
        return l
    end
    local title = label(38, 0, 130, 0, "PURPLE HUB", 15, Enum.Font.GothamBold, Color3.fromRGB(245, 246, 248))
    title.TextStrokeTransparency = 0.92
    local invite = label(186, 0, 182, 0, "discord.gg/Z7vPrxecnJ", 15, Enum.Font.GothamBlack, Color3.fromRGB(255, 255, 255), Enum.TextXAlignment.Center)
    invite.TextScaled = true
    local cap = Instance.new("UITextSizeConstraint")
    cap.MaxTextSize = 15
    cap.Parent = invite
    invite.TextStrokeColor3 = Color3.fromRGB(40, 14, 70)
    invite.TextStrokeTransparency = 0.4
    local shine = Instance.new("UIGradient")
    shine.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(140, 45, 210)),
        ColorSequenceKeypoint.new(0.35, Color3.fromRGB(200, 130, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.65, Color3.fromRGB(200, 130, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(140, 45, 210)),
    })
    shine.Parent = invite
    for _, x in ipairs({ 180, 374, 450 }) do
        make("Frame", { Position = UDim2.new(0, x, 0.5, -11), Size = UDim2.fromOffset(1, 22),
            BackgroundColor3 = Color3.fromRGB(110, 60, 170), BackgroundTransparency = 0.15, BorderSizePixel = 0, ZIndex = 62 })
    end
    label(388, 8, 50, 10, "FPS", 9.5, Enum.Font.GothamBold, T.DIM)
    local fps = label(388, 20, 52, 17, "0", 15, Enum.Font.GothamBold, Color3.fromRGB(190, 140, 255))
    label(462, 8, 50, 10, "PING", 9.5, Enum.Font.GothamBold, T.DIM)
    local ping = label(462, 20, 52, 17, "0ms", 15, Enum.Font.GothamBold, Color3.fromRGB(190, 140, 255))

    -- good / ok / bad, same colours as iCollectPro INSTA STEAL's bar
    local GOOD, OK, BAD = Color3.fromRGB(56, 214, 110), Color3.fromRGB(255, 165, 0), Color3.fromRGB(220, 60, 60)
    local frames, lastT, t0 = 0, os.clock(), os.clock()
    local conn
    conn = RunService.Heartbeat:Connect(function()
        if not iCollectPro_ALIVE() then conn:Disconnect() bar:Destroy() return end
        bar.Visible = iCollectProFx.pillBar
        if not iCollectProFx.pillBar then return end
        frames = frames + 1
        local now = os.clock()
        shine.Offset = Vector2.new((now - t0) * 0.35 % 2 - 1, 0)
        if now - lastT >= 1 then
            fps.Text = tostring(frames)
            fps.TextColor3 = frames >= 60 and GOOD or (frames >= 30 and OK or BAD)
            frames, lastT = 0, now
            local okP, p = pcall(function() return math.floor(player:GetNetworkPing() * 1000) end)
            if okP then
                ping.Text = p .. "ms"
                ping.TextColor3 = p < 80 and GOOD or (p < 150 and OK or BAD)
            end
        end
    end)
end)()
end)()

-- From Aurora, batch 4 (all off by default): Defender Bypass, Public Grab, Quick Grab, Custom FOV,
-- Extreme FPS, Hide AP Icon. Own function: the main chunk is at Luau's 200-register limit.
;(function()
    -- Defender Bypass: STEAL NOW flies in and starts holding as usual, then keeps the hold alive
    -- (re-pressing every ~2.4s, the game drops a hold after 2.6s) until the base owner leaves the
    -- server, and grabs that instant. Press STEAL NOW again to cancel; gives up after 2 minutes.
    function iCollectProUI.waitOwnerLeave(pod, ctx)
        local name = plotOwnerName(pod.plot)
        local owner
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Name == name or p.DisplayName == name then owner = p end
        end
        if not owner then return true end
        setStealStatus("WAITING FOR " .. string.upper(owner.Name) .. " TO LEAVE", nil)
        local left = false
        local conn = Players.PlayerRemoving:Connect(function(p) if p == owner then left = true end end)
        iCollectProUI.bypassWaiting = true
        local t0 = os.clock()
        while iCollectPro_ALIVE() and not left and iCollectProUI.bypassWaiting and iCollectProFx.defBypass and os.clock() - t0 < 120 do
            local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0 or not pod.prompt.Parent or not owner.Parent then
                left = not owner.Parent
                break
            end
            if tick() - (ctx.holdBeganAt or 0) >= 2.4 then
                for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
                ctx.holdBeganAt = tick()
                ctx.ragdollFireTime = ctx.holdBeganAt
            end
            task.wait()
        end
        conn:Disconnect()
        iCollectProUI.bypassWaiting = false
        if not left then setStealStatus("BYPASS CANCELLED", false) end
        return left
    end

    -- Public Grab = instant grab on foot (user's call 2026-09-30: "even if they walk up to the
    -- brainrot I want it instant grab"). No teleport. The game's steal is a 1.5s prompt hold timed by
    -- the server, so the hold is BANKED ahead: the nearest enemy podium gets its hold begun from 25
    -- studs out and kept alive (re-begun every 2.4s - the game drops a hold at ~2.6s), and the moment
    -- we're inside its 10-stud range with the hold 1.55s old, the grab fires. Logged the same day: a
    -- hold begun far away may not count, so it's begun once more on stepping into range - whichever
    -- lands first wins. Skipped while carrying, ragdolled, tool-blocked or during STEAL NOW / HIT.
    local function blocked()
        local t = player:GetAttribute("RagdollEndTime")
        return player:GetAttribute("BlockTools") == true or player:GetAttribute("Ragdoll") == true
            or (type(t) == "number" and t > Workspace:GetServerTimeNow())
    end
    local function nearestPrompt(hrp, maxD)
        local plots = Workspace:FindFirstChild("Plots")
        local best, bestD = nil, maxD
        for _, plot in ipairs(plots and plots:GetChildren() or {}) do
            local pods = isEnemyPlot(plot) and plot:FindFirstChild("AnimalPodiums")
            for _, pd in ipairs(pods and pods:GetChildren() or {}) do
                local spawn, prompt = podiumParts(pd)
                if spawn and prompt and prompt.Enabled then
                    local d = (spawn.Position - hrp.Position).Magnitude
                    if d <= bestD then best, bestD = prompt, d end
                end
            end
        end
        return best, bestD
    end
    local BANK, RANGE = 25, 9.5
    -- bank = the one hold we keep running (the server tracks one hold at a time)
    local bank = { prompt = nil, ctx = nil, inRange = false, lastFire = 0, fired = nil }
    task.spawn(function()
        while iCollectPro_ALIVE() do
            task.wait(0.03)
            local st = player:GetAttribute("Stealing")
            local stealing = st ~= nil and st ~= false
            if stealing and bank.fired then
                iCollectPro.log(string.format("PUBLIC GRAB: grabbed %.2fs after the grab fired  (hold %.2fs old, begun %s range)",
                    os.clock() - bank.fired.at, bank.fired.age, bank.fired.beganIn and "IN" or "OUT OF"))
                bank.fired = nil
            end
            local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if not (iCollectProFx.pubGrab and not iCollectProFx.quickGrab and hrp) or stealing or blocked()
                or iCollectPro.debounce or _G.iCollectPro_SemiTP_Busy then
                bank.prompt, bank.ctx = nil, nil
                continue
            end
            local prompt, d = nearestPrompt(hrp, BANK)
            local inR = prompt ~= nil and d <= RANGE
            if prompt ~= bank.prompt then
                -- a new nearest brainrot: begin its hold right away
                bank.prompt, bank.inRange, bank.beganIn = prompt, inR, inR
                bank.ctx = prompt and iCollectProHold.startStealHold(prompt) or nil
                if prompt and not bank.ctx and fireproximityprompt and inR then pcall(fireproximityprompt, prompt) end
            elseif prompt and bank.ctx then
                local ctx = bank.ctx
                local age = tick() - (ctx.holdBeganAt or 0)
                local need = (tonumber(prompt.HoldDuration) or 1.5) + 0.05
                -- in reach with a ripe hold: grab now (again every 0.15s until it lands)
                if inR and age >= need and age <= 2.5 and os.clock() - bank.lastFire >= 0.15 then
                    bank.lastFire = os.clock()
                    bank.fired = { at = os.clock(), age = age, beganIn = bank.beganIn }
                    for _, fn in ipairs(ctx.cb.trigger) do task.spawn(fn) end
                end
                -- just stepped into range, or the hold is about to be dropped: begin it again
                if (inR and not bank.inRange) or age >= 2.4 then
                    for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
                    ctx.holdBeganAt = tick()
                    bank.beganIn = inR
                end
                bank.inRange = inR
            end
        end
    end)

    -- Quick Grab (no movement): the server keeps ONE hold per player (the hold remote carries no
    -- podium), so while you're within 40 studs of an enemy podium a hold is kept banked - begun again
    -- whenever it would be over qgMaxAge by the time you walk into reach. The moment the podium under
    -- your feet / nearest one is within 12 studs and the hold is 1.55s+ old, the grab fires (every
    -- 0.1s until it lands). If a banked hold doesn't take, it's begun once more in range (1.55s).
    -- Logs "QUICK GRAB: grabbed ..." with the hold's age. Without getconnections it falls back to
    -- pressing the prompt every frame. If the brainrot
    -- gets knocked off you, the same podium is re-grabbed through its Triggered handler while the
    -- prompt still shows disabled (the server takes it ~6s before the prompt comes back).
    ;(function()
        local REACH, ENTRY_WINDOW, GATE, REGRAB_WINDOW = 12, 1.0, 0.06, 12
        -- ARM: bank a hold within this of any enemy podium; RANGE: where the grab should land
        local ARM, RANGE = 40, 10
        -- the oldest hold we trust (raised on its own if an older one is logged landing)
        iCollectPro.qgMaxAge = iCollectPro.qgMaxAge or 2.4
        local qgFired, qgBegan, fellBack = nil, "OUT OF", false
        local info = setmetatable({}, { __mode = "k" })
        local target, since, lastPress, holdPrompt = nil, 0, 0, nil
        local dropPrompt, dropAt, wasStealing = nil, 0, false
        local underSlot, underAt, cands, candAt = nil, 0, {}, 0
        local rp = RaycastParams.new()
        rp.FilterType = Enum.RaycastFilterType.Exclude

        -- Plots.<plot>.AnimalPodiums.<slot>.Base.Spawn.PromptAttachment.<prompt>
        local function infoOf(prompt)
            local i = info[prompt]
            if i then return i end
            local spawn = prompt.Parent and prompt.Parent.Parent
            local pod = spawn and spawn.Parent and spawn.Parent.Parent
            local plot = pod and pod.Parent and pod.Parent.Parent
            local root = plot and plot:FindFirstChild("MainRoot")
            if not (spawn and spawn:IsA("BasePart") and root) then return nil end
            i = { spawn = spawn, pod = pod, plot = plot, root = root, relZ = root.CFrame:PointToObjectSpace(spawn.Position).Z }
            info[prompt] = i
            return i
        end
        local function scan()
            table.clear(cands)
            local plots = Workspace:FindFirstChild("Plots")
            for _, plot in ipairs(plots and plots:GetChildren() or {}) do
                local pods = isEnemyPlot(plot) and plot:FindFirstChild("AnimalPodiums")
                for _, pd in ipairs(pods and pods:GetChildren() or {}) do
                    local _, prompt = podiumParts(pd)
                    if prompt then cands[#cands + 1] = prompt end
                end
            end
        end
        -- the podium right under your feet (looks through non-colliding parts)
        local function slotUnder(ch, hrp)
            rp.FilterDescendantsInstances = { ch }
            local hit = Workspace:Raycast(hrp.Position, Vector3.new(0, -7, 0), rp)
            local p = hit and hit.Instance
            while p and p ~= Workspace do
                if p.Parent and p.Parent.Name == "AnimalPodiums" then return p end
                p = p.Parent
            end
            return nil
        end
        local function press(p)
            lastPress = os.clock()
            pcall(fireproximityprompt, p, 0)
        end

        -- the enemy podium prompt under a spot in the world (players are looked through: after a
        -- body swap the swapper is standing on the spot we want to read)
        local spotRp = RaycastParams.new()
        spotRp.FilterType = Enum.RaycastFilterType.Exclude
        function iCollectPro.enemyPromptAt(pos)
            local skip = {}
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl.Character then skip[#skip + 1] = pl.Character end
            end
            spotRp.FilterDescendantsInstances = skip
            local hit = Workspace:Raycast(pos, Vector3.new(0, -7, 0), spotRp)
            local p = hit and hit.Instance
            while p and p ~= Workspace do
                if p.Parent and p.Parent.Name == "AnimalPodiums" then break end
                p = p.Parent
            end
            local plot = p and p ~= Workspace and p.Parent and p.Parent.Parent
            local _, prompt = podiumParts(p ~= Workspace and p or nil)
            if prompt and isEnemyPlot(plot) then return prompt end
            return nil
        end

        -- Anti Body Swap hands off here right after swapping back: the same grab as a 2nd / 3rd floor
        -- clone-in. `ctx` is a hold banked on the podium before we even swapped back, so it's already
        -- old enough to fire the moment we land; meanwhile the prompt is pressed every frame.
        -- Without a banked hold it just presses whatever enemy podium is under our feet.
        function iCollectPro.grabUnderFeet(secs, prompt, ctx)
            if prompt then
                pcall(function()
                    prompt.RequiresLineOfSight = false
                    prompt.MaxActivationDistance = math.huge
                end)
            end
            local need = prompt and (tonumber(prompt.HoldDuration) or 1.3) + 0.05 or 1.35
            local t0, lastFire, lastSig, anchor = os.clock(), 0, 0, nil
            while iCollectPro_ALIVE() and os.clock() - t0 < (secs or 2) do
                local st = player:GetAttribute("Stealing")
                if st ~= nil and st ~= false then return true end
                local ch = player.Character
                local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
                if not hrp then return false end
                -- walked off the podium: stop trying
                anchor = anchor or hrp.Position
                if (hrp.Position - anchor).Magnitude > 6 then return false end
                local p = prompt
                if not p then
                    local pod = slotUnder(ch, hrp)
                    local plot = pod and pod.Parent and pod.Parent.Parent
                    local _, pr = podiumParts(pod)
                    if pr and isEnemyPlot(plot) then p = pr end
                end
                if ctx then
                    local age = tick() - (ctx.holdBeganAt or 0)
                    -- the game drops a hold after ~2.6s: re-begin it before then
                    if age >= 2.4 then
                        for _, fn in ipairs(ctx.cb.hold) do task.spawn(fn) end
                        ctx.holdBeganAt = tick()
                    elseif age >= need and os.clock() - lastFire >= 0.15 then
                        lastFire = os.clock()
                        for _, fn in ipairs(ctx.cb.trigger) do task.spawn(fn) end
                    end
                end
                if p and p.Parent and p.Enabled and fireproximityprompt then
                    press(p)
                elseif p and p.Parent and firesignal and os.clock() - lastSig >= 0.1 then
                    -- a brainrot that was just dropped back shows its prompt disabled for a few
                    -- seconds; its Triggered handler still takes the grab (Quick Grab's re-grab trick)
                    lastSig = os.clock()
                    pcall(firesignal, p.Triggered, player)
                end
                RunService.Heartbeat:Wait()
            end
            return false
        end

        local conn
        conn = RunService.Heartbeat:Connect(function()
            if not iCollectPro_ALIVE() then conn:Disconnect() return end
            local st = player:GetAttribute("Stealing")
            local stealing = st ~= nil and st ~= false
            local now = os.clock()
            if stealing and not wasStealing then
                holdPrompt = target
                if qgFired then
                    local age = qgFired.age
                    iCollectPro.log(string.format("QUICK GRAB: grabbed %.2fs after the grab fired  (hold %.2fs old, begun %s range)",
                        now - qgFired.at, age, tostring(qgFired.began)))
                    -- a hold older than we thought still worked: bank it longer from now on
                    if age > iCollectPro.qgMaxAge then iCollectPro.qgMaxAge = math.min(age, 6) end
                end
                qgFired = nil
            end
            if not stealing and wasStealing and holdPrompt then dropPrompt, dropAt = holdPrompt, now end
            wasStealing = stealing
            if not iCollectProFx.quickGrab then target, qgFired = nil, nil return end
            -- iCollectPro.upperGrab: a 2nd/3rd floor STEAL NOW just cloned in and hands the grab to us
            if stealing or ((_G.iCollectPro_SemiTP_Busy or iCollectPro.debounce) and not iCollectPro.upperGrab) then target = nil return end
            local ch = player.Character
            local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
            if not hrp then return end

            -- re-grab what got knocked off you, while its prompt still reads disabled
            if dropPrompt and now - dropAt < REGRAB_WINDOW and dropPrompt.Parent and not dropPrompt.Enabled then
                local i = infoOf(dropPrompt)
                if i and (i.spawn.Position - hrp.Position).Magnitude <= REACH and firesignal then
                    pcall(firesignal, dropPrompt.Triggered, player)
                    return
                end
            elseif dropPrompt and (now - dropAt >= REGRAB_WINDOW or dropPrompt.Enabled) then
                dropPrompt = nil
            end

            if now - candAt > 1 then candAt = now scan() end
            if now - underAt > 0.05 then underAt = now underSlot = slotUnder(ch, hrp) end
            -- target = the podium under your feet, else the nearest one in reach;
            -- arm = the nearest enemy podium within ARM (whose hold we keep banked)
            local best, bestD, arm, armD = nil, REACH, nil, ARM
            for _, p in ipairs(cands) do
                local i = p.Parent and p.Enabled and infoOf(p)
                if i then
                    local d = (i.spawn.Position - hrp.Position).Magnitude
                    if d <= armD then arm, armD = p, d end
                    if (underSlot == nil or i.pod == underSlot) and d <= bestD then best, bestD = p, d end
                end
            end
            if best ~= target then target, since, fellBack, qgFired = best, now, false, nil end
            local cb = arm and iCollectProStealCallbacks(arm)
            local tcb = target and iCollectProStealCallbacks(target)
            if not (cb and tcb) then
                -- no getconnections: the old press-every-frame grab
                if target and fireproximityprompt and now - lastPress >= ((now - since < ENTRY_WINDOW) and 0 or GATE) then press(target) end
                return
            end
            local age = tick() - iCollectProHold.lastHold
            local need = (tonumber(target and target.HoldDuration) or 1.5) + 0.05
            local upper = iCollectPro.upperGrab -- STEAL NOW runs the hold then; we only fire
            -- 1) keep a hold banked so it's ripe the moment we reach a podium: if it would be too old
            -- by the time we're in reach (walking ETA), begin it again now
            if not upper then
                local eta = 0
                if bestD > RANGE or not target then
                    local sp = infoOf(target or arm)
                    local dir = sp and (sp.spawn.Position - hrp.Position) or Vector3.zero
                    local v = hrp.AssemblyLinearVelocity
                    local toward = dir.Magnitude > 0 and v:Dot(dir.Unit) or 0
                    eta = (((target and bestD) or armD) - RANGE) / math.max(toward, 8)
                end
                if age + eta > iCollectPro.qgMaxAge then
                    for _, fn in ipairs(cb.hold) do task.spawn(fn) end
                    age, qgBegan = 0, (bestD <= RANGE and target) and "IN" or "OUT OF"
                end
            end
            if not target then return end
            -- 2) in reach with a ripe hold: grab now, again every 0.1s until it lands
            if age >= need and age <= iCollectPro.qgMaxAge + 0.3 and now - lastPress >= 0.1 then
                lastPress = now
                if not qgFired then qgFired = { at = now, age = age, began = qgBegan } end
                for _, fn in ipairs(tcb.trigger) do task.spawn(fn) end
            end
            -- 3) fallback: the banked hold didn't take at arm's length (0.35s of grabs from well inside
            -- 10 studs) - begin it once more right here, grab lands 1.55s later
            if not upper and not fellBack and bestD <= 8.5 and qgFired and now - qgFired.at >= 0.35 then
                fellBack = true
                for _, fn in ipairs(tcb.hold) do task.spawn(fn) end
                qgBegan, qgFired = "IN (fallback)", nil
                iCollectPro.log("QUICK GRAB: banked hold didn't take - hold begun again in range")
            end
        end)
    end)()

    -- Custom FOV: field of view from the box, set every frame right after the camera scripts run
    -- (render only, nothing sent)
    local FOV_BIND = "iCollectProSemiTPFov"
    local fovWas = false
    pcall(RunService.UnbindFromRenderStep, RunService, FOV_BIND)
    RunService:BindToRenderStep(FOV_BIND, Enum.RenderPriority.Camera.Value + 1, function()
        if not iCollectPro_ALIVE() then pcall(RunService.UnbindFromRenderStep, RunService, FOV_BIND) return end
        local cam = Workspace.CurrentCamera
        if not cam then return end
        if iCollectProFx.fov then
            fovWas = true
            cam.FieldOfView = iCollectProUI.num("fovValue", 80, 30, 120)
        elseif fovWas then
            fovWas = false
            cam.FieldOfView = 70
        end
    end)

    -- Extreme FPS: on top of FPS Boost, strips textures (decals, mesh textures, surface looks),
    -- turns every part SmoothPlastic, kills post effects and stops other players' animations.
    -- Everything is remembered and put back when it's switched off.
    ;(function()
        local Lighting = game:GetService("Lighting")
        local saved, conns, on = {}, {}, false
        local POST = { BloomEffect = true, BlurEffect = true, SunRaysEffect = true, DepthOfFieldEffect = true, ColorCorrectionEffect = true, Atmosphere = false }
        local function keep(d, prop, value)
            local s = saved[d]
            if not s then s = {} saved[d] = s end
            if s[prop] == nil then s[prop] = d[prop] end
            d[prop] = value
        end
        -- our own visuals (tracer beams, the server ghost) are never stripped
        local function ours(d)
            local p = d
            for _ = 1, 3 do
                if not p then return false end
                if p.Name:sub(1, 11) == "iCollectPro" then return true end
                p = p.Parent
            end
            return false
        end
        local function strip(d)
            if ours(d) then return end
            if d:IsA("Decal") then keep(d, "Transparency", 1)
            elseif d:IsA("SurfaceAppearance") then
                local s = saved[d] or {}
                saved[d] = s
                if s.Parent == nil then s.Parent = d.Parent end
                d.Parent = nil
            elseif d:IsA("MeshPart") then
                if d.TextureID ~= "" then keep(d, "TextureID", "") end
                if d.Material ~= Enum.Material.SmoothPlastic then keep(d, "Material", Enum.Material.SmoothPlastic) end
            elseif d:IsA("SpecialMesh") then
                if d.TextureId ~= "" then keep(d, "TextureId", "") end
            elseif d:IsA("BasePart") then
                if d.Material ~= Enum.Material.SmoothPlastic then keep(d, "Material", Enum.Material.SmoothPlastic) end
            elseif d:IsA("ParticleEmitter") or d:IsA("Trail") or d:IsA("Beam") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") then
                if d.Enabled then keep(d, "Enabled", false) end
            elseif POST[d.ClassName] and d.Parent == Lighting then
                keep(d, "Enabled", false)
            end
        end
        local function quiet(char)
            local hum = char and char:WaitForChild("Humanoid", 5)
            local anim = hum and hum:WaitForChild("Animator", 5)
            if not anim then return end
            for _, tr in ipairs(anim:GetPlayingAnimationTracks()) do pcall(function() tr:Stop(0) end) end
            conns[#conns + 1] = anim.AnimationPlayed:Connect(function(tr)
                if on then pcall(function() tr:Stop(0) end) end
            end)
        end
        local function enable()
            keep(Lighting, "GlobalShadows", false)
            keep(Lighting, "EnvironmentDiffuseScale", 0)
            keep(Lighting, "EnvironmentSpecularScale", 0)
            pcall(function()
                local gs = UserSettings():GetService("UserGameSettings")
                saved.__quality = saved.__quality or gs.SavedQualityLevel
                gs.SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel1
            end)
            for _, d in ipairs(Lighting:GetChildren()) do pcall(strip, d) end
            local all = Workspace:GetDescendants()
            for i, d in ipairs(all) do
                pcall(strip, d)
                if i % 4000 == 0 then task.wait() end
            end
            conns[#conns + 1] = Workspace.DescendantAdded:Connect(function(d)
                if on then task.defer(function() pcall(strip, d) end) end
            end)
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player then
                    task.spawn(quiet, p.Character)
                    conns[#conns + 1] = p.CharacterAdded:Connect(function(c) if on then quiet(c) end end)
                end
            end
            conns[#conns + 1] = Players.PlayerAdded:Connect(function(p)
                conns[#conns + 1] = p.CharacterAdded:Connect(function(c) if on then quiet(c) end end)
            end)
        end
        local function disable()
            for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
            table.clear(conns)
            local q = saved.__quality
            saved.__quality = nil
            if q then pcall(function() UserSettings():GetService("UserGameSettings").SavedQualityLevel = q end) end
            for d, props in pairs(saved) do
                for prop, v in pairs(props) do
                    pcall(function() d[prop] = v end)
                end
            end
            table.clear(saved)
        end
        task.spawn(function()
            while iCollectPro_ALIVE() do
                if iCollectProFx.xfps and not on then on = true pcall(enable)
                elseif not iCollectProFx.xfps and on then on = false pcall(disable) end
                task.wait(1)
            end
            if on then on = false pcall(disable) end
        end)
    end)()

    -- Hide AP Icon: the admin-panel button in the top bar (TopbarStandard, icon 95529031547606)
    ;(function()
        local hidden = {}
        local function widgetOf(img)
            local g = img
            while g.Parent and g.Parent.Parent and g.Parent.Parent.Name ~= "Holders" do g = g.Parent end
            return (g.Parent and g.Parent.Parent and g.Parent.Parent.Name == "Holders") and g or nil
        end
        task.spawn(function()
            while iCollectPro_ALIVE() do
                local tb = player:FindFirstChild("PlayerGui") and player.PlayerGui:FindFirstChild("TopbarStandard")
                if iCollectProFx.hideAP and tb then
                    for _, d in ipairs(tb:GetDescendants()) do
                        if d.Name == "IconImage" and (d:IsA("ImageLabel") or d:IsA("ImageButton"))
                            and tostring(d.Image):find("95529031547606", 1, true) then
                            local w = widgetOf(d)
                            if w and w:IsA("GuiObject") then
                                if hidden[w] == nil then hidden[w] = w.Visible end
                                w.Visible = false
                            end
                        end
                    end
                elseif not iCollectProFx.hideAP and next(hidden) then
                    for w, v in pairs(hidden) do pcall(function() w.Visible = v end) end
                    table.clear(hidden)
                end
                task.wait(0.5)
            end
            for w, v in pairs(hidden) do pcall(function() w.Visible = v end) end
        end)
    end)()
end)()

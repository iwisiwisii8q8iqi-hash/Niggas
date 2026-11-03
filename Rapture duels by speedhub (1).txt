--[[
    RAPTURE DUEL - Script completo v8.0.61
    CAMBIOS v8.0.61:
    - Botones flotantes (Platform, Bat Bypass, Insta Reset, TP Bat) sin borde negro
    - Platform muestra fondo morado al activarse (ya visible sin el borde)
    - Bat Bypass auto-equipa el bat y quita cualquier otra herramienta en mano
    - Todo lo demás de v8.0.60 intacto
]]

print("deobf by discord.gg/speedhub")
print("deobf by discord.gg/speedhub")
print("deobf by discord.gg/speedhub")
print("deobf by discord.gg/speedhub")
print("deobf by discord.gg/speedhub")

skipIntroEnabled = true
if _G.RaptureDuelRunning then return end
_G.RaptureDuelRunning = true

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local HS = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local SoundService = game:GetService("SoundService")
local LP = Players.LocalPlayer
local camera = workspace.CurrentCamera

local _tick     = tick
local _clamp    = math.clamp
local _floor    = math.floor
local _abs      = math.abs
local _huge     = math.huge
local _sqrt     = math.sqrt
local _V3new    = Vector3.new
local _V3zero   = Vector3.zero
local _CFnew    = CFrame.new
local _CFlookAt = CFrame.lookAt

local _GetPlayersCached
do
    local cache, cacheTime = nil, 0
    _GetPlayersCached = function()
        local now = _tick()
        if cache and now - cacheTime < 0.03 then return cache end
        cache = Players:GetPlayers()
        cacheTime = now
        return cache
    end
end

local function waitForCharReady(char, timeout)
    timeout = timeout or 5
    local deadline = _tick() + timeout
    while (not char) or (not char.Parent)
          or (not char:FindFirstChild("HumanoidRootPart"))
          or (not char:FindFirstChildOfClass("Humanoid")) do
        if _tick() > deadline then return false end
        task.wait(0.05)
    end
    return true
end

NS = 60
CS = 29
LAGGER_SPEED = 15
LAGGER_CARRY_SPEED = 24.5
MEDUSA_COOLDOWN = 25
BAT_AIMBOT_SPEED = 58
BYPASS_AIMBOT_SPEED = 60
CONFIG_FILE = "RaptureDuel.json"
_isDraggingButton = false

do
    if isfile and isfile(CONFIG_FILE) then
        local ok, data = pcall(function() return HS:JSONDecode(readfile(CONFIG_FILE)) end)
        if ok and data and data.skipIntro ~= nil then
            skipIntroEnabled = data.skipIntro == true
        end
    end
end

local function runIntroIfNeeded()
    if skipIntroEnabled then return end

    local INTRO_GUI_NAME = "RaptureIntroV63"
    local BG_IMAGE = "rbxassetid://94388796312856"
    local SKIP_ASSET = "rbxassetid://110394624919809"
    local INTRO_SOUND = "rbxassetid://111775445806866"
    local SOUND_START_OFFSET = 14.0
    local TOTAL_DURATION = 7.0

    local LETTER_ASSETS = {
        "rbxassetid://101592737110933",
        "rbxassetid://88069001062145",
        "rbxassetid://85013165646264",
        "rbxassetid://128257320549476",
        "rbxassetid://114497576059150",
        "rbxassetid://101592737110933",
        "rbxassetid://89275219123832",
        "rbxassetid://94380601512253",
        "rbxassetid://114497576059150",
        "rbxassetid://89275219123832",
        "rbxassetid://123299205986740",
    }
    local LETTER_PX = { 75, 60, 60, 60, 60, 69, 60, 60, 60, 60, 60 }
    local LETTER_SPACING = 6
    local WORD_GAP = 18

    pcall(function()
        local old = CoreGui:FindFirstChild(INTRO_GUI_NAME)
        if old then old:Destroy() end
        local pg = LP:FindFirstChild("PlayerGui")
        if pg then
            local o = pg:FindFirstChild(INTRO_GUI_NAME)
            if o then o:Destroy() end
        end
    end)

    local introGui = Instance.new("ScreenGui")
    introGui.Name = INTRO_GUI_NAME
    introGui.ResetOnSpawn = false
    introGui.IgnoreGuiInset = true
    introGui.DisplayOrder = 999
    pcall(function()
        if syn and syn.protect_gui then syn.protect_gui(introGui) end
    end)
    if not pcall(function() introGui.Parent = CoreGui end) then
        introGui.Parent = LP:WaitForChild("PlayerGui")
    end

    local bg = Instance.new("ImageLabel")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    bg.BorderSizePixel = 0
    bg.Image = BG_IMAGE
    bg.ScaleType = Enum.ScaleType.Crop
    bg.ZIndex = 1
    bg.Parent = introGui

    local introSound = Instance.new("Sound")
    introSound.Name = "RaptureIntroSound"
    introSound.SoundId = INTRO_SOUND
    introSound.Volume = 1
    introSound.Looped = false
    introSound.PlaybackSpeed = 1
    introSound.Parent = SoundService

    local function startIntroSound()
        if not introSound then return end
        if not introSound.IsLoaded then
            local waited = 0
            while not introSound.IsLoaded and waited < 3 do
                task.wait(0.05)
                waited = waited + 0.05
            end
        end
        pcall(function()
            introSound.TimePosition = SOUND_START_OFFSET
        end)
        introSound:Play()
    end
    task.spawn(startIntroSound)

    local totalWidth = 0
    for i = 1, #LETTER_ASSETS do
        totalWidth = totalWidth + LETTER_PX[i]
        if i == 7 then totalWidth = totalWidth + WORD_GAP end
    end
    totalWidth = totalWidth + (#LETTER_ASSETS - 1) * LETTER_SPACING

    local letterHolder = Instance.new("Frame")
    letterHolder.AnchorPoint = Vector2.new(0.5, 0.5)
    letterHolder.Size = UDim2.new(0, totalWidth, 0, 75 * 1.3)
    letterHolder.Position = UDim2.new(0.5, 0, 0.5, -20)
    letterHolder.BackgroundTransparency = 1
    letterHolder.ZIndex = 4
    letterHolder.Parent = introGui

    local letters = {}
    local cursorX = 0

    for i = 1, #LETTER_ASSETS do
        local px = LETTER_PX[i]

        local glow = Instance.new("ImageLabel")
        glow.Name = "Glow_" .. i
        glow.AnchorPoint = Vector2.new(0, 0.5)
        glow.Size = UDim2.fromOffset(px, px)
        glow.Position = UDim2.new(0, cursorX, 0.5, 0)
        glow.BackgroundTransparency = 1
        glow.Image = LETTER_ASSETS[i]
        glow.ScaleType = Enum.ScaleType.Crop
        glow.ResampleMode = Enum.ResamplerMode.Pixelated
        glow.ImageColor3 = Color3.fromRGB(180, 220, 255)
        glow.ImageTransparency = 1
        glow.ZIndex = 3
        glow.Parent = letterHolder

        local letter = Instance.new("ImageLabel")
        letter.Name = "Letter_" .. i
        letter.AnchorPoint = Vector2.new(0, 0.5)
        letter.Size = UDim2.fromOffset(px, px)
        letter.Position = UDim2.new(0, cursorX, 0.5, 0)
        letter.BackgroundTransparency = 1
        letter.Image = LETTER_ASSETS[i]
        letter.ScaleType = Enum.ScaleType.Crop
        letter.ResampleMode = Enum.ResamplerMode.Pixelated
        letter.ImageColor3 = Color3.fromRGB(255, 255, 255)
        letter.ImageTransparency = 1
        letter.ZIndex = 4
        letter.Parent = letterHolder

        letters[i] = {
            obj = letter,
            glow = glow,
            baseX = cursorX,
            phase = (i - 1) * 0.45,
        }

        cursorX = cursorX + px + LETTER_SPACING
        if i == 7 then cursorX = cursorX + WORD_GAP end
    end

    local skipImg = Instance.new("ImageLabel")
    skipImg.AnchorPoint = Vector2.new(0.5, 1)
    skipImg.Size = UDim2.new(0, 200, 0, 50)
    skipImg.Position = UDim2.new(0.5, 0, 1, -30)
    skipImg.BackgroundTransparency = 1
    skipImg.Image = SKIP_ASSET
    skipImg.ScaleType = Enum.ScaleType.Fit
    skipImg.ResampleMode = Enum.ResamplerMode.Default
    skipImg.ZIndex = 5
    skipImg.ImageTransparency = 0
    skipImg.Parent = introGui

    task.spawn(function()
        task.wait(0.1)
        for i = 1, #letters do
            local data = letters[i]
            task.delay((i - 1) * 0.07, function()
                if data.obj and data.obj.Parent then
                    TS:Create(
                        data.obj,
                        TweenInfo.new(0.55, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                        { ImageTransparency = 0 }
                    ):Play()
                end
                if data.glow and data.glow.Parent then
                    TS:Create(
                        data.glow,
                        TweenInfo.new(0.55, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                        { ImageTransparency = 0.65 }
                    ):Play()
                end
            end)
        end
    end)

    local bobConnection
    local elapsed = 0
    local BOB_SPEED = 3.2
    local BOB_AMPLITUDE = 8

    bobConnection = RunService.RenderStepped:Connect(function(dt)
        if not letterHolder or not letterHolder.Parent then
            if bobConnection then bobConnection:Disconnect() end
            return
        end

        elapsed = elapsed + dt
        local t = elapsed * BOB_SPEED

        for i = 1, #letters do
            local data = letters[i]
            local obj = data.obj
            if obj and obj.Parent then
                local offsetY = math.sin(t + data.phase) * BOB_AMPLITUDE
                local newPos = UDim2.new(0, data.baseX, 0.5, offsetY)
                obj.Position = newPos
                if data.glow and data.glow.Parent then
                    data.glow.Position = newPos
                end
            end
        end
    end)

    local skipRequested = false

    local function stopIntroSound()
        if introSound then
            pcall(function()
                introSound:Stop()
                introSound:Destroy()
            end)
            introSound = nil
        end
    end

    local function closeIntro()
        if skipRequested then return end
        skipRequested = true
        if bobConnection then bobConnection:Disconnect() end
        stopIntroSound()
        if introGui then introGui:Destroy() end
    end

    local lastTapTime = 0
    local skipConn
    skipConn = UIS.InputBegan:Connect(function(input, gpe)
        if skipRequested then
            if skipConn then skipConn:Disconnect() end
            return
        end
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch
            or input.UserInputType == Enum.UserInputType.MouseButton2 then
            local currentTime = tick()
            if currentTime - lastTapTime <= 0.5 then
                closeIntro()
            else
                lastTapTime = currentTime
            end
        end
        if input.KeyCode == Enum.KeyCode.Space then
            closeIntro()
        end
    end)

    task.delay(TOTAL_DURATION, function()
        if not skipRequested then
            closeIntro()
        end
    end)

    while not skipRequested do
        task.wait(0.05)
    end
    if skipConn then pcall(function() skipConn:Disconnect() end) end
end

runIntroIfNeeded()

tpDownMode = tpDownMode or "Basic"
setTpDownVisual = nil
tpDownRefresh = nil

local function doTpDownBasic()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        root.CFrame = _CFnew(root.Position.X, -7, root.Position.Z) * CFrame.Angles(0, select(2, root.CFrame:ToEulerAnglesYXZ()), 0)
        root.Velocity = _V3zero
    end)
end

local function doTpDownBest()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root then return end

        local origin = root.Position + _V3new(0, 50, 0)

        local rp = RaycastParams.new()
        rp.FilterDescendantsInstances = { char }
        rp.FilterType = Enum.RaycastFilterType.Exclude
        rp.IgnoreWater = true
        rp.RespectCanCollide = true

        local direction = _V3new(0, -5000, 0)
        local result = workspace:Raycast(origin, direction, rp)

        if not result then
            local _, yRot, _ = root.CFrame:ToEulerAnglesYXZ()
            root.CFrame = _CFnew(root.Position.X, -7, root.Position.Z) * CFrame.Angles(0, yRot, 0)
            root.Velocity = _V3zero
            root.AssemblyLinearVelocity = _V3zero
            root.AssemblyAngularVelocity = _V3zero
            return
        end

        local hitPos = result.Position
        local hitNormal = result.Normal

        if hitNormal.Y < 0.7 then
            local origin2 = _V3new(origin.X, hitPos.Y - 0.5, origin.Z)
            local result2 = workspace:Raycast(origin2, direction, rp)
            if result2 and result2.Normal.Y > 0.7 then
                hitPos = result2.Position
            end
        end

        local hipHeight = (hum and hum.HipHeight) or 2
        local targetY = hitPos.Y + (hipHeight * 0.5) + 0.5

        local _, yRot, _ = root.CFrame:ToEulerAnglesYXZ()
        root.CFrame = _CFnew(root.Position.X, targetY, root.Position.Z) * CFrame.Angles(0, yRot, 0)

        root.Velocity = _V3zero
        root.AssemblyLinearVelocity = _V3zero
        root.AssemblyAngularVelocity = _V3zero
    end)
end

local function doTpDown()
    if tpDownMode == "Best" then
        doTpDownBest()
    else
        doTpDownBasic()
    end
end

soundButtonsEnabled = true
soundTabEnabled     = true
soundToggleEnabled  = true
soundArrowEnabled   = true

setSoundButtonsVisual = nil
setSoundTabVisual     = nil
setSoundToggleVisual  = nil
setSoundArrowVisual   = nil

local SFX_IDS = {
    arrow  = "rbxassetid://3398620867",
    toggle = "rbxassetid://6042053626",
    button = "rbxassetid://5852470908",
}

local _activeSounds = {
    button = {},
    toggle = {},
    arrow  = {},
}

local function _sfxClearList(list)
    for i = #list, 1, -1 do
        local s = list[i]
        if s then
            pcall(function()
                s:Stop()
                s:Destroy()
            end)
        end
        list[i] = nil
    end
end

function stopAllSoundsOfType(kind)
    local list = _activeSounds[kind]
    if not list then return end
    _sfxClearList(list)
end

function stopAllUISounds()
    for kind, list in pairs(_activeSounds) do
        _sfxClearList(list)
    end
end

local function _sfxPlay(kind, opts)
    if kind == "button" and soundButtonsEnabled == false then return end
    if kind == "toggle" and soundToggleEnabled == false then return end
    if kind == "arrow"  and (soundTabEnabled == false and soundArrowEnabled == false) then return end

    opts = opts or {}
    pcall(function()
        local id = SFX_IDS[kind]
        if not id then return end
        local s = Instance.new("Sound")
        s.Name = "RaptureUISound"
        s.SoundId = id
        s.Volume = opts.volume or 0.6
        s.PlaybackSpeed = opts.pitch or 1
        s.Parent = SoundService
        s:Play()

        local list = _activeSounds[kind]
        if list then
            table.insert(list, s)
            task.delay(2, function()
                for i = #list, 1, -1 do
                    if list[i] == s then
                        table.remove(list, i)
                        break
                    end
                end
                pcall(function() s:Stop() end)
                pcall(function() s:Destroy() end)
            end)
        else
            task.delay(2, function()
                pcall(function() s:Stop() end)
                pcall(function() s:Destroy() end)
            end)
        end
    end)
end

function playUISound(kind) _sfxPlay(kind) end
function playFloatingButtonSound() _sfxPlay("button", {volume = 0.5}) end
function playToggleSound()        _sfxPlay("toggle", {volume = 0.6}) end
function playArrowSound()         _sfxPlay("arrow",  {volume = 0.6}) end

function playArrowSoundRT()
    if soundArrowEnabled == false then return end
    _sfxPlay("arrow", {volume = 0.6})
end

function playTabSoundRT()
    if soundTabEnabled == false then return end
    _sfxPlay("arrow", {volume = 0.6})
end

_G.RapturePlayUISound = playUISound
_G.RapturePlayFloatingButtonSound = playFloatingButtonSound
_G.RapturePlayToggleSound = playToggleSound
_G.RapturePlayArrowSound = playArrowSound
_G.RaptureStopAllSoundsOfType = stopAllSoundsOfType
_G.RaptureStopAllUISounds = stopAllUISounds
_G.RaptureIsSoundEnabled = function()
    return soundButtonsEnabled or soundTabEnabled or soundToggleEnabled or soundArrowEnabled
end

local PURPLE_HOT       = Color3.fromRGB(160, 32, 240)
local PURPLE_MAGENTA   = Color3.fromRGB(180, 60, 255)
local PURPLE_LIGHT     = Color3.fromRGB(190, 120, 255)
local PURPLE_DARK      = Color3.fromRGB(80, 10, 130)
local PURPLE_DARKER    = Color3.fromRGB(45, 0, 70)
local WHITE_SHINE      = Color3.fromRGB(245, 235, 255)

local PASTEL_PINK  = PURPLE_LIGHT
local NEON_PINK    = PURPLE_HOT
local BLACK_BG     = Color3.fromRGB(0, 0, 0)
local BLACK_STROKE = Color3.fromRGB(0, 0, 0)
local INACTIVE_BG  = Color3.fromRGB(0, 0, 0)
local INACTIVE_TXT = Color3.fromRGB(255, 255, 255)
local ACTIVE_TXT   = Color3.fromRGB(255, 255, 255)
local GRAY_TRACK   = Color3.fromRGB(120, 120, 130)

local BTN_STROKE_THICKNESS = 2
local BTN_STROKE_TRANSPARENCY = 0
local BTN_BG_DARK = Color3.fromRGB(0, 0, 0)

local NEON_SWEEP_COLOR = PURPLE_LIGHT
local NEON_SWEEP_SHINE = WHITE_SHINE
local _neonSweepBtns = {}

local function addNeonShineSweep(parent, delay)
    if not parent or parent:FindFirstChild("NeonSweep") then return end
    parent.ClipsDescendants = true
    local sweep = Instance.new("Frame", parent)
    sweep.Name = "NeonSweep"
    sweep.Size = UDim2.new(0, 32, 2.2, 0)
    sweep.Position = UDim2.new(-0.35, 0, -0.6, 0)
    sweep.BackgroundColor3 = NEON_SWEEP_SHINE
    sweep.BackgroundTransparency = 0.4
    sweep.BorderSizePixel = 0
    sweep.Rotation = 22
    sweep.ZIndex = 250
    sweep.Active = false
    sweep.Selectable = false
    Instance.new("UICorner", sweep).CornerRadius = UDim.new(1, 0)
    local grad = Instance.new("UIGradient", sweep)
    grad.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0.00, 1),
        NumberSequenceKeypoint.new(0.50, 0.15),
        NumberSequenceKeypoint.new(1.00, 1),
    })
    task.spawn(function()
        while parent and parent.Parent do
            sweep.Position = UDim2.new(-0.35, 0, -0.6, 0)
            sweep.BackgroundTransparency = 0.4
            local tw = TS:Create(sweep, TweenInfo.new(1.7, Enum.EasingStyle.Linear), {
                Position = UDim2.new(1.25, 0, -0.6, 0),
                BackgroundTransparency = 0.95,
            })
            tw:Play()
            task.wait(delay or 2.4)
        end
    end)
end

local function registerNeonSweep(btn)
    if not btn then return end
    if btn:FindFirstChild("NeonSweep") then return end
    table.insert(_neonSweepBtns, btn)
    addNeonShineSweep(btn, 2.6)
end

_G._RaptureRegisterNeonSweep = registerNeonSweep

aimbotMode = "V1"
aimbotModeSetVisual = nil
antiRagSetVisual = nil

antiDieEnabled = antiDieEnabled or false
antiFlingEnabled = antiFlingEnabled or false
setAntiDieVisual = nil
setAntiFlingVisual = nil

bodyLockEnabled = false
bodyLockRange = 20
bodyLockRangeBox = nil
bodyLockSetVisual = nil

xrayBaseEnabled = false
xrayBrainrotEnabled = false
setXrayBaseVisual = nil
setXrayBrainrotVisual = nil

speedDisplayMode = speedDisplayMode or "Speed and Mode"
setSpeedDisplayVisual = nil
speedDisplayRefresh = nil

showSpeedLabel = showSpeedLabel or true
setShowSpeedVisual = nil

unwalkEnabled = unwalkEnabled or false
setUnwalkVisual = nil

infiniteJumpSetVisual = nil

-- ============================================================
-- PLATFORM SYSTEM
-- ============================================================
platformFloatingPos = nil
platformFloatingButton = nil
platformEnabled = false
platformSetVisual = nil
platformSecondsBox = nil

local _Platform = {
    part = nil,
    enabled = false,
    height = 5,
    leaveDistance = 7.5,
    fallCheckDelay = 2,
    isDisabling = false,
    autoDisableThread = nil,
    AUTO_DISABLE_TIME = 1.9,
    TP_DOWN_TRIGGER_DELAY = 0.2,
    TP_DOWN_DETECT_DROP = 8,
    TP_DOWN_DETECT_WINDOW = 0.4,
    lastRootY = nil,
    lastCheckTime = 0,
    tpDownDetectCooldown = 0,
    hbConn = nil,
    hbConn2 = nil,
}

local function platformCancelAutoDisable()
    if _Platform.autoDisableThread then
        pcall(task.cancel, _Platform.autoDisableThread)
        _Platform.autoDisableThread = nil
    end
end

local function platformDisable(triggerTpDown)
    local wasEnabled = _Platform.enabled
    _Platform.enabled = false
    _Platform.isDisabling = false
    platformEnabled = false
    if _Platform.part then
        pcall(function() _Platform.part:Destroy() end)
        _Platform.part = nil
    end
    if platformSetVisual then pcall(platformSetVisual, false) end
    if platformFloatingButton then
        local f = platformFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, false) end
    end
    if wasEnabled and triggerTpDown ~= false then
        task.delay(_Platform.TP_DOWN_TRIGGER_DELAY, function()
            pcall(doTpDown)
        end)
    end
end

local function platformCreate()
    if _Platform.part then pcall(function() _Platform.part:Destroy() end) end
    local part = Instance.new("Part")
    part.Name = "RaptureFollowPlatform"
    part.Size = Vector3.new(12, 1, 12)
    part.Anchored = true
    part.CanCollide = true
    part.Material = Enum.Material.SmoothPlastic
    part.Color = Color3.fromRGB(160, 32, 240)
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        part.CFrame = CFrame.new(root.Position - Vector3.new(0, _Platform.height, 0))
    end
    part.Parent = workspace
    _Platform.part = part
end

local function platformStartAutoDisable()
    platformCancelAutoDisable()
    _Platform.autoDisableThread = task.delay(_Platform.AUTO_DISABLE_TIME, function()
        _Platform.autoDisableThread = nil
        if _Platform.enabled then
            platformDisable(true)
        end
    end)
end

local function platformEnable()
    _Platform.enabled = true
    platformEnabled = true
    _Platform.isDisabling = false
    platformCreate()
    platformStartAutoDisable()
    if platformSetVisual then pcall(platformSetVisual, true) end
    if platformFloatingButton then
        local f = platformFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, true) end
    end
    if _G._RapturePlaySweep and platformFloatingButton then
        local f = platformFloatingButton:FindFirstChild("Frame")
        if f then pcall(_G._RapturePlaySweep, f) end
    end
end

local function platformToggle()
    if _Platform.enabled then
        platformCancelAutoDisable()
        platformDisable(true)
    else
        platformEnable()
    end
    task.defer(saveAllSettings)
    return _Platform.enabled
end

local function platformStartHeartbeat()
    if _Platform.hbConn then return end
    _Platform.hbConn = RunService.Heartbeat:Connect(function()
        if not _Platform.enabled or not _Platform.part or not _Platform.part.Parent then return end
        local character = LP.Character
        local root = character and character:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end

        _Platform.part.CFrame = CFrame.new(
            root.Position.X,
            _Platform.part.Position.Y,
            root.Position.Z
        )

        local horizontalDist = (Vector3.new(root.Position.X, 0, root.Position.Z) - Vector3.new(_Platform.part.Position.X, 0, _Platform.part.Position.Z)).Magnitude
        local verticalDiff = root.Position.Y - (_Platform.part.Position.Y + _Platform.part.Size.Y / 2)

        local isLeaving = horizontalDist > _Platform.leaveDistance
            or (humanoid:GetState() == Enum.HumanoidStateType.Freefall and verticalDiff < 1.5)
            or (verticalDiff < -1 and root.AssemblyLinearVelocity.Y < -5)

        if isLeaving and not _Platform.isDisabling then
            _Platform.isDisabling = true
            task.delay(_Platform.fallCheckDelay, function()
                if _Platform.enabled then
                    platformCancelAutoDisable()
                    platformDisable(true)
                end
            end)
        end
    end)
end

local function platformStartTPDownDetector()
    if _Platform.hbConn2 then return end
    _Platform.hbConn2 = RunService.Heartbeat:Connect(function()
        local character = LP.Character
        if not character then _Platform.lastRootY = nil return end
        local root = character:FindFirstChild("HumanoidRootPart")
        if not root then _Platform.lastRootY = nil return end

        local now = _tick()
        local curY = root.Position.Y

        if _Platform.lastRootY then
            local drop = _Platform.lastRootY - curY
            local elapsed = now - _Platform.lastCheckTime
            if drop >= _Platform.TP_DOWN_DETECT_DROP
               and elapsed <= _Platform.TP_DOWN_DETECT_WINDOW
               and (now - _Platform.tpDownDetectCooldown) > 0.5 then
                _Platform.tpDownDetectCooldown = now
                if _Platform.enabled then
                    platformCancelAutoDisable()
                    platformDisable(true)
                end
            end
        end
        _Platform.lastRootY = curY
        _Platform.lastCheckTime = now
    end)
end

LP.CharacterAdded:Connect(function()
    if _Platform.enabled then
        task.wait(1)
        if _Platform.enabled then
            platformCreate()
            platformStartAutoDisable()
        end
    end
end)

platformStartHeartbeat()
platformStartTPDownDetector()

_G.RapturePlatformToggle = platformToggle
_G.RapturePlatformDisable = platformDisable
_G.RapturePlatformIsEnabled = function() return _Platform.enabled end

spinEnabled = false
spinSpeed = 25
spinSpeedBox = nil
spinAutoBrainrotEnabled = false
spinAutoBrainrotSetVisual = nil
spinSetVisual = nil

local SpinState = {
    active = false,
    conn = nil,
    ctrlConn = nil,
    camConn = nil,
    dieConn = nil,
    healthConn = nil,
    maxHealthConn = nil,
}

local ANTI_DIE_CFG = {
    maxHealth       = 1000,
    healthThreshold = 1000,
    invincibility   = 0.5,
    autoRevive      = true,
    ragdollProtect  = true,
}

local antiDieState = {
    invincibleUntil = 0,
    lastHeal        = 0,
    lockedHealth    = 1000,
}

local function spinForceFullHealth(hum)
    if not hum or not hum.Parent then return end
    local target = antiDieState.lockedHealth
    pcall(function()
        if hum.MaxHealth ~= target then hum.MaxHealth = target end
        if hum.Health ~= target then hum.Health = target end
    end)
    antiDieState.invincibleUntil = _tick() + ANTI_DIE_CFG.invincibility    antiDieState.lastHeal = _tick()
end

local function spinPreventRagdoll(root, hum)
    if not ANTI_DIE_CFG.ragdollProtect then return end    local state = hum:GetState()
    if state == Enum.HumanoidStateType.Physics    or state == Enum.HumanoidStateType.Ragdoll
    or state == Enum.HumanoidStateType.FallingDown
    or state == Enum.HumanoidStateType.Dead then
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
            hum.PlatformStand = false
            hum.Sit = false
            hum.AutoRotate = false
        end)
        spinForceFullHealth(hum)
        if root then
            pcall(function() root.AssemblyAngularVelocity = Vector3.zero end)
        end
    end
end

local function spinAutoRevive(char)
    if not ANTI_DIE_CFG.autoRevive then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum then return end
    spinForceFullHealth(hum)
    pcall(function()
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        hum:ChangeState(Enum.HumanoidStateType.Running)
    end)
    if root then
        pcall(function()
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end)
    end
end

local function spinAttachHealthConn(char)
    if SpinState.healthConn then SpinState.healthConn:Disconnect(); SpinState.healthConn = nil end
    if SpinState.maxHealthConn then SpinState.maxHealthConn:Disconnect(); SpinState.maxHealthConn = nil end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then hum = char and char:WaitForChild("Humanoid", 3) end
    if not hum then return end
    SpinState.healthConn = hum:GetPropertyChangedSignal("Health"):Connect(function()
        if not SpinState.active then return end
        local target = antiDieState.lockedHealth
        if hum.Health ~= target then pcall(function() hum.Health = target end) end
        if hum.Health <= 0 then spinAutoRevive(char) end
    end)
    SpinState.maxHealthConn = hum:GetPropertyChangedSignal("MaxHealth"):Connect(function()
        if not SpinState.active then return end
        local target = antiDieState.lockedHealth
        if hum.MaxHealth ~= target then pcall(function() hum.MaxHealth = target end) end
    end)
    spinForceFullHealth(hum)
end

local function spinStartAntiDie()
    if SpinState.dieConn then SpinState.dieConn:Disconnect(); SpinState.dieConn = nil end
    SpinState.dieConn = RunService.Heartbeat:Connect(function()
        if not SpinState.active then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum then return end
        local target = antiDieState.lockedHealth
        if hum.MaxHealth ~= target then pcall(function() hum.MaxHealth = target end) end
        if hum.Health ~= target then pcall(function() hum.Health = target end) end
        pcall(function() hum.BreakJointsOnDeath = false end)
        if hum.Health <= 0 then spinAutoRevive(char) end
        spinPreventRagdoll(root, hum)
    end)
    if LP.Character then spinAttachHealthConn(LP.Character) end
end

local function spinStopAntiDie()
    if SpinState.dieConn then SpinState.dieConn:Disconnect(); SpinState.dieConn = nil end
    if SpinState.healthConn then SpinState.healthConn:Disconnect(); SpinState.healthConn = nil end
    if SpinState.maxHealthConn then SpinState.maxHealthConn:Disconnect(); SpinState.maxHealthConn = nil end
    antiDieState.invincibleUntil = 0
    antiDieState.lastHeal = 0
end

local function spinForceRotate(dt)
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end
    if hum.AutoRotate then hum.AutoRotate = false end
    pcall(function() hrp.AssemblyAngularVelocity = Vector3.zero end)
    local angle = math.rad(spinSpeed) * dt
    hrp.CFrame = hrp.CFrame * CFrame.Angles(0, angle, 0)
end

local function spinStartRotation()
    if SpinState.conn then SpinState.conn:Disconnect() end
    SpinState.conn = RunService.RenderStepped:Connect(spinForceRotate)
    if SpinState.ctrlConn then SpinState.ctrlConn:Disconnect() end
    SpinState.ctrlConn = RunService.RenderStepped:Connect(function()
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and hum.AutoRotate then hum.AutoRotate = false end
    end)
    if SpinState.camConn then SpinState.camConn:Disconnect() end
    SpinState.camConn = RunService.Heartbeat:Connect(function()
        if not SpinState.active then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and hum.AutoRotate then hum.AutoRotate = false end
    end)
end

local function spinStopRotation()
    if SpinState.conn then SpinState.conn:Disconnect(); SpinState.conn = nil end
    if SpinState.ctrlConn then SpinState.ctrlConn:Disconnect(); SpinState.ctrlConn = nil end
    if SpinState.camConn then SpinState.camConn:Disconnect(); SpinState.camConn = nil end
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.AutoRotate = true end
    end
end

local function spinActivate()
    if SpinState.active then return end
    SpinState.active = true
    spinStartRotation()
    spinStartAntiDie()
end

local function spinDeactivate()
    if not SpinState.active then return end
    SpinState.active = false
    spinStopRotation()
    spinStopAntiDie()
end

_G._RaptureSpinSet = function(state)
    state = state == true
    if state then spinActivate() else spinDeactivate() end
    if spinSetVisual then spinSetVisual(state) end
end

_G._RaptureSpinStart = function() spinActivate() end
_G._RaptureSpinStopSystem = function() spinDeactivate() end
_G._RaptureSpinIsActive = function() return SpinState.active end

local SpinAutoBrainrotConn = nil
local function spinAutoBrainrotStart()
    if SpinAutoBrainrotConn then return end
    SpinAutoBrainrotConn = RunService.Heartbeat:Connect(function()
        if not spinAutoBrainrotEnabled then
            if SpinState.brainrotMode then
                SpinState.brainrotMode = false
                SpinState.lastBrainrotState = false
                SpinState.manualOverride = false
            end
            return
        end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        local hasBrainrot = hum.WalkSpeed <= 30 and hum.WalkSpeed >= 20
        if hasBrainrot and not SpinState.lastBrainrotState then
            SpinState.brainrotMode = true
            SpinState.lastBrainrotState = true
            SpinState.manualOverride = false
            if not SpinState.active then spinActivate() end
        elseif not hasBrainrot and SpinState.lastBrainrotState then
            SpinState.brainrotMode = false
            SpinState.lastBrainrotState = false
            SpinState.manualOverride = false
            if not spinEnabled then spinDeactivate() end
        end
        if SpinState.brainrotMode and not SpinState.manualOverride and not SpinState.active then
            spinActivate()
        end
    end)
end

local function spinAutoBrainrotStop()
    if SpinAutoBrainrotConn then SpinAutoBrainrotConn:Disconnect(); SpinAutoBrainrotConn = nil end
    SpinState.brainrotMode = false
    SpinState.lastBrainrotState = false
    SpinState.manualOverride = false
end

_G._RaptureSpinAutoBrainrotSet = function(state)
    spinAutoBrainrotEnabled = state == true
    if spinAutoBrainrotEnabled then spinAutoBrainrotStart() else spinAutoBrainrotStop(); if not spinEnabled then spinDeactivate() end end
end

LP.CharacterAdded:Connect(function(char)
    if not SpinState.active then return end
    task.wait(0.3)
    spinAttachHealthConn(char)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = false
        spinForceFullHealth(hum)
    end
    spinStartAntiDie()
end)

_G.RaptureToolAssets = {}

stealMode = "V2"
local STEAL_MODE_LOCKED = "V2"
V2_STOP_PERCENT = 70
NORMAL_STOP_PERCENT = 90

backgroundIndex = 1
backgroundImages = {
    "109632847306308",
    "75252148958711",
    "98868387500938",
    "111509505485115",
    "116627189008926",
    "108531410885481",
    "125423918880160",
    "71792293290595",
    "119934726907659",
    "121229055542067",
    "91847549200081",
    "86767817748851",
    "125242341078692",
}

backgroundImageTransparency = 0
floatingButtonScale = 1
progressBarScale = 1
_floatingUIScales = {}

titleLbl = nil
miniBtn = nil
bestScripLabel = nil
raptureLinkLabel = nil
skipIntroToggleSet = nil

function getThemeColor() return PURPLE_LIGHT end

_G._RaptureHideMobileButtons = _G._RaptureHideMobileButtons or false
_G._RaptureMobileHideList    = _G._RaptureMobileHideList or {}

function _G._RaptureApplyMobileButtonsHidden()
    local master = _G._RaptureHideMobileButtons == true
    local list = _G._RaptureMobileHideList or {}

    local function included(key)
        if not master then return false end
        local v = list[key]
        if v == nil then return true end
        return v == true
    end

    local floatingGuis = {
        TpBatButton      = "tpBat",
        BatBypassButton  = "batBypass",
        InstaResetButton = "instaReset",
        PlatformButton   = "platform",
    }
    for guiName, key in pairs(floatingGuis) do
        local sg = CoreGui:FindFirstChild(guiName)
        if not sg then
            local pg = LP:FindFirstChildOfClass("PlayerGui")
            sg = pg and pg:FindFirstChild(guiName) or nil
        end
        if sg then
            sg.Enabled = not included(key)
        end
    end
    local panelGui = CoreGui:FindFirstChild("RaptureDuelMobilePanel")
    if not panelGui then
        local pg = LP:FindFirstChildOfClass("PlayerGui")
        panelGui = pg and pg:FindFirstChild("RaptureDuelMobilePanel") or nil
    end
    if panelGui then
        local container = panelGui:FindFirstChild("FloatingPanel")
        container = container and container:FindFirstChild("ButtonsContainer")
        if container then
            local map = {
                TpDown    = "tp",
                AutoBat   = "aimbot",
                AutoLeft  = "autoLeft",
                AutoRight = "autoRight",
                DropBR    = "drop",
                Carry     = "carry",
                Lagger1   = "lagger",
                Lagger2   = "laggerCarry",
            }
            for _, child in ipairs(container:GetChildren()) do
                if child:IsA("TextButton") then
                    local key = map[child.Name] or child.Name
                    child.Visible = not included(key)
                end
            end
        end
    end
end

_G._RaptureMobileHideIncluded = function(key)
    local master = _G._RaptureHideMobileButtons == true
    if not master then return false end
    local v = (_G._RaptureMobileHideList or {})[key]
    if v == nil then return true end
    return v == true
end

local _refreshHideListVisuals = nil
local _refreshSoundListVisuals = nil

customSoundsEnabled = false
setCustomSoundsVisual = nil

syncAfterHitEnabled = false
setSyncAfterHitVisual = nil

local _syncAfterHit = {
    active = false,
    lastTriggerTime = 0,
    lockUntil = 0,
    wasRagdolled = false,
    clearSince = 0,
    lastHitKind = nil,
    restoreToken = 0,
    conn = nil,
    guardConn = nil,
    selfMedusaUntil = 0,
    selfMedusaConns = {},
    userAutoGrabOff = false,
}

local SYNC_CFG = {
    bat = { reenable = 1.3, timer = 2.8 },
}
local SYNC_COOLDOWN = 1.2

local function _syncStrHasBat(n)
    if type(n) ~= "string" then return false end
    n = n:lower()
    return (n:find("bat") or n:find("morcego") or n:find("slap")) and true or false
end
local function _syncIsRagdolled(hum)
    if not hum then return false end
    local st = hum:GetState()
    if st == Enum.HumanoidStateType.Physics
       or st == Enum.HumanoidStateType.Ragdoll
       or st == Enum.HumanoidStateType.FallingDown then return true end
    local et = LP:GetAttribute("RagdollEndTime")
    if et and (et - workspace:GetServerTimeNow()) > 0 then return true end
    return false
end
local function _syncRagdollRemaining()
    local now = workspace:GetServerTimeNow()
    local best = 0
    local et = LP:GetAttribute("RagdollEndTime")
    if type(et) == "number" then
        local rem = et - now
        if rem > best then best = rem end
    end
    for _, obj in ipairs({ LP, LP.Character }) do
        if obj then
            local ok, attrs = pcall(function() return obj:GetAttributes() end)
            if ok and attrs then
                for k, v in pairs(attrs) do
                    local n = tostring(k):lower()
                    if type(v) == "number" and (n:find("ragdoll") or n:find("stun")) then
                        local rem = v - now
                        if rem > 0 and rem < 60 and rem > best then best = rem end
                        if v > 0 and v < 60 and v > best then best = v end
                    end
                end
            end
        end
    end
    return best
end
local function _syncToolNearby(matcher)
    local myChar = LP.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local root = plr.Character:FindFirstChild("HumanoidRootPart")
            local near = (not myRoot) or (not root) or (root.Position - myRoot.Position).Magnitude <= 20
            if near then
                for _, t in ipairs(plr.Character:GetChildren()) do
                    if t:IsA("Tool") and matcher(t.Name) then return true end
                end
            end
        end
    end
    return false
end
local function _syncHitDetected(hum)
    if not hum or not _syncIsRagdolled(hum) then return false end
    local batNear = _syncToolNearby(_syncStrHasBat)
    if not batNear then return false end
    if _syncRagdollRemaining() > 0 then return true end
    local st = hum:GetState()
    if st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll then return true end
    return false
end

local function _syncPauseAutoSteal()
    if CONFIG.AUTO_STEAL_ENABLED then
        CONFIG.AUTO_STEAL_ENABLED = false
        pcall(function() if stopAutoSteal then stopAutoSteal() end end)
        if setInstaGrab then pcall(setInstaGrab, false) end
        _syncAfterHit.userAutoGrabOff = false
    end
end

local function _syncResumeAutoSteal()
    _syncAfterHit.restoreToken = _syncAfterHit.restoreToken + 1
    if _syncAfterHit.userAutoGrabOff then return end
    if CONFIG.AUTO_STEAL_ENABLED then return end
    CONFIG.AUTO_STEAL_ENABLED = true
    pcall(function() if startAutoSteal then startAutoSteal() end end)
    if setInstaGrab then pcall(setInstaGrab, true) end
end

local function _syncStartSystems(char)
    if _syncAfterHit.conn then pcall(function() _syncAfterHit.conn:Disconnect() end) end
    _syncAfterHit.conn = nil
    _syncAfterHit.wasRagdolled = false
    _syncAfterHit.lastTriggerTime = 0
    _syncAfterHit.clearSince = 0
    _syncAfterHit.lockUntil = 0
    _syncAfterHit.conn = RunService.Heartbeat:Connect(function()
        if not syncAfterHitEnabled then
            _syncAfterHit.wasRagdolled = false
            _syncAfterHit.lockUntil = 0
            return
        end
        local hum = char and char.Parent and char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        local rag = _syncHitDetected(hum)
        local nowc = os.clock()
        if rag then
            _syncAfterHit.clearSince = 0
        else
            if _syncAfterHit.clearSince == 0 then _syncAfterHit.clearSince = nowc end
            if (nowc - _syncAfterHit.clearSince) < 0.6 then rag = _syncAfterHit.wasRagdolled end
        end
        if nowc < _syncAfterHit.lockUntil then
            rag = true
        elseif not rag then
            _syncAfterHit.lockUntil = 0
        end
        if rag and not _syncAfterHit.wasRagdolled and nowc >= _syncAfterHit.lockUntil
           and (nowc - _syncAfterHit.lastTriggerTime) >= SYNC_COOLDOWN then
            _syncAfterHit.lastTriggerTime = nowc
            local kind = "bat"
            local cfg = SYNC_CFG[kind]
            _syncAfterHit.lockUntil = nowc + math.max(cfg.timer or 0, cfg.reenable or 0) + 0.5
            _syncAfterHit.lastHitKind = kind
            _syncPauseAutoSteal()
            task.delay(cfg.reenable, function()
                if syncAfterHitEnabled then _syncResumeAutoSteal() end
            end)
        end
        _syncAfterHit.wasRagdolled = rag
    end)
end

local function _syncStopSystems()
    if _syncAfterHit.conn then pcall(function() _syncAfterHit.conn:Disconnect() end); _syncAfterHit.conn = nil end
    _syncAfterHit.wasRagdolled = false
    _syncAfterHit.lockUntil = 0
    _syncAfterHit.restoreToken = _syncAfterHit.restoreToken + 1
    if CONFIG.AUTO_STEAL_ENABLED == false then
        CONFIG.AUTO_STEAL_ENABLED = true
        pcall(function() if startAutoSteal then startAutoSteal() end end)
        if setInstaGrab then pcall(setInstaGrab, true) end
    end
end

LP.CharacterAdded:Connect(function(char)
    if syncAfterHitEnabled then
        task.wait(0.3)
        _syncStartSystems(char)
    end
end)
if LP.Character then
    task.spawn(function()
        task.wait(0.5)
        if syncAfterHitEnabled then _syncStartSystems(LP.Character) end
    end)
end

local PRINTED_SOUND_PACKS = {
    V1 = {
        toggle = { "rbxassetid://6042053626", "rbxassetid://3398620867", "rbxassetid://255881176", "rbxassetid://12222208" },
        tab    = { "rbxassetid://3398620867", "rbxassetid://6042053626", "rbxassetid://12222200" },
        pitch = 1, volume = 2,
    },
}
currentSoundPack = "V1"
local _soundCache = {}

local function getMenuSound(idList)
    local key = table.concat(idList, "|")
    local cached = _soundCache[key]
    if cached and cached.Parent then return cached end
    for _, id in ipairs(idList) do
        local ok, sound = pcall(function()
            local s = Instance.new("Sound")
            s.Name = "RaptureMenuSound"
            s.SoundId = id
            s.Parent = SoundService
            pcall(function() task.wait() end)
            return s
        end)
        if ok and sound then
            if sound.IsLoaded then _soundCache[key] = sound return sound end
            pcall(function() sound:Destroy() end)
        end
    end
    return nil
end

function playMenuSound(kind)
    if kind == "tab" then
        playArrowSound()
    else
        playToggleSound()
    end
end

do
    local SOUNDS = {
        Bat    = { id = "rbxassetid://5713085119", skip = 0.2 },
        Medusa = { id = "rbxassetid://3102797479", doublePlay = true, delay = 0.5 },
    }
    local SKIN_BAT_SOUNDS = {
        ["Bat Scythe"]  = "rbxassetid://140228522212884",
        ["Pool Noodle"] = "rbxassetid://140121604040434",
        ["Frying Pan"]  = "rbxassetid://81202220081219",
        ["Banhammer"]   = "rbxassetid://623904185",
    }
    local function ctMatchItem(inst)
        if not (inst:IsA("Tool") or inst:IsA("Model")) then return nil end
        local n = inst.Name:lower()
        if n:find("bat") or n:find("slap") then return { key = "Bat" } end
        if n:find("medusa") then return { key = "Medusa" } end
        return nil
    end
    local hooked = setmetatable({}, { __mode = "k" })
    local enabled = false
    local tracked = setmetatable({}, { __mode = "k" })
    local watchers = setmetatable({}, { __mode = "k" })
    local pendingHandle = setmetatable({}, { __mode = "k" })
    local teardown
    local function resolveBatSoundId(tool)
        local skinState = _G._RaptureBatSkinState
        local skinList = _G._RaptureBatSkinList
        if skinState and skinList then
            local idx = skinState.currentIndex
            local skin = skinList[idx]
            if skin and skin.label and SKIN_BAT_SOUNDS[skin.label] then
                return SKIN_BAT_SOUNDS[skin.label]
            end
        end
        return SOUNDS.Bat.id
    end
    local function playCustom(entry)
        local cs = entry.sound
        if not cs or not cs.Parent then return end
        local d = entry.data
        if entry.timer then pcall(task.cancel, entry.timer); entry.timer = nil end
        if d == SOUNDS.Bat then
            local newId = resolveBatSoundId(entry.tool)
            if newId and cs.SoundId ~= newId then cs.SoundId = newId end
        end
        pcall(function() cs:Stop(); cs.TimePosition = d.skip or 0; cs.Volume = 1; cs:Play() end)
        if d.doublePlay and d.delay then
            entry.timer = task.delay(d.delay, function()
                entry.timer = nil
                if not enabled then return end
                if cs and cs.Parent then
                    pcall(function() cs:Stop(); cs.TimePosition = d.skip or 0; cs.Volume = 1; cs:Play() end)
                end
            end)
        end
    end
    local function hookSound(entry, snd)
        if not snd:IsA("Sound") then return end
        if snd.Name == "CustomSound" then return end
        if hooked[snd] then return end
        if entry.muted[snd] ~= nil then return end
        entry.muted[snd] = snd.Volume
        hooked[snd] = true
        local function trigger()
            if not enabled then return end
            if snd.Playing or snd.TimePosition > 0 then
                pcall(function() snd.Volume = 0; snd:Stop() end)
                playCustom(entry)
            end
        end
        local c1 = snd:GetPropertyChangedSignal("Playing"):Connect(trigger)
        local c2 = snd:GetPropertyChangedSignal("TimePosition"):Connect(trigger)
        table.insert(entry.conns, c1)
        table.insert(entry.conns, c2)
        local c3
        c3 = snd.AncestryChanged:Connect(function(_, parent)
            if parent then return end
            pcall(function() c1:Disconnect() end)
            pcall(function() c2:Disconnect() end)
            pcall(function() c3:Disconnect() end)
            hooked[snd] = nil
            entry.muted[snd] = nil
            if entry.timer then pcall(task.cancel, entry.timer); entry.timer = nil end
        end)
        table.insert(entry.conns, c3)
        if snd.Playing then trigger() end
    end
    local function setupTool(tool)
        if tracked[tool] then return end
        local item = ctMatchItem(tool)
        if not item then return end
        local data = SOUNDS[item.key]
        if not data then return end
        local handle = tool:FindFirstChild("Handle")
        if not handle then
            if pendingHandle[tool] then return end
            local c
            c = tool.ChildAdded:Connect(function(child)
                if child.Name == "Handle" then
                    pcall(function() c:Disconnect() end)
                    pendingHandle[tool] = nil
                    if enabled then setupTool(tool) end
                end
            end)
            pendingHandle[tool] = c
            return
        end
        local old = handle:FindFirstChild("CustomSound")
        if old then pcall(function() old:Destroy() end) end
        local cs = Instance.new("Sound")
        cs.Name        = "CustomSound"
        local initialId = data.id
        if item.key == "Bat" then initialId = resolveBatSoundId(tool) end
        cs.SoundId     = initialId
        cs.Volume      = 1
        cs.Looped      = false
        cs.RollOffMode = Enum.RollOffMode.Inverse
        cs.MaxDistance = 1000
        cs.MinDistance = 1000
        cs.Parent      = handle
        local entry = { sound = cs, data = data, conns = {}, muted = {}, tool = tool }
        tracked[tool] = entry
        for _, d in ipairs(tool:GetDescendants()) do hookSound(entry, d) end
        table.insert(entry.conns, tool.DescendantAdded:Connect(function(d)
            if tracked[tool] then hookSound(entry, d) end
        end))
        table.insert(entry.conns, tool.AncestryChanged:Connect(function(_, parent)
            if not parent then teardown(tool) end
        end))
    end
    function teardown(tool)
        local entry = tracked[tool]
        if not entry then return end
        if entry.timer then pcall(task.cancel, entry.timer); entry.timer = nil end
        for _, conn in ipairs(entry.conns) do pcall(function() conn:Disconnect() end) end
        if entry.sound then pcall(function() entry.sound:Destroy() end) end
        for snd, vol in pairs(entry.muted) do
            hooked[snd] = nil
            if snd and snd.Parent then pcall(function() snd.Volume = vol end) end
        end
        tracked[tool] = nil
    end
    local function tryRegister(inst)
        if not enabled then return end
        if inst:IsA("Tool") and ctMatchItem(inst) then setupTool(inst) end
    end
    local function unwatch(container)
        local conns = watchers[container]
        if not conns then return end
        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        watchers[container] = nil
    end
    local function watch(container)
        if not container or watchers[container] then return end
        watchers[container] = { container.ChildAdded:Connect(tryRegister) }
        for _, child in ipairs(container:GetChildren()) do tryRegister(child) end
    end
    local function watchAll()
        watch(LP:FindFirstChildOfClass("Backpack"))
        watch(LP.Character)
    end
    LP.ChildAdded:Connect(function(child)
        if enabled and child:IsA("Backpack") then watch(child) end
    end)
    LP.CharacterAdded:Connect(function(char)
        for tool in pairs(tracked) do teardown(tool) end
        for _, c in pairs(pendingHandle) do pcall(function() c:Disconnect() end) end
        table.clear(pendingHandle)
        for container in pairs(watchers) do unwatch(container) end
        if not enabled then return end
        watch(char)
        task.defer(function() if enabled then watchAll() end end)
    end)
    _G.RaptureCustomSounds = {
        setEnabled = function(on)
            on = on and true or false
            if enabled == on then return end
            enabled = on
            if on then watchAll()
            else
                for tool in pairs(tracked) do teardown(tool) end
                for _, c in pairs(pendingHandle) do pcall(function() c:Disconnect() end) end
                table.clear(pendingHandle)
                for container in pairs(watchers) do unwatch(container) end
            end
            if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
        end,
        isEnabled = function() return enabled end,
    }
end

local _lastThemeUpdate = 0
local _lastThemeColor = nil

_G._RaptureDynamicSliders = _G._RaptureDynamicSliders or {}

function _RaptureApplyThemeToSliders()
    for _, entry in ipairs(_G._RaptureDynamicSliders) do
        if entry and entry.applyThemeColor then
            pcall(entry.applyThemeColor, PURPLE_LIGHT)
        end
    end
end

local function ensureShine(btn)
    local shine = btn:FindFirstChild("AnimatedShine")
    if not shine then
        shine = Instance.new("Frame")
        shine.Name = "AnimatedShine"
        shine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        shine.BackgroundTransparency = 0
        shine.BorderSizePixel = 0
        shine.AnchorPoint = Vector2.new(0.5, 0.5)
        shine.Size = UDim2.new(0.07, 0, 2.4, 0)
        shine.Position = UDim2.new(0.0, 0, 0.5, 0)
        shine.Rotation = 25
        shine.ZIndex = 14
        shine.Visible = false
        shine.Parent = btn
        local grad = Instance.new("UIGradient", shine)
        grad.Rotation = 90
        grad.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0.00, 1.00),
            NumberSequenceKeypoint.new(0.20, 0.95),
            NumberSequenceKeypoint.new(0.40, 0.75),
            NumberSequenceKeypoint.new(0.50, 0.00),
            NumberSequenceKeypoint.new(0.60, 0.30),
            NumberSequenceKeypoint.new(0.80, 0.85),
            NumberSequenceKeypoint.new(1.00, 1.00),
        })
        local corner = Instance.new("UICorner", shine)
        corner.CornerRadius = UDim.new(1, 0)
        local glow = Instance.new("Frame", shine)
        glow.Name = "Glow"
        glow.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        glow.BackgroundTransparency = 0.75
        glow.BorderSizePixel = 0
        glow.AnchorPoint = Vector2.new(0.5, 0.5)
        glow.Size = UDim2.new(2.8, 0, 1.6, 0)
        glow.Position = UDim2.new(0.5, 0, 0.5, 0)
        glow.ZIndex = 13
        glow.Parent = shine
        local glowCorner = Instance.new("UICorner", glow)
        glowCorner.CornerRadius = UDim.new(1, 0)
        local glowGrad = Instance.new("UIGradient", glow)
        glowGrad.Rotation = 90
        glowGrad.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0.00, 1.00),
            NumberSequenceKeypoint.new(0.50, 0.55),
            NumberSequenceKeypoint.new(1.00, 1.00),
        })
    end
    return shine
end

local function playShine(btn)
    if not btn then return end
    local shine = ensureShine(btn)
    if not shine then return end    btn:SetAttribute("_shineActive", true)
    if shine:GetAttribute("_shineTween") then
        pcall(function()
            local old = shine:GetAttribute("_shineTween")
            if old and old.Cancel then old:Cancel() end
        end)
    end
    if shine:GetAttribute("_fadeTween") then
        pcall(function()
            local old = shine:GetAttribute("_fadeTween")
            if old and old.Cancel then old:Cancel() end
        end)
    end
    local DURACION = 6.0
    local DUR_FADE_IN  = 0.45
    local DUR_FADE_OUT = 0.45
    local function runOnce()
        if not btn:GetAttribute("_shineActive") then
            shine.Visible = false
            return
        end
        if not shine or not shine.Parent then return end
        shine.Position = UDim2.new(-0.25, 0, 0.5, 0)
        shine.BackgroundTransparency = 1
        shine.Size = UDim2.new(0.07, 0, 2.4, 0)
        shine.Visible = true
        local fadeIn = TS:Create(shine,
            TweenInfo.new(DUR_FADE_IN, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
            { BackgroundTransparency = 0 }
        )
        shine:SetAttribute("_fadeTween", fadeIn)
        fadeIn:Play()
        local move = TS:Create(shine,
            TweenInfo.new(DURACION, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
            { Position = UDim2.new(1.25, 0, 0.5, 0) }
        )
        shine:SetAttribute("_shineTween", move)
        move:Play()
        move.Completed:Connect(function(state)
            if state ~= Enum.PlaybackState.Completed then return end
            if not btn:GetAttribute("_shineActive") then
                shine.Visible = false
                return
            end
            local fadeOut = TS:Create(shine,
                TweenInfo.new(DUR_FADE_OUT, Enum.EasingStyle.Sine, Enum.EasingDirection.In),
                { BackgroundTransparency = 1 }
            )
            shine:SetAttribute("_fadeTween", fadeOut)
            fadeOut:Play()
            fadeOut.Completed:Connect(function()
                if not btn:GetAttribute("_shineActive") then
                    shine.Visible = false
                    return
                end
                runOnce()
            end)
        end)
    end
    runOnce()
end

_G._RapturePlaySweep = playShine

local function applyNeonPinkStyle(btn, active)
    if not btn then return end
    local bgGrad = btn:FindFirstChild("BtnGrad")
    local shine  = btn:FindFirstChild("NeonShine")
    local glow   = btn:FindFirstChild("NeonGlow")
    local stroke = btn:FindFirstChildOfClass("UIStroke")
    if active then
        if not bgGrad then
            bgGrad = Instance.new("UIGradient")
            bgGrad.Name = "BtnGrad"
            bgGrad.Parent = btn
        end
        bgGrad.Rotation = 45
        bgGrad.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0.00, 0.05),
            NumberSequenceKeypoint.new(0.45, 0.00),
            NumberSequenceKeypoint.new(0.55, 0.00),
            NumberSequenceKeypoint.new(1.00, 0.05),
        })
        bgGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0.00, PURPLE_DARKER),
            ColorSequenceKeypoint.new(0.25, PURPLE_DARK),
            ColorSequenceKeypoint.new(0.50, PURPLE_MAGENTA),
            ColorSequenceKeypoint.new(0.75, PURPLE_LIGHT),
            ColorSequenceKeypoint.new(1.00, PURPLE_HOT),
        })
        btn.BackgroundColor3 = PURPLE_HOT
        if not shine then
            shine = Instance.new("Frame")
            shine.Name = "NeonShine"
            shine.BackgroundColor3 = WHITE_SHINE
            shine.BackgroundTransparency = 0.78
            shine.BorderSizePixel = 0
            shine.ZIndex = 8
            shine.AnchorPoint = Vector2.new(0.5, 0.5)
            shine.Size = UDim2.new(0.18, 0, 2.2, 0)
            shine.Position = UDim2.new(0.75, 0, 0.5, 0)
            shine.Rotation = 32
            shine.Parent = btn
            local shineGrad = Instance.new("UIGradient", shine)
            shineGrad.Rotation = 90
            shineGrad.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0.00, 1.00),
                NumberSequenceKeypoint.new(0.50, 0.60),
                NumberSequenceKeypoint.new(1.00, 1.00),
            })
            local shineCorner = Instance.new("UICorner", shine)
            shineCorner.CornerRadius = UDim.new(0, 999)
        else
            shine.Visible = true
        end
        if not glow then
            glow = Instance.new("ImageLabel")
            glow.Name = "NeonGlow"
            glow.BackgroundTransparency = 1
            glow.Image = "rbxassetid://5028857084"
            glow.ImageColor3 = PURPLE_LIGHT
            glow.ImageTransparency = 0.45
            glow.ScaleType = Enum.ScaleType.Slice
            glow.SliceCenter = Rect.new(24, 24, 276, 276)
            glow.Size = UDim2.new(1, 24, 1, 24)
            glow.Position = UDim2.new(0, -12, 0, -12)
            glow.ZIndex = 0
            glow.Parent = btn
        else
            glow.Visible = true
            glow.ImageColor3 = PURPLE_LIGHT
        end
    else
        if bgGrad then bgGrad:Destroy() end
        if shine then shine.Visible = false end
        if glow then glow.Visible = false end
        btn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        btn.BackgroundTransparency = 0
    end
    if not stroke then
        stroke = Instance.new("UIStroke")
        stroke.Name = "NeonStroke"
        stroke.Parent = btn
    end
    stroke.Color = Color3.fromRGB(0, 0, 0)
    stroke.Thickness = 0
    stroke.Transparency = 1
    local label = btn:FindFirstChild("TextLabel")
    if label then
        if active then
            label.TextColor3 = Color3.fromRGB(255, 255, 255)
            label.TextStrokeColor3 = PURPLE_DARKER
            label.TextStrokeTransparency = 0.35
        else
            label.TextColor3 = PASTEL_PINK
            label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            label.TextStrokeTransparency = 0.35
        end
    end
end

_G._RaptureApplyNeonPink = applyNeonPinkStyle

function updateAllUIThemeColors(color)
    color = color or PURPLE_LIGHT
    local now = _tick()
    if color == _lastThemeColor and now - _lastThemeUpdate < 0.05 then return end
    _lastThemeUpdate = now
    _lastThemeColor = color
    if progressFill then progressFill.BackgroundColor3 = NEON_PINK end
    if pbFrame then
        local border = pbFrame:FindFirstChildOfClass("UIStroke")
        if border then border.Color = BLACK_STROKE end
    end
    if speedLabel then speedLabel.TextColor3 = color end
    if bestScripLabel then bestScripLabel.TextColor3 = color end
    if raptureLinkLabel then raptureLinkLabel.TextColor3 = color end
    local char = LP.Character
    if char then
        local head = char:FindFirstChild("Head")
        if head then
            local bb = head:FindFirstChild("RaptureDuelSpeedIndicator")
            if bb then
                for _, child in ipairs(bb:GetChildren()) do
                    if child:IsA("TextLabel") or child:IsA("TextButton") then
                        child.TextColor3 = color
                    end
                end
            end
        end
    end
    if titleLbl then titleLbl.TextColor3 = color end
    if miniBtn then miniBtn.TextColor3 = color end
    if main and main:FindFirstChild("TopImage") then
        main.TopImage.ImageColor3 = color
    end
    for plr, billboard in pairs(espBillboardCache) do
        if billboard and billboard.Parent then
            local container = billboard:FindFirstChild("Frame")
            if container then
                local nameLbl = container:FindFirstChild("TextLabel")
                if nameLbl then nameLbl.TextColor3 = color end
            end
        end
    end
    if tpBatFloatingButton then
        local f = tpBatFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, batDesyncTpEnabled) end
    end
    if batBypassFloatingButton then
        local f = batBypassFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, batBypassEnabled) end
    end
    if instaResetFloatingButton then
        local f = instaResetFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, false) end
    end
    if platformFloatingButton then
        local f = platformFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, _Platform.enabled) end
    end
    for _, tab in ipairs(tabButtons or {}) do
        if tab:GetAttribute("IsActiveTab") then
            tab.TextColor3 = ACTIVE_TXT
            tab.BackgroundColor3 = color
        end
    end
    if MobilePanel then
        local container = MobilePanel:FindFirstChild("FloatingPanel")
        if container then
            local btnContainer = container:FindFirstChild("ButtonsContainer")
            if btnContainer then
                for _, btn in ipairs(btnContainer:GetChildren()) do
                    if btn:IsA("TextButton") then
                        applyNeonPinkStyle(btn, btn:GetAttribute("MobActive") == true)
                    end
                end
            end
        end
    end
    if lockBtn then lockBtn.TextColor3 = color end
end

speedMode = false
antiRagdollEnabled = false
antiRagdollVersion = "v1"
jumpEnabled = false
laggerToggled = false
laggerCarryToggled = false
medusaCounterEnabled = false
batCounterEnabled = false
autoLeftEnabled = false
autoRightEnabled = false
autoBatEnabled = false
dropMode = 2
removeAccessoriesEnabled = false
stretchEnabled = false
stretchFOV = 120
stretchValue = 0.7
uiLocked = true
uiScaleValue = 66
progressBarScaleValue = 55
espEnabled = false
antiLagEnabled = false
ragdollTimerEnabled = false

_G._RaptureKickWarningEnabled = false
kickWarningEnabled = false
setKickWarningVisual = nil
_G._RaptureKickWarning = nil

local espHighlightCache = {}
local espBillboardCache = {}
local espTracerCache = {}
local espGlowCache = {}
local espConn = nil

antiLagDescConn = nil
_antiLagStored = {}

savedProgressBarPos = nil
savedButtonPositions = {}
savedMobilePanelPos = nil
tpBatFloatingPos = nil
batBypassFloatingPos = nil
instaResetFloatingPos = nil
instaResetFloatingButton = nil

currentAnimPack = "Off"
tryardHeartbeatConn = nil
useCarrySystem = false
lastMoveDir = _V3zero

local function getMedusaHitTime() return _G._RaptureMedusaHitTime or 0 end
local function isMedusaHit() return (_tick() - getMedusaHitTime()) < 4 end
local function setMedusaHitTime() _G._RaptureMedusaHitTime = _tick() end

do    local _medusaWatcherConns = {}
    local function _clearMedusaWatcher()
        for _, c in ipairs(_medusaWatcherConns) do pcall(function() c:Disconnect() end) end
        _medusaWatcherConns = {}
    end
    local function _hookPart(part)
        if not part:IsA("BasePart") then return end
        local c = part:GetPropertyChangedSignal("Anchored"):Connect(function()
            if part.Anchored and part.Transparency == 1 then setMedusaHitTime() end
        end)
        table.insert(_medusaWatcherConns, c)
    end
    local function _attachMedusaWatcher(char)
        _clearMedusaWatcher()
        _G._RaptureMedusaHitTime = 0
        if not char then return end
        for _, part in ipairs(char:GetDescendants()) do _hookPart(part) end
        table.insert(_medusaWatcherConns, char.DescendantAdded:Connect(function(part) _hookPart(part) end))
    end
    if LP.Character then _attachMedusaWatcher(LP.Character) end
    LP.CharacterAdded:Connect(function(char) waitForCharReady(char, 10); _attachMedusaWatcher(char) end)
end

_G._RaptureRCD = _G._RaptureRCD or {conn=nil, charConn=nil, endTime=0, seconds=2.6}
ragdollCountdownLabel = nil
ragdollCountdownEnabled = false

function hookRagdollCountdown(char)
    if _G._RaptureRCD.conn then pcall(function() _G._RaptureRCD.conn:Disconnect() end); _G._RaptureRCD.conn = nil end
    if _G._RaptureRCD.charConn then pcall(function() _G._RaptureRCD.charConn:Disconnect() end); _G._RaptureRCD.charConn = nil end
    if not ragdollCountdownEnabled then return end
    local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 4)
    if not hum then return end
    local function beginCountdown()
        local duration = isMedusaHit() and 3.6 or _G._RaptureRCD.seconds
        _G._RaptureRCD.endTime = _tick() + duration
        if ragdollCountdownLabel then ragdollCountdownLabel.Visible = true end
    end
    local function isRagdollStateForCountdown()
        local st = hum:GetState()
        return hum.PlatformStand or st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll
    end
    _G._RaptureRCD.charConn = hum.StateChanged:Connect(function(_, newState)
        if newState == Enum.HumanoidStateType.Physics or newState == Enum.HumanoidStateType.Ragdoll then beginCountdown() end
    end)
    _G._RaptureRCD.conn = RunService.RenderStepped:Connect(function()
        if not ragdollCountdownEnabled then
            if _G._RaptureRCD.conn then pcall(function() _G._RaptureRCD.conn:Disconnect() end); _G._RaptureRCD.conn = nil end
            if ragdollCountdownLabel then ragdollCountdownLabel.Visible = false; ragdollCountdownLabel.Text = "" end
            return
        end
        if not ragdollCountdownLabel or not ragdollCountdownLabel.Parent then return end
        if isRagdollStateForCountdown() and _G._RaptureRCD.endTime < _tick() then beginCountdown() end
        local left = math.max(0, _G._RaptureRCD.endTime - _tick())
        if left > 0 then
            ragdollCountdownLabel.Visible = true
            ragdollCountdownLabel.Text = string.format("RAGDOLL %.1f", left)
            ragdollCountdownLabel.TextColor3 = left <= 1 and Color3.fromRGB(255, 230, 90) or getThemeColor()
        else
            ragdollCountdownLabel.Visible = false
            ragdollCountdownLabel.Text = ""
        end
    end)
end

local _antiFlingState = { connection = nil, threshold = 80, spinThreshold = 40 }
local _ANTI_FLING_DRIVE_GRACE = 0.3

local function _afHubSpeedLive()
    local live = _G.__RaptureLiveSpeed
    if type(live) ~= "table" then return false end
    local stamp, speed = tonumber(live.t), tonumber(live.v)
    if not stamp or not speed or speed <= 0 then return false end
    return (os.clock() - stamp) <= _ANTI_FLING_DRIVE_GRACE
end

local function _afOwnMoverActive()
    if autoBatEnabled then return true end
    if autoLeftEnabled or autoRightEnabled then return true end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then return true end
    if batDesyncTpEnabled then return true end
    return false
end

function startAntiFling()
    if _antiFlingState.connection then return end
    antiFlingEnabled = true
    _antiFlingState.connection = RunService.Heartbeat:Connect(function()
        if not antiFlingEnabled then return end
        local character = LP.Character
        if not character then return end
        local root = character:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid and (humanoid.Health <= 0 or humanoid.SeatPart) then return end
        if dropActive or _G.IsDropping then return end
        if _afOwnMoverActive() then return end
        if _afHubSpeedLive() then return end
        local velocity = root.AssemblyLinearVelocity
        local flat = Vector3.new(velocity.X, 0, velocity.Z)
        if flat.Magnitude > _antiFlingState.threshold then
            pcall(function()
                root.AssemblyLinearVelocity = Vector3.new(0, velocity.Y, 0)
                root.AssemblyAngularVelocity = Vector3.zero
            end)
            return
        end
        if root.AssemblyAngularVelocity.Magnitude > _antiFlingState.spinThreshold then
            pcall(function() root.AssemblyAngularVelocity = Vector3.zero end)
        end
    end)
end

function stopAntiFling()
    antiFlingEnabled = false
    if _antiFlingState.connection then
        _antiFlingState.connection:Disconnect()
        _antiFlingState.connection = nil
    end
end

LP.CharacterAdded:Connect(function()
    task.wait(0.3)
    if antiFlingEnabled then startAntiFling() end
end)

_antiDieEnabled = false
_antiDieStopped = false
_antiDieSources = { toggle = false }

_antiDie = {
    enabled = false, loop = nil, healthConn = nil, charConn = nil,
    lastHealTime = 0, invincibleUntil = 0,
    config = {
        healthThreshold = 50, invincibilityFrames = 0.75,
        fallDamageProtection = false, ragdollProtection = true, autoRevive = true,
    },
}

function _antiDie.SuperHeal(hum)
    if not hum or not hum.Parent then return end
    local maxHealth = hum.MaxHealth or 100
    if maxHealth <= 0 or maxHealth == math.huge then maxHealth = 100 end
    pcall(function()
        hum.Health = maxHealth
        if hum.MaxHealth < maxHealth then hum.MaxHealth = maxHealth end
    end)
    _antiDie.invincibleUntil = _tick() + _antiDie.config.invincibilityFrames
    _antiDie.lastHealTime = _tick()
end

function _antiDie.PreventDamage(root, hum)
    if not hum then return end
    if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
    if _tick() < _antiDie.invincibleUntil then
        if hum.Health < (hum.MaxHealth or 100) then hum.Health = hum.MaxHealth or 100 end
    end
    if _antiDie.config.ragdollProtection then
        local state = hum:GetState()
        if state == Enum.HumanoidStateType.Physics or
           state == Enum.HumanoidStateType.Ragdoll or
           state == Enum.HumanoidStateType.FallingDown or
           state == Enum.HumanoidStateType.Dead then
            pcall(function()
                hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end)
            _antiDie.SuperHeal(hum)
            if root then pcall(function() root.AssemblyAngularVelocity = Vector3.zero end) end
        end
    end
    if hum.Health <= 0 then
        _antiDie.SuperHeal(hum)
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
        if root then
            pcall(function()
                root.CFrame = CFrame.new(root.Position + Vector3.new(0, 2, 0))
                root.AssemblyLinearVelocity = Vector3.zero
            end)
        end
    end
end

function _antiDie.AutoRevive()
    if not _antiDie.config.autoRevive then return end
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum then return end
    if hum.Health <= 0 then
        _antiDie.SuperHeal(hum)
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
        if root then
            pcall(function()
                root.CFrame = CFrame.new(root.Position + Vector3.new(0, 3, 0))
                root.AssemblyLinearVelocity = Vector3.zero
            end)
        end
    end
end

function _antiDie.AttachHealth(char)
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then hum = char and char:WaitForChild("Humanoid", 3) end
    if not hum then return end
    _antiDie.healthConn = hum:GetPropertyChangedSignal("Health"):Connect(function()
        if not _antiDie.enabled then return end
        if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
        if hum.Health <= 0 then _antiDie.AutoRevive() end
    end)
    if hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
end

function _antiDie.StartEngine()
    if _antiDie.enabled and _antiDie.loop then return end
    _antiDie.enabled = true
    _antiDieEnabled = true
    antiDieEnabled = true
    if _antiDie.loop then _antiDie.loop:Disconnect(); _antiDie.loop = nil end
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    _antiDie.loop = RunService.Heartbeat:Connect(function()
        if not _antiDie.enabled then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum then return end
        if hum.Health <= 0 then _antiDie.AutoRevive()
        elseif hum.Health <= _antiDie.config.healthThreshold then _antiDie.SuperHeal(hum)
        elseif hum.Health < (hum.MaxHealth or 100) then _antiDie.SuperHeal(hum) end
        _antiDie.PreventDamage(root, hum)
    end)
    if LP.Character then _antiDie.AttachHealth(LP.Character) end
    if _antiDie.charConn then _antiDie.charConn:Disconnect(); _antiDie.charConn = nil end
    _antiDie.charConn = LP.CharacterAdded:Connect(function(char)
        if not _antiDie.enabled then return end
        task.wait(0.05)
        _antiDie.AttachHealth(char)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then _antiDie.SuperHeal(hum) end
    end)
end

function _antiDie.StopEngine()
    _antiDie.enabled = false
    _antiDieEnabled = false
    antiDieEnabled = false
    if _antiDie.loop then _antiDie.loop:Disconnect(); _antiDie.loop = nil end
    if _antiDie.healthConn then _antiDie.healthConn:Disconnect(); _antiDie.healthConn = nil end
    if _antiDie.charConn then _antiDie.charConn:Disconnect(); _antiDie.charConn = nil end
end

function _antiDieSetEnabled(enabled)
    enabled = enabled == true
    if enabled then _antiDie.StartEngine() else _antiDie.StopEngine() end
end

_G.AntiDie = {
    start = function() _antiDieStopped = false; _antiDieSetEnabled(true) end,
    stop = function() _antiDieSetEnabled(false) end,
}

local InfiniteJump = { enabled = false, jumpPower = 55, minVelocity = 30, jumpConn = nil, heartbeatConn = nil }
local function applyJump(root)
    if not root then return end
    pcall(function() root.Velocity = _V3new(root.Velocity.X, InfiniteJump.jumpPower, root.Velocity.Z) end)
end
local function onJumpRequest()
    if not InfiniteJump.enabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then applyJump(root) end
end
local function onHeartbeat()
    if not InfiniteJump.enabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local jumpHeld = UIS:IsKeyDown(Enum.KeyCode.Space) or (hum.Jump == true)
    if jumpHeld and root.Velocity.Y < InfiniteJump.minVelocity then applyJump(root) end
end
local function connectEvents()
    if InfiniteJump.jumpConn then InfiniteJump.jumpConn:Disconnect() end
    if InfiniteJump.heartbeatConn then InfiniteJump.heartbeatConn:Disconnect() end
    InfiniteJump.jumpConn = UIS.JumpRequest:Connect(onJumpRequest)
    InfiniteJump.heartbeatConn = RunService.Heartbeat:Connect(onHeartbeat)
end
function InfiniteJump.start()
    if InfiniteJump.enabled then return end
    InfiniteJump.enabled = true
    connectEvents()
end
function InfiniteJump.stop()
    InfiniteJump.enabled = false
    if InfiniteJump.jumpConn then InfiniteJump.jumpConn:Disconnect(); InfiniteJump.jumpConn = nil end
    if InfiniteJump.heartbeatConn then InfiniteJump.heartbeatConn:Disconnect(); InfiniteJump.heartbeatConn = nil end
end
InfiniteJump.start()

LP.CharacterAdded:Connect(function()
    task.wait(0.3)
    if InfiniteJump and InfiniteJump.enabled and InfiniteJump.start then
        InfiniteJump.start()
    end
end)

function getActiveMoveSpeed()
    if laggerCarryToggled then return LAGGER_CARRY_SPEED
    elseif laggerToggled then return LAGGER_SPEED
    elseif speedMode then return CS
    else return NS end
end

function getCurrentMoveModeName()
    if laggerCarryToggled then return "Lagger Carry" end
    if laggerToggled then return "Lagger" end
    if speedMode then return "Carry" end
    return "Normal"
end

function updateSpeedDisplayText()
    if not speedLabel then return end
    if showSpeedLabel == false then
        speedLabel.Visible = false
        speedLabel.Text = ""
        return
    end
    speedLabel.Visible = true
    local mode = speedDisplayMode or "Speed and Mode"
    if mode == "Speed Only" then
        speedLabel.Text = string.format("Speed: %.1f", getActiveMoveSpeed())
    elseif mode == "Mode Only" then
        speedLabel.Text = "Mode: " .. getCurrentMoveModeName()
    else
        speedLabel.Text = string.format("SPD: %.1f  MODE: %s", getActiveMoveSpeed(), getCurrentMoveModeName())
    end
end

local _velChecked = {}
local _hookedVelParts = {}
local function _setupVelChecked(char)
    _velChecked = {}
    if not char then return end
    local hrp = char:WaitForChild("HumanoidRootPart", 5)
    if hrp then _velChecked[hrp] = true end
    return hrp
end

local _hookVelSupported = nil
local function _hookVelHRP(hrp)
    if not hrp or _hookedVelParts[hrp] then return end
    if _hookVelSupported == false then return end
    if _hookVelSupported == nil then
        _hookVelSupported = (type(getrawmetatable) == "function") and (type(setreadonly) == "function")
            and (type(newcclosure) == "function") and (type(checkcaller) == "function")
    end
    if not _hookVelSupported then return end
    _hookedVelParts[hrp] = true
    local ok = pcall(function()
        local mt = getrawmetatable(hrp)
        if not mt then return end
        setreadonly(mt, false)
        local originalVelIndex = rawget(mt, "__index")
        mt.__index = newcclosure(function(self, key)
            if not checkcaller() and _velChecked[self]
               and (key == "AssemblyLinearVelocity" or key == "Velocity") then
                local real
                if type(originalVelIndex) == "function" then real = originalVelIndex(self, key)
                elseif type(originalVelIndex) == "table" then real = originalVelIndex[key] end
                if real and real.Magnitude > 20 then return real.Unit * 20 end
                return real
            end
            if type(originalVelIndex) == "function" then return originalVelIndex(self, key)
            elseif type(originalVelIndex) == "table" then return originalVelIndex[key] end
        end)
        setreadonly(mt, true)
    end)
    if not ok then _hookVelSupported = false end
end

if LP.Character then
    local _hrp0 = _setupVelChecked(LP.Character)
    _hookVelHRP(_hrp0)
end

local function _isRagdollState(hum)
    if not hum then return true end
    local st = hum:GetState()
    return hum.PlatformStand or st == Enum.HumanoidStateType.Physics
        or st == Enum.HumanoidStateType.Ragdoll or st == Enum.HumanoidStateType.FallingDown
end

local function _applyVelocitySpeed(dir, speed, hrp)
    if not hrp or not hrp.Parent then return end
    if autoBatEnabled then return end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then return end
    if dir and dir.Magnitude > 0.05 then
        pcall(function() if hrp.SetNetworkOwner then hrp:SetNetworkOwner(LP) end end)
        local unit = dir.Unit
        local vy = hrp.AssemblyLinearVelocity.Y
        hrp.AssemblyLinearVelocity = _V3new(unit.X * speed, vy, unit.Z * speed)
        _G.__RaptureLiveSpeed = { t = os.clock(), v = speed }
    else
        local vy = hrp.AssemblyLinearVelocity.Y
        hrp.AssemblyLinearVelocity = _V3new(0, vy, 0)
    end
end

local _bodyLock = {
    conn = nil,
    anchor = nil,
    attachment = nil,
    align = nil,
}

local function _bodyLockStop()
    if _bodyLock.align then pcall(function() _bodyLock.align:Destroy() end); _bodyLock.align = nil end
    if _bodyLock.attachment then pcall(function() _bodyLock.attachment:Destroy() end); _bodyLock.attachment = nil end
    if _bodyLock.anchor then pcall(function() _bodyLock.anchor:Destroy() end); _bodyLock.anchor = nil end
    local c = LP.Character
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then hum.AutoRotate = true end
end

local function _bodyLockFindTarget()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, minDist = nil, _huge
    local plist = _GetPlayersCached()
    for i = 1, #plist do        local plr = plist[i]
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local d = (tRoot.Position - root.Position).Magnitude
                if d <= bodyLockRange and d < minDist then
                    minDist = d
                    closest = tRoot
                end
            end
        end
    end
    return closest
end

local function _bodyLockStart()
    if _bodyLock.conn then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end

    local anchorPart = Instance.new("Part")
    anchorPart.Name = "RaptureBodyLockAnchor"
    anchorPart.Size = Vector3.new(1, 1, 1)
    anchorPart.Transparency = 1
    anchorPart.CanCollide = false
    anchorPart.CanQuery = false
    anchorPart.CanTouch = false
    anchorPart.Anchored = true
    anchorPart.CFrame = root.CFrame
    anchorPart.Parent = workspace:FindFirstChild("Terrain") or workspace

    local att1 = Instance.new("Attachment")
    att1.Name = "RaptureBodyLockAtt1"
    att1.Parent = root

    local att2 = Instance.new("Attachment")
    att2.Name = "RaptureBodyLockAtt2"
    att2.Parent = anchorPart

    local align = Instance.new("AlignOrientation")
    align.Name = "RaptureBodyLockAlign"
    align.Mode = Enum.OrientationAlignmentMode.TwoAttachment
    align.Attachment0 = att1
    align.Attachment1 = att2
    align.MaxTorque = 999999
    align.MaxAngularVelocity = 999999
    align.Responsiveness = 200
    align.Enabled = false
    align.Parent = anchorPart

    _bodyLock.anchor = anchorPart
    _bodyLock.attachment = att1    _bodyLock.align = align

    _bodyLock.conn = RunService.Heartbeat:Connect(function()
        if not bodyLockEnabled then return end
        local c = LP.Character
        local r = c and c:FindFirstChild("HumanoidRootPart")
        local h = c and c:FindFirstChildOfClass("Humanoid")
        if not r or not h or h.Health <= 0 then return end
        if not _bodyLock.anchor or not _bodyLock.anchor.Parent then return end
        local target = _bodyLockFindTarget()
        if target then
            h.AutoRotate = false
            align.Enabled = true
            local lookAt = Vector3.new(target.Position.X, r.Position.Y, target.Position.Z)
            _bodyLock.anchor.CFrame = CFrame.lookAt(r.Position, lookAt)
        else
            h.AutoRotate = true
            align.Enabled = false
        end
    end)
end

function startBodyLock()
    _bodyLockStop()
    _bodyLockStart()
end

function stopBodyLock()
    if _bodyLock.conn then _bodyLock.conn:Disconnect(); _bodyLock.conn = nil end
    _bodyLockStop()
end

LP.CharacterAdded:Connect(function()
    task.wait(0.3)
    if bodyLockEnabled then startBodyLock() end
end)

DEFAULT_KB = {
    DropBrainrot = {kb = Enum.KeyCode.X, gp = nil},
    AutoLeft     = {kb = Enum.KeyCode.Z, gp = nil},
    AutoRight    = {kb = Enum.KeyCode.C, gp = nil},
    AutoBat      = {kb = Enum.KeyCode.E, gp = nil},
    TPFloor      = {kb = Enum.KeyCode.F, gp = nil},
    GuiHide      = {kb = Enum.KeyCode.LeftControl, gp = nil},
    CarryToggle  = {kb = Enum.KeyCode.Q, gp = nil},
    LaggerMode   = {kb = Enum.KeyCode.R, gp = nil},
    TPBat        = {kb = Enum.KeyCode.V, gp = nil},
    InstaReset   = {kb = Enum.KeyCode.H, gp = nil},
    BatBypass    = {kb = Enum.KeyCode.B, gp = nil},
    Spin         = {kb = Enum.KeyCode.G, gp = nil},
    Platform     = {kb = Enum.KeyCode.P, gp = nil},
}
KB = {
    DropBrainrot = {kb = DEFAULT_KB.DropBrainrot.kb, gp = DEFAULT_KB.DropBrainrot.gp},
    AutoLeft     = {kb = DEFAULT_KB.AutoLeft.kb, gp = DEFAULT_KB.AutoLeft.gp},
    AutoRight    = {kb = DEFAULT_KB.AutoRight.kb, gp = DEFAULT_KB.AutoRight.gp},
    AutoBat      = {kb = DEFAULT_KB.AutoBat.kb, gp = DEFAULT_KB.AutoBat.gp},
    TPFloor      = {kb = DEFAULT_KB.TPFloor.kb, gp = DEFAULT_KB.TPFloor.gp},
    GuiHide      = {kb = DEFAULT_KB.GuiHide.kb, gp = DEFAULT_KB.GuiHide.gp},
    CarryToggle  = {kb = DEFAULT_KB.CarryToggle.kb, gp = DEFAULT_KB.CarryToggle.gp},
    LaggerMode   = {kb = DEFAULT_KB.LaggerMode.kb, gp = DEFAULT_KB.LaggerMode.gp},
    TPBat        = {kb = DEFAULT_KB.TPBat.kb, gp = DEFAULT_KB.TPBat.gp},
    InstaReset   = {kb = DEFAULT_KB.InstaReset.kb, gp = DEFAULT_KB.InstaReset.gp},
    BatBypass    = {kb = DEFAULT_KB.BatBypass.kb, gp = DEFAULT_KB.BatBypass.gp},
    Spin         = {kb = DEFAULT_KB.Spin.kb, gp = DEFAULT_KB.Spin.gp},
    Platform     = {kb = DEFAULT_KB.Platform.kb, gp = DEFAULT_KB.Platform.gp},
}

_isResetting = false
_lastSavedJSON = nil
_isLoading = false
CONFIG = { AUTO_STEAL_ENABLED = false, STEAL_RANGE = 61 }
local Steal = { AutoStealEnabled = false, StealRadius = CONFIG.STEAL_RANGE, StealDuration = 1.3, Data = {} }
local isStealing = false
local autoGrabSetDelayRadius = 9
local stealConnection = nil

local _plotsCache = nil
local _plotsCacheTime = 0
local function getPlotsRoot()
    local now = _tick()
    if _plotsCache and now - _plotsCacheTime < 2 and _plotsCache.Parent then return _plotsCache end
    _plotsCache = workspace:FindFirstChild("Plots")
    _plotsCacheTime = now
    return _plotsCache
end

local function isMyPlotByName(plotName)
    local plotsRoot = getPlotsRoot()
    if not plotsRoot then return false end
    local plot = plotsRoot:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end
    end
    return false
end

local function findNearestPrompt()
    local char = LP.Character
    if not char then return nil, nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil, nil end
    local plotsRoot = getPlotsRoot()
    if not plotsRoot then return nil, nil end
    local nearestPrompt, nearestDist, nearestName = nil, _huge, nil
    local rpos = root.Position
    for _, plot in ipairs(plotsRoot:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then continue end
        for _, pod in ipairs(pods:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base")
                local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local sp = spawn.Position
                    local dx = sp.X - rpos.X
                    local dy = sp.Y - rpos.Y
                    local dz = sp.Z - rpos.Z
                    local dist = _sqrt(dx*dx + dy*dy + dz*dz)
                    if dist < nearestDist and dist <= Steal.StealRadius then
                        local att = spawn:FindFirstChild("PromptAttachment")
                        if att then
                            for _, child in ipairs(att:GetChildren()) do
                                if child:IsA("ProximityPrompt") and child.ActionText and child.ActionText:find("Steal") then
                                    nearestPrompt = child
                                    nearestDist = dist
                                    nearestName = pod.Name
                                    break
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
    return nearestPrompt, nearestName
end

local function executeSteal(prompt, podName)
    if isStealing then return end
    if not Steal.AutoStealEnabled then return end
    if math.random(30) == 1 then
        for p in pairs(Steal.Data) do
            if not p.Parent then Steal.Data[p] = nil end
        end
    end
    if not Steal.Data[prompt] then
        Steal.Data[prompt] = { hold = {}, trigger = {}, ready = true }
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                    if c.Function then table.insert(Steal.Data[prompt].hold, c.Function) end
                end
                for _, c in ipairs(getconnections(prompt.Triggered)) do
                    if c.Function then table.insert(Steal.Data[prompt].trigger, c.Function) end
                end
            end
        end)
    end
    local data = Steal.Data[prompt]
    if not data.ready then return end
    data.ready = false
    isStealing = true
    if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
    if progressPct then progressPct.Text = "0%" end
    local pauseRatio = V2_STOP_PERCENT / 100
    task.spawn(function()
        for _, f in ipairs(data.hold) do task.spawn(f) end
        local startTime = _tick()
        local duration = Steal.StealDuration
        local stopTime = duration * pauseRatio
        local promptFired = false
        local phase = 1
        local phase2Start = 0
        local phase2Timeout = math.max(2.99 - stopTime - math.max(duration - stopTime, 0), 0.05)
        while isStealing and Steal.AutoStealEnabled and phase == 1 do
            local elapsed = _tick() - startTime
            if elapsed >= stopTime then phase = 2; phase2Start = _tick(); break end
            local progress = _clamp(elapsed / duration, 0, pauseRatio)
            if progressFill then progressFill.Size = UDim2.new(progress, 0, 1, 0) end
            if progressPct then progressPct.Text = _floor(progress * 100) .. "%" end
            if not prompt.Parent or not prompt.Parent.Parent then break end
            local char = LP.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp and (hrp.Position - prompt.Parent.Parent.Position).Magnitude > Steal.StealRadius then break end
            task.wait()
        end
        if phase == 2 then
            if progressFill then progressFill.Size = UDim2.new(pauseRatio, 0, 1, 0) end
            if progressPct then progressPct.Text = _floor(pauseRatio * 100) .. "%" end
        end
        while isStealing and Steal.AutoStealEnabled and phase == 2 do
            if _tick() - phase2Start >= phase2Timeout then
                if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
                if progressPct then progressPct.Text = "0%" end
                data.ready = true; isStealing = false
                task.wait()
                local newPrompt, newName = findNearestPrompt()
                if newPrompt then executeSteal(newPrompt, newName) end
                return
            end
            if not prompt.Parent or not prompt.Parent.Parent then
                isStealing = false
                if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
                if progressPct then progressPct.Text = "0%" end
                data.ready = true
                return
            end
            local char = LP.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local dist = (hrp.Position - prompt.Parent.Parent.Position).Magnitude
                if dist <= autoGrabSetDelayRadius then phase = 3; break
                elseif dist > Steal.StealRadius then
                    isStealing = false
                    if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
                    if progressPct then progressPct.Text = "0%" end
                    data.ready = true
                    return
                end
            end
            task.wait()
        end
        if isStealing and Steal.AutoStealEnabled and phase == 3 then
            local fillStart = _tick()
            local fillDuration = math.max(duration - stopTime, 0.05)
            while true do
                local fp = _clamp((_tick() - fillStart) / fillDuration, 0, 1)
                local totalProgress = pauseRatio + fp * (1 - pauseRatio)
                if fp >= 1 then totalProgress = 1 end
                if progressFill then progressFill.Size = UDim2.new(totalProgress, 0, 1, 0) end
                if progressPct then progressPct.Text = _floor(totalProgress * 100) .. "%" end
                if fp >= 1 and not promptFired then
                    promptFired = true
                    pcall(function()
                        for _, f in ipairs(data.trigger) do task.spawn(f) end
                        local remote = ReplicatedStorage:FindFirstChild("StealAnimal")
                        if remote and podName then remote:FireServer(podName) end
                        if prompt then prompt:Fire() end
                    end)
                    break
                end
                task.wait()
            end
        end
        if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
        if progressPct then progressPct.Text = "0%" end
        data.ready = true
        isStealing = false
    end)
end

function startAutoSteal()
    if stealConnection then
        local connected = false
        pcall(function() connected = stealConnection.Connected == true end)
        if connected then
            Steal.StealRadius = CONFIG.STEAL_RANGE
            Steal.AutoStealEnabled = true
            CONFIG.AUTO_STEAL_ENABLED = true
            return true
        end
        pcall(function() stealConnection:Disconnect() end)
        stealConnection = nil
    end
    Steal.StealRadius = CONFIG.STEAL_RANGE
    Steal.AutoStealEnabled = true
    CONFIG.AUTO_STEAL_ENABLED = true
    stealConnection = RunService.Heartbeat:Connect(function()
        if not Steal.AutoStealEnabled or isStealing then return end
        local p, n = findNearestPrompt()
        if p then executeSteal(p, n) end
    end)
    return true
end

function stopAutoSteal()
    if stealConnection then stealConnection:Disconnect(); stealConnection = nil end
    isStealing = false
    Steal.AutoStealEnabled = false
    CONFIG.AUTO_STEAL_ENABLED = false
    if progressFill then TS:Create(progressFill, TweenInfo.new(0.2), { Size = UDim2.new(0, 0, 1, 0) }):Play() end
    if progressPct then progressPct.Text = "0%" end
end

medusaDebounce = false
medusaLastUsed = 0
dropActive = false
lastDropTime = 0
origFOV = nil
fovEnabled = false
fovValue = 70
customFovConn = nil
setFovVisual = nil
fovSliderSet = nil

_anyKeyListening = false
_aimbotConn = nil
_prevAutoRotate = nil
tpBatConn = nil
tpBatPrevAutoRotate = nil
tpBatHitCD = false
TP_BAT_SWING_CD = 0.08
tpBatFloatingButton = nil
enemySpeedConn = nil
movementLoop = nil
steppedConn = nil
alConn = nil
arConn = nil
infJumpConn = nil
stretchConn = nil
stretchFovConn = nil
medusaResetConns = {}
dropConnections = {}
enemySpeedLabels = {}
Conns = {autoSteal = nil, batCounter = nil, anchor = {}, progress = nil, autoLeft = nil, autoRight = nil, medusaAuto = nil, noPlayerCol = nil}
keyButtonRefs = {}
progressFill = nil
progressPct = nil
pbFrame = nil
speedLabel = nil
normalBox, carryBox, laggerBox, lagger2Box, radInput, batSpeedBox, uiScaleBox, progressBarScaleBox = nil, nil, nil, nil, nil, nil, nil, nil
stretchValueBox = nil
medusaRadiusBox = nil
medusaDelayBox = nil
setJumpToggleState = nil
autoBatSetVisual, autoLeftSetVisual, autoRightSetVisual, setBatCounterVisual, setMedusaVisual = nil, nil, nil, nil, nil
setAutoMedusaVisual, setRagdollTimerVisual = nil, nil
setAntiRagVisual, setJumpVisual, setLockUIVisual, setInstaGrab = nil, nil, nil, nil
setESPVisual, setAntiLagVisual = nil, nil
mobSetAutoBat, mobSetAutoLeft, mobSetAutoRight, mobSetDropBR, mobSetTpDown, mobSetCarry, mobSetLagger1, mobSetLagger2 = nil, nil, nil, nil, nil, nil, nil, nil
main, gui = nil, nil
MobilePanel = nil
instaResetFloatingButton = nil
batBypassFloatingButton = nil
showGui = nil
hideGui = nil
mainUIScale = nil
pbScale = nil
tabButtons = {}
contentPages = {}
currentTab = "Speed"
lockBtn = nil
fpsLabel = nil
msLabel = nil
ragdollCountdownLabel = nil

skyTheme = "Off"
skySelectorLabel = nil
_originalLighting = nil
setNeonWeatherVisual = nil

SKY_PRESETS_LIST = {
    "Off", "Night", "Aurora", "Sunset", "Galaxy", "Cyber", "Sakura",
    "Pink Night", "Blood Moon", "Emerald Dawn", "Volcanic", "Arctic",
    "Midnight Ocean", "Vaporwave", "Toxic", "Solar Eclipse", "Hellscape",
    "Heaven", "Storm", "Sunrise", "Deep Space", "Lavender Dream",
    "Inferno", "Mint Sky", "Purple Sky",
}

local function vC3(d)
    return Color3.fromRGB(d[1], d[2], d[3])
end

SKY_PRESETS = {
    Off = { kind = "off" },
    Night = {
        clock = 22, brightness = 2,
        ambient = {110,100,130}, outAmb = {120,110,140},
        sky = { stars = 4000, moon = 18, sun = 0, moonTex = true },
        atm = { dens = 0.45, color = {120,60,180}, decay = {60,20,100}, glare = 0.5, haze = 1.2 },
    },
    Aurora = {
        clock = 14, brightness = 3,
        ambient = {150,120,150}, outAmb = {160,130,150},
        atm = { dens = 0.55, color = {85,80,200}, decay = {255,20,150}, glare = 2.5, haze = 3 },
        clouds = { cover = 0.7, dens = 0.7, color = {85,240,250} },
    },
    Sunset = {
        clock = 17.2, brightness = 2.5,
        ambient = {170,120,100}, outAmb = {180,130,110},
        sky = { stars = 0, sun = 25, moon = 0 },
        atm = { dens = 0.5, color = {255,130,60}, decay = {255,80,30}, glare = 2, haze = 2.5 },
        clouds = { cover = 0.55, dens = 0.55, color = {255,200,140} },
    },
    Galaxy = {
        clock = 0, brightness = 1.5,
        ambient = {70,60,100}, outAmb = {80,70,110},
        sky = { stars = 10000, moon = 30, sun = 0 },
        atm = { dens = 0.15, color = {40,20,80}, decay = {20,10,50}, glare = 0.3, haze = 0.5 },
    },
    Cyber = {
        clock = 21, brightness = 2.2,
        ambient = {90,130,170}, outAmb = {100,140,180},
        sky = { stars = 2000, moon = 12 },
        atm = { dens = 0.4, color = {0,200,255}, decay = {150,0,255}, glare = 2, haze = 2 },
        clouds = { cover = 0.4, dens = 0.6, color = {100,200,85} },
    },
    Sakura = {
        clock = 11, brightness = 3.5,
        ambient = {170,150,160}, outAmb = {180,160,170},
        sky = { sun = 8 },
        atm = { dens = 0.3, color = {85,200,220}, decay = {255,170,200}, glare = 1, haze = 1.5 },
        clouds = { cover = 0.6, dens = 0.4, color = {255,250,252} },
    },
    ["Pink Night"] = {
        clock = 23, brightness = 2.2,
        ambient = {120,60,110}, outAmb = {140,70,120},
        sky = { stars = 5000, moon = 22, sun = 0, moonTex = true },
        atm = { dens = 0.5, color = {85,80,180}, decay = {140,30,100}, glare = 0.7, haze = 1.4 },
        clouds = { cover = 0.3, dens = 0.5, color = {180,90,150} },
    },
    ["Blood Moon"] = {
        clock = 22.5, brightness = 1.8,
        ambient = {130,70,70}, outAmb = {150,80,80},
        sky = { stars = 1500, moon = 28, sun = 0, moonTex = true },
        atm = { dens = 0.45, color = {220,80,80}, decay = {120,40,40}, glare = 1, haze = 1.5 },
        clouds = { cover = 0.45, dens = 0.55, color = {120,60,60} },
    },
    ["Emerald Dawn"] = {
        clock = 6.5, brightness = 2.8,
        ambient = {130,170,140}, outAmb = {140,180,150},
        sky = { sun = 18, moon = 0, stars = 0 },
        atm = { dens = 0.4, color = {80,200,140}, decay = {40,150,90}, glare = 1.8, haze = 2.2 },
        clouds = { cover = 0.5, dens = 0.5, color = {200,255,220} },
    },
    Volcanic = {
        clock = 19, brightness = 2.2,
        ambient = {180,110,80}, outAmb = {200,120,90},
        sky = { stars = 200, sun = 12, moon = 0 },
        atm = { dens = 0.55, color = {255,110,60}, decay = {180,70,40}, glare = 2, haze = 2.2 },
        clouds = { cover = 0.65, dens = 0.7, color = {120,70,50} },
    },
    Arctic = {
        clock = 9, brightness = 3.2,
        ambient = {200,220,235}, outAmb = {210,230,245},
        sky = { sun = 10, stars = 0, moon = 0 },
        atm = { dens = 0.3, color = {180,220,255}, decay = {140,200,240}, glare = 1.5, haze = 1.8 },
        clouds = { cover = 0.7, dens = 0.6, color = {250,253,85} },
    },
    ["Midnight Ocean"] = {
        clock = 1.5, brightness = 1.7,
        ambient = {60,90,130}, outAmb = {70,100,140},
        sky = { stars = 6000, moon = 24, sun = 0, moonTex = true },
        atm = { dens = 0.5, color = {20,60,140}, decay = {10,30,90}, glare = 0.6, haze = 1.5 },
    },
    Vaporwave = {
        clock = 19.5, brightness = 2.4,
        ambient = {180,120,200}, outAmb = {190,130,210},
        sky = { stars = 1000, moon = 14 },
        atm = { dens = 0.45, color = {255,100,220}, decay = {120,60,255}, glare = 2.2, haze = 2.4 },
        clouds = { cover = 0.55, dens = 0.55, color = {200,150,85} },
    },
    Toxic = {
        clock = 13, brightness = 2.5,
        ambient = {140,180,80}, outAmb = {150,190,90},
        atm = { dens = 0.45, color = {100,220,40}, decay = {60,150,20}, glare = 1.5, haze = 1.8 },
        clouds = { cover = 0.55, dens = 0.55, color = {180,255,120} },
    },
    ["Solar Eclipse"] = {
        clock = 12, brightness = 1.2,
        ambient = {70,60,80}, outAmb = {80,70,90},
        sky = { stars = 3500, sun = 22, moon = 0 },
        atm = { dens = 0.4, color = {255,180,90}, decay = {50,40,60}, glare = 2, haze = 1.5 },
    },
    Hellscape = {
        clock = 18, brightness = 2.2,
        ambient = {200,100,80}, outAmb = {220,110,90},
        sky = { stars = 100, sun = 30, moon = 0 },
        atm = { dens = 0.6, color = {255,90,60}, decay = {140,50,40}, glare = 2.2, haze = 2.4 },
        clouds = { cover = 0.8, dens = 0.8, color = {90,50,40} },
    },
    Heaven = {
        clock = 12, brightness = 4,
        ambient = {240,240,210}, outAmb = {250,245,220},
        sky = { sun = 16, moon = 0, stars = 0 },
        atm = { dens = 0.25, color = {255,250,220}, decay = {85,240,200}, glare = 3, haze = 1.5 },
        clouds = { cover = 0.85, dens = 0.5, color = {255,255,255} },
    },
    Storm = {
        clock = 15, brightness = 1.6,
        ambient = {105,105,125}, outAmb = {115,115,135},
        sky = { stars = 0, sun = 6, moon = 0 },
        atm = { dens = 0.55, color = {90,100,130}, decay = {55,65,90}, glare = 0.5, haze = 2.2 },
        clouds = { cover = 0.85, dens = 0.5, color = {70,75,90} },
    },
    Sunrise = {
        clock = 6.2, brightness = 2.8,
        ambient = {220,180,130}, outAmb = {230,190,140},
        sky = { sun = 22, stars = 0, moon = 0 },
        atm = { dens = 0.45, color = {255,180,100}, decay = {255,140,80}, glare = 2.4, haze = 2.2 },
        clouds = { cover = 0.4, dens = 0.4, color = {85,220,180} },
    },
    ["Deep Space"] = {
        clock = 0, brightness = 1,
        ambient = {30,25,50}, outAmb = {40,35,60},
        sky = { stars = 15000, moon = 0, sun = 0 },
        atm = { dens = 0.05, color = {15,5,40}, decay = {5,0,20}, glare = 0.2, haze = 0.3 },
    },
    ["Lavender Dream"] = {
        clock = 18.5, brightness = 2.6,
        ambient = {180,160,220}, outAmb = {190,170,230},
        sky = { stars = 800, moon = 16, sun = 0 },
        atm = { dens = 0.4, color = {200,160,255}, decay = {160,120,220}, glare = 1.4, haze = 1.8 },
        clouds = { cover = 0.55, dens = 0.5, color = {220,200,85} },
    },
    Inferno = {
        clock = 17.5, brightness = 2.4,
        ambient = {220,140,90}, outAmb = {240,150,100},
        sky = { sun = 26, moon = 0, stars = 0 },
        atm = { dens = 0.5, color = {255,130,70}, decay = {200,80,40}, glare = 2.2, haze = 2.2 },
        clouds = { cover = 0.6, dens = 0.6, color = {200,110,80} },
    },
    ["Purple Sky"] = {
        clock = 14, brightness = 2,
        ambient = {107,17,150}, outAmb = {107,17,150},
        fog = {130,27,200},
        atm = { dens = 0.35, color = {160,60,230}, decay = {100,20,160}, glare = 1, haze = 1.5 },
    },
    ["Mint Sky"] = {
        clock = 10, brightness = 3.2,
        ambient = {180,230,210}, outAmb = {190,240,220},
        sky = { sun = 10 },
        atm = { dens = 0.32, color = {150,255,210}, decay = {100,220,180}, glare = 1.6, haze = 1.6 },
        clouds = { cover = 0.55, dens = 0.45, color = {240,85,250} },
    },
}

local function _v4mpClearSky()
    for _, child in ipairs(Lighting:GetChildren()) do
        if child:GetAttribute("_RaptureSky") then
            pcall(function() child:Destroy() end)
        end
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        for _, child in ipairs(terrain:GetChildren()) do
            if child:GetAttribute("_RaptureSky") then
                pcall(function() child:Destroy() end)
            end
        end
    end
end

function applyCustomSky(mode)
    _v4mpClearSky()
    local preset = SKY_PRESETS[mode]
    if not preset or preset.kind == "off" then
        Lighting.ClockTime = 14
        Lighting.Brightness = 2
        Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
        Lighting.Ambient = Color3.fromRGB(127, 127, 127)
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = true
        skyTheme = "Off"
        return
    end
    Lighting.FogStart = 0
    Lighting.FogEnd = 100000
    Lighting.FogColor = vC3(preset.fog or { 200, 200, 200 })
    Lighting.ColorShift_Top = Color3.fromRGB(0, 0, 0)
    Lighting.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)
    Lighting.GlobalShadows = true
    Lighting.ClockTime = preset.clock or 14
    Lighting.Brightness = preset.brightness or 2
    if preset.outAmb then Lighting.OutdoorAmbient = vC3(preset.outAmb) end
    if preset.ambient then Lighting.Ambient = vC3(preset.ambient) end
    if preset.sky then
        local sky = Instance.new("Sky")
        sky:SetAttribute("_RaptureSky", true)
        if preset.sky.stars then sky.StarCount = preset.sky.stars end
        if preset.sky.moon then sky.MoonAngularSize = preset.sky.moon end
        if preset.sky.sun then sky.SunAngularSize = preset.sky.sun end
        if preset.sky.moonTex then sky.MoonTextureId = "rbxasset://sky/moon.jpg" end
        sky.Parent = Lighting
    end
    if preset.atm then
        local atm = Instance.new("Atmosphere")
        atm:SetAttribute("_RaptureSky", true)
        atm.Density = preset.atm.dens or 0.3
        atm.Color = vC3(preset.atm.color)
        atm.Decay = vC3(preset.atm.decay)
        atm.Glare = preset.atm.glare or 1
        atm.Haze = preset.atm.haze or 1
        atm.Parent = Lighting
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if preset.clouds and terrain then
        local clouds = Instance.new("Clouds")
        clouds:SetAttribute("_RaptureSky", true)
        clouds.Cover = preset.clouds.cover or 0.5
        clouds.Density = preset.clouds.dens or 0.5
        clouds.Color = vC3(preset.clouds.color)
        clouds.Parent = terrain
    end
    skyTheme = mode
end

GAMEPAD_KEYS = {
    [Enum.KeyCode.ButtonA] = true, [Enum.KeyCode.ButtonB] = true,
    [Enum.KeyCode.ButtonX] = true, [Enum.KeyCode.ButtonY] = true,
    [Enum.KeyCode.ButtonL1] = true, [Enum.KeyCode.ButtonR1] = true,
    [Enum.KeyCode.ButtonL2] = true, [Enum.KeyCode.ButtonR2] = true,
    [Enum.KeyCode.ButtonL3] = true, [Enum.KeyCode.ButtonR3] = true,
    [Enum.KeyCode.ButtonStart] = true, [Enum.KeyCode.ButtonSelect] = true,
    [Enum.KeyCode.DPadUp] = true, [Enum.KeyCode.DPadDown] = true,
    [Enum.KeyCode.DPadLeft] = true, [Enum.KeyCode.DPadRight] = true,
}

MOVE_KEYS = {
    [Enum.KeyCode.W] = true, [Enum.KeyCode.A] = true,
    [Enum.KeyCode.S] = true, [Enum.KeyCode.D] = true,
    [Enum.KeyCode.Up] = true, [Enum.KeyCode.Left] = true,
    [Enum.KeyCode.Down] = true, [Enum.KeyCode.Right] = true,
}

BAT_COUNTER_SLAP_LIST = {
    "Bat", "Slap", "Iron Slap", "Gold Slap", "Diamond Slap",
    "Emerald Slap", "Ruby Slap", "Dark Matter Slap", "Flame Slap",
    "Nuclear Slap", "Galaxy Slap", "Glitched Slap"
}

AP = {
    L1 = _V3new(-476.48, -6.28, 92.73),
    L2 = _V3new(-483.12, -4.95, 94.80),
    L_FACE = _V3new(-482.25, -4.96, 92.09),
    R1 = _V3new(-476.16, -6.52, 25.62),
    R2 = _V3new(-483.06, -5.03, 25.48),
    R_FACE = _V3new(-482.06, -6.93, 35.47),
}

function isGamepadInput(inp)
    return inp and inp.UserInputType and inp.UserInputType.Name:match("^Gamepad") ~= nil
end

function isBindableInput(inp)
    if not inp or inp.KeyCode == Enum.KeyCode.Unknown then return false end
    if inp.UserInputType == Enum.UserInputType.Keyboard then return true end
    return isGamepadInput(inp) and GAMEPAD_KEYS[inp.KeyCode] == true
end

function kbMatch(entry, kc)
    return kc and (entry.kb == kc or (entry.gp and kc == entry.gp))
end

function resetProgressBar()
    if progressPct then progressPct.Text = "0%" end
    if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
end

function saveSkipIntroSetting() end
function setSkipIntro(enabled) skipIntroEnabled = true; saveAllSettings() end

local AntiRagdollV1 = {}
AntiRagdollV1.__index = AntiRagdollV1
local BOOST_SPEED = 400
local AR_DEFAULT_SPEED = 16
local stateV1 = { active = false, isBoosting = false, cachedChar = nil, ragdollConnections = {} }

local function disconnectAllV1()
    for _, conn in ipairs(stateV1.ragdollConnections) do pcall(function() conn:Disconnect() end) end
    stateV1.ragdollConnections = {}
end

local function cacheCharacterV1()
    local char = LP.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return false end
    stateV1.cachedChar = { character = char, humanoid = hum, root = root }
    return true
end

local function isRagdolledV1()
    if not stateV1.cachedChar or not stateV1.cachedChar.humanoid then return false end
    local hum = stateV1.cachedChar.humanoid
    local st = hum:GetState()
    local ragdollStates = {
        [Enum.HumanoidStateType.Physics] = true,
        [Enum.HumanoidStateType.Ragdoll] = true,
        [Enum.HumanoidStateType.FallingDown] = true,
    }
    return ragdollStates[st] or false
end

local function forceExitRagdollV1()
    if not stateV1.cachedChar or not stateV1.cachedChar.humanoid or not stateV1.cachedChar.root then return end
    local hum = stateV1.cachedChar.humanoid
    local root = stateV1.cachedChar.root
    pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
    for _, descendant in ipairs(stateV1.cachedChar.character:GetDescendants()) do
        if descendant:IsA("BallSocketConstraint") or
           (descendant:IsA("Attachment") and descendant.Name:find("RagdollAttachment")) then
            descendant:Destroy()
        end
    end
    if not stateV1.isBoosting then
        stateV1.isBoosting = true
        hum.WalkSpeed = BOOST_SPEED
    end
    if hum.Health > 0 then hum:ChangeState(Enum.HumanoidStateType.Running) end
    root.Anchored = false
end

local function heartbeatLoopV1()
    while stateV1.active do
        task.wait()
        if isRagdolledV1() then forceExitRagdollV1()
        elseif stateV1.isBoosting and not isRagdolledV1() then
            stateV1.isBoosting = false
            if stateV1.cachedChar and stateV1.cachedChar.humanoid then
                stateV1.cachedChar.humanoid.WalkSpeed = AR_DEFAULT_SPEED
            end
        end
    end
end

function AntiRagdollV1.start()
    if stateV1.active then return end
    AntiRagdollV1.stop()
    if not cacheCharacterV1() then return end
    stateV1.active = true
    stateV1.isBoosting = false
    local camConn = RunService.RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        if cam and stateV1.cachedChar and stateV1.cachedChar.humanoid then
            cam.CameraSubject = stateV1.cachedChar.humanoid
        end
    end)
    table.insert(stateV1.ragdollConnections, camConn)
    local respawnConn = LP.CharacterAdded:Connect(function()
        stateV1.isBoosting = false
        task.wait(0.5)
        cacheCharacterV1()
    end)
    table.insert(stateV1.ragdollConnections, respawnConn)
    task.spawn(heartbeatLoopV1)
end

function AntiRagdollV1.stop()
    stateV1.active = false
    if stateV1.isBoosting and stateV1.cachedChar and stateV1.cachedChar.humanoid then
        stateV1.cachedChar.humanoid.WalkSpeed = AR_DEFAULT_SPEED
    end
    stateV1.isBoosting = false
    disconnectAllV1()
    stateV1.cachedChar = nil
end

function AntiRagdollV1.isRunning() return stateV1.active end

local AntiRagdollV2 = { Enabled = false, Connection = nil, ResetCooldown = 0 }

local function startAntiRagdollV2()
    if AntiRagdollV2.Connection then return end
    AntiRagdollV2.Enabled = true
    AntiRagdollV2.Connection = RunService.Heartbeat:Connect(function()
        if not AntiRagdollV2.Enabled then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root then return end
        if hum.Health <= 0 or hum:GetState() == Enum.HumanoidStateType.Dead then return end
        local state = hum:GetState()
        local now = _tick()
        if state == Enum.HumanoidStateType.Physics or
           state == Enum.HumanoidStateType.Ragdoll or
           state == Enum.HumanoidStateType.FallingDown then
            if now - AntiRagdollV2.ResetCooldown > 0.15 then
                AntiRagdollV2.ResetCooldown = now
                pcall(function()
                    if hum:GetState() == Enum.HumanoidStateType.GettingUp then return end
                    if hum.Health <= 0 or hum:GetState() == Enum.HumanoidStateType.Dead then return end
                    hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                    root.Velocity = _V3zero
                    root.RotVelocity = _V3zero
                    root.AssemblyLinearVelocity = _V3zero
                    root.AssemblyAngularVelocity = _V3zero
                    for _, obj in ipairs(char:GetDescendants()) do
                        if obj:IsA("Motor6D") then obj.Enabled = true end
                        if obj:IsA("Constraint") then obj.Enabled = true end
                    end
                    workspace.CurrentCamera.CameraSubject = hum
                    local PM = LP.PlayerScripts:FindFirstChild("PlayerModule")
                    if PM then
                        local CM = require(PM:FindFirstChild("ControlModule"))
                        if CM then CM:Enable() end
                    end
                    hum.AutoRotate = true
                    hum.PlatformStand = false
                    hum.Sit = false
                end)
            end
        end
    end)
end

local function stopAntiRagdollV2()
    AntiRagdollV2.Enabled = false
    if AntiRagdollV2.Connection then
        AntiRagdollV2.Connection:Disconnect()
        AntiRagdollV2.Connection = nil
    end
    AntiRagdollV2.ResetCooldown = 0
end

function setAntiRagdollEnabled(enabled)
    antiRagdollEnabled = enabled
    if enabled then
        if antiRagdollVersion == "v1" then AntiRagdollV1.start()
        elseif antiRagdollVersion == "v2" then startAntiRagdollV2() end
    else
        if AntiRagdollV1.isRunning() then AntiRagdollV1.stop() end
        if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
    end
    saveAllSettings()
end

function setAntiRagdollVersion(version)
    antiRagdollVersion = version
    if antiRagdollEnabled then
        if AntiRagdollV1.isRunning() then AntiRagdollV1.stop() end
        if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
        if version == "v1" then AntiRagdollV1.start()
        elseif version == "v2" then startAntiRagdollV2() end
    end
    saveAllSettings()
end

function findMedusa()
    local c = LP.Character
    if not c then return nil end
    for _, t in ipairs(c:GetChildren()) do
        if t:IsA("Tool") then
            local n = t.Name:lower()
            if n:find("medusa") or n:find("head") or n:find("stone") then return t end
        end
    end
    local bp = LP:FindFirstChild("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") then
                local n = t.Name:lower()
                if n:find("medusa") or n:find("head") or n:find("stone") then return t end
            end
        end
    end
    return nil
end

function useMedusaCounter()
    if medusaDebounce then return end
    if _tick() - medusaLastUsed < MEDUSA_COOLDOWN then return end
    local c = LP.Character
    if not c then return end
    medusaDebounce = true
    local med = findMedusa()
    if not med then medusaDebounce = false; return end
    if med.Parent ~= c then
        local hum2 = c:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2:EquipTool(med) end
    end
    pcall(function() med:Activate() end)
    medusaLastUsed = _tick()
    medusaDebounce = false
end

function onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if medusaCounterEnabled and part.Anchored and part.Transparency == 1 then useMedusaCounter() end
    end)
end

function setupMedusaCounter(char)
    for _, c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end
    Conns.anchor = {}
    if not char or not medusaCounterEnabled then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end
    end
    table.insert(Conns.anchor, char.DescendantAdded:Connect(function(part)
        if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end
    end))
end

function stopMedusaCounter()
    for _, c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end
    Conns.anchor = {}
end

local function espCreateGlow(plr)
    local old = workspace:FindFirstChild("RaptureESPGlow_" .. tostring(plr.UserId))
    if old then old:Destroy() end
    local glow = Instance.new("Part")
    glow.Name = "RaptureESPGlow_" .. tostring(plr.UserId)
    glow.Shape = Enum.PartType.Ball
    glow.Size = Vector3.new(0.65, 0.65, 0.65)
    glow.Anchored = true
    glow.CanCollide = false
    glow.CanTouch = false
    glow.CanQuery = false
    glow.CastShadow = false
    glow.Material = Enum.Material.SmoothPlastic
    glow.Color = getThemeColor()
    glow.Transparency = 0.05
    glow.CFrame = CFrame.new(0, -10000, 0)
    glow.Parent = workspace
    local light = Instance.new("PointLight", glow)
    light.Color = getThemeColor()
    light.Brightness = 2.5
    light.Range = 9
    light.Shadows = false
    return glow
end

local function espCreateBox(char)
    if not char then return end
    local old = char:FindFirstChild("RaptureESPBox")
    if old then old:Destroy() end
    local h = Instance.new("Highlight")
    h.Name = "RaptureESPBox"
    h.Adornee = char
    h.FillColor = getThemeColor()
    h.OutlineColor = Color3.fromRGB(0, 0, 0)
    h.FillTransparency = 0.7
    h.OutlineTransparency = 0.1
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent = char
    return h
end

local function espCreateBillboard(plr, char)
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end
    local old = head:FindFirstChild("RaptureESPAvatar")
    if old then old:Destroy() end
    local bb = Instance.new("BillboardGui")
    bb.Name = "RaptureESPAvatar"
    bb.Size = UDim2.new(0, 80, 0, 60)
    bb.StudsOffset = Vector3.new(0, 2.5, 0)
    bb.AlwaysOnTop = true
    bb.Parent = head
    local container = Instance.new("Frame", bb)
    container.Size = UDim2.new(1, 0, 1, 0)
    container.BackgroundTransparency = 1
    local name = Instance.new("TextLabel", container)
    name.Size = UDim2.new(1, 0, 0, 20)
    name.Position = UDim2.new(0, 0, 0, 0)
    name.BackgroundTransparency = 1
    name.Text = plr.DisplayName or plr.Name
    name.TextColor3 = getThemeColor()
    name.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    name.TextStrokeTransparency = 0.3
    name.Font = Enum.Font.GothamBold
    name.TextSize = 12
    return bb
end

function startESP()
    if espConn then return end
    espEnabled = true
    local function setupPlayer(plr)
        if plr == LP then return end
        if espHighlightCache[plr] then return end
        local char = plr.Character
        if char then
            espBillboardCache[plr] = espCreateBillboard(plr, char)
            espHighlightCache[plr] = espCreateBox(char)
        end
        espGlowCache[plr] = espCreateGlow(plr)
    end
    local function removePlayer(plr)
        if espBillboardCache[plr] then pcall(function() espBillboardCache[plr]:Destroy() end); espBillboardCache[plr] = nil end
        if espHighlightCache[plr] then pcall(function() espHighlightCache[plr]:Destroy() end); espHighlightCache[plr] = nil end
        if espGlowCache[plr] then pcall(function() espGlowCache[plr]:Destroy() end); espGlowCache[plr] = nil end
    end
    for _, plr in ipairs(Players:GetPlayers()) do setupPlayer(plr) end
    Conns.espAdded = Players.PlayerAdded:Connect(setupPlayer)
    Conns.espRemoving = Players.PlayerRemoving:Connect(removePlayer)
    Conns.espCharAdded = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP then
            table.insert(Conns.espCharAdded, plr.CharacterAdded:Connect(function(char)
                task.wait(0.3)
                if espEnabled then
                    if espBillboardCache[plr] then pcall(function() espBillboardCache[plr]:Destroy() end) end
                    if espHighlightCache[plr] then pcall(function() espHighlightCache[plr]:Destroy() end) end
                    espBillboardCache[plr] = espCreateBillboard(plr, char)
                    espHighlightCache[plr] = espCreateBox(char)
                end
            end))
        end
    end
    espConn = RunService.RenderStepped:Connect(function()
        if not espEnabled then return end
        local accent = getThemeColor()
        for plr, glow in pairs(espGlowCache) do
            if glow and glow.Parent then
                local char = plr.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    glow.Color = accent
                    glow.CFrame = CFrame.new(hrp.Position - Vector3.new(0, (hrp.Size.Y * 0.5) + 2.35, 0))
                    local light = glow:FindFirstChildOfClass("PointLight")
                    if light then light.Color = accent end
                else
                    glow.CFrame = CFrame.new(0, -10000, 0)
                end
            end
        end
        for plr, hl in pairs(espHighlightCache) do
            if hl and hl.Parent then hl.FillColor = accent end
        end
        for plr, billboard in pairs(espBillboardCache) do
            if billboard and billboard.Parent then
                local container = billboard:FindFirstChild("Frame")
                if container then
                    local nameLbl = container:FindFirstChild("TextLabel")
                    if nameLbl then nameLbl.TextColor3 = accent end
                end
            end
        end
    end)
end

function stopESP()
    espEnabled = false
    if espConn then espConn:Disconnect(); espConn = nil end
    if Conns.espAdded then Conns.espAdded:Disconnect(); Conns.espAdded = nil end
    if Conns.espRemoving then Conns.espRemoving:Disconnect(); Conns.espRemoving = nil end
    if Conns.espCharAdded then
        for _, c in ipairs(Conns.espCharAdded) do pcall(function() c:Disconnect() end) end
        Conns.espCharAdded = {}
    end
    for plr in pairs(espBillboardCache) do pcall(function() espBillboardCache[plr]:Destroy() end) end
    for plr in pairs(espHighlightCache) do pcall(function() espHighlightCache[plr]:Destroy() end) end
    for plr in pairs(espGlowCache) do pcall(function() espGlowCache[plr]:Destroy() end) end
    espBillboardCache = {}
    espHighlightCache = {}
    espGlowCache = {}
end

_antiLagStored = {
    lighting = {},
    qualityLevel = nil,
    meshDetail = nil,
    terrainWater = nil,
    technology = nil,
}

local function applyAntiLagDerender(obj)
    pcall(function()
        if obj:GetAttribute("_AmbitiousHubSky") ~= nil then return end
        if obj:IsA("Sky") or obj:IsA("Atmosphere") or obj:IsA("Clouds") then return end
        if obj:IsA("Accessory") or obj:IsA("Hat") then
            local char = LP.Character
            if char and obj:IsDescendantOf(char) then return end
            obj:Destroy()
        elseif obj:IsA("BasePart") then
            obj.Material      = Enum.Material.Plastic
            obj.Reflectance   = 0
            obj.CastShadow    = false
            obj.TopSurface    = Enum.SurfaceType.Smooth
            obj.BottomSurface = Enum.SurfaceType.Smooth
            obj.FrontSurface  = Enum.SurfaceType.Smooth
            obj.BackSurface   = Enum.SurfaceType.Smooth
            obj.LeftSurface   = Enum.SurfaceType.Smooth
            obj.RightSurface  = Enum.SurfaceType.Smooth
            for _, child in ipairs(obj:GetChildren()) do
                if child:IsA("Decal") or child:IsA("Texture") then
                    child.Transparency = 0.5
                end
            end
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 0.5
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
            or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
            obj.Enabled = false
        elseif obj:IsA("PointLight") or obj:IsA("SpotLight")
            or obj:IsA("SurfaceLight") then
            obj.Enabled = false
        elseif obj:IsA("SelectionBox") and obj.Name ~= "AmbitiousHubESP" then
            pcall(function() obj:Destroy() end)
        end
    end)
end

function enableAntiLag()
    antiLagEnabled           = true
    removeAccessoriesEnabled = true
    _antiLagStored.lighting = {
        Brightness          = Lighting.Brightness,
        ClockTime           = Lighting.ClockTime,
        OutdoorAmbient      = Lighting.OutdoorAmbient,
        Ambient             = Lighting.Ambient,
        GlobalShadows       = Lighting.GlobalShadows,
        FogEnd              = Lighting.FogEnd,
        FogStart            = Lighting.FogStart,
        EnvironmentDiffuseScale  = Lighting.EnvironmentDiffuseScale,
        EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
        ShadowSoftness      = Lighting.ShadowSoftness,
        ExposureCompensation = Lighting.ExposureCompensation,
        Technology          = Lighting.Technology,
    }
    _antiLagStored.qualityLevel = settings().Rendering.QualityLevel
    _antiLagStored.meshDetail   = settings().Rendering.MeshPartDetailLevel
    _antiLagStored.technology   = Lighting.Technology
    pcall(function()
        settings().Rendering.QualityLevel        = 1
        settings().Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Disabled
    end)
    Lighting.GlobalShadows            = false
    Lighting.FogEnd                   = 1e10
    Lighting.FogStart                 = 0
    Lighting.Brightness               = 1
    Lighting.EnvironmentDiffuseScale  = 0
    Lighting.EnvironmentSpecularScale = 0
    Lighting.ShadowSoftness           = 0
    Lighting.ExposureCompensation     = 0
    pcall(function() Lighting.Technology = Enum.Technology.Compatibility end)
    for _, e in pairs(Lighting:GetChildren()) do
        pcall(function()
            if e:GetAttribute("_AmbitiousVivid") then return end
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect")
            or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect")
            or e:IsA("DepthOfFieldEffect") or e:IsA("BrightnessEffect")
            or e:IsA("DitheringEffect") then
                if not e:GetAttribute("_RaptureOrigEnabled") then
                    e:SetAttribute("_RaptureOrigEnabled", e.Enabled)
                end
                e.Enabled = false
            end
        end)
    end
    for _, obj in ipairs(workspace:GetDescendants()) do
        applyAntiLagDerender(obj)
    end
    for _, descendant in pairs(workspace:GetDescendants()) do
        pcall(function()
            if descendant:IsA("ParticleEmitter") then
                descendant.Enabled = false
            elseif descendant:IsA("Decal") then
                descendant.Transparency = 1
            elseif descendant:IsA("BasePart") then
                local char = LP.Character
                if char and descendant:IsDescendantOf(char) then return end
                descendant.Material      = Enum.Material.Plastic
                descendant.Reflectance   = 0
                descendant.CastShadow    = false
                descendant.TopSurface    = Enum.SurfaceType.Smooth
                descendant.BottomSurface = Enum.SurfaceType.Smooth
                descendant.FrontSurface  = Enum.SurfaceType.Smooth
                descendant.BackSurface   = Enum.SurfaceType.Smooth
                descendant.LeftSurface   = Enum.SurfaceType.Smooth
                descendant.RightSurface  = Enum.SurfaceType.Smooth
            end
        end)
    end
    pcall(function()
        _antiLagStored.terrainWater = {
            WaveSize     = workspace.Terrain.WaterWaveSize,
            WaveSpeed    = workspace.Terrain.WaterWaveSpeed,
            Reflectance  = workspace.Terrain.WaterReflectance,
            Transparency = workspace.Terrain.WaterTransparency,
            Decoration   = workspace.Terrain.Decoration,
        }
        workspace.Terrain.WaterWaveSize     = 0
        workspace.Terrain.WaterWaveSpeed    = 0
        workspace.Terrain.WaterReflectance  = 0
        workspace.Terrain.WaterTransparency = 0
        workspace.Terrain.Decoration        = false
    end)
    if antiLagDescConn then antiLagDescConn:Disconnect() end
    antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
        if obj:GetAttribute("_AmbitiousHubSky") then return end
        if obj:IsA("Sky") or obj:IsA("Atmosphere") or obj:IsA("Clouds") then return end
        if removeAccessoriesEnabled then applyAntiLagDerender(obj) end
    end)
end

function disableAntiLag()
    antiLagEnabled           = false
    removeAccessoriesEnabled = false
    if antiLagDescConn then
        antiLagDescConn:Disconnect()
        antiLagDescConn = nil
    end
    pcall(function()
        if _antiLagStored.qualityLevel then
            settings().Rendering.QualityLevel = _antiLagStored.qualityLevel
        end
        if _antiLagStored.meshDetail then
            settings().Rendering.MeshPartDetailLevel = _antiLagStored.meshDetail
        end
    end)
    pcall(function()
        local L = _antiLagStored.lighting
        if L then
            if L.Brightness ~= nil then Lighting.Brightness = L.Brightness end
            if L.ClockTime ~= nil then Lighting.ClockTime = L.ClockTime end
            if L.OutdoorAmbient ~= nil then Lighting.OutdoorAmbient = L.OutdoorAmbient end
            if L.Ambient ~= nil then Lighting.Ambient = L.Ambient end
            if L.GlobalShadows ~= nil then Lighting.GlobalShadows = L.GlobalShadows end
            if L.FogEnd ~= nil then Lighting.FogEnd = L.FogEnd end
            if L.FogStart ~= nil then Lighting.FogStart = L.FogStart end
            if L.EnvironmentDiffuseScale ~= nil then Lighting.EnvironmentDiffuseScale = L.EnvironmentDiffuseScale end
            if L.EnvironmentSpecularScale ~= nil then Lighting.EnvironmentSpecularScale = L.EnvironmentSpecularScale end
            if L.ShadowSoftness ~= nil then Lighting.ShadowSoftness = L.ShadowSoftness end
            if L.ExposureCompensation ~= nil then Lighting.ExposureCompensation = L.ExposureCompensation end
        end
        if _antiLagStored.technology then
            pcall(function() Lighting.Technology = _antiLagStored.technology end)
        end
    end)
    pcall(function()
        for _, e in pairs(Lighting:GetChildren()) do
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect")
            or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect")
            or e:IsA("DepthOfFieldEffect") or e:IsA("BrightnessEffect")
            or e:IsA("DitheringEffect") then
                local orig = e:GetAttribute("_RaptureOrigEnabled")
                if orig ~= nil then
                    e.Enabled = orig
                    e:SetAttribute("_RaptureOrigEnabled", nil)
                end
            end
        end
    end)
    pcall(function()
        local W = _antiLagStored.terrainWater
        if W then
            if W.WaveSize ~= nil then workspace.Terrain.WaterWaveSize = W.WaveSize end
            if W.WaveSpeed ~= nil then workspace.Terrain.WaterWaveSpeed = W.WaveSpeed end
            if W.Reflectance ~= nil then workspace.Terrain.WaterReflectance = W.Reflectance end
            if W.Transparency ~= nil then workspace.Terrain.WaterTransparency = W.Transparency end
            if W.Decoration ~= nil then workspace.Terrain.Decoration = W.Decoration end
        end
    end)
    pcall(function()
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                local char = LP.Character
                if char and part:IsDescendantOf(char) then continue end
            elseif part:IsA("ParticleEmitter") then
            elseif part:IsA("PointLight") or part:IsA("SpotLight")
                or part:IsA("SurfaceLight") then
                pcall(function() part.Enabled = true end)
            end
        end
    end)
    _antiLagStored = {
        lighting = {},
        qualityLevel = nil,
        meshDetail = nil,
        terrainWater = nil,
        technology = nil,
    }
end

function setupSpeedIndicator(char)
    local head = char:WaitForChild("Head", 5)
    if not head then return end
    local oldBB = head:FindFirstChild("RaptureDuelSpeedIndicator")
    if oldBB then oldBB:Destroy() end
    local bb = Instance.new("BillboardGui", head)
    bb.Name = "RaptureDuelSpeedIndicator"
    bb.Size = UDim2.new(0, 220, 0, 140)
    bb.StudsOffset = _V3new(0, 3.6, 0)
    bb.AlwaysOnTop = true
    raptureLinkLabel = Instance.new("TextLabel", bb)
    raptureLinkLabel.Name = "RaptureLinkLabel"
    raptureLinkLabel.Size = UDim2.new(1, 0, 0, 30)
    raptureLinkLabel.Position = UDim2.new(0, 0, 0, 0)
    raptureLinkLabel.BackgroundTransparency = 1
    raptureLinkLabel.Text = ".gg/raptureduel"
    raptureLinkLabel.TextColor3 = getThemeColor()
    raptureLinkLabel.Font = Enum.Font.GothamBlack
    raptureLinkLabel.TextSize = 28
    raptureLinkLabel.TextStrokeTransparency = 0
    raptureLinkLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    raptureLinkLabel.TextXAlignment = Enum.TextXAlignment.Center
    raptureLinkLabel.ZIndex = 5
    bestScripLabel = Instance.new("TextLabel", bb)
    bestScripLabel.Name = "BestScripLabel"
    bestScripLabel.Size = UDim2.new(1, 0, 0, 22)
    bestScripLabel.Position = UDim2.new(0, 0, 0, 34)
    bestScripLabel.BackgroundTransparency = 1
    bestScripLabel.Text = "Best Script Free"
    bestScripLabel.TextColor3 = getThemeColor()
    bestScripLabel.Font = Enum.Font.GothamBold
    bestScripLabel.TextSize = 18
    bestScripLabel.TextStrokeTransparency = 0
    bestScripLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    bestScripLabel.TextXAlignment = Enum.TextXAlignment.Center
    bestScripLabel.ZIndex = 5
    speedLabel = Instance.new("TextLabel", bb)
    speedLabel.Name = "SpeedLabel"
    speedLabel.Size = UDim2.new(1, 0, 0, 28)
    speedLabel.Position = UDim2.new(0, 0, 0, 60)
    speedLabel.BackgroundTransparency = 1
    speedLabel.Text = "Speed: " .. string.format("%.1f", getActiveMoveSpeed())
    speedLabel.TextColor3 = getThemeColor()
    speedLabel.Font = Enum.Font.GothamBold
    speedLabel.TextSize = 22
    speedLabel.TextStrokeTransparency = 0
    speedLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    speedLabel.ZIndex = 5
    speedLabel.Visible = showSpeedLabel
    ragdollCountdownLabel = Instance.new("TextLabel", bb)
    ragdollCountdownLabel.Name = "RagdollCountdown"
    ragdollCountdownLabel.Size = UDim2.new(1, 0, 0, 26)
    ragdollCountdownLabel.Position = UDim2.new(0, 0, 0, 96)
    ragdollCountdownLabel.BackgroundTransparency = 1
    ragdollCountdownLabel.Text = ""
    ragdollCountdownLabel.Visible = false
    ragdollCountdownLabel.TextColor3 = getThemeColor()
    ragdollCountdownLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    ragdollCountdownLabel.TextStrokeTransparency = 0
    ragdollCountdownLabel.Font = Enum.Font.GothamBlack
    ragdollCountdownLabel.TextSize = 22
    ragdollCountdownLabel.TextXAlignment = Enum.TextXAlignment.Center
    ragdollCountdownLabel.ZIndex = 10
    ragdollCountdownLabel.Parent = bb
    updateSpeedDisplayText()
end

local unwalkSavedAnimate = nil

function startUnwalk()
    local c = LP.Character
    if not c then return end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if hum then
        for _, t in ipairs(hum:GetPlayingAnimationTracks()) do pcall(function() t:Stop() end) end
    end
    local anim = c:FindFirstChild("Animate")
    if anim then unwalkSavedAnimate = anim:Clone(); anim:Destroy() end
end

function stopUnwalk()
    local c = LP.Character
    if c then
        local existing = c:FindFirstChild("Animate")
        if not existing then
            local src = game:GetService("StarterPlayer"):FindFirstChildOfClass("StarterCharacterScripts")
            local starterAnim = src and src:FindFirstChild("Animate")
            if starterAnim then starterAnim:Clone().Parent = c
            elseif unwalkSavedAnimate then unwalkSavedAnimate:Clone().Parent = c end
        end
    end
    unwalkSavedAnimate = nil
end

function refreshSpeedModeLabel()
    if setCarryModeVisual then setCarryModeVisual(speedMode) end
    if setLaggerModeVisual then setLaggerModeVisual(laggerToggled) end
    if setLaggerCarryVisual then setLaggerCarryVisual(laggerCarryToggled) end
    updateSpeedDisplayText()
end

function resetMovementState()
    refreshSpeedModeLabel()
    if mobSetCarry then mobSetCarry(speedMode) end
    if mobSetLagger1 then mobSetLagger1(laggerToggled) end
    if mobSetLagger2 then mobSetLagger2(laggerCarryToggled) end
end

function toggleCarryMode()
    if speedMode then speedMode = false
    else speedMode = true; laggerToggled = false; laggerCarryToggled = false end
    resetMovementState()
    saveAllSettings()
end

function toggleLaggerMode()
    if laggerToggled then laggerToggled = false
    else laggerToggled = true; speedMode = false; laggerCarryToggled = false end
    resetMovementState()
    saveAllSettings()
end

function toggleLaggerCarryMode()
    if laggerCarryToggled then laggerCarryToggled = false
    else laggerCarryToggled = true; speedMode = false; laggerToggled = false end
    resetMovementState()
    saveAllSettings()
end

function toggleLaggerCycle()
    if speedMode then
        speedMode = false; laggerToggled = true; laggerCarryToggled = false
    elseif laggerToggled then
        speedMode = false; laggerToggled = false; laggerCarryToggled = true
    else
        speedMode = true; laggerToggled = false; laggerCarryToggled = false
    end
    resetMovementState()
    saveAllSettings()
end

function stopAutoLeft()
    if alConn then alConn:Disconnect(); alConn = nil end
    alPhase = 1
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:Move(_V3zero, false) end
    end
    if autoLeftSetVisual then autoLeftSetVisual(false) end
    if mobSetAutoLeft then mobSetAutoLeft(false) end
end

function startAutoLeft()
    if autoRightEnabled then
        autoRightEnabled = false; stopAutoRight()
        if autoRightSetVisual then autoRightSetVisual(false) end
        if mobSetAutoRight then mobSetAutoRight(false) end
    end
    disableAllAimbots()
    if alConn then alConn:Disconnect() end
    alPhase = 1
    alConn = RunService.Heartbeat:Connect(function()
        if not autoLeftEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        local spd = NS
        if alPhase == 1 then
            local tgt = _V3new(AP.L1.X, root.Position.Y, AP.L1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                alPhase = 2
                local d = AP.L2 - root.Position
                local mv = _V3new(d.X, 0, d.Z).Unit
                hum:Move(mv, false)
                root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = AP.L1 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif alPhase == 2 then
            local tgt = _V3new(AP.L2.X, root.Position.Y, AP.L2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum:Move(_V3zero, false)
                root.AssemblyLinearVelocity = _V3zero
                autoLeftEnabled = false
                if alConn then alConn:Disconnect(); alConn = nil end
                alPhase = 1
                if autoLeftSetVisual then autoLeftSetVisual(false) end
                if mobSetAutoLeft then mobSetAutoLeft(false) end
                local facePos = _V3new(AP.L_FACE.X, root.Position.Y, AP.L_FACE.Z)
                if (facePos - root.Position).Magnitude > 0.01 then
                    root.CFrame = _CFnew(root.Position, facePos)
                end
                return
            end
            local d = AP.L2 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

function stopAutoRight()
    if arConn then arConn:Disconnect(); arConn = nil end
    arPhase = 1
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:Move(_V3zero, false) end
    end
    if autoRightSetVisual then autoRightSetVisual(false) end
    if mobSetAutoRight then mobSetAutoRight(false) end
end

function startAutoRight()
    if autoLeftEnabled then
        autoLeftEnabled = false; stopAutoLeft()
        if autoLeftSetVisual then autoLeftSetVisual(false) end
        if mobSetAutoLeft then mobSetAutoLeft(false) end
    end
    disableAllAimbots()
    if arConn then arConn:Disconnect() end
    arPhase = 1
    arConn = RunService.Heartbeat:Connect(function()
        if not autoRightEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        local spd = NS
        if arPhase == 1 then
            local tgt = _V3new(AP.R1.X, root.Position.Y, AP.R1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                arPhase = 2
                local d = AP.R2 - root.Position
                local mv = _V3new(d.X, 0, d.Z).Unit
                hum:Move(mv, false)
                root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = AP.R1 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif arPhase == 2 then
            local tgt = _V3new(AP.R2.X, root.Position.Y, AP.R2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum:Move(_V3zero, false)
                root.AssemblyLinearVelocity = _V3zero
                autoRightEnabled = false
                if arConn then arConn:Disconnect(); arConn = nil end
                arPhase = 1
                if autoRightSetVisual then autoRightSetVisual(false) end
                if mobSetAutoRight then mobSetAutoRight(false) end
                local facePos = _V3new(AP.R_FACE.X, root.Position.Y, AP.R_FACE.Z)
                if (facePos - root.Position).Magnitude > 0.01 then
                    root.CFrame = _CFnew(root.Position, facePos)
                end
                return
            end
            local d = AP.R2 - root.Position
            local mv = _V3new(d.X, 0, d.Z).Unit
            hum:Move(mv, false)
            root.AssemblyLinearVelocity = _V3new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

function getClosestTarget()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local rpos = root.Position
    local closest, minDist = nil, _huge
    local plist = _GetPlayersCached()
    for i = 1, #plist do
        local plr = plist[i]
        if plr ~= LP then
            local c = plr.Character
            if c then
                local tRoot = c:FindFirstChild("HumanoidRootPart")
                if tRoot then
                    local hum = c:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local dx = tRoot.Position.X - rpos.X
                        local dy = tRoot.Position.Y - rpos.Y
                        local dz = tRoot.Position.Z - rpos.Z
                        local d = dx*dx + dy*dy + dz*dz
                        if d < minDist then minDist = d; closest = tRoot end
                    end
                end
            end
        end
    end
    return closest
end

function trySwing()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local bat = findBat()
        if bat then
            if bat.Parent ~= char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(bat) end) end
            end
            pcall(function() bat:Activate() end)
        end
    end)
end

function stopAimbotAdapt()
    if _aimbotConn then pcall(function() _aimbotConn:Disconnect() end); _aimbotConn = nil end
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = (_prevAutoRotate == nil) and true or _prevAutoRotate
        hum.PlatformStand = false
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    end
    if root then
        root.AssemblyLinearVelocity = _V3new(0, -0.1, 0)
        root.AssemblyAngularVelocity = _V3zero
    end
    _prevAutoRotate = nil
    lastMoveDir = _V3zero
end

function startAimbotAdapt()
    if _aimbotConn then return end
    local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum0 then
        if _prevAutoRotate == nil then _prevAutoRotate = hum0.AutoRotate end
        hum0.AutoRotate = false
    end
    _aimbotConn = RunService.RenderStepped:Connect(function()
        if not autoBatEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        if not char:FindFirstChildOfClass("Tool") then
            local bat = findBat()
            if bat then pcall(function() hum:EquipTool(bat) end) end
        end
        local target = getClosestTarget()
        if not target then return end
        local myPos = root.Position
        local targetPos = target.Position
        local targetVel = target.AssemblyLinearVelocity
        local speedMult, predictAmount, heightOffset
        if aimbotMode == "V2" then
            speedMult = BAT_AIMBOT_SPEED * 1.05
            predictAmount = 0.22
            heightOffset = 3.2
        else
            speedMult = BAT_AIMBOT_SPEED
            predictAmount = 0.14
            heightOffset = 3.7
        end
        local predictPos = targetPos + targetVel * predictAmount
        predictPos = predictPos + target.CFrame.LookVector * 0.3
        local direction = predictPos - myPos
        local flatDir = _V3new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0 then flatDir = flatDir.Unit else flatDir = _V3new(0,0,0) end
        local desiredHeight = targetPos.Y + heightOffset
        local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
        if hum.FloorMaterial ~= Enum.Material.Air then yVel = math.max(yVel, 13) end
        yVel = _clamp(yVel, -70, 110)
        local desiredVel = _V3new(flatDir.X * speedMult, yVel, flatDir.Z * speedMult)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)
        local speed3 = targetVel.Magnitude
        local predictTime = _clamp(speed3 / 150, 0.05, 0.2)
        local predictedPos = targetPos + targetVel * predictTime
        local toPredict = predictedPos - myPos
        if toPredict.Magnitude > 0.1 then
            local goalCF = _CFlookAt(myPos, predictedPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = _clamp(rx, -2.5, 2.5); ry = _clamp(ry, -2.5, 2.5); rz = _clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(_V3new(rx * 42, ry * 42, rz * 42))
        end        local distToTarget = (root.Position - target.Position).Magnitude
        if distToTarget <= 8 then trySwing() end
    end)
end

function disableAutoBat()
    autoBatEnabled = false
    if autoBatSetVisual then autoBatSetVisual(false) end
    if mobSetAutoBat then mobSetAutoBat(false) end
    stopAimbotAdapt()
    if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
end

function enableAutoBat()
    if autoLeftEnabled then
        autoLeftEnabled = false
        if autoLeftSetVisual then autoLeftSetVisual(false) end
        stopAutoLeft()
    end
    if autoRightEnabled then
        autoRightEnabled = false
        if autoRightSetVisual then autoRightSetVisual(false) end
        stopAutoRight()
    end
    if batDesyncTpEnabled then toggleBatDesyncTp() end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
    autoBatEnabled = true
    if autoBatSetVisual then autoBatSetVisual(true) end
    if mobSetAutoBat then mobSetAutoBat(true) end
    startAimbotAdapt()
    if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
end

batDesyncTpEnabled = false
batDesyncTpSetVisual = nil
local _tpBatUnwalkForced = false

local batDesyncTpConn = nil
local hittingCooldownDesync = false

local function getBatDesync()
    local char = LP.Character
    if not char then return nil end
    local tool = char:FindFirstChild("Bat")
    if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then
        tool = bp2:FindFirstChild("Bat")
        if tool then tool.Parent = char; return tool end
    end
    return nil
end

local function tryHitBatDesync()
    if hittingCooldownDesync then return end
    hittingCooldownDesync = true
    pcall(function()
        local bat = getBatDesync()
        if bat then
            bat:Activate()
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then ev:FireServer() end
        end
    end)
    task.delay(0.08, function() hittingCooldownDesync = false end)
end

local function getClosestPlayerDesync()
    local char = LP.Character
    if not char then return nil, math.huge end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then
                local d = (hrp.Position - tr.Position).Magnitude
                if d < cd then cd = d; cp = p end
            end
        end
    end
    return cp, cd
end

local function batDesyncTpUpdate()
    if not batDesyncTpEnabled then
        if batDesyncTpConn then batDesyncTpConn:Disconnect(); batDesyncTpConn = nil end
        return
    end
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local target, dist = getClosestPlayerDesync()
    if target and target.Character then
        local tr = target.Character:FindFirstChild("HumanoidRootPart")
        if tr then
            if sethiddenproperty then
                pcall(sethiddenproperty, hrp, "PhysicsRepRootPart", tr)
            end
            local targetPos = tr.Position + Vector3.new(0, 0.9, 0)
            if (hrp.Position - targetPos).Magnitude > 8 then
                hrp.CFrame = CFrame.new(targetPos)
            end
            local cam = workspace.CurrentCamera
            if cam then
                cam.CFrame = CFrame.new(cam.CFrame.Position, tr.Position)
            end
            tryHitBatDesync()
        end
    end
end

function startBatDesyncTp()
    if batDesyncTpConn then return end
    batDesyncTpEnabled = true
    if not unwalkEnabled then
        startUnwalk()
        unwalkEnabled = true
        _tpBatUnwalkForced = true
        if setUnwalkVisual then setUnwalkVisual(true) end
    end
    batDesyncTpConn = RunService.Heartbeat:Connect(batDesyncTpUpdate)
    if batDesyncTpSetVisual then batDesyncTpSetVisual(true) end
    if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
end

function stopBatDesyncTp()
    if batDesyncTpConn then batDesyncTpConn:Disconnect(); batDesyncTpConn = nil end
    batDesyncTpEnabled = false
    if _tpBatUnwalkForced then
        stopUnwalk()
        unwalkEnabled = false
        _tpBatUnwalkForced = false
        if setUnwalkVisual then setUnwalkVisual(false) end
    end
    if batDesyncTpSetVisual then batDesyncTpSetVisual(false) end
    if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
end

function toggleBatDesyncTp()
    if batDesyncTpEnabled then
        stopBatDesyncTp()
    else
        disableAllAimbots()
        if autoLeftEnabled then
            autoLeftEnabled = false; stopAutoLeft()
            if autoLeftSetVisual then autoLeftSetVisual(false) end
            if mobSetAutoLeft then mobSetAutoLeft(false) end
        end
        if autoRightEnabled then
            autoRightEnabled = false; stopAutoRight()
            if autoRightSetVisual then autoRightSetVisual(false) end
            if mobSetAutoRight then mobSetAutoRight(false) end
        end
        startBatDesyncTp()
    end
    if batDesyncTpSetVisual then batDesyncTpSetVisual(batDesyncTpEnabled) end
    saveAllSettings()
end

LP.CharacterAdded:Connect(function(char)
    task.wait(0.15)
    if batDesyncTpEnabled and not batDesyncTpConn then
        startBatDesyncTp()
    end
end)

-- ============================================================
-- BAT BYPASS (versión de "Bypass mejorado rapture.txt")
-- ============================================================
batBypassEnabled = batBypassEnabled or false
batBypassSetVisual = nil
batBypassFloatingButton = nil
batBypassFloatingPos = nil

_G._RaptureBatBypass = _G._RaptureBatBypass or {
    enabled = false, conn = nil, safetyConn = nil, target = nil, equipped = false,
    intendedVelocity = Vector3.zero, attachment = nil, angularVelocity = nil,
    speed = 58, swingRange = 12,
}

local BAT_BYPASS_ANG = "_RaptureBypassAngV"
local BAT_BYPASS_ATT = "_RaptureBypassAtt"
local BAT_BYPASS_CLEAN = {
    "LockBAV","LockAngVel","LockBodyAtt","BatLock","LockBAVAtt",
    "AutoBatAtt","AutoBatAV","AntiBatDet","AntiAim","VelocityLock"
}

local function _batBypassCleanMovers(root)
    if not root then return end
    pcall(function()
        for _, n in ipairs(BAT_BYPASS_CLEAN) do
            local c = root:FindFirstChild(n)
            if c then c:Destroy() end
        end
    end)
end

local function _batBypassEnsureAngular(root)
    if not root then return end
    local B = _G._RaptureBatBypass
    pcall(function()
        local a = root:FindFirstChild(BAT_BYPASS_ANG); if a then a:Destroy() end
        local b = root:FindFirstChild(BAT_BYPASS_ATT); if b then b:Destroy() end
    end)
    local att = Instance.new("Attachment")
    att.Name = BAT_BYPASS_ATT
    att.Parent = root
    local angV = Instance.new("AngularVelocity")
    angV.Name = BAT_BYPASS_ANG
    angV.Attachment0 = att
    angV.RelativeTo = Enum.ActuatorRelativeTo.World
    angV.MaxTorque = math.huge
    angV.AngularVelocity = Vector3.zero
    angV.Parent = root
    B.attachment = att
    B.angularVelocity = angV
end

local function _batBypassRemoveAngular()
    local B = _G._RaptureBatBypass
    pcall(function() if B.angularVelocity and B.angularVelocity.Parent then B.angularVelocity:Destroy() end end)
    pcall(function() if B.attachment and B.attachment.Parent then B.attachment:Destroy() end end)
    B.angularVelocity = nil
    B.attachment = nil
end

local function _batBypassPickTarget(root)
    local best, bestD = nil, math.huge
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local d = (hrp.Position - root.Position).Magnitude
                if d < bestD then bestD = d; best = hrp end
            end
        end
    end
    return best, bestD
end

local function _batBypassGetBat(char)
    local eq = char and char:FindFirstChildOfClass("Tool")
    if eq then
        local n = eq.Name:lower()
        if n:find("bat") or n:find("slap") then return eq end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") then
                local n = t.Name:lower()
                if n:find("bat") or n:find("slap") then return t end
            end
        end
    end
    return nil
end

local function _batBypassIsBatName(name)
    if type(name) ~= "string" then return false end
    local n = name:lower()
    return (n:find("bat") or n:find("slap")) and true or false
end

local function _batBypassUnequipAllOtherTools(char, hum)
    if not char or not hum then return end
    for _, t in ipairs(char:GetChildren()) do
        if t:IsA("Tool") and not _batBypassIsBatName(t.Name) then
            pcall(function() hum:UnequipTools() end)
            break
        end
    end
end

local function _batBypassEnsureBatEquipped(char, hum)
    if not char or not hum then return end
    -- Si hay una tool equipada que no es bat, desequipar todo
    local equipped = char:FindFirstChildOfClass("Tool")
    if equipped and not _batBypassIsBatName(equipped.Name) then
        pcall(function() hum:UnequipTools() end)
        equipped = nil
    end
    -- Si ya hay un bat equipado, no hacer nada
    if equipped and _batBypassIsBatName(equipped.Name) then
        return equipped
    end
    -- Buscar bat y equiparlo
    local bat = _batBypassGetBat(char)
    if bat then
        if bat.Parent ~= char then
            pcall(function() hum:EquipTool(bat) end)
        end
        return bat
    end
    return nil
end

function _G.RaptureStopBatBypass(keepVisual)
    local B = _G._RaptureBatBypass
    B.enabled = false
    B.equipped = false
    B.target = nil
    if B.conn then B.conn:Disconnect(); B.conn = nil end
    if B.safetyConn then B.safetyConn:Disconnect(); B.safetyConn = nil end
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then pcall(function() root.Velocity = root.Velocity * 0.3 end) end
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then hum.AutoRotate = true end
    if B.angularVelocity then pcall(function() B.angularVelocity.AngularVelocity = Vector3.zero end) end
    _batBypassRemoveAngular()
    if keepVisual ~= false then
        batBypassEnabled = false
        if batBypassSetVisual then batBypassSetVisual(false) end
        if batBypassFloatingButton then
            local f = batBypassFloatingButton:FindFirstChild("Frame")
            if f then applyNeonPinkStyle(f, false) end
        end
        if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
    end
end

function _G.RaptureStartBatBypass()
    local B = _G._RaptureBatBypass
    _G.RaptureStopBatBypass(false)
    B.enabled = true
    B.equipped = false
    B.target = nil
    B.intendedVelocity = Vector3.zero
    B.speed = BAT_AIMBOT_SPEED or 58
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then
        _batBypassCleanMovers(root)
        _batBypassEnsureAngular(root)
    end
    B.conn = RunService.Heartbeat:Connect(function()
        if not B.enabled then return end
        local char = LP.Character
        local hum  = char and char:FindFirstChildOfClass("Humanoid")
        local hrp  = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp or not hum or hum.Health <= 0 then return end
        _batBypassCleanMovers(hrp)
        if not B.angularVelocity or not B.angularVelocity.Parent then _batBypassEnsureAngular(hrp) end
        -- Detectar si la tool equipada no es bat para forzar re-equipar
        local currentTool = char:FindFirstChildOfClass("Tool")
        local batEquipped = currentTool and _batBypassIsBatName(currentTool.Name)
        if not batEquipped then
            B.equipped = false
        end
        if not B.equipped then
            B.equipped = true
            _batBypassUnequipAllOtherTools(char, hum)
            _batBypassEnsureBatEquipped(char, hum)
        end
        local target, dist = _batBypassPickTarget(hrp)
        if not target then
            B.target = nil
            hum.AutoRotate = true
            if B.angularVelocity then B.angularVelocity.AngularVelocity = Vector3.zero end
            return
        end
        B.target = target
        local aimPos = target.Position + target.CFrame.LookVector * (target.Velocity.Magnitude < 0.1 and 1.5 or 5)
        local delta = aimPos - hrp.Position
        local flat  = Vector3.new(delta.X, 0, delta.Z)
        hum.AutoRotate = false
        if delta.Magnitude > 0.01 and flat.Magnitude > 0.01 then
            local curY   = hrp.Orientation.Y
            local yawD   = (math.deg(math.atan2(-flat.X, -flat.Z)) - curY + 180) % 360 - 180
            local curX   = hrp.Orientation.X
            local pitchD = (math.deg(math.atan2(delta.Y, flat.Magnitude)) - curX + 180) % 360 - 180
            local rotY   = math.clamp(math.rad(yawD)   * 40, -28, 28)
            local rotX   = math.clamp(math.rad(pitchD) * 40, -28, 28)
            local yawR   = math.rad(hrp.Orientation.Y)
            local fwd    = Vector3.new(math.cos(yawR), 0, -math.sin(yawR))
            B.angularVelocity.AngularVelocity = Vector3.new(0, rotY, 0) + fwd * rotX
        else
            B.angularVelocity.AngularVelocity = Vector3.zero
        end
        local spd = tonumber(B.speed) or 58
        B.intendedVelocity = (flat.Magnitude > 0.1 and flat.Unit * spd or Vector3.zero)
            + (math.abs(delta.Y) > 0.8 and Vector3.new(0, math.sign(delta.Y) * spd, 0) or Vector3.new(0, -2, 0))
        hrp.AssemblyLinearVelocity = B.intendedVelocity
        hrp.Velocity = B.intendedVelocity
        if flat.Magnitude > 0.3 then pcall(function() hum:Move(flat.Unit, false) end)
        else pcall(function() hum:Move(Vector3.zero, false) end) end
        if dist <= B.swingRange then
            local bat = char:FindFirstChildOfClass("Tool")
            if bat and _batBypassIsBatName(bat.Name) then
                pcall(function()
                    bat:Activate()
                    local re = bat:FindFirstChildWhichIsA("RemoteEvent")
                    if re then re:FireServer() end
                end)
            end
        end
    end)
    B.safetyConn = RunService.RenderStepped:Connect(function()
        if not B.enabled then return end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        if B.target and B.target.Parent then
            local delta = B.target.Position - hrp.Position
            local flat = Vector3.new(delta.X, 0, delta.Z)
            if flat.Magnitude > 0.3 then pcall(function() hum:Move(flat.Unit, false) end) end
            if math.abs(hrp.AssemblyLinearVelocity.X - B.intendedVelocity.X) > 30
               or math.abs(hrp.AssemblyLinearVelocity.Z - B.intendedVelocity.Z) > 30 then
                hrp.AssemblyLinearVelocity = B.intendedVelocity
            end
        end
        local v = hrp.Velocity
        if math.abs(v.X) > 350 or math.abs(v.Z) > 350 then hrp.Velocity = B.intendedVelocity end
        _batBypassCleanMovers(hrp)
    end)
    batBypassEnabled = true
    if batBypassSetVisual then batBypassSetVisual(true) end
    if batBypassFloatingButton then
        local f = batBypassFloatingButton:FindFirstChild("Frame")
        if f then applyNeonPinkStyle(f, true) end
    end
    if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
end

function _G.RaptureToggleBatBypass()
    if _G._RaptureBatBypass.enabled then
        _G.RaptureStopBatBypass()
    else
        disableAllAimbots()
        if autoLeftEnabled then
            autoLeftEnabled = false; stopAutoLeft()
            if autoLeftSetVisual then autoLeftSetVisual(false) end
            if mobSetAutoLeft then mobSetAutoLeft(false) end
        end
        if autoRightEnabled then
            autoRightEnabled = false; stopAutoRight()
            if autoRightSetVisual then autoRightSetVisual(false) end
            if mobSetAutoRight then mobSetAutoRight(false) end
        end
        _G.RaptureStartBatBypass()
    end
    saveAllSettings()
end

LP.CharacterAdded:Connect(function()
    task.wait(0.5)
    if _G._RaptureBatBypass.enabled then
        _G._RaptureBatBypass.equipped = false
        _G._RaptureBatBypass.target = nil
        local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            _batBypassCleanMovers(hrp)
            _batBypassEnsureAngular(hrp)
        end
    end
end)

function findBat()
    local char = LP.Character
    if not char then return nil end
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local t = char:FindFirstChild(name)
        if t and t:IsA("Tool") then return t end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(t) end) end
                return t
            end
        end
    end
    for _, ch in ipairs(char:GetChildren()) do
        if ch:IsA("Tool") and (ch.Name:lower():find("bat") or ch.Name:lower():find("slap")) then
            return ch
        end
    end
    return nil
end

function findBatForCounter()
    local char = LP.Character
    if not char then return nil end
    local backpack = LP:FindFirstChildOfClass("Backpack")
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local tool = char:FindFirstChild(name) or (backpack and backpack:FindFirstChild(name))
        if tool then return tool end
    end
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Tool") and (child.Name:lower():find("bat") or child.Name:lower():find("slap")) then
            return child
        end
    end
    return nil
end

function swingBatForCounter(bat, character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if bat.Parent ~= character and humanoid then
        pcall(function() humanoid:EquipTool(bat) end)
        task.wait(0.05)
    end
    local remote = bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then
        pcall(function() remote:FireServer() end)
        task.wait(0.1)
        pcall(function() remote:FireServer() end)
    else
        pcall(function() bat:Activate() end)
        task.wait(0.1)
        pcall(function() bat:Activate() end)
    end
end

batCounterDebounce = false

function stopBatCounter()
    if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter = nil end
    batCounterDebounce = false
end

function startBatCounter()
    if Conns.batCounter then return end
    Conns.batCounter = RunService.Heartbeat:Connect(function()
        if not batCounterEnabled then return end
        if batCounterDebounce then return end
        local character = LP.Character
        if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        local state = humanoid:GetState()
        if state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown then
            batCounterDebounce = true
            task.spawn(function()
                task.wait(0.15)
                local bat = findBatForCounter()
                if bat then swingBatForCounter(bat, character) end
                task.wait(0.3)
                batCounterDebounce = false
            end)
        end
    end)
end

local DROP_ASCEND_DURATION = 0.22
local DROP_ASCEND_SPEED = 160
local _dropConn = nil

function stopDropBrainrot()
    dropActive = false
    if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
    for _, t in ipairs(dropConnections) do
        if type(t) == "thread" then pcall(task.cancel, t)
        elseif type(t) == "RBXScriptConnection" then pcall(t.Disconnect, t) end
    end
    dropConnections = {}
    local c = LP.Character
    if c then
        local root = c:FindFirstChild("HumanoidRootPart")
        if root then root.AssemblyLinearVelocity = _V3zero end
    end
    if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
    if mobSetDropBR then mobSetDropBR(false) end
end

function runDropBrainrot()
    if dropActive then return end
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end
    dropActive = true
    if dropBrainrotSetVisual then dropBrainrotSetVisual(true) end
    if mobSetDropBR then mobSetDropBR(true) end
    local t0 = _tick()
    if _dropConn then _dropConn:Disconnect() end
    _dropConn = RunService.Heartbeat:Connect(function()
        local c = LP.Character
        local r = c and c:FindFirstChild("HumanoidRootPart")
        if not r then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            dropActive = false
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
            return
        end
        if not dropActive then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
            return
        end
        if _tick() - t0 >= DROP_ASCEND_DURATION then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            pcall(function()
                local rp = RaycastParams.new()
                rp.FilterDescendantsInstances = {c}
                rp.FilterType = Enum.RaycastFilterType.Exclude
                local rr = workspace:Raycast(r.Position, _V3new(0, -3000, 0), rp)
                if rr then
                    local hum2 = c:FindFirstChildOfClass("Humanoid")
                    local off = ((hum2 and hum2.HipHeight) or 2) + (r.Size.Y / 2)
                    r.CFrame = _CFnew(r.Position.X, rr.Position.Y + off, r.Position.Z)
                    r.AssemblyLinearVelocity = _V3zero
                    r.AssemblyAngularVelocity = _V3zero
                end
                if hum2 and hum2.Health > 0 then hum2:ChangeState(Enum.HumanoidStateType.Running) end
            end)
            dropActive = false
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if mobSetDropBR then mobSetDropBR(false) end
            return
        end
        local lv = r.AssemblyLinearVelocity
        r.AssemblyLinearVelocity = _V3new(lv.X, DROP_ASCEND_SPEED, lv.Z)
    end)
end

function executeDropWithToggle(setVisual)
    if dropActive then return end
    task.spawn(function()
        if setVisual then setVisual(true) end
        runDropBrainrot()
        while dropActive do task.wait() end
        task.wait(0.1)
        if setVisual then setVisual(false) end
    end)
end

CUSTOM_FOV_BIND = "RaptureDuelCustomFOV"

function enableCustomFov()
    local cam = workspace.CurrentCamera
    if cam and origFOV == nil then origFOV = cam.FieldOfView end
    fovEnabled = true
    if cam then
        pcall(function() cam.FieldOfViewMode = Enum.FieldOfViewMode.Diagonal end)
        pcall(function() cam.FieldOfView = fovValue end)
    end
    if customFovConn then customFovConn:Disconnect(); customFovConn = nil end
    pcall(function() RunService:UnbindFromRenderStep(CUSTOM_FOV_BIND) end)
    local prio = 200
    pcall(function() prio = Enum.RenderPriority.Camera.Value + 10 end)
    local ok = pcall(function()
        RunService:BindToRenderStep(CUSTOM_FOV_BIND, prio, function()
            if not fovEnabled then return end
            local c = workspace.CurrentCamera
            if c and c.FieldOfView ~= fovValue then c.FieldOfView = fovValue end
        end)
    end)
    if not ok then
        customFovConn = RunService.RenderStepped:Connect(function()
            if not fovEnabled then
                if customFovConn then customFovConn:Disconnect(); customFovConn = nil end
                return
            end
            local c = workspace.CurrentCamera
            if c then c.FieldOfView = fovValue end
        end)
    end
end

function disableCustomFov()
    fovEnabled = false
    pcall(function() RunService:UnbindFromRenderStep(CUSTOM_FOV_BIND) end)
    if customFovConn then customFovConn:Disconnect(); customFovConn = nil end
    local cam = workspace.CurrentCamera
    if cam then
        pcall(function() cam.FieldOfViewMode = Enum.FieldOfViewMode.Vertical end)
        pcall(function() cam.FieldOfView = origFOV or 70 end)
    end
end

function enableStretch()
    if stretchConn then return end
    stretchEnabled = true
    local cam = workspace.CurrentCamera
    if not cam then return end
    origFOV = cam.FieldOfView or 70
    local val = tonumber(stretchValue) or 0.7
    stretchConn = RunService.RenderStepped:Connect(function()
        if not stretchEnabled then
            stretchConn:Disconnect(); stretchConn = nil; return
        end
        local c = workspace.CurrentCamera
        if c then c.CFrame = c.CFrame * _CFnew(0,0,0,1,0,0,0,val,0,0,0,1) end
    end)
end

function disableStretch()
    stretchEnabled = false
    if stretchConn then stretchConn:Disconnect(); stretchConn = nil end
    local cam = workspace.CurrentCamera
    if cam then pcall(function() cam.FieldOfView = origFOV or 70 end) end
end

function paintFloatingBtn(btnFrame, active)
    if not btnFrame then return end
    applyNeonPinkStyle(btnFrame, active)
end

function applyFloatingButtonScale()
    for _, uiScale in ipairs(_floatingUIScales) do
        if uiScale and uiScale.Parent then
            uiScale.Scale = floatingButtonScale
        end
    end
end

local function drag(f)
    local dn, ds, sp, di = false, nil, nil, nil
    local endConn = nil
    local function stopDrag()
        dn = false; di = nil
        if endConn then endConn:Disconnect(); endConn = nil end
    end
    f.InputBegan:Connect(function(i)
        if uiLocked then return end
        if _isDraggingButton then return end
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dn = true; ds = i.Position; sp = f.Position
            if endConn then endConn:Disconnect() end
            endConn = i.Changed:Connect(function()
                if i.UserInputState == Enum.UserInputState.End then stopDrag() end
            end)
        end
    end)
    f.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then stopDrag() end
    end)
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then stopDrag() end
    end)
    f.InputChanged:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then di = i end
    end)
    UIS.InputChanged:Connect(function(i)
        if i == di and dn then
            if uiLocked then stopDrag(); return end
            if _isDraggingButton then return end
            if not ds or not sp then return end
            local nX = sp.X.Offset + (i.Position.X - ds.X)
            local nY = sp.Y.Offset + (i.Position.Y - ds.Y)
            f.Position = UDim2.new(sp.X.Scale, nX, sp.Y.Scale, nY)
        end
    end)
end

function setupMovementAndIndicators(char)
    if steppedConn then steppedConn:Disconnect(); steppedConn = nil end
    if movementLoop then movementLoop:Disconnect(); movementLoop = nil end
    local ccAcc = 0
    steppedConn = RunService.Heartbeat:Connect(function(dt)
        ccAcc = ccAcc + dt
        if ccAcc < 0.05 then return end
        ccAcc = 0
        local plist = _GetPlayersCached()
        for i = 1, #plist do
            local p = plist[i]
            if p ~= LP then
                local ch = p.Character
                if ch then
                    local parts = ch:GetChildren()
                    for j = 1, #parts do
                        local part = parts[j]
                        if part:IsA("BasePart") and part.CanCollide then part.CanCollide = false end
                    end
                end
            end
        end
    end)
    movementLoop = RunService.Heartbeat:Connect(function()
        local char2 = LP.Character
        if not char2 then return end
        local hum = char2:FindFirstChildOfClass("Humanoid")
        local hrp = char2:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end
        if not autoBatEnabled and not autoLeftEnabled and not autoRightEnabled
           and not batDesyncTpEnabled
           and not (_G._RaptureBatBypass and _G._RaptureBatBypass.enabled) then
            if _isRagdollState(hum) then
                lastMoveDir = _V3zero
            else
                local md = hum.MoveDirection
                local spd = getActiveMoveSpeed()
                local dir = nil
                if md.Magnitude > 0 then
                    lastMoveDir = md
                    dir = md
                elseif lastMoveDir.Magnitude > 0 then
                    for key in pairs(MOVE_KEYS) do
                        if UIS:IsKeyDown(key) then dir = lastMoveDir; break end
                    end
                end
                _applyVelocitySpeed(dir, spd, hrp)
            end
        end
        if speedLabel then
            updateSpeedDisplayText()
        end
    end)
    setupSpeedIndicator(char)
    startEnemySpeed()
end

function toggleLockUI(state)
    if state == nil then uiLocked = not uiLocked else uiLocked = state end
    if setLockUIVisual then setLockUIVisual(uiLocked) end
    if lockBtn then lockBtn.Text = uiLocked and "🔒" or "🔓" end
    saveAllSettings()
end

function disableAllAimbots()
    if autoBatEnabled then
        disableAutoBat()
        if autoBatSetVisual then autoBatSetVisual(false) end
        if mobSetAutoBat then mobSetAutoBat(false) end
    end
    if batDesyncTpEnabled then stopBatDesyncTp() end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
end

function stopAllBackgroundTasks()
    if movementLoop then movementLoop:Disconnect(); movementLoop = nil end
    if steppedConn then steppedConn:Disconnect(); steppedConn = nil end
    stopEnemySpeed()
    if stretchEnabled then disableStretch() end
    if AntiRagdollV1.isRunning() then AntiRagdollV1.stop() end
    if AntiRagdollV2.Enabled then stopAntiRagdollV2() end
    if antiFlingEnabled then stopAntiFling() end
    if antiDieEnabled then _antiDie.StopEngine() end
    stopBatCounter()
    stopMedusaCounter()
    stopAutoSteal()
    disableAutoBat()
    if batDesyncTpEnabled then stopBatDesyncTp() end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
    stopAutoLeft()
    stopAutoRight()
    if _G._RaptureSpinStopSystem then _G._RaptureSpinStopSystem() end
    if unwalkEnabled and not _tpBatUnwalkForced then stopUnwalk() end
    if antiLagEnabled then disableAntiLag() end
    if dropActive then stopDropBrainrot() end
    if espEnabled then stopESP() end
    if bodyLockEnabled then stopBodyLock() end
    if _G.RapturePlatformDisable then pcall(_G.RapturePlatformDisable, false) end
    if syncAfterHitEnabled then
        syncAfterHitEnabled = false
        if _syncAfterHit.conn then pcall(function() _syncAfterHit.conn:Disconnect() end); _syncAfterHit.conn = nil end
    end
    if _G._XrayBaseSet then _G._XrayBaseSet(false) end
    if _G._XrayBrainrotSet then _G._XrayBrainrotSet(false) end
    dropActive = false
    alPhase = 1
    arPhase = 1
    lastDropTime = 0
    medusaDebounce = false
    medusaLastUsed = 0
end

function buildConfigTable()
    local config = {
        normalSpeed = NS, carrySpeed = CS,
        laggerSpeed1 = LAGGER_SPEED, laggerSpeed2 = LAGGER_CARRY_SPEED,
        stealRadius = CONFIG.STEAL_RANGE,
        antiRagdollEnabled = antiRagdollEnabled,
        antiRagdollVersion = antiRagdollVersion,
        antiFlingEnabled = antiFlingEnabled,
        antiDieEnabled = antiDieEnabled,
        infiniteJumpEnabled = InfiniteJump and InfiniteJump.enabled == true,
        skipIntro = skipIntroEnabled,
        autoSteal = CONFIG.AUTO_STEAL_ENABLED,
        medusaCounter = medusaCounterEnabled,
        batCounter = batCounterEnabled,
        ragdollCountdown = ragdollCountdownEnabled,
        laggerToggled = laggerToggled,
        laggerCarryToggled = laggerCarryToggled,
        carryMode = speedMode,
        batAimbotSpeed = BAT_AIMBOT_SPEED,
        aimbotMode = aimbotMode,
        dropMode = 2,
        stretchEnabled = stretchEnabled, stretchFOV = stretchFOV, stretchValue = stretchValue,
        fovEnabled = fovEnabled, fovValue = fovValue,
        uiScale = uiScaleValue, progressBarScale = progressBarScaleValue,
        tpBatEnabled = batDesyncTpEnabled,
        batBypassEnabled = batBypassEnabled,
        espEnabled = espEnabled, antiLag = antiLagEnabled,
        customSounds = customSoundsEnabled,
        soundButtonsEnabled = soundButtonsEnabled,
        soundTabEnabled = soundTabEnabled,
        soundToggleEnabled = soundToggleEnabled,
        soundArrowEnabled = soundArrowEnabled,
        syncAfterHit = syncAfterHitEnabled,
        bodyLockEnabled = bodyLockEnabled,
        bodyLockRange = bodyLockRange,
        spinEnabled = spinEnabled,
        spinSpeed = spinSpeed,
        spinAutoBrainrot = spinAutoBrainrotEnabled,
        unwalkEnabled = unwalkEnabled,
        xrayBaseEnabled = xrayBaseEnabled,
        xrayBrainrotEnabled = xrayBrainrotEnabled,
        speedDisplayMode = speedDisplayMode or "Speed and Mode",
        showSpeedLabel = showSpeedLabel ~= false,
        tpDownMode = tpDownMode or "Basic",
        platformEnabled = platformEnabled,
        platformSeconds = _Platform.AUTO_DISABLE_TIME,
        skyTheme = skyTheme,
        raptureSkinIndex = _G._RaptureSkinState and _G._RaptureSkinState.currentIndex or 5,
        raptureBatSkinIndex = _G._RaptureBatSkinState and _G._RaptureBatSkinState.currentIndex or 1,
        mobileButtonPositions = savedButtonPositions,
        hideMobileButtons = _G._RaptureHideMobileButtons == true,
        mobileHideList    = _G._RaptureMobileHideList,
        dropBrainrotKey = {kb = KB.DropBrainrot.kb and KB.DropBrainrot.kb.Name, gp = KB.DropBrainrot.gp and KB.DropBrainrot.gp.Name},
        autoLeftKey = {kb = KB.AutoLeft.kb and KB.AutoLeft.kb.Name, gp = KB.AutoLeft.gp and KB.AutoLeft.gp.Name},
        autoRightKey = {kb = KB.AutoRight.kb and KB.AutoRight.kb.Name, gp = KB.AutoRight.gp and KB.AutoRight.gp.Name},
        autoBatKey = {kb = KB.AutoBat.kb and KB.AutoBat.kb.Name, gp = KB.AutoBat.gp and KB.AutoBat.gp.Name},
        tpFloorKey = {kb = KB.TPFloor.kb and KB.TPFloor.kb.Name, gp = KB.TPFloor.gp and KB.TPFloor.gp.Name},
        carryToggleKey = {kb = KB.CarryToggle.kb and KB.CarryToggle.kb.Name, gp = KB.CarryToggle.gp and KB.CarryToggle.gp.Name},
        laggerModeKey = {kb = KB.LaggerMode.kb and KB.LaggerMode.kb.Name, gp = KB.LaggerMode.gp and KB.LaggerMode.gp.Name},
        tpBatKey = {kb = KB.TPBat.kb and KB.TPBat.kb.Name, gp = KB.TPBat.gp and KB.TPBat.gp.Name},
        instaResetKey = {kb = KB.InstaReset.kb and KB.InstaReset.kb.Name, gp = KB.InstaReset.gp and KB.InstaReset.gp.Name},
        batBypassKey = {kb = KB.BatBypass.kb and KB.BatBypass.kb.Name, gp = KB.BatBypass.gp and KB.BatBypass.gp.Name},
        spinKey = {kb = KB.Spin.kb and KB.Spin.kb.Name, gp = KB.Spin.gp and KB.Spin.gp.Name},
        platformKey = {kb = KB.Platform.kb and KB.Platform.kb.Name, gp = KB.Platform.gp and KB.Platform.gp.Name},
        tpBatFloatingPos = tpBatFloatingPos,
        batBypassFloatingPos = batBypassFloatingPos,
        instaResetFloatingPos = instaResetFloatingPos,
        platformFloatingPos = platformFloatingPos,
        progressBarPos = savedProgressBarPos, lockUI = uiLocked,
        backgroundIndex = backgroundIndex,
        backgroundImageTransparency = backgroundImageTransparency,
        floatingButtonScale = floatingButtonScale,
    }
    if pbFrame then
        config.progressBarPos = {
            XScale = pbFrame.Position.X.Scale, XOffset = pbFrame.Position.X.Offset,
            YScale = pbFrame.Position.Y.Scale, YOffset = pbFrame.Position.Y.Offset
        }
    end
    return config
end

function saveAllSettings()
    if _isResetting then return true end
    local config = buildConfigTable()
    local json = HS:JSONEncode(config)
    if json == _lastSavedJSON then return true end
    local success, err = pcall(function() writefile(CONFIG_FILE, json) end)
    if success then _lastSavedJSON = json end
    return success
end

function saveAmbitiousConfig() saveAllSettings() end

function loadAllSettings()
    if not isfile or not isfile(CONFIG_FILE) then return false end
    local success, data = pcall(function() return HS:JSONDecode(readfile(CONFIG_FILE)) end)
    if not success or not data then return false end
    _isLoading = true
    NS = data.normalSpeed or NS
    CS = data.carrySpeed or CS
    LAGGER_SPEED = data.laggerSpeed1 or LAGGER_SPEED
    LAGGER_CARRY_SPEED = data.laggerSpeed2 or LAGGER_CARRY_SPEED
    CONFIG.STEAL_RANGE = data.stealRadius or CONFIG.STEAL_RANGE
    stealMode = "V2"
    if radInput then radInput.Text = tostring(CONFIG.STEAL_RANGE) end
    uiLocked = data.lockUI or true
    antiRagdollEnabled = data.antiRagdollEnabled or false
    if data.antiRagdollVersion then
        antiRagdollVersion = data.antiRagdollVersion
        if antiRagdollVersion ~= "v2" then antiRagdollVersion = "v1" end
    else antiRagdollVersion = "v1" end
    antiFlingEnabled = data.antiFlingEnabled or false
    antiDieEnabled = data.antiDieEnabled or false
    if data.infiniteJumpEnabled ~= nil and InfiniteJump then
        if data.infiniteJumpEnabled then
            if InfiniteJump.start then InfiniteJump.start() end
        else
            if InfiniteJump.stop then InfiniteJump.stop() end
        end
    end
    if data.skipIntro ~= nil then skipIntroEnabled = data.skipIntro == true end
    CONFIG.AUTO_STEAL_ENABLED = data.autoSteal or false
    medusaCounterEnabled = data.medusaCounter or false
    batCounterEnabled = data.batCounter or false
    ragdollCountdownEnabled = data.ragdollCountdown or false
    laggerToggled = data.laggerToggled or false
    speedMode = data.carryMode or false
    laggerCarryToggled = data.laggerCarryToggled or false
    espEnabled = data.espEnabled or false
    antiLagEnabled = data.antiLag or false
    customSoundsEnabled = data.customSounds == true
    if data.soundButtonsEnabled ~= nil then soundButtonsEnabled = data.soundButtonsEnabled == true end
    if data.soundTabEnabled     ~= nil then soundTabEnabled     = data.soundTabEnabled     == true end
    if data.soundToggleEnabled  ~= nil then soundToggleEnabled  = data.soundToggleEnabled  == true end
    if data.soundArrowEnabled   ~= nil then soundArrowEnabled   = data.soundArrowEnabled   == true end
    if data.aimbotMode then
        aimbotMode = data.aimbotMode
        if aimbotMode ~= "V2" then aimbotMode = "V1" end
    else aimbotMode = "V1" end
    batBypassEnabled        = data.batBypassEnabled == true
    syncAfterHitEnabled     = data.syncAfterHit == true
    bodyLockEnabled = data.bodyLockEnabled == true
    bodyLockRange = data.bodyLockRange or bodyLockRange
    if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
    spinEnabled = data.spinEnabled == true
    spinSpeed = data.spinSpeed or spinSpeed
    spinAutoBrainrotEnabled = data.spinAutoBrainrot == true
    if spinSpeedBox then spinSpeedBox.Text = tostring(spinSpeed) end
    unwalkEnabled = data.unwalkEnabled == true
    xrayBaseEnabled = data.xrayBaseEnabled == true
    xrayBrainrotEnabled = data.xrayBrainrotEnabled == true
    if data.speedDisplayMode then
        speedDisplayMode = data.speedDisplayMode
    else
        speedDisplayMode = "Speed and Mode"
    end
    if data.showSpeedLabel ~= nil then
        showSpeedLabel = data.showSpeedLabel == true
    else
        showSpeedLabel = true
    end
    if data.tpDownMode then
        tpDownMode = data.tpDownMode
        if tpDownMode ~= "Best" then tpDownMode = "Basic" end
    else
        tpDownMode = "Basic"
    end
    _Platform.AUTO_DISABLE_TIME = data.platformSeconds or 1.9
    if platformSecondsBox then platformSecondsBox.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME) end
    skyTheme = data.skyTheme or "Off"
    uiScaleValue = data.uiScale or uiScaleValue
    progressBarScaleValue = data.progressBarScale or progressBarScaleValue
    if mainUIScale then mainUIScale.Scale = uiScaleValue / 100 end
    if pbScale then pbScale.Scale = progressBarScaleValue / 100 end
    if data.raptureSkinIndex and _G._RaptureSkinState then
        local total = #(_G._RaptureSkinList or {})
        _G._RaptureSkinState.currentIndex = _clamp(tonumber(data.raptureSkinIndex) or 5, 1, total)
        if _G._RaptureSkinLabelRef then
            local skin = _G._RaptureSkinList and _G._RaptureSkinList[_G._RaptureSkinState.currentIndex]
            if skin then _G._RaptureSkinLabelRef.Text = skin.label end
        end
        local curSkin = _G._RaptureSkinList and _G._RaptureSkinList[_G._RaptureSkinState.currentIndex]
        if curSkin and curSkin.label ~= "OFF" then
            task.defer(function()
                if _G._RaptureApplySkin then pcall(_G._RaptureApplySkin, _G._RaptureSkinState.currentIndex) end
            end)
        end
    end
    if data.raptureBatSkinIndex and _G._RaptureBatSkinState then
        _G._RaptureBatSkinState.currentIndex = _clamp(tonumber(data.raptureBatSkinIndex) or 1, 1, #(_G._RaptureBatSkinList or {}))
        if _G._RaptureBatSkinLabelRef then
            local bs = _G._RaptureBatSkinList and _G._RaptureBatSkinList[_G._RaptureBatSkinState.currentIndex]
            if bs then _G._RaptureBatSkinLabelRef.Text = bs.label end
        end
        if _G._RaptureBatSkinState.currentIndex > 1 then
            task.defer(function()
                if _G._RaptureApplyBatSkin then pcall(_G._RaptureApplyBatSkin, _G._RaptureBatSkinState.currentIndex) end
            end)
        end
    end
    local tpBatStateLoaded = data.tpBatEnabled or false
    if tpBatStateLoaded then
        task.defer(function()
            startBatDesyncTp()
            if batDesyncTpSetVisual then batDesyncTpSetVisual(true) end        end)
    else
        if batDesyncTpSetVisual then batDesyncTpSetVisual(false) end
    end
    local function lk(e, d)
        if not d then return end
        if d.kb and Enum.KeyCode[d.kb] then e.kb = Enum.KeyCode[d.kb] end
        if d.gp and Enum.KeyCode[d.gp] then e.gp = Enum.KeyCode[d.gp] end
    end
    lk(KB.DropBrainrot, data.dropBrainrotKey)
    lk(KB.AutoLeft, data.autoLeftKey)
    lk(KB.AutoRight, data.autoRightKey)
    lk(KB.AutoBat, data.autoBatKey)
    lk(KB.TPFloor, data.tpFloorKey)
    lk(KB.CarryToggle, data.carryToggleKey)
    lk(KB.LaggerMode, data.laggerModeKey)
    lk(KB.TPBat, data.tpBatKey)
    lk(KB.InstaReset, data.instaResetKey)
    lk(KB.BatBypass, data.batBypassKey)
    lk(KB.Spin, data.spinKey)
    lk(KB.Platform, data.platformKey)
    if data.mobileButtonPositions then savedButtonPositions = data.mobileButtonPositions end
    if data.mobilePanelPos then savedMobilePanelPos = data.mobilePanelPos end
    if data.tpBatFloatingPos then tpBatFloatingPos = data.tpBatFloatingPos end
    if data.batBypassFloatingPos then batBypassFloatingPos = data.batBypassFloatingPos end
    if data.instaResetFloatingPos then instaResetFloatingPos = data.instaResetFloatingPos end
    if data.platformFloatingPos then platformFloatingPos = data.platformFloatingPos end
    if data.progressBarPos then savedProgressBarPos = data.progressBarPos end
    stretchEnabled = data.stretchEnabled or false
    stretchValue = data.stretchValue or 0.7
    fovValue = data.fovValue or 70
    fovEnabled = data.fovEnabled or false
    if fovSliderSet then fovSliderSet(fovValue) end
    if fovEnabled then enableCustomFov() end
    if setFovVisual then setFovVisual(fovEnabled) end
    stretchFOV = data.stretchFOV or 120
    BAT_AIMBOT_SPEED = data.batAimbotSpeed or BAT_AIMBOT_SPEED
    backgroundIndex = data.backgroundIndex or 1
    if backgroundIndex < 1 or backgroundIndex > #backgroundImages then backgroundIndex = 1 end
    backgroundImageTransparency = data.backgroundImageTransparency or 0
    floatingButtonScale = data.floatingButtonScale or floatingButtonScale
    _G._RaptureHideMobileButtons = data.hideMobileButtons == true
    _G._RaptureMobileHideList    = type(data.mobileHideList) == "table" and data.mobileHideList or {}
    autoBatEnabled = false
    autoLeftEnabled = false
    autoRightEnabled = false
    if skyTheme ~= "Off" then pcall(applyCustomSky, skyTheme) end
    if skySelectorLabel then skySelectorLabel.Text = skyTheme end
    refreshSpeedModeLabel()
    if _G._RaptureSpeedDisplayRefresh then
        pcall(_G._RaptureSpeedDisplayRefresh)
    end
    if tpDownRefresh then pcall(tpDownRefresh) end
    if speedLabel then
        speedLabel.Visible = showSpeedLabel
        if showSpeedLabel then
            updateSpeedDisplayText()
        else
            speedLabel.Text = ""
        end
    end
    _lastSavedJSON = HS:JSONEncode(buildConfigTable())
    _isLoading = false
    return true
end

function resetToFactoryDefaults()
    _isResetting = true
    local ok, err = pcall(function()
        stopAllBackgroundTasks()
        NS = 60; CS = 29; LAGGER_SPEED = 15; LAGGER_CARRY_SPEED = 24.5
        CONFIG.STEAL_RANGE = 61
        stealMode = "V2"
        speedMode = false; laggerToggled = false; laggerCarryToggled = false
        antiRagdollEnabled = false; antiRagdollVersion = "v1"
        antiFlingEnabled = false; antiDieEnabled = false
        skipIntroEnabled = false
        medusaCounterEnabled = false; batCounterEnabled = false; ragdollCountdownEnabled = false
        autoBatEnabled = false; autoLeftEnabled = false; autoRightEnabled = false
        unwalkEnabled = false; uiLocked = true
        CONFIG.AUTO_STEAL_ENABLED = false
        BAT_AIMBOT_SPEED = 58
        aimbotMode = "V1"
        customSoundsEnabled = false
        soundButtonsEnabled = true
        soundTabEnabled     = true
        soundToggleEnabled  = true
        soundArrowEnabled   = true
        batBypassEnabled = false
        syncAfterHitEnabled = false
        bodyLockEnabled = false
        bodyLockRange = 20
        spinEnabled = false
        spinSpeed = 25
        spinAutoBrainrotEnabled = false
        xrayBaseEnabled = false
        xrayBrainrotEnabled = false
        speedDisplayMode = "Speed and Mode"
        showSpeedLabel = true
        tpDownMode = "Basic"
        platformEnabled = false
        _Platform.AUTO_DISABLE_TIME = 1.9
        if platformSecondsBox then platformSecondsBox.Text = "1.9" end
        skyTheme = "Off"
        pcall(applyCustomSky, "Off")
        if skySelectorLabel then skySelectorLabel.Text = "Off" end
        if _G._RaptureSpinStopSystem then _G._RaptureSpinStopSystem() end
        if _G._RaptureSpinAutoBrainrotSet then _G._RaptureSpinAutoBrainrotSet(false) end        if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
        if _G.RapturePlatformDisable then pcall(_G.RapturePlatformDisable, false) end
        if _G.RaptureCustomSounds then pcall(function() _G.RaptureCustomSounds.setEnabled(false) end) end
        if _syncAfterHit.conn then pcall(function() _syncAfterHit.conn:Disconnect() end); _syncAfterHit.conn = nil end
        if _G._XrayBaseSet then _G._XrayBaseSet(false) end
        if _G._XrayBrainrotSet then _G._XrayBrainrotSet(false) end
        if _G._RaptureClearSkin then pcall(_G._RaptureClearSkin) end
        if _G._RaptureSkinState then _G._RaptureSkinState.currentIndex = 5 end
        if _G._RaptureSkinLabelRef then _G._RaptureSkinLabelRef.Text = "OFF" end
        if _G._RaptureClearBatSkin then pcall(_G._RaptureClearBatSkin) end
        if _G._RaptureBatSkinState then _G._RaptureBatSkinState.currentIndex = 1 end
        if _G._RaptureBatSkinLabelRef then _G._RaptureBatSkinLabelRef.Text = "OFF" end
        stretchEnabled = false; stretchValue = 0.7; fovValue = 70
        uiScaleValue = 66; progressBarScaleValue = 55
        backgroundIndex = 1; floatingButtonScale = 1
        espEnabled = false; antiLagEnabled = false
        _G._RaptureHideMobileButtons = false        _G._RaptureMobileHideList = {}
        for key, val in pairs(DEFAULT_KB) do
            if KB[key] then KB[key].kb = val.kb; KB[key].gp = val.gp end
        end
        if isfile and isfile(CONFIG_FILE) then pcall(delfile, CONFIG_FILE) end
        savedButtonPositions = {}
        if MobilePanel and MobilePanel:FindFirstChild("FloatingPanel") then
            local container = MobilePanel:FindFirstChild("FloatingPanel")
            if container:FindFirstChild("ButtonsContainer") then
                for _, btn in ipairs(container.ButtonsContainer:GetChildren()) do
                    if btn:IsA("TextButton") and btn.Name then
                        local defX, defY = getDefaultButtonPosition(btn.Name)
                        btn.Position = UDim2.new(0, defX, 0, defY)
                    end
                end
            end
        end
        task.defer(function()
            if setAntiRagVisual then setAntiRagVisual(false) end
            if setInstaGrab then setInstaGrab(false) end
            if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
            if setBatCounterVisual then setBatCounterVisual(false) end
            if setMedusaVisual then setMedusaVisual(false) end
            if setRagdollTimerVisual then setRagdollTimerVisual(false) end
            if autoLeftSetVisual then autoLeftSetVisual(false) end
            if autoRightSetVisual then autoRightSetVisual(false) end
            if autoBatSetVisual then autoBatSetVisual(false) end
            if batDesyncTpSetVisual then batDesyncTpSetVisual(false) end
            if batBypassSetVisual then batBypassSetVisual(false) end
            if platformSetVisual then platformSetVisual(false) end
            if setESPVisual then setESPVisual(false) end
            if setAntiLagVisual then setAntiLagVisual(false) end
            if setFovVisual then setFovVisual(false) end
            if setAntiDieVisual then setAntiDieVisual(false) end
            if setAntiFlingVisual then setAntiFlingVisual(false) end
            if bodyLockSetVisual then bodyLockSetVisual(false) end
            if spinSetVisual then spinSetVisual(false) end
            if spinAutoBrainrotSetVisual then spinAutoBrainrotSetVisual(false) end
            if setUnwalkVisual then setUnwalkVisual(false) end
            if setXrayBaseVisual then setXrayBaseVisual(false) end
            if setXrayBrainrotVisual then setXrayBrainrotVisual(false) end
            if _G.stretchToggleSetter then _G.stretchToggleSetter(false) end
            if aimbotModeSetVisual then aimbotModeSetVisual("V1") end
            if antiRagSetVisual then antiRagSetVisual("V1") end
            if setCustomSoundsVisual then setCustomSoundsVisual(false) end
            if setSoundButtonsVisual then setSoundButtonsVisual(true) end
            if setSoundTabVisual then setSoundTabVisual(true) end
            if setSoundToggleVisual then setSoundToggleVisual(true) end
            if setSoundArrowVisual then setSoundArrowVisual(true) end
            if setSyncAfterHitVisual then setSyncAfterHitVisual(false) end
            if speedDisplayRefresh then speedDisplayRefresh() end
            if setShowSpeedVisual then setShowSpeedVisual(true) end
            if setTpDownVisual then setTpDownVisual("Basic") end
            if infiniteJumpSetVisual then infiniteJumpSetVisual(true) end
            if skipIntroToggleSet then skipIntroToggleSet(false) end
            if _G._RaptureClearSkin then pcall(_G._RaptureClearSkin) end
            if _G._RaptureSkinLabelRef then _G._RaptureSkinLabelRef.Text = "OFF" end
            if _G._RaptureClearBatSkin then pcall(_G._RaptureClearBatSkin) end
            if _G._RaptureBatSkinLabelRef then _G._RaptureBatSkinLabelRef.Text = "OFF" end
            if _G._RaptureApplyMobileButtonsHidden then _G._RaptureApplyMobileButtonsHidden() end
            if _refreshHideListVisuals then _refreshHideListVisuals() end
            refreshSpeedModeLabel()
        end)
        updateAllUIThemeColors(PURPLE_LIGHT)
        _RaptureApplyThemeToSliders()
        _lastSavedJSON = nil
        saveAllSettings()
    end)
    _isResetting = false
    return ok
end

function updateProgressBarVisibility()
    if pbFrame then pbFrame.Visible = true end
end

function getDefaultButtonPosition(btnName)
    local BTN_W, BTN_H = 78, 50
    local GAP = 8
    local orderMap = {
        DropBR = 0, AutoLeft = 1, AutoBat = 2, AutoRight = 3,
        TpDown = 4, Carry = 5, Lagger1 = 6, Lagger2 = 7
    }
    local order = orderMap[btnName] or 0
    local row = _floor(order / 2)
    local col = order % 2
    return -(col + 1) * (BTN_W + GAP) - 10, row * (BTN_H + GAP + 10)
end

local _enemySpeedAcc = 0
function updateEnemySpeedLabels()
    _enemySpeedAcc = _enemySpeedAcc + 1
    if _enemySpeedAcc < 6 then return end
    _enemySpeedAcc = 0
    local color = getThemeColor()
    local players = _GetPlayersCached()
    for i = 1, #players do
        local player = players[i]
        if player ~= LP then
            local char = player.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local v = hrp.AssemblyLinearVelocity
                local speed = _sqrt(v.X*v.X + v.Z*v.Z)
                local label = enemySpeedLabels[player]
                if not label then
                    local head = char:FindFirstChild("Head")
                    if head then
                        local bb = Instance.new("BillboardGui")
                        bb.Size = UDim2.new(0, 100, 0, 25)
                        bb.StudsOffset = _V3new(0, 5.5, 0)
                        bb.AlwaysOnTop = true
                        bb.Name = "EnemySpeedGui"
                        bb.Parent = head
                        local tl = Instance.new("TextLabel", bb)
                        tl.Size = UDim2.new(1, 0, 1, 0)
                        tl.BackgroundTransparency = 1
                        tl.TextColor3 = color
                        tl.Font = Enum.Font.GothamBold
                        tl.TextScaled = true
                        tl.TextStrokeTransparency = 0
                        tl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                        enemySpeedLabels[player] = tl
                        label = tl
                    end
                elseif label.Parent and label.Parent.Parent ~= char then
                    local head = char:FindFirstChild("Head")
                    if head then label.Parent.Parent = head end
                end
                if label then
                    label.Text = string.format("%.1f", speed)
                    if label.TextColor3 ~= color then label.TextColor3 = color end
                end
            else
                local label = enemySpeedLabels[player]
                if label and label.Parent and label.Parent.Parent then label.Parent.Parent = nil end
                enemySpeedLabels[player] = nil
            end
        end
    end
end

function startEnemySpeed()
    if enemySpeedConn then enemySpeedConn:Disconnect() end
    enemySpeedConn = RunService.Heartbeat:Connect(updateEnemySpeedLabels)
end

function stopEnemySpeed()
    if enemySpeedConn then enemySpeedConn:Disconnect(); enemySpeedConn = nil end
end

do
    if not _G.InstaResetLoaded then
        _G.InstaResetLoaded = true
        local resetCooldown = false
        local resetThread = nil
        local _lastInstaResetRequest = 0
        local function instaResetFast()
            if resetCooldown then return end
            resetCooldown = true
            local character = LP.Character
            if not character then resetCooldown = false return end
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if not humanoid then resetCooldown = false return end
            resetThread = task.spawn(function()
                local attempts = 0
                while character and character.Parent and humanoid and humanoid.Health > 0 and attempts < 40 do
                    if LP.Character ~= character then break end
                    pcall(function()
                        humanoid.HipHeight = 1e30
                        humanoid.AutoRotate = true
                        local rootPart = character:FindFirstChild("HumanoidRootPart")
                        if rootPart then rootPart.CanCollide = false end
                    end)
                    if not character or not character.Parent or humanoid.Health <= 0 then break end
                    attempts = attempts + 1
                    task.wait(0.05)
                end
                if character and character.Parent and humanoid and humanoid.Health > 0 then
                    pcall(function() humanoid.Health = 0 end)
                end
                resetCooldown = false
                resetThread = nil
            end)
        end
        local function instaReset()
            local now = os.clock()
            if now - _lastInstaResetRequest < 0.75 then return end
            _lastInstaResetRequest = now
            instaResetFast()
        end
        LP.CharacterAdded:Connect(function()
            if resetThread then pcall(task.cancel, resetThread); resetThread = nil end
            resetCooldown = false
        end)
        _G.InstaReset = { Trigger = instaReset }
    end
end

local _xrayBase = {
    enabled = false,
    loopId = 0,
    conns = {},
    origTrans = setmetatable({}, { __mode = "k" }),
    ALPHA = 0.5,
    FOLDERS = {
        "Base", "PlotSign", "FriendPanel", "Cash", "Laser",
        "Decorations", "Skin", "Unlock", "Purchases"
    },
}

local function _xrayBaseApplyOne(obj, alpha)
    if not obj or not obj.Parent then return end
    pcall(function()
        if obj:IsA("BasePart") then
            if _xrayBase.origTrans[obj] == nil then _xrayBase.origTrans[obj] = obj.Transparency end
            local o = _xrayBase.origTrans[obj]
            if o < 1 then
                local target = o + (1 - o) * alpha
                if math.abs(obj.Transparency - target) > 0.01 then
                    obj.Transparency = target
                end
            end
        elseif obj:IsA("TextLabel") or obj:IsA("TextButton") then
            if _xrayBase.origTrans[obj] == nil then
                _xrayBase.origTrans[obj] = { text = obj.TextTransparency, bg = obj.BackgroundTransparency }
            end
            local o = _xrayBase.origTrans[obj]
            if o.text < 1 then
                local t = o.text + (1 - o.text) * alpha
                if math.abs(obj.TextTransparency - t) > 0.01 then obj.TextTransparency = t end
            end
            if o.bg < 1 then
                local t = o.bg + (1 - o.bg) * alpha
                if math.abs(obj.BackgroundTransparency - t) > 0.01 then obj.BackgroundTransparency = t end
            end
        elseif obj:IsA("Frame") or obj:IsA("ScrollingFrame") then
            if _xrayBase.origTrans[obj] == nil then _xrayBase.origTrans[obj] = obj.BackgroundTransparency end
            local o = _xrayBase.origTrans[obj]
            if o < 1 then
                local t = o + (1 - o) * alpha
                if math.abs(obj.BackgroundTransparency - t) > 0.01 then obj.BackgroundTransparency = t end
            end
        elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
            if _xrayBase.origTrans[obj] == nil then
                _xrayBase.origTrans[obj] = { img = obj.ImageTransparency, bg = obj.BackgroundTransparency }
            end
            local o = _xrayBase.origTrans[obj]
            if o.img < 1 then
                local t = o.img + (1 - o.img) * alpha
                if math.abs(obj.ImageTransparency - t) > 0.01 then obj.ImageTransparency = t end
            end
            if o.bg < 1 then
                local t = o.bg + (1 - o.bg) * alpha
                if math.abs(obj.BackgroundTransparency - t) > 0.01 then obj.BackgroundTransparency = t end
            end
        end
    end)
end

local function _xrayBaseApplySubtree(root, alpha, myLoop)
    if not root or not root.Parent then return end
    if myLoop ~= _xrayBase.loopId then return end
    _xrayBaseApplyOne(root, alpha)
    local n = 0
    for _, d in ipairs(root:GetDescendants()) do
        if myLoop ~= _xrayBase.loopId then return end
        _xrayBaseApplyOne(d, alpha)
        n = n + 1
        if n % 300 == 0 then task.wait() end
    end
    if myLoop ~= _xrayBase.loopId then return end
    table.insert(_xrayBase.conns, root.DescendantAdded:Connect(function(d)
        if myLoop ~= _xrayBase.loopId or not _xrayBase.enabled then return end
        task.defer(function() _xrayBaseApplyOne(d, alpha) end)
    end))
end

local function _xrayBaseApplyPlot(plot, alpha, myLoop)
    if not plot or myLoop ~= _xrayBase.loopId then return end
    for _, fname in ipairs(_xrayBase.FOLDERS) do
        if myLoop ~= _xrayBase.loopId then return end
        _xrayBaseApplySubtree(plot:FindFirstChild(fname), alpha, myLoop)
    end
    if myLoop ~= _xrayBase.loopId then return end
    table.insert(_xrayBase.conns, plot.ChildAdded:Connect(function(child)
        if myLoop ~= _xrayBase.loopId or not _xrayBase.enabled then return end
        for _, fname in ipairs(_xrayBase.FOLDERS) do
            if child.Name == fname then
                task.defer(function() _xrayBaseApplySubtree(child, alpha, myLoop) end)
                break
            end
        end
    end))
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if podiums then
        local function processPodium(p)
            if not p then return end
            for _, c in ipairs(p:GetChildren()) do
                if c.Name == "Claim" then
                    _xrayBaseApplySubtree(c, alpha, myLoop)
                elseif c.Name == "Base" then
                    _xrayBaseApplySubtree(c:FindFirstChild("Decorations"), alpha, myLoop)
                elseif c:IsA("Model") and c.Name ~= "Decorations" then
                    _xrayBaseApplySubtree(c, alpha, myLoop)
                end
            end
        end
        for _, p in ipairs(podiums:GetChildren()) do
            if myLoop ~= _xrayBase.loopId then return end
            processPodium(p)
        end
        table.insert(_xrayBase.conns, podiums.ChildAdded:Connect(function(p)
            if myLoop ~= _xrayBase.loopId or not _xrayBase.enabled then return end
            task.wait(0.1)
            if myLoop ~= _xrayBase.loopId then return end
            processPodium(p)
        end))
    end
end

function _G._XrayBaseSet(on)
    if on then
        if _xrayBase.enabled then return end
        _xrayBase.enabled = true
        _xrayBase.conns = {}
        _xrayBase.loopId = _xrayBase.loopId + 1
        local myLoop = _xrayBase.loopId
        task.spawn(function()
            local deadline = os.clock() + 60
            while not workspace:FindFirstChild("Plots") and os.clock() < deadline do
                task.wait(0.5)
            end
            if myLoop ~= _xrayBase.loopId then return end
            local plots = workspace:FindFirstChild("Plots")
            if not plots then return end
            for _, plot in ipairs(plots:GetChildren()) do
                if myLoop ~= _xrayBase.loopId then return end
                _xrayBaseApplyPlot(plot, _xrayBase.ALPHA, myLoop)
                task.wait()
            end
            if myLoop ~= _xrayBase.loopId then return end
            table.insert(_xrayBase.conns, plots.ChildAdded:Connect(function(plot)
                if myLoop ~= _xrayBase.loopId or not _xrayBase.enabled then return end
                task.wait(0.2)
                if myLoop ~= _xrayBase.loopId then return end
                _xrayBaseApplyPlot(plot, _xrayBase.ALPHA, myLoop)
            end))
        end)
    else
        if not _xrayBase.enabled then return end
        _xrayBase.enabled = false
        _xrayBase.loopId = _xrayBase.loopId + 1
        for _, c in ipairs(_xrayBase.conns) do
            pcall(function() c:Disconnect() end)
        end
        _xrayBase.conns = {}
        local snapshot = _xrayBase.origTrans
        _xrayBase.origTrans = setmetatable({}, { __mode = "k" })
        for obj, o in pairs(snapshot) do
            pcall(function()
                if not obj or not obj.Parent then return end
                if obj:IsA("BasePart") then
                    obj.Transparency = o
                elseif obj:IsA("TextLabel") or obj:IsA("TextButton") then
                    obj.TextTransparency = o.text
                    obj.BackgroundTransparency = o.bg
                elseif obj:IsA("Frame") or obj:IsA("ScrollingFrame") then
                    obj.BackgroundTransparency = o
                elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                    obj.ImageTransparency = o.img
                    obj.BackgroundTransparency = o.bg
                end
            end)
        end
    end
end

local _xrayBrainrot = {
    enabled = false,
    TRANSPARENCY = 0.85,
    HITBOX_MIN = 0.5,
    saved = setmetatable({}, { __mode = "k" }),
    names = nil,
    conns = {},
    renderConn = nil,
}

local function _xrayBrainrotIsHitbox(o)
    local n = string.lower(o.Name)
    return string.find(n, "hit") ~= nil
        or string.find(n, "target") ~= nil
        or string.find(n, "box") ~= nil
        or string.find(n, "root") ~= nil
        or string.find(n, "collect") ~= nil
        or string.find(n, "prompt") ~= nil
end

local function _xrayBrainrotApplyOne(o)
    if not o or not o.Parent then return end
    if not (o:IsA("BasePart") or o:IsA("Decal") or o:IsA("Texture")) then return end
    local entry = _xrayBrainrot.saved[o]
    if entry == nil then
        local orig = o.Transparency
        if orig > _xrayBrainrot.HITBOX_MIN and orig < 0.99
           and o:IsA("BasePart") and _xrayBrainrotIsHitbox(o) then
            orig = 1
        end
        _xrayBrainrot.saved[o] = { orig = orig }
        entry = _xrayBrainrot.saved[o]
    end
    if entry.orig < _xrayBrainrot.HITBOX_MIN and o.Transparency ~= _xrayBrainrot.TRANSPARENCY then
        o.Transparency = _xrayBrainrot.TRANSPARENCY
    end
end

local function _xrayBrainrotApplyModel(m)
    if not m or not m.Parent then return end
    for _, o in ipairs(m:GetDescendants()) do
        _xrayBrainrotApplyOne(o)
    end
end

local function _xrayBrainrotIsAnimal(m)
    return m:IsA("Model") and _xrayBrainrot.names and _xrayBrainrot.names[m.Name] == true
end

local function _xrayBrainrotAnimalAncestor(inst)
    local p = inst.Parent
    for _ = 1, 6 do
        if not p or p == workspace then return nil end
        if p:IsA("Model") and _xrayBrainrot.names and _xrayBrainrot.names[p.Name] then return p end
        p = p.Parent
    end
    return nil
end

local function _xrayBrainrotHandleInstance(inst)
    if not inst or not inst.Parent then return end
    if inst:IsA("Model") then
        if _xrayBrainrot.names and _xrayBrainrot.names[inst.Name] then
            _xrayBrainrotApplyModel(inst)
        end
        return
    end
    if inst:IsA("BasePart") or inst:IsA("Decal") or inst:IsA("Texture") then
        if _xrayBrainrotAnimalAncestor(inst) then _xrayBrainrotApplyOne(inst) end
    end
end

local function _xrayBrainrotLoadNames()
    if _xrayBrainrot.names then return _xrayBrainrot.names end
    local t = {}
    pcall(function()
        local datas = ReplicatedStorage:FindFirstChild("Datas")
        local m = datas and datas:FindFirstChild("Animals")
        if m and m:IsA("ModuleScript") then
            for k in pairs(require(m)) do t[k] = true end
        end
    end)
    _xrayBrainrot.names = t
    return t
end

function _G._XrayBrainrotSet(on)
    if on then
        if _xrayBrainrot.enabled then return end
        _xrayBrainrot.enabled = true
        _xrayBrainrot.conns = {}
        _xrayBrainrot.saved = setmetatable({}, { __mode = "k" })
        _xrayBrainrotLoadNames()
        table.insert(_xrayBrainrot.conns, workspace.DescendantAdded:Connect(_xrayBrainrotHandleInstance))
        table.insert(_xrayBrainrot.conns, game.DescendantAdded:Connect(function(inst)
            if inst:IsA("Model") or inst:IsA("BasePart") or inst:IsA("Decal") or inst:IsA("Texture") then
                _xrayBrainrotHandleInstance(inst)
            end
        end))
        table.insert(_xrayBrainrot.conns, workspace.ChildAdded:Connect(_xrayBrainrotHandleInstance))
        local function hookChar(char)
            if not char then return end
            table.insert(_xrayBrainrot.conns, char.ChildAdded:Connect(_xrayBrainrotHandleInstance))
            for _, child in ipairs(char:GetChildren()) do _xrayBrainrotHandleInstance(child) end
        end
        if LP.Character then hookChar(LP.Character) end
        table.insert(_xrayBrainrot.conns, LP.CharacterAdded:Connect(hookChar))
        table.insert(_xrayBrainrot.conns, game.DescendantAdded:Connect(function(inst)
            if inst:IsA("BasePart") or inst:IsA("Decal") or inst:IsA("Texture") then
                local anc = _xrayBrainrotAnimalAncestor(inst)
                if anc then _xrayBrainrotApplyOne(inst) end
            end
        end))
        _xrayBrainrot.renderConn = RunService.RenderStepped:Connect(function()
            for o, entry in pairs(_xrayBrainrot.saved) do
                if o and o.Parent and entry.orig < _xrayBrainrot.HITBOX_MIN then
                    if o.Transparency ~= _xrayBrainrot.TRANSPARENCY then
                        o.Transparency = _xrayBrainrot.TRANSPARENCY
                    end
                end
            end
        end)
        task.spawn(function()
            for _, d in ipairs(workspace:GetDescendants()) do
                if _xrayBrainrotIsAnimal(d) then _xrayBrainrotApplyModel(d) end
            end
        end)
    else
        if not _xrayBrainrot.enabled then return end
        _xrayBrainrot.enabled = false
        for _, c in ipairs(_xrayBrainrot.conns) do
            pcall(function() c:Disconnect() end)
        end
        _xrayBrainrot.conns = {}
        if _xrayBrainrot.renderConn then
            pcall(function() _xrayBrainrot.renderConn:Disconnect() end)
            _xrayBrainrot.renderConn = nil
        end
        local snapshot = _xrayBrainrot.saved
        _xrayBrainrot.saved = setmetatable({}, { __mode = "k" })
        for o, entry in pairs(snapshot) do
            pcall(function()
                if o and o.Parent then
                    o.Transparency = entry.orig
                end
            end)
        end
    end
end

minecraftBatSkinEnabled = minecraftBatSkinEnabled or false
minecraftBatSkinColorMode = minecraftBatSkinColorMode or "Default"

local MinecraftBatSkin = (function()
    local MB_ASSET_ID = "rbxassetid://18566246244"
    local MB_POS_OFFSET = _CFnew(-0.02, -0.52, -0.2)
    local MB_ROT_OFFSET = CFrame.Angles(math.rad(45), math.rad(0), math.rad(5))
    local MB_SCALE = 2
    local MB_COLORS = {
        ["Default"] = nil,
        ["Abyss Blue"] = Color3.fromRGB(0, 40, 150),
        ["Venom Green"] = Color3.fromRGB(20, 255, 50),
        ["Royal Gold"] = Color3.fromRGB(255, 200, 0),
        ["Velvet Rose"] = Color3.fromRGB(220, 20, 100),
        ["Crimson Night"] = Color3.fromRGB(90, 0, 20),
        ["RGB"] = "RGB",
    }
    local State = { enabled = false, colorMode = "Default", cachedVisualPart = nil, rgbConnection = nil }
    local okLoad, objects = pcall(function() return game:GetObjects(MB_ASSET_ID) end)
    if okLoad and objects and #objects > 0 then
        local model = objects[1]
        State.cachedVisualPart = model:IsA("BasePart") and model or model:FindFirstChildWhichIsA("BasePart", true)
    end
    local function mbSetColor(visual, color)
        if not visual or not color then return end
        visual.Color = color
        if visual:IsA("UnionOperation") then
            pcall(function() visual.UsePartColor = true end)
        end
        local sm = visual:FindFirstChildWhichIsA("SpecialMesh")
        if sm then sm.VertexColor = Vector3.new(color.R, color.G, color.B) end
    end
    local function mbFindBat()
        local char = LP.Character
        if char then
            local t = char:FindFirstChild("Bat")
            if t and t:IsA("Tool") then return t end
        end
        local bp = LP:FindFirstChildOfClass("Backpack")
        if bp then
            local t = bp:FindFirstChild("Bat")
            if t and t:IsA("Tool") then return t end
        end
        return nil
    end
    local function mbRemoveVisual(tool)
        if not tool then return end
        local old = tool:FindFirstChild("CustomUnionVisual")
        if old then pcall(function() old:Destroy() end) end
        local handle = tool:FindFirstChild("Handle")
        if handle then
            handle.Transparency = 0
            for _, child in ipairs(handle:GetChildren()) do
                if child:IsA("SpecialMesh") or child:IsA("Mesh") or child:IsA("Decal") then
                    pcall(function() child.Transparency = 0 end)
                end
            end
        end
    end
    local function mbApplyVisual(tool)
        if not tool or tool.Name ~= "Bat" or not State.cachedVisualPart then return end
        local handle = tool:WaitForChild("Handle", 2)
        if not handle then return end
        local oldVisual = tool:FindFirstChild("CustomUnionVisual")
        if oldVisual then oldVisual:Destroy() end
        handle.Transparency = 1
        for _, child in ipairs(handle:GetChildren()) do
            if child:IsA("SpecialMesh") or child:IsA("Mesh") or child:IsA("Decal") then
                pcall(function() child.Transparency = 1 end)
            end
        end
        local newVisual = State.cachedVisualPart:Clone()
        newVisual.Name = "CustomUnionVisual"
        newVisual.CanCollide = false; newVisual.Massless = true; newVisual.Anchored = false
        local tempModel = Instance.new("Model")
        newVisual.Parent = tempModel
        tempModel.PrimaryPart = newVisual
        pcall(function() tempModel:ScaleTo(MB_SCALE) end)
        newVisual.Parent = nil
        tempModel:Destroy()
        if State.colorMode == "RGB" then
            mbSetColor(newVisual, Color3.fromHSV(tick() % 3 / 3, 1, 1))
        elseif MB_COLORS[State.colorMode] then
            mbSetColor(newVisual, MB_COLORS[State.colorMode])
        end
        for _, child in ipairs(newVisual:GetChildren()) do
            if child:IsA("Weld") or child:IsA("WeldConstraint") then child:Destroy() end
        end
        newVisual.CFrame = handle.CFrame * MB_POS_OFFSET * MB_ROT_OFFSET
        newVisual.Parent = tool
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = handle; weld.Part1 = newVisual; weld.Parent = newVisual
    end
    local function mbStartRGB()
        if State.rgbConnection then return end
        State.rgbConnection = RunService.RenderStepped:Connect(function()
            if not State.enabled or State.colorMode ~= "RGB" then return end
            local tool = mbFindBat()
            if not tool then return end
            local visual = tool:FindFirstChild("CustomUnionVisual")
            if visual then mbSetColor(visual, Color3.fromHSV(tick() % 3 / 3, 1, 1)) end
        end)
    end
    local function mbStopRGB()
        if State.rgbConnection then
            State.rgbConnection:Disconnect(); State.rgbConnection = nil
        end
    end
    return {
        State = State, Colors = MB_COLORS,
        FindBat = mbFindBat,
        Apply = function()
            if not State.enabled then return end
            local tool = mbFindBat()
            if tool then mbApplyVisual(tool) end
            if State.colorMode == "RGB" then mbStartRGB() end
        end,
        Remove = function()
            mbStopRGB()
            local char = LP.Character
            if char then
                local t = char:FindFirstChild("Bat")
                if t then mbRemoveVisual(t) end
            end
            local bp = LP:FindFirstChildOfClass("Backpack")
            if bp then
                local t = bp:FindFirstChild("Bat")
                if t then mbRemoveVisual(t) end
            end
        end,
        SetColorMode = function(mode)
            State.colorMode = mode
            if not State.enabled then return end
            if mode == "RGB" then mbStartRGB() else mbStopRGB() end
            local tool = mbFindBat()
            if tool then mbApplyVisual(tool) end
        end,
    }
end)()

_G.MinecraftBatSkin = MinecraftBatSkin

do
    local BAT_SKIN_LIST = {
        { label = "OFF" },
        { label = "Minecraft" },
        { label = "Ronin Katana", assetId = 12187348, posOffset = CFrame.new(0, 0.6, 0),
          rotOffset = CFrame.Angles(math.rad(-90), 0, 0), scale = 1.4 },
        { label = "Pool Noodle", assetId = 164207842, posOffset = CFrame.new(0, 0.8, 0),
          rotOffset = CFrame.Angles(math.rad(-90), 0, 0), scale = 1.6 },
        { label = "Banhammer", assetId = 114452163024876, posOffset = CFrame.new(0, 0.3, 0),
          rotOffset = CFrame.Angles(math.rad(90), 0, 0), scale = 1 },
        { label = "Bat Scythe", assetId = 306971294, posOffset = CFrame.new(0, 0.4, 0),
          autoRotate = true, spinY = 0, scale = 1.3 },
        { label = "Stone Hammer", assetId = 13207169, posOffset = CFrame.new(0, 0.4, 0),
          rotOffset = CFrame.Angles(math.rad(-90), 0, 0), scale = 1.2 },
        { label = "Frying Pan", assetId = 24346755, posOffset = CFrame.new(0, 0.4, 0),
          rotOffset = CFrame.Angles(math.rad(-90), 0, 0), scale = 1.2 },
    }
    _G._RaptureBatSkinList = BAT_SKIN_LIST
    _G._RaptureBatSkinState = _G._RaptureBatSkinState or {
        currentIndex = 1,
        conn = nil,
        cache = {},
    }
    local BS = _G._RaptureBatSkinState

    local function bsLoadAsset(assetId)
        if BS.cache[assetId] then return BS.cache[assetId] end
        local parts = {}
        local ok, objs = pcall(function()
            return game:GetObjects("rbxassetid://" .. tostring(assetId))
        end)
        if ok and type(objs) == "table" and #objs > 0 then
            for _, obj in ipairs(objs) do
                obj.Archivable = true
                for _, d in ipairs(obj:GetDescendants()) do d.Archivable = true end
                if obj:IsA("BasePart") then
                    table.insert(parts, obj)
                else
                    for _, d in ipairs(obj:GetDescendants()) do
                        if d:IsA("BasePart") then table.insert(parts, d) end
                    end
                end
            end
        end
        if #parts == 0 then
            local ok2, model = pcall(function()
                return game:GetService("InsertService"):LoadAsset(assetId)
            end)
            if ok2 and model then
                model.Archivable = true
                for _, d in ipairs(model:GetDescendants()) do
                    d.Archivable = true
                    if d:IsA("BasePart") then table.insert(parts, d) end
                end
            end
        end
        if #parts > 0 then
            local cached = {}
            for _, part in ipairs(parts) do
                local okc, clone = pcall(function() return part:Clone() end)
                if okc and clone then table.insert(cached, clone) end
            end
            BS.cache[assetId] = cached
            return cached
        end
        return nil
    end

    local function bsComputeUpright(assetId)
        local parts = bsLoadAsset(assetId)
        if not parts or #parts == 0 then return CFrame.new() end
        local minV = Vector3.new(math.huge, math.huge, math.huge)
        local maxV = Vector3.new(-math.huge, -math.huge, -math.huge)
        for _, p in ipairs(parts) do
            local s = p.Size
            local cf = p.CFrame
            local corners = {
                Vector3.new(s.X/2, s.Y/2, s.Z/2), Vector3.new(-s.X/2, s.Y/2, s.Z/2),
                Vector3.new(s.X/2, -s.Y/2, s.Z/2), Vector3.new(-s.X/2, -s.Y/2, s.Z/2),
                Vector3.new(s.X/2, s.Y/2, -s.Z/2), Vector3.new(-s.X/2, s.Y/2, -s.Z/2),
                Vector3.new(s.X/2, -s.Y/2, -s.Z/2), Vector3.new(-s.X/2, -s.Y/2, -s.Z/2),
            }
            for _, offset in ipairs(corners) do
                local w = cf:PointToWorldSpace(offset)
                minV = Vector3.new(math.min(minV.X, w.X), math.min(minV.Y, w.Y), math.min(minV.Z, w.Z))
                maxV = Vector3.new(math.max(maxV.X, w.X), math.max(maxV.Y, w.Y), math.max(maxV.Z, w.Z))
            end
        end
        local size = maxV - minV
        if size.X >= size.Y and size.X >= size.Z then
            return CFrame.Angles(0, 0, math.rad(-90))
        elseif size.Z >= size.Y and size.Z >= size.X then
            return CFrame.Angles(math.rad(90), 0, 0)
        end
        return CFrame.new()
    end

    local function bsFindBat()
        local char = LP.Character
        if char then
            local t = char:FindFirstChild("Bat")
            if t and t:IsA("Tool") then return t end
        end
        local bp = LP:FindFirstChildOfClass("Backpack")
        if bp then
            local t = bp:FindFirstChild("Bat")
            if t and t:IsA("Tool") then return t end
        end
        return nil
    end

    local function bsClearVisual(tool)
        if not tool then return end
        local v = tool:FindFirstChild("CustomUnionVisual")
        if v then v:Destroy() end
        local h = tool:FindFirstChild("Handle")
        if h then
            h.Transparency = 0
            for _, c in ipairs(h:GetChildren()) do
                if c:IsA("SpecialMesh") or c:IsA("Mesh") or c:IsA("Decal") then
                    pcall(function() c.Transparency = 0 end)
                end
            end
        end
    end

    local function bsApplyAsset(tool, skin)
        if not tool or not skin or not skin.assetId then return end
        local handle = tool:WaitForChild("Handle", 2)
        if not handle then return end
        bsClearVisual(tool)
        handle.Transparency = 1
        for _, c in ipairs(handle:GetChildren()) do
            if c:IsA("SpecialMesh") or c:IsA("Mesh") or c:IsA("Decal") then
                pcall(function() c.Transparency = 1 end)
            end
        end
        local parts = bsLoadAsset(skin.assetId)
        if not parts or #parts == 0 then return end
        local finalRot
        if skin.rotOffset then
            finalRot = skin.rotOffset
        elseif skin.autoRotate then
            finalRot = bsComputeUpright(skin.assetId)
        else
            finalRot = CFrame.new()
        end
        if skin.spinY and skin.spinY ~= 0 then
            finalRot = finalRot * CFrame.Angles(0, math.rad(skin.spinY), 0)
        end
        local baseCF = handle.CFrame
        local container = Instance.new("Model")
        container.Name = "CustomUnionVisual"
        for _, srcPart in ipairs(parts) do
            local clone = srcPart:Clone()
            clone.Archivable = true
            clone.Anchored = false
            clone.CanCollide = false
            clone.CanTouch = false
            clone.CanQuery = false
            clone.Massless = true
            if skin.scale then
                pcall(function() clone.Size = clone.Size * skin.scale end)
            end
            clone.CFrame = baseCF * (skin.posOffset or CFrame.new()) * finalRot
            clone.Parent = container
        end
        container.PrimaryPart = container:GetChildren()[1]
        container.Parent = tool
        for _, part in ipairs(container:GetChildren()) do
            if part:IsA("BasePart") then
                local weld = Instance.new("WeldConstraint")
                weld.Name = "SkinWeld"
                weld.Part0 = handle
                weld.Part1 = part
                weld.Parent = part
            end
        end
    end

    function _G._RaptureApplyBatSkin(index)
        local skin = BAT_SKIN_LIST[index]
        if not skin then return end
        if BS.conn then BS.conn:Disconnect(); BS.conn = nil end
        local tool = bsFindBat()
        bsClearVisual(tool)
        if index == 1 then BS.currentIndex = 1 return end
        BS.currentIndex = index
        if skin.label == "Minecraft" then
            if MinecraftBatSkin and MinecraftBatSkin.State then
                minecraftBatSkinEnabled = true
                minecraftBatSkinColorMode = "RGB"
                MinecraftBatSkin.State.enabled = true
                MinecraftBatSkin.SetColorMode("RGB")
                MinecraftBatSkin.Apply()
            end
            return
        end
        if tool then
            pcall(bsApplyAsset, tool, skin)
        end
        BS.conn = RunService.Heartbeat:Connect(function()
            if BS.currentIndex ~= index then return end
            local t = bsFindBat()
            if t and not t:FindFirstChild("CustomUnionVisual") then
                pcall(bsApplyAsset, t, skin)
            end
        end)
    end

    function _G._RaptureClearBatSkin()
        if BS.conn then BS.conn:Disconnect(); BS.conn = nil end
        local tool = bsFindBat()
        bsClearVisual(tool)
        if MinecraftBatSkin and MinecraftBatSkin.State then
            minecraftBatSkinEnabled = false
            MinecraftBatSkin.State.enabled = false
            pcall(MinecraftBatSkin.Remove)
        end
        BS.currentIndex = 1
    end

    _G._RaptureBatSkinUIBuild = function(visualPage, mkRow, mkLabel)
        local bsRow = mkRow(visualPage, 42)
        mkLabel(bsRow, "Bat Skin")

        local bsContainer = Instance.new("Frame", bsRow)
        bsContainer.Size = UDim2.new(0, 160, 1, 0)
        bsContainer.Position = UDim2.new(1, -168, 0, 0)
        bsContainer.BackgroundTransparency = 1
        bsContainer.ZIndex = 8

        local bsLeftBtn = Instance.new("TextButton", bsContainer)
        bsLeftBtn.Size = UDim2.new(0, 28, 0, 26)
        bsLeftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        bsLeftBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        bsLeftBtn.BackgroundTransparency = 0.3
        bsLeftBtn.BorderSizePixel = 0
        bsLeftBtn.Text = "<"
        bsLeftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        bsLeftBtn.Font = Enum.Font.GothamBold
        bsLeftBtn.TextSize = 13
        bsLeftBtn.AutoButtonColor = false
        bsLeftBtn.ZIndex = 9
        Instance.new("UICorner", bsLeftBtn).CornerRadius = UDim.new(0, 6)
        local bls = Instance.new("UIStroke", bsLeftBtn)
        bls.Color = PURPLE_LIGHT; bls.Thickness = 1; bls.Transparency = 0.5

        local bsLabel = Instance.new("TextLabel", bsContainer)
        bsLabel.Size = UDim2.new(0, 80, 0, 26)
        bsLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        bsLabel.BackgroundTransparency = 1
        bsLabel.Text = BAT_SKIN_LIST[BS.currentIndex] and BAT_SKIN_LIST[BS.currentIndex].label or "OFF"
        bsLabel.TextColor3 = PURPLE_LIGHT
        bsLabel.Font = Enum.Font.GothamBold
        bsLabel.TextSize = 12
        bsLabel.TextXAlignment = Enum.TextXAlignment.Center
        bsLabel.ZIndex = 9

        local bsRightBtn = Instance.new("TextButton", bsContainer)
        bsRightBtn.Size = UDim2.new(0, 28, 0, 26)
        bsRightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        bsRightBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        bsRightBtn.BackgroundTransparency = 0.3
        bsRightBtn.BorderSizePixel = 0
        bsRightBtn.Text = ">"
        bsRightBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        bsRightBtn.Font = Enum.Font.GothamBold
        bsRightBtn.TextSize = 13
        bsRightBtn.AutoButtonColor = false
        bsRightBtn.ZIndex = 9
        Instance.new("UICorner", bsRightBtn).CornerRadius = UDim.new(0, 6)
        local brs = Instance.new("UIStroke", bsRightBtn)
        brs.Color = PURPLE_LIGHT; brs.Thickness = 1; brs.Transparency = 0.5

        local function bsCycle(direction)
            local total = #BAT_SKIN_LIST
            BS.currentIndex = BS.currentIndex + direction
            if BS.currentIndex < 1 then BS.currentIndex = total end
            if BS.currentIndex > total then BS.currentIndex = 1 end
            bsLabel.Text = BAT_SKIN_LIST[BS.currentIndex].label
            if _G._RaptureApplyBatSkin then
                pcall(_G._RaptureApplyBatSkin, BS.currentIndex)
            end
            task.defer(saveAllSettings)
        end

        bsLeftBtn.MouseButton1Click:Connect(function() playArrowSound(); bsCycle(-1) end)
        bsRightBtn.MouseButton1Click:Connect(function() playArrowSound(); bsCycle(1) end)

        _G._RaptureBatSkinLabelRef = bsLabel

        LP.CharacterAdded:Connect(function()
            task.wait(1)
            if BS.currentIndex > 1 then
                pcall(function() _G._RaptureApplyBatSkin(BS.currentIndex) end)
            end
        end)
    end
end

do
    local RAPTURE_SKINS = {
        {
            label = "Skin 1",
            bundleId = 1656405,
            headBundleId = 141850482765573,
            assets = { 126994913914189, 136546466948928, 108541037153855 },
            headMode = "paint_black",
            korblox = false,
            animPack = "Zombie",
        },
        {
            label = "Skin 2",
            bundleId = 311,
            assets = { 16787466865, 91321960147976 },
            headMode = "headless",
            headAssetId = 15093053680,
            korblox = true,
            korbloxAssetId = 139607718,
            animPack = "Elder",
        },
        {
            label = "Skin 3",
            assets = { 111963259797542, 83472489507401, 13743316863 },
            headMode = "headless",
            headAssetId = 15093053680,
            korblox = false,
            animPack = "Vampire",
        },
        {
            label = "Skin 4",
            assets = { 87798727522412, 72316629292372, 111963259797542, 12914045493, 131260358941605 },
            headMode = "none",
            korblox = false,
            animPack = "Zombie",
        },
        { label = "OFF" },
    }
    _G._RaptureSkinList = RAPTURE_SKINS
    _G._RaptureSkinState = _G._RaptureSkinState or {
        currentIndex = 5,
        appliedItems = {},
        savedAccessories = nil,
        savedClothes = nil,
        savedHeadColor = nil,
        savedHeadTransparency = nil,
        savedRightLegParts = nil,
        savedHeadState = nil,
        savedAnimate = nil,
        savedPlayingTracks = nil,
        bundleCache = {},
    }
    local SK = _G._RaptureSkinState

    local ANIM_PACKS = {
        Zombie  = { idle1="rbxassetid://616158929", idle2="rbxassetid://616160636", walk="rbxassetid://616168032", run="rbxassetid://616163682", jump="rbxassetid://616161997", fall="rbxassetid://616157476", climb="rbxassetid://616156119", swim="rbxassetid://616165109", swimidle="rbxassetid://616166655" },
        Elder   = { idle1="rbxassetid://845397899", idle2="rbxassetid://845397899", walk="rbxassetid://845403856", run="rbxassetid://845386501", jump="rbxassetid://845398858", fall="rbxassetid://845397673", climb="rbxassetid://845392038", swim="rbxassetid://845397899", swimidle="rbxassetid://845397899" },
        Vampire = { idle1="rbxassetid://1083445855", idle2="rbxassetid://1083445855", walk="rbxassetid://1083473930", run="rbxassetid://1083462077", jump="rbxassetid://1083455352", fall="rbxassetid://1083443587", climb="rbxassetid://1083439238", swim="rbxassetid://1083445855", swimidle="rbxassetid://1083445855" },
    }

    local skAnimConn = nil
    local skCurrentAnimPack = "Off"

    local function skApplyAnimPack(char, packName)
        skCurrentAnimPack = packName
        if skAnimConn then skAnimConn:Disconnect(); skAnimConn = nil end
        if packName == "Off" or not ANIM_PACKS[packName] then return end
        local pack = ANIM_PACKS[packName]

        local animate = char:FindFirstChild("Animate")
        if not animate then
            for _ = 1, 30 do
                task.wait(0.05)
                animate = char:FindFirstChild("Animate")
                if animate then break end
            end
        end
        if not animate then return end

        local function killInternalScripts(anim)
            for _, child in ipairs(anim:GetChildren()) do
                if child:IsA("LocalScript") or child:IsA("Script") then
                    pcall(function() child:Destroy() end)
                end
            end
        end
        killInternalScripts(animate)

        local function forceAnims()
            local c = LP.Character
            if c ~= char or not c.Parent then return end
            local anim = c:FindFirstChild("Animate")
            if not anim then return end
            local function s(o, id)
                if o and o:IsA("Animation") and o.AnimationId ~= id then
                    o.AnimationId = id
                end
            end
            s(anim.idle and anim.idle.Animation1, pack.idle1)
            s(anim.idle and anim.idle.Animation2, pack.idle2)
            s(anim.walk and anim.walk.WalkAnim, pack.walk)
            s(anim.run  and anim.run.RunAnim,   pack.run)
            s(anim.jump and anim.jump.JumpAnim, pack.jump)
            s(anim.fall and anim.fall.FallAnim, pack.fall)
            s(anim.climb and anim.climb.ClimbAnim, pack.climb)
            s(anim.swim and anim.swim.Swim, pack.swim)
            s(anim.swimidle and anim.swimidle.SwimIdle, pack.swimidle)
        end

        forceAnims()

        skAnimConn = RunService.Heartbeat:Connect(function()
            local c = LP.Character
            if c ~= char or not c.Parent then return end
            if skCurrentAnimPack ~= packName then return end
            local anim = c:FindFirstChild("Animate")
            if not anim then return end
            killInternalScripts(anim)
            forceAnims()
        end)
    end

    local function skStopAnimPack()
        if skAnimConn then skAnimConn:Disconnect(); skAnimConn = nil end
        skCurrentAnimPack = "Off"
    end

    local function skLoadAsset(assetId)
        assetId = tonumber(assetId)
        if not assetId then return nil end
        local ok, objs = pcall(function() return game:GetObjects("rbxassetid://" .. tostring(assetId)) end)
        if ok and type(objs) == "table" and #objs > 0 then return objs end
        local ok2, model = pcall(function() return game:GetService("InsertService"):LoadAsset(assetId) end)
        if ok2 and model then return { model } end
        return nil
    end

    local function skExtract(obj)
        if not obj then return nil end
        if obj:IsA("Accessory") or obj:IsA("Shirt") or obj:IsA("Pants") or obj:IsA("ShirtGraphic") then return obj end
        for _, class in ipairs({"Accessory", "Shirt", "Pants", "ShirtGraphic"}) do
            local found = obj:FindFirstChildWhichIsA(class, true)
            if found then return found end
        end
        return nil
    end

    local function skWeldAccessory(char, clone)
        local handle = clone:FindFirstChild("Handle")
        if not handle or not handle:IsA("BasePart") then return end
        local att = handle:FindFirstChildWhichIsA("Attachment")
        if not att then return end
        local targetAtt = nil
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("Attachment") and d.Name == att.Name and not d:IsDescendantOf(clone) then
                targetAtt = d
                break
            end
        end
        if targetAtt and targetAtt.Parent and targetAtt.Parent:IsA("BasePart") then
            local base = targetAtt.Parent
            handle.Anchored = false; handle.CanCollide = false
            handle.CanTouch = false; handle.CanQuery = false
            handle.Massless = true
            handle.CFrame = base.CFrame * targetAtt.CFrame * att.CFrame:Inverse()
            local weld = Instance.new("WeldConstraint")
            weld.Name = "RaptureAccessoryWeld"
            weld.Part0 = base; weld.Part1 = handle; weld.Parent = handle
        end
    end

    local function skInsert(char, assetId)
        if not char or not assetId then return false end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return false end
        local objs = skLoadAsset(assetId)
        if not objs then return false end
        local applied = false
        for _, obj in ipairs(objs) do
            local item = skExtract(obj)
            if item then
                local clone = item:Clone()
                clone:SetAttribute("RaptureSkinItem", true)
                if clone:IsA("Accessory") then
                    local ok = pcall(function() hum:AddAccessory(clone) end)
                    if not ok or clone.Parent ~= char then
                        clone.Parent = char
                        skWeldAccessory(char, clone)
                    end
                    if clone.Parent ~= char then clone.Parent = char end
                    local handle = clone:FindFirstChild("Handle")
                    if handle then
                        handle.CanCollide = false; handle.Massless = true
                        if not handle:FindFirstChild("RaptureAccessoryWeld") and not handle:FindFirstChildOfClass("Weld") then
                            skWeldAccessory(char, clone)
                        end
                    end
                    applied = true
                    table.insert(SK.appliedItems, clone)
                elseif clone:IsA("Shirt") then
                    for _, old in ipairs(char:GetChildren()) do
                        if old:IsA("Shirt") then pcall(function() old:Destroy() end) end
                    end
                    clone.Name = "RaptureSkinShirt"; clone.Parent = char
                    applied = true; table.insert(SK.appliedItems, clone)
                elseif clone:IsA("Pants") then
                    for _, old in ipairs(char:GetChildren()) do
                        if old:IsA("Pants") then pcall(function() old:Destroy() end) end
                    end
                    clone.Name = "RaptureSkinPants"; clone.Parent = char
                    applied = true; table.insert(SK.appliedItems, clone)
                elseif clone:IsA("ShirtGraphic") then
                    for _, old in ipairs(char:GetChildren()) do
                        if old:IsA("ShirtGraphic") then pcall(function() old:Destroy() end) end
                    end
                    clone.Name = "RaptureSkinTShirt"; clone.Parent = char
                    applied = true; table.insert(SK.appliedItems, clone)
                end
            end
            pcall(function() obj:Destroy() end)
        end
        return applied
    end

    local function skApplyKorblox(char, assetId)
        if not char then return end
        assetId = assetId or 139607718
        if not SK.savedRightLegParts then
            SK.savedRightLegParts = {}
            for _, n in ipairs({"RightUpperLeg", "RightLowerLeg", "RightFoot"}) do
                local p = char:FindFirstChild(n)
                if p and p:IsA("BasePart") then
                    SK.savedRightLegParts[n] = { t = p.Transparency, c = p.CanCollide }
                end
            end
        end
        for _, n in ipairs({"RightUpperLeg", "RightLowerLeg", "RightFoot"}) do
            local p = char:FindFirstChild(n)
            if p and p:IsA("BasePart") then p.Transparency = 1; p.CanCollide = false end
        end
        local oldK = char:FindFirstChild("RaptureKorbloxLeg")
        if oldK then oldK:Destroy() end
        local targetPart = char:FindFirstChild("RightUpperLeg")
        if not targetPart then return end
        local objs = skLoadAsset(assetId)
        if not objs then return end
        for _, obj in ipairs(objs) do
            local bp = obj:IsA("BasePart") and obj or obj:FindFirstChildWhichIsA("BasePart", true)
            if bp then
                local clone = bp:Clone()
                clone.Name = "RaptureKorbloxLeg"
                clone.Anchored = false; clone.CanCollide = false
                clone.CanTouch = false; clone.CanQuery = false
                clone.Massless = true
                clone:SetAttribute("RaptureSkinItem", true)
                clone.CFrame = targetPart.CFrame
                local weld = Instance.new("WeldConstraint")
                weld.Part0 = targetPart; weld.Part1 = clone; weld.Parent = clone
                clone.Parent = char
                table.insert(SK.appliedItems, clone)
            end
            pcall(function() obj:Destroy() end)
        end
    end

    local function skApplyHeadMode(char, skin)
        local head = char:FindFirstChild("Head")
        if not head then return end
        if skin.headMode == "headless" then
            if SK.savedHeadTransparency == nil then SK.savedHeadTransparency = head.Transparency end
            head.Transparency = 1
            for _, d in ipairs(head:GetChildren()) do
                if d:IsA("Decal") or d:IsA("Texture") then d.Transparency = 1 end
            end
            head:SetAttribute("RaptureHeadHidden", true)
        elseif skin.headMode == "paint_black" then
            if SK.savedHeadColor == nil then SK.savedHeadColor = head.Color end
            for _, d in ipairs(head:GetChildren()) do
                if d:IsA("Decal") or d:IsA("Texture") then d.Transparency = 1 end
            end
            head.Color = Color3.fromRGB(0, 0, 0)
            head:SetAttribute("RaptureHeadBlack", true)
            local bc = char:FindFirstChildOfClass("BodyColors")
            if bc then pcall(function() bc.HeadColor3 = Color3.fromRGB(0, 0, 0) end) end
        end
    end

    local function skExpandBundle(bundleId)
        if not bundleId then return {} end
        if SK.bundleCache[bundleId] then return SK.bundleCache[bundleId] end
        local out = {}
        local ok, details = pcall(function()
            return game:GetService("AssetService"):GetBundleDetailsAsync(bundleId)
        end)
        if ok and details and type(details.Items) == "table" then
            for _, item in ipairs(details.Items) do
                if item.Type == "Asset" and tonumber(item.Id) then
                    table.insert(out, tonumber(item.Id))
                end
            end
        end
        SK.bundleCache[bundleId] = out
        return out
    end

    function _G._RaptureApplySkin(index)
        local skin = RAPTURE_SKINS[index]
        if not skin then return end
        if skin.label == "OFF" then _G._RaptureClearSkin() return end
        local char = LP.Character
        if not char then return end

        if not SK.savedAccessories then
            SK.savedAccessories = {}
            for _, obj in ipairs(char:GetChildren()) do
                if obj:IsA("Accessory") then
                    local ok, clone = pcall(function() return obj:Clone() end)
                    if ok and clone then table.insert(SK.savedAccessories, clone) end
                end
            end
        end
        if not SK.savedClothes then
            SK.savedClothes = {}
            for _, obj in ipairs(char:GetChildren()) do
                if obj:IsA("Shirt") or obj:IsA("Pants") or obj:IsA("ShirtGraphic") then
                    local ok, clone = pcall(function() return obj:Clone() end)
                    if ok and clone then SK.savedClothes[obj.ClassName] = clone end
                end
            end
        end
        if not SK.savedHeadState then
            local head = char:FindFirstChild("Head")
            local neck = char:FindFirstChild("Neck")
            SK.savedHeadState = {
                headTransparency = head and head.Transparency or 0,
                headLocalTrans = head and head.LocalTransparencyModifier or 0,
                headColor = head and head.Color or Color3.fromRGB(234, 184, 146),
                neckEnabled = neck and neck.Enabled or true,
                faceTransparency = {},
            }
            if head then
                for _, d in ipairs(head:GetChildren()) do
                    if d:IsA("Decal") or d:IsA("Texture") then
                        SK.savedHeadState.faceTransparency[d] = d.Transparency
                    end
                end
            end
        end
        if not SK.savedAnimate then
            local animate = char:FindFirstChild("Animate")
            if animate then
                SK.savedAnimate = animate:Clone()
            end
        end

        for _, obj in ipairs(char:GetChildren()) do
            if obj:GetAttribute("RaptureSkinItem") then
                pcall(function() obj:Destroy() end)
            end
        end
        SK.appliedItems = {}

        for _, obj in ipairs(char:GetChildren()) do
            if obj:IsA("Accessory") then
                pcall(function() obj:Destroy() end)
            end
        end

        for _, obj in ipairs(char:GetChildren()) do
            if obj:IsA("Shirt") or obj:IsA("Pants") or obj:IsA("ShirtGraphic") then
                pcall(function() obj:Destroy() end)
            end
        end

        if skin.bundleId then
            for _, id in ipairs(skExpandBundle(skin.bundleId)) do
                pcall(skInsert, char, id); task.wait(0.1)
            end
        end
        if skin.headBundleId then
            for _, id in ipairs(skExpandBundle(skin.headBundleId)) do
                pcall(skInsert, char, id); task.wait(0.1)
            end
        end

        pcall(skApplyHeadMode, char, skin)
        task.wait(0.1)

        for _, id in ipairs(skin.assets or {}) do
            pcall(skInsert, char, id); task.wait(0.15)
        end

        if skin.korblox then
            task.wait(0.2)
            pcall(skApplyKorblox, char, skin.korbloxAssetId)
        end

        if skin.animPack then
            task.wait(0.3)
            pcall(skApplyAnimPack, char, skin.animPack)
            task.delay(0.5, function()
                if char and char.Parent then
                    pcall(skApplyAnimPack, char, skin.animPack)
                end
            end)
        end

        task.wait(0.15)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum:ChangeState(Enum.HumanoidStateType.Running)
            task.wait(0.05)
            hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
            task.wait(0.05)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end

        SK.currentIndex = index

        task.delay(1.0, function()
            if not char or not char.Parent then return end
            local hum2 = char:FindFirstChildOfClass("Humanoid")
            if hum2 then
                local animator = hum2:FindFirstChildOfClass("Animator")
                if animator then
                    for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                        pcall(function() track:Stop() end)
                    end
                end
            end
            if skin.animPack and skApplyAnimPack then
                pcall(skApplyAnimPack, char, skin.animPack)
            end
        end)
    end

    function _G._RaptureClearSkin()
        local char = LP.Character
        if not char then return end

        for _, obj in ipairs(char:GetChildren()) do
            if obj:GetAttribute("RaptureSkinItem") then
                pcall(function() obj:Destroy() end)
            end
        end        SK.appliedItems = {}

        local head = char:FindFirstChild("Head")
        if head and SK.savedHeadState then
            head.Transparency = SK.savedHeadState.headTransparency
            head.LocalTransparencyModifier = SK.savedHeadState.headLocalTrans
            head.Color = SK.savedHeadState.headColor
            for _, d in ipairs(head:GetChildren()) do
                if (d:IsA("Decal") or d:IsA("Texture")) and SK.savedHeadState.faceTransparency[d] then
                    d.Transparency = SK.savedHeadState.faceTransparency[d]
                end
            end
            head:SetAttribute("RaptureHeadHidden", nil)
            head:SetAttribute("RaptureHeadBlack", nil)
        end

        local neck = char:FindFirstChild("Neck")
        if neck and SK.savedHeadState then
            neck.Enabled = SK.savedHeadState.neckEnabled
        end

        if SK.savedRightLegParts then
            for n, d in pairs(SK.savedRightLegParts) do
                local p = char:FindFirstChild(n)
                if p and p:IsA("BasePart") then
                    p.Transparency = d.t
                    p.CanCollide = d.c
                end
            end
            SK.savedRightLegParts = nil
        end

        if SK.savedAccessories then
            for _, clone in ipairs(SK.savedAccessories) do
                local ok, c = pcall(function() return clone:Clone() end)
                if ok and c then c.Parent = char end
            end
        end

        if SK.savedClothes then
            for _, c in pairs(SK.savedClothes) do
                local ok, cl = pcall(function() return c:Clone() end)
                if ok and cl then
                    for _, old in ipairs(char:GetChildren()) do
                        if old.ClassName == cl.ClassName then
                            pcall(function() old:Destroy() end)
                        end
                    end
                    cl.Parent = char
                end
            end
        end

        if SK.savedAnimate then
            local oldAnim = char:FindFirstChild("Animate")
            if oldAnim then pcall(function() oldAnim:Destroy() end) end
            local newAnim = SK.savedAnimate:Clone()
            newAnim.Parent = char
            task.wait(0.1)
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                local animator = hum:FindFirstChildOfClass("Animator")
                if animator then
                    for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                        pcall(function() track:Stop() end)
                    end
                end
            end
        end

        skStopAnimPack()

        task.wait(0.1)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum:ChangeState(Enum.HumanoidStateType.Running)
            task.wait(0.05)
            hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
            task.wait(0.05)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end

        SK.savedAccessories = nil
        SK.savedClothes = nil
        SK.savedHeadState = nil
        SK.savedAnimate = nil
        SK.currentIndex = 5
    end

    _G._RaptureSkinUIBuild = function(visualPage, mkRow, mkLabel)
        local skinRow = mkRow(visualPage, 42)
        mkLabel(skinRow, "Skin")

        local skinContainer = Instance.new("Frame", skinRow)
        skinContainer.Size = UDim2.new(0, 160, 1, 0)
        skinContainer.Position = UDim2.new(1, -168, 0, 0)
        skinContainer.BackgroundTransparency = 1
        skinContainer.ZIndex = 8

        local skinLeftBtn = Instance.new("TextButton", skinContainer)
        skinLeftBtn.Size = UDim2.new(0, 28, 0, 26)
        skinLeftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        skinLeftBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        skinLeftBtn.BackgroundTransparency = 0.3
        skinLeftBtn.BorderSizePixel = 0
        skinLeftBtn.Text = "<"
        skinLeftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        skinLeftBtn.Font = Enum.Font.GothamBold
        skinLeftBtn.TextSize = 13
        skinLeftBtn.AutoButtonColor = false
        skinLeftBtn.ZIndex = 9
        Instance.new("UICorner", skinLeftBtn).CornerRadius = UDim.new(0, 6)
        local sls = Instance.new("UIStroke", skinLeftBtn)
        sls.Color = PURPLE_LIGHT; sls.Thickness = 1; sls.Transparency = 0.5

        local skinLabel = Instance.new("TextLabel", skinContainer)
        skinLabel.Size = UDim2.new(0, 80, 0, 26)
        skinLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        skinLabel.BackgroundTransparency = 1
        skinLabel.Text = RAPTURE_SKINS[SK.currentIndex] and RAPTURE_SKINS[SK.currentIndex].label or "OFF"
        skinLabel.TextColor3 = PURPLE_LIGHT
        skinLabel.Font = Enum.Font.GothamBold
        skinLabel.TextSize = 12
        skinLabel.TextXAlignment = Enum.TextXAlignment.Center
        skinLabel.ZIndex = 9

        local skinRightBtn = Instance.new("TextButton", skinContainer)
        skinRightBtn.Size = UDim2.new(0, 28, 0, 26)
        skinRightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        skinRightBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        skinRightBtn.BackgroundTransparency = 0.3
        skinRightBtn.BorderSizePixel = 0
        skinRightBtn.Text = ">"
        skinRightBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        skinRightBtn.Font = Enum.Font.GothamBold
        skinRightBtn.TextSize = 13
        skinRightBtn.AutoButtonColor = false
        skinRightBtn.ZIndex = 9
        Instance.new("UICorner", skinRightBtn).CornerRadius = UDim.new(0, 6)
        local srs = Instance.new("UIStroke", skinRightBtn)
        srs.Color = PURPLE_LIGHT; srs.Thickness = 1; srs.Transparency = 0.5

        local function skinCycle(direction)
            local total = #RAPTURE_SKINS
            SK.currentIndex = SK.currentIndex + direction
            if SK.currentIndex < 1 then SK.currentIndex = total end
            if SK.currentIndex > total then SK.currentIndex = 1 end
            local newSkin = RAPTURE_SKINS[SK.currentIndex]
            skinLabel.Text = newSkin.label
            if newSkin.label == "OFF" then
                _G._RaptureClearSkin()
            else
                task.spawn(function()
                    _G._RaptureApplySkin(SK.currentIndex)
                end)
            end
            task.defer(saveAllSettings)
        end

        skinLeftBtn.MouseButton1Click:Connect(function() playArrowSound(); skinCycle(-1) end)
        skinRightBtn.MouseButton1Click:Connect(function() playArrowSound(); skinCycle(1) end)

        _G._RaptureSkinLabelRef = skinLabel

        LP.CharacterAdded:Connect(function(char)
            task.wait(1.5)
            SK.savedAccessories = nil
            SK.savedClothes = nil
            SK.savedHeadState = nil
            SK.savedAnimate = nil
            SK.savedRightLegParts = nil
            local curSkin = RAPTURE_SKINS[SK.currentIndex]
            if curSkin and curSkin.label ~= "OFF" then
                local idx = SK.currentIndex
                task.wait(0.5)
                pcall(function() _G._RaptureApplySkin(idx) end)
                if curSkin.animPack then
                    task.delay(1.5, function()
                        if char and char.Parent and LP.Character == char then
                            pcall(skApplyAnimPack, char, curSkin.animPack)
                        end
                    end)
                end
            end
        end)
    end
end

function buildGui()
    local ROW_BG = Color3.fromRGB(0, 0, 0)
    local WHITE = Color3.fromRGB(255, 255, 255)
    local INP = Color3.fromRGB(10, 10, 10)
    local TAB_INACT = Color3.fromRGB(235, 235, 245)
    local GUI_W, GUI_H = 360, 520
    local old = CoreGui:FindFirstChild("RaptureDuel")
    if old then old:Destroy() end
    local pg = LP:FindFirstChild("PlayerGui")
    if pg then local o = pg:FindFirstChild("RaptureDuel"); if o then o:Destroy() end end
    gui = Instance.new("ScreenGui")
    gui.Name = "RaptureDuel"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 10
    gui.IgnoreGuiInset = true
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
    local guiOk = pcall(function() gui.Parent = CoreGui end)
    if not guiOk then gui.Parent = LP:WaitForChild("PlayerGui") end
    main = Instance.new("Frame", gui)
    main.Size = UDim2.new(0, GUI_W, 0, GUI_H)
    main.Position = UDim2.new(0, 20, 0, 2)
    main.BackgroundColor3 = Color3.fromRGB(12, 10, 16)
    main.BackgroundTransparency = 1
    main.BorderSizePixel = 0
    main.ClipsDescendants = true
    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 18)
    local bgImage = Instance.new("ImageLabel", main)
    bgImage.Name = "BackgroundImage"
    bgImage.Size = UDim2.new(1, 0, 1, 0)
    bgImage.Position = UDim2.new(0, 0, 0, 0)
    bgImage.BackgroundColor3 = Color3.fromRGB(12, 10, 16)
    bgImage.BackgroundTransparency = 1
    bgImage.Image = "rbxassetid://" .. backgroundImages[backgroundIndex]
    bgImage.ImageTransparency = backgroundImageTransparency
    bgImage.ScaleType = Enum.ScaleType.Crop
    bgImage.ZIndex = 0
    bgImage.ClipsDescendants = true
    Instance.new("UICorner", bgImage).CornerRadius = UDim.new(0, 18)
    local topImage = Instance.new("ImageLabel", main)
    topImage.Name = "TopImage"
    topImage.Size = UDim2.new(0, 380, 0, 95)
    topImage.Position = UDim2.new(0.5, -190, 0, -10)
    topImage.BackgroundTransparency = 1
    topImage.Image = "rbxassetid://127518136639531"
    topImage.ImageColor3 = PURPLE_LIGHT
    topImage.ScaleType = Enum.ScaleType.Fit
    topImage.ZIndex = 15
    topImage.Parent = main
    mainUIScale = Instance.new("UIScale", main)
    mainUIScale.Scale = uiScaleValue / 100
    local titleFrame = Instance.new("Frame", main)
    titleFrame.Size = UDim2.new(1, -140, 0, 40)
    titleFrame.Position = UDim2.new(0, 30, 0, 84)
    titleFrame.BackgroundTransparency = 1
    titleFrame.ZIndex = 20
    titleLbl = Instance.new("TextLabel", titleFrame)
    titleLbl.Size = UDim2.new(1, 0, 1, 0)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Text = ""
    titleLbl.TextColor3 = getThemeColor()
    titleLbl.Font = Enum.Font.GothamBlack
    titleLbl.TextSize = 20
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.ZIndex = 21
    local lockBtnFrame = Instance.new("TextButton", main)
    lockBtnFrame.Size = UDim2.new(0, 32, 0, 32)
    lockBtnFrame.Position = UDim2.new(1, -80, 0, 88)
    lockBtnFrame.BackgroundColor3 = Color3.fromRGB(30,30,35)
    lockBtnFrame.BackgroundTransparency = 0.6
    lockBtnFrame.BorderSizePixel = 0
    lockBtnFrame.Text = uiLocked and "🔒" or "🔓"
    lockBtnFrame.TextColor3 = getThemeColor()
    lockBtnFrame.Font = Enum.Font.GothamBold
    lockBtnFrame.TextSize = 18
    lockBtnFrame.AutoButtonColor = false
    lockBtnFrame.ZIndex = 200
    Instance.new("UICorner", lockBtnFrame).CornerRadius = UDim.new(0, 8)
    lockBtnFrame.MouseButton1Click:Connect(function() toggleLockUI() end)
    lockBtn = lockBtnFrame
    local closeBtn = Instance.new("TextButton", main)
    closeBtn.Size = UDim2.new(0, 32, 0, 32)
    closeBtn.Position = UDim2.new(1, -42, 0, 88)
    closeBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
    closeBtn.BackgroundTransparency = 0.6
    closeBtn.BorderSizePixel = 0
    closeBtn.Text = "-"
    closeBtn.TextColor3 = WHITE
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 26
    closeBtn.AutoButtonColor = false
    closeBtn.ZIndex = 200
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)
    closeBtn.MouseButton1Click:Connect(function() hideGui() end)
    miniBtn = Instance.new("TextButton", gui)
    miniBtn.Size = UDim2.new(0, 170, 0, 52)
    miniBtn.Position = UDim2.new(0, 16, 0, 58)
    miniBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    miniBtn.BackgroundTransparency = 0
    miniBtn.BorderSizePixel = 0
    miniBtn.Text = ""
    miniBtn.TextColor3 = PURPLE_LIGHT
    miniBtn.Font = Enum.Font.GothamBlack
    miniBtn.TextSize = 16
    miniBtn.ZIndex = 20
    miniBtn.Visible = false
    miniBtn.AutoButtonColor = false
    Instance.new("UICorner", miniBtn).CornerRadius = UDim.new(1, 0)
    local miniStroke = Instance.new("UIStroke", miniBtn)
    miniStroke.Color = PURPLE_LIGHT
    miniStroke.Thickness = BTN_STROKE_THICKNESS
    miniStroke.Transparency = BTN_STROKE_TRANSPARENCY
    local miniImg = Instance.new("ImageLabel", miniBtn)
    miniImg.Name = "MiniImage"
    miniImg.AnchorPoint = Vector2.new(0.5, 0.5)
    miniImg.Size = UDim2.new(0, 120, 0, 120)
    miniImg.Position = UDim2.new(0.5, 0, 0.5, 0)
    miniImg.BackgroundTransparency = 1
    miniImg.Image = "rbxassetid://127518136639531"
    miniImg.ScaleType = Enum.ScaleType.Fit
    miniImg.ZIndex = 21
    miniImg.Parent = miniBtn
    miniBtn.MouseButton1Click:Connect(function() showGui() end)
    local slideTween = nil
    local mainOriginalPos = main.Position
    showGui = function()
        if slideTween then slideTween:Cancel() end
        if not main then return end
        main.Visible = true
        miniBtn.Visible = false
        main.Position = UDim2.new(0, -GUI_W - 20, 0, 2)
        slideTween = TS:Create(main, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = mainOriginalPos})
        slideTween:Play()
    end
    hideGui = function()
        if slideTween then slideTween:Cancel() end
        if not main or not main.Visible then return end
        local targetPos = UDim2.new(0, -GUI_W - 20, 0, 2)
        slideTween = TS:Create(main, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos})
        slideTween:Play()
        slideTween.Completed:Connect(function()
            main.Visible = false
            miniBtn.Visible = true
            slideTween = nil
        end)
    end
    local TAB_W = 80
    local tabBar = Instance.new("Frame", main)
    tabBar.Name = "SideTabBar"
    tabBar.Size = UDim2.new(0, TAB_W, 0, 280)
    tabBar.Position = UDim2.new(1, -TAB_W - 8, 0, 134)
    tabBar.BackgroundTransparency = 1
    tabBar.BorderSizePixel = 0
    tabBar.ZIndex = 10
    local tabLayout = Instance.new("UIListLayout", tabBar)
    tabLayout.FillDirection = Enum.FillDirection.Vertical
    tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    tabLayout.VerticalAlignment = Enum.VerticalAlignment.Top
    tabLayout.Padding = UDim.new(0, 10)
    tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
    local tabContent = Instance.new("Frame", main)
    tabContent.Name = "TabContent"
    tabContent.Size = UDim2.new(1, -TAB_W - 20, 1, -139)
    tabContent.Position = UDim2.new(0, 10, 0, 134)
    tabContent.BackgroundTransparency = 1
    tabContent.ClipsDescendants = true
    tabContent.ZIndex = 5
    local tabs = {"Speed", "Combat", "Visual", "Config", "Keybinds"}
    local tabButtonsLocal = {}
    local contentPagesLocal = {}
    for i, name in ipairs(tabs) do
        local btn = Instance.new("TextButton", tabBar)
        btn.Size = UDim2.new(1, -10, 0, 36)
        btn.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
        btn.BackgroundTransparency = 0.15
        btn.BorderSizePixel = 0
        btn.Text = name
        btn.TextColor3 = TAB_INACT
        btn.Font = Enum.Font.GothamBlack
        btn.TextSize = 11
        btn.TextWrapped = true
        btn.AutoButtonColor = false
        btn.ZIndex = 11
        btn.LayoutOrder = i
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)
        btn:SetAttribute("IsActiveTab", i == 1)
        if i == 1 then
            btn.BackgroundColor3 = getThemeColor()
            btn.TextColor3 = ACTIVE_TXT
            btn.BackgroundTransparency = 0
        end
        local page = Instance.new("ScrollingFrame", tabContent)
        page.Size = UDim2.new(1, 0, 1, 0)
        page.BackgroundTransparency = 1
        page.BorderSizePixel = 0
        page.ClipsDescendants = true
        page.ScrollBarThickness = 3
        page.ScrollBarImageColor3 = getThemeColor()
        page.ScrollBarImageTransparency = 0.3
        page.CanvasSize = UDim2.new(0, 0, 0, 0)
        page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        page.ScrollingDirection = Enum.ScrollingDirection.Y
        page.ZIndex = 6
        page.Visible = (i == 1)
        local layout = Instance.new("UIListLayout", page)
        layout.SortOrder = Enum.SortOrder.LayoutOrder
        layout.Padding = UDim.new(0, 10)
        layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        local padding = Instance.new("UIPadding", page)
        padding.PaddingLeft = UDim.new(0, 2)
        padding.PaddingRight = UDim.new(0, 2)
        padding.PaddingTop = UDim.new(0, 4)
        padding.PaddingBottom = UDim.new(0, 20)
        contentPagesLocal[name] = page
        btn.MouseButton1Click:Connect(function()
            playArrowSound()
            for _, pg2 in pairs(contentPagesLocal) do pg2.Visible = false end
            page.Visible = true
            for _, b in ipairs(tabButtonsLocal) do
                b.TextColor3 = TAB_INACT
                b.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
                b:SetAttribute("IsActiveTab", false)
            end
            btn.TextColor3 = ACTIVE_TXT
            btn.BackgroundColor3 = getThemeColor()
            btn:SetAttribute("IsActiveTab", true)
        end)
        table.insert(tabButtonsLocal, btn)
    end
    tabButtons = tabButtonsLocal
    contentPages = contentPagesLocal
    local pageCounters = {}
    local function getNextOrder(page)
        if not pageCounters[page] then pageCounters[page] = 0 end
        pageCounters[page] = pageCounters[page] + 1
        return pageCounters[page]
    end
    local function mkRow(page, h)
        local f = Instance.new("Frame", page)
        f.Size = UDim2.new(1, -4, 0, h or 40)
        f.BackgroundColor3 = ROW_BG
        f.BackgroundTransparency = 0.05
        f.BorderSizePixel = 0
        f.LayoutOrder = getNextOrder(page)
        f.ZIndex = 7
        Instance.new("UICorner", f).CornerRadius = UDim.new(0, 10)
        local st = Instance.new("UIStroke", f)
        st.Color = BLACK_STROKE
        st.Thickness = BTN_STROKE_THICKNESS
        st.Transparency = 0.2
        registerNeonSweep(f)
        return f
    end
    local function mkLabel(row, txt)
        local l = Instance.new("TextLabel", row)
        l.Size = UDim2.new(0.6, 0, 1, 0)
        l.Position = UDim2.new(0, 12, 0, 0)
        l.BackgroundTransparency = 1
        l.Text = txt
        l.TextColor3 = WHITE
        l.Font = Enum.Font.GothamBold
        l.TextSize = 12
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextTruncate = Enum.TextTruncate.AtEnd
        l.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        l.TextStrokeTransparency = 0.5
        l.ZIndex = 8
        return l
    end
    local function mkPill(row, offset)
        local pill = Instance.new("Frame", row)
        pill.Name = "Track"
        pill.Size = UDim2.new(0, 46, 0, 22)
        pill.AnchorPoint = Vector2.new(1, 0.5)
        pill.Position = UDim2.new(1, -(offset or 10), 0.5, 0)
        pill.BackgroundColor3 = INACTIVE_BG
        pill.BackgroundTransparency = 0
        pill.BorderSizePixel = 0
        pill.ZIndex = 8
        Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)
        local stroke = Instance.new("UIStroke", pill)
        stroke.Color = BLACK_STROKE
        stroke.Thickness = BTN_STROKE_THICKNESS
        stroke.Transparency = BTN_STROKE_TRANSPARENCY
        local dot = Instance.new("Frame", pill)
        dot.Name = "Knob"
        dot.Size = UDim2.new(0, 16, 0, 16)
        dot.Position = UDim2.new(0, 3, 0.5, -8)
        dot.BackgroundColor3 = INACTIVE_TXT
        dot.BorderSizePixel = 0
        dot.ZIndex = 9
        Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
        return pill, dot
    end
    local function animPill(pill, dot, on)
        local stroke = pill:FindFirstChildOfClass("UIStroke")
        local info = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TS:Create(dot, info, {
            Position = on and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
            BackgroundColor3 = on and Color3.fromRGB(255,255,255) or INACTIVE_TXT,
        }):Play()
        TS:Create(pill, info, {
            BackgroundColor3 = on and PURPLE_HOT or INACTIVE_BG,
            BackgroundTransparency = 0,
        }):Play()
        if stroke then
            TS:Create(stroke, info, {
                Color = on and PURPLE_HOT or BLACK_STROKE,
                Transparency = on and 0.1 or 0.3,
                Thickness = BTN_STROKE_THICKNESS,
            }):Play()
        end
    end
    local function mkToggle(page, txt, cb)
        local row = mkRow(page, 40)
        mkLabel(row, txt)
        local pill, dot = mkPill(row, 10)
        local on = false
        local function sv(s) on = s; animPill(pill, dot, s) end
        local clk = Instance.new("TextButton", pill)
        clk.Size = UDim2.new(1,0,1,0)
        clk.BackgroundTransparency = 1
        clk.Text = ""
        clk.AutoButtonColor = false
        clk.ZIndex = 10
        clk.MouseButton1Click:Connect(function()
            playToggleSound()
            on = not on
            sv(on)
            pcall(cb, on)
            task.defer(saveAllSettings)
        end)
        return sv
    end
    local function mkPillVision(row, offset)
        local pill = Instance.new("Frame", row)
        pill.Name = "Track"
        pill.Size = UDim2.new(0, 44, 0, 26)
        pill.AnchorPoint = Vector2.new(1, 0.5)
        pill.Position = UDim2.new(1, -(offset or 10), 0.5, 0)
        pill.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
        pill.BackgroundTransparency = 1
        pill.BorderSizePixel = 0
        pill.ZIndex = 8
        pill.ClipsDescendants = false
        Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)
        local dot = Instance.new("Frame", pill)
        dot.Name = "Knob"
        dot.AnchorPoint = Vector2.new(0.5, 0.5)
        dot.Size = UDim2.new(0, 18, 0, 18)
        dot.Position = UDim2.new(0.5, 0, 0.5, 0)
        dot.BackgroundColor3 = Color3.fromRGB(92, 92, 104)
        dot.BorderSizePixel = 0
        dot.ZIndex = 9
        dot.ClipsDescendants = true
        Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
        local dotGrad = Instance.new("UIGradient", dot)
        dotGrad.Name = "VisionDotGradient"
        dotGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(235, 232, 248)),
            ColorSequenceKeypoint.new(0.5, Color3.fromRGB(128, 124, 150)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(48, 48, 58)),
        })
        dotGrad.Rotation = 90
        local dotScale = Instance.new("UIScale", dot)
        dotScale.Scale = 0.92
        local stroke = Instance.new("UIStroke", dot)
        stroke.Name = "VisionStroke"
        stroke.Color = PURPLE_HOT
        stroke.Thickness = 0
        stroke.Transparency = 1
        return pill, dot
    end
    local function animPillVision(pill, dot, on)
        if not pill or not dot then return end
        local onColor = PURPLE_HOT
        local offColor = Color3.fromRGB(92, 92, 104)
        local dotGrad = dot:FindFirstChild("VisionDotGradient")
        local dotScale = dot:FindFirstChildOfClass("UIScale")
        local stroke = dot:FindFirstChild("VisionStroke")
        if dotGrad then
            if on then
                dotGrad.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 230, 255)),
                    ColorSequenceKeypoint.new(0.46, PURPLE_HOT),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(154, 62, 230)),
                })
            else
                dotGrad.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(235, 232, 248)),
                    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(180, 180, 195)),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(48, 48, 58)),
                })
            end
            dotGrad.Rotation = 90
        end
        if dotScale then
            dotScale.Scale = on and 1.08 or 0.92
            TS:Create(dotScale, TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 }):Play()
        end
        TS:Create(dot, TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 18, 0, 19),
            BackgroundColor3 = on and onColor or offColor,
        }):Play()
        if stroke then
            TS:Create(stroke, TweenInfo.new(0.22), {
                Color = on and onColor or Color3.fromRGB(255, 255, 255),
                Transparency = on and 0.10 or 1,
                Thickness = on and 1.6 or 0,
            }):Play()
        end
    end
    local function mkBox(parent, default, w, xOff, cb)
        local tb = Instance.new("TextBox", parent)
        local bw = w or 50
        local xo = math.max(xOff or 56, bw + 12)
        tb.Size = UDim2.new(0, bw, 0, 24)
        tb.Position = UDim2.new(1, -xo, 0.5, -12)
        tb.BackgroundColor3 = INP
        tb.BackgroundTransparency = 0.3
        tb.BorderSizePixel = 0
        tb.Text = tostring(default)
        tb.TextColor3 = WHITE
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 11
        tb.ClearTextOnFocus = false
        tb.ZIndex = 8
        Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 8)
        local bs = Instance.new("UIStroke", tb)
        bs.Color = BLACK_STROKE
        bs.Thickness = BTN_STROKE_THICKNESS
        bs.Transparency = 0.3
        tb.FocusLost:Connect(function()
            if cb then
                local n = tonumber(tb.Text)
                if n then cb(n) else tb.Text = tostring(default) end
                task.defer(saveAllSettings)
            end
        end)
        return tb
    end
    local function mkStepRow(page, label, getValue, setValue, minV, maxV, step)
        local row = mkRow(page, 40)
        mkLabel(row, label)
        local valLabel = Instance.new("TextLabel", row)
        valLabel.Size = UDim2.new(0, 46, 0, 24)
        valLabel.Position = UDim2.new(1, -120, 0.5, -12)
        valLabel.BackgroundColor3 = INP
        valLabel.BackgroundTransparency = 0.3
        valLabel.BorderSizePixel = 0
        valLabel.Text = tostring(getValue())
        valLabel.TextColor3 = WHITE
        valLabel.Font = Enum.Font.GothamBold
        valLabel.TextSize = 11
        valLabel.ZIndex = 8
        Instance.new("UICorner", valLabel).CornerRadius = UDim.new(0, 8)
        local minusBtn = Instance.new("TextButton", row)
        minusBtn.Size = UDim2.new(0, 26, 0, 24)
        minusBtn.Position = UDim2.new(1, -152, 0.5, -12)
        minusBtn.BackgroundColor3 = INP
        minusBtn.BackgroundTransparency = 0.3
        minusBtn.BorderSizePixel = 0
        minusBtn.Text = "-"
        minusBtn.TextColor3 = WHITE
        minusBtn.Font = Enum.Font.GothamBlack
        minusBtn.TextSize = 16
        minusBtn.ZIndex = 8
        Instance.new("UICorner", minusBtn).CornerRadius = UDim.new(0, 8)
        local plusBtn = Instance.new("TextButton", row)
        plusBtn.Size = UDim2.new(0, 26, 0, 24)
        plusBtn.Position = UDim2.new(1, -70, 0.5, -12)
        plusBtn.BackgroundColor3 = INP
        plusBtn.BackgroundTransparency = 0.3
        plusBtn.BorderSizePixel = 0
        plusBtn.Text = "+"
        plusBtn.TextColor3 = WHITE
        plusBtn.Font = Enum.Font.GothamBlack
        plusBtn.TextSize = 16
        plusBtn.ZIndex = 8
        Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 8)
        local function updateLabel() valLabel.Text = tostring(getValue()) end
        minusBtn.MouseButton1Click:Connect(function()
            playArrowSound()
            local v = getValue() - step
            if v < minV then v = minV end
            setValue(v); updateLabel(); saveAllSettings()
        end)
        plusBtn.MouseButton1Click:Connect(function()
            playArrowSound()
            local v = getValue() + step
            if v > maxV then v = maxV end
            setValue(v); updateLabel(); saveAllSettings()
        end)
        return valLabel
    end
    local function mkCollapsibleSlider(page, label, options, getValue, setValue, opts)
        opts = opts or {}
        local wrapper = Instance.new("Frame", page)
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(page)
        wrapper.ClipsDescendants = false
        wrapper.ZIndex = 7
        local row = Instance.new("Frame", wrapper)
        row.Size = UDim2.new(1, 0, 0, 40)
        row.Position = UDim2.new(0, 0, 0, 0)
        row.BackgroundColor3 = ROW_BG
        row.BackgroundTransparency = 0.05
        row.BorderSizePixel = 0
        row.ZIndex = 8
        Instance.new("UICorner", row).CornerRadius = UDim.new(0, 10)
        local rowStroke = Instance.new("UIStroke", row)
        rowStroke.Color = BLACK_STROKE
        rowStroke.Thickness = BTN_STROKE_THICKNESS
        rowStroke.Transparency = 0.2
        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(1, -100, 1, 0)
        lbl.Position = UDim2.new(0, 12, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = label
        lbl.TextColor3 = WHITE
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 12
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.TextTruncate = Enum.TextTruncate.AtEnd
        lbl.ZIndex = 9
        local currentValueLbl = Instance.new("TextLabel", row)
        currentValueLbl.Size = UDim2.new(0, 60, 1, 0)
        currentValueLbl.Position = UDim2.new(1, -96, 0, 0)
        currentValueLbl.BackgroundTransparency = 1
        currentValueLbl.Text = tostring(getValue())
        currentValueLbl.TextColor3 = PURPLE_LIGHT
        currentValueLbl.Font = Enum.Font.GothamBold
        currentValueLbl.TextSize = 11
        currentValueLbl.TextXAlignment = Enum.TextXAlignment.Right
        currentValueLbl.ZIndex = 9
        local arrowBtn = Instance.new("TextButton", row)
        arrowBtn.Name = "Arrow"
        arrowBtn.Size = UDim2.new(0, 30, 0, 30)
        arrowBtn.Position = UDim2.new(1, -36, 0.5, -15)
        arrowBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        arrowBtn.BackgroundTransparency = 0.25
        arrowBtn.BorderSizePixel = 0
        arrowBtn.Text = "v"
        arrowBtn.TextColor3 = WHITE
        arrowBtn.Font = Enum.Font.GothamBold
        arrowBtn.TextSize = 14
        arrowBtn.AutoButtonColor = false
        arrowBtn.ZIndex = 10
        Instance.new("UICorner", arrowBtn).CornerRadius = UDim.new(0, 8)
        local arrowStroke = Instance.new("UIStroke", arrowBtn)
        arrowStroke.Color = PURPLE_LIGHT
        arrowStroke.Thickness = BTN_STROKE_THICKNESS
        arrowStroke.Transparency = 0.35
        local holder = Instance.new("Frame", wrapper)
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.ZIndex = 7
        holder.Visible = false
        local sliderRow = Instance.new("Frame", holder)
        sliderRow.Size = UDim2.new(1, 0, 0, 42)
        sliderRow.Position = UDim2.new(0, 0, 0, 0)
        sliderRow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        sliderRow.BackgroundTransparency = 0.15
        sliderRow.BorderSizePixel = 0
        sliderRow.ZIndex = 8
        Instance.new("UICorner", sliderRow).CornerRadius = UDim.new(0, 10)
        local sliderBg = Instance.new("Frame", sliderRow)
        sliderBg.Size = UDim2.new(1, -16, 1, -8)
        sliderBg.Position = UDim2.new(0, 8, 0, 4)
        sliderBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        sliderBg.BackgroundTransparency = 0.2
        sliderBg.BorderSizePixel = 0
        sliderBg.ClipsDescendants = true
        sliderBg.ZIndex = 9
        Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(0, 14)
        local sliderBgStroke = Instance.new("UIStroke", sliderBg)
        sliderBgStroke.Color = BLACK_STROKE
        sliderBgStroke.Thickness = BTN_STROKE_THICKNESS
        sliderBgStroke.Transparency = 0.32
        local optionBtns = {}
        local optionTexts = {}
        local segW = 1 / #options
        for i, opt in ipairs(options) do
            local optBtn = Instance.new("TextButton", sliderBg)
            optBtn.Name = "Opt_" .. tostring(i)
            optBtn.AnchorPoint = Vector2.new(0.5, 0.5)
            optBtn.Size = UDim2.new(segW, -4, 1, -8)
            optBtn.Position = UDim2.new(segW * (i - 0.5), 0, 0.5, 0)
            optBtn.BackgroundColor3 = INACTIVE_BG
            optBtn.BackgroundTransparency = 0
            optBtn.BorderSizePixel = 0
            optBtn.Text = ""
            optBtn.AutoButtonColor = false
            optBtn.ZIndex = 10
            Instance.new("UICorner", optBtn).CornerRadius = UDim.new(0, 13)
            local optStroke = Instance.new("UIStroke", optBtn)
            optStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            optStroke.Color = BLACK_STROKE
            optStroke.Thickness = BTN_STROKE_THICKNESS
            optStroke.Transparency = 0.6
            optStroke.Parent = optBtn
            local optLbl = Instance.new("TextLabel", optBtn)
            optLbl.AnchorPoint = Vector2.new(0.5, 0.5)
            optLbl.Size = UDim2.new(1, 0, 1, 0)
            optLbl.Position = UDim2.new(0.5, 0, 0.5, 0)
            optLbl.BackgroundTransparency = 1
            optLbl.Text = opt
            optLbl.TextColor3 = INACTIVE_TXT
            optLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            optLbl.TextStrokeTransparency = 0.4
            optLbl.TextSize = 11
            optLbl.Font = Enum.Font.GothamBlack
            optLbl.TextXAlignment = Enum.TextXAlignment.Center
            optLbl.ZIndex = 11
            optLbl.Parent = optBtn
            optionBtns[i] = optBtn
            optionTexts[i] = optLbl
            optBtn.MouseButton1Click:Connect(function()
                setValue(opt)
                for j, btn in ipairs(optionBtns) do
                    local sel = (j == i)
                    TS:Create(btn, TweenInfo.new(0.18), {
                        BackgroundColor3 = sel and PURPLE_HOT or INACTIVE_BG,
                    }):Play()
                    local st = btn:FindFirstChildOfClass("UIStroke")
                    if st then
                        TS:Create(st, TweenInfo.new(0.18), {
                            Color = sel and PURPLE_HOT or BLACK_STROKE,
                            Transparency = sel and 0.1 or 0.6,
                            Thickness = BTN_STROKE_THICKNESS,
                        }):Play()
                    end
                    TS:Create(optionTexts[j], TweenInfo.new(0.14), {
                        TextColor3 = INACTIVE_TXT,
                    }):Play()
                end
                currentValueLbl.Text = tostring(opt)
                task.defer(saveAllSettings)
            end)
        end
        local function applyVisual(mode)
            local idx = 1
            for i, opt in ipairs(options) do
                if opt == mode then idx = i break end
            end
            for j, btn in ipairs(optionBtns) do
                local sel = (j == idx)
                btn.BackgroundColor3 = sel and PURPLE_HOT or INACTIVE_BG
                local st = btn:FindFirstChildOfClass("UIStroke")
                if st then
                    st.Color = sel and PURPLE_HOT or BLACK_STROKE
                    st.Transparency = sel and 0.1 or 0.6
                    st.Thickness = BTN_STROKE_THICKNESS
                end
                optionTexts[j].TextColor3 = INACTIVE_TXT
            end
            currentValueLbl.Text = tostring(mode)
        end
        local function applyThemeColor(newColor)
            local idx = 1
            for i, opt in ipairs(options) do
                if opt == getValue() then idx = i break end
            end
            for j, btn in ipairs(optionBtns) do
                if j == idx then btn.BackgroundColor3 = newColor end
            end
            arrowStroke.Color = newColor
            sliderBgStroke.Color = newColor
            currentValueLbl.TextColor3 = newColor
        end
        table.insert(_G._RaptureDynamicSliders, { applyThemeColor = applyThemeColor })
        local expanded = false
        arrowBtn.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, 42) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 84) }):Play()
                TS:Create(arrowBtn, TweenInfo.new(0.2), {Rotation = 180}):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(arrowBtn, TweenInfo.new(0.2), {Rotation = 0}):Play()
                task.delay(0.22, function() if not expanded then holder.Visible = false end end)
            end
        end)
        applyVisual(getValue())
        return applyVisual, currentValueLbl, optionBtns, optionTexts
    end
    local function mkKeyButton(parent, kbEntry)
        local btn = Instance.new("TextButton", parent)
        btn.Size = UDim2.new(0, 70, 0, 24)
        btn.Position = UDim2.new(1, -78, 0.5, -12)
        btn.BackgroundColor3 = INP
        btn.BackgroundTransparency = 0.3
        btn.BorderSizePixel = 0
        local function getLabel() return (kbEntry.gp and kbEntry.gp.Name) or (kbEntry.kb and kbEntry.kb.Name) or "None" end
        btn.Text = getLabel()
        btn.TextColor3 = WHITE
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 10
        btn.ZIndex = 8
        btn.AutoButtonColor = false
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
        local li = false; local lc; local pv = btn.Text; local listenStart = 0
        btn.Activated:Connect(function()
            if li then li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; return end
            pv = btn.Text; li = true; _anyKeyListening = true; listenStart = _tick(); btn.Text = "..."
            lc = UIS.InputBegan:Connect(function(inp)
                if not li then return end
                if inp.KeyCode == Enum.KeyCode.Escape then li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; return end
                local isGp = isGamepadInput(inp)
                if isGp and _tick()-listenStart < 0.15 then return end
                if not isBindableInput(inp) then return end
                btn.Text = inp.KeyCode.Name; pv = inp.KeyCode.Name
                li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end
                if isGp then kbEntry.gp = inp.KeyCode; kbEntry.kb = nil else kbEntry.kb = inp.KeyCode; kbEntry.gp = nil end
                task.defer(saveAllSettings)
            end)
        end)
        table.insert(keyButtonRefs, {btn = btn, entry = kbEntry})
        return btn
    end
    local function addKeybindRow(page, labelText, kbEntry)
        local row = mkRow(page, 40)
        mkLabel(row, labelText)
        mkKeyButton(row, kbEntry)
    end
    local speedPage = contentPages["Speed"]
    do local row = mkRow(speedPage); mkLabel(row, "Normal Speed"); normalBox = mkBox(row, NS, 50, 60, function(v) if v > 0 and v <= 500 then NS = v end end) end
    do local row = mkRow(speedPage); mkLabel(row, "Carry Speed"); carryBox = mkBox(row, CS, 50, 60, function(v) if v > 0 and v <= 500 then CS = v end end) end
    do local row = mkRow(speedPage); mkLabel(row, "Lagger Carry"); laggerBox = mkBox(row, LAGGER_SPEED, 50, 60, function(v) if v > 0 and v <= 500 then LAGGER_SPEED = v end end) end
    do local row = mkRow(speedPage); mkLabel(row, "Lagger"); lagger2Box = mkBox(row, LAGGER_CARRY_SPEED, 50, 60, function(v) if v > 0 and v <= 500 then LAGGER_CARRY_SPEED = v end end) end
    local combatPage = contentPages["Combat"]
    setSyncAfterHitVisual = mkToggle(combatPage, "Synchronize After Hit", function(on)
        syncAfterHitEnabled = on
        if on then
            if LP.Character then _syncStartSystems(LP.Character) end
        else
            _syncStopSystems()
        end
        task.defer(saveAllSettings)
    end)
    setBatCounterVisual = mkToggle(combatPage, "Bat Counter", function(on)
        batCounterEnabled = on
        if on then startBatCounter() else stopBatCounter() end
        task.defer(saveAllSettings)
    end)
    setMedusaVisual = mkToggle(combatPage, "Medusa Counter", function(on)
        medusaCounterEnabled = on
        if on then if LP.Character then setupMedusaCounter(LP.Character) end
        else stopMedusaCounter() end
        task.defer(saveAllSettings)
    end)

    do
        local wrapper = Instance.new("Frame", combatPage)
        wrapper.Name = "TpDownWrapper"
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(combatPage)
        wrapper.ClipsDescendants = true
        wrapper.ZIndex = 7

        local header = Instance.new("Frame", wrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hStroke = Instance.new("UIStroke", header)
        hStroke.Color = BLACK_STROKE
        hStroke.Thickness = BTN_STROKE_THICKNESS
        hStroke.Transparency = 0.2
        registerNeonSweep(header)

        local hLabel = Instance.new("TextLabel", header)
        hLabel.Size = UDim2.new(1, -180, 1, 0)
        hLabel.Position = UDim2.new(0, 12, 0, 0)
        hLabel.BackgroundTransparency = 1
        hLabel.Text = "TP DOWN"
        hLabel.TextColor3 = WHITE
        hLabel.Font = Enum.Font.GothamBold
        hLabel.TextSize = 12
        hLabel.TextXAlignment = Enum.TextXAlignment.Left
        hLabel.ZIndex = 9

        local hValue = Instance.new("TextLabel", header)
        hValue.Size = UDim2.new(0, 110, 1, 0)
        hValue.Position = UDim2.new(1, -146, 0, 0)
        hValue.BackgroundTransparency = 1
        hValue.Text = tpDownMode or "Basic"
        hValue.TextColor3 = PURPLE_LIGHT
        hValue.Font = Enum.Font.GothamBold
        hValue.TextSize = 10
        hValue.TextXAlignment = Enum.TextXAlignment.Right
        hValue.ZIndex = 9

        local hArrow = Instance.new("TextButton", header)
        hArrow.Name = "Arrow"
        hArrow.Size = UDim2.new(0, 30, 0, 30)
        hArrow.Position = UDim2.new(1, -36, 0.5, -15)
        hArrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        hArrow.BackgroundTransparency = 0.25
        hArrow.BorderSizePixel = 0
        hArrow.Text = "v"
        hArrow.TextColor3 = WHITE
        hArrow.Font = Enum.Font.GothamBold
        hArrow.TextSize = 14
        hArrow.AutoButtonColor = false
        hArrow.ZIndex = 10
        Instance.new("UICorner", hArrow).CornerRadius = UDim.new(0, 8)
        local hArrowStroke = Instance.new("UIStroke", hArrow)
        hArrowStroke.Color = PURPLE_LIGHT
        hArrowStroke.Thickness = BTN_STROKE_THICKNESS
        hArrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", wrapper)
        holder.Name = "OptionsHolder"
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local optionsBox = Instance.new("Frame", holder)
        optionsBox.Size = UDim2.new(1, 0, 0, 42)
        optionsBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        optionsBox.BackgroundTransparency = 0.15
        optionsBox.BorderSizePixel = 0
        optionsBox.ZIndex = 8
        Instance.new("UICorner", optionsBox).CornerRadius = UDim.new(0, 10)

        local bgFrame = Instance.new("Frame", optionsBox)
        bgFrame.Size = UDim2.new(1, -16, 1, -8)
        bgFrame.Position = UDim2.new(0, 8, 0, 4)
        bgFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        bgFrame.BackgroundTransparency = 0.2
        bgFrame.BorderSizePixel = 0
        bgFrame.ClipsDescendants = true
        bgFrame.ZIndex = 9
        Instance.new("UICorner", bgFrame).CornerRadius = UDim.new(0, 14)
        local bgStroke = Instance.new("UIStroke", bgFrame)
        bgStroke.Color = BLACK_STROKE
        bgStroke.Thickness = BTN_STROKE_THICKNESS
        bgStroke.Transparency = 0.32

        local optionList = { "Basic", "Best" }
        local optionBtns = {}
        local optionTexts = {}
        local segW = 1 / #optionList

        local function applyTheme(newColor)
            for j, btn in ipairs(optionBtns) do
                if optionList[j] == (tpDownMode or "Basic") then
                    btn.BackgroundColor3 = newColor
                end
            end
            hValue.TextColor3 = newColor
            hArrowStroke.Color = newColor
            bgStroke.Color = newColor
        end

        local function refresh()
            for j, btn in ipairs(optionBtns) do
                local sel = (optionList[j] == (tpDownMode or "Basic"))
                btn.BackgroundColor3 = sel and PURPLE_HOT or INACTIVE_BG
                local st = btn:FindFirstChildOfClass("UIStroke")
                if st then
                    st.Color = sel and PURPLE_HOT or BLACK_STROKE
                    st.Transparency = sel and 0.1 or 0.6
                    st.Thickness = BTN_STROKE_THICKNESS
                end
                optionTexts[j].TextColor3 = INACTIVE_TXT
            end
            hValue.Text = tpDownMode or "Basic"
        end

        for i, opt in ipairs(optionList) do
            local optBtn = Instance.new("TextButton", bgFrame)
            optBtn.Name = "Opt_" .. tostring(i)
            optBtn.AnchorPoint = Vector2.new(0.5, 0.5)
            optBtn.Size = UDim2.new(segW, -4, 1, -8)
            optBtn.Position = UDim2.new(segW * (i - 0.5), 0, 0.5, 0)
            optBtn.BackgroundColor3 = INACTIVE_BG
            optBtn.BackgroundTransparency = 0
            optBtn.BorderSizePixel = 0
            optBtn.Text = ""
            optBtn.AutoButtonColor = false
            optBtn.ZIndex = 10
            Instance.new("UICorner", optBtn).CornerRadius = UDim.new(0, 13)
            local optStroke = Instance.new("UIStroke", optBtn)
            optStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            optStroke.Color = BLACK_STROKE
            optStroke.Thickness = BTN_STROKE_THICKNESS
            optStroke.Transparency = 0.6
            optStroke.Parent = optBtn
            local optLbl = Instance.new("TextLabel", optBtn)
            optLbl.AnchorPoint = Vector2.new(0.5, 0.5)
            optLbl.Size = UDim2.new(1, 0, 1, 0)
            optLbl.Position = UDim2.new(0.5, 0, 0.5, 0)
            optLbl.BackgroundTransparency = 1
            optLbl.Text = opt
            optLbl.TextColor3 = INACTIVE_TXT
            optLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            optLbl.TextStrokeTransparency = 0.4
            optLbl.TextSize = 10
            optLbl.Font = Enum.Font.GothamBlack
            optLbl.TextXAlignment = Enum.TextXAlignment.Center
            optLbl.ZIndex = 11
            optLbl.Parent = optBtn
            optionBtns[i] = optBtn
            optionTexts[i] = optLbl

            optBtn.MouseButton1Click:Connect(function()
                tpDownMode = opt
                refresh()
                if saveAllSettings then pcall(saveAllSettings) end
            end)
        end

        table.insert(_G._RaptureDynamicSliders, { applyThemeColor = applyTheme })
        applyTheme(PURPLE_LIGHT)
        refresh()

        setTpDownVisual = function(mode)
            tpDownMode = (mode == "Best") and "Best" or "Basic"
            refresh()
        end

        tpDownRefresh = refresh

        local expanded = false
        hArrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, 42) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 84) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function() if not expanded then holder.Visible = false end end)
            end
        end)
    end

    platformSetVisual = mkToggle(combatPage, "Platform", function(on)
        if on then
            if not _Platform.enabled then platformEnable() end
        else
            if _Platform.enabled then platformDisable(true) end
        end
        task.defer(saveAllSettings)
    end)

    do
        local wrapper = Instance.new("Frame", combatPage)
        wrapper.Name = "PlatformSecondsWrapper"
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(combatPage)
        wrapper.ClipsDescendants = true
        wrapper.ZIndex = 7

        local header = Instance.new("Frame", wrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hStroke = Instance.new("UIStroke", header)
        hStroke.Color = BLACK_STROKE
        hStroke.Thickness = BTN_STROKE_THICKNESS
        hStroke.Transparency = 0.2
        registerNeonSweep(header)

        local hLabel = Instance.new("TextLabel", header)
        hLabel.Size = UDim2.new(1, -180, 1, 0)
        hLabel.Position = UDim2.new(0, 12, 0, 0)
        hLabel.BackgroundTransparency = 1
        hLabel.Text = "Platform Seconds"
        hLabel.TextColor3 = WHITE
        hLabel.Font = Enum.Font.GothamBold
        hLabel.TextSize = 12
        hLabel.TextXAlignment = Enum.TextXAlignment.Left
        hLabel.ZIndex = 9

        local hValue = Instance.new("TextLabel", header)
        hValue.Size = UDim2.new(0, 110, 1, 0)
        hValue.Position = UDim2.new(1, -146, 0, 0)
        hValue.BackgroundTransparency = 1
        hValue.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME)
        hValue.TextColor3 = PURPLE_LIGHT
        hValue.Font = Enum.Font.GothamBold
        hValue.TextSize = 10
        hValue.TextXAlignment = Enum.TextXAlignment.Right
        hValue.ZIndex = 9

        local hArrow = Instance.new("TextButton", header)
        hArrow.Name = "Arrow"
        hArrow.Size = UDim2.new(0, 30, 0, 30)
        hArrow.Position = UDim2.new(1, -36, 0.5, -15)
        hArrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        hArrow.BackgroundTransparency = 0.25
        hArrow.BorderSizePixel = 0
        hArrow.Text = "v"
        hArrow.TextColor3 = WHITE
        hArrow.Font = Enum.Font.GothamBold
        hArrow.TextSize = 14
        hArrow.AutoButtonColor = false
        hArrow.ZIndex = 10
        Instance.new("UICorner", hArrow).CornerRadius = UDim.new(0, 8)
        local hArrowStroke = Instance.new("UIStroke", hArrow)
        hArrowStroke.Color = PURPLE_LIGHT
        hArrowStroke.Thickness = BTN_STROKE_THICKNESS
        hArrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", wrapper)
        holder.Name = "OptionsHolder"
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local optionsBox = Instance.new("Frame", holder)
        optionsBox.Size = UDim2.new(1, 0, 0, 42)
        optionsBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        optionsBox.BackgroundTransparency = 0.15
        optionsBox.BorderSizePixel = 0
        optionsBox.ZIndex = 8
        Instance.new("UICorner", optionsBox).CornerRadius = UDim.new(0, 10)

        local bgFrame = Instance.new("Frame", optionsBox)
        bgFrame.Size = UDim2.new(1, -16, 1, -8)
        bgFrame.Position = UDim2.new(0, 8, 0, 4)
        bgFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        bgFrame.BackgroundTransparency = 0.2
        bgFrame.BorderSizePixel = 0
        bgFrame.ClipsDescendants = true
        bgFrame.ZIndex = 9
        Instance.new("UICorner", bgFrame).CornerRadius = UDim.new(0, 14)
        local bgStroke = Instance.new("UIStroke", bgFrame)
        bgStroke.Color = BLACK_STROKE
        bgStroke.Thickness = BTN_STROKE_THICKNESS
        bgStroke.Transparency = 0.32

        local psc = Instance.new("TextBox", bgFrame)
        psc.Name = "PlatformSecondsBox"
        psc.Size = UDim2.new(0, 90, 0, 26)
        psc.Position = UDim2.new(0.5, -45, 0.5, -13)
        psc.BackgroundColor3 = INP
        psc.BackgroundTransparency = 0.3
        psc.BorderSizePixel = 0
        psc.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME)
        psc.TextColor3 = WHITE
        psc.Font = Enum.Font.GothamBold
        psc.TextSize = 13
        psc.ClearTextOnFocus = false
        psc.ZIndex = 10
        Instance.new("UICorner", psc).CornerRadius = UDim.new(0, 8)
        local pscStroke = Instance.new("UIStroke", psc)
        pscStroke.Color = PURPLE_HOT
        pscStroke.Thickness = 1.5
        pscStroke.Transparency = 0.3
        platformSecondsBox = psc

        local function applyPlatformSeconds()
            local n = tonumber(psc.Text)
            if n then
                _Platform.AUTO_DISABLE_TIME = _clamp(n, 1, 10)
                psc.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME)
                hValue.Text = psc.Text
            else
                psc.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME)
                hValue.Text = psc.Text
            end
            task.defer(saveAllSettings)
        end
        psc.FocusLost:Connect(applyPlatformSeconds)
        pcall(function()
            psc.ReturnPressedFromOnScreenKeyboard:Connect(applyPlatformSeconds)
        end)

        local expanded = false
        hArrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, 42) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 84) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function() if not expanded then holder.Visible = false end end)
            end
        end)
    end

    setRagdollTimerVisual = mkToggle(combatPage, "Ragdoll Countdown", function(on)
        ragdollCountdownEnabled = on
        if on then hookRagdollCountdown(LP.Character)
        else
            if _G._RaptureRCD.conn then pcall(function() _G._RaptureRCD.conn:Disconnect() end); _G._RaptureRCD.conn = nil end
            if _G._RaptureRCD.charConn then pcall(function() _G._RaptureRCD.charConn:Disconnect() end); _G._RaptureRCD.charConn = nil end
            if ragdollCountdownLabel then
                ragdollCountdownLabel.Visible = false
                ragdollCountdownLabel.Text = ""
            end
        end
        task.defer(saveAllSettings)
    end)
    setAntiDieVisual = mkToggle(combatPage, "Anti Die", function(on)
        antiDieEnabled = on
        if on then _antiDie.StartEngine() else _antiDie.StopEngine() end
        task.defer(saveAllSettings)
    end)

    infiniteJumpSetVisual = mkToggle(combatPage, "Infinite Jump", function(on)
        if on then
            if InfiniteJump and InfiniteJump.start then
                InfiniteJump.start()
            end
        else
            if InfiniteJump and InfiniteJump.stop then
                InfiniteJump.stop()
            end
        end
        task.defer(saveAllSettings)
    end)
    if infiniteJumpSetVisual then
        infiniteJumpSetVisual(InfiniteJump and InfiniteJump.enabled == true)
    end

    setAntiFlingVisual = mkToggle(combatPage, "Anti Fling", function(on)
        antiFlingEnabled = on
        if on then startAntiFling() else stopAntiFling() end
        task.defer(saveAllSettings)
    end)
    setAntiRagVisual = mkToggle(combatPage, "Anti Ragdoll", function(on) setAntiRagdollEnabled(on) end)
    antiRagSetVisual = mkCollapsibleSlider(combatPage, "Anti Ragdoll Version", {"V1", "V2"},
        function() return antiRagdollVersion == "v1" and "V1" or "V2" end,
        function(v)
            local newV = (v == "V2") and "v2" or "v1"
            setAntiRagdollVersion(newV)
            if saveAmbitiousConfig then pcall(saveAmbitiousConfig) end
        end)
    setInstaGrab = mkToggle(combatPage, "Auto Steal", function(on)
        CONFIG.AUTO_STEAL_ENABLED = on
        if on then pcall(startAutoSteal) else stopAutoSteal() end
        updateProgressBarVisibility()
    end)
    do local row = mkRow(combatPage); mkLabel(row, "Steal Radius"); radInput = mkBox(row, CONFIG.STEAL_RANGE, 50, 60, function(v) if v and v >= 5 and v <= 300 then CONFIG.STEAL_RANGE = _floor(v+0.5); Steal.StealRadius = CONFIG.STEAL_RANGE; saveAllSettings() end end) end
    bodyLockSetVisual = mkToggle(combatPage, "Body Lock", function(on)
        bodyLockEnabled = on
        if on then startBodyLock() else stopBodyLock() end
        task.defer(saveAllSettings)
    end)
    do
        local row = mkRow(combatPage)
        mkLabel(row, "Body Lock Range")
        bodyLockRangeBox = mkBox(row, bodyLockRange, 50, 60, function(v)
            if v and v > 0 then
                bodyLockRange = _clamp(_floor(v), 5, 200)
                if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
            end
        end)
    end
    spinSetVisual = mkToggle(combatPage, "Spin", function(on)
        if on then
            if spinAutoBrainrotEnabled then
                spinAutoBrainrotEnabled = false
                if spinAutoBrainrotSetVisual then spinAutoBrainrotSetVisual(false) end
            end
            spinEnabled = true
            if _G._RaptureSpinSet then _G._RaptureSpinSet(true) end
        else
            spinEnabled = false
            if _G._RaptureSpinSet then _G._RaptureSpinSet(false) end
        end
        task.defer(saveAllSettings)
    end)
    do
        local row = mkRow(combatPage)
        mkLabel(row, "Speed Spin")
        spinSpeedBox = mkBox(row, spinSpeed, 50, 60, function(v)
            if v and v > 0 then
                spinSpeed = _clamp(_floor(v), 1, 9999)
                if spinSpeedBox then spinSpeedBox.Text = tostring(spinSpeed) end
            end
        end)
    end
    spinAutoBrainrotSetVisual = mkToggle(combatPage, "Spin Auto Brainrot", function(on)
        if on then
            if spinEnabled then
                spinEnabled = false
                if spinSetVisual then spinSetVisual(false) end
                if _G._RaptureSpinSet then _G._RaptureSpinSet(false) end
            end
            spinAutoBrainrotEnabled = true
            if _G._RaptureSpinAutoBrainrotSet then _G._RaptureSpinAutoBrainrotSet(true) end
        else
            spinAutoBrainrotEnabled = false
            if _G._RaptureSpinAutoBrainrotSet then _G._RaptureSpinAutoBrainrotSet(false) end
        end
        task.defer(saveAllSettings)
    end)
    autoBatSetVisual = mkToggle(combatPage, "Bat Aimbot", function(on)
        if on then enableAutoBat() else disableAutoBat() end
        if mobSetAutoBat then mobSetAutoBat(on) end
        task.defer(saveAllSettings)
    end)
    do local row = mkRow(combatPage); mkLabel(row, "Bat Aimbot Speed"); batSpeedBox = mkBox(row, BAT_AIMBOT_SPEED, 50, 60, function(v) if v > 0 and v <= 200 then BAT_AIMBOT_SPEED = v end end) end
    batBypassSetVisual = mkToggle(combatPage, "Bat Bypass", function(on)
        if on then
            if not _G._RaptureBatBypass.enabled then _G.RaptureStartBatBypass() end
        else
            if _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
        end
        task.defer(saveAllSettings)
    end)
    batDesyncTpSetVisual = mkToggle(combatPage, "TP BAT", function(on)
        if on then if not batDesyncTpEnabled then toggleBatDesyncTp() end
        else if batDesyncTpEnabled then toggleBatDesyncTp() end end
        task.defer(saveAllSettings)
    end)
    dropBrainrotSetVisual = mkToggle(combatPage, "Drop Brainrot", function(on)
        if on then executeDropWithToggle(function(v) dropBrainrotSetVisual(v); if mobSetDropBR then mobSetDropBR(v) end end) end
    end)
    autoLeftSetVisual = mkToggle(combatPage, "Auto Left", function(on)
        autoLeftEnabled = on
        if on then startAutoLeft() else stopAutoLeft() end
        if mobSetAutoLeft then mobSetAutoLeft(on) end
        task.defer(saveAllSettings)
    end)
    autoRightSetVisual = mkToggle(combatPage, "Auto Right", function(on)
        autoRightEnabled = on
        if on then startAutoRight() else stopAutoRight() end
        if mobSetAutoRight then mobSetAutoRight(on) end
        task.defer(saveAllSettings)
    end)
    local visualPage = contentPages["Visual"]

    if _G._RaptureBatSkinUIBuild then
        _G._RaptureBatSkinUIBuild(visualPage, mkRow, mkLabel)
    end
    if _G._RaptureSkinUIBuild then
        _G._RaptureSkinUIBuild(visualPage, mkRow, mkLabel)
    end

    do
        local skyRow = mkRow(visualPage, 42)
        mkLabel(skyRow, "Sky Theme")

        local skyContainer = Instance.new("Frame", skyRow)
        skyContainer.Size = UDim2.new(0, 160, 1, 0)
        skyContainer.Position = UDim2.new(1, -168, 0, 0)
        skyContainer.BackgroundTransparency = 1
        skyContainer.ZIndex = 8

        local skyLeftBtn = Instance.new("TextButton", skyContainer)
        skyLeftBtn.Size = UDim2.new(0, 28, 0, 26)
        skyLeftBtn.Position = UDim2.new(0, 0, 0.5, -13)
        skyLeftBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        skyLeftBtn.BackgroundTransparency = 0.3
        skyLeftBtn.BorderSizePixel = 0
        skyLeftBtn.Text = "<"
        skyLeftBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        skyLeftBtn.Font = Enum.Font.GothamBold
        skyLeftBtn.TextSize = 13
        skyLeftBtn.AutoButtonColor = false
        skyLeftBtn.ZIndex = 9
        Instance.new("UICorner", skyLeftBtn).CornerRadius = UDim.new(0, 6)
        local sls = Instance.new("UIStroke", skyLeftBtn)
        sls.Color = PURPLE_LIGHT; sls.Thickness = 1; sls.Transparency = 0.5

        local skyLabel = Instance.new("TextLabel", skyContainer)
        skyLabel.Size = UDim2.new(0, 80, 0, 26)
        skyLabel.Position = UDim2.new(0.5, -40, 0.5, -13)
        skyLabel.BackgroundTransparency = 1
        skyLabel.Text = skyTheme or "Off"
        skyLabel.TextColor3 = PURPLE_LIGHT
        skyLabel.Font = Enum.Font.GothamBold
        skyLabel.TextSize = 12
        skyLabel.TextXAlignment = Enum.TextXAlignment.Center
        skyLabel.ZIndex = 9

        local skyRightBtn = Instance.new("TextButton", skyContainer)
        skyRightBtn.Size = UDim2.new(0, 28, 0, 26)
        skyRightBtn.Position = UDim2.new(1, -28, 0.5, -13)
        skyRightBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
        skyRightBtn.BackgroundTransparency = 0.3
        skyRightBtn.BorderSizePixel = 0
        skyRightBtn.Text = ">"
        skyRightBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        skyRightBtn.Font = Enum.Font.GothamBold
        skyRightBtn.TextSize = 13
        skyRightBtn.AutoButtonColor = false
        skyRightBtn.ZIndex = 9
        Instance.new("UICorner", skyRightBtn).CornerRadius = UDim.new(0, 6)
        local srs = Instance.new("UIStroke", skyRightBtn)
        srs.Color = PURPLE_LIGHT; srs.Thickness = 1; srs.Transparency = 0.5

        skySelectorLabel = skyLabel

        local function skyIndexOf(name)
            for i, v in ipairs(SKY_PRESETS_LIST) do
                if v == name then return i end
            end
            return 1
        end

        local function skyCycle(direction)
            local total = #SKY_PRESETS_LIST
            local idx = skyIndexOf(skyTheme or "Off")
            idx = idx + direction
            if idx < 1 then idx = total end
            if idx > total then idx = 1 end
            skyTheme = SKY_PRESETS_LIST[idx]
            skyLabel.Text = skyTheme
            pcall(applyCustomSky, skyTheme)
            task.defer(saveAllSettings)
        end

        skyLeftBtn.MouseButton1Click:Connect(function()
            playArrowSound()
            skyCycle(-1)
        end)
        skyRightBtn.MouseButton1Click:Connect(function()
            playArrowSound()
            skyCycle(1)
        end)
    end

    do
        local xrayBaseWrapper = Instance.new("Frame", visualPage)
        xrayBaseWrapper.Name = "XrayBaseWrapper"
        xrayBaseWrapper.Size = UDim2.new(1, -4, 0, 40)
        xrayBaseWrapper.BackgroundTransparency = 1
        xrayBaseWrapper.LayoutOrder = getNextOrder(visualPage)
        xrayBaseWrapper.ClipsDescendants = true
        xrayBaseWrapper.ZIndex = 7

        local header = Instance.new("Frame", xrayBaseWrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hdrStroke = Instance.new("UIStroke", header)
        hdrStroke.Color = BLACK_STROKE
        hdrStroke.Thickness = BTN_STROKE_THICKNESS
        hdrStroke.Transparency = 0.2
        local hdrLabel = Instance.new("TextLabel", header)
        hdrLabel.Size = UDim2.new(1, -60, 1, 0)
        hdrLabel.Position = UDim2.new(0, 12, 0, 0)
        hdrLabel.BackgroundTransparency = 1
        hdrLabel.Text = "X-Ray (Base)"
        hdrLabel.TextColor3 = WHITE
        hdrLabel.Font = Enum.Font.GothamBold
        hdrLabel.TextSize = 12
        hdrLabel.TextXAlignment = Enum.TextXAlignment.Left
        hdrLabel.ZIndex = 9
        local arrow = Instance.new("TextButton", header)
        arrow.Name = "Arrow"
        arrow.Size = UDim2.new(0, 30, 0, 30)
        arrow.Position = UDim2.new(1, -36, 0.5, -15)
        arrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        arrow.BackgroundTransparency = 0.25
        arrow.BorderSizePixel = 0
        arrow.Text = "v"
        arrow.TextColor3 = WHITE
        arrow.Font = Enum.Font.GothamBold
        arrow.TextSize = 14
        arrow.AutoButtonColor = false
        arrow.ZIndex = 10
        Instance.new("UICorner", arrow).CornerRadius = UDim.new(0, 8)
        local arrowStroke = Instance.new("UIStroke", arrow)
        arrowStroke.Color = PURPLE_LIGHT
        arrowStroke.Thickness = BTN_STROKE_THICKNESS
        arrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", xrayBaseWrapper)
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local innerPage = Instance.new("Frame", holder)
        innerPage.Size = UDim2.new(1, 0, 1, 0)
        innerPage.BackgroundTransparency = 1
        innerPage.ZIndex = 8

        local xrayBaseToggleRow = Instance.new("Frame", innerPage)
        xrayBaseToggleRow.Size = UDim2.new(1, 0, 0, 40)
        xrayBaseToggleRow.BackgroundColor3 = ROW_BG
        xrayBaseToggleRow.BackgroundTransparency = 0.05
        xrayBaseToggleRow.BorderSizePixel = 0
        xrayBaseToggleRow.ZIndex = 9
        Instance.new("UICorner", xrayBaseToggleRow).CornerRadius = UDim.new(0, 10)
        local xrayBaseToggleLbl = Instance.new("TextLabel", xrayBaseToggleRow)
        xrayBaseToggleLbl.Size = UDim2.new(0.6, 0, 1, 0)
        xrayBaseToggleLbl.Position = UDim2.new(0, 12, 0, 0)
        xrayBaseToggleLbl.BackgroundTransparency = 1
        xrayBaseToggleLbl.Text = "Activar X-Ray Base"
        xrayBaseToggleLbl.TextColor3 = WHITE
        xrayBaseToggleLbl.Font = Enum.Font.GothamBold
        xrayBaseToggleLbl.TextSize = 12
        xrayBaseToggleLbl.TextXAlignment = Enum.TextXAlignment.Left
        xrayBaseToggleLbl.ZIndex = 10
        local xrayBasePill, xrayBaseDot = mkPillVision(xrayBaseToggleRow, 10)
        local xrayBaseOn = xrayBaseEnabled
        local function setXrayBasePill(s)
            xrayBaseOn = s
            animPillVision(xrayBasePill, xrayBaseDot, s)
        end
        setXrayBasePill(xrayBaseOn)
        local xrayBaseClk = Instance.new("TextButton", xrayBasePill)
        xrayBaseClk.Size = UDim2.new(1,0,1,0)
        xrayBaseClk.BackgroundTransparency = 1
        xrayBaseClk.Text = ""
        xrayBaseClk.AutoButtonColor = false
        xrayBaseClk.ZIndex = 10
        xrayBaseClk.MouseButton1Click:Connect(function()
            playToggleSound()
            xrayBaseOn = not xrayBaseOn
            setXrayBasePill(xrayBaseOn)
            xrayBaseEnabled = xrayBaseOn
            if _G._XrayBaseSet then _G._XrayBaseSet(xrayBaseOn) end
            if setXrayBaseVisual then setXrayBaseVisual(xrayBaseOn) end
            task.defer(saveAllSettings)
        end)
        setXrayBaseVisual = function(s) setXrayBasePill(s) end

        local expanded = false
        local contentHeight = 40
        arrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, contentHeight) }):Play()
                TS:Create(xrayBaseWrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 40 + contentHeight + 2) }):Play()
                TS:Create(arrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(xrayBaseWrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(arrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function()
                    if not expanded then holder.Visible = false end
                end)
            end
        end)
    end

    do
        local xrayBrainWrapper = Instance.new("Frame", visualPage)
        xrayBrainWrapper.Name = "XrayBrainWrapper"
        xrayBrainWrapper.Size = UDim2.new(1, -4, 0, 40)
        xrayBrainWrapper.BackgroundTransparency = 1
        xrayBrainWrapper.LayoutOrder = getNextOrder(visualPage)
        xrayBrainWrapper.ClipsDescendants = true
        xrayBrainWrapper.ZIndex = 7

        local header = Instance.new("Frame", xrayBrainWrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hdrStroke = Instance.new("UIStroke", header)
        hdrStroke.Color = BLACK_STROKE
        hdrStroke.Thickness = BTN_STROKE_THICKNESS
        hdrStroke.Transparency = 0.2
        local hdrLabel = Instance.new("TextLabel", header)
        hdrLabel.Size = UDim2.new(1, -60, 1, 0)
        hdrLabel.Position = UDim2.new(0, 12, 0, 0)
        hdrLabel.BackgroundTransparency = 1
        hdrLabel.Text = "X-Ray (Brainrot)"
        hdrLabel.TextColor3 = WHITE
        hdrLabel.Font = Enum.Font.GothamBold
        hdrLabel.TextSize = 12
        hdrLabel.TextXAlignment = Enum.TextXAlignment.Left
        hdrLabel.ZIndex = 9
        local arrow = Instance.new("TextButton", header)
        arrow.Name = "Arrow"
        arrow.Size = UDim2.new(0, 30, 0, 30)
        arrow.Position = UDim2.new(1, -36, 0.5, -15)
        arrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        arrow.BackgroundTransparency = 0.25
        arrow.BorderSizePixel = 0
        arrow.Text = "v"
        arrow.TextColor3 = WHITE
        arrow.Font = Enum.Font.GothamBold
        arrow.TextSize = 14
        arrow.AutoButtonColor = false
        arrow.ZIndex = 10
        Instance.new("UICorner", arrow).CornerRadius = UDim.new(0, 8)
        local arrowStroke = Instance.new("UIStroke", arrow)
        arrowStroke.Color = PURPLE_LIGHT
        arrowStroke.Thickness = BTN_STROKE_THICKNESS
        arrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", xrayBrainWrapper)
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local innerPage = Instance.new("Frame", holder)
        innerPage.Size = UDim2.new(1, 0, 1, 0)
        innerPage.BackgroundTransparency = 1
        innerPage.ZIndex = 8

        local xrayBrainToggleRow = Instance.new("Frame", innerPage)
        xrayBrainToggleRow.Size = UDim2.new(1, 0, 0, 40)
        xrayBrainToggleRow.BackgroundColor3 = ROW_BG
        xrayBrainToggleRow.BackgroundTransparency = 0.05
        xrayBrainToggleRow.BorderSizePixel = 0
        xrayBrainToggleRow.ZIndex = 9
        Instance.new("UICorner", xrayBrainToggleRow).CornerRadius = UDim.new(0, 10)
        local xrayBrainToggleLbl = Instance.new("TextLabel", xrayBrainToggleRow)
        xrayBrainToggleLbl.Size = UDim2.new(0.6, 0, 1, 0)
        xrayBrainToggleLbl.Position = UDim2.new(0, 12, 0, 0)
        xrayBrainToggleLbl.BackgroundTransparency = 1
        xrayBrainToggleLbl.Text = "Activar X-Ray Brainrot"
        xrayBrainToggleLbl.TextColor3 = WHITE
        xrayBrainToggleLbl.Font = Enum.Font.GothamBold
        xrayBrainToggleLbl.TextSize = 12
        xrayBrainToggleLbl.TextXAlignment = Enum.TextXAlignment.Left
        xrayBrainToggleLbl.ZIndex = 10
        local xrayBrainPill, xrayBrainDot = mkPillVision(xrayBrainToggleRow, 10)
        local xrayBrainOn = xrayBrainrotEnabled
        local function setXrayBrainPill(s)
            xrayBrainOn = s
            animPillVision(xrayBrainPill, xrayBrainDot, s)
        end
        setXrayBrainPill(xrayBrainOn)
        local xrayBrainClk = Instance.new("TextButton", xrayBrainPill)
        xrayBrainClk.Size = UDim2.new(1,0,1,0)
        xrayBrainClk.BackgroundTransparency = 1
        xrayBrainClk.Text = ""
        xrayBrainClk.AutoButtonColor = false
        xrayBrainClk.ZIndex = 10
        xrayBrainClk.MouseButton1Click:Connect(function()
            playToggleSound()
            xrayBrainOn = not xrayBrainOn
            setXrayBrainPill(xrayBrainOn)
            xrayBrainrotEnabled = xrayBrainOn
            if _G._XrayBrainrotSet then _G._XrayBrainrotSet(xrayBrainOn) end
            if setXrayBrainrotVisual then setXrayBrainrotVisual(xrayBrainOn) end
            task.defer(saveAllSettings)
        end)
        setXrayBrainrotVisual = function(s) setXrayBrainPill(s) end

        local expanded = false
        local contentHeight = 40
        arrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, contentHeight) }):Play()
                TS:Create(xrayBrainWrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 40 + contentHeight + 2) }):Play()
                TS:Create(arrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(xrayBrainWrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(arrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function()
                    if not expanded then holder.Visible = false end
                end)
            end
        end)
    end

    do
        local wrapper = Instance.new("Frame", visualPage)
        wrapper.Name = "SpeedDisplayWrapper"
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(visualPage)
        wrapper.ClipsDescendants = true
        wrapper.ZIndex = 7

        local header = Instance.new("Frame", wrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hStroke = Instance.new("UIStroke", header)
        hStroke.Color = BLACK_STROKE
        hStroke.Thickness = BTN_STROKE_THICKNESS
        hStroke.Transparency = 0.2
        registerNeonSweep(header)

        local hLabel = Instance.new("TextLabel", header)
        hLabel.Size = UDim2.new(1, -180, 1, 0)
        hLabel.Position = UDim2.new(0, 12, 0, 0)
        hLabel.BackgroundTransparency = 1
        hLabel.Text = "Speed Display"
        hLabel.TextColor3 = WHITE
        hLabel.Font = Enum.Font.GothamBold
        hLabel.TextSize = 12
        hLabel.TextXAlignment = Enum.TextXAlignment.Left
        hLabel.ZIndex = 9

        local hValue = Instance.new("TextLabel", header)
        hValue.Size = UDim2.new(0, 110, 1, 0)
        hValue.Position = UDim2.new(1, -146, 0, 0)
        hValue.BackgroundTransparency = 1
        hValue.Text = speedDisplayMode or "Speed and Mode"
        hValue.TextColor3 = PURPLE_LIGHT
        hValue.Font = Enum.Font.GothamBold
        hValue.TextSize = 10
        hValue.TextXAlignment = Enum.TextXAlignment.Right
        hValue.ZIndex = 9

        local hArrow = Instance.new("TextButton", header)
        hArrow.Name = "Arrow"
        hArrow.Size = UDim2.new(0, 30, 0, 30)
        hArrow.Position = UDim2.new(1, -36, 0.5, -15)
        hArrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        hArrow.BackgroundTransparency = 0.25
        hArrow.BorderSizePixel = 0
        hArrow.Text = "v"
        hArrow.TextColor3 = WHITE
        hArrow.Font = Enum.Font.GothamBold
        hArrow.TextSize = 14
        hArrow.AutoButtonColor = false
        hArrow.ZIndex = 10
        Instance.new("UICorner", hArrow).CornerRadius = UDim.new(0, 8)
        local hArrowStroke = Instance.new("UIStroke", hArrow)
        hArrowStroke.Color = PURPLE_LIGHT
        hArrowStroke.Thickness = BTN_STROKE_THICKNESS
        hArrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", wrapper)
        holder.Name = "OptionsHolder"
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local optionsBox = Instance.new("Frame", holder)
        optionsBox.Size = UDim2.new(1, 0, 0, 42)
        optionsBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        optionsBox.BackgroundTransparency = 0.15
        optionsBox.BorderSizePixel = 0
        optionsBox.ZIndex = 8
        Instance.new("UICorner", optionsBox).CornerRadius = UDim.new(0, 10)

        local bgFrame = Instance.new("Frame", optionsBox)
        bgFrame.Size = UDim2.new(1, -16, 1, -8)
        bgFrame.Position = UDim2.new(0, 8, 0, 4)
        bgFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        bgFrame.BackgroundTransparency = 0.2
        bgFrame.BorderSizePixel = 0
        bgFrame.ClipsDescendants = true
        bgFrame.ZIndex = 9
        Instance.new("UICorner", bgFrame).CornerRadius = UDim.new(0, 14)
        local bgStroke = Instance.new("UIStroke", bgFrame)
        bgStroke.Color = BLACK_STROKE
        bgStroke.Thickness = BTN_STROKE_THICKNESS
        bgStroke.Transparency = 0.32

        local optionList = { "Speed Only", "Mode Only", "Speed and Mode" }
        local optionBtns = {}
        local optionTexts = {}
        local segW = 1 / #optionList

        local function applyTheme(newColor)
            for j, btn in ipairs(optionBtns) do
                if optionList[j] == (speedDisplayMode or "Speed and Mode") then
                    btn.BackgroundColor3 = newColor
                end
            end
            hValue.TextColor3 = newColor
            hArrowStroke.Color = newColor
            bgStroke.Color = newColor
        end

        local function refresh()
            for j, btn in ipairs(optionBtns) do
                local sel = (optionList[j] == (speedDisplayMode or "Speed and Mode"))
                btn.BackgroundColor3 = sel and PURPLE_HOT or INACTIVE_BG
                local st = btn:FindFirstChildOfClass("UIStroke")
                if st then
                    st.Color = sel and PURPLE_HOT or BLACK_STROKE
                    st.Transparency = sel and 0.1 or 0.6
                    st.Thickness = BTN_STROKE_THICKNESS
                end
                optionTexts[j].TextColor3 = INACTIVE_TXT
            end
            hValue.Text = speedDisplayMode or "Speed and Mode"
            updateSpeedDisplayText()
        end

        for i, opt in ipairs(optionList) do
            local optBtn = Instance.new("TextButton", bgFrame)
            optBtn.Name = "Opt_" .. tostring(i)
            optBtn.AnchorPoint = Vector2.new(0.5, 0.5)
            optBtn.Size = UDim2.new(segW, -4, 1, -8)
            optBtn.Position = UDim2.new(segW * (i - 0.5), 0, 0.5, 0)
            optBtn.BackgroundColor3 = INACTIVE_BG
            optBtn.BackgroundTransparency = 0
            optBtn.BorderSizePixel = 0
            optBtn.Text = ""
            optBtn.AutoButtonColor = false
            optBtn.ZIndex = 10
            Instance.new("UICorner", optBtn).CornerRadius = UDim.new(0, 13)
            local optStroke = Instance.new("UIStroke", optBtn)
            optStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            optStroke.Color = BLACK_STROKE
            optStroke.Thickness = BTN_STROKE_THICKNESS
            optStroke.Transparency = 0.6
            optStroke.Parent = optBtn
            local optLbl = Instance.new("TextLabel", optBtn)
            optLbl.AnchorPoint = Vector2.new(0.5, 0.5)
            optLbl.Size = UDim2.new(1, 0, 1, 0)
            optLbl.Position = UDim2.new(0.5, 0, 0.5, 0)
            optLbl.BackgroundTransparency = 1
            optLbl.Text = opt
            optLbl.TextColor3 = INACTIVE_TXT
            optLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            optLbl.TextStrokeTransparency = 0.4
            optLbl.TextSize = 10
            optLbl.Font = Enum.Font.GothamBlack
            optLbl.TextXAlignment = Enum.TextXAlignment.Center
            optLbl.ZIndex = 11
            optLbl.Parent = optBtn
            optionBtns[i] = optBtn
            optionTexts[i] = optLbl

            optBtn.MouseButton1Click:Connect(function()
                speedDisplayMode = opt
                refresh()
                if saveAllSettings then pcall(saveAllSettings) end
            end)
        end

        table.insert(_G._RaptureDynamicSliders, { applyThemeColor = applyTheme })
        applyTheme(PURPLE_LIGHT)
        refresh()

        _G._RaptureSpeedDisplayRefresh = refresh

        local expanded = false
        hArrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, 42) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 84) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function() if not expanded then holder.Visible = false end end)
            end
        end)

        speedDisplayRefresh = refresh
    end

    setShowSpeedVisual = mkToggle(visualPage, "Show Speed", function(on)
        showSpeedLabel = on
        if speedLabel then
            speedLabel.Visible = on
            if on then
                updateSpeedDisplayText()
            else
                speedLabel.Text = ""
            end
        end
        task.defer(saveAllSettings)
    end)
    if setShowSpeedVisual then setShowSpeedVisual(showSpeedLabel) end
    if speedLabel then
        speedLabel.Visible = showSpeedLabel
        if showSpeedLabel then updateSpeedDisplayText() else speedLabel.Text = "" end
    end

    setUnwalkVisual = mkToggle(visualPage, "Unwalk", function(on)
        unwalkEnabled = on
        if on then
            startUnwalk()
        else
            stopUnwalk()
        end
        task.defer(saveAllSettings)
    end)

    setESPVisual = mkToggle(visualPage, "ESP Players", function(on)
        if on then startESP() else stopESP() end
        task.defer(saveAllSettings)
    end)

    setAntiLagVisual = mkToggle(visualPage, "Anti Lag", function(on)
        if on then enableAntiLag() else disableAntiLag() end
        task.defer(saveAllSettings)
    end)
    do
        local row = mkRow(visualPage, 40)
        mkLabel(row, "Stretch Rez")
        local stretchPill, stretchDot = mkPill(row, 10)
        local stretchOn = false
        local function setStretch(s)
            stretchOn = s
            animPill(stretchPill, stretchDot, s)
            if s then enableStretch() else disableStretch() end
            stretchEnabled = s
            task.defer(saveAllSettings)
        end
        local stretchClk = Instance.new("TextButton", stretchPill)
        stretchClk.Size = UDim2.new(1,0,1,0)
        stretchClk.BackgroundTransparency = 1
        stretchClk.Text = ""
        stretchClk.ZIndex = 10
        stretchClk.MouseButton1Click:Connect(function()
            playToggleSound()
            setStretch(not stretchOn)
        end)
        _G.stretchToggleSetter = setStretch
    end
    do local row = mkRow(visualPage); mkLabel(row, "Stretch Value"); stretchValueBox = mkBox(row, stretchValue, 50, 60, function(v) if v and v >= 0.3 and v <= 1.5 then stretchValue = v; if stretchEnabled then disableStretch(); enableStretch() end; saveAllSettings() end end) end
    setFovVisual = mkToggle(visualPage, "FOV", function(on)
        if on then enableCustomFov() else disableCustomFov() end
        task.defer(saveAllSettings)
    end)
    if setFovVisual then setFovVisual(fovEnabled) end
    do
        local row = mkRow(visualPage)
        mkLabel(row, "FOV Value")
        local fovBox = mkBox(row, fovValue, 50, 60, function(v)
            if v and v >= 20 and v <= 120 then
                fovValue = _floor(v + 0.5)
                if not fovEnabled then enableCustomFov(); if setFovVisual then setFovVisual(true) end end
                local cam = workspace.CurrentCamera
                if cam then pcall(function() cam.FieldOfView = fovValue end) end
                saveAllSettings()
            end
        end)
        fovSliderSet = function(val) if fovBox then fovBox.Text = tostring(val) end end
    end
    local customSoundsSetVisual = mkToggle(visualPage, "Custom Sounds", function(on)
        customSoundsEnabled = on
        if _G.RaptureCustomSounds then _G.RaptureCustomSounds.setEnabled(on) end
        task.defer(saveAllSettings)
    end)
    setCustomSoundsVisual = customSoundsSetVisual
    local configPage = contentPages["Config"]
    uiScaleBox = mkStepRow(configPage, "UI Scale",
        function() return uiScaleValue end,
        function(v)
            uiScaleValue = v
            if mainUIScale then mainUIScale.Scale = v / 100 end
        end, 50, 150, 1)
    progressBarScaleBox = mkStepRow(configPage, "Progress Bar Scale",
        function() return progressBarScaleValue end,
        function(v)
            progressBarScaleValue = v
            if pbScale then pbScale.Scale = v / 100 end
        end, 30, 150, 1)
    mkStepRow(configPage, "Button Scale",
        function() return _floor(floatingButtonScale * 100) end,
        function(v)
            floatingButtonScale = v / 100
            applyFloatingButtonScale()
        end, 50, 200, 1)
    do
        local row = mkRow(configPage, 44)
        mkLabel(row, "Background Image")
        local btn = Instance.new("TextButton", row)
        btn.Size = UDim2.new(0, 80, 0, 24)
        btn.Position = UDim2.new(1, -92, 0.5, -12)
        btn.BackgroundColor3 = Color3.fromRGB(18, 14, 24)
        btn.BackgroundTransparency = 0.3
        btn.BorderSizePixel = 0
        btn.Text = "PICK"
        btn.TextColor3 = getThemeColor()
        btn.Font = Enum.Font.GothamBlack
        btn.TextSize = 10
        btn.ZIndex = 8
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)
        btn.Activated:Connect(function() openImagePicker() end)
    end

    skipIntroToggleSet = mkToggle(configPage, "Skip Intro", function(on)
        skipIntroEnabled = on
        task.defer(saveAllSettings)
    end)
    if skipIntroToggleSet then skipIntroToggleSet(skipIntroEnabled) end

    do
        local wrapper = Instance.new("Frame", configPage)
        wrapper.Name = "SoundEffectsWrapper"
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(configPage)
        wrapper.ClipsDescendants = true
        wrapper.ZIndex = 7

        local header = Instance.new("Frame", wrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hStroke = Instance.new("UIStroke", header)
        hStroke.Color = BLACK_STROKE
        hStroke.Thickness = BTN_STROKE_THICKNESS
        hStroke.Transparency = 0.2
        registerNeonSweep(header)

        local hLabel = Instance.new("TextLabel", header)
        hLabel.Size = UDim2.new(1, -60, 1, 0)
        hLabel.Position = UDim2.new(0, 12, 0, 0)
        hLabel.BackgroundTransparency = 1
        hLabel.Text = "Sound Effects"
        hLabel.TextColor3 = WHITE
        hLabel.Font = Enum.Font.GothamBold
        hLabel.TextSize = 12
        hLabel.TextXAlignment = Enum.TextXAlignment.Left
        hLabel.ZIndex = 9

        local hArrow = Instance.new("TextButton", header)
        hArrow.Name = "Arrow"
        hArrow.Size = UDim2.new(0, 30, 0, 30)
        hArrow.Position = UDim2.new(1, -36, 0.5, -15)
        hArrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        hArrow.BackgroundTransparency = 0.25
        hArrow.BorderSizePixel = 0
        hArrow.Text = "v"
        hArrow.TextColor3 = WHITE
        hArrow.Font = Enum.Font.GothamBold
        hArrow.TextSize = 14
        hArrow.AutoButtonColor = false
        hArrow.ZIndex = 10
        Instance.new("UICorner", hArrow).CornerRadius = UDim.new(0, 8)
        local hArrowStroke = Instance.new("UIStroke", hArrow)
        hArrowStroke.Color = PURPLE_LIGHT
        hArrowStroke.Thickness = BTN_STROKE_THICKNESS
        hArrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", wrapper)
        holder.Name = "OptionsHolder"
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local innerScroll = Instance.new("ScrollingFrame", holder)
        innerScroll.Size = UDim2.new(1, 0, 1, 0)
        innerScroll.BackgroundTransparency = 1
        innerScroll.BorderSizePixel = 0
        innerScroll.ClipsDescendants = true
        innerScroll.ScrollBarThickness = 3
        innerScroll.ScrollBarImageColor3 = getThemeColor()
        innerScroll.ScrollBarImageTransparency = 0.35
        innerScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        innerScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
        innerScroll.ScrollingDirection = Enum.ScrollingDirection.Y
        innerScroll.ZIndex = 8

        local innerLayout = Instance.new("UIListLayout", innerScroll)
        innerLayout.SortOrder = Enum.SortOrder.LayoutOrder
        innerLayout.Padding = UDim.new(0, 6)
        innerLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

        local innerPad = Instance.new("UIPadding", innerScroll)
        innerPad.PaddingLeft = UDim.new(0, 6)
        innerPad.PaddingRight = UDim.new(0, 6)
        innerPad.PaddingTop = UDim.new(0, 6)
        innerPad.PaddingBottom = UDim.new(0, 6)

        local soundOptions = {
            { key = "buttons", label = "Sound Buttons" },
            { key = "tab",     label = "Sound Tab" },
            { key = "toggle",  label = "Sound Toggle" },
            { key = "arrow",   label = "Sound Arrow" },
        }

        local soundGetters = {
            buttons = function() return soundButtonsEnabled end,
            tab     = function() return soundTabEnabled end,
            toggle  = function() return soundToggleEnabled end,
            arrow   = function() return soundArrowEnabled end,
        }
        local soundSetters = {
            buttons = function(v)
                soundButtonsEnabled = v
                if v == false then stopAllSoundsOfType("button") end
            end,
            tab = function(v)
                soundTabEnabled = v
                if v == false then stopAllSoundsOfType("arrow") end
            end,
            toggle = function(v)
                soundToggleEnabled = v
                if v == false then stopAllSoundsOfType("toggle") end
            end,
            arrow = function(v)
                soundArrowEnabled = v
                if v == false then stopAllSoundsOfType("arrow") end
            end,
        }

        local soundPillRefs = {}

        for i, entry in ipairs(soundOptions) do
            local row = Instance.new("Frame", innerScroll)
            row.Name = "SoundRow_" .. entry.key
            row.Size = UDim2.new(1, -6, 0, 30)
            row.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            row.BackgroundTransparency = 0.05
            row.BorderSizePixel = 0
            row.LayoutOrder = i
            row.ZIndex = 9
            Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)
            local rowStroke = Instance.new("UIStroke", row)
            rowStroke.Color = BLACK_STROKE
            rowStroke.Thickness = BTN_STROKE_THICKNESS
            rowStroke.Transparency = 0.25

            local lbl = Instance.new("TextLabel", row)
            lbl.Size = UDim2.new(1, -60, 1, 0)
            lbl.Position = UDim2.new(0, 10, 0, 0)
            lbl.BackgroundTransparency = 1
            lbl.Text = entry.label
            lbl.TextColor3 = WHITE
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            lbl.ZIndex = 10

            local pill, dot = mkPillVision(row, 10)
            local isOn = soundGetters[entry.key]()
            local function sv(state)
                animPillVision(pill, dot, state)
                soundSetters[entry.key](state)
                task.defer(saveAllSettings)
            end
            local clk = Instance.new("TextButton", pill)
            clk.Size = UDim2.new(1, 0, 1, 0)
            clk.BackgroundTransparency = 1
            clk.Text = ""
            clk.AutoButtonColor = false
            clk.ZIndex = 11
            clk.MouseButton1Click:Connect(function()
                playToggleSound()
                isOn = not isOn
                sv(isOn)
            end)
            sv(isOn)

            soundPillRefs[entry.key] = { setVisual = sv, get = soundGetters[entry.key] }

            if entry.key == "buttons" then setSoundButtonsVisual = sv
            elseif entry.key == "tab" then setSoundTabVisual = sv
            elseif entry.key == "toggle" then setSoundToggleVisual = sv
            elseif entry.key == "arrow" then setSoundArrowVisual = sv
            end
        end

        _refreshSoundListVisuals = function()
            for key, ref in pairs(soundPillRefs) do
                pcall(function() ref.setVisual(ref.get()) end)
            end
        end

        task.defer(function()
            if innerScroll and innerScroll.Parent then
                innerScroll.CanvasSize = UDim2.new(0, 0, 0, innerLayout.AbsoluteContentSize.Y + 12)
            end
        end)

        local expanded = false
        local contentHeight = #soundOptions * 30 + (#soundOptions - 1) * 6 + 12

        hArrow.MouseButton1Click:Connect(function()
            playArrowSound()
            expanded = not expanded
            if expanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, contentHeight) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 40 + contentHeight + 2) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(hArrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function() if not expanded then holder.Visible = false end end)
            end
        end)
    end

    local hideMobileMasterSetVisual = mkToggle(configPage, "Hide Mobile Buttons", function(on)
        _G._RaptureHideMobileButtons = on
        _G._RaptureApplyMobileButtonsHidden()
        task.defer(saveAllSettings)
    end)

    do
        local wrapper = Instance.new("Frame", configPage)
        wrapper.Name = "ButtonsToHideWrapper"
        wrapper.Size = UDim2.new(1, -4, 0, 40)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = getNextOrder(configPage)
        wrapper.ClipsDescendants = true
        wrapper.ZIndex = 7

        local header = Instance.new("Frame", wrapper)
        header.Size = UDim2.new(1, 0, 0, 40)
        header.BackgroundColor3 = ROW_BG
        header.BackgroundTransparency = 0.05
        header.BorderSizePixel = 0
        header.ZIndex = 8
        Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)
        local hdrStroke = Instance.new("UIStroke", header)
        hdrStroke.Color = BLACK_STROKE
        hdrStroke.Thickness = BTN_STROKE_THICKNESS
        hdrStroke.Transparency = 0.2
        registerNeonSweep(header)

        local hdrLabel = Instance.new("TextLabel", header)
        hdrLabel.Size = UDim2.new(1, -60, 1, 0)
        hdrLabel.Position = UDim2.new(0, 12, 0, 0)
        hdrLabel.BackgroundTransparency = 1
        hdrLabel.Text = "Buttons to Hide"
        hdrLabel.TextColor3 = WHITE
        hdrLabel.Font = Enum.Font.GothamBold
        hdrLabel.TextSize = 12
        hdrLabel.TextXAlignment = Enum.TextXAlignment.Left
        hdrLabel.ZIndex = 9

        local hdrArrow = Instance.new("TextButton", header)
        hdrArrow.Name = "Arrow"
        hdrArrow.Size = UDim2.new(0, 30, 0, 30)
        hdrArrow.Position = UDim2.new(1, -36, 0.5, -15)
        hdrArrow.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
        hdrArrow.BackgroundTransparency = 0.25
        hdrArrow.BorderSizePixel = 0
        hdrArrow.Text = "v"
        hdrArrow.TextColor3 = WHITE
        hdrArrow.Font = Enum.Font.GothamBold
        hdrArrow.TextSize = 14
        hdrArrow.AutoButtonColor = false
        hdrArrow.ZIndex = 10
        Instance.new("UICorner", hdrArrow).CornerRadius = UDim.new(0, 8)
        local hdrArrowStroke = Instance.new("UIStroke", hdrArrow)
        hdrArrowStroke.Color = PURPLE_LIGHT
        hdrArrowStroke.Thickness = BTN_STROKE_THICKNESS
        hdrArrowStroke.Transparency = 0.35

        local holder = Instance.new("Frame", wrapper)
        holder.Name = "OptionsHolder"
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.Position = UDim2.new(0, 0, 0, 42)
        holder.BackgroundTransparency = 1
        holder.ClipsDescendants = true
        holder.Visible = false
        holder.ZIndex = 7

        local innerScroll = Instance.new("ScrollingFrame", holder)
        innerScroll.Size = UDim2.new(1, 0, 1, 0)
        innerScroll.Position = UDim2.new(0, 0, 0, 0)
        innerScroll.BackgroundTransparency = 1
        innerScroll.BorderSizePixel = 0
        innerScroll.ClipsDescendants = true
        innerScroll.ScrollBarThickness = 3
        innerScroll.ScrollBarImageColor3 = getThemeColor()
        innerScroll.ScrollBarImageTransparency = 0.35
        innerScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        innerScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
        innerScroll.ScrollingDirection = Enum.ScrollingDirection.Y
        innerScroll.ElasticBehavior = Enum.ElasticBehavior.WhenScrollable
        innerScroll.ZIndex = 8

        local innerLayout = Instance.new("UIListLayout", innerScroll)
        innerLayout.SortOrder = Enum.SortOrder.LayoutOrder
        innerLayout.Padding = UDim.new(0, 6)
        innerLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

        local innerPad = Instance.new("UIPadding", innerScroll)
        innerPad.PaddingLeft = UDim.new(0, 6)
        innerPad.PaddingRight = UDim.new(0, 6)
        innerPad.PaddingTop = UDim.new(0, 6)
        innerPad.PaddingBottom = UDim.new(0, 6)

        local hideListData = {
            { key = "tpBat",       label = "TP Bat" },
            { key = "batBypass",   label = "Bat Bypass" },
            { key = "aimbot",      label = "Bat Aimbot" },
            { key = "carry",       label = "Carry Speed" },
            { key = "lagger",      label = "Lagger" },
            { key = "laggerCarry", label = "Lagger Carry" },
            { key = "instaReset",  label = "Insta Reset" },
            { key = "autoLeft",    label = "Auto Left" },
            { key = "autoRight",   label = "Auto Right" },
            { key = "drop",        label = "Drop Brainrot" },
            { key = "tp",          label = "TP Down" },
            { key = "platform",    label = "Platform" },
        }

        local hidePillRefs = {}

        for i, entry in ipairs(hideListData) do
            local row = Instance.new("Frame", innerScroll)
            row.Name = "HideRow_" .. entry.key
            row.Size = UDim2.new(1, -6, 0, 30)
            row.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            row.BackgroundTransparency = 0.05
            row.BorderSizePixel = 0
            row.LayoutOrder = i
            row.ZIndex = 9
            Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)
            local rowStroke = Instance.new("UIStroke", row)
            rowStroke.Color = BLACK_STROKE
            rowStroke.Thickness = BTN_STROKE_THICKNESS
            rowStroke.Transparency = 0.25

            local lbl = Instance.new("TextLabel", row)
            lbl.Size = UDim2.new(1, -60, 1, 0)
            lbl.Position = UDim2.new(0, 10, 0, 0)
            lbl.BackgroundTransparency = 1
            lbl.Text = entry.label
            lbl.TextColor3 = WHITE
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 10
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            lbl.ZIndex = 10

            local pill, dot = mkPillVision(row, 10)
            local isOn = _G._RaptureMobileHideIncluded(entry.key)
            local function sv(state)
                animPillVision(pill, dot, state)
                _G._RaptureMobileHideList[entry.key] = state
                _G._RaptureApplyMobileButtonsHidden()
                task.defer(saveAllSettings)
            end
            local clk = Instance.new("TextButton", pill)
            clk.Size = UDim2.new(1, 0, 1, 0)
            clk.BackgroundTransparency = 1
            clk.Text = ""
            clk.AutoButtonColor = false
            clk.ZIndex = 11
            clk.MouseButton1Click:Connect(function()
                playToggleSound()
                isOn = not isOn
                sv(isOn)
            end)
            sv(isOn)

            hidePillRefs[entry.key] = { setVisual = sv }
        end

        _refreshHideListVisuals = function()
            for _, ref in pairs(hidePillRefs) do
                pcall(function()
                    local key = nil
                    for k, r in pairs(hidePillRefs) do
                        if r == ref then key = k break end
                    end
                    if key then
                        ref.setVisual(_G._RaptureMobileHideIncluded(key))
                    end
                end)
            end
        end

        task.defer(function()
            if innerScroll and innerScroll.Parent then
                innerScroll.CanvasSize = UDim2.new(0, 0, 0, innerLayout.AbsoluteContentSize.Y + 12)
            end
        end)

        local hideExpanded = false
        local hideContentHeight = #hideListData * 30 + (#hideListData - 1) * 6 + 12
        hdrArrow.MouseButton1Click:Connect(function()
            playArrowSound()
            hideExpanded = not hideExpanded
            if hideExpanded then
                holder.Visible = true
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, 0, 0, hideContentHeight) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), { Size = UDim2.new(1, -4, 0, 40 + hideContentHeight + 2) }):Play()
                TS:Create(hdrArrow, TweenInfo.new(0.2), { Rotation = 180 }):Play()
                task.defer(function()
                    if innerScroll and innerScroll.Parent then
                        innerScroll.CanvasSize = UDim2.new(0, 0, 0, innerLayout.AbsoluteContentSize.Y + 12)
                    end
                end)
            else
                TS:Create(holder, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, 0, 0, 0) }):Play()
                TS:Create(wrapper, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), { Size = UDim2.new(1, -4, 0, 40) }):Play()
                TS:Create(hdrArrow, TweenInfo.new(0.2), { Rotation = 0 }):Play()
                task.delay(0.22, function()
                    if not hideExpanded then holder.Visible = false end
                end)
            end
        end)
    end

    do
        local row = mkRow(configPage, 40)
        local delBtn = Instance.new("TextButton", row)
        delBtn.Size = UDim2.new(1, -8, 0.85, 0)
        delBtn.Position = UDim2.new(0, 4, 0.07, 0)
        delBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
        delBtn.BackgroundTransparency = 0.3
        delBtn.BorderSizePixel = 0
        delBtn.Text = "DELETE CONFIG"
        delBtn.TextColor3 = WHITE
        delBtn.Font = Enum.Font.GothamBold
        delBtn.TextSize = 12
        delBtn.ZIndex = 8
        Instance.new("UICorner", delBtn).CornerRadius = UDim.new(0, 10)
        local deleteState = 0
        local delDebounce = false
        delBtn.MouseButton1Click:Connect(function()
            if delDebounce then return end
            if deleteState == 0 then
                deleteState = 1
                delBtn.Text = "CONFIRM?"
                task.delay(2, function()
                    if delBtn and delBtn.Parent and deleteState == 1 then
                        deleteState = 0
                        delBtn.Text = "DELETE CONFIG"
                    end
                end)
            elseif deleteState == 1 then
                delDebounce = true
                pcall(resetToFactoryDefaults)
                task.defer(function()
                    if setAntiRagVisual then setAntiRagVisual(false) end
                    if setInstaGrab then setInstaGrab(false) end
                    if dropBrainrotSetVisual then dropBrainrotSetVisual(false) end
                    if setBatCounterVisual then setBatCounterVisual(false) end
                    if setMedusaVisual then setMedusaVisual(false) end
                    if setRagdollTimerVisual then setRagdollTimerVisual(false) end
                    if autoLeftSetVisual then autoLeftSetVisual(false) end
                    if autoRightSetVisual then autoRightSetVisual(false) end
                    if autoBatSetVisual then autoBatSetVisual(false) end
                    if batDesyncTpSetVisual then batDesyncTpSetVisual(false) end
                    if batBypassSetVisual then batBypassSetVisual(false) end
                    if platformSetVisual then platformSetVisual(false) end
                    if setESPVisual then setESPVisual(false) end
                    if setAntiLagVisual then setAntiLagVisual(false) end
                    if setFovVisual then setFovVisual(false) end
                    if setAntiDieVisual then setAntiDieVisual(false) end
                    if setAntiFlingVisual then setAntiFlingVisual(false) end
                    if bodyLockSetVisual then bodyLockSetVisual(false) end
                    if spinSetVisual then spinSetVisual(false) end
                    if spinAutoBrainrotSetVisual then spinAutoBrainrotSetVisual(false) end
                    if setUnwalkVisual then setUnwalkVisual(false) end
                    if setXrayBaseVisual then setXrayBaseVisual(false) end
                    if setXrayBrainrotVisual then setXrayBrainrotVisual(false) end
                    if _G.stretchToggleSetter then _G.stretchToggleSetter(false) end
                    if aimbotModeSetVisual then aimbotModeSetVisual("V1") end
                    if antiRagSetVisual then antiRagSetVisual("V1") end
                    if setCustomSoundsVisual then setCustomSoundsVisual(false) end
                    if setSoundButtonsVisual then setSoundButtonsVisual(true) end
                    if setSoundTabVisual then setSoundTabVisual(true) end
                    if setSoundToggleVisual then setSoundToggleVisual(true) end
                    if setSoundArrowVisual then setSoundArrowVisual(true) end
                    if setSyncAfterHitVisual then setSyncAfterHitVisual(false) end
                    if speedDisplayRefresh then speedDisplayRefresh() end
                    if setShowSpeedVisual then setShowSpeedVisual(true) end
                    if hideMobileMasterSetVisual then hideMobileMasterSetVisual(false) end
                    if infiniteJumpSetVisual then infiniteJumpSetVisual(true) end
                    if skipIntroToggleSet then skipIntroToggleSet(false) end
                    if platformSecondsBox then platformSecondsBox.Text = "1.9" end
                    refreshSpeedModeLabel()
                end)
                delBtn.Text = "DELETED"
                task.delay(1.5, function() if delBtn and delBtn.Parent then delBtn.Text = "DELETE CONFIG"; delDebounce = false; deleteState = 0 end end)
            end
        end)
    end
    local keyPage = contentPages["Keybinds"]
    addKeybindRow(keyPage, "Carry Mode", KB.CarryToggle)
    addKeybindRow(keyPage, "Lagger Mode", KB.LaggerMode)
    addKeybindRow(keyPage, "Auto Left", KB.AutoLeft)
    addKeybindRow(keyPage, "Auto Right", KB.AutoRight)
    addKeybindRow(keyPage, "Auto Bat", KB.AutoBat)
    addKeybindRow(keyPage, "TP BAT", KB.TPBat)
    addKeybindRow(keyPage, "Insta Reset", KB.InstaReset)
    addKeybindRow(keyPage, "Bat Bypass", KB.BatBypass)
    addKeybindRow(keyPage, "Spin", KB.Spin)
    addKeybindRow(keyPage, "Platform", KB.Platform)
    addKeybindRow(keyPage, "TP Down", KB.TPFloor)
    addKeybindRow(keyPage, "Drop Brainrot", KB.DropBrainrot)
    addKeybindRow(keyPage, "Hide GUI", KB.GuiHide)
    pbFrame = Instance.new("Frame", gui)
    pbFrame.Size = UDim2.new(0, 480, 0, 76)
    pbFrame.Position = UDim2.new(0.5, -240, 1, -90)
    pbFrame.BackgroundTransparency = 1
    pbFrame.BorderSizePixel = 0
    pbFrame.Active = true
    pbFrame.ClipsDescendants = true
    pbFrame.Visible = true
    pbFrame.ZIndex = 10
    pbScale = Instance.new("UIScale", pbFrame)
    pbScale.Scale = progressBarScaleValue / 100
    if savedProgressBarPos then
        pbFrame.Position = UDim2.new(savedProgressBarPos.XScale or 0.5, savedProgressBarPos.XOffset or -240, savedProgressBarPos.YScale or 1, savedProgressBarPos.YOffset or -90)
    end
    Instance.new("UICorner", pbFrame).CornerRadius = UDim.new(0, 16)
    local pbBgImage = Instance.new("ImageLabel", pbFrame)
    pbBgImage.Name = "StealBarBg"
    pbBgImage.Size = UDim2.new(1, 0, 1, 0)
    pbBgImage.Position = UDim2.new(0, 0, 0, 0)
    pbBgImage.BackgroundTransparency = 1
    pbBgImage.Image = "rbxassetid://73439681360959"
    pbBgImage.ScaleType = Enum.ScaleType.Crop
    pbBgImage.ZIndex = 1
    Instance.new("UICorner", pbBgImage).CornerRadius = UDim.new(0, 16)
    local border = Instance.new("UIStroke", pbFrame)
    border.Color = BLACK_STROKE
    border.Thickness = BTN_STROKE_THICKNESS
    border.Transparency = 0.3
    local pctRow = Instance.new("Frame", pbFrame)
    pctRow.Size = UDim2.new(1, 0, 0, 26)
    pctRow.Position = UDim2.new(0, 0, 0, 4)
    pctRow.BackgroundTransparency = 1
    pctRow.ZIndex = 12
    progressPct = Instance.new("TextLabel", pctRow)
    progressPct.Size = UDim2.new(1, -28, 1, 0)
    progressPct.Position = UDim2.new(0, 14, 0, 0)
    progressPct.BackgroundTransparency = 1
    progressPct.Text = "0%"
    progressPct.TextColor3 = Color3.fromRGB(255, 255, 255)
    progressPct.Font = Enum.Font.GothamBlack
    progressPct.TextSize = 22
    progressPct.TextXAlignment = Enum.TextXAlignment.Right
    progressPct.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    progressPct.TextStrokeTransparency = 0.2
    progressPct.ZIndex = 13
    local infoRow = Instance.new("Frame", pbFrame)
    infoRow.Size = UDim2.new(1, 0, 0, 26)
    infoRow.Position = UDim2.new(0, 0, 0, 24)
    infoRow.BackgroundTransparency = 1
    infoRow.ZIndex = 12
    fpsLabel = Instance.new("TextLabel", infoRow)
    fpsLabel.Size = UDim2.new(0, 130, 1, 0)
    fpsLabel.Position = UDim2.new(0, 14, 0, 0)
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.Text = "FPS: --"
    fpsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    fpsLabel.Font = Enum.Font.GothamBlack
    fpsLabel.TextSize = 22
    fpsLabel.TextXAlignment = Enum.TextXAlignment.Left
    fpsLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    fpsLabel.TextStrokeTransparency = 0.2
    fpsLabel.ZIndex = 13
    msLabel = Instance.new("TextLabel", infoRow)
    msLabel.Size = UDim2.new(0, 130, 1, 0)
    msLabel.Position = UDim2.new(0, 14 + 130 + 12, 0, 0)
    msLabel.BackgroundTransparency = 1
    msLabel.Text = "MS: --"
    msLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    msLabel.Font = Enum.Font.GothamBlack
    msLabel.TextSize = 22
    msLabel.TextXAlignment = Enum.TextXAlignment.Left
    msLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    msLabel.TextStrokeTransparency = 0.2
    msLabel.ZIndex = 13
    local progressRow = Instance.new("Frame", pbFrame)
    progressRow.Size = UDim2.new(1, -20, 0, 14)
    progressRow.Position = UDim2.new(0, 10, 0, 56)
    progressRow.BackgroundTransparency = 1
    progressRow.ZIndex = 11
    local fillRegion = Instance.new("Frame", progressRow)
    fillRegion.Size = UDim2.new(1, 0, 1, 0)
    fillRegion.BackgroundColor3 = GRAY_TRACK
    fillRegion.BackgroundTransparency = 0
    fillRegion.BorderSizePixel = 0
    fillRegion.ClipsDescendants = true
    fillRegion.ZIndex = 12
    Instance.new("UICorner", fillRegion).CornerRadius = UDim.new(0, 10)
    local frStroke = Instance.new("UIStroke", fillRegion)
    frStroke.Color = Color3.fromRGB(0, 0, 0)
    frStroke.Thickness = BTN_STROKE_THICKNESS
    frStroke.Transparency = 0
    frStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    progressFill = Instance.new("Frame", fillRegion)
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.Position = UDim2.new(0, 0, 0, 0)
    progressFill.BackgroundColor3 = NEON_PINK
    progressFill.BorderSizePixel = 0
    progressFill.ZIndex = 13
    Instance.new("UICorner", progressFill).CornerRadius = UDim.new(0, 10)
    local glowGrad = Instance.new("UIGradient", progressFill)
    glowGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(190, 120, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(220, 180, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(190, 120, 255)),
    })
    glowGrad.Rotation = 90
    drag(pbFrame)
    task.spawn(function()
        local lastFrame = _tick()
        local fpsSamples = {}
        local fpsAvg = 60
        RunService.RenderStepped:Connect(function()
            local now = _tick()
            local dt = now - lastFrame
            lastFrame = now
            if dt > 0 then
                table.insert(fpsSamples, 1 / dt)
                if #fpsSamples > 30 then table.remove(fpsSamples, 1) end
                local sum = 0
                for _, v in ipairs(fpsSamples) do sum = sum + v end
                fpsAvg = sum / #fpsSamples
            end
        end)
        while true do
            local ping = 0
            pcall(function() ping = LP:GetNetworkPing() * 1000 end)
            if fpsLabel then fpsLabel.Text = string.format("FPS: %d", _floor(fpsAvg + 0.5)) end
            if msLabel then msLabel.Text = string.format("MS: %d", _floor(ping + 0.5)) end
            task.wait(0.5)
        end
    end)
    drag(main)
end

function openImagePicker()
    pcall(function()
        local old = CoreGui:FindFirstChild("RaptureImagePicker")
        if old then old:Destroy() end
        local pg = LP:FindFirstChild("PlayerGui")
        if pg then local o = pg:FindFirstChild("RaptureImagePicker"); if o then o:Destroy() end end
    end)
    local viewer = Instance.new("ScreenGui")
    viewer.Name = "RaptureImagePicker"
    viewer.ResetOnSpawn = false
    viewer.IgnoreGuiInset = true
    viewer.DisplayOrder = 400
    if not pcall(function() viewer.Parent = CoreGui end) then viewer.Parent = LP:WaitForChild("PlayerGui") end
    local dim = Instance.new("TextButton", viewer)
    dim.Size = UDim2.new(1, 0, 1, 0)
    dim.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    dim.BackgroundTransparency = 0.45
    dim.Text = ""
    dim.AutoButtonColor = false
    dim.ZIndex = 1
    dim.Activated:Connect(function() viewer:Destroy() end)
    local frame = Instance.new("Frame", viewer)
    frame.Size = UDim2.new(0, 420, 0, 380)
    frame.Position = UDim2.new(0.5, -210, 0.5, -190)
    frame.BackgroundColor3 = Color3.fromRGB(12, 10, 16)
    frame.BorderSizePixel = 0
    frame.ZIndex = 2
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 14)
    local st = Instance.new("UIStroke", frame)
    st.Color = Color3.fromRGB(60,50,75)
    st.Thickness = 1.8
    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, -48, 0, 30)
    title.Position = UDim2.new(0, 12, 0, 8)
    title.BackgroundTransparency = 1
    title.Text = "BACKGROUND IMAGE"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 14
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.ZIndex = 3
    local close = Instance.new("TextButton", frame)
    close.Size = UDim2.new(0, 28, 0, 28)
    close.Position = UDim2.new(1, -34, 0, 8)
    close.BackgroundColor3 = Color3.fromRGB(28,26,36)
    close.BorderSizePixel = 0
    close.Text = "X"
    close.TextColor3 = Color3.fromRGB(255,255,255)
    close.Font = Enum.Font.GothamBlack
    close.TextSize = 14
    close.ZIndex = 4
    Instance.new("UICorner", close).CornerRadius = UDim.new(0, 8)
    close.Activated:Connect(function() viewer:Destroy() end)
    local scroll = Instance.new("ScrollingFrame", frame)
    scroll.Size = UDim2.new(1, -16, 1, -52)
    scroll.Position = UDim2.new(0, 8, 0, 44)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 5
    scroll.ScrollBarImageColor3 = getThemeColor()
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.ZIndex = 3
    local grid = Instance.new("UIGridLayout", scroll)
    grid.CellSize = UDim2.new(0, 120, 0, 80)
    grid.CellPadding = UDim2.new(0, 8, 0, 8)
    grid.SortOrder = Enum.SortOrder.LayoutOrder
    for i, id in ipairs(backgroundImages) do
        local thumb = Instance.new("TextButton", scroll)
        thumb.Size = UDim2.new(0, 120, 0, 80)
        thumb.BackgroundColor3 = Color3.fromRGB(20, 18, 26)
        thumb.BorderSizePixel = 0
        thumb.Text = ""
        thumb.LayoutOrder = i
        thumb.ZIndex = 4
        Instance.new("UICorner", thumb).CornerRadius = UDim.new(0, 8)
        local thumbSt = Instance.new("UIStroke", thumb)
        thumbSt.Color = (backgroundIndex == i) and getThemeColor() or Color3.fromRGB(60, 50, 75)
        thumbSt.Thickness = (backgroundIndex == i) and 2 or 1
        thumbSt.Transparency = (backgroundIndex == i) and 0 or 0.4
        local img = Instance.new("ImageLabel", thumb)
        img.Size = UDim2.new(1, -8, 1, -8)
        img.Position = UDim2.new(0, 4, 0, 4)
        img.BackgroundTransparency = 1
        img.Image = "rbxassetid://" .. id
        img.ScaleType = Enum.ScaleType.Crop
        img.ZIndex = 5
        Instance.new("UICorner", img).CornerRadius = UDim.new(0, 6)
        thumb.Activated:Connect(function()
            viewer:Destroy()
            task.defer(function()
                pcall(function()
                    backgroundIndex = i
                    saveAllSettings()
                    local bgImg = main and main:FindFirstChild("BackgroundImage")
                    if bgImg then
                        bgImg.Image = "rbxassetid://" .. id
                        bgImg.ImageTransparency = backgroundImageTransparency
                    end
                end)
            end)
        end)
    end
    task.defer(function()
        if scroll and scroll.Parent then
            scroll.CanvasSize = UDim2.new(0, 0, 0, grid.AbsoluteContentSize.Y + 16)
        end
    end)
end

function createMobilePanel()
    local panel = Instance.new("ScreenGui")
    panel.Name = "RaptureDuelMobilePanel"
    panel.ResetOnSpawn = false
    panel.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    if not pcall(function() panel.Parent = CoreGui end) then panel.Parent = LP:WaitForChild("PlayerGui") end
    local BTN_W, BTN_H = 78, 50
    local GAP = 8
    local COLUMNS = 2
    local ROWS = 4
    local PANEL_W = BTN_W * COLUMNS + GAP * (COLUMNS - 1)
    local PANEL_H = BTN_H * ROWS + (GAP + 10) * (ROWS - 1)
    local container = Instance.new("Frame", panel)
    container.Name = "FloatingPanel"
    container.Size = UDim2.new(0, PANEL_W, 0, PANEL_H)
    container.Position = UDim2.new(1, -PANEL_W - 15, 0, 70)
    container.BackgroundTransparency = 1
    container.BorderSizePixel = 0
    container.Active = false
    container.ClipsDescendants = false
    local containerScale = Instance.new("UIScale", container)
    containerScale.Scale = floatingButtonScale
    table.insert(_floatingUIScales, containerScale)
    local btnContainer = Instance.new("Frame", container)
    btnContainer.Name = "ButtonsContainer"
    btnContainer.Size = UDim2.new(1, 0, 1, 0)
    btnContainer.BackgroundTransparency = 1
    local buttons = {}
    local buttonNames = {"DropBR", "AutoLeft", "AutoBat", "AutoRight", "TpDown", "Carry", "Lagger1", "Lagger2"}
    local buttonTexts = {"DROP\nBR", "AUTO\nLEFT", "BAT\nAIMBOT", "AUTO\nRIGHT", "TP\nDOWN", "CARRY\nSPD", "LAGGER\nCARRY", "LAGGER"}
    for i, name in ipairs(buttonNames) do
        local text = buttonTexts[i]
        local btn = Instance.new("TextButton", btnContainer)
        btn.Name = name
        btn.Size = UDim2.new(0, BTN_W, 0, BTN_H)
        btn.BackgroundColor3 = BTN_BG_DARK
        btn.BorderSizePixel = 0
        btn.Text = ""
        btn.AutoButtonColor = false
        btn.ZIndex = 10
        btn.ClipsDescendants = true
        local savedPos = savedButtonPositions[name]
        if savedPos then
            btn.Position = UDim2.new(0, savedPos.X or 0, 0, savedPos.Y or 0)
        else
            local defX, defY = getDefaultButtonPosition(name)
            btn.Position = UDim2.new(0, defX, 0, defY)
        end
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 14)
        local label = Instance.new("TextLabel", btn)
        label.Name = "TextLabel"
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = text
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.Font = Enum.Font.GothamBlack
        label.TextSize = 10
        label.TextWrapped = true
        label.ZIndex = 11
        applyNeonPinkStyle(btn, false)
        local active = false
        local function setActive(state)
            active = state
            btn:SetAttribute("MobActive", state and true or false)
            paintFloatingBtn(btn, state)
            if state then
                if _G._RapturePlaySweep then _G._RapturePlaySweep(btn) end
            else
                btn:SetAttribute("_shineActive", false)
                local s = btn:FindFirstChild("AnimatedShine")
                if s then
                    s.Visible = false
                    s.Position = UDim2.new(0, 0, 0.5, 0)
                    s.BackgroundTransparency = 1
                end
            end
        end
        setActive(false)
        local dragging = false
        local hasMoved = false
        local dragStart = nil
        local startPos = nil
        local movedDistance = 0
        btn.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true; hasMoved = false; movedDistance = 0
                dragStart = input.Position; startPos = btn.Position
                _isDraggingButton = true
            end
        end)
        btn.InputChanged:Connect(function(input)
            if not dragging then return end
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                local delta = input.Position - dragStart
                movedDistance = delta.Magnitude
                if not uiLocked then
                    hasMoved = true
                    btn.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
                end
            end
        end)
        btn.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                if dragging then
                    if movedDistance < 3 then
                        playFloatingButtonSound()
                        if name == "DropBR" then
                            if not autoBatEnabled then
                                setActive(true)
                                executeDropWithToggle(function(v) if dropBrainrotSetVisual then dropBrainrotSetVisual(v) end end)
                                task.delay(0.3, function() setActive(false) end)
                            end
                        elseif name == "AutoLeft" then
                            autoLeftEnabled = not autoLeftEnabled
                            setActive(autoLeftEnabled)
                            if autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
                            if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
                        elseif name == "AutoBat" then
                            if not autoBatEnabled then enableAutoBat() else disableAutoBat() end
                            setActive(autoBatEnabled)
                        elseif name == "AutoRight" then
                            autoRightEnabled = not autoRightEnabled
                            setActive(autoRightEnabled)
                            if autoRightEnabled then startAutoRight() else stopAutoRight() end
                            if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
                        elseif name == "TpDown" then
                            doTpDown()
                            setActive(true)
                            task.delay(0.2, function() setActive(false) end)
                        elseif name == "Carry" then
                            if not speedMode then
                                speedMode = true; laggerToggled = false; laggerCarryToggled = false
                                setActive(true)
                                if buttons.Lagger1 and buttons.Lagger1.setActive then buttons.Lagger1.setActive(false) end
                                if buttons.Lagger2 and buttons.Lagger2.setActive then buttons.Lagger2.setActive(false) end
                            else speedMode = false; setActive(false) end
                            refreshSpeedModeLabel()
                        elseif name == "Lagger1" then
                            if not laggerToggled then
                                laggerToggled = true; speedMode = false; laggerCarryToggled = false
                                setActive(true)
                                if buttons.Carry and buttons.Carry.setActive then buttons.Carry.setActive(false) end
                                if buttons.Lagger2 and buttons.Lagger2.setActive then buttons.Lagger2.setActive(false) end
                            else laggerToggled = false; setActive(false) end
                            refreshSpeedModeLabel()
                        elseif name == "Lagger2" then
                            if not laggerCarryToggled then
                                laggerCarryToggled = true; speedMode = false; laggerToggled = false
                                setActive(true)
                                if buttons.Carry and buttons.Carry.setActive then buttons.Carry.setActive(false) end
                                if buttons.Lagger1 and buttons.Lagger1.setActive then buttons.Lagger1.setActive(false) end
                            else laggerCarryToggled = false; setActive(false) end
                            refreshSpeedModeLabel()
                        end
                    elseif not uiLocked and hasMoved then
                        savedButtonPositions[name] = { X = btn.Position.X.Offset, Y = btn.Position.Y.Offset }
                        task.defer(saveAllSettings)
                    end
                    dragging = false; hasMoved = false
                    dragStart = nil; startPos = nil; movedDistance = 0
                    _isDraggingButton = false
                end
            end
        end)
        buttons[name] = {btn = btn, setActive = setActive, label = label}
        if name == "AutoBat" then mobSetAutoBat = setActive end
        if name == "AutoLeft" then mobSetAutoLeft = setActive end
        if name == "AutoRight" then mobSetAutoRight = setActive end
        if name == "DropBR" then mobSetDropBR = setActive end
        if name == "TpDown" then mobSetTpDown = setActive end
        if name == "Carry" then mobSetCarry = setActive end
        if name == "Lagger1" then mobSetLagger1 = setActive end
        if name == "Lagger2" then mobSetLagger2 = setActive end
    end
    return panel
end

function createTpBatFloatingButton()
    local panel = Instance.new("ScreenGui")
    panel.Name = "TpBatButton"
    panel.ResetOnSpawn = false
    panel.DisplayOrder = 21
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    if not pcall(function() panel.Parent = CoreGui end) then panel.Parent = LP:WaitForChild("PlayerGui") end
    local btnFrame = Instance.new("Frame", panel)
    btnFrame.Size = UDim2.new(0, 78, 0, 50)
    btnFrame.Name = "Frame"
    btnFrame.Position = tpBatFloatingPos and UDim2.new(tpBatFloatingPos.XScale or 0.5, tpBatFloatingPos.XOffset or 20, tpBatFloatingPos.YScale or 0, tpBatFloatingPos.YOffset or 10) or UDim2.new(0.5, 20, 0, 10)
    btnFrame.BackgroundColor3 = BTN_BG_DARK
    btnFrame.BorderSizePixel = 0
    btnFrame.ZIndex = 20
    btnFrame.ClipsDescendants = true
    Instance.new("UICorner", btnFrame).CornerRadius = UDim.new(0, 14)
    local label = Instance.new("TextLabel", btnFrame)
    label.Name = "TextLabel"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "TP\nBAT"
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBlack
    label.TextSize = 10
    label.TextWrapped = true
    label.ZIndex = 21
    local uiScale = Instance.new("UIScale", btnFrame)
    uiScale.Scale = floatingButtonScale
    table.insert(_floatingUIScales, uiScale)
    applyNeonPinkStyle(btnFrame, batDesyncTpEnabled)
    local dragging, hasMoved, dragStart, startPos
    btnFrame.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true; hasMoved = false; dragStart = inp.Position; startPos = btnFrame.Position
        end
    end)
    btnFrame.InputChanged:Connect(function(inp)
        if not dragging then return end
        if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
            local delta = inp.Position - dragStart
            if delta.Magnitude > 5 then hasMoved = true end
            if hasMoved and not uiLocked then
                btnFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
    btnFrame.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                if not hasMoved then
                    playFloatingButtonSound()
                    toggleBatDesyncTp()
                    applyNeonPinkStyle(btnFrame, batDesyncTpEnabled)
                    if batDesyncTpEnabled and _G._RapturePlaySweep then _G._RapturePlaySweep(btnFrame) end
                elseif not uiLocked and hasMoved then
                    tpBatFloatingPos = { XScale = btnFrame.Position.X.Scale, XOffset = btnFrame.Position.X.Offset, YScale = btnFrame.Position.Y.Scale, YOffset = btnFrame.Position.Y.Offset }
                    task.defer(saveAllSettings)
                end
                dragging = false; hasMoved = false
            end
        end
    end)
    tpBatFloatingButton = panel
    return panel
end

function createBatBypassFloatingButton()
    local panel = Instance.new("ScreenGui")
    panel.Name = "BatBypassButton"
    panel.ResetOnSpawn = false
    panel.DisplayOrder = 22
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    if not pcall(function() panel.Parent = CoreGui end) then panel.Parent = LP:WaitForChild("PlayerGui") end
    local btnFrame = Instance.new("Frame", panel)
    btnFrame.Size = UDim2.new(0, 78, 0, 50)
    btnFrame.Name = "Frame"
    btnFrame.Position = batBypassFloatingPos and UDim2.new(batBypassFloatingPos.XScale or 0.5, batBypassFloatingPos.XOffset or 60, batBypassFloatingPos.YScale or 0, batBypassFloatingPos.YOffset or 10) or UDim2.new(0.5, 60, 0, 10)
    btnFrame.BackgroundColor3 = BTN_BG_DARK
    btnFrame.BorderSizePixel = 0
    btnFrame.ZIndex = 20
    btnFrame.ClipsDescendants = true
    Instance.new("UICorner", btnFrame).CornerRadius = UDim.new(0, 14)
    local label = Instance.new("TextLabel", btnFrame)
    label.Name = "TextLabel"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "BAT\nBYPASS"
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBlack
    label.TextSize = 10
    label.TextWrapped = true
    label.ZIndex = 21
    local uiScale = Instance.new("UIScale", btnFrame)
    uiScale.Scale = floatingButtonScale
    table.insert(_floatingUIScales, uiScale)
    applyNeonPinkStyle(btnFrame, _G._RaptureBatBypass and _G._RaptureBatBypass.enabled or false)
    local dragging, hasMoved, dragStart, startPos
    btnFrame.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true; hasMoved = false; dragStart = inp.Position; startPos = btnFrame.Position
        end
    end)
    btnFrame.InputChanged:Connect(function(inp)
        if not dragging then return end
        if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
            local delta = inp.Position - dragStart
            if delta.Magnitude > 5 then hasMoved = true end
            if hasMoved and not uiLocked then
                btnFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
    btnFrame.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                if not hasMoved then
                    playFloatingButtonSound()
                    _G.RaptureToggleBatBypass()
                    applyNeonPinkStyle(btnFrame, _G._RaptureBatBypass and _G._RaptureBatBypass.enabled or false)
                    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled and _G._RapturePlaySweep then _G._RapturePlaySweep(btnFrame) end
                elseif not uiLocked and hasMoved then
                    batBypassFloatingPos = { XScale = btnFrame.Position.X.Scale, XOffset = btnFrame.Position.X.Offset, YScale = btnFrame.Position.Y.Scale, YOffset = btnFrame.Position.Y.Offset }
                    task.defer(saveAllSettings)
                end
                dragging = false; hasMoved = false
            end
        end
    end)
    batBypassFloatingButton = panel
    return panel
end

function createInstaResetFloatingButton()
    local panel = Instance.new("ScreenGui")
    panel.Name = "InstaResetButton"
    panel.ResetOnSpawn = false
    panel.DisplayOrder = 23
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    if not pcall(function() panel.Parent = CoreGui end) then panel.Parent = LP:WaitForChild("PlayerGui") end
    local btnFrame = Instance.new("Frame", panel)
    btnFrame.Size = UDim2.new(0, 78, 0, 50)
    btnFrame.Name = "Frame"
    btnFrame.Position = instaResetFloatingPos and UDim2.new(instaResetFloatingPos.XScale or 0.5, instaResetFloatingPos.XOffset or 100, instaResetFloatingPos.YScale or 0, instaResetFloatingPos.YOffset or 10) or UDim2.new(0.5, 100, 0, 10)
    btnFrame.BackgroundColor3 = BTN_BG_DARK
    btnFrame.BorderSizePixel = 0
    btnFrame.ZIndex = 20
    btnFrame.ClipsDescendants = true
    Instance.new("UICorner", btnFrame).CornerRadius = UDim.new(0, 14)
    local label = Instance.new("TextLabel", btnFrame)
    label.Name = "TextLabel"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "INSTA\nRESET"
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBlack
    label.TextSize = 10
    label.TextWrapped = true
    label.ZIndex = 21
    local uiScale = Instance.new("UIScale", btnFrame)
    uiScale.Scale = floatingButtonScale
    table.insert(_floatingUIScales, uiScale)
    applyNeonPinkStyle(btnFrame, false)
    local dragging, hasMoved, dragStart, startPos, activeInput
    btnFrame.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            activeInput = inp
            dragging = true; hasMoved = false; dragStart = inp.Position; startPos = btnFrame.Position
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if not dragging or not activeInput then return end
        local isTouchMove = activeInput.UserInputType == Enum.UserInputType.Touch and inp == activeInput
        local isMouseMove = activeInput.UserInputType == Enum.UserInputType.MouseButton1 and inp.UserInputType == Enum.UserInputType.MouseMovement
        if not isTouchMove and not isMouseMove then return end
        local delta = inp.Position - dragStart
        if delta.Magnitude > 12 then hasMoved = true end
        if hasMoved and not uiLocked then
            btnFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UIS.InputEnded:Connect(function(inp)
        if inp ~= activeInput then return end
        if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
        if dragging then
            if not hasMoved then
                playFloatingButtonSound()
                if _G.InstaReset and _G.InstaReset.Trigger then _G.InstaReset.Trigger() end
            elseif not uiLocked and hasMoved then
                instaResetFloatingPos = { XScale = btnFrame.Position.X.Scale, XOffset = btnFrame.Position.X.Offset, YScale = btnFrame.Position.Y.Scale, YOffset = btnFrame.Position.Y.Offset }
                task.defer(saveAllSettings)
            end
            dragging = false; hasMoved = false; activeInput = nil
        end
    end)
    instaResetFloatingButton = panel
    return panel
end

function createPlatformFloatingButton()
    local panel = Instance.new("ScreenGui")
    panel.Name = "PlatformButton"
    panel.ResetOnSpawn = false
    panel.DisplayOrder = 24
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(panel) end end)
    if not pcall(function() panel.Parent = CoreGui end) then panel.Parent = LP:WaitForChild("PlayerGui") end
    local btnFrame = Instance.new("Frame", panel)
    btnFrame.Size = UDim2.new(0, 78, 0, 50)
    btnFrame.Name = "Frame"
    btnFrame.Position = platformFloatingPos
        and UDim2.new(platformFloatingPos.XScale or 0.5, platformFloatingPos.XOffset or 140, platformFloatingPos.YScale or 0, platformFloatingPos.YOffset or 10)
        or UDim2.new(0.5, 140, 0, 10)
    btnFrame.BackgroundColor3 = BTN_BG_DARK
    btnFrame.BorderSizePixel = 0
    btnFrame.ZIndex = 20
    btnFrame.ClipsDescendants = true
    Instance.new("UICorner", btnFrame).CornerRadius = UDim.new(0, 14)
    local label = Instance.new("TextLabel", btnFrame)
    label.Name = "TextLabel"
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "PLATFORM"
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBlack
    label.TextSize = 10
    label.TextWrapped = true
    label.ZIndex = 21
    local uiScale = Instance.new("UIScale", btnFrame)
    uiScale.Scale = floatingButtonScale
    table.insert(_floatingUIScales, uiScale)
    applyNeonPinkStyle(btnFrame, _Platform.enabled)
    local dragging, hasMoved, dragStart, startPos
    btnFrame.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true; hasMoved = false; dragStart = inp.Position; startPos = btnFrame.Position
        end
    end)
    btnFrame.InputChanged:Connect(function(inp)
        if not dragging then return end
        if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
            local delta = inp.Position - dragStart
            if delta.Magnitude > 5 then hasMoved = true end
            if hasMoved and not uiLocked then
                btnFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
    btnFrame.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                if not hasMoved then
                    playFloatingButtonSound()
                    platformToggle()
                elseif not uiLocked and hasMoved then
                    platformFloatingPos = { XScale = btnFrame.Position.X.Scale, XOffset = btnFrame.Position.X.Offset, YScale = btnFrame.Position.Y.Scale, YOffset = btnFrame.Position.Y.Offset }
                    task.defer(saveAllSettings)
                end
                dragging = false; hasMoved = false
            end
        end
    end)
    platformFloatingButton = panel
    return panel
end

function updateUIFromLoaded()
    task.wait()
    if normalBox then normalBox.Text = tostring(NS) end
    if carryBox then carryBox.Text = tostring(CS) end
    if radInput then radInput.Text = tostring(CONFIG.STEAL_RANGE) end
    if laggerBox then laggerBox.Text = tostring(LAGGER_SPEED) end
    if lagger2Box then lagger2Box.Text = tostring(LAGGER_CARRY_SPEED) end
    if batSpeedBox then batSpeedBox.Text = tostring(BAT_AIMBOT_SPEED) end
    if uiScaleBox then uiScaleBox.Text = tostring(uiScaleValue) end
    if progressBarScaleBox then progressBarScaleBox.Text = tostring(progressBarScaleValue) end
    if stretchValueBox then stretchValueBox.Text = tostring(stretchValue) end
    if spinSpeedBox then spinSpeedBox.Text = tostring(spinSpeed) end
    if platformSecondsBox then platformSecondsBox.Text = string.format("%.1f", _Platform.AUTO_DISABLE_TIME) end
    refreshSpeedModeLabel()
    for _, ref in ipairs(keyButtonRefs) do
        local entry = ref.entry
        local label = (entry.gp and entry.gp.Name) or (entry.kb and entry.kb.Name) or "None"
        ref.btn.Text = label
    end
    if savedProgressBarPos and pbFrame then
        pbFrame.Position = UDim2.new(savedProgressBarPos.XScale or 0.5, savedProgressBarPos.XOffset or -240, savedProgressBarPos.YScale or 1, savedProgressBarPos.YOffset or -90)
    end
    local bgImg = main and main:FindFirstChild("BackgroundImage")
    if bgImg then
        bgImg.Image = "rbxassetid://" .. backgroundImages[backgroundIndex]
        bgImg.ImageTransparency = backgroundImageTransparency
    end
    applyFloatingButtonScale()
    if uiLocked and setLockUIVisual then setLockUIVisual(true) end
    if antiRagdollEnabled then
        if setAntiRagVisual then setAntiRagVisual(true) end
        if antiRagdollVersion == "v1" then AntiRagdollV1.start()
        elseif antiRagdollVersion == "v2" then startAntiRagdollV2() end
    end
    if antiDieEnabled then
        if setAntiDieVisual then setAntiDieVisual(true) end
        _antiDie.StartEngine()
    end
    if infiniteJumpSetVisual then
        infiniteJumpSetVisual(InfiniteJump and InfiniteJump.enabled == true)
    end
    if skipIntroToggleSet then skipIntroToggleSet(skipIntroEnabled) end
    if antiFlingEnabled then
        if setAntiFlingVisual then setAntiFlingVisual(true) end
        startAntiFling()
    end
    if bodyLockEnabled and bodyLockSetVisual then
        bodyLockSetVisual(true)
        startBodyLock()
    end
    if bodyLockRangeBox then bodyLockRangeBox.Text = tostring(bodyLockRange) end
    if spinEnabled and spinSetVisual then
        spinSetVisual(true)
        if _G._RaptureSpinSet then _G._RaptureSpinSet(true) end
    end
    if spinAutoBrainrotEnabled and spinAutoBrainrotSetVisual then
        spinAutoBrainrotSetVisual(true)
        if _G._RaptureSpinAutoBrainrotSet then _G._RaptureSpinAutoBrainrotSet(true) end
    end
    if spinSpeedBox then spinSpeedBox.Text = tostring(spinSpeed) end
    if unwalkEnabled and setUnwalkVisual then
        setUnwalkVisual(true)
        startUnwalk()
    end
    if CONFIG.AUTO_STEAL_ENABLED and setInstaGrab then setInstaGrab(true); pcall(startAutoSteal) end
    if medusaCounterEnabled and setMedusaVisual then setMedusaVisual(true); if LP.Character then setupMedusaCounter(LP.Character) end end
    if batCounterEnabled and setBatCounterVisual then setBatCounterVisual(true); startBatCounter() end
    if ragdollCountdownEnabled and setRagdollTimerVisual then setRagdollTimerVisual(true); hookRagdollCountdown(LP.Character) end
    if espEnabled and setESPVisual then setESPVisual(true); startESP() end
    if antiLagEnabled and setAntiLagVisual then setAntiLagVisual(true); enableAntiLag() end
    if stretchEnabled and _G.stretchToggleSetter then _G.stretchToggleSetter(true) end
    if setFovVisual then setFovVisual(fovEnabled) end
    if fovSliderSet then fovSliderSet(fovValue) end
    if batDesyncTpEnabled then
        if batDesyncTpSetVisual then batDesyncTpSetVisual(true) end
        if not batDesyncTpConn then startBatDesyncTp() end
    end
    if batBypassEnabled then
        if batBypassSetVisual then batBypassSetVisual(true) end
        if not _G._RaptureBatBypass.enabled then _G.RaptureStartBatBypass() end
    end
    if _G.RapturePlatformIsEnabled and _G.RapturePlatformIsEnabled() then
        if platformSetVisual then platformSetVisual(true) end
        if platformFloatingButton then
            local f = platformFloatingButton:FindFirstChild("Frame")
            if f then applyNeonPinkStyle(f, true) end
        end
    end
    if aimbotModeSetVisual then aimbotModeSetVisual(aimbotMode) end
    if antiRagSetVisual then antiRagSetVisual(antiRagdollVersion == "v1" and "V1" or "V2") end
    if customSoundsEnabled and setCustomSoundsVisual then
        setCustomSoundsVisual(true)
        if _G.RaptureCustomSounds then pcall(function() _G.RaptureCustomSounds.setEnabled(true) end) end
    end
    if setSoundButtonsVisual then setSoundButtonsVisual(soundButtonsEnabled) end
    if setSoundTabVisual then setSoundTabVisual(soundTabEnabled) end
    if setSoundToggleVisual then setSoundToggleVisual(soundToggleEnabled) end
    if setSoundArrowVisual then setSoundArrowVisual(soundArrowEnabled) end
    if _refreshSoundListVisuals then _refreshSoundListVisuals() end
    if syncAfterHitEnabled and setSyncAfterHitVisual then
        setSyncAfterHitVisual(true)
        if LP.Character then _syncStartSystems(LP.Character) end
    end
    if xrayBaseEnabled and setXrayBaseVisual then
        setXrayBaseVisual(true)
        if _G._XrayBaseSet then _G._XrayBaseSet(true) end
    end
    if xrayBrainrotEnabled and setXrayBrainrotVisual then
        setXrayBrainrotVisual(true)
        if _G._XrayBrainrotSet then _G._XrayBrainrotSet(true) end
    end
    if setShowSpeedVisual then setShowSpeedVisual(showSpeedLabel) end
    if speedDisplayRefresh then speedDisplayRefresh() end
    if _G._RaptureSpeedDisplayRefresh then _G._RaptureSpeedDisplayRefresh() end
    if tpDownRefresh then pcall(tpDownRefresh) end
    if speedLabel then
        speedLabel.Visible = showSpeedLabel
        if showSpeedLabel then
            updateSpeedDisplayText()
        else
            speedLabel.Text = ""
        end
    end
    if _G._RaptureApplyMobileButtonsHidden then _G._RaptureApplyMobileButtonsHidden() end
    if _refreshHideListVisuals then _refreshHideListVisuals() end
    if skyTheme ~= "Off" then pcall(applyCustomSky, skyTheme) end
    if skySelectorLabel then skySelectorLabel.Text = skyTheme end
    startEnemySpeed()
    toggleLockUI(uiLocked)
end

buildGui()
if loadAllSettings() then updateUIFromLoaded() end

task.defer(function()
    task.wait(0.15)
    if _RaptureApplyThemeToSliders then _RaptureApplyThemeToSliders() end
    for _, entry in ipairs(_G._RaptureDynamicSliders) do
        pcall(function() entry.applyThemeColor(PURPLE_LIGHT) end)
    end
end)

speedMode = false
laggerToggled = false
laggerCarryToggled = false
refreshSpeedModeLabel()

MobilePanel = createMobilePanel()
tpBatFloatingButton = createTpBatFloatingButton()
batBypassFloatingButton = createBatBypassFloatingButton()
instaResetFloatingButton = createInstaResetFloatingButton()
platformFloatingButton = createPlatformFloatingButton()

task.defer(function()
    task.wait(0.1)
    if _G._RaptureApplyMobileButtonsHidden then _G._RaptureApplyMobileButtonsHidden() end
end)

if LP.Character then
    task.wait(0.1)
    if waitForCharReady(LP.Character, 5) then setupMovementAndIndicators(LP.Character) end
end

LP.CharacterAdded:Connect(function(char)
    if stealConnection then stealConnection:Disconnect(); stealConnection = nil end    isStealing = false
    stopAutoLeft()
    stopAutoRight()
    stopBatCounter()
    stopMedusaCounter()
    if not _tpBatUnwalkForced then stopUnwalk() end
    stopDropBrainrot()
    if autoBatEnabled then disableAutoBat() end    if batDesyncTpEnabled then stopBatDesyncTp() end
    if _G._RaptureBatBypass and _G._RaptureBatBypass.enabled then _G.RaptureStopBatBypass() end
    task.wait(0.5)
    if not char.Parent then return end
    _hookedVelParts = {}
    _setupVelChecked(char)
    setupMovementAndIndicators(char)
    if antiRagdollEnabled then
        if antiRagdollVersion == "v1" then AntiRagdollV1.start()
        elseif antiRagdollVersion == "v2" then startAntiRagdollV2() end
    end
    if antiDieEnabled then _antiDie.StartEngine() end
    if antiFlingEnabled then startAntiFling() end
    if CONFIG.AUTO_STEAL_ENABLED then pcall(startAutoSteal) end
    if batDesyncTpEnabled then task.defer(startBatDesyncTp) end
    if medusaCounterEnabled then setupMedusaCounter(char) end
    if batCounterEnabled then startBatCounter() end
    if espEnabled then stopESP(); task.wait(0.2); startESP() end
    if ragdollCountdownEnabled then hookRagdollCountdown(char) end
    if unwalkEnabled and not _tpBatUnwalkForced then startUnwalk() end
    if bodyLockEnabled then startBodyLock() end
    if syncAfterHitEnabled then
        task.wait(0.3)
        _syncStartSystems(char)
    end
    if spinEnabled then
        task.wait(0.3)
        if _G._RaptureSpinSet then _G._RaptureSpinSet(true) end
    end
    refreshSpeedModeLabel()
end)

local lastLaggerToggle = 0
local LAGGER_COOLDOWN = 0.3

UIS.InputBegan:Connect(function(input, gpe)
    if _anyKeyListening then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if gpe or UIS:GetFocusedTextBox() then return end
    elseif not isGamepadInput(input) then return end
    if not isBindableInput(input) then return end
    local kc = input.KeyCode
    if not kc then return end
    if kbMatch(KB.LaggerMode, kc) then
        if _tick() - lastLaggerToggle >= LAGGER_COOLDOWN then
            lastLaggerToggle = _tick()
            toggleLaggerCycle()
        end
        return
    end
    if kbMatch(KB.CarryToggle, kc) then toggleCarryMode(); return end
    if kbMatch(KB.DropBrainrot, kc) then
        if not dropActive then
            if dropBrainrotSetVisual then dropBrainrotSetVisual(true) end
            executeDropWithToggle(dropBrainrotSetVisual)
        end
        return
    end
    if kbMatch(KB.TPFloor, kc) then doTpDown(); return end
    if kbMatch(KB.InstaReset, kc) then
        if _G.InstaReset and _G.InstaReset.Trigger then _G.InstaReset.Trigger() end
        return
    end
    if kbMatch(KB.BatBypass, kc) then
        _G.RaptureToggleBatBypass()
        if batBypassSetVisual then batBypassSetVisual(_G._RaptureBatBypass.enabled) end
        return
    end
    if kbMatch(KB.AutoLeft, kc) then
        autoLeftEnabled = not autoLeftEnabled
        if autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
        if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
        if mobSetAutoLeft then mobSetAutoLeft(autoLeftEnabled) end
        return
    end
    if kbMatch(KB.AutoRight, kc) then
        autoRightEnabled = not autoRightEnabled
        if autoRightEnabled then startAutoRight() else stopAutoRight() end
        if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
        if mobSetAutoRight then mobSetAutoRight(autoRightEnabled) end
        return
    end
    if kbMatch(KB.AutoBat, kc) then
        if not autoBatEnabled then
            enableAutoBat()
            if autoBatSetVisual then autoBatSetVisual(true) end
            if mobSetAutoBat then mobSetAutoBat(true) end
        else
            disableAutoBat()
            if autoBatSetVisual then autoBatSetVisual(false) end
            if mobSetAutoBat then mobSetAutoBat(false) end
        end
        return
    end
    if kbMatch(KB.TPBat, kc) then
        toggleBatDesyncTp()
        if batDesyncTpSetVisual then batDesyncTpSetVisual(batDesyncTpEnabled) end
        return
    end
    if kbMatch(KB.Spin, kc) then
        if not spinEnabled then
            if spinAutoBrainrotEnabled then
                spinAutoBrainrotEnabled = false
                if spinAutoBrainrotSetVisual then spinAutoBrainrotSetVisual(false) end
            end
            spinEnabled = true
            if spinSetVisual then spinSetVisual(true) end
            if _G._RaptureSpinSet then _G._RaptureSpinSet(true) end
        else
            spinEnabled = false
            if spinSetVisual then spinSetVisual(false) end
            if _G._RaptureSpinSet then _G._RaptureSpinSet(false) end
        end
        return
    end
    if kbMatch(KB.Platform, kc) then
        if _G.RapturePlatformToggle then _G.RapturePlatformToggle() end
        return
    end
    if kbMatch(KB.GuiHide, kc) then
        if main then
            if main.Visible then hideGui() else showGui() end
        end
        return
    end
end)

task.defer(function()
    task.wait(0.1)
    _RaptureApplyThemeToSliders()
end)

print("===========================================")
print("  RAPTURE DUEL CARGADO v8.0.61")
print("  Botones flotantes sin borde negro")
print("  Platform morado al activarse")
print("  Bat Bypass auto-equipa bat sin bugs")
print("  TP Down automatico 0.2s tras plataforma")
print("  Infinite Jump intacto")
print("===========================================")
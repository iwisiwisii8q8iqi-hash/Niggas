pcall(function()
    if setthreadidentity then
        setthreadidentity(8)
    end
end)
local P = game:GetService("Players")
local C = P.LocalPlayer
if not game:IsLoaded() then
    game.Loaded:Wait()
end
local P = {
    SelectedPetData = nil,
    AllAnimalsCache = nil,
    ListNeedsRedraw = true,
    MobileScaleObjects = {},
    RefreshUIScale = nil,
}

do
    local C = { AUTO_STEAL = false, RADIUS = 12 }
    local r = {
        {
            min = Vector3.new(-337.448303, -3.898971, -122.397758),
            max = Vector3.new(-328.004578, -3.898971, 242.625626),
        },
        {
            min = Vector3.new(-327.25766, -3.899109, -122.228622),
            max = Vector3.new(-320.600891, -3.899109, 242.612259),
        },
        {
            min = Vector3.new(-319.783386, -3.89897, -122.227089),
            max = Vector3.new(-312.908325, -3.89897, 242.585617),
        },
        {
            min = Vector3.new(-312.445648, -3.899108, -122.389832),
            max = Vector3.new(-305.489899, -3.899108, 242.456818),
        },
        {
            min = Vector3.new(-305.037048, -3.89897, -122.230743),
            max = Vector3.new(-293.957489, -3.89897, 242.606873),
        },
        {
            min = Vector3.new(-491.448608, -3.898972, -122.253258),
            max = Vector3.new(-481.811737, -3.898972, 242.615005),
        },
        {
            min = Vector3.new(-498.971069, -3.89897, -122.382767),
            max = Vector3.new(-491.74884, -3.89897, 242.612061),
        },
        {
            min = Vector3.new(-506.436737, -3.898972, -122.411476),
            max = Vector3.new(-499.318542, -3.898972, 242.615982),
        },
        {
            min = Vector3.new(-513.783569, -3.898972, -122.223297),
            max = Vector3.new(-506.801849, -3.898972, 242.62709),
        },
        {
            min = Vector3.new(-525.236938, -3.898972, -122.409813),
            max = Vector3.new(-514.265015, -3.898972, 242.608932),
        },
    }
    local R = game:GetService("Players")
    local d = game:GetService("RunService")
    local n = R.LocalPlayer
    do
        local R
        local function f()
            n.DevEnableMouseLock = true
            n.DevCameraOcclusionMode = Enum.DevCameraOcclusionMode.Invisicam
            if R then
                R:Disconnect()
            end
            R = d.RenderStepped:Connect(function()
                local R = workspace.CurrentCamera
                if R and R.CameraSubject and R.CameraType == Enum.CameraType.Custom then
                    R.CFrame = R.CFrame
                end
            end)
        end
        f()
        n.CharacterAdded:Connect(function()
            task.wait(0.5)
            f()
        end)
    end
    local R = {}
    local d = {}
    local f = 0
    function _G.getSafePollRate()
        if os.clock() < f then
            return 0.27
        end
        return 0.1
    end
    function _G.triggerSafePollBoost()
        f = os.clock() + 3
    end
    local f = {}
    local function J()
        local a = n.Character
        return a and a:FindFirstChild("HumanoidRootPart")
    end
    local function n(a)
        for e, x in ipairs(r) do
            if
                a.X >= math.min(x.min.X, x.max.X)
                and a.X <= math.max(x.min.X, x.max.X)
                and a.Z >= math.min(x.min.Z, x.max.Z)
                and a.Z <= math.max(x.min.Z, x.max.Z)
            then
                return e
            end
        end
    end
    local function r(a)
        local e = a.Parent
        if not e then
            return
        end
        if e:IsA("Attachment") and e.Parent then
            e = e.Parent
        end
        if e:IsA("BasePart") then
            return e.Position
        elseif e:IsA("Model") then
            return e:GetPivot().Position
        end
    end
    local function a(e)
        if not P then
            return false
        end
        local x = P.SelectedPetData
        if not x then
            return false
        end
        local Q = e:FindFirstAncestorOfClass("Model")
        if not Q then
            return false
        end
        if x.plot then
            local V = e:FindFirstAncestor(x.plot)
            if not V then
                return false
            end
        end
        if x.slot then
            local V = e:FindFirstAncestor(x.slot)
            if V then
                return true
            end
            if Q.Name == x.slot then
                return true
            end
            if Q.Parent and Q.Parent.Name == x.slot then
                return true
            end
        end
        if x.name then
            local e = string.lower(x.name)
            local x = Q
            while x do
                if x.Name and string.lower(x.Name) == e then
                    return true
                end
                x = x.Parent
            end
        end
        return false
    end
    local function e(x, Q)
        if not x or not x.Parent then
            return false
        end
        if not x.Enabled then
            return false
        end
        local V = r(x)
        if not V then
            return false
        end
        local r = x:FindFirstAncestorOfClass("Model")
        if r then
            local r = workspace:FindFirstChild("Plots")
            if r then
                local i = x:FindFirstAncestorWhichIsA("Model")
                while i and i.Parent ~= r do
                    i = i.Parent
                end
                if i then
                    local r = i:FindFirstChild("PlotSign")
                    if r then
                        local i = r:FindFirstChildWhichIsA("SurfaceGui", true)
                        local r = i and i:FindFirstChildWhichIsA("TextLabel", true)
                        if r then
                            local i = r.Text:lower()
                            if
                                i:find(game.Players.LocalPlayer.Name:lower(), 1, true)
                                or i:find(game.Players.LocalPlayer.DisplayName:lower(), 1, true)
                            then
                                return false
                            end
                        end
                    end
                end
            end
        end
        if _G.NEAREST_INSTANT_MODE == true then
            local r = n(Q)
            local i = n(V)
            if not r or r ~= i then
                return false
            end
        end
        if _G.NEAREST_INSTANT_MODE ~= true then
            if not a(x) then
                return false
            end
        end
        local r = typeof(x.MaxActivationDistance) == "number"
                and x.MaxActivationDistance > 0
                and x.MaxActivationDistance
            or C.RADIUS
        local n = math.min(C.RADIUS, r)
        return (V - Q).Magnitude <= n
    end
    local function r(n, a)
        local x = os.clock()
        local Q = d[n]
        if Q and x - Q < a then
            return false
        end
        d[n] = x
        return true
    end
    local function n(a, x, Q)
        if not a or not a.Parent then
            return
        end
        if not a.Enabled then
            return
        end
        if not r(a, Q) then
            return
        end
        for r = 1, x do
            pcall(function()
                fireproximityprompt(a, 0)
            end)
        end
    end
    local function r(a)
        if R[a] then
            return
        end
        R[a] = true
        local function x()
            local Q = J()
            if not Q then
                return
            end
            local V = Q.Position
            if e(a, V) then
                C.AUTO_STEAL = true
                local Q = os.clock()
                local V = f[a]
                if not V or Q - V >= 0.04 then
                    f[a] = Q
                    n(a, 25, 0)
                end
            end
        end
        task.defer(function()
            x()
        end)
        pcall(function()
            a:GetPropertyChangedSignal("Enabled"):Connect(function()
                if a.Enabled then
                    x()
                end
            end)
        end)
        a.AncestryChanged:Connect(function()
            if not a:IsDescendantOf(workspace) then
                R[a] = nil
                d[a] = nil
                f[a] = nil
            end
        end)
    end
    local function d()
        local f = workspace:FindFirstChild("Plots")
        if not f then
            return
        end
        for a, a in ipairs(f:GetChildren()) do
            local f = a:FindFirstChild("AnimalPodiums")
            if f then
                for a, a in ipairs(f:GetDescendants()) do
                    if a:IsA("ProximityPrompt") then
                        r(a)
                    end
                end
            end
        end
    end
    d()
    workspace.DescendantAdded:Connect(function(d)
        if d:IsA("ProximityPrompt") and d:FindFirstAncestor("AnimalPodiums") then
            r(d)
        end
    end)
    task.spawn(function()
        while task.wait(_G.getSafePollRate()) do
            local r = J()
            if not r then
                C.AUTO_STEAL = false
                continue
            end
            local d = r.Position
            local r = false
            for f in pairs(R) do
                if e(f, d) then
                    r = true
                    if C.AUTO_STEAL then
                        n(f, 5, 0.05)
                    end
                end
            end
            C.AUTO_STEAL = r
        end
    end)
end
local C = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    TweenService = game:GetService("TweenService"),
    HttpService = game:GetService("HttpService"),
    Workspace = game:GetService("Workspace"),
    Lighting = game:GetService("Lighting"),
    GuiService = game:GetService("GuiService"),
    TeleportService = game:GetService("TeleportService"),
}
local r = C.Players
local R = C.RunService
local d = C.UserInputService
local n = C.ReplicatedStorage
local f = C.TweenService
local J = C.HttpService
local a = C.Workspace
local e = C.Lighting
local e = C.GuiService
local e = C.TeleportService
local C = r.LocalPlayer
local x = C:WaitForChild("PlayerGui")
local Q
Q = setmetatable({}, {
    __index = function(V, V)
        local i = n.Packages.Net
        local O, U
        if V:sub(1, 3) == "RE/" then
            O = "RE/"
            U = V:sub(4)
        elseif V:sub(1, 3) == "RF/" then
            O = "RF/"
            U = V:sub(4)
        else
            return nil
        end
        local O
        for U, y in i:GetChildren() do
            if y.Name == V then
                O = i:GetChildren()[U + 1]
                break
            end
        end
        if O and not rawget(Q, V) then
            rawset(Q, V, O)
        end
        return rawget(Q, V)
    end,
})
local V = {}
function V:LarpNet(V)
    return Q[V]
end
local Q = a.CurrentCamera
local V = C:GetMouse()

local function V()
    return d.TouchEnabled and not d.KeyboardEnabled and not d.MouseEnabled
end
local d = V()
local UI_PANEL_W = 320
local UI_SCALE = 0.60
local V = {
    Positions = {
        AutoSteal = { X = 0, Y = 0, OffsetX = 14, OffsetY = 80 },
        TargetControls = { X = 0, Y = 0, OffsetX = 14 + UI_PANEL_W * UI_SCALE + 10, OffsetY = 80 },
        JobID = { X = 0.5, Y = 0, OffsetX = -180, OffsetY = 300 },
        InvisSteal = { X = 0, Y = 0, OffsetX = 14, OffsetY = 470 },
        MainUI = { X = 0.5, Y = 0.5, OffsetX = -275, OffsetY = -225 },
    },
    MenuKey = "LeftControl",
    MobileGuiScale = 0.5,
    UIScale = 1.0,
    ScaleAutoSteal = 1.0,
    ScaleTargetControls = 1.0,
    ScaleJobID = 1.0,
    ScaleInvisSteal = 1.0,
    ShowJobID = true,
    StealNearest = false,
    StealHighest = true,
    StealPriority = false,
    UILocked = false,
    HideAutoSteal = false,
    HideTargetControls = false,
    HideInvisSteal = false,
    CompactAutoSteal = false,
    InstantSteal = false,
    InstantStealV2 = false,
    nextBaseEnabled = false,
    XRay = false,
    PlayerESP = false,
    podiumESP = false,
    FloorPlatform = false,
    TurretESP = false,
    TrapESP = false,
    BrainrotESP = false,
    BrainrotESPMinGen = 10000000,
    AntiRagdoll = true,
    AntiBeeDisco = true,
    AntiDie = true,
    AutoTurret = false,
    ToolAimbot = false,
    StealBoost = false,
    StealBoostValue = 28,
    InvisibleStealKey = "V",
    AutoInvisDuringSteal = true,
    InvisDelayEnabled = false,
    InvisDelayMs = 0,
    InvisSinkValue = 2.5,
    InvisStealAngle = 233,
    AntiLag = false,
    CarpetSpeed = false,
    CarpetSpeedValue = 140,
    CarpetSpeedKey = "Q",
    DropKey = "R",
    InstantResetKey = "X",
    AutoKickKey = "K",
    StealBoostKey = "B",
    KickKey = "P",
    InfJump = true,
    AntiTrapEnabled = false,
    BaseTimerESP = false,
    HighValueSoundEnabled = true,
    HighValueSoundId = "100173184074904",
    HighValueMinGen = 5000000,
    InstantClonerEnabled = false,
    AutoKickOnSteal = false,
    MainUIHidden = false,
    FOV = 80,
    ResetCooldown = 2.5,
    ResetFlingTime = 5,
    PriorityList = {
        "Strawberry Elephant",
        "Meowl",
        "Skibidi Toilet",
        "Headless Horseman",
        "Dragon Gingerini",
        "Dragon Cannelloni",
        "Ketupat Bros",
        "Hydra Dragon Cannelloni",
        "La Supreme Combinasion",
        "Love Love Bear",
        "Ginger Gerat",
        "Cerberus",
        "Capitano Moby",
        "La Casa Boo",
        "Burguro and Fryuro",
        "Spooky and Pumpky",
        "Cooki and Milki",
        "Rosey and Teddy",
        "Popcuru and Fizzuru",
        "Reinito Sleighito",
        "Fragrama and Chocrama",
        "Garama and Madundung",
        "Ketchuru and Musturu",
        "La Secret Combinasion",
        "Tralaledon",
        "Tictac Sahur",
        "Ketupat Kepat",
        "Tang Tang Keletang",
        "Orcaledon",
        "La Ginger Sekolah",
        "Los Spaghettis",
        "Lavadorito Spinito",
        "Swaggy Bros",
        "La Taco Combinasion",
        "Los Primos",
        "Chillin Chili",
        "Tuff Toucan",
        "W or L",
        "Chillin Chili",
        "Chipso and Queso",
    },
}
function DeepCopy(i)
    if type(i) ~= "table" then
        return i
    end
    local O = {}
    for U, y in pairs(i) do
        O[U] = DeepCopy(y)
    end
    return O
end
function MergeDefaults(i, O)
    for U, y in pairs(O) do
        if type(y) == "table" then
            if type(i[U]) ~= "table" then
                i[U] = DeepCopy(y)
            else
                MergeDefaults(i[U], y)
            end
        elseif i[U] == nil then
            i[U] = y
        end
    end
end
local i = DeepCopy(V)

local function currentUIScale()
    return math.clamp(tonumber(i.UIScale) or 1.0, 0.3, 2.0)
end

local function deviceClass()
    local uis = game:GetService("UserInputService")
    local touch, kbd, mouse = false, false, false
    pcall(function()
        touch, kbd, mouse = uis.TouchEnabled, uis.KeyboardEnabled, uis.MouseEnabled
    end)
    if not touch or kbd or mouse then return "desktop" end
    local cam = workspace.CurrentCamera
    local vp = cam and cam.ViewportSize
    local shortSide = (vp and vp.X > 0 and vp.Y > 0) and math.min(vp.X, vp.Y) or 400
    return shortSide >= 600 and "tablet" or "phone"
end
P.DeviceClass = deviceClass

local function RegisterUIScale(targetFrame)
    if not targetFrame then return end
    local existing = targetFrame:FindFirstChildOfClass("UIScale")
    if existing then existing:Destroy() end
    local sc = Instance.new("UIScale")
    sc.Scale = currentUIScale()
    sc.Parent = targetFrame
    P.MobileScaleObjects[targetFrame] = sc
end

P.RefreshUIScale = function()
    local s = currentUIScale()
    for frame, sc in pairs(P.MobileScaleObjects) do
        if sc and sc.Parent == frame then
            sc.Scale = s
        else
            P.MobileScaleObjects[frame] = nil
        end
    end
end

local TGT = {}
TGT.MAC_RED = Color3.fromRGB(255, 95, 87)
TGT.MAC_YELLOW = Color3.fromRGB(254, 188, 46)
TGT.MAC_GREEN = Color3.fromRGB(40, 200, 64)
TGT.OUTLINE = Color3.fromRGB(186, 117, 245)
TGT.FPS = Color3.fromRGB(242, 170, 58)
TGT.PING = Color3.fromRGB(70, 224, 140)

function TGT.grad(obj, stops, rotation)
    local cs, ns = {}, {}
    for _, st in ipairs(stops) do
        cs[#cs + 1] = ColorSequenceKeypoint.new(st[1], st[2])
        ns[#ns + 1] = NumberSequenceKeypoint.new(st[1], st[3] or 0)
    end
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new(cs)
    g.Transparency = NumberSequence.new(ns)
    g.Rotation = rotation or 0
    g.Parent = obj
    return g
end

function TGT.panelGrad(obj)
    return TGT.grad(obj, {
        { 0, Color3.fromRGB(23, 9, 39), 0.06 },
        { 0.25, Color3.fromRGB(41, 16, 67), 0.08 },
        { 0.5, Color3.fromRGB(61, 24, 95), 0.10 },
        { 0.75, Color3.fromRGB(80, 34, 122), 0.12 },
        { 1, Color3.fromRGB(18, 7, 30), 0.06 },
    }, 135)
end

function TGT.strokeGrad(obj)
    return TGT.grad(obj, {
        { 0, Color3.fromRGB(129, 65, 181), 0.45 },
        { 0.5, Color3.fromRGB(186, 117, 245), 0.22 },
        { 1, Color3.fromRGB(100, 47, 149), 0.42 },
    }, 90)
end

function TGT.shadow(obj)
    local sh = Instance.new("ImageLabel")
    sh.Name = "Shadow"
    sh.ZIndex = 0
    sh.AnchorPoint = Vector2.new(0.5, 0.5)
    sh.Position = UDim2.new(0.5, 0, 0.5, 4)
    sh.Size = UDim2.new(1, 46, 1, 46)
    sh.BackgroundTransparency = 1
    sh.Image = "rbxassetid://6014261993"
    sh.ImageColor3 = Color3.fromRGB(11, 4, 19)
    sh.ImageTransparency = 0.22
    sh.ScaleType = Enum.ScaleType.Slice
    sh.SliceCenter = Rect.new(49, 49, 450, 450)
    sh.Parent = obj
    return sh
end

P.PinnedPanels = {}
function TGT.macDots(parent, dotSize, startX, gap, zindex, pinTarget)
    dotSize = dotSize or 11
    startX = startX or 15
    gap = gap or 18
    local cols = { TGT.MAC_RED, TGT.MAC_YELLOW, TGT.MAC_GREEN }
    for idx, col in ipairs(cols) do
        local d = Instance.new("Frame", parent)
        d.Name = "Dot" .. idx
        d.AnchorPoint = Vector2.new(0, 0.5)
        d.Position = UDim2.new(0, startX + gap * (idx - 1), 0.5, 0)
        d.Size = UDim2.fromOffset(dotSize, dotSize)
        d.BackgroundColor3 = col
        d.BackgroundTransparency = idx == 1 and 0 or 0.55
        d.BorderSizePixel = 0
        d.ZIndex = zindex or 105
        Instance.new("UICorner", d).CornerRadius = UDim.new(0, math.ceil(dotSize / 2))
    end
end

function TGT.dressPanel(panel, radius)
    panel.BackgroundColor3 = Color3.fromRGB(42, 17, 69)
    panel.BackgroundTransparency = 0.35
    TGT.panelGrad(panel)
    TGT.shadow(panel)
    local st = panel:FindFirstChildOfClass("UIStroke")
    if not st then
        st = Instance.new("UIStroke", panel)
        st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    end
    st.Color = TGT.OUTLINE
    st.Thickness = 1.4
    st.Transparency = 0
    TGT.strokeGrad(st)
    return st
end

function TGT.minimizeBtn(parent, zindex)
    local b = Instance.new("TextButton", parent)
    b.Name = "Minimize"
    b.AnchorPoint = Vector2.new(1, 0.5)
    b.Position = UDim2.new(1, -10, 0.5, 0)
    b.Size = UDim2.fromOffset(30, 27)
    b.BackgroundTransparency = 1
    b.BorderSizePixel = 0
    b.AutoButtonColor = false
    b.Font = Enum.Font.GothamBlack
    b.Text = "-"
    b.TextSize = 19
    b.TextColor3 = TGT.MAC_YELLOW
    b.ZIndex = zindex or 106
    return b
end
P.TGT = TGT

P.ToggleUIPainters = {}
_G.StickySyncToggleUI = function(name, on)
    local painter = P.ToggleUIPainters[name]
    if painter then pcall(painter, on) end
end

do
    local UIS = game:GetService("UserInputService")
    local function keyFor(field, fallback)
        local name = i[field]
        if type(name) ~= "string" or name == "" then name = fallback end
        return Enum.KeyCode[name]
    end
    UIS.InputBegan:Connect(function(input, processed)
        if processed then return end
        if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
        local code = input.KeyCode
        local menuName = i.SavedKeybinds and i.SavedKeybinds.MenuKey or i.MenuKey
        if code == Enum.KeyCode[menuName or "LeftControl"] then
            if #P.PanelsToReveal == 0 then P.ToggleUI() end
        elseif code == keyFor("DropKey", "R") then
            if type(_G.StickyDropBrainrot) == "function" then
                task.spawn(_G.StickyDropBrainrot)
            end
        elseif code == keyFor("InstantResetKey", "X") then
            if type(_G.StickyInstaReset) == "function" then
                task.spawn(_G.StickyInstaReset)
            end
        elseif code == keyFor("KickKey", "P") then
            if type(_G.StickyKickOut) == "function" then
                task.spawn(_G.StickyKickOut)
            end
        elseif code == keyFor("StealBoostKey", "B") then
            local on = not (i.StealBoost == true)
            if type(_G.setStealBoost) == "function" then
                task.spawn(_G.setStealBoost, on)
            end
        elseif code == keyFor("AutoKickKey", "K") then
            local on = not (_G.StickyAutoKickOnSteal == true)
            _G.StickyAutoKickOnSteal = on
            _G.StickySyncToggleUI("Auto Kick", on)
            if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
        end
    end)
end

P.PanelsToReveal = {}
P.HoldPanel = function(inst)
    if not inst then return end
    P.PanelsToReveal[#P.PanelsToReveal + 1] = inst
    inst.Visible = false
end

P.UIHidden = false
P.SetUIHidden = function(hidden)
    P.UIHidden = hidden and true or false
    i.MainUIHidden = P.UIHidden
    if P.MainUIFrame and P.MainUIFrame.Parent then
        P.MainUIFrame.Visible = not P.UIHidden
    end
    if P.MenuButton then
        P.MenuButton.Text = P.UIHidden and "+" or "x"
    end
    if _G.StickySaveConfigNow then pcall(_G.StickySaveConfigNow) end
end
P.ToggleUI = function()
    P.SetUIHidden(not P.UIHidden)
end
P.RevealPanels = function()
    local pending = P.PanelsToReveal
    P.PanelsToReveal = {}
    for _, inst in ipairs(pending) do
        if inst and inst.Parent then inst.Visible = true end
    end
    if P.MainUIFrame and P.MainUIFrame.Parent then
        local shouldHide = i.MainUIHidden == true
        P.MainUIFrame.Visible = not shouldHide
        P.UIHidden = shouldHide
        if P.MenuButton then
            P.MenuButton.Text = shouldHide and "+" or "x"
        end
    end
end
task.delay(15, function()
    if #P.PanelsToReveal > 0 then P.RevealPanels() end
end)

function NormalizeKeyName(O, U)
    if type(O) ~= "string" or O == "" then
        return U
    end
    local y = {
        ALT = "LeftAlt",
        LALT = "LeftAlt",
        RALT = "RightAlt",
        CTRL = "LeftControl",
        CONTROL = "LeftControl",
        LCTRL = "LeftControl",
        RCTRL = "RightControl",
        SHIFT = "LeftShift",
        LSHIFT = "LeftShift",
        RSHIFT = "RightShift",
        WIN = "LeftSuper",
        CMD = "LeftSuper",
        META = "LeftSuper",
    }
    local K = string.upper(O)
    local q = y[K] or O
    return Enum.KeyCode[q] and q or O
end
function PrettyKeyName(O)
    local U = {
        LeftAlt = "ALT",
        RightAlt = "RALT",
        LeftControl = "CTRL",
        RightControl = "RCTRL",
        LeftShift = "SHIFT",
        RightShift = "RSHIFT",
        LeftSuper = "WIN",
        RightSuper = "RWIN",
    }
    return U[O] or tostring(O or "")
end
if isfile and isfile("StickyHub.json") then
    pcall(function()
        local O = readfile("StickyHub.json")
        if not O or O == "" then
            return
        end
        local U = J:JSONDecode(O)
        if type(U) ~= "table" then
            return
        end
        local O = nil
        if type(U.PriorityList) == "table" then
            O = DeepCopy(U.PriorityList)
        end
        MergeDefaults(U, V)
        if O ~= nil then
            U.PriorityList = O
        end
        U.DefaultToNearest = nil
        U.DefaultToHighest = nil
        U.DefaultToPriority = nil
        i = U
    end)
end
if i.Positions then
    i.Positions.Settings = { X = 0.5, Y = 0.5, OffsetX = 0, OffsetY = 0 }
end
if i.LayoutRev ~= 4 then
    i.Positions = i.Positions or {}
    i.Positions.AutoSteal = DeepCopy(V.Positions.AutoSteal)
    i.Positions.TargetControls = DeepCopy(V.Positions.TargetControls)
    i.LayoutRev = 4
    i.FixedLayout = nil
    _needMigrationSave = true
end
function EnsureSavedKeybinds()
    i.SavedKeybinds = i.SavedKeybinds or {}
    i.SavedKeybinds.MenuKey =
        NormalizeKeyName(i.SavedKeybinds.MenuKey or i.MenuKey, V.MenuKey or "LeftControl")
end
function ApplySavedKeybindsToConfig()
    EnsureSavedKeybinds()
    i.MenuKey = i.SavedKeybinds.MenuKey
end
function NormalizeAllKeybinds()
    EnsureSavedKeybinds()
    ApplySavedKeybindsToConfig()
end
NormalizeAllKeybinds()
local function SanitizeForSave(v, depth, seen)
    local t = type(v)
    if t == "string" or t == "boolean" then
        return v
    elseif t == "number" then
        if v ~= v or v == math.huge or v == -math.huge then return 0 end
        return v
    elseif t == "table" then
        if depth > 8 or seen[v] then return nil end
        seen[v] = true
        local out = {}
        for k, val in pairs(v) do
            local kt = type(k)
            if kt == "string" or kt == "number" then
                local sv = SanitizeForSave(val, depth + 1, seen)
                if sv ~= nil then out[k] = sv end
            end
        end
        seen[v] = nil
        return out
    end
    return nil
end
local function U()
    if not writefile then return end
    local ok, err = pcall(function()
        NormalizeAllKeybinds()
        local y = SanitizeForSave(i, 0, {}) or {}
        y.MenuKey = i.SavedKeybinds and i.SavedKeybinds.MenuKey or y.MenuKey
        writefile("StickyHub.json", J:JSONEncode(y))
    end)
    if not ok then
        pcall(function() warn("[Sticky] config save failed: " .. tostring(err)) end)
    end
end
if _needMigrationSave then
    U()
end
do
    _G.StickyCarpetSpeed = i.CarpetSpeed == true
    _G.StickyCarpetSpeedValue = math.clamp(tonumber(i.CarpetSpeedValue) or 140, 20, 400)
    _G.StickyCarpetSpeedKeyName = type(i.CarpetSpeedKey) == "string" and i.CarpetSpeedKey or "Q"
    _G.StickyAutoKickOnSteal = i.AutoKickOnSteal == true
    _G.StickyFOV = tonumber(i.FOV) or 80
    _G.StickyResetCooldown = tonumber(i.ResetCooldown) or 2.5
    _G.StickyResetFlingTime = tonumber(i.ResetFlingTime) or 5
    _G.StickyAntiBee = i.AntiBeeDisco ~= false
    _G.StickyAntiDieDisabled = i.AntiDie == false
    _G.AntiTrapEnabledSaved = i.AntiTrapEnabled == true
    _G.PlotTimerESPSaved = i.BaseTimerESP == true
    _G.HighValueSoundEnabled = i.HighValueSoundEnabled ~= false
    _G.HighValueSoundId = type(i.HighValueSoundId) == "string" and i.HighValueSoundId or "100173184074904"
    _G.HighValueMinGen = tonumber(i.HighValueMinGen) or 5000000
    _G.StickyInstantCloner = i.InstantClonerEnabled == true

    local function syncGlobalsToConfig()
        i.CarpetSpeedValue = math.clamp(tonumber(_G.StickyCarpetSpeedValue) or 140, 20, 400)
        i.FOV = tonumber(_G.StickyFOV) or i.FOV
        i.ResetCooldown = tonumber(_G.StickyResetCooldown) or i.ResetCooldown
        i.ResetFlingTime = tonumber(_G.StickyResetFlingTime) or i.ResetFlingTime
        i.AntiBeeDisco = _G.StickyAntiBee ~= false
        i.AntiDie = _G.StickyAntiDieDisabled ~= true
        if type(_G.StickyCarpetSpeedKeyName) == "string" then
            i.CarpetSpeedKey = _G.StickyCarpetSpeedKeyName
        end
        if type(_G.StickyInfJump) == "table" and _G.StickyInfJump.IsEnabled then
            i.InfJump = _G.StickyInfJump.IsEnabled()
        end
        if _G.StickyCarpetSpeed ~= nil then i.CarpetSpeed = _G.StickyCarpetSpeed == true end
        if _G.StickyAutoKickOnSteal ~= nil then i.AutoKickOnSteal = _G.StickyAutoKickOnSteal == true end
        if _G.AntiTrapEnabledSaved ~= nil then i.AntiTrapEnabled = _G.AntiTrapEnabledSaved == true end
        if _G.PlotTimerESPSaved ~= nil then i.BaseTimerESP = _G.PlotTimerESPSaved == true end
        if _G.HighValueSoundEnabled ~= nil then i.HighValueSoundEnabled = _G.HighValueSoundEnabled ~= false end
        if _G.HighValueSoundId ~= nil then i.HighValueSoundId = tostring(_G.HighValueSoundId) end
        if _G.HighValueMinGen ~= nil then i.HighValueMinGen = tonumber(_G.HighValueMinGen) or 5000000 end
        if _G.StickyInstantCloner ~= nil then i.InstantClonerEnabled = _G.StickyInstantCloner == true end
    end

    local lastSnapshot = nil
    _G.StickySaveConfigNow = function()
        pcall(syncGlobalsToConfig)
        local ok, snap = pcall(function() return J:JSONEncode(SanitizeForSave(i, 0, {})) end)
        if ok and snap == lastSnapshot then return end
        lastSnapshot = ok and snap or nil
        U()
    end

    task.spawn(function()
        task.wait(5)
        while true do
            _G.StickySaveConfigNow()
            task.wait(5)
        end
    end)
end
do
    local ready = false
    local waiters = {}

    _G.StickyOnBoot = function(fn)
        if type(fn) ~= "function" then return end
        if ready then task.spawn(fn) return end
        waiters[#waiters + 1] = fn
    end

    task.spawn(function()
        local deadline = os.clock() + 30
        while not workspace:FindFirstChild("Plots") and os.clock() < deadline do
            task.wait(0.25)
        end
        local charDeadline = os.clock() + 15
        while os.clock() < charDeadline do
            local ch = C.Character
            if ch and ch:FindFirstChild("HumanoidRootPart") and ch:FindFirstChildOfClass("Humanoid") then
                break
            end
            task.wait(0.25)
        end
        task.wait(1.5)
        ready = true
        for _, fn in ipairs(waiters) do task.spawn(fn) end
        waiters = {}
    end)
end
do
    local Workspace = game:GetService("Workspace")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local Config = i
    local saveConfig = U

    local function setToggle(name, enabled)
        local sync = _G.StickySyncToggleUI
        if type(sync) == "function" then
            pcall(sync, name, enabled)
        end
    end

    if _G.StickyXray      == nil then _G.StickyXray      = true end
    if _G.StickyXrayAlpha == nil then _G.StickyXrayAlpha = 0.9  end

    ;(function()
        local FOLDERS = { "Base", "PlotSign", "FriendPanel", "Cash", "Laser",
            "Decorations", "Skin", "Unlock", "Purchases" }
        local orig = setmetatable({}, { __mode = "k" })
        local conns, gen = {}, 0

        local function paint(o, a)
            if not o:IsA("BasePart") then return end
            if orig[o] == nil then orig[o] = (o.Transparency == a) and 0 or o.Transparency end
            local base = orig[o]
            if base >= 1 then return end
            local want = base + (1 - base) * a
            if math.abs(o.Transparency - want) > 0.01 then o.Transparency = want end
        end

        local function calm()
            while _G.StickyStealHold do task.wait(0.15) end
        end

        local function track(root, a, id)
            if not root or id ~= gen then return end
            paint(root, a)
            local n = 0
            for _, d in ipairs(root:GetDescendants()) do
                if id ~= gen then return end
                paint(d, a)
                n = n + 1
                if n % 250 == 0 then task.wait() end
            end
            conns[#conns + 1] = root.DescendantAdded:Connect(function(d)
                if id == gen then paint(d, a) end
            end)
        end

        local function doPlot(plot, a, id)
            if not plot or id ~= gen then return end
            for _, fname in ipairs(FOLDERS) do
                if id ~= gen then return end
                track(plot:FindFirstChild(fname), a, id)
            end
            if id ~= gen then return end
            conns[#conns + 1] = plot.ChildAdded:Connect(function(c)
                if id ~= gen then return end
                for _, fname in ipairs(FOLDERS) do
                    if c.Name == fname then track(c, a, id) break end
                end
            end)
            local pods = plot:FindFirstChild("AnimalPodiums")
            if not pods then return end
            local function pod(pd)
                for _, c in ipairs(pd:GetChildren()) do
                    if c.Name == "Claim" then track(c, a, id)
                    elseif c.Name == "Base" then track(c:FindFirstChild("Decorations"), a, id) end
                end
            end
            for _, pd in ipairs(pods:GetChildren()) do pod(pd) end
            conns[#conns + 1] = pods.ChildAdded:Connect(function(pd)
                if id ~= gen then return end
                task.wait(0.1)
                if id == gen then pod(pd) end
            end)
        end

        local function stop()
            for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
            conns, gen = {}, gen + 1
        end

        function _G.StickyEnableXray()
            Config.XRay = true
            saveConfig()
            setToggle("XRay", true)
            setToggle("X-Ray", true)
            stop()
            local id = gen
            local a = math.clamp(tonumber(_G.StickyXrayAlpha) or 0.9, 0, 1)
            task.spawn(function()
                while id == gen and not workspace:FindFirstChild("Plots") do task.wait(0.5) end
                local plots = workspace:FindFirstChild("Plots")
                if id ~= gen or not plots then return end
                calm()
                for _, p in ipairs(plots:GetChildren()) do
                    if id ~= gen then return end
                    pcall(doPlot, p, a, id)
                    task.wait()
                    calm()
                end
                conns[#conns + 1] = plots.ChildAdded:Connect(function(p)
                    if id ~= gen then return end
                    task.wait(0.2)
                    pcall(doPlot, p, a, id)
                end)
            end)
        end

        function _G.StickyDisableXray()
            Config.XRay = false
            saveConfig()
            setToggle("XRay", false)
            setToggle("X-Ray", false)
            stop()
            local snap = orig
            orig = setmetatable({}, { __mode = "k" })
            for o, t in pairs(snap) do
                pcall(function() if o:IsA("BasePart") then o.Transparency = t end end)
            end
        end

        function setXRay(enabled)
            enabled = enabled and true or false
            if enabled then
                _G.StickyEnableXray()
            else
                _G.StickyDisableXray()
            end
            Config.XRay = enabled
            saveConfig()
            setToggle("XRay", enabled)
            setToggle("X-Ray", enabled)
        end

        _G.setXRay = setXRay
    end)()

    if _G.StickyAutoKickOnSteal == nil then _G.StickyAutoKickOnSteal = i.AutoKickOnSteal == true end
    if _G.StickyKickToPS        == nil then _G.StickyKickToPS        = false end

    ;(function()
        local function psCode(link)
            link = tostring(link or ""):match("^%s*(.-)%s*$")
            if link == "" then return nil end
            if not link:find("://", 1, true) then return link end
            return link:match("[?&]privateServerLinkCode=([^&]+)")
                or link:match("[?&]linkCode=([^&]+)")
                or link:match("[?&]code=([^&]+)")
        end

        local function kickOut()
            if _G.StickyKickToPS == true then
                local code = psCode(_G.StickyPrivateServerLink)
                if code and code ~= "" then
                    local ok = pcall(function()
                        game:GetService("ExperienceService"):LaunchExperience({
                            placeId = tonumber(_G.StickyPrivateServerPlaceId) or game.PlaceId,
                            linkCode = code,
                        })
                    end)
                    if ok then return end
                end
            end
            if pcall(function() game:Shutdown() end) then return end
            pcall(function() C:Kick("") end)
        end
        _G.StickyKickOut = kickOut

        task.spawn(function()
            local PG2 = C:FindFirstChildOfClass("PlayerGui") or C:WaitForChild("PlayerGui", 10)
            if not PG2 then return end
            local hooked = setmetatable({}, { __mode = "k" })
            local function hit(t)
                return type(t) == "string" and t:lower():find("you stole", 1, true) ~= nil
            end
            local function isText(o)
                return o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox")
            end
            local function watch(o)
                if hooked[o] then return end
                hooked[o] = true
                if _G.StickyAutoKickOnSteal == true and hit(o.Text) then kickOut() return end
                o:GetPropertyChangedSignal("Text"):Connect(function()
                    if _G.StickyAutoKickOnSteal == true and hit(o.Text) then kickOut() end
                end)
            end
            local function root(g)
                g.DescendantAdded:Connect(function(d) if isText(d) then watch(d) end end)
                local n = 0
                for _, d in ipairs(g:GetDescendants()) do
                    n = n + 1
                    if n % 200 == 0 then task.wait() end
                    if isText(d) then watch(d) end
                end
            end

            while _G.StickyAutoKickOnSteal ~= true do task.wait(1) end
            PG2.ChildAdded:Connect(root)
            for _, g in ipairs(PG2:GetChildren()) do root(g) end
        end)
    end)()

    do
        local Players = game:GetService("Players")
        local RunService = game:GetService("RunService")
        local UserInputService = game:GetService("UserInputService")
        local player = Players.LocalPlayer

        local enabled = false
        local conns = {}
        local firstAirSkip = true
        local lastJumpTime = 0
        local lastJumpCooldown = 0
        local baseY = nil
        local stallCount = 0
        local gamepadDown = false

        local weakVelocities = setmetatable({}, { __mode = "k" })

        local CFG = {
            jumpPower = 37,
            maxJumpHeight = 55,
            roofClearance = 4.2,
        }

        _G._StickyJumpMode = _G._StickyJumpMode or "Hold"

        local function isHoldMode()
            return _G._StickyJumpMode == "Hold"
        end

        local function isAlive(hrp)
            if not (hrp and hrp.Parent) then return false end
            if hrp.Parent ~= player.Character then return false end
            local hum = hrp.Parent:FindFirstChildOfClass("Humanoid")
            return hum ~= nil and hum.Health > 0
        end

        local function clearVelocity(hrp, tag)
            if not hrp then return end
            local lv = hrp:FindFirstChild(tag .. "LinearVelocity")
            if lv and lv:IsA("LinearVelocity") then
                weakVelocities[lv] = nil
                pcall(function()
                    lv.Enabled = false
                    lv.LineVelocity = 0
                    lv:Destroy()
                end)
            end
            local att = hrp:FindFirstChild(tag .. "Attachment")
            if att and att:IsA("Attachment") then
                pcall(function() att:Destroy() end)
            end
        end

        local function applyVelocity(hrp, tag, power, duration)
            if not isAlive(hrp) then return nil end
            local attName = tag .. "Attachment"
            local lvName = tag .. "LinearVelocity"

            local att = hrp:FindFirstChild(attName)
            if not (att and att:IsA("Attachment")) then
                att = Instance.new("Attachment")
                att.Name = attName
                att.Parent = hrp
            end

            local lv = hrp:FindFirstChild(lvName)
            if not (lv and lv:IsA("LinearVelocity")) then
                lv = Instance.new("LinearVelocity")
                lv.Name = lvName
                lv.Parent = hrp
            end

            lv.Attachment0 = att
            lv.RelativeTo = Enum.ActuatorRelativeTo.World
            lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Line
            lv.LineDirection = Vector3.new(0, 1, 0)
            pcall(function() lv.ForceLimitsEnabled = false end)

            local token = {}
            weakVelocities[lv] = token
            lv.LineVelocity = power or 0
            lv.Enabled = true

            task.delay(duration or 0, function()
                if weakVelocities[lv] ~= token then return end
                weakVelocities[lv] = nil
                if lv.Parent then clearVelocity(hrp, tag) end
            end)
            return lv
        end

        local function getVelocityY(hrp)
            if not (hrp and hrp.Parent) then return 0 end
            local ok, vel = pcall(function()
                return hrp:GetVelocityAtPosition(hrp.Position)
            end)
            return ok and vel.Y or 0
        end

        local rayParams = RaycastParams.new()
        rayParams.FilterType = Enum.RaycastFilterType.Exclude
        pcall(function() rayParams.RespectCanCollide = true end)

        local function roofCheck(hrp, dt)
            local char = player.Character
            if not (char and hrp and hrp.Parent) then return false end
            rayParams.FilterDescendantsInstances = { char }

            local velY = math.max(getVelocityY(hrp), 0)
            if velY <= 0.25 then return false end

            local dist = (CFG.roofClearance or 4.2) + math.min(velY * math.clamp(dt or 0.016, 0.004, 0.066), 0.9)
            if not workspace:Raycast(hrp.Position, Vector3.new(0, dist, 0), rayParams) then
                return false
            end

            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.Jump = false end
            local alv = hrp.AssemblyLinearVelocity
            local hasLV = hrp:FindFirstChild("InfJumpLinearVelocity") ~= nil
            if alv.Y > 0 and (hasLV or alv.Y > 60) then
                hrp.AssemblyLinearVelocity = Vector3.new(alv.X, 0, alv.Z)
            end
            lastJumpTime = tick() + 0.12
            return true
        end

        local function heightCheck(hrp)
            if not hrp or not baseY then return false end
            local diff = hrp.Position.Y - baseY
            if (CFG.maxJumpHeight or 55) <= diff then
                baseY = hrp.Position.Y
            end
            return false
        end

        local function doJump(hrp, hum, forced)
            if not (hrp and hum) or not isAlive(hrp) then return false end
            if tick() < lastJumpTime then return false end
            if heightCheck(hrp) then
                if hrp:FindFirstChild("InfJumpLinearVelocity") then
                    clearVelocity(hrp, "InfJump")
                end
                return false
            end
            if hum.FloorMaterial ~= Enum.Material.Air then
                firstAirSkip = true
                lastJumpCooldown = 0
                baseY = hrp.Position.Y
                return false
            end
            if firstAirSkip then
                firstAirSkip = false
                return false
            end
            if tick() - lastJumpCooldown < 0.08 then return false end
            if not forced and getVelocityY(hrp) > 32 then return false end

            lastJumpCooldown = tick()
            applyVelocity(hrp, "InfJump", CFG.jumpPower, 0.06)
            return true
        end

        local function isGamepad(input)
            return input.UserInputType.Name:sub(1, 7) == "Gamepad"
        end

        local function startLoop()
            if conns.loop then return end

            conns.padDown = UserInputService.InputBegan:Connect(function(input)
                if input.KeyCode == Enum.KeyCode.ButtonA and isGamepad(input) then
                    gamepadDown = true
                end
            end)
            conns.padUp = UserInputService.InputEnded:Connect(function(input)
                if input.KeyCode == Enum.KeyCode.ButtonA and isGamepad(input) then
                    gamepadDown = false
                end
            end)
            conns.jumpReq = UserInputService.JumpRequest:Connect(function()
                if not enabled then return end
                local char = player.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hrp and hum then doJump(hrp, hum, true) end
            end)

            conns.loop = RunService.Heartbeat:Connect(function(dt)
                if not enabled then return end
                local char = player.Character
                if not char then return end
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local hum = char:FindFirstChildOfClass("Humanoid")
                if not (hrp and hum) then return end

                roofCheck(hrp, dt)

                if hum.FloorMaterial ~= Enum.Material.Air then
                    firstAirSkip = true
                    lastJumpCooldown = 0
                    baseY = hrp.Position.Y
                    stallCount = 0
                    if hrp:FindFirstChild("InfJumpLinearVelocity") then
                        clearVelocity(hrp, "InfJump")
                    end
                    return
                end
                if firstAirSkip then firstAirSkip = false end

                local holding = isHoldMode()
                    and (UserInputService:IsKeyDown(Enum.KeyCode.Space) or gamepadDown or hum.Jump == true)

                local velY = getVelocityY(hrp)
                if velY < -120 then
                    stallCount = 0
                    applyVelocity(hrp, "InfJump", -200, 0.08)
                elseif holding then
                    local blocked = tick() < lastJumpTime or heightCheck(hrp)
                    if not blocked then
                        if velY < CFG.jumpPower * 0.15 then
                            stallCount += 1
                        else
                            stallCount = 0
                        end
                        if stallCount >= 4 then
                            lastJumpTime = tick() + 0.45
                            stallCount = 0
                            blocked = true
                        end
                    end
                    if blocked then
                        if hrp:FindFirstChild("InfJumpLinearVelocity") then
                            clearVelocity(hrp, "InfJump")
                        end
                    else
                        lastJumpCooldown = tick()
                        applyVelocity(hrp, "InfJump", CFG.jumpPower, 0.12)
                    end
                else
                    stallCount = 0
                    if hrp:FindFirstChild("InfJumpLinearVelocity") then
                        clearVelocity(hrp, "InfJump")
                    end
                end
            end)
        end

        local function stopLoop()
            for _, key in ipairs({"jumpReq", "loop", "padDown", "padUp"}) do
                if conns[key] then
                    pcall(function() conns[key]:Disconnect() end)
                end
            end
            local char = player.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp then clearVelocity(hrp, "InfJump") end
            conns = {}
            lastJumpCooldown = 0
            firstAirSkip = true
            stallCount = 0
            lastJumpTime = 0
            gamepadDown = false
        end

        local function setEnabled(state)
            enabled = state and true or false
            if enabled then startLoop() else stopLoop() end
        end

        player.CharacterAdded:Connect(function()
            baseY = nil
            firstAirSkip = true
            lastJumpCooldown = 0
            stallCount = 0
            lastJumpTime = 0
            gamepadDown = false
        end)

        _G.StickyInfJump = {
            Enable = function() setEnabled(true) end,
            Disable = function() setEnabled(false) end,
            Toggle = function() setEnabled(not enabled) end,
            IsEnabled = function() return enabled end,
            SetJumpPower = function(n) CFG.jumpPower = math.clamp(tonumber(n) or 37, 35, 60) end,
            SetMode = function(mode) _G._StickyJumpMode = (mode == "Hold") and "Hold" or "Single" end,
        }

        if i.InfJump ~= false then
            setEnabled(true)
        end
    end

    if _G.StickyCarpetSpeed        == nil then _G.StickyCarpetSpeed        = i.CarpetSpeed == true end
    if _G.StickyCarpetSpeedValue   == nil then _G.StickyCarpetSpeedValue   = tonumber(i.CarpetSpeedValue) or 140 end
    if _G.StickyCarpetSpeedKeyName == nil then _G.StickyCarpetSpeedKeyName = i.CarpetSpeedKey or "Q" end

    ;(function()
        local UIS = game:GetService("UserInputService")
        local RS = game:GetService("RunService")
        local conn
        _G.StickySetCarpetSpeed = function(enabled)
            _G.StickyCarpetSpeed = enabled and true or false
            _G.StickyCarpetSpeedActive = _G.StickyCarpetSpeed
            if conn then conn:Disconnect() conn = nil end
            if not _G.StickyCarpetSpeed then return end
            if _G.StickyEquipCarpet then task.spawn(function() pcall(_G.StickyEquipCarpet) end) end
            conn = RS.Heartbeat:Connect(function()
                if C:GetAttribute("Stealing") == true then
                    _G.StickySetCarpetSpeed(false)
                    return
                end
                local c = C.Character
                local hum = c and c:FindFirstChildOfClass("Humanoid")
                local part = c and (c:FindFirstChild("UpperTorso")
                    or c:FindFirstChild("Torso")
                    or c:FindFirstChild("HumanoidRootPart"))
                if not hum or not part then return end
                local engaging = _G.StickyCarpetEngaging and _G.StickyCarpetEngaging()
                if not engaging and _G.StickyEquipCarpet then pcall(_G.StickyEquipCarpet) end
                local spd = math.clamp(tonumber(_G.StickyCarpetSpeedValue) or 140, 20, 400)
                local md, keepY = hum.MoveDirection, part.Velocity.Y
                if md.Magnitude > 0 then
                    part.Velocity = Vector3.new(md.X * spd, keepY, md.Z * spd)
                else
                    part.Velocity = Vector3.new(0, keepY, 0)
                end
            end)
        end

        UIS.InputBegan:Connect(function(i, g)
            if g or i.UserInputType ~= Enum.UserInputType.Keyboard then return end
            if i.KeyCode.Name ~= (_G.StickyCarpetSpeedKeyName or "Q") then return end
            if C:GetAttribute("Stealing") == true then return end
            local on = not (_G.StickyCarpetSpeed == true)
            _G.StickySetCarpetSpeed(on)
            if on and _G.StickyEquipCarpet then
                task.spawn(function() pcall(_G.StickyEquipCarpet) end)
            end
            if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
        end)

        _G.StickyOnBoot(function()
            if i.CarpetSpeed == true then
                pcall(_G.StickySetCarpetSpeed, true)
            end
        end)
    end)()

    local playerESPEnabled = Config.PlayerESP == true
    local playerBillboards = {}
    local DANGER_TOOLS = {["Boogie Bomb"]=true,["Medusa's Head"]=true,["Body Swap Potion"]=true,["Laser Cape"]=true,["Rainbowrath Sword"]=true,["Gummy Bear"]=true}
    local function getHeldTool(p) local c=p.Character; if not c then return nil end; for _,o in ipairs(c:GetChildren()) do if o:IsA("Tool") then return o.Name end end; return nil end
    local function makePlayerBillboard(plr)
        local bb=Instance.new("BillboardGui"); bb.Name="PlayerESP_"..tostring(plr.UserId); bb.Size=UDim2.new(0,170,0,34)
        bb.StudsOffsetWorldSpace=Vector3.new(0,2.8,0); bb.AlwaysOnTop=true; bb.LightInfluence=0; bb.ResetOnSpawn=false
        local nameLbl=Instance.new("TextLabel",bb); nameLbl.Size=UDim2.new(1,0,0,18); nameLbl.BackgroundTransparency=1
        nameLbl.Font=Enum.Font.GothamBold; nameLbl.TextSize=14; nameLbl.TextColor3=Color3.fromRGB(255,255,255)
        nameLbl.TextStrokeTransparency=0.4; nameLbl.TextStrokeColor3=Color3.fromRGB(0,0,0); nameLbl.Text=plr.Name
        local toolLbl=Instance.new("TextLabel",bb); toolLbl.Name="ToolLabel"; toolLbl.Size=UDim2.new(1,0,0,13); toolLbl.Position=UDim2.new(0,0,0,18)
        toolLbl.BackgroundTransparency=1; toolLbl.Font=Enum.Font.GothamMedium; toolLbl.TextSize=11; toolLbl.TextColor3=Color3.fromRGB(100,220,255)
        toolLbl.TextStrokeTransparency=0.4; toolLbl.TextStrokeColor3=Color3.fromRGB(0,0,0); toolLbl.Text=getHeldTool(plr) or ""
        return bb,nameLbl
    end
    local function createOrRefreshPlayerESP(plr)
        if plr==LocalPlayer then return end; local hrp=plr.Character and plr.Character:FindFirstChild("HumanoidRootPart"); if not hrp then return end
        local hum=plr.Character:FindFirstChild("Humanoid"); if hum then hum.DisplayDistanceType=Enum.HumanoidDisplayDistanceType.None end
        local uid=plr.UserId; local entry=playerBillboards[uid]
        if not entry or not entry.bb or not entry.bb.Parent then
            if entry and entry.bb then pcall(function() entry.bb:Destroy() end) end
            local bb,nameLbl=makePlayerBillboard(plr); bb.Adornee=hrp; bb.Parent=hrp; playerBillboards[uid]={bb=bb,nameLbl=nameLbl,player=plr}
        elseif entry.bb.Adornee~=hrp then entry.bb.Adornee=hrp; entry.bb.Parent=hrp end
    end
    local function clearPlayerESP()
        for uid,entry in pairs(playerBillboards) do if entry.bb then pcall(entry.bb.Destroy,entry.bb) end; playerBillboards[uid]=nil end
    end
    _G.StickyOnBoot(function()
        while true do
            task.wait(playerESPEnabled and 0.5 or 2)
            if playerESPEnabled then
                for _,plr in ipairs(Players:GetPlayers()) do if plr~=LocalPlayer then pcall(createOrRefreshPlayerESP,plr) end end
                for uid,entry in pairs(playerBillboards) do if entry.bb and entry.bb.Parent then
                    pcall(function() local tl=entry.bb:FindFirstChild("ToolLabel"); if tl then local ht=getHeldTool(entry.player); tl.Text=ht or ""
                        if entry.nameLbl then entry.nameLbl.TextColor3=ht and DANGER_TOOLS[ht] and Color3.fromRGB(255,60,60) or Color3.fromRGB(255,255,255) end
                    end end)
                end end
            else clearPlayerESP() end
        end
    end)

    local function setPlayerESP(enabled)
        playerESPEnabled = enabled and true or false
        Config.PlayerESP = playerESPEnabled
        saveConfig()
        setToggle("PlayerESP", playerESPEnabled)
        setToggle("Player ESP", playerESPEnabled)
        if not playerESPEnabled then
            pcall(clearPlayerESP)
        end
    end

    _G.setPlayerESP = setPlayerESP

    _G.StickyOnBoot(function()
        if Config.XRay == true then
            pcall(setXRay, true)
        end
        setToggle("PlayerESP", playerESPEnabled)
    end)
end
do
    local config = i
    local saveConfig = U

    local function setToggle(name, enabled)
        local sync = _G.StickySyncToggleUI
        if type(sync) == "function" then
            pcall(sync, name, enabled)
        end
    end


    local function startNextBase()
        if _G.__NextBaseCleanup then pcall(_G.__NextBaseCleanup) end
        local CoreGui = game:GetService("CoreGui")
        local Plots = workspace:WaitForChild("Plots")
        local BASE_POSITIONS = {
            Vector3.new(-342.439, 10.399, 113.107),
            Vector3.new(-342.439, 10.465,   6.107),
            Vector3.new(-476.752, 10.465, 114.107),
            Vector3.new(-476.752, 10.465,   7.107),
            Vector3.new(-342.440, 10.464, 220.107),
            Vector3.new(-476.752, 10.465, 221.107),
            Vector3.new(-342.439, 10.465,-100.893),
            Vector3.new(-476.752, 10.465, -99.893),
        }
        local MATCH_TOL = 6
        local EMPTY_TEXT = "Empty Base"
        local ARROW = utf8.char(0x2B07)
        local function baseIndexFor(model)
            local ok, cf = pcall(function() return (model:GetBoundingBox()) end)
            if not ok then return nil end
            local p, bestI, bestD = cf.Position
            for i, bp in ipairs(BASE_POSITIONS) do
                local dx, dz = p.X - bp.X, p.Z - bp.Z
                local d = math.sqrt(dx * dx + dz * dz)
                if not bestD or d < bestD then bestI, bestD = i, d end
            end
            return (bestD and bestD <= MATCH_TOL) and bestI or nil
        end
        local bases = {}
        local connected = {}
        local conns = {}
        local anchor = Instance.new("Part")
        anchor.Name = "__NextBaseAnchor"
        anchor.Anchored, anchor.CanCollide, anchor.CanQuery, anchor.CanTouch = true, false, false, false
        anchor.Transparency = 1
        anchor.Size = Vector3.new(1, 1, 1)
        anchor.Parent = CoreGui
        local bb = Instance.new("BillboardGui")
        bb.Name = "NextBaseBillboard"
        bb.Adornee = anchor
        bb.Size = UDim2.fromScale(32, 13)
        bb.StudsOffset = Vector3.new(0, 10, 0)
        bb.MaxDistance = math.huge
        bb.AlwaysOnTop = true
        bb.LightInfluence = 0
        bb.Enabled = false
        bb.Parent = anchor
        local top = Instance.new("TextLabel", bb)
        top.BackgroundTransparency = 1
        top.AnchorPoint = Vector2.new(0.5, 0.5)
        top.Position = UDim2.fromScale(0.5, 0.30)
        top.Size = UDim2.fromScale(0.95, 0.50)
        top.Font = Enum.Font.GothamBlack
        top.Text = ARROW .. "  NEXT  " .. ARROW
        top.TextScaled = true
        top.TextColor3 = Color3.fromRGB(255, 60, 60)
        top.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        top.TextStrokeTransparency = 0
        local bottom = Instance.new("TextLabel", bb)
        bottom.BackgroundTransparency = 1
        bottom.AnchorPoint = Vector2.new(0.5, 0.5)
        bottom.Position = UDim2.fromScale(0.5, 0.72)
        bottom.Size = UDim2.fromScale(0.95, 0.42)
        bottom.Font = Enum.Font.GothamBlack
        bottom.Text = "EMPTY BASE"
        bottom.TextScaled = true
        bottom.TextColor3 = Color3.fromRGB(255, 255, 255)
        bottom.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        bottom.TextStrokeTransparency = 0
        local function isEmpty(label)
            return (label.Text:gsub("^%s+", ""):gsub("%s+$", "")) == EMPTY_TEXT
        end
        local function recompute()
            local targetIdx
            for i = 1, #BASE_POSITIONS do
                local b = bases[i]
                if b and b.label and isEmpty(b.label) then targetIdx = i break end
            end
            if targetIdx then
                anchor.CFrame = bases[targetIdx].cf
                bb.Enabled = true
            else
                bb.Enabled = false
            end
        end
        local function connectLabel(label)
            if connected[label] then return end
            connected[label] = true
            table.insert(conns, label:GetPropertyChangedSignal("Text"):Connect(recompute))
        end
        local function scan()
            for _, plot in ipairs(Plots:GetChildren()) do
                local sign  = plot:FindFirstChild("PlotSign")
                local model = sign and sign:FindFirstChild("Model")
                local gui   = sign and sign:FindFirstChild("SurfaceGui")
                local fr    = gui and gui:FindFirstChild("Frame")
                local label = fr and fr:FindFirstChild("TextLabel")
                if model and label then
                    local idx = baseIndexFor(model)
                    if idx then
                        bases[idx] = { label = label, cf = (select(1, model:GetBoundingBox())) }
                        connectLabel(label)
                    end
                end
            end
            recompute()
        end
        scan()
        table.insert(conns, Plots.DescendantAdded:Connect(function(d)
            if d:IsA("TextLabel") then task.defer(scan) end
        end))
        table.insert(conns, Plots.ChildAdded:Connect(function() task.defer(scan) end))
        _G.__NextBaseCleanup = function()
            for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
            if anchor then anchor:Destroy() end
            _G.__NextBaseCleanup = nil
        end
    end

    local function setNextBase(enabled)
        enabled = enabled and true or false
        config.nextBaseEnabled = enabled
        saveConfig()
        setToggle("NextBase", enabled)
        setToggle("Next Base", enabled)
        setToggle("Base Pointer", enabled)
        if enabled then
            task.spawn(function()
                pcall(startNextBase)
            end)
        elseif _G.__NextBaseCleanup then
            pcall(_G.__NextBaseCleanup)
        end
    end

    _G.setNextBase = setNextBase

    _G.StickyOnBoot(function()
        if config.nextBaseEnabled then
            pcall(setNextBase, true)
        end
    end)




    do
        local v2Enabled = false
        local v2Tracked = setmetatable({}, { __mode = "k" })
        local v2LastFire = setmetatable({}, { __mode = "k" })
        local v2LastEnableFire = setmetatable({}, { __mode = "k" })
        local v2LabelOf = setmetatable({}, { __mode = "k" })

        local V2_FIRE_DEBOUNCE = 0.08
        local V2_CYCLE_BURST = 1
        local V2_HOLD_LEAD = 0.05
        local V2_ENABLE_BURST, V2_ENABLE_COOLDOWN = 25, 0.08
        local V2_BURST_RANGE = 9.5
        local V2_BURST_COUNT, V2_BURST_DEBOUNCE = 5, 0.08
        local V2_SCAN_RATE = 0.05

        local V2State = {
            stealing = false, target = "", totalSteals = 0,
            lastResetAt = 0, cycle = 1.45, dip = 0.05,
        }
        _G.StickyV2State = V2State

        local v2NextFireAt = 0

        local function v2HRP()
            local char = C.Character
            return char and char:FindFirstChild("HumanoidRootPart")
        end

        local function v2PlotsFolder()
            return a:FindFirstChild("Plots")
        end

        local function v2PlotOf(inst)
            local folder = v2PlotsFolder()
            if not folder then return nil end
            local cur = inst
            while cur do
                if cur.Parent == folder then return cur end
                cur = cur.Parent
            end
            return nil
        end

        local v2MyPlotCache, v2MyPlotCacheAt = {}, 0
        local function v2IsMyPlot(plot)
            if not plot then return false end
            local now = tick()
            if now - v2MyPlotCacheAt > 3 then
                v2MyPlotCache = {}
                v2MyPlotCacheAt = now
            end
            local hit = v2MyPlotCache[plot.Name]
            if hit ~= nil then return hit end
            local result = false
            local sign = plot:FindFirstChild("PlotSign")
            if sign then
                local yb = sign:FindFirstChild("YourBase")
                if yb and yb:IsA("BillboardGui") then result = yb.Enabled == true end
                if not result then
                    local gui = sign:FindFirstChildWhichIsA("SurfaceGui", true)
                    local label = gui and gui:FindFirstChildWhichIsA("TextLabel", true)
                    if label and type(label.Text) == "string" then
                        local txt = label.Text:lower()
                        if txt:find(C.Name:lower(), 1, true)
                        or txt:find(C.DisplayName:lower(), 1, true) then
                            result = true
                        end
                    end
                end
            end
            v2MyPlotCache[plot.Name] = result
            return result
        end

        local function v2PromptPosition(prompt)
            local p = prompt.Parent
            if not p then return nil end
            if p:IsA("Attachment") then
                local ok, wp = pcall(function() return p.WorldPosition end)
                if ok and typeof(wp) == "Vector3" then return wp end
                p = p.Parent
            end
            if not p then return nil end
            if p:IsA("BasePart") then return p.Position end
            if p:IsA("Model") then
                local ok, pivot = pcall(function() return p:GetPivot().Position end)
                if ok then return pivot end
            end
            return nil
        end

        local function v2IsPromptAvailable(prompt)
            if not prompt or not prompt.Parent then return false end
            if not prompt.Enabled then return false end
            if C:GetAttribute("Stealing") == true then return false end
            local plot = v2PlotOf(prompt)
            if plot and v2IsMyPlot(plot) then return false end
            return true
        end

        local function v2CanFire(prompt, debounce)
            local t = os.clock()
            local last = v2LastFire[prompt]
            if last and (t - last) < debounce then return false end
            v2LastFire[prompt] = t
            return true
        end

        local function v2BeginHold(prompt)
            pcall(function()
                if typeof(firesignal) == "function" then
                    pcall(firesignal, prompt.PromptButtonHoldBegan)
                end
            end)
            pcall(function() prompt:InputHoldBegin() end)
        end

        local function v2FirePrompt(prompt, burst, debounce)
            if not prompt or not prompt.Parent then return end
            if not prompt.Enabled then return end
            local now = tick()
            if now < v2NextFireAt then return end
            if not v2CanFire(prompt, debounce) then return end
            v2BeginHold(prompt)
            task.delay(V2_HOLD_LEAD, function()
                if not prompt.Parent or not prompt.Enabled then return end
                for _ = 1, burst do
                    pcall(function()
                        if typeof(fireproximityprompt) == "function" then
                            fireproximityprompt(prompt, 1)
                        elseif typeof(firesignal) == "function" then
                            pcall(firesignal, prompt.Triggered)
                        end
                    end)
                end
            end)
            V2State.totalSteals = V2State.totalSteals + 1
            v2NextFireAt = now + V2State.cycle
            V2State.lastResetAt = now
        end

        local v2LastBurst = setmetatable({}, { __mode = "k" })
        local function v2BurstAt(prompt)
            if not prompt or not prompt.Parent then return end
            if not prompt.Enabled then return end
            local t = os.clock()
            local last = v2LastBurst[prompt]
            if last and (t - last) < V2_BURST_DEBOUNCE then return end
            v2LastBurst[prompt] = t
            for _ = 1, V2_BURST_COUNT do
                pcall(function()
                    if typeof(fireproximityprompt) == "function" then
                        fireproximityprompt(prompt, 1)
                    elseif typeof(firesignal) == "function" then
                        pcall(firesignal, prompt.Triggered)
                    end
                end)
            end
        end

        local function v2NameOf(prompt)
            local hit = v2LabelOf[prompt]
            if hit then return hit end
            local nm = ""
            pcall(function()
                local ot = prompt.ObjectText
                if type(ot) == "string" then nm = ot end
            end)
            if nm == "" then
                local cur = prompt.Parent
                while cur do
                    if cur:IsA("Model") and cur.Parent and cur.Parent.Name == "AnimalPodiums" then
                        nm = cur.Name
                        break
                    end
                    cur = cur.Parent
                end
                if nm == "" then nm = "Brainrot" end
            end
            v2LabelOf[prompt] = nm
            return nm
        end

        local function v2TrackPrompt(prompt)
            if v2Tracked[prompt] then return end
            v2Tracked[prompt] = true

            local function tryEnableFire()
                if not v2Enabled then return end
                if not v2HRP() then return end
                if not v2IsPromptAvailable(prompt) then return end
                local now = os.clock()
                local le = v2LastEnableFire[prompt]
                if not le or (now - le) >= V2_ENABLE_COOLDOWN then
                    v2LastEnableFire[prompt] = now
                    v2FirePrompt(prompt, V2_ENABLE_BURST, V2_ENABLE_COOLDOWN)
                end
            end

            task.defer(tryEnableFire)

            pcall(function()
                prompt:GetPropertyChangedSignal("Enabled"):Connect(function()
                    if prompt.Enabled then tryEnableFire() end
                end)
            end)

            prompt.AncestryChanged:Connect(function()
                if not prompt:IsDescendantOf(a) then
                    v2Tracked[prompt] = nil
                    v2LastFire[prompt] = nil
                    v2LastEnableFire[prompt] = nil
                end
            end)
        end

        local function v2ScanPrompts()
            local folder = v2PlotsFolder()
            if not folder then return end
            for _, plot in ipairs(folder:GetChildren()) do
                local podiums = plot:FindFirstChild("AnimalPodiums")
                if podiums then
                    for _, obj in ipairs(podiums:GetDescendants()) do
                        if obj:IsA("ProximityPrompt") then v2TrackPrompt(obj) end
                    end
                end
            end
        end

        a.DescendantAdded:Connect(function(obj)
            if obj:IsA("ProximityPrompt") and obj:FindFirstAncestor("AnimalPodiums") then
                v2TrackPrompt(obj)
            end
        end)

        task.spawn(function()
            while true do
                task.wait(V2_SCAN_RATE)
                if not v2Enabled then
                    V2State.stealing = false
                    V2State.target = ""
                elseif C:GetAttribute("Stealing") == true then
                    V2State.stealing = false
                    V2State.target = ""
                else
                    if not v2HRP() then
                        V2State.stealing = false
                        V2State.target = ""
                    else
                        local pick = _G.StickyV2TargetPrompt
                        if not pick or not pick.Parent or not pick.Enabled then
                            local hrpPos = v2HRP().Position
                            local bestD = math.huge
                            pick = nil
                            for prompt in pairs(v2Tracked) do
                                if v2IsPromptAvailable(prompt) then
                                    local pos = v2PromptPosition(prompt)
                                    local d = pos and (pos - hrpPos).Magnitude or math.huge
                                    if d < bestD then pick, bestD = prompt, d end
                                end
                            end
                        end
                        if pick and pick.Parent and pick.Enabled then
                            v2TrackPrompt(pick)
                            V2State.stealing = true
                            V2State.target = v2NameOf(pick)
                            v2FirePrompt(pick, V2_CYCLE_BURST, V2_FIRE_DEBOUNCE)
                            local pos = v2PromptPosition(pick)
                            local hrpPos = v2HRP() and v2HRP().Position
                            if pos and hrpPos then
                                local dist = (pos - hrpPos).Magnitude
                                if dist <= V2_BURST_RANGE then
                                    v2BurstAt(pick)
                                end
                            end
                        else
                            V2State.stealing = false
                            V2State.target = ""
                        end
                    end
                end
            end
        end)

        local function setInstantStealV2(on)
            on = on and true or false
            v2Enabled = on
            config.InstantStealV2 = on
            if on then
                pcall(v2ScanPrompts)
            else
                V2State.stealing = false
                V2State.target = ""
            end
            saveConfig()
            setToggle("InstantStealV2", on)
            setToggle("Instant Steal V2", on)
        end

        _G.setInstantStealV2 = setInstantStealV2

        _G.StickyOnBoot(function()
            if config.InstantStealV2 then
                pcall(setInstantStealV2, true)
            end
        end)
    end

    local function startPodiumESP()
        if _G.__PodiumESPCleanup then pcall(_G.__PodiumESPCleanup) end
        local RS = game:GetService("RunService")
        local hui = (gethui and gethui()) or game:GetService("CoreGui")
        local Plots = workspace:FindFirstChild("Plots")
        if not Plots then return end
        local function cam() return workspace.CurrentCamera end
        if not cam() then return end

        local MARK = Color3.fromRGB(255, 60, 60)
        local WHITE = Color3.fromRGB(255, 255, 255)
        local SHOW_NUMBERS = true
        local COLLIDE = true
        local FILL_T, INNER_T, THICK = 0.55, 0.42, 0.05
        local PULSE_SPEED, PULSE_AMOUNT = 2, 0.12
        local OUTER = Vector3.new(6, 0.25, 6)
        local INNER_SZ = Vector3.new(4, 0.25, 4)
        local INNER_UP = 0.25

        local TEMPLATE = {
            { 18.500,  1.531, -14.476,  90}, { 18.500,  1.531,  -6.976,  90}, { 18.500,  1.531,   0.524,  90},
            { 18.500,  1.531,   8.024,  90}, { 18.500,  1.531,  15.524,  90},
            {-18.536,  1.531,  15.524, -90}, {-18.536,  1.531,   8.024, -90}, {-18.536,  1.531,   0.524, -90},
            {-18.536,  1.531,  -6.976, -90}, {-18.536,  1.531, -14.476, -90},
            { 18.500, 19.531, -14.476,  90}, { 18.500, 19.531,  -6.976,  90}, { 18.500, 19.531,   0.524,  90},
            { 18.500, 19.531,   8.024,  90}, { 18.500, 19.531,  15.524,  90},
            {-18.380, 19.531, -14.452, -90}, {-18.380, 19.531,  -6.952, -90}, {-18.380, 19.531,   0.548, -90},
            { 18.500, 36.531, -12.476,  90}, { 18.500, 36.531,  -4.976,  90}, { 18.500, 36.531,   2.524,  90},
            { 18.500, 36.531,  10.024,  90}, { 18.500, 36.531,  17.524,  90},
            {-18.472, 36.531, -12.501, -90}, {-18.471, 36.531,  -5.001, -90}, {-18.471, 36.531,   2.499, -90},
            {-18.471, 36.531,   9.999, -90}, {-18.471, 36.531,  17.499, -90},
        }

        local _rng = Random.new(os.clock() * 1e6)
        local _POOL = { "Part", "Mesh", "MeshPart", "Union", "Wedge", "Cylinder", "Model", "Frame", "Handle", "Body", "Root" }
        local function fakeName() return _POOL[_rng:NextInteger(1, #_POOL)] end

        local markers, fills, conns = {}, {}, {}
        local collideParts, labelAnchors, labelsByPlot = {}, {}, {}
        local currentPlot, alive, pending = nil, true, false

        local function keep(x) markers[#markers + 1] = x; x.Parent = hui; return x end

        local function clear()
            for _, m in ipairs(markers) do pcall(function() m:Destroy() end) end
            table.clear(markers); table.clear(fills)
            for _, p in ipairs(collideParts) do pcall(function() p:Destroy() end) end
            table.clear(collideParts)
            for _, p in ipairs(labelAnchors) do pcall(function() p:Destroy() end) end
            table.clear(labelAnchors); table.clear(labelsByPlot)
            currentPlot = nil
        end

        local function box(adornee, cf, size, color, trans, isFill)
            local a = Instance.new("BoxHandleAdornment")
            a.Adornee = adornee
            a.Size = size
            a.CFrame = cf
            a.Color3 = color
            a.Transparency = trans
            a.AlwaysOnTop = true
            a.ZIndex = 0
            keep(a)
            if isFill then fills[#fills + 1] = { a = a, base = trans } end
            return a
        end

        local function edges(root, cf, size, color)
            local t = THICK * 1.6
            local hx, hz, y = size.X * 0.5, size.Z * 0.5, size.Y * 0.5
            box(root, cf * CFrame.new(0, y,  hz), Vector3.new(size.X, t, t), color, 0)
            box(root, cf * CFrame.new(0, y, -hz), Vector3.new(size.X, t, t), color, 0)
            box(root, cf * CFrame.new( hx, y, 0), Vector3.new(t, t, size.Z), color, 0)
            box(root, cf * CFrame.new(-hx, y, 0), Vector3.new(t, t, size.Z), color, 0)
        end

        local function anchorAt(worldCF, collidable)
            local c = cam()
            if not c then return nil end
            local a = Instance.new("Part")
            a.Name = fakeName()
            a.Anchored = true
            a.CanCollide = collidable and true or false
            a.CanQuery = false
            a.CanTouch = false
            a.CastShadow = false
            a.Massless = true
            a.Transparency = 1
            a.Size = collidable and OUTER or Vector3.new(0.1, 0.1, 0.1)
            a.CFrame = worldCF
            a.Parent = c
            if collidable then
                collideParts[#collideParts + 1] = a
            else
                labelAnchors[#labelAnchors + 1] = a
            end
            return a
        end

        local function labelFor(plot, adornee, slotNum, color)
            if not SHOW_NUMBERS then return end
            local bg = Instance.new("BillboardGui")
            bg.Adornee = adornee
            bg.Size = UDim2.new(2.2, 20, 1.35, 12)
            bg.StudsOffset = Vector3.new(0, 3.2, 0)
            bg.AlwaysOnTop = true
            bg.LightInfluence = 0
            bg.MaxDistance = 400
            bg.Active = true
            bg.Enabled = false

            local panel = Instance.new("Frame", bg)
            panel.Size = UDim2.fromScale(1, 1)
            panel.BackgroundColor3 = Color3.fromRGB(8, 8, 12)
            panel.BackgroundTransparency = 0.55
            panel.BorderSizePixel = 0
            Instance.new("UICorner", panel).CornerRadius = UDim.new(0.3, 0)

            local glow = Instance.new("UIStroke", panel)
            glow.Color = color
            glow.Thickness = 4
            glow.Transparency = 0.7
            glow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

            local num = Instance.new("TextButton", panel)
            num.Size = UDim2.fromScale(1, 1)
            num.BackgroundTransparency = 1
            num.Text = tostring(slotNum)
            num.Font = Enum.Font.GothamBlack
            num.TextScaled = true
            num.TextColor3 = WHITE
            num.AutoButtonColor = false

            local pad = Instance.new("UIPadding", num)
            pad.PaddingLeft = UDim.new(0.1, 0)
            pad.PaddingRight = UDim.new(0.1, 0)
            pad.PaddingTop = UDim.new(0.08, 0)
            pad.PaddingBottom = UDim.new(0.08, 0)

            local cons = Instance.new("UITextSizeConstraint", num)
            cons.MaxTextSize = 500
            cons.MinTextSize = 6

            local ns = Instance.new("UIStroke", num)
            ns.Color = color
            ns.Thickness = 2
            ns.LineJoinMode = Enum.LineJoinMode.Round

            keep(bg)
            labelsByPlot[plot] = labelsByPlot[plot] or {}
            table.insert(labelsByPlot[plot], bg)
        end

        local function build()
            if not alive then return end
            clear()
            for _, plot in ipairs(Plots:GetChildren()) do
                local root = plot:FindFirstChild("MainRoot")
                if root then
                    for idx = 1, #TEMPLATE do
                        local e = TEMPLATE[idx]
                        local worldCF = root.CFrame * (CFrame.new(e[1], e[2], e[3]) * CFrame.Angles(0, math.rad(e[4]), 0))
                        local a = anchorAt(worldCF, false)
                        if a then
                            box(a, CFrame.new(), OUTER, MARK, FILL_T, true)
                            box(a, CFrame.new(0, INNER_UP, 0), INNER_SZ, MARK, INNER_T, true)
                            edges(a, CFrame.new(), OUTER, MARK)
                            if COLLIDE then anchorAt(worldCF, true) end
                            labelFor(plot, a, idx, MARK)
                        end
                    end
                end
            end
        end

        local function findCurrentPlot()
            local char = C.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if not hrp then return nil end
            local pos = hrp.Position
            local best, bestDist = nil, math.huge
            for _, plot in ipairs(Plots:GetChildren()) do
                local root = plot:FindFirstChild("MainRoot")
                if root then
                    local d = (root.Position - pos).Magnitude
                    if d < bestDist then best, bestDist = plot, d end
                end
            end
            if bestDist > 100 then return nil end
            return best
        end

        local function updateVisibility()
            local newPlot = findCurrentPlot()
            if newPlot == currentPlot then return end
            currentPlot = newPlot
            for plot, list in pairs(labelsByPlot) do
                local show = (plot == currentPlot)
                for _, bg in ipairs(list) do
                    if bg and bg.Parent then bg.Enabled = show end
                end
            end
        end

        local function rebuild()
            if pending or not alive then return end
            pending = true
            task.delay(0.4, function()
                pending = false
                if not alive then return end
                pcall(build)
                pcall(updateVisibility)
            end)
        end

        build()
        updateVisibility()

        local function watch(plot)
            if not plot:FindFirstChild("MainRoot") then
                local root = plot:WaitForChild("MainRoot", 30)
                if root and alive then rebuild() end
            end
        end
        for _, plot in ipairs(Plots:GetChildren()) do task.spawn(watch, plot) end

        table.insert(conns, Plots.ChildAdded:Connect(function(plot)
            task.spawn(watch, plot)
            rebuild()
        end))
        table.insert(conns, workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
            if alive then rebuild() end
        end))

        local acc, visAcc = 0, 0
        table.insert(conns, RS.Heartbeat:Connect(function(dt)
            acc, visAcc = acc + dt, visAcc + dt
            if acc >= 0.05 then
                acc = 0
                local w = math.sin(os.clock() * PULSE_SPEED) * PULSE_AMOUNT
                for _, f in ipairs(fills) do
                    if f.a.Parent then f.a.Transparency = math.clamp(f.base + w, 0, 1) end
                end
            end
            if visAcc >= 0.5 then
                visAcc = 0
                updateVisibility()
            end
        end))

        _G.__PodiumESPCleanup = function()
            alive = false
            for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
            table.clear(conns)
            clear()
            _G.__PodiumESPCleanup = nil
        end
    end

    local function setPodiumESP(enabled)
        enabled = enabled and true or false
        config.podiumESP = enabled
        saveConfig()
        setToggle("PodiumESP", enabled)
        setToggle("Podium ESP", enabled)
        if enabled then
            task.spawn(function() pcall(startPodiumESP) end)
        elseif _G.__PodiumESPCleanup then
            pcall(_G.__PodiumESPCleanup)
        end
    end

    _G.setPodiumESP = setPodiumESP

    _G.StickyOnBoot(function()
        if config.podiumESP then
            pcall(setPodiumESP, true)
        end
    end)

    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Workspace = game:GetService("Workspace")
    local LocalPlayer = C

    local function _P(o, ...)
        for n = 1, select("#", ...), 2 do
            o[select(n, ...)] = select(n + 1, ...)
        end
        return o
    end

    local JUI = {}
    function JUI.new(cls, parent, ...)
        local o = Instance.new(cls)
        _P(o, ...)
        if parent then o.Parent = parent end
        return o
    end
    function JUI.corner(o, r)
        local c = Instance.new("UICorner", o)
        c.CornerRadius = UDim.new(0, r or 4)
        return o
    end
    function JUI.stroke(o, color, thickness, transparency)
        local s = Instance.new("UIStroke", o)
        s.Color = color
        s.Thickness = thickness or 1
        s.Transparency = transparency or 0
        return s
    end

    local hazardESPData = {}
    local hazardLoopStarted = false
    local sfind = string.find
    local TRAP_PATS = { "trap", "mine", "hive" }
    local HESP_KINDS = {
        turret = { color = Color3.fromRGB(255, 40, 40),  text = "TURRET" },
        trap   = { color = Color3.fromRGB(255, 150, 20), text = "TRAP" },
        mine   = { color = Color3.fromRGB(167, 142, 255), text = "SUBSPACE MINE" },
    }

    local function hazardKind(name)
        local ln = name:lower()
        if sfind(ln, "sentrybullet", 1, true) then return nil end
        if sfind(ln, "tripmine", 1, true) then return "mine" end
        if sfind(ln, "sentry", 1, true) then return "turret" end
        for _, p in ipairs(TRAP_PATS) do
            if sfind(ln, p, 1, true) then return "trap" end
        end
        return nil
    end

    local function createHazardESP(target, kind)
        local spec = HESP_KINDS[kind]
        local labelText = "! " .. spec.text .. " !"
        if kind == "mine" then
            local owner = target.Name:match("SubspaceTripmine(.+)")
            if owner then
                local pl = Players:FindFirstChild(owner)
                labelText = "! " .. ((pl and pl.DisplayName) or owner) .. "'s MINE !"
            end
        end
        local hl
        if target:IsA("Model") then
            hl = Instance.new("Highlight")
            _P(hl, "Name", "Sticky_HazardESP_HL", "Adornee", target,
                "FillColor", spec.color, "FillTransparency", 0.25,
                "OutlineColor", Color3.fromRGB(255, 255, 255), "OutlineTransparency", 0)
            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            hl.Parent = target
        else
            hl = Instance.new("SelectionBox")
            _P(hl, "Name", "Sticky_HazardESP_HL", "Adornee", target,
                "Color3", spec.color, "LineThickness", 0.12,
                "SurfaceColor3", spec.color, "SurfaceTransparency", 0.6, "Parent", target)
        end
        local bbAd = (target:IsA("BasePart") and target)
            or target:FindFirstChildWhichIsA("BasePart", true)
            or target
        local bb = Instance.new("BillboardGui")
        _P(bb, "Name", "Sticky_HazardESP_Label", "Adornee", bbAd,
            "Size", UDim2.new(0, 200, 0, 44), "StudsOffset", Vector3.new(0, 6, 0),
            "AlwaysOnTop", true, "Parent", target)
        JUI.new("TextLabel", bb,
            "Size", UDim2.new(1, 0, 1, 0), "BackgroundTransparency", 1,
            "Text", labelText, "TextColor3", spec.color,
            "TextStrokeColor3", Color3.fromRGB(0, 0, 0), "TextStrokeTransparency", 0,
            "Font", Enum.Font.GothamBold, "TextScaled", true)
        return { hl = hl, bb = bb, kind = kind }
    end

    local function destroyHazardEntry(obj, data)
        if data.hl and data.hl.Parent then pcall(function() data.hl:Destroy() end) end
        if data.bb and data.bb.Parent then pcall(function() data.bb:Destroy() end) end
        hazardESPData[obj] = nil
    end

    local function refreshHazardESP()
        local kindOn = {
            turret = config.TurretESP == true,
            trap   = config.TrapESP == true,
            mine   = config.TrapESP == true,
        }
        local current = {}
        if kindOn.turret or kindOn.trap or kindOn.mine then
            local hz = workspace:GetDescendants()
            for hi = 1, #hz do
                local inst = hz[hi]
                local cn = inst.ClassName
                if cn == "Model" or inst:IsA("BasePart") then
                    local kind = hazardKind(inst.Name)
                    if kind and kindOn[kind] then
                        local par, skip = inst, false
                        if inst:FindFirstChildWhichIsA("Humanoid", true) then skip = true end
                        while not skip and par and par ~= workspace do
                            if Players:GetPlayerFromCharacter(par)
                                or (par.ClassName == "Model" and par:FindFirstChildOfClass("Humanoid")) then
                                skip = true; break
                            end
                            if par ~= inst and hazardKind(par.Name) then skip = true; break end
                            par = par.Parent
                        end
                        if not skip then
                            current[inst] = true
                            local data = hazardESPData[inst]
                            if data and data.kind ~= kind then
                                destroyHazardEntry(inst, data); data = nil
                            end
                            if not data then
                                local ok, res = pcall(createHazardESP, inst, kind)
                                if ok then hazardESPData[inst] = res end
                            end
                        end
                    end
                end
            end
        end
        for obj, data in pairs(hazardESPData) do
            if not current[obj] or not obj.Parent then
                destroyHazardEntry(obj, data)
            end
        end
    end

    local function startHazardLoop()
        if hazardLoopStarted then return end
        hazardLoopStarted = true
        task.spawn(function()
            while true do
                task.wait(tonumber(_G.StickyHazardESPEvery) or 1.5)
                if config.TurretESP == true or config.TrapESP == true then
                    pcall(refreshHazardESP)
                elseif next(hazardESPData) ~= nil then
                    pcall(refreshHazardESP)
                end
            end
        end)
    end

    local function setTurretESP(enabled)
        enabled = enabled and true or false
        config.TurretESP = enabled
        saveConfig()
        setToggle("TurretESP", enabled)
        setToggle("Turret ESP", enabled)
        if enabled then startHazardLoop() end
        task.spawn(function() pcall(refreshHazardESP) end)
    end

    local function setTrapESP(enabled)
        enabled = enabled and true or false
        config.TrapESP = enabled
        saveConfig()
        setToggle("TrapESP", enabled)
        setToggle("Trap ESP", enabled)
        if enabled then startHazardLoop() end
        task.spawn(function() pcall(refreshHazardESP) end)
    end

    _G.setTurretESP = setTurretESP
    _G.setTrapESP = setTrapESP

    _G.StickyOnBoot(function()
        if config.TurretESP == true or config.TrapESP == true then
            startHazardLoop()
        end
    end)

    local ESP_OFFSET = Vector3.new(0, 3.2, 0)
    local MIN_ESP_GEN_DEFAULT = 10000000
    local GEN_SUFFIX = {}
    do
        local order = { "", "K", "M", "B", "T", "Qa", "Qi", "Sx", "Sp", "Oc", "No", "Dc",
            "Ud", "Dd", "Td", "Qad", "Qid", "Sxd", "Spd", "Ocd", "Nod", "Vg", "Uvg", "Dvg", "Tvg" }
        for idx, suf in ipairs(order) do
            GEN_SUFFIX[string.lower(suf)] = 1000 ^ (idx - 1)
        end
    end

    local function parseGenText(text)
        if type(text) ~= "string" then return nil end
        local num, suf = text:match("([%d%.]+)%s*(%a*)")
        local base = tonumber(num)
        if not base then return nil end
        local mult = GEN_SUFFIX[string.lower(suf or "")]
        return base * (mult or 1)
    end

    local brainrotGui = nil
    local brainrotCards = {}
    local brainrotConns = {}
    local brainrotOn = false

    local function brainrotRoot()
        if brainrotGui and brainrotGui.Parent then return brainrotGui end
        brainrotGui = Instance.new("ScreenGui")
        brainrotGui.Name = "BrainrotESP"
        brainrotGui.ResetOnSpawn = false
        brainrotGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        brainrotGui.DisplayOrder = 95
        local ok = pcall(function()
            brainrotGui.Parent = (gethui and gethui()) or game:GetService("CoreGui")
        end)
        if not ok then
            brainrotGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
        end
        return brainrotGui
    end

    local function removeCard(key)
        local card = brainrotCards[key]
        if not card then return end
        if card.billboard then pcall(function() card.billboard:Destroy() end) end
        brainrotCards[key] = nil
    end

    local function clearCards()
        for key in pairs(brainrotCards) do removeCard(key) end
    end

    local gameIconTables = nil
    local function loadGameIcons()
        if gameIconTables then return gameIconTables end
        local out = { mutations = {}, traits = {} }
        pcall(function()
            local datas = game:GetService("ReplicatedStorage"):FindFirstChild("Datas")
            if not datas then return end
            for field, folder in pairs({ mutations = "Mutations", traits = "Traits" }) do
                local mod = datas:FindFirstChild(folder)
                if mod then
                    local data = require(mod)
                    if type(data) == "table" then
                        for name, entry in pairs(data) do
                            if type(entry) == "table" and entry.Icon then
                                out[field][name] = entry.Icon
                            end
                        end
                    end
                end
            end
        end)
        gameIconTables = out
        return out
    end

    local function assetUrl(v)
        if v == nil then return nil end
        local str = tostring(v)
        if str == "" then return nil end
        if str:match("^%d+$") then str = "rbxassetid://" .. str end
        return str
    end

    local function petBadgeNames(pet)
        local muts, traits = {}, {}
        if type(pet) ~= "table" then return muts, traits end
        local mut = pet.mutation
        if type(mut) == "string" and mut ~= "" and mut ~= "None" and mut ~= "N/A" then
            muts[#muts + 1] = mut
        end
        local tr = pet.traits
        if tr == nil and type(pet.animalData) == "table" then tr = pet.animalData.traits end
        if type(tr) == "string" and tr ~= "" and tr ~= "None" then
            for name in tr:gmatch("[^,]+") do
                local trimmed = name:match("^%s*(.-)%s*$")
                if trimmed ~= "" and trimmed ~= "None" then traits[#traits + 1] = trimmed end
            end
        elseif type(tr) == "table" then
            for _, tv in ipairs(tr) do
                local n = (type(tv) == "string" and tv)
                    or (type(tv) == "table" and (tv.Name or tv.name or tv.Trait or tv.Id))
                if type(n) == "string" and n ~= "" then traits[#traits + 1] = n end
            end
        end
        return muts, traits
    end

    local function petBadgeIcons(pet)
        local ids = {}
        local tabs = loadGameIcons()
        local muts, traits = petBadgeNames(pet)
        for _, name in ipairs(muts) do
            local ic = assetUrl(tabs.mutations[name])
            if ic then ids[#ids + 1] = ic end
        end
        for _, name in ipairs(traits) do
            local ic = assetUrl(tabs.traits[name])
            if ic then ids[#ids + 1] = ic end
        end
        return ids
    end

    P.PetBadgeIcons = petBadgeIcons
    P.PetBadgeNames = petBadgeNames

    local animalByUid = {}
    local badgeIconsFor  -- assigned below, once the index exists

    local ICON_DIR = "PublicTP"
    local ICON_URLS = {
        "https://stealabrainrot.fandom.com/wiki/Special:FilePath/%s.png",
        "https://stealabrainrot.fandom.com/wiki/Special:FilePath/%s.jpg",
        "https://cdn.lura.blue/sab/%s.png",
    }
    local iconCache = {}
    local animalIcons = nil

    local function animalIconFor(displayName)
        if animalIcons == nil then
            animalIcons = {}
            pcall(function()
                local datas = game:GetService("ReplicatedStorage"):FindFirstChild("Datas")
                local mod = datas and datas:FindFirstChild("Animals")
                if not mod then return end
                local data = require(mod)
                if type(data) ~= "table" then return end
                for id, entry in pairs(data) do
                    if type(entry) == "table" then
                        local ic = assetUrl(entry.Icon or entry.Image or entry.Thumbnail)
                        if ic then
                            animalIcons[string.lower(tostring(id))] = ic
                            if entry.DisplayName then
                                animalIcons[string.lower(tostring(entry.DisplayName))] = ic
                            end
                        end
                    end
                end
            end)
        end
        return animalIcons[string.lower(tostring(displayName))]
    end

    local function ensureIconDir()
        if typeof(isfolder) ~= "function" or typeof(makefolder) ~= "function" then
            return false
        end
        local ok, exists = pcall(isfolder, ICON_DIR)
        if ok and exists then return true end
        return (pcall(makefolder, ICON_DIR))
    end

    local function looksLikeImage(body)
        return type(body) == "string" and #body > 200
            and body:sub(1, 1) ~= "{" and body:sub(1, 1) ~= "<"
    end

    local iconPending = {}

    local function iconFor(name)
        if not name or name == "" then return nil end
        local key = tostring(name):gsub(" ", "_")
        if iconCache[key] then return iconCache[key] end
        local builtin = animalIconFor(name)
        if builtin then
            iconCache[key] = builtin
            return builtin
        end
        if typeof(writefile) ~= "function" or typeof(isfile) ~= "function"
            or typeof(getcustomasset) ~= "function" then
            return nil
        end
        if iconPending[key] then return nil end
        iconPending[key] = true
        task.spawn(function()
            ensureIconDir()
            local path = ICON_DIR .. "/PublicTP_pet_" .. key:gsub("[^%w_%-]", "") .. ".png"
            local ok, exists = pcall(isfile, path)
            if ok and exists then
                local ok2, asset = pcall(getcustomasset, path)
                if ok2 and asset then iconCache[key] = asset end
                iconPending[key] = nil
                return
            end
            local encoded = key:gsub("[^%w_%-%.]", function(ch)
                return string.format("%%%02X", string.byte(ch))
            end)
            for _, template in ipairs(ICON_URLS) do
                local ok2, body = pcall(function()
                    return game:HttpGet(string.format(template, encoded))
                end)
                if ok2 and looksLikeImage(body) and pcall(writefile, path, body) then
                    local ok3, asset = pcall(getcustomasset, path)
                    if ok3 and asset then
                        iconCache[key] = asset
                        break
                    end
                end
            end
            iconPending[key] = nil
        end)
        return nil
    end

    local function makeCard(adornee, nameText, genText, key)
        local bb = Instance.new("BillboardGui")
        bb.Name = "BrainrotESP_" .. key
        bb.Adornee = adornee
        bb.AlwaysOnTop = true
        bb.Size = UDim2.new(0, 168, 0, 60)
        bb.StudsOffset = ESP_OFFSET
        bb.LightInfluence = 0
        bb.MaxDistance = 3000
        bb.Parent = brainrotRoot()

        local card = Instance.new("Frame", bb)
        card.Name = "Card"
        card.Size = UDim2.new(1, 0, 1, 0)
        card.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
        card.BackgroundTransparency = 0.12
        card.BorderSizePixel = 0
        Instance.new("UICorner", card).CornerRadius = UDim.new(0, 7)
        local stroke = Instance.new("UIStroke", card)
        stroke.Color = Color3.fromRGB(255, 215, 60)
        stroke.Thickness = 1
        stroke.Transparency = 0.25
        stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

        local nameLbl = Instance.new("TextLabel", card)
        nameLbl.Name = "NameLbl"
        nameLbl.Size = UDim2.new(1, -50, 0, 18)
        nameLbl.Position = UDim2.new(0, 7, 0, 4)
        nameLbl.BackgroundTransparency = 1
        nameLbl.Text = nameText
        nameLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLbl.Font = Enum.Font.GothamBlack
        nameLbl.TextScaled = true
        nameLbl.TextXAlignment = Enum.TextXAlignment.Left
        nameLbl.TextStrokeTransparency = 0
        nameLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)

        local genLbl = Instance.new("TextLabel", card)
        genLbl.Name = "GenLbl"
        genLbl.Size = UDim2.new(1, -50, 0, 16)
        genLbl.Position = UDim2.new(0, 7, 0, 22)
        genLbl.BackgroundTransparency = 1
        genLbl.Text = genText
        genLbl.TextColor3 = Color3.fromRGB(90, 255, 120)
        genLbl.Font = Enum.Font.GothamBold
        genLbl.TextScaled = true
        genLbl.TextXAlignment = Enum.TextXAlignment.Left
        genLbl.TextStrokeTransparency = 0
        genLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)

        local imgHolder = Instance.new("Frame", card)
        imgHolder.Name = "ImgHolder"
        imgHolder.Size = UDim2.new(0, 36, 0, 36)
        imgHolder.Position = UDim2.new(1, -40, 0.5, -18)
        imgHolder.BackgroundColor3 = Color3.fromRGB(30, 30, 36)
        imgHolder.BackgroundTransparency = 0.2
        imgHolder.BorderSizePixel = 0
        Instance.new("UICorner", imgHolder).CornerRadius = UDim.new(0, 6)

        local img = Instance.new("ImageLabel", imgHolder)
        img.Name = "Img"
        img.Size = UDim2.new(1, -4, 1, -4)
        img.Position = UDim2.fromOffset(2, 2)
        img.BackgroundTransparency = 1
        img.ScaleType = Enum.ScaleType.Fit
        img.Image = iconFor(nameText) or ""

        local badges = {}
        for slot = 1, 4 do
            local badge = Instance.new("ImageLabel", card)
            badge.Name = "Badge" .. slot
            badge.Size = UDim2.fromOffset(14, 14)
            badge.Position = UDim2.fromOffset(7 + (slot - 1) * 16, 41)
            badge.BackgroundTransparency = 1
            badge.ScaleType = Enum.ScaleType.Fit
            badge.Image = ""
            badge.Visible = false
            badges[slot] = badge
        end

        brainrotCards[key] = {
            billboard = bb, nameLbl = nameLbl, genLbl = genLbl, img = img, badges = badges,
        }
        return brainrotCards[key]
    end

    badgeIconsFor = function(key)
        local entry = animalByUid[key]
        return entry and petBadgeIcons(entry) or {}
    end

    local function applyBadges(card, key)
        if not card or not card.badges then return end
        local ids = badgeIconsFor(key)
        for slot, badge in ipairs(card.badges) do
            local id = ids[slot]
            if id then
                if badge.Image ~= id then badge.Image = id end
                badge.Visible = true
            else
                badge.Visible = false
            end
        end
    end

    local function rebuildAnimalIndex()
        animalByUid = {}
        local cache = P.AllAnimalsCache
        if type(cache) ~= "table" then return end
        for _, entry in ipairs(cache) do
            if entry and entry.uid then animalByUid[entry.uid] = entry end
        end
    end

    local function scanPlot(plot)
        local podiums = plot:FindFirstChild("AnimalPodiums")
        if not podiums then return end
        local minGen = tonumber(config.BrainrotESPMinGen) or MIN_ESP_GEN_DEFAULT
        for _, podium in ipairs(podiums:GetChildren()) do
            local base = podium:FindFirstChild("Base")
            local spawnPart = base and base:FindFirstChild("Spawn")
            local attach = spawnPart and spawnPart:FindFirstChild("PromptAttachment")
            local key = plot.Name .. "_" .. podium.Name
            local objectText, found = "", false
            if attach then
                for _, ch in ipairs(attach:GetChildren()) do
                    if ch:IsA("ProximityPrompt") and ch.Enabled then
                        objectText = ch.ObjectText or ""
                        if objectText ~= "" then
                            found = true
                            break
                        end
                    end
                end
            end
            local host = attach and attach.Parent
            if found and host and host:IsA("BasePart") then
                local nameText, genText = objectText:match("^(.-)%s*|%s*(.-)$")
                if nameText and genText then
                    nameText = nameText:match("^%s*(.-)%s*$")
                    genText = genText:match("^%s*(.-)%s*$")
                else
                    nameText, genText = objectText, objectText
                end
                if nameText == "" then nameText = podium.Name end
                local entry = animalByUid[key]
                local gen = (entry and type(entry.genValue) == "number")
                    and entry.genValue or parseGenText(genText)
                local passes = (minGen <= 0) or (gen ~= nil and gen >= minGen)
                if passes then
                    local card = brainrotCards[key]
                    if not card then
                        applyBadges(makeCard(host, nameText, genText, key), key)
                    else
                        if card.nameLbl.Text ~= nameText then
                            card.nameLbl.Text = nameText
                            if card.img then
                                card.img.Image = iconFor(nameText) or card.img.Image
                            end
                        end
                        card.genLbl.Text = genText
                        if card.billboard.Adornee ~= host then
                            card.billboard.Adornee = host
                        end
                        applyBadges(card, key)
                    end
                else
                    removeCard(key)
                end
            else
                removeCard(key)
            end
        end
    end

    local function prefetchIcons()
        local plots = workspace:FindFirstChild("Plots")
        if not plots then return end
        local seen = {}
        for _, plot in ipairs(plots:GetChildren()) do
            local podiums = plot:FindFirstChild("AnimalPodiums")
            if podiums then
                for _, podium in ipairs(podiums:GetChildren()) do
                    local base = podium:FindFirstChild("Base")
                    local spawnPart = base and base:FindFirstChild("Spawn")
                    local attach = spawnPart and spawnPart:FindFirstChild("PromptAttachment")
                    if attach then
                        for _, ch in ipairs(attach:GetChildren()) do
                            if ch:IsA("ProximityPrompt") and ch.Enabled then
                                local txt = ch.ObjectText or ""
                                if txt ~= "" then
                                    local nm = txt:match("^(.-)%s*|") or txt
                                    nm = nm:match("^%s*(.-)%s*$")
                                    if nm ~= "" and not seen[nm] then
                                        seen[nm] = true
                                        iconFor(nm)
                                    end
                                end
                                break
                            end
                        end
                    end
                end
            end
        end
    end

    local function fillPendingIcons()
        for _, card in pairs(brainrotCards) do
            if card.img and card.img.Image == "" and card.nameLbl then
                local asset = iconFor(card.nameLbl.Text)
                if asset then card.img.Image = asset end
            end
        end
    end

    local function scanAllPlots()
        local plots = workspace:FindFirstChild("Plots")
        if not plots then return end
        rebuildAnimalIndex()
        local seen = {}
        for _, plot in ipairs(plots:GetChildren()) do
            local podiums = plot:FindFirstChild("AnimalPodiums")
            if podiums then
                for _, podium in ipairs(podiums:GetChildren()) do
                    seen[plot.Name .. "_" .. podium.Name] = true
                end
            end
            scanPlot(plot)
        end
        for key in pairs(brainrotCards) do
            if not seen[key] then removeCard(key) end
        end
    end
    P.RefreshBrainrotESP = function()
        if brainrotOn then pcall(scanAllPlots) end
    end

    local function stopBrainrotESP()
        brainrotOn = false
        for _, conn in ipairs(brainrotConns) do
            pcall(function()
                if typeof(conn) == "RBXScriptConnection" then conn:Disconnect() end
            end)
        end
        table.clear(brainrotConns)
        clearCards()
    end

    local function startBrainrotESP()
        if brainrotOn then return end
        brainrotOn = true
        task.spawn(function() pcall(prefetchIcons) end)
        local plots = workspace:FindFirstChild("Plots")
        if plots then
            scanAllPlots()
            brainrotConns[#brainrotConns + 1] = plots.DescendantAdded:Connect(function(d)
                if not brainrotOn then return end
                if d:IsA("ProximityPrompt") then
                    task.defer(function()
                        if not brainrotOn then return end
                        local plot = d:FindFirstAncestorWhichIsA("Model")
                        while plot and plot.Parent and plot.Parent ~= plots do
                            plot = plot.Parent
                        end
                        if plot and plot.Parent == plots then pcall(scanPlot, plot) end
                    end)
                end
            end)
            brainrotConns[#brainrotConns + 1] = plots.DescendantRemoving:Connect(function(d)
                if not brainrotOn then return end
                if d:IsA("ProximityPrompt") then task.defer(function()
                    if brainrotOn then pcall(scanAllPlots) end
                end) end
            end)
            brainrotConns[#brainrotConns + 1] = plots.ChildAdded:Connect(function()
                if not brainrotOn then return end
                task.defer(function()
                    if brainrotOn then pcall(scanAllPlots) end
                end)
            end)
        end
        task.spawn(function()
            while brainrotOn do
                task.wait(1.5)
                if brainrotOn then
                    pcall(scanAllPlots)
                    pcall(fillPendingIcons)
                end
            end
        end)

        task.spawn(function()
            while brainrotOn do
                local pending = false
                for _, card in pairs(brainrotCards) do
                    if card.img and card.img.Image == "" then
                        pending = true
                        break
                    end
                end
                if pending then pcall(fillPendingIcons) end
                task.wait(pending and 0.15 or 0.75)
            end
        end)
    end

    local function setBrainrotESP(enabled)
        enabled = enabled and true or false
        config.BrainrotESP = enabled
        saveConfig()
        setToggle("BrainrotESP", enabled)
        setToggle("Brainrot ESP", enabled)
        if enabled then
            startBrainrotESP()
        else
            stopBrainrotESP()
        end
    end

    _G.setBrainrotESP = setBrainrotESP

    _G.StickyOnBoot(function()
        if config.BrainrotESP == true then startBrainrotESP() end
    end)

    do
        local UIS = game:GetService("UserInputService")
        local AIM_TOOLS = { ["Web Slinger"] = true, ["Paintball Gun"] = true, ["Laser Cape"] = true }
        local aimOn = false
        local hookedTools = setmetatable({}, { __mode = "k" })
        local mouseMod, aimConn, lastAimUp = nil, nil, 0

        local function getMouseModule()
            if mouseMod then return mouseMod end
            pcall(function()
                local pkg = ReplicatedStorage:FindFirstChild("Packages")
                if pkg then
                    local m = pkg:FindFirstChild("PlayerMouse")
                    if m then mouseMod = require(m) end
                end
            end)
            return mouseMod
        end

        local function bestEnemy()
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local cam = workspace.CurrentCamera
            if not root or not cam then return nil end
            local best, score = nil, math.huge
            local camPos, camDir = cam.CFrame.Position, cam.CFrame.LookVector
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer and plr.Character then
                    local r = plr.Character:FindFirstChild("HumanoidRootPart")
                    local h = plr.Character:FindFirstChildOfClass("Humanoid")
                    if r and h and h.Health > 0 then
                        local d = (r.Position - root.Position).Magnitude
                        if d <= 100 then
                            local to = r.Position - camPos
                            if to.Magnitude > 0.01 then
                                local ang = math.deg(math.acos(math.clamp(camDir:Dot(to.Unit), -1, 1)))
                                local sc = d + (ang > 200 and 1000 or 0)
                                if sc < score then score, best = sc, r end
                            end
                        end
                    end
                end
            end
            return best
        end

        local function overrideMouse()
            if not aimOn then return end
            local pm = getMouseModule()
            if not pm then return end
            local e = bestEnemy()
            if e and e.Parent then
                local vel = e.AssemblyLinearVelocity or Vector3.zero
                pcall(function()
                    pm.Hit = CFrame.new(e.Position + vel * 0.1)
                    pm.Target = e
                end)
            end
        end

        local function hookTool(tool)
            if not tool or hookedTools[tool] then return end
            hookedTools[tool] = true
            tool.Activated:Connect(overrideMouse)
            tool.Equipped:Connect(function()
                task.wait(0.1)
                local pm = getMouseModule()
                if pm and aimOn then
                    local old = pm.Button1Down
                    pm.Button1Down = function(...)
                        overrideMouse()
                        if old then return old(...) end
                    end
                end
            end)
        end

        local watched = setmetatable({}, { __mode = "k" })
        local function watchTools(parent)
            if not parent then return end
            for _, c in ipairs(parent:GetChildren()) do
                if AIM_TOOLS[c.Name] then hookTool(c) end
            end
            if watched[parent] then return end
            watched[parent] = true
            parent.ChildAdded:Connect(function(c)
                task.wait(0.05)
                if AIM_TOOLS[c.Name] then hookTool(c) end
            end)
        end

        local function rescan()
            pcall(watchTools, LocalPlayer:FindFirstChild("Backpack"))
            if LocalPlayer.Character then pcall(watchTools, LocalPlayer.Character) end
        end

        LocalPlayer.CharacterAdded:Connect(function(c)
            task.wait(0.2)
            if aimOn then
                pcall(watchTools, c)
                pcall(watchTools, LocalPlayer:FindFirstChild("Backpack"))
            end
        end)

        local function setToolAimbot(enabled)
            enabled = enabled and true or false
            aimOn = enabled
            config.ToolAimbot = enabled
            saveConfig()
            setToggle("ToolAimbot", enabled)
            setToggle("Tool Aimbot", enabled)
            if aimConn then
                pcall(function() aimConn:Disconnect() end)
                aimConn = nil
            end
            if not enabled then return end
            rescan()
            aimConn = RunService.RenderStepped:Connect(function()
                if not aimOn then return end
                if tick() - lastAimUp > 0.1 then
                    lastAimUp = tick()
                    bestEnemy()
                end
                if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
                    overrideMouse()
                end
            end)
        end

        _G.setToolAimbot = setToolAimbot
        _G.StickyOnBoot(function()
            if config.ToolAimbot == true then pcall(setToolAimbot, true) end
        end)
    end

    do
        local boostConn = nil

        local function boostValue()
            return math.clamp(math.floor((tonumber(config.StealBoostValue) or 28) + 0.5), 15, 35)
        end

        local function setStealBoost(enabled)
            enabled = enabled and true or false
            config.StealBoost = enabled
            saveConfig()
            setToggle("StealBoost", enabled)
            setToggle("Steal Boost", enabled)
            if boostConn then
                pcall(function() boostConn:Disconnect() end)
                boostConn = nil
            end
            if not enabled then return end
            boostConn = RunService.Heartbeat:Connect(function(dt)
                local character = LocalPlayer.Character
                local humanoid = character and character:FindFirstChildOfClass("Humanoid")
                local hrp = character and character:FindFirstChild("HumanoidRootPart")
                if not humanoid or not hrp or humanoid.Health <= 0 then return end
                local target = boostValue()
                if humanoid.MoveDirection.Magnitude > 0 and target > humanoid.WalkSpeed then
                    hrp.CFrame = hrp.CFrame
                        + humanoid.MoveDirection * (target - humanoid.WalkSpeed) * (dt or 0.016)
                end
            end)
        end

        local function setStealBoostValue(v)
            config.StealBoostValue = math.clamp(math.floor((tonumber(v) or 28) + 0.5), 15, 35)
            saveConfig()
            return config.StealBoostValue
        end

        _G.setStealBoost = setStealBoost
        _G.setStealBoostValue = setStealBoostValue
        _G.getStealBoostValue = boostValue
        _G.StickyOnBoot(function()
            if config.StealBoost == true then pcall(setStealBoost, true) end
        end)
    end


    local stopAntiRagdoll, startAntiRagdoll
    local AntiRagdollData = { conns = {}, charConn = nil, beat = nil, acs = nil, acsChar = nil, rootAC = nil }
    local antiRagOn = false

    local function arv_anyACDisabled(ch)
        if AntiRagdollData.acsChar ~= ch or not AntiRagdollData.acs then
            AntiRagdollData.acsChar = ch
            AntiRagdollData.acs = {}
            AntiRagdollData.rootAC = nil
            for _, d in ipairs(ch:GetDescendants()) do
                if d:IsA("AnimationConstraint") then
                    table.insert(AntiRagdollData.acs, d)
                    if d.Name == "Root" then AntiRagdollData.rootAC = d end
                end
            end
        end
        for _, ac in ipairs(AntiRagdollData.acs) do
            if not ac.Enabled and ac.Parent then return true end
        end
        return false
    end

    local ARV_LAUNCH_CLASSES = {
        BodyVelocity = true, BodyForce = true, BodyThrust = true, BodyGyro = true,
        BodyAngularVelocity = true, VectorForce = true, AngularVelocity = true, RocketPropulsion = true,
    }
    local ARV_FORCE_WHITELIST = {
        SpeedForce = true, InvisSpeedForce = true, JumpForce = true,
        FlightPower = true, FlightSpin = true, FlightHold = true,
    }

    local function arv_isFlying(ch)
        if _G.StickyCarpetFlightAware == false then return false end
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if not hrp then return false end
        return hrp:FindFirstChild("FlightPower") ~= nil
            or hrp:FindFirstChild("FlightSpin") ~= nil
            or hrp:FindFirstChild("FlightHold") ~= nil
    end

    local function arv_considerLaunch(d)
        if ARV_FORCE_WHITELIST[d.Name] then return false end
        local cn = d.ClassName
        local launch = ARV_LAUNCH_CLASSES[cn]
        if not launch and cn == "LinearVelocity" then launch = true end
        if not launch
            and (cn:find("Force") or cn:find("Velocity") or cn:find("Body") or cn:find("Propulsion"))
            and (d.Name:find("Impulse") or d.Name:find("Knockback") or d.Name:find("Launch")) then
            launch = true
        end
        if launch then
            pcall(function() d:Destroy() end)
            return true
        end
        return false
    end

    local function arv_stripLaunchForces(ch)
        local found = false
        local root = ch:FindFirstChild("HumanoidRootPart")
        if root then
            for _, d in ipairs(root:GetChildren()) do
                found = arv_considerLaunch(d) or found
            end
        end
        for _, d in ipairs(ch:GetChildren()) do
            found = arv_considerLaunch(d) or found
        end
        return found
    end

    local arv_ctlState = true
    local function arv_controls(enable)
        if arv_ctlState == enable then return end
        arv_ctlState = enable
        pcall(function()
            local ps = LocalPlayer:FindFirstChild("PlayerScripts")
            local pm = ps and ps:FindFirstChild("PlayerModule")
            if pm then
                local ctl = require(pm):GetControls()
                if enable then ctl:Enable() else ctl:Disable() end
            end
        end)
    end

    local function cleanRagdoll(char)
        char = char or LocalPlayer.Character
        if not char then return end
        pcall(function()
            local et = LocalPlayer:GetAttribute("RagdollEndTime")
            if et and (et - Workspace:GetServerTimeNow()) > 0 then
                _G.StickyRagdollUntil = et
            end
            LocalPlayer:SetAttribute("RagdollEndTime", 0)
        end)
        local rootAC = nil
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("BallSocketConstraint") or d:IsA("HingeConstraint") or d:IsA("NoCollisionConstraint") then
                pcall(function() d:Destroy() end)
            elseif d:IsA("Motor6D") and not d.Enabled then
                pcall(function() d.Enabled = true end)
            elseif d:IsA("AnimationConstraint") then
                if not d.Enabled then pcall(function() d.Enabled = true end) end
                if rootAC == nil and d.Name == "Root" then rootAC = d end
            end
        end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local rra = hrp and hrp:FindFirstChild("RootRigAttachment")
        if rra and rootAC then
            if rootAC.Attachment0 == nil or rootAC.Attachment0.Parent == nil then
                pcall(function() rootAC.Attachment0 = rra end)
            end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            pcall(function() if hum.PlatformStand then hum.PlatformStand = false end end)
            local st = hum:GetState()
            if st == Enum.HumanoidStateType.Physics
                or st == Enum.HumanoidStateType.Ragdoll
                or st == Enum.HumanoidStateType.FallingDown
                or st == Enum.HumanoidStateType.PlatformStanding then
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
            end
            pcall(function() Workspace.CurrentCamera.CameraSubject = hum end)
        end
        arv_controls(true)
    end
    _G.StickyUnRagdoll = cleanRagdoll

    local function arv_killSource()
        if typeof(getconnections) ~= "function" then return end
        pcall(function()
            local pkg = ReplicatedStorage:FindFirstChild("Packages")
            local re
            local net = pkg and pkg:FindFirstChild("Net")
            if net then re = net:FindFirstChild("RE/Ragdoll") end
            if not re then
                local rag = pkg and pkg:FindFirstChild("Ragdoll")
                re = rag and rag:FindFirstChild("Ragdoll")
            end
            if re and re:IsA("RemoteEvent") then
                for _, cn in ipairs(getconnections(re.OnClientEvent)) do
                    local ok = pcall(function() cn:Disable() end)
                    if not ok then pcall(function() cn:Disconnect() end) end
                end
            end
        end)
    end

    local function arv_disconnectHooks()
        for _, cn in ipairs(AntiRagdollData.conns) do
            pcall(function() cn:Disconnect() end)
        end
        AntiRagdollData.conns = {}
    end

    local function hookAntiRag(char)
        if not antiRagOn or not char then return end
        arv_disconnectHooks()
        arv_killSource()
        cleanRagdoll(char)
        local cET = LocalPlayer:GetAttributeChangedSignal("RagdollEndTime"):Connect(function()
            if not antiRagOn then return end
            local et = LocalPlayer:GetAttribute("RagdollEndTime")
            if et and (et - Workspace:GetServerTimeNow()) > 0 then
                cleanRagdoll(LocalPlayer.Character)
            end
        end)
        table.insert(AntiRagdollData.conns, cET)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            local cSt = hum.StateChanged:Connect(function(_, newState)
                if not antiRagOn then return end
                if newState == Enum.HumanoidStateType.Physics
                    or newState == Enum.HumanoidStateType.Ragdoll
                    or newState == Enum.HumanoidStateType.PlatformStanding then
                    cleanRagdoll(LocalPlayer.Character)
                end
            end)
            table.insert(AntiRagdollData.conns, cSt)
        end
        local cleanQueued = false
        local cDA = char.DescendantAdded:Connect(function(d)
            if not antiRagOn then return end
            if d:IsA("BallSocketConstraint")
                or d:IsA("HingeConstraint")
                or d:IsA("NoCollisionConstraint") then
                if cleanQueued then return end
                cleanQueued = true
                task.defer(function()
                    cleanQueued = false
                    cleanRagdoll(LocalPlayer.Character)
                end)
            end
        end)
        table.insert(AntiRagdollData.conns, cDA)
    end

    stopAntiRagdoll = function()
        antiRagOn = false
        arv_disconnectHooks()
        if AntiRagdollData.charConn then
            pcall(function() AntiRagdollData.charConn:Disconnect() end)
            AntiRagdollData.charConn = nil
        end
        if AntiRagdollData.beat then
            pcall(function() AntiRagdollData.beat:Disconnect() end)
            AntiRagdollData.beat = nil
        end
    end

    startAntiRagdoll = function(enabled)
        stopAntiRagdoll()
        antiRagOn = enabled and true or false
        if not antiRagOn then return end
        if LocalPlayer.Character then
            task.spawn(function() hookAntiRag(LocalPlayer.Character) end)
        end
        AntiRagdollData.charConn = LocalPlayer.CharacterAdded:Connect(function(ch)
            if not antiRagOn then return end
            pcall(function() ch:WaitForChild("Humanoid", 10) end)
            hookAntiRag(ch)
            task.spawn(function()
                for _ = 1, 12 do
                    task.wait(0.25)
                    if antiRagOn then arv_killSource() else break end
                end
            end)
        end)
        local arvLastScan = 0
        local arvLastVel = Vector3.zero
        AntiRagdollData.beat = RunService.Heartbeat:Connect(function()
            if not antiRagOn then return end
            local ch = LocalPlayer.Character; if not ch then return end
            local et = LocalPlayer:GetAttribute("RagdollEndTime")
            local ragdolled = et ~= nil and (et - Workspace:GetServerTimeNow()) > 0
            local flying = arv_isFlying(ch)
            if not ragdolled and not flying then
                local hum = ch:FindFirstChildOfClass("Humanoid")
                if hum then
                    local s = hum:GetState()
                    ragdolled = (s == Enum.HumanoidStateType.Physics
                        or s == Enum.HumanoidStateType.Ragdoll
                        or s == Enum.HumanoidStateType.PlatformStanding
                        or hum.PlatformStand == true)
                end
            end
            if not ragdolled then ragdolled = arv_anyACDisabled(ch) end
            if not _G.invisibleStealEnabled then
                local r = AntiRagdollData.rootAC
                if r and r.Parent and (r.Attachment0 == nil or r.Attachment0.Parent == nil) then
                    local hrp = ch:FindFirstChild("HumanoidRootPart")
                    local rra = hrp and hrp:FindFirstChild("RootRigAttachment")
                    if rra then pcall(function() r.Attachment0 = rra end) end
                end
            end
            if not ragdolled then
                local now = tick()
                if now - arvLastScan > (tonumber(_G.StickyRagScanInterval) or 0.2) then
                    arvLastScan = now
                    ragdolled = ch:FindFirstChildWhichIsA("BallSocketConstraint", true) ~= nil
                        or ch:FindFirstChildWhichIsA("HingeConstraint", true) ~= nil
                end
            end
            if ragdolled then _G.StickyRagdollPhysLastT = tick() end
            local launched = false
            if _G.StickyAntiLaunch ~= false and (ragdolled or _G.StickyTpActive) then
                launched = arv_stripLaunchForces(ch)
            end
            if ragdolled then cleanRagdoll(ch) end
            if ragdolled or launched then
                local root = ch:FindFirstChild("HumanoidRootPart")
                if root then
                    root.AssemblyAngularVelocity = Vector3.zero
                    if _G.StickyRagdollStayPut ~= false
                        and not _G.StickyTpActive
                        and not _G.StickyDropFlingActive then
                        root.AssemblyLinearVelocity = Vector3.zero
                    end
                end
            end
            if config.AntiRagdoll ~= false
                and not _G.StickyTpActive
                and not _G.StickyCarpetSpeedActive
                and not flying
                and not _G.StickyCarpetBuyFlying
                and not _G.StickyDropFlingActive then
                local root = ch:FindFirstChild("HumanoidRootPart")
                if root then
                    local v = root.AssemblyLinearVelocity
                    if (v - arvLastVel).Magnitude > (tonumber(_G.StickyLaunchSpikeThreshold) or 110) then
                        root.AssemblyLinearVelocity = Vector3.zero
                        root.AssemblyAngularVelocity = Vector3.zero
                        v = Vector3.zero
                    end
                    arvLastVel = v
                end
            else
                local root = ch:FindFirstChild("HumanoidRootPart")
                arvLastVel = root and root.AssemblyLinearVelocity or Vector3.zero
            end
        end)
    end

    local function setAntiRagdoll(enabled)
        enabled = enabled and true or false
        config.AntiRagdoll = enabled
        saveConfig()
        setToggle("AntiRagdoll", enabled)
        setToggle("Anti Ragdoll", enabled)
        startAntiRagdoll(enabled)
    end

    _G.setAntiRagdoll = setAntiRagdoll

    _G.StickyOnBoot(function()
        if config.AntiRagdoll ~= false then
            pcall(startAntiRagdoll, true)
        end
    end)

    local Lighting = game:GetService("Lighting")
    local LP = LocalPlayer
    local RS = ReplicatedStorage

    if _G.StickyAntiBee == nil then _G.StickyAntiBee = config.AntiBeeDisco ~= false end

    ;(function()
        local BAD = { Blue = true, DiscoEffect = true, BeeBlur = true,
            Flashbang = true, ColorCorrection = true }
        local function on() return _G.StickyAntiBee ~= false end

        local function nuke(o)
            if on() and o and o.Parent and BAD[o.Name] then pcall(function() o:Destroy() end) end
        end

        local buzz
        local function muteBuzz()
            if not on() then return end
            pcall(function()
                if not (buzz and buzz.Parent) then
                    local ctl = RS:FindFirstChild("Controllers")
                    local item = ctl and ctl:FindFirstChild("ItemController")
                    local bee = item and item:FindFirstChild("BeeLauncherController")
                    local s = bee and bee:FindFirstChild("Buzzing")
                    if s and s:IsA("Sound") then buzz = s end
                end
                if buzz then
                    buzz.Volume = 0
                    if buzz.IsPlaying then buzz:Stop() end
                end
            end)
        end

        local guarded = {}
        local function guard(Controls, original)
            if not Controls or guarded[Controls] then return end
            local base = original or Controls.moveFunction
            if not base then return end
            local function safeMove(self, mv, rtc) return base(self, mv, rtc) end
            guarded[Controls] = safeMove
            Controls.moveFunction = safeMove
            RunService.Heartbeat:Connect(function()
                if not on() then return end
                if Controls.moveFunction ~= safeMove then Controls.moveFunction = safeMove end
            end)
        end

        local function protect()
            pcall(function()
                local cc = RS:FindFirstChild("Controllers")
                local mod = cc and cc:FindFirstChild("CharacterController")
                local m = mod and require(mod)
                if type(m) == "table" then guard(m.Controls, m.originalMoveFunction) end
            end)
            pcall(function()
                local ps = LP:WaitForChild("PlayerScripts", 5)
                local pm = ps and ps:FindFirstChild("PlayerModule")
                if pm then guard(require(pm):GetControls()) end
            end)
        end

        _G.StickyOnBoot(function()
            LP:WaitForChild("PlayerScripts", 8)
            Lighting.DescendantAdded:Connect(nuke)
            do
                local n = 0
                for _, o in ipairs(Lighting:GetDescendants()) do
                    n = n + 1; if n % 150 == 0 then task.wait() end
                    nuke(o)
                end
            end
            protect()
            local acc = 1
            RunService.Heartbeat:Connect(function(dt)
                if not on() then return end
                local cam = workspace.CurrentCamera
                if cam and math.abs(cam.FieldOfView - 20) < 0.01 then
                    cam.FieldOfView = tonumber(_G.StickyFOV) or 70
                end
                acc = acc + dt
                if acc < 0.5 then return end
                acc = 0
                muteBuzz()
            end)
        end)
        LP.CharacterAdded:Connect(function() task.delay(1, protect) end)
    end)()

    local function setAntiBeeDisco(enabled)
        enabled = enabled and true or false
        config.AntiBeeDisco = enabled
        _G.StickyAntiBee = enabled
        saveConfig()
        setToggle("AntiBeeDisco", enabled)
        setToggle("Anti Bee/Disco", enabled)
    end

    _G.setAntiBeeDisco = setAntiBeeDisco

    if _G.StickyAntiDieDisabled == nil then
        _G.StickyAntiDieDisabled = config.AntiDie == false
    end

    if type(_G.StickyAntiDieSuppress) ~= "number" then
        _G.StickyAntiDieSuppress = 0
    end
    local function antiDieOff()
        return _G.StickyAntiDieDisabled == true
            or (tonumber(_G.StickyAntiDieSuppress) or 0) > 0
    end
    _G.StickyAntiDiePause = function(maxSeconds)
        _G.StickyAntiDieSuppress = (tonumber(_G.StickyAntiDieSuppress) or 0) + 1
        local released = false
        local function release()
            if released then return end
            released = true
            _G.StickyAntiDieSuppress = math.max(0, (tonumber(_G.StickyAntiDieSuppress) or 1) - 1)
        end
        task.delay(math.max(1, tonumber(maxSeconds) or 12), release)
        return release
    end

    task.spawn(function()
        while not Players.LocalPlayer do task.wait() end
        pcall(function()
            local conn, diedConn, hbConn
            local function harden(hum)
                pcall(function() hum.BreakJointsOnDeath = false end)
                pcall(function() hum.RequiresNeck = false end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false) end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false) end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false) end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false) end)
            end
            local function revive(hum)
                pcall(function() hum.Health = hum.MaxHealth end)
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
            end
            local bind
            local rebindPending = false
            local function rebindSoon()
                if rebindPending then return end
                rebindPending = true
                task.delay(0.1, function()
                    rebindPending = false
                    pcall(bind)
                end)
            end
            function bind()
                local char = LP.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if not hum then return end
                harden(hum)
                if conn then pcall(function() conn:Disconnect() end) end
                if diedConn then pcall(function() diedConn:Disconnect() end) end
                if hbConn then pcall(function() hbConn:Disconnect() end) end
                conn = hum:GetPropertyChangedSignal("Health"):Connect(function()
                    if antiDieOff() then return end
                    if hum.Health <= 0 then revive(hum) end
                end)
                diedConn = hum.Died:Connect(function()
                    if antiDieOff() then return end
                    revive(hum)
                end)
                local lastHarden = 0
                hbConn = RunService.Heartbeat:Connect(function()
                    if not hum or not hum.Parent or hum.Parent ~= LP.Character then
                        if LP.Character and LP.Character:FindFirstChildOfClass("Humanoid") then
                            rebindSoon()
                        end
                        return
                    end
                    if antiDieOff() then return end
                    local now = os.clock()
                    if now - lastHarden >= 0.5 then lastHarden = now; harden(hum) end
                    if hum.Health <= 0 then revive(hum) end
                    if _G.StickyStealHold and hum.Health < hum.MaxHealth then
                        pcall(function() hum.Health = hum.MaxHealth end)
                    end
                    local char2 = hum.Parent
                    local hrp = char2 and char2:FindFirstChild("HumanoidRootPart")
                    if _G.invisibleStealEnabled == true
                        and _G.StickyInvisRagdollGuard ~= false then char2 = nil end
                    if char2 and hrp then
                        local state = hum:GetState()
                        local rag = (state == Enum.HumanoidStateType.Physics
                            or state == Enum.HumanoidStateType.Ragdoll
                            or state == Enum.HumanoidStateType.FallingDown)
                        if not rag then
                            local et = tonumber(LP:GetAttribute("RagdollEndTime"))
                            if et and (et - workspace:GetServerTimeNow()) > 0 then rag = true end
                        end
                        if rag then
                            pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
                            pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
                            if not _G.__stickyResetBusy and not _G.StickyDropFlingActive
                                and LP:GetAttribute("Stealing") ~= true then
                                pcall(function() hrp.AssemblyLinearVelocity = Vector3.zero end)
                            end
                            local cam = workspace.CurrentCamera
                            if cam and cam.CameraSubject ~= hum then
                                pcall(function() cam.CameraSubject = hum end)
                            end
                            for _, obj in ipairs(char2:GetDescendants()) do
                                if obj:IsA("BallSocketConstraint") or (obj.Name and obj.Name:find("RagdollAttachment")) then
                                    pcall(function() obj:Destroy() end)
                                end
                            end
                        end
                    end
                    if hum:GetState() == Enum.HumanoidStateType.Dead then
                        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
                    end
                end)
            end
            bind()
            LP.CharacterAdded:Connect(function(char)
                local hum = char:WaitForChild("Humanoid", 5)
                if hum then harden(hum) end
                task.wait(0.1)
                bind()
            end)
            LP:GetPropertyChangedSignal("Character"):Connect(rebindSoon)
            _G.StickyAntiDieRebind = rebindSoon
        end)
    end)

    local function setAntiDie(enabled)
        enabled = enabled and true or false
        config.AntiDie = enabled
        _G.StickyAntiDieDisabled = not enabled
        if enabled then
            _G.StickyAntiDieSuppress = 0
            if type(_G.StickyAntiDieRebind) == "function" then
                pcall(_G.StickyAntiDieRebind)
            end
        end
        saveConfig()
        setToggle("AntiDie", enabled)
        setToggle("Anti Die", enabled)
    end

    _G.setAntiDie = setAntiDie

    local CARPET_NAMES = { "Flying Carpet", "Waverider", "Santa's Sleigh", "Witch's Broom", "Cupid's Wings" }
    local _lastCarpetName = nil

    local function findTool(n)
        local ch = LP.Character
        local bp = LP:FindFirstChild("Backpack")
        return (ch and ch:FindFirstChild(n)) or (bp and bp:FindFirstChild(n))
    end

    local function equipCarpet()
        local char = LP.Character
        if not char then return nil end
        if _lastCarpetName then
            local t = char:FindFirstChild(_lastCarpetName)
            if t and t.Parent == char then return _lastCarpetName end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return nil end
        for _, n in ipairs(CARPET_NAMES) do
            local t = findTool(n)
            if t and t:IsA("Tool") then
                if t.Parent ~= char then pcall(function() hum:EquipTool(t) end) end
                _lastCarpetName = n
                return n
            end
        end
        return nil
    end
    _G.StickyEquipCarpet = equipCarpet

    _G.StickyCarpetEngaging = function()
        local char = LP.Character
        if not char then return false end
        for _, n in ipairs(CARPET_NAMES) do
            local t = char:FindFirstChild(n)
            if t and t:IsA("Tool") then return true end
        end
        return false
    end

    do
        local drop = { running = false, connections = {}, rootPart = nil, savedCFrame = nil }

        local function stopDrop()
            drop.running = false
            _G.StickyDropFlingActive = false
            for _, conn in ipairs(drop.connections) do
                if typeof(conn) == "RBXScriptConnection" then
                    pcall(function() conn:Disconnect() end)
                end
            end
            drop.connections = {}
        end

        local function startDrop()
            if drop.running then return end
            local character = LP.Character
            if not character then return end
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            drop.running = true
            _G.StickyDropFlingActive = true
            drop.rootPart = hrp
            drop.savedCFrame = hrp.CFrame - hrp.CFrame.Position
            if _G.invisibleStealEnabled then
                hrp.CFrame = hrp.CFrame * CFrame.new(0, 3, 0)
            end
            table.insert(drop.connections, RunService.Stepped:Connect(function()
                if not drop.running then return end
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LP and player.Character then
                        for _, child in ipairs(player.Character:GetChildren()) do
                            if child:IsA("BasePart") then child.CanCollide = false end
                        end
                    end
                end
            end))

            table.insert(drop.connections, RunService.Stepped:Connect(function()
                if not drop.running or not hrp or not hrp.Parent then return end
                hrp.CFrame = CFrame.new(hrp.Position) * drop.savedCFrame
                hrp.RotVelocity = Vector3.zero
            end))

            task.spawn(function()
                while drop.running do
                    RunService.Heartbeat:Wait()
                    if not hrp or not hrp.Parent then break end
                    local velocity = hrp.Velocity
                    hrp.Velocity = velocity * 10000 + Vector3.new(0, 10000, 0)
                    RunService.RenderStepped:Wait()
                    if hrp and hrp.Parent then hrp.Velocity = velocity end
                    RunService.Stepped:Wait()
                    if hrp and hrp.Parent then
                        hrp.Velocity = velocity + Vector3.new(0, 0.1, 0)
                    end
                end
            end)
        end

        _G.StickyDropBrainrot = function()
            if drop.running then return end
            startDrop()
            task.delay(0.4, function()
                stopDrop()
                task.wait(0.05)
                if drop.rootPart and drop.rootPart.Parent and drop.savedCFrame then
                    pcall(function()
                        drop.rootPart.CFrame = CFrame.new(drop.rootPart.Position) * drop.savedCFrame
                        drop.rootPart.RotVelocity = Vector3.zero
                    end)
                end
            end)
        end
        _G.StickyStopWalkFling = stopDrop
    end

    do
        local _resetRemote = nil
        local _rawFS = nil
        local _resetFlooding = false
        local _armCapture
        local _arming = false

        _armCapture = function()
            if _resetRemote or _arming then return end
            if not hookfunction or _G.StickyResetCapture == false then return end
            _arming = true
            pcall(function()
                local newcc = newcclosure or function(f) return f end
                local _t0, _restored = os.clock(), false

                local function _restore(why)
                    if _restored then return end
                    _restored = true
                    pcall(function()
                        hookfunction(Instance.new("RemoteEvent").FireServer, _rawFS)
                    end)
                    _G.StickyResetCaptureMs = (os.clock() - _t0) * 1000
                    _G.StickyResetCaptureWhy = why
                    _arming = false
                end

                local CALLER = tostring(_G.StickyResetCaller or "ToolActivationController")
                local _gcs = getcallingscript
                if not _gcs then _restore("no-getcallingscript") return end
                local _fireCount = {}

                local function _capture(self)
                    local nm = self.Name
                    if nm:sub(1, 3) ~= "RE/" then return end
                    if not nm:match("^RE/%x%x%x%x%x%x%x%x") then return end
                    local s = _gcs()
                    if not s or s.Name ~= CALLER then return end
                    local c = (_fireCount[self] or 0) + 1
                    _fireCount[self] = c
                    if c >= (tonumber(_G.StickyResetMinFires) or 2) then
                        _resetRemote = self
                        _G.StickyResetRemote = self
                        _G.StickyResetRemoteName = nm
                        task.defer(_restore, "captured")
                    end
                end

                _rawFS = hookfunction(Instance.new("RemoteEvent").FireServer, newcc(function(self, ...)
                    if not _resetRemote and type((...)) == "string" then
                        pcall(_capture, self)
                    end
                    return _rawFS(self, ...)
                end))

                task.delay(tonumber(_G.StickyResetCaptureMax) or 8, function()
                    _restore("timeout")
                end)
            end)
        end
        _G.StickyResetArm = function() _armCapture() end

        task.spawn(function()
            local function watchChar(ch)
                if not ch then return end
                for _, o in ipairs(ch:GetChildren()) do
                    if o:IsA("Tool") and not _resetRemote then _armCapture() break end
                end
                ch.ChildAdded:Connect(function(o)
                    if not _resetRemote and o:IsA("Tool") then _armCapture() end
                end)
            end
            if LP.Character then watchChar(LP.Character) end
            LP.CharacterAdded:Connect(watchChar)
        end)

        local function _stowOnce()
            local held = false
            pcall(function()
                local ch = LP.Character
                if not ch then return end
                for _, t in ipairs(ch:GetChildren()) do
                    if t:IsA("Tool") then held = true; break end
                end
                if not held then return end
                local hum = ch:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:UnequipTools() end) end
                for _, t in ipairs(ch:GetChildren()) do
                    if t:IsA("Tool") then
                        pcall(function() t.Parent = LP:FindFirstChild("Backpack") end)
                    end
                end
            end)
            return held
        end

        _G.StickyInstantReset = function(times)
            if _G.StickyResetJunkFire == true then
                if _resetFlooding then return true end
                _resetFlooding = true
                task.spawn(function()
                    if _G.StickyResetStowTools ~= false then
                        _G.StickyResetStowActive = true
                        _stowOnce()
                        RunService.Heartbeat:Wait()
                    end
                    local _releaseAntiDie = _G.StickyAntiDiePause
                        and _G.StickyAntiDiePause((tonumber(_G.StickyResetRestoreMax) or 8) + 4)
                        or function() end
                    local _done = false
                    local function _finish()
                        if _done then return end
                        _done = true
                        pcall(_releaseAntiDie)
                        _G.StickyResetStowActive = false
                        _resetFlooding = false
                    end
                    local _rc
                    _rc = LP.CharacterAdded:Connect(function()
                        if _rc then _rc:Disconnect(); _rc = nil end
                        _finish()
                    end)
                    task.delay(tonumber(_G.StickyResetRestoreMax) or 8, function()
                        if _rc then _rc:Disconnect(); _rc = nil end
                        _finish()
                    end)
                    pcall(function()
                        local ch = LP.Character
                        local hum = ch and ch:FindFirstChildOfClass("Humanoid")
                        if hum then
                            pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true) end)
                            pcall(function() hum.BreakJointsOnDeath = true end)
                            hum:ChangeState(Enum.HumanoidStateType.Dead)
                        end
                    end)
                end)
                return true
            end

            local r = _resetRemote or _G.StickyResetRemote
            if not r then
                _armCapture()
                local _lim = tonumber(_G.StickyResetArmWait) or 0
                if _lim > 0 then
                    local _tw = os.clock()
                    while not _resetRemote and (os.clock() - _tw) < _lim do task.wait(0.1) end
                end
                r = _resetRemote or _G.StickyResetRemote
            end
            if not r then return false end
            if not r.Parent then _G.StickyResetRemoteDead = true return false end
            _G.StickyResetRemoteDead = false

            if _resetFlooding then return true end
            _resetFlooding = true

            local _stowed = false
            if _G.StickyResetStowTools ~= false then
                _G.StickyResetStowActive = true
                _stowed = _stowOnce()
            end

            local fs = (_G.StickyResetUseRawFS == true and _rawFS)
                or function(rr, a) return rr:FireServer(a) end

            local count = math.clamp(tonumber(times) or tonumber(_G.StickyResetBurst) or 50, 1, 50)

            task.spawn(function()
                local stop, fired = false, 0
                local conn
                pcall(function()
                    conn = LP.CharacterRemoving:Connect(function() stop = true end)
                end)

                if _G.StickyResetStowTools ~= false then
                    local wait0 = math.clamp(tonumber(_G.StickyResetStowWait) or 0.1, 0, 1)
                    local cap0 = math.clamp(tonumber(_G.StickyResetStowCap) or 0.5, wait0, 3)
                    local tEnf = os.clock()
                    local tClean = _stowed and tEnf or (tEnf - wait0)
                    while not stop and (os.clock() - tEnf) < cap0 do
                        if _stowOnce() then tClean = os.clock() end
                        if (os.clock() - tClean) >= wait0 then break end
                        RunService.Heartbeat:Wait()
                    end
                end

                local _oldChar = LP.Character
                local t0 = os.clock()
                local lim = tonumber(_G.StickyResetMaxTime) or 3
                for _ = 1, count do
                    if stop or LP.Character ~= _oldChar then break end
                    if (os.clock() - t0) >= lim then break end
                    pcall(fs, r, "randomstring")
                    fired = fired + 1
                    RunService.Heartbeat:Wait()
                end

                _G.StickyResetLastFires = fired
                _G.StickyResetLastMs = (os.clock() - t0) * 1000
                _G.StickyResetLastOk = (stop or LP.Character ~= _oldChar)

                if conn then pcall(function() conn:Disconnect() end) end
                _G.StickyResetStowActive = false
                _resetFlooding = false
            end)
            return true
        end

        local function _moveToolsToBackpack(character)
            if not character then return end
            local bp = LP:FindFirstChild("Backpack")
            if not bp then return end
            for _, ch in ipairs(character:GetChildren()) do
                if ch:IsA("Tool") then
                    pcall(function() ch.Parent = bp end)
                end
            end
        end

        _G.StickyExecuteReset = function()
            if _G.StickyInstantReset and _G.StickyInstantReset(50) then return end

            pcall(function() Players.RespawnTime = 0 end)

            local _releaseAntiDie = _G.StickyAntiDiePause and _G.StickyAntiDiePause(14)
                or function() end
            _G.__stickyResetBusy = true

            local respawnConn
            local _restored = false
            local function _restoreAntiDie()
                if _restored then return end
                _restored = true
                pcall(_releaseAntiDie)
                _G.__stickyResetBusy = false
            end
            respawnConn = LP.CharacterAdded:Connect(function(newChar)
                if respawnConn then respawnConn:Disconnect(); respawnConn = nil end
                task.defer(function()
                    pcall(function() newChar:WaitForChild("Humanoid", 12) end)
                    RunService.Heartbeat:Wait()
                    _restoreAntiDie()
                end)
            end)
            task.delay(10, function()
                if respawnConn then respawnConn:Disconnect(); respawnConn = nil end
                _restoreAntiDie()
            end)

            local character = LP.Character
            if not character then
                pcall(function() LP:LoadCharacter() end)
                return
            end

            pcall(function()
                local function _hr()
                    return character:FindFirstChildOfClass("Humanoid"),
                        character:FindFirstChild("HumanoidRootPart")
                end
                local humanoid, rootPart = _hr()
                if not (rootPart and humanoid) then return end

                pcall(function()
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
                    humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                    humanoid.BreakJointsOnDeath = true
                end)

                _moveToolsToBackpack(character)
                rootPart.CFrame = CFrame.new(0, 15000, 0)
                RunService.Heartbeat:Wait()

                _moveToolsToBackpack(character)
                RunService.Heartbeat:Wait()

                humanoid, rootPart = _hr()
                if not (humanoid and rootPart) then return end

                pcall(function() humanoid.Health = 0 end)
                pcall(function() humanoid:ChangeState(Enum.HumanoidStateType.Dead) end)
                if humanoid.Health > 0 then
                    pcall(function() humanoid:TakeDamage(humanoid.MaxHealth * 99) end)
                end
                if humanoid.Health > 0 then
                    pcall(function() character:BreakJoints() end)
                end

                humanoid, rootPart = _hr()
                if humanoid and rootPart and humanoid.Health > 0 then
                    pcall(function()
                        rootPart.AssemblyLinearVelocity = Vector3.zero
                        rootPart.CFrame = CFrame.new(
                            rootPart.Position.X,
                            workspace.FallenPartsDestroyHeight - 500,
                            rootPart.Position.Z)
                    end)
                end
            end)

            task.spawn(function()
                local _origChar = LP.Character
                for _ = 1, 8 do
                    pcall(function() LP:LoadCharacter() end)
                    task.wait(0.05)
                    if LP.Character and LP.Character ~= _origChar then break end
                end
            end)
        end

        _G.StickyInstaReset = _G.StickyExecuteReset

        if _G.StickyForceRespawn == nil then
            local _lastForce = 0
            _G.StickyForceRespawn = function()
                local now = os.clock()
                if now - _lastForce < 3 then return end
                _lastForce = now
                task.spawn(function() pcall(_G.StickyExecuteReset) end)
            end
        end

        if _G.StickyResetCaptureAtLoad ~= false then
            _G.StickyOnBoot(function() _armCapture() end)
        end
    end
    local autoTurretOn = false

    local function StickyAlive()
        local c = LocalPlayer.Character
        local hum = c and c:FindFirstChildOfClass("Humanoid")
        return (c ~= nil and hum ~= nil and hum.Health > 0)
    end

    local function adt_getChar()
        local c = LocalPlayer.Character
        return c, c and c:FindFirstChild("HumanoidRootPart"), c and c:FindFirstChildOfClass("Humanoid")
    end

    local adt_pin = setmetatable({}, { __mode = "k" })

    local function adt_isDeploying(target)
        local strict = _G.StickyTurretDeployStrict ~= false
        local pinned, why = false, nil
        local fp = {}
        for _, d in ipairs(target:GetDescendants()) do
            if d:IsA("TextLabel") then
                local t = d.Text
                if t ~= "" then
                    local ev = false
                    if t:lower():find("ready", 1, true) then
                        ev = true
                        if not pinned then pinned, why = true, "ready-label" end
                    elseif not strict then
                        if t:find("!", 1, true) or t:match("^%s*%d+%.?%d*%s*[sS]?%s*$") then
                            ev = true
                            if not pinned then pinned, why = true, "loose" end
                        end
                    else
                        local num = t:match("^%s*(%d+%.?%d*)%s*[sS]?%s*!*%s*$")
                        if num and (tonumber(num) or 99) <= 10 then
                            ev = true
                            if not pinned then pinned, why = true, "countdown=" .. num end
                        end
                    end
                    if ev then fp[#fp + 1] = t end
                end
            end
        end
        if not pinned then adt_pin[target] = nil return false, nil end
        local key = table.concat(fp, "\1")
        local now = tick()
        local p = adt_pin[target]
        if not p or p.key ~= key then
            adt_pin[target] = { key = key, at = now, why = why }
            return true, why
        end
        p.why = why
        if _G.StickyTurretStaticRelease ~= false
            and now - p.at >= (tonumber(_G.StickyTurretStaticPin) or 1.2) then
            return false, "static:" .. why
        end
        return true, why
    end

    local adt_seen = setmetatable({}, { __mode = "k" })
    local adt_armed = setmetatable({}, { __mode = "k" })
    local adt_tr = setmetatable({}, { __mode = "k" })
    local adt_cool = setmetatable({}, { __mode = "k" })
    local adt_gate = { tp = 0, speed = 0, buykick = 0, steal = 0, join = 0, noweapon = 0 }

    local function adt_gateDump()
        local parts = {}
        for _, k in ipairs({ "tp", "speed", "buykick", "steal", "join", "noweapon" }) do
            if adt_gate[k] > 0.05 then
                parts[#parts + 1] = string.format("%s %.2fs", k, adt_gate[k])
            end
        end
        return #parts > 0 and table.concat(parts, ", ") or "none"
    end

    local function adt_isPlayerRig(inst)
        if _G.StickyTurretSkipPlayers == false then return false end
        local par = inst
        while par and par ~= workspace do
            if Players:GetPlayerFromCharacter(par)
                or (par.ClassName == "Model" and par:FindFirstChildOfClass("Humanoid")) then
                return true
            end
            par = par.Parent
        end
        return false
    end

    local function adt_isTargetable(inst)
        if adt_isPlayerRig(inst) then return false end
        if _G.StickyTurretWaitReady == false then return true end
        if adt_armed[inst] then return true end
        local now = tick()
        local seen = adt_seen[inst]
        if not seen then seen = now; adt_seen[inst] = now end
        local tr = adt_tr[inst]
        if not tr then tr = { seen = now }; adt_tr[inst] = tr end
        if now - seen >= (tonumber(_G.StickyTurretDeploySpan) or 12) then
            adt_armed[inst] = true
            if tr.pinwhy and not tr.pinrel then tr.pinrel = "age-cap" end
            return true
        end
        local dep, why = adt_isDeploying(inst)
        if why and dep then tr.pinwhy = why end
        if dep then return false end
        if tr.pinwhy and not tr.pinrel then
            tr.pinrel = why and "static" or "deploy-done"
        end
        if _G.StickyTurretArmLatch ~= false then adt_armed[inst] = true end
        return true
    end

    local function adt_applyVisuals(target)
        for _, d in ipairs(target:GetDescendants()) do
            if d:IsA("BasePart") and d ~= target then
                d.Transparency = 0.5
                d.CanCollide = false
                d.CanTouch = false
                d.CanQuery = false
            elseif d:IsA("BillboardGui") and d.Name == "SentryLabel" then
                pcall(function() d:Destroy() end)
            elseif d:IsA("Decal") or d:IsA("Texture") then
                d.Transparency = 0.5
            end
        end
        if target:IsA("BasePart") and target.Name == "ProxyVisual" then
            target.Transparency = 1
            target.CanCollide = false
        end
    end

    local function adt_getWeapon()
        local c = LocalPlayer.Character
        if not c then return nil end
        local bp = LocalPlayer:FindFirstChild("Backpack")
        return (bp and bp:FindFirstChild("Bat")) or c:FindFirstChild("Bat")
            or (bp and bp:FindFirstChild("Gummy Bear")) or c:FindFirstChild("Gummy Bear")
    end

    local function adt_equipBat()
        local c, _, hum = adt_getChar()
        if not c or not hum then return end
        local w = adt_getWeapon()
        if w and w.Parent ~= c then hum:EquipTool(w) end
    end

    local adt_lastDeep = 0
    local function adt_getClosestSentry(includeCooled)
        local _, hrp = adt_getChar()
        if not hrp then return nil end
        local closest, shortest = nil, math.huge
        local sfind2 = string.find
        local now = tick()
        local hrpPos = hrp.Position
        local function _scan(list)
            for n = 1, #list do
                local inst = list[n]
                local nm = inst.Name
                if sfind2(nm, "Sentry", 1, true) and nm ~= "SentryBullet"
                    and (inst.ClassName == "Model" or inst:IsA("BasePart"))
                    and (includeCooled or not (adt_cool[inst] and now < adt_cool[inst]))
                    and adt_isTargetable(inst) then
                    local root = inst:IsA("BasePart") and inst
                        or inst:FindFirstChildWhichIsA("BasePart", true)
                    if root then
                        local dist = (hrpPos - root.Position).Magnitude
                        if dist < shortest then shortest = dist; closest = inst end
                    end
                end
            end
        end
        _scan(workspace:GetChildren())
        if not closest and (now - adt_lastDeep) > (tonumber(_G.StickyTurretDeepEvery) or 3) then
            adt_lastDeep = now
            _scan(workspace:GetDescendants())
        end
        return closest
    end

    function _G.StickyAutoTurretActive()
        return autoTurretOn == true
    end

    function _G.StickyToggleAutoTurret()
        autoTurretOn = not autoTurretOn
        if autoTurretOn then adt_lastDeep = 0 end
        return autoTurretOn
    end

    local adt_cycleActive = false

    local function adt_destroyCycle(targetSentry)
        if adt_cycleActive then return end
        adt_cycleActive = true
        local trc = adt_tr[targetSentry]
        if trc and not trc.engage then trc.engage = tick() end
        local tdSurf = _G.invisibleStealEnabled == true
        if tdSurf then _G.StickyInvisSurface = true end
        local t0 = tick()
        while targetSentry and targetSentry.Parent
            and LocalPlayer:GetAttribute("Stealing") ~= true
            and tick() - t0 < (tonumber(_G.StickyTurretCycleCap) or 6) do
            local c, hrp, hum = adt_getChar()
            if not c or not hrp or not hum then break end
            if not StickyAlive() then break end
            if _G.StickyTpActive then break end
            if _G.StickyTurretYieldToSpeed ~= false and _G.StickyCarpetSpeedActive then break end
            if _G.StickyTurretYieldToBuyKick ~= false then
                if _G.StickyAutoBuyActive and _G.StickyAutoBuyActive() then break end
            end
            local w = adt_getWeapon()
            adt_applyVisuals(targetSentry)
            local offset = hrp.CFrame.LookVector * 4
            local targetCF = CFrame.new(hrp.Position + offset, hrp.Position)
            if targetSentry:IsA("Model") then
                targetSentry:PivotTo(targetCF)
            elseif targetSentry:IsA("BasePart") then
                targetSentry.CFrame = targetCF
            end
            if w then
                if w.Parent ~= c then hum:EquipTool(w) end
                w:Activate()
            else
                break
            end
            task.wait(0.1)
        end
        if targetSentry and targetSentry.Parent
            and tick() - t0 >= (tonumber(_G.StickyTurretCycleCap) or 6) then
            adt_cool[targetSentry] = tick() + (tonumber(_G.StickyTurretCooldown) or 2.5)
        end
        if tdSurf then _G.StickyInvisSurface = false end
        if trc then
            local t1 = tick()
            local seen = trc.seen or t1
            local arm = trc.armed or seen
            local eng = trc.engage or t1
            local dead = not (targetSentry and targetSentry.Parent)
            local pin = trc.pinwhy and string.format(": %s -> %s", trc.pinwhy, trc.pinrel or "?") or ""
            _G.StickyTurretLastReport = string.format(
                "[Turret] %s | total %.2fs | seen->armed %.2fs (ready-wait%s) | armed->engage %.2fs (ours) | swinging %.2fs | blocked by: %s",
                dead and "DESTROYED" or "SURVIVED", t1 - seen, arm - seen, pin, eng - arm, t1 - eng, adt_gateDump())
        end
        for k in pairs(adt_gate) do adt_gate[k] = 0 end
        adt_cycleActive = false
    end

    function _G.StickyDestroyTurretIfThreat()
        if not (autoTurretOn and StickyAlive()) then return false end
        if _G.StickyTurretYieldToBuyKick ~= false then
            if _G.StickyAutoBuyActive and _G.StickyAutoBuyActive() then return false end
        end
        local _, hrp = adt_getChar()
        if not hrp then return false end
        local s = adt_getClosestSentry(true)
        if not s then return false end
        local root = s:IsA("BasePart") and s or s:FindFirstChildWhichIsA("BasePart", true)
        if not root then return false end
        if (hrp.Position - root.Position).Magnitude > (tonumber(_G.StickyTurretHitRange) or 45) then
            return false
        end
        if not adt_getWeapon() then return false end
        adt_equipBat()
        adt_destroyCycle(s)
        return true
    end

    function _G.StickyTurretThreatNearby()
        if not (autoTurretOn and StickyAlive()) then return false end
        local _, hrp = adt_getChar()
        if not hrp then return false end
        local s = adt_getClosestSentry(true)
        if not s then return false end
        local root = s:IsA("BasePart") and s or s:FindFirstChildWhichIsA("BasePart", true)
        if not root then return false end
        return (hrp.Position - root.Position).Magnitude <= (tonumber(_G.StickyTurretHitRange) or 45)
    end

    local adt_boot = tick()
    local adt_sweepIdx, adt_lastSweep = 0, 0
    local adt_lastScan, adt_hadTarget = 0, false
    local adt_sweeping = false
    local adt_known = 0
    local adt_whyLog = {}

    local function _why(s)
        if _G.StickyTurretWhy ~= s then
            _G.StickyTurretWhy = s
            local L = adt_whyLog
            L[#L + 1] = string.format("%.1f %s", tick() - adt_boot, s)
            if #L > 24 then table.remove(L, 1) end
            _G.StickyTurretWhyLog = table.concat(L, " | ")
        end
    end

    task.spawn(function()
        local adt_lastTick, adt_dt = tick(), 0
        local function _gate(name)
            if adt_known > 0 then
                adt_gate[name] = (adt_gate[name] or 0) + adt_dt
            end
        end
        while true do
            task.wait(0.1)
            local now = tick()
            adt_dt = now - adt_lastTick; adt_lastTick = now
            if not StickyAlive() then _why("not-in-game") continue end
            if _G.StickyTurretObserve ~= false then
                local n, kids = 0, workspace:GetChildren()
                for n2 = 1, #kids do
                    local inst = kids[n2]
                    local nm = inst.Name
                    if string.find(nm, "Sentry", 1, true) and nm ~= "SentryBullet"
                        and (inst.ClassName == "Model" or inst:IsA("BasePart"))
                        and not adt_isPlayerRig(inst) then
                        n = n + 1
                        local tr = adt_tr[inst]
                        if not tr then tr = { seen = now }; adt_tr[inst] = tr end
                        if autoTurretOn and adt_isTargetable(inst) and not tr.armed then
                            tr.armed = now
                        end
                    end
                end
                adt_known = n
            else
                adt_known = 0
            end
            if not autoTurretOn then _why("off") continue end
            if now - adt_boot < (tonumber(_G.StickyTurretJoinGrace) or 8) then
                _why("join-grace") _gate("join") continue
            end
            if _G.StickyTpActive then _why("gate:tp") _gate("tp") continue end
            if _G.StickyTurretYieldToSpeed ~= false and _G.StickyCarpetSpeedActive then
                _why("gate:speed") _gate("speed") continue
            end
            if _G.StickyTurretYieldToBuyKick ~= false then
                if _G.StickyAutoBuyActive and _G.StickyAutoBuyActive() then
                    _why("gate:buykick") _gate("buykick") continue
                end
            end
            if LocalPlayer:GetAttribute("Stealing") == true then
                _why("gate:steal") _gate("steal") continue
            end
            if adt_known == 0 and not adt_hadTarget
                and (now - adt_lastScan) < (tonumber(_G.StickyTurretIdleScanDt) or 0.3) then
                _why("idle:no-sentry") continue
            end
            adt_lastScan = now
            local targetSentry = adt_getClosestSentry()
            adt_hadTarget = targetSentry ~= nil
            if not targetSentry then
                _why(adt_known > 0 and "sentry-known-not-ready" or "idle:no-sentry")
                if _G.StickyTurretStreamSweep ~= false and not adt_sweeping
                    and now - adt_lastSweep > (tonumber(_G.StickyTurretSweepEvery) or 1) then
                    adt_lastSweep = now
                    local plots = workspace:FindFirstChild("Plots")
                    if plots then
                        local kids = plots:GetChildren()
                        if #kids > 0 then
                            adt_sweepIdx = adt_sweepIdx % #kids + 1
                            local mr = kids[adt_sweepIdx]:FindFirstChild("MainRoot")
                            if mr and workspace.StreamingEnabled then
                                adt_sweeping = true
                                local sweepPos = mr.Position
                                task.spawn(function()
                                    pcall(function()
                                        workspace:RequestStreamAroundAsync(sweepPos, 1)
                                    end)
                                    adt_sweeping = false
                                end)
                            end
                        end
                    end
                end
                continue
            end
            if not adt_getWeapon() then _why("no-weapon") _gate("noweapon") continue end
            _why("engaging")
            adt_equipBat()
            adt_destroyCycle(targetSentry)
        end
    end)

    local function setAutoTurret(enabled)
        enabled = enabled and true or false
        config.AutoTurret = enabled
        saveConfig()
        setToggle("AutoTurret", enabled)
        setToggle("Auto Turret", enabled)
        autoTurretOn = enabled
        if enabled then adt_lastDeep = 0 end
    end

    _G.setAutoTurret = setAutoTurret

    _G.StickyOnBoot(function()
        if config.AutoTurret == true then autoTurretOn = true end
    end)

    do
        local Lighting2 = game:GetService("Lighting")

        local antiLagEnabled = false
        local antiLagRunning = false
        local antiLagConn = nil
        local saved = nil

        local function applyAntiLagDerender(item)
            pcall(function()
                if item:IsA("Accessory") or item:IsA("Hat") then
                    item:Destroy()
                    return
                end
                if item:IsA("BasePart") then
                    item.Material = Enum.Material.Plastic
                    item.Reflectance = 0
                    item.CastShadow = false
                    return
                end
                if item:IsA("Decal") or item:IsA("Texture") then
                    item.Transparency = 1
                    return
                end
                if item:IsA("ParticleEmitter") or item:IsA("Trail") or item:IsA("Beam")
                    or item:IsA("Fire") or item:IsA("Smoke") or item:IsA("Sparkles") then
                    item.Enabled = false
                end
            end)
        end

        local function enableAntiLag()
            antiLagRunning = true
            antiLagEnabled = true

            if saved == nil then
                saved = {
                    Brightness = Lighting2.Brightness,
                    ClockTime = Lighting2.ClockTime,
                    OutdoorAmbient = Lighting2.OutdoorAmbient,
                    GlobalShadows = Lighting2.GlobalShadows,
                    FogEnd = Lighting2.FogEnd,
                    EnvDiffuse = Lighting2.EnvironmentDiffuseScale,
                    EnvSpecular = Lighting2.EnvironmentSpecularScale,
                    effects = {},
                }
                for _, child in ipairs(Lighting2:GetChildren()) do
                    if child:IsA("PostEffect") then
                        saved.effects[child] = child.Enabled
                    end
                end
            end

            Lighting2.GlobalShadows = false
            Lighting2.FogEnd = 10000000000
            Lighting2.Brightness = 1
            Lighting2.EnvironmentDiffuseScale = 0
            Lighting2.EnvironmentSpecularScale = 0

            for _, child in ipairs(Lighting2:GetChildren()) do
                pcall(function()
                    if child:IsA("BlurEffect") or child:IsA("SunRaysEffect")
                        or child:IsA("ColorCorrectionEffect") or child:IsA("BloomEffect")
                        or child:IsA("DepthOfFieldEffect") then
                        child.Enabled = false
                    end
                end)
            end

            for _, descendant in ipairs(workspace:GetDescendants()) do
                applyAntiLagDerender(descendant)
            end

            if antiLagConn then antiLagConn:Disconnect() end
            antiLagConn = workspace.DescendantAdded:Connect(function(descendant)
                if antiLagRunning then
                    applyAntiLagDerender(descendant)
                end
            end)
        end

        local function disableAntiLag()
            antiLagRunning = false
            antiLagEnabled = false
            if antiLagConn then
                antiLagConn:Disconnect()
                antiLagConn = nil
            end
            pcall(function()
                if saved then
                    Lighting2.Brightness = saved.Brightness
                    Lighting2.ClockTime = saved.ClockTime
                    Lighting2.OutdoorAmbient = saved.OutdoorAmbient
                    Lighting2.GlobalShadows = saved.GlobalShadows
                    Lighting2.FogEnd = saved.FogEnd
                    Lighting2.EnvironmentDiffuseScale = saved.EnvDiffuse
                    Lighting2.EnvironmentSpecularScale = saved.EnvSpecular
                    for eff, wasEnabled in pairs(saved.effects) do
                        if eff and eff.Parent then
                            pcall(function() eff.Enabled = wasEnabled end)
                        end
                    end
                end
                Lighting2.ExposureCompensation = 0
            end)
        end

        _G.StickyToggleAntiLag = function(state)
            if state == nil then state = not antiLagEnabled end
            if state then enableAntiLag() else disableAntiLag() end
        end

        local function setAntiLag(enabled)
            enabled = enabled and true or false
            config.AntiLag = enabled
            saveConfig()
            setToggle("AntiLag", enabled)
            setToggle("Anti Lag", enabled)
            task.spawn(function() pcall(_G.StickyToggleAntiLag, enabled) end)
        end
        _G.setAntiLag = setAntiLag

        _G.StickyOnBoot(function()
            if config.AntiLag == true then pcall(setAntiLag, true) end
        end)

        local FOV_PRESETS = { 80, 120, 180 }
        local fovConnection = nil

        local function applyFOV()
            if fovConnection then
                fovConnection:Disconnect()
                fovConnection = nil
            end
            local target = tonumber(_G.StickyFOV)
            if not target then return end
            fovConnection = RunService.RenderStepped:Connect(function()
                local cam = workspace.CurrentCamera
                if cam and cam.FieldOfView ~= target then
                    cam.FieldOfView = target
                end
            end)
        end

        local function setFOV(value)
            _G.StickyFOV = tonumber(value) or 80
            config.FOV = _G.StickyFOV
            saveConfig()
            applyFOV()
            return _G.StickyFOV
        end
        _G.StickySetFOV = setFOV

        _G.StickyCycleFOV = function()
            local cur = tonumber(_G.StickyFOV) or FOV_PRESETS[1]
            local idx = 1
            for n, v in ipairs(FOV_PRESETS) do
                if v == cur then idx = n break end
            end
            idx = idx % #FOV_PRESETS + 1
            return setFOV(FOV_PRESETS[idx])
        end

        _G.StickyOnBoot(function()
            _G.StickyFOV = tonumber(config.FOV) or 80
            applyFOV()
        end)
    end
end
local function q()
    local c = C:WaitForChild("PlayerScripts")
    local z = require(c:WaitForChild("PlayerModule"))
    return z:GetControls()
end
local c = q()
local function c()
    local z = pcall(function()
        game:Shutdown()
    end)
    if z then
        return
    end
    pcall(function()
        C:Kick("")
    end)
end
local W = {
    Background = Color3.fromRGB(18, 8, 8),
    Surface = Color3.fromRGB(42, 17, 69),
    SurfaceHighlight = Color3.fromRGB(61, 24, 95),
    Accent1 = Color3.fromRGB(188, 120, 255),
    Accent2 = Color3.fromRGB(132, 54, 217),
    TextPrimary = Color3.fromRGB(240, 240, 240),
    TextSecondary = Color3.fromRGB(140, 140, 150),
    Success = Color3.fromRGB(132, 54, 217),
    Error = Color3.fromRGB(255, 60, 80),
}
local function F(E, m)
    local A = x:FindFirstChild("lMWjwEoSnCPj")
    if A then
        A:Destroy()
    end
    local A = Instance.new("ScreenGui", x)
    A.Name = "lMWjwEoSnCPj"
    A.ResetOnSpawn = false
    local H = Instance.new("Frame", A)
    H.Size = UDim2.new(0, 290, 0, 54)
    H.Position = UDim2.new(0.5, -145.0, 0, 80)
    H.BackgroundColor3 = Color3.fromRGB(6, 6, 12)
    H.BackgroundTransparency = 1
    H.BorderSizePixel = 0
    Instance.new("UICorner", H).CornerRadius = UDim.new(0, 9)
    local _ = Instance.new("UIStroke", H)
    _.Thickness = 1
    _.Color = W.Accent2
    _.Transparency = 1
    local Z = Instance.new("Frame", H)
    Z.Size = UDim2.new(0, 3, 1, -12.0)
    Z.Position = UDim2.new(0, 5, 0, 6)
    Z.BackgroundColor3 = W.Accent1
    Z.BorderSizePixel = 0
    Z.BackgroundTransparency = 1
    Instance.new("UICorner", Z).CornerRadius = UDim.new(1, 0)
    local k = Instance.new("TextLabel", H)
    k.Size = UDim2.new(1, -22.0, 0, 18)
    k.Position = UDim2.new(0, 16, 0, 7)
    k.BackgroundTransparency = 1
    k.Text = E:upper()
    k.Font = Enum.Font.GothamBlack
    k.TextSize = 11
    k.TextColor3 = W.Accent1
    k.TextXAlignment = Enum.TextXAlignment.Left
    k.TextTransparency = 1
    local E = Instance.new("TextLabel", H)
    E.Size = UDim2.new(1, -22.0, 0, 15)
    E.Position = UDim2.new(0, 16, 0, 27)
    E.BackgroundTransparency = 1
    E.Text = m or ""
    E.Font = Enum.Font.GothamMedium
    E.TextSize = 10
    E.TextColor3 = W.TextSecondary
    E.TextXAlignment = Enum.TextXAlignment.Left
    E.TextTransparency = 1
    local m = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    f:Create(H, m, { BackgroundTransparency = 0.08 }):Play()
    f:Create(_, m, { Transparency = 0.3 }):Play()
    f:Create(Z, m, { BackgroundTransparency = 0 }):Play()
    f:Create(k, m, { TextTransparency = 0 }):Play()
    f:Create(E, m, { TextTransparency = 0 }):Play()
    task.delay(2, function()
        if not A.Parent then
            return
        end
        local m = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        f:Create(H, m, { BackgroundTransparency = 1 }):Play()
        f:Create(_, m, { Transparency = 1 }):Play()
        f:Create(Z, m, { BackgroundTransparency = 1 }):Play()
        f:Create(k, m, { TextTransparency = 1 }):Play()
        local H = f:Create(E, m, { TextTransparency = 1 })
        H:Play()
        H.Completed:Wait()
        if A.Parent then
            A:Destroy()
        end
    end)
end
local function z(I, p)
    local l, G = pcall(function()
        if type(I) ~= "string" or I == "" then
            return p
        end
        local b = Enum.KeyCode[I]
        if b then
            return b
        end
        if #I == 1 then
            local CG = string.upper(I)
            b = Enum.KeyCode[CG]
            if b then
                return b
            end
        end
        return p
    end)
    return l and G or p
end
local I = 0
local X = {}
local function t()
    for v = #X, 1, -1.0 do
        X[v] = nil
    end
    for v, M in ipairs(i.PriorityList or {}) do
        X[v] = M
    end
end
local function v()
    i.PriorityList = {}
    for M, w in ipairs(X) do
        i.PriorityList[M] = w
    end
end
t()
local function M(w)
    if not w then
        return nil
    end
    local b = a:FindFirstChild("Plots") and a.Plots:FindFirstChild(w.plot)
    if b then
        local D = b:FindFirstChild("AnimalPodiums")
        if D then
            local b = D:FindFirstChild(w.slot)
            if b then
                local w = b:FindFirstChild("Base")
                if w then
                    local b = w:FindFirstChild("Spawn")
                    if b then
                        return b
                    end
                    return w:FindFirstChildWhichIsA("BasePart") or w
                end
            end
        end
    end
    return nil
end
_G.StickyFindAdornee = M
local j = game:GetService("UserInputService")
local function D(F, E, m)
    local A, H, _, Z
    F.InputBegan:Connect(function(k)
        if i.UILocked then
            return
        end
        if
            k.UserInputType == Enum.UserInputType.MouseButton1
            or k.UserInputType == Enum.UserInputType.Touch
        then
            A = true
            _ = k.Position
            Z = E.Position
            k.Changed:Connect(function()
                if k.UserInputState == Enum.UserInputState.End then
                    A = false
                    if m then
                        i.Positions[m] = {
                            X = E.Position.X.Scale,
                            Y = E.Position.Y.Scale,
                            OffsetX = E.Position.X.Offset,
                            OffsetY = E.Position.Y.Offset,
                        }
                        U()
                    end
                end
            end)
        end
    end)
    F.InputChanged:Connect(function(F)
        if
            F.UserInputType == Enum.UserInputType.MouseMovement
            or F.UserInputType == Enum.UserInputType.Touch
        then
            H = F
        end
    end)
    j.InputChanged:Connect(function(F)
        if F == H and A then
            local m = F.Position - _
            E.Position = UDim2.new(Z.X.Scale, Z.X.Offset + m.X, Z.Y.Scale, Z.Y.Offset + m.Y)
        end
    end)
end
task.spawn(function()
    local S = n:WaitForChild("Packages")
    local CG = n:WaitForChild("Datas")
    local rG = n:WaitForChild("Shared")
    local rG = n:WaitForChild("Utils")
    local rG = require(S:WaitForChild("Synchronizer"))
    local function S(dG)
        if not dG then
            return false
        end
        if typeof(dG) == "Instance" then
            return dG == C
        end
        if type(dG) == "string" then
            return dG == C.Name
        end
        return false
    end
    local function dG(nG)
        local fG, JG = pcall(function()
            local aG = getthreadidentity and getthreadidentity() or nil
            if setthreadidentity then
                setthreadidentity(8)
            end
            local eG = rG:GetTableFromChannel(nG)
            if aG and setthreadidentity then
                pcall(setthreadidentity, aG)
            end
            return eG
        end)
        if fG and type(JG) == "table" then
            return JG
        end
        return nil
    end
    local rG = require(CG:WaitForChild("Animals"))
    local nG = require(CG:WaitForChild("Mutations"))
    local fG = require(CG:WaitForChild("Traits"))
    local CG = {
        "",
        "K",
        "M",
        "B",
        "T",
        "Qa",
        "Qi",
        "Sx",
        "Sp",
        "Oc",
        "No",
        "Dc",
        "Ud",
        "Dd",
        "Td",
        "Qad",
        "Qid",
        "Sxd",
        "Spd",
        "Ocd",
        "Nod",
        "Vg",
        "Uvg",
        "Dvg",
        "Tvg",
    }
    local function JG(aG, eG)
        eG = eG or 1
        local xG = math.abs(aG)
        local QG = math.max(1, xG)
        local xG = math.floor(math.log(QG, 1000))
        local QG = CG[xG + 1] or "e+" .. xG
        local CG = aG * (10 ^ eG / 1000 ^ xG)
        local aG = math.floor(CG) / 10 ^ eG
        return (("%." .. eG .. "f"):format(aG)):gsub("%.?0+$", "") .. QG
    end
    local function CG(aG, eG, xG)
        local QG = rG[aG]
        if not QG then
            return 0
        end
        local aG = QG.Generation or QG.Price * 0.1
        local QG = 1
        if eG and eG ~= "None" then
            local VG = nG[eG]
            if VG and VG.Modifier then
                QG = QG + VG.Modifier
            end
        end
        local nG = false
        if type(xG) == "table" then
            for eG, eG in ipairs(xG) do
                if eG == "Sleepy" then
                    nG = true
                else
                    local xG = fG[eG]
                    if xG and xG.MultiplierModifier then
                        QG = QG + xG.MultiplierModifier
                    end
                end
            end
        end
        local fG = math.round(aG * QG)
        if nG then
            fG = math.round(fG * 0.5)
        end
        return fG
    end
    local nG = true
    local fG = i.StealNearest == true
    local aG = i.StealHighest == true
    local eG = i.StealPriority == true
    if eG then
        fG, aG = false, false
    elseif aG then
        fG = false
    end
    i.StealNearest = fG
    i.StealHighest = aG
    i.StealPriority = eG
    if i.InstantSteal == nil then
        i.InstantSteal = false
    end
    local xG = i.InstantSteal == true
    if xG and i.InstantStealV2 == true then
        xG = false
        i.InstantSteal = false
    end
    _G.NEAREST_INSTANT_MODE = (i.StealNearest == true and (xG or i.InstantStealV2 == true))
    local QG = false
    local VG = false
    local iG = 1
    local OG = nil
    local UG = nil
    local yG = {}
    local function KG()
        nG = fG == true or aG == true or eG == true
    end
    KG()
    local qG = {}
    local cG = {}
    local zG = nil
    local BG = nil
    local oG = {}
    local function YG(TG)
        if not TG or not TG.plot then
            return false
        end
        local IG = a:FindFirstChild("Plots")
        if not IG then
            return false
        end
        local pG = IG:FindFirstChild(TG.plot)
        if not pG then
            return false
        end
        local TG = dG(pG.Name)
        if TG then
            return S(TG.Owner)
        end
        return false
    end
    local function TG(IG)
        if not IG or IG == "None" then
            return ""
        end
        local pG = ""
        if IG == "Cursed" then
            pG = "<font color='rgb(200,0,0)'>Cur</font><font color='rgb(0,0,0)'>sed</font>"
        elseif IG == "Gold" then
            pG = "<font color='rgb(255,215,0)'>Gold</font>"
        elseif IG == "Diamond" then
            pG = "<font color='rgb(0,255,255)'>Diamond</font>"
        elseif IG == "YinYang" then
            pG = "<font color='rgb(255,255,255)'>Yin</font><font color='rgb(0,0,0)'>Yang</font>"
        elseif IG == "Candy" then
            pG = "<font color='rgb(255,105,180)'>Candy</font>"
        elseif IG == "Divine" then
            pG = "<font color='rgb(255,255,255)'>Divine</font>"
        elseif IG == "Rainbow" then
            local jG = {
                "rgb(255,0,0)",
                "rgb(255,127,0)",
                "rgb(255,255,0)",
                "rgb(0,255,0)",
                "rgb(0,0,255)",
                "rgb(75,0,130)",
                "rgb(148,0,211)",
            }
            for lG = 1, #IG do
                pG = pG .. "<font color='" .. jG[(lG - 1) % #jG + 1] .. "'>" .. IG:sub(lG, lG) .. "</font>"
            end
        elseif IG == "Radioactive" then
            pG = "<font color='rgb(132,255,0)'>Radioactive</font>"
        elseif IG == "Galaxy" then
            pG = "<font color='rgb(170,85,255)'>Galaxy</font>"
        else
            pG = IG
        end
        return "<font weight='800'>" .. pG .. " </font>"
    end
    local function IG(pG)
        if pG == "Gold" then
            return Color3.fromRGB(255, 215, 0)
        elseif pG == "Diamond" then
            return Color3.fromRGB(0, 255, 255)
        elseif pG == "Cursed" then
            return Color3.fromRGB(200, 0, 0)
        elseif pG == "YinYang" then
            return Color3.fromRGB(255, 255, 255)
        elseif pG == "Candy" then
            return Color3.fromRGB(255, 105, 180)
        elseif pG == "Divine" then
            return Color3.fromRGB(255, 255, 255)
        elseif pG == "Rainbow" then
            return Color3.fromRGB(148, 0, 211)
        elseif pG == "Radioactive" then
            return Color3.fromRGB(132, 255, 0)
        elseif pG == "Galaxy" then
            return Color3.fromRGB(170, 85, 255)
        end
        return Color3.fromRGB(255, 70, 120)
    end
    local function pG(jG)
        local lG = jG and jG.petName or "Unknown"
        return lG
    end
    local function jG(lG)
        local WG = lG and lG.mutation
        local head, rich = "", false
        if WG and WG ~= "None" and WG ~= "" then
            head, rich = TG(WG), true
        end
        if P.PetBadgeNames then
            local ok, _, traits = pcall(P.PetBadgeNames, lG)
            if ok and type(traits) == "table" and #traits > 0 then
                local joined = table.concat(traits, ", ")
                head = head ~= "" and (head .. "  |  " .. joined) or joined
            end
        end
        return head, rich
    end
    local function TG()
        local lG = {}
        for WG, WG in ipairs(yG) do
            if WG.genValue >= 1 and not YG(WG) then
                table.insert(
                    lG,
                    {
                        petName = WG.name,
                        mpsText = WG.genText,
                        mpsValue = WG.genValue,
                        owner = WG.owner,
                        plot = WG.plot,
                        slot = WG.slot,
                        uid = WG.uid,
                        mutation = WG.mutation,
                        animalData = WG,
                    }
                )
            end
        end
        return lG
    end
    local lG = (gethui and gethui()) or game:GetService("CoreGui")
    local WG = Instance.new("ScreenGui")
    WG.Name = "cJPcVLFpwxgI"
    WG.ResetOnSpawn = false
    WG.Parent = lG
    local XG = Instance.new("Frame")
    local tG = d and 0.6 or 1
    local GG = {
        BG = Color3.fromRGB(36, 16, 50),
        SURF = Color3.fromRGB(42, 17, 69),
        SURF2 = Color3.fromRGB(61, 24, 95),
        TEXT = Color3.fromRGB(255, 255, 255),
        DIM = Color3.fromRGB(224, 193, 255),
        AQUA = Color3.fromRGB(188, 120, 255),
        AQUA2 = Color3.fromRGB(132, 54, 217),
        AQUA_STROKE = Color3.fromRGB(186, 117, 245),
    }
    do
        local ps = i.PanelSize and i.PanelSize.AutoSteal
        local w = tonumber(ps and ps.W) or UI_PANEL_W
        local h = tonumber(ps and ps.H) or 412
        XG.Size = UDim2.fromOffset(w, h)
        if ps and ps.W and ps.H then P.AutoStealCustomSize = true end
    end
    XG.Position = UDim2.new(
        i.Positions.AutoSteal.X,
        i.Positions.AutoSteal.OffsetX or 0,
        i.Positions.AutoSteal.Y,
        i.Positions.AutoSteal.OffsetY or 0
    )
    P.AutoStealFrame = XG
    P.HoldPanel(XG)
    RegisterUIScale(XG)
    XG.BorderSizePixel = 0
    XG.ClipsDescendants = false
    XG.Parent = WG
    Instance.new("UICorner", XG).CornerRadius = UDim.new(0, 14)
    TGT.dressPanel(XG, 14)
    local stealTopbar = Instance.new("Frame", XG)
    stealTopbar.Name = "Topbar"
    stealTopbar.Size = UDim2.new(1, 0, 0, 44)
    stealTopbar.BackgroundTransparency = 1
    stealTopbar.Active = true
    stealTopbar.ZIndex = 104
    D(stealTopbar, XG, "AutoSteal")
    TGT.macDots(stealTopbar, 12, 16, 20, 105, XG)
    local vG = Instance.new("TextLabel", stealTopbar)
    vG.Size = UDim2.new(1, -132, 1, 0)
    vG.Position = UDim2.new(0, 84, 0, 0)
    vG.BackgroundTransparency = 1
    vG.Text = "Steal Targets"
    vG.Font = Enum.Font.GothamBold
    vG.TextSize = 15
    vG.TextColor3 = GG.TEXT
    vG.TextXAlignment = Enum.TextXAlignment.Left
    vG.ZIndex = 105
    local stealMinBtn = TGT.minimizeBtn(stealTopbar, 106)
    local stealDivider = Instance.new("Frame", XG)
    stealDivider.AnchorPoint = Vector2.new(0.5, 0)
    stealDivider.Position = UDim2.new(0.5, 0, 0, 44)
    stealDivider.Size = UDim2.new(1, -24, 0, 1)
    stealDivider.BackgroundColor3 = TGT.OUTLINE
    stealDivider.BackgroundTransparency = 0.55
    stealDivider.BorderSizePixel = 0
    stealDivider.ZIndex = 104
    local WG = Instance.new("ScrollingFrame", XG)
    WG.Size = UDim2.new(1, -20.0, 1, -62.0)
    WG.Position = UDim2.new(0, 10, 0, 52)
    WG.BackgroundTransparency = 1
    WG.BorderSizePixel = 0
    WG.ClipsDescendants = true
    WG.ScrollingDirection = Enum.ScrollingDirection.Y
    WG.ScrollingEnabled = true
    WG.Active = true
    WG.Selectable = false
    WG.AutomaticCanvasSize = Enum.AutomaticSize.None
    WG.ElasticBehavior = Enum.ElasticBehavior.WhenScrollable
    WG.VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar
    WG.ScrollBarImageColor3 = GG.AQUA_STROKE
    WG.ScrollBarImageTransparency = 0.15
    WG.ScrollBarThickness = 6
    WG.CanvasSize = UDim2.new(0, 0, 0, 0)
    do
        local panel = P.AutoStealFrame
        local listScroll = WG
        local fullH = panel.Size.Y.Offset
        local minimized = false
        stealMinBtn.MouseButton1Click:Connect(function()
            minimized = not minimized
            if minimized then fullH = panel.Size.Y.Offset end
            listScroll.Visible = not minimized
            stealDivider.Visible = not minimized
            panel.Size = UDim2.fromOffset(panel.Size.X.Offset, minimized and 44 or fullH)
            stealMinBtn.Text = minimized and "+" or "-"
        end)
    end
    local XG = Instance.new("Frame", WG)
    XG.Name = "Holder"
    XG.BackgroundTransparency = 1
    XG.BorderSizePixel = 0
    XG.Position = UDim2.new(0, 0, 0, 0)
    XG.Size = UDim2.new(1, 0, 0, 0)
    XG.ClipsDescendants = false
    local vG = Instance.new("UIListLayout", XG)
    vG.Padding = UDim.new(0, 4)
    vG.SortOrder = Enum.SortOrder.LayoutOrder
    for _, host in ipairs({ x, lG }) do
        local old = host and host:FindFirstChild("bRXgmsaEVfze")
        if old then pcall(function() old:Destroy() end) end
    end
    local MG = Instance.new("ScreenGui")
    MG.Name = "bRXgmsaEVfze"
    MG.ResetOnSpawn = false
    MG.IgnoreGuiInset = true
    MG.DisplayOrder = 2147483646
    MG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    MG.Parent = lG
    local wG = {
        PANEL = Color3.fromRGB(42, 17, 69),
        PANEL2 = Color3.fromRGB(61, 24, 95),
        TEXT = Color3.fromRGB(255, 255, 255),
        STROKE = Color3.fromRGB(186, 117, 245),
        GLOW = Color3.fromRGB(188, 120, 255),
        TRACK = Color3.fromRGB(46, 20, 72),
        TRACK2 = Color3.fromRGB(61, 24, 95),
        FILL1 = Color3.fromRGB(188, 120, 255),
        FILL2 = Color3.fromRGB(224, 193, 255),
    }
    local bG = Instance.new("Frame", MG)
    bG.Name = "CurrentTargetHUD"
    bG.AnchorPoint = Vector2.new(0.5, 1)
    bG.Size = UDim2.new(0, 180 * tG, 0, 48 * tG)
    local _dev = deviceClass()
    local _touch = _dev ~= "desktop"
    local _barY = -150.0
    if _dev == "phone" then
        _barY = -100.0
    elseif _dev == "tablet" then
        _barY = -144.0
    end
    bG.Position = UDim2.new(0.5, 0, 1, _barY)
    bG.BackgroundColor3 = wG.PANEL
    bG.BackgroundTransparency = 0.02
    bG.BorderSizePixel = 0
    bG.ZIndex = 70
    local hudScale = Instance.new("UIScale", bG)
    hudScale.Scale = _touch and 1.0 or 1.25
    Instance.new("UICorner", bG).CornerRadius = UDim.new(0, math.floor(10 * tG))
    local MG = Instance.new("UIStroke", bG)
    MG.Color = wG.STROKE
    MG.Thickness = 1
    MG.Transparency = 0.35
    local MG = Instance.new("UIStroke", bG)
    MG.Color = wG.GLOW
    MG.Thickness = 3
    MG.Transparency = 0.84
    MG.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local MG = Instance.new("TextLabel", bG)
    MG.Name = "TargetName"
    MG.Size = UDim2.new(1, -10.0, 0, 16 * tG)
    MG.Position = UDim2.fromOffset(5 * tG, 4 * tG)
    MG.BackgroundTransparency = 1
    MG.Font = Enum.Font.GothamBold
    MG.TextSize = 10 * tG
    MG.TextColor3 = wG.TEXT
    MG.TextXAlignment = Enum.TextXAlignment.Center
    MG.TextTruncate = Enum.TextTruncate.AtEnd
    MG.ZIndex = 72
    MG.Text = "No target"
    local DG = Instance.new("Frame", bG)
    DG.Name = "ProgressBg"
    DG.Size = UDim2.new(1, -8.0 * tG, 0, 16 * tG)
    DG.Position = UDim2.fromOffset(4 * tG, 22 * tG)
    DG.BackgroundColor3 = wG.TRACK
    DG.BorderSizePixel = 0
    DG.ZIndex = 72
    Instance.new("UICorner", DG).CornerRadius = UDim.new(0, math.floor(6 * tG))
    local bG = Instance.new("UIStroke", DG)
    bG.Color = wG.STROKE
    bG.Thickness = 1
    bG.Transparency = 0.55
    local bG = Instance.new("Frame", DG)
    bG.Name = "InnerTrack"
    bG.Size = UDim2.new(1, -2.0, 1, -2.0)
    bG.Position = UDim2.fromOffset(1, 1)
    bG.BackgroundColor3 = wG.TRACK2
    bG.BackgroundTransparency = 0.15
    bG.BorderSizePixel = 0
    bG.ZIndex = 72
    Instance.new("UICorner", bG).CornerRadius = UDim.new(0, math.floor(5 * tG))
    local bG = Instance.new("Frame", DG)
    bG.Name = "ProgressFill"
    bG.Size = UDim2.new(0, 0, 1, 0)
    bG.BackgroundColor3 = wG.FILL1
    bG.BorderSizePixel = 0
    bG.ZIndex = 73
    Instance.new("UICorner", bG).CornerRadius = UDim.new(0, math.floor(6 * tG))
    local FG = Instance.new("UIGradient", bG)
    FG.Color =
        ColorSequence.new({ ColorSequenceKeypoint.new(0, wG.FILL1), ColorSequenceKeypoint.new(1, wG.FILL2) })
    local FG = Instance.new("UIStroke", bG)
    FG.Color = Color3.fromRGB(255, 130, 130)
    FG.Thickness = 1
    FG.Transparency = 0.45
    local FG = Instance.new("TextLabel", DG)
    FG.Name = "Percent"
    FG.Size = UDim2.new(1, 0, 1, 0)
    FG.BackgroundTransparency = 1
    FG.Font = Enum.Font.GothamBold
    FG.TextSize = 10 * tG
    FG.TextColor3 = wG.TEXT
    FG.TextStrokeTransparency = 0.7
    FG.TextXAlignment = Enum.TextXAlignment.Center
    FG.ZIndex = 74
    FG.Text = "0%"
    local wG = bG
    do
        for _, host in ipairs({ x, lG }) do
            local old = host and host:FindFirstChild("SxTopLayer")
            if old then pcall(function() old:Destroy() end) end
        end
        local bannerGui = Instance.new("ScreenGui")
        bannerGui.Name = "SxTopLayer"
        bannerGui.ResetOnSpawn = false
        bannerGui.IgnoreGuiInset = true
        bannerGui.DisplayOrder = 2147483647
        bannerGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        bannerGui.Parent = lG

        local stickyBar = Instance.new("Frame", bannerGui)
        stickyBar.Name = "StickyBar"
        stickyBar.AnchorPoint = Vector2.new(0.5, 1)
        stickyBar.AutomaticSize = Enum.AutomaticSize.X
        stickyBar.Size = UDim2.new(0, 0, 0, 38)
        local _sdev = deviceClass()
        stickyBar.Position = UDim2.new(0.5, 0, 1, _sdev == "phone" and -62 or -96)
        stickyBar.BackgroundColor3 = Color3.fromRGB(27, 10, 45)
        stickyBar.BackgroundTransparency = 0.18
        stickyBar.BorderSizePixel = 0
        stickyBar.ClipsDescendants = false
        stickyBar.ZIndex = 70
        Instance.new("UICorner", stickyBar).CornerRadius = UDim.new(0, 19)
        P.StripFrame = stickyBar
        P.HoldPanel(stickyBar)
        local stripScale = Instance.new("UIScale", stickyBar)
        stripScale.Scale = (_sdev == "phone" and 0.62) or (_sdev == "tablet" and 0.80) or 1.0

        if _sdev ~= "desktop" then
            local menuBtn = Instance.new("TextButton", bannerGui)
            menuBtn.Name = "MenuToggle"
            menuBtn.AnchorPoint = Vector2.new(1, 0.5)
            menuBtn.Position = UDim2.new(1, -12, 0.5, 0)
            menuBtn.Size = UDim2.fromOffset(46, 46)
            menuBtn.BackgroundColor3 = Color3.fromRGB(42, 17, 69)
            menuBtn.BackgroundTransparency = 0.12
            menuBtn.BorderSizePixel = 0
            menuBtn.AutoButtonColor = false
            menuBtn.Font = Enum.Font.GothamBlack
            menuBtn.Text = "x"
            menuBtn.TextSize = 20
            menuBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            menuBtn.ZIndex = 80
            Instance.new("UICorner", menuBtn).CornerRadius = UDim.new(1, 0)
            TGT.panelGrad(menuBtn)
            local menuStroke = Instance.new("UIStroke", menuBtn)
            menuStroke.Color = TGT.OUTLINE
            menuStroke.Thickness = 1.4
            menuStroke.Transparency = 0.18
            menuStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            TGT.strokeGrad(menuStroke)
            local menuBtnScale = Instance.new("UIScale", menuBtn)
            menuBtnScale.Scale = (_sdev == "phone") and 0.8 or 1.0
            P.MenuButton = menuBtn
            P.HoldPanel(menuBtn)
            menuBtn.MouseButton1Click:Connect(function()
                P.ToggleUI()
            end)
        end
        local sStroke = Instance.new("UIStroke", stickyBar)
        sStroke.Color = TGT.OUTLINE
        sStroke.Thickness = 1.2
        sStroke.Transparency = 0.3
        sStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        TGT.strokeGrad(sStroke)
        local sPad = Instance.new("UIPadding", stickyBar)
        sPad.PaddingLeft = UDim.new(0, 14)
        sPad.PaddingRight = UDim.new(0, 14)
        local sLayout = Instance.new("UIListLayout", stickyBar)
        sLayout.FillDirection = Enum.FillDirection.Horizontal
        sLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        sLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        sLayout.SortOrder = Enum.SortOrder.LayoutOrder
        sLayout.Padding = UDim.new(0, 10)

        local function stripSep(order)
            local sp = Instance.new("Frame", stickyBar)
            sp.LayoutOrder = order
            sp.Size = UDim2.new(0, 1, 0, 18)
            sp.BackgroundColor3 = TGT.OUTLINE
            sp.BackgroundTransparency = 0.5
            sp.BorderSizePixel = 0
            sp.ZIndex = 72
            return sp
        end

        local function stripStat(order, caption, colour)
            local holder = Instance.new("Frame", stickyBar)
            holder.LayoutOrder = order
            holder.Size = UDim2.fromOffset(42, 30)
            holder.BackgroundTransparency = 1
            holder.ZIndex = 72
            local cap = Instance.new("TextLabel", holder)
            cap.Size = UDim2.new(1, 0, 0, 11)
            cap.BackgroundTransparency = 1
            cap.Font = Enum.Font.GothamBold
            cap.Text = caption
            cap.TextSize = 9
            cap.TextColor3 = Color3.fromRGB(224, 193, 255)
            cap.TextTransparency = 0.35
            cap.ZIndex = 73
            local val = Instance.new("TextLabel", holder)
            val.Position = UDim2.new(0, 0, 0, 12)
            val.Size = UDim2.new(1, 0, 0, 17)
            val.BackgroundTransparency = 1
            val.Font = Enum.Font.GothamBlack
            val.Text = "--"
            val.TextSize = 14
            val.TextColor3 = colour
            val.ZIndex = 73
            return val
        end

        local dot = Instance.new("Frame", stickyBar)
        dot.LayoutOrder = 1
        dot.Size = UDim2.fromOffset(9, 9)
        dot.BackgroundColor3 = Color3.fromRGB(188, 120, 255)
        dot.BorderSizePixel = 0
        dot.ZIndex = 72
        Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

        local discordLabel = Instance.new("TextLabel", stickyBar)
        discordLabel.LayoutOrder = 2
        discordLabel.AutomaticSize = Enum.AutomaticSize.X
        discordLabel.Size = UDim2.new(0, 0, 0, 20)
        discordLabel.BackgroundTransparency = 1
        discordLabel.Font = Enum.Font.GothamBlack
        discordLabel.TextSize = 14
        discordLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        discordLabel.TextTruncate = Enum.TextTruncate.None
        discordLabel.TextWrapped = false
        discordLabel.ClipsDescendants = false
        discordLabel.ZIndex = 72
        discordLabel.Text = "DISCORD.GG/Z7VPRXECNJ"

        stripSep(3)
        local fpsValue = stripStat(4, "FPS", TGT.FPS)
        stripSep(5)
        local pingValue = stripStat(6, "PING", TGT.PING)

        task.spawn(function()
            local frames = 0
            local mark = os.clock()
            local conn = game:GetService("RunService").RenderStepped:Connect(function()
                frames = frames + 1
            end)
            while stickyBar.Parent do
                task.wait(1)
                local now = os.clock()
                local span = now - mark
                if span > 0 then
                    fpsValue.Text = tostring(math.floor(frames / span + 0.5))
                end
                frames = 0
                mark = now
                local ok, ms = pcall(function()
                    return game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()
                end)
                pingValue.Text = ok and (tostring(math.floor(ms + 0.5)) .. "ms") or "--"
            end
            pcall(function() conn:Disconnect() end)
        end)
    end
    local DG = x:FindFirstChild("kzBUJxAKwhtf")
    if DG then
        DG:Destroy()
    end
    local DG = Instance.new("ScreenGui")
    DG.Name = "kzBUJxAKwhtf"
    DG.ResetOnSpawn = false
    DG.IgnoreGuiInset = true
    DG.DisplayOrder = 999
    DG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    DG.Parent = lG
    local lG = Instance.new("Frame", DG)
    lG.Name = "TargetControlsFrame"
    P.TargetControlsFrame = lG
    P.HoldPanel(lG)
    local TC_PANEL_H = 406
    local TC_CONTENT_H = TC_PANEL_H - 50
    lG.AutomaticSize = Enum.AutomaticSize.None
    lG.Size = UDim2.new(0, UI_PANEL_W, 0, TC_PANEL_H)
    RegisterUIScale(lG)
    local tG = i.Positions.TargetControls and i.Positions.TargetControls.X
    local DG = i.Positions.TargetControls and i.Positions.TargetControls.Y
    local EG = i.Positions.TargetControls and i.Positions.TargetControls.OffsetX
    local mG = i.Positions.TargetControls and i.Positions.TargetControls.OffsetY
    if tG == nil or DG == nil then
        tG = i.Positions.AutoSteal.X + 0.28
        DG = i.Positions.AutoSteal.Y
        if tG > 0.78 then
            tG = math.max(0.02, i.Positions.AutoSteal.X - 0.28)
        end
        if DG > 0.72 then
            DG = 0.72
        end
        i.Positions.TargetControls = { X = tG, Y = DG }
    end
    lG.Position = UDim2.new(tG or 0.26, EG or 10, DG or 0.35, mG or 0)
    lG.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
    lG.BackgroundTransparency = 0
    lG.BorderSizePixel = 0
    lG.ClipsDescendants = false
    lG.ZIndex = 100
    local b = {
        BG = Color3.fromRGB(36, 16, 50),
        SURF = Color3.fromRGB(42, 17, 69),
        SURF2 = Color3.fromRGB(61, 24, 95),
        TEXT = Color3.fromRGB(255, 255, 255),
        DIM = Color3.fromRGB(224, 193, 255),
        AQUA = Color3.fromRGB(188, 120, 255),
        AQUA2 = Color3.fromRGB(132, 54, 217),
        AQUA_STROKE = Color3.fromRGB(186, 117, 245),
        GREEN1 = Color3.fromRGB(132, 54, 217),
        GREEN2 = Color3.fromRGB(151, 80, 216),
        GREEN_STROKE = Color3.fromRGB(188, 120, 255),
        OFF_BG = Color3.fromRGB(36, 16, 50),
        OFF_TEXT = Color3.fromRGB(224, 193, 255),
    }
    local function tG(DG, EG)
        local mG = Instance.new("UICorner")
        mG.CornerRadius = UDim.new(0, EG)
        mG.Parent = DG
        return mG
    end
    local function DG(EG, mG, AG, HG)
        local ZG = Instance.new("UIStroke")
        ZG.Color = mG
        ZG.Thickness = AG or 1
        ZG.Transparency = HG or 0
        ZG.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        ZG.Parent = EG
        return ZG
    end
    local function EG(mG, AG, HG, ZG, kG)
        f:Create(
            mG,
            TweenInfo.new(AG or 0.2, ZG or Enum.EasingStyle.Quint, kG or Enum.EasingDirection.Out),
            HG
        ):Play()
    end
    local function mG(AG, HG, ZG, kG)
        local uG = Instance.new("UIGradient")
        uG.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, HG), ColorSequenceKeypoint.new(1, ZG) })
        uG.Rotation = kG or 0
        uG.Parent = AG
        return uG
    end
    tG(lG, 16)
    TGT.dressPanel(lG, 16)
    local AG = Instance.new("Frame", lG)
    AG.Name = "Topbar"
    AG.Size = UDim2.new(1, 0, 0, 44)
    AG.BackgroundTransparency = 1
    AG.Active = true
    AG.ZIndex = 101
    D(AG, lG, "TargetControls")
    TGT.macDots(AG, 12, 16, 20, 103, lG)
    local HG = Instance.new("TextLabel", AG)
    HG.Size = UDim2.new(1, -132, 1, 0)
    HG.Position = UDim2.new(0, 84, 0, 0)
    HG.ZIndex = 102
    HG.BackgroundTransparency = 1
    HG.Text = "Target Controls"
    HG.Font = Enum.Font.GothamBold
    HG.TextSize = 15
    HG.TextColor3 = b.TEXT
    HG.TextXAlignment = Enum.TextXAlignment.Left
    local tcMinBtn = TGT.minimizeBtn(AG, 104)
    local AG = Instance.new("Frame", lG)
    AG.AnchorPoint = Vector2.new(0.5, 0)
    AG.Position = UDim2.new(0.5, 0, 0, 44)
    AG.Size = UDim2.new(1, -24, 0, 1)
    AG.BackgroundColor3 = TGT.OUTLINE
    AG.BackgroundTransparency = 0.55
    AG.BorderSizePixel = 0
    AG.ZIndex = 101
    local tcDivider = AG
    local tabContent = Instance.new("Frame", lG)
    tabContent.Name = "TabContent"
    tabContent.Size = UDim2.new(1, -16, 0, TC_CONTENT_H)
    tabContent.Position = UDim2.fromOffset(8, 50)
    tabContent.BackgroundTransparency = 1
    tabContent.ClipsDescendants = true
    tabContent.ZIndex = 101
    do
        local panel = P.TargetControlsFrame
        local minimized = false
        tcMinBtn.MouseButton1Click:Connect(function()
            minimized = not minimized
            tabContent.Visible = not minimized
            panel.Size = UDim2.new(0, panel.Size.X.Offset, 0, minimized and 44 or TC_PANEL_H)
            tcMinBtn.Text = minimized and "+" or "-"
        end)
    end
    local AG = Instance.new("ScrollingFrame", tabContent)
    AG.Name = "MainPage"
    AG.Size = UDim2.fromScale(1, 1)
    AG.BackgroundColor3 = b.SURF
    AG.BorderSizePixel = 0
    AG.ZIndex = 101
    AG.ClipsDescendants = true
    AG.ScrollBarThickness = 4
    AG.ScrollBarImageColor3 = b.AQUA2
    AG.CanvasSize = UDim2.new(0, 0, 0, 0)
    AG.AutomaticCanvasSize = Enum.AutomaticSize.Y
    tG(AG, 14)
    DG(AG, b.AQUA_STROKE, 1, 0.48)
    local lG = Instance.new("Frame", AG)
    lG.AutomaticSize = Enum.AutomaticSize.Y
    lG.Size = UDim2.new(1, -8.0, 0, 0)
    lG.Position = UDim2.fromOffset(4, 4)
    lG.BackgroundTransparency = 1
    lG.ZIndex = 102
    local AG = Instance.new("UIListLayout")
    AG.Padding = UDim.new(0, 5)
    AG.SortOrder = Enum.SortOrder.LayoutOrder
    AG.Parent = lG
    local AG = 1
    local function HG(parent, titleText, order)
        local row = Instance.new("Frame", parent)
        row.Name = titleText:gsub("%s+", "") .. "Row"
        row.Size = UDim2.new(1, 0, 0, 44)
        row.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        row.BackgroundTransparency = 0.61
        row.BorderSizePixel = 0
        row.LayoutOrder = order or 0
        row.ZIndex = 103
        tG(row, 10)
        TGT.grad(row, {
            { 0, Color3.fromRGB(55, 20, 73), 0.72 },
            { 0.5, Color3.fromRGB(151, 80, 216), 0.61 },
            { 1, Color3.fromRGB(36, 16, 50), 0.74 },
        }, 90)
        local rowStroke = DG(row, TGT.OUTLINE, 1, 0.62)

        local label = Instance.new("TextLabel", row)
        label.BackgroundTransparency = 1
        label.Position = UDim2.fromOffset(13, 0)
        label.Size = UDim2.new(1, -66, 1, 0)
        label.Font = Enum.Font.GothamBold
        label.Text = titleText
        label.TextColor3 = b.TEXT
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.TextTruncate = Enum.TextTruncate.AtEnd
        label.ZIndex = 104

        local track = Instance.new("TextButton", row)
        track.Name = titleText:gsub("%s+", "") .. "Toggle"
        track.AutoButtonColor = false
        track.AnchorPoint = Vector2.new(1, 0.5)
        track.Position = UDim2.new(1, -11, 0.5, 0)
        track.Size = UDim2.fromOffset(40, 21)
        track.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
        track.BackgroundTransparency = 0.25
        track.BorderSizePixel = 0
        track.Text = ""
        track.ZIndex = 104
        tG(track, 11)
        local trackStroke = DG(track, TGT.OUTLINE, 1, 0.45)

        local fill = Instance.new("Frame", track)
        fill.Name = "Fill"
        fill.Size = UDim2.new(1, 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(132, 54, 217)
        fill.BackgroundTransparency = 1
        fill.BorderSizePixel = 0
        fill.ZIndex = 104
        tG(fill, 11)
        local fillGrad = TGT.grad(fill, {
            { 0, Color3.fromRGB(132, 54, 217) },
            { 0.5, Color3.fromRGB(188, 120, 255) },
            { 1, Color3.fromRGB(84, 32, 138) },
        }, 45)
        fillGrad.Enabled = false

        local dot = Instance.new("Frame", track)
        dot.Name = "Dot"
        dot.AnchorPoint = Vector2.new(0, 0.5)
        dot.Position = UDim2.new(0, 3, 0.5, 0)
        dot.Size = UDim2.fromOffset(16, 16)
        dot.BackgroundColor3 = b.TEXT
        dot.BorderSizePixel = 0
        dot.ZIndex = 106
        tG(dot, 8)

        fill:GetPropertyChangedSignal("BackgroundTransparency"):Connect(function()
            local on = fill.BackgroundTransparency < 0.5
            fillGrad.Enabled = on
            dot.Position = on and UDim2.new(1, -19, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
        end)

        local stateLabel = Instance.new("TextLabel", row)
        stateLabel.BackgroundTransparency = 1
        stateLabel.AnchorPoint = Vector2.new(1, 0.5)
        stateLabel.Position = UDim2.new(1, -59, 0.5, 0)
        stateLabel.Size = UDim2.fromOffset(30, 16)
        stateLabel.Font = Enum.Font.GothamBold
        stateLabel.TextSize = 10
        stateLabel.Text = "OFF"
        stateLabel.TextColor3 = b.OFF_TEXT
        stateLabel.TextXAlignment = Enum.TextXAlignment.Right
        stateLabel.ZIndex = 104

        row.MouseEnter:Connect(function()
            EG(rowStroke, 0.14, { Transparency = 0.38 })
        end)
        row.MouseLeave:Connect(function()
            EG(rowStroke, 0.14, { Transparency = 0.62 })
        end)
        return {
            row = row, label = label, button = track, knob = fill,
            stateLabel = stateLabel, stroke = trackStroke, rowStroke = rowStroke,
        }
    end
    local ZG = HG(lG, "Nearest", 100)
    local kG = HG(lG, "Highest", 110)
    local uG = HG(lG, "Priority", 120)
    local LG
    LG = HG(lG, "Instant Steal", 130)
    local v2Row = HG(lG, "Instant Steal V2", 131)
    local v2On = i.InstantStealV2 == true
    local function actionRow(labelText, btnText, order, onPress)
        local row = Instance.new("Frame", lG)
        row.Name = labelText:gsub("%s+", "") .. "Row"
        row.Size = UDim2.new(1, 0, 0, 44)
        row.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        row.BackgroundTransparency = 0.61
        TGT.grad(row, { { 0, Color3.fromRGB(55, 20, 73), 0.72 }, { 0.5, Color3.fromRGB(151, 80, 216), 0.61 }, { 1, Color3.fromRGB(36, 16, 50), 0.74 } }, 90)
        row.BorderSizePixel = 0
        row.LayoutOrder = order
        row.ZIndex = 103
        tG(row, 10)
        local rowStroke = DG(row, TGT.OUTLINE, 1, 0.62)
        local lbl = Instance.new("TextLabel", row)
        lbl.BackgroundTransparency = 1
        lbl.Position = UDim2.fromOffset(10, 0)
        lbl.Size = UDim2.new(1, -90.0, 1, 0)
        lbl.Font = Enum.Font.GothamBold
        lbl.Text = labelText
        lbl.TextColor3 = b.TEXT
        lbl.TextSize = 13 * AG
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.ZIndex = 104
        local btn = Instance.new("TextButton", row)
        btn.Name = labelText:gsub("%s+", "") .. "Button"
        btn.AutoButtonColor = false
        btn.Size = UDim2.fromOffset(math.floor(72 * AG), math.floor(26 * AG))
        btn.Position = UDim2.new(1, -math.floor(80 * AG), 0.5, -math.floor(13 * AG))
        btn.BackgroundColor3 = b.AQUA2
        btn.BorderSizePixel = 0
        btn.Text = btnText
        btn.TextColor3 = b.TEXT
        btn.TextSize = 11 * AG
        btn.Font = Enum.Font.GothamBold
        btn.ZIndex = 104
        tG(btn, 6)
        DG(btn, b.AQUA_STROKE, 1, 0.55)
        row.MouseEnter:Connect(function()
            EG(row, 0.14, { BackgroundColor3 = Color3.fromRGB(170, 100, 232) })
            EG(rowStroke, 0.14, { Transparency = 0.38 })
        end)
        row.MouseLeave:Connect(function()
            EG(row, 0.14, { BackgroundColor3 = Color3.fromRGB(151, 80, 216) })
            EG(rowStroke, 0.14, { Transparency = 0.62 })
        end)
        btn.MouseButton1Click:Connect(function()
            btn.Text = "..."
            task.spawn(function()
                pcall(onPress)
                task.wait(0.35)
                btn.Text = btnText
            end)
        end)
        return row, btn
    end
    actionRow("Drop Brainrot", "DROP", 220, function()
        if type(_G.StickyDropBrainrot) == "function" then _G.StickyDropBrainrot() end
    end)
    actionRow("Instant Reset", "RESET", 230, function()
        if type(_G.StickyInstaReset) == "function" then _G.StickyInstaReset() end
    end)
    actionRow("Insta Clone", "CLONE", 240, function()
        if _G._stickyFastClone then _G._stickyFastClone() end
    end)
    do
        local carpetMainRow = HG(lG, "Carpet Spd", 140)
        local function paintCarpet(on)
            if on then
                carpetMainRow.button.BackgroundColor3 = b.GREEN1
                carpetMainRow.knob.BackgroundTransparency = 0
                carpetMainRow.stateLabel.Text = "ON"
                carpetMainRow.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                carpetMainRow.stroke.Color = b.GREEN_STROKE
                carpetMainRow.stroke.Transparency = 0.22
            else
                carpetMainRow.button.BackgroundColor3 = b.OFF_BG
                carpetMainRow.knob.BackgroundTransparency = 1
                carpetMainRow.stateLabel.Text = "OFF"
                carpetMainRow.stateLabel.TextColor3 = b.OFF_TEXT
                carpetMainRow.stroke.Color = b.AQUA_STROKE
                carpetMainRow.stroke.Transparency = 0.55
            end
            if carpetMainRow.rowStroke then
                carpetMainRow.rowStroke.Transparency = on and 0.38 or 0.62
            end
        end
        paintCarpet(_G.StickyCarpetSpeed == true)
        P.ToggleUIPainters["CarpetSpeed"] = paintCarpet
        P.ToggleUIPainters["Carpet Speed"] = paintCarpet
        carpetMainRow.button.MouseButton1Click:Connect(function()
            local on = not (_G.StickyCarpetSpeed == true)
            if _G.StickySetCarpetSpeed then _G.StickySetCarpetSpeed(on) end
            paintCarpet(on)
            if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
        end)
        _G.StickyOnBoot(function()
            paintCarpet(_G.StickyCarpetSpeed == true)
        end)
    end
    do
        local boostRow = HG(lG, "Steal Boost", 150)
        local function paintBoost(on)
            if on then
                boostRow.button.BackgroundColor3 = b.GREEN1
                boostRow.knob.BackgroundTransparency = 0
                boostRow.stateLabel.Text = "ON"
                boostRow.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                boostRow.stroke.Color = b.GREEN_STROKE
                boostRow.stroke.Transparency = 0.22
            else
                boostRow.button.BackgroundColor3 = b.OFF_BG
                boostRow.knob.BackgroundTransparency = 1
                boostRow.stateLabel.Text = "OFF"
                boostRow.stateLabel.TextColor3 = b.OFF_TEXT
                boostRow.stroke.Color = b.AQUA_STROKE
                boostRow.stroke.Transparency = 0.55
            end
            if boostRow.rowStroke then
                boostRow.rowStroke.Transparency = on and 0.38 or 0.62
            end
        end
        paintBoost(i.StealBoost == true)
        P.ToggleUIPainters["StealBoost"] = paintBoost
        P.ToggleUIPainters["Steal Boost"] = paintBoost
        boostRow.button.MouseButton1Click:Connect(function()
            local on = not (i.StealBoost == true)
            paintBoost(on)
            if type(_G.setStealBoost) == "function" then
                task.spawn(_G.setStealBoost, on)
            else
                i.StealBoost = on; U()
            end
        end)
        _G.StickyOnBoot(function()
            paintBoost(i.StealBoost == true)
        end)
    end
    do
        local function paintV2(on)
            v2On = on and true or false
            if v2On and xG then
                xG = false
                QG = false
                VG = false
                i.InstantSteal = false
            end
            _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
            if v2Row then
                if v2On then
                    v2Row.stateLabel.Text = "ON"
                    v2Row.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    v2Row.button.BackgroundColor3 = b.GREEN1
                    v2Row.knob.BackgroundTransparency = 0
                    v2Row.stroke.Color = b.GREEN_STROKE
                    v2Row.stroke.Transparency = 0.22
                else
                    v2Row.stateLabel.Text = "OFF"
                    v2Row.stateLabel.TextColor3 = b.OFF_TEXT
                    v2Row.button.BackgroundColor3 = b.OFF_BG
                    v2Row.knob.BackgroundTransparency = 1
                    v2Row.stroke.Color = b.AQUA_STROKE
                    v2Row.stroke.Transparency = 0.55
                end
                if v2Row.rowStroke then
                    v2Row.rowStroke.Transparency = v2On and 0.38 or 0.62
                end
            end
        end
        P.ToggleUIPainters["InstantStealV2"] = paintV2
        P.ToggleUIPainters["Instant Steal V2"] = paintV2
    end
        local controlsFrame = P.TargetControlsFrame
        local stealFrame = P.AutoStealFrame
        if controlsFrame and stealFrame then
            local function clampOnScreen()
                local camera = workspace.CurrentCamera
                local viewport = camera and camera.ViewportSize
                if not viewport or viewport.X < 1 or viewport.Y < 1 then return end
                local keyFor = { [stealFrame] = "AutoSteal", [controlsFrame] = "TargetControls" }
                local moved = false
                for _, frame in ipairs({ stealFrame, controlsFrame }) do
                    if frame.Parent then
                        local size = frame.AbsoluteSize
                        if size.X > 1 and size.Y > 1 then
                            local pos = frame.Position
                            local x = pos.X.Scale * viewport.X + pos.X.Offset
                            local y = pos.Y.Scale * viewport.Y + pos.Y.Offset
                            local cx = math.clamp(x, 6, math.max(6, viewport.X - size.X - 6))
                            local cy = math.clamp(y, 6, math.max(6, viewport.Y - size.Y - 6))
                            if math.abs(cx - x) > 1 or math.abs(cy - y) > 1 then
                                frame.Position = UDim2.fromOffset(math.floor(cx), math.floor(cy))
                                local k = keyFor[frame]
                                if k and i.Positions then
                                    i.Positions[k] = {
                                        X = 0, Y = 0,
                                        OffsetX = math.floor(cx), OffsetY = math.floor(cy),
                                    }
                                    moved = true
                                end
                            end
                        end
                    end
                end
                if moved and _G.StickySaveConfigNow then
                    task.spawn(_G.StickySaveConfigNow)
                end
            end
            if _G.StickyOnBoot then
                _G.StickyOnBoot(clampOnScreen)
            else
                task.delay(1, clampOnScreen)
            end
        end
do -- Main UI Panel
    local muiRoot = x:FindFirstChild("StickyMainUI")
    if muiRoot then muiRoot:Destroy() end
    muiRoot = Instance.new("ScreenGui")
    muiRoot.Name = "StickyMainUI"
    muiRoot.ResetOnSpawn = false
    muiRoot.IgnoreGuiInset = true
    muiRoot.DisplayOrder = 1000
    muiRoot.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    muiRoot.Parent = (gethui and gethui()) or game:GetService("CoreGui")

    local MAIN_W, MAIN_H = 550, 450
    local SIDEBAR_W = 150
    local HEADER_H = 44
    local mc = {
        BG = Color3.fromRGB(36, 16, 50),
        SURF = Color3.fromRGB(42, 17, 69),
        SURF2 = Color3.fromRGB(61, 24, 95),
        TEXT = Color3.fromRGB(255, 255, 255),
        DIM = Color3.fromRGB(224, 193, 255),
        AQUA = Color3.fromRGB(188, 120, 255),
        AQUA2 = Color3.fromRGB(132, 54, 217),
        AQUA_STROKE = Color3.fromRGB(186, 117, 245),
        GREEN1 = Color3.fromRGB(132, 54, 217),
        GREEN2 = Color3.fromRGB(151, 80, 216),
        GREEN_STROKE = Color3.fromRGB(188, 120, 255),
        OFF_BG = Color3.fromRGB(36, 16, 50),
        OFF_TEXT = Color3.fromRGB(224, 193, 255),
    }
    local function mcCorner(obj, rad)
        local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, rad); c.Parent = obj; return c
    end
    local function mcStroke(obj, col, thick, transp)
        local s = Instance.new("UIStroke"); s.Color = col; s.Thickness = thick or 1; s.Transparency = transp or 0
        s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Parent = obj; return s
    end
    local function mcTween(obj, dur, props, style, dir)
        f:Create(obj, TweenInfo.new(dur or 0.2, style or Enum.EasingStyle.Quint, dir or Enum.EasingDirection.Out), props):Play()
    end
    local function mcGrad(obj, c1, c2, rot)
        local g = Instance.new("UIGradient")
        g.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, c1), ColorSequenceKeypoint.new(1, c2) })
        g.Rotation = rot or 0; g.Parent = obj; return g
    end

    local mainFrame = Instance.new("Frame", muiRoot)
    mainFrame.Name = "MainUIFrame"
    mainFrame.Size = UDim2.fromOffset(MAIN_W, MAIN_H)
    local muiPos = i.Positions.MainUI
    mainFrame.Position = UDim2.new(muiPos and muiPos.X or 0.5, muiPos and muiPos.OffsetX or -275, muiPos and muiPos.Y or 0.5, muiPos and muiPos.OffsetY or -225)
    mainFrame.BackgroundColor3 = mc.BG
    mainFrame.BackgroundTransparency = 0
    mainFrame.BorderSizePixel = 0
    mainFrame.ZIndex = 100
    mainFrame.Visible = false
    P.MainUIFrame = mainFrame
    RegisterUIScale(mainFrame)
    mcCorner(mainFrame, 16)
    TGT.dressPanel(mainFrame, 16)

    local header = Instance.new("Frame", mainFrame)
    header.Name = "Header"
    header.Size = UDim2.new(1, 0, 0, HEADER_H)
    header.BackgroundColor3 = mc.SURF
    header.BackgroundTransparency = 0.5
    header.BorderSizePixel = 0
    header.Active = true
    header.ZIndex = 101
    mcCorner(header, 16)
    D(header, mainFrame, "MainUI")
    TGT.macDots(header, 12, 16, 20, 103, mainFrame)
    local muiTitle = Instance.new("TextLabel", header)
    muiTitle.Size = UDim2.new(1, -132, 1, 0)
    muiTitle.Position = UDim2.new(0, 84, 0, 0)
    muiTitle.ZIndex = 102
    muiTitle.BackgroundTransparency = 1
    muiTitle.Text = "Sticky Loves You"
    muiTitle.Font = Enum.Font.GothamBold
    muiTitle.TextSize = 16
    muiTitle.TextColor3 = mc.TEXT
    muiTitle.TextXAlignment = Enum.TextXAlignment.Left
    local muiCloseBtn = TGT.minimizeBtn(header, 104)
    muiCloseBtn.Text = "x"
    muiCloseBtn.MouseButton1Click:Connect(function()
        P.SetUIHidden(true)
    end)

    local divider = Instance.new("Frame", mainFrame)
    divider.Size = UDim2.new(1, 0, 0, 1)
    divider.Position = UDim2.fromOffset(0, HEADER_H)
    divider.BackgroundColor3 = TGT.OUTLINE
    divider.BackgroundTransparency = 0.55
    divider.BorderSizePixel = 0
    divider.ZIndex = 101

    local sidebar = Instance.new("Frame", mainFrame)
    sidebar.Name = "Sidebar"
    sidebar.Size = UDim2.new(0, SIDEBAR_W, 1, -HEADER_H)
    sidebar.Position = UDim2.fromOffset(0, HEADER_H)
    sidebar.BackgroundColor3 = mc.BG
    sidebar.BackgroundTransparency = 0.3
    sidebar.BorderSizePixel = 0
    sidebar.ZIndex = 101
    sidebar.ClipsDescendants = true
    mcCorner(sidebar, 16)

    local sidebarPad = Instance.new("UIPadding", sidebar)
    sidebarPad.PaddingTop = UDim.new(0, 8)
    sidebarPad.PaddingBottom = UDim.new(0, 16)
    sidebarPad.PaddingLeft = UDim.new(0, 8)
    sidebarPad.PaddingRight = UDim.new(0, 8)
    local sidebarLayout = Instance.new("UIListLayout", sidebar)
    sidebarLayout.Padding = UDim.new(0, 4)
    sidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder

    local TAB_NAMES = { "Player", "ESP", "Misc", "Keybinds", "UI" }
    local tabButtons = {}
    for idx, name in ipairs(TAB_NAMES) do
        local tb = Instance.new("TextButton", sidebar)
        tb.Name = name .. "Tab"
        tb.Size = UDim2.new(1, 0, 0, 38)
        tb.BackgroundColor3 = mc.OFF_BG
        tb.BackgroundTransparency = 0.3
        tb.BorderSizePixel = 0
        tb.AutoButtonColor = false
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 13
        tb.Text = name
        tb.TextColor3 = mc.OFF_TEXT
        tb.LayoutOrder = idx
        tb.ZIndex = 102
        mcCorner(tb, 8)
        tabButtons[name] = tb
    end

    local sideDiv = Instance.new("Frame", mainFrame)
    sideDiv.Size = UDim2.new(0, 1, 1, -HEADER_H)
    sideDiv.Position = UDim2.fromOffset(SIDEBAR_W, HEADER_H)
    sideDiv.BackgroundColor3 = TGT.OUTLINE
    sideDiv.BackgroundTransparency = 0.55
    sideDiv.BorderSizePixel = 0
    sideDiv.ZIndex = 101

    local contentArea = Instance.new("Frame", mainFrame)
    contentArea.Name = "ContentArea"
    contentArea.Size = UDim2.new(1, -SIDEBAR_W, 1, -HEADER_H)
    contentArea.Position = UDim2.new(0, SIDEBAR_W, 0, HEADER_H)
    contentArea.BackgroundColor3 = mc.SURF
    contentArea.BackgroundTransparency = 0.4
    contentArea.BorderSizePixel = 0
    contentArea.ClipsDescendants = true
    contentArea.ZIndex = 101
    mcCorner(contentArea, 16)

    local function makeScrollPage(name)
        local page = Instance.new("ScrollingFrame", contentArea)
        page.Name = name
        page.Size = UDim2.fromScale(1, 1)
        page.BackgroundTransparency = 1
        page.BorderSizePixel = 0
        page.ScrollBarThickness = 4
        page.ScrollBarImageColor3 = mc.AQUA2
        page.CanvasSize = UDim2.new(0, 0, 0, 0)
        page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        page.Visible = false
        page.ZIndex = 102
        local inner = Instance.new("Frame", page)
        inner.AutomaticSize = Enum.AutomaticSize.Y
        inner.Size = UDim2.new(1, -16, 0, 0)
        inner.Position = UDim2.fromOffset(8, 8)
        inner.BackgroundTransparency = 1
        inner.ZIndex = 103
        local lay = Instance.new("UIListLayout", inner)
        lay.Padding = UDim.new(0, 5)
        lay.SortOrder = Enum.SortOrder.LayoutOrder
        return page, inner
    end

    local playerPage, playerInner = makeScrollPage("PlayerPage")
    local espPage, espInner = makeScrollPage("ESPPage")
    local miscPage, miscInner = makeScrollPage("MiscPage")
    local keybindsPage, keybindsInner = makeScrollPage("KeybindsPage")
    local uiPage, uiInner = makeScrollPage("UIPage")

    local pages = { Player = playerPage, ESP = espPage, Misc = miscPage, Keybinds = keybindsPage, UI = uiPage }
    local currentMainTab = "Player"

    local function selectMainTab(which)
        currentMainTab = which
        for name, page in pairs(pages) do
            page.Visible = (name == which)
        end
        for name, btn in pairs(tabButtons) do
            local active = (name == which)
            btn.BackgroundColor3 = active and mc.GREEN1 or mc.OFF_BG
            btn.BackgroundTransparency = active and 0 or 0.3
            btn.TextColor3 = active and mc.TEXT or mc.OFF_TEXT
        end
    end
    for name, btn in pairs(tabButtons) do
        btn.MouseButton1Click:Connect(function() selectMainTab(name) end)
    end
    selectMainTab("Player")

    local function muiToggleRow(parent, titleText, order)
        local row = Instance.new("Frame", parent)
        row.Name = titleText:gsub("%s+", "") .. "Row"
        row.Size = UDim2.new(1, 0, 0, 44)
        row.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        row.BackgroundTransparency = 0.61
        row.BorderSizePixel = 0
        row.LayoutOrder = order or 0
        row.ZIndex = 103
        mcCorner(row, 10)
        TGT.grad(row, {
            { 0, Color3.fromRGB(55, 20, 73), 0.72 },
            { 0.5, Color3.fromRGB(151, 80, 216), 0.61 },
            { 1, Color3.fromRGB(36, 16, 50), 0.74 },
        }, 90)
        local rowStroke = mcStroke(row, TGT.OUTLINE, 1, 0.62)
        local label = Instance.new("TextLabel", row)
        label.BackgroundTransparency = 1
        label.Position = UDim2.fromOffset(13, 0)
        label.Size = UDim2.new(1, -66, 1, 0)
        label.Font = Enum.Font.GothamBold
        label.Text = titleText
        label.TextColor3 = mc.TEXT
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.TextTruncate = Enum.TextTruncate.AtEnd
        label.ZIndex = 104
        local track = Instance.new("TextButton", row)
        track.Name = titleText:gsub("%s+", "") .. "Toggle"
        track.AutoButtonColor = false
        track.AnchorPoint = Vector2.new(1, 0.5)
        track.Position = UDim2.new(1, -11, 0.5, 0)
        track.Size = UDim2.fromOffset(40, 21)
        track.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
        track.BackgroundTransparency = 0.25
        track.BorderSizePixel = 0
        track.Text = ""
        track.ZIndex = 104
        mcCorner(track, 11)
        local trackStroke = mcStroke(track, TGT.OUTLINE, 1, 0.45)
        local fill = Instance.new("Frame", track)
        fill.Name = "Fill"
        fill.Size = UDim2.new(1, 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(132, 54, 217)
        fill.BackgroundTransparency = 1
        fill.BorderSizePixel = 0
        fill.ZIndex = 104
        mcCorner(fill, 11)
        local fillGrad = TGT.grad(fill, {
            { 0, Color3.fromRGB(132, 54, 217) },
            { 0.5, Color3.fromRGB(188, 120, 255) },
            { 1, Color3.fromRGB(84, 32, 138) },
        }, 45)
        fillGrad.Enabled = false
        local dot = Instance.new("Frame", track)
        dot.Name = "Dot"
        dot.AnchorPoint = Vector2.new(0, 0.5)
        dot.Position = UDim2.new(0, 3, 0.5, 0)
        dot.Size = UDim2.fromOffset(16, 16)
        dot.BackgroundColor3 = mc.TEXT
        dot.BorderSizePixel = 0
        dot.ZIndex = 106
        mcCorner(dot, 8)
        fill:GetPropertyChangedSignal("BackgroundTransparency"):Connect(function()
            local on = fill.BackgroundTransparency < 0.5
            fillGrad.Enabled = on
            dot.Position = on and UDim2.new(1, -19, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
        end)
        local stateLabel = Instance.new("TextLabel", row)
        stateLabel.BackgroundTransparency = 1
        stateLabel.AnchorPoint = Vector2.new(1, 0.5)
        stateLabel.Position = UDim2.new(1, -59, 0.5, 0)
        stateLabel.Size = UDim2.fromOffset(30, 16)
        stateLabel.Font = Enum.Font.GothamBold
        stateLabel.TextSize = 10
        stateLabel.Text = "OFF"
        stateLabel.TextColor3 = mc.OFF_TEXT
        stateLabel.TextXAlignment = Enum.TextXAlignment.Right
        stateLabel.ZIndex = 104
        row.MouseEnter:Connect(function() mcTween(rowStroke, 0.14, { Transparency = 0.38 }) end)
        row.MouseLeave:Connect(function() mcTween(rowStroke, 0.14, { Transparency = 0.62 }) end)
        return {
            row = row, label = label, button = track, knob = fill,
            stateLabel = stateLabel, stroke = trackStroke, rowStroke = rowStroke,
        }
    end

    local function muiPaintRow(entry, on)
        if not entry then return end
        if on then
            entry.button.BackgroundColor3 = mc.GREEN1
            entry.knob.BackgroundTransparency = 0
            entry.stateLabel.Text = "ON"
            entry.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            entry.stroke.Color = mc.GREEN_STROKE
            entry.stroke.Transparency = 0.22
        else
            entry.button.BackgroundColor3 = mc.OFF_BG
            entry.knob.BackgroundTransparency = 1
            entry.stateLabel.Text = "OFF"
            entry.stateLabel.TextColor3 = mc.OFF_TEXT
            entry.stroke.Color = mc.AQUA_STROKE
            entry.stroke.Transparency = 0.55
        end
        if entry.rowStroke then
            entry.rowStroke.Transparency = on and 0.38 or 0.62
        end
    end

    local function muiInputRow(parent, labelText, order, defaultVal, onCommit)
        local fr = Instance.new("Frame", parent)
        fr.Size = UDim2.new(1, 0, 0, 44)
        fr.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        fr.BackgroundTransparency = 0.61
        TGT.grad(fr, { { 0, Color3.fromRGB(55, 20, 73), 0.72 }, { 0.5, Color3.fromRGB(151, 80, 216), 0.61 }, { 1, Color3.fromRGB(36, 16, 50), 0.74 } }, 90)
        fr.BorderSizePixel = 0
        fr.LayoutOrder = order
        fr.ZIndex = 103
        mcCorner(fr, 10)
        mcStroke(fr, TGT.OUTLINE, 1, 0.62)
        local lb = Instance.new("TextLabel", fr)
        lb.BackgroundTransparency = 1
        lb.Position = UDim2.fromOffset(10, 0)
        lb.Size = UDim2.new(0, 120, 1, 0)
        lb.Font = Enum.Font.GothamBold
        lb.Text = labelText
        lb.TextColor3 = mc.TEXT
        lb.TextSize = 11
        lb.TextXAlignment = Enum.TextXAlignment.Left
        lb.ZIndex = 104
        local inp = Instance.new("TextBox", fr)
        inp.Size = UDim2.new(0, 60, 0, 24)
        inp.Position = UDim2.new(1, -74, 0.5, -12)
        inp.BackgroundColor3 = mc.BG
        inp.BorderSizePixel = 0
        inp.Text = tostring(defaultVal)
        inp.Font = Enum.Font.Gotham
        inp.TextSize = 11
        inp.TextColor3 = mc.TEXT
        inp.PlaceholderColor3 = mc.DIM
        inp.ZIndex = 104
        mcCorner(inp, 6)
        inp.FocusLost:Connect(function() onCommit(inp) end)
        return fr, inp
    end

    local function muiActionRow(parent, labelText, order, initialText, onPress)
        local fr = Instance.new("Frame", parent)
        fr.Size = UDim2.new(1, 0, 0, 44)
        fr.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        fr.BackgroundTransparency = 0.61
        TGT.grad(fr, { { 0, Color3.fromRGB(55, 20, 73), 0.72 }, { 0.5, Color3.fromRGB(151, 80, 216), 0.61 }, { 1, Color3.fromRGB(36, 16, 50), 0.74 } }, 90)
        fr.BorderSizePixel = 0
        fr.LayoutOrder = order
        fr.ZIndex = 103
        mcCorner(fr, 10)
        mcStroke(fr, TGT.OUTLINE, 1, 0.62)
        local lb = Instance.new("TextLabel", fr)
        lb.BackgroundTransparency = 1
        lb.Position = UDim2.fromOffset(10, 0)
        lb.Size = UDim2.new(0, 120, 1, 0)
        lb.Font = Enum.Font.GothamBold
        lb.Text = labelText
        lb.TextColor3 = mc.TEXT
        lb.TextSize = 11
        lb.TextXAlignment = Enum.TextXAlignment.Left
        lb.ZIndex = 104
        local btn = Instance.new("TextButton", fr)
        btn.Size = UDim2.new(0, 60, 0, 24)
        btn.Position = UDim2.new(1, -74, 0.5, -12)
        btn.BackgroundColor3 = mc.AQUA2
        btn.BorderSizePixel = 0
        btn.AutoButtonColor = false
        btn.Text = tostring(initialText)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 11
        btn.TextColor3 = mc.TEXT
        btn.ZIndex = 104
        mcCorner(btn, 6)
        mcStroke(btn, mc.AQUA_STROKE, 1, 0.55)
        btn.MouseButton1Click:Connect(function()
            local ok, res = pcall(onPress)
            if ok and res ~= nil then btn.Text = tostring(res) end
        end)
        return fr, btn
    end

    -- ========== PLAYER TAB ==========
    local antiRagRow = muiToggleRow(playerInner, "Anti Ragdoll", 100)
    local infJumpRow = muiToggleRow(playerInner, "Inf Jump", 110)

    muiPaintRow(antiRagRow, i.AntiRagdoll ~= false)
    muiPaintRow(infJumpRow, type(_G.StickyInfJump) == "table" and _G.StickyInfJump.IsEnabled and _G.StickyInfJump.IsEnabled() or false)

    antiRagRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AntiRagdoll ~= false)
        muiPaintRow(antiRagRow, on)
        if type(_G.setAntiRagdoll) == "function" then task.spawn(_G.setAntiRagdoll, on)
        else i.AntiRagdoll = on; U() end
    end)
    infJumpRow.button.MouseButton1Click:Connect(function()
        if type(_G.StickyInfJump) == "table" and _G.StickyInfJump.Toggle then
            _G.StickyInfJump.Toggle()
            muiPaintRow(infJumpRow, _G.StickyInfJump.IsEnabled())
        end
        if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
    end)

    muiInputRow(playerInner, "Carpet Spd Val", 200, _G.StickyCarpetSpeedValue or 140, function(inp)
        local val = tonumber(inp.Text)
        if val then
            _G.StickyCarpetSpeedValue = math.clamp(val, 20, 400)
            inp.Text = tostring(_G.StickyCarpetSpeedValue)
        else
            inp.Text = tostring(_G.StickyCarpetSpeedValue or 140)
        end
        if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
    end)
    muiInputRow(playerInner, "Steal Boost Spd", 210, tonumber(i.StealBoostValue) or 28, function(inp)
        local val = tonumber(inp.Text)
        if val and type(_G.setStealBoostValue) == "function" then
            _G.setStealBoostValue(val)
        elseif val then
            i.StealBoostValue = math.clamp(math.floor(val + 0.5), 15, 35)
            U()
        end
        inp.Text = tostring(tonumber(i.StealBoostValue) or 28)
    end)

    -- ========== ESP TAB ==========
    local xrayRow = muiToggleRow(espInner, "X-Ray", 100)
    local espRow = muiToggleRow(espInner, "Player ESP", 110)
    local podiumRow = muiToggleRow(espInner, "Podium ESP", 120)
    local turretRow = muiToggleRow(espInner, "Turret ESP", 130)
    local trapRow = muiToggleRow(espInner, "Trap ESP", 140)
    local brainrotRow = muiToggleRow(espInner, "Brainrot ESP", 150)
    local baseRow = muiToggleRow(espInner, "Next Base", 160)
    local floorRow = muiToggleRow(espInner, "Floor Platform", 170)
    local baseTimerRow = muiToggleRow(espInner, "Base Timer ESP", 180)

    muiPaintRow(xrayRow, i.XRay == true)
    muiPaintRow(espRow, i.PlayerESP == true)
    muiPaintRow(podiumRow, i.podiumESP == true)
    muiPaintRow(turretRow, i.TurretESP == true)
    muiPaintRow(trapRow, i.TrapESP == true)
    muiPaintRow(brainrotRow, i.BrainrotESP == true)
    muiPaintRow(baseRow, i.nextBaseEnabled == true)
    muiPaintRow(floorRow, i.FloorPlatform == true)
    muiPaintRow(baseTimerRow, i.BaseTimerESP == true)

    xrayRow.button.MouseButton1Click:Connect(function()
        local on = not (i.XRay == true)
        muiPaintRow(xrayRow, on)
        if type(_G.setXRay) == "function" then task.spawn(_G.setXRay, on)
        else i.XRay = on; U() end
    end)
    espRow.button.MouseButton1Click:Connect(function()
        local on = not (i.PlayerESP == true)
        muiPaintRow(espRow, on)
        if type(_G.setPlayerESP) == "function" then task.spawn(_G.setPlayerESP, on)
        else i.PlayerESP = on; U() end
    end)
    podiumRow.button.MouseButton1Click:Connect(function()
        local on = not (i.podiumESP == true)
        muiPaintRow(podiumRow, on)
        if type(_G.setPodiumESP) == "function" then task.spawn(_G.setPodiumESP, on)
        else i.podiumESP = on; U() end
    end)
    turretRow.button.MouseButton1Click:Connect(function()
        local on = not (i.TurretESP == true)
        muiPaintRow(turretRow, on)
        if type(_G.setTurretESP) == "function" then task.spawn(_G.setTurretESP, on)
        else i.TurretESP = on; U() end
    end)
    trapRow.button.MouseButton1Click:Connect(function()
        local on = not (i.TrapESP == true)
        muiPaintRow(trapRow, on)
        if type(_G.setTrapESP) == "function" then task.spawn(_G.setTrapESP, on)
        else i.TrapESP = on; U() end
    end)
    brainrotRow.button.MouseButton1Click:Connect(function()
        local on = not (i.BrainrotESP == true)
        muiPaintRow(brainrotRow, on)
        if type(_G.setBrainrotESP) == "function" then task.spawn(_G.setBrainrotESP, on)
        else i.BrainrotESP = on; U() end
    end)
    baseRow.button.MouseButton1Click:Connect(function()
        local on = not (i.nextBaseEnabled == true)
        muiPaintRow(baseRow, on)
        if type(_G.setNextBase) == "function" then task.spawn(_G.setNextBase, on)
        else i.nextBaseEnabled = on; U() end
    end)
    floorRow.button.MouseButton1Click:Connect(function()
        local on = not (i.FloorPlatform == true)
        muiPaintRow(floorRow, on)
        if type(_G.StickySetFloorPlatform) == "function" then task.spawn(_G.StickySetFloorPlatform, on)
        else i.FloorPlatform = on; U() end
    end)

    do
        local function parseMinGen(text)
            if type(text) ~= "string" then return nil end
            local str = text:gsub("%s", ""):lower()
            if str == "" then return 0 end
            local num, suffix = str:match("^([%d%.]+)([kmbt]?)$")
            num = tonumber(num)
            if not num or num < 0 then return nil end
            if suffix == "k" then return num * 1e3 end
            if suffix == "m" then return num * 1e6 end
            if suffix == "b" then return num * 1e9 end
            if suffix == "t" then return num * 1e12 end
            return num * 1e6
        end
        local function showMinGen()
            local raw = tonumber(i.BrainrotESPMinGen) or 10000000
            if raw <= 0 then return "0" end
            if raw >= 1e12 then return tostring(raw / 1e12) .. "t" end
            if raw >= 1e9 then return tostring(raw / 1e9) .. "b" end
            if raw >= 1e6 then return tostring(raw / 1e6) .. "m" end
            if raw >= 1e3 then return tostring(raw / 1e3) .. "k" end
            return tostring(math.floor(raw))
        end
        muiInputRow(espInner, "Brainrot Min Gen", 200, showMinGen(), function(inp)
            local parsed = parseMinGen(inp.Text)
            if parsed then
                i.BrainrotESPMinGen = math.clamp(parsed, 0, 1e15)
                U()
            end
            inp.Text = showMinGen()
        end)
    end

    baseTimerRow.button.MouseButton1Click:Connect(function()
        local on = not (i.BaseTimerESP == true)
        i.BaseTimerESP = on
        _G.PlotTimerESPSaved = on
        muiPaintRow(baseTimerRow, on)
        if _G.setPlotTimerESPEnabled then
            pcall(_G.setPlotTimerESPEnabled, on)
        end
        U()
    end)

    -- ========== MISC TAB ==========
    local antiBeeRow = muiToggleRow(miscInner, "Anti Bee/Disco", 100)
    local antiDieRow = muiToggleRow(miscInner, "Anti Die", 110)
    local antiLagRow = muiToggleRow(miscInner, "Anti Lag", 120)
    local kickRow = muiToggleRow(miscInner, "Auto Kick", 130)
    local autoTurretRow = muiToggleRow(miscInner, "Auto Turret", 140)
    local toolAimRow = muiToggleRow(miscInner, "Tool Aimbot", 160)

    muiPaintRow(antiBeeRow, i.AntiBeeDisco ~= false)
    muiPaintRow(antiDieRow, i.AntiDie ~= false)
    muiPaintRow(antiLagRow, i.AntiLag == true)
    muiPaintRow(kickRow, _G.StickyAutoKickOnSteal == true)
    muiPaintRow(autoTurretRow, i.AutoTurret == true)
    muiPaintRow(toolAimRow, i.ToolAimbot == true)

    antiBeeRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AntiBeeDisco ~= false)
        muiPaintRow(antiBeeRow, on)
        if type(_G.setAntiBeeDisco) == "function" then task.spawn(_G.setAntiBeeDisco, on)
        else i.AntiBeeDisco = on; U() end
    end)
    antiDieRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AntiDie ~= false)
        muiPaintRow(antiDieRow, on)
        if type(_G.setAntiDie) == "function" then task.spawn(_G.setAntiDie, on)
        else i.AntiDie = on; U() end
    end)
    antiLagRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AntiLag == true)
        muiPaintRow(antiLagRow, on)
        if type(_G.setAntiLag) == "function" then task.spawn(_G.setAntiLag, on)
        else i.AntiLag = on; U() end
    end)
    kickRow.button.MouseButton1Click:Connect(function()
        local on = not (_G.StickyAutoKickOnSteal == true)
        _G.StickyAutoKickOnSteal = on
        muiPaintRow(kickRow, on)
        if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
    end)
    autoTurretRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AutoTurret == true)
        muiPaintRow(autoTurretRow, on)
        if type(_G.setAutoTurret) == "function" then task.spawn(_G.setAutoTurret, on)
        else i.AutoTurret = on; U() end
    end)
    toolAimRow.button.MouseButton1Click:Connect(function()
        local on = not (i.ToolAimbot == true)
        muiPaintRow(toolAimRow, on)
        if type(_G.setToolAimbot) == "function" then task.spawn(_G.setToolAimbot, on)
        else i.ToolAimbot = on; U() end
    end)

    local antiTrapRow = muiToggleRow(miscInner, "Anti Trap", 170)
    local hvSoundRow = muiToggleRow(miscInner, "Sound Alert", 180)

    muiPaintRow(antiTrapRow, i.AntiTrapEnabled == true)
    muiPaintRow(hvSoundRow, i.HighValueSoundEnabled ~= false)

    antiTrapRow.button.MouseButton1Click:Connect(function()
        local on = not (i.AntiTrapEnabled == true)
        i.AntiTrapEnabled = on
        _G.AntiTrapEnabledSaved = on
        muiPaintRow(antiTrapRow, on)
        if _G.AntiTrap and _G.AntiTrap.Toggle then
            pcall(_G.AntiTrap.Toggle)
        end
        U()
    end)
    hvSoundRow.button.MouseButton1Click:Connect(function()
        local on = not (_G.HighValueSoundEnabled ~= false)
        _G.HighValueSoundEnabled = on
        i.HighValueSoundEnabled = on
        muiPaintRow(hvSoundRow, on)
        U()
    end)

    muiInputRow(miscInner, "Sound ID", 185,
        type(i.HighValueSoundId) == "string" and i.HighValueSoundId or "100173184074904",
        function(inp)
            local text = inp.Text:gsub("%D", "")
            if text == "" then text = "100173184074904" end
            inp.Text = text
            _G.HighValueSoundId = text
            i.HighValueSoundId = text
            U()
        end)

    do
        local previewSound = nil
        muiActionRow(miscInner, "Test Sound", 186, "\226\150\182", function()
            pcall(function()
                if previewSound and previewSound.Playing then
                    previewSound:Stop()
                    previewSound:Destroy()
                    previewSound = nil
                    return
                end
                local id = type(_G.HighValueSoundId) == "string" and _G.HighValueSoundId:gsub("%D", "") or "100173184074904"
                if id == "" then return end
                previewSound = Instance.new("Sound")
                previewSound.SoundId = "rbxassetid://" .. id
                previewSound.Volume = 1
                previewSound.Parent = game:GetService("SoundService")
                previewSound:Play()
                previewSound.Ended:Once(function()
                    if previewSound then previewSound:Destroy(); previewSound = nil end
                end)
            end)
            return "\226\150\182"
        end)
    end


    do
        local function showAlertMinGen()
            local raw = tonumber(i.HighValueMinGen) or 5000000
            if raw <= 0 then return "0" end
            if raw >= 1e12 then return tostring(raw / 1e12) .. "t" end
            if raw >= 1e9 then return tostring(raw / 1e9) .. "b" end
            if raw >= 1e6 then return tostring(raw / 1e6) .. "m" end
            if raw >= 1e3 then return tostring(raw / 1e3) .. "k" end
            return tostring(math.floor(raw))
        end
        muiInputRow(miscInner, "Alert Min Gen", 190, showAlertMinGen(), function(inp)
            local str = inp.Text:gsub("%s", ""):lower()
            local num, suffix = str:match("^([%d%.]+)([kmbt]?)$")
            num = tonumber(num)
            if num and num >= 0 then
                local mult = 1e6
                if suffix == "k" then mult = 1e3
                elseif suffix == "m" then mult = 1e6
                elseif suffix == "b" then mult = 1e9
                elseif suffix == "t" then mult = 1e12 end
                local val = math.clamp(num * mult, 0, 50e6)
                i.HighValueMinGen = val
                _G.HighValueMinGen = val
                U()
            end
            inp.Text = showAlertMinGen()
        end)
    end

    muiActionRow(miscInner, "FOV", 200,
        math.floor(tonumber(i.FOV) or 80), function()
        if type(_G.StickyCycleFOV) == "function" then
            return math.floor(_G.StickyCycleFOV())
        end
        return math.floor(tonumber(i.FOV) or 80)
    end)

    do
        local muiPriorityRow = Instance.new("Frame", miscInner)
        muiPriorityRow.Name = "PriorityListRow"
        muiPriorityRow.Size = UDim2.new(1, 0, 0, 44)
        muiPriorityRow.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        muiPriorityRow.BackgroundTransparency = 0.61
        TGT.grad(muiPriorityRow, { { 0, Color3.fromRGB(55, 20, 73), 0.72 }, { 0.5, Color3.fromRGB(151, 80, 216), 0.61 }, { 1, Color3.fromRGB(36, 16, 50), 0.74 } }, 90)
        muiPriorityRow.BorderSizePixel = 0
        muiPriorityRow.LayoutOrder = 300
        muiPriorityRow.ZIndex = 103
        mcCorner(muiPriorityRow, 10)
        local prStroke = mcStroke(muiPriorityRow, TGT.OUTLINE, 1, 0.62)
        local prLabel = Instance.new("TextLabel", muiPriorityRow)
        prLabel.BackgroundTransparency = 1
        prLabel.Position = UDim2.fromOffset(10, 0)
        prLabel.Size = UDim2.new(1, -90, 1, 0)
        prLabel.Font = Enum.Font.GothamBold
        prLabel.Text = "Priority List"
        prLabel.TextColor3 = mc.TEXT
        prLabel.TextSize = 13
        prLabel.TextXAlignment = Enum.TextXAlignment.Left
        prLabel.ZIndex = 104
        local prBtn = Instance.new("TextButton", muiPriorityRow)
        prBtn.Name = "PriorityListButton"
        prBtn.AutoButtonColor = false
        prBtn.Size = UDim2.fromOffset(72, 26)
        prBtn.Position = UDim2.new(1, -80, 0.5, -13)
        prBtn.BackgroundColor3 = mc.AQUA2
        prBtn.BorderSizePixel = 0
        prBtn.Text = "OPEN"
        prBtn.TextColor3 = mc.TEXT
        prBtn.TextSize = 11
        prBtn.Font = Enum.Font.GothamBold
        prBtn.ZIndex = 104
        mcCorner(prBtn, 6)
        mcStroke(prBtn, mc.AQUA_STROKE, 1, 0.55)
        muiPriorityRow.MouseEnter:Connect(function()
            mcTween(muiPriorityRow, 0.14, { BackgroundColor3 = Color3.fromRGB(170, 100, 232) })
            mcTween(prStroke, 0.14, { Transparency = 0.38 })
        end)
        muiPriorityRow.MouseLeave:Connect(function()
            mcTween(muiPriorityRow, 0.14, { BackgroundColor3 = Color3.fromRGB(151, 80, 216) })
            mcTween(prStroke, 0.14, { Transparency = 0.62 })
        end)
        local muiPrEditorOpen = false
        local muiPrEditorGui = nil
        local function createMuiPriorityEditor()
            if muiPrEditorGui then
                muiPrEditorGui:Destroy()
                muiPrEditorGui = nil
                muiPrEditorOpen = false
                prBtn.Text = "OPEN"
                return
            end
            local gui = Instance.new("ScreenGui")
            gui.Name = "PriorityEditor"
            gui.ResetOnSpawn = false
            gui.Parent = (gethui and gethui()) or game:GetService("CoreGui")
            local frame = Instance.new("Frame")
            frame.Size = UDim2.new(0, 340, 0, 500)
            frame.AnchorPoint = Vector2.new(0.5, 0.5)
            frame.Position = UDim2.new(0.5, 0, 0.5, 0)
            frame.BackgroundColor3 = Color3.fromRGB(20, 10, 10)
            frame.BorderSizePixel = 0
            frame.Parent = gui
            Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)
            local stroke = Instance.new("UIStroke", frame)
            stroke.Color = mc.AQUA_STROKE
            stroke.Thickness = 1.2
            stroke.Transparency = 0.3
            local title = Instance.new("TextLabel", frame)
            title.Size = UDim2.new(1, 0, 0, 50)
            title.BackgroundTransparency = 1
            title.Text = "PRIORITY LIST"
            title.Font = Enum.Font.GothamBlack
            title.TextSize = 22
            title.TextColor3 = mc.TEXT
            title.TextXAlignment = Enum.TextXAlignment.Center
            local closeBtn = Instance.new("TextButton", frame)
            closeBtn.Size = UDim2.new(0, 36, 0, 36)
            closeBtn.Position = UDim2.new(1, -44, 0, 7)
            closeBtn.BackgroundColor3 = mc.SURF2
            closeBtn.Text = "X"
            closeBtn.Font = Enum.Font.GothamBold
            closeBtn.TextSize = 16
            closeBtn.TextColor3 = mc.TEXT
            closeBtn.AutoButtonColor = false
            Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)
            closeBtn.MouseButton1Click:Connect(function()
                gui:Destroy()
                muiPrEditorGui = nil
                muiPrEditorOpen = false
                prBtn.Text = "OPEN"
            end)
            local scroll = Instance.new("ScrollingFrame", frame)
            scroll.Size = UDim2.new(1, -16, 1, -80)
            scroll.Position = UDim2.new(0, 8, 0, 60)
            scroll.BackgroundTransparency = 1
            scroll.BorderSizePixel = 0
            scroll.ScrollBarThickness = 3
            scroll.ScrollBarImageColor3 = mc.AQUA_STROKE
            scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
            local listLayout = Instance.new("UIListLayout", scroll)
            listLayout.Padding = UDim.new(0, 4)
            listLayout.SortOrder = Enum.SortOrder.LayoutOrder
            listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
                scroll.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10)
            end)
            local function refreshList()
                for _, child in ipairs(scroll:GetChildren()) do
                    if child:IsA("Frame") and child.Name == "Item" then child:Destroy() end
                end
                for idx, pet in ipairs(X) do
                    local row = Instance.new("Frame", scroll)
                    row.Name = "Item"
                    row.Size = UDim2.new(1, 0, 0, 40)
                    row.BackgroundColor3 = mc.SURF2
                    row.BackgroundTransparency = 0.05
                    row.BorderSizePixel = 0
                    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)
                    local label = Instance.new("TextLabel", row)
                    label.Size = UDim2.new(1, -90, 1, 0)
                    label.Position = UDim2.new(0, 10, 0, 0)
                    label.BackgroundTransparency = 1
                    label.Text = tostring(idx) .. ". " .. pet
                    label.Font = Enum.Font.GothamBold
                    label.TextSize = 13
                    label.TextColor3 = mc.TEXT
                    label.TextXAlignment = Enum.TextXAlignment.Left
                    local up = Instance.new("TextButton", row)
                    up.Size = UDim2.new(0, 30, 0, 30)
                    up.Position = UDim2.new(1, -70, 0.5, -15)
                    up.BackgroundColor3 = mc.AQUA2
                    up.Text = "▲"
                    up.Font = Enum.Font.GothamBold
                    up.TextSize = 14
                    up.TextColor3 = mc.TEXT
                    up.AutoButtonColor = false
                    Instance.new("UICorner", up).CornerRadius = UDim.new(0, 5)
                    up.MouseButton1Click:Connect(function()
                        if idx > 1 then
                            X[idx], X[idx - 1] = X[idx - 1], X[idx]
                            v(); U(); refreshList()
                            P.ListNeedsRedraw = true
                            if P.UpdateAutoStealUI then P.UpdateAutoStealUI() end
                        end
                    end)
                    local down = Instance.new("TextButton", row)
                    down.Size = UDim2.new(0, 30, 0, 30)
                    down.Position = UDim2.new(1, -36, 0.5, -15)
                    down.BackgroundColor3 = mc.AQUA2
                    down.Text = "▼"
                    down.Font = Enum.Font.GothamBold
                    down.TextSize = 14
                    down.TextColor3 = mc.TEXT
                    down.AutoButtonColor = false
                    Instance.new("UICorner", down).CornerRadius = UDim.new(0, 5)
                    down.MouseButton1Click:Connect(function()
                        if idx < #X then
                            X[idx], X[idx + 1] = X[idx + 1], X[idx]
                            v(); U(); refreshList()
                            P.ListNeedsRedraw = true
                            if P.UpdateAutoStealUI then P.UpdateAutoStealUI() end
                        end
                    end)
                end
            end
            refreshList()
            muiPrEditorGui = gui
            muiPrEditorOpen = true
            prBtn.Text = "CLOSE"
            D(frame, frame)
        end
        prBtn.MouseButton1Click:Connect(createMuiPriorityEditor)
    end

    -- ========== KEYBINDS TAB ==========
    do
        local BindUIS = game:GetService("UserInputService")
        local capturing = nil
        local BINDS = {
            { label = "Menu",          field = "MenuKey",         default = "LeftControl" },
            { label = "Drop Brainrot", field = "DropKey",         default = "R" },
            { label = "Instant Reset", field = "InstantResetKey", default = "X" },
            { label = "Carpet Speed",  field = "CarpetSpeedKey",  default = "Q" },
            { label = "Steal Boost",   field = "StealBoostKey",   default = "B" },
            { label = "Auto Kick",     field = "AutoKickKey",     default = "K" },
            { label = "Kick Server",   field = "KickKey",         default = "P" },
            { label = "Invisible Steal", field = "InvisibleStealKey", default = "V" },
        }
        local function currentKey(entry)
            local name = i[entry.field]
            if type(name) ~= "string" or name == "" then name = entry.default end
            return name
        end
        for idx, entry in ipairs(BINDS) do
            local fr = Instance.new("Frame", keybindsInner)
            fr.Size = UDim2.new(1, 0, 0, 44)
            fr.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
            fr.BackgroundTransparency = 0.61
            fr.BorderSizePixel = 0
            fr.LayoutOrder = idx * 10
            fr.ZIndex = 103
            mcCorner(fr, 10)
            TGT.grad(fr, {
                { 0, Color3.fromRGB(55, 20, 73), 0.72 },
                { 0.5, Color3.fromRGB(151, 80, 216), 0.61 },
                { 1, Color3.fromRGB(36, 16, 50), 0.74 },
            }, 90)
            mcStroke(fr, TGT.OUTLINE, 1, 0.62)
            local lb = Instance.new("TextLabel", fr)
            lb.BackgroundTransparency = 1
            lb.Position = UDim2.fromOffset(13, 0)
            lb.Size = UDim2.new(1, -100, 1, 0)
            lb.Font = Enum.Font.GothamBold
            lb.Text = entry.label
            lb.TextColor3 = mc.TEXT
            lb.TextSize = 13
            lb.TextXAlignment = Enum.TextXAlignment.Left
            lb.TextTruncate = Enum.TextTruncate.AtEnd
            lb.ZIndex = 104
            local btn = Instance.new("TextButton", fr)
            btn.AnchorPoint = Vector2.new(1, 0.5)
            btn.Position = UDim2.new(1, -11, 0.5, 0)
            btn.Size = UDim2.fromOffset(74, 26)
            btn.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
            btn.BackgroundTransparency = 0.25
            btn.BorderSizePixel = 0
            btn.AutoButtonColor = false
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 11
            btn.TextColor3 = mc.TEXT
            btn.Text = PrettyKeyName(currentKey(entry))
            btn.ZIndex = 104
            mcCorner(btn, 8)
            mcStroke(btn, TGT.OUTLINE, 1, 0.45)
            btn.MouseButton1Click:Connect(function()
                if capturing then capturing() end
                btn.Text = "press key"
                local finished, conn = false, nil
                local function finish(keyName)
                    if finished then return end
                    finished = true
                    capturing = nil
                    if conn then pcall(function() conn:Disconnect() end) end
                    if keyName then
                        i[entry.field] = keyName
                        if entry.field == "MenuKey" then
                            i.SavedKeybinds = i.SavedKeybinds or {}
                            i.SavedKeybinds.MenuKey = keyName
                        elseif entry.field == "CarpetSpeedKey" then
                            _G.StickyCarpetSpeedKeyName = keyName
                        elseif entry.field == "InvisibleStealKey" then
                            local okKey, kc = pcall(function() return Enum.KeyCode[keyName] end)
                            if okKey and kc then _G.StickyInvisStealKey = kc end
                        end
                        U()
                    end
                    btn.Text = PrettyKeyName(currentKey(entry))
                end
                conn = BindUIS.InputBegan:Connect(function(input, processed)
                    if processed then return end
                    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
                    if input.KeyCode == Enum.KeyCode.Escape then finish(nil)
                    else finish(input.KeyCode.Name) end
                end)
                capturing = function() finish(nil) end
                task.delay(5, function() finish(nil) end)
            end)
        end
    end

    -- ========== UI TAB ==========
    do
        local UIS = game:GetService("UserInputService")
        local SLIDER_MIN = 0.3
        local SLIDER_MAX = 2.0
        local KNOB_W = 18

        local function makePanelScaleRow(parent, order)
            local fr = Instance.new("Frame", parent)
            fr.Size = UDim2.new(1, 0, 0, 40)
            fr.BackgroundColor3 = Color3.fromRGB(42, 17, 69)
            fr.BackgroundTransparency = 0.45
            fr.BorderSizePixel = 0
            fr.LayoutOrder = order
            fr.ZIndex = 103
            fr.Visible = false
            mcCorner(fr, 8)
            mcStroke(fr, TGT.OUTLINE, 1, 0.72)

            local lb = Instance.new("TextLabel", fr)
            lb.BackgroundTransparency = 1
            lb.Position = UDim2.fromOffset(13, 0)
            lb.Size = UDim2.new(0, 42, 1, 0)
            lb.Font = Enum.Font.GothamBold
            lb.Text = "Scale"
            lb.TextColor3 = mc.DIM
            lb.TextSize = 11
            lb.TextXAlignment = Enum.TextXAlignment.Left
            lb.ZIndex = 104

            local display = Instance.new("TextLabel", fr)
            display.BackgroundTransparency = 1
            display.AnchorPoint = Vector2.new(1, 0.5)
            display.Position = UDim2.new(1, -10, 0.5, 0)
            display.Size = UDim2.new(0, 40, 0, 20)
            display.Font = Enum.Font.GothamBold
            display.TextColor3 = mc.TEXT
            display.TextSize = 11
            display.TextXAlignment = Enum.TextXAlignment.Right
            display.ZIndex = 104

            local track = Instance.new("Frame", fr)
            track.AnchorPoint = Vector2.new(0.5, 0.5)
            track.Position = UDim2.new(0.5, 10, 0.5, 0)
            track.Size = UDim2.new(1, -120, 0, 6)
            track.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
            track.BorderSizePixel = 0
            track.ZIndex = 104
            mcCorner(track, 3)

            local fill = Instance.new("Frame", track)
            fill.Size = UDim2.new(0, 0, 1, 0)
            fill.BackgroundColor3 = mc.AQUA2
            fill.BorderSizePixel = 0
            fill.ZIndex = 105
            mcCorner(fill, 3)

            local knob = Instance.new("TextButton", track)
            knob.Size = UDim2.fromOffset(KNOB_W, KNOB_W)
            knob.AnchorPoint = Vector2.new(0.5, 0.5)
            knob.Position = UDim2.new(0, 0, 0.5, 0)
            knob.BackgroundColor3 = mc.TEXT
            knob.BorderSizePixel = 0
            knob.AutoButtonColor = false
            knob.Text = ""
            knob.ZIndex = 106
            mcCorner(knob, KNOB_W / 2)
            mcStroke(knob, mc.AQUA_STROKE, 1.5, 0.3)

            local dragging = false
            local onChanged = nil

            local function setAlpha(alpha)
                alpha = math.clamp(alpha, 0, 1)
                local trackW = track.AbsoluteSize.X
                if trackW < 1 then trackW = 1 end
                local px = math.floor(alpha * trackW)
                knob.Position = UDim2.new(0, px, 0.5, 0)
                fill.Size = UDim2.new(0, px, 1, 0)
            end

            local function alphaFromScale(s)
                return (math.clamp(s, SLIDER_MIN, SLIDER_MAX) - SLIDER_MIN) / (SLIDER_MAX - SLIDER_MIN)
            end

            local function scaleFromAlpha(a)
                local raw = SLIDER_MIN + a * (SLIDER_MAX - SLIDER_MIN)
                return math.floor(raw * 10 + 0.5) / 10
            end

            local function beginDrag()
                dragging = true
            end

            local function updateDrag(inputPos)
                if not dragging then return end
                local absX = track.AbsolutePosition.X
                local absW = track.AbsoluteSize.X
                if absW < 1 then return end
                local rel = (inputPos.X - absX) / absW
                local alpha = math.clamp(rel, 0, 1)
                setAlpha(alpha)
                local s = scaleFromAlpha(alpha)
                display.Text = tostring(math.floor(s * 100)) .. "%"
                if onChanged then onChanged(s) end
            end

            local function endDrag()
                dragging = false
            end

            knob.MouseButton1Down:Connect(beginDrag)
            knob.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.Touch then beginDrag() end
            end)

            UIS.InputChanged:Connect(function(input)
                if not dragging then return end
                if input.UserInputType == Enum.UserInputType.MouseMovement
                    or input.UserInputType == Enum.UserInputType.Touch then
                    updateDrag(input.Position)
                end
            end)

            UIS.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1
                    or input.UserInputType == Enum.UserInputType.Touch then
                    endDrag()
                end
            end)

            track.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1
                    or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = true
                    updateDrag(input.Position)
                end
            end)

            return {
                frame = fr,
                display = display,
                setAlpha = setAlpha,
                alphaFromScale = alphaFromScale,
                setOnChanged = function(fn) onChanged = fn end,
            }
        end

        local function applyPanelScale(panel, scale)
            if not panel or not panel.Parent then return end
            local sc = panel:FindFirstChildOfClass("UIScale")
            if sc then sc.Scale = scale end
        end

        local function getPanelScale(configKey)
            return math.clamp(tonumber(i[configKey]) or 1.0, SLIDER_MIN, SLIDER_MAX)
        end

        local PANELS = {
            { label = "Steal Targets",   order = 100, sliderOrder = 101, configKey = "ScaleAutoSteal",        hideKey = "HideAutoSteal",     panelRef = "AutoStealFrame",       invertHide = true },
            { label = "Target Controls", order = 110, sliderOrder = 111, configKey = "ScaleTargetControls",    hideKey = "HideTargetControls", panelRef = "TargetControlsFrame", invertHide = true },
            { label = "Job ID",          order = 120, sliderOrder = 121, configKey = "ScaleJobID",             hideKey = "ShowJobID",          panelRef = "JobIDFrame",          invertHide = false },
            { label = "Invis Steal",     order = 130, sliderOrder = 131, configKey = "ScaleInvisSteal",        hideKey = "HideInvisSteal",     panelRef = "InvisPanelFrame",     invertHide = true },
        }

        for _, def in ipairs(PANELS) do
            local toggleRow = muiToggleRow(uiInner, def.label, def.order)
            local slider = makePanelScaleRow(uiInner, def.sliderOrder)

            local function isVisible()
                if def.invertHide then
                    return not i[def.hideKey]
                else
                    return i[def.hideKey] ~= false
                end
            end

            local function setVisible(vis)
                if def.invertHide then
                    i[def.hideKey] = not vis
                else
                    i[def.hideKey] = vis
                end
            end

            local curScale = getPanelScale(def.configKey)
            slider.display.Text = tostring(math.floor(curScale * 100)) .. "%"
            slider.setAlpha(slider.alphaFromScale(curScale))
            muiPaintRow(toggleRow, isVisible())
            slider.frame.Visible = isVisible()

            slider.setOnChanged(function(s)
                i[def.configKey] = s
                applyPanelScale(P[def.panelRef], s)
                U()
            end)

            _G.StickyOnBoot(function()
                local panel = P[def.panelRef]
                if panel and panel.Parent then
                    applyPanelScale(panel, getPanelScale(def.configKey))
                    if not isVisible() then
                        panel.Visible = false
                    end
                end
            end)

            toggleRow.button.MouseButton1Click:Connect(function()
                local newVis = not isVisible()
                setVisible(newVis)
                local panel = P[def.panelRef]
                if panel and panel.Parent then
                    panel.Visible = newVis
                end
                muiPaintRow(toggleRow, newVis)
                slider.frame.Visible = newVis
                U()
            end)
        end

        do
            local uiScaleRow = Instance.new("Frame", uiInner)
            uiScaleRow.Size = UDim2.new(1, 0, 0, 44)
            uiScaleRow.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
            uiScaleRow.BackgroundTransparency = 0.61
            TGT.grad(uiScaleRow, { { 0, Color3.fromRGB(55, 20, 73), 0.72 }, { 0.5, Color3.fromRGB(151, 80, 216), 0.61 }, { 1, Color3.fromRGB(36, 16, 50), 0.74 } }, 90)
            uiScaleRow.BorderSizePixel = 0
            uiScaleRow.LayoutOrder = 300
            uiScaleRow.ZIndex = 103
            mcCorner(uiScaleRow, 10)
            mcStroke(uiScaleRow, TGT.OUTLINE, 1, 0.62)
            local uiScaleLabel = Instance.new("TextLabel", uiScaleRow)
            uiScaleLabel.BackgroundTransparency = 1
            uiScaleLabel.Position = UDim2.fromOffset(10, 0)
            uiScaleLabel.Size = UDim2.new(0, 120, 1, 0)
            uiScaleLabel.Font = Enum.Font.GothamBold
            uiScaleLabel.Text = "UI Scale"
            uiScaleLabel.TextColor3 = mc.TEXT
            uiScaleLabel.TextSize = 11
            uiScaleLabel.TextXAlignment = Enum.TextXAlignment.Left
            uiScaleLabel.ZIndex = 104
            local uiScaleDisplay = Instance.new("TextLabel", uiScaleRow)
            uiScaleDisplay.BackgroundTransparency = 1
            uiScaleDisplay.Position = UDim2.new(1, -140, 0.5, -10)
            uiScaleDisplay.Size = UDim2.new(0, 30, 0, 20)
            uiScaleDisplay.Font = Enum.Font.GothamBold
            uiScaleDisplay.TextColor3 = mc.TEXT
            uiScaleDisplay.TextSize = 11
            uiScaleDisplay.ZIndex = 104
            uiScaleDisplay.Text = tostring(math.floor((tonumber(i.UIScale) or 1.0) * 100)) .. "%"
            local minBtn = Instance.new("TextButton", uiScaleRow)
            minBtn.Size = UDim2.new(0, 30, 0, 24)
            minBtn.Position = UDim2.new(1, -100, 0.5, -12)
            minBtn.BackgroundColor3 = mc.AQUA2
            minBtn.BorderSizePixel = 0
            minBtn.AutoButtonColor = false
            minBtn.Text = "-"
            minBtn.Font = Enum.Font.GothamBold
            minBtn.TextSize = 14
            minBtn.TextColor3 = mc.TEXT
            minBtn.ZIndex = 104
            mcCorner(minBtn, 6)
            mcStroke(minBtn, mc.AQUA_STROKE, 1, 0.55)
            local maxBtn = Instance.new("TextButton", uiScaleRow)
            maxBtn.Size = UDim2.new(0, 30, 0, 24)
            maxBtn.Position = UDim2.new(1, -65, 0.5, -12)
            maxBtn.BackgroundColor3 = mc.AQUA2
            maxBtn.BorderSizePixel = 0
            maxBtn.AutoButtonColor = false
            maxBtn.Text = "+"
            maxBtn.Font = Enum.Font.GothamBold
            maxBtn.TextSize = 14
            maxBtn.TextColor3 = mc.TEXT
            maxBtn.ZIndex = 104
            mcCorner(maxBtn, 6)
            mcStroke(maxBtn, mc.AQUA_STROKE, 1, 0.55)
            minBtn.MouseButton1Click:Connect(function()
                i.UIScale = math.clamp((tonumber(i.UIScale) or 1.0) - 0.1, 0.3, 2.0)
                uiScaleDisplay.Text = tostring(math.floor((tonumber(i.UIScale) or 1.0) * 100)) .. "%"
                P.RefreshUIScale()
                U()
            end)
            maxBtn.MouseButton1Click:Connect(function()
                i.UIScale = math.clamp((tonumber(i.UIScale) or 1.0) + 0.1, 0.3, 2.0)
                uiScaleDisplay.Text = tostring(math.floor((tonumber(i.UIScale) or 1.0) * 100)) .. "%"
                P.RefreshUIScale()
                U()
            end)
        end
    end

    -- Register ToggleUIPainters for new Main UI toggles
    for label, entry in pairs({
        ["X-Ray"] = xrayRow, ["XRay"] = xrayRow,
        ["Player ESP"] = espRow, ["PlayerESP"] = espRow,
        ["Next Base"] = baseRow, ["NextBase"] = baseRow,
        ["Auto Kick"] = kickRow, ["AutoKick"] = kickRow,
        ["Inf Jump"] = infJumpRow,
        ["Podium ESP"] = podiumRow, ["PodiumESP"] = podiumRow,
        ["Floor Platform"] = floorRow, ["FloorPlatform"] = floorRow,
        ["Turret ESP"] = turretRow, ["TurretESP"] = turretRow,
        ["Trap ESP"] = trapRow, ["TrapESP"] = trapRow,
        ["Brainrot ESP"] = brainrotRow, ["BrainrotESP"] = brainrotRow,
        ["Tool Aimbot"] = toolAimRow, ["ToolAimbot"] = toolAimRow,
        ["Anti Ragdoll"] = antiRagRow, ["Anti Bee/Disco"] = antiBeeRow,
        ["Anti Die"] = antiDieRow, ["Auto Turret"] = autoTurretRow,
        ["Anti Lag"] = antiLagRow,
        ["Base Timer ESP"] = baseTimerRow, ["BaseTimerESP"] = baseTimerRow,
        ["Anti Trap"] = antiTrapRow, ["AntiTrapEnabled"] = antiTrapRow,
        ["Sound Alert"] = hvSoundRow, ["HighValueSoundEnabled"] = hvSoundRow,
    }) do
        P.ToggleUIPainters[label] = function(on) muiPaintRow(entry, on) end
    end

    P.RefreshUIScale()
end

    local function sG(SG, NG)
        KG()
        local function PB(CB, rB, RB)
            local dB = RB or b.AQUA
            if rB then
                CB.button.BackgroundColor3 = b.GREEN1
                CB.knob.BackgroundTransparency = 0
                CB.stateLabel.Text = "ON"
                CB.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                CB.stroke.Color = b.GREEN_STROKE
                CB.stroke.Transparency = 0.22
            else
                CB.button.BackgroundColor3 = b.OFF_BG
                CB.knob.BackgroundTransparency = 1
                CB.stateLabel.Text = "OFF"
                CB.stateLabel.TextColor3 = b.OFF_TEXT
                CB.stroke.Color = b.AQUA_STROKE
                CB.stroke.Transparency = 0.55
            end
            if CB.rowStroke then
                CB.rowStroke.Transparency = rB and 0.38 or 0.62
            end
            if CB.label then
                CB.label.TextColor3 = rB and b.TEXT or b.TEXT
            end
        end
        PB(ZG, fG, W.Accent1)
        PB(kG, aG, W.Accent1)
        PB(uG, eG, W.Accent2)
        if LG then
            if xG then
                LG.stateLabel.Text = "ON"
                LG.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                LG.button.BackgroundColor3 = b.GREEN1
                LG.knob.BackgroundTransparency = 0
                LG.stroke.Color = b.GREEN_STROKE
                LG.stroke.Transparency = 0.22
            else
                LG.stateLabel.Text = "OFF"
                LG.stateLabel.TextColor3 = b.OFF_TEXT
                LG.button.BackgroundColor3 = b.OFF_BG
                LG.knob.BackgroundTransparency = 1
                LG.stroke.Color = b.AQUA_STROKE
                LG.stroke.Transparency = 0.55
            end
            if LG.rowStroke then
                LG.rowStroke.Transparency = xG and 0.38 or 0.62
            end
        end
        if v2Row then
            if v2On then
                v2Row.stateLabel.Text = "ON"
                v2Row.stateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                v2Row.button.BackgroundColor3 = b.GREEN1
                v2Row.knob.BackgroundTransparency = 0
                v2Row.stroke.Color = b.GREEN_STROKE
                v2Row.stroke.Transparency = 0.22
            else
                v2Row.stateLabel.Text = "OFF"
                v2Row.stateLabel.TextColor3 = b.OFF_TEXT
                v2Row.button.BackgroundColor3 = b.OFF_BG
                v2Row.knob.BackgroundTransparency = 1
                v2Row.stroke.Color = b.AQUA_STROKE
                v2Row.stroke.Transparency = 0.55
            end
            if v2Row.rowStroke then
                v2Row.rowStroke.Transparency = v2On and 0.38 or 0.62
            end
        end
        if OG and NG then
            local PB = false
            for CB, rB in ipairs(NG) do
                if rB.uid == OG then
                    iG = CB
                    PB = true
                    break
                end
            end
        end
        if P.ListNeedsRedraw then
            for PB, PB in ipairs(XG:GetChildren()) do
                if PB:IsA("TextButton") then
                    PB:Destroy()
                end
            end
            oG = {}
            if NG and #NG > 0 then
                for PB = 1, #NG do
                    local CB = NG[PB]
                    local rB = Instance.new("TextButton")
                    rB.Size = UDim2.new(1, 0, 0, 38)
                    rB.BackgroundColor3 = GG.SURF2
                    rB.BorderSizePixel = 0
                    rB.Text = ""
                    rB.AutoButtonColor = false
                    rB.Parent = XG
                    rB.Position = UDim2.new(0, 0, 0, 0)
                    rB.ClipsDescendants = true
                    rB.ZIndex = 1
                    Instance.new("UICorner", rB).CornerRadius = UDim.new(0, 8)
                    local RB = Instance.new("Frame", rB)
                    RB.Name = "SelectedOverlay"
                    RB.Size = UDim2.new(1, 0, 1, 0)
                    RB.Position = UDim2.new(0, 0, 0, 0)
                    RB.BackgroundColor3 = Color3.fromRGB(8, 10, 12)
                    RB.BackgroundTransparency = 0.82
                    RB.BorderSizePixel = 0
                    RB.Visible = false
                    RB.ZIndex = 2
                    Instance.new("UICorner", RB).CornerRadius = UDim.new(0, 8)
                    local dB = Instance.new("UIGradient", RB)
                    dB.Rotation = 90
                    dB.Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
                        ColorSequenceKeypoint.new(0.45, Color3.fromRGB(18, 22, 18)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0)),
                    })
                    dB.Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, 0.12),
                        NumberSequenceKeypoint.new(0.18, 0.3),
                        NumberSequenceKeypoint.new(0.55, 0.52),
                        NumberSequenceKeypoint.new(1, 0.18),
                    })
                    local dB = Instance.new("Frame", rB)
                    dB.Size = UDim2.fromOffset(24, 24)
                    dB.Position = UDim2.fromOffset(8, 7)
                    dB.BackgroundColor3 = GG.SURF
                    dB.ZIndex = 3
                    dB.BorderSizePixel = 0
                    Instance.new("UICorner", dB).CornerRadius = UDim.new(0, 4)
                    local nB = Instance.new("UIStroke", dB)
                    nB.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
                    nB.Color = GG.AQUA_STROKE
                    nB.Thickness = 1
                    nB.Transparency = 0.45
                    local fB = Instance.new("TextLabel", dB)
                    fB.Size = UDim2.new(1, 0, 1, 0)
                    fB.BackgroundTransparency = 1
                    fB.Text = "#" .. PB
                    fB.Font = Enum.Font.GothamBold
                    fB.TextSize = 11
                    fB.TextColor3 = GG.TEXT
                    fB.TextXAlignment = Enum.TextXAlignment.Center
                    fB.ZIndex = 4
                    local JB = CB and CB.petName or "Unknown"
                    local aB = CB and CB.mpsText or "$0/s"
                    local eB = Instance.new("TextLabel", rB)
                    eB.Size = UDim2.new(1, -140.0, 0, 16)
                    eB.Position = UDim2.fromOffset(40, 2)
                    eB.BackgroundTransparency = 1
                    eB.RichText = false
                    eB.Text = JB
                    eB.Font = Enum.Font.GothamBold
                    eB.TextSize = 12
                    eB.TextColor3 = GG.TEXT
                    eB.TextXAlignment = Enum.TextXAlignment.Left
                    eB.TextTruncate = Enum.TextTruncate.None
                    eB.ClipsDescendants = false
                    eB.ZIndex = 4
                    local JB = Instance.new("UITextSizeConstraint", eB)
                    JB.MinTextSize = 8
                    JB.MaxTextSize = 12
                    local JB = Instance.new("TextLabel", rB)
                    JB.Size = UDim2.new(0, 90, 0, 16)
                    JB.Position = UDim2.new(1, -98.0, 0, 2)
                    JB.BackgroundTransparency = 1
                    JB.RichText = false
                    JB.Text = aB
                    JB.Font = Enum.Font.GothamBold
                    JB.TextSize = 12
                    JB.TextColor3 = Color3.fromRGB(188, 120, 255)
                    JB.TextXAlignment = Enum.TextXAlignment.Right
                    JB.TextTruncate = Enum.TextTruncate.AtEnd
                    JB.ZIndex = 4
                    local aB = Instance.new("TextLabel", rB)
                    aB.Size = UDim2.new(1, -140.0, 0, 16)
                    aB.Position = UDim2.fromOffset(40, 20)
                    aB.BackgroundTransparency = 1
                    local xB, QB = jG(CB)
                    aB.RichText = QB
                    aB.Text = xB
                    aB.Font = Enum.Font.GothamBold
                    aB.TextSize = 12
                    aB.TextColor3 = IG(CB.mutation)
                    aB.TextXAlignment = Enum.TextXAlignment.Left
                    aB.TextTruncate = Enum.TextTruncate.None
                    aB.ClipsDescendants = false
                    aB.ZIndex = 4
                    local QB = Instance.new("UITextSizeConstraint", aB)
                    QB.MinTextSize = 8
                    QB.MaxTextSize = 12
                    aB.Visible = xB ~= ""
                    local mutIcons = {}
                    for iconSlot = 1, 4 do
                        local badge = Instance.new("ImageLabel", rB)
                        badge.Name = "Badge" .. iconSlot
                        badge.Size = UDim2.fromOffset(13, 13)
                        badge.Position = UDim2.fromOffset(40 + (iconSlot - 1) * 15, 21)
                        badge.BackgroundTransparency = 1
                        badge.ScaleType = Enum.ScaleType.Fit
                        badge.Image = ""
                        badge.Visible = false
                        badge.ZIndex = 5
                        mutIcons[iconSlot] = badge
                    end
                    oG[PB] = {
                        mutIcons = mutIcons,
                        button = rB,
                        selectedOverlay = RB,
                        rankBox = dB,
                        rankBoxStroke = nB,
                        rank = fB,
                        info = eB,
                        rate = JB,
                        mutation = aB,
                        petData = CB,
                    }
                    rB.MouseButton1Click:Connect(function()
                        if UG == CB.uid then
                            UG = nil
                            OG = nil
                            _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
                        else
                            iG = PB
                            OG = CB.uid
                            UG = CB.uid
                            nG = true
                            _G.NEAREST_INSTANT_MODE = false
                        end
                        P.ListNeedsRedraw = true
                        sG(nG, TG())
                    end)
                end
            end
            P.ListNeedsRedraw = false
            local PB = #oG
            local CB = 0
            if PB > 0 then
                CB = PB * 38 + (PB - 1) * 4
            end
            XG.Size = UDim2.new(1, -0.0, 0, CB)
            WG.CanvasSize = UDim2.new(0, 0, 0, CB + 8)
        end
        for PB, PB in ipairs(oG) do
            local oG = PB.petData and UG and PB.petData.uid == UG
            PB.button.ZIndex = 1
            PB.button.BackgroundTransparency = 0
            PB.button.BackgroundColor3 = oG and Color3.fromRGB(132, 54, 217) or GG.SURF2
            if PB.selectedOverlay then
                PB.selectedOverlay.Visible = oG
                PB.selectedOverlay.ZIndex = 2
                PB.selectedOverlay.BackgroundColor3 = Color3.fromRGB(8, 10, 12)
                PB.selectedOverlay.BackgroundTransparency = oG and 0.9 or 1
            end
            if PB.rankBox then
                PB.rankBox.BackgroundColor3 = oG and Color3.fromRGB(105, 42, 175) or GG.SURF
                PB.rankBox.ZIndex = 3
            end
            if PB.rankBoxStroke then
                PB.rankBoxStroke.Color = oG and Color3.fromRGB(105, 42, 175) or GG.AQUA_STROKE
                PB.rankBoxStroke.Thickness = 1
                PB.rankBoxStroke.Transparency = oG and 1 or 0.45
            end
            if PB.rank then
                PB.rank.ZIndex = 4
                PB.rank.TextColor3 = oG and Color3.fromRGB(240, 255, 240) or GG.TEXT
            end
            if PB.info then
                PB.info.ZIndex = 4
                PB.info.RichText = false
                if PB.petData then
                    PB.info.Text = pG(PB.petData)
                end
                PB.info.TextColor3 = oG and Color3.fromRGB(255, 255, 255) or GG.TEXT
            end
            if PB.mutation then
                PB.mutation.ZIndex = 4
                if PB.petData then
                    local pG, GG = jG(PB.petData)
                    PB.mutation.RichText = GG
                    PB.mutation.Text = pG
                    PB.mutation.Visible = pG ~= ""
                end
                if PB.mutIcons then
                    local ids = {}
                    if PB.petData and P.PetBadgeIcons then
                        local ok, res = pcall(P.PetBadgeIcons, PB.petData)
                        if ok and type(res) == "table" then ids = res end
                    end
                    local shown = 0
                    for iconSlot, badge in ipairs(PB.mutIcons) do
                        local id = ids[iconSlot]
                        if id then
                            if badge.Image ~= id then badge.Image = id end
                            badge.Visible = true
                            shown = iconSlot
                        else
                            badge.Visible = false
                        end
                    end
                    local textX = 40 + shown * 15
                    PB.mutation.Position = UDim2.fromOffset(textX, 20)
                    PB.mutation.Size = UDim2.new(1, -(100 + textX), 0, 16)
                end
                PB.mutation.TextColor3 = oG and Color3.fromRGB(255, 255, 255)
                    or IG(PB.petData and PB.petData.mutation)
            end
            if PB.rate then
                PB.rate.ZIndex = 4
                if PB.petData and PB.petData.mpsText then
                    PB.rate.Text = PB.petData.mpsText
                end
                PB.rate.TextColor3 = oG and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(188, 120, 255)
            end
        end
        local oG = NG and NG[iG]
        P.SelectedPetData = oG
        if SG then
            if not fG then
                if oG then
                    MG.Text = string.format("%s - %s", oG.petName or "Unknown", oG.mpsText or "")
                else
                    MG.Text = "Searching..."
                end
            end
        else
            MG.Text = "Disabled"
            if zG then
                zG:Cancel()
                zG = nil
            end
            wG.Size = UDim2.new(0, 0, 1, 0)
        end
        FG.Text = string.format("%d%%", math.clamp(math.floor(wG.Size.X.Scale * 100 + 0.5), 0, 100))
        XG.Size = UDim2.new(1, -0.0, 0, math.max(0, vG.AbsoluteContentSize.Y))
        WG.CanvasSize = UDim2.new(0, 0, 0, math.max(0, vG.AbsoluteContentSize.Y) + 8)
    end
    vG:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        XG.Size = UDim2.new(1, -0.0, 0, math.max(0, vG.AbsoluteContentSize.Y))
        WG.CanvasSize = UDim2.new(0, 0, 0, math.max(0, vG.AbsoluteContentSize.Y) + 8)
    end)
    P.UpdateAutoStealUI = function()
        sG(nG, TG())
    end
    task.spawn(function()
        while bG and bG.Parent do
            local oG = math.clamp(math.floor(wG.Size.X.Scale * 100 + 0.5), 0, 100)
            FG.Text = tostring(oG) .. "%"
            task.wait(0.05)
        end
    end)
    ZG.button.MouseButton1Click:Connect(function()
        fG = not fG
        if fG then
            aG = false
            eG = false
            UG = nil
        end
        i.StealNearest = fG
        i.StealHighest = aG
        i.StealPriority = eG
        KG()
        _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
        U()
        P.ListNeedsRedraw = false
        sG(nG, TG())
    end)
    kG.button.MouseButton1Click:Connect(function()
        aG = not aG
        if aG then
            fG = false
            eG = false
            UG = nil
        end
        i.StealNearest = fG
        i.StealHighest = aG
        i.StealPriority = eG
        KG()
        _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
        U()
        P.ListNeedsRedraw = false
        sG(nG, TG())
    end)
    uG.button.MouseButton1Click:Connect(function()
        eG = not eG
        if eG then
            fG = false
            aG = false
            UG = nil
        end
        i.StealNearest = fG
        i.StealHighest = aG
        i.StealPriority = eG
        KG()
        _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
        U()
        P.ListNeedsRedraw = false
        sG(nG, TG())
    end)
    LG.button.MouseButton1Click:Connect(function()
        xG = not xG
        if xG then
            QG = false
            VG = false
            if v2On then
                v2On = false
                i.InstantStealV2 = false
                if _G.setInstantStealV2 then pcall(_G.setInstantStealV2, false) end
            end
        else
            QG = false
            VG = false
        end
        i.InstantSteal = xG
        _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
        U()
        P.ListNeedsRedraw = false
        sG(nG, TG())
    end)
    task.spawn(function()
        while true do
            task.wait(1.5)
            if not xG then
                continue
            end
            xG = false
            QG = false
            VG = false
            i.InstantSteal = false
            _G.NEAREST_INSTANT_MODE = false
            task.wait(0.05)
            xG = true
            i.InstantSteal = true
            _G.NEAREST_INSTANT_MODE = (fG and true)
        end
    end)
    v2Row.button.MouseButton1Click:Connect(function()
        v2On = not v2On
        if v2On and xG then
            xG = false
            QG = false
            VG = false
            i.InstantSteal = false
        end
        i.InstantStealV2 = v2On
        _G.NEAREST_INSTANT_MODE = (fG and (xG or v2On))
        U()
        P.ListNeedsRedraw = false
        sG(nG, TG())
        if _G.setInstantStealV2 then
            _G.setInstantStealV2(v2On)
        end
    end)
    local function b(KG)
        if not KG then
            return nil
        end
        local oG = cG[KG.uid]
        if oG and oG.Parent then
            return oG
        end
        local oG = a.Plots:FindFirstChild(KG.plot)
        if not oG then
            return nil
        end
        local IG = oG:FindFirstChild("AnimalPodiums")
        if not IG then
            return nil
        end
        local pG = dG(oG.Name)
        local jG = pG and pG.AnimalList
        if not jG then
            local pG = IG:FindFirstChild(KG.slot)
            if pG then
                local lG = pG:FindFirstChild("Base")
                local pG = lG and lG:FindFirstChild("Spawn")
                if pG then
                    local lG = pG:FindFirstChild("PromptAttachment")
                    if lG then
                        for pG, pG in ipairs(lG:GetChildren()) do
                            if pG:IsA("ProximityPrompt") then
                                cG[KG.uid] = pG
                                return pG
                            end
                        end
                    end
                end
            end
            return nil
        end
        if not jG then
            return nil
        end
        local pG = KG.name and KG.name:lower() or ""
        local lG = KG.slot
        local WG = nil
        for XG, tG in pairs(jG) do
            if type(tG) == "table" and tostring(XG) == lG then
                local jG, lG = tG.Index, rG[tG.Index]
                if lG and (lG.DisplayName or jG):lower() == pG then
                    WG = IG:FindFirstChild(tostring(XG))
                    break
                end
            end
        end
        if not WG then
            WG = IG:FindFirstChild(KG.slot)
        end
        if WG then
            local IG = WG:FindFirstChild("Base")
            local jG = IG and IG:FindFirstChild("Spawn")
            if jG then
                local IG = jG:FindFirstChild("PromptAttachment")
                if IG then
                    for lG, lG in ipairs(IG:GetChildren()) do
                        if lG:IsA("ProximityPrompt") and lG.Enabled and lG.ActionText == "Steal" then
                            cG[KG.uid] = lG
                            return lG
                        end
                    end
                end
                local IG = jG.Position
                local jG, lG = IG.X, IG.Z
                local WG = nil
                local XG = math.huge
                for tG, tG in pairs(oG:GetDescendants()) do
                    if tG:IsA("ProximityPrompt") and tG.Enabled and tG.ActionText == "Steal" then
                        local oG = tG.Parent
                        local GG = nil
                        if oG and oG:IsA("BasePart") then
                            GG = oG.Position
                        elseif oG and oG:IsA("Attachment") and oG.Parent and oG.Parent:IsA("BasePart") then
                            GG = oG.Parent.Position
                        end
                        if GG then
                            local oG = IG.Y
                            if pG:find("la secret combinasion") then
                                oG = IG.Y - 5
                            end
                            local IG = math.sqrt((GG.X - jG) ^ 2 + (GG.Z - lG) ^ 2)
                            if IG < 5 and GG.Y > oG then
                                local IG = GG.Y - oG
                                if IG < XG then
                                    XG = IG
                                    WG = tG
                                end
                            end
                        end
                    end
                end
                if WG then
                    cG[KG.uid] = WG
                    return WG
                end
            end
        end
        return nil
    end
    local function KG(oG)
        if qG[oG] then
            return
        end
        local IG = { holdCallbacks = {}, triggerCallbacks = {}, holdEndCallbacks = {}, ready = true }
        local pG, jG = pcall(getconnections, oG.PromptButtonHoldBegan)
        if pG and type(jG) == "table" then
            for pG, pG in ipairs(jG) do
                if type(pG.Function) == "function" then
                    table.insert(IG.holdCallbacks, pG.Function)
                end
            end
        end
        local pG, jG = pcall(getconnections, oG.Triggered)
        if pG and type(jG) == "table" then
            for pG, pG in ipairs(jG) do
                if type(pG.Function) == "function" then
                    table.insert(IG.triggerCallbacks, pG.Function)
                end
            end
        end
        local pG, jG = pcall(getconnections, oG.PromptButtonHoldEnded)
        if pG and type(jG) == "table" then
            for pG, pG in ipairs(jG) do
                if type(pG.Function) == "function" then
                    table.insert(IG.holdEndCallbacks, pG.Function)
                end
            end
        end
        if #IG.holdCallbacks > 0 or #IG.triggerCallbacks > 0 or #IG.holdEndCallbacks > 0 then
            qG[oG] = IG
        end
    end
    local function oG(IG)
        for pG, pG in ipairs(IG) do
            task.spawn(pG)
        end
    end
    local function IG(pG)
        local jG = dG(pG)
        if jG then
            return S(jG.Owner)
        end
        return false
    end
    local function pG()
        local jG = C.Character and C.Character:FindFirstChild("HumanoidRootPart")
        if not jG then
            return nil, math.huge, nil
        end
        local lG = workspace:FindFirstChild("Plots")
        if not lG then
            return nil, math.huge, nil
        end
        local WG, XG, tG = nil, math.huge, nil
        for GG, GG in ipairs(lG:GetChildren()) do
            if IG(GG.Name) then
                continue
            end
            local IG = math.huge
            pcall(function()
                IG = (GG:GetPivot().Position - jG.Position).Magnitude
            end)
            if IG > 100 then
                continue
            end
            local IG = GG:FindFirstChild("AnimalPodiums")
            if not IG then
                continue
            end
            for lG, lG in ipairs(IG:GetChildren()) do
                local IG = lG:FindFirstChild("Base")
                local GG = IG and IG:FindFirstChild("Spawn")
                if not GG then
                    continue
                end
                local IG = (GG.Position - jG.Position).Magnitude
                if IG > 60 or IG >= XG then
                    continue
                end
                local jG = GG:FindFirstChild("PromptAttachment")
                if not jG then
                    continue
                end
                local GG = jG:FindFirstChildOfClass("ProximityPrompt")
                if GG and GG.Parent and GG.Enabled then
                    WG = GG
                    XG = IG
                    tG = lG.Name
                end
            end
        end
        return WG, XG, tG
    end
    local function IG(jG, lG)
        local WG = qG[jG]
        if not WG or not WG.ready then
            return false
        end
        WG.ready = false
        task.spawn(function()
            if BG ~= lG then
                if zG then
                    zG:Cancel()
                end
                wG.Size = UDim2.new(0, 0, 1, 0)
                BG = lG
            end
            if #WG.holdCallbacks > 0 then
                oG(WG.holdCallbacks)
            end
            wG.Size = UDim2.new(0, 0, 1, 0)
            wG.BackgroundTransparency = 0
            zG = f:Create(wG, TweenInfo.new(1.2, Enum.EasingStyle.Linear), { Size = UDim2.new(1, 0, 1, 0) })
            zG:Play()
            zG.Completed:Wait()
            if BG == lG and #WG.triggerCallbacks > 0 then
                oG(WG.triggerCallbacks)
            end
            WG.ready = true
        end)
        return true
    end
    local function oG(jG, lG)
        if not jG or not jG.Parent then
            return false
        end
        KG(jG)
        if not qG[jG] then
            return false
        end
        if BG ~= lG then
            if zG then
                zG:Cancel()
                zG = nil
            end
            wG.Size = UDim2.new(0, 0, 1, 0)
        end
        return IG(jG, lG)
    end
    local function qG()
        for BG, BG in pairs(cG) do
            if BG and BG.Parent then
                KG(BG)
            end
        end
    end
    task.spawn(function()
        while task.wait(2) do
            if nG then
                qG()
            end
        end
    end)
    local KG = {}
    local function qG(BG)
        if not BG then
            return ""
        end
        local IG = ""
        for jG, lG in pairs(BG) do
            if type(lG) == "table" then
                IG = IG .. tostring(jG) .. tostring(lG.Index) .. tostring(lG.Mutation)
            end
        end
        return IG
    end
    local function BG(IG)
        local jG = false
        pcall(function()
            local lG = dG(IG.Name)
            if not lG then
                return
            end
            local WG = lG.AnimalList
            local XG = lG.Owner
            if
                not XG
                or S(XG)
                or typeof(XG) == "Instance" and not r:FindFirstChild(XG.Name)
                or type(XG) == "string" and not r:FindFirstChild(XG)
            then
                KG[IG.Name] = nil
                for S = #yG, 1, -1.0 do
                    if yG[S].plot == IG.Name then
                        table.remove(yG, S)
                        jG = true
                    end
                end
                return
            end
            if not WG then
                KG[IG.Name] = nil
                for S = #yG, 1, -1.0 do
                    if yG[S].plot == IG.Name then
                        table.remove(yG, S)
                        jG = true
                    end
                end
                return
            end
            local S = typeof(XG) == "Instance" and XG.Name or tostring(XG)
            local lG = qG(WG, S)
            if KG[IG.Name] == lG then
                return
            end
            for qG = #yG, 1, -1.0 do
                if yG[qG].plot == IG.Name then
                    table.remove(yG, qG)
                end
            end
            for qG, XG in pairs(WG) do
                if type(XG) == "table" then
                    local WG, tG = XG.Index, rG[XG.Index]
                    if tG then
                        local rG = XG.Mutation or "None"
                        if rG == "Yin Yang" then
                            rG = "YinYang"
                        end
                        local GG = XG.Traits and #XG.Traits > 0 and table.concat(XG.Traits, ", ") or "None"
                        local vG = CG(WG, XG.Mutation, XG.Traits)
                        local CG = "$" .. JG(vG) .. "/s"
                        table.insert(
                            yG,
                            {
                                name = tG.DisplayName or WG,
                                genText = CG,
                                genValue = vG,
                                mutation = rG,
                                traits = GG,
                                owner = S,
                                plot = IG.Name,
                                slot = tostring(qG),
                                uid = IG.Name .. "_" .. tostring(qG),
                            }
                        )
                    end
                end
            end
            KG[IG.Name] = lG
            jG = true
        end)
        if jG then
            table.sort(yG, function(S, CG)
                return S.genValue > CG.genValue
            end)
            P.AllAnimalsCache = yG
            P.ListNeedsRedraw = true
            if P.UpdateAutoStealUI then
                P.UpdateAutoStealUI()
            end
        end
    end
    local function S(CG)
        local rG, JG = nil, 0
        while not rG and JG < 40 do
            rG = dG(CG.Name)
            if not rG then
                JG = JG + 1
                task.wait(0.07)
            end
        end
        if not rG then
            return
        end
        BG(CG)
        local function rG(JG)
            if not JG then
                return
            end
            JG.ChildAdded:Connect(function()
                task.wait(0.15)
                BG(CG)
            end)
            JG.ChildRemoved:Connect(function()
                for JG = #yG, 1, -1.0 do
                    if yG[JG].plot == CG.Name then
                        table.remove(yG, JG)
                    end
                end
                KG[CG.Name] = nil
                _getPetsCache = nil
                for JG in pairs(cG) do
                    if JG:sub(1, #CG.Name) == CG.Name then
                        cG[JG] = nil
                    end
                end
                P.ListNeedsRedraw = true
                if P.UpdateAutoStealUI then
                    P.UpdateAutoStealUI()
                end
                task.wait(0.15)
                BG(CG)
            end)
        end
        local JG = CG:FindFirstChild("AnimalPodiums")
        rG(JG)
        CG.ChildAdded:Connect(function(JG)
            if JG.Name == "AnimalPodiums" then
                rG(JG)
                BG(CG)
            end
        end)
        CG.ChildRemoved:Connect(function(rG)
            if rG.Name == "AnimalPodiums" then
                for rG = #yG, 1, -1.0 do
                    if yG[rG].plot == CG.Name then
                        table.remove(yG, rG)
                    end
                end
                KG[CG.Name] = nil
                _getPetsCache = nil
                P.ListNeedsRedraw = true
                if P.UpdateAutoStealUI then
                    P.UpdateAutoStealUI()
                end
            end
        end)
        task.spawn(function()
            while CG.Parent do
                task.wait(10)
                BG(CG)
            end
        end)
    end
    local CG = a:WaitForChild("Plots", 8)
    if CG then
        for rG, rG in ipairs(CG:GetChildren()) do
            S(rG)
        end
        CG.ChildAdded:Connect(function(rG)
            task.wait(0.5)
            S(rG)
        end)
        CG.ChildRemoved:Connect(function(S)
            KG[S.Name] = nil
            _getPetsCache = nil
            for CG = #yG, 1, -1.0 do
                if yG[CG].plot == S.Name then
                    table.remove(yG, CG)
                end
            end
            for CG in pairs(cG) do
                if CG:sub(1, #S.Name) == S.Name then
                    cG[CG] = nil
                end
            end
            P.ListNeedsRedraw = true
            if P.UpdateAutoStealUI then
                P.UpdateAutoStealUI()
            end
        end)
    end
    local function S(CG)
        if #CG == 0 then
            return
        end
        if UG then
            for rG, dG in ipairs(CG) do
                if dG.uid == UG then
                    if iG ~= rG then
                        iG = rG
                        OG = dG.uid
                    end
                    MG.Text = string.format("%s - %s", dG.petName or "Unknown", dG.mpsText or "")
                    return
                end
            end
            UG = nil
        end
        if eG then
            for rG, rG in ipairs(X) do
                local dG = rG:lower()
                for rG, JG in ipairs(CG) do
                    if JG.petName and JG.petName:lower() == dG then
                        if iG ~= rG then
                            iG = rG
                            OG = JG.uid
                        end
                        return
                    end
                end
            end
            if iG ~= 1 then
                iG = 1
                OG = CG[1] and CG[1].uid
            end
        elseif fG then
            local rG = C.Character
            local dG = rG and rG:FindFirstChild("HumanoidRootPart")
            if dG then
                local rG, JG = 1, math.huge
                for eG, KG in ipairs(CG) do
                    local qG = KG.animalData and M(KG.animalData)
                    if qG and qG:IsA("BasePart") then
                        local KG = (dG.Position - qG.Position).Magnitude
                        if KG < JG then
                            JG = KG
                            rG = eG
                        end
                    end
                end
                if CG[rG] then
                    local dG = CG[rG]
                    MG.Text = string.format("%s - %s", dG.petName or "Unknown", dG.mpsText or "")
                end
                if not xG and iG ~= rG then
                    iG = rG
                    OG = CG[rG] and CG[rG].uid
                end
            end
        elseif aG then
            if iG ~= 1 then
                iG = 1
                OG = CG[1] and CG[1].uid
            end
        end
    end
    R.Heartbeat:Connect(function()
        if not nG then
            return
        end
        S(TG())
    end)
    task.spawn(function()
        while true do
            task.wait(0.5)
            if nG then
                local S = TG()
                if #S > 0 then
                    P.ListNeedsRedraw = false
                    sG(nG, S)
                end
            end
        end
    end)
    R.Heartbeat:Connect(function()
        if not nG then
            return
        end
        if xG or v2On then
            if zG then
                zG:Cancel()
                zG = nil
            end
            wG.Size = UDim2.new(1, 0, 1, 0)
            wG.BackgroundTransparency = 0
            if not VG then
                VG = true
                task.spawn(function()
                    if not game:IsLoaded() then
                        game.Loaded:Wait()
                    end
                    task.wait(0.5)
                    QG = true
                end)
            end
            if QG then
                local targetPrompt = nil
                if fG and not UG then
                    local S, CG, rG = pG()
                    if S and CG <= 60 then
                        targetPrompt = S
                    end
                else
                    local S = TG()
                    if #S > 0 then
                        if iG > #S then
                            iG = #S
                        end
                        if iG < 1 then
                            iG = 1
                        end
                        local CG = S[iG]
                        if CG and not YG(CG.animalData) then
                            local S = cG[CG.uid]
                            if not S or not S.Parent then
                                S = b(CG.animalData)
                            end
                            if S then
                                targetPrompt = S
                            end
                        end
                    end
                end
                _G.StickyV2TargetPrompt = targetPrompt
            end
            return
        end
        local S = TG()
        if #S == 0 then
            return
        end
        if iG > #S then
            iG = #S
        end
        if iG < 1 then
            iG = 1
        end
        local CG = S[iG]
        if not CG or YG(CG.animalData) then
            return
        end
        local S = cG[CG.uid]
        if not S or not S.Parent then
            S = b(CG.animalData)
        end
        if S then
            oG(S, CG.uid)
        end
    end)
    task.spawn(function()
        while task.wait(0.5) do
            sG(nG, TG())
        end
    end)
    task.spawn(function()
        task.wait(1)
        P.ListNeedsRedraw = true
        sG(nG, TG())
    end)
    task.spawn(function()
        while true do
            P.AllAnimalsCache = yG
            task.wait(0.5)
        end
    end)

    local jobIdGui = x:FindFirstChild("JobIDPanel")
    if jobIdGui then jobIdGui:Destroy() end
    jobIdGui = Instance.new("ScreenGui", x)
    jobIdGui.Name = "JobIDPanel"
    jobIdGui.ResetOnSpawn = false
    jobIdGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local JOB_W, JOB_H = 360, 132
    local jPos = i.Positions.JobID
    local jPanel = Instance.new("Frame", jobIdGui)
    jPanel.Name = "JobIDContainer"
    jPanel.Size = UDim2.fromOffset(JOB_W, JOB_H)
    jPanel.Position = UDim2.new(
        jPos and jPos.X or 0.5,
        jPos and jPos.OffsetX or -180,
        jPos and jPos.Y or 0,
        jPos and jPos.OffsetY or 300
    )
    jPanel.BorderSizePixel = 0
    jPanel.ZIndex = 100
    jPanel.ClipsDescendants = false
    Instance.new("UICorner", jPanel).CornerRadius = UDim.new(0, 14)
    TGT.dressPanel(jPanel, 14)
    P.JobIDFrame = jPanel
    P.HoldPanel(jPanel)
    RegisterUIScale(jPanel)

    local jHeader = Instance.new("Frame", jPanel)
    jHeader.Name = "Topbar"
    jHeader.Size = UDim2.new(1, 0, 0, 38)
    jHeader.BackgroundTransparency = 1
    jHeader.Active = true
    jHeader.ZIndex = 101
    D(jHeader, jPanel, "JobID")
    TGT.macDots(jHeader, 10, 15, 17, 103, jPanel)

    local jTitle = Instance.new("TextLabel", jHeader)
    jTitle.Size = UDim2.new(1, -122, 1, 0)
    jTitle.Position = UDim2.new(0, 72, 0, 0)
    jTitle.BackgroundTransparency = 1
    jTitle.Text = "Current Server Job ID"
    jTitle.Font = Enum.Font.GothamBold
    jTitle.TextSize = 14
    jTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    jTitle.TextXAlignment = Enum.TextXAlignment.Left
    jTitle.ZIndex = 102

    local jMinBtn = TGT.minimizeBtn(jHeader, 104)
    jMinBtn.Size = UDim2.fromOffset(30, 26)
    jMinBtn.TextSize = 18

    local jobIdRealText = tostring(game.JobId or "")
    if jobIdRealText == "" then jobIdRealText = "LOCAL SESSION - NO JOB ID" end

    local jIdBox = Instance.new("TextBox", jPanel)
    jIdBox.Name = "JobIDDisplay"
    jIdBox.Position = UDim2.fromOffset(14, 43)
    jIdBox.Size = UDim2.fromOffset(JOB_W - 28, 34)
    jIdBox.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
    jIdBox.BackgroundTransparency = 0.25
    jIdBox.BorderSizePixel = 0
    jIdBox.ClearTextOnFocus = false
    jIdBox.TextEditable = false
    jIdBox.Text = jobIdRealText
    jIdBox.Font = Enum.Font.Code
    jIdBox.TextSize = 13
    jIdBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    jIdBox.TextXAlignment = Enum.TextXAlignment.Center
    jIdBox.TextTruncate = Enum.TextTruncate.AtEnd
    jIdBox.ZIndex = 102
    Instance.new("UICorner", jIdBox).CornerRadius = UDim.new(0, 10)
    local jIdBoxStroke = Instance.new("UIStroke", jIdBox)
    jIdBoxStroke.Color = TGT.OUTLINE
    jIdBoxStroke.Thickness = 1
    jIdBoxStroke.Transparency = 0.5
    jIdBoxStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local jIdLabel = jIdBox

    local function jobActionBtn(name, label, xPos, width)
        local btn = Instance.new("TextButton", jPanel)
        btn.Name = name
        btn.Position = UDim2.fromOffset(xPos, 86)
        btn.Size = UDim2.fromOffset(width, 32)
        btn.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        btn.BackgroundTransparency = 0.28
        btn.BorderSizePixel = 0
        btn.AutoButtonColor = false
        btn.Font = Enum.Font.GothamBold
        btn.Text = label
        btn.TextSize = 13
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.ZIndex = 102
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 10)
        TGT.grad(btn, {
            { 0, Color3.fromRGB(55, 20, 73), 0.72 },
            { 0.5, Color3.fromRGB(151, 80, 216), 0.61 },
            { 1, Color3.fromRGB(36, 16, 50), 0.74 },
        }, 90)
        local st = Instance.new("UIStroke", btn)
        st.Color = TGT.OUTLINE
        st.Thickness = 1
        st.Transparency = 0.45
        st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        return btn
    end

    local halfW = math.floor((JOB_W - 28 - 10) / 2)
    local jCopyBtn = jobActionBtn("CopyBtn", "COPY JOB ID", 14, halfW)
    local jHideBtn = jobActionBtn("HideBtn", "HIDE", 14 + halfW + 10, JOB_W - 28 - halfW - 10)

    local jobIdHidden = false
    local jobIdMinimized = false

    jMinBtn.MouseButton1Click:Connect(function()
        jobIdMinimized = not jobIdMinimized
        jIdBox.Visible = not jobIdMinimized
        jCopyBtn.Visible = not jobIdMinimized
        jHideBtn.Visible = not jobIdMinimized
        jPanel.Size = UDim2.fromOffset(JOB_W, jobIdMinimized and 38 or JOB_H)
        jMinBtn.Text = jobIdMinimized and "+" or "-"
    end)

    local function setJobIdHidden(hidden)
        jobIdHidden = hidden
        jIdLabel.Text = hidden and "job id is hidden baby sticky" or jobIdRealText
        jHideBtn.Text = hidden and "SHOW" or "HIDE"
        i.ShowJobID = not hidden
        U()
    end

    jCopyBtn.MouseButton1Click:Connect(function()
        local writer = (type(setclipboard) == "function" and setclipboard)
            or (type(toclipboard) == "function" and toclipboard)
            or (type(writeclipboard) == "function" and writeclipboard)
        local copied = writer and pcall(writer, jobIdRealText)
        jCopyBtn.Text = copied and "COPIED" or "COPY UNAVAILABLE"
        task.delay(1.5, function()
            if jCopyBtn.Parent then jCopyBtn.Text = "COPY JOB ID" end
        end)
    end)

    jHideBtn.MouseButton1Click:Connect(function()
        setJobIdHidden(not jobIdHidden)
    end)

    if i.ShowJobID == false then
        setJobIdHidden(true)
    end

    local invGui = x:FindFirstChild("InvisStealPanel")
    if invGui then invGui:Destroy() end
    invGui = Instance.new("ScreenGui", x)
    invGui.Name = "InvisStealPanel"
    invGui.ResetOnSpawn = false
    invGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local INV_W = 300
    local INV_H = 44 + (40 * 4) + (52 * 3) + (6 * 6) + 10
    local invPos = i.Positions.InvisSteal
    local invPanel = Instance.new("Frame", invGui)
    invPanel.Name = "InvisStealContainer"
    invPanel.Size = UDim2.fromOffset(INV_W, INV_H)
    invPanel.Position = UDim2.new(
        invPos and invPos.X or 0, invPos and invPos.OffsetX or 14,
        invPos and invPos.Y or 0, invPos and invPos.OffsetY or 470
    )
    invPanel.BorderSizePixel = 0
    invPanel.ZIndex = 100
    invPanel.ClipsDescendants = false
    Instance.new("UICorner", invPanel).CornerRadius = UDim.new(0, 14)
    TGT.dressPanel(invPanel, 14)
    P.InvisPanelFrame = invPanel
    P.HoldPanel(invPanel)
    RegisterUIScale(invPanel)

    local invHeader = Instance.new("Frame", invPanel)
    invHeader.Name = "Topbar"
    invHeader.Size = UDim2.new(1, 0, 0, 38)
    invHeader.BackgroundTransparency = 1
    invHeader.Active = true
    invHeader.ZIndex = 101
    D(invHeader, invPanel, "InvisSteal")
    TGT.macDots(invHeader, 10, 15, 17, 103, invPanel)

    local invTitle = Instance.new("TextLabel", invHeader)
    invTitle.Size = UDim2.new(1, -122, 1, 0)
    invTitle.Position = UDim2.new(0, 72, 0, 0)
    invTitle.BackgroundTransparency = 1
    invTitle.Text = "Invis Steal"
    invTitle.Font = Enum.Font.GothamBold
    invTitle.TextSize = 14
    invTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    invTitle.TextXAlignment = Enum.TextXAlignment.Left
    invTitle.ZIndex = 102

    local invMinBtn = TGT.minimizeBtn(invHeader, 104)
    invMinBtn.Size = UDim2.fromOffset(30, 26)
    invMinBtn.TextSize = 18

    local invBody = Instance.new("Frame", invPanel)
    invBody.Name = "Body"
    invBody.Position = UDim2.fromOffset(10, 44)
    invBody.Size = UDim2.new(1, -20, 1, -54)
    invBody.BackgroundTransparency = 1
    invBody.ZIndex = 101
    local invLayout = Instance.new("UIListLayout", invBody)
    invLayout.Padding = UDim.new(0, 6)
    invLayout.SortOrder = Enum.SortOrder.LayoutOrder

    local function invCard(order, height)
        local fr = Instance.new("Frame", invBody)
        fr.LayoutOrder = order
        fr.Size = UDim2.new(1, 0, 0, height or 40)
        fr.BackgroundColor3 = Color3.fromRGB(151, 80, 216)
        fr.BackgroundTransparency = 0.61
        fr.BorderSizePixel = 0
        fr.ZIndex = 102
        Instance.new("UICorner", fr).CornerRadius = UDim.new(0, 10)
        TGT.grad(fr, {
            { 0, Color3.fromRGB(55, 20, 73), 0.72 },
            { 0.5, Color3.fromRGB(151, 80, 216), 0.61 },
            { 1, Color3.fromRGB(36, 16, 50), 0.74 },
        }, 90)
        local st = Instance.new("UIStroke", fr)
        st.Color = TGT.OUTLINE
        st.Thickness = 1
        st.Transparency = 0.62
        st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        return fr
    end

    local function invToggle(order, label, getFn, setFn)
        local fr = invCard(order, 40)
        local lb = Instance.new("TextLabel", fr)
        lb.BackgroundTransparency = 1
        lb.Position = UDim2.fromOffset(12, 0)
        lb.Size = UDim2.new(1, -70, 1, 0)
        lb.Font = Enum.Font.GothamBold
        lb.Text = label
        lb.TextColor3 = Color3.fromRGB(255, 255, 255)
        lb.TextSize = 12
        lb.TextXAlignment = Enum.TextXAlignment.Left
        lb.TextTruncate = Enum.TextTruncate.AtEnd
        lb.ZIndex = 103

        local track = Instance.new("TextButton", fr)
        track.AnchorPoint = Vector2.new(1, 0.5)
        track.Position = UDim2.new(1, -10, 0.5, 0)
        track.Size = UDim2.fromOffset(40, 21)
        track.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
        track.BackgroundTransparency = 0.25
        track.BorderSizePixel = 0
        track.AutoButtonColor = false
        track.Text = ""
        track.ZIndex = 103
        Instance.new("UICorner", track).CornerRadius = UDim.new(0, 11)
        local ts = Instance.new("UIStroke", track)
        ts.Color = TGT.OUTLINE
        ts.Thickness = 1
        ts.Transparency = 0.45
        ts.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

        local fill = Instance.new("Frame", track)
        fill.Size = UDim2.new(1, 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(132, 54, 217)
        fill.BackgroundTransparency = 1
        fill.BorderSizePixel = 0
        fill.ZIndex = 103
        Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 11)
        local fg = TGT.grad(fill, {
            { 0, Color3.fromRGB(132, 54, 217) },
            { 0.5, Color3.fromRGB(188, 120, 255) },
            { 1, Color3.fromRGB(84, 32, 138) },
        }, 45)
        fg.Enabled = false

        local dot = Instance.new("Frame", track)
        dot.AnchorPoint = Vector2.new(0, 0.5)
        dot.Position = UDim2.new(0, 3, 0.5, 0)
        dot.Size = UDim2.fromOffset(16, 16)
        dot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        dot.BorderSizePixel = 0
        dot.ZIndex = 105
        Instance.new("UICorner", dot).CornerRadius = UDim.new(0, 8)

        local function paint(on)
            fill.BackgroundTransparency = on and 0.1 or 1
            fg.Enabled = on and true or false
            dot.Position = on and UDim2.new(1, -19, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
            ts.Transparency = on and 0.2 or 0.45
            track.BackgroundColor3 = on and Color3.fromRGB(132, 54, 217)
                or Color3.fromRGB(36, 16, 50)
        end
        paint(getFn() and true or false)
        track.MouseButton1Click:Connect(function()
            local nv = not (getFn() and true or false)
            setFn(nv)
            paint(getFn() and true or false)
        end)
        return paint
    end

    local function invSlider(order, label, minV, maxV, step, decimals, getFn, setFn)
        local fr = invCard(order, 52)
        local lb = Instance.new("TextLabel", fr)
        lb.BackgroundTransparency = 1
        lb.Position = UDim2.fromOffset(12, 4)
        lb.Size = UDim2.new(1, -150, 0, 18)
        lb.Font = Enum.Font.GothamBold
        lb.Text = label
        lb.TextColor3 = Color3.fromRGB(224, 193, 255)
        lb.TextSize = 12
        lb.TextXAlignment = Enum.TextXAlignment.Left
        lb.ZIndex = 103

        local val = Instance.new("TextLabel", fr)
        val.BackgroundTransparency = 1
        val.Position = UDim2.new(1, -132, 0, 4)
        val.Size = UDim2.fromOffset(52, 18)
        val.Font = Enum.Font.GothamBlack
        val.TextColor3 = Color3.fromRGB(255, 255, 255)
        val.TextSize = 13
        val.ZIndex = 103

        local bar = Instance.new("Frame", fr)
        bar.Position = UDim2.fromOffset(12, 32)
        bar.Size = UDim2.new(1, -24, 0, 8)
        bar.BackgroundColor3 = Color3.fromRGB(36, 16, 50)
        bar.BackgroundTransparency = 0.2
        bar.BorderSizePixel = 0
        bar.ZIndex = 103
        Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
        local barFill = Instance.new("Frame", bar)
        barFill.Size = UDim2.new(0, 0, 1, 0)
        barFill.BackgroundColor3 = Color3.fromRGB(188, 120, 255)
        barFill.BorderSizePixel = 0
        barFill.ZIndex = 104
        Instance.new("UICorner", barFill).CornerRadius = UDim.new(1, 0)

        local function fmt(v)
            if decimals then return string.format("%.2f", v) end
            return tostring(math.floor(v + 0.5))
        end
        local function refresh()
            local v = math.clamp(tonumber(getFn()) or minV, minV, maxV)
            val.Text = fmt(v)
            barFill.Size = UDim2.new((v - minV) / (maxV - minV), 0, 1, 0)
        end

        local function stepper(xOff, text, delta)
            local btn = Instance.new("TextButton", fr)
            btn.Position = UDim2.new(1, xOff, 0, 4)
            btn.Size = UDim2.fromOffset(30, 20)
            btn.BackgroundColor3 = Color3.fromRGB(132, 54, 217)
            btn.BackgroundTransparency = 0.25
            btn.BorderSizePixel = 0
            btn.AutoButtonColor = false
            btn.Font = Enum.Font.GothamBold
            btn.Text = text
            btn.TextSize = 14
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.ZIndex = 103
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 7)
            local bs = Instance.new("UIStroke", btn)
            bs.Color = TGT.OUTLINE
            bs.Thickness = 1
            bs.Transparency = 0.45
            bs.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            btn.MouseButton1Click:Connect(function()
                local v = math.clamp((tonumber(getFn()) or minV) + delta, minV, maxV)
                setFn(v)
                refresh()
            end)
        end
        stepper(-72, "-", -step)
        stepper(-38, "+", step)
        refresh()
        return refresh
    end

    local paintInvisMain = invToggle(1, "Invisible Steal",
        function() return _G.invisibleStealEnabled == true end,
        function() if _G.toggleInvisibleSteal then pcall(_G.toggleInvisibleSteal) end end)

    invToggle(2, "Auto Invis on Steal",
        function() return i.AutoInvisDuringSteal ~= false end,
        function(v)
            if _G.setAutoInvisSteal then _G.setAutoInvisSteal(v) else i.AutoInvisDuringSteal = v; U() end
        end)

    invToggle(3, "Auto Fix Lagback",
        function() return i.AutoRecoverLagback ~= false end,
        function(v)
            i.AutoRecoverLagback = v
            _G.StickyAutoRecoverLagback = v
            U()
        end)

    invToggle(4, "Invis Delay",
        function() return i.InvisDelayEnabled == true end,
        function(v)
            i.InvisDelayEnabled = v
            _G.StickyInvisDelayEnabled = v
            U()
        end)

    local refreshRot = invSlider(5, "Rotation", 180, 360, 1, false,
        function() return i.InvisStealAngle or 233 end,
        function(v)
            i.InvisStealAngle = v
            _G.StickyInvisAngle = v
            U()
        end)

    local refreshDepth = invSlider(6, "Depth", 0.5, 5, 0.1, true,
        function() return i.InvisSinkValue or 2.5 end,
        function(v)
            i.InvisSinkValue = v
            _G.StickyInvisSink = v
            U()
        end)

    local refreshDelay = invSlider(7, "Invis Delay (sec)", 0, 1, 0.1, true,
        function() return (tonumber(i.InvisDelayMs) or 0) / 1000 end,
        function(v)
            i.InvisDelayMs = math.floor(v * 1000 + 0.5)
            _G.StickyInvisDelayMs = i.InvisDelayMs
            i.InvisDelayEnabled = i.InvisDelayMs > 0
            _G.StickyInvisDelayEnabled = i.InvisDelayEnabled
            U()
        end)

    do
        local minimized = false
        invMinBtn.MouseButton1Click:Connect(function()
            minimized = not minimized
            invBody.Visible = not minimized
            invPanel.Size = UDim2.fromOffset(INV_W, minimized and 38 or INV_H)
            invMinBtn.Text = minimized and "+" or "-"
        end)
    end

    do
        local prevVisual = _G.StickyInvisVisual
        _G.StickyInvisVisual = function(on)
            if prevVisual then pcall(prevVisual, on) end
            pcall(paintInvisMain, on and true or false)
        end
        task.spawn(function()
            local last = nil
            while true do
                task.wait(0.3)
                local now = _G.invisibleStealEnabled == true
                if now ~= last then
                    last = now
                    if _G.StickyInvisVisual then pcall(_G.StickyInvisVisual, now) end
                end
            end
        end)
    end

    P.RevealPanels()
end)


_G.StickyAutoInvisSteal = i.AutoInvisDuringSteal ~= false
_G.StickyInvisDelayEnabled = i.InvisDelayEnabled == true
_G.StickyInvisDelayMs = tonumber(i.InvisDelayMs) or 0
_G.StickyInvisSink = tonumber(i.InvisSinkValue) or 2.5
_G.StickyAutoRecoverLagback = i.AutoRecoverLagback ~= false
do
    local ok, code = pcall(function()
        return Enum.KeyCode[i.InvisibleStealKey or "V"]
    end)
    _G.StickyInvisStealKey = ok and code or Enum.KeyCode.V
end

_G.setInvisibleSteal = function(on)
    on = on and true or false
    if (_G.invisibleStealEnabled == true) ~= on and _G.toggleInvisibleSteal then
        pcall(_G.toggleInvisibleSteal)
    end
end
_G.setAutoInvisSteal = function(on)
    on = on and true or false
    _G.StickyAutoInvisSteal = on
    i.AutoInvisDuringSteal = on
    if _G.StickySyncToggleUI then _G.StickySyncToggleUI("Auto Invis Steal", on) end
    if _G.StickySaveConfigNow then task.spawn(_G.StickySaveConfigNow) end
end

do
    local LocalPlayer = game:GetService("Players").LocalPlayer
    local Workspace = workspace
    local RunService = game:GetService("RunService")
    local UserInputService = game:GetService("UserInputService")
    local StickyAlive = function() return true end
    local Config = i
    local function _P(o, ...)
        for n = 1, select("#", ...), 2 do
            o[select(n, ...)] = select(n + 1, ...)
        end
        return o
    end
task.spawn((function()
	local animPlaying = false
	local tracks = {}
	local clone, oldRoot, hip, connection
	local _invisSF, _invisSFAtt
	local _invisPoseAtt, _invisPoseBase, _invisPoseRestDY
		local invisRagdollACs, invisRagdollACChar
	local folderConnections = {}
	local serverGhosts = {}
	local lagbackCallCount = 0
	local lagbackWindowStart = 0
	local lastLagbackTime = 0
	local errorOrbActive = false
	local errorOrb = nil
	local function clearErrorOrb()
		if errorOrb and errorOrb.Parent then errorOrb:Destroy() end
		errorOrb = nil; errorOrbActive = false
	end
	local createServerGhost; createServerGhost = (function(position)
		local now = tick()
		if now - lastLagbackTime < 0.05 then return end
		lastLagbackTime = now
		if now - lagbackWindowStart > 1 then lagbackCallCount = 0; lagbackWindowStart = now end
		lagbackCallCount = lagbackCallCount + 1
		if lagbackCallCount >= 5 then
			if _G.StickyAutoRecoverLagback and _G.invisibleStealEnabled and _G.toggleInvisibleSteal and not _G.StickyRecoveryInProgress then
				_G.StickyRecoveryInProgress = true
				task.defer(function()
					pcall(_G.toggleInvisibleSteal)
					task.wait(0.3)
					_G.StickyRecoveryInProgress = false
				end)
				return
			end end end)
	local function clearAllGhosts()
		serverGhosts = {}; clearErrorOrb(); lagbackCallCount = 0; lastLagbackTime = 0
	end
	local function removeFolders()
		local pf = Workspace:FindFirstChild(LocalPlayer.Name)
		if not pf then return end
		local dr = pf:FindFirstChild("DoubleRig")
		if dr then
			local rr = dr:FindFirstChild("HumanoidRootPart") or dr:FindFirstChildWhichIsA("BasePart")
			if rr then createServerGhost(rr.Position) end
			dr:Destroy()
		end
		local cs = pf:FindFirstChild("Constraints")
		if cs then cs:Destroy() end
		local conn = pf.ChildAdded:Connect(function(child)
			if child.Name == "DoubleRig" then
				task.defer(function()
					local rr = child:FindFirstChild("HumanoidRootPart") or child:FindFirstChildWhichIsA("BasePart")
					if rr then createServerGhost(rr.Position) end
					child:Destroy()
				end)
			elseif child.Name == "Constraints" then child:Destroy() end
		end)
		table.insert(folderConnections, conn)
	end
	local function doClone()
		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
			hip = character.Humanoid.HipHeight
			oldRoot = character:FindFirstChild("HumanoidRootPart")
			if not oldRoot or not oldRoot.Parent then return false end
			for _, c in pairs(oldRoot:GetChildren()) do if c:IsA("Attachment") and c.Name:find("Beam") then c:Destroy() end end
			for _, c in pairs(oldRoot:GetChildren()) do if c:IsA("Beam") then c:Destroy() end end
			local tmp = Instance.new("Model"); tmp.Parent = game
			character.Parent = tmp
			clone = oldRoot:Clone()
			for _, c in ipairs(clone:GetChildren()) do
				if c:IsA("LinearVelocity") or c:IsA("BodyVelocity") or c:IsA("VectorForce")
					or (c:IsA("Attachment") and (c.Name == "LVAttachment" or c.Name == "InvisLVAtt")) then
					pcall(function()
c:Destroy() end)
				end end
			clone.Parent = character
			clone.Anchored = false
			oldRoot.Parent = Workspace.CurrentCamera
			clone.CFrame = oldRoot.CFrame; character.PrimaryPart = clone
			character.Parent = Workspace
			for _, v in pairs(character:GetDescendants()) do
				if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
					if v.Part0 == oldRoot then v.Part0 = clone end
					if v.Part1 == oldRoot then v.Part1 = clone end
				end end
			_invisPoseAtt, _invisPoseBase, _invisPoseRestDY = nil, nil, nil
			pcall(function()
				local _a0 = clone:FindFirstChild("RootRigAttachment")
				local _lt = character:FindFirstChild("LowerTorso")
				local _a1 = _lt and _lt:FindFirstChild("RootRigAttachment")
				if _a0 and _a1 then _invisPoseAtt = _a0 _invisPoseBase = _a0.CFrame _invisPoseRestDY = _a0.CFrame.Position.Y - _a1.CFrame.Position.Y end
			end)
			tmp:Destroy(); return true
		end
		return false
	end
	local function revertClone()
		local character = LocalPlayer.Character
		local g_hum = character and character:FindFirstChildOfClass("Humanoid")
		if not oldRoot or not oldRoot:IsDescendantOf(Workspace) or not character or not g_hum or g_hum.Health <= 0 then return end
		local tmp = Instance.new("Model"); tmp.Parent = game
		character.Parent = tmp
		oldRoot.Parent = character; character.PrimaryPart = oldRoot
		character.Parent = Workspace; oldRoot.CanCollide = true
		local _rootAC
		for _, v in pairs(character:GetDescendants()) do
			if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
				if v.Part0 == clone then v.Part0 = oldRoot end
				if v.Part1 == clone then v.Part1 = oldRoot end
			elseif v.Name == "Root" and v:IsA("AnimationConstraint") then
				_rootAC = v
			end end
		if clone then local p = clone.CFrame; clone:Destroy(); clone = nil; oldRoot.CFrame = p end
		_invisPoseAtt, _invisPoseBase, _invisPoseRestDY = nil, nil, nil
		pcall(function()
			local _rra = oldRoot and oldRoot:FindFirstChild("RootRigAttachment")
			if _rra and _rootAC and (_rootAC.Attachment0 == nil or _rootAC.Attachment0.Parent == nil) then _rootAC.Attachment0 = _rra end
		end)
		oldRoot = nil
		if character and character.Humanoid then character.Humanoid.HipHeight = hip end
			_G.StickyInvisSurface = false
		clearAllGhosts()
	end
	local function animationTrickery()
		if not (animPlaying and _G.invisibleStealEnabled) then return end
		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
			local anim = Instance.new("Animation")
			anim.AnimationId = "http://www.roblox.com/asset/?id=18537363391"
			local humanoid = character.Humanoid
			local animator = humanoid:FindFirstChild("Animator") or Instance.new("Animator", humanoid)
			local animTrack = animator:LoadAnimation(anim)
			animTrack.Priority = Enum.AnimationPriority.Action4
			animTrack:Play(0, 1, 0); anim:Destroy()
			for _, t in ipairs(tracks) do pcall(function()
t:Stop() end) end
			table.clear(tracks)
			table.insert(tracks, animTrack)
			animTrack.Stopped:Connect(function()
if animPlaying and _G.invisibleStealEnabled then animationTrickery() end end)
			task.delay(0, function()
				animTrack.TimePosition = 0.7
				task.delay(0.3, function()
if animTrack then animTrack:AdjustSpeed(math.huge) end end)
			end) end end
	local function turnOff()
		animPlaying = false
		_G.invisibleStealEnabled = false
		_G.StickyInvisAutoActive = false
		for _, t in pairs(tracks) do pcall(function()
t:Stop() end) end
		tracks = {}
		do
			local _ch = LocalPlayer.Character
			local _hum = _ch and _ch:FindFirstChildOfClass("Humanoid")
			local _anr = _hum and _hum:FindFirstChildOfClass("Animator")
			if _anr then
				for _, ft in ipairs(_anr:GetPlayingAnimationTracks()) do
					if ft.Animation and tostring(ft.Animation.AnimationId):find("18537363391") then pcall(function()
ft:AdjustSpeed(1); ft:Stop(0) end) end
				end end end
		if connection then connection:Disconnect(); connection = nil end
		if _invisSF then pcall(function()
_invisSF:Destroy() end); _invisSF = nil end
		if _invisSFAtt then pcall(function()
_invisSFAtt:Destroy() end); _invisSFAtt = nil end
		for _, c in ipairs(folderConnections) do if c then c:Disconnect() end end
		folderConnections = {}
		pcall(revertClone)
		local character = LocalPlayer.Character
		local humanoid = character and character:FindFirstChildOfClass("Humanoid")
		local hrp = character and (character.PrimaryPart or character:FindFirstChild("HumanoidRootPart"))
		if character then
			pcall(function()
				for _, p in ipairs(character:GetDescendants()) do if p:IsA("BasePart") then p.Anchored = false end end
			end) end
		if hrp then
			pcall(function()
				local pos = hrp.Position
				hrp.CFrame = CFrame.new(pos.X, pos.Y, pos.Z) * CFrame.Angles(0, hrp.Orientation.Y * math.pi / 180, 0)
				hrp.AssemblyLinearVelocity = Vector3.zero
				hrp.AssemblyAngularVelocity = Vector3.zero
			end) end
		if humanoid then
			pcall(function()
humanoid.PlatformStand = false end)
			pcall(function()
humanoid.Sit = false end)
			pcall(function()
humanoid.AutoRotate = true end)
			pcall(function()
if humanoid.WalkSpeed <= 0 then humanoid.WalkSpeed = 16 end end)
			pcall(function()
if humanoid.JumpPower <= 0 and humanoid.UseJumpPower then humanoid.JumpPower = 50 end end)
			pcall(function()
humanoid.PlatformStand = false end)
			pcall(function()
humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end)
			pcall(function()
humanoid:ChangeState(Enum.HumanoidStateType.Freefall) end)
		end
		pcall(function()
			local cam = Workspace.CurrentCamera
			if cam and humanoid then cam.CameraSubject = humanoid cam.CameraType = Enum.CameraType.Custom end
		end)
		pcall(function()
			local pm = LocalPlayer:FindFirstChild("PlayerScripts")
			if pm then pm = pm:FindFirstChild("PlayerModule") end
			if pm then require(pm):GetControls():Enable() end
		end)
		clearAllGhosts()
		if _G.StickyInvisVisual then pcall(_G.StickyInvisVisual, false) end
	end
	local function turnOn()
		if not StickyAlive() then return end
		if animPlaying then return end
		local character = LocalPlayer.Character
		if not character then return end
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if not humanoid then return end
		animPlaying = true; _G.invisibleStealEnabled = true
		if _G.StickyInvisVisual then pcall(_G.StickyInvisVisual, true) end
		tracks = {}; removeFolders()
		local success = doClone()
		if success then
			if _G.StickyInvisFreezeOnEngage ~= false then
				pcall(function()
					local pm = LocalPlayer:FindFirstChild("PlayerScripts")
					pm = pm and pm:FindFirstChild("PlayerModule")
					if pm then require(pm):GetControls():Enable() end
				end)
				local _fc = LocalPlayer.Character
				local _froot = _fc and (_fc.PrimaryPart or _fc:FindFirstChild("HumanoidRootPart"))
				local _fhum = _fc and _fc:FindFirstChildOfClass("Humanoid")
				if _froot then pcall(function()
_froot.AssemblyLinearVelocity = Vector3.zero; _froot.AssemblyAngularVelocity = Vector3.zero; _froot.Anchored = false end) end
				if oldRoot then pcall(function()
oldRoot.AssemblyLinearVelocity = Vector3.zero; oldRoot.AssemblyAngularVelocity = Vector3.zero end) end
				if _fhum then
					pcall(function()
_fhum:Move(Vector3.zero, false) end)
					pcall(function()
if _fhum.PlatformStand then _fhum.PlatformStand = false end end)
					pcall(function()
if _fhum.Sit then _fhum.Sit = false end end)
					pcall(function()
if _fhum.WalkSpeed <= 0 then _fhum.WalkSpeed = 16 end end)
				end end
				if _G.StickyInvisFold ~= false then task.wait(0.05); animationTrickery() end
			task.defer(function()
				if _G.resetPlotBeam then pcall(_G.resetPlotBeam) end
				task.wait(0.1)
				if _G.createPlotBeam then pcall(_G.createPlotBeam) end
			end)
			local lastSetPosition = nil; local skipFrames = 5
			local _invisFastMode = false; local _invisSlowAccum = 0
			local _invisRagGuard = function(_char, _hum)
				if (LocalPlayer:GetAttribute("RagdollEndTime") or 0) > 0 then LocalPlayer:SetAttribute("RagdollEndTime", 0) end
				if _hum then
					if _hum.PlatformStand then _hum.PlatformStand = false end
					local _s = _hum:GetState()
					if _s == Enum.HumanoidStateType.Physics or _s == Enum.HumanoidStateType.Ragdoll or _s == Enum.HumanoidStateType.FallingDown then
						_hum:ChangeState(Enum.HumanoidStateType.GettingUp)
					end end
				if invisRagdollACChar ~= _char then
					invisRagdollACChar = _char; invisRagdollACs = {}
					for _, d in ipairs(_char:GetDescendants()) do if d:IsA("AnimationConstraint") then table.insert(invisRagdollACs, d) end end
				end
				for _, ac in ipairs(invisRagdollACs) do if ac.Parent and not ac.Enabled then ac.Enabled = true end end
			end
			connection = RunService.PreSimulation:Connect(function(_psDt)
				local _invHum = character and character:FindFirstChildOfClass("Humanoid")
				if _invHum and _invHum.Health > 0 and oldRoot then
					local root = character.PrimaryPart or character:FindFirstChild("HumanoidRootPart")
					if root then
							if _G.StickyInvisRagdollGuard ~= false then pcall(_invisRagGuard, character, _invHum) end
							if _G.StickyInvisPoseFix ~= false and _invisPoseAtt and _invisPoseAtt.Parent and _invisPoseBase and _invisPoseRestDY then
								local _plt = character:FindFirstChild("LowerTorso")
								if _plt then
									local _appl = _invisPoseBase.Position.Y - _invisPoseAtt.CFrame.Position.Y
									local _lift = (_plt.Position.Y - root.Position.Y) + _appl - _invisPoseRestDY
									_lift = math.clamp(_lift, 0, tonumber(_G.StickyInvisPoseMaxLift) or 8)
									if math.abs(_lift - _appl) > 0.01 then _invisPoseAtt.CFrame = _invisPoseBase * CFrame.new(0, -_lift, 0) end
								end end
							if skipFrames > 0 then skipFrames = skipFrames - 1; lastSetPosition = nil
						elseif lastSetPosition then
							local currentPos = oldRoot.Position
							local jumpDist = (currentPos - lastSetPosition).Magnitude
							local _lbDt = tonumber(_psDt) or (1/60)
							local _lbThresh = math.max(3, root.AssemblyLinearVelocity.Magnitude * _lbDt * 3 + 0.5)
							if jumpDist > _lbThresh and not _G.StickyRecoveryInProgress then
								lastSetPosition = nil; createServerGhost(currentPos)
									if _G.StickyAutoRecoverLagback and _G.toggleInvisibleSteal then
										_G.StickyRecoveryInProgress = true
										task.defer(function()
											pcall(_G.toggleInvisibleSteal); task.wait(0.5)
											pcall(_G.toggleInvisibleSteal); _G.StickyRecoveryInProgress = false
										end) end end end
						if clone then clone.CanCollide = (_G.StickyInvisCloneCollide ~= false) end
						if not oldRoot then return end
						for _, c in pairs(oldRoot:GetChildren()) do if c:IsA("Beam") or (c:IsA("Attachment") and c.Name:find("Beam")) then c:Destroy() end end
						local rotAngle = _G.StickyInvisAngle or 180
						local sa = math.clamp(_G.StickyInvisSink or Config.InvisSinkValue or 2.5, 0.5, 10)
						local _rv = root.AssemblyLinearVelocity
						local _xzSpd = Vector3.new(_rv.X, 0, _rv.Z).Magnitude
						local _hyDt = tonumber(_psDt) or (1/60)
						if _invisFastMode then
							if _xzSpd < (_G.StickyInvisFastExit or 17) then
								_invisSlowAccum = _invisSlowAccum + _hyDt
								if _invisSlowAccum > 0.15 then _invisFastMode = false end
							else
								_invisSlowAccum = 0
							end
						elseif _xzSpd > (_G.StickyInvisFastEnter or 22) then
							_invisFastMode = true; _invisSlowAccum = 0
						end
						if _invisFastMode and _G.StickyInvisSpeedFix ~= false and not _G.StickyInvisSurface then
							if not _invisSF or _invisSF.Parent ~= oldRoot then
								if _invisSF then pcall(function()
_invisSF:Destroy() end) end
								if _invisSFAtt then pcall(function()
_invisSFAtt:Destroy() end) end
								_invisSFAtt = Instance.new("Attachment"); _invisSFAtt.Name = "InvisLVAtt"; _invisSFAtt.Parent = oldRoot
								_invisSF = Instance.new("LinearVelocity"); _invisSF.Name = "InvisSpeedForce"
								_P(_invisSF, "Attachment0", _invisSFAtt, "RelativeTo", Enum.ActuatorRelativeTo.World, "VelocityConstraintMode", Enum.VelocityConstraintMode.Vector, "ForceLimitMode", Enum.ForceLimitMode.PerAxis)
								_invisSF.MaxAxesForce = Vector3.new(math.huge, math.huge, math.huge)
								_invisSF.Parent = oldRoot
							end
							local _yv = math.clamp(((root.Position.Y - sa) - oldRoot.Position.Y) * (_G.StickyInvisYGain or 25), -150, 150)
							local _gx = _G.StickyInvisXZGain or 8
							local _vx = _rv.X + math.clamp((root.Position.X - oldRoot.Position.X) * _gx, -15, 15)
							local _vz = _rv.Z + math.clamp((root.Position.Z - oldRoot.Position.Z) * _gx, -15, 15)
							local _hv = Vector3.new(_vx, 0, _vz)
							local _mxz = _G.StickyInvisMaxXZ or 65
							if _hv.Magnitude > _mxz then _hv = _hv.Unit * _mxz end
							_invisSF.VectorVelocity = Vector3.new(_hv.X, _yv, _hv.Z)
							_invisSF.Enabled = true
							oldRoot.CFrame = CFrame.new(oldRoot.Position) * root.CFrame.Rotation * CFrame.Angles(math.rad(rotAngle), 0, 0)
							oldRoot.CanCollide = false
						else
							if _invisSF and _invisSF.Enabled then _invisSF.Enabled = false end
							local cf = root.CFrame - Vector3.new(0, sa, 0)
							oldRoot.CFrame = cf * CFrame.Angles(math.rad(rotAngle), 0, 0)
							if _G.StickyInvisSurface then oldRoot.CFrame = root.CFrame end
							oldRoot.AssemblyLinearVelocity = root.AssemblyLinearVelocity; oldRoot.CanCollide = false
						end
						lastSetPosition = _G.StickyInvisSurface and nil or oldRoot.Position
					end end end) end end
	local function _invisOrphanInCamera()
		local found = nil
		pcall(function()
			if Workspace.CurrentCamera then
				for _, c in ipairs(Workspace.CurrentCamera:GetChildren()) do
					if c:IsA("BasePart") and c.Name == "HumanoidRootPart" then found = c break end
				end end end)
		return found
	end
	local function _smartRevert()
		local character = LocalPlayer.Character
		if not character then return false end
		local orphans = {}
		if Workspace.CurrentCamera then
			for _, c in ipairs(Workspace.CurrentCamera:GetChildren()) do
				if c:IsA("BasePart") and c.Name == "HumanoidRootPart" then table.insert(orphans, c) end
			end end
		local fakeHRP = character.PrimaryPart
		if not fakeHRP or fakeHRP.Name ~= "HumanoidRootPart" then fakeHRP = character:FindFirstChild("HumanoidRootPart") end
		if #orphans == 0 then return false end
		local realHRP = orphans[1]
		local ok = pcall(function()
			local tmp = Instance.new("Model"); tmp.Parent = game
			local origParent = character.Parent
			character.Parent = tmp
			realHRP.Parent = character
			character.PrimaryPart = realHRP
			character.Parent = origParent or Workspace
			realHRP.CanCollide = true
			if fakeHRP and fakeHRP ~= realHRP then
				for _, v in pairs(character:GetDescendants()) do
					if v:IsA("Weld") or v:IsA("Motor6D") or v:IsA("WeldConstraint") then
						if v.Part0 == fakeHRP then v.Part0 = realHRP end
						if v.Part1 == fakeHRP then v.Part1 = realHRP end
					end end
				local p = fakeHRP.CFrame
				fakeHRP:Destroy()
				realHRP.CFrame = p
				pcall(function()
					local rra = realHRP:FindFirstChild("RootRigAttachment")
					if rra then
						for _, d in ipairs(character:GetDescendants()) do
							if d:IsA("AnimationConstraint") and d.Name == "Root" then
								if (d.Attachment0 == nil or d.Attachment0.Parent == nil) then d.Attachment0 = rra end
								break
							end end end end) end
			for i = 2, #orphans do
				pcall(function()  if orphans[i].Parent then orphans[i]:Destroy() end end)
			end
			local humanoid = character:FindFirstChildOfClass("Humanoid")
			if humanoid then pcall(function()
humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end) end
		end)
		return ok
	end
	_G.toggleInvisibleSteal = function()
		local invisLooksOn = animPlaying
			or _G.invisibleStealEnabled
			or (oldRoot and oldRoot.Parent ~= nil)
			or (clone and clone.Parent ~= nil)
			or (_invisOrphanInCamera() ~= nil)
		if invisLooksOn then
			pcall(turnOff)
			if _invisOrphanInCamera() then pcall(_smartRevert) end
			task.delay(0.25, function()
				if _invisOrphanInCamera() then pcall(function()
LocalPlayer:LoadCharacter() end) end
			end)
		else
			turnOn()
		end end
	task.defer(function()
		task.wait(0.5)
		if _invisOrphanInCamera() then pcall(_smartRevert) end
	end)
	UserInputService.InputBegan:Connect(function(input)
		if UserInputService:GetFocusedTextBox() then return end
		if _G.StickyInvisStealKey and input.KeyCode == _G.StickyInvisStealKey then pcall(_G.toggleInvisibleSteal) end
	end)
	local function onCharacterAdded(newChar)
		clearErrorOrb(); clearAllGhosts(); lagbackCallCount = 0
		pcall(function()
for _, c in pairs(Workspace.CurrentCamera:GetChildren()) do if c:IsA("BasePart") and c.Name == "HumanoidRootPart" then c:Destroy() end end end)
		if oldRoot then pcall(function()
oldRoot:Destroy() end); oldRoot = nil end
		if clone then pcall(function()
clone:Destroy() end); clone = nil end
		animPlaying = false; _G.invisibleStealEnabled = false
		task.wait(0.2)
		local camera = Workspace.CurrentCamera
		if camera and newChar then
			local h = newChar:FindFirstChildOfClass("Humanoid")
			if h then camera.CameraSubject = h; camera.CameraType = Enum.CameraType.Custom end
		end end
    LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
    local function setupDeathListener()
        local ch = LocalPlayer.Character
        if ch then
            local h = ch:FindFirstChildOfClass("Humanoid")
            if h then h.Died:Connect(function()
                clearErrorOrb(); clearAllGhosts(); lagbackCallCount = 0
                if _G.StickyForceRespawn then _G.StickyForceRespawn("deathListener") end
            end) end
        end end
    setupDeathListener()
    LocalPlayer.CharacterAdded:Connect(function()
task.wait(0.1); setupDeathListener() end)
end))
LocalPlayer.CharacterAdded:Connect(function()  _G.StickyInvisAutoActive = false end)
local function stickyOnOwnPlotSteal()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local plots = Workspace:FindFirstChild("Plots")
    if not hrp or not plots then return false end
    local myPlot
    for _, plot in ipairs(plots:GetChildren()) do
        local sign = plot:FindFirstChild("PlotSign")
        if sign then
            local yb = sign:FindFirstChild("YourBase")
            if yb and yb:IsA("BillboardGui") and yb.Enabled then myPlot = plot; break end
        end end
    if not myPlot then return false end
    local sh = myPlot:FindFirstChild("StealHitbox")
    if not sh or not sh:IsA("BasePart") then return false end
    local rel = sh.CFrame:PointToObjectSpace(hrp.Position)
    local half = sh.Size * 0.5
    local m = 5
    return math.abs(rel.X) <= half.X + m and math.abs(rel.Y) <= half.Y + m and math.abs(rel.Z) <= half.Z + m
end
_G.StickyOnOwnPlotSteal = stickyOnOwnPlotSteal
task.spawn(function()
    task.wait(1)
    local lastStealing = LocalPlayer:GetAttribute("Stealing") == true
    local notStealTicks = 0
    local invisDelaySeq = 0
    while true do
        task.wait(0.1)
        if _G.StickyAutoInvisSteal ~= false and _G.toggleInvisibleSteal then
            local stealing = LocalPlayer:GetAttribute("Stealing") == true
            if stealing and not lastStealing then
                if not _G.invisibleStealEnabled and not stickyOnOwnPlotSteal() then
                    local delayMs = (_G.StickyInvisDelayEnabled and tonumber(_G.StickyInvisDelayMs)) or 0
                    if delayMs > 0 then
                        invisDelaySeq = invisDelaySeq + 1
                        local mySeq = invisDelaySeq
                        task.delay(delayMs / 1000, function()
                            if mySeq ~= invisDelaySeq then return end
                            if _G.StickyAutoInvisSteal == false then return end
                            if LocalPlayer:GetAttribute("Stealing") ~= true then return end
                            if _G.invisibleStealEnabled then return end
                            pcall(_G.toggleInvisibleSteal)
                            _G.StickyInvisAutoActive = true
                        end)
                    else
                        pcall(_G.toggleInvisibleSteal)
                        _G.StickyInvisAutoActive = true
                    end end
            elseif (not stealing) and lastStealing then
                invisDelaySeq = invisDelaySeq + 1
                if _G.invisibleStealEnabled and _G.StickyInvisAutoActive then
                    pcall(_G.toggleInvisibleSteal)
                    if not _G.invisibleStealEnabled then _G.StickyInvisAutoActive = false end
                end end
            if (not stealing) and _G.invisibleStealEnabled then
                notStealTicks = notStealTicks + 1
                if (notStealTicks >= 4 and not _G.StickyRecoveryInProgress) or notStealTicks >= 20 then
                    pcall(_G.toggleInvisibleSteal); _G.StickyInvisAutoActive = false; _G.StickyRecoveryInProgress = false; notStealTicks = 0
                end
            else
                notStealTicks = 0
            end
            if (not stealing) and not _G.invisibleStealEnabled then
                local _ch = LocalPlayer.Character
                local _hum = _ch and _ch:FindFirstChildOfClass("Humanoid")
                local _anr = _hum and _hum:FindFirstChildOfClass("Animator")
                if _anr then
                    for _, ft in ipairs(_anr:GetPlayingAnimationTracks()) do
                        if ft.Animation and tostring(ft.Animation.AnimationId):find("18537363391") then
                            pcall(function()
ft:AdjustSpeed(1); ft:Stop(0) end)
                        end end end end
            lastStealing = stealing
        end end end)
end

do
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local Workspace = workspace
    local LP = Players.LocalPlayer

    local function plotsFolder()
        return Workspace:FindFirstChild("Plots")
    end

    local SlotBoxes = {}
    do
        local FLOOR_TOL, SAME_COL_R = 8, 3
        local cache = setmetatable({}, { __mode = "k" })

        local KNOWN_OFFSETS = { 18, 35 }
        local DEFAULT_SIZE = Vector3.new(6, 1, 6)

        SlotBoxes.PODIUM_LIMIT = 11

        local canonSize = nil

        local ownCache, ownCacheAt = setmetatable({}, { __mode = "k" }), 0

        local function isOwnPlot(plot)
            local now = os.clock()
            if now - ownCacheAt > 2 then
                ownCache = setmetatable({}, { __mode = "k" })
                ownCacheAt = now
            end

            local hit = ownCache[plot]
            if hit ~= nil then return hit end

            local mine = false
            local sign = plot:FindFirstChild("PlotSign")
            if sign then
                local yb = sign:FindFirstChild("YourBase")
                if yb and yb:IsA("BillboardGui") and yb.Enabled then mine = true end
            end

            ownCache[plot] = mine
            return mine
        end
        SlotBoxes.IsOwnPlot = isOwnPlot

        local function canonVolume(s)
            if not s then return math.huge end
            return s.X * s.Y * s.Z
        end

        local function readAnchor(slot)
            local base = slot:FindFirstChild("Base")
            local sp = base and base:FindFirstChild("Spawn")
            if sp and sp:IsA("BasePart") then return sp.CFrame, true end
            if base and base:IsA("BasePart") then return base.CFrame, true end
            local ok, piv = pcall(function() return slot:GetPivot() end)
            if ok then return piv, false end
            return nil, false
        end

        local function slotAnchor(slot)
            local c = cache[slot]
            if c and c.static then return c.cf, true end

            local cf, static = readAnchor(slot)
            if not cf then
                if c then return c.cf, false end
                return nil, false
            end
            if c and not static then return c.cf, false end

            cache[slot] = { cf = cf, static = static }
            return cf, static
        end

        local function rawSize(slot)
            local base = slot:FindFirstChild("Base")
            if base and base:IsA("BasePart") then return base.Size end
            if base and base:IsA("Model") then
                local ok, _, s = pcall(function() return base:GetBoundingBox() end)
                if ok then return s end
            end
            return nil
        end

        local function measureCanon()
            local plots = plotsFolder()
            local seen = {}
            if plots then
                for _, plot in ipairs(plots:GetChildren()) do
                    local pods = plot:FindFirstChild("AnimalPodiums")
                    if pods then
                        for _, sl in ipairs(pods:GetChildren()) do
                            local s = rawSize(sl)
                            if s then
                                local key = string.format("%.2f|%.2f|%.2f", s.X, s.Y, s.Z)
                                local e = seen[key]
                                if e then e.n = e.n + 1 else seen[key] = { n = 1, size = s } end
                            end
                        end
                    end
                end
            end

            local best, bestN = nil, 0
            for _, e in pairs(seen) do
                if e.n > bestN
                    or (e.n == bestN and canonVolume(e.size) < canonVolume(best)) then
                    best, bestN = e.size, e.n
                end
            end

            if best then canonSize = best end
            return canonSize or DEFAULT_SIZE
        end

        local function columnTaken(live, p)
            for _, q in ipairs(live) do
                local dx, dz = q.X - p.X, q.Z - p.Z
                if math.sqrt(dx * dx + dz * dz) <= SAME_COL_R
                    and math.abs(q.Y - p.Y) <= FLOOR_TOL then
                    return true
                end
            end
            return false
        end

        local function floorOffsets()
            local plots = plotsFolder()
            local best
            if plots then
                for _, plot in ipairs(plots:GetChildren()) do
                    local pods = plot:FindFirstChild("AnimalPodiums")
                    if pods then
                        local ys = {}
                        for _, sl in ipairs(pods:GetChildren()) do
                            local cf = slotAnchor(sl)
                            if cf then ys[#ys + 1] = cf.Position.Y end
                        end
                        table.sort(ys)
                        local lv = {}
                        for _, y in ipairs(ys) do
                            local found = false
                            for _, l in ipairs(lv) do
                                if math.abs(l - y) <= FLOOR_TOL then found = true break end
                            end
                            if not found then lv[#lv + 1] = y end
                        end
                        if not best or #lv > #best then best = lv end
                    end
                end
            end
            local offs = {}
            if best and #best >= 2 then
                for n = 2, #best do offs[#offs + 1] = best[n] - best[1] end
            end
            if #offs < #KNOWN_OFFSETS then
                offs = {}
                for n, v in ipairs(KNOWN_OFFSETS) do offs[n] = v end
            end
            return offs
        end

        function SlotBoxes.Collect()
            local out = {}
            local plots = plotsFolder()
            if not plots then return out end
            local offs = floorOffsets()
            local sz = measureCanon()
            local halfY = sz.Y * 0.5

            for _, plot in ipairs(plots:GetChildren()) do
                local pods = plot:FindFirstChild("AnimalPodiums")
                if pods and not isOwnPlot(plot) then
                    local anchors, rot, anyRot = {}, nil, nil
                    for _, sl in ipairs(pods:GetChildren()) do
                        local cf, static = slotAnchor(sl)
                        if cf then
                            anchors[#anchors + 1] = cf
                            if not anyRot then anyRot = cf - cf.Position end
                            if static and not rot then rot = cf - cf.Position end
                        end
                    end
                    rot = rot or anyRot

                    if #anchors > 0 then
                        local ys = {}
                        for _, cf in ipairs(anchors) do ys[#ys + 1] = cf.Position.Y end
                        table.sort(ys)

                        local lv = {}
                        for _, y in ipairs(ys) do
                            local hit = nil
                            for _, l in ipairs(lv) do
                                if math.abs(l[1] - y) <= FLOOR_TOL then hit = l break end
                            end
                            if hit then hit[#hit + 1] = y else lv[#lv + 1] = { y } end
                        end

                        local levelY = {}
                        for n, l in ipairs(lv) do levelY[n] = l[math.ceil(#l * 0.5)] end

                        local function snapY(y)
                            local pick, bd = y, math.huge
                            for _, ly in ipairs(levelY) do
                                local d = math.abs(ly - y)
                                if d < bd then bd, pick = d, ly end
                            end
                            return pick
                        end

                        local slots, live, minY = {}, {}, math.huge
                        for _, cf in ipairs(anchors) do
                            local p = cf.Position
                            local flat = rot + Vector3.new(p.X, snapY(p.Y) - halfY, p.Z)
                            slots[#slots + 1] = { cf = flat, sz = sz, plot = plot }
                            live[#live + 1] = flat.Position
                            minY = math.min(minY, flat.Position.Y)
                        end

                        local plotBoxes = {}
                        for _, s in ipairs(slots) do plotBoxes[#plotBoxes + 1] = s end
                        for _, s in ipairs(slots) do
                            if s.cf.Position.Y <= minY + FLOOR_TOL then
                                for _, dy in ipairs(offs) do
                                    local up = s.cf + Vector3.new(0, dy, 0)
                                    if not columnTaken(live, up.Position) then
                                        plotBoxes[#plotBoxes + 1] = { cf = up, sz = sz, plot = plot }
                                        live[#live + 1] = up.Position
                                    end
                                end
                            end
                        end

                        local tiers = {}
                        for _, bx in ipairs(plotBoxes) do
                            local y = bx.cf.Position.Y
                            local known = false
                            for _, ty in ipairs(tiers) do
                                if math.abs(ty - y) <= FLOOR_TOL then known = true break end
                            end
                            if not known then tiers[#tiers + 1] = y end
                        end

                        local topTier = -math.huge
                        for _, ty in ipairs(tiers) do
                            if ty > topTier then topTier = ty end
                        end

                        for _, bx in ipairs(plotBoxes) do
                            bx.isTop = #tiers >= 2 and bx.cf.Position.Y >= topTier - FLOOR_TOL
                            out[#out + 1] = bx
                        end
                    end
                end
            end
            return out
        end

        local countCache = setmetatable({}, { __mode = "k" })
        local countAt = setmetatable({}, { __mode = "k" })

        function SlotBoxes.PodiumCount(plot)
            if typeof(plot) == "string" then
                local plots = plotsFolder()
                plot = plots and plots:FindFirstChild(plot)
            end
            if typeof(plot) ~= "Instance" then return 0 end

            local hit = countCache[plot]
            if hit and (os.clock() - (countAt[plot] or 0)) < 1 then return hit end

            local pods = plot:FindFirstChild("AnimalPodiums")
            if not pods then return 0 end

            local real, total = 0, 0
            for _, sl in ipairs(pods:GetChildren()) do
                total = total + 1
                if sl:FindFirstChild("Base") then real = real + 1 end
            end
            if real == 0 then real = total end

            countCache[plot] = real
            countAt[plot] = os.clock()
            return real
        end

        local sigBuf = {}

        function SlotBoxes.Signature()
            local plots = plotsFolder()
            local n = 0
            if plots then
                for _, plot in ipairs(plots:GetChildren()) do
                    local pods = plot:FindFirstChild("AnimalPodiums")
                    if pods then
                        n = n + 1
                        sigBuf[n] = plot.Name
                            .. ":" .. tostring(SlotBoxes.PodiumCount(plot))
                            .. ":" .. (isOwnPlot(plot) and "1" or "0")
                    end
                end
            end
            for k = #sigBuf, n + 1, -1 do sigBuf[k] = nil end
            table.sort(sigBuf)
            return table.concat(sigBuf, "|")
        end
    end

    local enabled = false
    local holder = nil
    local buildPending = false
    local conns = {}
    local watchGen = 0
    local floors = {}

    local LEVEL_TOL = 8
    local CLUSTER_R = 60
    local THICKNESS = 1
    local EDGE_PAD = 2

    local FLOOR_COLOR = Color3.fromRGB(220, 40, 40)

    local function makeFloor(cf, size)
        if not holder then return end
        local p = Instance.new("Part")
        p.Name = "StickyFloorPlatform"
        p.Anchored = true
        p.CanCollide = true
        p.CanQuery = false
        p.CanTouch = false
        p.Massless = true
        p.Transparency = 1
        p.Material = Enum.Material.SmoothPlastic
        p.TopSurface = Enum.SurfaceType.Smooth
        p.BottomSurface = Enum.SurfaceType.Smooth
        p.Size = size
        p.CFrame = cf
        p.Parent = holder

        local box = Instance.new("SelectionBox")
        box.Adornee = p
        box.Color3 = FLOOR_COLOR
        box.SurfaceColor3 = FLOOR_COLOR
        box.LineThickness = 0.08
        box.Transparency = 0
        box.SurfaceTransparency = 0.7
        box.Parent = p
    end

    local CORNERS = {
        Vector3.new(-1, -1, -1), Vector3.new(-1, -1, 1),
        Vector3.new(-1,  1, -1), Vector3.new(-1,  1, 1),
        Vector3.new( 1, -1, -1), Vector3.new( 1, -1, 1),
        Vector3.new( 1,  1, -1), Vector3.new( 1,  1, 1),
    }

    local function rotOf(cf)
        return cf - cf.Position
    end

    local function sameRot(a, b)
        return a.RightVector:Dot(b.RightVector) > 0.999
            and a.UpVector:Dot(b.UpVector) > 0.999
    end

    local function cornersOf(cf, sz)
        local hx, hy, hz = sz.X * 0.5, sz.Y * 0.5, sz.Z * 0.5
        local out = {}
        for n, s in ipairs(CORNERS) do
            out[n] = cf * Vector3.new(s.X * hx, s.Y * hy, s.Z * hz)
        end
        return out
    end

    local function localBounds(rot, pts)
        local minX, maxX = math.huge, -math.huge
        local minZ, maxZ = math.huge, -math.huge
        local minY = math.huge
        for n = 1, #pts do
            local lp = rot:PointToObjectSpace(pts[n])
            if lp.X < minX then minX = lp.X end
            if lp.X > maxX then maxX = lp.X end
            if lp.Y < minY then minY = lp.Y end
            if lp.Z < minZ then minZ = lp.Z end
            if lp.Z > maxZ then maxZ = lp.Z end
        end
        return minX, maxX, minY, minZ, maxZ
    end

    local function build()
        if not holder then return end
        for _, c in ipairs(holder:GetChildren()) do c:Destroy() end
        floors = {}

        local boxes = SlotBoxes.Collect()
        if #boxes == 0 then return end

        local groups = {}
        for _, bx in ipairs(boxes) do
            local rot = rotOf(bx.cf)
            local pts = cornersOf(bx.cf, bx.sz)

            local hit = nil
            for _, g in ipairs(groups) do
                if g.plot == bx.plot and sameRot(g.rot, rot) then
                    local aX, bX, aY, aZ, bZ = localBounds(g.rot, pts)
                    if math.abs(g.y - aY) <= LEVEL_TOL
                        and bX >= (g.minX - CLUSTER_R) and aX <= (g.maxX + CLUSTER_R)
                        and bZ >= (g.minZ - CLUSTER_R) and aZ <= (g.maxZ + CLUSTER_R) then
                        hit = g
                        hit.minX = math.min(hit.minX, aX)
                        hit.maxX = math.max(hit.maxX, bX)
                        hit.minZ = math.min(hit.minZ, aZ)
                        hit.maxZ = math.max(hit.maxZ, bZ)
                        hit.y = math.min(hit.y, aY)
                        hit.n = hit.n + 1
                        hit.top = hit.top or (bx.isTop and true or false)
                        break
                    end
                end
            end

            if not hit then
                local aX, bX, aY, aZ, bZ = localBounds(rot, pts)
                groups[#groups + 1] = {
                    plot = bx.plot, rot = rot, y = aY, n = 1,
                    top = bx.isTop and true or false,
                    minX = aX, maxX = bX, minZ = aZ, maxZ = bZ,
                }
            end
        end

        for _, g in ipairs(groups) do
            if g.n >= 2 and g.top and SlotBoxes.PodiumCount(g.plot) < SlotBoxes.PODIUM_LIMIT then
                local sx = (g.maxX - g.minX) + EDGE_PAD * 2
                local sz = (g.maxZ - g.minZ) + EDGE_PAD * 2
                local cx = (g.minX + g.maxX) * 0.5
                local cz = (g.minZ + g.maxZ) * 0.5
                local cf = g.rot * CFrame.new(cx, g.y - THICKNESS * 0.5, cz)
                local size = Vector3.new(sx, THICKNESS, sz)
                makeFloor(cf, size)

                local c = cf.Position
                local hx = sx * 0.5 + 1
                local hy = THICKNESS * 0.5 + 6
                local hz = sz * 0.5 + 1
                floors[#floors + 1] = {
                    cf = cf, sz = size, plot = g.plot,
                    px = c.X, py = c.Y, pz = c.Z,
                    r2 = hx * hx + hy * hy + hz * hz,
                }
            end
        end
    end

    local function floorHit(f, pos)
        local lp = f.cf:PointToObjectSpace(pos)
        local hx, hy, hz = f.sz.X * 0.5, f.sz.Y * 0.5, f.sz.Z * 0.5
        return math.abs(lp.X) <= hx + 1 and math.abs(lp.Z) <= hz + 1
            and lp.Y >= hy - 1.5 and lp.Y <= hy + 6
    end

    local lastFloor = nil

    local function onTopFloor(pos)
        local memo = lastFloor and floors[lastFloor]
        if memo and floorHit(memo, pos) then return true, memo.plot end

        local px, py, pz = pos.X, pos.Y, pos.Z
        for n = 1, #floors do
            local f = floors[n]
            local dx, dy, dz = px - f.px, py - f.py, pz - f.pz
            if dx * dx + dy * dy + dz * dz <= f.r2 and floorHit(f, pos) then
                lastFloor = n
                return true, f.plot
            end
        end
        lastFloor = nil
        return false, nil
    end

    RunService.Heartbeat:Connect(function()
        if not enabled or #floors == 0 then return end
        if _G.StickyFloorEquipCarpet ~= true then return end
        if _G.StickyCarpetSpeed == true then return end

        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum or hum.Health <= 0 then return end

        if not onTopFloor(hrp.Position) then return end
        if _G.StickyEquipCarpet then pcall(_G.StickyEquipCarpet) end
    end)

    local function rebuild()
        if buildPending then return end
        buildPending = true
        task.delay(0.4, function()
            buildPending = false
            if enabled then pcall(build) end
        end)
    end

    local function setFloorPlatform(on)
        enabled = on and true or false
        i.FloorPlatform = enabled

        local old = Workspace:FindFirstChild("__StickyFloorPlatforms")
        if old then pcall(function() old:Destroy() end) end
        holder = nil
        floors = {}
        lastFloor = nil

        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        conns = {}
        watchGen = watchGen + 1

        if enabled then
            holder = Instance.new("Folder")
            holder.Name = "__StickyFloorPlatforms"
            holder.Parent = Workspace

            pcall(build)

            local plots = plotsFolder()
            if plots then
                local function watch(plot)
                    local pods = plot:FindFirstChild("AnimalPodiums")
                    if pods then
                        conns[#conns + 1] = pods.ChildAdded:Connect(rebuild)
                        conns[#conns + 1] = pods.ChildRemoved:Connect(rebuild)
                    else
                        conns[#conns + 1] = plot.ChildAdded:Connect(function(c)
                            if c.Name == "AnimalPodiums" then
                                conns[#conns + 1] = c.ChildAdded:Connect(rebuild)
                                conns[#conns + 1] = c.ChildRemoved:Connect(rebuild)
                                rebuild()
                            end
                        end)
                    end
                end

                for _, plot in ipairs(plots:GetChildren()) do watch(plot) end
                conns[#conns + 1] = plots.ChildAdded:Connect(function(plot)
                    watch(plot)
                    rebuild()
                end)
                conns[#conns + 1] = plots.ChildRemoved:Connect(rebuild)
            end

            local mine = watchGen
            task.spawn(function()
                local lastSig = nil
                while mine == watchGen and enabled do
                    local ok, sig = pcall(SlotBoxes.Signature)
                    if ok and sig ~= lastSig then
                        if lastSig ~= nil then rebuild() end
                        lastSig = sig
                    end
                    task.wait(2)
                end
            end)
        end

        if _G.StickySyncToggleUI then
            _G.StickySyncToggleUI("Floor Platform", enabled)
        end
        if _G.StickySaveConfigNow then
            task.spawn(function() pcall(_G.StickySaveConfigNow) end)
        else
            U()
        end
    end

    _G.StickySetFloorPlatform = setFloorPlatform
    _G.StickyIsFloorPlatformOn = function() return enabled end

    _G.StickyOnBoot(function()
        if i.FloorPlatform == true then
            task.spawn(function() pcall(setFloorPlatform, true) end)
        end
    end)
end

task.spawn(function()
    if _G.AntiTrap and _G.AntiTrap.Cleanup then
        pcall(_G.AntiTrap.Cleanup)
    end
    _G.AntiTrap = {
        Enabled = _G.AntiTrapEnabledSaved or false,
    }
    local items = {}
    local accentColor = Color3.fromRGB(132, 54, 217)

    local function isTrap(name)
        return name:lower():find("trap") ~= nil and name ~= "AntiStepBarrier"
    end

    local function createBarrier(part)
        if items[part] then return end
        local cFrame, vector
        if part:IsA("Model") then
            local ok
            ok, cFrame, vector = pcall(function()
                return part:GetBoundingBox()
            end)
            if not ok then cFrame = nil; vector = nil end
        elseif part:IsA("BasePart") then
            cFrame = part.CFrame
            vector = part.Size
        end
        if not cFrame then return end

        local barrier = Instance.new("Part")
        barrier.Name = "AntiStepBarrier"
        barrier.Anchored = true
        barrier.CanCollide = true
        barrier.CanTouch = false
        barrier.CanQuery = false
        barrier.Transparency = 0.6
        barrier.Material = Enum.Material.ForceField
        barrier.Color = accentColor
        barrier.Size = vector + Vector3.new(6, vector.Y, 6)
        barrier.CFrame = cFrame
        barrier.Parent = workspace

        local selBox = Instance.new("SelectionBox")
        selBox.Adornee = barrier
        selBox.LineThickness = 0.05
        selBox.Color3 = accentColor
        selBox.SurfaceColor3 = accentColor
        selBox.SurfaceTransparency = 1
        selBox.Transparency = 0.2
        selBox.Parent = barrier

        items[part] = barrier
    end

    local function removeBarrier(part)
        local entry = items[part]
        if entry then
            pcall(function() entry:Destroy() end)
            items[part] = nil
        end
    end

    local function scanAndBlock()
        if not _G.AntiTrap.Enabled then return end
        for _, child in ipairs(workspace:GetChildren()) do
            if isTrap(child.Name) and not items[child] then
                createBarrier(child)
            end
        end
    end

    workspace.ChildAdded:Connect(function(child)
        if not _G.AntiTrap.Enabled then return end
        task.delay(0.5, function()
            if isTrap(child.Name) and not items[child] then
                createBarrier(child)
            end
        end)
    end)

    workspace.ChildRemoved:Connect(function(child)
        if items[child] then
            removeBarrier(child)
        end
    end)

    local function cleanup()
        _G.AntiTrap.Enabled = false
        for k in pairs(items) do
            removeBarrier(k)
        end
    end

    _G.AntiTrap.Cleanup = cleanup
    _G.AntiTrap.Toggle = function()
        if _G.AntiTrap.Enabled then
            cleanup()
        else
            _G.AntiTrap.Enabled = true
            scanAndBlock()
        end
    end

    if _G.AntiTrap.Enabled then
        scanAndBlock()
    end
end)

task.spawn(function()
    task.wait(0.05)
    local Players = game:GetService("Players")
    local localPlayer = Players.LocalPlayer
    local plotTimerESPSaved = _G.PlotTimerESPSaved or false
    local timerGuis = {}
    local isOwnPlot = {}
    local Synchronizer = nil
    local timerColor = Color3.fromRGB(132, 54, 217)

    pcall(function()
        Synchronizer = require(game:GetService("ReplicatedStorage"):WaitForChild("Packages"):WaitForChild("Synchronizer"))
    end)

    local function isMyPlot(plotModel)
        local plotSign = plotModel:FindFirstChild("PlotSign")
        if plotSign then
            local surfaceGui = plotSign:FindFirstChildWhichIsA("SurfaceGui", true)
            if surfaceGui then
                local textLabel = surfaceGui:FindFirstChildWhichIsA("TextLabel", true)
                if textLabel and textLabel.Text then
                    local lower = textLabel.Text:lower()
                    if localPlayer.Name and lower:find(localPlayer.Name:lower(), 1, true) then
                        return true
                    end
                    if localPlayer.DisplayName and lower:find(localPlayer.DisplayName:lower(), 1, true) then
                        return true
                    end
                end
            end
        end
        return false
    end

    local function createTimerGui(plotModel)
        local friendPanel = plotModel:FindFirstChild("FriendPanel")
        local basePart = nil
        if friendPanel then
            basePart = friendPanel:FindFirstChildWhichIsA("BasePart") or friendPanel.PrimaryPart
        end
        basePart = basePart or plotModel:FindFirstChild("MainRoot")
        if not basePart then return nil end

        local existing = basePart:FindFirstChild("PlotTimerBG")
        if existing then existing:Destroy() end

        local bg = Instance.new("BillboardGui")
        bg.Name = "PlotTimerBG"
        bg.Size = UDim2.new(0, 250, 0, 50)
        bg.StudsOffset = Vector3.new(0, 5, 0)
        bg.AlwaysOnTop = true
        bg.MaxDistance = math.huge
        bg.Adornee = basePart
        bg.Parent = basePart

        local label = Instance.new("TextLabel")
        label.Name = "TimerLabel"
        label.Size = UDim2.fromScale(1, 1)
        label.Position = UDim2.fromScale(0, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = timerColor
        label.Font = Enum.Font.GothamBold
        label.TextSize = 28
        label.TextStrokeTransparency = 0.2
        label.TextStrokeColor3 = Color3.new(0, 0, 0)
        label.Text = "Loading..."
        label.Parent = bg

        return bg, isMyPlot(plotModel)
    end

    local function getPlotStatus(plotModel)
        local purchases = plotModel:FindFirstChild("Purchases")
        purchases = purchases and purchases:FindFirstChild("PlotBlock")
        purchases = purchases and purchases:FindFirstChild("Main")
        local billboardGui = purchases and purchases:FindFirstChild("BillboardGui")
        if not billboardGui then return "UNKNOWN", nil end

        local remaining = billboardGui:FindFirstChild("RemainingTime")
        local lockStudio = billboardGui:FindFirstChild("LockStudio")

        if remaining and remaining.Visible then
            local secs = tonumber((remaining.Text or ""):match("(%d+)%s*s"))
            if secs and secs > 0 then
                return "LOCKING", secs
            end
            return "LOCKED", 0
        end

        if purchases and purchases:IsA("BasePart")
            and purchases.Transparency <= 0.5
            and not (lockStudio and lockStudio.Visible) then
            return "LOCKED", nil
        end

        return "UNLOCKED", nil
    end

    local function updateTimerLabel(plotModel, gui)
        if not gui or not gui.Parent then return false end
        local timerLabel = gui:FindFirstChild("TimerLabel")
        if not timerLabel then return false end

        local status, secs = getPlotStatus(plotModel)

        if status == "LOCKING" and secs and secs > 0 then
            timerLabel.Text = secs .. "s"
            timerLabel.TextColor3 = timerColor
        else
            timerLabel.Text = "Unlocked"
            timerLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        end
        return true
    end

    local removePlotTimerESP = function()
        for _, gui in pairs(timerGuis) do
            if gui then
                pcall(function() gui:Destroy() end)
            end
        end
        timerGuis = {}
        isOwnPlot = {}
    end

    _G.setPlotTimerESPEnabled = function(enabled)
        plotTimerESPSaved = enabled
        _G.PlotTimerESPSaved = enabled
        if not enabled then
            removePlotTimerESP()
        end
    end

    local function scanPlots()
        if not plotTimerESPSaved then return end
        local plots = workspace:FindFirstChild("Plots")
        if not plots then return end

        local activePlots = {}
        for _, child in ipairs(plots:GetChildren()) do
            activePlots[child] = true
            if not timerGuis[child] or not timerGuis[child].Parent then
                local gui, mine = createTimerGui(child)
                timerGuis[child] = gui
                isOwnPlot[child] = mine
            end
            if timerGuis[child] then
                updateTimerLabel(child, timerGuis[child])
            end
        end

        for k, gui in pairs(timerGuis) do
            if not activePlots[k] then
                if gui then pcall(function() gui:Destroy() end) end
                timerGuis[k] = nil
                isOwnPlot[k] = nil
            end
        end
    end

    if plotTimerESPSaved then
        pcall(scanPlots)
    end

    Players.PlayerAdded:Connect(function()
        task.spawn(function()
            for j = 1, 16 do
                if plotTimerESPSaved then pcall(scanPlots) end
                task.wait(0.2)
            end
        end)
    end)
    Players.PlayerRemoving:Connect(function()
        task.spawn(function()
            for j = 1, 16 do
                if plotTimerESPSaved then pcall(scanPlots) end
                task.wait(0.2)
            end
        end)
    end)

    while true do
        if plotTimerESPSaved then
            pcall(scanPlots)
        end
        task.wait(1)
    end
end)

do
    local function playHighValueSound()
        if _G.HighValueSoundEnabled == false then return end
        pcall(function()
            local sound = Instance.new("Sound")
            sound.SoundId = "rbxassetid://" .. (_G.HighValueSoundId or "100173184074904")
            sound.Volume = 1
            sound.Parent = game:GetService("SoundService")
            sound:Play()
            game:GetService("Debris"):AddItem(sound, 10)
        end)
    end
    _G.StickyPlayHighValueSound = playHighValueSound

    local lastSoundTick = 0
    task.spawn(function()
        local Players = game:GetService("Players")
        while true do
            task.wait(2)
            if _G.HighValueSoundEnabled == false then continue end
            if tick() - lastSoundTick < 10 then continue end
            pcall(function()
                local lp = Players.LocalPlayer
                if not lp then return end
                local myPlot = nil
                for _, plot in ipairs(workspace:FindFirstChild("Plots"):GetChildren()) do
                    if plot:FindFirstChild("Owner") and plot.Owner.Value == lp then
                        myPlot = plot
                        break
                    end
                end
                local plots = workspace:FindFirstChild("Plots")
                if not plots then return end
                for _, plot in ipairs(plots:GetChildren()) do
                    if plot == myPlot then continue end
                    local podiums = plot:FindFirstChild("AnimalPodiums")
                    if not podiums then continue end
                    for _, podium in ipairs(podiums:GetChildren()) do
                        local base = podium:FindFirstChild("Base")
                        local spawnPart = base and base:FindFirstChild("Spawn")
                        local attach = spawnPart and spawnPart:FindFirstChild("PromptAttachment")
                        if not attach then continue end
                        for _, ch in ipairs(attach:GetChildren()) do
                            if ch:IsA("ProximityPrompt") and ch.Enabled then
                                local objText = ch.ObjectText or ""
                                if objText ~= "" then
                                    local minGen = tonumber(_G.HighValueMinGen) or 5000000
                                    if minGen > 0 then
                                        local genVal = nil
                                        pcall(function()
                                            local num, suf = objText:match("([%d%.]+)%s*(%a*)")
                                            local base = tonumber(num)
                                            if base then
                                                local suffixMult = {[""] = 1, k = 1e3, m = 1e6, b = 1e9, t = 1e12,
                                                    qa = 1e15, qi = 1e18, sx = 1e21, sp = 1e24, oc = 1e27}
                                                genVal = base * (suffixMult[string.lower(suf or "")] or 1)
                                            end
                                        end)
                                        if not genVal or genVal < minGen then break end
                                    end
                                    lastSoundTick = tick()
                                    playHighValueSound()
                                    return
                                end
                                break
                            end
                        end
                    end
                end
            end)
        end
    end)
end

do
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")

    local function waitForClone(timeout)
        timeout = timeout or 0.6
        local cloneName = tostring(Players.LocalPlayer.UserId) .. "_Clone"
        local t0 = tick()
        while tick() - t0 < timeout do
            local clone = workspace:FindFirstChild(cloneName)
            if clone then return clone end
            RunService.Heartbeat:Wait()
        end
        return workspace:FindFirstChild(cloneName)
    end

    _G._stickyFastClone = function()
        local lp = Players.LocalPlayer
        if not lp then return false end

        local character = lp.Character or lp.CharacterAdded:Wait()
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return false end

        local backpack = lp:FindFirstChildOfClass("Backpack")
        local quantumCloner = (backpack and backpack:FindFirstChild("Quantum Cloner"))
            or character:FindFirstChild("Quantum Cloner")
        if not quantumCloner then return false end

        if quantumCloner.Parent ~= character then
            pcall(function() humanoid:EquipTool(quantumCloner) end)
            local t0 = tick()
            while tick() - t0 < 0.2 do
                if quantumCloner.Parent == character then break end
                RunService.Heartbeat:Wait()
            end
        end

        local cloneName = tostring(lp.UserId) .. "_Clone"
        local stale = workspace:FindFirstChild(cloneName)
        if stale then pcall(function() stale:Destroy() end) end

        local placed = false
        local ok = pcall(function() quantumCloner:Activate() end)
        if ok then placed = true end

        if not ok and typeof(getconnections) == "function" then
            pcall(function()
                local conns = getconnections(quantumCloner.Activated)
                if conns and conns[1] and conns[1].Function then
                    conns[1].Function()
                    placed = true
                end
            end)
        end

        if placed then
            task.spawn(function()
                local cloneModel = waitForClone(0.6)
                if not cloneModel then return end

                local playerGui = lp:FindFirstChildOfClass("PlayerGui")
                if not playerGui then return end
                local toolsFrames = playerGui:FindFirstChild("ToolsFrames")
                local qcFrame = toolsFrames and toolsFrames:FindFirstChild("QuantumCloner")
                local btn = qcFrame and qcFrame:FindFirstChild("TeleportToClone")

                if not btn then
                    local t0 = tick()
                    while tick() - t0 < 0.3 do
                        RunService.Heartbeat:Wait()
                        toolsFrames = playerGui:FindFirstChild("ToolsFrames")
                        qcFrame = toolsFrames and toolsFrames:FindFirstChild("QuantumCloner")
                        btn = qcFrame and qcFrame:FindFirstChild("TeleportToClone")
                        if btn then break end
                    end
                end
                if not btn then return end

                local fired = false
                pcall(function() btn.Visible = true end)
                if typeof(firesignal) == "function" then
                    local fOk = pcall(firesignal, btn.MouseButton1Up)
                    if fOk then fired = true end
                end
                if not fired and typeof(getconnections) == "function" then
                    pcall(function()
                        local conns = getconnections(btn.MouseButton1Up)
                        if conns and conns[1] and conns[1].Function then
                            conns[1].Function()
                            fired = true
                        end
                    end)
                end
                if not fired and typeof(firesignal) == "function" then
                    pcall(firesignal, btn.MouseButton1Click)
                end
            end)
        end

        return placed
    end
end

print("auto steal ready baby sticky")
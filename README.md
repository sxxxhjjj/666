-- v085 汉化无后门完整版 | 基于原版 stbb.lua 完整修改
-- 修改：UI 全中文，内部存储/通信使用英文
local version = "Rework"
local ver = "v023.4-纯净版"

-- ====================== LOAD UI ======================
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

-- ====================== GameLoad ======================
repeat task.wait() until game:IsLoaded()

-- ====================== LoadingGui ======================
local p = game:GetService("Players").LocalPlayer
local pg = p:WaitForChild("PlayerGui")

local function waitLoadingGone()
    local gui = pg:FindFirstChild("LoadingGui")
    if gui then
        WindUI:Notify({ Title = "初始化", Content = "游戏加载中，请稍候...", Duration = 3, Icon = "download" })
        gui.AncestryChanged:Wait()
    end
end

waitLoadingGone()

WindUI:Notify({ Title = "初始化", Content = "加载完成，3秒后启动", Duration = 3, Icon = "shield-check" })
task.wait(3)

-- ====================== FPS UNLOCK ======================
local part = Instance.new("Part")
part.Size = Vector3.new(10, 1, 10)
part.Position = Vector3.new(-23.3435822, 61, 0.341766357)
part.Transparency = 1
part.Anchored = true
part.CanCollide = true
part.Material = Enum.Material.Neon
part.BrickColor = BrickColor.new("Bright blue")
part.Name = "DYHUB_WAITING_PART"
part.Parent = workspace

if setfpscap then
    setfpscap(1000000)
    WindUI:Notify({ Title = "服务", Content = "FPS 已解锁 | " .. ver, Duration = 3, Icon = "cpu" })
    warn("FPS Unlocked!")
else
    WindUI:Notify({ Title = "不支持", Content = "您的执行器不支持 setfpscap.", Duration = 3, Icon = "ban" })
end

-- ====================== CUSTOM CONFIG SYSTEM ======================
local HttpService = game:GetService("HttpService")
local ConfigFolder = "DYHUB_STBB_V0234"

local CustomConfig = {}
CustomConfig.__index = CustomConfig

function CustomConfig.new()
    local self = setmetatable({}, CustomConfig)
    self.ConfigData = {}
    self.ConfigPath = ConfigFolder .. "/config.json"
    if not isfolder(ConfigFolder) then makefolder(ConfigFolder) end
    self:Load()
    return self
end

function CustomConfig:Set(key, value) self.ConfigData[key] = value end

function CustomConfig:Get(key, default)
    if self.ConfigData[key] ~= nil then return self.ConfigData[key] end
    return default
end

function CustomConfig:Save()
    local success, err = pcall(function()
        writefile(self.ConfigPath, HttpService:JSONEncode(self.ConfigData))
    end)
    if success then warn("[DYHUB] Config saved!") else warn("[DYHUB] Save failed:", err) end
end

function CustomConfig:Load()
    if isfile(self.ConfigPath) then
        local success, result = pcall(function()
            return HttpService:JSONDecode(readfile(self.ConfigPath))
        end)
        if success and type(result) == "table" then
            self.ConfigData = result
            print("[DYHUB] Config loaded!")
        else
            warn("[DYHUB] Failed to load config, using defaults")
            self.ConfigData = {}
        end
    else
        print("[DYHUB] No config found, creating new one")
        self.ConfigData = {}
    end
end

function CustomConfig:AutoSave(interval)
    task.spawn(function()
        while true do
            task.wait(interval or 15)
            self:Save()
        end
    end)
end

local Config = CustomConfig.new()
Config:AutoSave(15)

-- ====================== WINDOW 2 ======================
local Players = game:GetService("Players")

-- 已删除 Premium 验证代码，直接设置为免费版
local userversion = "免费版"

-- ====================== WINDOW ======================
local Window = WindUI:CreateWindow({
    Title = "DYHUB",
    IconThemed = true,
    Icon = "rbxassetid://104487529937663",
    Author = "STBB | " .. userversion,
    Folder = "DYHUB",
    Size = UDim2.fromOffset(550, 380),
    Transparent = true,
    Theme = "Dark",
    BackgroundImageTransparency = 0.8,
    HasOutline = false,
    HideSearchBar = true,
    ScrollBarEnabled = true,
    User = { Enabled = true, Anonymous = false },
})

Window:Tag({ Title = version, Color = Color3.fromHex("#db7093") })

Window:EditOpenButton({
    Title = "DYHUB - 打开",
    Icon = "monitor",
    CornerRadius = UDim.new(0, 6),
    StrokeThickness = 2,
    Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)),
    Draggable = true
})

-- ====================== TABS ======================
local Info   = Window:Tab({ Title = "信息", Icon = "info" })
MainDivider  = Window:Divider()
local Main   = Window:Tab({ Title = "核心", Icon = "rocket" })
local Main4  = Window:Tab({ Title = "透视", Icon = "eye" })
local Main2  = Window:Tab({ Title = "玩家", Icon = "user" })
MainDivider1 = Window:Divider()
local Main5  = Window:Tab({ Title = "商店", Icon = "shopping-cart" })
local Main6  = Window:Tab({ Title = "收集", Icon = "hand" })
local Main7  = Window:Tab({ Title = "模式", Icon = "gamepad-2" })
MainDivider2 = Window:Divider()
local Main3  = Window:Tab({ Title = "设置", Icon = "settings" })
Window:SelectTab(1)

-- ======================== INFO ========================
Info:Section({ Title = "最近更新", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = "更新: 2026/05/16",
    Desc = "- [新增] 优先级系统\n- [新增] 恢复投票系统\n- [新增] 上帝模式\n- [新增] 解锁通行证\n- [修复] 怪物高度覆写\n- [修复] 透视核心\n- [优化] 删除地图",
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Info:Divider()
Info:Section({ Title = "DYHUB 信息", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({ Title = "纯净版", Desc = "无后门 | 中文界面 | 完整功能", Image = "rbxassetid://104487529937663", ImageSize = 30 })

-- ====================== SERVICES ======================
local TweenService       = game:GetService("TweenService")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService         = game:GetService("RunService")

-- ====================== PLAYER ======================
local LocalPlayer    = Players.LocalPlayer
local Client         = LocalPlayer
local Character      = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

-- ====================== GLOBAL TABLES ======================
GlobalTables = {
    redeemCodes = { "100MVisit2", "100MVisit1", "CamArmada", "CCTVBase", "ADelayedGameIsEventuallyGoodButRushedGameIsForeverBad" },
    -- 模式名称（中文显示），内部存储用英文映射
    ModeDisplay = { "普通模式", "模糊记忆", "极限模式", "困难模式", "疯狂模式", "噩梦模式", "首领连战", "暗黑维度", "地狱", "迷雾", "圣诞行动1", "僵尸行动1", "坚守模式", "入侵" },
    ModeInternal = {
        ["普通模式"] = "Normal Mode", ["模糊记忆"] = "Vague Memory", ["极限模式"] = "Extreme Mode",
        ["困难模式"] = "Hard Mode", ["疯狂模式"] = "Insane Mode", ["噩梦模式"] = "Nightmare Mode",
        ["首领连战"] = "Boss Rush", ["暗黑维度"] = "Dark Dimension", ["地狱"] = "Hell", ["迷雾"] = "Mist",
        ["圣诞行动1"] = "Christmas Act 1", ["僵尸行动1"] = "Zombie Act 1", ["坚守模式"] = "Holdout", ["入侵"] = "Invasion"
    },
    Votes = {
        "Normal","VeryHard","Hard","Insane","Nightmare","BossRush",
        "DarkDimension","Hell","ThunderStorm","Christmas","Zombie",
        "AstroV2","Astro","100MVisit"
    },
    -- 武器显示中文，内部存储英文
    WeaponDisplay = { "电击枪", "火焰喷射器", "鱼叉枪", "霰弹枪", "脉冲步枪", "鱼叉霰弹枪", "EPD", "小型激光枪" },
    WeaponInternal = {
        ["电击枪"] = "Stungun", ["火焰喷射器"] = "Flamethrower", ["鱼叉枪"] = "Harpoon Gun",
        ["霰弹枪"] = "Shot Gun", ["脉冲步枪"] = "Pulse Rifle", ["鱼叉霰弹枪"] = "Shot Harpoon Gun",
        ["EPD"] = "EPD", ["小型激光枪"] = "Small Laser Gun"
    },
    -- 道具显示中文，内部存储英文
    MiscDisplay = { "耳机", "泰坦呼叫", "特种泰坦呼叫", "扬声器呼叫", "手雷", "喷气背包", "透镜" },
    MiscInternal = {
        ["耳机"] = "HeadPhone", ["泰坦呼叫"] = "Titan-Request", ["特种泰坦呼叫"] = "SpecialTitan-Request",
        ["扬声器呼叫"] = "Speaker-Request", ["手雷"] = "Grenade", ["喷气背包"] = "Jetpack", ["透镜"] = "Lens"
    },
    Gamepasst = { "全部", "幸运加成", "稀有幸运加成", "传说幸运加成" },
    Gamepassts = {},
}

-- ====================== CONFIG VARIABLES ======================
local skillList          = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }
local skillDropdownValues = { "全部", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }

-- ====================== STATE VARIABLES ======================
local AutoFarmEnabled        = Config:Get("AutoFarmEnabled", false)
local FarmPosition           = Config:Get("FarmPosition", "Above")
local FarmMode               = Config:Get("FarmMode", "Tween")
local MiscOptions            = Config:Get("MiscOptions", {})
local AutoAttackEnabled      = false
local AutoSkillEnabled       = false
local AutoSkipHeliEnabled    = false
local BoostFPS_Active_dummy  = false
local AutoStartEnabled       = false
local AutoFillUpEnabled      = false
local SelectedSkills         = Config:Get("SelectedSkills", { "全部" })
local SafeModeEnabled        = false
local SafeValue              = Config:Get("SafeValue", 30)
local GodModeEnabled         = false
local GodModeValue           = Config:Get("GodModeValue", 30)
local GodModeTriggered       = false
local WaitingRespawn         = false
local IdlePosition           = CFrame.new(-23.3435822, 67, 0.341766357) * CFrame.Angles(math.rad(0), 0, 0)
local SkillDelay             = Config:Get("SkillDelay", 1)
local LoopDelay              = 0.5
local TweenSpeed             = 1
local HeightValue            = Config:Get("HeightValue", 3)
local NeedNoClip             = false
local LockActive             = false
local AutoStartConnection    = nil
local noBarrierConnection    = nil
local noBarrierActive        = Config:Get("NoBarrier", false)

-- ====================== NEW PRIORITY SYSTEM CONFIG ======================
local HighHPThreshold        = Config:Get("HighHPThreshold", 200)
local _currentTargetPriority = 0
local _interruptSignal       = false

local VirtualUser = game:GetService("VirtualUser")
local AntiAFK = Config:Get("AntiAfk", true)

-- 自动购买配置（存储英文名）
local AutoBuyWeaponEnabled   = Config:Get("AutoBuyWeaponEnabled", false)
local AutoBuyMiscEnabled     = Config:Get("AutoBuyMiscEnabled", false)
-- 兼容旧配置：若配置中是中文则转为英文
local rawWeapon = Config:Get("SelectedWeapon", "电击枪")
local rawMisc = Config:Get("SelectedMiscItem", "耳机")
local SelectedWeapon = GlobalTables.WeaponInternal[rawWeapon] or rawWeapon
local SelectedMiscItem = GlobalTables.MiscInternal[rawMisc] or rawMisc
-- 重新保存英文值
Config:Set("SelectedWeapon", SelectedWeapon)
Config:Set("SelectedMiscItem", SelectedMiscItem)

-- 自动模式配置（存储英文模式名）
local rawMode = Config:Get("AutoGameValue", "普通模式")
local AutoGameValue = GlobalTables.ModeInternal[rawMode] or rawMode
Config:Set("AutoGameValue", AutoGameValue)

-- ====================== FILL UP PART CONFIG ======================
local FILLUP_PART_PATH   = { "HelicopterShop", "ShopXDD", "PartForShop" }
local FILLUP_TARGET_POS  = Vector3.new(44.2756729, 26.3595276, -32.7318268)
local FILLUP_POS_THRESHOLD = 0.5
local FillUpRunning = false

local function GetFillUpPart()
    local obj = workspace
    for _, key in ipairs(FILLUP_PART_PATH) do
        obj = obj:FindFirstChild(key)
        if not obj then return nil end
    end
    return obj
end

local function IsFillUpPartReady()
    local p = GetFillUpPart()
    if not p then return false end
    return (p.CFrame.Position - FILLUP_TARGET_POS).Magnitude < FILLUP_POS_THRESHOLD
end

-- ====================== ALLY SYSTEM ======================
local AllyNames = {
    ["Heavy Soldier Toilet V2"]  = true,
    ["Quad Laser Toilet"]        = true,
    ["Strider Rocket Laser"]     = true,
    ["Helicopter Camera"]        = true,
    ["Heavy Soldier Toilet V1"]  = true,
    ["Rocket Heli v2"]           = true,
    ["Gunner Camera man"]        = true,
    ["Attack Helicopter"]        = true,
    ["Swat Mutant"]              = true,
    ["Huge DJ Toilet"]           = true,
}

local function IsAlly(mob)
    return AllyNames[mob.Name] ~= nil
end

-- ====================== TP SYSTEM ======================
function tp(pu79)
    pcall(function()
        local v80 = Client
        if v80 then v80 = Client.Character end
        if v80:FindFirstChild("Humanoid") and v80.Humanoid.Sit == true then v80.Humanoid.Sit = false end
        NeedNoClip = true
        local v81 = { Target = pu79.Target or print("目标错误"), Mod = pu79.Mod or CFrame.new(0, 0, 0) }
        v80:FindFirstChild("HumanoidRootPart").CFrame = v81.Target * v81.Mod
    end)
end

function Tp(p82)
    if Client.Character.Humanoid.Sit == true then Client.Character.Humanoid.Sit = false end
    local v83, v84, v85 = pairs(Client.Character:GetDescendants())
    while true do
        local v86
        v85, v86 = v83(v84, v85)
        if v85 == nil then break end
        if v86:IsA("BasePart") then v86.CanCollide = false end
    end
    if not Client.Character.HumanoidRootPart:FindFirstChild("BodyClip") then
        local v87 = Instance.new("BodyVelocity")
        v87.Parent = Client.Character.HumanoidRootPart
        v87.Name = "BodyClip"
        v87.Velocity = Vector3.new(0, 0, 0)
        v87.MaxForce = Vector3.new(5, math.huge, 5)
    end
    Client.Character.HumanoidRootPart.CFrame = p82
end

function tp1(p89)
    local v90 = game.Players.LocalPlayer
    if v90 and v90.Character and v90.Character:FindFirstChild("HumanoidRootPart") then
        v90.Character:FindFirstChild("HumanoidRootPart").CFrame = p89
    else
        warn("玩家角色或 HumanoidRootPart 未找到")
    end
end

-- ====================== UTILITY FUNCTIONS ======================
local function IsValidMob(obj)
    if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
        if Players:GetPlayerFromCharacter(obj) then return false end
        if IsAlly(obj) then return false end
        local humanoid = obj:FindFirstChild("Humanoid")
        if humanoid and humanoid.Health > 0 then return true end
    end
    return false
end

local function IsMobDead(mob)
    if not mob or not mob.Parent then return true end
    local humanoid = mob:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return true end
    return false
end

local function GetMobMaxHP(mob)
    local humanoid = mob and mob:FindFirstChild("Humanoid")
    if not humanoid then return 0 end
    return humanoid.MaxHealth or 0
end

-- ====================== MOB SELECTION ======================
local function GetNearestMob()
    local nearestMob, nearestDist = nil, math.huge
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) then
            local mobRoot = mob:FindFirstChild("HumanoidRootPart")
            if mobRoot then
                local d = (HumanoidRootPart.Position - mobRoot.Position).Magnitude
                if d < nearestDist then nearestDist = d; nearestMob = mob end
            end
        end
    end
    return nearestMob
end

local function GetHighestMob()
    local highestMob, highestY = nil, -math.huge
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    local myY = HumanoidRootPart and HumanoidRootPart.Position.Y or 0
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) then
            local mobRoot = mob:FindFirstChild("HumanoidRootPart")
            if mobRoot then
                local mobY = mobRoot.Position.Y
                if mobY > myY and mobY > highestY then highestY = mobY; highestMob = mob end
            end
        end
    end
    return highestMob
end

-- ============================================================
-- ====================== NEW PRIORITY SYSTEM =================
-- ============================================================

local function GetHelicopter()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if mob.Name:lower():find("helicopter") and IsValidMob(mob) then
            return mob
        end
    end
    return nil
end

local function GetGiantSTToilet()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    local giant = livingFolder:FindFirstChild("Giant ST toilet")
    if giant and IsValidMob(giant) then
        local lever = giant:FindFirstChild("lever")
        if lever then
            local prompt = lever:FindFirstChildOfClass("ProximityPrompt")
            if prompt then return giant, prompt end
        end
    end
    return nil, nil
end

local function GetHighHPMob()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    local bestMob, bestHP = nil, HighHPThreshold
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) then
            local hp = GetMobMaxHP(mob)
            if hp > bestHP then
                bestHP = hp
                bestMob = mob
            end
        end
    end
    return bestMob
end

local function GetPriorityMob()
    local giant, prompt = GetGiantSTToilet()
    if giant and prompt then return giant, "GiantST", prompt, 4 end
    local heli = GetHelicopter()
    if heli then return heli, "Helicopter", nil, 3 end
    local highHPMob = GetHighHPMob()
    if highHPMob then return highHPMob, "HighHP", nil, 2 end
    local nearMob = GetNearestMob()
    if nearMob then return nearMob, "NearestMob", nil, 1 end
    return nil, nil, nil, 0
end

local function CheckInterrupt(currentPriority)
    if currentPriority < 4 then
        local g, pr = GetGiantSTToilet()
        if g and pr then return true, 4 end
    end
    if currentPriority < 3 then
        if GetHelicopter() then return true, 3 end
    end
    if currentPriority < 2 then
        if GetHighHPMob() then return true, 2 end
    end
    return false, currentPriority
end

-- ============================================================
-- ====================== MOB VISUAL BOUNDS ===================
-- ============================================================

local function GetMobVisualBounds(mob)
    local minY, maxY = math.huge, -math.huge
    local centerX, centerZ, count = 0, 0, 0

    for _, part in ipairs(mob:GetDescendants()) do
        if part:IsA("BasePart") and part.Transparency < 0.9 and part.Size.Y > 0.1 then
            local pos = part.Position
            local hy  = part.Size.Y * 0.5
            if pos.Y - hy < minY then minY = pos.Y - hy end
            if pos.Y + hy > maxY then maxY = pos.Y + hy end
            centerX = centerX + pos.X
            centerZ = centerZ + pos.Z
            count   = count + 1
        end
    end

    if count == 0 then
        local hrp = mob:FindFirstChild("HumanoidRootPart")
        if hrp then
            return hrp.Position, hrp.Position.Y - 2, hrp.Position.Y + 2
        end
        return Vector3.new(0, 0, 0), 0, 4
    end

    local cx = centerX / count
    local cz = centerZ / count
    local cy = (minY + maxY) * 0.5
    return Vector3.new(cx, cy, cz), minY, maxY
end

-- ============================================================
-- ====================== MOB HEIGHT OVERRIDE =================
-- ============================================================

local PADDING_REDUCE_STEP    = Config:Get("PaddingReduceStep", 2)
local PADDING_SAFE_MIN       = Config:Get("PaddingSafeMin", -30)
local DMG_THRESHOLD          = Config:Get("DmgThreshold", 40)
local ANTI_CLIP_MARGIN       = Config:Get("AntiClipMargin", 3)
local PLAYER_HALF_HEIGHT     = 3

local MobHeightOverride   = {}
local MobConfirmedPadding = {}
local MobLastHealth       = {}
local MobCheckerCancelled = {}

local function GetAntiClipFloor(mob, position)
    local _, minY, maxY = GetMobVisualBounds(mob)
    local visualHeight = maxY - minY
    return -(visualHeight) + PLAYER_HALF_HEIGHT + ANTI_CLIP_MARGIN
end

local function GetEffectivePadding(mob)
    if MobConfirmedPadding[mob] ~= nil then return MobConfirmedPadding[mob] end
    if MobHeightOverride[mob] ~= nil then return MobHeightOverride[mob] end
    return HeightValue
end

local function ClampPaddingToAntiClip(mob, padding)
    local antiFloor = GetAntiClipFloor(mob, FarmPosition)
    local clamped = math.max(padding, antiFloor)
    clamped = math.max(clamped, PADDING_SAFE_MIN)
    return clamped
end

local function StartDamageChecker(mob)
    MobCheckerCancelled[mob] = false
    task.spawn(function()
        local humanoid = mob and mob:FindFirstChild("Humanoid")
        if not humanoid then return end
        if MobConfirmedPadding[mob] ~= nil then return end

        MobLastHealth[mob]     = humanoid.Health
        MobHeightOverride[mob] = ClampPaddingToAntiClip(mob, MobHeightOverride[mob] or HeightValue)

        local lastDamageTime = tick()
        local noDamageTimer  = 0
        local hitStreak      = 0
        local lastWasHit     = false
        local reducedOnce    = false

        while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do
            task.wait(0.3)
            if MobCheckerCancelled[mob] then break end
            if not mob or not mob.Parent or IsMobDead(mob) then break end
            humanoid = mob:FindFirstChild("Humanoid")
            if not humanoid then break end

            local currentHP = humanoid.Health
            local lastHP    = MobLastHealth[mob] or currentHP
            local dmgDealt  = lastHP - currentHP
            local gotHit    = dmgDealt > 0

            if gotHit then
                lastDamageTime = tick()
                noDamageTimer  = 0
                reducedOnce    = false
                if lastWasHit then hitStreak = hitStreak + 1 else hitStreak = 1 end
                lastWasHit = true
                local curPad = GetEffectivePadding(mob)
                if dmgDealt >= DMG_THRESHOLD and MobConfirmedPadding[mob] == nil then
                    if not MobCheckerCancelled[mob] then
                        MobConfirmedPadding[mob] = curPad
                        MobHeightOverride[mob]   = curPad
                    end
                    break
                end
                if hitStreak >= 2 and MobConfirmedPadding[mob] == nil then
                    if not MobCheckerCancelled[mob] then
                        MobConfirmedPadding[mob] = curPad
                        MobHeightOverride[mob]   = curPad
                    end
                    break
                end
            else
                lastWasHit    = false
                hitStreak     = 0
                noDamageTimer = tick() - lastDamageTime
            end

            if noDamageTimer >= 3 and not reducedOnce then
                reducedOnce = true
                local curPad = GetEffectivePadding(mob)
                local newPad = ClampPaddingToAntiClip(mob, curPad - PADDING_REDUCE_STEP)
                if newPad ~= curPad then MobHeightOverride[mob] = newPad end
            end

            if noDamageTimer >= 6 then
                lastDamageTime = tick()
                reducedOnce    = false
                local curPad   = GetEffectivePadding(mob)
                local newPad   = ClampPaddingToAntiClip(mob, curPad - PADDING_REDUCE_STEP)
                if newPad ~= curPad then MobHeightOverride[mob] = newPad end
            end

            MobLastHealth[mob] = currentHP
        end

        if not MobCheckerCancelled[mob] then
            MobHeightOverride[mob] = nil
            MobLastHealth[mob]     = nil
        end
    end)
end

local function ResetMobOverride(mob)
    MobCheckerCancelled[mob] = true
    MobHeightOverride[mob]   = nil
    MobConfirmedPadding[mob] = nil
    MobLastHealth[mob]       = nil
    task.delay(0.5, function()
        MobCheckerCancelled[mob] = nil
    end)
end

-- ============================================================
-- ====================== TARGET CFRAME =======================
-- ============================================================
local function GetTargetCFrame(mob, position)
    local mobRoot = mob:FindFirstChild("HumanoidRootPart")
    if not mobRoot then return nil end

    local padding = GetEffectivePadding(mob)
    local center, minY, maxY = GetMobVisualBounds(mob)

    if position == "Above" then
        local safeTargetY = math.max(maxY + padding, maxY + 0.5)
        local targetPos   = Vector3.new(center.X, safeTargetY, center.Z)
        local lookAtPos   = Vector3.new(center.X, maxY, center.Z)
        local lookCF      = CFrame.new(targetPos, lookAtPos)
        return lookCF * CFrame.Angles(math.rad(-10), 0, 0)

    elseif position == "Under" then
        local safeTargetY = math.min(minY - padding, minY - 0.5)
        local targetPos   = Vector3.new(center.X, safeTargetY, center.Z)
        local lookAtPos   = Vector3.new(center.X, minY, center.Z)
        local lookCF      = CFrame.new(targetPos, lookAtPos)
        return lookCF * CFrame.Angles(math.rad(10), 0, 0)
    end
end

local function TeleportToMob(mob)
    local cf = GetTargetCFrame(mob, FarmPosition)
    if not cf then return end
    if FarmMode == "Tween" then
        local tweenInfo = TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
        local tween = TweenService:Create(HumanoidRootPart, tweenInfo, { CFrame = cf })
        tween:Play()
        tween.Completed:Wait()
    elseif FarmMode == "tp" then
        tp({ Target = cf, Mod = CFrame.new(0, 0, 0) })
    elseif FarmMode == "Tp" then
        Tp(cf)
    elseif FarmMode == "tp1" then
        tp1(cf)
    end
end

local function LockToMob(mob)
    LockActive = true
    local connection
    connection = RunService.RenderStepped:Connect(function()
        if not AutoFarmEnabled or IsMobDead(mob) or not LockActive then
            connection:Disconnect()
            LockActive = false
            return
        end
        if not Character or not HumanoidRootPart then return end
        local cf = GetTargetCFrame(mob, FarmPosition)
        if cf then
            Character:PivotTo(cf)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

-- ====================== AUTO LOOPS ======================
local function StartAutoAttack()
    task.spawn(function()
        while AutoAttackEnabled and AutoFarmEnabled do
            local mob = GetPriorityMob()
            if mob and not WaitingRespawn then
                pcall(function() ReplicatedStorage.LMB:FireServer() end)
            end
            task.wait(0.05)
        end
    end)
end

local function StartAutoSkill()
    task.spawn(function()
        while AutoSkillEnabled and AutoFarmEnabled do
            local mob = GetPriorityMob()
            if mob and not WaitingRespawn then
                local keysToPress = {}
                if table.find(SelectedSkills, "全部") then
                    keysToPress = skillList
                else
                    keysToPress = SelectedSkills
                end
                for _, key in ipairs(keysToPress) do
                    if not AutoSkillEnabled or not AutoFarmEnabled then break end
                    local keyCode = Enum.KeyCode[key]
                    if keyCode then
                        pcall(function()
                            VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
                            task.wait(0.05)
                            VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
                        end)
                        task.wait(SkillDelay)
                    end
                end
            end
            task.wait(LoopDelay)
        end
    end)
end

local function TriggerAutoSkipHeli(state)
    pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(state) end)
end

local function HasHumanoid(obj)
    if obj:IsA("Model") then
        return obj:FindFirstChildOfClass("Humanoid") ~= nil
    end
    return false
end

local function IsLivingDescendant(obj)
    local current = obj
    while current and current ~= workspace do
        if current:IsA("Model") and current:FindFirstChildOfClass("Humanoid") then
            return true
        end
        current = current.Parent
    end
    return false
end

-- ====================== Delete Map SYSTEM ======================
local BoostFPS_OriginalData = {}
local BoostFPS_Active = false
local BoostFPS_RestoreConnection = nil
local BoostFPS_LightingData = {}

local function SaveAndBoostFPS()
    if BoostFPS_Active then return end
    BoostFPS_Active = true
    BoostFPS_OriginalData = {}
    BoostFPS_LightingData = {}

    local Lighting = game:GetService("Lighting")
    BoostFPS_LightingData = {
        Brightness        = Lighting.Brightness,
        GlobalShadows     = Lighting.GlobalShadows,
        FogEnd            = Lighting.FogEnd,
        FogStart          = Lighting.FogStart,
    }
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.Brightness    = 1
        Lighting.FogEnd        = 100000
        Lighting.FogStart      = 100000
    end)
    for _, effect in ipairs(Lighting:GetChildren()) do
        pcall(function()
            if effect:IsA("Atmosphere") or effect:IsA("BloomEffect") or
               effect:IsA("ColorCorrectionEffect") or effect:IsA("DepthOfFieldEffect") or
               effect:IsA("SunRaysEffect") or effect:IsA("Sky") then
                BoostFPS_LightingData["effect_" .. effect.Name] = { class = effect.ClassName, inst = effect }
                effect.Parent = nil
            end
        end)
    end

    pcall(function()
        for _, obj in ipairs(workspace:GetDescendants()) do
            if IsLivingDescendant(obj) then continue end
            if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") or obj:IsA("SpecialMesh") then
                if not IsLivingDescendant(obj) then
                    if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then
                        BoostFPS_OriginalData[obj] = {
                            Transparency = obj.Transparency,
                            CastShadow   = obj.CastShadow,
                            Material     = obj.Material,
                        }
                        obj.Transparency = 1
                        obj.CastShadow   = false
                        pcall(function() obj.Material = Enum.Material.SmoothPlastic end)
                    end
                end
            elseif obj:IsA("Decal") or obj:IsA("Texture") then
                BoostFPS_OriginalData[obj] = { Transparency = obj.Transparency }
                obj.Transparency = 1
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or
                   obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("SelectionBox") then
                BoostFPS_OriginalData[obj] = { Enabled = obj.Enabled }
                pcall(function() obj.Enabled = false end)
            elseif obj:IsA("SpecialMesh") then
                BoostFPS_OriginalData[obj] = { TextureId = obj.TextureId }
                obj.TextureId = ""
            end
        end
    end)

    BoostFPS_RestoreConnection = workspace.DescendantAdded:Connect(function(obj)
        if not BoostFPS_Active then return end
        if IsLivingDescendant(obj) then return end
        task.wait(0.05)
        pcall(function()
            if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then
                if not IsLivingDescendant(obj) then
                    obj.Transparency = 1
                    obj.CastShadow   = false
                end
            elseif obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = 1
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or
                   obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                pcall(function() obj.Enabled = false end)
            end
        end)
    end)

    print("[DYHUB] 删除地图: 开启")
end

local function RestoreBoostFPS()
    if not BoostFPS_Active then return end
    BoostFPS_Active = false

    if BoostFPS_RestoreConnection then
        BoostFPS_RestoreConnection:Disconnect()
        BoostFPS_RestoreConnection = nil
    end

    local Lighting = game:GetService("Lighting")
    pcall(function()
        if BoostFPS_LightingData.Brightness        ~= nil then Lighting.Brightness        = BoostFPS_LightingData.Brightness end
        if BoostFPS_LightingData.GlobalShadows     ~= nil then Lighting.GlobalShadows     = BoostFPS_LightingData.GlobalShadows end
        if BoostFPS_LightingData.FogEnd            ~= nil then Lighting.FogEnd            = BoostFPS_LightingData.FogEnd end
        if BoostFPS_LightingData.FogStart          ~= nil then Lighting.FogStart          = BoostFPS_LightingData.FogStart end
    end)
    for key, data in pairs(BoostFPS_LightingData) do
        if type(key) == "string" and key:sub(1, 7) == "effect_" then
            pcall(function()
                if data.inst then data.inst.Parent = Lighting end
            end)
        end
    end

    for obj, data in pairs(BoostFPS_OriginalData) do
        pcall(function()
            if not obj or not obj.Parent then return end
            if data.Transparency ~= nil and (obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") or obj:IsA("Decal") or obj:IsA("Texture")) then
                obj.Transparency = data.Transparency
            end
            if data.CastShadow ~= nil then obj.CastShadow = data.CastShadow end
            if data.Material   ~= nil then pcall(function() obj.Material = data.Material end) end
            if data.Enabled    ~= nil then pcall(function() obj.Enabled  = data.Enabled  end) end
            if data.TextureId  ~= nil then obj.TextureId = data.TextureId end
        end)
    end

    BoostFPS_OriginalData = {}
    BoostFPS_LightingData = {}
    print("[DYHUB] 删除地图: 关闭")
end

task.spawn(function()
    while true do
        task.wait(3)
        if BoostFPS_Active then
            pcall(function()
                for _, obj in ipairs(workspace:GetDescendants()) do
                    if IsLivingDescendant(obj) then continue end
                    if (obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation")) then
                        if not IsLivingDescendant(obj) then
                            if obj.Transparency < 0.99 and not BoostFPS_OriginalData[obj] then
                                obj.Transparency = 1
                                obj.CastShadow   = false
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- ====================== PLAYER HP HELPERS ======================
local function GetPlayerHPInfo()
    local humanoid = Character and Character:FindFirstChild("Humanoid")
    if not humanoid then return 100, 100 end
    return humanoid.Health, humanoid.MaxHealth
end

local function IsPlayerHPFull()
    local hp, maxHp = GetPlayerHPInfo()
    if maxHp <= 0 then return true end
    return hp >= maxHp
end

local function GetPlayerHealthPercent()
    local humanoid = Character and Character:FindFirstChild("Humanoid")
    if not humanoid then return 100 end
    if humanoid.MaxHealth <= 0 then return 100 end
    return (humanoid.Health / humanoid.MaxHealth) * 100
end

-- ====================== GOD MODE LOOP ======================
task.spawn(function()
    while true do
        task.wait(0.1)
        if GodModeEnabled then
            pcall(function()
                local char = LocalPlayer.Character
                if not char then return end
                local humanoid = char:FindFirstChild("Humanoid")
                if not humanoid then return end
                if humanoid.MaxHealth <= 0 then return end
                local hpPercent = (humanoid.Health / humanoid.MaxHealth) * 100
                if hpPercent < GodModeValue then
                    local head = char:FindFirstChild("Head")
                    if head then
                        head:Destroy()
                    else
                        humanoid.Health = 0
                    end
                end
            end)
        end
    end
end)

-- ====================== AUTO FILL UP ======================
local function DoFillUp()
    for i = 1, 2 do
        pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", "FillHP") end)
        if i < 2 then task.wait(0.3) end
    end
end

local function StartAutoFillUpLoop()
    if FillUpRunning then return end
    FillUpRunning = true
    task.spawn(function()
        while AutoFillUpEnabled and AutoFarmEnabled do
            if not IsPlayerHPFull() then
                if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end
                local waited = 0
                while not IsFillUpPartReady() and AutoFillUpEnabled do
                    waited = waited + 0.2
                    if waited >= 30 then break end
                    task.wait(0.2)
                end
                if IsFillUpPartReady() and AutoFillUpEnabled then DoFillUp(); task.wait(1) end
                if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end
            end
            task.wait(1)
        end
        FillUpRunning = false
    end)
end

-- ====================== BARRIER BYPASS ======================
local function startNoBarrier()
    if noBarrierConnection then return end
    noBarrierConnection = RunService.Heartbeat:Connect(function()
        pcall(function()
            local char = LocalPlayer.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            local pos = hrp.Position
            if math.abs(pos.X) > 1000 or math.abs(pos.Y) > 1000 or math.abs(pos.Z) > 1000 then
                hrp.CFrame = CFrame.new(Vector3.new(0, 50, 0))
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if humanoid then humanoid.Health = humanoid.MaxHealth end
            end
        end)
    end)
end

local function stopNoBarrier()
    if noBarrierConnection then
        noBarrierConnection:Disconnect()
        noBarrierConnection = nil
    end
end

-- ============================================================
-- ====================== AUTO VOTE MODE ======================
-- ============================================================

local AutoVoteEnabled       = Config:Get("AutoVoteEnabled", false)
-- AutoGameValue 已是英文
local AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false)
local AutoVoteValue         = Config:Get("AutoVoteValue", "Normal")

local _voteRespawnConn   = nil
local _voteIGRespawnConn = nil
local _syncRespawnConn   = nil

local function FireVote_Solo()
    if not AutoGameValue then return end
    pcall(function()
        ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue })
    end)
    print("[DYHUB] 自动投票已触发: " .. AutoGameValue)
end

local function FireGetReady()
    task.wait(2.5)
    pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("1", true) end)
end

local function FireVote_InGame()
    if not AutoVoteValue then return end
    pcall(function() ReplicatedStorage.Vote:FireServer(AutoVoteValue) end)
end

local function SetupSyncVoteAndStart()
    if _voteRespawnConn then _voteRespawnConn:Disconnect(); _voteRespawnConn = nil end
    if _syncRespawnConn then _syncRespawnConn:Disconnect(); _syncRespawnConn = nil end
    FireVote_Solo()
    task.spawn(function()
        task.wait(2.5)
        if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end
    end)
    _syncRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1.5)
        if AutoVoteEnabled and AutoStartEnabled then
            FireVote_Solo()
            task.spawn(function()
                task.wait(2.5)
                if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end
            end)
        end
    end)
end

local function SetupAutoVote_SoloOnly(enabled)
    if _voteRespawnConn then _voteRespawnConn:Disconnect(); _voteRespawnConn = nil end
    if not enabled then return end
    FireVote_Solo()
    _voteRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1.5)
        if AutoVoteEnabled and not AutoStartEnabled then FireVote_Solo() end
    end)
end

local function SetupAutoStartOnly(enabled)
    if AutoStartConnection then AutoStartConnection:Disconnect(); AutoStartConnection = nil end
    if not enabled then return end
    FireGetReady()
    AutoStartConnection = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1)
        if AutoStartEnabled and not AutoVoteEnabled then task.spawn(FireGetReady) end
    end)
end

local function RefreshVoteAndStartSetup()
    if _voteRespawnConn   then _voteRespawnConn:Disconnect();   _voteRespawnConn   = nil end
    if _syncRespawnConn   then _syncRespawnConn:Disconnect();   _syncRespawnConn   = nil end
    if AutoStartConnection then AutoStartConnection:Disconnect(); AutoStartConnection = nil end

    if AutoVoteEnabled and AutoStartEnabled then
        SetupSyncVoteAndStart()
    elseif AutoVoteEnabled and not AutoStartEnabled then
        SetupAutoVote_SoloOnly(true)
    elseif not AutoVoteEnabled and AutoStartEnabled then
        SetupAutoStartOnly(true)
    end
end

local function SetupAutoVote_InGame(enabled)
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect(); _voteIGRespawnConn = nil end
    if not enabled then return end
    FireVote_InGame()
    _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1.5)
        if AutoVoteinGameEnabled then FireVote_InGame() end
    end)
end

local function StartAutoStart()
    AutoStartEnabled = true
    RefreshVoteAndStartSetup()
end

local function StopAutoStart()
    AutoStartEnabled = false
    RefreshVoteAndStartSetup()
end

-- ====================== TELEPORT TO IDLE ======================
local function TeleportToIdle()
    LockActive = false
    task.wait(0.1)
    WaitingRespawn = true
    pcall(function()
        Character:PivotTo(IdlePosition)
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
    end)
end

-- ====================== PROXIMITY PROMPT HELPERS ======================
local function ActivateProximityPrompt(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
        if fireproximityprompt then fireproximityprompt(prompt) end
        prompt:InputHoldBegin()
        task.wait(0.05)
        prompt:InputHoldEnd()
    end)
end

local function ActivateAllFlushPrompts()
    pcall(function()
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Model") then
                local prompt = part:FindFirstChildOfClass("ProximityPrompt")
                if prompt then
                    local actionText = prompt.ActionText:lower()
                    if actionText:find("flush") or actionText:find("flash") or actionText:find("dragon") then
                        ActivateProximityPrompt(prompt)
                    end
                end
            end
        end
    end)
end

-- ============================================================
-- ====================== COLLECT SYSTEM ======================
-- ============================================================

local CollectItems = {
    "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter",
    "Astro Samples", "Weird Prism", "Key Card", "Zombie Core",
    "Flash Drives", "Presents",
}

local CollectGroupMap = {
    ["Astro Samples"] = {
        "Trooper Blast","Trooper Spinner","Specialist Blaster","Specialist Spinner",
        "Specialist Sword Arm","Strider Leg","Interceptor Wing","Interceptor Goggles",
        "Interceptor Spinner","Impactor Cannon","Impactor Laser","High Impactor Cannon",
        "High Impactor Laser","Destructor Laser","Destructor Blaster","Destructor Core",
        "Obliterator Blaster","Obliterator Spinner",
    },
    ["Presents"] = {
        "Gacha Capsule",
    },
}

local AutoCollectEnabled   = Config:Get("AutoCollectEnabled", false)
local SelectedCollectItems = Config:Get("SelectedCollectItems", {})
local CollectMode          = Config:Get("CollectMode", "Clean")

local KnownCollectItems = {}
local CollectRunning    = false

local function MatchesPattern(objectName, pattern)
    local objL, patL = objectName:lower(), pattern:lower()
    if objL == patL then return true end
    if #objL > #patL and objL:sub(1, #patL) == patL then
        local nc = objL:sub(#patL + 1, #patL + 1)
        if nc == " " or nc == "#" or nc == "_" or nc == "-" then return true end
    end
    if CollectGroupMap[pattern] then
        for _, gName in ipairs(CollectGroupMap[pattern]) do
            if objL == gName:lower() then return true end
        end
    end
    return false
end

local function IsCollectTarget(objectName)
    for _, pattern in ipairs(SelectedCollectItems) do
        if MatchesPattern(objectName, pattern) then return true end
    end
    return false
end

local function FindNewCollectItems()
    local found = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj and obj.Parent and IsCollectTarget(obj.Name) then
            if obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("BasePart") then
                if not KnownCollectItems[obj] then table.insert(found, obj) end
            end
        end
    end
    return found
end

local function GetItemRootPart(obj)
    if obj:IsA("Model") then return obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")
    elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then return obj end
    return nil
end

local function TweenToItem(itemRoot)
    if not itemRoot or not HumanoidRootPart then return end
    local targetPos = itemRoot.Position + Vector3.new(0, 3, 0)
    local targetCF  = CFrame.new(targetPos, itemRoot.Position)
    local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = targetCF })
    tween:Play()
    tween.Completed:Wait()
end

local function ActivateItemPrompts(obj)
    pcall(function()
        for _, child in ipairs(obj:GetDescendants()) do
            if child:IsA("ProximityPrompt") then
                child.HoldDuration = 0; child.MaxActivationDistance = 50
                if fireproximityprompt then fireproximityprompt(child) end
                child:InputHoldBegin(); task.wait(0.05); child:InputHoldEnd()
            end
        end
    end)
end

local function IsItemGone(obj) return not obj or not obj.Parent end

local function CollectSingleItem(obj)
    if IsItemGone(obj) then return end
    local itemRoot = GetItemRootPart(obj)
    if not itemRoot then return end
    TweenToItem(itemRoot)
    local lockConn
    lockConn = RunService.RenderStepped:Connect(function()
        if IsItemGone(obj) or not AutoCollectEnabled then lockConn:Disconnect(); return end
        if not itemRoot or not itemRoot.Parent then lockConn:Disconnect(); return end
        local targetCF = CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position)
        if Character and HumanoidRootPart then
            Character:PivotTo(targetCF)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)
    local timeout = 0
    repeat
        ActivateItemPrompts(obj); task.wait(0.1); timeout = timeout + 0.1
        if timeout > 10 then break end
    until IsItemGone(obj) or not AutoCollectEnabled
    lockConn:Disconnect()
    KnownCollectItems[obj] = true
end

local function AllMobsDead()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return true end
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) then return false end
    end
    return true
end

local function StartAutoCollectLoop()
    if CollectRunning then return end
    CollectRunning = true
    task.spawn(function()
        while AutoCollectEnabled do
            if #SelectedCollectItems > 0 then
                local itemsToCollect = FindNewCollectItems()
                if #itemsToCollect > 0 then
                    if CollectMode == "IDGF" then
                        LockActive = false; task.wait(0.1)
                        for _, obj in ipairs(itemsToCollect) do
                            if not AutoCollectEnabled then break end
                            if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end
                        end
                        if AutoFarmEnabled then TeleportToIdle(); WaitingRespawn = false end

                    elseif CollectMode == "Clean" then
                        local waitedClean = 0
                        while not AllMobsDead() and AutoCollectEnabled do
                            task.wait(0.5); waitedClean = waitedClean + 0.5
                            if waitedClean >= 120 then break end
                        end
                        if not AutoCollectEnabled then break end
                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end
                        LockActive = false; task.wait(0.1)
                        for _, obj in ipairs(FindNewCollectItems()) do
                            if not AutoCollectEnabled then break end
                            if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end
                        end
                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end
                        if not IsPlayerHPFull() and AutoFillUpEnabled then
                            local fw = 0
                            while not IsPlayerHPFull() and AutoFillUpEnabled and AutoCollectEnabled do
                                task.wait(0.5); fw = fw + 0.5; if fw >= 60 then break end
                            end
                        end
                        if AutoFarmEnabled then TeleportToIdle(); WaitingRespawn = false end
                    end
                else
                    for obj, _ in pairs(KnownCollectItems) do
                        if IsItemGone(obj) then KnownCollectItems[obj] = nil end
                    end
                end
            end
            task.wait(0.5)
        end
        CollectRunning = false
    end)
end

workspace.DescendantAdded:Connect(function(obj)
    if not AutoCollectEnabled or #SelectedCollectItems == 0 then return end
    if not IsCollectTarget(obj.Name) then return end
    if not (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("BasePart")) then return end
    print("[DYHUB] 收集: 新物品 " .. obj.Name)
end)

-- ============================================================
-- ====================== MAIN FARM LOOP ======================
-- ============================================================
local function StartFarmLoop()
    task.spawn(function()
        task.spawn(function()
            while AutoFarmEnabled do
                if WaitingRespawn and not LockActive then
                    pcall(function()
                        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = IdlePosition })
                        tween:Play(); tween.Completed:Wait()
                        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                    end)
                end
                task.wait(0.1)
            end
        end)

        while AutoFarmEnabled do
            if not Character or not Character.Parent then
                Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
                Client = LocalPlayer
            end

            local mob, mobType, extraData, priority = GetPriorityMob()

            if mob then
                WaitingRespawn = false
                _currentTargetPriority = priority

                if mobType == "GiantST" and extraData then
                    local cf = GetTargetCFrame(mob, FarmPosition)
                    if cf then
                        if FarmMode == "Tween" then
                            local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = cf })
                            tween:Play(); tween.Completed:Wait()
                        else tp1(cf) end
                    end
                    local giantLockConn
                    giantLockConn = RunService.RenderStepped:Connect(function()
                        if IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled then
                            giantLockConn:Disconnect(); return
                        end
                        local lockCF = GetTargetCFrame(mob, FarmPosition)
                        if lockCF and Character and HumanoidRootPart then
                            Character:PivotTo(lockCF)
                            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                        end
                    end)
                    repeat
                        task.wait(0.2)
                        ActivateProximityPrompt(extraData)
                        ActivateAllFlushPrompts()
                    until IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled
                    giantLockConn:Disconnect()

                else
                    if SafeModeEnabled and GetPlayerHealthPercent() < SafeValue then
                        local mobRoot = mob:FindFirstChild("HumanoidRootPart")
                        if mobRoot then
                            local safePos = mobRoot.Position + Vector3.new(0, 111, 0)
                            pcall(function()
                                Character:PivotTo(CFrame.new(safePos))
                                HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                                HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                            end)
                        end
                        task.wait(0.5)
                    else
                        StartDamageChecker(mob)
                        TeleportToMob(mob)

                        LockActive = true
                        local lockConn
                        lockConn = RunService.RenderStepped:Connect(function()
                            if not AutoFarmEnabled or IsMobDead(mob) or not LockActive then
                                lockConn:Disconnect(); LockActive = false; return
                            end
                            if not Character or not HumanoidRootPart then return end
                            local cf = GetTargetCFrame(mob, FarmPosition)
                            if cf then
                                Character:PivotTo(cf)
                                HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                                HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                            end
                        end)

                        repeat
                            task.wait(0.1)
                            local shouldInterrupt, newPriority = CheckInterrupt(priority)
                            if shouldInterrupt then
                                _interruptSignal = true
                                break
                            end
                        until IsMobDead(mob) or not AutoFarmEnabled

                        lockConn:Disconnect()
                        LockActive = false
                        _interruptSignal = false
                        ResetMobOverride(mob)
                    end
                end

            else
                _currentTargetPriority = 0
                TeleportToIdle()
                repeat task.wait(0.5) until GetPriorityMob() ~= nil or not AutoFarmEnabled
                WaitingRespawn = false
            end

            task.wait(0.1)
        end

        WaitingRespawn = false
        _currentTargetPriority = 0
    end)
end

-- ====================== MISC OPTIONS HANDLER ======================
local SyncFarmOnly = Config:Get("SyncFarmOnly", true)

local function HandleMiscOptions(selectedOptions)
    MiscOptions = selectedOptions
    local canRun = AutoFarmEnabled or not SyncFarmOnly

    local hasAutoAttack = table.find(selectedOptions, "自动攻击")
    if hasAutoAttack and not AutoAttackEnabled and canRun then
        AutoAttackEnabled = true; StartAutoAttack()
    elseif not hasAutoAttack then
        AutoAttackEnabled = false
    end

    local hasAutoSkill = table.find(selectedOptions, "自动技能")
    if hasAutoSkill and not AutoSkillEnabled and canRun then
        AutoSkillEnabled = true; StartAutoSkill()
    elseif not hasAutoSkill then
        AutoSkillEnabled = false
    end

    local hasAutoSkipHeli = table.find(selectedOptions, "自动跳过直升机")
    if hasAutoSkipHeli and not AutoSkipHeliEnabled and canRun then
        AutoSkipHeliEnabled = true; TriggerAutoSkipHeli(true)
    elseif not hasAutoSkipHeli and AutoSkipHeliEnabled then
        AutoSkipHeliEnabled = false; TriggerAutoSkipHeli(false)
    end

    local hasBoostFPS = table.find(selectedOptions, "删除地图")
    if hasBoostFPS and not BoostFPS_Active then
        SaveAndBoostFPS()
    elseif not hasBoostFPS and BoostFPS_Active then
        RestoreBoostFPS()
    end

    SafeModeEnabled = table.find(selectedOptions, "安全模式") ~= nil
    GodModeEnabled  = table.find(selectedOptions, "上帝模式") ~= nil

    local hasAutoStart = table.find(selectedOptions, "自动开局")
    if hasAutoStart and not AutoStartEnabled and canRun then
        StartAutoStart()
    elseif not hasAutoStart and AutoStartEnabled then
        StopAutoStart()
    end

    local hasAutoFillUp = table.find(selectedOptions, "自动补血")
    if hasAutoFillUp and not AutoFillUpEnabled then
        if canRun then AutoFillUpEnabled = true; StartAutoFillUpLoop() end
    elseif not hasAutoFillUp then
        AutoFillUpEnabled = false; FillUpRunning = false
    end

    Config:Set("MiscOptions", selectedOptions)
    Config:Save()
end

-- ====================== CHARACTER RESPAWN HANDLER ======================
LocalPlayer.CharacterAdded:Connect(function(char)
    Character        = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
    Client           = LocalPlayer
    MobHeightOverride   = {}
    MobConfirmedPadding = {}
    MobLastHealth       = {}
    task.wait(1)
    local cam = workspace.CurrentCamera
    cam.CameraSubject = HumanoidRootPart
    cam.CameraType    = Enum.CameraType.Custom
end)

-- ====================== UI: MAIN ======================
Main:Section({ Title = "自动挂机", Icon = "package" })

AutoFarmToggle = Main:Toggle({
    Title = "自动挂机",
    Desc = "根据优先级系统自动刷怪",
    Value = AutoFarmEnabled,
    Callback = function(state)
        AutoFarmEnabled = state
        if state then
            StartFarmLoop()
            HandleMiscOptions(MiscOptions)
            WindUI:Notify({ Title = "自动挂机", Content = "已开启", Duration = 2, Icon = "play" })
        else
            AutoAttackEnabled = false; AutoSkillEnabled = false
            AutoSkipHeliEnabled = false; AutoFillUpEnabled = false
            FillUpRunning = false; LockActive = false
            if AutoStartEnabled then StopAutoStart() end
            if SyncFarmOnly then
                TriggerAutoSkipHeli(false)
                WindUI:Notify({ Title = "自动挂机", Content = "已关闭（同步模式生效）", Duration = 3, Icon = "square" })
            else
                WindUI:Notify({ Title = "自动挂机", Content = "已关闭", Duration = 2, Icon = "square" })
            end
        end
        Config:Set("AutoFarmEnabled", state); Config:Save()
    end
})

Main:Section({ Title = "挂机设置", Icon = "settings" })

PositionDropdown = Main:Dropdown({
    Title = "站位位置",
    Values = { "Above", "Under" },
    Multi = false,
    Value = FarmPosition,
    Callback = function(value) FarmPosition = value; Config:Set("FarmPosition", value); Config:Save() end
})

ModeDropdown = Main:Dropdown({
    Title = "移动模式",
    Values = { "Tween" },
    Multi = false,
    Value = FarmMode,
    Callback = function(value) FarmMode = value; Config:Set("FarmMode", value); Config:Save() end
})

MiscDropdown = Main:Dropdown({
    Title = "辅助功能",
    Values = { "自动攻击", "自动技能", "自动开局", "自动跳过直升机", "自动补血", "安全模式", "上帝模式", "删除地图" },
    Multi = true,
    Value = MiscOptions,
    Callback = function(values)
        MiscOptions = values
        if not AutoFarmEnabled and SyncFarmOnly then
            local hasFeatures = #values > 0
            local onlyGodOrBoost = true
            for _, v in ipairs(values) do
                if v ~= "上帝模式" and v ~= "删除地图" then
                    onlyGodOrBoost = false; break
                end
            end
            if hasFeatures and not onlyGodOrBoost then
                WindUI:Notify({
                    Title = "辅助功能",
                    Content = "请先开启自动挂机（同步模式已开启）",
                    Duration = 3, Icon = "triangle-alert"
                })
            end
        end
        HandleMiscOptions(values)
    end
})

Main:Toggle({
    Title = "同步挂机模式",
    Desc = "开启后，辅助功能需要自动挂机开启才能生效",
    Value = SyncFarmOnly,
    Callback = function(state)
        SyncFarmOnly = state
        Config:Set("SyncFarmOnly", state)
        Config:Save()
        if state then
            WindUI:Notify({ Title = "同步挂机模式", Content = "开启：辅助功能需自动挂机开启", Duration = 3, Icon = "link" })
        else
            WindUI:Notify({ Title = "同步挂机模式", Content = "关闭：辅助功能可独立运行", Duration = 3, Icon = "unlink" })
            HandleMiscOptions(MiscOptions)
        end
    end
})

Main:Section({ Title = "通用设置", Icon = "zap" })

SkillDropdown = Main:Dropdown({
    Title = "自动技能按键",
    Values = skillDropdownValues,
    Multi = true,
    Value = SelectedSkills,
    Callback = function(values) SelectedSkills = values; Config:Set("SelectedSkills", values); Config:Save() end
})

SkillDelaySlider = Main:Slider({
    Title = "技能延迟（秒）",
    Value = { Min = 1, Max = 30, Default = SkillDelay },
    Step = 1,
    Callback = function(value) SkillDelay = value; Config:Set("SkillDelay", value); Config:Save() end
})

FarmHeightSlider = Main:Slider({
    Title = "挂机高度偏移（+Y）",
    Value = { Min = -30, Max = 30, Default = HeightValue },
    Step = 1,
    Callback = function(value)
        HeightValue = value; Config:Set("HeightValue", value); Config:Save()
        for mob, _ in pairs(MobHeightOverride) do
            if MobConfirmedPadding[mob] == nil then MobHeightOverride[mob] = nil end
        end
    end
})

Main:Slider({
    Title = "安全模式血量（%）",
    Value = { Min = 1, Max = 100, Default = SafeValue },
    Step = 1,
    Callback = function(value) SafeValue = value; Config:Set("SafeValue", value); Config:Save() end
})

Main:Slider({
    Title = "上帝模式触发血量（%）",
    Value = { Min = 1, Max = 99, Default = GodModeValue },
    Step = 1,
    Callback = function(value)
        GodModeValue = value
        Config:Set("GodModeValue", value)
        Config:Save()
    end
})

-- ====================== UI: PRIORITY SETTINGS ======================
Main:Section({ Title = "优先级设置", Icon = "list-ordered" })

Main:Paragraph({
    Title = "优先级顺序",
    Desc = "GiantST → 直升机 → 高血量精英 → 最近怪物，高级怪物出现时立即切换",
    Image = "rbxassetid://104487529937663",
    ImageSize = 26,
})

Main:Slider({
    Title = "高血量阈值（最大生命值）",
    Value = { Min = 1, Max = 100000, Default = HighHPThreshold },
    Step = 100,
    Callback = function(value)
        HighHPThreshold = value
        Config:Set("HighHPThreshold", value)
        Config:Save()
        print("[DYHUB] 高血量阈值已设置为 " .. value)
    end
})

-- ====================== UI: OVERRIDE SETTINGS ======================
Main:Section({ Title = "覆写设置", Icon = "ruler" })

PaddingReduceInput = Main:Input({
    Title = "设置覆写递减步长",
    Default = tostring(PADDING_REDUCE_STEP),
    Placeholder = "默认: 2",
    Callback = function(text)
        local num = tonumber(text)
        if num then PADDING_REDUCE_STEP = num; Config:Set("PaddingReduceStep", num); Config:Save()
        else warn("输入的数字无效") end
    end
})

PaddingSafeInput = Main:Input({
    Title = "设置最小安全高度（全局下限）",
    Default = tostring(PADDING_SAFE_MIN),
    Placeholder = "默认: -30",
    Callback = function(text)
        local num = tonumber(text)
        if num then PADDING_SAFE_MIN = num; Config:Set("PaddingSafeMin", num); Config:Save()
        else warn("输入的数字无效") end
    end
})

Main:Slider({
    Title = "防卡墙边距（格）",
    Value = { Min = 0, Max = 10, Default = ANTI_CLIP_MARGIN },
    Step = 1,
    Callback = function(value)
        ANTI_CLIP_MARGIN = value; Config:Set("AntiClipMargin", value); Config:Save()
    end
})

Main:Slider({
    Title = "伤害阈值（确认锁定）",
    Value = { Min = 1, Max = 500, Default = DMG_THRESHOLD },
    Step = 1,
    Callback = function(value)
        DMG_THRESHOLD = value; Config:Set("DmgThreshold", value); Config:Save()
    end
})

Main:Button({
    Title = "重置所有已确认位置",
    Desc = "清除所有已保存的怪物高度位置",
    Callback = function()
        MobConfirmedPadding = {}
        MobHeightOverride   = {}
        WindUI:Notify({ Title = "覆写重置", Content = "所有已确认位置已清除", Duration = 2, Icon = "refresh-cw" })
    end
})

Main:Section({ Title = "冲水设置", Icon = "toilet" })

local Flushaura      = Config:Get("flushaura", false)
local FlushAuraValue = Config:Get("FlushAuraValue", 5)

Main:Slider({
    Title = "冲水范围（格）",
    Value = { Min = 1, Max = 15, Default = FlushAuraValue },
    Step = 1,
    Callback = function(value) FlushAuraValue = value; Config:Set("FlushAuraValue", value); Config:Save() end
})

Main:Toggle({
    Title = "冲水光环",
    Desc = "自动触发范围内的冲水提示",
    Value = Flushaura,
    Callback = function(enabled)
        Flushaura = enabled; Config:Set("flushaura", enabled); Config:Save()
        if enabled then
            task.spawn(function()
                while Flushaura do
                    pcall(function()
                        local char = game.Players.LocalPlayer.Character
                        if not char then return end
                        local root = char:FindFirstChild("HumanoidRootPart")
                        if not root then return end
                        for _, prompt in pairs(workspace:GetDescendants()) do
                            if prompt:IsA("ProximityPrompt") then
                                local at = prompt.ActionText
                                if at == "Flush" or at == "Dragon Flash" or at == "flush" or at == "Flash" then
                                    local part = prompt.Parent
                                    if part and part:IsA("BasePart") then
                                        if (root.Position - part.Position).Magnitude <= FlushAuraValue then
                                            prompt.HoldDuration = 0; prompt.MaxActivationDistance = FlushAuraValue
                                            if fireproximityprompt then fireproximityprompt(prompt)
                                            else prompt:InputHoldBegin(); task.wait(); prompt:InputHoldEnd() end
                                        end
                                    end
                                end
                            end
                        end
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end
})

-- ============================================================
-- ====================== ESP SYSTEM =========================
-- ============================================================

local ESP = {
    Enabled       = Config:Get("EspEnabled", false),
    MobEnabled    = Config:Get("EspMobEnabled", true),
    PlayerEnabled = Config:Get("EspPlayerEnabled", true),
    ItemEnabled   = Config:Get("EspItemEnabled", true),
    Settings      = Config:Get("EspSettings", { "高亮", "距离", "血量", "名称" }),
    SelectedItems = Config:Get("EspSelectedItems", {}),
    MaxDistance   = 1500,
    _mobHighlights    = {},
    _playerHighlights = {},
    _itemHighlights   = {},
    ItemList = {
        "Clock Spider","X-18 Core","Green Energy Core","Weird Transmitter",
        "Presents","Weird Prism","Key Card","Zombie Core","Flash Drives","Astro Samples",
    },
}

local function IsESPItemTarget(objectName, selectedList)
    for _, pattern in ipairs(selectedList) do
        if objectName:lower() == pattern:lower() then return true end
        if #objectName > #pattern then
            if objectName:lower():sub(1, #pattern) == pattern:lower() then
                local nc = objectName:lower():sub(#pattern + 1, #pattern + 1)
                if nc == " " or nc == "#" or nc == "_" or nc == "-" then return true end
            end
        end
        if CollectGroupMap[pattern] then
            for _, gName in ipairs(CollectGroupMap[pattern]) do
                if objectName:lower() == gName:lower() then return true end
            end
        end
    end
    return false
end

local function CreateESPLabel(parent, labelText)
    local existing = parent:FindFirstChild("DYHUB_ESP_LABEL")
    if existing then existing:Destroy() end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "DYHUB_ESP_LABEL"; billboard.Size = UDim2.new(0, 120, 0, 40)
    billboard.StudsOffset = Vector3.new(0, 3, 0); billboard.AlwaysOnTop = true
    billboard.ResetOnSpawn = false; billboard.Adornee = parent; billboard.Parent = parent
    local frame = Instance.new("Frame"); frame.BackgroundTransparency = 1
    frame.Size = UDim2.fromScale(1, 1); frame.Parent = billboard
    local label = Instance.new("TextLabel"); label.BackgroundTransparency = 1
    label.Size = UDim2.fromScale(1, 1); label.Font = Enum.Font.GothamBold
    label.TextSize = 11; label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextStrokeTransparency = 0.4; label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.Text = labelText; label.Parent = frame
    return billboard, label
end

local function CreateHighlight(model, outlineColor, fillColor, fillTransparency)
    local existing = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT")
    if existing then existing:Destroy() end
    local hl = Instance.new("Highlight")
    hl.Name = "DYHUB_ESP_HIGHLIGHT"; hl.OutlineColor = outlineColor
    hl.FillColor = fillColor; hl.FillTransparency = fillTransparency or 0.9
    hl.OutlineTransparency = 0; hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Adornee = model; hl.Parent = model
    return hl
end

local function RemoveESP(model)
    pcall(function()
        local hl = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT"); if hl then hl:Destroy() end
        local hb = model:FindFirstChild("DYHUB_ESP_LABEL"); if hb then hb:Destroy() end
        local hrp = model:FindFirstChild("HumanoidRootPart")
        if hrp then local lb = hrp:FindFirstChild("DYHUB_ESP_LABEL"); if lb then lb:Destroy() end end
    end)
end

local function IsInRange(targetPart)
    if not targetPart or not HumanoidRootPart then return false end
    return (HumanoidRootPart.Position - targetPart.Position).Magnitude <= ESP.MaxDistance
end

local function BuildLabelText(model, showName, showHealth, showDistance)
    local parts = {}
    if showName then table.insert(parts, model.Name) end
    if showHealth then
        local humanoid = model:FindFirstChild("Humanoid")
        if humanoid then table.insert(parts, "❤ " .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth)) end
    end
    if showDistance then
        local hrp = model:FindFirstChild("HumanoidRootPart")
        if hrp and HumanoidRootPart then table.insert(parts, "📏 " .. math.floor((HumanoidRootPart.Position - hrp.Position).Magnitude) .. "m") end
    end
    return table.concat(parts, "\n")
end

local function BuildItemLabelText(obj, showName, showDistance)
    local parts = {}
    if showName then table.insert(parts, obj.Name) end
    if showDistance then
        local root = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or (obj:IsA("BasePart") and obj or nil)
        if root and HumanoidRootPart then table.insert(parts, "📏 " .. math.floor((HumanoidRootPart.Position - root.Position).Magnitude) .. "m") end
    end
    return table.concat(parts, "\n")
end

local function GetESPSettings()
    local s = ESP.Settings
    return {
        highlight = table.find(s, "高亮") ~= nil,
        distance  = table.find(s, "距离") ~= nil,
        health    = table.find(s, "血量") ~= nil,
        name      = table.find(s, "名称") ~= nil,
    }
end

local function ApplyMobESP(mob)
    if not mob or not mob.Parent then return end
    local hrp = mob:FindFirstChild("HumanoidRootPart"); if not hrp then return end
    local settings = GetESPSettings()
    if settings.highlight then CreateHighlight(mob, Color3.fromRGB(255, 50, 50), Color3.fromRGB(255, 255, 255), 0.9) end
    if settings.name or settings.health or settings.distance then
        local _, label = CreateESPLabel(hrp, "")
        task.spawn(function()
            while mob and mob.Parent and ESP.Enabled and ESP.MobEnabled do
                local humanoid = mob:FindFirstChild("Humanoid")
                if not humanoid or humanoid.Health <= 0 then break end
                if not IsInRange(hrp) then label.Visible = false; task.wait(0.5)
                else label.Visible = true; label.Text = BuildLabelText(mob, settings.name, settings.health, settings.distance); task.wait(0.15) end
            end
            RemoveESP(mob); ESP._mobHighlights[mob] = nil
        end)
    end
    ESP._mobHighlights[mob] = true
end

local function ScanMobs()
    local livingFolder = workspace:FindFirstChild("Living"); if not livingFolder then return end
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) and not ESP._mobHighlights[mob] then
            local hrp = mob:FindFirstChild("HumanoidRootPart")
            if hrp and IsInRange(hrp) then ApplyMobESP(mob) end
        end
    end
end

local function ApplyPlayerESP(playerChar)
    if not playerChar or not playerChar.Parent then return end
    local hrp = playerChar:FindFirstChild("HumanoidRootPart"); if not hrp then return end
    if playerChar == LocalPlayer.Character then return end
    local settings = GetESPSettings()
    if settings.highlight then CreateHighlight(playerChar, Color3.fromRGB(50, 255, 50), Color3.fromRGB(255, 255, 255), 0.9) end
    if settings.name or settings.health or settings.distance then
        local _, label = CreateESPLabel(hrp, "")
        task.spawn(function()
            while playerChar and playerChar.Parent and ESP.Enabled and ESP.PlayerEnabled do
                local humanoid = playerChar:FindFirstChild("Humanoid")
                if not humanoid or humanoid.Health <= 0 then break end
                if not IsInRange(hrp) then label.Visible = false; task.wait(0.5)
                else label.Visible = true; label.Text = BuildLabelText(playerChar, settings.name, settings.health, settings.distance); task.wait(0.15) end
            end
            RemoveESP(playerChar); ESP._playerHighlights[playerChar] = nil
        end)
    end
    ESP._playerHighlights[playerChar] = true
end

local function ScanPlayers()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            if not ESP._playerHighlights[char] then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp and IsInRange(hrp) then ApplyPlayerESP(char) end
            end
        end
    end
end

local function GetItemRoot(obj)
    if obj:IsA("Model") then return obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")
    elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then return obj end
    return nil
end

local function ApplyItemESP(obj)
    if not obj or not obj.Parent then return end
    local root = GetItemRoot(obj); if not root then return end
    local settings = GetESPSettings()
    if settings.highlight then CreateHighlight(obj, Color3.fromRGB(255, 215, 0), Color3.fromRGB(255, 255, 255), 0.9) end
    if settings.name or settings.distance then
        local _, label = CreateESPLabel(root, "")
        task.spawn(function()
            while obj and obj.Parent and ESP.Enabled and ESP.ItemEnabled do
                local currentRoot = GetItemRoot(obj); if not currentRoot then break end
                if not IsInRange(currentRoot) then label.Visible = false; task.wait(0.5)
                else label.Visible = true; label.Text = BuildItemLabelText(obj, settings.name, settings.distance); task.wait(0.25) end
            end
            RemoveESP(obj); ESP._itemHighlights[obj] = nil
        end)
    end
    ESP._itemHighlights[obj] = true
end

local function ScanItems()
    if #ESP.SelectedItems == 0 then return end
    for _, obj in ipairs(workspace:GetDescendants()) do
        if not ESP._itemHighlights[obj] and IsESPItemTarget(obj.Name, ESP.SelectedItems) then
            local root = GetItemRoot(obj)
            if root and IsInRange(root) then ApplyItemESP(obj) end
        end
    end
end

local function ClearAllESP()
    for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end
    ESP._mobHighlights = {}
    for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end
    ESP._playerHighlights = {}
    for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end
    ESP._itemHighlights = {}
end

local ESPConnection = nil

local function StartESPLoop()
    if ESPConnection then ESPConnection:Disconnect(); ESPConnection = nil end
    local tickCounter = 0
    ESPConnection = RunService.Heartbeat:Connect(function()
        tickCounter = tickCounter + 1
        if tickCounter % 30 == 0 and ESP.Enabled and ESP.MobEnabled    then pcall(ScanMobs) end
        if tickCounter % 47 == 0 and ESP.Enabled and ESP.PlayerEnabled then pcall(ScanPlayers) end
        if tickCounter % 61 == 0 and ESP.Enabled and ESP.ItemEnabled   then pcall(ScanItems) end
        if tickCounter >= 3660 then tickCounter = 0 end
    end)
end

local function StopESPLoop()
    if ESPConnection then ESPConnection:Disconnect(); ESPConnection = nil end
    ClearAllESP()
end

workspace.DescendantAdded:Connect(function(obj)
    if not ESP.Enabled or not ESP.ItemEnabled or #ESP.SelectedItems == 0 then return end
    task.wait(0.1)
    if IsESPItemTarget(obj.Name, ESP.SelectedItems) and not ESP._itemHighlights[obj] then
        local root = GetItemRoot(obj)
        if root and IsInRange(root) then ApplyItemESP(obj) end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        if not ESP.Enabled or not ESP.PlayerEnabled then return end
        task.wait(1)
        if not ESP._playerHighlights[char] then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp and IsInRange(hrp) then ApplyPlayerESP(char) end
        end
    end)
end)

local function WatchLivingFolder()
    local living = workspace:FindFirstChild("Living")
    if living then
        living.ChildAdded:Connect(function(obj)
            if not ESP.Enabled or not ESP.MobEnabled then return end
            task.wait(0.2)
            if IsValidMob(obj) and not ESP._mobHighlights[obj] then
                local hrp = obj:FindFirstChild("HumanoidRootPart")
                if hrp and IsInRange(hrp) then ApplyMobESP(obj) end
            end
        end)
    end
end

task.spawn(function()
    if not workspace:FindFirstChild("Living") then
        workspace.ChildAdded:Connect(function(child)
            if child.Name == "Living" then WatchLivingFolder() end
        end)
    else
        WatchLivingFolder()
    end
end)

-- ====================== UI: ESP TAB ======================
Main4:Section({ Title = "透视视觉", Icon = "eye" })

EspEnableToggle = Main4:Toggle({
    Title = "启用透视", Value = ESP.Enabled,
    Desc = "启用所有透视效果",
    Callback = function(state)
        ESP.Enabled = state; Config:Set("EspEnabled", state); Config:Save()
        if state then StartESPLoop() else StopESPLoop() end
    end
})

EspMobToggle = Main4:Toggle({
    Title = "怪物透视", Value = ESP.MobEnabled,
    Desc = "在怪物上方显示高亮和标签",
    Callback = function(state)
        ESP.MobEnabled = state; Config:Set("EspMobEnabled", state); Config:Save()
        if not state then for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end; ESP._mobHighlights = {} end
    end
})

EspPlayerToggle = Main4:Toggle({
    Title = "玩家透视", Value = ESP.PlayerEnabled,
    Desc = "在其他玩家上方显示高亮和标签",
    Callback = function(state)
        ESP.PlayerEnabled = state; Config:Set("EspPlayerEnabled", state); Config:Save()
        if not state then for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end; ESP._playerHighlights = {} end
    end
})

EspItemToggle = Main4:Toggle({
    Title = "物品透视", Value = ESP.ItemEnabled,
    Desc = "在可收集物品上显示高亮和标签",
    Callback = function(state)
        ESP.ItemEnabled = state; Config:Set("EspItemEnabled", state); Config:Save()
        if not state then for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end; ESP._itemHighlights = {} end
    end
})

Main4:Section({ Title = "透视设置", Icon = "settings" })

EspSettingsDropdown = Main4:Dropdown({
    Title = "透视选项", Multi = true,
    Values = { "高亮", "距离", "血量", "名称" },
    Value = ESP.Settings,
    Callback = function(value)
        ESP.Settings = value or {}; Config:Set("EspSettings", value); Config:Save()
        if ESP.Enabled then ClearAllESP() end
    end,
})

EspItemDropdown = Main4:Dropdown({
    Title = "透视物品", Multi = true,
    Values = ESP.ItemList,
    Value = ESP.SelectedItems,
    Callback = function(value)
        ESP.SelectedItems = value or {}; Config:Set("EspSelectedItems", value); Config:Save()
        for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end
        ESP._itemHighlights = {}
        if ESP.Enabled and ESP.ItemEnabled then pcall(ScanItems) end
    end,
})

-- ====================== UI: PLAYER TAB ======================
Main2:Section({ Title = "本地玩家", Icon = "user" })

local WSValue = Config:Get("WSValue", 16)
local JPValue = Config:Get("JPValue", 50)
local NoClip  = Config:Get("NoClip", false)

local function updatePlayerStats()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = WSValue
        LocalPlayer.Character.Humanoid.JumpPower = JPValue
    end
end

RunService.Stepped:Connect(function()
    if NoClip and LocalPlayer.Character then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    updatePlayerStats()
end)

Main2:Slider({
    Title = "移动速度",
    Value = { Min = 1, Max = 200, Default = WSValue },
    Step = 1,
    Callback = function(value) WSValue = value; Config:Set("WSValue", value); Config:Save(); updatePlayerStats() end
})

Main2:Slider({
    Title = "跳跃高度",
    Value = { Min = 1, Max = 500, Default = JPValue },
    Step = 1,
    Callback = function(value) JPValue = value; Config:Set("JPValue", value); Config:Save(); updatePlayerStats() end
})

nocliptoggle = Main2:Toggle({
    Title = "穿墙模式", Value = NoClip,
    Desc = "允许角色穿过墙壁和部件",
    Callback = function(state) NoClip = state; Config:Set("NoClip", state); Config:Save() end
})

Main2:Section({ Title = "兑换码", Icon = "bird" })

local SelectedCodes = Config:Get("SelectedCodes", {})

CodeDropdown = Main2:Dropdown({
    Title = "选择兑换码", Multi = true,
    Values = GlobalTables.redeemCodes, Value = SelectedCodes,
    Callback = function(value) SelectedCodes = value or {}; Config:Set("SelectedCodes", value); Config:Save() end,
})

Main2:Button({
    Title = "兑换选中码",
    Desc = "只兑换下拉框中选中的代码",
    Callback = function()
        for _, code in ipairs(SelectedCodes or {}) do
            pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(code); task.wait(0.2) end)
        end
    end,
})

Main2:Button({
    Title = "兑换所有码",
    Desc = "一次性兑换所有可用代码",
    Callback = function()
        for _, code in ipairs(GlobalTables.redeemCodes or {}) do
            pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(code); task.wait(0.5) end)
        end
    end,
})

-- ====================== UI: UNLOCK GAMEPASS ======================
Main2:Section({ Title = "解锁游戏通行证", Icon = "badge-dollar-sign" })

local SelectedGamepass = Config:Get("SelectedGamepass", {})
GlobalTables.Gamepassts = SelectedGamepass

GamepassDropdown = Main2:Dropdown({
    Title = "选择通行证",
    Multi = true,
    Values = GlobalTables.Gamepasst,
    Value = SelectedGamepass,
    Callback = function(value)
        GlobalTables.Gamepassts = value or {}
        SelectedGamepass = value or {}
        Config:Set("SelectedGamepass", value)
        Config:Save()
    end,
})

Main2:Button({
    Title = "解锁选中通行证",
    Desc = "本地解锁选中的游戏通行证",
    Callback = function()
        local gachaData = LocalPlayer:FindFirstChild("GachaData")
        if not gachaData then
            gachaData = Instance.new("Folder")
            gachaData.Name = "GachaData"
            gachaData.Parent = LocalPlayer
        end
        local toUnlock = {}
        for _, v in ipairs(GlobalTables.Gamepassts) do
            if v == "全部" then
                toUnlock = {"LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost"}
                break
            else
                table.insert(toUnlock, v)
            end
        end
        if #toUnlock == 0 then
            WindUI:Notify({ Title = "解锁通行证", Content = "请先选择通行证", Duration = 3, Icon = "alert-triangle" })
            return
        end
        local successCount = 0
        for _, gamepassName in ipairs(toUnlock) do
            pcall(function()
                local boolValue = gachaData:FindFirstChild(gamepassName)
                if not boolValue then
                    boolValue = Instance.new("BoolValue")
                    boolValue.Name = gamepassName
                    boolValue.Parent = gachaData
                end
                boolValue.Value = true
                successCount = successCount + 1
                task.wait(0.2)
            end)
        end
        WindUI:Notify({
            Title = "解锁通行证",
            Content = "已解锁 " .. successCount .. "/" .. #toUnlock .. " 个通行证",
            Duration = 3,
            Icon = "badge-check"
        })
    end,
})

-- ====================== UI: GAMEMODE TAB ======================
Main7:Section({ Title = "投票信息", TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = "自动投票: 游戏模式",
    Desc = "- [步骤1] 点击恢复投票系统\n- [步骤2] 在大厅中（游戏内）等待\n- [步骤3] 设置自动投票并等待",
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = "投票模式", Icon = "gamepad-2" })

Main7:Button({
    Title = "恢复投票系统",
    Desc = "⚠️ 首次使用自动投票前请点击此按钮",
    Callback = function()
        pcall(function()
            ReplicatedStorage.GetReadyRemote:FireServer("3", true)
            task.wait(1.25)
            ReplicatedStorage.GetReadyRemote:FireServer("3", false)
            task.wait(0.67)
            ReplicatedStorage.GetReadyRemote:FireServer("2", true)
            task.wait(1.25)
            ReplicatedStorage.GetReadyRemote:FireServer("2", false)
            task.wait(0.67)
            ReplicatedStorage.GetReadyRemote:FireServer("1", true)
        end)
        WindUI:Notify({
            Title = "恢复中",
            Content = "就绪，正在恢复投票系统...",
            Duration = 2,
            Icon = "check-circle"
        })
        task.wait(6)
        pcall(function()
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                char.HumanoidRootPart.CFrame = CFrame.new(-220, 3, -600)
            end
        end)
        WindUI:Notify({
            Title = "恢复中",
            Content = "投票系统恢复中，请稍等",
            Duration = 5,
            Icon = "check-circle"
        })
        task.wait(10)
        WindUI:Notify({
            Title = "恢复完成",
            Content = "投票系统已恢复，现在可以使用自动投票",
            Duration = 5,
            Icon = "check-circle"
        })
        pcall(function()
            ReplicatedStorage.GetReadyRemote:FireServer("3", true)
            task.wait(1.25)
            ReplicatedStorage.GetReadyRemote:FireServer("3", false)
            task.wait(0.67)
            ReplicatedStorage.GetReadyRemote:FireServer("2", true)
            task.wait(1.25)
            ReplicatedStorage.GetReadyRemote:FireServer("2", false)
        end)
    end
})

GameModeDropdown2 = Main7:Dropdown({
    Title = "设置投票模式",
    Values = GlobalTables.Votes,
    Multi = false,
    Value = AutoVoteValue,
    Callback = function(value)
        AutoVoteValue = value; Config:Set("AutoVoteValue", value); Config:Save()
        print("[DYHUB] 投票模式已选择: " .. tostring(value))
    end
})

AutoVoteIGToggle = Main7:Toggle({
    Title = "自动投票（局内）",
    Desc = "每回合自动为选定模式投票",
    Value = AutoVoteinGameEnabled,
    Callback = function(enabled)
        AutoVoteinGameEnabled = enabled; Config:Set("AutoVoteinGameEnabled", enabled); Config:Save()
        SetupAutoVote_InGame(enabled)
    end
})
Main7:Divider()
Main7:Section({ Title = "休闲信息", TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = "休闲模式: 任务选择",
    Desc = "- [步骤1] 在大厅中（不在游戏内）\n- [步骤2] 点击开始并进入经典模式选择界面\n- [步骤3] 选择休闲模式并完成传送\n- [步骤4] 运行脚本",
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = "游戏模式", Icon = "gamepad-2" })

GameModeDropdown = Main7:Dropdown({
    Title = "设置游戏模式",
    Values = GlobalTables.ModeDisplay,
    Multi = false,
    Value = (function()
        for ch, en in pairs(GlobalTables.ModeInternal) do
            if en == AutoGameValue then return ch end
        end
        return "普通模式"
    end)(),
    Callback = function(chineseValue)
        local englishValue = GlobalTables.ModeInternal[chineseValue]
        if englishValue then
            AutoGameValue = englishValue
            Config:Set("AutoGameValue", englishValue)
            Config:Save()
            print("[DYHUB] 游戏模式已选择: " .. englishValue)
        end
    end
})

-- 自动开始系统（大厅自动导航）
task.spawn(function()
    local playBtn = workspace:FindFirstChild("ForGui") and workspace.ForGui:FindFirstChild("SurfaceGui") and workspace.ForGui.SurfaceGui:FindFirstChild("Frame") and workspace.ForGui.SurfaceGui.Frame:FindFirstChild("Play")
    if playBtn then
        WindUI:Notify({ Title = "自动开始", Content = "检测到开始按钮，自动启动...", Duration = 2 })
        task.wait(1)
        local playGui = pg:FindFirstChild("Play")
        if not (playGui and playGui.Enabled) then
            pcall(function() playBtn:Activate() end)
            WindUI:Notify({ Title = "自动开始", Content = "已按下开始按钮", Duration = 2 })
        end
    end
    task.wait(1)
    local playGui = pg:FindFirstChild("Play")
    if playGui and playGui.Enabled then
        local classicBtn = playGui:FindFirstChild("Classic")
        if classicBtn then
            task.wait(1)
            pcall(function() classicBtn:Activate() end)
        end
        task.wait(1)
        local modeGui = pg:FindFirstChild("mode select2")
        if modeGui and modeGui.Enabled then
            local diffBtn = modeGui:FindFirstChild("MainFrame") and modeGui.MainFrame:FindFirstChild("DiffMode")
            if diffBtn then pcall(function() diffBtn:Activate() end) end
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(0.5)
        local loadingGui = pg:FindFirstChild("LoadingScreen")
        if loadingGui then pcall(function() loadingGui:Destroy() end) end
        local lobby = pg:FindFirstChild("Lobby")
        if lobby and lobby.Enabled then
            local btn = lobby:FindFirstChild("MainFrame") and lobby.MainFrame:FindFirstChild("Frame") and lobby.MainFrame.Frame:FindFirstChild("Create") and lobby.MainFrame.Frame.Create:FindFirstChild("TrackQuestButton")
            if btn and btn.Visible then
                pcall(function() btn:Activate() end)
                task.wait(0.5)
                if AutoVoteEnabled then
                    ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue })
                    WindUI:Notify({ Title = "大厅系统", Content = "游戏模式已创建", Duration = 2 })
                end
                break
            end
        end
    end
end)

AutoVoteToggle = Main7:Toggle({
    Title = "自动游戏模式（大厅）",
    Desc = "在大厅时自动创建选定的游戏模式",
    Value = AutoVoteEnabled,
    Callback = function(enabled)
        AutoVoteEnabled = enabled
        Config:Set("AutoVoteEnabled", enabled)
        Config:Save()
        if enabled then WindUI:Notify({ Title = "自动游戏模式", Content = "已启用", Duration = 2 }) end
    end
})

-- ====================== UI: AUTO BUY ======================
Main5:Section({ Title = "商店武器", Icon = "helicopter" })

-- 读取配置（英文），转换为中文显示
local currentWeaponEn = Config:Get("SelectedWeapon", "Stungun")
local currentWeaponDisplay = (function()
    for ch, en in pairs(GlobalTables.WeaponInternal) do
        if en == currentWeaponEn then return ch end
    end
    return "电击枪"
end)()

WeaponDropdown = Main5:Dropdown({
    Title = "选择武器",
    Values = GlobalTables.WeaponDisplay, Multi = false, Value = currentWeaponDisplay,
    Callback = function(chineseValue)
        local englishValue = GlobalTables.WeaponInternal[chineseValue]
        if englishValue then
            SelectedWeapon = englishValue
            Config:Set("SelectedWeapon", englishValue)
            Config:Save()
        end
    end
})

AutoBuyWeaponToggle = Main5:Toggle({
    Title = "自动购买武器", Value = AutoBuyWeaponEnabled,
    Callback = function(enabled)
        AutoBuyWeaponEnabled = enabled
        Config:Set("AutoBuyWeaponEnabled", enabled)
        Config:Save()
        if enabled then
            task.spawn(function()
                while AutoBuyWeaponEnabled do
                    if SelectedWeapon then
                        pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end)
                    end
                    task.wait(10)
                end
            end)
        end
    end
})

Main5:Button({
    Title = "购买武器（一次）",
    Callback = function()
        if SelectedWeapon then
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end)
        end
    end
})

Main5:Section({ Title = "商店道具", Icon = "helicopter" })

local currentMiscEn = Config:Get("SelectedMiscItem", "HeadPhone")
local currentMiscDisplay = (function()
    for ch, en in pairs(GlobalTables.MiscInternal) do
        if en == currentMiscEn then return ch end
    end
    return "耳机"
end)()

MiscShopDropdown = Main5:Dropdown({
    Title = "选择道具",
    Values = GlobalTables.MiscDisplay, Multi = false, Value = currentMiscDisplay,
    Callback = function(chineseValue)
        local englishValue = GlobalTables.MiscInternal[chineseValue]
        if englishValue then
            SelectedMiscItem = englishValue
            Config:Set("SelectedMiscItem", englishValue)
            Config:Save()
        end
    end
})

AutoBuyMiscToggle = Main5:Toggle({
    Title = "自动购买道具", Value = AutoBuyMiscEnabled,
    Callback = function(enabled)
        AutoBuyMiscEnabled = enabled
        Config:Set("AutoBuyMiscEnabled", enabled)
        Config:Save()
        if enabled then
            task.spawn(function()
                while AutoBuyMiscEnabled do
                    if SelectedMiscItem then
                        pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end)
                    end
                    task.wait(10)
                end
            end)
        end
    end
})

Main5:Button({
    Title = "购买道具（一次）",
    Callback = function()
        if SelectedMiscItem then
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end)
        end
    end
})

-- ====================== UI: COLLECT TAB ======================
Main6:Section({ Title = "物品收集", Icon = "package" })

AutoCollectToggle = Main6:Toggle({
    Title = "自动收集", Value = AutoCollectEnabled,
    Desc = "自动收集地图上出现的选定物品",
    Callback = function(state)
        AutoCollectEnabled = state; Config:Set("AutoCollectEnabled", state); Config:Save()
        if state then KnownCollectItems = {}; StartAutoCollectLoop()
        else CollectRunning = false end
    end
})

Main6:Section({ Title = "收集设置", Icon = "settings" })

CollectItemDropdown = Main6:Dropdown({
    Title = "收集物品",
    Values = CollectItems, Multi = true, Value = SelectedCollectItems,
    Callback = function(values) SelectedCollectItems = values or {}; Config:Set("SelectedCollectItems", values); Config:Save() end
})

CollectModeDropdown = Main6:Dropdown({
    Title = "收集模式",
    Values = { "Clean", "IDGF" }, Multi = false, Value = CollectMode,
    Callback = function(value) CollectMode = value; Config:Set("CollectMode", value); Config:Save() end
})

-- ====================== UI: SETTING TAB ======================
Main3:Section({ Title = "保存配置", Icon = "save" })

Main3:Button({
    Title = "立即保存配置",
    Desc = "将所有当前设置保存到配置文件",
    Callback = function()
        Config:Save()
        WindUI:Notify({ Title = "配置已保存", Content = "保存成功", Duration = 2, Icon = "save" })
    end
})

local AutoSaveEnabled = Config:Get("AutoSaveEnabled", true)
local AutoSaveDelay   = Config:Get("AutoSaveDelay", 15)

Main3:Toggle({
    Title = "自动保存配置", Value = AutoSaveEnabled,
    Desc = "按设定间隔自动保存配置",
    Callback = function(state) AutoSaveEnabled = state; Config:Set("AutoSaveEnabled", state); Config:Save() end
})

Main3:Input({
    Title = "保存间隔（秒）", Default = tostring(AutoSaveDelay), Placeholder = "默认: 15",
    Callback = function(text)
        local num = tonumber(text)
        if num and num >= 1 then AutoSaveDelay = num; Config:Set("AutoSaveDelay", num); Config:Save()
        else warn("[DYHUB] 间隔值无效") end
    end
})

Main3:Section({ Title = "服务器状态", Icon = "server" })

Main3:Button({
    Title = "更换服务器",
    Desc = "传送到此游戏的其他随机服务器",
    Callback = function()
        local TeleportService = game:GetService("TeleportService")
        local servers = {}
        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100"))
        end)
        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.id ~= game.JobId and server.playing < server.maxPlayers then
                    table.insert(servers, server.id)
                end
            end
        end
        if #servers > 0 then
            WindUI:Notify({ Title = "更换服务器", Content = "正在传送...", Duration = 2, Icon = "server" })
            task.wait(1)
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LocalPlayer)
        else
            WindUI:Notify({ Title = "更换失败", Content = "未找到可用服务器", Duration = 3, Icon = "alert-triangle" })
        end
    end
})

Main3:Button({
    Title = "重新加入",
    Desc = "重新加入当前游戏服务器",
    Callback = function()
        WindUI:Notify({ Title = "重新加入", Content = "正在重连...", Duration = 2, Icon = "refresh-cw" })
        task.wait(1)
        game:GetService("TeleportService"):Teleport(game.PlaceId, LocalPlayer)
    end
})

Main3:Section({ Title = "其他", Icon = "settings" })

NoBarrierToggle = Main3:Toggle({
    Title = "绕过边界（已失效）", Value = noBarrierActive,
    Desc = "尝试绕过隐形边界",
    Callback = function(value)
        noBarrierActive = value; Config:Set("NoBarrier", value); Config:Save()
        if value then startNoBarrier() else stopNoBarrier() end
    end
})

local antiafk = Main3:Toggle({
    Title = "防挂机检测", Value = AntiAFK,
    Desc = "防止因长时间不动被踢出游戏",
    Callback = function(enabled)
        AntiAFK = enabled; Config:Set("AntiAfk", enabled); Config:Save()
        if enabled then
            task.spawn(function()
                game.Players.LocalPlayer.Idled:Connect(function()
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton2(Vector2.new())
                end)
                while AntiAFK do
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton2(Vector2.new())
                    task.wait(60)
                end
            end)
        end
    end
})

-- ====================== 自动启动 ======================
if AutoFarmEnabled then
    task.wait(2)
    StartFarmLoop()
    HandleMiscOptions(MiscOptions)
end

if noBarrierActive then startNoBarrier() end

if ESP.Enabled then
    task.wait(2)
    StartESPLoop()
end

if AutoBuyWeaponEnabled then
    task.spawn(function()
        while AutoBuyWeaponEnabled do
            if SelectedWeapon then
                pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end)
            end
            task.wait(10)
        end
    end)
end

if AutoBuyMiscEnabled then
    task.spawn(function()
        while AutoBuyMiscEnabled do
            if SelectedMiscItem then
                pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end)
            end
            task.wait(10)
        end
    end)
end

if AutoCollectEnabled then
    task.wait(2)
    StartAutoCollectLoop()
end

if AutoVoteEnabled or AutoStartEnabled then
    RefreshVoteAndStartSetup()
end

if AutoVoteinGameEnabled then SetupAutoVote_InGame(true) end
-- ====================== 天文币刷取模块（集成到主脚本，不创建新窗口） ======================
-- 依赖主脚本中已有的：Main, WindUI, LocalPlayer, ReplicatedStorage, RunService, IdlePosition（可选）
-- 不使用 IdlePosition，而是使用突袭坐标作为传送点

-- ========== 模块变量 ==========
local AstroFarmRunning = false
local AstroOrbitConn = nil
local AstroStrikeRunning = false
local AstroForcePos = false
local AstroAngle = 0
local AstroGameDuration = 960        -- 单局最大等待时间（秒）
local AstroFlyRadius = 300           -- 飞行半径
local AstroFlySpeed = 2              -- 飞行速度
local AstroStrikePos = Vector3.new(-23.34, -0.19, 0.34)
local AstroFlyCenter = Vector3.new(0, 250, 0)
local TELEPORT_CF = CFrame.new(-23.34, -0.19, 0.34)   -- 传送点（突袭坐标）

-- ========== UI 检测函数（使用主脚本已有的 pg，如果没有则获取） ==========
local function IsInLobby()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return false end
    local lobby = pg:FindFirstChild("Lobby")
    return lobby and lobby.Enabled == true
end

local function HasVoteUI()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return false end
    for _, v in ipairs(pg:GetDescendants()) do
        if v:IsA("GuiObject") and v.Visible then
            local name = v.Name:lower()
            if name:find("vote") or name:find("map") or name:find("choose") then
                return true
            end
        end
    end
    return false
end

-- ========== 投票与准备（完全复刻原脚本方式） ==========
local function VoteAndPrepare()
    local vote = ReplicatedStorage:FindFirstChild("Vote")
    local getReady = ReplicatedStorage:FindFirstChild("GetReadyRemote")
    if vote then vote:FireServer("AstroV2") end
    task.wait(0.3)
    if getReady then getReady:FireServer("3", true) end
    task.wait(0.3)
    if getReady then getReady:FireServer("1", true) end
end

-- ========== 飞行模块（围绕地图中心，使用 Heartbeat 兼容忍者） ==========
local function StartOrbit()
    if AstroOrbitConn then AstroOrbitConn:Disconnect() end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    char.Humanoid.PlatformStand = true
    AstroAngle = 0
    AstroForcePos = false
    AstroOrbitConn = RunService.Heartbeat:Connect(function(dt)
        if AstroForcePos then
            AstroAngle = (AstroAngle + AstroFlySpeed * dt) % (math.pi * 2)
            return
        end
        AstroAngle = (AstroAngle + AstroFlySpeed * dt) % (math.pi * 2)
        local pos = AstroFlyCenter + Vector3.new(math.cos(AstroAngle) * AstroFlyRadius, 0, math.sin(AstroAngle) * AstroFlyRadius)
        local hrp = char.HumanoidRootPart
        hrp.CFrame = CFrame.new(pos, AstroFlyCenter)
        hrp.Velocity = Vector3.zero
        hrp.RotVelocity = Vector3.zero
    end)
end

local function StopOrbit()
    if AstroOrbitConn then AstroOrbitConn:Disconnect(); AstroOrbitConn = nil end
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.PlatformStand = false
    end
end

-- ========== 突袭模块（原样，加保护） ==========
local function StartStrike()
    if AstroStrikeRunning then return end
    AstroStrikeRunning = true
    task.spawn(function()
        while AstroStrikeRunning do
            pcall(function()
                local remote = ReplicatedStorage:FindFirstChild("VillanArcAirStrike")
                if remote and remote:IsA("RemoteEvent") then
                    remote:FireServer(AstroStrikePos)
                end
            end)
            task.wait(1)
        end
    end)
end

local function StopStrike()
    AstroStrikeRunning = false
end

-- ========== 传送至突袭坐标（短暂锁定位置） ==========
local function TeleportToStrikePos()
    AstroForcePos = true
    local start = tick()
    local conn = RunService.Heartbeat:Connect(function()
        if tick() - start > 1.5 then
            conn:Disconnect()
            AstroForcePos = false
            return
        end
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = TELEPORT_CF
            char.HumanoidRootPart.Velocity = Vector3.zero
        end
    end)
end

-- ========== 主循环 ==========
local function AstroFarmLoop()
    while AstroFarmRunning do
        -- 1. 等待大厅（Lobby UI 或投票 UI 出现）
        repeat task.wait(1) until IsInLobby() or HasVoteUI()
        
        -- 2. 投票并准备
        VoteAndPrepare()
        
        -- 3. 等待进入地图：投票 UI 消失 + 角色加载完成
        repeat
            task.wait(0.5)
        until (not HasVoteUI()) and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        task.wait(2)   -- 额外稳定
        
        -- 4. 开启飞行和突袭
        StartOrbit()
        StartStrike()
        
        -- 5. 等待游戏结束（最大超时，若中途回到大厅则提前结束）
        local startTime = tick()
        while AstroFarmRunning and (tick() - startTime) < AstroGameDuration do
            task.wait(2)
            if IsInLobby() or HasVoteUI() then
                break
            end
        end
        
        -- 6. 停止飞行和突袭
        StopOrbit()
        StopStrike()
        
        -- 7. 传送到突袭坐标
        TeleportToStrikePos()
        task.wait(2)
    end
end

local function StartAstroFarm()
    if AstroFarmRunning then return end
    AstroFarmRunning = true
    task.spawn(AstroFarmLoop)
end

local function StopAstroFarm()
    AstroFarmRunning = false
    StopOrbit()
    StopStrike()
end

-- ========== 添加 UI 控件到主脚本的 Main Tab（不创建新窗口） ==========
Main:Section({ Title = "天文币刷取", Icon = "star" })
Main:Toggle({
    Title = "自动刷天文币 (AstroV2)",
    Desc = "基于 UI 检测，进入地图后开始飞行+突袭，结束后传送到突袭坐标",
    Value = false,
    Callback = function(state)
        if state then
            StartAstroFarm()
            WindUI:Notify({ Title = "天文币刷取", Content = "已开启，请确保在大厅", Duration = 3, Icon = "star" })
        else
            StopAstroFarm()
            WindUI:Notify({ Title = "天文币刷取", Content = "已关闭", Duration = 2, Icon = "square" })
        end
    end
})
Main:Slider({
    Title = "单局最大等待时间（秒）",
    Value = { Min = 600, Max = 1200, Default = AstroGameDuration },
    Step = 30,
    Callback = function(v) AstroGameDuration = v end
})
Main:Slider({
    Title = "飞行半径",
    Value = { Min = 100, Max = 600, Default = AstroFlyRadius },
    Step = 50,
    Callback = function(v) AstroFlyRadius = v end
})
Main:Slider({
    Title = "飞行速度",
    Value = { Min = 1, Max = 10, Default = AstroFlySpeed },
    Step = 0.5,
    Callback = function(v) AstroFlySpeed = v end
})

print("[天文币模块] 已集成到主脚本，使用 UI 检测 + 突袭坐标传送")
print("[DYHUB] 版本 " .. version .. " " .. ver .. " 加载成功！")
print("[DYHUB] 配置系统已激活 | 自动保存间隔15秒")

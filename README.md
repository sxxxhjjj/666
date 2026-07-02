-- v190 | [Local Register Fix]
-- =========================
version = "Rework"
ver = "v023.91"
-- =========================

-- ====================== LOAD UI ======================
WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

-- ====================== GameLoad ======================
repeat task.wait() until game:IsLoaded()

-- ====================== LoadingGui ======================
p = game:GetService("Players").LocalPlayer
pg = p:WaitForChild("PlayerGui")

function waitLoadingGone(maxWait)
    maxWait = tonumber(maxWait) or 18
    local gui = pg:FindFirstChild("LoadingGui")
    if not gui then return true end

    WindUI:Notify({ Title = "初始化", Content = "游戏加载中，请稍候。", Duration = 3, Icon = "download" })

    local startedAt = tick()
    while gui and gui.Parent and tick() - startedAt < maxWait do
        task.wait(0.1)
    end

    if gui and gui.Parent then
        warn("[YYa] LoadingGui 未及时消失，继续执行。")
        return false
    end

    return true
end

waitLoadingGone(18)

WindUI:Notify({ Title = "初始化", Content = "加载完成，2 秒后启动。", Duration = 2, Icon = "shield-check" })
task.wait(2)

-- ====================== WAITING PART / FPS UNLOCK ======================
YYA_WAITING_PART_NAME = "YYA_WAITING_PART"
iddyhub = "rbxassetid://103720636367587"
YYA_WAITING_STAND_CF = CFrame.new(-23.3435822, 67, 0.341766357)
YYA_WAITING_PART_CF = CFrame.new(-23.3435822, 63.95, 0.341766357)
YYA_WAITING_PART_SIZE = Vector3.new(16, 1, 16)
YYA_WAITING_PART_VISIBLE_TRANSPARENCY = 1

function GetYYAWaitingStandCFrame()
    return YYA_WAITING_STAND_CF
end

function EnsureYYAWaitingPartImages(waitingPart)
    if not waitingPart or not waitingPart:IsA("BasePart") then return end

    local usedFaces = {}

    for _, obj in ipairs(waitingPart:GetChildren()) do
        if obj:IsA("Decal") and obj.Name == "yya_image" then
            if usedFaces[obj.Face] then
                obj:Destroy()
            else
                usedFaces[obj.Face] = obj
                obj.Texture = iddyhub
                obj.Transparency = 0
            end
        end
    end

    for _, face in ipairs(Enum.NormalId:GetEnumItems()) do
        if not usedFaces[face] then
            local decal = Instance.new("Decal")
            decal.Name = "yya_image"
            decal.Texture = iddyhub
            decal.Face = face
            decal.Transparency = 0
            decal.Parent = waitingPart
        end
    end
end

function ConfigureYYAWaitingPart(waitingPart)
    if not waitingPart or not waitingPart:IsA("BasePart") then return nil end

    waitingPart.Name = YYA_WAITING_PART_NAME
    waitingPart.Size = YYA_WAITING_PART_SIZE
    waitingPart.CFrame = YYA_WAITING_PART_CF
    waitingPart.Anchored = true
    waitingPart.CanTouch = false
    waitingPart.CanQuery = false
    waitingPart.CastShadow = false
    waitingPart.Material = Enum.Material.SmoothPlastic
    waitingPart.Color = Color3.fromRGB(45, 130, 255)
    waitingPart.TopSurface = Enum.SurfaceType.Smooth
    waitingPart.BottomSurface = Enum.SurfaceType.Smooth

    local active = AutoFarmEnabled == true
    waitingPart.CanCollide = active
    waitingPart.Transparency = active and YYA_WAITING_PART_VISIBLE_TRANSPARENCY or 1

    EnsureYYAWaitingPartImages(waitingPart)

    return waitingPart
end

function GetYYAWaitingPart()
    local keep = nil
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name == YYA_WAITING_PART_NAME and obj:IsA("BasePart") then
            if not keep then
                keep = obj
            else
                pcall(function() obj:Destroy() end)
            end
        end
    end
    return keep
end

function DestroyYYAWaitingPart()
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name == YYA_WAITING_PART_NAME and obj:IsA("BasePart") then
            pcall(function() obj:Destroy() end)
        end
    end
end

function EnsureYYAWaitingPart()
    local waitingPart = GetYYAWaitingPart()
    if not waitingPart then
        waitingPart = Instance.new("Part")
        waitingPart.Name = YYA_WAITING_PART_NAME
        waitingPart.Parent = workspace
    end
    return ConfigureYYAWaitingPart(waitingPart)
end

if setfpscap then
    setfpscap(240)
    WindUI:Notify({ Title = "服务", Content = "FPS 已解锁！ | " .. ver, Duration = 3, Icon = "cpu" })
    warn("FPS 已解锁！")
else
    WindUI:Notify({ Title = "无法使用", Content = "您的注入器不支持 setfpscap。", Duration = 3, Icon = "ban" })
end

-- ====================== CUSTOM CONFIG SYSTEM ======================
HttpService = game:GetService("HttpService")
ConfigFolder = "YYa_STBB"

CustomConfig = {}
CustomConfig.__index = CustomConfig

function CustomConfig.new()
    local self = setmetatable({}, CustomConfig)
    self.ConfigData = {}
    self.ConfigPath = ConfigFolder .. "/STBB_config.json"
    if isfolder and makefolder and not isfolder(ConfigFolder) then
        pcall(function() makefolder(ConfigFolder) end)
    end
    self:Load()
    return self
end

function CustomConfig:Set(key, value) self.ConfigData[key] = value end

function CustomConfig:Get(key, default)
    if self.ConfigData[key] ~= nil then return self.ConfigData[key] end
    return default
end

function CustomConfig:Save(force)
    if not writefile then return false end

    local now = tick()
    if not force and self._LastSaveAt and now - self._LastSaveAt < 0.75 then
        if not self._SaveQueued then
            self._SaveQueued = true
            task.delay(0.85, function()
                self._SaveQueued = false
                self:Save(true)
            end)
        end
        return true
    end

    local success, err = pcall(function()
        writefile(self.ConfigPath, HttpService:JSONEncode(self.ConfigData))
    end)

    if success then
        self._LastSaveAt = now
        return true
    else
        warn("[YYa] 保存失败:", err)
        return false
    end
end

function CustomConfig:Load()
    if isfile and readfile and isfile(self.ConfigPath) then
        local success, result = pcall(function()
            return HttpService:JSONDecode(readfile(self.ConfigPath))
        end)
        if success and type(result) == "table" then
            self.ConfigData = result
        else
            warn("[YYa] 加载配置失败，使用默认值")
            self.ConfigData = {}
        end
    else
        self.ConfigData = {}
    end
end

function CustomConfig:AutoSave(interval)
    if self._AutoSaveThread then
        pcall(function() task.cancel(self._AutoSaveThread) end)
        self._AutoSaveThread = nil
    end
    self._AutoSaveThread = task.spawn(function()
        while true do
            task.wait(interval or 15)
            self:Save()
        end
    end)
end

Config = CustomConfig.new()

-- ====================== UI DISPLAY NAME MAPPING ======================
GachaDisplayNames = { "1次抽奖", "10次抽奖", "100次抽奖", "1次幸运抽奖", "10次幸运抽奖" }
GachaMap = {
    ["1次抽奖"] = "1Spin",
    ["10次抽奖"] = "10Spins",
    ["100次抽奖"] = "100Spins",
    ["1次幸运抽奖"] = "1SpinLucky",
    ["10次幸运抽奖"] = "10SpinLucky",
}

CollectDisplayNames = { "时钟蜘蛛", "X-18 核心", "绿色能量核心", "奇怪发射器", "Astro 样本", "奇怪棱镜", "钥匙卡", "僵尸核心", "闪存驱动器", "礼物" }
CollectMap = {
    ["时钟蜘蛛"] = "Clock Spider",
    ["X-18 核心"] = "X-18 Core",
    ["绿色能量核心"] = "Green Energy Core",
    ["奇怪发射器"] = "Weird Transmitter",
    ["Astro 样本"] = "Astro Samples",
    ["奇怪棱镜"] = "Weird Prism",
    ["钥匙卡"] = "Key Card",
    ["僵尸核心"] = "Zombie Core",
    ["闪存驱动器"] = "Flash Drives",
    ["礼物"] = "Presents",
}

WeaponDisplayNames = { "电击枪", "火焰喷射器", "鱼叉枪", "霰弹枪", "脉冲步枪", "鱼叉霰弹枪", "EPD", "小型激光枪" }
WeaponMap = {
    ["电击枪"] = "Stungun",
    ["火焰喷射器"] = "Flamethrower",
    ["鱼叉枪"] = "Harpoon Gun",
    ["霰弹枪"] = "Shot Gun",
    ["脉冲步枪"] = "Pulse Rifle",
    ["鱼叉霰弹枪"] = "Shot Harpoon Gun",
    ["EPD"] = "EPD",
    ["小型激光枪"] = "Small Laser Gun",
}

MiscDisplayNames = { "头戴式耳机", "手雷", "喷气背包", "透镜" }
MiscMap = {
    ["头戴式耳机"] = "HeadPhone",
    ["手雷"] = "Grenade",
    ["喷气背包"] = "Jetpack",
    ["透镜"] = "Lens",
}

RequestDisplayNames = { "泰坦请求", "特殊泰坦请求", "扬声器请求" }
RequestMap = {
    ["泰坦请求"] = "Titan-Request",
    ["特殊泰坦请求"] = "SpecialTitan-Request",
    ["扬声器请求"] = "Speaker-Request",
}

GamepassDisplayNames = { "全部", "幸运加成", "稀有幸运加成", "传奇幸运加成" }
GamepassMap = {
    ["全部"] = "All",
    ["幸运加成"] = "LuckyBoost",
    ["稀有幸运加成"] = "RareLuckyBoost",
    ["传奇幸运加成"] = "LegendaryLuckyBoost",
}

FarmModeDisplayNames = { "普通模式", "Astro 坚守模式", "黑暗维度模式" }
FarmModeMap = {
    ["普通模式"] = "Normal Mode",
    ["Astro 坚守模式"] = "Astro Holdout Mode",
    ["黑暗维度模式"] = "Dark Dimension Mode",
}

FarmPositionDisplayNames = { "上方", "下方" }
FarmPositionMap = {
    ["上方"] = "Above",
    ["下方"] = "Under",
}

MovementDisplayNames = { "传送", "补间" }
MovementMap = {
    ["传送"] = "Teleport",
    ["补间"] = "Tween",
}

CollectModeDisplayNames = { "清洁", "IDGF" }
CollectModeMap = {
    ["清洁"] = "Clean",
    ["IDGF"] = "IDGF",
}

CameraModeDisplayNames = { "经典", "手动" }
CameraModeMap = {
    ["经典"] = "Classic",
    ["手动"] = "Manual",
}

VoteDisplayNames = { "普通", "非常困难", "困难", "疯狂", "噩梦", "Boss Rush", "黑暗维度", "地狱", "雷暴", "圣诞节", "僵尸", "Astro V2", "Astro", "1亿访问" }
VoteMap = {
    ["普通"] = "Normal",
    ["非常困难"] = "VeryHard",
    ["困难"] = "Hard",
    ["疯狂"] = "Insane",
    ["噩梦"] = "Nightmare",
    ["Boss Rush"] = "BossRush",
    ["黑暗维度"] = "DarkDimension",
    ["地狱"] = "Hell",
    ["雷暴"] = "ThunderStorm",
    ["圣诞节"] = "Christmas",
    ["僵尸"] = "Zombie",
    ["Astro V2"] = "AstroV2",
    ["Astro"] = "Astro",
    ["1亿访问"] = "100MVisit",
}

GameModeDisplayNames = { "普通", "困难", "非常困难", "疯狂", "噩梦", "Boss Rush", "黑暗维度", "地狱", "雷暴", "圣诞节", "僵尸", "Astro V2", "Astro", "1亿访问" }
GameModeMap = {
    ["普通"] = "Normal",
    ["困难"] = "Hard",
    ["非常困难"] = "VeryHard",
    ["疯狂"] = "Insane",
    ["噩梦"] = "Nightmare",
    ["Boss Rush"] = "BossRush",
    ["黑暗维度"] = "DarkDimension",
    ["地狱"] = "Hell",
    ["雷暴"] = "ThunderStorm",
    ["圣诞节"] = "Christmas",
    ["僵尸"] = "Zombie",
    ["Astro V2"] = "AstroV2",
    ["Astro"] = "Astro",
    ["1亿访问"] = "100MVisit",
}

TitanSpeakerUpgradeDisplayNames = { "喷气背包", "过载", "音波增幅器", "核心", "升级" }
TitanSpeakerUpgradeMap = {
    ["喷气背包"] = "Jetpack",
    ["过载"] = "OverCharge",
    ["音波增幅器"] = "SoundBooster",
    ["核心"] = "Core",
    ["升级"] = "Upgrade",
}

UTCMUpgradeDisplayNames = { "护盾", "冲击波", "透镜", "热度", "护甲" }
UTCMUpgradeMap = {
    ["护盾"] = "Shield",
    ["冲击波"] = "Blaster",
    ["透镜"] = "Lens",
    ["热度"] = "Heat",
    ["护甲"] = "Armor",
}

TVUpgradeDisplayNames = { "吸收", "共享过载", "护盾", "Astro 臂" }
TVUpgradeMap = {
    ["吸收"] = "Absorb",
    ["共享过载"] = "ShareOverCharge",
    ["护盾"] = "Shield",
    ["Astro 臂"] = "AstroArm",
}

ShopHourlyDisplayNames = {
    "幸运药水 I", "幸运药水 II", "幸运药水 III", "S-余烬",
    "BSX2:30", "BSX2:60", "BSX2:360",
    "闪存驱动器#1", "闪存驱动器#2", "闪存驱动器#3", "闪存驱动器#4", "闪存驱动器#5", "闪存驱动器#6",
    "大师卡：普通", "大师卡：普通泰坦", "大师卡：特殊泰坦",
}
ShopHourlyMap = {
    ["幸运药水 I"] = "LuckPotionI",
    ["幸运药水 II"] = "LuckPotionII",
    ["幸运药水 III"] = "LuckPotionIII",
    ["S-余烬"] = "S-Ember",
    ["BSX2:30"] = "BSX2:30",
    ["BSX2:60"] = "BSX2:60",
    ["BSX2:360"] = "BSX2:360",
    ["闪存驱动器#1"] = "FlashDrive#1",
    ["闪存驱动器#2"] = "FlashDrive#2",
    ["闪存驱动器#3"] = "FlashDrive#3",
    ["闪存驱动器#4"] = "FlashDrive#4",
    ["闪存驱动器#5"] = "FlashDrive#5",
    ["闪存驱动器#6"] = "FlashDrive#6",
    ["大师卡：普通"] = "MasterCard:Normal",
    ["大师卡：普通泰坦"] = "MasterCard:NormalTitan",
    ["大师卡：特殊泰坦"] = "MasterCard:SpecialTitan",
}

UseItemDisplayNames = { "礼物" }
UseItemMap = {
    ["礼物"] = "Presents",
}

function GetEnglishValue(map, displayName)
    return map[displayName] or displayName
end

function GetDisplayName(map, englishValue)
    for k, v in pairs(map) do
        if v == englishValue then
            return k
        end
    end
    return englishValue
end

-- ====================== WINDOW ======================
Players = game:GetService("Players")
LocalPlayer = Players.LocalPlayer
CoreGui = game:GetService("CoreGui")

Window = WindUI:CreateWindow({
    Title = "至尊版",
    IconThemed = true,
    Icon = "rbxassetid://103720636367587",
    Author = "STBB | 至尊版",
    Folder = "YYa",
    Size = UDim2.fromOffset(550, 380),
    Transparent = true,
    Theme = "Dark",
    BackgroundImageTransparency = 0.8,
    HasOutline = false,
    HideSearchBar = true,
    ScrollBarEnabled = true,
    User = { Enabled = true, Anonymous = false },
})

Window:SetToggleKey(Enum.KeyCode.K)

Window:Tag({ Title = "至尊版", Color = Color3.fromHex("#db7093") })

Window:EditOpenButton({
    Title = "至尊版 - 打开",
    Icon = "monitor",
    CornerRadius = UDim.new(0, 6),
    StrokeThickness = 2,
    Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)),
    Draggable = true
})

-- ====================== TABS ======================
Info = Window:Tab({ Title = "信息", Icon = "info" })
MainDivider = Window:Divider()
Main = Window:Tab({ Title = "主要", Icon = "rocket" })
Main4 = Window:Tab({ Title = "透视", Icon = "eye" })
Main2 = Window:Tab({ Title = "玩家", Icon = "user" })
MainDivider1 = Window:Divider()
Main5 = Window:Tab({ Title = "商店", Icon = "shopping-cart" })
Main6 = Window:Tab({ Title = "收集", Icon = "hand" })
Main7 = Window:Tab({ Title = "游戏模式", Icon = "gamepad-2" })
MainDivider2 = Window:Divider()
Main3 = Window:Tab({ Title = "设置", Icon = "settings" })
Window:SelectTab(1)

-- ======================== INFO ========================
Info:Section({ Title = "最新更新", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = "最新更新 | CL: " .. ver,
    Desc = "更新日期: 06/02/2026 | CL: " .. ver .. "\n• [新增] 杂项刷怪中重置波次\n• [新增] 上帝模式滑条下的重置波次滑块\n• [修复] 重置波次现在保持重置点延迟并优先于刷怪锁定\n• [修复] 当前波次已高于/低于目标时重置波次滑块立即触发\n• [修复] 刷怪 Astro 模式计时器波次耗尽时的漏洞\n• [修复] 设置中的相机模式与刷怪同步\n• [优化] 刷怪循环/钩子后代扫描",
    Image = "rbxassetid://103720636367587",
    ImageSize = 26,
})
Info:Divider()

-- ====================== SERVICES ======================
TweenService        = game:GetService("TweenService")
ReplicatedStorage   = game:GetService("ReplicatedStorage")
ReplicatedFirst     = game:GetService("ReplicatedFirst")
VirtualInputManager = game:GetService("VirtualInputManager")
RunService          = game:GetService("RunService")
UserInputService    = game:GetService("UserInputService")
Lighting            = game:GetService("Lighting")

-- ====================== PLAYER ======================
Client         = LocalPlayer
Character      = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

-- ====================== GLOBAL TABLES ======================
GlobalTables = {
    redeemCodes = { "100MVisit2", "100MVisit1", "CamArmada", "CCTVBase", "ADelayedGameIsEventuallyGoodButRushedGameIsForeverBad" },
    Weapon   = { "Stungun", "Flamethrower", "Harpoon Gun", "Shot Gun", "Pulse Rifle", "Shot Harpoon Gun", "EPD", "Small Laser Gun" },
    MiscShop = { "HeadPhone", "Grenade", "Jetpack", "Lens" },
    RequestTitanSpeaker = { "Titan-Request", "SpecialTitan-Request", "Speaker-Request" },
    Gamepasst = { "All", "LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost" },
    Gamepassts = {},
}
-- ====================== CONFIG VARIABLES ======================
skillList = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }
skillDropdownValues = { "全部", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }

-- ====================== FARM HELPERS ======================
function NormalizeFarmMode(mode)
    mode = tostring(mode or "补间")
    if mode == "传送" then return "传送" end
    if mode ~= "传送" and mode ~= "补间" then return "补间" end
    return mode
end

function NormalizeCollectMovement(mode)
    mode = tostring(mode or "补间")
    if mode ~= "传送" and mode ~= "补间" then return "补间" end
    return mode
end

function NormalizeCameraMode(mode)
    mode = tostring(mode or "手动")
    if mode == "Manuel" or mode:lower() == "manual" or mode == "手动" then return "手动" end
    if mode:lower() == "classic" or mode == "经典" then return "经典" end
    return "手动"
end

-- ====================== STATE VARIABLES ======================
AutoFarmEnabled = Config:Get("AutoFarmEnabled", false)
FarmPosition = Config:Get("FarmPosition", "上方")
FarmMode = NormalizeFarmMode(Config:Get("FarmMode", "补间"))
DarkDimensionCollecting = false
DarkDimensionLowValue = 0.900
DarkDimensionSafeValue = 0.950
DarkDimensionLastWarnAt = 0
DarkDimensionCollectToken = 0
DarkDimensionOrbSearchCooldown = 0
DarkDimensionJeffreyAvoidRange = 70
JeffreyTargetBlockUntil = {}
JeffreyLastUnsafeTargetAt = 0
JeffreySafeHoldUntil = 0
JeffreySafeRetargetDelay = 0.85
-- 删除 BypassJeffreyEnabled 变量
AntiJeffreyRange = Config:Get("AntiJeffreyRange", 50)
-- 删除 BypassJeffrey 相关变量
AntiJeffreyLoopRunning = false
AntiJeffreyGuardLoopRunning = false
AntiJeffreyLastPushAt = 0
JeffreyCacheList = {}
JeffreyCacheAt = 0
JeffreyCacheTTL = 0.55
AntiJeffreyEscapeBusy = false
AntiJeffreyLastEscapeAt = 0
AntiJeffreyEscapeCooldown = 0.32
AntiJeffreyEscapeStep = 70
AntiJeffreyDangerRange = 20
AntiJeffreyKillZoneRange = 5
AntiJeffreyHardEscapeStep = 70
AntiJeffreyCriticalEscapeStep = 90
AntiJeffreyEscapePauseUntil = 0
AntiJeffreyForceRetargetUntil = 0
AstroModeDoorTopCF = CFrame.new(-23.3435822, 67, 0.341766357)
AstroModeDoorBottomCF = CFrame.new(-23.3435822, 3, 0.341766357)
AstroModeFinalRunning = false
AstroModeLastFinalAt = 0
FarmAstroTokenEnabled = Config:Get("FarmAstroTokenEnabled", false)
FarmAstroTokenRunning = false
FarmAstroTokenPart = nil
FarmAstroTokenTween = nil
FarmAstroTokenNoClipConnection = nil
FarmAstroTokenPauseCollect = false
FarmAstroTokenLastCleanNotify = 0
FarmAstroTokenLastAutoFarmNotify = 0
FarmAstroTokenTimerHold = false
FarmAstroTokenTimerIgnoreUntil = 0
FarmAstroTokenRespawnCounter = 0
FarmAstroGodModePaused = false
FarmAstroReviveGodTriggered = false
FarmAstroFinalLockActive = false
FarmAstroTimerDropping = false
FarmAstroBottomGodTriggered = false
FarmAstroWaveTimerArmed = false
FarmAstroLastWaveTimer = nil
FarmAstroReviveTimerArmed = false
FarmAstroLastReviveTimer = nil
AutoAttackEnabled = false
AutoSkillEnabled = false
AutoSkipHeliEnabled = Config:Get("AutoSkipHeliEnabled", false)
DeleteMapEnabled = Config:Get("DeleteMapEnabled", false)
BoostFPS_Active_dummy = false
AutoStartEnabled = true
AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false)
AutoVoteValue = Config:Get("AutoVoteValue", "普通")
AutoVoteLoopRunning = false
AutoVoteLastFireAt = 0
AutoStartLastReadyAt = 0
GodModeEnabled = true
GodModeValue = Config:Get("GodModeValue", 50)
GodModeTriggered = false
ResetWaveValue = Config:Get("ResetWaveValue", 10)
ResetWaveEnabled = false
ResetWaveLoopRunning = false
ResetWaveTeleporting = false
ResetWaveTargetCF = CFrame.new(1250, 500, 1250)
ResetWaveHoldTime = 2
ResetWaveToken = 0
ResetWaveLastTriggeredWave = nil
ResetWaveLastTriggeredKey = nil
ResetWaveLastTeleportAt = 0
WaitingRespawn = false
IdlePosition = GetYYAWaitingStandCFrame() * CFrame.Angles(math.rad(0), 0, 0)
IdleHoldDistance = 12
IdleTeleportCooldown = 1.25
LastIdleTeleportAt = 0
IdlePositionReached = false
SkillDelay = Config:Get("SkillDelay", 1)
LoopDelay = 0.5
TweenSpeed = 1
HeightValue = Config:Get("HeightValue", 3)
NeedNoClip = false
LockActive = false
AutoStartConnection = nil
noBarrierConnection = nil
noBarrierActive = Config:Get("NoBarrier", false)
CameraMode = NormalizeCameraMode(Config:Get("CameraMode", "手动"))
if Config:Get("CameraMode", "手动") ~= CameraMode then Config:Set("CameraMode", CameraMode) end
FarmLoopRunning = false
AutoAttackLoopRunning = false
AutoSkillLoopRunning = false
FarmForceRetarget = false
FarmCollecting = false
CombatDebugEnabled = Config:Get("CombatDebugEnabled", false)
CombatDebugCooldowns = {}

EnhancedNormalEnabled = Config:Get("EnhancedNormalEnabled", false)
EnhancedDarkDimensionEnabled = Config:Get("EnhancedDarkDimensionEnabled", false)
EnhancedAstroEnabled = Config:Get("EnhancedAstroEnabled", false)

AutoFillUpEnabled = true
FillUpRunning = false

function UpdateYYAWaitingPartCollision()
    if AutoFarmEnabled ~= true then
        if DestroyYYAWaitingPart then DestroyYYAWaitingPart() end
        part = nil
        return
    end

    local waitingPart = EnsureYYAWaitingPart and EnsureYYAWaitingPart() or GetYYAWaitingPart()
    if not waitingPart then return end

    part = waitingPart
    pcall(function() ConfigureYYAWaitingPart(waitingPart) end)
end

UpdateYYAWaitingPartCollision()

workspace.ChildRemoved:Connect(function(obj)
    if obj and obj.Name == YYA_WAITING_PART_NAME and AutoFarmEnabled == true then
        task.defer(function() UpdateYYAWaitingPartCollision() end)
    end
end)

function CombatDebug(tag, message, cooldown, showNotify)
    if not CombatDebugEnabled then return end
    cooldown = cooldown or 3
    local now = tick()
    local key = tostring(tag or "Debug")

    if CombatDebugCooldowns[key] and now - CombatDebugCooldowns[key] < cooldown then return end
    CombatDebugCooldowns[key] = now

    local text = "[YYa][" .. key .. "] " .. tostring(message or "")
    print(text)

    if showNotify and WindUI then
        pcall(function()
            WindUI:Notify({ Title = "战斗调试", Content = tostring(message or ""), Duration = 3, Icon = "bug" })
        end)
    end
end

function IsMiscFarmAllowed()
    return true
end

function StopMiscFarmRuntime(reason)
end

function ApplyMiscFarmGate(reason)
    return true
end

CameraLastApplyAt = 0
CameraApplyCooldown = 0.22
CameraSyncToken = 0

function GetCameraTargetForMode(char)
    if not char or not char.Parent then return nil, nil end

    local humanoid = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")

    if CameraMode == "经典" then
        return char:FindFirstChild("Head") or humanoid or char:FindFirstChild("HumanoidRootPart"), humanoid
    end

    return humanoid or char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Head"), humanoid
end

function ApplyCameraMode(force)
    local now = tick()
    if force ~= true and now - (CameraLastApplyAt or 0) < (CameraApplyCooldown or 0.22) then return end
    CameraLastApplyAt = now

    pcall(function()
        local cam = workspace.CurrentCamera
        local char = LocalPlayer.Character or Character
        if not cam or not char then return end

        CameraMode = NormalizeCameraMode(CameraMode)
        local target, humanoid = GetCameraTargetForMode(char)
        if not target then return end

        if humanoid and not AutoFarmEnabled and not LockActive and not FarmCollecting then
            humanoid.AutoRotate = true
        end

        if cam.CameraType ~= Enum.CameraType.Custom then
            cam.CameraType = Enum.CameraType.Custom
        end

        if cam.CameraSubject ~= target then
            cam.CameraSubject = target
        end
    end)
end

function RequestCameraSync(force)
    CameraSyncToken = (CameraSyncToken or 0) + 1
    local token = CameraSyncToken
    task.delay(force and 0 or 0.05, function()
        if token == CameraSyncToken then ApplyCameraMode(force == true) end
    end)
end

LastFarmCameraStabilize = 0

function StabilizeFarmCamera()
    local now = tick()
    if now - (LastFarmCameraStabilize or 0) < 0.35 then return end
    LastFarmCameraStabilize = now

    if AutoFarmEnabled then ApplyCameraMode(false) end
end

function RestoreFarmCameraAndMovement()
    pcall(function()
        local char = LocalPlayer.Character or Character
        local humanoid = char and (char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid"))
        if humanoid then humanoid.AutoRotate = true end
        ApplyCameraMode(true)
    end)
end

workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function() RequestCameraSync(true) end)

MissingRemoteWarnAt = {}

function GetRemote(name)
    local remote = ReplicatedStorage and ReplicatedStorage:FindFirstChild(name)
    if not remote then
        local now = tick()
        if not MissingRemoteWarnAt[name] or now - MissingRemoteWarnAt[name] >= 10 then
            MissingRemoteWarnAt[name] = now
            warn("[YYa] 找不到远程事件: " .. tostring(name))
        end
        return nil
    end
    return remote
end

-- ====================== AUTO VOTE CORE / AUTO START SYNC ======================
function GetVoteUIFrame()
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return nil end

    local voteGui = playerGui:FindFirstChild("OpenVoteUI")
    if not voteGui then return nil end

    return voteGui:FindFirstChild("OPEN UI")
end

function IsVoteUIOpen()
    local frame = GetVoteUIFrame()
    return frame and frame.Visible == true
end

function HideVoteUI()
    local frame = GetVoteUIFrame()
    if frame then pcall(function() frame.Visible = false end) end
end

function FireAutoVote(force)
    if not force and not IsVoteUIOpen() then return false end

    local now = tick()
    if now - AutoVoteLastFireAt < 0.25 then return false end
    AutoVoteLastFireAt = now

    local remote = GetRemote("Vote")
    if not remote then pcall(function() remote = ReplicatedStorage:WaitForChild("Vote", 3) end) end
    if not remote then return false end

    local englishValue = VoteMap[AutoVoteValue] or AutoVoteValue

    local ok, err = pcall(function()
        remote:FireServer(englishValue)
    end)

    if ok then
        HideVoteUI()
        print("[YYa] 自动投票已触发:", AutoVoteValue, "->", englishValue)
        return true
    else
        warn("[YYa] 自动投票失败:", err)
        return false
    end
end

function StartAutoVoteLoop()
    if AutoVoteLoopRunning then return end
    AutoVoteLoopRunning = true

    task.spawn(function()
        while AutoVoteinGameEnabled do
            if IsVoteUIOpen() then
                if AutoStartEnabled and IsMiscFarmAllowed() then
                    FireGetReady(0)
                else
                    FireAutoVote(false)
                end
            end
            task.wait(0.2)
        end

        AutoVoteLoopRunning = false
    end)
end

-- ====================== NEW PRIORITY SYSTEM CONFIG ======================
HighHPThreshold = Config:Get("HighHPThreshold", 200)
_currentTargetPriority = 0
_interruptSignal = false

VirtualUser = game:GetService("VirtualUser")
AntiAFK = Config:Get("AntiAfk", true)

AutoBuyWeaponEnabled = Config:Get("AutoBuyWeaponEnabled", false)
AutoBuyMiscEnabled = Config:Get("AutoBuyMiscEnabled", false)
SelectedWeapon = Config:Get("SelectedWeapon", "电击枪")
SelectedMiscItem = Config:Get("SelectedMiscItem", "头戴式耳机")

-- ====================== FILL UP PART CONFIG ======================
FILLUP_PART_PATH = { "HelicopterShop", "ShopXDD", "PartForShop" }
FILLUP_TARGET_POS = Vector3.new(44.2756729, 26.3595276, -32.7318268)
FILLUP_POS_THRESHOLD = 0.5
FillUpRunning = false

function GetFillUpPart()
    local obj = workspace
    for _, key in ipairs(FILLUP_PART_PATH) do
        obj = obj:FindFirstChild(key)
        if not obj then return nil end
    end
    return obj
end

function IsFillUpPartReady()
    local p = GetFillUpPart()
    if not p then return false end
    return (p.CFrame.Position - FILLUP_TARGET_POS).Magnitude < FILLUP_POS_THRESHOLD
end

-- ====================== ALLY SYSTEM ======================
AllyNames = {
    ["Heavy Soldier Toilet V2"] = true,
    ["Quad Laser Toilet"] = true,
    ["Strider Rocket Laser"] = true,
    ["Helicopter Camera"] = true,
    ["Heavy Soldier Toilet V1"] = true,
    ["Rocket Heli v2"] = true,
    ["Gunner Camera man"] = true,
    ["Attack Helicopter"] = true,
    ["Swat Mutant"] = true,
    ["Huge DJ Toilet"] = true,
}

function IsAlly(mob) return AllyNames[mob.Name] ~= nil end

-- ====================== TP SYSTEM ======================
function tp(pu79)
    pcall(function()
        if not pu79 or not pu79.Target then return end
        local v80 = Client and Client.Character
        if not v80 then return end
        local hum = v80:FindFirstChildOfClass("Humanoid") or v80:FindFirstChild("Humanoid")
        local hrp = v80:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        if hum and hum.Sit == true then hum.Sit = false end
        NeedNoClip = true
        hrp.CFrame = pu79.Target * (pu79.Mod or CFrame.new(0, 0, 0))
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)
end

function Tp(p82)
    pcall(function()
        if not p82 then return end
        local char = Client and Client.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        if hum and hum.Sit == true then hum.Sit = false end
        for _, v86 in ipairs(char:GetDescendants()) do
            if v86:IsA("BasePart") then v86.CanCollide = false end
        end
        if not hrp:FindFirstChild("BodyClip") then
            local v87 = Instance.new("BodyVelocity")
            v87.Parent = hrp
            v87.Name = "BodyClip"
            v87.Velocity = Vector3.new(0, 0, 0)
            v87.MaxForce = Vector3.new(5, math.huge, 5)
        end
        hrp.CFrame = p82
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)
end

function tp1(p89)
    local v90 = game.Players.LocalPlayer
    if v90 and v90.Character and v90.Character:FindFirstChild("HumanoidRootPart") then
        v90.Character:FindFirstChild("HumanoidRootPart").CFrame = p89
    else
        warn("玩家角色或 HumanoidRootPart 未找到！")
    end
end
-- ====================== UTILITY FUNCTIONS ======================
function IsValidMob(obj)
    if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
        if Players:GetPlayerFromCharacter(obj) then return false end
        if IsAlly(obj) then return false end
        local humanoid = obj:FindFirstChild("Humanoid")
        if humanoid and humanoid.Health > 0 then return true end
    end
    return false
end

function IsMobDead(mob)
    if not mob or not mob.Parent then return true end
    local humanoid = mob:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return true end
    return false
end

function GetObjectRootPart(obj)
    if not obj or not obj.Parent then return nil end
    if obj:IsA("BasePart") then return obj end
    if obj:IsA("Model") then
        return obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart", true)
    end
    return nil
end

function IsJeffreyName(name)
    local n = tostring(name or ""):lower()
    return n == "jeffrey" or n == "jeffery"
end

function IsValidJeffreyObject(obj)
    if not obj or not IsJeffreyName(obj.Name) then return false end
    if obj:IsA("Model") then
        local hum = obj:FindFirstChildOfClass("Humanoid") or obj:FindFirstChild("Humanoid")
        if hum and hum.Health <= 0 then return false end
        return GetObjectRootPart(obj) ~= nil
    end
    return obj:IsA("BasePart") and obj.Parent ~= nil
end

function AddJeffreyRootFromObject(obj, list, seen)
    if obj and IsJeffreyName(obj.Name) and IsValidJeffreyObject(obj) then
        local root = GetObjectRootPart(obj)
        if root and not seen[root] then
            seen[root] = true
            table.insert(list, root)
        end
    end
end

function GetJeffreyRoots(forceRefresh)
    local now = tick()
    if not forceRefresh and now - JeffreyCacheAt <= JeffreyCacheTTL then return JeffreyCacheList end

    local list, seen = {}, {}
    pcall(function()
        for _, obj in ipairs(workspace:GetChildren()) do
            AddJeffreyRootFromObject(obj, list, seen)
        end

        local living = workspace:FindFirstChild("Living")
        if living then
            for _, obj in ipairs(living:GetDescendants()) do
                AddJeffreyRootFromObject(obj, list, seen)
            end
        end

        if #list == 0 then
            for _, obj in ipairs(workspace:GetDescendants()) do
                AddJeffreyRootFromObject(obj, list, seen)
            end
        end
    end)

    JeffreyCacheList = list
    JeffreyCacheAt = now
    return JeffreyCacheList
end

function GetNearestJeffreyRoot(pos, range, forceRefresh)
    if not pos then return nil, math.huge end
    local best, bestDist = nil, math.huge
    for _, root in ipairs(GetJeffreyRoots(forceRefresh == true)) do
        if root and root.Parent then
            local dist = (root.Position - pos).Magnitude
            if dist < bestDist then
                best = root
                bestDist = dist
            end
        end
    end
    if range and bestDist > range then return nil, bestDist end
    return best, bestDist
end

function IsJeffreyNearPosition(pos, range, forceRefresh)
    local root = GetNearestJeffreyRoot(pos, range or AntiJeffreyRange, forceRefresh == true)
    return root ~= nil
end

function GetMobRootPart(mob)
    if not mob or not mob.Parent then return nil end
    return mob:FindFirstChild("HumanoidRootPart") or mob.PrimaryPart or mob:FindFirstChildWhichIsA("BasePart", true)
end

function IsMobBlockedByJeffrey(mob, range)
    if not mob or not mob.Parent then return true end
    if IsJeffreyName(mob.Name) or IsMobTemporarilyBlocked(mob) then return true end
    local root = GetMobRootPart(mob)
    if not root then return true end
    range = math.max(range or DarkDimensionJeffreyAvoidRange, 65)
    if IsPositionBlockedByJeffrey(root.Position, range, false) then
        MarkMobUnsafeByJeffrey(mob, 2.5)
        return true
    end
    local cf = nil
    pcall(function() cf = GetTargetCFrame(mob, FarmPosition) end)
    if cf and IsPositionBlockedByJeffrey(cf.Position, range, false) then
        MarkMobUnsafeByJeffrey(mob, 2.5)
        return true
    end
    return false
end

function IsFarmJeffreyAvoidActive()
    return EnhancedDarkDimensionEnabled == true
end

function GetFarmJeffreyAvoidRange()
    return AntiJeffreyRange
end

function IsAntiJeffreyEscapePauseActive()
    return tick() < (AntiJeffreyEscapePauseUntil or 0)
end

function BreakFarmLockForJeffrey(reason, pauseTime)
    pauseTime = pauseTime or 0.35
    AntiJeffreyEscapePauseUntil = math.max(AntiJeffreyEscapePauseUntil or 0, tick() + pauseTime)
    AntiJeffreyForceRetargetUntil = math.max(AntiJeffreyForceRetargetUntil or 0, tick() + pauseTime + 0.2)
    FarmForceRetarget = true
    LockActive = false
    _interruptSignal = true
    WaitingRespawn = false

    pcall(function()
        RefreshCombatCharacter()
        if Character then
            local humanoid = Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.Sit = false
                humanoid.AutoRotate = false
            end
        end
        if HumanoidRootPart then
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)

    task.delay(pauseTime + 0.25, function()
        if tick() >= (AntiJeffreyForceRetargetUntil or 0) and not FarmCollecting and not DarkDimensionCollecting then
            FarmForceRetarget = false
            _interruptSignal = false
        end
    end)
end

function GetMinJeffreyDistanceAt(pos, forceRefresh)
    local minDist = math.huge
    for _, root in ipairs(GetJeffreyRoots(forceRefresh == true)) do
        if root and root.Parent then
            local d = (root.Position - pos).Magnitude
            if d < minDist then minDist = d end
        end
    end
    return minDist
end

function GetHorizontalDistance(a, b)
    if not a or not b then return math.huge end
    local dx = a.X - b.X
    local dz = a.Z - b.Z
    return math.sqrt(dx * dx + dz * dz)
end

function GetMinJeffreyHorizontalDistanceAt(pos, forceRefresh)
    local minDist = math.huge
    for _, root in ipairs(GetJeffreyRoots(forceRefresh == true)) do
        if root and root.Parent then
            local d = GetHorizontalDistance(root.Position, pos)
            if d < minDist then minDist = d end
        end
    end
    return minDist
end

function IsPositionBlockedByJeffrey(pos, range, forceRefresh)
    if not pos then return true end
    range = range or GetFarmJeffreyAvoidRange()
    return GetMinJeffreyHorizontalDistanceAt(pos, forceRefresh == true) <= range
end

function MarkMobUnsafeByJeffrey(mob, duration)
    if not mob then return end
    JeffreyTargetBlockUntil[mob] = tick() + (duration or 2.5)
    JeffreyLastUnsafeTargetAt = tick()
end

function IsMobTemporarilyBlocked(mob)
    if not mob then return true end
    local untilTime = JeffreyTargetBlockUntil[mob]
    if untilTime and tick() < untilTime then return true end
    if untilTime then JeffreyTargetBlockUntil[mob] = nil end
    return false
end

function HasAnyJeffreyRoot()
    local roots = GetJeffreyRoots(false)
    return roots and #roots > 0
end

function GetFarmTargetDangerRange()
    return AntiJeffreyRange
end

function IsFarmTargetSafeFromJeffrey(mob, forceRefresh)
    if not EnhancedDarkDimensionEnabled then return true end
    if not mob or not mob.Parent then return false end
    if IsJeffreyName(mob.Name) or IsMobTemporarilyBlocked(mob) then return false end

    local root = GetMobRootPart(mob)
    if not root then return false end

    local range = GetFarmTargetDangerRange()
    if IsPositionBlockedByJeffrey(root.Position, range, forceRefresh == true) then
        MarkMobUnsafeByJeffrey(mob, 2.5)
        return false
    end

    local cf = nil
    pcall(function() cf = GetTargetCFrame(mob, FarmPosition) end)
    if cf and IsPositionBlockedByJeffrey(cf.Position, range, forceRefresh == true) then
        MarkMobUnsafeByJeffrey(mob, 2.5)
        return false
    end

    return true
end

-- ============================================================
-- =============== BARRIER SAFE ESCAPE SYSTEM ==================
-- ============================================================
BarrierCacheParts = {}
BarrierCacheAt = 0
BarrierCacheTTL = 1.25
BarrierBoundsCache = nil
BarrierBoundsAt = 0
BarrierInsetPadding = 8
BarrierRayPadding = 7

function GetMapBarrierModel()
    local map = workspace:FindFirstChild("Map")
    if not map then return nil end
    return map:FindFirstChild("Barrier")
end

function GetBarrierParts(forceRefresh)
    local now = tick()
    if not forceRefresh and now - (BarrierCacheAt or 0) <= (BarrierCacheTTL or 1.25) then
        return BarrierCacheParts or {}
    end

    local parts = {}
    local model = GetMapBarrierModel()
    if model then
        for _, obj in ipairs(model:GetDescendants()) do
            if obj:IsA("BasePart") then
                table.insert(parts, obj)
            end
        end
        if model:IsA("BasePart") then
            table.insert(parts, model)
        end
    end

    BarrierCacheParts = parts
    BarrierCacheAt = now
    BarrierBoundsCache = nil
    return BarrierCacheParts
end

function AddBarrierCornerToBounds(pos, bounds)
    if not pos then return end
    if pos.X < bounds.minX then bounds.minX = pos.X end
    if pos.X > bounds.maxX then bounds.maxX = pos.X end
    if pos.Z < bounds.minZ then bounds.minZ = pos.Z end
    if pos.Z > bounds.maxZ then bounds.maxZ = pos.Z end
end

function GetBarrierBounds(forceRefresh)
    local now = tick()
    if not forceRefresh and BarrierBoundsCache and now - (BarrierBoundsAt or 0) <= (BarrierCacheTTL or 1.25) then
        return BarrierBoundsCache
    end

    local parts = GetBarrierParts(forceRefresh == true)
    if not parts or #parts == 0 then
        BarrierBoundsCache = nil
        return nil
    end

    local bounds = { minX = math.huge, maxX = -math.huge, minZ = math.huge, maxZ = -math.huge }
    for _, part in ipairs(parts) do
        if part and part.Parent and part:IsA("BasePart") then
            local sx, sy, sz = part.Size.X * 0.5, part.Size.Y * 0.5, part.Size.Z * 0.5
            local xs = { -sx, sx }
            local ys = { -sy, sy }
            local zs = { -sz, sz }
            for _, x in ipairs(xs) do
                for _, y in ipairs(ys) do
                    for _, z in ipairs(zs) do
                        AddBarrierCornerToBounds((part.CFrame * CFrame.new(x, y, z)).Position, bounds)
                    end
                end
            end
        end
    end

    if bounds.minX == math.huge or bounds.maxX == -math.huge or bounds.minZ == math.huge or bounds.maxZ == -math.huge then
        BarrierBoundsCache = nil
        return nil
    end

    BarrierBoundsCache = bounds
    BarrierBoundsAt = now
    return BarrierBoundsCache
end

function ClampValue(v, mn, mx)
    if mn > mx then
        local mid = (mn + mx) * 0.5
        return mid
    end
    if v < mn then return mn end
    if v > mx then return mx end
    return v
end

function ClampPositionInsideBarrier(pos, padding, forceRefresh)
    if not pos then return nil, false end
    local bounds = GetBarrierBounds(forceRefresh == true)
    if not bounds then return pos, false end

    padding = padding or BarrierInsetPadding or 8
    local minX, maxX = bounds.minX + padding, bounds.maxX - padding
    local minZ, maxZ = bounds.minZ + padding, bounds.maxZ - padding
    local x = ClampValue(pos.X, minX, maxX)
    local z = ClampValue(pos.Z, minZ, maxZ)
    local clamped = Vector3.new(x, pos.Y, z)
    return clamped, (math.abs(x - pos.X) > 0.05 or math.abs(z - pos.Z) > 0.05)
end

function IsPositionInsideBarrier(pos, padding, forceRefresh)
    if not pos then return false end
    local bounds = GetBarrierBounds(forceRefresh == true)
    if not bounds then return true end
    padding = padding or BarrierInsetPadding or 8
    return pos.X >= bounds.minX + padding and pos.X <= bounds.maxX - padding
        and pos.Z >= bounds.minZ + padding and pos.Z <= bounds.maxZ - padding
end

function RaycastBarrierPath(fromPos, toPos, forceRefresh)
    if not fromPos or not toPos then return nil end
    local parts = GetBarrierParts(forceRefresh == true)
    if not parts or #parts == 0 then return nil end

    local direction = toPos - fromPos
    if direction.Magnitude <= 0.1 then return nil end

    local params = RaycastParams.new()
    pcall(function() params.FilterType = Enum.RaycastFilterType.Include end)
    if tostring(params.FilterType):find("Include") == nil then
        pcall(function() params.FilterType = Enum.RaycastFilterType.Whitelist end)
    end
    params.FilterDescendantsInstances = parts
    params.IgnoreWater = true

    local ok, result = pcall(function()
        return workspace:Raycast(fromPos, direction, params)
    end)
    if ok then return result end
    return nil
end

function GetBarrierSafeEscapePosition(fromPos, wantedPos, forceRefresh)
    if not fromPos or not wantedPos then return nil, false end

    local adjusted = false
    local safePos, wasClamped = ClampPositionInsideBarrier(wantedPos, BarrierInsetPadding, forceRefresh == true)
    if wasClamped then adjusted = true end

    local hit = RaycastBarrierPath(fromPos, safePos, forceRefresh == true)
    if hit and hit.Position then
        local dir = safePos - fromPos
        if dir.Magnitude > 0.1 then
            safePos = hit.Position - dir.Unit * (BarrierRayPadding or 7)
            adjusted = true
        end
    end

    local safePos2, wasClamped2 = ClampPositionInsideBarrier(safePos, BarrierInsetPadding, false)
    if wasClamped2 then adjusted = true end
    safePos = safePos2

    if not IsPositionInsideBarrier(safePos, BarrierInsetPadding, false) then
        safePos = ClampPositionInsideBarrier(fromPos, BarrierInsetPadding, false)
        adjusted = true
    end

    return safePos, adjusted
end

workspace.DescendantAdded:Connect(function(obj)
    if obj and obj:IsA("BasePart") and obj.Name == "Barrier" then
        BarrierCacheAt = 0
        BarrierBoundsCache = nil
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    if obj and obj:IsA("BasePart") and obj.Name == "Barrier" then
        BarrierCacheAt = 0
        BarrierBoundsCache = nil
    end
end)

function MoveToJeffreySafeHold(reason)
    if not HasAnyJeffreyRoot() then return false end
    BreakFarmLockForJeffrey(reason or "Jeffrey 安全保持", 0.55)
    local cf = GetBestJeffreyEscapeCFrame(math.max(AntiJeffreyHardEscapeStep or 70, 85), true)
    if not cf then return false end
    JeffreySafeHoldUntil = tick() + 0.7
    return MoveFarmSpecialCFrame(cf, 0.18)
end

function ValidateFarmTargetBeforeMove(mob, reason)
    if IsFarmTargetSafeFromJeffrey(mob, true) then return true end
    BreakFarmLockForJeffrey(reason or "目标在 Jeffrey 危险范围内", 0.55)
    MoveToJeffreySafeHold(reason or "目标不安全，转移到安全位置")
    return false
end

function GetBestJeffreyEscapeCFrame(step, forceRefresh)
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then return nil end

    step = step or AntiJeffreyHardEscapeStep or 70
    local base = HumanoidRootPart.Position
    local look = HumanoidRootPart.CFrame.LookVector
    local dirs = {
        Vector3.new(1,0,0), Vector3.new(-1,0,0), Vector3.new(0,0,1), Vector3.new(0,0,-1),
        Vector3.new(1,0,1), Vector3.new(1,0,-1), Vector3.new(-1,0,1), Vector3.new(-1,0,-1),
    }
    local steps = { step, step * 0.75, step * 0.5, step * 0.32 }

    local bestPos, bestScore = nil, -math.huge
    for _, tryStep in ipairs(steps) do
        for _, dir in ipairs(dirs) do
            local unit = dir.Unit
            local wanted = Vector3.new(base.X + unit.X * tryStep, base.Y, base.Z + unit.Z * tryStep)
            local pos, adjusted = GetBarrierSafeEscapePosition(base, wanted, forceRefresh == true)
            if pos then
                local moveDist = (pos - base).Magnitude
                if moveDist >= 2 then
                    local score = GetMinJeffreyHorizontalDistanceAt(pos, forceRefresh == true) + math.min(moveDist, step) * 0.03
                    if adjusted then score = score - 6 end
                    if score > bestScore then
                        bestScore = score
                        bestPos = pos
                    end
                end
            end
        end
    end

    if not bestPos then
        bestPos = ClampPositionInsideBarrier(base, BarrierInsetPadding, true)
    end
    if not bestPos then return nil end
    return CFrame.new(bestPos, bestPos + look)
end

function GetJeffreyEscapeCFrame(range, step, forceRefresh)
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then return nil end
    range = range or GetFarmJeffreyAvoidRange()
    step = step or AntiJeffreyEscapeStep or 70

    local jeffrey = GetNearestJeffreyRoot(HumanoidRootPart.Position, range, forceRefresh == true)
    if not jeffrey then return nil end

    return GetBestJeffreyEscapeCFrame(step, forceRefresh == true)
end

function MoveAwayFromJeffrey(range, step, tweenTime, forceCritical)
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then return false end

    range = range or GetFarmJeffreyAvoidRange()
    local scanRange = math.max(range, AntiJeffreyDangerRange or 20)
    local jeffrey, dist = GetNearestJeffreyRoot(HumanoidRootPart.Position, scanRange, forceCritical == true)
    if not jeffrey then return false end

    local now = tick()
    local isKillZone = dist <= (AntiJeffreyKillZoneRange or 5)
    local isDanger = dist <= (AntiJeffreyDangerRange or 20)

    if AntiJeffreyEscapeBusy and not isKillZone then return true end
    if not isKillZone and now - AntiJeffreyLastEscapeAt < AntiJeffreyEscapeCooldown then return true end

    if isKillZone then
        step = math.max(step or 0, AntiJeffreyCriticalEscapeStep or 90)
        tweenTime = 0.08
    elseif isDanger then
        step = math.max(step or 0, AntiJeffreyHardEscapeStep or 70)
        tweenTime = tweenTime or 0.16
    else
        step = math.max(step or 0, AntiJeffreyEscapeStep or 70)
        tweenTime = tweenTime or 0.28
    end

    local cf = GetBestJeffreyEscapeCFrame(step, true)
    if not cf then return false end

    AntiJeffreyEscapeBusy = true
    AntiJeffreyLastEscapeAt = now
    BreakFarmLockForJeffrey("Jeffrey 逃离", isKillZone and 0.65 or 0.45)

    local ok = false
    if MoveFarmSpecialCFrame then
        ok = MoveFarmSpecialCFrame(cf, tweenTime)
    else
        pcall(function() Character:PivotTo(cf) end)
        ok = true
    end

    task.wait(0.03)
    pcall(function()
        if HumanoidRootPart then
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)

    AntiJeffreyEscapeBusy = false
    return ok
end

function HandleFarmJeffreyEmergency(mob)
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then return false end

    if not EnhancedDarkDimensionEnabled then return false end

    local range = math.max(GetFarmJeffreyAvoidRange(), DarkDimensionJeffreyAvoidRange or 70, AntiJeffreyDangerRange or 20)
    local scanRange = math.max(range, AntiJeffreyDangerRange or 20)
    local jeffrey, dist = GetNearestJeffreyRoot(HumanoidRootPart.Position, scanRange, true)

    if jeffrey and dist <= (AntiJeffreyKillZoneRange or 5) then
        MoveAwayFromJeffrey(scanRange, AntiJeffreyCriticalEscapeStep, 0.08, true)
        return true
    end

    if jeffrey and dist <= (AntiJeffreyDangerRange or 20) then
        MoveAwayFromJeffrey(scanRange, AntiJeffreyHardEscapeStep, 0.16, true)
        return true
    end

    if mob and not IsFarmTargetSafeFromJeffrey(mob, true) then
        MarkMobUnsafeByJeffrey(mob, 3)
        BreakFarmLockForJeffrey("目标被 Jeffrey 阻挡", 0.55)
        MoveToJeffreySafeHold("目标被 Jeffrey 阻挡")
        return true
    end

    if not mob and HasAnyJeffreyRoot() and tick() < (JeffreySafeHoldUntil or 0) then
        BreakFarmLockForJeffrey("Jeffrey 安全保持等待", 0.35)
        return true
    end

    if not mob and HasAnyJeffreyRoot() and tick() - (JeffreyLastUnsafeTargetAt or 0) <= 2 then
        MoveToJeffreySafeHold("所有目标在 Jeffrey 附近不安全")
        return true
    end

    if not mob and jeffrey and dist <= range then
        MoveAwayFromJeffrey(scanRange, AntiJeffreyHardEscapeStep, 0.22, true)
        return true
    end

    return false
end

function PushAwayFromJeffrey()
    if not EnhancedDarkDimensionEnabled then return false end
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then return false end

    local jeffrey, dist = GetNearestJeffreyRoot(HumanoidRootPart.Position, math.max(AntiJeffreyRange, AntiJeffreyDangerRange or 20), false)
    if not jeffrey then return false end

    local now = tick()
    if dist > (AntiJeffreyDangerRange or 20) and now - AntiJeffreyLastPushAt < 0.25 then return true end
    AntiJeffreyLastPushAt = now

    if dist <= (AntiJeffreyDangerRange or 20) or AutoFarmEnabled then
        return MoveAwayFromJeffrey(math.max(AntiJeffreyRange, AntiJeffreyDangerRange or 20), AntiJeffreyHardEscapeStep, 0.16, dist <= 20)
    end

    local pushSize = math.clamp(((AntiJeffreyRange - dist) / math.max(AntiJeffreyRange, 1)) * 18, 5, 28)
    local targetCF = GetBestJeffreyEscapeCFrame(pushSize, false)
    if not targetCF then return false end

    if MoveFarmSpecialCFrame then
        return MoveFarmSpecialCFrame(targetCF, 0.18)
    end

    pcall(function()
        Character:PivotTo(targetCF)
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
    end)
    return true
end
-- ====================== MOB CACHE SYSTEM ======================
MobCacheList = {}
MobCacheDirty = true
MobCacheFolder = nil
MobCacheLastRebuild = 0
MobCacheChildAddedConnection = nil
MobCacheChildRemovedConnection = nil
MobCacheWorkspaceAddedConnection = nil
MobCacheWorkspaceRemovedConnection = nil

function InvalidateMobCache(reason)
    MobCacheDirty = true
    CombatDebug("MobCache", "缓存失效: " .. tostring(reason or "未知"), 2)
end

function DisconnectMobCacheFolderHooks()
    if MobCacheChildAddedConnection then
        MobCacheChildAddedConnection:Disconnect()
        MobCacheChildAddedConnection = nil
    end
    if MobCacheChildRemovedConnection then
        MobCacheChildRemovedConnection:Disconnect()
        MobCacheChildRemovedConnection = nil
    end
end

function RestartCombatLoopsIfNeeded(reason)
    if AutoAttackEnabled and IsMiscFarmAllowed() and StartAutoAttack then
        CombatDebug("AutoAttackRestart", "重启检查: " .. tostring(reason or "未知"), 3)
        StartAutoAttack()
    end
    if AutoSkillEnabled and IsMiscFarmAllowed() and StartAutoSkill then
        CombatDebug("AutoSkillRestart", "重启检查: " .. tostring(reason or "未知"), 3)
        StartAutoSkill()
    end
end

function HookMobCacheFolder(folder)
    if MobCacheFolder == folder and MobCacheChildAddedConnection and MobCacheChildRemovedConnection then return end

    DisconnectMobCacheFolderHooks()
    MobCacheFolder = folder
    MobCacheList = {}
    MobCacheDirty = true

    if not folder then
        CombatDebug("MobCache", "Living 文件夹未找到。", 5)
        return
    end

    MobCacheChildAddedConnection = folder.ChildAdded:Connect(function(obj)
        InvalidateMobCache("怪物添加")
        CombatDebug("MobCacheAdded", "怪物出现: " .. tostring(obj and obj.Name or "nil"), 2)
        task.delay(0.15, function()
            InvalidateMobCache("怪物添加延迟扫描")
            RestartCombatLoopsIfNeeded("怪物添加")
        end)
        task.delay(0.75, function()
            InvalidateMobCache("怪物加载延迟扫描")
            RestartCombatLoopsIfNeeded("怪物加载")
        end)
    end)

    MobCacheChildRemovedConnection = folder.ChildRemoved:Connect(function(obj)
        InvalidateMobCache("怪物移除")
        CombatDebug("MobCacheRemoved", "怪物移除: " .. tostring(obj and obj.Name or "nil"), 2)
        task.delay(0.05, function()
            RestartCombatLoopsIfNeeded("怪物移除")
        end)
    end)

    CombatDebug("MobCache", "Living 文件夹已挂钩。", 5)
end

function SetupMobCacheWorkspaceHooks()
    if MobCacheWorkspaceAddedConnection then return end

    MobCacheWorkspaceAddedConnection = workspace.ChildAdded:Connect(function(obj)
        if obj and obj.Name == "Living" then
            HookMobCacheFolder(obj)
            InvalidateMobCache("Living 文件夹添加")
            task.delay(0.25, function()
                RestartCombatLoopsIfNeeded("Living 文件夹添加")
            end)
        end
    end)

    MobCacheWorkspaceRemovedConnection = workspace.ChildRemoved:Connect(function(obj)
        if obj and obj == MobCacheFolder then
            HookMobCacheFolder(nil)
            InvalidateMobCache("Living 文件夹移除")
        end
    end)
end

function RebuildMobCache()
    local folder = workspace:FindFirstChild("Living")
    if folder ~= MobCacheFolder then
        HookMobCacheFolder(folder)
    end

    MobCacheList = {}

    if folder then
        for _, mob in ipairs(folder:GetChildren()) do
            if IsValidMob(mob) then
                table.insert(MobCacheList, mob)
            end
        end
    end

    MobCacheDirty = false
    MobCacheLastRebuild = tick()
    CombatDebug("MobCacheRebuild", "缓存有效怪物数: " .. tostring(#MobCacheList), 3)
end

function GetCachedLivingMobs(forceRefresh)
    local folder = workspace:FindFirstChild("Living")
    if folder ~= MobCacheFolder then
        HookMobCacheFolder(folder)
    end

    if forceRefresh or MobCacheDirty or tick() - MobCacheLastRebuild > 2 then
        RebuildMobCache()
    end

    local alive = {}
    for _, mob in ipairs(MobCacheList) do
        if IsValidMob(mob) then
            table.insert(alive, mob)
        else
            MobCacheDirty = true
        end
    end

    if #alive == 0 and folder and #folder:GetChildren() > 0 and not forceRefresh then
        CombatDebug("MobCacheFallback", "缓存为空但 Living 有子对象，重建一次。", 3)
        RebuildMobCache()
        alive = {}
        for _, mob in ipairs(MobCacheList) do
            if IsValidMob(mob) then
                table.insert(alive, mob)
            end
        end
    end

    return alive
end

SetupMobCacheWorkspaceHooks()

-- ====================== MOB SELECTION ======================
function IsAstroMob(mob)
    return mob and mob.Name and mob.Name:lower():find("astro", 1, true) ~= nil
end

function GetFarmCandidateMobs(forceRefresh)
    local source = GetCachedLivingMobs(forceRefresh == true)
    if EnhancedDarkDimensionEnabled then
        local filtered = {}
        local range = GetFarmTargetDangerRange()
        for _, mob in ipairs(source) do
            if IsFarmTargetSafeFromJeffrey(mob, forceRefresh == true) then
                table.insert(filtered, mob)
            end
        end
        return filtered
    end
    return source
end

function GetNearestMob()
    if RefreshCombatCharacter then RefreshCombatCharacter() end
    if not HumanoidRootPart then return nil end

    local nearestMob, nearestDist = nil, math.huge
    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        local mobRoot = mob:FindFirstChild("HumanoidRootPart")
        if mobRoot then
            local d = (HumanoidRootPart.Position - mobRoot.Position).Magnitude
            if d < nearestDist then
                nearestDist = d
                nearestMob = mob
            end
        end
    end
    return nearestMob
end

function GetHighestMob()
    if RefreshCombatCharacter then RefreshCombatCharacter() end

    local highestMob, highestY = nil, -math.huge
    local myY = HumanoidRootPart and HumanoidRootPart.Position.Y or 0

    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        local mobRoot = mob:FindFirstChild("HumanoidRootPart")
        if mobRoot then
            local mobY = mobRoot.Position.Y
            if mobY > myY and mobY > highestY then
                highestY = mobY
                highestMob = mob
            end
        end
    end

    return highestMob
end

-- ============================================================
-- ====================== PRIORITY SYSTEM =====================
-- ============================================================

function GetHelicopter()
    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        if mob.Name:lower():find("helicopter") then
            return mob
        end
    end
    return nil
end

function GetGiantSTToilet()
    if EnhancedAstroEnabled then return nil, nil end
    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        if mob.Name == "Giant ST toilet" then
            local lever = mob:FindFirstChild("lever")
            if lever then
                local prompt = lever:FindFirstChildOfClass("ProximityPrompt")
                if prompt then return mob, prompt end
            end
        end
    end
    return nil, nil
end

function GetHighHPMob()
    local bestMob, bestHP = nil, HighHPThreshold
    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        local hp = GetMobMaxHP(mob)
        if hp > bestHP then
            bestHP = hp
            bestMob = mob
        end
    end
    return bestMob
end

function GetPriorityMob()
    if RefreshCombatCharacter then RefreshCombatCharacter() end
    if not HumanoidRootPart then return nil, nil, nil, 0 end

    if EnhancedAstroEnabled then
        for _, mob in ipairs(GetFarmCandidateMobs(false)) do
            if IsAstroMob(mob) then
                return mob, "Astro", nil, 1
            end
        end
        return nil, nil, nil, 0
    end

    local giant, prompt = nil, nil
    local heli, highMob, nearMob = nil, nil, nil
    local bestHP, nearDist = HighHPThreshold, math.huge
    local candidates = GetFarmCandidateMobs(false)

    for _, mob in ipairs(candidates) do
        if not giant and not EnhancedAstroEnabled and mob.Name == "Giant ST toilet" then
            local lever = mob:FindFirstChild("lever")
            if lever then
                local pr = lever:FindFirstChildOfClass("ProximityPrompt")
                if pr then
                    giant = mob
                    prompt = pr
                end
            end
        end

        if not heli and mob.Name:lower():find("helicopter") then
            heli = mob
        end

        local hp = GetMobMaxHP(mob)
        if hp > bestHP then
            bestHP = hp
            highMob = mob
        end

        local mobRoot = mob:FindFirstChild("HumanoidRootPart")
        if mobRoot and HumanoidRootPart then
            local d = (HumanoidRootPart.Position - mobRoot.Position).Magnitude
            if d < nearDist then
                nearDist = d
                nearMob = mob
            end
        end
    end

    if giant and prompt then return giant, "GiantST", prompt, 4 end
    if heli then return heli, "直升机", nil, 3 end
    if highMob then return highMob, "高血量", nil, 2 end
    if nearMob then return nearMob, "最近怪物", nil, 1 end

    return nil, nil, nil, 0
end

function CheckInterrupt(currentPriority)
    local mob, _, _, newPriority = GetPriorityMob()
    if mob and newPriority > (currentPriority or 0) then
        return true, newPriority
    end
    return false, currentPriority
end

-- ============================================================
-- ====================== MOB VISUAL BOUNDS ===================
-- ============================================================

MobBoundsCache = {}
MobBoundsCacheAt = {}
MobBoundsCacheTTL = 0.25

function ClearMobBoundsCache(mob)
    if mob then
        MobBoundsCache[mob] = nil
        MobBoundsCacheAt[mob] = nil
    else
        MobBoundsCache = {}
        MobBoundsCacheAt = {}
    end
end

function ComputeMobVisualBounds(mob)
    local minY, maxY = math.huge, -math.huge
    local centerX, centerZ, count = 0, 0, 0

    for _, part in ipairs(mob:GetDescendants()) do
        if part:IsA("BasePart") and part.Transparency < 0.9 and part.Size.Y > 0.1 then
            local pos = part.Position
            local hy = part.Size.Y * 0.5
            if pos.Y - hy < minY then minY = pos.Y - hy end
            if pos.Y + hy > maxY then maxY = pos.Y + hy end
            centerX = centerX + pos.X
            centerZ = centerZ + pos.Z
            count = count + 1
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

function GetMobVisualBounds(mob)
    if not mob then return Vector3.new(0, 0, 0), 0, 4 end

    local now = tick()
    local cached = MobBoundsCache[mob]
    if cached and MobBoundsCacheAt[mob] and now - MobBoundsCacheAt[mob] <= MobBoundsCacheTTL then
        return cached[1], cached[2], cached[3]
    end

    local center, minY, maxY = ComputeMobVisualBounds(mob)
    MobBoundsCache[mob] = { center, minY, maxY }
    MobBoundsCacheAt[mob] = now
    return center, minY, maxY
end

-- ============================================================
-- ====================== MOB HEIGHT OVERRIDE =================
-- ============================================================

PADDING_REDUCE_STEP = Config:Get("PaddingReduceStep", 2)
PADDING_SAFE_MIN = Config:Get("PaddingSafeMin", -30)
DMG_THRESHOLD = Config:Get("DmgThreshold", 40)
ANTI_CLIP_MARGIN = Config:Get("AntiClipMargin", 3)
PLAYER_HALF_HEIGHT = 3

MobHeightOverride = {}
MobConfirmedPadding = {}
MobLastHealth = {}
MobCheckerCancelled = {}

function GetAntiClipFloor(mob, position)
    local _, minY, maxY = GetMobVisualBounds(mob)
    local visualHeight = maxY - minY
    return -(visualHeight) + PLAYER_HALF_HEIGHT + ANTI_CLIP_MARGIN
end

function GetEffectivePadding(mob)
    if MobConfirmedPadding[mob] ~= nil then return MobConfirmedPadding[mob] end
    if MobHeightOverride[mob] ~= nil then return MobHeightOverride[mob] end
    return HeightValue
end

function ClampPaddingToAntiClip(mob, padding)
    local antiFloor = GetAntiClipFloor(mob, FarmPosition)
    local clamped = math.max(padding, antiFloor)
    clamped = math.max(clamped, PADDING_SAFE_MIN)
    return clamped
end

function StartDamageChecker(mob)
    MobCheckerCancelled[mob] = false
    task.spawn(function()
        local humanoid = mob and mob:FindFirstChild("Humanoid")
        if not humanoid then return end
        if MobConfirmedPadding[mob] ~= nil then return end

        MobLastHealth[mob] = humanoid.Health
        MobHeightOverride[mob] = ClampPaddingToAntiClip(mob, MobHeightOverride[mob] or HeightValue)

        local lastDamageTime = tick()
        local noDamageTimer = 0
        local hitStreak = 0
        local lastWasHit = false
        local reducedOnce = false

        while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do
            task.wait(0.3)
            if MobCheckerCancelled[mob] then break end
            if not mob or not mob.Parent or IsMobDead(mob) then break end
            humanoid = mob:FindFirstChild("Humanoid")
            if not humanoid then break end

            local currentHP = humanoid.Health
            local lastHP = MobLastHealth[mob] or currentHP
            local dmgDealt = lastHP - currentHP
            local gotHit = dmgDealt > 0

            if gotHit then
                lastDamageTime = tick()
                noDamageTimer = 0
                reducedOnce = false
                if lastWasHit then hitStreak = hitStreak + 1 else hitStreak = 1 end
                lastWasHit = true
                local curPad = GetEffectivePadding(mob)
                if dmgDealt >= DMG_THRESHOLD and MobConfirmedPadding[mob] == nil then
                    if not MobCheckerCancelled[mob] then
                        MobConfirmedPadding[mob] = curPad
                        MobHeightOverride[mob] = curPad
                    end
                    break
                end
                if hitStreak >= 2 and MobConfirmedPadding[mob] == nil then
                    if not MobCheckerCancelled[mob] then
                        MobConfirmedPadding[mob] = curPad
                        MobHeightOverride[mob] = curPad
                    end
                    break
                end
            else
                lastWasHit = false
                hitStreak = 0
                noDamageTimer = tick() - lastDamageTime
            end

            if noDamageTimer >= 3 and not reducedOnce then
                reducedOnce = true
                local curPad = GetEffectivePadding(mob)
                local newPad = ClampPaddingToAntiClip(mob, curPad - PADDING_REDUCE_STEP)
                if newPad ~= curPad then
                    MobHeightOverride[mob] = newPad
                end
            end

            if noDamageTimer >= 6 then
                lastDamageTime = tick()
                reducedOnce = false
                local curPad = GetEffectivePadding(mob)
                local newPad = ClampPaddingToAntiClip(mob, curPad - PADDING_REDUCE_STEP)
                if newPad ~= curPad then
                    MobHeightOverride[mob] = newPad
                end
            end

            MobLastHealth[mob] = currentHP
        end

        if not MobCheckerCancelled[mob] then
            MobHeightOverride[mob] = nil
            MobLastHealth[mob] = nil
        end
    end)
end

function ResetMobOverride(mob)
    MobCheckerCancelled[mob] = true
    MobHeightOverride[mob] = nil
    MobConfirmedPadding[mob] = nil
    MobLastHealth[mob] = nil
    task.delay(0.5, function()
        MobCheckerCancelled[mob] = nil
    end)
end

-- ============================================================
-- ====================== TARGET CFRAME =======================
-- ============================================================
function GetTargetCFrame(mob, position)
    local mobRoot = mob:FindFirstChild("HumanoidRootPart")
    if not mobRoot then return nil end

    local padding = GetEffectivePadding(mob)
    local center, minY, maxY = GetMobVisualBounds(mob)

    if position == "上方" then
        local safeTargetY = math.max(maxY + padding, maxY + 0.5)
        local targetPos = Vector3.new(center.X, safeTargetY, center.Z)
        local lookAtPos = Vector3.new(center.X, maxY, center.Z)
        local lookCF = CFrame.new(targetPos, lookAtPos)
        return lookCF * CFrame.Angles(math.rad(-10), 0, 0)

    elseif position == "下方" then
        local safeTargetY = math.min(minY - padding, minY - 0.5)
        local targetPos = Vector3.new(center.X, safeTargetY, center.Z)
        local lookAtPos = Vector3.new(center.X, minY, center.Z)
        local lookCF = CFrame.new(targetPos, lookAtPos)
        return lookCF * CFrame.Angles(math.rad(10), 0, 0)
    end
end

function GetStableFarmCFrame(cf)
    return cf
end

function WaitTweenWithTimeout(tween, timeout)
    if not tween then return false end
    timeout = tonumber(timeout) or 2

    local completed = false
    local conn
    conn = tween.Completed:Connect(function()
        completed = true
    end)

    local startedAt = tick()
    while not completed and tick() - startedAt < timeout do
        if ResetWaveTeleporting then break end
        if not AutoFarmEnabled and not FarmAstroTokenEnabled and not DarkDimensionCollecting then break end
        task.wait(0.03)
    end

    if conn then
        pcall(function() conn:Disconnect() end)
    end
    if not completed then
        pcall(function() tween:Cancel() end)
    end
    return completed
end

function MoveCharacterToFarmCFrame(cf)
    if ResetWaveTeleporting then return end
    if not Character or not HumanoidRootPart or not cf then return end

    local targetCF = GetStableFarmCFrame(cf)
    pcall(function()
        local humanoid = Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Sit = false
            humanoid.AutoRotate = false
        end

        Character:PivotTo(targetCF)
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        StabilizeFarmCamera()
    end)
end

function TeleportToMob(mob)
    local cf = GetTargetCFrame(mob, FarmPosition)
    if not cf then return end

    if FarmMode == "补间" then
        local tweenInfo = TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
        local tween = TweenService:Create(HumanoidRootPart, tweenInfo, { CFrame = GetStableFarmCFrame(cf) })
        tween:Play()
        WaitTweenWithTimeout(tween, (TweenSpeed or 1) + 0.45)
        if not ResetWaveTeleporting and not FarmForceRetarget then
            MoveCharacterToFarmCFrame(cf)
        end
    else
        MoveCharacterToFarmCFrame(cf)
    end
end
-- ============================================================
-- =========== DARK DIMENSION / Astro Holdout Mode MOVEMENT ===========
-- ============================================================

function MoveSpecialCharacterCFrame(cf)
    if ResetWaveTeleporting then return false end
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart or not cf then return false end

    local ok = pcall(function()
        local humanoid = Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Sit = false
            humanoid.AutoRotate = false
        end
        Character:PivotTo(cf)
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        StabilizeFarmCamera()
    end)

    return ok
end

function MoveFarmSpecialCFrame(cf, tweenTime)
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart or not cf then return false end

    if FarmMode == "补间" then
        local ok = pcall(function()
            local tween = TweenService:Create(
                HumanoidRootPart,
                TweenInfo.new(tweenTime or TweenSpeed or 0.35, Enum.EasingStyle.Linear, Enum.EasingDirection.Out),
                { CFrame = cf }
            )
            tween:Play()
            WaitTweenWithTimeout(tween, (tweenTime or TweenSpeed or 0.35) + 0.45)
            if not ResetWaveTeleporting then
                MoveSpecialCharacterCFrame(cf)
            end
        end)
        return ok
    end

    return MoveSpecialCharacterCFrame(cf)
end

function StopFarmLockForSanityCollect(reason)
    DarkDimensionCollectToken = (DarkDimensionCollectToken or 0) + 1
    DarkDimensionCollecting = true
    FarmCollecting = true
    FarmForceRetarget = true
    LockActive = false
    _interruptSignal = true

    pcall(function()
        RefreshCombatCharacter()
        if Character then
            local humanoid = Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.Sit = false
                humanoid.AutoRotate = false
            end
        end
        if HumanoidRootPart then
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)

    pcall(function() RunService.Heartbeat:Wait() end)
    task.wait(0.03)
    return DarkDimensionCollectToken
end

function GetSanityImageLabel()
    local gui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
    if not gui then return nil end
    local sanity = gui:FindFirstChild("SanityUI")
    if not sanity then return nil end
    return sanity:FindFirstChild("ImageLabel")
end

function GetSanityTransparency()
    local label = GetSanityImageLabel()
    if not label then return nil end
    local ok, value = pcall(function() return label.ImageTransparency end)
    if ok and type(value) == "number" then return value end
    return nil
end

function HasSanityTouchInterest(part)
    if not part or not part.Parent then return false end
    if part:FindFirstChild("TouchInterest") then return true end
    local ok, found = pcall(function() return part:FindFirstChildOfClass("TouchTransmitter") end)
    return ok and found ~= nil
end

function IsValidSanityTouchPart(part)
    return part and part:IsA("BasePart") and part.Parent and HasSanityTouchInterest(part)
end

function GetSanityTouchPart(orb)
    if not orb or not orb.Parent then return nil end
    if IsValidSanityTouchPart(orb) then return orb end

    local touch = orb:FindFirstChild("TouchInterest", true)
    if touch and touch.Parent and touch.Parent:IsA("BasePart") then
        return touch.Parent
    end

    for _, obj in ipairs(orb:GetDescendants()) do
        if IsValidSanityTouchPart(obj) then
            return obj
        end
    end

    return nil
end

function ScanSanityOrbContainer(container, bestPart, bestDist)
    if not container then return bestPart, bestDist end

    if container.Name == "OrbSanity" then
        local directPart = GetSanityTouchPart(container)
        if directPart then
            local directDist = (HumanoidRootPart.Position - directPart.Position).Magnitude
            if directDist < bestDist then
                bestPart = directPart
                bestDist = directDist
            end
        end
    end

    for _, obj in ipairs(container:GetDescendants()) do
        if obj.Name == "OrbSanity" then
            local part = GetSanityTouchPart(obj)
            if part then
                local dist = (HumanoidRootPart.Position - part.Position).Magnitude
                if dist < bestDist then
                    bestDist = dist
                    bestPart = part
                end
            end
        end
    end

    return bestPart, bestDist
end

function GetNearestSanityOrbPart()
    RefreshCombatCharacter()
    if not HumanoidRootPart then return nil end

    local bestPart, bestDist = nil, math.huge
    local living = workspace:FindFirstChild("Living")
    bestPart, bestDist = ScanSanityOrbContainer(living, bestPart, bestDist)

    if not bestPart and tick() - (DarkDimensionOrbSearchCooldown or 0) > 0.4 then
        DarkDimensionOrbSearchCooldown = tick()
        bestPart, bestDist = ScanSanityOrbContainer(workspace, bestPart, bestDist)
    end

    return bestPart
end

function TouchSanityOrb(part)
    RefreshCombatCharacter()
    if not part or not part.Parent or not HumanoidRootPart then return false end

    local touchCF = part.CFrame + Vector3.new(0, 2, 0)
    MoveSpecialCharacterCFrame(touchCF)

    local ok = pcall(function()
        if firetouchinterest and HasSanityTouchInterest(part) then
            for _ = 1, 3 do
                if not HasSanityTouchInterest(part) then break end
                firetouchinterest(HumanoidRootPart, part, 0)
                task.wait(0.08)
                firetouchinterest(HumanoidRootPart, part, 1)
                task.wait(0.08)
            end
        else
            MoveSpecialCharacterCFrame(touchCF)
            task.wait(0.2)
        end
    end)

    return ok
end

function MoveToSanityOrb(part)
    if not part or not part.Parent then return false end
    local topCF = part.CFrame + Vector3.new(0, 2, 0)
    local ok = MoveFarmSpecialCFrame(topCF, 0.35)
    if ok then
        task.wait(0.05)
        MoveSpecialCharacterCFrame(topCF)
    end
    return ok
end

function WarnDarkDimensionMissingOrb()
    local now = tick()
    if now - DarkDimensionLastWarnAt < 4 then return end
    DarkDimensionLastWarnAt = now
    CombatDebug("DarkDimension", "理智值低但未找到 OrbSanity 带有 TouchInterest。", 4, false)
end

function IsDarkDimensionSanityLow()
    if not EnhancedDarkDimensionEnabled or DarkDimensionCollecting or not AutoFarmEnabled then return false end
    local sanity = GetSanityTransparency()
    return sanity ~= nil and sanity < DarkDimensionLowValue
end

function HandleDarkDimensionSanityEmergency()
    if not IsDarkDimensionSanityLow() then return false end
    FarmForceRetarget = true
    LockActive = false
    return HandleDarkDimensionSanity()
end

function HandleDarkDimensionSanity()
    if not EnhancedDarkDimensionEnabled or DarkDimensionCollecting or not AutoFarmEnabled then return false end

    local sanity = GetSanityTransparency()
    if not sanity or sanity >= DarkDimensionLowValue then return false end

    local collectToken = StopFarmLockForSanityCollect("理智值低")

    local didCollect = false
    pcall(function()
        while AutoFarmEnabled and EnhancedDarkDimensionEnabled and DarkDimensionCollectToken == collectToken do
            RefreshCombatCharacter()
            if not Character or not HumanoidRootPart then break end
            LockActive = false
            FarmCollecting = true
            FarmForceRetarget = true

            sanity = GetSanityTransparency()
            if not sanity then break end
            if sanity >= DarkDimensionSafeValue then break end

            local part = GetNearestSanityOrbPart()
            if not part then
                WarnDarkDimensionMissingOrb()
                break
            end

            if MoveToSanityOrb(part) then
                task.wait(1.25)
                if DarkDimensionCollectToken ~= collectToken then break end
                MoveSpecialCharacterCFrame(part.CFrame + Vector3.new(0, 2, 0))
                TouchSanityOrb(part)
                didCollect = true
            end

            local waited = 0
            repeat
                task.wait(0.12)
                waited = waited + 0.12
                sanity = GetSanityTransparency()
            until not AutoFarmEnabled or not EnhancedDarkDimensionEnabled or not sanity or sanity >= DarkDimensionSafeValue or not part.Parent or not HasSanityTouchInterest(part) or waited >= 3
        end
    end)

    if DarkDimensionCollectToken == collectToken then
        DarkDimensionCollecting = false
        FarmCollecting = false
        _interruptSignal = false
        if AutoFarmEnabled then
            WaitingRespawn = false
            FarmForceRetarget = true
            if HandleFarmJeffreyEmergency then
                pcall(function() HandleFarmJeffreyEmergency(nil) end)
            end
            task.delay(JeffreySafeRetargetDelay or 0.85, function()
                if not DarkDimensionCollecting and not IsAntiJeffreyEscapePauseActive() then
                    FarmForceRetarget = false
                end
            end)
        else
            FarmForceRetarget = false
        end
    end

    return didCollect
end

function DoAstroModeFinalDoor()
    if AstroModeFinalRunning then return false end
    local now = tick()
    if now - AstroModeLastFinalAt < 2 then return false end
    AstroModeLastFinalAt = now
    AstroModeFinalRunning = true

    local ok = pcall(function()
        RefreshCombatCharacter()
        if not Character or not HumanoidRootPart then return end
        LockActive = false
        WaitingRespawn = true
        FarmForceRetarget = true

        MoveCharacterToFarmCFrame(AstroModeDoorTopCF)
        task.wait(0.12)

        if FarmMode == "补间" then
            MoveFarmSpecialCFrame(AstroModeDoorBottomCF, 0.45)
        else
            MoveCharacterToFarmCFrame(AstroModeDoorBottomCF)
        end

        task.wait(0.15)
        FarmForceRetarget = false
    end)

    AstroModeFinalRunning = false
    return ok
end

function LockToMob(mob)
    LockActive = true
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if not AutoFarmEnabled or IsMobDead(mob) or not LockActive or FarmForceRetarget or (IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive()) then
            connection:Disconnect()
            LockActive = false
            return
        end
        if not Character or not HumanoidRootPart then return end
        local cf = GetTargetCFrame(mob, FarmPosition)
        if cf then
            MoveCharacterToFarmCFrame(cf)
        end
    end)
end

-- ====================== AUTO LOOPS ======================
function RefreshCombatCharacter()
    if not Character or not Character.Parent then
        Character = LocalPlayer.Character
    end

    if Character and (not HumanoidRootPart or HumanoidRootPart.Parent ~= Character) then
        HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
    end

    Client = LocalPlayer
    return Character, HumanoidRootPart
end

function SafeGetPriorityMob()
    RefreshCombatCharacter()
    if not Character or not HumanoidRootPart then
        CombatDebug("CombatCharacter", "角色或 HumanoidRootPart 未就绪。", 4)
        return nil, nil, nil, 0
    end

    local ok, mob, mobType, extraData, priority = pcall(function()
        return GetPriorityMob()
    end)

    if ok then
        if mob then
            return mob, mobType, extraData, priority
        end

        if tick() - MobCacheLastRebuild > 0.6 then
            InvalidateMobCache("优先级返回 nil")
            local ok2, mob2, mobType2, extraData2, priority2 = pcall(function()
                RebuildMobCache()
                return GetPriorityMob()
            end)

            if ok2 and mob2 then
                CombatDebug("PriorityRecovered", "缓存重建后恢复优先级怪物: " .. tostring(mob2.Name), 3)
                return mob2, mobType2, extraData2, priority2
            end
        end

        CombatDebug("PriorityNoMob", "未找到有效怪物。", 4)
        return nil, nil, nil, 0
    end

    CombatDebug("PriorityError", "GetPriorityMob 失败: " .. tostring(mob), 3, true)
    warn("[YYa] GetPriorityMob 失败:", tostring(mob))
    InvalidateMobCache("优先级错误")
    return nil, nil, nil, 0
end

function StartAutoAttack()
    if AutoAttackLoopRunning then return end
    AutoAttackLoopRunning = true

    task.spawn(function()
        while AutoAttackEnabled do
            if FarmCollecting then
                task.wait(0.2)
            elseif IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive() then
                task.wait(0.08)
            elseif IsDarkDimensionSanityLow and IsDarkDimensionSanityLow() then
                task.wait(0.1)
            elseif IsMiscFarmAllowed() then
                local mob = SafeGetPriorityMob()
                if mob then
                    WaitingRespawn = false
                    local remote = GetRemote("LMB")
                    if remote then
                        pcall(function() remote:FireServer() end)
                    else
                        CombatDebug("AutoAttackRemote", "LMB 远程事件缺失。", 5, true)
                    end
                else
                    CombatDebug("AutoAttackNoMob", "自动攻击等待有效怪物。", 5)
                end
                task.wait(0.12)
            else
                CombatDebug("AutoAttackPaused", "自动攻击被同步锁定暂停。", 5)
                task.wait(0.25)
            end
        end

        AutoAttackLoopRunning = false
    end)
end

function StartAutoSkill()
    if AutoSkillLoopRunning then return end
    AutoSkillLoopRunning = true

    task.spawn(function()
        while AutoSkillEnabled do
            if FarmCollecting then
                task.wait(0.2)
            elseif IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive() then
                task.wait(0.08)
            elseif IsDarkDimensionSanityLow and IsDarkDimensionSanityLow() then
                task.wait(0.1)
            elseif IsMiscFarmAllowed() then
                local mob = SafeGetPriorityMob()
                if mob then
                    WaitingRespawn = false

                    local keysToPress = {}
                    if table.find(SelectedSkills, "全部") then
                        keysToPress = skillList
                    else
                        keysToPress = SelectedSkills
                    end

                    for _, key in ipairs(keysToPress) do
                        if not AutoSkillEnabled or not IsMiscFarmAllowed() or FarmCollecting or (IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive()) or (IsDarkDimensionSanityLow and IsDarkDimensionSanityLow()) then break end

                        local keyCode = Enum.KeyCode[key]
                        if keyCode then
                            pcall(function()
                                VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
                                task.wait(0.05)
                                VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
                            end)
                            task.wait(SkillDelay)
                        else
                            CombatDebug("AutoSkillKey", "无效技能键: " .. tostring(key), 5)
                        end
                    end
                else
                    CombatDebug("AutoSkillNoMob", "自动技能等待有效怪物。", 5)
                    task.wait(0.25)
                end
            else
                CombatDebug("AutoSkillPaused", "自动技能被同步锁定暂停。", 5)
                task.wait(0.25)
            end
            task.wait(0.05)
        end

        AutoSkillLoopRunning = false
    end)
end

function TriggerAutoSkipHeli(state)
    if not AutoSkipHeliEnabled then return end
    local remote = GetRemote("SetSettingAutoSkipWave")
    if remote then
        pcall(function() remote:FireServer(state) end)
    end
end

function HasHumanoid(obj)
    if obj:IsA("Model") then
        return obj:FindFirstChildOfClass("Humanoid") ~= nil
    end
    return false
end

function IsLivingDescendant(obj)
    local current = obj
    while current and current ~= workspace do
        if current:IsA("Model") and current:FindFirstChildOfClass("Humanoid") then
            return true
        end
        current = current.Parent
    end
    return false
end
-- ====================== Delete Map (Delete Map) SYSTEM ======================
BoostFPS_OriginalData = {}
BoostFPS_Active = false
BoostFPS_RestoreConnection = nil
BoostFPS_LightingData = {}

function SaveAndBoostFPS()
    if BoostFPS_Active then return end
    BoostFPS_Active = true
    BoostFPS_OriginalData = {}
    BoostFPS_LightingData = {}

    local Lighting = game:GetService("Lighting")
    BoostFPS_LightingData = {
        Brightness = Lighting.Brightness,
        GlobalShadows = Lighting.GlobalShadows,
        FogEnd = Lighting.FogEnd,
        FogStart = Lighting.FogStart,
    }
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.Brightness = 1
        Lighting.FogEnd = 100000
        Lighting.FogStart = 100000
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
                            CastShadow = obj.CastShadow,
                            Material = obj.Material,
                        }
                        obj.Transparency = 1
                        obj.CastShadow = false
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
                    obj.CastShadow = false
                end
            elseif obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = 1
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or
                   obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                pcall(function() obj.Enabled = false end)
            end
        end)
    end)

    print("[YYa] 删除地图: ON")
end

function RestoreBoostFPS()
    if not BoostFPS_Active then return end
    BoostFPS_Active = false

    if BoostFPS_RestoreConnection then
        BoostFPS_RestoreConnection:Disconnect()
        BoostFPS_RestoreConnection = nil
    end

    local Lighting = game:GetService("Lighting")
    pcall(function()
        if BoostFPS_LightingData.Brightness ~= nil then Lighting.Brightness = BoostFPS_LightingData.Brightness end
        if BoostFPS_LightingData.GlobalShadows ~= nil then Lighting.GlobalShadows = BoostFPS_LightingData.GlobalShadows end
        if BoostFPS_LightingData.FogEnd ~= nil then Lighting.FogEnd = BoostFPS_LightingData.FogEnd end
        if BoostFPS_LightingData.FogStart ~= nil then Lighting.FogStart = BoostFPS_LightingData.FogStart end
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
            if data.Material ~= nil then pcall(function() obj.Material = data.Material end) end
            if data.Enabled ~= nil then pcall(function() obj.Enabled = data.Enabled end) end
            if data.TextureId ~= nil then obj.TextureId = data.TextureId end
        end)
    end

    BoostFPS_OriginalData = {}
    BoostFPS_LightingData = {}
    print("[YYa] 删除地图: OFF (已恢复)")
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
                                obj.CastShadow = false
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- ====================== PLAYER HP HELPERS ======================
function GetPlayerHPInfo()
    local humanoid = Character and Character:FindFirstChild("Humanoid")
    if not humanoid then return 100, 100 end
    return humanoid.Health, humanoid.MaxHealth
end

function IsPlayerHPFull()
    local hp, maxHp = GetPlayerHPInfo()
    if maxHp <= 0 then return true end
    return hp >= maxHp
end

function GetPlayerHealthPercent()
    local humanoid = Character and Character:FindFirstChild("Humanoid")
    if not humanoid then return 100 end
    if humanoid.MaxHealth <= 0 then return 100 end
    return (humanoid.Health / humanoid.MaxHealth) * 100
end

-- ====================== GOD MODE CORE ======================
function IsCharacterDeadForGodMode(char, humanoid)
    return not char or not char.Parent or not humanoid or not humanoid.Parent
        or humanoid.Health <= 0 or humanoid:GetState() == Enum.HumanoidStateType.Dead
end

function ForceGodModeOnce(reason)
    local ok, result = pcall(function()
        local char = LocalPlayer.Character
        if not char then return false end

        local humanoid = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")
        if not humanoid then return false end
        if IsCharacterDeadForGodMode(char, humanoid) then return false end

        local destroyed = false

        local head = char:FindFirstChild("Head")
        if head then
            head:Destroy()
            destroyed = true
        end

        task.wait(0.05)

        if IsCharacterDeadForGodMode(char, humanoid) then
            CombatDebug("GodMode", "上帝模式已触发: " .. tostring(reason or "手动"), 2)
            return true
        end

        local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
        if torso then
            torso:Destroy()
            destroyed = true
        end

        if not destroyed and not IsCharacterDeadForGodMode(char, humanoid) then
            humanoid.Health = 0
        end

        CombatDebug("GodMode", "上帝模式已触发: " .. tostring(reason or "手动"), 2)
        return true
    end)

    return ok and result == true
end

-- ====================== GOD MODE LOOP (所有模式强制启用) ======================
task.spawn(function()
    while true do
        task.wait(0.1)
        if AutoFarmEnabled then
            pcall(function()
                local char = LocalPlayer.Character
                if not char then return end

                local humanoid = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")
                if not humanoid then return end
                if humanoid.MaxHealth <= 0 then return end

                local hpPercent = (humanoid.Health / humanoid.MaxHealth) * 100

                if hpPercent < GodModeValue then
                    ForceGodModeOnce("血量低于上帝模式阈值")
                end
            end)
        end
    end
end)

-- ====================== AUTO FILL UP (所有模式常驻) ======================
function DoFillUp()
    local remote = GetRemote("ShopSystem")
    if not remote then return end
    for i = 1, 2 do
        pcall(function() remote:FireServer("Buy", "FillHP") end)
        if i < 2 then task.wait(0.3) end
    end
end

function StartAutoFillUpLoop()
    if FillUpRunning then return end
    FillUpRunning = true
    task.spawn(function()
        while AutoFarmEnabled do
            if not IsPlayerHPFull() then
                if AutoSkipHeliEnabled then
                    TriggerAutoSkipHeli(false)
                end
                local waited = 0
                while not IsFillUpPartReady() and AutoFarmEnabled do
                    waited = waited + 0.2
                    if waited >= 30 then break end
                    task.wait(0.2)
                end
                if IsFillUpPartReady() and AutoFarmEnabled then
                    DoFillUp()
                    task.wait(1)
                end
                if AutoSkipHeliEnabled then
                    TriggerAutoSkipHeli(true)
                end
            end
            task.wait(1)
        end
        FillUpRunning = false
    end)
end

-- ====================== BARRIER BYPASS ======================
function startNoBarrier()
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
                if humanoid then
                    humanoid.Health = humanoid.MaxHealth
                end
            end
        end)
    end)
end

function stopNoBarrier()
    if noBarrierConnection then
        noBarrierConnection:Disconnect()
        noBarrierConnection = nil
    end
end

-- ============================================================
-- ====================== AUTO START MODE ======================
function FireGetReady(delayBefore)
    if delayBefore == nil then delayBefore = 0 end
    if delayBefore > 0 then task.wait(delayBefore) end

    local now = tick()
    if now - (AutoStartLastReadyAt or 0) < 0.85 then return false end
    AutoStartLastReadyAt = now

    if AutoVoteinGameEnabled then
        FireAutoVote(true)
        task.wait(0.2)
    end

    local remote = GetRemote("GetReadyRemote")
    if not remote then return false end

    local ok, err = pcall(function()
        remote:FireServer("1", true)
        task.wait(0.2)
        remote:FireServer("1", false)
        task.wait(0.2)
        remote:FireServer("2", false)
        task.wait(0.2)
        remote:FireServer("3", false)
        task.wait(0.2)
        remote:FireServer("1", true)
    end)

    if not ok then warn("[YYa] GetReadyRemote 失败:", err) end
    return ok
end

function SetupAutoStartOnly(enabled)
    if AutoStartConnection then
        AutoStartConnection:Disconnect()
        AutoStartConnection = nil
    end

    if not enabled then return end

    FireGetReady(0)

    AutoStartConnection = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1)
        if AutoStartEnabled then
            task.spawn(function() FireGetReady(1) end)
        end
    end)
end

function StartAutoStart()
    AutoStartEnabled = true
    SetupAutoStartOnly(true)
end

function StopAutoStart()
    AutoStartEnabled = false
    SetupAutoStartOnly(false)
end

-- ====================== TELEPORT TO IDLE ======================
function StopIdleVelocity()
    pcall(function()
        if HumanoidRootPart then
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

function IsNearIdlePosition()
    if not HumanoidRootPart then return false end
    return (HumanoidRootPart.Position - IdlePosition.Position).Magnitude <= IdleHoldDistance
end

function TeleportToIdle(force)
    LockActive = false
    WaitingRespawn = true
    IdlePosition = GetYYAWaitingStandCFrame() * CFrame.Angles(math.rad(0), 0, 0)
    UpdateYYAWaitingPartCollision()

    if not Character or not Character.Parent or not HumanoidRootPart then return end

    local now = tick()

    if not force and IsNearIdlePosition() then
        IdlePositionReached = true
        StopIdleVelocity()
        return
    end

    if not force and (now - LastIdleTeleportAt) < IdleTeleportCooldown then
        StopIdleVelocity()
        return
    end

    LastIdleTeleportAt = now
    IdlePositionReached = true

    pcall(function()
        Character:PivotTo(IdlePosition)
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
    end)
end

-- ====================== PROXIMITY PROMPT HELPERS ======================
function ActivateProximityPrompt(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
        if fireproximityprompt then fireproximityprompt(prompt) end
        prompt:InputHoldBegin()
        task.wait(0.03)
        prompt:InputHoldEnd()
    end)
end

FlushPromptCache = {}
FlushPromptCacheDirty = true
FlushPromptCacheLastScan = 0
FlushPromptCacheTTL = 8

function IsFlushPrompt(prompt)
    if not prompt or not prompt:IsA("ProximityPrompt") then return false end
    local actionText = tostring(prompt.ActionText or ""):lower()
    local objectText = tostring(prompt.ObjectText or ""):lower()
    local combined = actionText .. " " .. objectText .. " " .. tostring(prompt.Name or ""):lower()
    return combined:find("flush", 1, true) ~= nil
        or combined:find("flash", 1, true) ~= nil
        or combined:find("dragon", 1, true) ~= nil
end

function RegisterFlushPrompt(obj)
    if obj and obj:IsA("ProximityPrompt") and IsFlushPrompt(obj) then
        FlushPromptCache[obj] = true
    end
end

function RebuildFlushPromptCache()
    FlushPromptCache = {}
    pcall(function()
        for _, obj in ipairs(workspace:GetDescendants()) do
            RegisterFlushPrompt(obj)
        end
    end)
    FlushPromptCacheDirty = false
    FlushPromptCacheLastScan = tick()
end

workspace.DescendantAdded:Connect(function(obj)
    if obj and obj:IsA("ProximityPrompt") then
        task.defer(function() RegisterFlushPrompt(obj) end)
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    if obj and FlushPromptCache[obj] then
        FlushPromptCache[obj] = nil
    end
end)

LastFlushPromptActivateAllAt = 0

function ActivateAllFlushPrompts()
    local now = tick()
    if now - (LastFlushPromptActivateAllAt or 0) < 0.35 then return end
    LastFlushPromptActivateAllAt = now

    pcall(function()
        if FlushPromptCacheDirty or tick() - (FlushPromptCacheLastScan or 0) > (FlushPromptCacheTTL or 8) then
            RebuildFlushPromptCache()
        end

        for prompt in pairs(FlushPromptCache) do
            if prompt and prompt.Parent and IsFlushPrompt(prompt) then
                ActivateProximityPrompt(prompt)
            else
                FlushPromptCache[prompt] = nil
            end
        end
    end)
end
-- ============================================================
-- ====================== COLLECT SYSTEM ======================
-- ============================================================

CollectItems = {
    "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter",
    "Astro Samples", "Weird Prism", "Key Card", "Zombie Core",
    "Flash Drives", "Presents",
}

CollectGroupMap = {
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

AutoCollectEnabled   = Config:Get("AutoCollectEnabled", false)
SelectedCollectItems = Config:Get("SelectedCollectItems", {})
CollectMode          = Config:Get("CollectMode", "清洁")
CollectMovementMode  = NormalizeCollectMovement(Config:Get("CollectMovementMode", "补间"))

KnownCollectItems = {}
CollectRunning    = false
CollectCandidateCache = {}
CollectCacheDirty = true
CollectLastFullScan = 0

function MatchesPattern(objectName, pattern)
    local objL, patL = tostring(objectName or ""):lower(), tostring(pattern or ""):lower()
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

function IsCollectTarget(objectName)
    for _, pattern in ipairs(SelectedCollectItems) do
        if MatchesPattern(objectName, pattern) then return true end
    end
    return false
end

function IsCollectObject(obj)
    return obj and obj.Parent and (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("BasePart"))
end

function AddCollectCandidate(obj)
    if IsCollectObject(obj) and IsCollectTarget(obj.Name) then
        CollectCandidateCache[obj] = true
        return true
    end
    return false
end

function RebuildCollectCache()
    CollectCandidateCache = {}
    if #SelectedCollectItems > 0 then
        for _, obj in ipairs(workspace:GetDescendants()) do
            AddCollectCandidate(obj)
        end
    end
    CollectCacheDirty = false
    CollectLastFullScan = tick()
end

function FindNewCollectItems()
    if CollectCacheDirty or tick() - CollectLastFullScan > 5 then
        RebuildCollectCache()
    end

    local found = {}
    for obj, _ in pairs(CollectCandidateCache) do
        if not obj or not obj.Parent or not IsCollectTarget(obj.Name) then
            CollectCandidateCache[obj] = nil
            KnownCollectItems[obj] = nil
        elseif not KnownCollectItems[obj] and IsCollectObject(obj) then
            table.insert(found, obj)
        end
    end
    return found
end

function GetItemRootPart(obj)
    if obj:IsA("Model") then
        return obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")
    elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then
        return obj
    end
    return nil
end

function GetItemTargetCFrame(itemRoot)
    if not itemRoot then return nil end
    return CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position)
end

function MoveToItem(itemRoot)
    RefreshCombatCharacter()
    if not itemRoot or not Character or not HumanoidRootPart then return false end

    local targetCF = GetItemTargetCFrame(itemRoot)
    if not targetCF then return false end

    if CollectMovementMode == "传送" then
        pcall(function()
            Character:PivotTo(targetCF)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end)
        return true
    end

    local ok = pcall(function()
        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = targetCF })
        tween:Play()
        local started = tick()
        repeat
            task.wait(0.05)
            if not AutoCollectEnabled or IsItemGone(itemRoot) then
                pcall(function() tween:Cancel() end)
                break
            end
        until tween.PlaybackState ~= Enum.PlaybackState.Playing or tick() - started > math.max(TweenSpeed + 1, 3)
        pcall(function()
            Character:PivotTo(targetCF)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end)
    end)

    return ok
end

function ActivateItemPrompts(obj)
    pcall(function()
        for _, child in ipairs(obj:GetDescendants()) do
            if child:IsA("ProximityPrompt") then
                child.HoldDuration = 0
                child.MaxActivationDistance = 50
                if fireproximityprompt then fireproximityprompt(child) end
                child:InputHoldBegin()
                task.wait(0.04)
                child:InputHoldEnd()
            end
        end
    end)
end

function IsItemGone(obj)
    return not obj or not obj.Parent
end

function BeginCollectPause()
    FarmCollecting = true
    FarmForceRetarget = true
    LockActive = false
    if FarmAstroTokenEnabled then
        SetFarmAstroCollectPause(true)
    end
    task.wait(0.08)
end

function EndCollectPause()
    if FarmAstroTokenEnabled then
        SetFarmAstroCollectPause(false)
    end
    FarmCollecting = false
    FarmForceRetarget = true
    if AutoFarmEnabled then
        WaitingRespawn = false
        StartFarmLoop()
    end
    task.delay(0.6, function()
        FarmForceRetarget = false
    end)
end

function CollectSingleItem(obj)
    if IsItemGone(obj) then return end
    local itemRoot = GetItemRootPart(obj)
    if not itemRoot then return end

    MoveToItem(itemRoot)

    local timeout = 0
    while AutoCollectEnabled and not IsItemGone(obj) and timeout < 8 do
        itemRoot = GetItemRootPart(obj)
        if not itemRoot then break end

        if timeout == 0 or timeout % 0.3 < 0.16 then
            local targetCF = GetItemTargetCFrame(itemRoot)
            pcall(function()
                if targetCF and Character and HumanoidRootPart then
                    Character:PivotTo(targetCF)
                    HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                    HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                end
            end)
        end

        ActivateItemPrompts(obj)
        task.wait(0.15)
        timeout = timeout + 0.15
    end

    KnownCollectItems[obj] = true
end

function AllMobsDead()
    return #GetFarmCandidateMobs(false) == 0
end

function StartAutoCollectLoop()
    if CollectRunning then return end
    CollectRunning = true
    task.spawn(function()
        while AutoCollectEnabled do
            if FarmAstroTokenEnabled and CollectMode == "清洁" then
                NotifyFarmAstroCleanMode()
                task.wait(1)
                continue
            end

            if #SelectedCollectItems > 0 then
                local itemsToCollect = FindNewCollectItems()
                if #itemsToCollect > 0 then
                    if CollectMode == "IDGF" then
                        BeginCollectPause()
                        for _, obj in ipairs(itemsToCollect) do
                            if not AutoCollectEnabled then break end
                            if not IsItemGone(obj) then
                                CollectSingleItem(obj)
                            else
                                KnownCollectItems[obj] = true
                            end
                        end
                        EndCollectPause()

                    elseif CollectMode == "清洁" then
                        local waitedClean = 0
                        while not AllMobsDead() and AutoCollectEnabled do
                            task.wait(0.5)
                            waitedClean = waitedClean + 0.5
                            if waitedClean >= 120 then break end
                        end
                        if not AutoCollectEnabled then break end

                        if AutoSkipHeliEnabled then
                            TriggerAutoSkipHeli(false)
                        end
                        BeginCollectPause()
                        for _, obj in ipairs(FindNewCollectItems()) do
                            if not AutoCollectEnabled then break end
                            if not IsItemGone(obj) then
                                CollectSingleItem(obj)
                            else
                                KnownCollectItems[obj] = true
                            end
                        end
                        EndCollectPause()
                        if AutoSkipHeliEnabled then
                            TriggerAutoSkipHeli(true)
                        end

                        if not IsPlayerHPFull() and AutoFillUpEnabled then
                            local fw = 0
                            while not IsPlayerHPFull() and AutoFillUpEnabled and AutoCollectEnabled do
                                task.wait(0.5)
                                fw = fw + 0.5
                                if fw >= 60 then break end
                            end
                        end
                    end
                else
                    for obj, _ in pairs(KnownCollectItems) do
                        if IsItemGone(obj) then
                            KnownCollectItems[obj] = nil
                        end
                    end
                end
            end
            task.wait(0.65)
        end
        FarmCollecting = false
        CollectRunning = false
    end)
end

workspace.DescendantAdded:Connect(function(obj)
    if not AutoCollectEnabled or #SelectedCollectItems == 0 then return end
    if AddCollectCandidate(obj) then
        CombatDebug("CollectItem", "新物品已缓存: " .. tostring(obj.Name), 3)
    end
end)

-- ============================================================
-- ====================== FARM ASTRO TOKEN ====================
-- ============================================================
FARM_ASTRO_TOKEN_IMAGE = "rbxassetid://103720636367587"
FARM_ASTRO_TOP_A       = CFrame.new(-680, 167, 505)
FARM_ASTRO_TOP_B       = CFrame.new(495, 167, 505)

FARM_ASTRO_LOW_A       = CFrame.new(-680, -15, -555)
FARM_ASTRO_LOW_B       = CFrame.new(500, -15, -555)
FARM_ASTRO_TIMER_TOP_CF = CFrame.new(-23.3435822, 67, 0.341766357)
FARM_ASTRO_TIMER_BOTTOM_CF = CFrame.new(-23.3435822, 2, 0.341766357)
FARM_ASTRO_TIMER_SAFE_CF = FARM_ASTRO_TIMER_BOTTOM_CF
FARM_ASTRO_TIMER_PART_OFFSET = CFrame.new(0, -4, 0)
FARM_ASTRO_TWEEN_TIME  = 0.3
FARM_ASTRO_TIMER_DROP_TIME = 0.35

function NotifyFarmAstroAutoFarm()
    local now = tick()
    if now - FarmAstroTokenLastAutoFarmNotify < 3 then return end
    FarmAstroTokenLastAutoFarmNotify = now
    WindUI:Notify({
        Title = "Farm Astro Token",
        Content = "请先关闭自动刷怪再使用 Farm Astro Token。",
        Duration = 4,
        Icon = "triangle-alert"
    })
end

function NotifyFarmAstroCleanMode()
    local now = tick()
    if now - FarmAstroTokenLastCleanNotify < 5 then return end
    FarmAstroTokenLastCleanNotify = now
    WindUI:Notify({
        Title = "Farm Astro Token",
        Content = "Farm Astro Token 不会击杀怪物，因此清洁模式无法收集物品。请选择 IDGF 模式。",
        Duration = 5,
        Icon = "triangle-alert"
    })
end

function CheckFarmAstroCollectMode()
    if FarmAstroTokenEnabled and AutoCollectEnabled and CollectMode == "清洁" then
        NotifyFarmAstroCleanMode()
        return false
    end
    return true
end

function GetFarmAstroTimerLabel()
    local playerGui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return nil end
    local wavesGui = playerGui:FindFirstChild("WavesGui")
    if not wavesGui then return nil end
    local frame = wavesGui:FindFirstChild("Frame")
    if not frame then return nil end
    return frame:FindFirstChild("Timer")
end

function GetFarmAstroTimerValue()
    local timerLabel = GetFarmAstroTimerLabel()
    if not timerLabel then return nil end
    local textValue = tostring(timerLabel.Text or "")
    local numberText = textValue:match("(%d+)%s*$") or textValue:match("(%d+)")
    if numberText then return tonumber(numberText) end
    return nil
end

function UpdateFarmAstroWaveTimerArmed(timerValue)
    FarmAstroLastWaveTimer = timerValue
    if timerValue ~= nil and timerValue > 10 then
        FarmAstroWaveTimerArmed = true
    end
end

function IsFarmAstroTimerEnding()
    if tick() < FarmAstroTokenTimerIgnoreUntil then return false end
    local timerValue = GetFarmAstroTimerValue()
    UpdateFarmAstroWaveTimerArmed(timerValue)
    return timerValue ~= nil and timerValue <= 10 and FarmAstroWaveTimerArmed == true
end

function IsFarmAstroTimerResetForNextWave()
    local timerValue = GetFarmAstroTimerValue()
    return timerValue ~= nil and timerValue > 10
end

function ShouldKeepFarmAstroFinalLock()
    if not FarmAstroTokenEnabled then return false end
    if FarmAstroFinalLockActive or FarmAstroTokenTimerHold or FarmAstroTimerDropping then return true end
    local timerValue = GetFarmAstroTimerValue()
    return timerValue ~= nil and timerValue <= 3 and FarmAstroWaveTimerArmed == true
end

function HoldFarmAstroBottomLockOnce()
    pcall(function()
        local char, hrp, hum = GetFarmAstroCharacter()
        if not char or not hrp then return end
        if hum then
            hum.Sit = false
            hum.PlatformStand = false
            hum.AutoRotate = true
        end
        char:PivotTo(FARM_ASTRO_TIMER_BOTTOM_CF)
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)
end

function IsFarmAstroGodModeSelected()
    return true
end

function PauseFarmAstroGodModeForTimer()
    if not FarmAstroTokenEnabled then return false end
    if FarmAstroGodModePaused then return true end
    if tick() < FarmAstroTokenTimerIgnoreUntil then return false end

    local timerValue = GetFarmAstroTimerValue()
    UpdateFarmAstroWaveTimerArmed(timerValue)
    if timerValue ~= nil and timerValue <= 10 and FarmAstroWaveTimerArmed == true then
        FarmAstroGodModePaused = true
        GodModeTriggered = false
        CombatDebug("FarmAstroGodSync", "上帝模式百分比在波次计时器 " .. tostring(timerValue) .. " 时暂停", 2, false)
        return true
    end

    return false
end

function ResumeFarmAstroGodModeAfterRespawn(reason)
    local wasPaused = FarmAstroGodModePaused
    FarmAstroGodModePaused = false
    FarmAstroReviveGodTriggered = false
    FarmAstroReviveTimerArmed = false
    FarmAstroLastReviveTimer = nil
    FarmAstroFinalLockActive = false
    FarmAstroTimerDropping = false
    FarmAstroBottomGodTriggered = false
    FarmAstroReviveTimerArmed = false
    FarmAstroLastReviveTimer = nil
    FarmAstroWaveTimerArmed = false
    FarmAstroLastWaveTimer = nil

    if wasPaused then
        CombatDebug("FarmAstroGodSync", "上帝模式在 " .. tostring(reason or "重生") .. " 后恢复", 2, false)
    end
end

function IsFarmAstroReviveState()
    local char, hrp, humanoid = GetFarmAstroCharacter()
    if not char or not hrp or not humanoid then return false end
    if humanoid.Health <= 0 then return false end
    return humanoid.Health <= 1.05
end

function GetFarmAstroReviveTimerLabel()
    if not IsFarmAstroReviveState() then return nil end
    local char, hrp = GetFarmAstroCharacter()
    if not char or not hrp then return nil end
    local reviveUI = hrp:FindFirstChild("ReviveUI")
    if not reviveUI then return nil end
    if reviveUI.Enabled == false then return nil end
    local frame = reviveUI:FindFirstChild("Frame")
    if not frame then return nil end
    if frame:IsA("GuiObject") and frame.Visible == false then return nil end
    local label = frame:FindFirstChild("TextLabel")
    if not label then return nil end
    if label:IsA("GuiObject") and label.Visible == false then return nil end
    return label
end

function GetFarmAstroReviveTimerValue()
    local label = GetFarmAstroReviveTimerLabel()
    if not label then return nil end
    local textValue = tostring(label.Text or "")
    local numberText = textValue:match("^%s*[Tt][Ii][Mm][Ee][Rr]%s*:%s*(%d+)%s*$")
    if numberText then return tonumber(numberText) end
    return nil
end

function UpdateFarmAstroReviveTimerArmed(timerValue)
    FarmAstroLastReviveTimer = timerValue
    if not IsFarmAstroReviveState() then
        FarmAstroReviveTimerArmed = false
        return
    end
    if timerValue ~= nil and timerValue > 5 then
        FarmAstroReviveTimerArmed = true
    end
end

function CheckFarmAstroReviveGodModeOnce()
    if not FarmAstroTokenEnabled then
        FarmAstroReviveGodTriggered = false
        FarmAstroReviveTimerArmed = false
        FarmAstroLastReviveTimer = nil
        return
    end

    local reviveTimer = GetFarmAstroReviveTimerValue()
    UpdateFarmAstroReviveTimerArmed(reviveTimer)

    if reviveTimer == 5 and FarmAstroReviveTimerArmed == true then
        if not FarmAstroReviveGodTriggered then
            if ForceGodModeOnce("Farm Astro 复活计时器") then
                FarmAstroReviveGodTriggered = true
                FarmAstroReviveTimerArmed = false
            end
        end
    elseif reviveTimer == nil then
        FarmAstroReviveGodTriggered = false
        FarmAstroReviveTimerArmed = false
        FarmAstroLastReviveTimer = nil
    elseif reviveTimer > 5 then
        FarmAstroReviveGodTriggered = false
    end
end

function CheckFarmAstroBottomGodMode()
    if not FarmAstroTokenEnabled then return end
    if not FarmAstroFinalLockActive then return end
    if FarmAstroBottomGodTriggered then return end

    local reviveTimer = GetFarmAstroReviveTimerValue()
    UpdateFarmAstroReviveTimerArmed(reviveTimer)

    if reviveTimer == 5 and FarmAstroReviveTimerArmed == true then
        if ForceGodModeOnce("Farm Astro 底部锁定复活计时器") then
            FarmAstroBottomGodTriggered = true
            FarmAstroReviveGodTriggered = true
            FarmAstroReviveTimerArmed = false
        end
    elseif reviveTimer == nil then
        FarmAstroBottomGodTriggered = false
        FarmAstroReviveTimerArmed = false
        FarmAstroLastReviveTimer = nil
    elseif reviveTimer > 5 then
        FarmAstroBottomGodTriggered = false
    end
end

function FarmAstroRuntimeChecks()
    if not FarmAstroTokenEnabled then return end
    PauseFarmAstroGodModeForTimer()
    CheckFarmAstroReviveGodModeOnce()
    CheckFarmAstroBottomGodMode()
end

function GetFarmAstroCharacter()
    local char = LocalPlayer.Character or Character
    if (not char or not char.Parent) and workspace:FindFirstChild("Living") then
        char = workspace.Living:FindFirstChild(LocalPlayer.Name) or workspace.Living:FindFirstChild(LocalPlayer.DisplayName)
    end
    if char and char ~= Character then Character = char end
    if char and (not HumanoidRootPart or HumanoidRootPart.Parent ~= char) then
        HumanoidRootPart = char:FindFirstChild("HumanoidRootPart")
    end
    return char, HumanoidRootPart, char and (char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid"))
end

function CreateFarmAstroTokenPart()
    if FarmAstroTokenPart and FarmAstroTokenPart.Parent then return FarmAstroTokenPart end

    local part = Instance.new("Part")
    part.Name = "farm_astro_token"
    part.Size = Vector3.new(10, 1, 10)
    part.Anchored = true
    part.CanCollide = true
    part.CanTouch = false
    part.CanQuery = false
    part.Material = Enum.Material.Neon
    part.Transparency = 1
    part.CFrame = FARM_ASTRO_TOP_A
    part.Parent = workspace

    for _, face in ipairs(Enum.NormalId:GetEnumItems()) do
        local decal = Instance.new("Decal")
        decal.Name = "farm_astro_token_image"
        decal.Texture = FARM_ASTRO_TOKEN_IMAGE
        decal.Face = face
        decal.Transparency = 0
        decal.Parent = part
    end

    FarmAstroTokenPart = part
    return part
end

function FarmAstroSnapCharacterToPart()
    if not FarmAstroTokenPart or FarmAstroTokenPauseCollect then return end
    pcall(function()
        local char, hrp, hum = GetFarmAstroCharacter()
        if not char or not hrp then return end
        if hum then
            hum.Sit = false
            hum.PlatformStand = false
            hum.AutoRotate = true
        end
        char:PivotTo(FarmAstroTokenPart.CFrame * CFrame.new(0, 4, 0))
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)
end

function CancelFarmAstroTween()
    if FarmAstroTokenTween then
        pcall(function() FarmAstroTokenTween:Cancel() end)
        FarmAstroTokenTween = nil
    end
end

function MoveFarmAstroToTimerSafe()
    if FarmAstroFinalLockActive then return end

    CancelFarmAstroTween()
    CreateFarmAstroTokenPart()

    FarmAstroTokenTimerHold = false
    FarmAstroTimerDropping = true
    FarmAstroFinalLockActive = false
    FarmAstroBottomGodTriggered = false
    FarmAstroReviveTimerArmed = false
    FarmAstroLastReviveTimer = nil
    FarmAstroWaveTimerArmed = false
    FarmAstroLastWaveTimer = nil

    pcall(function()
        if FarmAstroTokenPart and FarmAstroTokenPart.Parent then
            FarmAstroTokenPart.CFrame = FARM_ASTRO_TIMER_BOTTOM_CF * FARM_ASTRO_TIMER_PART_OFFSET
        end
    end)

    pcall(function()
        local char, hrp, hum = GetFarmAstroCharacter()
        if not char or not hrp then return end
        if hum then
            hum.Sit = false
            hum.PlatformStand = false
            hum.AutoRotate = true
        end

        char:PivotTo(FARM_ASTRO_TIMER_TOP_CF)
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)

    pcall(function()
        local char, hrp, hum = GetFarmAstroCharacter()
        if not char or not hrp then return end
        local tween = TweenService:Create(
            hrp,
            TweenInfo.new(FARM_ASTRO_TIMER_DROP_TIME, Enum.EasingStyle.Linear, Enum.EasingDirection.Out),
            { CFrame = FARM_ASTRO_TIMER_BOTTOM_CF }
        )
        tween:Play()
        WaitTweenWithTimeout(tween, (FARM_ASTRO_TIMER_DROP_TIME or 0.35) + 0.45)
        if hum then
            hum.Sit = false
            hum.PlatformStand = false
            hum.AutoRotate = true
        end
        char:PivotTo(FARM_ASTRO_TIMER_BOTTOM_CF)
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
    end)

    FarmAstroTimerDropping = false
    FarmAstroTokenTimerHold = true
    FarmAstroFinalLockActive = true
    CheckFarmAstroBottomGodMode()
end

function WaitFarmAstroRespawnAfterTimer()
    MoveFarmAstroToTimerSafe()
    local lockStartedAt = tick()

    while FarmAstroTokenEnabled do
        FarmAstroRuntimeChecks()
        if FarmAstroFinalLockActive or FarmAstroTokenTimerHold then
            HoldFarmAstroBottomLockOnce()
        end

        if tick() - lockStartedAt >= 0.25 and IsFarmAstroTimerResetForNextWave() then
            break
        end

        task.wait(0.1)
    end

    FarmAstroTokenTimerHold = false
    FarmAstroFinalLockActive = false
    FarmAstroTimerDropping = false
    FarmAstroBottomGodTriggered = false
    FarmAstroReviveGodTriggered = false
    FarmAstroReviveTimerArmed = false
    FarmAstroLastReviveTimer = nil
    FarmAstroWaveTimerArmed = false
    FarmAstroLastWaveTimer = nil
    FarmAstroTokenTimerIgnoreUntil = tick() + 2
    ResumeFarmAstroGodModeAfterRespawn("Farm Astro 计时器重置")
end

FarmAstroNoClipParts = FarmAstroNoClipParts or {}
FarmAstroNoClipChar = nil
FarmAstroNoClipPartsAt = 0

function RebuildFarmAstroNoClipParts(char)
    FarmAstroNoClipParts = {}
    FarmAstroNoClipChar = char
    FarmAstroNoClipPartsAt = tick()
    if not char then return end

    pcall(function()
        for _, obj in ipairs(char:GetDescendants()) do
            if obj:IsA("BasePart") then
                table.insert(FarmAstroNoClipParts, obj)
            end
        end
    end)
end

function ApplyFarmAstroNoClipToCharacter(char)
    if not char then return end
    if FarmAstroNoClipChar ~= char or tick() - (FarmAstroNoClipPartsAt or 0) > 1.25 then
        RebuildFarmAstroNoClipParts(char)
    end

    for i = #FarmAstroNoClipParts, 1, -1 do
        local obj = FarmAstroNoClipParts[i]
        if obj and obj.Parent then
            obj.CanCollide = false
        else
            table.remove(FarmAstroNoClipParts, i)
        end
    end
end

function StartFarmAstroNoClip()
    if FarmAstroTokenNoClipConnection then return end
    FarmAstroTokenNoClipConnection = RunService.Heartbeat:Connect(function()
        if not FarmAstroTokenEnabled then return end
        pcall(function()
            local char, hrp, hum = GetFarmAstroCharacter()
            if not char then return end
            ApplyFarmAstroNoClipToCharacter(char)
            if hum then
                hum.Sit = false
                hum.PlatformStand = false
            end
            if not FarmAstroTokenPauseCollect and hrp then
                if FarmAstroTimerDropping then
                    hrp.AssemblyLinearVelocity = Vector3.zero
                    hrp.AssemblyAngularVelocity = Vector3.zero
                elseif FarmAstroFinalLockActive or FarmAstroTokenTimerHold then
                    char:PivotTo(FARM_ASTRO_TIMER_BOTTOM_CF)
                    hrp.AssemblyLinearVelocity = Vector3.zero
                    hrp.AssemblyAngularVelocity = Vector3.zero
                elseif FarmAstroTokenPart and FarmAstroTokenPart.Parent then
                    char:PivotTo(FarmAstroTokenPart.CFrame * CFrame.new(0, 4, 0))
                    hrp.AssemblyLinearVelocity = Vector3.zero
                    hrp.AssemblyAngularVelocity = Vector3.zero
                end
            end
        end)
    end)
end

function StopFarmAstroNoClip()
    if FarmAstroTokenNoClipConnection then
        FarmAstroTokenNoClipConnection:Disconnect()
        FarmAstroTokenNoClipConnection = nil
    end
end

function SetFarmAstroCollectPause(state)
    FarmAstroTokenPauseCollect = state == true
    CancelFarmAstroTween()
end

function TweenFarmAstroTokenTo(cf, duration)
    if not FarmAstroTokenPart or not FarmAstroTokenPart.Parent then return false end
    FarmAstroRuntimeChecks()
    if IsFarmAstroTimerEnding() then
        MoveFarmAstroToTimerSafe()
        return "timer_end"
    end
    CancelFarmAstroTween()

    FarmAstroTokenTween = TweenService:Create(
        FarmAstroTokenPart,
        TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out),
        { CFrame = cf }
    )
    FarmAstroTokenTween:Play()

    while FarmAstroTokenEnabled do
        FarmAstroRuntimeChecks()
        if IsFarmAstroTimerEnding() then
            MoveFarmAstroToTimerSafe()
            return "timer_end"
        end
        if FarmAstroTokenPauseCollect then
            CancelFarmAstroTween()
            return true
        end
        if not FarmAstroTokenTween or FarmAstroTokenTween.PlaybackState ~= Enum.PlaybackState.Playing then break end
        task.wait(0.05)
    end

    if not FarmAstroTokenEnabled then
        CancelFarmAstroTween()
        return false
    end

    FarmAstroTokenTween = nil
    pcall(function() FarmAstroTokenPart.CFrame = cf end)
    return true
end

function StartFarmAstroToken()
    if FarmAstroTokenRunning then return end
    if AutoFarmEnabled then
        FarmAstroTokenEnabled = false
        Config:Set("FarmAstroTokenEnabled", false)
        Config:Save()
        NotifyFarmAstroAutoFarm()
        return
    end

    FarmAstroTokenRunning = true
    NeedNoClip = true
    LockActive = false
    AutoAttackEnabled = false
    AutoSkillEnabled = false
    FarmAstroTokenTimerHold = false
    FarmAstroWaveTimerArmed = false
    FarmAstroLastWaveTimer = nil
    CreateFarmAstroTokenPart()
    StartFarmAstroNoClip()
    CheckFarmAstroCollectMode()

    task.spawn(function()
        while FarmAstroTokenEnabled do
            if FarmAstroTokenPauseCollect then
                repeat task.wait(0.2) until not FarmAstroTokenPauseCollect or not FarmAstroTokenEnabled
            end
            if not FarmAstroTokenEnabled then break end

            FarmAstroRuntimeChecks()

            if FarmAstroFinalLockActive or FarmAstroTokenTimerHold then
                WaitFarmAstroRespawnAfterTimer()
                continue
            end

            if IsFarmAstroTimerEnding() then
                WaitFarmAstroRespawnAfterTimer()
                continue
            end

            CreateFarmAstroTokenPart()
            FarmAstroTokenPart.CFrame = FARM_ASTRO_TOP_A
            FarmAstroSnapCharacterToPart()

            local topResult = TweenFarmAstroTokenTo(FARM_ASTRO_TOP_B, FARM_ASTRO_TWEEN_TIME)
            if topResult == "timer_end" then
                WaitFarmAstroRespawnAfterTimer()
                continue
            end
            if not topResult then break end

            if FarmAstroTokenPauseCollect then continue end
            if IsFarmAstroTimerEnding() then
                WaitFarmAstroRespawnAfterTimer()
                continue
            end

            FarmAstroTokenPart.CFrame = FARM_ASTRO_LOW_A
            FarmAstroSnapCharacterToPart()

            local lowResult = TweenFarmAstroTokenTo(FARM_ASTRO_LOW_B, FARM_ASTRO_TWEEN_TIME)
            if lowResult == "timer_end" then
                WaitFarmAstroRespawnAfterTimer()
                continue
            end
            if not lowResult then break end
        end

        CancelFarmAstroTween()
        StopFarmAstroNoClip()
        if FarmAstroTokenPart then
            pcall(function() FarmAstroTokenPart:Destroy() end)
        end
        FarmAstroTokenPart = nil
        FarmAstroTokenPauseCollect = false
        FarmAstroTokenTimerHold = false
        FarmAstroFinalLockActive = false
        FarmAstroTimerDropping = false
        FarmAstroBottomGodTriggered = false
        FarmAstroReviveGodTriggered = false
        FarmAstroWaveTimerArmed = false
        FarmAstroLastWaveTimer = nil
        FarmAstroTokenRunning = false
        RestoreFarmCameraAndMovement()
        ResumeFarmAstroGodModeAfterRespawn("Farm Astro 停止")
    end)
end

function StopFarmAstroToken(saveState)
    FarmAstroTokenEnabled = false
    FarmAstroTokenTimerHold = false
    FarmAstroFinalLockActive = false
    FarmAstroTimerDropping = false
    FarmAstroBottomGodTriggered = false
    FarmAstroReviveGodTriggered = false
    FarmAstroReviveTimerArmed = false
    FarmAstroLastReviveTimer = nil
    FarmAstroWaveTimerArmed = false
    FarmAstroLastWaveTimer = nil
    ResumeFarmAstroGodModeAfterRespawn("Farm Astro 已禁用")
    if saveState then
        Config:Set("FarmAstroTokenEnabled", false)
        Config:Save()
    end
    CancelFarmAstroTween()
end

task.spawn(function()
    while true do
        task.wait(0.1)
        if FarmAstroTokenEnabled then
            FarmAstroRuntimeChecks()
        else
            FarmAstroReviveGodTriggered = false
            FarmAstroBottomGodTriggered = false
            FarmAstroWaveTimerArmed = false
            FarmAstroLastWaveTimer = nil
        end
    end
end)
-- ============================================================
-- ====================== UI: MAIN TAB ======================
-- ============================================================

Main:Section({ Title = "自动刷怪", Icon = "package" })

AutoFarmToggle = Main:Toggle({
    Title = "自动刷怪",
    Desc = "基于优先级系统自动刷怪。",
    Value = AutoFarmEnabled,
    Callback = function(state)
        AutoFarmEnabled = state
        UpdateYYAWaitingPartCollision()
        if state then
            if EnhancedNormalEnabled then
                LoadEnhancedLoop("Normal")
            elseif EnhancedDarkDimensionEnabled then
                LoadEnhancedLoop("DarkDimension")
            elseif EnhancedAstroEnabled then
                LoadEnhancedLoop("Astro")
            end
            if type(StartFarmLoop) == "function" then
                StartFarmLoop()
                StartAutoAttack()
                StartAutoFillUpLoop()
            end
            WindUI:Notify({ Title = "自动刷怪", Content = "已启用，自动刷怪已启动！", Duration = 2, Icon = "play" })
        else
            FarmLoopRunning = false
            WaitingRespawn = false
            LockActive = false
            RestoreFarmCameraAndMovement()
            UpdateYYAWaitingPartCollision()
            WindUI:Notify({ Title = "自动刷怪", Content = "自动刷怪已关闭。", Duration = 2, Icon = "square" })
        end
        Config:Set("AutoFarmEnabled", state)
        Config:Save()
    end
})

Main:Divider()

Main:Section({ Title = "增强循环", Icon = "cpu" })

EnhancedNormalEnabled = Config:Get("EnhancedNormalEnabled", false)

NormalToggle = Main:Toggle({
    Title = "普通模式增强循环",
    Desc = "使用独立版高效循环（优先级选怪、传送/补间、无怪循环移动）",
    Value = EnhancedNormalEnabled,
    Callback = function(state)
        EnhancedNormalEnabled = state
        Config:Set("EnhancedNormalEnabled", state)
        Config:Save()
        if state then
            if EnhancedDarkDimensionEnabled then
                EnhancedDarkDimensionEnabled = false
                Config:Set("EnhancedDarkDimensionEnabled", false)
                pcall(function()
                    if DarkDimensionToggle and DarkDimensionToggle.Set then
                        DarkDimensionToggle:Set(false)
                    end
                end)
            end
            if EnhancedAstroEnabled then
                EnhancedAstroEnabled = false
                Config:Set("EnhancedAstroEnabled", false)
                pcall(function()
                    if AstroToggle and AstroToggle.Set then
                        AstroToggle:Set(false)
                    end
                end)
            end
            LoadEnhancedLoop("Normal")
            if AutoFarmEnabled then
                StartFarmLoop()
                StartAutoAttack()
                StartAutoFillUpLoop()
            end
            WindUI:Notify({ Title = "增强循环", Content = "普通模式增强已启用", Duration = 2, Icon = "check" })
        else
            WindUI:Notify({ Title = "增强循环", Content = "普通模式增强已禁用", Duration = 2, Icon = "square" })
        end
    end
})

Main:Slider({
    Title = "普通模式重置波数",
    Desc = "仅普通模式生效，到达设定波数时传送至重置点（默认10波）",
    Value = { Min = 1, Max = 25, Default = ResetWaveValue },
    Step = 1,
    Callback = function(value)
        ResetWaveValue = value
        Config:Set("ResetWaveValue", value)
        Config:Save()
    end
})

Main:Divider()

EnhancedDarkDimensionEnabled = Config:Get("EnhancedDarkDimensionEnabled", false)

DarkDimensionToggle = Main:Toggle({
    Title = "黑暗模式增强循环",
    Desc = "包含理智值收集、强制Jeffrey躲避、第5波+计时器≤10秒通关停止",
    Value = EnhancedDarkDimensionEnabled,
    Callback = function(state)
        EnhancedDarkDimensionEnabled = state
        Config:Set("EnhancedDarkDimensionEnabled", state)
        Config:Save()
        if state then
            if EnhancedNormalEnabled then
                EnhancedNormalEnabled = false
                Config:Set("EnhancedNormalEnabled", false)
                pcall(function()
                    if NormalToggle and NormalToggle.Set then
                        NormalToggle:Set(false)
                    end
                end)
            end
            if EnhancedAstroEnabled then
                EnhancedAstroEnabled = false
                Config:Set("EnhancedAstroEnabled", false)
                pcall(function()
                    if AstroToggle and AstroToggle.Set then
                        AstroToggle:Set(false)
                    end
                end)
            end
            AutoVoteValue = "黑暗维度"
            Config:Set("AutoVoteValue", "黑暗维度")
            LoadEnhancedLoop("DarkDimension")
            if AutoFarmEnabled then
                StartFarmLoop()
                StartAutoAttack()
                StartAutoFillUpLoop()
            end
            WindUI:Notify({ Title = "增强循环", Content = "黑暗模式增强已启用，自动投票黑暗维度", Duration = 3, Icon = "check" })
        else
            WindUI:Notify({ Title = "增强循环", Content = "黑暗模式增强已禁用", Duration = 2, Icon = "square" })
        end
    end
})

Main:Slider({
    Title = "黑暗模式反Jeffrey距离",
    Desc = "检测Jeffrey的距离，到达此距离时触发躲避（黑暗模式强制生效）",
    Value = { Min = 10, Max = 200, Default = AntiJeffreyRange },
    Step = 1,
    Callback = function(value)
        AntiJeffreyRange = value
        Config:Set("AntiJeffreyRange", value)
        Config:Save()
    end
})

Main:Divider()

EnhancedAstroEnabled = Config:Get("EnhancedAstroEnabled", false)

AstroToggle = Main:Toggle({
    Title = "天文模式增强循环",
    Desc = "自动投票AstroV2 + 自动准备，沿用原脚本天文模式逻辑（自动环绕、计时器检测、最终门）",
    Value = EnhancedAstroEnabled,
    Callback = function(state)
        EnhancedAstroEnabled = state
        Config:Set("EnhancedAstroEnabled", state)
        Config:Save()
        if state then
            if EnhancedNormalEnabled then
                EnhancedNormalEnabled = false
                Config:Set("EnhancedNormalEnabled", false)
                pcall(function()
                    if NormalToggle and NormalToggle.Set then
                        NormalToggle:Set(false)
                    end
                end)
            end
            if EnhancedDarkDimensionEnabled then
                EnhancedDarkDimensionEnabled = false
                Config:Set("EnhancedDarkDimensionEnabled", false)
                pcall(function()
                    if DarkDimensionToggle and DarkDimensionToggle.Set then
                        DarkDimensionToggle:Set(false)
                    end
                end)
            end
            AutoVoteValue = "Astro V2"
            Config:Set("AutoVoteValue", "Astro V2")
            LoadEnhancedLoop("Astro")
            if AutoFarmEnabled then
                StartFarmLoop()
                StartAutoAttack()
                StartAutoFillUpLoop()
            end
            WindUI:Notify({ Title = "增强循环", Content = "天文模式增强已启用，自动投票AstroV2", Duration = 3, Icon = "check" })
        else
            WindUI:Notify({ Title = "增强循环", Content = "天文模式增强已禁用", Duration = 2, Icon = "square" })
        end
    end
})

Main:Divider()

Main:Section({ Title = "通用功能", Icon = "settings" })

AutoSkipHeliEnabled = Config:Get("AutoSkipHeliEnabled", false)

Main:Toggle({
    Title = "自动跳过直升机",
    Desc = "所有模式下自动跳过直升机波次",
    Value = AutoSkipHeliEnabled,
    Callback = function(state)
        AutoSkipHeliEnabled = state
        Config:Set("AutoSkipHeliEnabled", state)
        Config:Save()
        if state then
            TriggerAutoSkipHeli(true)
            WindUI:Notify({ Title = "自动跳过直升机", Content = "已启用，所有模式生效", Duration = 2, Icon = "check" })
        else
            TriggerAutoSkipHeli(false)
            WindUI:Notify({ Title = "自动跳过直升机", Content = "已禁用", Duration = 2, Icon = "square" })
        end
    end
})

DeleteMapEnabled = Config:Get("DeleteMapEnabled", false)

Main:Toggle({
    Title = "删除地图",
    Desc = "所有模式下隐藏地图部件提升性能",
    Value = DeleteMapEnabled,
    Callback = function(state)
        DeleteMapEnabled = state
        Config:Set("DeleteMapEnabled", state)
        Config:Save()
        if state then
            SaveAndBoostFPS()
            WindUI:Notify({ Title = "删除地图", Content = "已启用，地图部件已隐藏", Duration = 2, Icon = "check" })
        else
            RestoreBoostFPS()
            WindUI:Notify({ Title = "删除地图", Content = "已禁用，地图部件已恢复", Duration = 2, Icon = "square" })
        end
    end
})

Main:Section({ Title = "全部技能", Icon = "keyboard" })

SkillDropdown = Main:Dropdown({
    Title = "全部技能（按键）",
    Desc = "选择自动技能将按下的键盘技能键。",
    Values = skillDropdownValues,
    Multi = true,
    Value = SelectedSkills,
    Callback = function(values)
        SelectedSkills = values
        Config:Set("SelectedSkills", values)
        Config:Save()
    end
})

Main:Slider({
    Title = "技能延迟（秒）",
    Desc = "设置每个自动技能按键之间的延迟（秒）。",
    Value = { Min = 1, Max = 60, Default = SkillDelay },
    Step = 1,
    Callback = function(value)
        SkillDelay = value
        Config:Set("SkillDelay", value)
        Config:Save()
    end
})
Main:Section({ Title = "刷怪设置", Icon = "settings" })

PositionDropdown = Main:Dropdown({
    Title = "刷怪位置",
    Desc = "选择角色在目标周围的站位。",
    Values = FarmPositionDisplayNames,
    Multi = false,
    Value = GetDisplayName(FarmPositionMap, FarmPosition) or FarmPosition,
    Callback = function(value)
        local english = FarmPositionMap[value] or value
        FarmPosition = english
        Config:Set("FarmPosition", english)
        Config:Save()
    end
})

ModeDropdown = Main:Dropdown({
    Title = "移动方式",
    Desc = "选择角色移动到每个目标的方式。",
    Values = MovementDisplayNames,
    Multi = false,
    Value = GetDisplayName(MovementMap, FarmMode) or FarmMode,
    Callback = function(value)
        local english = MovementMap[value] or value
        FarmMode = english
        Config:Set("FarmMode", english)
        Config:Save()
        WindUI:Notify({ Title = "移动方式", Content = "已选择: " .. tostring(value), Duration = 2, Icon = "mouse-pointer-click" })
    end
})

Main:Slider({
    Title = "刷怪高度（±Y）",
    Desc = "调整在怪物上方或下方刷怪时的垂直偏移。",
    Value = { Min = -150, Max = 150, Default = HeightValue },
    Step = 1,
    Callback = function(value)
        HeightValue = value
        Config:Set("HeightValue", value)
        Config:Save()
        for mob, _ in pairs(MobHeightOverride) do
            if MobConfirmedPadding[mob] == nil then
                MobHeightOverride[mob] = nil
            end
        end
    end
})

Main:Slider({
    Title = "上帝模式血量（%）",
    Desc = "设置上帝模式的血量百分比阈值，低于此值自伤触发复活。所有模式强制启用。",
    Value = { Min = 1, Max = 99, Default = GodModeValue },
    Step = 1,
    Callback = function(value)
        GodModeValue = value
        Config:Set("GodModeValue", value)
        Config:Save()
    end
})

Main:Section({ Title = "优先级设置", Icon = "list-ordered" })

Main:Paragraph({
    Title = "优先级设置",
    Desc = "中断：如果正在攻击低血量怪物且更高血量怪物出现，立即切换目标",
    Image = "rbxassetid://103720636367587",
    ImageSize = 26,
})

Main:Slider({
    Title = "高血量阈值（最大生命值）",
    Desc = "设置怪物成为高血量优先级所需的最大生命值。",
    Value = { Min = 1, Max = 100000, Default = HighHPThreshold },
    Step = 100,
    Callback = function(value)
        HighHPThreshold = value
        Config:Set("HighHPThreshold", value)
        Config:Save()
        print("[YYa] 高血量阈值设置为 " .. value)
    end
})

-- 已删除 BypassJeffreyToggle 及其 Section

Main:Section({ Title = "覆盖设置", Icon = "ruler" })

PaddingReduceInput = Main:Input({
    Title = "设置减少偏移",
    Default = tostring(PADDING_REDUCE_STEP),
    Placeholder = "默认: 2",
    Callback = function(text)
        local num = tonumber(text)
        if num then
            PADDING_REDUCE_STEP = num
            Config:Set("PaddingReduceStep", num)
            Config:Save()
        else
            warn("输入了不正确的数字！")
        end
    end
})

PaddingSafeInput = Main:Input({
    Title = "设置安全最小偏移（全局底线）",
    Default = tostring(PADDING_SAFE_MIN),
    Placeholder = "默认: -30",
    Callback = function(text)
        local num = tonumber(text)
        if num then
            PADDING_SAFE_MIN = num
            Config:Set("PaddingSafeMin", num)
            Config:Save()
        else
            warn("输入了不正确的数字！")
        end
    end
})

Main:Slider({
    Title = "防穿模边距（单位）",
    Desc = "增加额外间距以减少在怪物附近刷怪时的穿模。",
    Value = { Min = -10, Max = 10, Default = ANTI_CLIP_MARGIN },
    Step = 1,
    Callback = function(value)
        ANTI_CLIP_MARGIN = value
        Config:Set("AntiClipMargin", value)
        Config:Save()
    end
})

Main:Slider({
    Title = "伤害阈值（确认锁定）",
    Desc = "设置确认当前刷怪位置有效的伤害量。",
    Value = { Min = 1, Max = 500, Default = DMG_THRESHOLD },
    Step = 1,
    Callback = function(value)
        DMG_THRESHOLD = value
        Config:Set("DmgThreshold", value)
        Config:Save()
    end
})

Main:Button({
    Title = "重置所有已确认位置",
    Desc = "清除所有已保存的怪物高度位置并恢复默认。",
    Callback = function()
        MobConfirmedPadding = {}
        MobHeightOverride   = {}
        WindUI:Notify({ Title = "覆盖设置", Content = "所有已确认的怪物位置已清除。", Duration = 2, Icon = "refresh-cw" })
    end
})

Main:Section({ Title = "冲水设置", Icon = "toilet" })

Flushaura      = Config:Get("flushaura", false)
FlushAuraValue = Config:Get("FlushAuraValue", 5)

Main:Slider({
    Title = "冲水光环（单位）",
    Desc = "设置冲水光环激活附近提示的距离。",
    Value = { Min = 1, Max = 15, Default = FlushAuraValue },
    Step = 1,
    Callback = function(value)
        FlushAuraValue = value
        Config:Set("FlushAuraValue", value)
        Config:Save()
    end
})

Main:Toggle({
    Title = "冲水光环",
    Desc = "在设定半径内自动冲水附近的冲水提示。",
    Value = Flushaura,
    Callback = function(enabled)
        Flushaura = enabled
        Config:Set("flushaura", enabled)
        Config:Save()
        if enabled then
            task.spawn(function()
                while Flushaura do
                    pcall(function()
                        local char = game.Players.LocalPlayer.Character
                        if not char then return end
                        local root = char:FindFirstChild("HumanoidRootPart")
                        if not root then return end
                        if FlushPromptCacheDirty or tick() - (FlushPromptCacheLastScan or 0) > (FlushPromptCacheTTL or 8) then
                            RebuildFlushPromptCache()
                        end
                        for prompt in pairs(FlushPromptCache) do
                            if prompt and prompt.Parent and IsFlushPrompt(prompt) then
                                local parent = prompt.Parent
                                local part = parent:IsA("BasePart") and parent or parent:FindFirstAncestorWhichIsA("BasePart")
                                if part and (root.Position - part.Position).Magnitude <= FlushAuraValue then
                                    ActivateProximityPrompt(prompt)
                                end
                            else
                                FlushPromptCache[prompt] = nil
                            end
                        end
                    end)
                    task.wait(0.25)
                end
            end)
        end
    end
})
-- ============================================================
-- ====================== ENHANCED LOOP DEFINITIONS ============
-- ============================================================

function LoadEnhancedLoop(mode)
    print("[YYa] 加载增强循环模式:", mode)

    if mode == "Normal" then
        StartFarmLoop = function()
            if FarmLoopRunning then return end
            FarmLoopRunning = true
            task.spawn(function()
                while AutoFarmEnabled and EnhancedNormalEnabled do
                    if not Character or not Character.Parent then
                        Character = LocalPlayer.Character
                        if Character then
                            HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
                        end
                        if not HumanoidRootPart then
                            task.wait(1)
                            continue
                        end
                    end

                    if ResetWaveEnabled then
                        local currentWave = GetCurrentResetWave()
                        if currentWave and currentWave >= ResetWaveValue then
                            if not ResetWaveTeleporting then
                                TeleportResetWave(currentWave, ResetWaveValue, false, "普通模式重置波数")
                            end
                            task.wait(0.5)
                            continue
                        end
                    end

                    local mob, mobType, extraData, priority = GetPriorityMob()

                    if mob then
                        if mobType == "GiantST" and extraData then
                            TeleportToMob(mob)
                            pcall(function()
                                extraData.HoldDuration = 0
                                extraData.MaxActivationDistance = 50
                                if fireproximityprompt then fireproximityprompt(extraData) end
                                extraData:InputHoldBegin()
                                task.wait(0.03)
                                extraData:InputHoldEnd()
                            end)
                            repeat
                                task.wait(0.2)
                                local interrupt, _ = CheckInterrupt(priority)
                                if interrupt then
                                    FarmForceRetarget = true
                                    break
                                end
                            until IsMobDead(mob) or not AutoFarmEnabled or not EnhancedNormalEnabled
                        else
                            TeleportToMob(mob)
                            LockActive = true
                            local lockConn
                            lockConn = RunService.Heartbeat:Connect(function()
                                if not AutoFarmEnabled or IsMobDead(mob) or FarmForceRetarget or not LockActive or not EnhancedNormalEnabled then
                                    if lockConn then lockConn:Disconnect() end
                                    LockActive = false
                                    return
                                end
                                local cf = GetTargetCFrame(mob, FarmPosition)
                                if cf and Character and HumanoidRootPart then
                                    MoveCharacterToFarmCFrame(cf)
                                end
                            end)
                            repeat
                                task.wait(0.15)
                                local interrupt, _ = CheckInterrupt(priority)
                                if interrupt then
                                    FarmForceRetarget = true
                                    break
                                end
                            until IsMobDead(mob) or not AutoFarmEnabled or not EnhancedNormalEnabled
                            if lockConn then lockConn:Disconnect() end
                            LockActive = false
                        end
                    else
                        TeleportToIdle()
                        task.wait(0.3)
                    end

                    task.wait(0.05)
                end
                FarmLoopRunning = false
            end)
        end

    elseif mode == "DarkDimension" then
        StartFarmLoop = function()
            if FarmLoopRunning then return end
            FarmLoopRunning = true
            EnhancedDarkDimensionEnabled = true
            task.spawn(function()
                while AutoFarmEnabled and EnhancedDarkDimensionEnabled do
                    if not Character or not Character.Parent then
                        Character = LocalPlayer.Character
                        if Character then
                            HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
                        end
                        if not HumanoidRootPart then
                            task.wait(1)
                            continue
                        end
                    end

                    if HandleDarkDimensionSanity() then
                        task.wait(0.1)
                        continue
                    end

                    local currentWave = GetCurrentResetWave()
                    local timerValue = GetFarmAstroTimerValue()
                    if currentWave and currentWave >= 5 and timerValue and timerValue <= 10 then
                        print("[YYa] 黑暗模式通关！波次:", currentWave, "计时器:", timerValue)
                        WindUI:Notify({ Title = "黑暗模式通关", Content = "已通关！", Duration = 5, Icon = "trophy" })
                        AutoFarmEnabled = false
                        AutoAttackEnabled = false
                        AutoSkillEnabled = false
                        LockActive = false
                        FarmLoopRunning = false
                        while true do
                            TeleportToIdle(true)
                            task.wait(0.5)
                            local lobby = LocalPlayer.PlayerGui:FindFirstChild("Lobby")
                            if lobby and lobby.Enabled then
                                print("[YYa] 检测到回到大厅，关闭脚本")
                                AutoFarmEnabled = false
                                AutoAttackEnabled = false
                                AutoSkillEnabled = false
                                FarmLoopRunning = false
                                LockActive = false
                                if AutoStartConnection then AutoStartConnection:Disconnect() end
                                if noBarrierConnection then noBarrierConnection:Disconnect() end
                                if ESPConnection then ESPConnection:Disconnect() end
                                if BoostFPS_Active then RestoreBoostFPS() end
                                print("[YYa] 脚本已完全关闭，请重新注入以再次使用")
                                return
                            end
                            task.wait(0.5)
                        end
                    end

                    local mob, mobType, extraData, priority = GetPriorityMob()

                    if mob then
                        if IsMobBlockedByJeffrey(mob, AntiJeffreyRange) then
                            HandleFarmJeffreyEmergency(mob)
                            task.wait(0.2)
                            continue
                        end

                        if mobType == "GiantST" and extraData then
                            TeleportToMob(mob)
                            pcall(function()
                                extraData.HoldDuration = 0
                                extraData.MaxActivationDistance = 50
                                if fireproximityprompt then fireproximityprompt(extraData) end
                                extraData:InputHoldBegin()
                                task.wait(0.03)
                                extraData:InputHoldEnd()
                            end)
                            repeat
                                task.wait(0.2)
                                local interrupt, _ = CheckInterrupt(priority)
                                if interrupt then
                                    FarmForceRetarget = true
                                    break
                                end
                            until IsMobDead(mob) or not AutoFarmEnabled or not EnhancedDarkDimensionEnabled
                        else
                            TeleportToMob(mob)
                            LockActive = true
                            local lockConn
                            lockConn = RunService.Heartbeat:Connect(function()
                                if not AutoFarmEnabled or IsMobDead(mob) or FarmForceRetarget or not LockActive or not EnhancedDarkDimensionEnabled then
                                    if lockConn then lockConn:Disconnect() end
                                    LockActive = false
                                    return
                                end
                                if IsMobBlockedByJeffrey(mob, AntiJeffreyRange) then
                                    HandleFarmJeffreyEmergency(mob)
                                    if lockConn then lockConn:Disconnect() end
                                    LockActive = false
                                    return
                                end
                                local cf = GetTargetCFrame(mob, FarmPosition)
                                if cf and Character and HumanoidRootPart then
                                    MoveCharacterToFarmCFrame(cf)
                                end
                            end)
                            repeat
                                task.wait(0.15)
                                local interrupt, _ = CheckInterrupt(priority)
                                if interrupt then
                                    FarmForceRetarget = true
                                    break
                                end
                            until IsMobDead(mob) or not AutoFarmEnabled or not EnhancedDarkDimensionEnabled
                            if lockConn then lockConn:Disconnect() end
                            LockActive = false
                        end
                    else
                        TeleportToIdle()
                        task.wait(0.3)
                    end

                    task.wait(0.05)
                end
                FarmLoopRunning = false
            end)
        end

    elseif mode == "Astro" then
        StartFarmLoop = function()
            if FarmLoopRunning then return end
            FarmLoopRunning = true
            task.spawn(function()
                while AutoFarmEnabled and EnhancedAstroEnabled do
                    if not Character or not Character.Parent then
                        Character = LocalPlayer.Character
                        if Character then
                            HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
                        end
                        if not HumanoidRootPart then
                            task.wait(1)
                            continue
                        end
                    end

                    local mob = nil
                    for _, candidate in ipairs(GetCachedLivingMobs()) do
                        if IsAstroMob(candidate) then
                            mob = candidate
                            break
                        end
                    end

                    if mob then
                        TeleportToMob(mob)
                        LockActive = true
                        local lockConn
                        lockConn = RunService.Heartbeat:Connect(function()
                            if not AutoFarmEnabled or IsMobDead(mob) or FarmForceRetarget or not LockActive or not EnhancedAstroEnabled then
                                if lockConn then lockConn:Disconnect() end
                                LockActive = false
                                return
                            end
                            local cf = GetTargetCFrame(mob, FarmPosition)
                            if cf and Character and HumanoidRootPart then
                                MoveCharacterToFarmCFrame(cf)
                            end
                        end)
                        repeat
                            task.wait(0.15)
                            local timer = GetFarmAstroTimerValue()
                            if timer and timer <= 10 then
                                DoAstroModeFinalDoor()
                                break
                            end
                            local hasAstro = false
                            for _, c in ipairs(GetCachedLivingMobs()) do
                                if IsAstroMob(c) then
                                    hasAstro = true
                                    break
                                end
                            end
                            if not hasAstro then
                                DoAstroModeFinalDoor()
                                break
                            end
                        until IsMobDead(mob) or not AutoFarmEnabled or not EnhancedAstroEnabled
                        if lockConn then lockConn:Disconnect() end
                        LockActive = false
                    else
                        local timer = GetFarmAstroTimerValue()
                        if timer and timer <= 10 then
                            DoAstroModeFinalDoor()
                        else
                            TeleportToIdle()
                        end
                        task.wait(0.3)
                    end

                    task.wait(0.05)
                end
                FarmLoopRunning = false
            end)
        end
    end

    StartAutoAttack = function()
        if AutoAttackLoopRunning then return end
        AutoAttackLoopRunning = true
        task.spawn(function()
            while AutoAttackEnabled do
                local mob = GetPriorityMob()
                if mob then
                    local remote = GetRemote("LMB")
                    if remote then
                        pcall(function() remote:FireServer() end)
                    end
                end
                task.wait(0.12)
            end
            AutoAttackLoopRunning = false
        end)
    end

    print("[YYa] 增强循环加载完成，模式:", mode)
end

-- ============================================================
-- ====================== RESET WAVE FUNCTIONS ================
-- ============================================================
function GetResetWaveLabel()
    local playerGui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return nil end
    local wavesGui = playerGui:FindFirstChild("WavesGui")
    if not wavesGui then return nil end
    local frame = wavesGui:FindFirstChild("Frame")
    if not frame then return nil end
    return frame:FindFirstChild("TextLabel")
end

function GetCurrentResetWave()
    local label = GetResetWaveLabel()
    if not label then return nil end
    local ok, textValue = pcall(function() return tostring(label.Text or "") end)
    if not ok then return nil end
    local waveText = textValue:match("[Ww]ave%s*=?%s*(%d+)")
    if not waveText then waveText = textValue:match("(%d+)") end
    return tonumber(waveText)
end

function GetResetWaveTargetValue()
    local value = tonumber(ResetWaveValue) or 10
    value = math.floor(value)
    if value < 1 then value = 1 end
    return value
end

function GetResetWaveTriggerKey(currentWave, targetWave)
    return tostring(tonumber(currentWave) or "nil") .. ":" .. tostring(tonumber(targetWave) or "nil")
end

function ClearResetWaveTrigger(reason)
    ResetWaveLastTriggeredWave = nil
    ResetWaveLastTriggeredKey = nil
    CombatDebug("ResetWave", "触发已清除: " .. tostring(reason or "重置"), 3, false)
end

function IsResetWaveCharacterReady()
    RefreshCombatCharacter()
    if not Character or not Character.Parent or not HumanoidRootPart or not HumanoidRootPart.Parent then return false end
    local humanoid = Character:FindFirstChildOfClass("Humanoid") or Character:FindFirstChild("Humanoid")
    if humanoid and humanoid.Health <= 0 then return false end
    return true
end

function BreakFarmLockForResetWave()
    ResetWaveTeleporting = true
    FarmForceRetarget = true
    FarmCollecting = false
    LockActive = false
    _interruptSignal = true
    WaitingRespawn = false
    _currentTargetPriority = 0

    pcall(function()
        RefreshCombatCharacter()
        if Character then
            local humanoid = Character:FindFirstChildOfClass("Humanoid") or Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.Sit = false
                humanoid.PlatformStand = false
                humanoid.AutoRotate = true
            end
        end
        if HumanoidRootPart then
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
    end)

    pcall(function() RunService.Heartbeat:Wait() end)
end

function HoldResetWavePosition(token)
    local holdUntil = tick() + (ResetWaveHoldTime or 2)

    while ResetWaveEnabled and ResetWaveTeleporting and token == ResetWaveToken and tick() < holdUntil do
        if not AutoFarmEnabled then return false end
        if not IsResetWaveCharacterReady() then return false end

        pcall(function()
            local humanoid = Character:FindFirstChildOfClass("Humanoid") or Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.Sit = false
                humanoid.PlatformStand = false
                humanoid.AutoRotate = true
            end

            NeedNoClip = true
            Character:PivotTo(ResetWaveTargetCF)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
            StabilizeFarmCamera()
        end)

        task.wait(0.1)
    end

    return ResetWaveEnabled and token == ResetWaveToken and IsResetWaveCharacterReady()
end

function FinishResetWaveTeleport(token, completed, currentWave, targetWave)
    if token ~= ResetWaveToken then return end

    ResetWaveTeleporting = false

    if completed then
        ResetWaveLastTriggeredWave = currentWave
        ResetWaveLastTriggeredKey = GetResetWaveTriggerKey(currentWave, targetWave)
        CombatDebug("ResetWave", "已保持重置点 " .. tostring(ResetWaveHoldTime or 2) .. " 秒，波次 " .. tostring(currentWave), 2, false)
    else
        ClearResetWaveTrigger("传送中断")
    end

    FarmForceRetarget = false
    _interruptSignal = false
    LockActive = false

    if completed and ResetWaveEnabled and AutoFarmEnabled then
        task.defer(function()
            if ResetWaveEnabled and AutoFarmEnabled and not ResetWaveTeleporting then
                if EnhancedNormalEnabled then
                    LoadEnhancedLoop("Normal")
                    StartFarmLoop()
                end
            end
        end)
    end
end

function TeleportResetWave(currentWave, targetWave, force, reason)
    if ResetWaveTeleporting then return false end

    local now = tick()
    if not force and now - (ResetWaveLastTeleportAt or 0) < 0.6 then return false end
    ResetWaveLastTeleportAt = now

    currentWave = tonumber(currentWave) or GetCurrentResetWave()
    targetWave = tonumber(targetWave) or GetResetWaveTargetValue()
    if not currentWave or currentWave < targetWave then return false end

    ResetWaveToken = (ResetWaveToken or 0) + 1
    local token = ResetWaveToken

    BreakFarmLockForResetWave()

    local ok, completed = pcall(function()
        if not IsResetWaveCharacterReady() then return false end

        pcall(function()
            NeedNoClip = true
            Character:PivotTo(ResetWaveTargetCF)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end)

        return HoldResetWavePosition(token)
    end)

    FinishResetWaveTeleport(token, ok and completed == true, currentWave, targetWave)
    return ok and completed == true
end
-- ============================================================
-- ====================== MAIN7: GAME MODE TAB ================
-- ============================================================
Main7:Section({ Title = "投票信息", TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = "投票信息",
    Desc = "- [步骤 1] 点击恢复投票系统\n- [步骤 2] 在大厅中（游戏内）等待\n- [步骤 3] 设置自动投票并等待",
    Image = "rbxassetid://103720636367587",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = "投票系统", Icon = "gamepad-2" })

Main7:Button({
    Title = "恢复投票系统",
    Desc = "⚠️ 首次使用自动投票模式前按一次。",
    Callback = function()
        pcall(function()
            ReplicatedStorage.GetReadyRemote:FireServer("1", true)
            task.wait(0.5)
            ReplicatedStorage.GetReadyRemote:FireServer("1", false)
            task.wait(0.5)
            ReplicatedStorage.GetReadyRemote:FireServer("2", false)
            task.wait(0.5)
            ReplicatedStorage.GetReadyRemote:FireServer("3", false)
            task.wait(0.5)
            ReplicatedStorage.GetReadyRemote:FireServer("1", true)
        end)
        WindUI:Notify({
            Title = "恢复投票系统",
            Content = "准备中，恢复投票系统...",
            Duration = 6,
            Icon = "loader-circle"
        })
        task.wait(6)
        pcall(function()
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                char.HumanoidRootPart.CFrame = CFrame.new(-220, -10, -600)
            end
        end)
        WindUI:Notify({
            Title = "恢复投票系统",
            Content = "恢复投票系统，请稍候...",
            Duration = 10,
            Icon = "loader-circle"
        })
        task.wait(10)
        WindUI:Notify({
            Title = "恢复投票系统",
            Content = "投票系统已恢复！你现在可以使用自动投票模式了。",
            Duration = 5,
            Icon = "check"
        })
    end
})

GameModeDropdown2 = Main7:Dropdown({
    Title = "设置投票模式",
    Desc = "选择自动投票将投选的游戏模式。",
    Values = VoteDisplayNames,
    Multi = false,
    Value = GetDisplayName(VoteMap, AutoVoteValue) or AutoVoteValue,
    Callback = function(value)
        local english = VoteMap[value] or value
        AutoVoteValue = english
        Config:Set("AutoVoteValue", english)
        Config:Save()
        print("[YYa] 投票模式已选择:", value, "->", english)
    end
})

AutoVoteIGToggle = Main7:Toggle({
    Title = "自动投票（游戏中）",
    Desc = "每轮游戏自动投票并自动准备。开启前请先选择投票模式！",
    Value = AutoVoteinGameEnabled,
    Callback = function(enabled)
        AutoVoteinGameEnabled = enabled
        Config:Set("AutoVoteinGameEnabled", enabled)
        Config:Save()
        if enabled then
            if AutoStartEnabled and IsMiscFarmAllowed() then
                FireGetReady(0)
            else
                FireAutoVote(true)
            end
            StartAutoVoteLoop()
            WindUI:Notify({ Title = "自动投票", Content = "已启用，模式: " .. tostring(GetDisplayName(VoteMap, AutoVoteValue) or AutoVoteValue), Duration = 3, Icon = "check" })
        else
            AutoVoteLoopRunning = false
            WindUI:Notify({ Title = "自动投票", Content = "已禁用", Duration = 2, Icon = "square" })
        end
    end
})

Main7:Divider()
Main7:Section({ Title = "休闲模式", TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = "休闲模式任务选择",
    Desc = "- [步骤 1] 在大厅中（不在游戏内）\n- [步骤 2] 按 Play 并进入经典模式选择界面\n- [步骤 3] 选择休闲模式并完成传送\n- [步骤 4] 运行脚本",
    Image = "rbxassetid://103720636367587",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = "设置游戏模式", Icon = "gamepad-2" })

GameModeDropdown = Main7:Dropdown({
    Title = "设置游戏模式",
    Desc = "选择自动创建将创建的游戏模式。",
    Values = GameModeDisplayNames,
    Multi = false,
    Value = GetDisplayName(GameModeMap, AutoGameValue) or AutoGameValue,
    Callback = function(value)
        local english = GameModeMap[value] or value
        AutoGameValue = english
        Config:Set("AutoGameValue", english)
        Config:Save()
        print("[YYa] 游戏模式已选择: " .. tostring(value))
    end
})

DELAY = 1

function click_btn(btn)
    if btn and (btn:IsA("ImageButton") or btn:IsA("TextButton")) then
        pcall(function()
            if firesignal then
                firesignal(btn.MouseButton1Click)
                firesignal(btn.Activated)
            else
                btn:Activate()
            end
        end)
    end
end

function notify(title, content, icon)
    WindUI:Notify({
        Title = title,
        Content = content,
        Duration = 3,
        Icon = icon or "check"
    })
end

task.spawn(function()
    local playBtn =
        workspace:FindFirstChild("ForGui") and
        workspace.ForGui:FindFirstChild("SurfaceGui") and
        workspace.ForGui.SurfaceGui:FindFirstChild("Frame") and
        workspace.ForGui.SurfaceGui.Frame:FindFirstChild("Play")

    if playBtn then
        notify("自动游戏模式（大厅）", "检测到 Play 按钮，自动开始...")
        task.wait(DELAY)

        local playGui = pg:FindFirstChild("Play")

        if not (playGui and playGui.Enabled) then
            click_btn(playBtn)
            notify("自动游戏模式（大厅）", "已按下 Play 按钮")
        else
            notify("自动游戏模式（大厅）", "Play GUI 已打开")
        end
    end

    task.wait(DELAY)

    local playGui = pg:FindFirstChild("Play")
    if not (playGui and playGui.Enabled) then return end

    local classicBtn = playGui:FindFirstChild("Classic")

    if classicBtn then
        notify("自动游戏模式（大厅）", "正在选择经典模式...")
        task.wait(DELAY)
        click_btn(classicBtn)
    end

    task.wait(DELAY)

    local modeGui = pg:FindFirstChild("mode select2")

    if modeGui and modeGui.Enabled then
        local diffBtn =
            modeGui:FindFirstChild("MainFrame") and
            modeGui.MainFrame:FindFirstChild("DiffMode")

        if diffBtn then
            notify("自动游戏模式（大厅）", "正在选择难度...")
            task.wait(DELAY)
            click_btn(diffBtn)
        end
    end
end)

AutoVoteEnabled = Config:Get("AutoVoteEnabled", false)

task.spawn(function()
    while true do
        task.wait(0.5)

        local loadingGui = pg:FindFirstChild("LoadingScreen")

        if loadingGui then
            notify("自动游戏模式（大厅）", "检测到大厅，准备自动设置...")
            pcall(function() loadingGui:Destroy() end)
        end

        local lobby = pg:FindFirstChild("Lobby")

        if lobby and lobby.Enabled then
            notify("自动游戏模式（大厅）", "检测到大厅，准备自动设置...")

            local btn =
                lobby:FindFirstChild("MainFrame") and
                lobby.MainFrame:FindFirstChild("Frame") and
                lobby.MainFrame.Frame:FindFirstChild("Create") and
                lobby.MainFrame.Frame.Create:FindFirstChild("TrackQuestButton")

            if btn and btn.Visible then
                notify("自动游戏模式（大厅）", "正在按下 TrackQuestButton...")
                click_btn(btn)

                task.wait(0.5)

                if AutoVoteEnabled then
                    notify("自动游戏模式（大厅）", "正在创建游戏模式...")

                    ReplicatedStorage.MainHandler:FireServer({
                        [1] = "StartSolo",
                        [2] = AutoGameValue
                    })

                    notify("自动游戏模式（大厅）", "游戏模式创建成功！")
                else
                    notify("自动游戏模式（大厅）", "请使用自动游戏模式！")
                end

                break
            end
        end
    end
end)

AutoVoteToggle = Main7:Toggle({
    Title = "自动游戏模式（大厅）",
    Desc = "在大厅时自动创建所选游戏模式。",
    Value = AutoVoteEnabled,
    Callback = function(enabled)
        AutoVoteEnabled = enabled
        Config:Set("AutoVoteEnabled", enabled)
        Config:Save()

        if enabled then
            notify("自动游戏模式（大厅）", "已启用")
        else
            notify("自动游戏模式（大厅）", "已禁用", "x")
        end
    end
})

-- ============================================================
-- ====================== STARTUP =============================
-- ============================================================
function ApplySavedConfigOnStartup()
    task.wait(1)
    print("[YYa] 跳过原初始化，手动启动功能...")

    pcall(function()
        local humanoid = Character and Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = WSValue
            humanoid.JumpPower = JPValue
        end
    end)

    ApplyCameraMode(true)
    UpdateYYAWaitingPartCollision()

    if FullBrightEnabled then ApplyFullBright() end
    if NoFogEnabled then ApplyNoFog() end

    if EnhancedNormalEnabled then
        LoadEnhancedLoop("Normal")
        if AutoFarmEnabled then
            StartFarmLoop()
            StartAutoAttack()
            StartAutoFillUpLoop()
            StartAutoStart()
        end
    elseif EnhancedDarkDimensionEnabled then
        LoadEnhancedLoop("DarkDimension")
        if AutoFarmEnabled then
            StartFarmLoop()
            StartAutoAttack()
            StartAutoFillUpLoop()
            StartAutoStart()
        end
    elseif EnhancedAstroEnabled then
        LoadEnhancedLoop("Astro")
        if AutoFarmEnabled then
            StartFarmLoop()
            StartAutoAttack()
            StartAutoFillUpLoop()
            StartAutoStart()
        end
    end

    if AutoVoteinGameEnabled then
        StartAutoVoteLoop()
    end

    if AntiAFK then StartAntiAFK() end

    if ESP.Enabled then StartESPLoop() end

    if AutoCollectEnabled then
        KnownCollectItems = {}
        CollectCandidateCache = {}
        CollectCacheDirty = true
        StartAutoCollectLoop()
    end

    if DeleteMapEnabled then SaveAndBoostFPS() end

    if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end

    if noBarrierActive then startNoBarrier() end

    print("[YYa] 手动启动完成")
end

-- ============================================================
-- ====================== EXECUTION ============================
-- ============================================================
ApplySavedConfigOnStartup()

print("[YYa] 版本: " .. version .. " | 更新日志: " .. ver .. " 加载成功！")
print("[YYa] 配置系统已激活 | 自动保存间隔 " .. tostring(AutoSaveDelay) .. " 秒")
print("[YYa] 至尊版 - 增强循环已就绪")

-- ============================================================
-- ====================== 脚本结束 =============================
-- ============================================================

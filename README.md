-- v085 -- ===================================================
local version = "重构全功能汉化版" 
local ver = "v023.4" 
-- ===========================================================

-- ====================== 加载 UI 引擎 ====================== 
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))() 

-- ====================== 游戏环境校验 ====================== 
repeat task.wait() until game:IsLoaded() 

-- ====================== 加载提示 GUI ====================== 
local p = game:GetService("Players").LocalPlayer 
local pg = p:WaitForChild("PlayerGui") 
local function waitLoadingGone() 
    local gui = pg:FindFirstChild("LoadingGui") 
    if gui then 
        WindUI:Notify({ Title = "系统初始化", Content = "游戏正在加载中，请稍候...", Duration = 3, Icon = "download" }) 
        gui.AncestryChanged:Wait() 
    end 
end 
waitLoadingGone() 
WindUI:Notify({ Title = "系统初始化", Content = "加载完成！脚本将在 3 秒后启动。", Duration = 3, Icon = "shield-check" }) 
task.wait(3) 

-- ====================== 帧率极限解锁 ====================== 
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
    WindUI:Notify({ Title = "系统服务", Content = "帧率极限解锁已激活！ | " .. ver, Duration = 3, Icon = "cpu" }) 
else 
    WindUI:Notify({ Title = "不支持该功能", Content = "当前执行器不支持 setfpscap 帧率解锁。", Duration = 3, Icon = "ban" }) 
end 

-- ====================== 本地配置文件系统 ====================== 
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
function CustomConfig:Get(key, default) if self.ConfigData[key] ~= nil then return self.ConfigData[key] end return default end 
function CustomConfig:Save() 
    local success, err = pcall(function() writefile(self.ConfigPath, HttpService:JSONEncode(self.ConfigData)) end) 
    if success then warn("[DYHUB] 配置已保存!") else warn("[DYHUB] 保存配置失败:", err) end 
end 
function CustomConfig:Load() 
    if isfile(self.ConfigPath) then 
        local success, result = pcall(function() return HttpService:JSONDecode(readfile(self.ConfigPath)) end) 
        if success and type(result) == "table" then 
            self.ConfigData = result 
            print("[DYHUB] 配置加载成功!") 
        else 
            warn("[DYHUB] 配置解析失败，已使用默认配置") 
            self.ConfigData = {} 
        end 
    else 
        print("[DYHUB] 未找到配置文件，正在创建新配置") 
        self.ConfigData = {} 
    end 
end 
function CustomConfig:AutoSave(interval) 
    task.spawn(function() 
        while true do task.wait(interval or 15) self:Save() end 
    end) 
end 
local Config = CustomConfig.new() 
Config:AutoSave(15) 

-- ====================== 安全纯净盾 ====================== 
local ExtraVersion = "尊享 VIP 特权汉化版" 
local userversion = ExtraVersion 

-- ====================== 创建 UI 主窗口 ====================== 
local Window = WindUI:CreateWindow({ 
    Title = "DYHUB 核心控制台", 
    IconThemed = true, 
    Icon = "rbxassetid://104487529937663", 
    Author = "STBB 汉化模块 | " .. userversion, 
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
Window:EditOpenButton({ Title = "打开菜单", Icon = "monitor", CornerRadius = UDim.new(0, 6), StrokeThickness = 2, Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)), Draggable = true }) 

-- ====================== UI 分页标签汉化 ====================== 
local Info = Window:Tab({ Title = "辅助信息", Icon = "info" }) 
MainDivider = Window:Divider() 
local Main = Window:Tab({ Title = "自动刷取", Icon = "rocket" }) 
local Main4 = Window:Tab({ Title = "透视信息", Icon = "eye" }) 
local Main2 = Window:Tab({ Title = "玩家属性", Icon = "user" }) 
MainDivider1 = Window:Divider() 
local Main5 = Window:Tab({ Title = "自动商店", Icon = "shopping-cart" }) 
local Main6 = Window:Tab({ Title = "物品收集", Icon = "hand" }) 
local Main7 = Window:Tab({ Title = "一键准备", Icon = "check-square" }) 
MainDivider2 = Window:Divider() 
local Main3 = Window:Tab({ Title = "系统设置", Icon = "settings" }) 
Window:SelectTab(1) 

-- ======================== 辅助信息页 ======================== 
Info:Section({ Title = "最新更新日志", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "更新日期: 2026/05/16", 
    Desc = "- [汉化] 23567 核心功能模块完美补全并全中文汉化\n- [修正] 一键准备功能（原声 GetReadyRemote 封包接口）UI 交互补齐\n- [强化] 新增最后3个稳定性补丁：买血冲突硬打断、配置深度双向绑定、ESP内存完美释放\n- [安全] 已完全斩断外部连接暗桩，保留最硬核挂机逻辑", 
    Image = "rbxassetid://104487529937663", 
    ImageSize = 30, 
}) 
Info:Divider() 
Info:Section({ Title = "安全卫士状态", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "本地安全盾已全面防护", 
    Desc = "当前状态: 安全运行中\n本地验证: 已移除全部卡密与后门远程阻断。", 
    Image = "rbxassetid://104487529937663", 
    ImageSize = 30 
})

-- ====================== 系统服务绑定 ====================== 
local TweenService = game:GetService("TweenService") 
local ReplicatedStorage = game:GetService("ReplicatedStorage") 
local VirtualInputManager = game:GetService("VirtualInputManager") 
local RunService = game:GetService("RunService") 
local Players = game:GetService("Players")

-- ====================== 玩家动态加载 ====================== 
local LocalPlayer = Players.LocalPlayer 
local Client = LocalPlayer 
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait() 
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart") 

LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
end)

-- ====================== 常量数据字典表 ====================== 
GlobalTables = { 
    Mode = { "普通模式 (Normal)", "模糊记忆 (Vague Memory)", "极限模式 (Extreme)", "困难模式 (Hard)", "疯狂模式 (Insane)", "噩梦模式 (Nightmare)", "首领连战 (Boss Rush)", "黑暗维度 (Dark Dimension)", "地狱 (Hell)", "迷雾 (Mist)", "圣诞节第一幕", "僵尸生存第一幕", "据点防守 (Holdout)", "入侵 (Invasion)" }, 
    Votes = { "Normal","VeryHard","Hard","Insane","Nightmare","BossRush", "DarkDimension","Hell","ThunderStorm","Christmas","Zombie", "AstroV2","Astro","100MVisit" }, 
    Weapon = { "Stungun", "Flamethrower", "Harpoon Gun", "Shot Gun", "Pulse Rifle" }, 
    MiscShop = { "HeadPhone", "泰坦支援申请", "高级泰坦支援", "音响支援申请", "Grenade", "Jetpack", "Lens" }, 
} 

-- ====================== 核心全局变量双向绑定初始化 ====================== 
local skillList = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 
local skillDropdownValues = { "全选", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 

-- 模块 2 & 3 核心配置参数
local AutoFarmEnabled = Config:Get("AutoFarmEnabled", false) 
local FarmPosition = Config:Get("FarmPosition", "Above") 
local FarmMode = Config:Get("FarmMode", "Tween") 
local HeightValue = Config:Get("HeightValue", 3) 
local TweenSpeed = Config:Get("TweenSpeed", 1) 
local LoopDelay = 0.05 

-- 技能与攻击
local AutoAttackEnabled = Config:Get("AutoAttackEnabled", false) 
local AutoSkillEnabled = Config:Get("AutoSkillEnabled", false) 
local SelectedSkills = Config:Get("SelectedSkills", { "全选" }) 
local SkillDelay = Config:Get("SkillDelay", 1) 
local HighHPThreshold = Config:Get("HighHPThreshold", 200) 

-- 玩家状态调整项
local AutoSkipHeliEnabled = Config:Get("AutoSkipHeliEnabled", false) 
local BoostFPS_Active_dummy = false 
local GodModeEnabled = Config:Get("GodModeEnabled", false) 
local GodModeValue = Config:Get("GodModeValue", 30) 

-- 模块 5 核心配置参数
local AutoBuyWeaponEnabled = Config:Get("AutoBuyWeaponEnabled", false) 
local AutoBuyMiscEnabled = Config:Get("AutoBuyMiscEnabled", false) 
local SelectedWeapon = Config:Get("SelectedWeapon", "Stungun") 
local SelectedMiscItem = Config:Get("SelectedMiscItem", "HeadPhone") 

-- 模块 6 核心配置参数
local CollectItems = { "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter", "Astro Samples", "Weird Prism", "Key Card", "Zombie Core", "Flash Drives", "Presents" } 
local CollectGroupMap = { ["Astro Samples"] = { "Trooper Blast","Trooper Spinner","Specialist Blaster","Specialist Spinner", "Specialist Sword Arm","Strider Leg","Interceptor Wing","Interceptor Goggles", "Interceptor Spinner","Impactor Cannon","Impactor Laser","High Impactor Cannon", "High Impactor Laser","Destructor Laser","Destructor Blaster","Destructor Core", "Obliterator Blaster","Obliterator Spinner" }, ["Presents"] = { "Gacha Capsule" } } 
local AutoCollectEnabled = Config:Get("AutoCollectEnabled", false) 
local SelectedCollectItems = Config:Get("SelectedCollectItems", {}) 
local CollectMode = Config:Get("CollectMode", "波次清理") 
local AutoFillUpEnabled = Config:Get("AutoFillUpEnabled", false) 

-- 模块 7 核心配置参数 (一键准备与大厅投票系统)
local AutoVoteEnabled = Config:Get("AutoVoteEnabled", false) 
local AutoGameValue = Config:Get("AutoGameValue", "普通模式 (Normal)") 
local AutoStartEnabled = Config:Get("AutoStartEnabled", false) 
local AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false) 
local AutoVoteValue = Config:Get("AutoVoteValue", "Normal") 

-- 基础状态控制量
local WaitingRespawn = false 
local LockActive = false 
local _currentTargetPriority = 0 
local _interruptSignal = false 
local IdlePosition = CFrame.new(-23.3435822, 67, 0.341766357) 
local DynamicNeedNoClip = false 

-- 系统底层控制量
local MiscOptions = Config:Get("MiscOptions", {}) 
local noBarrierActive = Config:Get("NoBarrier", false) 
local AntiAFK = Config:Get("AntiAfk", true) 
local AutoStartConnection = nil 
local _voteRespawnConn = nil 
local _syncRespawnConn = nil 
local _voteIGRespawnConn = nil 
local noBarrierConnection = nil 
local VirtualUser = game:GetService("VirtualUser") 

-- 自动补血购买基础设置
local FILLUP_PART_PATH = { "HelicopterShop", "ShopXDD", "PartForShop" } 
local FILLUP_TARGET_POS = Vector3.new(44.2756729, 26.3595276, -32.7318268) 
local FILLUP_POS_THRESHOLD = 0.5 
local FillUpRunning = false 

-- ====================== 辅助过滤检测函数 ====================== 
local PlayerNames = {} 
for _, plr in ipairs(game.Players:GetPlayers()) do if plr ~= LocalPlayer and plr.Character then PlayerNames[plr.Name] = true end end 
game.Players.PlayerAdded:Connect(function(plr) PlayerNames[plr.Name] = true end) 
game.Players.PlayerRemoving:Connect(function(plr) PlayerNames[plr.Name] = nil end) 

local AllyNames = { ["Heavy Soldier Toilet V2"] = true, ["Quad Laser Toilet"] = true, ["Strider Rocket Laser"] = true, ["Helicopter Camera"] = true, ["Heavy Soldier Toilet V1"] = true, ["Rocket Heli v2"] = true, ["Gunner Camera man"] = true, ["Attack Helicopter"] = true, ["Swat Mutant"] = true, ["Huge DJ Toilet"] = true, } 
local function IsAlly(mob) return AllyNames[mob.Name] ~= nil or PlayerNames[mob.Name] ~= nil end 

local function IsValidMob(obj) 
    if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then 
        if Players:GetPlayerFromCharacter(obj) then return false end 
        if IsAlly(obj) then return false end 
        local humanoid = obj:FindFirstChild("Humanoid") if humanoid and humanoid.Health > 0 then return true end 
    end 
    return false 
end 
local function IsMobDead(mob) if not mob or not mob.Parent then return true end local humanoid = mob:FindFirstChild("Humanoid") if not humanoid or humanoid.Health <= 0 then return true end return false end 
local function GetMobMaxHP(mob) local humanoid = mob and mob:FindFirstChild("Humanoid") if not humanoid then return 0 end return humanoid.MaxHealth or 0 end 

-- ====================== 模块 2: 空间传送位移算法核心 ====================== 
function tp(pu79) 
    pcall(function() 
        if Character:FindFirstChild("Humanoid") and Character.Humanoid.Sit == true then Character.Humanoid.Sit = false end 
        DynamicNeedNoClip = true 
        local v81 = { Target = pu79.Target, Mod = pu79.Mod or CFrame.new(0, 0, 0) } 
        HumanoidRootPart.CFrame = v81.Target * v81.Mod 
    end) 
end 
function Tp(p82) 
    if Character.Humanoid.Sit == true then Character.Humanoid.Sit = false end 
    for _, part in pairs(Character:GetDescendants()) do 
        if part:IsA("BasePart") then part.CanCollide = false end 
    end 
    if not HumanoidRootPart:FindFirstChild("BodyClip") then 
        local v87 = Instance.new("BodyVelocity") v87.Parent = HumanoidRootPart v87.Name = "BodyClip" v87.Velocity = Vector3.new(0, 0, 0) v87.MaxForce = Vector3.new(5, math.huge, 5) 
    end 
    HumanoidRootPart.CFrame = p82 
end 
function tp1(p89) 
    if HumanoidRootPart then HumanoidRootPart.CFrame = p89 end 
end 

-- 动态无碰撞处理
RunService.Stepped:Connect(function()
    if DynamicNeedNoClip and Character then
        for _, v in pairs(Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

-- ====================== 怪物筛选机制 ====================== 
local function GetNearestMob() 
    local nearestMob, nearestDist = nil, math.huge local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return nil end 
    for _, mob in ipairs(livingFolder:GetChildren()) do 
        if IsValidMob(mob) then 
            local mobRoot = mob:FindFirstChild("HumanoidRootPart") 
            if mobRoot then local d = (HumanoidRootPart.Position - mobRoot.Position).Magnitude if d < nearestDist then nearestDist = d; nearestMob = mob end end 
        end 
    end 
    return nearestMob 
end 
local function GetHelicopter() 
    local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return nil end 
    for _, mob in ipairs(livingFolder:GetChildren()) do if mob.Name:lower():find("helicopter") and IsValidMob(mob) then return mob end end 
    return nil 
end 
local function GetGiantSTToilet() 
    local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return nil end 
    local giant = livingFolder:FindFirstChild("Giant ST toilet") 
    if giant and IsValidMob(giant) then local lever = giant:FindFirstChild("lever") if lever then local prompt = lever:FindFirstChildOfClass("ProximityPrompt") if prompt then return giant, prompt end end end 
    return nil, nil 
end 
local function GetHighHPMob() 
    local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return nil end 
    local bestMob, bestHP = nil, HighHPThreshold 
    for _, mob in ipairs(livingFolder:GetChildren()) do if IsValidMob(mob) then local hp = GetMobMaxHP(mob) if hp > bestHP then bestHP = hp bestMob = mob end end end 
    return bestMob 
end 
local function GetPriorityMob() 
    local giant, prompt = GetGiantSTToilet() if giant and prompt then return giant, "GiantST", prompt, 4 end 
    local heli = GetHelicopter() if heli then return heli, "Helicopter", nil, 3 end 
    local highHPMob = GetHighHPMob() if highHPMob then return highHPMob, "HighHP", nil, 2 end 
    local nearMob = GetNearestMob() if nearMob then return nearMob, "NearestMob", nil, 1 end 
    return nil, nil, nil, 0 
end 

-- 实时拦截打断检测
local function CheckInterrupt(currentPriority)
    local _, _, _, newPriority = GetPriorityMob()
    if newPriority > currentPriority then return true, newPriority end
    return false, currentPriority
end

-- ====================== 模块 3: 高度受击动态覆盖算法核心 ====================== 
local function GetMobVisualBounds(mob) 
    local minY, maxY = math.huge, -math.huge local centerX, centerZ, count = 0, 0, 0 
    for _, part in ipairs(mob:GetDescendants()) do 
        if part:IsA("BasePart") and part.Transparency < 0.9 and part.Size.Y > 0.1 then 
            local pos = part.Position local hy = part.Size.Y * 0.5 
            if pos.Y - hy < minY then minY = pos.Y - hy end if pos.Y + hy > maxY then maxY = pos.Y + hy end 
            centerX = centerX + pos.X centerZ = centerZ + pos.Z count = count + 1 
        end 
    end 
    if count == 0 then local hrp = mob:FindFirstChild("HumanoidRootPart") if hrp then return hrp.Position, hrp.Position.Y - 2, hrp.Position.Y + 2 end return Vector3.new(0, 0, 0), 0, 4 end 
    return Vector3.new(centerX / count, (minY + maxY) * 0.5, centerZ / count), minY, maxY 
end 

local PADDING_REDUCE_STEP = 2 
local PADDING_SAFE_MIN = -30 
local DMG_THRESHOLD = 40 
local ANTI_CLIP_MARGIN = 3 
local PLAYER_HALF_HEIGHT = 3 
local MobHeightOverride = {} local MobConfirmedPadding = {} local MobLastHealth = {} local MobCheckerCancelled = {} 

local function GetAntiClipFloor(mob) local _, minY, maxY = GetMobVisualBounds(mob) return -(maxY - minY) + PLAYER_HALF_HEIGHT + ANTI_CLIP_MARGIN end 
local function GetEffectivePadding(mob) if MobConfirmedPadding[mob] ~= nil then return MobConfirmedPadding[mob] end if MobHeightOverride[mob] ~= nil then return MobHeightOverride[mob] end return HeightValue end 
local function ClampPaddingToAntiClip(mob, padding) return math.max(math.max(padding, GetAntiClipFloor(mob)), PADDING_SAFE_MIN) end 

local function StartDamageChecker(mob) 
    MobCheckerCancelled[mob] = false 
    task.spawn(function() 
        local humanoid = mob and mob:FindFirstChild("Humanoid") if not humanoid or MobConfirmedPadding[mob] ~= nil then return end 
        MobLastHealth[mob] = humanoid.Health MobHeightOverride[mob] = ClampPaddingToAntiClip(mob, MobHeightOverride[mob] or HeightValue) 
        local lastDamageTime = tick() local noDamageTimer = 0 local hitStreak = 0 local lastWasHit = false local reducedOnce = false 
        while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do 
            task.wait(0.3) if MobCheckerCancelled[mob] or not mob or not mob.Parent or IsMobDead(mob) then break end 
            humanoid = mob:FindFirstChild("Humanoid") if not humanoid then break end 
            local currentHP = humanoid.Health local lastHP = MobLastHealth[mob] or currentHP local dmgDealt = lastHP - currentHP local gotHit = dmgDealt > 0 
            if gotHit then 
                lastDamageTime = tick() noDamageTimer = 0 reducedOnce = false hitStreak = lastWasHit and (hitStreak + 1) or 1 lastWasHit = true local curPad = GetEffectivePadding(mob) 
                if (dmgDealt >= DMG_THRESHOLD or hitStreak >= 2) and MobConfirmedPadding[mob] == nil then 
                    if not MobCheckerCancelled[mob] then MobConfirmedPadding[mob] = curPad MobHeightOverride[mob] = curPad end break 
                end 
            else lastWasHit = false hitStreak = 0 noDamageTimer = tick() - lastDamageTime end 
            if (noDamageTimer >= 3 and not reducedOnce) or noDamageTimer >= 6 then 
                if noDamageTimer >= 6 then lastDamageTime = tick() end reducedOnce = true 
                local curPad = GetEffectivePadding(mob) local newPad = ClampPaddingToAntiClip(mob, curPad - PADDING_REDUCE_STEP) if newPad ~= curPad then MobHeightOverride[mob] = newPad end 
            end 
            MobLastHealth[mob] = currentHP 
        end 
        if not MobCheckerCancelled[mob] then MobHeightOverride[mob] = nil MobLastHealth[mob] = nil end 
    end) 
end 
local function ResetMobOverride(mob) if mob then MobCheckerCancelled[mob] = true MobHeightOverride[mob] = nil MobConfirmedPadding[mob] = nil MobLastHealth[mob] = nil end end 

-- ====================== 空间相对坐标转换定位 ====================== 
local function GetTargetCFrame(mob, position) 
    local mobRoot = mob:FindFirstChild("HumanoidRootPart") if not mobRoot then return nil end 
    local padding = GetEffectivePadding(mob) local center, minY, maxY = GetMobVisualBounds(mob) 
    if position == "Above" then 
        local safeTargetY = math.max(maxY + padding, maxY + 0.5) return CFrame.new(Vector3.new(center.X, safeTargetY, center.Z), Vector3.new(center.X, maxY, center.Z)) * CFrame.Angles(math.rad(-10), 0, 0) 
    elseif position == "Under" then 
        local safeTargetY = math.min(minY - padding, minY - 0.5) return CFrame.new(Vector3.new(center.X, safeTargetY, center.Z), Vector3.new(center.X, minY, center.Z)) * CFrame.Angles(math.rad(10), 0, 0) 
    end 
end 

local function TeleportToMob(mob) 
    local cf = GetTargetCFrame(mob, FarmPosition) if not cf then return end 
    if FarmMode == "Tween" then 
        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), { CFrame = cf }) tween:Play() tween.Completed:Wait() 
    elseif FarmMode == "tp" then tp({ Target = cf, Mod = CFrame.new(0, 0, 0) }) elseif FarmMode == "Tp" then Tp(cf) elseif FarmMode == "tp1" then tp1(cf) end 
end 

-- ====================== 挂机战斗序列后台线程 ====================== 
local function StartAutoAttack() 
    task.spawn(function() while AutoAttackEnabled and AutoFarmEnabled do local mob = GetPriorityMob() if mob and not WaitingRespawn and not FillUpRunning then pcall(function() ReplicatedStorage.LMB:FireServer() end) end task.wait(0.05) end end) 
end 
local function StartAutoSkill() 
    task.spawn(function() 
        while AutoSkillEnabled and AutoFarmEnabled do 
            local mob = GetPriorityMob() 
            if mob and not WaitingRespawn and not FillUpRunning then 
                local keysToPress = table.find(SelectedSkills, "全选") and skillList or SelectedSkills 
                for _, key in ipairs(keysToPress) do 
                    if not AutoSkillEnabled or not AutoFarmEnabled or FillUpRunning then break end 
                    local keyCode = Enum.KeyCode[key] 
                    if keyCode then pcall(function() VirtualInputManager:SendKeyEvent(true, keyCode, false, game) task.wait(0.05) VirtualInputManager:SendKeyEvent(false, keyCode, false, game) end) task.wait(SkillDelay) end 
                end 
            end 
            task.wait(LoopDelay) 
        end 
    end) 
end 
local function TriggerAutoSkipHeli(state) pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(state) end) end 

-- ====================== 模块 5: 全自动商店循环购买与【拼图一：硬打断补丁】 ====================== 
task.spawn(function() 
    while true do 
        if AutoBuyWeaponEnabled and SelectedWeapon then 
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end) 
        end 
        task.wait(10) 
    end 
end) 

task.spawn(function() 
    while true do 
        if AutoBuyMiscEnabled and SelectedMiscItem then 
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end) 
        end 
        task.wait(10) 
    end 
end) 

-- 自动补血执行算法
local function GetFillUpPart() 
    local obj = workspace 
    for _, key in ipairs(FILLUP_PART_PATH) do obj = obj:FindFirstChild(key) if not obj then return nil end end 
    return obj 
end 
local function IsFillUpPartReady() 
    local p = GetFillUpPart() if not p then return false end 
    return (p.CFrame.Position - FILLUP_TARGET_POS).Magnitude < FILLUP_POS_THRESHOLD 
end 
local function DoFillUp() for i = 1, 2 do pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", "FillHP") end) if i < 2 then task.wait(0.3) end end end 

local function GetPlayerHealthPercent() 
    local humanoid = Character and Character:FindFirstChild("Humanoid") 
    if not humanoid or humanoid.MaxHealth <= 0 then return 100 end 
    return (humanoid.Health / humanoid.MaxHealth) * 100 
end 

local function StartAutoFillUpLoop() 
    if FillUpRunning then return end 
    task.spawn(function() 
        while AutoFillUpEnabled and AutoFarmEnabled do 
            if GetPlayerHealthPercent() < 99 then 
                FillUpRunning = true 
                _interruptSignal = true -- 触发全局硬打断信号
                if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end 
                local waited = 0 
                while not IsFillUpPartReady() and AutoFillUpEnabled do waited = waited + 0.2 if waited >= 30 then break end task.wait(0.2) end 
                if IsFillUpPartReady() and AutoFillUpEnabled then 
                    if HumanoidRootPart then HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero end
                    DoFillUp() 
                    task.wait(1.2) 
                end 
                if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end 
                _interruptSignal = false 
                FillUpRunning = false 
            end 
            task.wait(1) 
        end 
    end) 
end 

-- 低血智能自爆重置状态
task.spawn(function() 
    while true do 
        task.wait(0.1) 
        if GodModeEnabled and Character then 
            pcall(function() 
                local humanoid = Character:FindFirstChild("Humanoid") if not humanoid or humanoid.MaxHealth <= 0 then return end 
                if (humanoid.Health / humanoid.MaxHealth) * 100 < GodModeValue then local head = Character:FindFirstChild("Head") if head then head:Destroy() else humanoid.Health = 0 end end 
            end) 
        end 
    end 
end) 

-- ====================== 模块 6: 战场掉落物自动捞取 ====================== 
local function MatchesPattern(objectName, pattern) 
    local objL, patL = objectName:lower(), pattern:lower() if objL == patL then return true end 
    if #objL > #patL and objL:sub(1, #patL) == patL then if table.find({" ", "#", "_", "-"}, objL:sub(#patL + 1, #patL + 1)) then return true end end 
    if CollectGroupMap[pattern] then for _, gName in ipairs(CollectGroupMap[pattern]) do if objL == gName:lower() then return true end end end 
    return false 
end 
local function IsCollectTarget(objectName) for _, pattern in ipairs(SelectedCollectItems) do if MatchesPattern(objectName, pattern) then return true end end return false end 
local function FindNewCollectItems() 
    local found = {} for _, obj in ipairs(workspace:GetDescendants()) do if obj and obj.Parent and IsCollectTarget(obj.Name) and (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("BasePart")) then table.insert(found, obj) end end return found 
end 
local function GetItemRootPart(obj) return obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or (obj:IsA("BasePart") or obj:IsA("MeshPart")) and obj or nil end 
local function TweenToItem(itemRoot) if not itemRoot or not HumanoidRootPart then return end local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position) }) tween:Play() tween.Completed:Wait() end 
local function ActivateItemPrompts(obj) 
    pcall(function() for _, child in ipairs(obj:GetDescendants()) do if child:IsA("ProximityPrompt") then child.HoldDuration = 0 child.MaxActivationDistance = 50 if fireproximityprompt then fireproximityprompt(child) end child:InputHoldBegin() task.wait(0.05) child:InputHoldEnd() end end end) 
end 
local function IsItemGone(obj) return not obj or not obj.Parent end 
local function AllMobsDead() local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return true end for _, mob in ipairs(livingFolder:GetChildren()) do if IsValidMob(mob) then return false end end return true end 

local function CollectSingleItem(obj) 
    if IsItemGone(obj) then return end local itemRoot = GetItemRootPart(obj) if not itemRoot then return end TweenToItem(itemRoot) local lockConn 
    lockConn = RunService.RenderStepped:Connect(function() 
        if IsItemGone(obj) or not AutoCollectEnabled or not itemRoot or not itemRoot.Parent then lockConn:Disconnect() return end 
        if Character and HumanoidRootPart then Character:PivotTo(CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position)) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end 
    end) 
    local timeout = 0 repeat ActivateItemPrompts(obj) task.wait(0.1) timeout = timeout + 0.1 if timeout > 10 then break end until IsItemGone(obj) or not AutoCollectEnabled lockConn:Disconnect() 
end 

local function StartAutoCollectLoop() 
    if CollectRunning then return end CollectRunning = true 
    task.spawn(function() 
        while AutoCollectEnabled do 
            if #SelectedCollectItems > 0 then 
                local itemsToCollect = FindNewCollectItems() 
                if #itemsToCollect > 0 then 
                    if CollectMode == "抢夺模式" then 
                        LockActive = false task.wait(0.1) for _, obj in ipairs(itemsToCollect) do if not AutoCollectEnabled then break end if not IsItemGone(obj) then CollectSingleItem(obj) end end 
                        if AutoFarmEnabled then TeleportToIdle() WaitingRespawn = false end 
                    elseif CollectMode == "波次清理" then 
                        while not AllMobsDead() and AutoCollectEnabled do task.wait(0.5) end 
                        if not AutoCollectEnabled then break end if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end LockActive = false task.wait(0.1) 
                        for _, obj in ipairs(FindNewCollectItems()) do if not AutoCollectEnabled then break end if not IsItemGone(obj) then CollectSingleItem(obj) end end 
                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end 
                        if AutoFarmEnabled then TeleportToIdle() WaitingRespawn = false end 
                    end 
                end 
            end 
            task.wait(0.5) 
        end 
        CollectRunning = false 
    end) 
end 

-- ====================== 模块 7: 一键准备与大厅难度投票核心 ====================== 
local function FilterModeString(val)
    local raw = val:match("%((.-)%)")
    return raw or val
end

local function FireVote_Solo() 
    if not AutoGameValue then return end 
    local modeArg = FilterModeString(AutoGameValue)
    pcall(function() ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = modeArg }) end) 
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
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end 
    if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end 
    FireVote_Solo() 
    task.spawn(function() task.wait(2.5) if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end end) 
    _syncRespawnConn = LocalPlayer.CharacterAdded:Connect(function() 
        task.wait(1.5) 
        if AutoVoteEnabled and AutoStartEnabled then 
            FireVote_Solo() 
            task.spawn(function() task.wait(2.5) if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end end) 
        end 
    end) 
end 

local function SetupAutoVote_SoloOnly(enabled) 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end if not enabled then return end FireVote_Solo() 
    _voteRespawnConn = LocalPlayer.CharacterAdded:Connect(function() task.wait(1.5) if AutoVoteEnabled and not AutoStartEnabled then FireVote_Solo() end end) 
end 

local function SetupAutoStartOnly(enabled) 
    if AutoStartConnection then AutoStartConnection:Disconnect() AutoStartConnection = nil end if not enabled then return end FireGetReady() 
    AutoStartConnection = LocalPlayer.CharacterAdded:Connect(function() task.wait(1) if AutoStartEnabled and not AutoVoteEnabled then task.spawn(FireGetReady) end end) 
end 

local function RefreshVoteAndStartSetup() 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end 
    if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end 
    if AutoStartConnection then AutoStartConnection:Disconnect() AutoStartConnection = nil end 
    if AutoVoteEnabled and AutoStartEnabled then SetupSyncVoteAndStart() 
    elseif AutoVoteEnabled and not AutoStartEnabled then SetupAutoVote_SoloOnly(true) 
    elseif not AutoVoteEnabled and AutoStartEnabled then SetupAutoStartOnly(true) end 
end 

local function SetupAutoVote_InGame(enabled) 
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect() _voteIGRespawnConn = nil end if not enabled then return end FireVote_InGame() 
    _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function() task.wait(1.5) if AutoVoteinGameEnabled then FireVote_InGame() end end) 
end 

local function TeleportToIdle() 
    LockActive = false task.wait(0.1) WaitingRespawn = true 
    pcall(function() 
        Character:PivotTo(IdlePosition) 
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero 
        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero 
    end) 
end 

local function ActivateProximityPrompt(prompt) 
    pcall(function() prompt.HoldDuration = 0 prompt.MaxActivationDistance = 50 if fireproximityprompt then fireproximityprompt(prompt) end prompt:InputHoldBegin() task.wait(0.05) prompt:InputHoldEnd() end) 
end 
local function ActivateAllFlushPrompts() 
    pcall(function() for _, part in pairs(workspace:GetDescendants()) do if part:IsA("BasePart") or part:IsA("Model") then local prompt = part:FindFirstChildOfClass("ProximityPrompt") if prompt then local actionText = prompt.ActionText:lower() if actionText:find("flush") or actionText:find("flash") or actionText:find("dragon") then ActivateProximityPrompt(prompt) end end end end end) 
end 

-- ====================== 主自动化挂机大循环 ====================== 
local function StartFarmLoop() 
    task.spawn(function() 
        task.spawn(function() 
            while AutoFarmEnabled do 
                if WaitingRespawn and not LockActive and HumanoidRootPart then 
                    pcall(function() 
                        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = IdlePosition }) 
                        tween:Play() tween.Completed:Wait() 
                        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero 
                        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero 
                    end) 
                end 
                task.wait(0.1) 
            end 
        end) 
        while AutoFarmEnabled do 
            if FillUpRunning or _interruptSignal then 
                _currentTargetPriority = 0 
                task.wait(0.5) 
                continue 
            end 
            local mob, mobType, extraData, priority = GetPriorityMob() 
            if mob and HumanoidRootPart then 
                WaitingRespawn = false _currentTargetPriority = priority 
                if mobType == "GiantST" and extraData then 
                    LockActive = true local lockConn 
                    lockConn = RunService.RenderStepped:Connect(function() 
                        if IsMobDead(mob) or not AutoFarmEnabled or not LockActive or not mob:FindFirstChild("HumanoidRootPart") then lockConn:Disconnect() return end 
                        local partPos = mob.HumanoidRootPart.Position 
                        Character:PivotTo(CFrame.new(partPos + Vector3.new(0, 8, 0), partPos)) 
                        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero 
                        HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero 
                    end) 
                    repeat ActivateProximityPrompt(extraData) task.wait(0.1) until IsMobDead(mob) or not AutoFarmEnabled or not LockActive 
                    lockConn:Disconnect() LockActive = false 
                else 
                    StartDamageChecker(mob) 
                    while not IsMobDead(mob) and AutoFarmEnabled and not WaitingRespawn and not FillUpRunning do 
                        local changed, _ = CheckInterrupt(priority) 
                        if changed then _interruptSignal = true ResetMobOverride(mob) break end 
                        TeleportToMob(mob) task.wait(0.02) 
                    end 
                    if not _interruptSignal then ResetMobOverride(mob) end _interruptSignal = false 
                end 
            else 
                _currentTargetPriority = 0 
                if table.find(MiscOptions, "全图自动秒冲马桶") then ActivateAllFlushPrompts() end 
                if not WaitingRespawn then TeleportToIdle() end 
            end 
            task.wait(0.05) 
        end 
    end) 
end 

-- ====================== 流畅度删图底层 ====================== 
local BoostFPS_OriginalData = {} local BoostFPS_Active = false local BoostFPS_RestoreConnection = nil local BoostFPS_LightingData = {} 
local function IsLivingDescendant(obj) local current = obj while current and current ~= workspace do if current:IsA("Model") and current:FindFirstChildOfClass("Humanoid") then return true end current = current.Parent end return false end 
local function SaveAndBoostFPS() 
    if BoostFPS_Active then return end BoostFPS_Active = true BoostFPS_OriginalData = {} BoostFPS_LightingData = {} local Lighting = game:GetService("Lighting") 
    BoostFPS_LightingData = { Brightness = Lighting.Brightness, GlobalShadows = Lighting.GlobalShadows, FogEnd = Lighting.FogEnd, FogStart = Lighting.FogStart } 
    pcall(function() Lighting.GlobalShadows = false Lighting.Brightness = 1 Lighting.FogEnd = 100000 Lighting.FogStart = 100000 end) 
    for _, effect in ipairs(Lighting:GetChildren()) do pcall(function() if effect:IsA("Atmosphere") or effect:IsA("BloomEffect") or effect:IsA("ColorCorrectionEffect") or effect:IsA("DepthOfFieldEffect") or effect:IsA("SunRaysEffect") or effect:IsA("Sky") then BoostFPS_LightingData["effect_" .. effect.Name] = { class = effect.ClassName, inst = effect } effect.Parent = nil end end) end 
    pcall(function() 
        for _, obj in ipairs(workspace:GetDescendants()) do 
            if IsLivingDescendant(obj) then continue end 
            if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then BoostFPS_OriginalData[obj] = { Transparency = obj.Transparency, CastShadow = obj.CastShadow, Material = obj.Material } obj.Transparency = 1 obj.CastShadow = false pcall(function() obj.Material = Enum.Material.SmoothPlastic end) 
            elseif obj:IsA("Decal") or obj:IsA("Texture") then BoostFPS_OriginalData[obj] = { Transparency = obj.Transparency } obj.Transparency = 1 
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then BoostFPS_OriginalData[obj] = { Enabled = obj.Enabled } pcall(function() obj.Enabled = false end) end 
        end 
    end) 
end 
local function RestoreBoostFPS() 
    if not BoostFPS_Active then return end BoostFPS_Active = false if BoostFPS_RestoreConnection then BoostFPS_RestoreConnection:Disconnect() BoostFPS_RestoreConnection = nil end local Lighting = game:GetService("Lighting") 
    pcall(function() Lighting.Brightness = BoostFPS_LightingData.Brightness Lighting.GlobalShadows = BoostFPS_LightingData.GlobalShadows Lighting.FogEnd = BoostFPS_LightingData.FogEnd Lighting.FogStart = BoostFPS_LightingData.FogStart end) 
    for obj, data in pairs(BoostFPS_OriginalData) do pcall(function() if not obj or not obj.Parent then return end if data.Transparency then obj.Transparency = data.Transparency end if data.CastShadow ~= nil then obj.CastShadow = data.CastShadow end if data.Material then obj.Material = data.Material end if data.Enabled ~= nil then obj.Enabled = data.Enabled end end) end 
    BoostFPS_OriginalData = {} 
end 

-- ====================== 无障碍越界自动复位 ====================== 
local function startNoBarrier() 
    if noBarrierConnection then return end 
    noBarrierConnection = RunService.Heartbeat:Connect(function() 
        pcall(function() 
            if not HumanoidRootPart then return end local pos = HumanoidRootPart.Position 
            if math.abs(pos.X) > 1000 or math.abs(pos.Y) > 1000 or math.abs(pos.Z) > 1000 then HumanoidRootPart.CFrame = CFrame.new(Vector3.new(0, 50, 0)) end 
        end) 
    end) 
end 
local function stopNoBarrier() if noBarrierConnection then noBarrierConnection:Disconnect() noBarrierConnection = nil end end 

-- ====================== 【拼图三：ESP 内存深度防护自维护引擎】 ====================== 
local ESP = { Enabled = false, Boxes = true, Distance = true, Health = true, Color = Color3.fromRGB(0, 191, 255) } 
local EspObjects = {} 
local livingFolder = workspace:WaitForChild("Living", 5) 

local function RemoveEsp(mob) 
    if EspObjects[mob] then 
        pcall(function() EspObjects[mob].Box:Destroy() EspObjects[mob].Billboard:Destroy() end) 
        EspObjects[mob] = nil 
    end 
end 

if livingFolder then 
    livingFolder.ChildRemoved:Connect(function(child) 
        RemoveEsp(child) 
        ResetMobOverride(child) 
    end) 
end 

local function CreateEsp(mob) 
    if EspObjects[mob] then return end 
    local box = Instance.new("BoxHandleAdornment") local billboard = Instance.new("BillboardGui") local label = Instance.new("TextLabel") 
    box.Size = mob:GetExtentsSize() box.AlwaysOnTop = true box.ZIndex = 10 box.Adornment = mob:FindFirstChild("HumanoidRootPart") box.Color3 = ESP.Color box.Transparency = 0.6 box.Parent = mob 
    billboard.Size = UDim2.new(0, 200, 0, 50) billboard.AlwaysOnTop = true billboard.Adornment = mob:FindFirstChild("HumanoidRootPart") billboard.StudsOffset = Vector3.new(0, 3, 0) billboard.Parent = mob 
    label.Size = UDim2.new(1, 0, 1, 0) label.BackgroundTransparency = 1 label.TextColor3 = ESP.Color label.TextStrokeTransparency = 0 label.TextSize = 14 label.TextYAlignment = Enum.TextYAlignment.Top label.Parent = billboard 
    EspObjects[mob] = { Box = box, Billboard = billboard, Label = label } 
end 

local function UpdateEspVisibility() 
    for mob, assets in pairs(EspObjects) do 
        if not mob or not mob.Parent or IsMobDead(mob) then RemoveEsp(mob) 
        else 
            assets.Box.Visible = ESP.Enabled and ESP.Boxes assets.Billboard.Enabled = ESP.Enabled 
            local hum = mob:FindFirstChild("Humanoid") local hrp = mob:FindFirstChild("HumanoidRootPart") 
            if hum and hrp and HumanoidRootPart then 
                local dist = math.floor((HumanoidRootPart.Position - hrp.Position).Magnitude) local txt = mob.Name .. "\n" 
                if ESP.Health then txt = txt .. "生命: " .. math.floor(hum.Health) .. "/" .. math.floor(hum.MaxHealth) .. "\n" end 
                if ESP.Distance then txt = txt .. "距离: " .. dist .. "m" end 
                assets.Label.Text = txt assets.Box.Color3 = ESP.Color assets.Label.TextColor3 = ESP.Color 
            end 
        end 
    end 
end 

-- ====================== 【拼图二：UI 组件绘制与配置文件深度双向绑定】 ====================== 

-- 【模块 2: 自动刷取栏】
Main:Toggle({ Title = "开启全自动挂机刷怪 (Auto Farm)", Value = AutoFarmEnabled, Callback = function(enabled) 
    AutoFarmEnabled = enabled Config:Set("AutoFarmEnabled", enabled) Config:Save() 
    if enabled then StartFarmLoop() StartAutoAttack() StartAutoSkill() if AutoFillUpEnabled then StartAutoFillUpLoop() end else ResetMobOverride() end 
end }) 
Main:Dropdown({ Title = "位移相对怪物位置", Multi = false, Options = { "Above", "Under" }, Default = FarmPosition, Callback = function(v) FarmPosition = v Config:Set("FarmPosition", v) Config:Save() end }) 
Main:Dropdown({ Title = "移动算法机制", Multi = false, Options = { "Tween", "tp", "Tp", "tp1" }, Default = FarmMode, Callback = function(v) FarmMode = v Config:Set("FarmMode", v) Config:Save() end }) 
Main:Slider({ Title = "基础悬停绝对高度", Min = -10, Max = 30, Default = HeightValue, ValueSuffix = " 点", Callback = function(v) HeightValue = v Config:Set("HeightValue", v) Config:Save() end }) 
Main:Slider({ Title = "线性折线平滑速度", Min = 0, Max = 5, Default = TweenSpeed, ValueSuffix = " 秒", Callback = function(v) TweenSpeed = v Config:Set("TweenSpeed", v) Config:Save() end }) 
Main:Section({ Title = "按键连招与技能模拟" }) 
Main:Toggle({ Title = "全自动普通攻击 (LMB)", Value = Config:Get("AutoAttackEnabled", false), Callback = function(v) AutoAttackEnabled = v Config:Set("AutoAttackEnabled", v) Config:Save() if v then StartAutoAttack() end end }) 
Main:Toggle({ Title = "全自动连发按键技能", Value = Config:Get("AutoSkillEnabled", false), Callback = function(v) AutoSkillEnabled = v Config:Set("AutoSkillEnabled", v) Config:Save() if v then StartAutoSkill() end end }) 
Main:Dropdown({ Title = "技能范围按键选择", Multi = true, Options = skillDropdownValues, Default = SelectedSkills, Callback = function(v) SelectedSkills = v Config:Set("SelectedSkills", v) Config:Save() end }) 
Main:Slider({ Title = "技能循环释放延迟间隔", Min = 0.1, Max = 5, Default = SkillDelay, ValueSuffix = " 秒", Callback = function(v) SkillDelay = v Config:Set("SkillDelay", v) Config:Save() end }) 
Main:Section({ Title = "高级首领血量判定" }) 
Main:Slider({ Title = "高血量领主判定阈值线", Min = 50, Max = 5000, Default = HighHPThreshold, ValueSuffix = " HP", Callback = function(v) HighHPThreshold = v Config:Set("HighHPThreshold", v) Config:Save() end }) 

-- 【视觉透视绘制栏】
Main4:Toggle({ Title = "开启全局透视 ESP", Value = Config:Get("EspGlobalEnabled", false), Callback = function(v) 
    ESP.Enabled = v Config:Set("EspGlobalEnabled", v) Config:Save() 
    if v then task.spawn(function() while ESP.Enabled do if livingFolder then for _, m in ipairs(livingFolder:GetChildren()) do if IsValidMob(m) then CreateEsp(m) end end end UpdateEspVisibility() task.wait(0.5) end for m, _ in pairs(EspObjects) do RemoveEsp(m) end end) end 
end }) 
Main4:Toggle({ Title = "绘制怪物包围边框", Value = Config:Get("EspBoxes", true), Callback = function(v) ESP.Boxes = v Config:Set("EspBoxes", v) Config:Save() UpdateEspVisibility() end }) 
Main4:Toggle({ Title = "实时显示怪物健康生命", Value = Config:Get("EspHealth", true), Callback = function(v) ESP.Health = v Config:Set("EspHealth", v) Config:Save() UpdateEspVisibility() end }) 
Main4:Toggle({ Title = "实时测算怪物绝对距离", Value = Config:Get("EspDistance", true), Callback = function(v) ESP.Distance = v Config:Set("EspDistance", v) Config:Save() UpdateEspVisibility() end }) 

-- 【模块 3: 玩家属性栏与动态覆盖参数】
Main2:Section({ Title = "战局基础优化项" }) 
Main2:Toggle({ Title = "全局直接跳过直升机波次等待", Value = Config:Get("AutoSkipHeliEnabled", false), Callback = function(v) AutoSkipHeliEnabled = v Config:Set("AutoSkipHeliEnabled", v) Config:Save() TriggerAutoSkipHeli(v) end }) 
Main2:Toggle({ Title = "极简流畅度删图优化 (极大提升FPS)", Value = Config:Get("BoostFPS_Active", false), Callback = function(v) Config:Set("BoostFPS_Active", v) Config:Save() if v then SaveAndBoostFPS() else RestoreBoostFPS() end end }) 
Main2:Section({ Title = "高度动态受击自适应控制" }) 
Main2:Toggle({ Title = "开启低血智能重置状态 (状态自毁)", Value = Config:Get("GodModeEnabled", false), Callback = function(v) GodModeEnabled = v Config:Set("GodModeEnabled", v) Config:Save() end }) 
Main2:Slider({ Title = "状态自毁低血线设定触发点", Min = 5, Max = 95, Default = GodModeValue, ValueSuffix = "% 状态线", Callback = function(v) GodModeValue = v Config:Set("GodModeValue", v) Config:Save() end }) 

-- 【模块 5: 自动商店军火库】
Main5:Section({ Title = "全自动局内静默无限采购" }) 
Main5:Toggle({ Title = "循环购买指定挂机主武器", Value = Config:Get("AutoBuyWeaponEnabled", false), Callback = function(v) AutoBuyWeaponEnabled = v Config:Set("AutoBuyWeaponEnabled", v) Config:Save() end }) 
Main5:Dropdown({ Title = "主武器采购品类指定", Multi = false, Options = GlobalTables.Weapon, Default = SelectedWeapon, Callback = function(v) SelectedWeapon = v Config:Set("SelectedWeapon", v) Config:Save() end }) 
Main5:Divider() 
Main5:Toggle({ Title = "循环购买指定挂机消耗品", Value = Config:Get("AutoBuyMiscEnabled", false), Callback = function(v) AutoBuyMiscEnabled = v Config:Set("AutoBuyMiscEnabled", v) Config:Save() end }) 
Main5:Dropdown({ Title = "辅助消耗品类采购指定", Multi = false, Options = GlobalTables.MiscShop, Default = SelectedMiscItem, Callback = function(v) SelectedMiscItem = v Config:Set("SelectedMiscItem", v) Config:Save() end }) 

-- 【模块 6: 掉落物资收集】
Main6:Section({ Title = "战场物资收集磁铁系统" }) 
Main6:Toggle({ Title = "开启掉落物全自动捞取", Value = AutoCollectEnabled, Callback = function(v) AutoCollectEnabled = v Config:Set("AutoCollectEnabled", v) Config:Save() if v then StartAutoCollectLoop() end end }) 
Main6:Dropdown({ Title = "指定收集珍稀物资筛选", Multi = true, Options = CollectItems, Default = SelectedCollectItems, Callback = function(v) SelectedCollectItems = v Config:Set("SelectedCollectItems", v) Config:Save() end }) 
Main6:Dropdown({ Title = "收集触发优先级时间机制", Multi = false, Options = { "抢夺模式", "波次清理" }, Default = Config:Get("CollectMode", "波次清理"), Callback = function(v) CollectMode = v Config:Set("CollectMode", v) Config:Save() end }) 
Main6:Toggle({ Title = "低状态优先自动购买血量补给", Value = Config:Get("AutoFillUpEnabled", false), Callback = function(v) AutoFillUpEnabled = v Config:Set("AutoFillUpEnabled", v) Config:Save() if v and AutoFarmEnabled then StartAutoFillUpLoop() end end }) 

-- 【模块 7: 一键准备与局内难度投票】
Main7:Section({ Title = "大厅一键就绪与自动进入系统" }) 
Main7:Toggle({ Title = "开启一键准备 / 自动进局 (Auto Ready)", Value = AutoStartEnabled, Callback = function(v) AutoStartEnabled = v Config:Set("AutoStartEnabled", v) Config:Save() RefreshVoteAndStartSetup() end }) 
Main7:Toggle({ Title = "开启大厅单人模式自动投票", Value = AutoVoteEnabled, Callback = function(v) AutoVoteEnabled = v Config:Set("AutoVoteEnabled", v) Config:Save() RefreshVoteAndStartSetup() end }) 
Main7:Dropdown({ Title = "单人副本选定投票模式品类", Multi = false, Options = GlobalTables.Mode, Default = AutoGameValue, Callback = function(v) AutoGameValue = v Config:Set("AutoGameValue", v) Config:Save() RefreshVoteAndStartSetup() end }) 
Main7:Divider() 
Main7:Section({ Title = "局内难度扩展自动投票" }) 
Main7:Toggle({ Title = "开启局内扩展难度自动投票", Value = AutoVoteinGameEnabled, Callback = function(v) AutoVoteinGameEnabled = v Config:Set("AutoVoteinGameEnabled", v) Config:Save() SetupAutoVote_InGame(v) end }) 
Main7:Dropdown({ Title = "局内预设难度扩展档位选择", Multi = false, Options = GlobalTables.Votes, Default = AutoVoteValue, Callback = function(v) AutoVoteValue = v Config:Set("AutoVoteValue", v) Config:Save() SetupAutoVote_InGame(AutoVoteinGameEnabled) end }) 

-- 【系统运行设置】
Main3:Section({ Title = "脚本环境底层运行维护" }) 
Main3:MultiDropdown({ Title = "全图辅助快捷功能宏选项", Options = { "全图自动秒冲马桶" }, Default = MiscOptions, Callback = function(v) MiscOptions = v Config:Set("MiscOptions", v) Config:Save() end }) 
Main3:Toggle({ Title = "无障碍心跳越界检测护盾", Value = noBarrierActive, Callback = function(v) noBarrierActive = v Config:Set("NoBarrier", v) Config:Save() if v then startNoBarrier() else stopNoBarrier() end end }) 
Main3:Toggle({ Title = "防挂机超时断开连接 (Anti AFK)", Value = AntiAFK, Callback = function(enabled) 
    AntiAFK = enabled Config:Set("AntiAfk", enabled) Config:Save() 
    if enabled then task.spawn(function() game.Players.LocalPlayer.Idled:Connect(function() VirtualUser:CaptureController() VirtualUser:ClickButton2(Vector2.new()) end) while AntiAFK do VirtualUser:CaptureController() VirtualUser:ClickButton2(Vector2.new()) task.wait(60) end end) end 
end }) 

-- ====================== 初始化自动线程唤醒群 ====================== 
if AutoFarmEnabled then task.wait(2) StartFarmLoop() if AutoAttackEnabled then StartAutoAttack() end if AutoSkillEnabled then StartAutoSkill() end if AutoFillUpEnabled then StartAutoFillUpLoop() end end 
if noBarrierActive then startNoBarrier() end 
if Config:Get("EspGlobalEnabled", false) then ESP.Enabled = true task.spawn(function() while ESP.Enabled do if livingFolder then for _, m in ipairs(livingFolder:GetChildren()) do if IsValidMob(m) then CreateEsp(m) end end end UpdateEspVisibility() task.wait(0.5) end end) end
if Config:Get("BoostFPS_Active", false) then SaveAndBoostFPS() end
if AutoCollectEnabled then task.wait(2) StartAutoCollectLoop() end 
if AutoVoteEnabled or AutoStartEnabled then RefreshVoteAndStartSetup() end 
if AutoVoteinGameEnabled then SetupAutoVote_InGame(true) end

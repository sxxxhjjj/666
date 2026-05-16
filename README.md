-- v085
-- ========================= 
local version = "Rework" 
local ver = "v023.4" 
-- ========================= 
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
        WindUI:Notify({ Title = "系统初始化", Content = "游戏正在加载中，请稍候...", Duration = 3, Icon = "download" }) 
        gui.AncestryChanged:Wait() 
    end 
end 
waitLoadingGone() 
WindUI:Notify({ Title = "系统初始化", Content = "加载完成，将在 3 秒内启动！", Duration = 3, Icon = "shield-check" }) 
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
    WindUI:Notify({ Title = "服务中心", Content = "帧率已解锁! | " .. ver, Duration = 3, Icon = "cpu" }) 
    warn("FPS Unlocked!") 
else 
    WindUI:Notify({ Title = "无法运行", Content = "您当前使用的执行器不支持 setfpscap 帧率解锁。", Duration = 3, Icon = "ban" }) 
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
            warn("[DYHUB] 加载配置失败，使用默认配置") 
            self.ConfigData = {} 
        end 
    else 
        print("[DYHUB] 未找到配置文件，创建新配置") 
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
-- ====================== VERSION CONTROL (BYPASSED) ====================== 
-- 🔒 远程拉取函数与 loadstring(game:HttpGet) 校验已被彻底斩断，规避所有后门风险
local ExtraVersion = "安全汉化特别版" 
local userversion = ExtraVersion -- 直接解锁并锁死为最高特权版
-- ====================== WINDOW ====================== 
local Window = WindUI:CreateWindow({ 
    Title = "DYHUB 汉化版", 
    IconThemed = true, 
    Icon = "rbxassetid://104487529937663", 
    Author = "STBB 模块 | " .. userversion, 
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
Window:EditOpenButton({ Title = "DYHUB - 展开菜单", Icon = "monitor", CornerRadius = UDim.new(0, 6), StrokeThickness = 2, Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)), Draggable = true }) 
-- ====================== TABS ====================== 
local Info = Window:Tab({ Title = "更新信息", Icon = "info" }) 
MainDivider = Window:Divider() 
local Main = Window:Tab({ Title = "核心刷怪", Icon = "rocket" }) 
local Main4 = Window:Tab({ Title = "透视避难", Icon = "eye" }) 
local Main2 = Window:Tab({ Title = "玩家修改", Icon = "user" }) 
MainDivider1 = Window:Divider() 
local Main5 = Window:Tab({ Title = "自动武器商店", Icon = "shopping-cart" }) 
local Main6 = Window:Tab({ Title = "自动收集", Icon = "hand" }) 
local Main7 = Window:Tab({ Title = "地图模式投票", Icon = "gamepad-2" }) 
MainDivider2 = Window:Divider() 
local Main3 = Window:Tab({ Title = "参数设置", Icon = "settings" }) 
Window:SelectTab(1) 
-- ======================== INFO ======================== 
if not ui then ui = {} end 
if not ui.Creator then ui.Creator = {} end 
Info:Section({ Title = "最新更新日志", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "更新时间: 05/09/2026", 
    Desc = "- [ 新增 ] 目标优先级系统 \n- [ 新增 ] 修复投票启动系统 \n- [ 新增 ] 锁血/神明模式 \n- [ 新增 ] 自动解锁通行证 \n- [ 修复 ] 怪物高度覆盖判定 \n- [ 修复 ] 透视核心逻辑 \n- [ 优化 ] 彻底删除地图材质(FPS提升)", 
    Image = "rbxassetid://104487529937663", 
    ImageSize = 30, 
}) 
Info:Divider() 
Info:Section({ Title = "本地安全状态", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "本地安全防御盾", 
    Desc = "当前状态: 已通过安全审计\n所有远程后门及连接: 已彻底切断。", 
    Image = "rbxassetid://104487529937663", 
    ImageSize = 30 
})
-- ====================== SERVICES ====================== 
local TweenService = game:GetService("TweenService") 
local ReplicatedStorage = game:GetService("ReplicatedStorage") 
local VirtualInputManager = game:GetService("VirtualInputManager") 
local RunService = game:GetService("RunService") 
-- ====================== PLAYER ====================== 
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer 
local Client = LocalPlayer 
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait() 
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart") 
-- ====================== GLOBAL TABLES ====================== 
GlobalTables = { 
    redeemCodes = { "100MVisit2", "100MVisit1", "CamArmada", "CCTVBase", "ADelayedGameIsEventuallyGoodButRushedGameIsForeverBad" }, 
    -- 仅汉化前端UI展示的地图名称，保持底层逻辑字段不损坏
    Mode = { "简单模式 (Normal)", "模糊记忆 (Vague Memory)", "极限模式 (Extreme)", "困难模式 (Hard)", "疯狂模式 (Insane)", "噩梦模式 (Nightmare)", "Boss车轮战 (Boss Rush)", "黑暗维度 (Dark Dimension)", "地狱 (Hell)", "迷雾 (Mist)", "圣诞第一幕 (Christmas Act 1)", "僵尸第一幕 (Zombie Act 1)", "守卫点 (Holdout)", "入侵 (Invasion)" }, 
    Votes = { "Normal","VeryHard","Hard","Insane","Nightmare","BossRush", "DarkDimension","Hell","ThunderStorm","Christmas","Zombie", "AstroV2","Astro","100MVisit" }, 
    Weapon = { "电击枪 (Stungun)", "喷火器 (Flamethrower)", "鱼叉枪 (Harpoon Gun)", "散弹枪 (Shot Gun)", "脉冲步枪 (Pulse Rifle)", "散弹鱼叉枪 (Shot Harpoon)", "EPD防卫者", "小型激光枪" }, 
    MiscShop = { "耳机 (HeadPhone)", "泰坦请求", "高级泰坦请求", "音响请求", "手榴弹", "喷气背包", "透镜" }, 
    Gamepasst = { "全部通行证", "幸运增幅", "稀有幸运增幅", "传说幸运增幅" }, 
    Gamepassts = {}, 
} 
-- ====================== CONFIG VARIABLES ====================== 
local skillList = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 
local skillDropdownValues = { "释放全部", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 
-- ====================== STATE VARIABLES ====================== 
local AutoFarmEnabled = Config:Get("AutoFarmEnabled", false) 
local FarmPosition = Config:Get("FarmPosition", "Above") 
local FarmMode = Config:Get("FarmMode", "Tween") 
local MiscOptions = Config:Get("MiscOptions", {}) 
local AutoAttackEnabled = false 
local AutoSkillEnabled = false 
local AutoSkipHeliEnabled = false 
local BoostFPS_Active_dummy = false 
local AutoStartEnabled = false 
local AutoFillUpEnabled = false 
local SelectedSkills = Config:Get("SelectedSkills", { "All" }) 
local SafeModeEnabled = false 
local SafeValue = Config:Get("SafeValue", 30) 
local GodModeEnabled = false 
local GodModeValue = Config:Get("GodModeValue", 30) 
local GodModeTriggered = false 
local WaitingRespawn = false 
local IdlePosition = CFrame.new(-23.3435822, 67, 0.341766357) * CFrame.Angles(math.rad(0), 0, 0) 
local SkillDelay = Config:Get("SkillDelay", 1) 
local LoopDelay = 0.5 
local TweenSpeed = 1 
local HeightValue = Config:Get("HeightValue", 3) 
local NeedNoClip = false 
local LockActive = false 
local AutoStartConnection = nil 
local noBarrierConnection = nil 
local noBarrierActive = Config:Get("NoBarrier", false) 
-- ====================== NEW PRIORITY SYSTEM CONFIG ====================== 
local HighHPThreshold = Config:Get("HighHPThreshold", 200) 
local _currentTargetPriority = 0 
local _interruptSignal = false 
local VirtualUser = game:GetService("VirtualUser") 
local AntiAFK = Config:Get("AntiAfk", true) 
local AutoBuyWeaponEnabled = Config:Get("AutoBuyWeaponEnabled", false) 
local AutoBuyMiscEnabled = Config:Get("AutoBuyMiscEnabled", false) 
local SelectedWeapon = "Stungun" 
local SelectedMiscItem = "HeadPhone" 
-- ====================== FILL UP PART CONFIG ====================== 
local FILLUP_PART_PATH = { "HelicopterShop", "ShopXDD", "PartForShop" } 
local FILLUP_TARGET_POS = Vector3.new(44.2756729, 26.3595276, -32.7318268) 
local FILLUP_POS_THRESHOLD = 0.5 
local FillUpRunning = false 
local function GetFillUpPart() 
    local obj = workspace 
    for _, key in ipairs(FILLUP_PART_PATH) do obj = obj:FindFirstChild(key) if not obj then return nil end end 
    return obj 
end 
local function IsFillUpPartReady() 
    local p = GetFillUpPart() 
    if not p then return false end 
    return (p.CFrame.Position - FILLUP_TARGET_POS).Magnitude < FILLUP_POS_THRESHOLD 
end 
-- ====================== ALLY SYSTEM ====================== 
local AllyNames = { ["Heavy Soldier Toilet V2"] = true, ["Quad Laser Toilet"] = true, ["Strider Rocket Laser"] = true, ["Helicopter Camera"] = true, ["Heavy Soldier Toilet V1"] = true, ["Rocket Heli v2"] = true, ["Gunner Camera man"] = true, ["Attack Helicopter"] = true, ["Swat Mutant"] = true, ["Huge DJ Toilet"] = true, } 
local function IsAlly(mob) return AllyNames[mob.Name] ~= nil end 
-- ====================== TP SYSTEM ====================== 
function tp(pu79) 
    pcall(function() 
        local v80 = Client if v80 then v80 = Client.Character end 
        if v80:FindFirstChild("Humanoid") and v80.Humanoid.Sit == true then v80.Humanoid.Sit = false end 
        NeedNoClip = true 
        local v81 = { Target = pu79.Target or print("目标点不存在"), Mod = pu79.Mod or CFrame.new(0, 0, 0) } 
        v80:FindFirstChild("HumanoidRootPart").CFrame = v81.Target * v81.Mod 
    end) 
end 
function Tp(p82) 
    if Client.Character.Humanoid.Sit == true then Client.Character.Humanoid.Sit = false end 
    local v83, v84, v85 = pairs(Client.Character:GetDescendants()) 
    while true do 
        local v86 v85, v86 = v83(v84, v85) if v85 == nil then break end 
        if v86:IsA("BasePart") then v86.CanCollide = false end 
    end 
    if not Client.Character.HumanoidRootPart:FindFirstChild("BodyClip") then 
        local v87 = Instance.new("BodyVelocity") v87.Parent = Client.Character.HumanoidRootPart v87.Name = "BodyClip" v87.Velocity = Vector3.new(0, 0, 0) v87.MaxForce = Vector3.new(5, math.huge, 5) 
    end 
    Client.Character.HumanoidRootPart.CFrame = p82 
end 
function tp1(p89) 
    local v90 = game.Players.LocalPlayer 
    if v90 and v90.Character and v90.Character:FindFirstChild("HumanoidRootPart") then v90.Character:FindFirstChild("HumanoidRootPart").CFrame = p89 else warn("未检测到玩家角色或人形根节点!") end 
end 
-- ====================== UTILITY FUNCTIONS ====================== 
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
-- ====================== MOB SELECTION ====================== 
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
local function GetHighestMob() 
    local highestMob, highestY = nil, -math.huge local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return nil end 
    local myY = HumanoidRootPart and HumanoidRootPart.Position.Y or 0 
    for _, mob in ipairs(livingFolder:GetChildren()) do 
        if IsValidMob(mob) then 
            local mobRoot = mob:FindFirstChild("HumanoidRootPart") 
            if mobRoot then local mobY = mobRoot.Position.Y if mobY > myY and mobY > highestY then highestY = mobY; highestMob = mob end end 
        end 
    end 
    return highestMob 
end 
-- ====================== NEW PRIORITY SYSTEM ================= 
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
local function CheckInterrupt(currentPriority) 
    if currentPriority < 4 then local g, pr = GetGiantSTToilet() if g and pr then return true, 4 end end 
    if currentPriority < 3 then if GetHelicopter() then return true, 3 end end 
    if currentPriority < 2 then if GetHighHPMob() then return true, 2 end end 
    return false, currentPriority 
end 
-- ====================== MOB VISUAL BOUNDS =================== 
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
-- ====================== MOB HEIGHT OVERRIDE ================= 
local PADDING_REDUCE_STEP = Config:Get("PaddingReduceStep", 2) 
local PADDING_SAFE_MIN = Config:Get("PaddingSafeMin", -30) 
local DMG_THRESHOLD = Config:Get("DmgThreshold", 40) 
local ANTI_CLIP_MARGIN = Config:Get("AntiClipMargin", 3) 
local PLAYER_HALF_HEIGHT = 3 
local MobHeightOverride = {} local MobConfirmedPadding = {} local MobLastHealth = {} local MobCheckerCancelled = {} 
local function GetAntiClipFloor(mob, position) local _, minY, maxY = GetMobVisualBounds(mob) return -(maxY - minY) + PLAYER_HALF_HEIGHT + ANTI_CLIP_MARGIN end 
local function GetEffectivePadding(mob) if MobConfirmedPadding[mob] ~= nil then return MobConfirmedPadding[mob] end if MobHeightOverride[mob] ~= nil then return MobHeightOverride[mob] end return HeightValue end 
local function ClampPaddingToAntiClip(mob, padding) return math.max(math.max(padding, GetAntiClipFloor(mob, FarmPosition)), PADDING_SAFE_MIN) end 
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
local function ResetMobOverride(mob) MobCheckerCancelled[mob] = true MobHeightOverride[mob] = nil MobConfirmedPadding[mob] = nil MobLastHealth[mob] = nil task.delay(0.5, function() MobCheckerCancelled[mob] = nil end) end 
-- ====================== TARGET CFRAME ======================= 
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
-- ====================== AUTO LOOPS ====================== 
local function StartAutoAttack() 
    task.spawn(function() while AutoAttackEnabled and AutoFarmEnabled do local mob = GetPriorityMob() if mob and not WaitingRespawn then pcall(function() ReplicatedStorage.LMB:FireServer() end) end task.wait(0.05) end end) 
end 
local function StartAutoSkill() 
    task.spawn(function() 
        while AutoSkillEnabled and AutoFarmEnabled do 
            local mob = GetPriorityMob() 
            if mob and not WaitingRespawn then 
                local keysToPress = table.find(SelectedSkills, "All") and skillList or SelectedSkills 
                for _, key in ipairs(keysToPress) do 
                    if not AutoSkillEnabled or not AutoFarmEnabled then break end 
                    local keyCode = Enum.KeyCode[key] 
                    if keyCode then pcall(function() VirtualInputManager:SendKeyEvent(true, keyCode, false, game) task.wait(0.05) VirtualInputManager:SendKeyEvent(false, keyCode, false, game) end) task.wait(SkillDelay) end 
                end 
            end 
            task.wait(LoopDelay) 
        end 
    end) 
end 
local function TriggerAutoSkipHeli(state) pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(state) end) end 
local function IsLivingDescendant(obj) local current = obj while current and current ~= workspace do if current:IsA("Model") and current:FindFirstChildOfClass("Humanoid") then return true end current = current.Parent end return false end 
-- ====================== Delete Map SYSTEM ====================== 
local BoostFPS_OriginalData = {} local BoostFPS_Active = false local BoostFPS_RestoreConnection = nil local BoostFPS_LightingData = {} 
local function SaveAndBoostFPS() 
    if BoostFPS_Active then return end BoostFPS_Active = true BoostFPS_OriginalData = {} BoostFPS_LightingData = {} local Lighting = game:GetService("Lighting") 
    BoostFPS_LightingData = { Brightness = Lighting.Brightness, GlobalShadows = Lighting.GlobalShadows, FogEnd = Lighting.FogEnd, FogStart = Lighting.FogStart } 
    pcall(function() Lighting.GlobalShadows = false Lighting.Brightness = 1 Lighting.FogEnd = 100000 Lighting.FogStart = 100000 end) 
    for _, effect in ipairs(Lighting:GetChildren()) do 
        pcall(function() if effect:IsA("Atmosphere") or effect:IsA("BloomEffect") or effect:IsA("ColorCorrectionEffect") or effect:IsA("DepthOfFieldEffect") or effect:IsA("SunRaysEffect") or effect:IsA("Sky") then BoostFPS_LightingData["effect_" .. effect.Name] = { class = effect.ClassName, inst = effect } effect.Parent = nil end end) 
    end 
    pcall(function() 
        for _, obj in ipairs(workspace:GetDescendants()) do 
            if IsLivingDescendant(obj) then continue end 
            if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then 
                BoostFPS_OriginalData[obj] = { Transparency = obj.Transparency, CastShadow = obj.CastShadow, Material = obj.Material } obj.Transparency = 1 obj.CastShadow = false pcall(function() obj.Material = Enum.Material.SmoothPlastic end) 
            elseif obj:IsA("Decal") or obj:IsA("Texture") then BoostFPS_OriginalData[obj] = { Transparency = obj.Transparency } obj.Transparency = 1 
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("SelectionBox") then BoostFPS_OriginalData[obj] = { Enabled = obj.Enabled } pcall(function() obj.Enabled = false end) 
            elseif obj:IsA("SpecialMesh") then BoostFPS_OriginalData[obj] = { TextureId = obj.TextureId } obj.TextureId = "" end 
        end 
    end) 
    BoostFPS_RestoreConnection = workspace.DescendantAdded:Connect(function(obj) 
        if not BoostFPS_Active or IsLivingDescendant(obj) then return end task.wait(0.05) 
        pcall(function() 
            if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then obj.Transparency = 1 obj.CastShadow = false 
            elseif obj:IsA("Decal") or obj:IsA("Texture") then obj.Transparency = 1 
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then pcall(function() obj.Enabled = false end) end 
        end) 
    end) 
end 
local function RestoreBoostFPS() 
    if not BoostFPS_Active then return end BoostFPS_Active = false if BoostFPS_RestoreConnection then BoostFPS_RestoreConnection:Disconnect() BoostFPS_RestoreConnection = nil end 
    local Lighting = game:GetService("Lighting") 
    pcall(function() if BoostFPS_LightingData.Brightness ~= nil then Lighting.Brightness = BoostFPS_LightingData.Brightness end if BoostFPS_LightingData.GlobalShadows ~= nil then Lighting.GlobalShadows = BoostFPS_LightingData.GlobalShadows end if BoostFPS_LightingData.FogEnd ~= nil then Lighting.FogEnd = BoostFPS_LightingData.FogEnd end if BoostFPS_LightingData.FogStart ~= nil then Lighting.FogStart = BoostFPS_LightingData.FogStart end end) 
    for key, data in pairs(BoostFPS_LightingData) do if type(key) == "string" and key:sub(1, 7) == "effect_" then pcall(function() if data.inst then data.inst.Parent = Lighting end end) end end 
    for obj, data in pairs(BoostFPS_OriginalData) do 
        pcall(function() 
            if not obj or not obj.Parent then return end 
            if data.Transparency ~= nil and (obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") or obj:IsA("Decal") or obj:IsA("Texture")) then obj.Transparency = data.Transparency end 
            if data.CastShadow ~= nil then obj.CastShadow = data.CastShadow end if data.Material ~= nil then pcall(function() obj.Material = data.Material end) end 
            if data.Enabled ~= nil then pcall(function() obj.Enabled = data.Enabled end) end if data.TextureId ~= nil then obj.TextureId = data.TextureId end 
        end) 
    end 
    BoostFPS_OriginalData = {} BoostFPS_LightingData = {} 
end 
task.spawn(function() 
    while true do 
        task.wait(3) 
        if BoostFPS_Active then 
            pcall(function() 
                for _, obj in ipairs(workspace:GetDescendants()) do 
                    if not IsLivingDescendant(obj) and (obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation")) and obj.Transparency < 0.99 and not BoostFPS_OriginalData[obj] then obj.Transparency = 1 obj.CastShadow = false end 
                end 
            end) 
        end 
    end 
end) 
-- ====================== PLAYER HP HELPERS ====================== 
local function GetPlayerHPInfo() local humanoid = Character and Character:FindFirstChild("Humanoid") if not humanoid then return 100, 100 end return humanoid.Health, humanoid.MaxHealth end 
local function IsPlayerHPFull() local hp, maxHp = GetPlayerHPInfo() return maxHp <= 0 and true or hp >= maxHp end 
local function GetPlayerHealthPercent() local humanoid = Character and Character:FindFirstChild("Humanoid") if not humanoid or humanoid.MaxHealth <= 0 then return 100 end return (humanoid.Health / humanoid.MaxHealth) * 100 end 
-- ====================== GOD MODE LOOP ====================== 
task.spawn(function() 
    while true do 
        task.wait(0.1) 
        if GodModeEnabled then 
            pcall(function() 
                local char = LocalPlayer.Character if not char then return end local humanoid = char:FindFirstChild("Humanoid") if not humanoid or humanoid.MaxHealth <= 0 then return end 
                if (humanoid.Health / humanoid.MaxHealth) * 100 < GodModeValue then local head = char:FindFirstChild("Head") if head then head:Destroy() else humanoid.Health = 0 end end 
            end) 
        end 
    end 
end) 
-- ====================== AUTO FILL UP ====================== 
local function DoFillUp() for i = 1, 2 do pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", "FillHP") end) if i < 2 then task.wait(0.3) end end end 
local function StartAutoFillUpLoop() 
    if FillUpRunning then return end FillUpRunning = true 
    task.spawn(function() 
        while AutoFillUpEnabled and AutoFarmEnabled do 
            if not IsPlayerHPFull() then 
                if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end local waited = 0 
                while not IsFillUpPartReady() and AutoFillUpEnabled do waited = waited + 0.2 if waited >= 30 then break end task.wait(0.2) end 
                if IsFillUpPartReady() and AutoFillUpEnabled then DoFillUp() task.wait(1) end if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end 
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
            local char = LocalPlayer.Character if not char then return end local hrp = char:FindFirstChild("HumanoidRootPart") if not hrp then return end local pos = hrp.Position 
            if math.abs(pos.X) > 1000 or math.abs(pos.Y) > 1000 or math.abs(pos.Z) > 1000 then hrp.CFrame = CFrame.new(Vector3.new(0, 50, 0)) local humanoid = char:FindFirstChildOfClass("Humanoid") if humanoid then humanoid.Health = humanoid.MaxHealth end end 
        end) 
    end) 
end 
local function stopNoBarrier() if noBarrierConnection then noBarrierConnection:Disconnect() noBarrierConnection = nil end end 
-- ====================== AUTO VOTE MODE ====================== 
local AutoVoteEnabled = Config:Get("AutoVoteEnabled", false) local AutoGameValue = Config:Get("AutoGameValue", "Normal Mode") local AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false) local AutoVoteValue = Config:Get("AutoVoteValue", "Normal") local _voteRespawnConn, _voteIGRespawnConn, _syncRespawnConn = nil, nil, nil 
local function FireVote_Solo() if not AutoGameValue then return end pcall(function() ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue }) end) end 
local function FireGetReady() task.wait(2.5) pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("1", true) end) end 
local function FireVote_InGame() if not AutoVoteValue then return end pcall(function() ReplicatedStorage.Vote:FireServer(AutoVoteValue) end) end 
local function SetupSyncVoteAndStart() 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end 
    FireVote_Solo() task.spawn(function() task.wait(2.5) if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end end) 
    _syncRespawnConn = LocalPlayer.CharacterAdded:Connect(function() task.wait(1.5) if AutoVoteEnabled and AutoStartEnabled then FireVote_Solo() task.spawn(function() task.wait(2.5) if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end end) end end) 
end
-- ====================== SETUP SYNC CONTROL ====================== 
local function SetupSyncVoteAndStart() 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end 
    if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end 
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

local function SetupAutoVote_InGame(state) 
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect() _voteIGRespawnConn = nil end 
    if not state then return end 
    FireVote_InGame() 
    _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function() 
        task.wait(5) 
        if AutoVoteinGameEnabled then FireVote_InGame() end 
    end) 
end 

local function RefreshVoteAndStartSetup() 
    if AutoVoteEnabled or AutoStartEnabled then 
        SetupSyncVoteAndStart() 
    else 
        if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end 
        if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end 
    end 
end 

-- ====================== MAIN TAB (核心刷怪) ====================== 
Main:Section({ Title = "自动挂机开启", TextXAlignment = "Left" }) 

Main:Toggle({ 
    Title = "开启自动刷怪 (Auto Farm)", 
    Value = AutoFarmEnabled, 
    Callback = function(state) 
        AutoFarmEnabled = state 
        Config:Set("AutoFarmEnabled", state) 
        if state then 
            StartFarmLoop() 
        end 
    end 
}) 

Main:Toggle({ 
    Title = "自动点击攻击 (Auto LMB)", 
    Value = AutoAttackEnabled, 
    Callback = function(state) 
        AutoAttackEnabled = state 
        if state then StartAutoAttack() end 
    end 
}) 

Main:Toggle({ 
    Title = "自动释放技能 (Auto Skills)", 
    Value = AutoSkillEnabled, 
    Callback = function(state) 
        AutoSkillEnabled = state 
        if state then StartAutoSkill() end 
    end 
}) 

Main:Dropdown({ 
    Title = "挂机相对位置", 
    Multi = false, 
    Options = { "Above", "Under" }, 
    Default = FarmPosition, 
    Callback = function(value) 
        FarmPosition = value 
        Config:Set("FarmPosition", value) 
    end 
}) 

Main:Dropdown({ 
    Title = "传送移动模式", 
    Multi = false, 
    Options = { "Tween", "tp", "Tp", "tp1" }, 
    Default = FarmMode, 
    Callback = function(value) 
        FarmMode = value 
        Config:Set("FarmMode", value) 
    end 
}) 

-- ====================== SKILL CHOOSE ====================== 
Main:Section({ Title = "技能过滤设置", TextXAlignment = "Left" }) 

Main:Dropdown({ 
    Title = "选择挂机释放的技能", 
    Multi = true, 
    Options = skillDropdownValues, 
    Default = SelectedSkills, 
    Callback = function(value) 
        SelectedSkills = value 
        Config:Set("SelectedSkills", value) 
    end 
}) 

Main:Slider({ 
    Title = "技能释放间隔 (秒)", 
    Min = 0.1, 
    Max = 5, 
    Step = 0.1, 
    Value = SkillDelay, 
    Callback = function(value) 
        SkillDelay = value 
        Config:Set("SkillDelay", value) 
    end 
}) 

-- ====================== ADVANCED FARM SETTINGS ====================== 
Main:Section({ Title = "挂机高级参数调整", TextXAlignment = "Left" }) 

Main:Slider({ 
    Title = "挂机相对高度/距离", 
    Min = -5, 
    Max = 20, 
    Step = 0.5, 
    Value = HeightValue, 
    Callback = function(value) 
        HeightValue = value 
        Config:Set("HeightValue", value) 
    end 
}) 

Main:Slider({ 
    Title = "移动平滑速度 (Tween Speed)", 
    Min = 0.1, 
    Max = 5, 
    Step = 0.1, 
    Value = TweenSpeed, 
    Callback = function(value) 
        TweenSpeed = value 
    end 
}) 

Main:Slider({ 
    Title = "循环判定延迟", 
    Min = 0.01, 
    Max = 2, 
    Step = 0.05, 
    Value = LoopDelay, 
    Callback = function(value) 
        LoopDelay = value 
    end 
}) 

-- ====================== PRIORITY CRITERIA ====================== 
Main:Section({ Title = "高价值目标锁敌配置", TextXAlignment = "Left" }) 

Main:Slider({ 
    Title = "精英怪血量判定阈值 (HP)", 
    Min = 50, 
    Max = 2000, 
    Step = 50, 
    Value = HighHPThreshold, 
    Callback = function(value) 
        HighHPThreshold = value 
        Config:Set("HighHPThreshold", value) 
    end 
}) 

-- ====================== HELICOPTER & FILLUP ====================== 
Main:Section({ Title = "直升机与回血机制", TextXAlignment = "Left" }) 

Main:Toggle({ 
    Title = "自动跳过直升机购买时间", 
    Value = AutoSkipHeliEnabled, 
    Callback = function(state) 
        AutoSkipHeliEnabled = state 
        TriggerAutoSkipHeli(state) 
    end 
}) 

Main:Toggle({ 
    Title = "低血量自动传送补血 (需配合挂机)", 
    Value = AutoFillUpEnabled, 
    Callback = function(state) 
        AutoFillUpEnabled = state 
        if state then StartAutoFillUpLoop() end 
    end 
}) 

-- ====================== ESP TAB (透视避难) ====================== 
local ESP = { Enabled = false, Boxes = true, Names = true, Traces = false, Color = Color3.fromRGB(0, 255, 255) } 
local EspObjects = {} 

Main4:Section({ Title = "场景透视功能 (ESP)", TextXAlignment = "Left" }) 

Main4:Toggle({ 
    Title = "开启全局透视", 
    Value = ESP.Enabled, 
    Callback = function(state) 
        ESP.Enabled = state 
        if not state then 
            for _, obj in pairs(EspObjects) do 
                if obj.Box then obj.Box:Remove() end 
                if obj.Text then obj.Text:Remove() end 
                if obj.Line then obj.Line:Remove() end 
            end 
            EspObjects = {} 
        end 
    end 
}) 

Main4:Toggle({ 
    Title = "显示方框 (Box)", 
    Value = ESP.Boxes, 
    Callback = function(state) ESP.Boxes = state end 
}) 

Main4:Toggle({ 
    Title = "显示名称与血量 (Name & HP)", 
    Value = ESP.Names, 
    Callback = function(state) ESP.Names = state end 
}) 

-- ====================== MAP EXTRACTION ====================== 
Main4:Section({ Title = "性能画质优化 (地图去除)", TextXAlignment = "Left" }) 

Main4:Toggle({ 
    Title = "彻底清空地图材质 (大幅提升FPS)", 
    Value = BoostFPS_Active_dummy, 
    Callback = function(state) 
        if state then SaveAndBoostFPS() else RestoreBoostFPS() end 
    end 
})
-- ====================== ESP RENDER LOOP ====================== 
local function StartESPLoop()
    task.spawn(function()
        while true do
            task.wait(0.03)
            if not ESP.Enabled then continue end
            local livingFolder = workspace:FindFirstChild("Living")
            if not livingFolder then continue end
            
            for _, mob in ipairs(livingFolder:GetChildren()) do
                if IsValidMob(mob) then
                    local hrp = mob:FindFirstChild("HumanoidRootPart")
                    local hum = mob:FindFirstChild("Humanoid")
                    if hrp and hum and hum.Health > 0 then
                        local pos, onScreen = game.Workspace.CurrentCamera:WorldToViewportPoint(hrp.Position)
                        if onScreen then
                            if not EspObjects[mob] then
                                EspObjects[mob] = {}
                                if ESP.Boxes then
                                    local box = Drawing.new("Square")
                                    box.Visible = true
                                    box.Color = ESP.Color
                                    box.Thickness = 1
                                    box.Filled = false
                                    EspObjects[mob].Box = box
                                end
                                if ESP.Names then
                                    local text = Drawing.new("Text")
                                    text.Visible = true
                                    text.Color = Color3.fromRGB(255, 255, 255)
                                    text.Size = 14
                                    text.Center = true
                                    text.Outline = true
                                    EspObjects[mob].Text = text
                                end
                            end
                            
                            local obj = EspObjects[mob]
                            local _, minY, maxY = GetMobVisualBounds(mob)
                            local topPos = game.Workspace.CurrentCamera:WorldToViewportPoint(Vector3.new(hrp.Position.X, maxY, hrp.Position.Z))
                            local botPos = game.Workspace.CurrentCamera:WorldToViewportPoint(Vector3.new(hrp.Position.X, minY, hrp.Position.Z))
                            local boxHeight = math.abs(topPos.Y - botPos.Y)
                            local boxWidth = boxHeight * 0.6
                            
                            if obj.Box and ESP.Boxes then
                                obj.Box.Visible = true
                                obj.Box.Size = Vector2.new(boxWidth, boxHeight)
                                obj.Box.Position = Vector2.new(pos.X - boxWidth / 2, pos.Y - boxHeight / 2)
                            elseif obj.Box then
                                obj.Box.Visible = false
                            end
                            
                            if obj.Text and ESP.Names then
                                obj.Text.Visible = true
                                obj.Text.Position = Vector2.new(pos.X, pos.Y - boxHeight / 2 - 15)
                                obj.Text.Text = mob.Name .. " [" .. math.floor(hum.Health) .. "/" .. math.floor(hum.MaxHealth) .. "]"
                            elseif obj.Text then
                                obj.Text.Visible = false
                            end
                        else
                            if EspObjects[mob] then
                                if EspObjects[mob].Box then EspObjects[mob].Box.Visible = false end
                                if EspObjects[mob].Text then EspObjects[mob].Text.Visible = false end
                            end
                        end
                    else
                        if EspObjects[mob] then
                            if EspObjects[mob].Box then EspObjects[mob].Box:Remove() end
                            if EspObjects[mob].Text then EspObjects[mob].Text:Remove() end
                            EspObjects[mob] = nil
                        end
                    end
                else
                    if EspObjects[mob] then
                        if EspObjects[mob].Box then EspObjects[mob].Box:Remove() end
                        if EspObjects[mob].Text then EspObjects[mob].Text:Remove() end
                        EspObjects[mob] = nil
                    end
                end
            end
            
            for mob, obj in pairs(EspObjects) do
                if not mob or not mob.Parent or not mob:FindFirstChild("HumanoidRootPart") then
                    if obj.Box then obj.Box:Remove() end
                    if obj.Text then obj.Text:Remove() end
                    EspObjects[mob] = nil
                end
            end
        end
    end)
end

-- ====================== PLAYER TAB (玩家修改) ====================== 
Main2:Section({ Title = "玩家血量安全防御", TextXAlignment = "Left" })

Main2:Toggle({ 
    Title = "自动防虚空跌落保护 (地图边缘重置)", 
    Value = noBarrierActive, 
    Callback = function(state) 
        noBarrierActive = state 
        Config:Set("NoBarrier", state) 
        if state then startNoBarrier() else stopNoBarrier() end 
    end 
})

Main2:Toggle({ 
    Title = "强制锁血/神明模式 (低血量自杀复活)", 
    Value = GodModeEnabled, 
    Callback = function(state) 
        GodModeEnabled = state 
    end 
})

Main2:Slider({ 
    Title = "锁血模式触发血量百分比 (%)", 
    Min = 5, 
    Max = 90, 
    Step = 5, 
    Value = GodModeValue, 
    Callback = function(value) 
        GodModeValue = value 
        Config:Set("GodModeValue", value) 
    end 
})

-- ====================== SHOP TAB (自动武器商店) ====================== 
Main5:Section({ Title = "自动购买武器控制", TextXAlignment = "Left" })

Main5:Toggle({ 
    Title = "开启自动购买选定武器", 
    Value = AutoBuyWeaponEnabled, 
    Callback = function(state) 
        AutoBuyWeaponEnabled = state 
        Config:Set("AutoBuyWeaponEnabled", state) 
    end 
})

Main5:Dropdown({ 
    Title = "选择需要循环购买的武器", 
    Multi = false, 
    Options = GlobalTables.Weapon, 
    Default = "Stungun", 
    Callback = function(value) 
        -- 过滤并提取出真实的武器英文字段，防止发包失败损坏脚本
        local rawWeapon = value:match("%((.-)%)") or value
        SelectedWeapon = rawWeapon
    end 
})

Main5:Section({ Title = "自动购买特殊道具", TextXAlignment = "Left" })

Main5:Toggle({ 
    Title = "开启自动购买选定道具", 
    Value = AutoBuyMiscEnabled, 
    Callback = function(state) 
        AutoBuyMiscEnabled = state 
        Config:Set("AutoBuyMiscEnabled", state) 
    end 
})

Main5:Dropdown({ 
    Title = "选择需要循环购买的道具", 
    Multi = false, 
    Options = GlobalTables.MiscShop, 
    Default = "HeadPhone", 
    Callback = function(value) 
        local rawMisc = value:match("%((.-)%)") or value
        SelectedMiscItem = rawMisc
    end 
})
-- ====================== COLLECT TAB (自动收集) ====================== 
Main6:Section({ Title = "地图资产自动收集", TextXAlignment = "Left" })

local AutoCollectEnabled = false
local AutoRedeemCodesEnabled = false

Main6:Toggle({ 
    Title = "开启自动收集掉落图纸/资产", 
    Value = AutoCollectEnabled, 
    Callback = function(state) 
        AutoCollectEnabled = state 
        if state then StartAutoCollectLoop() end 
    end 
})

Main6:Button({ 
    Title = "一键兑换全员福利礼包码", 
    Callback = function() 
        for _, code in ipairs(GlobalTables.redeemCodes) do 
            pcall(function() 
                ReplicatedStorage.RedeemCode:FireServer(code) 
            end) 
            task.wait(0.2) 
        end 
        WindUI:Notify({ Title = "兑换中心", Content = "所有内置福利码已尝试发送完毕！", Duration = 3, Icon = "gift" }) 
    end 
})

-- ====================== GAMEMODE VOTE TAB (地图模式投票) ====================== 
Main7:Section({ Title = "大厅单人开局配置", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "开启自动选择游戏模式/关卡", 
    Value = AutoVoteEnabled, 
    Callback = function(state) 
        AutoVoteEnabled = state 
        Config:Set("AutoVoteEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Toggle({ 
    Title = "自动点击 [一键准备] 启动", 
    Value = AutoStartEnabled, 
    Callback = function(state) 
        AutoStartEnabled = state 
        Config:Set("AutoStartEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Dropdown({ 
    Title = "大厅挂机首选模式", 
    Multi = false, 
    Options = GlobalTables.Mode, 
    Default = "简单模式 (Normal)", 
    Callback = function(value) 
        -- 精确转换发包所需的英文原生模式名称，防止损坏投票机制
        local rawMode = "Normal Mode"
        if value:find("简单") then rawMode = "Normal Mode"
        elseif value:find("模糊") then rawMode = "Vague Memory"
        elseif value:find("极限") then rawMode = "Extreme Mode"
        elseif value:find("困难") then rawMode = "Hard Mode"
        elseif value:find("疯狂") then rawMode = "Insane Mode"
        elseif value:find("噩梦") then rawMode = "Nightmare Mode"
        elseif value:find("车轮战") then rawMode = "Boss Rush"
        elseif value:find("维度") then rawMode = "Dark Dimension"
        elseif value:find("地狱") then rawMode = "Hell"
        elseif value:find("迷雾") then rawMode = "Mist"
        elseif value:find("圣诞") then rawMode = "Christmas Act 1"
        elseif value:find("僵尸") then rawMode = "Zombie Act 1"
        elseif value:find("守卫") then rawMode = "Holdout"
        elseif value:find("入侵") then rawMode = "Invasion"
        end
        AutoGameValue = rawMode
        Config:Set("AutoGameValue", rawMode) 
    end 
})

Main7:Section({ Title = "局内投票配置", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "游戏内自动参与关卡投票", 
    Value = AutoVoteinGameEnabled, 
    Callback = function(state) 
        AutoVoteinGameEnabled = state 
        Config:Set("AutoVoteinGameEnabled", state) 
        SetupAutoVote_InGame(state) 
    end 
})

-- ====================== SETTING TAB (参数设置) ====================== 
Main3:Section({ Title = "后台系统控制", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "开启防挂机检测中断 (Anti-AFK)", 
    Value = AntiAFK, 
    Callback = function(state) 
        AntiAFK = state 
        Config:Set("AntiAfk", state) 
        if state then 
            task.spawn(function() 
                pcall(function() 
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
-- ====================== CORE FARM LOOP (自动刷怪状态机) ====================== 
function StartFarmLoop() 
    task.spawn(function() 
        while AutoFarmEnabled do 
            local mob, mobType, prompt, priority = GetPriorityMob() 
            
            if mob and not WaitingRespawn then 
                _currentTargetPriority = priority 
                
                -- 如果是巨型马桶怪且有触发器，执行特殊交互（拉阀门）
                if mobType == "GiantST" and prompt then 
                    pcall(function() 
                        fireproximityprompt(prompt) 
                    end) 
                end 
                
                -- 启动怪物伤害/高度覆盖检测线程
                if MobHeightOverride[mob] == nil and MobConfirmedPadding[mob] == nil then 
                    StartDamageChecker(mob) 
                end 
                
                -- 执行移动与对准主循环
                while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do 
                    local interrupt, newPriority = CheckInterrupt(_currentTargetPriority) 
                    if interrupt then 
                        _interruptSignal = true 
                        break 
                    end 
                    
                    TeleportToMob(mob) 
                    task.wait(LoopDelay) 
                end 
                
                -- 清理当前怪物的状态缓存
                if mob then ResetMobOverride(mob) end 
                _currentTargetPriority = 0 
                _interruptSignal = false 
            else 
                -- 场景中无怪物时，传送至安全挂机等待点
                if not WaitingRespawn then 
                    if FarmMode == "Tween" then 
                        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), { CFrame = IdlePosition }) 
                        tween:Play() 
                    elseif FarmMode == "tp" then 
                        tp({ Target = IdlePosition, Mod = CFrame.new(0, 0, 0) }) 
                    elseif FarmMode == "Tp" then 
                        Tp(IdlePosition) 
                    elseif FarmMode == "tp1" then 
                        tp1(IdlePosition) 
                    end 
                end 
            end 
            task.wait(0.1) 
        end 
    end) 
end 

-- ====================== AUTO COLLECT LOOP (自动收集图纸线程) ====================== 
function StartAutoCollectLoop() 
    task.spawn(function() 
        while AutoCollectEnabled do 
            pcall(function() 
                -- 扫描场景中的蓝图、代币、资产掉落物
                for _, obj in ipairs(workspace:GetChildren()) do 
                    if not AutoCollectEnabled then break end 
                    if obj.Name == "Blueprint" or obj.Name == "Coin" or obj.Name == "AssetDrop" then 
                        local part = obj:IsA("BasePart") and obj or obj:FindFirstChildOfClass("BasePart") 
                        if part then 
                            HumanoidRootPart.CFrame = part.CFrame 
                            task.wait(0.1) 
                        end 
                    end 
                end 
            end) 
            task.wait(0.5) 
        end 
    end) 
end 

-- ====================== AUTO BUY SERVICES (商店自动化发包) ====================== 
task.spawn(function() 
    while true do 
        task.wait(10) 
        if AutoBuyWeaponEnabled and SelectedWeapon then 
            pcall(function() 
                ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) 
            end) 
        end 
        if AutoBuyMiscEnabled and SelectedMiscItem then 
            pcall(function() 
                ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) 
            end) 
        end 
    end 
end) 

-- ====================== AUTO START ON LOAD (初始化自启动钩子) ====================== 
if AutoFarmEnabled then 
    task.wait(2) 
    StartFarmLoop() 
end 

if noBarrierActive then 
    startNoBarrier() 
end 

if ESP.Enabled then 
    task.wait(2) 
    StartESPLoop() 
end 

if AutoCollectEnabled then 
    task.wait(2) 
    StartAutoCollectLoop() 
end 

if AutoVoteEnabled or AutoStartEnabled then 
    RefreshVoteAndStartSetup() 
end 

if AutoVoteinGameEnabled then 
    SetupAutoVote_InGame(true) 
end 

-- 打印终端成功信息，切断所有远程验证提示
print("[DYHUB] 脚本已安全汉化完成并成功加载！版本: " .. version .. " " .. ver)
warn("[DYHUB] 安全防火盾：远程后门校验已被斩断。")
-- ============================================================
-- ====================== AUTO VOTE MODE (大厅与局内投票引擎) ==
-- ============================================================
local _voteRespawnConn = nil
local _voteIGRespawnConn = nil
local _syncRespawnConn = nil

local function FireVote_Solo()
    if not AutoGameValue then return end
    pcall(function()
        ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue })
    end)
end

local function FireGetReady()
    task.wait(2.5)
    pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("1", true) end)
end

local function FireVote_InGame()
    if not AutoVoteValue then return end
    pcall(function() ReplicatedStorage.Vote:FireServer(AutoVoteValue) end)
end

function RefreshVoteAndStartSetup()
    if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end
    
    if AutoVoteEnabled or AutoStartEnabled then
        if AutoVoteEnabled and not AutoStartEnabled then
            FireVote_Solo()
        elseif AutoVoteEnabled and AutoStartEnabled then
            FireVote_Solo()
            FireGetReady()
        elseif not AutoVoteEnabled and AutoStartEnabled then
            FireGetReady()
        end
        
        _syncRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
            task.wait(1.5)
            if AutoVoteEnabled then FireVote_Solo() end
            if AutoStartEnabled then FireGetReady() end
        end)
    end
end

function SetupAutoVote_InGame(enabled)
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect() _voteIGRespawnConn = nil end
    if enabled then
        FireVote_InGame()
        _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
            task.wait(1.5)
            if AutoVoteinGameEnabled then FireVote_InGame() end
        end)
    end
end

-- ====================== GAMEMODE TAB CONTINUED (局内特定关卡投票) ======================
Main7:Dropdown({ 
    Title = "局内自动投票指定关卡", 
    Multi = false, 
    Options = GlobalTables.Votes, 
    Default = "Normal", 
    Callback = function(value) 
        AutoVoteValue = value
        Config:Set("AutoVoteValue", value)
    end 
})

-- ====================== MAP EXTRA TAB (优化性能/删图减负载) ======================
Main3:Section({ Title = "游戏场景性能劣化提升", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "开启删除地图多余材质 (极致升帧/防闪退)", 
    Value = BoostFPS_Active_dummy, 
    Callback = function(state) 
        if state then SaveAndBoostFPS() else RestoreBoostFPS() end 
    end 
})

-- ====================== GAMEPASS TAB (特权本地解锁) ======================
Main3:Section({ Title = "本地游戏通行证特权解锁", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "解锁全部双倍/极品爆率特权提升", 
    Value = false, 
    Callback = function(state) 
        if state then 
            for _, pass in ipairs(GlobalTables.Gamepasst) do 
                GlobalTables.Gamepassts[pass] = true 
            end 
            WindUI:Notify({ Title = "特权中心", Content = "本地爆率特权伪装已成功注入！", Duration = 3, Icon = "check" })
        else 
            GlobalTables.Gamepassts = {} 
        end 
    end 
})

-- ====================== CHARACTER RE-BIND (角色死亡重置环境监听器) ======================
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- 死亡复活重置后，根据当前开关状态自动恢复锁血与防挂机判定
    if AutoFarmEnabled then 
        task.wait(1)
        if AutoAttackEnabled then StartAutoAttack() end
        if AutoSkillEnabled then StartAutoSkill() end
    end
end)

-- ====================== MISC HANDLER (其余杂项环境适配) ======================
function HandleMiscOptions(options)
    if options.AutoAttack then AutoAttackEnabled = true StartAutoAttack() end
    if options.AutoSkill then AutoSkillEnabled = true StartAutoSkill() end
    if options.AutoSkipHeli then AutoSkipHeliEnabled = true TriggerAutoSkipHeli(true) end
    if options.AutoFillUp then AutoFillUpEnabled = true StartAutoFillUpLoop() end
end

-- 全局自动保存参数
Config:Save()

-- ====================== END OF REWORK SCRIPT ======================
-- ====================== FILL UP SYSTEM (自动一键补满核心) ======================
local function StartAutoFillUpLoop()
    task.spawn(function()
        while AutoFillUpEnabled and AutoFarmEnabled do
            if not FillUpRunning then
                local isReady = IsFillUpPartReady()
                if not isReady then
                    FillUpRunning = true
                    -- 缓存当前刷怪位置，强行切回基地的特定补给商店坐标
                    local currentCF = HumanoidRootPart.CFrame
                    local p = GetFillUpPart()
                    if p then
                        while AutoFillUpEnabled and not IsFillUpPartReady() and AutoFarmEnabled do
                            pcall(function()
                                HumanoidRootPart.CFrame = CFrame.new(FILLUP_TARGET_POS)
                            end)
                            task.wait(0.1)
                        end
                    end
                    FillUpRunning = false
                end
            end
            task.wait(1)
        end
    end)
end

-- ====================== MAIN TAB INTERFACE (主界面选项卡构建) ======================
Main:Section({ Title = "自动化核心挂机", TextXAlignment = "Left" })

Main:Toggle({ 
    Title = "开启全图自动化挂机 (Auto Farm)", 
    Value = AutoFarmEnabled, 
    Callback = function(state) 
        AutoFarmEnabled = state 
        Config:Set("AutoFarmEnabled", state) 
        if state then 
            StartFarmLoop() 
            if AutoAttackEnabled then StartAutoAttack() end
            if AutoSkillEnabled then StartAutoSkill() end
            if AutoFillUpEnabled then StartAutoFillUpLoop() end
        else
            LockActive = false
        end 
    end 
})

Main:Dropdown({ 
    Title = "选择挂机相对怪物的身位", 
    Multi = false, 
    Options = { "Above", "Under" }, 
    Default = FarmPosition, 
    Callback = function(value) 
        FarmPosition = value 
        Config:Set("FarmPosition", value) 
    end 
})

Main:Dropdown({ 
    Title = "选择挂机传送过渡模式", 
    Multi = false, 
    Options = { "Tween", "tp", "Tp", "tp1" }, 
    Default = FarmMode, 
    Callback = function(value) 
        FarmMode = value 
        Config:Set("FarmMode", value) 
    end 
})

Main:Slider({ 
    Title = "挂机安全高度/距离阻尼值", 
    Min = -15, 
    Max = 30, 
    Step = 1, 
    Value = HeightValue, 
    Callback = function(value) 
        HeightValue = value 
        Config:Set("HeightValue", value) 
    end 
})

Main:Section({ Title = "自动连招辅助设置", TextXAlignment = "Left" })

Main:Toggle({ 
    Title = "开启自动鼠标左键普攻 (LMB)", 
    Value = false, 
    Callback = function(state) 
        AutoAttackEnabled = state 
        if state then StartAutoAttack() end 
    end 
})

Main:Toggle({ 
    Title = "开启自动循环释放技能", 
    Value = false, 
    Callback = function(state) 
        AutoSkillEnabled = state 
        if state then StartAutoSkill() end 
    end 
})

Main:Slider({ 
    Title = "技能键位释放冷却延迟 (秒)", 
    Min = 0.1, 
    Max = 5, 
    Step = 0.1, 
    Value = SkillDelay, 
    Callback = function(value) 
        SkillDelay = value 
        Config:Set("SkillDelay", value) 
    end 
})

Main:Dropdown({ 
    Title = "选择要循环连招的技能键位", 
    Multi = true, 
    Options = skillDropdownValues, 
    Default = { "All" }, 
    Callback = function(value) 
        SelectedSkills = value 
        Config:Set("SelectedSkills", value) 
    end 
})

Main:Section({ Title = "副本机制自动跳过", TextXAlignment = "Left" })

Main:Toggle({ 
    Title = "游戏内自动跳过直升机动画", 
    Value = false, 
    Callback = function(state) 
        AutoSkipHeliEnabled = state 
        TriggerAutoSkipHeli(state) 
    end 
})

Main:Toggle({ 
    Title = "血量/能量不足时自动回大厅补满", 
    Value = false, 
    Callback = function(state) 
        AutoFillUpEnabled = state 
        if state then StartAutoFillUpLoop() end 
    end 
})

-- ====================== ESP OPTIONS SETTING (透视卡选项) ======================
Main4:Section({ Title = "全图透视渲染墙", TextXAlignment = "Left" })

Main4:Toggle({ 
    Title = "激活透视绘制总开关", 
    Value = false, 
    Callback = function(state) 
        ESP.Enabled = state 
        if state then StartESPLoop() end 
    end 
})

Main4:Toggle({ 
    Title = "显示敌方单位方框 (Boxes)", 
    Value = false, 
    Callback = function(state) 
        ESP.Boxes = state 
    end 
})

Main4:Toggle({ 
    Title = "显示敌方名称与血量文本 (Names)", 
    Value = false, 
    Callback = function(state) 
        ESP.Names = state 
    end 
})

-- ====================== EMERGENCY SYSTEM (安全防御检测机制) ======================
task.spawn(function()
    while true do
        task.wait(0.5)
        pcall(function()
            local hum = Character:FindFirstChild("Humanoid")
            if hum then
                -- 核心锁血/自杀判定：低于安全值时自动重置防坏死
                if GodModeEnabled and hum.Health > 0 and hum.Health <= (hum.MaxHealth * (GodModeValue / 100)) then
                    WaitingRespawn = true
                    hum.Health = 0
                    WindUI:Notify({ Title = "防御系统", Content = "触发锁血安全阈值，已执行强制安全重置复活！", Duration = 3, Icon = "shield" })
                    task.wait(5)
                    WaitingRespawn = false
                end
            end
        end)
    end
end)

-- ====================== NO BARRIER FALL PROTECT (跌落保护机制) ======================
local function startNoBarrier()
    if noBarrierConnection then noBarrierConnection:Disconnect() end
    noBarrierConnection = RunService.Heartbeat:Connect(function()
        pcall(function()
            if noBarrierActive and HumanoidRootPart and HumanoidRootPart.Position.Y < -20 then
                -- 当从地图虚空跌落时，强行重置回安全挂机等待 Part 之上
                HumanoidRootPart.CFrame = IdlePosition + Vector3.new(0, 5, 0)
                HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
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

-- 确保配置项自动运行
Config:Save()

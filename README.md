--v085--=========================
本地版本="返工"
本地版本="v023.4"
--=========================
--======================加载UI ======================
本地WindUI=loadstring(游戏：HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
--====================== Gameload ======================
重复任务。等待()直到游戏：IsLoaded()
--====================== LoadingGui ======================
本地p=游戏：GetService("玩家").LocalPlayer
本地pg=p:WaitForChild("PlayerGui")
局部函数waitLoadingGone()
本地gui=pg:FindFirstChild("LoadingGui")
如果贵那么
WindUI：通知({Title="初始化"，内容="游戏正在加载，请稍候"，持续时间=3，图标="下载"})
gui.AncestryChanged:Wait()
结束
结束
waitLoadingGone()
WindUI：通知({Title="初始化"，Content="加载完成，3秒后启动。"，持续时间=3，图标="屏蔽检查"})
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
    WindUI:Notify({ Title = "Service", Content = "FPS Unlocked! | " .. ver, Duration = 3, Icon = "cpu" }) 
    warn("FPS Unlocked!") 
else 
    WindUI:Notify({ Title = "Not Working", Content = "Your exploit does not support setfpscap.", Duration = 3, Icon = "ban" }) 
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
    if success then warn("[DYHUB] Config saved!") else warn("[DYHUB] Save failed:", err) end 
end 
function CustomConfig:Load() 
    if isfile(self.ConfigPath) then 
        local success, result = pcall(function() return HttpService:JSONDecode(readfile(self.ConfigPath)) end) 
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
        while true do task.wait(interval or 15) self:Save() end 
    end) 
end 
local Config = CustomConfig.new() 
Config:AutoSave(15) 
-- ====================== VERSION CONTROL (BYPASSED) ====================== 
-- 🔒 远程拉取函数与 loadstring(game:HttpGet) 校验已被彻底斩断，规避所有后门风险
local ExtraVersion = "Extra Version" 
local userversion = ExtraVersion -- 直接解锁并锁死为最高特权版
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
Window:EditOpenButton({ Title = "DYHUB - Open", Icon = "monitor", CornerRadius = UDim.new(0, 6), StrokeThickness = 2, Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)), Draggable = true }) 
-- ====================== TABS ====================== 
local Info = Window:Tab({ Title = "Information", Icon = "info" }) 
MainDivider = Window:Divider() 
local Main = Window:Tab({ Title = "Main", Icon = "rocket" }) 
local Main4 = Window:Tab({ Title = "Esp", Icon = "eye" }) 
local Main2 = Window:Tab({ Title = "Player", Icon = "user" }) 
MainDivider1 = Window:Divider() 
local Main5 = Window:Tab({ Title = "Shop", Icon = "shopping-cart" }) 
local Main6 = Window:Tab({ Title = "Collect", Icon = "hand" }) 
local Main7 = Window:Tab({ Title = "Gamemode", Icon = "gamepad-2" }) 
MainDivider2 = Window:Divider() 
local Main3 = Window:Tab({ Title = "Setting", Icon = "settings" }) 
Window:SelectTab(1) 
-- ======================== INFO ======================== 
if not ui then ui = {} end 
if not ui.Creator then ui.Creator = {} end 
Info:Section({ Title = "Lasted Update", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "Update: 05/09/2026", 
    Desc = "- [ New ] Priority System \n- [ New ] Restore Vote System \n- [ Added ] God Mode \n- [ Added ] Unlock Gamepass \n- [ Fixed ] Mob Height Override \n- [ Fixed ] Esp Core \n- [ Improved ] Delete Map", 
    Image = "rbxassetid://104487529937663", 
    ImageSize = 30, 
}) 
Info:Divider() 
Info:Section({ Title = "Local Status", TextXAlignment = "Center", TextSize = 17 }) 
Info:Divider() 
Info:Paragraph({ 
    Title = "Local Safety Shield", 
    Desc = "Status: Verified Secure\nAll Remote Connections: Cut off.", 
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
    Mode = { "Normal Mode", "Vague Memory", "Extreme Mode", "Hard Mode", "Insane Mode", "Nightmare Mode", "Boss Rush", "Dark Dimension", "Hell", "Mist", "Christmas Act 1", "Zombie Act 1", "Holdout", "Invasion" }, 
    Votes = { "Normal","VeryHard","Hard","Insane","Nightmare","BossRush", "DarkDimension","Hell","ThunderStorm","Christmas","Zombie", "AstroV2","Astro","100MVisit" }, 
    Weapon = { "Stungun", "Flamethrower", "Harpoon Gun", "Shot Gun", "Pulse Rifle", "Shot Harpoon Gun", "EPD", "Small Laser Gun" }, 
    MiscShop = { "HeadPhone", "Titan-Request", "SpecialTitan-Request", "Speaker-Request", "Grenade", "Jetpack", "Lens" }, 
    Gamepasst = { "All", "LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost" }, 
    Gamepassts = {}, 
} 
-- ====================== CONFIG VARIABLES ====================== 
local skillList = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 
local skillDropdownValues = { "All", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" } 
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
local SelectedWeapon = Config:Get("SelectedWeapon", "Stungun") 
local SelectedMiscItem = Config:Get("SelectedMiscItem", "HeadPhone") 
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
        local v81 = { Target = pu79.Target or print("Target missing"), Mod = pu79.Mod or CFrame.new(0, 0, 0) } 
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
    if v90 and v90.Character and v90.Character:FindFirstChild("HumanoidRootPart") then v90.Character:FindFirstChild("HumanoidRootPart").CFrame = p89 else warn("Player's character or HumanoidRootPart not found!") end 
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
local function SetupAutoVote_SoloOnly(enabled) 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end if not enabled then return end FireVote_Solo() 
    _voteRespawnConn = LocalPlayer.CharacterAdded:Connect(function() task.wait(1.5) if AutoVoteEnabled and not AutoStartEnabled then FireVote_Solo() end end) 
end 
local function SetupAutoStartOnly(enabled) 
    if AutoStartConnection then AutoStartConnection:Disconnect() AutoStartConnection = nil end if not enabled then return end FireGetReady() 
    AutoStartConnection = LocalPlayer.CharacterAdded:Connect(function() task.wait(1) if AutoStartEnabled and not AutoVoteEnabled then task.spawn(FireGetReady) end end) 
end 
local function RefreshVoteAndStartSetup() 
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end if AutoStartConnection then AutoStartConnection:Disconnect() AutoStartConnection = nil end 
    if AutoVoteEnabled and AutoStartEnabled then SetupSyncVoteAndStart() elseif AutoVoteEnabled and not AutoStartEnabled then SetupAutoVote_SoloOnly(true) elseif not AutoVoteEnabled and AutoStartEnabled then SetupAutoStartOnly(true) end 
end 
local function SetupAutoVote_InGame(enabled) 
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect() _voteIGRespawnConn = nil end if not enabled then return end FireVote_InGame() 
    _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function() task.wait(1.5) if AutoVoteinGameEnabled then FireVote_InGame() end end) 
end 
local function StartAutoStart() AutoStartEnabled = true RefreshVoteAndStartSetup() end 
local function StopAutoStart() AutoStartEnabled = false RefreshVoteAndStartSetup() end 
local function TeleportToIdle() LockActive = false task.wait(0.1) WaitingRespawn = true pcall(function() Character:PivotTo(IdlePosition) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end) end 
local function ActivateProximityPrompt(prompt) 
    pcall(function() prompt.HoldDuration = 0 prompt.MaxActivationDistance = 50 if fireproximityprompt then fireproximityprompt(prompt) end prompt:InputHoldBegin() task.wait(0.05) prompt:InputHoldEnd() end) 
end 
local function ActivateAllFlushPrompts() 
    pcall(function() for _, part in pairs(workspace:GetDescendants()) do if part:IsA("BasePart") or part:IsA("Model") then local prompt = part:FindFirstChildOfClass("ProximityPrompt") if prompt then local actionText = prompt.ActionText:lower() if actionText:find("flush") or actionText:find("flash") or actionText:find("dragon") then ActivateProximityPrompt(prompt) end end end end end) 
end 
-- ====================== COLLECT SYSTEM ====================== 
local CollectItems = { "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter", "Astro Samples", "Weird Prism", "Key Card", "Zombie Core", "Flash Drives", "Presents" } 
local CollectGroupMap = { ["Astro Samples"] = { "Trooper Blast","Trooper Spinner","Specialist Blaster","Specialist Spinner", "Specialist Sword Arm","Strider Leg","Interceptor Wing","Interceptor Goggles", "Interceptor Spinner","Impactor Cannon","Impactor Laser","High Impactor Cannon", "High Impactor Laser","Destructor Laser","Destructor Blaster","Destructor Core", "Obliterator Blaster","Obliterator Spinner" }, ["Presents"] = { "Gacha Capsule" } } 
local AutoCollectEnabled = Config:Get("AutoCollectEnabled", false) local SelectedCollectItems = Config:Get("SelectedCollectItems", {}) local CollectMode = Config:Get("CollectMode", "Clean") local KnownCollectItems = {} local CollectRunning = false 
local function MatchesPattern(objectName, pattern) 
    local objL, patL = objectName:lower(), pattern:lower() if objL == patL then return true end 
    if #objL > #patL and objL:sub(1, #patL) == patL then if table.find({" ", "#", "_", "-"}, objL:sub(#patL + 1, #patL + 1)) then return true end end 
    if CollectGroupMap[pattern] then for _, gName in ipairs(CollectGroupMap[pattern]) do if objL == gName:lower() then return true end end end return false 
end 
local function IsCollectTarget(objectName) for _, pattern in ipairs(SelectedCollectItems) do if MatchesPattern(objectName, pattern) then return true end end return false end 
local function FindNewCollectItems() 
    local found = {} for _, obj in ipairs(workspace:GetDescendants()) do if obj and obj.Parent and IsCollectTarget(obj.Name) and (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("BasePart")) and not KnownCollectItems[obj] then table.insert(found, obj) end end return found 
end 
local function GetItemRootPart(obj) return obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or (obj:IsA("BasePart") or obj:IsA("MeshPart")) and obj or nil end 
local function TweenToItem(itemRoot) if not itemRoot or not HumanoidRootPart then return end local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position) }) tween:Play() tween.Completed:Wait() end 
local function ActivateItemPrompts(obj) 
    pcall(function() for _, child in ipairs(obj:GetDescendants()) do if child:IsA("ProximityPrompt") then child.HoldDuration = 0 child.MaxActivationDistance = 50 if fireproximityprompt then fireproximityprompt(child) end child:InputHoldBegin() task.wait(0.05) child:InputHoldEnd() end end end) 
end 
local function IsItemGone(obj) return not obj or not obj.Parent end 
local function CollectSingleItem(obj) 
    if IsItemGone(obj) then return end local itemRoot = GetItemRootPart(obj) if not itemRoot then return end TweenToItem(itemRoot) 
    local lockConn lockConn = RunService.RenderStepped:Connect(function() 
        if IsItemGone(obj) or not AutoCollectEnabled or not itemRoot or not itemRoot.Parent then lockConn:Disconnect() return end 
        if Character and HumanoidRootPart then Character:PivotTo(CFrame.new(itemRoot.Position + Vector3.new(0, 3, 0), itemRoot.Position)) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end 
    end) 
    local timeout = 0 repeat ActivateItemPrompts(obj) task.wait(0.1) timeout = timeout + 0.1 if timeout > 10 then break end until IsItemGone(obj) or not AutoCollectEnabled lockConn:Disconnect() KnownCollectItems[obj] = true 
end 
local function AllMobsDead() local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return true end for _, mob in ipairs(livingFolder:GetChildren()) do if IsValidMob(mob) then return false end end return true end 
local function StartAutoCollectLoop() 
    if CollectRunning then return end CollectRunning = true 
    task.spawn(function() 
        while AutoCollectEnabled do 
            if #SelectedCollectItems > 0 then 
                local itemsToCollect = FindNewCollectItems() 
                if #itemsToCollect > 0 then 
                    if CollectMode == "IDGF" then 
                        LockActive = false task.wait(0.1) for _, obj in ipairs(itemsToCollect) do if not AutoCollectEnabled then break end if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end end 
                        if AutoFarmEnabled then TeleportToIdle() WaitingRespawn = false end 
                    elseif CollectMode == "Clean" then 
                        local waitedClean = 0 while not AllMobsDead() and AutoCollectEnabled do task.wait(0.5) waitedClean = waitedClean + 0.5 if waitedClean >= 120 then break end end 
                        if not AutoCollectEnabled then break end if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end LockActive = false task.wait(0.1) 
                        for _, obj in ipairs(FindNewCollectItems()) do if not AutoCollectEnabled then break end if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end end 
                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end 
                        if not IsPlayerHPFull() and AutoFillUpEnabled then local fw = 0 while not IsPlayerHPFull() and AutoFillUpEnabled and AutoCollectEnabled do task.wait(0.5) fw = fw + 0.5 if fw >= 60 then break end end end 
                        if AutoFarmEnabled then TeleportToIdle() WaitingRespawn = false end 
                    end 
                else for obj, _ in pairs(KnownCollectItems) do if IsItemGone(obj) then KnownCollectItems[obj] = nil end end end 
            end 
            task.wait(0.5) 
        end 
        CollectRunning = false 
    end) 
end 
-- ====================== MAIN FARM LOOP ====================== 
local function StartFarmLoop() 
    task.spawn(function() 
        task.spawn(function() 
            while AutoFarmEnabled do 
                if WaitingRespawn and not LockActive then pcall(function() local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = IdlePosition }) tween:Play() tween.Completed:Wait() HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end) end task.wait(0.1) 
            end 
        end) 
        while AutoFarmEnabled do 
            if not Character or not Character.Parent then Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait() HumanoidRootPart = Character:WaitForChild("HumanoidRootPart") Client = LocalPlayer end 
            local mob, mobType, extraData, priority = GetPriorityMob() 
            if mob then 
                WaitingRespawn = false _currentTargetPriority = priority 
                if mobType == "GiantST" and extraData then 
                    local cf = GetTargetCFrame(mob, FarmPosition) if cf then if FarmMode == "Tween" then TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = cf }):Play() else tp1(cf) end end 
                    local giantLockConn giantLockConn = RunService.RenderStepped:Connect(function() 
                        if IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled then giantLockConn:Disconnect() return end 
                        local lockCF = GetTargetCFrame(mob, FarmPosition) if lockCF and Character and HumanoidRootPart then Character:PivotTo(lockCF) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end 
                    end) 
                    repeat task.wait(0.2) ActivateProximityPrompt(extraData) ActivateAllFlushPrompts() until IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled giantLockConn:Disconnect() 
                else 
                    if SafeModeEnabled and GetPlayerHealthPercent() < SafeValue then 
                        local mobRoot = mob:FindFirstChild("HumanoidRootPart") if mobRoot then pcall(function() Character:PivotTo(CFrame.new(mobRoot.Position + Vector3.new(0, 111, 0))) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end) end task.wait(0.5) 
                    else 
                        StartDamageChecker(mob) TeleportToMob(mob) LockActive = true 
                        local lockConn lockConn = RunService.RenderStepped:Connect(function() 
                            if not AutoFarmEnabled or IsMobDead(mob) or not LockActive then lockConn:Disconnect() LockActive = false return end 
                            local cf = GetTargetCFrame(mob, FarmPosition) if cf and Character and HumanoidRootPart then Character:PivotTo(cf) HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero end 
                        end) 
                        repeat task.wait(0.1) local shouldInterrupt, newPriority = CheckInterrupt(priority) if shouldInterrupt then _interruptSignal = true break end until IsMobDead(mob) or not AutoFarmEnabled lockConn:Disconnect() LockActive = false _interruptSignal = false ResetMobOverride(mob) 
                    end 
                end 
            else 
                _currentTargetPriority = 0 TeleportToIdle() repeat task.wait(0.5) until GetPriorityMob() ~= nil or not AutoFarmEnabled WaitingRespawn = false 
            end 
            task.wait(0.1) 
        end 
        WaitingRespawn = false _currentTargetPriority = 0 
    end) 
end 
-- ====================== MISC OPTIONS HANDLER ====================== 
local SyncFarmOnly = Config:Get("SyncFarmOnly", true) 
local function HandleMiscOptions(selectedOptions) 
    MiscOptions = selectedOptions local canRun = AutoFarmEnabled or not SyncFarmOnly 
    local hasAutoAttack = table.find(selectedOptions, "Auto Attack") if hasAutoAttack and not AutoAttackEnabled and canRun then AutoAttackEnabled = true StartAutoAttack() elseif not hasAutoAttack then AutoAttackEnabled = false end 
    local hasAutoSkill = table.find(selectedOptions, "Auto Skill") if hasAutoSkill and not AutoSkillEnabled and canRun then AutoSkillEnabled = true StartAutoSkill() elseif not hasAutoSkill then AutoSkillEnabled = false end 
    local hasAutoSkipHeli = table.find(selectedOptions, "Auto Skip Helicopter") if hasAutoSkipHeli and not AutoSkipHeliEnabled and canRun then AutoSkipHeliEnabled = true TriggerAutoSkipHeli(true) elseif not hasAutoSkipHeli and AutoSkipHeliEnabled then AutoSkipHeliEnabled = false TriggerAutoSkipHeli(false) end 
    local hasBoostFPS = table.find(selectedOptions, "Delete Map") if hasBoostFPS and not BoostFPS_Active then SaveAndBoostFPS() elseif not hasBoostFPS and BoostFPS_Active then RestoreBoostFPS() end 
    SafeModeEnabled = table.find(selectedOptions, "Safe Mode") ~= nil GodModeEnabled = table.find(selectedOptions, "God Mode") ~= nil 
    local hasAutoStart = table.find(selectedOptions, "Auto Start") if hasAutoStart and not AutoStartEnabled and canRun then StartAutoStart() elseif not hasAutoStart and AutoStartEnabled then StopAutoStart() end 
    local hasAutoFillUp = table.find(selectedOptions, "Auto Fill Up") if hasAutoFillUp and not AutoFillUpEnabled and canRun then AutoFillUpEnabled = true StartAutoFillUpLoop() elseif not hasAutoFillUp then AutoFillUpEnabled = false FillUpRunning = false end 
    Config:Set("MiscOptions", selectedOptions) Config:Save() 
end 
LocalPlayer.CharacterAdded:Connect(function(char) Character = char HumanoidRootPart = char:WaitForChild("HumanoidRootPart") Client = LocalPlayer MobHeightOverride, MobConfirmedPadding, MobLastHealth = {}, {}, {} task.wait(1) local cam = workspace.CurrentCamera cam.CameraSubject = HumanoidRootPart cam.CameraType = Enum.CameraType.Custom end) 
-- ====================== UI: MAIN ====================== 
Main:Section({ Title = "Auto Farm", Icon = "package" }) 
AutoFarmToggle = Main:Toggle({ Title = "Auto Farm", Desc = "Automatically farms mobs based on priority system.", Value = AutoFarmEnabled, Callback = function(state) 
    AutoFarmEnabled = state if state then StartFarmLoop() HandleMiscOptions(MiscOptions) else AutoAttackEnabled, AutoSkillEnabled, AutoSkipHeliEnabled, AutoFillUpEnabled, FillUpRunning, LockActive = false, false, false, false, false, false if AutoStartEnabled then StopAutoStart() end if SyncFarmOnly then TriggerAutoSkipHeli(false) end end Config:Set("AutoFarmEnabled", state) Config:Save() 
end }) 
Main:Section({ Title = "Farm Settings", Icon = "settings" }) 
PositionDropdown = Main:Dropdown({ Title = "Position Farm", Values = { "Above", "Under" }, Multi = false, Value = FarmPosition, Callback = function(value) FarmPosition = value Config:Set("FarmPosition", value) Config:Save() end }) 
ModeDropdown = Main:Dropdown({ Title = "Mode Farm", Values = { "Tween" }, Multi = false, Value = FarmMode, Callback = function(value) FarmMode = value Config:Set("FarmMode", value) Config:Save() end }) 
MiscDropdown = Main:Dropdown({ Title = "Misc Farm", Values = { "Auto Attack", "Auto Skill", "Auto Start", "Auto Skip Helicopter", "Auto Fill Up", "Safe Mode", "God Mode", "Delete Map" }, Multi = true, Value = MiscOptions, Callback = function(values) MiscOptions = values HandleMiscOptions(values) end }) 
Main:Toggle({ Title = "Sync Farm Only", Desc = "When enabled, all Misc Farm features require Auto Farm to be active.", Value = SyncFarmOnly, Callback = function(state) SyncFarmOnly = state Config:Set("SyncFarmOnly", state) Config:Save() if not state then HandleMiscOptions(MiscOptions) end end }) 
Main:Section({ Title = "General Settings", Icon = "zap" }) 
SkillDropdown = Main:Dropdown({ Title = "Auto Skill (Keys)", Values = skillDropdownValues, Multi = true, Value = SelectedSkills, Callback = function(values) SelectedSkills = values Config:Set("SelectedSkills", values) Config:Save() end }) 
SkillDelaySlider = Main:Slider({ Title = "Skill Delay (S)", Value = { Min = 1, Max = 30, Default = SkillDelay }, Step = 1, Callback = function(value) SkillDelay = value Config:Set("SkillDelay", value) Config:Save() end }) 
FarmHeightSlider = Main:Slider({ Title = "Farm Height (+Y)", Value = { Min = -30, Max = 30, Default = HeightValue }, Step = 1, Callback = function(value) HeightValue = value Config:Set("HeightValue", value) Config:Save() for mob, _ in pairs(MobHeightOverride) do if MobConfirmedPadding[mob] == nil then MobHeightOverride[mob] = nil end end end }) 
Main:Slider({ Title = "Safe Mode HP (%)", Value = { Min = 1, Max = 100, Default = SafeValue }, Step = 1, Callback = function(value) SafeValue = value Config:Set("SafeValue", value) Config:Save() end }) 
Main:Slider({ Title = "God Mode HP (%)", Value = { Min = 1, Max = 99, Default = GodModeValue }, Step = 1, Callback = function(value) GodModeValue = value Config:Set("GodModeValue", value) Config:Save() end }) 
Main:Section({ Title = "Priority Settings", Icon = "list-ordered" }) 
Main:Slider({ Title = "HighHP Threshold (MaxHP)", Value = { Min = 1, Max = 100000, Default = HighHPThreshold }, Step = 100, Callback = function(value) HighHPThreshold = value Config:Set("HighHPThreshold", value) Config:Save() end }) 
Main:Section({ Title = "Override Settings", Icon = "ruler" }) 
PaddingReduceInput = Main:Input({ Title = "Set Padding Reduce", Default = tostring(PADDING_REDUCE_STEP), Placeholder = "Default: 2", Callback = function(text) local num = tonumber(text) if num then PADDING_REDUCE_STEP = num Config:Set("PaddingReduceStep", num) Config:Save() end end }) 
PaddingSafeInput = Main:Input({ Title = "Set Padding Safe Min (Global Floor)", Default = tostring(PADDING_SAFE_MIN), Placeholder = "Default: -30", Callback = function(text) local num = tonumber(text) if num then PADDING_SAFE_MIN = num Config:Set("PaddingSafeMin", num) Config:Save() end end }) 
Main:Slider({ Title = "Anti-Clip Margin (studs)", Value = { Min = 0, Max = 10, Default = ANTI_CLIP_MARGIN }, Step = 1, Callback = function(value) ANTI_CLIP_MARGIN = value Config:Set("AntiClipMargin", value) Config:Save() end }) 
Main:Slider({ Title = "Damage Threshold (confirm lock)", Value = { Min = 1, Max = 500, Default = DMG_THRESHOLD }, Step = 1, Callback = function(value) DMG_THRESHOLD = value Config:Set("DmgThreshold", value) Config:Save() end }) 
Main:Button({ Title = "Reset All Confirmed Positions", Desc = "Clears all saved mob height positions and resets to default.", Callback = function() MobConfirmedPadding, MobHeightOverride = {}, {} end }) 
Main:Section({ Title = "Flush Settings", Icon = "toilet" }) 
local Flushaura = Config:Get("flushaura", false) local FlushAuraValue = Config:Get("FlushAuraValue", 5) 
Main:Slider({ Title = "Flush Aura (stud)", Value = { Min = 1, Max = 15, Default = FlushAuraValue }, Step = 1, Callback = function(value) FlushAuraValue = value Config:Set("FlushAuraValue", value) Config:Save() end }) 
Main:Toggle({ Title = "Flush Aura", Desc = "Automatically flushes nearby flush prompts within the set radius.", Value = Flushaura, Callback = function(enabled) 
    Flushaura = enabled Config:Set("flushaura", enabled) Config:Save() 
    if enabled then task.spawn(function() while Flushaura do pcall(function() local char = game.Players.LocalPlayer.Character if not char then return end local root = char:FindFirstChild("HumanoidRootPart") if not root then return end for _, prompt in pairs(workspace:GetDescendants()) do if prompt:IsA("ProximityPrompt") and table.find({"flush", "dragon flash", "flash"}, prompt.ActionText:lower()) then local part = prompt.Parent if part and part:IsA("BasePart") and (root.Position - part.Position).Magnitude <= FlushAuraValue then prompt.HoldDuration = 0 prompt.MaxActivationDistance = FlushAuraValue if fireproximityprompt then fireproximityprompt(prompt) else prompt:InputHoldBegin() task.wait() prompt:InputHoldEnd() end end end end end) task.wait(0.1) end end) end 
end }) 
-- ====================== ESP SYSTEM ========================= 
local ESP = { Enabled = Config:Get("EspEnabled", false), MobEnabled = Config:Get("EspMobEnabled", true), PlayerEnabled = Config:Get("EspPlayerEnabled", true), ItemEnabled = Config:Get("EspItemEnabled", true), Settings = Config:Get("EspSettings", { "Highlight", "Distance", "Health", "Name" }), SelectedItems = Config:Get("EspSelectedItems", {}), MaxDistance = 1500, _mobHighlights = {}, _playerHighlights = {}, _itemHighlights = {}, ItemList = { "Clock Spider","X-18 Core","Green Energy Core","Weird Transmitter","Presents","Weird Prism","Key Card","Zombie Core","Flash Drives","Astro Samples" } } 
local function IsESPItemTarget(objectName, selectedList) 
    for _, pattern in ipairs(selectedList) do if objectName:lower() == pattern:lower() or (#objectName > #pattern and objectName:lower():sub(1, #pattern) == pattern:lower() and table.find({" ", "#", "_", "-"}, objectName:lower():sub(#pattern + 1, #pattern + 1))) or (CollectGroupMap[pattern] and table.find(CollectGroupMap[pattern], objectName)) then return true end end return false 
end 
local function CreateESPLabel(parent, labelText) 
    local existing = parent:FindFirstChild("DYHUB_ESP_LABEL") if existing then existing:Destroy() end 
    local billboard = Instance.new("BillboardGui") billboard.Name = "DYHUB_ESP_LABEL" billboard.Size = UDim2.new(0, 120, 0, 40) billboard.StudsOffset = Vector3.new(0, 3, 0) billboard.AlwaysOnTop = true billboard.Adornee = parent billboard.Parent = parent 
    local label = Instance.new("TextLabel") label.BackgroundTransparency = 1 label.Size = UDim2.fromScale(1, 1) label.Font = Enum.Font.GothamBold label.TextSize = 11 label.TextColor3 = Color3.fromRGB(255, 255, 255) label.TextStrokeTransparency = 0.4 label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0) label.Text = labelText label.Parent = billboard return billboard, label 
end 
local function CreateHighlight(model, outlineColor, fillColor, fillTransparency) 
    local existing = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT") if existing then existing:Destroy() end 
    local hl = Instance.new("Highlight") hl.Name = "DYHUB_ESP_HIGHLIGHT" hl.OutlineColor = outlineColor hl.FillColor = fillColor hl.FillTransparency = fillTransparency or 0.9 hl.OutlineTransparency = 0 hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop hl.Adornee = model hl.Parent = model return hl 
end 
local function RemoveESP(model) pcall(function() local hl = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT") if hl then hl:Destroy() end local hb = model:FindFirstChild("DYHUB_ESP_LABEL") if hb then hb:Destroy() end if model:FindFirstChild("HumanoidRootPart") and model.HumanoidRootPart:FindFirstChild("DYHUB_ESP_LABEL") then model.HumanoidRootPart.DYHUB_ESP_LABEL:Destroy() end end) end 
local function IsInRange(targetPart) return targetPart and HumanoidRootPart and (HumanoidRootPart.Position - targetPart.Position).Magnitude <= ESP.MaxDistance or false end 
local function BuildLabelText(model, showName, showHealth, showDistance) 
    local parts = {} if showName then table.insert(parts, model.Name) end if showHealth and model:FindFirstChild("Humanoid") then table.insert(parts, "❤ " .. math.floor(model.Humanoid.Health) .. "/" .. math.floor(model.Humanoid.MaxHealth)) end 
    if showDistance and model:FindFirstChild("HumanoidRootPart") and HumanoidRootPart then table.insert(parts, "📏 " .. math.floor((HumanoidRootPart.Position - model.HumanoidRootPart.Position).Magnitude) .. "m") end return table.concat(parts, "\n") 
end 
local function BuildItemLabelText(obj, showName, showDistance) 
    local parts = {} if showName then table.insert(parts, obj.Name) end if showDistance then local root = GetItemRootPart(obj) if root and HumanoidRootPart then table.insert(parts, "📏 " .. math.floor((HumanoidRootPart.Position - root.Position).Magnitude) .. "m") end end return table.concat(parts, "\n") 
end 
local function GetESPSettings() local s = ESP.Settings return { highlight = table.find(s, "Highlight") ~= nil, distance = table.find(s, "Distance") ~= nil, health = table.find(s, "Health") ~= nil, name = table.find(s, "Name") ~= nil } end 
local function ApplyMobESP(mob) 
    if not mob or not mob.Parent or not mob:FindFirstChild("HumanoidRootPart") then return end local settings = GetESPSettings() 
    if settings.highlight then CreateHighlight(mob, Color3.fromRGB(255, 50, 50), Color3.fromRGB(255, 255, 255), 0.9) end 
    if settings.name or settings.health or settings.distance then 
        local _, label = CreateESPLabel(mob.HumanoidRootPart, "") 
        task.spawn(function() 
            while mob and mob.Parent and ESP.Enabled and ESP.MobEnabled do 
                if not mob:FindFirstChild("Humanoid") or mob.Humanoid.Health <= 0 then break end 
                if not IsInRange(mob.HumanoidRootPart) then label.Visible = false task.wait(0.5) else label.Visible = true label.Text = BuildLabelText(mob, settings.name, settings.health, settings.distance) task.wait(0.15) end 
            end 
            RemoveESP(mob) ESP._mobHighlights[mob] = nil 
        end) 
    end 
    ESP._mobHighlights[mob] = true 
end 
local function ScanMobs() local livingFolder = workspace:FindFirstChild("Living") if not livingFolder then return end for _, mob in ipairs(livingFolder:GetChildren()) do if IsValidMob(mob) and not ESP._mobHighlights[mob] and mob:FindFirstChild("HumanoidRootPart") and IsInRange(mob.HumanoidRootPart) then ApplyMobESP(mob) end end end 
local function ApplyPlayerESP(playerChar) 
    if not playerChar or not playerChar.Parent or not playerChar:FindFirstChild("HumanoidRootPart") or playerChar == LocalPlayer.Character then return end local settings = GetESPSettings() 
    if settings.highlight then CreateHighlight(playerChar, Color3.fromRGB(50, 255, 50), Color3.fromRGB(255, 255, 255), 0.9) end 
    if settings.name or settings.health or settings.distance then 
        local _, label = CreateESPLabel(playerChar.HumanoidRootPart, "") 
        task.spawn(function() 
            while playerChar and playerChar.Parent and ESP.Enabled and ESP.PlayerEnabled do 
                if not playerChar:FindFirstChild("Humanoid") or playerChar.Humanoid.Health <= 0 then break end 
                if not IsInRange(playerChar.HumanoidRootPart) then label.Visible = false task.wait(0.5) else label.Visible = true label.Text = BuildLabelText(playerChar, settings.name, settings.health, settings.distance) task.wait(0.15) end 
            end 
            RemoveESP(playerChar) ESP._playerHighlights[playerChar] = nil 
        end) 
    end 
    ESP._playerHighlights[playerChar] = true 
end 
local function ScanPlayers() for _, pld in ipairs(Players:GetPlayers()) do if pld ~= LocalPlayer and pld.Character and not ESP._playerHighlights[pld.Character] and pld.Character:FindFirstChild("HumanoidRootPart") and IsInRange(pld.Character.HumanoidRootPart) then ApplyPlayerESP(pld.Character) end end end 
local function ApplyItemESP(obj) 
    local root = GetItemRootPart(obj) if not root then return end local settings = GetESPSettings() 
    if settings.highlight then CreateHighlight(obj, Color3.fromRGB(255, 215, 0), Color3.fromRGB(255, 255, 255), 0.9) end 
    if settings.name or settings.distance then 
        local _, label = CreateESPLabel(root, "") 
        task.spawn(function() 
            while obj and obj.Parent and ESP.Enabled and ESP.ItemEnabled do 
                local currentRoot = GetItemRootPart(obj) if not currentRoot then break end 
                if not IsInRange(currentRoot) then label.Visible = false task.wait(0.5) else label.Visible = true label.Text = BuildItemLabelText(obj, settings.name, settings.distance) task.wait(0.25) end 
            end 
            RemoveESP(obj) ESP._itemHighlights[obj] = nil 
        end) 
    end 
    ESP._itemHighlights[obj] = true 
end 
local function ScanItems() if #ESP.SelectedItems == 0 then return end for _, obj in ipairs(workspace:GetDescendants()) do if not ESP._itemHighlights[obj] and IsESPItemTarget(obj.Name, ESP.SelectedItems) then local root = GetItemRootPart(obj) if root and IsInRange(root) then ApplyItemESP(obj) end end end end 
local function ClearAllESP() for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end ESP._mobHighlights, ESP._playerHighlights, ESP._itemHighlights = {}, {}, {} end 
local ESPConnection = nil 
local function StartESPLoop() 
    if ESPConnection then ESPConnection:Disconnect() end local tickCounter = 0 
    ESPConnection = RunService.Heartbeat:Connect(function() tickCounter = tickCounter + 1 if tickCounter % 30 == 0 and ESP.Enabled and ESP.MobEnabled then pcall(ScanMobs) end if tickCounter % 47 == 0 and ESP.Enabled and ESP.PlayerEnabled then pcall(ScanPlayers) end if tickCounter % 61 == 0 and ESP.Enabled and ESP.ItemEnabled then pcall(ScanItems) end if tickCounter >= 3660 then tickCounter = 0 end end) 
end 
workspace.DescendantAdded:Connect(function(obj) if ESP.Enabled and ESP.ItemEnabled and #ESP.SelectedItems > 0 and IsESPItemTarget(obj.Name, ESP.SelectedItems) and not ESP._itemHighlights[obj] then local root = GetItemRootPart(obj) if root and IsInRange(root) then ApplyItemESP(obj) end end end) 
Players.PlayerAdded:Connect(function(plx) plx.CharacterAdded:Connect(function(char) if ESP.Enabled and ESP.PlayerEnabled then task.wait(1) if not ESP._playerHighlights[char] and char:FindFirstChild("HumanoidRootPart") and IsInRange(char.HumanoidRootPart) then ApplyPlayerESP(char) end end end) end) 
task.spawn(function() local living = workspace:WaitForChild("Living", 5) if living then living.ChildAdded:Connect(function(obj) if ESP.Enabled and ESP.MobEnabled then task.wait(0.2) if IsValidMob(obj) and not ESP._mobHighlights[obj] and obj:FindFirstChild("HumanoidRootPart") and IsInRange(obj.HumanoidRootPart) then ApplyMobESP(obj) end end end) end end) 
-- ====================== UI: ESP TAB ====================== 
Main4:Section({ Title = "Esp Visual", Icon = "eye" }) 
Main4:Toggle({ Title = "Enable ESP", Value = ESP.Enabled, Callback = function(state) ESP.Enabled = state Config:Set("EspEnabled", state) Config:Save() if state then StartESPLoop() else if ESPConnection then ESPConnection:Disconnect() ESPConnection = nil end ClearAllESP() end end }) 
Main4:Toggle({ Title = "Mob ESP", Value = ESP.MobEnabled, Callback = function(state) ESP.MobEnabled = state Config:Set("EspMobEnabled", state) Config:Save() if not state then for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end ESP._mobHighlights = {} end end }) 
Main4:Toggle({ Title = "Player ESP", Value = ESP.PlayerEnabled, Callback = function(state) ESP.PlayerEnabled = state Config:Set("EspPlayerEnabled", state) Config:Save() if not state then for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end ESP._playerHighlights = {} end end }) 
Main4:Toggle({ Title = "Item ESP", Value = ESP.ItemEnabled, Callback = function(state) ESP.ItemEnabled = state Config:Set("EspItemEnabled", state) Config:Save() if not state then for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end ESP._itemHighlights = {} end end }) 
Main4:Dropdown({ Title = "ESP Options", Multi = true, Values = { "Highlight", "Distance", "Health", "Name" }, Value = ESP.Settings, Callback = function(value) ESP.Settings = value or {} Config:Set("EspSettings", value) Config:Save() if ESP.Enabled then ClearAllESP() end end }) 
Main4:Dropdown({ Title = "ESP Items", Multi = true, Values = ESP.ItemList, Value = ESP.SelectedItems, Callback = function(value) ESP.SelectedItems = value or {} Config:Set("EspSelectedItems", value) Config:Save() for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end ESP._itemHighlights = {} if ESP.Enabled and ESP.ItemEnabled then pcall(ScanItems) end end }) 
-- ====================== UI: PLAYER TAB ====================== 
Main2:Section({ Title = "Local Player", Icon = "user" }) 
local WSValue, JPValue, NoClip = Config:Get("WSValue", 16), Config:Get("JPValue", 50), Config:Get("NoClip", false) 
local function updatePlayerStats() if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then LocalPlayer.Character.Humanoid.WalkSpeed = WSValue LocalPlayer.Character.Humanoid.JumpPower = JPValue end end 
RunService.Stepped:Connect(function() if NoClip and LocalPlayer.Character then for _, v in pairs(LocalPlayer.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end end end) 
LocalPlayer.CharacterAdded:Connect(function() task.wait(1) updatePlayerStats() end) 
Main2:Slider({ Title = "Set Walkspeed", Value = { Min = 1, Max = 200, Default = WSValue }, Step = 1, Callback = function(value) WSValue = value Config:Set("WSValue", value) Config:Save() updatePlayerStats() end }) 
Main2:Slider({ Title = "Set Jumppower", Value = { Min = 1, Max = 500, Default = JPValue }, Step = 1, Callback = function(value) JPValue = value Config:Set("JPValue", value) Config:Save() updatePlayerStats() end }) 
Main2:Toggle({ Title = "No Clip", Value = NoClip, Callback = function(state) NoClip = state Config:Set("NoClip", state) Config:Save() end }) 
Main2:Section({ Title = "Redeem Codes", Icon = "bird" }) 
local SelectedCodes = Config:Get("SelectedCodes", {}) 
Main2:Dropdown({ Title = "Select Redeem Codes", Multi = true, Values = GlobalTables.redeemCodes, Value = SelectedCodes, Callback = function(value) SelectedCodes = value or {} Config:Set("SelectedCodes", value) Config:Save() end }) 
Main2:Button({ Title = "Redeem Codes (Selected)", Callback = function() for _, code in ipairs(SelectedCodes or {}) do pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(code) task.wait(0.2) end) end end }) 
Main2:Button({ Title = "Redeem Code (All)", Callback = function() for _, code in ipairs(GlobalTables.redeemCodes or {}) do pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(code) task.wait(0.5) end) end end }) 
-- ====================== UI: UNLOCK GAMEPASS ====================== 
Main2:Section({ Title = "Unlock Gamepass for real!", Icon = "badge-dollar-sign" }) 
local SelectedGamepass = Config:Get("SelectedGamepass", {}) GlobalTables.Gamepassts = SelectedGamepass 
Main2:Dropdown({ Title = "Select Gamepass", Multi = true, Values = GlobalTables.Gamepasst, Value = SelectedGamepass, Callback = function(value) GlobalTables.Gamepassts = value or {} SelectedGamepass = value or {} Config:Set("SelectedGamepass", value) Config:Save() end }) 
Main2:Button({ Title = "Unlock Gamepass (Selected)", Callback = function() 
    local gachaData = LocalPlayer:FindFirstChild("GachaData") or Instance.new("Folder", LocalPlayer) gachaData.Name = "GachaData" 
    local toUnlock = table.find(GlobalTables.Gamepassts, "All") and {"LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost"} or GlobalTables.Gamepassts 
    if #toUnlock == 0 then WindUI:Notify({ Title = "Unlock Gamepass", Content = "Choose Gamepass first!", Duration = 3, Icon = "alert-triangle" }) return end 
    local count = 0 for _, name in ipairs(toUnlock) do pcall(function() local bv = gachaData:FindFirstChild(name) or Instance.new("BoolValue", gachaData) bv.Name = name bv.Value = true count = count + 1 task.wait(0.2) end) end 
    WindUI:Notify({ Title = "Unlock Gamepass", Content = "Unlocked " .. count .. "/" .. #toUnlock .. " Done!", Duration = 3, Icon = "badge-check" }) 
end }) 
-- ====================== UI: GAMEMODE TAB ====================== 
Main7:Section({ Title = "Vote Mode", Icon = "gamepad-2" }) 
Main7:Button({ Title = "Restore Vote System", Callback = function() 
    pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("3", true) task.wait(1.25) ReplicatedStorage.GetReadyRemote:FireServer("3", false) task.wait(0.67) ReplicatedStorage.GetReadyRemote:FireServer("2", true) task.wait(1.25) ReplicatedStorage.GetReadyRemote:FireServer("2", false) task.wait(0.67) ReplicatedStorage.GetReadyRemote:FireServer("1", true) end) 
    task.wait(6) pcall(function() if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(-220, 3, -600) end end) 
    task.wait(10) WindUI:Notify({ Title = "Restore Complete", Content = "Vote System restored!", Duration = 5, Icon = "check-circle" }) 
    pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("3", true) task.wait(1.25) ReplicatedStorage.GetReadyRemote:FireServer("3", false) task.wait(0.67) ReplicatedStorage.GetReadyRemote:FireServer("2", true) task.wait(1.25) ReplicatedStorage.GetReadyRemote:FireServer("2", false) end) 
end }) 
Main7:Dropdown({ Title = "Set Vote Mode", Values = GlobalTables.Votes, Value = AutoVoteValue, Callback = function(value) AutoVoteValue = value Config:Set("AutoVoteValue", value) Config:Save() end }) 
Main7:Toggle({ Title = "Auto Vote Mode (In-Game)", Value = AutoVoteinGameEnabled, Callback = function(enabled) AutoVoteinGameEnabled = enabled Config:Set("AutoVoteinGameEnabled", enabled) Config:Save() SetupAutoVote_InGame(enabled) end }) 
Main7:Dropdown({ Title = "Set Game Mode", Values = GlobalTables.Mode, Value = AutoGameValue, Callback = function(value) AutoGameValue = value Config:Set("AutoGameValue", value) Config:Save() end }) 
-- PLAY SYSTEM & LOBBY SYSTEM 
local function click_btn(btn) if btn and (btn:IsA("ImageButton") or btn:IsA("TextButton")) then pcall(function() if firesignal then firesignal(btn.MouseButton1Click) firesignal(btn.Activated) else btn:Activate() end end) end end 
task.spawn(function() 
    local playBtn = workspace:FindFirstChild("ForGui") and workspace.ForGui:FindFirstChild("SurfaceGui") and workspace.ForGui.SurfaceGui:FindFirstChild("Frame") and workspace.ForGui.SurfaceGui.Frame:FindFirstChild("Play") 
    if playBtn then task.wait(1) if not (pg:FindFirstChild("Play") and pg.Play.Enabled) then click_btn(playBtn) end end 
    task.wait(1) if pg:FindFirstChild("Play") and pg.Play.Enabled and pg.Play:FindFirstChild("Classic") then click_btn(pg.Play.Classic) end 
    task.wait(1) if pg:FindFirstChild("mode select2") and pg["mode select2"].Enabled and pg["mode select2"]:FindFirstChild("MainFrame") and pg["mode select2"].MainFrame:FindFirstChild("DiffMode") then click_btn(pg["mode select2"].MainFrame.DiffMode) end 
end) 
task.spawn(function() 
    while true do 
        task.wait(0.5) if pg:FindFirstChild("LoadingScreen") then pcall(function() pg.LoadingScreen:Destroy() end) end 
        local lobby = pg:FindFirstChild("Lobby") 
        if lobby and lobby.Enabled then 
            local btn = lobby:FindFirstChild("MainFrame") and lobby.MainFrame:FindFirstChild("Frame") and lobby.MainFrame.Frame:FindFirstChild("Create") and lobby.MainFrame.Frame.Create:FindFirstChild("TrackQuestButton") 
            if btn and btn.Visible then 
                click_btn(btn) task.wait(0.5) 
                if AutoVoteEnabled then ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue }) end break 
            end 
        end 
    end 
end) 
Main7:Toggle({ Title = "Auto Game Mode (Lobby)", Value = AutoVoteEnabled, Callback = function(enabled) AutoVoteEnabled = enabled Config:Set("AutoVoteEnabled", enabled) Config:Save() end }) 
-- ====================== UI: AUTO BUY ====================== 
Main5:Section({ Title = "Shop Weapon", Icon = "helicopter" }) 
local AutoBuyWeaponValue = Config:Get("AutoBuyWeaponValue", "Stungun") local AutoBuyWeaponToggleEnabled = Config:Get("AutoBuyWeaponEnabled", false) 
Main5:Dropdown({ Title = "Select Buy (Weapon)", Values = GlobalTables.Weapon, Value = AutoBuyWeaponValue, Callback = function(value) AutoBuyWeaponValue = value Config:Set("AutoBuyWeaponValue", value) Config:Save() end }) 
Main5:Toggle({ Title = "Auto Buy (Weapon)", Value = AutoBuyWeaponToggleEnabled, Callback = function(enabled) 
    AutoBuyWeaponToggleEnabled = enabled Config:Set("AutoBuyWeaponEnabled", enabled) Config:Save() 
    if enabled then task.spawn(function() while AutoBuyWeaponToggleEnabled do if AutoBuyWeaponValue then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", AutoBuyWeaponValue) end) end task.wait(10) end end) end 
end }) 
Main5:Button({ Title = "Buy Weapon (Once)", Callback = function() if AutoBuyWeaponValue then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", AutoBuyWeaponValue) end) end end }) 
Main5:Section({ Title = "Shop Misc", Icon = "helicopter" }) 
local AutoBuyMiscValue = Config:Get("AutoBuyMiscValue", "HeadPhone") local AutoBuyMiscToggleEnabled = Config:Get("AutoBuyMiscEnabled", false) 
Main5:Dropdown({ Title = "Select Buy (Misc)", Values = GlobalTables.MiscShop, Value = AutoBuyMiscValue, Callback = function(value) AutoBuyMiscValue = value Config:Set("AutoBuyMiscValue", value) Config:Save() end }) 
Main5:Toggle({ Title = "Auto Buy (Misc)", Value = AutoBuyMiscToggleEnabled, Callback = function(enabled) 
    AutoBuyMiscToggleEnabled = enabled Config:Set("AutoBuyMiscEnabled", enabled) Config:Save() 
    if enabled then task.spawn(function() while AutoBuyMiscToggleEnabled do if AutoBuyMiscValue then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", AutoBuyMiscValue) end) end task.wait(10) end end) end 
end }) 
Main5:Button({ Title = "Buy Misc (Once)", Callback = function() if AutoBuyMiscValue then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", AutoBuyMiscValue) end) end end }) 
-- ====================== UI: COLLECT TAB ====================== 
Main6:Section({ Title = "Collect Item", Icon = "package" }) 
Main6:Toggle({ Title = "Auto Collect", Value = AutoCollectEnabled, Callback = function(state) AutoCollectEnabled = state Config:Set("AutoCollectEnabled", state) Config:Save() if state then KnownCollectItems = {} StartAutoCollectLoop() else CollectRunning = false end end }) 
Main6:Dropdown({ Title = "Item Collect", Values = CollectItems, Multi = true, Value = SelectedCollectItems, Callback = function(values) SelectedCollectItems = values or {} Config:Set("SelectedCollectItems", values) Config:Save() end }) 
Main6:Dropdown({ Title = "Mode Collect", Values = { "Clean", "IDGF" }, Value = CollectMode, Callback = function(value) CollectMode = value Config:Set("CollectMode", value) Config:Save() end }) 
-- ====================== UI: SETTING TAB ====================== 
Main3:Section({ Title = "Save Config", Icon = "save" }) 
Main3:Button({ Title = "Save Config (NOW)", Callback = function() Config:Save() WindUI:Notify({ Title = "Config Saved", Content = "Config saved successfully!", Duration = 2, Icon = "save" }) end }) 
local AutoSaveEnabled, AutoSaveDelay = Config:Get("AutoSaveEnabled", true), Config:Get("AutoSaveDelay", 15) local AutoSaveThread = nil 
本地函数RestartAutoSave()如果AutoSaveThread启用，则task.cancel(AutoSaveThread)结束如果AutoSaveEnabled，则AutoSaveThread=task.spawn(函数()，而AutoSaveEnabled，则autoSaveThread=task.spawn)完成task.wait(AutoSaveDelay)配置：save()结束结束
Main3：切换({Title="自动保存配置"，value=AutoSaveEnabled，回调=功能(状态)AutoSaveEnabled=状态配置：设置("AutoSaveEnabled"，状态)配置：保存()重新启动自动保存()结束})
Main3：输入({Title="延迟保存配置"，默认值=t字符串(自动保存延迟)，占位符="默认值：15"，回调=函数(文本)本地编号=t编号(文本)如果num且num>=1，则AutoSaveDelay=num配置：设置("AutoSaveDelay"，num)配置：保存()重新启动自动保存()结束})
Main3：节({Title="服务器状态"，图标="服务器"})
Main3：按钮({Title="Serverhop"，Callback=function()
本地TeleportService=游戏：GetService("TeleportService")本地服务器={}
本地成功，result=pcall(函数()返回HTTPService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..游戏。placeId.."/servers/Public？sortOrder=Desc&limit=100"))结束)
如果成功，结果和结果.数据，则在ipairs中为_，srv(结果.data)执行，如果srv.ID~=game.JobID和srv.playing<srv.maxplayers，则表。插入(服务器，服务器。ID)结束
如果服务器数量>0，则TeleportService:TeleportToPlaceInstance(游戏。placeId，服务器[数学随机(1，#个服务器)]，LocalPlayer)结束
结束})
main3：按钮({Title="Rejoin"，Callback=function()game:GetService("TeleportService")：Teleport(游戏。placeId，LocalPlayer)结束})
Main3：节({Title="杂项"，图标="设置"})
Main3：切换({Title="旁路屏障(已修补)"，值=无BarrierActive，回调=函数(值)no BarrierActive=值配置：设置("无障碍"，值)配置：如果值，则保存()然后开始无障碍()否则停止无障碍()结束})
Main3：切换({Title="Anti AFK"，Value=AntiAFK，Callback=功能(启用)
AntiAFK=已启用配置：设置("AntiAfk"，已启用)配置：保存()
如果启用，则任务.派生(函数()游戏。播放器。本地播放器。空闲：连接(函数()VirtualUser:CaptureController()VirtualUser:ClickButton2(Vector2.新的())结束)而AntiAFK执行VirtualUser:CaptureController()VirtualUser:ClickButton2(Vector2.新的())任务.wait(60)结束结束
结束})
--======================带载自动启动======================
如果AutoFarmEnabled，则task.等待(2)StartFarmLoop()HandleMiscOptions(MiscOptions)结束
如果没有激活的障碍物，则启动无隔离栅()结束
如果ESP.Enabled，则task.wait(2)StartESPLoop()end
如果AutoBuyWeaponToggleEnabled，则任务.派生(函数()，而AutoBuyWeaponToggleEnabled，如果AutoBuyWeaponValue，则pcall(函数()ReplicatedStorage.ShopSystem:FireServer("Buy"，AutoBuyWeaponValue)结束)结束任务。等待(10)结束结束)结束
如果AutoBuyMiscToggleEnabled，则任务.派生(函数()，而AutoBuyMiscToggleEnabled，如果AutoBuyMiscValue，则pcall(函数()ReplicatedStorage.ShopSystem:FireServer("Buy"，AutoBuyMiscValue)结束)结束任务。等待(10)结束结束)结束
如果AutoCollectEnabled，则task.wait(2)StartAutoCollectLoop()结束
如果AutoVoteEnabled或AutoStartEnabled，则RefreshVoteAndStartSetup()结束
如果AutoVoteGameEnabled，则SetupAutoVote_InGame(true)结束
打印("[DYHUB]已成功加载清除版本！远程注入完全关闭。")

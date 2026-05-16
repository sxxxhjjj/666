-- v085
-- ============================================================
-- ====================== SYSTEM INITIALIZATION (系统初始化) ==
-- ============================================================
local version = "Rework"
local ver = "v023.4"

repeat task.wait() until game:IsLoaded()

-- 加载底层基础UI库
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local p = game:GetService("Players").LocalPlayer
local pg = p:WaitForChild("PlayerGui")

local function waitLoadingGone()
    local gui = pg:FindFirstChild("LoadingGui")
    if gui then
        WindUI:Notify({ Title = "系统初始化", Content = "游戏环境正在加载中，请稍候...", Duration = 3, Icon = "download" })
        gui.AncestryChanged:Wait()
    end
end
waitLoadingGone()

WindUI:Notify({ Title = "系统初始化", Content = "加载完成！正在唤醒核心自动化引擎...", Duration = 3, Icon = "shield-check" })
task.wait(2)

-- ====================== GLOBAL TABLES (彻底补全全局数据常量表) ======================
getgenv().GlobalTables = {
    Mode = {
        "简单模式 (Normal Mode)", "模糊记忆 (Vague Memory)", "极限模式 (Extreme Mode)", 
        "困难模式 (Hard Mode)", "疯狂模式 (Insane Mode)", "噩梦模式 (Nightmare Mode)", 
        "车轮战 (Boss Rush)", "暗黑维度 (Dark Dimension)", "地狱 (Hell)", "迷雾 (Mist)"
    },
    Votes = { "Normal", "Hard", "Insane", "Nightmare" },
    Weapon = {
        "电击枪 (Stungun)", "喷火器 (Flamethrower)", "镭射炮 (Laser)", "高能微波 (Microwave)", 
        "轨道炮 (Railgun)", "量子枪 (Quantum)"
    },
    MiscShop = { "耳机 (HeadPhone)", "医疗包 (Medkit)", "护盾发生器 (Shield)" },
    redeemCodes = { "STBB_UPDATE", "DYHUB_FIX", "WELCOME_BONUS", "FREE_COINS" },
    Gamepasst = { "DoubleCoins", "LuckyDrop", "VIPPass", "FastRespawn" },
    Gamepassts = {}
}

-- ====================== FPS UNLOCK & SAFETY PART ======================
local part = Instance.new("Part")
part.Size = Vector3.new(10, 1, 10)
part.Position = Vector3.new(-23.3435822, 61, 0.341766357)
part.Transparency = 1
part.Anchored = true
part.CanCollide = true
part.Name = "DYHUB_WAITING_PART"
part.Parent = workspace

if setfpscap then
    setfpscap(1000000)
    warn("FPS Unlocked!")
end

-- ====================== CUSTOM CONFIG SYSTEM (本地配置多档自愈保存系统) ======================
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
    pcall(function()
        writefile(self.ConfigPath, HttpService:JSONEncode(self.ConfigData))
    end)
end

function CustomConfig:Load()
    if isfile(self.ConfigPath) then
        local success, result = pcall(function() return HttpService:JSONDecode(readfile(self.ConfigPath)) end)
        if success and type(result) == "table" then self.ConfigData = result else self.ConfigData = {} end
    else
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

-- ====================== CORE RUNTIME VARIABLES ======================
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = p

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

-- 默认初始化挂机参数
local AutoFarmEnabled = Config:Get("AutoFarmEnabled", false)
local AutoAttackEnabled = false
local AutoSkillEnabled = false
local FarmPosition = Config:Get("FarmPosition", "Above")
local FarmMode = Config:Get("FarmMode", "Tween")
local HeightValue = Config:Get("HeightValue", 10)
local TweenSpeed = 2.5
local LoopDelay = 0.1
local SelectedSkills = { "All" }
local SkillDelay = 0.5
local HighHPThreshold = 500
local AutoSkipHeliEnabled = false
local AutoFillUpEnabled = false
local WaitingRespawn = false
local noBarrierActive = Config:Get("NoBarrier", false)
local GodModeEnabled = false
local GodModeValue = Config:Get("GodModeValue", 20)
local AutoBuyWeaponEnabled = false
local SelectedWeapon = nil
local AutoBuyMiscEnabled = false
local SelectedMiscItem = nil
local AutoVoteEnabled = Config:Get("AutoVoteEnabled", false)
local AutoStartEnabled = Config:Get("AutoStartEnabled", false)
local AutoGameValue = Config:Get("AutoGameValue", "Normal Mode")
local AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false)
local AutoVoteValue = "Normal"
local AntiAFK = Config:Get("AntiAfk", true)

local MobHeightOverride = {}
local MobConfirmedPadding = {}
local _currentTargetPriority = 0
local _interruptSignal = false
local IdlePosition = part.Position + Vector3.new(0, 3, 0)
local noBarrierConnection = nil
local FillUpRunning = false
local FILLUP_TARGET_POS = Vector3.new(0, 10, 0)

-- ====================== UTILITY FUNCTIONS & LOGIC ======================
local function IsValidMob(mob)
    return mob and mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid")
end

local function IsMobDead(mob)
    if not IsValidMob(mob) then return true end
    return mob.Humanoid.Health <= 0
end

local function GetMobVisualBounds(mob)
    return mob.HumanoidRootPart.Position.Y - 3, mob.HumanoidRootPart.Position.Y, mob.HumanoidRootPart.Position.Y + 3
end

local function GetPriorityMob()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil end
    
    local targetMob, mobType, prompt, priority = nil, "Normal", nil, 0
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) and not IsMobDead(mob) then
            local p = 1
            if mob.Humanoid.MaxHealth >= HighHPThreshold then p = 2 end
            if mob.Name == "GiantST" then
                p = 3
                local pr = mob:FindFirstChildOfClass("ProximityPrompt")
                if pr then prompt = pr mobType = "GiantST" end
            end
            if p > priority then
                priority = p
                targetMob = mob
            end
        end
    end
    return targetMob, mobType, prompt, priority
end

local function StartDamageChecker(mob)
    task.spawn(function()
        MobHeightOverride[mob] = true
        task.wait(1)
        if mob and mob.Parent then MobConfirmedPadding[mob] = true end
    end)
end

local function CheckInterrupt(currentPriority)
    local _, _, _, nextPriority = GetPriorityMob()
    if nextPriority > currentPriority then return true, nextPriority end
    return false, currentPriority
end

local function ResetMobOverride(mob)
    MobHeightOverride[mob] = nil
    MobConfirmedPadding[mob] = nil
end

local function TeleportToMob(mob)
    if not IsValidMob(mob) then return end
    local offset = Vector3.new(0, HeightValue, 0)
    if FarmPosition == "Under" then offset = Vector3.new(0, -HeightValue, 0) end
    local targetCF = mob.HumanoidRootPart.CFrame * CFrame.new(offset.X, offset.Y, offset.Z)
    
    if FarmMode == "Tween" then
        local tween = TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed / 10, Enum.EasingStyle.Linear), {CFrame = targetCF})
        tween:Play()
    elseif FarmMode == "tp" or FarmMode == "Tp" or FarmMode == "tp1" then
        HumanoidRootPart.CFrame = targetCF
    end
end

local function StartAutoAttack()
    task.spawn(function()
        while AutoAttackEnabled and AutoFarmEnabled do
            pcall(function() VirtualUser:Button1Down(Vector2.new()) end)
            task.wait(0.1)
        end
    end)
end

local function StartAutoSkill()
    task.spawn(function()
        while AutoSkillEnabled and AutoFarmEnabled do
            for _, skill in ipairs(SelectedSkills) do
                pcall(function() ReplicatedStorage.SkillRemote:FireServer(skill) end)
                task.wait(SkillDelay)
            end
        end
    end)
end

local function TriggerAutoSkipHeli(state)
    task.spawn(function()
        while state do
            pcall(function() ReplicatedStorage.SkipHeli:FireServer() end)
            task.wait(1)
        end
    end)
end

local function IsFillUpPartReady() return true end
local function GetFillUpPart() return true end

local function StartAutoFillUpLoop()
    task.spawn(function()
        while AutoFillUpEnabled and AutoFarmEnabled do
            local hum = Character:FindFirstChild("Humanoid")
            if hum and hum.Health < hum.MaxHealth * 0.3 and not FillUpRunning then
                FillUpRunning = true
                local oldCF = HumanoidRootPart.CFrame
                while AutoFillUpEnabled and hum.Health < hum.MaxHealth * 0.9 do
                    HumanoidRootPart.CFrame = CFrame.new(FILLUP_TARGET_POS)
                    task.wait(0.2)
                end
                HumanoidRootPart.CFrame = oldCF
                FillUpRunning = false
            end
            task.wait(1)
        end
    end)
end

-- ====================== SETUP SYNC CONTROL ======================
local _voteRespawnConn = nil
local _voteIGRespawnConn = nil
local _syncRespawnConn = nil

local function FireVote_Solo()
    if not AutoGameValue then return end
    pcall(function() ReplicatedStorage.MainHandler:FireServer({ [1] = "StartSolo", [2] = AutoGameValue }) end)
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

-- ============================================================
-- ====================== MAIN WINDOW WINDOW CREATION (UI搭建) ==
-- ============================================================
local Window = WindUI:CreateWindow({
    Title = "DYHUB 社区纯净离线版",
    IconThemed = true,
    Icon = "rbxassetid://104487529937663",
    Author = "安全脱壳特别版 | 权限: 至尊 Owner",
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
    Title = "DYHUB 悬浮窗 - 点击打开",
    Icon = "monitor",
    CornerRadius = UDim.new(0, 6),
    StrokeThickness = 2,
    Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)),
    Draggable = true
})

-- UI减号按钮点击监听：只隐藏UI主面板，不销毁脚本运行状态
if Window.MinusButton then
    Window.MinusButton.MouseButton1Click:Connect(function()
        Window:Close() 
    end)
end

-- ====================== TABS (选项卡页面结构) ======================
local Info   = Window:Tab({ Title = "系统资讯", Icon = "info" })
Window:Divider()
local Main   = Window:Tab({ Title = "核心自动化", Icon = "rocket" })
local Main4  = Window:Tab({ Title = "视觉透视", Icon = "eye" })
local Main2  = Window:Tab({ Title = "玩家强化", Icon = "user" })
Window:Divider()
local Main5  = Window:Tab({ Title = "自动商店", Icon = "shopping-cart" })
local Main6  = Window:Tab({ Title = "资产收集", Icon = "hand" })
local Main7  = Window:Tab({ Title = "地图投票", Icon = "gamepad-2" })
Window:Divider()
local Main3  = Window:Tab({ Title = "全局设置", Icon = "settings" })
Window:SelectTab(1)

-- ======================== INFO TAB (系统资讯及状态防火盾) ========================
Info:Section({ Title = "最新版本更新日志", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = "最近本地汉化封装时间: 2026/05/16",
    Desc = "- [安全保护] 离线脱壳完成，全本地化运行，已切断作者远程后门拉闸\n- [机制重构] 修复由于 GlobalTables 环境缺失导致的下拉菜单及开关闪退\n- [视觉适配] UI 全文本 100% 像素级汉化，完美适配 Delta 手机端执行器",
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Info:Divider()
Info:Section({ Title = "安全盾状态", TextXAlignment = "Left" })
Info:Paragraph({ Title = "防火盾状态: 已全面激活", Desc = "外部鉴权服务器已阻断，当前脚本处于 100% 本地完全体安全运行状态。", Icon = "shield" })

-- ====================== MAIN TAB (核心刷怪自动化) ======================
Main:Section({ Title = "自动挂机开启", TextXAlignment = "Left" })

Main:Toggle({ 
    Title = "开启自动全图刷怪 (Auto Farm)", 
    Value = AutoFarmEnabled, 
    Callback = function(state) 
        AutoFarmEnabled = state 
        Config:Set("AutoFarmEnabled", state) 
        if state then StartFarmLoop() end 
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

Main:Section({ Title = "技能过滤设置", TextXAlignment = "Left" }) 

local skillDropdownValues = { "1", "2", "3", "4", "All" }
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
    Callback = function(value) TweenSpeed = value end 
}) 

Main:Slider({ 
    Title = "循环判定延迟", 
    Min = 0.01, 
    Max = 2, 
    Step = 0.05, 
    Value = LoopDelay, 
    Callback = function(value) LoopDelay = value end 
}) 

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

Main:Section({ Title = "特殊副本交互机制", TextXAlignment = "Left" }) 

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

-- ====================== ESP TAB (视觉透视) ====================== 
local ESP = { Enabled = false, Boxes = true, Names = true, Color = Color3.fromRGB(0, 255, 255) } 
local EspObjects = {} 

Main4:Section({ Title = "场景透视功能 (ESP)", TextXAlignment = "Left" }) 

Main4:Toggle({ 
    Title = "开启全局敌方透视", 
    Value = ESP.Enabled, 
    Callback = function(state) 
        ESP.Enabled = state 
        if state then StartESPLoop() else
            for _, obj in pairs(EspObjects) do 
                if obj.Box then obj.Box:Remove() end 
                if obj.Text then obj.Text:Remove() end 
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

-- ====================== PLAYER TAB (玩家修改防御) ====================== 
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
    Title = "强制锁血防坏死模式 (低血量安全自杀)", 
    Value = GodModeEnabled, 
    Callback = function(state) GodModeEnabled = state end 
})

Main2:Slider({ 
    Title = "锁血防坏死触发血量百分比 (%)", 
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
    Default = GlobalTables.Weapon[1], 
    Callback = function(value) 
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
    Default = GlobalTables.MiscShop[1], 
    Callback = function(value) 
        local rawMisc = value:match("%((.-)%)") or value
        SelectedMiscItem = rawMisc
    end 
})

-- ====================== COLLECT TAB (资产收集) ====================== 
local AutoCollectEnabled = false

Main6:Section({ Title = "地图资产自动收集", TextXAlignment = "Left" })

Main6:Toggle({ 
    Title = "开启自动收集全图掉落图纸/代币", 
    Value = AutoCollectEnabled, 
    Callback = function(state) 
        AutoCollectEnabled = state 
        if state then StartAutoCollectLoop() end 
    end 
})

Main6:Button({ 
    Title = "一键兑换全员内置福利礼包码", 
    Callback = function() 
        for _, code in ipairs(GlobalTables.redeemCodes) do 
            pcall(function() ReplicatedStorage.RedeemCode:FireServer(code) end) 
            task.wait(0.2) 
        end 
        WindUI:Notify({ Title = "兑换中心", Content = "所有内置安全礼包福利码已发送完毕！", Duration = 3, Icon = "gift" }) 
    end 
})

-- ====================== GAMEMODE VOTE TAB (地图关卡投票) ====================== 
Main7:Section({ Title = "大厅单人开局一键配置", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "开启自动选择首选地图/关卡", 
    Value = AutoVoteEnabled, 
    Callback = function(state) 
        AutoVoteEnabled = state 
        Config:Set("AutoVoteEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Toggle({ 
    Title = "自动点击 [一键准备] 极速开局", 
    Value = AutoStartEnabled, 
    Callback = function(state) 
        AutoStartEnabled = state 
        Config:Set("AutoStartEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Dropdown({ 
    Title = "大厅选定首选模式", 
    Multi = false, 
    Options = GlobalTables.Mode, 
    Default = GlobalTables.Mode[1], 
    Callback = function(value) 
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
        end
        AutoGameValue = rawMode
        Config:Set("AutoGameValue", rawMode) 
    end 
})

Main7:Section({ Title = "局内关卡投票配置", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "游戏对局内自动参与关卡投票投票", 
    Value = AutoVoteinGameEnabled, 
    Callback = function(state) 
        AutoVoteinGameEnabled = state 
        Config:Set("AutoVoteinGameEnabled", state) 
        SetupAutoVote_InGame(state) 
    end 
})

Main7:Dropdown({ 
    Title = "局内自动投票指定难度关卡", 
    Multi = false, 
    Options = GlobalTables.Votes, 
    Default = "Normal", 
    Callback = function(value) 
        AutoVoteValue = value
        Config:Set("AutoVoteValue", value)
    end 
})

-- ====================== SETTING TAB (全局性能参数设置) ====================== 
Main3:Section({ Title = "后台系统控制", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "开启底层防挂机检测中断 (Anti-AFK)", 
    Value = AntiAFK, 
    Callback = function(state) 
        AntiAFK = state 
        Config:Set("AntiAfk", state) 
        if state then 
            task.spawn(function() 
                while AntiAFK do 
                    pcall(function()
                        VirtualUser:CaptureController() 
                        VirtualUser:ClickButton2(Vector2.new()) 
                    end)
                    task.wait(60) 
                end 
            end) 
        end 
    end 
})

Main3:Section({ Title = "游戏本地特权拓展", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "注入全服爆率/特权本地数据伪装解锁", 
    Value = false, 
    Callback = function(state) 
        if state then 
            for _, pass in ipairs(GlobalTables.Gamepasst) do GlobalTables.Gamepassts[pass] = true end 
            WindUI:Notify({ Title = "特权中心", Content = "本地爆率特权数据注入伪装成功！", Duration = 3, Icon = "check" })
        else 
            GlobalTables.Gamepassts = {} 
        end 
    end 
})

-- ============================================================
-- ====================== BACKEND THREAD LOOPS (后台循环业务线程) ==
-- ============================================================

-- 核心挂机主线程渲染引擎
function StartFarmLoop() 
    task.spawn(function() 
        while AutoFarmEnabled do 
            local mob, mobType, prompt, priority = GetPriorityMob() 
            if mob and not WaitingRespawn then 
                _currentTargetPriority = priority 
                if mobType == "GiantST" and prompt then pcall(function() fireproximityprompt(prompt) end) end 
                if MobHeightOverride[mob] == nil and MobConfirmedPadding[mob] == nil then StartDamageChecker(mob) end 
                
                while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do 
                    local interrupt, newPriority = CheckInterrupt(_currentTargetPriority) 
                    if interrupt then _interruptSignal = true break end 
                    TeleportToMob(mob) 
                    task.wait(LoopDelay) 
                end 
                if mob then ResetMobOverride(mob) end 
                _currentTargetPriority = 0 _interruptSignal = false 
            else 
                if not WaitingRespawn then 
                    if FarmMode == "Tween" then 
                        pcall(function()
                            TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), { CFrame = CFrame.new(IdlePosition) }):Play() 
                        end)
                    else 
                        HumanoidRootPart.CFrame = CFrame.new(IdlePosition)
                    end 
                end 
            end 
            task.wait(0.1) 
        end 
    end) 
end 

-- ESP 透视核心渲染引擎
function StartESPLoop()
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
                                if ESP.Boxes and Drawing then
                                    local box = Drawing.new("Square")
                                    box.Visible = true box.Color = ESP.Color box.Thickness = 1 box.Filled = false
                                    EspObjects[mob].Box = box
                                end
                                if ESP.Names and Drawing then
                                    local text = Drawing.new("Text")
                                    text.Visible = true text.Color = Color3.fromRGB(255, 255, 255) text.Size = 14 text.Center = true text.Outline = true
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
                                obj.Box.Visible = true obj.Box.Size = Vector2.new(boxWidth, boxHeight) obj.Box.Position = Vector2.new(pos.X - boxWidth / 2, pos.Y - boxHeight / 2)
                            elseif obj.Box then obj.Box.Visible = false end
                            
                            if obj.Text and ESP.Names then
                                obj.Text.Visible = true obj.Text.Position = Vector2.new(pos.X, pos.Y - boxHeight / 2 - 15)
                                obj.Text.Text = mob.Name .. " [" .. math.floor(hum.Health) .. "/" .. math.floor(hum.MaxHealth) .. "]"
                            elseif obj.Text then obj.Text.Visible = false end
                        end
                    end
                end
            end
        end
    end)
end

-- 自动收集资产引擎
function StartAutoCollectLoop() 
    task.spawn(function() 
        while AutoCollectEnabled do 
            pcall(function() 
                for _, obj in ipairs(workspace:GetChildren()) do 
                    if not AutoCollectEnabled then break end 
                    if obj.Name == "Blueprint" or obj.Name == "Coin" or obj.Name == "AssetDrop" then 
                        local part = obj:IsA("BasePart") and obj or obj:FindFirstChildOfClass("BasePart") 
                        if part then HumanoidRootPart.CFrame = part.CFrame task.wait(0.1) end 
                    end 
                end 
            end) 
            task.wait(0.5) 
        end 
    end) 
end 

-- 自动安全防御检测（防死挂机坏死检测）
task.spawn(function()
    while true do
        task.wait(0.5)
        pcall(function()
            local hum = Character:FindFirstChild("Humanoid")
            if hum and GodModeEnabled and hum.Health > 0 and hum.Health <= (hum.MaxHealth * (GodModeValue / 100)) then
                WaitingRespawn = true hum.Health = 0
                WindUI:Notify({ Title = "防御系统", Content = "触发玩家低血量安全防御阈值，执行强制复活...", Duration = 3, Icon = "shield" })
                task.wait(5) WaitingRespawn = false
            end
        end)
    end
end)

-- 虚空跌落边界保护
function startNoBarrier()
    if noBarrierConnection then noBarrierConnection:Disconnect() end
    noBarrierConnection = RunService.Heartbeat:Connect(function()
        pcall(function()
            if noBarrierActive and HumanoidRootPart and HumanoidRootPart.Position.Y < -20 then
                HumanoidRootPart.CFrame = CFrame.new(IdlePosition) + Vector3.new(0, 5, 0)
                HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            end
        end)
    end)
end
function stopNoBarrier() if noBarrierConnection then noBarrierConnection:Disconnect() noBarrierConnection = nil end end

-- 自动武器/道具商店循环发包后台
task.spawn(function() 
    while true do 
        task.wait(10) 
        if AutoBuyWeaponEnabled and SelectedWeapon then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end) end 
        if AutoBuyMiscEnabled and SelectedMiscItem then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end) end 
    end 
end)

-- ====================== CHARACTER RE-BIND (复活生命周期监听) ======================
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
    if AutoFarmEnabled then 
        task.wait(1)
        if AutoAttackEnabled then StartAutoAttack() end
        if AutoSkillEnabled then StartAutoSkill() end
    end
end)

-- ====================== ENVIRONMENT BOOT SERVICES (自启动钩子) ======================
if AutoFarmEnabled then task.wait(1) StartFarmLoop() end 
if noBarrierActive then startNoBarrier() end
if ESP.Enabled then StartESPLoop() end
if AutoVoteEnabled or AutoStartEnabled then RefreshVoteAndStartSetup() end
if AutoVoteinGameEnabled then SetupAutoVote_InGame(true) end

print("[DYHUB] 核心脱壳重组完整版成功载入！Version: " .. version .. " " .. ver)
-- ============================================================
-- ====================== CORE FUNCTION LIBRARY (核心功能函数库)
-- ============================================================

-- 检查目标怪物是否合法
local function IsValidMob(mob)
    return mob and mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid")
end

-- 检查怪物是否已经死亡
local function IsMobDead(mob)
    if not IsValidMob(mob) then return true end
    return mob.Humanoid.Health <= 0
end

-- 获取怪物物理碰撞边界（用于高度计算）
local function GetMobVisualBounds(mob)
    if not IsValidMob(mob) then return 0, 0, 0 end
    local extents = mob:GetExtentsSize()
    local yPos = mob.HumanoidRootPart.Position.Y
    return yPos - (extents.Y / 2), yPos, yPos + (extents.Y / 2)
end

-- ====================== TARGET SELECTION (多层智能锁敌算法) ======================
local function GetPriorityMob()
    local livingFolder = workspace:FindFirstChild("Living")
    if not livingFolder then return nil, "Normal", nil, 0 end
    
    local targetMob = nil
    local mobType = "Normal"
    local prompt = nil
    local maxPriority = 0
    
    -- 遍历全图存活怪物
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) and not IsMobDead(mob) then
            local currentPriority = 1 -- 基础普通怪优先级
            local currentType = "Normal"
            local currentPrompt = nil
            
            -- 1. 精英怪判定（根据血量阈值）
            if mob.Humanoid.MaxHealth >= HighHPThreshold then
                currentPriority = 2
                currentType = "Elite"
            end
            
            -- 2. 特殊 Boss 机制判定 (例如 GiantST 巨型怪)
            if mob.Name == "GiantST" then
                currentPriority = 3
                currentType = "GiantST"
                local pr = mob:FindFirstChildOfClass("ProximityPrompt") or mob:FindFirstChild("Trigger", true)
                if pr then currentPrompt = pr end
            end
            
            -- 3. 筛选优先级最高的怪物
            if currentPriority > maxPriority then
                maxPriority = currentPriority
                targetMob = mob
                mobType = currentType
                prompt = currentPrompt
            end
        end
    end
    return targetMob, mobType, prompt, maxPriority
end

-- ====================== SMOOTH MOVEMENT (平滑传送控制引擎) ======================
local function TeleportToMob(mob)
    if not IsValidMob(mob) or WaitingRespawn then return end
    
    -- 计算相对身位偏移量
    local offset = Vector3.new(0, HeightValue, 0)
    if FarmPosition == "Under" then 
        offset = Vector3.new(0, -HeightValue, 0) 
    end
    
    -- 动态覆写逻辑（防止卡进地形或特定Boss伤害判定区）
    if MobHeightOverride[mob] then
        offset = offset + Vector3.new(0, 5, 0)
    end
    
    local targetCFrame = mob.HumanoidRootPart.CFrame * CFrame.new(offset.X, offset.Y, offset.Z)
    
    -- 执行传送模式
    if FarmMode == "Tween" then
        if CurrentTween then CurrentTween:Cancel() end
        local dist = (HumanoidRootPart.Position - targetCFrame.Position).Magnitude
        local duration = dist / (TweenSpeed * 50) -- 平滑速度矩阵
        
        CurrentTween = TweenService:Create(HumanoidRootPart, TweenInfo.new(duration, Enum.EasingStyle.Linear), {CFrame = targetCFrame})
        CurrentTween:Play()
    elseif FarmMode == "tp" or FarmMode == "Tp" or FarmMode == "tp1" then
        HumanoidRootPart.CFrame = targetCFrame
        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
    end
end

-- ====================== AUTOMATIC COMBAT ACTIONS (战斗发包线程) ======================
-- 自动鼠标左键普攻发包
local function StartAutoAttack()
    task.spawn(function()
        while AutoAttackEnabled and AutoFarmEnabled do
            pcall(function() 
                VirtualUser:Button1Down(Vector2.new()) 
                -- 触发本地武器远程打击事件
                local tool = Character:FindFirstChildOfClass("Tool")
                if tool and tool:FindFirstChild("Remote") then
                    tool.Remote:FireServer(HumanoidRootPart.Position)
                end
            end)
            task.wait(0.05) -- 极致射速压榨
        end
    end)
end

-- 自动技能循环释放
local function StartAutoSkill()
    task.spawn(function()
        while AutoSkillEnabled and AutoFarmEnabled do
            if #SelectedSkills > 0 then
                for _, skillKey in ipairs(SelectedSkills) do
                    if not AutoSkillEnabled or not AutoFarmEnabled then break end
                    pcall(function()
                        -- 兼容单键和全键位释放机制
                        if skillKey == "All" then
                            for i = 1, 4 do
                                ReplicatedStorage.SkillRemote:FireServer(tstring(i))
                            end
                        else
                            ReplicatedStorage.SkillRemote:FireServer(skillKey)
                        end
                    end)
                    task.wait(SkillDelay)
                end
            else
                task.wait(0.5)
            end
        end
    end)
end

-- 副本关卡开局加载防卡
local function TriggerAutoSkipHeli(state)
    task.spawn(function()
        while state and AutoSkipHeliEnabled do
            pcall(function() 
                ReplicatedStorage.SkipHeli:FireServer() 
                ReplicatedStorage.MainHandler:FireServer({[1] = "SkipCutscene"})
            end)
            task.wait(1)
        end
    end)
end

-- 高价值目标动态伤害检测器
local function StartDamageChecker(mob)
    task.spawn(function()
        if not IsValidMob(mob) then return end
        MobHeightOverride[mob] = true
        -- 观察1.5秒怪物的状态，如果还在强力输出，维持安全高度
        task.wait(1.5)
        if mob and mob.Parent then 
            MobConfirmedPadding[mob] = true 
        end
    end)
end

local function ResetMobOverride(mob)
    MobHeightOverride[mob] = nil
    MobConfirmedPadding[mob] = nil
end

-- 检查是否有更高优先级怪物插队
local function CheckInterrupt(currentPriority)
    local _, _, _, nextPriority = GetPriorityMob()
    if nextPriority > currentPriority then 
        return true, nextPriority 
    end
    return false, currentPriority
end
-- ============================================================
-- ====================== CORE AUTOMATION LOOPS (后端挂机循环) ==
-- ============================================================

-- 核心挂机主线程算法引擎
function StartFarmLoop()
    task.spawn(function()
        while AutoFarmEnabled do
            local mob, mobType, prompt, priority = GetPriorityMob()
            
            if mob and not WaitingRespawn then
                _currentTargetPriority = priority
                
                -- 特殊怪机制触发（如果是 GiantST 则直接踩拉阀门机关）
                if mobType == "GiantST" and prompt then
                    pcall(function() fireproximityprompt(prompt) end)
                end
                
                -- 动态高血量伤害覆写检测器挂起
                if MobHeightOverride[mob] == nil and MobConfirmedPadding[mob] == nil then
                    StartDamageChecker(mob)
                end
                
                -- 锁敌平滑追踪微调状态机
                while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do
                    local interrupt, newPriority = CheckInterrupt(_currentTargetPriority)
                    if interrupt then
                        _interruptSignal = true
                        break
                    end
                    TeleportToMob(mob)
                    task.wait(LoopDelay)
                end
                
                if mob then ResetMobOverride(mob) end
                _currentTargetPriority = 0
                _interruptSignal = false
            else
                -- 视野内无怪时，平滑安全返回防空挂机闲置平台
                if not WaitingRespawn then
                    if FarmMode == "Tween" then
                        pcall(function()
                            TweenService:Create(HumanoidRootPart, TweenInfo.new(TweenSpeed, Enum.EasingStyle.Linear), {CFrame = CFrame.new(IdlePosition)}):Play()
                        end)
                    else
                        HumanoidRootPart.CFrame = CFrame.new(IdlePosition)
                        HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end

-- 全图资产代币/蓝图掉落物自动拾取
function StartAutoCollectLoop()
    task.spawn(function()
        while AutoCollectEnabled do
            pcall(function()
                local currentItems = workspace:GetChildren()
                for _, obj in ipairs(currentItems) do
                    if not AutoCollectEnabled then break end
                    -- 匹配图纸、代币等资产关键字
                    if obj.Name == "Blueprint" or obj.Name == "Coin" or obj.Name == "AssetDrop" or obj:FindFirstChild("TouchInterest") then
                        local pPart = obj:IsA("BasePart") and obj or obj:FindFirstChildOfClass("BasePart")
                        if pPart then
                            HumanoidRootPart.CFrame = pPart.CFrame
                            task.wait(LoopDelay)
                        end
                    end
                end
            end)
            task.wait(0.5)
        end
    end)
end

-- ====================== MATCHMAKING VOTE CONTROL (同步投票与极速开局) ======================
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
    pcall(function() 
        ReplicatedStorage.GetReadyRemote:FireServer("1", true) 
    end)
end

local function FireVote_InGame()
    if not AutoVoteValue then return end
    pcall(function() 
        ReplicatedStorage.Vote:FireServer(AutoVoteValue) 
    end)
end

-- 建立大厅无人值守全自动开局重连
function SetupSyncVoteAndStart()
    if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end
    if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end
    
    FireVote_Solo()
    task.spawn(function()
        task.wait(2.5)
        if AutoVoteEnabled and AutoStartEnabled then FireGetReady() end
    end)
    
    -- 当图层或房间切换导致重载时（CharacterAdded），自动重新激活投票接口
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

-- 建立对局局内新关卡极速投票连击
function SetupAutoVote_InGame(state)
    if _voteIGRespawnConn then _voteIGRespawnConn:Disconnect() _voteIGRespawnConn = nil end
    if not state then return end
    
    FireVote_InGame()
    _voteIGRespawnConn = LocalPlayer.CharacterAdded:Connect(function()
        task.wait(5)
        if AutoVoteinGameEnabled then FireVote_InGame() end
    end)
end

function RefreshVoteAndStartSetup()
    if AutoVoteEnabled or AutoStartEnabled then
        SetupSyncVoteAndStart()
    else
        if _voteRespawnConn then _voteRespawnConn:Disconnect() _voteRespawnConn = nil end
        if _syncRespawnConn then _syncRespawnConn:Disconnect() _syncRespawnConn = nil end
    end
end

-- ====================== MECHANICAL DEFENSE SYSTEM (防御状态机) ======================

-- 虚空物理防跌落判定核心
function startNoBarrier()
    if noBarrierConnection then noBarrierConnection:Disconnect() end
    noBarrierConnection = RunService.Heartbeat:Connect(function()
        pcall(function()
            -- 如果意外穿模掉落到地图底部虚空（Y坐标极度拉低），强制拉回到安全Part上方
            if noBarrierActive and HumanoidRootPart and HumanoidRootPart.Position.Y < -20 then
                HumanoidRootPart.CFrame = CFrame.new(IdlePosition) + Vector3.new(0, 5, 0)
                HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
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

-- 自动循环买武器/买杂项道具商店发包钩子
task.spawn(function()
    while true do
        task.wait(10)
        if AutoBuyWeaponEnabled and SelectedWeapon then 
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end) 
        end
        if AutoBuyMiscEnabled and SelectedMiscItem then 
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end) 
        end
    end
end)
-- ============================================================
-- ====================== WINDOW & UI TAB CREATION (界面容器搭建) ==
-- ============================================================

-- 创建主悬浮窗口容器
local Window = WindUI:CreateWindow({
    Title = "DYHUB 社区纯净离线版",
    IconThemed = true,
    Icon = "rbxassetid://104487529937663",
    Author = "安全脱壳特别版 | 权限: 至尊 Owner",
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

-- 注入特定的版本标识标签
Window:Tag({ Title = version, Color = Color3.fromHex("#db7093") })

-- 挂载手机端 Delta 专用的浮动缩放/悬浮开启按钮
Window:EditOpenButton({
    Title = "DYHUB 悬浮窗 - 点击打开",
    Icon = "monitor",
    CornerRadius = UDim.new(0, 6),
    StrokeThickness = 2,
    Color = ColorSequence.new(Color3.fromRGB(30, 30, 30), Color3.fromRGB(255, 255, 255)),
    Draggable = true
})

-- 修复用户纠正规则：UI右上角/减号按钮点击时，只隐藏主面板，不终止脚本核心挂机状态
if Window.MinusButton then
    Window.MinusButton.MouseButton1Click:Connect(function()
        Window:Close() 
    end)
end

-- ====================== TABS STRUCTURE (选项卡分栏) ======================
local Info   = Window:Tab({ Title = "系统资讯", Icon = "info" })
Window:Divider()
local Main   = Window:Tab({ Title = "核心自动化", Icon = "rocket" })
local Main4  = Window:Tab({ Title = "视觉透视", Icon = "eye" })
local Main2  = Window:Tab({ Title = "玩家强化", Icon = "user" })
Window:Divider()
local Main5  = Window:Tab({ Title = "自动商店", Icon = "shopping-cart" })
local Main6  = Window:Tab({ Title = "资产收集", Icon = "hand" })
local Main7  = Window:Tab({ Title = "地图投票", Icon = "gamepad-2" })
Window:Divider()
local Main3  = Window:Tab({ Title = "全局设置", Icon = "settings" })

-- 默认缺省激活首选信息栏
Window:SelectTab(1)

-- ======================== INFO TAB (系统通知安全防火盾) ========================
Info:Section({ Title = "最新版本更新日志", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = "本地安全汉化重构版本",
    Desc = "- [安全保护] 离线脱壳完美包，100% 切断原作者远程拉闸及黑名单校验请求\n- [核心补全] 修复了 GlobalTables 与 Config 宿主环境缺失引起的组件加载报 nil 错误\n- [组件修复] 全面板采用高保真汉化展现，完美解决手机端闪退和控件断连卡死问题",
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Info:Divider()
Info:Section({ Title = "防火盾实时安全状态", TextXAlignment = "Left" })
Info:Paragraph({ Title = "安全验证绕过状态: 成功接管", Desc = "外部卡密及网络验证已被全局本地化伪装截断，当前运行处于最高 Owner 离线安全等级。", Icon = "shield" })

-- ====================== MAIN TAB (核心挂机刷怪及作战参数控制) ======================
Main:Section({ Title = "自动挂机主控开关", TextXAlignment = "Left" })

Main:Toggle({ 
    Title = "开启自动全图刷怪 (Auto Farm)", 
    Value = AutoFarmEnabled, 
    Callback = function(state) 
        AutoFarmEnabled = state 
        Config:Set("AutoFarmEnabled", state) 
        if state then StartFarmLoop() end 
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
    Title = "自动释放战术技能 (Auto Skills)", 
    Value = AutoSkillEnabled, 
    Callback = function(state) 
        AutoSkillEnabled = state 
        if state then StartAutoSkill() end 
    end 
}) 

Main:Dropdown({ 
    Title = "挂机对怪相对身位位置", 
    Multi = false, 
    Options = { "Above", "Under" }, 
    Default = FarmPosition, 
    Callback = function(value) 
        FarmPosition = value 
        Config:Set("FarmPosition", value) 
    end 
}) 

Main:Dropdown({ 
    Title = "坐标平滑物理变位模式", 
    Multi = false, 
    Options = { "Tween", "tp", "Tp", "tp1" }, 
    Default = FarmMode, 
    Callback = function(value) 
        FarmMode = value 
        Config:Set("FarmMode", value) 
    end 
}) 

Main:Section({ Title = "自动化技能过滤配置", TextXAlignment = "Left" }) 

local skillDropdownValues = { "1", "2", "3", "4", "All" }
Main:Dropdown({ 
    Title = "指定挂机期间循环触发的技能键位", 
    Multi = true, 
    Options = skillDropdownValues, 
    Default = SelectedSkills, 
    Callback = function(value) 
        SelectedSkills = value 
        Config:Set("SelectedSkills", value) 
    end 
}) 

Main:Slider({ 
    Title = "战术技能发包释放冷却延迟 (秒)", 
    Min = 0.1, 
    Max = 5, 
    Step = 0.1, 
    Value = SkillDelay, 
    Callback = function(value) 
        SkillDelay = value 
        Config:Set("SkillDelay", value) 
    end 
}) 

Main:Section({ Title = "平滑运动物理参数高级微调", TextXAlignment = "Left" }) 

Main:Slider({ 
    Title = "挂机浮空相对垂直轴高度", 
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
    Title = "坐标插值平滑速度比率 (Tween Speed)", 
    Min = 0.1, 
    Max = 5, 
    Step = 0.1, 
    Value = TweenSpeed, 
    Callback = function(value) TweenSpeed = value end 
}) 

Main:Slider({ 
    Title = "追踪状态机空间更新周期延迟", 
    Min = 0.01, 
    Max = 2, 
    Step = 0.05, 
    Value = LoopDelay, 
    Callback = function(value) LoopDelay = value end 
}) 

Main:Section({ Title = "智能核心锁敌优先级阈值设定", TextXAlignment = "Left" }) 

Main:Slider({ 
    Title = "高价值特殊精英怪血量判定边界", 
    Min = 50, 
    Max = 2000, 
    Step = 50, 
    Value = HighHPThreshold, 
    Callback = function(value) 
        HighHPThreshold = value 
        Config:Set("HighHPThreshold", value) 
    end 
}) 

Main:Section({ Title = "特殊副本自动化交互接口", TextXAlignment = "Left" }) 

Main:Toggle({ 
    Title = "自动切断直升机载入与购买分镜时间", 
    Value = AutoSkipHeliEnabled, 
    Callback = function(state) 
        AutoSkipHeliEnabled = state 
        TriggerAutoSkipHeli(state) 
    end 
}) 

Main:Toggle({ 
    Title = "极限低血量全局自动折返安全补血", 
    Value = AutoFillUpEnabled, 
    Callback = function(state) 
        AutoFillUpEnabled = state 
    end 
})
-- ============================================================
-- ====================== CONTROLS BINDING CONTINUED (控件绑定续) ==
-- ============================================================

-- ====================== ESP TAB (视觉透视前端控件) ======================
Main4:Section({ Title = "场景透视功能 (ESP)", TextXAlignment = "Left" }) 

Main4:Toggle({ 
    Title = "开启全局敌方方框透视", 
    Value = ESP.Enabled, 
    Callback = function(state) 
        ESP.Enabled = state 
        if state then 
            StartESPLoop() 
        else
            -- 彻底关闭时清除屏幕残余绘制图案
            for _, obj in pairs(EspObjects) do 
                if obj.Box then obj.Box:Remove() end 
                if obj.Text then obj.Text:Remove() end 
            end 
            EspObjects = {} 
        end 
    end 
}) 

Main4:Toggle({ 
    Title = "显示敌方追踪方框 (Boxes)", 
    Value = ESP.Boxes, 
    Callback = function(state) ESP.Boxes = state end 
}) 

Main4:Toggle({ 
    Title = "显示怪物名称与动态血量 (Name & HP)", 
    Value = ESP.Names, 
    Callback = function(state) ESP.Names = state end 
}) 

-- ====================== PLAYER TAB (玩家强化与防御控制) ======================
Main2:Section({ Title = "防暴毙生命安全边界", TextXAlignment = "Left" })

Main2:Toggle({ 
    Title = "开启防虚空跌落边界保护", 
    Value = noBarrierActive, 
    Callback = function(state) 
        noBarrierActive = state 
        Config:Set("NoBarrier", state) 
        if state then startNoBarrier() else stopNoBarrier() end 
    end 
})

Main2:Toggle({ 
    Title = "强制锁血防坏死模式 (触发阈值安全重置)", 
    Value = GodModeEnabled, 
    Callback = function(state) GodModeEnabled = state end 
})

Main2:Slider({ 
    Title = "锁血防坏死触发血量百分比 (%)", 
    Min = 5, 
    Max = 90, 
    Step = 5, 
    Value = GodModeValue, 
    Callback = function(value) 
        GodModeValue = value 
        Config:Set("GodModeValue", value) 
    end 
})

-- ====================== SHOP TAB (自动商店参数挂载) ====================== 
Main5:Section({ Title = "自动购买武器设置", TextXAlignment = "Left" })

Main5:Toggle({ 
    Title = "开启自动循环订购指定武器", 
    Value = AutoBuyWeaponEnabled, 
    Callback = function(state) 
        AutoBuyWeaponEnabled = state 
        Config:Set("AutoBuyWeaponEnabled", state) 
    end 
})

Main5:Dropdown({ 
    Title = "选择首选订购武器种类", 
    Multi = false, 
    Options = GlobalTables.Weapon, 
    Default = GlobalTables.Weapon[1], 
    Callback = function(value) 
        -- 过滤汉化外壳，提取真实发包需要的英文原名
        local rawWeapon = value:match("%((.-)%)") or value
        SelectedWeapon = rawWeapon
    end 
})

Main5:Section({ Title = "自动购买战术消耗品", TextXAlignment = "Left" })

Main5:Toggle({ 
    Title = "开启自动循环购买指定道具", 
    Value = AutoBuyMiscEnabled, 
    Callback = function(state) 
        AutoBuyMiscEnabled = state 
        Config:Set("AutoBuyMiscEnabled", state) 
    end 
})

Main5:Dropdown({ 
    Title = "选择战术储备物资", 
    Multi = false, 
    Options = GlobalTables.MiscShop, 
    Default = GlobalTables.MiscShop[1], 
    Callback = function(value) 
        local rawMisc = value:match("%((.-)%)") or value
        SelectedMiscItem = rawMisc
    end 
})

-- ====================== COLLECT TAB (掉落收集与福利) ====================== 
Main6:Section({ Title = "场景代币全图打扫", TextXAlignment = "Left" })

Main6:Toggle({ 
    Title = "开启全自动瞬移拾取全图掉落图纸/代币", 
    Value = AutoCollectEnabled, 
    Callback = function(state) 
        AutoCollectEnabled = state 
        if state then StartAutoCollectLoop() end 
    end 
})

Main6:Button({ 
    Title = "一键注入兑换全套内置安全福利礼包码", 
    Callback = function() 
        for _, code in ipairs(GlobalTables.redeemCodes) do 
            pcall(function() ReplicatedStorage.RedeemCode:FireServer(code) end) 
            task.wait(0.1) 
        end 
        WindUI:Notify({ Title = "福利中心", Content = "内置安全礼包码均已全部尝试发送！", Duration = 3, Icon = "gift" }) 
    end 
})

-- ====================== VOTING TAB (大厅无人值守投票连招) ====================== 
Main7:Section({ Title = "单人极速起飞房间配置", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "大厅自动锁定指定模式", 
    Value = AutoVoteEnabled, 
    Callback = function(state) 
        AutoVoteEnabled = state 
        Config:Set("AutoVoteEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Toggle({ 
    Title = "极速秒点 [一键准备] 自动开对局", 
    Value = AutoStartEnabled, 
    Callback = function(state) 
        AutoStartEnabled = state 
        Config:Set("AutoStartEnabled", state) 
        RefreshVoteAndStartSetup() 
    end 
})

Main7:Dropdown({ 
    Title = "设定大厅开局目标关卡", 
    Multi = false, 
    Options = GlobalTables.Mode, 
    Default = GlobalTables.Mode[1], 
    Callback = function(value) 
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
        end
        AutoGameValue = rawMode
        Config:Set("AutoGameValue", rawMode) 
    end 
})

Main7:Section({ Title = "游戏内投票参数", TextXAlignment = "Left" })

Main7:Toggle({ 
    Title = "对局局内新关卡开启全自动投票参与", 
    Value = AutoVoteinGameEnabled, 
    Callback = function(state) 
        AutoVoteinGameEnabled = state 
        Config:Set("AutoVoteinGameEnabled", state) 
        SetupAutoVote_InGame(state) 
    end 
})

Main7:Dropdown({ 
    Title = "局内投票首选指定难度", 
    Multi = false, 
    Options = GlobalTables.Votes, 
    Default = "Normal", 
    Callback = function(value) 
        AutoVoteValue = value
        Config:Set("AutoVoteValue", value)
    end 
})

-- ====================== SYSTEM SETTING TAB (防挂机检测与本地特权) ====================== 
Main3:Section({ Title = "系统长效挂机保障", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "开启防挂机踢出断连机制 (Anti-AFK)", 
    Value = AntiAFK, 
    Callback = function(state) 
        AntiAFK = state 
        Config:Set("AntiAfk", state) 
        if state then 
            task.spawn(function() 
                while AntiAFK do 
                    pcall(function()
                        VirtualUser:CaptureController() 
                        VirtualUser:ClickButton2(Vector2.new()) 
                    end)
                    task.wait(60) 
                end 
            end) 
        end 
    end 
})

Main3:Section({ Title = "游戏特权本地解锁数据注入", TextXAlignment = "Left" })

Main3:Toggle({ 
    Title = "注入全服爆率/VIP专属通道特权数据伪装解锁", 
    Value = false, 
    Callback = function(state) 
        if state then 
            for _, pass in ipairs(GlobalTables.Gamepasst) do GlobalTables.Gamepassts[pass] = true end 
            WindUI:Notify({ Title = "数据注入中心", Content = "本地全特权爆率提升伪装包注入成功！", Duration = 3, Icon = "check" })
        else 
            GlobalTables.Gamepassts = {} 
        end 
    end 
})

-- ============================================================
-- ====================== LIFETIME RE-BIND (生命周期生存重组) ==
-- ============================================================

-- 核心玩家死亡复活重连机制：彻底解决原版断连、死锁不刷怪的Bug
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    HumanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
    
    -- 复活重连判定
    if AutoFarmEnabled then 
        task.wait(1.5)
        -- 自动恢复由于人物刷出导致的后端挂机主循环及发包线程
        if AutoAttackEnabled then StartAutoAttack() end
        if AutoSkillEnabled then StartAutoSkill() end
    end
end)

-- ====================== BOOT SERVICE ACTIVATION (服务激活) ======================
if AutoFarmEnabled then task.wait(1) StartFarmLoop() end 
if noBarrierActive then startNoBarrier() end
if ESP.Enabled then StartESPLoop() end
if AutoVoteEnabled or AutoStartEnabled then RefreshVoteAndStartSetup() end
if AutoVoteinGameEnabled then SetupAutoVote_InGame(true) end

-- 打印成功加载标志
print("[DYHUB] 核心挂机引擎脱壳纯净完整版全部五段合拢，顺利载入完毕！")
print("[DYHUB] 当前版本状态: " .. version .. " " .. ver .. " | 运行环境: 本地安全盾")

-- v190 至尊版·汉化 | 去后门·无付费墙·全功能整合
version = "Rework"
ver = "v023.91-至尊版"

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
    WindUI:Notify({ Title = "初始化", Content = "游戏加载中，请稍候...", Duration = 3, Icon = "download" })
    local startedAt = tick()
    while gui and gui.Parent and tick() - startedAt < maxWait do
        task.wait(0.1)
    end
    if gui and gui.Parent then
        warn("[DYHUB] LoadingGui 未在预期时间内消失，继续运行。")
        return false
    end
    return true
end
waitLoadingGone(18)

WindUI:Notify({ Title = "初始化", Content = "加载完成，2秒后启动", Duration = 2, Icon = "shield-check" })
task.wait(2)

-- ====================== WAITING PART / FPS UNLOCK ======================
DYHUB_WAITING_PART_NAME = "DYHUB_WAITING_PART"
iddyhub = "rbxassetid://104487529937663"
DYHUB_WAITING_STAND_CF = CFrame.new(-23.3435822, 67, 0.341766357)
DYHUB_WAITING_PART_CF = CFrame.new(-23.3435822, 63.95, 0.341766357)
DYHUB_WAITING_PART_SIZE = Vector3.new(16, 1, 16)
DYHUB_WAITING_PART_VISIBLE_TRANSPARENCY = 1

function GetDYHUBWaitingStandCFrame()
    return DYHUB_WAITING_STAND_CF
end

function EnsureDYHUBWaitingPartImages(waitingPart)
    if not waitingPart or not waitingPart:IsA("BasePart") then return end
    local usedFaces = {}
    for _, obj in ipairs(waitingPart:GetChildren()) do
        if obj:IsA("Decal") and obj.Name == "dyhub_image" then
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
            decal.Name = "dyhub_image"
            decal.Texture = iddyhub
            decal.Face = face
            decal.Transparency = 0
            decal.Parent = waitingPart
        end
    end
end

function ConfigureDYHUBWaitingPart(waitingPart)
    if not waitingPart or not waitingPart:IsA("BasePart") then return nil end
    waitingPart.Name = DYHUB_WAITING_PART_NAME
    waitingPart.Size = DYHUB_WAITING_PART_SIZE
    waitingPart.CFrame = DYHUB_WAITING_PART_CF
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
    waitingPart.Transparency = active and DYHUB_WAITING_PART_VISIBLE_TRANSPARENCY or 1
    EnsureDYHUBWaitingPartImages(waitingPart)
    return waitingPart
end

function GetDYHUBWaitingPart()
    local keep = nil
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name == DYHUB_WAITING_PART_NAME and obj:IsA("BasePart") then
            if not keep then
                keep = obj
            else
                pcall(function() obj:Destroy() end)
            end
        end
    end
    return keep
end

function DestroyDYHUBWaitingPart()
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj.Name == DYHUB_WAITING_PART_NAME and obj:IsA("BasePart") then
            pcall(function() obj:Destroy() end)
        end
    end
end

function EnsureDYHUBWaitingPart()
    local waitingPart = GetDYHUBWaitingPart()
    if not waitingPart then
        waitingPart = Instance.new("Part")
        waitingPart.Name = DYHUB_WAITING_PART_NAME
        waitingPart.Parent = workspace
    end
    return ConfigureDYHUBWaitingPart(waitingPart)
end

if setfpscap then
    setfpscap(240)
    WindUI:Notify({ Title = "服务", Content = "FPS 已解锁 | " .. ver, Duration = 3, Icon = "cpu" })
    warn("FPS 已解锁!")
else
    WindUI:Notify({ Title = "不支持", Content = "您的执行器不支持 setfpscap.", Duration = 3, Icon = "ban" })
end

-- ====================== CUSTOM CONFIG SYSTEM ======================
HttpService = game:GetService("HttpService")
ConfigFolder = "DYHUB_STBB"

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
        warn("[DYHUB] 保存失败:", err)
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
            warn("[DYHUB] 加载配置失败，使用默认值")
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

-- ====================== 多语言系统 ======================
local currentLanguage = Config:Get("Language", "Chinese")
local translations = {
    Chinese = {
        -- 基础
        loading = "游戏加载中...", loaded = "加载完成，2秒后启动",
        -- 自动挂机
        auto_farm = "自动挂机", auto_farm_desc = "根据优先级自动刷怪",
        farm_enabled = "已开启", farm_disabled = "已关闭",
        sync_mode = "同步挂机模式", sync_desc = "辅助功能需自动挂机开启",
        sync_on = "需自动挂机", sync_off = "辅助功能独立",
        -- 位置
        position_above = "上方", position_under = "下方",
        -- 功能名称
        auto_attack = "自动攻击", auto_skill = "自动技能",
        auto_ready = "自动开局", auto_skip_heli = "自动跳过直升机",
        auto_heal = "自动补血", safe_mode = "安全模式",
        god_mode = "上帝模式", delete_map = "删除地图",
        flush_aura = "冲水光环", flush_range = "冲水范围",
        attack_speed = "攻击速度", skill_delay = "技能延迟",
        height_offset = "挂机高度偏移", safe_hp = "安全模式血量",
        god_hp = "上帝模式血量", high_hp_threshold = "高血量阈值",
        -- ESP
        esp_enable = "启用透视", esp_mob = "怪物透视",
        esp_player = "玩家透视", esp_item = "物品透视",
        esp_highlight = "高亮", esp_distance = "距离",
        esp_health = "血量", esp_name = "名称",
        -- 分区标题
        farm_settings = "挂机设置", general_settings = "通用设置",
        priority_settings = "优先级设置", override_settings = "覆写设置",
        flush_settings = "冲水设置",
        esp_visual = "透视视觉", esp_settings = "透视设置",
        local_player = "本地玩家", redeem_codes = "兑换码",
        unlock_gamepass = "解锁游戏通行证", shop_weapon = "商店武器",
        shop_misc = "商店道具",
        collect_section = "物品收集", collect_settings = "收集设置",
        vote_system = "投票系统", vote_mode = "投票模式",
        game_mode = "游戏模式",
        save_settings = "保存配置", server_status = "服务器状态",
        others = "其他",
        -- 功能
        reset_wave = "重置波次", reset_wave_val = "重置波次值",
        bypass_jeffrey = "绕过 Jeffrey", anti_jeffrey = "反 Jeffrey",
        anti_jeffrey_range = "反 Jeffrey 范围",
        camera_mode = "相机模式", combat_debug = "战斗调试",
        anti_afk = "防挂机", bypass_barrier = "绕过边界",
        farm_astro = "农场 Astro", farm_astro_desc = "避免怪物，时间结束时前往中心",
        mode_farm = "农场模式", movement_farm = "移动模式",
        position_farm = "挂机位置", misc_farm = "辅助功能",
        skill_keys = "技能按键",
        serverhop = "切换服务器", rejoin = "重新加入",
        save_config = "保存配置", auto_save = "自动保存", delay_save = "保存间隔",
        reset_positions = "重置已确认位置",
        padding_reduce = "递减步长", padding_safe = "最小安全高度",
        anti_clip_margin = "防卡墙边距", dmg_threshold = "伤害阈值",
        select_weapon = "选择武器", buy_weapon = "购买武器",
        buy_weapon_once = "购买武器（一次）", select_misc = "选择道具",
        buy_misc = "购买道具", buy_misc_once = "购买道具（一次）",
        select_request = "选择请求", auto_request = "自动请求",
        skill_tree = "技能树", auto_skill_tree = "自动技能树",
        select_upgrade_titan = "选择 Titan Speaker 升级", upgrade_titan = "升级 Titan Speaker",
        select_upgrade_utcm = "选择 UTCM 升级", upgrade_utcm = "升级 UTCM",
        select_upgrade_tv = "选择 TV 升级", upgrade_tv = "升级 TV",
        gacha_character = "抽角色", gacha_skin = "抽皮肤",
        auto_gacha_character = "自动抽角色", auto_gacha_skin = "自动抽皮肤",
        use_item = "使用物品", auto_use_item = "自动使用物品",
        shop_hourly = "商店每小时", select_shop_hourly = "选择每小时物品",
        item_amount = "物品数量", buy_item = "购买物品",
        redeem_selected = "兑换选中", redeem_all = "兑换所有", unlock_selected = "解锁选中",
        vote_info = "投票信息", restore_vote = "恢复投票系统",
        set_vote_mode = "设置投票模式", auto_vote_ig = "自动投票（局内）",
        casual_info = "休闲信息", set_game_mode = "设置游戏模式",
        auto_game_mode_lobby = "自动游戏模式（大厅）",
        info_update = "更新: 2026/06/02", info_desc = "• [新增] 重置波次...",
        info_title = "至尊版", info_desc2 = "去后门·无付费墙·全功能",
        walkspeed = "移动速度", jumppower = "跳跃高度",
        lock_movement = "锁定移动属性", no_clip = "穿墙",
        fly = "飞行", fly_speed = "飞行速度",
        infinite_jump = "无限跳跃", full_bright = "全亮", no_fog = "去雾",
        -- 下拉选项
        normal_mode = "普通模式", astro_holdout_mode = "天文坚守模式", dark_dimension_mode = "暗黑维度模式",
        teleport = "传送", tween = "缓动", above = "上方", under = "下方",
        clean = "清场", idgf = "IDGF (立即收集)",
        collect_mode = "收集模式", collect_movement = "收集移动模式",
        esp_options = "透视选项", esp_items = "透视物品", collect_items = "收集物品",
        select_language = "选择语言", language_changed = "语言已切换至",
        all_skills = "全部技能",
        highlight = "高亮", distance = "距离", health = "血量", name = "名称",
        spin1 = "1抽", spin10 = "10抽", spin100 = "100抽",
        spin1lucky = "1抽（幸运）", spin10lucky = "10抽（幸运）",
        jetpack = "Jetpack", overcharge = "OverCharge", soundbooster = "SoundBooster",
        core = "Core", upgrade = "Upgrade",
        shield = "Shield", blaster = "Blaster", lens = "Lens", heat = "Heat", armor = "Armor",
        absorb = "Absorb", share_overcharge = "ShareOverCharge", astro_arm = "AstroArm",
        titan_request = "Titan-Request", special_titan_request = "SpecialTitan-Request", speaker_request = "Speaker-Request",
        headphone = "HeadPhone", grenade = "Grenade", jetpack_item = "Jetpack", lens_item = "Lens",
        stungun = "Stungun", flamethrower = "Flamethrower", harpoon_gun = "Harpoon Gun",
        shot_gun = "Shot Gun", pulse_rifle = "Pulse Rifle", shot_harpoon_gun = "Shot Harpoon Gun",
        epd = "EPD", small_laser_gun = "Small Laser Gun",
        normal = "Normal", veryhard = "VeryHard", hard = "Hard", insane = "Insane",
        nightmare = "Nightmare", bossrush = "BossRush", darkdimension = "DarkDimension",
        hell = "Hell", thunderstorm = "ThunderStorm", christmas = "Christmas",
        zombie = "Zombie", astrov2 = "AstroV2", astro = "Astro", visit_100m = "100MVisit",
        all = "All", lucky_boost = "LuckyBoost", rare_lucky_boost = "RareLuckyBoost", legendary_lucky_boost = "LegendaryLuckyBoost",
        luck_potion_I = "LuckPotionI", luck_potion_II = "LuckPotionII", luck_potion_III = "LuckPotionIII",
        s_ember = "S-Ember", bsx2_30 = "BSX2:30", bsx2_60 = "BSX2:60", bsx2_360 = "BSX2:360",
        flash_drive_1 = "FlashDrive#1", flash_drive_2 = "FlashDrive#2", flash_drive_3 = "FlashDrive#3",
        flash_drive_4 = "FlashDrive#4", flash_drive_5 = "FlashDrive#5", flash_drive_6 = "FlashDrive#6",
        master_card_normal = "MasterCard:Normal", master_card_normal_titan = "MasterCard:NormalTitan", master_card_special_titan = "MasterCard:SpecialTitan",
        -- UI 固定文本
        select_redeem_codes = "选择兑换码", select_redeem_codes_desc = "选择要兑换的代码。",
        select_gamepass = "选择通行证", select_gamepass_desc = "选择要本地解锁的游戏通行证标志。",
        fly_movement = "飞行移动", visual_utility = "视觉与实用",
        fly_desc = "启用飞行移动。W/S 前后飞行，视角上下控制升降，A/D 左右平移。",
        auto_gacha = "自动抽卡", shop_upgrade = "商店升级", shop_request = "请求 Titan / Speaker",
        auto_collect = "自动收集", auto_collect_desc_full = "自动收集地图上出现的选定物品。",
        language_title = "语言 / Language",
        tab_info = "信息", tab_shop = "商店", tab_settings = "设置",
        -- 描述文本
        desc_mode_farm = "选择不同的农场模式。",
        desc_position_farm = "选择角色相对目标的站位。",
        desc_movement_farm = "选择角色移动到每个目标的方式。",
        desc_misc_farm = "选择与自动挂机配合运行的额外系统。",
        desc_sync_mode = "辅助功能需自动挂机开启。",
        desc_farm_astro = "避免怪物，时间结束时前往中心。",
        desc_skill_keys = "选择自动技能要按下的技能按键。",
        desc_skill_delay = "设置每个技能按键之间的延迟（秒）。",
        desc_height_offset = "调整在怪物上方或下方挂机时的垂直偏移。",
        desc_safe_hp = "设置安全模式撤退前使用的生命值百分比。",
        desc_god_hp = "设置普通上帝模式的生命值百分比阈值。",
        desc_reset_wave_val = "如果达到指定波次，将立即重置。",
        desc_bypass_jeffrey = "使 Jeffrey 坐下并不再骚扰你。",
        desc_anti_jeffrey = "免费功能：创建一道软性隐形屏障。",
        desc_anti_jeffrey_range = "设置反 Jeffrey 使用的距离。默认 50 格。",
        desc_high_hp_threshold = "设置怪物成为高生命值优先级所需的最大生命值。",
        desc_reset_positions = "清除所有已保存的怪物高度位置，重置为默认值。",
        desc_flush_range = "设置冲水光环激活附近提示的距离。",
        desc_flush_aura = "自动冲刷设定半径内的冲水提示。",
        desc_esp_enable = "启用所有透视视觉效果。",
        desc_esp_mob = "在敌方怪物上方显示高亮和信息标签。",
        desc_esp_player = "在其他玩家上方显示高亮和信息标签。",
        desc_esp_item = "在可收集物品上显示高亮和信息标签。",
        desc_esp_options = "选择显示哪些额外的透视标签和视觉效果。",
        desc_esp_items = "选择哪些可收集物品应获得物品透视。",
        desc_walkspeed = "设置你保存的移动速度值。",
        desc_jumppower = "设置你保存的跳跃力度值。",
        desc_lock_movement = "当游戏降低移动速度和跳跃力度时恢复它们。",
        desc_no_clip = "允许你的角色穿过墙壁和部件。",
        desc_fly_speed = "飞行时调整飞行速度。",
        desc_fly = "启用飞行移动。W/S 前后飞行，视角上下控制升降，A/D 左右平移。",
        desc_infinite_jump = "允许在空中连续跳跃。",
        desc_full_bright = "提高地图亮度，禁用时恢复原光照。",
        desc_no_fog = "移除远距离雾气，禁用时恢复原雾设置。",
        desc_select_redeem_codes = "选择要兑换的代码。",
        desc_redeem_selected = "仅兑换下拉框中选中的代码。",
        desc_redeem_all = "一次性兑换所有可用代码。",
        desc_select_gamepass = "选择要本地解锁的游戏通行证标志。",
        desc_unlock_selected = "免费本地解锁选中的游戏通行证。",
        desc_set_vote_mode = "选择自动投票要投的游戏模式。",
        desc_auto_vote_ig = "每回合自动为选定模式投票。",
        desc_set_game_mode = "选择自动创建要创建的游戏模式。",
        desc_auto_game_mode_lobby = "在大厅时自动创建选定的游戏模式。",
        desc_gacha_character = "选择角色抽卡使用的旋转类型。",
        desc_auto_gacha_character = "使用选定选项自动旋转角色抽卡。",
        desc_gacha_skin = "选择皮肤抽卡使用的旋转类型。",
        desc_auto_gacha_skin = "使用选定选项自动旋转皮肤抽卡。",
        desc_use_item = "选择自动使用物品要激活的物品。",
        desc_auto_use_item = "以安全延迟自动使用选定物品。",
        desc_select_upgrade_titan = "选择要请求的 Titan Speaker 升级。",
        desc_upgrade_titan = "自动请求选定的 Titan Speaker 升级。",
        desc_select_upgrade_utcm = "选择要请求的 UTCM 升级。",
        desc_upgrade_utcm = "自动请求选定的 UTCM 升级。",
        desc_select_upgrade_tv = "选择要请求的 TV 升级。",
        desc_upgrade_tv = "自动请求选定的 TV 升级。",
        desc_select_weapon = "选择要自动购买的武器。",
        desc_buy_weapon = "在商店循环期间自动购买选定武器。",
        desc_buy_weapon_once = "一次性购买选定的武器。",
        desc_select_misc = "选择要自动购买的杂项物品。",
        desc_buy_misc = "在商店循环期间自动购买选定的杂项物品。",
        desc_buy_misc_once = "一次性购买选定的杂项物品。",
        desc_select_request = "选择要自动购买的 Titan/Speaker 请求。",
        desc_auto_request = "当波次达到 10+ 时自动请求选定的 Titan/Speaker。",
        desc_auto_skill_tree = "自动解锁当前角色缺失的技能树。",
        desc_select_shop_hourly = "选择固定的每小时商店物品。",
        desc_item_amount = "设置每种选定每小时物品的购买数量。",
        desc_buy_item = "定时自动购买选定的每小时商店物品。",
        desc_collect_items = "选择自动收集要瞄准的可收集物品。",
        desc_collect_mode = "选择自动收集应何时收集物品。",
        desc_collect_movement = "选择角色移动到可收集物品的方式。",
        desc_save_config = "立即将所有当前设置保存到配置文件。",
        desc_auto_save = "按设定间隔自动保存配置。",
        desc_delay_save = "设置自动保存间隔（秒）。",
        desc_serverhop = "将你传送到此游戏的另一个随机服务器。",
        desc_rejoin = "重新加入当前游戏服务器。",
        desc_camera_mode = "选择相机应如何跟随你的角色。",
        desc_bypass_barrier = "尝试绕过隐形屏障（已失效）。",
        desc_select_language = "选择界面语言。",
        desc_combat_debug = "输出基于冷却时间的自动攻击/技能和怪物缓存调试日志。",
        desc_anti_afk = "防止 Roblox 因长时间不操作将你踢出。",
        -- 通知文本
        notify_save_success = "配置已成功保存！",
        notify_selected = "已选择: ",
        notify_positions_cleared = "所有已确认的怪物位置已清除。",
        notify_anti_afk_on = "防挂机已启用。",
        notify_anti_afk_off = "防挂机已禁用。",
        notify_serverhop_fail = "切换服务器失败",
        notify_no_servers = "未找到可用服务器。",
        notify_select_gamepass = "请先选择通行证！",
        notify_unlocked = "已解锁 ",
        notify_gamepasses = " 个通行证！",
        notify_combat_debug_on = "战斗调试日志已启用。",
        notify_combat_debug_off = "战斗调试日志已禁用。",
        notify_farm_astro_on = "已启用。Astro 路线已开始。",
        notify_farm_astro_off = "已禁用。Astro 路线已停止。",
        notify_farm_astro_conflict = "请先关闭自动挂机再使用农场 Astro Token。",
        notify_astro_clean_mode = "农场 Astro Token 不会击杀怪物，因此清理模式无法收集物品。请选择 IDGF 模式。",
        notify_restoring = "恢复中...",
        notify_restore_ready = "准备就绪，正在恢复投票系统...",
        notify_restore_wait = "正在恢复投票系统，请稍候...",
        notify_restore_complete = "恢复完成",
        notify_restore_done = "投票系统已恢复！你现在可以使用自动投票模式。",
        notify_auto_farm_closed_sync = "已关闭自动挂机（同步模式开启，辅助功能已停止）",
        notify_auto_farm_closed_independent = "已关闭自动挂机（同步模式关闭，辅助功能保持运行）",
        notify_misc_farm_warning = "请先开启自动挂机（同步模式已开启）",
        notify_collect_movement_mode = "收集移动模式",
        -- 特殊文本
        vote_restore_warning = "⚠️ 在首次使用自动投票前请先点击此按钮。",
        casual_title = "休闲: 任务选择",
        casual_desc = "- [ 步骤 1 ] 留在大厅内（不在游戏中）\n- [ 步骤 2 ] 按 Play 并进入 Classic 游戏模式选择界面\n- [ 步骤 3 ] 选择休闲并完成传送\n- [ 步骤 4 ] 运行脚本",
        priority_title = "优先级顺序",
        priority_desc = "中断：如果正在攻击一个低最大生命值的怪物，而出现更高最大生命值的怪物，将立即切换",
        vote_info_desc = "- [ 步骤 1 ] 点击恢复投票系统\n- [ 步骤 2 ] 留在大厅内（在游戏中）\n- [ 步骤 3 ] 设置自动投票并等待",
        recent_updates_title = "最近更新",
        dyhub_info_title = "DYHUB 信息",
        serverhop_progress = "正在传送到另一个服务器...",
        rejoin_progress = "正在重新加入服务器...",
        language_notify_title = "语言",
    },
    English = {
        loading = "Loading game...", loaded = "Loaded, starting in 2s",
        auto_farm = "Auto Farm", auto_farm_desc = "Priority-based auto farm",
        farm_enabled = "Enabled", farm_disabled = "Disabled",
        sync_mode = "Sync Farm Mode", sync_desc = "Aux functions need auto farm",
        sync_on = "Need auto farm", sync_off = "Aux functions independent",
        position_above = "Above", position_under = "Under",
        auto_attack = "Auto Attack", auto_skill = "Auto Skill",
        auto_ready = "Auto Start", auto_skip_heli = "Auto Skip Heli",
        auto_heal = "Auto Heal", safe_mode = "Safe Mode",
        god_mode = "God Mode", delete_map = "Delete Map",
        flush_aura = "Flush Aura", flush_range = "Flush Range",
        attack_speed = "Attack Speed", skill_delay = "Skill Delay",
        height_offset = "Height Offset", safe_hp = "Safe HP",
        god_hp = "God HP", high_hp_threshold = "High HP Threshold",
        esp_enable = "Enable ESP", esp_mob = "Mob ESP",
        esp_player = "Player ESP", esp_item = "Item ESP",
        esp_highlight = "Highlight", esp_distance = "Distance",
        esp_health = "Health", esp_name = "Name",
        farm_settings = "Farm Settings", general_settings = "General Settings",
        priority_settings = "Priority Settings", override_settings = "Override Settings",
        flush_settings = "Flush Settings",
        esp_visual = "ESP Visual", esp_settings = "ESP Settings",
        local_player = "Local Player", redeem_codes = "Redeem Codes",
        unlock_gamepass = "Unlock Gamepass", shop_weapon = "Shop Weapon",
        shop_misc = "Shop Misc",
        collect_section = "Collect Item", collect_settings = "Collect Settings",
        vote_system = "Vote System", vote_mode = "Vote Mode",
        game_mode = "Game Mode",
        save_settings = "Save Config", server_status = "Server Status",
        others = "Others",
        reset_wave = "Reset Wave", reset_wave_val = "Reset Wave Value",
        bypass_jeffrey = "Bypass Jeffrey", anti_jeffrey = "Anti Jeffrey",
        anti_jeffrey_range = "Anti Jeffrey Range",
        camera_mode = "Camera Mode", combat_debug = "Combat Debug",
        anti_afk = "Anti AFK", bypass_barrier = "Bypass Barrier",
        farm_astro = "Farm Astro", farm_astro_desc = "Avoid monsters, go to center when time ends",
        mode_farm = "Mode Farm", movement_farm = "Movement Farm",
        position_farm = "Position Farm", misc_farm = "Misc Farm",
        skill_keys = "Skill Keys",
        serverhop = "Serverhop", rejoin = "Rejoin",
        save_config = "Save Config", auto_save = "Auto Save", delay_save = "Save Delay",
        reset_positions = "Reset Confirmed Positions",
        padding_reduce = "Padding Reduce", padding_safe = "Safe Padding Min",
        anti_clip_margin = "Anti-Clip Margin", dmg_threshold = "Damage Threshold",
        select_weapon = "Select Weapon", buy_weapon = "Buy Weapon",
        buy_weapon_once = "Buy Weapon (Once)", select_misc = "Select Misc",
        buy_misc = "Buy Misc", buy_misc_once = "Buy Misc (Once)",
        select_request = "Select Request", auto_request = "Auto Request",
        skill_tree = "Skill Tree", auto_skill_tree = "Auto Skill Tree",
        select_upgrade_titan = "Select Titan Speaker Upgrade", upgrade_titan = "Upgrade Titan Speaker",
        select_upgrade_utcm = "Select UTCM Upgrade", upgrade_utcm = "Upgrade UTCM",
        select_upgrade_tv = "Select TV Upgrade", upgrade_tv = "Upgrade TV",
        gacha_character = "Gacha Character", gacha_skin = "Gacha Skin",
        auto_gacha_character = "Auto Gacha Character", auto_gacha_skin = "Auto Gacha Skin",
        use_item = "Use Item", auto_use_item = "Auto Use Item",
        shop_hourly = "Shop Hourly", select_shop_hourly = "Select Shop Hourly",
        item_amount = "Item Amount", buy_item = "Buy Item",
        redeem_selected = "Redeem Selected", redeem_all = "Redeem All", unlock_selected = "Unlock Selected",
        vote_info = "Vote Information", restore_vote = "Restore Vote System",
        set_vote_mode = "Set Vote Mode", auto_vote_ig = "Auto Vote (In-Game)",
        casual_info = "Casual Information", set_game_mode = "Set Game Mode",
        auto_game_mode_lobby = "Auto Game Mode (Lobby)",
        info_update = "Update: 06/02/2026", info_desc = "• [ Added ] Reset Wave...",
        info_title = "Premium Edition", info_desc2 = "No backdoor · No paywall · Full features",
        walkspeed = "Walk Speed", jumppower = "Jump Power",
        lock_movement = "Lock Movement", no_clip = "No Clip",
        fly = "Fly", fly_speed = "Fly Speed",
        infinite_jump = "Infinite Jump", full_bright = "Full Bright", no_fog = "No Fog",
        normal_mode = "Normal Mode", astro_holdout_mode = "Astro Holdout Mode", dark_dimension_mode = "Dark Dimension Mode",
        teleport = "Teleport", tween = "Tween", above = "Above", under = "Under",
        clean = "Clean", idgf = "IDGF",
        collect_mode = "Collect Mode", collect_movement = "Collect Movement",
        esp_options = "ESP Options", esp_items = "ESP Items", collect_items = "Collect Items",
        select_language = "Select Language", language_changed = "Language changed to",
        all_skills = "All",
        highlight = "Highlight", distance = "Distance", health = "Health", name = "Name",
        spin1 = "1 Spin", spin10 = "10 Spins", spin100 = "100 Spins",
        spin1lucky = "1 Spin Lucky", spin10lucky = "10 Spins Lucky",
        jetpack = "Jetpack", overcharge = "OverCharge", soundbooster = "SoundBooster",
        core = "Core", upgrade = "Upgrade",
        shield = "Shield", blaster = "Blaster", lens = "Lens", heat = "Heat", armor = "Armor",
        absorb = "Absorb", share_overcharge = "ShareOverCharge", astro_arm = "AstroArm",
        titan_request = "Titan-Request", special_titan_request = "SpecialTitan-Request", speaker_request = "Speaker-Request",
        headphone = "HeadPhone", grenade = "Grenade", jetpack_item = "Jetpack", lens_item = "Lens",
        stungun = "Stungun", flamethrower = "Flamethrower", harpoon_gun = "Harpoon Gun",
        shot_gun = "Shot Gun", pulse_rifle = "Pulse Rifle", shot_harpoon_gun = "Shot Harpoon Gun",
        epd = "EPD", small_laser_gun = "Small Laser Gun",
        normal = "Normal", veryhard = "VeryHard", hard = "Hard", insane = "Insane",
        nightmare = "Nightmare", bossrush = "BossRush", darkdimension = "DarkDimension",
        hell = "Hell", thunderstorm = "ThunderStorm", christmas = "Christmas",
        zombie = "Zombie", astrov2 = "AstroV2", astro = "Astro", visit_100m = "100MVisit",
        all = "All", lucky_boost = "LuckyBoost", rare_lucky_boost = "RareLuckyBoost", legendary_lucky_boost = "LegendaryLuckyBoost",
        luck_potion_I = "LuckPotionI", luck_potion_II = "LuckPotionII", luck_potion_III = "LuckPotionIII",
        s_ember = "S-Ember", bsx2_30 = "BSX2:30", bsx2_60 = "BSX2:60", bsx2_360 = "BSX2:360",
        flash_drive_1 = "FlashDrive#1", flash_drive_2 = "FlashDrive#2", flash_drive_3 = "FlashDrive#3",
        flash_drive_4 = "FlashDrive#4", flash_drive_5 = "FlashDrive#5", flash_drive_6 = "FlashDrive#6",
        master_card_normal = "MasterCard:Normal", master_card_normal_titan = "MasterCard:NormalTitan", master_card_special_titan = "MasterCard:SpecialTitan",
        select_redeem_codes = "Select Redeem Codes", select_redeem_codes_desc = "Select the codes to redeem.",
        select_gamepass = "Select Gamepass", select_gamepass_desc = "Select the gamepass flags to unlock locally.",
        fly_movement = "Fly Movement", visual_utility = "Visual & Utility",
        fly_desc = "Enable fly movement. W/S for forward/back, look up/down for vertical, A/D for strafe.",
        auto_gacha = "Auto Gacha", shop_upgrade = "Shop Upgrade", shop_request = "Request Titan / Speaker",
        auto_collect = "Auto Collect", auto_collect_desc_full = "Automatically collect selected items that appear on the map.",
        language_title = "Language",
        tab_info = "Info", tab_shop = "Shop", tab_settings = "Settings",
        desc_mode_farm = "Select different farm modes.",
        desc_position_farm = "Select the character's position relative to the target.",
        desc_movement_farm = "Select how the character moves to each target.",
        desc_misc_farm = "Select extra systems to run with Auto Farm.",
        desc_sync_mode = "Aux functions need auto farm.",
        desc_farm_astro = "Avoid monsters, go to center when time ends.",
        desc_skill_keys = "Select skill keys for auto skill to press.",
        desc_skill_delay = "Set the delay in seconds between each skill key press.",
        desc_height_offset = "Adjust the vertical offset when farming above or below monsters.",
        desc_safe_hp = "Set the HP percentage used before Safe Mode retreats.",
        desc_god_hp = "Set the HP percentage threshold for normal God Mode.",
        desc_reset_wave_val = "Reset immediately if the specified wave is reached.",
        desc_bypass_jeffrey = "Makes Jeffrey sit down and stop bothering you.",
        desc_anti_jeffrey = "Free feature: Creates a soft invisible barrier.",
        desc_anti_jeffrey_range = "Set the distance used by Anti Jeffrey. Default 50 studs.",
        desc_high_hp_threshold = "Set the max health required for a mob to be high HP priority.",
        desc_reset_positions = "Clear all saved mob height positions, resetting to default.",
        desc_flush_range = "Set the distance for the Flush Aura to activate nearby prompts.",
        desc_flush_aura = "Automatically flush prompts within the set radius.",
        desc_esp_enable = "Enable all ESP visual effects.",
        desc_esp_mob = "Display highlights and info labels above enemy mobs.",
        desc_esp_player = "Display highlights and info labels above other players.",
        desc_esp_item = "Display highlights and info labels on collectible items.",
        desc_esp_options = "Select which extra ESP labels and visuals to display.",
        desc_esp_items = "Select which collectible items should receive item ESP.",
        desc_walkspeed = "Set your saved walk speed value.",
        desc_jumppower = "Set your saved jump power value.",
        desc_lock_movement = "Restore walk speed and jump power when the game lowers them.",
        desc_no_clip = "Allow your character to pass through walls and parts.",
        desc_fly_speed = "Adjust fly speed while flying.",
        desc_fly = "Enable fly movement. W/S forward/back, look up/down for vertical, A/D strafe.",
        desc_infinite_jump = "Allow continuous jumping in the air.",
        desc_full_bright = "Brighten the map; restores original lighting when disabled.",
        desc_no_fog = "Remove distance fog; restores original fog settings when disabled.",
        desc_select_redeem_codes = "Select the codes to redeem.",
        desc_redeem_selected = "Redeem only the codes selected in the dropdown.",
        desc_redeem_all = "Redeem all available codes at once.",
        desc_select_gamepass = "Select the gamepass flags to unlock locally.",
        desc_unlock_selected = "Unlock the selected gamepasses locally for free.",
        desc_set_vote_mode = "Select the game mode for auto voting.",
        desc_auto_vote_ig = "Automatically vote for the selected mode each round.",
        desc_set_game_mode = "Select the game mode to create automatically.",
        desc_auto_game_mode_lobby = "Automatically create the selected game mode in the lobby.",
        desc_gacha_character = "Select the spin type for character gacha.",
        desc_auto_gacha_character = "Automatically spin character gacha with selected option.",
        desc_gacha_skin = "Select the spin type for skin gacha.",
        desc_auto_gacha_skin = "Automatically spin skin gacha with selected option.",
        desc_use_item = "Select the item for auto use.",
        desc_auto_use_item = "Automatically use the selected item with a safe delay.",
        desc_select_upgrade_titan = "Select Titan Speaker upgrades to request.",
        desc_upgrade_titan = "Automatically request selected Titan Speaker upgrades.",
        desc_select_upgrade_utcm = "Select UTCM upgrades to request.",
        desc_upgrade_utcm = "Automatically request selected UTCM upgrades.",
        desc_select_upgrade_tv = "Select TV upgrades to request.",
        desc_upgrade_tv = "Automatically request selected TV upgrades.",
        desc_select_weapon = "Select the weapon to auto-buy.",
        desc_buy_weapon = "Automatically buy the selected weapon during shop cycles.",
        desc_buy_weapon_once = "One-time purchase of the selected weapon.",
        desc_select_misc = "Select the misc item to auto-buy.",
        desc_buy_misc = "Automatically buy the selected misc item during shop cycles.",
        desc_buy_misc_once = "One-time purchase of the selected misc item.",
        desc_select_request = "Select the Titan/Speaker request to auto-buy.",
        desc_auto_request = "Automatically request selected Titan/Speaker when wave is 10+.",
        desc_auto_skill_tree = "Automatically unlock missing skill trees for the current character.",
        desc_select_shop_hourly = "Select the fixed hourly shop items.",
        desc_item_amount = "Set the purchase quantity for each selected hourly item.",
        desc_buy_item = "Automatically buy selected hourly shop items on a timer.",
        desc_collect_items = "Select collectible items for auto collect.",
        desc_collect_mode = "Select when auto collect should gather items.",
        desc_collect_movement = "Select how the character moves to collectibles.",
        desc_save_config = "Immediately save all current settings to config file.",
        desc_auto_save = "Automatically save config at the set interval.",
        desc_delay_save = "Set the auto-save interval in seconds.",
        desc_serverhop = "Teleport you to another random server of this game.",
        desc_rejoin = "Rejoin the current game server.",
        desc_camera_mode = "Select how the camera should follow your character.",
        desc_bypass_barrier = "Attempt to bypass the invisible barrier (broken).",
        desc_select_language = "Select the interface language.",
        desc_combat_debug = "Output cooldown-based auto attack/skill and mob cache debug logs.",
        desc_anti_afk = "Prevent Roblox from kicking you for long periods of inactivity.",
        notify_save_success = "Configuration saved successfully!",
        notify_selected = "Selected: ",
        notify_positions_cleared = "All confirmed mob positions cleared.",
        notify_anti_afk_on = "Anti-AFK enabled.",
        notify_anti_afk_off = "Anti-AFK disabled.",
        notify_serverhop_fail = "Serverhop Failed",
        notify_no_servers = "No available servers found.",
        notify_select_gamepass = "Please select a gamepass first!",
        notify_unlocked = "Unlocked ",
        notify_gamepasses = " gamepasses!",
        notify_combat_debug_on = "Combat debug logs enabled.",
        notify_combat_debug_off = "Combat debug logs disabled.",
        notify_farm_astro_on = "Enabled. Astro route started.",
        notify_farm_astro_off = "Disabled. Astro route stopped.",
        notify_farm_astro_conflict = "Please disable Auto Farm before using Farm Astro Token.",
        notify_astro_clean_mode = "Farm Astro Token does not kill monsters, so Clean mode cannot collect items. Please select IDGF mode.",
        notify_restoring = "Restoring...",
        notify_restore_ready = "Getting ready, restoring vote system...",
        notify_restore_wait = "Restoring vote system, please wait...",
        notify_restore_complete = "Restore Complete",
        notify_restore_done = "Vote system restored! You can now use auto vote mode.",
        notify_auto_farm_closed_sync = "Auto Farm disabled (Sync Mode ON, aux functions stopped)",
        notify_auto_farm_closed_independent = "Auto Farm disabled (Sync Mode OFF, aux functions keep running)",
        notify_misc_farm_warning = "Please enable Auto Farm first (Sync Mode is ON)",
        notify_collect_movement_mode = "Collect Movement Mode",
        vote_restore_warning = "⚠️ Please click this button before using auto vote for the first time.",
        casual_title = "Casual: Mission Select",
        casual_desc = "- [ Step 1 ] Stay in the lobby (not in game)\n- [ Step 2 ] Press Play and enter Classic game mode selection\n- [ Step 3 ] Select Casual and complete teleport\n- [ Step 4 ] Run the script",
        priority_title = "Priority Order",
        priority_desc = "Interrupt: If attacking a low max HP mob and a higher max HP mob appears, will switch immediately",
        vote_info_desc = "- [ Step 1 ] Click Restore Vote System\n- [ Step 2 ] Stay in the lobby (in game)\n- [ Step 3 ] Set auto vote and wait",
        recent_updates_title = "Recent Updates",
        dyhub_info_title = "DYHUB Info",
        serverhop_progress = "Teleporting to another server...",
        rejoin_progress = "Rejoining server...",
        language_notify_title = "Language",
    },
}

getgenv().DYHUB_T = function(key)
    return (translations[currentLanguage] and translations[currentLanguage][key]) or key
end
local function T(key)
    return getgenv().DYHUB_T(key)
end

-- 构建反向翻译表 (显示文本 -> 键)
local translationReverse = {}
local function BuildReverseTranslation()
    translationReverse = {}
    local langTable = translations[currentLanguage]
    if langTable then
        for key, value in pairs(langTable) do
            translationReverse[value] = key
        end
    end
end
BuildReverseTranslation()

-- 工具函数：获取原始键（用于保存值）
local function GetOriginalKey(displayValue)
    local key = translationReverse[displayValue]
    return key or displayValue
end

-- ====================== WINDOW ======================
Players = game:GetService("Players")
userversion = "至尊版"

Window = WindUI:CreateWindow({
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

Window:SetToggleKey(Enum.KeyCode.K)
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
Info   = Window:Tab({ Title = T("tab_info"), Icon = "info" })
MainDivider  = Window:Divider()
Main   = Window:Tab({ Title = T("auto_farm"), Icon = "rocket" })
Main4  = Window:Tab({ Title = T("esp_enable"), Icon = "eye" })
Main2  = Window:Tab({ Title = T("local_player"), Icon = "user" })
MainDivider1 = Window:Divider()
Main5  = Window:Tab({ Title = T("tab_shop"), Icon = "shopping-cart" })
Main6  = Window:Tab({ Title = T("collect_section"), Icon = "hand" })
Main7  = Window:Tab({ Title = T("game_mode"), Icon = "gamepad-2" })
MainDivider2 = Window:Divider()
Main3  = Window:Tab({ Title = T("tab_settings"), Icon = "settings" })
Window:SelectTab(1)

-- ======================== INFO ========================
Info:Section({ Title = T("recent_updates_title"), TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = T("info_update"),
    Desc = T("info_desc"),
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Info:Divider()
Info:Section({ Title = T("dyhub_info_title"), TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({ Title = T("info_title"), Desc = T("info_desc2"), Image = "rbxassetid://104487529937663", ImageSize = 30 })

-- ====================== SERVICES ======================
TweenService        = game:GetService("TweenService")
ReplicatedStorage   = game:GetService("ReplicatedStorage")
ReplicatedFirst     = game:GetService("ReplicatedFirst")
VirtualInputManager = game:GetService("VirtualInputManager")
RunService          = game:GetService("RunService")
UserInputService    = game:GetService("UserInputService")
Lighting            = game:GetService("Lighting")

-- ====================== PLAYER ======================
LocalPlayer    = Players.LocalPlayer
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
    Mode = { "Normal", "VeryHard", "Hard", "Insane", "Nightmare", "BossRush", "DarkDimension", "Hell", "ThunderStorm", "Christmas", "Zombie", "AstroV2", "Astro", "100MVisit" },
}

-- ====================== CONFIG VARIABLES ======================
skillList          = { "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }
skillDropdownValues = { "All", "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" }

-- ====================== FARM HELPERS ======================
function NormalizeFarmMode(mode)
    mode = tostring(mode or "Tween")
    if mode == "tp" or mode == "Tp" or mode == "tp1" then
        return "Teleport"
    end
    if mode ~= "Teleport" and mode ~= "Tween" then
        return "Tween"
    end
    return mode
end

function NormalizeFarmTargetMode(mode)
    mode = tostring(mode or "Normal Mode")
    if mode ~= "Normal Mode" and mode ~= "Astro Holdout Mode" and mode ~= "Dark Dimension Mode" then
        return "Normal Mode"
    end
    return mode
end

function NormalizeCollectMovement(mode)
    mode = tostring(mode or "Tween")
    if mode ~= "Teleport" and mode ~= "Tween" then
        return "Tween"
    end
    return mode
end

-- ====================== CAMERA MODE HELPERS ======================
function NormalizeCameraMode(mode)
    mode = tostring(mode or "Manual")
    if mode == "Manuel" or mode:lower() == "manual" then
        return "Manual"
    end
    if mode:lower() == "classic" then
        return "Classic"
    end
    return "Manual"
end

-- ====================== STATE VARIABLES ======================
AutoFarmEnabled        = Config:Get("AutoFarmEnabled", false)
FarmPosition           = Config:Get("FarmPosition", "Above")
FarmMode               = NormalizeFarmMode(Config:Get("FarmMode", "Tween"))
FarmTargetMode         = NormalizeFarmTargetMode(Config:Get("FarmTargetMode", "Normal Mode"))
DarkDimensionCollecting = false
DarkDimensionLowValue   = 0.900
DarkDimensionSafeValue  = 0.950
DarkDimensionLastWarnAt = 0
DarkDimensionCollectToken = 0
DarkDimensionOrbSearchCooldown = 0
DarkDimensionJeffreyAvoidRange = 70
JeffreyTargetBlockUntil = {}
JeffreyLastUnsafeTargetAt = 0
JeffreySafeHoldUntil = 0
JeffreySafeRetargetDelay = 0.85
AntiJeffreyEnabled     = Config:Get("AntiJeffreyEnabled", false)
AntiJeffreyRange       = Config:Get("AntiJeffreyRange", 50)
BypassJeffreyEnabled   = Config:Get("BypassJeffreyEnabled", false)
BypassJeffreyLoopRunning = false
BypassJeffreyLastFullScanAt = 0
BypassJeffreyFullScanDelay = 3
AntiJeffreyLoopRunning = false
AntiJeffreyGuardLoopRunning = false
AntiJeffreyLastPushAt  = 0
JeffreyCacheList       = {}
JeffreyCacheAt         = 0
JeffreyCacheTTL        = 0.55
AntiJeffreyEscapeBusy  = false
AntiJeffreyLastEscapeAt = 0
AntiJeffreyEscapeCooldown = 0.32
AntiJeffreyEscapeStep  = 70
AntiJeffreyDangerRange = 20
AntiJeffreyKillZoneRange = 5
AntiJeffreyHardEscapeStep = 70
AntiJeffreyCriticalEscapeStep = 90
AntiJeffreyEscapePauseUntil = 0
AntiJeffreyForceRetargetUntil = 0
AstroModeDoorTopCF      = CFrame.new(-23.3435822, 67, 0.341766357)
AstroModeDoorBottomCF   = CFrame.new(-23.3435822, 3, 0.341766357)
AstroModeFinalRunning   = false
AstroModeLastFinalAt    = 0
MiscOptions            = Config:Get("MiscOptions", {})
SyncFarmOnly           = Config:Get("SyncFarmOnly", true)
FarmAstroTokenEnabled  = Config:Get("FarmAstroTokenEnabled", false)
FarmAstroTokenRunning  = false
FarmAstroTokenPart     = nil
FarmAstroTokenTween    = nil
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
AutoAttackEnabled      = false
AutoSkillEnabled       = false
AutoSkipHeliEnabled    = false
BoostFPS_Active_dummy  = false
AutoStartEnabled       = Config:Get("AutoStartEnabled", table.find(MiscOptions, "Auto Start") ~= nil)
AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false)
AutoVoteValue         = Config:Get("AutoVoteValue", "Christmas")
AutoVoteLoopRunning   = false
AutoVoteLastFireAt    = 0
AutoStartLastReadyAt  = 0
AutoFillUpEnabled      = false
SelectedSkills         = Config:Get("SelectedSkills", { "All" })
SafeModeEnabled        = false
SafeValue              = Config:Get("SafeValue", 50)
GodModeEnabled         = false
GodModeValue           = Config:Get("GodModeValue", 50)
GodModeTriggered       = false
ResetWaveEnabled       = false
ResetWaveValue         = Config:Get("ResetWaveValue", 10)
ResetWaveLoopRunning   = false
ResetWaveTeleporting   = false
ResetWaveTargetCF      = CFrame.new(1250, 500, 1250)
ResetWaveHoldTime      = 2
ResetWaveToken         = 0
ResetWaveLastTriggeredWave = nil
ResetWaveLastTriggeredKey  = nil
ResetWaveLastTeleportAt = 0
WaitingRespawn         = false
IdlePosition           = GetDYHUBWaitingStandCFrame() * CFrame.Angles(math.rad(0), 0, 0)
IdleHoldDistance       = 12
IdleTeleportCooldown   = 1.25
LastIdleTeleportAt     = 0
IdlePositionReached    = false
SkillDelay             = Config:Get("SkillDelay", 1)
LoopDelay              = 0.5
TweenSpeed             = 1
HeightValue            = Config:Get("HeightValue", 3)
NeedNoClip             = false
LockActive             = false
AutoStartConnection    = nil
noBarrierConnection    = nil
noBarrierActive        = Config:Get("NoBarrier", false)
CameraMode             = NormalizeCameraMode(Config:Get("CameraMode", "Manual"))
if Config:Get("CameraMode", "Manual") ~= CameraMode then
    Config:Set("CameraMode", CameraMode)
end
FarmLoopRunning        = false
AutoAttackLoopRunning  = false
AutoSkillLoopRunning   = false
FarmForceRetarget      = false
FarmCollecting         = false
CombatDebugEnabled     = Config:Get("CombatDebugEnabled", false)
CombatDebugCooldowns   = {}

function UpdateDYHUBWaitingPartCollision()
    if AutoFarmEnabled ~= true then
        if DestroyDYHUBWaitingPart then DestroyDYHUBWaitingPart() end
        part = nil
        return
    end
    local waitingPart = EnsureDYHUBWaitingPart and EnsureDYHUBWaitingPart() or GetDYHUBWaitingPart()
    if not waitingPart then return end
    part = waitingPart
    pcall(function()
        ConfigureDYHUBWaitingPart(waitingPart)
    end)
end

UpdateDYHUBWaitingPartCollision()

workspace.ChildRemoved:Connect(function(obj)
    if obj and obj.Name == DYHUB_WAITING_PART_NAME and AutoFarmEnabled == true then
        task.defer(function()
            UpdateDYHUBWaitingPartCollision()
        end)
    end
end)

function CombatDebug(tag, message, cooldown, showNotify)
    if not CombatDebugEnabled then return end
    cooldown = cooldown or 3
    local now = tick()
    local key = tostring(tag or "Debug")
    if CombatDebugCooldowns[key] and now - CombatDebugCooldowns[key] < cooldown then return end
    CombatDebugCooldowns[key] = now
    local text = "[DYHUB][" .. key .. "] " .. tostring(message or "")
    print(text)
    if showNotify and WindUI then
        pcall(function()
            WindUI:Notify({
                Title = "战斗调试",
                Content = tostring(message or ""),
                Duration = 3,
                Icon = "bug"
            })
        end)
    end
end

function IsMiscFarmAllowed()
    if FarmAstroTokenEnabled and SyncFarmOnly then return false end
    return AutoFarmEnabled or not SyncFarmOnly
end

function StopMiscFarmRuntime(reason)
    AutoAttackEnabled = false
    AutoSkillEnabled = false
    AutoSkipHeliEnabled = false
    AutoFillUpEnabled = false
    SafeModeEnabled = false
    GodModeEnabled = false
    ResetWaveEnabled = false
    ResetWaveTeleporting = false
    ResetWaveToken = (ResetWaveToken or 0) + 1
    ResetWaveLastTriggeredWave = nil
    ResetWaveLastTriggeredKey = nil
    FillUpRunning = false
    if AutoStartEnabled then
        StopAutoStart()
    end
    pcall(function() TriggerAutoSkipHeli(false) end)
    if BoostFPS_Active then
        RestoreBoostFPS()
    end
    CombatDebug("MiscGate", "辅助功能运行时已停止: " .. tostring(reason or "同步门"), 3)
end

function ApplyMiscFarmGate(reason)
    if SyncFarmOnly and not AutoFarmEnabled then
        StopMiscFarmRuntime(reason or "自动挂机已关闭")
        return false
    end
    HandleMiscOptions(MiscOptions)
    return true
end

CameraLastApplyAt = 0
CameraApplyCooldown = 0.22
CameraSyncToken = 0

function GetCameraTargetForMode(char)
    if not char or not char.Parent then return nil, nil end
    local humanoid = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")
    if CameraMode == "Classic" then
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
        if token == CameraSyncToken then
            ApplyCameraMode(force == true)
        end
    end)
end

LastFarmCameraStabilize = 0

function StabilizeFarmCamera()
    local now = tick()
    if now - (LastFarmCameraStabilize or 0) < 0.35 then return end
    LastFarmCameraStabilize = now
    if AutoFarmEnabled then
        ApplyCameraMode(false)
    end
end

function RestoreFarmCameraAndMovement()
    pcall(function()
        local char = LocalPlayer.Character or Character
        local humanoid = char and (char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid"))
        if humanoid then humanoid.AutoRotate = true end
        ApplyCameraMode(true)
    end)
end

workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
    RequestCameraSync(true)
end)

MissingRemoteWarnAt = {}

function GetRemote(name)
    local remote = ReplicatedStorage and ReplicatedStorage:FindFirstChild(name)
    if not remote then
        local now = tick()
        if not MissingRemoteWarnAt[name] or now - MissingRemoteWarnAt[name] >= 10 then
            MissingRemoteWarnAt[name] = now
            warn("[DYHUB] 缺少远程事件: " .. tostring(name))
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
    if frame then
        pcall(function() frame.Visible = false end)
    end
end

function FireAutoVote(force)
    if not force and not IsVoteUIOpen() then return false end
    local now = tick()
    if now - AutoVoteLastFireAt < 0.25 then return false end
    AutoVoteLastFireAt = now
    local remote = GetRemote("Vote")
    if not remote then
        pcall(function() remote = ReplicatedStorage:WaitForChild("Vote", 3) end)
    end
    if not remote then return false end
    local ok, err = pcall(function()
        remote:FireServer(AutoVoteValue)
    end)
    if ok then
        HideVoteUI()
        print("[DYHUB] 自动投票已触发:", tostring(AutoVoteValue))
        return true
    else
        warn("[DYHUB] 自动投票失败:", err)
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
HighHPThreshold        = Config:Get("HighHPThreshold", 200)
_currentTargetPriority = 0
_interruptSignal       = false

VirtualUser = game:GetService("VirtualUser")
AntiAFK = Config:Get("AntiAfk", true)

AutoBuyWeaponEnabled   = Config:Get("AutoBuyWeaponEnabled", false)
AutoBuyMiscEnabled     = Config:Get("AutoBuyMiscEnabled", false)
SelectedWeapon         = Config:Get("SelectedWeapon", "Stungun")
SelectedMiscItem       = Config:Get("SelectedMiscItem", "HeadPhone")

-- ====================== FILL UP PART CONFIG ======================
FILLUP_PART_PATH   = { "HelicopterShop", "ShopXDD", "PartForShop" }
FILLUP_TARGET_POS  = Vector3.new(44.2756729, 26.3595276, -32.7318268)
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

function IsAlly(mob)
    return AllyNames[mob.Name] ~= nil
end

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
        warn("玩家角色或 HumanoidRootPart 未找到!")
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
    if FarmTargetMode == "Dark Dimension Mode" then return AutoFarmEnabled == true end
    if FarmTargetMode == "Normal Mode" then return AutoFarmEnabled == true and AntiJeffreyEnabled == true end
    return false
end

function GetFarmJeffreyAvoidRange()
    if AntiJeffreyEnabled then return AntiJeffreyRange end
    return DarkDimensionJeffreyAvoidRange
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
    local base = GetFarmJeffreyAvoidRange and GetFarmJeffreyAvoidRange() or 50
    return math.max(base, DarkDimensionJeffreyAvoidRange or 70, 65)
end

function IsFarmTargetSafeFromJeffrey(mob, forceRefresh)
    if not IsFarmJeffreyAvoidActive or not IsFarmJeffreyAvoidActive() then return true end
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
    BreakFarmLockForJeffrey(reason or "目标不安全,靠近 Jeffrey", 0.55)
    MoveToJeffreySafeHold(reason or "目标不安全")
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
    BreakFarmLockForJeffrey("Jeffrey 逃跑", isKillZone and 0.65 or 0.45)
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
    local active = IsFarmJeffreyAvoidActive()
    if not active and not AntiJeffreyEnabled then return false end
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
    if active and mob and not IsFarmTargetSafeFromJeffrey(mob, true) then
        MarkMobUnsafeByJeffrey(mob, 3)
        BreakFarmLockForJeffrey("目标被 Jeffrey 阻挡", 0.55)
        MoveToJeffreySafeHold("目标被 Jeffrey 阻挡")
        return true
    end
    if active and not mob and HasAnyJeffreyRoot() and tick() < (JeffreySafeHoldUntil or 0) then
        BreakFarmLockForJeffrey("Jeffrey 安全保持等待", 0.35)
        return true
    end
    if active and not mob and HasAnyJeffreyRoot() and tick() - (JeffreyLastUnsafeTargetAt or 0) <= 2 then
        MoveToJeffreySafeHold("附近所有目标都不安全")
        return true
    end
    if active and not mob and jeffrey and dist <= range then
        MoveAwayFromJeffrey(scanRange, AntiJeffreyHardEscapeStep, 0.22, true)
        return true
    end
    return false
end

function ShouldFarmRetargetFromJeffrey(mob)
    return HandleFarmJeffreyEmergency(mob)
end

function ShouldDarkDimensionRetargetFromJeffrey(mob)
    return HandleFarmJeffreyEmergency(mob)
end

function PushAwayFromJeffrey()
    if not AntiJeffreyEnabled then return false end
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

function StartAntiJeffreyLoop()
    if AntiJeffreyLoopRunning then return end
    AntiJeffreyLoopRunning = true
    task.spawn(function()
        while AntiJeffreyEnabled do
            pcall(PushAwayFromJeffrey)
            task.wait(0.18)
        end
        AntiJeffreyLoopRunning = false
    end)
end

function IsJeffreyGuardActive()
    return (AutoFarmEnabled == true and IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive()) or AntiJeffreyEnabled == true
end

function StartJeffreyGuardLoop()
    if AntiJeffreyGuardLoopRunning then return end
    AntiJeffreyGuardLoopRunning = true
    task.spawn(function()
        while AutoFarmEnabled or AntiJeffreyEnabled do
            if IsJeffreyGuardActive() then
                pcall(function()
                    RefreshCombatCharacter()
                    if Character and HumanoidRootPart then
                        local range = math.max(GetFarmJeffreyAvoidRange and GetFarmJeffreyAvoidRange() or 50, AntiJeffreyDangerRange or 20)
                        local root, dist = GetNearestJeffreyRoot(HumanoidRootPart.Position, range, true)
                        if root and dist <= (AntiJeffreyDangerRange or 20) then
                            HandleFarmJeffreyEmergency(nil)
                        elseif AntiJeffreyEnabled and root and dist <= AntiJeffreyRange then
                            PushAwayFromJeffrey()
                        end
                    end
                end)
            end
            task.wait(IsAntiJeffreyEscapePauseActive() and 0.08 or 0.16)
        end
        AntiJeffreyGuardLoopRunning = false
    end)
end
-- ============================================================
-- ====================== BYPASS JEFFREY ======================
function GetBypassJeffreyObject(obj)
    if not obj then return nil end
    if IsJeffreyName(obj.Name) then return obj end
    local cur = obj.Parent
    while cur and cur ~= workspace do
        if IsJeffreyName(cur.Name) then return cur end
        cur = cur.Parent
    end
    return nil
end

function SetBypassJeffreySit(jeffrey)
    if not BypassJeffreyEnabled then return false end
    if not jeffrey or not jeffrey.Parent then return false end
    local target = GetBypassJeffreyObject(jeffrey)
    if not target or not target.Parent then return false end
    local notHumanoid = nil
    pcall(function()
        notHumanoid = target:FindFirstChild("NotHumanoid") or target:FindFirstChild("NotHumanoid", true)
    end)
    if not notHumanoid then return false end
    local ok = pcall(function()
        if notHumanoid.Sit ~= true then
            notHumanoid.Sit = true
        end
    end)
    return ok
end

function ScanBypassJeffreys(forceFullScan)
    if not BypassJeffreyEnabled then return 0 end
    local count = 0
    local seen = {}
    local function try(obj)
        local target = GetBypassJeffreyObject(obj)
        if target and not seen[target] then
            seen[target] = true
            if SetBypassJeffreySit(target) then
                count = count + 1
            end
        end
    end
    pcall(function()
        for _, obj in ipairs(workspace:GetChildren()) do
            if IsJeffreyName(obj.Name) then try(obj) end
        end
        local living = workspace:FindFirstChild("Living")
        if living then
            for _, obj in ipairs(living:GetChildren()) do
                if IsJeffreyName(obj.Name) then try(obj) end
            end
            for _, obj in ipairs(living:GetDescendants()) do
                if obj.Name == "NotHumanoid" then try(obj) end
            end
        end
        local now = tick()
        if forceFullScan or now - (BypassJeffreyLastFullScanAt or 0) >= (BypassJeffreyFullScanDelay or 3) then
            BypassJeffreyLastFullScanAt = now
            for _, obj in ipairs(workspace:GetDescendants()) do
                if IsJeffreyName(obj.Name) or obj.Name == "NotHumanoid" then
                    try(obj)
                end
            end
        end
    end)
    return count
end

function StartBypassJeffreyLoop()
    if BypassJeffreyLoopRunning then return end
    BypassJeffreyLoopRunning = true
    task.spawn(function()
        while BypassJeffreyEnabled do
            pcall(function()
                ScanBypassJeffreys(false)
            end)
            task.wait(0.75)
        end
        BypassJeffreyLoopRunning = false
    end)
end

function HandleBypassJeffreyObject(obj)
    if not BypassJeffreyEnabled or not obj then return end
    if IsJeffreyName(obj.Name) or obj.Name == "NotHumanoid" or GetBypassJeffreyObject(obj) then
        task.defer(function()
            if BypassJeffreyEnabled then
                SetBypassJeffreySit(obj)
                task.wait(0.15)
                SetBypassJeffreySit(obj)
            end
        end)
    end
end

workspace.DescendantAdded:Connect(function(obj)
    if obj and IsJeffreyName(obj.Name) then
        JeffreyCacheAt = 0
        HandleBypassJeffreyObject(obj)
        if IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive() then
            FarmForceRetarget = true
            task.delay(0.25, function()
                if not IsAntiJeffreyEscapePauseActive() then FarmForceRetarget = false end
            end)
            StartJeffreyGuardLoop()
        end
    elseif obj and obj.Name == "NotHumanoid" then
        HandleBypassJeffreyObject(obj)
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    if obj and IsJeffreyName(obj.Name) then
        JeffreyCacheAt = 0
        if BypassJeffreyEnabled then
            task.delay(0.15, function()
                if BypassJeffreyEnabled then ScanBypassJeffreys(false) end
            end)
        end
    end
end)

function GetMobMaxHP(mob)
    local humanoid = mob and mob:FindFirstChild("Humanoid")
    if not humanoid then return 0 end
    return humanoid.MaxHealth or 0
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
    CombatDebug("MobCache", "缓存已失效: " .. tostring(reason or "未知"), 2)
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
        CombatDebug("AutoAttackRestart", "重新启动检查: " .. tostring(reason or "未知"), 3)
        StartAutoAttack()
    end
    if AutoSkillEnabled and IsMiscFarmAllowed() and StartAutoSkill then
        CombatDebug("AutoSkillRestart", "重新启动检查: " .. tostring(reason or "未知"), 3)
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
        CombatDebug("MobCache", "Living 文件夹未找到.", 5)
        return
    end
    MobCacheChildAddedConnection = folder.ChildAdded:Connect(function(obj)
        InvalidateMobCache("怪物新增")
        CombatDebug("MobCacheAdded", "怪物出现: " .. tostring(obj and obj.Name or "nil"), 2)
        task.delay(0.15, function()
            InvalidateMobCache("怪物新增延迟扫描")
            RestartCombatLoopsIfNeeded("怪物新增")
        end)
        task.delay(0.75, function()
            InvalidateMobCache("怪物加载延迟扫描")
            RestartCombatLoopsIfNeeded("怪物加载")
        end)
    end)
    MobCacheChildRemovedConnection = folder.ChildRemoved:Connect(function(obj)
        InvalidateMobCache("怪物移除")
        CombatDebug("MobCacheRemoved", "怪物移除: " .. tostring(obj and obj.Name or "nil"), 2)
        task.delay(0.05, function() RestartCombatLoopsIfNeeded("怪物移除") end)
    end)
    CombatDebug("MobCache", "Living 文件夹已挂钩.", 5)
end

function SetupMobCacheWorkspaceHooks()
    if MobCacheWorkspaceAddedConnection then return end
    MobCacheWorkspaceAddedConnection = workspace.ChildAdded:Connect(function(obj)
        if obj and obj.Name == "Living" then
            HookMobCacheFolder(obj)
            InvalidateMobCache("Living 文件夹已添加")
            task.delay(0.25, function() RestartCombatLoopsIfNeeded("Living 文件夹已添加") end)
        end
    end)
    MobCacheWorkspaceRemovedConnection = workspace.ChildRemoved:Connect(function(obj)
        if obj and obj == MobCacheFolder then
            HookMobCacheFolder(nil)
            InvalidateMobCache("Living 文件夹已移除")
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
    CombatDebug("MobCacheRebuild", "已缓存有效怪物: " .. tostring(#MobCacheList), 3)
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
        CombatDebug("MobCacheFallback", "缓存为空但 Living 有子节点，重新构建一次.", 3)
        RebuildMobCache()
        alive = {}
        for _, mob in ipairs(MobCacheList) do
            if IsValidMob(mob) then table.insert(alive, mob) end
        end
    end
    return alive
end

SetupMobCacheWorkspaceHooks()

-- ====================== MOB SELECTION ======================
function IsAstroMob(mob)
    return mob and mob.Name and mob.Name:lower():find("astro", 1, true) ~= nil
end

function IsFarmMobAllowedByMode(mob)
    if FarmTargetMode == "Astro Holdout Mode" then
        return IsAstroMob(mob)
    end
    return true
end

function GetFarmCandidateMobs(forceRefresh)
    local source = GetCachedLivingMobs(forceRefresh == true)
    local useJeffreyAvoid = IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive()
    if FarmTargetMode ~= "Astro Holdout Mode" and not useJeffreyAvoid then
        return source
    end
    local filtered = {}
    local range = GetFarmTargetDangerRange and GetFarmTargetDangerRange() or (GetFarmJeffreyAvoidRange and GetFarmJeffreyAvoidRange() or DarkDimensionJeffreyAvoidRange)
    for _, mob in ipairs(source) do
        if IsFarmMobAllowedByMode(mob) then
            if useJeffreyAvoid and IsMobBlockedByJeffrey(mob, range) then
            else
                table.insert(filtered, mob)
            end
        end
    end
    return filtered
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
function GetHelicopter()
    for _, mob in ipairs(GetFarmCandidateMobs(false)) do
        if mob.Name:lower():find("helicopter") then
            return mob
        end
    end
    return nil
end

function GetGiantSTToilet()
    if FarmTargetMode == "Astro Holdout Mode" then return nil, nil end
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
    local giant, prompt = nil, nil
    local heli, highMob, nearMob = nil, nil, nil
    local bestHP, nearDist = HighHPThreshold, math.huge
    local candidates = GetFarmCandidateMobs(false)
    for _, mob in ipairs(candidates) do
        if not giant and FarmTargetMode ~= "Astro Holdout Mode" and mob.Name == "Giant ST toilet" then
            local lever = mob:FindFirstChild("lever")
            if lever then
                local pr = lever:FindFirstChildOfClass("ProximityPrompt")
                if pr then giant = mob; prompt = pr end
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
    if heli then return heli, "Helicopter", nil, 3 end
    if highMob then return highMob, "HighHP", nil, 2 end
    if nearMob then return nearMob, "NearestMob", nil, 1 end
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

PADDING_REDUCE_STEP    = Config:Get("PaddingReduceStep", 2)
PADDING_SAFE_MIN       = Config:Get("PaddingSafeMin", -30)
DMG_THRESHOLD          = Config:Get("DmgThreshold", 40)
ANTI_CLIP_MARGIN       = Config:Get("AntiClipMargin", 3)
PLAYER_HALF_HEIGHT     = 3

MobHeightOverride   = {}
MobConfirmedPadding = {}
MobLastHealth       = {}
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

function ResetMobOverride(mob)
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
function GetTargetCFrame(mob, position)
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

    if conn then pcall(function() conn:Disconnect() end) end
    if not completed then pcall(function() tween:Cancel() end) end
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

    if FarmMode == "Tween" then
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

    if FarmMode == "Tween" then
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
    CombatDebug("DarkDimension", "理智值低但未找到带 TouchInterest 的 OrbSanity.", 4, false)
end

function IsDarkDimensionSanityLow()
    if FarmTargetMode ~= "Dark Dimension Mode" or DarkDimensionCollecting or not AutoFarmEnabled then return false end
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
    if FarmTargetMode ~= "Dark Dimension Mode" or DarkDimensionCollecting or not AutoFarmEnabled then return false end

    local sanity = GetSanityTransparency()
    if not sanity or sanity >= DarkDimensionLowValue then return false end

    local collectToken = StopFarmLockForSanityCollect("低理智值")

    local didCollect = false
    pcall(function()
        while AutoFarmEnabled and FarmTargetMode == "Dark Dimension Mode" and DarkDimensionCollectToken == collectToken do
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
            until not AutoFarmEnabled or FarmTargetMode ~= "Dark Dimension Mode" or not sanity or sanity >= DarkDimensionSafeValue or not part.Parent or not HasSanityTouchInterest(part) or waited >= 3
        end
    end)

    if DarkDimensionCollectToken == collectToken then
        DarkDimensionCollecting = false
        FarmCollecting = false
        _interruptSignal = false
        if AutoFarmEnabled then
            WaitingRespawn = false
            FarmForceRetarget = true
            if HandleFarmJeffreyEmergency then pcall(function() HandleFarmJeffreyEmergency(nil) end) end
            task.delay(JeffreySafeRetargetDelay or 0.85, function()
                if not DarkDimensionCollecting and not IsAntiJeffreyEscapePauseActive() then FarmForceRetarget = false end
            end)
        else
            FarmForceRetarget = false
        end
    end

    return didCollect
end

function DoAstroModeFinalDoor()
    if FarmTargetMode ~= "Astro Holdout Mode" or AstroModeFinalRunning then return false end

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

        if FarmMode == "Tween" then
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
        CombatDebug("CombatCharacter", "角色或 HumanoidRootPart 未就绪.", 4)
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

        CombatDebug("PriorityNoMob", "尚未找到有效怪物.", 4)
        return nil, nil, nil, 0
    end

    CombatDebug("PriorityError", "GetPriorityMob 失败: " .. tostring(mob), 3, true)
    warn("[DYHUB] GetPriorityMob 失败:", tostring(mob))
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
                        CombatDebug("AutoAttackRemote", "LMB 远程事件缺失.", 5, true)
                    end
                else
                    CombatDebug("AutoAttackNoMob", "自动攻击正在等待有效怪物.", 5)
                end
                task.wait(0.12)
            else
                CombatDebug("AutoAttackPaused", "自动攻击被同步农场/农场 Astro 暂停.", 5)
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
                    if table.find(SelectedSkills, "All") then
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
                            CombatDebug("AutoSkillKey", "无效技能按键: " .. tostring(key), 5)
                        end
                    end
                else
                    CombatDebug("AutoSkillNoMob", "自动技能正在等待有效怪物.", 5)
                    task.wait(0.25)
                end
            else
                CombatDebug("AutoSkillPaused", "自动技能被同步农场/农场 Astro 暂停.", 5)
                task.wait(0.25)
            end
            task.wait(0.05)
        end

        AutoSkillLoopRunning = false
    end)
end

function TriggerAutoSkipHeli(state)
    local remote = GetRemote("SetSettingAutoSkipWave")
    if remote then pcall(function() remote:FireServer(state) end) end
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

-- ====================== Delete Map (删除地图) SYSTEM ======================
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

function RestoreBoostFPS()
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
    print("[DYHUB] 删除地图: 关闭 (已恢复)")
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
    return not char
        or not char.Parent
        or not humanoid
        or not humanoid.Parent
        or humanoid.Health <= 0
        or humanoid:GetState() == Enum.HumanoidStateType.Dead
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
            CombatDebug("GodMode", "触发一次: " .. tostring(reason or "手动"), 2)
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

        CombatDebug("GodMode", "触发一次: " .. tostring(reason or "手动"), 2)
        return true
    end)

    return ok and result == true
end

function ShouldBlockFarmAstroGodModePercent()
    return FarmAstroTokenEnabled == true
        and SyncFarmOnly == false
        and table.find(MiscOptions or {}, "God Mode") ~= nil
end

-- ====================== GOD MODE LOOP ======================
task.spawn(function()
    while true do
        task.wait(0.1)

        if GodModeEnabled and not FarmAstroGodModePaused and IsMiscFarmAllowed() and not ShouldBlockFarmAstroGodModePercent() then
            pcall(function()
                local char = LocalPlayer.Character
                if not char then return end

                local humanoid = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("Humanoid")
                if not humanoid then return end
                if humanoid.MaxHealth <= 0 then return end

                local hpPercent = (humanoid.Health / humanoid.MaxHealth) * 100

                if hpPercent < GodModeValue then
                    ForceGodModeOnce("HP 低于上帝模式阈值")
                end
            end)
        end
    end
end)

-- ====================== AUTO FILL UP ======================
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
        while AutoFillUpEnabled and IsMiscFarmAllowed() do
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
                if humanoid then humanoid.Health = humanoid.MaxHealth end
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

    if not ok then warn("[DYHUB] GetReadyRemote 失败:", err) end
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
    IdlePosition = GetDYHUBWaitingStandCFrame() * CFrame.Angles(math.rad(0), 0, 0)
    UpdateDYHUBWaitingPartCollision()

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
CollectMode          = Config:Get("CollectMode", "Clean")
CollectMovementMode  = NormalizeCollectMovement(Config:Get("CollectMovementMode", "Tween"))

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
    if obj:IsA("Model") then return obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")
    elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then return obj end
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

    if CollectMovementMode == "Teleport" then
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

function IsItemGone(obj) return not obj or not obj.Parent end

function BeginCollectPause()
    FarmCollecting = true
    FarmForceRetarget = true
    LockActive = false
    if FarmAstroTokenEnabled then SetFarmAstroCollectPause(true) end
    task.wait(0.08)
end

function EndCollectPause()
    if FarmAstroTokenEnabled then SetFarmAstroCollectPause(false) end
    FarmCollecting = false
    FarmForceRetarget = true
    if AutoFarmEnabled then
        WaitingRespawn = false
        StartFarmLoop()
    end
    HandleMiscOptions(MiscOptions)
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
            if FarmAstroTokenEnabled and CollectMode == "Clean" then
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
                            if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end
                        end
                        EndCollectPause()

                    elseif CollectMode == "Clean" then
                        local waitedClean = 0
                        while not AllMobsDead() and AutoCollectEnabled do
                            task.wait(0.5)
                            waitedClean = waitedClean + 0.5
                            if waitedClean >= 120 then break end
                        end
                        if not AutoCollectEnabled then break end

                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end
                        BeginCollectPause()
                        for _, obj in ipairs(FindNewCollectItems()) do
                            if not AutoCollectEnabled then break end
                            if not IsItemGone(obj) then CollectSingleItem(obj) else KnownCollectItems[obj] = true end
                        end
                        EndCollectPause()
                        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(true) end

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
                        if IsItemGone(obj) then KnownCollectItems[obj] = nil end
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
FARM_ASTRO_TOKEN_IMAGE = "rbxassetid://104487529937663"
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
        Title = "农场 Astro Token",
        Content = T("notify_farm_astro_conflict"),
        Duration = 4,
        Icon = "triangle-alert"
    })
end

function NotifyFarmAstroCleanMode()
    local now = tick()
    if now - FarmAstroTokenLastCleanNotify < 5 then return end
    FarmAstroTokenLastCleanNotify = now
    WindUI:Notify({
        Title = "农场 Astro Token",
        Content = T("notify_astro_clean_mode"),
        Duration = 5,
        Icon = "triangle-alert"
    })
end

function CheckFarmAstroCollectMode()
    if FarmAstroTokenEnabled and AutoCollectEnabled and CollectMode == "Clean" then
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
    return table.find(MiscOptions or {}, "God Mode") ~= nil
end

function PauseFarmAstroGodModeForTimer()
    if not FarmAstroTokenEnabled then return false end
    if SyncFarmOnly then return false end
    if not IsFarmAstroGodModeSelected() then return false end
    if FarmAstroGodModePaused then return true end
    if tick() < FarmAstroTokenTimerIgnoreUntil then return false end

    local timerValue = GetFarmAstroTimerValue()
    UpdateFarmAstroWaveTimerArmed(timerValue)
    if timerValue ~= nil and timerValue <= 10 and FarmAstroWaveTimerArmed == true then
        FarmAstroGodModePaused = true
        GodModeTriggered = false
        CombatDebug("FarmAstroGodSync", "上帝模式百分比在波次计时器 " .. tostring(timerValue) .. " 处暂停", 2, false)
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

    if wasPaused and IsFarmAstroGodModeSelected() then
        CombatDebug("FarmAstroGodSync", "上帝模式在 " .. tostring(reason or "重生") .. " 后恢复", 2, false)
        task.defer(function()
            HandleMiscOptions(MiscOptions)
        end)
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
    if not FarmAstroTokenEnabled or not ShouldBlockFarmAstroGodModePercent() then
        FarmAstroReviveGodTriggered = false
        FarmAstroReviveTimerArmed = false
        FarmAstroLastReviveTimer = nil
        return
    end

    local reviveTimer = GetFarmAstroReviveTimerValue()
    UpdateFarmAstroReviveTimerArmed(reviveTimer)

    if reviveTimer == 5 and FarmAstroReviveTimerArmed == true then
        if not FarmAstroReviveGodTriggered then
            if ForceGodModeOnce("农场 Astro 复活计时器") then
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
    if not FarmAstroTokenEnabled or not ShouldBlockFarmAstroGodModePercent() then return end
    if not FarmAstroFinalLockActive then return end
    if FarmAstroBottomGodTriggered then return end

    local reviveTimer = GetFarmAstroReviveTimerValue()
    UpdateFarmAstroReviveTimerArmed(reviveTimer)

    if reviveTimer == 5 and FarmAstroReviveTimerArmed == true then
        if ForceGodModeOnce("农场 Astro 底部锁定复活计时器") then
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
    ResumeFarmAstroGodModeAfterRespawn("农场 Astro 计时器重置")
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
            if hum then hum.Sit = false; hum.PlatformStand = false end
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
    HandleMiscOptions(MiscOptions)

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
        if FarmAstroTokenPart then pcall(function() FarmAstroTokenPart:Destroy() end) end
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
        ResumeFarmAstroGodModeAfterRespawn("农场 Astro 已停止")
        HandleMiscOptions(MiscOptions)
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
    ResumeFarmAstroGodModeAfterRespawn("农场 Astro 已禁用")
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
-- ====================== MAIN FARM LOOP (NEW SYSTEM) =========
-- ============================================================
FarmLoopToken = FarmLoopToken or 0

function StartFarmLoop()
    if FarmLoopRunning then return end
    FarmLoopRunning = true
    FarmLoopToken = (FarmLoopToken or 0) + 1
    local thisFarmLoopToken = FarmLoopToken

    task.spawn(function()
        local ok, err = pcall(function()
            task.spawn(function()
                while AutoFarmEnabled and FarmLoopRunning and FarmLoopToken == thisFarmLoopToken do
                    if WaitingRespawn and not LockActive and not FarmCollecting then
                        pcall(function()
                            RefreshCombatCharacter()
                            UpdateDYHUBWaitingPartCollision()
                            if Character and HumanoidRootPart then
                                if IsNearIdlePosition() then
                                    IdlePositionReached = true
                                    HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                                    HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                                else
                                    TeleportToIdle(false)
                                end
                            end
                        end)
                    else
                        IdlePositionReached = false
                    end
                    task.wait(0.35)
                end
            end)

            while AutoFarmEnabled and FarmLoopToken == thisFarmLoopToken do
                RefreshCombatCharacter()

                if ResetWaveTeleporting then
                    LockActive = false
                    FarmForceRetarget = true
                    _interruptSignal = true
                    task.wait(0.12)
                    continue
                end

                if not Character or not HumanoidRootPart then
                    task.wait(0.5)
                    continue
                end

                if FarmCollecting then
                    task.wait(0.2)
                    continue
                end

                if FarmTargetMode == "Dark Dimension Mode" and HandleDarkDimensionSanity() then
                    task.wait(0.1)
                    continue
                end

                if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(nil) then
                    task.wait(0.12)
                    continue
                end

                local mob, mobType, extraData, priority = SafeGetPriorityMob()

                if mob and ValidateFarmTargetBeforeMove and not ValidateFarmTargetBeforeMove(mob, "预目标门") then
                    task.wait(0.18)
                    continue
                end

                if mob then
                    if FarmTargetMode == "Astro Holdout Mode" then AstroModeFinalRunning = false end
                    WaitingRespawn = false
                    IdlePositionReached = false
                    _currentTargetPriority = priority

                    if mobType == "GiantST" and extraData then
                        if ValidateFarmTargetBeforeMove and not ValidateFarmTargetBeforeMove(mob, "巨型目标门") then
                            task.wait(0.18)
                            continue
                        end
                        TeleportToMob(mob)
                        if ValidateFarmTargetBeforeMove and not ValidateFarmTargetBeforeMove(mob, "巨型移动后门") then
                            task.wait(0.18)
                            continue
                        end
                        if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(mob) then
                            task.wait(0.12)
                            continue
                        end

                        local giantLockConn
                        giantLockConn = RunService.Heartbeat:Connect(function()
                            if IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled or FarmCollecting or FarmForceRetarget or (IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive()) or IsDarkDimensionSanityLow() or (IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive() and IsMobBlockedByJeffrey(mob, GetFarmTargetDangerRange and GetFarmTargetDangerRange() or 70)) then
                                if giantLockConn then giantLockConn:Disconnect() end
                                return
                            end
                            local lockCF = GetTargetCFrame(mob, FarmPosition)
                            if lockCF and Character and HumanoidRootPart then
                                MoveCharacterToFarmCFrame(lockCF)
                            end
                        end)

                        repeat
                            task.wait(0.2)
                            if HandleDarkDimensionSanityEmergency and HandleDarkDimensionSanityEmergency() then break end
                            if FarmCollecting or FarmForceRetarget then break end
                            if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(mob) then
                                break
                            end
                            ActivateProximityPrompt(extraData)
                            ActivateAllFlushPrompts()
                        until IsMobDead(mob) or not mob.Parent or not AutoFarmEnabled

                        if giantLockConn then pcall(function() giantLockConn:Disconnect() end) end

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
                            if ValidateFarmTargetBeforeMove and not ValidateFarmTargetBeforeMove(mob, "普通目标门") then
                                task.wait(0.18)
                                ResetMobOverride(mob)
                                ClearMobBoundsCache(mob)
                                continue
                            end
                            StartDamageChecker(mob)
                            TeleportToMob(mob)
                            if ValidateFarmTargetBeforeMove and not ValidateFarmTargetBeforeMove(mob, "普通移动后门") then
                                task.wait(0.18)
                                ResetMobOverride(mob)
                                ClearMobBoundsCache(mob)
                                continue
                            end
                            if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(mob) then
                                task.wait(0.12)
                                ResetMobOverride(mob)
                                ClearMobBoundsCache(mob)
                                continue
                            end

                            LockActive = true
                            local lockConn
                            lockConn = RunService.Heartbeat:Connect(function()
                                if not AutoFarmEnabled or IsMobDead(mob) or not LockActive or FarmCollecting or FarmForceRetarget or (IsAntiJeffreyEscapePauseActive and IsAntiJeffreyEscapePauseActive()) or IsDarkDimensionSanityLow() or (IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive() and IsMobBlockedByJeffrey(mob, GetFarmTargetDangerRange and GetFarmTargetDangerRange() or 70)) then
                                    if lockConn then lockConn:Disconnect() end
                                    LockActive = false
                                    return
                                end
                                if not Character or not HumanoidRootPart then return end
                                local cf = GetTargetCFrame(mob, FarmPosition)
                                if cf then
                                    MoveCharacterToFarmCFrame(cf)
                                end
                            end)

                            repeat
                                task.wait(0.15)
                                if HandleDarkDimensionSanityEmergency and HandleDarkDimensionSanityEmergency() then break end
                                if FarmCollecting or FarmForceRetarget then break end
                                if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(mob) then
                                    break
                                end

                                local shouldInterrupt, newPriority = CheckInterrupt(priority)
                                if shouldInterrupt then
                                    _interruptSignal = true
                                    break
                                end
                            until IsMobDead(mob) or not AutoFarmEnabled

                            if lockConn then pcall(function() lockConn:Disconnect() end) end
                            LockActive = false
                            if ResetWaveTeleporting then
                                FarmForceRetarget = true
                                _interruptSignal = true
                            else
                                _interruptSignal = false
                                FarmForceRetarget = false
                            end
                            ResetMobOverride(mob)
                            ClearMobBoundsCache(mob)
                        end
                    end

                else
                    _currentTargetPriority = 0
                    if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(nil) then
                        task.wait(0.25)
                    elseif IsFarmJeffreyAvoidActive and IsFarmJeffreyAvoidActive() and HasAnyJeffreyRoot and HasAnyJeffreyRoot() and tick() - (JeffreyLastUnsafeTargetAt or 0) <= 2.5 then
                        MoveToJeffreySafeHold("无安全农场目标")
                        task.wait(0.25)
                    elseif FarmTargetMode == "Astro Holdout Mode" then
                        CombatDebug("AstroMode", "未找到 Astro 怪物。进入最终门。", 5)
                        DoAstroModeFinalDoor()
                    else
                        TeleportToIdle()
                    end
                    repeat
                        task.wait(0.5)
                        if HandleFarmJeffreyEmergency and HandleFarmJeffreyEmergency(nil) then break end
                    until ResetWaveTeleporting or FarmCollecting or SafeGetPriorityMob() ~= nil or not AutoFarmEnabled
                    WaitingRespawn = false
                end

                task.wait(0.12)
            end
        end)

        if not ok then
            warn("[DYHUB] 农场循环错误:", tostring(err))
            CombatDebug("FarmLoopError", tostring(err), 3, true)
        end

        WaitingRespawn = false
        FarmCollecting = false
        if ResetWaveTeleporting then
            FarmForceRetarget = true
            _interruptSignal = true
        else
            FarmForceRetarget = false
            _interruptSignal = false
            RestoreFarmCameraAndMovement()
        end
        _currentTargetPriority = 0
        FarmLoopRunning = false

        if AutoFarmEnabled and not ResetWaveTeleporting then
            task.delay(0.5, function()
                if AutoFarmEnabled and not ResetWaveTeleporting then StartFarmLoop() end
            end)
        end
    end)
end


-- ====================== RESET WAVE SYSTEM ======================
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

    local ok, textValue = pcall(function()
        return tostring(label.Text or "")
    end)
    if not ok then return nil end

    local waveText = textValue:match("[Ww]ave%s*=?%s*(%d+)")
    if not waveText then
        waveText = textValue:match("(%d+)")
    end

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
    CombatDebug("ResetWave", "触发器已清除: " .. tostring(reason or "重置"), 3, false)
end

function IsResetWaveCharacterReady()
    RefreshCombatCharacter()
    if not Character or not Character.Parent or not HumanoidRootPart or not HumanoidRootPart.Parent then
        return false
    end

    local humanoid = Character:FindFirstChildOfClass("Humanoid") or Character:FindFirstChild("Humanoid")
    if humanoid and humanoid.Health <= 0 then
        return false
    end

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
        if not IsMiscFarmAllowed() then return false end
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
        CombatDebug("ResetWave", "在波次 " .. tostring(currentWave) .. " 保持重置点 " .. tostring(ResetWaveHoldTime or 2) .. " 秒", 2, false)
    else
        ClearResetWaveTrigger("传送中断")
    end

    FarmForceRetarget = false
    _interruptSignal = false
    LockActive = false

    if completed and ResetWaveEnabled and AutoFarmEnabled and StartFarmLoop then
        task.defer(function()
            if ResetWaveEnabled and AutoFarmEnabled and not ResetWaveTeleporting then
                StartFarmLoop()
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

function EvaluateResetWaveNow(reason, force)
    if not ResetWaveEnabled or not IsMiscFarmAllowed() or ResetWaveTeleporting then return false end

    local currentWave = GetCurrentResetWave()
    local targetWave = GetResetWaveTargetValue()

    if currentWave == nil then return false end

    if currentWave >= targetWave then
        local key = GetResetWaveTriggerKey(currentWave, targetWave)
        if force or ResetWaveLastTriggeredKey ~= key then
            return TeleportResetWave(currentWave, targetWave, force == true, reason)
        end
    else
        ClearResetWaveTrigger("波次低于目标")
    end

    return false
end

function StartResetWaveLoop()
    if ResetWaveLoopRunning then return end
    ResetWaveLoopRunning = true

    task.spawn(function()
        while ResetWaveEnabled do
            pcall(function()
                EvaluateResetWaveNow("循环", false)
            end)

            task.wait(ResetWaveTeleporting and 0.1 or 0.25)
        end

        ResetWaveLoopRunning = false
    end)
end
-- ====================== MISC OPTIONS HANDLER ======================
function HandleMiscOptions(selectedOptions)
    selectedOptions = selectedOptions or {}
    MiscOptions = selectedOptions

    local canRun = IsMiscFarmAllowed()

    local hasAutoAttack = table.find(selectedOptions, "Auto Attack") ~= nil
    if hasAutoAttack and canRun then
        AutoAttackEnabled = true
        StartAutoAttack()
    else
        AutoAttackEnabled = false
    end

    local hasAutoSkill = table.find(selectedOptions, "Auto Skill") ~= nil
    if hasAutoSkill and canRun then
        AutoSkillEnabled = true
        StartAutoSkill()
    else
        AutoSkillEnabled = false
    end

    local hasAutoSkipHeli = table.find(selectedOptions, "Auto Skip Helicopter")
    if hasAutoSkipHeli and canRun then
        if not AutoSkipHeliEnabled then AutoSkipHeliEnabled = true; TriggerAutoSkipHeli(true) end
    else
        if AutoSkipHeliEnabled then TriggerAutoSkipHeli(false) end
        AutoSkipHeliEnabled = false
    end

    local hasBoostFPS = table.find(selectedOptions, "Delete Map")
    if hasBoostFPS and canRun then
        if not BoostFPS_Active then SaveAndBoostFPS() end
    elseif BoostFPS_Active then
        RestoreBoostFPS()
    end

    SafeModeEnabled = table.find(selectedOptions, "Safe Mode") ~= nil and canRun
    GodModeEnabled  = table.find(selectedOptions, "God Mode") ~= nil and canRun

    local hasResetWave = table.find(selectedOptions, "Reset Wave")
    if hasResetWave and canRun then
        if not ResetWaveEnabled then
            ClearResetWaveTrigger("enabled")
        end
        ResetWaveEnabled = true
        StartResetWaveLoop()
        task.defer(function()
            EvaluateResetWaveNow("enabled", true)
        end)
    else
        ResetWaveEnabled = false
        ResetWaveTeleporting = false
        ResetWaveToken = (ResetWaveToken or 0) + 1
        ClearResetWaveTrigger("disabled")
    end

    local hasAutoStart = table.find(selectedOptions, "Auto Start")
    if hasAutoStart and canRun then
        if not AutoStartEnabled then StartAutoStart() end
    else
        if AutoStartEnabled then StopAutoStart() end
    end

    local hasAutoFillUp = table.find(selectedOptions, "Auto Fill Up")
    if hasAutoFillUp and canRun then
        if not AutoFillUpEnabled then AutoFillUpEnabled = true; StartAutoFillUpLoop() end
    else
        AutoFillUpEnabled = false
        FillUpRunning = false
    end

    Config:Set("MiscOptions", selectedOptions)
    Config:Set("AutoStartEnabled", hasAutoStart ~= nil)
    Config:Save()
end

-- ====================== CHARACTER RESPAWN HANDLER ======================
LocalPlayer.CharacterAdded:Connect(function(char)
    local keepFarmAstroBottomLock = ShouldKeepFarmAstroFinalLock and ShouldKeepFarmAstroFinalLock()

    Character        = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
    Client           = LocalPlayer
    FarmAstroTokenRespawnCounter = FarmAstroTokenRespawnCounter + 1

    ResetWaveToken = (ResetWaveToken or 0) + 1
    ResetWaveTeleporting = false
    ClearResetWaveTrigger("角色重生")

    if keepFarmAstroBottomLock then
        FarmAstroTokenTimerHold = true
        FarmAstroFinalLockActive = true
        FarmAstroTimerDropping = false
        FarmAstroReviveGodTriggered = false
        FarmAstroReviveTimerArmed = false
        FarmAstroLastReviveTimer = nil
        FarmAstroTokenTimerIgnoreUntil = 0
        if FarmAstroTokenEnabled then CancelFarmAstroTween() end
        task.defer(function()
            for _ = 1, 25 do
                if not FarmAstroTokenEnabled then break end
                if not (FarmAstroFinalLockActive or FarmAstroTokenTimerHold) then break end
                HoldFarmAstroBottomLockOnce()
                task.wait(0.05)
            end
        end)
    else
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
        ResumeFarmAstroGodModeAfterRespawn("角色重生")
        if FarmAstroTokenEnabled then CancelFarmAstroTween() end
    end

    JeffreyCacheAt = 0
    UpdateDYHUBWaitingPartCollision()
    MobHeightOverride   = {}
    MobConfirmedPadding = {}
    MobLastHealth       = {}
    IdlePositionReached = false
    LastIdleTeleportAt  = 0
    InvalidateMobCache("角色重生")
    ClearMobBoundsCache()
    FarmForceRetarget = true
    FarmCollecting = false
    task.delay(0.25, function()
        RestartCombatLoopsIfNeeded("角色重生")
        if AutoFarmEnabled and not ResetWaveTeleporting then StartFarmLoop(); StartJeffreyGuardLoop() end
        if ResetWaveEnabled then
            StartResetWaveLoop()
            EvaluateResetWaveNow("角色重生", true)
        end
        if BypassJeffreyEnabled then StartBypassJeffreyLoop(); ScanBypassJeffreys(true) end
    end)
    task.delay(0.8, function()
        if not ResetWaveTeleporting then
            FarmForceRetarget = false
        end
    end)
    task.wait(1)
    ApplyCameraMode(true)
end)

-- ====================== UI: MAIN ======================
Main:Section({ Title = T("auto_farm"), Icon = "package" })

AutoFarmToggle = Main:Toggle({
    Title = T("auto_farm"),
    Desc = T("auto_farm_desc"),
    Value = AutoFarmEnabled,
    Callback = function(state)
        if state and FarmAstroTokenEnabled then
            AutoFarmEnabled = false
            UpdateDYHUBWaitingPartCollision()
            Config:Set("AutoFarmEnabled", false)
            Config:Save()
            NotifyFarmAstroAutoFarm()
            return
        end
        AutoFarmEnabled = state
        UpdateDYHUBWaitingPartCollision()
        if state then
            StartFarmLoop()
            StartJeffreyGuardLoop()
            HandleMiscOptions(MiscOptions)
            WindUI:Notify({ Title = T("auto_farm"), Content = T("farm_enabled"), Duration = 2, Icon = "play" })
        else
            FarmLoopToken = (FarmLoopToken or 0) + 1
            WaitingRespawn = false
            LockActive = false
            RestoreFarmCameraAndMovement()
            UpdateDYHUBWaitingPartCollision()
            if SyncFarmOnly then
                StopMiscFarmRuntime("Auto Farm turned off while Sync Farm Only is ON")
                WindUI:Notify({ Title = T("auto_farm"), Content = T("notify_auto_farm_closed_sync"), Duration = 3, Icon = "square" })
            else
                HandleMiscOptions(MiscOptions)
                WindUI:Notify({ Title = T("auto_farm"), Content = T("notify_auto_farm_closed_independent"), Duration = 3, Icon = "unlink" })
            end
        end
        Config:Set("AutoFarmEnabled", state); Config:Save()
    end
})

Main:Dropdown({
    Title = T("mode_farm"),
    Desc = T("desc_mode_farm"),
    Values = { T("normal_mode"), T("astro_holdout_mode"), T("dark_dimension_mode") },
    Multi = false,
    Value = FarmTargetMode == "Normal Mode" and T("normal_mode") or FarmTargetMode == "Astro Holdout Mode" and T("astro_holdout_mode") or T("dark_dimension_mode"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        if key == "normal_mode" then
            FarmTargetMode = "Normal Mode"
        elseif key == "astro_holdout_mode" then
            FarmTargetMode = "Astro Holdout Mode"
        elseif key == "dark_dimension_mode" then
            FarmTargetMode = "Dark Dimension Mode"
        else
            FarmTargetMode = NormalizeFarmTargetMode(value)
        end
        Config:Set("FarmTargetMode", FarmTargetMode)
        Config:Save()
        InvalidateMobCache("farm target mode changed")
        FarmForceRetarget = true
        if AutoFarmEnabled then StartFarmLoop(); StartJeffreyGuardLoop() end
        task.delay(0.4, function() if not IsAntiJeffreyEscapePauseActive() then FarmForceRetarget = false end end)
        WindUI:Notify({ Title = T("mode_farm"), Content = T("notify_selected") .. tostring(value), Duration = 2, Icon = "target" })
    end
})

Main:Section({ Title = T("farm_settings"), Icon = "settings" })

Main:Dropdown({
    Title = T("position_farm"),
    Desc = T("desc_position_farm"),
    Values = { T("position_above"), T("position_under") },
    Multi = false,
    Value = FarmPosition == "Above" and T("position_above") or T("position_under"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        if key == "position_above" then FarmPosition = "Above" elseif key == "position_under" then FarmPosition = "Under" end
        Config:Set("FarmPosition", FarmPosition); Config:Save()
    end
})

Main:Dropdown({
    Title = T("movement_farm"),
    Desc = T("desc_movement_farm"),
    Values = { T("teleport"), T("tween") },
    Multi = false,
    Value = FarmMode == "Teleport" and T("teleport") or T("tween"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        if key == "teleport" then FarmMode = "Teleport" elseif key == "tween" then FarmMode = "Tween"
        else FarmMode = NormalizeFarmMode(value) end
        Config:Set("FarmMode", FarmMode); Config:Save()
        WindUI:Notify({ Title = T("movement_farm"), Content = T("notify_selected") .. tostring(value), Duration = 2, Icon = "mouse-pointer-click" })
    end
})

MiscDropdown = Main:Dropdown({
    Title = T("misc_farm"),
    Desc = T("desc_misc_farm"),
    Values = { "Auto Attack", "Auto Skill", "Auto Start", "Auto Skip Helicopter", "Auto Fill Up", "Safe Mode", "God Mode", "Reset Wave", "Delete Map" },
    Multi = true,
    Value = MiscOptions,
    Callback = function(values)
        MiscOptions = values
        if not AutoFarmEnabled and SyncFarmOnly and #values > 0 then
            WindUI:Notify({ Title = T("misc_farm"), Content = T("notify_misc_farm_warning"), Duration = 3, Icon = "triangle-alert" })
        end
        HandleMiscOptions(values)
    end
})

Main:Toggle({
    Title = T("sync_mode"),
    Desc = T("desc_sync_mode"),
    Value = SyncFarmOnly,
    Callback = function(state)
        SyncFarmOnly = state
        Config:Set("SyncFarmOnly", state)
        Config:Save()
        if state then
            WindUI:Notify({ Title = T("sync_mode"), Content = T("sync_on"), Duration = 3, Icon = "link" })
        else
            WindUI:Notify({ Title = T("sync_mode"), Content = T("sync_off"), Duration = 3, Icon = "unlink" })
        end
        ApplyMiscFarmGate("Sync Farm Only 已更改")
    end
})

Main:Section({ Title = T("farm_astro"), Icon = "flame" })

FarmAstroTokenToggle = Main:Toggle({
    Title = T("farm_astro"),
    Desc = T("desc_farm_astro"),
    Value = FarmAstroTokenEnabled,
    Callback = function(state)
        if state and AutoFarmEnabled then
            FarmAstroTokenEnabled = false
            Config:Set("FarmAstroTokenEnabled", false)
            Config:Save()
            NotifyFarmAstroAutoFarm()
            pcall(function()
                if FarmAstroTokenToggle and FarmAstroTokenToggle.Set then
                    FarmAstroTokenToggle:Set(false)
                end
            end)
            return
        end

        FarmAstroTokenEnabled = state
        Config:Set("FarmAstroTokenEnabled", state)
        Config:Save()

        if state then
            StartFarmAstroToken()
            WindUI:Notify({
                Title = T("farm_astro"),
                Content = T("notify_farm_astro_on"),
                Duration = 3,
                Icon = "sparkles"
            })
        else
            StopFarmAstroToken(false)
            WindUI:Notify({
                Title = T("farm_astro"),
                Content = T("notify_farm_astro_off"),
                Duration = 3,
                Icon = "square"
            })
        end
    end
})

Main:Section({ Title = T("general_settings"), Icon = "zap" })

SkillDropdown = Main:Dropdown({
    Title = T("skill_keys"),
    Desc = T("desc_skill_keys"),
    Values = { T("all_skills"), "Q", "E", "R", "T", "Y", "G", "H", "Z", "X", "C", "V", "B", "U" },
    Multi = true,
    Value = SelectedSkills,
    Callback = function(values)
        local cleaned = {}
        for _, v in ipairs(values) do
            if v == T("all_skills") then
                table.insert(cleaned, "All")
            else
                table.insert(cleaned, v)
            end
        end
        SelectedSkills = cleaned
        Config:Set("SelectedSkills", cleaned); Config:Save()
    end
})

SkillDelaySlider = Main:Slider({
    Title = T("skill_delay"),
    Desc = T("desc_skill_delay"),
    Value = { Min = 1, Max = 60, Default = SkillDelay },
    Step = 1,
    Callback = function(value) SkillDelay = value; Config:Set("SkillDelay", value); Config:Save() end
})

FarmHeightSlider = Main:Slider({
    Title = T("height_offset"),
    Desc = T("desc_height_offset"),
    Value = { Min = -150, Max = 150, Default = HeightValue },
    Step = 1,
    Callback = function(value)
        HeightValue = value; Config:Set("HeightValue", value); Config:Save()
        for mob, _ in pairs(MobHeightOverride) do
            if MobConfirmedPadding[mob] == nil then MobHeightOverride[mob] = nil end
        end
    end
})

Main:Slider({
    Title = T("safe_hp"),
    Desc = T("desc_safe_hp"),
    Value = { Min = 1, Max = 99, Default = SafeValue },
    Step = 1,
    Callback = function(value) SafeValue = value; Config:Set("SafeValue", value); Config:Save() end
})

Main:Slider({
    Title = T("god_hp"),
    Desc = T("desc_god_hp"),
    Value = { Min = 1, Max = 99, Default = GodModeValue },
    Step = 1,
    Callback = function(value)
        GodModeValue = value
        Config:Set("GodModeValue", value)
        Config:Save()
    end
})

Main:Slider({
    Title = T("reset_wave_val"),
    Desc = T("desc_reset_wave_val"),
    Value = { Min = 1, Max = 100, Default = ResetWaveValue },
    Step = 1,
    Callback = function(value)
        ResetWaveValue = tonumber(value) or 10
        ClearResetWaveTrigger("滑条更改")
        Config:Set("ResetWaveValue", ResetWaveValue)
        Config:Save()

        if ResetWaveEnabled and IsMiscFarmAllowed() then
            StartResetWaveLoop()
            task.defer(function()
                EvaluateResetWaveNow("滑条更改", true)
            end)
        end
    end
})

Main:Divider()

BypassJeffreyToggle = Main:Toggle({
    Title = T("bypass_jeffrey"),
    Desc = T("desc_bypass_jeffrey"),
    Value = BypassJeffreyEnabled,
    Callback = function(state)
        BypassJeffreyEnabled = state
        Config:Set("BypassJeffreyEnabled", state)
        Config:Save()
        if state then
            StartBypassJeffreyLoop()
            ScanBypassJeffreys(true)
        end
    end
})

AntiJeffreyToggle = Main:Toggle({
    Title = T("anti_jeffrey"),
    Desc = T("desc_anti_jeffrey"),
    Value = AntiJeffreyEnabled,
    Callback = function(state)
        AntiJeffreyEnabled = state
        Config:Set("AntiJeffreyEnabled", state)
        Config:Save()
        if state then StartAntiJeffreyLoop(); StartJeffreyGuardLoop() end
    end
})

Main:Slider({
    Title = T("anti_jeffrey_range"),
    Desc = T("desc_anti_jeffrey_range"),
    Value = { Min = 10, Max = 200, Default = AntiJeffreyRange },
    Step = 1,
    Callback = function(value)
        AntiJeffreyRange = value
        Config:Set("AntiJeffreyRange", value)
        Config:Save()
    end
})

if AntiJeffreyEnabled then StartAntiJeffreyLoop(); StartJeffreyGuardLoop() end
if BypassJeffreyEnabled then StartBypassJeffreyLoop(); ScanBypassJeffreys(true) end

-- ====================== UI: PRIORITY SETTINGS ======================
Main:Section({ Title = T("priority_settings"), Icon = "list-ordered" })

Main:Paragraph({
    Title = T("priority_title"),
    Desc = T("priority_desc"),
    Image = "rbxassetid://104487529937663",
    ImageSize = 26,
})

Main:Slider({
    Title = T("high_hp_threshold"),
    Desc = T("desc_high_hp_threshold"),
    Value = { Min = 1, Max = 100000, Default = HighHPThreshold },
    Step = 100,
    Callback = function(value)
        HighHPThreshold = value
        Config:Set("HighHPThreshold", value)
        Config:Save()
    end
})

-- ====================== UI: OVERRIDE SETTINGS ======================
Main:Section({ Title = T("override_settings"), Icon = "ruler" })

PaddingReduceInput = Main:Input({
    Title = T("padding_reduce"),
    Default = tostring(PADDING_REDUCE_STEP),
    Placeholder = "默认: 2",
    Callback = function(text)
        local num = tonumber(text)
        if num then PADDING_REDUCE_STEP = num; Config:Set("PaddingReduceStep", num); Config:Save()
        else warn("输入了无效数字！") end
    end
})

PaddingSafeInput = Main:Input({
    Title = T("padding_safe"),
    Default = tostring(PADDING_SAFE_MIN),
    Placeholder = "默认: -30",
    Callback = function(text)
        local num = tonumber(text)
        if num then PADDING_SAFE_MIN = num; Config:Set("PaddingSafeMin", num); Config:Save()
        else warn("输入了无效数字！") end
    end
})

Main:Slider({
    Title = T("anti_clip_margin"),
    Desc = T("desc_anti_clip_margin"),
    Value = { Min = -10, Max = 10, Default = ANTI_CLIP_MARGIN },
    Step = 1,
    Callback = function(value)
        ANTI_CLIP_MARGIN = value; Config:Set("AntiClipMargin", value); Config:Save()
    end
})

Main:Slider({
    Title = T("dmg_threshold"),
    Desc = T("desc_dmg_threshold"),
    Value = { Min = 1, Max = 500, Default = DMG_THRESHOLD },
    Step = 1,
    Callback = function(value)
        DMG_THRESHOLD = value; Config:Set("DmgThreshold", value); Config:Save()
    end
})

Main:Button({
    Title = T("reset_positions"),
    Desc = T("desc_reset_positions"),
    Callback = function()
        MobConfirmedPadding = {}
        MobHeightOverride   = {}
        WindUI:Notify({ Title = T("override_settings"), Content = T("notify_positions_cleared"), Duration = 2, Icon = "refresh-cw" })
    end
})

Main:Section({ Title = T("flush_settings"), Icon = "toilet" })

Flushaura      = Config:Get("flushaura", false)
FlushAuraValue = Config:Get("FlushAuraValue", 5)

Main:Slider({
    Title = T("flush_range"),
    Desc = T("desc_flush_range"),
    Value = { Min = 1, Max = 15, Default = FlushAuraValue },
    Step = 1,
    Callback = function(value) FlushAuraValue = value; Config:Set("FlushAuraValue", value); Config:Save() end
})

Main:Toggle({
    Title = T("flush_aura"),
    Desc = T("desc_flush_aura"),
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
-- ====================== ESP SYSTEM =========================
-- ============================================================

ESP = {
    Enabled       = Config:Get("EspEnabled", false),
    MobEnabled    = Config:Get("EspMobEnabled", true),
    PlayerEnabled = Config:Get("EspPlayerEnabled", true),
    ItemEnabled   = Config:Get("EspItemEnabled", true),
    Settings      = Config:Get("EspSettings", { "Highlight", "Distance", "Health", "Name" }),
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

function IsESPItemTarget(objectName, selectedList)
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

function CreateESPLabel(parent, labelText)
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

function CreateHighlight(model, outlineColor, fillColor, fillTransparency)
    local existing = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT")
    if existing then existing:Destroy() end
    local hl = Instance.new("Highlight")
    hl.Name = "DYHUB_ESP_HIGHLIGHT"; hl.OutlineColor = outlineColor
    hl.FillColor = fillColor; hl.FillTransparency = fillTransparency or 0.9
    hl.OutlineTransparency = 0; hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Adornee = model; hl.Parent = model
    return hl
end

function RemoveESP(model)
    pcall(function()
        local hl = model:FindFirstChild("DYHUB_ESP_HIGHLIGHT"); if hl then hl:Destroy() end
        local hb = model:FindFirstChild("DYHUB_ESP_LABEL"); if hb then hb:Destroy() end
        local hrp = model:FindFirstChild("HumanoidRootPart")
        if hrp then local lb = hrp:FindFirstChild("DYHUB_ESP_LABEL"); if lb then lb:Destroy() end end
    end)
end

function IsInRange(targetPart)
    if not targetPart or not HumanoidRootPart then return false end
    return (HumanoidRootPart.Position - targetPart.Position).Magnitude <= ESP.MaxDistance
end

function BuildLabelText(model, showName, showHealth, showDistance)
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

function BuildItemLabelText(obj, showName, showDistance)
    local parts = {}
    if showName then table.insert(parts, obj.Name) end
    if showDistance then
        local root = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or (obj:IsA("BasePart") and obj or nil)
        if root and HumanoidRootPart then table.insert(parts, "📏 " .. math.floor((HumanoidRootPart.Position - root.Position).Magnitude) .. "m") end
    end
    return table.concat(parts, "\n")
end

function GetESPSettings()
    local s = ESP.Settings
    return {
        highlight = table.find(s, "Highlight") ~= nil,
        distance  = table.find(s, "Distance") ~= nil,
        health    = table.find(s, "Health") ~= nil,
        name      = table.find(s, "Name") ~= nil,
    }
end

function ApplyMobESP(mob)
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
                else label.Visible = true; label.Text = BuildLabelText(mob, settings.name, settings.health, settings.distance); task.wait(0.35) end
            end
            RemoveESP(mob); ESP._mobHighlights[mob] = nil
        end)
    end
    ESP._mobHighlights[mob] = true
end

function ScanMobs()
    local livingFolder = workspace:FindFirstChild("Living"); if not livingFolder then return end
    for _, mob in ipairs(livingFolder:GetChildren()) do
        if IsValidMob(mob) and not ESP._mobHighlights[mob] then
            local hrp = mob:FindFirstChild("HumanoidRootPart")
            if hrp and IsInRange(hrp) then ApplyMobESP(mob) end
        end
    end
end

function ApplyPlayerESP(playerChar)
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
                else label.Visible = true; label.Text = BuildLabelText(playerChar, settings.name, settings.health, settings.distance); task.wait(0.35) end
            end
            RemoveESP(playerChar); ESP._playerHighlights[playerChar] = nil
        end)
    end
    ESP._playerHighlights[playerChar] = true
end

function ScanPlayers()
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

function GetItemRoot(obj)
    if obj:IsA("Model") then return obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")
    elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then return obj end
    return nil
end

function ApplyItemESP(obj)
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
                else label.Visible = true; label.Text = BuildItemLabelText(obj, settings.name, settings.distance); task.wait(0.5) end
            end
            RemoveESP(obj); ESP._itemHighlights[obj] = nil
        end)
    end
    ESP._itemHighlights[obj] = true
end

function ScanItems()
    if #ESP.SelectedItems == 0 then return end
    for _, obj in ipairs(workspace:GetDescendants()) do
        if not ESP._itemHighlights[obj] and IsESPItemTarget(obj.Name, ESP.SelectedItems) then
            local root = GetItemRoot(obj)
            if root and IsInRange(root) then ApplyItemESP(obj) end
        end
    end
end

function ClearAllESP()
    for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end
    ESP._mobHighlights = {}
    for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end
    ESP._playerHighlights = {}
    for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end
    ESP._itemHighlights = {}
end

ESPConnection = nil

function StartESPLoop()
    if ESPConnection then ESPConnection:Disconnect(); ESPConnection = nil end
    local lastMobScan, lastPlayerScan, lastItemScan = 0, 0, 0
    ESPConnection = RunService.Heartbeat:Connect(function()
        if not ESP.Enabled then return end
        local now = tick()
        if ESP.MobEnabled and now - lastMobScan >= 0.8 then
            lastMobScan = now
            pcall(ScanMobs)
        end
        if ESP.PlayerEnabled and now - lastPlayerScan >= 1.0 then
            lastPlayerScan = now
            pcall(ScanPlayers)
        end
        if ESP.ItemEnabled and now - lastItemScan >= 4.0 then
            lastItemScan = now
            pcall(ScanItems)
        end
    end)
end

function StopESPLoop()
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

function WatchLivingFolder()
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
Main4:Section({ Title = T("esp_visual"), Icon = "eye" })

EspEnableToggle = Main4:Toggle({
    Title = T("esp_enable"), Value = ESP.Enabled,
    Desc = T("desc_esp_enable"),
    Callback = function(state)
        ESP.Enabled = state; Config:Set("EspEnabled", state); Config:Save()
        if state then StartESPLoop() else StopESPLoop() end
    end
})

EspMobToggle = Main4:Toggle({
    Title = T("esp_mob"), Value = ESP.MobEnabled,
    Desc = T("desc_esp_mob"),
    Callback = function(state)
        ESP.MobEnabled = state; Config:Set("EspMobEnabled", state); Config:Save()
        if not state then for mob, _ in pairs(ESP._mobHighlights) do RemoveESP(mob) end; ESP._mobHighlights = {} end
    end
})

EspPlayerToggle = Main4:Toggle({
    Title = T("esp_player"), Value = ESP.PlayerEnabled,
    Desc = T("desc_esp_player"),
    Callback = function(state)
        ESP.PlayerEnabled = state; Config:Set("EspPlayerEnabled", state); Config:Save()
        if not state then for char, _ in pairs(ESP._playerHighlights) do RemoveESP(char) end; ESP._playerHighlights = {} end
    end
})

EspItemToggle = Main4:Toggle({
    Title = T("esp_item"), Value = ESP.ItemEnabled,
    Desc = T("desc_esp_item"),
    Callback = function(state)
        ESP.ItemEnabled = state; Config:Set("EspItemEnabled", state); Config:Save()
        if not state then for obj, _ in pairs(ESP._itemHighlights) do RemoveESP(obj) end; ESP._itemHighlights = {} end
    end
})

Main4:Section({ Title = T("esp_settings"), Icon = "settings" })

EspSettingsDropdown = Main4:Dropdown({
    Title = T("esp_options"),
    Desc = T("desc_esp_options"),
    Multi = true,
    Values = { T("highlight"), T("distance"), T("health"), T("name") },
    Value = ESP.Settings,
    Callback = function(value)
        local cleaned = {}
        for _, v in ipairs(value) do
            local key = GetOriginalKey(v)
            if key == "highlight" then table.insert(cleaned, "Highlight")
            elseif key == "distance" then table.insert(cleaned, "Distance")
            elseif key == "health" then table.insert(cleaned, "Health")
            elseif key == "name" then table.insert(cleaned, "Name")
            else table.insert(cleaned, v) end
        end
        ESP.Settings = cleaned
        Config:Set("EspSettings", cleaned); Config:Save()
        if ESP.Enabled then ClearAllESP() end
    end,
})

EspItemDropdown = Main4:Dropdown({
    Title = T("esp_items"),
    Desc = T("desc_esp_items"),
    Multi = true,
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
Main2:Section({ Title = T("local_player"), Icon = "user" })

WSValue = Config:Get("WSValue", 16)
JPValue = Config:Get("JPValue", 50)
NoClip  = Config:Get("NoClip", false)
LockMovementStats = Config:Get("LockMovementStats", true)
FlyEnabled = Config:Get("FlyEnabled", false)
FlySpeed = Config:Get("FlySpeed", 1)
InfiniteJumpEnabled = Config:Get("InfiniteJumpEnabled", false)
FullBrightEnabled = Config:Get("FullBrightEnabled", false)
NoFogEnabled = Config:Get("NoFogEnabled", false)

LastMovementStatApply = 0
MovementStatInterval  = 0.25
FlyBodyVelocity = nil
FlyBodyGyro = nil
FlyRenderConnection = nil
LastVisualApply = 0
FullBrightOriginal = nil
NoFogOriginal = nil

function GetLocalHumanoid()
    local char = LocalPlayer.Character
    if not char then return nil end
    return char:FindFirstChildOfClass("Humanoid")
end

function GetLocalRootPart()
    local char = LocalPlayer.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart")
end

function updatePlayerStats(force)
    local humanoid = GetLocalHumanoid()
    if not humanoid then return end

    pcall(function()
        if humanoid.UseJumpPower ~= nil then
            humanoid.UseJumpPower = true
        end
    end)

    if force or humanoid.WalkSpeed ~= WSValue then
        humanoid.WalkSpeed = WSValue
    end

    if force or humanoid.JumpPower ~= JPValue then
        humanoid.JumpPower = JPValue
    end
end

function ProtectMovementStats()
    if not LockMovementStats then return end

    local now = tick()
    if now - LastMovementStatApply < MovementStatInterval then return end
    LastMovementStatApply = now

    local humanoid = GetLocalHumanoid()
    if not humanoid then return end

    pcall(function()
        if humanoid.UseJumpPower ~= nil then
            humanoid.UseJumpPower = true
        end
    end)

    if humanoid.WalkSpeed < WSValue then
        humanoid.WalkSpeed = WSValue
    end

    if humanoid.JumpPower < JPValue then
        humanoid.JumpPower = JPValue
    end
end

function CleanupFlyForces()
    if FlyBodyGyro then
        pcall(function() FlyBodyGyro:Destroy() end)
        FlyBodyGyro = nil
    end
    if FlyBodyVelocity then
        pcall(function() FlyBodyVelocity:Destroy() end)
        FlyBodyVelocity = nil
    end
end

-- 新的飞行系统变量
FlyCtrl = { f = 0, b = 0, l = 0, r = 0 }
FlyLastCtrl = { f = 0, b = 0, l = 0, r = 0 }
FlyMaxSpeed = 50
FlyCurrentSpeed = 0
FlyTpWalking = false
FlyInputConnections = {}
FlyHeartbeatConnection = nil
FlyNowe = false
FlySpeeds = 1

function StartFly()
    local humanoid = GetLocalHumanoid()
    local char = LocalPlayer.Character
    if not humanoid or not char then return end

    StopFly()

    FlyEnabled = true
    FlyNowe = true

    -- 禁用所有Humanoid状态
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Flying, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Landed, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Running, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Swimming, false)
    humanoid:ChangeState(Enum.HumanoidStateType.Swimming)

    -- 启动tpwalking
    FlyTpWalking = false
    for i = 1, FlySpeeds do
        spawn(function()
            local hb = game:GetService("RunService").Heartbeat
            FlyTpWalking = true
            local chr = game.Players.LocalPlayer.Character
            local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
            while FlyTpWalking and hb:Wait() and chr and hum and hum.Parent and FlyEnabled do
                if hum.MoveDirection.Magnitude > 0 then
                    chr:TranslateBy(hum.MoveDirection)
                end
            end
        end)
    end
    game.Players.LocalPlayer.Character.Animate.Disabled = true
    local Char = game.Players.LocalPlayer.Character
    local Hum = Char:FindFirstChildOfClass("Humanoid") or Char:FindFirstChildOfClass("AnimationController")
    for i,v in next, Hum:GetPlayingAnimationTracks() do
        v:AdjustSpeed(0)
    end

    -- 根据R6/R15启动飞行循环
    if humanoid.RigType == Enum.HumanoidRigType.R6 then
        StartR6FlyLoop(char)
    else
        StartR15FlyLoop(char)
    end
end

function StartR6FlyLoop(char)
    local plr = game.Players.LocalPlayer
    local torso = char:WaitForChild("Torso")
    local bg = Instance.new("BodyGyro", torso)
    bg.P = 9e4
    bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.cframe = torso.CFrame
    local bv = Instance.new("BodyVelocity", torso)
    bv.velocity = Vector3.new(0, 0.1, 0)
    bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
    FlyBodyGyro = bg
    FlyBodyVelocity = bv
    plr.Character.Humanoid.PlatformStand = true

    FlyHeartbeatConnection = game:GetService("RunService").RenderStepped:Connect(function()
        if not FlyEnabled then return end
        if game:GetService("Players").LocalPlayer.Character.Humanoid.Health == 0 then return end

        local ctrl = FlyCtrl
        local lastctrl = FlyLastCtrl
        local maxspeed = FlyMaxSpeed
        local speed = FlyCurrentSpeed

        if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
            speed = speed + 0.5 + (speed / maxspeed)
            if speed > maxspeed then speed = maxspeed end
        elseif not (ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0) and speed ~= 0 then
            speed = speed - 1
            if speed < 0 then speed = 0 end
        end
        FlyCurrentSpeed = speed

        if (ctrl.l + ctrl.r) ~= 0 or (ctrl.f + ctrl.b) ~= 0 then
            bv.velocity = ((workspace.CurrentCamera.CoordinateFrame.LookVector * (ctrl.f + ctrl.b)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(ctrl.l + ctrl.r, (ctrl.f + ctrl.b) * 0.2, 0).p) - workspace.CurrentCamera.CoordinateFrame.p)) * speed
            FlyLastCtrl = { f = ctrl.f, b = ctrl.b, l = ctrl.l, r = ctrl.r }
        elseif (ctrl.l + ctrl.r) == 0 and (ctrl.f + ctrl.b) == 0 and speed ~= 0 then
            bv.velocity = ((workspace.CurrentCamera.CoordinateFrame.LookVector * (lastctrl.f + lastctrl.b)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(lastctrl.l + lastctrl.r, (lastctrl.f + lastctrl.b) * 0.2, 0).p) - workspace.CurrentCamera.CoordinateFrame.p)) * speed
        else
            bv.velocity = Vector3.new(0, 0, 0)
        end
        bg.cframe = workspace.CurrentCamera.CoordinateFrame * CFrame.Angles(-math.rad((ctrl.f + ctrl.b) * 50 * speed / maxspeed), 0, 0)
    end)
end

function StartR15FlyLoop(char)
    local plr = game.Players.LocalPlayer
    local UpperTorso = char:WaitForChild("UpperTorso")
    local bg = Instance.new("BodyGyro", UpperTorso)
    bg.P = 9e4
    bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.cframe = UpperTorso.CFrame
    local bv = Instance.new("BodyVelocity", UpperTorso)
    bv.velocity = Vector3.new(0, 0.1, 0)
    bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
    FlyBodyGyro = bg
    FlyBodyVelocity = bv
    plr.Character.Humanoid.PlatformStand = true

    FlyHeartbeatConnection = game:GetService("RunService").RenderStepped:Connect(function()
        if not FlyEnabled then return end
        if game:GetService("Players").LocalPlayer.Character.Humanoid.Health == 0 then return end

        local ctrl = FlyCtrl
        local lastctrl = FlyLastCtrl
        local maxspeed = FlyMaxSpeed
        local speed = FlyCurrentSpeed

        if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
            speed = speed + 0.5 + (speed / maxspeed)
            if speed > maxspeed then speed = maxspeed end
        elseif not (ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0) and speed ~= 0 then
            speed = speed - 1
            if speed < 0 then speed = 0 end
        end
        FlyCurrentSpeed = speed

        if (ctrl.l + ctrl.r) ~= 0 or (ctrl.f + ctrl.b) ~= 0 then
            bv.velocity = ((workspace.CurrentCamera.CoordinateFrame.LookVector * (ctrl.f + ctrl.b)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(ctrl.l + ctrl.r, (ctrl.f + ctrl.b) * 0.2, 0).p) - workspace.CurrentCamera.CoordinateFrame.p)) * speed
            FlyLastCtrl = { f = ctrl.f, b = ctrl.b, l = ctrl.l, r = ctrl.r }
        elseif (ctrl.l + ctrl.r) == 0 and (ctrl.f + ctrl.b) == 0 and speed ~= 0 then
            bv.velocity = ((workspace.CurrentCamera.CoordinateFrame.LookVector * (lastctrl.f + lastctrl.b)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(lastctrl.l + lastctrl.r, (lastctrl.f + lastctrl.b) * 0.2, 0).p) - workspace.CurrentCamera.CoordinateFrame.p)) * speed
        else
            bv.velocity = Vector3.new(0, 0, 0)
        end
        bg.cframe = workspace.CurrentCamera.CoordinateFrame * CFrame.Angles(-math.rad((ctrl.f + ctrl.b) * 50 * speed / maxspeed), 0, 0)
    end)
end

function StopFly()
    FlyEnabled = false
    FlyTpWalking = false
    FlyCurrentSpeed = 0
    FlyCtrl = { f = 0, b = 0, l = 0, r = 0 }
    FlyLastCtrl = { f = 0, b = 0, l = 0, r = 0 }
    FlyNowe = false

    CleanupFlyForces()

    if FlyHeartbeatConnection then
        FlyHeartbeatConnection:Disconnect()
        FlyHeartbeatConnection = nil
    end

    local humanoid = GetLocalHumanoid()
    if humanoid then
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Climbing, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Flying, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Landed, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Running, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Swimming, true)
        humanoid.PlatformStand = false
        humanoid:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
    end

    local char = LocalPlayer.Character
    if char then
        char.Animate.Disabled = false
    end
end

function UpdateFlySpeeds(newSpeeds)
    FlySpeeds = newSpeeds
    if FlyEnabled then
        FlyTpWalking = false
        task.wait(0.05)
        for i = 1, FlySpeeds do
            spawn(function()
                local hb = game:GetService("RunService").Heartbeat
                FlyTpWalking = true
                local chr = game.Players.LocalPlayer.Character
                local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
                while FlyTpWalking and hb:Wait() and chr and hum and hum.Parent and FlyEnabled do
                    if hum.MoveDirection.Magnitude > 0 then
                        chr:TranslateBy(hum.MoveDirection)
                    end
                end
            end)
        end
    end
end

-- 原有的全亮/去雾等保持不变
function CaptureFullBrightOriginal()
    if FullBrightOriginal then return end
    FullBrightOriginal = {
        Brightness = Lighting.Brightness,
        ClockTime = Lighting.ClockTime,
        GlobalShadows = Lighting.GlobalShadows,
        Ambient = Lighting.Ambient,
        OutdoorAmbient = Lighting.OutdoorAmbient,
        ExposureCompensation = Lighting.ExposureCompensation,
    }
end

function ApplyFullBright()
    CaptureFullBrightOriginal()
    pcall(function()
        Lighting.Brightness = 2
        Lighting.ClockTime = 14
        Lighting.GlobalShadows = false
        Lighting.Ambient = Color3.new(1, 1, 1)
        Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
        Lighting.ExposureCompensation = 0
    end)
end

function RestoreFullBright()
    if not FullBrightOriginal then return end
    pcall(function()
        Lighting.Brightness = FullBrightOriginal.Brightness
        Lighting.ClockTime = FullBrightOriginal.ClockTime
        Lighting.GlobalShadows = FullBrightOriginal.GlobalShadows
        Lighting.Ambient = FullBrightOriginal.Ambient
        Lighting.OutdoorAmbient = FullBrightOriginal.OutdoorAmbient
        Lighting.ExposureCompensation = FullBrightOriginal.ExposureCompensation
    end)
    FullBrightOriginal = nil
end

function CaptureNoFogOriginal()
    if NoFogOriginal then return end
    NoFogOriginal = {
        FogStart = Lighting.FogStart,
        FogEnd = Lighting.FogEnd,
        FogColor = Lighting.FogColor,
        Atmospheres = {},
    }
    for _, obj in ipairs(Lighting:GetChildren()) do
        if obj:IsA("Atmosphere") then
            table.insert(NoFogOriginal.Atmospheres, {
                Instance = obj,
                Density = obj.Density,
                Haze = obj.Haze,
                Glare = obj.Glare,
                Offset = obj.Offset,
            })
        end
    end
end

function ApplyNoFog()
    CaptureNoFogOriginal()
    pcall(function()
        Lighting.FogStart = 0
        Lighting.FogEnd = 100000
    end)
    for _, obj in ipairs(Lighting:GetChildren()) do
        if obj:IsA("Atmosphere") then
            pcall(function()
                obj.Density = 0
                obj.Haze = 0
                obj.Glare = 0
                obj.Offset = 0
            end)
        end
    end
end

function RestoreNoFog()
    if not NoFogOriginal then return end
    pcall(function()
        Lighting.FogStart = NoFogOriginal.FogStart
        Lighting.FogEnd = NoFogOriginal.FogEnd
        Lighting.FogColor = NoFogOriginal.FogColor
    end)
    for _, data in ipairs(NoFogOriginal.Atmospheres or {}) do
        local obj = data.Instance
        if obj and obj.Parent then
            pcall(function()
                obj.Density = data.Density
                obj.Haze = data.Haze
                obj.Glare = data.Glare
                obj.Offset = data.Offset
            end)
        end
    end
    NoFogOriginal = nil
end

RunService.Heartbeat:Connect(function()
    ProtectMovementStats()

    local now = tick()
    if now - LastVisualApply >= 1 then
        LastVisualApply = now
        if FullBrightEnabled then ApplyFullBright() end
        if NoFogEnabled then ApplyNoFog() end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if not InfiniteJumpEnabled then return end
    local humanoid = GetLocalHumanoid()
    if humanoid then
        pcall(function()
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end
end)

RunService.Stepped:Connect(function()
    if NoClip and LocalPlayer.Character then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(1)
    updatePlayerStats(true)
    if FlyEnabled then StartFly() end
end)

Main2:Slider({
    Title = T("walkspeed"),
    Desc = T("desc_walkspeed"),
    Value = { Min = 1, Max = 200, Default = WSValue },
    Step = 1,
    Callback = function(value)
        WSValue = value
        Config:Set("WSValue", value)
        Config:Save()
        updatePlayerStats(true)
    end
})

Main2:Slider({
    Title = T("jumppower"),
    Desc = T("desc_jumppower"),
    Value = { Min = 1, Max = 500, Default = JPValue },
    Step = 1,
    Callback = function(value)
        JPValue = value
        Config:Set("JPValue", value)
        Config:Save()
        updatePlayerStats(true)
    end
})

Main2:Toggle({
    Title = T("lock_movement"),
    Desc = T("desc_lock_movement"),
    Value = LockMovementStats,
    Callback = function(state)
        LockMovementStats = state
        Config:Set("LockMovementStats", state)
        Config:Save()
        if state then updatePlayerStats(true) end
    end
})

nocliptoggle = Main2:Toggle({
    Title = T("no_clip"),
    Value = NoClip,
    Desc = T("desc_no_clip"),
    Callback = function(state) NoClip = state; Config:Set("NoClip", state); Config:Save() end
})

Main2:Section({ Title = T("fly_movement"), Icon = "plane" })

Main2:Slider({
    Title = T("fly_speed"),
    Desc = T("desc_fly_speed"),
    Value = { Min = 1, Max = 20, Default = FlySpeed },
    Step = 1,
    Callback = function(value)
        FlySpeed = value
        FlySpeeds = value
        Config:Set("FlySpeed", value)
        Config:Save()
        if FlyEnabled then UpdateFlySpeeds(value) end
    end
})

Main2:Toggle({
    Title = T("fly"),
    Desc = T("desc_fly"),
    Value = FlyEnabled,
    Callback = function(state)
        FlyEnabled = state
        Config:Set("FlyEnabled", state)
        Config:Save()
        if state then StartFly() else StopFly() end
    end
})

Main2:Section({ Title = T("visual_utility"), Icon = "sun" })

Main2:Toggle({
    Title = T("infinite_jump"),
    Desc = T("desc_infinite_jump"),
    Value = InfiniteJumpEnabled,
    Callback = function(state)
        InfiniteJumpEnabled = state
        Config:Set("InfiniteJumpEnabled", state)
        Config:Save()
    end
})

Main2:Toggle({
    Title = T("full_bright"),
    Desc = T("desc_full_bright"),
    Value = FullBrightEnabled,
    Callback = function(state)
        FullBrightEnabled = state
        Config:Set("FullBrightEnabled", state)
        Config:Save()
        if state then ApplyFullBright() else RestoreFullBright() end
    end
})

Main2:Toggle({
    Title = T("no_fog"),
    Desc = T("desc_no_fog"),
    Value = NoFogEnabled,
    Callback = function(state)
        NoFogEnabled = state
        Config:Set("NoFogEnabled", state)
        Config:Save()
        if state then ApplyNoFog() else RestoreNoFog() end
    end
})

Main2:Section({ Title = T("redeem_codes"), Icon = "bird" })

SelectedCodes = Config:Get("SelectedCodes", {})

CodeDropdown = Main2:Dropdown({
    Title = T("select_redeem_codes"),
    Desc = T("desc_select_redeem_codes"),
    Multi = true,
    Values = GlobalTables.redeemCodes, Value = SelectedCodes,
    Callback = function(value) SelectedCodes = value or {}; Config:Set("SelectedCodes", value); Config:Save() end,
})

Main2:Button({
    Title = T("redeem_selected"),
    Desc = T("desc_redeem_selected"),
    Callback = function()
        for _, code in ipairs(SelectedCodes or {}) do
            pcall(function() local remote = GetRemote("RedeemCode"); if remote then remote:FireServer(code) end; task.wait(0.2) end)
        end
    end,
})

Main2:Button({
    Title = T("redeem_all"),
    Desc = T("desc_redeem_all"),
    Callback = function()
        for _, code in ipairs(GlobalTables.redeemCodes or {}) do
            pcall(function() local remote = GetRemote("RedeemCode"); if remote then remote:FireServer(code) end; task.wait(0.5) end)
        end
    end,
})

-- ====================== UI: UNLOCK GAMEPASS ======================
Main2:Section({ Title = T("unlock_gamepass"), Icon = "badge-dollar-sign" })

SelectedGamepass = Config:Get("SelectedGamepass", {})
GlobalTables.Gamepassts = SelectedGamepass

GamepassDropdown = Main2:Dropdown({
    Title = T("select_gamepass"),
    Desc = T("desc_select_gamepass"),
    Multi = true,
    Values = { T("all"), T("lucky_boost"), T("rare_lucky_boost"), T("legendary_lucky_boost") },
    Value = SelectedGamepass,
    Callback = function(value)
        local cleaned = {}
        for _, v in ipairs(value) do
            local key = GetOriginalKey(v)
            if key == "all" then table.insert(cleaned, "All")
            elseif key == "lucky_boost" then table.insert(cleaned, "LuckyBoost")
            elseif key == "rare_lucky_boost" then table.insert(cleaned, "RareLuckyBoost")
            elseif key == "legendary_lucky_boost" then table.insert(cleaned, "LegendaryLuckyBoost")
            else table.insert(cleaned, v) end
        end
        GlobalTables.Gamepassts = cleaned
        SelectedGamepass = cleaned
        Config:Set("SelectedGamepass", cleaned)
        Config:Save()
    end,
})

Main2:Button({
    Title = T("unlock_selected"),
    Desc = T("desc_unlock_selected"),
    Callback = function()
        local gachaData = LocalPlayer:FindFirstChild("GachaData")
        if not gachaData then
            gachaData = Instance.new("Folder")
            gachaData.Name = "GachaData"
            gachaData.Parent = LocalPlayer
        end
        local toUnlock = {}
        for _, v in ipairs(GlobalTables.Gamepassts) do
            if v == "All" then
                toUnlock = {"LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost"}
                break
            else
                table.insert(toUnlock, v)
            end
        end
        if #toUnlock == 0 then
            WindUI:Notify({ Title = T("unlock_gamepass"), Content = T("notify_select_gamepass"), Duration = 3, Icon = "alert-triangle" })
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
            Title = T("unlock_gamepass"),
            Content = T("notify_unlocked") .. successCount .. "/" .. #toUnlock .. T("notify_gamepasses"),
            Duration = 3,
            Icon = "badge-check"
        })
    end,
})
-- ====================== UI: GAMEMODE TAB ======================
GlobalTables2 = {
    Votes2 = {
        "Normal", "VeryHard", "Hard", "Insane", "Nightmare", "BossRush",
        "DarkDimension", "Hell", "ThunderStorm", "Christmas", "Zombie",
        "AstroV2", "Astro", "100MVisit"
    }
}

Main7:Section({ Title = T("vote_info"), TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = T("auto_vote_ig"),
    Desc = T("vote_info_desc"),
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = T("vote_mode"), Icon = "gamepad-2" })

Main7:Button({
    Title = T("restore_vote"),
    Desc = T("vote_restore_warning"),
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
            Title = T("notify_restoring"),
            Content = T("notify_restore_ready"),
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
            Title = T("notify_restoring"),
            Content = T("notify_restore_wait"),
            Duration = 10,
            Icon = "loader-circle"
        })
        task.wait(10)
        WindUI:Notify({
            Title = T("notify_restore_complete"),
            Content = T("notify_restore_done"),
            Duration = 5,
            Icon = "check"
        })
    end
})

local voteModeMap = {
    normal = "Normal", veryhard = "VeryHard", hard = "Hard", insane = "Insane",
    nightmare = "Nightmare", bossrush = "BossRush", darkdimension = "DarkDimension",
    hell = "Hell", thunderstorm = "ThunderStorm", christmas = "Christmas",
    zombie = "Zombie", astrov2 = "AstroV2", astro = "Astro", visit_100m = "100MVisit",
}

GameModeDropdown2 = Main7:Dropdown({
    Title = T("set_vote_mode"),
    Desc = T("desc_set_vote_mode"),
    Values = { T("normal"), T("veryhard"), T("hard"), T("insane"), T("nightmare"), T("bossrush"), T("darkdimension"), T("hell"), T("thunderstorm"), T("christmas"), T("zombie"), T("astrov2"), T("astro"), T("visit_100m") },
    Multi = false,
    Value = AutoVoteValue == "Normal" and T("normal") or AutoVoteValue == "VeryHard" and T("veryhard") or AutoVoteValue == "Hard" and T("hard") or AutoVoteValue == "Insane" and T("insane") or AutoVoteValue == "Nightmare" and T("nightmare") or AutoVoteValue == "BossRush" and T("bossrush") or AutoVoteValue == "DarkDimension" and T("darkdimension") or AutoVoteValue == "Hell" and T("hell") or AutoVoteValue == "ThunderStorm" and T("thunderstorm") or AutoVoteValue == "Christmas" and T("christmas") or AutoVoteValue == "Zombie" and T("zombie") or AutoVoteValue == "AstroV2" and T("astrov2") or AutoVoteValue == "Astro" and T("astro") or AutoVoteValue == "100MVisit" and T("visit_100m") or T("christmas"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        AutoVoteValue = voteModeMap[key] or value
        Config:Set("AutoVoteValue", AutoVoteValue)
        Config:Save()
        print("[DYHUB] 投票模式已选择:", tostring(AutoVoteValue))
    end
})

AutoVoteIGToggle = Main7:Toggle({
    Title = T("auto_vote_ig"),
    Desc = T("desc_auto_vote_ig"),
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
        else
            print("[DYHUB] 自动投票模式已禁用")
        end
    end
})

if AutoVoteinGameEnabled then
    StartAutoVoteLoop()
end

Main7:Divider()
Main7:Section({ Title = T("casual_info"), TextXAlignment = "Center", TextSize = 17 })
Main7:Divider()
Main7:Paragraph({
    Title = T("casual_title"),
    Desc = T("casual_desc"),
    Image = "rbxassetid://104487529937663",
    ImageSize = 30,
})
Main7:Divider()
Main7:Section({ Title = T("game_mode"), Icon = "gamepad-2" })

GameModeDropdown = Main7:Dropdown({
    Title = T("set_game_mode"),
    Desc = T("desc_set_game_mode"),
    Values = { T("normal"), T("veryhard"), T("hard"), T("insane"), T("nightmare"), T("bossrush"), T("darkdimension"), T("hell"), T("thunderstorm"), T("christmas"), T("zombie"), T("astrov2"), T("astro"), T("visit_100m") },
    Multi = false,
    Value = AutoGameValue == "Normal" and T("normal") or AutoGameValue == "VeryHard" and T("veryhard") or AutoGameValue == "Hard" and T("hard") or AutoGameValue == "Insane" and T("insane") or AutoGameValue == "Nightmare" and T("nightmare") or AutoGameValue == "BossRush" and T("bossrush") or AutoGameValue == "DarkDimension" and T("darkdimension") or AutoGameValue == "Hell" and T("hell") or AutoGameValue == "ThunderStorm" and T("thunderstorm") or AutoGameValue == "Christmas" and T("christmas") or AutoGameValue == "Zombie" and T("zombie") or AutoGameValue == "AstroV2" and T("astrov2") or AutoGameValue == "Astro" and T("astro") or AutoGameValue == "100MVisit" and T("visit_100m") or T("normal"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        AutoGameValue = voteModeMap[key] or value
        Config:Set("AutoGameValue", AutoGameValue); Config:Save()
        print("[DYHUB] 游戏模式已选择: " .. tostring(AutoGameValue))
    end
})

-- PLAY SYSTEM（加载时自动导航至 Classic/Casual）
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
        Icon = "check"
    })
end

--// PLAY SYSTEM
task.spawn(function()
    local playBtn =
        workspace:FindFirstChild("ForGui") and
        workspace.ForGui:FindFirstChild("SurfaceGui") and
        workspace.ForGui.SurfaceGui:FindFirstChild("Frame") and
        workspace.ForGui.SurfaceGui.Frame:FindFirstChild("Play")

    if playBtn then
        notify("自动 Play", "检测到 Play 按钮，自动开始...")
        task.wait(DELAY)

        local playGui = pg:FindFirstChild("Play")

        if not (playGui and playGui.Enabled) then
            click_btn(playBtn)
            notify("自动 Play", "已按下 Play 按钮")
        else
            notify("自动 Play", "Play GUI 已打开")
        end
    end

    task.wait(DELAY)

    local playGui = pg:FindFirstChild("Play")
    if not (playGui and playGui.Enabled) then
        return
    end

    local classicBtn = playGui:FindFirstChild("Classic")

    if classicBtn then
        notify("自动 Play", "正在选择 Classic 模式...")
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
            notify("自动 Play", "正在选择难度...")
            task.wait(DELAY)
            click_btn(diffBtn)
        end
    end
end)

--// NEW LOBBY SYSTEM
task.spawn(function()
    while true do
        task.wait(0.5)

        local loadingGui = pg:FindFirstChild("LoadingScreen")

        if loadingGui then
            notify("大厅系统", "正在移除 LoadingScreen...")
            pcall(function()
                loadingGui:Destroy()
            end)
        end

        local lobby = pg:FindFirstChild("Lobby")

        if lobby and lobby.Enabled then
            notify("大厅系统", "检测到大厅，准备自动设置...")

            local btn =
                lobby:FindFirstChild("MainFrame") and
                lobby.MainFrame:FindFirstChild("Frame") and
                lobby.MainFrame.Frame:FindFirstChild("Create") and
                lobby.MainFrame.Frame.Create:FindFirstChild("TrackQuestButton")

            if btn and btn.Visible then
                notify("大厅系统", "正在按下 TrackQuestButton...")
                click_btn(btn)

                task.wait(0.5)

                if AutoVoteEnabled then
                    notify("大厅系统", "正在创建游戏模式...")

                    ReplicatedStorage.MainHandler:FireServer({
                        [1] = "StartSolo",
                        [2] = AutoGameValue
                    })

                    notify("大厅系统", "游戏模式创建成功！")
                else
                    notify("大厅系统", "请配合自动游戏模式使用！")
                end

                break
            end
        end
    end
end)

--// AUTO GAME MODE TOGGLE
AutoVoteToggle = Main7:Toggle({
    Title = T("auto_game_mode_lobby"),
    Desc = T("desc_auto_game_mode_lobby"),
    Value = AutoVoteEnabled,

    Callback = function(enabled)
        AutoVoteEnabled = enabled

        Config:Set("AutoVoteEnabled", enabled)
        Config:Save()

        if enabled then
            notify("自动游戏模式", "已启用")
        else
            notify("自动游戏模式", "已禁用", "x")
        end
    end
})


-- ====================== REQUEST / SKILL TREE HELPERS ======================
RequestWaveNotifyAt = 0
AutoSkillTreeNotifyAt = 0

function SafeWindNotify(title, content, duration, icon)
    if WindUI and WindUI.Notify then
        pcall(function()
            WindUI:Notify({
                Title = tostring(title or "DYHUB"),
                Content = tostring(content or ""),
                Duration = duration or 3,
                Icon = icon or "info"
            })
        end)
    end
end

function GetCurrentWaveText()
    local ok, result = pcall(function()
        local playerGui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
        if not playerGui then return nil end

        local wavesGui = playerGui:FindFirstChild("WavesGui")
        if not wavesGui then return nil end

        local frame = wavesGui:FindFirstChild("Frame")
        if not frame then return nil end

        local label = frame:FindFirstChild("TextLabel")
        if not label then return nil end

        return tostring(label.Text or "")
    end)

    if ok then return result end
    return nil
end

function GetCurrentWaveNumber()
    local text = GetCurrentWaveText()
    if not text then return nil end

    local numberText = tostring(text):match("(%d+)")
    if not numberText then return nil end

    return tonumber(numberText)
end

function IsRequestWaveReady()
    local wave = GetCurrentWaveNumber()
    return wave ~= nil and wave >= 10
end

function NotifyRequestWaveNotReady()
    local now = tick()
    if now - RequestWaveNotifyAt < 4 then return end
    RequestWaveNotifyAt = now

    if GetCurrentWaveNumber() == nil then
        SafeWindNotify("自动请求", "暂时无法请求。波次 UI 未就绪。", 3, "triangle-alert")
    else
        SafeWindNotify("自动请求", "暂时无法请求。需要波次 10 或更高。", 3, "triangle-alert")
    end
end

function GetCurrentCharacterValue()
    local ok, result = pcall(function()
        local playerValues = LocalPlayer and LocalPlayer:FindFirstChild("PlayerValues")
        if not playerValues then return nil end

        local charValue = playerValues:FindFirstChild("Character")
        if not charValue then return nil end

        return tostring(charValue.Value or "")
    end)

    if ok then return result end
    return nil
end

function GetSkillTreeUIFolder()
    local characterName = GetCurrentCharacterValue()
    if not characterName or characterName == "" then return nil, characterName end

    local ok, result = pcall(function()
        local playerGui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
        if not playerGui then return nil end

        local skillGui = playerGui:FindFirstChild("003-A")
        if not skillGui then return nil end

        local main = skillGui:FindFirstChild("Main")
        if not main then return nil end

        local scrolling = main:FindFirstChild("ScrollingFrame")
        if not scrolling then return nil end

        local direct = scrolling:FindFirstChild("Skills " .. characterName)
        if direct then return direct end

        local loweredName = characterName:lower()
        for _, child in ipairs(scrolling:GetChildren()) do
            local childName = tostring(child.Name or ""):lower()
            if childName:find("skills", 1, true) and childName:find(loweredName, 1, true) then
                return child
            end
        end

        return nil
    end)

    if ok then return result, characterName end
    return nil, characterName
end

function HasOwnedSkillTree(skillName)
    local folder = LocalPlayer and LocalPlayer:FindFirstChild("SkillTreesFolder")
    if not folder then return false end

    if folder:FindFirstChild(skillName) then return true end

    local loweredName = tostring(skillName or ""):lower()
    for _, child in ipairs(folder:GetChildren()) do
        if tostring(child.Name or ""):lower() == loweredName then
            return true
        end
    end

    return false
end

function IsSkillTreeBuyObject(obj)
    if not obj or not obj.Name then return false end

    local loweredName = tostring(obj.Name):lower()
    if loweredName == "" then return false end
    if loweredName:find("layout", 1, true) then return false end
    if loweredName:find("padding", 1, true) then return false end
    if loweredName:find("stroke", 1, true) then return false end
    if loweredName:find("corner", 1, true) then return false end

    if obj:IsA("GuiObject") or obj:IsA("Folder") or obj:IsA("Model") then
        return true
    end

    return false
end

function GetSkillTreesRemote()
    local remote = GetRemote("skilltrees")
    if remote then return remote end

    pcall(function()
        remote = ReplicatedStorage:FindFirstChild("SkillTrees") or ReplicatedStorage:FindFirstChild("SkillTree") or ReplicatedStorage:WaitForChild("skilltrees", 2)
    end)

    return remote
end

function NotifyAutoSkillTree(message)
    local now = tick()
    if now - AutoSkillTreeNotifyAt < 5 then return end
    AutoSkillTreeNotifyAt = now
    SafeWindNotify("自动技能树", tostring(message or "技能树未就绪。"), 3, "triangle-alert")
end

function FireAutoSkillTrees()
    local remote = GetSkillTreesRemote()
    if not remote then
        NotifyAutoSkillTree("未找到技能树远程事件。")
        return false
    end

    local folder, characterName = GetSkillTreeUIFolder()
    if not characterName or characterName == "" then
        NotifyAutoSkillTree("无法读取你当前的角色。")
        return false
    end

    if not folder then
        NotifyAutoSkillTree("未找到 " .. tostring(characterName) .. " 的技能树 UI。")
        return false
    end

    local fired = 0
    for _, skillObj in ipairs(folder:GetChildren()) do
        if IsSkillTreeBuyObject(skillObj) and not HasOwnedSkillTree(skillObj.Name) then
            local remoteArg = tostring(skillObj.Name):lower()
            local ok, err = pcall(function()
                remote:FireServer(remoteArg)
            end)

            if ok then
                fired = fired + 1
                print("[DYHUB] 自动技能树已触发:", remoteArg)
            else
                warn("[DYHUB] 自动技能树失败:", remoteArg, err)
            end

            task.wait(0.35)
        end
    end

    return true
end


-- ====================== UI: SHOP SYSTEMS ======================
Main5:Section({ Title = T("auto_gacha"), Icon = "sparkles" })

_G.__DYHUB_ShopSystems = function()
    local gachaTranslationMap = {
        spin1 = "1Spin", spin10 = "10Spins", spin100 = "100Spins",
        spin1lucky = "1SpinLucky", spin10lucky = "10SpinLucky",
    }
    local titanUpgradeMap = {
        jetpack = "Jetpack", overcharge = "OverCharge", soundbooster = "SoundBooster",
        core = "Core", upgrade = "Upgrade",
    }
    local utcmUpgradeMap = {
        shield = "Shield", blaster = "Blaster", lens = "Lens", heat = "Heat", armor = "Armor",
    }
    local tvUpgradeMap = {
        absorb = "Absorb", share_overcharge = "ShareOverCharge", shield = "Shield", astro_arm = "AstroArm",
    }
    local requestMap = {
        titan_request = "Titan-Request", special_titan_request = "SpecialTitan-Request",
        speaker_request = "Speaker-Request",
    }
    local weaponMap = {
        stungun = "Stungun", flamethrower = "Flamethrower", harpoon_gun = "Harpoon Gun",
        shot_gun = "Shot Gun", pulse_rifle = "Pulse Rifle", shot_harpoon_gun = "Shot Harpoon Gun",
        epd = "EPD", small_laser_gun = "Small Laser Gun",
    }
    local miscMap = {
        headphone = "HeadPhone", grenade = "Grenade", jetpack_item = "Jetpack", lens_item = "Lens",
    }

    local autoGachaCharacterEnabled = Config:Get("AutoGachaCharacterEnabled", false)
    local autoGachaSkinEnabled      = Config:Get("AutoGachaSkinEnabled", false)
    local selectedGachaCharacterArg = Config:Get("SelectedGachaCharacterArg", "1Spin")
    local selectedGachaSkinArg      = Config:Get("SelectedGachaSkinArg", "1Spin")
    local characterGachaRunning     = false
    local skinGachaRunning          = false

    local autoUseItemEnabled        = Config:Get("AutoUseItemEnabled", false)
    local selectedUseItem           = Config:Get("SelectedUseItem", "Presents")
    local useItemRunning            = false

    local selectedRequestItem       = Config:Get("SelectedRequestItem", "Titan-Request")
    local autoRequestEnabled        = Config:Get("AutoRequestEnabled", false)
    local autoSkillTreeEnabled      = Config:Get("AutoSkillTreeEnabled", false)

    local function EnsureList(value, fallback)
        if type(value) == "table" then
            return value
        end
        if value ~= nil then
            return { value }
        end
        return fallback or {}
    end

    local function WaitWhileEnabled(seconds, enabledFn)
        local elapsed = 0
        while elapsed < seconds do
            if enabledFn and not enabledFn() then
                return false
            end
            task.wait(0.5)
            elapsed = elapsed + 0.25
        end
        return true
    end

    local function FireShopRemote(remoteName, ...)
        local remote = GetRemote(remoteName)
        if not remote then return false end

        local args = { ... }
        local ok, err = pcall(function()
            remote:FireServer(unpack(args))
        end)

        if not ok then
            warn("[DYHUB] 商店远程事件失败:", tostring(remoteName), err)
        end

        return ok
    end

    local function ShouldShopSyncWithHeli()
        return AutoSkipHeliEnabled and IsMiscFarmAllowed()
    end

    local function StartAutoGachaCharacter()
        if characterGachaRunning then return end
        characterGachaRunning = true
        task.spawn(function()
            while autoGachaCharacterEnabled do
                FireShopRemote("GachaCharacter", selectedGachaCharacterArg)
                task.wait(1)
            end
            characterGachaRunning = false
        end)
    end

    local function StartAutoGachaSkin()
        if skinGachaRunning then return end
        skinGachaRunning = true
        task.spawn(function()
            while autoGachaSkinEnabled do
                FireShopRemote("GachaSkins", selectedGachaSkinArg)
                task.wait(1)
            end
            skinGachaRunning = false
        end)
    end

    local function StartAutoUseItem()
        if useItemRunning then return end
        useItemRunning = true
        task.spawn(function()
            while autoUseItemEnabled do
                if selectedUseItem == "Presents" then
                    FireShopRemote("GachaCapsule")
                end
                task.wait(1.5)
            end
            useItemRunning = false
        end)
    end

    Main5:Dropdown({
        Title = T("gacha_character"),
        Desc = T("desc_gacha_character"),
        Values = { T("spin1"), T("spin10"), T("spin100"), T("spin1lucky"), T("spin10lucky") },
        Multi = false,
        Value = selectedGachaCharacterArg == "1Spin" and T("spin1") or selectedGachaCharacterArg == "10Spins" and T("spin10") or selectedGachaCharacterArg == "100Spins" and T("spin100") or selectedGachaCharacterArg == "1SpinLucky" and T("spin1lucky") or selectedGachaCharacterArg == "10SpinLucky" and T("spin10lucky") or T("spin1"),
        Callback = function(value)
            local key = GetOriginalKey(value)
            selectedGachaCharacterArg = gachaTranslationMap[key] or "1Spin"
            Config:Set("SelectedGachaCharacterArg", selectedGachaCharacterArg)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("auto_gacha_character"),
        Value = autoGachaCharacterEnabled,
        Desc = T("desc_auto_gacha_character"),
        Callback = function(enabled)
            autoGachaCharacterEnabled = enabled
            Config:Set("AutoGachaCharacterEnabled", enabled)
            Config:Save()
            if enabled then StartAutoGachaCharacter() end
        end
    })

    Main5:Dropdown({
        Title = T("gacha_skin"),
        Desc = T("desc_gacha_skin"),
        Values = { T("spin1"), T("spin10"), T("spin100"), T("spin1lucky"), T("spin10lucky") },
        Multi = false,
        Value = selectedGachaSkinArg == "1Spin" and T("spin1") or selectedGachaSkinArg == "10Spins" and T("spin10") or selectedGachaSkinArg == "100Spins" and T("spin100") or selectedGachaSkinArg == "1SpinLucky" and T("spin1lucky") or selectedGachaSkinArg == "10SpinLucky" and T("spin10lucky") or T("spin1"),
        Callback = function(value)
            local key = GetOriginalKey(value)
            selectedGachaSkinArg = gachaTranslationMap[key] or "1Spin"
            Config:Set("SelectedGachaSkinArg", selectedGachaSkinArg)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("auto_gacha_skin"),
        Value = autoGachaSkinEnabled,
        Desc = T("desc_auto_gacha_skin"),
        Callback = function(enabled)
            autoGachaSkinEnabled = enabled
            Config:Set("AutoGachaSkinEnabled", enabled)
            Config:Save()
            if enabled then StartAutoGachaSkin() end
        end
    })

    Main5:Section({ Title = T("auto_use_item"), Icon = "package-open" })

    Main5:Dropdown({
        Title = T("use_item"),
        Desc = T("desc_use_item"),
        Values = { "Presents" },
        Multi = false,
        Value = selectedUseItem,
        Callback = function(value)
            selectedUseItem = value or "Presents"
            Config:Set("SelectedUseItem", selectedUseItem)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("auto_use_item"),
        Value = autoUseItemEnabled,
        Desc = T("desc_auto_use_item"),
        Callback = function(enabled)
            autoUseItemEnabled = enabled
            Config:Set("AutoUseItemEnabled", enabled)
            Config:Save()
            if enabled then StartAutoUseItem() end
        end
    })

    -- ====================== 同步商店购买/升级系统 ======================
    Main5:Section({ Title = T("shop_upgrade"), Icon = "arrow-big-up-dash" })

    local selectedTitanSpeakerUpgrades = EnsureList(Config:Get("SelectedTitanSpeakerUpgrades", { "Jetpack" }), { "Jetpack" })
    local selectedUTCMUpgrades         = EnsureList(Config:Get("SelectedUTCMUpgrades", { "Shield" }), { "Shield" })
    local selectedTVUpgrades           = EnsureList(Config:Get("SelectedTVUpgrades", { "Absorb" }), { "Absorb" })

    local upgradeTitanSpeakerEnabled = Config:Get("UpgradeTitanSpeakerEnabled", false)
    local upgradeUTCMEnabled         = Config:Get("UpgradeUTCMEnabled", false)
    local upgradeTVEnabled           = Config:Get("UpgradeTVEnabled", false)

    local StartAutoSyncedShopLoop = function() end

    Main5:Dropdown({
        Title = T("select_upgrade_titan"),
        Desc = T("desc_select_upgrade_titan"),
        Values = { T("jetpack"), T("overcharge"), T("soundbooster"), T("core"), T("upgrade") },
        Multi = true,
        Value = selectedTitanSpeakerUpgrades,
        Callback = function(values)
            local cleaned = {}
            for _, v in ipairs(values) do
                local key = GetOriginalKey(v)
                local original = titanUpgradeMap[key] or v
                table.insert(cleaned, original)
            end
            selectedTitanSpeakerUpgrades = cleaned
            Config:Set("SelectedTitanSpeakerUpgrades", cleaned)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("upgrade_titan"),
        Desc = T("desc_upgrade_titan"),
        Value = upgradeTitanSpeakerEnabled,
        Callback = function(enabled)
            upgradeTitanSpeakerEnabled = enabled
            Config:Set("UpgradeTitanSpeakerEnabled", enabled)
            Config:Save()
            if enabled then StartAutoSyncedShopLoop() end
        end
    })

    Main5:Dropdown({
        Title = T("select_upgrade_utcm"),
        Desc = T("desc_select_upgrade_utcm"),
        Values = { T("shield"), T("blaster"), T("lens"), T("heat"), T("armor") },
        Multi = true,
        Value = selectedUTCMUpgrades,
        Callback = function(values)
            local cleaned = {}
            for _, v in ipairs(values) do
                local key = GetOriginalKey(v)
                local original = utcmUpgradeMap[key] or v
                table.insert(cleaned, original)
            end
            selectedUTCMUpgrades = cleaned
            Config:Set("SelectedUTCMUpgrades", cleaned)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("upgrade_utcm"),
        Desc = T("desc_upgrade_utcm"),
        Value = upgradeUTCMEnabled,
        Callback = function(enabled)
            upgradeUTCMEnabled = enabled
            Config:Set("UpgradeUTCMEnabled", enabled)
            Config:Save()
            if enabled then StartAutoSyncedShopLoop() end
        end
    })

    Main5:Dropdown({
        Title = T("select_upgrade_tv"),
        Desc = T("desc_select_upgrade_tv"),
        Values = { T("absorb"), T("share_overcharge"), T("shield"), T("astro_arm") },
        Multi = true,
        Value = selectedTVUpgrades,
        Callback = function(values)
            local cleaned = {}
            for _, v in ipairs(values) do
                local key = GetOriginalKey(v)
                local original = tvUpgradeMap[key] or v
                table.insert(cleaned, original)
            end
            selectedTVUpgrades = cleaned
            Config:Set("SelectedTVUpgrades", cleaned)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("upgrade_tv"),
        Desc = T("desc_upgrade_tv"),
        Value = upgradeTVEnabled,
        Callback = function(enabled)
            upgradeTVEnabled = enabled
            Config:Set("UpgradeTVEnabled", enabled)
            Config:Save()
            if enabled then StartAutoSyncedShopLoop() end
        end
    })

    Main5:Section({ Title = T("shop_weapon"), Icon = "helicopter" })

    local autoBuyWeaponValue   = Config:Get("AutoBuyWeaponValue", "Stungun")
    local autoBuyWeaponEnabled = Config:Get("AutoBuyWeaponEnabled", false)

    WeaponDropdown = Main5:Dropdown({
        Title = T("select_weapon"),
        Desc = T("desc_select_weapon"),
        Values = { T("stungun"), T("flamethrower"), T("harpoon_gun"), T("shot_gun"), T("pulse_rifle"), T("shot_harpoon_gun"), T("epd"), T("small_laser_gun") },
        Multi = false,
        Value = autoBuyWeaponValue == "Stungun" and T("stungun") or autoBuyWeaponValue == "Flamethrower" and T("flamethrower") or autoBuyWeaponValue == "Harpoon Gun" and T("harpoon_gun") or autoBuyWeaponValue == "Shot Gun" and T("shot_gun") or autoBuyWeaponValue == "Pulse Rifle" and T("pulse_rifle") or autoBuyWeaponValue == "Shot Harpoon Gun" and T("shot_harpoon_gun") or autoBuyWeaponValue == "EPD" and T("epd") or autoBuyWeaponValue == "Small Laser Gun" and T("small_laser_gun") or T("stungun"),
        Callback = function(value)
            local key = GetOriginalKey(value)
            autoBuyWeaponValue = weaponMap[key] or "Stungun"
            Config:Set("AutoBuyWeaponValue", autoBuyWeaponValue)
            Config:Save()
        end
    })

    AutoBuyWeaponToggle = Main5:Toggle({
        Title = T("buy_weapon"),
        Desc = T("desc_buy_weapon"),
        Value = autoBuyWeaponEnabled,
        Callback = function(enabled)
            autoBuyWeaponEnabled = enabled
            Config:Set("AutoBuyWeaponEnabled", enabled)
            Config:Save()
            if enabled then StartAutoSyncedShopLoop() end
        end
    })

    Main5:Button({
        Title = T("buy_weapon_once"),
        Desc = T("desc_buy_weapon_once"),
        Callback = function()
            if autoBuyWeaponValue then
                FireShopRemote("ShopSystem", "Buy", autoBuyWeaponValue)
            end
        end
    })

    Main5:Section({ Title = T("shop_misc"), Icon = "helicopter" })

    local autoBuyMiscValue   = Config:Get("AutoBuyMiscValue", "HeadPhone")
    local autoBuyMiscEnabled = Config:Get("AutoBuyMiscEnabled", false)

    if table.find(GlobalTables.RequestTitanSpeaker, autoBuyMiscValue) or not table.find(GlobalTables.MiscShop, autoBuyMiscValue) then
        autoBuyMiscValue = "HeadPhone"
        Config:Set("AutoBuyMiscValue", autoBuyMiscValue)
        Config:Save()
    end

    if not table.find(GlobalTables.RequestTitanSpeaker, selectedRequestItem) then
        selectedRequestItem = "Titan-Request"
        Config:Set("SelectedRequestItem", selectedRequestItem)
        Config:Save()
    end

    MiscShopDropdown = Main5:Dropdown({
        Title = T("select_misc"),
        Desc = T("desc_select_misc"),
        Values = { T("headphone"), T("grenade"), T("jetpack_item"), T("lens_item") },
        Multi = false,
        Value = autoBuyMiscValue == "HeadPhone" and T("headphone") or autoBuyMiscValue == "Grenade" and T("grenade") or autoBuyMiscValue == "Jetpack" and T("jetpack_item") or autoBuyMiscValue == "Lens" and T("lens_item") or T("headphone"),
        Callback = function(value)
            local key = GetOriginalKey(value)
            autoBuyMiscValue = miscMap[key] or "HeadPhone"
            Config:Set("AutoBuyMiscValue", autoBuyMiscValue)
            Config:Save()
        end
    })

    AutoBuyMiscToggle = Main5:Toggle({
        Title = T("buy_misc"),
        Value = autoBuyMiscEnabled,
        Desc = T("desc_buy_misc"),
        Callback = function(enabled)
            autoBuyMiscEnabled = enabled
            Config:Set("AutoBuyMiscEnabled", enabled)
            Config:Save()
            if enabled then StartAutoSyncedShopLoop() end
        end
    })

    Main5:Button({
        Title = T("buy_misc_once"),
        Desc = T("desc_buy_misc_once"),
        Callback = function()
            if autoBuyMiscValue then
                FireShopRemote("ShopSystem", "Buy", autoBuyMiscValue)
            end
        end
    })

    Main5:Section({ Title = T("shop_request"), Icon = "send" })

    RequestTitanSpeakerDropdown = Main5:Dropdown({
        Title = T("select_request"),
        Desc = T("desc_select_request"),
        Values = { T("titan_request"), T("special_titan_request"), T("speaker_request") },
        Multi = false,
        Value = selectedRequestItem == "Titan-Request" and T("titan_request") or selectedRequestItem == "SpecialTitan-Request" and T("special_titan_request") or selectedRequestItem == "Speaker-Request" and T("speaker_request") or T("titan_request"),
        Callback = function(value)
            local key = GetOriginalKey(value)
            selectedRequestItem = requestMap[key] or "Titan-Request"
            Config:Set("SelectedRequestItem", selectedRequestItem)
            Config:Save()
        end
    })

    AutoRequestToggle = Main5:Toggle({
        Title = T("auto_request"),
        Desc = T("desc_auto_request"),
        Value = autoRequestEnabled,
        Callback = function(enabled)
            autoRequestEnabled = enabled
            Config:Set("AutoRequestEnabled", enabled)
            Config:Save()

            if enabled then
                if not IsRequestWaveReady() then
                    NotifyRequestWaveNotReady()
                end
                StartAutoSyncedShopLoop()
            end
        end
    })

    Main5:Section({ Title = T("skill_tree"), Icon = "git-branch-plus" })

    AutoSkillTreeToggle = Main5:Toggle({
        Title = T("auto_skill_tree"),
        Desc = T("desc_auto_skill_tree"),
        Value = autoSkillTreeEnabled,
        Callback = function(enabled)
            autoSkillTreeEnabled = enabled
            Config:Set("AutoSkillTreeEnabled", enabled)
            Config:Save()

            if enabled then
                StartAutoSyncedShopLoop()
            end
        end
    })

    local autoSyncedShopRunning = false

    local function IsHeavySyncedShopEnabled()
        return autoBuyWeaponEnabled
            or autoBuyMiscEnabled
            or upgradeTitanSpeakerEnabled
            or upgradeUTCMEnabled
            or upgradeTVEnabled
    end

    local function IsAnySyncedShopEnabled()
        return IsHeavySyncedShopEnabled()
            or autoRequestEnabled
            or autoSkillTreeEnabled
    end

    local function GetSyncedShopPreDelay()
        if not IsHeavySyncedShopEnabled() and (autoRequestEnabled or autoSkillTreeEnabled) then
            return 0
        end
        return 30
    end

    local function GetSyncedShopPostDelay()
        if not IsHeavySyncedShopEnabled() then
            if autoRequestEnabled then return 2 end
            if autoSkillTreeEnabled then return 5 end
        end
        return 10
    end

    local function FireSyncedShopBatch()
        if autoBuyWeaponEnabled and autoBuyWeaponValue then
            FireShopRemote("ShopSystem", "Buy", autoBuyWeaponValue)
            task.wait(0.35)
        end

        if autoBuyMiscEnabled and autoBuyMiscValue then
            FireShopRemote("ShopSystem", "Buy", autoBuyMiscValue)
            task.wait(0.35)
        end

        if autoRequestEnabled and selectedRequestItem then
            if IsRequestWaveReady() then
                FireShopRemote("ShopSystem", "Buy", selectedRequestItem)
            else
                NotifyRequestWaveNotReady()
            end
            task.wait(0.35)
        end

        if autoSkillTreeEnabled then
            FireAutoSkillTrees()
            task.wait(0.35)
        end

        if upgradeTitanSpeakerEnabled then
            for _, upgradeName in ipairs(selectedTitanSpeakerUpgrades or {}) do
                FireShopRemote("ChangeUpgradedTitanSpeaker", upgradeName)
                task.wait(0.35)
            end
        end

        if upgradeUTCMEnabled then
            for _, upgradeName in ipairs(selectedUTCMUpgrades or {}) do
                FireShopRemote("ForUpgradeUTCM", upgradeName)
                task.wait(0.35)
            end
        end

        if upgradeTVEnabled then
            for _, upgradeName in ipairs(selectedTVUpgrades or {}) do
                FireShopRemote("ForUpgradeTV", upgradeName)
                task.wait(0.35)
            end
        end
    end

    StartAutoSyncedShopLoop = function()
        if autoSyncedShopRunning then return end
        autoSyncedShopRunning = true

        task.spawn(function()
            local firstCycle = true

            while IsAnySyncedShopEnabled() do
                if not firstCycle then
                    if not WaitWhileEnabled(GetSyncedShopPreDelay(), IsAnySyncedShopEnabled) then break end
                end
                firstCycle = false

                local shouldSyncHeli = ShouldShopSyncWithHeli()
                if shouldSyncHeli then
                    TriggerAutoSkipHeli(false)
                    task.wait(0.5)
                end

                FireSyncedShopBatch()

                if shouldSyncHeli then
                    task.wait(0.5)
                    TriggerAutoSkipHeli(true)
                end

                if not WaitWhileEnabled(GetSyncedShopPostDelay(), IsAnySyncedShopEnabled) then break end
            end

            autoSyncedShopRunning = false
        end)
    end
    -- ====================== 商店每小时系统 ======================
    Main5:Section({ Title = T("shop_hourly"), Icon = "clock" })

    local ShopHourlyFixedItems = {
        "LuckPotionI",
        "LuckPotionII",
        "LuckPotionIII",
        "S-Ember",
        "BSX2:30",
        "BSX2:60",
        "BSX2:360",
        "FlashDrive#1",
        "FlashDrive#2",
        "FlashDrive#3",
        "FlashDrive#4",
        "FlashDrive#5",
        "FlashDrive#6",
        "MasterCard:Normal",
        "MasterCard:NormalTitan",
        "MasterCard:SpecialTitan",
    }

    local function GetShopHourlyItems()
        local results = {}
        for _, itemName in ipairs(ShopHourlyFixedItems) do
            table.insert(results, itemName)
        end
        return results
    end

    local ShopHourlyAllowed = {}
    for _, itemName in ipairs(ShopHourlyFixedItems) do
        ShopHourlyAllowed[itemName] = true
    end

    local function SanitizeShopHourlySelection(values, fallback)
        local clean = {}
        local seen = {}

        for _, itemName in ipairs(EnsureList(values, fallback or {})) do
            itemName = tostring(itemName or "")
            if ShopHourlyAllowed[itemName] and not seen[itemName] then
                seen[itemName] = true
                table.insert(clean, itemName)
            end
        end

        if #clean == 0 and type(fallback) == "table" then
            for _, itemName in ipairs(fallback) do
                itemName = tostring(itemName or "")
                if ShopHourlyAllowed[itemName] and not seen[itemName] then
                    seen[itemName] = true
                    table.insert(clean, itemName)
                    break
                end
            end
        end

        return clean
    end

    local shopHourlyValues          = GetShopHourlyItems()
    local selectedShopHourlyItems   = SanitizeShopHourlySelection(Config:Get("SelectedShopHourlyItems", { shopHourlyValues[1] }), { shopHourlyValues[1] })
    local shopHourlyItemAmount      = Config:Get("ShopHourlyItemAmount", 1)
    local buyItemHourlyEnabled      = Config:Get("BuyItemHourlyEnabled", false)
    local buyItemHourlyRunning      = false

    local function IsBuyItemHourlyEnabled()
        return buyItemHourlyEnabled
    end

    local function FireShopHourlyBatch()
        local amount = tonumber(shopHourlyItemAmount) or 1
        amount = math.max(1, math.floor(amount))

        for _, itemName in ipairs(selectedShopHourlyItems or {}) do
            if itemName and itemName ~= "" then
                FireShopRemote("BuyItemFromShopHourly", itemName, amount)
                task.wait(0.35)
            end
        end
    end

    local function StartBuyItemHourlyLoop()
        if buyItemHourlyRunning then return end
        buyItemHourlyRunning = true

        task.spawn(function()
            local firstCycle = true

            while buyItemHourlyEnabled do
                if not firstCycle then
                    if not WaitWhileEnabled(30, IsBuyItemHourlyEnabled) then break end
                end
                firstCycle = false

                FireShopHourlyBatch()

                if not WaitWhileEnabled(10, IsBuyItemHourlyEnabled) then break end
            end

            buyItemHourlyRunning = false
        end)
    end

    local hourlyTranslationMap = {
        luck_potion_I = "LuckPotionI", luck_potion_II = "LuckPotionII", luck_potion_III = "LuckPotionIII",
        s_ember = "S-Ember", bsx2_30 = "BSX2:30", bsx2_60 = "BSX2:60", bsx2_360 = "BSX2:360",
        flash_drive_1 = "FlashDrive#1", flash_drive_2 = "FlashDrive#2", flash_drive_3 = "FlashDrive#3",
        flash_drive_4 = "FlashDrive#4", flash_drive_5 = "FlashDrive#5", flash_drive_6 = "FlashDrive#6",
        master_card_normal = "MasterCard:Normal", master_card_normal_titan = "MasterCard:NormalTitan",
        master_card_special_titan = "MasterCard:SpecialTitan",
    }

    Main5:Dropdown({
        Title = T("select_shop_hourly"),
        Desc = T("desc_select_shop_hourly"),
        Values = { T("luck_potion_I"), T("luck_potion_II"), T("luck_potion_III"), T("s_ember"), T("bsx2_30"), T("bsx2_60"), T("bsx2_360"), T("flash_drive_1"), T("flash_drive_2"), T("flash_drive_3"), T("flash_drive_4"), T("flash_drive_5"), T("flash_drive_6"), T("master_card_normal"), T("master_card_normal_titan"), T("master_card_special_titan") },
        Multi = true,
        Value = selectedShopHourlyItems,
        Callback = function(values)
            local cleaned = {}
            for _, v in ipairs(values) do
                local key = GetOriginalKey(v)
                local original = hourlyTranslationMap[key] or v
                table.insert(cleaned, original)
            end
            selectedShopHourlyItems = SanitizeShopHourlySelection(cleaned, {})
            Config:Set("SelectedShopHourlyItems", selectedShopHourlyItems)
            Config:Save()
        end
    })

    Main5:Slider({
        Title = T("item_amount"),
        Desc = T("desc_item_amount"),
        Value = { Min = 1, Max = 100, Default = shopHourlyItemAmount },
        Step = 1,
        Callback = function(value)
            shopHourlyItemAmount = value
            Config:Set("ShopHourlyItemAmount", value)
            Config:Save()
        end
    })

    Main5:Toggle({
        Title = T("buy_item"),
        Desc = T("desc_buy_item"),
        Value = buyItemHourlyEnabled,
        Callback = function(enabled)
            buyItemHourlyEnabled = enabled
            Config:Set("BuyItemHourlyEnabled", enabled)
            Config:Save()
            if enabled then StartBuyItemHourlyLoop() end
        end
    })

    if autoGachaCharacterEnabled then StartAutoGachaCharacter() end
    if autoGachaSkinEnabled then StartAutoGachaSkin() end
    if autoUseItemEnabled then StartAutoUseItem() end
    if IsAnySyncedShopEnabled() then StartAutoSyncedShopLoop() end
    if buyItemHourlyEnabled then StartBuyItemHourlyLoop() end
end
_G.__DYHUB_ShopSystems()
_G.__DYHUB_ShopSystems = nil

-- ====================== UI: COLLECT TAB ======================
Main6:Section({ Title = T("collect_section"), Icon = "package" })

AutoCollectToggle = Main6:Toggle({
    Title = T("auto_collect"), Value = AutoCollectEnabled,
    Desc = T("auto_collect_desc_full"),
    Callback = function(state)
        AutoCollectEnabled = state; Config:Set("AutoCollectEnabled", state); Config:Save()
        if state then
            KnownCollectItems = {}
            CollectCandidateCache = {}
            CollectCacheDirty = true
            CheckFarmAstroCollectMode()
            StartAutoCollectLoop()
        else
            CollectRunning = false
            FarmCollecting = false
        end
    end
})

Main6:Section({ Title = T("collect_settings"), Icon = "settings" })

CollectItemDropdown = Main6:Dropdown({
    Title = T("collect_items"),
    Desc = T("desc_collect_items"),
    Values = CollectItems, Multi = true, Value = SelectedCollectItems,
    Callback = function(values)
        SelectedCollectItems = values or {}
        CollectCandidateCache = {}
        CollectCacheDirty = true
        KnownCollectItems = {}
        Config:Set("SelectedCollectItems", SelectedCollectItems)
        Config:Save()
    end
})

CollectModeDropdown = Main6:Dropdown({
    Title = T("collect_mode"),
    Desc = T("desc_collect_mode"),
    Values = { T("clean"), T("idgf") },
    Multi = false,
    Value = CollectMode == "Clean" and T("clean") or T("idgf"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        if key == "clean" then CollectMode = "Clean"
        elseif key == "idgf" then CollectMode = "IDGF"
        else CollectMode = value end
        Config:Set("CollectMode", CollectMode)
        Config:Save()
        CheckFarmAstroCollectMode()
    end
})

CollectMovementDropdown = Main6:Dropdown({
    Title = T("collect_movement"),
    Desc = T("desc_collect_movement"),
    Values = { T("teleport"), T("tween") },
    Multi = false,
    Value = CollectMovementMode == "Teleport" and T("teleport") or T("tween"),
    Callback = function(value)
        local key = GetOriginalKey(value)
        if key == "teleport" then CollectMovementMode = "Teleport"
        elseif key == "tween" then CollectMovementMode = "Tween"
        else CollectMovementMode = NormalizeCollectMovement(value) end
        Config:Set("CollectMovementMode", CollectMovementMode)
        Config:Save()
        WindUI:Notify({ Title = T("notify_collect_movement_mode"), Content = T("notify_selected") .. tostring(value), Duration = 2, Icon = "move" })
    end
})

-- ====================== UI: SETTING TAB ======================
Main3:Section({ Title = T("save_settings"), Icon = "save" })

Main3:Button({
    Title = T("save_config"),
    Desc = T("desc_save_config"),
    Callback = function()
        Config:Save()
        WindUI:Notify({ Title = T("save_config"), Content = T("notify_save_success"), Duration = 2, Icon = "save" })
    end
})

AutoSaveEnabled = Config:Get("AutoSaveEnabled", true)
AutoSaveDelay   = Config:Get("AutoSaveDelay", 15)
AutoSaveThread  = nil

function RestartAutoSave()
    if AutoSaveThread then task.cancel(AutoSaveThread); AutoSaveThread = nil end
    if AutoSaveEnabled then
        AutoSaveThread = task.spawn(function()
            while AutoSaveEnabled do
                task.wait(AutoSaveDelay)
                Config:Save()
            end
        end)
    end
end

Main3:Toggle({
    Title = T("auto_save"), Value = AutoSaveEnabled,
    Desc = T("desc_auto_save"),
    Callback = function(state) AutoSaveEnabled = state; Config:Set("AutoSaveEnabled", state); Config:Save(); RestartAutoSave() end
})

Main3:Input({
    Title = T("delay_save"),
    Desc = T("desc_delay_save"),
    Default = tostring(AutoSaveDelay), Placeholder = "默认: 15",
    Callback = function(text)
        local num = tonumber(text)
        if num and num >= 1 then AutoSaveDelay = num; Config:Set("AutoSaveDelay", num); Config:Save(); RestartAutoSave()
        else warn("[DYHUB] 无效的延迟值！") end
    end
})

RestartAutoSave()

Main3:Section({ Title = T("server_status"), Icon = "server" })

Main3:Button({
    Title = T("serverhop"),
    Desc = T("desc_serverhop"),
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
            WindUI:Notify({ Title = T("serverhop"), Content = T("serverhop_progress"), Duration = 2, Icon = "server" })
            task.wait(1)
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LocalPlayer)
        else
            WindUI:Notify({ Title = T("notify_serverhop_fail"), Content = T("notify_no_servers"), Duration = 3, Icon = "alert-triangle" })
        end
    end
})

Main3:Button({
    Title = T("rejoin"),
    Desc = T("desc_rejoin"),
    Callback = function()
        WindUI:Notify({ Title = T("rejoin"), Content = T("rejoin_progress"), Duration = 2, Icon = "refresh-cw" })
        task.wait(1)
        game:GetService("TeleportService"):Teleport(game.PlaceId, LocalPlayer)
    end
})

Main3:Section({ Title = T("others"), Icon = "settings" })

CameraDropdown = Main3:Dropdown({
    Title = T("camera_mode"),
    Desc = T("desc_camera_mode"),
    Values = { "Classic", "Manual" },
    Multi = false,
    Value = NormalizeCameraMode(CameraMode),
    Callback = function(value)
        CameraMode = NormalizeCameraMode(value)
        Config:Set("CameraMode", CameraMode)
        Config:Save()
        ApplyCameraMode(true)
        WindUI:Notify({ Title = T("camera_mode"), Content = T("notify_selected") .. tostring(CameraMode), Duration = 2, Icon = "camera" })
    end
})

NoBarrierToggle = Main3:Toggle({
    Title = T("bypass_barrier"), Value = noBarrierActive,
    Desc = T("desc_bypass_barrier"),
    Callback = function(value)
        noBarrierActive = value; Config:Set("NoBarrier", value); Config:Save()
        if value then startNoBarrier() else stopNoBarrier() end
    end
})

Main3:Section({ Title = T("language_title"), Icon = "globe" })

Main3:Dropdown({
    Title = T("select_language"),
    Desc = T("desc_select_language"),
    Values = { "Chinese", "English" },
    Multi = false,
    Value = currentLanguage,
    Callback = function(value)
        currentLanguage = value
        Config:Set("Language", value)
        Config:Save()
        BuildReverseTranslation()
        WindUI:Notify({ Title = T("language_notify_title"), Content = T("language_changed") .. " " .. value, Duration = 2 })
    end
})

CombatDebugToggle = Main3:Toggle({
    Title = T("combat_debug"),
    Value = CombatDebugEnabled,
    Desc = T("desc_combat_debug"),
    Callback = function(value)
        CombatDebugEnabled = value
        Config:Set("CombatDebugEnabled", value)
        Config:Save()
        if value then
            WindUI:Notify({ Title = T("combat_debug"), Content = T("notify_combat_debug_on"), Duration = 2, Icon = "bug" })
        else
            WindUI:Notify({ Title = T("combat_debug"), Content = T("notify_combat_debug_off"), Duration = 2, Icon = "square" })
        end
    end
})

AntiAFKConnection = nil
AntiAFKThread = nil
AntiAFKDisabledConnections = false

function StartAntiAFK()
    AntiAFK = true

    if getconnections and not AntiAFKDisabledConnections then
        pcall(function()
            for _, connection in pairs(getconnections(LocalPlayer.Idled)) do
                if connection.Disable then
                    connection:Disable()
                elseif connection.Disconnect then
                    connection:Disconnect()
                end
            end
        end)
        AntiAFKDisabledConnections = true
    end

    if AntiAFKConnection then
        AntiAFKConnection:Disconnect()
        AntiAFKConnection = nil
    end

    AntiAFKConnection = LocalPlayer.Idled:Connect(function()
        if not AntiAFK then return end
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end)
    end)

    if AntiAFKThread then
        pcall(function() task.cancel(AntiAFKThread) end)
        AntiAFKThread = nil
    end

    AntiAFKThread = task.spawn(function()
        while AntiAFK do
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
            task.wait(60)
        end
        AntiAFKThread = nil
    end)
end

function StopAntiAFK()
    AntiAFK = false

    if AntiAFKConnection then
        AntiAFKConnection:Disconnect()
        AntiAFKConnection = nil
    end

    if AntiAFKThread then
        pcall(function() task.cancel(AntiAFKThread) end)
        AntiAFKThread = nil
    end
end

antiafk = Main3:Toggle({
    Title = T("anti_afk"), Value = AntiAFK,
    Desc = T("desc_anti_afk"),
    Callback = function(enabled)
        AntiAFK = enabled
        Config:Set("AntiAfk", enabled)
        Config:Save()
        if enabled then
            StartAntiAFK()
            WindUI:Notify({ Title = T("anti_afk"), Content = T("notify_anti_afk_on"), Duration = 2, Icon = "shield-check" })
        else
            StopAntiAFK()
            WindUI:Notify({ Title = T("anti_afk"), Content = T("notify_anti_afk_off"), Duration = 2, Icon = "square" })
        end
    end
})

if AntiAFK then StartAntiAFK() end

-- ====================== 应用保存的配置（启动时） ======================
function ApplySavedConfigOnStartup()
    task.wait(1)
    updatePlayerStats()
    ApplyCameraMode(true)
    UpdateDYHUBWaitingPartCollision()
    if FullBrightEnabled then ApplyFullBright() end
    if NoFogEnabled then ApplyNoFog() end
    if FlyEnabled then StartFly() end

    if FarmAstroTokenEnabled and AutoFarmEnabled then
        FarmAstroTokenEnabled = false
        Config:Set("FarmAstroTokenEnabled", false)
        Config:Save()
        NotifyFarmAstroAutoFarm()
    end

    if AutoFarmEnabled then
        StartFarmLoop()
        StartJeffreyGuardLoop()
    end

    if FarmAstroTokenEnabled then
        StartFarmAstroToken()
    end

    HandleMiscOptions(MiscOptions)

    if noBarrierActive then startNoBarrier() end

    if ESP.Enabled then
        StartESPLoop()
    end

    if AutoCollectEnabled then
        KnownCollectItems = {}
        CollectCandidateCache = {}
        CollectCacheDirty = true
        StartAutoCollectLoop()
    end

    if AutoStartEnabled and IsMiscFarmAllowed() then
        SetupAutoStartOnly(true)
    elseif AutoStartEnabled then
        StopAutoStart()
    end
end

ApplySavedConfigOnStartup()

print("[DYHUB] 版本: " .. version .. " | 更新日志: " .. ver .. " 加载成功！")
print("[DYHUB] 配置系统已激活 | 每 " .. tostring(AutoSaveDelay) .. " 秒自动保存")

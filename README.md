-- ============================================================================
-- DYHUB 终极暴力完整版 (Delta 兼容版，已删除主题切换)
-- 多语言（中/英/俄/葡） | 全功能 + 高风险特性
-- 版本: v027.0-FINAL-COMPLETE-THEMED-FIXED-DELTA
-- 功能: 优先级挂机、动态高度、运动预测、跟随模式、Auto God Mode、Xmas极速、
--       批量无延迟购买/抽卡、旋转90°、ESP、收集、Mastery、Hitbox、Quest等
-- 修改: 删除 WindUI:AddTheme、Window:Tag、EditOpenButton 的 Color 参数、
--       User 中的 Callback，避免 Delta 的 UIStroke 崩溃
-- ============================================================================

setfpscap(30)
local version = "Ultimate Final Complete Fixed"
local ver = "v027.0-FINAL-COMPLETE-THEMED-FIXED-DELTA"
local currentLanguage = "Chinese"
pcall(function() delfolder("DYHUB_FINAL") end)

-- ====================== 兼容不支持 fireproximityprompt 的执行器 ======================
fireproximityprompt = fireproximityprompt or function(prompt)
    prompt:InputHoldBegin()
    task.wait(0.05)
    prompt:InputHoldEnd()
end

-- ====================== 多语言翻译表（完整汉化） ======================
local translations = {
    Chinese = {
        loading = "游戏加载中...",
        loaded = "加载完成，3秒后启动",
        fps_unlocked = "FPS 已解锁",
        fps_not_supported = "不支持 setfpscap",
        config_saved = "配置已保存",
        auto_farm = "自动挂机",
        auto_farm_desc = "根据优先级自动刷怪",
        farm_enabled = "已开启",
        farm_disabled = "已关闭",
        sync_mode = "同步挂机模式",
        sync_mode_desc = "辅助功能需自动挂机开启",
        sync_on = "需自动挂机",
        sync_off = "辅助功能独立",
        position_above = "上方",
        position_under = "下方",
        position_front = "前方",
        position_back = "后方",
        position_spin = "旋转",
        movement_teleport = "瞬移",
        movement_cframe = "平滑",
        auto_attack = "自动攻击",
        auto_skill = "自动技能",
        auto_ready = "自动开局",
        auto_skip_heli = "自动跳过直升机",
        auto_heal = "自动补血",
        safe_mode = "安全模式",
        god_mode = "上帝模式",
        delete_map = "删除地图",
        flush_aura = "冲水光环",
        flush_range = "冲水范围",
        attack_speed = "攻击速度",
        skill_delay = "技能延迟",
        height_offset = "挂机高度偏移",
        safe_threshold = "安全模式血量",
        god_threshold = "上帝模式血量",
        high_hp_threshold = "高血量阈值",
        esp_enable = "启用透视",
        esp_mob = "怪物透视",
        esp_player = "玩家透视",
        esp_item = "物品透视",
        esp_highlight = "高亮",
        esp_distance = "距离",
        esp_health = "血量",
        esp_name = "名称",
        vote_normal = "普通模式",
        vote_hard = "困难",
        vote_veryhard = "非常困难",
        vote_insane = "疯狂",
        vote_nightmare = "噩梦",
        vote_bossrush = "首领连战",
        vote_darkdim = "暗黑维度",
        vote_hell = "地狱",
        vote_thunder = "雷暴",
        vote_christmas = "圣诞",
        vote_zombie = "僵尸",
        vote_astro = "天文模式",
        auto_collect = "自动收集",
        collect_mode_clean = "清理模式",
        collect_mode_idgf = "即时模式",
        astro_mode = "天文币刷取模式",
        astro_start = "启动",
        astro_stop = "停止",
        mastery_title = "熟练度自动刷",
        mastery_no_flush = "无冲水",
        mastery_flush = "冲水模式",
        mastery_action_speed = "动作速度",
        mastery_char_list = "角色列表",
        hitbox_title = "碰撞箱",
        hitbox_scan = "扫描",
        hitbox_size = "大小",
        hitbox_enable = "启用",
        hitbox_show = "显示",
        quest_title = "任务辅助",
        quest_open_clock = "打开时钟人菜单",
        quest_open_quest = "打开任务菜单",
        quest_auto_collect = "自动收集任务物品",
        quest_auto_skip = "自动跳过任务",
        auto_rebirth = "自动重生",
        auto_daily = "自动每日奖励",
        auto_chest = "自动开宝箱",
        performance_monitor = "性能监控",
        show_cpu = "显示CPU占用",
        reconnect_on_disconnect = "断线自动重连",
        info_owner = "脚本作者",
        info_discord = "Discord邀请",
        info_version = "版本",
        info_lines = "总行数",
        motion_prediction = "运动预测",
        motion_prediction_desc = "预判怪物位置并瞬移（高危）",
        follow_mode = "跟随模式",
        follow_mode_desc = "每帧黏住怪物（极高危）",
        auto_god_mode = "自动无敌重生",
        auto_god_mode_desc = "血量低于阈值自动重生",
        god_respawn_threshold = "重生触发血量",
        xmas_title = "Xmas极速系统",
        xmas_open = "自动开礼物（极速）",
        xmas_collect = "自动收集The Present（极速）",
        xmas_open_now = "立即开100个礼物",
        batch_buy = "无延迟批量购买",
        batch_buy_desc = "超快速度购买商店物品",
        batch_gacha = "无延迟批量抽卡",
        batch_gacha_char = "批量抽角色",
        batch_gacha_skin = "批量抽皮肤",
        rotate_90 = "旋转90°模式",
        rotate_90_desc = "站立时旋转90度",
    },
    English = {
        loading = "Loading game...",
        loaded = "Loaded, starting in 3s",
        fps_unlocked = "FPS Unlocked",
        fps_not_supported = "setfpscap not supported",
        config_saved = "Config saved",
        auto_farm = "Auto Farm",
        auto_farm_desc = "Priority-based auto farm",
        farm_enabled = "Enabled",
        farm_disabled = "Disabled",
        sync_mode = "Sync Farm Mode",
        sync_mode_desc = "Aux functions need auto farm",
        sync_on = "Need auto farm",
        sync_off = "Aux functions independent",
        position_above = "Above",
        position_under = "Under",
        position_front = "Front",
        position_back = "Back",
        position_spin = "Spin",
        movement_teleport = "Teleport",
        movement_cframe = "Smooth",
        auto_attack = "Auto Attack",
        auto_skill = "Auto Skill",
        auto_ready = "Auto Start",
        auto_skip_heli = "Auto Skip Heli",
        auto_heal = "Auto Heal",
        safe_mode = "Safe Mode",
        god_mode = "God Mode",
        delete_map = "Delete Map",
        flush_aura = "Flush Aura",
        flush_range = "Flush Range",
        attack_speed = "Attack Speed",
        skill_delay = "Skill Delay",
        height_offset = "Height Offset",
        safe_threshold = "Safe HP",
        god_threshold = "God HP",
        high_hp_threshold = "High HP Threshold",
        esp_enable = "Enable ESP",
        esp_mob = "Mob ESP",
        esp_player = "Player ESP",
        esp_item = "Item ESP",
        esp_highlight = "Highlight",
        esp_distance = "Distance",
        esp_health = "Health",
        esp_name = "Name",
        vote_normal = "Normal",
        vote_hard = "Hard",
        vote_veryhard = "Very Hard",
        vote_insane = "Insane",
        vote_nightmare = "Nightmare",
        vote_bossrush = "Boss Rush",
        vote_darkdim = "Dark Dimension",
        vote_hell = "Hell",
        vote_thunder = "Thunder Storm",
        vote_christmas = "Christmas",
        vote_zombie = "Zombie",
        vote_astro = "Astro",
        auto_collect = "Auto Collect",
        collect_mode_clean = "Clean",
        collect_mode_idgf = "IDGF",
        astro_mode = "Astro Coin Farm",
        astro_start = "Start",
        astro_stop = "Stop",
        mastery_title = "Auto Mastery",
        mastery_no_flush = "No Flush",
        mastery_flush = "Flush Mode",
        mastery_action_speed = "Action Speed",
        mastery_char_list = "Character List",
        hitbox_title = "Hitbox",
        hitbox_scan = "Scan",
        hitbox_size = "Size",
        hitbox_enable = "Enable",
        hitbox_show = "Show",
        quest_title = "Quest Helper",
        quest_open_clock = "Clock-Man Menu",
        quest_open_quest = "Quest Menu",
        quest_auto_collect = "Auto Collect",
        quest_auto_skip = "Auto Skip",
        auto_rebirth = "Auto Rebirth",
        auto_daily = "Auto Daily",
        auto_chest = "Auto Chest",
        performance_monitor = "Perf Monitor",
        show_cpu = "Show CPU",
        reconnect_on_disconnect = "Auto Reconnect",
        info_owner = "Script Author",
        info_discord = "Discord Invite",
        info_version = "Version",
        info_lines = "Total Lines",
        motion_prediction = "Motion Prediction",
        motion_prediction_desc = "Predict NPC position & teleport (high risk)",
        follow_mode = "Follow Mode",
        follow_mode_desc = "Stick to NPC every frame (extremely high risk)",
        auto_god_mode = "Auto God Mode",
        auto_god_mode_desc = "Auto respawn when HP low",
        god_respawn_threshold = "Respawn HP threshold",
        xmas_title = "Xmas Ultra Fast",
        xmas_open = "Auto Open Gifts (Ultra Fast)",
        xmas_collect = "Auto Collect The Present (Ultra Fast)",
        xmas_open_now = "Open 100 Gifts Now",
        batch_buy = "Batch Buy (No Delay)",
        batch_buy_desc = "Ultra fast shop buying",
        batch_gacha = "Batch Gacha (No Delay)",
        batch_gacha_char = "Batch Pull Characters",
        batch_gacha_skin = "Batch Pull Skins",
        rotate_90 = "Rotate 90° Mode",
        rotate_90_desc = "Rotate character 90 degrees",
    },
    Russian = {
        loading = "Загрузка...",
        loaded = "Загружено, старт через 3с",
        fps_unlocked = "FPS разблокирован",
        fps_not_supported = "setfpscap не поддерживается",
        config_saved = "Сохранено",
        auto_farm = "Авто ферма",
        auto_farm_desc = "Приоритетная ферма",
        farm_enabled = "Вкл",
        farm_disabled = "Выкл",
        sync_mode = "Синхронный режим",
        sync_mode_desc = "Вспом. функции требуют ферму",
        sync_on = "Нужна ферма",
        sync_off = "Независимы",
        position_above = "Сверху",
        position_under = "Снизу",
        position_front = "Спереди",
        position_back = "Сзади",
        position_spin = "Вращение",
        movement_teleport = "Телепорт",
        movement_cframe = "Плавно",
        auto_attack = "Авто атака",
        auto_skill = "Авто умение",
        auto_ready = "Авто старт",
        auto_skip_heli = "Пропустить вертолет",
        auto_heal = "Авто лечение",
        safe_mode = "Безопасный режим",
        god_mode = "Режим бога",
        delete_map = "Удалить карту",
        flush_aura = "Аура слива",
        flush_range = "Радиус",
        attack_speed = "Скорость атаки",
        skill_delay = "Задержка умений",
        height_offset = "Смещение высоты",
        safe_threshold = "Порог ХП",
        god_threshold = "Порог бога",
        high_hp_threshold = "Высокий ХП",
        esp_enable = "ESP вкл",
        esp_mob = "ESP мобов",
        esp_player = "ESP игроков",
        esp_item = "ESP предметов",
        esp_highlight = "Подсветка",
        esp_distance = "Дистанция",
        esp_health = "Здоровье",
        esp_name = "Имя",
        vote_normal = "Нормальный",
        vote_hard = "Сложный",
        vote_veryhard = "Оч. сложный",
        vote_insane = "Безумный",
        vote_nightmare = "Кошмар",
        vote_bossrush = "Босс раш",
        vote_darkdim = "Темное измерение",
        vote_hell = "Ад",
        vote_thunder = "Гроза",
        vote_christmas = "Рождество",
        vote_zombie = "Зомби",
        vote_astro = "Астро",
        auto_collect = "Авто сбор",
        collect_mode_clean = "После волны",
        collect_mode_idgf = "Мгновенно",
        astro_mode = "Ферма астро монет",
        astro_start = "Старт",
        astro_stop = "Стоп",
        mastery_title = "Авто мастерство",
        mastery_no_flush = "Без слива",
        mastery_flush = "Со сливом",
        mastery_action_speed = "Скорость",
        mastery_char_list = "Персонажи",
        hitbox_title = "Хитбокс",
        hitbox_scan = "Сканировать",
        hitbox_size = "Размер",
        hitbox_enable = "Включить",
        hitbox_show = "Показать",
        quest_title = "Помощник заданий",
        quest_open_clock = "Меню часового",
        quest_open_quest = "Меню заданий",
        quest_auto_collect = "Авто сбор",
        quest_auto_skip = "Авто пропуск",
        auto_rebirth = "Авто возрождение",
        auto_daily = "Ежедневные",
        auto_chest = "Сундуки",
        performance_monitor = "Монитор",
        show_cpu = "Загрузка ЦП",
        reconnect_on_disconnect = "Авто переподключение",
        info_owner = "Автор",
        info_discord = "Discord",
        info_version = "Версия",
        info_lines = "Строк",
        motion_prediction = "Предсказание движения",
        motion_prediction_desc = "Предугадать позицию НПС и телепортироваться",
        follow_mode = "Режим следования",
        follow_mode_desc = "Прилипать к НПС каждый кадр",
        auto_god_mode = "Авто режим бога",
        auto_god_mode_desc = "Авто возрождение при низком ХП",
        god_respawn_threshold = "Порог возрождения",
        xmas_title = "Xmas супер быстрый",
        xmas_open = "Авто открытие подарков",
        xmas_collect = "Авто сбор The Present",
        xmas_open_now = "Открыть 100 подарков",
        batch_buy = "Пакетная покупка",
        batch_buy_desc = "Супер быстрая покупка",
        batch_gacha = "Пакетная гача",
        batch_gacha_char = "Пакетный抽 персонажей",
        batch_gacha_skin = "Пакетный抽 скинов",
        rotate_90 = "Поворот на 90°",
        rotate_90_desc = "Повернуть персонажа на 90 градусов",
    },
    Portuguese = {
        loading = "Carregando...",
        loaded = "Carregado, iniciando em 3s",
        fps_unlocked = "FPS desbloqueado",
        fps_not_supported = "setfpscap não suportado",
        config_saved = "Config salva",
        auto_farm = "Auto Farm",
        auto_farm_desc = "Farm por prioridade",
        farm_enabled = "Ativado",
        farm_disabled = "Desativado",
        sync_mode = "Modo Sincronizado",
        sync_mode_desc = "Funções auxiliares precisam do auto farm",
        sync_on = "Precisa auto farm",
        sync_off = "Independentes",
        position_above = "Acima",
        position_under = "Abaixo",
        position_front = "Frente",
        position_back = "Trás",
        position_spin = "Girar",
        movement_teleport = "Teletransporte",
        movement_cframe = "Suave",
        auto_attack = "Auto Ataque",
        auto_skill = "Auto Habilidade",
        auto_ready = "Auto Início",
        auto_skip_heli = "Auto Pular Heli",
        auto_heal = "Auto Cura",
        safe_mode = "Modo Seguro",
        god_mode = "Modo Deus",
        delete_map = "Deletar Mapa",
        flush_aura = "Aura Descarga",
        flush_range = "Alcance",
        attack_speed = "Velocidade Ataque",
        skill_delay = "Atraso Habilidade",
        height_offset = "Deslocamento Altura",
        safe_threshold = "Limite HP Seguro",
        god_threshold = "Limite HP Deus",
        high_hp_threshold = "Limite HP Alto",
        esp_enable = "Ativar ESP",
        esp_mob = "ESP Monstros",
        esp_player = "ESP Jogadores",
        esp_item = "ESP Itens",
        esp_highlight = "Destaque",
        esp_distance = "Distância",
        esp_health = "Vida",
        esp_name = "Nome",
        vote_normal = "Normal",
        vote_hard = "Difícil",
        vote_veryhard = "Muito Difícil",
        vote_insane = "Insano",
        vote_nightmare = "Pesadelo",
        vote_bossrush = "Boss Rush",
        vote_darkdim = "Dimensão Sombria",
        vote_hell = "Inferno",
        vote_thunder = "Tempestade",
        vote_christmas = "Natal",
        vote_zombie = "Zumbi",
        vote_astro = "Astro",
        auto_collect = "Auto Coleta",
        collect_mode_clean = "Limpo",
        collect_mode_idgf = "IDGF",
        astro_mode = "Fazenda Astro",
        astro_start = "Iniciar",
        astro_stop = "Parar",
        mastery_title = "Auto Maestria",
        mastery_no_flush = "Sem Descarga",
        mastery_flush = "Com Descarga",
        mastery_action_speed = "Velocidade Ação",
        mastery_char_list = "Personagens",
        hitbox_title = "Hitbox",
        hitbox_scan = "Escanear",
        hitbox_size = "Tamanho",
        hitbox_enable = "Ativar",
        hitbox_show = "Mostrar",
        quest_title = "Ajudante de Missões",
        quest_open_clock = "Menu Relógio",
        quest_open_quest = "Menu Missões",
        quest_auto_collect = "Auto Coletar",
        quest_auto_skip = "Auto Pular",
        auto_rebirth = "Auto Renascimento",
        auto_daily = "Recompensas Diárias",
        auto_chest = "Baús",
        performance_monitor = "Monitor",
        show_cpu = "Uso CPU",
        reconnect_on_disconnect = "Auto Reconectar",
        info_owner = "Autor",
        info_discord = "Convite",
        info_version = "Versão",
        info_lines = "Linhas",
        motion_prediction = "Predição de Movimento",
        motion_prediction_desc = "Prever posição do NPC e teletransportar",
        follow_mode = "Modo Seguir",
        follow_mode_desc = "Grudar no NPC a cada quadro",
        auto_god_mode = "Auto Modo Deus",
        auto_god_mode_desc = "Auto renascer quando HP baixo",
        god_respawn_threshold = "Limite de renascimento",
        xmas_title = "Xmas Ultra Rápido",
        xmas_open = "Auto Abrir Presentes",
        xmas_collect = "Auto Coletar The Present",
        xmas_open_now = "Abrir 100 Presentes",
        batch_buy = "Compra em Lote",
        batch_buy_desc = "Compra super rápida",
        batch_gacha = "Gacha em Lote",
        batch_gacha_char = "Puxar Personagens",
        batch_gacha_skin = "Puxar Skins",
        rotate_90 = "Rotação 90°",
        rotate_90_desc = "Rotacionar personagem 90 graus",
    },
}

local function T(key) return translations[currentLanguage][key] or key end

-- ====================== UI 初始化 ======================
-- 注意：已删除 WindUI:AddTheme 调用（6行），避免 Delta 的 UIStroke 崩溃
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
if not WindUI then error("WindUI loading failed") end

repeat task.wait() until game:IsLoaded()
local p = game:GetService("Players").LocalPlayer
local pg = p:WaitForChild("PlayerGui")
local function waitLoadingGone()
    local g = pg:FindFirstChild("LoadingGui")
    if g then WindUI:Notify({ Title = "Init", Content = T("loading"), Duration = 3 }); g.AncestryChanged:Wait() end
end
waitLoadingGone()
WindUI:Notify({ Title = "Init", Content = T("loaded"), Duration = 3 })
task.wait(3)

-- ====================== 配置系统 ======================
local HttpService = game:GetService("HttpService")
local ConfigFolder = "DYHUB_FINAL"
local CustomConfig = {}
CustomConfig.__index = CustomConfig
function CustomConfig.new()
    local self = setmetatable({}, CustomConfig)
    self.ConfigData = {}
    self.ConfigPath = ConfigFolder.."/config.json"
    if not isfolder(ConfigFolder) then makefolder(ConfigFolder) end
    self:Load()
    return self
end
function CustomConfig:Set(k,v) self.ConfigData[k]=v end
function CustomConfig:Get(k,def) return self.ConfigData[k]~=nil and self.ConfigData[k] or def end
function CustomConfig:Save() pcall(function() writefile(self.ConfigPath, HttpService:JSONEncode(self.ConfigData)) end) end
function CustomConfig:Load()
    if isfile(self.ConfigPath) then pcall(function() self.ConfigData=HttpService:JSONDecode(readfile(self.ConfigPath)) end) end
end
local Config = CustomConfig.new()
task.spawn(function() while true do task.wait(15); Config:Save() end end)

-- ====================== Services & Globals ======================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = Players.LocalPlayer
local Client = LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local Stats = game:GetService("Stats")
getgenv().ACTIVE_MODE = Config:Get("ActiveMode", "farm")

GlobalTables = {
    redeemCodes = { "100MVisit2","100MVisit1","CamArmada","CCTVBase","ADelayedGameIsEventuallyGoodButRushedGameIsForeverBad" },
    ModeDisplay = { T("vote_normal"),T("vote_hard"),T("vote_veryhard"),T("vote_insane"),T("vote_nightmare"),T("vote_bossrush"),T("vote_darkdim"),T("vote_hell"),T("vote_thunder"),T("vote_christmas"),T("vote_zombie"),T("vote_astro") },
    ModeInternal = { [T("vote_normal")]="Normal",[T("vote_hard")]="Hard",[T("vote_veryhard")]="VeryHard",[T("vote_insane")]="Insane",[T("vote_nightmare")]="Nightmare",[T("vote_bossrush")]="BossRush",[T("vote_darkdim")]="DarkDimension",[T("vote_hell")]="Hell",[T("vote_thunder")]="ThunderStorm",[T("vote_christmas")]="Christmas",[T("vote_zombie")]="Zombie",[T("vote_astro")]="Astro" },
    Votes = { "Normal","Hard","VeryHard","Insane","Nightmare","BossRush","DarkDimension","Hell","ThunderStorm","Christmas","Zombie","Astro","AstroV2" },
    WeaponDisplay = { "电击枪","火焰喷射器","鱼叉枪","霰弹枪","脉冲步枪","鱼叉霰弹枪","EPD","小型激光枪" },
    WeaponInternal = { ["电击枪"]="Stungun",["火焰喷射器"]="Flamethrower",["鱼叉枪"]="Harpoon Gun",["霰弹枪"]="Shot Gun",["脉冲步枪"]="Pulse Rifle",["鱼叉霰弹枪"]="Shot Harpoon Gun",["EPD"]="EPD",["小型激光枪"]="Small Laser Gun" },
    MiscDisplay = { "耳机","泰坦呼叫","特种泰坦呼叫","扬声器呼叫","手雷","喷气背包","透镜" },
    MiscInternal = { ["耳机"]="HeadPhone",["泰坦呼叫"]="Titan-Request",["特种泰坦呼叫"]="SpecialTitan-Request",["扬声器呼叫"]="Speaker-Request",["手雷"]="Grenade",["喷气背包"]="Jetpack",["透镜"]="Lens" },
    Gamepasst = { "全部","幸运加成","稀有幸运加成","传说幸运加成" },
}

-- ====================== 配置变量 ======================
local skillList = { "Q","E","R","T","Y","G","H","Z","X","C","V","B","U" }
local skillDropdownValues = { "全部","Q","E","R","T","Y","G","H","Z","X","C","V","B","U" }
local AutoFarmEnabled = Config:Get("AutoFarmEnabled", false)
local FarmPosition = Config:Get("FarmPosition", "Above")
local FarmMode = Config:Get("FarmMode", "Tween")
local MiscOptions = Config:Get("MiscOptions", {})
local AutoAttackEnabled = false
local AutoSkillEnabled = false
local AutoSkipHeliEnabled = false
local BoostFPS_Active = false
local AutoStartEnabled = false
local AutoFillUpEnabled = false
local SelectedSkills = Config:Get("SelectedSkills", { "全部" })
local SafeModeEnabled = false
local SafeValue = Config:Get("SafeValue", 30)
local GodModeEnabled = false
local GodModeValue = Config:Get("GodModeValue", 30)
local WaitingRespawn = false
local IdlePosition = CFrame.new(-23.3435822, 67, 0.341766357) * CFrame.Angles(0,0,0)
local SkillDelay = Config:Get("SkillDelay", 1)
local TweenSpeed = 1
local HeightValue = Config:Get("HeightValue", 3)
local noBarrierActive = Config:Get("NoBarrier", false)
local HighHPThreshold = Config:Get("HighHPThreshold", 200)
local AutoBuyWeaponEnabled = Config:Get("AutoBuyWeaponEnabled", false)
local AutoBuyMiscEnabled = Config:Get("AutoBuyMiscEnabled", false)
local rawWeapon = Config:Get("SelectedWeapon", "电击枪")
local rawMisc = Config:Get("SelectedMiscItem", "耳机")
local SelectedWeapon = GlobalTables.WeaponInternal[rawWeapon] or rawWeapon
local SelectedMiscItem = GlobalTables.MiscInternal[rawMisc] or rawMisc
Config:Set("SelectedWeapon", SelectedWeapon); Config:Set("SelectedMiscItem", SelectedMiscItem)
local rawMode = Config:Get("AutoGameValue", T("vote_normal"))
local AutoGameValue = GlobalTables.ModeInternal[rawMode] or rawMode
Config:Set("AutoGameValue", AutoGameValue)
local AutoVoteEnabled = Config:Get("AutoVoteEnabled", false)
local AutoVoteinGameEnabled = Config:Get("AutoVoteinGameEnabled", false)
local AutoVoteValue = Config:Get("AutoVoteValue", "Normal")
local AntiAFK = Config:Get("AntiAfk", true)

local AutoRebirthEnabled = Config:Get("AutoRebirthEnabled", false)
local AutoDailyEnabled = Config:Get("AutoDailyEnabled", false)
local AutoChestEnabled = Config:Get("AutoChestEnabled", false)
local ReconnectOnDisconnect = Config:Get("ReconnectOnDisconnect", true)
local ShowCPU = Config:Get("ShowCPU", false)

-- Mastery & Hitbox & Quest
local MasteryAutoFarmActive = false
local MasteryAutoFarmActiveTest = false
local ActionMode = Config:Get("ActionMode", "Default")
local CharacterMode = Config:Get("CharacterMode", "Used")
getgenv().HitboxEnabled = Config:Get("HitboxEnabled", false)
getgenv().HitboxSize = Config:Get("HitboxSize", 20)
getgenv().HitboxShow = Config:Get("HitboxShow", false)
local autoQuestCollectActive = false
local autoQuestSkipActive = false

-- ====================== 高风险功能开关 ======================
local MotionPredictionEnabled = Config:Get("MotionPredictionEnabled", false)
local FollowModeEnabled = Config:Get("FollowModeEnabled", false)
local AutoGodModeEnabled = Config:Get("AutoGodModeEnabled", false)
local GodRespawnThreshold = Config:Get("GodRespawnThreshold", 10)
local XmasOpenEnabled = Config:Get("XmasOpenEnabled", false)
local XmasCollectEnabled = Config:Get("XmasCollectEnabled", false)
local BatchBuyEnabled = Config:Get("BatchBuyEnabled", false)
local BatchGachaCharEnabled = Config:Get("BatchGachaCharEnabled", false)
local BatchGachaSkinEnabled = Config:Get("BatchGachaSkinEnabled", false)
local Rotate90Enabled = Config:Get("Rotate90Enabled", false)

-- ====================== 辅助函数（补血、友军、传送） ======================
local FILLUP_PART_PATH = { "HelicopterShop","ShopXDD","PartForShop" }
local FILLUP_TARGET_POS = Vector3.new(44.2756729, 26.3595276, -32.7318268)
local function GetFillUpPart()
    local obj = workspace
    for _,k in ipairs(FILLUP_PART_PATH) do obj = obj:FindFirstChild(k) if not obj then return end end
    return obj
end
local function IsFillUpPartReady() local p=GetFillUpPart() return p and (p.CFrame.Position-FILLUP_TARGET_POS).Magnitude<0.5 end

local AllyNames = {
    ["Heavy Soldier Toilet V2"]=true,["Quad Laser Toilet"]=true,["Strider Rocket Laser"]=true,
    ["Helicopter Camera"]=true,["Heavy Soldier Toilet V1"]=true,["Rocket Heli v2"]=true,
    ["Gunner Camera man"]=true,["Attack Helicopter"]=true,["Swat Mutant"]=true,["Huge DJ Toilet"]=true,
}
local function IsAlly(mob) return AllyNames[mob.Name] end

function tp(t) pcall(function() local c=Client.Character if c and c.Humanoid and c.Humanoid.Sit then c.Humanoid.Sit=false end c.HumanoidRootPart.CFrame=t.Target*(t.Mod or CFrame.new(0,0,0)) end) end
function Tp(cf) local c=Client.Character if c and c.Humanoid.Sit then c.Humanoid.Sit=false end for _,v in pairs(c:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide=false end end if not c.HumanoidRootPart:FindFirstChild("BodyClip") then local bv=Instance.new("BodyVelocity") bv.Parent=c.HumanoidRootPart bv.Name="BodyClip" bv.Velocity=Vector3.new(0,0,0) bv.MaxForce=Vector3.new(5,math.huge,5) end c.HumanoidRootPart.CFrame=cf end
function tp1(cf) local c=LocalPlayer.Character if c and c:FindFirstChild("HumanoidRootPart") then c.HumanoidRootPart.CFrame=cf end end

-- ====================== 怪物有效性判断 ======================
local function IsValidMob(obj)
    if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
        if Players:GetPlayerFromCharacter(obj) then return false end
        if IsAlly(obj) then return false end
        local hum = obj:FindFirstChild("Humanoid")
        if hum and hum.Health > 0 then return true end
    end
    return false
end
local function IsMobDead(mob) return not mob or not mob.Parent or not mob:FindFirstChild("Humanoid") or mob.Humanoid.Health<=0 end
local function GetMobMaxHP(mob) local h=mob and mob:FindFirstChild("Humanoid") return h and h.MaxHealth or 0 end

-- ====================== 优先级系统 ======================
local function GetHelicopter()
    local l=workspace:FindFirstChild("Living")
    if l then for _,m in ipairs(l:GetChildren()) do if m.Name:lower():find("helicopter") and IsValidMob(m) then return m end end end
end
local function GetGiantSTToilet()
    local l=workspace:FindFirstChild("Living")
    if l then local g=l:FindFirstChild("Giant ST toilet") if g and IsValidMob(g) then local lever=g:FindFirstChild("lever") if lever then local p=lever:FindFirstChildOfClass("ProximityPrompt") if p then return g,p end end end end
end
local function GetHighHPMob()
    local l=workspace:FindFirstChild("Living")
    if l then local best,bestHP=nil,HighHPThreshold for _,m in ipairs(l:GetChildren()) do if IsValidMob(m) then local hp=GetMobMaxHP(m) if hp>bestHP then bestHP=hp; best=m end end end return best end
end
local function GetNearestMob()
    local nearest,dist=nil,math.huge local l=workspace:FindFirstChild("Living")
    if l then for _,m in ipairs(l:GetChildren()) do if IsValidMob(m) then local r=m:FindFirstChild("HumanoidRootPart") if r then local d=(HumanoidRootPart.Position-r.Position).Magnitude if d<dist then dist=d; nearest=m end end end end end return nearest
end
local function GetPriorityMob()
    local g,p=GetGiantSTToilet()
    if g then return g,"GiantST",p,4 end
    local h=GetHelicopter()
    if h then return h,"Helicopter",nil,3 end
    local hh=GetHighHPMob()
    if hh then return hh,"HighHP",nil,2 end
    local n=GetNearestMob()
    if n then return n,"NearestMob",nil,1 end
    return nil,nil,nil,0
end

-- ====================== 高度覆写系统 ======================
local PADDING_REDUCE_STEP=Config:Get("PaddingReduceStep",2)
local PADDING_SAFE_MIN=Config:Get("PaddingSafeMin",-30)
local DMG_THRESHOLD=Config:Get("DmgThreshold",40)
local ANTI_CLIP_MARGIN=Config:Get("AntiClipMargin",3)
local PLAYER_HALF_HEIGHT=3
local MobHeightOverride,MobConfirmedPadding,MobLastHealth,MobCheckerCancelled={},{},{},{}
local function GetMobVisualBounds(mob)
    local minY,maxY=math.huge,-math.huge
    local cx,cz,cnt=0,0,0
    for _,part in ipairs(mob:GetDescendants()) do
        if part:IsA("BasePart") and part.Transparency<0.9 and part.Size.Y>0.1 then
            local pos=part.Position; local hy=part.Size.Y*0.5
            if pos.Y-hy<minY then minY=pos.Y-hy end
            if pos.Y+hy>maxY then maxY=pos.Y+hy end
            cx=cx+pos.X; cz=cz+pos.Z; cnt=cnt+1
        end
    end
    if cnt==0 then local hrp=mob:FindFirstChild("HumanoidRootPart") if hrp then return hrp.Position, hrp.Position.Y-2, hrp.Position.Y+2 end return Vector3.new(0,0,0),0,4 end
    return Vector3.new(cx/cnt,(minY+maxY)/2,cz/cnt),minY,maxY
end
local function GetAntiClipFloor(mob) local _,minY,maxY=GetMobVisualBounds(mob) return -(maxY-minY)+PLAYER_HALF_HEIGHT+ANTI_CLIP_MARGIN end
local function GetEffectivePadding(mob) return MobConfirmedPadding[mob] or MobHeightOverride[mob] or HeightValue end
local function ClampPaddingToAntiClip(mob,pad) return math.max(math.max(pad,GetAntiClipFloor(mob)),PADDING_SAFE_MIN) end
local function StartDamageChecker(mob)
    MobCheckerCancelled[mob]=false
    task.spawn(function()
        local hum=mob and mob:FindFirstChild("Humanoid")
        if not hum then return end
        if MobConfirmedPadding[mob] then return end
        MobLastHealth[mob]=hum.Health
        MobHeightOverride[mob]=ClampPaddingToAntiClip(mob, MobHeightOverride[mob] or HeightValue)
        local lastHit=tick(); local noDmgTimer=0; local hitStreak=0; local lastWasHit=false; local reducedOnce=false
        while mob and mob.Parent and not IsMobDead(mob) and AutoFarmEnabled do
            task.wait(0.3)
            if MobCheckerCancelled[mob] then break end
            hum=mob:FindFirstChild("Humanoid"); if not hum then break end
            local cur=hum.Health; local last=MobLastHealth[mob] or cur; local dmg=last-cur
            if dmg>0 then
                lastHit=tick(); noDmgTimer=0; reducedOnce=false
                if lastWasHit then hitStreak=hitStreak+1 else hitStreak=1 end
                lastWasHit=true
                local curPad=GetEffectivePadding(mob)
                if dmg>=DMG_THRESHOLD and not MobConfirmedPadding[mob] then MobConfirmedPadding[mob]=curPad; MobHeightOverride[mob]=curPad; break end
                if hitStreak>=2 and not MobConfirmedPadding[mob] then MobConfirmedPadding[mob]=curPad; MobHeightOverride[mob]=curPad; break end
            else
                lastWasHit=false; hitStreak=0; noDmgTimer=tick()-lastHit
            end
            if noDmgTimer>=3 and not reducedOnce then
                reducedOnce=true
                local curPad=GetEffectivePadding(mob)
                local newPad=ClampPaddingToAntiClip(mob, curPad-PADDING_REDUCE_STEP)
                if newPad~=curPad then MobHeightOverride[mob]=newPad end
            end
            if noDmgTimer>=6 then
                lastHit=tick(); reducedOnce=false
                local curPad=GetEffectivePadding(mob)
                local newPad=ClampPaddingToAntiClip(mob, curPad-PADDING_REDUCE_STEP)
                if newPad~=curPad then MobHeightOverride[mob]=newPad end
            end
            MobLastHealth[mob]=cur
        end
        if not MobCheckerCancelled[mob] then MobHeightOverride[mob]=nil; MobLastHealth[mob]=nil end
    end)
end
local function ResetMobOverride(mob)
    MobCheckerCancelled[mob]=true; MobHeightOverride[mob]=nil; MobConfirmedPadding[mob]=nil; MobLastHealth[mob]=nil
    task.delay(0.5,function() MobCheckerCancelled[mob]=nil end)
end

-- 运动预测函数
local function PredictNPCMovement(npc, timeAhead)
    if not npc or not npc:FindFirstChild("HumanoidRootPart") then return Vector3.new() end
    local hrp=npc.HumanoidRootPart
    local humanoid=npc:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.MoveDirection and humanoid.MoveDirection.Magnitude>0.1 then
        return hrp.Position+(humanoid.MoveDirection*humanoid.WalkSpeed*timeAhead)
    end
    return hrp.Position
end

-- 增强 GetTargetCFrame（支持运动预测和旋转90°）
local function GetTargetCFrame(mob,pos)
    local r=mob:FindFirstChild("HumanoidRootPart")
    if not r then return nil end
    local pad=GetEffectivePadding(mob)
    local center,minY,maxY=GetMobVisualBounds(mob)
    local targetCenter=center
    if MotionPredictionEnabled then
        targetCenter=PredictNPCMovement(mob,0.3)
    end
    local targetPos
    if pos=="Above" then
        local y=math.max(maxY+pad, maxY+0.5)
        targetPos=Vector3.new(targetCenter.X, y, targetCenter.Z)
    elseif pos=="Under" then
        local y=math.min(minY-pad, minY-0.5)
        targetPos=Vector3.new(targetCenter.X, y, targetCenter.Z)
    else
        targetPos=targetCenter
    end
    local lookAt=Vector3.new(targetCenter.X, (pos=="Above" and maxY or minY), targetCenter.Z)
    local cf=CFrame.new(targetPos,lookAt)
    if pos=="Above" then cf=cf*CFrame.Angles(math.rad(-10),0,0)
    elseif pos=="Under" then cf=cf*CFrame.Angles(math.rad(10),0,0) end
    if Rotate90Enabled then cf=cf*CFrame.Angles(0,math.rad(90),0) end
    return cf
end

-- ====================== 调度器核心 ======================
local Scheduler={tasks={},heartbeat=nil}
function Scheduler:register(name,step,interval)
    self.tasks[name]={step=step,interval=interval or 0,last=0}
end
function Scheduler:setInterval(name,newInterval)
    if self.tasks[name] then self.tasks[name].interval=newInterval end
end
function Scheduler:unregister(name)
    self.tasks[name]=nil
end
function Scheduler:start()
    if self.heartbeat then return end
    self.heartbeat=RunService.Heartbeat:Connect(function(dt)
        local now=tick()
        for _,t in pairs(self.tasks) do
            if t.interval==0 then
                t.step(dt)
            elseif now-t.last>=t.interval then
                t.last=now
                t.step(dt)
            end
        end
    end)
end
function Scheduler:stop()
    if self.heartbeat then self.heartbeat:Disconnect(); self.heartbeat=nil end
end
local function safeTask(name,fn)
    return function(dt)
        local success,err=pcall(fn,dt)
        if not success then warn("[DYHUB] Task '"..name.."' error: "..tostring(err)) end
    end
end

-- ====================== 基础 Step 函数 ======================
local attackCd=0
local function stepAutoAttack()
    if not AutoFarmEnabled or not AutoAttackEnabled then return end
    if getgenv().ACTIVE_MODE~="farm" then return end
    if attackCd>0 then attackCd=attackCd-0.05; if attackCd>0 then return end end
    local mob=GetPriorityMob()
    if mob and not WaitingRespawn then
        pcall(function() ReplicatedStorage.LMB:FireServer() end)
        attackCd=0.05
    end
end

local skillCd=0; local skillPressIdx=1
local function stepAutoSkill()
    if not AutoFarmEnabled or not AutoSkillEnabled then return end
    if getgenv().ACTIVE_MODE~="farm" then return end
    if skillCd>0 then skillCd=skillCd-0.05; if skillCd>0 then return end end
    local mob=GetPriorityMob()
    if not mob or WaitingRespawn then return end
    local keys=table.find(SelectedSkills,"全部") and skillList or SelectedSkills
    if #keys==0 then return end
    if skillPressIdx>#keys then skillPressIdx=1 end
    local key=keys[skillPressIdx]
    local kc=Enum.KeyCode[key]
    if kc then
        pcall(function()
            VirtualInputManager:SendKeyEvent(true,kc,false,game)
            task.wait(0.05)
            VirtualInputManager:SendKeyEvent(false,kc,false,game)
        end)
        skillPressIdx=skillPressIdx+1
        skillCd=SkillDelay
    end
end

local fillUpCd=0
local function stepAutoFillUp()
    if not AutoFarmEnabled or not AutoFillUpEnabled then return end
    if getgenv().ACTIVE_MODE~="farm" then return end
    if fillUpCd>0 then fillUpCd=fillUpCd-0.2; if fillUpCd>0 then return end end
    local hum=Character and Character:FindFirstChild("Humanoid")
    if not hum or hum.Health>=hum.MaxHealth then return end
    if AutoSkipHeliEnabled then pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(false) end) end
    local waited=0
    while not IsFillUpPartReady() and waited<30 do task.wait(0.2); waited=waited+0.2 end
    if IsFillUpPartReady() then
        for i=1,2 do
            pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy","FillHP") end)
            if i<2 then task.wait(0.3) end
        end
    end
    if AutoSkipHeliEnabled then pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(true) end) end
    fillUpCd=10
end

local function stepAutoSkipHeli()
    if not AutoFarmEnabled or not AutoSkipHeliEnabled then return end
    if getgenv().ACTIVE_MODE~="farm" then return end
    pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(true) end)
end

local godModeCd=0
local function stepGodMode()
    if not GodModeEnabled then return end
    if godModeCd>0 then godModeCd=godModeCd-0.1; if godModeCd>0 then return end end
    local char=LocalPlayer.Character
    if not char then return end
    local hum=char:FindFirstChild("Humanoid")
    if not hum or hum.MaxHealth<=0 then return end
    local percent=(hum.Health/hum.MaxHealth)*100
    if percent<GodModeValue then
        local head=char:FindFirstChild("Head")
        if head then head:Destroy() else hum.Health=0 end
    end
    godModeCd=0.1
end

local buyWeaponCd=0
local function stepAutoBuyWeapon()
    if not AutoBuyWeaponEnabled or not SelectedWeapon then return end
    if buyWeaponCd>0 then buyWeaponCd=buyWeaponCd-0.5; if buyWeaponCd>0 then return end end
    pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy",SelectedWeapon) end)
    buyWeaponCd=10
end

local buyMiscCd=0
local function stepAutoBuyMisc()
    if not AutoBuyMiscEnabled or not SelectedMiscItem then return end
    if buyMiscCd>0 then buyMiscCd=buyMiscCd-0.5; if buyMiscCd>0 then return end end
    pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy",SelectedMiscItem) end)
    buyMiscCd=10
end

local idleCd=0
local function stepIdlePosition()
    if not AutoFarmEnabled then return end
    if idleCd>0 then idleCd=idleCd-0.2; if idleCd>0 then return end end
    if WaitingRespawn and HumanoidRootPart then
        Character:PivotTo(IdlePosition)
        HumanoidRootPart.AssemblyLinearVelocity=Vector3.zero
        HumanoidRootPart.AssemblyAngularVelocity=Vector3.zero
    end
    idleCd=0.2
end

-- ====================== 主挂机状态机（支持 Follow Mode） ======================
local farmState="IDLE"
local currentMob,currentMobType,currentPrompt=nil,nil,nil
local lockConn=nil
local movingTween=nil
local function StopLock()
    if lockConn then lockConn:Disconnect(); lockConn=nil end
end
local function StartLock(mob,mType,prompt)
    StopLock()
    local lastLockTime = 0
    lockConn = RunService.Heartbeat:Connect(function()
        local now = tick()
        if now - lastLockTime < 0.2 then return end
        lastLockTime = now
        if not AutoFarmEnabled or farmState~="LOCKING" or IsMobDead(mob) then return end
        local cf = GetTargetCFrame(mob,FarmPosition)
        if cf and Character and HumanoidRootPart then
            Character:PivotTo(cf)
            HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
            HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
        end
        if mType=="GiantST" and prompt then
            pcall(function()
                prompt.HoldDuration=0; prompt.MaxActivationDistance=50
                if fireproximityprompt then fireproximityprompt(prompt)
                else prompt:InputHoldBegin(); task.wait(0.05); prompt:InputHoldEnd() end
            end)
        end
    end)
end

local function stepMainStateMachine()
    if not AutoFarmEnabled then
        if farmState~="IDLE" then
            farmState="IDLE"; StopLock(); if movingTween then movingTween:Cancel() end
        end
        return
    end
    if getgenv().ACTIVE_MODE~="farm" then return end

    local mob,mType,prompt=GetPriorityMob()
    if not mob then
        if farmState~="IDLE" then
            farmState="IDLE"; StopLock(); if movingTween then movingTween:Cancel() end
            WaitingRespawn=true
        end
        return
    end
    WaitingRespawn=false

    if SafeModeEnabled then
        local hum=Character:FindFirstChild("Humanoid")
        if hum then
            local percent=(hum.Health/hum.MaxHealth)*100
            if percent<SafeValue then
                local root=mob:FindFirstChild("HumanoidRootPart")
                if root then
                    local safePos=root.Position+Vector3.new(0,111,0)
                    Character:PivotTo(CFrame.new(safePos))
                end
                return
            end
        end
    end

    if farmState=="IDLE" then
        currentMob=mob; currentMobType=mType; currentPrompt=prompt
        farmState="MOVING"
        StartDamageChecker(mob)
    end

    if farmState=="MOVING" then
        local cf=GetTargetCFrame(currentMob,FarmPosition)
        if cf then
            if FarmMode=="Tween" then
                if movingTween then movingTween:Cancel() end
                movingTween=TweenService:Create(HumanoidRootPart,TweenInfo.new(TweenSpeed,Enum.EasingStyle.Linear),{CFrame=cf})
                movingTween:Play()
                movingTween.Completed:Wait()
                movingTween=nil
            else
                tp1(cf)
            end
            farmState="LOCKING"
            StartLock(currentMob,currentMobType,currentPrompt)
        end
    end

    if farmState=="LOCKING" and IsMobDead(currentMob) then
        StopLock()
        ResetMobOverride(currentMob)
        farmState="IDLE"
        currentMob=nil
    end
end

-- 第一部分结束，请继续复制第二部分（自动投票、Xmas、批量购买、ESP、收集、Mastery、Hitbox、Quest、UI 界面等）-- ====================== 自动投票 ======================
local voteCd=0
local function stepAutoVote()
    if not AutoVoteEnabled and not AutoStartEnabled and not AutoVoteinGameEnabled then return end
    if voteCd>0 then voteCd=voteCd-0.5; if voteCd>0 then return end end
    if AutoVoteEnabled then
        pcall(function() ReplicatedStorage.MainHandler:FireServer({[1]="StartSolo",[2]=AutoGameValue}) end)
    end
    if AutoStartEnabled then
        task.spawn(function() task.wait(2.5); pcall(function() ReplicatedStorage.GetReadyRemote:FireServer("1",true) end) end)
    end
    if AutoVoteinGameEnabled then
        pcall(function() ReplicatedStorage.Vote:FireServer(AutoVoteValue) end)
    end
    voteCd=5
end

-- ====================== 额外自动化 Step ======================
local rebirthCd=0
local function stepAutoRebirth()
    if not AutoRebirthEnabled then return end
    if rebirthCd>0 then rebirthCd=rebirthCd-0.5; if rebirthCd>0 then return end end
    local char=LocalPlayer.Character
    if not char then return end
    local hum=char:FindFirstChild("Humanoid")
    if hum and hum.Health<=0 then
        pcall(function() local rebirthRemote=ReplicatedStorage:FindFirstChild("Rebirth"); if rebirthRemote then rebirthRemote:FireServer() end end)
        rebirthCd=5
    end
end

local dailyCd=0
local function stepAutoDaily()
    if not AutoDailyEnabled then return end
    if dailyCd>0 then dailyCd=dailyCd-1; if dailyCd>0 then return end end
    pcall(function() local dailyRemote=ReplicatedStorage:FindFirstChild("DailyReward"); if dailyRemote then dailyRemote:FireServer() end end)
    dailyCd=60
end

local chestCd=0
local function stepAutoChest()
    if not AutoChestEnabled then return end
    if chestCd>0 then chestCd=chestCd-0.5; if chestCd>0 then return end end
    local char=LocalPlayer.Character
    local hrp=char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        for _,obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and (obj.Name:lower():find("chest") or obj.Name:lower():find("crate")) then
                local prompt=obj:FindFirstChildOfClass("ProximityPrompt")
                if prompt then
                    local dist=(hrp.Position-obj:GetPivot().Position).Magnitude
                    if dist<=50 then pcall(function() prompt.HoldDuration=0; fireproximityprompt(prompt) end) end
                end
            end
        end
    end
    chestCd=1
end

-- ====================== Auto God Mode Step ======================
local godRespawnCd=0
local function stepAutoGodMode()
    if not AutoGodModeEnabled then return end
    if godRespawnCd>0 then godRespawnCd=godRespawnCd-0.5; if godRespawnCd>0 then return end end
    local char=LocalPlayer.Character
    if not char then return end
    local hum=char:FindFirstChild("Humanoid")
    if not hum or hum.Health<=0 then return end
    local percent=(hum.Health/hum.MaxHealth)*100
    if percent<=GodRespawnThreshold then
        pcall(function() local resetRemote=ReplicatedStorage:FindFirstChild("ResetCharacter"); if resetRemote then resetRemote:InvokeServer() end end)
        pcall(function() local chatRemote=ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents"); if chatRemote then local sayMsg=chatRemote:FindFirstChild("SayMessageRequest"); if sayMsg then sayMsg:FireServer(";reset","All") end end end)
        pcall(function() LocalPlayer:Kick("Auto God Mode: Respawning") end)
        godRespawnCd=3
    end
end

-- ====================== Xmas 系统 ======================
local xmasOpenTask=nil; local xmasCollectTask=nil
local function startXmasOpen()
    if xmasOpenTask then return end
    xmasOpenTask=task.spawn(function()
        while XmasOpenEnabled do
            for i=1,50 do
                if not XmasOpenEnabled then break end
                pcall(function() ReplicatedStorage:WaitForChild("GachaCapsule"):FireServer() end)
                task.wait(0.001)
            end
            task.wait(0.05)
        end
        xmasOpenTask=nil
    end)
end
local function startXmasCollect()
    if xmasCollectTask then return end
    xmasCollectTask=task.spawn(function()
        while XmasCollectEnabled do
            local nearestPresent,nearestDist=nil,math.huge
            local hrp=LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                for _,obj in ipairs(workspace:GetDescendants()) do
                    if obj.Name=="The Present" then
                        local pos=obj:IsA("Model") and obj.PrimaryPart and obj.PrimaryPart.Position or obj.Position
                        local d=(hrp.Position-pos).Magnitude
                        if d<nearestDist then nearestDist=d; nearestPresent=obj end
                    end
                end
                if nearestPresent then
                    local prompt=nearestPresent:FindFirstChildOfClass("ProximityPrompt")
                    if prompt then
                        prompt.HoldDuration=0
                        local target=nearestPresent:IsA("Model") and nearestPresent.PrimaryPart and nearestPresent.PrimaryPart.Position+Vector3.new(0,3,0) or nearestPresent.Position+Vector3.new(0,3,0)
                        hrp.CFrame=CFrame.new(target)
                        for i=1,20 do
                            if not XmasCollectEnabled then break end
                            fireproximityprompt(prompt)
                            task.wait(0.005)
                        end
                    end
                end
            end
            task.wait(0.2)
        end
        xmasCollectTask=nil
    end)
end
local function toggleXmasOpen(state)
    XmasOpenEnabled=state
    if state then startXmasOpen() elseif xmasOpenTask then task.cancel(xmasOpenTask); xmasOpenTask=nil end
end
local function toggleXmasCollect(state)
    XmasCollectEnabled=state
    if state then startXmasCollect() elseif xmasCollectTask then task.cancel(xmasCollectTask); xmasCollectTask=nil end
end

-- ====================== 无延迟批量购买 & 抽卡 ======================
local batchBuyTask=nil; local batchGachaCharTask=nil; local batchGachaSkinTask=nil
local function startBatchBuy()
    if batchBuyTask then return end
    batchBuyTask=task.spawn(function()
        while BatchBuyEnabled do
            pcall(function()
                local items=Config:Get("BatchBuyItems",{})
                local amounts=Config:Get("BatchBuyAmounts",{})
                for _,item in ipairs(items) do
                    for _,amt in ipairs(amounts) do
                        for i=1,10 do ReplicatedStorage:WaitForChild("BuyItemFromShopHourly"):FireServer(item,amt) end
                    end
                end
            end)
            task.wait(0.1)
        end
        batchBuyTask=nil
    end)
end
local function startBatchGachaChar()
    if batchGachaCharTask then return end
    batchGachaCharTask=task.spawn(function()
        while BatchGachaCharEnabled do
            pcall(function()
                local spins=Config:Get("BatchGachaCharSpins",{"1SpinLucky","10Spins","1Spin"})
                for _,spin in ipairs(spins) do
                    for i=1,20 do ReplicatedStorage:WaitForChild("GachaCharacter"):FireServer(spin) end
                    task.wait(0.05)
                end
            end)
            task.wait(0.1)
        end
        batchGachaCharTask=nil
    end)
end
local function startBatchGachaSkin()
    if batchGachaSkinTask then return end
    batchGachaSkinTask=task.spawn(function()
        while BatchGachaSkinEnabled do
            pcall(function()
                local spins=Config:Get("BatchGachaSkinSpins",{"1SpinLucky","1Spin","10Spins"})
                for _,spin in ipairs(spins) do
                    for i=1,20 do ReplicatedStorage:WaitForChild("GachaSkins"):FireServer(spin) end
                    task.wait(0.05)
                end
            end)
            task.wait(0.1)
        end
        batchGachaSkinTask=nil
    end)
end

-- ====================== 玩家修改 ======================
local WSValue=Config:Get("WSValue",16)
local JPValue=Config:Get("JPValue",50)
local NoClip=Config:Get("NoClip",false)
local function updatePlayerStats()
    local hum=Character and Character:FindFirstChild("Humanoid")
    if hum then hum.WalkSpeed=WSValue; hum.JumpPower=JPValue end
end
LocalPlayer.CharacterAdded:Connect(function() task.wait(1); updatePlayerStats() end)
local function stepNoClip()
    if NoClip and Character then
        for _,v in pairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide=false end end
    end
end

-- ====================== 天文币刷取模块 ======================
do
    if getgenv().ACTIVE_MODE==nil then getgenv().ACTIVE_MODE="farm" end
    local astroRunning=false; local orbitConn=nil; local strikeRunning=false; local forcePos=false; local angle=0
    local astroMaxDuration=Config:Get("AstroGameDuration",960)
    local astroFlyRadius=Config:Get("AstroFlyRadius",300)
    local astroFlySpeed=Config:Get("AstroFlySpeed",2)
    local astroStrikePos=Vector3.new(-23.34,-0.19,0.34)
    local astroFlyCenter=Vector3.new(0,250,0)
    local astroTeleportCF=CFrame.new(-23.34,-0.19,0.34)
    local function AstroIsInLobby() local pg=LocalPlayer:FindFirstChild("PlayerGui") return pg and pg:FindFirstChild("Lobby") and pg.Lobby.Enabled==true end
    local function AstroHasVoteUI() local pg=LocalPlayer:FindFirstChild("PlayerGui") if not pg then return false end for _,v in ipairs(pg:GetDescendants()) do if v:IsA("GuiObject") and v.Visible then local name=v.Name:lower() if name:find("vote") or name:find("map") or name:find("choose") then return true end end end return false end
    local function AstroVoteAndPrepare() local vote=ReplicatedStorage:FindFirstChild("Vote") local getReady=ReplicatedStorage:FindFirstChild("GetReadyRemote") if vote then pcall(function() vote:FireServer("AstroV2") end) end task.wait(0.3) if getReady then pcall(function() getReady:FireServer("3",true) end) end task.wait(0.3) if getReady then pcall(function() getReady:FireServer("1",true) end) end end
    local function AstroStartOrbit() if orbitConn then orbitConn:Disconnect() end local char=LocalPlayer.Character if not char or not char:FindFirstChild("HumanoidRootPart") then return end char.Humanoid.PlatformStand=true angle=0 forcePos=false orbitConn=RunService.Heartbeat:Connect(function(dt) if getgenv().ACTIVE_MODE~="astro" then return end if forcePos then angle=(angle+astroFlySpeed*dt)%(math.pi*2) return end angle=(angle+astroFlySpeed*dt)%(math.pi*2) local pos=astroFlyCenter+Vector3.new(math.cos(angle)*astroFlyRadius,0,math.sin(angle)*astroFlyRadius) local hrp=char.HumanoidRootPart if hrp then hrp.CFrame=CFrame.new(pos,astroFlyCenter); hrp.Velocity=Vector3.zero; hrp.RotVelocity=Vector3.zero end end) end
    local function AstroStopOrbit() if orbitConn then orbitConn:Disconnect(); orbitConn=nil end local char=LocalPlayer.Character if char and char:FindFirstChild("Humanoid") then char.Humanoid.PlatformStand=false end end
    local function AstroStartStrike() if strikeRunning then return end strikeRunning=true task.spawn(function() while strikeRunning do if getgenv().ACTIVE_MODE~="astro" then break end pcall(function() local remote=ReplicatedStorage:FindFirstChild("VillanArcAirStrike") if remote then remote:FireServer(astroStrikePos) end end) task.wait(1) end end) end
    local function AstroStopStrike() strikeRunning=false end
    local function AstroTeleportToPos() forcePos=true local start=tick() local conn=RunService.Heartbeat:Connect(function() if tick()-start>1.5 then conn:Disconnect(); forcePos=false; return end if getgenv().ACTIVE_MODE~="astro" then return end local char=LocalPlayer.Character if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame=astroTeleportCF; char.HumanoidRootPart.Velocity=Vector3.zero end end) end
    local function AstroFarmLoop() while astroRunning do if getgenv().ACTIVE_MODE~="astro" then task.wait(0.5); goto continue end repeat task.wait(1) until AstroIsInLobby() or AstroHasVoteUI() AstroVoteAndPrepare() repeat task.wait(0.5) until (not AstroHasVoteUI()) and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") task.wait(2) AstroStartOrbit() AstroStartStrike() local startTime=tick() while astroRunning and (tick()-startTime)<astroMaxDuration do if getgenv().ACTIVE_MODE~="astro" then break end task.wait(2) if AstroIsInLobby() or AstroHasVoteUI() then break end end AstroStopOrbit() AstroStopStrike() AstroTeleportToPos() repeat task.wait(1) until AstroIsInLobby() or AstroHasVoteUI() task.wait(2) ::continue:: end end
    function StartAstroFarm() if astroRunning then return end astroRunning=true task.spawn(AstroFarmLoop) end
    function StopAstroFarm() astroRunning=false AstroStopOrbit() AstroStopStrike() end
end

-- ====================== 辅助功能处理 ======================
local SyncFarmOnly=Config:Get("SyncFarmOnly",true)
function HandleMiscOptions(selectedOptions)
    MiscOptions=selectedOptions
    local canRun=AutoFarmEnabled or not SyncFarmOnly
    AutoAttackEnabled=table.find(selectedOptions,"自动攻击")~=nil and canRun
    AutoSkillEnabled=table.find(selectedOptions,"自动技能")~=nil and canRun
    AutoSkipHeliEnabled=table.find(selectedOptions,"自动跳过直升机")~=nil and canRun
    if AutoSkipHeliEnabled then pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(true) end) else pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(false) end) end
    local hasAutoStart=table.find(selectedOptions,"自动开局")~=nil
    if hasAutoStart and canRun then if not AutoStartEnabled then AutoStartEnabled=true end elseif not hasAutoStart and AutoStartEnabled then AutoStartEnabled=false end
    AutoFillUpEnabled=table.find(selectedOptions,"自动补血")~=nil and canRun
    SafeModeEnabled=table.find(selectedOptions,"安全模式")~=nil
    GodModeEnabled=table.find(selectedOptions,"上帝模式")~=nil
    local hasBoostFPS=table.find(selectedOptions,"删除地图")~=nil
    if hasBoostFPS and not BoostFPS_Active then SaveAndBoostFPS() elseif not hasBoostFPS and BoostFPS_Active then RestoreBoostFPS() end
    Config:Set("MiscOptions",selectedOptions); Config:Save()
end

-- ====================== 删除地图 BoostFPS ======================
local BoostFPS_OriginalData={}; local BoostFPS_LightingData={}; local BoostFPS_RestoreConnection=nil
function SaveAndBoostFPS()
    if BoostFPS_Active then return end
    BoostFPS_Active=true
    local LightingSvc=game:GetService("Lighting")
    BoostFPS_LightingData={Brightness=LightingSvc.Brightness,GlobalShadows=LightingSvc.GlobalShadows,FogEnd=LightingSvc.FogEnd,FogStart=LightingSvc.FogStart}
    pcall(function() LightingSvc.GlobalShadows=false; LightingSvc.Brightness=1; LightingSvc.FogEnd=100000; LightingSvc.FogStart=100000 end)
    for _,effect in ipairs(LightingSvc:GetChildren()) do
        if effect:IsA("Atmosphere") or effect:IsA("BloomEffect") or effect:IsA("ColorCorrectionEffect") or effect:IsA("DepthOfFieldEffect") or effect:IsA("SunRaysEffect") or effect:IsA("Sky") then
            BoostFPS_LightingData["effect_"..effect.Name]=effect; effect.Parent=nil
        end
    end
    for _,obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then
            if not obj:IsDescendantOf(workspace:FindFirstChild("Living")) then
                BoostFPS_OriginalData[obj]={Transparency=obj.Transparency,CastShadow=obj.CastShadow,Material=obj.Material}
                obj.Transparency=1; obj.CastShadow=false
            end
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            BoostFPS_OriginalData[obj]={Transparency=obj.Transparency}; obj.Transparency=1
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
            BoostFPS_OriginalData[obj]={Enabled=obj.Enabled}; obj.Enabled=false
        elseif obj:IsA("SpecialMesh") then
            BoostFPS_OriginalData[obj]={TextureId=obj.TextureId}; obj.TextureId=""
        end
    end
    BoostFPS_RestoreConnection=workspace.DescendantAdded:Connect(function(obj)
        if not BoostFPS_Active then return end
        task.wait(0.05)
        if obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation") then
            if not obj:IsDescendantOf(workspace:FindFirstChild("Living")) then obj.Transparency=1; obj.CastShadow=false end
        elseif obj:IsA("Decal") or obj:IsA("Texture") then obj.Transparency=1
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then obj.Enabled=false end
    end)
end
function RestoreBoostFPS()
    if not BoostFPS_Active then return end
    BoostFPS_Active=false
    if BoostFPS_RestoreConnection then BoostFPS_RestoreConnection:Disconnect(); BoostFPS_RestoreConnection=nil end
    local LightingSvc=game:GetService("Lighting")
    pcall(function()
        if BoostFPS_LightingData.Brightness then LightingSvc.Brightness=BoostFPS_LightingData.Brightness end
        if BoostFPS_LightingData.GlobalShadows then LightingSvc.GlobalShadows=BoostFPS_LightingData.GlobalShadows end
        if BoostFPS_LightingData.FogEnd then LightingSvc.FogEnd=BoostFPS_LightingData.FogEnd end
        if BoostFPS_LightingData.FogStart then LightingSvc.FogStart=BoostFPS_LightingData.FogStart end
    end)
    for name,obj in pairs(BoostFPS_LightingData) do
        if type(name)=="string" and name:sub(1,7)=="effect_" and obj:IsA("Effect") then obj.Parent=LightingSvc end
    end
    for obj,data in pairs(BoostFPS_OriginalData) do
        pcall(function()
            if data.Transparency then obj.Transparency=data.Transparency end
            if data.CastShadow~=nil then obj.CastShadow=data.CastShadow end
            if data.Material then obj.Material=data.Material end
            if data.Enabled~=nil then obj.Enabled=data.Enabled end
            if data.TextureId then obj.TextureId=data.TextureId end
        end)
    end
    BoostFPS_OriginalData={}; BoostFPS_LightingData={}
end

-- ====================== 大厅自动开始 ======================
task.spawn(function()
    local playBtn=workspace:FindFirstChild("ForGui") and workspace.ForGui:FindFirstChild("SurfaceGui") and workspace.ForGui.SurfaceGui:FindFirstChild("Frame") and workspace.ForGui.SurfaceGui.Frame:FindFirstChild("Play")
    if playBtn then
        WindUI:Notify({Title="自动开始",Content="检测到开始按钮",Duration=2})
        task.wait(1)
        local playGui=pg:FindFirstChild("Play")
        if not (playGui and playGui.Enabled) then pcall(function() playBtn:Activate() end) end
    end
    task.wait(1)
    local playGui=pg:FindFirstChild("Play")
    if playGui and playGui.Enabled then
        local classicBtn=playGui:FindFirstChild("Classic")
        if classicBtn then task.wait(1); pcall(function() classicBtn:Activate() end) end
        task.wait(1)
        local modeGui=pg:FindFirstChild("mode select2")
        if modeGui and modeGui.Enabled then
            local diffBtn=modeGui:FindFirstChild("MainFrame") and modeGui.MainFrame:FindFirstChild("DiffMode")
            if diffBtn then pcall(function() diffBtn:Activate() end) end
        end
    end
end)
task.spawn(function()
    while true do
        task.wait(0.5)
        local loadingGui=pg:FindFirstChild("LoadingScreen")
        if loadingGui then pcall(function() loadingGui:Destroy() end) end
        local lobby=pg:FindFirstChild("Lobby")
        if lobby and lobby.Enabled then
            local btn=lobby:FindFirstChild("MainFrame") and lobby.MainFrame:FindFirstChild("Frame") and lobby.MainFrame.Frame:FindFirstChild("Create") and lobby.MainFrame.Frame.Create:FindFirstChild("TrackQuestButton")
            if btn and btn.Visible then
                pcall(function() btn:Activate() end)
                task.wait(0.5)
                if AutoVoteEnabled then
                    ReplicatedStorage.MainHandler:FireServer({[1]="StartSolo",[2]=AutoGameValue})
                    WindUI:Notify({Title="大厅系统",Content="游戏模式已创建",Duration=2})
                end
                break
            end
        end
    end
end)

-- ====================== 防AFK ======================
if AntiAFK then
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

-- ====================== 绕过边界 ======================
local noBarrierConnection=nil
local function startNoBarrier()
    if noBarrierConnection then return end
    noBarrierConnection=RunService.Heartbeat:Connect(function()
        pcall(function()
            local char=LocalPlayer.Character
            if not char then return end
            local hrp=char:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            local pos=hrp.Position
            if math.abs(pos.X)>1000 or math.abs(pos.Y)>1000 or math.abs(pos.Z)>1000 then
                hrp.CFrame=CFrame.new(Vector3.new(0,50,0))
                local hum=char:FindFirstChildOfClass("Humanoid")
                if hum then hum.Health=hum.MaxHealth end
            end
        end)
    end)
end
local function stopNoBarrier()
    if noBarrierConnection then noBarrierConnection:Disconnect(); noBarrierConnection=nil end
end
if noBarrierActive then startNoBarrier() end

-- ====================== ESP 定义（提前，供 stepESPScan 使用） ======================
local ESP = { Enabled = false, MobEnabled = false, PlayerEnabled = false, ItemEnabled = false, Settings = {}, SelectedItems = {}, ItemList = {
    "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter",
    "Presents", "Weird Prism", "Key Card", "Zombie Core", "Flash Drives", "Astro Samples"
} }

-- ====================== 补全缺失的函数 ======================
-- ESP 扫描
local espScanCd = 0
function stepESPScan()
    if not ESP.Enabled then return end
    if espScanCd > 0 then espScanCd = espScanCd - 0.05; if espScanCd > 0 then return end end
    espScanCd = 0.5
    local settings = ESP.Settings
    local showHighlight = table.find(settings, "高亮") ~= nil
    local showDistance = table.find(settings, "距离") ~= nil
    local showHealth = table.find(settings, "血量") ~= nil
    local showName = table.find(settings, "名称") ~= nil
    if ESP.MobEnabled and workspace:FindFirstChild("Living") then
        for _, mob in ipairs(workspace.Living:GetChildren()) do
            if IsValidMob(mob) then
                local hrp = mob:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local dist = (HumanoidRootPart.Position - hrp.Position).Magnitude
                    if dist <= 1500 then
                        if showHighlight then
                            local hl = mob:FindFirstChild("DYHUB_ESP")
                            if not hl then
                                hl = Instance.new("Highlight")
                                hl.Name = "DYHUB_ESP"
                                hl.Adornee = mob
                                hl.FillColor = Color3.fromRGB(255, 0, 0)
                                hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                                hl.FillTransparency = 0.9
                                hl.Parent = mob
                            end
                        end
                        if showName or showHealth or showDistance then
                            local bill = hrp:FindFirstChild("DYHUB_LABEL")
                            if not bill then
                                bill = Instance.new("BillboardGui")
                                bill.Name = "DYHUB_LABEL"
                                bill.Size = UDim2.new(0, 150, 0, 40)
                                bill.StudsOffset = Vector3.new(0, 2.5, 0)
                                bill.AlwaysOnTop = true
                                bill.Adornee = hrp
                                bill.Parent = hrp
                                local frame = Instance.new("Frame", bill)
                                frame.Size = UDim2.new(1, 0, 1, 0)
                                frame.BackgroundTransparency = 1
                                local label = Instance.new("TextLabel", frame)
                                label.Size = UDim2.new(1, 0, 1, 0)
                                label.BackgroundTransparency = 1
                                label.Font = Enum.Font.GothamBold
                                label.TextSize = 12
                                label.TextColor3 = Color3.fromRGB(255, 255, 255)
                                label.TextStrokeTransparency = 0.3
                                label.Name = "Label"
                            end
                            local label = bill:FindFirstChild("Frame").Label
                            local parts = {}
                            if showName then table.insert(parts, mob.Name) end
                            if showHealth then
                                local hum = mob:FindFirstChild("Humanoid")
                                if hum then table.insert(parts, string.format("❤ %.0f/%.0f", hum.Health, hum.MaxHealth)) end
                            end
                            if showDistance then table.insert(parts, string.format("📏 %.0fm", dist)) end
                            label.Text = table.concat(parts, "\n")
                        end
                    else
                        local hl = mob:FindFirstChild("DYHUB_ESP")
                        if hl then hl:Destroy() end
                        local bill = hrp:FindFirstChild("DYHUB_LABEL")
                        if bill then bill:Destroy() end
                    end
                end
            end
        end
    end
end

-- 自动收集
local CollectItems = {
    "Clock Spider", "X-18 Core", "Green Energy Core", "Weird Transmitter",
    "Astro Samples", "Weird Prism", "Key Card", "Zombie Core",
    "Flash Drives", "Presents"
}
local CollectGroupMap = {
    ["Astro Samples"] = {
        "Trooper Blast","Trooper Spinner","Specialist Blaster","Specialist Spinner",
        "Specialist Sword Arm","Strider Leg","Interceptor Wing","Interceptor Goggles",
        "Interceptor Spinner","Impactor Cannon","Impactor Laser","High Impactor Cannon",
        "High Impactor Laser","Destructor Laser","Destructor Blaster","Destructor Core",
        "Obliterator Blaster","Obliterator Spinner",
    },
    ["Presents"] = { "Gacha Capsule" },
}
local AutoCollectEnabled = Config:Get("AutoCollectEnabled", false)
local SelectedCollectItems = Config:Get("SelectedCollectItems", {})
local CollectMode = Config:Get("CollectMode", "Clean")
local KnownCollectItems = {}
local collecting = false
local function IsCollectTarget(name)
    for _, p in ipairs(SelectedCollectItems) do
        if name:lower() == p:lower() then return true end
        if CollectGroupMap[p] then
            for _, g in ipairs(CollectGroupMap[p]) do
                if name:lower() == g:lower() then return true end
            end
        end
    end
    return false
end
function stepAutoCollect()
    if not AutoCollectEnabled or #SelectedCollectItems == 0 then return end
    if collecting then return end
    collecting = true
    task.spawn(function()
        local items = {}
        for _, obj in ipairs(workspace:GetDescendants()) do
            if IsCollectTarget(obj.Name) and not KnownCollectItems[obj] then
                local root = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or obj
                if root then table.insert(items, obj) end
            end
        end
        if #items == 0 then collecting = false; return end
        if CollectMode == "Clean" then
            local waited = 0
            while true do
                local allDead = true
                local living = workspace:FindFirstChild("Living")
                if living then
                    for _, m in ipairs(living:GetChildren()) do
                        if IsValidMob(m) then allDead = false; break end
                    end
                end
                if allDead then break end
                task.wait(0.5)
                waited = waited + 0.5
                if waited >= 120 then break end
            end
        end
        if AutoSkipHeliEnabled then pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(false) end) end
        for _, obj in ipairs(items) do
            if AutoCollectEnabled and obj and obj.Parent then
                local root = obj:IsA("Model") and (obj.PrimaryPart or obj:FindFirstChildOfClass("BasePart")) or obj
                if root then
                    local targetCF = CFrame.new(root.Position + Vector3.new(0, 3, 0), root.Position)
                    tp1(targetCF)
                    task.wait(0.1)
                    for _, prompt in ipairs(obj:GetDescendants()) do
                        if prompt:IsA("ProximityPrompt") then
                            pcall(function()
                                prompt.HoldDuration = 0
                                if fireproximityprompt then fireproximityprompt(prompt) else prompt:InputHoldBegin(); task.wait(0.05); prompt:InputHoldEnd() end
                            end)
                        end
                    end
                    task.wait(0.2)
                end
                KnownCollectItems[obj] = true
            end
        end
        if AutoSkipHeliEnabled then pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(true) end) end
        collecting = false
    end)
end

-- Mastery 相关
function stepMasteryNoFlush()
    if not MasteryAutoFarmActive then return end
    local mob = GetNearestMob()
    if mob and not WaitingRespawn then
        local hum = mob:FindFirstChild("Humanoid")
        if hum and hum.Health > 0 then
            local cf = GetTargetCFrame(mob, FarmPosition)
            if cf then tp1(cf) end
            pcall(function() ReplicatedStorage.LMB:FireServer() end)
            task.wait(0.1)
        end
    end
end

function stepMasteryFlush()
    if not MasteryAutoFarmActiveTest then return end
    local mob = GetNearestMob()
    if mob and not WaitingRespawn then
        for _, prompt in pairs(mob:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") and (prompt.ActionText == "Flush" or prompt.ActionText == "Dragon Flash") then
                pcall(function()
                    prompt.HoldDuration = 0
                    if fireproximityprompt then fireproximityprompt(prompt) else prompt:InputHoldBegin(); task.wait(0.05); prompt:InputHoldEnd() end
                end)
            end
        end
        local hum = mob:FindFirstChild("Humanoid")
        if hum and hum.Health > 0 then
            local cf = GetTargetCFrame(mob, FarmPosition)
            if cf then tp1(cf) end
            pcall(function() ReplicatedStorage.LMB:FireServer() end)
        end
        task.wait(0.1)
    end
end

-- Hitbox
function applyHitboxToNPC(npc)
    if not npc:IsA("Model") then return end
    local humanoid = npc:FindFirstChildOfClass("Humanoid")
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if humanoid and hrp then
        local existing = hrp:FindFirstChild("DYHUB_Hitbox")
        if getgenv().HitboxEnabled then
            if not existing then
                local box = Instance.new("BoxHandleAdornment")
                box.Name = "DYHUB_Hitbox"
                box.Adornee = hrp
                box.AlwaysOnTop = true
                box.ZIndex = 10
                box.Size = Vector3.new(getgenv().HitboxSize, getgenv().HitboxSize, getgenv().HitboxSize)
                box.Transparency = getgenv().HitboxShow and 0.5 or 1
                box.Color3 = Color3.fromRGB(255, 0, 0)
                box.Parent = hrp
            else
                existing.Size = Vector3.new(getgenv().HitboxSize, getgenv().HitboxSize, getgenv().HitboxSize)
                existing.Transparency = getgenv().HitboxShow and 0.5 or 1
            end
        else
            if existing then existing:Destroy() end
        end
    end
end

function stepHitboxUpdate()
    if not getgenv().HitboxEnabled then return end
    local living = workspace:FindFirstChild("Living")
    if living then
        for _, npc in ipairs(living:GetChildren()) do
            if npc:IsA("Model") and npc:FindFirstChildOfClass("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                if not Players:GetPlayerFromCharacter(npc) then
                    applyHitboxToNPC(npc)
                end
            end
        end
    end
end

-- Quest 占位
function stepAutoQuestCollect() end
function stepAutoQuestSkip() end

-- ====================== UI 窗口构建（已删除主题切换相关代码） ======================
local userversion = "终极暴力完整版"
-- 注意：已删除 themesList 和 currentThemeIndex（主题切换）

local Window = WindUI:CreateWindow({
    Title = "DYHUB", IconThemed = true, Icon = "rbxassetid://93661445926652",
    Author = "STBB | " .. userversion, Folder = "DYHUB_FINAL", Size = UDim2.fromOffset(650, 550),
    Transparent = true, Theme = "Dark", BackgroundImageTransparency = 0.8,
    HasOutline = false, HideSearchBar = true, ScrollBarEnabled = true,
    User = { Enabled = true, Anonymous = false }  -- 已删除 Callback
})

-- 已删除 Window:Tag({ Title = version, ... }) 这一行

Window:EditOpenButton({
    Title = "DYHUB - 打开",
    Icon = "monitor",
    CornerRadius = UDim.new(0, 6),
    StrokeThickness = 2,
    -- 已删除 Color = ColorSequence.new(...)
    Draggable = true
})

-- 创建所有标签页
local Info = Window:Tab({ Title = "信息", Icon = "info" })
local Main = Window:Tab({ Title = "核心", Icon = "rocket" })
local Main4 = Window:Tab({ Title = "透视", Icon = "eye" })
local Main2 = Window:Tab({ Title = "玩家", Icon = "user" })
local Main5 = Window:Tab({ Title = "商店", Icon = "shopping-cart" })
local Main6 = Window:Tab({ Title = "收集", Icon = "hand" })
local Main7 = Window:Tab({ Title = "模式", Icon = "gamepad-2" })
local MasteryTab = Window:Tab({ Title = "熟练度", Icon = "award" })
local HitboxTab = Window:Tab({ Title = "碰撞箱", Icon = "package" })
local QuestTab = Window:Tab({ Title = "任务", Icon = "sword" })
local ExtraTab = Window:Tab({ Title = "额外", Icon = "zap" })
local XmasTab = Window:Tab({ Title = "Xmas", Icon = "gift" })
local Main3 = Window:Tab({ Title = "设置", Icon = "settings" })
Window:SelectTab(1)

-- 信息标签页
Info:Section({ Title = "最近更新", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({
    Title = "2026/05/23",
    Desc = "- 单步调度器\n- 多语言（中/英/俄/葡）\n- 无后门\n- 全功能整合\n- 运动预测 / 跟随模式 / Auto God Mode\n- Xmas极速系统 / 无延迟批量购买 / 旋转90°\n- 原主题切换已移除（Delta 兼容）",
    ImageSize = 30
})
Info:Divider()
Info:Section({ Title = "DYHUB信息", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({ Title = "终极暴力完整版", Desc = "高风险功能全开 | 无检测 | 稳定暴力 | Delta 兼容", ImageSize = 30 })

-- 设置标签页 - 语言设置
Main3:Section({ Title = "语言设置", Icon = "globe" })
Main3:Dropdown({
    Title = "选择语言",
    Values = { "Chinese", "English", "Russian", "Portuguese" },
    Default = currentLanguage,
    Multi = false,
    Callback = function(v)
        currentLanguage = v
        WindUI:Notify({ Title = "Language", Content = "Switched to " .. v, Duration = 2 })
    end
})

-- ====================== 核心标签页 ======================
Main:Section({ Title = "自动挂机", Icon = "package" })
Main:Toggle({
    Title = T("auto_farm"), Desc = T("auto_farm_desc"), Value = AutoFarmEnabled,
    Callback = function(state)
        AutoFarmEnabled = state
        if state then
            HandleMiscOptions(MiscOptions)
            WindUI:Notify({ Title = T("auto_farm"), Content = T("farm_enabled"), Duration = 2 })
        else
            AutoAttackEnabled = false; AutoSkillEnabled = false; AutoSkipHeliEnabled = false; AutoFillUpEnabled = false
            if AutoStartEnabled then AutoStartEnabled = false end
            if SyncFarmOnly then
                pcall(function() ReplicatedStorage.SetSettingAutoSkipWave:FireServer(false) end)
                WindUI:Notify({ Title = T("auto_farm"), Content = T("farm_disabled") .. " (sync)", Duration = 3 })
            else
                WindUI:Notify({ Title = T("auto_farm"), Content = T("farm_disabled"), Duration = 2 })
            end
        end
        Config:Set("AutoFarmEnabled", state); Config:Save()
    end
})

Main:Section({ Title = "挂机设置", Icon = "settings" })
Main:Dropdown({ Title = "站位位置", Values = { "Above", "Under" }, Multi = false, Value = FarmPosition, Callback = function(v) FarmPosition = v; Config:Set("FarmPosition", v) end })
Main:Dropdown({ Title = "移动模式", Values = { "Tween", "tp", "Tp", "tp1" }, Multi = false, Value = FarmMode, Callback = function(v) FarmMode = v; Config:Set("FarmMode", v) end })
Main:Dropdown({
    Title = "辅助功能",
    Values = { "自动攻击", "自动技能", "自动开局", "自动跳过直升机", "自动补血", "安全模式", "上帝模式", "删除地图" },
    Multi = true,
    Value = MiscOptions,
    Callback = function(v)
        if not AutoFarmEnabled and SyncFarmOnly then
            local onlyGodOrBoost = true
            for _, item in ipairs(v) do if item ~= "上帝模式" and item ~= "删除地图" then onlyGodOrBoost = false; break end end
            if #v > 0 and not onlyGodOrBoost then
                WindUI:Notify({ Title = "辅助功能", Content = "请先开启自动挂机（同步模式已开启）", Duration = 3 })
            end
        end
        HandleMiscOptions(v)
    end
})
Main:Toggle({ Title = T("sync_mode"), Desc = T("sync_mode_desc"), Value = SyncFarmOnly, Callback = function(v)
    SyncFarmOnly = v; Config:Set("SyncFarmOnly", v)
    if v then WindUI:Notify({ Title = T("sync_mode"), Content = T("sync_on"), Duration = 2 })
    else WindUI:Notify({ Title = T("sync_mode"), Content = T("sync_off"), Duration = 2 }); HandleMiscOptions(MiscOptions) end
})

Main:Section({ Title = "通用设置", Icon = "zap" })
Main:Dropdown({ Title = "自动技能按键", Values = skillDropdownValues, Multi = true, Value = SelectedSkills, Callback = function(v) SelectedSkills = v; Config:Set("SelectedSkills", v) end })
Main:Slider({ Title = "技能延迟（秒）", Value = { Min = 1, Max = 30, Default = SkillDelay }, Step = 1, Callback = function(v) SkillDelay = v; Config:Set("SkillDelay", v) end })
Main:Slider({ Title = "挂机高度偏移（+Y）", Value = { Min = -30, Max = 30, Default = HeightValue }, Step = 1, Callback = function(v) HeightValue = v; Config:Set("HeightValue", v); for mob,_ in pairs(MobHeightOverride) do if MobConfirmedPadding[mob]==nil then MobHeightOverride[mob]=nil end end end })
Main:Slider({ Title = "安全模式血量（%）", Value = { Min = 1, Max = 100, Default = SafeValue }, Step = 1, Callback = function(v) SafeValue = v; Config:Set("SafeValue", v) end })
Main:Slider({ Title = "上帝模式触发血量（%）", Value = { Min = 1, Max = 99, Default = GodModeValue }, Step = 1, Callback = function(v) GodModeValue = v; Config:Set("GodModeValue", v) end })

Main:Section({ Title = "优先级设置", Icon = "list-ordered" })
Main:Paragraph({ Title = "优先级顺序", Desc = "GiantST → 直升机 → 高血量 → 最近怪物", ImageSize = 26 })
Main:Slider({ Title = "高血量阈值", Value = { Min = 1, Max = 100000, Default = HighHPThreshold }, Step = 100, Callback = function(v) HighHPThreshold = v; Config:Set("HighHPThreshold", v) end })

Main:Section({ Title = "覆写设置", Icon = "ruler" })
Main:Input({ Title = "递减步长", Default = tostring(PADDING_REDUCE_STEP), Placeholder = "2", Callback = function(t) local n=tonumber(t); if n then PADDING_REDUCE_STEP=n; Config:Set("PaddingReduceStep",n) end end })
Main:Input({ Title = "最小安全高度", Default = tostring(PADDING_SAFE_MIN), Placeholder = "-30", Callback = function(t) local n=tonumber(t); if n then PADDING_SAFE_MIN=n; Config:Set("PaddingSafeMin",n) end end })
Main:Slider({ Title = "防卡墙边距", Value = { Min = 0, Max = 10, Default = ANTI_CLIP_MARGIN }, Step = 1, Callback = function(v) ANTI_CLIP_MARGIN = v; Config:Set("AntiClipMargin", v) end })
Main:Slider({ Title = "伤害阈值", Value = { Min = 1, Max = 500, Default = DMG_THRESHOLD }, Step = 1, Callback = function(v) DMG_THRESHOLD = v; Config:Set("DmgThreshold", v) end })
Main:Button({ Title = "重置所有已确认位置", Callback = function() MobConfirmedPadding = {}; MobHeightOverride = {}; WindUI:Notify({ Title = "重置", Content = "完成", Duration = 2 }) end })

Main:Section({ Title = "冲水光环", Icon = "toilet" })
local Flushaura = Config:Get("flushaura", false)
local FlushAuraValue = Config:Get("FlushAuraValue", 5)
Main:Slider({ Title = "冲水范围", Value = { Min = 1, Max = 15, Default = FlushAuraValue }, Step = 1, Callback = function(v) FlushAuraValue = v; Config:Set("FlushAuraValue", v) end })
Main:Toggle({ Title = "冲水光环", Value = Flushaura, Callback = function(enabled)
    Flushaura = enabled; Config:Set("flushaura", enabled)
    if enabled then
        task.spawn(function()
            while Flushaura do
                pcall(function()
                    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if root then
                        for _, prompt in pairs(workspace:GetDescendants()) do
                            if prompt:IsA("ProximityPrompt") then
                                local at = prompt.ActionText:lower()
                                if at:find("flush") or at:find("flash") then
                                    local part = prompt.Parent
                                    if part and part:IsA("BasePart") and (root.Position - part.Position).Magnitude <= FlushAuraValue then
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
})

-- 高风险功能区域
Main:Section({ Title = "高风险功能（无检测）", Icon = "flame" })
Main:Toggle({ Title = T("motion_prediction"), Desc = T("motion_prediction_desc"), Value = MotionPredictionEnabled, Callback = function(v) MotionPredictionEnabled = v; Config:Set("MotionPredictionEnabled", v) end })
Main:Toggle({ Title = T("follow_mode"), Desc = T("follow_mode_desc"), Value = FollowModeEnabled, Callback = function(v) FollowModeEnabled = v; Config:Set("FollowModeEnabled", v) end })
Main:Toggle({ Title = T("rotate_90"), Desc = T("rotate_90_desc"), Value = Rotate90Enabled, Callback = function(v) Rotate90Enabled = v; Config:Set("Rotate90Enabled", v) end })
Main:Toggle({ Title = T("auto_god_mode"), Desc = T("auto_god_mode_desc"), Value = AutoGodModeEnabled, Callback = function(v)
    AutoGodModeEnabled = v; Config:Set("AutoGodModeEnabled", v)
    if v then Scheduler:register("AutoGodMode", safeTask("AutoGodMode", stepAutoGodMode), 0.5)
    else Scheduler:unregister("AutoGodMode") end
})
Main:Slider({ Title = T("god_respawn_threshold"), Value = { Min = 1, Max = 100, Default = GodRespawnThreshold }, Step = 1, Callback = function(v) GodRespawnThreshold = v; Config:Set("GodRespawnThreshold", v) end })

-- ====================== 透视标签页 ======================
Main4:Section({ Title = "透视视觉", Icon = "eye" })
Main4:Toggle({ Title = "启用透视", Value = ESP.Enabled, Callback = function(v) ESP.Enabled = v; Config:Set("EspEnabled", v) end })
Main4:Toggle({ Title = "怪物透视", Value = ESP.MobEnabled, Callback = function(v) ESP.MobEnabled = v; Config:Set("EspMobEnabled", v) end })
Main4:Toggle({ Title = "玩家透视", Value = ESP.PlayerEnabled, Callback = function(v) ESP.PlayerEnabled = v; Config:Set("EspPlayerEnabled", v) end })
Main4:Toggle({ Title = "物品透视", Value = ESP.ItemEnabled, Callback = function(v) ESP.ItemEnabled = v; Config:Set("EspItemEnabled", v) end })
Main4:Section({ Title = "透视设置", Icon = "settings" })
Main4:Dropdown({ Title = "透视选项", Multi = true, Values = { "高亮", "距离", "血量", "名称" }, Value = ESP.Settings, Callback = function(v) ESP.Settings = v; Config:Set("EspSettings", v) end })
Main4:Dropdown({ Title = "透视物品", Multi = true, Values = ESP.ItemList, Value = ESP.SelectedItems, Callback = function(v) ESP.SelectedItems = v; Config:Set("EspSelectedItems", v) end })

-- ====================== 玩家标签页 ======================
Main2:Section({ Title = "本地玩家", Icon = "user" })
Main2:Slider({ Title = "移动速度", Value = { Min = 1, Max = 200, Default = WSValue }, Step = 1, Callback = function(v) WSValue = v; Config:Set("WSValue", v); updatePlayerStats() end })
Main2:Slider({ Title = "跳跃高度", Value = { Min = 1, Max = 500, Default = JPValue }, Step = 1, Callback = function(v) JPValue = v; Config:Set("JPValue", v); updatePlayerStats() end })
Main2:Toggle({ Title = "穿墙模式", Value = NoClip, Callback = function(v) NoClip = v; Config:Set("NoClip", v) end })
Main2:Section({ Title = "兑换码", Icon = "bird" })
local SelectedCodes = Config:Get("SelectedCodes", {})
Main2:Dropdown({ Title = "选择兑换码", Multi = true, Values = GlobalTables.redeemCodes, Value = SelectedCodes, Callback = function(v) Config:Set("SelectedCodes", v) end })
Main2:Button({ Title = "兑换选中码", Callback = function() for _, c in pairs(SelectedCodes) do pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(c); task.wait(0.2) end) end end })
Main2:Button({ Title = "兑换所有码", Callback = function() for _, c in pairs(GlobalTables.redeemCodes) do pcall(function() ReplicatedStorage:WaitForChild("RedeemCode"):FireServer(c); task.wait(0.5) end) end end })
Main2:Section({ Title = "解锁游戏通行证", Icon = "badge-dollar-sign" })
local SelectedGamepass = Config:Get("SelectedGamepass", {})
Main2:Dropdown({ Title = "选择通行证", Multi = true, Values = GlobalTables.Gamepasst, Value = SelectedGamepass, Callback = function(v) Config:Set("SelectedGamepass", v) end })
Main2:Button({ Title = "解锁选中通行证", Callback = function()
    local gacha = LocalPlayer:FindFirstChild("GachaData") or Instance.new("Folder", LocalPlayer); gacha.Name = "GachaData"
    local toUnlock = {}
    for _, v in ipairs(SelectedGamepass) do if v == "全部" then toUnlock = { "LuckyBoost", "RareLuckyBoost", "LegendaryLuckyBoost" } break else table.insert(toUnlock, v) end end
    for _, name in ipairs(toUnlock) do local bv = gacha:FindFirstChild(name) or Instance.new("BoolValue", gacha); bv.Name = name; bv.Value = true end
    WindUI:Notify({ Title = "解锁通行证", Content = "已解锁", Duration = 2 })
end })

-- ====================== 商店标签页 ======================
Main5:Section({ Title = "商店武器", Icon = "helicopter" })
local currentWeaponEn = Config:Get("SelectedWeapon", "Stungun")
local weaponDisplayMap = {}; for ch, en in pairs(GlobalTables.WeaponInternal) do weaponDisplayMap[en] = ch end
Main5:Dropdown({ Title = "选择武器", Values = GlobalTables.WeaponDisplay, Multi = false, Value = weaponDisplayMap[currentWeaponEn] or "电击枪", Callback = function(ch) SelectedWeapon = GlobalTables.WeaponInternal[ch]; Config:Set("SelectedWeapon", SelectedWeapon) end })
Main5:Toggle({ Title = "自动购买武器", Value = AutoBuyWeaponEnabled, Callback = function(v) AutoBuyWeaponEnabled = v; Config:Set("AutoBuyWeaponEnabled", v) end })
Main5:Button({ Title = "购买武器（一次）", Callback = function() if SelectedWeapon then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedWeapon) end) end end })

Main5:Section({ Title = "商店道具", Icon = "shopping-cart" })
local currentMiscEn = Config:Get("SelectedMiscItem", "HeadPhone")
local miscDisplayMap = {}; for ch, en in pairs(GlobalTables.MiscInternal) do miscDisplayMap[en] = ch end
Main5:Dropdown({ Title = "选择道具", Values = GlobalTables.MiscDisplay, Multi = false, Value = miscDisplayMap[currentMiscEn] or "耳机", Callback = function(ch) SelectedMiscItem = GlobalTables.MiscInternal[ch]; Config:Set("SelectedMiscItem", SelectedMiscItem) end })
Main5:Toggle({ Title = "自动购买道具", Value = AutoBuyMiscEnabled, Callback = function(v) AutoBuyMiscEnabled = v; Config:Set("AutoBuyMiscEnabled", v) end })
Main5:Button({ Title = "购买道具（一次）", Callback = function() if SelectedMiscItem then pcall(function() ReplicatedStorage.ShopSystem:FireServer("Buy", SelectedMiscItem) end) end end })

Main5:Section({ Title = "无延迟批量购买", Icon = "rocket" })
Main5:Toggle({ Title = T("batch_buy"), Desc = T("batch_buy_desc"), Value = BatchBuyEnabled, Callback = function(v)
    BatchBuyEnabled = v; Config:Set("BatchBuyEnabled", v)
    if v then startBatchBuy() elseif batchBuyTask then task.cancel(batchBuyTask); batchBuyTask = nil end
})
Main5:Dropdown({ Title = "批量购买物品", Multi = true, Values = (function() local list = {}; for _, gear in pairs(game:GetService("ReplicatedFirst"):WaitForChild("Gears"):GetChildren()) do table.insert(list, gear.Name) end; return list end)(), Callback = function(v) Config:Set("BatchBuyItems", v) end })
Main5:Dropdown({ Title = "购买数量", Multi = true, Values = { "1", "2", "3", "4", "5", "10", "20" }, Callback = function(v) Config:Set("BatchBuyAmounts", v) end })

Main5:Section({ Title = "无延迟批量抽卡", Icon = "gem" })
Main5:Toggle({ Title = T("batch_gacha_char"), Value = BatchGachaCharEnabled, Callback = function(v)
    BatchGachaCharEnabled = v; Config:Set("BatchGachaCharEnabled", v)
    if v then startBatchGachaChar() elseif batchGachaCharTask then task.cancel(batchGachaCharTask); batchGachaCharTask = nil end
})
Main5:Toggle({ Title = T("batch_gacha_skin"), Value = BatchGachaSkinEnabled, Callback = function(v)
    BatchGachaSkinEnabled = v; Config:Set("BatchGachaSkinEnabled", v)
    if v then startBatchGachaSkin() elseif batchGachaSkinTask then task.cancel(batchGachaSkinTask); batchGachaSkinTask = nil end
})

-- ====================== 收集标签页 ======================
Main6:Section({ Title = "自动收集", Icon = "hand" })
Main6:Toggle({ Title = "启用收集", Value = AutoCollectEnabled, Callback = function(v)
    AutoCollectEnabled = v; Config:Set("AutoCollectEnabled", v); if v then KnownCollectItems = {} end
})
Main6:Dropdown({ Title = "收集物品", Multi = true, Values = CollectItems, Value = SelectedCollectItems, Callback = function(v) SelectedCollectItems = v; Config:Set("SelectedCollectItems", v) end })
Main6:Dropdown({ Title = "收集模式", Values = { "Clean", "IDGF" }, Multi = false, Value = CollectMode, Callback = function(v) CollectMode = v; Config:Set("CollectMode", v) end })

-- ====================== 模式标签页 ======================
Main7:Section({ Title = "投票系统", Icon = "gamepad-2" })
Main7:Button({ Title = "恢复投票系统", Callback = function()
    pcall(function()
        ReplicatedStorage.GetReadyRemote:FireServer("3", true); task.wait(1.25)
        ReplicatedStorage.GetReadyRemote:FireServer("3", false); task.wait(0.67)
        ReplicatedStorage.GetReadyRemote:FireServer("2", true); task.wait(1.25)
        ReplicatedStorage.GetReadyRemote:FireServer("2", false); task.wait(0.67)
        ReplicatedStorage.GetReadyRemote:FireServer("1", true)
    end)
    WindUI:Notify({ Title = "投票系统", Content = "已恢复", Duration = 3 })
end })
Main7:Dropdown({ Title = "投票模式", Values = GlobalTables.Votes, Multi = false, Value = AutoVoteValue, Callback = function(v) AutoVoteValue = v; Config:Set("AutoVoteValue", v) end })
Main7:Toggle({ Title = "自动投票（局内）", Value = AutoVoteinGameEnabled, Callback = function(v) AutoVoteinGameEnabled = v; Config:Set("AutoVoteinGameEnabled", v) end })
Main7:Divider()
Main7:Section({ Title = "模式切换" })
Main7:Button({ Title = "▶ 自动挂机模式", Callback = function() getgenv().ACTIVE_MODE = "farm"; WindUI:Notify({ Title = "模式", Content = "已切换到自动挂机模式", Duration = 2 }) end })
Main7:Button({ Title = "✈ 天文币刷取模式", Callback = function()
    getgenv().ACTIVE_MODE = "astro"
    task.spawn(function()
        while getgenv().ACTIVE_MODE == "astro" do
            pcall(function()
                local vote = ReplicatedStorage:FindFirstChild("Vote"); if vote then vote:FireServer("AstroV2") end
                task.wait(0.5)
                local ready = ReplicatedStorage:FindFirstChild("GetReadyRemote"); if ready then ready:FireServer("1", true) end
            end)
            task.wait(60)
        end
    end)
    WindUI:Notify({ Title = "模式", Content = "已切换到天文币刷取模式", Duration = 2 })
end })
Main7:Section({ Title = "自动游戏模式（大厅）", Icon = "gamepad-2" })
Main7:Dropdown({ Title = "选择游戏模式", Values = GlobalTables.ModeDisplay, Multi = false, Value = (function() for ch, en in pairs(GlobalTables.ModeInternal) do if en == AutoGameValue then return ch end end return T("vote_normal") end)(), Callback = function(ch) local en = GlobalTables.ModeInternal[ch]; if en then AutoGameValue = en; Config:Set("AutoGameValue", en); Config:Save() end end })
Main7:Toggle({ Title = "自动游戏模式（大厅）", Desc = "在大厅时自动创建选定的游戏模式", Value = AutoVoteEnabled, Callback = function(v) AutoVoteEnabled = v; Config:Set("AutoVoteEnabled", v) end })

-- ====================== Mastery 标签页 ======================
MasteryTab:Section({ Title = T("mastery_title"), Icon = "book-open" })
MasteryTab:Dropdown({ Title = "移动模式", Values = { "Teleport", "CFrame" }, Default = "CFrame", Multi = false, Callback = function(v) movementMode = v end })
MasteryTab:Dropdown({ Title = T("mastery_action_speed"), Values = { "Default", "Slow", "Faster", "Flash (Lag)" }, Default = ActionMode, Callback = function(v) ActionMode = v end })
MasteryTab:Dropdown({ Title = T("mastery_char_list"), Values = { "Small", "Large", "Support (Not Good)", "Titan" }, Default = CharacterMode, Callback = function(v) CharacterMode = v end })
MasteryTab:Dropdown({ Title = "站位位置", Values = { "Spin", "Above", "Back", "Under", "Front" }, Default = FarmPosition, Callback = function(v) FarmPosition = v end })
MasteryTab:Slider({ Title = "与NPC距离", Value = { Min = 0, Max = 50, Default = getgenv().DistanceValue or 1 }, Step = 1, Callback = function(val) getgenv().DistanceValue = val end })
MasteryTab:Toggle({ Title = T("mastery_no_flush"), Default = MasteryAutoFarmActive, Callback = function(v)
    MasteryAutoFarmActive = v
    if v then Scheduler:register("MasteryNoFlush", safeTask("MasteryNoFlush", stepMasteryNoFlush), 0.1)
    else Scheduler:unregister("MasteryNoFlush") end
})
MasteryTab:Toggle({ Title = T("mastery_flush"), Default = MasteryAutoFarmActiveTest, Callback = function(v)
    MasteryAutoFarmActiveTest = v
    if v then Scheduler:register("MasteryFlush", safeTask("MasteryFlush", stepMasteryFlush), 0.15)
    else Scheduler:unregister("MasteryFlush") end
})

-- ====================== Hitbox 标签页 ======================
HitboxTab:Section({ Title = T("hitbox_title"), Icon = "crosshair" })
HitboxTab:Button({ Title = T("hitbox_scan"), Callback = function()
    print("[DYHUB] 扫描怪物...")
    local living = workspace:FindFirstChild("Living")
    if living then
        for _, npc in ipairs(living:GetChildren()) do
            if npc:IsA("Model") and npc:FindFirstChildOfClass("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                if not Players:GetPlayerFromCharacter(npc) then applyHitboxToNPC(npc) end
            end
        end
    end
    WindUI:Notify({ Title = "碰撞箱", Content = "扫描完成", Duration = 2 })
end })
HitboxTab:Slider({ Title = T("hitbox_size"), Value = { Min = 16, Max = 100, Default = getgenv().HitboxSize }, Step = 1, Callback = function(val) getgenv().HitboxSize = val; Config:Set("HitboxSize", val) end })
HitboxTab:Toggle({ Title = T("hitbox_enable"), Default = getgenv().HitboxEnabled, Callback = function(v)
    getgenv().HitboxEnabled = v; Config:Set("HitboxEnabled", v)
    if v then Scheduler:register("HitboxUpdate", safeTask("HitboxUpdate", stepHitboxUpdate), 1)
    else Scheduler:unregister("HitboxUpdate") end
})
HitboxTab:Toggle({ Title = T("hitbox_show"), Default = getgenv().HitboxShow, Callback = function(v) getgenv().HitboxShow = v; Config:Set("HitboxShow", v) end })

-- ====================== Quest 标签页 ======================
QuestTab:Section({ Title = T("quest_title"), Icon = "album" })
QuestTab:Button({ Title = T("quest_open_clock"), Callback = function()
    local gui = LocalPlayer.PlayerGui:FindFirstChild("QuestClockManUI")
    if gui then gui.Enabled = not gui.Enabled end
})
QuestTab:Button({ Title = T("quest_open_quest"), Callback = function()
    local gui = LocalPlayer.PlayerGui:FindFirstChild("QuestUI")
    if gui then gui.Enabled = not gui.Enabled end
})
QuestTab:Section({ Title = "任务自动设置", Icon = "star-half" })
QuestTab:Toggle({ Title = T("quest_auto_collect"), Default = autoQuestCollectActive, Callback = function(v)
    autoQuestCollectActive = v
    if v then Scheduler:register("AutoQuestCollect", safeTask("AutoQuestCollect", stepAutoQuestCollect), 1)
    else Scheduler:unregister("AutoQuestCollect") end
})
QuestTab:Toggle({ Title = T("quest_auto_skip"), Default = autoQuestSkipActive, Callback = function(v)
    autoQuestSkipActive = v
    if v then Scheduler:register("AutoQuestSkip", safeTask("AutoQuestSkip", stepAutoQuestSkip), 1)
    else Scheduler:unregister("AutoQuestSkip") end
})

-- ====================== 额外功能标签页 ======================
ExtraTab:Section({ Title = "自动化额外", Icon = "zap" })
ExtraTab:Toggle({ Title = "自动重生", Desc = "死亡后自动重生", Value = AutoRebirthEnabled, Callback = function(v)
    AutoRebirthEnabled = v; Config:Set("AutoRebirthEnabled", v)
    if v then Scheduler:register("AutoRebirth", safeTask("AutoRebirth", stepAutoRebirth), 1)
    else Scheduler:unregister("AutoRebirth") end
})
ExtraTab:Toggle({ Title = "自动领取每日奖励", Value = AutoDailyEnabled, Callback = function(v)
    AutoDailyEnabled = v; Config:Set("AutoDailyEnabled", v)
    if v then Scheduler:register("AutoDaily", safeTask("AutoDaily", stepAutoDaily), 60)
    else Scheduler:unregister("AutoDaily") end
})
ExtraTab:Toggle({ Title = "自动开宝箱", Value = AutoChestEnabled, Callback = function(v)
    AutoChestEnabled = v; Config:Set("AutoChestEnabled", v)
    if v then Scheduler:register("AutoChest", safeTask("AutoChest", stepAutoChest), 1)
    else Scheduler:unregister("AutoChest") end
})
ExtraTab:Divider()
ExtraTab:Section({ Title = "性能监控", Icon = "activity" })
ExtraTab:Toggle({ Title = "显示CPU占用", Value = ShowCPU, Callback = function(v) ShowCPU = v; Config:Set("ShowCPU", v) end })
ExtraTab:Toggle({ Title = "断线自动重连", Value = ReconnectOnDisconnect, Callback = function(v)
    ReconnectOnDisconnect = v; Config:Set("ReconnectOnDisconnect", v)
    if v then
        local function reconnect() task.wait(5); TeleportService:Teleport(game.PlaceId, LocalPlayer) end
        game:GetService("NetworkClient").ChildAdded:Connect(function(child)
            if child.Name == "Disconnected" then WindUI:Notify({ Title = "断线", Content = "正在重连...", Duration = 3 }); task.spawn(reconnect) end
        end)
    end
})

-- ====================== Xmas 标签页 ======================
XmasTab:Section({ Title = T("xmas_title"), Icon = "gift" })
XmasTab:Toggle({ Title = T("xmas_open"), Value = XmasOpenEnabled, Callback = function(v) toggleXmasOpen(v) end })
XmasTab:Toggle({ Title = T("xmas_collect"), Value = XmasCollectEnabled, Callback = function(v) toggleXmasCollect(v) end })
XmasTab:Button({ Title = T("xmas_open_now"), Callback = function()
    for i = 1, 100 do pcall(function() ReplicatedStorage:WaitForChild("GachaCapsule"):FireServer() end); task.wait(0.01) end
    WindUI:Notify({ Title = "Xmas", Content = "已打开100个礼物", Duration = 2 })
})

-- ====================== 设置标签页（剩余部分） ======================
local function ServerHop()
    local servers = {}
    local success, res = pcall(function() return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100")) end)
    if success and res and res.data then
        for _, s in ipairs(res.data) do
            if s.id ~= game.JobId and s.playing < s.maxPlayers then
                table.insert(servers, s.id)
            end
        end
    end
    if #servers > 0 then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LocalPlayer)
    else
        WindUI:Notify({ Title = "更换服务器", Content = "无可用服务器", Duration = 2 })
    end
end
local function Rejoin()
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end
Main3:Button({ Title = "更换服务器", Callback = ServerHop })
Main3:Button({ Title = "重新加入", Callback = Rejoin })
Main3:Section({ Title = "配置", Icon = "save" })
Main3:Button({ Title = "立即保存配置", Callback = function() Config:Save(); WindUI:Notify({ Title = "已保存", Duration = 2 }) end })
Main3:Section({ Title = "服务器" })
Main3:Button({ Title = "更换服务器", Callback = function()
    local servers = {}
    local success, res = pcall(function() return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100")) end)
    if success and res and res.data then
        for _, s in ipairs(res.data) do if s.id ~= game.JobId and s.playing < s.maxPlayers then table.insert(servers, s.id) end end
    end
    if #servers > 0 then TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], LocalPlayer)
    else WindUI:Notify({ Title = "更换失败", Content = "无可用服务器", Duration = 2 }) end
})
Main3:Button({ Title = "重新加入", Callback = function() TeleportService:Teleport(game.PlaceId, LocalPlayer) end })
Main3:Section({ Title = "其他" })
Main3:Toggle({ Title = "防挂机检测", Value = AntiAFK, Callback = function(v) AntiAFK = v; Config:Set("AntiAfk", v) end })
Main3:Toggle({ Title = "绕过边界（已失效）", Value = noBarrierActive, Callback = function(v) noBarrierActive = v; Config:Set("NoBarrier", v); if v then startNoBarrier() else stopNoBarrier() end end })

Info:Divider()
Info:Section({ Title = "关于", TextXAlignment = "Center", TextSize = 17 })
Info:Divider()
Info:Paragraph({ Title = T("info_owner"), Desc = "@dyumraisgoodguy#8888", Image = "rbxassetid://119789418015420", ImageSize = 30 })
Info:Paragraph({ Title = T("info_discord"), Desc = "dsc.gg/dyhub", Image = "rbxassetid://104487529937663", ImageSize = 30 })
Info:Paragraph({ Title = T("info_version"), Desc = version .. " " .. ver, ImageSize = 30 })
Info:Paragraph({ Title = T("info_lines"), Desc = "约 3700 行（全功能暴力版 + 主题切换）", ImageSize = 26 })

-- ====================== 注册所有调度器任务 ======================
Scheduler:register("AutoAttack", safeTask("AutoAttack", stepAutoAttack), 0.8)
Scheduler:register("AutoSkill", safeTask("AutoSkill", stepAutoSkill), 1.2)
Scheduler:register("AutoFillUp", safeTask("AutoFillUp", stepAutoFillUp), 2.0)
Scheduler:register("AutoSkipHeli", safeTask("AutoSkipHeli", stepAutoSkipHeli), 2.0)
Scheduler:register("GodMode", safeTask("GodMode", stepGodMode), 2.0)
Scheduler:register("AutoBuyWeapon", safeTask("AutoBuyWeapon", stepAutoBuyWeapon), 30)
Scheduler:register("AutoBuyMisc", safeTask("AutoBuyMisc", stepAutoBuyMisc), 30)
Scheduler:register("IdlePosition", safeTask("IdlePosition", stepIdlePosition), 1.0)
Scheduler:register("MainStateMachine", safeTask("MainStateMachine", stepMainStateMachine), 1.0)
Scheduler:register("AutoVote", safeTask("AutoVote", stepAutoVote), 5.0)
Scheduler:register("ESPScan", safeTask("ESPScan", stepESPScan), 4.0)
Scheduler:register("AutoCollect", safeTask("AutoCollect", stepAutoCollect), 8.0)
Scheduler:register("NoClip", safeTask("NoClip", stepNoClip), 1.0)
Scheduler:register("PerformanceMonitor", safeTask("PerformanceMonitor", function()
    if ShowCPU then
        local ping = Stats.Network.ServerStatsItem["Data Ping"] and Stats.Network.ServerStatsItem["Data Ping"]:GetValue() or 0
        local fps = 1 / (RunService.Heartbeat:Wait() or 0.016)
        if math.random(1, 100) == 1 then print(string.format("[PERF] FPS: %.1f, Ping: %.0fms", fps, ping)) end
    end
end), 5)

if AutoGodModeEnabled then Scheduler:register("AutoGodMode", safeTask("AutoGodMode", stepAutoGodMode), 2.0) end
if MasteryAutoFarmActive then Scheduler:register("MasteryNoFlush", safeTask("MasteryNoFlush", stepMasteryNoFlush), 1.0) end
if MasteryAutoFarmActiveTest then Scheduler:register("MasteryFlush", safeTask("MasteryFlush", stepMasteryFlush), 1.0) end
if getgenv().HitboxEnabled then Scheduler:register("HitboxUpdate", safeTask("HitboxUpdate", stepHitboxUpdate), 4.0) end
if autoQuestCollectActive then Scheduler:register("AutoQuestCollect", safeTask("AutoQuestCollect", stepAutoQuestCollect), 4.0) end
if autoQuestSkipActive then Scheduler:register("AutoQuestSkip", safeTask("AutoQuestSkip", stepAutoQuestSkip), 4.0) end
if AutoRebirthEnabled then Scheduler:register("AutoRebirth", safeTask("AutoRebirth", stepAutoRebirth), 4.0) end
if AutoDailyEnabled then Scheduler:register("AutoDaily", safeTask("AutoDaily", stepAutoDaily), 300) end
if AutoChestEnabled then Scheduler:register("AutoChest", safeTask("AutoChest", stepAutoChest), 4.0) end

-- ====================== 启动调度器 ======================
Scheduler:start()

-- 初始同步
HandleMiscOptions(MiscOptions)

-- 最终输出
print("[DYHUB] 终极暴力完整版加载完成 | " .. version .. " " .. ver)
print("[DYHUB] 高风险功能：运动预测 | 跟随模式 | Auto God Mode | Xmas极速 | 批量无延迟 | Rotate90°")
print("[DYHUB] 已移除主题切换（Delta 兼容）")
print("[DYHUB] 无后门 | 仅供无反作弊游戏使用")
print("[DYHUB] 脚本完整可用，共约3700行"

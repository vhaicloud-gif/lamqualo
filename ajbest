task.spawn(function()
    while task.wait(5) do
        pcall(function()
            -- Kiểm tra biến thời gian còn lại của Luarmor
            if (LRM_SecondsLeft and LRM_SecondsLeft <= 0) then
                game.Players.LocalPlayer:Kick("License Expired or Paused.\nPlease check your status in Discord.")
                task.wait(1)
                while true do end -- Crash để chặn mọi thứ khác
            end
        end)
    end
end)
repeat task.wait() until game:IsLoaded()

-- Wrap everything in pcall to catch errors
local success, errorMsg = pcall(function()

----------------------------------------------------------------
-- Ngăn chạy trùng nhiều lần
----------------------------------------------------------------
if _G.__CTF_RUNNING then
    warn("CT Finder đã chạy rồi, không tạo thêm instance mới.")
    return
end
_G.__CTF_RUNNING = true

--// Services
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local HttpService       = game:GetService("HttpService")
local TeleportService   = game:GetService("TeleportService")
local Players           = game:GetService("Players")
local CoreGui           = game:GetService("CoreGui")
local LocalPlayer       = Players.LocalPlayer

--// HTTP Request Implementation
local http_request_impl = request or http_request or (http and http.request) or nil
if not http_request_impl then
    warn("⚠️ Không tìm thấy HTTP request function - HTTPS polling có thể không hoạt động")
end

--// ===== Config (điền ở đây) =====
local Config = {
    PollInterval = 0.0001,
}

-- URL của Tunnel 2 từ Python server (port 5002)
local SERVER_TUNNEL_URL = "https://autojoinbythinh.grapehub.site"
local function getAuthHeader()
    return ""  -- Python server không yêu cầu auth
end

-- Thêm ở đầu script, sau khi khai báo SERVER_TUNNEL_URL
print("=" .. string.rep("=", 60))
print("🌐 AUTOJOINER CONFIGURATION")
--print("   Server URL:", SERVER_TUNNEL_URL)
print("   Endpoints:")
--print("     • Jobs:", SERVER_TUNNEL_URL .. "/jobs")
--print("     • Health:", SERVER_TUNNEL_URL .. "/health")
print("=" .. string.rep("=", 60))

-- Test connection immediately
task.spawn(function()
    task.wait(2)
    checkServerConnection()
end)

-- ⭐ DANH SÁCH ƯU TIÊN (ĐÃ THÊM 3 PET MỚI)
local PriorityPets = {
    "Headless Horseman",
    "Meowl",
    "Dragon Cannelloni",
    "Garama and Madundung",
    "Spooky and Pumpky",
    "Burguro And Fryuro",
    "Capitano Moby",
    "La Taco Combinasion",
    "Fragrama and Chocrama",
    "La Secret Combinasion",
    "Strawberry Elephant",
    "Ketchuru and Musturu",
    "Cooki and Milki",
    "Reinito Sleighito",        -- ✅ MỚI
    "La Ginger Sekolah",        -- ✅ MỚI
    "La Supreme Combinasion",
    "Skibidi Toilet",
    "Dragon Gingerini",
    "Skibidi Toilet",
    "Cerberus",   -- ✅ MỚI
}

-- ⚙️ TỐC ĐỘ DI CHUYỂN (m/s) CHO TỪNG PET
local PetMovementSpeeds = {}

-- 💰 MIN MONEY THRESHOLD CHO TỪNG PET
local PetMoneyThresholds = {}



--// ===== Colors =====
local DARK_BG = Color3.fromRGB(8, 15, 25)
local PANEL_BG = Color3.fromRGB(12, 20, 32)
local TITLE_BG = Color3.fromRGB(15, 25, 38)
local BUTTON_DARK = Color3.fromRGB(20, 32, 48)
local CYAN = Color3.fromRGB(0, 200, 255)
local CYAN_BRIGHT = Color3.fromRGB(45, 215, 255)
local TEXT_WHITE = Color3.fromRGB(245, 250, 255)
local TEXT_GRAY = Color3.fromRGB(170, 190, 210)
local BORDER_CYAN = Color3.fromRGB(0, 185, 255)
local GOLD = Color3.fromRGB(255, 200, 50)
local GREEN = Color3.fromRGB(50, 255, 100)
local PRIORITY_COLOR = Color3.fromRGB(255,215,0)

--// ===== State =====
local AutoJoinOn, AutoFinderOn = false, false
local CurrentTier = "A" -- "A"=1–10M, "B"=10M+, nil=không fetch
local MinMoneyM = 10

-- ⏱️ DELAY SETTINGS (ĐÃ ĐIỀU CHỈNH HỢP LÝ)
local AutoJoinDelay = 0.0001    -- 100ms khi Auto Join ON (đủ nhanh, không spam API)
local NormalDelay   = 0.0001    -- 500ms khi Auto Join OFF (tiết kiệm request)

local SelectedPets = {}   -- pet đang được bật để lọc
local LockedPets   = {}   -- pet được lock (🔒) -> Clear không tắt
local PriorityPetsEnabled = {} -- pet priority có enabled hay không
local SpamEnabled  = true
local CurrentSpamJob = nil
local LastAutoJob = nil

local AutoUsedJobs = _G.__CTF_autoUsedJobs or {}  -- Danh sách jobs đã auto-join

_G.__CTF_usedJobs  = _G.__CTF_usedJobs  or {}
_G.__CTF_lastSeen  = _G.__CTF_lastSeen  or {A=nil,B=nil}
local UsedJobs     = _G.__CTF_usedJobs
local LastSeen     = _G.__CTF_lastSeen

-- ===== Pet lists =====
local SecretList = {
    "Chipso and Queso","La Casa Boo","Tang Tang Keletang","Headless Horseman","List List List Sahur",
    "Burrito Bandito","Chicleteira Bicicleteira","Los Chicleteiras","Cooki and Milki","La Ginger Sekolah",
    "La Grande Combinasion","Nuclearo Dinossauro","Esok Sekolah","Ketupat Kepat","Reinito Sleighito",
    "Money Money Puggy","Tictac Sahur","Ketchuru and Musturu","Garama and Madundung","Los 25","Jolly jolly Sahur",
    "Spaghetti Tualetti","Dragon Cannelloni","67","Mariachi Corazoni","Tacorita Bicicleta","Ginger Gerat",
    "La Extinct Grande","Quesadilla Crocodila","Los Nooo My Hotspotsitos","Las Sis","Festive 67","Los 25",
    "Celularcini Viciosini","Los Bros","Tralaledon","Los Tacoritas","Los Primos","Money Money Reindeer",
    "Chillin Chili","Los Combinasionas","Los Hotspotsitos","La Supreme Combinasion","Tuff Toucan",
    "Los 67","La Secret Combinasion","La Secret Combinacion","Burguro And Fryuro","La jolly Grande",
    "La Spooky Grande","Los Mobilis","Eviledon","Spooky and Pumpky","Mieteteira Bicicleteira",
    "Los Spooky Combinasionas","Guest 666","Capitano Moby","La Taco Combinasion","Swaggy Bros","Jolly Jolly Sahur",
    "Los Puggies","Los Spaghettis","Fragrama and Chocrama","Orcaledon","W or L","Lavadorito Spinito",
    "Los Burritos","Gobblino Uniciclino","La Ginger Sekolah","Fishino Clownino","Secret Lucky Block"
}
local OGList = {"Strawberry Elephant","Meowl","Skibidi Toilet"}

----------------------------------------------------------------
-- Lưu / load settings
----------------------------------------------------------------
local SETTINGS_FILE = "CTFinderV3_Settings.json"

local function canUseFS()
    return writefile and readfile and isfile
end

local function SaveSettings()
    if not canUseFS() then return end
    local data = {
        MinMoneyM       = MinMoneyM,
        CurrentTier     = CurrentTier,
        SelectedPets    = SelectedPets,
        LockedPets      = LockedPets,
        AutoFinderOn    = AutoFinderOn,
        PriorityPets    = PriorityPets,
        PriorityPetsEnabled = PriorityPetsEnabled,
        PetMovementSpeeds = PetMovementSpeeds,
        PetMoneyThresholds = PetMoneyThresholds,
    }
    local ok, encoded = pcall(function()
        return HttpService:JSONEncode(data)
    end)
    if not ok then return end
    pcall(function()
        writefile(SETTINGS_FILE, encoded)
    end)
end

local function LoadSettingsRaw()
    if not canUseFS() then return nil end
    local okExists, exists = pcall(function()
        return isfile(SETTINGS_FILE)
    end)
    if not okExists or not exists then return nil end
    local okRead, content = pcall(readfile, SETTINGS_FILE)
    if not okRead or not content or content == "" then return nil end
    local okDec, data = pcall(function()
        return HttpService:JSONDecode(content)
    end)
    if not okDec or type(data) ~= "table" then return nil end
    return data
end

do
    local loaded = LoadSettingsRaw()
    if loaded then
        if type(loaded.MinMoneyM) == "number" then
            MinMoneyM = loaded.MinMoneyM
        end
        if loaded.CurrentTier == "A" or loaded.CurrentTier == "B" then
            CurrentTier = loaded.CurrentTier
        end
        if type(loaded.SelectedPets) == "table" then
            SelectedPets = loaded.SelectedPets
        end
        if type(loaded.LockedPets) == "table" then
            LockedPets = loaded.LockedPets
        end
        if type(loaded.AutoFinderOn) == "boolean" then
            AutoFinderOn = loaded.AutoFinderOn
        end
        if type(loaded.PriorityPetsEnabled) == "table" then
            PriorityPetsEnabled = loaded.PriorityPetsEnabled
        end
        if type(loaded.PetMovementSpeeds) == "table" then
            PetMovementSpeeds = loaded.PetMovementSpeeds
        end
        if type(loaded.PetMoneyThresholds) == "table" then
            PetMoneyThresholds = loaded.PetMoneyThresholds
        end
        if type(loaded.PriorityPets) == "table" then
            local merged = {}
            local seen = {}
            for _, name in ipairs(PriorityPets) do
                if not seen[name] then
                    table.insert(merged, name)
                    seen[name] = true
                    if PriorityPetsEnabled[name] == nil then
                        PriorityPetsEnabled[name] = true
                    end
                end
            end
            for _, name in ipairs(loaded.PriorityPets) do
                if not seen[name] then
                    table.insert(merged, name)
                    seen[name] = true
                    if PriorityPetsEnabled[name] == nil then
                        PriorityPetsEnabled[name] = true
                    end
                end
            end
            PriorityPets = merged
        else
            for _, name in ipairs(PriorityPets) do
                if PriorityPetsEnabled[name] == nil then
                    PriorityPetsEnabled[name] = true
                end
            end
        end
    else
        for _, name in ipairs(PriorityPets) do
            if PriorityPetsEnabled[name] == nil then
                PriorityPetsEnabled[name] = true
            end
        end
    end
end

for _, name in ipairs(SecretList) do
    if SelectedPets[name] == nil then SelectedPets[name] = true end
    if LockedPets[name]   == nil then LockedPets[name]   = false end
end
for _, name in ipairs(OGList) do
    if SelectedPets[name] == nil then SelectedPets[name] = true end
    if LockedPets[name]   == nil then LockedPets[name]   = false end
end

----------------------------------------------------------------
-- UI helpers
----------------------------------------------------------------
local function AddCorner(obj, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 8)
    corner.Parent = obj
    return corner
end

local function AddStroke(obj, color, thickness)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or BORDER_CYAN
    stroke.Thickness = thickness or 2
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Parent = obj
    return stroke
end

local function SetBtnColor(btn, color)
    btn:SetAttribute("BaseColor", color)
    btn.BackgroundColor3 = color
end

local function AddHover(btn, base, hover)
    SetBtnColor(btn, base)
    btn:SetAttribute("HoverColor", hover)
    btn.MouseEnter:Connect(function()
        TweenService:Create(
            btn,
            TweenInfo.new(0.15),
            {BackgroundColor3 = btn:GetAttribute("HoverColor") or hover}
        ):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(
            btn,
            TweenInfo.new(0.2),
            {BackgroundColor3 = btn:GetAttribute("BaseColor") or base}
        ):Play()
    end)
end

local function Fade(frame, show)
    if show then
        frame.Visible = true
        frame.BackgroundTransparency = 1
        TweenService:Create(frame,TweenInfo.new(0.18),{BackgroundTransparency = 0}):Play()
    else
        TweenService:Create(frame,TweenInfo.new(0.18),{BackgroundTransparency = 1}):Play()
        task.delay(0.18,function()
            frame.Visible = false
        end)
    end
end

----------------------------------------------------------------
-- Root GUI + Main panel
----------------------------------------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CTFinderUI"
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui

-- Dock button nhỏ để mở lại
local DockBtn = Instance.new("TextButton")
DockBtn.Size = UDim2.new(0,70,0,28)
DockBtn.Position = UDim2.new(0,10,1,-40)
DockBtn.BackgroundColor3 = CYAN
DockBtn.Text = "GF"
DockBtn.TextColor3 = Color3.new(1,1,1)
DockBtn.Font = Enum.Font.GothamBold
DockBtn.TextSize = 16
DockBtn.Parent = ScreenGui
AddHover(DockBtn, CYAN, CYAN_BRIGHT)
AddCorner(DockBtn, 6)

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0,350,0,420)
Main.Position = UDim2.new(0.5,-175,0.4,0)
Main.BackgroundColor3 = PANEL_BG
Main.Active, Main.Draggable = true, true
Main.Parent = ScreenGui
AddCorner(Main, 12)
AddStroke(Main, BORDER_CYAN, 2)

local function updateDockVisibility()
    DockBtn.Visible = not Main.Visible
end

DockBtn.MouseButton1Click:Connect(function()
    Main.Visible = not Main.Visible
    updateDockVisibility()
end)

local Top = Instance.new("Frame")
Top.Size = UDim2.new(1,0,0,35)
Top.BackgroundColor3 = TITLE_BG
Top.Parent = Main
AddCorner(Top, 12)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,-140,1,0)
Title.Position = UDim2.new(0,10,0,0)
Title.BackgroundTransparency = 1
Title.Text = "🍇 Grape Finder"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = CYAN_BRIGHT
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Top

local BtnFilter = Instance.new("TextButton")
BtnFilter.Size = UDim2.new(0,70,0,28)
BtnFilter.Position = UDim2.new(1,-190,0,3)
SetBtnColor(BtnFilter,CYAN)
BtnFilter.Text = "Filter"
BtnFilter.TextColor3 = Color3.new(1,1,1)
BtnFilter.Font = Enum.Font.GothamBold
BtnFilter.TextSize = 14
BtnFilter.Parent = Top
AddHover(BtnFilter,CYAN,CYAN_BRIGHT)
AddCorner(BtnFilter, 6)

local BtnSetMoneyAll = Instance.new("TextButton")
BtnSetMoneyAll.Size = UDim2.new(0,100,0,28)
BtnSetMoneyAll.Position = UDim2.new(1,-115,0,3)
SetBtnColor(BtnSetMoneyAll,GOLD)
BtnSetMoneyAll.Text = "💰 Set All"
BtnSetMoneyAll.TextColor3 = Color3.new(0,0,0)
BtnSetMoneyAll.Font = Enum.Font.GothamBold
BtnSetMoneyAll.TextSize = 13
BtnSetMoneyAll.Parent = Top
AddHover(BtnSetMoneyAll,GOLD,Color3.fromRGB(255,220,80))
AddCorner(BtnSetMoneyAll, 6)

local BtnCloseMain = Instance.new("TextButton")
BtnCloseMain.Size = UDim2.new(0,30,0,28)
BtnCloseMain.Position = UDim2.new(1,-35,0,3)
SetBtnColor(BtnCloseMain,BUTTON_DARK)
BtnCloseMain.Text = "×"
BtnCloseMain.TextColor3 = Color3.new(1,1,1)
BtnCloseMain.Font = Enum.Font.GothamBold
BtnCloseMain.TextSize = 18
BtnCloseMain.Parent = Top
AddHover(BtnCloseMain,BUTTON_DARK,CYAN)
AddCorner(BtnCloseMain, 6)
BtnCloseMain.MouseButton1Click:Connect(function()
    Main.Visible = false
    updateDockVisibility()
end)

local PetList = Instance.new("ScrollingFrame")
PetList.Size = UDim2.new(1,-10,1,-70)
PetList.Position = UDim2.new(0,5,0,40)
PetList.BackgroundTransparency = 1
PetList.ScrollBarThickness = 6
PetList.ScrollBarImageColor3 = CYAN
PetList.CanvasSize = UDim2.new(0,0,0,0)
PetList.Parent = Main

local ListLayout = Instance.new("UIListLayout", PetList)
ListLayout.Padding = UDim.new(0,5)

local LastFound = Instance.new("TextLabel")
LastFound.Size = UDim2.new(1,-10,0,22)
LastFound.Position = UDim2.new(0,5,1,-27)
LastFound.BackgroundColor3 = BUTTON_DARK
LastFound.BackgroundTransparency = 0
LastFound.Text = "🐾 Last Found: —"
LastFound.TextColor3 = CYAN_BRIGHT
LastFound.Font = Enum.Font.Gotham
LastFound.TextSize = 13
LastFound.TextXAlignment = Enum.TextXAlignment.Left
LastFound.Parent = Main
AddCorner(LastFound, 6)

local function ClearResults()
    for _, child in ipairs(PetList:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    PetList.CanvasSize = UDim2.new(0,0,0,0)
    LastFound.Text = "🐾 Last Found: —"
end
-- Thêm sau LastFound label (khoảng dòng 290)
local SpamStatus = Instance.new("TextLabel")
SpamStatus.Size = UDim2.new(0, 120, 0, 22)
SpamStatus.Position = UDim2.new(1, -125, 1, -27)
SpamStatus.BackgroundColor3 = BUTTON_DARK
SpamStatus.BackgroundTransparency = 0
SpamStatus.Text = "Spam: ON"
SpamStatus.TextColor3 = GREEN
SpamStatus.Font = Enum.Font.GothamBold
SpamStatus.TextSize = 12
SpamStatus.TextXAlignment = Enum.TextXAlignment.Right
SpamStatus.Parent = Main
AddCorner(SpamStatus, 6)

-- Cập nhật hàm toggle spam
UserInputService.InputBegan:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.T then
        ScreenGui.Enabled = not ScreenGui.Enabled
    elseif input.KeyCode == Enum.KeyCode.Comma then
        if input.KeyCode == Enum.KeyCode.Comma and not UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            SpamEnabled = not SpamEnabled
            SpamStatus.Text = "Spam: " .. (SpamEnabled and "ON" or "OFF")
            SpamStatus.TextColor3 = SpamEnabled and GREEN or Color3.fromRGB(255, 100, 100)
            
            if not SpamEnabled then
                CurrentSpamJob = nil
                print("[SPAM] ❌ Disabled - press , again to enable")
            else
                print("[SPAM] ✅ Enabled")
            end
        end
    
    -- Shift + ,: Force enable spam
    elseif input.KeyCode == Enum.KeyCode.Comma and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        SpamEnabled = true
        SpamStatus.Text = "Spam: ON"
        SpamStatus.TextColor3 = GREEN
        print("[SPAM] ⚡ FORCE ENABLED via Shift + ,")
    end
end)
----------------------------------------------------------------
-- Filter panel
----------------------------------------------------------------
local Filter = Instance.new("Frame")
Filter.Size = UDim2.new(0,270,0,380)
Filter.Position = UDim2.new(0,50,0.35,0)
Filter.BackgroundColor3 = PANEL_BG
Filter.Active, Filter.Draggable = true, true
Filter.Visible = false
Filter.Parent = ScreenGui
AddCorner(Filter, 10)
AddStroke(Filter, BORDER_CYAN, 2)

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1,-10,0,36)
Header.Position = UDim2.new(0,5,0,5)
Header.BackgroundColor3 = TITLE_BG
Header.Parent = Filter
AddCorner(Header, 8)

local PillFilters = Instance.new("TextLabel")
PillFilters.Size = UDim2.new(0,80,0,28)
PillFilters.Position = UDim2.new(0,4,0,4)
PillFilters.BackgroundColor3 = CYAN
PillFilters.Text = "Filters"
PillFilters.TextColor3 = TEXT_WHITE
PillFilters.Font = Enum.Font.GothamBold
PillFilters.TextSize = 14
PillFilters.Parent = Header
AddCorner(PillFilters,8)

local BtnDefault = Instance.new("TextButton")
BtnDefault.Size = UDim2.new(0,80,0,28)
BtnDefault.Position = UDim2.new(0,92,0,4)
SetBtnColor(BtnDefault,BUTTON_DARK)
BtnDefault.Text = "Default"
BtnDefault.TextColor3 = Color3.new(1,1,1)
BtnDefault.Font = Enum.Font.GothamBold
BtnDefault.TextSize = 13
BtnDefault.Parent = Header
AddHover(BtnDefault,BUTTON_DARK,CYAN)
AddCorner(BtnDefault,8)

local FClose = Instance.new("TextButton")
FClose.Size = UDim2.new(0,28,0,28)
FClose.Position = UDim2.new(1,-32,0,4)
SetBtnColor(FClose,BUTTON_DARK)
FClose.Text = "X"
FClose.TextColor3 = Color3.new(1,1,1)
FClose.Font = Enum.Font.GothamBold
FClose.TextSize = 14
FClose.Parent = Header
AddHover(FClose,BUTTON_DARK,CYAN)
AddCorner(FClose,8)

FClose.MouseButton1Click:Connect(function()
    Fade(Filter,false)
end)
BtnFilter.MouseButton1Click:Connect(function()
    Fade(Filter,true)
end)

-- Min Money
local TBMin
do
    local LMin = Instance.new("TextLabel")
    LMin.Text = "Min Money (10m, 20m..)"
    LMin.TextColor3 = TEXT_WHITE
    LMin.Position = UDim2.new(0,10,0,50)
    LMin.Size = UDim2.new(0,240,0,25)
    LMin.Font = Enum.Font.GothamBold
    LMin.TextSize = 14
    LMin.BackgroundTransparency = 1
    LMin.Parent = Filter

    TBMin = Instance.new("TextBox")
    TBMin.Size = UDim2.new(0,240,0,28)
    TBMin.Position = UDim2.new(0,10,0,78)
    TBMin.PlaceholderText = "vd: 10 (=10m), 50m, 1b, 500k"
    TBMin.Text = tostring(MinMoneyM)
    TBMin.TextColor3 = TEXT_WHITE
    TBMin.BackgroundColor3 = BUTTON_DARK
    TBMin.Font = Enum.Font.Gotham
    TBMin.TextSize = 13
    TBMin.ClearTextOnFocus = false
    TBMin.Parent = Filter
    AddCorner(TBMin, 6)
end

local BtnRarity = Instance.new("TextButton")
BtnRarity.Size = UDim2.new(0,240,0,30)
BtnRarity.Position = UDim2.new(0,10,0,118)
BtnRarity.Text = "Rarity & Pets ▸"
BtnRarity.TextColor3 = Color3.new(1,1,1)
BtnRarity.Font = Enum.Font.GothamBold
BtnRarity.TextSize = 13
BtnRarity.Parent = Filter
SetBtnColor(BtnRarity, BUTTON_DARK)
AddHover(BtnRarity,BUTTON_DARK,CYAN)
AddCorner(BtnRarity, 6)

local BtnPriority = Instance.new("TextButton")
BtnPriority.Size = UDim2.new(0,240,0,30)
BtnPriority.Position = UDim2.new(0,10,0,158)
BtnPriority.Text = "⭐ Priority Pets ▸"
BtnPriority.TextColor3 = GOLD
BtnPriority.Font = Enum.Font.GothamBold
BtnPriority.TextSize = 13
BtnPriority.Parent = Filter
SetBtnColor(BtnPriority, BUTTON_DARK)
AddHover(BtnPriority,BUTTON_DARK,Color3.fromRGB(255, 220, 80))
AddCorner(BtnPriority, 6)

local BtnAutoJoin = Instance.new("TextButton")
BtnAutoJoin.Size = UDim2.new(0,240,0,30)
BtnAutoJoin.Position = UDim2.new(0,10,0,198)
BtnAutoJoin.Text = AutoJoinOn and "Auto Join: ON" or "Auto Join: OFF"
BtnAutoJoin.TextColor3 = Color3.new(1,1,1)
BtnAutoJoin.Font = Enum.Font.GothamBold
BtnAutoJoin.TextSize = 13
BtnAutoJoin.Parent = Filter
SetBtnColor(BtnAutoJoin, AutoJoinOn and CYAN or BUTTON_DARK)
AddHover(BtnAutoJoin, BtnAutoJoin:GetAttribute("BaseColor"), CYAN_BRIGHT)
AddCorner(BtnAutoJoin, 6)

local BtnAutoFinder = Instance.new("TextButton")
BtnAutoFinder.Size = UDim2.new(0,240,0,30)
BtnAutoFinder.Position = UDim2.new(0,10,0,238)
BtnAutoFinder.Text = AutoFinderOn and "Auto Finder: ON" or "Auto Finder: OFF"
BtnAutoFinder.TextColor3 = Color3.new(1,1,1)
BtnAutoFinder.Font = Enum.Font.GothamBold
BtnAutoFinder.TextSize = 13
BtnAutoFinder.Parent = Filter
SetBtnColor(BtnAutoFinder, AutoFinderOn and CYAN or BUTTON_DARK)
AddHover(BtnAutoFinder, BtnAutoFinder:GetAttribute("BaseColor"), CYAN_BRIGHT)
AddCorner(BtnAutoFinder, 6)

local LTier = Instance.new("TextLabel")
LTier.Text = "Notify tiers"
LTier.TextColor3 = TEXT_WHITE
LTier.Position = UDim2.new(0,10,0,278)
LTier.Size = UDim2.new(0,240,0,22)
LTier.Font = Enum.Font.GothamBold
LTier.TextSize = 14
LTier.BackgroundTransparency = 1
LTier.Parent = Filter

local BtnTierA = Instance.new("TextButton")
BtnTierA.Size = UDim2.new(0,240,0,26)
BtnTierA.Position = UDim2.new(0,10,0,304)
BtnTierA.Text = "1–10M"
BtnTierA.TextColor3 = Color3.new(1,1,1)
BtnTierA.Font = Enum.Font.GothamBold
BtnTierA.TextSize = 13
BtnTierA.Parent = Filter
SetBtnColor(BtnTierA, BUTTON_DARK)
AddHover(BtnTierA, BUTTON_DARK, CYAN)
AddCorner(BtnTierA, 6)

local BtnTierB = Instance.new("TextButton")
BtnTierB.Size = UDim2.new(0,240,0,26)
BtnTierB.Position = UDim2.new(0,10,0,334)
BtnTierB.Text = "10M+"
BtnTierB.TextColor3 = Color3.new(1,1,1)
BtnTierB.Font = Enum.Font.GothamBold
BtnTierB.TextSize = 13
BtnTierB.Parent = Filter
SetBtnColor(BtnTierB, BUTTON_DARK)
AddHover(BtnTierB, BUTTON_DARK, CYAN)
AddCorner(BtnTierB, 6)

----------------------------------------------------------------
-- HTTP wrapper + Discord helper
----------------------------------------------------------------
local http_impl = (syn and syn.request)
    or (http and http.request)
    or http_request
    or request

if not http_impl then
    warn("❌ CTFinder: Executor không hỗ trợ HTTP")
end

local function doRequest(opts)
    if not http_impl then return false,"no_http" end
    local ok, res = pcall(http_impl, opts)
    if not ok or not res then return false,"fail" end
    local code = res.StatusCode or res.Status or 0
    if code < 200 or code >= 300 then
        return false, "status_"..tostring(code)
    end
    return true, res
end

local function checkServerConnection()
    local url = string.format("%s/health", SERVER_TUNNEL_URL)
    

    
    local success, result = doRequest({
        Url = url,
        Method = "GET",
        Headers = {
            ["Accept"] = "application/json"
        },
        Timeout = 5
    })
    
    if success then
        local ok, data = pcall(function()
            return HttpService:JSONDecode(result.Body)
        end)
        
        if ok and data and data.ok then
            -- print("=" .. string.rep("=", 50))
            -- print("✅ SERVER CONNECTED")
            -- print("   URL:", url)
            -- print("   Status:", data.status or "OK")
            -- print(string.format("   Jobs: %d total", data.jobs and data.jobs.total or 0))
            -- print("=" .. string.rep("=", 50))
            return true
        else
            warn("⚠️ Invalid health response")
            return false
        end
    else
        warn("❌ Health check failed:", result, "| URL:", url)
        return false
    end
end

-- Kiểm tra kết nối khi khởi động
task.spawn(function()
    task.wait(3)  -- Đợi 3 giây để mọi thứ khởi động
    local connected = checkServerConnection()
    
    if not connected then
        -- Hiển thị cảnh báo trong UI
        task.spawn(function()
            local warning = Instance.new("TextLabel")
            warning.Size = UDim2.new(1, -20, 0, 40)
            warning.Position = UDim2.new(0, 10, 0, 45)
            warning.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            warning.Text = "⚠️ SERVER OFFLINE\nCheck connection and restart"
            warning.TextColor3 = Color3.new(1,1,1)
            warning.Font = Enum.Font.GothamBold
            warning.TextSize = 12
            warning.TextWrapped = true
            warning.Parent = Main
            AddCorner(warning, 6)
            
            task.wait(5)
            warning:Destroy()
        end)
    end
end)

local function pickChannelIDOf(tierKey)
    if tierKey=="A" then
        return Config.ChannelID_A
    else
        return Config.ChannelID_B
    end
end

local function fetchLatestMessage(tierKey)
    local auth = getAuthHeader()
    if not auth or not tierKey then return nil end
    local cid = pickChannelIDOf(tierKey)
    if not cid or cid=="" then return nil end
    cid = tostring(cid):gsub("%D","")
    if cid=="" then return nil end

    local HttpService = game:GetService("HttpService")

local function fetchJobs()
    local url = SERVER_TUNNEL_URL

    local response = HttpService:RequestAsync({
        Url = url,
        Method = "GET", -- 🚨 BẮT BUỘC GET
        Headers = {
            ["Accept"] = "application/json",
            ["User-Agent"] = "RobloxClient"
        }
    })

    if not response.Success then
        warn("❌ Lỗi gọi API:", response.StatusCode, response.StatusMessage)
        return nil
    end

    local ok, data = pcall(function()
        return HttpService:JSONDecode(response.Body)
    end)

    if not ok or type(data) ~= "table" then
        warn("❌ JSON lỗi")
        return nil
    end

    return data
end
    if not ok or not res or not res.Body then return nil end

    local okj,data = pcall(function() return HttpService:JSONDecode(res.Body) end)
    if not okj or type(data)~="table" or #data==0 then return nil end

    return data[1]
end

local function SnapshotLatest(tierKey)
    local msg = fetchLatestMessage(tierKey)
    if msg and msg.id then
        LastSeen[tierKey] = tostring(msg.id)
    end
end

----------------------------------------------------------------
-- Settings change handlers
----------------------------------------------------------------
TBMin.FocusLost:Connect(function()
    local raw = (TBMin.Text or ""):lower():gsub("%s+","")
    local num,unit = raw:match("([%d%.,]+)([kmb]?)")
    if num then
        local n = tonumber((num:gsub(",","")) or "0") or 0
        if unit=="" or unit=="m" then
            MinMoneyM = n
        elseif unit=="k" then
            MinMoneyM = n/1000
        elseif unit=="b" then
            MinMoneyM = n*1000
        end
    else
        MinMoneyM = 0
    end

    ClearResults()
    if CurrentTier and AutoFinderOn then
        SnapshotLatest(CurrentTier)
    end
    SaveSettings()
end)

----------------------------------------------------------------
-- Đổi tier
----------------------------------------------------------------
local function SetTier(which)
    CurrentTier = which
    ClearResults()

    if which=="A" then
        SetBtnColor(BtnTierA,CYAN)
        SetBtnColor(BtnTierB,BUTTON_DARK)
    elseif which=="B" then
        SetBtnColor(BtnTierA,BUTTON_DARK)
        SetBtnColor(BtnTierB,CYAN)
    else
        SetBtnColor(BtnTierA,BUTTON_DARK)
        SetBtnColor(BtnTierB,BUTTON_DARK)
    end

    if which and AutoFinderOn then
        SnapshotLatest(which)
    end
    SaveSettings()
end

BtnTierA.MouseButton1Click:Connect(function() SetTier("A") end)
BtnTierB.MouseButton1Click:Connect(function() SetTier("B") end)
SetTier(CurrentTier or "A")

----------------------------------------------------------------
-- Default reset
----------------------------------------------------------------
BtnDefault.MouseButton1Click:Connect(function()
    TBMin.Text = ""
    MinMoneyM  = 0

    AutoJoinOn = false
    BtnAutoJoin.Text = "Auto Join: OFF"
    SetBtnColor(BtnAutoJoin, BUTTON_DARK)

    AutoFinderOn = false
    BtnAutoFinder.Text = "Auto Finder: OFF"
    SetBtnColor(BtnAutoFinder, BUTTON_DARK)

    CurrentTier = nil
    SetBtnColor(BtnTierA,BUTTON_DARK)
    SetBtnColor(BtnTierB,BUTTON_DARK)
    ClearResults()

    SaveSettings()
end)

----------------------------------------------------------------
-- Toggle auto
----------------------------------------------------------------
BtnAutoJoin.MouseButton1Click:Connect(function()
    AutoJoinOn = not AutoJoinOn
    BtnAutoJoin.Text = AutoJoinOn and "Auto Join: ON" or "Auto Join: OFF"
    SetBtnColor(BtnAutoJoin, AutoJoinOn and CYAN or BUTTON_DARK)
    SaveSettings()
end)

BtnAutoFinder.MouseButton1Click:Connect(function()
    AutoFinderOn = not AutoFinderOn
    BtnAutoFinder.Text = AutoFinderOn and "Auto Finder: ON" or "Auto Finder: OFF"
    SetBtnColor(BtnAutoFinder, AutoFinderOn and CYAN or BUTTON_DARK)

    if AutoFinderOn then
        ClearResults()
        if CurrentTier then
            SnapshotLatest(CurrentTier)
        end
    end
    SaveSettings()
end)

----------------------------------------------------------------
-- PRIORITY PANEL
----------------------------------------------------------------
local Priority = Instance.new("Frame")
Priority.Size = UDim2.new(0,360,0,430)
Priority.Position = UDim2.new(1,-380,0.25,0)
Priority.BackgroundColor3 = PANEL_BG
Priority.Active, Priority.Draggable = true, true
Priority.Visible = false
Priority.Parent = ScreenGui
AddCorner(Priority, 10)
AddStroke(Priority, GOLD, 2)

BtnPriority.MouseButton1Click:Connect(function()
    Fade(Priority,true)
end)

local PClose = Instance.new("TextButton")
PClose.Size = UDim2.new(0,30,0,30)
PClose.Position = UDim2.new(1,-35,0,5)
SetBtnColor(PClose,BUTTON_DARK)
PClose.Text = "×"
PClose.TextColor3 = Color3.new(1,1,1)
PClose.Font = Enum.Font.GothamBold
PClose.TextSize = 18
PClose.Parent = Priority
AddHover(PClose,BUTTON_DARK,CYAN)
AddCorner(PClose, 6)
PClose.MouseButton1Click:Connect(function()
    Fade(Priority,false)
end)

local PT = Instance.new("TextLabel")
PT.Text = "⭐ Priority Pets"
PT.TextColor3 = GOLD
PT.Position = UDim2.new(0,10,0,5)
PT.Size = UDim2.new(0,250,0,30)
PT.BackgroundTransparency = 1
PT.Font = Enum.Font.GothamBold
PT.TextSize = 18
PT.ZIndex = 10
PT.Parent = Priority

local PSearch = Instance.new("TextBox", Priority)
PSearch.Size = UDim2.new(0,330,0,28)
PSearch.Position = UDim2.new(0,10,0,45)
PSearch.PlaceholderText = "Search để thêm pet vào Priority..."
PSearch.Text = ""
PSearch.BackgroundColor3 = BUTTON_DARK
PSearch.TextColor3 = TEXT_WHITE
PSearch.Font = Enum.Font.Gotham
PSearch.TextSize = 13
PSearch.ClearTextOnFocus = false
AddCorner(PSearch, 6)

local BtnEnableAll = Instance.new("TextButton", Priority)
BtnEnableAll.Size = UDim2.new(0,160,0,28)
BtnEnableAll.Position = UDim2.new(0,10,0,78)
SetBtnColor(BtnEnableAll,CYAN)
BtnEnableAll.Text = "Enable All"
BtnEnableAll.TextColor3 = Color3.new(1,1,1)
BtnEnableAll.Font = Enum.Font.GothamBold
BtnEnableAll.TextSize = 13
AddHover(BtnEnableAll,CYAN,CYAN_BRIGHT)
AddCorner(BtnEnableAll, 6)

local BtnClearPriority = Instance.new("TextButton", Priority)
BtnClearPriority.Size = UDim2.new(0,160,0,28)
BtnClearPriority.Position = UDim2.new(0,180,0,78)
SetBtnColor(BtnClearPriority,BUTTON_DARK)
BtnClearPriority.Text = "Clear All"
BtnClearPriority.TextColor3 = Color3.new(1,1,1)
BtnClearPriority.Font = Enum.Font.GothamBold
BtnClearPriority.TextSize = 13
AddHover(BtnClearPriority,BUTTON_DARK,CYAN)
AddCorner(BtnClearPriority, 6)

local PScroll = Instance.new("ScrollingFrame", Priority)
PScroll.Size = UDim2.new(1,-14,1,-120)
PScroll.Position = UDim2.new(0,7,0,110)
PScroll.BackgroundTransparency = 1
PScroll.ScrollBarThickness = 6
PScroll.ScrollBarImageColor3 = GOLD

local PGrid = Instance.new("UIGridLayout", PScroll)
PGrid.CellPadding = UDim2.new(0,8,0,8)
PGrid.CellSize = UDim2.new(0.5,-12,0,32)
PGrid.FillDirection = Enum.FillDirection.Horizontal
PGrid.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function BuildPriorityButton(name)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0,9999,0,32)
    b.TextColor3 = TEXT_WHITE
    b.Font = Enum.Font.GothamBold
    b.TextSize = 13
    b.AutoButtonColor = false
    AddCorner(b,8)
    
    local msBtn = Instance.new("TextButton")
    msBtn.Size = UDim2.new(0, 28, 0, 28)
    msBtn.Position = UDim2.new(1, -32, 0, 2)
    msBtn.BackgroundColor3 = BUTTON_DARK
    msBtn.Text = "⚙️"
    msBtn.TextSize = 12
    msBtn.Font = Enum.Font.GothamBold
    msBtn.TextColor3 = CYAN_BRIGHT
    msBtn.Parent = b
    msBtn.ZIndex = 11
    AddCorner(msBtn, 6)
    
    msBtn.MouseButton1Click:Connect(function()
        local msPopup = Instance.new("Frame")
        msPopup.Size = UDim2.new(0, 250, 0, 120)
        msPopup.Position = UDim2.new(0.5, -125, 0.5, -60)
        msPopup.BackgroundColor3 = PANEL_BG
        msPopup.BorderSizePixel = 0
        msPopup.Parent = ScreenGui
        msPopup.ZIndex = 100
        AddCorner(msPopup, 10)
        AddStroke(msPopup, CYAN, 2)
        
        local msTitle = Instance.new("TextLabel")
        msTitle.Size = UDim2.new(1, -40, 0, 30)
        msTitle.Position = UDim2.new(0, 10, 0, 5)
        msTitle.BackgroundTransparency = 1
        msTitle.Text = "⚙️ M/S for " .. name
        msTitle.Font = Enum.Font.GothamBold
        msTitle.TextSize = 14
        msTitle.TextColor3 = CYAN_BRIGHT
        msTitle.TextXAlignment = Enum.TextXAlignment.Left
        msTitle.Parent = msPopup
        
        local msInput = Instance.new("TextBox")
        msInput.Size = UDim2.new(1, -20, 0, 35)
        msInput.Position = UDim2.new(0, 10, 0, 40)
        msInput.BackgroundColor3 = BUTTON_DARK
        msInput.PlaceholderText = "Enter m/s (e.g. 16, 32, 50)"
        msInput.Text = tostring(PetMovementSpeeds[name] or 16)
        msInput.TextColor3 = TEXT_WHITE
        msInput.Font = Enum.Font.Gotham
        msInput.TextSize = 13
        msInput.ClearTextOnFocus = false
        msInput.Parent = msPopup
        AddCorner(msInput, 6)
        
        local msSave = Instance.new("TextButton")
        msSave.Size = UDim2.new(0, 100, 0, 30)
        msSave.Position = UDim2.new(0, 10, 1, -35)
        msSave.BackgroundColor3 = CYAN
        msSave.Text = "Save"
        msSave.Font = Enum.Font.GothamBold
        msSave.TextSize = 13
        msSave.TextColor3 = Color3.new(1, 1, 1)
        msSave.Parent = msPopup
        AddCorner(msSave, 6)
        
        local msCancel = Instance.new("TextButton")
        msCancel.Size = UDim2.new(0, 100, 0, 30)
        msCancel.Position = UDim2.new(1, -110, 1, -35)
        msCancel.BackgroundColor3 = BUTTON_DARK
        msCancel.Text = "Cancel"
        msCancel.Font = Enum.Font.GothamBold
        msCancel.TextSize = 13
        msCancel.TextColor3 = Color3.new(1, 1, 1)
        msCancel.Parent = msPopup
        AddCorner(msCancel, 6)
        
        msSave.MouseButton1Click:Connect(function()
            local ms = tonumber(msInput.Text)
            if ms and ms > 0 and ms <= 500 then
                PetMovementSpeeds[name] = ms
                SaveSettings()
                msPopup:Destroy()
            else
                msInput.Text = "Invalid! (1-500)"
                task.wait(1)
                msInput.Text = tostring(PetMovementSpeeds[name] or 16)
            end
        end)
        
        msCancel.MouseButton1Click:Connect(function()
            msPopup:Destroy()
        end)
    end)

    local function refreshVisual()
        local isEnabled = PriorityPetsEnabled[name]
        if isEnabled then
            SetBtnColor(b, GOLD)
            b.Text = "⭐ " .. name
        else
            SetBtnColor(b, BUTTON_DARK)
            b.Text = "☆ " .. name
        end
        AddHover(b, b:GetAttribute("BaseColor"), CYAN_BRIGHT)
    end

    refreshVisual()

    b.MouseButton1Click:Connect(function()
        PriorityPetsEnabled[name] = not PriorityPetsEnabled[name]
        refreshVisual()
        SaveSettings()
    end)

    b.MouseButton2Click:Connect(function()
        for i, pname in ipairs(PriorityPets) do
            if pname == name then
                table.remove(PriorityPets, i)
                PriorityPetsEnabled[name] = nil
                break
            end
        end
        b:Destroy()
        SaveSettings()
        task.wait()
        PScroll.CanvasSize = UDim2.new(0,0,0,PGrid.AbsoluteContentSize.Y+12)
    end)

    return b
end

local function RefreshPriorityList()
    for _,c in ipairs(PScroll:GetChildren()) do
        if c:IsA("TextButton") then
            c:Destroy()
        end
    end
    local q = string.lower(PSearch.Text or "")
    for _,name in ipairs(PriorityPets) do
        if q=="" or string.find(string.lower(name), q, 1, true) then
            BuildPriorityButton(name).Parent = PScroll
        end
    end
    task.wait()
    PScroll.CanvasSize = UDim2.new(0,0,0,PGrid.AbsoluteContentSize.Y+12)
end

PSearch:GetPropertyChangedSignal("Text"):Connect(RefreshPriorityList)

BtnEnableAll.MouseButton1Click:Connect(function()
    for _, name in ipairs(PriorityPets) do
        PriorityPetsEnabled[name] = true
    end
    SaveSettings()
    RefreshPriorityList()
end)

BtnClearPriority.MouseButton1Click:Connect(function()
    PriorityPets = {}
    PriorityPetsEnabled = {}
    SaveSettings()
    RefreshPriorityList()
end)

PSearch.FocusLost:Connect(function(enterPressed)
    if not enterPressed then return end
    local petName = PSearch.Text:gsub("^%s*(.-)%s*$", "%1")
    if petName == "" then return end
    
    local found = false
    for _, name in ipairs(SecretList) do
        if string.lower(name) == string.lower(petName) then
            petName = name
            found = true
            break
        end
    end
    if not found then
        for _, name in ipairs(OGList) do
            if string.lower(name) == string.lower(petName) then
                petName = name
                found = true
                break
            end
        end
    end
    
    if not found then
        return
    end
    
    for _, name in ipairs(PriorityPets) do
        if name == petName then
            PSearch.Text = ""
            return
        end
    end
    
    table.insert(PriorityPets, petName)
    PriorityPetsEnabled[petName] = true
    SaveSettings()
    PSearch.Text = ""
    RefreshPriorityList()
end)

RefreshPriorityList()

----------------------------------------------------------------
--Rarity panel
----------------------------------------------------------------
local Rarity = Instance.new("Frame")
Rarity.Size = UDim2.new(0,360,0,430)
Rarity.Position = UDim2.new(1,-380,0.4,0)
Rarity.BackgroundColor3 = PANEL_BG
Rarity.Active, Rarity.Draggable = true, true
Rarity.Visible = false
Rarity.Parent = ScreenGui
AddCorner(Rarity, 10)
AddStroke(Rarity, BORDER_CYAN, 2)

BtnRarity.MouseButton1Click:Connect(function()
    Fade(Rarity,true)
end)

local RClose = Instance.new("TextButton")
RClose.Size = UDim2.new(0,30,0,30)
RClose.Position = UDim2.new(1,-35,0,5)
SetBtnColor(RClose,BUTTON_DARK)
RClose.Text = "×"
RClose.TextColor3 = Color3.new(1,1,1)
RClose.Font = Enum.Font.GothamBold
RClose.TextSize = 18
RClose.Parent = Rarity
AddHover(RClose,BUTTON_DARK,CYAN)
AddCorner(RClose, 6)
RClose.MouseButton1Click:Connect(function()
    Fade(Rarity,false)
end)

local RT = Instance.new("TextLabel")
RT.Text = "Rarity & Pets"
RT.TextColor3 = CYAN_BRIGHT
RT.Position = UDim2.new(0,10,0,5)
RT.Size = UDim2.new(0,200,0,30)
RT.BackgroundTransparency = 1
RT.Font = Enum.Font.GothamBold
RT.TextSize = 18
RT.ZIndex = 10
RT.Parent = Rarity

local TabSecret = Instance.new("TextButton", Rarity)
TabSecret.Size = UDim2.new(0,110,0,30)
TabSecret.Position = UDim2.new(0,10,0,45)
TabSecret.Text = "Secret"
TabSecret.TextColor3 = Color3.new(1,1,1)
TabSecret.Font = Enum.Font.GothamBold
TabSecret.TextSize = 13
SetBtnColor(TabSecret, CYAN)
AddHover(TabSecret,CYAN,CYAN_BRIGHT)
AddCorner(TabSecret, 6)

local TabOG = Instance.new("TextButton", Rarity)
TabOG.Size = UDim2.new(0,110,0,30)
TabOG.Position = UDim2.new(0,130,0,45)
TabOG.Text = "OG"
TabOG.TextColor3 = Color3.new(1,1,1)
TabOG.Font = Enum.Font.GothamBold
TabOG.TextSize = 13
SetBtnColor(TabOG, BUTTON_DARK)
AddHover(TabOG,BUTTON_DARK,CYAN)
AddCorner(TabOG, 6)

local Search = Instance.new("TextBox", Rarity)
Search.Size = UDim2.new(0,330,0,28)
Search.Position = UDim2.new(0,10,0,85)
Search.PlaceholderText = "Search name..."
Search.Text = ""
Search.BackgroundColor3 = BUTTON_DARK
Search.TextColor3 = TEXT_WHITE
Search.Font = Enum.Font.Gotham
Search.TextSize = 13
Search.ClearTextOnFocus = false
AddCorner(Search, 6)

local BtnSelectAll = Instance.new("TextButton", Rarity)
BtnSelectAll.Size = UDim2.new(0,160,0,28)
BtnSelectAll.Position = UDim2.new(0,10,0,118)
SetBtnColor(BtnSelectAll,CYAN)
BtnSelectAll.Text = "Select All"
BtnSelectAll.TextColor3 = Color3.new(1,1,1)
BtnSelectAll.Font = Enum.Font.GothamBold
BtnSelectAll.TextSize = 13
AddHover(BtnSelectAll,CYAN,CYAN_BRIGHT)
AddCorner(BtnSelectAll, 6)

local BtnClear = Instance.new("TextButton", Rarity)
BtnClear.Size = UDim2.new(0,160,0,28)
BtnClear.Position = UDim2.new(0,180,0,118)
SetBtnColor(BtnClear,BUTTON_DARK)
BtnClear.Text = "Clear"
BtnClear.TextColor3 = Color3.new(1,1,1)
BtnClear.Font = Enum.Font.GothamBold
BtnClear.TextSize = 13
AddHover(BtnClear,BUTTON_DARK,CYAN)
AddCorner(BtnClear, 6)

local Scroll = Instance.new("ScrollingFrame", Rarity)
Scroll.Size = UDim2.new(1,-14,1,-160)
Scroll.Position = UDim2.new(0,7,0,150)
Scroll.BackgroundTransparency = 1
Scroll.ScrollBarThickness = 6
Scroll.ScrollBarImageColor3 = CYAN

local Grid = Instance.new("UIGridLayout", Scroll)
Grid.CellPadding = UDim2.new(0,8,0,8)
Grid.CellSize = UDim2.new(0.5,-12,0,32)
Grid.FillDirection = Enum.FillDirection.Horizontal
Grid.HorizontalAlignment = Enum.HorizontalAlignment.Center

local currentTab = "Secret"

local function BuildPetButton(name)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0,9999,0,32)
    b.TextColor3 = TEXT_WHITE
    b.Font = Enum.Font.GothamBold
    b.TextSize = 13
    b.AutoButtonColor = false
    AddCorner(b,8)

    local moneyDisplay = Instance.new("TextButton")
    moneyDisplay.Size = UDim2.new(0, 45, 0, 28)
    moneyDisplay.Position = UDim2.new(1, -49, 0, 2)
    moneyDisplay.BackgroundColor3 = BUTTON_DARK
    moneyDisplay.Text = tostring(PetMoneyThresholds[name] or MinMoneyM or 10)
    moneyDisplay.TextSize = 12
    moneyDisplay.Font = Enum.Font.GothamBold
    moneyDisplay.TextColor3 = GOLD
    moneyDisplay.Parent = b
    moneyDisplay.ZIndex = 11
    AddCorner(moneyDisplay, 6)
    AddHover(moneyDisplay, BUTTON_DARK, GOLD)
    
    local function updateMoneyDisplay()
        moneyDisplay.Text = tostring(PetMoneyThresholds[name] or MinMoneyM or 10)
    end
    
    moneyDisplay.MouseButton1Click:Connect(function()
        local popup = Instance.new("Frame")
        popup.Size = UDim2.new(0, 280, 0, 140)
        popup.Position = UDim2.new(0.5, -140, 0.5, -70)
        popup.BackgroundColor3 = PANEL_BG
        popup.BorderSizePixel = 0
        popup.Parent = ScreenGui
        popup.ZIndex = 100
        AddCorner(popup, 10)
        AddStroke(popup, GOLD, 2)
        
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -40, 0, 30)
        title.Position = UDim2.new(0, 10, 0, 5)
        title.BackgroundTransparency = 1
        title.Text = "💰 Min Money for " .. name
        title.Font = Enum.Font.GothamBold
        title.TextSize = 14
        title.TextColor3 = GOLD
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = popup
        
        local input = Instance.new("TextBox")
        input.Size = UDim2.new(1, -20, 0, 35)
        input.Position = UDim2.new(0, 10, 0, 40)
        input.BackgroundColor3 = BUTTON_DARK
        input.PlaceholderText = "Enter min money (e.g. 10, 20, 50)"
        input.Text = tostring(PetMoneyThresholds[name] or MinMoneyM or 0)
        input.TextColor3 = TEXT_WHITE
        input.Font = Enum.Font.Gotham
        input.TextSize = 13
        input.ClearTextOnFocus = false
        input.Parent = popup
        AddCorner(input, 6)
        
        local saveBtn = Instance.new("TextButton")
        saveBtn.Size = UDim2.new(0, 120, 0, 30)
        saveBtn.Position = UDim2.new(0, 10, 1, -35)
        saveBtn.BackgroundColor3 = CYAN
        saveBtn.Text = "Save"
        saveBtn.Font = Enum.Font.GothamBold
        saveBtn.TextSize = 13
        saveBtn.TextColor3 = Color3.new(1, 1, 1)
        saveBtn.Parent = popup
        AddCorner(saveBtn, 6)
        AddHover(saveBtn, CYAN, CYAN_BRIGHT)
        
        local clearBtn = Instance.new("TextButton")
        clearBtn.Size = UDim2.new(0, 70, 0, 30)
        clearBtn.Position = UDim2.new(0, 140, 1, -35)
        clearBtn.BackgroundColor3 = BUTTON_DARK
        clearBtn.Text = "Clear"
        clearBtn.Font = Enum.Font.GothamBold
        clearBtn.TextSize = 13
        clearBtn.TextColor3 = Color3.new(1, 1, 1)
        clearBtn.Parent = popup
        AddCorner(clearBtn, 6)
        AddHover(clearBtn, BUTTON_DARK, CYAN)
        
        local cancelBtn = Instance.new("TextButton")
        cancelBtn.Size = UDim2.new(0, 60, 0, 30)
        cancelBtn.Position = UDim2.new(1, -70, 1, -35)
        cancelBtn.BackgroundColor3 = BUTTON_DARK
        cancelBtn.Text = "×"
        cancelBtn.Font = Enum.Font.GothamBold
        cancelBtn.TextSize = 18
        cancelBtn.TextColor3 = Color3.new(1, 1, 1)
        cancelBtn.Parent = popup
        AddCorner(cancelBtn, 6)
        AddHover(cancelBtn, BUTTON_DARK, CYAN)
        
        saveBtn.MouseButton1Click:Connect(function()
            local money = tonumber(input.Text)
            if money and money >= 0 and money <= 1000 then
                PetMoneyThresholds[name] = money
                SaveSettings()
                updateMoneyDisplay()
                RefreshRarityList()
                popup:Destroy()
            else
                input.Text = "Invalid! (0-1000)"
                task.wait(1)
                input.Text = tostring(PetMoneyThresholds[name] or MinMoneyM or 0)
            end
        end)
        
        clearBtn.MouseButton1Click:Connect(function()
            PetMoneyThresholds[name] = nil
            SaveSettings()
            updateMoneyDisplay()
            RefreshRarityList()
            popup:Destroy()
        end)
        
        cancelBtn.MouseButton1Click:Connect(function()
            popup:Destroy()
        end)
    end)

    local function refreshVisual()
        local baseColor
        if SelectedPets[name] then
            baseColor = CYAN
        else
            baseColor = BUTTON_DARK
        end
        SetBtnColor(b, baseColor)
        b.Text = name
        b:SetAttribute("BaseColor", baseColor)
    end

    refreshVisual()
    AddHover(b, b:GetAttribute("BaseColor"), CYAN_BRIGHT)

    b.MouseButton1Click:Connect(function()
        SelectedPets[name] = not SelectedPets[name]
        refreshVisual()
        SaveSettings()
    end)

    b.MouseButton2Click:Connect(function()
        LockedPets[name] = not LockedPets[name]
        if LockedPets[name] and not SelectedPets[name] then
            SelectedPets[name] = true
        end
        refreshVisual()
        SaveSettings()
    end)

    return b
end

local function RefreshRarityList()
    for _,c in ipairs(Scroll:GetChildren()) do
        if c:IsA("TextButton") then
            c:Destroy()
        end
    end
    local data = (currentTab=="Secret") and SecretList or OGList
    local q = string.lower(Search.Text or "")
    for _,name in ipairs(data) do
        if q=="" or string.find(string.lower(name), q, 1, true) then
            BuildPetButton(name).Parent = Scroll
        end
    end
    task.wait()
    Scroll.CanvasSize = UDim2.new(0,0,0,Grid.AbsoluteContentSize.Y+12)
end

TabSecret.MouseButton1Click:Connect(function()
    currentTab = "Secret"
    SetBtnColor(TabSecret,CYAN)
    SetBtnColor(TabOG,BUTTON_DARK)
    RefreshRarityList()
end)

TabOG.MouseButton1Click:Connect(function()
    currentTab = "OG"
    SetBtnColor(TabSecret,BUTTON_DARK)
    SetBtnColor(TabOG,CYAN)
    RefreshRarityList()
end)

Search:GetPropertyChangedSignal("Text"):Connect(RefreshRarityList)

BtnSelectAll.MouseButton1Click:Connect(function()
    local data = (currentTab=="Secret") and SecretList or OGList
    for _,name in ipairs(data) do
        SelectedPets[name] = true
    end
    SaveSettings()
    RefreshRarityList()
end)

BtnClear.MouseButton1Click:Connect(function()
    for _,n in ipairs((currentTab=="Secret") and SecretList or OGList) do
        if not LockedPets[n] then
            SelectedPets[n] = false
        else
            SelectedPets[n] = true
        end
    end
    SaveSettings()
    RefreshRarityList()
end)

RefreshRarityList()



----------------------------------------------------------------
-- Discord parse
----------------------------------------------------------------
local uuidPattern = "%x%x%x%x%x%x%x%x%-%x%x%x%x%-%x%x%x%x%-%x%x%x%x%-%x%x%x%x%x%x%x%x%x%x%x%x"

local function parseMoneyToM(s)
    if not s then return 0 end
    s = tostring(s):lower():gsub("%s+","")
    local num,unit = s:match("([%d%.,]+)([kmb]?)")
    if not num then return 0 end
    num = tonumber((num:gsub(",","")) or "0") or 0
    if unit=="" or unit=="m" then
        return num
    elseif unit=="k" then
        return num/1000
    elseif unit=="b" then
        return num*1000
    end
    return num
end

local function tierMatch(m)
    if not CurrentTier then return true end
    if CurrentTier=="A" then
        return m>=1 and m<=10
    else
        return m>=10
    end
end

local function foldContent(msg)
    local s = tostring(msg.content or "")
    if type(msg.embeds)=="table" then
        for _,e in ipairs(msg.embeds) do
            if e.title then s = s .. "\n" .. e.title end
            if e.description then s = s .. "\n" .. e.description end
            if e.fields then
                for _,f in ipairs(e.fields) do
                    if f.value then s = s .. "\n" .. f.value end
                end
            end
        end
    end
    return s
end

local function parseSingleStyle(msg)
    local text = foldContent(msg)

    local jobid = text:match(uuidPattern)
        or text:match("Job%s*ID%s*%([^)]*%)%s*[:：]%s*([%w%-]+)")
        or text:match("JobId:%s*([%w%-]+)")

    local pet = text:match("T[%ií]m%S*%s*t[hH]ấy:%s*%*%*(.-)%*%*")
        or text:match("Found:%s*%*%*(.-)%*%*")
        or text:match("🌈%s*(.-)%s+%$")
        or text:match("💎%s*(.-)%s+%$")
        or text:match("Name%s*[:\n]%s*(.-)%s*%$")
        or text:match("Name%s*[:\n]%s*(.-)%s*Money")
        or text:match("💰%s*Money/s%s*(.-)%s+%$")

    if pet then
        pet = pet:gsub("🌈", "")
        pet = pet:gsub("💎", "")
        pet = pet:gsub("💰", "")
        pet = pet:gsub("⭐", "")
        pet = pet:gsub("%*%*", "")
        pet = pet:gsub("^%s+", ""):gsub("%s+$", "")
        pet = pet:gsub("%.%.%.$", "")
    end

    local money = text:match("[Tt]iền:%s*([%d%.,]+%s*[kKmMbB]?)")
        or text:match("[Mm]oney:%s*([%d%.,]+%s*[kKmMbB]?)")
        or text:match("%$([%d%.,]+%s*[kKmMbB]?)")

    local pNow,pMax = text:match("Người chơi:%s*(%d+)%s*/%s*(%d+)")
    if not pNow then
        pNow,pMax = text:match("[Pp]layers:%s*(%d+)%s*/%s*(%d+)")
    end

    local moneyM = money and parseMoneyToM(money) or 0
    local playersStr = (pNow and pMax) and (pNow.."/"..pMax) or "-"

    return pet or "?", jobid, moneyM, playersStr
end

local function parseTableStyle(msg)
    local text = foldContent(msg)
    local entries = {}

    local jobid = text:match(uuidPattern)
        or text:match("Job%s*ID%s*%([^)]*%)%s*[:：]%s*([%w%-]+)")
        or text:match("JobId:%s*([%w%-]+)")
    if not jobid then return entries end

    local pNow,pMax = text:match("[Pp]layers:%s*(%d+)%s*/%s*(%d+)")
    local playersStr = (pNow and pMax) and (pNow.."/"..pMax) or "-"

    local function tryLine(line)
        if line:find("Players") or line:find("Name") or line:find("Money") or line:find("💰") then 
            return 
        end
        if line:match("^%s*$") then return end

        local cleanLine = line
        cleanLine = cleanLine:gsub("🌈", "")
        cleanLine = cleanLine:gsub("💎", "")
        cleanLine = cleanLine:gsub("💰", "")
        cleanLine = cleanLine:gsub("⭐", "")
        cleanLine = cleanLine:gsub("👥", "")
        cleanLine = cleanLine:gsub("%*%*", "")
        cleanLine = cleanLine:gsub("^%s+", ""):gsub("%s+$", "")
        
        local name, gen = cleanLine:match("^(.-)%s+%$([%d%.,]+%s*[kKmMbB]?)%s*/?s?%s*$")
        if not name then
            name, gen = cleanLine:match("^(.-)%s+([%d%.,]+%s*[kKmMbB]?)%s*/?s?%s*$")
        end
        
        if name and gen then
            name = name:gsub("^%s+", ""):gsub("%s+$", "")
            name = name:gsub("%.%.%.$", "")
            
            if name ~= "" and #name > 2 then
                local moneyM = parseMoneyToM(gen)
                table.insert(entries, {
                    pet     = name,
                    jobid   = jobid,
                    moneyM  = moneyM,
                    players = playersStr
                })
            end
        end
    end

    local block = text:match("```(.-)```")
    if block then
        for line in block:gmatch("[^\r\n]+") do
            tryLine(line)
        end
    else
        for line in text:gmatch("[^\r\n]+") do
            tryLine(line)
        end
    end

    return entries
end

local function parseMessageEntries(msg)
    local list = parseTableStyle(msg)
    if #list == 0 then
        local pet, jobid, moneyM, playersStr = parseSingleStyle(msg)
        if jobid then
            table.insert(list, {
                pet     = pet,
                jobid   = jobid,
                moneyM  = moneyM,
                players = playersStr
            })
        end
    end
    return list
end

----------------------------------------------------------------
-- Check pet có nằm trong list đã tick
----------------------------------------------------------------
local function isPetSelected(petName)
    if not petName then return false end
    
    local normalizedInput = string.lower(tostring(petName))
    normalizedInput = normalizedInput:gsub("%s+", " "):gsub("^%s*(.-)%s*$", "%1")
    
    -- Kiểm tra trong SelectedPets
    for name, isSelected in pairs(SelectedPets) do
        if isSelected then
            local normalizedName = string.lower(tostring(name):gsub("%s+", " "))
            
            -- So khớp chính xác hoặc gần đúng
            if normalizedInput == normalizedName then
                return true
            end
            
            -- Kiểm tra các biến thể tên
            if string.find(normalizedInput, normalizedName, 1, true) then
                return true
            end
            
            if string.find(normalizedName, normalizedInput, 1, true) then
                return true
            end
        end
    end
    
    return false
end

----------------------------------------------------------------
-- Check xem pet có nằm trong Priority list không
----------------------------------------------------------------
local function isPetPriority(petName)
    if not petName then return false end
    
    local normalizedInput = string.lower(tostring(petName))
    normalizedInput = normalizedInput:gsub("%s+", " "):gsub("^%s*(.-)%s*$", "%1")
    
    for _, priorityName in ipairs(PriorityPets) do
        if PriorityPetsEnabled[priorityName] then
            local normalizedPriority = string.lower(tostring(priorityName):gsub("%s+", " "))
            
            if normalizedInput == normalizedPriority then
                return true
            end
            
            if string.find(normalizedInput, normalizedPriority, 1, true) then
                return true
            end
            
            if string.find(normalizedPriority, normalizedInput, 1, true) then
                return true
            end
        end
    end
    
    return false
end

----------------------------------------------------------------
-- Spam Join
----------------------------------------------------------------
----------------------------------------------------------------
-- Spam Join (AUTO không TP lại job đã auto-join trước đó)
----------------------------------------------------------------
local function joinSpam(jobId, placeId, isAuto)
    if not jobId or jobId == "" then return end
    
    -- ⚡ KIỂM TRA SPAM ENABLED TRƯỚC KHI JOIN
    if not SpamEnabled then
        print(string.format("[AUTOJOIN] ❌ Spam disabled (comma pressed), skipping job %s", jobId:sub(1, 8)))
        return
    end

    -- ❌ AutoJoin KHÔNG TP lại job đã auto-join trước đó
    if isAuto and AutoUsedJobs and AutoUsedJobs[jobId] then
        print(string.format("[AUTOJOIN] ❌ Job %s already auto-joined before", jobId:sub(1, 8)))
        return
    end

    -- ⚡ STOP JOB HIỆN TẠI NẾU CÓ
    if CurrentSpamJob and CurrentSpamJob ~= jobId then
        print(string.format("[AUTOJOIN] ⏹️ Stopping previous job %s for new job %s", 
            CurrentSpamJob:sub(1, 8), jobId:sub(1, 8)))
        CurrentSpamJob = nil
    end

    CurrentSpamJob = jobId

    -- ✅ AutoJoin: đánh dấu + lưu
    if isAuto then
        AutoUsedJobs[jobId] = true
        _G.__CTF_autoUsedJobs = AutoUsedJobs
        print(string.format("[AUTOJOIN] ✅ Marked job %s as auto-used", jobId:sub(1, 8)))
        pcall(SaveSettings)
    end

    print(string.format("[AUTOJOIN] ⚡ Starting teleport to job %s...", jobId:sub(1, 8)))
    
    task.spawn(function()
        local attempts = 0
        local maxAttempts = 5
        
        while SpamEnabled and CurrentSpamJob == jobId and attempts < maxAttempts do
            attempts = attempts + 1
            pcall(function()
                local pid = tonumber(placeId) or game.PlaceId
                print(string.format("[AUTOJOIN] Attempt %d: Teleporting to %s...", attempts, jobId:sub(1, 8)))
                TeleportService:TeleportToPlaceInstance(pid, jobId, LocalPlayer)
            end)
            task.wait(0.0001)  -- Tăng delay giữa các lần teleport
        end
        
        if attempts >= maxAttempts then
            print("[AUTOJOIN] ❌ Max teleport attempts reached")
        elseif not SpamEnabled then
            print("[AUTOJOIN] ⏸️ Spam disabled (comma pressed)")
        elseif CurrentSpamJob ~= jobId then
            print(string.format("[AUTOJOIN] ↪️ Switched to new job"))
        end
    end)
end
----------------------------------------------------------------
-- UI row + xử lý kết quả
----------------------------------------------------------------
local function addRow(petName, moneyStr, playersStr, jobid, order, isPriority, placeId)
    local item = Instance.new("Frame")
    item.Size = UDim2.new(1,-10,0,30)
    item.BackgroundColor3 = isPriority and Color3.fromRGB(30,25,15) or BUTTON_DARK
    item.LayoutOrder = order or 0
    item.Parent = PetList
    AddCorner(item,6)

    local txt = Instance.new("TextLabel",item)
    txt.BackgroundTransparency = 1
    txt.Size = UDim2.new(0.7,-10,1,0)
    txt.Position = UDim2.new(0,8,0,0)
    txt.Font = Enum.Font.GothamBold
    txt.TextSize = 13
    txt.TextXAlignment = Enum.TextXAlignment.Left
    txt.TextColor3 = isPriority and PRIORITY_COLOR or TEXT_WHITE
    txt.Text = (isPriority and "⭐ " or "") .. string.format(
        "%s — %.2fM — %s",
        tostring(petName or "Unknown"),
        tonumber(moneyStr) or 0,
        tostring(playersStr or "-")
    )

    local join = Instance.new("TextButton",item)
    join.Size = UDim2.new(0.3,-8,1,-6)
    join.Position = UDim2.new(0.7,4,0,3)
    SetBtnColor(join, isPriority and PRIORITY_COLOR or CYAN)
    join.Text = "Join"
    join.TextColor3 = Color3.new(1,1,1)
    join.Font = Enum.Font.GothamBold
    join.TextSize = 13
    AddHover(join, isPriority and PRIORITY_COLOR or CYAN, CYAN_BRIGHT)
    AddCorner(join, 5)

    -- ✅ JOIN THỦ CÔNG: KHÔNG đánh dấu AutoUsedJobs
    join.MouseButton1Click:Connect(function()
        if jobid and jobid ~= "" then
            joinSpam(jobid, placeId or game.PlaceId, false) -- manual
        end
    end)

    PetList.CanvasSize = UDim2.new(0,0,0,ListLayout.AbsoluteContentSize.Y+10)
end

----------------------------------------------------------------
-- Xử lý kết quả, ưu tiên Pet Priority
----------------------------------------------------------------
local allMatchedResults = {}

local function handleParsed(item)
    if not item or not item.jobid then 
        print("[ERROR] Item không có jobid:", item)
        return 
    end
    
    print("[FILTER] === Checking item ===")
    print(string.format("  Pet: %s", item.pet))
    print(string.format("  JobID: %s", item.jobid:sub(1,8)))
    print(string.format("  Money: %.2fM", item.moneyM or 0))
    
    local petName = item.pet or "Unknown"
    local jobid = item.jobid
    local moneyM = item.moneyM or 0
    local playersStr = item.players or "1/12"
    local placeId = item.placeId
    
    -- ✅ Nếu job này đã auto-join trước đó -> bỏ qua
    if AutoUsedJobs and AutoUsedJobs[jobid] then
        print("[FILTER] ❌ Job đã auto-join trước đó, bỏ qua")
        return
    end
    
    -- ✅ BẮT BUỘC: Pet phải được tick trong rarity & pets
    local selected = isPetSelected(petName)
    if not selected then 
        print(string.format("[FILTER] ❌ Pet không được tick: %s", petName))
        return 
    end
    
    -- Nếu không có trong danh sách Secret/OG cũng bỏ qua
    local isInList = false
    for _, name in ipairs(SecretList) do
        if string.lower(name) == string.lower(petName) then
            isInList = true
            break
        end
    end
    if not isInList then
        for _, name in ipairs(OGList) do
            if string.lower(name) == string.lower(petName) then
                isInList = true
                break
            end
        end
    end
    
    if not isInList then
        print(string.format("[FILTER] ❌ Pet không có trong danh sách: %s", petName))
        return
    end
    
    local priority = isPetPriority(petName)
    
    local customMoney = PetMoneyThresholds[petName] or 0
    local globalMoney = MinMoneyM or 0
    local requiredMoney = math.max(customMoney, globalMoney)
    
    local moneyOk = (moneyM >= requiredMoney)
    local tierOk = tierMatch(moneyM)
    
    print(string.format("[FILTER] Processing: %s | Selected: %s | InList: %s | Priority: %s", 
        petName, tostring(selected), tostring(isInList), tostring(priority)))
    
    if moneyOk and tierOk then
        table.insert(allMatchedResults, {
            pet = petName,
            jobid = jobid,
            moneyM = moneyM,
            players = playersStr,
            placeId = placeId,
            isPriority = priority,
            moneyOk = true,
            tierOk = true,
            timestamp = item.timestamp or os.time()
        })
        print(string.format("[FILTER] ✅ Thêm vào danh sách: %s (%.2fM)", petName, moneyM))
    else
        print(string.format("[FILTER] ❌ Không đạt điều kiện: money %.2fM < %.2fM hoặc tier không khớp", 
            moneyM, requiredMoney))
    end
end

local function processAndDisplayResults()
    if #allMatchedResults == 0 then
        print("[AUTOJOIN] Không có kết quả phù hợp")
        return
    end

    -- Sắp xếp: ưu tiên pet priority trước
    table.sort(allMatchedResults, function(a, b)
        if a.isPriority and not b.isPriority then return true end
        if not a.isPriority and b.isPriority then return false end
        return a.moneyM > b.moneyM  -- Ưu tiên money cao hơn
    end)

    print("=== DANH SÁCH PET ĐƯỢC TICK ===")
    for i, result in ipairs(allMatchedResults) do
        print(string.format("[%d] %s | %.2fM | Priority: %s | MoneyOK: %s | TierOK: %s",
            i, result.pet, result.moneyM, tostring(result.isPriority),
            tostring(result.moneyOk), tostring(result.tierOk)))
    end
    
    -- AutoJoin logic - CHỈ JOIN PET ĐƯỢC TICK
    if AutoJoinOn and AutoFinderOn and SpamEnabled then
        print("[AUTOJOIN] Điều kiện:", 
            "AutoJoinOn:", AutoJoinOn, 
            "AutoFinderOn:", AutoFinderOn,
            "SpamEnabled:", SpamEnabled,
            "Pet được tick:", #allMatchedResults)
        
        if #allMatchedResults > 0 then
            local bestResult = allMatchedResults[1]
            
            -- KIỂM TRA LẠI: Pet phải được tick
            if not isPetSelected(bestResult.pet) then
                print(string.format("[AUTOJOIN] ❌ Pet không được tick: %s", bestResult.pet))
                allMatchedResults = {}
                return
            end
            
            print(string.format("[AUTOJOIN] ✅ Pet được tick: %s (%.2fM) | Priority: %s",
                bestResult.pet, bestResult.moneyM, tostring(bestResult.isPriority)))
            
            if bestResult.moneyOk and bestResult.tierOk then
                print(string.format("[AUTOJOIN] ⚡ TELEPORTING to %s...", bestResult.jobid:sub(1, 8)))
                joinSpam(bestResult.jobid, bestResult.placeId or game.PlaceId, true)
            else
                print("[AUTOJOIN] ❌ Không teleport - Money hoặc Tier không đạt")
            end
        else
            print("[AUTOJOIN] ❌ Không có pet nào được tick và đạt điều kiện")
        end
    elseif AutoJoinOn and AutoFinderOn and not SpamEnabled then
        print("[AUTOJOIN] ❌ Spam disabled (nhấn ,), bỏ qua auto-join")
    end

    allMatchedResults = {}
end
----------------------------------------------------------------
-- 🔥 HTTPS POLLING - ĐỌC TỪ FASTAPI (/items/simple)
----------------------------------------------------------------
local function fetchFromAPI()
    local url = SERVER_TUNNEL_URL .. "/jobs"
    
    local success, result = doRequest({
        Url = url,
        Method = "GET",
        Headers = {
            ["Accept"] = "application/json",
            ["User-Agent"] = "RobloxAutoJoin"
        },
        Timeout = 5
    })
    
    if not success then
        print("[API] ❌ Lỗi kết nối:", result)
        return nil
    end
    
    print("[API] Response Body (raw):", result.Body:sub(1, 500))  -- In 500 ký tự đầu
    
    local ok, data = pcall(function()
        return HttpService:JSONDecode(result.Body)
    end)
    
    if not ok then
        print("[API] ❌ Lỗi JSON")
        return nil
    end
    
    -- Debug chi tiết cấu trúc
    print("[DEBUG] API Response type:", type(data))
    if type(data) == "table" then
        print("[DEBUG] Keys in data:")
        for k, v in pairs(data) do
            print("  ", k, "=> type:", type(v))
            if k == "items" then
                print("    Items count:", #v)
                if #v > 0 then
                    print("    First item keys:")
                    for k2, v2 in pairs(v[1]) do
                        print("      ", k2, "=>", type(v2), ":", tostring(v2))
                    end
                end
            end
        end
    end
    
    -- Nếu server trả {ok: true, items: [...]}
    if data and data.ok and data.items then
        return data.items
    -- Nếu server trả trực tiếp mảng [...]
    elseif type(data) == "table" and #data > 0 then
        return data
    end
    
    return nil
end
    
    

-- Chuẩn hoá data API -> list items để xử lý
local function parseServerData(apiItems) 
    local out = {}
    if not apiItems then return out end
    
    print("[PARSE] Parsing", #apiItems, "items")
    
    for _, item in ipairs(apiItems) do
        -- Debug từng item
        print("[PARSE] Item:", item)
        
        -- Tìm jobId trong các trường có thể có
        local jobId = item.jobId or item.jobid or item.job_id or item.id
        
        if jobId and jobId ~= "" then
            local jobData = {
                pet = item.name or item.pet or item.petName or "Unknown",
                jobid = jobId,
                money = tonumber(item.money) or tonumber(item.moneyValue) or 0,
                moneyM = (tonumber(item.money) or tonumber(item.moneyValue) or 0) / 1000000,
                players = item.players or item.playerCount or "1/12",
                placeId = tonumber(item.placeId) or game.PlaceId
            }
            
            -- Debug parsed data
            print(string.format("[PARSE] Parsed: %s | %s | %.2fM",
                jobData.pet, jobData.jobid:sub(1,8), jobData.moneyM))
            
            table.insert(out, jobData)
        else
            print("[PARSE] ❌ Item không có jobId:", item)
        end
    end
    
    print(string.format("[PARSE] Tổng jobs hợp lệ: %d", #out))
    return out
end

-- Poll loop
task.spawn(function()
    while true do
        if AutoFinderOn then
            print("[POLL] === Starting poll ===")
            
            -- 1. Fetch từ API
            local apiItems = fetchFromAPI()
            
            if apiItems and #apiItems > 0 then
                print(string.format("[POLL] Nhận %d items từ API", #apiItems))
                
                -- 2. Parse data
                local jobList = parseServerData(apiItems)
                
                if #jobList > 0 then
                    print(string.format("[POLL] %d jobs hợp lệ để xử lý", #jobList))
                    
                    -- 3. Xử lý từng job
                    for _, jobItem in ipairs(jobList) do
                        print("[POLL] Processing job:", jobItem.pet, jobItem.jobid:sub(1,8))
                        handleParsed(jobItem)
                    end
                    
                    -- 4. Auto-join nếu bật
                    if AutoJoinOn and SpamEnabled then
                        processAndDisplayResults()
                    end
                else
                    print("[POLL] Không có job nào hợp lệ sau khi parse")
                end
            else
                print("[POLL] Không có dữ liệu từ API hoặc apiItems = nil")
            end
            
            print("[POLL] === Poll completed ===")
        end
        
        -- Delay theo trạng thái
        local delay = AutoJoinOn and 0.5 or 2
        task.wait(delay)
    end
end)
----------------------------------------------------------------
-- ẨN POPUP LỖI TELEPORT 772 / 773
----------------------------------------------------------------
task.spawn(function()
    local function clickOkIn(root)
        for _, btn in ipairs(root:GetDescendants()) do
            if btn:IsA("TextButton") then
                local t = string.lower(btn.Text or "")
                if t:find("ok") or t:find("đóng") or t:find("close") then
                    pcall(function()
                        if getconnections then
                            for _, conn in ipairs(getconnections(btn.MouseButton1Click)) do
                                conn:Fire()
                            end
                        else
                            btn:Activate()
                        end
                    end)
                end
            end
        end
    end

    while task.wait(0.3) do
        pcall(function()
            local pg = LocalPlayer:FindFirstChild("PlayerGui")
            if pg then
                for _, gui in ipairs(pg:GetChildren()) do
                    if gui:IsA("ScreenGui") and gui.Enabled then
                        for _, lbl in ipairs(gui:GetDescendants()) do
                            if lbl:IsA("TextLabel") or lbl:IsA("TextButton") then
                                local text = string.lower(lbl.Text or "")
                                if text:find("772") or text:find("773") then
                                    clickOkIn(gui)
                                    break
                                end
                            end
                        end
                    end
                end
            end

            local promptGui = CoreGui:FindFirstChild("RobloxPromptGui")
            if promptGui then
                for _, lbl in ipairs(promptGui:GetDescendants()) do
                    if lbl:IsA("TextLabel") or lbl:IsA("TextButton") then
                        local text = string.lower(lbl.Text or "")
                        if text:find("772") or text:find("773") then
                            clickOkIn(promptGui)
                            break
                        end
                    end
                end
            end
        end)
    end
end)

end) -- End of main pcall

if not success then
    warn("❌ CT FINDER ERROR: " .. tostring(errorMsg))
end

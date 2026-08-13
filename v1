--// Name + Level + Distance + HP + CTAG + Larger Box ESP

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local Live = workspace:FindFirstChild("Live")
local Camera = workspace.CurrentCamera

if not Live then
    warn("[ESP] workspace.Live not found")
    return
end

if not Camera then
    warn("[ESP] CurrentCamera not found")
    return
end

local ESP = {}

--------------------------------------------------
-- SETTINGS
--------------------------------------------------

local TEXT_COLOR = Color3.fromRGB(34, 139, 34)
local BAR_COLOR = Color3.fromRGB(34, 139, 34)
local BOX_COLOR = Color3.fromRGB(255, 255, 255)

local BLACK = Color3.fromRGB(0, 0, 0)
local EMPTY_COLOR = Color3.fromRGB(35, 35, 35)

local TEXT_SIZE = 12
local TEXT_LINE_HEIGHT = 13

-- Text is now higher above the player's head
local TEXT_WORLD_OFFSET = Vector3.new(0, 3, 0)

-- HP + box range
local MAX_DETAIL_DISTANCE = 300

-- HP bar
local BAR_WIDTH = 16
local BAR_X_OFFSET = 76
local OFFSET_SCALE = 0.7

local OUTLINE_SIZE = 1
local INNER_PADDING = 1

local HEIGHT_MULTIPLIER = 0.9

local MIN_BAR_HEIGHT = 8
local MAX_BAR_HEIGHT = 120

local MIN_BAR_WIDTH = 4
local WIDTH_SCALE = 0.18

-- Larger bounding box
local BOX_WIDTH_SCALE = 0.72
local BOX_HEIGHT_SCALE = 1.12

--------------------------------------------------
-- TARGET FILTER
--------------------------------------------------

local function IsValidTarget(Character)
    local ok, Name = pcall(function()
        return Character.Name
    end)

    if not ok or type(Name) ~= "string" then
        return false
    end

    -- Ignore internal models
    if string.sub(Name, 1, 1) == "." then
        return false
    end

    -- Ignore ourselves
    if LocalPlayer and Name == LocalPlayer.Name then
        return false
    end

    return true
end

--------------------------------------------------
-- CHARACTER NAME
--------------------------------------------------

local function GetCharacterName(Character)
    local Humanoid = Character:FindFirstChild("Humanoid")

    if not Humanoid then
        return nil
    end

    local ok, CharacterName = pcall(function()
        return Humanoid:GetAttribute("CharacterName")
    end)

    if not ok then
        return nil
    end

    if type(CharacterName) ~= "string" then
        return nil
    end

    if CharacterName == "" then
        return nil
    end

    return CharacterName
end

--------------------------------------------------
-- LEVEL
--------------------------------------------------

local function GetPlayerLevel(Character)
    local ok, Level = pcall(function()
        return Character:GetAttribute("Level")
    end)

    if not ok or Level == nil then
        return nil
    end

    local NumericLevel = tonumber(Level)

    if not NumericLevel then
        return nil
    end

    return math.floor(NumericLevel)
end

--------------------------------------------------
-- NAME + LEVEL
--------------------------------------------------

local function GetDisplayText(Character)
    local CharacterName = GetCharacterName(Character)

    if not CharacterName then
        return nil
    end

    local Level = GetPlayerLevel(Character)

    if Level then
        return CharacterName
            .. " ["
            .. tostring(Level)
            .. "]"
    end

    return CharacterName
end

--------------------------------------------------
-- DISTANCE
--------------------------------------------------

local function GetDistance(Character)
    if not LocalPlayer then
        return nil
    end

    local LocalCharacter =
        Live:FindFirstChild(LocalPlayer.Name)

    if not LocalCharacter then
        return nil
    end

    local LocalRoot =
        LocalCharacter:FindFirstChild("HumanoidRootPart")

    local TargetRoot =
        Character:FindFirstChild("HumanoidRootPart")

    if not LocalRoot or not TargetRoot then
        return nil
    end

    local ok, Distance = pcall(function()
        local A = LocalRoot.Position
        local B = TargetRoot.Position

        local dx = B.X - A.X
        local dy = B.Y - A.Y
        local dz = B.Z - A.Z

        return math.sqrt(
            dx * dx +
            dy * dy +
            dz * dz
        )
    end)

    if not ok then
        return nil
    end

    return Distance
end

--------------------------------------------------
-- COMBAT TAG
--------------------------------------------------

local function GetCombatTags(Character)
    local Humanoid = Character:FindFirstChild("Humanoid")

    if not Humanoid then
        return 0
    end

    local ok, DangerCounter = pcall(function()
        return Humanoid:GetAttribute("DangerCounter")
    end)

    if not ok or DangerCounter == nil then
        return 0
    end

    local Count = tonumber(DangerCounter)

    if not Count then
        return 0
    end

    return math.max(
        0,
        math.floor(Count)
    )
end

--------------------------------------------------
-- HEALTH
--------------------------------------------------

local function GetHealthData(Character)
    local Humanoid = Character:FindFirstChild("Humanoid")

    if not Humanoid then
        return nil
    end

    local HealthOK, Health = pcall(function()
        return Humanoid.Health
    end)

    local MaxHealthOK, MaxHealth = pcall(function()
        return Humanoid.MaxHealth
    end)

    if not HealthOK or not MaxHealthOK then
        return nil
    end

    if type(Health) ~= "number" then
        return nil
    end

    if type(MaxHealth) ~= "number" then
        return nil
    end

    if MaxHealth <= 0 then
        return nil
    end

    local Ratio =
        math.max(
            0,
            math.min(
                1,
                Health / MaxHealth
            )
        )

    local Percent =
        math.floor(
            (Ratio * 100) + 0.5
        )

    return {
        Health = Health,
        MaxHealth = MaxHealth,
        Ratio = Ratio,
        Percent = Percent
    }
end

--------------------------------------------------
-- CREATE ESP
--------------------------------------------------

local function CreateESP(Character)
    if ESP[Character] then
        return
    end

    if not IsValidTarget(Character) then
        return
    end

    local Head = Character:FindFirstChild("Head")
    local Torso = Character:FindFirstChild("Torso")

    if not Head or not Torso then
        return
    end

    local DisplayText = GetDisplayText(Character)
    local HealthData = GetHealthData(Character)

    if not DisplayText or not HealthData then
        return
    end

    --------------------------------------------------
    -- NAME + LEVEL
    --------------------------------------------------

    local NameText = Drawing.new("Text")

    NameText.Text = DisplayText
    NameText.Size = TEXT_SIZE
    NameText.Center = true
    NameText.Outline = true
    NameText.Color = TEXT_COLOR
    NameText.Visible = false

    --------------------------------------------------
    -- DISTANCE + HP
    --------------------------------------------------

    local InfoText = Drawing.new("Text")

    InfoText.Text = ""
    InfoText.Size = TEXT_SIZE
    InfoText.Center = true
    InfoText.Outline = true
    InfoText.Color = TEXT_COLOR
    InfoText.Visible = false

    --------------------------------------------------
    -- CTAG
    --------------------------------------------------

    local CombatText = Drawing.new("Text")

    CombatText.Text = ""
    CombatText.Size = TEXT_SIZE
    CombatText.Center = true
    CombatText.Outline = true
    CombatText.Color = TEXT_COLOR
    CombatText.Visible = false

    --------------------------------------------------
    -- HP BAR
    --------------------------------------------------

    local Outline = Drawing.new("Square")

    Outline.Filled = true
    Outline.Color = BLACK
    Outline.Visible = false

    local Background = Drawing.new("Square")

    Background.Filled = true
    Background.Color = EMPTY_COLOR
    Background.Visible = false

    local Fill = Drawing.new("Square")

    Fill.Filled = true
    Fill.Color = BAR_COLOR
    Fill.Visible = false

    --------------------------------------------------
    -- HP DIVIDERS
    --------------------------------------------------

    local Dividers = {}

    for i = 1, 4 do
        local Line = Drawing.new("Line")

        Line.Color = BLACK
        Line.Thickness = 1
        Line.Visible = false

        Dividers[i] = Line
    end

    --------------------------------------------------
    -- BOX
    --------------------------------------------------

    local BoxLines = {}

    for i = 1, 4 do
        local Line = Drawing.new("Line")

        Line.Color = BOX_COLOR
        Line.Thickness = 1
        Line.Visible = false

        BoxLines[i] = Line
    end

    --------------------------------------------------

    ESP[Character] = {
        NameText = NameText,
        InfoText = InfoText,
        CombatText = CombatText,

        DisplayText = DisplayText,

        Outline = Outline,
        Background = Background,
        Fill = Fill,
        Dividers = Dividers,

        BoxLines = BoxLines
    }
end

--------------------------------------------------
-- HIDE HP BAR
--------------------------------------------------

local function HideHealthBar(Data)
    Data.Outline.Visible = false
    Data.Background.Visible = false
    Data.Fill.Visible = false

    for _, Line in ipairs(Data.Dividers) do
        Line.Visible = false
    end
end

--------------------------------------------------
-- HIDE BOX
--------------------------------------------------

local function HideBox(Data)
    for _, Line in ipairs(Data.BoxLines) do
        Line.Visible = false
    end
end

--------------------------------------------------
-- HIDE ALL
--------------------------------------------------

local function HideESP(Data)
    Data.NameText.Visible = false
    Data.InfoText.Visible = false
    Data.CombatText.Visible = false

    HideHealthBar(Data)
    HideBox(Data)
end

--------------------------------------------------
-- UPDATE ESP
--------------------------------------------------

local function UpdateESP(Character)
    local Data = ESP[Character]

    if not Data then
        CreateESP(Character)
        return
    end

    if not IsValidTarget(Character) then
        HideESP(Data)
        return
    end

    local Head = Character:FindFirstChild("Head")
    local Torso = Character:FindFirstChild("Torso")

    local LeftLeg =
        Character:FindFirstChild("Left Leg")

    local RightLeg =
        Character:FindFirstChild("Right Leg")

    if not Head or not Torso then
        HideESP(Data)
        return
    end

    local DisplayText = GetDisplayText(Character)
    local HealthData = GetHealthData(Character)
    local CombatTags = GetCombatTags(Character)
    local Distance = GetDistance(Character)

    if
        not DisplayText
        or not HealthData
        or not Distance
    then
        HideESP(Data)
        return
    end

    --------------------------------------------------
    -- UPDATE NAME
    --------------------------------------------------

    if Data.DisplayText ~= DisplayText then
        Data.DisplayText = DisplayText
        Data.NameText.Text = DisplayText
    end

    --------------------------------------------------
    -- TEXT ANCHOR
    -- NOW +3 STUDS ABOVE HEAD
    --------------------------------------------------

    local TextOK,
        TextScreen,
        TextOnScreen =
        pcall(function()

        return Camera:WorldToScreenPoint(
            Head.Position
            + TEXT_WORLD_OFFSET
        )
    end)

    if
        not TextOK
        or not TextScreen
        or not TextOnScreen
    then
        Data.NameText.Visible = false
        Data.InfoText.Visible = false
        Data.CombatText.Visible = false

    else

        --------------------------------------------------
        -- LINE 1
        --------------------------------------------------

        Data.NameText.Position =
            Vector2.new(
                TextScreen.X,
                TextScreen.Y
            )

        Data.NameText.Visible = true

        --------------------------------------------------
        -- LINE 2
        --------------------------------------------------

        local RoundedDistance =
            math.floor(
                Distance + 0.5
            )

        Data.InfoText.Text =
            tostring(RoundedDistance)
            .. " Studs ["
            .. tostring(HealthData.Percent)
            .. "%]"

        Data.InfoText.Position =
            Vector2.new(
                TextScreen.X,
                TextScreen.Y
                + TEXT_LINE_HEIGHT
            )

        Data.InfoText.Visible = true

        --------------------------------------------------
        -- LINE 3
        --------------------------------------------------

        if CombatTags > 0 then
            Data.CombatText.Text =
                "CTAG: "
                .. tostring(CombatTags)

            Data.CombatText.Position =
                Vector2.new(
                    TextScreen.X,
                    TextScreen.Y
                    + (TEXT_LINE_HEIGHT * 2)
                )

            Data.CombatText.Visible = true
        else
            Data.CombatText.Visible = false
        end
    end

    --------------------------------------------------
    -- TORSO PROJECTION
    --------------------------------------------------

    local TorsoOK,
        TorsoScreen,
        TorsoOnScreen =
        pcall(function()

        return Camera:WorldToScreenPoint(
            Torso.Position
        )
    end)

    if
        not TorsoOK
        or not TorsoScreen
        or not TorsoOnScreen
    then
        HideHealthBar(Data)
        HideBox(Data)
        return
    end

    --------------------------------------------------
    -- BODY TOP
    --------------------------------------------------

    local HeadOK, HeadScreen =
        pcall(function()

        return Camera:WorldToScreenPoint(
            Head.Position
            + Vector3.new(0, 1, 0)
        )
    end)

    if not HeadOK or not HeadScreen then
        HideHealthBar(Data)
        HideBox(Data)
        return
    end

    --------------------------------------------------
    -- BODY BOTTOM
    --------------------------------------------------

    local BottomPart =
        LeftLeg
        or RightLeg
        or Torso

    local BottomOK, BottomScreen =
        pcall(function()

        return Camera:WorldToScreenPoint(
            BottomPart.Position
            - Vector3.new(0, 1, 0)
        )
    end)

    if not BottomOK or not BottomScreen then
        HideHealthBar(Data)
        HideBox(Data)
        return
    end

    --------------------------------------------------
    -- BODY HEIGHT
    --------------------------------------------------

    local BodyHeight =
        math.abs(
            BottomScreen.Y
            - HeadScreen.Y
        )

    --------------------------------------------------
    -- BOX
    -- ONLY WITHIN 300 STUDS
    --------------------------------------------------

    if Distance <= MAX_DETAIL_DISTANCE then

        local OriginalTop =
            math.min(
                HeadScreen.Y,
                BottomScreen.Y
            )

        local OriginalBottom =
            math.max(
                HeadScreen.Y,
                BottomScreen.Y
            )

        local OriginalCenterY =
            (
                OriginalTop
                + OriginalBottom
            ) / 2

        --------------------------------------------------
        -- LARGER BOX
        --------------------------------------------------

        local BoxHeight =
            BodyHeight
            * BOX_HEIGHT_SCALE

        local BoxWidth =
            BodyHeight
            * BOX_WIDTH_SCALE

        local BoxTop =
            OriginalCenterY
            - (BoxHeight / 2)

        local BoxBottom =
            OriginalCenterY
            + (BoxHeight / 2)

        local BoxLeft =
            TorsoScreen.X
            - (BoxWidth / 2)

        local BoxRight =
            TorsoScreen.X
            + (BoxWidth / 2)

        --------------------------------------------------
        -- TOP
        --------------------------------------------------

        Data.BoxLines[1].From =
            Vector2.new(
                BoxLeft,
                BoxTop
            )

        Data.BoxLines[1].To =
            Vector2.new(
                BoxRight,
                BoxTop
            )

        --------------------------------------------------
        -- BOTTOM
        --------------------------------------------------

        Data.BoxLines[2].From =
            Vector2.new(
                BoxLeft,
                BoxBottom
            )

        Data.BoxLines[2].To =
            Vector2.new(
                BoxRight,
                BoxBottom
            )

        --------------------------------------------------
        -- LEFT
        --------------------------------------------------

        Data.BoxLines[3].From =
            Vector2.new(
                BoxLeft,
                BoxTop
            )

        Data.BoxLines[3].To =
            Vector2.new(
                BoxLeft,
                BoxBottom
            )

        --------------------------------------------------
        -- RIGHT
        --------------------------------------------------

        Data.BoxLines[4].From =
            Vector2.new(
                BoxRight,
                BoxTop
            )

        Data.BoxLines[4].To =
            Vector2.new(
                BoxRight,
                BoxBottom
            )

        for _, Line in ipairs(Data.BoxLines) do
            Line.Visible = true
        end

    else
        HideBox(Data)
    end

    --------------------------------------------------
    -- HP BAR
    -- ONLY WITHIN 300 STUDS
    --------------------------------------------------

    if Distance > MAX_DETAIL_DISTANCE then
        HideHealthBar(Data)
        return
    end

    --------------------------------------------------
    -- BAR HEIGHT
    --------------------------------------------------

    local BarHeight =
        BodyHeight
        * HEIGHT_MULTIPLIER

    BarHeight =
        math.max(
            MIN_BAR_HEIGHT,
            math.min(
                MAX_BAR_HEIGHT,
                BarHeight
            )
        )

    --------------------------------------------------
    -- BAR WIDTH
    --------------------------------------------------

    local ScaledWidth =
        math.max(
            MIN_BAR_WIDTH,
            math.min(
                BAR_WIDTH,
                BarHeight * WIDTH_SCALE
            )
        )

    --------------------------------------------------
    -- BAR OFFSET
    --------------------------------------------------

    local ScaledOffset =
        math.min(
            BAR_X_OFFSET,
            BarHeight * OFFSET_SCALE
        )

    --------------------------------------------------
    -- BAR POSITION
    --------------------------------------------------

    local BarX =
        TorsoScreen.X
        - ScaledOffset

    local BarY =
        TorsoScreen.Y
        - (BarHeight / 2)

    --------------------------------------------------
    -- BLACK OUTLINE
    --------------------------------------------------

    Data.Outline.Position =
        Vector2.new(
            BarX - OUTLINE_SIZE,
            BarY - OUTLINE_SIZE
        )

    Data.Outline.Size =
        Vector2.new(
            ScaledWidth
            + (OUTLINE_SIZE * 2),

            BarHeight
            + (OUTLINE_SIZE * 2)
        )

    Data.Outline.Visible = true

    --------------------------------------------------
    -- HP BACKGROUND
    --------------------------------------------------

    Data.Background.Position =
        Vector2.new(
            BarX,
            BarY
        )

    Data.Background.Size =
        Vector2.new(
            ScaledWidth,
            BarHeight
        )

    Data.Background.Visible = true

    --------------------------------------------------
    -- HP FILL
    --------------------------------------------------

    local InnerWidth =
        math.max(
            1,
            ScaledWidth
            - (INNER_PADDING * 2)
        )

    local InnerHeight =
        math.max(
            1,
            BarHeight
            - (INNER_PADDING * 2)
        )

    local FillHeight =
        InnerHeight
        * HealthData.Ratio

    local FillX =
        BarX
        + INNER_PADDING

    local FillY =
        BarY
        + BarHeight
        - INNER_PADDING
        - FillHeight

    Data.Fill.Position =
        Vector2.new(
            FillX,
            FillY
        )

    Data.Fill.Size =
        Vector2.new(
            InnerWidth,
            FillHeight
        )

    Data.Fill.Visible =
        FillHeight > 0

    --------------------------------------------------
    -- 20% HP DIVIDERS
    --------------------------------------------------

    local SectionHeight =
        BarHeight / 5

    for i = 1, 4 do
        local DividerY =
            BarY
            + (SectionHeight * i)

        local Line =
            Data.Dividers[i]

        Line.From =
            Vector2.new(
                BarX,
                DividerY
            )

        Line.To =
            Vector2.new(
                BarX + ScaledWidth,
                DividerY
            )

        Line.Visible = true
    end
end

--------------------------------------------------
-- CLEANUP
--------------------------------------------------

local function CleanupESP()
    local CurrentCharacters = {}

    for _, Character in ipairs(
        Live:GetChildren()
    ) do
        CurrentCharacters[Character] = true
    end

    local RemoveList = {}

    for Character in pairs(ESP) do
        if not CurrentCharacters[Character] then
            table.insert(
                RemoveList,
                Character
            )
        end
    end

    for _, Character in ipairs(RemoveList) do
        local Data = ESP[Character]

        if Data then
            pcall(function()

                Data.NameText:Remove()
                Data.InfoText:Remove()
                Data.CombatText:Remove()

                Data.Outline:Remove()
                Data.Background:Remove()
                Data.Fill:Remove()

                for _, Line in ipairs(
                    Data.Dividers
                ) do
                    Line:Remove()
                end

                for _, Line in ipairs(
                    Data.BoxLines
                ) do
                    Line:Remove()
                end
            end)
        end

        ESP[Character] = nil
    end
end

--------------------------------------------------
-- MAIN LOOP
--------------------------------------------------

print(
    "[ESP] Name + Level + Distance + HP + CTAG + Larger Box started"
)

while true do
    task.wait(0.03)

    for _, Character in ipairs(
        Live:GetChildren()
    ) do
        if IsValidTarget(Character) then
            UpdateESP(Character)
        end
    end

    CleanupESP()
end

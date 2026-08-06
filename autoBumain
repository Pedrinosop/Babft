local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local R = { highlights = {}, hlContainer = nil }
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SoundService = game:GetService("SoundService")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local bit = bit32

local LocalPlayer = Players.LocalPlayer or Players:WaitForChild("LocalPlayer", 30)
local function safeWaitChild(parent, childName, timeout)
    local started = tick()
    timeout = timeout or 30
    while parent and (tick() - started) < timeout do
        local ok, child = pcall(function()
            return parent:FindFirstChild(childName)
        end)
        if ok and child then return child end
        task.wait(0.05)
    end
    local ok, child = pcall(function()
        return parent and parent:FindFirstChild(childName)
    end)
    if ok then return child end
    return nil
end
local Character = LocalPlayer.Character
while not Character do
    Character = LocalPlayer.Character
    task.wait(0.05)
end
local function getHumanoid(model)
    if not model then return nil end
    local h = model:FindFirstChildOfClass("Humanoid") or model:FindFirstChild("Humanoid")
    if h then return h end
    return safeWaitChild(model, "Humanoid", 10)
end
local Humanoid = getHumanoid(Character)
local BlocksFolder = safeWaitChild(Workspace, "Blocks", 30) or safeWaitChild(Workspace, "Block", 30)
local BlockData = safeWaitChild(LocalPlayer, "Data", 30)
local BuildingParts = safeWaitChild(ReplicatedStorage, "BuildingParts", 30)

local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

local FOLDER_PATH = "SOPERA_WORKSPACE"
local FOLDER_PREFIX = FOLDER_PATH .. "/"
local SETTINGS_PATH = "SoPeRa2_Settings.json"
local CUSTOM_SCRIPTS_PATH = "SoPeRa2_CustomScripts.json"
local BUILD_SEARCH_PATHS = {
    FOLDER_PREFIX,
    "BABFT/",
    "BABFT/Build/",
    "Build/",
}

local PreviewFolder = Workspace:FindFirstChild("SPRB_Preview") or Instance.new("Folder")
PreviewFolder.Name = "SPRB_Preview"
PreviewFolder.Parent = Workspace

local Settings = {
    previewTransparency = 0.5,
    guiTransparency = 0.15,
    autoPreview = true,
    showBlockCounts = true,
    uiScale = isMobile and 0.72 or 1.0,
    mobileMode = isMobile,
    buildScale = 1.0,
    buildOffsetX = 0,
    buildOffsetY = 3,
    buildOffsetZ = 0,
    infBlockEnabled = false,
    infBlockSlot1 = 2,
    infBlockSlot2 = 3,
    skyHeight = 500,
    primaryColor = Color3.fromRGB(90, 60, 200),
    secondaryColor = Color3.fromRGB(180, 140, 255),
    uiMinimized = false,
    windowPosX = -1,
    windowPosY = -1,
    windowWidth = -1,
    windowHeight = -1,
    replacePosX = -1,
    replacePosY = -1,
    replaceW = -1,
    replaceH = -1,
    paintPosX = -1,
    paintPosY = -1,
    paintW = -1,
    paintH = -1,

    excludedBlocks = {},
    buildSpeed = 0,
    bgMode = "default",
    bgAnim = "grid",
    bgAnimEnabled = true,
    bgAnimAutoColor = true,
    bgAnimColor = Color3.fromRGB(90, 60, 200),
    bgAnimCount = 12,
    bgAnimSpeed = 1.0,
    bgAnimSize = 1.0,
    bgCustomColor = Color3.fromRGB(12, 8, 24),
    blockReplacements = {},
    lang = "en",
}

local Settings_DEFAULTS = {}
for _k, _v in pairs(Settings) do
    Settings_DEFAULTS[_k] = _v
end

local selectedPlayer = nil
local currentBuild = {}
local isBuilding = false
local stopBuild = false
local shareBlocksOriginal = false
local previewActive = false
local selectedObjectName = nil
local previewParts = {}
local selectionBoxes = {}

local updatePreviewButtonGlobal = nil
local updateBlocksDisplayGlobal = nil
local StatusLabelRef = nil
local MiscStatusLabelRef = nil
local InfProgressLabelRef = nil
local InfProgressFillRef = nil
local ProgressBarFillRef = nil
local DupeInfoLabelRef = nil
local DupePercentLabelRef = nil

local Colors = {
    BG = Color3.fromRGB(8, 8, 8),
    Panel = Color3.fromRGB(18, 18, 18),
    PanelSoft = Color3.fromRGB(14, 14, 14),
    PanelElevated = Color3.fromRGB(24, 24, 24),
    Border = Color3.fromRGB(255, 255, 255),
    Text = Color3.fromRGB(255, 255, 255),
    Muted = Color3.fromRGB(140, 140, 140),
    ActiveBG = Color3.fromRGB(255, 255, 255),
    ActiveText = Color3.fromRGB(0, 0, 0),
    Green = Color3.fromRGB(80, 200, 80),
    Red = Color3.fromRGB(200, 80, 80),
    AccentSoft = Color3.fromRGB(180, 180, 180),
    AccentGlow = Color3.fromRGB(235, 235, 235),
}

local UISoundConfig = {
    volume = 2,
    click = "rbxassetid://139719503904449",
    open = "rbxassetid://136108770017536",
    close = "rbxassetid://119354387183704",
    hover = "rbxassetid://7218169592",
    success = "rbxassetid://90420386076500",
    error = "rbxassetid://131661013076677",
    explode = "rbxassetid://139771888058836",
}

local function showFormatWarning(parentFrame)
    pcall(function()
        local overlay = Instance.new("Frame")
        overlay.Size = UDim2.new(1, 0, 1, 0)
        overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        overlay.BackgroundTransparency = 0.4
        overlay.ZIndex = 500
        overlay.Parent = parentFrame
        local oCr = Instance.new("UICorner"); oCr.CornerRadius = UDim.new(0, 8); oCr.Parent = overlay

        local card = Instance.new("Frame")
        card.Size = UDim2.new(0, 280, 0, 120)
        card.Position = UDim2.new(0.5, -140, 0.5, -60)
        card.BackgroundColor3 = Color3.fromRGB(30, 20, 20)
        card.BorderSizePixel = 0
        card.ZIndex = 501
        card.Parent = parentFrame
        local cCr = Instance.new("UICorner"); cCr.CornerRadius = UDim.new(0, 10); cCr.Parent = card
        local cSt = Instance.new("UIStroke"); cSt.Color = Color3.fromRGB(255, 80, 80); cSt.Thickness = 2; cSt.Parent = card

        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -20, 0, 28)
        title.Position = UDim2.new(0, 10, 0, 8)
        title.BackgroundTransparency = 1
        title.Text = "⚠ Format Not Recognized"
        title.TextColor3 = Color3.fromRGB(255, 100, 100)
        title.TextSize = 14
        title.Font = Enum.Font.GothamBold
        title.ZIndex = 502
        title.Parent = card

        local body = Instance.new("TextLabel")
        body.Size = UDim2.new(1, -20, 0, 36)
        body.Position = UDim2.new(0, 10, 0, 38)
        body.BackgroundTransparency = 1
        body.Text = "All working files are in Telegram:"
        body.TextColor3 = Color3.fromRGB(200, 200, 200)
        body.TextSize = 11
        body.Font = Enum.Font.Gotham
        body.TextWrapped = true
        body.ZIndex = 502
        body.Parent = card

        local tgBtn = Instance.new("TextButton")
        tgBtn.Size = UDim2.new(0, 140, 0, 28)
        tgBtn.Position = UDim2.new(0.5, -70, 0, 78)
        tgBtn.BackgroundColor3 = Color3.fromRGB(40, 100, 200)
        tgBtn.BorderSizePixel = 0
        tgBtn.Text = "@babft"
        tgBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        tgBtn.TextSize = 13
        tgBtn.Font = Enum.Font.GothamBold
        tgBtn.AutoButtonColor = false
        tgBtn.ZIndex = 502
        tgBtn.Parent = card
        local bCr = Instance.new("UICorner"); bCr.CornerRadius = UDim.new(0, 6); bCr.Parent = tgBtn

        local copiedLabel = Instance.new("TextLabel")
        copiedLabel.Size = UDim2.new(0, 100, 0, 18)
        copiedLabel.Position = UDim2.new(0.5, -50, 1, -18)
        copiedLabel.BackgroundTransparency = 1
        copiedLabel.Text = ""
        copiedLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        copiedLabel.TextSize = 10
        copiedLabel.Font = Enum.Font.GothamBold
        copiedLabel.ZIndex = 502
        copiedLabel.Parent = card

        tgBtn.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            pcall(function() setclipboard("@babft") end)
            copiedLabel.Text = "Copied!"
            task.delay(1.5, function() if copiedLabel and copiedLabel.Parent then copiedLabel.Text = "" end end)
        end)


        overlay.InputBegan:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                pcall(function() overlay:Destroy() end)
                pcall(function() card:Destroy() end)
            end
        end)


        task.delay(8, function()
            pcall(function() overlay:Destroy() end)
            pcall(function() card:Destroy() end)
        end)
    end)
end

local function syncColors()
    Colors.Border = Settings.secondaryColor

    local textLum = (0.299 * Settings.secondaryColor.R) + (0.587 * Settings.secondaryColor.G) + (0.114 * Settings.secondaryColor.B)
    if textLum < 0.55 then
        Colors.Text = Settings.secondaryColor:Lerp(Color3.fromRGB(255, 255, 255), math.max(0.55 - textLum, 0.2))
    else
        Colors.Text = Settings.secondaryColor
    end
    Colors.ActiveBG = Settings.primaryColor
    local bg = Settings.primaryColor
    local lum = (0.299 * bg.R) + (0.587 * bg.G) + (0.114 * bg.B)
    if lum > 0.5 then
        Colors.ActiveText = Color3.fromRGB(20, 20, 20)
        Colors.Muted = Color3.fromRGB(60, 60, 60)
    else
        Colors.ActiveText = Color3.fromRGB(255, 255, 255)
        Colors.Muted = Settings.secondaryColor:Lerp(Color3.fromRGB(255, 255, 255), 0.18)
    end
    Colors.AccentSoft = Settings.secondaryColor:Lerp(Settings.primaryColor, 0.4)
    Colors.AccentGlow = Settings.primaryColor:Lerp(Color3.fromRGB(255, 255, 255), 0.28)

    Colors.BG = Color3.fromRGB(8, 8, 8):Lerp(Settings.primaryColor, 0.15)
    Colors.Panel = Color3.fromRGB(18, 18, 18):Lerp(Settings.primaryColor, 0.18)
    Colors.PanelSoft = Color3.fromRGB(14, 14, 14):Lerp(Settings.primaryColor, 0.16)
    Colors.PanelElevated = Color3.fromRGB(24, 24, 24):Lerp(Settings.primaryColor, 0.2)
end

local _bgAnimConns = {}
local function applyWindowBackground(MainFrame_ref)
    if not MainFrame_ref then return end

    for _, c in pairs(MainFrame_ref:GetChildren()) do
        if c.Name:find("^SPRB_") then
            c:Destroy()
        end
    end
    for _, conn in ipairs(_bgAnimConns) do
        pcall(function() conn:Disconnect() end)
    end
    _bgAnimConns = {}

    local mode = Settings.bgMode or "default"
    local gt = Settings.guiTransparency or 0.15

    local function applySolidBg(col)
        TweenService:Create(MainFrame_ref, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
            BackgroundTransparency = gt,
            BackgroundColor3 = col
        }):Play()
    end

    if mode == "color" then
        applySolidBg(Settings.bgCustomColor)
    else
        applySolidBg(Colors.BG)
    end

    if Settings.bgAnimEnabled then
        local anim = Settings.bgAnim or "lava"
        local animColor = Settings.bgAnimColor or Color3.fromRGB(90, 60, 200)
        local count = math.floor(Settings.bgAnimCount or 8)
        local speed = Settings.bgAnimSpeed or 1.0
        local container = Instance.new("Frame")
        container.Name = "SPRB_BgAnim"
        container.Size = UDim2.new(1, 0, 1, 0)
        container.BackgroundTransparency = 1
        container.ClipsDescendants = true
        container.ZIndex = 0
        container.Parent = MainFrame_ref


        if anim == "grid" then
            local animSize = Settings.bgAnimSize or 1.0
            local cellSize = math.clamp(math.floor(20 * animSize), 6, 80)
            local gap = 1
            local stride = cellSize + gap
            local cells = {}
            local randomCenter = {x = 0.5, y = 0.5, vx = 0.12, vy = 0.09}
            local autoColor = Settings.bgAnimAutoColor
            local h, s, v0 = Color3.toHSV(animColor)

            local colIdx = 0
            for gy = 0, 800, stride do
                colIdx = 0
                for gx = 0, 1200, stride do
                    local sq = Instance.new("Frame")
                    sq.Name = "SPRB_Grid_" .. gy .. "_" .. colIdx
                    sq.Size = UDim2.new(0, cellSize, 0, cellSize)
                    sq.Position = UDim2.new(0, gx, 0, gy)
                    sq.BackgroundColor3 = animColor
                    sq.BackgroundTransparency = 0.6
                    sq.BorderSizePixel = 0
                    sq.ZIndex = 0
                    sq.Parent = container
                    cells[#cells + 1] = {frame = sq, gx = gx, gy = gy}
                    colIdx = colIdx + 1
                end
            end



            local conn
            conn = RunService.RenderStepped:Connect(function(dt)
                if not container or not container.Parent then
                    pcall(function() conn:Disconnect() end)
                    return
                end
                local absW = container.AbsoluteSize.X
                local absH = container.AbsoluteSize.Y
                if absW < 1 or absH < 1 then return end


                randomCenter.x = randomCenter.x + randomCenter.vx * dt * speed
                randomCenter.y = randomCenter.y + randomCenter.vy * dt * speed
                if randomCenter.x < 0.1 or randomCenter.x > 0.9 then randomCenter.vx = -randomCenter.vx end
                if randomCenter.y < 0.1 or randomCenter.y > 0.9 then randomCenter.vy = -randomCenter.vy end
                local cx = randomCenter.x * absW
                local cy = randomCenter.y * absH

                local t = tick()


                local curH = h
                if autoColor then



                    local phase = (t * 0.04) % 1

                    local shift = math.sin(phase * math.pi * 2) * 0.08
                    local compShift = math.sin(phase * math.pi * 2 + math.pi) * 0.04
                    curH = (h + shift + compShift) % 1
                end
                for _, sq in ipairs(cells) do
                    local px = sq.gx + cellSize * 0.5
                    local py = sq.gy + cellSize * 0.5
                    local dx = px - cx
                    local dy = py - cy
                    local dist = math.sqrt(dx * dx + dy * dy)


                    local wave = (math.sin(dist * 0.02 - t * speed * 2.0) + 1) / 2

                    local hueShift = wave * 0.06

                    local lightness = 0.08 + wave * 0.18
                    local sat = s * (0.3 + wave * 0.5)
                    sq.frame.BackgroundColor3 = Color3.fromHSV((curH + hueShift) % 1, sat, lightness)

                    sq.frame.BackgroundTransparency = 0.25 + (1 - wave) * 0.45
                end
            end)
            table.insert(_bgAnimConns, conn)


        elseif anim == "constellation" then
            local particles = {}
            local pCount = math.clamp(count * 4, 12, 60)
            local linkDist = 0.18
            local autoColor = Settings.bgAnimAutoColor
            local h, s, v0 = Color3.toHSV(animColor)
            for i = 1, pCount do
                local p = Instance.new("Frame")
                p.Name = "SPRB_ConstP_" .. i
                p.Size = UDim2.new(0, 4, 0, 4)
                p.Position = UDim2.new(math.random(), 0, math.random(), 0)
                p.BackgroundColor3 = animColor
                p.BackgroundTransparency = 0.1
                p.BorderSizePixel = 0
                p.ZIndex = 0
                p.Parent = container
                local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(1, 0); cr.Parent = p
                particles[i] = {
                    frame = p,
                    vx = (math.random() - 0.5) * 0.06 * speed,
                    vy = (math.random() - 0.5) * 0.06 * speed,
                }
            end

            local linePool = {}
            local maxLines = pCount * 3
            for i = 1, maxLines do
                local ln = Instance.new("Frame")
                ln.Name = "SPRB_ConstL_" .. i
                ln.Size = UDim2.new(0, 1, 0, 1)
                ln.BackgroundColor3 = animColor
                ln.BackgroundTransparency = 1
                ln.BorderSizePixel = 0
                ln.ZIndex = 0
                ln.Parent = container
                linePool[i] = {frame = ln, active = false}
            end

            local conn
            conn = RunService.RenderStepped:Connect(function(dt)
                if not container or not container.Parent then
                    pcall(function() conn:Disconnect() end)
                    return
                end
                local absW = container.AbsoluteSize.X
                local absH = container.AbsoluteSize.Y
                if absW < 1 or absH < 1 then return end


                local curH = h
                if autoColor then
                    local phase = (tick() * 0.04) % 1
                    local shift = math.sin(phase * math.pi * 2) * 0.08
                    local compShift = math.sin(phase * math.pi * 2 + math.pi) * 0.04
                    curH = (h + shift + compShift) % 1
                end

                local curColor = Color3.fromHSV(curH, s, v0)
                for _, pt in ipairs(particles) do
                    local f = pt.frame
                    local px = f.Position.X.Scale + pt.vx * dt
                    local py = f.Position.Y.Scale + pt.vy * dt
                    if px < 0 or px > 1 then pt.vx = -pt.vx; px = math.clamp(px, 0, 1) end
                    if py < 0 or py > 1 then pt.vy = -pt.vy; py = math.clamp(py, 0, 1) end
                    f.Position = UDim2.new(px, 0, py, 0)
                    if autoColor then
                        f.BackgroundColor3 = curColor
                    end
                end

                local linkPx = linkDist * absW
                local lineIdx = 1
                for i = 1, #particles do
                    local a = particles[i].frame
                    local ax, ay = a.Position.X.Scale * absW, a.Position.Y.Scale * absH
                    for j = i + 1, #particles do
                        local b = particles[j].frame
                        local bx, by = b.Position.X.Scale * absW, b.Position.Y.Scale * absH
                        local dx = ax - bx
                        local dy = ay - by
                        local dist = math.sqrt(dx * dx + dy * dy)
                        if dist < linkPx and lineIdx <= maxLines then
                            local ln = linePool[lineIdx]
                            ln.active = true
                            local midX = (ax + bx) / 2
                            local midY = (ay + by) / 2
                            local len = math.sqrt(dx * dx + dy * dy)
                            local angle = math.atan2(dy, dx)
                            local alpha = 1 - dist / linkPx
                            ln.frame.Size = UDim2.new(0, len, 0, 1.5)
                            ln.frame.Position = UDim2.new(0, midX - len / 2, 0, midY - 0.75)
                            ln.frame.Rotation = math.deg(angle)
                            ln.frame.BackgroundTransparency = 1 - alpha * 0.5
                            if autoColor then
                                ln.frame.BackgroundColor3 = curColor
                            end
                            lineIdx = lineIdx + 1
                        end
                    end
                end
                for i = lineIdx, maxLines do
                    if linePool[i].active then
                        linePool[i].frame.BackgroundTransparency = 1
                        linePool[i].active = false
                    end
                end
            end)
            table.insert(_bgAnimConns, conn)


        elseif anim == "waves" then
            local layers = math.clamp(count, 3, 8)
            local autoColor = Settings.bgAnimAutoColor
            local waveDots = {}
            local dotsPerLayer = 30
            local h, s, v0 = Color3.toHSV(animColor)
            for l = 1, layers do
                local layerDots = {}
                for d = 0, dotsPerLayer do
                    local dot = Instance.new("Frame")
                    dot.Name = "SPRB_Wave_" .. l .. "_" .. d
                    local dotSz = 8
                    dot.Size = UDim2.new(0, dotSz, 0, dotSz)
                    dot.BackgroundColor3 = Color3.fromHSV((h + l * 0.04) % 1, s, v0 * (0.5 + l * 0.08))
                    dot.BackgroundTransparency = 0.2 + (l - 1) * 0.06
                    dot.BorderSizePixel = 0
                    dot.ZIndex = 0
                    dot.Parent = container
                    local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(1, 0); cr.Parent = dot
                    layerDots[#layerDots + 1] = {frame = dot, idx = d}
                end
                waveDots[l] = {dots = layerDots, layer = l}
            end

            local conn
            conn = RunService.RenderStepped:Connect(function(dt)
                if not container or not container.Parent then
                    pcall(function() conn:Disconnect() end)
                    return
                end
                local absW = container.AbsoluteSize.X
                local absH = container.AbsoluteSize.Y
                if absW < 1 or absH < 1 then return end

                local t = tick()

                local dotSize = math.clamp((Settings.bgAnimSize or 1.0) * 8, 3, 60)

                local curH = h
                if autoColor then
                    local phase = (t * 0.04) % 1
                    local shift = math.sin(phase * math.pi * 2) * 0.08
                    local compShift = math.sin(phase * math.pi * 2 + math.pi) * 0.04
                    curH = (h + shift + compShift) % 1
                end

                for _, wl in ipairs(waveDots) do
                    local l = wl.layer
                    local amp = (28 + l * 14) * (absH / 400)
                    local waveSpeed = 0.4 + l * 0.15
                    local yBase = absH * 0.5 + l * absH * 0.08 - layers * absH * 0.04
                    for _, dot in ipairs(wl.dots) do
                        local xFrac = dot.idx / dotsPerLayer
                        local px = xFrac * absW
                        local waveY = math.sin(xFrac * 6.28 + t * waveSpeed * speed + l) * amp
                        local py = yBase + waveY

                        dot.frame.Size = UDim2.new(0, dotSize, 0, dotSize)
                        dot.frame.Position = UDim2.new(0, px - dotSize * 0.5, 0, py - dotSize * 0.5)
                        if autoColor then
                            dot.frame.BackgroundColor3 = Color3.fromHSV((curH + l * 0.04) % 1, s, v0 * (0.5 + l * 0.08))
                        end
                    end
                end
            end)
            table.insert(_bgAnimConns, conn)


        elseif anim == "smoke" then
            local smokeParts = {}
            local sCount = math.clamp(count * 3, 8, 90)
            local animSize = Settings.bgAnimSize or 1.0
            local autoColor = Settings.bgAnimAutoColor
            local h, s, v0 = Color3.toHSV(animColor)

            for i = 1, sCount do
                local isSpark = math.random() < 0.3
                local p = Instance.new("Frame")
                p.Name = "SPRB_SmokeP_" .. i
                if isSpark then
                    local sz = math.random(2, 5) * animSize
                    p.Size = UDim2.new(0, sz, 0, sz)
                else
                    local sz = math.random(15, 50) * animSize
                    p.Size = UDim2.new(0, sz, 0, sz)
                end
                p.Position = UDim2.new(math.random() * 0.6 + 0.2, 0, 1, 0)
                p.BackgroundColor3 = animColor
                p.BackgroundTransparency = 0.6
                p.BorderSizePixel = 0
                p.ZIndex = 0
                p.Parent = container
                local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(1, 0); cr.Parent = p
                smokeParts[i] = {
                    frame = p,
                    isSpark = isSpark,
                    vy = isSpark and -(math.random() * 120 + 40) / 350 or -(math.random() * 30 + 10) / 350,
                    vx = (math.random() - 0.5) * 0.03,
                    life = math.random(),
                    maxLife = isSpark and (math.random() * 1.0 + 0.3) or (math.random() * 4 + 3),
                    baseSize = isSpark and math.random(2, 5) * animSize or math.random(15, 50) * animSize,
                    wobble = math.random() * 10,
                    growRate = isSpark and 0 or (math.random() * 0.3 + 0.4),
                }
            end

            local conn
            conn = RunService.RenderStepped:Connect(function(dt)
                if not container or not container.Parent then
                    pcall(function() conn:Disconnect() end)
                    return
                end
                local absW = container.AbsoluteSize.X
                local absH = container.AbsoluteSize.Y
                if absW < 1 or absH < 1 then return end


                local curH = h
                if autoColor then
                    local phase = (tick() * 0.04) % 1
                    local shift = math.sin(phase * math.pi * 2) * 0.08
                    local compShift = math.sin(phase * math.pi * 2 + math.pi) * 0.04
                    curH = (h + shift + compShift) % 1
                end

                for _, sp in ipairs(smokeParts) do
                    sp.life = sp.life + dt * speed * 1.0
                    local t = sp.life / sp.maxLife
                    if t > 1 then
                        sp.life = 0
                        t = 0
                        sp.frame.Position = UDim2.new(math.random() * 0.6 + 0.2, 0, 1, 0)
                        sp.vy = sp.isSpark and -(math.random() * 120 + 40) / 350 or -(math.random() * 30 + 10) / 350
                        sp.vx = (math.random() - 0.5) * 0.03
                        sp.isSpark = math.random() < 0.3
                    end

                    local px = sp.frame.Position.X.Scale + sp.vx * dt * speed
                    local py = sp.frame.Position.Y.Scale + sp.vy * dt * speed

                    px = px + math.sin(tick() * 1.5 + sp.wobble) * 0.004 * speed
                    sp.frame.Position = UDim2.new(px, 0, py, 0)

                    if sp.isSpark then

                        local scale = 1 - t * 0.8
                        local sz = math.max(2, sp.baseSize * scale * 0.6)
                        sp.frame.Size = UDim2.new(0, sz, 0, sz)

                        local sat = s * (1 - t * 0.9)
                        local val = v0 + (1 - v0) * t * 0.95
                        sp.frame.BackgroundColor3 = Color3.fromHSV(curH, sat, val)
                        local alpha = math.sin(math.clamp(t, 0, 1) * math.pi)
                        sp.frame.BackgroundTransparency = 1 - alpha * 0.7
                    else

                        local growFactor = 0.6 + t * sp.growRate
                        local sz = math.max(6, sp.baseSize * growFactor)
                        sp.frame.Size = UDim2.new(0, sz, 0, sz)

                        local sat = s * (1 - t * 0.7)
                        local val = v0 + (1 - v0) * t * 0.6
                        sp.frame.BackgroundColor3 = Color3.fromHSV(curH, sat * 0.5, val)

                        local alpha = math.sin(math.clamp(t, 0, 1) * math.pi)
                        sp.frame.BackgroundTransparency = 1 - alpha * 0.3
                    end
                end
            end)
            table.insert(_bgAnimConns, conn)


        elseif anim == "balls" then
            local balls = {}
            local bCount = math.clamp(count, 4, 30)
            local animSize = Settings.bgAnimSize or 1.0
            local autoColor = Settings.bgAnimAutoColor
            local h, s, v0 = Color3.toHSV(animColor)
            for i = 1, bCount do
                local sz = math.clamp(math.random(10, 30) * animSize, 6, 120)
                local b = Instance.new("Frame")
                b.Name = "SPRB_Ball_" .. i
                b.Size = UDim2.new(0, sz, 0, sz)
                b.Position = UDim2.new(math.random() * 0.8 + 0.1, 0, math.random() * 0.8 + 0.1, 0)
                b.BackgroundColor3 = animColor
                b.BackgroundTransparency = 0.35
                b.BorderSizePixel = 0
                b.ZIndex = 0
                b.Parent = container
                local cr = Instance.new("UICorner"); cr.CornerRadius = UDim.new(1, 0); cr.Parent = b

                local ballHueOff = (i / bCount) * 0.15
                balls[i] = {
                    frame = b,
                    vx = (math.random() - 0.5) * 0.3 * speed,
                    vy = (math.random() - 0.5) * 0.3 * speed,
                    size = sz,
                    hueOff = ballHueOff,
                }
            end

            local conn
            conn = RunService.RenderStepped:Connect(function(dt)
                if not container or not container.Parent then
                    pcall(function() conn:Disconnect() end)
                    return
                end
                local absW = container.AbsoluteSize.X
                local absH = container.AbsoluteSize.Y
                if absW < 1 or absH < 1 then return end


                local curH = h
                if autoColor then
                    local phase = (tick() * 0.04) % 1
                    local shift = math.sin(phase * math.pi * 2) * 0.08
                    local compShift = math.sin(phase * math.pi * 2 + math.pi) * 0.04
                    curH = (h + shift + compShift) % 1
                end

                for _, bl in ipairs(balls) do
                    local f = bl.frame
                    local px = f.Position.X.Scale + bl.vx * dt
                    local py = f.Position.Y.Scale + bl.vy * dt
                    local szX = bl.size / absW
                    local szY = bl.size / absH
                    if px - szX / 2 < 0 then bl.vx = math.abs(bl.vx); px = szX / 2
                    elseif px + szX / 2 > 1 then bl.vx = -math.abs(bl.vx); px = 1 - szX / 2 end
                    if py - szY / 2 < 0 then bl.vy = math.abs(bl.vy); py = szY / 2
                    elseif py + szY / 2 > 1 then bl.vy = -math.abs(bl.vy); py = 1 - szY / 2 end
                    f.Position = UDim2.new(px, 0, py, 0)

                    if autoColor then
                        f.BackgroundColor3 = Color3.fromHSV((curH + bl.hueOff) % 1, s, v0)
                    end
                end
            end)
            table.insert(_bgAnimConns, conn)
        end
    end

    for _, c in pairs(MainFrame_ref:GetChildren()) do
        if c:IsA("GuiObject") and c.ZIndex < 1 and not c.Name:find("^SPRB_") then
            c.ZIndex = 1
        end
    end
end

local function playUISound(soundId)
    if not soundId or soundId == "" then return end
    task.spawn(function()
        local s = Instance.new("Sound")
        s.SoundId = soundId
        s.Volume = UISoundConfig.volume
        s.Parent = SoundService
        s.Ended:Connect(function()
            s:Destroy()
        end)
        pcall(function() s:Play() end)
        task.delay(2, function()
            if s and s.Parent then s:Destroy() end
        end)
    end)
end

local function ensureFolder()
    if isfolder(FOLDER_PATH) then
        return
    end

    makefolder(FOLDER_PATH)
end

local _cachedSearchPaths = nil

local function scanAllFolders(maxDepth)
    local paths, seen = {}, {}
    local function scanRecursive(basePath, depth)
        if depth <= 0 then return end
        local ok, items = pcall(listfiles, basePath)
        if not ok or type(items) ~= "table" then return end
        for _, fp in ipairs(items) do
            if type(fp) == "string" and isfolder(fp) and not seen[fp] then
                seen[fp] = true
                paths[#paths + 1] = fp
                scanRecursive(fp, depth - 1)
            end
        end
    end
    scanRecursive(".", maxDepth or 2)
    return paths
end

local function getBuildSearchPaths()
    if _cachedSearchPaths then return _cachedSearchPaths end
    local paths, seen = {}, {}
    for _, path in ipairs(BUILD_SEARCH_PATHS) do
        if not seen[path] then
            paths[#paths + 1] = path
            seen[path] = true
        end
    end
    if not seen[FOLDER_PREFIX] then
        paths[#paths + 1] = FOLDER_PREFIX
    end

    local allFolders = scanAllFolders(2)
    for _, fp in ipairs(allFolders) do
        if not seen[fp] then
            paths[#paths + 1] = fp
            seen[fp] = true
        end
    end
    _cachedSearchPaths = paths
    return paths
end

local function invalidateSearchCache()
    _cachedSearchPaths = nil
end

local function loadSettings()
    if not isfile(SETTINGS_PATH) then return end
    local raw = ""
    local ok1, err1 = pcall(function() raw = readfile(SETTINGS_PATH) end)
    if not ok1 or type(raw) ~= "string" or #raw == 0 then
        pcall(function() delfile(SETTINGS_PATH) end)
        return
    end
    local data
    local ok2, err2 = pcall(function() data = HttpService:JSONDecode(raw) end)
    if not ok2 or type(data) ~= "table" then
        pcall(function() delfile(SETTINGS_PATH) end)
        return
    end

    for k, v in pairs(data) do
        pcall(function()
            if Settings[k] == nil then return end
            if typeof(Settings[k]) == "Color3" then
                if type(v) == "table" then
                    local r = tonumber(v.R or v.r or v[1]) or 0
                    local g = tonumber(v.G or v.g or v[2]) or 0
                    local b = tonumber(v.B or v.b or v[3]) or 0
                    if r > 1 or g > 1 or b > 1 then
                        r = r / 255; g = g / 255; b = b / 255
                    end
                    r = math.clamp(r, 0, 1)
                    g = math.clamp(g, 0, 1)
                    b = math.clamp(b, 0, 1)
                    Settings[k] = Color3.new(r, g, b)
                end
            elseif type(v) ~= "table" then
                local tv = type(v)
                local sk = type(Settings[k])
                if (sk == "number" and tv == "number") or (sk == "string" and tv == "string") or (sk == "boolean" and tv == "boolean") then
                    Settings[k] = v
                end
            elseif type(Settings[k]) == "table" then
                Settings[k] = v
            end
        end)
    end
    Settings.blockReplacements = {}

    for k, v in pairs(Settings) do
        pcall(function()
            if typeof(Settings_DEFAULTS[k]) == "Color3" and typeof(v) ~= "Color3" then
                Settings[k] = Settings_DEFAULTS[k]
            end
        end)
    end

    if Settings.bgMode == "image" then Settings.bgMode = "default" end
    if type(Settings.guiTransparency) == "number" then
        Settings.guiTransparency = math.clamp(Settings.guiTransparency, 0, 0.95)
    end
    if type(Settings.uiScale) == "number" then
        Settings.uiScale = math.clamp(Settings.uiScale, 0.4, 2.0)
    end
    if type(Settings.buildScale) == "number" then
        Settings.buildScale = math.clamp(Settings.buildScale, 0.1, 10)
    end
end

local _saveQueued = false

local function saveSettings()

    if _saveQueued then return end
    _saveQueued = true
    task.delay(0.5, function()
        _saveQueued = false

        local d = {}
        for k, v in pairs(Settings) do
            if k == "blockReplacements" then continue end
            if typeof(v) == "Color3" then
                local r = math.clamp(v.R, 0, 1)
                local g = math.clamp(v.G, 0, 1)
                local b = math.clamp(v.B, 0, 1)
                d[k] = {r, g, b}
            elseif type(v) == "table" then
                local safe = true
                local clean = {}
                for tk, tv in pairs(v) do
                    local tvt = type(tv)
                    if tvt == "string" or tvt == "number" or tvt == "boolean" then
                        clean[tk] = tv
                    else
                        safe = false; break
                    end
                end
                if safe then d[k] = clean end
            elseif type(v) == "string" or type(v) == "number" or type(v) == "boolean" then
                d[k] = v
            end
        end
        local ok, json = pcall(function() return HttpService:JSONEncode(d) end)
        if ok and type(json) == "string" and #json > 0 then
            pcall(function() writefile(SETTINGS_PATH, json) end)
        else
        end
    end)
end

local function loadCustomScripts()
    if not isfile(CUSTOM_SCRIPTS_PATH) then return {windows={}} end
    local ok, data = pcall(function() return HttpService:JSONDecode(readfile(CUSTOM_SCRIPTS_PATH)) end)
    return (ok and data and data.windows) and data or {windows={}}
end

local function saveCustomScripts(data)
    writefile(CUSTOM_SCRIPTS_PATH, HttpService:JSONEncode(data or {windows={}}))
end

local function getBlockID(blockName)
    local c = BlockData:FindFirstChild(blockName)
    return c and c.Value or 0
end

local function getRealBlockCount(blockName)
    pcall(function()
        local bg = LocalPlayer.PlayerGui:FindFirstChild("BuildGui")
        if bg then
            local inv = bg:FindFirstChild("InventoryFrame")
            if inv then
                local sf = inv:FindFirstChild("ScrollingFrame")
                if sf then
                    local bf = sf:FindFirstChild(blockName)
                    if bf then
                        local at = bf:FindFirstChild("AmountText")
                        if at and at.Text then
                            return tonumber(at.Text) or 0
                        end
                    end
                end
            end
        end
    end)
    return getBlockID(blockName)
end

local function getPlayerZone(player)
    for _, zone in pairs(Workspace:GetChildren()) do
        if zone:FindFirstChild("TeamColor") and zone.TeamColor.Value == player.TeamColor then
            return zone
        end
    end
end

local function getPlayerList()
    local list = {}
    for _, p in pairs(Players:GetPlayers()) do
        local d = p.Name
        if p.DisplayName ~= p.Name then d = p.DisplayName .. " (" .. p.Name .. ")" end
        if p == LocalPlayer then d = d .. " [ME]" end
        table.insert(list, {name=p.Name, display=d})
    end
    return list
end

local function cfStr(cf)
    return string.format("%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f,%.4f", cf:GetComponents())
end

local function parseNums(s)
    if type(s) ~= "string" then return {} end
    local r = {}
    for p in s:gmatch("[^,%s]+") do
        local n = tonumber(p)
        if n then table.insert(r, n) end
    end
    return r
end

local function strV3(s)
    if type(s) == "string" then
        local c = {}
        for v in s:gmatch("[^,]+") do table.insert(c, tonumber(v:match("^%s*(.-)%s*$")) or 0) end
        return #c >= 3 and Vector3.new(c[1],c[2],c[3]) or Vector3.new(0,0,0)
    elseif type(s) == "table" then

        local x = tonumber(s[1] or s.X) or 0
        local y = tonumber(s[2] or s.Y) or 0
        local z = tonumber(s[3] or s.Z) or 0
        return Vector3.new(x, y, z)
    end
    return Vector3.new(0,0,0)
end

local function v3Str(v) return string.format("%.4f,%.4f,%.4f", v.X, v.Y, v.Z) end

local function strCF(s)
    if type(s) ~= "string" then return nil end
    local c = {}
    for v in s:gmatch("[^,]+") do
        local n = tonumber(v:match("^%s*(.-)%s*$"))
        if n then table.insert(c, n) end
    end
    if #c >= 12 then return CFrame.new(table.unpack(c)) end
    return nil
end

local function colStr(c) return string.format("%.4f,%.4f,%.4f", c.R, c.G, c.B) end
local function strCol(s)
    if type(s) ~= "string" then return Color3.new(1,1,1) end
    local c = {}
    for v in s:gmatch("[^,]+") do table.insert(c, tonumber(v) or 1) end
    if #c >= 3 then
        local r,g,b = c[1],c[2],c[3]

        if r > 1 or g > 1 or b > 1 then
            r = r/255; g = g/255; b = b/255
        end
        return Color3.new(math.clamp(r,0,1), math.clamp(g,0,1), math.clamp(b,0,1))
    end
    return Color3.new(1,1,1)
end

local function getBlockCF(bi)

    if bi.CFrame then
        if type(bi.CFrame) == "string" then
            local cf = strCF(bi.CFrame)
            if cf then return cf end
        elseif type(bi.CFrame) == "table" and #bi.CFrame >= 12 then

            return CFrame.new(table.unpack(bi.CFrame))
        end
    end

    local posRaw = bi.Position or bi.position or bi.Pos or bi.pos
    local rotRaw = bi.Rotation or bi.rotation or bi.Rot or bi.rot
    if posRaw then
        local pos = strV3(posRaw)
        if rotRaw then
            local r = {}
            if type(rotRaw) == "string" then
                for v in rotRaw:gmatch("[^,]+") do
                    local n = tonumber(v:match("^%s*(.-)%s*$"))
                    if n then table.insert(r, math.rad(n)) end
                end
            elseif type(rotRaw) == "table" then
                for i = 1, 3 do
                    local v = rotRaw[i] or rotRaw[("XYZ"):sub(i,i)]
                    table.insert(r, math.rad(tonumber(v) or 0))
                end
            end
            if #r >= 3 then
                return CFrame.new(pos) * CFrame.Angles(r[1], r[2], r[3])
            end
        end
        return CFrame.new(pos)
    end

    return CFrame.new(0, 0, 0)
end

local function parseColorRGBA(rgba)
    if type(rgba) ~= "table" then return "1.000000, 1.000000, 1.000000" end
    local r = (tonumber(rgba[1]) or 255) / 255
    local g = (tonumber(rgba[2]) or 255) / 255
    local b = (tonumber(rgba[3]) or 255) / 255
    return string.format("%.6f, %.6f, %.6f", math.clamp(r, 0, 1), math.clamp(g, 0, 1), math.clamp(b, 0, 1))
end

local function geomGetAABB(shape)
    if type(shape) ~= "table" then return nil end
    local t = tonumber(shape.type) or -1
    local d = shape.data
    if type(d) ~= "table" then return nil end
    local function min(a, b) return (a < b) and a or b end
    local function max(a, b) return (a > b) and a or b end
    if (t == 0 or t == 2) and #d >= 4 then
        local x0, y0, x1, y1 = d[1], d[2], d[3], d[4]
        return {min(x0, x1), min(y0, y1), max(x0, x1), max(y0, y1)}
    elseif t == 1 and #d >= 5 then
        local x0, y0, x1, y1, ang = d[1], d[2], d[3], d[4], d[5]
        local cx = (x0 + x1) / 2
        local cy = (y0 + y1) / 2
        local rx = math.abs(x1 - x0) / 2
        local ry = math.abs(y1 - y0) / 2
        local rad = math.rad(ang)
        local hw = math.sqrt((rx * math.cos(rad)) ^ 2 + (ry * math.sin(rad)) ^ 2)
        local hh = math.sqrt((rx * math.sin(rad)) ^ 2 + (ry * math.cos(rad)) ^ 2)
        return {cx - hw, cy - hh, cx + hw, cy + hh}
    elseif t == 3 and #d >= 3 then
        local cx, cy, r = d[1], d[2], d[3]
        return {cx - r, cy - r, cx + r, cy + r}
    elseif t == 4 and #d >= 6 then
        local xs = {d[1], d[3], d[5]}
        local ys = {d[2], d[4], d[6]}
        local xMin, xMax = xs[1], xs[1]
        local yMin, yMax = ys[1], ys[1]
        for i = 2, 3 do
            xMin = min(xMin, xs[i]); xMax = max(xMax, xs[i])
            yMin = min(yMin, ys[i]); yMax = max(yMax, ys[i])
        end
        return {xMin, yMin, xMax, yMax}
    elseif t == 5 and #d >= 4 then
        local x0, y0, x1, y1 = d[1], d[2], d[3], d[4]
        local lw = tonumber(d[5]) or 1
        local half = lw / 2
        return {min(x0, x1) - half, min(y0, y1) - half, max(x0, x1) + half, max(y0, y1) + half}
    end
    return nil
end

local function geomAABBIntersects(a, b)
    return not (a[3] <= b[1] or b[3] <= a[1] or a[4] <= b[2] or b[4] <= a[2])
end

local function geomComputeZPositions(shapes, thickness)
    local step = thickness
    local placed = {}
    local results = {}
    for i, shape in ipairs(shapes) do
        local aabb = geomGetAABB(shape)
        if not aabb then
            results[i] = 0
        else
            local maxZ = -step
            for _, prev in ipairs(placed) do
                local prevAABB, prevZ = prev[1], prev[2]
                if geomAABBIntersects(aabb, prevAABB) then
                    if prevZ > maxZ then maxZ = prevZ end
                end
            end
            local z = maxZ + step
            results[i] = z
            placed[#placed + 1] = {aabb, z}
        end
    end
    return results
end

local function geomMakeBlock(cx, cy, zPos, w, h, thickness, rotZ, colorStr, transparency, blockId)




    return {
        ShowShadow = true,
        CanCollide = true,
        Color = colorStr,
        Anchored = true,
        BoolValues = {},
        Rotation = string.format("0.000, 0.000, %.3f", -rotZ),
        Transparency = transparency,
        Position = string.format("%.6f, %.6f, %.6f", cx, -cy, zPos + thickness / 2),
        ID = blockId,
        NumberValues = {},
        Size = string.format("%.6f, %.6f, %.6f", w, h, thickness),
    }
end

local function geomGetBounds(shapes)
    local minX, minY = math.huge, math.huge
    local maxX, maxY = -math.huge, -math.huge
    local function addPoint(x, y)
        x, y = tonumber(x), tonumber(y)
        if not x or not y then return end
        minX = math.min(minX, x)
        minY = math.min(minY, y)
        maxX = math.max(maxX, x)
        maxY = math.max(maxY, y)
    end

    for _, shape in ipairs(shapes) do
        if type(shape) == "table" and type(shape.data) == "table" then
            local stype = tonumber(shape.type) or -1
            local data = shape.data
            if (stype == 0 or stype == 1 or stype == 2) and #data >= 4 then
                addPoint(data[1], data[2])
                addPoint(data[3], data[4])
            elseif stype == 3 and #data >= 3 then
                local x, y, r = tonumber(data[1]), tonumber(data[2]), tonumber(data[3]) or 0
                if x and y then
                    addPoint(x - r, y - r)
                    addPoint(x + r, y + r)
                end
            elseif stype == 4 and #data >= 6 then
                addPoint(data[1], data[2])
                addPoint(data[3], data[4])
                addPoint(data[5], data[6])
            elseif stype == 5 and #data >= 4 then
                addPoint(data[1], data[2])
                addPoint(data[3], data[4])
            end
        end
    end

    if minX == math.huge then return nil end
    return minX, minY, maxX, maxY
end


local compactBlockToJSON
local writeBuildJSON

local function convertGeometrizeJsonToBlocks(jsonText, scale, thickness, material, targetWidth, targetLength)
    local decodeOk, rawData = pcall(function()
        return HttpService:JSONDecode(jsonText)
    end)
    if not decodeOk or type(rawData) ~= "table" then
        return nil, nil, "Invalid JSON: " .. tostring(rawData)
    end

    local shapes = rawData.shapes or rawData
    if type(shapes) ~= "table" or #shapes == 0 then
        return nil, nil, "No shapes found in JSON"
    end
    scale = tonumber(scale) or 0.035
    thickness = tonumber(thickness) or 0.001
    if scale <= 0 or thickness <= 0 then return nil, nil, "Scale/thickness must be > 0" end
    material = tostring(material or "PlasticBlock")
    targetWidth = tonumber(targetWidth) or 0
    targetLength = tonumber(targetLength) or 0

    local scaleX, scaleY = scale, scale
    if targetWidth > 0 or targetLength > 0 then
        local minX, minY, maxX, maxY = geomGetBounds(shapes)
        if minX then
            local rawW = math.max(0.001, maxX - minX)
            local rawH = math.max(0.001, maxY - minY)
            if targetWidth > 0 then scaleX = targetWidth / rawW end
            if targetLength > 0 then scaleY = targetLength / rawH end
        end
    end

    local zPositions = geomComputeZPositions(shapes, thickness)
    local blocks = {}
    local currentId = 1

    for i, shape in ipairs(shapes) do
        if type(shape) ~= "table" then continue end
        local stype = tonumber(shape.type) or -1
        local data = shape.data
        local color = shape.color
        if type(data) ~= "table" then continue end

        local transparency = 0
        local colorStr = parseColorRGBA(color)
        local layerZ = zPositions[i] or 0

        if stype == 1 and #data >= 5 then
            local x0, y0, x1, y1, angle = data[1], data[2], data[3], data[4], data[5]
            local cx = (x0 + x1) / 2 * scaleX
            local cy = (y0 + y1) / 2 * scaleY
            local w = math.abs(x1 - x0) * scaleX
            local h = math.abs(y1 - y0) * scaleY
            blocks[#blocks + 1] = geomMakeBlock(cx, cy, layerZ, w, h, thickness, tonumber(angle) or 0, colorStr, transparency, currentId)
            currentId = currentId + 1
        elseif (stype == 0 or stype == 2) and #data >= 4 then
            local x0, y0, x1, y1 = data[1], data[2], data[3], data[4]
            local cx = (x0 + x1) / 2 * scaleX
            local cy = (y0 + y1) / 2 * scaleY
            local w = math.abs(x1 - x0) * scaleX
            local h = math.abs(y1 - y0) * scaleY
            blocks[#blocks + 1] = geomMakeBlock(cx, cy, layerZ, w, h, thickness, 0, colorStr, transparency, currentId)
            currentId = currentId + 1
        elseif stype == 3 and #data >= 3 then
            local cxRaw, cyRaw, r = data[1], data[2], data[3]
            local w = r * 2 * scaleX
            local h = r * 2 * scaleY
            blocks[#blocks + 1] = geomMakeBlock(cxRaw * scaleX, cyRaw * scaleY, layerZ, w, h, thickness, 0, colorStr, transparency, currentId)
            currentId = currentId + 1
        elseif stype == 4 and #data >= 6 then
            local ptsX = {data[1], data[3], data[5]}
            local ptsY = {data[2], data[4], data[6]}
            local cx = (ptsX[1] + ptsX[2] + ptsX[3]) / 3 * scaleX
            local cy = (ptsY[1] + ptsY[2] + ptsY[3]) / 3 * scaleY
            local xMin, xMax = math.min(ptsX[1], ptsX[2], ptsX[3]), math.max(ptsX[1], ptsX[2], ptsX[3])
            local yMin, yMax = math.min(ptsY[1], ptsY[2], ptsY[3]), math.max(ptsY[1], ptsY[2], ptsY[3])
            local w = (xMax - xMin) * scaleX
            local h = (yMax - yMin) * scaleY
            blocks[#blocks + 1] = geomMakeBlock(cx, cy, layerZ, w, h, thickness, 0, colorStr, transparency, currentId)
            currentId = currentId + 1
        elseif stype == 5 and #data >= 4 then
            local x0, y0, x1, y1 = data[1], data[2], data[3], data[4]
            local lw = tonumber(data[5]) or 1
            local dx = (x1 - x0) * scaleX
            local dy = (y1 - y0) * scaleY
            local cx = (x0 + x1) / 2 * scaleX
            local cy = (y0 + y1) / 2 * scaleY
            local length = math.sqrt(dx ^ 2 + dy ^ 2)
            local angleZ = math.deg(math.atan2(-dy, dx))
            blocks[#blocks + 1] = geomMakeBlock(cx, cy, layerZ, length, lw * math.min(scaleX, scaleY), thickness, -angleZ, colorStr, transparency, currentId)
            currentId = currentId + 1
        end
    end

    if #blocks == 0 then
        return nil, nil, "Image produced no blocks"
    end


    local minY = math.huge
    for _, blk in ipairs(blocks) do
        if blk.Position then
            local nums = parseNums(blk.Position)
            if #nums >= 3 and nums[2] < minY then
                minY = nums[2]
            end
        end
    end
    if minY < 4 then
        local yShift = 4 - minY
        for _, blk in ipairs(blocks) do
            if blk.Position then
                local nums = parseNums(blk.Position)
                if #nums >= 3 then
                    nums[2] = nums[2] + yShift
                    blk.Position = string.format("%.6f, %.6f, %.6f", nums[1], nums[2], nums[3])
                end
            end
        end
    end

    return material, blocks, nil
end

local function convertGeometrizeJsonToBuild(jsonText, buildName, scale, thickness, material, targetWidth, targetLength)
    local outMaterial, blocks, err = convertGeometrizeJsonToBlocks(jsonText, scale, thickness, material, targetWidth, targetLength)
    if not outMaterial then
        return nil, err
    end
    local safeName = tostring(buildName or ""):gsub("^%s*(.-)%s*$", "%1")
    if safeName == "" then
        return nil, "Output name is empty"
    end
    ensureFolder()
    local outPath = FOLDER_PREFIX .. safeName .. ".Build"
    writeBuildJSON(outPath, outMaterial, blocks)
    return outPath, nil
end

local function trimStr(s)
    return tostring(s or ""):gsub("^%s*(.-)%s*$", "%1")
end

local function startsWithDrivePath(s)
    return s:match("^%a:[/\\]") ~= nil or s:sub(1, 2) == "\\\\"
end

local function isAbsolutePath(s)
    s = trimStr(s)
    return s:sub(1, 1) == "/" or startsWithDrivePath(s)
end

local function getFileStem(path)
    local name = tostring(path or ""):match("([^/\\]+)$") or tostring(path or "")
    return (name:gsub("%.[^%.]+$", ""))
end

local function joinPath(dir, name)
    if dir == "" then return name end
    local sep = dir:match("[/\\]$") and "" or "/"
    return dir .. sep .. name
end

local function resolveConverterPath(rawPath)
    local candidate = trimStr(rawPath)
    if candidate == "" then
        return nil, "Select a converter file"
    end
    candidate = candidate:gsub('^"(.*)"$', "%1")
    if isfile(candidate) then
        return candidate
    end
    local probes = {}
    if isAbsolutePath(candidate) then
        probes[#probes + 1] = candidate
    else
        probes[#probes + 1] = FOLDER_PREFIX .. candidate
        probes[#probes + 1] = candidate
    end
    for _, probe in ipairs(probes) do
        if isfile(probe) then
            return probe
        end
    end
    return nil, "File not found: " .. candidate
end

local function getParentDir(path)
    local cleaned = trimStr(path)
    if cleaned == "" then
        return nil
    end
    if isfolder(cleaned) then
        return cleaned
    end
    return cleaned:match("^(.*)[/\\][^/\\]+$")
end

local function rgbStr(r, g, b)
    if r > 1 or g > 1 or b > 1 then
        r, g, b = r / 255, g / 255, b / 255
    end
    return string.format("%.4f,%.4f,%.4f", math.clamp(r, 0, 1), math.clamp(g, 0, 1), math.clamp(b, 0, 1))
end

local function cfToAsuBlock(cf, sizeVec, colorStr, blockId, transparency)
    local rx, ry, rz = cf:ToEulerAnglesXYZ()
    local pos = cf.Position
    return {
        ShowShadow = true,
        CanCollide = true,
        Color = colorStr or rgbStr(1, 1, 1),
        Anchored = true,
        BoolValues = {},
        Rotation = string.format("%.6f, %.6f, %.6f", math.deg(rx), math.deg(ry), math.deg(rz)),
        Transparency = transparency or 0,
        Position = string.format("%.6f, %.6f, %.6f", pos.X, pos.Y, pos.Z),
        ID = blockId or 0,
        NumberValues = {},
        Size = string.format("%.6f, %.6f, %.6f", sizeVec.X, sizeVec.Y, sizeVec.Z),
    }
end

local function cloneJsonValue(value)
    if type(value) ~= "table" then
        return value
    end
    local out = {}
    for k, v in pairs(value) do
        out[k] = cloneJsonValue(v)
    end
    return out
end

local ASU_MAPPED_KEYS = {

    Position = true, position = true, Pos = true, pos = true,
    Rotation = true, rotation = true, Rot = true, rot = true,
    CFrame = true, cframe = true,

    Size = true, size = true,
    Color = true, color = true, Col = true, col = true,
    Transparency = true, transparency = true,
    ShowShadow = true, showShadow = true,
    Material = true, material = true,
    Text = true, text = true,

    Anchored = true, anchored = true,
    CanCollide = true, canCollide = true,

    BoolValues = true, boolValues = true,
    NumberValues = true, numberValues = true,
    BindTable = true, bindTable = true,
    ID = true, id = true,

    SecondaryPartPosition = true, secondaryPartPosition = true,
    SecondaryPartRotation = true, secondaryPartRotation = true,
    SecCFrame = true, secCFrame = true,
    Stiffness = true, stiffness = true,
    Damping = true, damping = true,
    TargetLength = true, targetLength = true,
    MaxLength = true, maxLength = true,
    MinLength = true, minLength = true,
    Length = true, length = true,
    AngleLimit = true, angleLimit = true,
    MatchRotation = true, matchRotation = true,
    ShowConstraint = true, showConstraint = true,
    ServoTorque = true, servoTorque = true,
    ServoSpeed = true, servoSpeed = true,
    BarLength = true, barLength = true,
    WheelTorque = true, wheelTorque = true,
    MaxForce = true, maxForce = true,
    Speed = true, speed = true,
    WaitDuration = true, waitDuration = true,
    Health = true, health = true,
    ExtendLength = true, extendLength = true,
    LastDirrection = true, lastDirrection = true,
}

local function collectAsuExtras(block)
    local extras = {}
    for k, v in pairs(block) do
        if not ASU_MAPPED_KEYS[k] then
            extras[k] = cloneJsonValue(v)
        end
    end
    if next(extras) then
        return extras
    end
    return nil
end

local function mergePropertyMaps(boolValues, numberValues, extras)
    local outBool = {}
    local outNum = {}

    if type(boolValues) == "table" then
        for k, v in pairs(boolValues) do
            outBool[k] = v
        end
    end
    if type(numberValues) == "table" then
        for k, v in pairs(numberValues) do
            outNum[k] = v
        end
    end
    if type(extras) == "table" then
        for k, v in pairs(extras) do
            local vt = type(v)
            if vt == "boolean" then
                if outBool[k] == nil then
                    outBool[k] = v
                end
            elseif vt == "number" then
                if outNum[k] == nil then
                    outNum[k] = v
                end
            elseif vt == "string" then
                local lower = string.lower(v)
                if outBool[k] == nil and (lower == "true" or lower == "false") then
                    outBool[k] = lower == "true"
                elseif outNum[k] == nil then
                    local n = tonumber(v)
                    if n ~= nil then
                        outNum[k] = n
                    end
                end
            end
        end
    end

    return outBool, outNum
end

local function currentExecutorName()
    local ok, name = pcall(function()
        if identifyexecutor then
            return identifyexecutor()
        end
        if getexecutorname then
            return getexecutorname()
        end
        return ""
    end)
    if ok and name then
        return tostring(name)
    end
    return ""
end

local function pointInsideAnyButton(container, x, y)
    for _, child in pairs(container:GetChildren()) do
        if child:IsA("TextButton") or child:IsA("ImageButton") then
            local pos = child.AbsolutePosition
            local size = child.AbsoluteSize
            if x >= pos.X and x <= pos.X + size.X and y >= pos.Y and y <= pos.Y + size.Y then
                return true
            end
        end
    end
    return false
end

compactBlockToJSON = function(b)

    local parts = {}
    parts[#parts+1] = '{"ShowShadow":'
    parts[#parts+1] = b.ShowShadow and "true" or "false"
    parts[#parts+1] = ',"CanCollide":'
    parts[#parts+1] = b.CanCollide and "true" or "false"
    parts[#parts+1] = ',"Color":"'
    parts[#parts+1] = tostring(b.Color or "1,1,1")
    parts[#parts+1] = '","Anchored":'
    parts[#parts+1] = b.Anchored and "true" or "false"
    parts[#parts+1] = ',"BoolValues":{}'
    parts[#parts+1] = ',"Rotation":"'
    parts[#parts+1] = tostring(b.Rotation or "0,0,0")
    parts[#parts+1] = '","Transparency":'
    parts[#parts+1] = tostring(b.Transparency or 0)
    parts[#parts+1] = ',"Position":"'
    parts[#parts+1] = tostring(b.Position or "0,0,0")
    parts[#parts+1] = '","ID":'
    parts[#parts+1] = tostring(b.ID or 0)
    parts[#parts+1] = ',"NumberValues":{}'
    parts[#parts+1] = ',"Size":"'
    parts[#parts+1] = tostring(b.Size or "1,1,1")
    parts[#parts+1] = '"}'
    return table.concat(parts)
end

writeBuildJSON = function(outPath, materialName, cleanBlocks)

    local parts = {}
    parts[#parts+1] = '[["'
    parts[#parts+1] = materialName
    parts[#parts+1] = '"],{"'
    parts[#parts+1] = materialName
    parts[#parts+1] = '":['
    for i, block in ipairs(cleanBlocks) do
        if i > 1 then parts[#parts+1] = ',' end
        parts[#parts+1] = compactBlockToJSON(block)
    end
    parts[#parts+1] = ']}]'
    writefile(outPath, table.concat(parts))
end

local function writeConvertedBuild(buildName, material, blocks)
    local safeName = trimStr(buildName)
    if safeName == "" then
        return nil, "Output name is empty"
    end
    if type(blocks) ~= "table" or #blocks == 0 then
        return nil, "Nothing to convert"
    end
    local cleanBlocks = {}
    local seen = {}
    for _, block in ipairs(blocks) do
        if type(block) == "table" then
            local key = table.concat({
                tostring(block.Position or ""),
                tostring(block.Rotation or ""),
                tostring(block.Size or ""),
                tostring(block.Color or ""),
                tostring(block.Transparency or 0),
                tostring(block.Material or material or ""),
            }, "|")
            if not seen[key] then
                seen[key] = true
                cleanBlocks[#cleanBlocks + 1] = block
            end
        end
    end
    if #cleanBlocks == 0 then
        return nil, "Nothing to convert"
    end
    ensureFolder()
    local outPath = FOLDER_PREFIX .. safeName .. ".Build"
    local materialName = tostring(material or "PlasticBlock")
    if materialName == "Auto" then
        local materials = {}
        local groups = {}
        for _, block in ipairs(cleanBlocks) do
            local blockMaterial = tostring(block.Material or "PlasticBlock")
            if not groups[blockMaterial] then
                groups[blockMaterial] = {}
                materials[#materials + 1] = blockMaterial
            end
            block.Material = nil
            groups[blockMaterial][#groups[blockMaterial] + 1] = block
        end
        table.sort(materials, function(a, b) return a:lower() < b:lower() end)

        local mParts = {}
        mParts[#mParts+1] = '['

        mParts[#mParts+1] = '['
        for mi, m in ipairs(materials) do
            if mi > 1 then mParts[#mParts+1] = ',' end
            mParts[#mParts+1] = '"' .. m .. '"'
        end
        mParts[#mParts+1] = '],'

        mParts[#mParts+1] = '{'
        for gi, m in ipairs(materials) do
            if gi > 1 then mParts[#mParts+1] = ',' end
            mParts[#mParts+1] = '"' .. m .. '":['
            local gBlocks = groups[m]
            for bi, block in ipairs(gBlocks) do
                if bi > 1 then mParts[#mParts+1] = ',' end
                mParts[#mParts+1] = compactBlockToJSON(block)
            end
            mParts[#mParts+1] = ']'
        end
        mParts[#mParts+1] = '}]'
        writefile(outPath, table.concat(mParts))
    else
        for _, block in ipairs(cleanBlocks) do block.Material = nil end
        writeBuildJSON(outPath, materialName, cleanBlocks)
    end
    return outPath, nil
end

local convertMinecraftSchematicToBlocks
local convertMinecraftSchematicToBuild
local convertObjToBlocks
local convertObjToBuild
do
local function bytesToString(bytes)
    local chunks = {}
    for i = 1, #bytes, 4096 do
        local chars = {}
        for j = i, math.min(i + 4095, #bytes) do
            chars[#chars + 1] = string.char(bytes[j])
        end
        chunks[#chunks + 1] = table.concat(chars)
    end
    return table.concat(chunks)
end

local function gzNoEof(value)
    if value == nil then
        error("Unexpected end of gzip stream", 0)
    end
    return value
end

local function gzHasBit(bits, mask)
    return bits % (mask + mask) >= mask
end

local function gzMakeOutputState(writer)
    return {
        outbs = writer,
        window = {},
        windowPos = 1,
    }
end

local function gzOutput(outState, byte)
    outState.outbs(byte)
    outState.window[outState.windowPos] = byte
    outState.windowPos = outState.windowPos % 32768 + 1
end

local function gzBitstreamFromString(data)
    local index = 1
    local bufByte = 0
    local bufBits = 0
    local stream = {}

    function stream:nbitsLeftInByte()
        return bufBits
    end

    function stream:read(nbits)
        nbits = nbits or 1
        while bufBits < nbits do
            if index > #data then
                return nil
            end
            local byte = string.byte(data, index)
            index = index + 1
            bufByte = bufByte + bit.lshift(byte, bufBits)
            bufBits = bufBits + 8
        end
        local bitsOut
        if nbits == 0 then
            bitsOut = 0
        elseif nbits == 32 then
            bitsOut = bufByte
            bufByte = 0
        else
            bitsOut = bit.band(bufByte, bit.rshift(0xffffffff, 32 - nbits))
            bufByte = bit.rshift(bufByte, nbits)
        end
        bufBits = bufBits - nbits
        return bitsOut
    end

    return stream
end

local function gzHuffmanTable(init, isFull)
    local entries = {}
    if isFull then
        for value, nbits in pairs(init) do
            if nbits and nbits ~= 0 then
                entries[#entries + 1] = {value = value, nbits = nbits}
            end
        end
    else
        for i = 1, #init - 2, 2 do
            local firstVal, nbits, nextVal = init[i], init[i + 1], init[i + 2]
            if nbits and nbits ~= 0 then
                for value = firstVal, nextVal - 1 do
                    entries[#entries + 1] = {value = value, nbits = nbits}
                end
            end
        end
    end
    table.sort(entries, function(a, b)
        return a.nbits == b.nbits and a.value < b.value or a.nbits < b.nbits
    end)

    local code = 1
    local currentBits = 0
    for _, entry in ipairs(entries) do
        if entry.nbits ~= currentBits then
            code = code * 2^(entry.nbits - currentBits)
            currentBits = entry.nbits
        end
        entry.code = code
        code = code + 1
    end

    local minBits = math.huge
    local lookup = {}
    for _, entry in ipairs(entries) do
        minBits = math.min(minBits, entry.nbits)
        lookup[entry.code] = entry.value
    end
    if minBits == math.huge then
        error("Invalid Huffman table", 0)
    end

    local firstCodeCache = {}
    local function msb(bitsValue, nbits)
        local result = 0
        for _ = 1, nbits do
            result = bit.lshift(result, 1) + bit.band(bitsValue, 1)
            bitsValue = bit.rshift(bitsValue, 1)
        end
        return result
    end

    local function firstCode(bitsValue)
        local cached = firstCodeCache[bitsValue]
        if cached == nil then
            cached = 2^minBits + msb(bitsValue, minBits)
            firstCodeCache[bitsValue] = cached
        end
        return cached
    end

    local tableObj = {}
    function tableObj:read(stream)
        local curCode = 1
        local nbits = 0
        while true do
            if nbits == 0 then
                curCode = firstCode(gzNoEof(stream:read(minBits)))
                nbits = minBits
            else
                local nextBit = gzNoEof(stream:read(1))
                nbits = nbits + 1
                curCode = curCode * 2 + nextBit
            end
            local value = lookup[curCode]
            if value ~= nil then
                return value
            end
        end
    end

    return tableObj
end

local function gzParseHeader(stream)
    local FLG_FHCRC = 2^1
    local FLG_FEXTRA = 2^2
    local FLG_FNAME = 2^3
    local FLG_FCOMMENT = 2^4

    local id1 = stream:read(8)
    local id2 = stream:read(8)
    if id1 ~= 31 or id2 ~= 139 then
        error("Not a gzip stream", 0)
    end
    local method = gzNoEof(stream:read(8))
    local flags = gzNoEof(stream:read(8))
    gzNoEof(stream:read(32))
    gzNoEof(stream:read(8))
    gzNoEof(stream:read(8))

    if method ~= 8 then
        error("Unsupported gzip method", 0)
    end

    if gzHasBit(flags, FLG_FEXTRA) then
        local xlen = gzNoEof(stream:read(16))
        for _ = 1, xlen do
            gzNoEof(stream:read(8))
        end
    end

    local function skipZString()
        repeat
            local byte = gzNoEof(stream:read(8))
        until byte == 0
    end

    if gzHasBit(flags, FLG_FNAME) then
        skipZString()
    end
    if gzHasBit(flags, FLG_FCOMMENT) then
        skipZString()
    end
    if gzHasBit(flags, FLG_FHCRC) then
        gzNoEof(stream:read(16))
    end
end

local function gzParseHuffmanTables(stream)
    local hlit = stream:read(5)
    local hdist = stream:read(5)
    local hclen = gzNoEof(stream:read(4))

    local codeLenInit = {}
    local codeLenOrder = {16, 17, 18, 0, 8, 7, 9, 6, 10, 5, 11, 4, 12, 3, 13, 2, 14, 1, 15}
    for i = 1, hclen + 4 do
        codeLenInit[codeLenOrder[i]] = stream:read(3)
    end
    local codeLenTable = gzHuffmanTable(codeLenInit, true)

    local function decode(count)
        local init = {}
        local nbits = 0
        local value = 0
        while value < count do
            local codeLen = codeLenTable:read(stream)
            local repeatCount
            if codeLen <= 15 then
                repeatCount = 1
                nbits = codeLen
            elseif codeLen == 16 then
                repeatCount = 3 + gzNoEof(stream:read(2))
            elseif codeLen == 17 then
                repeatCount = 3 + gzNoEof(stream:read(3))
                nbits = 0
            elseif codeLen == 18 then
                repeatCount = 11 + gzNoEof(stream:read(7))
                nbits = 0
            else
                error("Broken dynamic Huffman table", 0)
            end
            for _ = 1, repeatCount do
                init[value] = nbits
                value = value + 1
            end
        end
        return gzHuffmanTable(init, true)
    end

    return decode(hlit + 257), decode(hdist + 1)
end

local gzLenBase, gzLenExtra, gzDistBase, gzDistExtra
local function gzParseCompressedItem(stream, outState, litTable, distTable)
    local value = litTable:read(stream)
    if value < 256 then
        gzOutput(outState, value)
        return false
    end
    if value == 256 then
        return true
    end

    if not gzLenBase then
        gzLenBase = {[257] = 3}
        local step = 1
        for i = 258, 285, 4 do
            for j = i, i + 3 do
                gzLenBase[j] = gzLenBase[j - 1] + step
            end
            if i ~= 258 then
                step = step * 2
            end
        end
        gzLenBase[285] = 258
    end
    if not gzLenExtra then
        gzLenExtra = {}
        for i = 257, 285 do
            local delta = math.max(i - 261, 0)
            gzLenExtra[i] = bit.rshift(delta, 2)
        end
        gzLenExtra[285] = 0
    end

    local runLength = gzLenBase[value] + (stream:read(gzLenExtra[value]) or 0)

    if not gzDistBase then
        gzDistBase = {[0] = 1}
        local step = 1
        for i = 1, 29, 2 do
            for j = i, i + 1 do
                gzDistBase[j] = gzDistBase[j - 1] + step
            end
            if i ~= 1 then
                step = step * 2
            end
        end
    end
    if not gzDistExtra then
        gzDistExtra = {}
        for i = 0, 29 do
            local delta = math.max(i - 2, 0)
            gzDistExtra[i] = bit.rshift(delta, 1)
        end
    end

    local distValue = distTable:read(stream)
    local distance = gzDistBase[distValue] + (stream:read(gzDistExtra[distValue]) or 0)
    for _ = 1, runLength do
        local pos = (outState.windowPos - 1 - distance) % 32768 + 1
        local prev = outState.window[pos]
        if prev == nil then
            error("Invalid distance in gzip stream", 0)
        end
        gzOutput(outState, prev)
    end
    return false
end

local function gzParseBlock(stream, outState)
    local isFinal = stream:read(1)
    local blockType = stream:read(2)
    if isFinal == nil or blockType == nil then
        error("Unexpected end of deflate stream", 0)
    end

    if blockType == 0 then
        stream:read(stream:nbitsLeftInByte())
        local length = gzNoEof(stream:read(16))
        local nlen = gzNoEof(stream:read(16))
        if bit.band(bit.bnot(length), 0xFFFF) ~= nlen then
            error("Stored block length mismatch", 0)
        end
        for _ = 1, length do
            gzOutput(outState, gzNoEof(stream:read(8)))
        end
    elseif blockType == 1 or blockType == 2 then
        local litTable, distTable
        if blockType == 2 then
            litTable, distTable = gzParseHuffmanTables(stream)
        else
            litTable = gzHuffmanTable({0, 8, 144, 9, 256, 7, 280, 8, 288, nil}, false)
            distTable = gzHuffmanTable({0, 5, 32, nil}, false)
        end
        repeat
        until gzParseCompressedItem(stream, outState, litTable, distTable)
    else
        error("Unsupported deflate block type", 0)
    end

    return isFinal ~= 0
end

local function gunzipString(data)
    if type(data) ~= "string" or #data < 18 then
        return nil, "Gzip file is too small"
    end
    if string.byte(data, 1) ~= 0x1F or string.byte(data, 2) ~= 0x8B then
        return nil, "Not a gzip stream"
    end
    local outputBytes = {}
    local ok, err = pcall(function()
        local stream = gzBitstreamFromString(data)
        gzParseHeader(stream)
        local outState = gzMakeOutputState(function(byte)
            outputBytes[#outputBytes + 1] = byte
        end)
        repeat
        until gzParseBlock(stream, outState)
    end)
    if not ok then
        return nil, tostring(err)
    end
    return bytesToString(outputBytes), nil
end

local function decodeMaybeGzip(data)
    if type(data) ~= "string" then
        return nil, "Expected binary string"
    end
    if #data >= 2 and string.byte(data, 1) == 0x1F and string.byte(data, 2) == 0x8B then
        return gunzipString(data)
    end
    return data, nil
end

local function makeNbtReader(data)
    local r = {data = data, pos = 1}

    function r:u8()
        local b = string.byte(self.data, self.pos)
        self.pos = self.pos + 1
        return b or 0
    end

    function r:s8()
        local v = self:u8()
        return v >= 128 and (v - 256) or v
    end

    function r:u16()
        local b1 = self:u8()
        local b2 = self:u8()
        return b1 * 256 + b2
    end

    function r:s16()
        local v = self:u16()
        return v >= 32768 and (v - 65536) or v
    end

    function r:u32()
        local b1 = self:u8()
        local b2 = self:u8()
        local b3 = self:u8()
        local b4 = self:u8()
        return (((b1 * 256) + b2) * 256 + b3) * 256 + b4
    end

    function r:s32()
        local v = self:u32()
        return v >= 2147483648 and (v - 4294967296) or v
    end

    function r:float32()
        local v = self:u32()
        if v == 0 then return 0 end
        local sign = bit.band(v, 0x80000000) ~= 0 and -1 or 1
        local exp = bit.band(bit.rshift(v, 23), 0xFF)
        local mant = bit.band(v, 0x7FFFFF)
        if exp == 0 then
            return sign * (mant / 8388608) * 2^-126
        end
        if exp == 255 then
            return sign * math.huge
        end
        return sign * (1 + mant / 8388608) * 2^(exp - 127)
    end

    function r:float64()
        local hi = self:u32()
        local lo = self:u32()
        local sign = bit.band(hi, 0x80000000) ~= 0 and -1 or 1
        local exp = bit.band(bit.rshift(hi, 20), 0x7FF)
        local mantHi = bit.band(hi, 0xFFFFF)
        local mant = mantHi * 4294967296 + lo
        if exp == 0 then
            return sign * (mant / 4503599627370496) * 2^-1022
        end
        if exp == 2047 then
            return sign * math.huge
        end
        return sign * (1 + mant / 4503599627370496) * 2^(exp - 1023)
    end

    function r:str()
        local len = self:u16()
        local s = self.data:sub(self.pos, self.pos + len - 1)
        self.pos = self.pos + len
        return s
    end

    return r
end

local function parseNbtPayload(reader, tagType)
    if tagType == 1 then
        return reader:s8()
    elseif tagType == 2 then
        return reader:s16()
    elseif tagType == 3 then
        return reader:s32()
    elseif tagType == 4 then
        local hi = reader:u32()
        local lo = reader:u32()
        return {hi = hi, lo = lo}
    elseif tagType == 5 then
        return reader:float32()
    elseif tagType == 6 then
        return reader:float64()
    elseif tagType == 7 then
        local len = reader:s32()
        local arr = {}
        for i = 1, len do
            arr[i] = reader:u8()
        end
        return arr
    elseif tagType == 8 then
        return reader:str()
    elseif tagType == 9 then
        local childType = reader:u8()
        local len = reader:s32()
        local arr = {}
        for i = 1, len do
            arr[i] = parseNbtPayload(reader, childType)
        end
        return arr
    elseif tagType == 10 then
        local out = {}
        while true do
            local childType = reader:u8()
            if childType == 0 then
                break
            end
            local name = reader:str()
            out[name] = parseNbtPayload(reader, childType)
        end
        return out
    elseif tagType == 11 then
        local len = reader:s32()
        local arr = {}
        for i = 1, len do
            arr[i] = reader:s32()
        end
        return arr
    elseif tagType == 12 then
        local len = reader:s32()
        local arr = {}
        for i = 1, len do
            arr[i] = {hi = reader:u32(), lo = reader:u32()}
        end
        return arr
    end
    return nil
end

local function parseNbt(data)
    local reader = makeNbtReader(data)
    local rootType = reader:u8()
    if rootType ~= 10 then
        return nil, "NBT root is not a compound"
    end
    local _ = reader:str()
    return parseNbtPayload(reader, 10), nil
end

local MC_DYE_COLORS = {
    white = rgbStr(249, 255, 254),
    orange = rgbStr(249, 128, 29),
    magenta = rgbStr(199, 78, 189),
    light_blue = rgbStr(58, 179, 218),
    yellow = rgbStr(254, 216, 61),
    lime = rgbStr(128, 199, 31),
    pink = rgbStr(243, 139, 170),
    gray = rgbStr(71, 79, 82),
    light_gray = rgbStr(157, 157, 151),
    cyan = rgbStr(22, 156, 156),
    purple = rgbStr(137, 50, 184),
    blue = rgbStr(60, 68, 170),
    brown = rgbStr(131, 84, 50),
    green = rgbStr(94, 124, 22),
    red = rgbStr(176, 46, 38),
    black = rgbStr(29, 29, 33),
}

local DYE_ORDER = {
    "light_blue", "light_gray", "magenta", "orange", "yellow", "purple", "brown",
    "white", "black", "gray", "lime", "cyan", "blue", "green", "pink", "red"
}

local function mcBlockStateToColor(blockState)
    local state = tostring(blockState or ""):lower()
    local base = state:match("^[^%[]+") or state
    if base == "" then return rgbStr(190, 190, 190) end
    if base == "minecraft:air" or base:find("cave_air", 1, true) or base:find("void_air", 1, true) then
        return nil
    end
    for _, dye in ipairs(DYE_ORDER) do
        if base:find(dye, 1, true) then
            return MC_DYE_COLORS[dye]
        end
    end
    if base:find("water", 1, true) or base:find("ice", 1, true) then return rgbStr(64, 120, 255) end
    if base:find("lava", 1, true) or base:find("magma", 1, true) then return rgbStr(255, 111, 0) end
    if base:find("grass", 1, true) or base:find("fern", 1, true) or base:find("leaf", 1, true) or base:find("vine", 1, true)
        or base:find("moss", 1, true) or base:find("bush", 1, true) or base:find("crop", 1, true)
        or base:find("beetroot", 1, true) or base:find("bamboo", 1, true) then
        return rgbStr(88, 148, 74)
    end
    if base:find("sand", 1, true) or base:find("end_stone", 1, true) then return rgbStr(219, 211, 160) end
    if base:find("dirt", 1, true) or base:find("mud", 1, true) or base:find("farmland", 1, true) or base:find("root", 1, true) then
        return rgbStr(134, 96, 67)
    end
    if base:find("log", 1, true) or base:find("plank", 1, true) or base:find("wood", 1, true) or base:find("slab", 1, true)
        or base:find("fence", 1, true) or base:find("trapdoor", 1, true) or base:find("ladder", 1, true)
        or base:find("campfire", 1, true) or base:find("barrel", 1, true) then
        return rgbStr(151, 109, 77)
    end
    if base:find("brick", 1, true) or base:find("nether", 1, true) or base:find("terracotta", 1, true) then
        return rgbStr(153, 84, 72)
    end
    if base:find("glass", 1, true) then return rgbStr(190, 220, 235) end
    if base:find("gold", 1, true) then return rgbStr(249, 236, 78) end
    if base:find("diamond", 1, true) or base:find("prismarine", 1, true) then return rgbStr(92, 219, 213) end
    if base:find("emerald", 1, true) then return rgbStr(72, 186, 88) end
    if base:find("copper", 1, true) then return rgbStr(193, 108, 74) end
    if base:find("quartz", 1, true) or base:find("snow", 1, true) then return rgbStr(238, 238, 238) end
    if base:find("obsidian", 1, true) or base:find("deepslate", 1, true) or base:find("blackstone", 1, true) then
        return rgbStr(55, 52, 66)
    end
    if base:find("stone", 1, true) or base:find("cobble", 1, true) or base:find("andesite", 1, true)
        or base:find("diorite", 1, true) or base:find("granite", 1, true) or base:find("ore", 1, true)
        or base:find("iron", 1, true) then
        return rgbStr(137, 137, 137)
    end
    return rgbStr(185, 185, 185)
end

local LEGACY_WOOL_BY_DATA = {
    [0] = "minecraft:white_wool", [1] = "minecraft:orange_wool", [2] = "minecraft:magenta_wool", [3] = "minecraft:light_blue_wool",
    [4] = "minecraft:yellow_wool", [5] = "minecraft:lime_wool", [6] = "minecraft:pink_wool", [7] = "minecraft:gray_wool",
    [8] = "minecraft:light_gray_wool", [9] = "minecraft:cyan_wool", [10] = "minecraft:purple_wool", [11] = "minecraft:blue_wool",
    [12] = "minecraft:brown_wool", [13] = "minecraft:green_wool", [14] = "minecraft:red_wool", [15] = "minecraft:black_wool",
}

local LEGACY_GLASS_BY_DATA = {
    [0] = "minecraft:white_stained_glass", [1] = "minecraft:orange_stained_glass", [2] = "minecraft:magenta_stained_glass", [3] = "minecraft:light_blue_stained_glass",
    [4] = "minecraft:yellow_stained_glass", [5] = "minecraft:lime_stained_glass", [6] = "minecraft:pink_stained_glass", [7] = "minecraft:gray_stained_glass",
    [8] = "minecraft:light_gray_stained_glass", [9] = "minecraft:cyan_stained_glass", [10] = "minecraft:purple_stained_glass", [11] = "minecraft:blue_stained_glass",
    [12] = "minecraft:brown_stained_glass", [13] = "minecraft:green_stained_glass", [14] = "minecraft:red_stained_glass", [15] = "minecraft:black_stained_glass",
}

local LEGACY_CONCRETE_BY_DATA = {
    [0] = "minecraft:white_concrete", [1] = "minecraft:orange_concrete", [2] = "minecraft:magenta_concrete", [3] = "minecraft:light_blue_concrete",
    [4] = "minecraft:yellow_concrete", [5] = "minecraft:lime_concrete", [6] = "minecraft:pink_concrete", [7] = "minecraft:gray_concrete",
    [8] = "minecraft:light_gray_concrete", [9] = "minecraft:cyan_concrete", [10] = "minecraft:purple_concrete", [11] = "minecraft:blue_concrete",
    [12] = "minecraft:brown_concrete", [13] = "minecraft:green_concrete", [14] = "minecraft:red_concrete", [15] = "minecraft:black_concrete",
}

local function legacyIdToState(blockId, dataValue)
    local dv = tonumber(dataValue) or 0
    local simple = {
        [0] = "minecraft:air",
        [1] = "minecraft:stone",
        [2] = "minecraft:grass_block",
        [3] = "minecraft:dirt",
        [4] = "minecraft:cobblestone",
        [12] = "minecraft:sand",
        [13] = "minecraft:gravel",
        [17] = "minecraft:oak_log",
        [18] = "minecraft:oak_leaves",
        [20] = "minecraft:glass",
        [24] = "minecraft:sandstone",
        [41] = "minecraft:gold_block",
        [42] = "minecraft:iron_block",
        [45] = "minecraft:bricks",
        [48] = "minecraft:mossy_cobblestone",
        [49] = "minecraft:obsidian",
        [57] = "minecraft:diamond_block",
        [79] = "minecraft:ice",
        [80] = "minecraft:snow_block",
        [82] = "minecraft:clay",
        [87] = "minecraft:netherrack",
        [89] = "minecraft:glowstone",
        [98] = "minecraft:stone_bricks",
        [112] = "minecraft:nether_bricks",
        [121] = "minecraft:end_stone",
        [133] = "minecraft:emerald_block",
        [155] = "minecraft:quartz_block",
        [159] = "minecraft:orange_terracotta",
        [168] = "minecraft:prismarine",
        [172] = "minecraft:terracotta",
        [173] = "minecraft:coal_block",
        [174] = "minecraft:packed_ice",
        [179] = "minecraft:red_sandstone",
    }
    if blockId == 5 then
        local kinds = {"oak_planks", "spruce_planks", "birch_planks", "jungle_planks", "acacia_planks", "dark_oak_planks"}
        return "minecraft:" .. (kinds[(dv % #kinds) + 1] or "oak_planks")
    end
    if blockId == 35 then return LEGACY_WOOL_BY_DATA[dv] or "minecraft:white_wool" end
    if blockId == 95 or blockId == 160 then return LEGACY_GLASS_BY_DATA[dv] or "minecraft:white_stained_glass" end
    if blockId == 251 or blockId == 252 then return LEGACY_CONCRETE_BY_DATA[dv] or "minecraft:white_concrete" end
    return simple[blockId] or "minecraft:stone"
end

local function greedyMergeVoxels(voxels)
    if #voxels == 0 then return {} end
    table.sort(voxels, function(a, b)
        if a.y ~= b.y then return a.y < b.y end
        if a.z ~= b.z then return a.z < b.z end
        return a.x < b.x
    end)

    local cellMap = {}
    local used = {}
    for _, v in ipairs(voxels) do
        cellMap[v.x .. "," .. v.y .. "," .. v.z] = v
    end

    local function sameColor(x, y, z, color)
        local key = x .. "," .. y .. "," .. z
        local v = cellMap[key]
        return v and not used[key] and v.color == color
    end

    local boxes = {}
    for _, v in ipairs(voxels) do
        local key = v.x .. "," .. v.y .. "," .. v.z
        if not used[key] then
            local color = v.color
            local x2, y2, z2 = v.x, v.y, v.z
            while sameColor(x2 + 1, v.y, v.z, color) do
                x2 = x2 + 1
            end
            local canGrowY = true
            while canGrowY do
                for x = v.x, x2 do
                    if not sameColor(x, y2 + 1, v.z, color) then
                        canGrowY = false
                        break
                    end
                end
                if canGrowY then
                    y2 = y2 + 1
                end
            end
            local canGrowZ = true
            while canGrowZ do
                for y = v.y, y2 do
                    for x = v.x, x2 do
                        if not sameColor(x, y, z2 + 1, color) then
                            canGrowZ = false
                            break
                        end
                    end
                    if not canGrowZ then break end
                end
                if canGrowZ then
                    z2 = z2 + 1
                end
            end
            for z = v.z, z2 do
                for y = v.y, y2 do
                    for x = v.x, x2 do
                        used[x .. "," .. y .. "," .. z] = true
                    end
                end
            end
            boxes[#boxes + 1] = {x1 = v.x, y1 = v.y, z1 = v.z, x2 = x2, y2 = y2, z2 = z2, color = color}
        end
    end
    return boxes
end

local function decodePaletteVarInts(bytes, expectedCount)
    local values = {}
    local value = 0
    local shift = 0
    for _, b in ipairs(bytes or {}) do
        value = value + bit.lshift(bit.band(b, 0x7F), shift)
        if bit.band(b, 0x80) == 0 then
            values[#values + 1] = value
            value = 0
            shift = 0
            if expectedCount and #values >= expectedCount then
                break
            end
        else
            shift = shift + 7
        end
    end
    return values
end

local function buildSchemVoxels(root)
    local width = tonumber(root.Width) or 0
    local height = tonumber(root.Height) or 0
    local length = tonumber(root.Length) or 0
    local palette = root.Palette
    local rawData = root.BlockData or root.Blocks
    if width <= 0 or height <= 0 or length <= 0 then
        return nil, "Bad schematic size"
    end
    if type(palette) ~= "table" or type(rawData) ~= "table" then
        return nil, "Palette or block data missing"
    end
    local reversePalette = {}
    for state, id in pairs(palette) do
        reversePalette[tonumber(id) or 0] = state
    end
    local total = width * height * length
    local ids = decodePaletteVarInts(rawData, total)
    if #ids < total then
        return nil, "Schematic block data is truncated"
    end
    local voxels = {}
    local idx = 1
    for y = 0, height - 1 do
        for z = 0, length - 1 do
            for x = 0, width - 1 do
                local state = reversePalette[ids[idx] or 0] or "minecraft:air"
                idx = idx + 1
                local color = mcBlockStateToColor(state)
                if color then
                    voxels[#voxels + 1] = {x = x, y = y, z = z, color = color}
                end
            end
        end
    end
    return {width = width, height = height, length = length, voxels = voxels}
end

local function buildLegacySchematicVoxels(root)
    local width = tonumber(root.Width) or 0
    local height = tonumber(root.Height) or 0
    local length = tonumber(root.Length) or 0
    local blocks = root.Blocks
    local data = root.Data
    local addBlocks = root.AddBlocks
    if width <= 0 or height <= 0 or length <= 0 then
        return nil, "Bad legacy schematic size"
    end
    if type(blocks) ~= "table" or type(data) ~= "table" then
        return nil, "Legacy block arrays missing"
    end
    local total = width * height * length
    local voxels = {}
    for i = 0, total - 1 do
        local baseId = blocks[i + 1] or 0
        local blockId = baseId
        if type(addBlocks) == "table" then
            local packed = addBlocks[math.floor(i / 2) + 1] or 0
            local high = (i % 2 == 0) and bit.band(packed, 0x0F) or bit.rshift(packed, 4)
            blockId = baseId + high * 256
        end
        local dv = data[i + 1] or 0
        local state = legacyIdToState(blockId, dv)
        local color = mcBlockStateToColor(state)
        if color then
            local x = i % width
            local z = math.floor(i / width) % length
            local y = math.floor(i / (width * length))
            voxels[#voxels + 1] = {x = x, y = y, z = z, color = color}
        end
    end
    return {width = width, height = height, length = length, voxels = voxels}
end

convertMinecraftSchematicToBlocks = function(filePath, scale, material)
    local raw = readfile(filePath)
    local nbtRaw, zipErr = decodeMaybeGzip(raw)
    if not nbtRaw then
        return nil, nil, zipErr
    end
    local root, nbtErr = parseNbt(nbtRaw)
    if not root then
        return nil, nil, nbtErr
    end
    if type(root.Schematic) == "table" then
        root = root.Schematic
    end
    scale = tonumber(scale) or 1
    material = tostring(material or "PlasticBlock")
    if scale <= 0 then
        return nil, nil, "Scale must be > 0"
    end

    local data, err
    if type(root.Palette) == "table" then
        data, err = buildSchemVoxels(root)
    else
        data, err = buildLegacySchematicVoxels(root)
    end
    if not data then
        return nil, nil, err
    end

    local boxes = greedyMergeVoxels(data.voxels)
    local blocks = {}
    for i, box in ipairs(boxes) do
        local sizeVec = Vector3.new(
            (box.x2 - box.x1 + 1) * scale,
            (box.y2 - box.y1 + 1) * scale,
            (box.z2 - box.z1 + 1) * scale
        )
        local center = Vector3.new(
            ((box.x1 + box.x2 + 1) / 2 - data.width / 2) * scale,
            ((box.y1 + box.y2 + 1) / 2) * scale,
            ((box.z1 + box.z2 + 1) / 2 - data.length / 2) * scale
        )
        blocks[#blocks + 1] = cfToAsuBlock(CFrame.new(center), sizeVec, box.color, i, 0)
    end
    if #blocks == 0 then
        return nil, nil, "Schematic produced no blocks"
    end
    return material, blocks, nil
end

convertMinecraftSchematicToBuild = function(filePath, buildName, scale, material)
    local outMaterial, blocks, err = convertMinecraftSchematicToBlocks(filePath, scale, material)
    if not outMaterial then
        return nil, err
    end
    return writeConvertedBuild(buildName, outMaterial, blocks)
end

local function parseObjData(text, filePath)
    local materialColors = {}
    local vertices = {}
    local faces = {}
    for line in text:gmatch("[^\r\n]+") do
        local tag, rest = line:match("^%s*(%S+)%s*(.-)%s*$")
        if tag == "v" then
            local nums = parseNums(rest)
            if #nums >= 3 then
                local entry = {pos = Vector3.new(nums[1], nums[2], nums[3])}
                if #nums >= 6 then
                    entry.color = {nums[4], nums[5], nums[6]}
                end
                vertices[#vertices + 1] = entry
            end
        elseif tag == "f" then
            local face = {}
            for token in rest:gmatch("%S+") do
                local idxToken = token:match("([^/]+)")
                local idx = tonumber(idxToken)
                if idx then
                    if idx < 0 then
                        idx = #vertices + idx + 1
                    end
                    if vertices[idx] then
                        face[#face + 1] = idx
                    end
                end
            end
            if #face >= 3 then
                faces[#faces + 1] = {indices = face}
            end
        end
    end
    return {vertices = vertices, faces = faces, materialColors = materialColors}
end

local function chooseFaceColor(face, vertices, faceMaterial, materialColors)
    if faceMaterial and materialColors[faceMaterial] then
        return materialColors[faceMaterial]
    end
    local rs, gs, bs, count = 0, 0, 0, 0
    for _, idx in ipairs(face) do
        local v = vertices[idx]
        if v and v.color then
            rs = rs + v.color[1]
            gs = gs + v.color[2]
            bs = bs + v.color[3]
            count = count + 1
        end
    end
    if count > 0 then
        return rgbStr(rs / count, gs / count, bs / count)
    end
    return rgbStr(220, 220, 220)
end

local function centerPoints(points)
    local minV = Vector3.new(math.huge, math.huge, math.huge)
    local maxV = Vector3.new(-math.huge, -math.huge, -math.huge)
    for _, p in ipairs(points) do
        minV = Vector3.new(math.min(minV.X, p.X), math.min(minV.Y, p.Y), math.min(minV.Z, p.Z))
        maxV = Vector3.new(math.max(maxV.X, p.X), math.max(maxV.Y, p.Y), math.max(maxV.Z, p.Z))
    end
    local center = (minV + maxV) * 0.5
    local shifted = {}
    for i, p in ipairs(points) do
        shifted[i] = p - center
    end
    return shifted, center
end

local function makeFacePanel(points, scale, thickness, colorStr, blockId)
    if #points < 3 then
        return nil
    end
    local centered, baseCenter = centerPoints(points)
    local scaled = {}
    for i, p in ipairs(centered) do
        scaled[i] = Vector3.new(p.X * scale, p.Y * scale, p.Z * scale)
    end

    local normal = nil
    for i = 2, #scaled - 1 do
        local cross = (scaled[i] - scaled[1]):Cross(scaled[i + 1] - scaled[1])
        if cross.Magnitude > 1e-9 then
            normal = cross.Unit
            break
        end
    end
    if not normal then
        return nil
    end

    local longest = nil
    local bestDist = 0
    for i = 1, #scaled do
        for j = i + 1, #scaled do
            local d = (scaled[j] - scaled[i]).Magnitude
            if d > bestDist then
                bestDist = d
                longest = scaled[j] - scaled[i]
            end
        end
    end
    if not longest or longest.Magnitude < 1e-5 then
        return nil
    end

    local axisX = longest - normal * longest:Dot(normal)
    if axisX.Magnitude < 1e-5 then
        axisX = normal:Cross(Vector3.new(0, 1, 0))
        if axisX.Magnitude < 1e-5 then
            axisX = normal:Cross(Vector3.new(1, 0, 0))
        end
    end
    axisX = axisX.Unit
    local axisY = normal:Cross(axisX)
    if axisY.Magnitude < 1e-5 then
        return nil
    end
    axisY = axisY.Unit

    local minU, minV = math.huge, math.huge
    local maxU, maxV = -math.huge, -math.huge
    local center = Vector3.zero
    for _, p in ipairs(scaled) do
        center = center + p
    end
    center = center / #scaled
    for _, p in ipairs(scaled) do
        local rel = p - center
        local u = rel:Dot(axisX)
        local v = rel:Dot(axisY)
        minU = math.min(minU, u)
        maxU = math.max(maxU, u)
        minV = math.min(minV, v)
        maxV = math.max(maxV, v)
    end

    local width = math.max(0.02, maxU - minU)
    local height = math.max(0.02, maxV - minV)
    local faceCenter = (baseCenter * scale) + center + axisX * ((minU + maxU) * 0.5) + axisY * ((minV + maxV) * 0.5)
    local cf = CFrame.fromMatrix(faceCenter, axisX, axisY, normal)
    local rx, ry, rz = cf:ToEulerAnglesXYZ()
    rx = math.rad(snapAngle90(math.deg(rx)))
    ry = math.rad(snapAngle90(math.deg(ry)))
    rz = math.rad(snapAngle90(math.deg(rz)))
    cf = CFrame.new(faceCenter) * CFrame.Angles(rx, ry, rz)
    return cfToAsuBlock(cf, Vector3.new(width, height, thickness), colorStr, blockId, 0)
end

local function makeFacePanelCentered(points, thickness, colorStr, blockId)

    if #points < 3 then
        return nil
    end

    local normal = nil
    for i = 2, #points - 1 do
        local cross = (points[i] - points[1]):Cross(points[i + 1] - points[1])
        if cross.Magnitude > 1e-9 then
            normal = cross.Unit
            break
        end
    end
    if not normal then
        return nil
    end

    local longest = nil
    local bestDist = 0
    for i = 1, #points do
        for j = i + 1, #points do
            local d = (points[j] - points[i]).Magnitude
            if d > bestDist then
                bestDist = d
                longest = points[j] - points[i]
            end
        end
    end
    if not longest or longest.Magnitude < 1e-5 then
        return nil
    end

    local axisX = longest - normal * longest:Dot(normal)
    if axisX.Magnitude < 1e-5 then
        axisX = normal:Cross(Vector3.new(0, 1, 0))
        if axisX.Magnitude < 1e-5 then
            axisX = normal:Cross(Vector3.new(1, 0, 0))
        end
    end
    axisX = axisX.Unit
    local axisY = normal:Cross(axisX)
    if axisY.Magnitude < 1e-5 then
        return nil
    end
    axisY = axisY.Unit

    local minU, minV = math.huge, math.huge
    local maxU, maxV = -math.huge, -math.huge
    local centroid = Vector3.zero
    for _, p in ipairs(points) do
        centroid = centroid + p
    end
    centroid = centroid / #points
    for _, p in ipairs(points) do
        local rel = p - centroid
        local u = rel:Dot(axisX)
        local v = rel:Dot(axisY)
        minU = math.min(minU, u)
        maxU = math.max(maxU, u)
        minV = math.min(minV, v)
        maxV = math.max(maxV, v)
    end

    local width = math.max(0.02, maxU - minU)
    local height = math.max(0.02, maxV - minV)
    local faceCenter = centroid + axisX * ((minU + maxU) * 0.5) + axisY * ((minV + maxV) * 0.5)
    local cf = CFrame.fromMatrix(faceCenter, axisX, axisY, normal)
    local rx, ry, rz = cf:ToEulerAnglesXYZ()
    rx = math.rad(snapAngle90(math.deg(rx)))
    ry = math.rad(snapAngle90(math.deg(ry)))
    rz = math.rad(snapAngle90(math.deg(rz)))
    cf = CFrame.new(faceCenter) * CFrame.Angles(rx, ry, rz)
    return cfToAsuBlock(cf, Vector3.new(width, height, thickness), colorStr, blockId, 0)
end

local function makeEdgeBlock(p0, p1, scale, thickness, colorStr, blockId)
    local center = (p0 + p1) * 0.5 * scale
    local delta = (p1 - p0) * scale
    if delta.Magnitude < 1e-5 then
        return nil
    end
    local axisX = delta.Unit
    local up = math.abs(axisX:Dot(Vector3.new(0, 1, 0))) > 0.98 and Vector3.new(1, 0, 0) or Vector3.new(0, 1, 0)
    local axisZ = axisX:Cross(up)
    if axisZ.Magnitude < 1e-5 then
        axisZ = axisX:Cross(Vector3.new(0, 0, 1))
    end
    axisZ = axisZ.Unit
    local axisY = axisZ:Cross(axisX).Unit
    local cf = CFrame.fromMatrix(center, axisX, axisY, axisZ)
    return cfToAsuBlock(cf, Vector3.new(delta.Magnitude, thickness, thickness), colorStr, blockId, 0)
end

local function snapAngle90(deg)
    return math.floor(deg / 90 + 0.5) * 90
end

local function makeEdgeBlockFromRef(pA, pB, thickness, color, blockId, faceNormal)
    local delta = pB - pA
    local dist = delta.Magnitude
    if dist < 0.01 then return nil end
    local dir = delta.Unit

    local maxEdgeLen = 50
    if dist > maxEdgeLen then dist = maxEdgeLen end

    local shrink = math.min(dist * 0.06, thickness * 0.5)
    local effectiveDist = dist - shrink
    if effectiveDist < 0.01 then effectiveDist = dist end
    local shrinkHalf = (dist - effectiveDist) / 2
    local eA = pA + dir * shrinkHalf
    local eB = pB - dir * shrinkHalf
    local mid = (eA + eB) / 2

    local maxPos = 300
    mid = Vector3.new(
        math.clamp(mid.X, -maxPos, maxPos),
        math.clamp(mid.Y, -maxPos, maxPos),
        math.clamp(mid.Z, -maxPos, maxPos)
    )

    local absDir = Vector3.new(math.abs(dir.X), math.abs(dir.Y), math.abs(dir.Z))
    local isDiagonal = not (absDir.X > 0.95 or absDir.Y > 0.95 or absDir.Z > 0.95)

    local upHint
    if faceNormal and faceNormal.Magnitude > 0.0001 then
        local projN = faceNormal - dir * faceNormal:Dot(dir)
        if projN.Magnitude > 0.0001 then
            upHint = projN.Unit
        else
            upHint = Vector3.new(0, 1, 0)
        end
    else
        local absX, absY, absZ = math.abs(dir.X), math.abs(dir.Y), math.abs(dir.Z)
        if absY <= absX and absY <= absZ then
            upHint = Vector3.new(0, 1, 0)
        elseif absX <= absY and absX <= absZ then
            upHint = Vector3.new(1, 0, 0)
        else
            upHint = Vector3.new(0, 0, 1)
        end
    end
    local right = dir:Cross(upHint)
    if right.Magnitude < 0.0001 then
        local alternatives = {Vector3.new(0, 1, 0), Vector3.new(1, 0, 0), Vector3.new(0, 0, 1)}
        for _, alt in ipairs(alternatives) do
            right = dir:Cross(alt)
            if right.Magnitude > 0.0001 then
                upHint = alt
                break
            end
        end
        if right.Magnitude < 0.0001 then return nil end
    end
    right = right.Unit
    local upFixed = right:Cross(dir).Unit

    local cf = CFrame.fromMatrix(mid, right, upFixed, dir)

    if not isDiagonal then
        local rx, ry, rz = cf:ToEulerAnglesXYZ()
        rx = math.rad(snapAngle90(math.deg(rx)))
        ry = math.rad(snapAngle90(math.deg(ry)))
        rz = math.rad(snapAngle90(math.deg(rz)))
        cf = CFrame.new(mid) * CFrame.Angles(rx, ry, rz)
    end

    local effThick = math.clamp(thickness, 0.05, 4)
    local effLen = math.clamp(effectiveDist, 0.01, maxEdgeLen)

    local colorStr = nil
    if type(color) == "string" and color ~= "" then
        colorStr = color
    elseif typeof(color) == "Color3" then
        colorStr = string.format("%.3f,%.3f,%.3f", color.R, color.G, color.B)
    end
    return cfToAsuBlock(cf, Vector3.new(effThick, effThick, effLen), colorStr, blockId, 0)
end

local function createTriangleStrips(p1, p2, p3, stripSize, color, detailMul, blockIdStart)
    local parts = {}
    local v1 = p2 - p1
    local normal = v1:Cross(p3 - p1)
    if normal.Magnitude < 0.0001 then return parts end
    normal = normal.Unit

    local e1Len = (p2 - p1).Magnitude
    local e2Len = (p3 - p2).Magnitude
    local e3Len = (p3 - p1).Magnitude
    local longestDir
    if e1Len >= e2Len and e1Len >= e3Len then
        longestDir = (p2 - p1).Unit
    elseif e2Len >= e1Len and e2Len >= e3Len then
        longestDir = (p3 - p2).Unit
    else
        longestDir = (p3 - p1).Unit
    end

    local projDir = longestDir - normal * (normal.X * longestDir.X + normal.Y * longestDir.Y + normal.Z * longestDir.Z)
    if projDir.Magnitude < 0.0001 then return {} end
    projDir = projDir.Unit

    local localY = projDir:Cross(normal)
    if localY.Magnitude < 0.0001 then return {} end
    localY = localY.Unit

    local localX = localY:Cross(normal)
    if localX.Magnitude < 0.0001 then return {} end
    localX = localX.Unit

    local h1 = p1:Dot(localY)
    local h2 = p2:Dot(localY)
    local h3 = p3:Dot(localY)
    local minH = math.min(h1, h2, h3)
    local maxH = math.max(h1, h2, h3)
    local span = maxH - minH
    if span < 0.0001 then return {} end

    local numStrips = math.max(1, math.ceil(span / stripSize * detailMul))

    local edgeData = {
        {p1, p2, h1, h2},
        {p2, p3, h2, h3},
        {p3, p1, h3, h1}
    }

    for stripIdx = 0, numStrips - 1 do
        local stripH = minH + (stripIdx + 0.5) * span / numStrips
        local intersections = {}

        for _, edge in ipairs(edgeData) do
            local eA, eB, eHA, eHB = edge[1], edge[2], edge[3], edge[4]
            if stripH >= math.min(eHA, eHB) - 1e-5 and stripH <= math.max(eHA, eHB) + 1e-5 then
                local spanH = eHB - eHA
                local t = 0.5
                if math.abs(spanH) > 1e-5 then
                    t = math.clamp((stripH - eHA) / spanH, 0, 1)
                end
                local pt = eA + (eB - eA) * t
                intersections[#intersections + 1] = pt
            end
        end

        if #intersections >= 2 then

            local bestDist = 0
            local iP1 = intersections[1]
            local iP2 = intersections[2]
            for i = 1, #intersections do
                for j = i + 1, #intersections do
                    local d = (intersections[j] - intersections[i]).Magnitude
                    if d > bestDist then
                        bestDist = d
                        iP1 = intersections[i]
                        iP2 = intersections[j]
                    end
                end
            end

            if bestDist > 0.0001 then
                local stripCenter = (iP1 + iP2) / 2
                local stripDir = (iP2 - iP1).Unit

                local orientNormal = normal - stripDir * (stripDir.X * normal.X + stripDir.Y * normal.Y + stripDir.Z * normal.Z)
                if orientNormal.Magnitude < 0.0001 then
                    orientNormal = Vector3.new(0, 1, 0)
                end
                orientNormal = -orientNormal

                local cf = CFrame.fromMatrix(stripCenter, stripDir:Cross(orientNormal).Unit, orientNormal, stripDir)
                local colorStr = nil
                if type(color) == "string" and color ~= "" then
                    colorStr = color
                elseif typeof(color) == "Color3" then
                    colorStr = string.format("%.3f,%.3f,%.3f", color.R, color.G, color.B)
                end
                parts[#parts + 1] = cfToAsuBlock(cf, Vector3.new(stripSize, stripSize, bestDist), colorStr, blockIdStart + #parts, 0)
            end
        end
    end

    return parts
end

local function triangulateFace(faceIndices)
    local tris = {}
    if #faceIndices < 3 then
        return tris
    end
    for i = 2, #faceIndices - 1 do
        tris[#tris + 1] = {faceIndices[1], faceIndices[i], faceIndices[i + 1]}
    end
    return tris
end

local function addVoxelCell(cellMap, x, y, z, color)
    local key = x .. "," .. y .. "," .. z
    if not cellMap[key] then
        cellMap[key] = {x = x, y = y, z = z, color = color}
    end
end

local function rasterizeTriangleToVoxels(cellMap, p1, p2, p3, voxelSize, color)
    local e1 = p2 - p1
    local e2 = p3 - p1
    local e3 = p3 - p2
    local longest = math.max(e1.Magnitude, e2.Magnitude, e3.Magnitude)
    local steps = math.max(1, math.ceil(longest / math.max(voxelSize * 0.7, 0.05)))
    for i = 0, steps do
        for j = 0, steps - i do
            local a = i / steps
            local b = j / steps
            local c = 1 - a - b
            local p = p1 * c + p2 * a + p3 * b
            addVoxelCell(
                cellMap,
                math.floor(p.X / voxelSize + 0.5),
                math.floor(p.Y / voxelSize + 0.5),
                math.floor(p.Z / voxelSize + 0.5),
                color
            )
        end
    end
end

local function voxelCellsToBlocks(cellMap, voxelSize, blockIdStart)
    local voxels = {}
    local minX, minY, minZ = math.huge, math.huge, math.huge
    local maxX, maxY, maxZ = -math.huge, -math.huge, -math.huge
    for _, v in pairs(cellMap) do
        voxels[#voxels + 1] = v
        minX = math.min(minX, v.x)
        minY = math.min(minY, v.y)
        minZ = math.min(minZ, v.z)
        maxX = math.max(maxX, v.x)
        maxY = math.max(maxY, v.y)
        maxZ = math.max(maxZ, v.z)
    end
    local boxes = greedyMergeVoxels(voxels)
    local blocks = {}
    local cx = (minX + maxX + 1) * 0.5
    local cy = (minY + maxY + 1) * 0.5
    local cz = (minZ + maxZ + 1) * 0.5
    for i, box in ipairs(boxes) do
        local sizeVec = Vector3.new(
            (box.x2 - box.x1 + 1) * voxelSize,
            (box.y2 - box.y1 + 1) * voxelSize,
            (box.z2 - box.z1 + 1) * voxelSize
        )
        local center = Vector3.new(
            ((box.x1 + box.x2 + 1) * 0.5 - cx) * voxelSize,
            ((box.y1 + box.y2 + 1) * 0.5 - cy) * voxelSize,
            ((box.z1 + box.z2 + 1) * 0.5 - cz) * voxelSize
        )
        blocks[#blocks + 1] = cfToAsuBlock(CFrame.new(center), sizeVec, box.color, (blockIdStart or 1) + i - 1, 0)
    end
    return blocks
end

local function buildObjBlocks(parsedObj, scale, thickness, mode, colorOpts)
    mode = tostring(mode or "solid"):lower()
    local vertices = parsedObj.vertices or {}
    local faces = parsedObj.faces or {}
    local materialColors = parsedObj.materialColors or {}
    local blocks = {}
    local blockId = 1
    local stripSize = tonumber(thickness) or 0.5
    local opts = colorOpts or {}
    local colorMode = opts.colorMode or "mesh"
    local solidDetail = tonumber(opts.detail) or 1
    local totalFaces = #faces
    local faceIndex = 0
    local gradDir = opts.gradDir or "y_asc"

    local gradMin, gradMax

    local function resolveColor(meshColor, pos)
        if colorMode == "custom" then
            return string.format("%.3f,%.3f,%.3f", opts.customR or 0.4, opts.customG or 0.6, opts.customB or 1.0)
        elseif colorMode == "random" then
            return string.format("%.3f,%.3f,%.3f", math.random(), math.random(), math.random())
        elseif colorMode == "gradient" then
            local t = 0
            if gradDir == "index" then
                t = totalFaces > 1 and (faceIndex / (totalFaces - 1)) or 0
            elseif pos and gradMin and gradMax then
                if gradDir == "y_asc" then
                    t = (pos.Y - gradMin.Y) / math.max(0.0001, gradMax.Y - gradMin.Y)
                elseif gradDir == "y_desc" then
                    t = 1 - (pos.Y - gradMin.Y) / math.max(0.0001, gradMax.Y - gradMin.Y)
                elseif gradDir == "x_asc" then
                    t = (pos.X - gradMin.X) / math.max(0.0001, gradMax.X - gradMin.X)
                elseif gradDir == "x_desc" then
                    t = 1 - (pos.X - gradMin.X) / math.max(0.0001, gradMax.X - gradMin.X)
                elseif gradDir == "z_asc" then
                    t = (pos.Z - gradMin.Z) / math.max(0.0001, gradMax.Z - gradMin.Z)
                elseif gradDir == "z_desc" then
                    t = 1 - (pos.Z - gradMin.Z) / math.max(0.0001, gradMax.Z - gradMin.Z)
                elseif gradDir == "radial_out" then
                    local gc = (gradMin + gradMax) / 2
                    t = (pos - gc).Magnitude / math.max(0.0001, (gradMax - gradMin).Magnitude / 2)
                elseif gradDir == "radial_in" then
                    local gc = (gradMin + gradMax) / 2
                    t = 1 - (pos - gc).Magnitude / math.max(0.0001, (gradMax - gradMin).Magnitude / 2)
                else
                    t = totalFaces > 1 and (faceIndex / (totalFaces - 1)) or 0
                end
            else
                t = totalFaces > 1 and (faceIndex / (totalFaces - 1)) or 0
            end
            t = math.clamp(t, 0, 1)
            local g1R, g1G, g1B = opts.grad1R or 1, opts.grad1G or 0.3, opts.grad1B or 0.1
            local g2R, g2G, g2B = opts.grad2R or 0.1, opts.grad2G or 0.3, opts.grad2B or 1
            local cr = g1R + (g2R - g1R) * t
            local cg = g1G + (g2G - g1G) * t
            local cb = g1B + (g2B - g1B) * t
            return string.format("%.3f,%.3f,%.3f", cr, cg, cb)
        else
            return meshColor
        end
    end

    local verts = {}
    for _, v in ipairs(vertices) do
        verts[#verts + 1] = v.pos * scale
    end

    local faceIndices = {}
    for _, f in ipairs(faces) do
        faceIndices[#faceIndices + 1] = f.indices
    end

    if #faceIndices > 0 then
    end

    if #verts == 0 then return blocks end

    local bMin = verts[1]
    local bMax = verts[1]
    for _, v in ipairs(verts) do
        bMin = Vector3.new(math.min(bMin.X, v.X), math.min(bMin.Y, v.Y), math.min(bMin.Z, v.Z))
        bMax = Vector3.new(math.max(bMax.X, v.X), math.max(bMax.Y, v.Y), math.max(bMax.Z, v.Z))
    end
    local center = (bMin + bMax) / 2
    local localMin = bMin - center
    local localMax = bMax - center

    local yShift = -(localMin.Y) + 4
    center = center + Vector3.new(0, yShift, 0)
    localMin = bMin - center
    localMax = bMax - center

    gradMin = localMin
    gradMax = localMax

    local centeredVerts = {}
    for _, v in ipairs(verts) do
        centeredVerts[#centeredVerts + 1] = v - center
    end

    local edgeCount = {}
    local edgeOrder = {}
    local edgeMap = {}
    local edgeFaces = {}
    local allEdges = {}
    for _, face in ipairs(faceIndices) do
        local seenInFace = {}
        for i = 1, #face do
            local a = face[i]
            local b = face[(i % #face) + 1]
            if a >= 1 and a <= #centeredVerts and b >= 1 and b <= #centeredVerts and a ~= b then
                local k1, k2 = math.min(a, b), math.max(a, b)
                local key = k1 .. "_" .. k2
                if not seenInFace[key] then
                    seenInFace[key] = true
                    edgeCount[key] = (edgeCount[key] or 0) + 1
                    if not edgeMap[key] then
                        edgeMap[key] = {centeredVerts[a], centeredVerts[b], a, b}
                        edgeOrder[#edgeOrder + 1] = key
                        edgeFaces[key] = {}
                    end
                    edgeFaces[key][#edgeFaces[key] + 1] = face
                end
            end
        end
    end
    local edges = {}
    for _, key in ipairs(edgeOrder) do
        local count = edgeCount[key]
        local keep = true
        if count >= 2 then
            local fs = edgeFaces[key]
            local e = edgeMap[key]
            local va, vb = e[3], e[4]
            local pa, pb = centeredVerts[va], centeredVerts[vb]
            if pa and pb then
                local normals = {}
                for fi = 1, #fs do
                    local face = fs[fi]
                    local oppVerts = {}
                    for _, idx in ipairs(face) do
                        if idx ~= va and idx ~= vb then
                            oppVerts[#oppVerts + 1] = centeredVerts[idx]
                        end
                    end
                    if #oppVerts >= 1 then
                        local bestN = nil
                        local bestMag = 0
                        for _, ov in ipairs(oppVerts) do
                            if ov then
                                local n = (pb - pa):Cross(ov - pa)
                                if n.Magnitude > bestMag then
                                    bestN = n
                                    bestMag = n.Magnitude
                                end
                            end
                        end
                        if bestN and bestMag > 0.0001 then
                            normals[#normals + 1] = bestN.Unit
                        end
                    end
                end
                if #normals >= 2 then
                    local allCoplanar = true
                    for i = 2, #normals do
                        if math.abs(normals[1]:Dot(normals[i])) > 0.995 then
                        else
                            allCoplanar = false
                            break
                        end
                    end
                    if allCoplanar then
                        keep = false
                    end
                end
            end
        end
        if keep then
            local eNormal = nil
            local efs = edgeFaces[key]
            if efs and #efs >= 1 then
                local sumN = Vector3.zero
                local nCount = 0
                for _, face in ipairs(efs) do
                    if #face >= 3 then
                        local fp1 = centeredVerts[face[1]]
                        local fp2 = centeredVerts[face[2]]
                        local fp3 = centeredVerts[face[3]]
                        if fp1 and fp2 and fp3 then
                            local fn = (fp2 - fp1):Cross(fp3 - fp1)
                            if fn.Magnitude > 0.0001 then
                                sumN = sumN + fn.Unit
                                nCount = nCount + 1
                            end
                        end
                    end
                end
                if nCount > 0 and sumN.Magnitude > 0.0001 then
                    eNormal = sumN.Unit
                end
            end
            edges[#edges + 1] = {edgeMap[key][1], edgeMap[key][2], eNormal}
        end

        do
            local eNormal2 = nil
            local efs2 = edgeFaces[key]
            if efs2 and #efs2 >= 1 then
                local sumN2 = Vector3.zero
                local nC2 = 0
                for _, face in ipairs(efs2) do
                    if #face >= 3 then
                        local fp1 = centeredVerts[face[1]]
                        local fp2 = centeredVerts[face[2]]
                        local fp3 = centeredVerts[face[3]]
                        if fp1 and fp2 and fp3 then
                            local fn = (fp2 - fp1):Cross(fp3 - fp1)
                            if fn.Magnitude > 0.0001 then
                                sumN2 = sumN2 + fn.Unit
                                nC2 = nC2 + 1
                            end
                        end
                    end
                end
                if nC2 > 0 and sumN2.Magnitude > 0.0001 then
                    eNormal2 = sumN2.Unit
                end
            end
            allEdges[#allEdges + 1] = {edgeMap[key][1], edgeMap[key][2], eNormal2}
        end
    end

    local function getColor(pos)
        for _, f in ipairs(faces) do
            local color = chooseFaceColor(f, vertices, f.material, materialColors)
            if color then return color end
        end
        return nil
    end

    local function getColorForPos(pos)

        local bestDist = math.huge
        local bestColor = nil
        for _, f in ipairs(faces) do
            local indices = f.indices
            if #indices >= 3 then
                for i = 2, #indices - 1 do
                    local p1 = centeredVerts[indices[1]]
                    local p2 = centeredVerts[indices[i]]
                    local p3 = centeredVerts[indices[i+1]]
                    if p1 and p2 and p3 then
                        local centroid = (p1 + p2 + p3) / 3
                        local dist = (pos - centroid).Magnitude
                        if dist < bestDist then
                            bestDist = dist
                            bestColor = chooseFaceColor(indices, vertices, f.material, materialColors)
                        end
                    end
                end
            end
        end
        return bestColor
    end

    if mode == "face" then
        for fi, f in ipairs(faces) do
            faceIndex = fi
            local points = {}
            for _, idx in ipairs(f.indices) do
                points[#points + 1] = centeredVerts[idx] or vertices[idx].pos
            end
            local meshColor = chooseFaceColor(f.indices, vertices, f.material, materialColors)
            local cx, cy, cz = 0, 0, 0
            for _, p in ipairs(points) do cx = cx + p.x; cy = cy + p.y; cz = cz + p.z end
            local n = #points
            local faceCentroid = Vector3.new(cx/n, cy/n, cz/n)
            local color = resolveColor(meshColor, faceCentroid)
            local block = makeFacePanelCentered(points, thickness, color, blockId)
            if block then
                blocks[#blocks + 1] = block
                blockId = blockId + 1
            end
        end
    elseif mode == "wireframe" then
        for ei, edge in ipairs(allEdges) do
            faceIndex = math.ceil(ei / 3)
            local pA, pB, eNormal = edge[1], edge[2], edge[3]
            if not pA or not pB then continue end
            local mid = (pA + pB) / 2
            local meshColor = getColorForPos(mid)

            local color = resolveColor(meshColor, mid)
            if not color or color == "" then
                color = resolveColor(nil, mid)
            end
            if not color or color == "" then
                color = string.format("%.3f,%.3f,%.3f", opts.customR or 0.4, opts.customG or 0.6, opts.customB or 1.0)
            end

            local wfThick = math.max(stripSize, 0.25)
            local block = makeEdgeBlockFromRef(pA, pB, wfThick, color, blockId, eNormal)
            if block then
                blocks[#blocks + 1] = block
                blockId = blockId + 1
            end
        end
    elseif mode == "solid" then
        local function pointInTriangle2D(px, py, ax, ay, bx, by, cx, cy)
            local d1 = (px - bx) * (ay - by) - (ax - bx) * (py - by)
            local d2 = (px - cx) * (by - cy) - (bx - cx) * (py - cy)
            local d3 = (px - ax) * (cy - ay) - (cx - ax) * (py - ay)
            local hasNeg = (d1 < 0) or (d2 < 0) or (d3 < 0)
            local hasPos = (d1 > 0) or (d2 > 0) or (d3 > 0)
            return not (hasNeg and hasPos)
        end
        local function largestRectInGrid(grid, rows, cols)
            local heights = {}
            for c = 0, cols - 1 do heights[c] = 0 end
            local best = nil
            for r = 0, rows - 1 do
                for c = 0, cols - 1 do
                    heights[c] = grid[r][c] and (heights[c] + 1) or 0
                end
                local stack = {}
                for c = 0, cols do
                    local h = (c < cols) and heights[c] or 0
                    while #stack > 0 and heights[stack[#stack]] >= h do
                        local top = table.remove(stack)
                        local height = heights[top]
                        local left = (#stack > 0) and (stack[#stack] + 1) or 0
                        local width = c - left
                        local area = height * width
                        if height > 0 and (not best or area > best.area) then
                            best = {area = area, r0 = r + 1 - height, r1 = r, c0 = left, c1 = c - 1}
                        end
                    end
                    stack[#stack + 1] = c
                end
            end
            return best
        end
        local function decomposeFaceMosaic(p1, p2, p3, cellSize, color, blockIdStart)
            local minP = Vector3.new(math.min(p1.X, p2.X, p3.X), math.min(p1.Y, p2.Y, p3.Y), math.min(p1.Z, p2.Z, p3.Z))
            local maxP = Vector3.new(math.max(p1.X, p2.X, p3.X), math.max(p1.Y, p2.Y, p3.Y), math.max(p1.Z, p2.Z, p3.Z))
            local v1 = p2 - p1; local normal = v1:Cross(p3 - p1)
            if normal.Magnitude < 0.0001 then return {} end
            normal = normal.Unit
            local e1Len = (p2 - p1).Magnitude; local e2Len = (p3 - p2).Magnitude; local e3Len = (p3 - p1).Magnitude
            local longestDir
            if e1Len >= e2Len and e1Len >= e3Len then longestDir = (p2 - p1).Unit
            elseif e2Len >= e1Len and e2Len >= e3Len then longestDir = (p3 - p2).Unit
            else longestDir = (p3 - p1).Unit end
            local projDir = longestDir - normal * (normal:Dot(longestDir))
            if projDir.Magnitude < 0.0001 then return {} end
            projDir = projDir.Unit
            local localY = projDir:Cross(normal)
            if localY.Magnitude < 0.0001 then return {} end
            localY = localY.Unit
            local localX = localY:Cross(normal)
            if localX.Magnitude < 0.0001 then return {} end
            localX = localX.Unit
            local h1 = p1:Dot(localY); local h2 = p2:Dot(localY); local h3 = p3:Dot(localY)
            local w1 = p1:Dot(localX); local w2 = p2:Dot(localX); local w3 = p3:Dot(localX)
            local minH, maxH = math.min(h1, h2, h3), math.max(h1, h2, h3)
            local minW, maxW = math.min(w1, w2, w3), math.max(w1, w2, w3)
            local spanH = maxH - minH; local spanW = maxW - minW
            if spanH < 0.01 or spanW < 0.01 then return {} end
            local rows = math.max(1, math.ceil(spanH / cellSize))
            local cols = math.max(1, math.ceil(spanW / cellSize))
            local grid = {}
            local aW, aH = p1:Dot(localX), p1:Dot(localY)
            local bW, bH = p2:Dot(localX), p2:Dot(localY)
            local cW, cH = p3:Dot(localX), p3:Dot(localY)
            for r = 0, rows - 1 do
                grid[r] = {}
                local cy = minH + (r + 0.5) * (spanH / rows)
                for c = 0, cols - 1 do
                    local cx = minW + (c + 0.5) * (spanW / cols)
                    grid[r][c] = pointInTriangle2D(cx, cy, aW, aH, bW, bH, cW, cH)
                end
            end
            local result = {}
            local cellW = spanW / cols; local cellH = spanH / rows
            while true do
                local rect = largestRectInGrid(grid, rows, cols)
                if not rect then break end
                local cx0 = minW + rect.c0 * cellW
                local cx1 = minW + (rect.c1 + 1) * cellW
                local cy0 = minH + rect.r0 * cellH
                local cy1 = minH + (rect.r1 + 1) * cellH
                local ccx = (cx0 + cx1) / 2; local ccy = (cy0 + cy1) / 2
                local center3 = p1 + localX * (ccx - aW) + localY * (ccy - aH)
                local sizeW = cx1 - cx0; local sizeH = cy1 - cy0
                local cf = CFrame.fromMatrix(center3, localX, localY, normal)
                local colorStr = nil
                if type(color) == "string" and color ~= "" then colorStr = color
                elseif typeof(color) == "Color3" then colorStr = string.format("%.3f,%.3f,%.3f", color.R, color.G, color.B) end
                result[#result + 1] = cfToAsuBlock(cf, Vector3.new(sizeW, sizeH, thickness), colorStr, blockIdStart + #result, 0)
                for r = rect.r0, rect.r1 do
                    for c = rect.c0, rect.c1 do grid[r][c] = false end
                end
                if #result > 2000 then break end
            end
            return result
        end
        for fi, face in ipairs(faceIndices) do
            faceIndex = fi
            if not face or #face < 3 then
                continue
            end
            for i = 2, #face - 1 do
                local p1 = centeredVerts[face[1]]
                local p2 = centeredVerts[face[i]]
                local p3 = centeredVerts[face[i + 1]]
                if p1 and p2 and p3 then
                    local centroid = (p1 + p2 + p3) / 3
                    local meshColor = getColorForPos(centroid)
                    local color = resolveColor(meshColor, centroid)
                    local triBlocks = decomposeFaceMosaic(p1, p2, p3, stripSize * solidDetail, color, blockId)
                    if #triBlocks == 0 then
                        local dist = (p2 - p1).Magnitude
                        if dist < stripSize * solidDetail then
                            triBlocks = decomposeFaceMosaic(p1, p2, p3, dist * 0.5, color, blockId)
                        end
                    end
                    if #triBlocks == 0 then
                        local triN = (p2 - p1):Cross(p3 - p1)
                        if triN.Magnitude > 0.0001 then triN = triN.Unit else triN = nil end
                        local block = makeEdgeBlockFromRef(p1, p2, stripSize, color, blockId, triN)
                        if block then triBlocks = {block} end
                    end
                    for _, blk in ipairs(triBlocks) do
                        blocks[#blocks + 1] = blk
                        blockId = blockId + 1
                    end
                end
            end
        end
        local wfThickness = math.max(0.03, stripSize)
        for ei, edge in ipairs(edges) do
            faceIndex = math.ceil(ei / 3)
            local pA, pB, eNormal = edge[1], edge[2], edge[3]
            local mid = (pA + pB) / 2
            local meshColor = getColorForPos(mid)
            local color = resolveColor(meshColor, mid)
            local block = makeEdgeBlockFromRef(pA, pB, wfThickness, color, blockId, eNormal)
            if block then
                blocks[#blocks + 1] = block
                blockId = blockId + 1
            end
        end
    elseif mode == "voxel" then
        local cellMap = {}
        local voxelSize = math.max(0.05, stripSize)
        for fi, f in ipairs(faces) do
            faceIndex = fi
            local meshColor = chooseFaceColor(f.indices, vertices, f.material, materialColors)
            local fcx, fcy, fcz = 0, 0, 0
            for _, idx in ipairs(f.indices) do local p = centeredVerts[idx] or vertices[idx].pos; fcx = fcx + p.x; fcy = fcy + p.y; fcz = fcz + p.z end
            local fn = #f.indices
            local voxCentroid = Vector3.new(fcx/fn, fcy/fn, fcz/fn)
            local color = resolveColor(meshColor, voxCentroid)
            for _, tri in ipairs(triangulateFace(f.indices)) do
                local vp1 = centeredVerts[tri[1]] or vertices[tri[1]].pos
                local vp2 = centeredVerts[tri[2]] or vertices[tri[2]].pos
                local vp3 = centeredVerts[tri[3]] or vertices[tri[3]].pos
                rasterizeTriangleToVoxels(cellMap, vp1, vp2, vp3, voxelSize / math.max(scale, 0.001), color)
            end
        end
        blocks = voxelCellsToBlocks(cellMap, voxelSize, blockId)
    end
    return blocks
end

convertObjToBlocks = function(filePath, scale, thickness, mode, material, colorOpts)
    scale = tonumber(scale) or 1
    thickness = tonumber(thickness) or 0.2
    material = tostring(material or "PlasticBlock")
    if scale <= 0 or thickness <= 0 then
        return nil, nil, "Scale/thickness must be > 0"
    end
    local readFileOk, text = pcall(readfile, filePath)
    if not readFileOk then
        return nil, nil, "Failed to read file: " .. tostring(text)
    end
    local parsed = parseObjData(text, filePath)
    if #(parsed.vertices or {}) == 0 then
        return nil, nil, "No vertices found in OBJ file"
    end
    local blocks = buildObjBlocks(parsed, scale, thickness, mode, colorOpts)
    if #blocks == 0 then
        return nil, nil, "No blocks produced"
    end
    local minY = math.huge
    for _, blk in ipairs(blocks) do
        if blk.Position then
            local nums = parseNums(blk.Position)
            if #nums >= 3 and nums[2] < minY then
                minY = nums[2]
            end
        end
    end
    if minY < 4 then
        local yShift = 4 - minY
        for _, blk in ipairs(blocks) do
            if blk.Position then
                local nums = parseNums(blk.Position)
                if #nums >= 3 then
                    nums[2] = nums[2] + yShift
                    blk.Position = string.format("%.6f, %.6f, %.6f", nums[1], nums[2], nums[3])
                end
            end
        end
    end
    return material, blocks, nil
end

convertObjToBuild = function(filePath, buildName, scale, thickness, material, mode, colorOpts)
    local outMaterial, blocks, err = convertObjToBlocks(filePath, scale, thickness, mode, material, colorOpts)
    if not outMaterial then
        return nil, err
    end
    return writeConvertedBuild(buildName, outMaterial, blocks)
end
end

local function bytesToFloatLE(b1, b2, b3, b4)
    local v = (((b4 * 256) + b3) * 256 + b2) * 256 + b1
    if v == 0 then return 0 end
    local sign = bit.band(v, 0x80000000) ~= 0 and -1 or 1
    local exp = bit.band(bit.rshift(v, 23), 0xFF)
    local mant = bit.band(v, 0x7FFFFF)
    if exp == 0 then
        return sign * (mant / 8388608) * 2^-126
    end
    if exp == 255 then
        return sign * math.huge
    end
    return sign * (1 + mant / 8388608) * 2^(exp - 127)
end

local function asuToCF(pos, rot)
    local p = {0,0,0}
    if type(pos) == "string" then
        local v = parseNums(pos)
        if #v >= 3 then p = {v[1],v[2],v[3]} end
    elseif type(pos) == "table" then p = pos end
    local r = {0,0,0}
    if type(rot) == "string" then
        local v = parseNums(rot)
        if #v >= 3 then r = {math.rad(v[1]),math.rad(v[2]),math.rad(v[3])} end
    elseif type(rot) == "table" then
        for i,v in ipairs(rot) do r[i] = math.rad(tonumber(v) or 0) end
    end
    return CFrame.new(p[1] or 0, p[2] or 0, p[3] or 0) * CFrame.Angles(r[1] or 0, r[2] or 0, r[3] or 0)
end

local function convertAsuToPRS(asuData)
    if type(asuData) ~= "table" then return nil end
    local prs = {}
    local entriesById = {}
    local pendingBindTables = {}
    local globalIdCounter = 1
    for blockName, blocks in pairs(asuData) do
        if type(blocks) == "table" then
            prs[blockName] = prs[blockName] or {}
            for _, block in ipairs(blocks) do
                if type(block) == "table" then
                    local pos = block.Position or block.position or block.Pos or block.pos
                    local rot = block.Rotation or block.rotation or block.Rot or block.rot
                    if not pos then continue end
                    local cf
                    if type(pos) == "string" or type(rot) == "string" then
                        cf = asuToCF(tostring(pos), rot and tostring(rot) or "0,0,0")
                    else
                        cf = CFrame.new(
                            (type(pos)=="table") and (pos[1] or pos.X or 0) or 0,
                            (type(pos)=="table") and (pos[2] or pos.Y or 0) or 0,
                            (type(pos)=="table") and (pos[3] or pos.Z or 0) or 0
                        ) * CFrame.Angles(
                            math.rad((type(rot)=="table") and (rot[1] or rot.X or 0) or 0),
                            math.rad((type(rot)=="table") and (rot[2] or rot.Y or 0) or 0),
                            math.rad((type(rot)=="table") and (rot[3] or rot.Z or 0) or 0)
                        )
                    end
                    local sz = block.Size or block.size
                    local col = block.Color or block.color or block.Col or block.col
                    local sizeStr = nil
                    if sz then
                        local sizeVec = Vector3.new(1,1,1)
                        if type(sz) == "string" then
                            local v = parseNums(sz)
                            if #v >= 3 then sizeVec = Vector3.new(v[1],v[2],v[3]) end
                        elseif type(sz) == "table" then
                            sizeVec = Vector3.new(sz[1] or sz.X or 1, sz[2] or sz.Y or 1, sz[3] or sz.Z or 1)
                        elseif type(sz) == "number" then
                            sizeVec = Vector3.new(sz,sz,sz)
                        end
                        sizeStr = v3Str(sizeVec)
                    end
                    local colStr2 = nil
                    if col and type(col) == "string" then
                        local cv = parseNums(col)
                        if #cv >= 3 then
                            local r2,g2,b2 = cv[1],cv[2],cv[3]
                            if r2>1 or g2>1 or b2>1 then r2=r2/255; g2=g2/255; b2=b2/255 end
                            colStr2 = string.format("%.4f,%.4f,%.4f", math.clamp(r2,0,1), math.clamp(g2,0,1), math.clamp(b2,0,1))
                        end
                    end
                    local extras = collectAsuExtras(block)
                    local mergedBoolValues, mergedNumberValues = mergePropertyMaps(block.BoolValues, block.NumberValues, extras)
                    if mergedNumberValues then
                        if mergedNumberValues.LastDirrection ~= nil and mergedNumberValues.LastDirection == nil then
                            mergedNumberValues.LastDirection = mergedNumberValues.LastDirrection
                            mergedNumberValues.LastDirrection = nil
                        end
                    end
                    local assignedId = block.ID or globalIdCounter
                    globalIdCounter = globalIdCounter + 1
                    local entry = {
                        CFrame = cfStr(cf),
                        ID = assignedId,
                        Size = sizeStr,
                        Col = colStr2,
                        Transparency = block.Transparency,
                        Anchored = block.Anchored,
                        CanCollide = block.CanCollide,
                        ShowShadow = block.ShowShadow,
                        ASUExtra = extras,
                        BoolValues = mergedBoolValues,
                        NumberValues = mergedNumberValues,
                    }

                    if block.SecondaryPartPosition then entry.SecondaryPartPosition = tostring(block.SecondaryPartPosition) end
                    if block.SecondaryPartRotation then entry.SecondaryPartRotation = tostring(block.SecondaryPartRotation) end
                    if block.Stiffness ~= nil then entry.Stiffness = block.Stiffness end
                    if block.Damping ~= nil then entry.Damping = block.Damping end
                    if block.TargetLength ~= nil then entry.TargetLength = block.TargetLength end
                    if block.MaxLength ~= nil then entry.MaxLength = block.MaxLength end
                    if block.MinLength ~= nil then entry.MinLength = block.MinLength end
                    if block.Length ~= nil then entry.Length = block.Length end
                    if block.AngleLimit ~= nil then entry.AngleLimit = block.AngleLimit end
                    if block.MatchRotation ~= nil then entry.MatchRotation = block.MatchRotation end
                    if block.ShowConstraint ~= nil then entry.ShowConstraint = block.ShowConstraint end
                    if block.ServoTorque ~= nil then entry.ServoTorque = block.ServoTorque end
                    if block.ServoSpeed ~= nil then entry.ServoSpeed = block.ServoSpeed end
                    if block.BarLength ~= nil then entry.BarLength = block.BarLength end
                    if block.WheelTorque ~= nil then entry.WheelTorque = block.WheelTorque end
                    if block.Text then entry.Text = block.Text end
                    if type(block.BindTable) == "table" then
                        entry.BindTable = cloneJsonValue(block.BindTable)
                        pendingBindTables[#pendingBindTables + 1] = entry.BindTable
                    end
                    if entry.ID ~= nil then
                        entriesById[entry.ID] = entry
                        entriesById[tostring(entry.ID)] = entry
                    end
                    table.insert(prs[blockName], entry)
                end
            end
        end
    end
    for _, bindTable in ipairs(pendingBindTables) do
        for _, row in ipairs(bindTable) do
            if type(row) == "table" then
                local target = entriesById[row[1]] or entriesById[tostring(row[1])]
                local bindName = row[2]
                if target and bindName then
                    target.NumberValues = target.NumberValues or {}
                    target.NumberValues[bindName] = tonumber(row[3]) or row[3]
                end
            end
        end
    end
    return prs
end

local function convertedBlocksToPRS(material, blocks)
    if type(blocks) ~= "table" or #blocks == 0 then
        return nil
    end
    local materialName = tostring(material or "PlasticBlock")
    if materialName ~= "Auto" then
        return convertAsuToPRS({[materialName] = blocks})
    end
    local groups = {}
    for _, block in ipairs(blocks) do
        local blockMaterial = tostring(block.Material or "PlasticBlock")
        groups[blockMaterial] = groups[blockMaterial] or {}
        groups[blockMaterial][#groups[blockMaterial] + 1] = block
    end
    return convertAsuToPRS(groups)
end

local function convertRobloxAssetToBlocks(assetId, scale)
    local id = tostring(assetId or ""):match("%d+")
    if not id then return nil, "Enter a valid Roblox asset ID" end
    scale = math.clamp(tonumber(scale) or 1, 0.05, 10)
    local ok, loaded = pcall(function()
        return game:GetObjects("rbxassetid://" .. id)
    end)
    if not ok or type(loaded) ~= "table" or not loaded[1] then
        return nil, "Roblox could not load this asset ID"
    end
    local root = loaded[1]
    local parts = {}
    for _, obj in ipairs(root:GetDescendants()) do
        if obj:IsA("BasePart") then parts[#parts + 1] = obj end
    end
    if root:IsA("BasePart") then parts[#parts + 1] = root end
    if #parts == 0 then return nil, "The asset has no BasePart or MeshPart" end
    local function mapRobloxMaterial(mat)
        local n = tostring(mat and mat.Name or ""):lower()
        if n:find("wood") then return "WoodBlock" end
        if n:find("metal") or n:find("diamond") or n:find("foil") then return "MetalBlock" end
        if n:find("glass") then return "GlassBlock" end
        if n:find("neon") then return "NeonBlock" end
        if n:find("fabric") then return "FabricBlock" end
        if n:find("granite") then return "GraniteBlock" end
        if n:find("marble") then return "MarbleBlock" end
        if n:find("slate") then return "SlateBlock" end
        if n:find("brick") then return "BrickBlock" end
        if n:find("cobble") then return "CobblestoneBlock" end
        if n:find("grass") then return "GrassBlock" end
        if n:find("ice") then return "IceBlock" end
        if n:find("sand") then return "SandBlock" end
        if n:find("concrete") then return "ConcreteBlock" end
        if n:find("corroded") then return "CorrodedMetalBlock" end
        if n:find("pebble") then return "PebbleBlock" end
        return "PlasticBlock"
    end
    local center = Vector3.zero
    for _, part in ipairs(parts) do center = center + part.Position end
    center = center / #parts
    local blocks = {}
    for index, part in ipairs(parts) do
        local rx, ry, rz = part.CFrame:ToOrientation()
        blocks[#blocks + 1] = {
            Position = {(part.Position.X - center.X) * scale, (part.Position.Y - center.Y) * scale, (part.Position.Z - center.Z) * scale},
            Rotation = {math.deg(rx), math.deg(ry), math.deg(rz)},
            Size = {math.max(0.05, part.Size.X * scale), math.max(0.05, part.Size.Y * scale), math.max(0.05, part.Size.Z * scale)},
            Color = colStr(part.Color),
            Transparency = part.Transparency,
            Anchored = true,
            CanCollide = part.CanCollide,
            Material = mapRobloxMaterial(part.Material),
            ID = index,
        }
    end
    pcall(function() root:Destroy() end)
    return blocks, nil
end

local function convertPRStoBH(prsData)
    if type(prsData) ~= "table" then return nil end
    local data = {}
    for blockName, blocks in pairs(prsData) do
        if type(blocks) == "table" and #blocks > 0 then
            local arr = {}
            for idx, bi in ipairs(blocks) do
                local entry = {}
                entry.ID = bi.ID or idx
                entry.Anchored = bi.Anchored ~= false
                entry.CanCollide = bi.CanCollide ~= false
                entry.Transparency = bi.Transparency or 0
                entry.CastShadow = bi.ShowShadow ~= false
                if bi.CFrame then
                    if type(bi.CFrame) == "string" then
                        local nums = {}
                        for v in bi.CFrame:gmatch("[^,]+") do
                            local n = tonumber(v:match("^%s*(.-)%s*$"))
                            if n then table.insert(nums, n) end
                        end
                        entry.CFrame = nums
                    elseif type(bi.CFrame) == "table" then
                        entry.CFrame = bi.CFrame
                    end
                end
                if bi.Size then
                    if type(bi.Size) == "string" then
                        local nums = {}
                        for v in bi.Size:gmatch("[^,]+") do
                            local n = tonumber(v:match("^%s*(.-)%s*$"))
                            if n then table.insert(nums, n) end
                        end
                        entry.Size = nums
                    elseif type(bi.Size) == "table" then
                        entry.Size = bi.Size
                    end
                end
                if bi.Col then
                    local cv = {}
                    for v in bi.Col:gmatch("[^,]+") do
                        local n = tonumber(v:match("^%s*(.-)%s*$"))
                        if n then table.insert(cv, n) end
                    end
                    if #cv >= 3 then
                        local r = math.floor((cv[1] or 1) * 255 + 0.5)
                        local g = math.floor((cv[2] or 1) * 255 + 0.5)
                        local b = math.floor((cv[3] or 1) * 255 + 0.5)
                        entry.Color = string.format("%02x%02x%02x", r, g, b)
                    end
                end
                local mVals = {}
                if bi.NumberValues and type(bi.NumberValues) == "table" then
                    for k, v in pairs(bi.NumberValues) do mVals[k] = v end
                end
                if bi.BoolValues and type(bi.BoolValues) == "table" then
                    for k, v in pairs(bi.BoolValues) do mVals[k] = v end
                end
                if next(mVals) then entry.MValues = mVals end
                if bi.SecCFrame then
                    if type(bi.SecCFrame) == "string" then
                        local nums = {}
                        for v in bi.SecCFrame:gmatch("[^,]+") do
                            local n = tonumber(v:match("^%s*(.-)%s*$"))
                            if n then table.insert(nums, n) end
                        end
                        entry.SecCFrame = nums
                    elseif type(bi.SecCFrame) == "table" then
                        entry.SecCFrame = bi.SecCFrame
                    end
                end
                if bi.BindTable and type(bi.BindTable) == "table" then
                    local idToBlock = {}
                    for bName, bList in pairs(prsData) do
                        if type(bList) == "table" then
                            for _, bEntry in ipairs(bList) do
                                if bEntry.ID then idToBlock[bEntry.ID] = bName end
                            end
                        end
                    end
                    for _, bindRow in ipairs(bi.BindTable) do
                        if type(bindRow) == "table" and bindRow[1] then
                            local targetID = bindRow[1]
                            local bindName = bindRow[2]
                            local bindValue = bindRow[3]
                            local targetName = idToBlock[targetID]
                            if not entry.Binds then entry.Binds = {} end
                            table.insert(entry.Binds, {targetID, bindName, bindValue})
                        end
                    end
                end
                table.insert(arr, entry)
            end
            data[blockName] = arr
        end
    end
    return {Data = data, AutoBuild_Version = "v1"}
end

local function saveBuildToFile(fileName, buildData)
    ensureFolder()
    local bhData = convertPRStoBH(buildData)
    if bhData then

        local function jsonEncode(val)
            if val == nil then return "null" end
            if type(val) == "boolean" then return val and "true" or "false" end
            if type(val) == "number" then return tostring(val) end
            if type(val) == "string" then return '"' .. val:gsub('\\','\\\\'):sub(1, 200) .. '"' end
            if type(val) ~= "table" then return "null" end

            local isArray = true
            local maxIdx = 0
            for k in pairs(val) do
                if type(k) == "number" and k == math.floor(k) and k >= 1 then
                    if k > maxIdx then maxIdx = k end
                else
                    isArray = false; break
                end
            end
            if isArray and maxIdx == #val then

                local parts = {}
                parts[#parts+1] = '['
                for i = 1, #val do
                    if i > 1 then parts[#parts+1] = ',' end
                    parts[#parts+1] = jsonEncode(val[i])
                end
                parts[#parts+1] = ']'
                return table.concat(parts)
            else

                local parts = {}
                parts[#parts+1] = '{'
                local sortedKeys = {}
                for k in pairs(val) do sortedKeys[#sortedKeys+1] = k end
                table.sort(sortedKeys, function(a, b) return tostring(a) < tostring(b) end)
                local first = true
                for _, k in ipairs(sortedKeys) do
                    if not first then parts[#parts+1] = ',' end
                    first = false
                    parts[#parts+1] = '"' .. tostring(k) .. '":'
                    parts[#parts+1] = jsonEncode(val[k])
                end
                parts[#parts+1] = '}'
                return table.concat(parts)
            end
        end
        local ok, result = pcall(function()
            return jsonEncode(bhData)
        end)
        if ok and result then
            writefile(FOLDER_PREFIX .. fileName .. ".Build", result)
            return true, "BH"
        end
    end
    return false
end

local function convertMcLaren(rawData)

    local converted = {}
    local invertedBinds = {}
    local idToKey = {}

    for blockName, blocks in pairs(rawData) do
        if type(blocks) == "table" and #blocks > 0 then
            local arr = {}
            for idx, bi in ipairs(blocks) do
                local entry = {}
                entry.ID = bi.ID
                entry.Transparency = bi.Transparency
                entry.Anchored = bi.Anchored
                entry.CanCollide = bi.CanCollide
                entry.CFrame = bi.CFrame
                entry.Size = bi.Size

                if bi.Color then
                    local hex = tostring(bi.Color):gsub("^#","")
                    local r = tonumber(hex:sub(1,2), 16) or 255
                    local g = tonumber(hex:sub(3,4), 16) or 255
                    local b = tonumber(hex:sub(5,6), 16) or 255
                    entry.Col = string.format("%.6f,%.6f,%.6f", r/255, g/255, b/255)
                end

                if bi.CastShadow ~= nil then
                    entry.ShowShadow = bi.CastShadow
                end

                if bi.MValues and type(bi.MValues) == "table" then
                    local numV, boolV = {}, {}
                    for k, v in pairs(bi.MValues) do
                        if type(v) == "number" then
                            numV[k] = v
                        elseif type(v) == "boolean" then
                            boolV[k] = v
                        else
                            numV[k] = v
                        end
                    end
                    if next(numV) then entry.NumberValues = numV end
                    if next(boolV) then entry.BoolValues = boolV end
                end

                if bi.SecCFrame then
                    entry.SecCFrame = bi.SecCFrame
                end

                local knownKeys = {ID=true, Transparency=true, Anchored=true, CanCollide=true,
                    CFrame=true, Size=true, Color=true, CastShadow=true, Binds=true, MValues=true, SecCFrame=true}
                local extra = {}
                for k, v in pairs(bi) do
                    if not knownKeys[k] then extra[k] = v end
                end
                if next(extra) then entry.ASUExtra = extra end
                arr[#arr+1] = entry

                if bi.ID then
                    idToKey[bi.ID] = {blockName = blockName, idx = idx}
                end

                if bi.Binds and type(bi.Binds) == "table" then
                    for _, bindRow in ipairs(bi.Binds) do
                        if type(bindRow) == "table" and bindRow[1] then
                            local sourceID = bindRow[1]
                            local bindName = bindRow[2]
                            local bindValue = bindRow[3]
                            if not invertedBinds[sourceID] then
                                invertedBinds[sourceID] = {}
                            end

                            local bindEntry = {bi.ID, bindName, bindValue}
                            table.insert(invertedBinds[sourceID], bindEntry)
                        end
                    end
                end
            end
            converted[blockName] = arr
        end
    end

    for sourceID, bindList in pairs(invertedBinds) do
        local key = idToKey[sourceID]
        if key then
            local arr = converted[key.blockName]
            if arr and arr[key.idx] then
                arr[key.idx].BindTable = bindList
            end
        end
    end

    return converted
end

local function loadBuildFromFile(fileName)
    ensureFolder()
    local json
    local searchPaths = {FOLDER_PREFIX, FOLDER_PATH .. "/", ""}
    for _, root in ipairs(searchPaths) do
        local paths = {
            root .. fileName .. ".Build",
            root .. fileName .. ".build",
            root .. fileName .. ".json",
            root .. fileName .. ".bh",
            root .. fileName .. ".BH",
            root .. fileName .. ".txt",
            root .. fileName,
        }
        for _, p in ipairs(paths) do
            if isfile(p) then json = readfile(p) ; break end
        end
        if json then break end
    end
    if not json then return nil, nil end

    if json:find("BuilderHub") or json:find("%[Build%]") then
        local buildLine = json:match("Build%s*=%s*(%b{})")
        if buildLine then
            local okB, decB = pcall(function() return HttpService:JSONDecode(buildLine) end)
            if okB and decB and decB.b and type(decB.b) == "table" then
                local dataObj = decB.b
                for bn, blist in pairs(dataObj) do
                    if type(blist) == "table" then
                        for _, b in ipairs(blist) do
                            if type(b) == "table" then
                                if b.p then b.Position = b.p end
                                if b.r then b.Rotation = b.r end
                                if b.cl then b.Color = b.cl end
                                if b.sz then b.Size = b.sz end
                                if b.a ~= nil then b.Anchored = b.a end
                                if b.cc ~= nil then b.CanCollide = b.cc end
                                if b.ss ~= nil then b.ShowShadow = b.ss end
                                if b.i then b.ID = b.i end
                                if b.nv then b.NumberValues = b.nv end
                                if b.bd then b.BindTable = b.bd end
                            end
                        end
                    end
                end
                return dataObj, "Asu"
            end
        end
    end

    local ok, dec = pcall(function() return HttpService:JSONDecode(json) end)
    if not ok or not dec then return nil, nil end

    if type(dec) == "table" and dec.Data and type(dec.Data) == "table" and not dec.format then
        local inner = dec.Data
        for _, v in pairs(inner) do
            if type(v) == "table" and #v > 0 and type(v[1]) == "table" then
                local fb = v[1]
                if fb.CFrame and (type(fb.CFrame) == "table" or type(fb.CFrame) == "string") then
                    return convertMcLaren(inner), "BH"
                end
            end
        end
    end
    if type(dec) == "table" and #dec >= 2 then
        local cats, dataObj = dec[1], dec[2]
        if type(cats) == "table" and type(dataObj) == "table" then
            for _, blocks in pairs(dataObj) do
                if type(blocks) == "table" and #blocks > 0 then
                    local fb = blocks[1]
                    if type(fb) == "table" and (fb.Position or fb.position) then
                        return dataObj, "Asu"
                    end
                end
            end
        end
    end
    if dec.format and dec.data then
        if dec.format == "Asu" then
            return dec.data, "Asu"
        end
        return dec.data, dec.format
    end

    if type(dec) == "table" and dec.b and type(dec.b) == "table" and dec.t then
        local dataObj = dec.b
        for _, blocks in pairs(dataObj) do
            if type(blocks) == "table" and #blocks > 0 then
                local fb = blocks[1]
                if type(fb) == "table" and (fb.p or fb.cl or fb.i) then
                    for bn, blist in pairs(dataObj) do
                        if type(blist) == "table" then
                            for _, b in ipairs(blist) do
                                if type(b) == "table" then
                                    if b.p then b.Position = b.p end
                                    if b.r then b.Rotation = b.r end
                                    if b.cl then b.Color = b.cl end
                                    if b.sz then b.Size = b.sz end
                                    if b.a ~= nil then b.Anchored = b.a end
                                    if b.cc ~= nil then b.CanCollide = b.cc end
                                    if b.ss ~= nil then b.ShowShadow = b.ss end
                                    if b.i then b.ID = b.i end
                                    if b.nv then b.NumberValues = b.nv end
                                    if b.sv then
                                        b.StringValues = b.sv
                                        if b.sv.ActionName then
                                            b.NumberValues = b.NumberValues or {}
                                        end
                                    end
                                    if b.bd then b.BindTable = b.bd end
                                end
                            end
                        end
                    end
                    return dataObj, "Asu"
                end
            end
        end
    end
    if type(dec) == "table" and not dec.format and #dec == 0 then
        local hasBlockData = false
        for _, v in pairs(dec) do
            if type(v) == "table" and #v > 0 and type(v[1]) == "table" then
                hasBlockData = true
                local fb = v[1]

                if fb.CFrame and type(fb.CFrame) == "string" then
                    return dec, "PRS"
                end

                if fb.CFrame and type(fb.CFrame) == "table" then
                    for _, blocks in pairs(dec) do
                        if type(blocks) == "table" then
                            for _, b in ipairs(blocks) do
                                if type(b) == "table" then
                                    if type(b.CFrame) == "table" then
                                        b.Position = table.concat({b.CFrame[1], b.CFrame[2], b.CFrame[3]}, ", ")
                                        b.Rotation = "0, 0, 0"
                                        b.CFrame = nil
                                    end
                                    if type(b.Size) == "table" then
                                        b.Size = table.concat({b.Size[1], b.Size[2], b.Size[3]}, ", ")
                                    end
                                    if b.MValues and type(b.MValues) == "table" then
                                        local mv_num, mv_bool = {}, {}
                                        for mk, mv in pairs(b.MValues) do
                                            if type(mv) == "boolean" then
                                                mv_bool[mk] = mv
                                            else
                                                mv_num[mk] = mv
                                            end
                                        end
                                        if next(mv_num) then b.NumberValues = mv_num end
                                        if next(mv_bool) then b.BoolValues = mv_bool end
                                    end
                                    if b.Binds then b.BindTable = b.Binds end
                                    if b.CastShadow ~= nil then b.ShowShadow = b.CastShadow end
                                end
                            end
                        end
                    end
                    return dec, "Asu"
                end

                if fb.Position or fb.position or fb.Pos or fb.pos then
                    return dec, "Asu"
                end
            end
        end
        if hasBlockData then
            return dec, "Asu"
        end
    end
    return dec, "PRS"
end

local _buildFilesCache = nil
local _buildFilesCacheTime = 0
local function getSavedBuilds()
    if _buildFilesCache and (tick() - _buildFilesCacheTime) < 300 then
        return _buildFilesCache
    end
    ensureFolder()
    local builds, seen = {}, {}
    local function scanDir(dir, depth)
        if depth > 3 then return end
        local ok, items = pcall(listfiles, dir)
        if not ok or type(items) ~= "table" then return end
        for _, fp in ipairs(items) do
            if isfolder(fp) then
                scanDir(fp, depth + 1)
            else
                local n = fp:match("([^/\\]+)%.Build$") or fp:match("([^/\\]+)%.build$")
                if n and not seen[n:lower()] then
                    table.insert(builds, n)
                    seen[n:lower()] = true
                end
            end
        end
        task.wait()
    end
    scanDir(FOLDER_PATH, 0)
    table.sort(builds, function(a, b) return a:lower() < b:lower() end)
    _buildFilesCache = builds
    _buildFilesCacheTime = tick()
    return builds
end

local function teleportCharacterTo(pos)
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    hrp.CFrame = CFrame.new(pos)
    hrp.AssemblyLinearVelocity = Vector3.zero
    hrp.AssemblyAngularVelocity = Vector3.zero
    return true
end

local function touchPart(part)
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not part then return false end
    if firetouchinterest then
        pcall(function()
            firetouchinterest(hrp, part, 0)
            task.wait()
            firetouchinterest(hrp, part, 1)
        end)
        return true
    end
    return false
end

local function equipAllTools()
    local ch = Character or LocalPlayer.Character
    if not ch or not LocalPlayer:FindFirstChild("Backpack") then return end
    local buildToolNames = {"BuildingTool", "PaintingTool", "PropertiesTool", "ScalingTool", "DeleteTool", "TrowelTool", "BindTool"}
    for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
        if tool:IsA("Tool") then
            local isBuild = false
            for _, bn in ipairs(buildToolNames) do
                if tool.Name == bn then isBuild = true; break end
            end
            if isBuild then
                pcall(function() tool.Parent = ch end)
            end
        end
    end
    task.wait(0.03)
    for _, tool in pairs(ch:GetChildren()) do
        if tool:IsA("Tool") then
            pcall(function() tool:Activate() end)
        end
    end
end

local function setStatus(text)
    if StatusLabelRef then StatusLabelRef.Text = text end
    if MiscStatusLabelRef then MiscStatusLabelRef.Text = text end
end

local function placeBlock(blockName, cframe, relativeTo)
    equipAllTools()
    local ch = Character or LocalPlayer.Character
    local tool = ch and ch:FindFirstChild("BuildingTool")
    if not tool then return false end
    relativeTo = relativeTo or getPlayerZone(LocalPlayer)
    if not relativeTo then return false end
    tool.RF:InvokeServer(
        blockName,
        getBlockID(blockName),
        relativeTo,
        relativeTo.CFrame:ToObjectSpace(cframe),
        true
    )
    return true
end

local function rescaleBlock(block, cf, sz)
    if not block or not block:FindFirstChild("PPart") then return false end
    equipAllTools()
    local ch = Character or LocalPlayer.Character
    local tool = ch and ch:FindFirstChild("ScalingTool")
    if not tool then return false end
    local ok = pcall(function()
        tool.RF:InvokeServer(block, sz, cf)
    end)
    if not ok then
        pcall(function() block.PPart.CFrame = cf end)
        return false
    end
    return true
end

local function paintBlock(block, color)
    if not block or not block:FindFirstChild("PPart") then return false end
    if block.PPart.Color == color then return true end
    equipAllTools()
    local ch = Character or LocalPlayer.Character
    local tool = ch and ch:FindFirstChild("PaintingTool")
    if not tool then return false end
    local ok = pcall(function()
        tool.RF:InvokeServer({block, color})
    end)
    return ok
end

local function moveBlock(block, cf)
    if not block or not block:FindFirstChild("PPart") then return false end
    equipAllTools()
    local ch = Character or LocalPlayer.Character
    local tool = ch and ch:FindFirstChild("ScalingTool")
    if not tool then
        pcall(function() block.PPart.CFrame = cf end)
        return true
    end
    local ok = pcall(function()
        tool.RF:InvokeServer(block, block.PPart.Size, cf)
    end)
    if not ok then
        pcall(function() block.PPart.CFrame = cf end)
    end
    return true
end

local function copyBuild()
    if not selectedPlayer then return nil end
    local playerBlocks = BlocksFolder:FindFirstChild(selectedPlayer.Name)
    if not playerBlocks then return nil end
    local playerZone = getPlayerZone(selectedPlayer)
    if not playerZone then return nil end
    local buildData = {}
    local idCounter = 1
    local idToBlock = {}
    for _, block in pairs(playerBlocks:GetChildren()) do
        if block:FindFirstChild("PPart") then
            local ppart = block.PPart
            local relCF = playerZone.CFrame:ToObjectSpace(ppart.CFrame)
            local isBlock = block.Name:sub(-5) == "Block"
            local realTransp = ppart.Transparency
            if not isBlock then
                for _, desc in pairs(block:GetChildren()) do
                    if (desc:IsA("BasePart") or desc:IsA("UnionOperation")) and desc ~= ppart and desc.Transparency < 1 then
                        realTransp = desc.Transparency
                        break
                    end
                end
            end
            buildData[block.Name] = buildData[block.Name] or {}
            local entry = {
                CFrame = cfStr(relCF),
                Size = v3Str(ppart.Size),
                Col = colStr(ppart.Color),
                Transparency = realTransp,
                Anchored = ppart.Anchored,
                CanCollide = ppart.CanCollide,
                ShowShadow = ppart.CastShadow ~= false,
                ID = idCounter,
            }
            idCounter = idCounter + 1
            local boolVals = {}
            local numVals = {}
            for _, child in pairs(block:GetChildren()) do
                if child:IsA("BoolValue") then
                    boolVals[child.Name] = child.Value
                elseif (child:IsA("NumberValue") or child:IsA("IntValue")) and not child.Name:find("^Bind") then
                    numVals[child.Name] = child.Value
                end
            end
            for _, child in pairs(ppart:GetChildren()) do
                if child:IsA("BoolValue") then
                    boolVals[child.Name] = child.Value
                elseif (child:IsA("NumberValue") or child:IsA("IntValue")) and not child.Name:find("^Bind") then
                    numVals[child.Name] = child.Value
                end
            end
            if block.Name:find("Piston") then
                local fwd = block:GetAttribute("Forward")
                numVals.LastDirection = (fwd == true) and 1 or 0
            end
            if next(boolVals) then entry.BoolValues = boolVals end
            if next(numVals) then entry.NumberValues = numVals end
            table.insert(buildData[block.Name], entry)
            idToBlock[idCounter - 1] = block
        end
    end
    do
        local BKEYS = {"BindFire","BindActivate","BindUp","BindLeft","BindDown","BindRight"}
        local CONTROLLER_NAMES = {
            SwitchBig = true, Button = true, CarSeat = true, Switch = true,
            SensorBlock = true, RemoteController = true, PilotSeat = true,
            Lever = true, Gate = true, Delay = true,
        }
        local tBinds = {}
        for _, blk in pairs(playerBlocks:GetChildren()) do
            if blk:FindFirstChild("PPart") then
                for _, bk in ipairs(BKEYS) do
                    local bv = blk:FindFirstChild(bk)
                    if bv then
                        local keyCode = nil
                        local kc = bv:FindFirstChild("DefaultInputKeyCode")
                        if kc and (kc:IsA("IntValue") or kc:IsA("NumberValue")) then
                            keyCode = kc.Value
                        end
                        local tid = nil
                        if bv:IsA("ObjectValue") and bv.Value then
                            for id2, b2 in pairs(idToBlock) do
                                if b2 == bv.Value then tid = id2 break end
                            end
                        elseif bv:IsA("IntValue") or bv:IsA("NumberValue") then
                            for id2, b2 in pairs(idToBlock) do
                                if b2 == blk then tid = id2 break end
                            end
                        end
                        if tid then
                            tBinds[tid] = tBinds[tid] or {}
                            table.insert(tBinds[tid], {bk, keyCode or bv.Value or -1})
                        end
                    end
                end
            end
        end
        for _, blk in pairs(playerBlocks:GetChildren()) do
            if not blk:FindFirstChild("PPart") then continue end
            if not CONTROLLER_NAMES[blk.Name] then continue end
            local bid = nil
            for id2, b2 in pairs(idToBlock) do if b2 == blk then bid = id2 break end end
            if not bid then continue end
            local bEntry = nil
            for _, ent in ipairs(buildData[blk.Name] or {}) do
                if ent.ID == bid then bEntry = ent break end
            end
            if not bEntry then continue end
            local bSet = {}
            for _, ch in pairs(blk:GetChildren()) do
                if ch:IsA("ObjectValue") and ch.Value then
                    for id2, b2 in pairs(idToBlock) do if b2 == ch.Value then bSet[id2] = true break end end
                end
            end
            local pp2 = blk:FindFirstChild("PPart")
            if pp2 then
                for _, ch in pairs(pp2:GetChildren()) do
                    if ch:IsA("ObjectValue") and ch.Value then
                        for id2, b2 in pairs(idToBlock) do if b2 == ch.Value then bSet[id2] = true break end end
                    end
                end
            end
            local bt = {}
            for tid, bds in pairs(tBinds) do
                if bSet[tid] then
                    for _, bd in ipairs(bds) do table.insert(bt, {tid, bd[1], bd[2]}) end
                end
            end
            local hasT = false
            for _ in pairs(tBinds) do hasT = true break end
            if #bt == 0 and hasT then
                local asgn = {}
                for bn2, _ in pairs(buildData) do
                    for _, e2 in ipairs(buildData[bn2]) do
                        if e2.BindTable then
                            for _, r in ipairs(e2.BindTable) do if r[1] then asgn[r[1]] = true end end
                        end
                    end
                end
                for tid, bds in pairs(tBinds) do
                    if not asgn[tid] then
                        for _, bd in ipairs(bds) do table.insert(bt, {tid, bd[1], bd[2]}) end
                    end
                end
            end
            if #bt > 0 then bEntry.BindTable = bt end
        end
    end
    return buildData
end

local function getBlock(expected, list, used)
    local best, bestDist = nil, math.huge
    local tpos = expected.skyWorldCF.Position
    for _, b in ipairs(list) do
        if b and b.Parent and b:FindFirstChild("PPart") and b.Name == expected.Name and not used[b] then
            local d = (b.PPart.Position - tpos).Magnitude
            if d < bestDist then bestDist = d ; best = b end
        end
    end
    if best then used[best] = true end
    return best
end

local function calcSlots(sz)
    if not sz then return 1 end
    local vol = sz.X * sz.Y * sz.Z
    return math.max(1, math.ceil(vol / 8))
end

local function isRegularBlock(blockName)
    return blockName:sub(-5) == "Block"
end

local function buildDataToFlat(buildData, myZone)
    local regularFlat = {}
    local funcFlat = {}
    local sc = Settings.buildScale
    local off = Vector3.new(Settings.buildOffsetX, Settings.buildOffsetY, Settings.buildOffsetZ)
    local BASE_SKY = Settings.skyHeight or 500

    for blockName, blocks in pairs(buildData) do

        if Settings.excludedBlocks[blockName] then continue end

        local effectiveName = blockName
        if Settings.blockReplacements and Settings.blockReplacements[blockName] then
            effectiveName = Settings.blockReplacements[blockName]
        end
        local regular = isRegularBlock(effectiveName)
        for _, bi in pairs(blocks) do
            local relCF = getBlockCF(bi)
            local pos = (relCF.Position * sc) + off
            local scaledCF = CFrame.new(pos) * (relCF - relCF.Position)
            local worldCF = myZone.CFrame:ToWorldSpace(scaledCF)
            local hasSz = bi.Size ~= nil and bi.Size ~= ""
            local sz = hasSz and (strV3(bi.Size) * sc) or nil
            local hasCo = bi.Col ~= nil and bi.Col ~= ""
            local col = hasCo and strCol(bi.Col) or nil
            local mergedBoolValues, mergedNumberValues = mergePropertyMaps(bi.BoolValues, bi.NumberValues, bi.ASUExtra)
            local entry = {
                Name = effectiveName,
                ID = bi.ID,
                worldCF = worldCF,
                skyWorldCF = nil,
                Relative = myZone,
                Size = sz,
                Col = col,
                hasSz = hasSz,
                hasCo = hasCo,
                isRegular = regular,
                slotCount = regular and calcSlots(sz) or 1,
                Transparency = bi.Transparency,
                Anchored = bi.Anchored,
                CanCollide = bi.CanCollide,
                ShowShadow = bi.ShowShadow,
                BoolValues = mergedBoolValues,
                NumberValues = mergedNumberValues,
                BindTable = bi.BindTable,
                ASUExtra = bi.ASUExtra,
                IsTwoPart = (bi.SecondaryPartPosition ~= nil
                    or (bi.ASUExtra and bi.ASUExtra.SecondaryPartPosition ~= nil)
                    or (bi.SecCFrame ~= nil)
                    or (bi.ASUExtra and bi.ASUExtra.SecCFrame ~= nil)),
                SecondaryWorldCF = nil,
            }
            if entry.IsTwoPart then

                local rawSecCF = bi.SecCFrame or (bi.ASUExtra and bi.ASUExtra.SecCFrame)
                if rawSecCF and type(rawSecCF) == "table" and #rawSecCF >= 12 then

                    local secRelCF = CFrame.new(table.unpack(rawSecCF))
                    local secPos = secRelCF.Position
                    local scaledSecPos = Vector3.new(secPos.X * sc, secPos.Y * sc, secPos.Z * sc) + off
                    local secRotCF = secRelCF - secRelCF.Position
                    local secWorldCF = CFrame.new(scaledSecPos) * secRotCF
                    entry.SecondaryWorldCF = myZone.CFrame:ToWorldSpace(secWorldCF)
                else

                    local rawSecPos = bi.SecondaryPartPosition or (bi.ASUExtra and bi.ASUExtra.SecondaryPartPosition)
                    local secPos = parseNums(rawSecPos)
                    local secPosV = #secPos >= 3 and Vector3.new(secPos[1]*sc, secPos[2]*sc, secPos[3]*sc) + off or Vector3.zero
                    local ppRotCF = relCF - relCF.Position
                    local secCF = CFrame.new(secPosV) * ppRotCF
                    entry.SecondaryWorldCF = myZone.CFrame:ToWorldSpace(secCF)
                end
                entry.SpringProps = {}

                local extra = bi.ASUExtra or {}
                local numV = bi.NumberValues or {}
                local function getProp(k)
                    if bi[k] ~= nil then return bi[k] end
                    if numV[k] ~= nil then return numV[k] end
                    if extra[k] ~= nil then return extra[k] end
                    return nil
                end
                local stiff = getProp("Stiffness")
                if stiff then entry.SpringProps.Stiffness = tostring(stiff) end
                local damp = getProp("Damping")
                if damp then entry.SpringProps.Damping = tostring(damp) end
                local tl = getProp("TargetLength")
                if tl then entry.SpringProps.TargetLength = tostring(tl) end
                local mxl = getProp("MaxLength")
                if mxl then entry.SpringProps.MaxLength = tostring(mxl) end
                local mnl = getProp("MinLength")
                if mnl then entry.SpringProps.MinLength = tostring(mnl) end
                local ln = getProp("Length")
                if ln then entry.SpringProps.Length = tostring(ln) end
                local al = getProp("AngleLimit")
                if al then entry.SpringProps.AngleLimit = tostring(al) end
                local mr = getProp("MatchRotation")
                if mr ~= nil then entry.SpringProps.MatchRotation = mr end
                local sc2 = getProp("ShowConstraint")
                if sc2 ~= nil then entry.SpringProps.ShowConstraint = sc2 end
            end
            if regular then
                regularFlat[#regularFlat+1] = entry
            else
                entry.skyWorldCF = worldCF
                funcFlat[#funcFlat+1] = entry
            end
        end
    end

    local AREA = 25
    local HALF = AREA / 2
    local SPACING = 3
    local COLS = math.floor(AREA / SPACING)
    local PER_LAYER = COLS * COLS
    local cx = myZone.Position.X
    local cz = myZone.Position.Z
    local startY = myZone.Position.Y + 20
    for i, v in ipairs(regularFlat) do
        local idx = i - 1
        local layer = math.floor(idx / PER_LAYER)
        local inLayer = idx % PER_LAYER
        local col = inLayer % COLS
        local row = math.floor(inLayer / COLS)
        local x = cx - HALF + col * SPACING + math.random(0, 1)
        local z = cz - HALF + row * SPACING + math.random(0, 1)
        local y = startY + layer * SPACING
        v.skyWorldCF = CFrame.new(x, y, z) * (v.worldCF - v.worldCF.Position)
    end

    local flat = {}
    for _, v in ipairs(regularFlat) do flat[#flat+1] = v end
    for _, v in ipairs(funcFlat) do flat[#flat+1] = v end
    return flat
end

local function saveToSlot(slot)
    local Event = workspace:FindFirstChild("SaveBoatData")
    if not Event then setStatus("SaveBoatData not found!") ; return false end
    local ok = pcall(function()
        Event:InvokeServer(slot)
    end)
    task.wait(2)
    return ok
end

local function loadFromSlot(slot)
    local Event = workspace:FindFirstChild("LoadBoatData")
    if not Event then setStatus("LoadBoatData not found!") ; return false end
    pcall(function()
        Event:FireServer(slot, 0)
    end)
    task.wait(3)
    return true
end

local function tryInfMerge(slot1, slot2, expectedMinCount, statusCb)
    local folder = BlocksFolder:FindFirstChild(LocalPlayer.Name)
    local beforeCount = folder and #folder:GetChildren() or 0
    local function waitForMergeLoad(minCount, timeoutSec)
        local deadline = tick() + (timeoutSec or 8)
        local lastCount = beforeCount
        local stableFor = 0
        while tick() < deadline and not stopBuild do
            task.wait(0.15)
            local nowCount = folder and #folder:GetChildren() or 0
            if nowCount ~= lastCount then
                lastCount = nowCount
                stableFor = 0
            else
                stableFor = stableFor + 0.15
            end
            if nowCount >= minCount and stableFor >= 0.45 then
                return nowCount
            end
        end
        return folder and #folder:GetChildren() or 0
    end
    for attempt = 1, 5 do
        if stopBuild then return false, "stopped" end
        if statusCb then
            statusCb("INF merge try " .. attempt .. "/5")
        end
        pcall(function() workspace.LoadBoatData:FireServer(slot1, 0) end)
        local slot1Count = waitForMergeLoad(math.max(1, math.floor((expectedMinCount or beforeCount) * 0.35)), 5 + attempt)
        if statusCb then
            statusCb("INF base loaded " .. slot1Count)
        end
        task.wait(0.35 + attempt * 0.1)
        pcall(function() workspace.LoadBoatData:FireServer(slot2, 0) end)
        local loadedCount = waitForMergeLoad(math.max(beforeCount + 1, math.floor((expectedMinCount or 0) * 0.75)), 8 + attempt)
        if statusCb then
            statusCb("INF loaded " .. loadedCount .. " blocks, saving merge")
        end
        pcall(function()
            workspace.SaveBoatData:InvokeServer(slot1)
        end)
        task.wait(1)
        local nowCount = folder and #folder:GetChildren() or 0
        if nowCount >= math.max(beforeCount + 1, math.floor((expectedMinCount or 0) * 0.7)) then
            return true, attempt
        end
    end
    return false, "high ping or merge failed"
end

local BLOCKED_PROPS = {Health=true, LastGlobalTick=true}

local function applyNumberValues(b, numVals, propRF)
    if not numVals or not b or type(numVals) ~= "table" then return end
    for propName, propVal in pairs(numVals) do
        if BLOCKED_PROPS[propName] then continue end
        local remotePropName = propName
        if b.Name == "Piston" and propName == "ExtendLength" then
            remotePropName = "Piston length"
        elseif b.Name == "Piston" and propName == "Speed" then
            remotePropName = "Piston speed"
        elseif b.Name == "Piston" and propName == "LastDirection" then
            remotePropName = "Piston direction"
        elseif b.Name == "Servo" and propName == "Angle" then
            remotePropName = "Servo angle"
        elseif b.Name == "Servo" and propName == "Speed" then
            remotePropName = "Servo speed"
        elseif b.Name == "JetTurbine" and propName == "Speed" then
            remotePropName = "Jet speed"
        elseif b.Name == "JetTurbine" and (propName == "Force" or propName == "JetForce" or propName == "MaxForce") then
            remotePropName = "Jet force"
        elseif b.Name == "Motor" and propName == "MaxSpeed" then
            remotePropName = "Max speed"
        elseif b.Name == "Motor" and propName == "WheelTorque" then
            remotePropName = "Wheel torque"
        elseif b.Name == "Motor" and propName == "ReverseSpin" then
            remotePropName = "Reverse spin"
        end
        local numericVal = tonumber(propVal)
        if numericVal == nil and type(propVal) == "boolean" then
            numericVal = propVal and 1 or 0
        end
        if numericVal == nil then numericVal = 0 end
        pcall(function()
            for _, target in ipairs({b, b.PPart}) do
                if target then
                    local pv = target:FindFirstChild(propName) or target:FindFirstChild(propName, true)
                    if not pv and remotePropName ~= propName then
                        pv = target:FindFirstChild(remotePropName) or target:FindFirstChild(remotePropName, true)
                    end
                    if pv then
                        if pv:IsA("NumberValue") or pv:IsA("IntValue") then
                            pv.Value = numericVal
                        elseif pv:IsA("BoolValue") then
                            pv.Value = numericVal ~= 0
                        end
                    end
                end
            end
        end)
        if propRF then
            task.spawn(function() pcall(function() propRF:InvokeServer(remotePropName, {b}, tostring(numericVal)) end) end)
        end
    end
end

local PROP_TIMEOUT = 0.5
local function invokeWithTimeout(rf, args, timeout)
    if not rf then return false end
    local done = false
    local ok = false
    task.spawn(function()
        ok = pcall(function() rf:InvokeServer(unpack(args)) end)
        done = true
    end)
    local t0 = tick()
    while not done and tick() - t0 < (timeout or PROP_TIMEOUT) do
        task.wait(0.05)
        if stopBuild then return false end
    end
    return ok
end

local function firePropertyRF(propRF, ...)
    if not propRF then return false end
    local args = {...}
    task.spawn(function() pcall(function() propRF:InvokeServer(unpack(args)) end) end)
    return true
end

local function activateButtonBlock(buttonBlock)
    if not buttonBlock then return false end
    local ok, blockFunctions = pcall(function()
        local scripts = ReplicatedStorage:FindFirstChild("Scripts")
        local module = scripts and scripts:FindFirstChild("BlockFunctions")
        if module then return require(module) end
        return nil
    end)
    if not ok or not blockFunctions or type(blockFunctions.addBulkToRunQueue) ~= "function" then return false end
    local blockData = {
        buttonBlock,
        true,
        true,
        false,
        LocalPlayer.Character or Character,
        true,
        true,
        true,
        true,
        true
    }
    return pcall(function()
        blockFunctions.addBulkToRunQueue(LocalPlayer.UserId, {blockData})
    end)
end

local function activatePistonViaQueue(pistonBlock)
    if not pistonBlock then return false end
    local currentChar = LocalPlayer.Character or Character
    if not currentChar then return false end
    local inputLocalScript = ReplicatedStorage:FindFirstChild("InputLocalScript")
    if not inputLocalScript then return false end
    local queueRF = inputLocalScript:FindFirstChild("QueueBlocksRequest")
    if not queueRF then return false end
    local args = {
        {
            {
                pistonBlock,
                true,
                false,
                false,
                currentChar,
                true,
                false,
                true,
                true,
                false
            }
        }
    }
    return pcall(function()
        queueRF:FireServer(unpack(args))
    end)
end

local function getPropertiesRF()
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    local propTool = (Character and Character:FindFirstChild("PropertiesTool")) or (backpack and backpack:FindFirstChild("PropertiesTool"))
    if not propTool then
        equipAllTools()
        propTool = (Character and Character:FindFirstChild("PropertiesTool")) or (backpack and backpack:FindFirstChild("PropertiesTool"))
    end
    return propTool and propTool:FindFirstChild("SetPropertieRF")
end

local function getEntrySizeVector(v)
    if not v then return nil end
    local raw = v.Size or v.size
    if typeof(raw) == "Vector3" then return raw end
    if type(raw) == "table" then
        return Vector3.new(tonumber(raw[1]) or 0, tonumber(raw[2]) or 0, tonumber(raw[3]) or 0)
    end
    if type(raw) == "string" then
        local nums = parseNums(raw)
        if #nums >= 3 then return Vector3.new(nums[1], nums[2], nums[3]) end
    end
    return nil
end

local function shouldLegacySwitch(entry)
    if not entry or not entry.block or not entry.v or entry.block.Name ~= "Switch" then return false end
    local bv = entry.v.BoolValues
    if bv and bv.Legacy == false then return false end
    return true
end

local function applyLegacySwitches(styledList, propRF)
    propRF = propRF or getPropertiesRF()
    if not propRF then return 0 end
    local targets = {}
    for _, entry in ipairs(styledList or {}) do
        if shouldLegacySwitch(entry) then
            targets[#targets + 1] = entry.block
        end
    end
    if #targets == 0 then return 0 end
    for i = 1, #targets, 40 do
        local chunk = {}
        for j = i, math.min(i + 39, #targets) do chunk[#chunk + 1] = targets[j] end
        pcall(function() propRF:InvokeServer("Legacy", chunk) end)
        task.wait(0.05)
    end
    return #targets
end

local function applyBoolValues(b, boolVals, propRF)
    if not boolVals or not b or type(boolVals) ~= "table" then return end
    for propName, propVal in pairs(boolVals) do
        if BLOCKED_PROPS[propName] then continue end
        local desired = propVal == true
        local currentValue = nil
        pcall(function()
            for _, target in ipairs({b, b.PPart}) do
                if target then
                    local pv = target:FindFirstChild(propName) or target:FindFirstChild(propName, true)
                    if pv and pv:IsA("BoolValue") then
                        if currentValue == nil then currentValue = pv.Value end
                    end
                end
            end
        end)
        local needToggle = false
        if currentValue == nil then
            needToggle = desired
        else
            needToggle = currentValue ~= desired
        end
        if propRF and needToggle then
            firePropertyRF(propRF, propName, {b})
        end
    end
end

local TRANSPARENCY_STEPS = {0, 0.25, 0.5, 0.75, 1}

local function getTransparencyStepIndex(value)
    local bestIndex = 1
    local bestDiff = math.huge
    for i, stepValue in ipairs(TRANSPARENCY_STEPS) do
        local diff = math.abs((tonumber(value) or 0) - stepValue)
        if diff < bestDiff then
            bestDiff = diff
            bestIndex = i
        end
    end
    return bestIndex
end

local function setBlockProperties(b, v, propRF, skipTransp)
    if not b or not b:FindFirstChild("PPart") then return end
    if not propRF then
        equipAllTools()
        local propTool = Character:FindFirstChild("PropertiesTool")
        propRF = propTool and propTool:FindFirstChild("SetPropertieRF")
        if not propRF then return end
    end
    if v.Transparency ~= nil and not skipTransp then
        local transparency = tonumber(v.Transparency)
        if transparency then
            transparency = math.clamp(transparency, 0, 1)
            if propRF then
                firePropertyRF(propRF, "Transparency", {b}, tostring(math.floor(transparency * 100 + 0.5)))
            end
        end
    end
    if v.CanCollide ~= nil and not isSpecialPropBlock(b.Name) then
        if propRF and b.PPart.CanCollide ~= (v.CanCollide == true) then firePropertyRF(propRF, "Collision", {b}) end
    end

    if v.ShowShadow ~= nil then
        if propRF and b.PPart.CastShadow ~= (v.ShowShadow == true) then firePropertyRF(propRF, "Cast shadow", {b}) end
    end
    applyBoolValues(b, v.BoolValues, propRF)
    applyNumberValues(b, v.NumberValues, propRF)
end


local function isSpecialPropBlock(name)
    return name == "Piston" or name == "Hinge" or name == "Bar" or name == "Rope" or name == "Spring"
end


local function isMoveWeldBlock(name)
    return name == "Piston" or name == "Hinge"
end


local function isRotateConstraintBlock(name)
    return name == "Bar" or name == "Rope" or name == "Spring"
end

local function pasteBuild(buildData, statusCb)
    if not buildData or isBuilding then return false end
    isBuilding = true
    stopBuild = false

    local teamLeaderName = nil
    local isTeamLeader = false
    pcall(function()
        local myTeam = LocalPlayer.Team
        if myTeam then
            local tlObj = myTeam:FindFirstChild("TeamLeader")
            if tlObj then
                teamLeaderName = tlObj.Value and tostring(tlObj.Value) or tostring(tlObj.Value)
                if not teamLeaderName or teamLeaderName == "" then
                    pcall(function()
                        teamLeaderName = tlObj.Value.Name
                    end)
                end
                isTeamLeader = (teamLeaderName == LocalPlayer.Name)
            end
        end
    end)
    shareBlocksOriginal = false
    pcall(function()
        local settingsFolder = LocalPlayer:FindFirstChild("Settings")
        if settingsFolder then
            local sbVal = settingsFolder:FindFirstChild("ShareBlocks")
            if sbVal then
                shareBlocksOriginal = (sbVal.Value == true)
            end
        end
    end)
    local shareBlocks = shareBlocksOriginal
    pcall(function()
        local settingsFolder = LocalPlayer:FindFirstChild("Settings")
        if settingsFolder then
            local sbVal = settingsFolder:FindFirstChild("ShareBlocks")
            if sbVal then
                shareBlocks = (sbVal.Value == true)
            end
        end
    end)
    if isTeamLeader and not shareBlocks then
        shareBlocks = true
        pcall(function()
            local settingsFolder = LocalPlayer:FindFirstChild("Settings")
            if not settingsFolder then
                settingsFolder = Instance.new("Folder")
                settingsFolder.Name = "Settings"
                settingsFolder.Parent = LocalPlayer
            end
            local sbVal = settingsFolder:FindFirstChild("ShareBlocks")
            if not sbVal then
                sbVal = Instance.new("BoolValue")
                sbVal.Name = "ShareBlocks"
                sbVal.Parent = settingsFolder
            end
            sbVal.Value = true
        end)
    end
    local buildTargetName = LocalPlayer.Name
    if not isTeamLeader and shareBlocks and teamLeaderName then
        buildTargetName = teamLeaderName
    end

    buildStartTime = tick()

    local myZone = getPlayerZone(LocalPlayer)
    if not myZone then isBuilding = false ; return false end

    local flat = buildDataToFlat(buildData, myZone)
    local total = #flat
    buildTotalBlocks = total
    if total == 0 then isBuilding = false ; return false end

    local folder = BlocksFolder:FindFirstChild(buildTargetName)
    if not folder then
        folder = Instance.new("Folder")
        folder.Name = buildTargetName
        folder.Parent = BlocksFolder
    end

    equipAllTools()
    local placeTool = Character:FindFirstChild("BuildingTool")
    local scaleTool = Character:FindFirstChild("ScalingTool")
    local paintTool = Character:FindFirstChild("PaintingTool")
    local deleteTool = Character:FindFirstChild("DeleteTool") or Character:FindFirstChild("DeletingTool")
    local bindTool = Character:FindFirstChild("BindTool")

    local function updProg(msg, pct)
        if statusCb then statusCb(msg, pct) end
        local safePct = math.clamp(tonumber(pct) or 0, 0, 100)
        local pctInt = math.floor(safePct + 0.5)
        if StatusLabelRef then
            StatusLabelRef.Text = "  " .. tostring(msg or "")
        end
        if ProgressBarFillRef then
            TweenService:Create(ProgressBarFillRef, TweenInfo.new(0.12), {Size = UDim2.new(safePct / 100, 0, 1, 0)}):Play()
        end
        if DupePercentLabelRef then
            DupePercentLabelRef.Text = pctInt .. "%"
        end
        if DupeInfoLabelRef then
            local remaining = math.max(0, math.floor(buildTotalBlocks * (100 - safePct) / 100))
            local elapsed = buildStartTime > 0 and (tick() - buildStartTime) or 0
            local etaStr = ""
            if safePct > 2 and elapsed > 1 then
                local totalEst = elapsed / (safePct / 100)
                local eta = math.max(0, totalEst - elapsed)
                if eta >= 60 then
                    etaStr = string.format(" | ETA %dm %ds", math.floor(eta/60), math.floor(eta % 60))
                else
                    etaStr = string.format(" | ETA %ds", math.floor(eta))
                end
            end
            DupeInfoLabelRef.Text = "  " .. tostring(msg or "") .. "  |  " .. remaining .. " left" .. etaStr
        end
        if InfProgressLabelRef and InfProgressFillRef then
            local isInf = tostring(msg or ""):sub(1, 3) == "INF"
            InfProgressLabelRef.Visible = isInf
            InfProgressFillRef.Parent.Visible = isInf
            if isInf then
                InfProgressLabelRef.Text = string.format("INF %d%%", pctInt)
                TweenService:Create(InfProgressFillRef, TweenInfo.new(0.08), {Size = UDim2.new(safePct / 100, 0, 1, 0)}):Play()
            end
        end
    end

    local function waitForN(minN, maxWait)
        local t0 = tick()
        local lastN, sameFor = 0, 0
        local stableNeed = minN > 200 and 0.25 or 0.4
        repeat
            task.wait(0.12)
            local n = #folder:GetChildren()
            if n == lastN then sameFor = sameFor + 0.12 else sameFor = 0 end
            lastN = n
        until (lastN >= minN and sameFor >= stableNeed) or tick()-t0 > maxWait or stopBuild
        return lastN
    end

    local function findNearest(name, skyPos, list, used)
        local best, bestD = nil, math.huge
        for _, b in ipairs(list) do
            if b and b.Parent and not used[b] and b.Name == name then
                local ppart = b:FindFirstChild("PPart")
                if ppart and ppart.Parent then
                    local d = (ppart.Position - skyPos).Magnitude
                    if d < bestD then bestD = d ; best = b end
                end
            end
        end
        if best then used[best] = true end
        return best
    end

    local placeRF = placeTool and placeTool:FindFirstChild("RF")
    local scaleRF = scaleTool and scaleTool:FindFirstChild("RF")

    local function forceAnchorBlocks(baseList)
        if not propertiesRF then return end
        local needAnchor = {}
        for _, b in ipairs(baseList) do
            if b and b:FindFirstChild("PPart") and not b.PPart.Anchored then
                needAnchor[#needAnchor+1] = b
            end
        end
        if #needAnchor > 0 then
            local batchSize = 50
            for i = 1, #needAnchor, batchSize do
                local batch = {}
                for j = i, math.min(i + batchSize - 1, #needAnchor) do
                    batch[#batch+1] = needAnchor[j]
                end
                pcall(function() propertiesRF:InvokeServer("Anchored", batch) end)
                if i + batchSize <= #needAnchor then task.wait(0.03) end
            end
        end
    end
    local paintRF = (paintTool and paintTool:FindFirstChild("RF"))
        or (LocalPlayer.Backpack:FindFirstChild("PaintingTool") and LocalPlayer.Backpack.PaintingTool:FindFirstChild("RF"))
    local deleteRF = deleteTool and deleteTool:FindFirstChild("RF")
    local propertiesTool = Character:FindFirstChild("PropertiesTool")
    local propertiesRF = propertiesTool and propertiesTool:FindFirstChild("SetPropertieRF")
    local bindRF = bindTool and bindTool:FindFirstChild("RF")
    local placedById = {}

    local function fireScale(b, sz, cf)
        if not scaleRF or not b then return end
        task.spawn(function()
            pcall(function() scaleRF:InvokeServer(b, sz, cf) end)
        end)
    end

    local function fastPlace(v)
        if not placeRF then return end
        task.spawn(function()
            pcall(function()
                if v.IsTwoPart and v.SecondaryWorldCF then
                    placeRF:InvokeServer(
                        v.Name, getBlockID(v.Name), v.Relative,
                        v.Relative.CFrame:ToObjectSpace(v.skyWorldCF),
                        true,
                        v.SecondaryWorldCF,
                        v.skyWorldCF
                    )
                else
                    placeRF:InvokeServer(
                        v.Name, getBlockID(v.Name), v.Relative,
                        v.Relative.CFrame:ToObjectSpace(v.skyWorldCF),
                        true
                    )
                end
            end)
        end)
    end

    local function fastRescale(b, cf, sz)
        if not b or not b:FindFirstChild("PPart") then return false end
        pcall(function()
            b.PPart.Size = sz
            b.PPart.CFrame = cf
        end)
        if scaleRF then
            fireScale(b, sz, cf)
            return true
        end
        return false
    end

    local function batchPaintSync(pairs_list)
        if not paintRF or #pairs_list == 0 then return end
        task.spawn(function()
            pcall(function() paintRF:InvokeServer(pairs_list) end)
            task.wait(0.5)
            for _, pair in ipairs(pairs_list) do
                local b = pair[1]
                local col = pair[2]
                if b and b.Parent and col then
                    local c3 = strCol(tostring(col))
                    pcall(function()
                        for _, desc in ipairs(b:GetDescendants()) do
                            if desc:IsA("Decal") then
                                desc.Color3 = c3
                            end
                        end
                    end)
                end
            end
        end)
    end

    local function fastMove(b, cf)
        if not b or not b:FindFirstChild("PPart") then return end
        pcall(function() b.PPart.CFrame = cf end)
        if scaleRF then
            fireScale(b, b.PPart.Size, cf)
        end
    end

        local PLACE_BATCH = 60
    local STYLE_BATCH = 90
    local MOVE_BATCH = 90

    local function runPlacePhase(subset, label, p0, p1)
            local BATCH = 60
            local delayTime = Settings.buildSpeed > 0 and Settings.buildSpeed * 0.01 or 0
            for i = 1, #subset do
                    if stopBuild then break end
                    fastPlace(subset[i])

                    if delayTime > 0 and i % BATCH == 0 then
                        task.wait(delayTime)
                    end
            end
            updProg(label .. #subset .. " placed", p1)
    end

    local function runStylePhase(subset, baseList, used, p0, p1)
                local regularStyled = {}
                local funcStyled = {}
                local paintQueue = {}

                local nameGroups = {}
                for _, blk in ipairs(baseList) do
                        if blk and blk.Parent and blk.Name then
                                nameGroups[blk.Name] = nameGroups[blk.Name] or {}
                                table.insert(nameGroups[blk.Name], blk)
                        end
                end
                local nameIdx = {}

                for i = 1, #subset do
                        if stopBuild then break end
                        local v = subset[i]
                        local b = nil

                        nameIdx[v.Name] = (nameIdx[v.Name] or 0) + 1
                        local group = nameGroups[v.Name]
                        if group then
                                for j = nameIdx[v.Name], #group do
                                        local candidate = group[j]
                                        if candidate and candidate.Parent and not used[candidate] and candidate:FindFirstChild("PPart") then
                                                b = candidate
                                                used[b] = true
                                                nameIdx[v.Name] = j
                                                break
                                        end
                                end
                        end

                        if not b then
                                b = findNearest(v.Name, v.skyWorldCF.Position, baseList, used)
                        end
                        if b and b:FindFirstChild("PPart") then
                                if v.ID ~= nil then
                                        placedById[v.ID] = b
                                        placedById[tostring(v.ID)] = b
                                end
                                if v.isRegular then
                                        if v.hasCo and v.Col then
                                                paintQueue[#paintQueue+1] = {b, v.Col}
                                        end
                                        if v.hasSz and v.Size then
                                                fastRescale(b, v.skyWorldCF, v.Size)
                                                regularStyled[#regularStyled+1] = {block=b, worldCF=v.worldCF, v=v}
                                        else
                                                pcall(function() b.PPart.CFrame = v.skyWorldCF end)
                                                regularStyled[#regularStyled+1] = {block=b, worldCF=v.worldCF, v=v}
                                        end
                                else
                                        funcStyled[#funcStyled+1] = {block=b, worldCF=v.worldCF, v=v}
                                end
                        end
                end

                updProg("Painting " .. #paintQueue .. " blocks...", p0 + 70)
                if #paintQueue > 0 then
                        batchPaintSync(paintQueue)
                end

                local allStyled = {}
                for _, e in ipairs(regularStyled) do allStyled[#allStyled+1] = e end
                for _, e in ipairs(funcStyled) do allStyled[#allStyled+1] = e end

                return allStyled
        end

    local function applyBindTables(styledList, p0, p1)

        if not bindTool or not bindTool.Parent then
            bindTool = Character:FindFirstChild("BindTool") or LocalPlayer.Backpack:FindFirstChild("BindTool")
            if bindTool and bindTool.Parent ~= Character then bindTool.Parent = Character ; task.wait(0.05) end
            bindRF = bindTool and bindTool:FindFirstChild("RF")
        end
        if not bindRF then updProg("No BindTool RF, skipping binds", p0) return end
        local unbindRF = bindTool and bindTool:FindFirstChild("UnbindRF")
        local hasAnyBinds = false
        for _, e in ipairs(styledList) do
            if e.v and type(e.v.BindTable) == "table" then
                local bt = e.v.BindTable
                local cnt = 0
                for _ in pairs(bt) do cnt = cnt + 1 end
                if cnt > 0 then hasAnyBinds = true break end
            end
        end
        if not hasAnyBinds then return end
        do
            local unbound = {}
            for _, entry in ipairs(styledList) do
                if stopBuild then break end
                local bt = entry.v and entry.v.BindTable
                if type(bt) == "table" then
                    local sb = entry.block
                    if sb and unbindRF and not unbound[sb] then
                        unbound[sb] = true
                        invokeWithTimeout(unbindRF, {{sb}})
                    end
                end
            end
        end
        local done = 0
        local total = 0
        for _, entry in ipairs(styledList) do
            local bt = entry.v and entry.v.BindTable
            if type(bt) == "table" then
                for _ in pairs(bt) do total = total + 1 end
            end
        end
        for i, entry in ipairs(styledList) do
            if stopBuild then break end
            local bindTable = entry.v and entry.v.BindTable
            if type(bindTable) ~= "table" then continue end
            local seatBlock = entry.block
            if not seatBlock then continue end

            local isSwitchType = seatBlock.Name:find("Switch") ~= nil
                or seatBlock.Name:find("Delay") ~= nil
                or seatBlock.Name:find("Sensor") ~= nil
            local actionMap = {}
            for _, bindRow in pairs(bindTable) do
                if type(bindRow) ~= "table" then continue end
                local targetBlock = placedById[bindRow[1]] or placedById[tostring(bindRow[1])]
                local bindName = bindRow[2]
                local bindValue = tonumber(bindRow[3]) or bindRow[3]
                if not targetBlock or not bindName then continue end
                local bindObject = targetBlock:FindFirstChild(bindName) or targetBlock:FindFirstChild(bindName, true)
                if not bindObject then

                    pcall(function() bindObject = targetBlock:WaitForChild(bindName, 3) end)
                    if not bindObject then
                        pcall(function() bindObject = targetBlock:FindFirstChild(bindName, true) end)
                    end
                end
                if not bindObject then continue end

                local actionName
                if bindName == "BindUp" then
                    actionName = "Push"
                elseif bindName == "BindDown" then
                    actionName = "Pull"
                elseif bindName == "BindFire" or bindName == "BindActivate" then
                    actionName = "Activate"
                else
                    actionName = bindName:gsub("^Bind", "")
                end
                if not actionMap[actionName] then
                    actionMap[actionName] = {objs = {}, keys = {}}
                end
                table.insert(actionMap[actionName].objs, bindObject)
                table.insert(actionMap[actionName].keys, bindValue)
                done = done + 1
            end
            for actName, group in pairs(actionMap) do
                local firstArg = {[actName] = group.objs}
                local keyVal = #group.keys == 1 and group.keys[1] or group.keys

                local thirdArg = isSwitchType and {} or {[actName] = keyVal}
                if isSwitchType then

                    task.spawn(function()
                        pcall(function() bindRF:InvokeServer(firstArg, seatBlock, thirdArg, false) end)
                    end)
                else
                    invokeWithTimeout(bindRF, {firstArg, seatBlock, thirdArg, false})
                end
            end
            if i % 5 == 0 then task.wait() end
        end
        if done > 0 then updProg("Bound " .. done .. " controls", p1) end
    end

    local function applyPropertiesPhase(styledList, p0, p1, skipTransp)
        if not propertiesRF then updProg("No PropertiesTool RF, skipping props", p0) return end

        local anchorOn, anchorOff, ccOn = {}, {}, {}
        for idx, entry in ipairs(styledList) do
            if not entry.block or not entry.v or not entry.block:FindFirstChild("PPart") then continue end
            local v = entry.v
            local b = entry.block
            local isBlock = b.Name:sub(-5) == "Block"
            local pp = b.PPart

            local wantAnchor
            if v.Anchored ~= nil then
                wantAnchor = v.Anchored == true
            elseif not isBlock then
                wantAnchor = true
            end
            if wantAnchor ~= nil and pp.Anchored ~= wantAnchor then
                if wantAnchor then anchorOn[#anchorOn+1] = b else anchorOff[#anchorOff+1] = b end
            end

            if not pp.CanCollide and not isSpecialPropBlock(b.Name) then
                ccOn[#ccOn+1] = b
            end
        end

        if #anchorOn > 0 then pcall(function() propertiesRF:InvokeServer("Anchored", anchorOn) end) end
        task.wait(0.05)
        if #anchorOff > 0 then pcall(function() propertiesRF:InvokeServer("Anchored", anchorOff) end) end
        task.wait(0.05)

        if #ccOn > 0 then pcall(function() propertiesRF:InvokeServer("Collision", ccOn) end) end
        task.wait(0.1)

        local batchGroups = {}
        for _, entry in ipairs(styledList) do
            if not entry.block or not entry.v or not entry.block:FindFirstChild("PPart") then continue end
            local v = entry.v
            local b = entry.block
            if not skipTransp and v.Transparency ~= nil and not isSpecialPropBlock(b.Name) then
                local t = tonumber(v.Transparency)
                if t then
                    t = math.clamp(t, 0, 1)
                    local tStr = tostring(math.floor(t * 100 + 0.5))
                    batchGroups["Transparency"] = batchGroups["Transparency"] or {}
                    batchGroups["Transparency"][tStr] = batchGroups["Transparency"][tStr] or {}
                    table.insert(batchGroups["Transparency"][tStr], b)
                end
            end
            if v.ShowShadow ~= nil then
                local val = v.ShowShadow == true
                local key = val and "1" or "0"
                batchGroups["Cast shadow"] = batchGroups["Cast shadow"] or {}
                batchGroups["Cast shadow"][key] = batchGroups["Cast shadow"][key] or {}
                table.insert(batchGroups["Cast shadow"][key], b)
            end
        end
        local batchDone = 0
        for propName, groups in pairs(batchGroups) do
            for valKey, blocks in pairs(groups) do
                if stopBuild then break end
                if #blocks > 0 then
                    pcall(function() propertiesRF:InvokeServer(propName, blocks, valKey) end)
                    batchDone = batchDone + 1
                    if batchDone % 10 == 0 then task.wait(0.05) end
                end
            end
        end
        task.wait(0.1)

        local pbTotal = 0
        for _, entry in ipairs(styledList) do
            if not entry.block or not entry.v then continue end
            local v = entry.v
            local isSwitch = entry.block.Name == "Switch"
            if isSwitch or (v.NumberValues and next(v.NumberValues)) or (v.BoolValues and next(v.BoolValues)) or (v.SpringProps and next(v.SpringProps)) then
                pbTotal = pbTotal + 1
            end
        end
        local pbDone = 0
        local numValBatch = {}
        for _, entry in ipairs(styledList) do
            if stopBuild then break end
            if not entry.block or not entry.v then continue end
            local v = entry.v
            local b = entry.block
            if b.Name == "Switch" and shouldLegacySwitch(entry) then
                v.BoolValues = v.BoolValues or {}
                if v.BoolValues.Legacy == nil then
                    v.BoolValues.Legacy = true
                end
            end

            if not ((v.NumberValues and next(v.NumberValues)) or (v.BoolValues and next(v.BoolValues)) or (v.SpringProps and next(v.SpringProps))) then continue end
            applyBoolValues(b, v.BoolValues, propertiesRF)
            applyNumberValues(b, v.NumberValues, propertiesRF)
            if v.SpringProps and next(v.SpringProps) then
                local sp = v.SpringProps
                if sp.Stiffness then firePropertyRF(propertiesRF, "Stiffness", {b}, sp.Stiffness) end
                if sp.Damping then firePropertyRF(propertiesRF, "Damping", {b}, sp.Damping) end
                if sp.TargetLength then firePropertyRF(propertiesRF, "Target length", {b}, sp.TargetLength) end
                if sp.MaxLength then firePropertyRF(propertiesRF, "Max length", {b}, sp.MaxLength) end
                if sp.MinLength then firePropertyRF(propertiesRF, "Min length", {b}, sp.MinLength) end
                if sp.Length then firePropertyRF(propertiesRF, "Length", {b}, sp.Length) end
                if sp.AngleLimit then firePropertyRF(propertiesRF, "Angle limit", {b}, sp.AngleLimit) end
                if sp.MatchRotation then firePropertyRF(propertiesRF, "Match rotation", {b}) end
                if sp.ShowConstraint then firePropertyRF(propertiesRF, "Show constraint", {b}) end
            end
            pbDone = pbDone + 1
            if pbDone % 30 == 0 then task.wait() end
        end
        updProg("Properties done", p1)
    end

    local function runMovePhase(styledList, p0, p1)
            local moveOpRF = nil
            local trowelTool = Character:FindFirstChild("TrowelTool") or LocalPlayer.Backpack:FindFirstChild("TrowelTool")
            if trowelTool then
                    moveOpRF = trowelTool:FindFirstChild("OperationRF")
                    if trowelTool.Parent ~= Character then trowelTool.Parent = Character ; task.wait(0.05) end
            end
            for i, entry in ipairs(styledList) do
                    if stopBuild then break end
                    local b = entry.block
                    if b and b:FindFirstChild("PPart") then
                            local cf = entry.worldCF
                            local isBlock = b.Name:sub(-5) == "Block"
                            pcall(function() b.PPart.CFrame = cf end)
                            if isBlock and scaleRF then
                                    task.spawn(function()
                                            pcall(function() scaleRF:InvokeServer(b, b.PPart.Size, cf) end)
                                    end)
                            elseif not isBlock and moveOpRF then
                                    task.spawn(function()
                                            pcall(function() moveOpRF:InvokeServer({b}, cf, cf, "Move") end)
                                    end)
                            end
                    end
            end
            updProg("Moved " .. #styledList .. " blocks", p1)
    end

    local function deleteBlock(b)
        if not b or not b.Parent then return false end
        local ok = false
        if deleteRF then
            ok = pcall(function() deleteRF:InvokeServer(b) end) or ok
            ok = pcall(function() deleteRF:InvokeServer({b}) end) or ok
            ok = pcall(function() deleteRF:InvokeServer({{b}}) end) or ok
        end
        if not ok then
            pcall(function() b:Destroy() end)
        end
        return true
    end

    local function deletePlacedEntries(entries, p0, p1)
        local baseList = folder:GetChildren()
        local used = {}
        local deleted = 0
        for i, v in ipairs(entries) do
            if stopBuild then break end
            local b = findNearest(v.Name, v.skyWorldCF.Position, baseList, used)
            if b and deleteBlock(b) then
                deleted = deleted + 1
            end
            if i % PLACE_BATCH == 0 then
                task.wait()
                updProg("INF deleting saved blocks " .. i .. "/" .. #entries, p0 + math.floor(i / #entries * (p1 - p0)))
            end
        end
        task.wait(0.15)
        return deleted
    end

    if Settings.infBlockEnabled then
        equipAllTools()
        placeTool = Character:FindFirstChild("BuildingTool")
        scaleTool = Character:FindFirstChild("ScalingTool")
        placeRF = placeTool and placeTool:FindFirstChild("RF")
        scaleRF = scaleTool and scaleTool:FindFirstChild("RF")

        local needed = {}
        for _, v in ipairs(flat) do
            if v.isRegular and isRegularBlock(v.Name) then
                needed[v.Name] = (needed[v.Name] or 0) + (v.slotCount or 1)
            end
        end

        local missingList = {}
        for name, need in pairs(needed) do
            local have = getBlockID(name)
            if have < need then
                local missing = need - have
                missingList[#missingList+1] = {
                    Name = name,
                    Need = need,
                    Missing = missing,
                    ScaleCalls = math.max(1, math.ceil(missing / 4) + 1)
                }
            end
        end
        local totalInfCalls = 0
        for _, miss in ipairs(missingList) do
            totalInfCalls = totalInfCalls + (miss.ScaleCalls or 0)
        end
        local doneInfCalls = 0

        local function waitForBlockByName(name, beforeMap, maxWait)
            for _, b in ipairs(folder:GetChildren()) do
                if b.Name == name and not beforeMap[b] then
                    return b
                end
            end
            RunService.Heartbeat:Wait()
            for _, b in ipairs(folder:GetChildren()) do
                if b.Name == name and not beforeMap[b] then
                    return b
                end
            end
            return folder:FindFirstChild(name)
        end

        local myZone = getPlayerZone(LocalPlayer)
        local dupeSize = Vector3.new(251.79899597168, 1.1754999560161e-38, 298.99996948242)
        if myZone and math.abs(myZone.CFrame.LookVector.X) > 0.5 then
            dupeSize = Vector3.new(298.99996948242, 1.1754999560161e-38, 251.79899597168)
        end
        local dupeCFrame = myZone and CFrame.new(myZone.Position.X, -3.0000000054978e38, myZone.Position.Z, 1, 0, 0, 0, 1, 0, 0, 0, 1)
        for mi, miss in ipairs(missingList) do
            if stopBuild then break end
            if myZone and placeRF and scaleRF then
                local p0 = math.floor((mi - 1) / math.max(#missingList, 1) * 34)
                updProg("INF filling " .. miss.Name .. " +" .. miss.Missing, p0)

                local beforeMap = {}
                for _, b in ipairs(folder:GetChildren()) do beforeMap[b] = true end

                pcall(function()
                    placeRF:InvokeServer(
                        miss.Name,
                        getBlockID(miss.Name),
                        myZone,
                        CFrame.new(0, -3.0000000054978e38, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1),
                        true
                    )
                end)

                local dupBlock = waitForBlockByName(miss.Name, beforeMap, 4)
                if dupBlock then
                    local scaleCalls = miss.ScaleCalls or math.max(1, math.ceil(miss.Missing / 4) + 1)
                    for n = 1, scaleCalls do
                        if stopBuild then break end
                        task.spawn(function()
                            pcall(function()
                                scaleRF:InvokeServer(dupBlock, dupeSize, dupeCFrame)
                            end)
                        end)
                    end
                    doneInfCalls = doneInfCalls + scaleCalls
                    local infPct = totalInfCalls > 0 and math.floor(doneInfCalls / totalInfCalls * 35) or 35
                    updProg("INF filling " .. miss.Name .. " " .. doneInfCalls .. "/" .. totalInfCalls, infPct)
                end
            end
        end

        updProg("INF ready, building", 35)
    end

    equipAllTools()
    placeTool = Character:FindFirstChild("BuildingTool")
    scaleTool = Character:FindFirstChild("ScalingTool")
    paintTool = Character:FindFirstChild("PaintingTool")
    deleteTool = Character:FindFirstChild("DeleteTool") or Character:FindFirstChild("DeletingTool")
    propertiesTool = Character:FindFirstChild("PropertiesTool")
    bindTool = Character:FindFirstChild("BindTool")
    placeRF = placeTool and placeTool:FindFirstChild("RF")
    scaleRF = scaleTool and scaleTool:FindFirstChild("RF")
    paintRF = (paintTool and paintTool:FindFirstChild("RF"))
        or (LocalPlayer.Backpack:FindFirstChild("PaintingTool") and LocalPlayer.Backpack.PaintingTool:FindFirstChild("RF"))
    deleteRF = deleteTool and deleteTool:FindFirstChild("RF")
    propertiesRF = propertiesTool and propertiesTool:FindFirstChild("SetPropertieRF")
    bindRF = bindTool and bindTool:FindFirstChild("RF")
    local pistonFlat, restFlat = {}, {}
    for _, v in ipairs(flat) do
        if v.Name == "Piston" then
            pistonFlat[#pistonFlat + 1] = v
        else
            restFlat[#restFlat + 1] = v
        end
    end

    local allStyled, styledPistons = {}, {}
    if #pistonFlat > 0 and not stopBuild then
        updProg("Placing " .. #pistonFlat .. " pistons...", 0)
        runPlacePhase(pistonFlat, "Pistons ", 0, 15)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        updProg("Waiting for pistons...", 16)
        waitForN(math.floor(#pistonFlat * 0.9), math.max(3, #pistonFlat * 0.05))
        styledPistons = runStylePhase(pistonFlat, folder:GetChildren(), {}, 18, 30)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        updProg("Moving " .. #styledPistons .. " pistons...", 30)
        runMovePhase(styledPistons, 30, 40)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        updProg("Piston properties (no transparency)...", 40)
        applyPropertiesPhase(styledPistons, 40, 50, true)

        local pistonsToActivate = {}
        for _, entry in ipairs(styledPistons) do
            local ld = entry.v and entry.v.NumberValues and entry.v.NumberValues.LastDirection
            if entry.block and entry.block:FindFirstChild("PPart") and ld == 1 then
                pistonsToActivate[#pistonsToActivate + 1] = {block = entry.block, worldCF = entry.worldCF}
            end
        end



        if #pistonsToActivate >= 2 and not stopBuild then
            updProg("Pre-positioning pistons (moving up 10k studs)...", 48)
            local trowelTool = Character:FindFirstChild("TrowelTool") or LocalPlayer.Backpack:FindFirstChild("TrowelTool")
            if trowelTool then
                if trowelTool.Parent ~= Character then trowelTool.Parent = Character ; task.wait(0.05) end
                local moveOpRF = trowelTool:FindFirstChild("OperationRF")
                if moveOpRF then

                    for _, pd in ipairs(pistonsToActivate) do
                        local b = pd.block
                        if b and b:FindFirstChild("PPart") then
                            local currentCF = b:GetPivot()
                            local highCF = currentCF * CFrame.new(0, 10000, 0)
                            pcall(function() b.PPart.CFrame = highCF end)
                            task.spawn(function()
                                pcall(function() moveOpRF:InvokeServer({b}, currentCF, highCF, "Move") end)
                            end)
                            task.wait(0.1)
                        end
                    end
                    task.wait(0.5)



                    local pistonSet = {}
                    for _, pd in ipairs(pistonsToActivate) do pistonSet[pd.block] = true end
                    local otherPositions = {}
                    for _, b in ipairs(folder:GetChildren()) do
                        if not pistonSet[b] and b:FindFirstChild("PPart") then
                            otherPositions[#otherPositions + 1] = b.PPart.Position
                        end
                    end


                    local baseX = pistonsToActivate[1].block:FindFirstChild("PPart") and pistonsToActivate[1].block.PPart.Position.X or 0
                    local baseZ = pistonsToActivate[1].block:FindFirstChild("PPart") and pistonsToActivate[1].block.PPart.Position.Z or 0
                    local highY = pistonsToActivate[1].block:FindFirstChild("PPart") and pistonsToActivate[1].block.PPart.Position.Y or 10000

                    for idx, pd in ipairs(pistonsToActivate) do
                        local b = pd.block
                        if b and b:FindFirstChild("PPart") then

                            local targetX = baseX + (idx - 1) * 5
                            local targetZ = baseZ
                            local targetY = highY


                            local needsAdjust = true
                            local adjustOffset = 0
                            while needsAdjust and adjustOffset < 50 do
                                needsAdjust = false
                                local testPos = Vector3.new(targetX + adjustOffset, targetY, targetZ)
                                for _, op in ipairs(otherPositions) do
                                    if (testPos - op).Magnitude < 5 then
                                        needsAdjust = true
                                        adjustOffset = adjustOffset + 5
                                        break
                                    end
                                end

                                for prevIdx = 1, idx - 1 do
                                    local prevB = pistonsToActivate[prevIdx].block
                                    if prevB and prevB:FindFirstChild("PPart") then
                                        if (testPos - prevB.PPart.Position).Magnitude < 5 then
                                            needsAdjust = true
                                            adjustOffset = adjustOffset + 5
                                            break
                                        end
                                    end
                                end
                            end

                            targetX = targetX + adjustOffset
                            local currentCF = b:GetPivot()

                            local worldCF = pd.worldCF
                            local spacedCF = CFrame.new(targetX, targetY, targetZ) * (worldCF - worldCF.Position)
                            pcall(function() b.PPart.CFrame = spacedCF end)
                            task.spawn(function()
                                pcall(function() moveOpRF:InvokeServer({b}, currentCF, spacedCF, "Move") end)
                            end)
                            task.wait(0.1)
                        end
                    end
                    task.wait(0.5)
                end
            end
        end
        if #pistonsToActivate > 0 and not stopBuild and placeRF and myZone then
            updProg("Activating " .. #pistonsToActivate .. " pistons via button...", 50)
            local buttonBlockID = getBlockID("Button")
            local activationBindTool = Character:FindFirstChild("BindTool") or LocalPlayer.Backpack:FindFirstChild("BindTool")
            local activationBindRF = activationBindTool and activationBindTool:FindFirstChild("RF")
            local activationUnbindRF = activationBindTool and activationBindTool:FindFirstChild("UnbindRF")
            if buttonBlockID > 0 and activationBindRF then
                local beforeSet = {}
                for _, b in ipairs(folder:GetChildren()) do beforeSet[b] = true end
                local hrp = Character:FindFirstChild("HumanoidRootPart") or myZone
                pcall(function() placeRF:InvokeServer("Button", buttonBlockID, myZone, myZone.CFrame:ToObjectSpace(CFrame.new(hrp.Position + Vector3.new(0, 5, 0))), true) end)
                task.wait(0.5)
                local placedBtn
                for _, b in ipairs(folder:GetChildren()) do
                    if not beforeSet[b] and b.Name == "Button" and b:FindFirstChild("PPart") then placedBtn = b ; break end
                end
                if placedBtn then
                    local pushParts, pullParts = {}, {}
                    for _, pd in ipairs(pistonsToActivate) do
                        local bUp = pd.block:FindFirstChild("BindUp") or pd.block:FindFirstChild("BindUp", true)
                        local bDn = pd.block:FindFirstChild("BindDown") or pd.block:FindFirstChild("BindDown", true)
                        if bUp then pullParts[#pullParts + 1] = bUp end
                        if bDn then pushParts[#pushParts + 1] = bDn end
                    end
                    local bindFirstArg = {}
                    if #pushParts > 0 then bindFirstArg.Push = pushParts end
                    if #pullParts > 0 then bindFirstArg.Pull = pullParts end
                    if next(bindFirstArg) then
                        invokeWithTimeout(activationBindRF, {bindFirstArg, placedBtn, {}, false})
                        task.wait(0.3)
                        activateButtonBlock(placedBtn)
                        task.wait(0.15)
                        activateButtonBlock(placedBtn)
                        task.wait(2)
                    end
                    pcall(function() deleteBlock(placedBtn) end)
                    updProg("Pistons activated", 55)
                else
                    updProg("Piston activation: button not found", 55)
                end
            end
        end
        for _, entry in ipairs(styledPistons) do allStyled[#allStyled + 1] = entry end
        task.wait(0.2)
    end

    local restP0 = #pistonFlat > 0 and 60 or 0
    if #restFlat > 0 and not stopBuild then
        updProg("Placing " .. #restFlat .. " blocks...", restP0)
        runPlacePhase(restFlat, "Placing ", restP0, restP0 + 20)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        updProg("Waiting for blocks...", restP0 + 21)
        waitForN(math.floor((#pistonFlat + #restFlat) * 0.88), math.max(6, (#pistonFlat + #restFlat) * 0.02))
        local rUsed = {}
        for _, entry in ipairs(allStyled) do if entry.block then rUsed[entry.block] = true end end
        local styledRest = runStylePhase(restFlat, folder:GetChildren(), rUsed, restP0 + 23, restP0 + 50)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        updProg("Moving " .. #styledRest .. " blocks...", restP0 + 50)
        runMovePhase(styledRest, restP0 + 50, restP0 + 55)
        if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end
        applyPropertiesPhase(styledRest, restP0 + 55, restP0 + 60)
        for _, entry in ipairs(styledRest) do allStyled[#allStyled + 1] = entry end
    end

    local legacyCount = applyLegacySwitches(allStyled, propertiesRF)
    if legacyCount > 0 then
        updProg("Legacy switches: " .. legacyCount, 94)
        task.wait(0.25)
    end

    applyBindTables(allStyled, 95, 99)
    if stopBuild then isBuilding = false ; setStatus("Stopped") ; return false end

    if #styledPistons > 0 and not stopBuild then
        updProg("Activating pistons via QueueBlocksRequest...", 99)
        for _, entry in ipairs(styledPistons) do
            if entry.block and entry.block.Parent and entry.block.Name == "Piston" and entry.block:FindFirstChild("PPart") then
                local ld = entry.v and entry.v.NumberValues and entry.v.NumberValues.LastDirection
                if ld == 1 then
                    activatePistonViaQueue(entry.block)
                    task.wait(0.15)
                end
            end
        end
    end

    if not stopBuild and #allStyled > 0 then
        updProg("Waiting for blocks to settle...", 97)
        local totalCount = #allStyled
        local maxWait = 8
        local startT = tick()
        local function countSettled()
            local settledCount = 0
            for _, entry in ipairs(allStyled) do
                local b = entry.block
                if b and b.Parent and b:FindFirstChild("PPart") then
                    local pp = b.PPart
                    local target = entry.worldCF
                    if target and (pp.Position - target.Position).Magnitude < 1.5 then
                        settledCount = settledCount + 1
                    end
                else
                    settledCount = settledCount + 1
                end
            end
            return settledCount
        end
        while countSettled() < totalCount * 0.9 and tick() - startT < maxWait do
            if stopBuild then break end
            task.wait(0.2)
        end
        updProg("Weld fix: ensuring collision ON on everything...", 97)
        if propertiesRF then
            local needCCOn = {}
            for _, entry in ipairs(allStyled) do
                if entry.block and entry.block.Parent and entry.block:FindFirstChild("PPart") and not isSpecialPropBlock(entry.block.Name) then
                    if not entry.block.PPart.CanCollide then
                        needCCOn[#needCCOn+1] = entry.block
                    end
                end
            end
            if #needCCOn > 0 then
                for i = 1, #needCCOn, 50 do
                    local chunk = {}
                    for j = i, math.min(i + 49, #needCCOn) do
                        chunk[#chunk+1] = needCCOn[j]
                    end
                    pcall(function() propertiesRF:InvokeServer("Collision", chunk) end)
                end
                task.wait(0.2)
            end
        end

        for _, entry in ipairs(allStyled) do
            if entry.v and entry.v.IsTwoPart and entry.block and entry.block.Parent and entry.block:FindFirstChild("PPart") and not isSpecialPropBlock(entry.block.Name) then
                pcall(function()
                    entry.block.PPart.CanCollide = true
                    for _, desc in ipairs(entry.block:GetDescendants()) do
                        if desc:IsA("BasePart") then
                            desc.CanCollide = true
                        end
                    end
                end)
            end
        end

        updProg("Weld fix (non-Block, batch Rotate)...", 98)
        local nonBlockList = {}
        local trowelTool = Character:FindFirstChild("TrowelTool") or LocalPlayer.Backpack:FindFirstChild("TrowelTool")
        if trowelTool and trowelTool.Parent ~= Character then trowelTool.Parent = Character ; task.wait(0.05) end
        local weldOpRF = trowelTool and trowelTool:FindFirstChild("OperationRF")

        for _, entry in ipairs(allStyled) do
            if stopBuild then break end
            if entry.block and entry.block.Parent and entry.block:FindFirstChild("PPart") then
                local isBlock = entry.block.Name:sub(-5) == "Block"
                if not isBlock then
                    nonBlockList[#nonBlockList+1] = entry
                end
            end
        end

        if #nonBlockList > 0 and weldOpRF then
            local identityCF = CFrame.new(0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1)
            local BATCH = 15

            local moveNonBlockList = {}
            local rotateConstraintList = {}
            local otherNonBlockList = {}
            for _, entry in ipairs(nonBlockList) do
                if entry.block and isMoveWeldBlock(entry.block.Name) then
                    moveNonBlockList[#moveNonBlockList + 1] = entry
                elseif entry.block and isRotateConstraintBlock(entry.block.Name) then
                    rotateConstraintList[#rotateConstraintList + 1] = entry
                else
                    otherNonBlockList[#otherNonBlockList + 1] = entry
                end
            end


            if #otherNonBlockList > 0 then
                for pass = 1, 3 do
                    if stopBuild then break end
                    updProg("Weld fix Rotate pass " .. pass .. "/3 (" .. #otherNonBlockList .. " parts)", 98)
                    local rotated = 0
                    for i = 1, #otherNonBlockList, BATCH do
                        if stopBuild then break end
                        local batch = {}
                        for j = i, math.min(i + BATCH - 1, #otherNonBlockList) do
                            local e = otherNonBlockList[j]
                            if e.block and e.block.Parent and e.block:FindFirstChild("PPart") then
                                batch[#batch+1] = e.block
                            end
                        end
                        if #batch > 0 then
                            pcall(function() weldOpRF:InvokeServer(batch, identityCF, identityCF, "Rotate") end)
                            rotated = rotated + #batch
                            task.wait(0.08)
                        end
                    end
                    task.wait(0.25)
                    if rotated == 0 then break end
                end
            end


            if #rotateConstraintList > 0 then
                updProg("Weld fix Bar/Rope/Spring (" .. #rotateConstraintList .. " parts)...", 98)
                for i = 1, #rotateConstraintList, BATCH do
                    if stopBuild then break end
                    local batch = {}
                    local batchCFs = {}
                    for j = i, math.min(i + BATCH - 1, #rotateConstraintList) do
                        local e = rotateConstraintList[j]
                        if e.block and e.block.Parent and e.block:FindFirstChild("PPart") and e.worldCF then
                            batch[#batch+1] = e.block
                            batchCFs[#batchCFs+1] = e.worldCF
                        end
                    end
                    if #batch > 0 then

                        for k, b in ipairs(batch) do
                            pcall(function()
                                local cf = batchCFs[k]
                                local rotCF = cf - cf.Position

                                b.PPart.CFrame = cf * CFrame.Angles(0, 0.01, 0)
                                weldOpRF:InvokeServer({b}, rotCF, rotCF * CFrame.Angles(0, 0.01, 0), "Rotate")
                            end)
                            task.wait(0.08)
                        end
                        task.wait(0.1)

                        for k, b in ipairs(batch) do
                            pcall(function()
                                local cf = batchCFs[k]
                                local rotCF = cf - cf.Position
                                b.PPart.CFrame = cf
                                weldOpRF:InvokeServer({b}, rotCF * CFrame.Angles(0, 0.01, 0), rotCF, "Rotate")
                            end)
                            task.wait(0.08)
                        end
                        task.wait(0.1)
                    end
                end
            end


            if #moveNonBlockList > 0 then
                updProg("Weld fix constraint blocks (" .. #moveNonBlockList .. " parts)...", 98)
                for i = 1, #moveNonBlockList, BATCH do
                    if stopBuild then break end
                    local batch = {}
                    local batchCFs = {}
                    for j = i, math.min(i + BATCH - 1, #moveNonBlockList) do
                        local e = moveNonBlockList[j]
                        if e.block and e.block.Parent and e.block:FindFirstChild("PPart") and e.worldCF then
                            batch[#batch+1] = e.block
                            batchCFs[#batchCFs+1] = e.worldCF
                        end
                    end
                    if #batch > 0 then

                        for k, b in ipairs(batch) do
                            pcall(function()
                                local cf = batchCFs[k]

                                b.PPart.CFrame = cf * CFrame.new(0, 0.1, 0)
                                weldOpRF:InvokeServer({b}, cf, cf * CFrame.new(0, 0.1, 0), "Move")
                            end)
                            task.wait(0.08)
                        end
                        task.wait(0.1)

                        for k, b in ipairs(batch) do
                            pcall(function()
                                local cf = batchCFs[k]
                                b.PPart.CFrame = cf
                                weldOpRF:InvokeServer({b}, cf * CFrame.new(0, 0.1, 0), cf, "Move")
                            end)
                            task.wait(0.08)
                        end
                        task.wait(0.1)
                    end
                end
            end


            updProg("Weld fix: restoring positions...", 98)
            for i = 1, #nonBlockList, BATCH do
                if stopBuild then break end
                local batch = {}
                local batchCFs = {}
                for j = i, math.min(i + BATCH - 1, #nonBlockList) do
                    local e = nonBlockList[j]


                    if e.block and e.block.Parent and e.block:FindFirstChild("PPart") and e.worldCF
                        and not isRotateConstraintBlock(e.block.Name) then
                        batch[#batch+1] = e.block
                        batchCFs[#batchCFs+1] = e.worldCF
                    end
                end
                if #batch > 0 then
                    for k, b in ipairs(batch) do
                        pcall(function()
                            local targetCF = batchCFs[k]
                            local currentCF = b:GetPivot()
                            b.PPart.CFrame = targetCF
                            weldOpRF:InvokeServer({b}, currentCF, targetCF, "Move")
                        end)
                        task.wait(0.05)
                    end
                    task.wait(0.05)
                end
            end
            task.wait(0.3)
        end

        if #nonBlockList > 0 and not stopBuild then
            local propTool = Character:FindFirstChild("PropertiesTool") or LocalPlayer.Backpack:FindFirstChild("PropertiesTool")
            if propTool and propTool.Parent ~= Character then propTool.Parent = Character ; task.wait(0.05) end
            propertiesRF = propTool and propTool:FindFirstChild("SetPropertieRF") or propertiesRF

            if propertiesRF then
                local anOn, anOff, ccToggleOn = {}, {}, {}
                for _, entry in ipairs(nonBlockList) do
                    local b = entry.block
                    local v = entry.v
                    local pp = b.PPart

                    local isConstraint = isSpecialPropBlock(b.Name)

                    local wantA = (v.Anchored == nil) or (v.Anchored == true)
                    if pp.Anchored ~= wantA then
                        if wantA then anOn[#anOn+1] = b else anOff[#anOff+1] = b end
                    end

                    if not isConstraint and not pp.CanCollide then
                        ccToggleOn[#ccToggleOn+1] = b
                    end
                end

                if #anOn > 0 then pcall(function() propertiesRF:InvokeServer("Anchored", anOn) end) end
                task.wait(0.05)
                if #anOff > 0 then pcall(function() propertiesRF:InvokeServer("Anchored", anOff) end) end
                task.wait(0.05)
                if #ccToggleOn > 0 then
                    pcall(function() propertiesRF:InvokeServer("Collision", ccToggleOn) end)
                    task.wait(0.1)
                end

                local twoPartCCToggle = {}
                for _, entry in ipairs(nonBlockList) do
                    if entry.v and entry.v.IsTwoPart and entry.block and entry.block.Parent and not isSpecialPropBlock(entry.block.Name) then
                        pcall(function()
                            entry.block.PPart.CanCollide = true
                            for _, desc in ipairs(entry.block:GetDescendants()) do
                                if desc:IsA("BasePart") then
                                    desc.CanCollide = true
                                end
                            end
                        end)
                        if not entry.block.PPart.CanCollide then
                            twoPartCCToggle[#twoPartCCToggle+1] = entry.block
                        end
                    end
                end
                if #twoPartCCToggle > 0 then
                    pcall(function() propertiesRF:InvokeServer("Collision", twoPartCCToggle) end)
                end
                task.wait(0.15)

                local nbTransp = 0
                for _, entry in ipairs(nonBlockList) do
                    if stopBuild then break end
                    local v = entry.v
                    if v and v.Transparency ~= nil and not isSpecialPropBlock(entry.block.Name) then
                        local t = tonumber(v.Transparency)
                        if t then
                            t = math.clamp(t, 0, 1)
                            pcall(function()
                                propertiesRF:InvokeServer("Transparency", {entry.block}, tostring(math.floor(t * 100 + 0.5)))
                            end)
                            pcall(function()
                                for _, desc in ipairs(entry.block:GetDescendants()) do
                                    if desc:IsA("BasePart") and desc.Name ~= "PPart" then
                                        desc.Transparency = t
                                    end
                                end
                            end)
                            nbTransp = nbTransp + 1
                        end
                    end
                    if nbTransp % 10 == 0 then task.wait(0.05) end
                end
                if nbTransp > 0 then task.wait(0.1) end
            end
            updProg("Weld fix done (" .. #nonBlockList .. " non-Block rotated)", 99)
        else
            updProg("Weld fix done", 99)
        end
    end

    if not stopBuild and propertiesRF then
        local ccOffList = {}
        for _, entry in ipairs(allStyled) do
            if entry.block and entry.block.Parent and entry.block:FindFirstChild("PPart") and not isSpecialPropBlock(entry.block.Name) then
                local v = entry.v
                if v and v.CanCollide == false and entry.block.PPart.CanCollide then
                    ccOffList[#ccOffList+1] = entry.block
                end
            end
        end
        if #ccOffList > 0 then
            updProg("Disabling collision on " .. #ccOffList .. " parts...", 99)
            for i = 1, #ccOffList, 50 do
                local chunk = {}
                for j = i, math.min(i + 49, #ccOffList) do
                    chunk[#chunk+1] = ccOffList[j]
                end
                pcall(function() propertiesRF:InvokeServer("Collision", chunk) end)
            end
            task.wait(0.1)
        end
    end
    if #styledPistons > 0 and propertiesRF and not stopBuild then
        updProg("Piston transparency...", 99)
        local pDone = 0
        for _, entry in ipairs(styledPistons) do
            if stopBuild then break end
            if entry.block and entry.v and entry.v.Transparency ~= nil and entry.block.Name == "Piston" and entry.block:FindFirstChild("PPart") then
                local transparency = tonumber(entry.v.Transparency)
                if transparency then
                    transparency = math.clamp(transparency, 0, 1)
                    pcall(function()
                        firePropertyRF(propertiesRF, "Transparency", {entry.block}, tostring(math.floor(transparency * 100 + 0.5)))
                    end)
                    pDone = pDone + 1
                end
            end
            if pDone > 0 and pDone % 30 == 0 then task.wait() end
        end
        if pDone > 0 then
            task.wait(0.3)
            updProg("Piston transparency done (" .. pDone .. ")", 99)
        end
    end


    if not stopBuild and propertiesRF then
        local hingeTranspDone = 0
        for _, entry in ipairs(allStyled) do
            if stopBuild then break end
            if entry.block and entry.v and entry.v.Transparency ~= nil and entry.block.Name == "Hinge" and entry.block:FindFirstChild("PPart") then
                local transparency = tonumber(entry.v.Transparency)
                if transparency then
                    transparency = math.clamp(transparency, 0, 1)
                    pcall(function()
                        firePropertyRF(propertiesRF, "Transparency", {entry.block}, tostring(math.floor(transparency * 100 + 0.5)))
                    end)
                    hingeTranspDone = hingeTranspDone + 1
                end
            end
            if hingeTranspDone > 0 and hingeTranspDone % 30 == 0 then task.wait() end
        end
        if hingeTranspDone > 0 then
            task.wait(0.3)
            updProg("Hinge transparency done (" .. hingeTranspDone .. ")", 99)
        end
    end


    if not stopBuild and propertiesRF then
        local brsTranspDone = 0
        for _, entry in ipairs(allStyled) do
            if stopBuild then break end
            if entry.block and entry.v and entry.v.Transparency ~= nil and isRotateConstraintBlock(entry.block.Name) and entry.block:FindFirstChild("PPart") then
                local transparency = tonumber(entry.v.Transparency)
                if transparency then
                    transparency = math.clamp(transparency, 0, 1)
                    pcall(function()
                        firePropertyRF(propertiesRF, "Transparency", {entry.block}, tostring(math.floor(transparency * 100 + 0.5)))
                    end)
                    brsTranspDone = brsTranspDone + 1
                end
            end
            if brsTranspDone > 0 and brsTranspDone % 30 == 0 then task.wait() end
        end
        if brsTranspDone > 0 then
            task.wait(0.3)
            updProg("Bar/Rope/Spring transparency done (" .. brsTranspDone .. ")", 99)
        end
    end


    if not stopBuild and propertiesRF then
        local ccOffConstraint = {}
        for _, entry in ipairs(allStyled) do
            if entry.block and entry.block.Parent and entry.block:FindFirstChild("PPart") then
                local b = entry.block
                local v = entry.v
                if isSpecialPropBlock(b.Name) and v and v.CanCollide == false then
                    if b.PPart.CanCollide then
                        ccOffConstraint[#ccOffConstraint+1] = b
                    end
                end
            end
        end
        if #ccOffConstraint > 0 then
            updProg("Disabling collision on " .. #ccOffConstraint .. " constraint blocks...", 99)
            for i = 1, #ccOffConstraint, 50 do
                local chunk = {}
                for j = i, math.min(i + 49, #ccOffConstraint) do
                    chunk[#chunk+1] = ccOffConstraint[j]
                end
                pcall(function() propertiesRF:InvokeServer("Collision", chunk) end)
            end
            task.wait(0.1)
        end
    end

    updProg(stopBuild and "Stopped" or "Done! " .. total .. " blocks", 100)
    isBuilding = false
    pcall(function()
        local settingsFolder = LocalPlayer:FindFirstChild("Settings")
        if settingsFolder then
            local sbVal = settingsFolder:FindFirstChild("ShareBlocks")
            if sbVal then
                sbVal.Value = shareBlocksOriginal
            end
        end
    end)
    return true, placedById
end



local recentlyPlacedBlocks = {}

local function clearPreview()
    for _, o in pairs(PreviewFolder:GetChildren()) do o:Destroy() end
    for _, b in pairs(selectionBoxes) do if b then pcall(function() b:Destroy() end) end end
    previewParts = {}
    selectionBoxes = {}
    previewActive = false
    selectedObjectName = nil
    if updatePreviewButtonGlobal then updatePreviewButtonGlobal() end
end

local function createPreview(buildData, selBlock)
    clearPreview()
    local myZone = getPlayerZone(LocalPlayer)
    if not myZone then return false end
    local sc = Settings.buildScale
    local off = Vector3.new(Settings.buildOffsetX, Settings.buildOffsetY, Settings.buildOffsetZ)
    local targetT = Settings.previewTransparency
    local allFadeParts = {}
    local created = 0
    for blockName, blocks in pairs(buildData) do
        local tmpl = BuildingParts:FindFirstChild(blockName)
        if not tmpl then continue end
        for _, bi in pairs(blocks) do
            local relCF = getBlockCF(bi)
            local pos = (relCF.Position * sc) + off
            local scaledCF = CFrame.new(pos) * (relCF - relCF.Position)
            local worldCF = myZone.CFrame:ToWorldSpace(scaledCF)
            local pb = tmpl:Clone()
            if pb:FindFirstChild("PPart") then

                local partOffsets = {}
                for _, d in pairs(pb:GetDescendants()) do
                    if (d:IsA("BasePart") or d:IsA("UnionOperation")) and d ~= pb.PPart then
                        partOffsets[d] = pb.PPart.CFrame:ToObjectSpace(d.CFrame)
                    end
                end
                pb.PPart.CFrame = worldCF

                for d, offsetCF in pairs(partOffsets) do
                    pcall(function() d.CFrame = worldCF * offsetCF end)
                end
                local rawSize = bi.Size or bi.size
                if rawSize and rawSize ~= "" then
                    pcall(function() pb.PPart.Size = strV3(rawSize) * sc end)
                end
                local rawCol = bi.Col or bi.Color or bi.color or bi.col
                if rawCol and rawCol ~= "" then
                    pcall(function() pb.PPart.Color = strCol(tostring(rawCol)) end)
                end
                pb.PPart.Transparency = 1
                pb.PPart.CanCollide = false
                pb.PPart.Anchored = true
                allFadeParts[#allFadeParts + 1] = pb.PPart
                for _, d in pairs(pb:GetDescendants()) do
                    if d:IsA("BasePart") or d:IsA("UnionOperation") then
                        d.Transparency = 1
                        d.CanCollide = false
                        d.Anchored = true
                        allFadeParts[#allFadeParts + 1] = d
                    end
                end
                pb.Name = blockName
                pb.Parent = PreviewFolder
                if selBlock and blockName == selBlock then
                    local hl = Instance.new("Highlight")
                    hl.Adornee = pb.PPart
                    hl.FillColor = Color3.fromRGB(255,255,255)
                    hl.OutlineColor = Color3.fromRGB(255,255,255)
                    hl.FillTransparency = 0.7
                    hl.OutlineTransparency = 0.3
                    hl.Parent = pb.PPart
                    selectionBoxes[blockName] = hl
                end
                table.insert(previewParts, pb)
                created = created + 1
                if created % 80 == 0 then task.wait() end
            end
        end
    end
    previewActive = true
    if updatePreviewButtonGlobal then updatePreviewButtonGlobal() end
    if updateBlocksDisplayGlobal then updateBlocksDisplayGlobal() end
    task.spawn(function()
        local startT = tick()
        local dur = 0.45
        while true do
            local el = tick() - startT
            local a = math.clamp(el / dur, 0, 1)
            local tVal = 1 + (targetT - 1) * a
            for _, p in ipairs(allFadeParts) do
                if p and p.Parent then p.Transparency = tVal end
            end
            if a >= 1 then break end
            if not previewActive then break end
            task.wait(0.03)
        end
    end)
    return true
end

local function updateSelectionHighlight(blockName)
    for _, b in pairs(selectionBoxes) do if b then pcall(function() b:Destroy() end) end end
    selectionBoxes = {}
    if blockName then
        for _, p in pairs(previewParts) do
            if p.Name == blockName and p:FindFirstChild("PPart") then
                local hl = Instance.new("Highlight")
                hl.Adornee = p.PPart
                hl.FillColor = Color3.fromRGB(255,255,255)
                hl.OutlineColor = Color3.fromRGB(255,255,255)
                hl.FillTransparency = 0.7
                hl.OutlineTransparency = 0.3
                hl.Parent = p.PPart
                selectionBoxes[blockName] = hl
                break
            end
        end
    end
end

local preTerminateCallbacks = {}

local function terminateScript(screenGui)
    for _, fn in ipairs(preTerminateCallbacks) do pcall(fn) end
    stopBuild = true

    pcall(function()
        local sf = LocalPlayer:FindFirstChild("Settings")
        if sf then
            local sb = sf:FindFirstChild("ShareBlocks")
            if sb then sb.Value = shareBlocksOriginal end
        end
    end)
    pcall(function()
        workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
        LocalPlayer.CameraMaxZoomDistance = 400
        LocalPlayer.CameraMinZoomDistance = 0.5
    end)
    clearPreview()

    pcall(function()
        local shpf = workspace:FindFirstChild("ShapePreview")
        if shpf then shpf:Destroy() end
    end)

    pcall(function()
        local explSound = Instance.new("Sound")
        explSound.SoundId = UISoundConfig.explode
        explSound.Volume = 2
        explSound.Parent = SoundService
        explSound:Play()
        task.delay(3, function() pcall(function() explSound:Destroy() end) end)
    end)

    local mf = screenGui and screenGui:FindFirstChild("MainFrame")
    if mf then
        task.spawn(function()

            local t = TweenService:Create(mf, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(255,40,20)})
            t:Play() ; task.wait(0.1)

            local origPos = mf.Position
            for shakeStep = 1, 8 do
                local offsetX = math.random(-6, 6)
                local offsetY = math.random(-6, 6)
                mf.Position = UDim2.new(origPos.X.Scale, origPos.X.Offset + offsetX, origPos.Y.Scale, origPos.Y.Offset + offsetY)
                task.wait(0.03)
            end
            mf.Position = origPos

            local explRing = Instance.new("Frame")
            explRing.Size = UDim2.new(0, 20, 0, 20)
            explRing.Position = UDim2.new(0.5, -10, 0.5, -10)
            explRing.BackgroundColor3 = Color3.fromRGB(255, 120, 30)
            explRing.BackgroundTransparency = 0.3
            explRing.BorderSizePixel = 0
            explRing.ZIndex = 999
            local ringCorner = Instance.new("UICorner"); ringCorner.CornerRadius = UDim.new(1, 0); ringCorner.Parent = explRing
            explRing.Parent = mf
            TweenService:Create(explRing, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Size = UDim2.new(2, 0, 2, 0),
                Position = UDim2.new(-0.5, 0, -0.5, 0),
                BackgroundTransparency = 1,
            }):Play()
            task.wait(0.35)
            pcall(function() explRing:Destroy() end)

            TweenService:Create(mf, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
                Size = UDim2.new(0, 8, 0, 8),
                Position = UDim2.new(0.5, -4, 0.5, -4),
                BackgroundTransparency = 1,
            }):Play()
            task.wait(0.3)
            pcall(function() screenGui:Destroy() end)
        end)
        task.wait(0.7)
    else
        pcall(function() if screenGui then screenGui:Destroy() end end)
    end
    pcall(function()
        local cg = game:GetService("CoreGui")
        for _, o in pairs(cg:GetChildren()) do
            if o.Name:find("SPRB") or o.Name:find("CScript_") or o.Name:find("KnifeHUD_SPRB") then
                pcall(function() o:Destroy() end)
            end
        end
    end)
    pcall(function()
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then
            for _, o in pairs(pg:GetChildren()) do
                if o.Name:find("SPRB") or o.Name:find("CScript_") or o.Name:find("KnifeHUD_SPRB") then
                    pcall(function() o:Destroy() end)
                end
            end
        end
    end)
    stopBuild = false
    isBuilding = false
    setStatus = function() end
end

local UI = nil
local rebuildUI = nil

local function addResizeGripLine(parent, i, color)
    local line = Instance.new("Frame")
    line.Size = UDim2.new(0, 10 - (i * 2), 0, 1)
    line.Position = UDim2.new(1, -12 + (i * 3), 1, -3 - (i * 4))
    line.Rotation = -45
    line.BackgroundColor3 = color
    line.BackgroundTransparency = 0.35
    line.BorderSizePixel = 0
    line.ZIndex = 119
    line.Parent = parent
end

local function createOpenButton(screenGui, textColor)
    local openBtn = Instance.new("TextButton")
    openBtn.Name = "OpenBtn"
    openBtn.Size = UDim2.new(0, 48, 0, 48)
    openBtn.Position = UDim2.new(0, 8, 1, -64)
    openBtn.BackgroundColor3 = Colors.Panel
    openBtn.BackgroundTransparency = 0.2
    openBtn.BorderSizePixel = 0
    openBtn.Text = "SPRB"
    openBtn.TextColor3 = textColor
    openBtn.TextSize = 13
    openBtn.Font = Enum.Font.GothamBold
    openBtn.ZIndex = 100
    openBtn.Visible = false
    openBtn.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 3)
    corner.Parent = openBtn

    local stroke = Instance.new("UIStroke")
    stroke.Color = Colors.Border
    stroke.Thickness = 2
    stroke.Parent = openBtn

    return openBtn
end

local function createResizeGrip(mainFrame, color)
    local grip = Instance.new("Frame")
    grip.Name = "ResizeGrip"
    grip.Size = UDim2.new(0, 16, 0, 16)
    grip.Position = UDim2.new(1, -18, 1, -18)
    grip.BackgroundTransparency = 1
    grip.ZIndex = 119
    grip.Parent = mainFrame

    for i = 1, 3 do
        addResizeGripLine(grip, i, color)
    end
end

local function bindWindowButtons(closeBtn, openBtn, showGUI, hideGUI)
    closeBtn.MouseButton1Click:Connect(function()
        task.spawn(hideGUI)
    end)

    openBtn.MouseButton1Click:Connect(function()
        openBtn.Visible = false
        showGUI()
    end)
end

local farmState = {
    active = false, runs = 0, earned = 0, startTime = 0, totalTime = 0,
    positions = {
        Vector3.new(-61.26, 69.78, 1273.84),
        Vector3.new(-48.86, 85.36, 1441.15),
        Vector3.new(-46.04, 72.56, 2211.08),
        Vector3.new(-67.08, 72.74, 3726.19),
        Vector3.new(-55.31, 82.60, 4523.01),
        Vector3.new(-52.80, 116.90, 5937.94),
        Vector3.new(-62.71, 82.72, 6709.27),
        Vector3.new(-53.46, 94.83, 8241.62),
        Vector3.new(-56.53, -362.03, 9486.16),
    }
}
local function getGold()
    local dv = LocalPlayer:FindFirstChild("Data")
    if dv then local g = dv:FindFirstChild("Gold") if g then return g.Value end end
    return 0
end
local function fmtTime(s)
    s = math.floor(s)
    local h = math.floor(s / 3600) local m = math.floor((s % 3600) / 60)
    if h > 0 then return string.format("%d:%02d:%02d", h, m, s % 60) end
    return string.format("%d:%02d", m, s % 60)
end
local function getFarmStats()
    local elapsed = farmState.active and (tick() - farmState.startTime + farmState.totalTime) or farmState.totalTime
    local hrs = elapsed / 3600
    return { active = farmState.active, runs = farmState.runs, earned = farmState.earned, time = fmtTime(elapsed), rate = hrs > 0.01 and math.floor(farmState.earned / hrs) or 0 }
end

local _toggleTGFarm = nil
local _farmTgBtnRef = nil
local Lang = {
    en = {
        Build = "Build", Convert = "Convert", Schematic = "Schematic",
        Misc = "Misc", Settings = "Settings",
        Import = "Import", Export = "Export", Clear = "Clear",
        BuildIt = "Build", Stop = "Stop", Preview = "Preview",
        AutoPreview = "Auto Preview", ShowCounts = "Show Counts",
        ReplaceTool = "Give Change Tool", PaintPlus = "Give Paint+",
        DragonH = "Dragon Harpoon", CookieW = "Cookie Wheels",
        MegaT = "Orange Mega Turbines", PineT = "Buy Pine Tree",
        EasterTP = "Easter Event Place", ChristmasTP = "Christmas Event Place",
        TestTP = "Test Place",
        PaintPlusTitle = "Paint+", ResetSel = "Reset",
        SelectedBlock = "SELECTED BLOCK", Colors = "COLORS",
        FromColor = "FROM Color", ToColor = "TO Color",
        Transparency = "Transparency",
        Actions = "ACTIONS",
        SwapColor = "Color Swap", PaintMat = "Paint Material",
        PaintAll = "Paint ALL", RandColors = "Random Colors",
        RandPerMat = "Random per Mat", SetTransp = "Set Transparency",
        PaintSingle = "Paint Single", InvertColors = "Invert FROM/TO",
        CopyToFrom = "TO = FROM", RandSaturated = "Random Saturated",
        ClickToSelect = "Click a block to select",
        None = "None", BackpackTools = "BACKPACK TOOLS",
        ShopPurchases = "SHOP PURCHASES", Teleports = "TELEPORTS",
        CombatUtils = "COMBAT UTILS",
        AddRemove = "Add / Remove Selected Player",
        ClearSel = "Clear Selected Players",
        NoFolder = "No folder!", NoBlocks = "No blocks!",
        NeedPaint = "Need Paint tool!", NeedProps = "Need Properties!",
        SelectBlock = "Select a block!", NoMatching = "No matching!",
        SelectionCleared = "Selection cleared",
        Selected = "Selected: ",
        Done = "Done: ",
        Painting = "Painting ", Swapping = "Swapping ",
        Randomizing = "Randomizing ", ColoringByMat = "Coloring by mat...",
        SettingTransp = "Setting transparency...",
        Saturating = "Saturating ",
        ColorsInverted = "Colors inverted",
        TOCopied = "TO copied from FROM",
    },

}





local function createUI()

    tween = nil; bump = nil; stylizeCard = nil; makeTab = nil
    makeBtn = nil; makeInput = nil; makeLabel = nil; makeSlider = nil
    makeColorPicker = nil; makeNumInput = nil; makeDropdown = nil
    makeExSub = nil; makeBuildSub = nil

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SPRBBuilder"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.IgnoreGuiInset = true

    local DropdownLayer = Instance.new("Frame")
    DropdownLayer.Name = "DropdownLayer"
    DropdownLayer.Size = UDim2.new(1, 0, 1, 0)
    DropdownLayer.BackgroundTransparency = 1
    DropdownLayer.BorderSizePixel = 0
    DropdownLayer.ZIndex = 500
    DropdownLayer.Parent = ScreenGui
    local activeDropdownClose = nil

    local baseW = 640
    local baseH = 360

    local isMobileMode = Settings.mobileMode
    if isMobileMode then Settings.uiScale = math.min(Settings.uiScale, 0.78) end
    syncColors()

    function tween(obj, ti, props)
        return TweenService:Create(obj, ti, props)
    end

    function bump(btn, scale)
        local uiScale = btn:FindFirstChild("HoverScale")
        if not uiScale then
            uiScale = Instance.new("UIScale")
            uiScale.Name = "HoverScale"
            uiScale.Scale = 1
            uiScale.Parent = btn
        end
        tween(uiScale, TweenInfo.new(0.14, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = scale}):Play()
    end

    function stylizeCard(obj, bgColor, strokeColor, corner)
        obj.BackgroundColor3 = bgColor or Colors.Panel
        obj.BorderSizePixel = 0
        local c = obj:FindFirstChildOfClass("UICorner") or Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, corner or 3)
        c.Parent = obj
        local s = obj:FindFirstChildOfClass("UIStroke") or Instance.new("UIStroke")
        s.Color = strokeColor or Colors.Border
        s.Transparency = 0.82
        s.Thickness = 1
        s.Parent = obj
        return c, s
    end

    local OpenBtn = nil
    local MinBtn = nil
    local TabsBar = nil
    local ContentArea = nil
    local updateTabSizes = nil
    local refreshContentCanvases = nil
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, baseW * Settings.uiScale, 0, baseH * Settings.uiScale)
    MainFrame.Position = UDim2.new(0.5, -(baseW*Settings.uiScale)/2, 0.5, -(baseH*Settings.uiScale)/2)
    MainFrame.BackgroundColor3 = Colors.BG
    MainFrame.BackgroundTransparency = Settings.guiTransparency
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.ClipsDescendants = true
    MainFrame.Visible = false
    MainFrame.Parent = ScreenGui
    local minimized = Settings.uiMinimized == true
    local MIN_WINDOW_W = isMobileMode and 300 or 380
    local MIN_WINDOW_H = isMobileMode and 280 or 300

    local function clampFramePosition(x, y, width, height)
        local viewport = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
        local maxX = math.max(0, viewport.X - width)
        local maxY = math.max(0, viewport.Y - height)
        return math.clamp(x, 0, maxX), math.clamp(y, 0, maxY)
    end

    local function clampFrameSize(width, height)
        local viewport = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
        local maxW = math.max(MIN_WINDOW_W, viewport.X - 12)
        local maxH = math.max(MIN_WINDOW_H, viewport.Y - 12)
        return math.clamp(width, MIN_WINDOW_W, maxW), math.clamp(height, MIN_WINDOW_H, maxH)
    end

    local function getTargetSize()
        local width = tonumber(Settings.windowWidth)
        local height = tonumber(Settings.windowHeight)
        if not width or width < 0 then width = baseW * Settings.uiScale end
        if not height or height < 0 then height = baseH * Settings.uiScale end
        width, height = clampFrameSize(width, height)
        if minimized then height = 44 end
        return width, height
    end

    local function getStoredPosition(width, height)
        local viewport = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
        local x = tonumber(Settings.windowPosX)
        local y = tonumber(Settings.windowPosY)
        if x == nil or y == nil or x < 0 or y < 0 then
            x = (viewport.X - width) / 2
            y = (viewport.Y - height) / 2
        end
        return clampFramePosition(x, y, width, height)
    end

    local function saveFramePosition(x, y)
        Settings.windowPosX = math.floor(x + 0.5)
        Settings.windowPosY = math.floor(y + 0.5)
    end

    local function saveFrameSize(width, height)
        if minimized then return end
        width, height = clampFrameSize(width, height)
        Settings.windowWidth = math.floor(width + 0.5)
        Settings.windowHeight = math.floor(height + 0.5)
    end
    local MFCorner = Instance.new("UICorner")
    MFCorner.CornerRadius = UDim.new(0, 6)
    MFCorner.Parent = MainFrame

    local MFStroke = Instance.new("UIStroke")
    MFStroke.Color = Colors.Border
    MFStroke.Transparency = 0.78
    MFStroke.Thickness = 1
    MFStroke.Parent = MainFrame

    local function showGUI()
        playUISound(UISoundConfig.open)
        MainFrame.Visible = true
        local targetW, targetH = getTargetSize()
        local targetX, targetY = getStoredPosition(targetW, targetH)
        local startW = targetW * 0.24
        local startH = 44
        local startX = 0
        local startY = 0
        local startXO = targetX + (targetW / 2) - (startW / 2)
        local startYO = targetY + (targetH / 2) - (startH / 2)
        if OpenBtn and OpenBtn.Parent then
            local ap = OpenBtn.AbsolutePosition
            local as = OpenBtn.AbsoluteSize
            startXO = ap.X + (as.X / 2) - (startW / 2)
            startYO = ap.Y + (as.Y / 2) - (startH / 2)
        end
        MainFrame.Size = UDim2.new(0, startW, 0, startH)
        MainFrame.Position = UDim2.new(startX, startXO, startY, startYO)
        MainFrame.BackgroundTransparency = 1
        MainFrame.Rotation = 0
        if TabsBar then TabsBar.Visible = false end
        if ContentArea then ContentArea.Visible = false end
        if MinBtn then MinBtn.Text = minimized and "+" or "-" end
        local openTween = tween(MainFrame, TweenInfo.new(0.32, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, targetW, 0, targetH),
            Position = UDim2.new(0, targetX, 0, targetY),
            BackgroundTransparency = Settings.guiTransparency,
            Rotation = 0
        })
        openTween:Play()
        openTween.Completed:Connect(function()
            if not MainFrame.Visible then return end
            if TabsBar then TabsBar.Visible = not minimized end
            if ContentArea then ContentArea.Visible = not minimized end
            if updateTabSizes then updateTabSizes() end
            if refreshContentCanvases then refreshContentCanvases() end
            MainFrame.Rotation = 0
            applyWindowBackground(MainFrame)
        end)
        local boot = Instance.new("Frame")
        boot.Size = UDim2.new(1, 0, 1, 0)
        boot.BackgroundColor3 = Colors.BG
        boot.BorderSizePixel = 0
        boot.ZIndex = 40
        boot.Parent = MainFrame
        tween(boot, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1}):Play()
        task.delay(0.2, function()
            if boot and boot.Parent then boot:Destroy() end
        end)
    end

    local function hideGUI()
        playUISound(UISoundConfig.close)
        local endW = 44
        local endH = 44
        local currentPos = MainFrame.AbsolutePosition
        local currentSize = MainFrame.AbsoluteSize
        local pos = UDim2.new(0, currentPos.X + (currentSize.X / 2) - 22, 0, currentPos.Y + (currentSize.Y / 2) - 22)
        if OpenBtn and OpenBtn.Parent then
            OpenBtn.Visible = true
            OpenBtn.TextTransparency = 1
            OpenBtn.BackgroundTransparency = 1
            OpenBtn.Position = UDim2.new(0, 8, 1, -64)
            local ap = OpenBtn.AbsolutePosition
            local as = OpenBtn.AbsoluteSize
            pos = UDim2.new(0, ap.X + (as.X / 2) - (endW / 2), 0, ap.Y + (as.Y / 2) - (endH / 2))
        end
        tween(MainFrame, TweenInfo.new(0.24, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
            Size = UDim2.new(0, endW, 0, endH),
            Position = pos,
            BackgroundTransparency = 1,
            Rotation = 0
        }):Play()
        if OpenBtn then
            tween(OpenBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                BackgroundTransparency = 0.2,
                TextTransparency = 0
            }):Play()
        end
        task.wait(0.23)
        MainFrame.Visible = false
    end

    local Header = Instance.new("Frame")
    Header.Size = UDim2.new(1, 0, 0, 44)
    Header.BackgroundColor3 = Colors.PanelElevated
    Header.BackgroundTransparency = 0.02
    Header.BorderSizePixel = 0
    Header.ZIndex = 2
    Header.Parent = MainFrame

    local HCorner = Instance.new("UICorner")
    HCorner.CornerRadius = UDim.new(0, 6)
    HCorner.Parent = Header

    local HeaderStroke = Instance.new("UIStroke")
    HeaderStroke.Color = Colors.Border
    HeaderStroke.Transparency = 0.88
    HeaderStroke.Thickness = 1
    HeaderStroke.Parent = Header

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0.68, 0, 0, 32)
    Title.Position = UDim2.new(0, 12, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = "SPRB // V5"
    Title.TextColor3 = Colors.Text
    Title.TextSize = 24
    Title.Font = Enum.Font.GothamBold
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.ZIndex = 3
    Title.Parent = Header

    local headerGrad = Instance.new("UIGradient")
    headerGrad.Name = "SPRB_HeaderGrad"
    headerGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
        ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
    })
    headerGrad.Rotation = 90
    headerGrad.Parent = Header

    local function makeHeaderBtn(txt, xOff)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 30, 0, 30)
        b.Position = UDim2.new(1, xOff, 0.5, -15)
        b.BackgroundColor3 = Colors.Panel
        b.BackgroundTransparency = 0
        b.BorderSizePixel = 0
        b.Text = txt
        b.TextColor3 = Colors.Text
        b.TextSize = 15
        b.Font = Enum.Font.GothamBold
        b.ZIndex = 4
        b.Parent = Header
        stylizeCard(b, Colors.Panel, Colors.Border, 6)
        local hScale = Instance.new("UIScale"); hScale.Scale = 1; hScale.Parent = b
        b.MouseEnter:Connect(function()
            tween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.PanelElevated}):Play()
            tween(hScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.1}):Play()
        end)
        b.MouseLeave:Connect(function()
            tween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.Panel}):Play()
            tween(hScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale = 1}):Play()
        end)
        b.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            tween(hScale, TweenInfo.new(0.06, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 0.9}):Play()
            task.wait(0.06)
            tween(hScale, TweenInfo.new(0.12, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1.1}):Play()
        end)
        return b
    end

    MinBtn = makeHeaderBtn("-", -66)
    local CloseBtn = makeHeaderBtn("X", -34)

    TabsBar = Instance.new("Frame")
    TabsBar.Size = UDim2.new(0, 82, 1, -60)
    TabsBar.Position = UDim2.new(0, 10, 0, 52)
    TabsBar.BackgroundColor3 = Colors.PanelSoft
    TabsBar.BackgroundTransparency = Settings.guiTransparency or 0.15
    TabsBar.BorderSizePixel = 0
    TabsBar.ClipsDescendants = true
    TabsBar.Parent = MainFrame
    stylizeCard(TabsBar, Colors.PanelSoft, Colors.Border, 4)

    local TBLayout = Instance.new("UIListLayout")
    TBLayout.FillDirection = Enum.FillDirection.Vertical
    TBLayout.Padding = UDim.new(0, 4)
    TBLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
    TBLayout.Parent = TabsBar

    local TBPad = Instance.new("UIPadding")
    TBPad.PaddingLeft = UDim.new(0,6)
    TBPad.PaddingRight = UDim.new(0,6)
    TBPad.PaddingTop = UDim.new(0,6)
    TBPad.PaddingBottom = UDim.new(0,6)
    TBPad.Parent = TabsBar

    ContentArea = Instance.new("Frame")
    ContentArea.Size = UDim2.new(1, -100, 1, -60)
    ContentArea.Position = UDim2.new(0, 96, 0, 52)
    ContentArea.BackgroundTransparency = 1
    ContentArea.ClipsDescendants = true
    ContentArea.Parent = MainFrame
    TabsBar.Visible = not minimized
    ContentArea.Visible = not minimized
    MinBtn.Text = minimized and "+" or "-"

    local function setScrollCanvas(scrollFrame, contentHeight, pad)
        local h = math.max(0, math.floor((contentHeight or 0) + (pad or 0)))
        scrollFrame.ScrollingDirection = Enum.ScrollingDirection.Y
        pcall(function() scrollFrame.AutomaticCanvasSize = Enum.AutomaticSize.None end)
        pcall(function() scrollFrame.ElasticBehavior = Enum.ElasticBehavior.Never end)
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, h)
        local maxScroll = math.max(0, h - scrollFrame.AbsoluteSize.Y)
        if scrollFrame.CanvasPosition.Y > maxScroll then
            scrollFrame.CanvasPosition = Vector2.new(0, maxScroll)
        end
    end

    local tabOrder = 0
    local tabButtons = {}
    updateTabSizes = function()
        if not TabsBar then return end
        local count = #tabButtons
        if count == 0 then return end
        local available = math.max(30, TabsBar.AbsoluteSize.Y - 16 - ((count - 1) * 6))
        local tabH = math.floor(math.max(30, available / count))
        for _, btn in ipairs(tabButtons) do
            if btn and btn.Parent then
                btn.Size = UDim2.new(1, 0, 0, tabH)
                btn.TextSize = math.clamp(math.floor(tabH / 4), 13, 18)
            end
        end
    end

    function makeTab(name, label)
        tabOrder += 1
        local btn = Instance.new("TextButton")
        btn.Name = name .. "Tab"
        btn.LayoutOrder = tabOrder
        btn.Size = UDim2.new(1, 0, 0, 34)
        btn.BackgroundColor3 = Colors.PanelElevated
        btn.BackgroundTransparency = Settings.guiTransparency or 0.15
        btn.BorderSizePixel = 0
        btn.Text = label
        btn.TextColor3 = Colors.Muted
        btn.TextSize = 14
        btn.Font = Enum.Font.GothamMedium
        btn.TextXAlignment = Enum.TextXAlignment.Center
        btn.TextTruncate = Enum.TextTruncate.AtEnd
        btn.Parent = TabsBar
        tabButtons[#tabButtons + 1] = btn
        stylizeCard(btn, Colors.PanelElevated, Colors.Border, 4)
        local tScale = Instance.new("UIScale"); tScale.Scale = 1; tScale.Parent = btn
        btn.MouseEnter:Connect(function()
            if btn.TextColor3 ~= Colors.ActiveText then
                tween(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = math.max(0, (Settings.guiTransparency or 0.15) - 0.1)}):Play()
                tween(tScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.03}):Play()
            end
        end)
        btn.MouseLeave:Connect(function()
            if btn.TextColor3 ~= Colors.ActiveText then
                tween(btn, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = Settings.guiTransparency or 0.15}):Play()
                tween(tScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale = 1}):Play()
            else
                tween(tScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale = 1}):Play()
            end
        end)
        btn.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
        end)

        local frame = Instance.new("ScrollingFrame")
        frame.Name = name .. "Frame"
        frame.Size = UDim2.new(1, -14, 1, -12)
        frame.Position = UDim2.new(0, 7, 0, 6)
        frame.BackgroundTransparency = 1
        frame.ClipsDescendants = true
        frame.ScrollBarThickness = 0
        frame.ScrollBarImageColor3 = Colors.Muted
        frame.CanvasSize = UDim2.new(0, 0, 0, 0)
        pcall(function() frame.ElasticBehavior = Enum.ElasticBehavior.Never end)
        frame.Visible = false
        frame.Parent = ContentArea

        local fl = Instance.new("UIListLayout")
        fl.Padding = UDim.new(0, 6)
        fl.SortOrder = Enum.SortOrder.LayoutOrder
        fl.Parent = frame

        local _tabResizeGuard = false
        fl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            if _tabResizeGuard then return end
            _tabResizeGuard = true
            setScrollCanvas(frame, fl.AbsoluteContentSize.Y, 16)
            task.defer(function() _tabResizeGuard = false end)
        end)

        return btn, frame
    end

    refreshContentCanvases = function()
        if not ContentArea then return end
        for _, f in pairs(ContentArea:GetChildren()) do
            if f:IsA("ScrollingFrame") then
                local layout = f:FindFirstChildOfClass("UIListLayout")
                if layout then
                    setScrollCanvas(f, layout.AbsoluteContentSize.Y, 16)
                end
            end
        end
    end

    local activeContentFrame = nil
    local tabFrameOrder = {}
    local function switchTab(frame)
        local previousFrame = activeContentFrame
        if previousFrame == frame then return end
        activeContentFrame = frame

        local prevIdx = 0
        local newIdx = 0
        for i, t in ipairs(tabFrameOrder) do
            if t == previousFrame then prevIdx = i end
            if t == frame then newIdx = i end
        end
        local goingDown = newIdx > prevIdx
        if previousFrame and previousFrame.Parent then
            local exitY = goingDown and -12 or (ContentArea.AbsoluteSize.Y + 12)
            tween(previousFrame, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Position = UDim2.new(0, 7, 0, exitY)
            }):Play()
            task.delay(0.14, function()
                if previousFrame ~= activeContentFrame and previousFrame and previousFrame.Parent then
                    previousFrame.Visible = false
                    previousFrame.Position = UDim2.new(0, 7, 0, 6)
                end
            end)
        end
        for _, f in pairs(ContentArea:GetChildren()) do
            if f:IsA("ScrollingFrame") and f ~= frame and f ~= previousFrame then
                f.Visible = false
                f.Position = UDim2.new(0, 7, 0, 6)
            end
        end
        for _, b in pairs(TabsBar:GetChildren()) do
            if b:IsA("TextButton") then
                local oldTG = b:FindFirstChild("SPRB_TabGrad")
                if oldTG then oldTG:Destroy() end
                tween(b, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    BackgroundColor3 = Colors.PanelElevated,
                    BackgroundTransparency = 0.06,
                    TextColor3 = Colors.Muted
                }):Play()
            end
        end
        frame.Visible = true
        frame.CanvasPosition = Vector2.new(0, 0)
        local enterY = goingDown and (ContentArea.AbsoluteSize.Y + 12) or -12
        frame.Position = UDim2.new(0, 7, 0, enterY)
        tween(frame, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Position = UDim2.new(0, 7, 0, 6)
        }):Play()
        local tb = TabsBar:FindFirstChild(frame.Name:gsub("Frame", "Tab"))
        if tb then

            local oldTG = tb:FindFirstChild("SPRB_TabGrad")
            if oldTG then oldTG:Destroy() end

            local tabGrad = Instance.new("UIGradient")
            tabGrad.Name = "SPRB_TabGrad"
            tabGrad.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
            tabGrad.Rotation = 90
            tabGrad.Parent = tb
            tween(tb, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                BackgroundTransparency = 0,
                TextColor3 = Colors.ActiveText
            }):Play()
        end
    end

    task.wait()
    function makeBtn(name, txt, parent, cb)
        local b = Instance.new("TextButton")
        b.Name = name
        b.Size = UDim2.new(1, 0, 0, 32)
        b.BackgroundColor3 = Colors.PanelElevated
        b.BackgroundTransparency = 0
        b.BorderSizePixel = 0
        b.Text = txt
        b.TextColor3 = Colors.Text
        b.TextSize = 12
        b.Font = Enum.Font.GothamMedium
        b.Parent = parent
        local _, bs = stylizeCard(b, Colors.PanelElevated, Colors.Border, 3)
        if bs then bs.Transparency = 0.92 end
        local bScale = Instance.new("UIScale"); bScale.Scale = 1; bScale.Parent = b
        b.MouseEnter:Connect(function()
            playUISound(UISoundConfig.hover)
            tween(bs, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = 0.6, Color = Colors.ActiveBG}):Play()
            tween(bScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.02}):Play()
        end)
        b.MouseLeave:Connect(function()
            tween(bs, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = 0.92, Color = Colors.Border}):Play()
            tween(bScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale = 1}):Play()
        end)
        b.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            tween(b, TweenInfo.new(0.06), {BackgroundTransparency = 0.3}):Play()
            tween(bScale, TweenInfo.new(0.06, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 0.97}):Play()
            task.wait(0.06)
            tween(b, TweenInfo.new(0.1), {BackgroundTransparency = 0}):Play()
            tween(bScale, TweenInfo.new(0.1, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1.02}):Play()
            if cb then task.spawn(cb) end
        end)
        return b
    end

    function makeInput(name, ph, parent)
        local f = Instance.new("Frame")
        f.Name = name .. "Frame"
        f.Size = UDim2.new(1, 0, 0, 30)
        f.BackgroundColor3 = Colors.PanelSoft
        f.BackgroundTransparency = 0
        f.BorderSizePixel = 0
        f.Parent = parent
        local _, fs = stylizeCard(f, Colors.PanelSoft, Colors.Border, 3)
        fs.Transparency = 0.78
        local inp = Instance.new("TextBox")
        inp.Name = name
        inp.Size = UDim2.new(1, -14, 1, 0)
        inp.Position = UDim2.new(0, 7, 0, 0)
        inp.BackgroundTransparency = 1
        inp.PlaceholderText = ph
        inp.PlaceholderColor3 = Colors.Muted
        inp.Text = ""
        inp.TextColor3 = Colors.Text
        inp.TextSize = 12
        inp.Font = Enum.Font.Gotham
        inp.TextXAlignment = Enum.TextXAlignment.Left
        inp.ClearTextOnFocus = false
        inp.Parent = f
        inp.Focused:Connect(function() tween(fs, TweenInfo.new(0.12), {Color = Colors.ActiveBG, Transparency = 0.25}):Play() end)
        inp.FocusLost:Connect(function() tween(fs, TweenInfo.new(0.12), {Color = Colors.Border, Transparency = 0.78}):Play() end)
        return inp
    end

    function makeLabel(txt, parent)
        local l = Instance.new("TextLabel")
        l.Size = UDim2.new(1, 0, 0, 20)
        l.BackgroundTransparency = 1
        l.Text = "  " .. txt
        l.TextColor3 = Colors.Muted
        l.TextSize = 11
        l.Font = Enum.Font.GothamBold
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.Parent = parent
        return l
    end

    function makeSlider(name, minV, maxV, curV, parent, lblTxt, fmtFn, onChange)
        local cont = Instance.new("Frame")
        cont.Name = name .. "Cont"
        cont.Size = UDim2.new(1, 0, 0, 34)
        cont.Active = true
        cont.BackgroundColor3 = Colors.PanelSoft
        cont.BackgroundTransparency = 0
        cont.BorderSizePixel = 0
        cont.Parent = parent
        stylizeCard(cont, Colors.PanelSoft, Colors.Border, 3)

        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0.42, -8, 1, 0)
        lbl.Position = UDim2.new(0, 8, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = lblTxt
        lbl.TextColor3 = Color3.fromRGB(200, 200, 200)
        lbl.TextSize = 12
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = cont

        local valLbl = Instance.new("TextBox")
        valLbl.Name = name .. "Val"
        valLbl.Size = UDim2.new(0.28, 0, 1, 0)
        valLbl.Position = UDim2.new(0.42, 0, 0, 0)
        valLbl.BackgroundTransparency = 1
        valLbl.Text = fmtFn and fmtFn(curV) or tostring(curV)
        valLbl.TextColor3 = Colors.Text
        valLbl.TextSize = 12
        valLbl.Font = Enum.Font.GothamMedium
        valLbl.TextXAlignment = Enum.TextXAlignment.Center
        valLbl.ClearTextOnFocus = false
        valLbl.MultiLine = false
        valLbl.Parent = cont

        local currentVal = curV
        local step = (maxV - minV) <= 1 and 0.01 or (((maxV - minV) <= 10) and 0.05 or 1)

        local function applyVal(v, fire)
            currentVal = math.clamp(v, minV, maxV)
            if step >= 1 then currentVal = math.floor(currentVal + 0.5) end
            valLbl.Text = fmtFn and fmtFn(currentVal) or tostring(currentVal)
            if fire and onChange then onChange(currentVal) end
        end

        local function stepBtn(txt, xPos, delta)
            local b = Instance.new("TextButton")
            b.Size = UDim2.new(0.15, -4, 0, 24)
            b.Position = UDim2.new(xPos, 0, 0.5, -12)
            b.BackgroundColor3 = Colors.PanelElevated
            b.BorderSizePixel = 0
            b.Text = txt
            b.TextColor3 = Colors.Text
            b.TextSize = 13
            b.Font = Enum.Font.GothamMedium
            b.Parent = cont
            stylizeCard(b, Colors.PanelElevated, Colors.Border, 3)
            b.MouseButton1Click:Connect(function()
                applyVal(currentVal + delta, true)
            end)
            return b
        end

        stepBtn("-", 0.70, -step)
        stepBtn("+", 0.85, step)
        valLbl.FocusLost:Connect(function()

            local raw = valLbl.Text:gsub("[^%d%.%-]", "")
            local n = tonumber(raw)
            if n then
                applyVal(n, true)
            else
                valLbl.Text = fmtFn and fmtFn(currentVal) or tostring(currentVal)
            end
        end)

        local function setVal(v)
            applyVal(v, false)
        end

        return cont, setVal, function() return currentVal end
    end

    function makeColorPicker(name, currentColor, parent, onChange)
        local cont = Instance.new("Frame")
        cont.Name = name .. "ColorPicker"
        cont.Size = UDim2.new(1, 0, 0, 24)
        cont.BackgroundTransparency = 1
        cont.Parent = parent
        cont.ClipsDescendants = true

        local mainRow = Instance.new("Frame")
        mainRow.Size = UDim2.new(1, 0, 0, 24)
        mainRow.BackgroundTransparency = 1
        mainRow.Name = "MainRow"
        mainRow.Parent = cont
        local mrl = Instance.new("UIListLayout")
        mrl.FillDirection = Enum.FillDirection.Horizontal
        mrl.Padding = UDim.new(0, 4)
        mrl.VerticalAlignment = Enum.VerticalAlignment.Center
        mrl.Parent = mainRow

        local previewBtn = Instance.new("TextButton")
        previewBtn.Name = name .. "Preview"
        previewBtn.Size = UDim2.new(0, 22, 0, 22)
        previewBtn.BackgroundColor3 = currentColor
        previewBtn.BorderSizePixel = 0
        previewBtn.Text = ""
        previewBtn.AutoButtonColor = false
        previewBtn.Parent = mainRow
        local pvCr = Instance.new("UICorner"); pvCr.CornerRadius = UDim.new(0, 4); pvCr.Parent = previewBtn
        local pvSt = Instance.new("UIStroke"); pvSt.Color = Color3.fromRGB(80,80,80); pvSt.Thickness = 1; pvSt.Parent = previewBtn

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, -52, 0, 22)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = name
        nameLabel.TextColor3 = Colors.Text
        nameLabel.TextSize = 11
        nameLabel.Font = Enum.Font.GothamSemibold
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Parent = mainRow

        local editBtn = Instance.new("TextButton")
        editBtn.Name = name .. "EditBtn"
        editBtn.Size = UDim2.new(0, 22, 0, 22)
        editBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        editBtn.BorderSizePixel = 0
        editBtn.Text = "v"
        editBtn.TextColor3 = Colors.Muted
        editBtn.TextSize = 10
        editBtn.Font = Enum.Font.GothamBold
        editBtn.AutoButtonColor = false
        editBtn.Parent = mainRow
        local ebCr = Instance.new("UICorner"); ebCr.CornerRadius = UDim.new(0, 4); ebCr.Parent = editBtn

        local expandPanel = Instance.new("Frame")
        expandPanel.Name = "ExpandPanel"
        expandPanel.Size = UDim2.new(1, 0, 0, 0)
        expandPanel.AutomaticSize = Enum.AutomaticSize.Y
        expandPanel.Position = UDim2.new(0, 0, 0, 24)
        expandPanel.BackgroundTransparency = 1
        expandPanel.Visible = false
        expandPanel.ClipsDescendants = true
        expandPanel.Parent = cont
        local panelLayout = Instance.new("UIListLayout")
        panelLayout.Padding = UDim.new(0, 4)
        panelLayout.Parent = expandPanel

        local curR = math.floor(currentColor.R * 255)
        local curG = math.floor(currentColor.G * 255)
        local curB = math.floor(currentColor.B * 255)
        local sliderRefs = {}

        local function makeChannelSlider(labelText, initialVal, barColor)
            local row = Instance.new("Frame")
            row.Size = UDim2.new(1, 0, 0, 18)
            row.BackgroundTransparency = 1
            row.Parent = expandPanel

            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(0, 14, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.Text = labelText
            lbl.TextColor3 = Color3.fromRGB(180, 180, 180)
            lbl.TextSize = 11
            lbl.Font = Enum.Font.GothamBold
            lbl.Parent = row

            local track = Instance.new("Frame")
            track.Name = labelText .. "Track"
            track.Size = UDim2.new(1, -42, 0, 10)
            track.Position = UDim2.new(0, 18, 0.5, -5)
            track.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
            track.BorderSizePixel = 0
            track.Parent = row
            local trCr = Instance.new("UICorner"); trCr.CornerRadius = UDim.new(0, 5); trCr.Parent = track

            local fill = Instance.new("Frame")
            fill.Name = "Fill"
            fill.Size = UDim2.new(initialVal / 255, 0, 1, 0)
            fill.BackgroundColor3 = barColor
            fill.BorderSizePixel = 0
            fill.Parent = track
            local flCr = Instance.new("UICorner"); flCr.CornerRadius = UDim.new(0, 5); flCr.Parent = fill

            local thumb = Instance.new("Frame")
            thumb.Name = "Thumb"
            thumb.Size = UDim2.new(0, 14, 0, 14)
            thumb.Position = UDim2.new(initialVal / 255, -7, 0.5, -7)
            thumb.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
            thumb.BorderSizePixel = 0
            thumb.Parent = track
            local thCr = Instance.new("UICorner"); thCr.CornerRadius = UDim.new(0, 7); thCr.Parent = thumb

            local valLbl = Instance.new("TextLabel")
            valLbl.Size = UDim2.new(0, 24, 1, 0)
            valLbl.Position = UDim2.new(1, -24, 0, 0)
            valLbl.BackgroundTransparency = 1
            valLbl.Text = tostring(initialVal)
            valLbl.TextColor3 = Color3.fromRGB(200, 200, 200)
            valLbl.TextSize = 11
            valLbl.Font = Enum.Font.GothamBold
            valLbl.TextXAlignment = Enum.TextXAlignment.Right
            valLbl.Parent = row

            local dragging = false
            local function updateFromX(xPos)
                local relX = math.clamp((xPos - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
                local val = math.floor(relX * 255 + 0.5)
                val = math.clamp(val, 0, 255)
                fill.Size = UDim2.new(relX, 0, 1, 0)
                thumb.Position = UDim2.new(relX, -7, 0.5, -7)
                valLbl.Text = tostring(val)
                return val
            end

            local function fireChange()
                local c = Color3.fromRGB(curR, curG, curB)
                TweenService:Create(previewBtn, TweenInfo.new(0.15), {BackgroundColor3 = c}):Play()
                if onChange then onChange(c) end
            end

            thumb.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = true
                end
            end)
            track.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = true
                    local val = updateFromX(input.Position.X)
                    if labelText == "R" then curR = val
                    elseif labelText == "G" then curG = val
                    else curB = val end
                    fireChange()
                end
            end)
            UserInputService.InputChanged:Connect(function(input)
                if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                    local val = updateFromX(input.Position.X)
                    if labelText == "R" then curR = val
                    elseif labelText == "G" then curG = val
                    else curB = val end
                    fireChange()
                end
            end)
            UserInputService.InputEnded:Connect(function(input)
                if dragging and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
                    dragging = false
                end
            end)

            sliderRefs[labelText] = { fill = fill, thumb = thumb, valLbl = valLbl }
            return row
        end

        makeChannelSlider("R", curR, Color3.fromRGB(220, 60, 60))
        makeChannelSlider("G", curG, Color3.fromRGB(60, 200, 60))
        makeChannelSlider("B", curB, Color3.fromRGB(60, 100, 220))

        local hexRow = Instance.new("Frame"); hexRow.Size = UDim2.new(1, 0, 0, 18); hexRow.BackgroundTransparency = 1; hexRow.Parent = expandPanel
        local hexLbl = Instance.new("TextLabel"); hexLbl.Size = UDim2.new(0, 28, 1, 0); hexLbl.BackgroundTransparency = 1
        hexLbl.Text = "HEX"; hexLbl.TextColor3 = Color3.fromRGB(180,180,180); hexLbl.TextSize = 11
        hexLbl.Font = Enum.Font.GothamBold; hexLbl.Parent = hexRow
        local hexInp = Instance.new("TextBox"); hexInp.Size = UDim2.new(1, -42, 1, 0); hexInp.Position = UDim2.new(0, 30, 0, 0)
        hexInp.BackgroundColor3 = Color3.fromRGB(30,30,30); hexInp.BackgroundTransparency = 0; hexInp.BorderSizePixel = 0
        hexInp.Text = string.format("#%02X%02X%02X", curR, curG, curB)
        hexInp.TextColor3 = Color3.fromRGB(200,200,200); hexInp.TextSize = 11; hexInp.Font = Enum.Font.GothamBold
        hexInp.PlaceholderText = "#FF0000"; hexInp.PlaceholderColor3 = Color3.fromRGB(100,100,100)
        hexInp.TextXAlignment = Enum.TextXAlignment.Left; hexInp.ClearTextOnFocus = false; hexInp.Parent = hexRow
        local hxCr2 = Instance.new("UICorner"); hxCr2.CornerRadius = UDim.new(0, 4); hxCr2.Parent = hexInp
        local function updateHexDisp()
            hexInp.Text = string.format("#%02X%02X%02X", curR, curG, curB)
        end
        hexInp.FocusLost:Connect(function()
            local txt = hexInp.Text:gsub("^#", "")
            if #txt == 6 then
                local hr = tonumber(txt:sub(1,2), 16) or 0
                local hg = tonumber(txt:sub(3,4), 16) or 0
                local hb = tonumber(txt:sub(5,6), 16) or 0
                curR = math.clamp(hr,0,255); curG = math.clamp(hg,0,255); curB = math.clamp(hb,0,255)
                for ch, val in pairs({R=curR, G=curG, B=curB}) do
                    local sr = sliderRefs[ch]
                    if sr then local rel = val/255; sr.fill.Size = UDim2.new(rel,0,1,0); sr.thumb.Position = UDim2.new(rel,-7,0.5,-7); sr.valLbl.Text = tostring(val) end
                end
                local c = Color3.fromRGB(curR, curG, curB)
                TweenService:Create(previewBtn, TweenInfo.new(0.15), {BackgroundColor3 = c}):Play()
                if onChange then onChange(c) end
            end
            updateHexDisp()
        end)

        local isOpen = false
        editBtn.MouseButton1Click:Connect(function()
            isOpen = not isOpen
            expandPanel.Visible = isOpen
            if isOpen then
                TweenService:Create(cont, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 108)}):Play()
                editBtn.Text = "^"
                updateHexDisp()
            else
                TweenService:Create(cont, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 24)}):Play()
                editBtn.Text = "v"
            end
        end)

        return {
            getColor = function() return Color3.fromRGB(curR, curG, curB) end,
            setColor = function(c)
                curR = math.floor(c.R * 255 + 0.5); curG = math.floor(c.G * 255 + 0.5); curB = math.floor(c.B * 255 + 0.5)
                TweenService:Create(previewBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(curR, curG, curB)}):Play()
                for ch, val in pairs({R=curR, G=curG, B=curB}) do
                    local sr = sliderRefs[ch]
                    if sr then local rel = val/255; sr.fill.Size = UDim2.new(rel,0,1,0); sr.thumb.Position = UDim2.new(rel,-7,0.5,-7); sr.valLbl.Text = tostring(val) end
                end
                updateHexDisp()
            end,
            preview = previewBtn,
            container = cont,
        }
    end

    function makeNumInput(label, default, minV, maxV, stepV, parent, onChange)
        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, 22)
        row.BackgroundTransparency = 1
        row.Parent = parent
        local fmt = (stepV >= 1 and math.floor(default) == default) and "%d" or "%.1f"
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0.36, 0, 1, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = label
        lbl.TextColor3 = Colors.Muted
        lbl.TextSize = 10
        lbl.Font = Enum.Font.GothamBold
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = row
        local val = default
        local mBtn = Instance.new("TextButton")
        mBtn.Size = UDim2.new(0.12, 0, 1, 0)
        mBtn.Position = UDim2.new(0.36, 0, 0, 0)
        mBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        mBtn.BorderSizePixel = 0
        mBtn.Text = "-"
        mBtn.TextColor3 = Colors.Text
        mBtn.TextSize = 12
        mBtn.Font = Enum.Font.GothamBold
        mBtn.Parent = row
        local mC = Instance.new("UICorner"); mC.CornerRadius = UDim.new(0, 3); mC.Parent = mBtn
        local tbox = Instance.new("TextBox")
        tbox.Size = UDim2.new(0.34, 0, 1, 0)
        tbox.Position = UDim2.new(0.48, 0, 0, 0)
        tbox.BackgroundColor3 = Colors.PanelElevated
        tbox.BorderSizePixel = 0
        tbox.Text = string.format(fmt, default)
        tbox.TextColor3 = Colors.Text
        tbox.TextSize = 11
        tbox.Font = Enum.Font.GothamBold
        tbox.ClearTextOnFocus = false
        tbox.Parent = row
        local tC = Instance.new("UICorner"); tC.CornerRadius = UDim.new(0, 3); tC.Parent = tbox
        local pBtn = Instance.new("TextButton")
        pBtn.Size = UDim2.new(0.12, 0, 1, 0)
        pBtn.Position = UDim2.new(0.82, 0, 0, 0)
        pBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        pBtn.BorderSizePixel = 0
        pBtn.Text = "+"
        pBtn.TextColor3 = Colors.Text
        pBtn.TextSize = 12
        pBtn.Font = Enum.Font.GothamBold
        pBtn.Parent = row
        local pC = Instance.new("UICorner"); pC.CornerRadius = UDim.new(0, 3); pC.Parent = pBtn
        local function upd() tbox.Text = string.format(fmt, val); if onChange then onChange(val) end end
        mBtn.MouseButton1Click:Connect(function()
            val = math.max(minV or -9999, val - stepV) upd()
        end)
        pBtn.MouseButton1Click:Connect(function()
            val = math.min(maxV or 9999, val + stepV) upd()
        end)
        tbox.FocusLost:Connect(function()
            local n = tonumber(tbox.Text)
            if n then val = math.max(minV or -9999, math.min(maxV or 9999, n)) end
            upd()
        end)
        return function() return val end, function(v) val = v; upd() end
    end

    function makeDropdown(name, getOpts, parent, cb)
        local closedH = 34
        local df = Instance.new("Frame")
        df.Name = name .. "DF"
        df.Size = UDim2.new(1, 0, 0, closedH)
        df.BackgroundColor3 = Colors.PanelSoft
        df.BackgroundTransparency = 0
        df.BorderSizePixel = 0
        df.ZIndex = 10
        df.Parent = parent
        local _, dfs = stylizeCard(df, Colors.PanelSoft, Colors.Border, 3)
        dfs.Transparency = 0.68

        local dbtn = Instance.new("TextButton")
        dbtn.Name = name
        dbtn.Size = UDim2.new(1, -30, 1, 0)
        dbtn.Position = UDim2.new(0, 8, 0, 0)
        dbtn.BackgroundTransparency = 1
        dbtn.Text = "Select..."
        dbtn.TextColor3 = Colors.Muted
        dbtn.TextSize = 12
        dbtn.Font = Enum.Font.Gotham
        dbtn.TextXAlignment = Enum.TextXAlignment.Left
        dbtn.ZIndex = 203
        dbtn.Parent = df

        local arrow = Instance.new("TextLabel")
        arrow.Size = UDim2.new(0, 20, 1, 0)
        arrow.Position = UDim2.new(1, -24, 0, 0)
        arrow.BackgroundTransparency = 1
        arrow.Text = "v"
        arrow.TextColor3 = Colors.Muted
        arrow.TextSize = 12
        arrow.Font = Enum.Font.GothamBold
        arrow.ZIndex = 203
        arrow.Parent = df


        local container = Instance.new("Frame")
        container.Name = name .. "Popup"
        container.BackgroundColor3 = Colors.Panel
        container.BackgroundTransparency = 0
        container.BorderSizePixel = 0
        container.ZIndex = 210
        container.Visible = false
        container.ClipsDescendants = true
        container.Parent = DropdownLayer
        local contC = Instance.new("UICorner"); contC.CornerRadius = UDim.new(0, 6); contC.Parent = container
        local contS = Instance.new("UIStroke"); contS.Color = Colors.Border; contS.Thickness = 1; contS.Parent = container


        local searchBox = Instance.new("TextBox")
        searchBox.Name = "SearchBox"
        searchBox.Size = UDim2.new(1, -12, 0, 26)
        searchBox.Position = UDim2.new(0, 6, 0, 6)
        searchBox.BackgroundColor3 = Colors.PanelSoft
        searchBox.BackgroundTransparency = 0
        searchBox.BorderSizePixel = 0
        searchBox.Text = ""
        searchBox.PlaceholderText = "Search..."
        searchBox.PlaceholderColor3 = Colors.Muted
        searchBox.TextColor3 = Colors.Text
        searchBox.TextSize = 12
        searchBox.Font = Enum.Font.Gotham
        searchBox.ZIndex = 212
        searchBox.ClearTextOnFocus = false
        searchBox.Parent = container
        local sbC = Instance.new("UICorner"); sbC.CornerRadius = UDim.new(0, 4); sbC.Parent = searchBox
        local sbP = Instance.new("UIPadding"); sbP.PaddingLeft = UDim.new(0, 6); sbP.Parent = searchBox


        local listFrame = Instance.new("Frame")
        listFrame.Name = "ListFrame"
        listFrame.BackgroundTransparency = 1
        listFrame.ClipsDescendants = true
        listFrame.ZIndex = 211
        listFrame.Parent = container

        local scroll = Instance.new("ScrollingFrame")
        scroll.Name = "Scroll"
        scroll.Size = UDim2.new(1, 0, 1, 0)
        scroll.BackgroundTransparency = 1
        scroll.ScrollBarThickness = 4
        scroll.ScrollBarImageColor3 = Colors.Muted
        scroll.ZIndex = 211
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.Parent = listFrame
        local sL = Instance.new("UIListLayout"); sL.Padding = UDim.new(0, 1); sL.Parent = scroll
        local sP = Instance.new("UIPadding"); sP.PaddingTop = UDim.new(0, 2); sP.PaddingBottom = UDim.new(0, 4); sP.Parent = scroll

        local optionBtns = {}
        local currentOpts = {}
        local isOpen = false
        local closeTween = nil

        local function closeList()
            if not isOpen then return end
            isOpen = false
            if closeTween then closeTween:Cancel() end
            closeTween = TweenService:Create(container, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, container.AbsoluteSize.X, 0, 4)})
            closeTween:Play()
            task.delay(0.15, function()
                container.Visible = false
                container.Size = UDim2.new(0, 0, 0, 0)
            end)
            df.ZIndex = 10
            arrow.Text = "v"
        end

        local function populateList(filter)
            for _, ob in ipairs(optionBtns) do pcall(function() ob:Destroy() end) end
            optionBtns = {}
            local fl = tostring(filter or ""):lower():gsub("^%s*(.-)%s*$", "%1")
            local filtered = {}
            for _, opt in pairs(currentOpts) do
                local optName = type(opt) == "table" and (opt.name or opt.display or tostring(opt)) or tostring(opt)
                local label = type(opt) == "table" and (opt.display or opt.name or tostring(opt)) or tostring(opt)
                if fl == "" or tostring(label):lower():find(fl, 1, true) or tostring(optName):lower():find(fl, 1, true) then
                    filtered[#filtered+1] = {optName = optName, label = label, opt = opt}
                end
            end
            for _, item in ipairs(filtered) do
                local ob = Instance.new("TextButton")
                ob.Name = item.optName
                ob.Size = UDim2.new(1, -12, 0, 24)
                ob.BackgroundColor3 = Colors.PanelElevated
                ob.BackgroundTransparency = 0
                ob.BorderSizePixel = 0
                ob.Text = "  " .. item.label
                ob.TextColor3 = Colors.Text
                ob.TextSize = 11
                ob.Font = Enum.Font.Gotham
                ob.TextXAlignment = Enum.TextXAlignment.Left
                ob.ZIndex = 212
                ob.Parent = scroll
                local obc = Instance.new("UICorner"); obc.CornerRadius = UDim.new(0, 3); obc.Parent = ob
                ob.MouseEnter:Connect(function()
                    TweenService:Create(ob, TweenInfo.new(0.1), {BackgroundColor3 = Colors.ActiveBG:Lerp(Colors.PanelElevated, 0.6)}):Play()
                end)
                ob.MouseLeave:Connect(function()
                    TweenService:Create(ob, TweenInfo.new(0.1), {BackgroundColor3 = Colors.PanelElevated}):Play()
                end)
                ob.MouseButton1Click:Connect(function()
                    dbtn.Text = item.label
                    dbtn.TextColor3 = Colors.Text
                    closeList()
                    if cb then cb(item.optName) end
                end)
                optionBtns[#optionBtns+1] = ob
            end
            scroll.CanvasSize = UDim2.new(0, 0, 0, math.max(30, #filtered * 25 + 6))
        end

        searchBox:GetPropertyChangedSignal("Text"):Connect(function()
            populateList(searchBox.Text)
        end)

        local function refresh()
            searchBox.Text = ""
            for _, ob in ipairs(optionBtns) do pcall(function() ob:Destroy() end) end
            optionBtns = {}
            local opts = getOpts()
            if type(opts) ~= "table" then opts = {} end
            currentOpts = opts
            populateList("")
        end

        dbtn.MouseButton1Click:Connect(function()
            if isOpen then
                closeList()
                if activeDropdownClose == closeList then activeDropdownClose = nil end
                return
            end

            if activeDropdownClose then pcall(activeDropdownClose) end
            activeDropdownClose = closeList

            refresh()
            df.ZIndex = 200

            local ap = df.AbsolutePosition
            local as = df.AbsoluteSize
            local viewport = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
            local cw = math.max(160, as.X)
            local opts = currentOpts
            local numOpts = 0
            for _ in pairs(opts) do numOpts = numOpts + 1 end

            local showSearch = numOpts >= 8
            local searchH = showSearch and 36 or 0
            local ch = math.min(290, math.max(80, numOpts * 25 + searchH + 10))
            local x = ap.X
            local yBelow = ap.Y + as.Y + 3
            local y = yBelow
            if yBelow + ch > viewport.Y - 6 then
                y = ap.Y - ch - 3
            end
            x = math.clamp(x, 6, math.max(6, viewport.X - cw - 6))
            y = math.clamp(y, 6, math.max(6, viewport.Y - ch - 6))

            searchBox.Visible = showSearch

            container.Position = UDim2.new(0, x, 0, y)
            container.Size = UDim2.new(0, cw, 0, 4)
            listFrame.Position = UDim2.new(0, 0, 0, searchH)
            listFrame.Size = UDim2.new(1, 0, 1, -(searchH + 4))

            container.Visible = true
            isOpen = true
            TweenService:Create(container, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, cw, 0, ch)}):Play()
            arrow.Text = "^"
            if showSearch then
                task.defer(function() searchBox:CaptureFocus() end)
            end
        end)

        DropdownLayer.InputBegan:Connect(function(inp)
            if not isOpen then return end
            if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
            local mx, my = inp.Position.X, inp.Position.Y
            local dfp, dfsz = df.AbsolutePosition, df.AbsoluteSize
            local cp = container.AbsolutePosition
            local cs = container.AbsoluteSize
            local inDf = (mx >= dfp.X and mx <= dfp.X + dfsz.X and my >= dfp.Y and my <= dfp.Y + dfsz.Y)
            local inPopup = (mx >= cp.X and mx <= cp.X + cs.X and my >= cp.Y and my <= cp.Y + cs.Y)
            if (not inDf) and (not inPopup) then
                closeList()
                if activeDropdownClose == closeList then activeDropdownClose = nil end
            end
        end)

        return dbtn, refresh
    end

    task.wait()
    local refreshColors = function()

        applyWindowBackground(MainFrame)
        MFStroke.Color = Colors.Border

        Header.BackgroundColor3 = Colors.PanelElevated
        HeaderStroke.Color = Colors.Border
        Title.TextColor3 = Colors.Text

        local hg = Header:FindFirstChild("SPRB_HeaderGrad")
        if hg then
            hg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
        end
        local rg = Header:FindFirstChild("SPRB_ReplaceGrad")
        if rg then
            rg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
        end
        local pg = Header:FindFirstChild("SPRB_PaintGrad")
        if pg then
            pg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
        end

        for _, child in pairs(Header:GetChildren()) do
            if child:IsA("TextButton") then
                child.BackgroundColor3 = Colors.Panel
                child.TextColor3 = Colors.Text
                local s = child:FindFirstChildOfClass("UIStroke")
                if s then s.Color = Colors.Border end
            end
        end

        TabsBar.BackgroundColor3 = Colors.PanelSoft
        local tbStroke = TabsBar:FindFirstChildOfClass("UIStroke")
        if tbStroke then tbStroke.Color = Colors.Border end

        for _, btn in ipairs(tabButtons) do
            local isActive = btn:FindFirstChild("SPRB_TabGrad") ~= nil
            if isActive then
                local tg = btn:FindFirstChild("SPRB_TabGrad")
                if tg then
                    tg.Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                        ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
                    })
                end
            end
            btn.TextColor3 = isActive and Colors.ActiveText or Colors.Muted
            local s = btn:FindFirstChildOfClass("UIStroke")
            if s then s.Color = Colors.Border end
        end

        for _, name in pairs({"buildSubBar", "exSubBar", "stSubBar"}) do
            local bar = MainFrame:FindFirstChild(name, true)
            if bar and bar:IsA("Frame") then
                bar.BackgroundColor3 = Colors.PanelSoft

                for _, b in pairs(bar:GetChildren()) do
                    if b:IsA("TextButton") then
                        local sg = b:FindFirstChild("SPRB_SubGrad")
                        local isActive = sg ~= nil
                        if isActive then
                            sg.Color = ColorSequence.new({
                                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
                            })
                        end
                        b.TextColor3 = isActive and Colors.ActiveText or Colors.Muted
                    end
                end
            end
        end

        if StatusLabelRef and StatusLabelRef.Parent then
            StatusLabelRef.TextColor3 = Colors.Text
            StatusLabelRef.BackgroundColor3 = Colors.PanelSoft
            local s = StatusLabelRef:FindFirstChildOfClass("UIStroke")
            if s then s.Color = Colors.Border end
        end

        if ProgressBarFillRef and ProgressBarFillRef.Parent then
            ProgressBarFillRef.BackgroundColor3 = Colors.ActiveBG
        end
        if InfProgressFillRef and InfProgressFillRef.Parent then
            InfProgressFillRef.BackgroundColor3 = Colors.ActiveBG
        end

        if DupeInfoLabelRef and DupeInfoLabelRef.Parent then
            DupeInfoLabelRef.TextColor3 = Colors.Muted
            DupeInfoLabelRef.BackgroundColor3 = Colors.Panel
        end
        if DupePercentLabelRef and DupePercentLabelRef.Parent then
            DupePercentLabelRef.TextColor3 = Colors.Text
        end
        if MiscStatusLabelRef and MiscStatusLabelRef.Parent then
            MiscStatusLabelRef.TextColor3 = Colors.Text
        end

        for _, tabFrame in pairs(ContentArea:GetChildren()) do
            if tabFrame:IsA("ScrollingFrame") then
                for _, subFrame in pairs(tabFrame:GetChildren()) do
                    if subFrame:IsA("ScrollingFrame") then
                        for _, child in pairs(subFrame:GetChildren()) do
                            if child:IsA("TextButton") and child.BackgroundColor3 ~= Colors.ActiveBG then

                                if not child:FindFirstChild("SPRB_SubGrad") and not child:FindFirstChild("SPRB_TabGrad") then

                                    local r, g, b = child.BackgroundColor3.R * 255, child.BackgroundColor3.G * 255, child.BackgroundColor3.B * 255
                                    if not (r < 30 and g > 20 and b < 30) then
                                        child.BackgroundColor3 = Colors.PanelElevated
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end

    local T1btn, T1frame = makeTab("Build", "BUILD")
    local T2btn, T2frame = makeTab("Blocks", "BLOCKS")
    local T3btn, T3frame = makeTab("Exploit", "EXPLOIT")
    local T4btn, T4frame = makeTab("Settings", "SETTINGS")
    tabFrameOrder = {T1frame, T2frame, T3frame, T4frame}
    task.wait()

    local StatusLabel = Instance.new("TextLabel")
    StatusLabel.Name = "StatusLabel"
    StatusLabel.Size = UDim2.new(1, 0, 0, 26)
    StatusLabel.BackgroundColor3 = Colors.PanelSoft
    StatusLabel.BackgroundTransparency = 0
    StatusLabel.BorderSizePixel = 0
    StatusLabel.Text = "  Ready"
    StatusLabel.TextColor3 = Colors.Text
    StatusLabel.TextSize = 12
    StatusLabel.Font = Enum.Font.GothamSemibold
    StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
    StatusLabel.Parent = T1frame
    stylizeCard(StatusLabel, Colors.PanelSoft, Colors.Border, 3)
    StatusLabelRef = StatusLabel

    local DupeInfoFrame = Instance.new("Frame")
    DupeInfoFrame.Name = "DupeInfoFrame"
    DupeInfoFrame.Size = UDim2.new(1, 0, 0, 36)
    DupeInfoFrame.BackgroundTransparency = 1
    DupeInfoFrame.Parent = T1frame
    local difl = Instance.new("UIListLayout")
    difl.Padding = UDim.new(0, 2)
    difl.Parent = DupeInfoFrame

    local ProgressBarBG = Instance.new("Frame")
    ProgressBarBG.Name = "ProgressBarBG"
    ProgressBarBG.Size = UDim2.new(1, 0, 0, 14)
    ProgressBarBG.BackgroundColor3 = Colors.PanelElevated
    ProgressBarBG.BackgroundTransparency = 0
    ProgressBarBG.BorderSizePixel = 0
    ProgressBarBG.Parent = DupeInfoFrame
    local pbc = Instance.new("UICorner") ; pbc.CornerRadius = UDim.new(1,0) ; pbc.Parent = ProgressBarBG
    local ProgressBarFill = Instance.new("Frame")
    ProgressBarFill.Name = "ProgressBarFill"
    ProgressBarFill.Size = UDim2.new(0, 0, 1, 0)
    ProgressBarFill.BackgroundColor3 = Colors.ActiveBG
    ProgressBarFill.BackgroundTransparency = 0
    ProgressBarFill.BorderSizePixel = 0
    ProgressBarFill.Parent = ProgressBarBG
    local pbfc = Instance.new("UICorner") ; pbfc.CornerRadius = UDim.new(1,0) ; pbfc.Parent = ProgressBarFill
    ProgressBarFillRef = ProgressBarFill

    local DupePercentLabel = Instance.new("TextLabel")
    DupePercentLabel.Name = "DupePercentLabel"
    DupePercentLabel.Size = UDim2.new(0, 40, 0, 14)
    DupePercentLabel.Position = UDim2.new(0.5, -20, 0, 0)
    DupePercentLabel.BackgroundTransparency = 1
    DupePercentLabel.Text = "0%"
    DupePercentLabel.TextColor3 = Color3.fromRGB(255,255,255)
    DupePercentLabel.TextSize = 9
    DupePercentLabel.Font = Enum.Font.GothamBold
    DupePercentLabel.ZIndex = 5
    DupePercentLabel.Parent = ProgressBarBG

    local DupeInfoLabel = Instance.new("TextLabel")
    DupeInfoLabel.Name = "DupeInfoLabel"
    DupeInfoLabel.Size = UDim2.new(1, 0, 0, 18)
    DupeInfoLabel.BackgroundColor3 = Colors.Panel
    DupeInfoLabel.BackgroundTransparency = 0
    DupeInfoLabel.BorderSizePixel = 0
    DupeInfoLabel.Text = "  Ready to build"
    DupeInfoLabel.TextColor3 = Colors.Muted
    DupeInfoLabel.TextSize = 10
    DupeInfoLabel.Font = Enum.Font.GothamSemibold
    DupeInfoLabel.TextXAlignment = Enum.TextXAlignment.Left
    DupeInfoLabel.Parent = DupeInfoFrame
    local dilc = Instance.new("UICorner") ; dilc.CornerRadius = UDim.new(0,3) ; dilc.Parent = DupeInfoLabel
    DupeInfoLabelRef = DupeInfoLabel
    DupePercentLabelRef = DupePercentLabel

    local InfProgressWrap = Instance.new("Frame")
    InfProgressWrap.Name = "InfProgressWrap"
    InfProgressWrap.Size = UDim2.new(1, 0, 0, 14)
    InfProgressWrap.BackgroundTransparency = 1
    InfProgressWrap.Visible = false
    InfProgressWrap.Parent = T1frame

    local InfProgressBG = Instance.new("Frame")
    InfProgressBG.Name = "InfProgressBG"
    InfProgressBG.Size = UDim2.new(1, 0, 0, 5)
    InfProgressBG.Position = UDim2.new(0, 0, 1, -6)
    InfProgressBG.BackgroundColor3 = Colors.PanelElevated
    InfProgressBG.BackgroundTransparency = 0
    InfProgressBG.BorderSizePixel = 0
    InfProgressBG.Parent = InfProgressWrap
    local ipbc = Instance.new("UICorner") ; ipbc.CornerRadius = UDim.new(1,0) ; ipbc.Parent = InfProgressBG

    local InfProgressFill = Instance.new("Frame")
    InfProgressFill.Name = "InfProgressFill"
    InfProgressFill.Size = UDim2.new(0, 0, 1, 0)
    InfProgressFill.BackgroundColor3 = Colors.ActiveBG
    InfProgressFill.BorderSizePixel = 0
    InfProgressFill.Parent = InfProgressBG
    local ipfc = Instance.new("UICorner") ; ipfc.CornerRadius = UDim.new(1,0) ; ipfc.Parent = InfProgressFill

    local InfProgressLabel = Instance.new("TextLabel")
    InfProgressLabel.Name = "InfProgressLabel"
    InfProgressLabel.Size = UDim2.new(1, 0, 0, 10)
    InfProgressLabel.Position = UDim2.new(0, 0, 0, 0)
    InfProgressLabel.BackgroundTransparency = 1
    InfProgressLabel.Text = "INF 0%"
    InfProgressLabel.TextColor3 = Colors.Muted
    InfProgressLabel.TextSize = 9
    InfProgressLabel.Font = Enum.Font.GothamBold
    InfProgressLabel.TextXAlignment = Enum.TextXAlignment.Center
    InfProgressLabel.Parent = InfProgressWrap
    InfProgressLabelRef = InfProgressLabel
    InfProgressFillRef = InfProgressFill

    local updateBlocksDisplay, updateObjectsList

    local exSubBar = Instance.new("Frame")
    exSubBar.Size = UDim2.new(1, -6, 0, 28)
    exSubBar.BackgroundColor3 = Colors.PanelSoft
    exSubBar.BackgroundTransparency = 0
    exSubBar.BorderSizePixel = 0
    exSubBar.Parent = T3frame
    local exSubLayout = Instance.new("UIListLayout")
    exSubLayout.FillDirection = Enum.FillDirection.Horizontal
    exSubLayout.Padding = UDim.new(0, 2)
    exSubLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    exSubLayout.Parent = exSubBar
    local exSubPad = Instance.new("UIPadding")
    exSubPad.PaddingTop = UDim.new(0,3)
    exSubPad.PaddingBottom = UDim.new(0, 3)
    exSubPad.PaddingLeft = UDim.new(0, 3)
    exSubPad.PaddingRight = UDim.new(0, 3)
    exSubPad.Parent = exSubBar
    local exContent = Instance.new("Frame")
    exContent.Size = UDim2.new(1, -4, 1, -36)
    exContent.Position = UDim2.new(0, 0, 0, 34)
    exContent.BackgroundTransparency = 1
    exContent.Parent = T3frame

    function makeExSub(name, label)
        local btn = Instance.new("TextButton")
        btn.Name = name .. "ExBtn"
        btn.Size = UDim2.new(0.188, -2, 1, 0)
        btn.BackgroundColor3 = Colors.PanelElevated
        btn.BackgroundTransparency = 0
        btn.BorderSizePixel = 0
        btn.Text = label
        btn.TextColor3 = Colors.Muted
        btn.TextSize = 9
        btn.Font = Enum.Font.GothamSemibold
        btn.Parent = exSubBar
        local bc = Instance.new("UICorner")
        bc.CornerRadius = UDim.new(0, 3)
        bc.Parent = btn

        local fr = Instance.new("ScrollingFrame")
        fr.Name = name .. "ExFrame"
        fr.Size = UDim2.new(1, 0, 1, 0)
        fr.BackgroundTransparency = 1
        fr.ScrollBarThickness = 0
        fr.ScrollBarImageColor3 = Colors.Muted
        fr.CanvasSize = UDim2.new(0,0,0,0)
        pcall(function() fr.ElasticBehavior = Enum.ElasticBehavior.Never end)
        fr.Visible = false
        fr.Parent = exContent

        local fl = Instance.new("UIListLayout")
        fl.Padding = UDim.new(0, 5)
        fl.SortOrder = Enum.SortOrder.LayoutOrder
        fl.Parent = fr
        local _exResizeGuard = false
        fl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            if _exResizeGuard then return end
            _exResizeGuard = true
            setScrollCanvas(fr, fl.AbsoluteContentSize.Y, 10)
            task.defer(function() _exResizeGuard = false end)
        end)

        btn.MouseButton1Click:Connect(function()
            for _, f in pairs(exContent:GetChildren()) do if f:IsA("ScrollingFrame") then f.Visible = false end end
            for _, b in pairs(exSubBar:GetChildren()) do
                if b:IsA("TextButton") then
                    b.BackgroundColor3 = Colors.PanelElevated
                    b.TextColor3 = Colors.Muted
                    local og = b:FindFirstChild("SPRB_SubGrad")
                    if og then og:Destroy() end
                end
            end
            fr.Visible = true
            btn.BackgroundColor3 = Colors.ActiveBG
            btn.TextColor3 = Colors.ActiveText
            local sg = Instance.new("UIGradient")
            sg.Name = "SPRB_SubGrad"
            sg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
            sg.Rotation = 90
            sg.Parent = btn
        end)

        return btn, fr
    end

    task.wait()
    local objBtn, objFr = makeExSub("Obj", "CONV")
    local infBtn, infFr = makeExSub("Inf", "INF")
    local movBtn, movFr = makeExSub("Mov", "MOVE")
    local miscBtn, miscFr = makeExSub("Misc", "MISC")
    local rainBtn, rainFr = makeExSub("Shape", "SHAPE")

    task.spawn(function()
        task.wait(0.1)
        for _, f in pairs(exContent:GetChildren()) do if f:IsA("ScrollingFrame") then f.Visible = false end end
        objFr.Visible = true
        objBtn.BackgroundColor3 = Colors.ActiveBG
        objBtn.TextColor3 = Colors.ActiveText
        local ig = Instance.new("UIGradient")
        ig.Name = "SPRB_SubGrad"
        ig.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
            ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
        })
        ig.Rotation = 90
        ig.Parent = objBtn
    end)

    local buildSubBar = Instance.new("Frame")
    buildSubBar.Size = UDim2.new(1, -6, 0, 24)
    buildSubBar.BackgroundColor3 = Colors.PanelSoft
    buildSubBar.BackgroundTransparency = 0
    buildSubBar.BorderSizePixel = 0
    buildSubBar.Parent = T1frame
    local bSubLayout = Instance.new("UIListLayout")
    bSubLayout.FillDirection = Enum.FillDirection.Horizontal
    bSubLayout.Padding = UDim.new(0, 2)
    bSubLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    bSubLayout.Parent = buildSubBar
    local bSubPad = Instance.new("UIPadding")
    bSubPad.PaddingTop = UDim.new(0, 2)
    bSubPad.PaddingBottom = UDim.new(0, 2)
    bSubPad.PaddingLeft = UDim.new(0, 2)
    bSubPad.PaddingRight = UDim.new(0, 2)
    bSubPad.Parent = buildSubBar

    local buildContent = Instance.new("Frame")
    buildContent.Size = UDim2.new(1, -4, 1, -30)
    buildContent.Position = UDim2.new(0, 0, 0, 26)
    buildContent.BackgroundTransparency = 1
    buildContent.Parent = T1frame

    function makeBuildSub(name, label)
        local btn = Instance.new("TextButton")
        btn.Name = name .. "BSubBtn"
        btn.Size = UDim2.new(0.5, -1, 1, 0)
        btn.BackgroundColor3 = Colors.PanelElevated
        btn.BackgroundTransparency = 0
        btn.BorderSizePixel = 0
        btn.Text = label
        btn.TextColor3 = Colors.Muted
        btn.TextSize = 9
        btn.Font = Enum.Font.GothamSemibold
        btn.Parent = buildSubBar
        local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(0, 3); bc.Parent = btn
        local fr = Instance.new("ScrollingFrame")
        fr.Name = name .. "BSubFrame"
        fr.Size = UDim2.new(1, 0, 1, 0)
        fr.BackgroundTransparency = 1
        fr.ScrollBarThickness = 0
        fr.ScrollBarImageColor3 = Colors.Muted
        fr.CanvasSize = UDim2.new(0,0,0,0)
        pcall(function() fr.ElasticBehavior = Enum.ElasticBehavior.Never end)
        fr.Visible = false
        fr.Parent = buildContent
        local fl = Instance.new("UIListLayout")
        fl.Padding = UDim.new(0, 5)
        fl.SortOrder = Enum.SortOrder.LayoutOrder
        fl.Parent = fr
        local _buildResizeGuard = false
        fl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            if _buildResizeGuard then return end
            _buildResizeGuard = true
            setScrollCanvas(fr, fl.AbsoluteContentSize.Y, 10)
            task.defer(function() _buildResizeGuard = false end)
        end)
        btn.MouseButton1Click:Connect(function()
            for _, f in pairs(buildContent:GetChildren()) do if f:IsA("ScrollingFrame") then f.Visible = false end end
            for _, b in pairs(buildSubBar:GetChildren()) do
                if b:IsA("TextButton") then
                    b.BackgroundColor3 = Colors.PanelElevated
                    b.TextColor3 = Colors.Muted
                    local og = b:FindFirstChild("SPRB_SubGrad")
                    if og then og:Destroy() end
                end
            end
            fr.Visible = true
            btn.BackgroundColor3 = Colors.ActiveBG
            btn.TextColor3 = Colors.ActiveText
            local sg = Instance.new("UIGradient")
            sg.Name = "SPRB_SubGrad"
            sg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
            sg.Rotation = 90
            sg.Parent = btn
        end)
        return btn, fr
    end

    local farmWin = nil
    local farmSettings = { step = 2, tgToken = "", tgChatID = "", tgInterval = 0, renderEnabled = true, autoHop = false, autoFarm = false, autoFarmFile = "", autoBuild = false, autoBuildFile = "" }
    local saveFarmSettings = nil
    customTools = nil
    _openReplaceGUI = nil
    _openPaintGUI = nil
    local stealerDropFrame


    local buildBuildBtn, buildBuildFrame = makeBuildSub("Build", "BUILD")
    local buildStealBtn, buildStealFrame = makeBuildSub("Saver", "SAVER")
    buildBuildFrame.Visible = true
    buildBuildBtn.BackgroundColor3 = Colors.ActiveBG
    buildBuildBtn.TextColor3 = Colors.ActiveText
    do
        local ig = Instance.new("UIGradient")
        ig.Name = "SPRB_SubGrad"
        ig.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
            ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
        })
        ig.Rotation = 90
        ig.Parent = buildBuildBtn
    end

    local function createBuildContent()
    makeLabel("PLAYER", buildStealFrame)
    local playerDD, refreshPlayers = makeDropdown("PlayerDD", getPlayerList, buildStealFrame, function(pName)
        selectedPlayer = Players:FindFirstChild(pName)
    end)

    makeLabel("BUILD FILES", buildBuildFrame)
    local fileDD, refreshFiles = makeDropdown("FileDD", getSavedBuilds, buildBuildFrame, function(fName)
        local finp = buildStealFrame:FindFirstChild("FileInputFrame") and buildStealFrame.FileInputFrame:FindFirstChild("FileInput")
        if finp then finp.Text = fName end
        task.spawn(function()
            local ok, err = pcall(function()
                local lb, lf = loadBuildFromFile(fName)
                if lb then
                    if lf == "Asu" then currentBuild = convertAsuToPRS(lb) else currentBuild = lb end
                    if currentBuild then
                        if Settings.autoPreview then createPreview(currentBuild) end
                        if updateBlocksDisplayGlobal then updateBlocksDisplayGlobal() end
                        if lf == "Asu" then setStatus("  Loaded ASU (save creates BH, ASU not updated)") end
                    end
                end
            end)
            if not ok then setStatus("  Load error: " .. tostring(err)) end
        end)
    end)

    makeBtn("RefreshFilesBtn", "Refresh File List", buildBuildFrame, function()
        invalidateSearchCache()
        _buildFilesCache = nil
        refreshFiles()
    end)

    do
        local babftRow = Instance.new("Frame")
        babftRow.Size = UDim2.new(1, 0, 0, 22)
        babftRow.BackgroundTransparency = 1
        babftRow.Parent = buildBuildFrame
        local babftInfo = Instance.new("TextLabel")
        babftInfo.Size = UDim2.new(1, -90, 1, 0)
        babftInfo.BackgroundTransparency = 1
        babftInfo.Text = "Get build files from TG:"
        babftInfo.TextColor3 = Colors.Muted
        babftInfo.TextSize = 11
        babftInfo.Font = Enum.Font.GothamMedium
        babftInfo.TextXAlignment = Enum.TextXAlignment.Left
        babftInfo.Parent = babftRow
        local babftBtn = Instance.new("TextButton")
        babftBtn.Size = UDim2.new(0, 80, 1, 0)
        babftBtn.Position = UDim2.new(1, -80, 0, 0)
        babftBtn.BackgroundTransparency = 1
        babftBtn.Text = "@babft"
        babftBtn.TextColor3 = Color3.fromRGB(60, 130, 230)
        babftBtn.TextSize = 12
        babftBtn.Font = Enum.Font.GothamBold
        babftBtn.TextXAlignment = Enum.TextXAlignment.Right
        babftBtn.AutoButtonColor = false
        babftBtn.Parent = babftRow
        babftBtn.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            pcall(function() setclipboard("@babft") end)
            babftBtn.Text = "Copied!"
            babftBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
            task.delay(1.5, function()
                if babftBtn and babftBtn.Parent then
                    babftBtn.Text = "@babft"
                    babftBtn.TextColor3 = Color3.fromRGB(60, 130, 230)
                end
            end)
        end)
    end

    makeLabel("FILE NAME", buildStealFrame)
    local fileInput = makeInput("FileInput", "Enter file name...", buildStealFrame)

    makeLabel("OBJECTS", buildBuildFrame)
    local objListFrame = Instance.new("ScrollingFrame")
    objListFrame.Name = "ObjList"
    objListFrame.Size = UDim2.new(1, 0, 0, 120)
    objListFrame.BackgroundColor3 = Colors.Panel
    objListFrame.BackgroundTransparency = 0
    objListFrame.BorderSizePixel = 0
    objListFrame.ScrollBarThickness = 0
    objListFrame.ScrollBarImageColor3 = Colors.Muted
    objListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    objListFrame.Parent = buildBuildFrame
    local olfc = Instance.new("UICorner")
    olfc.CornerRadius = UDim.new(0, 5)
    olfc.Parent = objListFrame
    local oll = Instance.new("UIListLayout")
    oll.Padding = UDim.new(0, 2)
    oll.SortOrder = Enum.SortOrder.LayoutOrder
    oll.Parent = objListFrame
    local olp = Instance.new("UIPadding")
    olp.PaddingTop = UDim.new(0, 3)
    olp.PaddingBottom = UDim.new(0, 3)
    olp.PaddingLeft = UDim.new(0, 3)
    olp.PaddingRight = UDim.new(0, 3)
    olp.Parent = objListFrame
    oll:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        setScrollCanvas(objListFrame, oll.AbsoluteContentSize.Y, 8)
    end)

    updateObjectsList = function()
        for _, c in pairs(objListFrame:GetChildren()) do if c:IsA("TextButton") then c:Destroy() end end
        if not currentBuild or not next(currentBuild) then return end
        for blockName, blocks in pairs(currentBuild) do
            local cnt = type(blocks) == "table" and #blocks or 0
            local ob = Instance.new("TextButton")
            ob.Size = UDim2.new(1, -4, 0, 26)
            ob.BackgroundColor3 = Colors.PanelElevated
            ob.BackgroundTransparency = 0
            ob.BorderSizePixel = 0
            ob.Text = "  " .. blockName .. " (" .. cnt .. ")"
            ob.TextColor3 = Colors.Text
            ob.TextSize = 11
            ob.Font = Enum.Font.Gotham
            ob.TextXAlignment = Enum.TextXAlignment.Left
            ob.Parent = objListFrame
            local obc = Instance.new("UICorner")
            obc.CornerRadius = UDim.new(0, 4)
            obc.Parent = ob
            ob.MouseButton1Click:Connect(function()
                for _, b in pairs(objListFrame:GetChildren()) do
                    if b:IsA("TextButton") then
                        b.BackgroundColor3 = Colors.PanelElevated
                        b.TextColor3 = Colors.Text
                    end
                end
                ob.BackgroundColor3 = Colors.ActiveBG
                ob.TextColor3 = Colors.ActiveText
                selectedObjectName = blockName
                updateSelectionHighlight(blockName)
            end)
        end
    end

    makeLabel("ACTIONS", buildBuildFrame)

    local function getCurrentBuildCount()
        local total = 0
        if currentBuild then
            for blockName, bs in pairs(currentBuild) do
                if type(bs) == "table" then
                    local regular = isRegularBlock(blockName)
                    for _, bi in pairs(bs) do
                        local sz = nil
                        if regular and bi and bi.Size and bi.Size ~= "" then
                            sz = strV3(bi.Size) * (Settings.buildScale or 1)
                        end
                        total = total + (regular and calcSlots(sz) or 1)
                    end
                end
            end
        end
        return total
    end

    task.wait()
    local function refreshMiniCounter(label)
        if not label then return end
        local total = getCurrentBuildCount()
        local placed = 0
        local myBlocks = BlocksFolder:FindFirstChild(LocalPlayer.Name)
        if myBlocks then placed = #myBlocks:GetChildren() end
        label.Text = "Blocks: " .. total .. " | Placed: " .. placed
    end

    local function runBuildFromFile(progBar, counterLabel)
        if isBuilding then setStatus("  Already building!") ; return end
        local fn = fileInput.Text
        if fn == "" then setStatus("  Enter a file name") ; return end
        setStatus("  Loading " .. fn .. "...")
        local lb, lf = loadBuildFromFile(fn)
        if not lb then setStatus("  File not found: " .. fn) ; return end
        if lf == "Asu" then currentBuild = convertAsuToPRS(lb) else currentBuild = lb end
        if not currentBuild or not next(currentBuild) then setStatus("  Build empty") ; return end
        if getCurrentBuildCount() == 0 then setStatus("  No blocks found") ; return end


        if not Settings.infBlockEnabled then
            local totalNeeded = 0
            local totalHave = 0
            for blockName, blocks in pairs(currentBuild) do
                if type(blocks) == "table" then
                    local regular = isRegularBlock(blockName)
                    local needed = 0
                    for _, bi in pairs(blocks) do
                        local sz = nil
                        if regular and bi and bi.Size and bi.Size ~= "" then
                            sz = strV3(bi.Size) * (Settings.buildScale or 1)
                        end
                        needed = needed + (regular and calcSlots(sz) or 1)
                    end
                    local have = getRealBlockCount(blockName)
                    totalNeeded = totalNeeded + needed
                    totalHave = totalHave + math.min(have, needed)
                end
            end
            if totalNeeded > totalHave and totalHave > 0 then
                local pct = math.floor(totalHave / totalNeeded * 100)
                local warnGui = Instance.new("ScreenGui")
                warnGui.Name = "BlockLimitWarning"; warnGui.ResetOnSpawn = false
                warnGui.IgnoreGuiInset = true
                pcall(function() warnGui.Parent = LocalPlayer.PlayerGui end)
                local bg = Instance.new("Frame"); bg.Size = UDim2.new(1,0,1,0); bg.BackgroundColor3 = Color3.new(0,0,0); bg.BackgroundTransparency = 0.5; bg.Parent = warnGui
                local card = Instance.new("Frame"); card.Size = UDim2.new(0,320,0,150); card.Position = UDim2.new(0.5,-160,0.5,-75); card.BackgroundColor3 = Colors.BG; card.BorderSizePixel = 0; card.Parent = warnGui
                Instance.new("UICorner", card).CornerRadius = UDim.new(0,8)
                local cSt = Instance.new("UIStroke"); cSt.Color = Colors.Red; cSt.Thickness = 1.5; cSt.Transparency = 0.3; cSt.Parent = card
                local title = Instance.new("TextLabel"); title.Size = UDim2.new(1,-16,0,24); title.Position = UDim2.new(0,8,0,10); title.BackgroundTransparency = 1; title.Text = "NOT ENOUGH BLOCKS"; title.TextColor3 = Colors.Red; title.TextSize = 15; title.Font = Enum.Font.GothamBold; title.TextXAlignment = Enum.TextXAlignment.Left; title.Parent = card
                local msg = Instance.new("TextLabel"); msg.Size = UDim2.new(1,-16,0,36); msg.Position = UDim2.new(0,8,0,36); msg.BackgroundTransparency = 1; msg.Text = "Need: " .. totalNeeded .. " | Have: " .. totalHave .. " (" .. pct .. "%)"; msg.TextColor3 = Colors.Muted; msg.TextSize = 13; msg.Font = Enum.Font.GothamMedium; msg.TextXAlignment = Enum.TextXAlignment.Left; msg.TextWrapped = true; msg.Parent = card
                local okBtn = Instance.new("TextButton"); okBtn.Size = UDim2.new(0,100,0,32); okBtn.Position = UDim2.new(0,12,0,86); okBtn.BackgroundColor3 = Color3.fromRGB(180,50,20); okBtn.TextColor3 = Color3.fromRGB(255,220,220); okBtn.Text = "OK"; okBtn.TextSize = 13; okBtn.Font = Enum.Font.GothamBold; okBtn.BorderSizePixel = 0; okBtn.AutoButtonColor = true; okBtn.Parent = card
                Instance.new("UICorner", okBtn).CornerRadius = UDim.new(0,6)
                local infHint = Instance.new("TextLabel"); infHint.Size = UDim2.new(1,-16,0,16); infHint.Position = UDim2.new(0,8,0,120); infHint.BackgroundTransparency = 1; infHint.Text = "Enable INF BLOCKS to place without limits"; infHint.TextColor3 = Color3.fromRGB(255,180,80); infHint.TextSize = 10; infHint.Font = Enum.Font.GothamBold; infHint.TextXAlignment = Enum.TextXAlignment.Left; infHint.Parent = card
                local enBtn = Instance.new("TextButton"); enBtn.Size = UDim2.new(0,140,0,32); enBtn.Position = UDim2.new(1,-152,0,86); enBtn.BackgroundColor3 = Color3.fromRGB(30,80,30); enBtn.TextColor3 = Color3.fromRGB(140,255,140); enBtn.Text = "ENABLE"; enBtn.TextSize = 13; enBtn.Font = Enum.Font.GothamBold; enBtn.BorderSizePixel = 0; enBtn.AutoButtonColor = true; enBtn.Parent = card
                Instance.new("UICorner", enBtn).CornerRadius = UDim.new(0,6)
                local warned = false
                local cancelled = false
                local function closeWarn(cancel) cancelled = cancel or false; warned = true; pcall(function() warnGui:Destroy() end) end
                okBtn.MouseButton1Click:Connect(function() closeWarn(false) end)
                enBtn.MouseButton1Click:Connect(function()
                    Settings.infBlockEnabled = true
                    pcall(function()
                        local b = infFr and infFr:FindFirstChild("InfBlockToggle")
                        if b then b.Text = "Inf Block: ON"; b.BackgroundColor3 = Color3.fromRGB(16,32,16) end
                    end)
                    saveSettings()
                    closeWarn(false)
                end)
                bg.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then closeWarn(false) end end)
                local t0 = tick()
                while not warned and tick() - t0 < 30 do task.wait(0.1) end
                closeWarn(false)
                if cancelled then setStatus("  Build cancelled"); return end
            end
        end
        updateObjectsList()
        refreshMiniCounter(counterLabel)
        if updateBlocksDisplayGlobal then updateBlocksDisplayGlobal() end
        if progBar then progBar.Size = UDim2.new(0, 0, 1, 0) end
        setStatus("  Keep all tools equipped until build finishes")
        task.wait(0.8)
        local buildOk, placedIds
        buildOk, buildErr = pcall(function()
            local ok, ids = pasteBuild(currentBuild, function(msg, pct)
                setStatus("  " .. msg)
                refreshMiniCounter(counterLabel)
                if progBar then
                    TweenService:Create(progBar, TweenInfo.new(0.12), {Size = UDim2.new(math.clamp(pct/100,0,1), 0, 1, 0)}):Play()
                end
            end)
            placedIds = ids
            return ok
        end)
        if not buildOk then
            setStatus("  Build error: " .. tostring(buildErr))
            playUISound(UISoundConfig.error)
        elseif stopBuild then
            setStatus("  Build stopped")
        else
            playUISound(UISoundConfig.success)

            if placedIds then
                recentlyPlacedBlocks = {}
                local cnt = 0
                for _, blk in pairs(placedIds) do
                    if type(blk) == "userdata" and blk:FindFirstChild("PPart") then
                        recentlyPlacedBlocks[blk] = true
                        cnt = cnt + 1
                    end
                end
                if _RG then _RG.recentlyPlacedBlocks = recentlyPlacedBlocks end
            end
        end

        stopBuild = false
        isBuilding = false
        pcall(function()
            local sf = LocalPlayer:FindFirstChild("Settings")
            if sf then
                local sb = sf:FindFirstChild("ShareBlocks")
                if sb then sb.Value = shareBlocksOriginal end
            end
        end)
        refreshMiniCounter(counterLabel)
    end

    makeBtn("StealBuildBtn", "Save Build", buildStealFrame, function()
        setStatus("  Save Build - copy from player")
    end)

    makeBtn("BuildBtn", "Build", buildBuildFrame, function()
        local dupeInfo = buildBuildFrame:FindFirstChild("DupeInfoFrame")
        local progBG = dupeInfo and dupeInfo:FindFirstChild("ProgressBarBG")
        local progBar = progBG and progBG:FindFirstChild("ProgressBarFill")
        runBuildFromFile(progBar)
    end)

    makeBtn("StopBuildBtn", "Stop Build", buildBuildFrame, function()
        if isBuilding then
            stopBuild = true
            setStatus("  Stopping...")
        else
            setStatus("  Not building")
        end
    end)

    local PreviewBtn = makeBtn("PreviewBtn", "Preview", buildBuildFrame, function()
        if not currentBuild or not next(currentBuild) then setStatus("  No build loaded") ; return end
        if previewActive then
            clearPreview()
            updateObjectsList()
            setStatus("  Preview cleared")
        else
            createPreview(currentBuild)
            updateObjectsList()
            setStatus("  Preview created")
        end
    end)
    updatePreviewButtonGlobal = function()
        if PreviewBtn then PreviewBtn.Text = previewActive and "Clear Preview" or "Preview" end
    end
    local DockCounter = Instance.new("TextLabel")
    DockCounter.Name = "AutoBuildCounter"
    DockCounter.Size = UDim2.new(1, 0, 0, 26)
    DockCounter.BackgroundColor3 = Colors.PanelSoft
    DockCounter.BackgroundTransparency = 0
    DockCounter.BorderSizePixel = 0
    DockCounter.Text = "  Blocks: 0 | Placed: 0"
    DockCounter.TextColor3 = Colors.Text
    DockCounter.TextSize = 11
    DockCounter.Font = Enum.Font.GothamSemibold
    DockCounter.TextXAlignment = Enum.TextXAlignment.Left
    DockCounter.Parent = T1frame
    stylizeCard(DockCounter, Colors.PanelSoft, Colors.Border, 3)

    RunService.Heartbeat:Connect(function()
        if DockCounter and DockCounter.Parent and T1frame.Visible then
            refreshMiniCounter(DockCounter)
            DockCounter.Text = "  " .. DockCounter.Text
        end
    end)

    makeLabel("BUILD SCALE / OFFSET", buildBuildFrame)
    local bsInput = makeInput("BuildScale", "Scale (default 1.0)", buildBuildFrame)
    bsInput.Text = tostring(Settings.buildScale)
    bsInput.FocusLost:Connect(function()
        local v = tonumber(bsInput.Text)
        if v and v >= 0.01 and v <= 20 then
            Settings.buildScale = v
            if previewActive and currentBuild then createPreview(currentBuild, selectedObjectName) end
        else bsInput.Text = tostring(Settings.buildScale) end
    end)

    local function makeOffsetInput(axis, key)
        local inp = makeInput(axis.."Off", axis .. " Offset", buildBuildFrame)
        inp.Text = tostring(Settings[key])
        inp.FocusLost:Connect(function()
            local v = tonumber(inp.Text)
            if v then
                Settings[key] = v
                if previewActive and currentBuild then createPreview(currentBuild, selectedObjectName) end
            else inp.Text = tostring(Settings[key]) end
        end)
    end
    makeOffsetInput("X", "buildOffsetX")
    makeOffsetInput("Y", "buildOffsetY")
    makeOffsetInput("Z", "buildOffsetZ")

    makeLabel("BUILD SPEED", buildBuildFrame)
    local speedInput = makeInput("BuildSpeed", "0=instant, 1-10=delay", buildBuildFrame)
    speedInput.Text = tostring(Settings.buildSpeed)
    speedInput.FocusLost:Connect(function()
        local v = tonumber(speedInput.Text)
        if v and v >= 0 and v <= 10 then
            Settings.buildSpeed = v
        else speedInput.Text = tostring(Settings.buildSpeed) end
    end)

    makeLabel("EXCLUDED BLOCKS (click to toggle)", buildBuildFrame)
    local exclScroll = Instance.new("ScrollingFrame")
    exclScroll.Name = "ExclScroll"
    exclScroll.Size = UDim2.new(1, 0, 0, 100)
    exclScroll.BackgroundColor3 = Colors.Panel
    exclScroll.BackgroundTransparency = 0
    exclScroll.BorderSizePixel = 0
    exclScroll.ScrollBarThickness = 0
    exclScroll.ScrollBarImageColor3 = Colors.Muted
    exclScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    exclScroll.Parent = buildBuildFrame
    local esc = Instance.new("UICorner")
    esc.CornerRadius = UDim.new(0, 5)
    esc.Parent = exclScroll
    local esl = Instance.new("UIListLayout")
    esl.Padding = UDim.new(0, 2)
    esl.SortOrder = Enum.SortOrder.LayoutOrder
    esl.Parent = exclScroll
    local esp = Instance.new("UIPadding")
    esp.PaddingTop = UDim.new(0, 3)
    esp.PaddingBottom = UDim.new(0, 3)
    esp.PaddingLeft = UDim.new(0, 3)
    esp.PaddingRight = UDim.new(0, 3)
    esp.Parent = exclScroll
    esl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        setScrollCanvas(exclScroll, esl.AbsoluteContentSize.Y, 8)
    end)

    local function refreshExclusionList()
        for _, c in pairs(exclScroll:GetChildren()) do if c:IsA("TextButton") then c:Destroy() end end
        if not currentBuild or not next(currentBuild) then return end
        for blockName, blocks in pairs(currentBuild) do
            local cnt = type(blocks) == "table" and #blocks or 0
            local isExcl = Settings.excludedBlocks[blockName] or false
            local ob = Instance.new("TextButton")
            ob.Size = UDim2.new(1, -4, 0, 22)
            ob.BackgroundColor3 = isExcl and Color3.fromRGB(60, 16, 16) or Colors.PanelElevated
            ob.BackgroundTransparency = 0
            ob.BorderSizePixel = 0
            ob.Text = "  " .. (isExcl and "[X] " or "[ ] ") .. blockName .. " (" .. cnt .. ")"
            ob.TextColor3 = isExcl and Color3.fromRGB(255, 120, 120) or Colors.Text
            ob.TextSize = 10
            ob.Font = Enum.Font.Gotham
            ob.TextXAlignment = Enum.TextXAlignment.Left
            ob.Parent = exclScroll
            local obc = Instance.new("UICorner")
            obc.CornerRadius = UDim.new(0, 4)
            obc.Parent = ob
            ob.MouseButton1Click:Connect(function()
                if Settings.excludedBlocks[blockName] then
                    Settings.excludedBlocks[blockName] = nil
                else
                    Settings.excludedBlocks[blockName] = true
                end
                refreshExclusionList()
            end)
        end
    end

    function updateBlocksDisplay()
        for _, c in pairs(T2frame:GetChildren()) do
            if c.Name == "BlocksScrollFrame" or c.Name == "BlocksInfoLabel" then
                c:Destroy()
            end
        end
        if not currentBuild or not next(currentBuild) then return end

        local totalNeeded = 0
        local totalHave = 0
        local blockStats = {}
        for blockName, blocks in pairs(currentBuild) do
            if type(blocks) == "table" then
                local regular = isRegularBlock(blockName)
                local needed = 0
                for _, bi in pairs(blocks) do
                    local sz = nil
                    if regular and bi and bi.Size and bi.Size ~= "" then
                        sz = strV3(bi.Size) * (Settings.buildScale or 1)
                    end
                    needed = needed + (regular and calcSlots(sz) or 1)
                end
                local have = getRealBlockCount(blockName)
                totalNeeded = totalNeeded + needed
                totalHave = totalHave + math.min(have, needed)
                blockStats[blockName] = {needed=needed, have=have, count=#blocks, regular=regular}
            end
        end

        local infLabel = Instance.new("TextLabel")
        infLabel.Name = "BlocksInfoLabel"
        infLabel.Size = UDim2.new(1, 0, 0, 22)
        infLabel.BackgroundColor3 = Color3.fromRGB(14,14,14)
        infLabel.BackgroundTransparency = 0
        infLabel.BorderSizePixel = 0
        infLabel.TextColor3 = Colors.Muted
        infLabel.TextSize = 10
        infLabel.Font = Enum.Font.GothamSemibold
        infLabel.TextXAlignment = Enum.TextXAlignment.Left
        local maxInvSlot = 0
        for blockName, stat in pairs(blockStats) do
            if stat.have > maxInvSlot then maxInvSlot = stat.have end
        end
        local partsNeeded = maxInvSlot > 0 and math.ceil(totalNeeded / maxInvSlot) or "?"
        if Settings.infBlockEnabled then
            infLabel.Text = "  INF: ~" .. partsNeeded .. " parts | Total blocks: " .. totalNeeded
            infLabel.TextColor3 = Color3.fromRGB(180, 140, 50)
        else
            local pct = totalNeeded > 0 and math.floor(math.min(totalHave,totalNeeded)/totalNeeded*100) or 0
            infLabel.Text = "  Total: " .. totalNeeded .. " blocks  |  Have: " .. pct .. "%"
            infLabel.TextColor3 = pct >= 100 and Colors.Green or Colors.Muted
        end
        local ilc = Instance.new("UICorner") ; ilc.CornerRadius = UDim.new(0,4) ; ilc.Parent = infLabel
        infLabel.Parent = T2frame

        local bscroll = Instance.new("ScrollingFrame")
        bscroll.Name = "BlocksScrollFrame"
        bscroll.Size = UDim2.new(1, 0, 1, -30)
        bscroll.Position = UDim2.new(0, 0, 0, 28)
        bscroll.BackgroundTransparency = 1
        bscroll.ScrollBarThickness = 0
        bscroll.ScrollBarImageColor3 = Color3.fromRGB(70,70,70)
        bscroll.CanvasSize = UDim2.new(0,0,0,0)
        bscroll.Parent = T2frame

        local bgl = Instance.new("UIGridLayout")
        bgl.CellSize = UDim2.new(0, 90, 0, 115)
        bgl.CellPadding = UDim2.new(0, 4, 0, 4)
        bgl.SortOrder = Enum.SortOrder.LayoutOrder
        bgl.Parent = bscroll
        local bgp = Instance.new("UIPadding")
        bgp.PaddingTop = UDim.new(0, 4) ; bgp.PaddingBottom = UDim.new(0, 4)
        bgp.PaddingLeft = UDim.new(0, 4) ; bgp.PaddingRight = UDim.new(0, 4)
        bgp.Parent = bscroll
        bgl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            setScrollCanvas(bscroll, bgl.AbsoluteContentSize.Y, 10)
        end)

        for blockName, stat in pairs(blockStats) do
            local needed = stat.needed
            local have = stat.have
            local enough = have >= needed

            local bf = Instance.new("Frame")
            bf.BackgroundColor3 = Color3.fromRGB(16,16,16)
            bf.BackgroundTransparency = 0
            bf.BorderSizePixel = 0
            bf.Parent = bscroll
            local bfc = Instance.new("UICorner") ; bfc.CornerRadius = UDim.new(0,5) ; bfc.Parent = bf
            local bfs = Instance.new("UIStroke")
            bfs.Color = enough and Color3.fromRGB(40,90,40) or Color3.fromRGB(90,40,40)
            bfs.Thickness = 1 ; bfs.Parent = bf

            local img = Instance.new("ImageLabel")
            img.Size = UDim2.new(1,-8,0,60)
            img.Position = UDim2.new(0,4,0,4)
            img.BackgroundTransparency = 1
            img.ScaleType = Enum.ScaleType.Fit
            img.Image = "rbxasset://textures/ui/GuiImagePlaceholder.png"
            img.Parent = bf
            pcall(function()
                local bg = LocalPlayer.PlayerGui:FindFirstChild("BuildGui")
                local inv = bg and bg:FindFirstChild("InventoryFrame")
                local sf = inv and inv:FindFirstChild("ScrollingFrame")
                local bframe = sf and sf:FindFirstChild("BlocksFrame")
                local bb = bframe and bframe:FindFirstChild(blockName)
                local ti = bb and bb:FindFirstChild("TypeIcon")
                if ti and ti.Image ~= "" then img.Image = ti.Image end
            end)

            local nl = Instance.new("TextLabel")
            nl.Size = UDim2.new(1,0,0,14)
            nl.Position = UDim2.new(0,0,0,66)
            nl.BackgroundTransparency = 1
            nl.Text = blockName:gsub("Block",""):gsub("([A-Z])"," %1"):match("^%s*(.-)%s*$")
            nl.TextColor3 = Colors.Text
            nl.TextSize = 9 ; nl.Font = Enum.Font.GothamSemibold
            nl.TextScaled = true ; nl.Parent = bf

            local cl = Instance.new("TextLabel")
            cl.Size = UDim2.new(1,0,0,13)
            cl.Position = UDim2.new(0,0,0,81)
            cl.BackgroundTransparency = 1
            cl.Text = needed .. " need / " .. have .. " have"
            cl.TextColor3 = enough and Colors.Green or Colors.Red
            cl.TextSize = 10 ; cl.Font = Enum.Font.GothamSemibold
            cl.TextScaled = true ; cl.Parent = bf

            local bc = Instance.new("TextLabel")
            bc.Size = UDim2.new(1,0,0,11)
            bc.Position = UDim2.new(0,0,0,95)
            bc.BackgroundTransparency = 1
            bc.Text = stat.count .. " part" .. (stat.count~=1 and "s" or "")
            bc.TextColor3 = Colors.Muted
            bc.TextSize = 9 ; bc.Font = Enum.Font.GothamMedium
            bc.TextScaled = true ; bc.Parent = bf

            local pbar = Instance.new("Frame")
            pbar.Size = UDim2.new(1,-8,0,3)
            pbar.Position = UDim2.new(0,4,1,-5)
            pbar.BackgroundColor3 = Color3.fromRGB(35,35,35)
            pbar.BorderSizePixel = 0 ; pbar.Parent = bf
            local pbc = Instance.new("UICorner") ; pbc.CornerRadius = UDim.new(1,0) ; pbc.Parent = pbar
            local pfill = Instance.new("Frame")
            pfill.Size = UDim2.new(math.clamp(have/math.max(needed,1),0,1),0,1,0)
            pfill.BackgroundColor3 = enough and Colors.Green or Colors.Red
            pfill.BorderSizePixel = 0 ; pfill.Parent = pbar
            local pfillc = Instance.new("UICorner") ; pfillc.CornerRadius = UDim.new(1,0) ; pfillc.Parent = pfill

            local isExcl = Settings.excludedBlocks[blockName] or false
            local hasRepl = Settings.blockReplacements and Settings.blockReplacements[blockName] or nil
            local isBlockType = blockName:sub(-5) == "Block"

            local mBtn = Instance.new("TextButton")
            mBtn.Name = "MatRepl_" .. blockName
            mBtn.Size = UDim2.new(0, 16, 0, 16)
            mBtn.Position = UDim2.new(1, -36, 0, 2)
            mBtn.BackgroundColor3 = hasRepl and Color3.fromRGB(30, 80, 140) or Color3.fromRGB(40, 40, 40)
            mBtn.BackgroundTransparency = 0
            mBtn.BorderSizePixel = 0
            mBtn.Text = "M"
            mBtn.TextColor3 = hasRepl and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(120, 120, 120)
            mBtn.TextSize = 10
            mBtn.Font = Enum.Font.GothamBold
            mBtn.ZIndex = 10
            mBtn.Parent = bf
            local mBtnC = Instance.new("UICorner") ; mBtnC.CornerRadius = UDim.new(0, 3) ; mBtnC.Parent = mBtn
            mBtn.MouseButton1Click:Connect(function()
                if activeDropdownClose then pcall(activeDropdownClose) end

                local allOpts = {}
                for _, bp in ipairs(BuildingParts:GetChildren()) do
                    local bn = bp.Name
                    local endsBlock = bn:sub(-5) == "Block"
                    if isBlockType and endsBlock then
                        allOpts[#allOpts+1] = bn
                    elseif not isBlockType and not endsBlock then
                        allOpts[#allOpts+1] = bn
                    end
                end
                table.sort(allOpts, function(a, b) return a:lower() < b:lower() end)

                local container = Instance.new("Frame")
                container.Name = "MatPopup_" .. blockName
                container.BackgroundColor3 = Colors.Panel
                container.BackgroundTransparency = 0
                container.BorderSizePixel = 0
                container.ZIndex = 210
                container.Visible = false
                container.ClipsDescendants = true
                container.Parent = DropdownLayer
                local contC = Instance.new("UICorner") ; contC.CornerRadius = UDim.new(0, 6) ; contC.Parent = container
                local contS = Instance.new("UIStroke") ; contS.Color = Colors.Border ; contS.Thickness = 1 ; contS.Parent = container

                local searchBox = Instance.new("TextBox")
                searchBox.Name = "SearchBox"
                searchBox.Size = UDim2.new(1, -12, 0, 26)
                searchBox.Position = UDim2.new(0, 6, 0, 6)
                searchBox.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
                searchBox.BackgroundTransparency = 0
                searchBox.BorderSizePixel = 0
                searchBox.Text = ""
                searchBox.PlaceholderText = "Search..."
                searchBox.PlaceholderColor3 = Color3.fromRGB(90, 90, 90)
                searchBox.TextColor3 = Color3.fromRGB(220, 220, 220)
                searchBox.TextSize = 12
                searchBox.Font = Enum.Font.Gotham
                searchBox.ZIndex = 212
                searchBox.ClearTextOnFocus = false
                searchBox.Parent = container
                local sbC = Instance.new("UICorner") ; sbC.CornerRadius = UDim.new(0, 4) ; sbC.Parent = searchBox
                local sbP = Instance.new("UIPadding") ; sbP.PaddingLeft = UDim.new(0, 6) ; sbP.Parent = searchBox

                local listFrame = Instance.new("Frame")
                listFrame.Name = "ListFrame"
                listFrame.BackgroundTransparency = 1
                listFrame.ClipsDescendants = true
                listFrame.ZIndex = 211
                listFrame.Parent = container

                local scroll = Instance.new("ScrollingFrame")
                scroll.Name = "Scroll"
                scroll.Size = UDim2.new(1, 0, 1, 0)
                scroll.BackgroundTransparency = 1
                scroll.ScrollBarThickness = 4
                scroll.ScrollBarImageColor3 = Colors.Muted
                scroll.ZIndex = 211
                scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
                scroll.Parent = listFrame
                local sL = Instance.new("UIListLayout") ; sL.Padding = UDim.new(0, 1) ; sL.Parent = scroll
                local sP = Instance.new("UIPadding") ; sP.PaddingTop = UDim.new(0, 2) ; sP.PaddingBottom = UDim.new(0, 4) ; sP.Parent = scroll

                local optionBtns = {}
                local function populateList(filter)
                    for _, ob in ipairs(optionBtns) do pcall(function() ob:Destroy() end) end
                    optionBtns = {}
                    local filtered = {}
                    local fl = filter:lower():gsub("^%s*(.-)%s*$", "%1")
                    for _, opt in ipairs(allOpts) do
                        if fl == "" or opt:lower():find(fl, 1, true) then
                            filtered[#filtered+1] = opt
                        end
                    end
                    for _, opt in ipairs(filtered) do
                        local ob = Instance.new("TextButton")
                        ob.Size = UDim2.new(1, -12, 0, 24)
                        ob.BackgroundColor3 = (hasRepl == opt) and Color3.fromRGB(30, 60, 100) or Colors.PanelElevated
                        ob.BackgroundTransparency = 0
                        ob.BorderSizePixel = 0
                        ob.Text = "  " .. opt
                        ob.TextColor3 = (hasRepl == opt) and Color3.fromRGB(255, 255, 255) or Colors.Text
                        ob.TextSize = 11
                        ob.Font = Enum.Font.Gotham
                        ob.TextXAlignment = Enum.TextXAlignment.Left
                        ob.ZIndex = 212
                        ob.Parent = scroll
                        local obc = Instance.new("UICorner") ; obc.CornerRadius = UDim.new(0, 3) ; obc.Parent = ob
                        ob.MouseEnter:Connect(function()
                            TweenService:Create(ob, TweenInfo.new(0.1), {BackgroundColor3 = (hasRepl == opt) and Color3.fromRGB(40, 75, 120) or Colors.ActiveBG:Lerp(Colors.PanelElevated, 0.6)}):Play()
                        end)
                        ob.MouseLeave:Connect(function()
                            TweenService:Create(ob, TweenInfo.new(0.1), {BackgroundColor3 = (hasRepl == opt) and Color3.fromRGB(30, 60, 100) or Colors.PanelElevated}):Play()
                        end)
                        ob.MouseButton1Click:Connect(function()
                            Settings.blockReplacements = Settings.blockReplacements or {}
                            if hasRepl == opt then
                                Settings.blockReplacements[blockName] = nil
                                mBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
                                mBtn.TextColor3 = Color3.fromRGB(120, 120, 120)
                                nl.Text = blockName:gsub("Block",""):gsub("([A-Z])"," %1"):match("^%s*(.-)%s*$")
                                nl.TextColor3 = Colors.Text
                            else
                                Settings.blockReplacements[blockName] = opt
                                mBtn.BackgroundColor3 = Color3.fromRGB(30, 80, 140)
                                mBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                                nl.Text = opt:gsub("Block",""):gsub("([A-Z])"," %1"):match("^%s*(.-)%s*$")
                                nl.TextColor3 = Color3.fromRGB(100, 180, 255)
                            end
                            saveSettings()
                            closeMatPopup()
                            pcall(updateBlocksDisplay)
                        end)
                        optionBtns[#optionBtns+1] = ob
                    end
                    scroll.CanvasSize = UDim2.new(0, 0, 0, math.max(30, #filtered * 25 + 6))
                end

                searchBox:GetPropertyChangedSignal("Text"):Connect(function()
                    populateList(searchBox.Text)
                end)

                local ap = mBtn.AbsolutePosition
                local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
                local cw = 220
                local ch = 260
                local px = math.clamp(ap.X - cw + 16, 6, vp.X - cw - 6)
                local py = ap.Y + 18
                if py + ch > vp.Y - 6 then py = ap.Y - ch - 4 end
                container.Position = UDim2.new(0, px, 0, py)
                container.Size = UDim2.new(0, cw, 0, ch)
                listFrame.Position = UDim2.new(0, 0, 0, 36)
                listFrame.Size = UDim2.new(1, 0, 1, -40)

                populateList("")
                container.Visible = true
                container.Size = UDim2.new(0, cw, 0, 4)
                TweenService:Create(container, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, cw, 0, ch)}):Play()
                task.defer(function() searchBox:CaptureFocus() end)

                local function closeMatPopup()
                    TweenService:Create(container, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, cw, 0, 4)}):Play()
                    task.delay(0.12, function()
                        pcall(function() container:Destroy() end)
                    end)
                    if activeDropdownClose == closeMatPopup then activeDropdownClose = nil end
                end
                activeDropdownClose = closeMatPopup
                DropdownLayer.InputBegan:Connect(function(inp)
                    if not container.Visible then return end
                    if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
                    local mx, my = inp.Position.X, inp.Position.Y
                    local cp = container.AbsolutePosition
                    local cs = container.AbsoluteSize
                    if mx < cp.X or mx > cp.X + cs.X or my < cp.Y or my > cp.Y + cs.Y then
                        closeMatPopup()
                    end
                end)
            end)

            local xBtn = Instance.new("TextButton")
            xBtn.Name = "ExclX_" .. blockName
            xBtn.Size = UDim2.new(0, 16, 0, 16)
            xBtn.Position = UDim2.new(1, -18, 0, 2)
            xBtn.BackgroundColor3 = isExcl and Color3.fromRGB(180, 40, 40) or Color3.fromRGB(40, 40, 40)
            xBtn.BackgroundTransparency = 0
            xBtn.BorderSizePixel = 0
            xBtn.Text = "X"
            xBtn.TextColor3 = isExcl and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(120, 120, 120)
            xBtn.TextSize = 10
            xBtn.Font = Enum.Font.GothamBold
            xBtn.ZIndex = 10
            xBtn.Parent = bf
            local xBtnC = Instance.new("UICorner") ; xBtnC.CornerRadius = UDim.new(0, 3) ; xBtnC.Parent = xBtn
            xBtn.MouseButton1Click:Connect(function()
                if Settings.excludedBlocks[blockName] then
                    Settings.excludedBlocks[blockName] = nil
                else
                    Settings.excludedBlocks[blockName] = true
                end

                local nowExcl = Settings.excludedBlocks[blockName] or false
                xBtn.BackgroundColor3 = nowExcl and Color3.fromRGB(180, 40, 40) or Color3.fromRGB(40, 40, 40)
                xBtn.TextColor3 = nowExcl and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(120, 120, 120)
                bfs.Color = nowExcl and Color3.fromRGB(120, 40, 40) or (enough and Color3.fromRGB(40,90,40) or Color3.fromRGB(90,40,40))

                pcall(refreshExclusionList)
            end)
        end
    end
    updateBlocksDisplayGlobal = updateBlocksDisplay
    end

    task.wait()
    local _origUpdateBlocks = updateBlocksDisplay
    updateBlocksDisplay = function()
        _origUpdateBlocks()
        refreshExclusionList()
    end
    updateBlocksDisplayGlobal = updateBlocksDisplay
    makeLabel("REQUIRED BLOCKS", T2frame)
    makeBtn("RefreshBlocksBtn", "Refresh", T2frame, function() updateBlocksDisplay() end)

    createBuildContent()



    local function createConverterContent()

    makeLabel("CONVERTER", objFr)

    local selectedConvFile = nil
    local selectedConvKind = nil
    local selectedObjMode = "solid"
    local convFileBtn, refreshConvFiles

    local function openConverterHelp()
        local overlay = Instance.new("Frame")
        overlay.Size = UDim2.new(1, 0, 1, 0)
        overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        overlay.BackgroundTransparency = 0.35
        overlay.ZIndex = 300
        overlay.Parent = ScreenGui

        local card = Instance.new("Frame")
        card.Size = UDim2.new(0, 520, 0, 320)
        card.Position = UDim2.new(0.5, -260, 0.5, -160)
        card.BackgroundColor3 = Colors.Panel
        card.BorderSizePixel = 0
        card.ZIndex = 301
        card.Parent = overlay
        stylizeCard(card, Colors.Panel, Colors.Border, 6)

        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -12, 0, 22)
        title.Position = UDim2.new(0, 6, 0, 6)
        title.BackgroundTransparency = 1
        title.Text = "Converter Tutorial"
        title.TextColor3 = Colors.Text
        title.TextSize = 14
        title.Font = Enum.Font.GothamBold
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.ZIndex = 302
        title.Parent = card

        local body = Instance.new("TextLabel")
        body.Size = UDim2.new(1, -12, 1, -44)
        body.Position = UDim2.new(0, 6, 0, 32)
        body.BackgroundTransparency = 1
        body.TextColor3 = Colors.Muted
        body.TextSize = 11
        body.Font = Enum.Font.Gotham
        body.TextXAlignment = Enum.TextXAlignment.Left
        body.TextYAlignment = Enum.TextYAlignment.Top
        body.TextWrapped = true
        body.ZIndex = 302
        body.Text =
            "Converter supports image JSON, OBJ meshes, and Minecraft .schem/.schematic.\n\n" ..
            "You can select files from SoPeRa_Builds or paste a full file path.\n\n" ..
            "Image JSON:\n" ..
            "Go to https://www.samcodes.co.uk/project/geometrize-haxe-web/\n" ..
            "Upload an image, open settings and set:\n" ..
            "- Disable every shape type except Rotated Rectangles\n" ..
            "- Shape Opacity: 255\n" ..
            "- Initial Background Opacity: 255\n" ..
            "- Random Shapes Per Step: 100\n" ..
            "- Shape Mutations Per Step: 100\n" ..
            "Wait for load, save as JSON, choose IMAGE here, then convert.\n\n" ..
            "OBJ makes thin panels from mesh faces. SCHEM/SCHEMATIC greedily merges voxels by color."
        body.Parent = card

        overlay.InputBegan:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                overlay:Destroy()
            end
        end)
    end

    local convTop = Instance.new("Frame")
    convTop.Name = "ConvTop"
    convTop.Size = UDim2.new(1, 0, 0, 34)
    convTop.BackgroundTransparency = 1
    convTop.Parent = objFr
    local convTopLayout = Instance.new("UIListLayout")
    convTopLayout.FillDirection = Enum.FillDirection.Horizontal
    convTopLayout.Padding = UDim.new(0, 6)
    convTopLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    convTopLayout.Parent = convTop

    local convFileWrap = Instance.new("Frame")
    convFileWrap.Name = "ConvFileWrap"
    convFileWrap.Size = UDim2.new(1, -128, 1, 0)
    convFileWrap.BackgroundTransparency = 1
    convFileWrap.Parent = convTop

    local convRefreshBtn = Instance.new("TextButton")
    convRefreshBtn.Name = "ConvRefresh"
    convRefreshBtn.Size = UDim2.new(0, 76, 1, 0)
    convRefreshBtn.BackgroundColor3 = Colors.PanelElevated
    convRefreshBtn.BackgroundTransparency = 0
    convRefreshBtn.BorderSizePixel = 0
    convRefreshBtn.Text = "Refresh"
    convRefreshBtn.TextColor3 = Colors.Text
    convRefreshBtn.TextSize = 11
    convRefreshBtn.Font = Enum.Font.GothamSemibold
    convRefreshBtn.Parent = convTop
    stylizeCard(convRefreshBtn, Colors.PanelElevated, Colors.Border, 3)

    local helpBtn = Instance.new("TextButton")
    helpBtn.Name = "ConvHelp"
    helpBtn.Size = UDim2.new(0, 40, 1, 0)
    helpBtn.BackgroundColor3 = Colors.PanelElevated
    helpBtn.BackgroundTransparency = 0
    helpBtn.BorderSizePixel = 0
    helpBtn.Text = "?"
    helpBtn.TextColor3 = Colors.Text
    helpBtn.TextSize = 14
    helpBtn.Font = Enum.Font.GothamBold
    helpBtn.Parent = convTop
    stylizeCard(helpBtn, Colors.PanelElevated, Colors.Border, 12)
    helpBtn.MouseButton1Click:Connect(openConverterHelp)

    makeLabel("SOURCE FILE", objFr)
    local convPathIn = makeInput("ConvPath", "Choose a source file", objFr)
    local sourceMode = "file"
    local sourceModeRow = Instance.new("Frame")
    sourceModeRow.Name = "ConverterSourceMode"
    sourceModeRow.Size = UDim2.new(1, 0, 0, 32)
    sourceModeRow.BackgroundTransparency = 1
    sourceModeRow.Parent = objFr
    local sourceModeLayout = Instance.new("UIListLayout")
    sourceModeLayout.FillDirection = Enum.FillDirection.Horizontal
    sourceModeLayout.Padding = UDim.new(0, 6)
    sourceModeLayout.Parent = sourceModeRow
    local function makeSourceModeButton(name, text)
        local b = Instance.new("TextButton")
        b.Name = name
        b.Size = UDim2.new(0.5, -3, 1, 0)
        b.BackgroundColor3 = Colors.PanelElevated
        b.BorderSizePixel = 0
        b.Text = text
        b.TextColor3 = Colors.Text
        b.TextSize = 12
        b.Font = Enum.Font.GothamBold
        b.AutoButtonColor = false
        b.Parent = sourceModeRow
        stylizeCard(b, Colors.PanelElevated, Colors.Border, 4)
        return b
    end
    local sourceFileBtn = makeSourceModeButton("SourceFileMode", "FILE")
    local sourceAssetBtn = makeSourceModeButton("SourceAssetMode", "ROBLOX ID")

    local _convFilesCache = nil
    local _convFilesCacheTime = 0
    local _convFilesLoading = false
    local _convFilesLoaded = false
    local _convRefreshCb = nil
    local function getConverterFileOptions()
        if _convFilesCache and _convFilesLoaded and (tick() - _convFilesCacheTime) < 300 then
            return _convFilesCache
        end
        if _convFilesLoading then
            return _convFilesCache or {{display = "Loading files..."}}
        end
        _convFilesLoading = true
        task.spawn(function()
            ensureFolder()
            local files = {}
            local seenFiles = {}
            local function scanDir(dir, depth)
                if depth > 3 then return end
                local okF, items = pcall(listfiles, dir)
                if okF and type(items) == "table" then
                    for _, fp in ipairs(items) do
                        local fps = tostring(fp)
                        if isfolder(fps) then
                            scanDir(fps, depth + 1)
                        else
                            local low = fps:lower()
                            if low:match("%.json$") or low:match("%.obj$") or low:match("%.schem$") or low:match("%.schematic$") then
                                if not seenFiles[fps] then
                                    seenFiles[fps] = true
                                    local name = fps:match("([^/\\]+)$") or fps
                                    local parent = getParentDir(fp) or dir
                                    local display = (parent == "." or parent == FOLDER_PATH) and name or (name .. "  [" .. parent:gsub(FOLDER_PATH .. "/", "") .. "]")
                                    files[#files + 1] = {name = fp, display = display}
                                end
                            end
                        end
                    end
                    task.wait()
                end
            end
            scanDir(FOLDER_PATH, 0)
            table.sort(files, function(a, b) return tostring(a.display):lower() < tostring(b.display):lower() end)
            if #files == 0 then
            files = {{name = "", display = "No converter files found"}}
            end
            _convFilesCache = files
            _convFilesCacheTime = tick()
            _convFilesLoaded = true
            _convFilesLoading = false
            if _convRefreshCb then _convRefreshCb() end
        end)
        return {{display = "Loading files..."}}
    end
    local function setConvRefreshCb(cb) _convRefreshCb = cb end
    task.spawn(function()
        task.wait(0.3)
        getConverterFileOptions()
    end)

    local convSettings = Instance.new("Frame")
    convSettings.Name = "ConverterSettings"
    convSettings.Size = UDim2.new(1, 0, 0, 0)
    convSettings.BackgroundTransparency = 1
    convSettings.Visible = false
    convSettings.Parent = objFr
    local convSettingsLayout = Instance.new("UIListLayout")
    convSettingsLayout.Padding = UDim.new(0, 6)
    convSettingsLayout.Parent = convSettings

    local jsonSettings = Instance.new("Frame")
    jsonSettings.Name = "JsonSettings"
    jsonSettings.Size = UDim2.new(1, 0, 0, 0)
    jsonSettings.BackgroundTransparency = 1
    jsonSettings.Visible = false
    jsonSettings.Parent = convSettings
    local jsonSettingsLayout = Instance.new("UIListLayout")
    jsonSettingsLayout.Padding = UDim.new(0, 6)
    jsonSettingsLayout.Parent = jsonSettings

    makeLabel("IMAGE JSON (ROTATED RECTANGLES) — Geometrize output", jsonSettings)
    local outNameIn = makeInput("ConvOutName", "Output build name", jsonSettings)
    outNameIn.Text = "image_build"
    local scaleIn = makeInput("ConvScale", "Scale", jsonSettings)
    scaleIn.Text = "0.035"
    local widthIn = makeInput("ConvWidth", "Width studs (0 = scale)", jsonSettings)
    widthIn.Text = "0"
    local lengthIn = makeInput("ConvLength", "Length studs (0 = scale)", jsonSettings)
    lengthIn.Text = "0"
    local thickIn = makeInput("ConvThick", "Thickness", jsonSettings)
    thickIn.Text = "0.001"
    makeLabel("MATERIAL", jsonSettings)
    local matBtn, _ = makeDropdown("ConvMat", function()
        return {"PlasticBlock", "WoodBlock", "MetalBlock", "TitaniumBlock", "GlassBlock", "NeonBlock", "FabricBlock", "GraniteBlock", "MarbleBlock", "SlateBlock", "BrickBlock", "CobblestoneBlock", "DiamondPlateBlock", "FoilBlock", "GrassBlock", "IceBlock", "SandBlock", "ConcreteBlock", "CorrodedMetalBlock", "PebbleBlock"}
    end, jsonSettings, function(_) end)
    matBtn.Text = "PlasticBlock"
    matBtn.TextColor3 = Colors.Text

    local objSettings = Instance.new("Frame")
    objSettings.Name = "ObjSettings"
    objSettings.Size = UDim2.new(1, 0, 0, 0)
    objSettings.BackgroundTransparency = 1
    objSettings.Visible = false
    objSettings.Parent = convSettings
    local objSettingsLayout = Instance.new("UIListLayout")
    objSettingsLayout.Padding = UDim.new(0, 6)
    objSettingsLayout.Parent = objSettings
    makeLabel("OBJ SURFACE", objSettings)
    local objOutNameIn = makeInput("ObjOutName", "Output build name", objSettings)
    objOutNameIn.Text = "mesh_build"
    task.wait()
    local objScaleGet, objScaleSet = makeNumInput("Scale:", 1, 0.01, 100, 0.1, objSettings)
    local objThickGet, objThickSet = makeNumInput("Thickness:", 0.2, 0.01, 10, 0.05, objSettings)
    makeLabel("MODE", objSettings)
    local objModeBtn, _ = makeDropdown("ObjMode", function()
        return {
            {name = "face", display = "Face"},
            {name = "solid", display = "Solid"},
            {name = "wireframe", display = "Wireframe"},
            {name = "voxel", display = "Voxel"},
        }
    end, objSettings, function(mode)
        selectedObjMode = mode
    end)
    objModeBtn.Text = "Solid"
    objModeBtn.TextColor3 = Colors.Text
    makeLabel("MATERIAL", objSettings)
    local objMatBtn, _ = makeDropdown("ObjConvMat", function()
        local opts = {}
        pcall(function()
            for _, bp in ipairs(BuildingParts:GetChildren()) do
                if bp.Name:sub(-5) == "Block" then
                    opts[#opts + 1] = bp.Name
                end
            end
        end)
        if #opts == 0 then
            opts = {"PlasticBlock", "WoodBlock", "MetalBlock", "TitaniumBlock", "GlassBlock", "NeonBlock"}
        end
        table.sort(opts, function(a, b) return a:lower() < b:lower() end)
        return opts
    end, objSettings, function(_) end)
    objMatBtn.Text = "PlasticBlock"
    objMatBtn.TextColor3 = Colors.Text

    local objSolidDetail = 1
    makeNumInput("Detail:", 1, 0.2, 3, 0.1, objSettings, function(v) objSolidDetail = v end)

    local selectedObjColorMode = "custom"
    local objCustomR, objCustomG, objCustomB = 0.4, 0.6, 1.0
    local objGrad1R, objGrad1G, objGrad1B = 1.0, 0.3, 0.1
    local objGrad2R, objGrad2G, objGrad2B = 0.1, 0.3, 1.0
    local selectedGradDir = "y_asc"
    local colorInnerLayout = nil

    do
    local updateColorSectionContent = nil
    local updateGradPreview = nil
    local updateCustomPreview = nil
    local toggleColorSection = nil
    local function makeChannelSlider(parent, labelText, initialVal, barColor, onChange)
        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, 18)
        row.BackgroundTransparency = 1
        row.Parent = parent

        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0, 14, 1, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = labelText
        lbl.TextColor3 = Color3.fromRGB(180, 180, 180)
        lbl.TextSize = 11
        lbl.Font = Enum.Font.GothamBold
        lbl.Parent = row

        local track = Instance.new("Frame")
        track.Name = labelText .. "Track"
        track.Size = UDim2.new(1, -42, 0, 10)
        track.Position = UDim2.new(0, 18, 0.5, -5)
        track.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        track.BorderSizePixel = 0
        track.Parent = row
        local trCr = Instance.new("UICorner"); trCr.CornerRadius = UDim.new(0, 5); trCr.Parent = track

        local fill = Instance.new("Frame")
        fill.Name = "Fill"
        fill.Size = UDim2.new(initialVal / 255, 0, 1, 0)
        fill.BackgroundColor3 = barColor
        fill.BorderSizePixel = 0
        fill.Parent = track
        local flCr = Instance.new("UICorner"); flCr.CornerRadius = UDim.new(0, 5); flCr.Parent = fill

        local thumb = Instance.new("Frame")
        thumb.Name = "Thumb"
        thumb.Size = UDim2.new(0, 14, 0, 14)
        thumb.Position = UDim2.new(initialVal / 255, -7, 0.5, -7)
        thumb.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
        thumb.BorderSizePixel = 0
        thumb.Parent = track
        local thCr = Instance.new("UICorner"); thCr.CornerRadius = UDim.new(0, 7); thCr.Parent = thumb

        local valLbl = Instance.new("TextLabel")
        valLbl.Size = UDim2.new(0, 24, 1, 0)
        valLbl.Position = UDim2.new(1, -24, 0, 0)
        valLbl.BackgroundTransparency = 1
        valLbl.Text = tostring(initialVal)
        valLbl.TextColor3 = Color3.fromRGB(200, 200, 200)
        valLbl.TextSize = 11
        valLbl.Font = Enum.Font.GothamBold
        valLbl.TextXAlignment = Enum.TextXAlignment.Right
        valLbl.Parent = row

        local dragging = false
        local function updateFromX(xPos)
            local relX = math.clamp((xPos - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
            local val = math.floor(relX * 255 + 0.5)
            val = math.clamp(val, 0, 255)
            fill.Size = UDim2.new(relX, 0, 1, 0)
            thumb.Position = UDim2.new(relX, -7, 0.5, -7)
            valLbl.Text = tostring(val)
            return val
        end

        thumb.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
            end
        end)
        track.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                local val = updateFromX(input.Position.X)
                onChange(val)
            end
        end)

        local conn
        conn = UserInputService.InputChanged:Connect(function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local val = updateFromX(input.Position.X)
                onChange(val)
            end
        end)

        local conn2
        conn2 = UserInputService.InputEnded:Connect(function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
                dragging = false
            end
        end)

        return row
    end

    local colorSectionCont = Instance.new("Frame")
    colorSectionCont.Name = "ObjColorSection"
    colorSectionCont.Size = UDim2.new(1, 0, 0, 24)
    colorSectionCont.BackgroundTransparency = 1
    colorSectionCont.ClipsDescendants = true
    colorSectionCont.Parent = objSettings

    local colorMainRow = Instance.new("Frame")
    colorMainRow.Size = UDim2.new(1, 0, 0, 24)
    colorMainRow.BackgroundTransparency = 1
    colorMainRow.Name = "MainRow"
    colorMainRow.Parent = colorSectionCont
    local cmrl = Instance.new("UIListLayout")
    cmrl.FillDirection = Enum.FillDirection.Horizontal
    cmrl.Padding = UDim.new(0, 4)
    cmrl.VerticalAlignment = Enum.VerticalAlignment.Center
    cmrl.Parent = colorMainRow

    local colorPreviewSwatch = Instance.new("Frame")
    colorPreviewSwatch.Size = UDim2.new(0, 22, 0, 22)
    colorPreviewSwatch.BackgroundColor3 = Color3.new(objCustomR, objCustomG, objCustomB)
    colorPreviewSwatch.BorderSizePixel = 0
    colorPreviewSwatch.Parent = colorMainRow
    Instance.new("UICorner", colorPreviewSwatch).CornerRadius = UDim.new(0, 4)
    local cps = Instance.new("UIStroke"); cps.Color = Color3.fromRGB(80, 80, 80); cps.Thickness = 1; cps.Parent = colorPreviewSwatch

    local colorSectionLabel = Instance.new("TextLabel")
    colorSectionLabel.Size = UDim2.new(1, -52, 0, 22)
    colorSectionLabel.BackgroundTransparency = 1
    colorSectionLabel.Text = "COLOR"
    colorSectionLabel.TextColor3 = Colors.Text
    colorSectionLabel.TextSize = 11
    colorSectionLabel.Font = Enum.Font.GothamSemibold
    colorSectionLabel.TextXAlignment = Enum.TextXAlignment.Left
    colorSectionLabel.Parent = colorMainRow

    local colorEditBtn = Instance.new("TextButton")
    colorEditBtn.Name = "ColorEditBtn"
    colorEditBtn.Size = UDim2.new(0, 22, 0, 22)
    colorEditBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    colorEditBtn.BorderSizePixel = 0
    colorEditBtn.Text = "v"
    colorEditBtn.TextColor3 = Colors.Muted
    colorEditBtn.TextSize = 10
    colorEditBtn.Font = Enum.Font.GothamBold
    colorEditBtn.AutoButtonColor = false
    colorEditBtn.Parent = colorMainRow
    Instance.new("UICorner", colorEditBtn).CornerRadius = UDim.new(0, 4)

    local colorInnerPanel = Instance.new("Frame")
    colorInnerPanel.Name = "InnerPanel"
    colorInnerPanel.Size = UDim2.new(1, 0, 0, 0)
    colorInnerPanel.AutomaticSize = Enum.AutomaticSize.Y
    colorInnerPanel.Position = UDim2.new(0, 0, 0, 24)
    colorInnerPanel.BackgroundTransparency = 1
    colorInnerPanel.Visible = false
    colorInnerPanel.ClipsDescendants = true
    colorInnerPanel.Parent = colorSectionCont
    colorInnerLayout = Instance.new("UIListLayout")
    colorInnerLayout.Padding = UDim.new(0, 4)
    colorInnerLayout.Parent = colorInnerPanel

    makeLabel("MODE", colorInnerPanel)
    local objColorBtn, _ = makeDropdown("ObjColorMode", function()
        return {
            {name = "random", display = "Random"},
            {name = "custom", display = "Custom"},
            {name = "gradient", display = "Gradient"},
        }
    end, colorInnerPanel, function(mode)
        selectedObjColorMode = mode
        updateColorSectionContent()
        if mode == "custom" or mode == "gradient" then
            local h = 24 + math.max(34, colorInnerLayout.AbsoluteContentSize.Y + 4)
            TweenService:Create(colorSectionCont, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, h)}):Play()
        end
        resizeConv()
    end)
    objColorBtn.Text = "Custom"
    objColorBtn.TextColor3 = Colors.Text

    task.wait()
    local customContent = Instance.new("Frame")
    customContent.Name = "CustomContent"
    customContent.Size = UDim2.new(1, 0, 0, 0)
    customContent.AutomaticSize = Enum.AutomaticSize.Y
    customContent.BackgroundTransparency = 1
    customContent.Visible = false
    customContent.Parent = colorInnerPanel
    local customContentLayout = Instance.new("UIListLayout")
    customContentLayout.Padding = UDim.new(0, 4)
    customContentLayout.Parent = customContent

    local ccPreview = Instance.new("Frame")
    ccPreview.Size = UDim2.new(1, 0, 0, 18)
    ccPreview.BackgroundColor3 = Color3.new(objCustomR, objCustomG, objCustomB)
    ccPreview.BorderSizePixel = 0
    ccPreview.Parent = customContent
    Instance.new("UICorner", ccPreview).CornerRadius = UDim.new(0, 4)
    local ccPrevStroke = Instance.new("UIStroke"); ccPrevStroke.Color = Colors.Border; ccPrevStroke.Thickness = 1; ccPrevStroke.Parent = ccPreview

    updateCustomPreview = function()
        ccPreview.BackgroundColor3 = Color3.new(objCustomR, objCustomG, objCustomB)
        colorPreviewSwatch.BackgroundColor3 = Color3.new(objCustomR, objCustomG, objCustomB)
    end
    makeChannelSlider(customContent, "R", math.floor(objCustomR * 255), Color3.fromRGB(220, 60, 60), function(v) objCustomR = v / 255; updateCustomPreview() end)
    makeChannelSlider(customContent, "G", math.floor(objCustomG * 255), Color3.fromRGB(60, 200, 60), function(v) objCustomG = v / 255; updateCustomPreview() end)
    makeChannelSlider(customContent, "B", math.floor(objCustomB * 255), Color3.fromRGB(60, 100, 220), function(v) objCustomB = v / 255; updateCustomPreview() end)

    local gradientContent = Instance.new("Frame")
    gradientContent.Name = "GradientContent"
    gradientContent.Size = UDim2.new(1, 0, 0, 0)
    gradientContent.AutomaticSize = Enum.AutomaticSize.Y
    gradientContent.BackgroundTransparency = 1
    gradientContent.Visible = false
    gradientContent.Parent = colorInnerPanel
    local gradientContentLayout = Instance.new("UIListLayout")
    gradientContentLayout.Padding = UDim.new(0, 4)
    gradientContentLayout.Parent = gradientContent

    local g1Label = Instance.new("TextLabel")
    g1Label.Size = UDim2.new(1, 0, 0, 16)
    g1Label.BackgroundTransparency = 1
    g1Label.Text = "COLOR 1"
    g1Label.TextColor3 = Colors.Muted
    g1Label.TextSize = 10
    g1Label.Font = Enum.Font.GothamBold
    g1Label.TextXAlignment = Enum.TextXAlignment.Left
    g1Label.Parent = gradientContent

    local grad1Preview = Instance.new("Frame")
    grad1Preview.Size = UDim2.new(1, 0, 0, 18)
    grad1Preview.BackgroundColor3 = Color3.new(objGrad1R, objGrad1G, objGrad1B)
    grad1Preview.BorderSizePixel = 0
    grad1Preview.Parent = gradientContent
    Instance.new("UICorner", grad1Preview).CornerRadius = UDim.new(0, 4)
    local g1s = Instance.new("UIStroke"); g1s.Color = Colors.Border; g1s.Thickness = 1; g1s.Parent = grad1Preview

    makeChannelSlider(gradientContent, "R", math.floor(objGrad1R * 255), Color3.fromRGB(220, 60, 60), function(v) objGrad1R = v / 255; updateGradPreview() end)
    makeChannelSlider(gradientContent, "G", math.floor(objGrad1G * 255), Color3.fromRGB(60, 200, 60), function(v) objGrad1G = v / 255; updateGradPreview() end)
    makeChannelSlider(gradientContent, "B", math.floor(objGrad1B * 255), Color3.fromRGB(60, 100, 220), function(v) objGrad1B = v / 255; updateGradPreview() end)

    local g2Label = Instance.new("TextLabel")
    g2Label.Size = UDim2.new(1, 0, 0, 16)
    g2Label.BackgroundTransparency = 1
    g2Label.Text = "COLOR 2"
    g2Label.TextColor3 = Colors.Muted
    g2Label.TextSize = 10
    g2Label.Font = Enum.Font.GothamBold
    g2Label.TextXAlignment = Enum.TextXAlignment.Left
    g2Label.Parent = gradientContent

    local grad2Preview = Instance.new("Frame")
    grad2Preview.Size = UDim2.new(1, 0, 0, 18)
    grad2Preview.BackgroundColor3 = Color3.new(objGrad2R, objGrad2G, objGrad2B)
    grad2Preview.BorderSizePixel = 0
    grad2Preview.Parent = gradientContent
    Instance.new("UICorner", grad2Preview).CornerRadius = UDim.new(0, 4)
    local g2s = Instance.new("UIStroke"); g2s.Color = Colors.Border; g2s.Thickness = 1; g2s.Parent = grad2Preview

    makeChannelSlider(gradientContent, "R", math.floor(objGrad2R * 255), Color3.fromRGB(220, 60, 60), function(v) objGrad2R = v / 255; updateGradPreview() end)
    makeChannelSlider(gradientContent, "G", math.floor(objGrad2G * 255), Color3.fromRGB(60, 200, 60), function(v) objGrad2G = v / 255; updateGradPreview() end)
    makeChannelSlider(gradientContent, "B", math.floor(objGrad2B * 255), Color3.fromRGB(60, 100, 220), function(v) objGrad2B = v / 255; updateGradPreview() end)

    local gradPreviewBar = Instance.new("Frame")
    gradPreviewBar.Size = UDim2.new(1, 0, 0, 20)
    gradPreviewBar.BackgroundColor3 = Color3.new(1, 1, 1)
    gradPreviewBar.BorderSizePixel = 0
    gradPreviewBar.Parent = gradientContent
    Instance.new("UICorner", gradPreviewBar).CornerRadius = UDim.new(0, 4)
    local gpbs = Instance.new("UIStroke"); gpbs.Color = Colors.Border; gpbs.Thickness = 1; gpbs.Parent = gradPreviewBar
    local gradUI = Instance.new("UIGradient")
    gradUI.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.new(objGrad1R, objGrad1G, objGrad1B)),
        ColorSequenceKeypoint.new(1, Color3.new(objGrad2R, objGrad2G, objGrad2B)),
    })
    gradUI.Parent = gradPreviewBar

    updateGradPreview = function()
        grad1Preview.BackgroundColor3 = Color3.new(objGrad1R, objGrad1G, objGrad1B)
        grad2Preview.BackgroundColor3 = Color3.new(objGrad2R, objGrad2G, objGrad2B)
        colorPreviewSwatch.BackgroundColor3 = Color3.new(objGrad1R, objGrad1G, objGrad1B)
        gradUI.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.new(objGrad1R, objGrad1G, objGrad1B)),
            ColorSequenceKeypoint.new(1, Color3.new(objGrad2R, objGrad2G, objGrad2B)),
        })
    end
    updateGradPreview()

    local gDirLabel = Instance.new("TextLabel")
    gDirLabel.Size = UDim2.new(1, 0, 0, 16)
    gDirLabel.BackgroundTransparency = 1
    gDirLabel.Text = "DIRECTION"
    gDirLabel.TextColor3 = Colors.Muted
    gDirLabel.TextSize = 10
    gDirLabel.Font = Enum.Font.GothamBold
    gDirLabel.TextXAlignment = Enum.TextXAlignment.Left
    gDirLabel.Parent = gradientContent

    local gradDirBtn, _ = makeDropdown("GradDirDD", function()
        return {
            {name = "y_asc", display = "Bottom > Top"},
            {name = "y_desc", display = "Top > Bottom"},
            {name = "x_asc", display = "Left > Right"},
            {name = "x_desc", display = "Right > Left"},
            {name = "z_asc", display = "Front > Back"},
            {name = "z_desc", display = "Back > Front"},
            {name = "radial_out", display = "Inside > Out"},
            {name = "radial_in", display = "Outside > In"},
            {name = "index", display = "Face Index"},
        }
    end, gradientContent, function(dir)
        selectedGradDir = dir
    end)
    gradDirBtn.Text = "Bottom > Top"
    gradDirBtn.TextColor3 = Colors.Text

    local colorSectionOpen = false

    updateColorSectionContent = function()
        if selectedObjColorMode == "custom" then
            customContent.Visible = true
            gradientContent.Visible = false
            colorPreviewSwatch.BackgroundColor3 = Color3.new(objCustomR, objCustomG, objCustomB)
        elseif selectedObjColorMode == "gradient" then
            customContent.Visible = false
            gradientContent.Visible = true
            colorPreviewSwatch.BackgroundColor3 = Color3.new(objGrad1R, objGrad1G, objGrad1B)
        else
            customContent.Visible = false
            gradientContent.Visible = false
            colorPreviewSwatch.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        end
    end

    toggleColorSection = function(forceState)
        if forceState == true then
            colorSectionOpen = true
        elseif forceState == false then
            colorSectionOpen = false
        else
            colorSectionOpen = not colorSectionOpen
        end
        colorInnerPanel.Visible = colorSectionOpen
        if colorSectionOpen then
            RunService.Heartbeat:Wait()
            local innerH = colorInnerLayout.AbsoluteContentSize.Y
            local h = 24 + math.max(34, innerH + 4)
            TweenService:Create(colorSectionCont, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, h)}):Play()
            colorEditBtn.Text = "^"
        else
            TweenService:Create(colorSectionCont, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(1, 0, 0, 24)}):Play()
            colorEditBtn.Text = "v"
        end
        resizeConv()
    end

    colorEditBtn.MouseButton1Click:Connect(function()
        toggleColorSection()
    end)
    end

    local convPreviewButtons = {}
    local function updateConverterPreviewButtons()
        for _, entry in ipairs(convPreviewButtons) do
            if entry.button and entry.button.Parent then
                entry.button.Text = previewActive and "Clear Preview" or entry.label
            end
        end
    end

    local function applyConvertedPreview(materialName, blocks, label)
        local cleanBlocks = {}
        local seen = {}
        for _, block in ipairs(blocks or {}) do
            if type(block) == "table" then
                local key = table.concat({
                    tostring(block.Position or ""),
                    tostring(block.Rotation or ""),
                    tostring(block.Size or ""),
                    tostring(block.Color or ""),
                    tostring(block.Transparency or 0),
                }, "|")
                if not seen[key] then
                    seen[key] = true
                    cleanBlocks[#cleanBlocks + 1] = block
                end
            end
        end
        local prs = convertedBlocksToPRS(materialName, cleanBlocks)
        if not prs or not next(prs) then
            setStatus("  Preview build is empty")
            return
        end
        clearPreview()
        createPreview(prs)
        updateConverterPreviewButtons()
        setStatus("  Preview ready: " .. tostring(label or "source"))
    end
    local objPreviewBtn = makeBtn("ObjConvPreview", "Preview Mesh", objSettings, function()
        if previewActive then
            clearPreview()
            updateConverterPreviewButtons()
            setStatus("  Preview cleared")
            return
        end
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local low = fullPath:lower()
        local sc = objScaleGet() or 1
        local th = objThickGet() or 0.2
        local material = (objMatBtn.Text and objMatBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        local colorOpts = {
            colorMode = selectedObjColorMode,
            detail = objSolidDetail,
            customR = objCustomR, customG = objCustomG, customB = objCustomB,
            grad1R = objGrad1R, grad1G = objGrad1G, grad1B = objGrad1B,
            grad2R = objGrad2R, grad2G = objGrad2G, grad2B = objGrad2B,
            gradDir = selectedGradDir,
        }
        local ok, outMaterial, blocks, err = pcall(function()
            return convertObjToBlocks(fullPath, sc, th, selectedObjMode, material, colorOpts)
        end)
        if not ok then
            setStatus("  Preview error: " .. tostring(outMaterial))
            return
        end
        if err then
            setStatus("  Preview error: " .. tostring(err))
            return
        end
        applyConvertedPreview(outMaterial, blocks, getFileStem(fullPath))
    end)
    convPreviewButtons[#convPreviewButtons + 1] = {button = objPreviewBtn, label = "Preview Mesh"}
    task.wait()
    makeBtn("ObjConvRun", "Convert Mesh -> .Build", objSettings, function()
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local outName = trimStr(objOutNameIn.Text)
        if outName == "" then setStatus("  Enter output name") ; return end
        local low = fullPath:lower()
        local sc = objScaleGet() or 1
        local th = objThickGet() or 0.2
        local material = (objMatBtn.Text and objMatBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        local colorOpts = {
            colorMode = selectedObjColorMode,
            detail = objSolidDetail,
            customR = objCustomR, customG = objCustomG, customB = objCustomB,
            grad1R = objGrad1R, grad1G = objGrad1G, grad1B = objGrad1B,
            grad2R = objGrad2R, grad2G = objGrad2G, grad2B = objGrad2B,
            gradDir = selectedGradDir,
        }
        local ok, outPath, err = pcall(function()
            return convertObjToBuild(fullPath, outName, sc, th, material, selectedObjMode, colorOpts)
        end)
        if not ok then
            setStatus("  Convert error: " .. tostring(outPath))
            return
        end
        if err then
            setStatus("  Convert error: " .. tostring(err))
            return
        end
        setStatus("  Saved: " .. tostring(outPath))
        refreshFiles()
        refreshConvFiles()
    end)

    local schemSettings = Instance.new("Frame")
    schemSettings.Name = "SchemSettings"
    schemSettings.Size = UDim2.new(1, 0, 0, 0)
    schemSettings.BackgroundTransparency = 1
    schemSettings.Visible = false
    schemSettings.Parent = convSettings
    local schemSettingsLayout = Instance.new("UIListLayout")
    schemSettingsLayout.Padding = UDim.new(0, 6)
    schemSettingsLayout.Parent = schemSettings
    makeLabel("MINECRAFT SCHEMATIC", schemSettings)
    local schemOutNameIn = makeInput("SchemOutName", "Output build name", schemSettings)
    schemOutNameIn.Text = "schem_build"
    local schemScaleIn = makeInput("SchemScale", "Studs per block", schemSettings)
    schemScaleIn.Text = "1"
    makeLabel("MATERIAL", schemSettings)
    local schemMatBtn, _ = makeDropdown("SchemConvMat", function()
        return {"PlasticBlock", "WoodBlock", "MetalBlock", "TitaniumBlock", "GlassBlock", "NeonBlock", "FabricBlock", "GraniteBlock", "MarbleBlock", "SlateBlock", "BrickBlock", "CobblestoneBlock", "DiamondPlateBlock", "FoilBlock", "GrassBlock", "IceBlock", "SandBlock", "ConcreteBlock", "CorrodedMetalBlock", "PebbleBlock"}
    end, schemSettings, function(_) end)
    schemMatBtn.Text = "PlasticBlock"
    schemMatBtn.TextColor3 = Colors.Text
    local schemPreviewBtn = makeBtn("SchemConvPreview", "Preview Schematic", schemSettings, function()
        if previewActive then
            clearPreview()
            updateConverterPreviewButtons()
            setStatus("  Preview cleared")
            return
        end
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local sc = tonumber(schemScaleIn.Text) or 1
        local material = (schemMatBtn.Text and schemMatBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        local ok, outMaterial, blocks, err = pcall(function()
            return convertMinecraftSchematicToBlocks(fullPath, sc, material)
        end)
        if not ok then
            setStatus("  Preview error: " .. tostring(outMaterial))
            return
        end
        if err then
            setStatus("  Preview error: " .. tostring(err))
            return
        end
        applyConvertedPreview(outMaterial, blocks, getFileStem(fullPath))
    end)
    convPreviewButtons[#convPreviewButtons + 1] = {button = schemPreviewBtn, label = "Preview Schematic"}
    makeBtn("SchemConvRun", "Convert Schematic -> .Build", schemSettings, function()
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local outName = trimStr(schemOutNameIn.Text)
        local sc = tonumber(schemScaleIn.Text) or 1
        local material = (schemMatBtn.Text and schemMatBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        if outName == "" then setStatus("  Enter output name") ; return end
        local ok, outPath, err = pcall(function()
            return convertMinecraftSchematicToBuild(fullPath, outName, sc, material)
        end)
        if not ok then
            setStatus("  Convert error: " .. tostring(outPath))
            return
        end
        if err then
            setStatus("  Convert error: " .. tostring(err))
            return
        end
        setStatus("  Saved: " .. tostring(outPath))
        refreshFiles()
        refreshConvFiles()
    end)

    local assetSettings = Instance.new("Frame")
    assetSettings.Name = "AssetSettings"
    assetSettings.Size = UDim2.new(1, 0, 0, 0)
    assetSettings.BackgroundTransparency = 1
    assetSettings.Visible = false
    assetSettings.Parent = convSettings
    local assetSettingsLayout = Instance.new("UIListLayout")
    assetSettingsLayout.Padding = UDim.new(0, 6)
    assetSettingsLayout.Parent = assetSettings
    makeLabel("ROBLOX MODEL ASSET", assetSettings)
    local assetIdIn = makeInput("AssetId", "ROBLOX ASSET ID, e.g. 123456789", assetSettings)
    local assetOutNameIn = makeInput("AssetOutName", "Output build name", assetSettings)
    assetOutNameIn.Text = "roblox_model"
    local assetScaleIn = makeInput("AssetScale", "Scale (default 1)", assetSettings)
    assetScaleIn.Text = "1"
    local assetMatBtn, _ = makeDropdown("AssetConvMat", function()
        return {
            {name = "Auto", display = "Auto source material"},
            "PlasticBlock", "WoodBlock", "MetalBlock", "TitaniumBlock", "GlassBlock", "NeonBlock",
            "FabricBlock", "GraniteBlock", "MarbleBlock", "SlateBlock", "BrickBlock", "CobblestoneBlock",
            "DiamondPlateBlock", "FoilBlock", "GrassBlock", "IceBlock", "SandBlock", "ConcreteBlock",
            "CorrodedMetalBlock", "PebbleBlock"
        }
    end, assetSettings, function(_) end)
    assetMatBtn.Text = "Auto source material"
    assetMatBtn.TextColor3 = Colors.Text
    local function getAssetMaterial()
        if assetMatBtn.Text and assetMatBtn.Text:lower():find("auto") then return "Auto" end
        return (assetMatBtn.Text and assetMatBtn.Text:match("([^%s]+)")) or "PlasticBlock"
    end
    local assetPreviewBtn = makeBtn("AssetPreview", "Preview Roblox Model", assetSettings, function()
        local blocks, err = convertRobloxAssetToBlocks(assetIdIn.Text, tonumber(assetScaleIn.Text) or 1)
        if not blocks then setStatus("  Import error: " .. tostring(err)); playUISound(UISoundConfig.error); showFormatWarning(objFr); return end
        local material = getAssetMaterial()
        applyConvertedPreview(material, blocks, "Roblox asset")
    end)
    convPreviewButtons[#convPreviewButtons + 1] = {button = assetPreviewBtn, label = "Preview Roblox Model"}
    makeBtn("AssetConvRun", "Import Roblox Model -> .Build", assetSettings, function()
        local outName = trimStr(assetOutNameIn.Text)
        if outName == "" then setStatus("  Enter output name"); return end
        local blocks, err = convertRobloxAssetToBlocks(assetIdIn.Text, tonumber(assetScaleIn.Text) or 1)
        if not blocks then setStatus("  Import error: " .. tostring(err)); playUISound(UISoundConfig.error); showFormatWarning(objFr); return end
        local material = getAssetMaterial()
        local outPath, writeErr = writeConvertedBuild(outName, material, blocks)
        if not outPath then setStatus("  Save error: " .. tostring(writeErr)); playUISound(UISoundConfig.error); return end
        setStatus("  Saved: " .. tostring(outPath))
        playUISound(UISoundConfig.success)
        refreshFiles()
    end)

    local function inferConvKind(fileName)
        local s = tostring(fileName or ""):lower()
        if s:match("^%d+$") or s:match("rbxassetid://%d+") or s:match("roblox%.com/library/%d+") then return "asset" end
        if s:match("%.json$") then return "json" end
        if s:match("%.obj$") then return "obj" end
        if s:match("%.schem$") or s:match("%.schematic$") then return "schem" end
        return nil
    end

    local function resizeConv()
        if not convSettings.Visible then return end
        local targetH = 0
        if selectedConvKind == "json" then
            local h = math.max(34, math.floor(jsonSettingsLayout.AbsoluteContentSize.Y + 8))
            jsonSettings.Size = UDim2.new(1, 0, 0, h)
            objSettings.Size = UDim2.new(1, 0, 0, 0)
            schemSettings.Size = UDim2.new(1, 0, 0, 0)
            assetSettings.Size = UDim2.new(1, 0, 0, 0)
            targetH = h
        elseif selectedConvKind == "obj" then
            local h = math.max(34, math.floor(objSettingsLayout.AbsoluteContentSize.Y + 8))
            jsonSettings.Size = UDim2.new(1, 0, 0, 0)
            objSettings.Size = UDim2.new(1, 0, 0, h)
            schemSettings.Size = UDim2.new(1, 0, 0, 0)
            assetSettings.Size = UDim2.new(1, 0, 0, 0)
            targetH = h
        elseif selectedConvKind == "asset" then
            local h = math.max(34, math.floor(assetSettingsLayout.AbsoluteContentSize.Y + 8))
            jsonSettings.Size = UDim2.new(1, 0, 0, 0)
            objSettings.Size = UDim2.new(1, 0, 0, 0)
            schemSettings.Size = UDim2.new(1, 0, 0, 0)
            assetSettings.Size = UDim2.new(1, 0, 0, h)
            targetH = h
        elseif selectedConvKind == "schem" then
            local h = math.max(34, math.floor(schemSettingsLayout.AbsoluteContentSize.Y + 8))
            jsonSettings.Size = UDim2.new(1, 0, 0, 0)
            objSettings.Size = UDim2.new(1, 0, 0, 0)
            schemSettings.Size = UDim2.new(1, 0, 0, h)
            assetSettings.Size = UDim2.new(1, 0, 0, 0)
            targetH = h
        else
            jsonSettings.Size = UDim2.new(1, 0, 0, 0)
            objSettings.Size = UDim2.new(1, 0, 0, 0)
            schemSettings.Size = UDim2.new(1, 0, 0, 0)
            assetSettings.Size = UDim2.new(1, 0, 0, 0)
        end
        convSettings.Size = UDim2.new(1, 0, 0, targetH)
    end

    jsonSettingsLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(resizeConv)
    objSettingsLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(resizeConv)
    colorInnerLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(resizeConv)
    schemSettingsLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(resizeConv)
    assetSettingsLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(resizeConv)

    local function refreshSourceModeVisual()
        local isAsset = sourceMode == "asset"
        sourceFileBtn.BackgroundColor3 = isAsset and Colors.PanelElevated or Colors.ActiveBG
        sourceFileBtn.TextColor3 = isAsset and Colors.Text or Colors.ActiveText
        sourceAssetBtn.BackgroundColor3 = isAsset and Colors.ActiveBG or Colors.PanelElevated
        sourceAssetBtn.TextColor3 = isAsset and Colors.ActiveText or Colors.Text
        if convFileWrap then convFileWrap.Visible = not isAsset end
        if convRefreshBtn then convRefreshBtn.Visible = not isAsset end
        if convPathIn and convPathIn.Parent then convPathIn.Parent.Visible = not isAsset end
    end

    local function showConvKind(kind)
        selectedConvKind = kind
        convSettings.Visible = selectedConvKind ~= nil
        jsonSettings.Visible = selectedConvKind == "json"
        objSettings.Visible = selectedConvKind == "obj"
        schemSettings.Visible = selectedConvKind == "schem"
        assetSettings.Visible = selectedConvKind == "asset"
        resizeConv()
    end

    local function setConvFile(fileName)
        selectedConvFile = trimStr(fileName)
        selectedConvKind = inferConvKind(selectedConvFile)
        sourceMode = selectedConvKind == "asset" and "asset" or "file"
        if selectedConvKind == "asset" then
            local id = selectedConvFile:match("%d+") or ""
            assetIdIn.Text = id
            selectedConvFile = id
        end
        refreshSourceModeVisual()
        convPathIn.Text = selectedConvFile
        if convFileBtn then
            convFileBtn.Text = getFileStem(selectedConvFile) ~= "" and getFileStem(selectedConvFile) or selectedConvFile
            convFileBtn.TextColor3 = Colors.Text
        end

        showConvKind(selectedConvKind)

        local stem = getFileStem(selectedConvFile)
        if stem ~= "" then
            if selectedConvKind == "json" then
                outNameIn.Text = stem
            elseif selectedConvKind == "obj" then
                objOutNameIn.Text = stem
            elseif selectedConvKind == "schem" then
                schemOutNameIn.Text = stem
            elseif selectedConvKind == "asset" then
                assetOutNameIn.Text = "roblox_" .. stem
            end
        end
    end

    sourceFileBtn.MouseButton1Click:Connect(function()
        sourceMode = "file"
        refreshSourceModeVisual()
        local txt = trimStr(convPathIn.Text)
        if txt ~= "" then setConvFile(txt) else showConvKind(nil) end
    end)

    sourceAssetBtn.MouseButton1Click:Connect(function()
        sourceMode = "asset"
        selectedConvFile = trimStr(assetIdIn.Text)
        refreshSourceModeVisual()
        showConvKind("asset")
    end)

    assetIdIn.FocusLost:Connect(function()
        local id = tostring(assetIdIn.Text or ""):match("%d+") or ""
        assetIdIn.Text = id
        sourceMode = "asset"
        selectedConvFile = id
        if id ~= "" then assetOutNameIn.Text = "roblox_" .. id end
        refreshSourceModeVisual()
        showConvKind("asset")
    end)
    refreshSourceModeVisual()

    convFileBtn, refreshConvFiles = makeDropdown("ConvFile", getConverterFileOptions, convFileWrap, function(fileName)
        if not fileName or fileName == "" or fileName == "No converter files found" then return end
        setConvFile(fileName)
    end)
    setConvRefreshCb(function()
        if refreshConvFiles then refreshConvFiles() end
    end)
    convRefreshBtn.MouseButton1Click:Connect(function()
        refreshConvFiles()
        local opts = getConverterFileOptions()
        local count = (type(opts[1]) == "string") and 0 or #opts
        setStatus("  Converter file list refreshed: " .. tostring(count))
    end)
    convPathIn.FocusLost:Connect(function()
        _convFilesCache = nil
        local txt = trimStr(convPathIn.Text)
        if txt ~= "" then
            if isfolder(txt) then
                selectedConvFile = txt
                selectedConvKind = nil
                convSettings.Visible = false
                jsonSettings.Visible = false
                objSettings.Visible = false
                schemSettings.Visible = false
                resizeConv()
                setStatus("  Source folder set: " .. txt)
            else
                setConvFile(txt)
            end
        end
    end)

    local jsonPreviewBtn = makeBtn("ConvPreview", "Preview Image JSON", jsonSettings, function()
        if previewActive then
            clearPreview()
            updateConverterPreviewButtons()
            setStatus("  Preview cleared")
            return
        end
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local sc = tonumber(scaleIn.Text) or 0.035
        local targetW = tonumber(widthIn.Text) or 0
        local targetL = tonumber(lengthIn.Text) or 0
        local th = tonumber(thickIn.Text) or 0.001
        local material = (matBtn and matBtn.Text and matBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        local ok, outMaterial, blocks, err = pcall(function()
            local txt = readfile(fullPath)
            if not txt then error("Failed to read file: " .. tostring(fullPath)) end
            return convertGeometrizeJsonToBlocks(txt, sc, th, material, targetW, targetL)
        end)
        if not ok then
            setStatus("  Preview error: " .. tostring(outMaterial))
            return
        end
        if err then
            setStatus("  Preview error: " .. tostring(err))
            return
        end
        applyConvertedPreview(outMaterial, blocks, getFileStem(fullPath))
    end)
    convPreviewButtons[#convPreviewButtons + 1] = {button = jsonPreviewBtn, label = "Preview Image JSON"}
    task.wait()
    makeBtn("ConvRun", "Convert -> .Build", jsonSettings, function()
        if selectedConvKind ~= "json" then
            setStatus("  Select a .json file")
            return
        end
        local fullPath, pathErr = resolveConverterPath(convPathIn.Text ~= "" and convPathIn.Text or selectedConvFile)
        if not fullPath then
            setStatus("  " .. tostring(pathErr))
            return
        end
        local outName = trimStr(outNameIn.Text)
        local sc = tonumber(scaleIn.Text) or 0.035
        local targetW = tonumber(widthIn.Text) or 0
        local targetL = tonumber(lengthIn.Text) or 0
        local th = tonumber(thickIn.Text) or 0.001
        local material = (matBtn and matBtn.Text and matBtn.Text:match("([^%s]+)")) or "PlasticBlock"
        if outName == "" then setStatus("  Enter output name") ; return end
        if not fullPath:lower():match("%.json$") then
            setStatus("  JSON name must end with .json")
            return
        end
        local ok, outOrErr, err2 = pcall(function()
            local txt = readfile(fullPath)
            if not txt then error("Failed to read file: " .. tostring(fullPath)) end
            return convertGeometrizeJsonToBuild(txt, outName, sc, th, material, targetW, targetL)
        end)
        if not ok then
            setStatus("  Convert error: " .. tostring(outOrErr))
            return
        end
        local outPath = outOrErr
        if err2 then
            setStatus("  Convert error: " .. tostring(err2))
            return
        end
        setStatus("  Saved: " .. tostring(outPath))
        refreshFiles()
        refreshConvFiles()
    end)

    end
    createConverterContent()
    local function createInfBlockContent()
    makeLabel("INF BLOCK", infFr)
    local infToggle = makeBtn("InfBlockToggle", "Inf Block: " .. (Settings.infBlockEnabled and "ON" or "OFF"), infFr, function()
        Settings.infBlockEnabled = not Settings.infBlockEnabled
        local b = infFr:FindFirstChild("InfBlockToggle")
        if b then
            b.Text = "Inf Block: " .. (Settings.infBlockEnabled and "ON" or "OFF")
            b.BackgroundColor3 = Settings.infBlockEnabled and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        saveSettings()
    end)

    end
    createInfBlockContent()
    local function createNoclipFlyContent()
    local noclipActive = false
    local noclipConn = nil
    makeLabel("NOCLIP / FLY", movFr)
    makeBtn("NoclipBtn", "NoClip: OFF", movFr, function()
        noclipActive = not noclipActive
        local b = movFr:FindFirstChild("NoclipBtn")
        if b then
            b.Text = "NoClip: " .. (noclipActive and "ON" or "OFF")
            b.BackgroundColor3 = noclipActive and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if noclipActive then
            noclipConn = RunService.Stepped:Connect(function()
                if LocalPlayer.Character then
                    for _, p in pairs(LocalPlayer.Character:GetDescendants()) do
                        if p:IsA("BasePart") then p.CanCollide = false end
                    end
                end
            end)
        else
            if noclipConn then noclipConn:Disconnect() ; noclipConn = nil end
            if LocalPlayer.Character then
                for _, p in pairs(LocalPlayer.Character:GetDescendants()) do
                    if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" then p.CanCollide = true end
                end
            end
        end
    end)

    local flyActive = false
    local flySpeed = 50
    local flyConn = nil
    local flyBV = nil
    makeLabel("Fly Speed:", movFr)
    local flySlider, flySetVal = makeSlider("FlySpd", 0, 4242, flySpeed, movFr, "Fly Speed",
        function(v) return math.floor(v) end,
        function(v) flySpeed = math.floor(v) end
    )
    makeBtn("FlyBtn", "Fly: OFF", movFr, function()
        flyActive = not flyActive
        local b = movFr:FindFirstChild("FlyBtn")
        if b then
            b.Text = "Fly: " .. (flyActive and "ON" or "OFF")
            b.BackgroundColor3 = flyActive and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if flyActive then
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hrp then

                if hum then hum.PlatformStand = true end
                flyBV = flyBV or Instance.new("BodyVelocity")
                flyBV.Name = "FlyBV"
                flyBV.MaxForce = Vector3.new(9e9,9e9,9e9)
                flyBV.Velocity = Vector3.zero
                flyBV.Parent = hrp
                local flyGyro = Instance.new("BodyGyro")
                flyGyro.Name = "FlyGyro"
                flyGyro.MaxTorque = Vector3.new(9e9,9e9,9e9)
                flyGyro.P = 9e4
                flyGyro.D = 1000
                flyGyro.Parent = hrp
            end
            flyConn = RunService.Heartbeat:Connect(function()
                if not LocalPlayer.Character then return end
                local hrp2 = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not hrp2 then return end
                local cam = Workspace.CurrentCamera
                local camCF = cam.CFrame
                local mv = Vector3.zero

                if UserInputService:IsKeyDown(Enum.KeyCode.W) then mv = mv + camCF.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then mv = mv - camCF.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then mv = mv - camCF.RightVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then mv = mv + camCF.RightVector end

                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then mv = mv + Vector3.new(0,1,0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then mv = mv - Vector3.new(0,1,0) end

                if flyBV then
                    flyBV.Velocity = mv.Magnitude > 0.001 and (mv.Unit * flySpeed) or Vector3.zero
                end

                local gyro = hrp2:FindFirstChild("FlyGyro")
                if gyro then gyro.CFrame = camCF end
            end)
        else
            if flyConn then flyConn:Disconnect() ; flyConn = nil end
            if flyBV then flyBV:Destroy() ; flyBV = nil end
            local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local gyro = hrp:FindFirstChild("FlyGyro")
                if gyro then gyro:Destroy() end
            end
            local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
    end)
    end
    createNoclipFlyContent()

    farmWin = nil
    farmSettings = { step = 2, tgToken = "", tgChatID = "", tgInterval = 0, renderEnabled = true, autoHop = false, autoFarm = false, autoFarmFile = "", autoBuild = false, autoBuildFile = "" }
    saveFarmSettings = nil

    function _G.createBhopContent()
    do

    makeLabel("BHOP (CS:GO Movement)", movFr)
    local bhopEnabled = false
    local bhopConn = nil
    local BhopMovement = nil

    local BhopCfg = {
        STEP_OFFSET=1.2, MASS=16, AIR_FRICTION=0.4, FRICTION=6, GRAVITY=10,
        JUMP_VELOCITY=30, GROUND_ACCEL=14, GROUND_DECCEL=10, AIR_ACCEL=52,
        AIR_SPEED=42, RUN_SPEED=32, WALK_SPEED=10, CROUCH_SPEED=10,
        AIR_MAX_SPEED=36.5, AIR_MAX_SPEED_FRIC=3, AIR_MAX_SPEED_FRIC_DEC=.5,
        MIN_SLOPE_ANGLE=40, MAX_SLOPE_ANGLE=75, LEG_HEIGHT=2.2,
        TORSO_TO_FEET=5.0, FEET_HB_SIZE=Vector3.new(1,0.1,1),
        TORSO_HB_SIZE=Vector3.new(3,1,3), FOOT_OFFSET_AMOUNT=1.2
    }

    local function initBhop()
        workspace.Gravity = 100
        local M = {}
        M.Keys = {W=0,S=0,D=0,A=0,Space=0}
        M.player = LocalPlayer
        M.character = LocalPlayer.Character
        while not M.character do
            M.character = LocalPlayer.Character
            task.wait(0.05)
        end
        M.collider = safeWaitChild(M.character, "HumanoidRootPart", 10)
        if not M.collider then return nil end
        M.config = BhopCfg
        local mover = Instance.new("LinearVelocity", M.collider)
        local a0 = Instance.new("Attachment", M.collider)
        a0.Name = "MovementAttachment"
        mover.Attachment0 = a0
        mover.MaxForce = 10000000
        mover.VelocityConstraintMode = Enum.VelocityConstraintMode.Plane
        mover.PrimaryTangentAxis = Vector3.new(1,0,0)
        mover.SecondaryTangentAxis = Vector3.new(0,0,1)
        M.mover = mover
        M.states = {grounded=false, air_friction=0, input_vec=Vector3.zero, surfing=false, jumping=false}

        local function getParams()
            local p = RaycastParams.new()
            p.FilterDescendantsInstances = {M.character, Workspace.CurrentCamera}
            p.FilterType = Enum.RaycastFilterType.Exclude
            p.RespectCanCollide = false
            return p
        end
        local function getAngle(n) return math.deg(math.acos(n:Dot(Vector3.yAxis))) end
        local function isGrounded(dir)
            local cf = CFrame.new(M.collider.Position) - Vector3.new(0, M.collider.Size.Y/2, 0)
            dir = dir or Vector3.new(0, -1*M.collider.Size.Y - BhopCfg.FOOT_OFFSET_AMOUNT, 0)
            local res = Workspace:Blockcast(cf, BhopCfg.FEET_HB_SIZE, dir, getParams())
            if not res then return false end
            local steep = getAngle(res.Normal)
            if steep >= BhopCfg.MIN_SLOPE_ANGLE and steep <= BhopCfg.MAX_SLOPE_ANGLE then return false, true, res, steep end
            return true, false, res
        end
        local function rotChar()
            local cam = Workspace.CurrentCamera
            local rl = M.collider.Position + cam.CoordinateFrame.lookVector
            M.collider.CFrame = CFrame.new(M.collider.Position, Vector3.new(rl.x, M.collider.Position.y, rl.z))
            M.collider.RotVelocity = Vector3.new()
        end
        local function getMoveDir(gn)
            local fwd = M.Keys.W + -M.Keys.S
            local side = M.Keys.A + -M.Keys.D
            gn = gn or Vector3.new(0,1,0)
            if fwd == 0 and side == 0 then M.states.input_vec = Vector3.zero ; return Vector3.zero end
            M.states.input_vec = Vector3.new(-side,0,-fwd).Unit
            local fm = gn:Cross(M.collider.CFrame.RightVector)
            local sm = gn:Cross(fm)
            return (fm*fwd + sm*side).Unit
        end
        local function applyFriction(mod, inAir)
            local vel = inAir and M.collider.Velocity or Vector3.new(M.mover.PlaneVelocity.X,0,M.mover.PlaneVelocity.Y)
            local spd = vel.Magnitude
            mod = mod or 1
            if spd <= 0 then return end
            local fric = inAir and BhopCfg.AIR_FRICTION or BhopCfg.FRICTION
            local ctrl = spd < BhopCfg.GROUND_DECCEL and BhopCfg.GROUND_DECCEL or spd
            local drop = ctrl * fric * M.dt * mod
            local ns = math.max(spd - drop, 0)
            if spd > 0 and ns > 0 then ns = ns / spd end
            vel = vel * ns
            M.mover.PlaneVelocity = Vector2.new(vel.X, vel.Z)
        end
        local function applyGroundAccel(wd, ws)
            local cv = Vector3.new(M.mover.PlaneVelocity.X,0,M.mover.PlaneVelocity.Y)
            local cs = cv:Dot(wd)
            local add = ws - cs
            if add <= 0 then return end
            local ac = math.min(BhopCfg.GROUND_ACCEL * M.dt * ws, add)
            local nv = cv + ac * wd
            if nv.Magnitude > BhopCfg.RUN_SPEED then nv = nv.Unit * math.min(nv.Magnitude, BhopCfg.RUN_SPEED) end
            M.mover.PlaneVelocity = Vector2.new(nv.X, nv.Z)
        end
        local function applyAirAccel(wd, ws)
            local cv = Vector3.new(M.mover.PlaneVelocity.X,0,M.mover.PlaneVelocity.Y)
            local cs = cv:Dot(wd)
            local add = ws - cs
            if add <= 0 then return end
            local ac = math.min(BhopCfg.AIR_ACCEL * M.dt * ws, add)
            local nv = cv + ac * wd
            M.mover.PlaneVelocity = Vector2.new(nv.X, nv.Z)
        end
        local function applyGroundVel(gn)
            local wd = getMoveDir(gn)
            local ws = wd.Magnitude * BhopCfg.RUN_SPEED
            if M.states.air_friction <= 0 then
                applyFriction()
            else
                local sub = BhopCfg.AIR_MAX_SPEED_FRIC_DEC * M.dt * 60
                local curr = M.states.air_friction
                local fric = curr - sub
                if fric < 0 then fric = curr + fric end
                applyFriction(math.max(1, fric/BhopCfg.FRICTION))
                M.states.air_friction = math.max(0, curr - sub)
            end
            applyGroundAccel(wd, ws)
        end
        local function applyAirVel()
            local vel = Vector3.new(M.mover.PlaneVelocity.X,0,M.mover.PlaneVelocity.Y)
            local wd = getMoveDir(Vector3.new(0,1,0))
            local ws = wd.Magnitude * BhopCfg.AIR_SPEED
            if vel.Magnitude > BhopCfg.AIR_MAX_SPEED then M.states.air_friction = BhopCfg.AIR_MAX_SPEED_FRIC end
            if M.states.air_friction > 0 and not M.states.surfing then applyFriction(0.01 * M.states.air_friction, false) end
            applyAirAccel(wd, ws)
        end
        local function gravity()
            local mod = M.config.GRAVITY * M.dt
            M.collider.AssemblyLinearVelocity = Vector3.new(
                M.collider.AssemblyLinearVelocity.X,
                M.collider.AssemblyLinearVelocity.Y - mod,
                M.collider.AssemblyLinearVelocity.Z
            )
        end
        local function jump()
            M.states.jumping = true
            M.collider.AssemblyLinearVelocity = Vector3.new(
                M.collider.AssemblyLinearVelocity.X,
                M.config.JUMP_VELOCITY,
                M.collider.AssemblyLinearVelocity.Z
            )
        end
        local function process()
            local grnd, surf, res = isGrounded()
            M.states.grounded = grnd or false
            M.states.surfing = surf or false
            if M.collider.AssemblyLinearVelocity.Y < 0 then M.states.jumping = false end
            rotChar()
            if M.states.jumping or not M.states.grounded then
                applyAirVel()
                gravity()
            elseif M.Keys.Space > 0 then
                jump()
                applyAirVel()
            else
                applyGroundVel(res and res.Normal or Vector3.new(0,1,0))
            end
        end
        UserInputService.InputBegan:Connect(function(inp, gp)
            if inp.KeyCode and M.Keys[inp.KeyCode.Name] ~= nil then M.Keys[inp.KeyCode.Name] = 1 end
        end)
        UserInputService.InputEnded:Connect(function(inp, gp)
            if inp.KeyCode and M.Keys[inp.KeyCode.Name] ~= nil then M.Keys[inp.KeyCode.Name] = 0 end
        end)
        bhopConn = RunService.RenderStepped:Connect(function(dt)
            M.dt = dt
            process()
        end)
        return M
    end

    task.wait()
    makeBtn("BhopBtn", "BHOP (CS:GO): OFF", movFr, function()
        bhopEnabled = not bhopEnabled
        local b = movFr:FindFirstChild("BhopBtn")
        if b then
            b.Text = "BHOP (CS:GO): " .. (bhopEnabled and "ON" or "OFF")
            b.BackgroundColor3 = bhopEnabled and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if bhopEnabled then
            BhopMovement = initBhop()
        else
            if bhopConn then bhopConn:Disconnect() ; bhopConn = nil end
            if BhopMovement and BhopMovement.mover then pcall(function() BhopMovement.mover:Destroy() end) end
            workspace.Gravity = 196.2
        end
    end)

    makeLabel("TOUCH FLING", movFr)
    local flingActive = false
    makeBtn("FlingBtn", "Touch Fling: OFF", movFr, function()
        flingActive = not flingActive
        local b = movFr:FindFirstChild("FlingBtn")
        if b then
            b.Text = "Touch Fling: " .. (flingActive and "ON" or "OFF")
            b.BackgroundColor3 = flingActive and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if flingActive then
            task.spawn(function()
                local movel = 0.1
                while flingActive do
                    local c = LocalPlayer.Character
                    local hrp = c and c:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local vel = hrp.Velocity
                        hrp.Velocity = vel * 10000 + Vector3.new(0,10000,0)
                        RunService.RenderStepped:Wait()
                        hrp.Velocity = vel
                        RunService.Stepped:Wait()
                        hrp.Velocity = vel + Vector3.new(0,movel,0)
                        movel = -movel
                    end
                    RunService.Heartbeat:Wait()
                end
            end)
        end
    end)

    makeLabel("KNIFE HUD", movFr)
    local knifeEnabled = false
    local knifeGui = nil
    makeBtn("KnifeBtn", "Knife HUD: OFF", movFr, function()
        knifeEnabled = not knifeEnabled
        local b = movFr:FindFirstChild("KnifeBtn")
        if b then
            b.Text = "Knife HUD: " .. (knifeEnabled and "ON" or "OFF")
            b.BackgroundColor3 = knifeEnabled and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if knifeEnabled then
            if knifeGui then knifeGui:Destroy() end
            knifeGui = Instance.new("ScreenGui")
            knifeGui.Name = "KnifeHUD_SPRB"
            knifeGui.IgnoreGuiInset = true
            knifeGui.DisplayOrder = 9999
            knifeGui.ResetOnSpawn = false
            local km = Instance.new("ImageLabel")
            km.Size = UDim2.new(1,0,1,0)
            km.BackgroundTransparency = 1
            km.Image = "rbxassetid://13519444594"
            km.ScaleType = Enum.ScaleType.Fit
            km.ZIndex = 10
            km.Parent = knifeGui
            pcall(function() knifeGui.Parent = game:GetService("CoreGui") end)
            if not knifeGui.Parent then knifeGui.Parent = safeWaitChild(LocalPlayer, "PlayerGui", 10) end
            LocalPlayer.CameraMode = Enum.CameraMode.LockFirstPerson
        else
            if knifeGui then knifeGui:Destroy() ; knifeGui = nil end
            LocalPlayer.CameraMode = Enum.CameraMode.Classic
        end
    end)

    task.wait()
    end
    end
    function createReplaceLogic()
    customTools = {}

    local function findBlockFromPart(hit)
        if not hit then return nil end
        if not BlocksFolder then return nil end
        local obj = hit


        while obj and obj ~= Workspace do

            if obj:FindFirstChild("PPart") then

                local p = obj.Parent
                while p and p ~= Workspace do
                    if p == BlocksFolder then
                        return obj
                    end
                    p = p.Parent
                end

            end

            if obj.Name == "PPart" and obj.Parent then
                local model = obj.Parent
                if model:FindFirstChild("PPart") then
                    local p = model.Parent
                    while p and p ~= Workspace do
                        if p == BlocksFolder then
                            return model
                        end
                        p = p.Parent
                    end
                end
            end
            obj = obj.Parent
        end
        return nil
    end

    local function getFolder()
        local name = LocalPlayer.Name
        pcall(function()
            local tlObj = LocalPlayer.Team and LocalPlayer.Team:FindFirstChild("TeamLeader")
            if tlObj then
                local tl = tlObj.Value and tostring(tlObj.Value) or nil
                if not tl or tl == "" then pcall(function() tl = tlObj.Value.Name end) end
                if tl and tl ~= LocalPlayer.Name then
                    local sb = LocalPlayer:FindFirstChild("Settings") and LocalPlayer.Settings:FindFirstChild("ShareBlocks")
                    if sb and sb.Value == true then name = tl end
                end
            end
        end)
        return BlocksFolder:FindFirstChild(name)
    end

    R = {
        srcBlock = nil, srcName = nil, srcColor = nil, srcTransp = 0,
        repBlock = nil, repName = nil, repColor = nil,
        mode = "idle",
        highlights = {}, hlContainer = nil,
        isRunning = false,
        savedBlocks = {}, changedBlocks = {},
        toggleMat = false, toggleCol = false,
    }

    R.hlContainer = Instance.new("Folder")
    R.hlContainer.Name = "SPRB_RHL"
    pcall(function() R.hlContainer.Parent = Workspace end)

    local function clearHLs()
        for _, h in pairs(R.highlights) do
            if h and h.Parent then pcall(function() h:Destroy() end) end
        end
        R.highlights = {}
    end

    local function addHL(block, color)
        local pp = block:FindFirstChild("PPart")
        if not pp then return nil end
        if R.highlights[block] and R.highlights[block].Parent then
            pcall(function() R.highlights[block]:Destroy() end)
        end
        local h = Instance.new("BoxHandleAdornment")
        h.Adornee = pp
        h.Size = pp.Size + Vector3.new(0.15, 0.15, 0.15)
        h.Color3 = color
        h.Transparency = 0.2
        h.AlwaysOnTop = true
        h.ZIndex = 10
        h.Parent = R.hlContainer
        R.highlights[block] = h
        return h
    end

    local function collectReplaceBlocks(folder)
        local list = {}
        local wantMat = R.toggleMat
        local wantCol = R.toggleCol
        local hasSnapshot = recentlyPlacedBlocks and next(recentlyPlacedBlocks) ~= nil
        if not wantMat and not wantCol then
            if R.repBlock and R.repBlock.Parent and R.repBlock:FindFirstChild("PPart") then

                if not hasSnapshot or recentlyPlacedBlocks[R.repBlock] then
                    list[#list + 1] = R.repBlock
                end
            end
            return list
        end
        if #R.savedBlocks > 0 then
            for _, b in ipairs(R.savedBlocks) do
                if b and b.Parent and b:FindFirstChild("PPart") then
                    if not hasSnapshot or recentlyPlacedBlocks[b] then
                        list[#list + 1] = b
                    end
                end
            end
            return list
        end
        local rr = R.repColor and math.floor(R.repColor.R * 255) or -1
        local rg = R.repColor and math.floor(R.repColor.G * 255) or -1
        local rb = R.repColor and math.floor(R.repColor.B * 255) or -1
        local folders = {}
        if folder then
            folders[#folders + 1] = folder
            local repParent = R.repBlock and R.repBlock.Parent
            if repParent and repParent ~= folder and repParent.Parent == BlocksFolder then
                folders[#folders + 1] = repParent
            end
        else
            for _, pf in ipairs(BlocksFolder:GetChildren()) do
                if pf:IsA("Folder") then folders[#folders + 1] = pf end
            end
        end
        for _, f in ipairs(folders) do
            for _, b in pairs(f:GetChildren()) do
                local pp = b:FindFirstChild("PPart")
                if not pp then continue end

                if hasSnapshot and not recentlyPlacedBlocks[b] then continue end
                local match = true
                if wantMat and b.Name ~= R.repName then match = false end
                if wantCol and R.repColor then
                    local c = pp.Color
                    if math.floor(c.R*255) ~= rr or math.floor(c.G*255) ~= rg or math.floor(c.B*255) ~= rb then
                        match = false
                    end
                end
                if match then list[#list + 1] = b end
            end
        end
        return list
    end

    local function snapshotReplaceBlocks()
        R.savedBlocks = {}
        local folder = getFolder()
        if not folder then return end
        local wantMat = R.toggleMat
        local wantCol = R.toggleCol
        if not wantMat and not wantCol then
            if R.repBlock and R.repBlock:FindFirstChild("PPart") then
                R.savedBlocks[#R.savedBlocks + 1] = R.repBlock
            end
            return
        end
        local rr = R.repColor and math.floor(R.repColor.R * 255) or -1
        local rg = R.repColor and math.floor(R.repColor.G * 255) or -1
        local rb = R.repColor and math.floor(R.repColor.B * 255) or -1
        local folders = {}
        if folder then
            folders[#folders + 1] = folder
            local repParent = R.repBlock and R.repBlock.Parent
            if repParent and repParent ~= folder and repParent.Parent == BlocksFolder then
                folders[#folders + 1] = repParent
            end
        else
            for _, pf in ipairs(BlocksFolder:GetChildren()) do
                if pf:IsA("Folder") then folders[#folders + 1] = pf end
            end
        end
        for _, f in ipairs(folders) do
            for _, b in pairs(f:GetChildren()) do
                local pp = b:FindFirstChild("PPart")
                if not pp then continue end
                local match = true
                if wantMat and b.Name ~= R.repName then match = false end
                if wantCol and R.repColor then
                    local c = pp.Color
                    if math.floor(c.R*255) ~= rr or math.floor(c.G*255) ~= rg or math.floor(c.B*255) ~= rb then
                        match = false
                    end
                end
                if match then R.savedBlocks[#R.savedBlocks + 1] = b end
            end
        end
    end

    local function refreshHLs()
        clearHLs()
        if R.mode == "idle" then return end
        if R.srcBlock and R.srcBlock.Parent then
            addHL(R.srcBlock, Colors.Green)
        end
        if R.mode == "have_both" then
            if R.repBlock and R.repBlock.Parent then
                addHL(R.repBlock, Colors.Red)
            end
            local blocks = #R.savedBlocks > 0 and R.savedBlocks or (function()
                local folder = getFolder()
                return folder and collectReplaceBlocks(folder) or {}
            end)()
            for _, b in ipairs(blocks) do
                if b ~= R.repBlock then
                    addHL(b, Colors.ActiveBG)
                end
            end
        end
    end
    return findBlockFromPart, getFolder, clearHLs, addHL, refreshHLs, collectReplaceBlocks, snapshotReplaceBlocks
    end
    local _findBlockFromPart, _getFolder, _clearHLs, _addHL, _refreshHLs, _collectReplaceBlocks, _snapshotReplaceBlocks = createReplaceLogic()

    _RG = {}
    _RG.R = R
    _RG.clearHLs = _clearHLs
    _RG.addHL = _addHL
    _RG.refreshHLs = _refreshHLs
    _RG.collectReplaceBlocks = _collectReplaceBlocks
    _RG.snapshotReplaceBlocks = _snapshotReplaceBlocks
    _RG.getFolder = _getFolder
    _RG.findBlockFromPart = _findBlockFromPart
    _RG.customTools = customTools
    _RG.recentlyPlacedBlocks = recentlyPlacedBlocks
    function _G.createReplaceGUI_Part1(_RG)
    local clearHLs = _RG.clearHLs; local addHL = _RG.addHL; local refreshHLs = _RG.refreshHLs
    local collectReplaceBlocks = _RG.collectReplaceBlocks; local snapshotReplaceBlocks = _RG.snapshotReplaceBlocks
    local getFolder = _RG.getFolder; local findBlockFromPart = _RG.findBlockFromPart
    local customTools = _RG.customTools
    local sg = Instance.new("ScreenGui")
    sg.Name = "SPRB_ReplaceGUI"
    sg.ResetOnSpawn = false
    sg.IgnoreGuiInset = true
    sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    sg.Enabled = false
    pcall(function() sg.Parent = safeWaitChild(LocalPlayer, "PlayerGui", 10) end)

    local function rtween(obj, ti, props)
        local t = TweenService:Create(obj, ti, props); t:Play(); return t
    end

    local rBaseW, rBaseH = 480, 312
    local rW = math.floor(rBaseW * (Settings.uiScale or 1))
    local rH = math.floor(rBaseH * (Settings.uiScale or 1))
    do
        local sw = tonumber(Settings.replaceW)
        local sh = tonumber(Settings.replaceH)
        if sw and sh and sw >= 300 and sh >= 280 then
            rW = sw; rH = sh
        end
    end

    local panel = Instance.new("Frame")
    panel.Name = "Panel"
    panel.Size = UDim2.new(0, rW, 0, rH)
    do
        local rpx = tonumber(Settings.replacePosX)
        local rpy = tonumber(Settings.replacePosY)
        if rpx and rpy and rpx >= 0 and rpy >= 0 then
            panel.Position = UDim2.new(0, rpx, 0, rpy)
        else
            panel.Position = UDim2.new(0.5, -rW/2, 0.5, -rH/2)
        end
    end
    panel.BackgroundColor3 = Colors.BG
    panel.BackgroundTransparency = Settings.guiTransparency or 0.15
    panel.BorderSizePixel = 0
    panel.ClipsDescendants = true
    panel.Active = true
    panel.Parent = sg
    stylizeCard(panel, Colors.BG, Colors.Border, 6)
    local rpStroke = panel:FindFirstChildOfClass("UIStroke")
    if rpStroke then rpStroke.Transparency = 0.78; rpStroke.Thickness = 1 end

    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 44)
    titleBar.BackgroundColor3 = Colors.PanelElevated
    titleBar.BackgroundTransparency = 0.02
    titleBar.BorderSizePixel = 0
    titleBar.ZIndex = 2
    titleBar.Parent = panel
    local rtbCr = Instance.new("UICorner"); rtbCr.CornerRadius = UDim.new(0, 6); rtbCr.Parent = titleBar
    local rtbSt = Instance.new("UIStroke"); rtbSt.Color = Colors.Border; rtbSt.Transparency = 0.88; rtbSt.Thickness = 1; rtbSt.Parent = titleBar
    local rtbGrad = Instance.new("UIGradient")
    rtbGrad.Name = "SPRB_ReplaceGrad"
    rtbGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
        ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
    })
    rtbGrad.Rotation = 90; rtbGrad.Parent = titleBar

    local titleL = Instance.new("TextLabel")
    titleL.Size = UDim2.new(0.68, 0, 0, 32); titleL.Position = UDim2.new(0, 12, 0, 0)
    titleL.BackgroundTransparency = 1; titleL.ZIndex = 3
    titleL.Text = "REPLACE"; titleL.TextColor3 = Colors.Text
    titleL.TextSize = 24; titleL.Font = Enum.Font.GothamBold
    titleL.TextXAlignment = Enum.TextXAlignment.Left; titleL.Parent = titleBar

    local function makeRHeaderBtn(txt, xOff)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 30, 0, 30); b.Position = UDim2.new(1, xOff, 0.5, -15)
        b.BackgroundColor3 = Colors.Panel; b.BackgroundTransparency = 0; b.BorderSizePixel = 0
        b.Text = txt; b.TextColor3 = Colors.Text; b.TextSize = 15; b.Font = Enum.Font.GothamBold
        b.ZIndex = 4; b.AutoButtonColor = false; b.Parent = titleBar
        stylizeCard(b, Colors.Panel, Colors.Border, 6)
        b.MouseEnter:Connect(function() rtween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.PanelElevated}) end)
        b.MouseLeave:Connect(function() rtween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.Panel}) end)
        return b
    end
    local resetBtn = makeRHeaderBtn("-", -66)
    local closeBtn = makeRHeaderBtn("X", -34)

    local infoL = Instance.new("TextLabel")
    infoL.Name = "Info"
    infoL.Size = UDim2.new(1, -24, 0, 40); infoL.Position = UDim2.new(0, 12, 0, 54)
    infoL.BackgroundColor3 = Colors.PanelSoft; infoL.BackgroundTransparency = 0.3
    infoL.BorderSizePixel = 0; infoL.ZIndex = 3
    infoL.Text = "1st=TEMPLATE  2nd=WHAT TO CHANGE"; infoL.TextColor3 = Colors.Muted
    infoL.TextSize = 11; infoL.Font = Enum.Font.GothamBold
    infoL.TextWrapped = true; infoL.TextXAlignment = Enum.TextXAlignment.Left
    infoL.TextYAlignment = Enum.TextYAlignment.Center; infoL.Parent = panel
    local infoCr = Instance.new("UICorner"); infoCr.CornerRadius = UDim.new(0, 5); infoCr.Parent = infoL

    local dragOverlay = Instance.new("Frame")
    dragOverlay.Name = "DragOverlay"
    dragOverlay.Size = UDim2.new(1, -104, 1, 0)
    dragOverlay.Position = UDim2.new(0, 0, 0, 0)
    dragOverlay.BackgroundTransparency = 1
    dragOverlay.ZIndex = 10
    dragOverlay.Parent = titleBar
    local rDragActive = false
    local rDragStart, rPanelAbsX, rPanelAbsY
    dragOverlay.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            rDragActive = true; rDragStart = input.Position
            local abs = panel.AbsolutePosition
            rPanelAbsX = abs.X; rPanelAbsY = abs.Y
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if rDragActive and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - rDragStart
            local newX = rPanelAbsX + d.X
            local newY = rPanelAbsY + d.Y
            local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280,720)
            newX = math.clamp(newX, 4, math.max(4, vp.X - panel.AbsoluteSize.X - 4))
            newY = math.clamp(newY, 4, math.max(4, vp.Y - panel.AbsoluteSize.Y - 4))
            panel.Position = UDim2.new(0, newX, 0, newY)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if rDragActive and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            rDragActive = false
            Settings.replacePosX = math.floor(panel.AbsolutePosition.X + 0.5)
            Settings.replacePosY = math.floor(panel.AbsolutePosition.Y + 0.5)
            Settings.replaceW = math.floor(panel.AbsoluteSize.X + 0.5)
            Settings.replaceH = math.floor(panel.AbsoluteSize.Y + 0.5)
            saveSettings()
        end
    end)

    local rResizeHandle = Instance.new("TextButton")
    rResizeHandle.Name = "ResizeHandle"
    rResizeHandle.Size = UDim2.new(0, 18, 0, 18)
    rResizeHandle.Position = UDim2.new(1, -18, 1, -18)
    rResizeHandle.BackgroundColor3 = Colors.Panel
    rResizeHandle.BackgroundTransparency = 0.4
    rResizeHandle.BorderSizePixel = 0
    rResizeHandle.Text = ""
    rResizeHandle.ZIndex = 10
    rResizeHandle.AutoButtonColor = false
    rResizeHandle.Parent = panel
    local rRCr = Instance.new("UICorner"); rRCr.CornerRadius = UDim.new(0, 3); rRCr.Parent = rResizeHandle
    local rResizeIcon = Instance.new("ImageLabel")
    rResizeIcon.Size = UDim2.new(1, -4, 1, -4)
    rResizeIcon.Position = UDim2.new(0, 2, 0, 2)
    rResizeIcon.BackgroundTransparency = 1
    rResizeIcon.Image = "rbxassetid://2797468795"
    rResizeIcon.ImageColor3 = Colors.Muted
    rResizeIcon.ImageTransparency = 0.3
    rResizeIcon.ZIndex = 11
    rResizeIcon.Parent = rResizeHandle
    local function setupResize()
        local rResizeActive = false
        local rResizeStart, rResizeSizeStart
        local rMinW = 300
        local rMinH = 400
        local rMaxW = 800
        local rMaxH = 800
        rResizeHandle.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                rResizeActive = true
                rResizeStart = input.Position
                rResizeSizeStart = panel.Size
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if rResizeActive and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - rResizeStart
                local newW = math.clamp(rResizeSizeStart.X.Offset + d.X, rMinW, rMaxW)
                local newH = math.clamp(rResizeSizeStart.Y.Offset + d.Y, rMinH, rMaxH)
                panel.Size = UDim2.new(0, newW, 0, newH)
                rW = newW
                rH = newH
                _RG.rW = rW; _RG.rH = rH
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if rResizeActive and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
                rResizeActive = false
                Settings.replaceW = rW; Settings.replaceH = rH; saveSettings()
            end
        end)
    end
    setupResize()

    local function makeToggle(labelText, parent)
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 26)
        fr.BackgroundTransparency = 1
        fr.Parent = parent or panel
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 44, 0, 20)
        btn.Position = UDim2.new(0, 0, 0.5, -10)
        btn.BackgroundColor3 = Colors.PanelElevated
        btn.BorderSizePixel = 0
        btn.Text = "OFF"
        btn.TextColor3 = Colors.Muted
        btn.TextSize = 9; btn.Font = Enum.Font.GothamBold
        btn.AutoButtonColor = false
        btn.Parent = fr
        local bCr = Instance.new("UICorner"); bCr.CornerRadius = UDim.new(0, 5); bCr.Parent = btn
        local bSt = Instance.new("UIStroke"); bSt.Color = Colors.Border; bSt.Transparency = 0.75; bSt.Thickness = 1; bSt.Parent = btn
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -52, 0, 20)
        lbl.Position = UDim2.new(0, 52, 0.5, -10)
        lbl.BackgroundTransparency = 1
        lbl.Text = labelText
        lbl.TextColor3 = Colors.Text
        lbl.TextSize = 12; lbl.Font = Enum.Font.GothamBold
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = fr
        local state = false
        local function updateVisual()
            btn.Text = state and "ON" or "OFF"
            btn.BackgroundColor3 = state and Colors.ActiveBG or Colors.PanelElevated
            btn.TextColor3 = state and Colors.ActiveText or Colors.Muted
            bSt.Transparency = state and 0.3 or 0.75
        end
        btn.MouseButton1Click:Connect(function()
            state = not state
            updateVisual()
            rtween(btn, TweenInfo.new(0.12, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 48, 0, 22)})
            task.delay(0.12, function() if btn and btn.Parent then btn.Size = UDim2.new(0, 44, 0, 20) end end)
        end)
        return {
            btn = btn,
            getState = function() return state end,
            setSilent = function(v) state = v; updateVisual() end,
        }
    end

    local togglesFrame = Instance.new("Frame")
    togglesFrame.Size = UDim2.new(1, -24, 0, 64); togglesFrame.Position = UDim2.new(0, 12, 0, 104)
    togglesFrame.BackgroundTransparency = 1; togglesFrame.Parent = panel
    local tLayout = Instance.new("UIListLayout"); tLayout.Padding = UDim.new(0, 6); tLayout.Parent = togglesFrame

    local toggleMat = makeToggle("Same Material", togglesFrame)
    local toggleCol = makeToggle("Same Color", togglesFrame)

    local function onToggleChanged()
        R.toggleMat = toggleMat.getState()
        R.toggleCol = toggleCol.getState()
        if R.mode == "have_both" then
            refreshHLs()
            local folder = getFolder()
            local count = folder and #collectReplaceBlocks(folder) or 0
            infoL.Text = "SRC: " .. (R.srcName or "?") .. " -> REP: " .. (R.repName or "?") .. "\n" .. count .. " blocks | READY"
        end
    end
    toggleMat.btn.MouseButton1Click:Connect(onToggleChanged)
    toggleCol.btn.MouseButton1Click:Connect(onToggleChanged)

    local btnRow = Instance.new("Frame")
    btnRow.Size = UDim2.new(1, -24, 0, 34); btnRow.Position = UDim2.new(0, 12, 0, 180)
    btnRow.BackgroundTransparency = 1; btnRow.Parent = panel
    local btnLay = Instance.new("UIListLayout")
    btnLay.FillDirection = Enum.FillDirection.Horizontal
    btnLay.Padding = UDim.new(0, 8)
    btnLay.SortOrder = Enum.SortOrder.LayoutOrder
    btnLay.Parent = btnRow

    local doneBtn = Instance.new("TextButton")
    doneBtn.Size = UDim2.new(0.5, -4, 0, 34)
    doneBtn.BackgroundColor3 = Colors.PanelElevated
    doneBtn.BackgroundTransparency = 0; doneBtn.BorderSizePixel = 0
    doneBtn.Text = "CHANGE"
    doneBtn.TextColor3 = Colors.Text
    doneBtn.TextSize = 13; doneBtn.Font = Enum.Font.GothamBold
    doneBtn.AutoButtonColor = false; doneBtn.Parent = btnRow
    stylizeCard(doneBtn, Colors.PanelElevated, Colors.Border, 5)
    doneBtn.MouseEnter:Connect(function()
        rtween(doneBtn, TweenInfo.new(0.12), {BackgroundColor3 = Colors.Panel})
    end)
    doneBtn.MouseLeave:Connect(function()
        rtween(doneBtn, TweenInfo.new(0.12), {BackgroundColor3 = Colors.PanelElevated})
    end)
    doneBtn.MouseButton1Click:Connect(function()
        rtween(doneBtn, TweenInfo.new(0.08), {BackgroundTransparency = 0.4})
        task.wait(0.08)
        rtween(doneBtn, TweenInfo.new(0.12), {BackgroundTransparency = 0})
    end)

    resetBtn.MouseEnter:Connect(function()
        rtween(resetBtn, TweenInfo.new(0.12), {BackgroundColor3 = Colors.PanelElevated})
    end)
    resetBtn.MouseLeave:Connect(function()
        rtween(resetBtn, TweenInfo.new(0.12), {BackgroundColor3 = Colors.Panel})
    end)


    local rResizeGrip = Instance.new("Frame")
    rResizeGrip.Name = "ResizeGrip"; rResizeGrip.Size = UDim2.new(0, 22, 0, 22)
    rResizeGrip.Position = UDim2.new(1, -22, 1, -22); rResizeGrip.BackgroundTransparency = 1
    rResizeGrip.ZIndex = 50; rResizeGrip.Parent = panel
    local rGripIcon = Instance.new("TextLabel"); rGripIcon.Size = UDim2.new(1, 0, 1, 0)
    rGripIcon.BackgroundTransparency = 1; rGripIcon.Text = "\xe2\x97\xa3"
    rGripIcon.TextColor3 = Colors.Muted; rGripIcon.TextSize = 14; rGripIcon.ZIndex = 51
    rGripIcon.TextXAlignment = Enum.TextXAlignment.Right; rGripIcon.TextYAlignment = Enum.TextYAlignment.Bottom
    rGripIcon.Parent = rResizeGrip
    local rRsActive = false
    local rRsStart, rRsPanelSz
    rResizeGrip.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            rRsActive = true; rRsStart = i.Position; rRsPanelSz = panel.AbsoluteSize
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if rRsActive and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - rRsStart
            local nw = math.max(360, rRsPanelSz.X + d.X)
            local nh = math.max(280, rRsPanelSz.Y + d.Y)
            panel.Size = UDim2.new(0, nw, 0, nh)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if rRsActive and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then rRsActive = false end
    end)

    local helpCard = Instance.new("Frame")
    helpCard.Size = UDim2.new(1, -24, 0, 68); helpCard.Position = UDim2.new(0, 12, 0, 222)
    helpCard.BackgroundColor3 = Colors.PanelElevated; helpCard.BackgroundTransparency = 0.35
    helpCard.BorderSizePixel = 0; helpCard.Parent = panel
    stylizeCard(helpCard, Colors.PanelElevated, Colors.Border, 5)
    local hSt = helpCard:FindFirstChildOfClass("UIStroke"); if hSt then hSt.Transparency = 0.8 end
    local helpTitle = Instance.new("TextLabel")
    helpTitle.Size = UDim2.new(1, -16, 0, 16); helpTitle.Position = UDim2.new(0, 8, 0, 6)
    helpTitle.BackgroundTransparency = 1; helpTitle.Text = "HOW IT WORKS"; helpTitle.TextColor3 = Colors.Muted
    helpTitle.TextSize = 10; helpTitle.Font = Enum.Font.GothamBold; helpTitle.TextXAlignment = Enum.TextXAlignment.Left
    helpTitle.Parent = helpCard
    local helpBody = Instance.new("TextLabel")
    helpBody.Size = UDim2.new(1, -16, 0, 42); helpBody.Position = UDim2.new(0, 8, 0, 22)
    helpBody.BackgroundTransparency = 1
    helpBody.Text = "1. TEMPLATE   2. TARGET   3. FILTERS   4. CHANGE"
    helpBody.TextColor3 = Colors.Muted; helpBody.TextSize = 10; helpBody.Font = Enum.Font.GothamMedium
    helpBody.TextWrapped = true; helpBody.TextXAlignment = Enum.TextXAlignment.Left
    helpBody.TextYAlignment = Enum.TextYAlignment.Top; helpBody.Parent = helpCard

    RunService.Heartbeat:Connect(function()
        for block, h in pairs(R.highlights) do
            if h and h.Parent and block and block.Parent and block:FindFirstChild("PPart") then
                h.Adornee = block.PPart
                h.Size = block.PPart.Size + Vector3.new(0.15, 0.15, 0.15)
            else
                pcall(function() h:Destroy() end)
                R.highlights[block] = nil
            end
        end
    end)
    _RG.sg = sg; _RG.panel = panel; _RG.infoL = infoL; _RG.resetBtn = resetBtn; _RG.closeBtn = closeBtn
    _RG.rW = rW; _RG.rH = rH; _RG.toggleMat = toggleMat; _RG.toggleCol = toggleCol; _RG.doneBtn = doneBtn
    end

    function _G.createReplaceGUI_Part2(_RG)
    local clearHLs = _RG.clearHLs; local addHL = _RG.addHL; local refreshHLs = _RG.refreshHLs
    local collectReplaceBlocks = _RG.collectReplaceBlocks; local snapshotReplaceBlocks = _RG.snapshotReplaceBlocks
    local getFolder = _RG.getFolder; local findBlockFromPart = _RG.findBlockFromPart
    local customTools = _RG.customTools
    local panel = _RG.panel; local infoL = _RG.infoL; local resetBtn = _RG.resetBtn
    local toggleMat = _RG.toggleMat; local toggleCol = _RG.toggleCol; local doneBtn = _RG.doneBtn
    local rW = _RG.rW; local rH = _RG.rH
    local function rtween(obj, ti, props)
        local t = TweenService:Create(obj, ti, props); t:Play(); return t
    end

    local function fullReset()
        if R.isRunning then return end
        clearHLs()
        R.srcBlock = nil; R.srcName = nil; R.srcColor = nil; R.srcTransp = 0
        R.repBlock = nil; R.repName = nil; R.repColor = nil
        R.mode = "idle"; R.savedBlocks = {}
        toggleMat.setSilent(false); toggleCol.setSilent(false)
        infoL.Text = "1st=TEMPLATE  2nd=WHAT TO CHANGE"
    end
    resetBtn.MouseButton1Click:Connect(function()
        fullReset()
        rtween(resetBtn, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Rotation = -360})
        task.delay(0.5, function() if resetBtn and resetBtn.Parent then resetBtn.Rotation = 0 end end)
    end)

    local function executeReplace()
        if R.isRunning then return end
        if isBuilding then infoL.Text = "Builder busy!"; return end
        if not R.srcName or not R.repName then return end

        R.toggleMat = toggleMat.getState()
        R.toggleCol = toggleCol.getState()

        local folder = getFolder()
        if not folder then infoL.Text = "No zone!"; return end



        recentlyPlacedBlocks = {}
        pcall(function()
            for _, b in pairs(folder:GetChildren()) do
                if b:FindFirstChild("PPart") then
                    recentlyPlacedBlocks[b] = true
                end
            end
        end)
        if _RG then _RG.recentlyPlacedBlocks = recentlyPlacedBlocks end
        if next(recentlyPlacedBlocks) == nil then
            infoL.Text = "No blocks to change!"
            R.isRunning = false
            return
        end

        local toReplace = collectReplaceBlocks(folder)
        if #toReplace == 0 then infoL.Text = "No matching blocks!"; return end

        local myZone = getPlayerZone(LocalPlayer)
        if not myZone then infoL.Text = "No zone!"; return end

        local buildData = {}
        local idCounter = 1
        for _, b in ipairs(toReplace) do
            local pp = b:FindFirstChild("PPart")
            if not pp then continue end
            local relCF = myZone.CFrame:ToObjectSpace(pp.CFrame)
            local realTransp = pp.Transparency
            if not (R.srcName:sub(-5) == "Block") then
                for _, desc in pairs(b:GetChildren()) do
                    if (desc:IsA("BasePart") or desc:IsA("UnionOperation")) and desc ~= pp and desc.Transparency < 1 then
                        realTransp = desc.Transparency
                        break
                    end
                end
            end
            buildData[R.srcName] = buildData[R.srcName] or {}
            local entry = {
                CFrame = cfStr(relCF),
                Size = v3Str(pp.Size),
                Col = colStr(R.srcColor),
                Transparency = R.srcTransp,
                Anchored = pp.Anchored,
                CanCollide = pp.CanCollide,
                ShowShadow = pp.CastShadow ~= false,
                ID = idCounter,
            }
            idCounter = idCounter + 1
            local numVals = {}
            local boolVals = {}
            for _, child in pairs(b:GetChildren()) do
                if child:IsA("BoolValue") then
                    boolVals[child.Name] = child.Value
                elseif (child:IsA("NumberValue") or child:IsA("IntValue")) and not child.Name:find("^Bind") then
                    numVals[child.Name] = child.Value
                end
            end
            for _, child in pairs(pp:GetChildren()) do
                if child:IsA("BoolValue") then
                    boolVals[child.Name] = child.Value
                elseif (child:IsA("NumberValue") or child:IsA("IntValue")) and not child.Name:find("^Bind") then
                    numVals[child.Name] = child.Value
                end
            end
            if next(boolVals) then entry.BoolValues = boolVals end
            if next(numVals) then entry.NumberValues = numVals end
            table.insert(buildData[R.srcName], entry)
        end

        local totalBlocks = idCounter - 1
        if totalBlocks == 0 then infoL.Text = "No matching!"; return end

        R.isRunning = true
        clearHLs()

        local ch = Character or LocalPlayer.Character
        local rpTool = ch and ch:FindFirstChild("ReplaceSelect")
        if rpTool then pcall(function() rpTool.Parent = LocalPlayer.Backpack end) end
        task.wait(0.05)
        equipAllTools()
        task.wait(0.1)

        local dTool = ch and (ch:FindFirstChild("DeleteTool") or ch:FindFirstChild("DeletingTool"))
        local deleteRF = dTool and dTool:FindFirstChild("RF")
        if not deleteRF then
            infoL.Text = "Need Delete!"
            R.isRunning = false
            return
        end

        task.spawn(function()
            infoL.Text = "Deleting " .. totalBlocks .. "..."
            for _, b in ipairs(toReplace) do
                task.spawn(function()
                    if b and b.Parent then
                        pcall(function() deleteRF:InvokeServer(b) end)
                    end
                end)
            end
            task.wait(0.1)
            for _, b in ipairs(toReplace) do
                if b and b.Parent then
                    pcall(function() deleteRF:InvokeServer(b) end)
                end
            end
            task.wait(0.15)

            infoL.Text = "Building " .. totalBlocks .. "..."

            local buildOk, placedIds = pasteBuild(buildData, function(msg, pct)
                infoL.Text = msg
            end)
            if not buildOk then
                infoL.Text = "Build error!"
            else


                R.changedBlocks = {}
                recentlyPlacedBlocks = {}
                local changedCount = 0
                if placedIds then
                    for id, blk in pairs(placedIds) do
                        if type(blk) == "userdata" and blk:FindFirstChild("PPart") then
                            R.changedBlocks[#R.changedBlocks+1] = blk
                            recentlyPlacedBlocks[blk] = true
                            changedCount = changedCount + 1
                        end
                    end
                end

                if changedCount == 0 then
                    pcall(function()
                        for _, b in pairs(folder:GetChildren()) do
                            if b:FindFirstChild("PPart") and not recentlyPlacedBlocks[b] then
                                R.changedBlocks[#R.changedBlocks+1] = b
                                recentlyPlacedBlocks[b] = true
                                changedCount = changedCount + 1
                            end
                        end
                    end)
                end
                if _RG then _RG.recentlyPlacedBlocks = recentlyPlacedBlocks end
                infoL.Text = "Done: " .. totalBlocks .. " (" .. changedCount .. " changed)"
            end
            R.isRunning = false
            R.mode = "idle"
            R.savedBlocks = {}
            task.wait(0.3)
            local rpTool2 = LocalPlayer.Backpack:FindFirstChild("ReplaceSelect")
            if rpTool2 then pcall(function() local ch = Character or LocalPlayer.Character; local hum = ch and ch:FindFirstChildOfClass("Humanoid") or Humanoid; if hum then hum:EquipTool(rpTool2) end end) end
        end)
    end

    doneBtn.MouseButton1Click:Connect(executeReplace)

    do
        rW = math.floor(430 * (Settings.uiScale or 1))
        rH = math.floor(260 * (Settings.uiScale or 1))
        do
            local sw = tonumber(Settings.replaceW)
            local sh = tonumber(Settings.replaceH)
            if sw and sh and sw >= 300 and sh >= 280 then
                rW = sw; rH = sh
            end
        end
        panel.Size = UDim2.new(0, rW, 0, rH)
        do
            local rpx = tonumber(Settings.replacePosX)
            local rpy = tonumber(Settings.replacePosY)
            if rpx and rpy and rpx >= 0 and rpy >= 0 then
                panel.Position = UDim2.new(0, rpx, 0, rpy)
            else
                panel.Position = UDim2.new(0.5, -rW/2, 0.5, -rH/2)
            end
        end
        for _, child in ipairs(panel:GetChildren()) do
            if child:IsA("GuiObject") then child.Visible = false end
        end
        local uiRoot = Instance.new("Frame")
        uiRoot.Name = "ReplaceUIV2"
        uiRoot.Size = UDim2.new(1, 0, 1, 0)
        uiRoot.BackgroundTransparency = 1
        uiRoot.Visible = true
        uiRoot.Parent = panel
        local header = Instance.new("Frame")
        header.Name = "Header"
        header.Size = UDim2.new(1, 0, 0, 44)
        header.BackgroundColor3 = Colors.PanelElevated
        header.BackgroundTransparency = 0.02
        header.BorderSizePixel = 0
        header.Parent = uiRoot
        local hCr = Instance.new("UICorner"); hCr.CornerRadius = UDim.new(0, 6); hCr.Parent = header
        local hStroke = Instance.new("UIStroke"); hStroke.Color = Colors.Border; hStroke.Transparency = 0.88; hStroke.Thickness = 1; hStroke.Parent = header
        local hGrad = Instance.new("UIGradient"); hGrad.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Settings.primaryColor), ColorSequenceKeypoint.new(1, Settings.secondaryColor)}); hGrad.Rotation = 90; hGrad.Parent = header
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -112, 1, 0)
        title.Position = UDim2.new(0, 16, 0, 0)
        title.BackgroundTransparency = 1
        title.Text = "CHANGE"
        title.TextColor3 = Colors.Text
        title.TextSize = 24
        title.Font = Enum.Font.GothamBold
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = header
        local function headerButton(name, text, x)
            local b = Instance.new("TextButton")
            b.Name = name
            b.Size = UDim2.new(0, 34, 0, 34)
            b.Position = UDim2.new(1, x, 0.5, -17)
            b.BackgroundColor3 = Colors.PanelElevated
            b.BorderSizePixel = 0
            b.Text = text
            b.TextColor3 = Colors.Text
            b.TextSize = 15
            b.Font = Enum.Font.GothamBold
            b.AutoButtonColor = false
            b.Parent = header
            stylizeCard(b, Colors.PanelElevated, Colors.Border, 5)
            return b
        end
        resetBtn = headerButton("Reset", "R", -80)
        closeBtn = headerButton("Close", "X", -40)
        local status = Instance.new("Frame")
        status.Name = "Status"
        status.Size = UDim2.new(1, -24, 0, 54)
        status.Position = UDim2.new(0, 12, 0, 56)
        status.BackgroundColor3 = Colors.PanelElevated
        status.BackgroundTransparency = 0.08
        status.BorderSizePixel = 0
        status.Parent = uiRoot
        stylizeCard(status, Colors.PanelElevated, Colors.Border, 5)
        infoL = Instance.new("TextLabel")
        infoL.Name = "Info"
        infoL.Size = UDim2.new(1, -16, 1, 0)
        infoL.Position = UDim2.new(0, 8, 0, 0)
        infoL.BackgroundTransparency = 1
        infoL.Text = "Click template, then click target"
        infoL.TextColor3 = Colors.Text
        infoL.TextSize = 13
        infoL.Font = Enum.Font.GothamBold
        infoL.TextWrapped = true
        infoL.TextXAlignment = Enum.TextXAlignment.Left
        infoL.TextYAlignment = Enum.TextYAlignment.Center
        infoL.Parent = status
        local filterRow = Instance.new("Frame")
        filterRow.Name = "Filters"
        filterRow.Size = UDim2.new(1, -24, 0, 34)
        filterRow.Position = UDim2.new(0, 12, 0, 122)
        filterRow.BackgroundTransparency = 1
        filterRow.Parent = uiRoot
        local fLay = Instance.new("UIListLayout")
        fLay.FillDirection = Enum.FillDirection.Horizontal
        fLay.Padding = UDim.new(0, 8)
        fLay.Parent = filterRow
        local function makeCompactToggle(text)
            local b = Instance.new("TextButton")
            b.Size = UDim2.new(0.5, -4, 1, 0)
            b.BackgroundColor3 = Colors.PanelElevated
            b.BorderSizePixel = 0
            b.Text = text .. ": OFF"
            b.TextColor3 = Colors.Text
            b.TextSize = 12
            b.Font = Enum.Font.GothamBold
            b.AutoButtonColor = false
            b.Parent = filterRow
            stylizeCard(b, Colors.PanelElevated, Colors.Border, 5)
            local state = false
            local function set(v)
                state = v == true
                b.Text = text .. ": " .. (state and "ON" or "OFF")
                b.BackgroundColor3 = state and Colors.ActiveBG or Colors.PanelElevated
                b.TextColor3 = state and Colors.ActiveText or Colors.Text
            end
            b.MouseButton1Click:Connect(function()
                set(not state)
                R.toggleMat = toggleMat.getState()
                R.toggleCol = toggleCol.getState()
                if R.mode == "have_both" then
                    refreshHLs()
                    local folder = getFolder()
                    local count = folder and #collectReplaceBlocks(folder) or 0
                    infoL.Text = (R.srcName or "?") .. " -> " .. (R.repName or "?") .. "\n" .. count .. " blocks ready"
                end
            end)
            set(false)
            return {btn = b, getState = function() return state end, setSilent = set}
        end
        toggleMat = makeCompactToggle("MATERIAL")
        toggleCol = makeCompactToggle("COLOR")
        doneBtn = Instance.new("TextButton")
        doneBtn.Name = "Execute"
        doneBtn.Size = UDim2.new(1, -24, 0, 42)
        doneBtn.Position = UDim2.new(0, 12, 1, -54)
        doneBtn.BackgroundColor3 = Colors.PanelElevated
        doneBtn.BorderSizePixel = 0
        doneBtn.Text = "CHANGE"
        doneBtn.TextColor3 = Colors.Text
        doneBtn.TextSize = 14
        doneBtn.Font = Enum.Font.GothamBold
        doneBtn.AutoButtonColor = false
        doneBtn.Parent = uiRoot
        stylizeCard(doneBtn, Colors.PanelElevated, Colors.Border, 5)
        doneBtn.MouseButton1Click:Connect(executeReplace)
        resetBtn.MouseButton1Click:Connect(function()
            fullReset()
            infoL.Text = "Selection cleared"
        end)
        local dragActiveV2 = false
        local dragStartV2, panelStartV2
        header.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragActiveV2 = true
                dragStartV2 = input.Position
                panelStartV2 = panel.Position
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if dragActiveV2 and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - dragStartV2
                local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
                local nx = math.clamp(panelStartV2.X.Offset + d.X, 4, math.max(4, vp.X - panel.AbsoluteSize.X - 4))
                local ny = math.clamp(panelStartV2.Y.Offset + d.Y, 4, math.max(4, vp.Y - panel.AbsoluteSize.Y - 4))
                panel.Position = UDim2.new(0, nx, 0, ny)
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragActiveV2 = false end
        end)
        local grip = Instance.new("TextButton")
        grip.Name = "Resize"
        grip.Size = UDim2.new(0, 24, 0, 24)
        grip.Position = UDim2.new(1, -24, 1, -24)
        grip.BackgroundTransparency = 1
        grip.Text = "+"
        grip.TextColor3 = Colors.Muted
        grip.TextSize = 16
        grip.Font = Enum.Font.GothamBold
        grip.Parent = uiRoot
        local resizing = false
        local resizeStart, resizeSize
        grip.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                resizing = true
                resizeStart = input.Position
                resizeSize = panel.AbsoluteSize
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if resizing and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - resizeStart
                rW = math.clamp(resizeSize.X + d.X, 360, 680)
                rH = math.clamp(resizeSize.Y + d.Y, 230, 520)
                panel.Size = UDim2.new(0, rW, 0, rH)
                _RG.rW = rW; _RG.rH = rH
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then resizing = false end
        end)
    end
    _RG.fullReset = fullReset; _RG.executeReplace = executeReplace
    _RG.resetBtn = resetBtn; _RG.closeBtn = closeBtn; _RG.infoL = infoL
    _RG.toggleMat = toggleMat; _RG.toggleCol = toggleCol; _RG.doneBtn = doneBtn
    _RG.rW = rW; _RG.rH = rH
    end

    function _G.createReplaceGUI_Part3(_RG)
    local clearHLs = _RG.clearHLs; local addHL = _RG.addHL; local refreshHLs = _RG.refreshHLs
    local collectReplaceBlocks = _RG.collectReplaceBlocks; local snapshotReplaceBlocks = _RG.snapshotReplaceBlocks
    local getFolder = _RG.getFolder; local findBlockFromPart = _RG.findBlockFromPart
    local customTools = _RG.customTools
    local sg = _RG.sg; local panel = _RG.panel; local infoL = _RG.infoL
    local resetBtn = _RG.resetBtn; local closeBtn = _RG.closeBtn
    local toggleMat = _RG.toggleMat; local toggleCol = _RG.toggleCol; local doneBtn = _RG.doneBtn
    local fullReset = _RG.fullReset; local executeReplace = _RG.executeReplace
    local function rtween(obj, ti, props)
        local t = TweenService:Create(obj, ti, props); t:Play(); return t
    end

    local function showReplaceGUI(animated)
    local rW = _RG.rW or rW; local rH = _RG.rH or rH
        if sg.Enabled and panel.Visible then return end

        do
            local sw = tonumber(Settings.replaceW)
            local sh = tonumber(Settings.replaceH)
            if sw and sh and sw >= 300 and sh >= 280 then
                rW = sw; rH = sh
            end
        end
        sg.Enabled = true
        panel.Visible = true

        local rpx = tonumber(Settings.replacePosX)
        local rpy = tonumber(Settings.replacePosY)
        if rpx and rpy and rpx >= 0 and rpy >= 0 then
            panel.Position = UDim2.new(0, rpx, 0, rpy)
        else
            local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
            panel.Position = UDim2.new(0, (vp.X - rW) / 2, 0, (vp.Y - rH) / 2)
        end
        if not animated then
            panel.Size = UDim2.new(0, rW, 0, rH)
            panel.BackgroundTransparency = Settings.guiTransparency or 0.15
            return
        end
        playUISound(UISoundConfig.open)
        local curPos = panel.Position
        local startW = math.max(140, math.floor(rW * 0.32))
        local startH = 44
        local startOffX = curPos.X.Offset + (rW - startW) / 2
        local startOffY = curPos.Y.Offset + (rH - startH) / 2
        panel.Size = UDim2.new(0, startW, 0, startH)
        panel.Position = UDim2.new(0, startOffX, 0, startOffY)
        panel.BackgroundTransparency = 1
        rtween(panel, TweenInfo.new(0.32, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, rW, 0, rH),
            Position = UDim2.new(0, curPos.X.Offset, 0, curPos.Y.Offset),
            BackgroundTransparency = Settings.guiTransparency or 0.15
        })
        local boot = Instance.new("Frame")
        boot.Size = UDim2.new(1, 0, 1, 0); boot.BackgroundColor3 = Colors.BG; boot.BorderSizePixel = 0; boot.ZIndex = 40; boot.Parent = panel
        rtween(boot, TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
        task.delay(0.24, function() if boot and boot.Parent then boot:Destroy() end end)
    end

    local function hideReplaceGUI(animated)
    local rW = _RG.rW or rW; local rH = _RG.rH or rH
        if not sg.Enabled then return end

        Settings.replacePosX = math.floor(panel.AbsolutePosition.X + 0.5)
        Settings.replacePosY = math.floor(panel.AbsolutePosition.Y + 0.5)
        Settings.replaceW = math.floor(panel.AbsoluteSize.X + 0.5)
        Settings.replaceH = math.floor(panel.AbsoluteSize.Y + 0.5)
        saveSettings()
        if not animated then
            sg.Enabled = false
            panel.Size = UDim2.new(0, rW, 0, rH)
            panel.BackgroundTransparency = Settings.guiTransparency or 0.15
            return
        end
        playUISound(UISoundConfig.close)
        local curPos = panel.Position
        local endW, endH = 44, 44
        local endOffX = curPos.X.Offset + (rW - endW) / 2
        local endOffY = curPos.Y.Offset + (rH - endH) / 2
        rtween(panel, TweenInfo.new(0.24, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
            Size = UDim2.new(0, endW, 0, endH),
            Position = UDim2.new(0, endOffX, 0, endOffY),
            BackgroundTransparency = 1
        })
        task.wait(0.25)
        sg.Enabled = false
        panel.Size = UDim2.new(0, rW, 0, rH)
        panel.Position = curPos
        panel.BackgroundTransparency = Settings.guiTransparency or 0.15
    end

    _openReplaceGUI = function() showReplaceGUI(true) end

    closeBtn.MouseButton1Click:Connect(function()
        task.spawn(function()
            hideReplaceGUI(true)
            pcall(function()
                local ch = Character or LocalPlayer.Character
                local t = ch and ch:FindFirstChild("ReplaceSelect")
                if t then t.Parent = LocalPlayer.Backpack end
            end)
        end)
    end)

    local replaceTool = Instance.new("Tool")
    replaceTool.Name = "ReplaceSelect"
    replaceTool.RequiresHandle = false
    replaceTool.CanBeDropped = false
    local tt = Instance.new("StringValue"); tt.Name = "Tooltip"; tt.Value = "Click blocks to change"; tt.Parent = replaceTool
    table.insert(customTools, replaceTool)
    replaceTool.Equipped:Connect(function()
        task.defer(function()
            if replaceTool.Parent == Character then showReplaceGUI(not R.isRunning) end
        end)
    end)
    replaceTool.Unequipped:Connect(function()
        task.defer(function()
            if replaceTool.Parent ~= Character then hideReplaceGUI(not R.isRunning) end
        end)
    end)
    replaceTool.AncestryChanged:Connect(function(_, parent)
        if not parent then for i, v in ipairs(customTools) do if v == replaceTool then table.remove(customTools, i) break end end end
    end)
    R.selectReplaceTarget = function()
        if R.isRunning then return end
        local mouse = LocalPlayer:GetMouse()
        local target = mouse.Target
        if not target then infoL.Text = "Click on a block!"; return end


        local findFn = findBlockFromPart or (_RG and _RG.findBlockFromPart)
        if not findFn then
            infoL.Text = "Block finder not ready"
            return
        end
        local block = findFn(target)
        if not block then infoL.Text = "Not a block: " .. tostring(target); return end
        local pp = block:FindFirstChild("PPart")
        if not pp then return end
        local bName = block.Name
        local bColor = pp.Color
        local bMat = pp.Material
        local bTransp = pp.Transparency

        if R.mode == "idle" then
            R.srcBlock = block
            R.srcName = bName
            R.srcColor = bColor
            R.srcTransp = bTransp
            R.mode = "have_src"
            refreshHLs()
            infoL.Text = "TEMPLATE: " .. bName .. "\nNow click what to replace"

        elseif R.mode == "have_src" then
            if block == R.srcBlock then
                fullReset()
            else
                R.repBlock = block
                R.repName = bName
                R.repColor = bColor
                R.mode = "have_both"
                R.toggleMat = toggleMat.getState()
                R.toggleCol = toggleCol.getState()
                snapshotReplaceBlocks()
                refreshHLs()
                local count = #R.savedBlocks
                infoL.Text = R.srcName .. " -> " .. R.repName .. "\n" .. count .. " blocks | READY"
            end

        elseif R.mode == "have_both" then
            if block == R.srcBlock then
                fullReset()
            elseif block == R.repBlock then
                R.repBlock = nil; R.repName = nil; R.repColor = nil
                R.mode = "have_src"
                refreshHLs()
                infoL.Text = "SRC: " .. R.srcName .. "\nNow click block to REPLACE"
            else
                R.repBlock = block
                R.repName = bName
                R.repColor = bColor
                R.toggleMat = toggleMat.getState()
                R.toggleCol = toggleCol.getState()
                snapshotReplaceBlocks()
                refreshHLs()
                local count = #R.savedBlocks
                infoL.Text = "SRC: " .. R.srcName .. " -> REP: " .. R.repName .. "\n" .. count .. " blocks | READY"
            end
        end
    end
    local _rSelBusy = false
    local function safeSelectReplace()
        if _rSelBusy then return end
        _rSelBusy = true
        R.selectReplaceTarget()
        task.delay(0.15, function() _rSelBusy = false end)
    end
    replaceTool.Activated:Connect(safeSelectReplace)
    LocalPlayer:GetMouse().Button1Down:Connect(function()
        if replaceTool.Parent == Character then safeSelectReplace() end
    end)

    end

    function _G.createReplaceGUIContent()
        _G.createReplaceGUI_Part1(_RG)
        _G.createReplaceGUI_Part2(_RG)
        _G.createReplaceGUI_Part3(_RG)
    end

    function cleanupTools()
        for _, t in ipairs(customTools) do
            pcall(function() if t.Parent then t:Destroy() end end)
        end
        pcall(function()
            local g1 = LocalPlayer.PlayerGui:FindFirstChild("SPRB_PaintGUI")
            if g1 then g1:Destroy() end
            local g2 = LocalPlayer.PlayerGui:FindFirstChild("SPRB_ReplaceGUI")
            if g2 then g2:Destroy() end
        end)
        pcall(function()
            local h1 = Workspace:FindFirstChild("SPRB_RHL")
            if h1 then h1:Destroy() end
        end)
    end
    table.insert(preTerminateCallbacks, cleanupTools)
    task.wait()
    if not _G._afterCreateUI then _G._afterCreateUI = {} end
    table.insert(_G._afterCreateUI, function()
        _G.createReplaceGUIContent()
    end)

    _PG = {}
    function _G.createPaintGUI_Part1(_PG)
    P = { isRunning = false, selectedBlock = nil, selectedName = nil, selectedColor = nil, transpValue = 0, shimmerActive = false, shimmerHue = 0, shimmerSpeed = 6, shimmerSat = 0.85, shimmerVal = 0.92, shimmerSpread = 360, shimmerThread = nil }

    local function ptween(obj, ti, props)
        local t = TweenService:Create(obj, ti, props); t:Play(); return t
    end

    local sg = Instance.new("ScreenGui")
    sg.Name = "SPRB_PaintGUI"; sg.ResetOnSpawn = false; sg.IgnoreGuiInset = true
    sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    pcall(function() sg.Parent = safeWaitChild(LocalPlayer, "PlayerGui", 10) end)
    sg.Enabled = false

    local pW = math.floor(640 * (Settings.uiScale or 1))
    local pH = math.floor(360 * (Settings.uiScale or 1))
    do
        local sw = tonumber(Settings.paintW)
        local sh = tonumber(Settings.paintH)
        if sw and sh and sw >= 340 and sh >= 420 then
            pW = sw; pH = sh
        end
    end

    local panel = Instance.new("Frame")
    panel.Name = "Panel"
    panel.Size = UDim2.new(0, pW, 0, pH)
    do
        local ppx = tonumber(Settings.paintPosX)
        local ppy = tonumber(Settings.paintPosY)
        if ppx and ppy and ppx >= 0 and ppy >= 0 then
            panel.Position = UDim2.new(0, ppx, 0, ppy)
        else
            panel.Position = UDim2.new(0.5, -pW/2, 0.5, -pH/2)
        end
    end
    panel.BackgroundColor3 = Colors.BG
    panel.BackgroundTransparency = Settings.guiTransparency or 0.15
    panel.BorderSizePixel = 0
    panel.ClipsDescendants = true
    panel.Active = true
    panel.Parent = sg
    stylizeCard(panel, Colors.BG, Colors.Border, 6)
    do local pStroke = panel:FindFirstChildOfClass("UIStroke"); if pStroke then pStroke.Transparency = 0.78; pStroke.Thickness = 1 end end

    local resetBtn, closeBtn, infoL
    do
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 44)
    titleBar.BackgroundColor3 = Colors.PanelElevated
    titleBar.BackgroundTransparency = 0.02
    titleBar.BorderSizePixel = 0
    titleBar.ZIndex = 2
    titleBar.Parent = panel
    local tbCr = Instance.new("UICorner"); tbCr.CornerRadius = UDim.new(0, 6); tbCr.Parent = titleBar
    local tbSt = Instance.new("UIStroke"); tbSt.Color = Colors.Border; tbSt.Transparency = 0.88; tbSt.Thickness = 1; tbSt.Parent = titleBar
    local tbGrad = Instance.new("UIGradient")
    tbGrad.Name = "SPRB_PaintGrad"
    tbGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
        ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
    })
    tbGrad.Rotation = 90; tbGrad.Parent = titleBar

    local titleL = Instance.new("TextLabel")
    titleL.Size = UDim2.new(0.68, 0, 0, 32); titleL.Position = UDim2.new(0, 12, 0, 0)
    titleL.BackgroundTransparency = 1; titleL.ZIndex = 3
    titleL.Text = "PAINT"; titleL.TextColor3 = Colors.Text
    titleL.TextSize = 24; titleL.Font = Enum.Font.GothamBold
    titleL.TextXAlignment = Enum.TextXAlignment.Left; titleL.Parent = titleBar

    local function makePHeaderBtn(txt, xOff)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0, 30, 0, 30); b.Position = UDim2.new(1, xOff, 0.5, -15)
        b.BackgroundColor3 = Colors.Panel; b.BackgroundTransparency = 0; b.BorderSizePixel = 0
        b.Text = txt; b.TextColor3 = Colors.Text; b.TextSize = 15; b.Font = Enum.Font.GothamBold
        b.ZIndex = 4; b.AutoButtonColor = false; b.Parent = titleBar
        stylizeCard(b, Colors.Panel, Colors.Border, 6)
        b.MouseEnter:Connect(function() ptween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.PanelElevated}) end)
        b.MouseLeave:Connect(function() ptween(b, TweenInfo.new(0.12), {BackgroundColor3 = Colors.Panel}) end)
        return b
    end
    resetBtn = makePHeaderBtn("\xe2\x86\xba", -66)
    closeBtn = makePHeaderBtn("X", -34)


    infoL = Instance.new("TextLabel"); infoL.Name = "Info"
    infoL.Size = UDim2.new(0, 200, 0, 28); infoL.Position = UDim2.new(1, -264, 0, 8)
    infoL.BackgroundTransparency = 1; infoL.ZIndex = 3; infoL.Text = ""
    infoL.TextColor3 = Colors.Text; infoL.TextSize = 11; infoL.Font = Enum.Font.GothamBold
    infoL.TextTruncate = Enum.TextTruncate.AtEnd; infoL.TextXAlignment = Enum.TextXAlignment.Right
    infoL.TextYAlignment = Enum.TextYAlignment.Center; infoL.Parent = titleBar

    local dragOverlay = Instance.new("Frame")
    dragOverlay.Name = "DragOverlay"
    dragOverlay.Size = UDim2.new(1, -104, 1, 0)
    dragOverlay.Position = UDim2.new(0, 0, 0, 0)
    dragOverlay.BackgroundTransparency = 1
    dragOverlay.ZIndex = 10
    dragOverlay.Parent = titleBar
    local dragActive = false
    local dragStart, dragAbsX, dragAbsY
    dragOverlay.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragActive = true; dragStart = input.Position
            local abs = panel.AbsolutePosition
            dragAbsX = abs.X; dragAbsY = abs.Y
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragActive and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            local newX = dragAbsX + d.X
            local newY = dragAbsY + d.Y
            local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
            newX = math.clamp(newX, -pW + 80, vp.X - 80)
            newY = math.clamp(newY, 0, vp.Y - 40)
            panel.Position = UDim2.new(0, newX, 0, newY)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if dragActive and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then dragActive = false end
    end)
    end

    do
    local resizeHandle = Instance.new("TextButton")
    resizeHandle.Name = "ResizeHandle"
    resizeHandle.Size = UDim2.new(0, 18, 0, 18)
    resizeHandle.Position = UDim2.new(1, -18, 1, -18)
    resizeHandle.BackgroundColor3 = Colors.Panel
    resizeHandle.BackgroundTransparency = 0.4
    resizeHandle.BorderSizePixel = 0
    resizeHandle.Text = ""
    resizeHandle.ZIndex = 10
    resizeHandle.AutoButtonColor = false
    resizeHandle.Parent = panel
    local rHCr = Instance.new("UICorner"); rHCr.CornerRadius = UDim.new(0, 3); rHCr.Parent = resizeHandle
    local resizeIcon = Instance.new("ImageLabel")
    resizeIcon.Size = UDim2.new(1, -4, 1, -4)
    resizeIcon.Position = UDim2.new(0, 2, 0, 2)
    resizeIcon.BackgroundTransparency = 1
    resizeIcon.Image = "rbxassetid://2797468795"
    resizeIcon.ImageColor3 = Colors.Muted
    resizeIcon.ImageTransparency = 0.3
    resizeIcon.ZIndex = 11
    resizeIcon.Parent = resizeHandle

    local resizeActive = false
    local resizeStartPos, resizeStartSize
    local MIN_PW = 340
    local MIN_PH = 420
    local MAX_PW = 900
    local MAX_PH = 900
    resizeHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            resizeActive = true
            resizeStartPos = input.Position
            resizeStartSize = panel.Size
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if resizeActive and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - resizeStartPos
            local newW = math.clamp(resizeStartSize.X.Offset + d.X, MIN_PW, MAX_PW)
            local newH = math.clamp(resizeStartSize.Y.Offset + d.Y, MIN_PH, MAX_PH)
            panel.Size = UDim2.new(0, newW, 0, newH)
            pW = newW
            pH = newH
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if resizeActive and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            resizeActive = false
            Settings.paintW = pW; Settings.paintH = pH; saveSettings()
        end
    end)
    end

    local content = Instance.new("Frame")
    content.Name = "Content"; content.Size = UDim2.new(1, -16, 1, -100)
    content.Position = UDim2.new(0, 8, 0, 92); content.BackgroundTransparency = 1; content.Parent = panel

    local subTabBar
    do
    subTabBar = Instance.new("Frame")
    subTabBar.Name = "SubTabBar"; subTabBar.Size = UDim2.new(1, 0, 0, 36)
    subTabBar.BackgroundColor3 = Colors.PanelSoft; subTabBar.BackgroundTransparency = 0.3
    subTabBar.BorderSizePixel = 0; subTabBar.Parent = panel
    stylizeCard(subTabBar, Colors.PanelSoft, Colors.Border, 6)
    subTabBar.Position = UDim2.new(0, 8, 0, 50)
    do local sbl = Instance.new("UIListLayout"); sbl.FillDirection = Enum.FillDirection.Horizontal; sbl.Padding = UDim.new(0, 4); sbl.VerticalAlignment = Enum.VerticalAlignment.Center; sbl.Parent = subTabBar end
    do local sbp = Instance.new("UIPadding"); sbp.PaddingLeft = UDim.new(0, 6); sbp.PaddingRight = UDim.new(0, 6); sbp.PaddingTop = UDim.new(0, 4); sbp.PaddingBottom = UDim.new(0, 4); sbp.Parent = subTabBar end
    end

    local subTabs = {}
    local subPages = {}
    local activeSubTab = nil
    local staggerFadeIn
    local function switchSubTab(idx)
        if activeSubTab == idx then return end
        activeSubTab = idx
        for i, btn in ipairs(subTabs) do
            local isActive = (i == idx)
            ptween(btn, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                BackgroundColor3 = isActive and Colors.ActiveBG or Colors.PanelElevated,
                BackgroundTransparency = isActive and 0 or 0.06,
                TextColor3 = isActive and Colors.ActiveText or Colors.Muted,
            })
        end
        for i, page in ipairs(subPages) do
            if i == idx then
                page.Visible = true
                page.Position = UDim2.new(0, 10, 0, 0)
                ptween(page, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 0, 0, 0)}):Play()
                staggerFadeIn(page, 0.035)
            else
                page.Visible = false
            end
        end
        playUISound(UISoundConfig.click)
    end
    local subTabNames = {"SELECT", "PAINT", "SHIMMER"}
    for i, name in ipairs(subTabNames) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.333, -3, 1, 0)
        btn.BackgroundColor3 = (i == 1) and Colors.ActiveBG or Colors.PanelElevated
        btn.BackgroundTransparency = (i == 1) and 0 or 0.06
        btn.BorderSizePixel = 0
        btn.Text = name; btn.TextColor3 = (i == 1) and Colors.ActiveText or Colors.Muted
        btn.TextSize = 11; btn.Font = Enum.Font.GothamBold
        btn.AutoButtonColor = false; btn.Parent = subTabBar
        local bcr = Instance.new("UICorner"); bcr.CornerRadius = UDim.new(0, 4); bcr.Parent = btn
        local bst = Instance.new("UIStroke"); bst.Color = Colors.Border; bst.Transparency = 0.75; bst.Thickness = 1; bst.Parent = btn
        local bsc = Instance.new("UIScale"); bsc.Scale = 1; bsc.Parent = btn
        btn.MouseEnter:Connect(function()
            if i ~= activeSubTab then
                ptween(btn, TweenInfo.new(0.12), {BackgroundTransparency = 0})
                ptween(bsc, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.04})
            end
        end)
        btn.MouseLeave:Connect(function()
            if i ~= activeSubTab then
                ptween(btn, TweenInfo.new(0.12), {BackgroundTransparency = 0.06})
                ptween(bsc, TweenInfo.new(0.12), {Scale = 1})
            end
        end)
        btn.MouseButton1Click:Connect(function() switchSubTab(i) end)
        subTabs[i] = btn
    end

    local pageSelect = Instance.new("Frame")
    pageSelect.Name = "PageSelect"; pageSelect.Size = UDim2.new(1, 0, 1, 0)
    pageSelect.BackgroundTransparency = 1; pageSelect.Parent = content
    local pagePaint = Instance.new("Frame")
    pagePaint.Name = "PagePaint"; pagePaint.Size = UDim2.new(1, 0, 1, 0)
    pagePaint.BackgroundTransparency = 1; pagePaint.Visible = false; pagePaint.Parent = content
    local pageShimmer = Instance.new("Frame")
    pageShimmer.Name = "PageShimmer"; pageShimmer.Size = UDim2.new(1, 0, 1, 0)
    pageShimmer.BackgroundTransparency = 1; pageShimmer.Visible = false; pageShimmer.Parent = content
    subPages = {pageSelect, pagePaint, pageShimmer}
    activeSubTab = 1

    local leftCol = Instance.new("Frame"); leftCol.Size = UDim2.new(1, 0, 1, 0)
    leftCol.BackgroundTransparency = 1; leftCol.Name = "LeftCol"; leftCol.Parent = pageSelect
    do local ll = Instance.new("UIListLayout"); ll.Padding = UDim.new(0, 6); ll.SortOrder = Enum.SortOrder.LayoutOrder; ll.Parent = leftCol end

    local paintCol = Instance.new("ScrollingFrame")
    paintCol.Size = UDim2.new(1, -4, 1, 0); paintCol.Position = UDim2.new(0, 2, 0, 0)
    paintCol.BackgroundTransparency = 1; paintCol.BorderSizePixel = 0
    paintCol.ScrollBarThickness = 3; paintCol.ScrollBarImageColor3 = Colors.Muted
    paintCol.CanvasSize = UDim2.new(0, 0, 0, 0); paintCol.AutomaticCanvasSize = Enum.AutomaticSize.Y; pcall(function() paintCol.ElasticBehavior = Enum.ElasticBehavior.Never end)
    paintCol.ScrollingDirection = Enum.ScrollingDirection.Y
    paintCol.Name = "PaintCol"; paintCol.Parent = pagePaint
    do local pl = Instance.new("UIListLayout"); pl.Padding = UDim.new(0, 4); pl.SortOrder = Enum.SortOrder.LayoutOrder; pl.Parent = paintCol end

    local shimmerCol = Instance.new("ScrollingFrame"); shimmerCol.Size = UDim2.new(1, -4, 1, 0)
    shimmerCol.BackgroundTransparency = 1; shimmerCol.BorderSizePixel = 0
    shimmerCol.ScrollBarThickness = 3; shimmerCol.ScrollBarImageColor3 = Colors.Muted
    shimmerCol.CanvasSize = UDim2.new(0, 0, 0, 0); shimmerCol.AutomaticCanvasSize = Enum.AutomaticSize.Y; pcall(function() shimmerCol.ElasticBehavior = Enum.ElasticBehavior.Never end)
    shimmerCol.ScrollingDirection = Enum.ScrollingDirection.Y
    shimmerCol.Name = "ShimmerCol"; shimmerCol.Parent = pageShimmer
    do local sl = Instance.new("UIListLayout"); sl.Padding = UDim.new(0, 6); sl.SortOrder = Enum.SortOrder.LayoutOrder; sl.Parent = shimmerCol end

    local function sectionLbl(text, parent, order)
        local h = Instance.new("TextLabel"); h.LayoutOrder = order or 0; h.Size = UDim2.new(1, 0, 0, 18)
        h.BackgroundTransparency = 1; h.Text = "  " .. text; h.TextColor3 = Colors.Muted; h.TextSize = 11
        h.Font = Enum.Font.GothamBold; h.TextXAlignment = Enum.TextXAlignment.Left; h.Parent = parent
        local dot = Instance.new("Frame"); dot.Size = UDim2.new(0, 4, 0, 4); dot.Position = UDim2.new(0, 2, 0.5, -2)
        dot.BackgroundColor3 = Colors.ActiveBG; dot.BackgroundTransparency = 0.3; dot.BorderSizePixel = 0; dot.Parent = h
        local dCr = Instance.new("UICorner"); dCr.CornerRadius = UDim.new(1, 0); dCr.Parent = dot
        return h
    end
    local function mkCard(parent, order, h)
        local f = Instance.new("Frame"); f.LayoutOrder = order or 0; f.Size = UDim2.new(1, 0, 0, h or 4)
        f.BackgroundColor3 = Colors.PanelElevated; f.BackgroundTransparency = 0.35
        f.BorderSizePixel = 0; f.Parent = parent
        stylizeCard(f, Colors.PanelElevated, Colors.Border, 6)
        local s = f:FindFirstChildOfClass("UIStroke"); if s then s.Transparency = 0.75 end
        local bgSave = f.BackgroundTransparency
        f.MouseEnter:Connect(function()
            ptween(f, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 0.18})
            if s then ptween(s, TweenInfo.new(0.15), {Transparency = 0.45, Color = Colors.ActiveBG}) end
        end)
        f.MouseLeave:Connect(function()
            ptween(f, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = bgSave})
            if s then ptween(s, TweenInfo.new(0.15), {Transparency = 0.75, Color = Colors.Border}) end
        end)
        return f
    end

    local function staggerFadeInFn(parent, delay)
        task.spawn(function()
            local children = {}
            for _, c in ipairs(parent:GetChildren()) do
                if c:IsA("GuiObject") then children[#children+1] = c end
            end
            for i, c in ipairs(children) do
                task.wait(delay or 0.03)
                pcall(function()
                    local origTrans = c.BackgroundTransparency
                    c.BackgroundTransparency = 1
                    local origPos = c.Position
                    c.Position = UDim2.new(origPos.X.Scale, origPos.X.Offset + 8, origPos.Y.Scale, origPos.Y.Offset)
                    ptween(c, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {BackgroundTransparency = origTrans, Position = origPos})
                end)
            end
        end)
    end
    staggerFadeIn = staggerFadeInFn

    sectionLbl("SELECTED BLOCK", leftCol, 0)
    local selColorPrev, selNameL, selInfoL
    do
    local selCard = mkCard(leftCol, 1, 46)
    selColorPrev = Instance.new("Frame"); selColorPrev.Size = UDim2.new(0, 34, 0, 34)
    selColorPrev.Position = UDim2.new(0, 8, 0.5, -17)
    selColorPrev.BackgroundColor3 = Color3.fromRGB(128, 128, 128); selColorPrev.BorderSizePixel = 0; selColorPrev.Parent = selCard
    local scpCr = Instance.new("UICorner"); scpCr.CornerRadius = UDim.new(0, 6); scpCr.Parent = selColorPrev
    local scpSt = Instance.new("UIStroke"); scpSt.Color = Colors.Border; scpSt.Transparency = 0.65; scpSt.Thickness = 1; scpSt.Parent = selColorPrev
    selNameL = Instance.new("TextLabel"); selNameL.Size = UDim2.new(1, -52, 0, 18); selNameL.Position = UDim2.new(0, 48, 0, 6)
    selNameL.BackgroundTransparency = 1; selNameL.Text = "None"; selNameL.TextColor3 = Colors.Text
    selNameL.TextSize = 12; selNameL.Font = Enum.Font.GothamBold; selNameL.TextXAlignment = Enum.TextXAlignment.Left
    selNameL.TextTruncate = Enum.TextTruncate.AtEnd; selNameL.Parent = selCard
    selInfoL = Instance.new("TextLabel"); selInfoL.Size = UDim2.new(1, -52, 0, 14); selInfoL.Position = UDim2.new(0, 48, 0, 26)
    selInfoL.BackgroundTransparency = 1; selInfoL.Text = "Click a block to select"; selInfoL.TextColor3 = Colors.Muted
    selInfoL.TextSize = 11; selInfoL.Font = Enum.Font.GothamMedium; selInfoL.TextXAlignment = Enum.TextXAlignment.Left
    selInfoL.TextTruncate = Enum.TextTruncate.AtEnd; selInfoL.Parent = selCard
    end

    task.wait()
    sectionLbl("COLORS", leftCol, 2)
    local fromPicker = makeColorPicker("FROM COLOR", Color3.fromRGB(255, 60, 60), leftCol, function(c) end)
    local toPicker = makeColorPicker("TO COLOR", Color3.fromRGB(60, 200, 80), leftCol, function(c) end)

    do
    makeSlider("PaintTransp", 0, 100, math.floor((P.transpValue or 0) * 100 + 0.5), leftCol, "Transparency",
        function(v) return math.floor(v + 0.5) .. "%" end,
        function(v)
            P.transpValue = v / 100
        end
    )
    end
    _PG.sg = sg; _PG.panel = panel; _PG.pW = pW; _PG.pH = pH
    _PG.resetBtn = resetBtn; _PG.closeBtn = closeBtn; _PG.infoL = infoL
    _PG.content = content; _PG.subTabBar = subTabBar
    _PG.subTabs = subTabs; _PG.subPages = subPages; _PG.activeSubTab = activeSubTab
    _PG.pageSelect = pageSelect; _PG.pagePaint = pagePaint; _PG.pageShimmer = pageShimmer
    _PG.leftCol = leftCol; _PG.paintCol = paintCol; _PG.shimmerCol = shimmerCol
    _PG.staggerFadeIn = staggerFadeIn; _PG.ptween = ptween
    _PG.sectionLbl = sectionLbl; _PG.mkCard = mkCard
    _PG.selColorPrev = selColorPrev; _PG.selNameL = selNameL; _PG.selInfoL = selInfoL
    _PG.fromPicker = fromPicker; _PG.toPicker = toPicker
    end

    function _G.createPaintGUI_Part2(_PG)
    local panel = _PG.panel; local infoL = _PG.infoL; local resetBtn = _PG.resetBtn
    local closeBtn = _PG.closeBtn; local pW = _PG.pW; local pH = _PG.pH
    local paintCol = _PG.paintCol; local shimmerCol = _PG.shimmerCol; local sg = _PG.sg
    local ptween = _PG.ptween; local sectionLbl = _PG.sectionLbl; local mkCard = _PG.mkCard
    local selColorPrev = _PG.selColorPrev; local selNameL = _PG.selNameL; local selInfoL = _PG.selInfoL
    local fromPicker = _PG.fromPicker; local toPicker = _PG.toPicker
    local leftCol = _PG.leftCol

    local getFolder = _RG and _RG.getFolder
    local updateShimmerVisual

    do
    local resizeGrip = Instance.new("Frame")
    resizeGrip.Name = "ResizeGrip"; resizeGrip.Size = UDim2.new(0, 22, 0, 22)
    resizeGrip.Position = UDim2.new(1, -22, 1, -22); resizeGrip.BackgroundTransparency = 1
    resizeGrip.ZIndex = 50; resizeGrip.Parent = panel
    local gripIcon = Instance.new("TextLabel"); gripIcon.Size = UDim2.new(1, 0, 1, 0)
    gripIcon.BackgroundTransparency = 1; gripIcon.Text = "\xe2\x97\xa3"
    gripIcon.TextColor3 = Colors.Muted; gripIcon.TextSize = 12; gripIcon.ZIndex = 6
    gripIcon.TextXAlignment = Enum.TextXAlignment.Right; gripIcon.TextYAlignment = Enum.TextYAlignment.Bottom
    gripIcon.Parent = resizeGrip
    local rsActive = false
    local rsStart, rsPanelSz
    resizeGrip.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            rsActive = true; rsStart = i.Position; rsPanelSz = panel.AbsoluteSize
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if rsActive and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - rsStart
            local nw = math.max(420, rsPanelSz.X + d.X)
            local nh = math.max(280, rsPanelSz.Y + d.Y)
            panel.Size = UDim2.new(0, nw, 0, nh)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if rsActive and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then rsActive = false end
    end)
    end

    local function getPaintRF()
        local ch = Character or LocalPlayer.Character
        local pa = (ch and ch:FindFirstChild("PaintingTool")) or LocalPlayer.Backpack:FindFirstChild("PaintingTool")
        local pr = (ch and ch:FindFirstChild("PropertiesTool")) or LocalPlayer.Backpack:FindFirstChild("PropertiesTool")
        return pa and pa:FindFirstChild("RF"), pr and pr:FindFirstChild("SetPropertieRF")
    end
    local function getAllBlocks(folder)
        local list = {}; if not folder then return list end
        for _, b in pairs(folder:GetChildren()) do if b:FindFirstChild("PPart") then list[#list+1] = b end end; return list
    end
    local function colMatch(c1, c2)
        return math.floor(c1.R*255) == math.floor(c2.R*255) and math.floor(c1.G*255) == math.floor(c2.G*255) and math.floor(c1.B*255) == math.floor(c2.B*255)
    end
    local function unequipAndReturn()
        local ch = Character or LocalPlayer.Character
        local t = ch and ch:FindFirstChild("PaintToolExtended")
        if t then pcall(function() t.Parent = LocalPlayer.Backpack end) end
        task.wait(0.05); equipAllTools(); task.wait(0.1)
    end
    local function unequipBuildTools()
        local ch = Character or LocalPlayer.Character
        local buildToolNames = {"PaintToolExtended", "PaintingTool", "PropertiesTool", "ScalingTool", "BuildingTool", "ReplaceSelect"}
        for _, tool in pairs(ch and ch:GetChildren() or {}) do
            if tool:IsA("Tool") then
                for _, bn in ipairs(buildToolNames) do
                    if tool.Name == bn then
                        pcall(function() tool.Parent = LocalPlayer.Backpack end)
                        break
                    end
                end
            end
        end
    end
    local function doPaint(targets, toCol, doTransp)
        local paintRF, propRF = getPaintRF()
        if not paintRF then infoL.Text = "Need Paint!"; return false end
        if type(targets) ~= "table" or #targets == 0 then infoL.Text = "Nothing selected"; return false end
        local batch = {}; for _, b in ipairs(targets) do batch[#batch+1] = {b, toCol} end
        for bi = 1, #batch, 200 do
            local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end
            local ok = pcall(function() paintRF:InvokeServer(chunk) end)
            if not ok then infoL.Text = "Paint remote failed"; playUISound(UISoundConfig.error); return false end
            task.wait()
        end
        if doTransp and propRF and P.transpValue > 0 then
            local ts = tostring(math.floor(P.transpValue*100+0.5))
            for bi = 1, #targets, 50 do
                local chunk = {}; for j = bi, math.min(bi+49, #targets) do chunk[#chunk+1] = targets[j] end
                local ok = pcall(function() propRF:InvokeServer("Transparency", chunk, ts) end)
                if not ok then infoL.Text = "Transparency remote failed"; playUISound(UISoundConfig.error); return false end
                task.wait()
            end
        end
        return true
    end
    local function resetSelection()
        P.selectedBlock = nil; P.selectedName = nil; P.selectedColor = nil
        selNameL.Text = "None"; selInfoL.Text = "Click a block to select"
        ptween(selColorPrev, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(128,128,128)})
    end
    resetBtn.MouseButton1Click:Connect(function()
        resetSelection(); infoL.Text = "Selection cleared"
        ptween(resetBtn, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Rotation = -360})
        task.delay(0.5, function() if resetBtn and resetBtn.Parent then resetBtn.Rotation = 0 end end)
    end)


    do
    sectionLbl("PAINT ACTIONS", paintCol, 0)
    local function mkActBtn(name, text, parent, order, cb)
        local b = Instance.new("TextButton"); b.Name = name; b.LayoutOrder = order or 0
        b.Size = UDim2.new(1, 0, 0, 34); b.BackgroundColor3 = Colors.PanelElevated; b.BackgroundTransparency = 0
        b.BorderSizePixel = 0; b.Text = ""; b.AutoButtonColor = false; b.Parent = parent
        local _, bs = stylizeCard(b, Colors.PanelElevated, Colors.Border, 4)
        local acc = Instance.new("Frame"); acc.Size = UDim2.new(0, 3, 0.6, 0); acc.Position = UDim2.new(0, 6, 0.2, 0)
        acc.BackgroundColor3 = Colors.ActiveBG; acc.BackgroundTransparency = 0.6; acc.BorderSizePixel = 0; acc.Parent = b
        local accCr = Instance.new("UICorner"); accCr.CornerRadius = UDim.new(0, 2); accCr.Parent = acc
        local tl = Instance.new("TextLabel"); tl.Size = UDim2.new(1, -22, 1, 0); tl.Position = UDim2.new(0, 16, 0, 0)
        tl.BackgroundTransparency = 1; tl.Text = text; tl.TextColor3 = Colors.Text
        tl.TextSize = 12; tl.Font = Enum.Font.GothamMedium; tl.TextXAlignment = Enum.TextXAlignment.Left
        tl.TextTruncate = Enum.TextTruncate.AtEnd; tl.Parent = b
        local aScale = Instance.new("UIScale"); aScale.Scale = 1; aScale.Parent = b
        b.MouseEnter:Connect(function()
            ptween(bs, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = 0.4, Color = Colors.ActiveBG})
            ptween(acc, TweenInfo.new(0.15), {BackgroundTransparency = 0.1, Size = UDim2.new(0, 4, 0.72, 0)})
            ptween(b, TweenInfo.new(0.15), {BackgroundColor3 = Colors.Panel})
            ptween(aScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.015})
        end)
        b.MouseLeave:Connect(function()
            ptween(bs, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = 0.82, Color = Colors.Border})
            ptween(acc, TweenInfo.new(0.2), {BackgroundTransparency = 0.6, Size = UDim2.new(0, 3, 0.6, 0)})
            ptween(b, TweenInfo.new(0.2), {BackgroundColor3 = Colors.PanelElevated})
            ptween(aScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Scale = 1})
        end)
        b.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            ptween(b, TweenInfo.new(0.06), {BackgroundTransparency = 0.4})
            ptween(aScale, TweenInfo.new(0.06, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 0.97})
            task.wait(0.06)
            ptween(b, TweenInfo.new(0.12), {BackgroundTransparency = 0})
            ptween(aScale, TweenInfo.new(0.1, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1.015})
            if cb then task.spawn(cb) end
        end)
        return b
    end

    mkActBtn("SwapColor", "Color Swap", paintCol, 1, function()
        if P.isRunning then return end
        if not P.selectedName then infoL.Text = "Select a block!"; return end
        local fromC, toC = fromPicker.getColor(), toPicker.getColor()
        local folder = getFolder(); if not folder then infoL.Text = "No zone!"; return end
        local targets = {}
        for _, b in ipairs(getAllBlocks(folder)) do
            if b.Name == P.selectedName and b:FindFirstChild("PPart") and colMatch(b.PPart.Color, fromC) then targets[#targets+1] = b end
        end
        if #targets == 0 then infoL.Text = "No matching!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            infoL.Text = "Swapping " .. #targets .. "..."
            if doPaint(targets, toC, true) then infoL.Text = "Done: " .. #targets .. " changed" end
            P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("PaintMat", "Paint Material", paintCol, 2, function()
        if P.isRunning then return end
        if not P.selectedName then infoL.Text = "Select a block!"; return end
        local toC = toPicker.getColor(); local folder = getFolder()
        if not folder then infoL.Text = "No folder!"; return end
        local targets = {}
        for _, b in ipairs(getAllBlocks(folder)) do if b.Name == P.selectedName then targets[#targets+1] = b end end
        if #targets == 0 then infoL.Text = "No blocks!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            infoL.Text = "Painting " .. #targets .. "..."
            if doPaint(targets, toC, true) then infoL.Text = "Done: " .. #targets .. " painted" end
            P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("PaintAll", "Paint ALL", paintCol, 3, function()
        if P.isRunning then return end
        local toC = toPicker.getColor(); local folder = getFolder()
        if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder); if #all == 0 then infoL.Text = "No blocks!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            infoL.Text = "Painting ALL: " .. #all .. "..."
            if doPaint(all, toC, true) then infoL.Text = "Done: " .. #all .. " painted" end
            P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("RandColors", "Random Colors", paintCol, 4, function()
        if P.isRunning then return end
        local folder = getFolder(); if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder); if #all == 0 then infoL.Text = "No blocks!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            local paintRF = getPaintRF()
            if not paintRF then infoL.Text = "Need Paint tool!"; P.isRunning = false; return end
            infoL.Text = "Randomizing " .. #all .. "..."
            local batch = {}
            for _, b in ipairs(all) do batch[#batch+1] = {b, Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255))} end
            for bi = 1, #batch, 200 do local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end; pcall(function() paintRF:InvokeServer(chunk) end) end
            infoL.Text = "Done: " .. #all .. " randomized"; P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("RandPerMat", "Random per Mat", paintCol, 5, function()
        if P.isRunning then return end
        local folder = getFolder(); if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder); if #all == 0 then infoL.Text = "No blocks!"; return end
        local matC = {}
        for _, b in ipairs(all) do if not matC[b.Name] then matC[b.Name] = Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255)) end end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            local paintRF = getPaintRF()
            if not paintRF then infoL.Text = "Need Paint tool!"; P.isRunning = false; return end
            infoL.Text = "Coloring by mat..."
            local batch = {}; for _, b in ipairs(all) do batch[#batch+1] = {b, matC[b.Name]} end
            for bi = 1, #batch, 200 do local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end; pcall(function() paintRF:InvokeServer(chunk) end) end
            infoL.Text = "Done: " .. #all .. " painted"; P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("SetTransp", "Set Transparency", paintCol, 6, function()
        if P.isRunning then return end
        local tVal = P.transpValue
        if not tVal or tVal < 0 or tVal > 1 then infoL.Text = "Transparency 0-1!"; return end
        local folder = getFolder(); if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder); if #all == 0 then infoL.Text = "No blocks!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            local _, propRF = getPaintRF()
            if not propRF then infoL.Text = "Need Properties!"; P.isRunning = false; return end
            infoL.Text = "Setting transparency..."
            local ts = tostring(math.floor(tVal*100+0.5))
            for bi = 1, #all, 50 do local chunk = {}; for j = bi, math.min(bi+49, #all) do chunk[#chunk+1] = all[j] end; pcall(function() propRF:InvokeServer("Transparency", chunk, ts) end) end
            infoL.Text = "Done: " .. #all .. " transparency set"; P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("PaintSingle", "Paint Single", paintCol, 7, function()
        if P.isRunning then return end
        if not P.selectedBlock or not P.selectedBlock:FindFirstChild("PPart") then infoL.Text = "Select a block!"; return end
        local toC = toPicker.getColor()
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            if doPaint({P.selectedBlock}, toC, true) then infoL.Text = "Done: 1 painted" end
            P.isRunning = false; unequipBuildTools()
        end)
    end)
    mkActBtn("InvertColors", "Invert FROM/TO", paintCol, 8, function()
        local fc, tc = fromPicker.getColor(), toPicker.getColor()
        fromPicker.setColor(tc); toPicker.setColor(fc); infoL.Text = "Colors inverted"
    end)
    mkActBtn("CopyToFrom", "TO = FROM", paintCol, 9, function()
        toPicker.setColor(fromPicker.getColor()); infoL.Text = "TO copied from FROM"
    end)
    mkActBtn("RandomSaturated", "Random Saturated", paintCol, 10, function()
        if P.isRunning then return end
        local folder = getFolder(); if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder); if #all == 0 then infoL.Text = "No blocks!"; return end
        P.isRunning = true; unequipAndReturn()
        task.spawn(function()
            local paintRF = getPaintRF()
            if not paintRF then infoL.Text = "Need Paint tool!"; P.isRunning = false; return end
            infoL.Text = "Saturating " .. #all .. "..."
            local batch = {}
            for _, b in ipairs(all) do
                local h = math.random() * 360; local s = 0.7 + math.random() * 0.3; local v = 0.6 + math.random() * 0.4
                local c = Color3.fromHSV(h/360, s, v)
                batch[#batch+1] = {b, c}
            end
            for bi = 1, #batch, 200 do local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end; pcall(function() paintRF:InvokeServer(chunk) end) end
            infoL.Text = "Done: " .. #all .. " saturated"; P.isRunning = false; unequipBuildTools()
        end)
    end)
    end

    do
    local function makeShimmerSlider(labelText, minVal, maxVal, default, fmt, parent, order)
        local card = mkCard(parent, order, 32)
        local lbl = Instance.new("TextLabel"); lbl.Size = UDim2.new(0, 90, 1, 0); lbl.Position = UDim2.new(0, 10, 0, 0)
        lbl.BackgroundTransparency = 1; lbl.Text = labelText; lbl.TextColor3 = Colors.Muted
        lbl.TextSize = 11; lbl.Font = Enum.Font.GothamBold; lbl.TextXAlignment = Enum.TextXAlignment.Left; lbl.Parent = card
        local track = Instance.new("Frame"); track.Size = UDim2.new(1, -140, 0, 12); track.Position = UDim2.new(0, 100, 0.5, -6)
        track.BackgroundColor3 = Colors.PanelSoft; track.BorderSizePixel = 0; track.Parent = card
        local trCr = Instance.new("UICorner"); trCr.CornerRadius = UDim.new(0, 6); trCr.Parent = track
        local trSt = Instance.new("UIStroke"); trSt.Color = Colors.Border; trSt.Transparency = 0.8; trSt.Thickness = 1; trSt.Parent = track
        local fill = Instance.new("Frame"); fill.Size = UDim2.new(0, 0, 1, 0); fill.BackgroundColor3 = Colors.ActiveBG
        fill.BackgroundTransparency = 0.3; fill.BorderSizePixel = 0; fill.Parent = track
        local flCr = Instance.new("UICorner"); flCr.CornerRadius = UDim.new(0, 6); flCr.Parent = fill
        local thumb = Instance.new("Frame"); thumb.Size = UDim2.new(0, 18, 0, 18); thumb.Position = UDim2.new(0, -9, 0.5, -9)
        thumb.BackgroundColor3 = Color3.fromRGB(245, 245, 250); thumb.BorderSizePixel = 0; thumb.Parent = track
        local thCr = Instance.new("UICorner"); thCr.CornerRadius = UDim.new(0, 9); thCr.Parent = thumb
        local thSt = Instance.new("UIStroke"); thSt.Color = Colors.Border; thSt.Transparency = 0.35; thSt.Thickness = 1; thSt.Parent = thumb
        local valL = Instance.new("TextLabel"); valL.Size = UDim2.new(0, 36, 1, 0); valL.Position = UDim2.new(1, -40, 0, 0)
        valL.BackgroundTransparency = 1; valL.Text = string.format(fmt, default); valL.TextColor3 = Colors.Muted
        valL.TextSize = 12; valL.Font = Enum.Font.GothamBold; valL.TextXAlignment = Enum.TextXAlignment.Right; valL.Parent = card
        local curVal = default
        local function setVal(v)
            curVal = math.clamp(v, minVal, maxVal)
            local rel = (curVal - minVal) / (maxVal - minVal)
            fill.Size = UDim2.new(rel, 0, 1, 0); thumb.Position = UDim2.new(rel, -9, 0.5, -9)
            valL.Text = string.format(fmt, curVal)
        end
        setVal(default)
        local sDrag = false
        local function updS(xPos)
            local rel = math.clamp((xPos - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
            setVal(minVal + rel * (maxVal - minVal))
        end
        thumb.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then sDrag = true end end)
        track.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then sDrag = true; updS(i.Position.X) end end)
        UserInputService.InputChanged:Connect(function(i) if sDrag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then updS(i.Position.X) end end)
        UserInputService.InputEnded:Connect(function(i) if sDrag and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then sDrag = false end end)
        return {
            get = function() return curVal end,
            set = setVal,
        }
    end

    (function()
    sectionLbl("SHIMMER", shimmerCol, 0)
    local shimmerCard = mkCard(shimmerCol, 1, 28)
    local shLbl = Instance.new("TextLabel"); shLbl.Size = UDim2.new(0, 78, 1, 0); shLbl.Position = UDim2.new(0, 8, 0, 0)
    shLbl.BackgroundTransparency = 1; shLbl.Text = "Shimmer"; shLbl.TextColor3 = Colors.Muted
    shLbl.TextSize = 10; shLbl.Font = Enum.Font.GothamBold; shLbl.TextXAlignment = Enum.TextXAlignment.Left; shLbl.Parent = shimmerCard
    local shToggleBtn = Instance.new("TextButton"); shToggleBtn.Size = UDim2.new(0, 54, 0, 20); shToggleBtn.Position = UDim2.new(0, 88, 0.5, -10)
    shToggleBtn.BackgroundColor3 = Colors.PanelElevated; shToggleBtn.BorderSizePixel = 0
    shToggleBtn.Text = "OFF"; shToggleBtn.TextColor3 = Colors.Muted; shToggleBtn.TextSize = 9; shToggleBtn.Font = Enum.Font.GothamBold
    shToggleBtn.AutoButtonColor = false; shToggleBtn.Parent = shimmerCard
    local shTgCr = Instance.new("UICorner"); shTgCr.CornerRadius = UDim.new(0, 5); shTgCr.Parent = shToggleBtn
    local shTgSt = Instance.new("UIStroke"); shTgSt.Color = Colors.Border; shTgSt.Transparency = 0.75; shTgSt.Thickness = 1; shTgSt.Parent = shToggleBtn
    local shInfoL = Instance.new("TextLabel"); shInfoL.Size = UDim2.new(1, -158, 1, 0); shInfoL.Position = UDim2.new(1, -70, 0, 0)
    shInfoL.BackgroundTransparency = 1; shInfoL.Text = "off"; shInfoL.TextColor3 = Colors.Muted
    shInfoL.TextSize = 9; shInfoL.Font = Enum.Font.GothamMedium; shInfoL.TextXAlignment = Enum.TextXAlignment.Right; shInfoL.Parent = shimmerCard

    local shSpeed = makeShimmerSlider("Speed", 1, 30, 6, "%.0f", shimmerCol, 8)
    local shSat = makeShimmerSlider("Saturation", 0, 1, 0.85, "%.2f", shimmerCol, 9)
    local shBright = makeShimmerSlider("Brightness", 0.2, 1, 0.92, "%.2f", shimmerCol, 10)
    local shSpread = makeShimmerSlider("Spread", 0, 360, 360, "%.0f", shimmerCol, 11)

    local shModeCard = mkCard(shimmerCol, 6, 26)
    local shModeLbl = Instance.new("TextLabel"); shModeLbl.Size = UDim2.new(0, 50, 1, 0); shModeLbl.Position = UDim2.new(0, 6, 0, 0)
    shModeLbl.BackgroundTransparency = 1; shModeLbl.Text = "Mode"; shModeLbl.TextColor3 = Colors.Muted
    shModeLbl.TextSize = 9; shModeLbl.Font = Enum.Font.GothamBold; shModeLbl.TextXAlignment = Enum.TextXAlignment.Left; shModeLbl.Parent = shModeCard
    local shModeBtn = Instance.new("TextButton"); shModeBtn.Size = UDim2.new(1, -60, 0, 20); shModeBtn.Position = UDim2.new(0, 56, 0.5, -10)
    shModeBtn.BackgroundColor3 = Colors.PanelElevated; shModeBtn.BorderSizePixel = 0
    shModeBtn.Text = "Rainbow"; shModeBtn.TextColor3 = Colors.Text; shModeBtn.TextSize = 9; shModeBtn.Font = Enum.Font.GothamSemibold
    shModeBtn.AutoButtonColor = false; shModeBtn.Parent = shModeCard
    local shMCr = Instance.new("UICorner"); shMCr.CornerRadius = UDim.new(0, 4); shMCr.Parent = shModeBtn
    local shMSt = Instance.new("UIStroke"); shMSt.Color = Colors.Border; shMSt.Transparency = 0.7; shMSt.Thickness = 1; shMSt.Parent = shModeBtn
    local shModes = {"Rainbow", "Pulse", "Solid", "Gradient"}
    local shModeIdx = 1
    local shCurrentMode = "Rainbow"
    local shModeMenu
    local shModeCloseConn
    local function closeShModeMenu()
        if shModeMenu and shModeMenu.Parent then shModeMenu:Destroy() end
        shModeMenu = nil
        if shModeCloseConn then shModeCloseConn:Disconnect(); shModeCloseConn = nil end
    end
    shModeBtn.MouseButton1Click:Connect(function()
        if shModeMenu and shModeMenu.Parent then closeShModeMenu(); return end
        local absPos = shModeBtn.AbsolutePosition
        local absSize = shModeBtn.AbsoluteSize
        shModeMenu = Instance.new("Frame")
        shModeMenu.Size = UDim2.new(0, absSize.X, 0, #shModes * 22)
        shModeMenu.Position = UDim2.new(0, absPos.X, 0, absPos.Y + absSize.Y + 4)
        shModeMenu.BackgroundColor3 = Colors.Panel
        shModeMenu.BorderSizePixel = 0; shModeMenu.ZIndex = 500; shModeMenu.Parent = sg
        local mCr = Instance.new("UICorner"); mCr.CornerRadius = UDim.new(0, 4); mCr.Parent = shModeMenu
        local mSt = Instance.new("UIStroke"); mSt.Color = Colors.Border; mSt.Thickness = 1; mSt.Parent = shModeMenu
        local mLay = Instance.new("UIListLayout"); mLay.Padding = UDim.new(0, 0); mLay.Parent = shModeMenu
        for idx, mode in ipairs(shModes) do
            local item = Instance.new("TextButton"); item.Size = UDim2.new(1, 0, 0, 22)
            item.BackgroundColor3 = (idx == shModeIdx) and Colors.ActiveBG or Colors.Panel
            item.BorderSizePixel = 0; item.Text = mode; item.TextColor3 = (idx == shModeIdx) and Colors.ActiveText or Colors.Text
            item.TextSize = 9; item.Font = Enum.Font.GothamMedium; item.AutoButtonColor = false; item.Parent = shModeMenu
            item.MouseButton1Click:Connect(function()
                shModeIdx = idx; shCurrentMode = mode; shModeBtn.Text = mode
                closeShModeMenu()
                playUISound(UISoundConfig.click)
            end)
        end
        shModeCloseConn = UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                local mp = input.Position
                local mp2 = Vector2.new(mp.X, mp.Y)
                local ap = shModeMenu.AbsolutePosition
                local as = shModeMenu.AbsoluteSize
                if mp2.X < ap.X or mp2.X > ap.X + as.X or mp2.Y < ap.Y or mp2.Y > ap.Y + as.Y then
                    closeShModeMenu()
                end
            end
        end)
    end)

    local function updateShimmerToggleVisual()
        if P.shimmerActive then
            shToggleBtn.Text = "ON"
            shToggleBtn.BackgroundColor3 = Colors.ActiveBG
            shToggleBtn.TextColor3 = Colors.ActiveText
            shTgSt.Transparency = 0.3
        else
            shToggleBtn.Text = "OFF"
            shToggleBtn.BackgroundColor3 = Colors.PanelElevated
            shToggleBtn.TextColor3 = Colors.Muted
            shTgSt.Transparency = 0.75
        end
    end
    updateShimmerVisual = updateShimmerToggleVisual

    local function stopShimmer()
        P.shimmerActive = false
        updateShimmerToggleVisual()
        shInfoL.Text = "off"
    end

    local function startShimmer()
        if P.shimmerActive then return end
        local folder = getFolder()
        if not folder then infoL.Text = "No folder!"; return end
        local all = getAllBlocks(folder)
        if #all == 0 then infoL.Text = "No blocks!"; return end
        P.shimmerActive = true
        updateShimmerToggleVisual()
        shInfoL.Text = #all .. " blocks"
        P.shimmerThread = task.spawn(function()
            local cached = all
            local lastCount = #cached
            local refreshCounter = 0
            while P.shimmerActive and sg and sg.Parent do
                local rf = getPaintRF()
                if rf then
                    refreshCounter = refreshCounter + 1
                    if refreshCounter >= 20 then
                        refreshCounter = 0
                        local f = getFolder()
                        if f then
                            local fresh = getAllBlocks(f)
                            if #fresh ~= lastCount then cached = fresh; lastCount = #cached end
                        end
                    end
                    local n = #cached
                    if n > 0 then
                        local spread = shSpread.get()
                        local sat = shSat.get()
                        local val = shBright.get()
                        local baseHue = P.shimmerHue
                        local mode = shCurrentMode
                        local batch = {}
                        for i, b in ipairs(cached) do
                            if b and b.Parent then
                                local c
                                if mode == "Rainbow" then
                                    local h = ((baseHue + (i * spread / n)) % 360) / 360
                                    c = Color3.fromHSV(h, sat, val)
                                elseif mode == "Pulse" then
                                    local p = 0.5 + 0.5 * math.sin((baseHue + i * 6) * math.pi / 180)
                                    local h = (baseHue % 360) / 360
                                    c = Color3.fromHSV(h, sat, math.clamp(val * (0.4 + 0.6 * p), 0.1, 1))
                                elseif mode == "Solid" then
                                    local h = (baseHue % 360) / 360
                                    c = Color3.fromHSV(h, sat, val)
                                elseif mode == "Gradient" then
                                    local t = (i / n)
                                    local h = ((baseHue + t * spread) % 360) / 360
                                    c = Color3.fromHSV(h, sat, val)
                                else
                                    local h = ((baseHue + (i * spread / n)) % 360) / 360
                                    c = Color3.fromHSV(h, sat, val)
                                end
                                batch[#batch+1] = {b, c}
                            end
                        end
                        local chunkSize = 150
                        for bi = 1, #batch, chunkSize do
                            if not P.shimmerActive then break end
                            local chunk = {}
                            for j = bi, math.min(bi+chunkSize-1, #batch) do chunk[#chunk+1] = batch[j] end
                            pcall(function() rf:InvokeServer(chunk) end)
                            if #batch > chunkSize then task.wait(0.02) end
                        end
                    end
                end
                P.shimmerHue = (P.shimmerHue + shSpeed.get()) % 360
                task.wait(0.22)
            end
            P.shimmerThread = nil
        end)
    end

    shToggleBtn.MouseButton1Click:Connect(function()
        if P.shimmerActive then
            stopShimmer()
            infoL.Text = "Shimmer stopped"
        else
            startShimmer()
            infoL.Text = "Shimmer started"
        end
    end)
    end)()
    end
    _PG.updateShimmerVisual = updateShimmerVisual
    _PG.getPaintRF = getPaintRF; _PG.getAllBlocks = getAllBlocks; _PG.colMatch = colMatch
    _PG.unequipAndReturn = unequipAndReturn; _PG.unequipBuildTools = unequipBuildTools
    _PG.doPaint = doPaint; _PG.resetSelection = resetSelection
    end

    function _G.createPaintGUI_Part3(_PG)
    local sg = _PG.sg; local panel = _PG.panel; local pW = _PG.pW; local pH = _PG.pH
    local resetBtn = _PG.resetBtn; local closeBtn = _PG.closeBtn; local infoL = _PG.infoL
    local content = _PG.content; local subTabBar = _PG.subTabBar
    local subTabs = _PG.subTabs; local subPages = _PG.subPages; local activeSubTab = _PG.activeSubTab
    local pageSelect = _PG.pageSelect; local pagePaint = _PG.pagePaint; local pageShimmer = _PG.pageShimmer
    local leftCol = _PG.leftCol; local paintCol = _PG.paintCol; local shimmerCol = _PG.shimmerCol
    local staggerFadeIn = _PG.staggerFadeIn; local ptween = _PG.ptween
    local selColorPrev = _PG.selColorPrev; local selNameL = _PG.selNameL; local selInfoL = _PG.selInfoL
    local fromPicker = _PG.fromPicker; local toPicker = _PG.toPicker
    local updateShimmerVisual = _PG.updateShimmerVisual
    local getPaintRF = _PG.getPaintRF; local getAllBlocks = _PG.getAllBlocks; local colMatch = _PG.colMatch
    local unequipAndReturn = _PG.unequipAndReturn; local unequipBuildTools = _PG.unequipBuildTools
    local doPaint = _PG.doPaint; local resetSelection = _PG.resetSelection
    local findBlockFromPartFn = function(...) return _RG.findBlockFromPart and _RG.findBlockFromPart(...) or nil end

    local getFolder = _RG and _RG.getFolder
    local function buildPaintUIV2()

        if not getFolder then getFolder = function() return nil end end
        if not doPaint then doPaint = function() return false end end
        if not getPaintRF then getPaintRF = function() return nil, nil end end
        if not getAllBlocks then getAllBlocks = function() return {} end end
        if not colMatch then colMatch = function() return false end end
        if not unequipAndReturn then unequipAndReturn = function() end end
        if not unequipBuildTools then unequipBuildTools = function() end end
        if not resetSelection then resetSelection = function() end end
        if not stylizeCard then return end
        if not makeLabel then return end
        if not makeBtn then return end
        if not makeSlider then return end
        if not makeColorPicker then return end
        pW = math.floor(640 * (Settings.uiScale or 1))
        pH = math.floor(360 * (Settings.uiScale or 1))
        do
            local sw = tonumber(Settings.paintW)
            local sh = tonumber(Settings.paintH)
            if sw and sh and sw >= 340 and sh >= 420 then
                pW = sw; pH = sh
            end
        end
        panel.Size = UDim2.new(0, pW, 0, pH)
        do
            local ppx = tonumber(Settings.paintPosX)
            local ppy = tonumber(Settings.paintPosY)
            if ppx and ppy and ppx >= 0 and ppy >= 0 then
                panel.Position = UDim2.new(0, ppx, 0, ppy)
            else
                panel.Position = UDim2.new(0.5, -pW/2, 0.5, -pH/2)
            end
        end

        for _, child in ipairs(panel:GetChildren()) do
            if child:IsA("GuiObject") then
                pcall(function() child:Destroy() end)
            end
        end
        local v = {}
        local function mk(className, parent, props)
            local o = Instance.new(className)
            for k, val in pairs(props or {}) do o[k] = val end
            o.Parent = parent
            return o
        end
        local function box(parent, h, dark)
            local f = mk("Frame", parent, {Size = UDim2.new(1, -4, 0, h), BackgroundColor3 = dark and Color3.fromRGB(16,16,18) or Colors.PanelElevated, BackgroundTransparency = 0.08, BorderSizePixel = 0})
            stylizeCard(f, f.BackgroundColor3, Colors.Border, 5)
            return f
        end
        local function setText(obj, txt) if obj then obj.Text = txt end end
        v.root = mk("Frame", panel, {Name="PaintUIV2", Size=UDim2.new(1,0,1,0), BackgroundTransparency=1, Visible=true})
        v.head = mk("Frame", v.root, {Size=UDim2.new(1,0,0,44), BackgroundColor3=Colors.PanelElevated, BackgroundTransparency=0.02, BorderSizePixel=0})
        Instance.new("UICorner", v.head).CornerRadius = UDim.new(0,6)
        local hStroke = Instance.new("UIStroke"); hStroke.Color = Colors.Border; hStroke.Transparency = 0.88; hStroke.Thickness = 1; hStroke.Parent = v.head
        local hGrad = Instance.new("UIGradient"); hGrad.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Settings.primaryColor), ColorSequenceKeypoint.new(1, Settings.secondaryColor)}); hGrad.Rotation = 90; hGrad.Parent = v.head
        mk("TextLabel", v.head, {Size=UDim2.new(1,-124,1,0), Position=UDim2.new(0,16,0,0), BackgroundTransparency=1, Text="PAINT+", TextColor3=Colors.Text, TextSize=24, Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Left})
        infoL = mk("TextLabel", v.head, {Size=UDim2.new(0,200,1,0), Position=UDim2.new(1,-298,0,0), BackgroundTransparency=1, Text="Ready", TextColor3=Colors.Muted, TextSize=12, Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Right, TextTruncate=Enum.TextTruncate.AtEnd})
        local function headBtn(txt, x)
            local b = mk("TextButton", v.head, {Size=UDim2.new(0,36,0,36), Position=UDim2.new(1,x,0.5,-18), BackgroundColor3=Colors.PanelElevated, BorderSizePixel=0, Text=txt, TextColor3=Colors.Text, TextSize=15, Font=Enum.Font.GothamBold, AutoButtonColor=false})
            stylizeCard(b, Colors.PanelElevated, Colors.Border, 5)
            return b
        end
        resetBtn = headBtn("R", -84)
        closeBtn = headBtn("X", -42)
        v.body = mk("ScrollingFrame", v.root, {Size=UDim2.new(1,-24,1,-72), Position=UDim2.new(0,12,0,56), BackgroundTransparency=1, BorderSizePixel=0, ScrollBarThickness=4, ScrollBarImageColor3=Colors.Muted, CanvasSize=UDim2.new(0,0,0,0), AutomaticCanvasSize=Enum.AutomaticSize.Y, ElasticBehavior=Enum.ElasticBehavior.Never})
        local bodyLayout = Instance.new("UIListLayout", v.body)
        bodyLayout.Padding = UDim.new(0, 6)
        bodyLayout.SortOrder = Enum.SortOrder.LayoutOrder


        v.sel = box(v.body, 58, false)
        v.sel.LayoutOrder = 0
        selColorPrev = mk("Frame", v.sel, {Size=UDim2.new(0,42,0,42), Position=UDim2.new(0,10,0.5,-21), BackgroundColor3=Color3.fromRGB(128,128,128), BorderSizePixel=0})
        Instance.new("UICorner", selColorPrev).CornerRadius = UDim.new(0,5)
        selNameL = mk("TextLabel", v.sel, {Size=UDim2.new(1,-66,0,24), Position=UDim2.new(0,62,0,7), BackgroundTransparency=1, Text="None", TextColor3=Colors.Text, TextSize=16, Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Left, TextTruncate=Enum.TextTruncate.AtEnd})
        selInfoL = mk("TextLabel", v.sel, {Size=UDim2.new(1,-66,0,18), Position=UDim2.new(0,62,0,32), BackgroundTransparency=1, Text="Click block with Paint+ equipped", TextColor3=Colors.Muted, TextSize=11, Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Left, TextTruncate=Enum.TextTruncate.AtEnd})


        makeLabel("COLORS", v.body).LayoutOrder = 1

        fromPicker = makeColorPicker("FROM COLOR", Color3.fromRGB(255,60,60), v.body, function(c) end)
        fromPicker.container.LayoutOrder = 2
        toPicker = makeColorPicker("TO COLOR", Color3.fromRGB(60,200,80), v.body, function(c) end)
        toPicker.container.LayoutOrder = 3

        makeSlider("PaintTransp", 0, 100, math.floor((P.transpValue or 0) * 100 + 0.5), v.body, "TRANSPARENCY", function(v) return math.floor(v + 0.5) .. "%" end, function(v) P.transpValue = v / 100 end).LayoutOrder = 4

        local function targets(kind)
            local f = getFolder()
            if not f then setText(infoL, "No folder"); return nil end
            if kind == "all" then return getAllBlocks(f) end
            if not P.selectedName then setText(infoL, "Select a block"); return nil end
            local arr = {}
            for _, b in ipairs(getAllBlocks(f)) do
                if kind == "mat" and b.Name == P.selectedName then arr[#arr+1] = b end
                if kind == "swap" and b.Name == P.selectedName and b:FindFirstChild("PPart") and colMatch(b.PPart.Color, fromPicker.getColor()) then arr[#arr+1] = b end
            end
            return arr
        end
        local function paintList(kind)
            if P.isRunning then return end
            local list = kind == "one" and {P.selectedBlock} or targets(kind)
            if not list or #list == 0 then setText(infoL, "No blocks"); return end
            P.isRunning = true; unequipAndReturn()
            task.spawn(function()
                if doPaint(list, toPicker.getColor(), true) then setText(infoL, "Painted " .. #list) end
                P.isRunning = false; unequipBuildTools()
            end)
        end


        makeLabel("PAINT ACTIONS", v.body).LayoutOrder = 5
        makeBtn("PaintSelectedBtn", "PAINT SELECTED", v.body, function() if P.selectedBlock then paintList("one") else setText(infoL,"Select a block") end end).LayoutOrder = 6
        makeBtn("SameMatBtn", "SAME MATERIAL", v.body, function() paintList("mat") end).LayoutOrder = 7
        makeBtn("FromToBtn", "FROM -> TO", v.body, function() paintList("swap") end).LayoutOrder = 8
        makeBtn("PaintAllBtn", "PAINT ALL", v.body, function() paintList("all") end).LayoutOrder = 9
        makeBtn("RandPerBlockBtn", "RANDOM PER BLOCK", v.body, function()
            if P.isRunning then return end
            local folder = getFolder()
            if not folder then setText(infoL, "No folder"); return end
            local all = getAllBlocks(folder)
            if #all == 0 then setText(infoL, "No blocks"); return end
            P.isRunning = true; unequipAndReturn()
            task.spawn(function()
                local paintRF = getPaintRF()
                if not paintRF then setText(infoL, "Need Paint!"); P.isRunning = false; return end
                setText(infoL, "Randomizing " .. #all .. "...")
                local batch = {}
                for _, b in ipairs(all) do
                    batch[#batch+1] = {b, Color3.fromHSV(math.random(), 0.7 + math.random() * 0.3, 0.6 + math.random() * 0.4)}
                end
                for bi = 1, #batch, 200 do
                    local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end
                    pcall(function() paintRF:InvokeServer(chunk) end); task.wait()
                end
                setText(infoL, "Done: " .. #all .. " randomized"); P.isRunning = false; unequipBuildTools()
            end)
        end).LayoutOrder = 10
        makeBtn("RandPerMatBtn", "RANDOM PER MAT", v.body, function()
            if P.isRunning then return end
            local folder = getFolder()
            if not folder then setText(infoL, "No folder"); return end
            local all = getAllBlocks(folder)
            if #all == 0 then setText(infoL, "No blocks"); return end
            local matC = {}
            for _, b in ipairs(all) do
                if not matC[b.Name] then
                    matC[b.Name] = Color3.fromHSV(math.random(), 0.7 + math.random() * 0.3, 0.6 + math.random() * 0.4)
                end
            end
            P.isRunning = true; unequipAndReturn()
            task.spawn(function()
                local paintRF = getPaintRF()
                if not paintRF then setText(infoL, "Need Paint!"); P.isRunning = false; return end
                setText(infoL, "Coloring by mat...")
                local batch = {}; for _, b in ipairs(all) do batch[#batch+1] = {b, matC[b.Name]} end
                for bi = 1, #batch, 200 do
                    local chunk = {}; for j = bi, math.min(bi+199, #batch) do chunk[#chunk+1] = batch[j] end
                    pcall(function() paintRF:InvokeServer(chunk) end); task.wait()
                end
                setText(infoL, "Done: " .. #all .. " painted"); P.isRunning = false; unequipBuildTools()
            end)
        end).LayoutOrder = 11


        makeLabel("COLOR TOOLS", v.body).LayoutOrder = 12
        makeBtn("SwapColorsBtn", "SWAP COLORS", v.body, function() local a,b=fromPicker.getColor(),toPicker.getColor(); fromPicker.setColor(b); toPicker.setColor(a); setText(infoL,"Colors swapped") end).LayoutOrder = 13
        makeBtn("CopyColorBtn", "TO = FROM", v.body, function() toPicker.setColor(fromPicker.getColor()); setText(infoL,"Copied") end).LayoutOrder = 14


        makeLabel("PROPERTIES", v.body).LayoutOrder = 15
        makeBtn("ApplyTranspBtn", "APPLY TRANSPARENCY", v.body, function()
            if P.isRunning then return end
            local list = targets("all")
            if not list or #list == 0 then setText(infoL,"No blocks"); return end
            P.isRunning = true; unequipAndReturn()
            task.spawn(function()
                local _, rf = getPaintRF()
                if not rf then setText(infoL,"Need Properties"); P.isRunning=false; return end
                local ts = tostring(math.floor(P.transpValue*100+0.5))
                for i=1,#list,50 do local ch={}; for j=i,math.min(i+49,#list) do ch[#ch+1]=list[j] end; pcall(function() rf:InvokeServer("Transparency", ch, ts) end); task.wait() end
                setText(infoL,"Transparency set"); P.isRunning=false; unequipBuildTools()
            end)
        end).LayoutOrder = 16

        makeLabel("SHIMMER", v.body).LayoutOrder = 17
        makeBtn("ShimmerBtn", "SHIMMER", v.body, function()
            P.shimmerActive = not P.shimmerActive
            setText(infoL, P.shimmerActive and "Shimmer on" or "Shimmer off")
            if not P.shimmerActive then return end
            task.spawn(function()
                while P.shimmerActive and sg and sg.Parent do
                    local f, rf = getFolder(), getPaintRF()
                    if f and rf then
                        local all, batch = getAllBlocks(f), {}
                        for i,b in ipairs(all) do batch[#batch+1] = {b, Color3.fromHSV(((P.shimmerHue+i*360/math.max(#all,1))%360)/360, 0.9, 1)} end
                        for i=1,#batch,180 do if not P.shimmerActive then break end; local ch={}; for j=i,math.min(i+179,#batch) do ch[#ch+1]=batch[j] end; pcall(function() rf:InvokeServer(ch) end); task.wait() end
                    end
                    P.shimmerHue = (P.shimmerHue + 8) % 360
                    task.wait(0.2)
                end
            end)
        end).LayoutOrder = 18

        resetBtn.MouseButton1Click:Connect(function() resetSelection(); setText(infoL, "Selection cleared") end)


        local drag = {on=false}
        v.head.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                drag.on=true; drag.p=input.Position
                local abs = panel.AbsolutePosition
                drag.sx=abs.X; drag.sy=abs.Y
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if drag.on and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - drag.p
                local newX = drag.sx + d.X
                local newY = drag.sy + d.Y
                local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280,720)
                newX = math.clamp(newX, 4, math.max(4, vp.X - panel.AbsoluteSize.X - 4))
                newY = math.clamp(newY, 4, math.max(4, vp.Y - panel.AbsoluteSize.Y - 4))
                panel.Position = UDim2.new(0, newX, 0, newY)
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if drag.on and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
                drag.on = false
                Settings.paintPosX = math.floor(panel.AbsolutePosition.X + 0.5)
                Settings.paintPosY = math.floor(panel.AbsolutePosition.Y + 0.5)
                Settings.paintW = math.floor(panel.AbsoluteSize.X + 0.5)
                Settings.paintH = math.floor(panel.AbsoluteSize.Y + 0.5)
                saveSettings()
            end
        end)
    end
    buildPaintUIV2()

    P.showPaintGUI = function(animated)
        if sg.Enabled and panel.Visible then return end

        do
            local sw = tonumber(Settings.paintW)
            local sh = tonumber(Settings.paintH)
            if sw and sh and sw >= 340 and sh >= 420 then
                pW = sw; pH = sh
            end
        end
        sg.Enabled = true
        panel.Visible = true

        local ppx = tonumber(Settings.paintPosX)
        local ppy = tonumber(Settings.paintPosY)
        if ppx and ppy and ppx >= 0 and ppy >= 0 then
            panel.Position = UDim2.new(0, ppx, 0, ppy)
        else
            local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
            panel.Position = UDim2.new(0, (vp.X - pW) / 2, 0, (vp.Y - pH) / 2)
        end
        if not animated then
            panel.Size = UDim2.new(0, pW, 0, pH)
            panel.BackgroundTransparency = Settings.guiTransparency or 0.15
            return
        end
        playUISound(UISoundConfig.open)
        local curPos = panel.Position
        local startW = math.max(140, math.floor(pW * 0.32))
        local startH = 44
        local startOffX = curPos.X.Offset + (pW - startW) / 2
        local startOffY = curPos.Y.Offset + (pH - startH) / 2
        panel.Size = UDim2.new(0, startW, 0, startH)
        panel.Position = UDim2.new(0, startOffX, 0, startOffY)
        panel.BackgroundTransparency = 1
        ptween(panel, TweenInfo.new(0.32, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, pW, 0, pH),
            Position = UDim2.new(0, curPos.X.Offset, 0, curPos.Y.Offset),
            BackgroundTransparency = Settings.guiTransparency or 0.15
        })
        local boot = Instance.new("Frame")
        boot.Size = UDim2.new(1, 0, 1, 0); boot.BackgroundColor3 = Colors.BG; boot.BorderSizePixel = 0; boot.ZIndex = 40; boot.Parent = panel
        ptween(boot, TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
        task.delay(0.24, function() if boot and boot.Parent then boot:Destroy() end end)
        task.delay(0.3, function()
            staggerFadeIn(pageSelect, 0.04)
        end)
    end

    P.hidePaintGUI = function(animated)
        if not sg.Enabled then return end

        Settings.paintPosX = math.floor(panel.AbsolutePosition.X + 0.5)
        Settings.paintPosY = math.floor(panel.AbsolutePosition.Y + 0.5)
        Settings.paintW = math.floor(panel.AbsoluteSize.X + 0.5)
        Settings.paintH = math.floor(panel.AbsoluteSize.Y + 0.5)
        saveSettings()
        if not animated then
            sg.Enabled = false
            panel.Size = UDim2.new(0, pW, 0, pH)
            panel.BackgroundTransparency = Settings.guiTransparency or 0.15
            return
        end
        playUISound(UISoundConfig.close)
        local curPos = panel.Position
        local endW, endH = 44, 44
        local endOffX = curPos.X.Offset + (pW - endW) / 2
        local endOffY = curPos.Y.Offset + (pH - endH) / 2
        ptween(panel, TweenInfo.new(0.24, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
            Size = UDim2.new(0, endW, 0, endH),
            Position = UDim2.new(0, endOffX, 0, endOffY),
            BackgroundTransparency = 1
        })
        task.wait(0.25)
        sg.Enabled = false
        panel.Size = UDim2.new(0, pW, 0, pH)
        panel.Position = curPos
        panel.BackgroundTransparency = Settings.guiTransparency or 0.15
    end

    _openPaintGUI = function() P.showPaintGUI(true) end

    closeBtn.MouseButton1Click:Connect(function()
        task.spawn(function()
            P.hidePaintGUI(true)
            pcall(function()
                local ch = Character or LocalPlayer.Character
                local t = ch and ch:FindFirstChild("PaintToolExtended")
                if t then t.Parent = LocalPlayer.Backpack end
            end)
        end)
    end)

    task.spawn(function()
    local pt = Instance.new("Tool")
    pt.Name = "PaintToolExtended"; pt.RequiresHandle = false; pt.CanBeDropped = false
    local ptt = Instance.new("StringValue"); ptt.Name = "Tooltip"; ptt.Value = "Paint+ - Click block to select"; ptt.Parent = pt
    table.insert(customTools, pt)
    pt.Equipped:Connect(function()
        task.defer(function()
            if pt.Parent == Character then P.showPaintGUI(not P.isRunning) end
        end)
    end)
    pt.Unequipped:Connect(function()
        task.defer(function()
            if pt.Parent ~= Character then P.hidePaintGUI(not P.isRunning) end
        end)
    end)
    pt.AncestryChanged:Connect(function(_, parent)
        if not parent then for i, v in ipairs(customTools) do if v == pt then table.remove(customTools, i) break end end end
    end)
    P.selectPaintTarget = function()
        if P.isRunning then return end
        local mouse = LocalPlayer:GetMouse()
        local target = mouse.Target
        if not target then infoL.Text = "Click on a block!"; return end

        local findFn = findBlockFromPartFn or (_RG and _RG.findBlockFromPart)
        if not findFn then
            infoL.Text = "Block finder not ready"
            return
        end
        local block = findFn(target)
        if not block then infoL.Text = "Not a block: " .. tostring(target); return end
        local pp = block:FindFirstChild("PPart")
        if not pp then return end
        P.selectedBlock = block; P.selectedName = block.Name; P.selectedColor = pp.Color
        selNameL.Text = block.Name
        local cr, cg, cb = math.floor(pp.Color.R*255), math.floor(pp.Color.G*255), math.floor(pp.Color.B*255)
        selInfoL.Text = string.format("RGB: %d, %d, %d  |  T: %.2f", cr, cg, cb, pp.Transparency)
        if ptween and selColorPrev then ptween(selColorPrev, TweenInfo.new(0.2), {BackgroundColor3 = pp.Color}) end
        if fromPicker and fromPicker.setColor then fromPicker.setColor(pp.Color) end
        infoL.Text = "Selected: " .. block.Name
    end
    local _pSelBusy = false
    local function safeSelectPaint()
        if _pSelBusy then return end
        _pSelBusy = true
        P.selectPaintTarget()
        task.delay(0.15, function() _pSelBusy = false end)
    end
    pt.Activated:Connect(safeSelectPaint)
    LocalPlayer:GetMouse().Button1Down:Connect(function()
        if pt.Parent == Character then safeSelectPaint() end
    end)

    local pResizeGrip = Instance.new("Frame")
    pResizeGrip.Name = "ResizeGrip"; pResizeGrip.Size = UDim2.new(0, 22, 0, 22)
    pResizeGrip.Position = UDim2.new(1, -22, 1, -22); pResizeGrip.BackgroundTransparency = 1
    pResizeGrip.ZIndex = 50; pResizeGrip.Parent = panel
    local pGripIcon = Instance.new("TextLabel"); pGripIcon.Size = UDim2.new(1, 0, 1, 0)
    pGripIcon.BackgroundTransparency = 1; pGripIcon.Text = "\xe2\x97\xa3"
    pGripIcon.TextColor3 = Colors.Muted; pGripIcon.TextSize = 14; pGripIcon.ZIndex = 51
    pGripIcon.TextXAlignment = Enum.TextXAlignment.Right; pGripIcon.TextYAlignment = Enum.TextYAlignment.Bottom
    pGripIcon.Parent = pResizeGrip
    local pRsActive = false
    local pRsStart, pRsPanelSz
    pResizeGrip.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            pRsActive = true; pRsStart = i.Position; pRsPanelSz = panel.AbsoluteSize
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if pRsActive and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - pRsStart
            local nw = math.max(360, pRsPanelSz.X + d.X)
            local nh = math.max(280, pRsPanelSz.Y + d.Y)
            panel.Size = UDim2.new(0, nw, 0, nh)
        end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if pRsActive and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then pRsActive = false end
    end)
    end)
    end

    function _G.createPaintGUIContent()
        _G.createPaintGUI_Part1(_PG)
        _G.createPaintGUI_Part2(_PG)
        _G.createPaintGUI_Part3(_PG)
    end
    if not _G._afterCreateUI then _G._afterCreateUI = {} end
    table.insert(_G._afterCreateUI, function()
        _G.createPaintGUIContent()
    end)

    task.wait()
    function createToolsContent()
    makeLabel("TOOLS", miscFr)
    makeBtn("GiveReplaceTool", "Give Change", miscFr, function()
        local rt = LocalPlayer.Backpack:FindFirstChild("ReplaceSelect")
        if not rt then pcall(function()
            for _, ct in ipairs(customTools) do
                if ct.Name == "ReplaceSelect" then
                    pcall(function() ct.Parent = LocalPlayer.Backpack end)
                    break
                end
            end
        end) end
        local rt2 = LocalPlayer.Backpack:FindFirstChild("ReplaceSelect")
        if rt2 then pcall(function() local ch = Character or LocalPlayer.Character; local hum = ch and ch:FindFirstChildOfClass("Humanoid") or Humanoid; if hum then hum:EquipTool(rt2) end end) end
        task.wait(0.06)
        pcall(function()
            local g = LocalPlayer.PlayerGui:FindFirstChild("SPRB_ReplaceGUI")
            if g and not g.Enabled and _openReplaceGUI then _openReplaceGUI() end
        end)
    end)
    makeBtn("GivePaintTool", "Give Paint+", miscFr, function()
        local pt = LocalPlayer.Backpack:FindFirstChild("PaintToolExtended")
        if not pt then pcall(function()
            for _, ct in ipairs(customTools) do
                if ct.Name == "PaintToolExtended" then
                    pcall(function() ct.Parent = LocalPlayer.Backpack end)
                    break
                end
            end
        end) end
        local pt2 = LocalPlayer.Backpack:FindFirstChild("PaintToolExtended")
        if pt2 then pcall(function() local ch = Character or LocalPlayer.Character; local hum = ch and ch:FindFirstChildOfClass("Humanoid") or Humanoid; if hum then hum:EquipTool(pt2) end end) end
        task.wait(0.06)
        pcall(function()
            local g = LocalPlayer.PlayerGui:FindFirstChild("SPRB_PaintGUI")
            if g and not g.Enabled and _openPaintGUI then _openPaintGUI() end
        end)
    end)

    pcall(function()
        local mt = getrawmetatable(game)
        if mt and setreadonly then
            local oldNC = mt.__namecall
            setreadonly(mt, false)
            mt.__namecall = newcclosure(function(self, ...)
                local m = getnamecallmethod()
                if m == "Kick" or m == "kick" then cleanupTools() end
                return oldNC(self, ...)
            end)
            setreadonly(mt, true)
        end
    end)
    makeLabel("SHOP", miscFr)
    makeBtn("DragonH", "Dragon Harpoon", miscFr, function() workspace.PromptRobuxEvent:InvokeServer(1109792341,"Product") end)
    makeBtn("CookieW", "Cookie Wheels", miscFr, function() workspace.PromptRobuxEvent:InvokeServer(1126385328,"Product") end)
    makeBtn("MegaT", "Orange Mega Turbines", miscFr, function() workspace.PromptRobuxEvent:InvokeServer(139121474,"Product") end)
    makeBtn("PineT", "Buy Pine Tree", miscFr, function() workspace.ItemBoughtFromShop:InvokeServer("PineTree",1) end)

    makeLabel("TELEPORTS", miscFr)
    makeBtn("EasterTP", "Easter Event Place", miscFr, function() game:GetService("TeleportService"):Teleport(1930863474) end)
    makeBtn("ChristmasTP", "Christmas Event Place", miscFr, function() game:GetService("TeleportService"):Teleport(1930866268) end)
    makeBtn("TestTP", "Test Place", miscFr, function() game:GetService("TeleportService"):Teleport(1930665568) end)

    end
    createToolsContent()
    function createStealerContent()
    local stealerDropOpen = false
    stealerDropFrame = nil

    makeLabel("SCRIPT STEALER", miscFr)

    local stealerToggleBtn = Instance.new("TextButton")
    stealerToggleBtn.Name = "StealerToggleBtn"
    stealerToggleBtn.Size = UDim2.new(1, 0, 0, 30)
    stealerToggleBtn.BackgroundColor3 = Colors.PanelElevated
    stealerToggleBtn.BackgroundTransparency = 0
    stealerToggleBtn.BorderSizePixel = 0
    stealerToggleBtn.Text = "Script Saver"
    stealerToggleBtn.TextColor3 = Colors.Muted
    stealerToggleBtn.TextSize = 12
    stealerToggleBtn.Font = Enum.Font.GothamBold
    stealerToggleBtn.AutoButtonColor = false
    stealerToggleBtn.Parent = miscFr
    local _, stBtnStroke = stylizeCard(stealerToggleBtn, Colors.PanelElevated, Colors.Border, 3)
    stBtnStroke.Transparency = 0.92
    local stBtnScale = Instance.new("UIScale"); stBtnScale.Scale = 1; stBtnScale.Parent = stealerToggleBtn

    stealerDropFrame = Instance.new("Frame")
    stealerDropFrame.Name = "StealerDropPanel"
    stealerDropFrame.Size = UDim2.new(1, 0, 0, 0)
    stealerDropFrame.BackgroundColor3 = Colors.PanelSoft
    stealerDropFrame.BackgroundTransparency = 0.04
    stealerDropFrame.BorderSizePixel = 0
    stealerDropFrame.ClipsDescendants = true
    stealerDropFrame.Visible = true
    stealerDropFrame.Parent = miscFr
    local sdCr = Instance.new("UICorner"); sdCr.CornerRadius = UDim.new(0, 4); sdCr.Parent = stealerDropFrame
    local sdSt = Instance.new("UIStroke"); sdSt.Color = Colors.Border; sdSt.Transparency = 0.7; sdSt.Thickness = 1; sdSt.Parent = stealerDropFrame

    local sdContent = Instance.new("Frame")
    sdContent.Size = UDim2.new(1, -8, 1, -4); sdContent.Position = UDim2.new(0, 4, 0, 2)
    sdContent.BackgroundTransparency = 1; sdContent.Parent = stealerDropFrame
    local sdLay = Instance.new("UIListLayout"); sdLay.Padding = UDim.new(0, 4); sdLay.SortOrder = Enum.SortOrder.LayoutOrder; sdLay.Parent = sdContent
    local sdPad = Instance.new("UIPadding"); sdPad.PaddingTop = UDim.new(0, 4); sdPad.PaddingBottom = UDim.new(0, 4); sdPad.PaddingLeft = UDim.new(0, 2); sdPad.PaddingRight = UDim.new(0, 2); sdPad.Parent = sdContent

    local betaIcon = Instance.new("TextLabel")
    betaIcon.Size = UDim2.new(1, 0, 0, 40); betaIcon.BackgroundTransparency = 1
    betaIcon.Text = "\239\154\160 BETA \226\150\184 COMING SOON"
    betaIcon.TextColor3 = Settings.primaryColor; betaIcon.TextSize = 18; betaIcon.Font = Enum.Font.GothamBold
    betaIcon.TextXAlignment = Enum.TextXAlignment.Center; betaIcon.LayoutOrder = 1; betaIcon.Parent = sdContent

    local betaDesc = Instance.new("TextLabel")
    betaDesc.Size = UDim2.new(1, 0, 0, 60); betaDesc.BackgroundTransparency = 1
    betaDesc.Text = "Script Saver — save and copy scripts.\nComing soon!"
    betaDesc.TextColor3 = Colors.Text; betaDesc.TextSize = 12; betaDesc.Font = Enum.Font.GothamMedium
    betaDesc.TextWrapped = true; betaDesc.TextXAlignment = Enum.TextXAlignment.Center; betaDesc.LayoutOrder = 2; betaDesc.Parent = sdContent

    local betaWarn = Instance.new("TextLabel")
    betaWarn.Size = UDim2.new(1, 0, 0, 30); betaWarn.BackgroundTransparency = 1
    betaWarn.Text = "This section is not functional yet."
    betaWarn.TextColor3 = Color3.fromRGB(220, 150, 50); betaWarn.TextSize = 11; betaWarn.Font = Enum.Font.GothamBold
    betaWarn.TextWrapped = true; betaWarn.TextXAlignment = Enum.TextXAlignment.Center; betaWarn.LayoutOrder = 3; betaWarn.Parent = sdContent

    local betaBorder = Instance.new("Frame")
    betaBorder.Size = UDim2.new(1, 0, 0, 1); betaBorder.BackgroundTransparency = 0
    betaBorder.BackgroundColor3 = Settings.primaryColor; betaBorder.BorderSizePixel = 0
    betaBorder.LayoutOrder = 4; betaBorder.Parent = sdContent

    local betaSub = Instance.new("TextLabel")
    betaSub.Size = UDim2.new(1, 0, 0, 20); betaSub.BackgroundTransparency = 1
    betaSub.Text = "v6 | Under Construction"
    betaSub.TextColor3 = Colors.Muted; betaSub.TextSize = 10; betaSub.Font = Enum.Font.Gotham
    betaSub.TextXAlignment = Enum.TextXAlignment.Center; betaSub.LayoutOrder = 5; betaSub.Parent = sdContent

    local DROP_FULL_H = 170

    stealerToggleBtn.MouseButton1Click:Connect(function()
        stealerDropOpen = not stealerDropOpen
        playUISound(UISoundConfig.click)
        tween(stBtnScale, TweenInfo.new(0.08), {Scale = 0.97}):Play()
        task.delay(0.08, function() tween(stBtnScale, TweenInfo.new(0.12, Enum.EasingStyle.Back), {Scale = 1}):Play() end)
        if stealerDropOpen then
            stealerToggleBtn.Text = "Script Saver [^]"
            tween(stealerDropFrame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                Size = UDim2.new(1, 0, 0, DROP_FULL_H)
            }):Play()
        else
            stealerToggleBtn.Text = "Script Saver"
            tween(stealerDropFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
                Size = UDim2.new(1, 0, 0, 0)
            }):Play()
        end
    end)
    end
    createStealerContent()

    function createDupeContent()
    makeLabel("DUPE BUILD", miscFr)
    local _dupeSaveCache = nil
    local _dupeSaveCacheTime = 0
    local function getDupeSaveList()
        if _dupeSaveCache and (tick() - _dupeSaveCacheTime) < 300 then
            return _dupeSaveCache
        end
        ensureFolder()
        local saves = {}
        local seen = {}
        local searchDirs = {FOLDER_PREFIX, "BABFT/", "BABFT/Build/", "Build/", "."}
        for _, dir in ipairs(searchDirs) do
            if isfolder(dir) then
                local ok, items = pcall(listfiles, dir)
                if ok and type(items) == "table" then
                    for _, fp in ipairs(items) do
                        local fps = tostring(fp)
                        local low = fps:lower()
                        if (low:match("%.build$") or low:match("%.json$")) and not seen[fps] then
                            seen[fps] = true
                            local name = fps:match("([^/\\]+)$") or fps
                            local parent = getParentDir(fp) or dir
                            local display = (parent == "." or parent == FOLDER_PATH) and name or (name .. "  [" .. parent .. "]")
                            saves[#saves + 1] = {name = fps, display = display}
                        end
                    end
                end
            end
        end
        table.sort(saves, function(a, b) return tostring(a.display):lower() < tostring(b.display):lower() end)
        if #saves == 0 then saves[1] = {display = "No saves found"} end
        _dupeSaveCache = saves
        _dupeSaveCacheTime = tick()
        return saves
    end
    local dupeSaveBtn, _ = makeDropdown("DupeSaveDD", getDupeSaveList, miscFr, function(sel) end)
    local dupeAmtIn = makeInput("DupeAmt", "Amount (1-999)", miscFr)
    dupeAmtIn.Text = "10"
    local dupeSlotIn = makeInput("DupeSlot", "Slot (1-99)", miscFr)
    dupeSlotIn.Text = "42"
    local dupeStatusLbl = makeLabel("Ready to dupe", miscFr)
    local dupeProgress = Instance.new("Frame")
    dupeProgress.Name = "DupeProgress"
    dupeProgress.Size = UDim2.new(1, -8, 0, 6)
    dupeProgress.BackgroundColor3 = Colors.PanelElevated
    dupeProgress.BorderSizePixel = 0
    dupeProgress.Parent = miscFr
    local dpc = Instance.new("UICorner") ; dpc.CornerRadius = UDim.new(0,3) ; dpc.Parent = dupeProgress
    local dpf = Instance.new("Frame")
    dpf.Name = "Fill"
    dpf.Size = UDim2.new(0, 0, 1, 0)
    dpf.BackgroundColor3 = Color3.fromRGB(80,200,80)
    dpf.BorderSizePixel = 0
    dpf.Parent = dupeProgress
    local dpfc = Instance.new("UICorner") ; dpfc.CornerRadius = UDim.new(0,3) ; dpfc.Parent = dpf
    makeBtn("DupeBtn", "Dupe Build", miscFr, function()
        local selSave = dupeSaveBtn.Text
        if not selSave or selSave == "Select..." or selSave == "No saves found" then
            setStatus("  Pick a save first"); return
        end
        local dupe = tonumber(dupeAmtIn.Text) or 10
        local slot = tonumber(dupeSlotIn.Text) or 42
        if dupe < 1 or dupe > 999 then setStatus("  Amount must be 1-999") ; return end
        if slot < 1 or slot > 99 then setStatus("  Slot must be 1-99") ; return end
        dupeStatusLbl.Text = "Duping 0/" .. dupe .. " (slot " .. slot .. ")..."
        dpf.Size = UDim2.new(0, 0, 1, 0)
        for i = 1, dupe do
            workspace.LoadBoatData:FireServer(slot, 0)
            task.wait(0.05)
            local pct = i / dupe
            dpf.Size = UDim2.new(pct, 0, 1, 0)
            dupeStatusLbl.Text = "Duping " .. i .. "/" .. dupe .. " (" .. math.floor(pct*100) .. "%)"
        end
        dpf.Size = UDim2.new(1, 0, 1, 0)
        dpf.BackgroundColor3 = Color3.fromRGB(80,200,80)
        dupeStatusLbl.Text = "Dupe done! " .. dupe .. "x loaded from slot " .. slot
        end)

    farmWin = nil

    end
    createDupeContent()
    function createFarmContent()
    do
        local ok, data = pcall(function() return HttpService:JSONDecode(readfile("SPRB_FarmSettings.json")) end)
        if ok and data then
            farmSettings.step = tonumber(data.step) or farmSettings.step
            farmSettings.tgToken = data.tgToken or farmSettings.tgToken
            farmSettings.tgChatID = data.tgChatID or farmSettings.tgChatID
            farmSettings.tgInterval = tonumber(data.tgInterval) or farmSettings.tgInterval
            farmSettings.renderEnabled = data.renderEnabled ~= nil and data.renderEnabled or farmSettings.renderEnabled
            farmSettings.autoHop = data.autoHop == true
            farmSettings.autoFarm = data.autoFarm == true
            farmSettings.autoFarmFile = data.autoFarmFile or farmSettings.autoFarmFile
            farmSettings.autoBuild = data.autoBuild == true
            farmSettings.autoBuildFile = data.autoBuildFile or farmSettings.autoBuildFile
        end
    end
    saveFarmSettings = function()
        pcall(function() writefile("SPRB_FarmSettings.json", HttpService:JSONEncode(farmSettings)) end)
    end

    makeBtn("AutoFarmOpenBtn", "Auto Farm", miscFr, function()
        local FARM_COLLAPSED_H = 76
        local FARM_EXPANDED_H = 248

        if farmWin then
            if farmWin.Visible then
                tween(farmWin, TweenInfo.new(0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
                    Size = UDim2.new(0, 0, 0, 0),
                    BackgroundTransparency = 1
                }):Play()
                task.delay(0.22, function()
                    farmWin.Visible = false
                end)
            else
                farmWin.Visible = true
                farmWin.Size = UDim2.new(0, 0, 0, 0)
                farmWin.BackgroundTransparency = 1
                tween(farmWin, TweenInfo.new(0.32, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                    Size = UDim2.new(0, 148, 0, FARM_COLLAPSED_H),
                    BackgroundTransparency = 0.12
                }):Play()
            end
            return
        end

        farmWin = Instance.new("Frame")
        farmWin.Name = "FarmWindow"
        farmWin.Size = UDim2.new(0, 0, 0, 0)
        farmWin.Position = UDim2.new(0, 16, 0.5, -FARM_COLLAPSED_H / 2)
        farmWin.BackgroundColor3 = Colors.BG
        farmWin.BackgroundTransparency = 1
        farmWin.BorderSizePixel = 0
        farmWin.Active = true
        farmWin.ClipsDescendants = true
        farmWin.Visible = true
        farmWin.Parent = ScreenGui
        local wCr = Instance.new("UICorner"); wCr.CornerRadius = UDim.new(0, 6); wCr.Parent = farmWin
        local wSt = Instance.new("UIStroke"); wSt.Color = Colors.Border; wSt.Transparency = 0.52; wSt.Thickness = 1; wSt.Parent = farmWin

        local fGlow = Instance.new("Frame")
        fGlow.Size = UDim2.new(1, -16, 0, 1.5)
        fGlow.Position = UDim2.new(0, 8, 0, 5)
        fGlow.BackgroundColor3 = Colors.ActiveBG
        fGlow.BorderSizePixel = 0
        fGlow.BackgroundTransparency = 1
        fGlow.ZIndex = 2
        fGlow.Parent = farmWin
        local gCr = Instance.new("UICorner"); gCr.CornerRadius = UDim.new(1, 0); gCr.Parent = fGlow

        local fBar = Instance.new("Frame")
        fBar.Name = "Bar"
        fBar.Size = UDim2.new(1, 0, 0, 24)
        fBar.BackgroundTransparency = 1
        fBar.BorderSizePixel = 0
        fBar.ZIndex = 3
        fBar.Parent = farmWin
        local fTtl = Instance.new("TextLabel")
        fTtl.Size = UDim2.new(1, -60, 1, 0); fTtl.Position = UDim2.new(0, 8, 0, 0)
        fTtl.BackgroundTransparency = 1; fTtl.Text = "Auto Farm"
        fTtl.TextColor3 = Colors.Text; fTtl.TextSize = 12; fTtl.Font = Enum.Font.GothamSemibold
        fTtl.TextXAlignment = Enum.TextXAlignment.Left; fTtl.ZIndex = 4; fTtl.Parent = fBar

        local fClBtn = Instance.new("TextButton")
        fClBtn.Size = UDim2.new(0, 18, 0, 18); fClBtn.Position = UDim2.new(1, -22, 0, 3)
        fClBtn.BackgroundTransparency = 1; fClBtn.Text = "x"
        fClBtn.TextColor3 = Colors.Muted; fClBtn.TextSize = 12; fClBtn.Font = Enum.Font.GothamBold
        fClBtn.ZIndex = 4; fClBtn.AutoButtonColor = false; fClBtn.Parent = fBar
        fClBtn.MouseEnter:Connect(function() fClBtn.TextColor3 = Colors.Text end)
        fClBtn.MouseLeave:Connect(function() fClBtn.TextColor3 = Colors.Muted end)
        fClBtn.MouseButton1Click:Connect(function()
            if farmState.active then return end
            tween(farmWin, TweenInfo.new(0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
                Size = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 1
            }):Play()
            task.delay(0.22, function() farmWin.Visible = false end)
        end)

        local fContent = Instance.new("Frame")
        fContent.Name = "CT"
        fContent.Size = UDim2.new(1, -8, 1, -26)
        fContent.Position = UDim2.new(0, 4, 0, 26)
        fContent.BackgroundTransparency = 1
        fContent.ClipsDescendants = true
        fContent.Parent = farmWin

        local tglBtn = Instance.new("TextButton")
        tglBtn.Size = UDim2.new(1, 0, 0, 26)
        tglBtn.BackgroundColor3 = Colors.PanelElevated
        tglBtn.BorderSizePixel = 0
        tglBtn.Text = "Start"; tglBtn.TextColor3 = Colors.Text
        tglBtn.TextSize = 12; tglBtn.Font = Enum.Font.GothamSemibold
        tglBtn.AutoButtonColor = false; tglBtn.ZIndex = 2
        tglBtn.Parent = fContent
        local tCr = Instance.new("UICorner"); tCr.CornerRadius = UDim.new(0, 4); tCr.Parent = tglBtn
        local tSt = Instance.new("UIStroke"); tSt.Color = Colors.Border; tSt.Transparency = 0.55; tSt.Thickness = 1; tSt.Parent = tglBtn

        tglBtn.MouseEnter:Connect(function()
            tween(tSt, TweenInfo.new(0.1), {Transparency = 0.3}):Play()
        end)
        tglBtn.MouseLeave:Connect(function()
            if not farmState.active then
                tween(tSt, TweenInfo.new(0.1), {Transparency = 0.55}):Play()
            end
        end)

        local stLbl = Instance.new("TextLabel")
        stLbl.Size = UDim2.new(1, 0, 0, 14); stLbl.Position = UDim2.new(0, 0, 0, 30)
        stLbl.BackgroundTransparency = 1; stLbl.Text = "Ready"
        stLbl.TextColor3 = Colors.Muted; stLbl.TextSize = 10; stLbl.Font = Enum.Font.GothamMedium
        stLbl.TextXAlignment = Enum.TextXAlignment.Left; stLbl.Parent = fContent

        local infoLbl = Instance.new("TextLabel")
        infoLbl.Size = UDim2.new(1, 0, 0, 14); infoLbl.Position = UDim2.new(0, 0, 0, 44)
        infoLbl.BackgroundTransparency = 1; infoLbl.Text = "0 runs | 0:00 | 0/hr"
        infoLbl.TextColor3 = Color3.fromRGB(100, 100, 100); infoLbl.TextSize = 10; infoLbl.Font = Enum.Font.GothamMedium
        infoLbl.TextXAlignment = Enum.TextXAlignment.Left; infoLbl.Parent = fContent

        local fDragging = false
        local fDragStart = Vector2.new(0, 0)
        local fDragAbsPos = Vector2.new(0, 0)
        fBar.InputBegan:Connect(function(inp)
            if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
            fDragging = true
            fDragStart = Vector2.new(inp.Position.X, inp.Position.Y)
            fDragAbsPos = farmWin.AbsolutePosition
        end)
        UserInputService.InputChanged:Connect(function(inp)
            if not fDragging then return end
            if inp.UserInputType ~= Enum.UserInputType.MouseMovement and inp.UserInputType ~= Enum.UserInputType.Touch then return end
            local nx = fDragAbsPos.X + (inp.Position.X - fDragStart.X)
            local ny = fDragAbsPos.Y + (inp.Position.Y - fDragStart.Y)
            farmWin.Position = UDim2.new(0, nx, 0, ny)
        end)
        UserInputService.InputEnded:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                fDragging = false
            end
        end)

        local function updLbls()
            local s = getFarmStats()
            infoLbl.Text = s.runs .. " runs | " .. s.time .. " | " .. s.rate .. "/hr"
        end

        local platform = nil
        local function createPlatform()
            if platform and platform.Parent then platform:Destroy() end
            platform = Instance.new("Part")
            platform.Name = "SPRB_FarmPlat"
            platform.Size = Vector3.new(8, 0.5, 8)
            platform.Anchored = true
            platform.CanCollide = true
            platform.Transparency = 0.8
            platform.Material = Enum.Material.ForceField
            platform.Color = Color3.fromRGB(160, 160, 160)
            platform.Parent = Workspace
        end
        local function removePlatform()
            if platform and platform.Parent then platform:Destroy(); platform = nil end
        end

        local tgActive = false
        local function tgSend(text)
            local token = farmSettings.tgToken or ""
            local chatId = farmSettings.tgChatID or ""
            if token == "" or chatId == "" then return false end
            local clean = string.gsub(text, " ", "%%20")
            clean = string.gsub(clean, "\n", "%%0A")
            local url = "https://api.telegram.org/bot" .. token .. "/sendMessage?chat_id=" .. chatId .. "&text=" .. clean
            return pcall(function() game:HttpPost(url, "") end)
        end

        local tgBtnRef = nil
        local function toggleTG()
            tgActive = not tgActive
            if tgBtnRef then
                tgBtnRef.Text = "Telegram: " .. (tgActive and "ON" or "OFF")
                tgBtnRef.BackgroundColor3 = tgActive and Color3.fromRGB(16,32,16) or Colors.PanelElevated
            end
            if tgActive then
                local token = farmSettings.tgToken or ""
                local chatId = farmSettings.tgChatID or ""
                if token == "" or chatId == "" then
                    tgActive = false
                    if tgBtnRef then
                        tgBtnRef.Text = "Telegram: OFF"
                        tgBtnRef.BackgroundColor3 = Colors.PanelElevated
                    end
                    return
                end
                local lastUpId = 0
                local initUrl = "https://api.telegram.org/bot" .. token .. "/getUpdates?limit=1&offset=-1"
                local ok, res = pcall(function() return game:HttpGet(initUrl) end)
                if ok and res then
                    local upId = res:match('"update_id":(%d+)')
                    if upId then lastUpId = tonumber(upId) end
                end
                task.spawn(function()
                    local autoInt = farmSettings.tgInterval or 0
                    local lastAuto = tick()
                    while tgActive do
                        local getUrl = "https://api.telegram.org/bot" .. token .. "/getUpdates?offset=" .. (lastUpId + 1) .. "&timeout=3"
                        local s2, resp = pcall(function() return game:HttpGet(getUrl) end)
                        if s2 and resp and resp ~= "" then
                            local upId = resp:match('"update_id":(%d+)')
                            if upId then
                                lastUpId = tonumber(upId)
                                local chatPat = '"id"%s*:%s*' .. tostring(chatId) .. '[^0-9]'
                                local hasChat = resp:find(chatPat) ~= nil
                                if hasChat and (resp:find('"/stats"') or resp:find('"/status"')) then
                                    tgSend("Heard!")
                                    task.wait(0.5)
                                    local st = getFarmStats()
                                    tgSend("Farm Status:\nActive: " .. tostring(st.active) .. "\nRuns: " .. st.runs .. "\nEarned: " .. st.earned .. "\nTime: " .. st.time .. "\nRate: " .. st.rate .. "/hr")
                                end
                            end
                        end
                        if autoInt > 0 and tick() - lastAuto >= autoInt * 60 then
                            local st = getFarmStats()
                            tgSend("Farm Update:\nRuns: " .. st.runs .. "\nEarned: " .. st.earned .. "\nTime: " .. st.time .. "\nRate: " .. st.rate .. "/hr")
                            lastAuto = tick()
                        end
                        task.wait(1.5)
                    end
                end)
            end
        end

        _toggleTGFarm = toggleTG
        if _farmTgBtnRef then
            tgBtnRef = _farmTgBtnRef
        end

        tglBtn.MouseButton1Click:Connect(function()
            farmState.active = not farmState.active
            if farmState.active then
                tglBtn.Text = "Stop"
                tween(tSt, TweenInfo.new(0.15), {Color = Colors.Text, Transparency = 0.25}):Play()
                tween(fGlow, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0.05}):Play()
                stLbl.Text = "Starting..."
                farmState.startTime = tick()
                task.spawn(function()
                    while farmState.active do updLbls(); task.wait(1) end
                end)
                task.spawn(function()
                    local claimRF = safeWaitChild(workspace, "ClaimRiverResultsGold", 5)
                    if not claimRF then
                        stLbl.Text = "Remote not found"
                        farmState.active = false
                        tglBtn.Text = "Start"
                        tween(tSt, TweenInfo.new(0.15), {Color = Colors.Border, Transparency = 0.55}):Play()
                        tween(fGlow, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
                        return
                    end
                    createPlatform()
                    local step = tonumber(farmSettings.step) or 2
                    step = math.max(0.5, step)
                    while farmState.active do
                        local beforeGold = getGold()
                        for i, pos in ipairs(farmState.positions) do
                            if not farmState.active then break end
                            local c = LocalPlayer.Character
                            local hrp = c and c:FindFirstChild("HumanoidRootPart")
                            if not hrp then
                                stLbl.Text = "No character"
                                task.wait(2)
                                break
                            end

                            if platform and platform.Parent then
                                platform.CFrame = CFrame.new(pos.X, pos.Y - 3.5, pos.Z)
                            end
                            hrp.CFrame = CFrame.new(pos)
                            stLbl.Text = i .. "/" .. #farmState.positions
                            task.wait(step)
                        end
                        if not farmState.active then break end
                        pcall(function() claimRF:FireServer() end)
                        task.wait(2)
                        local afterGold = getGold()
                        local earned = math.max(0, afterGold - beforeGold)
                        farmState.earned = farmState.earned + earned
                        farmState.runs = farmState.runs + 1
                        stLbl.Text = "Run " .. farmState.runs
                        updLbls()
                    end
                    removePlatform()
                    farmState.totalTime = farmState.totalTime + (tick() - farmState.startTime)
                    stLbl.Text = "Stopped | " .. farmState.runs .. " runs"
                    tglBtn.Text = "Start"
                    tween(tSt, TweenInfo.new(0.15), {Color = Colors.Border, Transparency = 0.55}):Play()
                    tween(fGlow, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
                    updLbls()
                end)
            else
                farmState.totalTime = farmState.totalTime + (tick() - farmState.startTime)
                removePlatform()
                stLbl.Text = "Stopped | " .. farmState.runs .. " runs"
                tglBtn.Text = "Start"
                tween(tSt, TweenInfo.new(0.15), {Color = Colors.Border, Transparency = 0.55}):Play()
                tween(fGlow, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
                updLbls()
            end
        end)

        tween(farmWin, TweenInfo.new(0.32, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 148, 0, FARM_COLLAPSED_H),
            BackgroundTransparency = 0.12
        }):Play()
        tween(fGlow, TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0.85}):Play()

        LocalPlayer.CharacterAdded:Connect(function(newChar)
            task.wait(1)
            if farmState.active then
                createPlatform()
            end
        end)
    end)

    end
    createFarmContent()
    if not _G._afterCreateUI then _G._afterCreateUI = {} end
    table.insert(_G._afterCreateUI, function()
        _G.createBhopContent()
    end)

    local _SG = {}
    function createShapeGUI_Part1A(_SG)
    makeLabel("SHAPE BUILDER", rainFr)
    local shapeBlockName = "WoodBlock"
    local function getBlockOpts()
        local opts = {}
        for _, bp in ipairs(BuildingParts:GetChildren()) do
            if bp.Name:sub(-5) == "Block" then
                table.insert(opts, bp.Name)
            end
        end
        table.sort(opts, function(a, b) return a:lower() < b:lower() end)
        return opts
    end
    local shapeBlockDD, _ = makeDropdown("ShapeBlockDD", getBlockOpts, rainFr, function(nm)
        shapeBlockName = nm
        _SG.shapeBlockName = nm
    end)
    shapeBlockDD.Text = shapeBlockName
    shapeBlockDD.TextColor3 = Colors.Text

    local shapeFrameNames = {"sphere", "donut", "cube", "pyramid", "cylinder", "cone", "floors", "gear"}
    local shapeFrames = {}

    local shapeTypes = {"sphere", "donut", "cube", "pyramid", "cylinder", "cone", "floors", "gear"}
    local shapeType = "sphere"
    local shapeFileInput = nil
    local sDD, _ = makeDropdown("ShapeTypeDD", function() return shapeTypes end, rainFr, function(nm)
        shapeType = nm
        _SG.shapeType = nm
        for _, sn in ipairs(shapeFrameNames) do
            local f = shapeFrames[sn]
            if f then f.Visible = (sn == nm) end
        end

        if shapeFileInput then shapeFileInput.Text = nm end
    end)
    sDD.Text = shapeType
    sDD.TextColor3 = Colors.Text

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getR = makeNumInput("Radius:", 10, 0.01, 999999, 1, fr)
        local getSeg = makeNumInput("Segments:", 12, 3, 9999, 1, fr)
        local getThick = makeNumInput("Thickness:", 0.2, 0.01, 999999, 0.1, fr)
        shapeFrames.sphere = fr
        shapeFrames._sph = {getR, getSeg, getThick}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getMR = makeNumInput("Major Radius:", 10, 0.01, 999999, 1, fr)
        local getmr = makeNumInput("Minor Radius:", 3, 0.01, 999999, 0.5, fr)
        local getMS = makeNumInput("Major Segs:", 24, 3, 9999, 2, fr)
        local getms = makeNumInput("Minor Segs:", 12, 3, 9999, 2, fr)
        local getThick = makeNumInput("Thickness:", 0.2, 0.01, 999999, 0.1, fr)
        shapeFrames.donut = fr
        shapeFrames._don = {getMR, getmr, getMS, getms, getThick}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getSz = makeNumInput("Size:", 5, 0.01, 999999, 1, fr)
        local getLy = makeNumInput("Layers:", 1, 1, 999999, 1, fr)
        local getBSz = makeNumInput("Block Size:", 4, 0.01, 999999, 1, fr)
        shapeFrames.cube = fr
        shapeFrames._cub = {getSz, getLy, getBSz}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getBase = makeNumInput("Base:", 8, 0.01, 999999, 1, fr)
        local getLy = makeNumInput("Layers:", 6, 1, 999999, 1, fr)
        local getBSz = makeNumInput("Block Size:", 4, 0.01, 999999, 1, fr)
        shapeFrames.pyramid = fr
        shapeFrames._pyr = {getBase, getLy, getBSz}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getR = makeNumInput("Radius:", 6, 0.01, 999999, 1, fr)
        local getH = makeNumInput("Height:", 10, 0.01, 999999, 1, fr)
        local getSeg = makeNumInput("Segments:", 12, 3, 9999, 1, fr)
        local getThick = makeNumInput("Thickness:", 0.2, 0.01, 999999, 0.1, fr)
        shapeFrames.cylinder = fr
        shapeFrames._cyl = {getR, getH, getSeg, getThick}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getSeg = makeNumInput("Segments:", 36, 3, 9999, 1, fr)
        local getThick = makeNumInput("Thickness:", 0.2, 0.01, 999999, 0.1, fr)
        local getSubDens = makeNumInput("Sub dens:", 1.2, 0.1, 10, 0.1, fr)
        local getTipReduc = makeNumInput("Tip reduc%:", 50, 0, 100, 1, fr)
        local getWOverlap = makeNumInput("W overlap:", 1.08, 1.0, 2.0, 0.01, fr)

        makeLabel("Cone Sections (d x h)", fr)

        local secContainer = Instance.new("Frame")
        secContainer.Size = UDim2.new(1, 0, 0, 0)
        secContainer.BackgroundTransparency = 1
        secContainer.AutomaticSize = Enum.AutomaticSize.Y
        secContainer.Parent = fr
        local secLay = Instance.new("UIListLayout")
        secLay.Padding = UDim.new(0, 2)
        secLay.SortOrder = Enum.SortOrder.LayoutOrder
        secLay.Parent = secContainer

        local coneSections = {}
        local function rebuildSections(count)
            for _, s in ipairs(coneSections) do
                if s.row and s.row.Parent then s.row:Destroy() end
            end
            for k in pairs(coneSections) do coneSections[k] = nil end
            local defaultsD = {20, 16, 10, 5, 0}
            local defaultsH = {4, 4, 4, 4, 4}
            for i = 1, count do
                local row = Instance.new("Frame")
                row.Size = UDim2.new(1, 0, 0, 22)
                row.BackgroundColor3 = Colors.PanelSoft
                row.BackgroundTransparency = 0.3
                row.BorderSizePixel = 0
                row.LayoutOrder = i
                row.Parent = secContainer
                local rC = Instance.new("UICorner"); rC.CornerRadius = UDim.new(0, 3); rC.Parent = row

                local numLbl = Instance.new("TextLabel")
                numLbl.Size = UDim2.new(0, 18, 0, 14)
                numLbl.Position = UDim2.new(0, 4, 0.5, -7)
                numLbl.BackgroundColor3 = Colors.PanelElevated
                numLbl.BorderSizePixel = 0
                numLbl.Text = tostring(i)
                numLbl.TextColor3 = Colors.Muted
                numLbl.TextSize = 9
                numLbl.Font = Enum.Font.GothamBold
                numLbl.Parent = row
                local nC = Instance.new("UICorner"); nC.CornerRadius = UDim.new(0, 3); nC.Parent = numLbl

                local dLbl = Instance.new("TextLabel")
                dLbl.Size = UDim2.new(0, 44, 1, 0)
                dLbl.Position = UDim2.new(0, 22, 0, 0)
                dLbl.BackgroundTransparency = 1
                dLbl.Text = "Diameter"
                dLbl.TextColor3 = Colors.Muted
                dLbl.TextSize = 9
                dLbl.Font = Enum.Font.GothamBold
                dLbl.Parent = row

                local dIn = Instance.new("TextBox")
                dIn.Size = UDim2.new(0, 44, 0, 16)
                dIn.Position = UDim2.new(0, 66, 0.5, -8)
                dIn.BackgroundColor3 = Colors.PanelElevated
                dIn.BorderSizePixel = 0
                dIn.Text = tostring(defaultsD[i] or math.max(0, 24 - (i-1)*6))
                dIn.TextColor3 = Colors.Text
                dIn.TextSize = 10
                dIn.Font = Enum.Font.GothamBold
                dIn.ClearTextOnFocus = false
                dIn.Parent = row
                local dC = Instance.new("UICorner"); dC.CornerRadius = UDim.new(0, 3); dC.Parent = dIn

                local hLbl = Instance.new("TextLabel")
                hLbl.Size = UDim2.new(0, 38, 1, 0)
                hLbl.Position = UDim2.new(0, 112, 0, 0)
                hLbl.BackgroundTransparency = 1
                hLbl.Text = "Height"
                hLbl.TextColor3 = Colors.Muted
                hLbl.TextSize = 9
                hLbl.Font = Enum.Font.GothamBold
                hLbl.Parent = row

                local hIn = Instance.new("TextBox")
                hIn.Size = UDim2.new(0, 44, 0, 16)
                hIn.Position = UDim2.new(0, 150, 0.5, -8)
                hIn.BackgroundColor3 = Colors.PanelElevated
                hIn.BorderSizePixel = 0
                hIn.Text = tostring(defaultsH[i] or 6)
                hIn.TextColor3 = Colors.Text
                hIn.TextSize = 10
                hIn.Font = Enum.Font.GothamBold
                hIn.ClearTextOnFocus = false
                hIn.Parent = row
                local hC = Instance.new("UICorner"); hC.CornerRadius = UDim.new(0, 3); hC.Parent = hIn

                local dVal = defaultsD[i] or math.max(0, 24 - (i-1)*6)
                local hVal = defaultsH[i] or 6
                dIn.FocusLost:Connect(function()
                    local n = tonumber(dIn.Text)
                    if n then dVal = math.max(0, n) end
                    dIn.Text = tostring(dVal)
                end)
                hIn.FocusLost:Connect(function()
                    local n = tonumber(hIn.Text)
                    if n then hVal = math.max(0.5, n) end
                    hIn.Text = tostring(hVal)
                end)

                coneSections[#coneSections+1] = {
                    row = row,
                    getD = function() return dVal end,
                    getH = function() return hVal end,
                }
            end
        end

        local getSecCount = makeNumInput("Sections:", 4, 1, 20, 1, fr, function(v)
            rebuildSections(math.floor(v))
        end)
        rebuildSections(4)

        shapeFrames.cone = fr
        shapeFrames._cone = {getSecCount, getSeg, getThick, coneSections, getSubDens, getTipReduc, getWOverlap}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getLy = makeNumInput("Layers:", 5, 1, 999999, 1, fr)
        local getW = makeNumInput("Width:", 20, 0.01, 999999, 5, fr)
        local getD = makeNumInput("Depth:", 20, 0.01, 999999, 5, fr)
        local getBH = makeNumInput("Floor H:", 4, 0.01, 999999, 1, fr)
        local getGap = makeNumInput("Gap:", 8, 0, 999999, 1, fr)
        shapeFrames.floors = fr
        shapeFrames._flr = {getLy, getW, getD, getBH, getGap}
    end

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.Parent = fr
        local getGR = makeNumInput("Radius:", 10, 0.05, 5000, 1, fr)
        local getGCube = makeNumInput("Cube Size:", 1, 0.1, 999999, 0.5, fr)
        local getGToothLen = makeNumInput("Tooth Len Mul:", 2, 0.1, 999999, 0.1, fr)
        local getGToothH = makeNumInput("Tooth H Mul:", 1.5, 0.1, 999999, 0.1, fr)
        shapeFrames.gear = fr
        shapeFrames._gear = {getGR, getGCube, getGToothLen, getGToothH}
    end

    for _, sn in ipairs(shapeFrameNames) do
        local f = shapeFrames[sn]
        if f then f.Visible = (sn == shapeType) end
    end

    local getHOff = makeNumInput("Height Offset:", 15, 0, 100, 1, rainFr)

    makeLabel("FILE NAME", rainFr)

    do
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(1, 0, 0, 0)
        fr.BackgroundTransparency = 1
        fr.AutomaticSize = Enum.AutomaticSize.Y
        fr.Parent = rainFr
        local lay = Instance.new("UIListLayout")
        lay.Parent = fr
        local getSolidW = makeNumInput("Width:", 8, 0.01, 999999, 1, fr)
        local getSolidH = makeNumInput("Height:", 3, 0.01, 999999, 1, fr)
        local getSolidD = makeNumInput("Depth:", 8, 0.01, 999999, 1, fr)
        local getSolidBSz = makeNumInput("Block Size:", 4, 0.01, 999999, 0.5, fr)
        local getSolidOverlap = makeNumInput("Overlap%:", 5, 0, 50, 1, fr)
        shapeFrames.solid = fr
        fr.Visible = false
        shapeFrames._sol = {getSolidW, getSolidH, getSolidD, getSolidBSz, getSolidOverlap}
    end

    shapeFileInput = makeInput("ShapeFileInput", shapeType, rainFr)
    shapeFileInput.Text = shapeType
    _SG.shapeBlockName = shapeBlockName; _SG.shapeFrames = shapeFrames
    _SG.shapeType = shapeType; _SG.shapeFileInput = shapeFileInput
    _SG.getHOff = getHOff
    end

    function createShapeGUI_Part1B(_SG)
    local shapeBlockName = _SG.shapeBlockName; local shapeFrames = _SG.shapeFrames
    local shapeType = _SG.shapeType; local shapeFileInput = _SG.shapeFileInput
    local getHOff = _SG.getHOff
    local function genSphere()
        local entries = {}
        local R = shapeFrames._sph[1]()
        local seg = math.floor(shapeFrames._sph[2]())
        local thick = shapeFrames._sph[3]()
        local latStep = math.pi / seg

        for i = 1, seg - 1 do
            local lat = i * latStep
            local sinLat = math.sin(lat)
            local cosLat = math.cos(lat)
            local Rring = R * sinLat


            local ringN = math.max(3, math.floor(seg * 2 * sinLat))
            local lonStep = 2 * math.pi / ringN
            local bandH = 2 * R * math.sin(latStep / 2) * 1.05
            local bandW = 2 * Rring * math.sin(lonStep / 2) * 1.05

            for j = 0, ringN - 1 do
                local lon = j * lonStep
                local pos = Vector3.new(Rring * math.cos(lon), R * cosLat, Rring * math.sin(lon))


                local cf = CFrame.lookAt(pos, Vector3.zero) * CFrame.Angles(0, math.pi, 0)

                table.insert(entries, {
                    CFrame = cfStr(cf),
                    Size = v3Str(Vector3.new(bandW, bandH, thick)),
                    Anchored = true,
                    CanCollide = false,
                    Transparency = 0,
                    ShowShadow = true,
                })
            end
        end


        local poleSize = v3Str(Vector3.new(R * latStep * 1.1, R * latStep * 1.1, thick))
        table.insert(entries, {
            CFrame = cfStr(CFrame.new(0, R, 0) * CFrame.Angles(math.pi/2, 0, 0)),
            Size = poleSize, Anchored = true, CanCollide = false, Transparency = 0, ShowShadow = true
        })
        table.insert(entries, {
            CFrame = cfStr(CFrame.new(0, -R, 0) * CFrame.Angles(-math.pi/2, 0, 0)),
            Size = poleSize, Anchored = true, CanCollide = false, Transparency = 0, ShowShadow = true
        })

        return entries
    end

    local function genDonut()
        local entries = {}
        local Rm = shapeFrames._don[1]()
        local rm = shapeFrames._don[2]()
        local Nm = math.floor(shapeFrames._don[3]())
        local nm = math.floor(shapeFrames._don[4]())
        local thick = shapeFrames._don[5]()
        local majorArc = 2 * Rm * math.sin(math.pi / Nm) * 1.08
        local minorArc = 2 * rm * math.sin(math.pi / nm) * 1.08
        for i = 1, Nm do
            local u = (i * 2 * math.pi) / Nm
            local cosU, sinU = math.cos(u), math.sin(u)
            local dirM = Vector3.new(cosU, 0, sinU)
            local tanM = Vector3.new(-sinU, 0, cosU)
            local center = dirM * Rm
            for j = 1, nm do
                local v = (j * 2 * math.pi) / nm
                local norm = dirM * math.cos(v) + Vector3.new(0, math.sin(v), 0)
                local pos = center + norm * rm
                local rightV = tanM:Cross(norm).Unit
                local cf = CFrame.fromMatrix(pos, rightV, tanM, norm)
                table.insert(entries, {
                    CFrame = cfStr(cf),
                    Size = v3Str(Vector3.new(minorArc, majorArc, thick)),
                    Anchored = true,
                    CanCollide = false,
                    Transparency = 0,
                    ShowShadow = true,
                })
            end
        end
        return entries
    end

    local function genCube()
        local entries = {}
        local sz = math.floor(shapeFrames._cub[1]())
        local layers = math.floor(shapeFrames._cub[2]())
        local bsz = shapeFrames._cub[3]()
        local half = (sz + 1) / 2
        for x = 1, sz do for y = 1, layers do for z = 1, sz do
            if x==1 or x==sz or y==1 or y==layers or z==1 or z==sz then
                local pos = Vector3.new((x - half) * bsz, (y - 1) * bsz, (z - half) * bsz)
                local cf = CFrame.new(pos)
                table.insert(entries, {
                    CFrame = cfStr(cf),
                    Size = v3Str(Vector3.new(bsz, bsz, bsz)),
                    Anchored = true,
                    CanCollide = true,
                    Transparency = 0,
                    ShowShadow = true,
                })
            end
        end end end
        return entries
    end

    local function genPyramid()
        local entries = {}
        local base = math.floor(shapeFrames._pyr[1]())
        local layers = math.floor(shapeFrames._pyr[2]())
        local bsz = shapeFrames._pyr[3]()
        for layer = 1, layers do
            local s = math.max(1, math.ceil(base * (1 - (layer-1)/layers)))
            local half = (s + 1) / 2
            for x = 1, s do for z = 1, s do
                if x==1 or x==s or z==1 or z==s or layer==layers then
                    local pos = Vector3.new((x - half) * bsz, (layer-1)*bsz, (z - half) * bsz)
                    table.insert(entries, {
                        CFrame = cfStr(CFrame.new(pos)),
                        Size = v3Str(Vector3.new(bsz, bsz, bsz)),
                        Anchored = true,
                        CanCollide = true,
                        Transparency = 0,
                        ShowShadow = true,
                    })
                end
            end end
        end
        return entries
    end

    local function genCylinder()
        local entries = {}
        local R = shapeFrames._cyl[1]()
        local H = shapeFrames._cyl[2]()
        local seg = math.floor(shapeFrames._cyl[3]())
        local thick = shapeFrames._cyl[4]()
        local stp = 2 * math.pi / seg
        local pw = 2 * R * math.sin(stp / 2) * 1.08
        local ph = H
        for j = 1, seg do
            local angle = (j * 2 * math.pi) / seg
            local x = R * math.cos(angle)
            local z = R * math.sin(angle)
            local pos = Vector3.new(x, 0, z)
            local norm = Vector3.new(math.cos(angle), 0, math.sin(angle))
            local up = Vector3.new(0, 1, 0)
            local right = up:Cross(norm).Unit
            local cf = CFrame.fromMatrix(pos, right, up, norm)
            table.insert(entries, {
                CFrame = cfStr(cf),
                Size = v3Str(Vector3.new(pw, ph, thick)),
                Anchored = true,
                CanCollide = false,
                Transparency = 0,
                ShowShadow = true,
            })
        end
        return entries
    end

    local function genCone()
        local entries = {}
        local sections = shapeFrames._cone[4]
        local globalSeg = math.floor(shapeFrames._cone[2]())
        local thick = shapeFrames._cone[3]()
        local subDens = shapeFrames._cone[5]()
        local tipReducPct = shapeFrames._cone[6]()
        local wOverlap = shapeFrames._cone[7]()
        local ringSettings = {}

        for s = 1, #sections do
            local rBot = sections[s].getD() / 2
            local rTop = (s < #sections) and (sections[s+1].getD() / 2) or 0
            local secH = sections[s].getH()
            ringSettings[#ringSettings+1] = {
                RadiusBottom = rBot,
                RadiusTop = rTop,
                Height = secH,
                Thickness = thick,
            }
        end

        local totalH = 0
        for _, ring in ipairs(ringSettings) do
            totalH = totalH + ring.Height
        end

        local currentYOffset = -totalH / 2

        for ringIndex, config in ipairs(ringSettings) do
            local rBottom = config.RadiusBottom
            local rTop = config.RadiusTop
            local sectionHeight = config.Height
            local thickness = config.Thickness

            if sectionHeight <= 0 then continue end
            local maxRadius = math.max(rBottom, rTop)
            if maxRadius < 0.01 then currentYOffset = currentYOffset + sectionHeight; continue end

            local numSubSteps = math.max(6, math.ceil(sectionHeight * subDens))
            local subH = sectionHeight / numSubSteps

            for step = 1, numSubSteps do
                local fracBot = (step - 1) / numSubSteps
                local fracTop = step / numSubSteps
                local subRBot = rBottom + (rTop - rBottom) * fracBot
                local subRTop = rBottom + (rTop - rBottom) * fracTop
                local subDeltaR = subRBot - subRTop
                local subSlantAngle = math.atan2(subDeltaR, subH)


                local subSlantLength = math.sqrt(subH^2 + subDeltaR^2) * 1.05
                local subAvgR = (subRBot + subRTop) / 2



                local ratio = subAvgR / maxRadius
                local exponent = tipReducPct / 100
                local currentSeg = math.max(3, math.floor(globalSeg * (ratio ^ exponent)))
                if subAvgR < 0.05 then continue end

                local partWidth = 2 * subAvgR * math.sin(math.pi / currentSeg) * wOverlap
                local subCenterY = currentYOffset + (step - 0.5) * subH

                for i = 1, currentSeg do
                    local angle = (i * 2 * math.pi) / currentSeg
                    local cosAngle = math.cos(angle)
                    local sinAngle = math.sin(angle)
                    local normal = Vector3.new(cosAngle, 0, sinAngle)
                    local upVector = Vector3.new(0, 1, 0)
                    local rightVector = upVector:Cross(normal).Unit
                    local partPos = Vector3.new(0, subCenterY, 0) + (normal * subAvgR)
                    local baseCFrame = CFrame.fromMatrix(partPos, rightVector, upVector, normal)
                    local cf = baseCFrame * CFrame.Angles(-subSlantAngle, 0, 0)

                    table.insert(entries, {
                        CFrame = cfStr(cf),
                        Size = v3Str(Vector3.new(partWidth, subSlantLength, thickness)),
                        Anchored = true,
                        CanCollide = false,
                        Transparency = 0,
                        ShowShadow = true,
                    })
                end
            end

            currentYOffset = currentYOffset + sectionHeight
        end
        return entries
    end

    local function genFloors()
        local entries = {}
        local nLy = math.floor(shapeFrames._flr[1]())
        local fW = shapeFrames._flr[2]()
        local fD = shapeFrames._flr[3]()
        local fH = shapeFrames._flr[4]()
        local fGap = shapeFrames._flr[5]()
        for layer = 1, nLy do
            local yy = (layer - 1) * (fH + fGap)
            local pos = Vector3.new(0, yy, 0)
            table.insert(entries, {
                CFrame = cfStr(CFrame.new(pos)),
                Size = v3Str(Vector3.new(fW, fH, fD)),
                Anchored = true,
                CanCollide = true,
                Transparency = 0,
                ShowShadow = true,
            })
        end
        return entries
    end

    local function genGear()
        local entries = {}
        local Radius = shapeFrames._gear[1]()
        local CubeSize = shapeFrames._gear[2]()
        local ToothLengthMultiplier = shapeFrames._gear[3]()
        local ToothHeightMultiplier = shapeFrames._gear[4]()


        table.insert(entries, {
            CFrame = cfStr(CFrame.new(0, 0, 0)),
            Size = v3Str(Vector3.new(CubeSize, CubeSize, CubeSize)),
            Anchored = true,
            CanCollide = false,
            Transparency = 0,
            ShowShadow = true,
        })

        local Circumference = 2 * math.pi * Radius
        local calculatedN = math.floor(Circumference / CubeSize)
        local N = calculatedN - (calculatedN % 2)
        if N < 4 then N = 4 end

        local PerfectRadius = CubeSize / (2 * math.sin(math.pi / N))
        for i = 1, N do
            local angle = (i * 2 * math.pi) / N
            local cosAngle = math.cos(angle)
            local sinAngle = math.sin(angle)
            local normal = Vector3.new(cosAngle, 0, sinAngle)
            local upVector = Vector3.new(0, 1, 0)
            local rightVector = normal:Cross(upVector).Unit
            local partPos = normal * PerfectRadius
            local entrySize
            if i % 2 == 0 then

                local toothLength = CubeSize * ToothLengthMultiplier
                local toothHeight = CubeSize * ToothHeightMultiplier
                local shift = (toothLength - CubeSize) / 2
                partPos = partPos + normal * shift
                entrySize = Vector3.new(CubeSize, toothHeight, toothLength)
            else

                entrySize = Vector3.new(CubeSize, CubeSize, CubeSize)
            end
            local cf = CFrame.fromMatrix(partPos, rightVector, upVector, normal)
            table.insert(entries, {
                CFrame = cfStr(cf),
                Size = v3Str(entrySize),
                Anchored = true,
                CanCollide = false,
                Transparency = 0,
                ShowShadow = true,
            })
        end
        return entries
    end

    local function genSolid()
        local entries = {}


        local W = shapeFrames._sol[1]()
        local H = shapeFrames._sol[2]()
        local D = shapeFrames._sol[3]()
        local bsz = shapeFrames._sol[4]()
        local overlapPct = shapeFrames._sol[5]() / 100
        local overlap = bsz * overlapPct
        local effBsz = bsz - overlap
        local wCount = math.ceil(W / effBsz)
        local hCount = math.ceil(H / effBsz)
        local dCount = math.ceil(D / effBsz)
        local halfW = (wCount - 1) * effBsz / 2
        local halfD = (dCount - 1) * effBsz / 2
        for ix = 1, wCount do
            for iy = 1, hCount do
                for iz = 1, dCount do

                    if ix == 1 or ix == wCount or iy == 1 or iy == hCount or iz == 1 or iz == dCount then
                        local px = (ix - 1) * effBsz - halfW
                        local py = (iy - 1) * effBsz
                        local pz = (iz - 1) * effBsz - halfD
                        local cf = CFrame.new(px, py, pz)
                        table.insert(entries, {
                            CFrame = cfStr(cf),
                            Size = v3Str(Vector3.new(bsz, bsz, bsz)),
                            Anchored = true,
                            CanCollide = true,
                            Transparency = 0,
                            ShowShadow = true,
                        })
                    end
                end
            end
        end
        return entries
    end

    local function genEntries()
        local st = _SG.shapeType or shapeType
        local entries
        if st == "sphere" then entries = genSphere()
        elseif st == "donut" then entries = genDonut()
        elseif st == "cube" then entries = genCube()
        elseif st == "pyramid" then entries = genPyramid()
        elseif st == "cylinder" then entries = genCylinder()
        elseif st == "cone" then entries = genCone()
        elseif st == "floors" then entries = genFloors()
        elseif st == "gear" then entries = genGear()
        elseif st == "solid" then entries = genSolid()
        end
        entries = entries or {}
        return entries
    end
    _SG.shapeBlockName = shapeBlockName; _SG.shapeFrames = shapeFrames
    _SG.shapeType = shapeType; _SG.shapeFileInput = shapeFileInput
    _SG.getHOff = getHOff; _SG.genEntries = genEntries
    end

    function createShapeGUI_Part1(_SG)
        createShapeGUI_Part1A(_SG)
        createShapeGUI_Part1B(_SG)
    end

    function createShapeGUI_Part2(_SG)
    local shapeFrames = _SG.shapeFrames
    local shapeType = _SG.shapeType; local shapeFileInput = _SG.shapeFileInput
    local getHOff = _SG.getHOff; local genEntries = _SG.genEntries
    task.wait()
    local shPrevFolder = Instance.new("Folder")
    shPrevFolder.Name = "ShapePreview"
    shPrevFolder.Parent = workspace
    local function clearShPrev()
        for _, p in pairs(shPrevFolder:GetChildren()) do p:Destroy() end
    end
    local shPrevActive = false

    local shPreviewBtn = makeBtn("ShPreviewBtn", "Preview", rainFr, function()
        if shPrevActive then
            clearShPrev()
            shPrevActive = false
            local _spBtn = shPreviewBtn or rainFr:FindFirstChild("ShPreviewBtn"); if _spBtn then _spBtn.Text = "Preview" end
            return
        end
        local curBlockName = _SG.shapeBlockName or shapeBlockName or "WoodBlock"
        local entries = genEntries()
        if #entries == 0 then setStatus("  Empty shape") return end
        local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
        local origin = hrp and (hrp.CFrame + Vector3.new(0, getHOff(), 0)) or CFrame.new(0, getHOff(), 0)
        local targetT = Settings.previewTransparency or 0.5
        local tmpl = BuildingParts:FindFirstChild(curBlockName)
        local allFadeParts = {}
        for _, ent in ipairs(entries) do
            local cf = strCF(ent.CFrame)
            local sz = ent.Size and strV3(ent.Size) or Vector3.new(4, 4, 4)
            local worldCF = origin * cf
            if tmpl and tmpl:FindFirstChild("PPart") then
                local pb = tmpl:Clone()
                if pb:FindFirstChild("PPart") then
                    local partOffsets = {}
                    for _, d in pairs(pb:GetDescendants()) do
                        if (d:IsA("BasePart") or d:IsA("UnionOperation")) and d ~= pb.PPart then
                            partOffsets[d] = pb.PPart.CFrame:ToObjectSpace(d.CFrame)
                        end
                    end
                    pb.PPart.CFrame = worldCF
                    for d, offsetCF in pairs(partOffsets) do
                        pcall(function() d.CFrame = worldCF * offsetCF end)
                    end
                    pcall(function() pb.PPart.Size = sz end)
                    pb.PPart.Transparency = 1
                    pb.PPart.CanCollide = false
                    pb.PPart.Anchored = true
                    allFadeParts[#allFadeParts+1] = pb.PPart
                    for _, d in pairs(pb:GetDescendants()) do
                        if d:IsA("BasePart") or d:IsA("UnionOperation") then
                            d.Transparency = 1
                            d.CanCollide = false
                            d.Anchored = true
                            allFadeParts[#allFadeParts+1] = d
                        end
                    end
                    pb.Name = curBlockName
                    pb.Parent = shPrevFolder
                end
            else

                local c = Instance.new("Part")
                c.Size = sz
                c.CFrame = worldCF
                c.Transparency = targetT
                c.Color = Color3.fromRGB(100, 200, 255)
                c.Anchored = true
                c.CanCollide = false
                c.Material = Enum.Material.ForceField
                c.Parent = shPrevFolder
            end
        end
        if #allFadeParts > 0 then
            task.spawn(function()
                local startT = tick()
                local dur = 0.45
                while true do
                    local el = tick() - startT
                    local a = math.clamp(el / dur, 0, 1)
                    local tVal = 1 + (targetT - 1) * a
                    for _, p in ipairs(allFadeParts) do
                        if p and p.Parent then p.Transparency = tVal end
                    end
                    if a >= 1 then break end
                    task.wait(0.03)
                end
            end)
        end
        shPrevActive = true
        local _spBtn = shPreviewBtn or rainFr:FindFirstChild("ShPreviewBtn"); if _spBtn then _spBtn.Text = "Clear Preview" end
        setStatus("  Preview: " .. #entries .. " blocks")
    end)

    makeBtn("ShSaveBtn", "SAVE File", rainFr, function()
        clearShPrev()
        local curBlockName = _SG.shapeBlockName or shapeBlockName or "WoodBlock"
        local entries = genEntries()
        if #entries == 0 then setStatus("  Empty (0 blocks!)"); return end
        local fileName = (shapeFileInput and shapeFileInput.Text ~= "" and shapeFileInput.Text:match("^%s*(.-)%s*$")) or "my_shape"
        if fileName == "" then fileName = "my_shape" end
        local myZone = getPlayerZone(LocalPlayer)
        local cx, cy, cz = 0, 0, 0
        if myZone and Character then
            local hrp = Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local rp = myZone.CFrame:ToObjectSpace(hrp.CFrame).Position
                cx, cy, cz = rp.X, rp.Y, rp.Z
            end
        end
        local hOff = getHOff()
        local blockList = {}
        for _, ent in ipairs(entries) do
            local baseCF = strCF(ent.CFrame)
            local newPos = baseCF.Position + Vector3.new(cx, cy + hOff, cz)
            local newCF = CFrame.new(newPos) * (baseCF - baseCF.Position)
            local e2 = {CFrame = cfStr(newCF), Anchored = true}
            if ent.Size then e2.Size = ent.Size end
            if ent.CanCollide ~= nil then e2.CanCollide = ent.CanCollide end
            if ent.Transparency ~= nil then e2.Transparency = ent.Transparency end
            if ent.ShowShadow ~= nil then e2.ShowShadow = ent.ShowShadow end
            table.insert(blockList, e2)
        end
        local buildData = {[curBlockName] = blockList}
        local ok, dbg = saveBuildToFile(fileName, buildData)
        if ok then
            setStatus("  Saved: " .. fileName .. " (" .. #entries .. ")")
            if refreshFiles then refreshFiles() end
        else
            setStatus("  Save failed: " .. tostring(dbg))
        end
    end)
    end

    function createShapeContent()
        createShapeGUI_Part1(_SG)
        createShapeGUI_Part2(_SG)
    end
    createShapeContent()

    function createSettingsContent()
    local stSubBar = Instance.new("Frame")
    stSubBar.Size = UDim2.new(1, -6, 0, 24)
    stSubBar.BackgroundColor3 = Colors.PanelSoft
    stSubBar.BackgroundTransparency = 0
    stSubBar.BorderSizePixel = 0
    stSubBar.Parent = T4frame
    local stSubLayout = Instance.new("UIListLayout")
    stSubLayout.FillDirection = Enum.FillDirection.Horizontal
    stSubLayout.Padding = UDim.new(0, 2)
    stSubLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    stSubLayout.Parent = stSubBar
    local stSubPad = Instance.new("UIPadding")
    stSubPad.PaddingTop = UDim.new(0, 2)
    stSubPad.PaddingBottom = UDim.new(0, 2)
    stSubPad.PaddingLeft = UDim.new(0, 2)
    stSubPad.PaddingRight = UDim.new(0, 2)
    stSubPad.Parent = stSubBar

    local stContent = Instance.new("Frame")
    stContent.Size = UDim2.new(1, -4, 1, -30)
    stContent.Position = UDim2.new(0, 0, 0, 26)
    stContent.BackgroundTransparency = 1
    stContent.Parent = T4frame

    local function makeStSub(name, label)
        local btn = Instance.new("TextButton")
        btn.Name = name .. "StSubBtn"
        btn.Size = UDim2.new(0.333, -2, 1, 0)
        btn.BackgroundColor3 = Colors.PanelElevated
        btn.BackgroundTransparency = 0
        btn.BorderSizePixel = 0
        btn.Text = label
        btn.TextColor3 = Colors.Muted
        btn.TextSize = 10
        btn.Font = Enum.Font.GothamSemibold
        btn.Parent = stSubBar
        local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(0, 3); bc.Parent = btn
        local fr = Instance.new("ScrollingFrame")
        fr.Name = name .. "StSubFrame"
        fr.Size = UDim2.new(1, 0, 1, 0)
        fr.BackgroundTransparency = 1
        fr.ScrollBarThickness = 0
        fr.ScrollBarImageColor3 = Colors.Muted
        fr.CanvasSize = UDim2.new(0,0,0,0)
        pcall(function() fr.ElasticBehavior = Enum.ElasticBehavior.Never end)
        fr.Visible = false
        fr.Parent = stContent
        local fl = Instance.new("UIListLayout")
        fl.Padding = UDim.new(0, 5)
        fl.SortOrder = Enum.SortOrder.LayoutOrder
        fl.Parent = fr
        local _stResizeGuard = false
        fl:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            if _stResizeGuard then return end
            _stResizeGuard = true
            setScrollCanvas(fr, fl.AbsoluteContentSize.Y, 10)
            task.defer(function() _stResizeGuard = false end)
        end)
        btn.MouseButton1Click:Connect(function()
            for _, f in pairs(stContent:GetChildren()) do if f:IsA("ScrollingFrame") then f.Visible = false end end
            for _, b in pairs(stSubBar:GetChildren()) do
                if b:IsA("TextButton") then
                    b.BackgroundColor3 = Colors.PanelElevated
                    b.TextColor3 = Colors.Muted
                    local og = b:FindFirstChild("SPRB_SubGrad")
                    if og then og:Destroy() end
                end
            end
            fr.Visible = true
            btn.BackgroundColor3 = Colors.ActiveBG
            btn.TextColor3 = Colors.ActiveText
            local sg = Instance.new("UIGradient")
            sg.Name = "SPRB_SubGrad"
            sg.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
                ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
            })
            sg.Rotation = 90
            sg.Parent = btn
        end)
        return btn, fr
    end

    local guiSubBtn, guiSubFr = makeStSub("GUI", "GUI")
    local buildSubBtn, buildSubFr = makeStSub("Build", "BUILD")
    local farmSubBtn, farmSubFr = makeStSub("Farm", "FARM")
    guiSubFr.Visible = true
    guiSubBtn.BackgroundColor3 = Colors.ActiveBG
    guiSubBtn.TextColor3 = Colors.ActiveText
    do
        local ig = Instance.new("UIGradient")
        ig.Name = "SPRB_SubGrad"
        ig.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0.0, Settings.primaryColor),
            ColorSequenceKeypoint.new(1.0, Settings.secondaryColor)
        })
        ig.Rotation = 90
        ig.Parent = guiSubBtn
    end

    makeLabel("COLORS", guiSubFr)

    makeColorPicker("Primary", Settings.primaryColor, guiSubFr, function(c)
        Settings.primaryColor = c
        syncColors()
        applyWindowBackground(MainFrame)
        saveSettings()
        if refreshColors then refreshColors() end
    end)

    makeColorPicker("Secondary", Settings.secondaryColor, guiSubFr, function(c)
        Settings.secondaryColor = c
        syncColors()
        applyWindowBackground(MainFrame)
        saveSettings()
        if refreshColors then refreshColors() end
    end)

    task.wait()

    makeLabel("BACKGROUND", guiSubFr)
    local bgModes = {"default", "color"}
    local bgModeLabels = {default = "Default", color = "Custom Color"}
    local bgRowF = Instance.new("Frame")
    bgRowF.Size = UDim2.new(1, 0, 0, 28)
    bgRowF.BackgroundTransparency = 1
    bgRowF.Parent = guiSubFr
    local bgRowL = Instance.new("UIListLayout")
    bgRowL.FillDirection = Enum.FillDirection.Horizontal
    bgRowL.Padding = UDim.new(0, 3)
    bgRowL.Parent = bgRowF
    local bgModeBtns = {}
    for _, mode in ipairs(bgModes) do
        local mb = Instance.new("TextButton")
        mb.Size = UDim2.new(0.5, -2, 1, 0)
        mb.BackgroundColor3 = (Settings.bgMode == mode) and Colors.ActiveBG or Colors.PanelElevated
        mb.BackgroundTransparency = 0
        mb.BorderSizePixel = 0
        mb.Text = bgModeLabels[mode]
        mb.TextColor3 = (Settings.bgMode == mode) and Colors.ActiveText or Colors.Text
        mb.TextSize = 11
        mb.Font = Enum.Font.GothamSemibold
        mb.Parent = bgRowF
        local mbc = Instance.new("UICorner"); mbc.CornerRadius = UDim.new(0, 4); mbc.Parent = mb
        bgModeBtns[mode] = mb
        mb.MouseButton1Click:Connect(function()
            Settings.bgMode = mode
            for _, b in pairs(bgModeBtns) do
                b.BackgroundColor3 = Colors.PanelElevated
                b.TextColor3 = Colors.Text
            end
            mb.BackgroundColor3 = Colors.ActiveBG
            mb.TextColor3 = Colors.ActiveText
            applyWindowBackground(MainFrame)
            saveSettings()
        end)
    end

    makeColorPicker("Custom BG", Settings.bgCustomColor, guiSubFr, function(c)
        Settings.bgCustomColor = c
        if Settings.bgMode == "color" then applyWindowBackground(MainFrame) end
        saveSettings()
    end)

    makeLabel("INTERFACE", guiSubFr)
    makeSlider("UIScale", 0.5, 2.0, Settings.uiScale, guiSubFr, "UI Scale",
        function(v) return math.floor(v*100) .. "%" end,
        function(v)
            Settings.uiScale = math.clamp(v, 0.5, 2.0)
            Settings.windowWidth = math.floor(baseW * Settings.uiScale + 0.5)
            Settings.windowHeight = math.floor(baseH * Settings.uiScale + 0.5)
            local nW, nH = getTargetSize()
            local cx, cy = clampFramePosition(tonumber(Settings.windowPosX) or MainFrame.Position.X.Offset, tonumber(Settings.windowPosY) or MainFrame.Position.Y.Offset, nW, nH)
            saveFramePosition(cx, cy)
            MainFrame.Size = UDim2.new(0, nW, 0, nH)
            MainFrame.Position = UDim2.new(0, cx, 0, cy)
            updateTabSizes()
            if refreshContentCanvases then refreshContentCanvases() end
            saveSettings()
        end
    )

    makeSlider("GUITrans", 0, 95, math.min(math.floor(Settings.guiTransparency * 100 + 0.5), 95), guiSubFr, "GUI Transparency",
        function(v) return math.floor(v + 0.5) .. "%" end,
        function(v)
            Settings.guiTransparency = v / 100
            applyWindowBackground(MainFrame)
            saveSettings()
        end
    )

    makeSlider("PrevTrans", 0, 100, math.floor(Settings.previewTransparency * 100 + 0.5), guiSubFr, "Preview Transparency",
        function(v) return math.floor(v + 0.5) .. "%" end,
        function(v)
            Settings.previewTransparency = v / 100
            saveSettings()
        end
    )

    makeBtn("MobileModeBtn", "Mobile Mode: " .. (Settings.mobileMode and "ON" or "OFF"), guiSubFr, function()
        Settings.mobileMode = not Settings.mobileMode
        local b = guiSubFr:FindFirstChild("MobileModeBtn")
        if b then b.Text = "Mobile Mode: " .. (Settings.mobileMode and "ON" or "OFF") end
        if Settings.mobileMode then
            Settings.uiScale = math.min(Settings.uiScale, 0.78)
            Settings.windowWidth = math.floor(baseW * Settings.uiScale + 0.5)
            Settings.windowHeight = math.floor(baseH * Settings.uiScale + 0.5)
            local nW, nH = getTargetSize()
            local cx, cy = clampFramePosition(tonumber(Settings.windowPosX) or MainFrame.Position.X.Offset, tonumber(Settings.windowPosY) or MainFrame.Position.Y.Offset, nW, nH)
            saveFramePosition(cx, cy)
            MainFrame.Size = UDim2.new(0, nW, 0, nH)
            MainFrame.Position = UDim2.new(0, cx, 0, cy)
            updateTabSizes()
            if refreshContentCanvases then refreshContentCanvases() end
        end
        saveSettings()
    end)

    makeLabel("BG ANIMATION", guiSubFr)

    do
        local bgAnimDD, _ = makeDropdown("BgAnimDD", function()
            return {
                {name = "grid", display = "Grid"},
                {name = "constellation", display = "Constellation"},
                {name = "waves", display = "Waves"},
                {name = "smoke", display = "Smoke"},
                {name = "balls", display = "Bouncing Balls"},
            }
        end, guiSubFr, function(nm)
            Settings.bgAnim = nm
            saveSettings()
            applyWindowBackground(MainFrame)
        end)
        bgAnimDD.Text = Settings.bgAnim or "grid"
        bgAnimDD.TextColor3 = Colors.Text
    end
    makeBtn("BgAnimToggleBtn", "Bg Anim: " .. (Settings.bgAnimEnabled and "ON" or "OFF"), guiSubFr, function()
        Settings.bgAnimEnabled = not Settings.bgAnimEnabled
        local b = guiSubFr:FindFirstChild("BgAnimToggleBtn")
        if b then b.Text = "Bg Anim: " .. (Settings.bgAnimEnabled and "ON" or "OFF") end
        applyWindowBackground(MainFrame)
        saveSettings()
    end)
    makeBtn("BgAnimAutoColorBtn", "Auto Color: " .. (Settings.bgAnimAutoColor and "ON" or "OFF"), guiSubFr, function()
        Settings.bgAnimAutoColor = not Settings.bgAnimAutoColor
        local b = guiSubFr:FindFirstChild("BgAnimAutoColorBtn")
        if b then b.Text = "Auto Color: " .. (Settings.bgAnimAutoColor and "ON" or "OFF") end
        applyWindowBackground(MainFrame)
        saveSettings()
    end)
    makeColorPicker("BgAnimColor", Settings.bgAnimColor or Color3.fromRGB(90, 60, 200), guiSubFr, function(c)
        Settings.bgAnimColor = c
        applyWindowBackground(MainFrame)
        saveSettings()
    end)
    local bgAnimCountIn
    bgAnimCountIn = makeNumInput("Bg Anim Count:", Settings.bgAnimCount or 12, 1, 350, 1, guiSubFr, function(v)
        Settings.bgAnimCount = v
        applyWindowBackground(MainFrame)
        saveSettings()
    end)
    local bgAnimSpeedIn
    bgAnimSpeedIn = makeNumInput("Bg Anim Speed:", Settings.bgAnimSpeed or 1.0, 0.1, 5.0, 0.1, guiSubFr, function(v)
        Settings.bgAnimSpeed = v
        applyWindowBackground(MainFrame)
        saveSettings()
    end)
    local bgAnimSizeIn
    bgAnimSizeIn = makeNumInput("Bg Anim Size:", Settings.bgAnimSize or 1.0, 0.2, 10.0, 0.1, guiSubFr, function(v)
        Settings.bgAnimSize = v
        applyWindowBackground(MainFrame)
        saveSettings()
    end)

        makeBtn("SaveAllBtn", "Save All Settings", guiSubFr, function()
        saveSettings()
        setStatus("  Settings saved")
    end)

    makeLabel("DANGER ZONE", guiSubFr)
    local termBtn = makeBtn("TerminateBtn", "TERMINATE SCRIPT", guiSubFr, function()
        terminateScript(ScreenGui)
    end)
    termBtn.BackgroundColor3 = Color3.fromRGB(50, 10, 10)
    local termStroke = Instance.new("UIStroke")
    termStroke.Color = Color3.fromRGB(180, 30, 30)
    termStroke.Thickness = 1
    termStroke.Parent = termBtn

    makeLabel("SAVE FORMAT", buildSubFr)
    local asuNote = Instance.new("TextLabel")
    asuNote.Size = UDim2.new(1, 0, 0, 20)
    asuNote.BackgroundTransparency = 1
    asuNote.Text = "BH only | ASU loaded but not updated"
    asuNote.TextColor3 = Color3.fromRGB(160, 160, 160)
    asuNote.TextSize = 11
    asuNote.Font = Enum.Font.GothamBold
    asuNote.TextXAlignment = Enum.TextXAlignment.Left
    asuNote.Parent = buildSubFr

    makeSlider("SkyH", 0, 10000, Settings.skyHeight, buildSubFr, "Sky Base Height",
        function(v) return math.floor(v) end,
        function(v)
            Settings.skyHeight = math.floor(v)
            saveSettings()
        end
    )

    makeLabel("TOGGLES", buildSubFr)
    local function makeStToggleBtn(key, label, parent)
        local b = makeBtn(key .. "StToggleBtn", label .. ": " .. (Settings[key] and "ON" or "OFF"), parent, function()
            Settings[key] = not Settings[key]
            local btn = parent:FindFirstChild(key .. "StToggleBtn")
            if btn then
                btn.Text = label .. ": " .. (Settings[key] and "ON" or "OFF")
                btn.BackgroundColor3 = Settings[key] and Color3.fromRGB(16,32,16) or Colors.PanelElevated
            end
            saveSettings()
        end)
        b.BackgroundColor3 = Settings[key] and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        return b
    end

    makeStToggleBtn("autoPreview", "Auto Preview", buildSubFr)
    makeStToggleBtn("showBlockCounts", "Show Block Counts", buildSubFr)

    makeLabel("AUTO FARM SETTINGS", farmSubFr)
    makeLabel("DELAY (SEC)", farmSubFr)
    local farmDelayIn = makeInput("FarmDelay", "2", farmSubFr)
    farmDelayIn.Text = tostring(farmSettings and farmSettings.step or 2)
    farmDelayIn.FocusLost:Connect(function()
        local n = tonumber(farmDelayIn.Text)
        if n and n >= 0.5 then if farmSettings then farmSettings.step = n end; farmDelayIn.Text = tostring(n); if saveFarmSettings then saveFarmSettings() end
        else farmDelayIn.Text = tostring(farmSettings and farmSettings.step or 2) end
    end)

    makeLabel("TELEGRAM TOKEN", farmSubFr)
    local farmTgTokenIn = makeInput("FarmTgToken", "Bot token...", farmSubFr)
    farmTgTokenIn.Text = (farmSettings and farmSettings.tgToken) or ""
    farmTgTokenIn.FocusLost:Connect(function()
        if farmSettings then farmSettings.tgToken = farmTgTokenIn.Text end
        if saveFarmSettings then saveFarmSettings() end
    end)

    makeLabel("CHAT ID", farmSubFr)
    local farmTgChatIn = makeInput("FarmTgChat", "Chat ID...", farmSubFr)
    farmTgChatIn.Text = (farmSettings and farmSettings.tgChatID) or ""
    farmTgChatIn.FocusLost:Connect(function()
        if farmSettings then farmSettings.tgChatID = farmTgChatIn.Text end
        if saveFarmSettings then saveFarmSettings() end
    end)

    makeLabel("AUTO-SEND (MIN, 0=OFF)", farmSubFr)
    local farmTgIntIn = makeInput("FarmTgInt", "0", farmSubFr)
    farmTgIntIn.Text = tostring(farmSettings and farmSettings.tgInterval or 0)
    farmTgIntIn.FocusLost:Connect(function()
        local n = tonumber(farmTgIntIn.Text)
        if n and n >= 0 then if farmSettings then farmSettings.tgInterval = n end end
        farmTgIntIn.Text = tostring(farmSettings and farmSettings.tgInterval or 0)
        if saveFarmSettings then saveFarmSettings() end
    end)

    local farmTgToggleBtn = makeBtn("FarmTgToggleBtn", "Telegram: OFF", farmSubFr, function()

        if _toggleTGFarm then _toggleTGFarm() end
    end)
    _farmTgBtnRef = farmTgToggleBtn

    makeLabel("PERFORMANCE", farmSubFr)
    local renderToggleBtn = makeBtn("RenderToggleBtn", "Rendering: ON", farmSubFr, function()
        if farmSettings then farmSettings.renderEnabled = not farmSettings.renderEnabled end
        local disabled = farmSettings and (not farmSettings.renderEnabled)
        _G.rndr_dis = disabled
        local b = farmSubFr:FindFirstChild("RenderToggleBtn")
        if b then
            b.Text = "Rendering: " .. (not disabled and "ON" or "OFF")
            b.BackgroundColor3 = not disabled and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end

        pcall(function()
            local rs = game:GetService("RunService")
            rs:Set3dRenderingEnabled(not disabled)
        end)
        if saveFarmSettings then saveFarmSettings() end
    end)
    renderToggleBtn.BackgroundColor3 = Color3.fromRGB(16,32,16)

    task.wait()
    makeLabel("AUTO HOP", farmSubFr)
    local autoHopBtn = makeBtn("AutoHopBtn", "Auto Hop: " .. (farmSettings and farmSettings.autoHop and "ON" or "OFF"), farmSubFr, function()
        if farmSettings then farmSettings.autoHop = not farmSettings.autoHop end
        local b = farmSubFr:FindFirstChild("AutoHopBtn")
        if b then
            b.Text = "Auto Hop: " .. (farmSettings and farmSettings.autoHop and "ON" or "OFF")
            b.BackgroundColor3 = farmSettings and farmSettings.autoHop and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if saveFarmSettings then saveFarmSettings() end
    end)
    autoHopBtn.BackgroundColor3 = farmSettings and farmSettings.autoHop and Color3.fromRGB(16,32,16) or Colors.PanelElevated

    makeLabel("AUTO FARM ON JOIN", farmSubFr)
    local autoFarmBtn = makeBtn("AutoFarmJoinBtn", "Auto Farm: " .. (farmSettings and farmSettings.autoFarm and "ON" or "OFF"), farmSubFr, function()
        if farmSettings then farmSettings.autoFarm = not farmSettings.autoFarm end
        local b = farmSubFr:FindFirstChild("AutoFarmJoinBtn")
        if b then
            b.Text = "Auto Farm: " .. (farmSettings and farmSettings.autoFarm and "ON" or "OFF")
            b.BackgroundColor3 = farmSettings and farmSettings.autoFarm and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if saveFarmSettings then saveFarmSettings() end
    end)
    autoFarmBtn.BackgroundColor3 = farmSettings and farmSettings.autoFarm and Color3.fromRGB(16,32,16) or Colors.PanelElevated
    if farmSettings and not farmSettings.autoFarmFile then farmSettings.autoFarmFile = "" end
    local autoFarmFileIn = makeInput("AutoFarmFileIn", "farm file name (optional)", farmSubFr)
    autoFarmFileIn.Text = (farmSettings and farmSettings.autoFarmFile) or ""
    autoFarmFileIn.FocusLost:Connect(function()
        if farmSettings then farmSettings.autoFarmFile = autoFarmFileIn.Text end
        if saveFarmSettings then saveFarmSettings() end
    end)

    makeLabel("AUTO BUILD ON JOIN", farmSubFr)
    local autoBuildBtn = makeBtn("AutoBuildJoinBtn", "Auto Build: " .. (farmSettings and farmSettings.autoBuild and "ON" or "OFF"), farmSubFr, function()
        if farmSettings then farmSettings.autoBuild = not farmSettings.autoBuild end
        local b = farmSubFr:FindFirstChild("AutoBuildJoinBtn")
        if b then
            b.Text = "Auto Build: " .. (farmSettings and farmSettings.autoBuild and "ON" or "OFF")
            b.BackgroundColor3 = farmSettings and farmSettings.autoBuild and Color3.fromRGB(16,32,16) or Colors.PanelElevated
        end
        if saveFarmSettings then saveFarmSettings() end
    end)
    autoBuildBtn.BackgroundColor3 = farmSettings and farmSettings.autoBuild and Color3.fromRGB(16,32,16) or Colors.PanelElevated
    if farmSettings and not farmSettings.autoBuildFile then farmSettings.autoBuildFile = "" end
    local autoBuildFileIn = makeInput("AutoBuildFileIn", "build file name", farmSubFr)
    autoBuildFileIn.Text = (farmSettings and farmSettings.autoBuildFile) or ""

    autoBuildFileIn.FocusLost:Connect(function()
        if farmSettings then farmSettings.autoBuildFile = autoBuildFileIn.Text end
        if saveFarmSettings then saveFarmSettings() end
    end)


    do
        local asuBtn = Instance.new("TextButton")
        asuBtn.Name = "AsuHeartBtn"
        asuBtn.Size = UDim2.new(1, 0, 0, 52)
        asuBtn.BackgroundColor3 = Color3.fromRGB(80, 20, 30)
        asuBtn.BackgroundTransparency = 0
        asuBtn.BorderSizePixel = 0
        asuBtn.Text = "asu \226\153\161"
        asuBtn.TextColor3 = Color3.fromRGB(255, 150, 170)
        asuBtn.TextSize = 24
        asuBtn.Font = Enum.Font.GothamBold
        asuBtn.AutoButtonColor = false
        asuBtn.Parent = buildSubFr
        local asuCr = Instance.new("UICorner"); asuCr.CornerRadius = UDim.new(0, 8); asuCr.Parent = asuBtn
        local asuScale = Instance.new("UIScale"); asuScale.Scale = 1; asuScale.Parent = asuBtn
        asuBtn.MouseButton1Click:Connect(function()
            playUISound(UISoundConfig.click)
            tween(asuScale, TweenInfo.new(0.08), {Scale = 0.92}):Play()
            task.delay(0.08, function() tween(asuScale, TweenInfo.new(0.15, Enum.EasingStyle.Back), {Scale = 1}):Play() end)

            task.spawn(function()
                local gui = ScreenGui
                if not gui then return end
                local btnAbsPos = asuBtn.AbsolutePosition
                local btnAbsSize = asuBtn.AbsoluteSize
                local centerX = btnAbsPos.X + btnAbsSize.X / 2
                local centerY = btnAbsPos.Y + btnAbsSize.Y / 2
                local hearts = {}
                for h = 1, 8 do
                    local heart = Instance.new("TextLabel")
                    heart.Size = UDim2.new(0, 36, 0, 36)
                    heart.BackgroundTransparency = 1
                    heart.Text = "\226\153\161"
                    heart.TextColor3 = Color3.fromRGB(255, math.random(80, 180), math.random(120, 200))
                    heart.TextSize = 28
                    heart.Font = Enum.Font.GothamBold
                    heart.ZIndex = 999
                    heart.Parent = gui
                    local startX = centerX + math.random(-40, 40)
                    local startY = centerY + math.random(-10, 10)
                    heart.Position = UDim2.new(0, startX, 0, startY)
                    hearts[#hearts+1] = heart
                    local targetY = startY - math.random(60, 160)
                    local targetX = startX + math.random(-50, 50)
                    tween(heart, TweenInfo.new(math.random(12, 20)/10, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                        Position = UDim2.new(0, targetX, 0, targetY),
                        TextTransparency = 1,
                    }):Play()
                end
                task.wait(2)
                for _, h in ipairs(hearts) do
                    pcall(function() h:Destroy() end)
                end
            end)
        end)
    end
    end

    createSettingsContent()
    switchTab(T1frame)
    updateTabSizes()
    TabsBar:GetPropertyChangedSignal("AbsoluteSize"):Connect(updateTabSizes)
    ContentArea:GetPropertyChangedSignal("AbsoluteSize"):Connect(refreshContentCanvases)
    T1btn.MouseButton1Click:Connect(function() switchTab(T1frame) end)
    T2btn.MouseButton1Click:Connect(function() switchTab(T2frame) ; updateBlocksDisplayGlobal() end)
    T3btn.MouseButton1Click:Connect(function() switchTab(T3frame) end)
    T4btn.MouseButton1Click:Connect(function() switchTab(T4frame) end)

    do
    local dragging = false
    local resizing = false
    local resizeMode = nil
    local dragStart, dragStartPos
    local resizeStartPos, resizeStartSize
    local frameMotionTween = nil

    local function tweenFrame(props, dur, style, dir)
        if frameMotionTween then pcall(function() frameMotionTween:Cancel() end) end
        frameMotionTween = TweenService:Create(MainFrame, TweenInfo.new(dur or 0.08, style or Enum.EasingStyle.Back, dir or Enum.EasingDirection.Out), props)
        frameMotionTween:Play()
        return frameMotionTween
    end

    local function makeResizeHandle(name, pos, size, mode)
        local h = Instance.new("TextButton")
        h.Name = name
        h.Position = pos
        h.Size = size
        h.BackgroundTransparency = 1
        h.BorderSizePixel = 0
        h.Text = ""
        h.AutoButtonColor = false
        h.Active = true
        h.ZIndex = 120
        h.Parent = MainFrame
        h.InputBegan:Connect(function(inp)
            if minimized then return end
            if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
            resizing = true
            resizeMode = mode
            dragStart = Vector2.new(inp.Position.X, inp.Position.Y)
            resizeStartPos = MainFrame.AbsolutePosition
            resizeStartSize = MainFrame.AbsoluteSize
            if frameMotionTween then
                pcall(function() frameMotionTween:Cancel() end)
                frameMotionTween = nil
            end
        end)
        return h
    end

    makeResizeHandle("ResizeRight", UDim2.new(1, -9, 0, 8), UDim2.new(0, 18, 1, -16), "right")
    makeResizeHandle("ResizeLeft", UDim2.new(0, -9, 0, 8), UDim2.new(0, 18, 1, -16), "left")
    makeResizeHandle("ResizeBottom", UDim2.new(0, 8, 1, -9), UDim2.new(1, -16, 0, 18), "bottom")
    makeResizeHandle("ResizeTop", UDim2.new(0, 8, 0, -9), UDim2.new(1, -16, 0, 18), "top")
    makeResizeHandle("ResizeBR", UDim2.new(1, -18, 1, -18), UDim2.new(0, 26, 0, 26), "bottomright")
    makeResizeHandle("ResizeBL", UDim2.new(0, -8, 1, -18), UDim2.new(0, 26, 0, 26), "bottomleft")
    makeResizeHandle("ResizeTR", UDim2.new(1, -18, 0, -8), UDim2.new(0, 26, 0, 26), "topright")
    makeResizeHandle("ResizeTL", UDim2.new(0, -8, 0, -8), UDim2.new(0, 26, 0, 26), "topleft")

    createResizeGrip(MainFrame, Colors.Muted)

    Header.InputBegan:Connect(function(inp)
        if inp.UserInputType ~= Enum.UserInputType.MouseButton1 and inp.UserInputType ~= Enum.UserInputType.Touch then return end
        if pointInsideAnyButton(Header, inp.Position.X, inp.Position.Y) then return end
        dragging = true
        dragStart = Vector2.new(inp.Position.X, inp.Position.Y)
        dragStartPos = MainFrame.Position
        if frameMotionTween then
            pcall(function() frameMotionTween:Cancel() end)
            frameMotionTween = nil
        end
    end)

    UserInputService.InputChanged:Connect(function(inp)
        if inp.UserInputType ~= Enum.UserInputType.MouseMovement and inp.UserInputType ~= Enum.UserInputType.Touch then return end
        if resizing and resizeStartPos and resizeStartSize then
            local dx = inp.Position.X - dragStart.X
            local dy = inp.Position.Y - dragStart.Y
            local x, y = resizeStartPos.X, resizeStartPos.Y
            local w, h = resizeStartSize.X, resizeStartSize.Y
            if resizeMode:find("right") then w = resizeStartSize.X + dx end
            if resizeMode:find("bottom") then h = resizeStartSize.Y + dy end
            if resizeMode:find("left") then
                w = resizeStartSize.X - dx
                x = resizeStartPos.X + dx
            end
            if resizeMode:find("top") then
                h = resizeStartSize.Y - dy
                y = resizeStartPos.Y + dy
            end
            local cw, ch = clampFrameSize(w, h)
            if resizeMode:find("left") then x = resizeStartPos.X + (resizeStartSize.X - cw) end
            if resizeMode:find("top") then y = resizeStartPos.Y + (resizeStartSize.Y - ch) end
            x, y = clampFramePosition(x, y, cw, ch)
            MainFrame.Size = UDim2.new(0, cw, 0, ch)
            MainFrame.Position = UDim2.new(0, x, 0, y)
            MainFrame.Rotation = 0
            updateTabSizes()
            if refreshContentCanvases then refreshContentCanvases() end
            saveFramePosition(x, y)
            saveFrameSize(cw, ch)
            return
        end
        if not dragging then return end
        local dx = inp.Position.X - dragStart.X
        local dy = inp.Position.Y - dragStart.Y
        local nX, nY = clampFramePosition(dragStartPos.X.Offset + dx, dragStartPos.Y.Offset + dy, MainFrame.AbsoluteSize.X, MainFrame.AbsoluteSize.Y)
        if frameMotionTween then
            pcall(function() frameMotionTween:Cancel() end)
            frameMotionTween = nil
        end
        tweenFrame({
            Position = UDim2.new(0, nX, 0, nY),
            Rotation = 0
        }, 0.045, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        saveFramePosition(nX, nY)
    end)

    UserInputService.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            local wasDragging = dragging
            dragging = false
            resizing = false
            resizeMode = nil
            if MainFrame.Visible then
                local props = {Rotation = 0}
                if wasDragging then
                    props.Position = UDim2.new(0, tonumber(Settings.windowPosX) or MainFrame.Position.X.Offset, 0, tonumber(Settings.windowPosY) or MainFrame.Position.Y.Offset)
                end
                local t = tweenFrame(props, 0.12, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
                t.Completed:Connect(function()
                    if MainFrame and MainFrame.Parent then MainFrame.Rotation = 0 end
                end)
            end
            saveSettings()
        end
    end)

    MinBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        Settings.uiMinimized = minimized
        local targetW, targetH = getTargetSize()
        local currentPos = MainFrame.AbsolutePosition
        local targetX, targetY = clampFramePosition(currentPos.X, currentPos.Y, targetW, targetH)
        saveFramePosition(targetX, targetY)
        TabsBar.Visible = false
        ContentArea.Visible = false
        MinBtn.Text = minimized and "+" or "-"
        local foldTween = TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, targetW, 0, targetH),
            Position = UDim2.new(0, targetX, 0, targetY)
        })
        foldTween:Play()
        foldTween.Completed:Connect(function()
            if not MainFrame.Visible then return end
            TabsBar.Visible = not minimized
            ContentArea.Visible = not minimized
            updateTabSizes()
            if refreshContentCanvases then refreshContentCanvases() end
            MainFrame.Rotation = 0
        end)
        saveSettings()
    end)
    end

    OpenBtn = createOpenButton(ScreenGui, Colors.Text)
    bindWindowButtons(CloseBtn, OpenBtn, showGUI, hideGUI)

    pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
    if not ScreenGui.Parent then ScreenGui.Parent = safeWaitChild(LocalPlayer, "PlayerGui", 10) end

    showGUI()
    task.delay(0.35, function() applyWindowBackground(MainFrame) end)

    return ScreenGui
end

ensureFolder()
loadSettings()
syncColors()

saveSettings()

rebuildUI = function()
    if UI then pcall(function() UI:Destroy() end) end
    UI = createUI()

    if _G._afterCreateUI then
        for _, cb in ipairs(_G._afterCreateUI) do
            pcall(cb)
        end
    end
end

do
local function dbg(msg)
end

local function makeLoadingGui()
    local pgui = safeWaitChild(LocalPlayer, "PlayerGui", 10)
    local sg = Instance.new("ScreenGui")
    sg.Name = "SPRB_Loader"; sg.ResetOnSpawn = false; sg.IgnoreGuiInset = true
    sg.ZIndexBehavior = Enum.ZIndexBehavior.Global
    sg.DisplayOrder = 999999
    pcall(function() sg.Parent = pgui end)
    pcall(function()
        StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.All, false)
    end)
    local hiddenGuis = {}
    for _, gui in ipairs(pgui:GetChildren()) do
        if gui:IsA("ScreenGui") and gui ~= sg and gui.Enabled then
            hiddenGuis[#hiddenGuis + 1] = gui
            gui.Enabled = false
        end
    end
    local blur = Instance.new("BlurEffect")
    blur.Name = "SPRB_LoadBlur"
    blur.Size = 22
    blur.Parent = Lighting

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0); bg.BackgroundColor3 = Color3.fromRGB(235, 235, 235)
    bg.BackgroundTransparency = 0.42
    bg.BorderSizePixel = 0; bg.ZIndex = 1; bg.Parent = sg
    local bgGrad = Instance.new("UIGradient")
    bgGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(160, 160, 160)),
    })
    bgGrad.Rotation = 90; bgGrad.Parent = bg

    local glow = Instance.new("Frame")
    glow.Size = UDim2.new(0, 600, 0, 600); glow.Position = UDim2.new(0.5, -300, 0.5, -300)
    glow.BackgroundColor3 = Color3.fromRGB(255, 255, 255); glow.BackgroundTransparency = 0.82
    glow.BorderSizePixel = 0; glow.ZIndex = 2; glow.Parent = bg
    local glowCr = Instance.new("UICorner"); glowCr.CornerRadius = UDim.new(1, 0); glowCr.Parent = glow

    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 460, 0, 230); card.Position = UDim2.new(0.5, -230, 0.5, -115)
    card.BackgroundColor3 = Color3.fromRGB(12, 12, 12); card.BorderSizePixel = 0; card.ZIndex = 3; card.Parent = bg
    local cc = Instance.new("UICorner"); cc.CornerRadius = UDim.new(0, 12); cc.Parent = card
    local cs = Instance.new("UIStroke"); cs.Color = Color3.fromRGB(245, 245, 245); cs.Transparency = 0.35; cs.Thickness = 1.5; cs.Parent = card
    local cGrad = Instance.new("UIGradient")
    cGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(28, 28, 28)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(8, 8, 8)),
    })
    cGrad.Rotation = 90; cGrad.Parent = card

    local spinner = Instance.new("Frame")
    spinner.Size = UDim2.new(0, 44, 0, 44); spinner.Position = UDim2.new(0, 20, 0, 22)
    spinner.BackgroundTransparency = 1; spinner.ZIndex = 4; spinner.Parent = card
    local spArc = Instance.new("TextLabel"); spArc.Size = UDim2.new(1, 0, 1, 0)
    spArc.BackgroundTransparency = 1; spArc.Text = "\xe2\x97\x90"
    spArc.TextColor3 = Color3.fromRGB(245, 245, 245); spArc.TextSize = 34
    spArc.Font = Enum.Font.GothamBold; spArc.Parent = spinner

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -84, 0, 28); title.Position = UDim2.new(0, 76, 0, 20)
    title.BackgroundTransparency = 1; title.Text = "SPRB"; title.TextColor3 = Color3.fromRGB(245, 245, 245)
    title.TextSize = 24; title.Font = Enum.Font.GothamBold; title.TextXAlignment = Enum.TextXAlignment.Left
    title.ZIndex = 4; title.Parent = card

    local verL = Instance.new("TextLabel")
    verL.Size = UDim2.new(1, -84, 0, 16); verL.Position = UDim2.new(0, 76, 0, 48)
    verL.BackgroundTransparency = 1; verL.Text = "Loading system..."; verL.TextColor3 = Color3.fromRGB(170, 170, 170)
    verL.TextSize = 11; verL.Font = Enum.Font.GothamMedium; verL.TextXAlignment = Enum.TextXAlignment.Left
    verL.ZIndex = 4; verL.Parent = card

    local statusL = Instance.new("TextLabel")
    statusL.Name = "Status"; statusL.Size = UDim2.new(1, -32, 0, 18); statusL.Position = UDim2.new(0, 16, 0, 86)
    statusL.BackgroundTransparency = 1; statusL.Text = "Initializing..."; statusL.TextColor3 = Color3.fromRGB(230, 230, 230)
    statusL.TextSize = 13; statusL.Font = Enum.Font.GothamSemibold; statusL.TextXAlignment = Enum.TextXAlignment.Left
    statusL.ZIndex = 4; statusL.Parent = card

    local subL = Instance.new("TextLabel")
    subL.Name = "Sub"; subL.Size = UDim2.new(1, -32, 0, 14); subL.Position = UDim2.new(0, 16, 0, 108)
    subL.BackgroundTransparency = 1; subL.Text = "Preparing environment"; subL.TextColor3 = Color3.fromRGB(150, 150, 150)
    subL.TextSize = 10; subL.Font = Enum.Font.GothamMedium; subL.TextXAlignment = Enum.TextXAlignment.Left
    subL.ZIndex = 4; subL.Parent = card

    local barBg = Instance.new("Frame")
    barBg.Size = UDim2.new(1, -32, 0, 10); barBg.Position = UDim2.new(0, 16, 0, 138)
    barBg.BackgroundColor3 = Color3.fromRGB(40, 40, 40); barBg.BorderSizePixel = 0; barBg.ZIndex = 4; barBg.Parent = card
    local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(1, 0); bc.Parent = barBg
    local bStroke = Instance.new("UIStroke"); bStroke.Color = Color3.fromRGB(110, 110, 110); bStroke.Transparency = 0.55; bStroke.Thickness = 1; bStroke.Parent = barBg

    local barFill = Instance.new("Frame")
    barFill.Name = "Fill"; barFill.Size = UDim2.new(0, 0, 1, 0); barFill.BackgroundColor3 = Color3.fromRGB(245, 245, 245)
    barFill.BorderSizePixel = 0; barFill.ZIndex = 5; barFill.Parent = barBg
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(1, 0); fc.Parent = barFill
    local fillGrad = Instance.new("UIGradient")
    fillGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 150, 150)),
    })
    fillGrad.Parent = barFill

    local shimmer = Instance.new("Frame")
    shimmer.Name = "Shimmer"; shimmer.Size = UDim2.new(0.3, 0, 1, 0)
    shimmer.BackgroundColor3 = Color3.fromRGB(255, 255, 255); shimmer.BackgroundTransparency = 0.75
    shimmer.BorderSizePixel = 0; shimmer.ZIndex = 6; shimmer.Parent = barFill
    local shCr = Instance.new("UICorner"); shCr.CornerRadius = UDim.new(1, 0); shCr.Parent = shimmer

    local pctL = Instance.new("TextLabel")
    pctL.Name = "Pct"; pctL.Size = UDim2.new(1, -32, 0, 16); pctL.Position = UDim2.new(0, 16, 0, 154)
    pctL.BackgroundTransparency = 1; pctL.Text = "0%"; pctL.TextColor3 = Color3.fromRGB(170, 170, 170)
    pctL.TextSize = 11; pctL.Font = Enum.Font.GothamBold; pctL.TextXAlignment = Enum.TextXAlignment.Right
    pctL.ZIndex = 4; pctL.Parent = card

    local errL = Instance.new("TextLabel")
    errL.Name = "Err"; errL.Size = UDim2.new(1, -32, 0, 44); errL.Position = UDim2.new(0, 16, 0, 178)
    errL.BackgroundTransparency = 1; errL.Text = ""; errL.TextColor3 = Color3.fromRGB(255, 120, 120)
    errL.TextSize = 10; errL.Font = Enum.Font.GothamMedium; errL.TextWrapped = true
    errL.TextXAlignment = Enum.TextXAlignment.Left; errL.TextYAlignment = Enum.TextYAlignment.Top
    errL.ZIndex = 4; errL.Parent = card

    local currentPct = 0
    local targetPct = 0
    local function setProgress(pct, status, sub)
        targetPct = math.clamp(pct, 0, 1)
        pcall(function()
            if status then statusL.Text = status end
            if sub then subL.Text = sub end
        end)
    end
    local function setError(msg)
        pcall(function()
            errL.Text = tostring(msg or "Unknown error")
            statusL.Text = "Load failed"
            subL.Text = "Loading stopped"
            targetPct = 1
        end)
    end

    local animActive = true
    local shimmerPos = -0.3
    local glowPhase = 0
    task.spawn(function()
        local conn
        conn = RunService.RenderStepped:Connect(function()
            if not animActive then pcall(function() conn:Disconnect() end) return end
            pcall(function()
                currentPct = currentPct + (targetPct - currentPct) * 0.10
                barFill.Size = UDim2.new(currentPct, 0, 1, 0)
                pctL.Text = tostring(math.floor(currentPct * 100 + 0.5)) .. "%"
                spinner.Rotation = spinner.Rotation + 7
                shimmerPos = shimmerPos + 0.018
                if shimmerPos > 1 then shimmerPos = -0.3 end
                shimmer.Position = UDim2.new(shimmerPos, 0, 0, 0)
                glowPhase = glowPhase + 0.035
                local p = 0.88 + 0.08 * math.sin(glowPhase)
                glow.BackgroundTransparency = p
                local gs = 560 + 50 * math.sin(glowPhase)
                glow.Size = UDim2.new(0, gs, 0, gs)
                glow.Position = UDim2.new(0.5, -gs/2, 0.5, -gs/2)
            end)
        end)
    end)

    local function fadeOut()
        animActive = false
        pcall(function()
            local t = TweenService:Create(bg, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            local t2 = TweenService:Create(card, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            local t3 = TweenService:Create(glow, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            t:Play(); t2:Play(); t3:Play()
            task.wait(0.55)
            sg:Destroy()
            pcall(function() blur:Destroy() end)
            pcall(function()
                StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.All, true)
            end)
            for _, gui in ipairs(hiddenGuis) do
                if gui and gui.Parent then pcall(function() gui.Enabled = true end) end
            end
        end)
    end

    return { gui = sg, setProgress = setProgress, setError = setError, fadeOut = fadeOut }
end

local function runLoader()
    local loader = makeLoadingGui()
    dbg("Loader started")

    loader.setProgress(0.05, "Preparing...", "Initial delay")
    task.wait(0.3)

    loader.setProgress(0.15, "Loading settings...", "Reading config")
    dbg("Stage 1: settings")
    pcall(ensureFolder)
    pcall(loadSettings)
    pcall(syncColors)
    pcall(saveSettings)
    task.wait(0.1)

    loader.setProgress(0.30, "Building interface...", "Creating UI elements")
    dbg("Stage 2: createUI start")
    task.wait(0.1)

    local uiOk, uiErr = xpcall(function()
        UI = createUI()
    end, function(err)
        local tb = debug.traceback(tostring(err), 2)
        return tb
    end)
    if not uiOk then
        dbg("createUI ERROR:\n" .. tostring(uiErr))
        loader.setError(uiErr)
        task.wait(5)
        loader.fadeOut()
        return
    end
    dbg("Stage 2: createUI done")

    if _G._afterCreateUI then
        for _, cb in ipairs(_G._afterCreateUI) do
            cb()
        end
        _G._afterCreateUI = nil
    end
    loader.setProgress(0.88, "Finalizing...", "Starting services")
    task.wait(0.45)

    loader.setProgress(1.0, "Ready!", "SPRB loaded successfully")
    dbg("Loader complete")
    task.wait(0.8)
    loader.fadeOut()

    local tutorialSeen = nil
    pcall(function()
        if isfile("SOPERA_tutorial.txt") then tutorialSeen = readfile("SOPERA_tutorial.txt") end
    end)
    if tutorialSeen ~= "1" then
        task.wait(0.5)
        local tutSg = Instance.new("ScreenGui")
        tutSg.Name = "SPRB_Tutorial"; tutSg.ResetOnSpawn = false; tutSg.IgnoreGuiInset = true
        tutSg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        pcall(function() tutSg.Parent = safeWaitChild(LocalPlayer, "PlayerGui", 10) end)

        local tutBg = Instance.new("Frame")
        tutBg.Size = UDim2.new(1, 0, 1, 0); tutBg.BackgroundColor3 = Color3.fromRGB(235, 235, 235)
        tutBg.BackgroundTransparency = 0.42; tutBg.BorderSizePixel = 0; tutBg.Parent = tutSg
        local bgGrad = Instance.new("UIGradient")
        bgGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 150, 150)),
        })
        bgGrad.Rotation = 90; bgGrad.Parent = tutBg

        local tutCard = Instance.new("Frame")
        tutCard.Size = UDim2.new(0, 480, 0, 340); tutCard.Position = UDim2.new(0.5, -240, 0.5, -170)
        tutCard.BackgroundColor3 = Color3.fromRGB(12, 12, 12); tutCard.BorderSizePixel = 0; tutCard.Parent = tutBg
        local tcCr = Instance.new("UICorner"); tcCr.CornerRadius = UDim.new(0, 14); tcCr.Parent = tutCard
        local tcSt = Instance.new("UIStroke"); tcSt.Color = Color3.fromRGB(245, 245, 245); tcSt.Transparency = 0.35; tcSt.Thickness = 2; tcSt.Parent = tutCard
        local tcGrad = Instance.new("UIGradient")
        tcGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(28, 28, 28)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(8, 8, 8)),
        })
        tcGrad.Rotation = 90; tcGrad.Parent = tutCard

        local tutGlow = Instance.new("Frame")
        tutGlow.Size = UDim2.new(0, 560, 0, 560); tutGlow.Position = UDim2.new(0.5, -280, 0.5, -280)
        tutGlow.BackgroundColor3 = Color3.fromRGB(255, 255, 255); tutGlow.BackgroundTransparency = 0.84
        tutGlow.BorderSizePixel = 0; tutGlow.ZIndex = 0; tutGlow.Parent = tutBg
        local tgCr = Instance.new("UICorner"); tgCr.CornerRadius = UDim.new(1, 0); tgCr.Parent = tutGlow

        local tutTitle = Instance.new("TextLabel")
        tutTitle.Size = UDim2.new(1, -28, 0, 36); tutTitle.Position = UDim2.new(0, 14, 0, 20)
        tutTitle.BackgroundTransparency = 1; tutTitle.Text = "SPRB // V5"; tutTitle.TextColor3 = Color3.fromRGB(245, 245, 245)
        tutTitle.TextSize = 26; tutTitle.Font = Enum.Font.GothamBold; tutTitle.TextXAlignment = Enum.TextXAlignment.Left
        tutTitle.Parent = tutCard

        local tutSub = Instance.new("TextLabel")
        tutSub.Size = UDim2.new(1, -28, 0, 18); tutSub.Position = UDim2.new(0, 14, 0, 58)
        tutSub.BackgroundTransparency = 1; tutSub.Text = "Build Tool for Build A Boat For Treasure"; tutSub.TextColor3 = Color3.fromRGB(170, 170, 170)
        tutSub.TextSize = 12; tutSub.Font = Enum.Font.GothamMedium; tutSub.TextXAlignment = Enum.TextXAlignment.Left
        tutSub.Parent = tutCard

        local divider = Instance.new("Frame")
        divider.Size = UDim2.new(1, -28, 0, 1); divider.Position = UDim2.new(0, 14, 0, 84)
        divider.BackgroundColor3 = Color3.fromRGB(120, 120, 120); divider.BackgroundTransparency = 0.5; divider.BorderSizePixel = 0; divider.Parent = tutCard

        local tutBody = Instance.new("TextLabel")
        tutBody.Size = UDim2.new(1, -28, 0, 80); tutBody.Position = UDim2.new(0, 14, 0, 94)
        tutBody.BackgroundTransparency = 1
        tutBody.Text = "Put your .Build / .json / .obj files in:\nSOPERA_WORKSPACE folder\n\nFor help, tutorials, and updates - join our Telegram:"
        tutBody.TextColor3 = Color3.fromRGB(215, 215, 215); tutBody.TextSize = 13; tutBody.Font = Enum.Font.GothamMedium
        tutBody.TextWrapped = true; tutBody.TextXAlignment = Enum.TextXAlignment.Left; tutBody.TextYAlignment = Enum.TextYAlignment.Top
        tutBody.Parent = tutCard

        local tgLink = Instance.new("TextButton")
        tgLink.Size = UDim2.new(1, -28, 0, 44); tgLink.Position = UDim2.new(0, 14, 0, 184)
        tgLink.BackgroundColor3 = Color3.fromRGB(245, 245, 245); tgLink.BorderSizePixel = 0
        tgLink.Text = "t.me/SoPeRaChan"; tgLink.TextColor3 = Color3.fromRGB(8, 8, 8)
        tgLink.TextSize = 16; tgLink.Font = Enum.Font.GothamBold; tgLink.Parent = tutCard
        local tlCr = Instance.new("UICorner"); tlCr.CornerRadius = UDim.new(0, 8); tlCr.Parent = tgLink
        local tlGrad = Instance.new("UIGradient")
        tlGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(170, 170, 170)),
        })
        tlGrad.Rotation = 90; tlGrad.Parent = tgLink
        local tlSt = Instance.new("UIStroke"); tlSt.Color = Color3.fromRGB(80, 80, 80); tlSt.Transparency = 0.45; tlSt.Thickness = 1; tlSt.Parent = tgLink
        local tlScale = Instance.new("UIScale"); tlScale.Scale = 1; tlScale.Parent = tgLink
        tgLink.MouseEnter:Connect(function()
            TweenService:Create(tlScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.03}):Play()
        end)
        tgLink.MouseLeave:Connect(function()
            TweenService:Create(tlScale, TweenInfo.new(0.12), {Scale = 1}):Play()
        end)
        tgLink.MouseButton1Click:Connect(function()
            pcall(function() setclipboard("t.me/SoPeRaChan") end)
            local orig = tgLink.Text
            tgLink.Text = "Copied! Join TG and ask in chat"
            TweenService:Create(tlScale, TweenInfo.new(0.08), {Scale = 0.95}):Play()
            task.wait(0.1)
            TweenService:Create(tlScale, TweenInfo.new(0.12, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1.03}):Play()
            task.wait(2.5)
            tgLink.Text = orig
        end)

        local tutClose = Instance.new("TextButton")
        tutClose.Size = UDim2.new(1, -28, 0, 44); tutClose.Position = UDim2.new(0, 14, 0, 240)
        tutClose.BackgroundColor3 = Color3.fromRGB(28, 28, 28); tutClose.BorderSizePixel = 0
        tutClose.Text = "Got it!"; tutClose.TextColor3 = Color3.fromRGB(245, 245, 245)
        tutClose.TextSize = 16; tutClose.Font = Enum.Font.GothamBold; tutClose.Parent = tutCard
        local tccCr = Instance.new("UICorner"); tccCr.CornerRadius = UDim.new(0, 8); tccCr.Parent = tutClose
        local tccSt = Instance.new("UIStroke"); tccSt.Color = Color3.fromRGB(110, 110, 110); tccSt.Transparency = 0.55; tccSt.Thickness = 1; tccSt.Parent = tutClose
        local tcScale = Instance.new("UIScale"); tcScale.Scale = 1; tcScale.Parent = tutClose
        tutClose.MouseEnter:Connect(function()
            TweenService:Create(tcScale, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Scale = 1.02}):Play()
        end)
        tutClose.MouseLeave:Connect(function()
            TweenService:Create(tcScale, TweenInfo.new(0.12), {Scale = 1}):Play()
        end)
        tutClose.MouseButton1Click:Connect(function()
            pcall(function() writefile("SOPERA_tutorial.txt", "1") end)
            TweenService:Create(tcScale, TweenInfo.new(0.08), {Scale = 0.95}):Play()
            task.wait(0.1)
            local t = TweenService:Create(tutBg, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            local t2 = TweenService:Create(tutCard, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            local t3 = TweenService:Create(tutGlow, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
            t:Play(); t2:Play(); t3:Play()
            task.wait(0.45)
            tutSg:Destroy()
        end)
    end
end

runLoader()
end

LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    Humanoid = getHumanoid(newChar)
end)

task.spawn(function()
    if not farmSettings.autoHop then return end
    LocalPlayer.OnTeleport:Connect(function(state)
        if state == Enum.TeleportState.Failed then
            task.wait(2)
            pcall(function() TeleportService:Teleport(game.PlaceId, LocalPlayer) end)
        end
    end)
end)

task.spawn(function()
    if not farmSettings.autoHop then return end
    while true do
        task.wait(30)
        local fps = 60
        pcall(function()
            local times = {}
            for i = 1, 10 do
                local t = tick()
                RunService.Heartbeat:Wait()
                times[i] = tick() - t
            end
            local avg = 0
            for _, t in ipairs(times) do avg = avg + t end
            avg = avg / #times
            fps = 1 / avg
        end)
        if fps < 5 then
            pcall(function() TeleportService:Teleport(game.PlaceId, LocalPlayer) end)
            return
        end
    end
end)
task.spawn(function()
    if not farmSettings.autoFarm then return end
    task.wait(5)
    pcall(function()
        local btn = miscFr:FindFirstChild("AutoFarmOpenBtn")
        if btn then
            btn.MouseButton1Click:Fire()
            task.wait(1)
            if farmSettings.autoFarmFile ~= "" then
                local startBtn = Workspace:FindFirstChild("SPRB_FarmPlat") and Workspace.SPRB_FarmPlat.Parent and miscFr:FindFirstChildWhichIsA("TextButton", true)
            end
        end
    end)
end)
task.spawn(function()
    if not farmSettings.autoBuild then return end
    task.wait(8)
    pcall(function()
        if farmSettings.autoBuildFile ~= "" then
            local buildData = loadBuildFromFile(farmSettings.autoBuildFile)
            if buildData and next(buildData) then
                local _, placedIds = pasteBuild(buildData)
                if placedIds then
                    recentlyPlacedBlocks = {}
                    local cnt = 0
                    for _, blk in pairs(placedIds) do
                        if type(blk) == "userdata" and blk:FindFirstChild("PPart") then
                            recentlyPlacedBlocks[blk] = true
                            cnt = cnt + 1
                        end
                    end
                end
            end
        end
    end)
end)

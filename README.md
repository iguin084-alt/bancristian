-- ============================================================
-- XENON ESP + AIMBOT — GUI Própria
-- ============================================================
do
    if _G.XenonThreads then for _,t in ipairs(_G.XenonThreads) do pcall(task.cancel,t) end end
    _G.XenonThreads={}
    if _G.XenonConnections then for _,c in ipairs(_G.XenonConnections) do pcall(function() c:Disconnect() end) end end
    _G.XenonConnections={}
    local guiNames={"XenonESPHub","XenonRingGui"}
    local function lg(p) if not p then return end; for _,n in ipairs(guiNames) do local o=p:FindFirstChild(n); if o then pcall(function() o:Destroy() end) end end end
    pcall(function() lg(game:GetService("CoreGui")) end)
    pcall(function() if typeof(gethui)=="function" then lg(gethui()) end end)
    pcall(function() local pg=game:GetService("Players").LocalPlayer:FindFirstChildOfClass("PlayerGui"); if pg then lg(pg) end end)
    _G.Aimbot=false; _G.ESP=false; _G.Tracer=false
    if _G._XenonTracerLines then for _,ln in pairs(_G._XenonTracerLines) do pcall(function() ln:Remove() end) end; _G._XenonTracerLines=nil end
    _G._XenonTracerExecID=nil
end

-- Serviços
local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS        = game:GetService("UserInputService")
local TweenSvc   = game:GetService("TweenService")
local LP         = Players.LocalPlayer or Players:GetPropertyChangedSignal("LocalPlayer"):Wait() and Players.LocalPlayer
local Camera     = workspace.CurrentCamera or workspace:WaitForChild("Camera")

_G.AIMBOT_RADIUS = 250
_G.AIMBOT_SMOOTH = 0.25
_G.ModoMobile    = false   -- mira fixa no centro + sliders touch

local function RegThread(t) table.insert(_G.XenonThreads,t); return t end
local function RegConn(c) table.insert(_G.XenonConnections,c); return c end

-- ============================================================
-- GUI PRÓPRIA — janela arrastável com ESP + Aimbot
-- ============================================================
local _guiParent
pcall(function() _guiParent=game:GetService("CoreGui") end)
if not _guiParent then _guiParent=LP:WaitForChild("PlayerGui") end

local sg = Instance.new("ScreenGui")
sg.Name = "XenonESPHub"
sg.ResetOnSpawn = false
sg.IgnoreGuiInset = true
sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
pcall(function() sg.Parent = game:GetService("CoreGui") end)
if not sg.Parent then sg.Parent = LP:WaitForChild("PlayerGui") end

-- Janela principal
local win = Instance.new("Frame", sg)
win.Name = "Window"
win.Size = UDim2.new(0, 240, 0, 300)
win.Position = UDim2.new(0, 60, 0, 80)
win.BackgroundColor3 = Color3.fromRGB(14, 14, 20)
win.BorderSizePixel = 0
win.ClipsDescendants = true
Instance.new("UICorner", win).CornerRadius = UDim.new(0, 10)
local winStroke = Instance.new("UIStroke", win)
winStroke.Color = Color3.fromRGB(100, 60, 200)
winStroke.Thickness = 1.5

-- Botão flutuante (aparece quando a janela está fechada)
local fabBtn = Instance.new("TextButton", sg)
fabBtn.Name = "FABButton"
fabBtn.Size = UDim2.new(0, 48, 0, 48)
fabBtn.Position = UDim2.new(1, -62, 0.5, -24)
fabBtn.BackgroundColor3 = Color3.fromRGB(70, 40, 150)
fabBtn.BorderSizePixel = 0
fabBtn.Text = "⚡"
fabBtn.Font = Enum.Font.GothamBold
fabBtn.TextSize = 20
fabBtn.TextColor3 = Color3.new(1,1,1)
fabBtn.ZIndex = 20
fabBtn.Visible = false
Instance.new("UICorner", fabBtn).CornerRadius = UDim.new(1, 0)
local fabStroke = Instance.new("UIStroke", fabBtn)
fabStroke.Color = Color3.fromRGB(140, 100, 255)
fabStroke.Thickness = 2

-- Arrastar o FAB
do
    local drag2, ds2, sp2 = false, nil, nil
    local moved2 = false
    fabBtn.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag2 = true; moved2 = false
            ds2 = i.Position; sp2 = fabBtn.Position
        end
    end)
    fabBtn.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag2 = false
            if not moved2 then
                win.Visible = true; fabBtn.Visible = false
            end
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if drag2 and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - ds2
            if math.abs(d.X) > 5 or math.abs(d.Y) > 5 then moved2 = true end
            fabBtn.Position = UDim2.new(sp2.X.Scale, sp2.X.Offset+d.X, sp2.Y.Scale, sp2.Y.Offset+d.Y)
        end
    end)
end

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame", win)
titleBar.Size = UDim2.new(1, 0, 0, 36)
titleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
titleBar.BorderSizePixel = 0
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 10)

local titleLabel = Instance.new("TextLabel", titleBar)
titleLabel.Size = UDim2.new(1, -40, 1, 0)
titleLabel.Position = UDim2.new(0, 14, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ Xenon  ESP + Aimbot"
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 13
titleLabel.TextColor3 = Color3.fromRGB(180, 140, 255)
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Botão fechar
local closeBtn = Instance.new("TextButton", titleBar)
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -34, 0, 4)
closeBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 13
closeBtn.TextColor3 = Color3.new(1,1,1)
closeBtn.BorderSizePixel = 0
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
closeBtn.MouseButton1Click:Connect(function()
    win.Visible = false
    fabBtn.Visible = true
end)

-- Botão de Lock de alvo (só visível no modo mobile)
local lockBtn = Instance.new("TextButton", sg)
lockBtn.Name = "LockBtn"
lockBtn.Size = UDim2.new(0, 56, 0, 56)
lockBtn.Position = UDim2.new(1, -62, 0.5, 40)
lockBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
lockBtn.BorderSizePixel = 0
lockBtn.Text = "🔓"
lockBtn.Font = Enum.Font.GothamBold
lockBtn.TextSize = 22
lockBtn.TextColor3 = Color3.new(1,1,1)
lockBtn.ZIndex = 20
lockBtn.Visible = false
Instance.new("UICorner", lockBtn).CornerRadius = UDim.new(1, 0)
local lockStroke = Instance.new("UIStroke", lockBtn)
lockStroke.Color = Color3.fromRGB(100, 100, 140)
lockStroke.Thickness = 2
_G._Xenon_LockBtn = lockBtn
_G._AlvoTravado = nil
_G._AlvoTravadoPlayer = nil

-- Arrastar lock btn
do
    local drag3, ds3, sp3, moved3 = false, nil, nil, false
    lockBtn.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag3 = true; moved3 = false
            ds3 = i.Position; sp3 = lockBtn.Position
        end
    end)
    lockBtn.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag3 = false
            if not moved3 then
                -- Toggle lock
                if _G._AlvoTravado then
                    -- Destravar
                    _G._AlvoTravado = nil
                    _G._AlvoTravadoPlayer = nil
                    lockBtn.Text = "🔓"
                    lockBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
                    lockStroke.Color = Color3.fromRGB(100, 100, 140)
                else
                    -- Travar no alvo atual (sinaliza pro aimbot travar)
                    _G._PedindoLock = true
                    lockBtn.Text = "🔒"
                    lockBtn.BackgroundColor3 = Color3.fromRGB(160, 30, 30)
                    lockStroke.Color = Color3.fromRGB(255, 80, 80)
                end
            end
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if drag3 and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - ds3
            if math.abs(d.X) > 5 or math.abs(d.Y) > 5 then moved3 = true end
            lockBtn.Position = UDim2.new(sp3.X.Scale, sp3.X.Offset+d.X, sp3.Y.Scale, sp3.Y.Offset+d.Y)
        end
    end)
end
UIS.InputBegan:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.K then
        win.Visible = not win.Visible
        fabBtn.Visible = not win.Visible
    end
end)

-- Arrastar janela (mouse + touch)
do
    local drag, dragStart, startPos = false, nil, nil
    local touchDragged = false
    titleBar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag = true; touchDragged = false
            dragStart = i.Position; startPos = win.Position
        end
    end)
    titleBar.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag = false
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - dragStart
            win.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+d.X, startPos.Y.Scale, startPos.Y.Offset+d.Y)
        end
    end)
end

-- Container de conteúdo com scroll
local content = Instance.new("ScrollingFrame", win)
content.Size = UDim2.new(1, 0, 1, -36)
content.Position = UDim2.new(0, 0, 0, 36)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.ScrollBarThickness = 3
content.ScrollBarImageColor3 = Color3.fromRGB(100, 60, 200)
content.CanvasSize = UDim2.new(0, 0, 0, 0)
content.AutomaticCanvasSize = Enum.AutomaticSize.Y

local layout = Instance.new("UIListLayout", content)
layout.Padding = UDim.new(0, 6)
layout.SortOrder = Enum.SortOrder.LayoutOrder
local pad = Instance.new("UIPadding", content)
pad.PaddingLeft = UDim.new(0, 10)
pad.PaddingRight = UDim.new(0, 10)
pad.PaddingTop = UDim.new(0, 10)
pad.PaddingBottom = UDim.new(0, 10)

-- Helpers de UI
local function makeSection(title, order)
    local f = Instance.new("Frame", content)
    f.Size = UDim2.new(1, 0, 0, 22)
    f.BackgroundColor3 = Color3.fromRGB(30, 20, 50)
    f.BorderSizePixel = 0
    f.LayoutOrder = order
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
    local lbl = Instance.new("TextLabel", f)
    lbl.Size = UDim2.new(1, -10, 1, 0)
    lbl.Position = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = "── "..title.." ──"
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 11
    lbl.TextColor3 = Color3.fromRGB(120, 80, 220)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    return f
end

local function makeToggle(labelText, order, default, onChange)
    local row = Instance.new("Frame", content)
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundColor3 = Color3.fromRGB(22, 22, 32)
    row.BorderSizePixel = 0
    row.LayoutOrder = order
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)

    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -60, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = labelText
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 12
    lbl.TextColor3 = Color3.fromRGB(210, 210, 220)
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local togBg = Instance.new("Frame", row)
    togBg.Size = UDim2.new(0, 42, 0, 22)
    togBg.Position = UDim2.new(1, -52, 0.5, -11)
    togBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    togBg.BorderSizePixel = 0
    Instance.new("UICorner", togBg).CornerRadius = UDim.new(1, 0)

    local togCircle = Instance.new("Frame", togBg)
    togCircle.Size = UDim2.new(0, 16, 0, 16)
    togCircle.Position = UDim2.new(0, 3, 0.5, -8)
    togCircle.BackgroundColor3 = Color3.fromRGB(150, 150, 160)
    togCircle.BorderSizePixel = 0
    Instance.new("UICorner", togCircle).CornerRadius = UDim.new(1, 0)

    local on = default or false
    local tweenOn  = TweenSvc:Create(togCircle, TweenInfo.new(0.15), {Position=UDim2.new(0,23,0.5,-8), BackgroundColor3=Color3.fromRGB(140,100,255)})
    local tweenOff = TweenSvc:Create(togCircle, TweenInfo.new(0.15), {Position=UDim2.new(0,3,0.5,-8),  BackgroundColor3=Color3.fromRGB(150,150,160)})
    local tweenBgOn  = TweenSvc:Create(togBg, TweenInfo.new(0.15), {BackgroundColor3=Color3.fromRGB(80,50,150)})
    local tweenBgOff = TweenSvc:Create(togBg, TweenInfo.new(0.15), {BackgroundColor3=Color3.fromRGB(50,50,60)})

    local function setState(v)
        on = v
        if on then tweenOn:Play(); tweenBgOn:Play() else tweenOff:Play(); tweenBgOff:Play() end
        if onChange then onChange(on) end
    end
    setState(on)

    local btn = Instance.new("TextButton", row)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.MouseButton1Click:Connect(function() setState(not on) end)

    return row, function(v) setState(v) end
end

local function makeSlider(labelText, order, min, max, default, onChange)
    local row = Instance.new("Frame", content)
    row.Size = UDim2.new(1, 0, 0, 50)
    row.BackgroundColor3 = Color3.fromRGB(22, 22, 32)
    row.BorderSizePixel = 0
    row.LayoutOrder = order
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)

    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -60, 0, 20)
    lbl.Position = UDim2.new(0, 12, 0, 6)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 12
    lbl.TextColor3 = Color3.fromRGB(210, 210, 220)
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local valLbl = Instance.new("TextLabel", row)
    valLbl.Size = UDim2.new(0, 50, 0, 20)
    valLbl.Position = UDim2.new(1, -58, 0, 6)
    valLbl.BackgroundTransparency = 1
    valLbl.Font = Enum.Font.GothamBold
    valLbl.TextSize = 12
    valLbl.TextColor3 = Color3.fromRGB(140, 100, 255)
    valLbl.TextXAlignment = Enum.TextXAlignment.Right

    local track = Instance.new("Frame", row)
    track.Size = UDim2.new(1, -24, 0, 6)
    track.Position = UDim2.new(0, 12, 0, 34)
    track.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(100, 60, 200)
    fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local val = default
    local function setVal(v)
        v = math.clamp(v, min, max)
        val = v
        local pct = (v - min) / (max - min)
        fill.Size = UDim2.new(pct, 0, 1, 0)
        local isInt = (math.floor(min)==min and math.floor(max)==max and math.floor(default)==default)
        valLbl.Text = isInt and tostring(math.floor(v)) or string.format("%.2f", v)
        lbl.Text = labelText
        if onChange then onChange(v) end
    end
    setVal(val)

    local dragging = false
    local function calcVal(mx)
        local abs = track.AbsolutePosition
        local sz  = track.AbsoluteSize
        local pct = math.clamp((mx - abs.X) / sz.X, 0, 1)
        setVal(min + pct * (max - min))
    end

    local btn = Instance.new("TextButton", track)
    btn.Size = UDim2.new(1, 0, 3, -track.Size.Y.Offset); btn.Position = UDim2.new(0,0,-1,0)
    btn.BackgroundTransparency = 1; btn.Text = ""
    btn.InputBegan:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            dragging=true; calcVal(i.Position.X)
        end
    end)
    btn.InputEnded:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            dragging=false
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if dragging and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then
            calcVal(i.Position.X)
        end
    end)
    return row
end

-- ============================================================
-- SEÇÃO ESP
-- ============================================================
makeSection("ESP", 1)
makeToggle("ESP (nomes + HP)", 2, false, function(v)
    _G.ESP = v
    if not v and _G._Xenon_LimparTodoESP then _G._Xenon_LimparTodoESP() end
end)
makeToggle("Tracer (linhas)", 3, false, function(v)
    _G.Tracer = v
    if not v and _G._Xenon_LimparTodosTracers then _G._Xenon_LimparTodosTracers() end
end)
makeToggle("Ignorar time aliado", 4, false, function(v)
    if _G._Xenon_SetEspIgnorarTime then _G._Xenon_SetEspIgnorarTime(v) end
end)
makeToggle("Mostrar patente", 5, true, function(v)
    if _G._Xenon_SetEspPatente then _G._Xenon_SetEspPatente(v) end
end)
makeToggle("Só tracer (sem nome/HP)", 6, false, function(v)
    if _G._Xenon_SetEspSoCaixas then _G._Xenon_SetEspSoCaixas(v) end
end)

-- ============================================================
-- SEÇÃO AIMBOT
-- ============================================================
makeSection("Item ESP", 8)
makeToggle("🔪 Item ESP (Knife=vermelho / Gun=azul)", 9, false, function(v) _G.ItemESP = v end)
makeToggle("🎯 Só mirar em quem tem Knife", 9, false, function(v) _G.AimbotSoKnife = v end)

makeSection("Aimbot", 10)
makeToggle("Aimbot", 11, false, function(v) _G.Aimbot = v end)
makeToggle("Auto (sempre ativo)", 12, false, function(v) _G.AimbotAuto = v end)
makeToggle("📱 Modo Mobile (mira centro)", 13, false, function(v)
    _G.ModoMobile = v
    -- Mostra/esconde botão de lock
    if _G._Xenon_LockBtn then
        _G._Xenon_LockBtn.Visible = v
    end
    -- Destrava ao desligar modo mobile
    if not v then
        _G._AlvoTravado = nil
        _G._AlvoTravadoPlayer = nil
    end
end)
makeSlider("FOV", 14, 10, 300, 100, function(v) _G.AIMBOT_RADIUS = v end)
makeSlider("Suavidade", 15, 0.05, 1.0, 0.25, function(v) _G.AIMBOT_SMOOTH = v end)
makeToggle("⚡ Snap (sem delay)", 16, false, function(v) _G.AimbotSnap = v end)
makeToggle("🖱️ Auto Fire (PC)", 17, false, function(v) _G.AutoFire = v end)
makeToggle("🎯 Auto Fire Smart (LOS)", 18, false, function(v) _G.AutoFireSmart = v end)

-- Trigger key
do
    local triggers = {"mouse1","mouse2","key (RShift)"}
    local trigIdx = 1
    local row = Instance.new("Frame", content)
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundColor3 = Color3.fromRGB(22,22,32)
    row.BorderSizePixel = 0
    row.LayoutOrder = 19
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)
    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(0.55, 0, 1, 0); lbl.Position = UDim2.new(0,12,0,0)
    lbl.BackgroundTransparency=1; lbl.Text="Trigger"; lbl.Font=Enum.Font.Gotham
    lbl.TextSize=12; lbl.TextColor3=Color3.fromRGB(210,210,220); lbl.TextXAlignment=Enum.TextXAlignment.Left
    local cycleBtn = Instance.new("TextButton", row)
    cycleBtn.Size = UDim2.new(0, 110, 0, 24); cycleBtn.Position = UDim2.new(1,-118,0.5,-12)
    cycleBtn.BackgroundColor3 = Color3.fromRGB(50,35,90); cycleBtn.BorderSizePixel=0
    cycleBtn.Font=Enum.Font.GothamBold; cycleBtn.TextSize=11; cycleBtn.TextColor3=Color3.fromRGB(200,180,255)
    cycleBtn.Text = triggers[trigIdx]
    Instance.new("UICorner",cycleBtn).CornerRadius=UDim.new(0,6)
    cycleBtn.MouseButton1Click:Connect(function()
        trigIdx = (trigIdx % #triggers) + 1
        cycleBtn.Text = triggers[trigIdx]
        local map = {"mouse1","mouse2","key"}
        _G._AimbotTrigger = map[trigIdx]
    end)
end

-- ============================================================
-- LISTA UNIFICADA DE PLAYERS (ESP + Aimbot)
-- ============================================================
makeSection("Players", 20)
do
    -- Sets compartilhados
    local _alvosSet = {}          -- key = name:lower() -> true  (aimbot)
    local _espMarcados = {}       -- key = userId -> true         (esp)
    _G._Xenon_AlvosSet = _alvosSet

    -- Modos
    local espSoMarcados   = false
    local aimbotSoFixados = false  -- já controlado pelo _alvosSet vazio ou não

    -- ── Barra de busca ──────────────────────────────────────────
    local buscaRow = Instance.new("Frame", content)
    buscaRow.Size = UDim2.new(1, 0, 0, 30)
    buscaRow.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
    buscaRow.BorderSizePixel = 0
    buscaRow.LayoutOrder = 21
    Instance.new("UICorner", buscaRow).CornerRadius = UDim.new(0, 8)

    local searchBox = Instance.new("TextBox", buscaRow)
    searchBox.Size = UDim2.new(1, -10, 0, 22)
    searchBox.Position = UDim2.new(0, 5, 0.5, -11)
    searchBox.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    searchBox.BorderSizePixel = 0
    searchBox.Font = Enum.Font.Gotham
    searchBox.TextSize = 11
    searchBox.TextColor3 = Color3.fromRGB(220, 220, 230)
    searchBox.PlaceholderText = "🔍 Buscar players (ex: a)"
    searchBox.PlaceholderColor3 = Color3.fromRGB(90, 90, 110)
    searchBox.Text = ""
    searchBox.ClearTextOnFocus = false
    Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 5)
    local sbPad = Instance.new("UIPadding", searchBox)
    sbPad.PaddingLeft = UDim.new(0, 6)

    -- ── Toggles de modo ────────────────────────────────────────
    local function makeMiniToggle(parent, text, order, color, onToggle)
        local f = Instance.new("Frame", parent)
        f.Size = UDim2.new(0.5, -3, 0, 26)
        f.BackgroundColor3 = Color3.fromRGB(22, 22, 32)
        f.BorderSizePixel = 0
        f.LayoutOrder = order
        Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
        local btn = Instance.new("TextButton", f)
        btn.Size = UDim2.new(1, 0, 1, 0)
        btn.BackgroundTransparency = 1
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 10
        btn.TextColor3 = Color3.fromRGB(160, 160, 180)
        btn.Text = "○ " .. text
        local ativo = false
        btn.MouseButton1Click:Connect(function()
            ativo = not ativo
            btn.Text = (ativo and "● " or "○ ") .. text
            btn.TextColor3 = ativo and color or Color3.fromRGB(160, 160, 180)
            f.BackgroundColor3 = ativo and Color3.fromRGB(25, 35, 50) or Color3.fromRGB(22, 22, 32)
            onToggle(ativo)
        end)
        return f
    end

    local modoRow = Instance.new("Frame", content)
    modoRow.Size = UDim2.new(1, 0, 0, 26)
    modoRow.BackgroundTransparency = 1
    modoRow.BorderSizePixel = 0
    modoRow.LayoutOrder = 22
    local modoLayout = Instance.new("UIListLayout", modoRow)
    modoLayout.FillDirection = Enum.FillDirection.Horizontal
    modoLayout.Padding = UDim.new(0, 6)

    makeMiniToggle(modoRow, "ESP só marcados", 1, Color3.fromRGB(100, 180, 255), function(v)
        espSoMarcados = v
        if _G._Xenon_SetEspListaBranca then _G._Xenon_SetEspListaBranca(v) end
    end)
    makeMiniToggle(modoRow, "Aimbot só fixados", 2, Color3.fromRGB(100, 220, 100), function(v)
        -- Quando desliga, limpa o set pra mira em todos
        if not v then
            for k in pairs(_alvosSet) do _alvosSet[k] = nil end
        end
        aimbotSoFixados = v
    end)

    -- ── Lista de players ────────────────────────────────────────
    local lista = Instance.new("Frame", content)
    lista.Size = UDim2.new(1, 0, 0, 0)
    lista.AutomaticSize = Enum.AutomaticSize.Y
    lista.BackgroundTransparency = 1
    lista.LayoutOrder = 23
    Instance.new("UIListLayout", lista).Padding = UDim.new(0, 3)

    local function refreshLista(filtro)
        filtro = (filtro or ""):lower():match("^%s*(.-)%s*$")
        for _, c in ipairs(lista:GetChildren()) do
            if not c:IsA("UIListLayout") then c:Destroy() end
        end

        local todosPlayers = Players:GetPlayers()
        local alguem = false

        for _, p in ipairs(todosPlayers) do
            if p ~= Players.LocalPlayer then
                local nome = p.Name
                local nomeLow = nome:lower()
                local dispLow = p.DisplayName:lower()

                -- Filtro de busca por prefixo
                if filtro ~= "" and not (nomeLow:sub(1, #filtro) == filtro or dispLow:sub(1, #filtro) == filtro) then
                    continue
                end

                alguem = true
                local espOn   = _espMarcados[p.UserId] == true
                local aimbOn  = _alvosSet[nomeLow] == true

                -- Cor de fundo baseada no estado
                local bgColor
                if espOn and aimbOn then bgColor = Color3.fromRGB(30, 40, 55)
                elseif espOn then       bgColor = Color3.fromRGB(20, 35, 50)
                elseif aimbOn then      bgColor = Color3.fromRGB(20, 45, 25)
                else                    bgColor = Color3.fromRGB(22, 22, 30) end

                local row = Instance.new("Frame", lista)
                row.Size = UDim2.new(1, 0, 0, 34)
                row.BackgroundColor3 = bgColor
                row.BorderSizePixel = 0
                Instance.new("UICorner", row).CornerRadius = UDim.new(0, 7)

                -- Nome
                local lbl = Instance.new("TextLabel", row)
                lbl.Size = UDim2.new(1, -130, 1, 0)
                lbl.Position = UDim2.new(0, 10, 0, 0)
                lbl.BackgroundTransparency = 1
                lbl.Font = Enum.Font.Gotham
                lbl.TextSize = 11
                lbl.TextColor3 = Color3.fromRGB(210, 210, 220)
                lbl.TextXAlignment = Enum.TextXAlignment.Left
                lbl.Text = p.DisplayName ~= nome and (p.DisplayName .. "\n(" .. nome .. ")") or nome
                lbl.TextWrapped = true

                -- Botão ESP
                local espBtn = Instance.new("TextButton", row)
                espBtn.Size = UDim2.new(0, 44, 0, 24)
                espBtn.Position = UDim2.new(1, -130, 0.5, -12)
                espBtn.BorderSizePixel = 0
                espBtn.Font = Enum.Font.GothamBold
                espBtn.TextSize = 9
                espBtn.TextColor3 = Color3.new(1,1,1)
                Instance.new("UICorner", espBtn).CornerRadius = UDim.new(0, 5)
                espBtn.Text = espOn and "ESP✓" or "ESP"
                espBtn.BackgroundColor3 = espOn and Color3.fromRGB(30, 90, 150) or Color3.fromRGB(45, 45, 60)

                -- Botão Aimbot
                local aimBtn = Instance.new("TextButton", row)
                aimBtn.Size = UDim2.new(0, 44, 0, 24)
                aimBtn.Position = UDim2.new(1, -80, 0.5, -12)
                aimBtn.BorderSizePixel = 0
                aimBtn.Font = Enum.Font.GothamBold
                aimBtn.TextSize = 9
                aimBtn.TextColor3 = Color3.new(1,1,1)
                Instance.new("UICorner", aimBtn).CornerRadius = UDim.new(0, 5)
                aimBtn.Text = aimbOn and "AIM✓" or "AIM"
                aimBtn.BackgroundColor3 = aimbOn and Color3.fromRGB(35, 110, 50) or Color3.fromRGB(45, 45, 60)

                -- Botão ambos
                local ambosBtn = Instance.new("TextButton", row)
                ambosBtn.Size = UDim2.new(0, 26, 0, 24)
                ambosBtn.Position = UDim2.new(1, -30, 0.5, -12)
                ambosBtn.BorderSizePixel = 0
                ambosBtn.Font = Enum.Font.GothamBold
                ambosBtn.TextSize = 9
                ambosBtn.TextColor3 = Color3.new(1,1,1)
                Instance.new("UICorner", ambosBtn).CornerRadius = UDim.new(0, 5)
                local ambosOn = espOn and aimbOn
                ambosBtn.Text = ambosOn and "✕" or "✓"
                ambosBtn.BackgroundColor3 = ambosOn and Color3.fromRGB(130, 30, 30) or Color3.fromRGB(80, 55, 20)

                local uid = p.UserId
                local key = nomeLow

                espBtn.MouseButton1Click:Connect(function()
                    if _espMarcados[uid] then
                        _espMarcados[uid] = nil
                        if _G._Xenon_SetEspJogadorSelecionado then _G._Xenon_SetEspJogadorSelecionado(uid, false) end
                    else
                        _espMarcados[uid] = true
                        if _G._Xenon_SetEspJogadorSelecionado then _G._Xenon_SetEspJogadorSelecionado(uid, true) end
                    end
                    refreshLista(searchBox.Text)
                end)

                aimBtn.MouseButton1Click:Connect(function()
                    if _alvosSet[key] then _alvosSet[key] = nil
                    else _alvosSet[key] = true end
                    refreshLista(searchBox.Text)
                end)

                ambosBtn.MouseButton1Click:Connect(function()
                    if _espMarcados[uid] and _alvosSet[key] then
                        -- remove ambos
                        _espMarcados[uid] = nil; _alvosSet[key] = nil
                        if _G._Xenon_SetEspJogadorSelecionado then _G._Xenon_SetEspJogadorSelecionado(uid, false) end
                    else
                        -- adiciona ambos
                        _espMarcados[uid] = true; _alvosSet[key] = true
                        if _G._Xenon_SetEspJogadorSelecionado then _G._Xenon_SetEspJogadorSelecionado(uid, true) end
                    end
                    refreshLista(searchBox.Text)
                end)
            end
        end

        if not alguem then
            local vazio = Instance.new("TextLabel", lista)
            vazio.Size = UDim2.new(1, 0, 0, 24)
            vazio.BackgroundTransparency = 1
            vazio.Font = Enum.Font.Gotham
            vazio.TextSize = 11
            vazio.TextColor3 = Color3.fromRGB(100, 100, 110)
            vazio.Text = filtro ~= "" and "Nenhum player começa com '"..filtro.."'" or "Nenhum jogador no servidor"
        end
    end

    -- Busca em tempo real ao digitar
    searchBox:GetPropertyChangedSignal("Text"):Connect(function()
        refreshLista(searchBox.Text)
    end)

    -- Botão atualizar
    local refreshBtn = Instance.new("TextButton", content)
    refreshBtn.Size = UDim2.new(1, 0, 0, 26)
    refreshBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 48)
    refreshBtn.BorderSizePixel = 0
    refreshBtn.LayoutOrder = 24
    refreshBtn.Font = Enum.Font.GothamBold
    refreshBtn.TextSize = 11
    refreshBtn.TextColor3 = Color3.fromRGB(150, 150, 190)
    refreshBtn.Text = "↻  Atualizar lista"
    Instance.new("UICorner", refreshBtn).CornerRadius = UDim.new(0, 6)
    refreshBtn.MouseButton1Click:Connect(function() refreshLista(searchBox.Text) end)

    Players.PlayerAdded:Connect(function() refreshLista(searchBox.Text) end)
    Players.PlayerRemoving:Connect(function(p)
        _espMarcados[p.UserId] = nil
        _alvosSet[p.Name:lower()] = nil
        if _G._Xenon_SetEspJogadorSelecionado then _G._Xenon_SetEspJogadorSelecionado(p.UserId, false) end
        refreshLista(searchBox.Text)
    end)

    refreshLista()
end

-- ============================================================
-- ESP / TRACER (lógica)
-- ============================================================
;(function()
local tracerLines={}; _G._XenonTracerLines=tracerLines
local espCache={}
local _espIgnorarTime=false
local _espTimesIgnorados={}
local _espMostrarPatente=true
local _espJogadoresSelecionados={}
local _espListaBranca=false
local _espSoCaixas=false  -- true = só caixas, sem tracers nem nome
local _ignoradosSet={}    -- set de nomes ignorados no ESP e Aimbot
_G._Xenon_IgnoradosSet=_ignoradosSet
_G._Xenon_AdicionarIgnorado=function(nome)
    _ignoradosSet[nome:lower()]=true
    -- Limpa ESP do player se estiver visível
    for _,p in pairs(Players:GetPlayers()) do
        if p.Name:lower()==nome:lower() or p.DisplayName:lower()==nome:lower() then
            LimparESP(p)
        end
    end
end
_G._Xenon_RemoverIgnorado=function(nome) _ignoradosSet[nome:lower()]=nil end

local function DestruirESPDrawings(d)
    if not d then return end
end

local function LimparTracer(p) local ln=tracerLines[p]; if ln then pcall(function() ln.Visible=false end); pcall(function() ln:Remove() end) end; tracerLines[p]=nil end
local function LimparTodosTracers() for p,ln in pairs(tracerLines) do pcall(function() ln:Remove() end); tracerLines[p]=nil end end
local function LimparESP(p)
    local c2=espCache[p]
    if c2 and c2.drawings then DestruirESPDrawings(c2.drawings) end
    espCache[p]=nil
    LimparTracer(p)
end
local function LimparTodoESP() for _,p in pairs(Players:GetPlayers()) do if p~=LP then LimparESP(p) end end end
local function GetTeamColor(p) if p.Team then local tc=p.TeamColor; return Color3.fromRGB(tc.r*255,tc.g*255,tc.b*255) end; return Color3.fromRGB(255,80,80) end
local function GetHP(p) if p.Character then local h=p.Character:FindFirstChildOfClass("Humanoid"); if h then return h.Health,h.MaxHealth end end; return 0,100 end
local function GetDistancia(p) local myChar=LP.Character; local tChar=p.Character; if not myChar or not tChar then return 0 end; local myRoot=myChar:FindFirstChild("HumanoidRootPart"); local tRoot=tChar:FindFirstChild("HumanoidRootPart"); if not myRoot or not tRoot then return 0 end; return math.floor((myRoot.Position-tRoot.Position).Magnitude) end
local function GetPatente(p) local dn=p.DisplayName or ""; local m=dn:match("%[(.-)%]") or dn:match("%((.-)%)"); return m or "" end
local function DeveMostrarESP(p)
    if not _G.ESP then return false end
    -- Checagem de ignorados
    if _ignoradosSet[p.Name:lower()] or _ignoradosSet[p.DisplayName:lower()] then return false end
    -- Lista branca: se ativa, só mostra players marcados
    if _espListaBranca then
        if not _espJogadoresSelecionados[p.UserId] then return false end
    end
    -- Filtro de time
    if not _espIgnorarTime then local lpTeam=LP.Team; if lpTeam and p.Team==lpTeam then return false end end
    return true
end

local _espPlayerCache={}
local function RefreshEspCache() _espPlayerCache={}; for _,p in pairs(Players:GetPlayers()) do if p~=LP then _espPlayerCache[#_espPlayerCache+1]=p end end end
RefreshEspCache()
RegConn(Players.PlayerAdded:Connect(RefreshEspCache))
RegConn(Players.PlayerRemoving:Connect(function() RefreshEspCache() end))

local _visaoParams=RaycastParams.new(); _visaoParams.FilterType=Enum.RaycastFilterType.Exclude
local function TemVisao(orig,alvoPos) local ig={}; if LP.Character then ig[1]=LP.Character end; for _,p in ipairs(_espPlayerCache) do if p.Character then ig[#ig+1]=p.Character end end; _visaoParams.FilterDescendantsInstances=ig; return workspace:Raycast(orig,alvoPos-orig,_visaoParams)==nil end
local function CriarLinha() local ln=Drawing.new("Line"); ln.Thickness=1.5; ln.Color=Color3.new(1,1,1); ln.Visible=false; ln.ZIndex=5; return ln end
RegConn(Players.PlayerRemoving:Connect(function(p) LimparESP(p); RefreshEspCache() end))

-- Cria os objetos Drawing para um player
local function CriarESPDrawings()
    local d = {}
    d.linhas = {}  -- sem caixa


    return d
end

-- Retorna os 8 cantos do bounding box do personagem em screen space
local function GetBoundingBox(char)
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    -- Aproxima com pontos do corpo
    local partes = {"Head","HumanoidRootPart","LeftFoot","RightFoot","LeftHand","RightHand"}
    local minX,minY,maxX,maxY = math.huge,math.huge,-math.huge,-math.huge
    local qualquerVisivel = false
    for _,nome in ipairs(partes) do
        local p = char:FindFirstChild(nome)
        if p and p:IsA("BasePart") then
            local sp, _ = Camera:WorldToViewportPoint(p.Position)
            if sp.Z > 0 then  -- na frente da câmera
                qualquerVisivel = true
                if sp.X < minX then minX = sp.X end
                if sp.Y < minY then minY = sp.Y end
                if sp.X > maxX then maxX = sp.X end
                if sp.Y > maxY then maxY = sp.Y end
            end
        end
    end
    if not qualquerVisivel then return nil end
    -- Margem pequena
    local mg = 4
    return minX-mg, minY-mg, maxX+mg, maxY+mg
end

local function AtualizarESP(p)
    if not DeveMostrarESP(p) then LimparESP(p); return end
    if not p.Character then LimparESP(p); return end
    local char=p.Character
    local hum=char:FindFirstChildOfClass("Humanoid")
    if hum and (hum.Health<=0 or hum:GetState()==Enum.HumanoidStateType.Dead) then LimparESP(p); return end
    local head=char:FindFirstChild("Head"); if not head then LimparESP(p); return end

    local c2=espCache[p]
    if not c2 then c2={drawings=CriarESPDrawings()}; espCache[p]=c2 end

    local x1,y1,x2,y2 = GetBoundingBox(char)
    if not x1 then
        -- fora de tela: esconde mas mantém
        local d=c2.drawings

    end

    local tc = GetTeamColor(p)
    local d  = c2.drawings


end

local _tracerExecID=tostring(os.clock()); _G._XenonTracerExecID=_tracerExecID
RegConn(RunService.RenderStepped:Connect(function()
    if _G._XenonTracerExecID~=_tracerExecID then for _,ln in pairs(tracerLines) do pcall(function() ln:Remove() end) end; return end
    if not _G.Tracer then LimparTodosTracers(); return end
    local myChar=LP.Character; local myHRP=myChar and myChar:FindFirstChild("HumanoidRootPart")
    local vp=Camera.ViewportSize; local origem=Vector2.new(vp.X/2,vp.Y)
    if myHRP then local sp,vis=Camera:WorldToViewportPoint(myHRP.Position); if vis then origem=Vector2.new(sp.X,sp.Y) end end
    local camPos=Camera.CFrame.Position
    for _,p in ipairs(_espPlayerCache) do
        if not DeveMostrarESP(p) or not p.Character then LimparTracer(p)
        else
            local hrp=p.Character:FindFirstChild("HumanoidRootPart")
            if not hrp then LimparTracer(p)
            else
                local sp2,vis2=Camera:WorldToViewportPoint(hrp.Position)
                if not vis2 then local ln=tracerLines[p]; if ln then ln.Visible=false end
                else
                    if not tracerLines[p] then tracerLines[p]=CriarLinha() end
                    local ln=tracerLines[p]
                    ln.Color=TemVisao(camPos,hrp.Position) and Color3.new(1,1,1) or Color3.fromRGB(220,50,50)
                    ln.From=origem; ln.To=Vector2.new(sp2.X,sp2.Y); ln.Visible=true
                end
            end
        end
    end
end))
local espHBThrottle=0
RegConn(RunService.Heartbeat:Connect(function(dt)
    if _G._XenonTracerExecID~=_tracerExecID then return end
    if not _G.ESP then return end
    espHBThrottle=espHBThrottle-dt; if espHBThrottle>0 then return end; espHBThrottle=0.12
    for _,p in ipairs(_espPlayerCache) do AtualizarESP(p) end
end))
RegConn(LP.CharacterAdded:Connect(function()
    for _,p in pairs(Players:GetPlayers()) do if p~=LP then LimparESP(p) end end
    LimparTodosTracers()
    -- Força rebuild do cache ao trocar de área (lobby→arena, etc.)
    task.wait(0.5)
    RefreshEspCache()
end))
for _,p in pairs(Players:GetPlayers()) do
    RegConn(p.CharacterRemoving:Connect(function() LimparESP(p) end))
    RegConn(p.CharacterAdded:Connect(function()
        task.wait(0.3)
        RefreshEspCache()
        LimparESP(p)
    end))
end
RegConn(Players.PlayerAdded:Connect(function(p)
    RegConn(p.CharacterRemoving:Connect(function() LimparESP(p) end))
    RegConn(p.CharacterAdded:Connect(function()
        task.wait(0.3)
        RefreshEspCache()
    end))
end))
-- Refresh periódico pro caso de teleporte sem CharacterAdded
RegConn(RunService.Heartbeat:Connect(function()
    -- a cada ~5s verifica se algum player novo entrou no cache
end))
task.spawn(function()
    while task.wait(4) do
        RefreshEspCache()
        for _,p in pairs(Players:GetPlayers()) do
            if p~=LP and not p.Character then LimparESP(p) end
        end
    end
end)
_G._Xenon_LimparTodoESP=LimparTodoESP
_G._Xenon_LimparTodosTracers=LimparTodosTracers
_G._Xenon_SetEspIgnorarTime=function(v) _espIgnorarTime=v end
_G._Xenon_SetEspPatente=function(v) _espMostrarPatente=v end
_G._Xenon_SetEspListaBranca=function(v) _espListaBranca=v end
_G._Xenon_SetEspJogadorSelecionado=function(userId, v)
    _espJogadoresSelecionados[userId] = v and true or nil
    -- Força reavaliação só do player afetado
    for _,p in pairs(Players:GetPlayers()) do
        if p.UserId == userId then
            LimparESP(p)
            break
        end
    end
end
_G._Xenon_GetEspListaBranca=function() return _espListaBranca end
_G._Xenon_GetEspJogadoresSelecionados=function() return _espJogadoresSelecionados end
_G._Xenon_EspSelecionados=_espJogadoresSelecionados
_G._Xenon_GetEspPlayerCache=function() return _espPlayerCache end
_G._Xenon_GetEspIgnorarTime=function() return _espIgnorarTime end
_G._Xenon_TemVisao=TemVisao
_G._Xenon_TimesIgnorados=_espTimesIgnorados
_G._Xenon_SetEspSoCaixas=function(v) _espSoCaixas=v end
end)()

-- ============================================================
-- ITEM ESP (Knife = vermelho / Gun = azul)
-- ============================================================
;(function()

local ITEM_COLORS = {
    Knife = Color3.fromRGB(220, 40,  40),
    Gun   = Color3.fromRGB(40,  120, 220),
}

-- box por player: { box = Drawing, label = Drawing, cor = Color3 }
local itemBoxes = {}
_G._Xenon_ItemBoxes = itemBoxes

local function criarItemBox(cor)
    local box   = Drawing.new("Square")
    box.Filled  = false
    box.Color   = cor
    box.Thickness = 2
    box.Visible = false

    local label = Drawing.new("Text")
    label.Color   = cor
    label.Size    = 13
    label.Outline = true
    label.Center  = true
    label.Visible = false

    return { box = box, label = label, cor = cor }
end

local function removerItemBox(p)
    local d = itemBoxes[p]
    if not d then return end
    pcall(function() d.box:Remove() end)
    pcall(function() d.label:Remove() end)
    itemBoxes[p] = nil
end

local function getToolNome(p)
    -- Verifica backpack e character (arma equipada fica no char)
    local containers = {}
    local bp = p:FindFirstChildOfClass("Backpack")
    if bp then table.insert(containers, bp) end
    if p.Character then table.insert(containers, p.Character) end

    for _, container in ipairs(containers) do
        for _, item in ipairs(container:GetChildren()) do
            if item:IsA("Tool") then
                for toolName in pairs(ITEM_COLORS) do
                    if item.Name == toolName then
                        return toolName
                    end
                end
            end
        end
    end
    return nil
end

-- Conexões por player pra detectar tool add/remove
local playerConns = {}

local function watchPlayer(p)
    if p == LP then return end

    playerConns[p] = playerConns[p] or {}

    local function onChildChanged(container)
        -- Reconnecta quando character muda
        local conns = playerConns[p]

        local function bindContainer(c)
            local addConn = c.ChildAdded:Connect(function(child)
                if not _G.ItemESP then return end
                if not child:IsA("Tool") then return end
                for toolName in pairs(ITEM_COLORS) do
                    if child.Name == toolName then
                        -- Remove ESP de todos os outros players que tinham essa mesma tool
                        for otherP, d in pairs(itemBoxes) do
                            if otherP ~= p and d.toolName == toolName then
                                removerItemBox(otherP)
                            end
                        end
                        -- Cria/atualiza box pra esse player
                        removerItemBox(p)
                        itemBoxes[p] = criarItemBox(ITEM_COLORS[toolName])
                        itemBoxes[p].toolName  = toolName
                        itemBoxes[p].labelText = toolName
                        break
                    end
                end
            end)
            local remConn = c.ChildRemoved:Connect(function(child)
                if not child:IsA("Tool") then return end
                -- Checa se é uma das tools monitoradas
                local eMonitorada = false
                for toolName in pairs(ITEM_COLORS) do
                    if child.Name == toolName then eMonitorada = true; break end
                end
                if not eMonitorada then return end
                -- Aguarda 1 frame — o Roblox move a tool entre containers antes de remover
                task.defer(function()
                    if not p or not p.Parent then removerItemBox(p); return end
                    local ainda = getToolNome(p)
                    if not ainda then
                        removerItemBox(p)
                    elseif itemBoxes[p] and itemBoxes[p].toolName ~= ainda then
                        -- Trocou de tool (ex: Knife -> Gun)
                        removerItemBox(p)
                        itemBoxes[p] = criarItemBox(ITEM_COLORS[ainda])
                        itemBoxes[p].toolName  = ainda
                        itemBoxes[p].labelText = ainda
                    end
                end)
            end)
            table.insert(conns, addConn)
            table.insert(conns, remConn)
        end

        local bp = p:FindFirstChildOfClass("Backpack")
        if bp then bindContainer(bp) end
        if p.Character then bindContainer(p.Character) end
    end

    -- Bind imediato
    onChildChanged()

    -- Quando equipa/desequipa o character muda
    local charConn = p.CharacterAdded:Connect(function(char)
        -- Desconecta conns antigas e reconecta
        for _, c in ipairs(playerConns[p] or {}) do pcall(function() c:Disconnect() end) end
        playerConns[p] = {}
        task.wait(0.1)
        onChildChanged()
    end)
    table.insert(playerConns[p], charConn)

    -- Verifica estado inicial
    local toolAtual = getToolNome(p)
    if toolAtual then
        -- Remove ESP de outros que já tinham essa tool
        for otherP, d in pairs(itemBoxes) do
            if otherP ~= p and d.toolName == toolAtual then
                removerItemBox(otherP)
            end
        end
        itemBoxes[p] = criarItemBox(ITEM_COLORS[toolAtual])
        itemBoxes[p].toolName  = toolAtual
        itemBoxes[p].labelText = toolAtual
    end
end

local function unwatchPlayer(p)
    for _, c in ipairs(playerConns[p] or {}) do pcall(function() c:Disconnect() end) end
    playerConns[p] = nil
    removerItemBox(p)
end

-- Inicia watch em todos os players existentes
for _, p in ipairs(Players:GetPlayers()) do watchPlayer(p) end
RegConn(Players.PlayerAdded:Connect(watchPlayer))
RegConn(Players.PlayerRemoving:Connect(unwatchPlayer))

-- Loop de renderização — atualiza box na tela
RegConn(RunService.RenderStepped:Connect(function()
    for p, d in pairs(itemBoxes) do
        if not _G.ItemESP then
            d.box.Visible   = false
            d.label.Visible = false
        elseif not p.Character or not p.Character:FindFirstChild("HumanoidRootPart") then
            d.box.Visible   = false
            d.label.Visible = false
        else
            local hrp = p.Character.HumanoidRootPart
            local head = p.Character:FindFirstChild("Head")

            -- Pega extremos do bounding box do personagem na tela
            local parts = { hrp }
            if head then table.insert(parts, head) end

            -- Calcula bounding box 2D
            local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
            local alguemVisivel = false

            -- Usa offset pra pegar topo e base do personagem
            local offsets = {
                Vector3.new(0,  3, 0),   -- topo (cabeça)
                Vector3.new(0, -3, 0),   -- base (pés)
                Vector3.new( 1.5, 0, 0),
                Vector3.new(-1.5, 0, 0),
            }

            for _, offset in ipairs(offsets) do
                local worldPos = hrp.Position + offset
                local scrPos, vis = Camera:WorldToViewportPoint(worldPos)
                if vis then
                    alguemVisivel = true
                    if scrPos.X < minX then minX = scrPos.X end
                    if scrPos.Y < minY then minY = scrPos.Y end
                    if scrPos.X > maxX then maxX = scrPos.X end
                    if scrPos.Y > maxY then maxY = scrPos.Y end
                end
            end

            if alguemVisivel then
                local w = maxX - minX
                local h = maxY - minY

                d.box.Size     = Vector2.new(w, h)
                d.box.Position = Vector2.new(minX, minY)
                d.box.Color    = d.cor
                d.box.Visible  = true

                d.label.Text     = (p.DisplayName or p.Name) .. " [" .. (d.labelText or "") .. "]"
                d.label.Position = Vector2.new((minX + maxX) / 2, minY - 16)
                d.label.Color    = d.cor
                d.label.Visible  = true
            else
                d.box.Visible   = false
                d.label.Visible = false
            end
        end
    end
end))

end)()
;(function()
local clicando=false
RegConn(UIS.InputBegan:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 then clicando=true end
    if i.UserInputType==Enum.UserInputType.MouseButton2 then clicando=true end
    if i.UserInputType==Enum.UserInputType.Touch then clicando=true end
end))
RegConn(UIS.InputEnded:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 then clicando=false end
    if i.UserInputType==Enum.UserInputType.MouseButton2 then clicando=false end
    if i.UserInputType==Enum.UserInputType.Touch then clicando=false end
end))

_G._AimbotTrigger="mouse1"; _G._AimbotTriggerKey=Enum.KeyCode.RightShift; _G.AimbotAuto=false

local function AimbotAtivado()
    if _G.AimbotAuto then return true end
    -- No modo mobile, ativa sempre que o aimbot estiver ligado
    if _G.ModoMobile then return true end
    local trigger=_G._AimbotTrigger or "mouse1"
    if trigger=="mouse1" then return UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
    elseif trigger=="mouse2" then return UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    elseif trigger=="key" then local k=_G._AimbotTriggerKey; if k then return UIS:IsKeyDown(k) end end
    return clicando
end

local ringDrawing=nil; local ringImage=nil; local hasDrawing=false
pcall(function() local t=Drawing.new("Circle"); if t and t.Visible~=nil then hasDrawing=true; t:Remove() end end)
if hasDrawing then pcall(function()
    ringDrawing=Drawing.new("Circle"); ringDrawing.Visible=false; ringDrawing.Thickness=1.5
    ringDrawing.Color=Color3.fromRGB(100,100,115); ringDrawing.Filled=false; ringDrawing.NumSides=72
    local mp=UIS:GetMouseLocation(); ringDrawing.Position=mp; ringDrawing.Radius=_G.AIMBOT_RADIUS
end) end
if not ringDrawing then
    local sg_r=Instance.new("ScreenGui"); sg_r.Name="XenonRingGui"; sg_r.ResetOnSpawn=false; sg_r.IgnoreGuiInset=true
    local function GetGuiParent2() local ok1,cg=pcall(function() return game:GetService("CoreGui") end); if ok1 and cg then local ok2=pcall(function() local t=Instance.new("Hint",cg); t:Destroy() end); if ok2 then return cg end end; if typeof(gethui)=="function" then local ok3,h=pcall(gethui); if ok3 and h then return h end end; return LP:WaitForChild("PlayerGui") end
    sg_r.Parent=GetGuiParent2()
    ringImage=Instance.new("ImageLabel",sg_r); ringImage.BackgroundTransparency=1; ringImage.Image="rbxassetid://4911161031"; ringImage.ImageColor3=Color3.fromRGB(100,100,115); ringImage.ImageTransparency=0.25; ringImage.ZIndex=10; ringImage.Visible=false; ringImage.AnchorPoint=Vector2.new(0.5,0.5); ringImage.Size=UDim2.new(0,_G.AIMBOT_RADIUS*2,0,_G.AIMBOT_RADIUS*2); ringImage.Position=UDim2.new(0.5,0,0.5,0)
end
local lastRadius=_G.AIMBOT_RADIUS
local function SetRingVisible(v) pcall(function() if ringDrawing then ringDrawing.Visible=v elseif ringImage then ringImage.Visible=v end end) end
local function SetRingColor(c) if ringDrawing then pcall(function() ringDrawing.Color=c end) elseif ringImage then pcall(function() ringImage.ImageColor3=c end) end end
local function SetRingRadius(r) if ringDrawing then pcall(function() local mp=UIS:GetMouseLocation(); ringDrawing.Radius=r; ringDrawing.Position=mp end) elseif ringImage then pcall(function() ringImage.Size=UDim2.new(0,r*2,0,r*2) end) end end

-- Snap via RenderStepped — compatível com todos executors
RegConn(RunService.RenderStepped:Connect(function()
    if not (_G.Aimbot and _G.AimbotSnap) then return end
    if _G.ModoMobile then return end
    local myChar = LP.Character; if not myChar then return end
    local camPos = Camera.CFrame.Position
    local _ign    = _G._Xenon_IgnoradosSet or {}
    local _alvos  = _G._Xenon_AlvosSet or {}
    local _temAlvos = next(_alvos) ~= nil
    local alvo, menorDist = nil, math.huge

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character and p.Character ~= myChar then
            local passaFiltro
            if _temAlvos then
                passaFiltro = _alvos[p.Name:lower()] or _alvos[p.DisplayName:lower()]
            else
                passaFiltro = not _ign[p.Name:lower()] and not _ign[p.DisplayName:lower()]
            end
            if passaFiltro and _G.AimbotSoKnife then
                local boxes = _G._Xenon_ItemBoxes
                passaFiltro = boxes and boxes[p] and boxes[p].toolName == "Knife"
            end
            if passaFiltro then
                local hum = p.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local _, vis, depth = Camera:WorldToViewportPoint(hrp.Position)
                        if vis and depth > 0 and depth < menorDist then
                            menorDist = depth
                            alvo = hrp
                        end
                    end
                end
            end
        end
    end

    if alvo then
        Camera.CFrame = CFrame.new(camPos, alvo.Position)
    end
end))

RegConn(RunService.RenderStepped:Connect(function()
    if _G.Aimbot then
        SetRingVisible(true)
        if _G.AIMBOT_RADIUS ~= lastRadius then SetRingRadius(tonumber(_G.AIMBOT_RADIUS) or 250); lastRadius=_G.AIMBOT_RADIUS end
        local vp2 = Camera.ViewportSize
        local mp2 = _G.ModoMobile and Vector2.new(vp2.X/2, vp2.Y/2) or UIS:GetMouseLocation()
        if ringDrawing then pcall(function() ringDrawing.Position=mp2 end) end
        if ringImage   then pcall(function() ringImage.Position=UDim2.new(0,mp2.X,0,mp2.Y) end) end
    else SetRingVisible(false); return end

    if not AimbotAtivado() then SetRingColor(Color3.fromRGB(100,100,115)); return end

    -- Modo Mobile: mira no centro da tela; PC: mira no cursor
    local vp = Camera.ViewportSize
    local mp = _G.ModoMobile and Vector2.new(vp.X/2, vp.Y/2) or UIS:GetMouseLocation()
    local camPos=Camera.CFrame.Position
    local myChar=LP.Character
    -- Usa Players:GetPlayers() direto — sem filtro de time, pega todos os inimigos
    local alvo,menorDist,alvoScreen=nil,math.huge,nil

    local _ign = _G._Xenon_IgnoradosSet or {}
    local _alvos = _G._Xenon_AlvosSet or {}
    local _temAlvos = next(_alvos) ~= nil
    for _,p in ipairs(Players:GetPlayers()) do
        if p~=LP and p.Character and p.Character~=myChar then
            -- Se há alvos fixos, só mira neles; senão usa blacklist normal
            local passaFiltro
            if _temAlvos then
                passaFiltro = _alvos[p.Name:lower()] or _alvos[p.DisplayName:lower()]
            else
                passaFiltro = not _ign[p.Name:lower()] and not _ign[p.DisplayName:lower()]
            end
            if passaFiltro and _G.AimbotSoKnife then
                local boxes = _G._Xenon_ItemBoxes
                passaFiltro = boxes and boxes[p] and boxes[p].toolName == "Knife"
            end
            if passaFiltro then
            local hum=p.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health>0 then
                local head=p.Character:FindFirstChild("Head")
                if head then
                    local pos,_=Camera:WorldToViewportPoint(head.Position)
                    if pos.Z > 0 then  -- na frente da câmera
                        local sp=Vector2.new(pos.X,pos.Y)
                        local d=(sp-mp).Magnitude
                        if d<(tonumber(_G.AIMBOT_RADIUS) or 250) and d<menorDist then
                            if _G._Xenon_TemVisao and _G._Xenon_TemVisao(camPos, head.Position) then
                                alvo=head; menorDist=d; alvoScreen=sp
                            end
                        end
                    end
                end
            end
            end -- passaFiltro
        end -- p~=LP
    end

    if alvo and alvoScreen then
        SetRingColor(Color3.fromRGB(220,60,60))
        local smooth=math.clamp(tonumber(_G.AIMBOT_SMOOTH) or 0.25,0.05,1.0)
        if _G.ModoMobile then
            -- Se pediu lock, trava agora no alvo encontrado
            if _G._PedindoLock then
                _G._PedindoLock = nil
                _G._AlvoTravado = alvo
                _G._AlvoTravadoPlayer = alvo.Parent -- personagem
            end
            -- Usa alvo travado se existir e ainda válido, senão usa o mais próximo
            local alvoFinal = alvo
            if _G._AlvoTravado then
                local ok, pos = pcall(function() return _G._AlvoTravado.Position end)
                if ok and _G._AlvoTravadoPlayer and _G._AlvoTravadoPlayer.Parent then
                    alvoFinal = _G._AlvoTravado
                else
                    -- Alvo saiu do jogo, destravar
                    _G._AlvoTravado = nil
                    _G._AlvoTravadoPlayer = nil
                    if _G._Xenon_LockBtn then
                        _G._Xenon_LockBtn.Text = "🔓"
                        _G._Xenon_LockBtn.BackgroundColor3 = Color3.fromRGB(40,40,60)
                    end
                end
            end
            local targetCF = _G.AimbotSnap
                and CFrame.new(camPos, alvoFinal.Position)
                or Camera.CFrame:Lerp(CFrame.new(camPos, alvoFinal.Position), smooth)
            Camera.CFrame = targetCF
        else
            -- Snap PC é tratado no BindToRenderStep de alta prioridade
            if not _G.AimbotSnap then
                local dx=alvoScreen.X-mp.X; local dy=alvoScreen.Y-mp.Y
                local dist=math.sqrt(dx*dx+dy*dy)
                if dist>1 then
                    local ok=pcall(mousemoverel,dx*smooth,dy*smooth)
                    if not ok then
                        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(camPos, alvo.Position), smooth)
                    end
                end
            end
            -- Auto Fire: sinaliza thread dedicada
            if _G.AutoFire and not _G.ModoMobile then
                _G._AutoFireAtivo = true
            else
                _G._AutoFireAtivo = false
            end
        end
    else
        -- Sem alvo no FOV: se travado, continua mirando nele mesmo fora do FOV
        if _G.ModoMobile and _G._AlvoTravado then
            local ok, _ = pcall(function() return _G._AlvoTravado.Position end)
            if ok and _G._AlvoTravadoPlayer and _G._AlvoTravadoPlayer.Parent then
                local smooth2=math.clamp(tonumber(_G.AIMBOT_SMOOTH) or 0.25,0.05,1.0)
                local targetCF = _G.AimbotSnap
                    and CFrame.new(camPos, _G._AlvoTravado.Position)
                    or Camera.CFrame:Lerp(CFrame.new(camPos, _G._AlvoTravado.Position), smooth2)
                Camera.CFrame = targetCF
                SetRingColor(Color3.fromRGB(220,120,30)) -- laranja = travado fora do FOV
            else
                _G._AlvoTravado = nil; _G._AlvoTravadoPlayer = nil
                if _G._Xenon_LockBtn then _G._Xenon_LockBtn.Text="🔓"; _G._Xenon_LockBtn.BackgroundColor3=Color3.fromRGB(40,40,60) end
                SetRingColor(Color3.fromRGB(100,100,115))
            end
        else
            SetRingColor(Color3.fromRGB(100,100,115))
            _G._AutoFireAtivo = false
        end
    end
end))
end)()

-- ============================================================
-- AUTO FIRE NORMAL (sem delay, press/release direto)
-- ============================================================
_G._AutoFireAtivo = false
RegThread(task.spawn(function()
    while true do
        if _G.AutoFire and _G._AutoFireAtivo and not _G.ModoMobile and not _G.AutoFireSmart then
            pcall(mouse1press)
            pcall(mouse1release)
        end
        task.wait()
    end
end))

-- ============================================================
-- AUTO FIRE SMART — visibilidade na tela + LOS + snap + fire
-- ============================================================
if _G.AutoFireSmart == nil then _G.AutoFireSmart = false end
_G._SmartFireAlvo    = nil
_G._SmartFireAtivado = false

-- LOS check: passa a part direto, exclui char local e char do alvo
local function temLOS(camPos, part)
    local myChar = LP.Character
    if not myChar then return false end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local exclude = {myChar}
    if part and part.Parent then table.insert(exclude, part.Parent) end
    params.FilterDescendantsInstances = exclude
    local dir = part.Position - camPos
    local result = workspace:Raycast(camPos, dir, params)
    return result == nil
end

-- Smart Fire loop — RenderStepped compatível com todos executors
local _smartOk = pcall(function()
    RunService:BindToRenderStep("XenonSmartFire", Enum.RenderPriority.Camera.Value + 2, function()
    if not _G.AutoFireSmart then
        _G._SmartFireAlvo    = nil
        _G._SmartFireAtivado = false
        return
    end

    local myChar = LP.Character
    if not myChar then _G._SmartFireAlvo = nil; _G._SmartFireAtivado = false; return end

    local camPos    = Camera.CFrame.Position
    local _alvos    = _G._Xenon_AlvosSet or {}
    local _temAlvos = next(_alvos) ~= nil

    local melhorAlvo, melhorDist = nil, math.huge

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character and p.Character ~= myChar then
            -- Se tem alvos fixados, só processa quem tiver marcado na lista AIM
            if _temAlvos and not (_alvos[p.Name:lower()] or _alvos[p.DisplayName:lower()]) then
                continue
            end
            if _G.AimbotSoKnife then
                local boxes = _G._Xenon_ItemBoxes
                if not (boxes and boxes[p] and boxes[p].toolName == "Knife") then continue end
            end

            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local part = p.Character:FindFirstChild("Head")
                          or p.Character:FindFirstChild("HumanoidRootPart")
                if part then
                    -- Qualquer player visível na tela (sem filtro de FOV circular)
                    local _, vis, depth = Camera:WorldToViewportPoint(part.Position)
                    if vis and depth > 0 and depth < melhorDist then
                        if temLOS(camPos, part) then
                            melhorDist = depth
                            melhorAlvo = part
                        end
                    end
                end
            end
        end
    end

    _G._SmartFireAlvo = melhorAlvo

    if melhorAlvo then
        Camera.CFrame    = CFrame.new(camPos, melhorAlvo.Position)
        _G._SmartFireAtivado = true
    else
        _G._SmartFireAtivado = false
    end
    end)
end)
if not _smartOk then
    -- Fallback: executor não suporta BindToRenderStep, usa RenderStepped normal
    RegConn(RunService.RenderStepped:Connect(function()
        if not _G.AutoFireSmart then _G._SmartFireAlvo=nil; _G._SmartFireAtivado=false; return end
        local myChar=LP.Character; if not myChar then _G._SmartFireAlvo=nil; _G._SmartFireAtivado=false; return end
        local camPos=Camera.CFrame.Position
        local _alvos=_G._Xenon_AlvosSet or {}; local _temAlvos=next(_alvos)~=nil
        local melhorAlvo,melhorDist=nil,math.huge
        for _,p in ipairs(Players:GetPlayers()) do
            if p~=LP and p.Character and p.Character~=myChar then
                if _temAlvos and not(_alvos[p.Name:lower()] or _alvos[p.DisplayName:lower()]) then continue end
                if _G.AimbotSoKnife then local boxes=_G._Xenon_ItemBoxes; if not(boxes and boxes[p] and boxes[p].toolName=="Knife") then continue end end
                local hum=p.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health>0 then
                    local part=p.Character:FindFirstChild("Head") or p.Character:FindFirstChild("HumanoidRootPart")
                    if part then
                        local _,vis,depth=Camera:WorldToViewportPoint(part.Position)
                        if vis and depth>0 and depth<melhorDist and temLOS(camPos,part) then
                            melhorDist=depth; melhorAlvo=part
                        end
                    end
                end
            end
        end
        _G._SmartFireAlvo=melhorAlvo
        if melhorAlvo then Camera.CFrame=CFrame.new(camPos,melhorAlvo.Position); _G._SmartFireAtivado=true
        else _G._SmartFireAtivado=false end
    end))
end

-- Thread de disparo — task.wait() mínimo, reconfirma LOS antes de cada tiro
RegThread(task.spawn(function()
    while true do
        if _G.AutoFireSmart and _G._SmartFireAtivado then
            local alvo = _G._SmartFireAlvo
            if alvo and alvo.Parent then
                local camPos = Camera.CFrame.Position
                if temLOS(camPos, alvo) then
                    pcall(mouse1press)
                    pcall(mouse1release)
                end
            end
        end
        task.wait()
    end
end))

print("✅ Xenon ESP + Aimbot carregado")

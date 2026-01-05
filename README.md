-- F1 QUALIFY HUD - DEFINITIVO (SEM BUG LABEL)
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

-------------------------------------------------
-- CONFIG
-------------------------------------------------
local LOGO_PATH = "C:/F1/F1logo.png"

-------------------------------------------------
-- FUNÇÕES
-------------------------------------------------
local function formatTime(t)
	local m = math.floor(t / 60)
	local s = math.floor(t % 60)
	local ms = math.floor((t % 1) * 1000)
	return string.format("%02d:%02d:%03d", m, s, ms)
end

local function normalizeName(name)
	name = string.lower(name)
	return string.upper(name:sub(1,1)) .. name:sub(2)
end

-------------------------------------------------
-- VARIÁVEIS
-------------------------------------------------
local running = false
local startTime = 0
local conn
local records = {}
local rows = {}

-------------------------------------------------
-- GUI
-------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Parent = game.CoreGui

-------------------------------------------------
-- CONTROLE
-------------------------------------------------
local control = Instance.new("Frame", gui)
control.Size = UDim2.new(0,340,0,300)
control.Position = UDim2.new(0.5,-170,0.5,-150)
control.BackgroundColor3 = Color3.fromRGB(20,20,20)
control.Active = true
control.Draggable = true
Instance.new("UICorner", control)

-- BOTÕES
local function makeTopButton(txt, x, color)
	local b = Instance.new("TextButton", control)
	b.Size = UDim2.new(0,30,0,30)
	b.Position = UDim2.new(1,x,0,5)
	b.Text = txt
	b.Font = Enum.Font.GothamBold
	b.TextSize = 18
	b.TextColor3 = Color3.new(1,1,1)
	b.BackgroundColor3 = color
	return b
end

local btnClose = makeTopButton("X", -35, Color3.fromRGB(150,0,0))
local btnMin   = makeTopButton("–", -70, Color3.fromRGB(70,70,70))

-- TEMPO
local timeLabel = Instance.new("TextLabel", control)
timeLabel.Size = UDim2.new(1,0,0,60)
timeLabel.Position = UDim2.new(0,0,0,40)
timeLabel.BackgroundTransparency = 1
timeLabel.Text = "00:00:000"
timeLabel.Font = Enum.Font.GothamBold
timeLabel.TextSize = 36
timeLabel.TextColor3 = Color3.new(1,1,1)

-- NOME
local nameBox = Instance.new("TextBox", control)
nameBox.Size = UDim2.new(0.85,0,0,36)
nameBox.Position = UDim2.new(0.075,0,0,115)
nameBox.PlaceholderText = "Nome do piloto"
nameBox.Font = Enum.Font.Gotham
nameBox.TextSize = 20
nameBox.TextColor3 = Color3.new(1,1,1)
nameBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
Instance.new("UICorner", nameBox)

-- BOTÕES
local btnStart = Instance.new("TextButton", control)
btnStart.Size = UDim2.new(0.6,0,0,42)
btnStart.Position = UDim2.new(0.2,0,0,165)
btnStart.Text = "INICIAR"
btnStart.Font = Enum.Font.GothamBold
btnStart.TextSize = 20
btnStart.TextColor3 = Color3.new(1,1,1)
btnStart.BackgroundColor3 = Color3.fromRGB(200,0,0)

local btnReset = Instance.new("TextButton", control)
btnReset.Size = UDim2.new(0.6,0,0,36)
btnReset.Position = UDim2.new(0.2,0,0,220)
btnReset.Text = "RESET GERAL"
btnReset.Font = Enum.Font.GothamBold
btnReset.TextSize = 16
btnReset.TextColor3 = Color3.new(1,1,1)
btnReset.BackgroundColor3 = Color3.fromRGB(80,80,80)

-------------------------------------------------
-- HUB QUALIFY
-------------------------------------------------
local qualy = Instance.new("Frame", gui)
qualy.Size = UDim2.new(0,320,1,0)
qualy.Position = UDim2.new(0,0,0,0)
qualy.BackgroundColor3 = Color3.fromRGB(10,10,10)

-- LOGO
local logo = Instance.new("ImageLabel", qualy)
logo.Size = UDim2.new(1,0,0,80)
logo.BackgroundTransparency = 1
logo.Image = getcustomasset(LOGO_PATH)
logo.ScaleType = Enum.ScaleType.Fit

-- LISTA
local list = Instance.new("Frame", qualy)
list.Size = UDim2.new(1,0,1,-90)
list.Position = UDim2.new(0,0,0,90)
list.BackgroundTransparency = 1

-------------------------------------------------
-- RANKING (SEM BUG)
-------------------------------------------------
local function refreshRanking()
	local arr = {}
	for _, v in pairs(records) do table.insert(arr, v) end
	table.sort(arr, function(a,b) return a.time < b.time end)

	for i, d in ipairs(arr) do
		local row = rows[d.key]

		if not row then
			row = Instance.new("Frame", list)
			row.Size = UDim2.new(1,-10,0,40)
			row.BackgroundColor3 = Color3.fromRGB(25,25,25)
			rows[d.key] = row

			local pos = Instance.new("TextLabel", row)
			pos.Name = "PosLabel"
			pos.Size = UDim2.new(0,40,1,0)
			pos.BackgroundTransparency = 1
			pos.Text = ""
			pos.Font = Enum.Font.GothamBold
			pos.TextSize = 22
			pos.TextColor3 = Color3.new(1,1,1)

			local driver = Instance.new("TextLabel", row)
			driver.Name = "DriverLabel"
			driver.Size = UDim2.new(0.55,0,1,0)
			driver.Position = UDim2.new(0,45,0,0)
			driver.BackgroundTransparency = 1
			driver.Text = ""
			driver.Font = Enum.Font.GothamBold
			driver.TextSize = 22
			driver.TextColor3 = Color3.new(1,1,1)
			driver.TextXAlignment = Enum.TextXAlignment.Left

			local time = Instance.new("TextLabel", row)
			time.Name = "TimeLabel"
			time.Size = UDim2.new(0.3,-10,1,0)
			time.Position = UDim2.new(0.7,0,0,0)
			time.BackgroundTransparency = 1
			time.Text = ""
			time.Font = Enum.Font.Gotham
			time.TextSize = 20
			time.TextColor3 = Color3.fromRGB(220,220,220)
			time.TextXAlignment = Enum.TextXAlignment.Right
		end

		row.PosLabel.Text = tostring(i)
		row.DriverLabel.Text = string.upper(d.name)
		row.TimeLabel.Text = formatTime(d.time)
		row.BackgroundColor3 = (i == 1)
			and Color3.fromRGB(80,0,0)
			or Color3.fromRGB(25,25,25)

		TweenService:Create(row, TweenInfo.new(0.3), {
			Position = UDim2.new(0,5,0,(i-1)*42)
		}):Play()
	end
end

-------------------------------------------------
-- CRONÔMETRO
-------------------------------------------------
btnStart.MouseButton1Click:Connect(function()
	if not running then
		running = true
		startTime = tick()
		btnStart.Text = "PARAR"
		conn = RunService.RenderStepped:Connect(function()
			timeLabel.Text = formatTime(tick() - startTime)
		end)
	else
		running = false
		btnStart.Text = "INICIAR"
		if conn then conn:Disconnect() end

		local raw = nameBox.Text ~= "" and nameBox.Text or "Desconhecido"
		local name = normalizeName(raw)
		local key = string.lower(name)
		local t = tick() - startTime

		if not records[key] or t < records[key].time then
			records[key] = {name = name, time = t, key = key}
		end

		timeLabel.Text = "00:00:000"
		refreshRanking()
	end
end)

-------------------------------------------------
-- BOTÕES
-------------------------------------------------
btnReset.MouseButton1Click:Connect(function()
	records = {}
	rows = {}
	list:ClearAllChildren()
end)

btnClose.MouseButton1Click:Connect(function()
	gui:Destroy()
end)

btnMin.MouseButton1Click:Connect(function()
	control.Visible = not control.Visible
end)

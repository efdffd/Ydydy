--------------------------------------------------
-- SERVIÇOS
--------------------------------------------------
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

--------------------------------------------------
-- ESTADOS
--------------------------------------------------
local ESP_ON = false
local INF_JUMP = false
local TARGET_ON = false
local THREAT_ON = false

local ESP = {}

-- Configurações
local THREAT_DISTANCE = 12
local SPEED_THRESHOLD = 60

--------------------------------------------------
-- INTERFACE MOBILE
--------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "MobileDebugUI"
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false

local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0,120,0,40)
openBtn.Position = UDim2.new(0,20,0.5,-20)
openBtn.Text = "MENU"
openBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
openBtn.TextColor3 = Color3.new(1,1,1)
openBtn.Parent = gui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,260,0,300)
frame.Position = UDim2.new(0.5,-130,0.5,-150)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.Visible = false
frame.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.Text = "DEBUG TOOLS"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.Parent = frame

local function makeButton(text, y)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0.9,0,0,40)
	b.Position = UDim2.new(0.05,0,0,y)
	b.Text = text
	b.BackgroundColor3 = Color3.fromRGB(120,0,0)
	b.TextColor3 = Color3.new(1,1,1)
	b.Parent = frame
	return b
end

local espBtn    = makeButton("ESP: OFF", 40)
local jumpBtn   = makeButton("INF JUMP: OFF", 90)
local targetBtn = makeButton("TARGET VISUAL: OFF", 140)
local threatBtn = makeButton("THREAT WARNING: OFF", 190)

openBtn.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

--------------------------------------------------
-- TOGGLES
--------------------------------------------------
espBtn.MouseButton1Click:Connect(function()
	ESP_ON = not ESP_ON
	espBtn.Text = ESP_ON and "ESP: ON" or "ESP: OFF"
	espBtn.BackgroundColor3 = ESP_ON and Color3.fromRGB(0,120,0) or Color3.fromRGB(120,0,0)

	for _, v in pairs(ESP) do
		v.Highlight.Enabled = ESP_ON
		v.Gui.Enabled = ESP_ON
	end
end)

jumpBtn.MouseButton1Click:Connect(function()
	INF_JUMP = not INF_JUMP
	jumpBtn.Text = INF_JUMP and "INF JUMP: ON" or "INF JUMP: OFF"
	jumpBtn.BackgroundColor3 = INF_JUMP and Color3.fromRGB(0,120,0) or Color3.fromRGB(120,0,0)
end)

targetBtn.MouseButton1Click:Connect(function()
	TARGET_ON = not TARGET_ON
	targetBtn.Text = TARGET_ON and "TARGET VISUAL: ON" or "TARGET VISUAL: OFF"
	targetBtn.BackgroundColor3 = TARGET_ON and Color3.fromRGB(0,120,0) or Color3.fromRGB(120,0,0)
end)

threatBtn.MouseButton1Click:Connect(function()
	THREAT_ON = not THREAT_ON
	threatBtn.Text = THREAT_ON and "THREAT WARNING: ON" or "THREAT WARNING: OFF"
	threatBtn.BackgroundColor3 = THREAT_ON and Color3.fromRGB(0,120,0) or Color3.fromRGB(120,0,0)
end)

--------------------------------------------------
-- OVERLAY DE ALERTA (THREAT)
--------------------------------------------------
local warningGui = Instance.new("Frame")
warningGui.Size = UDim2.new(1,0,1,0)
warningGui.BackgroundColor3 = Color3.fromRGB(255,0,0)
warningGui.BackgroundTransparency = 1
warningGui.Visible = false
warningGui.ZIndex = 50
warningGui.Parent = gui

--------------------------------------------------
-- INFINITY JUMP REAL
--------------------------------------------------
UserInputService.JumpRequest:Connect(function()
	if INF_JUMP then
		local char = LocalPlayer.Character
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		if hum then
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

--------------------------------------------------
-- ESP
--------------------------------------------------
local function addESP(player)
	if player == LocalPlayer then return end

	local highlight = Instance.new("Highlight")
	highlight.FillTransparency = 0.6
	highlight.Enabled = ESP_ON
	highlight.Parent = player.Character or player.CharacterAdded:Wait()

	local bbg = Instance.new("BillboardGui")
	bbg.Size = UDim2.new(0,150,0,40)
	bbg.StudsOffset = Vector3.new(0,3,0)
	bbg.AlwaysOnTop = true
	bbg.Enabled = ESP_ON

	local txt = Instance.new("TextLabel")
	txt.Size = UDim2.new(1,0,1,0)
	txt.BackgroundTransparency = 1
	txt.TextColor3 = Color3.new(1,1,1)
	txt.TextStrokeTransparency = 0
	txt.TextScaled = true
	txt.Parent = bbg

	bbg.Parent = workspace

	ESP[player] = {
		Highlight = highlight,
		Gui = bbg,
		Text = txt
	}
end

--------------------------------------------------
-- TARGET VISUAL (CORRIGIDO)
--------------------------------------------------
local TargetHighlight = Instance.new("Highlight")
TargetHighlight.FillColor = Color3.fromRGB(255,255,0)
TargetHighlight.FillTransparency = 0.4
TargetHighlight.Enabled = false
TargetHighlight.Parent = workspace

--------------------------------------------------
-- LOOP PRINCIPAL
--------------------------------------------------
RunService.RenderStepped:Connect(function()
	local myChar = LocalPlayer.Character
	local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
	if not myHRP then return end

	local closestChar = nil
	local closestDist = math.huge
	local danger = false

	-- ESP + Target
	for player, data in pairs(ESP) do
		local char = player.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		if hrp then
			data.Highlight.Adornee = char
			data.Gui.Adornee = hrp

			if player.Team == LocalPlayer.Team then
				data.Highlight.FillColor = Color3.fromRGB(0,255,0)
			else
				data.Highlight.FillColor = Color3.fromRGB(255,0,0)
			end

			local dist = (myHRP.Position - hrp.Position).Magnitude
			data.Text.Text = player.Name .. "\n" .. math.floor(dist) .. " studs"

			if TARGET_ON and player.Team ~= LocalPlayer.Team and dist < closestDist then
				closestDist = dist
				closestChar = char
			end
		end
	end

	if TARGET_ON and closestChar then
		TargetHighlight.Enabled = true
		TargetHighlight.Adornee = closestChar
	else
		TargetHighlight.Enabled = false
		TargetHighlight.Adornee = nil
	end

	-- THREAT WARNING
	if THREAT_ON then
		for _, obj in ipairs(workspace:GetDescendants()) do
			if obj:IsA("BasePart") and not obj.CanCollide then
				if obj.AssemblyLinearVelocity.Magnitude > SPEED_THRESHOLD then
					if (obj.Position - myHRP.Position).Magnitude < THREAT_DISTANCE then
						danger = true
						break
					end
				end
			end
		end
	end

	warningGui.Visible = danger
	warningGui.BackgroundTransparency = danger and 0.85 or 1
end)

--------------------------------------------------
-- PLAYERS
--------------------------------------------------
Players.PlayerAdded:Connect(addESP)
for _, p in ipairs(Players:GetPlayers()) do
	addESP(p)
end

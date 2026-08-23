local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local MAX_SPEED = 1000
local speed = 50
local speedEnabled = false
local flyEnabled = false
local goingUp = false
local goingDown = false
local flyVelocity

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "KING_Menu"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Botão flutuante
local openButton = Instance.new("TextButton")
openButton.Size = UDim2.fromOffset(65, 65)
openButton.Position = UDim2.new(0, 20, 0.5, -32)
openButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.Text = "KING"
openButton.TextScaled = true
openButton.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = openButton

-- Menu
local menu = Instance.new("Frame")
menu.Size = UDim2.fromOffset(280, 300)
menu.Position = UDim2.new(0, 95, 0.5, -150)
menu.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
menu.Visible = false
menu.Parent = gui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 15)
menuCorner.Parent = menu

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundTransparency = 1
title.Text = "KING MENU"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Parent = menu

-- Caixa de Speed
local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(1, -20, 0, 45)
speedBox.Position = UDim2.fromOffset(10, 55)
speedBox.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
speedBox.TextColor3 = Color3.fromRGB(255, 255, 255)
speedBox.PlaceholderText = "Speed 1-1000"
speedBox.Text = "50"
speedBox.TextScaled = true
speedBox.Parent = menu

-- Botão Speed
local speedButton = Instance.new("TextButton")
speedButton.Size = UDim2.new(1, -20, 0, 45)
speedButton.Position = UDim2.fromOffset(10, 110)
speedButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
speedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
speedButton.Text = "SPEED: OFF"
speedButton.TextScaled = true
speedButton.Parent = menu

-- Botão Fly
local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(1, -20, 0, 45)
flyButton.Position = UDim2.fromOffset(10, 165)
flyButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.Text = "FLY: OFF"
flyButton.TextScaled = true
flyButton.Parent = menu

-- Subir
local upButton = Instance.new("TextButton")
upButton.Size = UDim2.fromOffset(125, 45)
upButton.Position = UDim2.fromOffset(10, 220)
upButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
upButton.TextColor3 = Color3.fromRGB(255, 255, 255)
upButton.Text = "▲ SUBIR"
upButton.TextScaled = true
upButton.Parent = menu

-- Descer
local downButton = Instance.new("TextButton")
downButton.Size = UDim2.fromOffset(125, 45)
downButton.Position = UDim2.fromOffset(145, 220)
downButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
downButton.TextColor3 = Color3.fromRGB(255, 255, 255)
downButton.Text = "▼ DESCER"
downButton.TextScaled = true
downButton.Parent = menu

-- Abrir/fechar
openButton.Activated:Connect(function()
	menu.Visible = not menu.Visible
end)

-- Speed
speedButton.Activated:Connect(function()
	speedEnabled = not speedEnabled

	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")

	if speedEnabled then
		speed = math.clamp(tonumber(speedBox.Text) or 50, 1, MAX_SPEED)
		speedBox.Text = tostring(speed)
		speedButton.Text = "SPEED: ON"

		if humanoid then
			humanoid.WalkSpeed = speed
		end
	else
		speedButton.Text = "SPEED: OFF"

		if humanoid then
			humanoid.WalkSpeed = 16
		end
	end
end)

-- Alterar Speed
speedBox.FocusLost:Connect(function()
	local value = tonumber(speedBox.Text)

	if value then
		speed = math.clamp(value, 1, MAX_SPEED)
		speedBox.Text = tostring(speed)
	else
		speedBox.Text = tostring(speed)
	end
end)

-- Fly
flyButton.Activated:Connect(function()
	flyEnabled = not flyEnabled

	local character = player.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")

	if not root then
		flyEnabled = false
		return
	end

	if flyEnabled then
		flyButton.Text = "FLY: ON"

		flyVelocity = Instance.new("BodyVelocity")
		flyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
		flyVelocity.Velocity = Vector3.zero
		flyVelocity.Parent = root
	else
		flyButton.Text = "FLY: OFF"

		if flyVelocity then
			flyVelocity:Destroy()
			flyVelocity = nil
		end
	end
end)

-- Subir
upButton.MouseButton1Down:Connect(function()
	goingUp = true
end)

upButton.MouseButton1Up:Connect(function()
	goingUp = false
end)

-- Descer
downButton.MouseButton1Down:Connect(function()
	goingDown = true
end)

downButton.MouseButton1Up:Connect(function()
	goingDown = false
end)

-- Movimento
RunService.RenderStepped:Connect(function()
	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local root = character:FindFirstChild("HumanoidRootPart")

	if not humanoid or not root then return end

	if speedEnabled then
		humanoid.WalkSpeed = speed
	end

	if flyEnabled and flyVelocity then
		local direction = humanoid.MoveDirection
		local vertical = 0

		if goingUp then
			vertical = speed
		elseif goingDown then
			vertical = -speed
		end

		flyVelocity.Velocity = Vector3.new(
			direction.X * speed,
			vertical,
			direction.Z * speed
		)
	end
end)

-- Resetar ao renascer
player.CharacterAdded:Connect(function()
	speedEnabled = false
	flyEnabled = false
	goingUp = false
	goingDown = false

	if flyVelocity then
		flyVelocity:Destroy()
		flyVelocity = nil
	end
end)

local ReplicatedStorage = game:GetService("ReplicatedStorage")

local event = ReplicatedStorage:WaitForChild("InvincibleEvent")

local cooldowns = {}

event.OnServerEvent:Connect(function(player)

	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChild("Humanoid")
	if not humanoid then return end

	-- Cooldown
	if cooldowns[player] then
		if tick() - cooldowns[player] < 300 then
			return
		end
	end

	-- Leaderstats
	local leaderstats = player:FindFirstChild("leaderstats")
	if not leaderstats then return end

	local coins = leaderstats:FindFirstChild("Coins")
	if not coins then return end

	-- Verifica moedas
	if coins.Value < 500 then
		return
	end

	-- Remove moedas
	coins.Value = coins.Value - 500

	-- Salva cooldown
	cooldowns[player] = tick()

	-- Invencibilidade
	local forceField = Instance.new("ForceField")
	forceField.Visible = true
	forceField.Parent = character

	-- Espera 20 segundos
	task.wait(20)

	-- Remove invencibilidade
	if forceField then
		forceField:Destroy()
	end

end)

game.Players.PlayerRemoving:Connect(function(player)
	cooldowns[player] = nil
end)
local player = game.Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UIS = game:GetService("UserInputService")

local event = ReplicatedStorage:WaitForChild("InvincibleEvent")

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("TextButton")
button.Parent = screenGui

button.Size = UDim2.new(0, 200, 0, 50)
button.Position = UDim2.new(0.5, -100, 0.8, 0)

button.Text = "INVENCÍVEL (500)"
button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
button.TextScaled = true
button.BorderSizePixel = 0

-- Estados
local cooldown = false

button.MouseButton1Click:Connect(function()

	if cooldown then
		return
	end

	cooldown = true

	-- Ativa
	button.Text = "ATIVADO"
	button.BackgroundColor3 = Color3.fromRGB(0, 255, 0)

	event:FireServer()

	-- Duração ativa
	task.wait(20)

	-- Cooldown visual
	for i = 300,1,-1 do
		button.Text = "Cooldown "..i
		button.BackgroundColor3 = Color3.fromRGB(100,100,100)
		task.wait(1)
	end

	-- Volta normal
	button.Text = "INVENCÍVEL (500)"
	button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

	cooldown = false

end)

-- ====================
-- DRAG SYSTEM
-- ====================

local dragging = false
local dragInput
local dragStart
local startPos

local function update(input)

	local delta = input.Position - dragStart

	button.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end

button.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPos = button.Position

		input.Changed:Connect(function()

			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end

		end)
	end
end)

button.InputChanged:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseMovement
	or input.UserInputType == Enum.UserInputType.Touch then

		dragInput = input

	end
end)

UIS.InputChanged:Connect(function(input)

	if input == dragInput and dragging then
		update(input)
	end

end)

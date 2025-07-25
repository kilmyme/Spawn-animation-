local player = game.Players.LocalPlayer
local gui = player:WaitForChild("PlayerGui")
local UserInputService = game:GetService("UserInputService")

local animAtivado = false
local animationConnection
local originalRunId = nil

-- Remove GUI antiga se existir
local oldGui = gui:FindFirstChild("AnimsButtonGui")
if oldGui then oldGui:Destroy() end

-- Cria GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AnimsButtonGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = gui

-- Cria botão
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 50, 0, 50)
button.Position = UDim2.new(0, 20, 0, 20)
button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
button.TextColor3 = Color3.new(1,1,1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 18
button.Text = "ON"
button.Parent = screenGui

-- Arredondar botão
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = button

-- Tornar arrastável
local dragging = false
local dragInput, dragStart, startPos

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
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
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
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		update(input)
	end
end)

-- Função de ativar
local function ativarAnim()
	local char = player.Character or player.CharacterAdded:Wait()
	local animate = char:WaitForChild("Animate")
	local humanoid = char:WaitForChild("Humanoid")

	local runAnim = animate:FindFirstChild("run")
	if runAnim and runAnim:FindFirstChild("RunAnim") then
		originalRunId = runAnim.RunAnim.AnimationId
		runAnim.RunAnim.AnimationId = "rbxassetid://616163682" -- Naruto Run
	end

	if animationConnection then animationConnection:Disconnect() end
	animationConnection = humanoid.AnimationPlayed:Connect(function(track)
		track.Looped = true
	end)
end

-- Função de desativar
local function desativarAnim()
	local char = player.Character or player.CharacterAdded:Wait()
	local animate = char:WaitForChild("Animate")
	local humanoid = char:WaitForChild("Humanoid")

	local runAnim = animate:FindFirstChild("run")
	if runAnim and runAnim:FindFirstChild("RunAnim") and originalRunId then
		runAnim.RunAnim.AnimationId = originalRunId
	end

	if animationConnection then
		animationConnection:Disconnect()
		animationConnection = nil
	end
end

-- Clique do botão
button.MouseButton1Click:Connect(function()
	animAtivado = not animAtivado

	if animAtivado then
		ativarAnim()
		button.Text = "✔"
	else
		desativarAnim()
		button.Text = "ON"
	end
end)

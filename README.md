local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera
local mouse = localPlayer:GetMouse()

-- Configurações
local aimRadius = 20
local smoothing = 0.15
local targetPart = "Head"

-- GUI FOV
local gui = Instance.new("ScreenGui", localPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimAssistGui"
gui.ResetOnSpawn = false

local circle = Instance.new("Frame", gui)
circle.Name = "FOVCircle"
circle.AnchorPoint = Vector2.new(0.5, 0.5)
circle.Size = UDim2.new(0, aimRadius * 2, 0, aimRadius * 2)
circle.Position = UDim2.new(0.5, 0, 0.5, 0)
circle.BackgroundTransparency = 0.8
circle.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
circle.BorderSizePixel = 0
Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

-- Mensagem “fearless boa fé”
local introGui = Instance.new("ScreenGui", localPlayer.PlayerGui)
introGui.Name = "IntroText"
introGui.ResetOnSpawn = false

local introLabel = Instance.new("TextLabel", introGui)
introLabel.Size = UDim2.new(1, 0, 0.1, 0)
introLabel.Position = UDim2.new(0.5, 0, 0.4, 0)
introLabel.AnchorPoint = Vector2.new(0.5, 0.5)
introLabel.BackgroundTransparency = 1
introLabel.TextScaled = true
introLabel.Font = Enum.Font.GothamBlack
introLabel.Text = "fearless boa fé"
introLabel.TextColor3 = Color3.new(1, 1, 1)
introLabel.TextStrokeTransparency = 1
introLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
introLabel.TextTransparency = 1

task.spawn(function()
	for i = 1, 20 do
		introLabel.TextTransparency = 1 - i * 0.05
		introLabel.TextStrokeTransparency = 1 - i * 0.05
		task.wait(0.03)
	end
	task.wait(2.5)
	for i = 1, 20 do
		introLabel.TextTransparency = i * 0.05
		introLabel.TextStrokeTransparency = i * 0.05
		task.wait(0.03)
	end
	introGui:Destroy()
end)

-- Funções principais
local function getClosestEnemy()
	local closest = nil
	local shortest = math.huge

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer and player.Character and player.Character:FindFirstChild(targetPart) then
			local part = player.Character[targetPart]
			local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(mouse.X, mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
				if dist < aimRadius and dist < shortest then
					local direction = (part.Position - camera.CFrame.Position).Unit
					local rayParams = RaycastParams.new()
					rayParams.FilterDescendantsInstances = {localPlayer.Character}
					rayParams.FilterType = Enum.RaycastFilterType.Blacklist
					local result = Workspace:Raycast(camera.CFrame.Position, direction * 1000, rayParams)
					if result and result.Instance and result.Instance:IsDescendantOf(player.Character) then
						closest = part
						shortest = dist
					end
				end
			end
		end
	end
	return closest
end

local function aimAtTarget(target)
	if not target then return end
	local targetPos = target.Position + Vector3.new(0, 0.1, 0)
	local direction = (targetPos - camera.CFrame.Position).Unit
	local lookCFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + direction)
	camera.CFrame = camera.CFrame:Lerp(lookCFrame, smoothing)
end

RunService.RenderStepped:Connect(function()
	circle.Position = UDim2.new(0.5, 0, 0.5, 0)
	local target = getClosestEnemy()
	aimAtTarget(target)
end)

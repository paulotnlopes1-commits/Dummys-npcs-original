-- SERVIÇOS
local Players = game:GetService("Players")

-- GUI
local player = Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "SpawnerUltraFast"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 260, 0, 400)
frame.Position = UDim2.new(0.5, -130, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,35)
title.Text = "SPAWNER ULTRA FAST"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundColor3 = Color3.fromRGB(15,15,15)
title.Font = Enum.Font.SourceSansBold

local userBox = Instance.new("TextBox", frame)
userBox.PlaceholderText = "Nome do Jogador"
userBox.Size = UDim2.new(0.9,0,0,35)
userBox.Position = UDim2.new(0.05,0,0.1,0)

local countBox = Instance.new("TextBox", frame)
countBox.Text = "1"
countBox.Size = UDim2.new(0.9,0,0,35)
countBox.Position = UDim2.new(0.05,0,0.2,0)

-- BOTÃO RIG
local rigChoice = "R15"
local rigBtn = Instance.new("TextButton", frame)
rigBtn.Size = UDim2.new(0.9,0,0,35)
rigBtn.Position = UDim2.new(0.05,0,0.3,0)
rigBtn.Text = "MODO: R15"
rigBtn.BackgroundColor3 = Color3.fromRGB(130,0,255)
rigBtn.TextColor3 = Color3.new(1,1,1)

rigBtn.MouseButton1Click:Connect(function()
	rigChoice = (rigChoice == "R15") and "R6" or "R15"
	rigBtn.Text = "MODO: "..rigChoice
	rigBtn.BackgroundColor3 = (rigChoice == "R15") and Color3.fromRGB(130,0,255) or Color3.fromRGB(0, 120, 215)
end)

-- BOTÃO: NPC OU PARADO
local behaviorChoice = "NPC"
local behaviorBtn = Instance.new("TextButton", frame)
behaviorBtn.Size = UDim2.new(0.9,0,0,35)
behaviorBtn.Position = UDim2.new(0.05,0,0.4,0)
behaviorBtn.Text = "COMPORTAMENTO: NPC"
behaviorBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 0)
behaviorBtn.TextColor3 = Color3.new(1,1,1)

behaviorBtn.MouseButton1Click:Connect(function()
	behaviorChoice = (behaviorChoice == "NPC") and "PARADO" or "NPC"
	behaviorBtn.Text = "COMPORTAMENTO: "..behaviorChoice
	behaviorBtn.BackgroundColor3 = (behaviorChoice == "NPC") and Color3.fromRGB(200, 150, 0) or Color3.fromRGB(80, 80, 80)
end)

--------------------------------------------------
-- SISTEMA DE ANIMAÇÕES
--------------------------------------------------
local function ligarAnimacoes(dummy, tipo)
	local hum = dummy:WaitForChild("Humanoid")
	if dummy:FindFirstChild("Animate") then dummy.Animate:Destroy() end

	local animator = hum:FindFirstChildOfClass("Animator") or Instance.new("Animator", hum)

	local anims = {
		R6 = {
			idle = "rbxassetid://180435571",
			walk = "rbxassetid://180426354",
			jump = "rbxassetid://125750702",
			fall = "rbxassetid://180436148"
		},
		R15 = {
			idle = "rbxassetid://507766388",
			walk = "rbxassetid://507777826",
			run  = "rbxassetid://507767714",
			jump = "rbxassetid://507768710",
			fall = "rbxassetid://507767968"
		}
	}

	local tracks = {}
	for name, id in pairs(anims[tipo]) do
		local anim = Instance.new("Animation")
		anim.AnimationId = id
		tracks[name] = animator:LoadAnimation(anim)
		if name == "idle" then tracks[name].Priority = Enum.AnimationPriority.Idle
		elseif name == "walk" or name == "run" then tracks[name].Priority = Enum.AnimationPriority.Movement
		else tracks[name].Priority = Enum.AnimationPriority.Action end
	end

	local current
	local function play(name)
		if current ~= tracks[name] then
			if current then current:Stop(0.2) end
			current = tracks[name]
			current:Play(0.2)
		end
	end

	task.spawn(function()
		while dummy and dummy.Parent do
			local speed = hum.RootPart and hum.RootPart.Velocity.Magnitude or 0
			local state = hum:GetState()
			if state == Enum.HumanoidStateType.Jumping or state == Enum.HumanoidStateType.Freefall then
				if hum.RootPart and hum.RootPart.Velocity.Y > 0 then play("jump") else play("fall") end
			elseif speed > 0.5 then
				if tipo == "R15" and speed > 12 then play("run") else play("walk") end
				if current then current:AdjustSpeed(speed / 16) end
			else play("idle") end
			task.wait(0.1)
		end
	end)
end

local function npcAndar(dummy)
	local hum = dummy:WaitForChild("Humanoid")
	task.spawn(function()
		while dummy and dummy.Parent do
			local pos = dummy.PrimaryPart.Position
			local destino = pos + Vector3.new(math.random(-15,15), 0, math.random(-15,15))
			hum:MoveTo(destino)
			hum.MoveToFinished:Wait(2)
			task.wait(2)
		end
	end)
end

--------------------------------------------------
-- SPAWN OTIMIZADO (SEM DELAY)
--------------------------------------------------
local spawnBtn = Instance.new("TextButton", frame)
spawnBtn.Text = "SPAWNAR INSTANTÂNEO"
spawnBtn.Size = UDim2.new(0.9,0,0,45)
spawnBtn.Position = UDim2.new(0.05,0,0.55,0)
spawnBtn.BackgroundColor3 = Color3.fromRGB(0,180,0)
spawnBtn.TextColor3 = Color3.new(1,1,1)
spawnBtn.Font = Enum.Font.SourceSansBold

spawnBtn.MouseButton1Click:Connect(function()
	local name = userBox.Text
	local qtd = tonumber(countBox.Text) or 1
	
	-- Busca ID e Descrição UMA VEZ para ganhar velocidade
	local success, userId = pcall(function() return Players:GetUserIdFromNameAsync(name) end)
	if not success then return end
	
	local descSuccess, description = pcall(function() return Players:GetHumanoidDescriptionFromUserId(userId) end)
	if not descSuccess then return end

	-- Loop de criação acelerado
	for i = 1, qtd do
		local rigType = (rigChoice == "R6") and Enum.HumanoidRigType.R6 or Enum.HumanoidRigType.R15
		local dummy = Players:CreateHumanoidModelFromDescription(description, rigType)
		
		dummy.Parent = workspace
		dummy.Name = name
		
		local char = player.Character
		if char and char:FindFirstChild("HumanoidRootPart") then
			dummy:SetPrimaryPartCFrame(char.HumanoidRootPart.CFrame * CFrame.new(i*5, 0, -10))
		end

		ligarAnimacoes(dummy, rigChoice)
		if behaviorChoice == "NPC" then
			npcAndar(dummy)
		end
	end
end)

local clearBtn = Instance.new("TextButton", frame)
clearBtn.Text = "LIMPAR"
clearBtn.Size = UDim2.new(0.9, 0, 0, 30)
clearBtn.Position = UDim2.new(0.05, 0, 0.85, 0)
clearBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
clearBtn.TextColor3 = Color3.new(1,1,1)
clearBtn.MouseButton1Click:Connect(function()
	for _, obj in ipairs(workspace:GetChildren()) do
		if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and not Players:GetPlayerFromCharacter(obj) then
			obj:Destroy()
		end
	end
end)

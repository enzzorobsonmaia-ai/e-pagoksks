-- LocalScript final — GUI + Chão invisível + ESP Player
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Remove GUI antiga
local existing = player:WaitForChild("PlayerGui"):FindFirstChild("ChaoGUI")
if existing then existing:Destroy() end

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ChaoGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Container arrastável
local container = Instance.new("Frame")
container.Name = "Container"
container.Size = UDim2.new(0, 220, 0, 240)
container.Position = UDim2.new(0.5, -110, 0.6, 0)
container.BackgroundTransparency = 1
container.Active = true
container.Draggable = true
container.Parent = screenGui

-- Função para criar botão cinza com fundo preto transparente e texto
local function criarBotao(nome, texto, posY)
	local btn = Instance.new("TextButton")
	btn.Name = nome
	btn.Size = UDim2.new(0, 200, 0, 50)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(128,128,128) -- cor cinza
	btn.Text = ""
	btn.Parent = container
	btn.Font = Enum.Font.FredokaOne
	btn.TextScaled = true

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 1, 0)
	label.Position = UDim2.new(0,0,0,0)
	label.BackgroundColor3 = Color3.fromRGB(0,0,0)
	label.BackgroundTransparency = 0.5
	label.Text = texto
	label.TextColor3 = Color3.fromRGB(255,255,255)
	label.Font = Enum.Font.FredokaOne
	label.TextScaled = true
	label.Parent = btn
	return btn
end

-- Botões
local botaoAtivar = criarBotao("BotaoAtivar", "Ativar Chão", 10)
local botaoParar  = criarBotao("BotaoParar", "Parar Chão", 70)
local botaoTirar  = criarBotao("BotaoTirar", "Tirar Chão", 130)
local botaoESP    = criarBotao("BotaoESP", "ESP Player", 190)

-- Texto "by Darkzin"
local assinatura = Instance.new("TextLabel")
assinatura.Size = UDim2.new(0, 200, 0, 30)
assinatura.Position = UDim2.new(0,10,0,240)
assinatura.BackgroundColor3 = Color3.fromRGB(0,0,0)
assinatura.BackgroundTransparency = 0.5
assinatura.Text = "by Darkzin"
assinatura.Font = Enum.Font.FredokaOne
assinatura.TextScaled = true
assinatura.TextColor3 = Color3.fromRGB(255,255,255)
assinatura.Parent = container

-- Botão redondo que abre/fecha menu
local menuButton = Instance.new("TextButton")
menuButton.Name = "MenuButton"
menuButton.Size = UDim2.new(0, 60, 0, 60)
menuButton.Position = UDim2.new(0.05, 0, 0.8, 0)
menuButton.BackgroundColor3 = Color3.fromRGB(128,128,128)
menuButton.Text = ""
menuButton.AutoButtonColor = false
menuButton.BorderSizePixel = 0
menuButton.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1,0)
corner.Parent = menuButton

-- Cor do botão mudando aleatoriamente
task.spawn(function()
	while menuButton and menuButton.Parent do
		menuButton.BackgroundColor3 = Color3.fromRGB(math.random(50,200), math.random(50,200), math.random(50,200))
		task.wait(0.5)
	end
end)

-- Toggle visibilidade container
local containerVisivel = true
menuButton.MouseButton1Click:Connect(function()
	containerVisivel = not containerVisivel
	container.Visible = containerVisivel
end)

-- ===== Drag =====
local function makeDragHandle(handle, target)
	local dragging = false
	local dragStart, startPos
	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = target.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - dragStart
			target.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
		end
	end)
end

makeDragHandle(container, container)
makeDragHandle(botaoAtivar, container)
makeDragHandle(botaoParar, container)
makeDragHandle(botaoTirar, container)
makeDragHandle(botaoESP, container)
makeDragHandle(assinatura, container)
makeDragHandle(menuButton, menuButton)

-- ===== Chão invisível =====
local velocidade = 6
local limiteSubida = 500
local chao = nil
local subindo = false
local baseY = nil

local humanoidRoot
local function updateCharacterRefs()
	local char = player.Character
	if char then
		humanoidRoot = char:WaitForChild("HumanoidRootPart")
	end
end
updateCharacterRefs()
player.CharacterAdded:Connect(updateCharacterRefs)

local function criarChao()
	if not chao and humanoidRoot then
		baseY = humanoidRoot.Position.Y - 10
		chao = Instance.new("Part")
		chao.Name = "ChaoGigante"
		chao.Size = Vector3.new(2048, 2, 2048)
		chao.Anchored = true
		chao.CanCollide = false
		chao.Transparency = 1
		chao.Position = Vector3.new(0, baseY, 0)
		chao.Parent = workspace
	end
end

RunService.Heartbeat:Connect(function(dt)
	if chao and subindo and baseY and humanoidRoot then
		local targetY = baseY + limiteSubida
		if chao.Position.Y < targetY then
			local desloc = math.min(velocidade * dt, targetY - chao.Position.Y)
			chao.CFrame = chao.CFrame * CFrame.new(0, desloc, 0)
		end
		local apoioY = chao.Position.Y + 3
		if humanoidRoot.Position.Y < apoioY then
			local c = humanoidRoot.CFrame
			humanoidRoot.CFrame = CFrame.new(c.Position.X, apoioY, c.Position.Z)
		end
	end
end)

-- ===== Botões Chão =====
botaoAtivar.MouseButton1Click:Connect(function()
	criarChao()
	subindo = true
	local label = botaoAtivar:FindFirstChild("Label")
	if label then label.Text = "Ativar Chão" end
end)

botaoParar.MouseButton1Click:Connect(function()
	subindo = false
end)

botaoTirar.MouseButton1Click:Connect(function()
	if chao then
		chao:Destroy()
		chao = nil
		subindo = false
	end
end)

-- ===== ESP Player =====
local espAtivo = false
local highlights = {}
local nameTags = {}

local function createESPForPlayer(plr)
	if not plr.Character then return end
	local char = plr.Character
	if not highlights[plr] then
		local highlight = Instance.new("Highlight")
		highlight.Name = "DarkzinESP"
		highlight.Adornee = char
		highlight.FillColor = Color3.fromRGB(255,0,0)
		highlight.FillTransparency = 0.8
		highlight.OutlineColor = Color3.fromRGB(255,0,0)
		highlight.OutlineTransparency = 0
		highlight.Parent = workspace
		highlights[plr] = highlight
	end
	if not nameTags[plr] and char:FindFirstChild("Head") then
		local bb = Instance.new("BillboardGui")
		bb.Name = "DarkzinName"
		bb.Adornee = char:FindFirstChild("Head")
		bb.Size = UDim2.new(0,180,0,40)
		bb.AlwaysOnTop = true
		bb.Parent = char
		local tl = Instance.new("TextLabel")
		tl.Size = UDim2.new(1,0,1,0)
		tl.BackgroundTransparency = 1
		tl.Text = plr.Name
		tl.Font = Enum.Font.FredokaOne
		tl.TextColor3 = Color3.fromRGB(255,0,0)
		tl.TextScaled = true
		tl.Parent = bb
		nameTags[plr] = bb
	end
end

local function removeESPForPlayer(plr)
	if highlights[plr] then
		pcall(function() highlights[plr]:Destroy() end)
		highlights[plr] = nil
	end
	if nameTags[plr] then
		pcall(function() nameTags[plr]:Destroy() end)
		nameTags[plr] = nil
	end
end

botaoESP.MouseButton1Click:Connect(function()
	espAtivo = not espAtivo
	if espAtivo then
		for _, plr in pairs(Players:GetPlayers()) do
			if plr ~= player then createESPForPlayer(plr) end
		end
	else
		for plr, _ in pairs(highlights) do removeESPForPlayer(plr) end
		for plr, _ in pairs(nameTags) do removeESPForPlayer(plr) end
		highlights = {}
		nameTags = {}
	end
end)

Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function()
		if espAtivo and plr ~= player then
			task.wait(0.5)
			createESPForPlayer(plr)
		end
	end)
end)

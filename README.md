-- LocalScript para o hub OLDCRIPT no Blox Fruits
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

-- Criação da GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.Name = "OLDCRIPTHub"
ScreenGui.ResetOnSpawn = false

-- Frame principal do hub
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 300)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Preto
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

-- Borda neon azul
local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(0, 191, 255) -- Azul neon
UIStroke.Thickness = 3
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Parent = MainFrame

-- Título "OLDCRIPT"
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 50)
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "OLDCRIPT"
TitleLabel.TextColor3 = Color3.fromRGB(0, 191, 255)
TitleLabel.TextSize = 30
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.Parent = MainFrame

-- Botão de toggle (mostrar/esconder hub)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Position = UDim2.new(0, 10, 0, 10)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
ToggleButton.Text = "☰"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 20
ToggleButton.Parent = ScreenGui

-- Função para alternar visibilidade do hub
local isHubVisible = true
ToggleButton.MouseButton1Click:Connect(function()
	isHubVisible = not isHubVisible
	MainFrame.Visible = isHubVisible
end)

-- Container para abas
local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(1, 0, 0, 40)
TabContainer.Position = UDim2.new(0, 0, 0, 50)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = MainFrame

-- Container para conteúdo das abas
local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, 0, 1, -90)
ContentContainer.Position = UDim2.new(0, 0, 0, 90)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

-- Lista de abas
local tabs = {"Farm", "Mar", "Missões", "Teleporte", "Loja"}
local tabButtons = {}
local tabContents = {}

-- Criação das abas e conteúdos
for i, tabName in ipairs(tabs) do
	local TabButton = Instance.new("TextButton")
	TabButton.Size = UDim2.new(0, 80, 0, 40)
	TabButton.Position = UDim2.new(0, (i-1)*80, 0, 0)
	TabButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	TabButton.Text = tabName
	TabButton.TextColor3 = Color3.fromRGB(0, 191, 255)
	TabButton.TextSize = 16
	TabButton.Parent = TabContainer
	table.insert(tabButtons, TabButton)

	local ContentFrame = Instance.new("Frame")
	ContentFrame.Size = UDim2.new(1, 0, 1, 0)
	ContentFrame.BackgroundTransparency = 1
	ContentFrame.Visible = (i == 1) -- Apenas a primeira aba visível inicialmente
	ContentFrame.Parent = ContentContainer
	tabContents[tabName] = ContentFrame

	-- Placeholder para conteúdo das abas
	local ContentLabel = Instance.new("TextLabel")
	ContentLabel.Size = UDim2.new(1, 0, 1, 0)
	ContentLabel.BackgroundTransparency = 1
	ContentLabel.Text = "Conteúdo da aba " .. tabName .. " (a ser implementado)"
	ContentLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	ContentLabel.TextSize = 20
	ContentLabel.Parent = ContentFrame

	-- Evento de clique na aba
	TabButton.MouseButton1Click:Connect(function()
		for _, content in pairs(tabContents) do
			content.Visible = false
		end
		tabContents[tabName].Visible = true
	end)
end

-- Funcionalidade da aba Farm
local FarmContent = tabContents["Farm"]
local FarmLabel = FarmContent:FindFirstChildOfClass("TextLabel")
FarmLabel.Text = "Autofarm: Voar e atacar NPCs com arma selecionada"

-- Botão para ativar/desativar autofarm
local FarmToggleButton = Instance.new("TextButton")
FarmToggleButton.Size = UDim2.new(0, 150, 0, 40)
FarmToggleButton.Position = UDim2.new(0, 10, 0, 50)
FarmToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
FarmToggleButton.Text = "Ativar Autofarm"
FarmToggleButton.TextColor3 = Color3.fromRGB(0, 191, 255)
FarmToggleButton.TextSize = 16
FarmToggleButton.Parent = FarmContent

-- Variáveis de controle do autofarm
local isFarming = false
local farmConnection = nil

-- Função para fazer o jogador voar a uma altura fixa
local function flyAbove(targetPosition)
	local character = LocalPlayer.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end
	local rootPart = character.HumanoidRootPart

	-- Adiciona BodyVelocity para voar
	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
	bodyVelocity.Velocity = Vector3.new(0, 0, 0)
	bodyVelocity.Parent = rootPart

	-- Mantém o jogador a 20 studs acima do alvo
	RunService:BindToRenderStep("Fly", Enum.RenderPriority.Character.Value, function()
		if not isFarming then
			bodyVelocity:Destroy()
			RunService:UnbindFromRenderStep("Fly")
			return
		end
		rootPart.CFrame = CFrame.new(targetPosition + Vector3.new(0, 20, 0))
	end)
end

-- Função para detectar NPCs próximos e atacar
local function attackNPCs()
	local character = LocalPlayer.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end
	local rootPart = character.HumanoidRootPart

	-- Raio da aura/hitbox (longa distância: 30 studs)
	local auraRadius = 30

	-- Loop para atacar NPCs
	farmConnection = RunService.Heartbeat:Connect(function()
		if not isFarming then
			farmConnection:Disconnect()
			return
		end

		-- Encontra NPCs próximos
		local closestNPC = nil
		local closestDistance = math.huge
		for _, npc in pairs(Workspace.NPCs:GetChildren()) do
			if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
				local npcRoot = npc.HumanoidRootPart
				local distance = (rootPart.Position - npcRoot.Position).Magnitude
				if distance < closestDistance and distance <= auraRadius and npc.Humanoid.Health > 0 then
					closestNPC = npc
					closestDistance = distance
				end
			end
		end

		if closestNPC then
			-- Voa acima do NPC
			flyAbove(closestNPC.HumanoidRootPart.Position)

			-- Simula ataque com arma equipada (fruta, estilo de luta, espada ou arma)
			local tool = character:FindFirstChildOfClass("Tool")
			if tool then
				-- Dispara evento de ataque genérico (simulado)
				local humanoid = closestNPC.Humanoid
				if humanoid.Health > 0 then
					humanoid:TakeDamage(10) -- Dano simulado (ajuste conforme necessário)
				end
			end
		end
	end)
end

-- Evento do botão de autofarm
FarmToggleButton.MouseButton1Click:Connect(function()
	isFarming = not isFarming
	FarmToggleButton.Text = isFarming and "Desativar Autofarm" or "Ativar Autofarm"
	if isFarming then
		attackNPCs()
	else
		if farmConnection then
			farmConnection:Disconnect()
		end
		-- Remove BodyVelocity se existir
		local rootPart = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if rootPart and rootPart:FindFirstChild("BodyVelocity") then
			rootPart.BodyVelocity:Destroy()
		end
		RunService:UnbindFromRenderStep("Fly")
	end
end)

-- Inicializa a GUI
MainFrame.Visible = true

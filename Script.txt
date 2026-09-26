-- // G7S HUB - Versão Otimizada (Com Anti Hit Ajustado e Blindado) \\ --
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- Variáveis de Controle Global
local flying = false
local noclipEnabled = false
local antiHitEnabled = false
local flySpeed = 50
local normalSpeed = 16
local flyConnection = nil
local noclipConnection = nil
local antiHitConnection = nil
local rgbConnection = nil
local keys = {W = false, S = false, A = false, D = false, Space = false}

-- Estado RGB Automático Ativado por Padrão
local isRgbActive = true

-- Coordenada exata da Galáxia de Esmeralda
local emeraldGalaxyCFrame = CFrame.new(-3.59818411, 3296.18726, 2291.36353, -0.966708481, -1.08330047e-08, -0.255880266, -1.1440463e-09, 1, -3.80140506e-08, 0.255880266, -3.64557664e-08, -0.966708481)

-- Remove interfaces anteriores para evitar duplicações
if player.PlayerGui:FindFirstChild("G7SHubGui") then
	player.PlayerGui.G7SHubGui:Destroy()
end
if player.PlayerGui:FindFirstChild("G7SOpenButtonGui") then
	player.PlayerGui.G7SOpenButtonGui:Destroy()
end

-- Tabela para rastrear todos os elementos de texto e bordas que mudam com o RGB
local coloredElements = {}

local function registerColorElement(element, elementType)
	table.insert(coloredElements, {obj = element, type = elementType})
end

local function applyColorToAll(color)
	for _, item in ipairs(coloredElements) do
		if item.obj and item.obj.Parent then
			if item.type == "Text" then
				item.obj.TextColor3 = color
			elseif item.type == "Stroke" then
				item.obj.Color = color
			end
		end
	end
end

-- Criação da GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "G7SHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 310, 0, 400)
mainFrame.Position = UDim2.new(0.5, -155, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
mainFrame.BackgroundTransparency = 0.15
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 14)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(80, 80, 70)
mainStroke.Thickness = 1.5
mainStroke.Parent = mainFrame

-- Cabeçalho
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 35)
titleLabel.Position = UDim2.new(0, 0, 0, 12)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "G7S HUB"
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame
registerColorElement(titleLabel, "Text")

local subTitle = Instance.new("TextLabel")
subTitle.Size = UDim2.new(1, 0, 0, 20)
subTitle.Position = UDim2.new(0, 0, 0, 38)
subTitle.BackgroundTransparency = 1
subTitle.Text = "TIKTOK"
subTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
subTitle.TextSize = 10
subTitle.Font = Enum.Font.GothamMedium
subTitle.Parent = mainFrame

local subTitleLink = Instance.new("TextLabel")
subTitleLink.Size = UDim2.new(1, 0, 0, 20)
subTitleLink.Position = UDim2.new(0, 0, 0, 52)
subTitleLink.BackgroundTransparency = 1
subTitleLink.Text = "gbb7"
subTitleLink.TextSize = 12
subTitleLink.Font = Enum.Font.GothamSemibold
subTitleLink.Parent = mainFrame
registerColorElement(subTitleLink, "Text")

-- Botão de Abrir Flutuante
local openButtonGui = Instance.new("ScreenGui")
openButtonGui.Name = "G7SOpenButtonGui"
openButtonGui.ResetOnSpawn = false
openButtonGui.Parent = player:WaitForChild("PlayerGui")

local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0, 110, 0, 42)
openBtn.Position = UDim2.new(0, 20, 0.5, -21)
openBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
openBtn.BackgroundTransparency = 0.2
openBtn.Text = "⚡ G7S HUB"
openBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
openBtn.TextSize = 13
openBtn.Font = Enum.Font.GothamBold
openBtn.Visible = false
openBtn.Active = true
openBtn.Draggable = true
openBtn.Parent = openButtonGui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(0, 10)
openCorner.Parent = openBtn

local openStroke = Instance.new("UIStroke")
openStroke.Thickness = 2
openStroke.Parent = openBtn
registerColorElement(openStroke, "Stroke")

-- Botão Fechar (X)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -38, 0, 12)
closeBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 38)
closeBtn.Text = "X"
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = mainFrame
registerColorElement(closeBtn, "Text")

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = false
	openBtn.Visible = true
end)

openBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = true
	openBtn.Visible = false
end)

-- Sistema de Abas (MAIN / SETTING)
local tabMainBtn = Instance.new("TextButton")
tabMainBtn.Size = UDim2.new(0.46, 0, 0, 30)
tabMainBtn.Position = UDim2.new(0.03, 0, 0, 78)
tabMainBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
tabMainBtn.Text = "MAIN"
tabMainBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
tabMainBtn.TextSize = 11
tabMainBtn.Font = Enum.Font.GothamBold
tabMainBtn.Parent = mainFrame
Instance.new("UICorner", tabMainBtn).CornerRadius = UDim.new(0, 8)

local tabSettingsBtn = Instance.new("TextButton")
tabSettingsBtn.Size = UDim2.new(0.46, 0, 0, 30)
tabSettingsBtn.Position = UDim2.new(0.51, 0, 0, 78)
tabSettingsBtn.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
tabSettingsBtn.TextColor3 = Color3.fromRGB(150, 150, 150)
tabSettingsBtn.TextSize = 11
tabSettingsBtn.Font = Enum.Font.GothamBold
tabSettingsBtn.Text = "SETTING"
tabSettingsBtn.Parent = mainFrame
Instance.new("UICorner", tabSettingsBtn).CornerRadius = UDim.new(0, 8)

-- Containers de Conteúdo
local mainContainer = Instance.new("ScrollingFrame")
mainContainer.Size = UDim2.new(1, -24, 1, -125)
mainContainer.Position = UDim2.new(0, 12, 0, 116)
mainContainer.BackgroundTransparency = 1
mainContainer.BorderSizePixel = 0
mainContainer.CanvasSize = UDim2.new(0, 0, 0, 360)
mainContainer.ScrollBarThickness = 3
mainContainer.Visible = true
mainContainer.Parent = mainFrame

local settingsContainer = Instance.new("ScrollingFrame")
settingsContainer.Size = UDim2.new(1, -24, 1, -125)
settingsContainer.Position = UDim2.new(0, 12, 0, 116)
settingsContainer.BackgroundTransparency = 1
settingsContainer.BorderSizePixel = 0
settingsContainer.CanvasSize = UDim2.new(0, 0, 0, 320)
settingsContainer.ScrollBarThickness = 3
settingsContainer.Visible = false
settingsContainer.Parent = mainFrame

-- Layouts automáticos organizados por ordem
local mainLayout = Instance.new("UIListLayout")
mainLayout.SortOrder = Enum.SortOrder.LayoutOrder
mainLayout.Padding = UDim.new(0, 8)
mainLayout.Parent = mainContainer

local settingsLayout = Instance.new("UIListLayout")
settingsLayout.SortOrder = Enum.SortOrder.LayoutOrder
settingsLayout.Padding = UDim.new(0, 8)
settingsLayout.Parent = settingsContainer

tabMainBtn.MouseButton1Click:Connect(function()
	mainContainer.Visible = true
	settingsContainer.Visible = false
	tabMainBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	tabMainBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	tabSettingsBtn.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
	tabSettingsBtn.TextColor3 = Color3.fromRGB(150, 150, 150)
end)

tabSettingsBtn.MouseButton1Click:Connect(function()
	mainContainer.Visible = false
	settingsContainer.Visible = true
	tabSettingsBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	tabSettingsBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	tabMainBtn.BackgroundColor3 = Color3.fromRGB(28, 28, 35)
	tabMainBtn.TextColor3 = Color3.fromRGB(150, 150, 150)
end)

-- Funções Auxiliares de Componentes
local function createButton(parent, name, order, height, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, height or 38)
	btn.LayoutOrder = order
	btn.BackgroundColor3 = Color3.fromRGB(38, 38, 42)
	btn.BackgroundTransparency = 0.3
	btn.Text = name
	btn.TextSize = 13
	btn.Font = Enum.Font.GothamBold
	btn.Parent = parent
	registerColorElement(btn, "Text")

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 10)
	corner.Parent = btn

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(60, 60, 65)
	stroke.Thickness = 1.2
	stroke.Parent = btn

	btn.MouseButton1Click:Connect(function()
		callback(btn)
	end)
	return btn
end

local function createInput(parent, placeholder, order, callback)
	local box = Instance.new("TextBox")
	box.Size = UDim2.new(1, 0, 0, 38)
	box.LayoutOrder = order
	box.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
	box.BackgroundTransparency = 0.3
	box.PlaceholderText = placeholder
	box.Text = ""
	box.PlaceholderColor3 = Color3.fromRGB(120, 120, 100)
	box.TextSize = 11
	box.Font = Enum.Font.GothamMedium
	box.Parent = parent
	registerColorElement(box, "Text")

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 10)
	corner.Parent = box

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(50, 50, 55)
	stroke.Thickness = 1
	stroke.Parent = box

	box.FocusLost:Connect(function(enterPressed)
		if enterPressed then
			local num = tonumber(box.Text)
			if num then callback(num) end
		end
	end)
end

-- CONTEÚDO DA ABA MAIN --
createButton(mainContainer, "BASE", 1, 38, function()
	local char = player.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char.HumanoidRootPart.CFrame = CFrame.new(0, 5, 0)
	end
end)

createButton(mainContainer, "BEST AREA", 2, 38, function()
	local char = player.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char.HumanoidRootPart.CFrame = emeraldGalaxyCFrame
	end
end)

createButton(mainContainer, "ATIVAR FLY: OFF", 3, 38, function(btn)
	flying = not flying
	local char = player.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	local hrp = char:FindFirstChild("HumanoidRootPart")

	if flying then
		btn.Text = "ATIVAR FLY: ON"
		btn.BackgroundColor3 = Color3.fromRGB(40, 90, 50)
		if hum then hum.PlatformStand = true end

		if flyConnection then flyConnection:Disconnect() end
		flyConnection = RunService.RenderStepped:Connect(function()
			if not flying or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
			local root = player.Character.HumanoidRootPart
			local cam = Workspace.CurrentCamera
			local moveDir = Vector3.new(0, 0, 0)

			if keys.W then moveDir = moveDir + cam.CFrame.LookVector end
			if keys.S then moveDir = moveDir - cam.CFrame.LookVector end
			if keys.A then moveDir = moveDir - cam.CFrame.RightVector end
			if keys.D then moveDir = moveDir + cam.CFrame.RightVector end
			if keys.Space then moveDir = moveDir + Vector3.new(0, 1, 0) end

			root.AssemblyLinearVelocity = moveDir * flySpeed
			root.CFrame = CFrame.new(root.Position, root.Position + cam.CFrame.LookVector)
		end)
	else
		btn.Text = "ATIVAR FLY: OFF"
		btn.BackgroundColor3 = Color3.fromRGB(38, 38, 42)
		if flyConnection then flyConnection:Disconnect() end
		if hum then hum.PlatformStand = false end
		if hrp then hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0) end
	end
end)

-- Função ANTI HIT Ajustada (Mantém os inimigos BEM mais longe e travados)
createButton(mainContainer, "ANTI HIT: OFF", 4, 38, function(btn)
	antiHitEnabled = not antiHitEnabled
	if antiHitEnabled then
		btn.Text = "ANTI HIT: ON"
		btn.BackgroundColor3 = Color3.fromRGB(40, 90, 50)
		
		if antiHitConnection then antiHitConnection:Disconnect() end
		antiHitConnection = RunService.RenderStepped:Connect(function()
			if not antiHitEnabled then return end
			local char = player.Character
			if not char or not char:FindFirstChild("HumanoidRootPart") then return end
			local minhaRoot = char.HumanoidRootPart
			local distanciaSeguranca = 25 -- Distância aumentada para deixar o personagem bem longe de você
			
			-- Varre todo o Workspace procurando por inimigos, mobs ou outros players próximos
			for _, obj in ipairs(Workspace:GetDescendants()) do
				if obj:IsA("Model") and obj ~= char then
					local enemyHum = obj:FindFirstChildOfClass("Humanoid")
					local enemyRoot = obj:FindFirstChild("HumanoidRootPart")
					
					if enemyHum and enemyRoot and enemyHum.Health > 0 then
						local isPlayer = Players:GetPlayerFromCharacter(obj)
						if not isPlayer or isPlayer ~= player then
							local distancia = (minhaRoot.Position - enemyRoot.Position).Magnitude
							
							if distancia < distanciaSeguranca then
								local direcao = (enemyRoot.Position - minhaRoot.Position).Unit
								-- Trava o eixo Y para não afundar nem jogar o player pro céu
								direcao = Vector3.new(direcao.X, 0, direcao.Z)
								
								if direcao.Magnitude > 0 then
									enemyRoot.CFrame = CFrame.new(minhaRoot.Position + (direcao * distanciaSeguranca)) + Vector3.new(0, 2, 0)
								end
								
								enemyRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
								if enemyHum.WalkSpeed > 0 then
									enemyHum:MoveTo(enemyRoot.Position)
								end
							end
						end
					end
				end
			end
		end)
	else
		btn.Text = "ANTI HIT: OFF"
		btn.BackgroundColor3 = Color3.fromRGB(38, 38, 42)
		if antiHitConnection then antiHitConnection:Disconnect() end
	end
end)

createInput(mainContainer, "⚙️ VELOCIDADE VOO (PADRÃO: 50) [ENTER]", 5, function(val)
	flySpeed = val
end)

createInput(mainContainer, "⚡ WALKSPEED (PADRÃO: 16) [ENTER]", 6, function(val)
	normalSpeed = val
	local char = player.Character
	if char and char:FindFirstChildOfClass("Humanoid") then
		char:FindFirstChildOfClass("Humanoid").WalkSpeed = normalSpeed
	end
end)

createButton(mainContainer, "APLICAR WALKSPEED ATUAL", 7, 38, function()
	local char = player.Character
	if char and char:FindFirstChildOfClass("Humanoid") then
		char:FindFirstChildOfClass("Humanoid").WalkSpeed = normalSpeed
	end
end)

-- CONTEÚDO DA ABA SETTING --
createButton(settingsContainer, "NOCLIP: OFF", 1, 38, function(btn)
	noclipEnabled = not noclipEnabled
	if noclipEnabled then
		btn.Text = "NOCLIP: ON"
		btn.BackgroundColor3 = Color3.fromRGB(40, 90, 50)
		if noclipConnection then noclipConnection:Disconnect() end
		noclipConnection = RunService.Stepped:Connect(function()
			local char = player.Character
			if char then
				for _, part in ipairs(char:GetDescendants()) do
					if part:IsA("BasePart") then
						part.CanCollide = false
					end
				end
			end
		end)
	else
		btn.Text = "NOCLIP: OFF"
		btn.BackgroundColor3 = Color3.fromRGB(38, 38, 42)
		if noclipConnection then noclipConnection:Disconnect() end
		local char = player.Character
		if char then
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
					part.CanCollide = true
				end
			end
		end
	end
end)

createButton(settingsContainer, "MODO RGB AUTOMÁTICO: ON", 2, 38, function(btn)
	isRgbActive = not isRgbActive
	if isRgbActive then
		btn.Text = "MODO RGB AUTOMÁTICO: ON"
		btn.BackgroundColor3 = Color3.fromRGB(40, 90, 50)
	else
		btn.Text = "MODO RGB AUTOMÁTICO: OFF"
		btn.BackgroundColor3 = Color3.fromRGB(38, 38, 42)
	end
end)

createButton(settingsContainer, "FECHAR COMPLETAMENTE (DESTROY)", 3, 38, function()
	flying = false
	noclipEnabled = false
	antiHitEnabled = false
	if flyConnection then flyConnection:Disconnect() end
	if noclipConnection then noclipConnection:Disconnect() end
	if antiHitConnection then antiHitConnection:Disconnect() end
	if rgbConnection then rgbConnection:Disconnect() end
	screenGui:Destroy()
	openButtonGui:Destroy()
end)

createButton(settingsContainer, "DISCORD", 4, 38, function()
	if setclipboard then
		setclipboard("https://discord.gg/vjfjMKT85D")
	end
end)

-- Notificação de Link Copiado (Posicionada abaixo do botão Discord na aba Setting)
local notifyLabel = Instance.new("TextLabel")
notifyLabel.Size = UDim2.new(1, 0, 0, 32)
notifyLabel.LayoutOrder = 5
notifyLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
notifyLabel.BackgroundTransparency = 0.2
notifyLabel.Text = "✓ LINK DO DISCORD COPIADO!"
notifyLabel.TextSize = 11
notifyLabel.Font = Enum.Font.GothamBold
notifyLabel.Visible = false
notifyLabel.Parent = settingsContainer

Instance.new("UICorner", notifyLabel).CornerRadius = UDim.new(0, 8)
local notifyStroke = Instance.new("UIStroke")
notifyStroke.Thickness = 1
notifyStroke.Parent = notifyLabel
registerColorElement(notifyStroke, "Stroke")
registerColorElement(notifyLabel, "Text")

local isNotifying = false
local function showDiscordNotification()
	if isNotifying then return end
	isNotifying = true
	
	notifyLabel.Visible = true
	notifyLabel.TextTransparency = 1
	notifyStroke.Transparency = 1
	
	local tweenInText = TweenService:Create(notifyLabel, TweenInfo.new(0.2), {TextTransparency = 0, BackgroundTransparency = 0.2})
	local tweenInStroke = TweenService:Create(notifyStroke, TweenInfo.new(0.2), {Transparency = 0})
	tweenInText:Play()
	tweenInStroke:Play()
	
	task.delay(3, function()
		local tweenOutText = TweenService:Create(notifyLabel, TweenInfo.new(0.2), {TextTransparency = 1, BackgroundTransparency = 1})
		local tweenOutStroke = TweenService:Create(notifyStroke, TweenInfo.new(0.2), {Transparency = 1})
		tweenOutText:Play()
		tweenOutStroke:Play()
		
		tweenOutText.Completed:Connect(function()
			notifyLabel.Visible = false
			isNotifying = false
		end)
	end)
end

settingsContainer:GetChildren()[4].MouseButton1Click:Connect(function()
	showDiscordNotification()
end)

-- Loop RGB Dinâmico Contínuo
local tickVal = 0
rgbConnection = RunService.RenderStepped:Connect(function(dt)
	if isRgbActive then
		tickVal = tickVal + dt * 0.5
		local rainbowColor = Color3.fromHSV(tickVal % 1, 0.9, 1)
		applyColorToAll(rainbowColor)
	end
end)

-- Captura de Teclas do Fly
UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.W then keys.W = true
	elseif input.KeyCode == Enum.KeyCode.S then keys.S = true
	elseif input.KeyCode == Enum.KeyCode.A then keys.A = true
	elseif input.KeyCode == Enum.KeyCode.D then keys.D = true
	elseif input.KeyCode == Enum.KeyCode.Space then keys.Space = true end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then keys.W = false
	elseif input.KeyCode == Enum.KeyCode.S then keys.S = false
	elseif input.KeyCode == Enum.KeyCode.A then keys.A = false
	elseif input.KeyCode == Enum.KeyCode.D then keys.D = false
	elseif input.KeyCode == Enum.KeyCode.Space then keys.Space = false end
end)

-- LocalScript (colocar em StarterGui)
-- Painel de administrador (interface bonita, arrastável, fechar/abrir).
-- Comunica-se com ServerScriptService/AdminServer.lua via RemoteEvent em ReplicatedStorage.AdminRemote

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- Ensure RemoteEvent exists
local remote = ReplicatedStorage:WaitForChild("AdminRemote")

-- Helper: cria objetos UI com propriedades
local function new(instClass, props)
	local inst = Instance.new(instClass)
	for k, v in pairs(props or {}) do
		if k == "Parent" then
			inst.Parent = v
		else
			inst[k] = v
		end
	end
	return inst
end

-- Create ScreenGui
local screenGui = new("ScreenGui", {
	Name = "AdminPanelGui",
	ResetOnSpawn = false,
	Parent = player:WaitForChild("PlayerGui")
})

-- Background blur (subtil)
local overlay = new("Frame", {
	Name = "Overlay",
	Parent = screenGui,
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.5, 0),
	Size = UDim2.new(1, 0, 1, 0),
	BackgroundColor3 = Color3.fromRGB(10, 10, 15),
	BackgroundTransparency = 0.85,
	Visible = false
})

-- Open button (quando GUI fechada)
local openButton = new("TextButton", {
	Name = "OpenButton",
	Parent = screenGui,
	AnchorPoint = Vector2.new(1, 1),
	Position = UDim2.new(0.98, 0, 0.98, 0),
	Size = UDim2.new(0, 64, 0, 64),
	Text = "Admin",
	Font = Enum.Font.GothamSemibold,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(255,255,255),
	BackgroundColor3 = Color3.fromRGB(30,30,40),
	AutoButtonColor = true
})
local openCorner = new("UICorner", {Parent = openButton, CornerRadius = UDim.new(0, 12)})
local openStroke = new("UIStroke", {Parent = openButton, Color = Color3.fromRGB(80,160,255), Thickness = 2})
local openGrad = new("UIGradient", {Parent = openButton, Color = ColorSequence.new(Color3.fromRGB(60,130,240), Color3.fromRGB(40,80,220))})

-- Main panel
local main = new("Frame", {
	Name = "Main",
	Parent = screenGui,
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.5, 0),
	Size = UDim2.new(0, 520, 0, 360),
	BackgroundColor3 = Color3.fromRGB(20, 24, 32),
	Visible = true
})
local mainCorner = new("UICorner", {Parent = main, CornerRadius = UDim.new(0, 14)})
local mainStroke = new("UIStroke", {Parent = main, Color = Color3.fromRGB(50, 90, 170), Thickness = 2})
local bgGrad = new("UIGradient", {Parent = main, Color = ColorSequence.new(Color3.fromRGB(24,28,36), Color3.fromRGB(18,20,26)), Rotation = 270})

-- Titlebar (arrastável)
local titleBar = new("Frame", {
	Name = "TitleBar",
	Parent = main,
	Size = UDim2.new(1, 0, 0, 48),
	BackgroundTransparency = 1
})
local titleLabel = new("TextLabel", {
	Name = "Title",
	Parent = titleBar,
	Position = UDim2.new(0, 16, 0, 8),
	Size = UDim2.new(0.6, 0, 1, -10),
	Text = "Painel de Administrador",
	TextXAlignment = Enum.TextXAlignment.Left,
	Font = Enum.Font.GothamBold,
	TextSize = 20,
	TextColor3 = Color3.fromRGB(230,230,255),
	BackgroundTransparency = 1
})
local closeButton = new("TextButton", {
	Name = "Close",
	Parent = titleBar,
	AnchorPoint = Vector2.new(1, 0),
	Position = UDim2.new(1, -12, 0, 8),
	Size = UDim2.new(0, 32, 0, 32),
	Text = "X",
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	TextColor3 = Color3.fromRGB(255,255,255),
	BackgroundColor3 = Color3.fromRGB(160,40,60)
})
new("UICorner", {Parent = closeButton, CornerRadius = UDim.new(0, 8)})
new("UIStroke", {Parent = closeButton, Color = Color3.fromRGB(200,80,100), Thickness = 1})

-- Content area (ex.: botões de ação)
local content = new("Frame", {
	Name = "Content",
	Parent = main,
	Position = UDim2.new(0, 16, 0, 64),
	Size = UDim2.new(1, -32, 1, -80),
	BackgroundTransparency = 1
})

-- Left column (lista de comandos)
local left = new("Frame", {
	Parent = content,
	Size = UDim2.new(0.45, 0, 1, 0),
	BackgroundColor3 = Color3.fromRGB(29, 33, 44)
})
new("UICorner", {Parent = left, CornerRadius = UDim.new(0, 12)})
new("UIStroke", {Parent = left, Color = Color3.fromRGB(60,100,200), Thickness = 1})
local leftPadding = new("UIPadding", {Parent = left, PaddingTop = UDim.new(0,12), PaddingLeft = UDim.new(0,12)})

-- Right column (detalhes / logs)
local right = new("Frame", {
	Parent = content,
	AnchorPoint = Vector2.new(1, 0),
	Position = UDim2.new(1, 0, 0, 0),
	Size = UDim2.new(0.53, 0, 1, 0),
	BackgroundColor3 = Color3.fromRGB(22, 26, 32)
})
new("UICorner", {Parent = right, CornerRadius = UDim.new(0, 12)})
new("UIStroke", {Parent = right, Color = Color3.fromRGB(55,85,160), Thickness = 1})
local rightLabel = new("TextLabel", {
	Parent = right,
	Position = UDim2.new(0, 12, 0, 12),
	Size = UDim2.new(1, -24, 0, 28),
	Text = "Logs",
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	TextColor3 = Color3.fromRGB(220,220,230),
	BackgroundTransparency = 1
})
local logBox = new("TextBox", {
	Parent = right,
	Position = UDim2.new(0, 12, 0, 48),
	Size = UDim2.new(1, -24, 1, -60),
	MultiLine = true,
	ClearTextOnFocus = false,
	Text = "Nenhuma ação ainda...",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(200,200,215),

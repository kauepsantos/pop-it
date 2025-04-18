local Players = game:GetService("Players")
local PlayerGui = Players.LocalPlayer:WaitForChild("PlayerGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Criando a interface (Frame)
local GUI = Instance.new("ScreenGui")
GUI.Name = "ItemShopGUI"
GUI.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Parent = GUI
MainFrame.Size = UDim2.new(0, 300, 0, 250)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MainFrame.Active = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

local TitleFrame = Instance.new("Frame")
TitleFrame.Parent = MainFrame
TitleFrame.Size = UDim2.new(1, 0, 0, 40)
TitleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local Title = Instance.new("TextLabel")
Title.Parent = TitleFrame
Title.Text = "tubergamer(1.2)"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Size = UDim2.new(0.85, 0, 1, 0)
Title.BackgroundTransparency = 1

-- Botão de Minimizar
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Parent = TitleFrame
MinimizeButton.Text = "-"
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 18
MinimizeButton.Size = UDim2.new(0.15, 0, 1, 0)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
MinimizeButton.TextColor3 = Color3.new(1, 1, 1)

local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Parent = MainFrame
ScrollFrame.Size = UDim2.new(1, -10, 1, -50)
ScrollFrame.Position = UDim2.new(0, 5, 0, 45)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 5
ScrollFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 200)

local ItemList = Instance.new("UIListLayout")
ItemList.Parent = ScrollFrame
ItemList.SortOrder = Enum.SortOrder.LayoutOrder
ItemList.Padding = UDim.new(0, 5)

-- Lista de itens (garantindo que "Plumber 1" esteja correto)
local items = {
    "Blue Gumdrop",
    "Purple Lolli",
    "Pink Lolli",
    "Candi Corn",
    "Plumber 1"
}

-- Função de compra
local function BuyItem(itemName)
    local RemoteEvents = ReplicatedStorage:WaitForChild("RemoteEvents")
    local BuyEvent = RemoteEvents:FindFirstChild("BuyItemCash") -- Evita erro se o evento não existir
    
    if BuyEvent then
        BuyEvent:FireServer(itemName)
        print(itemName.." comprado!")
    else
        warn("Evento BuyItemCash não encontrado! Não foi possível comprar "..itemName)
    end
end

-- Criando botões para cada item com efeito arco-íris
for _, item in ipairs(items) do
    local button = Instance.new("TextButton")
    button.Parent = ScrollFrame
    button.Size = UDim2.new(1, -10, 0, 40)
    button.Text = item
    button.Font = Enum.Font.GothamBold
    button.TextSize = 16
    button.TextColor3 = Color3.new(1, 1, 1)
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)

    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = button

    button.MouseButton1Click:Connect(function()
        print("Botão pressionado: "..item) -- Confirma que o botão está sendo clicado
        BuyItem(item) -- Chama a função de compra
    end)

    -- Efeito arco-íris nos botões
    local hue = 0
    RunService.Heartbeat:Connect(function(delta)
        hue = (hue + delta * 0.5) % 1
        button.TextColor3 = Color3.fromHSV(hue, 1, 1)
    end)
end

-- Efeito arco-íris e troca de nome no título
local hueTitle = 0

RunService.Heartbeat:Connect(function(delta)
    hueTitle = (hueTitle + delta * 0.5) % 1
    Title.TextColor3 = Color3.fromHSV(hueTitle, 1, 1)

    -- Alterna o nome do título constantemente
    if tick() % 2 < 1 then
        Title.Text = "tubergamer(1.2)"
    else
        Title.Text = "tubergamer(...)"
    end
end)

-- Variável de estado para minimizar/maximizar
local isMinimized = false

MinimizeButton.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        MainFrame.Size = UDim2.new(0, 300, 0, 40)
        ScrollFrame.Visible = false
    else
        MainFrame.Size = UDim2.new(0, 300, 0, 250)
        ScrollFrame.Visible = true
    end
end)

-- Função para permitir arrastar o Frame
local dragging, dragInput, dragStart, startPos

local function Update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        Update(input)
    end
end)

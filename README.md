-- MiniCity Secure UI com as funções do seu Farm de Mini City

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- Proteção e Anti-Múltiplas GUIs
if CoreGui:FindFirstChild("MiniCitySecureUI") then
    CoreGui.MiniCitySecureUI:Destroy()
end

local function proteger(obj)
    if syn and syn.protect_gui then
        syn.protect_gui(obj)
    elseif get_hidden_gui or gethui then
        local hiddenUI = get_hidden_gui and get_hidden_gui() or gethui()
        obj.Parent = hiddenUI
        return
    end
    obj.Parent = CoreGui
end

-- Criando a UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MiniCitySecureUI"
ScreenGui.ResetOnSpawn = false
proteger(ScreenGui)

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 240)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -120)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BackgroundTransparency = 0.2
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.Parent = ScreenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "Mini City - Farm Painel"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.Parent = MainFrame

-- Função: Salvar Posição
local SaveButton = Instance.new("TextButton")
SaveButton.Size = UDim2.new(0.8, 0, 0, 40)
SaveButton.Position = UDim2.new(0.1, 0, 0.3, 0)
SaveButton.Text = "Salvar Posição"
SaveButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
SaveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveButton.Font = Enum.Font.Gotham
SaveButton.TextSize = 16
SaveButton.Parent = MainFrame

local saveCorner = Instance.new("UICorner")
saveCorner.CornerRadius = UDim.new(0, 8)
saveCorner.Parent = SaveButton

SaveButton.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        _G.SavedPosition = character.HumanoidRootPart.Position
        print("[MiniCity] Posição salva!")
    end
end)

-- Função: Iniciar Farm
local FarmButton = Instance.new("TextButton")
FarmButton.Size = UDim2.new(0.8, 0, 0, 40)
FarmButton.Position = UDim2.new(0.1, 0, 0.55, 0)
FarmButton.Text = "Iniciar Farm Peça"
FarmButton.BackgroundColor3 = Color3.fromRGB(200, 80, 80)
FarmButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FarmButton.Font = Enum.Font.Gotham
FarmButton.TextSize = 16
FarmButton.Parent = MainFrame

local farmCorner = Instance.new("UICorner")
farmCorner.CornerRadius = UDim.new(0, 8)
farmCorner.Parent = FarmButton

FarmButton.MouseButton1Click:Connect(function()
    local function fireAllProximityPrompts()
        local objetosMissao = workspace:FindFirstChild("MapaGeral")
        if objetosMissao then
            objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
            if objetosMissao then
                objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
                if objetosMissao then
                    for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                        if prompt:IsA("ProximityPrompt") then
                            prompt.HoldDuration = 0
                            prompt:InputHoldBegin()
                            prompt:InputHoldEnd()
                        end
                    end
                end
            end
        end
    end

    local function findPlayerProximityPrompt()
        local playerName = player.Name
        local objetosMissao = workspace:FindFirstChild("MapaGeral")
        if objetosMissao then
            objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
            if objetosMissao then
                objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
                if objetosMissao then
                    for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                        if prompt:IsA("ProximityPrompt") and prompt.Name == playerName then
                            return prompt
                        end
                    end
                end
            end
        end
        return nil
    end

    while true do
        if not _G.SavedPosition then break end
        local character = player.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then break end

        fireAllProximityPrompts()
        character.HumanoidRootPart.CFrame = CFrame.new(_G.SavedPosition)

        task.wait(0.35)

        game:GetService("ReplicatedStorage"):WaitForChild("RemoteNovos"):WaitForChild("trabalhos"):FireServer("missaoPECAS")

        local playerPrompt = findPlayerProximityPrompt()
        if playerPrompt and playerPrompt.Parent then
            character.HumanoidRootPart.CFrame = CFrame.new(playerPrompt.Parent.Position)
        end

        task.wait(0.2)
    end
end)

-- Arrastar o painel
local dragging, dragInput, dragStart, startPos
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
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

getgenv().cuppink = true

if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Player = game.Players.LocalPlayer

local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    local MainFrame = Instance.new("Frame")
    local UICorner = Instance.new("UICorner")
    local BrandingLabel = Instance.new("TextLabel") 
    local MinimizeButton = Instance.new("TextButton")
    local CloseButton = Instance.new("TextButton")
    local SidebarFrame = Instance.new("Frame")
    local UICornerSidebar = Instance.new("UICorner")
    local GeneralButton = Instance.new("TextButton")

    ScreenGui.Parent = Player.PlayerGui

    -- Main Frame Styling (more modern)
    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 28) -- Dark background
    MainFrame.BackgroundTransparency = 0.2
    MainFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
    MainFrame.Size = UDim2.new(0, 600, 0, 450)
    MainFrame.Active = true
    MainFrame.Draggable = true

    UICorner.CornerRadius = UDim.new(0, 20)  -- Rounded corners
    UICorner.Parent = MainFrame

    -- Branding label
    BrandingLabel.Parent = MainFrame
    BrandingLabel.Text = "Ibicatu SP - Made by: easyFIFA17 & Nobre_CK"
    BrandingLabel.Font = Enum.Font.GothamBold
    BrandingLabel.TextSize = 18
    BrandingLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    BrandingLabel.BackgroundTransparency = 1
    BrandingLabel.Size = UDim2.new(1, -20, 0, 40)
    BrandingLabel.Position = UDim2.new(0, 10, 0, 5) 
    BrandingLabel.TextXAlignment = Enum.TextXAlignment.Center

    -- Minimize Button
    MinimizeButton.Parent = MainFrame
    MinimizeButton.Text = "-"
    MinimizeButton.Font = Enum.Font.Gotham
    MinimizeButton.TextSize = 24
    MinimizeButton.BackgroundTransparency = 1
    MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeButton.Size = UDim2.new(0, 30, 0, 25)
    MinimizeButton.Position = UDim2.new(1, -70, 0, 10)
    MinimizeButton.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
    end)

    -- Close Button
    CloseButton.Parent = MainFrame
    CloseButton.Text = "x"
    CloseButton.Font = Enum.Font.Gotham
    CloseButton.TextSize = 22
    CloseButton.BackgroundTransparency = 1
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Size = UDim2.new(0, 30, 0, 25)
    CloseButton.Position = UDim2.new(1, -40, 0, 10)
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

    -- Sidebar Frame
    SidebarFrame.Parent = MainFrame
    SidebarFrame.BackgroundColor3 = Color3.fromRGB(44, 44, 44)  -- Sidebar darker color
    SidebarFrame.Size = UDim2.new(0, 120, 1, 0)

    UICornerSidebar.CornerRadius = UDim.new(0, 20)  -- Rounded corners for sidebar
    UICornerSidebar.Parent = SidebarFrame

    -- Sidebar Button for General
    GeneralButton.Parent = SidebarFrame
    GeneralButton.Text = "General"
    GeneralButton.Font = Enum.Font.Gotham
    GeneralButton.TextSize = 16
    GeneralButton.BackgroundTransparency = 1
    GeneralButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    GeneralButton.Size = UDim2.new(1, -20, 0, 50)
    GeneralButton.Position = UDim2.new(0, 10, 0, 30)
    GeneralButton.TextXAlignment = Enum.TextXAlignment.Center

    local function createButton(text, positionY, callback)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(0, 420, 0, 50)
        Button.Position = UDim2.new(0, 140, positionY, 0)
        Button.Text = text
        Button.Font = Enum.Font.Gotham
        Button.TextSize = 16
        Button.BackgroundColor3 = Color3.fromRGB(54, 54, 54)  -- Slightly lighter background for buttons
        Button.BackgroundTransparency = 0.4
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.TextXAlignment = Enum.TextXAlignment.Center
        Button.Parent = MainFrame

        local UICorner = Instance.new("UICorner")
        UICorner.CornerRadius = UDim.new(0, 15)  -- More rounded corners for buttons
        UICorner.Parent = Button

        Button.MouseButton1Click:Connect(callback)

        return Button
    end

    local copiedModule
    local carName
    local containerName = Player.Name .. "sCar"
    local moduleName = "A-Chassis Tune"

    local function findModule(owner)
        local container = workspace:FindFirstChild(owner .. "sCar")
        if not container then return nil end
        return container:FindFirstChild(moduleName)
    end

    local function getCarName(owner)
        local container = workspace:FindFirstChild(owner .. "sCar")
        if container then
            local carNameValue = container:FindFirstChild("CarName")
            if carNameValue then return carNameValue.Value end
        end
        return "Desconhecido"
    end

    createButton("Excluir Placa do carro", 0.6, function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            local body = container:FindFirstChild("Body")
            if body then
                local placaBR = body:FindFirstChild("PlacaBR")
                if placaBR then
                    placaBR:Destroy()
                end
            end
        end
    end)

    -- Modificação para copiar motor de outros carros
    createButton("Copiar motor carro (de outro jogador)", 0.1, function()
        -- Exibe uma lista de jogadores para copiar o motor de um dos carros
        local playerList = {}
        for _, otherPlayer in pairs(game.Players:GetPlayers()) do
            if otherPlayer ~= Player then
                table.insert(playerList, otherPlayer)
            end
        end

        -- Criação de uma interface simples para o jogador escolher outro jogador para copiar o motor
        local SelectPlayerGui = Instance.new("ScreenGui")
        local Frame = Instance.new("Frame")
        local UICorner = Instance.new("UICorner")
        local ListLayout = Instance.new("UIListLayout")
        local CloseButton = Instance.new("TextButton")
        local ScrollFrame = Instance.new("ScrollingFrame")

        SelectPlayerGui.Parent = Player.PlayerGui
        Frame.Parent = SelectPlayerGui
        Frame.Size = UDim2.new(0, 300, 0, 300)
        Frame.Position = UDim2.new(0.35, 0, 0.35, 0)
        Frame.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
        UICorner.Parent = Frame

        -- Scrollable Player List
        ScrollFrame.Parent = Frame
        ScrollFrame.Size = UDim2.new(1, -20, 1, -50)
        ScrollFrame.Position = UDim2.new(0, 10, 0, 50)
        ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, #playerList * 40)
        ScrollFrame.ScrollBarThickness = 10
        ListLayout.Parent = ScrollFrame
        ListLayout.Padding = UDim.new(0, 10)

        -- Close button inside player list
        CloseButton.Parent = Frame
        CloseButton.Text = "Fechar"
        CloseButton.Font = Enum.Font.Gotham
        CloseButton.TextSize = 18
        CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        CloseButton.BackgroundTransparency = 1
        CloseButton.Position = UDim2.new(1, -70, 1, -30)
        CloseButton.MouseButton1Click:Connect(function()
            SelectPlayerGui:Destroy()
        end)

        for _, otherPlayer in ipairs(playerList) do
            local PlayerButton = Instance.new("TextButton")
            PlayerButton.Size = UDim2.new(0, 280, 0, 40)
            PlayerButton.Text = otherPlayer.Name
            PlayerButton.Font = Enum.Font.Gotham
            PlayerButton.TextSize = 16
            PlayerButton.BackgroundColor3 = Color3.fromRGB(54, 54, 54)
            PlayerButton.BackgroundTransparency = 0.4
            PlayerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            PlayerButton.Parent = ScrollFrame

            PlayerButton.MouseButton1Click:Connect(function()
                local otherModule = findModule(otherPlayer.Name)
                if otherModule then
                    copiedModule = otherModule:Clone()
                    carName = getCarName(otherPlayer.Name)
                    SelectPlayerGui:Destroy()
                end
            end)
        end
    end)

    createButton("Colar motor", 0.2, function()
        if copiedModule then
            local container = workspace:FindFirstChild(containerName)
            if container then
                local existingModule = container:FindFirstChild(moduleName)
                if existingModule then existingModule:Destroy() end
                copiedModule.Parent = container
                copiedModule.Name = moduleName
                copiedModule = nil
            end
        end
    end)

    createButton("Aumentar som turbo", 0.3, function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            local engine = container:FindFirstChild("Body"):FindFirstChild("Engine")
            if engine then
                local turboSound = engine:FindFirstChild("Turbo")
                if turboSound then
                    turboSound.Volume = 0.6
                end
            end
        end
    end)

    createButton("Rebaixar Suspensao (nao visivel para os outros)", 0.4, function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            local wheels = container:FindFirstChild("Wheels")
            if wheels then
                local FL = wheels:FindFirstChild("FL")
                local FR = wheels:FindFirstChild("FR")
                local RL = wheels:FindFirstChild("RL")
                local RR = wheels:FindFirstChild("RR")

                local function adjustWheelSuspension(wheel)
                    local suspension = wheel:FindFirstChild("SuspensionGeometry")
                    if suspension then
                        local spring = suspension:FindFirstChild("Spring")
                        if spring and spring:IsA("SpringConstraint") then
                            spring.MaxLength = 1.82
                        end
                    end
                end

                if FL then adjustWheelSuspension(FL) end
                if FR then adjustWheelSuspension(FR) end
                if RL then adjustWheelSuspension(RL) end
                if RR then adjustWheelSuspension(RR) end
            end
        end
    end)

    createButton("Aumentar Estabilidade (Gravidade) Rebaixa Visivel", 0.5, function()
        game.Workspace.Gravity = 400
    end)

    Player.CharacterAdded:Connect(function()
        ScreenGui.Enabled = true
    end)

    game:GetService("UserInputService").InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.LeftControl then
            MainFrame.Visible = true
        end
    end)
end

createGUI()

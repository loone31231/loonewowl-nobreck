local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Player = game.Players.LocalPlayer

-- Inicialização da GUI Fluent
local Window = Fluent:CreateWindow({
    Title = "Ibicatu SP🚗 ",
    SubTitle = "by Nobre_CK & easyFIFA17",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- Variáveis principais
local copiedModule = nil
local containerName = Player.Name .. "sCar"
local moduleName = "A-Chassis Tune"

-- Função para encontrar um módulo específico em um carro
local function findModule(owner)
    local container = workspace:FindFirstChild(owner .. "sCar")
    if container then
        return container:FindFirstChild(moduleName)
    end
    return nil
end

-- Função para buscar jogadores com carros
local function getPlayersWithCars()
    local playersWithCars = {}
    for _, otherPlayer in ipairs(game.Players:GetPlayers()) do
        if otherPlayer ~= Player then
            local container = workspace:FindFirstChild(otherPlayer.Name .. "sCar")
            if container then
                table.insert(playersWithCars, otherPlayer.Name)
            end
        end
    end
    return playersWithCars
end

-- Dropdown para copiar motor de outro jogador
Tabs.Main:AddDropdown("Copiar motor de outro jogador", {
    Title = "Copiar motor de outro jogador",
    Values = getPlayersWithCars(),
    Multi = false,
    Default = nil,
    Callback = function(selectedPlayerName)
        if selectedPlayerName then
            local originalModule = findModule(selectedPlayerName)
            if originalModule then
                copiedModule = originalModule:Clone()
                Fluent:Notify({
                    Title = "Motor Copiado",
                    Content = "O motor do carro de " .. selectedPlayerName .. " foi copiado com sucesso.",
                    Duration = 5
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Não foi possível encontrar o carro do jogador selecionado.",
                    Duration = 5
                })
            end
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Nenhum jogador selecionado.",
                Duration = 5
            })
        end
    end
})

-- Botão para copiar motor do próprio carro (sem notificação)
Tabs.Main:AddButton({
    Title = "Copiar motor do seu carro",
    Description = "Copiar o motor do seu próprio carro.",
    Callback = function()
        local originalModule = findModule(Player.Name)
        if originalModule then
            copiedModule = originalModule:Clone()
        end
    end
})

-- Botão para colar motor copiado no próprio carro (sem notificação)
Tabs.Main:AddButton({
    Title = "Colar motor no carro",
    Description = "Colar o motor copiado no seu carro.",
    Callback = function()
        if copiedModule then
            local container = workspace:FindFirstChild(containerName)
            if container then
                local existingModule = container:FindFirstChild(moduleName)
                if existingModule then
                    existingModule:Destroy()
                end
                copiedModule.Parent = container
                copiedModule.Name = moduleName
                copiedModule = nil
            end
        end
    end
})

-- Botão para excluir a placa do carro
Tabs.Main:AddButton({
    Title = "Excluir Placa do Carro",
    Description = "Excluir a placa do seu carro (CLIENT)",
    Callback = function()
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
    end
})

-- Entrada para editar a placa do carro
Tabs.Main:AddInput("Editar Identifier da Placa", {
    Title = "Editar placa do carro",
    Default = default,
    Placeholder = default,
    Description = "(CLIENT)",
    Numeric = false,
    Finished = false,
    Callback = function(Value)
        local container = workspace:FindFirstChild(Player.Name .. "sCar")
        if container then
            local body = container:FindFirstChild("Body")
            if body then
                local placas = {body:FindFirstChild("PlacaBR"), body:FindFirstChild("PlacaBRdip")}
                for _, placa in ipairs(placas) do
                    if placa and placa:FindFirstChild("Plates") then
                        local plates = placa.Plates
                        for _, plate in ipairs(plates:GetChildren()) do
                            if plate:FindFirstChild("SGUI") and plate.SGUI:FindFirstChild("Identifier") then
                                plate.SGUI.Identifier.Text = Value
                            end
                        end
                    end
                end
            end
        end
    end
})

-- Slider para ajustar suspensão
Tabs.Main:AddSlider("Suspensão", {
    Title = "Ajustar Suspensão",
    Default = 1.6,
    Description = "Ajustar altura da suspensao";
    Min = 1.2,
    Max = 2.0,
    Rounding = 2,
    Callback = function(value)
        local container = workspace:FindFirstChild(containerName)
        if container and container:FindFirstChild("Wheels") then
            for _, wheel in ipairs(container.Wheels:GetChildren()) do
                if wheel:FindFirstChild("SuspensionGeometry") then
                    local spring = wheel.SuspensionGeometry:FindFirstChild("Spring")
                    if spring and spring:IsA("SpringConstraint") then
                        spring.MinLength = value
                        spring.MaxLength = value + 0.03
                    end
                end
            end
        end
    end
})

-- Botão para ajustar gravidade
Tabs.Main:AddButton({
    Title = "Ajustar Gravidade",
    Description = "Ajusta a gravidade para aumentar a estabilidade.",
    Callback = function()
        game.Workspace.Gravity = 400
    end
})

-- Botão para ajustar som do turbo
Tabs.Main:AddButton({
    Title = "Aumentar som do turbo",
    Description = "(CLIENT)",
    Callback = function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            local body = container:FindFirstChild("Body")
            if body and body:FindFirstChild("Engine") then
                local turboSound = body.Engine:FindFirstChild("Turbo")
                if turboSound and turboSound:IsA("Sound") then
                    turboSound.Volume = 1
                end
            end
        end
    end
})

-- Notificação de carregamento
Fluent:Notify({
    Title = "Fluent",
    Content = "O script foi carregado com sucesso.",
    Duration = 8
})

-- Configuração de salvar e carregar
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:SetFolder("FluentScriptHub/specific-game")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

Window:SelectTab(1)

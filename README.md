local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Definindo o player
local Player = game.Players.LocalPlayer

-- Função de inicialização da GUI Fluent
local Window = Fluent:CreateWindow({
    Title = "Ibicatu Sp  1.0",
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

local Options = Fluent.Options

-- Funções para manipular o carro
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

local function adjustAndLockSounds(container)
    local body = container:FindFirstChild("Body")
    if body then
        local engine = body:FindFirstChild("Engine")
        if engine then
            local turboSound = engine:FindFirstChild("Turbo")
            if turboSound and turboSound:IsA("Sound") then
                turboSound.Volume = 0.6
                turboSound.Changed:Connect(function(property)
                    if property == "Volume" and turboSound.Volume ~= 0.6 then
                        turboSound.Volume = 0.6
                    end
                end)
            end
            local bovSound = engine:FindFirstChild("Bov")
            if bovSound and bovSound:IsA("Sound") then
                bovSound.Volume = 0.5
            end
        end
    end
end

-- Dropdown para selecionar outro jogador e copiar motor (primeiro)
local Dropdown = Tabs.Main:AddDropdown("Selecionar jogador para copiar motor", { 
    Title = "Copiar motor de jogador",
    Values = {},  -- Vai ser preenchido dinamicamente com a lista de jogadores
    Multi = false,
    Default = Default,
    Callback = function(selectedPlayerName)
        print("Você selecionou: " .. selectedPlayerName)
        -- Copiar motor do jogador selecionado
        local otherModule = findModule(selectedPlayerName)
        if otherModule then
            copiedModule = otherModule:Clone()
            carName = getCarName(selectedPlayerName)
            print("Motor copiado de: " .. carName)
        else
            print("Não foi possível encontrar o carro do jogador: " .. selectedPlayerName)
        end
    end
})

-- Função para atualizar a lista de jogadores no Dropdown
local function updatePlayerList()
    local playerNames = {}
    for _, otherPlayer in ipairs(game.Players:GetPlayers()) do
        if otherPlayer ~= Player then
            table.insert(playerNames, otherPlayer.Name)
        end
    end
    Dropdown:SetValues(playerNames)  -- Atualiza os valores do Dropdown com a lista de jogadores
end

-- Chamar a função de atualização da lista de jogadores quando o script começar
updatePlayerList()

-- Copiar motor do seu próprio carro (segunda opção)
Tabs.Main:AddButton({
    Title = "Copiar motor do seu carro",
    Description = "Copiar o motor do seu próprio carro.",
    Callback = function()
        local originalModule = findModule(Player.Name)
        if originalModule then
            copiedModule = originalModule:Clone()
            carName = getCarName(Player.Name)
            print("Motor copiado do seu carro: " .. carName)
        end
    end
})

-- Função para colar o motor copiado no seu carro
Tabs.Main:AddButton({
    Title = "Colar motor no carro",
    Description = "Colar o motor copiado no seu carro.",
    Callback = function()
        if copiedModule then
            local container = workspace:FindFirstChild(containerName)
            if container then
                local existingModule = container:FindFirstChild(moduleName)
                if existingModule then existingModule:Destroy() end
                copiedModule.Parent = container
                copiedModule.Name = moduleName
                adjustAndLockSounds(container)
                copiedModule = nil
                print("Motor colado no carro.")
            end
        end
    end
})

-- Função para excluir a placa do carro
Tabs.Main:AddButton({
    Title = "Excluir Placa do Carro",
    Description = "Excluir a placa do seu carro.",
    Callback = function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            local body = container:FindFirstChild("Body")
            if body then
                local placaBR = body:FindFirstChild("PlacaBR")
                if placaBR then
                    placaBR:Destroy()
                    print("Placa do carro excluída.")
                end
            end
        end
    end
})

-- Função para ajustar o som do turbo
Tabs.Main:AddButton({
    Title = "Ajustar som do turbo",
    Description = "Ajusta o som do turbo no carro.",
    Callback = function()
        local container = workspace:FindFirstChild(containerName)
        if container then
            adjustAndLockSounds(container)
            print("Som do turbo ajustado.")
        end
    end
})

-- Slider para ajustar a suspensão do carro
Tabs.Main:AddSlider("Suspensão", {
    Title = "Ajustar Suspensão",
    Description = "Ajuste a altura da suspensão do carro.",
    Default = 1.6,  -- Valor inicial para o MinLength
    Min = 1.2,      -- Valor mínimo para o MinLength
    Max = 2.0,      -- Valor máximo para o MinLength
    Rounding = 2,    -- Arredondar para 2 casas decimais
    Callback = function(value)
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
                            spring.MinLength = value               -- Ajusta o MinLength com o valor do slider
                            spring.MaxLength = value + 0.03        -- Ajusta o MaxLength, sempre 0.03 a mais que o MinLength
                        end
                    end
                end

                if FL then adjustWheelSuspension(FL) end
                if FR then adjustWheelSuspension(FR) end
                if RL then adjustWheelSuspension(RL) end
                if RR then adjustWheelSuspension(RR) end
            end
            print("Suspensão do carro ajustada para: " .. value)
        end
    end
})

-- Função para ajustar a gravidade
Tabs.Main:AddButton({
    Title = "Ajustar Gravidade",
    Description = "Ajusta a gravidade para aumentar a estabilidade.",
    Callback = function()
        game.Workspace.Gravity = 400
        print("Gravidade ajustada.")
    end
})

-- Configuração e Notificação de Carregamento
Fluent:Notify({
    Title = "Fluent",
    Content = "O script foi carregado com sucesso.",
    Duration = 8
})

-- Configuração para salvar e carregar
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:SetFolder("FluentScriptHub/specific-game")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

Window:SelectTab(1)

-- Exemplos de manipulação e interação com a GUI fluente
Fluent:Notify({
    Title = "Exemplo",
    Content = "Carregamento do script finalizado.",
    Duration = 5
})

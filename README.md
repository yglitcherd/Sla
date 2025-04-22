-- Carregar Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criar Janela
local Window = Rayfield:CreateWindow({
    Name = "Auto Stats",
    LoadingTitle = "Loading...",
    LoadingSubtitle = "By Dex",
    ConfigurationSaving = {Enabled = true, FileName = "AutoTeleportHub"},
})

-- Tabs
local FarmTab = Window:CreateTab("Auto Farm", "rocket")
local ConfigTab = Window:CreateTab("Config", "settings")

-- Lista de Stats
local Stats = {
    { Name = "Auto Multiplier", Path = "Multiplier", BlockedColors = {Color3.fromRGB(106, 0, 0)} },
    { Name = "Auto Rebirth", Path = "Rebirth", BlockedColors = {Color3.fromRGB(1, 24, 125)} },
    { Name = "Auto Prestige", Path = "Prestige", BlockedColors = {Color3.fromRGB(85, 20, 129)} },
    { Name = "Auto Ultra", Path = "Ultra", BlockedColors = {Color3.fromRGB(121, 114, 32)} },
    { Name = "Auto Mega", Path = "Mega", BlockedColors = {Color3.fromRGB(0, 104, 116)} },
    { Name = "Auto Omega", Path = "Omega", BlockedColors = {Color3.fromRGB(107, 48, 0)} },
    { Name = "Auto Quantum", Path = "Quantum", BlockedColors = {Color3.fromRGB(118, 72, 120)} },
    { Name = "Auto Divine", Path = "Divine", BlockedColors = {Color3.fromRGB(108, 108, 108)} },
    { Name = "Auto Celestial", Path = "Celestial", BlockedColors = {Color3.fromRGB(0, 112, 69)} },
    { Name = "Auto Dune", Path = "Dune", BlockedColors = {Color3.fromRGB(112, 76, 29)} },
    { Name = "Auto Mirage", Path = "Mirage", BlockedColors = {Color3.fromRGB(27, 104, 48)} },
    { Name = "Auto Oasis", Path = "Oasis", BlockedColors = {Color3.fromRGB(27, 72, 107)} },
    { Name = "Auto Sandstorm", Path = "Sandstorm", BlockedColors = {Color3.fromRGB(131, 131, 33)} },
    { Name = "Auto Pharaoh", Path = "Pharaoh", BlockedColors = {Color3.fromRGB(138, 35, 135)} },
    { Name = "Auto Pyramid", Path = "Pyramid", BlockedColors = {Color3.fromRGB(65, 35, 139)} },
    { Name = "Auto Frost", Path = "Frost", BlockedColors = {Color3.fromRGB(28, 105, 108)} },
    { Name = "Auto Blizzard", Path = "Blizzard", BlockedColors = {Color3.fromRGB(130, 61, 33)} },
    { Name = "Auto Glacier", Path = "Glacier", BlockedColors = {Color3.fromRGB(30, 117, 79)} },
    { Name = "Auto Avalanche", Path = "Avalanche", BlockedColors = {Color3.fromRGB(113, 29, 29)} },
    { Name = "Auto Iceberg", Path = "Iceberg", BlockedColors = {Color3.fromRGB(31, 58, 121)} },
    { Name = "Auto Snowflake", Path = "Snowflake", BlockedColors = {Color3.fromRGB(125, 125, 125)} },
}

local Toggles = {}
local Delays = {} -- Delay individual de cada stat

-- Função para verificar cor bloqueada
local function IsBlocked(color, blockedList)
    for _, blockedColor in ipairs(blockedList) do
        if color == blockedColor then
            return true
        end
    end
    return false
end

-- Função para pegar o melhor botão (maior número e liberado)
local function GetBestButton(stat)
    local Best, Max = nil, -1
    -- Buscar dinamicamente o path correto
    for _, btn in pairs(workspace.__GAME_CONTENT:GetChildren()) do
        if btn.Name:match(stat.Path) then  -- Checa se o nome do botão corresponde ao Path
            for _, button in pairs(btn:GetChildren()) do
                if button.Name == "Touch" then
                    local num = tonumber(button.Name)
                    local top = button:FindFirstChild("Model") and button.Model:FindFirstChild("Top2")
                    if num and top and not IsBlocked(top.Color, stat.BlockedColors) and num > Max then
                        Best, Max = button, num
                    end
                end
            end
        end
    end
    return Best
end

-- Atualizar lista de stats ativos
local ActiveStats = {}
local function UpdateActiveStats()
    ActiveStats = {}
    for index, stat in ipairs(Stats) do
        if Toggles[stat.Name] then
            table.insert(ActiveStats, {Index = index, Stat = stat})
        end
    end
end

-- Loop principal
local CurrentStatIndex = 1
local function StartFarm()
    task.spawn(function()
        while true do
            if #ActiveStats == 0 then
                task.wait(1)
            else
                local selected = ActiveStats[CurrentStatIndex]
                if selected then
                    local stat = selected.Stat
                    local hrp = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    local delay = Delays[stat.Name] or 1
                    local timer = 0
                    while timer < delay do
                        if hrp then
                            local btn = GetBestButton(stat)
                            if btn then
                                hrp.CFrame = btn.Touch.CFrame -- Teleporte diretamente para o Touch
                            end
                        end
                        task.wait(0.5) -- Pequeno intervalo para spam menor
                        timer = timer + 0.5
                    end
                    -- Avança pro próximo stat
                    CurrentStatIndex = CurrentStatIndex + 1
                    if CurrentStatIndex > #ActiveStats then
                        CurrentStatIndex = 1
                    end
                end
            end
        end
    end)
end

-- Criar Toggles no FarmTab
for _, stat in ipairs(Stats) do
    FarmTab:CreateToggle({
        Name = stat.Name,
        CurrentValue = false,
        Callback = function(state)
            Toggles[stat.Name] = state
            UpdateActiveStats()
        end,
    })
end

-- Criar Inputs de Delay no ConfigTab
for _, stat in ipairs(Stats) do
    ConfigTab:CreateInput({
        Name = "Delay "..stat.Name.." (segundos)",
        PlaceholderText = "1",
        CurrentValue = "1",
        Callback = function(value)
            local num = tonumber(value)
            if num then
                Delays[stat.Name] = num
            end
        end,
    })
end

-- Começar o Farm
StartFarm()

-- LocalScript (coloque em StarterPlayerScripts)
-- Script de GUI persistente e sistema de registro de "features" (sem funcionalidades de cheat)
-- Autor exibido: gggwyw6-rgb

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Flag persistente para lembrar se o usuário fechou o script permanentemente
local CLOSED_FLAG_NAME = "MyScriptClosedFlag"
local closedFlag = player:FindFirstChild(CLOSED_FLAG_NAME)
if not closedFlag then
    closedFlag = Instance.new("BoolValue")
    closedFlag.Name = CLOSED_FLAG_NAME
    closedFlag.Value = false
    closedFlag.Parent = player
end

-- Se o usuário já pediu para fechar, não cria a GUI nem ativa nada
if closedFlag.Value then
    return
end

-- Sistema simples para registrar/desregistrar features do script
local Features = {}
local function registerFeature(name, disableFunc)
    Features[name] = {
        disable = disableFunc
    }
end
local function disableAllFeatures()
    for name, feature in pairs(Features) do
        local ok, err = pcall(function()
            if feature.disable then
                feature.disable()
            end
        end)
        if not ok then
            warn("Erro ao desabilitar feature " .. tostring(name) .. ": " .. tostring(err))
        end
    end
    Features = {}
end

-- Cria ou reutiliza a ScreenGui
local screenGui = playerGui:FindFirstChild("MyPvPScriptGui")
if screenGui and screenGui:IsA("ScreenGui") then
    -- Se já existe, reutiliza (mas mantém ResetOnSpawn configurado)
else
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MyPvPScriptGui"
    -- importante: evita que a GUI seja removida automaticamente ao respawn
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui
end

-- Se já existe um container principal, reutiliza; senão cria
local mainFrame = screenGui:FindFirstChild("MainFrame")
if not mainFrame then
    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 320, 0, 120)
    mainFrame.Position = UDim2.new(0.7, 0, 0.05, 0)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BorderSizePixel = 0
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.Parent = screenGui

    -- Título com seu nome
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -8, 0, 30)
    title.Position = UDim2.new(0, 4, 0, 4)
    title.BackgroundTransparency = 1
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Text = "Script by gggwyw6-rgb" -- altere se quiser outro nome
    title.TextColor3 = Color3.fromRGB(220,220,220)
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 18
    title.Parent = mainFrame

    -- Área de conteúdo (ex: toggles, status)
    local content = Instance.new("Frame")
    content.Name = "Content"
    content.Size = UDim2.new(1, -8, 1, -46)
    content.Position = UDim2.new(0, 4, 0, 38)
    content.BackgroundTransparency = 1
    content.Parent = mainFrame

    -- Botões: Ocultar e Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseButton"
    closeBtn.Size = UDim2.new(0, 80, 0, 28)
    closeBtn.Position = UDim2.new(1, -86, 0, 4)
    closeBtn.AnchorPoint = Vector2.new(0, 0)
    closeBtn.Text = "Fechar"
    closeBtn.Font = Enum.Font.SourceSans
    closeBtn.TextSize = 16
    closeBtn.BackgroundColor3 = Color3.fromRGB(170, 40, 40)
    closeBtn.TextColor3 = Color3.new(1,1,1)
    closeBtn.Parent = mainFrame

    local hideBtn = Instance.new("TextButton")
    hideBtn.Name = "HideButton"
    hideBtn.Size = UDim2.new(0, 80, 0, 28)
    hideBtn.Position = UDim2.new(1, -176, 0, 4)
    hideBtn.Text = "Ocultar"
    hideBtn.Font = Enum.Font.SourceSans
    hideBtn.TextSize = 16
    hideBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    hideBtn.TextColor3 = Color3.new(1,1,1)
    hideBtn.Parent = mainFrame

    -- Botão que aparece quando a GUI está oculta para permitir mostrar de novo
    local showButton = screenGui:FindFirstChild("ShowButton")
    if not showButton then
        showButton = Instance.new("TextButton")
        showButton.Name = "ShowButton"
        showButton.Size = UDim2.new(0, 80, 0, 28)
        showButton.Position = UDim2.new(1, -86, 0, 4)
        showButton.AnchorPoint = Vector2.new(0, 0)
        showButton.Text = "Mostrar"
        showButton.Font = Enum.Font.SourceSans
        showButton.TextSize = 16
        showButton.BackgroundColor3 = Color3.fromRGB(80, 160, 80)
        showButton.TextColor3 = Color3.new(1,1,1)
        showButton.Visible = false
        showButton.Parent = screenGui
    end

    -- Conexões (guarda para desconectar ao fechar)
    local connections = {}

    local function hideGui()
        mainFrame.Visible = false
        showButton.Visible = true
    end
    local function showGui()
        mainFrame.Visible = true
        showButton.Visible = false
    end

    -- Botão ocultar
    table.insert(connections, hideBtn.MouseButton1Click:Connect(function()
        hideGui()
    end))

    -- Botão mostrar
    table.insert(connections, showButton.MouseButton1Click:Connect(function()
        showGui()
    end))

    -- Botão fechar: marca a flag persistente, desabilita features, remove GUI
    table.insert(connections, closeBtn.MouseButton1Click:Connect(function()
        -- Marca que o usuário fechou o script (persistente entre respawns)
        closedFlag.Value = true

        -- Desabilita todas as features registradas
        disableAllFeatures()

        -- Desconecta conexões locais se houver
        for _, con in ipairs(connections) do
            if con and con.Disconnect then
                pcall(function() con:Disconnect() end)
            end
        end

        -- Remove GUI
        pcall(function() screenGui:Destroy() end)
    end))

    -- Exemplo de feature registrada (segura e inofensiva)
    -- Substitua/adicione com funcionalidades legítimas do seu próprio jogo
    local exampleConnection
    registerFeature("ExampleFeature", function()
        -- função para desabilitar a feature (exemplo)
        if exampleConnection and exampleConnection.Disconnect then
            pcall(function() exampleConnection:Disconnect() end)
            exampleConnection = nil
        end
        print("ExampleFeature desabilitada")
    end)

    -- Exemplo de ativação de feature: aqui só mostra um texto no content
    local exampleLabel = Instance.new("TextLabel")
    exampleLabel.Size = UDim2.new(1, 0, 0, 24)
    exampleLabel.Position = UDim2.new(0, 0, 0, 0)
    exampleLabel.BackgroundTransparency = 1
    exampleLabel.Text = "Features seguras ativas: ExampleFeature"
    exampleLabel.TextColor3 = Color3.fromRGB(200,200,200)
    exampleLabel.Font = Enum.Font.SourceSans
    exampleLabel.TextSize = 14
    exampleLabel.Parent = content

    -- (Se houvesse conexões relacionadas à feature, armazene em exampleConnection e desconecte na função de disable acima.)
end

-- NOTE: Este script NÃO contém e NÃO auxiliará na criação de cheats (aimbot, ESP para vantagem em jogos multiplayer etc).
-- Use o sistema de registro acima para adicionar e remover features legítimas e seguras.

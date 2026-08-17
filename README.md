-- LocalScript - Coloque este script em StarterPlayerScripts

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = Workspace.CurrentCamera

-- Configurações visuais do menu
local UI_OFFSET = Vector2.new(10, 10) -- Espaçamento do canto da tela
local UI_BUTTON_HEIGHT = 30
local UI_BUTTON_WIDTH = 150
local UI_SPACING = 5
local UI_COLOR = Color3.fromRGB(50, 50, 50) -- Cor de fundo dos elementos da UI
local UI_TEXT_COLOR = Color3.fromRGB(255, 255, 255) -- Cor do texto dos elementos da UI
local UI_BORDER_COLOR = Color3.fromRGB(100, 100, 100) -- Cor da borda dos elementos da UI
local UI_BORDER_SIZE = 2

-- Configurações do ESP
local ESP_ENABLED = false
local ESP_NAME_ENABLED = true
local ESP_BOX_ENABLED = true
local ESP_HEALTH_BAR_ENABLED = true
local ESP_DISTANCE_ENABLED = true
local ESP_COLOR_PLAYER = Color3.fromRGB(255, 0, 0) -- Cor do ESP para outros jogadores
local ESP_COLOR_FRIEND = Color3.fromRGB(0, 255, 0) -- Cor do ESP para amigos (se implementado)
local ESP_MAX_DISTANCE = 500 -- Distância máxima para renderizar o ESP

-- Variáveis de estado da UI
local UI_OPEN = false
local UI_FRAME -- Frame principal da UI
local ESP_TOGGLE_BUTTON -- Botão para ligar/desligar o ESP

-- Dicionário para armazenar os visuais do ESP de cada jogador
local espVisuals = {}

-- Função para criar um novo Frame para a UI
local function createUIFrame(parent, position, size, color, name)
    local frame = Instance.new("Frame")
    frame.Name = name or "UIFrame"
    frame.Size = UDim2.new(0, size.X, 0, size.Y)
    frame.Position = UDim2.new(0, position.X, 0, position.Y)
    frame.BackgroundColor3 = color
    frame.BorderSizePixel = UI_BORDER_SIZE
    frame.BorderColor3 = UI_BORDER_COLOR
    frame.Parent = parent
    return frame
end

-- Função para criar um novo TextLabel
local function createTextLabel(parent, position, size, text, textColor, bgColor, name, font, textSize)
    local label = Instance.new("TextLabel")
    label.Name = name or "TextLabel"
    label.Size = UDim2.new(0, size.X, 0, size.Y)
    label.Position = UDim2.new(0, position.X, 0, position.Y)
    label.BackgroundColor3 = bgColor or UI_COLOR
    label.BackgroundTransparency = (bgColor == nil) and 1 or 0
    label.TextColor3 = textColor or UI_TEXT_COLOR
    label.TextScaled = false
    label.TextSize = textSize or 14
    label.Font = font or Enum.Font.SourceSans
    label.Text = text
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextYAlignment = Enum.TextYAlignment.Center
    label.BorderSizePixel = 0
    label.Parent = parent
    return label
end

-- Função para criar um novo TextButton
local function createTextButton(parent, position, size, text, textColor, bgColor, name, font, textSize)
    local button = Instance.new("TextButton")
    button.Name = name or "TextButton"
    button.Size = UDim2.new(0, size.X, 0, size.Y)
    button.Position = UDim2.new(0, position.X, 0, position.Y)
    button.BackgroundColor3 = bgColor or UI_COLOR
    button.TextColor3 = textColor or UI_TEXT_COLOR
    button.TextScaled = false
    button.TextSize = textSize or 14
    button.Font = font or Enum.Font.SourceSans
    button.Text = text
    button.BorderSizePixel = UI_BORDER_SIZE
    button.BorderColor3 = UI_BORDER_COLOR
    button.Parent = parent
    return button
end

-- Função para inicializar a UI principal
local function initUI()
    UI_FRAME = createUIFrame(PlayerGui, UI_OFFSET, Vector2.new(UI_BUTTON_WIDTH + UI_SPACING * 2, UI_BUTTON_HEIGHT * 2 + UI_SPACING * 3), UI_COLOR, "ESP_UI")
    UI_FRAME.Visible = false -- Começa invisível

    local titleLabel = createTextLabel(UI_FRAME, Vector2.new(UI_SPACING, UI_SPACING), Vector2.new(UI_BUTTON_WIDTH, UI_BUTTON_HEIGHT), "Brainrot ESP", UI_TEXT_COLOR, nil, "Title")
    titleLabel.TextXAlignment = Enum.TextXAlignment.Center
    titleLabel.TextSize = 18

    ESP_TOGGLE_BUTTON = createTextButton(UI_FRAME, Vector2.new(UI_SPACING, UI_BUTTON_HEIGHT + UI_SPACING * 2), Vector2.new(UI_BUTTON_WIDTH, UI_BUTTON_HEIGHT), "ESP: OFF", UI_TEXT_COLOR, UI_COLOR, "ESP_Toggle_Button")
    ESP_TOGGLE_BUTTON.MouseButton1Click:Connect(function()
        ESP_ENABLED = not ESP_ENABLED
        ESP_TOGGLE_BUTTON.Text = "ESP: " .. (ESP_ENABLED and "ON" or "OFF")
        -- Limpa visuais se desativado
        if not ESP_ENABLED then
            for _, visuals in pairs(espVisuals) do
                for _, obj in pairs(visuals) do
                    if obj:IsA("GuiObject") then
                        obj:Destroy()
                    end
                end
            end
            table.clear(espVisuals)
        end
    end)
end

-- Função para criar os visuais do ESP para um jogador
local function createPlayerESPVisuals(player)
    local visuals = {}

    -- Frame principal do ESP (invisível, apenas para agrupar)
    local espHolder = Instance.new("Frame")
    espHolder.Name = player.Name .. "_ESPHolder"
    espHolder.BackgroundTransparency = 1
    espHolder.Size = UDim2.new(0, 0, 0, 0) -- Será redimensionado dinamicamente
    espHolder.Position = UDim2.new(0, 0, 0, 0)
    espHolder.ZIndex = 10 -- Garante que o ESP esteja acima da maioria dos elementos
    espHolder.Parent = PlayerGui

    -- Nome
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "NameESP"
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = ESP_COLOR_PLAYER
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.BackgroundTransparency = 1
    nameLabel.Size = UDim2.new(1, 0, 0.2, 0) -- 20% da altura do espHolder
    nameLabel.Position = UDim2.new(0, 0, -0.2, 0) -- Acima da caixa
    nameLabel.ZIndex = 10
    nameLabel.Parent = espHolder
    visuals.Name = nameLabel

    -- Caixa
    local boxFrame = Instance.new("Frame")
    boxFrame.Name = "BoxESP"
    boxFrame.BackgroundTransparency = 1
    boxFrame.BorderSizePixel = 1
    boxFrame.BorderColor3 = ESP_COLOR_PLAYER
    boxFrame.Size = UDim2.new(1, 0, 1, 0) -- Ocupa todo o espHolder
    boxFrame.ZIndex = 10
    boxFrame.Parent = espHolder
    visuals.Box = boxFrame

    -- Health Bar (fundo)
    local healthBarBg = Instance.new("Frame")
    healthBarBg.Name = "HealthBarBackground"
    healthBarBg.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    healthBarBg.BorderSizePixel = 1
    healthBarBg.BorderColor3 = Color3.fromRGB(0, 0, 0)
    healthBarBg.Size = UDim2.new(0.1, 0, 1, 0) -- 10% da largura, altura total da caixa
    healthBarBg.Position = UDim2.new(-0.15, 0, 0, 0) -- À esquerda da caixa
    healthBarBg.ZIndex = 10
    healthBarBg.Parent = espHolder
    visuals.HealthBarBg = healthBarBg

    -- Health Bar (preenchimento)
    local healthBarFill = Instance.new("Frame")
    healthBarFill.Name = "HealthBarFill"
    healthBarFill.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    healthBarFill.BorderSizePixel = 0
    healthBarFill.Size = UDim2.new(1, 0, 1, 0) -- Ocupa o healthBarBg
    healthBarFill.Position = UDim2.new(0, 0, 0, 0)
    healthBarFill.ZIndex = 11
    healthBarFill.Parent = healthBarBg
    visuals.HealthBarFill = healthBarFill

    -- Distância
    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Name = "DistanceESP"
    distanceLabel.Text = "0m"
    distanceLabel.TextColor3 = ESP_COLOR_PLAYER
    distanceLabel.TextScaled = true
    distanceLabel.Font = Enum.Font.SourceSansBold
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.Size = UDim2.new(1, 0, 0.2, 0) -- 20% da altura do espHolder
    distanceLabel.Position = UDim2.new(0, 0, 1, 0) -- Abaixo da caixa
    distanceLabel.ZIndex = 10
    distanceLabel.Parent = espHolder
    visuals.Distance = distanceLabel
    
    visuals.Holder = espHolder -- Adiciona o holder principal para fácil acesso e destruição

    return visuals
end

-- Função para atualizar os visuais do ESP para um jogador
local function updatePlayerESPVisuals(player, visuals)
    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")

    if not character or not humanoid or not rootPart or not visuals.Holder then
        -- Se o personagem não existe ou faltam partes, destrói os visuais
        if visuals.Holder then visuals.Holder:Destroy() end
        return
    end

    local screenPos, onScreen = Camera:WorldToScreenPoint(rootPart.Position)
    local distance = (Camera.CFrame.Position - rootPart.Position).Magnitude

    if onScreen and distance <= ESP_MAX_DISTANCE and ESP_ENABLED then
        visuals.Holder.Visible = true

        -- Calcula a caixa 2D ao redor do personagem
        local headPos = character.Head.Position
        local hrpPos = rootPart.Position

        -- Tentativa simples de caixa (pode ser mais complexa com WorldToScreenPoint para todos os cantos)
        -- Usamos a cabeça e o root part como pontos de referência.
        -- O Y da caixa é a diferença entre a cabeça e os pés, o X é um valor fixo ou calculado.
        local headScreenPos, headOnScreen = Camera:WorldToScreenPoint(headPos)
        local hrpScreenPos, hrpOnScreen = Camera:WorldToScreenPoint(hrpPos)

        if headOnScreen and hrpOnScreen then
            local boxHeight = math.abs(headScreenPos.Y - hrpScreenPos.Y) * 1.5 -- Ajuste para cobrir mais do corpo
            local boxWidth = boxHeight * 0.5 -- Proporção típica de um personagem

            local boxCenterX = hrpScreenPos.X
            local boxCenterY = hrpScreenPos.Y - boxHeight / 2 -- Posiciona o centro da caixa no HRP

            local boxX = boxCenterX - boxWidth / 2
            local boxY = headScreenPos.Y - (boxHeight * 0.1) -- Ajuste para começar um pouco acima da cabeça

            visuals.Holder.Size = UDim2.new(0, boxWidth, 0, boxHeight)
            visuals.Holder.Position = UDim2.new(0, boxX, 0, boxY)

            -- Atualiza Nome
            if ESP_NAME_ENABLED then
                visuals.Name.Visible = true
                visuals.Name.Text = player.Name
            else
                visuals.Name.Visible = false
            end

            -- Atualiza Caixa
            if ESP_BOX_ENABLED then
                visuals.Box.Visible = true
            else
                visuals.Box.Visible = false
            end

            -- Atualiza Health Bar
            if ESP_HEALTH_BAR_ENABLED and humanoid.MaxHealth > 0 then
                visuals.HealthBarBg.Visible = true
                local healthRatio = humanoid.Health / humanoid.MaxHealth
                visuals.HealthBarFill.Size = UDim2.new(1, 0, healthRatio, 0)
                visuals.HealthBarFill.Position = UDim2.new(0, 0, 1 - healthRatio, 0)
                visuals.HealthBarFill.BackgroundColor3 = Color3.fromRGB(255 * (1 - healthRatio), 255 * healthRatio, 0) -- Verde para cheio, vermelho para vazio
            else
                visuals.HealthBarBg.Visible = false
            end

            -- Atualiza Distância
            if ESP_DISTANCE_ENABLED then
                visuals.Distance.Visible = true
                visuals.Distance.Text = math.floor(distance) .. "m"
            else
                visuals.Distance.Visible = false
            end
        else
            visuals.Holder.Visible = false
        end
    else
        visuals.Holder.Visible = false
    end
end

-- Loop principal de renderização do ESP
RunService.RenderStepped:Connect(function()
    if not ESP_ENABLED then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end -- Não renderiza ESP para o jogador local

        if not espVisuals[player.UserId] then
            espVisuals[player.UserId] = createPlayerESPVisuals(player)
        end

        updatePlayerESPVisuals(player, espVisuals[player.UserId])
    end

    -- Limpa ESPs de jogadores que saíram
    for userId, visuals in pairs(espVisuals) do
        local player = Players:GetPlayerByUserId(userId)
        if not player or player == LocalPlayer then
            if visuals.Holder then visuals.Holder:Destroy() end
            espVisuals[userId] = nil
        end
    end
end)

-- Lidar com novos jogadores entrando
Players.PlayerAdded:Connect(function(player)
    if player == LocalPlayer then return end
    -- Se o ESP já está ativo, cria os visuais imediatamente
    if ESP_ENABLED then
        espVisuals[player.UserId] = createPlayerESPVisuals(player)
    end
end)

-- Lidar com jogadores saindo
Players.PlayerRemoving:Connect(function(player)
    if player == LocalPlayer then return end
    if espVisuals[player.UserId] then
        if espVisuals[player.UserId].Holder then
            espVisuals[player.UserId].Holder:Destroy()
        end
        espVisuals[player.UserId] = nil
    end
end)

-- Toggle da UI com "LeftControl"
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if gameProcessedEvent then return end

    if input.KeyCode == Enum.KeyCode.LeftControl then
        UI_OPEN = not UI_OPEN
        if UI_FRAME then
            UI_FRAME.Visible = UI_OPEN
        end
    end
end)

-- Inicializa a UI ao carregar o script
initUI()

-- Mensagem de depuração (pode ser removida em um ambiente real)
print("Brainrot ESP script carregado. Pressione LeftControl para abrir/fechar a UI.")

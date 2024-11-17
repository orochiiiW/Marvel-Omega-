local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local holdingKey = false -- Variável para verificar se a tecla está sendo segurada
local holdStartTime = nil -- Armazena o momento em que a tecla começou a ser segurada

local function onHoldKey(target)
    print("Agora segure a tecla E por 15 segundos para completar a ação.")
    holdStartTime = nil -- Reinicia o tempo inicial
    holdingKey = true -- Marca que o jogador está tentando segurar a tecla

    -- Escutando quando a tecla "E" é pressionada
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.E and holdingKey then
            holdStartTime = tick() -- Armazena o tempo inicial
            print("Tecla E pressionada. Segurando...")
        end
    end)

    -- Escutando quando a tecla "E" é solta
    UserInputService.InputEnded:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.E then
            holdingKey = false
            holdStartTime = nil
            print("Tecla E solta. Recomece o processo.")
        end
    end)

    -- Monitorando o tempo de pressionamento
    while holdingKey do
        if holdStartTime and (tick() - holdStartTime) >= 15 then
            print("Tecla E foi segurada por 15 segundos! Ação concluída.")
            holdingKey = false -- Para evitar repetição
            break
        end
        task.wait(0.1) -- Pequena pausa para evitar uso excessivo de recursos
    end
end

local function teleportToItem(item)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = item.CFrame + Vector3.new(0, 5, 0) -- Teleporta acima do item
        print("Teletransportado para o item: " .. item.Name)
        onHoldKey(item) -- Chama a função de segurar a tecla após o teletransporte
    end
end

local function createESP(target)
    if target:FindFirstChild("ESP") or target:FindFirstChild("Highlight") then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP"
    billboard.Adornee = target
    billboard.Size = UDim2.new(5, 0, 2.5, 0) -- Dimensões ajustadas para o tamanho do texto menor
    billboard.AlwaysOnTop = true

    local textLabel = Instance.new("TextLabel", billboard)
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.new(1, 0, 0) -- Cor do texto (vermelho)
    textLabel.Text = target.Name
    textLabel.TextScaled = false
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 50 -- Tamanho ajustado para 50

    billboard.Parent = target

    local highlight = Instance.new("Highlight")
    highlight.Name = "Highlight"
    highlight.Adornee = target
    highlight.FillTransparency = 1 -- Remove o preenchimento interno
    highlight.OutlineColor = Color3.new(0, 1, 0) -- Cor da borda (verde)
    highlight.OutlineTransparency = 0 -- Contorno totalmente visível
    highlight.Parent = target

    teleportToItem(target)
end

local function highlightItems()
    local itemsFolder = workspace:FindFirstChild("Items") -- Altere 'Items' para o nome da pasta de itens no mapa

    if itemsFolder then
        for _, item in pairs(itemsFolder:GetChildren()) do
            if item:IsA("BasePart") or item:IsA("Model") then
                createESP(item)
            end
        end
    end
end

workspace.ChildAdded:Connect(function(child)
    if child:IsA("BasePart") or child:IsA("Model") then
        createESP(child)
    end
end)

highlightItems()

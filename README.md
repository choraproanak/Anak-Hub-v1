-- LocalScript em StarterGui

local player = game.Players.LocalPlayer

-- Função para criar o Hub
local function createHub()
    if player.PlayerGui:FindFirstChild("AnakHub") then return end

    local gui = Instance.new("ScreenGui")
    gui.Name = "AnakHub"
    gui.Parent = player.PlayerGui

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 270, 0, 260)
    frame.Position = UDim2.new(0.5, -135, 0.5, -130)
    frame.BackgroundColor3 = Color3.new(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = true

    local round = Instance.new("UICorner", frame)
    round.CornerRadius = UDim.new(0, 16)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, 0, 0, 38)
    title.Position = UDim2.new(0, 0, 0, 10)
    title.BackgroundTransparency = 1
    title.Text = "Anak Hub v1"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 32
    title.TextColor3 = Color3.fromRGB(255, 0, 0)

    local function createButton(name, posY, key)
        local btn = Instance.new("TextButton", frame)
        btn.Size = UDim2.new(0.85, 0, 0, 38)
        btn.Position = UDim2.new(0.5, -0.85*270/2, 0, posY)
        btn.BackgroundColor3 = Color3.fromRGB(31, 255, 69)
        btn.TextColor3 = Color3.fromRGB(0, 0, 0)
        btn.Text = name.." OFF ("..key..")"
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 22
        btn.BorderSizePixel = 0
        local btnRound = Instance.new("UICorner", btn)
        btnRound.CornerRadius = UDim.new(0, 8)
        return btn
    end

    local noclipBtn = createButton("Noclip:", 58, "F")
    local espBtn = createButton("ESP:", 106, "G")
    local tpBtn = createButton("Teleporte:", 154, "E")

    local credits = Instance.new("TextLabel", frame)
    credits.Size = UDim2.new(1, 0, 0, 26)
    credits.Position = UDim2.new(0, 0, 1, -28)
    credits.BackgroundTransparency = 1
    credits.Text = "creditos: choraproanak"
    credits.Font = Enum.Font.GothamBold
    credits.TextSize = 20
    credits.TextColor3 = Color3.fromRGB(255, 255, 255)

    -- Funcionalidades
    local noclipOn = false
    local espOn = false
    local tpOn = false

    -- Noclip
    local function setNoclip(state)
        noclipOn = state
        noclipBtn.Text = "Noclip: "..(noclipOn and "ON (F)" or "OFF (F)")
        noclipBtn.BackgroundColor3 = noclipOn and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(31, 255, 69)
    end

    noclipBtn.MouseButton1Click:Connect(function()
        setNoclip(not noclipOn)
    end)

    -- Noclip loop
    game:GetService("RunService").Stepped:Connect(function()
        if noclipOn and player.Character then
            for _, part in pairs(player.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        elseif player.Character then
            for _, part in pairs(player.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end)

    -- ESP (highlight os players)
    local espObjects = {}
    local function setESP(state)
        espOn = state
        espBtn.Text = "ESP: "..(espOn and "ON (G)" or "OFF (G)")
        espBtn.BackgroundColor3 = espOn and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(31, 255, 69)

        -- Remove existentes
        for _, v in pairs(espObjects) do
            v:Destroy()
        end
        espObjects = {}

        if espOn then
            for _, plr in pairs(game.Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local highlight = Instance.new("Highlight", plr.Character)
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    table.insert(espObjects, highlight)
                end
            end
        end
    end

    espBtn.MouseButton1Click:Connect(function()
        setESP(not espOn)
    end)

    game.Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function(char)
            if espOn then
                local highlight = Instance.new("Highlight", char)
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                table.insert(espObjects, highlight)
            end
        end)
    end)

    -- Teleporte (teleporta para onde o mouse está mirando)
    local function setTP(state)
        tpOn = state
        tpBtn.Text = "Teleporte: "..(tpOn and "ON (E)" or "OFF (E)")
        tpBtn.BackgroundColor3 = tpOn and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(31, 255, 69)
    end

    tpBtn.MouseButton1Click:Connect(function()
        setTP(not tpOn)
    end)

    game:GetService("UserInputService").InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.KeyCode == Enum.KeyCode.F then
            setNoclip(not noclipOn)
        elseif input.KeyCode == Enum.KeyCode.G then
            setESP(not espOn)
        elseif input.KeyCode == Enum.KeyCode.E then
            setTP(not tpOn)
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 and tpOn then
            local mouse = player:GetMouse()
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.p + Vector3.new(0,3,0))
            end
        end
    end)
end

-- Cria o hub ao entrar no jogo
createHub()

-- Cria o hub toda vez que o PlayerGui for resetado (após morrer/resetar personagem)
player.CharacterAdded:Connect(function()
    createHub()
end)

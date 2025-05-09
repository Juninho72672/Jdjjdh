if _G.evadeFullScriptLoaded then return end
_G.evadeFullScriptLoaded = true

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Configurações
local SPEED = 24
local DEFAULT_SPEED = 16
local speedEnabled = false

-- GUI: status de velocidade
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "EvadeUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = game.CoreGui

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0, 200, 0, 40)
statusLabel.Position = UDim2.new(0.5, -100, 0.1, 0)
statusLabel.BackgroundTransparency = 0.4
statusLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statusLabel.TextStrokeTransparency = 0
statusLabel.Font = Enum.Font.SourceSansBold
statusLabel.TextSize = 28
statusLabel.Visible = false
statusLabel.Parent = screenGui

local function updateStatus()
    statusLabel.Text = "Speed " .. (speedEnabled and "ON" or "OFF")
    statusLabel.Visible = true
    task.delay(1.5, function()
        statusLabel.Visible = false
    end)
end

-- Tecla "T" para ativar/desativar velocidade
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.T then
        speedEnabled = not speedEnabled
        updateStatus()
    end
end)

-- Loop principal: velocidade e stamina
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = speedEnabled and SPEED or DEFAULT_SPEED
    end

    -- Stamina infinita via getgc
    for _, v in pairs(getgc(true)) do
        if typeof(v) == "table" and rawget(v, "sprint") and rawget(v, "current") and rawget(v, "max") then
            v.current = v.max
        end
    end
end)

-- ESP: destaca derrubados de vermelho
task.spawn(function()
    while task.wait(1) do
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                local status = player.Character:FindFirstChild("Downed")
                local existingHighlight = hrp and hrp:FindFirstChild("DownESP")

                if status and status.Value == true then
                    if not existingHighlight then
                        local hl = Instance.new("Highlight")
                        hl.Name = "DownESP"
                        hl.FillColor = Color3.fromRGB(255, 0, 0)
                        hl.OutlineColor = Color3.fromRGB(0, 0, 0)
                        hl.FillTransparency = 0.3
                        hl.OutlineTransparency = 0
                        hl.Adornee = player.Character
                        hl.Parent = hrp
                    end
                elseif existingHighlight then
                    existingHighlight:Destroy()
                end
            end
        end
    end
end)

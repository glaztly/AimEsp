--[[
    Roblox LocalScript: Aimbot + ESP (com smoothness)
    - Pressione R para ativar/desativar o aimbot (a mira gruda suavemente na cabeça do jogador mais próximo do mouse).
    - Pressione F para ativar/desativar o ESP (destaca todos jogadores em vermelho, exceto você).
    - Coloque este LocalScript em StarterPlayerScripts.
--]]

local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local localPlayer = Players.LocalPlayer

-- ========= AIMBOT ===========
local AIMBOT_ENABLED = false
local SMOOTHNESS = 0.1 -- 0.1 = muito suave, 1 = instantâneo

-- Utility: Find closest player's head to mouse cursor
local function getClosestPlayerToMouse()
    local mouse = localPlayer:GetMouse()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local headPos = Camera:WorldToViewportPoint(head.Position)
            if headPos.Z > 0 then -- Only consider if on screen
                local dist = (Vector2.new(headPos.X, headPos.Y) - Vector2.new(mouse.X, mouse.Y)).magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closestPlayer = player
                end
            end
        end
    end

    return closestPlayer
end

-- Called every frame when aimbot is enabled
local function aimAtClosestHeadSmooth()
    local closest = getClosestPlayerToMouse()
    if closest and closest.Character and closest.Character:FindFirstChild("Head") then
        local head = closest.Character.Head
        local currentCFrame = Camera.CFrame
        local desiredCFrame = CFrame.new(Camera.CFrame.Position, head.Position)
        -- Interpolação suave (Lerp)
        Camera.CFrame = currentCFrame:Lerp(desiredCFrame, SMOOTHNESS)
    end
end

-- Toggle aimbot with R
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.R then
        AIMBOT_ENABLED = not AIMBOT_ENABLED
    end
end)

-- Bind to RenderStepped for smooth aiming
RunService.RenderStepped:Connect(function()
    if AIMBOT_ENABLED then
        aimAtClosestHeadSmooth()
    end
end)

-- ========== ESP/Highlight ============
local ESP_ENABLED = false
local HIGHLIGHT_NAME = "ESP_Highlight"

-- Create or remove highlights on all players except local player
local function setHighlights(enabled)
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character then
            local char = player.Character
            -- Try to find existing highlight
            local highlight = char:FindFirstChild(HIGHLIGHT_NAME)
            if enabled then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = HIGHLIGHT_NAME
                    highlight.FillColor = Color3.new(1, 0, 0)
                    highlight.OutlineColor = Color3.new(1, 0, 0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineTransparency = 0
                    highlight.Parent = char
                end
            else
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

-- Update highlights for players who join or respawn
local function onCharacterAdded(char)
    if ESP_ENABLED and char ~= localPlayer.Character then
        local highlight = char:FindFirstChild(HIGHLIGHT_NAME)
        if not highlight then
            highlight = Instance.new("Highlight")
            highlight.Name = HIGHLIGHT_NAME
            highlight.FillColor = Color3.new(1, 0, 0)
            highlight.OutlineColor = Color3.new(1, 0, 0)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Parent = char
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= localPlayer then
        player.CharacterAdded:Connect(onCharacterAdded)
    end
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer then
        player.CharacterAdded:Connect(onCharacterAdded)
    end
end

-- Toggle ESP with F key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F then
        ESP_ENABLED = not ESP_ENABLED
        setHighlights(ESP_ENABLED)
    end
end)

-- Optional: Clean up highlights if player leaves
Players.PlayerRemoving:Connect(function(player)
    if player.Character then
        local highlight = player.Character:FindFirstChild(HIGHLIGHT_NAME)
        if highlight then
            highlight:Destroy()
        end
    end
end)

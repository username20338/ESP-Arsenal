# ESP-Arsennal
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = game:GetService("Workspace").CurrentCamera
local LocalPlayer = Players.LocalPlayer
local ESPEnabled = true

local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()
local Window = Library:MakeWindow({Name = "ESP Arsenal", HidePremium = false, SaveConfig = true, ConfigFolder = "ESPArsenal"})
local Tab = Window:MakeTab({Name = "ESP", Icon = "rbxassetid://4483345998", PremiumOnly = false})

local ESPBoxes = {}

local function CreateESP(player)
    if player == LocalPlayer then return end
    
    local Box = Drawing.new("Square")
    Box.Thickness = 2
    Box.Filled = false
    Box.Transparency = 1
    Box.Visible = false
    ESPBoxes[player] = Box
    
    local function Update()
        if ESPEnabled and player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            
            if onScreen then
                local sizeY = math.clamp(1000 / screenPos.Z, 30, 100)
                local sizeX = sizeY * 0.6
                
                Box.Size = Vector2.new(sizeX, sizeY)
                Box.Position = Vector2.new(screenPos.X - sizeX / 2, screenPos.Y - sizeY / 2)
                
                if player.Team == LocalPlayer.Team then
                    Box.Color = Color3.fromRGB(0, 100, 0) -- Verde escuro para aliados
                else
                    Box.Color = Color3.fromRGB(255, 0, 0) -- Vermelho para inimigos
                end
                
                Box.Visible = true
            else
                Box.Visible = false
            end
        else
            Box.Visible = false
        end
    end
    
    local connection = RunService.RenderStepped:Connect(Update)
    
    player.CharacterRemoving:Connect(function()
        Box.Visible = false
        connection:Disconnect()
        Box:Remove()
        ESPBoxes[player] = nil
    end)
end

local function ToggleESP(state)
    ESPEnabled = state
    if not ESPEnabled then
        for _, box in pairs(ESPBoxes) do
            box.Visible = false
        end
    end
end

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(function(player)
    if ESPBoxes[player] then
        ESPBoxes[player]:Remove()
        ESPBoxes[player] = nil
    end
end)

Tab:AddToggle({
    Name = "Ativar ESP",
    Default = true,
    Callback = ToggleESP
})

Library:Init()

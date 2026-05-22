local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Marretão Pro | Visual Clean",
   LoadingTitle = "Limpando Visuais...",
   LoadingSubtitle = "by Gemini",
   ConfigurationSaving = { Enabled = true, FolderName = "MarretaHub" }
})

local TabESP = Window:CreateTab("Visual (ESP)", 4483362458)
local TabPlayer = Window:CreateTab("Jogador", 4483362458)
local TabMisc = Window:CreateTab("Outros", 4483362458)

local espPlayersActive = false
local espComputersActive = false
local walkSpeedValue, jumpPowerValue = 16, 50
local lockSpeedActive, lockJumpActive = false, false

-- === LOOP DE FORÇA DE ATRIBUTOS ===
task.spawn(function()
    while true do
        local hum = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            if lockSpeedActive then hum.WalkSpeed = walkSpeedValue end
            if lockJumpActive then hum.JumpPower = jumpPowerValue; hum.UseJumpPower = true end
        end
        task.wait(0.1)
    end
end)

-- === ESP JOGADORES (SÓ CONTORNO) ===
local function createPlayerESP(p)
    if p == game.Players.LocalPlayer then return end
    local function setup(char)
        local head = char:WaitForChild("Head", 10)
        
        -- Highlight configurado para ser transparente por dentro
        local highlight = Instance.new("Highlight", char)
        highlight.FillTransparency = 1 -- TOTALMENTE TRANSPARENTE POR DENTRO
        highlight.OutlineTransparency = 0 -- CONTORNO VISÍVEL
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)

        local billboard = Instance.new("BillboardGui", head)
        billboard.Size, billboard.AlwaysOnTop, billboard.StudsOffset = UDim2.new(0, 200, 0, 70), true, Vector3.new(0, 3.5, 0)
        local label = Instance.new("TextLabel", billboard)
        label.BackgroundTransparency, label.Size, label.TextSize, label.Font = 1, UDim2.new(1, 0, 1, 0), 16, Enum.Font.SourceSansBold
        label.TextStrokeTransparency = 0

        task.spawn(function()
            while char and char.Parent and espPlayersActive do
                local isBeast = char:FindFirstChild("Hammer") or char:FindFirstChild("Gemstone")
                local lpRoot = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if lpRoot and char:FindFirstChild("HumanoidRootPart") then
                    local dist = math.floor((char.HumanoidRootPart.Position - lpRoot.Position).Magnitude)
                    local color = isBeast and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 255, 0)
                    
                    label.Text = string.format("%s [%s]\n(%sm)", p.DisplayName, isBeast and "BESTA" or "SOBREVIVENTE", dist)
                    label.TextColor3 = color
                    highlight.OutlineColor = color -- Só muda a cor da linha
                end
                task.wait(0.1)
            end
            highlight:Destroy(); billboard:Destroy()
        end)
    end
    if p.Character then setup(p.Character) end
    p.CharacterAdded:Connect(setup)
end

-- === ESP COMPUTADORES (SÓ CONTORNO) ===
local function applyComputerESP()
    for _, obj in pairs(game.Workspace:GetDescendants()) do
        if obj.Name == "ComputerTable" and obj:IsA("Model") then
            
            local highlight = Instance.new("Highlight", obj)
            highlight.FillTransparency = 1 -- TOTALMENTE TRANSPARENTE POR DENTRO
            highlight.OutlineTransparency = 0

            local billboard = Instance.new("BillboardGui", obj)
            billboard.Size, billboard.AlwaysOnTop, billboard.StudsOffset = UDim2.new(0, 150, 0, 50), true, Vector3.new(0, 4, 0)
            local label = Instance.new("TextLabel", billboard)
            label.Size, label.BackgroundTransparency, label.TextSize, label.Font = UDim2.new(1, 0, 1, 0), 1, 15, Enum.Font.SourceSansBold
            label.TextStrokeTransparency = 0

            task.spawn(function()
                while obj and obj.Parent and espComputersActive do
                    local screen = obj:FindFirstChild("Screen", true)
                    local lpRoot = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if lpRoot and (obj.PrimaryPart or obj:FindFirstChild("BoxInstance")) then
                        local pos = obj.PrimaryPart and obj.PrimaryPart.Position or obj:GetModelCFrame().p
                        local dist = math.floor((pos - lpRoot.Position).Magnitude)
                        
                        local color = Color3.fromRGB(0, 150, 255) -- Azul padrão
                        local status = "Computador"

                        if screen then
                            local c = screen.Color
                            if c.G > c.R and c.G > c.B and c.G > 0.4 then
                                color = Color3.fromRGB(0, 255, 0) -- Verde
                                status = "Computador Completo"
                            elseif c.R > c.G and c.R > c.B and c.R > 0.4 then
                                color = Color3.fromRGB(255, 0, 0) -- Vermelho
                                status = "Computador ERROR"
                            end
                        end

                        label.Text = status .. " ("..dist.."m)"
                        label.TextColor3 = color
                        highlight.OutlineColor = color -- Apenas a linha externa brilha
                    end
                    task.wait(0.1)
                end
                highlight:Destroy(); billboard:Destroy()
            end)
        end
    end
end

-- === INTERFACE ===
TabESP:CreateToggle({
   Name = "ESP Jogadores (Contorno)",
   CurrentValue = false,
   Callback = function(v) espPlayersActive = v; if v then for _, p in pairs(game.Players:GetPlayers()) do createPlayerESP(p) end end end
})

TabESP:CreateToggle({
   Name = "ESP Computadores (Contorno)",
   CurrentValue = false,
   Callback = function(v) espComputersActive = v; if v then applyComputerESP() end end
})

TabPlayer:CreateSlider({
   Name = "Velocidade", Range = {10, 100}, Increment = 1, Suffix = "Speed", CurrentValue = 16,
   Callback = function(v) walkSpeedValue = v end
})
TabPlayer:CreateToggle({ Name = "Travar Velocidade", CurrentValue = false, Callback = function(v) lockSpeedActive = v end })

TabPlayer:CreateSlider({
   Name = "Pulo", Range = {50, 200}, Increment = 1, Suffix = "Power", CurrentValue = 50,
   Callback = function(v) jumpPowerValue = v end
})
TabPlayer:CreateToggle({ Name = "Travar Pulo", CurrentValue = false, Callback = function(v) lockJumpActive = v end })

TabMisc:CreateButton({
   Name = "Relogar",
   Callback = function() game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, game.JobId, game.Players.LocalPlayer) end
})
# Fleethefacility

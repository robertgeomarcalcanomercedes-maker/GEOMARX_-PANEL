# GEOMARX_-PANEL
VENTA
-- [[ STREET LIFE - ANTI-KICK EDITION BY 👑GEOMARX_ ]] --

repeat wait() until game:IsLoaded()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

-- CONFIGURACIÓN
_G.SilentEnabled = true
_G.EspEnabled = true
_G.AntiKick = true
_G.FOV_Radius = 135
_G.TargetPart = "Head"

local currentTarget = nil

-- --- PROTECCIÓN ANTI-KICK ---
if _G.AntiKick then
    local mt = getrawmetatable(game)
    local oldIndex = mt.__index
    setreadonly(mt, false)

    mt.__index = newcclosure(function(self, k)
        if k == "Kick" or k == "kick" then
            return function() print("Intento de Kick bloqueado con éxito 😎") end
        end
        return oldIndex(self, k)
    end)
    setreadonly(mt, true)
    
    -- Bloquear reportes automáticos al servidor
    hookfunction(LP.Kick, function() return nil end)
end

-- --- LÍNEA BLANCA ---
local Snapline = Drawing.new("Line")
Snapline.Thickness = 2
Snapline.Color = Color3.fromRGB(255, 255, 255)
Snapline.Transparency = 1
Snapline.Visible = false

-- --- ESP NARANJA ---
local function CreateEsp(player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(255, 120, 0)
    Box.Thickness = 2
    Box.Filled = false

    RunService.RenderStepped:Connect(function()
        if _G.EspEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character.Humanoid.Health > 0 then
            local pos, vis = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if vis then
                local size = 2300 / pos.Z
                Box.Size = Vector2.new(size, size * 1.5)
                Box.Position = Vector2.new(pos.X - Box.Size.X / 2, pos.Y - Box.Size.Y / 2)
                Box.Visible = true
            else Box.Visible = false end
        else
            Box.Visible = false
        end
    end)
end

-- --- INTERFAZ ELITE ---
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LP:WaitForChild("PlayerGui")
ScreenGui.DisplayOrder = 999
ScreenGui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 190, 0, 260)
Main.Position = UDim2.new(0.1, 0, 0.4, 0)
Main.BackgroundColor3 = Color3.fromRGB(160, 0, 0)
Main.BorderSizePixel = 4
Main.BorderColor3 = Color3.fromRGB(255, 130, 0)
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 60)
Title.Text = "CREADOR:\n👑GEOMARX_"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.FredokaOne
Title.TextSize = 22
Title.Parent = Main

local function CreateToggle(text, y, var)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, y)
    btn.BackgroundColor3 = _G[var] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(100, 0, 0)
    btn.Text = text .. ": " .. (_G[var] and "ON" or "OFF")
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.Parent = Main
    
    btn.MouseButton1Click:Connect(function()
        _G[var] = not _G[var]
        btn.Text = text .. ": " .. (_G[var] and "ON" or "OFF")
        btn.BackgroundColor3 = _G[var] and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(100, 0, 0)
    end)
end

CreateToggle("SILENT AIM", 70, "SilentEnabled")
CreateToggle("ESP NARANJA", 120, "EspEnabled")
CreateToggle("ANTI-KICK", 170, "AntiKick")

-- --- LÓGICA SILENT AIM (CON FILTRO DE CÁMARA) ---
local function GetClosest()
    local target, dist = nil, _G.FOV_Radius
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LP and v.Character and v.Character:FindFirstChild(_G.TargetPart) and v.Character.Humanoid.Health > 0 then
            local pos, vis = Camera:WorldToViewportPoint(v.Character[_G.TargetPart].Position)
            if vis then
                local mDist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                if mDist < dist then
                    target = v.Character[_G.TargetPart]
                    dist = mDist
                end
            end
        end
    end
    return target
end

local old; old = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    if _G.SilentEnabled and not checkcaller() and currentTarget then
        -- Solo rayos largos (Balas), ignoramos cámara (<100)
        if method == "Raycast" and args[2].Magnitude > 100 then
            args[2] = (currentTarget.Position - args[1]).Unit * args[2].Magnitude
            return old(self, unpack(args))
        elseif method == "FindPartOnRayWithIgnoreList" and args[1].Direction.Magnitude > 100 then
            args[1] = Ray.new(args[1].Origin, (currentTarget.Position - args[1].Origin).Unit * args[1].Direction.Magnitude)
            return old(self, unpack(args))
        end
    end
    return old(self, ...)
end)

RunService.RenderStepped:Connect(function()
    currentTarget = GetClosest()
    if currentTarget and _G.SilentEnabled then
        local pos, vis = Camera:WorldToViewportPoint(currentTarget.Position)
        if vis then
            Snapline.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            Snapline.To = Vector2.new(pos.X, pos.Y)
            Snapline.Visible = true
        else Snapline.Visible = false end
    else Snapline.Visible = false end
end)

for _, v in pairs(Players:GetPlayers()) do if v ~= LP then CreateEsp(v) end end
Players.PlayerAdded:Connect(function(p) CreateEsp(p) end)

print("GEOMARX HUB: ANTI-KICK CARGADO")

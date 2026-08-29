-- idk 
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local screen = Instance.new("ScreenGui")
screen.Name = "Overlay"
screen.ResetOnSpawn = false
screen.Parent = playerGui

local box = Instance.new("Frame")
box.Size = UDim2.new(0, 165, 0, 32)
box.Position = UDim2.new(0, 15, 0, 15)
box.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
box.BackgroundTransparency = 0.1
box.BorderSizePixel = 0
box.Active = true
box.Parent = screen

local round = Instance.new("UICorner")
round.CornerRadius = UDim.new(0, 6)
round.Parent = box

local text = Instance.new("TextLabel")
text.Size = UDim2.new(1, -10, 1, 0)
text.Position = UDim2.new(0, 5, 0, 0)
text.BackgroundTransparency = 1
text.Font = Enum.Font.GothamMedium
text.TextSize = 13
text.TextColor3 = Color3.fromRGB(235, 235, 235)
text.TextXAlignment = Enum.TextXAlignment.Center
text.Text = "0 fps  •  0 ms"
text.Parent = box

local dragging
local dragInput
local dragStart
local startPosition

local function move(input)
    local delta = input.Position - dragStart

    box.Position = UDim2.new(
        startPosition.X.Scale,
        startPosition.X.Offset + delta.X,
        startPosition.Y.Scale,
        startPosition.Y.Offset + delta.Y
    )
end

box.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = box.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

box.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        move(input)
    end
end)

local frames = 0
local last = tick()
local currentFps = 0

RunService.RenderStepped:Connect(function()
    frames += 1

    local now = tick()
    if now - last >= 0.5 then
        currentFps = math.floor(frames / (now - last))
        frames = 0
        last = now
    end
end)

task.spawn(function()
    while screen.Parent do
        local ping = 0

        pcall(function()
            ping = math.floor(
                Stats.Network.ServerStatsItem["Data Ping"]:GetValue() + 0.5
            )
        end)

        text.Text = string.format("%d fps  •  %d ms", currentFps, ping)

        task.wait(0.25)
    end
end)

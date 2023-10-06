local wsp = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local Players = game:GetService("Players")
local Local = Players.LocalPlayer

local Camera = wsp.CurrentCamera
local Balls = wsp:WaitForChild("Balls")

getgenv().Signal = Signal or {}

function PlayerPoints()
    local tbl = {}
    for i, v in pairs(Players:GetPlayers()) do
        local UserId, HumanoidRootPart = tostring(v.UserId), v.Character and v.Character:FindFirstChild("HumanoidRootPart")
        if HumanoidRootPart and v == Local then
            tbl[UserId] = Camera:WorldToScreenPoint(HumanoidRootPart.Position)
        end
    end
    return tbl
end

function Parry()
    if Local.Character then
        local Remote = game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("ParryAttempt")
        local WorldToScreenPoint = Camera:WorldToScreenPoint(Local.Character.HumanoidRootPart.Position)
        local args = {
            [1] = 0.5,
            [2] = wsp.CurrentCamera.CFrame,
            [3] = PlayerPoints(),
            [4] = {
                [1] = WorldToScreenPoint.X,
                [2] = WorldToScreenPoint.Y
            }
        }
        Remote:FireServer(unpack(args))
    end
end

local Debounce, LastPlayer, LastTime = false

function Anticipate(Time)
    if Debounce then return end
    
    if LastTime then
        local Sum = (Time - LastTime)
        if (Sum >= -25 and Sum <= 25) then
            if Sum >= 25 or Sum <= -25 then
                return true
            end
        end
    end
    
    LastTime = Time
end

function calculateProjectileTime(initialPosition, targetPosition, initialVelocity)
    local distance = (targetPosition - initialPosition).Magnitude
    local time = distance / initialVelocity.Magnitude
    return time
end

function calculateDistance(projectilePosition, objectPosition)
    return math.abs((projectilePosition - objectPosition).Magnitude)
end

function canObjectParry(projectilePosition, objectPosition, projectileVelocity, objectVelocity)
    local timeToIntercept = calculateProjectileTime(projectilePosition, objectPosition, projectileVelocity)
    local distanceToIntercept = calculateDistance(projectilePosition + projectileVelocity * timeToIntercept, objectPosition + objectVelocity * timeToIntercept)
    local Anticipate = Anticipate(timeToIntercept)

    local isSpamParry = (timeToIntercept <= 0.3)

    if isSpamParry then
        repeat
            Parry()
            wait(0.05)
        until not isSpamParry  -- รอจนกว่า isSpamParry จะเป็นเท็จ
    else
        -- กรณีอื่น ๆ ที่คุณต้องการทำงาน
    end
end

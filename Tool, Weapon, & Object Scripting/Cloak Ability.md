
## Description (See Bottom for Code)
This is a cloak toggle that can be toggled on or off with a keybind of choice (C in this example). It is also linked with a GUI I made to help show how easy it is to integrate with other objects. The script uses tweening to ensure optimized transparency changes as well as allowing for easy transparency modifications as the player's status changes. A cooldown is also built in to ensure that spamming is not possible, and that spamming a keybind will not cause any script activity to occur while on cooldown.

## Video Demonstration


### Cloak In Action

https://github.com/user-attachments/assets/f040268b-950d-4e82-a014-43d73a52b123

*Shows off the cloaking mechanic, such as how the player becomes more visible if they move, and become more transparent if they stay still. Shows off integrated GUI.*


## Code Snippets and Explanation
```
local function cloakRun(vParam, tweenGoalParam)
	local cloakTween = tweenService:Create(vParam, tweenInfo, tweenGoalParam)
	if vParam.Name == "Head" then
		local cloakTweenFace = tweenService:Create(vParam.face, tweenInfo, tweenGoalParam)
		cloakTweenFace:Play()
	end
	cloakTween:Play()
end
```
- Function that will handle the tweening. Takes vParams (v from for loop) and tween goal params (as they will change depending on player status)
- This tween method is very optimized, and causes no lag and barely any script activity

```
for _,v in pairs(plr.Character:GetDescendants()) do
		if v:IsA("MeshPart") or v:IsA("Part") then
			if v.Name ~= "HumanoidRootPart" then
				cloakRun(v, {Transparency = .85})
					if plr.Character.Humanoid.MoveDirection ~= Vector3.new(0,0,0) then
						cloakRun(v, {Transparency = .35})
					end
					
				plr.Character.Humanoid.Running:Connect(function(speed)
						if Cloakstatus == "Active" then
						if speed >0 then
						cloakRun(v, {Transparency = .35})
					else

						cloakRun(v, {Transparency = .85})
					end
end
				end)
			end end
	end
```
- Runs through all descendants in a player and gets all parts/meshparts
- MoveDirection is there to see if player toggles cloak while moving and apply movement transparency, as Humanoid.Running only runs once at the start of the player moving, or when they stop moving.
- As you can see the cloakRun function integrates perfectly, allowing for a more optimized and easier to understand code.

```
(CLIENT)
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.C then
		script.Parent.cloakRemote:FireServer("Active")
		
		
	end
end)

script.Parent.CloakStr.Changed:Connect(function(text)
	editStat.Text = text
end)

script.Parent.CoolStr.Changed:Connect(function(text)
	CDText.Text = text
end)
```
- The GUI text editing is handled on the client. The .Changed is triggered by the string values that get updated by the server script. I chose the server script to update these values as it allows for more accurate timing with the ability itself.


# All Code

```
(Client)
local UIS = game:GetService("UserInputService")
local plr = game.Players.LocalPlayer
local char = plr.Character
local UI = plr.PlayerGui:WaitForChild("CloakUI").Frame
local CDText = UI.CD
local editStat = UI.EditStatus

UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.C then
		script.Parent.cloakRemote:FireServer("Active")
		
		
	end
end)

script.Parent.CloakStr.Changed:Connect(function(text)
	editStat.Text = text
end)

script.Parent.CoolStr.Changed:Connect(function(text)
	CDText.Text = text
end)
```

```
(Server)
local tweenService = game:GetService("TweenService")
local Cloakstatus = "Ready"
local ToggleCooldown = 5
local ToggleOff = false
local cloakStr = script.Parent.CloakStr
local tweenInfo = TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out, 0, false)

local function cloakRun(vParam, tweenGoalParam)
	local cloakTween = tweenService:Create(vParam, tweenInfo, tweenGoalParam)
	if vParam.Name == "Head" then
		local cloakTweenFace = tweenService:Create(vParam.face, tweenInfo, tweenGoalParam)
		cloakTweenFace:Play()
	end
	cloakTween:Play()
end


script.Parent.cloakRemote.OnServerEvent:Connect(function(plr, status)

	if Cloakstatus == "Ready" then
		Cloakstatus = status
		ToggleOff = false
		cloakStr.Value = "ACTIVATED"
	for _,v in pairs(plr.Character:GetDescendants()) do
		if v:IsA("MeshPart") or v:IsA("Part") then
			if v.Name ~= "HumanoidRootPart" then
				v.CastShadow = false
				cloakRun(v, {Transparency = .85})
					if plr.Character.Humanoid.MoveDirection ~= Vector3.new(0,0,0) then
						cloakRun(v, {Transparency = .35})
					end
					
				plr.Character.Humanoid.Running:Connect(function(speed)
						if Cloakstatus == "Active" then
						if speed >0 then
						cloakRun(v, {Transparency = .35})
					else

						cloakRun(v, {Transparency = .85})
					end
end
				end)
			end end
	end
	script.Parent.CoolStr.Value = "Cooldown"
		task.wait(ToggleCooldown)
		ToggleOff = true
		script.Parent.CoolStr.Value = "Ready"
		
		
	elseif  ToggleOff == true then
		ToggleOff = false

		Cloakstatus = nil
		cloakStr.Value = "DEACTIVATED"
		for _,v in pairs(plr.Character:GetDescendants()) do
			if v:IsA("MeshPart") or v:IsA("Part") then
				if v.Name ~= "HumanoidRootPart" then
					cloakRun(v, {Transparency = 0})
				end end end
		script.Parent.CoolStr.Value = "Cooldown"
		task.wait(ToggleCooldown)
		Cloakstatus = "Ready"
		script.Parent.CoolStr.Value = "Ready"
	else
		print("Cooldown Active")
	end
end)
```

## Description (See Bottom for Code)
This is a jackal shield based off the shield from Halo. This was created around May of 2025. The shield works by pressing "H" to activate the shield. Pressing "F" will allow the user to raise/lower the shield in-front of them. The shield does take damage, and will change color based off how much damage is taken. Once depleted, the sheild will deactivate and be put on cooldown. Once the cooldown is over, the shield will recharge to full health.

## Video Demonstration


### Shield in Action

https://github.com/user-attachments/assets/eb455841-3c52-49c1-a290-bf5a2e35cc8f



*The video above demonstrates the arm & shield movements that are possible. The video also shows the shield taking damage and regenerating after being depleted.*


## Code Snippets and Explanation
```
hp.Changed:Connect(function()
	 healthRatio = (math.abs(hp.Value-100)) 
	if ogval > hp.Value then
		dmgTaken = true
	end

	shield.Color = Color3.fromRGB(5+(healthRatio*2.5),160-(healthRatio*1.6),200-(healthRatio*1.7))
	
	if hp.Value <= 0 then
		if not shieldRecharging then
			shieldRecharging = true
			model.Handle.ElectricEmitter.Enabled = true
		hp.Value = 0
		shield.Transparency = 1
		shield.CanTouch = false
		shield.CanQuery = false
		end
	end
```

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
UIS = game:GetService("UserInputService")

local char = game.Players.LocalPlayer.Character
local inputDeb = false
local OnOffDeb =false
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.F then
		if not inputDeb then
			print("F")
			inputDeb = true
		script.Parent.shieldRemote:FireServer()
			task.wait(.4)
			inputDeb = false
	end
	end
end)



UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.H then
		if not OnOffDeb then
			OnOffDeb = true
			print("H")
			script.Parent.ToggleOnRemote:FireServer()
			task.wait(1)
			OnOffDeb = false
		end
		end
	end)
```

```
(Server)
local Folder = script.Parent
local model = Folder.ShieldModel
local shield = model.shield
local hp = Folder.HP
local char = model.Parent.Parent
local animation1 = Instance.new("Animation")
animation1.AnimationId = "https://www.roblox.com/asset/?id=102956666099677"
local playEquipAnim = char.Humanoid:LoadAnimation(animation1)
playEquipAnim.Priority = Enum.AnimationPriority.Action4
local shieldUp = false
local shieldActive = false
local shieldRecharging = false
local tweenService = game:GetService("TweenService")
shield.CanCollide = false
local healthRatio = nil
local dmgTaken = false
local ogval  = 0
--tween transparency and brightness

--When shield recharging or turned on. Shield itself will be white, with strong colored highlight/outline, then return to normal
--electric charge to handle when shield pops

--Edit PivotOffset Position of ShieldModel to get arm placement


local function ShieldToggle(On)
	local tweenGoal = {}
	shield.CanTouch = true
	shield.CanQuery = true
	if On == true then
	tweenGoal.Transparency = .5
	tweenGoal.Size = Vector3.new(4, 0.4, 4)
	else
		tweenGoal.Transparency = 1
		tweenGoal.Size = Vector3.new(0.001, 0.001, 0.001)
	end
	local tweenInfo = TweenInfo.new(.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, 0, false)
	local tween = tweenService:Create(shield, tweenInfo, tweenGoal)
	tween:Play()
	shieldRecharging = false
end

local rechargeDeb = false


model:MoveTo(char.LeftLowerArm.Position)

Folder.armWeld.Part1 = char.LeftLowerArm
	

hp.Changed:Connect(function()
	 healthRatio = (math.abs(hp.Value-100)) 
	if ogval > hp.Value then
		dmgTaken = true
	end

	shield.Color = Color3.fromRGB(5+(healthRatio*2.5),160-(healthRatio*1.6),200-(healthRatio*1.7))
	
	if hp.Value <= 0 then
		if not shieldRecharging then
			shieldRecharging = true
			model.Handle.ElectricEmitter.Enabled = true
		hp.Value = 0
		shield.Transparency = 1
		shield.CanTouch = false
		shield.CanQuery = false
		end
	end
	
	ogval = hp.Value
	--if rechargeDeb == false then
	
--	end

end)

local function Recharge()
	local Count = 0
	
	while task.wait(.5) do
	if dmgTaken == false and hp.Value < 100 then
	Count +=1
	if Count == 10 then
		print("regen")
		if shieldRecharging == true then
				ShieldToggle(true)
				shield.CanTouch = true
				shield.CanQuery = true
				model.Handle.ElectricEmitter.Enabled = false
				end
				repeat
					task.wait(.05)
					hp.Value +=1
					healthRatio +=(math.abs(hp.Value-100)) 
					shield.Color = Color3.fromRGB(5+(healthRatio*2.5),160+(healthRatio*1.6),200-(healthRatio*1.7))
					if hp.Value > 100 then
						hp.Value = 100
					end
				until dmgTaken == true or hp.Value == 100
				Count = 0
				shieldRecharging = false
	end
	else
	end
		dmgTaken = false	
	end
end

task.spawn(Recharge)

script.Parent.shieldRemote.OnServerEvent:Connect(function(plr)

	
	if shieldUp == false then
		shieldUp = true
	
		playEquipAnim:Play()

	task.wait(playEquipAnim.Length -.04)
	playEquipAnim:AdjustSpeed(0)

	else
	
	playEquipAnim:AdjustSpeed(-1)
		shieldUp = false
	end
	

end)

script.Parent.ToggleOnRemote.OnServerEvent:Connect(function(plr)
	if shieldActive == false and shieldRecharging == false then
		shieldActive = true
		ShieldToggle(true)
	elseif shieldActive == true and  shieldRecharging == false then
		shieldActive = false 
		ShieldToggle(false)
	end
end)


```

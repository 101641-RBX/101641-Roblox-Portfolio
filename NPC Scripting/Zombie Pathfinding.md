# Scripting Overview

## Description
This is a NPC Pathfinding script that tracks a players location and has the NPC pathfind to them. The magic comes with the use of coroutines. I use a coroutine.wrap() and close system to make it so only 1 coroutine is running at a time without it causing delays or skips in the pathfinding. This allows for a highly optimized pathfinding system with the benefit of a smooth pathfinding.

## Video Demonstration

### Zombie Pathfinding (Single NPC)

https://github.com/user-attachments/assets/47df111b-a2f5-41de-9e3e-43f03decc0e1


*Video shows how the NPC has full pathfinding capabilities and can adapt to blocks on the move*


### NPC Horde


https://github.com/user-attachments/assets/b2b534d3-434f-4bb7-9f8b-78112dc22d8d


*Video shows the script performance when 250 NPCS are active. Also wanted to show how this is including the NPC having a targetting system with line of sight raycasting, as well as random node generation for slight offsetting, as well as physics to make the NPC face the target.*

## Code Snippets and Explanation
```
coroutine.wrap(function() 
			local lastAttack =  workspace:GetServerTimeNow()
			while task.wait(.08) do --works fine when stopped
				if HRP and Tgt then
					MovementState()
					if workspace:GetServerTimeNow() - lastAttack >= .7 then 
					task.spawn(Attack)
						lastAttack = workspace:GetServerTimeNow()
					end
					local state = Humanoid:GetState()
					if state ~= Enum.HumanoidStateType.Jumping and state ~= Enum.HumanoidStateType.Freefall 
						and Humanoid.FloorMaterial ~= Enum.Material.Air then
						update()
					else
						Humanoid:MoveTo(Tgt.Position)
					end
				end
				end
		end)()
```
- This is the main while loop, and the only while loop in the entire pathfinding script. This is also the only wrap coroutine which gets closed & created on a constant interval.
- The reason I have the Humanoid:GetState() checks is to ensure the NPC doesn't repath while jumping, which can cause the NPC to freeze mid jump and backtrack.
- The Humanoid:MoveTo(Tgt.Position) is there to ensure momentum is applied when the bot is jumping, that way the bot isn't jumping in place. This MoveTo is not using computeasync().



```
while stop == false do
				local nodeCopy = node:Clone()
				nodeCopy.BrickColor = BrickColor.new("Black")
				nodeCopy.Rotation = Part.Rotation
				nodeCopy.Parent = workspace.Nodes
				nodeCopy.CanCollide = true
				nodeCopy.CanTouch = true
				nodeCopy.Position = sideType

				nodeCopy.CFrame =  nodeCopy1.CFrame * CFrame.new(0,0,CframeDown)
				CframeDown +=8

				if (nodeCopy.Position - corners.BackLeftCorner).Magnitude <=10 then
					LeftReached = true
				end
				local partsInRegion = workspace:GetPartBoundsInBox(CFrame.new(nodeCopy.Position), nodeCopy.Size, overlapParams)
				for _, cut in pairs(partsInRegion) do
					if cut.Name == "CUTOFF" then
						stop = true
						nodeCopy:Destroy()
						break
					end
				end
				task.wait()
			end
```
- This is a portion of one function that handles creating and placing the node
- "Stop" is kept false until the node generation reaches the back of the part (cutoff)
- The CframeDown is the increment that the part moves across (left to right), can be modified if you want shorter
- "LeftReached = true" is ran when the node hits the backright corner, which is the last spot the node is suppose to touch, which will stop the script from running.
- The partsInRegion is ran after the node generates to see if it is inside the cutoff part. If it is, then the script stops and node generation of the part is complete.

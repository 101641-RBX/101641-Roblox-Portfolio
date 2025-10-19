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
local resetYAxis = Vector3.new(1,0,1)
	local Topside = Vector3.new(0,Part.Position.Y + (Part.Size.Y / 2))

	
	local sides = {
		front = Part.CFrame.LookVector * ((Part.Size.Z / 2)+sidesOffset) +Part.Position * resetYAxis + Topside,
		back = -Part.CFrame.LookVector * ((Part.Size.Z / 2)+sidesOffset) +Part.Position* resetYAxis + Topside,
		right = Part.CFrame.RightVector * ((Part.Size.X/2)+sidesOffset) + Part.Position* resetYAxis + Topside,
		left = -Part.CFrame.RightVector * ((Part.Size.X/2)+4) + Part.Position * resetYAxis + Topside,

	}

	local corners = {
		FrontLeftCorner = (sides.left + sides.front - Part.Position)  ,
		FrontRightCorner = (sides.right + sides.front - Part.Position)   ,
		BackLeftCorner = (sides.left + sides.back - Part.Position) ,
		BackRightCorner = (sides.right + sides.back - Part.Position)  ,
	}
```
- Gets the corners and sides of the part, which will help indicate when the node should stop generating
- The topside ensures that the nodes are placed on top of the part. The (*resetYAxis + Topside) ensures that the nodes are placed on the surface and not on a fixed y-axis, allowing for the nodes to adjust to rotation


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

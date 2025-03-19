# Scripting Overview

## Description
This script generates parts called "Nodes" that will be placed at fixed intervals on top of the main part. To prevent endless generation of the nodes, cutoffs are generated at the corners and front/back of the main part, which when reached will force the script to start generating further down the main part. **This works for all part sizes, as well as all rotation.**

## Why Use Nodes?
- **More Control:** Allows for you to manually set where the NPC should move to, allowing you to forcefully randomize pathfinding which can make NPCs less predictable.
- **More Gameplay Opportunities**: Since nodes are physical parts, you can access more properties and use them to help influence NPC behavior. Some examples are:
  - Raycasting from a node to enemey, to see if the node is behind cover or not
  - Get all possible nodes in a region, and have NPCs pick a random node based off the region (helps with optimization)
  - Can delete nodes that are inside other parts, guaranteeing that the NPC will never move to that node


## Video Demonstration

### Node Generating



https://github.com/user-attachments/assets/d9c52dfc-0d11-4c7e-8236-50dde8dd1e13



*Video Example of the node spawning and how it can be used for any shape and rotation. Note that having multiple of these run at once will increase script activity, but the script will terminate itself once completed. This is ok as this script is suppose to run upon server startup where everything is loading in*


### NPC Pathfinding with Nodes
https://github.com/user-attachments/assets/0ab63a68-fbfb-41cb-b00b-15574cf6ebb1

*Vide Example showing how basic NPC pathfinding can work with nodes*

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



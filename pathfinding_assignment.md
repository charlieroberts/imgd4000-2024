# IMGD 4000 A* assignment

1. Implement the [A* algorithm](https://www.redblobgames.com/pathfinding/a-star/introduction.html), 
the most commonly found pathfinding algorithm in gaming and the basis. The tutorial above will walk
you through the algorithm in detail, but feel free to ask questions in the Discord server. We will
cover some setup code for this assignment in class; you might want to wait until then to begin this.

## Requirements
1. Implement A* in C++, by subclassing the AIController class. 
Your level map should feature a variety of different types of terrain (these can just be different colored blocks)
with different weights attached to them, walls, and a goal for your AI to travel to. If you want to create the level in
Unreal, you could potentially use tags on each block to store the weight information; this would allow you to author the level
entirely in the editor.
2. Create a video showing your AI traversing the level.
3. Post your repo to GitHub. Include a README that links to your gameplay video and provides 
a brief description of the project.
4. DM me the repo link in Discord by the last day of class. 

## Grading
80% : does the pathfinding algorithm work?   
20% : correctly followed instructions (posted video, made GitHub repo etc.)

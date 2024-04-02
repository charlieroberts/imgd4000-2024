# Unreal Soundscape Tutorial

[Another tutorial in Unreal](https://docs.unrealengine.com/5.1/en-US/soundscape-in-unreal-engine/)

1. Create a new first or third person project
2. Turn the Soundscape plugin on (in Edit > Plugins). You’ll have to restart the editor.
3. Go ahead and find a few different audiofiles / metasounds to use. For each audiofile / metasound, make sure you specify an attenuation class, otherwise your sound won’t be spatialized in the soundscape
4. For each sound, create a new Soundscape Color (Audio > Soundscape > Soundscape Color). Each one of these will represent one of the sounds in your finial soundscape palette. Make sure to choose the randomize pitch option, and also to check Spawn Behavior > Continuously Respawn so that the sound will be repeatedly triggered. Experiment with other settings as you see fit.
5. Make your Sound Palette (Audio > Soundscape > Sound Palette). For now, leave the Playback Conditions blank, but go ahead and add in all your various colors.
6. Soundscapes are triggered by setting Gameplay states, aka Tags. Open your project settings and select Project > Gameplay Tags, and then open the Gameplay Tag Manager. Click the + button to add a new tag, and call it ACTIVE. Set the Source to be DefaultGamePlayTags.ini, and press the Add New Tag button.
7. While you’re in your project settings, let’s add the Soundscape Palette to the project. Do a search for Soundscape, and then add your palette to the Soundscape Palette Collection. This step is important for getting soundscapes to work, so don’t forget it! Close your project settings when you’re done.
8. Now we’ll setup your sound palette to be triggered by the ACTIVE tag. Open your palette again and click the Soundscape Palette Playback Condition.  Edit it, and set the Root Expression to “Any Tags Match”. Under Tags, choose ACTIVE.
9. OK, almost done! Open your level Blueprint. Add an Event BeginPlay node, and then a Get Soundscape Subsystem node.
10. Drag a pin from the Get Soundscape Subsystem node, choose the Set State node. Choose your ACTIVE tag from the Soundscape State.
11. Connect the Event BeginPlay output to your Set State trigger. Save and compile your [level blueprint](./level.png) 
12. Test your level! You should here your soundscape in action.

## Unreal Editor Tutorial #1
Goal: Make a (arguably) prettily lit scene from **scratch**, with some physics, collisions, and C++ hit detection. 
Along the way we’ll look at some coding in Blueprints, some C++ coding, and the general use of the Unreal Editor.

### Getting started
This tutorial assumes that you have both [Unreal Engine 5.2](https://www.unrealengine.com/download) and Visual Studio or VS Code installed. 
CLion is another option and [there are free educational licenses available](https://www.jetbrains.com/community/education/#students) as is Xcode if you're on a Mac.
If you **don’t** have both Unreal and an appropriate editor installed, no worries... just follow along with the screencast and then try things out
later (you can actually get through most of today with just the Unreal Editor). 
Or if you just learn better by watching, feel free to just watch, listen, and ask questions.

We’re going to go ahead and start a new blank game project, and then populate a blank level with lighting, plugins, and meshes to create a simple scene. 
We’ll then add physics / collisions to the the scene, make a new glowing material using Blueprints, 
and then add some interaction by writing some (relatively) simple C++.

#### Creating our project / level
Go ahead and launch Unreal Editor (henceforth known as UE5). You should see a window with various templates to select from; choose `Game`. From the resulting menu, choose `Blank` when it asks what type of game to setup for you. Change the programming method from `Blueprints` to `C++`, set the Quality Preset to `Scalabale`
(this will potentially make things a bit easier on your GPU) and check the `Starter Content` box... this will import some presets to play with
for materials etc. Name the project and hit the `Create` button.

The result Is not very "blank" at all. There’s a bunch of lighting and even a landscape setup in the level that Unreal creates by default. 
That’s OK; we can easily replace this with a truly blank level. But first let’s go ahead and fly around in the default level Unreal has given us. Hit the `Play` (green triangle0 button to do this; you can then hit the `Escape` key to leave the level simulator and return to the editor. The WASD keys (plus others) and the trackpad will let you navigate while the level simulator is engaged; use the Unreal documentation to get [more information about viewport control](https://docs.unrealengine.com/en-US/Engine/UI/LevelEditor/Viewports/ViewportControls/index.html).  

After you’ve flown around the default level a bit, go ahead and create a new level. Select `File > New Level` and choose `Blank Level`. 
Tip: If you go to `Preferences > General > Loading & Saving > Startup > Load Level` at Startup you can select what level is generated 
when you first create a new project.

### Populating our new level
Levels consist of meshes, lighting, blueprints, and other assets working in tandem to create a scene. These are called "actors" in Unreal. Choose `Window > Place Actor` to 
view a pane for placing these on the stage. 
Let’s start by adding a simple Plane Mesh to our level to serve as the "ground". Choose `Place Actor > Shapes > Plane` and drag it out into the level. 
In the Details view that appears on the right (beneath the World Outliner) go ahead and set `Transform > Location` to be `0,0,0` 
and set `Transform > Scale` to be `5,5,5`. 

OK, we have a mesh that serves as a “ground”, but there are currently no lights in our level. As a result, if you hit the “Play” button, you won’t see anything appear. To correct this, drag a `Place Actor > Lights > Directional Light` out and above our ground mesh. You should see a warning that the lights need to be “rebuilt”; this is basically a shader compilation / code generation step. Go ahead and hit `Build > Build All Lighting`. When the lighting is built hit `Play` and you should now see your mesh illuminated.

Doesn’t look so great at the moment, does it? We can change things a bit by playing around with controls for the light found in the contextual `Details` view, such as the light intensity (try lowering this significantly, to around 3.) and the light color. Still a bit boring, right?

### Adding a material
Let’s add a material object to our mesh that will determine its color / texture. If you didn’t add the Starter Content pack when you created your project, go to `Content Browser (bottom left corner of the editor, you might need to open it) > Add New > Add Feature or Content Pack > Content Packs > Starter Content`  and then hit `Add to Project`. This will give us a bunch of materials to play around with.

Select our ground mesh, and then click on the dropdown in `Details > Material` that currently reads `Basic Static Mesh`. Scroll through all the different options you find there and try one or two out. When you’re done exploring, do a search for "Hex" and then choose `M_Tech_Hex_Tile_Pulse` (or really whatever other material looks interesting). Once the material is applied things should look a little more fancy.

### Editing a material with Blueprints
OK, let’s add a new mesh to the scene, and then edit its material to change its color.

1. Add a `Place Actor > Shapes > Sphere` mesh to your scene.
3. Set the sphere’s Transform > Location to be 0,0,75 (the z-axis is correlated with "height" in Unreal, sigh...)

You’ll note that the sphere doesn’t currently have much color, due to its default material and the current lights in the scene (it might be reflecting some color from the plane material). Let’s create and edit a new material for the sphere.

1. Let's view our Content Browser. Choose `Window > Content Browser 1` if it's not already open. 
2. Click the `+ Add` button and select `Material`. Give the material a name and double click it in the content browser.
3. This will launch Blueprints, the visual programming environment for Unreal. You will currently see one node, which contains the default parameters for a material. We will feed a couple of new values to this node.
4. Right-click on the blueprint. This will bring up a node creation menu where you can select from all the various blueprints nodes. Type "VectorParameter" to enter it into the search field and then hit enter when the `VectorParameter` node is selected. Name the new node "Color".
5. Drag the topmost output of the new VectorParameter node to the `Base Color` input of our material. Click on the color editor to choose a new color.
6. Right-click on the blueprint and create a new ScalarParameter node, and name it "Metallic". Connect its output to `Metallic` input of our material.  Set the default for the parameter to be `1`. 
7. Save the material by hitting the disk icon in the upper left corner of the blueprints window or hitting `Ctrl/Cmd+S`. Close the Blueprints window.
8. Select your sphere and apply the new material... it will now appear in the Material dropdown for the Mesh component. You should see changes! If not, try playing with the intensity of your directional light and its z-positioning (make sure it’s above your sphere).

*IMPORTANT* You can focus the camera on any object in your scene by selecting it in the World Outliner view and hitting the F key. This is the bestest shortcut to remember in UE.

OK, that’s a basic introduction to playing around in Blueprints. 

### Adding a plugin
Let’s create a beautiful sunrise for our scene, because all our scenes should have beautiful sunrises. This will also give us a chance to see how plugins work. Before we get install the plugin, we need to save the project and our current level as the editor will have to restart once the plugin has loaded. Press the `Save Current` button in the main toolbar (disk icon in upper left corner), and give your level a name if you haven't already. Then choose `File > Save All` just to make sure all our project details are saved. 

Here’s a link to the tutorial on [Geographically Accurate Sun Positioning | Unreal Engine Documentation](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/SunPositioner/index.html). Complete the first three steps, the last of which will require you to restart the Unreal editor. Instead of continuing, follow the steps below (although feel free to come back to this tutorial and experiment later).

In your Editor Modes panel, drag `Lights > Sun and Sky` into your level. This will blow out your scene with white light, but if you select the SunSky in your World Outliner you’ll be able to to change the current time of day under `Details > Time > Solar Time`. Set the value to be 6 (for 6AM). Now you should have a nice sunrise.

### Adding some physics
It doesn’t get much easier than this. Select our sphere (which should currently be positioned above our ground). In `Details > Physics`, check the `Simulate Physics` option and make sure `Enable Gravity` is checked as well. Hit `Play`.

You should see your sphere fall to the ground and then stop. We can get more interesting movement by slanting the ground. Select your ground mesh, and then set its `Transform > Rotation > Y` to be `10` degrees. Try hitting play again… we now have a rolling ball.

Let’s add an obstacle. Drag out a `Basic > Cube` mesh from our placement menu. Position and scale it so that it becomes an obstacle for the rolling movement of our sphere; keep testing until the sphere actually strikes the cube.

### Writing some code
Let’s quickly add a hit test for when our cube is struck that will log a message to our output window.
1. Select the cube in the editor
2. In Details, select `+ Add Component`
3. Choose the `Actor Component`, and name the new component "ActorCollideComponent"
4. Right-click the new component and select `C++ > Open ActorCollideComponent.h`
5. We need to paste a signature for our hit test into the header file. Add the following under the first `public` section of the class definition... we just want the `UFUNCTION()` line and the `OnActorHit` function signature directly beneath it:
```c++
public: 
	// Sets default values for this component's properties
  UActorCollideComponent();
// ... paste begins here
	UFUNCTION()
	void OnActorHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComponent, FVector NormalImpulse, const FHitResult& Hit );
```

6. Now open up the corresponding .cpp file. We want it to look like this:
```c++

#include "ActorCollideComponent.h"

// Sets default values for this component's properties
UActorCollideComponent::UActorCollideComponent()
{
	// Set this component to be initialized when the game starts, and to be ticked every frame.  You can turn these features
	// off to improve performance if you don't need them.
	PrimaryComponentTick.bCanEverTick = true;
}

// Called when the game starts
void UActorCollideComponent::BeginPlay()
{
	Super::BeginPlay();
	UE_LOG(LogTemp, Warning, TEXT("begin"));

	AActor * actor = GetOwner();
	UStaticMeshComponent * cube = actor->FindComponentByClass<UStaticMeshComponent>();

  cube->OnComponentHit.AddDynamic(this, &UActorCollideComponent::OnActorHit);
}

// Called every frame
void UActorCollideComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	//UE_LOG(LogTemp, Warning, TEXT("tick"));
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
}

void UActorCollideComponent::OnActorHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComponent, FVector NormalImpulse, const FHitResult& Hit ) {
  UE_LOG(LogTemp, Warning, TEXT("HIT!"));
}
```

7. Back in UE5, make sure both the rolling sphere and your cube are generating hit messages. You can find this in `Details > Collisions > Simulation Generates Hit Events`.

8. If you're using VS Code for your code editing, in UE5 select `Tools > Refresh Visual Studio Code Project` to setup the build information. Then in VS Code, select `Terminal > Run Build Task` and the choose the `DebugGame Build` option from the resulting dropdown menu. This will compile your files and hot load them into the Unreal Editor.
9. Alternatively, there should be a button in the lower right corner of your Unreal interface (next to the save info and the version control info)
    that looks like a grid of squares... you can just hit this to compile your code and view errors in the Output Log if needed.

11. Hit Play
12. Celebrate???

### Your light speed tour of UE5 comes to an end
OK, that was actually quite a lot. We created some lights, added some meshes, loaded a plugin, turned on physics, edited a material in Blueprints, generated a C++ component, and got a basic hit test to work. There’s a chance we didn’t get all the way through this on the first day of class… if so we’ll finish up next time.

If you were watching but not following along, part of your homework is to explicitly go and work through this on your own. If you have questions, attend office hours!

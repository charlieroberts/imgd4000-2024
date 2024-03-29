# Accessing C++ from Blueprints
Today we will add three features to our project from last class. In the first, we'll extend our C++ code from last class with a hit test; if
we strike an obstacle we'll destroy it. After that, we'll augment this by attaching a fire particle effect to it. This will
take advantage of Blueprints, and part of the goal of this class is to spend a bit more time in the various environments Blueprints
gives us while also connecting Blueprints to C++.

## Concepts for hit tests
 Last time we made a simple example of a projectile that flew off into space. However, for many types of of "shots" it’s better to be near-instantaneous... think of a laser, for example.  In these cases we want to simply see if a hit occurs at the moment the shot is **fired**.  Unreal enables us to do this with `traces`.

A trace in Unreal is the equivalent of a `Raycast` object in other game systems. We’ll project a line from one point to another. see if any objects are struck in the process, and return the first one we hit that **blocks** (does not allow objects to pass through it). Although you can [define custom filters for the types of objects you want to perform hit tests with](https://www.unrealengine.com/en-US/blog/collision-filtering) in this case we’ll use one of UE5’s built-in filters, which checks against all objects that are visible. 

## Code
Again, this starts with the [code from the last tutorial](./day2_more_c++.md). We will primarily be replacing our the `SpherePawn::fire` method. But first we need to include a helper that will let us see where we’re shooting, so that we can visually identify if hits should occur. Include the following at the top of `SpherePawn.m`:

```c++
#include "DrawDebugHelpers.h"
```

This gives us a variety of different shapes that we can dynamically draw in our world at different locations and with different orientations. In this case we’ll be using lines. Now go to our `.fire` method.

We need to define a start point and an endpoint for our line. 	Our code already defines a starting location at the beginning of our fire method, let’s also augment its X component:

```c++
FVector start = GetActorLocation();
start.Z += 100.f;
start.Y += 100.f;
start.X += 150.f;
```

Next, we want to define an end position. This will simply be our start + (actorForward * distance). We already looked at getting the actor’s current forward direction in last class:

```c++
float distance = 1000.f;
FVector fv = GetActorForwardVector();
FVector end = ((fv * distance) + start);
```

Now let’s draw a line using a function from our draw debug helpers:

```c++
DrawDebugLine(GetWorld(), start, end, FColor::Red, false, 2.f, 0, 5.f);
```

The first four parameters are probably self-explanatory. The fifth(currently `false`) dictates whether the line is persistent. The sixth is the amount of time the line is on screen, then depth priority, and last but not least line thickness.

If you compile and play now, you should have some shooting lines that appear.

### The hit test

OK, let’s find out if something is hit. We need to create a couple of structs to pass to our hit test.

```c++
FCollisionQueryParams cqp;
FHitResult hr;
```

We’ll use the default query parameters that our `cqp` struct gives us, while our `hr` struct will be populated with the results of our hit test.

The call to our hit test is:

```c++
GetWorld()->LineTraceSingleByChannel(hr, start, end, ECC_Visibility, cqp);
```

...where `ECC_Visibility` defines that trace filter that we want to apply to our collision test; in this case, is the object visible. Again, the results of our hit test will be place in the `FHitResult` struct, `hr`.

Now let’s look and see if a hit was found on a blocking object:

```c++
if( hr.bBlockingHit == true ) {	
    if( hr.GetActor() != this ) {
        UE_LOG(LogTemp, Warning, TEXT("HIT! %s"), *hr.GetActor()->GetName() );
        hr.GetActor()->Destroy();
    }
}
```

If you have any code related to spawning projectiles from the last class left in `.fire` go ahead and move it now. 

OK! If you hit compile and then play, we should be getting the lines drawn, but no other results. Go ahead and drag a Cube actor (or any other shape) out into the world in a place where the rays will be able to strike it. Try it again, and you should see the instance name of the object presented in the message console. The cube should also be destroyed.

Instead of using `UE_LOG` we can also draw our debugging messsages to the screen. Change the above block of code to the following to try this out:

```c++
if( hr.bBlockingHit == true ) {	
    if( hr.GetActor() != this ) {
        GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::White, FString::Printf( TEXT("HIT! %s"), *hr.GetActor()->GetName() ) );

        hr.GetActor()->Destroy();
    }
}
```

This is a bit quicker to look at, but doesn't save the message to a log file. Of course, there's nothing to stop you from using both `UE_LOG` and `AddOnScreenDebugMessage` to get the best of both worlds.

## Adding a particle system

### Creating a “hit” event for Blueprints
We’re going to edit / dynamically place our particle emitter using Blueprints, but before we can do that we need to create 
an event that will tell us when our hit test actually strikes the enemy cube. 

We’ll add this “HitSomething” function to the header for our `SpherePawn` class:

```c++
UFUNCTION(BlueprintNativeEvent)
void HitSomething(class UStaticMeshComponent *m);
void HitSomething_Implementation(class UStaticMeshComponent *m);
```

Important (and bizzaro) things to point out, that all relate to the bindings between these C++ classes, the editor, and the reflection engine:

1. The `BlueprintNativeEvent` specifier will enable us to create a node in Blueprints that outputs an event trigger.
2. We need to create two separate functions. One generates our blueprints event, and one contains an implementation for the event. 
3. You must name the implementation the same as the Blueprints event + `_Implementation` or the compiler will throw an error. This is a very specific way to create bindings between C++ / Blueprints / the editor. This method will be called if the event is not used in the Blueprint. Since we know we are going to use the event in our Blueprint we could basically leave this blank, however, you MUST include the function in the header/c++ files or your class will not compile.

OK, with that out of the way, we can all our `HitSomething` method from our `Fire` method (assuming we do actually hit something). In the `SpherePawn` C++ file, switch the following line in the `Fire` method:

```c++
hr.GetActor()->Destroy();
```

 to the following:
```c++
HitSomething( Cast<UStaticMeshComponent>(hr.GetActor()->GetRootComponent()) );
```

For our implementation, we'll just leave it with a log statement:

```c++
void ASpherePawn::HitSomething_Implementation(class UStaticMeshComponent *meshThatWasHit ) {
UE_LOG(LogTemp, Warning, TEXT("HIT SOMETHING CALLED"));
}
```

Note that calling `HitSomething` (which we defined as a UFUNCTION with the `BlueprintNativeEvent` specifier) is what actually triggers the event in Blueprints. We have to include that call.

Compile your C++ code and then move on to creating / editing a blueprint subclass of it. DO NOT TEST IT AT THIS STAGE OR YOU WILL CRASH THE ENGINE.

### Into the Blueprint
OK, so now we want to make a Blueprint “subclass” out of our `SpherePawn` C++ class, so that we can access the Blueprint event we created. 
Go ahead and delete any previous instance of the C++ `SpherePawn` class that you had placed in the scene. 
Next, right-click on the SpherePawn C++ class in your content browser
and choose “Create Blueprint class based on SpherePawn”.  Name the new blueprint `SpherePawn_BP`.

Open up the resulting Blueprint. Click on the “Event Graph” tab to open the visual programming environment, or just select the corresponding tab if it's already open.
We’re not going to use any of the events they give us, instead we’ll use the one we just added in C++... go ahead and delete the other events.

Right-click in any of the blank space and then type “Hit” to search through the available nodes. 
You should see a “Event Hit Something” appear in the dropdown menu; select it. Great! We now have access to the C++ event we created.
The arrow at the top of this node is the control flow for event processing, while the blue output marked `M` is the mesh that has been hit.

Now let’s spawn our particle emitter. Right-click on the Blueprints canvas again and search for “Spawn Emitter Attached”. 
Connect the topmost control flow output from our “Event Hit Something” node to the topmost control flow input in our “Spawn Emitter Attached” node. 

Next, we need to select the particle system that will be spawned. Click on the “Emitter Template” input for the spawn node, 
and then select the “P_Fire” that we migrated to this project earlier. If you don't see this as an option, you probably need to
import the Starter Content into your project.

Almost there! Connect the “M” output to the “Attach to Component” input of our spawn node. Set “Location Type” on the spawn node to be “Snap to Target, Including Scale”.

Last step: drag an instance of your `SpherePawn_BP` component into your level. Assign it a mesh of your choosing for representation. 
Make sure you to tell the pawn to “possess” via the Auto Possess Player option. Make sure the SpherePawn is selected (not the mesh component) and 
then choose Pawn > Auto Possess Player > Player 0 from the Details tree menu.

OK! Compile your blueprint, hit Save, and then hit Play. You should see the particle system placed when you hit the cube with your line trace. It might help to make the cube a bigger target (by scaling it) so it’s easier to hit.

### The cube has burned and must be removed
Let's now setup our blueprint so that after the fire has burned for a few seconds our target is removed from the scene. We'll also set it up so that our fire continues to burn for a second or two afterwards.

1. In order to create delays in ourr Blueprints we can use the `Delay` node. Drag a pin
from the white control node at the top of `Spawn Emitter Attached` and then search for `Delay`. Use `3.0` or whatever else you want for our delay duration.

2. At the end of these three seconds we want to set out cube visibility to 0. While we could simply destroy the cube actor, our particles have been attached to it and would immediately also disappear if we did this. By changing the visibility of the mesh to 0 other components remain visible. Drag a pin from the `Completed` control flow output of our `Delay` node, and search for `Set Visibility`. Connect the `M` output from our event to the `Target` output of our new `Set Visibility` node. Make sure the `New Visibility` argument is unchecked (false). If you save/compile and test your blueprint now, you should see the cube vanish after three seconds while the fire remains.

3. Next we'll drag a pin from the control flow output of `Set Visibility` and create a second `Delay` node; set the duration for `2.0` seconds. Drag a pin from the `Completed` control flow output and choose the `Destroy Actor` node.

4. Last but not least we need the actor we want to destroy. Drag a pin from the `M` output of our event; remember that this represents our `StaticMeshComponent`. To get the `Actor` that owns the mesh, we can select the `Get Owner` node. Take the return value of `Get Owner` and connect it to the `Target` input of our `Destroy Actor` node.  
Save and compile the blueprint, and then test your scene to see the results. Your final blueprint should look something [like this](https://github.com/charlieroberts/imgd4000-2024/blob/main/fire_blueprint.png), minus all the formatting and comments.

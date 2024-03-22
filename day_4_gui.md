# Adding a GUI
In this tutorial we'll learn the basics of adding a GUI, linking it to our GameModeBase class, and linking the GUI to a C++ pawn class. We'll start with a brand new blank project, blank level for this tutorial.

## Setup
After creating the new C++ project + blank level, go ahead and make a new Pawn C++ subclass, and name it SpherePawn. Add two properties to the C++ header:

```c++
	UPROPERTY(EditAnywhere)
	class UStaticMeshComponent * Mesh;
	
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	float Health;
```

We're going to create a simple progress bar that will track the   `Health` property of our `SpherePawn` instance. Go ahead and edit the `.cpp` file, we want to change the constructor and the tick method.

```c++
ASpherePawn::ASpherePawn()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
  PrimaryActorTick.bCanEverTick = true;
  Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");
  Health = 1.f;
}

void ASpherePawn::Tick(float DeltaTime)
{
  Super::Tick(DeltaTime);
  Health -= .001f;
}
```

With these two methods in place, our `Health` will decrease on every frame of gameplay. Compile your code.

Drag a directional light into your blank level, and then drag and instance of your `SpherePawn` class to the level. Go ahead and a mesh of your choice for the instance. In the details view for the instance, set `SpherePawn -> Pawn -> Auto Possess` to `Player 0`. Your `Tick` method will not be called until the `SpherePawn` has possessed the player, so this is an important step.

## Modifying our build file
Next we want to modify the build file for our project so that we can add the necessary UI libraries to the engine. Go ahead and open the C# build file for your project, located alongside the rest of your project’s source code. This configures many of the default libraries that are included in your build. Add "UMG" (for Unreal Motion Graphics) to the first dependency module configuration, so it reads:

```c++
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG" });
```

Next, uncomment the Slate UI line immediately beneath the one you just edited. This includes everything we need to start doing GUI development in our project.

## Editing our GameMode subclass
The GameMode is responsible for broadly managing the game, including displaying relevant statistics via HUD, managing win/lose conditions and more. This is the class we’ll use to go ahead and display the GUI we create using the Unreal Motion Graphics editor (UMG).

Go ahead and open up your project’s `GameModeBase` subclass header;  the Unreal project template will have made this for you automatically. Add the following public properties to your class:

```c++
public:
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Health")
	TSubclassOf<class UUserWidget> HUDWidgetClass;
	
	UPROPERTY()
	class UUserWidget * CurrentWidget;

	virtual void BeginPlay() override;
```

Next, add override the `BeginPlay` method in the C++ file and include the UserWidget header file. Make sure to change your project name where appropriate, and don’t remove the A prefix in the class name that denotes this an Actor:	

```c++
#include "YOURPROJECTNAMEGameModeBase.h"
#include "Blueprint/UserWidget.h"
```

```c++
void AYOURPROJECTNAMEHEREGameModeBase::BeginPlay() {
    Super::BeginPlay();

    CurrentWidget = CreateWidget<UUserWidget>(GetWorld(), HUDWidgetClass);
    CurrentWidget->AddToViewport();
}
```

## Create a Blueprint to display our Health.
In the editor content browser, 	right-click and choose User Interface > Widget Blueprint. Name it `HealthHUD`.

In the designer view, drag out a `Panel > Canvas Panel`, and then layout a simple Progress Bar widget and place an accompanying text label. With the progress bar you just added selected, choose Details > Progress > Bind. This will create a new binding for your element that will enable you to update your progress bar in a blueprint.

In the resulting blueprint graph view, create a [blueprint like this image](./widget_blueprint.png)

Compile and save this blueprint.

## Link everything together

1. Create a blueprint subclass of our `GameModeBase`. Name it `GameMode_BP`.
2. Double click on it and set the Health > HUDWidgetClass to be our `Health HUD`.
3. Choose Edit > Project Settings > Maps and Modes > Default Modes > Default GameMode and set it to be `GameMode_BP` using the associated dropdown menu.

Try Compiling the C++ code if you haven’t done so in a while, and then run Play. You should see a health bar counting down!

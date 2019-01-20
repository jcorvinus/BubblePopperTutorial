# BubblePopperTutorial
Beginner tutorial for Leap Motion's Interaction Engine.

Bubble popper is a simple digital toy - A spout will spawn a series of bubbles, and the player can pop them with a satisfying effect. In this tutorial, we will construct it.

NOTE: This is a VR tutorial. The following hardware is required:
- VR Headset (Vive, Rift, Windows Mixed Reality, etc)
- Leap Motion controller
- Computer capable of driving the aforementioned hardware.

# Getting Started
Open Unity, select the 'new project' option.
Name the project 'bubble popper'

You will now be faced with an empty unity scene. It's time to import the following unity packages:

- Leap_Motion_Core_Assets_ 4.4.0.unitypackage
- Leap_Motion_Interaction_Engine_1.2.0.unitypackage

You can get them [here](https://developer.leapmotion.com/unity#5436356)

Now we'll want to make a folder to hold all of our project-specific content. We'll call it "_BubblePopper", with the underscore being for getting the folder to sort near the top alphabetically. Having a strict separation between our content and third party assets is a good standard practice - it makes turning your project into a distributable unity asset package easier.

We'll make the following sub folders: Scenes, Scripts, Materials

##Enabling VR support
In the menu bar, click on Edit->Project Settings->Player. In the window that appears, click on 'Player Settings' in the bottom-left corner of the window. In the Inspector, find 'XR Settings' and expand it, if it is not already expanded. Set the order of the VR SDKs such that your favorite one loads first. Mine is as follows:

![XR Settings](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/XRSettings.png)

## Scene Setup
Delete the 'Main Camera' object in the root of the scene.
Now, we'll set up our scene. Find the "/LeapMotion/Core/Prefabs/Leap Rig" prefab object and drag it into the scene heirarchy. This is our camera rig, it has components associated with hand tracking and VR rendering.
 
In the scene heirarchy, right-click. In the context menu, select '3D Object/Cube'. Re-name this Cube to 'Floor', and place it like so:

![Floor Setup](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/FloorSetup.png)

Next, we'll apply a material to the floor so it doesn't look so boring. Find the "Blue Dark Standard.mat" material and drag it directly onto the floor model.

Now we'll need a pedestal for our bubbles to appear out of. Let's create another cube object. Name it 'Pedestal'. The exact placement and scale will differ for you because you will probably not be the same exact physical proportions as myself. Nevertheless, this is the placement I'm using:

![Pedestal Setup](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/PedestalSetup.jpg)

Place it however you see fit for yourself. Now, we'll need to add an emitter model. Create a 3d cylinder object, and name it "Emitter". It will come with a capsule collider. Remove or disable it. Place it in the middle of the pedestal and scale it down. Give it whatever material you like. Mine is arranged like so:

![Emitter Setup](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/EmitterSetup.jpg)

Now, we'll want to save our scene. Save it as "_BubblePopper/Scenes/Main.unity"

## Making our Bubble
Create a 3d sphere object and name it 'bubble'. The placement doesn't matter but set the scale to 0.09971523 on all dimensions. We'll want to make it translucent, so we'll need to make a new material. In the "_BubblePopper" folder, right-click and create a new material. Name it 'Bubble'. Set the opacity type to 'transparent', and then set the main color to the following:

![Bubble Color Settings](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/BubbleColorSettings.png)

The material should look like the following:

![Bubble Material](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/BubbleMaterial.png)

Apply the material to the bubble object using whichever method you desire.

Now we'll need to start specifying the behavior for our bubble. We'll need to add one more prefab called the InteractionManager. Find "/LeapMotion/Modules/InteractionEngine/Prefabs/Interaction Manager" and add it to the root of the scene. It should not be a child object of anything. Interaction Engine is initially set up for controller support, but it's incomplete and we'd have to do more work to get it working properly. For now, let's just skip that. Expand InteractionManager to see its child gameobjects. You'll see several controller objects. Disable all of them by selecting them and un-checking the gameobject enable checkbox. Like so:

![Disable Controllers](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/DisableControllers.png)

The touch point for all interactions in InteractionEngine is the InteractionBehaviour component. Let's add one to our bubble now. For this specific object, un-check the 'Move Object when grasped' checkbox. This is because instead of being grabbable, our bubble needs to pop when it is grabbed. We will also want to disable gravity. You may have noticed that adding InteractionBehavior also added a RigidBody component. This is Unity's basic component for physics. There is a 'Use Gravity' field. Un-check the checkbox.

To actually add the popping behavior, we're going to create a new script that manages this bubble object. In our "_BubblePopper/Scripts/" folder create a new script and call it "Bubble". After unity is done compiling, go back to the Bubble object in the heirarchy. In the inspector, click "Add Component", then type "Bubble" into the search field and click the script we made. It should get added to the GameObject as a component.

Our inspector should now look like this:

![Bubble Script And Interaction Behavior](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/BubbleScriptAndInteractionBehavior.png)

Double click the 'script' field of the bubble component to open up the Bubble.cs script. Change it to read like so:

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

using Leap.Unity.Interaction;

namespace BubblePopper
{
	public class Bubble : MonoBehaviour
	{
		InteractionBehaviour interaction;

		private void Awake()
		{
			interaction = GetComponent<InteractionBehaviour>();
		}

		// Use this for initialization
		void Start()
		{
			interaction.OnGraspBegin += DoBubblePop;
		}

		void DoBubblePop()
		{
			Destroy(gameObject);
		}
	}
}
```

We'll go through what each of these things means now. The 'using Leap.Unity.Interaction;' statement is necessary to use Interaction engine - that's where most of the Interaction Engine code lives.

The InteractionBehavior variable called 'interaction' is a reference to the component we made earlier and placeed on the gameObject. Awake is a message that Unity will automatically call when the object first appears in the scene. We set the value of interaction to the result of GetComponent. What this does is tells Unity to find the component on our object that is of type InteractionBehavior, so that we can access it.

Start() gets called after Awake(). It is considered best practice to do any dependency-free initialization in Awake, but any initialization that requires other objects to be set up in Start(). This can save us lots of needless glue code.

In Start(), all we're doing is hooking into the OnGraspBegin event. InteractionBehavior will fire this when an Interaction object counts as 'grabbed'. From there, we've added the simplest possible bubble handling - simply destroying the game object.

We're almost ready to do our first test. In the menu bar, click on the Window->Leap Motion option. In the window that appears, if there are any warnings, click their respective 'auto-fix' button. Close the window, and we're now ready to enter play mode. If you've done everything correctly, you should get this:

[Bubble Pop.webm](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/BubblePopper.webm)

As you can see, if we grab or pinch our bubble, it disappears! This is kind of boring though. Part of making good XR interactions is adding reactivity to them. The more fine, intentional details you can add, the more your user will connect with your program/app/experience/whatever. So let's add a pop sound to our bubble!

I had a friend grab a sound effect for me online. Get it [here](http://soundbible.com/2067-Blop.html). We'll need to create a new folder in our "_BubblePopper" folder to place sounds in. I'll call mine "_BubblePopper/Audio/". Paste the sound clip into this folder. Unity will notice and start loading. When it's done, hop back into Unity. Select the Bubble object in the scene view or scene graph. To play audio, gameObjects need an "Audio Source" component. Add one using the Add component menu. Once the component is on your object, click and drag your audio clip from the project view, onto the "AudioClip" field of the Audio Source component. Un-check the "Play On Awake" checkbox.

Spatialized Audio is a powerful feature of XR, so let's take a moment to set it up (somewhat) properly. In the menu bar, click Edit->Project Settings->Audio. In the Audio Manager, select a spatializer plugin. I used Oculus Spatializer because it was available. There are many spatializer plugins available, feel free to look around the internet for more.

Now, back on our Bubble object, find the Audio Source component again. Set the 'Spatial Blend' slider all the way to '3D'. Then un-fold the '3D Sound Settings' foldout. The default settings are appropriate for large-scale video games (in terms of volume). Because we're building interactions on the human scale, they are not appropriate. Set the 'Min Distance' to 0.1, and 'Max Distance' to 5. There should be a 'Spatialize' checkbox. Check it.

The audio source is set up, but we do not currently have the means to tell it to play our Audio. Open up Bubble.cs in the code editor. Underneath the interaction variable declaration, add the following variable declaration:

```
	AudioSource audio;
```

And in the Awake() method, add the following underneath our interaction component acquisition:

```
	audio = GetComponent<AudioSource>();
```

Now that we've gotten our audio source component, we can tell it to play when desired. Change the DoBubblePop() method to look like so:

```
		void DoBubblePop()
		{
			audio.Play();
			Destroy(gameObject);
		}
```

Enter play mode, and pop the bubble. You should hear the pop sound!

So the bubble is ready to go. But our original prompt called for a bubble spawner and the ability to pop many bubbles. Doing this will require creating a prefab and spawner. Create a new folder called "_BubblePopper/Prefabs". Navigate to it in the Project view, and then click-and-drag the Bubble object from the scene graph into this folder. If you were successful, you will have a new prefab in the folder like this:

![Bubble Prefab](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/BubblePrefab.png)

You may now delete the object from the scene graph. Then save the scene. Note that Unity sometimes doesn't save prefab changes until a scene save is initiated, so remember to save a scene when you want a prefab to be saved.

## Blowing Bubbles
Now, we'll need to create a new script that handles spawning our bubbles. Create a new script in the "_BubblePopper/Scripts/" folder and name it "BubbleEmitter". Select the 'Emitter' object in the scene graph and add the BubbleEmitter component. Now open the script. Put the monobehavior class into the same namespace we used before (BubblePopper).

Now, to control the emitter's behavior, we'll need some variables. Add the following to the class:

```
		[SerializeField] GameObject bubblePrefab;
		[SerializeField] float timeBetweenBursts;
		[Range(2, 6)]
		[SerializeField] int bubblesPerBurst;
		[SerializeField] bool isSpawning;
```

bubblePrefab is a reference to the bubble prefab object we made earlier - we'll need to hook it up in the inspector. The other variables don't do anything yet, but we'll use them to control buble spawning. Basically, we'll spawn a 'burst' of bubbles every time the interval elapses.

To do this, we'll use a [Coroutine](https://docs.unity3d.com/Manual/Coroutines.html). Add the following variable to the code, just under the ones we made earlier:

```
		Coroutine spawnCoroutine;
```

And define the following method:
```
		IEnumerator BubbleSpawn()
		{
			while(isSpawning)
			{
				// do spawn
				for(int i=0; i < bubblesPerBurst; i++)
				{
					GameObject bubbleObject = GameObject.Instantiate(bubblePrefab);
					bubbleObject.transform.position = transform.position;
					yield return new WaitForEndOfFrame(); // do one per-frame so that they can collide-resolve with each other.
				}

				yield return new WaitForSeconds(timeBetweenBursts);
			}
		}
```

Every time this executes, it will spawn the desired number of bubbles, place them at the emitter's position, then wait for the burst timer to elapse. It won't stop spawning until we tell it to. Now, this code won't start on its own until we invoke it. Add the following method:

```
		void StartSpawning()
		{
			isSpawning = true;
			spawnCoroutine = StartCoroutine(BubbleSpawn());
		}
```

And we'll call it from the Start() method:

```
		// Use this for initialization
		void Start()
		{
			StartSpawning();
		}
```

Now, we'll need to hook up our inspector references and set the variables properly. Go back to Unity, and find the "Emitter" object inside of the scene graph. After unity is done compiling, the inspector will change to show us the following:

![Initial Emitter](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/InitialEmitter.png)
The first field, "BubblePrefab" needs to be filled with the bubble prefab we made earlier. To make this easier, we're going to lock the Inspector panel so that it doesn't change on us while we're navigating the project view. With the emitter object selected, go to the Inspector window and navigate towards the upper right corner. You'll see the lock icon pictured below:

![Inspector Lock](https://github.com/jcorvinus/BubblePopperTutorial/blob/master/Images/inspectorLock.png)

Click on the lock icon and now we can navigate around freely without the inspector losing focus on us. Navigate to the  "_BubblePopper/Prefabs" folder. Click and drag the "Bubble" prefab onto the "BubblePrefab" field of the BubbleEmitter component. For time between bursts, let's give it a value of 1.25. Slide the "Bubbles per burst" value to whatever you feel appropriate within the valid range. Leave the 'IsSpawning' value where it is. Un-lock the inspector while its state is fresh in your mind.

Enter play mode and.... well that's not helpful. The bubbles pile up on each other and de-collide in un-helpful ways. The first issue is definitely that the bubbles are being pushed inside of the pedestal. Let's change the spawn height to clear this. Our bubbles are 0.09971523f, so change the line:

```
bubbleObject.transform.position = transform.position;
```

to:

```
bubbleObject.transform.position = transform.position + (Vector3.up * 0.11f);
```

The first thing we'll want to do is make it so that the bubbles have an upwards velocity to them. Select the bubble prefab and in the inspector, add a 'Constant Force' component. Set the force value to 0, 0.25, 0 

Enter play mode again and the bubbles fly up now. It's still not very controlled, but we can improve things from here. Go back into the BubbleSpawn coroutine and change it to the following:

```
		IEnumerator BubbleSpawn()
		{
			while(isSpawning)
			{
				// do spawn
				for(int i=0; i < bubblesPerBurst; i++)
				{
					GameObject bubbleObject = GameObject.Instantiate(bubblePrefab);
					bubbleObject.transform.position = transform.position + (Vector3.up * 0.11f);

					ConstantForce bubbleForce = bubbleObject.GetComponent<ConstantForce>();
					Vector3 forceModifier = Random.onUnitSphere;
					forceModifier *= 0.15f; // scale it down. A lot.
					forceModifier = new Vector3(forceModifier.x, 0.25f, forceModifier.z);
					bubbleForce.force = forceModifier;

					float spawnDelay = Random.Range(0.3f, 0.6f);
					yield return new WaitForSeconds(spawnDelay); // do one per-frame so that they can collide-resolve with each other.
				}

				yield return new WaitForSeconds(timeBetweenBursts);
			}
		}
```

What the new lines are doing, is adding a random direction to the bubble's velocity so that they don't all stack up on top of each other. We do force the y component to a known good value so that the bubbles are guaranteed to go up and not stick. There is a problem though, and that is once a bubble is spawned, it's alive forever. One can easily imagine how this would chew through memory and eventually become a problem. Open up Bubble.cs again. Add the following method:


```
		void DestroyBubble()
		{
			Destroy(gameObject);
		}
```

And change the Start() method to the following:

```
		// Use this for initialization
		void Start()
		{
			interaction.OnGraspBegin += DoBubblePop;

			Invoke("DestroyBubble", 10);
		}
```

This will call the DestroyBubble method after 10 seconds from the time Start() is called. We now have a somewhat fixed limit to the number of bubbles that will ever spawn. It should be noted though, that this is a sub-optimal method of creating objects like this, objects that are intentionally-short lived and spawned frequently. We did it this way here for simplicity's sake, but in a production project you should use [Object Pools](https://unity3d.com/learn/tutorials/topics/scripting/object-pooling)

Have fun popping bubbles!


## Ways to improve the program on your own:
Try spawning a [particle effect](https://docs.unity3d.com/Manual/class-ParticleSystem.html) when the bubble pops to add some juice to the interaction.

Add a popped bubble counter.

Change the color of the bubbles

Defer bubble spawning until the user presses an InteractionButton (See the Basic UI interaction engine sample scene for how to do this)

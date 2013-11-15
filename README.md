three.js leap FPcontrols
=================

## Introduction

[Three.js](http://threejs.org) provides several control classes in order to interact with your 3D world.
They are located under `three.js/examples/js/controls` in your Three.js directory.
With **three.js leap FPcontrols** you can use your leap motion controller to look arround and move forward with any camera object.

## Requirements 

- a [leap motion](http://leapmotion.com) controller
- [leapjs](http://js.leapmotion.com) (javascript sdk for leap motion controller)
- [Three.js](http://threejs.org) (javascript library for creating and displaying 3D computer graphics on a browser)
- a browser which supports WebGL, e.g. Google Chrome


## Usage

First, load the necessary javascript files in your HTML document
```html
<script src="three.min.js"></script>
<script src="leap.min.js"></script>
<script src="js/LeapFPControls.js"></script>
```

Create a scene, a camera, some object... add everything to the scene, create a renderer object... usual stuff with THREE.js

Then create a "controls" object
```javascript
var controls = new THREE.LeapFirstPersonControls(camera);
```

There are several parameters you can set, none of them mandatory as they have reasonable default values, but you can edit them:

```javascript
	                controls.movementSpeed = MOVESPEED;
					controls.lookSpeed = LOOKSPEED;
					controls.lookVertical = true; 
					controls.noFly = true;
					controls.canWalk = true;
					controls.neutralArea = 0.05;

					```
With canWalk enabled, you move forward whenever your finger is past the 0 z-axis threshold.
With neutralArea you set the size of the area where the cursor is considered centered (holding your finger precisely on the 0,0 coordinate is almost impossible, and it becomes anoying if your view is always turning, so we define an area as being equivalent to 0,0).

LeapFPControls tracks one single finger (or pointable tool) at a time, it looks at its position and its angle. It needs values inside a "leapvars" object with the following format:

```javascript
var leapvars =
		 {
		 	leapX: 50,
		 	leapY: 50,
		 	leapZ:100,
		 	leapDirX:0,
		 	leapDirY:0,
		 	leapDirZ:0,
		 	fingerIn: false
		 };
		 ```

You then need to update the controls through your "leap-loop" with every iteration


```javascript
leap.loop(function(frame, done) {
		  		
	//Get finger position values	
				  

	var delta = clock.getDelta();
	controls.update(delta);
	done();
});
```


A full implementation of the Leap settings would look like this:

```javascript
var controls = new THREE.LeapFirstPersonControls(camera);

var leapvars =
		 {
		 	leapX: 50,
		 	leapY: 50,
		 	leapZ:100,
		 	leapDirX:0,
		 	leapDirY:0,
		 	leapDirZ:0,
		 	fingerIn: false
		 };



function startLeap() 
	 {
				 // Setup Leap loop with frame callback function
			    var leap = new Leap.Controller({enableGestures: true});
			        var region = new Leap.UI.Region(
			          [0, 150, -100], 
			          [200, 250, 100]
			        );

				leap.addStep(new Leap.UI.Cursor())
				leap.addStep(region.listener({nearThreshold:50}));

				console.log("leap initialized");

				//BIG loop
				leap.loop(function(frame, done) {
		  		// Body of callback function

					//pointer
					if (frame.cursorPosition) {
				      	leap_enabled = true;
				      	leapvars.fingerIn = true;

				        var leapPosition = region.mapToXY(frame.cursorPosition, window.innerWidth, window.innerHeight);

				        leapvars.leapX = (leapPosition[0]);
				        leapvars.leapY = (leapPosition[1]);
				        leapvars.leapZ = leapPosition[2];
				   
				      }
				      else
				      {
				      	leapvars.fingerIn = false;
				      }

				      if(frame.pointables.length > 0)
				      {
				      	var pointable = frame.pointables[0];
				      	leapvars.leapDirX = -pointable.direction[0];
				      	leapvars.leapDirY = -pointable.direction[1];
				      	leapvars.leapDirZ = pointable.direction[2]; 	 
				      }

				  	var delta = clock.getDelta();
					controls.update(delta);
				    done();
				});
	}
```



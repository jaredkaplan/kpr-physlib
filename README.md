Simple Physics Library for KPR
==============================

A simple particle based physics library using canvas in KPR - based off of Björn Lindberg's excellent [Blob Salad](http://dev.opera.com/articles/blob-sallad-canvas-tag-and-javascript) article, and Thomas Jakobsen's paper on [Advanced Character Physics](http://web.archive.org/web/20100111035201/http://www.teknikus.dk/tj/gdc2001.htm). This library allows you to create models by specifying point masses and constraints, as well as surface areas for rendering. Although it only provides a very basic set of features required for doing physically-based modeling, it can still be used to produce some interesting simulations and effects.

Getting Started
---------------

This library can be imported into Kinoma Studio and added to an application's build path settings. You can then use the library modules to create models and animations that get rendered into a KPR Canvas object.

Usage
-----

The first step in using the physics library is creating a *Canvas* object that inherits the *Scene* module *CanvasBehavior*. The *CanvasBehavior* provides an interface for specifying the simulation environment and adding models to the *Canvas*.

```javascript
// get reference to required library modules
var Scene = require( "Scene" );
var Vector = require( "Vector" );

// create the configuration data for the new scene canvas
var config = new Scene.SceneConfiguration( 100, Vector.newInstance( 0, 3.0 ), null, "white", true, true, true );

// create the canvas object
var canvas = new Canvas( {top:0, left:0, bottom:0, right:0} );

// set the canvas's behavior to a new Scene module CanvasBehavior
// and pass in the scene configuration data
canvas.behavior = new Scene.CanvasBehavior( canvas, {config:config} );

// add the scene canvas to the application
application.add( canvas );
```
Creating a *Model* and adding it to the *Canvas*:

```javascript
// get a reference to the Model module
var Model = require( "Model" );

// create a new empty model    
var model = Model.newInstance();

// add some points to the model
var p1 = model.addPoint( .5, 0 );
var p2 = model.addPoint( 1, .5 );
var p3 = model.addPoint( .5, 1 );
var p4 = model.addPoint( 0, .5 );

// add the model to the scene at the specified location
canvas.delegate( "addModel", model, Vector.newInstance( 1, 0 ) );
```

Once you have defined a model with some points, you will need to create constraints that specify rules for how close points can get to each other. When the simulation runs it will satisfy each of the specified constraints for each update cycle. Different classes of constraints can be specified to achieve various behaviors used to satisfy the constraint.

```javascript
// get a reference to the Constraint module
var Constraint = require( "Constraint" );

// add constraints to the model
model.addConstraint( new Constraint.Stick( p1, p2 ) );
model.addConstraint( new Constraint.Stick( p2, p3 ) );
model.addConstraint( new Constraint.Stick( p3, p4 ) );
model.addConstraint( new Constraint.Stick( p4, p1 ) );
model.addConstraint( new Constraint.Stick( p1, p3 ) );
model.addConstraint( new Constraint.Stick( p2, p4 ) );
```

Finally, in order for a model to render inside of your canvas, you must define surfaces on the model that you wish to be drawn. A *Surface* is an object that specifies a list of points that make up the surface, as well as the *Renderer* that is used to draw it. 

```javascript
// get a reference to the required module
var Surface = require( "Surface" );
var DefaultSurfaceRenderer = require( "DefaultSurfaceRenderer" );

// create a new default surface configuration
var surfaceConfig = new DefaultSurfaceRenderer.DefaultSurfaceConfiguration( "plain", {width:10,color:"#7F7F7F"}, {color:"green"} );

// create the surface instance
var surface = Surface.newInstance( model.points, DefaultSurfaceRenderer.getInstance(), surfaceConfig );
			
// add the new surface to the model
model.addSurface( surface );
```

The default surface renderer allows you to specify a few drawing options, such as the color and line style of the surface, but it is possible to create custom renderers as well. A custom render can extend the default surface renderer to add additional drawing functionality.

```javascript
// create the custom surface renderer constructor
var CustomSurfaceRenderer = function() {
	 DefaultSurfaceRenderer.DefaultSurfaceRenderer.call( this );
};

// create the custom surface renderer class definition
CustomSurfaceRenderer.prototype = Object.create( DefaultSurfaceRenderer.prototype, 
{
	 draw:
	 { 
		  value: function( ctx, scale, env, model, pts, config )
		  {
				DefaultSurfaceRenderer.prototype.draw.call( this, ctx, scale, env, model, pts, config );

				// get the center point for the model
				var center = model.calculateCenter();

				// draw a circle around the model using the center point and
				// the distance to the first point as the radius
				ctx.beginPath();
				ctx.arc( center.x * scale, center.y * scale, center.distance( pts[0] ) * scale + 10, 0, 2 * Math.PI, false );
				ctx.closePath();
				ctx.lineWidth = 3;
				ctx.strokeStyle = "red";
				ctx.stroke();
		  }
	 }
});

// create the surface instance
var surface = Surface.newInstance( model.points, new CustomSurfaceRenderer(), surfaceConfig );
			
// add the new surface to the model
model.addSurface( surface );
```

It's possible to add behavior for responding to user touch events by specifying an *Interaction* object in your canvas. A simple behavior can be added that responds to selecting a model for example. Default behaviors defined in the *Interaction* class can be extended to provide common behavior such as selecting and moving models in the canvas.

```javascript
var Interaction = require( "Interaction" );

// create the custom interaction behavior constructor
var CustomInteractionBehavior = function() {
	 Interaction.DefaultDragInteractionBehavior.call( this );
};

// create the custom interaction behavior class definition
CustomInteractionBehavior.prototype = Object.create( Interaction.DefaultDragInteractionBehavior.prototype, 
{
	 handleSelectModel:
	 {
		  value: function( content, model, id, pt, ticks ) {
				// call inherited handleSelectModel behavior
				Interaction.DefaultDragInteractionBehavior.prototype.handleSelectModel.call( this, content, model, id, pt, ticks );
		  }
	 }
});

// set the canvas interaction behavior to our custom class
canvas.delegate( "setInteractionBehavior", new CustomInteractionBehavior() );

// enable touch events on our canvas object
canvas.active = true;
```

In order to simplify the creation and persistance of models, there is an interface for loading models from json data.

```javascript
// create json data describing model point masses and constraints
var data = {
	  points:[{x:0,y:0},{x:1,y:0},{x:1,y:1},{x:.0,y:1}],
	  constraints:[{p1:0,p2:1},{p1:1,p2:2},{p1:2,p2:3},{p1:3,p2:1},{p1:0,p2:2},{p1:1,p2:3}],
	  surfaces:[{points:[0,1,2,3],renderer:"DefaultSurfaceRenderer",config:{fill:{color:"yellow"},stroke:{width:3,color:"black"}}}]
};
  
// add the model to the canvas
canvas.delegate( "addModel", Model.newInstance( data ) );
```

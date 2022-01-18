<script>
import Matter from 'matter-js';
import { onMount } from 'svelte';
    
onMount(() => {
    createScene();
});
    
function createScene() {
// module aliases
var Engine = Matter.Engine,
    Events = Matter.Events,
    Render = Matter.Render,
    Runner = Matter.Runner,
    Body = Matter.Body,
    Common = Matter.Common,
    Composite = Matter.Composite,
    Composites = Matter.Composites,
    Constraint = Matter.Constraint,
    MouseConstraint = Matter.MouseConstraint,
    Mouse = Matter.Mouse,
    Bodies = Matter.Bodies,
    Vector = Matter.Vector;
    
// create an engine
var engine = Engine.create();
var world = engine.world;
    
// create a renderer
var render = Render.create({
    element: document.getElementById("matter-js"),
    engine: engine,
    options: {
        width: 800,
        height: 600,
        showAngleIndicator: true,
        pixelRatio: 1,
        background: 'rgba(0, 0, 0, 0)',
        hasBounds: false,
        enabled: true,
        wireframes: false,
        showSleeping: true,
        }
    });
    
// run the renderer
Render.run(render);
    
// create runner
var runner = Runner.create();
    
// run the engine
Runner.run(runner, engine);

// create stairs
var stairCount = (render.bounds.max.y - render.bounds.min.y) / 50;

var stack = Composites.stack(0, 0, stairCount + 2, 1, 0, 0, function(x, y, column) {
    return Bodies.rectangle(x - 50, y + column * 50, 100, 1000, {
        isStatic: true,
        render: {
            fillStyle: '#060a19',
            strokeStyle: '#ffffff',
            lineWidth: 1
        }
    });
});

// create obstacles
var obstacles = Composites.stack(300, 0, 15, 3, 10, 10, function(x, y, column) {
        var sides = Math.round(Common.random(1, 8)),
            options = {
                render: {
                    fillStyle: Common.choose(['#f19648', '#f5d259', '#f55a3c', '#063e7b', '#ececd1'])
                }
            };

        switch (Math.round(Common.random(0, 1))) {
        case 0:
            if (Common.random() < 0.8) {
                return Bodies.rectangle(x, y, Common.random(25, 50), Common.random(25, 50), options);
            } else {
                return Bodies.rectangle(x, y, Common.random(80, 120), Common.random(25, 30), options);
            }
        case 1:
            return Bodies.polygon(x, y, sides, Common.random(25, 50), options);
        }
    });


Composite.add(world, [stack, obstacles]);

var timeScaleTarget = 1,
        counter = 0;

    Events.on(engine, 'afterUpdate', function(event) {
        // tween the timescale for slow-mo
        if (mouse.button === -1) {
            engine.timing.timeScale += (timeScaleTarget - engine.timing.timeScale) * 0.05;
        } else {
            engine.timing.timeScale = 1;
        }

        counter += 1;

        // every 1.5 sec
        if (counter >= 60 * 1.5) {



            // reset counter
            counter = 0;
        }

        for (var i = 0; i < stack.bodies.length; i += 1) {
            var body = stack.bodies[i];

            // animate stairs
            Body.translate(body, {
                x: -0.5 * engine.timing.timeScale,
                y: -0.5 * engine.timing.timeScale
            });

            // loop stairs when they go off screen
            if (body.position.x < -50) {
                Body.setPosition(body, {
                    x: 50 * (stack.bodies.length - 1),
                    y: 25 + render.bounds.max.y + (body.bounds.max.y - body.bounds.min.y) * 0.5
                });
                
                Body.setVelocity(body, {
                    x: 0,
                    y: 0
                });
            }
        }

        for (i = 0; i < ragdolls.composites.length; i += 1) {
            var ragdoll = ragdolls.composites[i],
                bounds = Composite.bounds(ragdoll);

            // move ragdolls back to the top of the screen
            if (bounds.min.y > render.bounds.max.y + 100) {
                Composite.translate(ragdoll, {
                    x: -bounds.min.x * 0.9,
                    y: -render.bounds.max.y - 400
                });
            }
        }

        for (i = 0; i < obstacles.bodies.length; i += 1) {
            var body = obstacles.bodies[i],
                bounds = body.bounds;

            // move obstacles back to the top of the screen
            if (bounds.min.y > render.bounds.max.y + 100) {
                Body.translate(body, {
                    x: -bounds.min.x,
                    y: -render.bounds.max.y - 300
                });
            }
        }
    });

// add mouse control and make the mouse revolute
var mouse = Mouse.create(render.canvas),
    mouseConstraint = MouseConstraint.create(engine, {
        mouse: mouse,
        constraint: {
            stiffness: 0.6,
            length: 0,
            angularStiffness: 0,
            render: {
                visible: false
            }
        }
    });

Composite.add(world, mouseConstraint);

// keep the mouse in sync with rendering
render.mouse = mouse;


};
    
</script>
    
<div class="flex justify-center">
    <div id="matter-js" class="z-10"></div>
</div>
    

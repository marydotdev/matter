<script>
import Matter from 'matter-js';
import { onMount } from 'svelte';
    
onMount(() => {
    createScene();
});
    
function createScene() {
// module aliases
var Engine = Matter.Engine,
    Render = Matter.Render,
    Runner = Matter.Runner,
    Composites = Matter.Composites,
    Events = Matter.Events,
    Constraint = Matter.Constraint,
    MouseConstraint = Matter.MouseConstraint,
    Mouse = Matter.Mouse,
    Composite = Matter.Composite,
    Bodies = Matter.Bodies;
    
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
    
// add bodies
var ground = Bodies.rectangle(395, 600, 815, 50, { isStatic: true, render: { fillStyle: '#060a19' } }),
    rockOptions = { density: 0.004 },
    rock = Bodies.polygon(170, 450, 8, 20, rockOptions),
    anchor = { x: 170, y: 450 },
    elastic = Constraint.create({ 
        pointA: anchor, 
        bodyB: rock, 
        stiffness: 0.05
    });

var pyramid = Composites.pyramid(500, 300, 9, 10, 0, 0, function(x, y) {
    return Bodies.rectangle(x, y, 25, 40);
});

var ground2 = Bodies.rectangle(610, 250, 200, 20, { isStatic: true, render: { fillStyle: '#060a19' } });

var pyramid2 = Composites.pyramid(550, 0, 5, 10, 0, 0, function(x, y) {
    return Bodies.rectangle(x, y, 25, 40);
});

Composite.add(engine.world, [ground, pyramid, ground2, pyramid2, rock, elastic]);

Events.on(engine, 'afterUpdate', function() {
    if (mouseConstraint.mouse.button === -1 && (rock.position.x > 190 || rock.position.y < 430)) {
        rock = Bodies.polygon(170, 450, 7, 20, rockOptions);
        Composite.add(engine.world, rock);
        elastic.bodyB = rock;
    }
});
    
// add mouse control
var mouse = Mouse.create(render.canvas),
    mouseConstraint = MouseConstraint.create(engine, {
        mouse: mouse,
        constraint: {
            stiffness: 0.2,
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
    

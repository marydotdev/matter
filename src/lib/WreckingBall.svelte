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
    MouseConstraint = Matter.MouseConstraint,
    Mouse = Matter.Mouse,
    Composite = Matter.Composite,
    Constraint = Matter.Constraint,
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
var rows = 10,
    yy = 600 - 25 - 40 * rows;
    
    var stack = Composites.stack(450, yy, 5, rows, 0, 0, function(x, y) {
        return Bodies.rectangle(x, y, 30, 30);
    });
    
    Composite.add(world, [
        stack,
        // floor
        Bodies.rectangle(400, 600, 600, 50, { isStatic: true }),
    ]);
    
    var ball = Bodies.circle(400, 400, 30, { density: 0.04, frictionAir: 0.005});
    
    Composite.add(world, ball);
    Composite.add(world, Constraint.create({
        pointA: { x: 400, y: 250 },
        bodyB: ball
    }));
    
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

    

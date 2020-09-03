<script lang="ts">
    import * as PIXI from 'pixi.js';
    import { Vector } from 'matter-js';
    import { onDestroy, onMount } from 'svelte';
    const phidget22 = (window as any).phidget22;
    
    let phiConnection: any | undefined;
    let voltRatio = 0.5;
    let voltRatioPhidget: {
        open: (timeout?: number) => Promise<void>;
        close: () => void;
        setIsHubPortDevice: (b: boolean) => void;
        setHubPort: (port: number) => void;
        setDataInterval: (i: number) => void;
        onVoltageRatioChange: (vr: number) => void;
     } | undefined;
     
    const setupVoltRatio = async () => {
        phiConnection = new phidget22.Connection({
            hostname: 'localhost',
            port: 8989,
            passwd: '',
        });

        await phiConnection.connect();
        console.log('Phidgets connected');

        voltRatioPhidget = new phidget22.VoltageRatioInput();
        voltRatioPhidget.setIsHubPortDevice(true);
        voltRatioPhidget.setHubPort(0);
        voltRatioPhidget.onVoltageRatioChange = vr => voltRatio = vr;
        
        console.log('Connecting voltratio');
        await voltRatioPhidget.open();
        console.log('voltRatio open');
        voltRatioPhidget.setDataInterval(50);
    };

    let div: HTMLDivElement | undefined;
    let app: PIXI.Application | undefined;

    onMount(async () => {
        if (div === undefined) {
            console.error('DIV element missing');
            return;
        }

        app = new PIXI.Application({
            width: 1280,
            height: 800,
        });
        div.appendChild(app.view);

        const shipTex = PIXI.Texture.from('/assets/ship.png');
        const ship = {
            sprite: PIXI.Sprite.from(shipTex),
            velocity: Vector.create(0, 0),
            acceleration: Vector.create(0, 0),
        }
        ship.sprite.anchor.set(0.5);
        app.stage.addChild(ship.sprite);
        ship.sprite.position.set(app.screen.width/2, app.screen.height/2);

        await setupVoltRatio();


        const MAX_VEL = 7.5;

        app.ticker.add((dt: number) => {
            const rotationChange = -voltRatio + 0.5;
            ship.sprite.rotation += rotationChange * dt * 0.25;

            ship.acceleration.y = Math.sin(ship.sprite.rotation - Math.PI/2);
            ship.acceleration.x = Math.cos(ship.sprite.rotation - Math.PI/2);

            Vector.add(ship.velocity, ship.acceleration, ship.velocity);
            if (Vector.magnitudeSquared(ship.velocity) > MAX_VEL) {
                ship.velocity = Vector.normalise(ship.velocity);
                ship.velocity = Vector.mult(ship.velocity, MAX_VEL);
            }
            Vector.add(ship.sprite, ship.velocity, ship.sprite);

            const { x, y } = ship.sprite;
            if (x < 0) ship.sprite.x = app.screen.width;
            if (x > app.screen.width) ship.sprite.x = 0;
            if (y < 0) ship.sprite.y = app.screen.height;
            if (y > app.screen.height) ship.sprite.y = 0;
        });
    });

    onDestroy(() => {
        if (app) app.destroy();
        if (voltRatioPhidget) voltRatioPhidget.close();
        if (phiConnection) {
            phiConnection.close();
            phiConnection.delete();
        }
    })
</script>

<div bind:this={div}>
    
</div>

<style>
    div {
        border: 1px solid red;
    }
</style>
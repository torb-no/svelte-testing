<script type="ts">
    const phidget22 = (window as any).phidget22;
    
    const phiConnection = new phidget22.Connection({
        hostname: 'localhost',
        port: 8989,
        passwd: '',
    });
    
    let value = -1;

    (async () => {

        await phiConnection.connect();

        const voltRatio = new phidget22.VoltageRatioInput();
        voltRatio.setIsHubPortDevice(true);
        voltRatio.setHubPort(0);
        
        voltRatio.onVoltageRatioChange = (vr: number) => {
            value = vr;
        };
        
        await voltRatio.open();
        
        voltRatio.setDataInterval(50);

    })();
    
</script>

<div>
    voltratio: {value}
</div>
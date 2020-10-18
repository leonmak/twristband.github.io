<script>
    import AHRS from 'ahrs';
    const madgwick = new AHRS({
        sampleInterval: 1, // debug freq
    });
    window.navigator.permissions.query({name:'bluetooth'})

    // client
    window.navigator.bluetooth.requestDevice({
        filters: [{
            services: ['4fafc301-1fb5-459e-8fcc-c5c9c331914b']
        }]})
            .then(device => device.gatt.connect())
            .then(server => {
                // Getting Battery Service...
                return server.getPrimaryService('4fafc301-1fb5-459e-8fcc-c5c9c331914b');
            })
            .then(service => {
                // Getting Battery Level Characteristic...
                return service.getCharacteristic('beb5483c-36e1-4688-b7f5-ea07361b26a8');
            })
            .then(characteristic => characteristic.startNotifications())
            .then(characteristic => {
                characteristic.addEventListener('characteristicvaluechanged', handleCharacteristicValueChanged);
                console.info('Notifications have been started.');
            })
            .catch(error => {
                console.error(error);
            });


    function handleCharacteristicValueChanged(event) {
        const dataView = event.target.value;
        // parse float array (little endian esp32, 4 bytes for float)
        const [ax, ay, az, gx, gy, gz, mx, my, mz] = [...Array(9).keys()]
                .map(i => dataView.getFloat32(i * 4, true).toFixed(2));
        // LSM9DS1 is left handed, madgwick is right handed
        // https://www.lythaniel.fr/index.php/2016/08/20/lsm9ds1-madgwicks-ahr-filter-and-robot-orientation
        madgwick.update(gy, gx, gz, ay, ax, az, my, -mx, mz);

        console.log(madgwick.getEulerAngles());
        console.log(madgwick.getQuaternion());
    }

</script>
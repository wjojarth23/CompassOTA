<script>
  let device;
  let characteristic;
  let status = "Not connected";
  let file;

  // Connect to Bluetooth device
  async function connectBluetooth() {
    try {
      status = "Connecting...";
      device = await navigator.bluetooth.requestDevice({
        acceptAllDevices: true,
        optionalServices: ["00001800-0000-1000-8000-00805f9b34fb"], // Generic Access service
      });

      const server = await device.gatt.connect();
      const service = await server.getPrimaryService("00001800-0000-1000-8000-00805f9b34fb");
      characteristic = await service.getCharacteristic("00002a00-0000-1000-8000-00805f9b34fb"); // Device Name characteristic

      status = `Connected to ${device.name}`;
    } catch (error) {
      console.error(error);
      status = "Connection failed.";
    }
  }

  // Upload the firmware file
  async function uploadFirmware() {
    if (!characteristic) {
      status = "No Bluetooth device connected!";
      return;
    }

    if (!file) {
      status = "Please select a file!";
      return;
    }

    status = "Uploading...";
    try {
      const arrayBuffer = await file.arrayBuffer();
      const chunkSize = 512; // ESP32 Bluetooth buffer size
      for (let i = 0; i < arrayBuffer.byteLength; i += chunkSize) {
        const chunk = arrayBuffer.slice(i, i + chunkSize);
        await characteristic.writeValue(new Uint8Array(chunk));
        status = `Uploading... ${(i / arrayBuffer.byteLength) * 100}%`;
      }

      status = "Upload complete!";
    } catch (error) {
      console.error(error);
      status = "Upload failed!";
    }
  }
</script>

<style>
  button {
    margin: 10px;
    padding: 10px 20px;
    font-size: 16px;
  }
  input {
    margin: 10px;
  }
</style>

<h1>ESP32 OTA via Bluetooth</h1>
<p>Status: {status}</p>

<button on:click={connectBluetooth}>Connect to Bluetooth</button>
<br />

<label for="file">Select firmware file:</label>
<input id="file" type="file" on:change={(e) => (file = e.target.files[0])} />
<br />

<button on:click={uploadFirmware}>Upload Firmware</button>

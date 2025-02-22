### Web MIDI

#### Experimental! 

Hydra can be used with [Web MIDI](https://webaudio.github.io/web-midi-api/) for an extra layer of control to your visuals.  At this time this requires some running of code on the
browser console (Press F12 in Chrome to access).  This page only considers MIDI Continuous Controllers (CC) but other types of data may be accessible.

This is a generic script that doesn't care what Midi Channel you're broadcasting on and maps a normalized value 0.0-1.0 into an array named cc. 

### Console Script
This portion should be ran in the console & will register Web MIDI & map the incoming CC data to a set of parameters. For simplicity, these
parameters are named to match the CC number. The CC values are normally in a range from 0-127, but we've also normalized them to be in a range of 0.0-1.0.

```js
// register WebMIDI
navigator.requestMIDIAccess()
    .then(onMIDISuccess, onMIDIFailure);

function onMIDISuccess(midiAccess) {
    // console.log(midiAccess);
    var inputs = midiAccess.inputs;
    var outputs = midiAccess.outputs;
    
    console.log('Available MIDI Inputs & ID:')
    inputs.forEach((i) => console.log(i.name, i.id)) 
   
    for (var input of midiAccess.inputs.values()){
        input.onmidimessage = getMIDIMessage;
    }
}

function onMIDIFailure() {
    console.log('Could not access your MIDI devices.');
}

// Create an array to hold our cc values and init to a normalized value
var cc = Array(128).fill(0.5);

// Change the device name to one of the names (or ID) from the available midi 
// inputs to make sure Hydra only reacts to the input of that controller
var controller = null;

getMIDIMessage = function(midiMessage) {
    var dev = midiMessage.target.name;
    var id = midiMessage.target.id;
    
    // Only set midi values if the input equals the device name or id (or is null)
    if (dev === controller || id == controller || !controller){
        var arr = midiMessage.data;
        var index = arr[1];

        // Normalize CC values to 0.0 - 1.0
        cc[index] = arr[2] / 127;

        // Uncomment to monitor incoming Midi
        // console.log(`Midi received: cc#${index}, value: ${arr[2]}, norm: ${cc[index]}`);
    }
}
```

### Hydra script
Now that these controls have been assigned to the cc[] array, we can start using them in Hydra.  As we've normalized the values 0-1 we can use
as-is with most functions or quickly remap them with various math.  

```js
// example midi mappings - Korg NanoKontrol2 CCs

// color controls with first three knobs
noise(4).color( ()=>cc[16], ()=>cc[17], ()=>cc[18] ).out()

// rotate & scale with first two faders
osc(10,0.2,0.5).rotate( ()=>(cc[0]*6.28)-3.14 ).scale( ()=>(cc[1]) ).out()

```

### Select specific controller
If you have multiple MIDI controllers connected to your computer Hydra will pick up the values from all controllers. You can select a specific device by adding the name of the device as a string to the `device` variable. You can find the list of connected devices with name and ID printed to the console.

```js
controller = "Korg NanoKontrol2"
```

In the case you have multiple controllers with the same name you can also use the devices ID like so:

```js
controller = -1511378908
```

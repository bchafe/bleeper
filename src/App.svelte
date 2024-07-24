<script>
  import * as Tone from "tone";
  
  //dropdown
  import XYPad from './XYPad.svelte'
  import MultiState from './MultiState.svelte'
  const height = 400;
	const width = 400;
	const color = '#fff'
	const background = '#000'
  let paramX = 0, paramY = 0, pageX = 0, pageY = 0;

 

  //Tone.getContext().lookAhead = 0;
  let isStarted = false;
  let isSpaceDown = false;
  
  //osc params
  let frequency = 440;
  let harmonicity = 0.2;
  let modulationIndex = 3;
  let carrierWave = "square";
  let modWave = "triangle";
 
  //chebyshev params
  let chebyOn = true;
  let order = 11;
  let chebyOversample = "none";
  let chebyWet = 0.1;

  //delay params
  let delayOn = true;
  let delayTime = Tone.Time("8n").toSeconds();
  let feedback = 0.5;
  let delayWet = 0.1;

  //lfo params
  let lfoOn = true;
  let lfoWave = "sine";
  let lfoFrequency = 10;
  let lfoMin = 0;
  let lfoMax = 5000;


  //tone.js node declarations
  let channel = new Tone.Channel({volume: -24}).toDestination();
  //let meter = new Tone.Meter();
  //channel.connect(meter);
  
  let limiter = new Tone.Limiter(-8);

  let cheby = new Tone.Chebyshev({
    order: order,
    oversample: chebyOversample,
    wet: chebyWet
  });
  
  let delay = new Tone.FeedbackDelay({
    delayTime: delayTime,
    feedback: feedback,
    wet: delayWet
  });
 
  let env = new Tone.AmplitudeEnvelope({
    attack: 0.001,
    decay: 0,
    sustain: 1,
    release: 0.01
  });
  

  let osc = new Tone.FMOscillator({
        frequency: frequency,
        type: carrierWave,
        modulationType: modWave,
        harmonicity: harmonicity,
        modulationIndex: modulationIndex
  }).chain(env, cheby, delay, limiter, channel).start();

  //xypad, lfo dropdown vars and functions
  let dropdownXindex = 1;
  let dropdownYindex = 2;
  let dropdownOptions = [
    {
      name: "None",
      max: 0,
      set value(val){
        
      },
    },
    {
      name: "OSC Frequency",
      max: 5000,
      set value(val){
        osc.frequency.value = val;
      },
    },
    {
      name: "OSC Harmonicity",
      max: 8,
      set value(val){
        osc.harmonicity.value = val;
      },
    }
  ];

  dropdownOptions.forEach( (option, i) => {
    //assign defaults for isSelected props (excluding option "None")
    if(i > 0){
      option['isSelectedX'] = false;
      option['isSelectedY'] = false;
      option['isSelectedLFO'] = false;
    }
  });

  function dropdownXindexChange(){
    var index = dropdownXindex;
    for(var i = 0; i < dropdownOptions.length; i++){
      //set selected to true for current selected option, false for the rest.
      dropdownOptions[i].isSelectedX = (i === index) ? true : false;
    }
    //assign value from interface to internal param (not reactive)
    dropdownOptions[dropdownXindex].value = paramX * dropdownOptions[dropdownXindex].max;
  }

  function dropdownYindexChange(){
    var index = dropdownYindex;
    for(var i = 0; i < dropdownOptions.length; i++){
      //set selected to true for current selected option, false for the rest.
      dropdownOptions[i].isSelectedY = (i === index) ? true : false;
    }
    //assign value from interface to internal param (not reactive)
    dropdownOptions[dropdownYindex].value = paramY * dropdownOptions[dropdownYindex].max;
  }

  //functions
  function toggleCheby(){
    chebyOn = !chebyOn;
    if(chebyOn){
      cheby.wet.value = chebyWet;
    }
    else{
      cheby.wet.value = 0;
    }
  }

  function toggleDelay(){
    delayOn = !delayOn;
    if(delayOn){
      delay.wet.value = delayWet;
    }
    else{
      delay.wet.value = 0;
    }
  }

  function triggerAttack(){
    init();
    console.log("Trigger attack");
    env.triggerAttack(Tone.context.currentTime, 1);
  }

  function triggerRelease(){
    console.log("Trigger release");
    env.triggerRelease(Tone.context.currentTime);
    //console.log(meter.getValue());
  }

  function onKeyDown(event){
    switch(event.key){
      case " ":
        event.preventDefault();
        if (event.repeat) return;
        isSpaceDown = true;
        triggerAttack();
        break;
    }
  }

  function onKeyUp(event){
    switch(event.key){
      case " ":
        event.preventDefault();
        isSpaceDown = false;
        triggerRelease();
        break;
    }
  }

  function init(){
    if(!isStarted){
      Tone.start();
      isStarted = true;
      console.log("context started");
    }
  }


  //reactive statements

  //Osc params reactive
  $: if (isStarted) osc.frequency.value = osc.toFrequency(frequency);
  $: if (isStarted) osc.harmonicity.value = harmonicity;
  $: if (isStarted) osc.modulationIndex.value = osc.toFrequency(modulationIndex);
  $: if (isStarted) osc.type = carrierWave; 
  $: if (isStarted) osc.modulationType = modWave;
  //Delay params reactive
  $: if (isStarted) delay.delayTime.value = delayTime;
  $: if (isStarted) delay.feedback.value = feedback;
  $: if (isStarted) if(delayOn) delay.wet.value = delayWet;
  //Chebyshev params reactive
  $: if (isStarted) cheby.order = order;
  $: if (isStarted) if(chebyOn) cheby.wet.value = chebyWet;
  $: if (isStarted) cheby.oversample = chebyOversample;

  //xypad reactive
  $: {      
      dropdownOptions[dropdownXindex].value = paramX * dropdownOptions[dropdownXindex].max;
      dropdownOptions[dropdownYindex].value = paramY * dropdownOptions[dropdownYindex].max;
  }


  
  function oscdebug(){
    var cache = [];
    let out = JSON.stringify(osc, (key, value) => {
      if (typeof value === 'object' && value !== null) {
        // Duplicate reference found, discard key
        if (cache.includes(value)) return;

        // Store value in our collection
        cache.push(value);
      }
      return value;
    });
    cache = null; // Enable garbage collection
    console.log(out);

  }

</script>

 
  <svelte:window 
on:keydown={onKeyDown} 
on:keyup={onKeyUp}
on:blur={() => onKeyUp(new KeyboardEvent('keyup', {'key': ' '}))}
on:contextmenu={() => onKeyUp(new KeyboardEvent('keyup', {'key': ' '}))}
/>



<div class="param-grid">
  <div class="param" id="osc-controls">
    <h3>FM Oscillator</h3>
    <p>Carrier Wave:
      <MultiState --width="250px" key={["sine", "triangle", "sawtooth", "square"]} value={["sine", "triangle", "sawtooth", "square"]} bind:currentValue={carrierWave} defaultIndex={3}></MultiState>
    </p>

    <p>Modulator Wave:
      <MultiState --width="250px" key={["sine", "triangle", "sawtooth", "square"]} value={["sine", "triangle", "sawtooth", "square"]} bind:currentValue={modWave} defaultIndex={1}></MultiState>
    </p>
    
    <p><label for="freq">Frequency: {frequency.toString().padStart(4, "0")}</label>
      <input type="range" id="freq" min="20" max="5000" step="1" bind:value={frequency}>
    </p>
    <p><label for="harm">Harmonicity {harmonicity.toFixed(1)}</label>
      <input type="range" id="harm" min="0" max="8" step="0.1" bind:value={harmonicity}>
    </p>
    <p><label for="modind">Mod. Index {modulationIndex.toFixed(1)}</label>
      <input type="range" id="modind" min="0" max="20" step="0.1" bind:value={modulationIndex}>
    </p>
  
  </div>

  <div class="param" id="cheby-controls">
    <h3><input type="checkbox" on:click={toggleCheby} checked={true}>Chebyshev Distortion</h3>
    <p><label for="order">Order: {order.toString().padStart(2, "0")}</label>
      <input type="range" id="order" min="0" max="24" step="1" bind:value={order}>
    </p>

    <p>
      Oversampling: 
      <MultiState --width="250px" key={["none", "2x", "4x"]} value={["none", "2x", "4x"]} bind:currentValue={chebyOversample}></MultiState>
    </p>

    <p><label for="wet">Dry/wet: {chebyWet.toFixed(2)}</label>
      <input type="range" id="wet" min="0" max="1" step="0.01" bind:value={chebyWet}>
    </p>
  </div>

  <div class="param" id="delay-controls">
    <h3><input type="checkbox" on:click={toggleDelay} checked={true}>Delay</h3>
    <p><label for="dtime">Time: {(delayTime*1000).toString().padStart(4, "0") + " ms"}</label>
      <input type="range" id="dtime" min="0" max="1" step="0.01" bind:value={delayTime}>
    </p>

    <p><label for="fb">Feedback: {feedback.toFixed(2)}</label>
      <input type="range" id="fb" min="0" max="1" step="0.01" bind:value={feedback}>
    </p>

    <p><label for="wet">Dry/wet: {delayWet.toFixed(2)}</label>
      <input type="range" id="wet" min="0" max="1" step="0.01" bind:value={delayWet}>
    </p>
  </div>
  <div class="param" id="xy-container">
    <div class="xy-dropdown-container">
      X:
      <select id="xdropdown" bind:value={dropdownXindex} on:change={dropdownXindexChange} on:click={() => onKeyUp(new KeyboardEvent('keyup', {'key': ' '}))}>
        {#each dropdownOptions as option, i}
          <option value={i} disabled={option.isSelectedY || option.isSelectedLFO}>
            {option.name}
          </option>
        {/each}
      </select>
  
      Y:
      <select id="ydropdown" bind:value={dropdownYindex} on:change={dropdownYindexChange} on:click={() => onKeyUp(new KeyboardEvent('keyup', {'key': ' '}))}>
        {#each dropdownOptions as option, i}
          <option value={i} disabled={option.isSelectedX || option.isSelectedLFO}>
            {option.name}
          </option>
        {/each}
      </select>
    </div>
  
    <XYPad
            {height}
            {width}
            {color} 
            {background}
            bind:paramX
            bind:paramY
  />
  <p>X: {paramX.toFixed(2)}, Y: {paramY.toFixed(2)}</p>
  </div>
</div>


<!--<button on:mousedown={() => triggerAttack()} on:mouseup={() => triggerRelease()} >fire</button>-->





<style>
  .xy-dropdown-container{
    margin: 10px;
  }
  
  .param-grid{
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
  .param{
    margin: 10px;
    border: 1px solid grey;
  }

  .param p{
    margin: 10px;
    text-align: left;
  }

  canvas{
    background-color: white;
    cursor: pointer;
  }

  button{
    border-radius: 0;
    background-color: black;
  }

  button.active{
    background-color: grey;
  }

  button:focus{
    outline: none;
  }

  button:hover{
    border-color: white;
  }


  input[type="checkbox"]{
    -webkit-appearance: none;
    appearance: none;
    background-color: #000;
    margin-right: 10px;
    font: inherit;
    color: grey;
    width: 1em;
    height: 1em;
    border: 2px solid grey;
    border-radius: 50%;
    transform: translateY(.3em);
    transition: border-color 0.1s;
  }

  input[type="checkbox"]:focus{
    outline: none;
  }



  input[type="checkbox"]:checked{
    background-color: white;
    border-color: black;
  }

  input[type="checkbox"]:hover{
    border-color: white;
  }

  input[type="range"]{
    width: 300px;
  }
</style>
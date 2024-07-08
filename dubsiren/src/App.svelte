<script>
  import * as Tone from "tone";
  

  //Tone.getContext().lookAhead = 0;
  let isStarted = false;
  let isSpaceDown = false;
  
  //osc params
  let frequency = 440;
  let harmonicity = 0.2;
  let modulationIndex = 3;
 
  //chebyshev params
  let chebyOn = true;
  let order = 11;
  let oversample = "none";
  let chebyWet = 0.1;

  //delay params
  let delayOn = true;
  let delayTime = Tone.Time("8n").toSeconds();
  let feedback = 0.5;
  let delayWet = 0.1;



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

  let channel = new Tone.Channel({volume: -24}).toDestination();
  
  let cheby = new Tone.Chebyshev({
    order: 11,
    oversample: "none",
    wet: 0.1
  });
  
  let delay = new Tone.FeedbackDelay({
    delayTime: "8n",
    feedback: 0.5,
    wet: 0.1
  });

  let env = new Tone.AmplitudeEnvelope({
    attack: 0.001,
    decay: 0,
    sustain: 1,
    release: 0.01
  });
  

  let osc = new Tone.FMOscillator({
        frequency: 200,
        type: "square",
        modulationType: "triangle",
        harmonicity: 0.2,
        modulationIndex: 3
  }).chain(env, cheby, delay, channel).start();


  function triggerAttack(){
    init();
    console.log("Trigger attack");
    env.triggerAttack(Tone.context.currentTime, 1);
  }

  function triggerRelease(){
    console.log("Trigger release");
    env.triggerRelease(Tone.context.currentTime);
  }

  function onKeyDown(event){
    if (event.repeat) return;
    switch(event.key){
      case " ":
        isSpaceDown = true;
        event.preventDefault();
        triggerAttack();
        break;
    }
  }

  function onKeyUp(event){
    switch(event.key){
      case " ":
        isSpaceDown = false;
        event.preventDefault();
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

$: if (isStarted){
  //Osc params reactive
  osc.frequency.value = osc.toFrequency(frequency);
  osc.harmonicity.value = harmonicity;
  osc.modulationIndex.value = osc.toFrequency(modulationIndex);
  //Delay params reactive
  delay.delayTime.value = delayTime;
  delay.feedback.value = feedback;
  if(delayOn) delay.wet.value = delayWet;
  //Cheby params reactive
  cheby.order = order;
  if(chebyOn) cheby.wet.value = chebyWet;

  
}

</script>

<div class="param-grid">
  <div class="param" id="osc-controls">
    <h3>Oscillator</h3>
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
    <h3>Chebyshev Distortion <button on:click={toggleCheby} class="toggle {chebyOn ? 'active' : ''}"></button></h3>
    <p><label for="order">Order {order}</label>
      <input type="range" id="order" min="0" max="24" step="1" bind:value={order}>
    </p>

    <p>
      Oversampling: 
      <button on:click={() => cheby.oversample = "none"}>None</button>
      <button on:click={() => cheby.oversample = "2x"}>2x</button>
      <button on:click={() => cheby.oversample = "4x"}>4x</button>
    </p>

    <p><label for="wet">Dry/wet: {chebyWet.toFixed(2)}</label>
      <input type="range" id="wet" min="0" max="1" step="0.01" bind:value={chebyWet}>
    </p>
  </div>

  <div class="param" id="delay-controls">
    <h3>Delay <button on:click={toggleDelay} class="toggle {delayOn ? 'active' : ''}"></button></h3>
    <p><label for="dtime">Time: {delayTime.toFixed(2)}</label>
      <input type="range" id="dtime" min="0" max="1" step="0.01" bind:value={delayTime}>
    </p>

    <p><label for="fb">Feedback: {feedback.toFixed(2)}</label>
      <input type="range" id="fb" min="0" max="1" step="0.01" bind:value={feedback}>
    </p>

    <p><label for="wet">Dry/wet: {delayWet.toFixed(2)}</label>
      <input type="range" id="wet" min="0" max="1" step="0.01" bind:value={delayWet}>
    </p>
  </div>
</div>
<button on:mousedown={() => triggerAttack()} on:mouseup={() => triggerRelease()} >fire</button>

<svelte:window 
on:keydown={onKeyDown} 
on:keyup={onKeyUp} />


<style>
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
  
  .toggle{
    background: black;
    width: 20px;
    height: 20px;
    border-radius: 0%;
    margin: 2px;
    border: 2px solid grey;
  }

  .toggle.active{
    background: white;
  }
</style>
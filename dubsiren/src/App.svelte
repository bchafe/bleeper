<script>
  import * as Tone from "tone";

  //Tone.getContext().lookAhead = 0;
  let isStarted = false;
  let isSpaceDown = false;
//osc params
  let frequency = 440;
  let harmonicity = 0.2;
  let modulationIndex = 3;
 
  let channel = new Tone.Channel({volume: -24}).toDestination();
  
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
  }).chain(env, delay, channel).start();

  

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
  osc.frequency.value = osc.toFrequency(frequency);
  osc.harmonicity.value = harmonicity;
  osc.modulationIndex.value = osc.toFrequency(modulationIndex);
}

</script>


<div class="osc-controls">
  <h3>Osc. Controls</h3>
  <p><label for="freq">Frequency: {frequency}</label>
    <input type="range" id="freq" min="20" max="5000" step="1" bind:value={frequency}>
  </p>
  <p><label for="freq">Harmonicity {harmonicity}</label>
    <input type="range" id="freq" min="0" max="8" step="0.1" bind:value={harmonicity}>
  </p>
  <p><label for="freq">Mod. Index {modulationIndex}</label>
    <input type="range" id="freq" min="0" max="20" step="0.1" bind:value={modulationIndex}>
  </p>
  <button on:mousedown={() => triggerAttack()} on:mouseup={() => triggerRelease()} >fire</button>
</div>

<svelte:window 
on:keydown={onKeyDown} 
on:keyup={onKeyUp} />

<style>

</style>
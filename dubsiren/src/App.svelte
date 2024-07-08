<script>
  import * as Tone from "tone";

  //Tone.getContext().lookAhead = 0;
  let isStarted = false;
  let isSpaceDown = false;
//osc params
  let frequency = 440;
  let harmonicity = 0.2;
  let modulationIndex = 3;
 
//delay params
  let delayTime = Tone.Time("8n").toSeconds();
  let feedback = 0.5;
  let wet = 0.1;

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

  delay.delayTime.value = delayTime;
  delay.feedback.value = feedback;
  delay.wet.value = wet;
}

</script>


<div class="osc-controls">
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
  <button on:mousedown={() => triggerAttack()} on:mouseup={() => triggerRelease()} >fire</button>
</div>

<div class="delay-controls">
  <h3>Delay</h3>
  <p><label for="dtime">Time: {delayTime.toFixed(2)}</label>
    <input type="range" id="dtime" min="0" max="1" step="0.01" bind:value={delayTime}>
  </p>

  <p><label for="fb">Feedback: {feedback.toFixed(2)}</label>
    <input type="range" id="fb" min="0" max="1" step="0.01" bind:value={feedback}>
  </p>

  <p><label for="wet">Dry/wet: {wet.toFixed(2)}</label>
    <input type="range" id="wet" min="0" max="1" step="0.01" bind:value={wet}>
  </p>
</div>

<svelte:window 
on:keydown={onKeyDown} 
on:keyup={onKeyUp} />

<style>

</style>
<script>
  import * as Tone from "tone";

  let freq = 440;
 
  let channel = new Tone.Channel({volume: -16}).toDestination();
  
  let delay = new Tone.FeedbackDelay();

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
  }).chain(env, channel).start();

  function updateFrequency(){
    osc.frequency.value = osc.toFrequency(freq);
  }

  function triggerAttack(){
    console.log("Trigger attack");
    env.triggerAttack(0,1);
  }

  function triggerRelease(){
    console.log("Trigger release");
    env.triggerRelease(0);
  }

</script>


<div class="controls">
  <p><label for="freq">Frequency: {freq}</label>
    <input type="range" id="freq" min="20" max="5000" step="1" bind:value={freq} on:change={() => updateFrequency()}>
  </p>
  <button on:mousedown={() => triggerAttack()} on:mouseup={() => triggerRelease()} >fire</button>
</div>
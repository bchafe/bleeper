<script>
	import { onMount } from 'svelte'
	
	export let width = 300
	export let height = 300
	export let color = '#555'
	export let background = '#111'
    

    export let paramX = 0;
    export let paramY = 0;

	export let border = 10;

	let pageX = 0;
    let pageY = 0;

	let canvas
	let context
	let isInteracting
	let start
	
	let t, l, originY, originX 

	//mouse event shield
	const mouseEventShield = document.createElement("div");
  	mouseEventShield.setAttribute("class", "mouse-over-shield");
  	mouseEventShield.addEventListener("mouseover", (e) => {
    	e.preventDefault();
    	e.stopPropagation();
 	});

	



	onMount(() => {
		context = canvas.getContext('2d');
		context.lineWidth = 2;
		context.strokeStyle = color;
	})
	
	
	const handleSize = (e) => {
		pageX = e.pageX;
		pageY = e.pageY;
	}
	
	const handleStart = ((e) => { 
		isInteracting = true;
		
		handleSize(e);

		if (e.stopPropagation) e.stopPropagation();
		if (e.preventDefault) e.preventDefault();
		
		//activate mouseEventShield
		if (e.type === "mousedown") document.body.append(mouseEventShield);

		//get origin of canvas
		const { top, left } = canvas.getBoundingClientRect(); 
		originY = top;
		originX = left;

		//calculate relative position from page position
		let x = pageX - originX;
		let y = pageY - originY;

		//xy value processing and output
		if(x < border) x = 0;
		else if (x > width - border) x = width;
		
		if(y < border) y = 0;
		else if (y > height - border) y = height;
		
		paramX = x / width;
		paramY = y / height;
		paramY = 1-paramY; //flip Y value so coords are akin to (X>0,Y>0) on a cartesian plane.

	

		//clear canvas
		context.clearRect(0, 0, width, height);

		//draw circle
		context.beginPath();
		context.arc(x, y, border, 0, 2 * Math.PI);
		context.closePath();
		context.stroke();         
	})
	
	
	const handleEnd = (e) => { 
		isInteracting = false;

		if (e.type === "mouseup") {
      		if (document.body.contains(mouseEventShield))
        		document.body.removeChild(mouseEventShield);
		}
	}
	
	const handleMove = ((e) => {
		if(!isInteracting) return
		handleStart(e);
	})
	




</script>

<svelte:window
  on:touchmove|nonpassive={handleMove}
  on:touchcancel={handleEnd}
  on:touchend={handleEnd}
  on:mousemove={handleMove}
  on:mouseup={handleEnd}
  on:resize={handleSize}
/>
<canvas
				{width}
				{height}
				style:background
				bind:this={canvas} 
				on:mousedown={handleStart}	
				on:touchstart={handleStart}
				/>


<style>
	canvas{
		border: 2px solid grey;
	}
	
	.mouse-over-shield {
      position: fixed;
      top: 0px;
      left: 0px;
      height: 100%;
      width: 100%;
      background-color: rgba(255, 0, 0, 0);
      z-index: 10000;
      cursor: grabbing;
    }
</style>
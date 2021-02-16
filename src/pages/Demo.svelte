	<!-- this page displays a demo version of the rating box that doesn't store data -->

<script>
    import { createEventDispatcher } from 'svelte';
	import RatingBox from '../RatingBox.svelte';
	import CustomVideo from '../CustomVideo.svelte';
    import { experiment } from '../utils.js';
    
    const dispatch = createEventDispatcher();
    
	export let src;
    export let time = 0;
    export let ratingType;
	let pathway = `${experiment}/demo`;
	let paused = false;
    let rating = 50.0;
    
	function handlePause() {
		paused = true;
    };
    
	function handlePlay() {
		paused = false;	
	};

    function handleEnd() {
		dispatch('finished');
    };
    
    function handleBack() {
		dispatch('back');
	}; 
</script>

<style>
	p {
		font-weight: normal;
		padding: none;
	}

	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 1em;
		margin: 0 auto !important;
		min-width: 400px !important;
		max-width: 1000px !important;
	}

    .next {
    background-color: lightblue;
	}

    .back {
        background-color: lightcoral;
    }

    .key-box {
        align-items: center;
        text-align: center;
        margin: 0 auto !important;
    }

    h2 {
        text-align: center;
    }
</style>

<div>
	<h2>This is a demo. Please scroll down and Zoom in or out until you see the two boxes fully.</h2>
	<div class="container">
		<h3>Video: </h3>
		<p>Click on the video to start/pause (the video may take a few seconds to load, so please wait.) </p>
		<CustomVideo
			src={src}
			bind:time={time}
			on:pause={handlePause}
			on:play={handlePlay}
		></CustomVideo>
		<br>
		<h3>Rating Box: </h3>
		<p>	As you watch the video, please indicate whether the shapes are having a social or random (non-social) interaction at each timepoint by constantly pressing either the <strong> up arrow </strong> or the <strong>down arrow </strong> key.
<strong>Make sure you are updating your rating throughout the video especially whenever your assessment changes.</strong> <br> Hold either arrow key to accelerate in that direction, and release it to reset acceleration.</p>
		<RatingBox 
			pathway={pathway}
			rating={rating}
			bind:time={time}
			paused={paused}
			ratingType={ratingType}
            ratingBoxUse='demo page'>
		</RatingBox>
	</div>
	<div class="key-box">
		<button class="back" on:click={handleBack}>Back</button>
		<button class="next" on:click={handleEnd}>Next</button>
	</div>
</div>


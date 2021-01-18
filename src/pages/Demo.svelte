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
			on:finished={handleEnd}
		></CustomVideo>
		<h3>Rating Box: </h3>
		<p>	Press <strong> up </strong> arrow and <strong>down</strong> arrow keys continuously to move the ratings towards social or random (non-social) interactions. <br> Hold either arrow key to accelerate in that direction. <br>Release key to reset acceleration.</p>
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


<!-- this page displays the task, consisting of a video and rating box -->

<script>
    import { createEventDispatcher } from 'svelte';
	import RatingBox from '../RatingBox.svelte';
	import CustomVideo from '../CustomVideo.svelte';
    import { db, auth, serverTime, params } from '../utils.js';

    const dispatch = createEventDispatcher();

	export let src;
    export let time;
    export let pathway;
	export let ratingType;
	let paused = true;
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

</script>

<style>
	main {
		padding: 1em;
		margin: 0 auto;
		min-width: 400px !important;
		max-width: 1000px !important;
	}

	h2 {
		font-weight: normal;
		padding: none;
		width: 50%;
	}

	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

</style>


<main>
	<div class="container">
		<CustomVideo
			src={src}
			bind:time={time}
			on:pause={handlePause}
			on:play={handlePlay}
			on:finished={handleEnd}
		></CustomVideo>
		<RatingBox
			pathway={pathway}
			rating={rating}
			bind:time={time}
			bind:paused={paused}
			ratingType={ratingType}>
		</RatingBox>
		<h3 style="text-align:center">Click on the video to start, and <strong>continuously</strong> rate how <strong>social (UP arrow key)  </strong> or <strong> random (DOWN arrow key) </strong> you perceive the interactions between the shapes to be</h3>
    <h3 style="text-align:center">Non-responders on this page may have their HITs rejected</h3>
	</div>
</main>

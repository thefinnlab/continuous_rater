<!-- first instruction page -->
<script>
  // This is the Instructions page. It loops over the instructions array as a user reads and when click to the last page it notifies the main App.svelte component by dispatching a 'finished' event. When the last page of the instructions are reached the forward button turns into a "Take Quiz" button, but currently there is no quiz and it goes straight to the experiment
  import { createEventDispatcher } from 'svelte';
  export let ratingType;
  export let numOptions; //from App.svelte

  // Add/remove items here to create more instructions pages
  const ratingInstruct = 'The first part of this HIT requires you to provide continuous ratings on a moving bar while watching a short video clip involving moving shapes. The  ratings will indicate the degree to which you perceive a social interaction between the shapes (as opposed to random movement of the shapes) in the video clip at that given time point. <br><br>Following the video, you will be asked to answer a series of questions. <br><br>You should not redo this HIT. '
  // You may redo this HIT up to ' + numOptions + ' more times (you are provided with a different video each time). '
  const instructions = [
    ratingInstruct
  ];

  const dispatch = createEventDispatcher();
  let currentPage = 0;

    function handleEnd() {
		dispatch('finished');
    };

  const backward = () => {
    currentPage -= 1;
    currentPage = Math.max(currentPage, 0);
  };
  const forward = () => {
    // Check if we're increasing the value of currentPage beyond the number of instructions, if so tell App.svelte we're ready to move to the quiz
    if (currentPage + 1 === instructions.length) {
      dispatch('finished');
    } else {
      currentPage += 1;
      currentPage = Math.min(currentPage, instructions.length - 1);
    }
  };
</script>

<style>
  .container {
    margin: 0 auto !important; 
    max-width: 800px;
    text-align: center;
  }

  .text-box {
        text-align: left;
		padding: 2%;
		background-color: rgba(255, 255, 255, 0.4);
		border: 2px solid grey;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);   
	}


  
  .controls {
    min-width: 25%;
  }

  button {
    background-color: lightblue;
  } 
</style>
 

<div class="container">
  <h1>Instructions</h1>  
  <div class="text-box">
    <div class="content">
      {@html instructions[currentPage]}
    </div>
  </div>
  <br>
  <button class="button is-link controls" on:click={handleEnd}>
    {#if currentPage === instructions.length - 1}
      Go To Demo
    {:else}
      <span class="icon">
        Next
        <i class="fas fa-forward" />
      </span>
    {/if}
  </button>
</div>

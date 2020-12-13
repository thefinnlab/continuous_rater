<!-- this page handles the consent -->

<script>
    import { createEventDispatcher } from 'svelte';
    import { studyLocation, studyAim, studyTasks, email} from '../utils.js';
    const dispatch = createEventDispatcher();
    let consentRejected = false;

    function handleYes() {
        dispatch('finished');
    };

    function handleNo() {
        consentRejected = true;
        dispatch('return');
    };
</script>


<style>
    .container {
        margin: 0 auto !important;
        align-items: center;
        text-align: center;
        width: 50%;
    }
    .consent-box {
        text-align: left;
        padding: 2%;
        background-color: rgba(255, 255, 255, 0.4);
        border: 2px solid grey;
        border-radius: 2px;
        box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);   
    }
    .intro {
        font-style: italic;
    }
    .yes {
        background-color: lightblue;
    }
    .no {
        background-color: lightcoral;
    }
</style>

<div class="container">
    {#if consentRejected}
        <p>Thank you for your time. You may exit the page.</p>
    {:else}
		
		<h2>DARTMOUTH COLLEGE</h2>
		<h2>Naturalistic Cognition</h2>
		<h2>RESEARCH PROJECT INFORMATION SHEET</h2>
			
        <p class="intro">Please read the following material that explains this research study. 
            We want you to understand what you are being asked to do and what risks and benefits 
            --if any-- are associated with the study. Consent with this form will indicate that 
            you have been informed about the study and that you want to participate.</p>
		<p>(Please scroll down to read the full text.)</p>
		
		
		<div class="consent-box">
            
            <p>This research project is being conducted by researchers from the Department of Psychological & Brain Sciences at Dartmouth College, Hanover, NH, USA.  This study aims to understand how individuals respond to movies and stories, and how their responses relate to aspects of their personality, cognitive abilities, and other traits.</p>

            <p>Your participation is voluntary.  Participation involves watching videos, reading or listening to stories, playing simple video games and other computerized tasks, answering questions about yourself, and similar activities.  You may choose to not answer any or all questions.  You must be 18 or older to participate. You may immediately end your participation if any aspect of the research procedure makes you too uncomfortable to continue. If you end your participation prematurely due to discomfort or technical difficulties, please contact the researcher to make sure you receive your payment. You may contact the researcher Emily Finn at emily.s.finn@dartmouth.edu.</p>

            <p>The information collected will be anonymous and no identifying information will be stored with the data collected during the experiment. Identifying information will not be used in any presentation or paper written about this project and such presentations will represent the aggregation of data from groups of people.</p>
			
            <p>Because your responses will not be individually identified, we will not be providing you with personal feedback or referrals based on any of your answers. We will not be monitoring your answers to any items, including questions about suicidality and self-harm, either in real time or after your participation is complete. If you are in crisis, and need immediate support or intervention, call, or go the website of the <a href="http://www.suicidepreventionlifeline.org/"> National Suicide Prevention Lifeline</a> (1-800-273-8255). <strong>  Trained crisis workers are available to talk 24 hours a day, 7 days a week.</strong> </p>
			
			<p>For general information on mental health and to locate treatment services in your area, call the Substance Abuse and Mental Health Services Administration (SAMHSA) Treatment Referral Helpline at 1-800-662-HELP (4357). SAMHSA also has a <a href=
			"https://findtreatment.samhsa.gov/"> Behavioral Health Treatment Locator</a> on its website that can be searched by location.</p>
			
			<ad>
			Questions about this project may be directed to:<br>
			Emily Finn, Principal Investigator<br>
			HB 6207, Moore Hall, Hanover, NH 03755.<br>
			Email: emily.s.finn@dartmouth.edu<br>
			</ad>


        </div>

        <h2>We need your consent to proceed. Do you understand and consent to these terms?</h2>

        <button class="yes" on:click={handleYes}>I agree</button>
        <button class="no" on:click={handleNo}>No thanks, I do not want to do this HIT</button>
    {/if}
</div>
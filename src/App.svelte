<!-- THIS IS THE VERSION OF THE SVELTE RATING APP INTENDED FOR USE WITH MOTH VIDEOS -->

<!-- TODOs:
	examine weird build errors on JS console when using with MTurk (GET ERROR)
-->

<script>
    import { db, auth, serverTime, params, ratingTypes, dev,
            experiment, userGroup, labName, email} from './utils.js';
    
    import { onMount } from 'svelte';
	import Intro from './pages/Intro.svelte';
	import Botcheck from './pages/Botcheck.svelte';
	import Consent from './pages/Consent.svelte';
	import Instructions1 from './pages/Instructions1.svelte';
	import Instructions2 from './pages/Instructions2.svelte';
    import Demo from './pages/Demo.svelte';
	import Task from './pages/Task.svelte';
	import Debrief from './pages/Debrief.svelte';
    import Complete from './pages/Complete.svelte';
    import Loading from './components/Loading.svelte';
    import Header from './components/Header.svelte';
    import MTurkPreview from './pages/MTurkPreview.svelte';

	// path details
	const ratingsPath = `${experiment}/ratings`;
	const ratingsDoc = db.doc(ratingsPath);
	const subjectGroupPath = `${experiment}/subjects/${userGroup}`;
    const subjectGroupCollection = db.collection(subjectGroupPath);
    const stimuliPath = `${experiment}/stimuli`;
    const stimuliDoc = db.doc(stimuliPath);
	const demo_stimuliPath = `${experiment}/demo_stimuli`;
    const demo_stimuliDoc = db.doc(demo_stimuliPath);

	// declare and set other necessary variables
	let currVid;
	let currVidSrc;
	let demo_currVid;
	let demo_currVidSrc;
	let currRating;
	let subjectPath;
	let ratingDocPathway;
	let currentState;
    let consentStatus;
    let alreadyWatched = [];
    let moviesRemaining = [];
	let demo_moviesRemaining = [];
    let numOptions;
    let time = 0;
    let initExperiment = false;
    
    // use to validate build type in JS console
    console.log(dev);
    console.log('tester boiii');

    const resetTestWorker = async () => {
        // Change to the new state within Svelte
        if (params.workerId === 'test-worker') {
            currentState = 'consent';
            let subjectRef = subjectGroupCollection.e(params.workerId);
            subjectRef.get().then(function(doc) {
                try {
                    let currPath = `${ratingsPath}/${params.workerId}`;
                    db.collection(currPath).get().then(function(ratingList) {
                        // deletes all previous ratings
                        ratingList.forEach(function(doc) {
                            console.log('deleting: ', doc.id);
                            db.collection(currPath).doc(doc.id).delete()   
                        });      
                        // updates subject log
                        subjectRef.update({
                            startTime: serverTime,
                            consentStatus: 'incomplete'
                        });
                        console.log('reset test-worker');
                    });
                } catch (error) {
                    console.error(error);
                }
            });
        } else {
            console.log(`Reset user requested but workerId is ${params.workerId}`);
        }
    };
    
    
    
	
	// *****************************
	// main function
    // *****************************

	// Before we render anything see if we have a db entry for this subject based upon the URL parameters. If not 
	// create an entry with a new random stimulus order and put them into the instructions state. 
    // If we do, load their trial order and current experiment state

    if (params.assignmentId === 'ASSIGNMENT_ID_NOT_AVAILABLE') {
        currentState = 'mturk-preview';
    } else if (
        params.workerId &&
        params.assignmentId !== 'ASSIGNMENT_ID_NOT_AVAILABLE' &&
        params.hitId
    ) {
        initExperiment = true;
    } else {
        currentState = 'non-mturk';
    }
	
    onMount(async () => { // right when DOM is created
        if (initExperiment) {
            try {
                auth.onAuthStateChanged(async (user) => {
                    if (!user) { // if no user
                        try { // grab the worker and assignment ID and attempt login
                            await auth.signInWithEmailAndPassword(
                                `${params.workerId}@experiment.com`,
                                params.workerId
                            );
                            console.log('user found...signing in with credentials');
                            // then look for document
                        } catch (error) {
                            if (error.code === 'auth/user-not-found') {
                                console.log('no user found...creating new credentials');
                                // if login fails, create new user
                                await auth.createUserWithEmailAndPassword(
                                    `${params.workerId}@experiment.com`,
                                    params.workerId
                                );
                            } else {
                                console.log("not working");
                                console.error(error);
                            }
                        }
                    } else {
                        console.log('user authenticated...');
                        let currUser = auth.currentUser;
                        try { // if user already signed in, grab relevant document
                            let subjectRef = subjectGroupCollection.doc(params.workerId);
                            subjectPath = `${subjectGroupPath}/${params.workerId}`; // setting for use in HTML below
                            subjectRef.get().then(function(doc) {
                                if (doc.exists) { // load old document
                                    console.log('previous document found...loading state...');
                                    // updates most recent login time
                                    subjectRef.update({
                                        mostRecentTime: serverTime
                                    });
                                } else { // create a new document
                                    subjectGroupCollection.doc(params.workerId).set({name: 'unknown'});
                                    console.log('no previous documents found...creating new...');
                                    subjectPath = `${subjectGroupPath}/${params.workerId}`; // setting for use in HTML below
                                    subjectRef.set({
                                        workerId: params.workerId,
                                        assignmentId: params.assignmentId,
                                        hitId: params.hitId,
                                        userId: currUser.uid,
                                        startTime: serverTime,
                                        consentStatus: 'incomplete'
                                    });
                                }
                                // grab stimuli doc and add all movies to list
                                stimuliDoc.get().then(function(stimuliTable) {
                                    for (var field in stimuliTable.data()) {
                                        moviesRemaining.push(field);         
                                    }
                                    // check to see which movies subject has already viewed (if any)
                                    let currPath = `${ratingsPath}/${params.workerId}`;
                                    db.collection(currPath).get().then(function(ratingList) {
                                        // removes already completed movies from option set
                                        ratingList.forEach(function(doc) {
                                            moviesRemaining = removeItemOnce(moviesRemaining, doc.id.split("-")[0]);
                                        });
                                        // see how many movies are left
                                        numOptions = 1; //moviesRemaining.length;
                                        console.log('moviesRemaining: ', moviesRemaining);
                                        // if any movie-rating pairings left, load and start
                                        if (numOptions > 0) {
                                        // choose random movie and rating type
                                            let movieIndex = Math.floor(Math.random()*moviesRemaining.length);
                                            let ratingIndex = Math.floor(Math.random()*ratingTypes.length); // always 0 for us (since ratingTypes = ['social])
                                            console.log('ratingTypes.length:',ratingTypes.length)
											
											currVid = moviesRemaining[movieIndex];
											currRating = ratingTypes[ratingIndex]; // 'social' in our case
                                            let vidPlusRating = `${currVid}-${currRating}`;//videoname-social
                                            ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;
                                            // grab URL for video sourcing 
                                            currVidSrc = stimuliTable.data()[currVid];
											console.log('url:',currVidSrc)
                                            updateState('consent');
                                            
                                        } else {
                                            console.log("no options left!");
                                            updateState('complete');
                                        }
                                    });
                                });
								
								// DEMO PAGE - grab stimuli doc and add all movies to list
                                demo_stimuliDoc.get().then(function(demo_stimuliTable) {
                                    for (var field in demo_stimuliTable.data()) {
                                        demo_moviesRemaining.push(field);         
                                    }
                                    // check to see which movies subject has already viewed (if any)
                                    let currPath = `${ratingsPath}/${params.workerId}`;
                                    db.collection(currPath).get().then(function(ratingList) {
                                        // removes already completed movies from option set
                                        ratingList.forEach(function(doc) {
                                            demo_moviesRemaining = removeItemOnce(demo_moviesRemaining, doc.id.split("-")[0]);
                                        });
                                        // see how many movies are left
                                        numOptions = 1; //demo_moviesRemaining.length;
                                        console.log('demo_moviesRemaining: ', demo_moviesRemaining);
                                        // if any movie-rating pairings left, load and start
                                        if (numOptions > 0) {
                                        // choose random movie and rating type
                                            let demo_movieIndex = Math.floor(Math.random()*demo_moviesRemaining.length);
                                            //let ratingIndex = Math.floor(Math.random()*ratingTypes.length);
                                            demo_currVid = demo_moviesRemaining[demo_movieIndex];
											console.log('demo_currVid:',demo_currVid)
											//currRating = ratingTypes[ratingIndex]; 
                                            //let vidPlusRating = `${currVid}-${currRating}`; 
                                            //ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;
                                            // grab URL for video sourcing 
                                            demo_currVidSrc = demo_stimuliTable.data()[demo_currVid];
											console.log('demo url:',demo_currVidSrc)
                                            updateState('consent'); // REMOVE THIS?
                                            
                                        } else {
                                            console.log("no options left!");
                                            updateState('complete');
                                        }
                                    });
                                });
                            });	
                        } catch (error) {
                            console.error(error);
                        }
                    }
                });
            } catch (error) {
                console.error(error);
            }
        }
	});

	// *****************************
	// handler functions
	// *****************************

	// this function updates the current state of the user to 
	// dynamically render different parts of the experiment (i.e. instructions, quiz, etc)
  	const updateState = async (newState) => {
    	// Change to the new state within Svelte
		currentState = newState;
		try {
			await db.doc(`${experiment}/subjects/${userGroup}/${params.workerId}`).update({
				currentState
			});
			console.log('updated user state');
		} catch (error) {
			console.error(error);
		}
	};

	// registers user as a bot
	const failedBot = async () => {
		try {
			await db.doc(`${experiment}/subjects/${userGroup}/${params.workerId}`).update({
				botStatus: "bot"
			});
			console.log('user identified as bot');
		} catch (error) {
			console.error(error);
		}
	};

	// registers rejected consent form
	const failedConsent = async () => {
		try {
			await db.doc(`${experiment}/subjects/${userGroup}/${params.workerId}`).update({
				consentStatus: 'failed'
			});
			console.log('user rejected consent');
		} catch (error) {
			console.error(error);
		}
	};

	// registers accepted consent form
	const agreedConsent = async () => {
		try {
			await db.doc(`${experiment}/subjects/${userGroup}/${params.workerId}`).update({
				consentStatus: 'signed'
			});
			updateState('botcheck-instruct');
			console.log('user accepted consent');
		} catch (error) {
			console.error(error);
		}
	};

	// function used to remove previously watched videos from array
	function removeItemOnce(arr, value) {
  		var index = arr.indexOf(value);
  		if (index > -1) {
    		arr.splice(index, 1);
  		}
  		return arr;
	};
</script>


<style>
    .content {
        position: relative;
    }
    .header {
        left: 0;
    }
</style>


<div class="content">
    <div class="header">
        <Header on:resetTestWorker={resetTestWorker}></Header>
    </div>
	{#if !currentState}
        <Loading>Loading...</Loading>
    {:else if currentState === 'mturk-preview'}
        <MTurkPreview />
	{:else if currentState === 'intro'}
		<Intro on:finished={() => updateState('consent')}></Intro>
	{:else if currentState === 'consent'}
		<Consent on:finished={() => agreedConsent()} on:returned={() => failedConsent()}></Consent>
	{:else if currentState === 'botcheck-instruct'}
		<Botcheck on:finished={() => updateState('instructions1')} on:failed={() => failedBot()}></Botcheck>
	{:else if currentState === 'botcheck-task'}
		<Botcheck on:finished={() => updateState('task')} on:failed={() => failedBot()}></Botcheck>
	{:else if currentState === 'instructions1'}
		<Instructions1 ratingType={currRating} numOptions={numOptions} on:finished={() => updateState('demo')} />
	{:else if currentState === 'demo'}
		<Demo 
			src={demo_currVidSrc}
			time={time} 
			ratingType={currRating} 
			on:back={() => updateState('instructions1')} 
			on:finished={() => updateState('instructions2')} />
	{:else if currentState === 'instructions2'}
		<Instructions2 on:back={() => updateState('demo')} on:finished={() => updateState('task')} />
	{:else if currentState === 'task'}
		<Task 
			src={currVidSrc}
			ratingType={currRating}
			time={time} 
			pathway={ratingDocPathway} 
			on:finished={() => updateState('debrief')} 
		></Task>
	{:else if currentState === 'debrief'}
		<Debrief
			subPath={subjectPath}
			email={email}
			labName={labName}
			numOptions={numOptions}
		></Debrief>
	{:else if currentState === 'complete'}
		<Complete></Complete>
	{/if}  	 
</div>





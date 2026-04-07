<script lang="ts">
  import bigLogo from "./assets/images/logo-large.svg";
  import smallLogo from "./assets/images/logo-small.svg";
  import iconPersonalBest from "./assets/images/icon-personal-best.svg";
  import iconRestart from "./assets/images/icon-restart.svg";
  import TestComplete from "./lib/test-complete.svelte";
  import BaselineEstablished from "./lib/baseline-established.svelte";
  import HighScoreSmashed from "./lib/high-score-smashed.svelte";
  import StatisticsControl from "./lib/statistics.svelte";
  import texts from "./assets/data.json";

  type ResultState = "Complete" | "Baseline" | "HigherScore" | "None";

  let innerWidth = $state(0);
  let isMobile = $derived(innerWidth < 640);
  let isDesktop = $derived(innerWidth > 640);

  let passage = $state(true);
  let testComplete = $state(true);
  let resultState = $state("None" as ResultState);

  // Test variables
  let currentText = $state({
    id: "",
    text: "Some words are *italic*, some are **bold**.",
  });
  let selectedDifficulty: string = $state("Easy");
  let selectedMode: string = $state("Timed (60s)");
  let stopTest = $state(false);
  let finalWpm = $state(0);
  let finalAccuracy = $state(0);
  let finalCharacters = $state(0);
  let totalCharacters = $derived(currentText.text.length);
  let totalTime = $state(0);
  let startTime = $state(0);
  let isActive = $state(false);
  let errors = $state(0);
  let userInput: string = $state('');

  function loadNewText() {
    if (selectedDifficulty === "Easy") {
      currentText = texts.Easy[Math.floor(Math.random() * texts.Easy.length)];
    } else if (selectedDifficulty === "Medium") {
      currentText =
        texts.Medium[Math.floor(Math.random() * texts.Medium.length)];
    } else {
      currentText = texts.Hard[Math.floor(Math.random() * texts.Hard.length)];
    }
  }

  function startTest() {
    passage = !passage;
    loadNewText();
    isActive = true;
    startTime = Date.now();

    if (selectedMode === "Timed (60s)") {
      startTimer();
    } else {
      let timeLeft = 0;
      setInterval(() => {
        timeLeft++;
        if (userInput.length >= totalCharacters) {
          endTest();
        }
      }, 1000);
    }
  }

  function typingText(input: string) {
    userInput = input;
  }

  // Start the countdown timer
  function startTimer() {
    let timeLeft = 60;

    setInterval(() => {
      timeLeft--;
      if (timeLeft <= 0) {
        endTest();
      }
      if (userInput.length >= totalCharacters) {
        endTest();
      }
    }, 1000);
  }

  function endTest() {
    isActive = false;
    stopTest = true;
    updateStats();
    showResults();
  }

  function updateStats() {
    // const timeElapsed = (Date.now() - startTime) / 1000 / 60;
    // const grossWPM = currentIndex / 5 / timeElapsed;
    // const netWPM = Math.max(0, Math.round(grossWPM - errors / timeElapsed));
    // const accuracy =
    //   totalCharacters > 0
    //     ? Math.round(((totalCharacters - errors) / totalCharacters) * 100)
    //     : 100;
  }

  // Show results modal with final stats
  function showResults() {
    resultState = "Complete";
  }

  // Reset the typing test
  function resultStateChange(event: ResultState) {
    resultState = event;
  }

  function restart() {
    stopTest = false;
    finalWpm = 0;
    finalAccuracy = 0;
    finalCharacters = 0;
    totalCharacters = 0;
    testComplete = true;
    errors = 0;
    userInput = ''
    isActive = false
    totalTime = 0
     loadNewText()
  }
</script>

<svelte:window bind:innerWidth />
<div class="flex flex-col gap-8 p-6 w-full h-full">
  <!--Header-->
  <div class=" flex items-center justify-between w-full relative">
    {#if isMobile}
      <img src={smallLogo} alt="Logo" />
    {:else}
      <img src={bigLogo} alt="Logo" />
    {/if}

    <p class="flex gap-1">
      <img class="pl-1" src={iconPersonalBest} alt="Personal best" />
      {#if isDesktop}
        Personal best
      {:else}
        Best
      {/if}: <span class="text-white">92 WPM</span>
    </p>
  </div>

  <!--Statistics-->

  <div class="relative flex flex-col gap-2 flex-1">
    <div class="flex justify-between items-center gap-2 md:p-2 max-md:flex-col">
      <ul class="flex list-none gap-2 max-md:pl-2 max-md:pr-2">
        <li class="separator max-md:text-center">
          WPM: <span class="text-white value">{finalWpm}</span>
        </li>
        <li class="separator max-md:text-center">
          Accuracy: <span class="text-red-500 value">{finalAccuracy}%</span>
        </li>
        <li class="max-md:text-center">
          Time: <span class="text-yellow-400 value">{totalTime}</span>
        </li>
      </ul>

      <StatisticsControl bind:selectedMode bind:selectedDifficulty />
    </div>

    <div class="relative flex-1">
      <div class="flex flex-col gap-2 items-center h-full text-center">
        <textarea
          class="w-full h-full"
          placeholder={currentText.text}
          bind:value={userInput}
          disabled={stopTest}
          oninput={(e) => typingText(e.currentTarget.value)}
        ></textarea>
        {#if testComplete}
          <button
            class="button-secondary"
            aria-label="Beat This Score"
            onclick={restart}
          >
            Beat This Score
            <img src={iconRestart} alt="icon-restart" class="w-4 h-4" />
          </button>
        {/if}
      </div>
      <!-- Passage appear at the beginning-->
      {#if passage}
        <div
          class="absolute inset-0 backdrop-blur-sm flex flex-col items-center justify-center"
        >
          <button
            class="button-primary mb-4"
            aria-label="start typing test"
            onclick={startTest}
          >
            Start Typing Test
          </button>
          <p class="text-white">Or click the text and start typing</p>
        </div>
      {/if}
    </div>

    <!-- Results - positioned to cover entire statistics + textarea area -->
    {#if resultState === "Complete"}
      <div class="absolute inset-0">
        <TestComplete
          wpm={finalWpm}
          accuracy={finalAccuracy}
          typedCharacters={finalCharacters}
          {totalCharacters}
          resultStateChange={(e: any) => resultStateChange(e)}
        />
      </div>
    {/if}

    {#if resultState === "Baseline"}
      <div class="absolute inset-0">
        <BaselineEstablished
          wpm={finalWpm}
          accuracy={finalAccuracy}
          typedCharacters={finalCharacters}
          {totalCharacters}
          resultStateChange={(e: any) => resultStateChange(e)}
        />
      </div>
    {/if}

    {#if resultState === "HigherScore"}
      <div class="absolute inset-0">
        <HighScoreSmashed
          wpm={finalWpm}
          accuracy={finalAccuracy}
          typedCharacters={finalCharacters}
          {totalCharacters}
          resultStateChange={(e: any) => resultStateChange(e)}
        />
      </div>
    {/if}
  </div>

  <footer class="text-center">
    Challenge by <a href="https://www.frontendmentor.io?ref=challenge">
      Frontend Mentor
    </a>. Coded by <span>Anastasiia</span>.
  </footer>
</div>

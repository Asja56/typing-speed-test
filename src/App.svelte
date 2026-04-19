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
  import { onMount } from "svelte";

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
    text: "Some words",
  });
  let selectedDifficulty: string = $state("Easy");
  let selectedMode: string = $state("Timed (60s)");
  let stopTest = $state(false);
  let finalWpm = $state(0);
  let personalBestWpm = $state(0);
  let finalAccuracy = $state(0);
  let finalCharacters = $state(0);
  let totalCharacters = $derived(currentText.text.length);
  let currentIndex = $state(0);
  let totalTime = $state(60);
  let startTime = $state(0);
  let errors = $state(0);
  let wasWrong: boolean[] = [];
  let userInput: string = $state("");
  let textContentElement: HTMLElement;
  let timerId: ReturnType<typeof setInterval>;

  $effect(() => {
    selectedDifficulty;
    loadNewText();
  });

  onMount(() => {
    const stored = localStorage.getItem("personalBestWpm");
    personalBestWpm = stored ? Number(stored) : 0;
  });

  function loadNewText() {
    const map = {
      Easy: texts.Easy,
      Medium: texts.Medium,
      Hard: texts.Hard,
    };

    const list = map[selectedDifficulty];
    const randomItem = list[Math.floor(Math.random() * list.length)];

    currentText = { ...randomItem };
  }

  function clickOnStartTest() {
    passage = !passage;
    startTest();
  }

  function startTest() {
    stopTest = false;
    startTime = Date.now();

    textContentElement.innerHTML = "";
    currentText.text.split("").forEach((character) => {
      const charSpan = document.createElement("span");
      charSpan.dataset.original = character;
      charSpan.innerHTML = character;
      textContentElement.appendChild(charSpan);
      charSpan.classList.remove("text-green-500", "underline", "text-red-500");
    });
    startTimer();

    wasWrong = Array(totalCharacters).fill(false);
  }

  function typingText(input: string) {
    userInput = input;

    const arrayQuoteSpan = textContentElement.querySelectorAll("span");
    const arrayUserInput = input.split("");

    arrayQuoteSpan.forEach((charSpan, index) => {
      const originalChar = charSpan.dataset.original;
      const character = arrayUserInput[index];

      charSpan.classList.remove("text-red-500", "underline", "text-green-500");
      if (!character) {
        charSpan.innerHTML = originalChar!;
        return;
      }
      if (character === originalChar) {
        charSpan.innerHTML = originalChar!;
        charSpan.classList.add("text-green-500");
        wasWrong[index] = false;
      } else {
        charSpan.classList.add("underline", "text-red-500");
        charSpan.innerHTML = character;
        if (!wasWrong[index]) {
          errors++;
        }

        wasWrong[index] = true;
      }

      console.log("errors", errors);

      currentIndex = index;
      if (currentIndex >= totalCharacters) {
        endTest();
      }

      updateStats();
    });
  }

  // Start the countdown timer
  function startTimer() {
    if (timerId) clearInterval(timerId);

    let timeLeft = selectedMode === "Timed (60s)" ? 60 : 0;

    timerId = setInterval(() => {
      if (selectedMode === "Timed (60s)") {
        timeLeft--;
      } else {
        timeLeft++;
      }

      totalTime = timeLeft;

      if (timeLeft <= 0 && selectedMode === "Timed (60s)") {
        endTest();
      }

      if (userInput.length >= totalCharacters) {
        endTest();
      }
    }, 1000);
  }

  function endTest() {
    if (stopTest) return;
    stopTest = true;
    if (timerId) {
      clearInterval(timerId);
    }

    updateStats();
    showResults();
  }

  function updateStats() {
    const endTime = Date.now();
    const timeElapsed = (endTime - startTime) / 1000 / 60;
    const grossWPM = currentIndex / 5 / timeElapsed;
    const netWPM = Math.max(0, Math.round(grossWPM - errors / timeElapsed));

    const accuracy =
      totalCharacters > 0
        ? Math.round(((totalCharacters - errors) / totalCharacters) * 100)
        : 100;

    finalWpm = isFinite(netWPM) ? netWPM : 0;
    finalAccuracy = accuracy;
    finalCharacters = currentIndex;
  }

  // Show results modal with final stats
  function showResults() {
    if (personalBestWpm === 0) {
      resultState = "Baseline";
      personalBestWpm = finalWpm;
    } else if (finalWpm > personalBestWpm) {
      resultState = "HigherScore";
      personalBestWpm = finalWpm;
    } else {
      resultState = "Complete";
    }

    localStorage.setItem("personalBestWpm", `${personalBestWpm}`);
  }

  // Reset the typing test
  function resultStateChange(event: ResultState) {
    resultState = event;
    restart();
  }

  function restart() {
    if (timerId) {
      clearInterval(timerId);
    }
    finalWpm = 0;
    finalAccuracy = 0;
    finalCharacters = 0;
    testComplete = true;
    errors = 0;
    userInput = "";
    stopTest = false;
    totalTime = 60;
    resultState = "None";

    document.getElementsByTagName('textarea')[0].value = ''
    startTest();
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
      {/if}: <span class="text-white">{personalBestWpm} WPM</span>
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
          Time: <span class="text-yellow-400 value" id="timer"
            >00:{totalTime}</span
          >
        </li>
      </ul>

      <StatisticsControl bind:selectedMode bind:selectedDifficulty />
    </div>

    <div class="relative flex-1">
      <div class="flex flex-col gap-2 items-center h-full text-center">
        <div class="relative w-full h-full">
          <div class="absolute top-0 left-0">
            <div
              class="text-left"
              id="textContent"
              bind:this={textContentElement}
            ></div>
          </div>
          <textarea
            class="w-full h-full"
            disabled={stopTest}
            value=''
            oninput={(e) => typingText(e.currentTarget.value)}
          ></textarea>
        </div>
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
            onclick={clickOnStartTest}
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
          {errors}
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
          {errors}
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
          {errors}
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

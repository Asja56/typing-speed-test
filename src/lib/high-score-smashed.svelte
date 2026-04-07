<script lang="ts">
  import iconNewPB from "../assets/images/icon-new-pb.svg";
  import iconRestart from "../assets/images/icon-restart.svg";
  import confetti from "../assets/images/pattern-confetti.svg";
  type ResultState = "Complete" | "Baseline" | "HigherScore" | "None";

    let { resultStateChange, wpm, accuracy, typedCharacters, totalCharacters } =
    $props();
</script>

<div class="h-full flex flex-col bg-neutral-900">
  <!-- Content -->
  <div class="flex flex-col gap-4 items-center grow">
    <img src={iconNewPB} alt="High Score Smashed" />
    <h1 class="text-white font-semibold text-center">High Score Smashed!</h1>
    <p class="text-center">
      You're getting faster. That was incredible typing.
    </p>

    <ul
      class="grid md:grid-cols-3 list-none gap-3 mt-6 mb-6 max-md:grid-rows-3 max-md:w-full"
    >
      <li class="chip-result">
        WPM: <br />
        <span class="text-white font-semibold">{wpm}</span>
      </li>
      <li class="chip-result">
        Accuracy: <br />
        <span class="text-red-500 font-semibold">{accuracy}%</span>
      </li>
      <li class="chip-result">
        Characters:
        <div>
          <span class="text-green-500 font-semibold"> {typedCharacters} </span>/
          <span class="text-red-500 font-semibold"> {totalCharacters} </span>
        </div>
      </li>
    </ul>

    <button class="button-secondary" aria-label="Beat This Score"  onclick={() => resultStateChange("None" as ResultState)}>
      Beat This Score
      <img src={iconRestart} alt="icon-restart" class="w-4 h-4" />
    </button>
  </div>

  <img src={confetti} alt="star" />
</div>

<script lang="ts">
  import iconTestComplete from "../assets/images/icon-completed.svg";
  import iconRestart from "../assets/images/icon-restart.svg";
  import leftStar from "../assets/images/pattern-star-2.svg";
  import rightStar from "../assets/images/pattern-star-1.svg";
  type ResultState = "Complete" | "Baseline" | "HigherScore" | "None";

  let previousAccuracy;

  let { resultStateChange, wpm, accuracy, typedCharacters, errors } =
    $props();


</script>

<div class="flex gap-4 h-full bg-neutral-900 md:pl-16 md:pr-16 max-md:flex-col">
  <!-- Left star -->
  <img
    src={leftStar}
    alt="star"
    class="self-start md:mt-40 max-md:max-w-6 max-md:max-h-6"
  />

  <!-- Content -->
  <div class="flex flex-col gap-4 items-center grow">
    <img src={iconTestComplete} class="image-shadow" alt="Test Complete" />
    <h1 class="text-white font-semibold text-center">Test Complete!</h1>
    <p class="text-center">Solid run. Keep pushing to beat your high score.</p>

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
          <span class="text-red-500 font-semibold"> {errors} </span>
        </div>
      </li>
    </ul>

    <button class="button-secondary" aria-label="Go Again"  onclick={() => resultStateChange("None" as ResultState)}>
      Go Again
      <img src={iconRestart} alt="icon-restart" class="w-4 h-4" />
    </button>
  </div>

  <!-- Right star -->
  <img
    src={rightStar}
    alt="star"
    class="md:self-center max-md:self-end max-md:max-w-6 max-md:max-h-6"
  />
</div>

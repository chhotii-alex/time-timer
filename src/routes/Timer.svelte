
<script context="module">
  let current;

  const nameInputs = new Map();

</script>

<script>
    import { onMount, tick } from 'svelte';
    import { currentEditID } from "./stores.js";
import TimeWidget from "./TimeWidget.svelte";

  export let category;
  export let stop_time;
  export let saveAll;
  export let deleteItem;

  let nameInput;
  let isOpen = false;

  let seconds = totalTime(category.intervals);
  if (category.going) {
     current = category;
  }

  onMount( () => {
     const timerId = setInterval(update, 333);

     return () => {
         clearInterval(timerId);
    }
  });

  onMount(() => {
    nameInputs.set(category.id, nameInput);
    return () => nameInputs.delete(category.id);
  });

async function editableGrabFocus() {
    await tick();
    let input = nameInputs.get($currentEditID);
    if (input) {
      input.focus();
    }
  }

async function startEditing() {
    $currentEditID = category.id;
    await editableGrabFocus();
  }

function checkForEnterKey(event) {
    if (event.key === "Enter") {
      event.preventDefault();
      event.target.blur();
      endEditing();
    }
}


async function endEditing() {
    let input = nameInputs.get($currentEditID);
    if (input) {
      input.blur();
    }
    $currentEditID = -1;
  }

let lastIntervalSeconds;

function update() {
     seconds = totalTime(category.intervals);
     let lastIndex = category.intervals.length - 1;
     if (lastIndex >= 0) {
        if (category.intervals[lastIndex].length < 2) {
          lastIntervalSeconds = totalTime([category.intervals[lastIndex]]);
        }
     }
  }

function totalTime(intervals) {
    let seconds = 0;
    for (let interval of intervals) {
        let endTime = Date.now();
        if (interval.length > 1) {
            endTime = interval[1];
        }
        let millis = endTime - interval[0];
        seconds += millis / 1000;
    }
    return Math.round(seconds);
}

function formatTime(seconds) {
    let minutes = Math.floor(seconds / 60);
    seconds = seconds % 60;
    seconds = seconds.toString().padStart(2, '0');

    let hours = Math.floor(minutes / 60);
    minutes = minutes % 60;
    minutes = minutes.toString().padStart(2, '0');
    return `${hours}:${minutes}:${seconds}`;
}

function start_time(cat) {
  let now = Date.now();
  cat.intervals.push([now]);
  update();
  cat.going = true;
}

function toggle() {
  let newState = !(category.going);
  if (newState) {
    if (category !== current) {
       if (current && current.going) {
          stop_time(current);
       }
       current = category;
    }
  }
  if (newState) {
     start_time(category);
  }
  else {
     stop_time(category);
  }
  category = category;
  saveAll();
}

function deleteInterval(index) {
   category.intervals.splice(index, 1);
   category.intervals = category.intervals;
   saveAll();
}

function deleteMe() {
  deleteItem(category);
}

</script>

  <h3>
    {category.name}
  </h3>
  <input bind:value={category.name} bind:this={nameInput}
          on:keypress={checkForEnterKey}
        on:change={saveAll}
    class:nondisplay={$currentEditID != category.id}/>
    <p class="centered">
      {formatTime(seconds)}<br/>
    </p>
  <button on:click={() => toggle()}
    class="full-width big"
  >
    {#if category.going}
      Stop
    {:else}
      Go
    {/if}
   </button>

<details>
  <summary>Details and Editing...</summary>
    {#each category.intervals as interval, i (interval)}
       <p>
         <button on:click={() => deleteInterval(i)}>-</button>
         {#each interval as end}
           <TimeWidget bind:millis={end} />
         {/each}
         {#if interval.length == 2}
           {formatTime(totalTime([interval]))}
         {:else}
            {formatTime(lastIntervalSeconds)}
         {/if}
       </p>
    {/each}
    <button on:click={startEditing}>
      Edit Title
    </button>
    <textarea bind:value={category.detailText} on:input={saveAll} cols="25" rows="5" ></textarea>
    <details bind:open={isOpen} >
     <summary>Delete...</summary>
     Delete timer?
     <button on:click={ () => deleteMe() } >
       Confirm
     </button>
  </details>
</details>

<hr/>

<style>
 h3 {
   text-align: center;
 }
 .centered {
   text-align: center;
 }
 .full-width {
   width: 90%;
 }
 .big {
   height: 60px;
   margin: 5%;
 } 
 .nondisplay {
    display: none;
  }
  button {
    color: inherit;
    background-color: inherit;
    border: 0.5rem outset;
    font: inherit;
  }
  button:active {
    border: 0.5rem inset;
  }
</style>

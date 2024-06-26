
<script>

/*
TODO:
* try hosting on panix
* test on TV-- legacy?

*/

import Timer from "./Timer.svelte";
import TimeWidget from "./TimeWidget.svelte";

const isBrowser = typeof window !== "undefined";

function categoriesFromStorage() {
  if (isBrowser) {
     if (localStorage.getItem("timers")) {
         let newCat = JSON.parse(localStorage.getItem("timers"));
         return newCat;
     }
  }
  return [];
}

let catagories = categoriesFromStorage();

function nextHigherId() {
   let id = 1;
   for (let cat of catagories) {
      if (cat.id) {
         if (cat.id > id) {
            id = cat.id;
         }
      }
   }
   return id + 1;
}

function newCategory() {
   let newCat = {
      id: nextHigherId(),
      name: '',
      going: false,
      intervals: [],
      detailText: '',
   }
   return newCat;
}

function addNew() {
   catagories.push(newCategory());
   catagories = catagories;
   saveAll();
}

function stop_time(cat) {
  let now = Date.now();
    let lastIndex = cat.intervals.length - 1;
    cat.intervals[lastIndex].push(now);
    cat.going = false;

    catagories = catagories;
}
let isOpen = false;

function clearAll() {
  for (let cat of catagories) {
     cat.going = false;
     cat.intervals = [];
  }
  
    catagories = catagories;
    saveAll();
   isOpen = false;
}

function deleteItem(cat) {
         let index = catagories.indexOf(cat);
         if (index != -1) {
             catagories.splice(index, 1);
         }
         catagories = catagories;
         saveAll();
}

function saveAll() {
   if (isBrowser) {
      localStorage.setItem("timers", JSON.stringify(catagories));
   }
}

</script>

<div class="main">

{#each catagories as category}
  <Timer {category} {stop_time} {saveAll} {deleteItem} />
{/each}


<details bind:open={isOpen} >
   <summary>Clear All...</summary>
   Clear all accumulated time?
   <button on:click={ () => clearAll() } >
      Confirm
   </button>
</details>

<details>
  <summary>More...</summary>
  <button on:click={ () => addNew() } >
    Add New Timer
  </button>
</details>

</div>

<style>
.main {
   padding: 2%;
      background-color: PaleGoldenRod;
      color: DarkSlateBlue;
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

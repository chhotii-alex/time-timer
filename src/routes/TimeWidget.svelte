<script>

import { tick } from 'svelte';
export let millis;
let dateStr;
let timeStr;

function zfill(num) {
  return `${num}`.padStart(2, '0');
}

function isValid(date) {
   console.log("Value of date:", date.valueOf());
   return !isNaN(date.valueOf());
}

async function setDateString(ms) {
   console.log("Setting date string from milliseconds:", ms);
   let d = new Date(ms);
   let str;
   if (isValid(d)) {
       str = `${d.getFullYear()}-${zfill(d.getMonth()+1)}-${zfill(d.getDate())}`;
   }
   else {
     str = "";
   }
   if (str != dateStr) {
      console.log("Setting datestr.");
      console.log(str)
      await tick();
      dateStr = str;
   }
}

async function setTimeString(ms) {
   let d = new Date(ms);
   let str;
   if (isValid(d)) {
     str   = `${zfill(d.getHours())}:${zfill(d.getMinutes())}:${zfill(d.getSeconds())}`;
   }
   else {
     str = '';
   }
  if (str != timeStr) {
     console.log("Setting timestr.")
     console.log(str)
     await tick();
     timeStr = str;
  }
}

$: setDateString(millis);
$: setTimeString(millis);

function isClose(ms1, ms2) {
   return (Math.abs(ms1 - ms2) < 1000);
}

async function setDatetimeFromStrings(dateString, timeString) {
  let update = false;
  let ms = Number.NaN;
  let str = dateString + " " + timeString;
  let d = new Date(str);
  if (isValid(d)) {
     console.log("Date object:", d);
     ms = d.valueOf();
     if (!isClose(ms, millis)) {
        update = true;
     }
  }
  if (update) {
     console.log("Updating when from timewidget.")
     console.log(str)
      await tick();
     millis = ms;
  }
}

$: setDatetimeFromStrings(dateStr, timeStr);

</script>

<input type="date" bind:value={dateStr} />
<input type="time" bind:value={timeStr} step="1"/>

<style>


</style>
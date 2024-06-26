<script>

import { tick } from 'svelte';
export let millis;
let dateStr;
let timeStr;

function zfill(num) {
  return `${num}`.padStart(2, '0');
}

function isValid(date) {
   return !isNaN(date.valueOf());
}

async function setDateString(ms) {
   let d = new Date(ms);
   let str;
   if (isValid(d)) {
       str = `${d.getFullYear()}-${zfill(d.getMonth()+1)}-${zfill(d.getDate())}`;
   }
   else {
     str = "";
   }
   if (str != dateStr) {
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
     ms = d.valueOf();
     if (!isClose(ms, millis)) {
        update = true;
     }
  }
  if (update) {
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
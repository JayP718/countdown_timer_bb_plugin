<script>
  import { getContext } from "svelte";
  import { onDestroy } from "svelte";

  export let field;
  export let fieldState=0;
  export let fieldApi;
  export let shape;
  export let size;
  export let complete_text;
  export let color;
  
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const { styleable } = getContext("sdk");
  const formApi = formContext?.formApi;
  
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "datetime",
    0,
    false,
    null,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  setInterval(() => distance = distance-1000, 1000)

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  $: distance = Date.parse(fieldState.value)-new Date().getTime();

  $: days = Math.floor(distance/86400000);
  $: hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  $: minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
  $: seconds = Math.floor((distance % (1000 * 60)) / 1000);
</script>

<div use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else if !fieldState?.value}
  <div class="placeholder">Select a valid datetime field.</div>
  {:else}
    <button class="button" style="border-radius: {shape};  font-size: {size};  background-color: {color};">
      {#if distance/1000<=0} {complete_text} {/if}
      {#if days>0} {days} days {/if} 
      {#if hours>0} {hours} hrs {/if} 
      {#if minutes>0} {minutes} mins {/if} 
      {#if seconds>0} {seconds} secs {/if}
    </button>
  {/if}
</div>

<style>
  .button {
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
   
  }
</style>

<!--
  @component
  Implement the preview for the Select widget.
  @see https://decapcms.org/docs/widgets/#select
-->
<script>
  export let locale = '';
  // svelte-ignore unused-export-let
  export let keyPath = '';
  export let fieldConfig = {};
  export let currentValue = undefined;

  $: ({ multiple } = fieldConfig);
  $: listFormatter = new Intl.ListFormat(locale, { style: 'narrow', type: 'conjunction' });
</script>

{#if multiple && Array.isArray(currentValue) && currentValue.length}
  <p>{listFormatter.format([...currentValue])}</p>
{:else if typeof currentValue === 'string' && currentValue.trim()}
  <p>{currentValue}</p>
{/if}

<!--
  @component
  Implement the editor for the Relation widget.
  @see https://decapcms.org/docs/widgets/#relation
  @todo Support search fields.
-->
<script>
  import { getOptions } from '$lib/components/contents/details/widgets/relation/helper';
  import SelectEditor from '$lib/components/contents/details/widgets/select/select-editor.svelte';
  import { getEntries, getFile } from '$lib/services/contents';

  export let locale = '';
  export let keyPath = '';
  /** @type {RelationField} */
  export let fieldConfig;
  export let currentValue = undefined;

  $: ({
    // Widget-specific options
    collection: collectionName,
    file: fileName,
    // search_fields: searchFields,
  } = fieldConfig);

  $: refEntries = fileName ? [getFile(collectionName, fileName)] : getEntries(collectionName);
  $: options = getOptions(locale, fieldConfig, refEntries);
</script>

<SelectEditor {locale} {keyPath} {fieldConfig} bind:currentValue {options} />

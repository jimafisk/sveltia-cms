<script>
  import { Group, Icon, Listbox, Option } from '@sveltia/ui';
  import { _ } from 'svelte-i18n';
  import { siteConfig } from '$lib/services/config';
  import { selectedCollection } from '$lib/services/contents';
  import { goto } from '$lib/services/navigation';

  $: collections = $siteConfig.collections.filter(({ hide }) => !hide);
</script>

<Group class="primary-sidebar">
  <section>
    <h2>{$_('collections')}</h2>
    <Listbox>
      {#each collections as { name, label, icon, hide = false } (name)}
        {#if !hide}
          <Option
            label={label || name}
            selected={$selectedCollection.name === name}
            on:click={() => {
              goto(`/collections/${name}`);
            }}
          >
            <Icon slot="start-icon" name={icon || 'edit_note'} />
          </Option>
        {/if}
      {/each}
    </Listbox>
  </section>
</Group>

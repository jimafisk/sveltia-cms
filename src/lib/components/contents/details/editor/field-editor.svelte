<script>
  import { Icon, Menu, MenuButton, MenuItem, Separator } from '@sveltia/ui';
  import { marked } from 'marked';
  import { _ } from 'svelte-i18n';
  import CopyMenuItem from '$lib/components/contents/details/editor/copy-menu-item.svelte';
  import BooleanEditor from '$lib/components/contents/details/widgets/boolean/boolean-editor.svelte';
  import ColorEditor from '$lib/components/contents/details/widgets/color/color-editor.svelte';
  import DateTimeEditor from '$lib/components/contents/details/widgets/date-time/date-time-editor.svelte';
  import FileEditor from '$lib/components/contents/details/widgets/file/file-editor.svelte';
  import ListEditor from '$lib/components/contents/details/widgets/list/list-editor.svelte';
  import MarkdownEditor from '$lib/components/contents/details/widgets/markdown/markdown-editor.svelte';
  import NumberEditor from '$lib/components/contents/details/widgets/number/number-editor.svelte';
  import ObjectEditor from '$lib/components/contents/details/widgets/object/object-editor.svelte';
  import RelationEditor from '$lib/components/contents/details/widgets/relation/relation-editor.svelte';
  import SelectEditor from '$lib/components/contents/details/widgets/select/select-editor.svelte';
  import StringEditor from '$lib/components/contents/details/widgets/string/string-editor.svelte';
  import TextEditor from '$lib/components/contents/details/widgets/text/text-editor.svelte';
  import { entryDraft, revertChanges } from '$lib/services/contents/editor';
  import { escapeRegExp } from '$lib/services/utils/strings';

  export let locale = '';
  export let keyPath = '';
  export let fieldConfig = {};

  const widgets = {
    boolean: BooleanEditor,
    color: ColorEditor,
    date: DateTimeEditor, // alias
    datetime: DateTimeEditor,
    file: FileEditor,
    image: FileEditor, // alias
    list: ListEditor,
    markdown: MarkdownEditor,
    number: NumberEditor,
    object: ObjectEditor,
    relation: RelationEditor,
    select: SelectEditor,
    string: StringEditor,
    text: TextEditor,
  };

  // @todo Save & restore draft from local storage.

  $: ({
    label = '',
    hint = '',
    widget = 'string',
    i18n = false,
    pattern = undefined,
    multiple = false,
  } = fieldConfig);
  $: ({ hasLocales, locales, defaultLocale = 'default' } = $entryDraft.collection._i18n);
  $: otherLocales = hasLocales ? locales.filter((l) => l !== locale) : [];
  $: canTranslate = hasLocales && (i18n === true || i18n === 'translate');
  $: canDuplicate = hasLocales && i18n === 'duplicate';
  $: keyPathRegex = new RegExp(`^${escapeRegExp(keyPath)}\\.\\d+$`);
  $: isList = multiple || widget === 'list';

  // Multiple values are flattened in the value map object
  $: currentValue = isList
    ? Object.entries($entryDraft.currentValues[locale])
        .filter(([_keyPath]) => _keyPath.match(keyPathRegex))
        .map(([, val]) => val)
        .filter((val) => val !== undefined)
    : $entryDraft.currentValues[locale][keyPath];
  $: originalValue = isList
    ? Object.entries($entryDraft.originalValues[locale])
        .filter(([_keyPath]) => _keyPath.match(keyPathRegex))
        .map(([, val]) => val)
        .filter((val) => val !== undefined)
    : $entryDraft.originalValues[locale][keyPath];
  $: validity = $entryDraft.validities[locale][keyPath];
</script>

{#if widget !== 'hidden' && (locale === defaultLocale || canTranslate || canDuplicate)}
  {@const canCopy = canTranslate && otherLocales.length}
  {@const canRevert = !(canDuplicate && locale !== defaultLocale)}
  <section data-widget={widget} data-key-path={keyPath}>
    <header>
      <h4>{label}</h4>
      {#if canCopy || canRevert}
        <MenuButton class="ghost iconic small" popupPosition="bottom-right">
          <Icon slot="start-icon" name="more_vert" label={$_('show_menu')} />
          <Menu slot="popup">
            {#if canCopy}
              {#if ['markdown', 'string', 'text', 'list', 'object'].includes(widget)}
                <CopyMenuItem {keyPath} {locale} translate={true} />
                {#if otherLocales.length > 1}
                  <Separator />
                {/if}
              {/if}
              <CopyMenuItem {keyPath} {locale} />
            {/if}
            {#if canCopy && canRevert}
              <Separator />
            {/if}
            {#if canRevert}
              <MenuItem
                label={$_('revert_changes')}
                disabled={currentValue === originalValue}
                on:click={() => {
                  revertChanges(locale, keyPath);
                }}
              />
            {/if}
          </Menu>
        </MenuButton>
      {/if}
    </header>
    {#if validity?.valid === false}
      <div class="validation">
        {#if validity.valueMissing}
          <div>
            <Icon name="error" label={$_('error')} />
            {$_('validation.value_missing')}
          </div>
        {/if}
        {#if validity.rangeUnderflow}
          <div>
            <Icon name="error" label={$_('error')} />
            {$_('validation.range_underflow', { values: { min: fieldConfig.min } })}
          </div>
        {/if}
        {#if validity.rangeOverflow}
          <div>
            <Icon name="error" label={$_('error')} />
            {$_('validation.range_overflow', { values: { max: fieldConfig.max } })}
          </div>
        {/if}
        {#if validity.patternMismatch}
          <div>
            <Icon name="error" label={$_('error')} />
            {pattern[1]}
          </div>
        {/if}
      </div>
    {/if}
    <div>
      {#if !(widget in widgets)}
        <div>{$_('unsupported_widget_x', { values: { name: widget } })}</div>
      {:else if isList}
        <svelte:component this={widgets[widget]} {keyPath} {locale} {fieldConfig} {currentValue} />
      {:else}
        <svelte:component
          this={widgets[widget]}
          {keyPath}
          {locale}
          {fieldConfig}
          bind:currentValue={$entryDraft.currentValues[locale][keyPath]}
        />
      {/if}
    </div>
    {#if hint}
      <div class="hint">{@html marked.parse(hint)}</div>
    {/if}
  </section>
{/if}

<style lang="scss">
  section {
    padding: 16px;

    &[data-widget='object'] {
      & > header + div {
        border-width: 2px;
        border-color: var(--secondary-border-color);
        border-radius: var(--control--medium--border-radius);
      }
    }

    &:not(:last-child) {
      border-width: 0 0 1px;
      border-color: var(--primary-border-color);
    }
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 0 0 8px;

    h4 {
      font-size: var(--font-size--small);
      font-weight: 600;
      color: var(--secondary-foreground-color);
    }

    & + div {
      :global(input[type='color']),
      :global(input[type='date']),
      :global(input[type='datetime-local']),
      :global(input[type='time']),
      :global(input[type='number']) {
        outline: 0;
        border: 0;
        color: inherit;
      }

      :global(input[type='text']),
      :global(textarea) {
        width: 100%;
      }

      :global(input[type='color']),
      :global(input[type='number']) {
        background-color: var(--input-background-color);
        border-width: 1px;
        border-color: var(--primary-border-color);
        border-radius: var(--control--medium--border-radius);
      }

      :global(input[type='file']),
      :global(input[type='checkbox']),
      & > :global(div) {
        color: inherit;
      }

      :global(input[type='date']),
      :global(input[type='datetime-local']),
      :global(input[type='time']) {
        width: auto;
        text-transform: uppercase;
        background-color: transparent;
      }

      :global(:root[data-theme='dark']) & {
        :global(input[type='date']::-webkit-calendar-picker-indicator),
        :global(input[type='datetime-local']::-webkit-calendar-picker-indicator),
        :global(input[type='time']::-webkit-calendar-picker-indicator) {
          filter: invert(1);
        }
      }

      :global(span) {
        display: block;
      }
    }
  }

  .validation {
    color: var(--danger-foreground-color);

    div {
      display: flex;
      gap: 4px;
      margin: 8px 0;

      :global(.icon) {
        flex: none;
        font-size: 20px; // !hardcoded
      }
    }
  }

  .hint {
    margin: 8px 0 0;
    font-size: var(--font-size--small);
    opacity: 0.75;

    :global(p) {
      margin: 0;
    }
  }
</style>

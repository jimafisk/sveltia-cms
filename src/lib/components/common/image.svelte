<script>
  /** @type {('lazy' | 'eager')} */
  export let loading = 'lazy';
  export let src = '';
  export let alt = '';
  export let checkerboard = false;
  export let cover = false;
</script>

<img class:checkerboard class:cover {loading} {src} {alt} />

<style lang="scss">
  img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    background-color: var(--tertiary-background-color);

    &.cover {
      object-fit: cover;
    }

    &.checkerboard {
      // prettier-ignore
      background-image:
        linear-gradient(45deg, #eee 25%, transparent 25%, transparent 75%, #eee 75%),
        linear-gradient(45deg, #eee 25%, #fff 25%, #fff 75%, #eee 75%);
      background-size: 16px 16px;
      background-position: 0 0, 8px 8px;
    }
  }
</style>

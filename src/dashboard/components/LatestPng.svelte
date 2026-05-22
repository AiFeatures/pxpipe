<script lang="ts">
  // Image viewer. When selectedImageId is null, follows the latest render
  // via cache-busting on preview_meta. When pinned to an id, fetches that
  // specific image from the ring; degrades gracefully if the id has been
  // evicted.

  import { recent, selectedImageId } from '../stores/index.js';

  $: hasPreview = $recent.data?.has_preview === true;
  $: meta = $recent.data?.preview_meta ?? '';
  $: imageIds = $recent.data?.image_ids ?? [];
  $: pinned = $selectedImageId;

  // when pinned, check if the id is still resident in the ring
  $: pinnedEvicted = pinned != null && !imageIds.includes(pinned);

  // url for the <img> element
  $: imgUrl = pinned != null
    ? `/proxy-latest-png?id=${pinned}`
    : (hasPreview ? '/proxy-latest-png?t=' + encodeURIComponent(meta) : '');

  // caption text
  $: caption = pinned != null
    ? `image #${pinned}`
    : (meta ? meta + ' - showing top-left at native resolution' : '');
</script>

<div class="wrap">
  {#if pinned != null}
    <div class="pin-bar">
      <button class="back-btn" on:click={() => selectedImageId.set(null)}>← latest</button>
    </div>
    {#if pinnedEvicted}
      <div class="evicted">(image #{pinned} no longer in buffer)</div>
    {:else}
      <div class="preview-crop">
        <img src={imgUrl} alt="image #{pinned}" />
      </div>
    {/if}
  {:else if hasPreview}
    <div class="preview-crop">
      <img src={imgUrl} alt="latest rendered" />
    </div>
  {:else}
    <div class="sub">(none yet)</div>
  {/if}
</div>
<div class="small">{caption}</div>

<style>
  .wrap {
    margin-top: 0;
  }
  /* Crop is done client-side via CSS (object-position + overflow:hidden).
     The legacy dashboard pulled a separately-cropped PNG which doubled
     image traffic. The full 1466×1568 image lives on disk; we just show
     the top-left corner at native res. */
  .preview-crop {
    width: 100%;
    height: 400px;
    overflow: hidden;
    background: #fff;
    border: 1px solid #30363d;
    border-radius: 4px;
    padding: 4px;
    box-sizing: border-box;
  }
  .preview-crop img {
    display: block;
    width: auto;
    height: auto;
    max-width: none;
    image-rendering: pixelated;
  }
  .sub {
    color: #6e7681;
    font-size: 12px;
  }
  .small {
    font-size: 11px;
    color: #6e7681;
    margin-top: 8px;
  }
  .pin-bar {
    margin-bottom: 8px;
  }
  .back-btn {
    font-size: 11px;
    background: #21262d;
    color: #58a6ff;
    border: 1px solid #30363d;
    border-radius: 4px;
    padding: 2px 8px;
    cursor: pointer;
  }
  .back-btn:hover {
    background: #30363d;
  }
  .evicted {
    font-size: 11px;
    color: #6e7681;
    padding: 12px 0;
  }
</style>

<script>
    import Icon from "./MaterialIcon.svelte";
  
    export let selected = []

    let complete = false;
  
    async function handleClick() {

      const url = selected ? window.location+'#'+selected.map(d=>d[0]).join(','): window.location;
      const text = "Explore how the average price of items is changing";
      const title = `ONS Shopping Price Comparison Tool`;
  
      try {
        if (navigator.canShare) {
          await navigator.share({ text, url, title });
        } else {
          await navigator.clipboard.writeText(url);
          complete = true;
        }
      } catch (e) {
        console.error(e);
      }
    }

  </script>
  
  <button class="flex items-center gap-2 custom-ring -ml-0.5" on:click={handleClick} class:hyperlink={!complete}>
    <div class="text-2xl text-ons-black">
      <Icon kind="share" />
    </div>
    {#if complete}
      <slot name="complete">Link copied!</slot>
    {:else}
      <slot>Share this basket</slot>
    {/if}
  </button>
  
  <style>
    .text-2xl{
        font-size: 1.5rem;
        line-height: 2rem;
    }

    .gap-2{
        gap: .5rem;
    }

    .items-center{
        align-items: center;
    }

    .flex{
        display: flex;
    }

    .text-ons-black{
        --tw-text-opacity: 1;
        color: rgb(34 34 34/var(--tw-text-opacity));
    }

    button{
        -webkit-appearance: button;
        appearance: button;
        background-color: transparent;
        background-image: none;
        border:none;
        font-size: 16px;

    }
    .hyperlink{
        --tw-text-opacity: 1;
        color: rgb(32 96 149/var(--tw-text-opacity));
        text-decoration-line: underline;
        text-decoration-thickness: 1px;
        text-underline-offset: 3px;
    }


    .hyperlink:hover{
        text-decoration-thickness: 3px;
    }
  </style>
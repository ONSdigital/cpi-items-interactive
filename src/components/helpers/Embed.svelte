<script>
    import Icon from "./MaterialIcon.svelte";
    import { fade } from "svelte/transition";
  
    let dialog;
    let button;
    let copied;

    $: open = false;  
  </script>
  
  <div class="">
    <button
      class="flex items-center gap-2 custom-ring hyperlink "
      on:click={() => {
        open = true;
        dialog.showModal();
      }}
    >
      <div class="text-2xl text-ons-black">
        <Icon kind="code" />
      </div>
      <div>Embed this tool</div>
    </button>
  </div>
  
  <dialog bind:this={dialog} on:close={() => (open = false)} class="m-auto max-w-5xl p-6">
     <h2>Embed this tool</h2>

    <button class="custom-ring" title="Close dialog" value="cancel" on:click={()=>dialog.close()}><Icon kind="close" /></button>
    <div class="px-4 border-y-8 border-ons-grey-5 bg-ons-grey-5 text-sm font-mono break-all h-24 overflow-y-scroll">
        EMBED CODE HERE
      </div>
      <div class="flex flex-col gap-3 pt-1">
        <button
          class="flex items-center justify-center gap-2 custom-ring bg-ons-leaf-green hover:bg-[#0d753c] shadow shadow-[#073d20] text-onswhite font-semibold rounded py-3 px-4 whitespace-nowrap"
          style="text-rendering: optimizeLegibility"
          bind:this={button}
        >
          <div>Copy to clipboard</div>
          <div class="text-2xl">
            <Icon kind="contentCopy" />
          </div>
        </button>
        {#if copied}
          <div class="text-center" out:fade={{ duration: 1000 }}>Copied!</div>
        {/if}
      </div> 
  </dialog>
  

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
<script>
    import Icon from "./MaterialIcon.svelte";

    export let dataDownloadUrl="https://github.com/onsdigital/cpi-items-actions/raw/main/datadownload.xlsx";
  </script>
  
  <div class="">
    <a
        href={dataDownloadUrl}
        class="flex items-center gap-2 custom-ring hyperlink -ml-0.5"
        data-gtm-type="data-download"
      >
        <div class="text-2xl text-ons-black">
          <Icon kind="fileDownload" />
        </div>
        <div>Download all data</div>
      </a>
  </div>
  

  <style>
    div{
      font-size: 16px;
    }

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
<script>
    export let options = ["Browse", "Search"];
    export let selected = options[0];
    let button = 'Browse';

    $: selected=selected
</script>

<div class="grid">
    {#each options as option, i}
        <div class="grid-cell">
            <label for={"button" + i} class:selected="{button === option}" on:click="{()=> button = option}" on:keydown="{()=> button = option}">
                <div class='button-label'><span>{option}</span></div>            
                <input bind:group={selected} type="radio" class="visuallyhidden" id={"button" + i} value={option} name="button"/></label>
        </div>
    {/each}
</div>

<style>
    span{
        font-weight: 400;
        margin:20px;
    }

    .grid {
        display: flex;
        /* flex-wrap: wrap; */
        list-style: none;
        /* margin: 5px 0 0 0; */
        margin-bottom: 27px;
        gap:5px;
    }

    .grid-cell {
        display: block;
        border: solid 2px #206095;
        /* flex:1 0 auto; */
        background-color: white;
    }


    .visuallyhidden {
        position: absolute;
        width: 1px;
        height: 1px;
        margin: -1px;
        padding: 0;
        overflow: hidden;
        clip: rect(0, 0, 0, 0);
        border: 0;
    }

    .grid-cell label {
        font-size: 16px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        /* margin-right: -1px; */
        color: #206095;
        height:40px;
    }

    label.selected{
        color: white;
        background: #206095;
    }


    label:not(.selected):hover {
        background-color: rgba(31, 95, 147, 0.2);
        cursor: pointer;
    }

    .grid-cell:focus-within{
        position: relative;
        box-shadow: 0 0 0px 3pt orange;
        outline: 2px dotted transparent;
        z-index: 10;
    }

    .button-label{
        text-align: center;
    }
</style>

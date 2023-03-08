<script>
	import { csv } from "d3-fetch";
	import { onMount } from "svelte";
	import { autoType } from "d3-dsv";
    import Select from "svelte-select";
    import {format} from 'd3-format';
    import {timeParse,timeFormat} from 'd3-time-format';
	
	let chained;
	let items;
    let avgprice;
    let monthlygrowth;
    let annualgrowth;
    let value = [];
    let sorted
    const groupBy = (item)=> item.CATEGORY
	
	
	onMount(async () => {
		(chained = await csv(
			"https://gist.githubusercontent.com/henryjameslau/8e98fc34b7aac3f7f98251d0a856e129/raw/f9c696905c74700a7a0bf2649cd0f4675c18399d/chained.csv",
			autoType
		)),
		(items = await csv(
			"https://gist.githubusercontent.com/henryjameslau/24fd5bf8000c2f04a2140ced6684da61/raw/3bd04b7516fca33e5128542416bac744bd4327ab/metadata.csv",
			autoType
		)),
        (avgprice = await csv(
            "https://gist.githubusercontent.com/henryjameslau/3ea034526695d7987863c4d6076449d3/raw/0a8c8145e0085269b18de7dd725666c69035797d/avgprice.csv",
            autoType
        )),
        (monthlygrowth = await csv(
            "https://gist.githubusercontent.com/henryjameslau/7b3e6c953c5efa15869d1f1dfadb4294/raw/d5b96cb7ff890a4d0edbd9bd0592d7a0c63e21c8/monthlygrowth.csv",
            autoType
        )),
        (annualgrowth = await csv(
            "https://gist.githubusercontent.com/henryjameslau/76f038ec284f08d65a3b224986ef57f3/raw/96ac1ccbbf88de2733aac4f1184a57ee78e3a8c5/annualgrowth.csv",
            autoType
        ));
		sorted = items.sort((a,b) => (a.CATEGORY.localeCompare(b.CATEGORY) || a.ITEM_DESC.localeCompare(b.ITEM_DESC)))
        console.log(items)
	})
 
    $:console.log(value)
</script>

<h1>Add items to your basket</h1>
<p>Use the dropdown to select, type to search, then add it to your basket.</p>
{#if items}
    {#if value}<p>{value.length} item{value.length>1 ? "s" : ''} selected.</p>{/if}
    <Select placeholder={"Click and select or type to search for items"} items={sorted} bind:justValue={value} label="ITEM_DESC" itemId="ITEM_ID" multiple={true} multiFullItemClearable {groupBy} >
        <div slot="clear-icon"> Clear all </div> 
    </Select>
{/if}

<h3>Your basket</h3>
{#if avgprice}
<p>Average prices of items in {timeFormat("%B %Y")(timeParse("%Y-%m-%d")(avgprice.columns[avgprice.columns.length-1]))} and the latest monthly and annual growth rates</p>
{/if}


{#if Array.isArray(value)}
<hr>
{#each value as item}
    <p>Name: {items.filter(d=>d.ITEM_ID==item)[0]['ITEM_DESC']}</p>
    <p>Weight or size: {items.filter(d=>d.ITEM_ID==item)[0]['WEIGHT\\SIZE']}</p>
    <p>Average price: £{format(",.2f")(avgprice.filter(d=>d.ITEM_ID==item)[0][avgprice.columns[avgprice.columns.length-1]])}</p>
    <p>Monthly growth: {format(".1f")(monthlygrowth.filter(d=>d.ITEM_ID==item)[0][monthlygrowth.columns[monthlygrowth.columns.length-1]])}%</p>
    <p>Annual growth: {format(".1f")(annualgrowth.filter(d=>d.ITEM_ID==item)[0][annualgrowth.columns[annualgrowth.columns.length-1]])}%</p>
    <hr>
{/each}
{/if}



<style>
    :global(.list-group-title){
        text-transform: none;
    }
</style>
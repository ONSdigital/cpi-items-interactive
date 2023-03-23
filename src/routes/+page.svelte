<script>
	import { csv } from 'd3-fetch';
	import { onMount } from 'svelte';
	import { autoType } from 'd3-dsv';
	import { groups } from 'd3-array';
	// import Select from 'svelte-select';
	import { format } from 'd3-format';
	// import { timeParse, timeFormat } from 'd3-time-format';
	// import { slide } from 'svelte/transition';
	import {quintOut} from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	// import { Table } from 'svelte-tabular-table'
	import SortTable from "../components/helpers/SortTable.svelte"

	let chained;
	let items;
	let avgprice;
	let monthlygrowth;
	let annualgrowth;
	let allitems
	// let checked = [];
	let isChecked = {};
	let sorted;
	let grouped;
	let selected;
	let checkedOrder= {};
	let y;
	// const groupBy = (item) => item.CATEGORY;

	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	onMount(async () => {
		// (chained = await csv(
		// 	'https://gist.githubusercontent.com/henryjameslau/8e98fc34b7aac3f7f98251d0a856e129/raw/f9c696905c74700a7a0bf2649cd0f4675c18399d/chained.csv',
		// 	autoType
		// )),
			(items = await csv(
				'https://gist.githubusercontent.com/henryjameslau/24fd5bf8000c2f04a2140ced6684da61/raw/3bd04b7516fca33e5128542416bac744bd4327ab/metadata.csv',
				autoType
			)),
			(avgprice = await csv(
				'https://gist.githubusercontent.com/henryjameslau/3ea034526695d7987863c4d6076449d3/raw/0a8c8145e0085269b18de7dd725666c69035797d/avgprice.csv',
				autoType
			)),
			(monthlygrowth = await csv(
				'https://gist.githubusercontent.com/henryjameslau/7b3e6c953c5efa15869d1f1dfadb4294/raw/d5b96cb7ff890a4d0edbd9bd0592d7a0c63e21c8/monthlygrowth.csv',
				autoType
			)),
			(annualgrowth = await csv(
				'https://gist.githubusercontent.com/henryjameslau/76f038ec284f08d65a3b224986ef57f3/raw/96ac1ccbbf88de2733aac4f1184a57ee78e3a8c5/annualgrowth.csv',
				autoType
			));

		sorted = items.sort(
		 	(a, b) => a['SUB CATEGORY 1'].toString().localeCompare(b['SUB CATEGORY 1']) ||a.ITEM_DESC.localeCompare(b.ITEM_DESC)
		);

		grouped = groups(items, (d) => d['SUB CATEGORY 1']);

	});

	let columns = [
		{label:"Name",prop:"Name",sort:true,type:"text"},
		{label:"Weight or size",prop:"Weight or size",sort:true,type:"text",formatFn:(d)=> d=='null' ? '' : d},
		{label:"Average price",prop:"Average price",sort:true,type:"text",formatFn:(d)=>'£'+format(',.2f')(d)},
		{label:"Monthly growth",prop:"Monthly growth",sort:true,type:"number",formatFn:(d)=>format('.1f')(d)+'%'},
		{label:"Annual growth",prop:"Annual growth",sort:true,type:"number",formatFn:(d)=>format('.1f')(d)+'%'}
	]



	$: selected = Object.entries(isChecked).filter(d=>d[1]==true);
	$: selectedOrdered=selected.map(d=>[...d,checkedOrder[d[0]]]).sort((a,b)=>b[2]-a[2])

	$: data=selectedOrdered.map(item=>({
		'Name':items.filter((d) => d.ITEM_ID == item[0])[0]['ITEM_DESC'],
		'Weight or size':items.filter((d) => d.ITEM_ID == item[0])[0]['WEIGHT\\SIZE'] == null ? '' : items.filter((d) => d.ITEM_ID == item[0])[0]['WEIGHT\\SIZE'],
		'Average price': avgprice.filter((d) => d.ITEM_ID == item[0])[0][
							avgprice.columns[avgprice.columns.length - 1]
						],
		'Monthly growth':monthlygrowth.filter((d) => d.ITEM_ID == item[0])[0][
							monthlygrowth.columns[monthlygrowth.columns.length - 1]
						],
		'Annual growth':annualgrowth.filter((d) => d.ITEM_ID == item[0])[0][
							annualgrowth.columns[annualgrowth.columns.length - 1]
						],
		'id':item[0]
	}))

	$: console.log(data)

	$: config = {
		init: {
			name: 'sortable-example',
			keys: ['Name', 'Weight or size', 'Average price', 'Monthly growth', 'Annual growth'],
			index: '_id',
			data
		},
		features: {
			sortable: {
				key: 'name'
			}
		}
	}

	function scroll(){
		y=allitems.scrollTop
	}

    function removeItem(id){
        isChecked[id]=false;
    }

	function handleChange(){
		checkedOrder[this.getAttribute('id')]=new Date().getTime()
	}




</script>

<h1>Add items to your basket</h1>
<!-- <p>Use the dropdown to select, type to search, then add it to your basket.</p>
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
{/if} -->

{#if grouped}
	<div bind:this={allitems} on:scroll={scroll} id="allitems">
		<div style="opacity: {1 - Math.max(0, y / 40)}" id='scrollmore'>Scroll to see more items</div>
		{#each grouped as groups}
			<h3>{groups[0]}</h3>
			<div class="flex">
				{#each groups[1].filter(e=>!selected.map(e=>+e[0]).includes(e.ITEM_ID)) as item(item.ITEM_ID)}
					<div class="item" in:receive="{{key: item.ITEM_ID}}" out:send="{{key: item.ITEM_ID}}">
						<input type="checkbox" id={item.ITEM_ID} on:change={handleChange} bind:checked={isChecked[item.ITEM_ID]} />
						<label for={item.ITEM_ID}
							><span class="bold">{item.ITEM_DESC}</span>
							{item['WEIGHT\\SIZE'] ? item['WEIGHT\\SIZE'] : ''}</label
						>
					</div>
				{/each}
			</div>
		{/each}
		<div style="opacity: {1 - Math.max(0, y / 90)}" id='scrollmoremore'>Scroll more more ↓</div>
	</div>
{/if}


<div id="results">
	<h2>Your selected items</h2>

	{#if selected.length>0}
	<SortTable {columns} rows={data} mobile={false} on:remove={(e)=>removeItem(e.detail)}/>
	{/if}
	
	<!-- {#if selected}	
		{#each selectedOrdered as item(item[0])}
			<div in:receive="{{key: item.ITEM_ID}}" out:send="{{key: item.ITEM_ID}}">
				<p>Name: {items.filter((d) => d.ITEM_ID == item[0])[0]['ITEM_DESC']}</p>
				<p>Weight or size: {items.filter((d) => d.ITEM_ID == item[0])[0]['WEIGHT\\SIZE']}</p>
				<p>
					Average price: £{format(',.2f')(
						avgprice.filter((d) => d.ITEM_ID == item[0])[0][
							avgprice.columns[avgprice.columns.length - 1]
						]
					)}
				</p>
				<p>
					Monthly growth: {format('.1f')(
						monthlygrowth.filter((d) => d.ITEM_ID == item[0])[0][
							monthlygrowth.columns[monthlygrowth.columns.length - 1]
						]
					)}%
				</p>
				<p>
					Annual growth: {format('.1f')(
						annualgrowth.filter((d) => d.ITEM_ID == item[0])[0][
							annualgrowth.columns[annualgrowth.columns.length - 1]
						]
					)}
				</p>
				<button on:click={removeItem(item[0])}>Remove</button>
				<hr />
			</div>
			
		{/each}
	{/if} -->

	<!-- {#if data}
	<Table {...config}/>
	{/if} -->

</div>

<style>
	:global(.list-group-title) {
		text-transform: none;
	}

	div#allitems:hover{
		box-shadow: 3px 3px 3px rgb(236, 236, 236),-3px -3px 3px rgb(236, 236, 236),-3px 3px 3px rgb(236, 236, 236),3px -3px 3px rgb(236, 236, 236);
	}

	div.item {
		margin: 5px;
		border: 2px solid #206095;
	}

	.bold {
		font-weight: 700;
	}

	.flex {
		display: flex;
		flex-flow: wrap;
	}

	label {
		margin: 2px;
	}

	div#allitems {
		height: 400px;
		overflow-y: scroll;
		position: relative;
		text-align: center;
		padding-top: 2em;
		box-sizing: border-box;
	}

	div#scrollmore{
		display: block;
	}

	div#scrollmoremore{
		position:sticky;
		bottom:0;
		background:white;
		pointer-events: none;
		padding:1em 0em;
	}

    /* p{
        margin:0;
    } */
</style>

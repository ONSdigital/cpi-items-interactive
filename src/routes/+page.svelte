<script>
	import { csv } from 'd3-fetch';
	import { onMount } from 'svelte';
	import { autoType } from 'd3-dsv';
	import { groups } from 'd3-array';
	import { format } from 'd3-format';
	import { timeParse } from 'd3-time-format';
	import {quintOut} from 'svelte/easing';
	import { crossfade,fade } from 'svelte/transition';
	import SortTable from "../components/helpers/SortTable.svelte";
	import Summary from "../components/helpers/Summary.svelte";

	let items; //metadata
	let avgprice;
	let monthlygrowth;
	let annualgrowth;
	let allitems
	let isChecked = {}; // object for seeing which items are selected
	let grouped; // nested items with hierarchy
	let selected;
	let checkedOrder= {}; //object to store which order the items are selected in
	let y;
	let filter;
	let regex;
	let typing = false;


	onMount(async () => {
			(items = await csv(
				'https://gist.githubusercontent.com/henryjameslau/8084919379b42b260e84c28ec7986612/raw/754a00aaba4c7d2ec759282732bec1f0cdad190e/metadata-lowest-category.csv',
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
	});

	let columns = [
		{label:"Name",prop:"Name",sort:true,type:"text"},
		{label:"Weight or size",prop:"Weight or size",sort:true,type:"text",formatFn:(d)=> d=='null' ? '' : d},
		{label:"Average price",prop:"Average price",sort:true,type:"number",formatFn:(d)=>'£'+format(',.2f')(d)},
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
		'id':item[0],
		'timeline':Object.entries(avgprice.filter((d) => d.ITEM_ID == item[0])[0]).filter(d=>d[0]!='ITEM_ID').map(d=>({date:timeParse("%Y-%m-%d")(d[0]),'value':d[1]})),
		'pricelastmonth':avgprice.filter((d) => d.ITEM_ID == item[0])[0][
							avgprice.columns[avgprice.columns.length - 2]
						]
	}))

	function scroll(){
		y=allitems.scrollTop
	}

    function removeItem(id){
        isChecked[id]=false;
    }

	function clearAll(){
		selected.forEach(d=>isChecked[d[0]]=false)
	}

	function handleChange(){
		checkedOrder[this.getAttribute('id')]=new Date().getTime()
	}

	function clearInput(){
		filter=""
	}

	function input(){
		typing = true;
		setTimeout(()=>{typing=false;},200)
	}

	$: [send, receive] = crossfade({
		duration: (filter&&typing) ? 0 : 200,

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;
			return {
				duration: filter&&typing ? 0 : 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	$: if(filter) regex = new RegExp(filter, "gi")

	$: if(filter){
		grouped = groups(items, (d) => d['lowestCategory']);
		grouped = grouped.filter(d=>(d[0].match(regex)) || (d[1].some(e=>e.ITEM_DESC.match(regex))))
			.map(function(d){
			if(d[0].match(regex)){return [d[0],d[1]]}
			else{return [d[0],d[1].filter(e=>e.ITEM_DESC.match(regex))]}
		}) 
		console.log(grouped)
	}

	$:if(items&&!filter){
		grouped = groups(items.sort(
		 	(a, b) => a['lowestCategory'].toString().localeCompare(b['lowestCategory']) ||a.ITEM_DESC.localeCompare(b.ITEM_DESC)
		), (d) => d['lowestCategory']);
	}

</script>
<div id="top">
	<div id='title'>
		<h1>How is the average price of items changing over time?</h1>
		<p>Use our interactive to see how the price of different items have changed since last year.</p>

	</div>

	<div id="input">
	<h2>Add items to your basket</h2>
	<p>Select items to add them to your basket.</p>
	<div>
		<label for="filter">Filter items:</label><input id="filter" type=text bind:value={filter} on:input={input}/><button on:click={clearInput}>Clear filter</button>
	</div>
	

	{#if grouped}
		<div bind:this={allitems} on:scroll={scroll} id="allitems">
			<div style="opacity: {1 - Math.max(0, y / 40)}" id='scrollmore'>Scroll to see more items</div>
			{#each grouped as groups}
				<!-- https://stackoverflow.com/questions/3294576/javascript-highlight-substring-keeping-original-case-but-searching-in-case-inse -->
				{#if groups[1].filter(e=>!selected.map(e=>+e[0]).includes(e.ITEM_ID)).length>0}
				<h3>{@html filter ? groups[0].replace(regex, (str)=> '<span class="highlight">'+str+'</span>') : groups[0]}</h3>
				<div class="flex">
					{#each groups[1].filter(e=>!selected.map(e=>+e[0]).includes(e.ITEM_ID)) as item(item.ITEM_ID)}
						<div class="item" in:receive|local="{{key: item.ITEM_ID}}" out:send|local="{{key: item.ITEM_ID}}">

							<input type="checkbox" id={item.ITEM_ID} on:change={handleChange} bind:checked={isChecked[item.ITEM_ID]} />
							<label for={item.ITEM_ID}
								><span class="bold">{@html filter ? item.ITEM_DESC.replace(regex, (str)=> '<span class="highlight">'+str+'</span>') : item.ITEM_DESC}</span>
								{item['WEIGHT\\SIZE'] ? item['WEIGHT\\SIZE'] : ''}</label
							>
						</div>
					{/each}
				</div>
				{/if}
			{/each}
			<div style="opacity: {1 - Math.max(0, y / 90)}" id='scrollmoremore'>Scroll more more ↓</div>
		</div>
	{/if}
	</div>
</div>

<div id="results">
	<h2>Your basket</h2>
	<p>
		Average prices of items in [date] and the latest monthly and annual growth rate.
	</p>

	{#if selected.length>0}
	<SortTable {columns} rows={data} mobile={false} on:clearAll={clearAll} on:remove={(e)=>removeItem(e.detail)}/>
	{/if}

</div>

{#if data.length>0}
<div id="summary">
	<Summary {data}/>
</div>
{/if}

<style>
	div#allitems:hover{
		box-shadow: 3px 3px 3px rgb(236, 236, 236),-3px -3px 3px rgb(236, 236, 236),-3px 3px 3px rgb(236, 236, 236),3px -3px 3px rgb(236, 236, 236);
	}

	div.item {
		margin: 5px;
		border: 2px solid #206095;
	}

	div#top{
		background-color: #E9EFF4;
	}

	div#title, div#input{
		padding:25px;
	}

	:global(span.highlight){
		background-color: #FFFF00;
	}

	.bold {
		font-weight: 700;
	}

	.flex {
		display: flex;
		flex-flow: wrap;
	}

	div#allitems {
		height: 400px;
		overflow-y: scroll;
		position: relative;
		text-align: center;
		padding-top: 2em;
		box-sizing: border-box;
		background-color: white;
		margin-top: 10px;
	}

	div#results{
		background-color: #F5F5F6;
		margin-top: 10px;
		padding: 25px;
	}

	div#summary{
		background-color: #E9EFF4;
		margin-top: 10px;
		padding: 25px;
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

	#results >h2{
		color: #206095;
	}

    h1,h2,h3{
        margin:0;
    }
</style>

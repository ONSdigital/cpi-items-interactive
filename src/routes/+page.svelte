<script>
	import { csv } from 'd3-fetch';
	import { onMount } from 'svelte';
	import { autoType } from 'd3-dsv';
	import { groups } from 'd3-array';
	import { format } from 'd3-format';
	import { timeParse, timeFormat } from 'd3-time-format';
	import { quintOut } from 'svelte/easing';
	import { crossfade,fade } from 'svelte/transition';
	import SortTable from "../components/helpers/SortTable.svelte";
	import Summary from "../components/helpers/Summary.svelte";
	import Share from "../components/helpers/Share.svelte"
	import DownloadData from "../components/helpers/DownloadData.svelte";
	import Embed from "../components/helpers/Embed.svelte"
	import Feedback from "../components/helpers/Feedback.svelte"
	import pym from "pym.js";

	let items; //metadata
	let itemsSorted
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
	let pymChild;
	let w;
	let mainElementHeight;
	let lastmonth;

	onMount(async () => {
			(items = await csv(
				'../metadata.csv',
				autoType
			)),
			(avgprice = await csv(
				'../avgprice.csv',
				autoType
			)),
			(monthlygrowth = await csv(
				'../monthlygrowth.csv',
				autoType
			)),
			(annualgrowth = await csv(
				'../annualgrowth.csv',
				autoType
			));

			itemsSorted = items.sort(
		 	(a, b) => a['Category1'].toString().localeCompare(b['Category1']) || a['Category2'].toString().localeCompare(b['Category2']) ||a.ITEM_DESC.localeCompare(b.ITEM_DESC)
			)

			grouped = groups(itemsSorted,d=>d.Category1,d=>d.Category2)

			console.log(grouped)

			lastmonth=timeParse("%Y-%m-%d")(monthlygrowth.columns[monthlygrowth.columns.length-1])

			pymChild = new pym.Child


			let checkitems = new RegExp('[0-9]{6}')
			let parent = new URLSearchParams(document.location.search).get("parentUrl");
			let child = window.location.href.includes('#')
			let childcode = child ? child.split("#")[1].split(',').map(d=>+d) : null
			let parentcode = parent ? parent.split("#")[1].split(',').map(d=>+d) : null;
			
			if (parentcode && parentcode.every(d=>checkitems.test(d))){
				parentcode.forEach(d=>isChecked[d]=true)
				
			}
			if (childcode && childcode.every(d=>checkitems.test(d))){
				childcode.forEach(d=>isChecked[d]=true)
				
			}
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

    function removeItem(id){
        isChecked[id]=false;
		updateHeight()
    }

	function clearAll(){
		selected.forEach(d=>isChecked[d[0]]=false)
		updateHeight()
	}

	function handleChange(){
		checkedOrder[this.getAttribute('id')]=new Date().getTime()
		updateHeight()
	}



	function updateHeight(event) {
        // note that i'm actually ignoring the event object itself
		// only process if the first main element exists    
        if (document.getElementsByTagName('main')[0]) {
			
            // sending height of the first <main> tag to the Pym parent
            // as a message instead of using pymChild.sendHeight, which sends the 
            // height of the <body> tag. 
			setTimeout(()=>{
				mainElementHeight = document.getElementsByTagName('main')[0].offsetHeight.toString()
				console.log('sendHeight',mainElementHeight)
				pymChild.sendMessage('height', mainElementHeight);
			},250)	
        }
    }
	$: if(w&&pymChild) {
		updateHeight()
	} // trick to update height on w change

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

</script>
<main>

<div id="top" bind:clientWidth={w}>
	<div id='title'>
		<h1>How is the average price of items changing over time?</h1>
		<p>Use our interactive to see how the price of different items have changed since last year.</p>

	</div>

	<div id="input">
	<h2>Add items to your basket</h2>
	<p>Select items to add them to your basket.</p>
		{#if grouped}
		{#each grouped as group}
			<details>
				<summary>{group[0]}</summary>
				{#each group[1] as subgroup}
					<details class='subgroup'>
						<summary>{subgroup[0]}</summary>
						<div class='flex'>
						{#each subgroup[1] as item(item.ITEM_ID)}
							<div>
								<input type="checkbox" id={item.ITEM_ID} on:change={handleChange} bind:checked={isChecked[item.ITEM_ID]} />
								<label for={item.ITEM_ID}>
									<span class="bold">{item.ITEM_DESC}</span>
									{item['WEIGHT\\SIZE'] ? item['WEIGHT\\SIZE'] : ''}
								</label>
							</div>
						{/each}
						</div>
					</details>
				{/each}
			</details>
		{/each}
		{/if}
	</div>
</div>

<div id="results">
	<h2>Your basket</h2>
	<p>
		Average prices of items in {timeFormat("%B %Y")(lastmonth)} and the latest monthly and annual growth rate.
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

<div>
	<h2>Use and share</h2>
	<div class="flex items-center gap-x-6 gap-y-0.5 lg:gap-x-8 flex-wrap ">
		
		<!-- <Share/>
		

		<Feedback/>

		<DownloadData/> 

		<Embed/> -->
	</div>
		
	
</div>

</main>

<style>




	div#top{
		background-color: #E9EFF4;
	}

	div#title, div#input{
		padding:25px;
	}

	:global(span.highlight){
		background-color: #f0f762;
	}

	.subgroup{
		margin-left:10px;
	}

	.flex {
		display: flex;
		flex-flow: wrap;
		flex-direction: column;
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


	#results >h2{
		color: #206095;
	}

    h1,h2{
        margin:0;
    }


	.gap-y-0\.5 {
		row-gap: .125rem;
	}

	.gap-x-6 {
	-moz-column-gap: 1.5rem;
	column-gap: 1.5rem;
	}
	.items-center {
		align-items: center;
	}
	.flex-wrap {
		flex-wrap: wrap;
	}
	.flex {
		display: flex;
	}

</style>

<script>
	import { csv } from 'd3-fetch';
	import { onMount } from 'svelte';
	import { autoType } from 'd3-dsv';
	import { groups } from 'd3-array';
	import { format } from 'd3-format';
	import { timeParse, timeFormat } from 'd3-time-format';
	import { quintOut } from 'svelte/easing';
	import { crossfade,fade,scale } from 'svelte/transition';
	import SortTable from "../components/helpers/SortTable.svelte";
	import Summary from "../components/helpers/Summary.svelte";
	import Share from "../components/helpers/Share.svelte"
	import DownloadData from "../components/helpers/DownloadData.svelte";
	import Embed from "../components/helpers/Embed.svelte"
	import Feedback from "../components/helpers/Feedback.svelte"
	import pym from "pym.js";
	import Select from 'svelte-select'
	
	let items; //metadata
	let itemsSorted
	let avgprice;
	let monthlygrowth;
	let annualgrowth;
	let isChecked = {}; // object for seeing which items are selected
	let grouped; // nested items with hierarchy
	let selected;
	let checkedOrder= {}; //object to store which order the items are selected in
	let filter;
	let pymChild;
	let w;
	let mainElementHeight;
	let lastmonth;
	let value="";
	let visible = false;
	let sortOrder = ["Food and drink"]

	onMount(async () => {
			(items = await csv(
				'./metadata.csv',
				autoType
			)),
			(avgprice = await csv(
				'./avgprice.csv',
				autoType
			)),
			(monthlygrowth = await csv(
				'./monthlygrowth.csv',
				autoType
			)),
			(annualgrowth = await csv(
				'./annualgrowth.csv',
				autoType
			));

			itemsSorted = items.sort(
		 	(a, b) => sortOrder.indexOf(b['Category1']) - sortOrder.indexOf(a['Category1']) || a['Category1'].toString().localeCompare(b['Category1']) || a['Category2'].toString().localeCompare(b['Category2']) ||a.ITEM_DESC.localeCompare(b.ITEM_DESC)
			)

			grouped = groups(itemsSorted,d=>d.Category1,d=>d.Category2)


			lastmonth=timeParse("%Y-%m-%d")(monthlygrowth.columns[monthlygrowth.columns.length-1])

			pymChild = new pym.Child


			let checkitems = new RegExp('[0-9]{6}')
			let parent = new URLSearchParams(document.location.search).get("parentUrl");
			let child = window.location.href.includes('#')
			let childpage = window.location.href
			let childcode = child ? childpage.split("#")[1].split(',').map(d=>+d) : null
			let parentcode = parent ? parent.split("#")[1].split(',').map(d=>+d) : null;
			
			if (parentcode && parentcode.every(d=>checkitems.test(d))){
				parentcode.forEach(d=>isChecked[d]=true)
				
			}
			if (childcode && childcode.every(d=>checkitems.test(d))){
				childcode.forEach(d=>isChecked[d]=true)
				
			}

	});


	let columns = []
	$: if(w<425){
		columns=[
			{label:"Name",prop:"Name",sort:true,type:"text"},
		{label:"Weight or size",prop:"Weight or size",sort:true,type:"text",formatFn:(d)=> d=='null' ? '' : d},
		{label:"Average price",prop:"Average price",sort:true,type:"number",formatFn:(d)=>'£'+format(',.2f')(d)},
		]
	}else if(w<500){
		columns=[
			{label:"Name",prop:"Name",sort:true,type:"text"},
		{label:"Weight or size",prop:"Weight or size",sort:true,type:"text",formatFn:(d)=> d=='null' ? '' : d},
		{label:"Average price",prop:"Average price",sort:true,type:"number",formatFn:(d)=>'£'+format(',.2f')(d)},
		{label:"Annual growth",prop:"Annual growth",sort:true,type:"number",formatFn:(d)=>d==null ? 'Unavailable' : format('.1f')(d)+'%'}
		]
	}else{
		columns= [
		{label:"Name",prop:"Name",sort:true,type:"text"},
		{label:"Weight or size",prop:"Weight or size",sort:true,type:"text",formatFn:(d)=> d=='null' ? '' : d},
		{label:"Average price",prop:"Average price",sort:true,type:"number",formatFn:(d)=>'£'+format(',.2f')(d)},
		{label:"Price last year",prop:"pricelastyear",sort:true,type:"number",formatFn:(d)=>d==null ? 'Unavailable' : '£'+format(',.2f')(d)},
		{label:"Annual growth",prop:"Annual growth",sort:true,type:"number",formatFn:(d)=>d==null ? 'Unavailable' : format('.1f')(d)+'%'}
	]
	}

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
						],
		'pricelastyear':avgprice.filter((d)=>d.ITEM_ID == item[0])[0][avgprice.columns[avgprice.columns.length-13]]
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

	function addFromSearch(id){
		isChecked[id]=true;
		updateHeight()
		clearSearch()
		visible=true;
		setTimeout(()=>visible=false,1500)
	}

	function clearSearch(){
		value=""
	}

	function oninput(e){
		document.activeElement.addEventListener('keydown',(d)=>
		{
			if(d.keyCode==13){addItemFromSearchEnter(e.detail.ITEM_ID)}
		})
		
	}

	function addItemFromSearchEnter(id){
		isChecked[id]=true;
		updateHeight()
		clearSearch()
		visible=true;
		setTimeout(()=>visible=false,1500)
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
		<hr class="white"/>
	</div>
	

	<div class="input">
		<h2>Shopping items search</h2>
		<p>Search for items from the list to add to your shopping basket.</p>

		<Select on:input={oninput} --selected-item-color="#206095" --item-hover-color="#206095" --item-color="#206095" --group-title-color="#206095" --group-title-text-transform="none" --placeholder-color="#206095" --border-radius="0" --border-focused="2px solid #206095" --border-hover="2px solid #206095" --border="2px solid #206095" placeholder="Type to search for items" items={items} groupBy={(item)=>item.Category1} label="ITEM_DESC" clearable={false} id="ITEM_ID" bind:value />

		<div id="searchbuttons" class='hflex'>
			<button class="{value == '' ? "disabled" : ""}" on:click={addFromSearch(value.ITEM_ID)}>Add to basket</button>
			{#if value}<button on:click={clearSearch}>Clear search</button>{/if}
			{#if visible}
			<div in:scale out:fade id='itemadded'>Item added!<img height=14 width=14 alt="" src='./tick.svg'/></div>
			{/if}
		</div>	
		<hr class="white"/>

		<h2>Shopping items list</h2>
		<p>Choose items from our shopping list to add them to your basket.</p>
	</div>

	<div id='receiptheader'>
		<svg viewBox="0 0 360 6">
			<path d="M 0,6 L 12,0 L 24,6 L 36,0 L 48,6 L 60,0 L 72,6 L 84,0 L 96,6 L 108,0 L 120,6 L 132,0 L 144,6 L 156,0 L 168,6 L 180,0 L 192,6 L 204,0 L 216,6 L 228,0 L 240,6 L 252,0 L 264,6 L 276,0 L 288,6 L 300,0 L 312,6 L 324,0 L 336,6 L 348,0 L 360,6" fill="#F8FAFC"></path>
		  </svg>
	</div>

	<div class="input receiptbackground">

<!-- 	
		<div>
			<label for="filter">Filter items:</label><input id="filter" type=text bind:value={filter} on:input={input}/><button on:click={clearInput}>Clear filter</button>
		</div> -->
		
			<div id="allitems">
				{#if grouped}
					{#each grouped as groups, i}
						<details open={i==0}>
						<summary><h2>{groups[0]}</h2>
							<span class="ons-details__icon">
								<svg class="ons-svg-icon" viewBox="0 0 8 13" xmlns="http://www.w3.org/2000/svg" focusable="false" fill="currentColor"><path d="M5.74,14.28l-.57-.56a.5.5,0,0,1,0-.71h0l5-5-5-5a.5.5,0,0,1,0-.71h0l.57-.56a.5.5,0,0,1,.71,0h0l5.93,5.93a.5.5,0,0,1,0,.7L6.45,14.28a.5.5,0,0,1-.71,0Z" transform="translate(-5.02 -1.59)"></path></svg>
							</span>	
							</summary>	
							
						{#each groups[1] as subgroup}
							<h3>{subgroup[0]}</h3>
							<div class="hflex">
								{#each subgroup[1].filter(e=>!selected.map(e=>+e[0]).includes(e.ITEM_ID)) as item(item.ITEM_ID)}
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
						</details>
					{/each}
				{/if}
			</div>

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
	<div class="hflex items-center gap-x-6 gap-y-0.5 lg:gap-x-8 flex-wrap ">
		
		<Share {selected} />
		

		<Feedback/>

		<DownloadData/> 

		<!-- <Embed/> -->
	</div>
		
	
</div>

</main>


<!-- <svelte:body on:keydown={keydown}/> -->

<style>
	*{
		box-sizing: border-box;
	}

	#receiptheader{
		overflow: hidden;
		height:17px;
	}

	.receiptbackground{
		background-color: #F8FAFC;
	}

	div#top{
		background-color: #E9EFF4;
	}

	div#title, div.input{
		padding:25px;
		padding-bottom: 0;
		padding-top:0;
	}

	.white{
		border:2px solid white;
	}

	summary{
		border-top: 1px solid #707071;
		margin:0;
		padding-bottom: .9rem;
		padding-top: 1rem;
		width:100%;
		color: #206095;
		cursor: pointer;
		display:inline-block;
		outline:none;
		padding-left: 1.5rem;
		pointer-events: initial;
		position: relative;
	}

	summary h2{
		line-height: 1.4;
		margin: 0 1rem 0 0;
		display: inline-block;
		font-size: 18px;
		font-weight: 700;
		text-underline-position: under;
		transform:translateY(-1px)
	}

	summary h2:hover:not(:focus){
	text-decoration: underline solid var(--ons-color-text-link-hover) 2px;
	color: var(--ons-color-text-link-hover);

	}


	summary:focus h2 {
	background-color: var(--ons-color-focus);
	-webkit-box-decoration-break: clone;
	box-decoration-break: clone;
	-webkit-box-shadow: 0 -2px var(--ons-color-focus),0 4px var(--ons-color-text-link-focus);
	box-shadow: 0 -2px var(--ons-color-focus),0 4px var(--ons-color-text-link-focus);
	color: var(--ons-color-text-link-focus);
	outline: 3px solid transparent;
	outline-offset: 1px;
	text-decoration: none;
}

	.ons-details__icon{
		top: .8rem;
		display: inline-block;
		fill: var(--ons-color-text-link);
		height: 1.5rem;
		left: -.15rem;
		position: absolute;
		width: 1.5rem;
	}

	.ons-svg-icon {
		height: 1rem;
		vertical-align: middle;
		width: 1rem;
	}

	details[open] .ons-details__icon {
		left: -.1rem;
		top: 1.2rem;
		-webkit-transform: rotate(90deg);
		transform: rotate(90deg);
	}

	.bold {
		font-size:16px;
		font-weight: 600;
	}

	.hflex{
		display:flex;
		flex-flow: wrap;
	}

	.hflex:last-child{
		margin-bottom: 20px;
	}

	div#allitems {
		height: 400px;
		overflow-y: scroll;
		position: relative;
		text-align: left;
		padding-top: 1.5em;
		background-color: #F8FAFC;
	}

	:global(.focused){
		outline:3px solid orange;
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

	.item{
		position: relative;
		margin:2px 0;
	}

	.item span{
		font-size: 16px;
	}

	input[type=checkbox] {
		-webkit-appearance: none;
		-moz-appearance: none;
		appearance: none;
		background-color: var(--ons-color-input-bg);
		border: 2px solid #222;
		border-radius: .2rem;
		cursor: pointer;
		height: 22px;
		left: 0;
		top: 0;
		width: 22px;
		position:absolute;
		z-index: 1;
	}

	input[type=checkbox]:focus{
		outline: 3px solid #F39431;
	}

	.item label{
		padding: 0 10px 0 33px;
		cursor: pointer;
		display: block;
		width: 100%;
	}


	#searchbuttons button{
		height:40px;
		background: #0F8243;
		color:white;
		border: none;
		box-sizing: border-box;
		font-size: 14px;
		font-weight:700;
		padding: 12px 10px;
		margin: 20px 20px 20px 0;
	}

	#searchbuttons button:focus{
		outline: 3px solid #F39431;
	}

	#searchbuttons button.disabled{
		background:#9FCDB4;
	}

	#itemadded{
		height:40px;
		background-color: #206095;
		color: white;
		font-weight: 700;
		font-size:14px;
		padding: 12px 10px;
		margin: 20px 20px 20px 0;
	}

	#itemadded img{
		margin-left:10px;
	}

	#results >h2{
		color: #206095;
	}


	h1,h2{
        margin:0;
    }

	h3{
		font-family: Open Sans;
		font-size: 16px;
		font-weight: 700;
		line-height: 25px;
		letter-spacing: 0em;
		text-align: left;
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

	:root {
	--ons-color-black: #222;
	--ons-color-grey-100: #414042;
	--ons-color-grey-75: #707071;
	--ons-color-grey-35: #bcbcbd;
	--ons-color-grey-25: #d5d5d6;
	--ons-color-grey-15: #e2e2e3;
	--ons-color-grey-5: #f5f5f6;
	--ons-color-white: #fff;
	--ons-color-night-blue: #003c57;
	--ons-color-spring-green: #a8bd3a;
	--ons-color-ocean-blue: #206095;
	--ons-color-sky-blue: #27a0cc;
	--ons-color-aqua-teal: #00a3a6;
	--ons-color-leaf-green: #0f8243;
	--ons-color-ruby-red: #d0021b;
	--ons-color-jaffa-orange: #fa6401;
	--ons-color-sun-yellow: #fbc900;
	--ons-color-neon-yellow: #f0f762;
	--ons-color-ocean-blue-tint: #e9eff4;
	--ons-color-ocean-blue-vibrant: #1f8fd8;
	--ons-color-spring-green-tint: #c3c5b8;
	--ons-color-leaf-green-tint: #e7f3ec;
	--ons-color-leaf-green-vibrant: #10ca64;
	--ons-color-leaf-green-dark-10: #073d20;
	--ons-color-leaf-green-dark-5: #0c6b37;
	--ons-color-ruby-red-tint: #fae6e8;
	--ons-color-ruby-red-vibrant: #fd112d;
	--ons-color-jaffa-orange-tint: #fff0e6;
	--ons-color-jaffa-orange-vibrant: #ff7b24;
	--ons-color-sun-yellow-dark: #e2b500;
	--ons-color-branded: var(--ons-color-ocean-blue);
	--ons-color-branded-text: var(--ons-color-ocean-blue);
	--ons-color-branded-tint: var(--ons-color-ocean-blue-tint);
	--ons-color-branded-secondary: var(--ons-color-night-blue);
	--ons-color-branded-tertiary: var(--ons-color-aqua-teal);
	--ons-color-branded-supporting: var(--ons-color-spring-green);
	--ons-color-branded-supporting-tint: var(--ons-color-branded-supporting);
	--ons-color-header: var(--ons-color-branded);
	--ons-color-header-neutral: var(--ons-color-white);
	--ons-color-header-masthead: var(--ons-color-white);
	--ons-color-header-masthead-internal: var(--ons-color-branded-secondary);
	--ons-color-header-masthead-neutral: var(--ons-color-black);
	--ons-color-header-title: var(--ons-color-text);
	--ons-color-header-navigation-links: var(--ons-color-text);
	--ons-color-service-links: var(--ons-color-white);
	--ons-color-cta-bg: var(--ons-color-branded-tint);
	--ons-color-banner-bg: var(--ons-color-grey-5);
	--ons-color-banner-bg-dark: var(--ons-color-grey-15);
	--ons-color-page-light: var(--ons-color-white);
	--ons-color-banner-browser-bg: var(--ons-color-black);
	--ons-color-hero-bg: var(--ons-color-branded-tint);
	--ons-color-hero-bg-dark: var(--ons-color-branded);
	--ons-color-text: var(--ons-color-black);
	--ons-color-text-light: var(--ons-color-grey-75);
	--ons-color-text-inverse: var(--ons-color-white);
	--ons-color-text-banner: var(--ons-color-black);
	--ons-color-text-link: var(--ons-color-ocean-blue);
	--ons-color-text-link-hover: var(--ons-color-night-blue);
	--ons-color-text-link-active: var(--ons-color-night-blue);
	--ons-color-text-link-focus: var(--ons-color-black);
	--ons-color-text-inverse-link: var(--ons-color-white);
	--ons-color-text-inverse-link-hover: var(--ons-color-grey-5);
	--ons-color-text-banner-link: var(--ons-color-grey-100);
	--ons-color-text-banner-link-hover: var(--ons-color-black);
	--ons-color-text-metadata: var(--ons-color-black);
	--ons-color-focus: var(--ons-color-sun-yellow);
	--ons-color-focus-dark: var(--ons-color-sun-yellow-dark);
	--ons-color-highlight: var(--ons-color-neon-yellow);
	--ons-color-borders: var(--ons-color-grey-75);
	--ons-color-borders-indent: var(--ons-color-grey-35);
	--ons-color-borders-list: var(--ons-color-grey-35);
	--ons-color-borders-document-image: var(--ons-color-grey-15);
	--ons-color-borders-document-image-focus: var(--ons-color-black);
	--ons-color-placeholder: var(--ons-color-grey-15);
	--ons-color-button: var(--ons-color-leaf-green);
	--ons-color-button-secondary: var(--ons-color-grey-15);
	--ons-color-button-shadow: var(--ons-color-leaf-green-dark-10);
	--ons-color-button-hover: var(--ons-color-leaf-green-dark-5);
	--ons-color-button-secondary-shadow: var(--ons-color-grey-75);
	--ons-color-button-secondary-hover: var(--ons-color-grey-25);
	--ons-color-input-border: var(--ons-color-black);
	--ons-color-input-bg: var(--ons-color-white);
	--ons-color-info: var(--ons-color-ocean-blue);
	--ons-color-info-tint: var(--ons-color-ocean-blue-tint);
	--ons-color-info-vibrant: var(--ons-color-ocean-blue-vibrant);
	--ons-color-success: var(--ons-color-leaf-green);
	--ons-color-success-tint: var(--ons-color-leaf-green-tint);
	--ons-color-success-vibrant: var(--ons-color-leaf-green-vibrant);
	--ons-color-errors: var(--ons-color-ruby-red);
	--ons-color-errors-tint: var(--ons-color-ruby-red-tint);
	--ons-color-errors-vibrant: var(--ons-color-ruby-red-vibrant);
	--ons-color-pending: var(--ons-color-jaffa-orange);
	--ons-color-pending-tint: var(--ons-color-jaffa-orange-tint);
	--ons-color-pending-vibrant: var(--ons-color-jaffa-orange-vibrant);
	--ons-color-dead: var(--ons-color-grey-75);
	--ons-color-instruction: var(--ons-color-jaffa-orange);
	--ons-color-instruction-tint: var(--ons-color-jaffa-orange-tint);
	--ons-color-text-disabled: var(--ons-color-grey-35);
	--ons-color-border-disabled: var(--ons-color-grey-35);
}
</style>

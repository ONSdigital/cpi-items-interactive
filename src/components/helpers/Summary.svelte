<script>
	import { format } from 'd3-format';


export let data;
let total
let totallastmonth
let totallastyear
let pricedifflastyear
let pricedifflastmonth
let maxannualgrowth
let fmt=format(",.2f")

$: if(data.length>0){
    total = data.reduce((acc,cur)=>acc+cur['Average price'],0)
    totallastmonth = data.reduce((acc,cur)=>acc+cur['pricelastmonth'],0)
    totallastyear = data.reduce((acc,cur)=>acc+cur['pricelastyear'],0)
    pricedifflastmonth=total-totallastmonth
    pricedifflastyear=total-totallastyear
    // https://seanconnolly.dev/javascript-find-element-with-max-value
    maxannualgrowth=data.reduce((prev,curr)=>{
        return prev['Annual growth'] > curr['Annual growth'] ? prev : curr
    })

    console.log(pricedifflastyear)
}

function lowercasefirstletter(string) {
    return string.charAt(0).toLowerCase() + string.slice(1);
}
</script>

{#if data.length>5 && total<250}
    <p class='highlight'>
        The total average price for all the items in your basket is £{fmt(total)}. This was £{fmt(pricedifflastmonth)} {pricedifflastmonth>0 ? 'more' :' less'} than last month.
    </p>
{:else}
    <p class='highlight'>
        The total average price for all items in your basket is £{fmt(total)}. This was £{fmt(pricedifflastyear)} {pricedifflastyear>0 ? 'more' :' less'} than last year.
    </p>    
{/if}



<!-- Check items have annual growth in them -->
{#if maxannualgrowth['Annual growth']!=null}
<p>
    Over the last year, {lowercasefirstletter(maxannualgrowth['Name'])} saw the highest {maxannualgrowth['Monthly growth']>0 ? 'increase' : 'decrease'} at {format('.1f')(maxannualgrowth['Annual growth'])}%.
</p>
{/if}

<style>
    p{
        color:#206095;
        font-size: 18px;
    }

    p.highlight{
        font-size: 21px;
        font-weight: 700;
    }
</style>


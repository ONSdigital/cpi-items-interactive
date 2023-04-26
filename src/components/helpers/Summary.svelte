<script>
	import { format,formatDefaultLocale } from 'd3-format';


export let data;
let total
let totallastmonth
let totallastyear
let pricedifflastyear
let pricedifflastmonth
let totalOfItemsAvailableLastMonth
let totalOfItemsAvailableLastyear
let maxannualgrowth
let locale=formatDefaultLocale({
  "decimal": ".",
  "thousands": ",",
  "grouping": [3],
  "currency": ["£", ""]
})
let currency=format("$,.2f")

$: if(data.length>0){
    total = data.reduce((acc,cur)=>acc+cur['Average price'],0)
    totalOfItemsAvailableLastMonth=data.filter(d=>d['pricelastmonth']==null).reduce((acc,cur)=>acc+cur['Average price'],0)
    totalOfItemsAvailableLastyear=data.filter(d=>d['pricelastyear']==null).reduce((acc,cur)=>acc+cur['Average price'],0)
    totallastmonth = data.filter(d=>d['pricelastmonth']>0).reduce((acc,cur)=>acc+cur['pricelastmonth'],0)
    totallastyear = data.filter(d=>d['pricelastyear']>0).reduce((acc,cur)=>acc+cur['pricelastyear'],0)
    pricedifflastmonth=total-totalOfItemsAvailableLastMonth-totallastmonth
    pricedifflastyear=total-totalOfItemsAvailableLastyear-totallastyear
    // https://seanconnolly.dev/javascript-find-element-with-max-value
    maxannualgrowth=data.reduce((prev,curr)=>{
        return prev['Annual growth'] > curr['Annual growth'] ? prev : curr
    })
}

function lowercasefirstletter(string) {
    return string.charAt(0).toLowerCase() + string.slice(1);
}
</script>

{#if data.length>5 && total<250}
    <p class='highlight'>
        The total average price for all the items in your basket is {currency(total)}. Comparing with items available, this was {currency(pricedifflastmonth)} {pricedifflastmonth>0 ? 'more' :' less'} than last month and 
    {currency(pricedifflastyear)} {pricedifflastyear>0 ? 'more' :' less'} than last year.
    </p>
{:else}
    <p class='highlight'>
        The total average price for all items in your basket is {currency(total)}. Comparing with items available, this was {currency(pricedifflastyear)} {pricedifflastyear>0 ? 'more' :' less'} than last year.
    </p>    
{/if}



<!-- Check items have annual growth in them -->
{#if maxannualgrowth['Annual growth']!=null}
<p>
    Over the last year, {lowercasefirstletter(maxannualgrowth['justName'])} saw the largest {maxannualgrowth['Annual growth']>0 ? 'increase' : 'decrease'} at {format('.0f')(maxannualgrowth['Annual growth'])}%.
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


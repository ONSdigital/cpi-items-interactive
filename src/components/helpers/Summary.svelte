<script>
	import { format } from 'd3-format';


export let data;
let total
let totallastmonth
let pricedifflastmonth
let maxannualgrowth
let fmt=format(",.2f")

$: if(data.length>0){
    total = data.reduce((acc,cur)=>acc+cur['Average price'],0)
    totallastmonth = data.reduce((acc,cur)=>acc+cur['pricelastmonth'],0)
    pricedifflastmonth=total-totallastmonth
    // https://seanconnolly.dev/javascript-find-element-with-max-value
    maxannualgrowth=data.reduce((prev,curr)=>{
        return prev['Annual growth'] > curr['Annual growth'] ? prev : curr
    })
}

function lowercasefirstletter(string) {
    return string.charAt(0).toLowerCase() + string.slice(1);
}
</script>



<p class='highlight'>
    The total average price for all the items in your basket is £{fmt(total)}. This was £{fmt(pricedifflastmonth)} {pricedifflastmonth>0 ? 'more' :' less'} than last month.
</p>
<p>
    Over the last year, {lowercasefirstletter(maxannualgrowth['Name'])} saw the highest {maxannualgrowth['Monthly growth']>0 ? 'increase' : 'decrease'} at {format('.1f')(maxannualgrowth['Annual growth'])}%.
</p>

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


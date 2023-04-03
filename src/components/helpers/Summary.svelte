<script>
	import { format } from 'd3-format';


export let data;
let total
let totallastmonth
let pricedifflastmonth
let maxmonthlygrowth
let fmt=format(",.2f")

$: if(data.length>0){
    total = data.reduce((acc,cur)=>acc+cur['Average price'],0)
    totallastmonth = data.reduce((acc,cur)=>acc+cur['pricelastmonth'],0)
    pricedifflastmonth=total-totallastmonth
    // https://seanconnolly.dev/javascript-find-element-with-max-value
    maxmonthlygrowth=data.reduce((prev,curr)=>{
        return prev['Monthly growth'] > curr['Monthly growth'] ? prev : curr
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
    Over the last month, {lowercasefirstletter(maxmonthlygrowth['Name'])} saw the highest {maxmonthlygrowth['Monthly growth']>0 ? 'increase' : 'decrease'} at {format('.1f')(maxmonthlygrowth['Monthly growth'])}%.
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


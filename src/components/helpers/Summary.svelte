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

<div class='background'>
    <div class='title'>
        <h2>Your price changes</h2>
    </div>
    <div id='description' class='vflex dashbottom'>
        <div class='hflex'>
            <p class='bold s16'>Description</p>
            <p class='bold s16'>Price</p>
            
        </div>
        <div class='hflex'>
            <p class='s21'>Basket total</p>
            <p class='bold s21'>{currency(total)}</p>
        </div>

    </div>
    <div id='results' class='dashbottom'>
        <p class='bold s16'>Results</p>
        <div class='hflex'>
            <div class='first-item'>
                {#if data.length>5 && total<250}
                    <p class='s21'>
                        Comparing with items available, this was <span class='bold'>{currency(pricedifflastmonth)} {pricedifflastmonth>0 ? 'more' :' less'} than last month</span> and 
                    <span class='bold'>{currency(pricedifflastyear)} {pricedifflastyear>0 ? 'more' :' less'} than last year.</span>
                    </p>
                {:else}
                    <p class='s21'>
                        Comparing with items available, this was <span class='bold'>{currency(pricedifflastyear)} {pricedifflastyear>0 ? 'more' :' less'} than last year.</span>
                    </p>    
                {/if}
            </div>
                
            <div class='rel last-item'>
                <div id="pound" class='abs'>
                    <img alt="" src="./pound.svg">
                </div>
                <div class='abs' class:less={pricedifflastyear<0} id='arrow'>
                    <img height=14 width=14 alt="" src="./arrow.svg">
                </div>
            </div>
        </div>
        
        
        
        
    </div>
    <div id='category'>
        <p class='bold s16'>{maxannualgrowth['Category1']}</p>
        <div class='hflex'>
            <div class='first-item'>
                <!-- Check items have annual growth in them -->
                {#if maxannualgrowth['Annual growth']!=null}
                <p class='s21'>
                    Over the last year, <span class='bold'>{lowercasefirstletter(maxannualgrowth['justName'])}</span> saw the largest {maxannualgrowth['Annual growth']>0 ? 'increase' : 'decrease'} at <span class='bold'>{format('.0f')(maxannualgrowth['Annual growth'])}%.</span>
                </p>
                {/if}
            </div>
            <div class=last-item>
                <img class="{maxannualgrowth['Category1'].replace(/\s/g, '').toLowerCase()} icon" alt=''/>
            </div>
        </div>
        
        
    </div>
    



    
</div>
<div id='receiptheader'>
    <svg viewBox="0 0 360 6">
        <path d="M 0,6 L 12,0 L 24,6 L 36,0 L 48,6 L 60,0 L 72,6 L 84,0 L 96,6 L 108,0 L 120,6 L 132,0 L 144,6 L 156,0 L 168,6 L 180,0 L 192,6 L 204,0 L 216,6 L 228,0 L 240,6 L 252,0 L 264,6 L 276,0 L 288,6 L 300,0 L 312,6 L 324,0 L 336,6 L 348,0 L 360,6" fill="#E9EFF4"></path>
    </svg>
</div>







<style>

    img.icon{
        width:40px;
        height:40px;
        float:right;
    }

    .foodanddrink{
        content:url("../../../basket.svg");
    }

    .clothingandfootwear{
        content:url("../../../clothing.svg");
    }

    /* .foodanddrinkestablishments{
        content:url("./basket.svg");
    } */

    .health{
        content:url("../../../health.svg");
    }

    .householditems{
        content:url("../../../household.svg");
    }

    .recreationandculture{
        content:url("../../../recreation.svg");
    }

    /* .services{
        content:url("./basket.svg");
    } */

    .transport{
        content:url("../../../transport.svg");
    }

    
    .first-item{
        flex:1 1 auto;
    }

    .last-item{
        flex:0 0 50px;
    }

    .hflex p{
        margin:0;
    }

    .abs{
        position: absolute;
    }

    #pound{
        right:15px;
    }

    #arrow{
        right:0;
    }

    .less{
        transform: rotate(180deg);
    }

    #results,#category{
        position:relative;
    }

    .dashbottom{
        border-bottom: 1px dashed #206095;
    }

    #description{
        padding-top: 10px;
        padding-bottom:25px;
    }

    .hflex{
        margin:5px 0;
        display: flex;
        justify-content: space-between;
    }

    #results .hflex{
        padding-bottom: 25px;
    }

    .vflex{
        display: flex;
        flex-direction: column;
    }

    .title{
        font-size: 16px;
        font-weight: 700;
        /* text-align: center; */
        border-width:0 0 5px 0;
        border-style:solid;
        border-image:url("data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4KPCEtLSBHZW5lcmF0b3I6IEFkb2JlIElsbHVzdHJhdG9yIDI3LjIuMCwgU1ZHIEV4cG9ydCBQbHVnLUluIC4gU1ZHIFZlcnNpb246IDYuMDAgQnVpbGQgMCkgIC0tPgo8c3ZnIHZlcnNpb249IjEuMSIgaWQ9IkxheWVyXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4IgoJIHZpZXdCb3g9IjAgMCAxNi40IDgiIHN0eWxlPSJlbmFibGUtYmFja2dyb3VuZDpuZXcgMCAwIDE2LjQgODsiIHhtbDpzcGFjZT0icHJlc2VydmUiPgo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPgoJLnN0MHtmaWxsLXJ1bGU6ZXZlbm9kZDtjbGlwLXJ1bGU6ZXZlbm9kZDtmaWxsOiMyMDYwOTU7c3Ryb2tlOiMyMDYwOTU7c3Ryb2tlLXdpZHRoOjAuNzA4NztzdHJva2UtbGluZWNhcDpyb3VuZDt9Cjwvc3R5bGU+CjxnIGlkPSJzdmdHcm91cCI+Cgk8cGF0aCB2ZWN0b3ItZWZmZWN0PSJub24tc2NhbGluZy1zdHJva2UiIGNsYXNzPSJzdDAiIGQ9Ik03LjUsMC40aDEuNEw4LjYsMy41bDMuMS0wLjlsMC4yLDEuNGwtMywwLjNsMS45LDIuNUw5LjYsNy41TDguMiw0LjcKCQlMNi45LDcuNUw1LjYsNi44bDEuOS0yLjVMNC41LDMuOWwwLjItMS40bDMsMC45TDcuNSwwLjR6Ii8+CjwvZz4KPC9zdmc+Cg==") 0 0 70 0 space;
    }

    svg{
        transform: rotate(180deg);
    }
    .background{
        background-color: #E9EFF4;
		margin-top: 10px;
		padding: 25px;
    }

    #receiptheader{
        height:14px;
        overflow: hidden;
        position:relative;
    }

    #receiptheader svg{
        position:absolute;
    }

    p{
        color:#206095;
        font-size: 18px;
    }

    .bold{
        font-weight: 700;
    }

    .rel{
        position:relative
    }

    .s16{
        font-size: 16px;
        margin-top:16px;
        margin-bottom:6px;
    }

    .s21{
        font-size:21px;
        margin:0 0 25px 0;
    }

    h2{
        color:#206095;
    }
</style>


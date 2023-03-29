<script>
export let data;
let total

$: if(data){
    total = data.reduce((acc,cur)=>acc+cur['Average price'],0)
}
</script>

<p>
    Total is {total}.
</p>



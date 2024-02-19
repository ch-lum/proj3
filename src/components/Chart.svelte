<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let tempData = [];

    onMount(async () => {
        const res = await fetch('words.csv'); 
        const csv = await res.text();
        tempData = d3.csvParse(csv, d3.autoType)
        console.log(tempData);
    });


</script>

<div class="visualization">
    <svg width="600", height="400">
        <g>
			<!-- Place for Pie -->
		</g>
    </svg>
</div>
<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let tempData = [];
    let svg;

    onMount(async () => {
        const res = await fetch('words.csv'); 
        const csv = await res.text();
        tempData = d3.csvParse(csv, d3.autoType)
        console.log(tempData[0]);

        const margin = { top: 20, right: 20, bottom: 20, left: 20 };
        const width = 1200 - margin.left - margin.right;
        const height = 1200 - margin.top - margin.bottom;


        const dodge = _dodge();

        const xScale = d3.scaleLinear()
            .domain(d3.extent(tempData, d => d['proportion']))
            .range([margin.left, margin.left + width]);

        const radiusScale = d3.scaleSqrt()
            .domain([5, d3.max(tempData, d => d['total'])])
            .range([1, 40]);

        // const yScale = d3.scaleLinear()
        //     .domain([0, d3.max(tempData, d => d.proportion)])
        //     .range([height, 0]);

        const xAxis = d3.axisBottom(xScale);
        // const yAxis = d3.axisLeft(yScale);

        console.log(xAxis);

        const svg = d3.select('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        svg.append('g')
            .attr('transform', `translate(0, ${height})`)
            .call(xAxis);

        // svg.append('g')
        //     .call(yAxis);

        // const simulation = d3.forceSimulation(tempData)
        //     .force("x", d3.forceX().x(d => xScale(d['proportion'])))
        //     .force("y", d3.forceY().y(height/2))
        //     .force("collide", d3.forceCollide().radius(d => radiusScale(d['total']) + 0.5));

        // simulation.tick(10000);

        // svg.append('g')
        //     .selectAll("circle")
        //     .data(tempData)
        //     .enter()
        //     .append('circle')
        //         .attr("cx", (d) => xScale(d['proportion']))
        //         .attr("cy", (d) => d.y + height/2)
        //         .attr("r", (d) => radiusScale(d['total']))
        //         .attr("fill-opacity", 0.4)
            // .append('title')
            //     .text(d => d.data.name);

        svg.append('g')
            .selectAll('circle')
            .data(dodge(tempData, {radius: d => (radiusScale(d['total']) * 2), x: d => xScale(d['proportion'])}))
            .enter()
            .append('circle')
                .attr("cx", d => d.x)
                .attr("cy", d => height - d.y)
                .attr("r", d => (d.radius) / 2)
                .attr("fill-opacity", 0.4)
            .append("title")
                .text(d => d.data.name);

        
    });

    // function _dodge() {return((data, {radius, x}) => {
    //         // console.log(radius(40));
    //         // console.log(x)
    //         // const radius2 = radius ** 2;
    //         const circles = data.map(d => ({x: x(d), radius: radius(d), data: d})).sort((a, b) => a.x - b.x);
    //         const epsilon = 1e-3;
    //         let head = null, tail = null;

    //         function intersects(x, y) {
    //             let a = head;
    //             while (a) {
    //                 if (radius2 - epsilon > (a.x - x) ** 2 + (a.y - y) ** 2) {
    //                     return true;
    //                 }
    //                 a = a.next;
    //             }
    //             return false
    //         }

    //         for (const b of circles) {
    //             while (head && head.x < b.x - radius2) head = head.next;

    //             if (intersects(b.x, b.y = 0)) {
    //                 let a = head;
    //                 b.y = Infinity;
    //                 do {
    //                     let y1 = a.y + Math.sqrt(radius2 - (a.x - b.x) ** 2);
    //                     let y2 = a.y - Math.sqrt(radius2 - (a.x - b.x) ** 2);
    //                     if (Math.abs(y1) < Math.abs(b.y) && !intersects(b.x, y1)) b.y = y1;
    //                     if (Math.abs(y2) < Math.abs(b.y) && !intersects(b.x, y2)) b.y = y2;
    //                     a = a.next;
    //                 } while (a);
    //             }

    //             b.next = null;
    //             if (head === null) head = tail = b;
    //             else tail = tail.next = b;
    //         }

    //         return circles
    //     }    
    // )}

    function _dodge() {
        return ((data, { radius, x }) => {
            // console.log(data.map(d => radius(d)));

            const epsilon = 1e-3;
            let head = null, tail = null;

            const circles = data.map(d => ({ x: x(d), radius: radius(d), data: d })).sort((a, b) => {
                const radiusComparison = b.radius - a.radius;

                return radiusComparison !== 0 ? radiusComparison : b.x - a.x;
            });

            const maxRadius = circles[0].radius;

            function intersects(x, y, r) {
                let a = head;
                while (a) {
                    const ar = (a.radius > r) ? a.radius : r;
                    if (ar ** 2 - epsilon> (a.x - x) ** 2 + (a.y - y) ** 2) {
                        return true;
                    }
                    a = a.next;
                }
                return false;
            }
            
            for (const b of circles) {
                const r = b.radius;

                while (head && (head.x < b.x - maxRadius ** 2 || head.x > b.x + maxRadius ** 2)) head = head.next;

                if (intersects(b.x, b.y = 0, r)) {
                    let a = head;
                    b.y = Infinity;
                    do {
                        const ar = (a.radius > r) ? a.radius : r;
                        
                        const y1 = a.y + Math.sqrt(ar**2 - (a.x - b.x) ** 2);
                        const y2 = a.y - Math.sqrt(ar**2 - (a.x - b.x) ** 2);

                        if (Math.abs(y1) < Math.abs(b.y) && !intersects(b.x, y1, r)) b.y = y1;
                        if (Math.abs(y2) < Math.abs(b.y) && !intersects(b.x, y2, r)) b.y = y2;

                        a = a.next;
                    } while (a);
                }

                b.next = null;
                if (head === null) head = tail = b;
                else tail = tail.next = b;
            }
            return circles;
        });
    }
</script>

<style>
    svg {
      width: 100%;
      height: 100%;
    }
</style>
  
<svg bind:this={svg}></svg>
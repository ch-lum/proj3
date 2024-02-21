<script>
    console.log("reset");

    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let tempData = [];
    let svg;
    let comments = [];

    let hovered = -1;
    let recorded_mouse_position = { x: 0, y: 0 };
    let anyClicked = false;

    const initialOpacity = 0.5;

    onMount(async () => {
        let res = await fetch('words.csv'); 
        let csv = await res.text();
        tempData = d3.csvParse(csv, d3.autoType)
        res = await fetch('comments.csv');
        csv = await res.text();
        comments = d3.csvParse(csv, d3.autoType)

        const allComments = comments.map(d => d.text)
        const posComments = comments.filter(d => d.label === "POSITIVE").map(d => d.text)
        const negComments = comments.filter(d => d.label === "NEGATIVE").map(d => d.text)
        // console.log(posComments.length)
        // console.log(tempData[0]);
        // console.log(comments[0]);

        const margin = { top: 20, right: 20, bottom: 20, left: 20 };
        const width = 1200 - margin.left - margin.right;
        const height = 700 - margin.top - margin.bottom;

        const tooltip = d3.select("#tooltip")
        const negatives = d3.select("#comments-neg")
        const positives = d3.select("#comments-pos")

        const dodge = _dodge();

        const xScale = d3.scaleLinear()
            .domain([0,1])
            .range([margin.left, margin.left + width]);

        const radiusScale = d3.scaleSqrt()
            .domain([10, d3.max(tempData, d => d['total'])])
            .range([2, 40]);

        const colorScale = d3.scaleLinear()
            .domain([xScale(0), xScale(0.5), xScale(1)])
            .range(["#d4322c", "#f9f7ae", "#22964f"]);
        
        // const yScale = d3.scaleLinear()
        //     .domain([0, d3.max(tempData, d => d.proportion)])
        //     .range([height, 0]);

        const xAxis = d3.axisBottom(xScale);
        // const yAxis = d3.axisLeft(yScale);

        // console.log(xAxis);

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
                .attr("cy", d => height / 2 - d.y)
                .attr("r", d => (d.radius) / 2)
                .attr("fill", d => colorScale(d.x))
                .attr("fill-opacity", initialOpacity)
                .attr("stroke", "grey")
                .append("title")
                .text(d => d.data.name)
                
        svg.selectAll("circle")
            .on("mouseenter", handleMouseEnter)
            .on("mouseleave", handleMouseLeave)
            .on("click", handleClick);

        document.body.addEventListener("click", handleBodyClick)
        document.addEventListener("wheel", event => {
            event.preventDefault()
            document.getElementById("comments-neg").scrollTop += event.deltaY
            document.getElementById("comments-pos").scrollTop += event.deltaY
        }, { passive: false })

        function findComments(word) {
            const containsWord = {
                "positive": posComments.filter(d => d.toLowerCase().includes(word)),
                "negative": negComments.filter(d => d.toLowerCase().includes(word))
            }
            console.log(containsWord)
            return containsWord
        }

        function colorSubstrings(original, substring, color) {
            return original.replace(
                new RegExp(substring, 'gi'),
                match => `<span style="color: ${color};">${match}</span>`
            );
        }
        
        function appear(d) {
            // WORD
            
            tooltip.style("opacity", 1)
            
            tooltip.select("#range").text(d.data.word + "\n" + "(" + d.data.proportion.toFixed(2) + ")")
            
            const x = xScale(d.data['proportion']) + margin["left"];
            const y = -d.y + height / 2 + margin["top"]
            
            const yAdjust = (y > 100) ? -2 : 1

            const wordComments = findComments(d.data.word)
            const posSubset = wordComments.positive
            const negSubset = wordComments.negative
            console.log(posSubset)
            

            tooltip.style("transform", `translate(`
                + `calc(-50% + ${x}px),`
                + `calc(${yAdjust} * 100% + ${y}px)`
                + `)`
            )

            tooltip.style("color", colorScale(d.x))

            // NEGATIVE COMMENTS
            document.getElementById("comments-neg").scrollTop = 0
            negatives.style("opacity", 1)
            negatives.style("transform", `translate(`
                + `calc(-150% + ${x}px),`
                + `calc(-50% + ${y}px)`
                + `)`
            )

            let negBody = negSubset.join("\n\n------\n\n")
            negBody = colorSubstrings(negBody, d.data.word, "red")

            negatives.select("#neg-header").text("Negative Comments (" + negSubset.length + ")")
            negatives.select("#neg-body").html(negBody)

            // POSITIVE COMMENTS
            document.getElementById("comments-pos").scrollTop = 0
            positives.style("opacity", 1)
            positives.style("transform", `translate(`
                + `calc(50% + ${x}px),`
                + `calc(-50% + ${y}px)`
                + `)`
            )

            let posBody = posSubset.join("\n\n------\n\n")
            posBody = colorSubstrings(posBody, d.data.word, "green")

            positives.select("#pos-header").text("Positive Comments (" + posSubset.length + ")")
            positives.select("#pos-body").html(posBody)
        }
        
        function handleBodyClick() {
            anyClicked = false

            d3.selectAll("circle")
                .attr('fill-opacity', initialOpacity)
                .attr('clicked', false)
            
            tooltip.style("opacity", 0)
            negatives.style("opacity", 0)
            positives.style("opacity", 0)

        }

        function handleMouseEnter(event, d) {
            if (!anyClicked) {
                d3.select(this).attr('fill-opacity', 1);
                appear(d)
                console.log(d.data.total)
            }
        }
    
        function handleMouseLeave(event, d) {
            if (!anyClicked) {
                d3.select(this).attr('fill-opacity', initialOpacity);
                tooltip.style("opacity", 0)
                tooltip.style("transform", `translate(-1000px, -1000px)`)
                negatives.style("opacity", 0)
                negatives.style("transform", `translate(-1000px, -1000px)`)
                positives.style("opacity", 0)
                positives.style("transform", `translate(-1000px, -1000px)`)
            }

        }
    
        function handleClick(event, d) {
            d.clicked = !d.clicked;

            d3.selectAll("circle").attr('fill-opacity', initialOpacity)
                .attr('clicked', false)
            // Set the opacity based on the "clicked" property
            d3.select(this).attr('fill-opacity', d.clicked ? 1 : initialOpacity);

            if (anyClicked){
                appear(d)
            }

            anyClicked = (d.clicked) ? true : false;
            event.stopPropagation();

        }                
    });
        
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
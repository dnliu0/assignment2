<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    export let variable;
    export let filter;
    export let update;
    export let criteria;
    export let marks;
    export let marks2;
    let brushLayer;
    let brush;

    let margin = {top: 10, right: 80, bottom: 30, left: 60};
    let width = 500;
    let height = 200;
    let chartW = width - margin.left - margin.right;
    let chartH = height - margin.top - margin.bottom;

    
    let xAxis;
    let yAxis;
    
    let tempCriteria = ['Topic alignment',
 'Comprehensive Coverage',
 'Required Attribute',
 'Data Ranges',
 'Patterns',
 'Granularity',
 'Collection Timeframe',
 'Update Frequency',
 'Composition of Data Fields',
 'Sample Data',
 'Methods Used',
 'Good Practices Adherence',
 'Unusable Data',
 'Completeness',
 'Access to the Full Dataset',
 'Suitable Format',
 'High Quality Supplements',
 'Data Provider Communication',
 'Reputable Source',
 'Recommendations',
 'Popular Dataset',
 'Methods Comply with Legal Requirements',
 'Access Terms'];

    brush = d3.brush()
        .extent([[0, 0], [chartW, chartH]])
        .on("brush", brushed)
        .on("end", brushended);        

    function brushed(event) {
        // var extent = brush.extent();
        // if (!xScale.invert) {
        //     var d = xScale.domain(),
        //     r = xScale.range();
        //     extent = extent.map(function(e) { return d[d3.bisect(r, e) - 1]; });
        // }
        if (event && event.selection) {
            filter = [[xScale.invert(event.selection[0][0]), yScale.invert(event.selection[0][1])], 
            [xScale.invert(event.selection[1][0]), yScale.invert(event.selection[1][1])]];
            console.log(filter);
            update();
        }
    }

    function brushended(event) {
        if (event && !event.selection) {
            filter = [];
            update();
        }
       
    }
    
    $: xScale = d3.scaleLinear()
        .range([0, chartW])
        .domain([0, fullData.length]);
    // $: binData = d3.histogram()
    //     .value((d) => d['mentioned'])
    //     .domain(xScale.domain())
        // .thresholds(xScale.ticks(10));
    // $: backgroundBins = binData(fullData);
    // $: bins = binData(data);
    // $: yScale = d3.scaleLinear()
    //     .range([chartH, 0])
    //     .domain([0, d3.max(backgroundBins, (d) => d.length)]);
    $: yScale = d3.scaleLinear().range([chartH, 0]).domain([0, d3.max(fullData, (d) => d.count)])
    $: {	
            d3.select(brushLayer)
                .call(brush);
            d3.select(xAxis)
                .call(d3.axisBottom(xScale));
            d3.select(yAxis)
                .call(d3.axisLeft(yScale));
        }
        $: {
            d3.select(xAxis).selectAll("text").remove()
            d3.select(xAxis)
            .append("text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px")
            .style("font-weight", "bold")
            .style("fill", "black")
            .style("transform", `translate(${chartW / 2}px, ${40}px)`)
            .text("Task Type")
            d3.select(xAxis)
            .append("text").style("font-family", "sans-serif")
            .style("font-size", "11px")
            .style("font-weight", "bold")
            .style("fill", "black")
            .style("transform", `translate(${10}px, ${20}px)`)
            .text("Exploratory")
            d3.select(xAxis)
            .append("text").style("font-family", "sans-serif")
            .style("font-size", "11px")
            .style("font-weight", "bold")
            .style("fill", "black")
            .style("transform", `translate(${(10 + chartW)/2}px, ${20}px)`)
            .text("Rough Confirmatory")
            d3.select(xAxis)
            .append("text").style("font-family", "sans-serif")
            .style("font-size", "11px")
            .style("font-weight", "bold")
            .style("fill", "black")
            .style("transform", `translate(${chartW}px, ${20}px)`)
            .text("Confirmatory")
            d3.select(yAxis)
            .append("text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px")
            .style("font-weight", "bold")
            .style("fill", "black")
            .style("transform", `translate(${-margin.left / 2 + 8}px, ${chartH / 2 -80}px) rotate(-90deg)`)
            .text("Number of Criteria Mentioned")


            d3.select(marks2).selectAll("circle")
            .data(fullData).join("circle")
            .style("fill", "grey")
            .attr("class", "backgroundbar")
            .attr("cx", d => xScale(d.idx))
            .attr("cy", d => yScale(d.count))
            .attr("r", 6);

            d3.select(marks).selectAll("circle")
            .data(data).join("circle")
            .style("fill", "goldenrod")
            .attr("class", "backgroundbar")
            .attr("cx", d => xScale(d.idx))
            .attr("cy", d => yScale(d.count))
            .attr("r", 6);

    //         d3.select(marks).selectAll("circle").enter()
    // .append("circle")
    // .attr("cx", d => xScale(d.idx))
    // .attr("cy", d => yScale(d.count))
    // .attr("r", 0) // Start with radius 0 for animation
    // .style("fill", "goldenrod");

            
            // .join(
            //     enter => enter.append("circle")
            //         .attr("class", "bar")
            //         .attr("cx", d => xScale(d.idx))
            //         .attr("cy", d => yScale(d.count))
            //         .attr("r", 0) // Start radius at 0 for animation
            //         .transition().duration(750)
                    
            //         .attr("r", 6), // Animate to full size
            //     update => update
            //         .transition().duration(750)
            //         .attr("cx", d => xScale(d.idx))
            //         .attr("cy", d => yScale(d.count)),
            //     exit => exit.remove()
            // );
            
    }
    //$: console.log(fullData[0]['mentioned']);
</script>

<main>
    <svg {width} {height}>
        <g style="transform: translate({margin.left}px, {margin.top}px)" 
        bind:this={marks2}></g>
        <g transform="translate({margin.left}, {margin.top})">
            <!-- {#each fullData as d, i}
                <circle class = "backgroundbar"
                    cx={xScale(d.idx)} 
                    cy={yScale(d.count)}
                    r="6"/>
            {/each}
            {#each data as d, i}
                <circle class = "bar"
                    cx={xScale(d.idx)} 
                    cy={yScale(d.count)}
                    r="6"/>
            {/each} -->
            <!-- {#each fullData as d, i}
                <rect class = "backgroundbar"
                    x={xScale(i)} 
                    y={yScale(d.count)}
                    width={xScale(i)-xScale(i-1)}
                    height={chartH - yScale(d.count)}/>
            {/each}
            {#each fullData as d, i}
                <rect class = "bar"
                    x={xScale(i)} 
                    y={yScale(d.count)}
                    width={xScale(i)-xScale(i-1)}
                    height={chartH - yScale(d.count)}/>
            {/each} -->
            
        </g>
        <g style="transform: translate({margin.left}px, {margin.top}px)" 
      bind:this={marks}></g>

        <g transform="translate({margin.left}, {margin.top})"
            bind:this={brushLayer} />
       
        <g transform="translate({margin.left}, {chartH + margin.top})" 
            bind:this={xAxis} />

        <g transform="translate({margin.left}, {margin.top})"
            bind:this={yAxis} />
      </svg>
</main>

<style>
    .bar {
        fill: goldenrod;
        opacity: 1;
        stroke: white;
        stroke-width: 1px;
    }

    .backgroundbar {
        fill: grey;
        opacity: 1;
    }
 </style>
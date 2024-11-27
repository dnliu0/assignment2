<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    export let variable;
    export let filter;
    export let update;
    export let criteria;
    export let brushLayer;
    export let brush;

    let margin = {top: 10, right: 80, bottom: 170, left: 60};
    let width = 800;
    let height = 400;
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

    brush = d3.brushX()
        .extent([[0, 0], [chartW, chartH]])
        .on("brush", brushed)
        .on("end", brushended);        

    function brushed(event) {
        if (event && event.selection) {
            const range = xScale.domain().map(xScale);
            const i0 = d3.bisectRight(range, event.selection[0]);
            const i1 = d3.bisectRight(range, event.selection[1]);
            filter = [i0, i1];
            update();
        }
        // if (event && event.selection) {
            
        //     filter = [xScale.invert(event.selection[0]), xScale.invert(event.selection[1])];
        //     //console.log(filter);
        //     update();
        // }
    }

    function brushended(event) {
        if (event && !event.selection) {
            filter = [];
            update();
        }
       
    }
    function makeData(data) {
        let barData = [];
        tempCriteria.forEach(criterion => {
            let count = 0;
            data.forEach(d => {
                // console.log(criterion.toString())
                // console.log(d.mentioned)
                if (d.mentioned.includes(criterion)) {
                    count += 1;
                }
            })
        barData.push({
                criterion: criterion,
                count: count
            }) ;
            count = 0;
        });
        return barData;
    }
    
  
    $: xScale = d3.scaleBand(tempCriteria, [0, chartW]);
    $: barData = makeData(fullData);
    $: partialBarData = makeData(data);
    // $: console.log(partialBarData)
    // d3.scaleLinear()
    //     .range([0, chartW])
    //     .domain([0, fullData.length]);
    // $: binData = d3.histogram()
    //     .value((d) => d['mentioned'])
    //     .domain(xScale.domain())
        // .thresholds(xScale.ticks(10));
    // $: backgroundBins = binData(fullData);
    // $: bins = binData(data);
    // $: yScale = d3.scaleLinear()
    //     .range([chartH, 0])
    //     .domain([0, d3.max(backgroundBins, (d) => d.length)]);

    $: yScale = d3.scaleLinear().range([chartH, 0]).domain([0, d3.max(barData, (d) => d.count)])
    $: {	
            d3.select(brushLayer)
                .call(brush);
            d3.select(xAxis)
                .call(d3.axisBottom(xScale)).selectAll("text")
                .attr("transform", "rotate(-45)")
                .style("text-anchor", "end");
            d3.select(yAxis)
                .call(d3.axisLeft(yScale));
        }
    //$: console.log(fullData[0]['mentioned']);
  
</script>

<main>
    <svg {width} {height}>
        <g transform="translate({margin.left}, {margin.top})">
            {#each barData as d, i}
                <rect class = "backgroundbar"
                    x={7 + xScale(d.criterion)} 
                    y={yScale(d.count)}
                    width={24}
                    height={chartH - yScale(d.count)}/>
            {/each}
            <!-- {console.log(partialBarData)} -->
            {#each partialBarData as d, i}
                <rect class = "bar"
                    x={7 + xScale(d.criterion)} 
                    y={yScale(d.count)}
                    width={24}
                    height={chartH - yScale(d.count)}/>
            {/each}
            <!-- {#each fullData as d, i}
                <rect class = "bar"
                    x={xScale(i)} 
                    y={yScale(d.count)}
                    width={xScale(i)-xScale(i-1)}
                    height={chartH - yScale(d.count)}/>
            {/each} -->
            
        </g>

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
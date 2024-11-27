<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    export let variable;
    export let filter;
    export let update;
    export let criteria;

    let margin = {top: 10, right: 80, bottom: 30, left: 60};
    let width = 500;
    let height = 200;
    let chartW = width - margin.left - margin.right;
    let chartH = height - margin.top - margin.bottom;

    let brushLayer;
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

    const brush = d3.brushX()
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
            
            filter = [xScale.invert(event.selection[0]), xScale.invert(event.selection[1])];
            //console.log(filter);
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
        .domain([0, d3.max(fullData, (d) => d.count)]);
    $: binData = d3.histogram()
        .value((d) => d.count)
        .domain(xScale.domain())
        .thresholds(xScale.ticks(13));
    $: backgroundBins = binData(fullData);
    // $: bins = binData(data);
    // $: yScale = d3.scaleLinear()
    //     .range([chartH, 0])
    //     .domain([0, d3.max(backgroundBins, (d) => d.length)]);
    $: yScale = d3.scaleLinear().range([chartH, 0]).domain([0, 12])
    $: {	
            // d3.select(brushLayer)
            //     .call(brush);
            d3.select(xAxis)
                .call(d3.axisBottom(xScale));
            d3.select(yAxis)
                .call(d3.axisLeft(yScale));
        }
    //$: console.log(fullData[0]['mentioned']);
</script>

<main>
    <svg {width} {height}>
        <g transform="translate({margin.left}, {margin.top})">
            <!-- {#each fullData as d, i}
                <circle class = "bar"
                    cx={xScale(i)} 
                    cy={yScale(d.count)}
                    r="6"/>
            {/each}
            {#each fullData as d, i}
                <circle class = "backgroundbar"
                    cx={xScale(i)} 
                    cy={yScale(d.count)}
                    r="6"/>
            {/each} -->
            {#each backgroundBins as d, i}
                <rect class = "backgroundbar"
                x={xScale(d.x0)} 
                y={yScale(d.length)}
                width={xScale(d.x1) - xScale(d.x0)}
                height={chartH - yScale(d.length)}/>
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
        opacity: 0.7;
        stroke: white;
        stroke-width: 1px;
    }

    .backgroundbar {
        fill: grey;
        opacity: 0.7;
    }
 </style>
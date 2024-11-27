<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    export let variable;
    export let filter;
    export let update;

    let margin = {top: 10, right: 80, bottom: 30, left: 60};
    let width = 500;
    let height = 200;
    let chartW = width - margin.left - margin.right;
    let chartH = height - margin.top - margin.bottom;

    let brushLayer;
    let xAxis;
    let yAxis;

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
        
        if (event && event.selection) {
        // If xScale is ordinal, handle the mapping of brush selection to domain values
        if (!xScale.invert) {
            const domain = xScale.domain();
            const range = xScale.range();
            
            // Convert the selection into indices for the domain array based on the range
            const startIndex = d3.bisectLeft(range, event.selection[0]);
            const endIndex = d3.bisectRight(range, event.selection[1]);
            
            // Ensure indices are within the valid domain range
            const startValue = domain[Math.max(0, startIndex)];
            const endValue = domain[Math.min(domain.length - 1, endIndex - 1)];
            
            filter = [startValue, endValue];
        } else {
            // For continuous scales, use invert as usual
            filter = [xScale.invert(event.selection[0]), xScale.invert(event.selection[1])];
        }
        update(); // Call update with the new filter range
    }
        // xScale.domain(brush.empty() ? d : extent);
    }

    function brushended(event) {
        if (event && !event.selection) {
            filter = [];
            update();
        }
       
    }

    $: xScale = d3.scaleLinear()
        .range([0, chartW])
        .domain(d3.extent(fullData.map((d) => d.properties[variable])));
    $: binData = d3.histogram()
        .value((d) => d.properties[variable])
        .domain(xScale.domain())
        .thresholds(xScale.ticks(10));
    $: backgroundBins = binData(fullData);
    $: bins = binData(data);
    $: yScale = d3.scaleLinear()
        .range([chartH, 0])
        .domain([0, d3.max(backgroundBins, (d) => d.length)]);
    $: {	
            d3.select(brushLayer)
                .call(brush);
            d3.select(xAxis)
                .call(d3.axisBottom(xScale));
            d3.select(yAxis)
                .call(d3.axisLeft(yScale));
        }
</script>

<main>
    <svg {width} {height}>
        <g transform="translate({margin.left}, {margin.top})">
            {#each backgroundBins as d}
                <rect class = "backgroundbar"
                    x={xScale(d.x0)} 
                    y={yScale(d.length)}
                    width={xScale(d.x1) - xScale(d.x0)}
                    height={chartH - yScale(d.length)}/>
            {/each}
            {#each bins as d}
                <rect class = "bar"
                    x={xScale(d.x0)} 
                    y={yScale(d.length)}
                    width={xScale(d.x1) - xScale(d.x0)}
                    height={chartH - yScale(d.length)}/>
            {/each}
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
        stroke: white;
        stroke-width: 1px;
    }

    .backgroundbar {
        fill: grey;
    }
 </style>
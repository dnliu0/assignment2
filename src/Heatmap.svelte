<script>
    import * as d3 from 'd3';

    export let data;
    export let fullData;
    // export let filter;
    // export let update;

    let width = 700;
    let height = 500;
    var margin = {top: 30, right: 60, bottom: 30, left: 150},
        chartW = width - margin.left - margin.right,
        chartH = height - margin.top - margin.bottom;
    let brushLayer;
    let xAxis;
    let yAxis;
    let uniqueAspects = ['Data Meet Purpose', 'Data Quality Check', 'Pay Access Decision'];
    let uniqueInfoUsed = ['Source reputation', 'User reviews', 'Metadata', 'Data collection conditions', 'Prevalence of missing data', 'Ability to access full datasets', 'Sample Data', 'Visualizations'];
    $: heatmapData = [];
    $: combineMap = new Map();
    $: fullData.forEach(d => {
        uniqueAspects.forEach(a => {
            uniqueInfoUsed.forEach(i => {
            if (d['aspects'].includes(a) && d['Information Used'].includes(i)) {
                let key = `${a}-${i}`;
                if (!combineMap.has(key)) {
                    combineMap.set(key, 0);
                    heatmapData.push({ aspect: a, info: i, count: 0 });
                } else {
                    combineMap.set(key, combineMap.get(key) + 1);
                    let entry = heatmapData.find(entry => entry.aspect === a && entry.info === i);
                    if (entry) {
                        entry.count++; 
                    }
                }
            }
            })
        })
    });

    $: xScale = d3.scaleBand()
        .range([0, chartW])
        .domain(uniqueAspects);
    $: yScale = d3.scaleBand()
        .range([chartH, 0])
        .domain(uniqueInfoUsed);
    
    // $: binData = d3.svg.

    // $: binData = d3.histogram()
    //     .value((d) => d)
    //     .domain(xScale.domain())
    //     .thresholds(xScale.ticks(10));
    // $: backgroundBins = binData(fullData);
    // $: bins = binData(data);
    
    
    $: scale = d3.scaleSequential(d3.interpolatePiYG).domain([d3.min(heatmapData, (d) => d.count), 
    d3.max(heatmapData, (d) => d.count)]);
    $: console.log(heatmapData)
    
    $: {	
            // d3.select(brushLayer)
            //     .call(brush);
            d3.select(xAxis)
                .call(d3.axisBottom(xScale));
            d3.select(yAxis)
                .call(d3.axisLeft(yScale));
        }
    $: console.log(heatmapData[1])
    // $: console.log(scale(heatmapData[0].count))
</script>

<main>
    <svg {width} {height}>
        <g transform="translate({margin.left}, {margin.top})">
            {#each heatmapData as row}
            {console.log(row.count)}
                <rect
                    x={xScale(row.aspect)} 
                    y={yScale(row.info)}
                    fill={scale(row.count)}
                    width={xScale.bandwidth()}
                    height={yScale.bandwidth()}/>
            {/each}
            <!-- {#each bins as d}
                <rect class = "bar"
                    x={xScale(d.x0)} 
                    y={yScale(d.length)}
                    width={xScale(d.x1) - xScale(d.x0)}
                    height={chartH - yScale(d.length)}/>
            {/each} -->
        </g>

        <!-- <g transform="translate({margin.left}, {margin.top})"
            bind:this={brushLayer} /> -->
       
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
<script>
    import * as d3 from 'd3';
    import {legendColor} from 'd3-svg-legend';
    export let data;
    export let fullData;
    export let variable;
    export let new_cuisine_list;
    let width = 600;
    let height = 400;

    let proj = d3.geoMercator().scale(40000).center([-73.94, 40.70]).translate([width/2, height/2]);
    let path = d3.geoPath().projection(proj);
    $: uniqueValues = Array.from(new Set(data.map(d => d.properties["max_cuisine"])));
    console.log(uniqueValues);
    $: scale = variable === "max_cuisine" 
    ? d3.scaleOrdinal()
        .domain(['American',
                      'Chinese',
                      'Pizza',
                      'Coffee/Tea',
                      'Mexican',
                      'Latin American',
                      'Bakery Products/Desserts',
                      'Japanese',
                      'Italian',
                      'Caribbean']) 
        .range(d3.schemeTableau10) 
    : d3.scaleSequential(d3.interpolatePiYG)
        .domain([
            d3.min(data, (d) => +d.properties[variable]), 
            d3.max(data, (d) => +d.properties[variable])
        ]);

    $: scale2 = d3.scaleSequential(d3.interpolatePiYG)
        .domain([
            d3.min(data, (d) => +d.properties[variable]), 
            d3.max(data, (d) => +d.properties[variable])
        ]);
        $: scale3 = d3.scaleOrdinal()
        .domain(uniqueValues) 
        .range(d3.schemeTableau10) 
    // d3.scaleSequential(d3.interpolatePiYG)
    //     .domain([d3.min(data.map((d) => +d.properties.max_animal_per_zip.max_num)), d3.median(data.map((d) => +d.properties.max_animal_per_zip.max_num)), d3.max(data.map((d) => +d.properties.max_animal_per_zip.max_num))]);
    
    // d3.scaleSequential(data.map((d) => d.properties.diversity_count), d3.interpolatePiYG);
    let legend;
    $: {
        const colorLengend = legendColor()
        .shape('rect')
        .cells(7)
        .scale(scale)
        d3.select(legend)
        .call(colorLengend);
    }

    // let legend2;
    // $: {
    //     const colorLengend = legendColor()
    //     .shape('rect')
    //     .cells(9)
    //     .scale(scale)
    //     d3.select(legend2)
    //     .call(colorLengend);
    // }
</script>

<main>
    <!-- <svg {width}{height}>
        {#each data as d}
          console.log(d);
        
          <path d={path(d)} style="fill: {scale(d.properties.max_cuisine_per_zip.cuisine)}"/>
        {/each}
        <g transform="translate({width - 140}, 50)" bind:this={legend}/>
      </svg> -->
      <svg {width}{height}>
        {#each data as d}
          <!-- <path d={path(d)}/> -->
          <path d={path(d)} style="fill: {scale(d.properties[variable])};" />
        {/each}
        {#each fullData as d}
          <path class = "geooverlay"
            d={path(d)}/>
        {/each}
        <g transform="translate({width - 100}, 50)" bind:this={legend}/>
        
      </svg>
</main>

<style>
  .geooverlay {
      stroke: grey;
      stroke-width: 1px;
      fill: none;
    }
</style>
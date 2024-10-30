<script>
  import * as d3 from 'd3';
	import {onMount} from 'svelte';
  
  import Map from './Map.svelte';
  import Histogram from './Histogram.svelte';

	let data = [];
  let fullData = [];
  let filter1 = [];
  let filter2 = [];
  let cuisine_list = ['American',
                      'Chinese',
                      'Pizza',
                      'Coffee/Tea',
                      'Mexican',
                      'Latin American',
                      'Bakery Products/Desserts',
                      'Japanese',
                      'Italian',
                      'Caribbean']
  // let var1 = "percentWhite";
  let var1 = "diversity";
  let var3 = 'max_cuisine';
  let var2 = 'count';

	onMount(async function() {
    // load data from csv (source: https://chicagohealthatlas.org/download)
    // let table = d3.csv('chi-health-data.csv', (d) => ({
    //       ...d,
    //       'Name': d['Name'].substring(6),
    //       'EKW_2022': +d['EKW_2022'],
    //       'PCT-W_2018-2022': +d['PCT-W_2018-2022'],
    //       'VAC_2018-2022': +d['VAC_2018-2022']
    //     }));
    
    //load data from 
    let table = d3.csv('latest.csv', (d) => ({
          ...d,
          'ZIPCODE': d['ZIPCODE'].toString().slice(0, 5),
        }));

    // let geocoord = d3.json('census-tracts.geojson')
    //   .then((d) => d.features);
    // https://www.kaggle.com/datasets/saidakbarp/nyc-zipcode-geodata?resource=download
    let geocoord = d3.json('zip-code.geojson')
      .then((d) => d.features);
    
    await Promise.all([table, geocoord]).then((values) => {
      // console.log(values);
      let table = values[0];
      let modified = table;
      let geocoord = values[1];
      for (let k = 0; k < table.length; k++) {
        if (!cuisine_list.includes(table[k]['CUISINE DESCRIPTION'])) {
          modified[k]['CUISINE DESCRIPTION'] = 'Others'
        }
        if (table[k]['CUISINE DESCRIPTION'] == 'Bakery Products/Desserts') {
          modified[k]['CUISINE DESCRIPTION'] = 'Desserts'
        }
      }
      let max_cuisine = countByCuisine(modified);
      let diversity = getDiversity(table);
      let count = getCount(table);
      // console.log(max_animal_per_zip);
      // join the variables we want to show on the map
      for (let i = 0; i < geocoord.length; i++) {
        let zip = geocoord[i].properties.modzcta;
        let pop = geocoord[i].properties.pop_est;
        pop += pop;
        // console.log(boro)
        let found = false;
        let j = 0;
        while (!found && table.length > j) {
          if (table[j]['ZIPCODE'] == zip) {
            found = true;
            data.push(geocoord[i]);
            data[data.length - 1].properties['max_cuisine'] = max_cuisine[zip].cuisine;
            data[data.length - 1].properties['count'] = count[zip];
            console.log(count)
            data[data.length - 1].properties['diversity'] = diversity[zip];
          } else {
            j++;
          }
        }
      }
      console.log(data);
      fullData = [...data];
    });
	});
  function getDiversity(table){
    let zip_count = {};
    for (let k = 0; k < table.length; k++) {
      let curr_zip = table[k]['ZIPCODE'];
      let cuisine = table[k]['CUISINE DESCRIPTION'];
      if (!zip_count[curr_zip]) {
            zip_count[curr_zip] = {};
      }
          if (zip_count[curr_zip][cuisine]) {
            zip_count[curr_zip][cuisine] += 1;
          } else {
            zip_count[curr_zip][cuisine] = 1;
          }
              
    }
    let diversity_count = {};
      
      for (let z in zip_count) {
          diversity_count[z] = Object.keys(zip_count[z]).length;
      }
      return diversity_count;  
  }
  function getCount(table){
    let zip_count = {};
        for (let k = 0; k < table.length; k++) {
          let curr_zip = table[k]['ZIPCODE'];
          if (zip_count[curr_zip]) {
            zip_count[curr_zip] += 1;
            
          } else {
            zip_count[curr_zip] = 1;
          }
        }
  
      return zip_count;
  
}

  function countByCuisine(table){
    let zip_count = {};
        for (let k = 0; k < table.length; k++) {
          let curr_zip = table[k]['ZIPCODE'];
          let cuisine = table[k]['CUISINE DESCRIPTION'];
          
          if (!zip_count[curr_zip]) {
            zip_count[curr_zip] = {};
          }
          if (zip_count[curr_zip][cuisine]) {
            zip_count[curr_zip][cuisine] += 1;
            
          } else {
            zip_count[curr_zip][cuisine] = 1;
          }
        }
        let max_cuisine_per_zip = {};
        for (let z in zip_count) {
          let cuisine_types = zip_count[z];
          let max_cuisine;
          let max_num = -1;
          for (let a in cuisine_types) {
              if (cuisine_types[a] > max_num) {
                  max_cuisine = a;
                  max_num = cuisine_types[a];
              }
          }
          max_cuisine_per_zip[z] = {
              cuisine: max_cuisine,
              count: max_num
          };
      }
      return max_cuisine_per_zip;
  
}

  function countByAnimalType(table){
    let zip_count = {};
        for (let k = 0; k < table.length; k++) {
          let curr_zip = table[k]['Incident Zip'];
          let animal = table[k]['Descriptor'];
          
          if (!zip_count[curr_zip]) {
            zip_count[curr_zip] = {};
          }
          if (zip_count[curr_zip][animal]) {
            zip_count[curr_zip][animal] += 1;
            
          } else {
            zip_count[curr_zip][animal] = 1;
          }
        }
        let max_animal_per_zip = {};
        for (let z in zip_count) {
          let animal_types = zip_count[z];
          let max_animal;
          let max_num = -1;
          for (let a in animal_types) {
              if (animal_types[a] > max_num) {
                  max_animal = a;
                  max_num = animal_types[a];
              }
          }
          max_animal_per_zip[z] = {
              animal: max_animal,
              count: max_num
          };
      }
      return max_animal_per_zip;
  }

  function updateData(){
    if (filter1.length > 0 && filter2.length > 0) {
      data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1] && d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1]));
    } else if (filter1.length > 0 && filter2.length == 0) {
      data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1]));
    } else if (filter1.length == 0 && filter2.length > 0) {
      data = fullData.filter((d) => (d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1]));
    } else {
      data = [...fullData];
    }
  }
  
</script>

<main>
  <h1>NYC Restaurants</h1>

  <div class="flex-container row">
    <div class="map"><Map data={data} fullData={fullData} variable={var1}></Map></div>
    <div class="map"><Map data={data} fullData={fullData} variable={var3}></Map></div>
    <div class="flex-container col">
      <div class="hist"><Histogram data={data} fullData={fullData} variable={var1} bind:filter={filter1} update={updateData}></Histogram></div>
      <div class="hist"><Histogram data={data} fullData={fullData} variable={var2} bind:filter={filter2} update={updateData}></Histogram></div>
    </div>
  </div>
</main>

<style>
  
  /* .flex-container {
    display: flex;
    justify-content: center;  
    height: 100%;
    padding: 15px;
    gap: 5px;
  }

  .flex-container > div{
    padding: 8px;
  }

  .flex-container .row {
    flex-direction: row;  
  }

  .flex-container .col {
    flex-direction: column;  
  }

  .map { 
    flex-grow:1;
  }
			
  .hist { 
    flex-grow:0;
  } */
</style>
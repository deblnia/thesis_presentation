<script>
	import { setContext, onMount } from "svelte";
	import { getData, setColors } from "./utils.js";
	import { themes, regions, colors, datakeys } from './config.js';
	import { ScatterChart } from "@onsvisual/svelte-charts";
	import ONSHeader from "./ONSHeader.svelte";
	import ONSFooter from "./ONSFooter.svelte";
	import Header from "./Header.svelte";
	import Section from "./Section.svelte";
	import Media from "./Media.svelte";
	import Scroller from "./Scroller.svelte";
	import Filler from "./Filler.svelte";
	import Divider from "./Divider.svelte";
	import Map from "./Map.svelte";

	// STYLE CONFIG
	// Set theme globally (options are 'light' or 'dark')
	let theme = "light";
	setContext("theme", theme);
	setColors(themes, theme);

	// SCROLLYTELLING CONFIG
	// Config
	const threshold = 0.65;
	// State
	let index = [];
	let indexPrev = [];
	onMount(() => {
		indexPrev = [...index];
	});

	// CHART CODE
	// Config
	const categories = regions.map((d) => d.code);
	const url = "https://gist.githubusercontent.com/deblnia/5e5b9fc16bda9bb679ffbe3fb6354f41/raw/16d32e9ceeb93989a025e03b96b316cd47a6faa9/genesweep_for_viz.csv";
	const catKey = "region_code";
	// State
	let data;
	let places;
	let selected;
	let xKey = "income";
	let yKey = "employment";

	getData(url)
		.then((result) => (data = result))
		.then(() => {
			// console.log(data.slice(5)); 
			// data = data.flat(); 
			// console.log(data.slice(5)); 
			let codes = [];
			let array = [[], [], []];
			data.forEach((d) => {
				// console.log(d); 
				if(data.period === 1){
					array[0].push(d)
				}
				else if (data.period === 2){
					array[1].push(d)
				}
				else{
					array[2].push(d)
				}
		});
		console.log(data.length); 
		console.log(array); 
		array.sort((a, b) => a.name.localeCompare(b.name));
		places = array;
	});
	
	// Actions for CHART scroller
	const chartActions = [
		() => {
			xKey = "income";
			yKey = "employment";
		},
		() => {
			xKey = "income";
			yKey = "health";
		},
		() => {
			xKey = "housing";
			yKey = "health";
		}
	];
	
	// Reactive code to trigger CHART actions
	$: if (index[0] != indexPrev[0]) {
		if (chartActions[+index[0]]) {
			chartActions[+index[0]]();
		}
		indexPrev[0] = index[0];
	}
	
	// MAP CODE
	// Config
	const mapstyle = 'https://bothness.github.io/ons-basemaps/data/style-omt.json';
	const mapbounds = {
		ew: [[-74.634644, 40.398145], [-72.420899, 41.115904]],
		fareham: [[-73.468547, 40.858060], [-73.431984, 40.880777]]
	};
	// State
	let map = null;

	// Actions for MAP scroller
	const mapActions = [
		() => { map.fitBounds(mapbounds.ew) },
		() => { map.fitBounds(mapbounds.fareham) }
		// () => { map.fitBounds(mapbounds.newport) }
	];
	
	// Reactive code to trigger MAP actions
	$: if (map && index[1] != indexPrev[1]) {
		if (mapActions[+index[1]]) {
			mapActions[+index[1]]();
		}
		indexPrev[1] = index[1];
	}

</script>

<!-- <ONSHeader filled={true} /> -->

<Header bgimage="./img/bg-dark.jpg" bgfixed={false} theme="dark">
	<h1 class="text-shadow">Genesweep</h1>
	<p class="inset-medium text-big text-shadow">
		A Short Exploration of How Scientists' Guess
	</p>
	<div class="text-shadow" style="margin-top: 48px;">
		Scroll to begin<br />
		<img src="./img/scroll-down-white.svg" class="svg-icon bounce" alt="down arrow"/>
	</div>
</Header>

<Divider /> 

<Section>
	<h2>This story revolves around one lab.</h2>
	<p class="mb">
		It's a biology lab. Picture test tubes, microscopes, that kind of thing. 
	</p>
</Section>

<Scroller {threshold} bind:index={index[1]}>
	<div slot="background">
		<figure>
			<div class="col-full height-full">
				<Map style={mapstyle} bind:map={map} />
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section>
			<div class="col-medium">
				<p>But it's not just <span class="em em-muted"> any old lab</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>See, <span class="em em-muted">Cold Springs Harbor Laboratory</span> was founded by James Watson (who got half the Nobel Prize for discovering the structure of DNA, and then became racist).</p>
			</div>
		</section>
	</div>
</Scroller>

<Divider />

<Filler theme="dark">
	<p class="text-big">
		It's a prestigious lab. 
	</p>
</Filler>

<Section>
	<h2>And because of its prestige, Cold Springs Harbor Laboratory gets to be involved in all the important stuff in biology.</h2>
	<p>
		Importantly, for this project, that includes the Human Genome Project
		<details class="footnote">
			<summary> [1]</summary>
			<div class="footnote-content">This project was a pretty big deal, almost to the point of being a thesis in itself. If successful, the researchers thought, the finding could usher in an era of personalized medicine. The government, which was competing with private firms at the time, gave the project almost $5 billion, making it one of the largest and best funded scientific enterprises to date. </div>
		  </details>. From 1999 to 2002, CSHL played host to the National Genomic Conferences, where the best and brightest in the field would come and talk about the state of sequencing the human genome. 
	</p>
</Section>

<Media
	col="wide"
	grid="narrow"
	caption="Pictures from the National Genomic Conferences in 2000, curtesy of the CSHL archives.">
	<div class="media"><img src="./img/am1.jpg" alt="Girl in a jacket"></div>
	<div class="media"><img src="./img/am2.jpg" alt="Girl in a jacket"></div>
	<div class="media"><img src="./img/am3.jpg" alt="Girl in a jacket"></div>
	<div class="media"><img src="./img/am4.jpg" alt="Girl in a jacket"></div>
</Media>

<Divider />

<Section>
	<h2>But importantly for our story, there's also a bar at CSHL.</h2>
	<p>
		And people play games in bars <details class="footnote">
			<summary> [2]</summary>
			<div class="footnote-content"> The bar at CSHL is called The Eagle, evoking the publichouse in Cambridge by the same name. According to legend, Francis Crick and James Watson interrupted lunch at the Eagle in Cambridge on Feburary 28th, 1953 to announce their proposal for the structure of DNA.</div>
		  </details>. 
	</p>
</Section>

<Divider />
<br>
<br> 
<br>
<br>
<br>
<br> 
<!-- <br>
<br>
<br>  -->

<Media col="full" height={500}>
	<div class="media"><img src="./img/bar.jpg" alt="The Bar at CSHL"></div>
</Media>

<!-- <Divider /> -->

<br>
<br>
<br>
<br>
<br>
<br>
<br> 

<Section>
	<h2>Bar games are interesting, especially when groups of scientists play them.</h2>
	<p>
		There have been a couple of <a href="https://en.wikipedia.org/wiki/Scientific_wager" target="_blank"> high-profile scientific wagers </a>, but in this instance, because we have several data points, we can leverage  <a href="https://en.wikipedia.org/wiki/The_Wisdom_of_Crowds" target="_blank"> laws of central tendancy </a> to get a sense of how the scientific community (as whole)'s expectations changed throughout the project. 
	</p>
	<blockquote class="text-indent">
		"Dr. Dear was asked how he had predicted such a low number three years ago when numbers around 50,000 were popular. It was late at night, he said, and he had been drinking in a bar. Second, at that hour, people's behavior did not seem so different from that of fruit flies, then known to have 13,500 genes, implying that twice that number would be ample for humans. Third, his birth date, April 27, 1962, immediately suggested 27,462 as the correct number."&mdash;The New York Times
	</blockquote>
	<p>
		And those expectations are important! In case you haven't noticed, we are definitely not living in an era of genetically personalized medicine. And that's largely because scientist's still don't have definitive consensus about how many genes are in the human genome, or rather, 
	</p>
	<p>
		Below is a grid that could contain charts or any other kind of visual media.
		The grid can fit in a standard, wide or full-width column, and the media
		width itself can be narrow (min 200px), medium (min 300px), wide (min 500px)
		or full-width. The grid is responsive, and will re-flow on smaller screens. 
	</p>
</Section>

<!-- <Media
	col="medium"
	grid="medium"
	caption='This is an optional caption for the above media. It can contain HTML code and <a href="#">hyperlinks</a>, and wrap onto multiple lines.'>
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
</Media> -->

<Section>
	<h2>This is a dynamic chart section</h2>
	<p>
		The chart below will respond to the captions as you scroll down. 
		The mode is set to splitscreen, with captions on the left on larger screens.
	</p>
</Section>

<Scroller {threshold} bind:index={index[0]} splitscreen={true}>
	<div slot="background">
		<figure>
			<div class="col-wide height-full middle">
				{#if data && xKey && yKey}
				<div class="chart">
					<ScatterChart diameter={3} {data} {xKey} {yKey} {catKey} {colors} {categories} />
				</div>
				{/if}
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section>
			<div class="col-medium">
				<p>There is a strong correlation between the IMD mesasures for <span class="em em-muted">income</span> 
					and <span class="em em-muted">employment</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>There is a strong correlation between the IMD mesasures for <span class="em em-muted">income</span> and 
					<span class="em em-muted">health</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>There is a weak correlation between the IMD mesasures for <span class="em em-muted">housing</span> and 
					<span class="em em-muted">health</span>.</p>
			</div>
		</section>
		<!-- <section>
			<div class="col-medium">
				<h3>Explore the data</h3>
				{#if data}
				<nobr>
					<span class="label-block">X Axis</span>
					<select bind:value={xKey}>
						{#each Object.keys(datakeys) as key}
							<option value={key}>{datakeys[key]}</option>
						{/each}
					</select>
				</nobr>
				<nobr>
					<span class="label-block">Y Axis</span>
					<select bind:value={yKey}>
						{#each Object.keys(datakeys) as key}
							<option value={key}>{datakeys[key]}</option>
						{/each}
					</select>
				</nobr>
				{#if places}
				<nobr>
						<span class="label-block">District</span>
						<!-- svelte-ignore a11y-no-onchange -->
						<!-- <select bind:value={selected}>
							<option value={null}>All</option>
							{#each places as place}
								<option value={{ value: place.code, col: 'lad_code' }}>
									{place.name}
								</option>
							{/each}
						</select> -->
					<!-- </nobr> -->
				<!-- {/if} -->
				<!-- {/if} -->
			<!-- </div> -->
		<!-- </section> --> 
	<!-- </div> -->
</Scroller>

<Divider />


<Section>
	<p>
		I had combined this data with data from the web on the scientists' individual publications and research keywords, with the hopes of showing some sort of between-group variance among these elite scientists. There's a lot of literature that would suggest that there is a difference there. 
	</p>
	<br> 
	<p> 
		But I haven't found anything too convincing yet.
	</p>
	<br> 
	<p> 
		Thanks for reading anyway. 
	</p> 
</Section>

<ONSFooter />

<style>
	/* Styles specific to elements within the demo */
	.svg-icon {
		width: 48px;
		height: 48px;
	}
	.bounce {
		animation-duration: 2s;
		animation-iteration-count: infinite;
		animation-name: bounce;
		animation-timing-function: ease;
	}
	@keyframes bounce {
		0%   { transform: translateY(10px); }
		30%  { transform: translateY(-10px); }
		50%  { transform: translateY(10px); }
		100% { transform: translateY(10px); }
	}
	/* .label-block {
		display: inline-block;
		text-align: right;
		width: 80px;
	} */
	select {
		width: 210px;
	}
	.chart {
		margin-top: 45px;
		height: 100vh;
		width: calc(100% - 5px);
	}
	/* The properties below make the media DIVs grey, for visual purposes in demo */
	.media {
	  background-color: #f0f0f0;
	  display: -webkit-box;
	  display: -ms-flexbox;
	  display: flex;
	  -webkit-box-orient: vertical;
	  	-webkit-box-direction: normal;
	      -ms-flex-flow: column;
	          flex-flow: column;
	  -webkit-box-pack: center;
	      -ms-flex-pack: center;
	          justify-content: center;
	  text-align: center;
	  color: #aaa;
  }
  details.footnote {
          display: inline;
          position: relative;
      }

      /* hide details arrow */
    details.footnote > summary { list-style: none; } /* firefox */
      details.footnote > summary::-webkit-details-marker { display: none; } /* webkit */

    details.footnote > summary {
          color: #aaa;
          cursor: pointer;
      }
      
    details.footnote > .footnote-content {
          position: absolute;
          background: white;
          border: 1px solid lightgray;
          padding: 1em;
          min-width: 40ch;
      }
</style>
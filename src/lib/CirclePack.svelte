<script>
  import data from "../data/data.json";
  import * as d3 from "d3";

  const length = 800;

  //legend data (if desire) and color scale
  const legendData = [
    { category: "Critically Endangered", color: "#b30000" },
    { category: "Endangered", color: "#e34a33" },
    { category: "Vulnerable", color: "#fc8d59" },
    { category: "Near Threatened", color: "#fdbb84" },
    { category: "Least Concern", color: "#fdd49e" },
    { category: "Data Deficient", color: "#d9d9d9" },
  ];
  const categories = legendData.map((d) => d.category);
  const colors = legendData.map((d) => d.color);
  const colorScale = d3.scaleOrdinal().domain(categories).range(colors);

  //convert data to hierarchical format
  const groupingFn = [(d) => d.redlistCategory, (d) => d.scientificName];
  const rollupData = d3.rollup(data, (v) => v.length, ...groupingFn);
  const childrenAccessorFn = ([key, value]) => value.size && Array.from(value);
  const hierarchyData = d3
    .hierarchy(rollupData, childrenAccessorFn)
    .sum(([key, value]) => value)
    .sort((a, b) => b.value - a.value);

  // Layout + node data prep
  const root = d3.pack().size([length, length]).padding(2)(hierarchyData);
  const nodes = root.descendants();
</script>

<svg width={length} height={length}>
  <!-- two each loops so the text will be on top of all circles -->
  {#each nodes as d}
    <circle
      transform={`translate(${d.x + 1},${d.y + 1})`}
      r={d.r}
      fill={d.height === 1 ? colorScale(d.data[0]) : "white"}
      opacity={d.height === 1 ? 1 : 0.5}
    />
  {/each}
  {#each nodes as d}
    <text
      transform={`translate(${d.x + 1},${d.y + 1})`}
      dy="3"
      text-anchor="middle"
      >{d.height === 1 ? d.data[0] : ""}
    </text>
  {/each}
</svg>

<style>
    text {
       font-size:14px;
       font-weight: 700;
       paint-order: stroke;
       stroke: white;
       stroke-width: 2px;
       stroke-linecap: butt;
       stroke-linejoin: miter;
    }
</style>

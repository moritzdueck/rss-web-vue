<script setup lang="ts">
import {onMounted, ref} from "vue";
import * as d3 from "d3";

const svgRef = ref(null);

interface DataPoint {
  jahr: number,
  klasse_steuerbares_einkommen_code: number
  klasse_steuerbares_einkommen_chf: string
  indikator: string
  wert: string,
  lowerBound?: number,
  upperBound?: number
}

onMounted(async () => {

  const width = 600;
  const height = 600;

  const svgContainer = d3.select(svgRef.value)
      .attr("viewBox", [0, 0, width, height])
      .property("value", [])
      .attr("width", width)
      .attr("height", height)


  let data: DataPoint[] = await fetch('https://data.bl.ch/api/v2/catalog/datasets/10590/exports/json').then(response => response.json());
  data = data.map(d => {
    d.lowerBound = +(d.klasse_steuerbares_einkommen_chf.split('-')[0] as string).split('').filter(char => '0123456789'.includes(char)).join('');
    d.upperBound = +(d.klasse_steuerbares_einkommen_chf.split('-')[1] as string)?.split('').filter(char => '0123456789'.includes(char)).join('') || d.lowerBound + 10000;
    return d;
  });

  const xScale = d3.scaleLinear()
      .domain([0, d3.max(data, (d => (d.upperBound || 0))) as number])
      .range([20, innerWidth-20])

  const yScale = d3.scaleLinear()
      .domain([d3.min(data, (d => d.jahr)) as number, d3.max(data, (d => d.jahr)) as number])
      .range([20, innerHeight-20])

  const rScale = d3.scaleLinear()
      .domain([1, d3.max(data, (d => (d.upperBound || 0))) as number])
      .range([2, 20])

  svgContainer.selectAll("circle")
      .data(data)
      .enter()
      .append("circle")
      .attr("cy", (d, i) => yScale(d.jahr))
      .attr("cx", d => xScale(d.lowerBound + Math.random() * (d.upperBound - d.lowerBound)))
      .attr("r",  d => rScale(d.lowerBound + Math.random() * (d.upperBound - d.lowerBound)))
      .attr("fill", "black")

  d3.forceSimulation(data)
      .force('x', d3.forceX(d => xScale(d.lowerBound + Math.random() * (d.upperBound - d.lowerBound))).strength(0.1))
      .force('y', d3.forceY(d => yScale(d.jahr)).strength(0.1))
      .force('collide', d3.forceCollide(d => rScale(d.lowerBound + Math.random() * (d.upperBound - d.lowerBound))))
      .on('tick', () => {
        svgContainer.selectAll("circle")
            .attr("cy", (d, i) => d.y)
            .attr("cx", d => d.x)
      })

});

</script>

<template>
  <div>
    <svg ref="svgRef"></svg>
  </div>
</template>

<style scoped>

</style>

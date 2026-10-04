<script setup>
import { onMounted, ref } from 'vue'
import * as d3 from 'd3'

const chartRef = ref(null)

const width = 900
const height = 640
const margin = { top: 50, right: 80, bottom: 50, left: 110 }

const industries = ['電子', '金融', '其他']
const color = d3.scaleOrdinal()
  .domain(industries)
  .range(['#4e79a7', '#f28e2b', '#9c9c9c'])

onMounted(async () => {
  const data = await d3.csv(`${import.meta.env.BASE_URL}data/stocks.csv`, d => ({
    rank: +d.rank,
    code: d.code,
    name: d.name,
    households: +d.households,
    industry: d.industry,
  }))
  data.sort((a, b) => d3.descending(a.households, b.households))

  const svg = d3.select(chartRef.value)
    .append('svg')
    .attr('viewBox', [0, 0, width, height])

  const x = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.households)])
    .nice()
    .range([margin.left, width - margin.right])

  const y = d3.scaleBand()
    .domain(data.map(d => d.name))
    .range([margin.top, height - margin.bottom])
    .padding(0.2)

  svg.append('g')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).tickSize(-(height - margin.top - margin.bottom)).tickFormat(''))
    .call(g => g.select('.domain').remove())
    .call(g => g.selectAll('line').attr('stroke', '#e5e5e5'))

  svg.append('g')
    .selectAll('rect')
    .data(data)
    .join('rect')
    .attr('x', x(0))
    .attr('y', d => y(d.name))
    .attr('width', d => x(d.households) - x(0))
    .attr('height', y.bandwidth())
    .attr('fill', d => color(d.industry))

  svg.append('g')
    .selectAll('text')
    .data(data)
    .join('text')
    .attr('class', 'value-label')
    .attr('x', d => x(d.households) + 4)
    .attr('y', d => y(d.name) + y.bandwidth() / 2)
    .attr('dy', '0.35em')
    .text(d => d3.format(',')(d.households))

  svg.append('g')
    .attr('class', 'axis')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).ticks(6).tickFormat(d3.format(',')))

  svg.append('g')
    .attr('class', 'axis')
    .attr('transform', `translate(${margin.left},0)`)
    .call(d3.axisLeft(y).tickSizeOuter(0))
    .call(g => g.selectAll('.tick text').text(d => {
      const s = data.find(item => item.name === d)
      return `${s.name} ${s.code}`
    }))

  svg.append('text')
    .attr('x', (margin.left + width - margin.right) / 2)
    .attr('y', height - 10)
    .attr('text-anchor', 'middle')
    .attr('font-size', 13)
    .text('交易戶數（戶）')

  const legend = svg.append('g')
    .attr('class', 'legend')
    .attr('transform', `translate(${width - margin.right - 200},${margin.top - 30})`)
    .selectAll('g')
    .data(industries)
    .join('g')
    .attr('transform', (d, i) => `translate(${i * 70},0)`)

  legend.append('rect')
    .attr('width', 14)
    .attr('height', 14)
    .attr('fill', d => color(d))

  legend.append('text')
    .attr('x', 20)
    .attr('y', 7)
    .attr('dy', '0.35em')
    .text(d => d)
})
</script>

<template>
  <div ref="chartRef" class="chart"></div>
</template>

<style scoped>
.chart :deep(svg) {
  display: block;
  width: 100%;
  height: auto;
}

.chart :deep(.axis text) {
  font-size: 13px;
}

.chart :deep(.value-label) {
  font-size: 12px;
  fill: #333;
}

.chart :deep(.legend text) {
  font-size: 13px;
}
</style>

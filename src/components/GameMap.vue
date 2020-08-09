<template>
  <div :id="team + 'GameMap'" />
</template>
<script>
import * as d3 from 'd3'
export default {
  name: 'GameMap',
  props: {
    team: {
      type: String,
      required: true
    },
    games: {
      type: Array,
      required: true
    }
  },
  mounted () {
    // Configuration vars
    const X_TICKS = 10
    const CELL_SIZE = 40
    const MARGIN = { top: 20, right: 20, bottom: 0, left: 110 }
    var WIDTH = 750 - MARGIN.right - MARGIN.left
    var HEIGHT = 400 - MARGIN.top - MARGIN.bottom

    // should we calculate uniqueDates or uniqueGames?
    var uniqueDates = d3.set(this.games.map(function (d) { return d.game_date })).values()
    var yAxisTicks = Math.ceil(uniqueDates.length / X_TICKS)

    // --------SCALES-----------
    var xScale = d3.scaleBand()
      .domain([...Array(X_TICKS).keys()])
      .range([0, X_TICKS * CELL_SIZE])

    var yScale = d3.scaleBand()
      .domain([...Array(yAxisTicks).keys()])
      .range([0, yAxisTicks * CELL_SIZE])

    // --------AXIS-----------
    var xAxis = d3.axisTop().scale(xScale).tickSize(0)
    var yAxis = d3.axisLeft().scale(yScale).tickSize(0)

    var svg = d3.select(this.$el)
      .append('svg')
      .attr('width', WIDTH + MARGIN.left + MARGIN.right)
      .attr('height', HEIGHT + MARGIN.top + MARGIN.bottom)
      .append('g')
      .attr('transform', 'translate(' + MARGIN.left + ',' + MARGIN.top + ')')

    // Create an SVG group Element for the Axis elements and call the xAxis function
    svg.append('g')
      .attr('class', 'axis')
      .call(xAxis)

    svg.append('g')
      .attr('class', 'axis')
      .call(yAxis)

    // Can we loop over all grouped data (e.g: team games) and create rects for all of them ?
    console.log(this.team)
    console.log(this.games)
    svg.selectAll('rect')
      .data(this.games)
      .enter().append('g').append('rect')
      .attr('class', 'cell')
      .attr('width', CELL_SIZE)
      .attr('height', CELL_SIZE)
      .attr('y', function (d, i) {
        if (i < X_TICKS) {
          return yScale(0)
        } else {
          return yScale(i.toString()[0])
        }
      })
      .attr('x', function (d, i) { return xScale(i % X_TICKS) })
      .attr('fill', function (d, i) {
        if (d.win_status === 1) { console.log(i); return '#ccc' } else { return '#000' }
      })
    // svg.append('g')
    //   .attr('class', 'y axis')
    //   .call(yAxis)
    //   .selectAll('text')
    //   .attr('font-weight', 'normal')

    // svg.append('g')
    //   .attr('class', 'x axis')
    //   .call(xAxis)
    //   .selectAll('text')
    //   .attr('font-weight', 'normal')
    //   .style('text-anchor', 'start')
    //   .attr('dx', '.8em')
    //   .attr('dy', '.5em')
    //   .attr('transform', function (d) {
    //     return 'rotate(-65)'
    //   })
  }
}
</script>
<style>
  .axis path,
  .axis line {
    fill: none;
    stroke: black;
    shape-rendering: crispEdges;
  }

  .axis text {
    font-family: sans-serif;
    font-size: 11px;
  }
</style>

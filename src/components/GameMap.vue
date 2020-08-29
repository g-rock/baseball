<template>
  <div :id="cleanTeamID" />
</template>
<script>
/* eslint-disable */
import colors from './color.js'
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
  computed: {
    cleanTeamID: function () {
      let teamSplit = this.team.replace('.', '').split(' ')
      return teamSplit.join('_')
    }
  },
  mounted () {
    // Configuration vars
    const X_TICKS = 10
    const CELL_SIZE = 40
    const LOGO_SIZE = CELL_SIZE
    const MARGIN = { top: LOGO_SIZE, right: 20, bottom: 0, left: 5 }
    var WIDTH = 750 - MARGIN.right - MARGIN.left
    var HEIGHT = 400 - MARGIN.top - MARGIN.bottom
    var PADDING_INNER = 0.1 // padding between rects

    // should we calculate uniqueDates or uniqueGames?
    var uniqueDates = d3.set(this.games.map(function (d) { return d.game_date })).values()
    var yAxisTicks = Math.ceil(uniqueDates.length / X_TICKS)

    // --------SCALES-----------
    var xScale = d3.scaleBand()
      .range([0, X_TICKS * CELL_SIZE])
      .domain([...Array(X_TICKS).keys()])
      .paddingInner(PADDING_INNER * 2)

    var yScale = d3.scaleBand()
      .range([0, yAxisTicks * CELL_SIZE])
      .domain([...Array(yAxisTicks).keys()])
      .paddingInner(PADDING_INNER)

    // --------AXIS-----------
    var xAxis = d3.axisTop().scale(xScale).tickSize(0)
    var yAxis = d3.axisRight().scale(yScale).tickSize(0)

    var svg = d3.select(this.$el)
      .append('svg')
      .attr('width', WIDTH + MARGIN.left + MARGIN.right)
      .attr('height', HEIGHT + MARGIN.top + MARGIN.bottom)
      .append('g')

    // svg.append('text')
    //   .attr('class', 'title')
    //   .attr('x', 0)
    //   .attr('y', MARGIN.top)
    //   .attr('font-family', 'sans-serif')
    //   .attr('fill', colors[this.team].main)
    //   .text(this.team)

    var imgs = svg.selectAll('img').data([0]);
    imgs.enter()
        .append('img:image')
        .attr('xlink:href', require('@/assets/logos/' + this.team + '.gif'))
        .attr('x', 0)
        .attr('y', 0)
        .attr('width', 60)
        .attr('height', LOGO_SIZE)
        .append('title')
        .text(this.team)

    var gameGroup = svg.append('g')
      .attr('transform', 'translate(' + MARGIN.left + ',' + (MARGIN.top + 10) + ')')

    // var xAxisGroup = gameGroup.append('g')
    //   .attr('class', 'axis')
    //   .call(xAxis)

    var yAxisGroup = gameGroup.append('g')
      .attr('class', 'axis')
      // PADDING_INNER should be a very tiny number, so it's ok to multiply by 100
      .attr('transform', 'translate(' + (X_TICKS * CELL_SIZE + PADDING_INNER * 100) + ',0)')
      .call(yAxis)

    yAxisGroup.selectAll('text')
      .each(function(d, i) {
        return d3.select(this).text((d + 1) * 10)
      }) 

    var Tooltip = d3.select('#' + this.cleanTeamID)
      .append('div')
      .style('opacity', 0)
      .attr('class', 'tooltip')
      .style('background-color', 'white')
      .style('border', 'solid')
      .style('border-width', '2px')
      .style('border-radius', '5px')
      .style('padding', '5px')

    var mouseover = function(d) {
      d3.select(this)
        .attr('stroke-linejoin', 'round')
        .attr('stroke-opacity', 1)
        .attr('stroke-width', 2)
        .style('stroke', 'black')
    }
    var mousemove = function(d) {
      Tooltip
        .html('The exact value of<br>this cell is: ' + d.value)
        .style('left', (d3.mouse(this)[0]+70) + 'px')
        .style('top', (d3.mouse(this)[1]) + 'px')
    }
    var mouseleave = function(d) {
      d3.select(this)
        .attr('stroke-linejoin', 'miter')
        .attr('stroke-opacity', 0)
        .attr('stroke-width', 1)
        .style('stroke', 'white')
    }
    // Can we loop over all grouped data (e.g: team games) and create rects for all of them ?
    gameGroup.selectAll('rect')
      .data(this.games)
      .enter()
      .append('g')
      .append('rect')
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
        .attr('margin-left', 2)
        .attr('fill', function (d, i) {
          return colors[d.team_name].main
        })
        .attr('fill-opacity', function (d, i) {
          if (d.win_status === 0) { return 0.5 } else { return 0.9 }
        })
        .attr('stroke', 'white')
        .attr('stroke-opacity', 0)
      .on('mouseover', mouseover)
      .on('mousemove', mousemove)
      .on('mouseleave', mouseleave)
  }
}
</script>
<style>
  .axis path {
    stroke: white;
  }

  .axis text {
    font-family: sans-serif;
    font-size: 11px;
  }
</style>

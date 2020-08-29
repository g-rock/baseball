<template>
  <div id="container">
    <div
      v-for="(team, index) in data"
      :key="index"
    >
      <GameMap
        :team="team.key"
        :games="team.values"
      />
    </div>
  </div>
</template>
<script>
import * as d3 from 'd3'
import GameMap from './GameMap'
export default {
  name: 'GameContainer',
  components: {
    GameMap
  },
  data () {
    return {
      data: null
    }
  },
  mounted () {
    const CSV_F = 'last_year.csv'
    d3.timeFormat('%Y-%m-%d')
    d3.csv(CSV_F)
      .then(function (responseData) {
        const formattedData = responseData
        formattedData.forEach(function (d, i) {
          d.win_status = +d.win_status
          return d
        })
        const nest = d3.nest()
          .key(function (d) { return d.team_name }).sortKeys(d3.ascending)
          .entries(formattedData)
        this.data = nest
      }.bind(this))
  }
}
</script>

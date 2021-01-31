<template>
  <div>
    <div class="sort-console">
      <div>Sort by</div>
      <el-radio-group v-model="radio">
        <el-radio label="count">Match count</el-radio>
        <el-radio label="win">Winnings</el-radio>
        <el-radio label="lose">Losings</el-radio>
      </el-radio-group>
    </div>

    <svg id='headtohead'
         width='400'
         height='1800'
         class='headtohead'></svg>
  </div>

</template>

<script>
import * as d3 from 'd3'

export default {
  props: {
    matchData: {
      type: Array,
      default () {
        return [
        ];
      }
    }
  },
  data () {
    return {
      radio: 'count',
      svg: null,
      width: null,
      height: null,
      margin: { top: 60, right: 20, bottom: 10, left: 10 },
      tooltipMargin: { top: 0, right: 10, bottom: 2, left: 10 },
      innerWidth: null,
      innerHeight: null,
      xScale: null,
      yScale: null,
      xAxisLable: 'Opponents',
      yAxisLable: 'Head to Head',
      xValue: d => d.index,
      yValue: d => d.opponent,
      yAxisGroup: null,
      g: null,
      opponentRecord: [],
      renderData: []
    }
  },
  components: {

  },
  mounted () {
    /*
    *   Init data
    */
    this.svg = d3.select('#headtohead')
    this.width = +this.svg.attr('width')
    this.height = +this.svg.attr('height')
    this.innerWidth = this.width - this.margin.left - this.margin.right
    this.innerHeight = this.height - this.margin.top - this.margin.bottom

    this.init()
    this.renderinit()
    this.sortupdate()
  },
  watch: {
    'radio': {
      handler () {
        this.sortupdate()
      }
    }
  },
  methods: {
    init () {
      let allOpponents = Array.from(new Set(this.matchData.map(d => d.opponent)))
      this.opponentRecord = {}
      allOpponents.forEach(item => {
        this.opponentRecord[item] = {
          'win': [],
          'lose': []
        }
      })

      this.matchData.forEach(item => {
        if (item.result == 'win')
          this.opponentRecord[item.opponent].win.push(item)
        else
          this.opponentRecord[item.opponent].lose.push(item)
      })

      this.renderData = []

      for (var key in this.opponentRecord) {
        this.opponentRecord[key].win.sort((a, b) => {
          return a.net - b.net
        })
        this.opponentRecord[key].win.forEach((item, index) => {
          item.index = index + 1
          this.renderData.push(item)
        })
        this.opponentRecord[key].lose.sort((a, b) => {
          return b.net - a.net
        })
        this.opponentRecord[key].lose.forEach((item, index) => {
          item.index = -(index + 1)
          this.renderData.push(item)
        })
        var winCount = this.opponentRecord[key].win.length
        var loseCount = this.opponentRecord[key].lose.length
        this.opponentRecord[key].win = winCount
        this.opponentRecord[key].lose = loseCount
      }
    },

    renderinit () {
      this.svg.append('g').append('text')
        .attr('fill', '#999')
        .attr('x', this.innerWidth / 2)
        .attr('y', 32)
        .attr('font-size', 24)
        .attr('text-anchor', 'middle')
        .text('Head to Head')
      // calculate X axis range
      var maxIndex = Math.max.apply(Math, this.renderData.map(item => { return item.index }))
      var minIndex = Math.min.apply(Math, this.renderData.map(item => { return item.index }))
      var maxX = Math.max(Math.abs(maxIndex), Math.abs(minIndex))

      this.xScale = d3.scaleLinear()
        .domain([maxX, -maxX])
        .range([0, this.innerWidth])
      this.yScale = d3.scaleBand()
        .domain(this.renderData.map(this.yValue))
        .range([0, this.innerHeight])

      this.g = this.svg.append('g').attr('id', 'maingroup')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)

      const yAxis = d3.axisLeft(this.yScale)
        .tickSize(0)

      const xAxis = d3.axisBottom(this.xScale)
        .tickSize(0)

      /*
      *   ring
      */
      const ring = this.svg.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .append('circle')
        .attr('class', 'ring')
        .attr('r', 6)
        .attr('fill', 'none')
        .attr('stroke', 'black')
        .attr('opacity', 1)
        .attr('cx', -this.margin.left * 2)
        .attr('cy', -this.margin.top * 2)

      /*
      *   tooltip
      */
      const tooltip = this.svg.append('g')
        .attr('id', 'h2htooltip')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 12)
        .attr('text-anchor', 'middle')

      tooltip
        .append('rect')
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#FFF')
        .attr('fill-opacity', 0.6)
        .attr('stroke', '#999')

      tooltip
        .append('text')
        .attr('class', 'tip1')
        .attr('dy', '1em')

      tooltip
        .append('text')
        .attr('class', 'tip2')
        .attr('dy', '2em')

      tooltip
        .append('text')
        .attr('class', 'tip3')
        .attr('dy', '3em')

      tooltip
        .append('text')
        .attr('class', 'tip4')
        .attr('dy', '4em')
        .style('font-weight', 'bold')

      /*
      *   circle
      */
      this.g.selectAll('circle')
        .data(this.renderData)
        .join('circle')
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => {
          return d.sets > 2 ? 4 : 6
        })
        .attr('fill', d => {
          return d.result == 'win' ? '#FF9900' : '#0099FF'
        })
        .attr('fill-opacity', 0.6)
        .on('mouseover', (d, k) => {
          ring
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.yScale(this.yValue(k)))
            .attr('r', () => {
              return k.sets > 2 ? 6 : 8
            })
            .attr('opacity', 1)

          tooltip.select('.tip1')
            .text(k.tournament)
          tooltip.select('.tip2')
            .text(k.dateStr)
          tooltip.select('.tip3')
            .text('Opponent: ' + k.opponent)
          tooltip.select('.tip4')
            .text(k.score)
          var textLength = [
            tooltip.select('.tip1').node().getComputedTextLength(),
            tooltip.select('.tip2').node().getComputedTextLength(),
            tooltip.select('.tip3').node().getComputedTextLength(),
            tooltip.select('.tip4').node().getComputedTextLength()
          ]
          tooltip
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k)) - Math.max(...textLength) / 2},
                                                      ${this.margin.top + this.yScale(this.yValue(k)) - 60})`)
            .attr('opacity', 1)
            .select('rect')
            .attr('width', Math.max(...textLength) + this.tooltipMargin.left + this.tooltipMargin.right)
            .attr('height', 48 + this.tooltipMargin.top + this.tooltipMargin.bottom)

          tooltip.selectAll('text')
            .attr('dx', this.tooltipMargin.left + Math.max(...textLength) / 2)

        })
        .on('mouseout', d => {
          ring
            .attr('cx', -this.margin.left)
            .attr('cy', -this.margin.top)
            .attr('opacity', 0)
          tooltip
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
        })
        .on('click', (d, k) => {
          this.g.selectAll('circle')
            .attr('fill-opacity', dd => {
              return dd.opponent == k.opponent ? 1 : 0.2
            })
          this.$emit('nodeClick', k)
          this.yAxisGroup
            .attr('opacity', dd => {
              return dd == k.opponent ? 1 : 0.2
            })
        })

      this.yAxisGroup = this.g.append('g').call(yAxis)
        .attr('transform', `translate(${this.innerWidth / 2}, 0)`)
        .attr('id', 'yaxis')
        .selectAll('text')
        .attr('dy', -2)
        .attr('transform', d => {
          if (this.opponentRecord[d].win < this.opponentRecord[d].lose)
            return `translate(${this.xScale(this.opponentRecord[d].win + 1) - this.innerWidth / 2}, 0) rotate(0)`
          else
            return `translate(${this.xScale(-this.opponentRecord[d].lose - 1) - this.innerWidth / 2}, 0) rotate(0)`
        })
        .attr('font-size', 10)
        .attr('text-anchor', d => {
          if (this.opponentRecord[d].win < this.opponentRecord[d].lose)
            return 'end'
          else
            return 'start'
        })
        .attr('opacity', d => {
          if (this.opponentRecord[d].win + this.opponentRecord[d].lose > 2)
            return 1
          else
            return 0.1
        })
      this.yAxisGroup.selectAll('.domain').remove()

      this.svg
        .on('click', () => {
          if (ring.attr('opacity') == 0) {
            this.g.selectAll('circle')
              .attr('fill-opacity', 0.6)
            this.$emit('cancleClick')
            this.yAxisGroup
              .attr('opacity', d => {
                if (this.opponentRecord[d].win + this.opponentRecord[d].lose > 2)
                  return 1
                else
                  return 0.1
              })
          }
        })
    },

    highlight (item) {
      var top = 0
      this.g.selectAll('circle')
        .attr('fill-opacity', dd => {
          if (dd.opponent == item.opponent) {
            top = this.yScale(this.yValue(dd))
            return 1
          }
          else
            return 0.2
        })
      this.yAxisGroup
        .attr('opacity', dd => {
          return dd == item.opponent ? 1 : 0.2
        })
      this.$emit('highlight', top)
    },

    cancleHighlight () {
      this.g.selectAll('circle')
        .attr('fill-opacity', 0.6)
      this.yAxisGroup
        .attr('opacity', d => {
          if (this.opponentRecord[d].win + this.opponentRecord[d].lose > 2)
            return 1
          else
            return 0.1
        })
    },

    getOpponentRecord (item) {
      return {
        'player': item.player,
        'opponent': item.opponent,
        'win': this.opponentRecord[item.opponent].win,
        'lose': this.opponentRecord[item.opponent].lose
      }
    },

    figureupdate () {
      this.init()
      this.renderupdate()
      this.sortupdate()

    },

    sortupdate () {
      var allOpponents = []
      for (var key in this.opponentRecord) {
        allOpponents.push({
          'opponent': key,
          'win': this.opponentRecord[key].win,
          'lose': this.opponentRecord[key].lose,
          'count': this.opponentRecord[key].win + this.opponentRecord[key].lose
        })
      }

      allOpponents.sort((a, b) => b[this.radio] - a[this.radio])

      this.yScale = d3.scaleBand()
        .domain(allOpponents.map(this.yValue))
        .range([0, this.innerHeight])

      const yAxis = d3.axisLeft(this.yScale)
        .tickSize(0)

      this.g.selectAll('circle')
        .transition().ease(d3.easeLinear).duration(500)
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => {
          return d.sets > 2 ? 4 : 6
        })

      this.g.select('#yaxis')
        .transition().ease(d3.easeLinear).duration(500)
        .call(yAxis)

    },

    renderupdate () {

      // calculate X axis range
      var maxIndex = Math.max.apply(Math, this.renderData.map(item => { return item.index }))
      var minIndex = Math.min.apply(Math, this.renderData.map(item => { return item.index }))
      var maxX = Math.max(Math.abs(maxIndex), Math.abs(minIndex))

      this.xScale = d3.scaleLinear()
        .domain([maxX, -maxX])
        .range([0, this.innerWidth])
      this.yScale = d3.scaleBand()
        .domain(this.renderData.map(this.yValue))
        .range([0, this.innerHeight])


      const yAxis = d3.axisLeft(this.yScale)
        .tickSize(0)

      const xAxis = d3.axisBottom(this.xScale)
        .tickSize(0)

      this.svg.selectAll('circle').remove()
      /*
      *   ring
      */
      const ring = this.svg.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .append('circle')
        .attr('class', 'ring')
        .attr('r', 6)
        .attr('fill', 'none')
        .attr('stroke', 'black')
        .attr('opacity', 1)
        .attr('cx', -this.margin.left * 2)
        .attr('cy', -this.margin.top * 2)

      /*
      *   tooltip
      */
      this.svg.select('#h2htooltip').remove()

      const tooltip = this.svg.append('g')
        .attr('id', 'h2htooltip')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 12)
        .attr('text-anchor', 'middle')

      tooltip
        .append('rect')
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#FFF')
        .attr('fill-opacity', 0.6)
        .attr('stroke', '#999')

      tooltip
        .append('text')
        .attr('class', 'tip1')
        .attr('dy', '1em')

      tooltip
        .append('text')
        .attr('class', 'tip2')
        .attr('dy', '2em')

      tooltip
        .append('text')
        .attr('class', 'tip3')
        .attr('dy', '3em')

      tooltip
        .append('text')
        .attr('class', 'tip4')
        .attr('dy', '4em')
        .style('font-weight', 'bold')

      /*
      *   circle
      */
      this.g.selectAll('circle')
        .data(this.renderData)
        .join('circle')
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('fill', d => {
          return d.result == 'win' ? '#FF9900' : '#0099FF'
        })
        .attr('fill-opacity', 0.6)
        .attr('r', 0)
        .transition().ease(d3.easeLinear).duration(500)
        .attr('r', d => {
          return d.sets > 2 ? 4 : 6
        })

      this.g.selectAll('circle')
        .on('mouseover', (d, k) => {
          ring
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.yScale(this.yValue(k)))
            .attr('r', () => {
              return k.sets > 2 ? 6 : 8
            })
            .attr('opacity', 1)

          tooltip.select('.tip1')
            .text(k.tournament)
          tooltip.select('.tip2')
            .text(k.dateStr)
          tooltip.select('.tip3')
            .text('Opponent: ' + k.opponent)
          tooltip.select('.tip4')
            .text(k.score)
          var textLength = [
            tooltip.select('.tip1').node().getComputedTextLength(),
            tooltip.select('.tip2').node().getComputedTextLength(),
            tooltip.select('.tip3').node().getComputedTextLength(),
            tooltip.select('.tip4').node().getComputedTextLength()
          ]
          tooltip
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k)) - Math.max(...textLength) / 2},
                                                      ${this.margin.top + this.yScale(this.yValue(k)) - 60})`)
            .attr('opacity', 1)
            .select('rect')
            .attr('width', Math.max(...textLength) + this.tooltipMargin.left + this.tooltipMargin.right)
            .attr('height', 48 + this.tooltipMargin.top + this.tooltipMargin.bottom)

          tooltip.selectAll('text')
            .attr('dx', this.tooltipMargin.left + Math.max(...textLength) / 2)

        })
        .on('mouseout', d => {
          ring
            .attr('cx', -this.margin.left)
            .attr('cy', -this.margin.top)
            .attr('opacity', 0)
          tooltip
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
        })
        .on('click', (d, k) => {
          this.g.selectAll('circle')
            .attr('fill-opacity', dd => {
              return dd.opponent == k.opponent ? 1 : 0.2
            })
          this.$emit('nodeClick', k)
          this.yAxisGroup
            .attr('opacity', dd => {
              return dd == k.opponent ? 1 : 0.2
            })
        })

      this.g.select('#yaxis').remove()
      this.yAxisGroup = this.g.append('g').call(yAxis)
        .attr('transform', `translate(${this.innerWidth / 2}, 0)`)
        .attr('id', 'yaxis')
        .selectAll('text')
        .attr('dy', -2)
        .attr('transform', d => {
          if (this.opponentRecord[d].win < this.opponentRecord[d].lose)
            return `translate(${this.xScale(this.opponentRecord[d].win + 1) - this.innerWidth / 2}, 0) rotate(0)`
          else
            return `translate(${this.xScale(-this.opponentRecord[d].lose - 1) - this.innerWidth / 2}, 0) rotate(0)`
        })
        .attr('font-size', 10)
        .attr('text-anchor', d => {
          if (this.opponentRecord[d].win < this.opponentRecord[d].lose)
            return 'end'
          else
            return 'start'
        })
        .attr('opacity', d => {
          if (this.opponentRecord[d].win + this.opponentRecord[d].lose > 2)
            return 1
          else
            return 0.1
        })
      this.yAxisGroup.selectAll('.domain').remove()

      this.svg
        .on('click', () => {
          if (ring.attr('opacity') == 0) {
            this.g.selectAll('circle')
              .attr('fill-opacity', 0.6)
            this.$emit('cancleClick')
            this.yAxisGroup
              .attr('opacity', d => {
                if (this.opponentRecord[d].win + this.opponentRecord[d].lose > 2)
                  return 1
                else
                  return 0.1
              })
          }
        })
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.sort-console {
  border-radius: 6px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  margin: 6px;
  padding: 8px;
}
</style>
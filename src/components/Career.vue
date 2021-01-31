<template>
  <div class="svg-wrap"
       id="svg-wrap">
    <svg id='careersvg'
         height='600'
         class='careersvg'></svg>
    <el-slider v-model="value"
               class="slider"
               :min="sliderRange[0]"
               :max="sliderRange[1]"
               :step=24*3600*1000
               :marks="marks"
               @change="rangeChange"
               :format-tooltip="dateFormat"
               range>
    </el-slider>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  props: {
    matchData: {
      type: Array,
      default () {
        return []
      }
    },
    rankingData: {
      type: Array,
      default () {
        return []
      }
    },
    careerNews: {
      type: Array,
      default () {
        return []
      }
    },
    winningRate: {
      type: Array,
      default () {
        return []
      }
    },
    svgWidth: {
      type: Number,
      default () {
        return 1000
      }
    }
  },
  data () {
    return {
      svg: null,
      width: null,
      height: null,
      margin: { top: 50, right: 100, bottom: 60, left: 100 },
      tooltipMargin: { top: 0, right: 10, bottom: 2, left: 10 },
      innerWidth: null,
      innerHeight: null,
      xScale: null,
      yScale: null,
      rateScale: null,
      rankScale: null,
      xAxisLable: 'Career Path',
      yAxisLable: 'Net Total Points',
      rateAxisLabel: 'Winning Rate',
      rankAxisLabel: 'Rank/World Tour Rank',
      xValue: d => d.date,
      yValue: d => d.net,
      rateValue: d => d.winningRate,
      g: null,
      circle: null,
      value: [30, 60],
      sliderRange: [0, 100],
      marks: {},
      bigWinColor: '#FF9900',
      smallWinColor: '#FF9900',
      bigLoseColor: '#0099FF',
      smallLoseColor: '#0099FF',
      bigCircleR: 6,
      smallCircleR: 4
    }
  },
  created () {
    this.init()
  },
  components: {},
  mounted () {
    /*
    *   Init data
    */
    this.svg = d3.select('#careersvg')
    this.width = +this.svgWidth
    this.height = +this.svg.attr('height')
    this.innerWidth = this.width - this.margin.left - this.margin.right
    this.innerHeight = this.height - this.margin.top - this.margin.bottom

    this.renderinit(this.matchData, this.winningRate, this.rankingData, this.careerNews)
  },
  watch: {
    'svgWidth': {
      handler () {
        this.widthupdate()
      }
    }
  },
  methods: {
    init () {
      var maxTime = Math.max.apply(Math, this.matchData.map(item => { return item.date }))
      var minTime = Math.min.apply(Math, this.matchData.map(item => { return item.date }))
      this.value = [minTime, maxTime]
      this.sliderRange = [minTime, maxTime]
      var maxYear = new Date(maxTime).getFullYear()
      var temp = new Date(minTime)
      temp.setDate(1)
      temp.setMonth(0)
      this.marks = {}
      while (temp.getFullYear() < maxYear) {
        this.marks[temp.setFullYear(temp.getFullYear() + 1)] = temp.getFullYear().toString()
      }
    },
    renderinit (data, rateData, rankData, news) {

      this.svg
        .attr('width', this.width)
        .attr('height', this.height)

      var maxTime = Math.max.apply(Math, this.matchData.map(item => { return item.date }))
      var minTime = Math.min.apply(Math, this.matchData.map(item => { return item.date }))

      maxTime = new Date(maxTime)
      minTime = new Date(minTime)
      maxTime.setMonth(11)
      minTime.setMonth(0)
      maxTime.setDate(31)
      minTime.setDate(1)

      /*
      *   Scales Defined
      */
      this.xScale = d3.scaleTime()
        .domain([minTime, maxTime])
        .range([0, this.innerWidth])

      this.yScale = d3.scaleLinear()
        .domain(d3.extent(data, this.yValue).reverse())
        .range([0, this.innerHeight])
        .nice()

      this.rateScale = d3.scaleLinear()
        .domain([1, 0])
        .range([0, this.innerHeight / 2])

      this.rankScale = d3.scaleLinear()
        .domain(d3.extent(rankData, d => d.worldRanking))
        .range([0, this.innerHeight / 2])

      /*
      *   SVG layers 
      */

      this.back = this.svg.append('g').attr('id', 'backgroup')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)

      this.g = this.svg.append('g').attr('id', 'maingroup')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)

      this.front = this.svg.append('g').attr('id', 'frontgroup')

      this.uiLayer = this.svg.append('g').attr('id', 'uigroup')


      /*
      *   Axis Defined
      */
      const yAxis = d3.axisLeft(this.yScale)
        .ticks(6)

      const rateAxis = d3.axisRight(this.rateScale)

      const rankAxis = d3.axisRight(this.rankScale)

      const xAxis = d3.axisBottom(this.xScale)
        .tickSize(-6)

      /*
      *   Axis attached
      */

      let xAxisGroup = this.g.append('g').call(xAxis)
        .attr('transform', `translate(0, ${this.innerHeight})`)
        .attr('id', 'xaxis')

      let yAxisGroup = this.g.append('g').call(yAxis)
        .attr('id', 'yaxis')
      yAxisGroup.selectAll('line')
        .attr('x1', 0)
        .attr('x2', this.innerWidth)
        .attr('stroke', '#666')
        .attr('stroke-dasharray', '5,5')
        .attr('stroke-width', 1)
      yAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 2)
        .attr('y', -40)
        .attr('fill', '#333333')
        .text(this.yAxisLable)
        .attr('text-anchor', 'middle')
      yAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      let rateAxisGroup = this.g.append('g').call(rateAxis)
        .attr('id', 'rateAxis')
        .attr('transform', `translate(${this.innerWidth}, 0)`)
      rateAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 60)
        .attr('fill', '#333333')
        .text(this.rateAxisLabel)
        .attr('text-anchor', 'middle')
      rateAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      let rankAxisGroup = this.g.append('g').call(rankAxis)
        .attr('id', 'rankAxis')
        .attr('transform', `translate(${this.innerWidth}, ${this.innerHeight / 2})`)
      rankAxisGroup.selectAll('text')
        .attr('dx', 20)
      rankAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 80)
        .attr('fill', '#333333')
        .text(this.rankAxisLabel)
        .attr('text-anchor', 'middle')
      rankAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      /*
      *   ring
      */
      const ring = this.uiLayer.append('g')
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
      const tooltip = this.uiLayer.append('g')
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
      *   news board
      */
      const newsboard = this.uiLayer.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 12)
        .attr('text-anchor', 'start')

      newsboard
        .append('rect')
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#FFF')
        .attr('fill-opacity', 0.6)
        .attr('stroke', '#999')

      newsboard
        .append('text')
        .attr('class', 'title')
        .attr('dy', '1em')

      /*
      *   rate label
      */
      const rateLabel = this.front.append('g')
        .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 18)
        .attr('text-anchor', 'middle')

      rateLabel
        .append('text')
        .attr('font-size', 12)
        .attr('class', 'label1')
        .attr('dy', '0em')

      rateLabel
        .append('text')
        .attr('class', 'label2')
        .attr('dy', '1em')
        .style('font-weight', 'bold')

      /*
      *   rank Label
      */
      const rankLabel = this.front.append('g')
        .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 18)
        .attr('text-anchor', 'middle')

      rankLabel
        .append('text')
        .style('font-weight', 'bold')
        .attr('class', 'label1')
        .attr('dy', '0em')

      rankLabel
        .append('text')
        .attr('font-size', 12)
        .attr('class', 'label2')
        .attr('dy', '1em')

      /*
      *   circle
      */
      this.circle = this.front.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .attr('id', 'maincircle')
        .selectAll('circle')
        .data(data)
        .join('circle')
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => {
          return d.sets > 2 ? this.smallCircleR : this.bigCircleR
        })
        .attr('fill', d => {
          if (d.result == 'win')
            if (d.sets > 2)
              return this.smallWinColor
            else
              return this.bigWinColor
          else
            if (d.sets > 2)
              return this.smallLoseColor
            else
              return this.bigLoseColor
        })
        .attr('fill-opacity', 0.6)

      this.circle
        .on('mouseover', (d, k) => {
          ring
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.yScale(this.yValue(k)))
            .attr('r', () => {
              return k.sets > 2 ? this.smallCircleR + 2 : this.bigCircleR + 2
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
          this.circle
            .attr('fill-opacity', dd => {
              return dd.opponent == k.opponent ? 1 : 0.2
            })
          this.$emit('nodeClick', k)
        })

      this.svg
        .on('click', () => {
          if (ring.attr('opacity') == 0) {
            this.circle
              .attr('fill-opacity', 0.6)
            this.$emit('cancleClick')
          }

        })


      /*
      *   news ring
      */
      const newsRing = this.front.append('g')
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
      *  news circle
      */
      const newsCircle = this.front.append('g')
        .attr('id', 'newscircle')
        .selectAll('circle')
        .data(news)
        .join('circle')
        .attr('r', 4)
        .attr('cx', d => this.xScale(this.xValue(d)) + this.margin.left)
        .attr('cy', this.innerHeight + this.margin.top + 20)
        .attr('fill', '#CCC')
        .attr('opacity', 0.6)
        .on('mouseover', (d, k) => {
          newsboard.select('.title')
            .text(k.title)
            .attr('dx', this.tooltipMargin.left)
          var textLength = newsboard.select('.title').node().getComputedTextLength()
          newsboard
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k)) - textLength / 2}, ${this.margin.top + this.innerHeight - 30})`)
            .attr('opacity', 1)
            .select('rect')
            .attr('width', textLength + this.tooltipMargin.left + this.tooltipMargin.right)
            .attr('height', 14 + this.tooltipMargin.top + this.tooltipMargin.bottom)

          newsRing
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.innerHeight + 20)
            .attr('opacity', 1)
        })
        .on('mouseout', () => {
          newsboard
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          newsRing
            .attr('cx', -this.margin.left)
            .attr('cy', -this.margin.top)
            .attr('opacity', 0)
        })
        .on('click', (d, k) => {
          this.$emit('newsclick', k)
        })

      /*
      *   rate lines
      */
      const line = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rateScale(this.rateValue(d)))
        .curve(d3.curveLinear)

      this.g.append('path').attr('id', 'ratecurve')
        .datum(rateData)
        .attr('d', line(rateData))
        .attr('fill', 'none')
        .attr('stroke', '#339966')
        .attr('stroke-width', 1)

      /*
      *   rate line nodes
      */
      const lineNodes = this.svg.append('g')
        .attr('id', 'linenodes')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .selectAll('circle')
        .data(rateData)
        .join('circle')
        .attr('fill', '#339966')
        .attr('r', 3)
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.rateScale(this.rateValue(d)))

      /*
      *   rank curves
      */
      var LowestRank = Math.max.apply(Math, rankData.map(item => { return item.worldRanking }))
      var earliestRank = rankData[0].date
      var latestRank = rankData[rankData.length - 1].date

      rankData.splice(0, 0, {
        date: earliestRank,
        worldRanking: LowestRank,
        fake: true
      })

      rankData.push({
        date: latestRank,
        worldRanking: LowestRank,
        fake: true
      })

      const worldRankCurve = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .curve(d3.curveLinear)

      this.back.append('path').attr('id', 'worldrankback')
        .datum(rankData)
        .attr('d', worldRankCurve(rankData))
        .attr('fill', '#AAA')
        .attr('fill-opacity', 0.1)

      this.back.append('path').attr('id', 'worldrank')
        .datum(rankData)
        .attr('d', worldRankCurve(rankData.filter(d => !d.fake)))
        .attr('fill', 'none')
        .attr('stroke', '#CCC')
        .attr('stroke-width', 1)

      /*
      *   rank line nodes
      */
      const rankNodes = this.front.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .selectAll('circle')
        .data(rankData.filter(d => !d.fake))
        .join('circle')
        .attr('id', 'ranknodes')
        .attr('fill', '#000')
        .style('pointer-events', 'none')
        .attr('r', 4)
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .attr('opacity', 0)

      const rankNodeLines = this.front.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .selectAll('line')
        .data(rankData.filter(d => !d.fake))
        .join('line')
        .attr('id', 'ranknodelines')
        .attr('x1', d => this.xScale(this.xValue(d)))
        .attr('x2', d => this.xScale(this.xValue(d)))
        .attr('y1', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .attr('y2', this.innerHeight + 20)
        .attr('stroke', 'black')
        .style('pointer-events', 'none')
        .attr('opacity', 0)

      /*
      *   back rect
      */
      const rateRect = this.back.append('g')
        .selectAll('rect')
        .data(rateData)
        .join('rect')
        .attr('id', 'backrect')
        .attr('width', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          if (start < 0)
            start = 0
          date.setMonth(11)
          var end = this.xScale(date.setDate(31))
          if (end > this.innerWidth)
            end = this.innerWidth
          return end - start
        })
        .attr('height', this.innerHeight)
        .attr('x', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          return start > 0 ? start : 0
        })
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#DDD')
        .attr('fill-opacity', 0.4)
        .style('opacity', 0)
        .on('mouseover', (d, k) => {
          d.target.style.opacity = 1
          lineNodes
            .attr('r', dd => {
              return dd.date.getFullYear() == k.date.getFullYear() ? 5 : 3
            })
          rateLabel.select('.label1')
            .text(k.date.getFullYear() + ' Winning Rate')
          rateLabel.select('.label2')
            .text((k.winningRate * 100).toPrecision(3) + '%')
          rateLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k))}, ${this.margin.top - 20})`)
            .attr('opacity', 1)
        })
        .on('mouseout', d => {
          d.target.style.opacity = 0
          lineNodes
            .attr('r', 3)
          rateLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
        })

      /*
      *   Rank Rect
      */
      const rankRect = this.back.append('g')
        .selectAll('rect')
        .data(rankData.filter(d => !d.fake))
        .join('rect')
        .attr('id', 'rankrect')
        .attr('opacity', 0)
        .attr('width', 4)
        .attr('height', d => this.innerHeight / 2 - this.rankScale(d.worldRanking))
        .attr('r', 3)
        .attr('x', d => this.xScale(this.xValue(d)))
        .attr('y', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .on('mouseover', (d, k) => {
          rankNodes
            .attr('opacity', dd => {
              return dd.date == k.date ? 1 : 0
            })
          lineNodes
            .attr('r', dd => {
              return dd.date.getFullYear() == k.date.getFullYear() ? 5 : 3
            })
          var rectDate = null
          rateRect
            .style('opacity', d => {
              if (d.date.getFullYear() == k.date.getFullYear()) {
                rectDate = d.date
                return 1
              }
              else
                return 0
            })
          rateLabel.select('.label1')
            .text(k.date.getFullYear() + ' Winning Rate')
          rateLabel.select('.label2')
            .text((rateData.filter(dd => dd.date.getFullYear() == k.date.getFullYear())[0].winningRate * 100).toPrecision(3) + '%')
          rateLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(rectDate)}, ${this.margin.top - 20})`)
            .attr('opacity', 1)

          rankLabel.select('.label1')
            .text('World Rank: ' + k.worldRanking)
          rankLabel.select('.label2')
            .text(k.dateStr)
          rankLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k))}, ${this.innerHeight + this.margin.top + 40})`)
            .attr('opacity', 1)
          rankNodeLines
            .attr('opacity', dd => {
              return dd.date == k.date ? 1 : 0
            })
        })
        .on('mouseout', () => {
          rankNodes
            .attr('opacity', 0)
          rateRect
            .style('opacity', 0)
          lineNodes
            .attr('r', 3)
          rateLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          rankLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          rankNodeLines
            .attr('opacity', 0)
        })

    },

    highlight (item) {
      this.circle
        .attr('fill-opacity', dd => {
          return dd.opponent == item.opponent ? 1 : 0.2
        })
    },

    cancleHighlight () {
      this.circle
        .attr('fill-opacity', 0.6)
    },

    circleFilter (options) {
      this.circle
        .attr('fill-opacity', dd => {
          var picked = false
          options.forEach(item => {
            if (dd.tournament.indexOf(item) != -1)
              picked = true
          })
          return picked ? 1 : 0.2
        })
    },

    dateFormat (timestamp) {
      var date = new Date(timestamp)
      return date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate()
    },

    rangeChange (value) {
      var dateMin = new Date(value[0])
      var dateMax = new Date(value[1])
      /* this.matchData, this.winningRate, this.rankingData, this.careerNews */
      this.renderupdate(
        this.matchData.filter(d => d.date > dateMin && d.date < dateMax),
        this.winningRate.filter(d => d.date.getFullYear() >= dateMin.getFullYear() && d.date.getFullYear() <= dateMax.getFullYear()),
        this.rankingData.filter(d => d.date > dateMin && d.date < dateMax),
        this.careerNews.filter(d => d.date > dateMin && d.date < dateMax)
      )
    },

    figureupdate () {
      this.init()
      this.renderupdate(
        this.matchData,
        this.winningRate,
        this.rankingData,
        this.careerNews
      )
    },

    renderupdate (data, rateData, rankData, news) {

      this.uiLayer.selectAll().remove()

      this.svg
        .transition().ease(d3.easeLinear).duration(500)
        .attr('width', this.width)
        .attr('height', this.height)

      var maxTime = Math.max.apply(Math, data.concat(rateData).map(item => { return item.date }))
      var minTime = Math.min.apply(Math, data.concat(rateData).map(item => { return item.date }))

      maxTime = new Date(maxTime)
      minTime = new Date(minTime)

      /*
      *   Scales Defined
      */
      this.xScale = d3.scaleTime()
        .domain([minTime, maxTime])
        .range([0, this.innerWidth])

      this.yScale = d3.scaleLinear()
        .domain(d3.extent(data, this.yValue).reverse())
        .range([0, this.innerHeight])
        .nice()

      this.rateScale = d3.scaleLinear()
        .domain([1, 0])
        .range([0, this.innerHeight / 2])

      this.rankScale = d3.scaleLinear()
        .domain(d3.extent(rankData, d => d.worldRanking))
        .range([0, this.innerHeight / 2])


      /*
      *   Axis Defined
      */
      const yAxis = d3.axisLeft(this.yScale)
        .ticks(6)

      const rateAxis = d3.axisRight(this.rateScale)

      const rankAxis = d3.axisRight(this.rankScale)

      const xAxis = d3.axisBottom(this.xScale)
        .tickSize(-6)

      /*
      *   Axis attached
      */

      this.g.select('#xaxis').remove()
      let xAxisGroup = this.g.append('g').call(xAxis)
        .attr('transform', `translate(0, ${this.innerHeight})`)
        .attr('id', 'xaxis')

      this.g.select('#yaxis').remove()
      let yAxisGroup = this.g.append('g').call(yAxis)
        .attr('id', 'yaxis')
      yAxisGroup.selectAll('line')
        .attr('x1', 0)
        .attr('x2', this.innerWidth)
        .attr('stroke', '#666')
        .attr('stroke-dasharray', '5,5')
        .attr('stroke-width', 1)
      yAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 2)
        .attr('y', -40)
        .attr('fill', '#333333')
        .text(this.yAxisLable)
        .attr('text-anchor', 'middle')
      yAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      this.g.select('#rateAxis').remove()
      let rateAxisGroup = this.g.append('g').call(rateAxis)
        .attr('id', 'rateAxis')
        .attr('transform', `translate(${this.innerWidth}, 0)`)
      rateAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 60)
        .attr('fill', '#333333')
        .text(this.rateAxisLabel)
        .attr('text-anchor', 'middle')
      rateAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      this.g.select('#rankAxis').remove()
      let rankAxisGroup = this.g.append('g').call(rankAxis)
        .attr('id', 'rankAxis')
        .attr('transform', `translate(${this.innerWidth}, ${this.innerHeight / 2})`)
      rankAxisGroup.selectAll('text')
        .attr('dx', 20)
      rankAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 80)
        .attr('fill', '#333333')
        .text(this.rankAxisLabel)
        .attr('text-anchor', 'middle')
      rankAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      /*
      *   ring
      */
      const ring = this.uiLayer.append('g')
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
      const tooltip = this.uiLayer.append('g')
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
      *   news board
      */
      const newsboard = this.uiLayer.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 12)
        .attr('text-anchor', 'start')

      newsboard
        .append('rect')
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#FFF')
        .attr('fill-opacity', 0.6)
        .attr('stroke', '#999')

      newsboard
        .append('text')
        .attr('class', 'title')
        .attr('dy', '1em')

      /*
      *   rate label
      */
      const rateLabel = this.front.append('g')
        .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 18)
        .attr('text-anchor', 'middle')

      rateLabel
        .append('text')
        .attr('font-size', 12)
        .attr('class', 'label1')
        .attr('dy', '0em')

      rateLabel
        .append('text')
        .attr('class', 'label2')
        .attr('dy', '1em')
        .style('font-weight', 'bold')

      /*
      *   rank Label
      */
      const rankLabel = this.front.append('g')
        .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
        .attr('opacity', 0)
        .attr('font-size', 18)
        .attr('text-anchor', 'middle')

      rankLabel
        .append('text')
        .style('font-weight', 'bold')
        .attr('class', 'label1')
        .attr('dy', '0em')

      rankLabel
        .append('text')
        .attr('font-size', 12)
        .attr('class', 'label2')
        .attr('dy', '1em')


      /*
      
            const newsCircle = this.front.select('#newscircle').selectAll('circle')
      
            newsCircle.data(news).exit().remove()
      
            var newscircleenter = newsCircle.data(news).enter()
              .append('circle')
              .attr('r', 4)
              .attr('fill', '#CCC')
              .attr('opacity', 0.6)
              .attr('cx', d => this.xScale(this.xValue(d)) + this.margin.left)
              .attr('cy', this.innerHeight + this.margin.top + 20)
      
            newsCircle.merge(newscircleenter).transition().ease(d3.easeLinear).duration(500)
              .attr('r', 4)
              .attr('cx', d => this.xScale(this.xValue(d)) + this.margin.left)
              .attr('cy', this.innerHeight + this.margin.top + 20)
              .attr('fill', '#CCC')
              .attr('opacity', 0.6)
      
            newsCircle
      */
      /*
      *   circle
      */

      this.circle = this.front.select('#maincircle').selectAll('circle')

      this.circle.data(data).exit().remove()

      var circleenter = this.circle.data(data).enter()
        .append('circle')
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => 0)
        .attr('fill', d => {
          if (d.result == 'win')
            if (d.sets > 2)
              return this.smallWinColor
            else
              return this.bigWinColor
          else
            if (d.sets > 2)
              return this.smallLoseColor
            else
              return this.bigLoseColor
        })
        .attr('fill-opacity', 0.6)

      this.circle.merge(circleenter)

      this.circle = this.front.select('#maincircle').selectAll('circle')

      this.circle
        .attr('fill', d => {
          if (d.result == 'win')
            if (d.sets > 2)
              return this.smallWinColor
            else
              return this.bigWinColor
          else
            if (d.sets > 2)
              return this.smallLoseColor
            else
              return this.bigLoseColor
        })
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => 0)
        .transition().ease(d3.easeLinear).duration(500)
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.yScale(this.yValue(d)))
        .attr('r', d => {
          return d.sets > 2 ? this.smallCircleR : this.bigCircleR
        })
        .attr('fill-opacity', 0.6)
      this.circle
        .on('mouseover', (d, k) => {
          ring
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.yScale(this.yValue(k)))
            .attr('r', () => {
              return k.sets > 2 ? this.smallCircleR + 2 : this.bigCircleR + 2
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
          this.circle
            .attr('fill-opacity', dd => {
              return dd.opponent == k.opponent ? 1 : 0.2
            })
          this.$emit('nodeClick', k)
        })



      /*
      *   news ring
      */
      const newsRing = this.front.append('g')
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
      *  news circle
      */
      this.front.select('#newscircle').remove()

      const newsCircle = this.front.append('g')
        .attr('id', 'newscircle')
        .selectAll('circle')
        .data(news)
        .join('circle')
        .attr('r', 4)
        .attr('cx', d => this.xScale(this.xValue(d)) + this.margin.left)
        .attr('cy', this.innerHeight + this.margin.top + 20)
        .attr('fill', '#CCC')
        .attr('opacity', 0.6)
        .on('mouseover', (d, k) => {
          newsboard.select('.title')
            .text(k.title)
            .attr('dx', this.tooltipMargin.left)
          var textLength = newsboard.select('.title').node().getComputedTextLength()
          newsboard
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k)) - textLength / 2}, ${this.margin.top + this.innerHeight - 30})`)
            .attr('opacity', 1)
            .select('rect')
            .attr('width', textLength + this.tooltipMargin.left + this.tooltipMargin.right)
            .attr('height', 14 + this.tooltipMargin.top + this.tooltipMargin.bottom)

          newsRing
            .attr('cx', this.xScale(this.xValue(k)))
            .attr('cy', this.innerHeight + 20)
            .attr('opacity', 1)
        })
        .on('mouseout', () => {
          newsboard
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          newsRing
            .attr('cx', -this.margin.left)
            .attr('cy', -this.margin.top)
            .attr('opacity', 0)
        })
        .on('click', (d, k) => {
          this.$emit('newsclick', k)
        })


      /*
      *   rate lines
      */
      const line = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rateScale(this.rateValue(d)))
        .curve(d3.curveLinear)

      this.g.select('#ratecurve')
        .datum(rateData)
        .attr('fill', 'none')
        .attr('stroke', '#339966')
        .attr('stroke-width', 1)
        .transition().ease(d3.easeLinear).duration(500)
        .attr('d', line(rateData))

      /*
      *   rate line nodes
      */
      const lineNodes = this.svg.select('#linenodes').selectAll('circle').data(rateData)
      lineNodes.exit().remove()

      var linenodesenter = lineNodes.enter()
        .append('circle')
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.rateScale(this.rateValue(d)))

      lineNodes.merge(linenodesenter).transition().ease(d3.easeLinear).duration(500)
        .attr('fill', '#339966')
        .attr('r', 3)
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.rateScale(this.rateValue(d)))

      /*
      *   rank curves
      */
      var LowestRank = Math.max.apply(Math, rankData.map(item => { return item.worldRanking }))
      if (rankData.length) {
        var earliestRank = rankData[0].date
        var latestRank = rankData[rankData.length - 1].date
      }


      rankData.splice(0, 0, {
        date: earliestRank,
        worldRanking: LowestRank,
        fake: true
      })

      rankData.push({
        date: latestRank,
        worldRanking: LowestRank,
        fake: true
      })

      const worldRankCurve = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .curve(d3.curveLinear)

      this.back.select('#worldrankback')
        .datum(rankData)
        .attr('fill', '#AAA')
        .attr('fill-opacity', 0.1)
        //.transition().ease(d3.easeLinear).duration(100)
        .attr('d', worldRankCurve(rankData))

      this.back.select('#worldrank')
        .datum(rankData)
        .attr('fill', 'none')
        .attr('stroke', '#CCC')
        .attr('stroke-width', 1)
        //.transition().ease(d3.easeLinear).duration(100)
        .attr('d', worldRankCurve(rankData.filter(d => !d.fake)))

      /*
      *   rank line nodes
      */
      const rankNodes = this.front.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .selectAll('circle')
        .data(rankData.filter(d => !d.fake))
        .join('circle')
        .attr('id', 'ranknodes')
        .attr('fill', '#000')
        .style('pointer-events', 'none')
        .attr('r', 4)
        .attr('cx', d => this.xScale(this.xValue(d)))
        .attr('cy', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .attr('opacity', 0)

      const rankNodeLines = this.front.append('g')
        .attr('transform', `translate(${this.margin.left}, ${this.margin.top})`)
        .selectAll('line')
        .data(rankData.filter(d => !d.fake))
        .join('line')
        .attr('id', 'ranknodelines')
        .attr('x1', d => this.xScale(this.xValue(d)))
        .attr('x2', d => this.xScale(this.xValue(d)))
        .attr('y1', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .attr('y2', this.innerHeight + 20)
        .attr('stroke', 'black')
        .style('pointer-events', 'none')
        .attr('opacity', 0)

      /*
      *   back rect
      */
      this.back.selectAll('g').remove()
      const rateRect = this.back.append('g')
        .selectAll('rect')
        .data(rateData)
        .join('rect')
        .attr('id', 'backrect')
        .attr('width', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          if (start < 0)
            start = 0
          date.setMonth(11)
          var end = this.xScale(date.setDate(31))
          if (end > this.innerWidth)
            end = this.innerWidth
          return end - start
        })
        .attr('height', this.innerHeight)
        .attr('x', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          return start > 0 ? start : 0
        })
        .attr('rx', 6)
        .attr('ry', 6)
        .attr('fill', '#DDD')
        .attr('fill-opacity', 0.4)
        .style('opacity', 0)
        .on('mouseover', (d, k) => {
          d.target.style.opacity = 1
          lineNodes
            .attr('r', dd => {
              return dd.date.getFullYear() == k.date.getFullYear() ? 5 : 3
            })
          rateLabel.select('.label1')
            .text(k.date.getFullYear() + ' Winning Rate')
          rateLabel.select('.label2')
            .text((k.winningRate * 100).toPrecision(3) + '%')
          rateLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k))}, ${this.margin.top - 20})`)
            .attr('opacity', 1)
        })
        .on('mouseout', d => {
          d.target.style.opacity = 0
          lineNodes
            .attr('r', 3)
          rateLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
        })

      /*
      *   Rank Rect
      */
      const rankRect = this.back.append('g')
        .selectAll('rect')
        .data(rankData.filter(d => !d.fake))
        .join('rect')
        .attr('id', 'rankrect')
        .attr('opacity', 0)
        .attr('width', 4)
        .attr('height', d => this.innerHeight / 2 - this.rankScale(d.worldRanking))
        .attr('r', 3)
        .attr('x', d => this.xScale(this.xValue(d)))
        .attr('y', d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .on('mouseover', (d, k) => {
          rankNodes
            .attr('opacity', dd => {
              return dd.date == k.date ? 1 : 0
            })
          lineNodes
            .attr('r', dd => {
              return dd.date.getFullYear() == k.date.getFullYear() ? 5 : 3
            })
          var rectDate = null
          rateRect
            .style('opacity', d => {
              if (d.date.getFullYear() == k.date.getFullYear()) {
                rectDate = d.date
                return 1
              }
              else
                return 0
            })
          rateLabel.select('.label1')
            .text(k.date.getFullYear() + ' Winning Rate')
          rateLabel.select('.label2')
            .text((rateData.filter(dd => dd.date.getFullYear() == k.date.getFullYear())[0].winningRate * 100).toPrecision(3) + '%')
          rateLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(rectDate)}, ${this.margin.top - 20})`)
            .attr('opacity', 1)

          rankLabel.select('.label1')
            .text('World Rank: ' + k.worldRanking)
          rankLabel.select('.label2')
            .text(k.dateStr)
          rankLabel
            .attr('transform', `translate(${this.margin.left + this.xScale(this.xValue(k))}, ${this.innerHeight + this.margin.top + 40})`)
            .attr('opacity', 1)
          rankNodeLines
            .attr('opacity', dd => {
              return dd.date == k.date ? 1 : 0
            })
        })
        .on('mouseout', () => {
          rankNodes
            .attr('opacity', 0)
          rateRect
            .style('opacity', 0)
          lineNodes
            .attr('r', 3)
          rateLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          rankLabel
            .attr('transform', `translate(${-this.margin.left}, ${-this.margin.top})`)
            .attr('opacity', 0)
          rankNodeLines
            .attr('opacity', 0)
        })

      this.svg
        .on('click', () => {
          if (ring.attr('opacity') == 0) {
            this.circle
              .attr('fill-opacity', 0.6)
            this.$emit('cancleClick')
          }

        })

    },

    widthupdate () {
      this.width = this.svgWidth

      this.svg
        .transition().ease(d3.easeLinear).duration(500)
        .attr('width', this.width)

      this.innerWidth = this.width - this.margin.left - this.margin.right
      this.innerHeight = this.height - this.margin.top - this.margin.bottom

      /*
      *   Scales Justify
      */
      this.xScale
        .range([0, this.innerWidth])
      this.yScale
        .range([0, this.innerHeight])
        .nice()
      this.rateScale
        .range([0, this.innerHeight / 2])
      this.rankScale
        .range([0, this.innerHeight / 2])

      /*
      *   Axis Defined
      */
      const yAxis = d3.axisLeft(this.yScale)
        .ticks(6)
      const rateAxis = d3.axisRight(this.rateScale)
      const rankAxis = d3.axisRight(this.rankScale)
      const xAxis = d3.axisBottom(this.xScale)
        .tickSize(-6)

      /*
      *   Axis attached
      */
      this.g.select('#xaxis').remove()
      let xAxisGroup = this.g.append('g').call(xAxis)
        .attr('transform', `translate(0, ${this.innerHeight})`)
        .attr('id', 'xaxis')

      this.g.select('#yaxis').remove()
      let yAxisGroup = this.g.append('g').call(yAxis)
        .attr('id', 'yaxis')
      yAxisGroup.selectAll('line')
        .attr('x1', 0)
        .attr('x2', this.innerWidth)
        .attr('stroke', '#666')
        .attr('stroke-dasharray', '5,5')
        .attr('stroke-width', 1)
      yAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 2)
        .attr('y', -40)
        .attr('fill', '#333333')
        .text(this.yAxisLable)
        .attr('text-anchor', 'middle')
      yAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      this.g.select('#rateAxis').remove()
      let rateAxisGroup = this.g.append('g').call(rateAxis)
        .attr('id', 'rateAxis')
        .attr('transform', `translate(${this.innerWidth}, 0)`)
      rateAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 60)
        .attr('fill', '#333333')
        .text(this.rateAxisLabel)
        .attr('text-anchor', 'middle')
      rateAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      this.g.select('#rankAxis').remove()
      let rankAxisGroup = this.g.append('g').call(rankAxis)
        .attr('id', 'rankAxis')
        .attr('transform', `translate(${this.innerWidth}, ${this.innerHeight / 2})`)
      rankAxisGroup.selectAll('text')
        .attr('dx', 20)
      rankAxisGroup.append('text')
        .attr('font-size', '2em')
        .attr('transform', 'rotate(-90)')
        .attr('x', -this.innerHeight / 4)
        .attr('y', 80)
        .attr('fill', '#333333')
        .text(this.rankAxisLabel)
        .attr('text-anchor', 'middle')
      rankAxisGroup.selectAll('.domain').remove() // 去掉坐标轴实线

      this.circle
        .transition().ease(d3.easeLinear).duration(500)
        .attr('cx', d => this.xScale(this.xValue(d)))

      /*
      *  news circle
      */
      const newsCircle = this.front.select('#newscircle').selectAll('circle')

      newsCircle
        .attr('cx', d => this.xScale(this.xValue(d)) + this.margin.left)

      /*
      *   rate lines !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
      */
      const line = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rateScale(this.rateValue(d)))
        .curve(d3.curveLinear)

      this.g.select('#ratecurve')
        .transition().ease(d3.easeLinear).duration(500)
        .attr('d', d => line(d))

      /*
      *   rate line nodes
      */
      this.svg.select('#linenodes').selectAll('circle')
        .transition().ease(d3.easeLinear).duration(500)
        .attr('cx', d => this.xScale(this.xValue(d)))

      /*
      *   rank curves
      */
      const worldRankCurve = d3.line()
        .x(d => this.xScale(this.xValue(d)))
        .y(d => this.rankScale(d.worldRanking) + this.innerHeight / 2)
        .curve(d3.curveLinear)

      //var rankData = this.back.select('#worldrankback').datum()
      this.back.select('#worldrankback')
        .attr('d', d => worldRankCurve(d))

      this.back.select('#worldrank')
        .attr('d', d => worldRankCurve(d.filter(item => !item.fake)))

      /*
      *   rank line nodes
      */
      this.front
        .selectAll('#ranknodes')
        .attr('cx', d => this.xScale(this.xValue(d)))

      this.front
        .selectAll('#ranknodelines')
        .attr('x1', d => this.xScale(this.xValue(d)))
        .attr('x2', d => this.xScale(this.xValue(d)))

      /*
      *   back rect
      */
      this.back
        .selectAll('#backrect')
        .attr('width', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          if (start < 0)
            start = 0
          date.setMonth(11)
          var end = this.xScale(date.setDate(31))
          if (end > this.innerWidth)
            end = this.innerWidth
          return end - start
        })
        .attr('x', d => {
          var date = new Date(d.date)
          date.setMonth(0)
          var start = this.xScale(date.setDate(1))
          return start > 0 ? start : 0
        })

      /*
      *   Rank Rect
      */
      this.back
        .selectAll('#rankrect')
        .attr('x', d => this.xScale(this.xValue(d)))
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.svg-wrap {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}

.slider {
  height: 80px;
  width: 1000px;
}
</style>
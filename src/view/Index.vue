<template>
  <div class="wrap"
       style="min-width: 400px;">
    <el-backtop target=".main-scrollbar .el-scrollbar__wrap"></el-backtop>
    <el-container>
      <el-header class="headerBar"
                 style="height: 40px;">
        <el-switch v-model="asideBarSwitchValue"
                   active-text="Head to Head">
        </el-switch>
        <div style="font-size: 32px;">Badminton Stars Career Path Visualization</div>
        <el-switch v-model="consoleSwitchValue"
                   active-text="Console">
        </el-switch>
      </el-header>
      <el-container style="height: 90vh;">
        <el-aside class="asideBar"
                  id="asideBar"
                  width="400px">
          <el-scrollbar id="aside-scroll"
                        style="height: 100%;">
            <div class="aside-wrap">
              <HeadtoHead ref="h2h"
                          :matchData="figureData[currentFigure].matchData"
                          @nodeClick="h2hClick"
                          @cancleClick="cancleClick"
                          @highlight="scroll" />
            </div>
          </el-scrollbar>
        </el-aside>
        <el-main class="main"
                 id="main">
          <el-scrollbar ref="scrollbar"
                        class="main-scrollbar"
                        id="main-scrollbar"
                        style="height: 100%;">
            <div class="avatar-wrap row-wrap">
              <img class="left-img"
                   src="../../static/images/0.-HSBC-BWF-World-Tour-Inside-Pagec.png" />
              <img class="right-img"
                   id="right-img"
                   :src="figureData[currentFigure].info.backImg" />
              <img class="bottom-img"
                   src="../../static/images/0.-HSBC-BWF-World-Tour-Inside-Page.png" />
              <img class="bwf-img"
                   src="../../static/images/BWF-Logo-Text-Sizedw.png" />
              <div class="left-avatar-wrap row-wrap"
                   id="left-avatar-wrap">
                <div class="figure-img"
                     id="figure-img">
                  <img :src="figureData[currentFigure].info.figureImg" />
                </div>
                <div class="info-wrap"
                     id="figure-info-wrap">
                  <div class="info-title">{{ figureData[currentFigure].name }}</div>
                  <div class="info-tag">personal information</div>
                  <div class="info-rows">
                    <div class="info-details row-wrap"
                         v-for="(item, index) in figureData[currentFigure].info.info"
                         :key="index">
                      <div class="info-item-key">{{item.item}}</div>
                      <div class="info-item-value">{{item.value}}</div>
                    </div>
                  </div>
                  <div class="info-tag">brief introduction</div>
                  <div class="info-intro">{{ figureData[currentFigure].info.intro }}</div>

                </div>
              </div>
              <div class="right-avatar-wrap"></div>
              <div class></div>
            </div>

            <div style="height: 450px; position: relative;">
              <el-link class="right-arrow"
                       id="right-arrow"
                       :underline="false"
                       @click="changeFigure"><i class="el-icon-arrow-right"></i></el-link>
            </div>
            <div class="main-wrap">
              <Career ref="career"
                      @newsclick="newsClick"
                      :matchData="figureData[currentFigure].matchData"
                      :rankingData="figureData[currentFigure].rankingData"
                      :careerNews="figureData[currentFigure].careerNews"
                      :winningRate="winningRate"
                      @nodeClick="careerClick"
                      @cancleClick="cancleClick"
                      :svgWidth="svgWidth" />
            </div>
          </el-scrollbar>
        </el-main>
        <el-aside class="console"
                  id="console"
                  width="0">
          <el-scrollbar id="aside-scroll"
                        style="height: 100%;">
            <div class="aside-wrap">
              <div class="console-title">Console Panel</div>
              <div class="row-wrap">
                <div>Filter</div>
                <el-switch v-model="filterSwitch">
                </el-switch>
              </div>
              <div class="row-wrap">
                <el-checkbox-group v-model="filterList">
                  <el-divider>Single Match</el-divider>
                  <el-checkbox v-for="item in singleMatch"
                               :key="item"
                               :label="item"
                               :disabled="!filterSwitch"></el-checkbox>
                  <el-divider>Group Match</el-divider>
                  <el-checkbox v-for="item in groupMatch"
                               :key="item"
                               :label="item"
                               :disabled="!filterSwitch"></el-checkbox>
                </el-checkbox-group>
              </div>
              <div style="margin-top: 20px;"></div>

              <div class="console-title">Information Panel</div>

              <div class="h2h-info console-card"
                   v-if="h2hInfo.player">

                <div class="row-wrap">
                  <div class="player">
                    <div class="player-name">{{ h2hInfo.player }}</div>
                    <div class="player-win">{{ h2hInfo.win }}</div>
                  </div>
                  <div>VS</div>
                  <div class="opponent">
                    <div class="opponent-name">{{ h2hInfo.opponent }}</div>
                    <div class="opponent-lose">{{ h2hInfo.lose }}</div>
                  </div>
                </div>
              </div>
              <div v-if="tournamentTotal.length">
                <div class="h2h-info console-card"
                     v-for="(item, index) in tournamentTotal"
                     :key="index">
                  <div class="tournament-title">{{ item.title }}</div>
                  <div class="row-wrap" style="margin: 0 20px;">
                    <div class="player">
                      <div class="player-name">Win</div>
                      <div class="player-win">{{ item.win }}</div>
                    </div>
                    <div class="opponent">
                      <div class="opponent-name">Lose</div>
                      <div class="opponent-lose">{{ item.lose }}</div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="news-info console-card"
                   v-if="newsDetail.title">
                <div class="news-title">{{ newsDetail.title }}</div>
                <div class="news-date">{{ newsDetail.dateStr }}</div>
                <div class="row-wrap news-wrap">
                  <img class="news-img"
                       v-if="newsDetail.imgSrc"
                       :src="newsDetail.imgSrc" />
                  <div class="news-content">{{ newsDetail.content }}</div>
                </div>
              </div>

              <div :class="'tournament-info tournament-'+item.result"
                   v-for="(item, index) in tournamentInfo"
                   :key="index">
                <div class="row-wrap flex-wrap">
                  <div class="tournament-title">{{ item.tournament }}</div>
                  <div class="tournament-date">{{item.dateStr}}</div>
                </div>
                <div class="row-wrap">
                  <div class="tournament-player">{{item.player}}</div>
                  <div class="tournament-score">{{item.score}}</div>
                  <div class="tournament-player">{{item.opponent}}</div>
                </div>
              </div>
              <div style="margin-top: 40px;"></div>
            </div>
          </el-scrollbar>
        </el-aside>
      </el-container>
    </el-container>
    <el-footer class="footer row-wrap">
      <div>Badminton Stars Career Path Visualization | Visualization for Information & Cross Media 2020 | all right reserved @ LuniumLuk</div>
    </el-footer>
  </div>
</template>

<script>
import Career from '../components/Career'
import HeadtoHead from '../components/HeadtoHead'
import '../../static/js/data.js'
import '../../static/js/momota-match.js'
import '../../static/js/momota-ranking.js'
import '../../static/js/momota-news.js'
import '../../static/js/axelsen-match.js'
import '../../static/js/axelsen-ranking.js'
import '../../static/js/lindan-match.js'
import '../../static/js/lindan-ranking.js'
import '../../static/js/leechongwei-match.js'
import '../../static/js/leechongwei-ranking.js'
import '../../static/js/shiyuqi-match.js'
import '../../static/js/shiyuqi-ranking.js'

export default {
  data () {
    return {
      currentFigure: 0,
      winningRate: [],
      asideBarSwitchValue: true,
      consoleSwitchValue: false,
      svgWidth: 500,
      filterSwitch: false,
      singleMatch: [
        'CHINA OPEN', 'INDONESIA OPEN', 'ALL ENGLAND OPEN', 'WORLD CHAMPIONSHIPS', 'OLYMPIC GAMES', 'SUPERSERIES FINALS'
      ],
      groupMatch: [
        'SUDIRMAN CUP', 'THOMAS & UBER CUP'
      ],
      filterList: [],
      tournamentInfo: [],
      h2hInfo: {},
      svgSelected: false,
      tournamentTotal: [],
      newsDetail: {},
      figureData: [
        {
          name: 'Kento MOMOTA',
          matchData: MOMOTA_MATCH,
          rankingData: MOMOTA_RANKING,
          careerNews: MOMOTA_NEWS,
          info: FIGURE_DATA.momota
        },
        {
          name: 'Viktor AXELSEN',
          matchData: AXELSEN_MATCH,
          rankingData: AXELSEN_RANKING,
          careerNews: [],
          info: FIGURE_DATA.axelsen
        },
        {
          name: 'Lin DAN',
          matchData: LINDAN_MATCH,
          rankingData: LINDAN_RANKING,
          careerNews: [],
          info: FIGURE_DATA.lindan
        },
        {
          name: 'Lee CHONGWEI',
          matchData: LEECHONGWEI_MATCH,
          rankingData: LEECHONGWEI_RANKING,
          careerNews: [],
          info: FIGURE_DATA.leechongwei
        },
        {
          name: 'Shi YUQI',
          matchData: SHIYUQI_MATCH,
          rankingData: SHIYUQI_RANKING,
          careerNews: [],
          info: FIGURE_DATA.shiyuqi
        }
      ]
    }
  },
  components: {
    Career,
    HeadtoHead
  },
  created () {
    this.processData()
    var that = this
    window.onresize = function () {
      var svgDiv = document.getElementById("svg-wrap")
      that.svgWidth = svgDiv.clientWidth || svgDiv.offsetWidth
    }
  },
  mounted () {
    var svgDiv = document.getElementById("svg-wrap")
    this.svgWidth = svgDiv.clientWidth || svgDiv.offsetWidth

    let scrollbarEl = this.$refs.scrollbar.wrap
    scrollbarEl.onscroll = () => {
      if (scrollbarEl.scrollTop > 0)
        document.getElementById('right-arrow').style.right = '-200px'
      else
        document.getElementById('right-arrow').style.right = '0'
    }
  },
  watch: {
    'asideBarSwitchValue': {
      handler () {
        setTimeout(() => {
          var svgDiv = document.getElementById("svg-wrap")
          this.svgWidth = svgDiv.clientWidth || svgDiv.offsetWidth
        }, 500)
        if (this.asideBarSwitchValue == false)
          document.getElementById('asideBar').style.width = 0
        else {
          document.getElementById('asideBar').style.width = '400px'
          //this.consoleSwitchValue = false
        }
      }
    },
    'consoleSwitchValue': {
      handler () {
        setTimeout(() => {
          var svgDiv = document.getElementById("svg-wrap")
          this.svgWidth = svgDiv.clientWidth || svgDiv.offsetWidth
        }, 500)
        if (this.consoleSwitchValue == false)
          document.getElementById('console').style.width = 0
        else {
          document.getElementById('console').style.width = '400px'
          //this.asideBarSwitchValue = false
        }
      }
    },
    'filterSwitch': {
      handler () {
        if (this.filterSwitch) {
          this.$refs.career.circleFilter(this.filterList)
          this.$refs.h2h.cancleHighlight()
          this.tournamentInfo = this.figureData[this.currentFigure].matchData.filter(d => {
            var picked = false
            this.filterList.forEach(item => {
              if (d.tournament.indexOf(item) != -1)
                picked = true
            })
            return picked
          }).sort((a, b) => a.date - b.date)
          this.tournamentTotal = []
          this.filterList.forEach(item => {
            var win = 0
            var lose = 0
            this.tournamentInfo.forEach(d => {
              if (d.tournament.indexOf(item) != -1) {
                if (d.result == 'win')
                  win += 1
                else
                  lose += 1
              }
            })
            this.tournamentTotal.push({
              'title': item,
              'win': win,
              'lose': lose
            })
          })
          this.h2hInfo = {}
        } else {
          if (!this.svgSelected) {
            this.$refs.career.cancleHighlight()
            this.tournamentInfo = []
          }
        }

      }
    },
    'filterList': {
      handler () {
        if (this.filterSwitch)
          this.$refs.career.circleFilter(this.filterList)
        this.tournamentInfo = this.figureData[this.currentFigure].matchData.filter(d => {
          var picked = false
          this.filterList.forEach(item => {
            if (d.tournament.indexOf(item) != -1)
              picked = true
          })
          return picked
        }).sort((a, b) => a.date - b.date)
        this.tournamentTotal = []
        this.filterList.forEach(item => {
          var win = 0
          var lose = 0
          this.tournamentInfo.forEach(d => {
            if (d.tournament.indexOf(item) != -1) {
              if (d.result == 'win')
                win += 1
              else
                lose += 1
            }
          })
          this.tournamentTotal.push({
            'title': item,
            'win': win,
            'lose': lose
          })
        })
      }
    },
    'tournamentTotal': {
      handler () {
        if (this.tournamentTotal.length) {
          this.h2hInfo = {}
          this.newsDetail = {}
        }
      }
    },
    'h2hInfo': {
      handler () {
        if (this.h2hInfo.player) {
          this.tournamentTotal = []
          this.newsDetail = {}
        }
      }
    }
  },
  methods: {
    processData () {

      this.figureData[this.currentFigure].matchData.forEach(item => {
        var netPoints = 0
        var margin = 0
        item.sets = 0
        if (item.score == '')
          item.net = 0
        else {
          item.score.split(', ').forEach(d => {
            var point = Number(d.split('-')[0]) - Number(d.split('-')[1])
            margin += point > 0 ? 1 : -1
            netPoints += point
            item.sets += 1
          })
          item.net = netPoints
        }
        if (item.result == 'win' && margin < 0) {
          item.net = -item.net
        }
        if (item.result == 'lose' && margin > 0) {
          item.net = -item.net
        }
        item.date = new Date(item.date)
        item.dateStr = item.date.toDateString()
      })

      let allYears = Array.from(new Set(this.figureData[this.currentFigure].matchData.map(d => d.date.getFullYear())))
      let yearWinningRate = {}
      allYears.forEach(item => {
        yearWinningRate[item] = {
          date: new Date(item, 5, 30),
          totalGames: 0,
          win: 0,
          lose: 0,
          winningRate: 0
        }
      })

      this.figureData[this.currentFigure].matchData.forEach(item => {
        if (item.result == 'win') {
          yearWinningRate[item.date.getFullYear()].win += 1
        } else {
          yearWinningRate[item.date.getFullYear()].lose += 1
        }
      })

      this.winningRate = []

      for (var key in yearWinningRate) {
        var item = yearWinningRate[key]
        item.totalGames = item.win + item.lose
        item.winningRate = item.win / item.totalGames
        this.winningRate.push(item)
      }

      this.figureData[this.currentFigure].rankingData.forEach(item => {
        item.date = new Date(item.date)
        item.dateStr = item.date.toLocaleDateString()
      })

      const months = {
        'Jan': 0, 'Feb': 1, 'Mar': 2, 'Apr': 3, 'May': 4, 'Jun': 5,
        'Jul': 6, 'Aug': 7, 'Sep': 8, 'Oct': 9, 'Nov': 10, 'Dec': 11
      }
      this.figureData[this.currentFigure].careerNews.forEach(item => {
        if (typeof (item.date) == 'object')
          return
        var date = item.date.replace(',', '').split(' ')
        item.date = new Date(date[2] + '-' + (months[date[0]] + 1) + '-' + date[1])
        item.dateStr = item.date.toLocaleDateString()
      })
    },

    h2hClick (item) {
      this.svgSelected = true
      this.filterSwitch = false
      this.$refs.career.highlight(item)
      this.tournamentInfo = this.figureData[this.currentFigure].matchData.filter(d => d.opponent == item.opponent)
      this.h2hInfo = this.$refs.h2h.getOpponentRecord(item)
    },

    careerClick (item) {
      this.svgSelected = true
      this.filterSwitch = false
      this.$refs.h2h.highlight(item)
      this.tournamentInfo = this.figureData[this.currentFigure].matchData.filter(d => d.opponent == item.opponent)
      this.h2hInfo = this.$refs.h2h.getOpponentRecord(item)
    },

    cancleClick () {
      this.svgSelected = false
      this.filterSwitch = false
      this.$refs.h2h.cancleHighlight()
      this.$refs.career.cancleHighlight()
      this.tournamentInfo = []
      this.h2hInfo = {}
    },

    newsClick (item) {
      this.newsDetail = item
    },

    scroll (top) {
      document.getElementById('aside-scroll').getElementsByClassName('el-scrollbar__wrap')[0].scrollTop = top
    },

    changeFigure () {
      var that = this
      var nextFigure = 0
      this.tournamentTotal = []
      this.tournamentInfo = []
      this.newsDetail = {}
      this.h2hInfo = {}
      if (this.currentFigure < this.figureData.length - 1)
        nextFigure = this.currentFigure + 1
      else
        nextFigure = 0
      this.filterSwitch = false
      document.getElementById('right-img').style.opacity = 0
      document.getElementById('left-avatar-wrap').style['margin-left'] = '100vw'
      setTimeout(() => {
        that.currentFigure = nextFigure
        that.processData()
        document.getElementById('right-img').style.opacity = 1
        document.getElementById('left-avatar-wrap').style.transition = 'ease 0ms'
        document.getElementById('left-avatar-wrap').style['margin-left'] = '-100vw'
        setTimeout(() => {
          document.getElementById('left-avatar-wrap').style.transition = 'ease 500ms'
          document.getElementById('left-avatar-wrap').style['margin-left'] = '0'
          that.$refs.career.figureupdate()
          that.$refs.h2h.figureupdate()
        }, 50)
      }, 500)
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
@import url("../../static/css/main.css");

.el-scrollbar__wrap {
  overflow-x: hidden;
  height: calc(100% + 17px);
}

.el-scrollbar__bar.is-horizontal {
  display: none;
}

.el-divider--horizontal {
  margin: 10px 0 20px 0;
}
</style>
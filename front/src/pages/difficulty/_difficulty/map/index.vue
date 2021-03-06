// マップ選択
// params.difficulty(easy, normal, hard)に対応するマップの一覧を表示する
<template lang="pug">
  .page
    .return
      button(@click="pushDifficulty") ←レベル選択に戻る
    .map-area
      .map-list
        div(v-for="(map, index) in mapList", :key="index")
          puzzle-map(
            :map-name="map.name",
            :map-image="`${map.kind}/${map.z}-${map.x}-${map.y}-${map.n}/sar`",
            @onClick="pushPuzzle(map.id)")
    jaxa.jaxa
</template>

<script>
import PuzzleMap from '~/components/map/MapImage'
import Jaxa from '~/components/Jaxa'
import { mapGetters } from 'vuex';
export default {
  components: {
    PuzzleMap,
    Jaxa
  },
  data () {
    return {
      difficulty: 'normal'
    }
  },
  asyncData(context) {
    return {
      // asyncDataでreturnすると、dataにマージされる
      difficulty: context.params.difficulty
    }
  },
  validate({ params }) {
    // 難易度チェック(easy, normal, hard以外は404)
    const d = params.difficulty
    return (d === 'easy' || d === 'normal' || d === 'hard')
  },
  methods: {
    pushPuzzle (id) { // 選択した難易度・マップに対応するパズル画面に遷移する
      this.$router.push(`/difficulty/${this.difficulty}/map/${id}`)
    },
    pushDifficulty () { // 難易度選択画面に遷移する
      this.$router.push('/difficulty')
    },
  },
  mounted () {
    const deviceType = this.$ua.deviceType()
    // PCの場合はスクロール調整を行わない
    if (deviceType === 'pc') {
      return
    }

    var mapList = this.$el.getElementsByClassName('map-list')

    // マップリストのスクロール処理
    mapList[0].addEventListener('scroll', function(event) {
     if (mapList[0].scrollTop === 0) {
        mapList[0].scrollTop = 1;
      }
      else if (mapList[0].scrollTop + mapList[0].clientHeight === mapList[0].scrollHeight) {
        mapList[0].scrollTop = mapList[0].scrollTop - 1;
      }
    },{ passive: false })
  },
  computed: {
    mapList() { // 表示するマップのリストを作成する
      const difficultyMap =  { // 難易度と画像分割数との対応
        easy: 3,
        normal: 4,
        hard: 5,
      }
      const d = this.difficulty // 難易度選択画面で選択した難易度
      const puzzles = this.puzzles // storeからパズルの設定を取得
      // 指定された難易度の画像をもつObjectを取得
      const list =  puzzles.filter(puzzle => {
        return puzzle.parameters.find(parameter => {
          return parameter.split_n === difficultyMap[d]
        })
      })
      let target = []
      // Objectから必要な値を取得
      list.forEach(e => {
        const p = e.parameters.filter(v => {
          return v.split_n === difficultyMap[d]
        })
        target.push({
          id: e.id,
          name: e.name,
          kind: p[0].kind,
          x: p[0].x,
          y: p[0].y,
          z: p[0].z,
          n: p[0].split_n
        })
      })

      return target
    },
    ...mapGetters(['puzzles'])
  }
}
</script>


<style lang="scss" scoped>
.page {
  margin-left: auto;
  margin-right: auto;

  .return {
    margin-left: 34px;
    padding-top: 42px;
    height: 24px;
    color: #fff;
    text-align: left;
    letter-spacing: normal;
    font-weight: bold;
    font-style: normal;
    font-stretch: normal;
    font-size: 24px;
    font-family: NotoSansCJKjp;
    line-height: 1.5;
  }

  .map-area {
    margin-top: 90px;

    .map-list {
      overflow: auto;
      width: 640px;
      height: 1000px;

      .puzzle-map {
        margin-right: auto;
        margin-bottom: 82px;
        margin-left: auto;
      }
    }
  }

  .jaxa {
    position: absolute;
    top: 1110px;
    left: 530px;
    z-index: 10;
    color: #5b6277;
  }
}
</style>

<template lang="pug">
  v-container(fluid grid-list-md)
    div.headline
      template(v-if="day === 0") 企業
      template(v-else) {{ day }}日目
    span {{ filteredRequestedCircleCount }}サークル (壁:{{filteredRequestedWallCircleCount}}, シャッター:{{filteredRequestedShutterCircleCount}})
    v-layout(row wrap)
      v-flex
        v-checkbox(label="通常" v-model="filter.normal")
      v-flex
        v-checkbox(label="壁" v-model="filter.wall")
      v-flex
        v-checkbox(label="シャッター" v-model="filter.shutter")
      v-flex(xs12 sm12)
        v-autocomplete(label="ジャンプ" hide-no-data hide-selected :item-text="v => `${v.locationString} ${v.name} ${v.author}`" item-value="id" clearable return-object placeholder="サークル名または作家名を入力" :items="filteredRequests" v-model="jumpSelectedCircle" append-outer-icon="navigation" @click:append-outer="jumpCircle")

    v-layout(row wrap)
      v-progress-linear(v-if="$apollo.queries.fetchData.loading" indeterminate)
      v-flex(v-else xs12 sm12 md6 lg4 v-for="circle in filteredRequests" :key="circle.id" :id="`list-circle-${circle.id}`")
        v-card
          v-card-title.headline.lighten-4(:class="[{'orange': circle.locationType === 1}, {'red': circle.locationType === 2}, {'green': circle.locationType === 0}]")
            router-link(:to="`/circles/${circle.id}`" style="text-decoration: none;") {{ circle.locationString }} {{ circle.name }}
          v-card-text.blue-grey.lighten-5(v-if="circle.prioritized.length > 0")
            div(v-for="p in circle.prioritized" :key="p.userId" class="black--text")
              | 第{{p.rank}}希望：
              router-link(:to="`/planning/users/${p.userId}`") {{ p.user.displayName }}
          v-divider
          v-list(three-line)
            v-list-tile(v-for="item in circle.requestedItems" :key="item.id")
              v-list-tile-content
                v-list-tile-title {{ item.name }}
                v-list-tile-sub-title {{ priceString(item.price) }} × 計{{ requestedNum(item.requests) }}個
                v-list-tile-sub-title
                  span(v-for="(request, idx) in item.requests" :key="request.id")
                    span
                      router-link(:to="`/planning/users/${request.userId}`") {{ request.user.displayName }}
                      | ({{ request.num }})
                    template(v-if="idx !== item.requests.length - 1") ,&nbsp;
    v-fade-transition
      v-btn(fixed dark fab bottom right color="blue darken-2" @click="$vuetify.goTo(0)")
        v-icon arrow_upward

</template>

<script>
import gql from 'graphql-tag'

const getData = gql`
  query($day: Int!) {
    requestedCircles(day: $day) {
      id
      name
      author
      day
      locationString
      locationType
      prioritized {
        userId
        user {
          displayName
        }
        rank
      }
      requestedItems {
        id
        name
        price
        requests {
          id
          userId
          user {
            displayName
          }
          num
        }
      }
    }
  }
`

export default {
  name: 'AllRequestList',
  props: {
    day: {
      type: Number,
      default: 0
    }
  },
  data: function () {
    return {
      fetchData: {
        requestedCircles: []
      },
      filter: {
        normal: true,
        wall: true,
        shutter: true
      },
      jumpSelectedCircle: null
    }
  },
  apollo: {
    fetchData: {
      query: getData,
      fetchPolicy: 'cache-and-network',
      variables: function () {
        return {
          day: this.day
        }
      },
      update: data => data
    }
  },
  computed: {
    filteredRequests: function () {
      return this.fetchData.requestedCircles.filter(v => {
        let ok = true
        switch (v.locationType) {
          case 0:
            ok = this.filter.normal
            break
          case 1:
            ok = this.filter.wall
            break
          case 2:
            ok = this.filter.shutter
            break
        }
        return ok
      })
    },
    filteredRequestedCircleCount: function () {
      return this.filteredRequests.length
    },
    filteredRequestedWallCircleCount: function () {
      return this.filteredRequests.reduce((x, y) => x + (y.locationType === 1 ? 1 : 0), 0)
    },
    filteredRequestedShutterCircleCount: function () {
      return this.filteredRequests.reduce((x, y) => x + (y.locationType === 2 ? 1 : 0), 0)
    }
  },
  methods: {
    priceString: p => p >= 0 ? `${p}円` : '価格未登録',
    requestedNum: requests => requests.reduce((x, y) => x + y.num, 0),
    jumpCircle () {
      if (this.jumpSelectedCircle) this.$vuetify.goTo(`#list-circle-${this.jumpSelectedCircle.id}`, { offset: -70 })
    }
  }
}
</script>

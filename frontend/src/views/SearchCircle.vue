<template lang="pug">
  v-container(fluid v-scroll="onScroll")
    v-text-field(:disabled="$apollo.queries.circles.loading" :loading="$apollo.queries.circles.loading" append-icon="search" v-model="query" placeholder="企業/サークル/作家名を入力")
    v-layout(row wrap)
      v-checkbox(label="1日目" :value="1" v-model="filterDays")
      v-checkbox(label="2日目" :value="2" v-model="filterDays")
      v-checkbox(label="企業"  :value="0" v-model="filterDays")
    v-container(fluid grid-list-md)
      v-data-iterator(:items="circles" :loading="$apollo.queries.circles.loading" content-tag="v-layout" row wrap hide-actions)
        v-flex(slot="item" slot-scope="props" xs12 sm6 md4 lg3)
          v-card.lighten-5(:to="`/circles/${props.item.id}`" :class="[{'blue': props.item.day === 1}, {'teal': props.item.day === 2}, {'lime': props.item.day === 3}, {'orange': props.item.day === 0}]")
            v-card-text(:class="black--text")
              div.caption {{ props.item.locationString }}
              div(v-if="props.item.day !== 0") {{ props.item.name }} - {{ props.item.author }}
              div(v-else) {{ props.item.name }}
        p(slot="no-data") 見つかりませんでした
    v-fade-transition
      v-btn(fixed dark fab bottom right color="blue darken-2" @click="$vuetify.goTo(0)" v-show="offsetTop > 100")
        v-icon arrow_upward

</template>

<script>
import gql from 'graphql-tag'

const searchgql = gql`
  query ($q: String!, $days: [Int!]) {
    circles(q: $q, days: $days) {
      id
      day
      name
      author
      locationString(day: true)
    }
  }
`

export default {
  name: 'SearchCircle',
  data: function () {
    return {
      offsetTop: 0,
      query: '',
      filterDays: [0, 1, 2, 3, 4],
      circles: []
    }
  },
  apollo: {
    circles: {
      query: searchgql,
      debounce: 1000,
      variables: function () {
        return {
          q: this.query,
          days: this.filterDays
        }
      },
      skip: function () { return this.query === '' },
      update: data => data.circles
    }
  },
  methods: {
    onScroll () {
      this.offsetTop = window.pageYOffset || document.documentElement.scrollTop
    }
  }
}
</script>

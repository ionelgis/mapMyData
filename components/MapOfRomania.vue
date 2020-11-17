<template>
  <svg
    baseprofile="tiny"
    fill="#7c7c7c"
    height="705"
    stroke="#ffffff"
    stroke-linecap="round"
    stroke-linejoin="round"
    stroke-width="2"
    version="1.2"
    viewbox="0 0 1000 705"
    width="1000"
    xmlns="http://www.w3.org/2000/svg"
  >
    <g v-for="path in paths" :key="path.id">
      <path
        :id="path.id"
        :d="path.d"
        :name="path.name"
        :fill="getColor(path.id)"
      />
      <text font-size="15" fill="blue">hre</text>
    </g>
    <circle id="0" cx="701.8" cy="293.1" />
    <circle id="1" cx="589.2" cy="57.6" />
    <circle id="2" cx="416.9" cy="151.6" />
  </svg>
</template>

<script>
import mockData from 'assets/mockData'
import { scaleLinear } from 'd3-scale'
import { color as d3Color } from 'd3-color'
import paths from '../assets/RomaniaSvgPaths'

export default {
  name: 'MapOfRomania',
  props: {
    mapData: {
      type: Array,
      required: true
    },
    colors: {
      type: Array,
      required: true
    },
    pivot: {
      type: [Number, String],
      default: 'average'
    },
    noDataColor: {
      type: String,
      default: 'silver'
    }
  },
  data: () => ({
    paths,
    mockData,
    range: {
      min: 0,
      max: 0,
      average: 0
    }
  }),
  computed: {
    minValue () {
      return 100000
    }
  },
  created () {
    const sorted = this.mockData.filter(item => item.value).map(({ value }) => value).sort(this.compare)
    this.range.min = sorted[0]
    this.range.max = sorted[sorted.length - 1]
    // eslint-disable-next-line no-return-assign
    this.range.average = sorted.reduce((acc, val) => acc += val, 0) / sorted.length
  },
  methods: {
    compare (a, b) {
      if (a < b) {
        return -1
      }
      if (a > b) {
        return 1
      }
      return 0
    },
    getColor (id) {
      const foundData = this.mockData.find(item => item.id === id)
      if (foundData && foundData.value) {
        if (this.colors.length === 1) {
          return this.singleColor(this.colors[0], foundData.value)
        } else if (this.colors.length === 2) {
          return this.simpleRangeColor(this.colors, foundData.value)
        } else if (this.colors.length === 3) {
          const pivot = this.pivot && this.pivot !== 'average' ? this.pivot : this.range.average
          return this.pivotRangeColor(this.colors, pivot, foundData.value)
        }
      } else {
        return this.noDataColor
      }
    },
    singleColor (color, value) {
      const scaledColor = scaleLinear()
        .domain([this.range.min, this.range.max])
        .range([0.1, 1])

      const scColor = d3Color(color)
      scColor.opacity = scaledColor(value)
      return scColor + ''
    },
    simpleRangeColor (rangeColors, value) {
      const scaledColor = scaleLinear()
        .domain([this.range.min, this.range.max])
        .range(rangeColors)
      return scaledColor(value)
    },
    pivotRangeColor (rangeColors, pivot, value) {
      const scaledColor = scaleLinear()
        .domain([this.range.min, pivot, this.range.max])
        .range(rangeColors)
      return scaledColor(value)
    }
  }
}
</script>

<style scoped>

</style>

<template>
  <label for="gte">From:</label>
  <input type="datetime-local" id="gte" v-model="gte" />

  <label for="lte">To:</label>
  <input type="datetime-local" id="lte" v-model="lte" />

  <LineChart v-if="chartData" :chartData="chartData" :options="chartOptions" />
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch } from 'vue'
import axios from 'axios'
import { LineChart } from 'vue-chart-3'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

export default defineComponent({
  name: 'ChartComponent',
  components: { LineChart },
  setup() {
    const gte = ref('')
    const lte = ref('')
    const chartData = ref<any>(null)
    const chartOptions = {
      scales: {
        x: {
          title: {
            display: true,
            text: 'Timeframe'
          }
        },
        y: {
          title: {
            display: true,
            text: 'Sensor Value'
          }
        }
      }
    }

    function appendTimezone(dateStr: string, offset: string = '+07:00'): string {
      if (!dateStr) {
        return ''
      }
      return `${dateStr}${offset}`
    }

    function formatDate(date: Date) {
      if (typeof date === 'string') {
        date = new Date(date)
      }

      const day = date.getDate().toString().padStart(2, '0')
      const monthNames = [
        'Jan',
        'Feb',
        'Mar',
        'Apr',
        'May',
        'Jun',
        'Jul',
        'Aug',
        'Sep',
        'Oct',
        'Nov',
        'Dec'
      ]
      const month = monthNames[date.getMonth()]
      const year = date.getFullYear()
      const hours = date.getHours().toString().padStart(2, '0')
      const minutes = date.getMinutes().toString().padStart(2, '0')

      return `${day} ${month} ${year}, ${hours}:${minutes}`
    }

    async function fetchData() {
      const endpoint = '/sensor-record.json'
      const payload = {
        gte: appendTimezone(gte.value),
        lte: appendTimezone(lte.value)
      }

      try {
        console.log(
          'Calling API using POST method with this value',
          'gte',
          appendTimezone(gte.value),
          'lte',
          appendTimezone(lte.value)
        )
        const response = await axios.post(endpoint, payload)
        const data = response.data

        const labels = data.map((item: any) => formatDate(new Date(item.date)))
        const datasetData = data.map((item: any) => item.sensorRecordData[0].value)

        chartData.value = {
          labels,
          datasets: [
            {
              label: 'Sensor Value',
              data: datasetData,
              borderColor: '#0079AF',
              backgroundColor: 'rgba(0, 121, 175, 0.1)',
              fill: true
            }
          ]
        }
      } catch (error) {
        console.error('Failed to fetch data:', error)
      }
    }

    watch([gte, lte], async (newValues, oldValues) => {
      if (newValues[0] !== oldValues[0] || newValues[1] !== oldValues[1]) {
        fetchData()
      }
    })

    onMounted(async () => {
      fetchData()
    })

    return { chartData, chartOptions, gte, lte }
  }
})
</script>

<template>
  <card>
    <!-- fix for purgeCss -->
    <div class="hidden bg-gray-400 bg-green-400 bg-blue-400 bg-yellow-400">&nbsp;</div>

    <div class="text-sm">
      <div class="flex items-stretch border-b-2 border-black font-bold py-2">
        <div class="flex items-center flex-grow flex-shrink-0" style="flex-basis: 8rem;">
          Event: <input
            type="text"
            class="flex-auto ml-1"
            v-model="event"
            placeholder="Filter"
          >
        </div>
        <div class="flex items-center flex-grow flex-shrink-0" style="flex-basis: 15rem;">
          Socket: <input
            type="text"
            class="flex-auto ml-1"
            v-model="socket"
            placeholder="Filter"
          >
        </div>
        <div class="flex items-center flex-grow flex-shrink-0" style="flex-basis: 20rem;">
          Document Name: <input
            type="text"
            class="flex-auto ml-1"
            v-model="document"
            placeholder="Filter"
          >
        </div>
        <div class="flex items-center flex-grow flex-shrink-0" style="flex-basis: 6rem;">
          Time: <input
            type="text"
            class="flex-auto ml-1"
            v-model="time"
            placeholder="Filter"
          >
        </div>
      </div>

      <RecycleScroller
        :items="filteredLog"
        :item-size="46"
        v-slot="{ item }"
      >
        <div class="flex border-t border-black py-3 hover:bg-yellow-300 items-stretch">
          <div class="flex-grow flex-shrink-0" style="flex-basis: 8rem;">
            <span
              class="px-2 py-1 rounded text-sm"
              :class="{
                'bg-gray-400': item.color === 'gray',
                'bg-green-400': item.color === 'green',
                'bg-blue-400': item.color === 'blue',
                'bg-yellow-400': item.color === 'yellow',
              }"
              v-if="item.label"
            >
              {{ item.label }}
            </span>
          </div>
          <div class="flex-grow flex-shrink-0" style="flex-basis: 15rem;">{{ item.socket }}</div>
          <div class="flex-grow flex-shrink-0" style="flex-basis: 20rem;">{{ item.details }}</div>
          <div class="flex-grow flex-shrink-0" style="flex-basis: 6rem;">{{ item.time }}</div>
        </div>
      </RecycleScroller>
    </div>
  </card>
</template>

<script>
import collect from 'collect.js'
import moment from 'moment'
import Card from './Card'

const formatTime = timestamp => moment(timestamp)
  .format('HH:mm:ss')

export default {
  components: {
    Card,
  },

  props: {
    connections: {
      type: Array,
      required: true,
    },

    documents: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      socket: '',
      document: '',
      time: '',
      event: '',
    }
  },

  computed: {
    log() {
      return collect()
        .merge(this.connections)
        .merge(this.documents)
        .sortByDesc('timestamp')
        .values()
        .map((item, index) => this.mapper(item, index))
        .toArray()
    },
    filteredLog() {
      return collect(this.log)
        .filter(
          item => (!this.socket || item.socket.indexOf(this.socket) !== -1)
            && (!this.document || item.details.indexOf(this.document) !== -1)
            && (!this.event || item.label.indexOf(this.event) !== -1)
            && (!this.time || item.time.indexOf(this.time) !== -1),
        )
        .toArray()
    },
  },

  methods: {
    mapper(item, index) {
      const handlers = {
        connectionLog(data) {
          return {
            color: data.action === 'connected' ? 'green' : 'gray',
            details: data.documentName,
            label: data.action,
            socket: data.socketId,
          }
        },

        documentLog(data) {
          return {
            color: data.action === 'created' ? 'blue' : 'yellow',
            details: data.documentName,
            label: data.action,
            socket: data.socketId,
          }
        },
      }

      const data = handlers[item.key] ? handlers[item.key](item.value) : {
        details: '',
        label: false,
        socket: '',
      }

      return {
        ...data,
        id: index,
        time: formatTime(item.timestamp),
      }
    },
  },
}
</script>

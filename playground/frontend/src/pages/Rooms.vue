<template>
  <div>
    <h1 class="text-3xl mb-8">
      Rooms
    </h1>

    <table class="border border-gray-300 w-full">
      <tr :key="room.name" v-for="room in rooms">
        <td class="p-3 border border-gray-300">{{ room.name }}</td>
        <td class="p-3 border border-gray-300 text-center">{{ room.status }}</td>
        <td class="p-3 border border-gray-300 text-center">{{ room.numberOfUsers }}</td>
        <td class="p-3 border border-gray-300">{{ room.states }}</td>
        <td class="p-3 border border-gray-300 text-center">
          <button v-if="room.status !== 'connected'" @click="room.connect()" class="border-2 border-black bg-black text-white px-4 py-2 rounded">connect</button>
          <button v-if="room.status !== 'disconnected'" @click="room.disconnect()" class="border-2 border-black px-4 py-2 rounded">disconnect</button>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
import * as Y from 'yjs'
// import { WebsocketProvider } from 'y-websocket'
import { HocuspocusProvider } from '@hocuspocus/provider'

class Room {
  doc = new Y.Doc()

  name = ''

  status = 'disconnected'

  states = []

  constructor(name) {
    this.name = name
    // this.provider = new WebsocketProvider('ws://localhost:1234', this.name, this.doc)
    this.provider = new HocuspocusProvider({
      url: 'ws://localhost:1234',
      document: this.doc,
      name: this.name,
      onStatus: ({ status }) => {
        this.status = status
      },
      onAwarenessUpdate: ({ states }) => {
        this.states = states
      },
    })

    this.provider.setAwarenessField('user', { name: `Jon @ ${this.name}` })
  }

  get numberOfUsers() {
    return this.provider.awareness.getStates().size
  }

  connect() {
    this.provider.connect()
  }

  disconnect() {
    this.provider.disconnect()
  }

  destroy() {
    this.provider.destroy()
  }
}

export default {
  data() {
    return {
      rooms: [],
    }
  },

  mounted() {
    for (let i = 1; i < 5; i += 1) {
      this.rooms.push(new Room(`room-${i}`))
    }
  },

  beforeDestroy() {
    this.rooms.forEach(room => {
      room.destroy()
    })
  },
}
</script>

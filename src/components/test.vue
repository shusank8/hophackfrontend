<!-- <template>
  <div class="wrap">
    <button @click="toggleRecord">
      {{ isRecording ? 'Stop & Upload' : 'Start Recording' }}
    </button>

    <p v-if="status">{{ status }}</p>

    <audio v-if="audioUrl" :src="audioUrl" controls></audio>
  </div>
</template>

<script setup>
import { ref, onBeforeUnmount } from 'vue'

/**
 * --- State ---
 */
const isRecording = ref(false)
const status = ref('')
const mediaRecorder = ref(null)
const chunks = ref([])
const stream = ref(null)
const audioUrl = ref('')

/**
 * --- Config ---
 * Replace with your backend WebSocket endpoint
 */
const WS_URL = 'ws://127.0.0.1:54321/functions/v1/hophacksbackend'

/**
 * --- Function similar to your `sendclicked()` ---
 * Opens a WebSocket, and on "open" sends the full audio blob once.
 */
async function sendAudio(blob) {
  return new Promise(async (resolve, reject) => {
    try {
      const socket = new WebSocket(WS_URL)
      socket.binaryType = 'arraybuffer'

      socket.addEventListener('open', async () => {
        console.log('socket opened')

        // Optional: send a small JSON meta frame first
        // const mimeType = blob.type || 'audio/webm'
        // const filename = 'recording.webm'
        // socket.send(JSON.stringify({ type: 'audio_meta', mimeType, filename }))

        // Send entire audio as a single binary message
        const arrayBuffer = await blob.arrayBuffer()
        socket.send(arrayBuffer)

        // Optional: signal end of this upload
        // socket.send(JSON.stringify({ type: 'audio_end' }))
      })

      socket.addEventListener('message', (event) => {
        console.log('Server says:', event.data)
        // You can inspect event.data and resolve early if you get an ACK
        // Example: if server replies with {"type":"ack","status":"ok"}
        try {
          const msg = JSON.parse(event.data)
          if (msg?.type === 'ack' && msg?.status === 'ok') {
            socket.close()
            resolve(msg)
          }
        } catch {
          // not JSON; ignore or handle as needed
        }
      })

      socket.addEventListener('close', () => {
        console.log('socket closed')
        // If we haven’t resolved yet, resolve gracefully
        resolve()
      })

      socket.addEventListener('error', (err) => {
        console.error('socket error', err)
        reject(new Error('WebSocket error'))
      })
    } catch (e) {
      reject(e)
    }
  })
}

/**
 * --- Record toggle ---
 */
const toggleRecord = async () => {
  if (!isRecording.value) {
    // Start recording
    status.value = 'Requesting microphone...'
    try {
      stream.value = await navigator.mediaDevices.getUserMedia({ audio: true })
      mediaRecorder.value = new MediaRecorder(stream.value)

      mediaRecorder.value.ondataavailable = (e) => {
        if (e.data && e.data.size > 0) {
          chunks.value.push(e.data)
        }
      }

      mediaRecorder.value.onstop = async () => {
        try {
          status.value = 'Recording stopped. Preparing upload...'

          const mimeType = mediaRecorder.value.mimeType || 'audio/webm'
          const blob = new Blob(chunks.value, { type: mimeType })
          chunks.value = []

          // Local playback
          audioUrl.value = URL.createObjectURL(blob)

          status.value = 'Uploading via WebSocket...'
          await sendAudio(blob)

          status.value = 'Upload sent!'
        } catch (err) {
          status.value = `Upload failed: ${err.message}`
        } finally {
          // Clean up mic
          if (stream.value) {
            stream.value.getTracks().forEach(t => t.stop())
            stream.value = null
          }
        }
      }

      // No timeslice -> buffer everything until stop()
      mediaRecorder.value.start()
      isRecording.value = true
      status.value = 'Recording...'
    } catch (err) {
      status.value = `Error: ${err.message}`
    }
  } else {
    // Stop -> triggers onstop above
    isRecording.value = false
    mediaRecorder.value?.stop()
  }
}

/**
 * --- Cleanup on unmount ---
 */
onBeforeUnmount(() => {
  if (stream.value) stream.value.getTracks().forEach(t => t.stop())
})
</script> -->

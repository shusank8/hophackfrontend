<template>
  <div class="grid grid-cols-3 h-svh bg-red-300">
    <div class="grid col-end-2 h-svh">
      <iframe :src="link" class="h-svh w-1/2"></iframe>
    </div>
    <div class="col-start-3 h-svh">
      <div v-for="msg in messages" :key="msg.id">
        <div v-if="msg.role == 'question'">
          {{ msg.content }}
        </div>
        <div v-else-if="msg.role == 'final_response'">
          {{ msg.content }}
        </div>
        <div v-else-if="msg.role == 'unique_goal'">
          {{ msg.content }}
        </div>
      </div>
      <input placeholder="ASK ME ANYTHING!" v-model="text_input" />
      <div class="wrap">
        <button @click="toggleRecord">
          {{ isRecording ? "Stop & Upload" : "Start Recording" }}
        </button>

        <p v-if="status">{{ status }}</p>

        <audio v-if="audioUrl" :src="audioUrl" controls></audio>
      </div>

      <button @click="sendclicked">SEND</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onBeforeUnmount } from "vue";
// import Test from "./test.vue";
var text_input = ref("");
var link = ref();
const isRecording = ref(false);
const status = ref("");
const mediaRecorder = ref(null);
const chunks = ref([]);
const stream = ref(null);
const audioUrl = ref("");
const emit = defineEmits(["messageFromChild"]);
var generatingresponse = ref(false);
let messages = ref([]);

async function sendclicked() {
  generatingresponse.value = true;
  const socket = new WebSocket(import.meta.env.VITE_CHAT_URL);
  socket.addEventListener("open", () => {
    console.log("socket openedd");
    messages.value.push({ role: "question", content: text_input.value });

    socket.send(JSON.stringify(text_input.value));
  });
  socket.addEventListener("message", (event) => {
    console.log(event.data);
    var data = JSON.parse(event.data);
    if (data.speechtext) {
      messages.value.push({
        role: "question",
        content: data.speechtext,
      });
    } else if (data.unique_goal) {
      messages.value.push({
        role: "unique_goal",
        content: data.unique_goal,
      });
      console.log(messages.value);
      // unique_goal
    } else if (data.liveUrl) {
      link.value = data.liveUrl;
      messages.value.push({
        role: "liveUrl",
        content: data.liveUrl,
      });
      // liveUrl
    } else {
      // "final_response"
      messages.value.push({
        role: "final_response",
        content: data.final_response,
      });
      generatingresponse.value = false;
    }
  });
}

async function sendAudio(blob) {
  generatingresponse.value = true;
  const socket = new WebSocket(
    "ws://127.0.0.1:54321/functions/v1/hophacksbackend"
  );
  socket.addEventListener("open", async () => {
    console.log("socket opened");

    // Send entire audio as a single binary message
    const arrayBuffer = await blob.arrayBuffer();
    console.log("not this");
    socket.send(arrayBuffer);
  });

  socket.addEventListener("message", (event) => {
    var data = JSON.parse(event.data);
    // unique_goal, liveUrl, final_response

    if (data.speechtext) {
      messages.value.push({
        role: "question",
        content: data.speechtext,
      });
    } else if (data.unique_goal) {
      messages.value.push({
        role: "unique_goal",
        content: data.unique_goal,
      });
      console.log(messages.value);
      // unique_goal
    } else if (data.liveUrl) {
      link.value = data.liveUrl;
      messages.value.push({
        role: "liveUrl",
        content: data.liveUrl,
      });
      // liveUrl
    } else {
      // "final_response"
      messages.value.push({
        role: "final_response",
        content: data.final_response,
      });
      generatingresponse.value = false;
    }
  });
}

const toggleRecord = async () => {
  if (!isRecording.value) {
    // Start recording
    status.value = "Requesting microphone...";
    try {
      stream.value = await navigator.mediaDevices.getUserMedia({ audio: true });
      mediaRecorder.value = new MediaRecorder(stream.value);

      mediaRecorder.value.ondataavailable = (e) => {
        if (e.data && e.data.size > 0) {
          chunks.value.push(e.data);
        }
      };

      mediaRecorder.value.onstop = async () => {
        try {
          status.value = "Recording stopped. Preparing upload...";

          const mimeType = mediaRecorder.value.mimeType || "audio/webm";
          const blob = new Blob(chunks.value, { type: mimeType });
          chunks.value = [];

          // Local playback
          audioUrl.value = URL.createObjectURL(blob);

          status.value = "Uploading via WebSocket...";
          await sendAudio(blob);

          status.value = "Upload sent!";
        } catch (err) {
          status.value = `Upload failed: ${err.message}`;
        } finally {
          // Clean up mic
          if (stream.value) {
            stream.value.getTracks().forEach((t) => t.stop());
            stream.value = null;
          }
        }
      };

      // No timeslice -> buffer everything until stop()
      mediaRecorder.value.start();
      isRecording.value = true;
      status.value = "Recording...";
    } catch (err) {
      status.value = `Error: ${err.message}`;
    }
  } else {
    // Stop -> triggers onstop above
    isRecording.value = false;
    mediaRecorder.value?.stop();
  }
};
onBeforeUnmount(() => {
  if (stream.value) stream.value.getTracks().forEach((t) => t.stop());
});
</script>

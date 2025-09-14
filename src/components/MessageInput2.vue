<template>
  <div
    :class="[
      // full-bleed when browserneeded; comfy card when not
      browserneeded
        ? 'm-0 p-0 rounded-none border-0 shadow-none ring-0'
        : 'mx-auto my-4 md:my-8 max-w-[1400px] rounded-3xl border border-white/60 bg-white/70 backdrop-blur-xl shadow-xl ring-1 ring-black/5 p-3 md:p-4',
    ]"
  >
    <h1 class="text-xl pl-8">AI BROWSER</h1>
    <div
      :class="[
        'grid overflow-hidden',
        browserneeded
          ? 'grid-cols-1 lg:grid-cols-3 p-0 m-0'
          : 'place-items-center p-3 md:p-4',
      ]"
    >
      <!-- Left (iframe) -->
      <div
        :class="[
          browserneeded
            ? 'col-span-2 relative animate-in-from-left m-0 p-0'
            : 'hidden',
        ]"
      >
        <div
          :class="[
            browserneeded
              ? 'h-svh w-full m-0 p-3 pb-8'
              : 'h-full w-full p-2 lg:p-3',
          ]"
        >
          <div
            :class="[
              browserneeded
                ? 'h-svh w-full rounded-none border-0 bg-transparent backdrop-blur-0 shadow-none ring-0'
                : 'h-[70svh] md:h-[78svh] lg:h-[84svh] rounded-2xl border border-white/50 bg-white/60 backdrop-blur-md shadow-lg ring-1 ring-black/5 overflow-hidden transition-all duration-500',
            ]"
          >
            <iframe
              :src="link"
              class="h-full w-full rounded-4xl mb-10"
            ></iframe>
          </div>
        </div>
      </div>

      <!-- Right (messages) -->
      <div
        :class="[
          'flex flex-col',
          browserneeded
            ? 'col-span-1 border-l border-white/60 bg-white/50 backdrop-blur-md animate-in-from-right m-0'
            : 'w-full max-w-2xl',
        ]"
      >
        <!-- Messages list -->
        <div
          :class="[
            'flex-1 overflow-y-auto  mt-5 ',
            browserneeded ? 'p-0 space-y-3' : 'p-3 md:p-4 space-y-2.5',
          ]"
          style="max-height: 80vh"
        >
          <div v-for="msg in messages" :key="msg.id">
            <div
              v-if="msg.role == 'question'"
              class="rounded-2xl px-4 py-3 border border-gray-20 bg-gray-30/80 shadow-sm ring-1 ring-black/5"
            >
              {{ msg.content }}
            </div>
            <div
              v-else-if="msg.role == 'final_response'"
              class="rounded-2xl px-4 py-3 border border-white/60 bg-white/80 shadow-sm ring-1 ring-black/5"
            >
              {{ msg.content }}
            </div>
            <div
              v-else-if="msg.role == 'unique_goal'"
              class="rounded-2xl px-4 py-3 border border-emerald-200/90 bg-emerald-50/80 shadow-sm ring-1 ring-black/5"
            >
              {{ msg.content }}
            </div>
          </div>
        </div>
        <div v-if="generatingresponse == true">
          <i class="pi pi-spin pi-spinner" style="font-size: 1rem"></i>
        </div>

        <!-- Input / controls -->
        <div
          :class="[
            'border-t border-white/60 bg-white/70 backdrop-blur-md mb-10 ',
            browserneeded ? ' ml-4 pl-3 ' : 'p-3 md:p-4',
          ]"
        >
          <button>
            <i class="pi-spin"></i>
          </button>
          <div
            :class="[
              'grid items-center',
              // single row layout; tighter when browserneeded
              'grid-cols-12',
              browserneeded ? 'gap-2 px-2 py-2' : 'gap-3',
            ]"
          >
            <!-- Text input -->
            <input
              placeholder="ASK ME ANYTHING!"
              v-model="text_input"
              :class="[
                'rounded-xl border border-gray-200/80 bg-white/80 shadow-sm outline-none ring-1 ring-transparent focus:ring-gray-300 placeholder:text-gray-400 transition',
                // width per mode (keeps all three on one line)
                browserneeded
                  ? 'col-span-8 px-4 py-2 '
                  : 'col-span-9 px-4 py-3',
              ]"
            />

            <!-- SEND -->
            <button
              @click="sendclicked"
              :class="[
                'inline-flex items-center justify-center rounded-xl border border-white/60 ring-1 ring-black/5 bg-white/30 text-gray-900 backdrop-blur-xl shadow-sm hover:bg-white/40 hover:shadow-md active:scale-[0.98] focus:outline-none focus-visible:ring-2 focus-visible:ring-gray-400/60 transition',
                browserneeded
                  ? 'col-span-1 px-3 py-2 '
                  : 'col-span-1 px-4 py-3',
              ]"
            >
              <i class="pi pi-send"></i>
            </button>

            <!-- Record on the same line -->
            <button
              @click="toggleRecord"
              :class="[
                'inline-flex items-center justify-center rounded-xl border border-white/60 ring-1 ring-black/5 bg-white/30 text-gray-900 backdrop-blur-xl shadow-sm hover:bg-white/40 hover:shadow-md active:scale-[0.98] focus:outline-none focus-visible:ring-2 focus-visible:ring-gray-400/60 transition',
                browserneeded
                  ? 'col-span-1 px-3 py-2 '
                  : 'col-span-1 px-4 py-3',
              ]"
            >
              <i :class="isRecording ? 'pi pi-stop' : 'pi pi-microphone'"></i>
            </button>
            <button
              @click="mute"
              :class="[
                'inline-flex items-center justify-center rounded-xl border border-white/60 ring-1 ring-black/5 bg-white/30 text-gray-900 backdrop-blur-xl shadow-sm hover:bg-white/40 hover:shadow-md active:scale-[0.98] focus:outline-none focus-visible:ring-2 focus-visible:ring-gray-400/60 transition',
                browserneeded
                  ? 'col-span-1 px-3 py-2 '
                  : 'col-span-1 px-4 py-3',
              ]"
            >
              <i
                class="pi pi-volume-off"
                :class="soundmute ? 'text-red-500' : ''"
              ></i>
            </button>

            <!-- Audio preview (full width, below row when present) -->
            <div v-if="audioUrl" class="col-span-12 pt-2">
              <audio :src="audioUrl" controls class="w-1/2"></audio>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<!-- style (scoped) -->
<style scoped>
@keyframes slideInLeft {
  0% {
    transform: translateX(-24px);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}
@keyframes slideInRight {
  0% {
    transform: translateX(24px);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}
.animate-in-from-left {
  animation: slideInLeft 0.55s ease-out both;
}
.animate-in-from-right {
  animation: slideInRight 0.55s ease-out both;
}
</style>

<script setup>
import { ref, onBeforeUnmount } from "vue";
import "primeicons/primeicons.css";

// import Test from "./test.vue";
import { IoIosSend } from "react-icons/io";
var text_input = ref("");
var link = ref();
const isRecording = ref(false);
const status = ref("");
const mediaRecorder = ref(null);
const chunks = ref([]);
const stream = ref(null);
const audioUrl = ref("");
const soundmute = ref(true);
const emit = defineEmits(["messageFromChild"]);
var generatingresponse = ref(false);
let messages = ref([]);
var browserneeded = ref(false);
function speak(text) {
  if (soundmute.value == false) {
    const utterance = new SpeechSynthesisUtterance(text);
    speechSynthesis.speak(utterance);
  }
}

async function sendclicked() {
  generatingresponse.value = true;
  const socket = new WebSocket(import.meta.env.VITE_CHAT_URL);
  socket.addEventListener("open", () => {
    console.log("socket openedd");
    messages.value.push({ role: "question", content: text_input.value });

    socket.send(JSON.stringify(messages.value));
  });
  socket.addEventListener("message", (event) => {
    console.log(event.data);
    var data = JSON.parse(event.data);
    if (data.unique_goal) {
      messages.value.push({
        role: "unique_goal",
        content: data.unique_goal,
      });
      speak(data.unique_goal);

      // unique_goal
    } else if (data.liveUrl) {
      link.value = data.liveUrl;
      browserneeded.value = true;
      messages.value.push({
        role: "liveUrl",
        content: data.liveUrl,
      });
      speak(data.liveUrl);

      // liveUrl
    } else {
      // "final_response"
      messages.value.push({
        role: "final_response",
        content: data.final_response,
      });
      speak(data.final_response);
      generatingresponse.value = false;
    }
  });
}

async function sendAudio(blob) {
  generatingresponse.value = true;
  const socket = new WebSocket(import.meta.env.VITE_CHAT_URL);
  socket.addEventListener("open", async () => {
    console.log("socket opened");

    // Send entire audio as a single binary message
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
      var all = JSON.stringify(messages.value) + data.speechtext;
      socket.send(all);
    } else if (data.unique_goal) {
      messages.value.push({
        role: "unique_goal",
        content: data.unique_goal,
      });
      speak(data.unique_goal);

      // unique_goal
    } else if (data.liveUrl) {
      link.value = data.liveUrl;
      browserneeded.value = true;
      messages.value.push({
        role: "liveUrl",
        content: data.liveUrl,
      });
      speak(data.liveUrl);
      // liveUrl
    } else {
      // "final_response"
      messages.value.push({
        role: "final_response",
        content: data.final_response,
      });
      generatingresponse.value = false;
      speak(data.final_response);
    }
  });
}

function mute() {
  soundmute.value = !soundmute.value;
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

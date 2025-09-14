<!-- <template>
  <input placeholder="ASK ME ANYTHING!" v-model="text_input" />
  <button
    @click="voicemode_activate"
    :class="voice_mode ? 'bg-green-600 text-white' : 'bg-gray-300 text-black'"
  >
    VOICE
  </button>

  <button @click="sendclicked">SEND</button>
  <iframe :src="link"></iframe>
  <Test />
</template>

<script setup>
import { ref } from "vue";
import Test from "./test.vue";
var text_input = ref("");
var voice_mode = ref(false);
var link = ref("");

// import { createClient } from "@supabase/supabase-js";


function voicemode_activate() {
  voice_mode.value = !voice_mode.value;
  console.log(voice_mode.value);
}

async function sendclicked() {
  const socket = new WebSocket(
    "ws://127.0.0.1:54321/functions/v1/hophacksbackend"
  );
  // console.log(text_input.value);
  socket.addEventListener("open", () => {
    console.log("socket openedd");
    socket.send(JSON.stringify(text_input.value));
  });

  socket.addEventListener("message", (event) => {
    console.log(event.data);
  });
}



</script> -->

<template>
  <div id="app">
    <SelectUsername
      v-if="!usernameAlreadySelected"
      @onInput="onUsernameSelection"
    />
    <Chat v-else />
  </div>
</template>

<script>
import { ref } from "vue";
import socket from "./socket";
import SelectUsername from "./components/SelectUsername.vue";
import Chat from "./components/Chat.vue";

export default {
  name: "App",
  components: {
    SelectUsername,
    Chat,
  },
  setup() {
    const usernameAlreadySelected = ref(false);

    socket.on("connect_error", (err) => {
      if (err.message === "invalid username") {
        usernameAlreadySelected.value = false;
      }
    });

    const onUsernameSelection = (username) => {
      usernameAlreadySelected.value = true;
      socket.auth = { username };
      socket.connect();
    };

    return {
      usernameAlreadySelected,

      onUsernameSelection,
    };
  },
  unmounted() {
    socket.off("connect_error");
  },
};
</script>

<style>
body {
  margin: 0;
}
@font-face {
  font-family: Lato;
  src: url("/fonts/Lato-Regular.ttf");
}
#app {
  font-family: Lato, Arial, sans-serif;
  font-size: 14px;
}
</style>

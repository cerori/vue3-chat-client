<template>
  <div>
    <div class="header">
      <StatusIcon :connected="user.connected" />{{ user.username }}
    </div>

    <ul>
      <li
        v-for="(message, index) in user.messages"
        :key="index"
        class="message"
      >
        <div v-if="displaySender(message, index)" class="sender">
          {{ message.fromSelf ? "(yourself)" : user.username }}
        </div>
        {{ message.content }}
      </li>
    </ul>

    <form @submit.prevent="onSubmit" class="form">
      <textarea v-model="input" placeholder="Your message..." class="input" />
      <button :disabled="!isValid" class="send-button">Send</button>
    </form>
  </div>
</template>

<script>
import { ref, toRefs, computed } from "vue";
import StatusIcon from "./StatusIcon.vue";

export default {
  components: { StatusIcon },
  props: {
    user: Object,
  },
  setup(props, { emit }) {
    const input = ref("");
    const { user } = toRefs(props);

    const onSubmit = () => {
      emit("onInput", input.value);
      input.value = "";
    };

    const displaySender = (message, index) =>
      index === 0 ||
      user.value.messages[index - 1].fromSelf !==
        user.value.messages[index].fromSelf;

    const isValid = computed(() => input.value.length > 0);

    return {
      input,
      isValid,

      onSubmit,
      displaySender,
    };
  },
};
</script>

<style scoped>
.header {
  line-height: 40px;
  padding: 10px 20px;
  border-bottom: 1px solid #dddddd;
}
.messages {
  margin: 0;
  padding: 20px;
}
.message {
  list-style: none;
}
.sender {
  font-weight: bold;
  margin-top: 5px;
}
.form {
  padding: 10px;
}
.input {
  width: 80%;
  resize: none;
  padding: 10px;
  line-height: 1.5;
  border-radius: 5px;
  border: 1px solid #000;
}
.send-button {
  vertical-align: top;
}
</style>

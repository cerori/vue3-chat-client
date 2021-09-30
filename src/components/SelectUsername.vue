<template>
  <div class="select-username">
    <form @submit.prevent="onSubmit">
      <input v-model="username" placeholder="Your username..." />
      <button :disabled="!isValid">Send</button>
    </form>
  </div>
</template>

<script>
import { ref, computed } from "vue";
export default {
  setup(props, { emit }) {
    const username = ref("");
    const isValid = computed(() => username.value.length > 2);

    const onSubmit = () => {
      emit("onInput", username.value);
    };

    return {
      username,
      isValid,

      onSubmit,
    };
  },
};
</script>

<style scoped>
.select-username {
  width: 300px;
  margin: 200px auto 0;
}
</style>

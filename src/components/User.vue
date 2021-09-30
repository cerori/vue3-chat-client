<template>
  <div class="user" @click="onClick" :class="{ selected: selected }">
    <div class="description">
      <div class="name">
        {{ user.username }} {{ user.self ? " (yourself)" : "" }}
      </div>
      <div class="status">
        <StatusIcon :connected="user.connected" />{{ status }}
      </div>
    </div>
    <div v-if="user.hasNewMessage" class="new-message">!</div>
  </div>
</template>

<script>
import { toRefs, computed } from "vue";
import StatusIcon from "./StatusIcon.vue";

export default {
  props: {
    user: Object,
    selected: Boolean,
  },
  components: { StatusIcon },
  setup(props, { emit }) {
    //console.log(props);
    const { user } = toRefs(props);
    const status = computed(() =>
      user.value.connected ? "online" : "offline"
    );

    const onClick = () => {
      emit("select");
    };

    return {
      status,

      onClick,
    };
  },
};
</script>

<style scoped>
.selected {
  background-color: #1164a3;
}
.user {
  padding: 10px;
}
.description {
  display: inline-block;
}
.status {
  color: #92959e;
}
.new-messages {
  color: white;
  background-color: red;
  width: 20px;
  border-radius: 5px;
  text-align: center;
  float: right;
  margin-top: 10px;
}
</style>

<template>
  <div>
    <div class="left-panel">
      <User
        v-for="user in users"
        :key="user.userID"
        :user="user"
        :selected="selectedUser === user"
        @select="onSelectUser(user)"
      />
    </div>
    <MessagePanel
      v-if="selectedUser"
      :user="selectedUser"
      @onInput="onMessage"
      class="right-panel"
    />
  </div>
</template>

<script>
import { ref } from "vue";
import socket from "../socket";
import User from "./User.vue";
import MessagePanel from "./MessagePanel.vue";

export default {
  components: { User, MessagePanel },

  setup() {
    const _users = ref([]);
    const _selectedUser = ref(null);

    socket.on("connect", () => {
      _users.value.forEach((user) => {
        if (user.self) {
          user.connected = true;
        }
      });
    });

    socket.on("disconnect", () => {
      _users.value.forEach((user) => {
        if (user.self) {
          user.connected = false;
        }
      });
    });

    socket.on("users", (users) => {
      users.forEach((user) => {
        user.self = user.userID === socket.id;
        initReactiveProperties(user);
      });

      _users.value = users.sort((a, b) => {
        if (a.self) return -1;
        if (b.self) return 1;
        if (a.username < b.username) return -1;
        return a.username > b.username ? 1 : 0;
      });
    });

    socket.on("user connected", (user) => {
      initReactiveProperties(user);
      _users.value.push(user);
    });

    socket.on("user disconnected", (id) => {
      for (let i = 0; _users.value.length; i++) {
        const user = _users.value[i];
        if (user.userID === id) {
          user.connected = false;
          break;
        }
      }
    });

    socket.on("private message", ({ content, from }) => {
      for (let i = 0; i < _users.value.length; i++) {
        const user = _users.value[i];
        if (user.userID == from) {
          user.messages.push({
            content,
            fromSelf: false,
          });
          if (user != _selectedUser.value) {
            user.hasNewMessage = true;
          }
          break;
        }
      }
    });

    const initReactiveProperties = (user) => {
      user.connected = true;
      user.messages = [];
      user.hasNewMessage = false;
    };

    const onMessage = (content) => {
      if (_selectedUser.value) {
        socket.emit("private message", {
          content,
          to: _selectedUser.value.userID,
        });
        _selectedUser.value.messages.push({
          content,
          fromSelf: true,
        });
      }
    };
    const onSelectUser = (user) => {
      _selectedUser.value = user;
      user.hasNewMessage = false;
    };

    return {
      users: _users,
      selectedUser: _selectedUser,

      onMessage,
      onSelectUser,
    };
  },
  unmounted() {
    socket.Useroff("connect");
    socket.off("disconnect");
    socket.off("users");
    socket.off("user connected");
    socket.off("user disconnected");
    socket.off("private message");
  },
};
</script>

<style>
.left-panel {
  position: fixed;
  left: 0;
  top: 0;
  bottom: 0;
  width: 260px;
  overflow-x: hidden;
  background-color: #3f0e40;
  color: white;
}
.right-panel {
  margin-left: 260px;
}
</style>

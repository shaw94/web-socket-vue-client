<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </header>
</template>

<script>
import Echo from "laravel-echo";
import Pusher from "pusher-js";
import axios from "axios";

const { data: token } = await axios.post("http://127.0.0.1:8000/api/sanctum/token", {
  email: "web@websockets.app",
  password: "password",
})

let echo = new Echo({
  broadcaster: "pusher",
  key: "chat-app-key",
  wsHost: "127.0.0.1",
  wsPort: 6001,
  forceTLS: false,
  cluster: "mt1",
  disableStats: true,
  authorizer: (channel, options) => {
    // console.log(options);
    return {
      authorize: (socketId, callback) => {
        axios({
          method: "POST",
          url: "http://127.0.0.1:8000/api/broadcasting/auth",
          headers: {
            Authorization: `Bearer ${token}`,
          },
          data: {
            socket_id: socketId,
            channel_name: channel.name,
          },
        })
          .then((response) => {
            callback(false, response.data);
          })
          .catch((error) => {
            callback(true, error);
          });
      },
    };
  },
});

echo.private('private.chat.1')
  .listen(".chat-message", (message) => console.log(message));

echo.join(`presence.chat.1`)
  .here((users) => console.log({ users: users }))
  .listen(".message", (message) => console.log(message));

// const response = await axios.post('http://127.0.0.1:8000/api/event', {
//   message: 'What it is?'
// });

// console.log(response);
</script>
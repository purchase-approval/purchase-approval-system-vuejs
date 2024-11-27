<script lang="ts">
import Stomp from "webstomp-client";

export default class WebSocket {
  /*static async getStompClient() {
    if (WebSocket.stompClient && WebSocket.stompClient.connected) {
      /!* eslint-disable *!/
      console.log('Ws Connection is successfully.');
      return WebSocket.stompClient;
    } else {
      return await this.getStompClient();
    }
  }*/

  static connect(uuid) {
    WebSocket.isConnection = false;
    WebSocket.socket = new SockJS(import.meta.env.VITE_VUE_APP_WS_REGISTER_ENDPOINT);
    WebSocket.stompClient = Stomp.over(WebSocket.socket);
    WebSocket.stompClient.connect({}, frame => {
        WebSocket.stompClient.subscribe("/topic/status/" + uuid,
          function (messageOutput) {
            console.log(JSON.parse(messageOutput.body));
          });
      }
    );
  }

  static async subscribe(uuid) {
    this.connect(uuid);
  }
}
</script>

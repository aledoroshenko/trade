<template>
  <div id="app">
    <div>
      <h2>Socket status: {{ socketStatus }}</h2>
    </div>
    <div>
      <h2>Available ISINs</h2>
      <div v-for="isin in availableIsins" :key="isin">
        <p>
          {{ isin }}
          <button :disabled="isButtonDisabled" @click="subscribe(isin)">
            {{
              subscribedIsins &&
              subscribedIsins[isin] &&
              subscribedIsins[isin].subscriptionAlive
                ? "Subscribed"
                : "Subscribe"
            }}
          </button>
          <button :disabled="isButtonDisabled" @click="unsubscribe(isin)">
            Unsubscribe
          </button>
          <button
            :disabled="isButtonDisabled"
            @click="removeSubscription(isin)"
          >
            Remove
          </button>
        </p>
      </div>
    </div>

    <div>
      <h2>Subscribed ISINs</h2>
      <div v-if="subscribedIsins === null">
        No subscriptions
      </div>
      <div v-else>
        <div v-for="isin in subscribedIsins" :key="isin.isin">
          <div v-if="isin.price === null">
            {{ isin.isin }}: loading prices...
          </div>
          <div v-else :class="{ active: isin.subscriptionAlive }">
            {{ isin.isin }}: {{ isin.price }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data: function() {
    return {
      websocket: null,
      availableIsins: ["DE000BASF111", "INE323A01026", "DE000A2H89E6"],
      subscribedIsins: null,
      socketStatus: "DISCONNECTED"
    };
  },
  computed: {
    isButtonDisabled() {
      return this.socketStatus !== "CONNECTED";
    }
  },
  mounted: function() {
    this.connect();
  },
  methods: {
    subscribe(isin) {
      if (this.websocket.readyState !== 1) {
        return console.log("Cant subscribe");
      }

      this.websocket.send(
        JSON.stringify({
          event: "events",
          data: { type: "subscribe", subscribe: isin }
        })
      );

      this.subscribedIsins = {
        ...this.subscribedIsins,
        [isin]: {
          isin,
          price: null,
          bid: null,
          ask: null,
          subscriptionAlive: true
        }
      };
    },
    unsubscribe(isin) {
      this.websocket.send(
        JSON.stringify({
          event: "events",
          data: { type: "unsubscribe", unsubscribe: isin }
        })
      );

      this.subscribedIsins[isin] = {
        ...this.subscribedIsins[isin],
        subscriptionAlive: false
      };
    },
    connect() {
      console.log("Starting websocket");
      this.websocket = new WebSocket("ws://localhost:8080");

      this.websocket.onopen = this.open;
      this.websocket.onmessage = this.message;
      this.websocket.onerror = this.error;
      this.websocket.onclose = this.close;
    },
    open(event) {
      console.log(event);
      console.log("Successfully connected to the server");
      this.socketStatus = "CONNECTED";
    },
    close(event) {
      console.log(event);
      this.socketStatus = "CLOSED";
    },
    error(event) {
      console.log(event);
      this.socketStatus = "ERROR";
    },
    message({ data }) {
      const priceData = JSON.parse(data);
      this.subscribedIsins[priceData.isin] = {
        ...this.subscribedIsins[priceData.isin],
        ...priceData
      };
    },
    removeSubscription(isin) {
      const temp = { ...this.subscribedIsins };
      delete temp[isin];
      this.subscribedIsins = { ...temp };
    }
  }
};
</script>

<style>
.active {
  color: green;
}
</style>

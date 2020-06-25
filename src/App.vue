<template>
  <v-app>
    <v-main class="grey lighten-5">
      <v-container class="white">
        <v-row>
          <v-col>
            <v-row>
              <v-col v-if="socketStatus === 'CLOSED'">
                <v-alert type="error">
                  Connection unavailable
                </v-alert>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <h1>ISINs</h1>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <h3>Available</h3>

                <subscription-toolbar
                  :isins="availableIsins"
                  :activeSubscriptions="activeSubscriptions"
                  @subscribe="onSubscribe"
                  @unsubscribe="onUnsubscribe"
                  @remove="onRemove"
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" sm="8" md="4">
                <h3>Subscribed</h3>
                <p>Active subscriptions highlighted green</p>

                <div v-if="subscribedIsins === null">
                  No subscriptions
                </div>

                <div v-else>
                  <v-simple-table>
                    <template v-slot:default>
                      <thead>
                        <tr>
                          <th class="text-left">ISIN</th>
                          <th class="text-left">Price</th>
                          <th class="text-left">Bid price</th>
                          <th class="text-left">Ask price</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr
                          v-for="isin in subscribedIsins"
                          :key="isin.name"
                          :class="{
                            subscriptionAlive: isin.subscriptionAlive
                          }"
                        >
                          <template v-if="isin.price === null">
                            <td>{{ isin.isin }}</td>
                            <td colspan="3">connecting to market...</td>
                          </template>

                          <template v-else>
                            <td>{{ isin.isin }}</td>
                            <td>{{ isin.price }}</td>
                            <td>{{ isin.price }}</td>
                            <td>{{ isin.price }}</td>
                          </template>
                        </tr>
                      </tbody>
                    </template>
                  </v-simple-table>
                </div>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import SubscriptionToolbar from "./components/SubscriptionToolbar";

export default {
  name: "App",
  components: { SubscriptionToolbar },
  data: function() {
    return {
      websocket: null,
      availableIsins: ["DE000BASF111", "INE323A01026", "DE000A2H89E6"],
      subscribedIsins: null,
      socketStatus: "DISCONNECTED",
      timerId: null,
      needResubscribe: false
    };
  },
  computed: {
    activeSubscriptions: function() {
      if (!this.subscribedIsins) {
        return [];
      }

      return Object.values(this.subscribedIsins).reduce(
        (memo, isin) => (isin.subscriptionAlive ? [...memo, isin.isin] : memo),
        []
      );
    }
  },
  mounted: function() {
    this.connect();
  },
  methods: {
    onSubscribe(isin) {
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
          ask: null
        }
      };
    },
    onUnsubscribe(isin) {
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
      if (this.timerId !== null) {
        clearInterval(this.timerId);
      }

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

      if (this.needResubscribe) {
        console.log("Needs resubscribing");
        this.activeSubscriptions.forEach(isin => {
          this.onSubscribe(isin);
        });
      }
    },
    close(event) {
      console.log(event);
      this.socketStatus = "CLOSED";

      this.needResubscribe = true;

      this.timerId = setInterval(() => {
        console.log("Reconnecting");
        this.connect();
      }, 2000);
    },
    error(event) {
      console.log(event);
      this.socketStatus = "ERROR";
    },
    message({ data }) {
      const priceData = JSON.parse(data);
      this.subscribedIsins[priceData.isin] = {
        ...this.subscribedIsins[priceData.isin],
        ...priceData,
        subscriptionAlive: true
      };
    },
    onRemove(isin) {
      const temp = { ...this.subscribedIsins };
      delete temp[isin];
      this.subscribedIsins = { ...temp };
    }
  }
};
</script>

<style>
.subscriptionAlive td {
  color: green;
}
</style>

<template>
  <v-row>
    <v-col col-12>
      <div class="isin-wrapper" v-for="isin in isins" :key="isin">
        <div class="isin-label">{{ isin }}</div>
        <div class="buttons-wrapper">
          <div class="mx-2">
            <v-btn
              v-if="isActive(isin)"
              x-small
              color="success"
              @click="$emit('unsubscribe', isin)"
              >Unsubscribe</v-btn
            >
            <v-btn v-else x-small @click="$emit('subscribe', isin)"
              >Subscribe</v-btn
            >
          </div>
          <div class="mx-2">
            <v-btn x-small @click="$emit('remove', isin)">Remove</v-btn>
          </div>
        </div>
      </div>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: "SubscriptionToolbar",
  props: ["isins", "activeSubscriptions"],
  computed: {},
  methods: {
    isActive(isin) {
      debugger;
      return this.activeSubscriptions.includes(isin);
    }
  }
};
</script>

<style>
.isin-wrapper {
  display: flex;
  margin-bottom: 20px;
}

.isin-label {
  width: 120px;
}

.buttons-wrapper {
  display: flex;
}
</style>

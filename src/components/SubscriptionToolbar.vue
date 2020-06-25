<template>
  <div>
    <div v-for="isin in isins" :key="isin">
      <p>
        {{ isin }}
        <button :disabled="isButtonDisabled" @click="$emit('subscribe', isin)">
          Subscribe
        </button>
        <button
          :disabled="isButtonDisabled"
          @click="$emit('unsubscribe', isin)"
        >
          Unsubscribe
        </button>
        <button :disabled="isButtonDisabled" @click="$emit('remove', isin)">
          Remove
        </button>
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: "SubscriptionToolbar",
  props: ["isins", "activeSubscriptions"],
  computed: {
    isButtonDisabled() {
      return false;
    }
  }
};
</script>

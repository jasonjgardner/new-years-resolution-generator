<template>
  <button :disabled="!!disabled" @click="clicked" type="button">
    {{ text || title }}
  </button>
</template>

<script>
export default {
  name: "ButtonReel",
  props: {
    choices: {
      type: Array,
      required: true,
      validator: (val) =>
        Array.isArray(val) &&
        val.length === val.filter((v) => typeof v === "string").length,
      default: () => [],
    },
    speed: {
      type: Number,
      validator: (val) => val > 1,
      default: 200,
    },
    timer: {
      type: Number,
      validator: (val) => val >= 1000,
      default: 30000,
    },
    title: {
      type: String,
      default: "Loading...",
    },
  },
  data: () => ({
    text: "",
    interval: undefined,
    timeout: undefined,
    disabled: false,
  }),
  methods: {
    clicked() {
      this.$emit("clicked", this.text);
      this.stop();
      this.disabled = true;
    },
    random() {
      return this.choices[Math.floor(Math.random() * this.choices.length)];
    },
    stop() {
      clearInterval(this.interval);
      clearTimeout(this.timeout);
    },
  },
  mounted() {
    this.text = this.title || this.random();

    if (this.disabled) {
      return;
    }

    this.interval = setInterval(() => {
      this.text = this.random();
    }, this.speed || 100);

    this.timeout = setTimeout(this.clicked, this.timer || 10000);
  },
  unmounted() {
    this.stop();
  },
};
</script>

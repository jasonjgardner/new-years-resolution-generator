<template>
  <div class="sidebar__log" ref="log">
    <slot></slot>
    <div class="columns" :style="log.length < 11 ? `--cols:1` : `cols:2`">
      <ol class="list">
        <li v-for="(entry, entryIdx) in log" :key="entryIdx">
          {{ entry }}
        </li>
      </ol>
    </div>

    <div class="sidebar__actions">
      <button class="btn" type="reset" @click.prevent="clearLog">
        Restart
      </button>
      <button class="btn">Share</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "ResolutionLog",
  props: {
    log: {
      type: Array,
      default: () => [],
    },
  },
  methods: {
    clearLog() {
      this.$emit("clear");
    },
  },
};
</script>
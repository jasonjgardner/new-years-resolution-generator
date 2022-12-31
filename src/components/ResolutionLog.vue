<template>
  <div :class="['sidebar__log', { sharing: isSharing }]" ref="log">
    <header class="sidebar__header">
      <slot></slot>
    </header>

    <ol class="list">
      <li v-for="(entry, entryIdx) in log" :key="entryIdx">
        {{ entry }}
      </li>
    </ol>

    <div class="sidebar__actions">
      <button class="btn" type="reset" @click.prevent="clearLog">
        Restart
      </button>
      <button class="btn" type="submit" @click.prevent="shareList">
        Share
      </button>
    </div>
  </div>
</template>

<script>
import html2canvas from "html2canvas";
export default {
  name: "ResolutionLog",
  props: {
    log: {
      type: Array,
      default: () => [],
    },
  },
  data: () => ({
    isSharing: false,
    error: null,
  }),
  methods: {
    clearLog() {
      this.$emit("clear");
    },
    shareList() {
      this.isSharing = true;

      setTimeout(async () => {
        try {
          const canvas = await html2canvas(this.$refs.log);
          const dataUrl = canvas.toDataURL();
          const blob = await (await fetch(dataUrl)).blob();
          const image = new File([blob], "resolutions.png", {
            type: blob.type,
          });

          await navigator.share({
            title: `My ${this.goalYear} Resolutions`,
            text: this.log.join("\n"),
            files: [image],
          });
        } catch (err) {
          this.error = err;
        }
        this.isSharing = false;
      }, 1000);
    },
  },
};
</script>

<style>
.sidebar__log {
  background-color: var(--color-background);
  color: var(--color-on-background);
}

.sidebar__log.sharing {
  padding: var(--vr, 1rem);
}

.sharing .sidebar__actions,
.sharing .sidebar__header .btn {
  display: none;
}
</style>

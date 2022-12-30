<template>
  <main id="generator" :class="{ sharing: isSharing }">
    <header class="header">
      <h1>
        <span class="bg">New Year's Resolution Generator</span>
      </h1>

      <button class="btn toggle" @click="showSidebar = !showSidebar">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="30"
          height="30"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <line x1="3" y1="12" x2="21" y2="12"></line>
          <line x1="3" y1="6" x2="21" y2="6"></line>
          <line x1="3" y1="18" x2="21" y2="18"></line>
        </svg>
        <span class="badge">{{ log.length }}</span>
      </button>
    </header>

    <aside
      class="error"
      v-if="error && error.length > 0"
      v-text="error"
    ></aside>

    <div class="container">
      <article :class="['main', { active: !complete }]" ref="generated">
        <h3>
          My <time>{{ goalYear }}</time> New Year&rsquo;s resolution is:
        </h3>

        <div class="buttons">
          <div v-for="(set, itr) in sets" class="btn-wrapper" :key="itr">
            <transition name="fade">
              <ButtonReel
                v-if="idx > itr"
                class="btn"
                :speed="speed(itr)"
                :choices="set"
                @clicked="step"
              />
            </transition>
          </div>
        </div>

        <div :class="['actions']" v-show="idx > 1">
          <button
            :class="['btn', idx % 3 === 1 ? 'active' : 'btn-secondary']"
            type="reset"
            @click.prevent="restart"
            title="Restart"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="30"
              height="30"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path
                d="M10 15v4a3 3 0 0 0 3 3l4-9V2H5.72a2 2 0 0 0-2 1.7l-1.38 9a2 2 0 0 0 2 2.3zm7-13h2.67A2.31 2.31 0 0 1 22 4v7a2.31 2.31 0 0 1-2.33 2H17"
              ></path>
            </svg>
            <span>Restart</span>
          </button>

          <button
            v-show="idx % 3 === 1"
            :class="['btn', 'active']"
            type="submit"
            @click.prevent="save"
            title="Save"
            style="order: 1"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="30"
              height="30"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path
                d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"
              ></path>
            </svg>
          </button>

          <transition name="fade">
            <button
              v-show="complete"
              :class="['btn']"
              type="button"
              @click="share"
              title="Share your resolution"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="30"
                height="30"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <circle cx="18" cy="5" r="3"></circle>
                <circle cx="6" cy="12" r="3"></circle>
                <circle cx="18" cy="19" r="3"></circle>
                <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
                <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
              </svg>
            </button>
          </transition>
        </div>

        <footer class="credit">
          Created by Jason&rsquo;s New Year&rsquo;s Resolution Generator
        </footer>
      </article>

      <aside :class="['sidebar', { visible: showSidebar }]">
        <ResolutionLog :log="log" @clear="log = []"
          ><h2>
            My
            <time>{{ goalYear }}</time> Resolutions
          </h2>

          <button class="btn close" @click="showSidebar = false">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="30"
              height="30"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </ResolutionLog>
      </aside>
    </div>
  </main>
</template>

<script>
import html2canvas from "html2canvas";
import ButtonReel from "./components/ButtonReel.vue";
import ResolutionLog from "./components/ResolutionLog.vue";
import choices from "./choices.ts";

let restartTimeout;

const shuffle = (arr = []) => {
  for (let itr = arr.length - 1; itr > 0; itr--) {
    const idx = Math.floor(Math.random() * (itr + 1));
    [arr[itr], arr[idx]] = [arr[idx], arr[itr]];
  }

  return arr;
};

export default {
  name: "App",
  components: {
    ButtonReel,
    ResolutionLog,
  },
  data: () => ({
    idx: 1,
    rate: 10,
    value: "",
    log: [],
    sets: [],
    complete: false,
    isSharing: false,
    showSidebar: false,
    error: null,
  }),
  mounted() {
    this.shuffleDeck();
  },
  methods: {
    clearLog() {
      this.log = [];
    },
    shuffleDeck() {
      this.sets = Object.values(choices).map((v) => shuffle(v));
    },
    save() {
      this.log.push(`${this.value.trim()}.`);
      this.restart();
    },
    step(text) {
      this.value += `${text} `;
      this.idx++;
      this.complete = this.idx > this.sets.length;

      if (this.complete) {
        restartTimeout = setTimeout(this.restart, 30000);
      }
    },
    restart() {
      clearTimeout(restartTimeout);
      this.complete = false;
      this.isSharing = false;
      this.shuffleDeck();
      this.value = "";
      this.idx = -1;
      this.$nextTick(() => {
        this.idx = 1;
      });
      this.error = null;
    },
    speed(x) {
      return Math.min(1000, Math.max(10, this.rate * (this.sets.length - x)));
    },
    share() {
      clearTimeout(restartTimeout);
      this.isSharing = true;

      setTimeout(async () => {
        try {
          const canvas = await html2canvas(this.$refs.generated);
          const dataUrl = canvas.toDataURL();
          const blob = await (await fetch(dataUrl)).blob();
          const image = new File([blob], "resolution.png", {
            type: blob.type,
          });

          await navigator.share({
            title: `My ${this.goalYear} Resolution`,
            text: this.value,
            files: [image],
          });
        } catch (err) {
          this.error = err;
        }
        this.isSharing = false;
        this.restart();
      }, 1000);
    },
    shareLog() {
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
  computed: {
    goalYear() {
      const now = new Date();

      const year = now.getFullYear();

      if (now.getMonth() > 1) {
        return year + 1;
      }

      return year;
    },
    setKey() {
      const id = this.sets
        .flat()
        .join("")
        .replace(/[^A-Za-z]/, "");

      return `${id}`
        .split("")
        .sort(() => 0.5 - Math.random())
        .join("")
        .trim()
        .substring(50);
    },
  },
};
</script>

<style>
.sidebar {
  background-color: rgba(3, 3, 3, 0.8);
  color: #fff;
  height: 100%;
  min-height: 99vh;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  width: 80vw;
  transform: translateX(100%);
  transition: transform 0.3s ease-in-out;
  overflow-y: auto;
  z-index: 1;
}

.sidebar.visible {
  transform: translateX(0);
}

@supports (backdrop-filter: blur(10px)) {
  .sidebar {
    backdrop-filter: blur(10px);
  }
}
</style>
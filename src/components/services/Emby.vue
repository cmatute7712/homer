<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-6">
        <template v-if="item.subtitle">
          {{ item.subtitle }}
        </template>
        <template v-else>
          {{ embyCount }}
        </template>
      </p>
    </template>
    <template #indicator>
      <div v-if="status" class="status" :class="status">
        {{ status }}
      </div>
    </template>
  </Generic>
</template>

<script>
import service from "@/mixins/service.js";
import Generic from "./Generic.vue";

export default {
  name: "Emby",
  mixins: [service],
  props: {
    item: Object,
  },
  components: {
    Generic,
  },
  data: () => ({
    status: "",
    albumCount: 0,
    songCount: 0,
    movieCount: 0,
    seriesCount: 0,
    episodeCount: 0,
  }),
  computed: {    
    embyCount: function () {
      const libraryType = this.item.libraryType;
      const counts = [];

      libraryType.forEach((type, index) => {
        if (type === "music") {
          counts.push(`${this.songCount} songs`);
          counts.push(`${this.albumCount} albums`);
        } else if (type === "movies") {
          counts.push(`${this.movieCount} movies`);
        } else if (type === "series") {
          counts.push(`${this.episodeCount} episodes`);
          counts.push(`${this.seriesCount} series`);
        } else {
          return `wrong library type 💀`;
        }
      });

      const allowedParams = this.item.embyCountParams || "";
      const params = allowedParams.split(",").map((param) => param.trim());

      const filteredCounts = counts.filter((count, index) => {
        const countType = count.split(" ")[1];
        return params.some((param) => countType.includes(param));
      });

      return filteredCounts.join(", ");
    },
  },
  created() {
    this.fetchServerStatus();

    if (!this.item.subtitle && this.status !== "dead")
      this.fetchServerMediaStats();
  },
  methods: {
    fetchServerStatus: async function () {
      this.fetch("/System/info/public")
        .then((response) => {
          if (response.Id) this.status = "running";
          else throw new Error();
        })
        .catch((e) => {
          console.log(e);
          this.status = "dead";
        });
    },
    fetchServerMediaStats: async function () {
      const headers = {
        "X-Emby-Token": this.item.apikey,
      };

      var data = await this.fetch("/items/counts", { headers }).catch((e) => {
        console.log(e);
      });

      this.albumCount = data.AlbumCount;
      this.songCount = data.SongCount;
      this.movieCount = data.MovieCount;
      this.seriesCount = data.SeriesCount;
      this.episodeCount = data.EpisodeCount;
    },
  },
};
</script>

<style scoped lang="scss">
.status {
  font-size: 0.8rem;
  color: var(--text-title);

  &.running:before {
    background-color: #94e185;
    border-color: #78d965;
    box-shadow: 0 0 5px 1px #94e185;
  }

  &.dead:before {
    background-color: #c9404d;
    border-color: #c42c3b;
    box-shadow: 0 0 5px 1px #c9404d;
  }

  &:before {
    content: " ";
    display: inline-block;
    width: 7px;
    height: 7px;
    margin-right: 10px;
    border: 1px solid #000;
    border-radius: 7px;
  }
}
</style>

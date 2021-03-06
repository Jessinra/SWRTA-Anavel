<template>
  <div class="suggestionBoxContainer">
    <transition name="fade" mode="in-out">
      <div v-show="isMounted">
        <b-container>
          <b-row class="title bg-primary">
            <b-col>Suggested Picks</b-col>
          </b-row>
          <UnitBoxGrid
            :row="MAX_ROW"
            :col="MAX_COL"
            :units="suggestedUnits"
          />
        </b-container>
      </div>
    </transition>
  </div>
</template>

<script>
import UnitBoxGrid from '@/components/UnitBoxGrid.vue';
import { mapGetters, mapState } from 'vuex';

export default {
  components: {
    UnitBoxGrid,
  },
  data() {
    return {
      MAX_ROW: 1,
      MAX_COL: 5,
      isMounted: false,
    };
  },
  computed: {
    ...mapState(['URLS', 'DEVELOPMENT_MODE', 'DUMMY_RESPONSE']),
    ...mapGetters(['getPickIds', 'isAbleToPick']),
  },
  asyncComputed: {
    async suggestedUnits() {
      if (this.isAbleToPick) {
        return await this.fetchSuggestion({
          type: 'id',
          input: this.getPickIds,
          k: this.MAX_ROW * this.MAX_COL,
        });
      } else {
        return [];
      }
    },
  },
  methods: {
    async fetchSuggestion(payload) {
      if (this.DEVELOPMENT_MODE) {
        return this.DUMMY_RESPONSE['predictPick']['result'];
      }

      try {
        let response = await fetch(this.URLS.PREDICT_PICK_API, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(payload),
        });
        let content = await response.json();
        return content['result'];
      } catch (error) {
        return [];
      }
    },
  },
  mounted() {
    this.isMounted = true;
  },
};
</script>

<style scoped>
.suggestionBoxContainer {
  background-color: #2a2a2a;
  padding: 10px;
}

.title {
  font-size: 1vw;
  margin-bottom: 0.5rem;
  padding: 0.35rem 0;
}

.fade-enter-active {
  transition: opacity 2s ease-out;
}
.fade-enter {
  opacity: 0;
}

@media screen and (max-width: 991px) {
  .title {
    font-size: 4vw;
  }
}
</style>

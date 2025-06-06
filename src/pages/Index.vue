<template>
  <div id="page" :class="view + 'View'">
    <main>
      <view-deck-flat
        v-if="view === 'flat'"
        :groupData="deckData"
        :selectable="true"
        :level="1"
        @selection-changed="setSelection"
      />
      <view-deck-card-group
        v-else
        :groupData="deckData"
        :level="1"
        :selectable="true"
        @selection-changed="setSelection"
      />
    </main>
    <footer :class="{overBudget: !$root.noBudgetMode && remaining < 0}">
      <div>
        <div v-if="!$root.noBudgetMode">
          <span class="remaining">
            Remaining Coins: {{ remaining }}
          </span>
          <span class="budget">
            Budget: {{this.deckData.budget}}
          </span>
        </div>
        <div v-else>
          <span>
            Coins Spent: {{ spent }}
          </span>
        </div>
        <div>
          <label>
            Random/Hell Mode (no budget):
            <input type="checkbox" v-model="$root.noBudgetMode">
          </label>
        </div>
      </div>
      <div v-if="$root.deckDataModified" >
        <button
          @click="$root.discardDraft"
          class="danger"
        >
          Discard Changes
        </button>
        <button
          @click="$router.push({name: 'Source'})"
          class="success"
        >
          Save Unsaved Changes
        </button>
        <button
          @click="$router.push({name: 'Editor'})"
        >
          Continue Editing
        </button>
      </div>
      <div>
        <button
          @click="toggleView"
          v-text="(this.view === 'card' ? 'Flat' : 'Card') + ' View'"
        />
        <button
          :disabled="!!changeRequiredMsg"
          @click="confirmChoices"
          v-text="changeRequiredMsg ? changeRequiredMsg : 'Confirm Choices'"
        />
      </div>
    </footer>
  </div>
</template>

<script>
import ViewDeckCardGroup from '@/components/ViewDeckCardGroup.vue'
import ViewDeckFlat from '@/components/ViewDeckFlat.vue'
import deckMixin from '@/mixins/deck'

export default {
  name: 'Index',
  mixins: [deckMixin],
  props: ['hash'],
  created () {
    if (this.metaDeckData.isIssue) {
      return this.$router.push({name: 'DataDebug'})
    }
  },
  data () {
    return {
      view: 'card'
    }
  },
  computed: {
    spent () {
      return this.selected
        .map(choice => choice.cost || 0)
        .reduce((a, b) => a + b, 0)
    },
    remaining () {
      return this.deckData.budget - this.spent
    },
    groupsNeedingMoreSelections () {
      return this.choiceGroups
        .filter(group => group.minSelectable > 0)
        .filter(group => this.subtreeSelected(group).length < group.minSelectable)
    },
    changeRequiredMsg () {
      const msgs = []
      if (this.groupsNeedingMoreSelections.length > 0) {
        msgs.push('More Selections Required')
      }
      if (!this.$root.noBudgetMode && this.remaining < 0) {
        msgs.push('Over Budget')
      }
      return msgs.length > 0 ? msgs.join(' & ') : null
    }
  },
  components: {ViewDeckCardGroup, ViewDeckFlat},
  methods: {
    toggleView () {
      this.view = this.view === 'card' ? 'flat' : 'card'
    },
    confirmChoices () {
      this.$router.push({
        name: 'Share',
        params: {hash: this.getStatusHash()}
      })
    }
  }
}
</script>

<style scoped lang="scss">
  @import '../mixins/deck.scss';
  @import '../mixins/ui.scss';

  /* Custom CSS */
  /* ... */
</style>

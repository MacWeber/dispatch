<template>
  <v-container grid-list-md>
    <query-select v-model="selectedQuery" :project="project" />
    <v-list>
      <v-list-item v-for="(i, idx) in value" :key="i.id">
        <v-list-item-content>
          <v-list-item-title>
            {{ i.name }}
          </v-list-item-title>
          <v-list-item-subtitle>
            {{ i.description }}
          </v-list-item-subtitle>
        </v-list-item-content>
        <v-list-item-action>
          <v-btn icon @click="remove(idx)">
            <v-icon> mdi-delete </v-icon>
          </v-btn>
        </v-list-item-action>
      </v-list-item>
    </v-list>
  </v-container>
</template>

<script>
import { mapFields } from "vuex-map-fields"
import QuerySelect from "@/data/query/QuerySelect.vue"
import { cloneDeep } from "lodash"

export default {
  name: "SourceEditQueriesTab",

  components: {
    QuerySelect,
  },

  props: {
    value: {
      type: Array,
      default: function () {
        return []
      },
    },
    project: {
      type: Object,
      default: null,
    },
  },

  computed: {
    ...mapFields("source", ["selected.queries", "selected.loading"]),
  },

  methods: {
    add() {
      const value = cloneDeep(this.value)
      value.push(this.selectedQuery)
      this.selectedQuery = null
      this.$emit("input", value)
    },
    remove(idx) {
      const value = cloneDeep(this.value)
      value.splice(idx, 1)
      this.$emit("input", value)
    },
  },

  created() {
    this.$watch(
      (vm) => [vm.selectedQuery],
      () => {
        if (!this.selectedQuery) return
        this.add()
      }
    )
  },

  data() {
    return {
      selectedQuery: null,
    }
  },
}
</script>

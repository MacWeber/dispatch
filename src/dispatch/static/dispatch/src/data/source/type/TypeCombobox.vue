<template>
  <v-combobox
    :items="items"
    :label="label"
    :loading="loading"
    :search-input.sync="search"
    @update:search-input="getFilteredData()"
    chips
    clearable
    deletable-chips
    hide-selected
    item-text="name"
    item-value="id"
    multiple
    no-filter
    v-model="types"
  >
    <template #no-data>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>
            No Projects matching "
            <strong>{{ search }}</strong
            >".
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </template>
    <template #item="data">
      <template>
        <v-list-item-content>
          <v-list-item-title>{{ data.item.name }}</v-list-item-title>
          <v-list-item-subtitle style="width: 200px" class="text-truncate">
            {{ data.item.description }}
          </v-list-item-subtitle>
        </v-list-item-content>
      </template>
    </template>
    <template #append-item>
      <v-list-item v-if="more" @click="loadMore()">
        <v-list-item-content>
          <v-list-item-subtitle> Load More </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </template>
  </v-combobox>
</template>

<script>
import { cloneDeep, debounce } from "lodash"

import SearchUtils from "@/search/utils"
import TypeApi from "@/data/source/type/api"

export default {
  name: "SourceTypeComboBox",
  props: {
    value: {
      type: Array,
      default: function () {
        return []
      },
    },
    label: {
      type: String,
      default: function () {
        return "Types"
      },
    },
  },

  data() {
    return {
      loading: false,
      items: [],
      more: false,
      numItems: 5,
      search: null,
    }
  },

  computed: {
    types: {
      get() {
        return cloneDeep(this.value)
      },
      set(value) {
        this.search = null
        let _types = value.filter((v) => {
          if (typeof v === "string") {
            return false
          }
          return true
        })
        this.$emit("input", _types)
      },
    },
  },

  created() {
    this.fetchData({})
  },

  methods: {
    loadMore() {
      this.numItems = this.numItems + 5
      this.fetchData()
    },
    fetchData() {
      this.error = null
      this.loading = "error"

      let filterOptions = {
        q: this.search,
        itemsPerPage: this.numItems,
      }

      filterOptions = SearchUtils.createParametersFromTableOptions({ ...filterOptions })

      TypeApi.getAll(filterOptions).then((response) => {
        this.items = response.data.items
        this.total = response.data.total

        if (this.items.length < this.total) {
          this.more = true
        } else {
          this.more = false
        }

        this.loading = false
      })
    },
    getFilteredData: debounce(function () {
      this.fetchData()
    }, 500),
  },
}
</script>

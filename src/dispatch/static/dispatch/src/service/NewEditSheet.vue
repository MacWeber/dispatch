<template>
  <ValidationObserver v-slot="{ invalid, validated }">
    <v-navigation-drawer v-model="showCreateEdit" app clipped right width="500">
      <template #prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title v-if="id" class="title"> Edit </v-list-item-title>
            <v-list-item-title v-else class="title"> New </v-list-item-title>
            <v-list-item-subtitle>Service</v-list-item-subtitle>
          </v-list-item-content>
          <v-btn icon color="info" :disabled="invalid || !validated" @click="save()">
            <v-icon>save</v-icon>
          </v-btn>
          <v-btn icon color="secondary" @click="closeCreateEdit()">
            <v-icon>close</v-icon>
          </v-btn>
        </v-list-item>
      </template>
      <v-card flat>
        <v-card-text>
          <v-container grid-list-md>
            <v-layout wrap>
              <v-flex xs12>
                <span class="subtitle-2">Details</span>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="Name" rules="required" immediate>
                  <v-text-field
                    v-model="name"
                    slot-scope="{ errors, valid }"
                    :error-messages="errors"
                    :success="valid"
                    label="Name"
                    hint="A name for your service."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="Description" rules="required" immediate>
                  <v-textarea
                    v-model="description"
                    slot-scope="{ errors, valid }"
                    label="Description"
                    :error-messages="errors"
                    :success="valid"
                    hint="A description for your service."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="Type" rules="required" immediate>
                  <v-select
                    v-model="type"
                    slot-scope="{ errors, valid }"
                    :loading="loading"
                    :items="oncall_plugins"
                    label="Type"
                    :error-messages="errors"
                    hint="Oncall plugin to use."
                    :success="valid"
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="External Id" rules="required" immediate>
                  <v-text-field
                    v-model="external_id"
                    slot-scope="{ errors, valid }"
                    label="External Id"
                    :error-messages="errors"
                    :success="valid"
                    hint="An external identifier for service."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <v-checkbox v-model="health_metrics" label="Collect Health Metrics" />
              </v-flex>
              <v-flex xs12>
                <v-checkbox v-model="is_active" label="Enabled" />
              </v-flex>
              <v-flex xs12>
                <span class="subtitle-2"
                  >Engagement
                  <v-tooltip max-width="250px" bottom>
                    <template #activator="{ on, attrs }">
                      <v-icon v-bind="attrs" v-on="on"> help_outline </v-icon>
                    </template>
                    This service will be used to automatically engage services for any incident or
                    case matching the following filters.
                  </v-tooltip>
                </span>
              </v-flex>
              <v-flex xs12>
                <search-filter-combobox
                  v-model="filters"
                  :project="project"
                  label="Filters"
                  hint="Select one or more filters that will determine when a service is engaged."
                />
              </v-flex>
              <v-flex xs12>
                <span class="subtitle-2"
                  >Evergreen
                  <v-tooltip max-width="250px" bottom>
                    <template #activator="{ on, attrs }">
                      <v-icon v-bind="attrs" v-on="on"> help_outline </v-icon>
                    </template>
                    Dispatch will send the owner a reminder email to the resource owner, reminding
                    them to keep the resource current.
                  </v-tooltip>
                </span>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="Owner" immediate>
                  <v-text-field
                    v-model="evergreen_owner"
                    slot-scope="{ errors, valid }"
                    label="Owner"
                    :error-messages="errors"
                    :success="valid"
                    hint="Owner of this service."
                    clearable
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="Reminder Interval" immediate>
                  <v-text-field
                    v-model="evergreen_reminder_interval"
                    slot-scope="{ errors, valid }"
                    label="Reminder Interval"
                    :error-messages="errors"
                    :success="valid"
                    type="number"
                    hint="Number of days that should elapse between reminders sent to the service owner."
                    placeholder="90"
                    clearable
                    min="1"
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <v-checkbox
                  v-model="evergreen"
                  hint="Enabling evergreen will send periodic reminders to the owner to update this service."
                  label="Enabled"
                />
              </v-flex>
            </v-layout>
          </v-container>
        </v-card-text>
      </v-card>
    </v-navigation-drawer>
  </ValidationObserver>
</template>

<script>
import { mapFields } from "vuex-map-fields"
import { mapActions } from "vuex"
import { ValidationObserver, ValidationProvider, extend } from "vee-validate"
import { required } from "vee-validate/dist/rules"

import SearchFilterCombobox from "@/search/SearchFilterCombobox.vue"
import SearchUtils from "@/search/utils"
import PluginApi from "@/plugin/api"

extend("required", {
  ...required,
  message: "This field is required",
})

export default {
  name: "ServiceNewEditSheet",

  components: {
    ValidationObserver,
    ValidationProvider,
    SearchFilterCombobox,
  },

  computed: {
    ...mapFields("service", [
      "selected.description",
      "selected.evergreen",
      "selected.evergreen_owner",
      "selected.evergreen_reminder_interval",
      "selected.external_id",
      "selected.filters",
      "selected.health_metrics",
      "selected.id",
      "selected.is_active",
      "selected.loading",
      "selected.name",
      "selected.project",
      "selected.type",
      "dialogs.showCreateEdit",
    ]),
    ...mapFields("route", ["query"]),
  },

  methods: {
    ...mapActions("service", ["closeCreateEdit"]),
    save() {
      const self = this
      this.$store.dispatch("service/save").then(function (data) {
        self.$emit("new-service-created", data)
      })
    },
  },

  data() {
    return {
      oncall_plugins: [],
    }
  },

  created() {
    if (this.query.project) {
      this.project = { name: this.query.project }
    }
    this.loading = "error"

    let filterOptions = {
      itemsPerPage: -1,
    }

    if (this.project) {
      filterOptions = {
        ...filterOptions,
        filters: {
          project: [this.project],
        },
      }
    }

    let typeFilter = [
      {
        model: "Plugin",
        field: "type",
        op: "==",
        value: "oncall",
      },
    ]

    filterOptions = SearchUtils.createParametersFromTableOptions({ ...filterOptions }, typeFilter)

    PluginApi.getAllInstances(filterOptions).then((response) => {
      this.loading = false
      this.oncall_plugins = response.data.items.map((p) => p.plugin.slug)
    })
  },
}
</script>

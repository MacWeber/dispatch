<template>
  <ValidationObserver v-slot="{ invalid, validated }">
    <v-navigation-drawer
      v-model="showCreateEdit"
      app
      clipped
      right
      width="800"
      :permanent="$vuetify.breakpoint.mdAndDown"
    >
      <template #prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title v-if="id" class="title"> Edit </v-list-item-title>
            <v-list-item-title v-else class="title"> New </v-list-item-title>
            <v-list-item-subtitle>Status</v-list-item-subtitle>
          </v-list-item-content>
          <v-btn
            icon
            color="info"
            :loading="loading"
            :disabled="invalid || !validated"
            @click="save()"
          >
            <v-icon>save</v-icon>
          </v-btn>
          <v-btn icon color="secondary" @click="closeCreateEdit">
            <v-icon>close</v-icon>
          </v-btn>
        </v-list-item>
        <v-card flat>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12>
                  <ValidationProvider name="Name" rules="required" immediate>
                    <v-text-field
                      v-model="name"
                      slot-scope="{ errors, valid }"
                      :error-messages="errors"
                      :success="valid"
                      label="Name"
                      hint="Name of status."
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
                      :error-messages="errors"
                      :success="valid"
                      label="Description"
                      hint="Description of status."
                      clearable
                      required
                    />
                  </ValidationProvider>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>
        </v-card>
      </template>
    </v-navigation-drawer>
  </ValidationObserver>
</template>

<script>
import { mapFields } from "vuex-map-fields"
import { mapActions } from "vuex"
import { ValidationProvider, ValidationObserver, extend } from "vee-validate"
import { required } from "vee-validate/dist/rules"

extend("required", {
  ...required,
  message: "This field is required",
})

export default {
  name: "SourceStatusNewEditSheet",

  components: {
    ValidationProvider,
    ValidationObserver,
  },

  computed: {
    ...mapFields("sourceStatus", [
      "selected.project",
      "selected.id",
      "selected.name",
      "selected.description",
      "selected.loading",
      "dialogs.showCreateEdit",
    ]),
    ...mapFields("route", ["query"]),
  },

  methods: {
    ...mapActions("sourceStatus", ["closeCreateEdit"]),
    save() {
      const self = this
      this.$store.dispatch("sourceStatus/save").then(function (data) {
        self.$emit("new-source-status-created", data)
      })
    },
  },

  created() {
    if (this.query.project) {
      this.project = { name: this.query.project }
    }
  },
}
</script>

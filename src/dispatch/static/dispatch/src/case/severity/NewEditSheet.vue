<template>
  <ValidationObserver v-slot="{ invalid, validated }">
    <v-navigation-drawer v-model="showCreateEdit" app clipped right width="500">
      <template #prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title v-if="id" class="title"> Edit </v-list-item-title>
            <v-list-item-title v-else class="title"> New </v-list-item-title>
            <v-list-item-subtitle>Case severity</v-list-item-subtitle>
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
                    hint="A name for your case severity."
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
                    hint="A description for your case severity."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="View Order" rules="required" immediate>
                  <v-text-field
                    v-model="view_order"
                    slot-scope="{ errors, valid }"
                    label="View Order"
                    :error-messages="errors"
                    :success="valid"
                    type="number"
                    hint="Enter a value to indicate the order in which you want this severity to be shown in a list (lowest numbers are shown first)."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <color-picker-input label="Color" v-model="color" />
              </v-flex>
              <v-flex xs12>
                <v-checkbox
                  v-model="default_case_severity"
                  label="Default Case Severity"
                  hint="Check if this case severity should be the default."
                />
              </v-flex>
              <v-flex xs12>
                <v-checkbox
                  v-model="enabled"
                  label="Enabled"
                  hint="Determines whether this case severity is availible for new cases."
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
import { ValidationObserver, ValidationProvider, extend } from "vee-validate"
import { mapActions } from "vuex"
import { mapFields } from "vuex-map-fields"
import { required } from "vee-validate/dist/rules"

import ColorPickerInput from "@/components/ColorPickerInput.vue"

extend("required", {
  ...required,
  message: "This field is required",
})

export default {
  name: "CaseSeverityNewEditSheet",

  components: {
    ColorPickerInput,
    ValidationObserver,
    ValidationProvider,
  },

  data() {
    return {}
  },

  computed: {
    ...mapFields("case_severity", [
      "dialogs.showCreateEdit",
      "selected.color",
      "selected.default",
      "selected.description",
      "selected.enabled",
      "selected.id",
      "selected.loading",
      "selected.name",
      "selected.project",
      "selected.view_order",
    ]),
    ...mapFields("case_severity", {
      default_case_severity: "selected.default",
    }),
    ...mapFields("route", ["query"]),
  },

  methods: {
    ...mapActions("case_severity", ["save", "closeCreateEdit"]),
  },

  created() {
    if (this.query.project) {
      this.project = { name: this.query.project }
    }
  },
}
</script>

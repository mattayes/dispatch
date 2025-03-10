<template>
  <ValidationObserver v-slot="{ invalid, validated }">
    <v-navigation-drawer v-model="showCreateEdit" app clipped right width="500">
      <template #prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title v-if="id" class="title"> Edit </v-list-item-title>
            <v-list-item-title v-else class="title"> New </v-list-item-title>
            <v-list-item-subtitle>Workflow</v-list-item-subtitle>
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
                <ValidationProvider name="name" rules="required" immediate>
                  <v-text-field
                    v-model="name"
                    slot-scope="{ errors, valid }"
                    label="Name"
                    :error-messages="errors"
                    :success="valid"
                    hint="A name for your workflow."
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="resourceId" rules="required" immediate>
                  <v-text-field
                    v-model="resource_id"
                    slot-scope="{ errors, valid }"
                    label="Resource Id"
                    :error-messages="errors"
                    :success="valid"
                    required
                    hint="External resource id that refers to this workflow."
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <ValidationProvider name="description" rules="required" immediate>
                  <v-textarea
                    v-model="description"
                    slot-scope="{ errors, valid }"
                    label="Description"
                    :error-messages="errors"
                    :success="valid"
                    hint="The workflow's description."
                    clearable
                    required
                  />
                </ValidationProvider>
              </v-flex>
              <v-flex xs12>
                <plugin-instance-combobox
                  v-model="plugin_instance"
                  type="workflow"
                  :project="project"
                  label="Plugin"
                />
              </v-flex>
              <v-flex xs12>
                <v-checkbox
                  v-model="enabled"
                  hint="Disabled workflows will not be made available for selection during incidents."
                  label="Enabled"
                />
              </v-flex>
              <v-flex xs12>
                <workflow-parameters-input v-model="parameters" />
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

import PluginInstanceCombobox from "@/plugin/PluginInstanceCombobox.vue"
import WorkflowParametersInput from "@/workflow/WorkflowParametersInput.vue"

extend("required", {
  ...required,
  message: "This field is required",
})

export default {
  name: "WorkflowNewEditSheet",

  components: {
    ValidationObserver,
    ValidationProvider,
    PluginInstanceCombobox,
    WorkflowParametersInput,
  },

  computed: {
    ...mapFields("workflow", [
      "selected.name",
      "selected.description",
      "selected.enabled",
      "selected.resource_id",
      "selected.parameters",
      "selected.id",
      "selected.plugin_instance",
      "selected.project",
      "selected.loading",
      "dialogs.showCreateEdit",
    ]),
    ...mapFields("route", ["query"]),
  },

  methods: {
    ...mapActions("workflow", ["save", "closeCreateEdit"]),
  },

  created() {
    this.project = { name: this.query.project }
  },
}
</script>

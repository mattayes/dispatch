<template>
  <div>
    <v-data-table :headers="headers" :items="items">
      <template #item.name="{ item }">
        <a :href="item.weblink" target="_blank" style="text-decoration: none">
          {{ item.name }}
          <v-icon small>open_in_new</v-icon>
        </a>
      </template>
      <template #item.project.name="{ item }">
        <v-chip small :color="item.project.color" text-color="white">
          {{ item.project.name }}
        </v-chip>
      </template>
      <template #item.data-table-actions="{ item }">
        <v-menu bottom left>
          <template #activator="{ on }">
            <v-btn icon v-on="on">
              <v-icon>mdi-dots-vertical</v-icon>
            </v-btn>
          </template>
          <v-list>
            <v-list-item @click="createEditShow(item)">
              <v-list-item-title>View / Edit</v-list-item-title>
            </v-list-item>
            <v-list-item @click="removeShow(item)">
              <v-list-item-title>Delete</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import { mapActions } from "vuex"
export default {
  name: "DocumentSummaryTable",

  components: {},
  data() {
    return {
      headers: [
        { text: "Name", value: "name", sortable: false },
        { text: "Description", value: "description", sortable: false },
        { text: "Project", value: "project.name", sortable: true },
        { text: "", value: "data-table-actions", sortable: false, align: "end" },
      ],
    }
  },

  props: {
    items: {
      default: function () {
        return []
      },
      type: Array,
    },
    loading: {
      default: function () {
        return false
      },
      type: [String, Boolean],
    },
  },

  methods: {
    ...mapActions("document", ["createEditShow", "removeShow"]),
  },
}
</script>

<template>
  <v-container fluid grid-list-xl>
    <!-- <cases-drill-down-sheet -->
    <!--   :show="showDrillDown" -->
    <!--   :items="detailItems" -->
    <!--   @close="showDrillDown = false" -->
    <!-- /> -->
    <v-layout row wrap>
      <v-flex class="d-flex justify-start" lg6 sm6 xs12>
        <v-btn color="info" @click="copyView"> Share View </v-btn>
      </v-flex>
      <v-flex class="d-flex justify-end" lg6 sm6 xs12>
        <case-dialog-filter
          @update="update"
          @loading="setLoading"
          :projects="defaultUserProjects"
        />
      </v-flex>
    </v-layout>
    <v-layout row wrap>
      <!-- Widgets Start -->
      <v-flex lg3 sm6 xs12>
        <stat-widget icon="domain" :title="totalCases | toNumberString" sup-title="Cases" />
      </v-flex>
      <v-flex lg3 sm6 xs12>
        <stat-widget
          icon="domain"
          :title="totalCasesTriaged | toNumberString"
          sup-title="Cases Triaged"
        />
      </v-flex>
      <v-flex lg3 sm6 xs12>
        <stat-widget
          icon="domain"
          :title="totalCasesEscalated | toNumberString"
          sup-title="Cases Escalated"
        />
      </v-flex>
      <v-flex lg3 sm6 xs12>
        <stat-widget
          icon="watch_later"
          :title="totalHours | toNumberString"
          sup-title="Total Hours (New to Closed)"
        />
      </v-flex>
      <!-- Widgets Ends -->
      <!-- Statistics Start -->
      <v-flex lg6 sm6 xs12>
        <case-type-bar-chart-card
          v-model="groupedItems"
          :loading="loading"
          @detailsSelected="detailsSelected($event)"
        />
      </v-flex>
      <v-flex lg6 sm6 xs12>
        <case-severity-bar-chart-card
          v-model="groupedItems"
          :loading="loading"
          @detailsSelected="detailsSelected($event)"
        />
      </v-flex>
      <!-- <v-flex lg6 sm6 xs12> -->
      <!--   <case-priority-bar-chart-card -->
      <!--     v-model="groupedItems" -->
      <!--     :loading="loading" -->
      <!--     @detailsSelected="detailsSelected($event)" -->
      <!--   /> -->
      <!-- </v-flex> -->
      <v-flex lg6 sm6 xs12>
        <case-new-triage-average-time-card v-model="groupedItems" :loading="loading" />
      </v-flex>
      <v-flex lg6 sm6 xs12>
        <case-triage-escalated-average-time-card v-model="groupedItems" :loading="loading" />
      </v-flex>
      <v-flex lg6 sm6 xs12>
        <case-escalated-closed-average-time-card v-model="groupedItems" :loading="loading" />
      </v-flex>
      <v-flex lg6 sm6 xs12>
        <case-new-closed-average-time-card v-model="groupedItems" :loading="loading" />
      </v-flex>
      <!-- Statistics Ends -->
    </v-layout>
  </v-container>
</template>

<script>
import { mapFields } from "vuex-map-fields"
import { groupBy, sumBy } from "lodash"
import { parseISO } from "date-fns"
import differenceInHours from "date-fns/differenceInHours"

import CaseDialogFilter from "@/dashboard/case/CaseDialogFilter.vue"
// import CasesDrillDownSheet from "@/dashboard/case/CasesDrillDownSheet.vue"
// import CasePriorityBarChartCard from "@/dashboard/case/CasePriorityBarChartCard.vue"
import CaseEscalatedClosedAverageTimeCard from "@/dashboard/case/CaseEscalatedClosedAverageTimeCard.vue"
import CaseNewClosedAverageTimeCard from "@/dashboard/case/CaseNewClosedAverageTimeCard.vue"
import CaseNewTriageAverageTimeCard from "@/dashboard/case/CaseNewTriageAverageTimeCard.vue"
import CaseSeverityBarChartCard from "@/dashboard/case/CaseSeverityBarChartCard.vue"
import CaseTriageEscalatedAverageTimeCard from "@/dashboard/case/CaseTriageEscalatedAverageTimeCard.vue"
import CaseTypeBarChartCard from "@/dashboard/case/CaseTypeBarChartCard.vue"
import StatWidget from "@/components/StatWidget.vue"

export default {
  name: "CaseDashboard",

  components: {
    CaseDialogFilter,
    // CasePriorityBarChartCard,
    CaseSeverityBarChartCard,
    CaseTypeBarChartCard,
    // CasesDrillDownSheet,
    CaseEscalatedClosedAverageTimeCard,
    CaseNewClosedAverageTimeCard,
    CaseNewTriageAverageTimeCard,
    CaseTriageEscalatedAverageTimeCard,
    StatWidget,
  },

  data() {
    return {
      tab: null,
      loading: "error",
      items: [],
      detailItems: [],
      showDrillDown: false,
    }
  },

  methods: {
    update(data) {
      this.items = data.filter(function (item) {
        return (
          !item.case_type.exclude_from_metrics &&
          !item.duplicates.filter((d) => data.includes(d)).length
        )
      })
    },
    detailsSelected(event) {
      this.detailItems = event
      this.showDrillDown = true
    },
    setLoading(data) {
      this.loading = data
    },
    copyView: function () {
      let store = this.$store
      this.$copyText(window.location).then(
        function () {
          store.commit(
            "notification_backend/addBeNotification",
            {
              text: "View copied to clipboard.",
            },
            { root: true }
          )
        },
        function () {
          store.commit(
            "notification_backend/addBeNotification",
            {
              text: "Failed to copy view to clipboard.",
              color: "red",
            },
            { root: true }
          )
        }
      )
    },
  },

  computed: {
    ...mapFields("route", ["query.project"]),
    ...mapFields("auth", ["currentUser.projects"]),

    casesByYear() {
      return groupBy(this.items, function (item) {
        return parseISO(item.reported_at).getYear()
      })
    },
    casesByMonth() {
      // add year info if necessary
      if (Object.keys(this.casesByYear).length > 1) {
        return groupBy(this.items, function (item) {
          return parseISO(item.reported_at).toLocaleString("default", {
            month: "short",
            year: "numeric",
          })
        })
      } else {
        return groupBy(this.items, function (item) {
          return parseISO(item.reported_at).toLocaleString("default", {
            month: "short",
          })
        })
      }
    },
    casesByQuarter() {
      return groupBy(this.items, function (item) {
        return "Q" + Math.floor(parseISO(item.reported_at).getMonth() + 3) / 3
      })
    },
    groupedItems() {
      return this.casesByMonth
    },
    totalCases() {
      return this.items.length
    },
    totalCasesTriaged() {
      return sumBy(this.items, function (item) {
        if (item.triage_at) {
          return 1
        }
      })
    },
    totalCasesEscalated() {
      return sumBy(this.items, function (item) {
        if (item.escalated_at && item.incidents.length > 0) {
          return 1
        }
      })
    },
    totalHours() {
      return sumBy(this.items, function (item) {
        let endTime = new Date().toISOString()
        if (item.closed_at) {
          endTime = item.closed_at
        }
        return differenceInHours(parseISO(endTime), parseISO(item.reported_at))
      })
    },
    defaultUserProjects: {
      get() {
        let d = null
        if (this.projects) {
          let d = this.projects.filter((v) => v.default === true)
          return d.map((v) => v.project)
        }
        return d
      },
    },
  },
}
</script>

<template>
  <q-page style="background: `grey`">
    <q-form @submit="this.search">
      <q-input color="primary" filled v-model="query" label="Search">
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
    </q-form>
    <q-list bordered separator>
      <q-item v-for="obj in this.results" :key="obj">
        <q-item-section>
          <q-expansion-item
            :label="obj[`Model Name`]"
            :caption="obj[`Task`] + ` --- ` + obj[`Training Data`].join(`, `)"
            expand-separator
          >
            <q-markup-table>
              <thead>
                <tr>
                  <th class="text-left">Property</th>
                  <th class="text-right">Value</th>
                </tr>
                <tr v-for="k in Object.keys(obj)" :key="k">
                  <td class="text-left">{{ k }}</td>
                  <td class="text-right">{{ obj[k] }}</td>
                </tr>
              </thead>
            </q-markup-table>
          </q-expansion-item>
        </q-item-section>
        <q-item-section side top>
          <q-btn
            size="lg"
            icon="link"
            color="secondary"
            flat
            :href="obj[`Url`]"
            target="_blank"
          />
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>
<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "IndexPage",
  props: {
    filters: null, // Strangely, initializing as array causes a type mismatch
    models: null,
  },
  data() {
    return {
      results: [],
      query: "",
    };
  },
  watch: {
    filters: {
      handler(n, o) {
        this.search();
      },
      deep: true,
    },
  },
  methods: {
    search() {
      var res = this.models.chain();

      // Name
      if (this.query.length > 0) {
        res.find({ "Model Name": { $contains: this.query } });
      }

      // Data
      var datasetFilters = this.filters
        .filter((f) => f.attribute == "Training Data")
        .map((f) => f.keyword);
      if (datasetFilters.length > 0) {
        res.find({ "Training Data": { $contains: datasetFilters } });
      }

      // Type
      var typeFilters = this.filters
        .filter((f) => f.attribute == "Type")
        .map((f) => f.keyword);
      if (typeFilters.length > 0) {
        res.find({ Type: { $in: typeFilters } });
      }

      // Task
      var taskFilters = this.filters
        .filter((f) => f.attribute == "Task")
        .map((f) => f.keyword);
      if (taskFilters.length > 0) {
        res.find({ Task: { $in: taskFilters } });
      }

      // Evaluation Result
      var evalFilters = this.filters
        .filter((f) => f.attribute == "Evaluation Results")
        .map((f) => {
          return { name: f.keyword, value: f.value };
        });
      if (evalFilters.length > 0) {
        res.where(function (obj) {
          return evalFilters.every((p) => {
            return obj["Evaluation Results"].some((c) => {
              return c.metric.includes(p.name) && c.value > p.value;
            });
          });
        });
      }
      this.results = res.data();
    },
  },
});
</script>

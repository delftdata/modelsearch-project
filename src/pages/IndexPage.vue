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
            @show="tabs[obj[`Model Name`]] = `Main`"
            :label="obj[`Model Name`]"
            :caption="obj[`Task`] + ` --- ` + obj[`Training Data`].join(`, `)"
            expand-separator
          >
            <q-tabs v-model="tabs[obj[`Model Name`]]" stretch>
              <q-tab name="Main" label="Main" />
              <q-tab name="Datasets" label="Datasets" />
              <q-tab name="Metrics" label="Metrics" />
              <q-tab name="Hyperparameters" label="Hyperparameters" />
            </q-tabs>

            <q-tab-panels v-model="tabs[obj[`Model Name`]]" animated>
              <q-tab-panel name="Main">
                <q-markup-table>
                  <thead>
                    <tr>
                      <th class="text-left">Property</th>
                      <th class="text-right">Value</th>
                    </tr>
                    <tr
                      v-for="k in [`Model Name`, `Type`, `Task`, `Url`]"
                      :key="k"
                    >
                      <td class="text-left">{{ k }}</td>
                      <td class="text-right">{{ obj[k] }}</td>
                    </tr>
                  </thead>
                </q-markup-table>
              </q-tab-panel>
              <q-tab-panel name="Datasets">
                <q-list>
                  <q-card>
                    <q-card-section><b>Datasets</b></q-card-section>
                    <q-separator></q-separator>
                    <q-list-item v-for="v in obj[`Training Data`]" :key="v">
                      <q-card-section
                        ><a
                          :href="`https://huggingface.co/datasets/${v}`"
                          target="_blank"
                          >{{ v }}</a
                        ></q-card-section
                      >
                    </q-list-item>
                  </q-card>
                </q-list>
              </q-tab-panel>
              <q-tab-panel name="Metrics">
                <q-markup-table>
                  <thead>
                    <tr>
                      <th class="text-left">Metric</th>
                      <th class="text-right">Value</th>
                    </tr>
                    <tr v-for="k in obj[`Evaluation Results`]" :key="k">
                      <td class="text-left">{{ k[`metric`] }}</td>
                      <td class="text-right">
                        {{ k[`value`] }}
                      </td>
                    </tr>
                  </thead>
                </q-markup-table>
              </q-tab-panel>
              <q-tab-panel name="Hyperparameters"></q-tab-panel>
            </q-tab-panels>
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
      tabs: {},
    };
  },
  watch: {
    filters: {
      handler(n, o) {
        if (this.query.length > 0 || n.length > 0) {
          this.search();
        } else {
          this.results = [];
        }
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
              return (
                c.metric.toLowerCase().includes(p.name) && c.value > p.value
              );
            });
          });
        });
      }
      this.results = res.data();
    },
  },
});
</script>

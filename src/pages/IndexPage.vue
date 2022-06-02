<template>
  <q-page style="background: `grey`">
    <q-form @submit="this.search">
      <q-input color="primary" filled v-model="query" label="Search">
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
    </q-form>

    <div class="q-pa-md" v-if="this.docFlag && this.results.length == 0">
      <div class="row">
        <div class="col"></div>

        <div class="col-8">
          <p class="q-pa-lg q-mb-none text-h3 text-center">
            A metadata search engine enabling exploration of the full ML
            life-cycle.
          </p>
          <p class="q-pa-lg q-mb-none text-body1 text-center">
            ModelSearch enables retrieving ML models and relevant attributes
            based on our proposed metadata representation, facilitating complex
            inference queries over model zoo.
          </p>
        </div>

        <div class="col"></div>
      </div>
    </div>

    <div class="row" v-if="this.docFlag && this.results.length == 0">
      <div class="col-1"></div>
      <div class="col-10">
        <p class="q-pa-lg q-mb-none text-h4 text-left">How it works</p>
        <p class="q-pa-lg q-mb-none text-body1 text-left">
          The left panel shows different filtering conditions: Type, Task,
          Training Data, Hyperparameters, and Evaluation Results. You can select
          multiple conditions and the selected filters will be listed below.
          Besides selecting options from the drop-down menu, you can also set
          constraints on certain attributes. The selected filters will be
          presented to remind users. The models that can meet the specified
          conditions will be shown on the right panel, as a form of a expandable
          model card.
        </p>
        <div class="q-pa-md">
          <div class="q-col-gutter-md row items-start">
            <div class="col-2"></div>
            <div class="col-3">
              <q-img
                fit="scale-down"
                style="height: 370px; max-width: 300px"
                src="task-screenshot.png"
              >
                <div class="absolute-bottom-right text-body2 text-center">
                  Select from options
                </div>
              </q-img>
            </div>

            <div class="col-3">
              <div class="q-col-gutter-md dol items-start">
                <div class="col-6">
                  <q-img
                    fit="scale-down"
                    style="height: 170px; max-width: 300px"
                    src="result-screenshot.png"
                  >
                    <div class="absolute-bottom-right text-body2 text-center">
                      Specify constraints
                    </div>
                  </q-img>
                </div>
                <div class="col-1"></div>
                <div class="col-6">
                  <q-img
                    fit="scale-down"
                    style="height: 170px; max-width: 300px"
                    src="selected-screenshot.png"
                  >
                    <div class="absolute-bottom-right text-body2 text-center">
                      Show selected filters
                    </div>
                  </q-img>
                </div>
              </div>
              <div class="col-1"></div>
            </div>
          </div>
        </div>

        <p class="q-pa-lg q-mb-none text-body1 text-left">
          Users may also search models based on keywords.
        </p>
        <div class="q-pa-md">
          <div class="q-col-gutter-md row items-start">
            <div class="col-4"></div>
            <div class="col-6">
              <q-img
                fit="scale-down"
                style="height: 100px; max-width: 300px"
                src="search-screenshot.png"
              >
                <div class="absolute-bottom-right text-body2 text-center">
                  Select from options
                </div>
              </q-img>
            </div>
            <div class="col-3"></div>
          </div>
        </div>
      </div>
    </div>
    <div class="row" v-if="this.docFlag && this.results.length == 0">
      <div class="col"></div>
      <div class="col-10">
        <p class="q-pa-lg q-mb-none text-h5 text-left">Filtering Conditions</p>
        <p class="q-pa-lg q-mb-none text-body1 text-left">
          We provide the list of conditions for users to explore. We include the
          categories of each conditions so that users are aware of the scope of
          the search. For now, the categories are extracted from HuggingFace. We
          will include more metadata in the future.
        </p>
        <q-list padding>
          <q-item v-for="attr in attrList" :key="attr.label">
            <q-item-section avatar>
              <q-avatar icon="square" :text-color="getColor(attr.label)" />
            </q-item-section>
            <q-expansion-item
              :label="attr[`label`]"
              :caption="attr[`type`]"
              style="overflow-x: scroll; width: 100%"
              expand-separator
            >
              <q-markup-table v-if="attr.type == 'categorical'">
                <thead>
                  <tr v-for="category in attr.categories" :key="category.label">
                    <td class="text-left">{{ category.label }}</td>
                  </tr>
                </thead>
              </q-markup-table>

              <q-markup-table v-if="attr.type == 'metric'">
                <thead>
                  <tr v-for="metric in attr.metrics" :key="metric.label">
                    <td class="text-left">{{ metric.label }}</td>
                  </tr>
                </thead>
              </q-markup-table>
            </q-expansion-item>
          </q-item>
        </q-list>
      </div>
      <div class="col"></div>
    </div>

    <q-list bordered separator>
      <q-item v-for="obj in this.results" :key="obj">
        <q-item-section>
          <q-expansion-item
            @show="tabs[obj[`Model Name`]] = `Main`"
            :label="obj[`Model Name`]"
            :caption="obj[`Task`]"
            style="overflow-x: scroll; width: 100%"
            expand-separator
          >
            <q-tabs v-model="tabs[obj[`Model Name`]]" stretch>
              <q-tab name="Main" label="Main" />
              <q-tab name="Datasets" label="Datasets">
                <q-badge
                  v-if="obj[`Training Data`].length > 0"
                  color="primary"
                  floating
                  >{{ obj[`Training Data`].length }}</q-badge
                >
              </q-tab>
              <q-tab name="Metrics" label="Metrics">
                <q-badge
                  v-if="Object.keys(obj[`Evaluation Results`]).length > 0"
                  color="primary"
                  floating
                  >{{ Object.keys(obj[`Evaluation Results`]).length }}</q-badge
                >
              </q-tab>
              <q-tab name="Hyperparameters" label="Hyperparameters">
                <q-badge
                  v-if="Object.keys(obj[`Hyperparameters`]).length > 0"
                  color="primary"
                  floating
                  >{{ Object.keys(obj[`Hyperparameters`]).length }}</q-badge
                >
              </q-tab>
            </q-tabs>

            <q-tab-panels v-model="tabs[obj[`Model Name`]]" animated>
              <q-tab-panel name="Main">
                <q-markup-table>
                  <thead>
                    <tr>
                      <th class="text-left">Property</th>
                      <th class="text-left">Value</th>
                    </tr>
                    <tr
                      v-for="k in [`Model Name`, `Type`, `Task`, `Url`]"
                      :key="k"
                    >
                      <td class="text-left">{{ k }}</td>
                      <td class="text-left">{{ obj[k] }}</td>
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
                      <th class="text-left">Value</th>
                    </tr>
                    <tr v-for="k in obj[`Evaluation Results`]" :key="k">
                      <td class="text-left">{{ k[`metric`] }}</td>
                      <td class="text-left">
                        {{ k[`value`] }}
                      </td>
                    </tr>
                  </thead>
                </q-markup-table>
              </q-tab-panel>
              <q-tab-panel name="Hyperparameters">
                <q-markup-table style="max-width: 100%">
                  <thead>
                    <tr>
                      <th class="text-left">Parameter</th>
                      <th class="text-left">Value</th>
                    </tr>
                    <tr v-for="k in obj[`Hyperparameters`]" :key="k">
                      <td class="text-left">{{ k[`metric`] }}</td>
                      <td class="text-left">
                        {{ k[`value`] }}
                      </td>
                    </tr>
                  </thead>
                </q-markup-table>
              </q-tab-panel>
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
    flag: Boolean,
    attrList: null,
  },
  data() {
    return {
      results: [],
      query: "",
      tabs: {},
      docFlag: this.flag,
    };
  },
  watch: {
    flag: {
      immediate: true,
      handler(val, oldVal) {
        // do your stuff
        this.docFlag = val;
      },
    },
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
    applyMetricFilter(queryObject, attributeName) {
      var filteredFilters = this.filters
        .filter((f) => f.attribute == attributeName)
        .map((f) => {
          return { name: f.keyword, value: f.value, sign: f.sign };
        });
      if (filteredFilters.length > 0) {
        queryObject.where(function (obj) {
          return filteredFilters.every((p) => {
            return obj[attributeName].some((c) => {
              var bl = c.metric.toLowerCase().includes(p.name);
              switch (p.sign) {
                case ">":
                  return bl && c.value > p.value;
                case "=":
                  return bl && c.value == p.value;
                case "<":
                  return bl && c.value < p.value;
                case "~":
                  return (
                    bl && c.value > 0.9 * p.value && c.value < 1.1 * p.value
                  );
                case "⊇":
                  return (
                    bl &&
                    String(c.value).toLowerCase().includes(String(p.value))
                  );
              }
              return false;
            });
          });
        });
      }
    },
    getColor(attr) {
      var color;
      this.attrList.forEach((element) => {
        if (element.label == attr) {
          color = element.color;
        }
      });
      return color;
    },
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
      this.applyMetricFilter(res, "Evaluation Results");

      // Hyperparameters
      this.applyMetricFilter(res, "Hyperparameters");

      this.results = res.data();
    },
  },
});
</script>

<template>
  <q-layout view="hHh Lpr fFf">
    <q-header reveal elevated>
      <q-toolbar>
        <q-btn flat @click="leftDrawerOpen = !leftDrawerOpen" round dense icon="menu" />
        <q-toolbar-title> ML Model Search </q-toolbar-title>
        <q-space />
        <div class="cursor-pointer non-selectable">
          <q-btn flat clickable icon="home" @click="showHome"/>
          <q-btn flat clickable label="About" @click="showDocumentation"/>
        </div>
        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-footer bordered class="bg-white text-primary">
        <q-toolbar>
          <q-toolbar-title></q-toolbar-title>
          <q-img
                fit="scale-down"
                style="height: 50px; max-width: 100px"
                src="TU_P1_full-color.png"
          />
      </q-toolbar>
    </q-footer>


    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      class="bg-white"
      :width="270"
      :mini-width="100"
      :breakpoint="500"
    >
      
      <q-list padding>
        <p class="q-pa-lg q-mb-none text-h6 text-uppercase">Search Filters</p>
        <q-item
          v-for="(val, index) in attributes"
          :key="val.label"
          dense
          clickable
          v-ripple
        >
          <q-expansion-item
            ref="expanders"
            icon="square"
            :id="val"
            @show="
              this.closeExpanders(index, $event);
              attributeModelSigns[val.label] = `>`;
            "
            :expand-icon-class="getColor(val.label)"
            v-model="attributeExpanderStates[val.label]"
            style="width: 100%"
            dense
            :label="val.label"
          >
            <template v-slot:header>
              <q-item-section avatar>
                <q-avatar icon="square" :text-color="getColor(val.label)" />
              </q-item-section>

              <q-item-section> {{ val.label }} </q-item-section>
            </template>

            <q-card v-if="val.type == 'string'">
              <q-card-section>
                <q-form
                  @submit="
                    this.closeExpanders();
                    this.submitFilter(val.label);
                  "
                >
                  <q-input
                    filled
                    v-model="attributeModels[val.label]"
                    label="Name"
                  />
                </q-form>
              </q-card-section>
            </q-card>

            <q-card v-if="val.type == 'categorical'">
              <q-card-section>
                <q-form ref="form">
                  <q-select
                    filled
                    @update:model-value="
                      this.closeExpanders();
                      this.submitFilter(val.label);
                    "
                    v-model="attributeModels[val.label]"
                    :options="val.categories"
                    label="Type"
                  />
                </q-form>
              </q-card-section>
            </q-card>

            <q-card v-if="val.type == 'metric'">
              <q-card-section>
                <q-form
                  @submit="
                    this.closeExpanders();
                    this.submitFilter(val.label);
                  "
                >
                  <q-select
                    filled
                    use-input
                    fill-input
                    input-debounce="0"
                    hide-selected
                    @input-value="(inputVal) => setModel(val.label, inputVal)"
                    v-model="attributeModels[val.label]"
                    :options="val.metrics"
                    label="Metric"
                  />
                  <div
                    style="display: table"
                    v-if="attributeModels[val.label]?.type == `number`"
                  >
                    <q-select
                      filled
                      style="width: 25%; display: table-cell"
                      v-model="attributeModelSigns[val.label]"
                      :options="[`>`, `=`, `<`, `~`, `⊇`]"
                      label="Sign"
                    />
                    <q-input
                      filled
                      style="width: 50%; display: table-cell"
                      v-model="attributeModelValues[val.label]"
                      label="Value"
                    />
                  </div>
                  <q-input
                    v-else
                    filled
                    v-model="attributeModelValues[val.label]"
                    label="Contains"
                  />
                  <q-btn v-show="false" label="Submit" type="submit"></q-btn>
                </q-form>
              </q-card-section>
            </q-card>
          </q-expansion-item>
        </q-item>
      </q-list>
      <q-separator></q-separator>
      <p class="q-pa-lg q-mb-none text-h6 text-uppercase">Selected Filters</p>
      <q-chip
        removable
        v-for="val in this.activeFilters"
        :key="val.keyword"
        :color="getColor(val.attribute)"
        @remove="this.removeFilter(val)"
        square
      >
        {{ val.keyword }}
        <q-badge v-if="val.value" color="grey"
          >{{ val.sign }} {{ val.value }}
        </q-badge>
      </q-chip>
    </q-drawer>

    <q-page-container>
      <IndexPage
        ref="results"
        :filters="this.activeFilters"
        :models="this.models"
        :flag="this.flag"
        :attrList="attrList"
      />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import IndexPage from "pages/IndexPage.vue";
import loki from "lokijs";
import { useQuasar } from "quasar";
import { nextTick } from "vue";
const attrList = [
  {
    color: "teal-3",
    label: "Type",
    type: "categorical",
    categories: [
      { label: "Audio" },
      { label: "Computer Vision" },
      { label: "Multimodal" },
      { label: "Natural Language Processing" },
      { label: "Reinforcement Learning" },
      { label: "Structured" },
    ],
  },
  {
    color: "lime-3",
    label: "Task",
    type: "categorical",
    categories: [
      { label: "Audio Classification" },
      { label: "Audio To Audio" },
      { label: "Automatic Speech Recognition" },
      { label: "Conversational" },
      { label: "Feature Extraction" },
      { label: "Fill Mask" },
      { label: "Image Classification" },
      { label: "Image Segmentation" },
      { label: "Image To Image" },
      { label: "Image To Text" },
      { label: "Object Detection" },
      { label: "Question Answering" },
      { label: "Reinforcement Learning" },
      { label: "Sentence Similarity" },
      { label: "Summarization" },
      { label: "Table Question Answering" },
      { label: "Tabular Classification" },
      { label: "Text Classification" },
      { label: "Text Generation" },
      { label: "Text To Image" },
      { label: "Text To Speech" },
      { label: "Text2text Generation" },
      { label: "Token Classification" },
      { label: "Translation" },
      { label: "Unconditional Image Generation" },
      { label: "Voice Activity Detection" },
      { label: "Zero Shot Classification" },
      { label: "Zero Shot Image Classification" },
    ],
  },
  {
    color: "purple-3",
    label: "Training Data",
    type: "string",
  },
  {
    color: "brown-3",
    label: "Hyperparameters",
    type: "metric",
    metrics: [
      { label: "learning_rate", type: "number" },
      { label: "batch_size", type: "number" },
      { label: "seed", type: "number" },
      { label: "total_train_batch_size", type: "number" },
      { label: "optimizer", type: "string" },
      { label: "num_epochs", type: "number" },
    ],
  },
  {
    color: "indigo-3",
    label: "Evaluation Results",
    type: "metric",
    metrics: [
      { label: "accuracy", type: "number" },
      { label: "bertscore", type: "number" },
      { label: "bleu", type: "number" },
      { label: "bleurt", type: "number" },
      { label: "cer", type: "number" },
      { label: "chrf", type: "number" },
      { label: "code_eval", type: "number" },
      { label: "comet", type: "number" },
      { label: "competition_math", type: "number" },
      { label: "coval", type: "number" },
      { label: "cuad", type: "number" },
      { label: "exact_match", type: "number" },
      { label: "f1", type: "number" },
      { label: "frugalscore", type: "number" },
      { label: "glue", type: "number" },
      { label: "google_bleu", type: "number" },
      { label: "indic_glue", type: "number" },
      { label: "mae", type: "number" },
      { label: "mahalanobis", type: "number" },
      { label: "matthews_correlation", type: "number" },
      { label: "mauve", type: "number" },
      { label: "mean_iou", type: "number" },
      { label: "meteor", type: "number" },
      { label: "mse", type: "number" },
      { label: "pearsonr", type: "number" },
      { label: "perplexity", type: "number" },
      { label: "precision", type: "number" },
      { label: "recall", type: "number" },
      { label: "roc_auc", type: "number" },
      { label: "rouge", type: "number" },
      { label: "sacrebleu", type: "number" },
      { label: "sari", type: "number" },
      { label: "seqeval", type: "number" },
      { label: "spearmanr", type: "number" },
      { label: "squad", type: "number" },
      { label: "squad_v2", type: "number" },
      { label: "super_glue", type: "number" },
      { label: "ter", type: "number" },
      { label: "wer", type: "number" },
      { label: "wiki_split", type: "number" },
      { label: "xnli", type: "number" },
      { label: "xtreme_s", type: "number" },
    ],
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {
    IndexPage,
  },
  data() {
    return {
      attributeModels: Object.assign(
        {},
        ...attrList.map((x) => {
          return { [x.label]: null };
        })
      ),
      attributeModelValues: Object.assign(
        {},
        ...attrList.map((x) => {
          return { [x.label]: null };
        })
      ),
      attributeModelSigns: Object.assign(
        {},
        ...attrList.map((x) => {
          return { [x.label]: null };
        })
      ),
      attributeExpanderStates: attrList.map((x) => {
        return { [x.label]: false };
      }),
      activeFilters: [],
      flag: true,
      attrList,
    };
  },
  computed: {
    console: () => console,
  },
  methods: {
    removeFilter(model) {
      this.activeFilters = this.activeFilters.filter((x) => x !== model);
    },
    showDocumentation() {
      this.flag = true;
      this.activeFilters = [];
    },
    showHome() {
      this.flag = false;
      this.activeFilters = [];
    },
    getColor(attr) {
      var color;
      this.attributes.forEach((element) => {
        if (element.label == attr) {
          color = element.color;
        }
      });
      return color;
    },
    closeExpanders(index, e) {
      this.$refs.expanders.forEach((x, i) => {
        if (i != index) {
          x.hide();
        }
      });
    },
    submitFilter(attr) {
      var context = this;
      nextTick(function () {
        if (context.attributeModels[attr]?.type == "string") {
          context.attributeModelSigns[attr] = "⊇";
        }

        context.activeFilters.push({
          attribute: attr,
          keyword:
            context.attributeModels[attr].label == null
              ? context.attributeModels[attr]
              : context.attributeModels[attr].label,
          value: context.attributeModelValues[attr],
          sign: context.attributeModelSigns[attr],
        });
        context.attributeModels[attr] = null;
        context.attributeModelValues[attr] = null;
      });
    },
    setModel(key, val) {
      this.attributeModels[key] = { label: val, type: "number" };
    },
  },
  mounted() {
    var context = this;
    import("../assets/dump.json").then((jsonData) => {
      this.loaded = true;
      this.models.insert(JSON.parse(JSON.stringify(jsonData))["default"]);
      context.$q.loading.hide();
    });
  },
  setup() {
    const $q = useQuasar();
    $q.loading.show({
      delay: 100, // ms
    });
    const leftDrawerOpen = ref(false);
    var db = new loki("mlhub.db");
    var models = db.addCollection("models");
    // The JSON is copied because Vue automatically turns it into a property, interfering with LokiJS

    return {
      models: models,
      original: attrList,
      attributes: attrList,
      $q,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>

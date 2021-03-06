<template>
  <div class="multi-select-wrapper">
    <template v-if="isFormInput">
      <input
        v-for="value in values"
        :key="value"
        type="hidden"
        :name="isMultiple ? name + '[]' : name"
        :value="value"
      >
    </template>
    <multiselect
      v-model="input"
      :options="options"
      :disabled="disabled"
      :track-by="trackBy"
      :label="label"
      :close-on-select="!isMultiple"
      :show-labels="isMultiple"
      :multiple="isMultiple"
      :loading="loading"
      :taggable="isTaggable"
      :tag-position="tagPosition"
      :placeholder="placeholder"
      @tag="create"
      @search-change="search"
      @input="$emit('input', input)"
    />
  </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";
import _ from "underscore";

export default {
  name: "VSelect",
  components: {
    multiselect: Multiselect,
  },
  props: {
    searchUrl: {
      type: String,
      required: true,
    },
    name: {
      type: String,
    },
    value: {
      default: null,
      required: false,
    },
    placeholder: {
      type: String,
      default: null,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    trackBy: {
      type: String,
      default: "id",
    },
    tagPosition: {
      type: String,
      default: "bottom",
    },
    label: {
      type: String,
      default: "label",
    },
    getUrl: { // does this ever get used???
      type: String,
      default: null,
    },
    createUrl: {
      type: String,
      required: false,
    },

    isMultiple: {
      type: Boolean,
      default: false,
    },
    isTaggable: {
      type: Boolean,
      default: false,
    },
    isFormInput: {
      type: Boolean,
      default: false,
    },
    clearOnEventName: {
      type: String,
      required: false,
    },
  },
  data() {
    return {
      input: [],
      options: [],
      loading: false,
    };
  },
  computed: {
    values() {
      let input = this.input;
      if (!input) {
        return [];
      }
      if (!Array.isArray(input)) {
        input = [input];
      }
      return input.map(value => {
        return value[this.trackBy];
      });
    },
  },
  created() {
    this.search("", () => {
      if (this.value) {
        let values = this.value;
        if (typeof values === "string") {
          values = JSON.parse(values);
        }
        if (!Array.isArray(values)) {
          values = [values];
        }
        for (let value of values) {
          if (this.options.some(option => option === value) === false) {
            this.options.push(value);
          }
          this.input.push(value);
        }
      }
    });
    this.search = _.debounce(this.search, 200);
    if (this.clearOnEventName) {
      this.$root.$on(this.clearOnEventName, () => (this.input = null));
    }
  },
  methods: {
    async get(id) {
      if (this.getUrl && id) {
        this.loading = true;
        await axios
          .get(this.getUrl, {
            params: {
              id: id,
            },
          })
          .then(response => {
            let data = response.data.data;
            this.input.push(data);
            this.$emit("get", data);
          })
          .catch(console.error)
          .finally(() => {
            this.loading = false;
          });
      }
    },
    async create(newOption) {
      if (this.createUrl) {
        this.loading = true;
        console.log({ newOption }, this.createUrl)
        await axios
          .post(this.createUrl, {
            name: newOption,
          })
          .then(response => {
            let value = response.data;
            this.options.push(value);
            if (this.isMultiple) {
              this.input.push(value);
            } else {
              this.input = value;
            }
          })
          .catch(console.error)
          .finally(() => {
            this.$emit('input', this.input)
            this.loading = false;
          });
      }
    },
    async search(search = "", callback = null) {
      this.loading = true;
      await axios
        .get(this.searchUrl, {
          params: {
            s: search,
          },
        })
        .then(response => {
          this.options = response.data;
          if (callback) {
            callback(response);
          }
        })
        .catch(console.error)
        .finally(() => {
          this.loading = false;
        });
    },
  },
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

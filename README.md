# MultiselectWrapper 
Probably the most complete *selecting* solution for Vue.js 2.0, without jQuery.

This package is a wrapper around VueMultiSelect for our API based implementation. All select options are API generated and additional options for creating options are available as well. 

### Vue 3.0 Support
For Vue 3.0 compatible version see [`next`](https://github.com/shentao/vue-multiselect/tree/next) branch.

<a href="https://vue-multiselect.js.org/">
  <img src="/multiselect-screen-203.png" alt="Vue-Multiselect Screen">
</a>

## Documentation
Visit: [vue-multiselect.js.org](https://vue-multiselect.js.org/#sub-getting-started)

## Features & characteristics:
* Single select
* Multiple select
* Tagging
* Dropdowns
* Filtering
* Search with suggestions


## Breaking changes:
* Instead of Vue.partial for custom option templates you can use a custom render function.
* The `:key` props has changed to `:track-by`, due to conflicts with Vue 2.0.
* Support for `v-model`
* `@update` has changed to `@input` to also work with v-model
* `:selected` has changed to `:value` for the same reason
* Browserify users: if you wish to import `.vue` files, please add `vueify` transform.

## Install & basic usage

```bash
npm i @dcodegroup-au/vue-multiselect
```

### Configuration Options
* All select options are API generated and additional feature for creating options with the isTaggable props is available along with a createUrl prop.
* This component takes a required searchUrl prop which is an endpoint to a resource which returns a JSON array of objects. The optional label prop will set the option content and must corresponds to a key value from the JSON array items return from the searchUrl.

```//example.vue.js
<multiselect-wrapper
  :\value="form.severity"
  track-by="id"
  label="label"
  search-url="/meta/endpoint"
  :\is-multiple="false"
  :\is-taggable="false"
  @\input="val => form.severity = val" // required for receiving updated input values
/>
```

```//example.blade.php
<multiselect-wrapper
  track-by="id"
  label="label"
  search-url="/meta/endpoint"
  :\is-multiple="true"
  :\is-taggable="false"
  :\is-form-input="true" // required for form inputs
  name="severity" // required for form inputs
></multiselect-wrapper >```

## The complete wrapper component

```<template>
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
    getUrl: {
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
      this.eventBus.$on(this.clearOnEventName, () => (this.input = null));
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

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>```
<!--
### Single select / dropdown
``` jade
multiselect(
  :value="value",
  :options="source",
  :searchable="false",
  :close-on-select="false",
  :allow-empty="false",
  @input="updateSelected",
  label="name",
  placeholder="Select one",
  track-by="name"
)
```
### Single select with search
``` jade
multiselect(
  v-model="value",
  :options="source",
  :close-on-select="true",
  :clear-on-select="false",
  placeholder="Select one",
  label="name",
  track-by="name"
)
```
### Multiple select with search
``` jade
multiselect(
  v-model="multiValue",
  :options="source",
  :multiple="true",
  :close-on-select="true",
  placeholder="Pick some",
  label="name",
  track-by="name"
)
```
### Tagging
with `@tag` event
``` jade
multiselect(
  v-model="taggingSelected",
  :options="taggingOptions",
  :multiple="true",
  :taggable="true",
  @tag="addTag",
  tag-placeholder="Add this as new tag",
  placeholder="Type to search or add tag",
  label="name",
  track-by="code"
)
```
### Asynchronous dropdown
``` jade
multiselect(
  v-model="selectedCountries",
  :options="countries",
  :multiple="multiple",
  :searchable="searchable",
  @search-change="asyncFind",
  placeholder="Type to search",
  label="name"
  track-by="code"
)
  span(slot="noResult").
    Oops! No elements found. Consider changing the search query.
```
``` javascript
methods: {
  asyncFind (query) {
    this.countries = findService(query)
  }
}
```-->

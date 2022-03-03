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
<v-multi-select
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
<v-multi-select
track-by="id"
label="label"
search-url="/meta/endpoint"
:\is-multiple="true"
:\is-taggable="false"
:\is-form-input="true"
name="severity" // required for form inputs
></v-multi-select>```

## The complete wrapper component
```vue
<template>
    <div class="multi-select-wrapper">
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
    ...
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
```
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
```
 -->

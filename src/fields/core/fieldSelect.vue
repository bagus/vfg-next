<template>
  <select
    :id="getFieldID(schema)"
    v-model="value"
    v-attributes="'input'"
    class="form-control"
    :disabled="disabled"
    :name="schema.inputName"
    :class="schema.fieldClasses"
  >
    <option
      v-if="!selectOptions.hideNoneSelectedText"
      :disabled="schema.required"
      :value="null"
    >
      <template v-if="selectOptions.noneSelectedText">
        {{
          formOptions.i18n && $te(selectOptions.noneSelectedText)
            ? $t(selectOptions.noneSelectedText)
            : selectOptions.noneSelectedText
        }}
      </template>
      <template v-else> &lt;Nothing selected&gt; </template>
    </option>
    <template v-for="(item, index) in items">
      <optgroup
        v-if="item.group"
        :key="index + '_optgroup'"
        :label="getGroupName(item)"
      >
        <template v-if="item.ops">
          <option v-for="i in item.ops" :key="i" :value="getItemValue(i)">
            <template v-if="formOptions.i18n && $te(getItemName(i))">
              {{ $t(getItemName(i)) }}
            </template>
            <template v-else>
              {{ getItemName(i) }}
            </template>
          </option>
        </template>
      </optgroup>
      <option
        v-else-if="!item.group"
        :key="index + '_option'"
        :value="getItemValue(item)"
      >
        <template v-if="formOptions.i18n && $te(getItemName(item))">
          {{ $t(getItemName(item)) }}
        </template>
        <template v-else>
          {{ getItemName(item) }}
        </template>
        {{}}
      </option>
    </template>
  </select>
</template>

<script>
import { isObject, isNil, find } from "lodash";
import abstractField from "../abstractField";

import { defineComponent } from "vue";

export default defineComponent({
  mixins: [abstractField],

  computed: {
    selectOptions() {
      return this.schema.selectOptions || {};
    },

    items() {
      let values = this.schema.values;
      if (typeof values == "function") {
        return this.groupValues(values.apply(this, [this.model, this.schema]));
      } else return this.groupValues(values);
    },
  },

  methods: {
    formatValueToField(value) {
      if (isNil(value)) {
        return null;
      }
      return value;
    },

    groupValues(values) {
      let array = [];
      let arrayElement = {};

      values.forEach((item) => {
        arrayElement = null;

        if (item.group && isObject(item)) {
          // There is in a group.

          // Find element with this group.
          arrayElement = find(array, (i) => i.group === item.group);

          if (arrayElement) {
            // There is such a group.

            arrayElement.ops.push({
              id: item.id,
              name: item.name,
            });
          } else {
            // There is not such a group.

            // Initialising.
            arrayElement = {
              group: "",
              ops: [],
            };

            // Set group.
            arrayElement.group = item.group;

            // Set Group element.
            arrayElement.ops.push({
              id: item.id,
              name: item.name,
            });

            // Add array.
            array.push(arrayElement);
          }
        } else {
          // There is not in a group.
          array.push(item);
        }
      });

      // With Groups.
      return array;
    },

    getGroupName(item) {
      if (item && item.group) {
        return item.group;
      }

      throw "Group name is missing! https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
    },

    getItemValue(item) {
      if (isObject(item)) {
        if (
          typeof this.schema["selectOptions"] !== "undefined" &&
          typeof this.schema["selectOptions"]["value"] !== "undefined"
        ) {
          return item[this.schema.selectOptions.value];
        } else {
          // Use 'id' instead of 'value' cause of backward compatibility
          if (typeof item["id"] !== "undefined") {
            return item.id;
          } else {
            throw "`id` is not defined. If you want to use another key name, add a `value` property under `selectOptions` in the schema. https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
          }
        }
      } else {
        return item;
      }
    },

    getItemName(item) {
      if (isObject(item)) {
        if (
          typeof this.schema["selectOptions"] !== "undefined" &&
          typeof this.schema["selectOptions"]["name"] !== "undefined"
        ) {
          return item[this.schema.selectOptions.name];
        } else {
          if (typeof item["name"] !== "undefined") {
            return item.name;
          } else {
            throw "`name` is not defined. If you want to use another key name, add a `name` property under `selectOptions` in the schema. https://icebob.gitbooks.io/vueformgenerator/content/fields/select.html#select-field-with-object-items";
          }
        }
      } else {
        return item;
      }
    },
  },
});
</script>

<style lang="sass"></style>

<template>
  <input
    :id="getFieldID(schema)"
    type="text"
    :autocomplete="schema.autocomplete"
    :disabled="disabled"
    :placeholder="schema.placeholder"
    :readonly="schema.readonly"
    :name="schema.inputName"
  />
</template>

<script>
/* global $ */
import abstractField from "../abstractField";
import { defaults } from "lodash";
import { defineComponent } from "vue";

export default defineComponent({
  mixins: [abstractField],

  data() {
    return {
      picker: null,
    };
  },

  watch: {
    model() {
      if (window.$ && window.$.fn.spectrum) {
        this.picker.spectrum("set", this.value);
      }
    },

    disabled(val) {
      if (val) this.picker.spectrum("disable");
      else this.picker.spectrum("enable");
    },
  },

  mounted() {
    this.$nextTick(function () {
      if (window.$ && window.$.fn.spectrum) {
        this.picker = $(this.$el)
          .spectrum("destroy")
          .spectrum(
            defaults(this.schema.colorOptions || {}, {
              showInput: true,
              showAlpha: true,
              disabled: this.schema.disabled,
              allowEmpty: !this.schema.required,
              preferredFormat: "hex",
              change: (color) => {
                this.value = color ? color.toString() : null;
              },
            })
          );
        this.picker.spectrum("set", this.value);
      } else {
        console.warn(
          "Spectrum color library is missing. Please download from http://bgrins.github.io/spectrum/ and load the script and CSS in the HTML head section!"
        );
      }
    });
  },

  beforeUnmount() {
    if (this.picker) this.picker.spectrum("destroy");
  },
});
</script>

<style lang="scss"></style>

<template>
  <default-field :field="field" :errors="errors" :show-help-text="showHelpText" class="simple-repeatable">
    <template slot="field">
      <div class="flex flex-col">
        <!-- Title columns -->
        <div class="simple-repeatable-header-row flex border-b border-40 py-2">
          <div v-for="(repField, i) in field.repeatableFields" :key="i" class="font-bold text-90 text-md w-full mr-2">
            {{ repField.name }}
          </div>
        </div>

        <draggable v-model="fieldsWithValues" :options="{ handle: '.vue-draggable-handle' }">
          <div
            v-for="fields in fieldsWithValues"
            :key="fields[0].attribute"
            class="simple-repeatable-row flex py-2 pl-2 relative"
          >
            <div class="vue-draggable-handle absolute flex justify-center items-center cursor-pointer">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="22" height="22" class="fill-current">
                <path
                  d="M4 5h16a1 1 0 0 1 0 2H4a1 1 0 1 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2zm0 6h16a1 1 0 0 1 0 2H4a1 1 0 0 1 0-2z"
                />
              </svg>
            </div>

            <component
              v-for="(repField, i) in fields"
              :key="i"
              :is="`form-${repField.component}`"
              :field="repField"
              class="mr-2"
            />
          </div>
        </draggable>

        <button @click="addRow" class="btn btn-default btn-primary" type="button">Add row</button>
      </div>
    </template>
  </default-field>
</template>

<script>
import Draggable from 'vuedraggable';
import { FormField, HandlesValidationErrors } from 'laravel-nova';

let UNIQUE_ID_INDEX = 0;

export default {
  mixins: [FormField, HandlesValidationErrors],

  components: { Draggable },

  props: ['resourceName', 'resourceId', 'field'],

  data() {
    return {
      fieldsWithValues: [],
    };
  },

  methods: {
    setInitialValue() {
      let value = [];
      try {
        value = JSON.parse(this.field.value) || [];
      } catch (e) {
        value = [];
      }

      this.fieldsWithValues = value.map(this.copyFields);

      console.info(this.fieldsWithValues);
    },

    copyFields(value) {
      return this.field.repeatableFields.map(field => ({
        ...field,
        attribute: `${field.attribute}---${UNIQUE_ID_INDEX++}`,
        value: value && value[field.attribute],
      }));
    },

    fill(formData) {
      const allValues = [];

      for (const fields of this.fieldsWithValues) {
        const rowValues = {};

        for (const field of fields) {
          const formData = new FormData();
          field.fill(formData);

          const normalizedAttribute = field.attribute.replace(/---\d+/, '');
          rowValues[normalizedAttribute] = formData.get(field.attribute);
        }

        allValues.push(rowValues);
      }

      formData.append(this.field.attribute, JSON.stringify(allValues));
    },

    addRow() {
      this.fieldsWithValues.push(this.copyFields());
    },
  },

  computed: {
    defaultAttributes() {
      return {
        type: 'number',
        min: this.field.min,
        max: this.field.max,
        step: this.field.step,
        pattern: this.field.pattern,
        placeholder: this.field.placeholder || this.field.name,
        class: this.errorClasses,
      };
    },

    extraAttributes() {
      return {
        ...this.defaultAttributes,
        ...this.field.extraAttributes,
      };
    },
  },
};
</script>

<style lang="scss" scoped>
.simple-repeatable {
  .simple-repeatable-row {
    // Select field
    > * {
      width: 100%;
      border: none !important;

      // Hide name
      > :not(svg):nth-child(1) {
        display: none;
      }

      // Fix field width and padding
      > :nth-child(2) {
        width: 100% !important;
        padding: 0 !important;
      }
    }

    .vue-draggable-handle {
      height: 36px;
      width: 36px;
      left: -36px;

      &:hover {
        opacity: 0.8;
      }
    }
  }

  > :nth-child(1) {
    min-width: 20%;
  }

  // Make field area full width
  > :nth-child(2) {
    width: 100% !important;
  }
}
</style>
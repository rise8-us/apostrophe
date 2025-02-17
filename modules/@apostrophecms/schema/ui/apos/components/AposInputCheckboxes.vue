<template>
  <AposInputWrapper
    :field="field" :error="effectiveError"
    :uid="uid"
    :modifiers="modifiers"
    :display-options="displayOptions"
  >
    <template #body>
      <AposCombo
        v-if="field.style === 'combo' && choices.length"
        :choices="choices"
        :field="field"
        :value="value"
        @select-items="selectItems"
      />

      <AposCheckbox
        v-else
        :for="getChoiceId(uid, choice.value)"
        v-for="choice in choices"
        :key="choice.value"
        :id="getChoiceId(uid, choice.value)"
        :choice="choice"
        :field="field"
        v-model="value.data"
      />
    </template>
  </AposInputWrapper>
</template>

<script>
import AposInputMixin from 'Modules/@apostrophecms/schema/mixins/AposInputMixin';
import AposInputChoicesMixin from 'Modules/@apostrophecms/schema/mixins/AposInputChoicesMixin';

export default {
  name: 'AposInputCheckboxes',
  mixins: [ AposInputMixin, AposInputChoicesMixin ],
  beforeMount () {
    this.value.data = Array.isArray(this.value.data) ? this.value.data : [];
  },
  methods: {
    getChoiceId(uid, value) {
      return uid + value.replace(/\s/g, '');
    },
    watchValue () {
      this.error = this.value.error;
      this.next = this.value.data || [];
    },
    validate(values) {
      // The choices and values should always be arrays.
      if (!Array.isArray(this.choices) || !Array.isArray(values)) {
        return 'malformed';
      }

      if (this.field.required && !values.length) {
        return 'required';
      }

      if (this.field.min) {
        if ((values != null) && (values.length < this.field.min)) {
          return this.$t('apostrophe:minUi', { number: this.field.min });
        }
      }
      if (this.field.max) {
        if ((values != null) && (values.length > this.field.max)) {
          return this.$t('apostrophe:maxUi', { number: this.field.max });
        }
      }

      if (Array.isArray(values)) {
        values.forEach(chosen => {
          if (!this.choices.map(choice => {
            return choice.value;
          }).includes(chosen)) {
            return 'invalid';
          }
        });
      }

      return false;
    },
    selectItems(choice) {
      if (choice.value === '__all') {
        this.value.data = this.choices.length === this.value.data.length
          ? []
          : this.choices.map(({ value }) => value);

        return;
      }

      if (this.value.data.includes(choice.value)) {
        this.value.data = this.value.data.filter((val) => val !== choice.value);
      } else {
        this.value.data.push(choice.value);
      }
    }
  }
};
</script>

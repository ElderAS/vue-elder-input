<template>
  <div class="elder-input">
    <label :for="id" v-if="label" class="elder-input__label">
      {{ label }}
      <span v-if="isRequired" class="elder-input__label-required">*</span>
      <slot name="after-label"></slot>
    </label>
    <span v-if="hasSublabel" class="elder-input__sublabel">
      <slot name="sublabel">{{ sublabel }}</slot>
    </span>
    <div class="elder-input__wrapper">
      <slot name="left"></slot>
      <div
        class="elder-input__field"
        :class="{
          'elder-input__field--icon': icon,
          'elder-input__field--prefixed': hasPrefix,
          'elder-input__field--suffixed': hasSuffix,
          'elder-input__field--focus': focused,
          'elder-input__field--disabled': isDisabled,
          'elder-input__field--readonly': isReadonly,
          'elder-input__field--valid': hasValidation && validComp,
          'elder-input__field--invalid': hasValidation && !validComp,
        }"
      >
        <label :for="id" v-if="hasPrefix" class="elder-input__prefix">
          <slot name="prefix">{{ prefix }}</slot>
        </label>
        <label v-if="hasIcon" :for="id" class="elder-input__icon">
          <slot name="icon">
            <font-awesome-icon v-if="icon" :icon="icon" />
          </slot>
        </label>
        <div class="elder-input__value">
          <slot :onBlur="onBlur" :onFocus="onFocus">
            <component
              :is="component"
              v-on="{ ...$listeners }"
              v-bind="{ ...$attrs, ...maskComp, type, id }"
              v-model="valueComp"
              class="elder-input__element"
              :class="['elder-input--alignment-' + align]"
              ref="input"
              @focus="onFocus"
              @blur="onBlur"
            />
          </slot>
        </div>
        <label v-if="hasValidation" :for="id" class="elder-input__validation">
          <Icon :icon="['fas', validComp ? 'check-circle' : 'times-circle']" />
        </label>
        <label :for="id" v-if="hasSuffix" class="elder-input__suffix">
          <slot name="suffix">{{ suffix }}</slot>
        </label>
      </div>
      <slot name="right"></slot>
    </div>
    <div v-if="hasValidation && hasValidationMessage && !validComp" class="elder-input__validation-message">
      <slot name="validation-message">{{ validationMessage }}</slot>
    </div>
    <slot name="below"></slot>
  </div>
</template>

<script>
import Icon from '@kvass/vue2-icon'
import { IMaskComponent } from 'vue-imask'
import { AttributeBoolean } from './utils'
import InputComponent from './Input'

export default {
  props: {
    value: [String, Number, Date],
    label: String,
    type: {
      type: String,
      default: 'text',
    },
    mask: {
      type: Object,
      default: () => ({}),
    },
    isValid: Boolean,
    validate: [Boolean, Function],
    validateImmediate: Boolean,
    validationMessage: String,
    prefix: String,
    suffix: String,
    icon: [Array, String],
    align: {
      type: String,
      default: 'left',
      enum: ['left', 'right', 'center'],
    },
    sublabel: String,
  },
  watch: {
    value: {
      handler(val) {
        this.$nextTick(() => this.validateValue())
        if ((val && this.validateImmediate) || val != this.lastInternalUpdate) this.visited = true
      },
      immediate: true,
    },
  },
  computed: {
    valueComp: {
      get() {
        if ([null, undefined].includes(this.value)) return ''
        return this.value.toString()
      },
      set(val) {
        if (this.type === 'number' || this.maskComp.mask === Number) val = parseFloat(val)
        this.lastInternalUpdate = val
        this.$emit('input', val)
      },
    },
    validComp() {
      return this.hasIsValidProp ? this.isValid : this.valid
    },
    hasIsValidProp() {
      return 'isValid' in this.$options.propsData
    },
    component() {
      if (this.hasMask) return IMaskComponent
      return InputComponent
    },
    hasMask() {
      return this.maskComp.mask
    },
    hasPrefix() {
      return this.prefix || this.$slots.prefix
    },
    hasSuffix() {
      return this.suffix || this.$slots.suffix
    },
    hasValidationMessage() {
      return this.validationMessage || this.$slots['validation-message']
    },
    hasIcon() {
      return this.icon || this.$slots.icon
    },
    hasValidation() {
      if (!this.isRequired && !this.value) return false
      return this.visited && (this.validate || this.hasIsValidProp)
    },
    hasSublabel() {
      return this.sublabel || this.$slots.sublabel
    },
    maskComp() {
      if (!Object.keys(this.mask).length) return this.mask

      return {
        ...this.mask,
        mask: this.mask.mask === 'Number' ? Number : this.mask.mask,
      }
    },
    isDisabled: AttributeBoolean('disabled'),
    isRequired: AttributeBoolean('required'),
    isReadonly: AttributeBoolean('readonly'),
  },
  data() {
    return {
      id: null,
      focused: false,
      visited: false,
      valid: false,
      lastInternalUpdate: undefined,
    }
  },
  methods: {
    validateValue() {
      if (!this.validate) return

      if (typeof this.validate === 'function') return (this.valid = this.validate(this.value))

      if (this.$refs.input) this.valid = (this.$refs.input.$el || this.$refs.input).checkValidity()
    },
    onFocus() {
      this.focused = true
    },
    onBlur() {
      this.visited = true
      this.focused = false
    },
  },
  created() {
    this.id = this._uid
  },
  components: {
    Icon,
    IMaskComponent,
  },
}
</script>

<style lang="scss">
$variables: (
  'primary': #3a9acd,
  'success': #33ca62,
  'error': #e83b35,
  'border-radius': 3px,
  'border-color': #eaeaea,
  'text-color': #222,
  'input-color': #f2f2f2,
  'input-prefix-color': rgba(black, 0.3),
);

@function GetVariable($key) {
  @return var(--vue-elder-#{$key}, map-get($variables, $key));
}

.elder-input {
  $component: 'elder-input';
  $spacing: 1.1em;

  display: flex;
  flex-direction: column;

  text-align: left;

  color: GetVariable('text-color');

  &__label {
    font-weight: bold;

    display: flex;
    gap: $spacing / 2;

    &-required {
      color: GetVariable('error');
    }
  }

  &__sublabel {
    font-size: 0.9em;
    display: block;

    opacity: 0.5;
  }

  &__wrapper {
    display: flex;
    flex-grow: 1;

    &:not(:first-child) {
      margin-top: 0.5em;
    }
  }

  &__field {
    position: relative;

    display: flex;
    flex-grow: 1;

    border: 1px solid GetVariable('border-color');
    border-radius: GetVariable('border-radius');
    background-color: white;

    &:not(:first-child) {
      margin-left: 0.5em;
    }

    &:not(:last-child) {
      margin-right: 0.5em;
    }

    &--focus {
      border-color: GetVariable('primary');
    }

    &--readonly .#{$component}-value {
      color: GetVariable('input-prefix-color');
    }

    &--disabled {
      position: relative;

      &:before {
        position: absolute;
        top: 0;
        left: 0;

        width: 100%;
        height: 100%;

        content: '';

        opacity: 0.4;
        background-color: GetVariable('input-color');
      }
    }

    &.#{$component}__field--invalid {
      border-color: GetVariable('error');
    }
  }

  &--alignment-left {
    text-align: left;
  }
  &--alignment-right {
    text-align: right;
  }
  &--alignment-center {
    text-align: center;
  }

  &__suffix,
  &__prefix {
    padding: $spacing;

    color: GetVariable('input-prefix-color');
    background-color: GetVariable('input-color');
  }

  &__suffix {
    flex-shrink: 0;

    border-left: 1px solid GetVariable('border-color');
  }

  &__prefix {
    flex-shrink: 0;

    border-right: 1px solid GetVariable('border-color');
  }

  &__validation {
    display: flex;
    align-items: center;

    padding-right: $spacing;

    .#{$component}__field--invalid & {
      color: GetVariable('error');
    }

    .#{$component}__field--valid & {
      color: GetVariable('success');
    }
  }

  &__validation-message {
    font-size: 0.8em;

    margin-top: 0.5em;

    color: GetVariable('error');
  }

  &__icon {
    display: flex;
    align-items: center;
    justify-content: center;

    padding-left: $spacing;
  }

  &__value {
    display: flex;
    flex-grow: 1;

    & > * {
      flex-grow: 1;
    }
  }

  &__element {
    font: inherit;

    width: 100%;
    padding: $spacing;

    color: inherit;
    border: none;
    outline: none;
    background-color: transparent;

    -webkit-appearance: none;

    &::-webkit-input-placeholder {
      color: GetVariable('input-prefix-color');
    }
    &::-moz-placeholder {
      color: GetVariable('input-prefix-color');
    }
    &:-ms-input-placeholder {
      color: GetVariable('input-prefix-color');
    }
    &:-moz-placeholder {
      color: GetVariable('input-prefix-color');
    }
  }
}
</style>

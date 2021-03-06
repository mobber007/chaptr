<template>
  <div :class="cssClasses">
    <textarea
      v-validate="validation"
      :data-vv-as="placeholder"
      :name="name"
      :id="id"
      :required="required"
      :value="value"
      :type="type"
      :autocomplete="type"
      :disabled="disabled"
      :readonly="readonly"
      :class="[
        value ? $style.hasValue : ''
      ]"
      v-bind="$attrs"
      v-on="handlers"
    ></textarea>
    <span :class="$style.bar"></span>
    <label :for="name">
      {{ placeholder }}<sup v-if="required">*</sup>
    </label>
    <div :class="$style.message">{{ messageOrError }}</div>
  </div>
</template>

<script lang="ts">
  import { Validator } from 'vee-validate';

  export default {
    name:     'VueTextarea',
    inject:   {
      $validator: {
        default: new Validator({}, {}),
      },
    },
    props:    {
      name:         {
        type:     String,
        required: true,
      },
      id:           {
        type:     String,
        required: true,
      },
      placeholder:  {
        type:    String,
        default: '',
      },
      required:     {
        type:    Boolean,
        default: false,
      },
      value:        {
        type:    String,
        default: '',
      },
      type:         {
        type:    String,
        default: 'text',
      },
      disabled:     {
        type:    Boolean,
        default: false,
      },
      readonly:     {
        type:    Boolean,
        default: false,
      },
      message:      {
        type:    String,
        default: '',
      },
      errorMessage: {
        type:    String,
        default: '',
      },
      validation:   {
        default: '',
      },
    },
    computed: {
      isValid() {
        return this.errors ? this.errors.first(this.name) === null : true;
      },
      messageOrError() {
        return this.isValid ? this.message : this.errorMessage;
      },
      cssClasses() {
        const classes = [this.$style.vueTextarea];

        if (this.disabled) {
          classes.push(this.$style.disabled);
        }

        if (!this.isValid) {
          classes.push(this.$style.error);
        }

        return classes;
      },
      handlers() {
        delete this.$listeners.input;

        return {
          ...this.$listeners,
          input: (e: any) => {
            this.$emit('input', e.target.value);
          },
        };
      },
    },
  };
</script>

<style lang="scss" module>
  @import "../../styles";

  .vueTextarea {
    position: relative;
    margin:   $input-margin;

    textarea,
    textarea:active,
    textarea:focus,
    textarea:hover {
      outline: none !important;
    }
    textarea {
      background-color: $input-background-color;
      border:           none;
      border-bottom:    $input-border-bottom;
      padding:          $input-padding;
      display:          block;
      width:            100%;
      font-family:      $input-font-family;
      font-size:        $input-font-size;
      color:            $input-color;
      height:           $input-height * 5;
      border-radius:    0;
    }
    textarea:focus ~ label,
    textarea.hasValue ~ label {
      top:         -($space-unit * 2.5);
      font-size:   $input-placeholder-active-font-size;
      font-weight: $input-placeholder-active-font-weight;
      color:       $input-placeholder-active-font-color;
      height:      $input-placeholder-active-height;
    }
    textarea:focus ~ .bar:before,
    textarea:focus ~ .bar:after {
      width: 50%;
    }

    label {
      color:          $input-placeholder-color;
      font-size:      $input-placeholder-font-size;
      font-weight:    $input-placeholder-font-weight;
      position:       absolute;
      pointer-events: none;
      top:            $input-placeholder-top;
      transition:     0.2s ease all;
    }
  }

  .error {
    textarea {
      border-bottom-color: $brand-warn;
    }
    textarea:focus ~ label,
    textarea.hasValue ~ label {
      color: $brand-warn;
    }

    .bar {
      &:before,
      &:after {
        background: $brand-warn;
      }
    }
  }

  .bar {
    position: relative;
    display:  block;
    width:    100%;

    &:before,
    &:after {
      content:    '';
      height:     $input-bar-height;
      width:      0;
      bottom:     0;
      position:   absolute;
      background: $input-bar-color;
      transition: all 0.2s ease-in-out;
    }
    &:before {
      left: 50%;
    }
    &:after {
      right: 50%;
    }
  }

  .message {
    display:     block;
    height:      $input-message-height;
    padding:     $input-message-padding;
    position:    relative;
    color:       $input-message-color;
    font-size:   $input-message-font-size;
    font-weight: $input-message-font-weight;
  }

  .disabled {
    opacity: .6;
  }
</style>

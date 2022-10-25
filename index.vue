<template>
  <component
    :is="tag"
    v-bind="$attrs"
    :type="type"
    class="button"
    :class="[
      `_${color}`,
      `_${size}`,
      {
        'font-h2': size === 'default',
        'font-semibold': size === 'md',
        _outline: outline,
        _squared: squared,
        _multiline: multiline,
        _disabled: $attrs.disabled,
      },
    ]"
    @click="$emit('click')"
  >
    <slot />
  </component>
</template>

<script>
export default {
  inheritAttrs: false,
  props: {
    color: {
      type: String,
      default: "green",
      validator(prop) {
        return ["green", "gray", "black"].includes(prop);
      },
    },
    size: {
      type: String,
      default: "default",
      validator(prop) {
        return ["default", "md"].includes(prop);
      },
    },
    outline: {
      type: Boolean,
      default: false,
    },
    squared: {
      type: Boolean,
      default: false,
    },
    multiline: {
      type: Boolean,
      default: false,
    },
    tag: {
      type: String,
      default: "button",
    },
  },
  computed: {
    type() {
      if (this.$attrs.type) {
        return this.$attrs.type;
      }

      if (this.tag === "button") {
        return "button";
      }

      return undefined;
    },
  },
};
</script>

<style lang="scss" scoped>
.button {
  @include mixin.clickable();
  @include mixin.truncate-text();
  box-sizing: border-box;
  position: relative;
  display: inline-block;
  vertical-align: top;
  max-width: 100%;
  height: 48px;
  padding: 0 28px;
  font-family: var.$font-family-sf-pro-text;
  text-align: center;
  text-transform: none;
  color: var.$white;
  background-color: var.$green-main;
  border: 1px solid transparent;
  border-radius: 48px;
  outline: none;
  touch-action: manipulation;
  transition: var.$transition-fast;

  &:hover {
    background-color: var.$green-hover;
  }

  &:active {
    background-color: var.$green-dark;
    transition-duration: 0s;
  }

  &._squared {
    border-radius: var.$rounded-8;
  }

  &:not(._multiline) {
    line-height: 46px;

    &._md {
      line-height: 38px;
    }
  }

  &._md {
    height: 40px;
    padding-right: 12px;
    padding-left: 12px;
  }

  &._multiline {
    height: auto;
    min-height: 48px;
    white-space: normal;
    padding-top: 13px;
    padding-bottom: 14px;

    &._md {
      min-height: 40px;
      padding-top: 10px;
      padding-bottom: 10px;
    }
  }

  &._gray {
    color: var.$black;
    background-color: var.$gray-light;

    &:hover {
      background-color: var.$gray-hover;
    }

    &:active {
      background-color: var.$gray-dark;
    }
  }

  &._black {
    color: var.$white;
    background-color: black;

    &:hover {
      background-color: black;
    }

    &:active {
      background-color: black;
    }
  }

  &:disabled,
  &._disabled {
    color: var.$secondary;
    background-color: var.$gray-light;
    pointer-events: none;
  }

  &._outline {
    color: var.$black;
    background-color: transparent;
    border-color: var.$green-main;

    &:hover {
      background-color: transparent;
      border-color: var.$green-hover;
    }

    &:active {
      background-color: transparent;
      border-color: var.$green-dark;
    }

    &._gray {
      border-color: var.$gray-dark;

      &:hover {
        border-color: var.$secondary;
      }

      &:active {
        border-color: var.$black;
      }
    }

    &._black {
      border-color: black;

      &:hover {
        border-color: black;
      }

      &:active {
        border-color: black;
      }
    }

    &._disabled {
      color: var.$secondary;
      border-color: var.$gray-dark;
    }
  }
}
</style>

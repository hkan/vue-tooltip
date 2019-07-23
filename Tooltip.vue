<style lang="postcss">
  .tooltip {
    @apply absolute -mt-2 rounded py-2 px-3 text-white z-40;
    background: rgba(0, 0, 0, 0.7);
    transition: opacity 0.15s ease-out;
    transform: translate(-50%, -100%);
    max-width: 300px;
    text-transform: none;
    pointer-events: none;

    &:after {
      content: "";
      position: absolute;
      top: 100%;
      left: 50%;
      margin-left: -5px;
      border-left: 5px solid transparent;
      border-right: 5px solid transparent;
      border-top: 5px solid rgba(0, 0, 0, 0.7);
    }

    &.bottom {
      @apply mt-2;
      transform: translate(-50%, 0);

      &:after {
        top: unset;
        bottom: 100%;
        border-top: 0;
        border-bottom: 5px solid rgba(0, 0, 0, 0.7);
      }
    }
  }
</style>

<script>
export default {
  props: {
    title: {
      type: String,
      required: true,
    },
    delay: {
      type: Number,
      default: 150,
    },
    bottom: {
      type: Boolean,
      default: false,
    },
  },

  methods: {
    onMouseOver() {
      this.removeFadeListener();

      if (this.$tooltip !== null) {
        this.$tooltip.style.opacity = 1;
        return;
      }

      this._timeout = setTimeout(() => {
        this._timeout = null;
        this.createTooltip();
      }, this.delay);
    },

    onMouseOut() {
      if (this._timeout) {
        return clearTimeout(this._timeout);
      }

      this.$tooltip.addEventListener('transitionend', this.onFadeOut, { passive: true });
      this.$tooltip.style.opacity = 0;
    },

    createTooltip() {
      const tt = this.$tooltip = document.createElement('div');

      tt.classList.add('tooltip');
      tt.setAttribute('role', 'tooltip');
      tt.innerText = this.title;
      tt.style.opacity = 0;

      const rectangle = this.$el.getBoundingClientRect();

      document.body.appendChild(tt);

      if (this.bottom) {
        tt.classList.add('bottom');
      }

      tt.style.top = `${rectangle.top + (this.bottom ? rectangle.height : 0) + window.scrollY}px`;
      tt.style.left = `${rectangle.right - (rectangle.right - rectangle.left) / 2}px`;

      setTimeout(() => tt.style.opacity = 1, 100);
    },

    onFadeOut() {
      if (!this.$tooltip) {
        return;
      }

      this.removeFadeListener();
      document.body.removeChild(this.$tooltip);
      this.$tooltip = null;
    },

    removeFadeListener() {
      if (!this.$tooltip) {
        return;
      }

      this.$tooltip.removeEventListener('transitionend', this.onFadeOut);
    },
  },

  mounted() {
    this.onMouseOver = this.onMouseOver.bind(this);
    this.onMouseOut = this.onMouseOut.bind(this);
    this.onFadeOut = this.onFadeOut.bind(this);
    this.$tooltip = null;

    this.$el.addEventListener('mouseenter', this.onMouseOver, { passive: true });
    this.$el.addEventListener('mouseleave', this.onMouseOut, { passive: true });
  },

  beforeDestroy() {
    if (this.$tooltip) {
      this.onMouseOut();
    }

    this.$el.removeEventListener('mouseenter', this.onMouseOver);
    this.$el.removeEventListener('mouseleave', this.onMouseOut);
  },

  render() {
    const el = this.$slots.default[0];

    if (el.text && !el.type) {
      return <span>{el}</span>;
    }

    return el;
  },
};
</script>


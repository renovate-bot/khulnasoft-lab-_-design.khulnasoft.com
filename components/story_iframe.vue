<script>
import { iframeResize } from 'iframe-resizer';
import { GlLoadingIcon } from '../helpers/gitlab_ui';
import { SkipOneTrustDirective } from '../directives/skip_one_trust_directive';

export default {
  components: {
    GlLoadingIcon,
  },
  directives: {
    skipOneTrust: SkipOneTrustDirective,
  },
  props: {
    url: {
      type: String,
      required: true,
    },
    iframePadding: {
      type: String,
      required: false,
      default: null,
    },
  },
  data() {
    return {
      loaded: false,
    };
  },
  computed: {
    iFrameClass() {
      return {
        'gl-opacity-0': !this.loaded,
      };
    },
    iframeResizeOptions() {
      const options = {};
      if (this.iframePadding) {
        options.bodyPadding = this.iframePadding;
      }
      return options;
    },
  },
  methods: {
    iFrameLoaded({ target }) {
      iframeResize(this.iframeResizeOptions, target);
      this.loaded = true;
    },
  },
};
</script>

<template>
  <div class="gl-relative">
    <div
      v-if="!loaded"
      class="gl-absolute gl-bottom-0 gl-left-0 gl-right-0 gl-top-0 gl-flex gl-flex-col gl-items-center gl-justify-center"
    >
      <gl-loading-icon size="lg" class="gl-mb-3" />
      Loading story...
    </div>
    <iframe
      v-skip-one-trust
      :src="url"
      :class="iFrameClass"
      class="responsive-iframe gl-min-w-full gl-border-none"
      allow="clipboard-write"
      @load="iFrameLoaded"
    ></iframe>
  </div>
</template>

<script>
import { GlAlert, GlNav, GlNavItem } from '../../helpers/gitlab_ui';
import { buildMeta } from '../../helpers/seo';

/*
We only need the "section" and "slug" of the routes to find the file.
Currently the "third" component is the "tab" (e.g. implementation on component pages)
and that is handled inside `componentinfo.vue` until we have better routing:
https://gitlab.com/gitlab-org/gitlab-services/design.gitlab.com/-/issues/1293
*/
const getPathFromRoute = (route) => {
  const { section, slug } = route.params;
  return [section, slug].filter(Boolean).join('/');
};

const componentNameToLabelMap = {
  dropdowns: 'dropdown',
  forms: 'form',
  labels: 'label',
  modals: 'modal',
  'radio-button': 'radio',
  tables: 'table',
  tabs: 'tab',
  toggles: 'toggle',
};

export default {
  components: {
    GlAlert,
    GlNav,
    GlNavItem,
  },
  scrollToTop: true,
  editThisPage: {
    resolve: ({ route }) => `contents/${getPathFromRoute(route)}.md`,
  },
  async asyncData({ $content, route, error }) {
    const path = getPathFromRoute(route);

    let page = null;

    try {
      page = await $content(path).fetch();
    } catch (e) {
      error({ statusCode: 404, path, message: `${path} not found`, stack: e.stack });
    }

    if (Array.isArray(page)) {
      error({
        statusCode: 500,
        path,
        message: `@nuxt/content returned an array of pages instead of a single page for '${path}'`,
      });
    }

    return { page };
  },
  data() {
    return {
      page: {},
    };
  },
  head() {
    return {
      title: this.page.name,
      meta: buildMeta({
        titleChunk: this.page.name,
        path: this.page.path,
        description: this.page.description,
      }),
    };
  },
  computed: {
    componentLabel() {
      const { section, slug } = this.$route.params;
      if (section !== 'components') {
        return null;
      }
      if (this.page.componentLabel !== undefined) {
        return this.page.componentLabel;
      }
      return componentNameToLabelMap[slug] || slug;
    },
    hasComponents() {
      return Boolean(this.page?.components?.length);
    },
    showTabs() {
      return Boolean(this.tabs.length);
    },
    tabs() {
      let { tabs = [] } = this.page;

      if (this.componentLabel) {
        tabs = [
          {
            route: 'section-slug',
            title: 'Usage',
          },
          ...tabs,
        ];

        if (this.hasComponents) {
          tabs.push({
            route: 'section-slug-code',
            title: 'Implementation (Vue.js)',
          });

          tabs.push({
            route: 'section-slug-lookbook',
            title: 'Implementation (Rails)',
          });
        }

        tabs.push({
          route: 'section-slug-contribute',
          title: 'Contribute',
        });
      }

      if (this.page.foundationLabel) {
        tabs = [
          {
            route: 'section-slug',
            title: 'Overview',
          },
          ...tabs,
        ];

        tabs.push({
          route: 'section-slug-contribute',
          title: 'Contribute',
        });
      }
      return tabs;
    },
    lastUpdatedAt() {
      const { lastGitUpdate } = this.page || {};
      if (!lastGitUpdate) {
        return null;
      }
      return new Date(lastGitUpdate).toLocaleString('en-US', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
      });
    },
  },
};
</script>

<template>
  <div class="container gl-py-7">
    <div class="md typography gl-mb-5">
      <h1 id="skipTarget" class="gl-heading-display !gl-mb-4 !gl-mt-0" tabindex="-1">
        {{ page.name }}
      </h1>
      <div v-if="page.deprecated" class="app-styles gl-mb-3">
        <gl-alert :dismissible="false" variant="warning">
          Please refrain from using this component - it is about to be deprecated!
        </gl-alert>
      </div>
      <p v-if="page.description">{{ page.description }}</p>
    </div>
    <div v-if="showTabs" class="app-styles">
      <gl-nav class="gl-tabs-nav !gl-mb-5">
        <gl-nav-item
          v-for="tab in tabs"
          :key="tab.route"
          exact
          :to="{ name: tab.route, params: $route.params }"
          link-classes="gl-tab-nav-item"
          active-class="gl-tab-nav-item-active"
        >
          {{ tab.title }}
        </gl-nav-item>
      </gl-nav>
    </div>
    <nuxt-child
      :page="page"
      :component-label="componentLabel"
      :foundation-label="page.foundationLabel"
    />
    <p v-if="lastUpdatedAt" class="gl-mt-5 gl-text-center">
      Last updated at:&nbsp;<time :datetime="lastUpdatedAt">{{ lastUpdatedAt }}</time>
    </p>
  </div>
</template>

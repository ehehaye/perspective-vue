<template>
  <div class="perspective-wrapper">
    <perspective-viewer ref="viewer" settings></perspective-viewer>

    <loading-overlay :visible="loading" />
  </div>
</template>

<script>
import { importStyle } from "@/utils/index.js";
import LoadingOverlay from "./LoadingOverlay.vue";

const CDN_PREFIX =
  process.env.NODE_ENV === "production"
    ? "https://cdn.jsdelivr.net/npm"
    : `${process.env.BASE_URL}libs`;

importStyle(`${CDN_PREFIX}/@perspective-dev/viewer/dist/css/pro.css`);
importStyle(`${CDN_PREFIX}/@perspective-dev/viewer/dist/css/intl/custom.css`);

let index = 0;

export default {
  name: 'PerspectiveWrapper',
  components: {
    LoadingOverlay,
  },
  props: {
    defaultPlugin: {
      type: String,
      default: "Datagrid",
    },
    arrow: {
      type: String,
      default: "https://cdn.jsdelivr.net/npm/superstore-arrow/superstore.arrow",
    },
    title: {
      type: String,
      default: "",
    },
    readonly: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      $viewer: null,
      loading: false,
      error: null,
      currentConfig: {},
      pluginConfigMap: new Map(),
    }
  },
  methods: {
    async init() {
      this.loading = true;
      this.error = null;
      try {
        const [perspective, buf] = await Promise.all([
          this.loadPerspectiveModules(),
          this.loadBuffer(),
        ]);

        const worker = await perspective.worker();
        this.$viewer = this.$refs.viewer;

        this.$viewer.addEventListener("perspective-config-update", this.onConfigUpdate);
        this.injectCss();

        const name = `my_arrow_${index++}`;
        await worker.table(buf, { name });
        await this.$viewer.load(worker);
        await this.$viewer.restore({
          table: name,
          plugin: this.defaultPlugin,
          settings: true,
          title: this.title || this.getTableName(),
        });
      } catch (e) {
        this.error = e;
        console.error("[PerspectiveWrapper] init failed:", e);
      } finally {
        this.loading = false;
      }
    },
    getTableName() {
      const path = this.arrow.split(/[?#]/)[0];
      const file = path.slice(path.lastIndexOf("/") + 1);
      return file.replace(/\.[^.]+$/, "") || "untitled";
    },
    injectCss() {
      if (this.readonly) {
        const sheet = new CSSStyleSheet();
        sheet.replaceSync(`#menu-bar { display: none !important; }`);
  
        const shadow = this.$viewer.shadowRoot
        shadow.adoptedStyleSheets = [...shadow.adoptedStyleSheets, sheet];
      }
    },
    onConfigUpdate(e) {
      this.setConfig(e.detail.getConfig());
    },
    async setConfig(config) {
      const { plugin } = config;

      // Switch to a previously saved plugin → restore its config
      if (
        this.currentConfig.plugin &&
        this.currentConfig.plugin !== plugin &&
        this.pluginConfigMap.has(plugin)
      ) {
        const savedConfig = this.pluginConfigMap.get(plugin);
        this.currentConfig = savedConfig;
        await this.$viewer.restore(savedConfig);
        return;
      }

      // Same plugin update / first initialization → update current config
      this.currentConfig = config;
      this.pluginConfigMap.set(plugin, config);
    },
    async loadPerspectiveModules() {
      const [perspectiveModule] = await Promise.all([
        import(/* webpackIgnore: true */ `${CDN_PREFIX}/@perspective-dev/client/dist/cdn/perspective.js`),
        import(/* webpackIgnore: true */ `${CDN_PREFIX}/@perspective-dev/viewer/dist/cdn/perspective-viewer.js`),
        import(/* webpackIgnore: true */ `${CDN_PREFIX}/@perspective-dev/viewer-charts/dist/cdn/perspective-viewer-charts.js`),
        import(/* webpackIgnore: true */ `${CDN_PREFIX}/@perspective-dev/viewer-datagrid/dist/cdn/perspective-viewer-datagrid.js`),
      ]);
      return perspectiveModule.default;
    },
    async loadBuffer() {
      return await fetch(this.arrow).then(r => r.arrayBuffer());
    }
  },
  mounted() {
    this.init();
  },
  beforeDestroy() {
    // https://perspective-dev.github.io/guide/how_to/javascript/deleting.html
    if (this.$viewer) {
      this.$viewer.removeEventListener("perspective-config-update", this.onConfigUpdate);
      this.$viewer.delete();
      this.$viewer = null;
    }
  },
}
</script>

<style>
.perspective-wrapper {
  position: relative;
  height: 600px;
}

perspective-viewer {
  display: block;
  height: 100%;
}
</style>

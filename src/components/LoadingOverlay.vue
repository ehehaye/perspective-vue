<template>
  <div v-show="visible" class="loading-overlay">
    <div class="loading-overlay__spinner"></div>
    <span class="loading-overlay__text">{{ text }}</span>
  </div>
</template>

<script>
export default {
  name: 'LoadingOverlay',
  props: {
    visible: {
      type: Boolean,
      default: false,
    },
    text: {
      type: String,
      default: '加载中...',
    },
  },
}
</script>

<style scoped>
.loading-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: rgba(255, 255, 255, 0.7);
  z-index: 10;
}

.loading-overlay__spinner {
  width: 36px;
  height: 36px;
  border: 3px solid #e0e0e0;
  border-top-color: #4a90d9;
  border-radius: 50%;
  animation: loading-overlay-spin 0.8s linear infinite;
}

.loading-overlay__text {
  font-size: 14px;
  color: #666;
}

@keyframes loading-overlay-spin {
  to {
    transform: rotate(360deg);
  }
}
</style>

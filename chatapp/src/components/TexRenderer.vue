<template>
  <div ref="container" class="math-container"></div>
</template>

<script setup>
import { onMounted, onUpdated, ref, defineProps } from 'vue';
import katex from 'katex';
import 'katex/dist/katex.min.css'; // KaTeX のスタイルをインポート

const props = defineProps({
  formula: {
    type: String,
    required: true
  }
});

const container = ref(null);

// 初期レンダリング
const renderFormula = () => {
  if (container.value) {
    try {
      katex.render(props.formula, container.value, {
        throwOnError: false,
        displayMode: true,
      });
    } catch (err) {
      console.error('KaTeX error:', err);
    }
  }
};

// コンポーネントのマウント時とプロパティ変更時にレンダリング
onMounted(renderFormula);
onUpdated(renderFormula);
</script>

<style scoped>
.math-container {
  font-size: 16px;
  margin: 10px 0;
}
</style>

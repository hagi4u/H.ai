# Vue 3 패턴

## Setup Composition API 파일 구조

`.vue` 파일 블록 순서는 항상 `<script>`, `<template>`, `<style>`이어야 한다.

```vue
<!-- 1. <script setup> (TypeScript 필수) -->
<script setup lang="ts">
import { ref } from "vue";
import Component from "@/components/Component.vue";

interface Props {
  value: string;
}

const props = withDefaults(defineProps<Props>(), {
  value: "",
});

const emit = defineEmits<{
  (e: "update", value: string): void;
}>();

const state = ref("");

const handleAction = (): void => {
  emit("update", state.value);
};
</script>

<!-- 2. <template> -->
<template>
  <div>
    <input v-model="state" @change="handleAction" />
  </div>
</template>

<!-- 3. <style> -->
<style scoped lang="scss">
div {
  padding: 1rem;
}
</style>
```

### 주의사항

- **외부 컴포넌트 사용 시**: import 후 자동으로 등록됨 (이름이 자동 매핑됨)
  - ❌ `defineComponent`로 별도 등록 불필요
- **TypeScript**: `withDefaults()`, `defineProps<Type>()`, `defineEmits<{...}>()` 사용
- **반응성**: `ref()`, `computed()`, `watch()`는 setup에서 직접 호출

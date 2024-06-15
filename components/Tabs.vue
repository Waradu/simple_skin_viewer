<template>
  <div class="tabs wrdu-floating">
    <div
      v-for="option in options"
      :key="option.value"
      class="tab wrdu-element"
      :class="{ active: model === option.value }"
      @click="updateModel(option.value)"
    >
      {{ option.text }}
    </div>
  </div>
</template>

<script lang="ts" setup>
defineProps({
  options: {
    type: Object as () => { value: string; text: string }[],
    required: true,
  },
});

const model = defineModel<string>({ required: true });

function updateModel(value: string) {
  model.value = value;
}
</script>

<style scoped>
.tabs {
  padding: 5px;
  display: flex;
  gap: 5px;

  .tab {
    border-radius: 100px;
    display: flex;
    justify-content: center;
    padding: 8px;
    width: 100%;
    cursor: pointer;
    align-items: center;
    font-size: 16px;
  }
}
</style>

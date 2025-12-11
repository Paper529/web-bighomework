<template>
  <div
    class="file-uploader"
    @drop.prevent="handleDrop"
    @dragover.prevent="isDragging = true"
    @dragleave.prevent="isDragging = false"
    @click="triggerFileInput"
  >
    <input
      ref="fileInput"
      type="file"
      :accept="accept"
      @change="handleFileChange"
      style="display: none"
    />
    <slot :isDragging="isDragging"></slot>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

interface Props {
  accept?: string;
}

const props = defineProps<Props>();

const emit = defineEmits<{
  'file-selected': [file: File];
}>();

const fileInput = ref<HTMLInputElement | null>(null);
const isDragging = ref(false);

const triggerFileInput = () => {
  fileInput.value?.click();
};

const handleFileChange = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  if (file) {
    emit('file-selected', file);
  }
};

const handleDrop = (event: DragEvent) => {
  isDragging.value = false;
  const file = event.dataTransfer?.files[0];
  if (file) {
    emit('file-selected', file);
  }
};
</script>

<style scoped>
.file-uploader {
  cursor: pointer;
}
</style>


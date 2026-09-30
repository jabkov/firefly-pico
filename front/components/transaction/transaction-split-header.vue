<template>
  <div class="flex-center-vertical px-3 pt-2">
    <div class="flex-1 text-size-14 font-weight-600">{{ $t('transaction.split_part', { index: props.index + 1, total: props.total }) }}</div>
    <van-button size="small" plain hairline type="danger" icon="delete-o" class="cursor-pointer" @click="emit('remove')">{{ $t('delete') }}</van-button>
  </div>
</template>

<script setup>
const props = defineProps({
  index: {
    type: Number,
    required: true,
  },
  total: {
    type: Number,
    required: true,
  },
})

const emit = defineEmits(['remove'])
</script>

<template>
  <div>
    id: {{ data.physicalId }}
    <div v-if="data.adopted">
      {{ data.name }} - {{ data.gender }}
    </div>
  </div>
</template>

<script>
export default {
  name: "PopupPony",
  props: {
    data: {
      type: Object,
      default: () => ({})
    }
  }
};
</script>

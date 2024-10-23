<template>
    <div class="modal fade show" tabindex="-1" style="display: block;">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="">{{ isEditing ? 'แก้ไขข้อมูล' : 'เพิ่มข้อมูล' }}</h5>
            <button type="button" class="btn-close" @click="$emit('close')"></button>
            <slot name="header"></slot>
          </div>
          <div class="modal-body">
            <slot name="body"></slot>
          </div>
          <div class="modal-footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </div>
  </template>
  <script>
export default {
  props: {
    isEditing: {
      type: Boolean,
      required: true
    }
  }
}
</script>
  <style scoped>
  .modal {
    background-color: rgba(0, 0, 0, 0.5); 
  }
  </style>
  
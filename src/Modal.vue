<template>
  <div class="modal is-active">
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title"><slot name="head"></slot></p>
        <button class="delete" aria-label="close" @click="$emit('close')"></button>
      </header>
      <section class="modal-card-body">
        <slot></slot>
      </section>
      <footer class="modal-card-foot">
        <slot name="foot">
          <button class="button is-success" @click="$emit('close')">Okay</button>
        </slot>
      </footer>
    </div>
  </div>
</template>

<script>
export default {

}
</script>

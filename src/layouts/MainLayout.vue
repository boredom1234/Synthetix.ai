<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script lang="ts" setup></script>

<style scoped lang="scss">
.q-page-container {
  background: rgb(17, 17, 64);
  background: linear-gradient(
    135deg,
    rgba(17, 17, 64, 0.9) 0%,
    rgba(0, 68, 68, 0.9) 50%,
    rgba(0, 0, 68, 0.9) 100%
  );
  background-size: 50px 50px; /* set the size of the dots */
  background-color: #232323; /* set the color of the dots */
  background-image: radial-gradient(
    circle,
    #fff 1px,
    transparent 1px
  ); /* create the dots */
  animation: connect-dots 2s linear infinite; /* add the animation */
}

@keyframes connect-dots {
  from {
    background-position: 0 0; /* start from top-left */
  }
  to {
    background-position: -50px -50px; /* end at bottom-right */
  }
}
</style>

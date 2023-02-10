<script setup lang="ts">
  import type {Hello} from '$/types'
  import {boo} from '$/types'

  const s:Hello = 'Hello'

  console.log('Vue', s, boo);

</script>

<template>
  <h5>Hello</h5>
</template>


<template>
  <div class="hello">
   {{msg}}
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){ return {msg: 'pepperonipizza'}}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello { border: 1px solid red;}
</style>

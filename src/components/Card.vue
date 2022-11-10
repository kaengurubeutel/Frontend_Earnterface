
<script setup>

defineProps({
  bot: String,
  msg: String,
})

</script>

<template>
        <p v-if="bot === 'true'" class="text bot">{{msg}}</p>
        <p v-if="bot === 'false'" class="text user">{{msg}}</p>
</template>


<style scoped>


.text{
  margin-top: 50px;
  width: 500px;
  padding: 20px;
  border-radius: 13px;
}

.bot{
  position: relative;
  right: 100px;
  background-color: #5A92E4;
  padding-left: 30px;
  text-align: left;
}

.user{
  text-align: right;
  padding-right: 30px;
  position: relative;
  left: 100px;
  background-color: #59D391;
}

</style>
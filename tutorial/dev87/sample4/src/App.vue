<script setup>
const handleClick = () => {
  alert("ボタンがクリックされました")
}

const handleClickOver = () => {
  alert("ボタンがホバーされました")
}
</script>

<template>
  <h1>Sample4</h1>
  <button @click="handleClick">click</button>
  <button @mouseover="handleClickOver">MouseOver</button>
</template>





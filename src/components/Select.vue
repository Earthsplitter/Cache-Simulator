<template>
  <div>
    <label :for="model.name">{{model.name}}:</label>
    <select :id="model.name" v-model="model.selected">
      <option v-for="option in model.options">
        {{option}}
    </option>
    </select>
  </div>
</template>

<script>
  export default {
    name: 'select',
    props: ['model']
  }
</script>

<style scoped>
  select {
    margin-left: 20px;
  }
  div {
    margin: 10px 0;
  }
  label:hover {
    cursor: pointer;
    color: #00bc9b;
  }
</style>

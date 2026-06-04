<template>
  <div class="w-6/12 m-auto px-6 py-12 border border-gray-300 rounded-x1">
    <h1 class=""></h1>
    <form>
      <label for="task-title">Task title:</label> <br />
      <input
        type="text"
        class="px-4 w-full mt-2 py-2 border border-gray-200 rounded-b-md"
      />

      <!-- Priority -->
      <div>
        <label class="font-medium">Task Priority:</label>
        <select
          v-model="taskPriority"
          class="w-full mt-2 px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-400"
        >
          <option disabled selected>Choose Priority</option>
          <option value="High">High</option>
          <option value="Medium">Medium</option>
          <option value="Low">Low</option>
        </select>
      </div>
    </form>
  </div>
</template>

<script setup>
const isShowForm = false;
</script>

<style lang="scss" scoped></style>

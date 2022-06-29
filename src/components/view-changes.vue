<template>
  <div class="heat-map view-changes">
    <div v-if="Selecteddate">
      <h2>Showing changes that occured on {{ Selecteddate }}</h2>
      <div v-for="change in detail" :key="change.productId" class="description">
        <i class="fa fa-calendar-check-o"></i> {{ change.description }}
      </div>
    </div>
    <div v-else><h2>Select a date to display</h2></div>
  </div>
</template>
<script lang="ts">
import Details from "@/domains/DateDetails";
import { defineComponent, toRef } from "vue";

export default defineComponent({
  props: ["details", "Selecteddate"],
  setup(props) {
    let detail = toRef(props, "details");
    return { detail };
  },
});
</script>
<style>
.view-changes {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.view-changes h2 {
  font-weight: 800;
}
.description {
  padding: 10px;
  margin-top: 20px;
  text-align: left;
}
</style>

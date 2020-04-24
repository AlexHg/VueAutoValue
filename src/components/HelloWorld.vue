<template>
  <div class="hello">
    <button @click="addGridElement">Agregar</button>
    <div v-for="gridElement in grid" :key="gridElement.id">
      <input type="number" v-model="gridElement.percent" @blur="validatePercent(gridElement.id)">
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  methods: {
    addGridElement() {
      this.grid.push({
        ...this.proto,
        id: this.grid.length
      });
      //console.log(this.grid.length - 1);
      this.validatePercent(this.grid.length - 1);
    },
    validatePercent(index) {
      let virtual_grid = [...this.grid];
      //console.log(virtual_grid[index]);
      let virtual_grid_selected_element = { ...virtual_grid[index] };
      virtual_grid[index].percent = 0;
      virtual_grid.push(virtual_grid_selected_element);

      let global_current_percent = 0;
      //console.log(virtual_grid);

      virtual_grid.map((gridElement, key) => {
        let temporal_current_percent = global_current_percent;
        let overflow = 0;
        temporal_current_percent += parseInt(gridElement.percent, 0);
        console.log(temporal_current_percent, this.global_max_percent);
        if (temporal_current_percent > this.global_max_percent) {
          overflow = this.global_max_percent - global_current_percent;
          console.log(`Te pasaste por ${overflow}, piensalo mejor!`);
          gridElement.percent = overflow;
        } else {
          global_current_percent = temporal_current_percent;
        }
        return gridElement;
      });
      virtual_grid[index].percent = virtual_grid.pop().percent;
      //this.grid = virtual_grid;
      console.log("_________");
      return virtual_grid[index];
    },
    calculatePercent() {}
  },
  data() {
    return {
      global_max_percent: 100,
      proto: {
        id: 0,
        percent: 0
      },
      grid: []
    };
  }
};
</script>
